# `docker build` options — Tra cứu nhanh

> **Loại:** Reference · Dùng cho Docker Buildx/BuildKit hiện đại. Kiểm tra `docker buildx build --help` trên phiên bản cục bộ nếu option thay đổi.

[← Dockerfile instructions](instructions.md) · [Dockerfile Reference](README.md)

---

## 1. Grammar

```text
docker build [OPTIONS] PATH | URL | -
docker buildx build [OPTIONS] PATH | URL | -
```

Trong Docker CLI hiện đại, `docker build` thường dùng BuildKit/Buildx backend. Positional argument cuối là context.

## 2. Bảng option thường dùng

| Option | Dạng | Mục đích |
|---|---|---|
| `--tag`, `-t` | `-t name:tag` | Đặt Image reference output |
| `--file`, `-f` | `-f path` | Chọn Dockerfile |
| `--build-arg` | `--build-arg K=V` | Truyền giá trị cho `ARG` |
| `--target` | `--target stage` | Dừng/output tại named stage |
| `--platform` | `--platform linux/amd64` | Chọn target platform |
| `--pull` | flag | Kiểm tra/pull base mới theo reference |
| `--no-cache` | flag | Không tái sử dụng Dockerfile step cache |
| `--progress` | `plain|tty|rawjson|quiet` | Định dạng tiến trình build |
| `--secret` | `id=...,src=...` | Cấp secret mount cho build |
| `--ssh` | `default|id=...` | Forward SSH agent/socket cho build step |
| `--cache-from` | backend spec | Nguồn import cache |
| `--cache-to` | backend spec | Nơi export cache |
| `--output`, `-o` | output spec | Chọn loại output |
| `--load` | flag | Load single-platform result vào local image store |
| `--push` | flag | Push result lên registry |
| `--label` | `key=value` | Thêm label vào output Image |
| `--metadata-file` | path | Ghi metadata build vào file |

## 3. Context và Dockerfile path

```bash
docker build --file docker/Dockerfile.prod --tag my-app:1.0 .
```

| Giá trị | Resolve |
|---|---|
| `docker/Dockerfile.prod` | Dockerfile path được CLI/builder chọn |
| `.` | Context root là current terminal directory |
| `COPY src/ /app/src/` | Source resolve từ `<context>/src`, không từ thư mục `docker/` |

## 4. Tag output

```bash
docker build --tag my-app:1.0 --tag registry.example/my-app:1.0 .
```

Có thể gắn nhiều Tag cho cùng build result. Tag là reference, không phải bằng chứng nội dung immutable; dùng Digest/provenance khi cần xác nhận artifact.

## 5. Build argument

Dockerfile:

```dockerfile
ARG APP_VERSION=dev
LABEL org.opencontainers.image.version=$APP_VERSION
```

Build:

```bash
docker build --build-arg APP_VERSION=1.2.0 --tag my-app:1.2.0 .
```

Nếu viết `--build-arg APP_VERSION` không có `=`, CLI có thể lấy giá trị environment cùng tên từ client environment. Điều này tiện nhưng làm đầu vào kém rõ ràng; không dùng cho secret.

## 6. Target stage

```bash
docker build --target build --tag my-app:build-debug .
```

Output là stage `build`, không phải final stage. Dùng để debug/test artifact; Image có thể chứa source và tool build.

## 7. Cache và base refresh

```bash
docker build --pull --no-cache --progress=plain --tag my-app:rebuild .
```

| Option | Tác động |
|---|---|
| `--pull` | Cố lấy base Image mới hơn theo reference. |
| `--no-cache` | Không reuse cache của Dockerfile steps. |

Hai option độc lập. Chúng làm build “mới” hơn nhưng không bảo đảm dependency bên trong package manager đã thay đổi nếu version/repository vẫn resolve như cũ.

## 8. Build secret

Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=repo_token \
    TOKEN="$(cat /run/secrets/repo_token)" \
    && ./download-private-artifact "$TOKEN"
```

CLI:

```bash
docker build --secret id=repo_token,src=repo-token.txt --tag my-app:1.0 .
```

| Field | Nghĩa |
|---|---|
| `id=repo_token` | Tên nối CLI secret với Dockerfile mount |
| `src=repo-token.txt` | File trên client/host được cấp cho build |
| `/run/secrets/repo_token` | Default mount path trong build step |

Không `COPY` secret file và không ghi nội dung secret vào output filesystem/log.

## 9. Platform

```bash
docker buildx build --platform linux/amd64,linux/arm64 --tag registry.example/my-app:1.0 --push .
```

- Danh sách sau `--platform` là target output platform.
- Multi-platform output thường phải push registry hoặc dùng output phù hợp; local Docker image store có thể không load multi-platform result đầy đủ tùy cấu hình.
- Build thành công không đảm bảo native dependency chạy đúng trên mọi architecture; cần test từng platform.

## 10. `--load`, `--push` và `--output`

```bash
docker buildx build --load --tag my-app:local .
docker buildx build --push --tag registry.example/my-app:1.0 .
docker buildx build --output type=local,dest=out .
```

| Option | Output đi đâu? |
|---|---|
| `--load` | Local image store, thường single platform |
| `--push` | Registry |
| `--output type=local` | Filesystem directory, phù hợp stage xuất artifact |

## 11. PowerShell và Bash multiline

Bash:

```bash
docker buildx build \
  --platform linux/amd64 \
  --tag my-app:1.0 \
  .
```

PowerShell:

```powershell
docker buildx build `
  --platform linux/amd64 `
  --tag my-app:1.0 `
  .
```

> [!WARNING]
> Trong PowerShell, backtick phải là ký tự cuối dòng. Khoảng trắng sau backtick làm hỏng line continuation. Lệnh một dòng an toàn hơn khi copy giữa tài liệu và terminal.

## 12. Kiểm chứng output

```bash
docker image inspect my-app:1.0
docker image history my-app:1.0
```

Với push:

```bash
docker buildx imagetools inspect registry.example/my-app:1.0
```

Kiểm tra reference, platform manifest, Image configuration và history phù hợp mục tiêu; không chỉ dựa vào exit code build.

## Tài liệu chính thức

- Docker CLI, [`docker buildx build`](https://docs.docker.com/reference/cli/docker/buildx/build/)
- Docker Docs, [Build secrets](https://docs.docker.com/build/building/secrets/)
- Docker Docs, [Multi-platform builds](https://docs.docker.com/build/building/multi-platform/)

[← Dockerfile instructions](instructions.md) · [Dockerfile Reference](README.md)
