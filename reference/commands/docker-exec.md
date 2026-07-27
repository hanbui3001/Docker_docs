# `docker container exec`

> Tạo một process phụ trong một Container đang chạy bằng filesystem, namespace và configuration runtime phù hợp của Container đó.

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)

## Tra nhanh

| Mục | Giá trị |
|---|---|
| Modern form | `docker container exec` |
| Alias | `docker exec` |
| Điều kiện | Container đang chạy |
| Tạo Container mới? | Không |
| Tạo process mới? | Có |

## Cú pháp

```text
docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

```bash
docker container exec --interactive --tty web sh
```

| Token | Owner | Ý nghĩa |
|---|---|---|
| `--interactive`, `-i` | Docker exec | Giữ stdin mở. |
| `--tty`, `-t` | Docker exec | Cấp pseudo-terminal. |
| `web` | Docker exec | Container target. |
| `sh` | Runtime process | Executable chạy bên trong. |

## Option thường dùng

| Option | Ý nghĩa |
|---|---|
| `--interactive`, `-i` | Giữ stdin. |
| `--tty`, `-t` | Cấp TTY. |
| `--env`, `-e` | Thêm/override env cho process exec, không sửa vĩnh viễn Container config. |
| `--user`, `-u` | Chạy process bằng user/UID cụ thể. |
| `--workdir`, `-w` | Working directory cho process. |
| `--detach`, `-d` | Chạy process exec ở nền. |
| `--privileged` | Mở rộng quyền process exec. Dùng cực kỳ thận trọng. |

## Shell và command trực tiếp

```bash
docker container exec -it web sh
docker container exec web nginx -t
```

Lệnh đầu cần Image có `sh`. Lệnh thứ hai chạy executable trực tiếp, thường chính xác và ít phụ thuộc shell hơn. `bash` có thể không tồn tại; distroless Image có thể không có shell.

## Environment và shell expansion

```bash
docker container exec web sh -c 'echo "$APP_ENV"'
```

Dấu nháy đơn giúp Bash host không mở rộng `$APP_ENV`; `sh` bên trong mới resolve biến. Trong PowerShell, quoting có quy tắc khác nhưng chuỗi single-quoted cũng thường giữ `$` literal:

```powershell
docker container exec web sh -c 'echo "$APP_ENV"'
```

Nếu bỏ `sh -c`, Docker không tự cung cấp shell để xử lý pipe, redirect hoặc biến.

## Trạng thái trước và sau

- Trước: Container phải `running`; executable phải tồn tại.
- Trong lệnh: runtime tạo process phụ.
- Sau: process phụ có exit code riêng; Container chính vẫn chạy trừ khi command phụ tác động tới nó.
- Thay đổi file do exec ghi vào writable layer có thể mất khi Container bị remove.

## Verification

```bash
docker container exec web id
docker container top web
```

`id` cho biết user của process phụ; `top` có thể giúp quan sát process nếu nó còn chạy đủ lâu.

## Lỗi thường gặp

| Lỗi | Nguyên nhân |
|---|---|
| `container is not running` | Target đang `exited`; dùng inspect/logs. |
| `executable file not found` | Command/shell không có trong Image hoặc PATH. |
| Permission denied | User hiện tại thiếu quyền hoặc file không executable. |
| Biến rỗng bất ngờ | Host shell đã mở rộng trước hoặc process không có env dự kiến. |

## Liên quan

- `docker container attach`: nối process chính, không tạo shell phụ.
- `docker container run`: tạo Container mới.
- [Docker CLI official reference](https://docs.docker.com/reference/cli/docker/container/exec/)

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)
