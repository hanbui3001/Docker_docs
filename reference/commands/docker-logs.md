# `docker container logs`

> Lấy log stream mà logging driver của Container đã thu thập, thường từ stdout và stderr.

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)

## Tra nhanh

| Mục | Giá trị |
|---|---|
| Modern form | `docker container logs` |
| Alias | `docker logs` |
| Điều kiện | Container tồn tại; có thể đang chạy hoặc đã dừng |
| Đọc mọi file log? | Không |
| Mutation | Không |

## Cú pháp

```text
docker container logs [OPTIONS] CONTAINER
```

```bash
docker container logs --tail 100 --since 10m web
```

| Token | Ý nghĩa |
|---|---|
| `--tail 100` | Lấy tối đa 100 dòng cuối. |
| `--since 10m` | Chỉ record từ 10 phút gần đây. |
| `web` | Tên/ID Container. |

## Option thường dùng

| Option | Ý nghĩa |
|---|---|
| `--follow`, `-f` | Stream log mới cho đến khi ngắt. |
| `--tail N` | Số dòng cuối; `all` lấy toàn bộ. |
| `--since TIME` | Log sau mốc thời gian hoặc duration. |
| `--until TIME` | Log trước mốc kết thúc. |
| `--timestamps`, `-t` | Thêm timestamp. |
| `--details` | Hiện extra attribute được logging driver hỗ trợ. |

## Ví dụ debug

```bash
docker container logs --timestamps --tail 200 web
docker container logs --follow --since 30s web
```

`--follow` giữ terminal chờ record mới; dùng `Ctrl+C` để dừng client theo dõi, không mặc định dừng Container.

## Giới hạn

- Lệnh không quét `/var/log` hay mọi file trong filesystem.
- Kết quả phụ thuộc logging driver/configuration.
- Container đã bị remove không còn được tham chiếu bằng lệnh này.
- Log trống có thể do ứng dụng ghi file, chưa ghi output hoặc logging driver khác; không tự chứng minh ứng dụng không chạy.

## Verification bổ sung

```bash
docker container inspect web --format 'Status={{.State.Status}} LogDriver={{.HostConfig.LogConfig.Type}}'
```

## Liên quan

- `docker container inspect`: state/config.
- `docker container attach`: nối vào stdio process chính.
- [`docker container exec`](docker-exec.md): chạy command chẩn đoán phụ.
- [Docker CLI official reference](https://docs.docker.com/reference/cli/docker/container/logs/)

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)
