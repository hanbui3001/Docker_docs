# `docker container run`

> Tạo một Container mới từ Image rồi khởi động process chính của Container đó.

[← CLI quick reference](README.md) · [Bài học tạo Container](../../learning-path/02-cli-and-lifecycle/03-tao-va-chay-container.md)

## Tra nhanh

| Mục | Giá trị |
|---|---|
| Modern form | `docker container run` |
| Alias | `docker run` |
| Tương đương mental model | `container create` + `container start` |
| Dùng lại Container cũ? | Không |
| Object mới | Container, writable layer, runtime configuration |

## Cú pháp

```text
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

```bash
docker container run --name web --detach --publish 8080:80 nginx:alpine
```

| Token | Owner | Ý nghĩa |
|---|---|---|
| `--name web` | Docker run | Đặt tên object. |
| `--detach` / `-d` | Docker run | Chạy nền, in Container ID. |
| `--publish 8080:80` / `-p` | Docker run | Host port `8080` → Container port `80` TCP. |
| `nginx:alpine` | Docker run | Image argument. |

Token sau `IMAGE` là command/argument của process, không mặc định còn thuộc Docker CLI.

## Option thường dùng

| Option | Ý nghĩa | Cảnh báo |
|---|---|---|
| `--name NAME` | Tên Container duy nhất trên daemon. | Stopped Container vẫn giữ tên. |
| `--detach`, `-d` | Detached mode. | Không giữ process sống. |
| `--interactive`, `-i` | Giữ stdin mở. | Thường kết hợp `-t` khi dùng shell. |
| `--tty`, `-t` | Cấp pseudo-TTY. | Không cần cho command non-interactive. |
| `--publish`, `-p` | Publish port. | `HOST:CONTAINER`, không đảo chiều. |
| `--env`, `-e` | Đặt environment variable. | Secret có thể lộ trong history/metadata. |
| `--env-file FILE` | Đọc biến từ file. | File vẫn cần được bảo vệ. |
| `--volume`, `-v` | Mount short syntax. | Part 03 giải thích source/destination. |
| `--mount` | Mount dạng key-value rõ nghĩa. | Dễ đọc hơn cho cấu hình phức tạp. |
| `--rm` | Tự remove sau khi dừng. | Mất writable layer và object để inspect. |
| `--restart POLICY` | Cấu hình restart policy. | Không thay health monitoring. |
| `--pull POLICY` | Quyết định pull Image trước create. | Đọc help theo CLI hiện tại. |

## Command override

```bash
docker container run --rm alpine sh -c "echo hello"
```

`sh -c "echo hello"` chạy bên trong Container. Host shell xử lý quoting trước; sau đó `sh` trong Container parse script.

## Bash multiline

```bash
docker container run \
  --name web \
  --detach \
  --publish 8080:80 \
  nginx:alpine
```

## PowerShell multiline

```powershell
docker container run `
  --name web `
  --detach `
  --publish 8080:80 `
  nginx:alpine
```

Backtick phải là ký tự cuối dòng, không có trailing space.

## Trạng thái trước và sau

- Trước: Image phải resolve được; tên Container chưa tồn tại.
- Trong quá trình: daemon tạo config, writable layer, network/mount rồi start process.
- Sau thành công: Container thường `running`; workload ngắn có thể đã `exited`.
- Sau create failure: có thể không có object hoặc object ở trạng thái lỗi tùy giai đoạn; đọc error và inventory.

## Verification

```bash
docker container ls --all --filter name=web
docker container inspect web --format 'Status={{.State.Status}} Image={{.Image}}'
```

## Cleanup

```bash
docker container stop web
docker container rm web
```

## Liên quan

- `docker container create`: chỉ tạo object.
- `docker container start`: chạy object có sẵn.
- [Docker CLI official reference](https://docs.docker.com/reference/cli/docker/container/run/)

[← CLI quick reference](README.md) · [Bài học tạo Container](../../learning-path/02-cli-and-lifecycle/03-tao-va-chay-container.md)
