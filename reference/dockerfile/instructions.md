# Dockerfile instructions — Tra cứu nhanh

> **Loại:** Reference · Cú pháp ngắn gọn, scope, default và điểm dễ nhầm. Phần giải thích theo luồng nằm tại [3. Các instruction cơ bản](../../learning-path/04-dockerfile/03-cac-instruction-co-ban.md).

[Dockerfile Reference](README.md) · [`docker build` options →](docker-build-options.md)

---

## 1. Bảng nhanh

| Instruction | Tác động chính | Thời điểm |
|---|---|---|
| `FROM` | Mở stage từ base Image | Build |
| `RUN` | Chạy process và lưu filesystem change | Build |
| `COPY` | Copy từ context/stage/Image vào stage hiện tại | Build |
| `ADD` | Copy với hành vi bổ sung | Build |
| `ARG` | Khai báo build variable | Build |
| `ENV` | Đặt environment default | Build + runtime config |
| `WORKDIR` | Đặt working directory | Build + runtime config |
| `USER` | Đặt user/group mặc định | Build + runtime config |
| `EXPOSE` | Ghi metadata port | Runtime config |
| `HEALTHCHECK` | Ghi command kiểm tra health | Runtime config |
| `ENTRYPOINT` | Đặt executable chính | Runtime config |
| `CMD` | Đặt command hoặc argument mặc định | Runtime config |
| `LABEL` | Ghi metadata key-value | Image config |
| `SHELL` | Đổi shell mặc định cho shell-form instruction | Build/runtime config tùy instruction |

## 2. `FROM`

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>|@<digest>] [AS <name>]
```

```dockerfile
FROM eclipse-temurin:21-jdk AS build
```

- Mỗi `FROM` mở stage mới.
- `AS build` tạo tên dùng bởi `COPY --from=build` hoặc `--target build`.
- `ARG` dùng trong `FROM` phải khai báo trước `FROM`.

## 3. `RUN`

```dockerfile
RUN <command>                    # shell form
RUN ["executable", "arg1"]      # exec form
```

```dockerfile
RUN apt-get update && apt-get install -y curl
```

- Chạy tại build time trong stage hiện tại.
- Shell form dùng shell mặc định, thường `/bin/sh -c` trên Linux.
- Dùng `&&` để step dừng khi command trước thất bại và để dữ liệu cùng vòng đời nằm trong một step.

## 4. `COPY`

```dockerfile
COPY [OPTIONS] <src>... <dest>
COPY --from=<stage|image|context> <src>... <dest>
```

```dockerfile
WORKDIR /app
COPY app.jar app.jar
```

| Thành phần | Scope |
|---|---|
| Source `app.jar` | `<context-root>/app.jar` |
| Destination `app.jar` | `/app/app.jar` trong stage hiện tại |

```dockerfile
COPY --from=build /workspace/app.jar app.jar
```

Source chuyển sang filesystem stage `build`; destination vẫn thuộc stage hiện tại. Options thường dùng:

- `--from=<name>`: chọn source khác context mặc định.
- `--chown=<user>:<group>`: đặt ownership trên Linux.
- `--chmod=<mode>`: đặt permission khi frontend hỗ trợ.
- `--link`: hành vi layer/link nâng cao; kiểm tra compatibility trước khi dùng.

## 5. `ADD`

```dockerfile
ADD [OPTIONS] <src>... <dest>
```

- Có thể tự giải nén local tar archive và hỗ trợ source bổ sung theo frontend.
- Dùng `COPY` cho copy thông thường để ý định hẹp và dễ dự đoán.

## 6. `ARG`

```dockerfile
ARG <name>[=<default>]
```

```dockerfile
ARG APP_VERSION=dev
```

- Truyền bằng `docker build --build-arg APP_VERSION=1.0 .`.
- Scope bắt đầu từ dòng khai báo trong stage.
- Không tự tồn tại trong Container environment.
- Không dùng cho secret.

## 7. `ENV`

```dockerfile
ENV <key>=<value> [<key>=<value>...]
```

```dockerfile
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75"
```

- Ghi default vào Image configuration và có trong process environment.
- Có thể override lúc run bằng `--env`/Compose.
- Không lưu credential cố định trong Image.

## 8. `WORKDIR`

```dockerfile
WORKDIR /path
```

- Tạo directory nếu thiếu.
- Path tương đối nối với `WORKDIR` hiện tại.
- Ảnh hưởng `RUN`, `COPY`, `ADD`, `CMD`, `ENTRYPOINT` và runtime working directory.

## 9. `USER`

```dockerfile
USER <user>[:<group>]
```

```dockerfile
USER 10001:10001
```

- Ảnh hưởng `RUN` phía sau và process runtime.
- Cần chuẩn bị ownership/permission trước khi chuyển sang non-root.

## 10. `EXPOSE`

```dockerfile
EXPOSE <port>[/<protocol>]
```

```dockerfile
EXPOSE 8080
EXPOSE 53/udp
```

- Chỉ metadata; không publish host port.
- Publish bằng `docker run -p HOST:CONTAINER` hoặc Compose `ports`.

## 11. `HEALTHCHECK`

```dockerfile
HEALTHCHECK [OPTIONS] CMD <command>
HEALTHCHECK NONE
```

Options:

| Option | Ý nghĩa |
|---|---|
| `--interval=<duration>` | Khoảng giữa các lần kiểm tra |
| `--timeout=<duration>` | Timeout mỗi lần |
| `--start-period=<duration>` | Khoảng khởi động chưa tính thất bại như bình thường |
| `--start-interval=<duration>` | Nhịp kiểm tra trong start period khi được hỗ trợ |
| `--retries=<n>` | Thất bại liên tiếp trước `unhealthy` |

Command chạy bên trong Container; mọi executable được gọi phải tồn tại trong Image.

## 12. `ENTRYPOINT` và `CMD`

```dockerfile
ENTRYPOINT ["executable", "arg"]
CMD ["default-arg"]
```

Kết hợp:

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
CMD ["--spring.profiles.active=prod"]
```

- Argument sau Image trong `docker run IMAGE ...` thay `CMD`.
- `--entrypoint` thay `ENTRYPOINT`.
- Nếu chỉ có `CMD ["java", "-jar", "app.jar"]`, toàn bộ command dễ được override.
- Ưu tiên exec form cho process chính và signal handling rõ.

## 13. `LABEL`

```dockerfile
LABEL <key>=<value> [<key>=<value>...]
```

```dockerfile
LABEL org.opencontainers.image.title="sample-api"
```

Đọc bằng:

```bash
docker image inspect --format '{{json .Config.Labels}}' sample-api:1.0
```

## 14. `SHELL`

```dockerfile
SHELL ["executable", "parameters"]
```

Windows example:

```dockerfile
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';"]
```

`SHELL` đổi shell mặc định cho shell-form `RUN`, `CMD`, `ENTRYPOINT`. Exec form đã nêu executable trực tiếp nên không cần shell mặc định.

## 15. So sánh nhanh

| Cặp | Khác biệt cốt lõi |
|---|---|
| `RUN` / `CMD` | Build-time execution / runtime default |
| `CMD` / `ENTRYPOINT` | Default dễ override / executable cốt lõi |
| `COPY` / `ADD` | Copy rõ ràng / thêm auto-extract và source behavior |
| `ARG` / `ENV` | Build scope / Image + runtime environment |
| `EXPOSE` / `-p` | Metadata / host-port mapping thật |

## Tài liệu chính thức

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

[Dockerfile Reference](README.md) · [`docker build` options →](docker-build-options.md)
