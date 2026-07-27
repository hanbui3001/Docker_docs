# 3. Bind mount

> **Tóm tắt một câu:** Bind mount nối trực tiếp một file hoặc thư mục cụ thể trên host vào Container, vì vậy nó linh hoạt nhưng làm Container phụ thuộc mạnh hơn vào cấu trúc và quyền của host.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

[← 2. Docker Volume](02-docker-volume.md) · [Mục lục Storage & Networking](README.md) · [4. tmpfs và lựa chọn storage →](04-tmpfs-va-lua-chon-storage.md)

---

## 1. Khi nào cần bind mount?

Trong development, developer muốn sửa source code bằng IDE trên host và thấy thay đổi ngay trong Container. Hoặc một service cần đọc file cấu hình đã tồn tại ở vị trí xác định trên máy chạy Docker. Bind mount giải quyết bằng cách ánh xạ trực tiếp host path vào container path.

**Host path** là đường dẫn trong filesystem của máy chạy Docker daemon. **Container path** là vị trí process bên trong Container nhìn thấy. Chúng có thể giống chữ nhưng vẫn thuộc hai filesystem scope khác nhau.

## 2. Cú pháp và cách resolve path

```bash
docker run --mount type=bind,source=/home/duc/project,target=/workspace alpine
```

| Thành phần | Scope | Ý nghĩa |
|---|---|---|
| `type=bind` | Mount parser | Source là đường dẫn host, không phải Volume name. |
| `source=/home/duc/project` | Host của daemon | Thư mục phải tồn tại và được daemon truy cập. |
| `target=/workspace` | Filesystem Container | Vị trí source xuất hiện trong Container. |

Dạng rút gọn:

```bash
docker run -v /home/duc/project:/workspace alpine
```

`source:destination` vẫn đọc từ ngoài vào trong: host source → container destination.

### PowerShell và đường dẫn Windows

```powershell
docker run --mount "type=bind,source=${PWD},target=/workspace" alpine
```

`${PWD}` được PowerShell resolve trước khi Docker CLI nhận argument. Docker không tự hiểu `${PWD}` như một biến nội bộ. Dấu quote giúp chuỗi không bị tách nếu đường dẫn có khoảng trắng.

> [!NOTE]
> Với Docker Desktop, daemon chạy trong môi trường được quản lý và Docker Desktop thực hiện chia sẻ đường dẫn host. Quyền truy cập, ổ đĩa được chia sẻ và khác biệt separator vẫn có thể ảnh hưởng kết quả.

## 3. Trạng thái trước và sau

Trước lệnh, host có `project/app.txt`; Image có thể có hoặc không có `/workspace`. Sau khi mount, process đọc `/workspace/app.txt` thực chất đang đọc file host. Sửa file từ Container có thể sửa file host nếu mount cho phép ghi.

```bash
docker run --rm --mount type=bind,source=/home/duc/project,target=/workspace alpine ls -la /workspace
```

Lệnh chỉ chứng minh mapping. Output phải phản ánh nội dung thật của source trên host.

## 4. Read-only và rủi ro ghi ngược

```bash
docker run --mount type=bind,source=/etc/myapp,target=/config,readonly myapp:1.0
```

`readonly` ngăn process ghi qua mount theo cơ chế mount của runtime. Đây là lựa chọn tốt cho cấu hình chỉ đọc, nhưng không thay thế việc giới hạn quyền và bảo vệ secret.

Bind mount mặc định có thể ghi. Một process chạy với quyền rộng có thể sửa hoặc xóa file host trong phạm vi được mount. Vì vậy không nên mount tùy tiện thư mục nhạy cảm như filesystem root hoặc socket Docker.

## 5. Bind mount khác Volume ở đâu?

| Tiêu chí | Bind mount | Volume |
|---|---|---|
| Source | Path cụ thể trên host | Object do Docker quản lý |
| Tính di động | Phụ thuộc layout host | Ít phụ thuộc path host hơn |
| Development source | Rất phù hợp | Không phải lựa chọn tự nhiên |
| Dữ liệu database | Có thể dùng nhưng phụ thuộc host | Thường là mặc định dễ quản lý hơn |
| Thao tác ngoài Docker | Trực tiếp | Không khuyến khích phụ thuộc vị trí nội bộ |

## 6. Quan niệm dễ gây hiểu nhầm

### “Bind mount copy file vào Container.”

Sai về cơ chế. Nó không tạo bản sao độc lập; nó làm source host xuất hiện tại destination. Sửa một phía có thể quan sát ở phía còn lại.

### “Relative path luôn tính từ Dockerfile.”

Sai scope. Với CLI, shell hoặc Docker CLI resolve path theo ngữ cảnh command. Với Compose, relative host path thường được resolve dựa trên project/Compose file theo quy tắc Compose. Dockerfile không sở hữu runtime mount này.

### “Mount vào `/app` sẽ trộn file host với file Image.”

Thông thường mount che nội dung cũ tại destination. Nếu Image có `/app/app.jar` nhưng host source không có file đó, process không còn thấy file Image qua `/app` trong lúc mount.

## 7. Tự kiểm tra

1. Ai resolve `${PWD}` trong ví dụ PowerShell?
2. Vì sao bind mount source code thuận tiện nhưng giảm tính di động?
3. `readonly` bảo vệ phạm vi nào và không giải quyết vấn đề nào?

## 8. Tóm tắt

Bind mount là liên kết trực tiếp giữa host path và container path. Luôn xác định hai scope, cách shell resolve source, quyền đọc/ghi và nội dung Image bị che tại destination.

## Tài liệu tham khảo

- Docker Docs, [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

[← 2. Docker Volume](02-docker-volume.md) · [Mục lục Storage & Networking](README.md) · [4. tmpfs và lựa chọn storage →](04-tmpfs-va-lua-chon-storage.md)
