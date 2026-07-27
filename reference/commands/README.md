# Docker CLI Quick Reference

> **Loại:** Reference · **Phạm vi:** Docker Engine CLI cơ bản<br>
> **Nguồn chính:** [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)

[← Part 02. Docker CLI & Lifecycle](../../learning-path/02-cli-and-lifecycle/README.md)

Trang này dùng để tra nhanh. Nếu chưa hiểu state transition hoặc option scope, đọc [Part 02](../../learning-path/02-cli-and-lifecycle/README.md) trước.

## Cú pháp nền tảng

```text
docker [GLOBAL OPTIONS] OBJECT ACTION [COMMAND OPTIONS] [ARGUMENTS]
```

| Thành phần | Ví dụ | Vai trò |
|---|---|---|
| Global option | `--context prod` | Chọn behavior/endpoint của Docker CLI. |
| Object | `image`, `container`, `volume` | Nhóm tài nguyên. |
| Action | `ls`, `run`, `stop`, `inspect` | Hành động. |
| Command option | `--all`, `--name web` | Điều chỉnh action cụ thể. |
| Argument | `web`, `nginx:alpine` | Target hoặc input. |

## Kiểm tra Engine

| Lệnh | Tác dụng |
|---|---|
| `docker version` | Hiện phiên bản Client và Server/Engine. |
| `docker info` | Hiện configuration và trạng thái tổng quan của daemon. |
| `docker system df` | Tóm tắt disk usage Docker. |

## Image

| Lệnh | Tác dụng | Ghi chú |
|---|---|---|
| `docker image pull nginx:alpine` | Tải Image. | [Chi tiết](docker-pull.md) |
| `docker image ls` | Liệt kê Image cục bộ. | Alias: `docker images`. |
| `docker image inspect IMAGE` | JSON metadata/config/layer. | Chỉ đọc. |
| `docker image history IMAGE` | Xem history entry. | Không phải Dockerfile nguyên vẹn. |
| `docker image build -t app:1.0 .` | Build Image. | Part 04 giải thích sâu. |
| `docker image tag SRC TARGET` | Tạo reference mới. | Không copy toàn bộ layer. |
| `docker image push NAME[:TAG]` | Push lên Registry. | Cần naming/quyền phù hợp. |
| `docker image rm IMAGE` | Xóa Image/reference chỉ định. | Có thể bị dependency cản. |
| `docker image prune` | Xóa dangling Image. | `--all` mở rộng scope. |

## Tạo và liệt kê Container

| Lệnh | Tác dụng | Ghi chú |
|---|---|---|
| `docker container create IMAGE` | Tạo object ở state `created`. | Chưa chạy process. |
| `docker container run IMAGE` | Create rồi start Container mới. | [Chi tiết](docker-run.md) |
| `docker container ls` | Liệt kê Container đang chạy. | Alias: `docker ps`. [Chi tiết](docker-ps.md) |
| `docker container ls --all` | Liệt kê cả stopped Container. | Alias: `docker ps -a`. |

## Lifecycle

| Lệnh | State transition chính |
|---|---|
| `docker container start web` | `created/exited -> running` |
| `docker container stop web` | `running -> exited`, ưu tiên graceful shutdown. |
| `docker container restart web` | Stop/start cùng object. |
| `docker container kill web` | Gửi signal, mặc định kết thúc cưỡng bức. |
| `docker container pause web` | `running -> paused`. |
| `docker container unpause web` | `paused -> running`. |
| `docker container rm web` | Xóa stopped Container. |
| `docker container rm --force web` | Kết thúc cưỡng bức rồi xóa target. |

## Quan sát và tương tác

| Lệnh | Tác dụng | Chi tiết |
|---|---|---|
| `docker container inspect web` | JSON configuration và state. | Dùng `--format` để lọc. |
| `docker container logs web` | Đọc log stream. | [Chi tiết](docker-logs.md) |
| `docker container exec -it web sh` | Tạo process phụ trong Container chạy. | [Chi tiết](docker-exec.md) |
| `docker container stats --no-stream web` | Snapshot resource usage. | Không tự kết luận root cause. |
| `docker container top web` | Liệt kê process. | Output phụ thuộc platform. |
| `docker container port web` | Hiện published port. | Không chứng minh app ready. |
| `docker container cp web:/path ./path` | Copy giữa host và Container. | Cẩn thận path direction. |
| `docker container attach web` | Gắn vào stdio process chính. | Khác `exec`. |

## Registry

| Lệnh | Tác dụng |
|---|---|
| `docker login` | Xác thực với Registry. |
| `docker logout` | Xóa credential/session theo Registry. |
| `docker image push NAME:TAG` | Phân phối Image. |
| `docker image pull NAME:TAG` | Tải Image. |

## Cleanup

> [!WARNING]
> Luôn chạy `docker system df` và inventory object trước các lệnh prune. `unused` theo Docker không có nghĩa dữ liệu vô giá trị với bạn.

| Lệnh | Phạm vi mặc định |
|---|---|
| `docker container prune` | Mọi stopped Container phù hợp filter. |
| `docker image prune` | Dangling Image. |
| `docker builder prune` | Build cache đủ điều kiện. |
| `docker system prune` | Nhiều nhóm unused object; không xóa running Container hay named Volume mặc định. |

## Bash và PowerShell

Bash multiline dùng `\`; PowerShell dùng backtick và không được có khoảng trắng sau backtick. Bash/PowerShell 7 hỗ trợ `&&`; Windows PowerShell 5.1 dùng `$LASTEXITCODE` để chỉ chạy bước sau khi bước trước thành công.

## Help

```bash
docker --help
docker image --help
docker container run --help
```

[← Part 02. Docker CLI & Lifecycle](../../learning-path/02-cli-and-lifecycle/README.md)
