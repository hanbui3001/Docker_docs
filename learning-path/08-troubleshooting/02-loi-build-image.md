# 2. Chẩn đoán lỗi build Image

> **Tóm tắt một câu:** Lỗi build thường thuộc một trong bốn scope: Dockerfile parser, build context, build step hoặc artifact/platform.

[← 1. Phương pháp chẩn đoán](01-phuong-phap-chan-doan.md) · [Mục lục Troubleshooting](README.md) · [3. Container không khởi động →](03-container-khong-khoi-dong.md)

---

## 1. Tách bốn pha

| Pha | Symptom thường gặp |
|---|---|
| Parse Dockerfile | unknown instruction, invalid syntax |
| Load context | file excluded, context quá lớn, path ngoài context |
| Execute step | package manager, compiler hoặc network command thất bại |
| Export Image | platform, disk, permission hoặc tag issue |

Chạy build có progress rõ:

```bash
docker build --progress=plain -t example/api:debug .
```

Dấu chấm cuối là build context. `-t` chỉ đặt Image reference cho kết quả; nó không chọn Dockerfile. Dùng `-f path/to/Dockerfile` khi Dockerfile ở vị trí khác, nhưng source của `COPY` vẫn resolve theo context.

## 2. `COPY failed` và scope path

Với:

```dockerfile
COPY build/libs/app.jar app.jar
```

Source `build/libs/app.jar` thuộc build context; destination `app.jar` thuộc filesystem của stage hiện tại và resolve theo `WORKDIR`. Kiểm tra:

1. Artifact có tồn tại trước build không?
2. `.dockerignore` có loại nó không?
3. Context đang là thư mục nào?
4. Với multi-stage, artifact có nằm ở filesystem của stage được chỉ bởi `--from` không?

Không sửa bằng cách đổi context thành một thư mục quá rộng nếu điều đó gửi secret, `.git` hoặc dữ liệu thừa vào builder.

## 3. Cache che hay làm lộ vấn đề?

`--no-cache` buộc thực thi lại build step; nó hữu ích để xác định cache có liên quan nhưng không làm application runtime nhanh hơn.

```bash
docker build --no-cache --progress=plain -t example/api:debug .
```

Nếu build chỉ thành công khi có cache, step có thể phụ thuộc file/network không ổn định. Nếu chỉ thất bại khi không cache, dependency source hoặc command build có thể không tái lập.

## 4. Java artifact không đúng chỗ

Gradle và Maven có output path khác nhau; tên JAR còn phụ thuộc project configuration. Dùng:

```bash
Get-ChildItem build\libs
Get-ChildItem target
```

trong PowerShell, hoặc:

```bash
ls -la build/libs
ls -la target
```

trong Bash. Đừng giả định `app.jar` đã tồn tại chỉ vì Dockerfile dùng tên đó.

## 5. Platform mismatch

Image hoặc binary build cho architecture khác có thể gây `exec format error` lúc chạy. Kiểm tra:

```bash
docker image inspect example/api:debug --format '{{.Os}}/{{.Architecture}}'
docker version --format '{{.Server.Os}}/{{.Server.Arch}}'
```

Multi-platform build cần builder và base Image hỗ trợ platform mục tiêu; build thành công không luôn đồng nghĩa test runtime native đã được thực hiện.

## 6. Tóm tắt

1. Xác định build thất bại ở parse, context, step hay export.
2. Phân tích `COPY` theo source scope và destination scope.
3. Dùng plain progress và no-cache như công cụ chẩn đoán, không phải “fix” mặc định.

## Tài liệu tham khảo

- Docker Docs, [Build context](https://docs.docker.com/build/concepts/context/)
- Docker Docs, [Build cache](https://docs.docker.com/build/cache/)

[← 1. Phương pháp chẩn đoán](01-phuong-phap-chan-doan.md) · [Mục lục Troubleshooting](README.md) · [3. Container không khởi động →](03-container-khong-khoi-dong.md)
