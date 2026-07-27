# Part 01. Docker Foundations

[← Mục lục sách](../../README.md)

Phần này xây dựng mental model nền tảng cần thiết để hiểu mọi chủ đề Docker ở các phần sau.

Loại: Learning path index
Cấp độ: Beginner
Điều kiện: Không yêu cầu kiến thức Docker trước đó
Thời gian dự kiến: Khoảng 3 giờ cho toàn bộ sáu chapter, chưa tính thời gian tự kiểm tra và thực hành quan sát.

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
| [1. Docker là gì?](01-docker-la-gi.md) | Có thể đọc |
| [2. Docker hoạt động như thế nào?](02-docker-hoat-dong-nhu-the-nao.md) | Có thể đọc |
| [3. Docker Image](03-docker-image.md) | Có thể đọc |
| [4. Docker Container](04-docker-container.md) | Có thể đọc |
| [5. Image và Container](05-image-va-container.md) | Có thể đọc |
| [6. Bức tranh tổng thể](06-buc-tranh-tong-the.md) | Có thể đọc |

## Cách đọc

Người mới nên đọc tuần tự từ chapter 1 đến chapter 6. Mỗi chapter xây thêm một lớp mental model: nhận diện vấn đề, hiểu kiến trúc, phân biệt Image và Container, rồi kết nối các object trong một luồng tổng thể.

Nếu đã có kinh nghiệm, bạn có thể mở thẳng chapter cần ôn. Khi bấm một thuật ngữ sang Glossary, dùng mục **Quay lại nơi đang học** ở cuối định nghĩa để trở về đúng đoạn vừa đọc.

## Checklist hoàn thành

Bạn hoàn thành Foundation khi có thể diễn đạt bằng lời của mình, thay vì chỉ lặp lại định nghĩa:

- Docker giúp giải quyết vấn đề gì và giới hạn của phạm vi Foundation nằm ở đâu.
- Client, API và daemon phối hợp ở mức khái quát như thế nào.
- Image khác Container ở điểm nào.
- Vì sao một Image có thể tạo nhiều Container mà thay đổi trong từng Container không sửa Image nguồn.
- Image và Container nằm ở đâu trong bức tranh hệ sinh thái Docker.
- Chủ đề nào đã đủ nền tảng để học tiếp và chủ đề nào được chủ động dành cho phần sau.

[← Mục lục sách](../../README.md)
