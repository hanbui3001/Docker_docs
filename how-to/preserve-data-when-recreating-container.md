# Giữ dữ liệu khi Recreate Container

> **Mục tiêu:** Tạo Container mới với Image hoặc configuration mới mà vẫn giữ dữ liệu đang nằm trong Volume, bind mount hoặc writable layer cần cứu.

> **Loại:** How-to · **Điều kiện:** Biết Container cũ, Image mới và đường dẫn dữ liệu của ứng dụng; có đủ dung lượng backup<br>
> **Đọc nền:** [Dữ liệu trong Container](../learning-path/03-storage-and-networking/01-du-lieu-trong-container.md) · [Docker Volume](../learning-path/03-storage-and-networking/02-docker-volume.md) · [Bind mount](../learning-path/03-storage-and-networking/03-bind-mount.md)

[← Chẩn đoán published port](diagnose-published-port.md) · [Mục lục How-to](README.md) · [Dọn disk an toàn →](clean-docker-disk-safely.md)

---

## 1. Vấn đề

`docker container run` luôn tạo Container mới. Nếu dữ liệu chỉ nằm trong writable layer của Container cũ, remove sẽ xóa dữ liệu đó. Nếu dữ liệu nằm trong named Volume hoặc bind mount, bạn phải gắn đúng source vào đúng destination khi tạo replacement.

> [!WARNING]
> Không chạy `docker container rm --volumes`, `docker volume rm` hoặc prune Volume trước khi hoàn tất backup và xác minh Container mới đọc được dữ liệu.

## 2. Kiểm tra nơi dữ liệu đang được lưu

```bash
docker container inspect <container> --format '{{json .Mounts}}'
```

Đọc từng mount:

| Trường | Ý nghĩa |
|---|---|
| `Type=volume` | Dữ liệu thuộc Docker Volume; `Name`/`Source` nhận diện nguồn. |
| `Type=bind` | `Source` là path thật trên host. |
| `Destination` | Path mà ứng dụng nhìn thấy trong Container. |
| `RW=true/false` | Mount được ghi hay read-only. |

Nếu path dữ liệu không xuất hiện trong `.Mounts`, dữ liệu có thể đang ở writable layer. Xác nhận đúng path theo tài liệu ứng dụng trước khi tiếp tục.

Ghi lại Image, command và environment cần tái tạo:

```bash
docker container inspect <container> --format 'Image={{.Config.Image}} Entrypoint={{json .Config.Entrypoint}} Cmd={{json .Config.Cmd}}'
docker container inspect <container> --format '{{json .Config.Env}}'
```

Không chia sẻ output environment nếu có secret.

## 3. Dừng ghi và tạo backup

Dừng Container để tránh dữ liệu thay đổi trong lúc sao chép:

```bash
docker container stop --time 30 <container>
```

Với database, ưu tiên công cụ backup nhất quán của database trước khi stop nếu quy trình vận hành yêu cầu. Copy trực tiếp file database không luôn tạo bản backup hợp lệ.

### Trường hợp A: named Volume

Xác nhận Volume tồn tại:

```bash
docker volume inspect <volume>
```

Named Volume sống độc lập với Container. Không cần copy chỉ để recreate, nhưng vẫn nên có backup riêng đối với dữ liệu quan trọng.

### Trường hợp B: bind mount

Backup thư mục `Source` bằng công cụ host phù hợp. Docker chỉ mount path đó; việc backup và quyền file thuộc trách nhiệm hệ điều hành host.

### Trường hợp C: writable layer

Copy dữ liệu ra host trước khi remove:

```bash
docker container cp <container>:/path/to/data ./container-data-backup
```

`<container>:/path/to/data` là source trong Container cũ; `./container-data-backup` là destination host. Kiểm tra file backup có tồn tại và có kích thước/nội dung hợp lý trước khi tiếp tục.

## 4. Giữ Container cũ làm rollback

Đổi tên thay vì xóa ngay:

```bash
docker container rename <container> <container>-old
```

Rename giải phóng tên cho replacement nhưng giữ nguyên object, writable layer và metadata. Container cũ vẫn đang dừng.

## 5. Tạo replacement với storage đúng

### Dùng named Volume

```bash
docker container run --name <container> --detach --mount type=volume,source=<volume>,destination=/path/to/data <new-image>
```

| Token | Giá trị phải giữ đúng |
|---|---|
| `source=<volume>` | Tên Volume cũ, không phải tên mới ngẫu nhiên. |
| `destination=/path/to/data` | Path ứng dụng mới dùng để đọc/ghi dữ liệu. |
| `<new-image>` | Image replacement đã chọn. |

### Dùng bind mount

```bash
docker container run --name <container> --detach --mount type=bind,source=/absolute/host/path,destination=/path/to/data <new-image>
```

`source` phải resolve thành path host cũ. Trên PowerShell, quote chuỗi mount nếu path có ký tự đặc biệt hoặc khoảng trắng:

```powershell
docker container run --name <container> --detach --mount "type=bind,source=C:\data\myapp,destination=/path/to/data" <new-image>
```

### Khôi phục dữ liệu từ writable-layer backup

Tạo replacement với một Volume mới, rồi copy backup vào path đã mount:

```bash
docker volume create <new-volume>
docker container run --name <container> --detach --mount type=volume,source=<new-volume>,destination=/path/to/data <new-image>
docker container cp ./container-data-backup/. <container>:/path/to/data/
```

Việc copy có thể tạo owner/permission không phù hợp với user của ứng dụng; kiểm tra log và quyền thay vì tự động chạy Container bằng root.

## 6. Xác minh trước khi xóa Container cũ

```bash
docker container ls --all --filter name=<container>
docker container inspect <container> --format '{{json .Mounts}}'
docker container logs --tail 100 <container>
```

Xác minh thêm bằng chức năng ứng dụng: đọc record, upload test hoặc chạy health endpoint. Kiểm tra `.Mounts` phải cho thấy đúng source và destination; Container `running` một mình chưa chứng minh dữ liệu đúng.

## 7. Rollback

Nếu replacement lỗi:

```bash
docker container stop <container>
docker container rm <container>
docker container rename <container>-old <container>
docker container start <container>
```

Rollback chỉ an toàn khi schema hoặc dữ liệu chưa bị Image mới migrate theo cách không tương thích. Với database migration, cần kế hoạch rollback dữ liệu riêng.

## 8. Cleanup sau thời gian theo dõi

Khi replacement đã ổn định và backup đã được xác minh:

```bash
docker container rm <container>-old
```

Lệnh này xóa Container cũ, không mặc định xóa named Volume đã gắn. Chỉ xóa Volume cũ khi đã xác nhận nó không còn được dùng và dữ liệu có recovery phù hợp.

## Tài liệu tham khảo

- Docker Docs, [Volumes](https://docs.docker.com/engine/storage/volumes/)
- Docker Docs, [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- Docker CLI, [`docker container cp`](https://docs.docker.com/reference/cli/docker/container/cp/)

[← Chẩn đoán published port](diagnose-published-port.md) · [Mục lục How-to](README.md) · [Dọn disk an toàn →](clean-docker-disk-safely.md)
