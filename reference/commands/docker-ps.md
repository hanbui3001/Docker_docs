# `docker container ls`

> Liệt kê Container theo view và filter; lệnh chỉ đọc, không thay đổi lifecycle state.

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)

## Tra nhanh

| Mục | Giá trị |
|---|---|
| Modern form | `docker container ls` |
| Alias | `docker ps` |
| Mặc định | Chỉ Container đang chạy |
| Hiện tất cả | `--all`, `-a` |
| Mutation | Không |

## Cú pháp

```text
docker container ls [OPTIONS]
```

```bash
docker container ls --all --filter status=exited
```

| Token | Ý nghĩa |
|---|---|
| `container ls` | Chọn inventory Container. |
| `--all` | Bao gồm cả Container không chạy. |
| `--filter status=exited` | Thu hẹp theo state `exited`. |

## Option thường dùng

| Option | Ý nghĩa |
|---|---|
| `--all`, `-a` | Hiện mọi state. |
| `--quiet`, `-q` | Chỉ in ID. |
| `--filter`, `-f` | Lọc theo name, status, ancestor, label và tiêu chí hỗ trợ. |
| `--format` | Định dạng output bằng template. |
| `--no-trunc` | Không rút gọn ID/command. |
| `--last N`, `-n N` | Hiện N Container tạo gần nhất. |
| `--latest`, `-l` | Hiện Container tạo gần nhất. |
| `--size`, `-s` | Thêm size writable layer/root filesystem view. |

## Cột output

`CREATED` là tuổi object; `STATUS` phản ánh lifecycle/health; `COMMAND` có thể bị rút gọn; `PORTS` chỉ cho biết publish config, không chứng minh service ready.

## Ví dụ

```bash
docker container ls --all --filter ancestor=nginx:alpine
docker container ls --filter name=web --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker container ls --all --quiet
```

Lệnh cuối thường được dùng làm input cho script. Nếu danh sách rỗng, đừng giả định command nhận danh sách sẽ luôn xử lý an toàn; kiểm tra shell/script logic.

## Verification

Vì đây là lệnh quan sát, verification là so sánh với inspect target:

```bash
docker container inspect web --format '{{.State.Status}}'
```

## Liên quan

- `docker container inspect`: chi tiết object cụ thể.
- `docker container stats`: resource runtime.
- [Docker CLI official reference](https://docs.docker.com/reference/cli/docker/container/ls/)

[← CLI quick reference](README.md) · [Bài học quan sát](../../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)
