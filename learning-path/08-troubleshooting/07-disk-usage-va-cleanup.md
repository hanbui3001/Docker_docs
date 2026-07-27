# 7. Disk usage và cleanup

> **Tóm tắt một câu:** Khi Docker chiếm nhiều disk, phải đo theo object và xác định dữ liệu có thể tái tạo trước khi prune; cleanup không có một lệnh “an toàn cho mọi máy”.

> **Thuật ngữ:** **Reclaimable** là dung lượng Docker ước tính có thể thu hồi, không đồng nghĩa dữ liệu chắc chắn vô giá trị. **Dangling Image** là Image không còn tag nhưng content có thể vẫn được object khác tham chiếu. **Build cache** là kết quả trung gian giúp build sau nhanh hơn.

[← 6. Lỗi Docker Compose](06-loi-docker-compose.md) · [Mục lục Troubleshooting](README.md) · [8. Diagnostic playbook →](08-diagnostic-playbook.md)

---

## 1. Symptom: disk gần đầy

Trước khi xóa, xác nhận filesystem nào đầy và Docker data root nằm đâu:

```bash
docker info --format 'DockerRootDir={{.DockerRootDir}}'
docker system df -v
```

`docker system df -v` phân tách Images, Containers, local Volumes và build cache. Host disk tool vẫn cần thiết vì log, source, database ngoài Docker hoặc filesystem khác cũng có thể là nguyên nhân.

## 2. Hypothesis theo loại object

| Evidence | Hypothesis |
|---|---|
| Nhiều stopped Container | Writable layer/log của Container cũ tích lũy |
| Nhiều Image không dùng | Release/dev build cũ còn local |
| Build cache lớn | Build lặp nhiều context/stage/platform |
| Volume lớn | Dữ liệu nghiệp vụ, cache hoặc orphaned Volume |
| Container log lớn | Logging không rotation hoặc application log quá nhiều |

**Orphaned resource** là object không còn workflow hiện tại tham chiếu nhưng vẫn tồn tại. “Không attach vào Container” chưa đủ kết luận Volume có thể xóa; nó có thể là backup tạm hoặc chờ Container mới mount lại.

## 3. Điều tra Container và Image

```bash
docker container ls -a --size
docker image ls
docker image ls --filter dangling=true
docker system df -v
```

Kiểm tra Container nào đã dừng, vì sao còn giữ và writable layer có dữ liệu cần lấy không. Với Image, xác định pipeline/developer nào còn cần version cũ và liệu Registry có bản có thể pull lại.

### Correction tăng dần mức phá hủy

```bash
docker container prune
docker image prune
docker builder prune
```

Mỗi lệnh có prompt và scope khác nhau. Đọc danh sách/ước lượng trước; không gộp tất cả chỉ vì muốn nhanh.

### Verification

```bash
docker system df -v
```

So dung lượng trước/sau và xác nhận workload quan trọng vẫn chạy/build được. Cleanup build cache có thể làm build tiếp theo chậm hơn; đó không phải regression chức năng.

## 4. Điều tra Volume

```bash
docker volume ls
docker volume inspect <volume>
docker container ls -a --filter volume=<volume>
```

Ghi lại Volume name, labels, mountpoint, project và Container từng dùng. Nếu cần biết dữ liệu gì bên trong, dùng Image công cụ phù hợp và ưu tiên read-only mount.

> [!CAUTION]
> `docker volume prune` xóa Volume không được Container tham chiếu theo tiêu chí của Engine. Một Volume không attached vẫn có thể chứa database cần giữ. Phải có inventory và backup trước khi xác nhận.

### Verification

Sau cleanup, đối chiếu danh sách Volume với inventory, kiểm tra database/application data và khả năng restore nếu có backup.

## 5. Container log và disk tăng liên tục

### Evidence

```bash
docker inspect <container> --format 'driver={{.HostConfig.LogConfig.Type}} config={{json .HostConfig.LogConfig.Config}} log={{.LogPath}}'
docker logs --since 10m <container> | head
```

Đường dẫn và cơ chế log phụ thuộc logging driver. Không sửa/xóa trực tiếp file nội bộ Docker khi Engine đang quản lý nó.

### Correction

- Giảm log vô ích tại application.
- Cấu hình rotation/retention phù hợp với logging driver.
- Chuyển log tới hệ thống tập trung khi cần retention/audit.

### Verification

Quan sát tốc độ tăng disk qua khoảng thời gian đại diện và kiểm tra log mới vẫn truy xuất được.

## 6. `docker system prune` và `-a`

> [!WARNING]
> `docker system prune` xóa nhiều nhóm object không dùng; `-a` mở rộng sang mọi Image không được Container tham chiếu, không chỉ dangling Image. `--volumes` mở rộng sang Volume theo tiêu chí prune. Không chạy trên máy chứa dữ liệu quan trọng nếu chưa đọc dry inventory, backup và hiểu chính xác version Docker đang dùng.

Không có dry-run hoàn chỉnh cho mọi prune command. Cách an toàn là liệt kê theo từng object, xóa mục tiêu cụ thể và đo lại.

## 7. Quy trình cleanup an toàn

```text
Measure -> classify -> identify owner -> confirm recoverability
-> remove smallest scope -> measure again -> verify workloads
```

Ưu tiên `docker container rm <id>` hoặc `docker image rm <reference>` cho object đã xác nhận trước khi dùng prune diện rộng.

## 8. Quan niệm dễ sai

### “RECLAIMABLE là dung lượng xóa chắc chắn an toàn.”

Đó là ước tính kỹ thuật về reference, không hiểu giá trị nghiệp vụ hay kế hoạch sử dụng lại.

### “Xóa cache không ảnh hưởng gì.”

Không làm thay đổi source, nhưng build sau phải tải/chạy lại step, tăng thời gian và network usage.

## 9. Tóm tắt

1. Đo Docker root và usage theo object.
2. Xác định owner, giá trị và khả năng tái tạo.
3. Xóa phạm vi nhỏ nhất trước.
4. Volume và `system prune -a --volumes` cần mức cảnh báo cao nhất.

## Tài liệu tham khảo

- Docker CLI, [docker system df](https://docs.docker.com/reference/cli/docker/system/df/)
- Docker CLI, [docker system prune](https://docs.docker.com/reference/cli/docker/system/prune/)

[← 6. Lỗi Docker Compose](06-loi-docker-compose.md) · [Mục lục Troubleshooting](README.md) · [8. Diagnostic playbook →](08-diagnostic-playbook.md)
