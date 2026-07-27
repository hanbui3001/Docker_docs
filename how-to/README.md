# Docker How-to Guides

> Các hướng dẫn ngắn để hoàn thành một tác vụ Docker cụ thể. Mỗi guide giả định bạn đã có mental model nền tảng và tập trung vào kiểm tra trạng thái, thay đổi, xác minh và recovery.

[Mục lục sách](../README.md) · [Docker CLI & Lifecycle](../learning-path/02-cli-and-lifecycle/README.md) · [Storage & Networking](../learning-path/03-storage-and-networking/README.md)

## Chẩn đoán

| Tác vụ | Khi nào dùng |
|---|---|
| [Inspect một Container đã dừng](inspect-stopped-container.md) | Container không chạy và bạn cần giữ bằng chứng trước khi sửa hoặc xóa. |
| [Chẩn đoán published port](diagnose-published-port.md) | Host không truy cập được ứng dụng qua port đã publish. |

## Dữ liệu và cleanup

| Tác vụ | Khi nào dùng |
|---|---|
| [Giữ dữ liệu khi recreate Container](preserve-data-when-recreating-container.md) | Cần tạo Container mới nhưng không được mất dữ liệu hiện tại. |
| [Dọn disk Docker an toàn](clean-docker-disk-safely.md) | Docker chiếm nhiều dung lượng và cần thu hồi theo scope có kiểm soát. |

## Cách sử dụng

Thay các placeholder như `<container>` và `<volume>` bằng giá trị thật sau khi đã kiểm tra inventory. Không chạy nguyên một khối lệnh destructive nếu chưa hiểu target và recovery được ghi trong guide.

[Mục lục sách](../README.md) · [Docker CLI & Lifecycle](../learning-path/02-cli-and-lifecycle/README.md) · [Storage & Networking](../learning-path/03-storage-and-networking/README.md)
