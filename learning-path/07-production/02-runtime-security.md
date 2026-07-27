# 2. User, permission và runtime security

> **Tóm tắt một câu:** Runtime security giảm quyền process có thể sử dụng và giảm blast radius nếu ứng dụng hoặc dependency bị khai thác.

> **Loại:** Explanation · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Docker Engine security](https://docs.docker.com/engine/security/) · [Dockerfile USER](https://docs.docker.com/reference/dockerfile/#user)

> **Sau chapter này, bạn có thể:**
> - Phân biệt identity, file ownership, Linux capability và privileged mode.
> - Đọc cú pháp `USER`, `COPY --chown`, `cap_drop` và `read_only` theo đúng scope.
> - Kiểm tra effective runtime user và security configuration thay vì chỉ tin Dockerfile.

> **Thuật ngữ:** **Least privilege** là chỉ cấp quyền tối thiểu cần thiết. **Blast radius** là phạm vi tài nguyên có thể bị ảnh hưởng. **Capability** là một đơn vị quyền đặc biệt của Linux được tách nhỏ từ quyền root. **UID/GID** là định danh số của user/group mà kernel dùng khi kiểm tra quyền file; tên `app` chỉ là ánh xạ dễ đọc trong filesystem của Container.

[← 1. Production readiness](01-production-readiness.md) · [Mục lục Production](README.md) · [3. Configuration và secret →](03-configuration-va-secret.md)

---

## 1. Root trong Container vẫn cần được xem xét

Linux Container dùng namespaces, capabilities, seccomp và mount configuration để hạn chế góc nhìn của process. Tuy nhiên, process chạy UID `0` thường vẫn có nhiều quyền hơn cần thiết. Khi có mount nhạy cảm, capability nguy hiểm hoặc `--privileged`, khoảng cách giữa Container với host bị thu hẹp mạnh.

**Namespace** — cơ chế làm process nhìn thấy phạm vi tài nguyên riêng như process IDs, network interface hoặc mount tree. **Seccomp** — cơ chế lọc system call mà process được phép yêu cầu kernel thực hiện. Hai cơ chế này giảm bề mặt tác động nhưng không làm process root trở nên vô hại.

Chạy non-root không làm ứng dụng bất khả xâm phạm. Nó loại bỏ một nhóm quyền mặc định và buộc thiết kế filesystem rõ ràng hơn.

## 2. `USER` và ownership là hai phần khác nhau

```dockerfile
RUN addgroup --system app && adduser --system --ingroup app app
WORKDIR /app
COPY --chown=app:app app.jar app.jar
USER app
ENTRYPOINT ["java", "-jar", "app.jar"]
```

| Cú pháp | Owner/scope | State sau instruction |
|---|---|---|
| `COPY --chown=app:app app.jar app.jar` | Image đang build | File đích thuộc user `app`, group `app`; source vẫn thuộc build context. |
| `USER app` | Image configuration | Đặt identity mặc định cho instruction tiếp theo và runtime process về sau. |
| `ENTRYPOINT [...]` | Image configuration | Process runtime mặc định sẽ được start dưới effective user sau runtime override. |

`USER app` không tự đổi owner của file đã copy trước đó. `COPY --chown=app:app` đặt owner ngay khi tạo file trong Image filesystem. Nếu chỉ đổi user nhưng thư mục cần ghi vẫn thuộc root, ứng dụng có thể gặp `Permission denied`.

```bash
docker container run --rm my-app:1.0 id
docker image inspect my-app:1.0 --format '{{json .Config.User}}'
```

Lệnh đầu tạo Container tạm và in effective UID/GID; `--rm` xóa Container sau khi process kết thúc. Lệnh sau chỉ đọc mặc định trong Image. Runtime option `--user` có thể override giá trị mà không sửa Image, vì vậy Image config và runtime identity phải được kiểm tra riêng.

## 3. Capability và privileged mode

Docker cấp một tập capability mặc định cho Linux Container. Workload có thể bỏ thêm quyền không cần dùng:

```yaml
services:
  api:
    image: example/api:1.0
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

`cap_drop` và `cap_add` thuộc Service `api`, được Compose chuyển thành runtime capability set cho Container. `ALL` bỏ toàn bộ capability có thể bỏ theo cơ chế này; `NET_BIND_SERVICE` thêm lại quyền bind low port trên Linux khi workload thật sự cần.

Không dùng `privileged: true` như cách nhanh để sửa permission: privileged mode mở rộng quyền rất lớn và che mất nguyên nhân thật. Least privilege yêu cầu bắt đầu từ quyền nhỏ, quan sát lỗi cụ thể rồi chỉ thêm capability có lý do và test.

```bash
docker container inspect api --format 'User={{json .Config.User}} CapDrop={{json .HostConfig.CapDrop}} CapAdd={{json .HostConfig.CapAdd}} Privileged={{.HostConfig.Privileged}}'
```

Các field thuộc Container runtime metadata. Lệnh không chứng minh toàn bộ kernel policy, nhưng phát hiện nhanh trường hợp Compose chưa recreate Container hoặc option nằm sai scope.

## 4. Read-only root filesystem

```yaml
services:
  api:
    image: example/api:1.0
    read_only: true
    tmpfs:
      - /tmp
```

`read_only: true` ngăn ghi vào root filesystem của Container. **tmpfs** — filesystem tạm nằm trong memory của host theo cơ chế Linux — tạo vùng ghi cho `/tmp`. Cấu hình này không ngăn ghi vào Volume được mount read-write, không mã hóa dữ liệu và tmpfs vẫn tiêu thụ memory.

Trước `up`, Container chưa có mount `/tmp` này. Sau khi create, `/tmp` là mount ghi được trong khi root filesystem còn lại read-only.

```bash
docker container inspect api --format 'ReadonlyRootfs={{.HostConfig.ReadonlyRootfs}} Mounts={{json .Mounts}}'
docker container exec api sh -c 'touch /should-fail'
docker container exec api sh -c 'touch /tmp/should-work'
```

Hai lệnh `exec` giả định Image có `sh`. Phép thử đầu nên thất bại nếu root filesystem thực sự read-only; phép thử thứ hai chỉ thành công nếu `/tmp` đã được cấp vùng ghi và permission phù hợp.

## 5. Các ranh giới cần review cùng nhau

- Không mount Docker socket vào workload thông thường; socket cho phép gửi yêu cầu điều khiển Docker Engine và có blast radius rất lớn.
- Không đưa credential vào Image layer hoặc command line nếu có cơ chế an toàn hơn.
- Ưu tiên base Image có nguồn rõ ràng, được cập nhật và không chứa package thừa.
- Vulnerability scan là tín hiệu hỗ trợ quyết định, không phải chứng nhận an toàn tuyệt đối.
- Mount, Linux capability, seccomp profile và network exposure phải được đánh giá cùng identity; một lớp tốt không bù hoàn toàn cho lớp khác mở quá rộng.

## 6. Quan niệm dễ sai

### “Root trong Container không liên quan gì đến root trên host.”

- **Phân loại:** Sai do tuyệt đối hóa.
- **Vì sao nghe hợp lý:** Namespace làm process chỉ nhìn thấy một phần hệ thống và Container thường không có toàn bộ host filesystem.
- **Lỗi kỹ thuật:** Isolation giảm quyền nhìn thấy host, nhưng UID, capabilities, mounts và runtime configuration quyết định mức tác động thực tế.
- **Cách nói đúng hơn:** Root trong Container bị giới hạn hơn host root trong cấu hình thông thường, nhưng vẫn là identity quyền cao cần thu hẹp.

### “Image chạy non-root thì luôn ghi được file cần thiết.”

- **Phân loại:** Sai.
- **Lỗi kỹ thuật:** Identity và filesystem ownership là hai phần khác nhau.
- **Kiểm chứng:** Dùng `id`, `ls -ln <path>` và thử ghi vào đúng thư mục ứng dụng cần sử dụng.

### “`read_only: true` nghĩa Container không thể ghi dữ liệu ở đâu.”

- **Phân loại:** Sai do bỏ qua mount.
- **Lỗi kỹ thuật:** Root filesystem có thể read-only trong khi Volume, bind mount hoặc tmpfs vẫn được mount read-write.
- **Kiểm chứng:** Đọc `.HostConfig.ReadonlyRootfs` cùng `.Mounts`, rồi thử đúng target path.

## 7. Tự kiểm tra mental model

1. Vì sao `USER app` không sửa được `Permission denied` nếu `/app/data` vẫn thuộc UID `0`?
2. `cap_add` khác `privileged: true` ở mức độ mở rộng quyền như thế nào?
3. Tại sao phải kiểm tra runtime Container sau khi sửa Compose security keys?

## 8. Tóm tắt

1. Non-root là nền tảng tốt nhưng phải đi cùng ownership phù hợp.
2. Capability, seccomp và read-only filesystem giảm quyền theo các chiều khác nhau.
3. `privileged` và Docker socket là mở rộng quyền lớn, không phải cách sửa lỗi chung.
4. Desired security configuration phải được đối chiếu với effective runtime state.

## Tài liệu tham khảo

- Docker Docs, [Docker security](https://docs.docker.com/engine/security/)
- Dockerfile Reference, [USER](https://docs.docker.com/reference/dockerfile/#user)
- Docker Docs, [Runtime privilege and Linux capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)

[← 1. Production readiness](01-production-readiness.md) · [Mục lục Production](README.md) · [3. Configuration và secret →](03-configuration-va-secret.md)
