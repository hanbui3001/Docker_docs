# 5. Chẩn đoán lỗi storage và permission

> **Tóm tắt một câu:** Khi file “biến mất” hoặc ứng dụng báo permission denied, trước hết phải xác định dữ liệu đang thuộc writable layer, Volume hay bind mount, rồi so path, ownership và user thực thi ở đúng filesystem scope.

> **Thuật ngữ:** **Ownership** là user/group sở hữu file. **UID/GID** là định danh số của user/group mà kernel dùng kiểm tra quyền. **Mount shadowing** là việc mount che nội dung vốn có tại destination trong Image.

[← 4. Lỗi port và network](04-loi-port-va-network.md) · [Mục lục Troubleshooting](README.md) · [6. Lỗi Docker Compose →](06-loi-docker-compose.md)

---

## 1. Symptom: dữ liệu “mất” sau recreate

### Hypothesis

- Dữ liệu chỉ nằm trong writable layer của Container cũ.
- Container mới mount nhầm Volume hoặc không mount Volume.
- Compose project name thay đổi nên tạo Volume có tên khác.
- Mount mới che path chứa dữ liệu trong Image.
- Ứng dụng ghi vào path khác với path được mount.

### Evidence

```bash
docker container inspect <old-or-current-container> --format '{{json .Mounts}}'
docker volume ls
docker volume inspect <volume>
docker inspect <container> --format '{{.Config.Image}}'
```

Đọc từng mount:

| Trường | Câu hỏi |
|---|---|
| `Type` | `volume`, `bind` hay `tmpfs`? |
| `Source` | Volume name/path thật nào được dùng? |
| `Destination` | App có ghi đúng path này không? |
| `RW` | Mount có cho ghi không? |

Với Compose, kiểm tra tên Volume đã resolve:

```bash
docker compose config
docker compose config --volumes
```

### Correction

- Mount lại đúng named Volume vào đúng application data directory.
- Giữ project name ổn định hoặc khai báo Volume name/external có chủ đích.
- Chuyển dữ liệu từ writable layer cũ trước khi remove Container nếu object còn tồn tại.
- Sửa app data path thay vì mount một thư mục “gần đúng”.

### Verification

Ghi một record thử qua ứng dụng, recreate Container có kiểm soát, rồi đọc lại record. Chỉ nhìn thấy Volume trong `docker volume ls` chưa chứng minh app đang dùng nó.

## 2. Symptom: `Permission denied`

### Hypothesis

- Process chạy bằng UID/GID không có quyền trên source/destination.
- Bind mount mang ownership của host vào Container.
- Volume cũ được khởi tạo bởi user khác.
- Mount là read-only.
- Host security policy như SELinux chặn access.

### Evidence

```bash
docker exec <container> id
docker exec <container> sh -c 'ls -ldn /data; ls -lan /data | head'
docker inspect <container> --format '{{json .Mounts}}'
docker inspect <container> --format 'user={{json .Config.User}} readonlyRootfs={{.HostConfig.ReadonlyRootfs}}'
```

`ls -n` hiển thị UID/GID số, tránh bị tên user khác nhau giữa host và Container làm nhiễu. So quyền trên từng thư mục cha; có quyền file nhưng không có execute/traverse trên directory cha vẫn thất bại.

### Correction

- Chọn owner/group phù hợp với UID/GID process.
- Khởi tạo permission trong entrypoint hoặc init step có phạm vi hẹp.
- Với bind mount development, đồng bộ UID/GID hoặc cấp group access hợp lý.
- Bỏ read-only chỉ khi workload thực sự cần ghi; ưu tiên mount riêng path cần ghi.

> [!WARNING]
> Không dùng `chmod -R 777`, chạy Container bằng root hoặc `--privileged` như fix mặc định. Các cách này mở rộng quyền quá mức, có thể che sai ownership và tạo rủi ro trên host-mounted data.

### Verification

Chạy thao tác đọc/ghi bằng đúng user application:

```bash
docker exec --user <app-uid> <container> sh -c 'touch /data/.permission-check && rm /data/.permission-check'
```

Sau đó kiểm tra ứng dụng thực hiện operation thật, vì `touch` không kiểm chứng database locking, rename hoặc fsync.

## 3. Symptom: file trong Image không còn thấy

### Hypothesis

Mount đang che nội dung tại destination.

### Evidence

So sánh Image không mount với Container có mount:

```bash
docker run --rm <image> ls -la /app
docker exec <container> ls -la /app
docker inspect <container> --format '{{json .Mounts}}'
```

Nếu Image có `/app/app.jar` nhưng bind mount source rỗng được gắn vào `/app`, Container chỉ thấy nội dung source mount.

### Correction

Mount vào subdirectory hẹp hơn, bổ sung file cần thiết ở source, hoặc tách binary bất biến khỏi directory runtime data.

### Verification

Inspect mount destination và xác nhận cả artifact lẫn data path tồn tại đúng scope sau recreate.

## 4. Cleanup và khôi phục an toàn

> [!CAUTION]
> `docker volume rm`, `docker compose down --volumes` và `docker system prune --volumes` có thể xóa dữ liệu không tái tạo được. Trước khi chạy, inspect Volume, xác định Container/project sử dụng, đánh giá backup và ghi lại tên chính xác.

Không xóa Volume để “tạo lại permission sạch” khi chưa sao lưu. Nếu cần điều tra dữ liệu, mount Volume read-only vào một Container công cụ phù hợp.

## 5. Quan niệm dễ sai

### “Cùng username thì chắc cùng quyền.”

Kernel chủ yếu so UID/GID số. User `app` trong Image có thể mang UID khác user `app` trên host.

### “Volume có dữ liệu nên Container mới tự thấy.”

Volume phải được mount đúng object và destination. Sự tồn tại của storage không tạo attachment tự động.

## 6. Tóm tắt

1. Xác định storage type, source, destination và read/write mode.
2. So UID/GID và permission bằng số ở đúng path.
3. Phân biệt dữ liệu bị xóa với dữ liệu bị mount che hoặc mount nhầm.
4. Không xóa Volume hay mở quyền toàn cục trước khi có backup và bằng chứng.

## Tài liệu tham khảo

- Docker Docs, [Volumes](https://docs.docker.com/engine/storage/volumes/)
- Docker Docs, [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

[← 4. Lỗi port và network](04-loi-port-va-network.md) · [Mục lục Troubleshooting](README.md) · [6. Lỗi Docker Compose →](06-loi-docker-compose.md)
