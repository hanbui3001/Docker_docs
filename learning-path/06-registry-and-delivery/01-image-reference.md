# 1. Image reference

> **Tóm tắt một câu:** Image reference là địa chỉ có cấu trúc dùng để chọn Registry, repository và phiên bản theo tag hoặc digest; các phần bị bỏ qua được Docker điền bằng quy tắc mặc định chứ không hề “không tồn tại”.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Docker image tag](https://docs.docker.com/reference/cli/docker/image/tag/)

[Mục lục Registry & Delivery](README.md) · [2. Registry, repository và Docker Hub →](02-registry-repository-va-docker-hub.md)

---

## 1. Dạng tổng quát

```text
[REGISTRY_HOST[:PORT]/][NAMESPACE/]REPOSITORY[:TAG][@DIGEST]
```

Ví dụ đầy đủ:

```text
registry.example.com:5000/payments/api:1.4.2@sha256:abc123...
```

| Token | Vai trò |
|---|---|
| `registry.example.com:5000` | Registry host và port tùy chọn. |
| `payments` | Namespace/organization phân nhóm repository. |
| `api` | Repository name. |
| `1.4.2` | Tag dễ đọc, có thể thay đổi trỏ sang nội dung khác. |
| `sha256:abc123...` | Digest định danh nội dung đã resolve. |

**Reference parsing** là quá trình Docker phân tách chuỗi thành các thành phần có nghĩa. Dấu `/` không chỉ phân tách folder; phần đầu được xem là registry khi có đặc điểm hostname như `.` hoặc `:` hay là `localhost`. Dấu `:` cuối path thường mở đầu tag, trong khi `:` trong registry có thể là port.

## 2. Giá trị mặc định

```text
nginx
```

Thông thường được hiểu gần tương đương:

```text
docker.io/library/nginx:latest
```

- Bỏ registry: dùng Docker Hub mặc định.
- Bỏ namespace cho official image: dùng `library` theo quy ước Docker Hub.
- Bỏ tag và digest: dùng tag `latest`.

`latest` chỉ là tag mặc định, không chứng minh đây là bản mới nhất theo thời gian, ổn định nhất hay an toàn nhất.

## 3. Tag và digest trong cùng reference

```bash
docker pull alpine:3.21@sha256:<digest>
```

Khi cả tag và digest xuất hiện, digest cố định nội dung cần lấy; tag làm tăng khả năng đọc và có thể được kiểm tra có khớp với digest tại thời điểm resolve. Cú pháp này hữu ích khi muốn người đọc biết version logic nhưng deployment vẫn khóa artifact.

**Canonical reference** thường là reference theo digest, vì nó trỏ tới content cụ thể. **Familiar reference** là cách viết ngắn, dễ đọc như `alpine:3.21`.

## 4. Local reference và remote destination

```bash
docker image tag api:local registry.example.com/team/api:1.4.2
```

Vế đầu chọn Image trong local image store. Vế sau tạo thêm reference mà lệnh push có thể dùng để chọn Registry/repository đích. Lệnh không tự gửi dữ liệu lên network.

Trước lệnh, Image có reference `api:local`. Sau lệnh, cùng Image ID có thể xuất hiện dưới cả hai tên. Đây là thêm tên, không phải build lại nội dung.

## 5. Lỗi thường gặp

### Nhầm registry port với tag

Trong `localhost:5000/api:1.0`, `5000` là port vì nằm ở phần host trước `/`; `1.0` là tag vì nằm sau repository.

### Nhầm namespace với repository

Trong `team/backend/api:1.0`, cách tổ chức path phụ thuộc Registry, nhưng repository reference là toàn bộ path phù hợp sau registry, không nên suy ra nó là folder host thông thường.

### Cho rằng bỏ tag nghĩa Docker chọn “bản tốt nhất”

Docker chọn `latest`; publisher quyết định tag đó trỏ đâu. Không có quá trình đánh giá chất lượng tự động.

## 6. Tự kiểm tra

Hãy phân tích từng phần của:

```text
ghcr.io/acme/orders:v2@sha256:0123...
localhost:5000/demo/api
redis:7-alpine
```

## 7. Tóm tắt

Reference là địa chỉ có grammar. Luôn resolve phần bị ẩn, phân biệt registry host/port với repository/tag và dùng digest khi cần xác định chính xác artifact.

## Tài liệu tham khảo

- Docker Docs, [docker image tag](https://docs.docker.com/reference/cli/docker/image/tag/)
- Docker Docs, [Image digests](https://docs.docker.com/dhi/core-concepts/digests/)

[Mục lục Registry & Delivery](README.md) · [2. Registry, repository và Docker Hub →](02-registry-repository-va-docker-hub.md)
