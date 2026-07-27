# Docker Compose commands

> **Loại:** Reference · **Cấp độ:** Beginner đến Intermediate  
> **Mục đích:** Chọn command theo mục tiêu, xác định option thuộc parser nào và dự đoán state transition.

[Mục lục Compose Reference](README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)

---

## 1. Command grammar

```text
docker compose [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [SERVICE...]
```

```bash
docker compose -f compose.dev.yaml -p shop up -d --build backend
```

| Phần | Ví dụ | Scope |
|---|---|---|
| Global option | `-f compose.dev.yaml` | Chọn Compose file. |
| Global option | `-p shop` | Chọn project name. |
| Command | `up` | Hành động chính. |
| Command option | `-d`, `--build` | Chỉ áp dụng cho `up`. |
| Service argument | `backend` | Logical Service target. |

## 2. Bảng chọn nhanh

| Mục tiêu | Command | Container có bị xóa/tạo mới? |
|---|---|---|
| Kiểm tra model | `config` | Không. |
| Tạo/reconcile và chạy project | `up` | Có thể create/recreate. |
| Dừng nhưng giữ Container | `stop` | Không xóa. |
| Start Container đã dừng | `start` | Không tạo mới. |
| Dừng rồi start cùng Container | `restart` | Không áp dụng đầy đủ config mới. |
| Xóa project Containers/Networks | `down` | Có, xóa. |
| Xóa stopped Service Containers | `rm` | Có, xóa target. |
| Xem trạng thái | `ps` | Không. |
| Xem log | `logs` | Không. |
| Chạy process trong Container hiện có | `exec` | Không tạo Container mới. |
| Chạy one-off Container | `run` | Tạo Container mới. |
| Build Image | `build` | Không start Container. |
| Pull Image | `pull` | Không recreate Container hiện có. |
| Liệt kê Image project | `images` | Không. |

## 3. Global options thường dùng

| Option | Ý nghĩa |
|---|---|
| `-f`, `--file FILE` | Chọn Compose file; có thể lặp để merge nhiều file. |
| `-p`, `--project-name NAME` | Đặt project name. |
| `--env-file FILE` | Chọn nguồn environment cho interpolation/Compose CLI theo command context. |
| `--profile NAME` | Bật profile; có thể lặp. |
| `--project-directory DIR` | Chọn project directory dùng cho discovery/path rules. |
| `--parallel N` | Giới hạn song song cho operation được hỗ trợ. |
| `--dry-run` | Hiển thị plan operation ở command được hỗ trợ mà không áp dụng. |

Global option đứng sau `docker compose` và trước command.

## 4. `config`

```text
docker compose config [OPTIONS] [SERVICE...]
```

```bash
docker compose config
docker compose config --services
docker compose config --images
docker compose config --environment
```

| Option | Output |
|---|---|
| `--services` | Service names. |
| `--images` | Resolved Image references. |
| `--environment` | Environment dùng cho interpolation. |
| `--profiles` | Profile names. |
| `--quiet` | Chỉ validate, không render model. |

State transition: không tạo Docker resource. Dùng trước `up` để phát hiện model/path/interpolation sai.

## 5. `up`

```text
docker compose up [OPTIONS] [SERVICE...]
```

```bash
docker compose up -d --build
```

| Option | Ý nghĩa |
|---|---|
| `-d`, `--detach` | Chạy nền. |
| `--build` | Build Image trước khi start. |
| `--pull POLICY` | Điều khiển pull trước khi run theo policy hỗ trợ. |
| `--force-recreate` | Recreate ngay cả khi Compose không phát hiện config/Image change. |
| `--no-recreate` | Không recreate existing Container. |
| `--remove-orphans` | Xóa Container của project không còn Service trong model hiện tại. |
| `--wait` | Chờ Service running/healthy theo semantics hỗ trợ; thường đi với detached. |

State transition: pull/build khi cần; tạo Network/Volume; create/start/recreate Container; giữ resource đã phù hợp.

`--force-recreate` làm mất writable state trong Container cũ khi object bị thay. Dữ liệu cần giữ phải nằm trong Volume/bind mount phù hợp.

## 6. `ps`

```text
docker compose ps [OPTIONS] [SERVICE...]
```

```bash
docker compose ps
docker compose ps --all
docker compose ps --services --filter status=running
```

| Option | Ý nghĩa |
|---|---|
| `-a`, `--all` | Bao gồm stopped Container. |
| `--services` | Chỉ in service names. |
| `-q`, `--quiet` | Chỉ in Container IDs. |
| `--filter` | Lọc theo field được hỗ trợ như status. |
| `--format` | Định dạng output. |

State transition: read-only.

## 7. `logs`

```text
docker compose logs [OPTIONS] [SERVICE...]
```

```bash
docker compose logs --follow --tail 100 --timestamps backend
```

| Option | Ý nghĩa |
|---|---|
| `-f`, `--follow` | Theo log mới. |
| `--tail N` | Số dòng cuối. |
| `-t`, `--timestamps` | Thêm timestamp. |
| `--since TIME` | Chỉ log sau mốc/khoảng hỗ trợ. |
| `--no-log-prefix` | Bỏ prefix service/container. |

`Ctrl+C` dừng việc follow; detached Container vẫn chạy.

## 8. `stop`, `start`, `restart`

```bash
docker compose stop -t 30 backend
docker compose start backend
docker compose restart -t 30 backend
```

