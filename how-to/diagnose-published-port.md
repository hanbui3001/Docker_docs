# Chẩn đoán Published Port không truy cập được

> **Mục tiêu:** Xác định lỗi nằm ở Container state, port mapping, địa chỉ listen của ứng dụng hay đường truy cập từ host.

> **Loại:** How-to · **Điều kiện:** Container đã được tạo với `--publish`/`-p`; biết tên Container và port mong đợi<br>
> **Đọc nền:** [Quan sát và debug Container](../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md) · [Port publishing](../learning-path/03-storage-and-networking/06-port-publishing.md)

[← Inspect stopped Container](inspect-stopped-container.md) · [Mục lục How-to](README.md) · [Giữ dữ liệu khi recreate →](preserve-data-when-recreating-container.md)

---

## 1. Vấn đề

Ví dụ bạn mong `http://localhost:8080` đi tới ứng dụng đang nghe port `80` trong Container, nhưng request timeout hoặc connection refused. Đừng đổi ngẫu nhiên firewall, `EXPOSE` hay port; kiểm tra từng lớp theo thứ tự.

## 2. Kiểm tra Container và mapping thực tế

```bash
docker container ls --all --filter name=<container>
docker container port <container>
```

Nếu Container `exited`, chuyển sang [Inspect stopped Container](inspect-stopped-container.md). Nếu output port có dạng `80/tcp -> 0.0.0.0:8080`, Docker đang publish host port `8080` tới Container port `80` trên mọi địa chỉ host phù hợp.

Đọc cấu hình đã resolve:

```bash
docker container inspect <container> --format 'Status={{.State.Status}} Ports={{json .NetworkSettings.Ports}}'
```

Không có mapping cho `80/tcp` nghĩa là Container hiện tại chưa được tạo với publish rule đó. `EXPOSE 80` trong Image không tự publish host port.

## 3. Kiểm tra ứng dụng có listen trong Container

Nếu Image có công cụ phù hợp:

```bash
docker container exec <container> sh -c 'ss -lnt 2>/dev/null || netstat -lnt 2>/dev/null'
```

`exec` tạo process chẩn đoán phụ; `sh -c` cho phép shell xử lý `||` và redirect. Image tối giản có thể không có `sh`, `ss` hoặc `netstat`; khi đó dùng logs và healthcheck thay vì cài công cụ tạm vào Container.

```bash
docker container logs --tail 100 <container>
```

Ứng dụng cần listen đúng Container port đã publish. Nếu nó chỉ bind `127.0.0.1` bên trong Container, traffic đi vào interface Container có thể không tới được; đa số web service trong Container cần listen `0.0.0.0` hoặc địa chỉ phù hợp.

## 4. Kiểm tra từ host

Bash hoặc terminal có `curl`:

```bash
curl --verbose http://127.0.0.1:8080/
```

PowerShell:

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 8080
Invoke-WebRequest -Uri http://127.0.0.1:8080/ -UseBasicParsing
```

`Test-NetConnection` kiểm tra kết nối TCP; `Invoke-WebRequest` kiểm tra thêm HTTP. TCP thành công nhưng HTTP lỗi thường hướng tới route, protocol hoặc ứng dụng, không phải thiếu port mapping.

## 5. Sửa mapping bằng recreate

Port publishing là create-time configuration. Không thể thêm `-p` vào một Container hiện có bằng `docker container start`.

Ví dụ tạo replacement để thử trên host port tạm `18080`:

```bash
docker container run --name <container>-port-test --detach --publish 127.0.0.1:18080:80 <image>
```

| Token | Ý nghĩa |
|---|---|
| `127.0.0.1:18080` | Chỉ publish trên loopback host, dùng port test `18080`. |
| `80` | Port mà ứng dụng phải listen trong Container. |
| `<image>` | Image dùng để tạo replacement; thêm lại env/mount cần thiết của ứng dụng. |

> [!WARNING]
> Không recreate stateful Container mà bỏ mount. Hãy inspect `.Mounts` và dùng [guide bảo toàn dữ liệu](preserve-data-when-recreating-container.md) trước.

## 6. Xác minh

```bash
docker container port <container>-port-test
docker container logs --tail 100 <container>-port-test
curl --verbose http://127.0.0.1:18080/
```

Trên PowerShell, thay lệnh cuối bằng `Invoke-WebRequest`. Chỉ chuyển sang port chính sau khi replacement trả response mong đợi.

## 7. Rollback và cleanup

Nếu replacement không giải quyết lỗi, giữ Container gốc nguyên trạng và xóa bản test:

```bash
docker container stop <container>-port-test
docker container rm <container>-port-test
```

Nếu đã thay Container chính, rollback bằng cách dừng/xóa replacement rồi start Container cũ, miễn là tên, host port và dữ liệu cũ vẫn còn. Không thể chạy đồng thời hai Container cùng bind một host IP/port.

## Tài liệu tham khảo

- Docker CLI, [`docker container run --publish`](https://docs.docker.com/reference/cli/docker/container/run/#publish)
- Docker CLI, [`docker container port`](https://docs.docker.com/reference/cli/docker/container/port/)
- Docker Docs, [Publishing and exposing ports](https://docs.docker.com/get-started/docker-concepts/running-containers/publishing-ports/)

[← Inspect stopped Container](inspect-stopped-container.md) · [Mục lục How-to](README.md) · [Giữ dữ liệu khi recreate →](preserve-data-when-recreating-container.md)
