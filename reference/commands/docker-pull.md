# `docker image pull`

> Tải Image reference và content còn thiếu từ Registry vào local Image store.

[← CLI quick reference](README.md) · [Bài học Image commands](../../learning-path/02-cli-and-lifecycle/02-lenh-quan-ly-image.md)

## Tra nhanh

| Mục | Giá trị |
|---|---|
| Modern form | `docker image pull` |
| Alias | `docker pull` |
| Object bị thay đổi | Local Image store |
| Tạo Container? | Không |
| Destructive? | Không; có thể dùng network/disk và cập nhật Tag local |

## Cú pháp

```text
docker image pull [OPTIONS] NAME[:TAG|@DIGEST]
```

```bash
docker image pull nginx:alpine
```

| Token | Loại | Ý nghĩa |
|---|---|---|
| `docker image` | Object group | Quản lý Image. |
| `pull` | Action | Resolve reference qua Registry rồi tải content thiếu. |
| `nginx:alpine` | Argument | Repository `nginx`, Tag `alpine`. |

Nếu bỏ Tag, Docker thường dùng `latest`; `latest` chỉ là tên Tag, không bảo đảm mới nhất theo thời gian hay phù hợp production.

## Option thường dùng

| Option | Ý nghĩa |
|---|---|
| `--platform linux/amd64` | Chọn platform cụ thể khi reference là multi-platform. |
| `--all-tags` | Tải mọi tagged Image trong repository được hỗ trợ. |
| `--quiet` | Giảm output. |

## Tag và Digest

```bash
docker image pull alpine:3.20
docker image pull alpine@sha256:<digest>
```

Tag có thể được publisher cập nhật. Digest định danh content cụ thể hơn; exact digest phải lấy từ nguồn tin cậy hoặc output Registry/pull.

## Trạng thái trước và sau

- Trước: local daemon có thể thiếu reference, manifest hoặc layer.
- Sau: reference local resolve tới Image và content cần thiết có trong content store.
- Không đổi: không có Container mới, không có process mới.

## Verification

```bash
docker image inspect nginx:alpine --format 'Id={{.Id}} Digests={{json .RepoDigests}}'
```

## Lỗi thường gặp

| Lỗi | Nguyên nhân cần kiểm tra |
|---|---|
| `pull access denied` | Sai repository, private Registry hoặc chưa login. |
| `manifest unknown` | Tag/Digest không tồn tại. |
| `no matching manifest` | Reference không có platform phù hợp. |
| Timeout/TLS | Network, proxy, certificate hoặc Registry endpoint. |

## Liên quan

- [`docker container run`](docker-run.md) có thể tự pull theo pull policy khi Image thiếu.
- `docker image push` đi theo hướng local store → Registry.
- [Docker CLI official reference](https://docs.docker.com/reference/cli/docker/image/pull/)

[← CLI quick reference](README.md) · [Bài học Image commands](../../learning-path/02-cli-and-lifecycle/02-lenh-quan-ly-image.md)
