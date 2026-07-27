# Dọn Disk Docker an toàn

> **Mục tiêu:** Thu hồi dung lượng Docker theo từng object class, bắt đầu từ target có thể nhận diện và tránh xóa Volume hoặc bằng chứng debug ngoài ý muốn.

> **Loại:** How-to · **Điều kiện:** Có quyền chạy Docker CLI; đã xác định máy/daemon đúng; dữ liệu quan trọng có backup<br>
> **Đọc nền:** [Dọn dẹp tài nguyên Docker](../learning-path/02-cli-and-lifecycle/06-don-dep-tai-nguyen.md) · [Docker Volume](../learning-path/03-storage-and-networking/02-docker-volume.md)

[← Giữ dữ liệu khi recreate](preserve-data-when-recreating-container.md) · [Mục lục How-to](README.md)

---

## 1. Vấn đề

Docker chiếm nhiều disk, nhưng một lệnh prune rộng có thể xóa stopped Container cần điều tra, Image cần dùng lại hoặc dữ liệu Volume. Docker không có dry-run chung cho mọi lệnh prune; vì vậy phải inventory trước và dọn từ scope hẹp đến rộng.

## 2. Xác nhận daemon và đo hiện trạng

```bash
docker context show
docker system df --verbose
```

`docker context show` giúp tránh dọn nhầm daemon. `system df --verbose` nhóm Image, Container, local Volume và build cache; `RECLAIMABLE` là ước lượng kỹ thuật, không phải xác nhận dữ liệu vô giá trị.

Lưu inventory trước cleanup:

```bash
docker container ls --all
docker image ls
docker volume ls
docker network ls
```

Nếu đang xử lý sự cố, lưu log/inspect cần thiết trước. Xóa Container sẽ làm mất khả năng truy cập các bằng chứng đó bằng Container ID.

## 3. Xóa target đã xác định

Ưu tiên tên/ID cụ thể:

```bash
docker container stop <container>
docker container rm <container>
docker image rm <image>
```

`container rm` xóa stopped Container và writable layer; `image rm` xóa reference/content khi dependency cho phép. Nếu Docker báo conflict, kiểm tra dependency thay vì thêm `--force` ngay.

> [!WARNING]
> Trước `container rm`, chạy `docker container inspect <container> --format '{{json .Mounts}}'` và bảo toàn dữ liệu không nằm trong mount theo [guide recreate](preserve-data-when-recreating-container.md).

## 4. Dọn stopped Container theo filter

Liệt kê trước:

```bash
docker container ls --all --filter status=exited
```

Sau khi review, có thể giới hạn theo tuổi:

```bash
docker container prune --filter "until=24h"
```

`until=24h` chọn object dừng đủ lâu theo cách command hỗ trợ. Prune xóa cả tập khớp filter; prompt không liệt kê đầy đủ giá trị nghiệp vụ của từng Container.

## 5. Dọn Image

Bắt đầu bằng dangling Image:

```bash
docker image ls --filter dangling=true
docker image prune
```

Chỉ dùng phạm vi rộng hơn sau khi đã review Image và khả năng pull/build lại:

```bash
docker image prune --all
```

> [!WARNING]
> `--all` xóa mọi Image không được Container hiện có tham chiếu, không chỉ Image lỗi. Tag có thể di chuyển, nên pull lại cùng Tag không bảo đảm lấy đúng content cũ nếu bạn không giữ Digest.

## 6. Dọn build cache

```bash
docker builder prune
```

Build cache có thể tái tạo nhưng build sau sẽ chậm hơn và có thể phải tải dependency lại. Chỉ thêm `--all` khi chấp nhận mất cache chưa được Docker xem là dangling theo phạm vi mặc định.

## 7. Chỉ dùng `system prune` sau khi hiểu scope

```bash
docker system prune
```

Lệnh gom nhiều nhóm unused object như stopped Container, network không dùng, dangling Image và build cache. Nó không xóa running Container và không mặc định xóa named Volume.

```bash
docker system prune --all
```

`--all` mở rộng Image scope. `--force` chỉ bỏ prompt xác nhận; nó không tự đồng nghĩa với `--all` hoặc `--volumes`.

> [!WARNING]
> Tránh `docker system prune --all --volumes` khi chưa audit Volume và backup. Volume có thể chứa database hoặc upload duy nhất; recovery không đến từ Image.

## 8. Xác minh kết quả

```bash
docker system df --verbose
docker container ls --all
docker image ls
docker volume ls
```

So sánh với inventory ban đầu:

- Dung lượng mục tiêu đã giảm chưa?
- Container/Image cần giữ còn không?
- Named Volume và ứng dụng quan trọng còn hoạt động không?
- Build hoặc pull tiếp theo có còn nguồn để tái tạo artifact không?

## 9. Recovery

- Image bị xóa: pull theo Digest đã lưu hoặc build lại từ source/Dockerfile.
- Build cache bị xóa: build lại; chấp nhận thời gian dài hơn.
- Container bị xóa: recreate từ Image và configuration đã lưu.
- Writable layer hoặc Volume bị xóa: chỉ khôi phục được từ backup/nguồn dữ liệu khác.

Không có lệnh Docker “undo prune”. Vì vậy recovery phải được chuẩn bị trước destructive command.

## 10. Cleanup định kỳ

Nếu cần tự động hóa, dùng label và filter để xác định tài nguyên demo do workflow sở hữu thay vì chạy prune toàn hệ thống. Luôn log context, inventory, command và kết quả để biết tác vụ đã tác động daemon nào.

## Tài liệu tham khảo

- Docker CLI, [`docker system df`](https://docs.docker.com/reference/cli/docker/system/df/)
- Docker CLI, [`docker system prune`](https://docs.docker.com/reference/cli/docker/system/prune/)
- Docker CLI, [`docker container prune`](https://docs.docker.com/reference/cli/docker/container/prune/)
- Docker CLI, [`docker image prune`](https://docs.docker.com/reference/cli/docker/image/prune/)
- Docker CLI, [`docker builder prune`](https://docs.docker.com/reference/cli/docker/builder/prune/)

[← Giữ dữ liệu khi recreate](preserve-data-when-recreating-container.md) · [Mục lục How-to](README.md)
