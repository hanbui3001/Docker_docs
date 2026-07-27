# 6. Dọn dẹp tài nguyên Docker

> **Tóm tắt một câu:** Cleanup an toàn bắt đầu bằng inventory và scope; lệnh càng tổng quát càng cần hiểu chính xác object nào được coi là unused và dữ liệu nào không thể khôi phục.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 35 phút<br>
> **Nguồn chính:** [`docker system df`](https://docs.docker.com/reference/cli/docker/system/df/) · [`docker system prune`](https://docs.docker.com/reference/cli/docker/system/prune/)

> **Sau chapter này, bạn có thể:**
> - Đo disk usage trước khi xóa.
> - Phân biệt remove có mục tiêu với prune theo tập.
> - Giải thích phạm vi mặc định và phạm vi mở rộng bởi `--all`, `--volumes`, `--force`.
> - Xây quy trình cleanup có khả năng kiểm chứng và recovery hợp lý.

[← 5. Quan sát và debug Container](05-quan-sat-va-debug-container.md) · [Mục lục Part 02](README.md)

---

## 1. “Không chạy” không đồng nghĩa “không dùng”

Docker quản lý nhiều object có lifecycle độc lập: Container, Image, Network, Volume và build cache. Một Image không chạy process nhưng có thể cần để start/recreate; một stopped Container có thể giữ writable data; một named Volume có thể không attach nhưng chứa database cần giữ.

Cleanup đúng cần trả lời:

1. Object nào chiếm dung lượng?
2. Object nào còn dependency?
3. Dữ liệu nào có nguồn tái tạo, dữ liệu nào là bản duy nhất?
4. Lệnh sẽ xóa target cụ thể hay mọi object khớp điều kiện `unused`?

## 2. Quan sát trước bằng `system df`

```bash
docker system df
docker system df --verbose
```

Output nhóm dung lượng Image, Container, local Volume và build cache. `RECLAIMABLE` là ước lượng phần có thể thu hồi theo logic Docker, không phải lời khẳng định dữ liệu đó vô giá trị với bạn.

`--verbose` giúp nhìn chi tiết reference và shared/unique size. Hãy kết hợp với inventory:

```bash
docker container ls --all
docker image ls
docker volume ls
docker network ls
```

## 3. Remove có mục tiêu

Ưu tiên lệnh chỉ rõ target khi biết chính xác tài nguyên demo:

```bash
docker container stop web
docker container rm web
docker image rm my-app:demo
```

Đây là cleanup dễ review: tên object nằm ngay trong command. Nếu remove thất bại do dependency, lỗi là tín hiệu cần kiểm tra quan hệ, không phải lời mời dùng `--force` ngay.

### Bash và PowerShell khi nối lệnh

Bash và PowerShell 7:

```bash
docker container stop web && docker container rm web
```

Windows PowerShell 5.1:

```powershell
docker container stop web
if ($LASTEXITCODE -eq 0) { docker container rm web }
```

Dùng `;` sẽ chạy lệnh sau bất kể exit code của lệnh trước, nên không tương đương `&&`.

## 4. Container prune

```bash
docker container prune
```

Lệnh xóa mọi stopped Container phù hợp filter. Trước lệnh, các Container `exited`/`created` có thể giữ metadata, logs và writable layer. Sau lệnh, object bị xóa không thể `start` hoặc `inspect` lại.

> [!WARNING]
> Stopped Container có thể chứa dữ liệu chỉ nằm trong writable layer. `container prune` không biết dữ liệu đó quan trọng với bạn hay không.

Thu hẹp theo thời gian hoặc label khi phù hợp:

```bash
docker container prune --filter "until=24h"
```

Filter giúp giới hạn tập nhưng vẫn cần đọc danh sách object trước.

## 5. Image prune

```bash
docker image prune
```

Mặc định xóa dangling Image. Thêm `--all`:

```bash
docker image prune --all
```

sẽ mở rộng tới Image không được Container hiện có tham chiếu. Điều này có thể xóa Image hợp lệ đang cache cho lần chạy/deploy sau, khiến bạn phải pull hoặc build lại.

Recovery thường là pull/build lại nếu nguồn còn tồn tại; nhưng Tag có thể đã di chuyển, nên không bảo đảm lấy lại đúng content nếu không biết Digest.

## 6. Builder prune

```bash
docker builder prune
```

**Build cache** là dữ liệu builder giữ để tái sử dụng kết quả bước build. Xóa cache không nhất thiết xóa tagged Image hiện có, nhưng build sau có thể chậm hơn và phải tải/tính lại nhiều bước.

`--all` mở rộng cache bị xóa. Đây là trade-off giữa disk space và tốc độ build, không phải “rác luôn an toàn”.

## 7. System prune

```bash
docker system prune
```

Theo phạm vi mặc định hiện hành, lệnh dọn nhiều nhóm unused object như stopped Container, network không dùng, dangling Image và build cache. Nó không xóa running Container. Named Volume không bị xóa theo mặc định.

```bash
docker system prune --all
```

`--all` mở rộng từ dangling Image sang mọi unused Image. Nó vẫn không có nghĩa “xóa sạch Docker” và không mặc định xóa named Volume.

```bash
docker system prune --volumes
```

> [!WARNING]
> `--volumes` mở rộng cleanup sang Volume đủ điều kiện. Volume thường chứa dữ liệu bền vững như database; recovery chỉ có thể dựa vào backup hoặc nguồn dữ liệu khác, không dựa vào Image.

Đừng dùng `docker system prune -a --volumes` như lệnh sửa mọi lỗi. Nó thay đổi nhiều object class cùng lúc, làm mất bằng chứng debug và cache hữu ích.

## 8. `--force` có hai nghĩa thường gặp

| Ngữ cảnh | Ý nghĩa |
|---|---|
| `docker container rm --force web` | Cho phép kết thúc cưỡng bức Container đang chạy rồi xóa target. |
| `docker system prune --force` | Bỏ prompt xác nhận; không tự mở rộng object class như `--all` hay `--volumes`. |

Option cùng tên không bảo đảm cùng hậu quả ở mọi command. Luôn đọc help của action cụ thể.

## 9. Quy trình cleanup an toàn

```text
Inventory -> xác định owner/purpose -> kiểm tra dependency
-> remove target cụ thể -> verify -> mới cân nhắc prune rộng hơn
```

Ví dụ kiểm chứng:

```bash
docker system df
docker container ls --all
docker image ls
```

Sau cleanup, chạy lại cùng inventory để xác nhận đúng object biến mất và object cần giữ còn tồn tại. Nếu mục tiêu là giải phóng disk, so sánh `docker system df` trước/sau thay vì chỉ tin output “Deleted”.

## 10. Quan niệm dễ gây hiểu nhầm

### 10.1 “Prune chỉ xóa đồ hỏng”

- **Phân loại:** Sai.
- **Lỗi kỹ thuật:** Docker đánh giá unused/dangling theo quan hệ object, không hiểu giá trị nghiệp vụ hoặc ý định dùng lại.
- **Cách nói tốt hơn:** Prune xóa tập object thỏa điều kiện kỹ thuật của command.

### 10.2 “`system prune -a` xóa cả named Volume”

- **Phân loại:** Sai theo phạm vi mặc định.
- **Lỗi kỹ thuật:** `-a` mở rộng Image scope; Volume cần option/phạm vi riêng.
- **Cảnh báo:** Không vì vậy mà Volume luôn an toàn; command khác như Compose down với volume có thể xóa chúng.

### 10.3 “Nếu cần thì pull lại Image giống hệt”

- **Phân loại:** Chỉ đúng khi reference vẫn resolve cùng content.
- **Lỗi kỹ thuật:** Tag có thể di chuyển; không lưu Digest thì pull sau có thể nhận Image mới.
- **Cách nói tốt hơn:** Digest và registry retention quyết định khả năng tái tạo đúng artifact.

## 11. Tự kiểm tra mental model

1. Vì sao stopped Container có thể không phải rác?
2. `image prune` và `image prune --all` khác tập mục tiêu thế nào?
3. `system prune --force` có tự tương đương `--all --volumes` không?
4. Xóa build cache ảnh hưởng điều gì dù ứng dụng hiện tại vẫn chạy?

## 12. Tóm tắt

1. Đo và inventory trước khi xóa.
2. Remove theo target an toàn và dễ kiểm chứng hơn prune rộng.
3. `--all`, `--volumes` và `--force` thay đổi các khía cạnh khác nhau.
4. Docker biết dependency kỹ thuật, không biết giá trị dữ liệu của bạn.
5. Cleanup hoàn tất khi verify đúng scope, không chỉ khi command trả exit code 0.

## 13. Học tiếp

Bạn đã hoàn thành Part 02. Tiếp theo là Part 03 — Storage & Networking, nơi writable layer, Volume, bind mount, network và port publishing được giải thích sâu hơn. Dùng [Docker CLI quick reference](../../reference/commands/README.md) để ôn lệnh.

## Tài liệu tham khảo

- Docker CLI, [`docker system df`](https://docs.docker.com/reference/cli/docker/system/df/)
- Docker CLI, [`docker system prune`](https://docs.docker.com/reference/cli/docker/system/prune/)
- Docker CLI, [`docker image prune`](https://docs.docker.com/reference/cli/docker/image/prune/)
- Docker CLI, [`docker container prune`](https://docs.docker.com/reference/cli/docker/container/prune/)
- Docker CLI, [`docker builder prune`](https://docs.docker.com/reference/cli/docker/builder/prune/)

[← 5. Quan sát và debug Container](05-quan-sat-va-debug-container.md) · [Mục lục Part 02](README.md)
