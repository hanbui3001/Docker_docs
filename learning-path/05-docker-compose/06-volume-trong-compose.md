# 6. Volume trong Compose

> **Tóm tắt một câu:** Service-level `volumes` mô tả mount vào Container, còn top-level `volumes` khai báo named Volume resource; source và target giống nhau về hình thức nhưng thuộc hai filesystem và hai lifecycle khác nhau.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 40 phút  
> **Nguồn chính:** [Compose volumes](https://docs.docker.com/reference/compose-file/volumes/) · [Service volumes](https://docs.docker.com/reference/compose-file/services/#volumes)

[← 5. Environment và interpolation](05-environment-va-interpolation.md) · [Mục lục Part 05](README.md) · [7. Healthcheck và dependency →](07-healthcheck-va-dependency.md)

---

## 1. Vấn đề: Container có thể bị thay thế

Compose thường recreate Container khi Image hoặc cấu hình thay đổi. File chỉ nằm trong writable layer của Container sẽ đi theo lifecycle của Container và mất khi Container bị remove.

Database cần dữ liệu sống lâu hơn một instance. Mount tách vị trí lưu trữ khỏi writable layer:

```text
Named Volume database-data
        │ mount
        ▼
Container path /var/lib/mysql
```

## 2. Hai key `volumes`, hai scope

```yaml
services:
  database:
    image: mysql:8.4
    volumes:
      - database-data:/var/lib/mysql

volumes:
  database-data:
```

| Đường dẫn | Vai trò |
|---|---|
| `services.database.volumes[0]` | Mount Volume vào Container của Service. |
| `volumes.database-data` | Khai báo logical named Volume trong project model. |

Top-level declaration không tự mount. Service-level entry không tự giải thích mọi thuộc tính resource; nó chọn source, target và mount options.

## 3. Phân tích short syntax

Dạng tổng quát:

```text
SOURCE:TARGET[:MODE]
```

Ví dụ:

```yaml
volumes:
  - database-data:/var/lib/mysql
```

| Token | Scope | Giá trị đã resolve |
|---|---|---|
| `database-data` | Compose project | Logical named Volume source; runtime name thường có project prefix. |
| `:` thứ nhất | Parser Compose | Phân cách source và target. |
| `/var/lib/mysql` | Container filesystem | Absolute target path nhìn thấy trong Container. |
| `MODE` | Mount option | Bị bỏ qua nên dùng mặc định đọc-ghi. |

Hai phía không phải hai file giống nhau:

- Source là Docker-managed Volume object trên host/runtime.
- Target là vị trí mount trong filesystem view của Container.

Trước khi mount, Image có thể đã có nội dung tại `/var/lib/mysql`. Sau mount, nội dung tại target bị mount che khuất trong góc nhìn Container và đường dẫn trỏ vào Volume. “Che khuất” không nhất thiết xóa file trong Image; bỏ mount có thể làm nội dung Image hiện lại.

## 4. Named Volume, bind mount và anonymous Volume

### Named Volume

```yaml
- database-data:/var/lib/mysql
```

Docker quản lý storage object; tên logical được khai báo top-level. Phù hợp dữ liệu ứng dụng cần persistence mà không cần người dùng chỉnh trực tiếp bằng host editor.

### Bind mount

```yaml
- ./config:/app/config:ro
```

Source là host path `./config`; target là `/app/config`; mode `ro` đặt read-only từ phía Container. Bind mount gắn chặt với cấu trúc filesystem host và hữu ích khi development cần sửa file trực tiếp.

### Anonymous Volume

```yaml
- /var/lib/mysql
```

Chỉ có target; Docker tạo Volume không có logical name dễ quản lý trong Compose file. Nó có thể tồn tại ngoài ý muốn và khó nhận diện hơn, nên named Volume thường rõ ràng hơn cho dữ liệu cần giữ.

## 5. Long syntax

```yaml
services:
  database:
    volumes:
      - type: volume
        source: database-data
        target: /var/lib/mysql
        read_only: false

volumes:
  database-data:
```

| Key | Ý nghĩa |
|---|---|
| `type` | Loại mount như `volume`, `bind`, `tmpfs`. |
| `source` | Named Volume hoặc host path tùy type. |
| `target` | Absolute path trong Container. |
| `read_only` | Có cho phép Container ghi qua mount không. |

Long syntax hữu ích khi cần option riêng của bind/volume, kiểm soát rõ type hoặc tránh compact string khó đọc.

## 6. Runtime name và project name

Với project `shop`:

```yaml
volumes:
  database-data:
```

Runtime Volume thường gần dạng `shop_database-data`. Logical name trong Compose vẫn là `database-data`.

Muốn ép runtime name:

```yaml
volumes:
  database-data:
    name: shared-database-data
```

`name` không bị project prefix theo cùng cách; điều này có thể chủ động chia sẻ nhưng cũng tăng nguy cơ hai project đụng cùng dữ liệu.

## 7. External Volume

```yaml
volumes:
  database-data:
    external: true
    name: production-database-data
```

**External resource** — resource tồn tại ngoài lifecycle tạo/xóa thông thường của Compose project. Compose mong Volume đã tồn tại và báo lỗi nếu không tìm thấy; nó không tự tạo resource external.

External không có nghĩa dữ liệu được backup, immutable hay an toàn. Nó chỉ nói ownership/lifecycle nằm ngoài project này.

## 8. `down` và dữ liệu

```bash
docker compose down
```

Mặc định xóa project Container và Network do Compose quản lý, nhưng named Volume không bị xóa chỉ vì `down` thông thường.

> [!WARNING]
> `docker compose down --volumes` xóa named Volume được khai báo trong project và anonymous Volume gắn với Container theo phạm vi command. Dữ liệu database trong các Volume đó có thể mất không thể hoàn tác nếu không có backup.

Trước khi xóa:

```bash
docker compose volumes
docker volume ls
```

Lệnh đầu liệt kê Volume của Compose project nếu phiên bản CLI hỗ trợ; lệnh sau quan sát Volume toàn Docker host.

## 9. MySQL initialization và Volume cũ

Các Image database thường chạy logic khởi tạo chỉ khi data directory trống. Ví dụ `MYSQL_ROOT_PASSWORD` có thể được dùng khi Volume mới chưa có database.

```text
Lần đầu: empty Volume + env init -> database files được tạo
Lần sau: existing Volume + env mới -> Image phát hiện data đã tồn tại
```

Đổi environment trong Compose không nhất thiết thay credential đã được lưu bên trong database. Đây không phải Compose “cache sai”; đó là lifecycle của ứng dụng database và persistent data.

## 10. Cách quan sát mount

```bash
docker compose ps -q database
docker container inspect <container-id>
docker volume inspect <volume-name>
```

Trong PowerShell có thể lấy ID:

```powershell
$containerId = docker compose ps -q database
docker container inspect $containerId
```

Bash:

```bash
container_id=$(docker compose ps -q database)
docker container inspect "$container_id"
```

Quan sát nhánh `Mounts`: `Type`, `Source`, `Destination`, `RW`. Không chỉnh trực tiếp thư mục nội bộ Docker dùng để quản lý Volume.

## 11. Quan niệm dễ gây hiểu nhầm

### 11.1 “Top-level `volumes` đã gắn dữ liệu vào database.”

Sai: nó chỉ khai báo resource; Service còn cần mount entry.

### 11.2 “Xóa Container sẽ xóa named Volume.”

Sai trong lifecycle thông thường: hai object tách biệt. Nhưng lệnh có option xóa Volume vẫn có thể phá dữ liệu.

### 11.3 “Đổi password environment sẽ cập nhật database cũ.”

Sai với nhiều Image database: init variable thường chỉ tác động data directory trống.

### 11.4 “Bind mount và named Volume giống nhau vì đều có `source:target`.”

Sai: source owner, portability, cách quản lý và use case khác nhau.

## 12. Tóm tắt

1. Service-level `volumes` tạo mount; top-level `volumes` khai báo named resource.
2. Short syntax là `SOURCE:TARGET[:MODE]`.
3. Named Volume do Docker quản lý; bind mount dùng host path; anonymous Volume thiếu logical name rõ ràng.
4. Mount che nội dung tại target trong góc nhìn Container.
5. `down --volumes` có thể phá dữ liệu; environment mới không tự sửa state đã lưu trong Volume.

## Tài liệu tham khảo

- Docker Docs, [Volumes top-level element](https://docs.docker.com/reference/compose-file/volumes/)
- Docker Docs, [Service volumes](https://docs.docker.com/reference/compose-file/services/#volumes)
- Docker Docs, [Persisting container data](https://docs.docker.com/get-started/docker-concepts/running-containers/persisting-container-data/)

[← 5. Environment và interpolation](05-environment-va-interpolation.md) · [Mục lục Part 05](README.md) · [7. Healthcheck và dependency →](07-healthcheck-va-dependency.md)