| Command | State trước | State sau | Giữ Container? |
|---|---|---|---|
| `stop` | running | stopped | Có. |
| `start` | stopped | running | Có. |
| `restart` | running/stopped | running | Có, dùng config đã create. |

`-t`, `--timeout` đặt thời gian shutdown trước cưỡng bức theo command support.

Sửa `environment`, `ports` hoặc `volumes` rồi `restart` không áp dụng model mới. Dùng `up -d` để reconcile.

## 9. `down`

```text
docker compose down [OPTIONS] [SERVICES...]
```

```bash
docker compose down --remove-orphans
```

| Option | Ảnh hưởng |
|---|---|
| `--remove-orphans` | Xóa orphan Containers trong project scope. |
| `-v`, `--volumes` | Xóa named/anonymous Volumes thuộc phạm vi command. |
| `--rmi local|all` | Xóa Image theo policy. |
| `-t`, `--timeout` | Shutdown timeout. |

> [!WARNING]
> `down --volumes` có thể xóa dữ liệu không thể phục hồi. `down --rmi all` có thể xóa Image cần build/pull lại. Chạy `ps`, `images`, `volumes` và xác nhận backup trước.

## 10. `rm`

```bash
docker compose rm -f -s -v backend
```

| Option | Ý nghĩa |
|---|---|
| `-f`, `--force` | Không hỏi xác nhận. |
| `-s`, `--stop` | Stop trước khi remove. |
| `-v`, `--volumes` | Remove anonymous Volume gắn với target theo semantics command. |

`rm` nhắm Service Container, không teardown toàn project Network như `down`.

## 11. `exec`

```text
docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

```bash
docker compose exec backend sh
docker compose exec -T database mysqladmin ping -h localhost
```

| Token/option | Ý nghĩa |
|---|---|
| `backend` | Service có existing running Container. |
| `sh` | Executable bên trong Container. |
| `-T` | Tắt pseudo-TTY, hữu ích trong CI/script. |
| `--index N` | Chọn replica index khi Service có nhiều Container. |
| `-e KEY=VALUE` | Thêm/override env cho exec process. |
| `-u USER` | Chạy process bằng user chỉ định. |

State transition: không tạo Container; tạo process mới trong existing Container.

## 12. `run`

```text
docker compose run [OPTIONS] SERVICE [COMMAND] [ARGS...]
```

```bash
docker compose run --rm backend java -version
docker compose run --rm --no-deps backend ./migrate.sh
```

| Option | Ý nghĩa |
|---|---|
| `--rm` | Xóa one-off Container khi kết thúc. |
| `--no-deps` | Không start dependency. |
| `-e KEY=VALUE` | Override environment. |
| `--entrypoint COMMAND` | Override entrypoint. |
| `--service-ports` | Dùng port mapping của Service; có thể xung đột. |
| `-p`, `--publish` | Publish thêm port cho one-off Container. |

State transition: tạo Container mới từ Service configuration. Theo mặc định không publish Service ports để tránh collision với project đang chạy.

## 13. `build`

```bash
docker compose build --pull backend
docker compose build --no-cache backend
```

| Option | Ý nghĩa |
|---|---|
| `--pull` | Cố lấy base Image mới hơn theo builder behavior. |
| `--no-cache` | Không dùng build cache cho layer. |
| `--build-arg KEY=VALUE` | Cấp build arg. |
| `--progress` | Chọn kiểu progress output. |
| `--push` | Push Image sau build nếu cấu hình/hỗ trợ phù hợp. |

State transition: tạo/cập nhật Image; không tự recreate running Container. Sau build thường dùng `up -d`.

## 14. `pull` và `push`

```bash
docker compose pull database
docker compose push backend
```

- `pull` tải Image của Service từ Registry; không start/recreate Container.
- `push` đẩy Image có reference phù hợp; Service chỉ có `build` nhưng không có usable `image` name có thể không push theo cách mong muốn.

## 15. `images`, `volumes`, `port`, `top`

```bash
docker compose images
docker compose volumes
docker compose port backend 8080
docker compose top backend
```

| Command | Kết quả |
|---|---|
| `images` | Image được project Containers sử dụng. |
| `volumes` | Volume liên quan project theo CLI support. |
| `port SERVICE PRIVATE_PORT` | Published host endpoint cho target port. |
| `top` | Process đang chạy trong Service Containers. |

Các command này chủ yếu read-only.

## 16. Multiline theo shell

Bash:

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d --build
```

PowerShell:

```powershell
docker compose `
  -f compose.yaml `
  -f compose.dev.yaml `
  up -d --build
```

Không để whitespace sau backtick PowerShell.

## 17. Workflow an toàn tối thiểu

```text
config -> up -d -> ps -> logs
       -> thay đổi model
       -> config -> up -d -> ps
       -> stop hoặc down tùy mục tiêu
```

Khi nghi ngờ dữ liệu:

1. Xác định project bằng `docker compose ls` và `-p`/`-f` cụ thể.
2. Liệt kê `ps`, `images`, `volumes`.
3. Backup persistent data.
4. Chỉ sau đó dùng `down --volumes`, `rm -v` hoặc Image removal.

## Tài liệu liên quan

- [Compose file keys](compose-file-keys.md)
- [8. Compose CLI và lifecycle](../../learning-path/05-docker-compose/08-compose-cli-va-lifecycle.md)
- Docker Docs, [Docker Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)

[Mục lục Compose Reference](README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)
