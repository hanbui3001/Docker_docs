# Part 01. Docker Foundations

[← Mục lục sách](../../README.md)

Phần này xây dựng mental model nền tảng cần thiết để hiểu mọi chủ đề Docker ở các phần sau.

Loại: Learning path index
Cấp độ: Beginner
Điều kiện: Không yêu cầu kiến thức Docker trước đó

## Phạm vi của Foundation

Foundation tập trung vào các khái niệm giúp người mới nhìn Docker như một hệ thống thống nhất:

- Docker là gì và giải quyết nhóm vấn đề nào.
- Kiến trúc Docker ở mức nền tảng, gồm vai trò của client, API và daemon.
- **Image** — gói mẫu chỉ đọc dùng làm đầu vào để tạo Container.
- **Container** — môi trường chạy cụ thể được tạo từ Image.
- Mối quan hệ giữa Image và Container, gồm việc một Image có thể tạo nhiều Container độc lập.
- Bức tranh tổng thể về các thành phần chính trong hệ sinh thái Docker và cách chúng kết nối với nhau.

Volume, Network và Registry chỉ được giới thiệu ngắn khi cần hoàn thiện bức tranh hệ sinh thái. Quy trình phân phối Image qua Registry, cú pháp Dockerfile, cú pháp Compose và vận hành production được dành cho các phần sau; Foundation không đi sâu vào cách cấu hình hay vận hành các chủ đề đó.

## Lộ trình chapter

| Chapter | Trạng thái |
|---|---|
| 1. Docker là gì? | Đã lên kế hoạch |
| 2. Docker hoạt động như thế nào? | Đã lên kế hoạch |
| 3. Docker Image | Đang triển khai |
| 4. Docker Container | Đã lên kế hoạch |
| 5. Image và Container | Đã lên kế hoạch |
| 6. Bức tranh tổng thể | Đã lên kế hoạch |

## Cách đọc trong giai đoạn hiện tại

Chapter Docker Image được ưu tiên triển khai trước để kiểm chứng format Explanation chuẩn sẽ dùng cho các chapter lý thuyết sau này. Đây là thứ tự xuất bản tạm thời, không phải khuyến nghị người mới bỏ qua vĩnh viễn chapter 1 và 2.

Khi đủ sáu chapter, người mới nên học theo thứ tự từ 1 đến 6 để có ngữ cảnh trước khi đi sâu vào Image. Trong giai đoạn đầu, chapter Image đóng vai trò bản mẫu để xác nhận cách trình bày mental model, định nghĩa chính xác, cơ chế và quan niệm dễ gây hiểu nhầm.

## Checklist hoàn thành

Bạn hoàn thành Foundation khi có thể diễn đạt bằng lời của mình, thay vì chỉ lặp lại định nghĩa:

- Docker giúp giải quyết vấn đề gì và giới hạn của phạm vi Foundation nằm ở đâu.
- Client, API và daemon phối hợp ở mức khái quát như thế nào.
- Image khác Container ở điểm nào.
- Vì sao một Image có thể tạo nhiều Container mà thay đổi trong từng Container không sửa Image nguồn.
- Image và Container nằm ở đâu trong bức tranh hệ sinh thái Docker.
- Chủ đề nào đã đủ nền tảng để học tiếp và chủ đề nào được chủ động dành cho phần sau.

[← Mục lục sách](../../README.md)
