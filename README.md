# Docker: Từ nền tảng đến vận hành

> Tài liệu Docker bằng tiếng Việt, tập trung vào mental model, syntax,
> cách kiểm chứng và những quan niệm dễ gây hiểu nhầm.

## Tài liệu này dành cho ai?

Bộ tài liệu dành cho người mới học Docker, developer muốn củng cố mental model và người cần tra cứu cách Docker xử lý Image, Container, build, delivery và vận hành. Sau khi đi hết lộ trình, người đọc có thể giải thích các thành phần chính, kiểm soát vòng đời Container, đóng gói ứng dụng và chẩn đoán vấn đề dựa trên trạng thái quan sát được.

Bạn có thể đọc tuần tự như một cuốn sách hoặc đi thẳng đến khu vực phù hợp với nhu cầu hiện tại:

| Nhu cầu | Khu vực |
|---|---|
| Học từ đầu | Learning Path |
| Thực hành có hướng dẫn | Tutorials |
| Hoàn thành một công việc cụ thể | How-to |
| Tra cứu syntax | Reference |
| Xem lỗi và cách sửa thực tế | Case Studies |

## Lộ trình tám phần

```mermaid
flowchart LR
    P01["Part 01<br/>Foundations"] --> P02["Part 02<br/>CLI & Lifecycle"]
    P02 --> P03["Part 03<br/>Storage & Networking"]
    P03 --> P04["Part 04<br/>Dockerfile"]
    P04 --> P05["Part 05<br/>Docker Compose"]
    P05 --> P06["Part 06<br/>Registry & Delivery"]
    P06 --> P07["Part 07<br/>Production"]
    P07 --> P08["Part 08<br/>Troubleshooting"]
```

Lộ trình đi từ **hiểu** mental model, **điều khiển** vòng đời, **lưu trữ và kết nối** dữ liệu, **đóng gói** ứng dụng, **phối hợp** nhiều service, **phân phối** Image, **vận hành** hệ thống đến **chẩn đoán** sự cố. Mỗi phần dùng kiến thức của phần trước làm nền, nhưng các khu vực tra cứu và how-to vẫn có thể được dùng độc lập khi cần giải quyết một công việc cụ thể.

## Mục lục tổng

| Phần | Trọng tâm | Trạng thái |
|---|---|---|
| [Part 01. Docker Foundations](learning-path/01-foundations/README.md) | Mental model nền tảng về Docker, Image, Container và hệ sinh thái | Có thể đọc |
| [1. Docker là gì?](learning-path/01-foundations/01-docker-la-gi.md) | Vấn đề Docker giải quyết, use case và giới hạn | Có thể đọc |
| [2. Docker hoạt động như thế nào?](learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md) | Client, API, Engine, Daemon và các luồng build, pull, run | Có thể đọc |
| [3. Docker Image](learning-path/01-foundations/03-docker-image.md) | Cấu trúc Image, layer, metadata và quan hệ với Container | Có thể đọc |
| [4. Docker Container](learning-path/01-foundations/04-docker-container.md) | Runtime instance, lifecycle, isolation và writable state | Có thể đọc |
| [5. Image và Container](learning-path/01-foundations/05-image-va-container.md) | Mental model kết hợp và các quan hệ vòng đời | Có thể đọc |
| [6. Bức tranh tổng thể](learning-path/01-foundations/06-buc-tranh-tong-the.md) | Luồng từ Dockerfile đến Image, Registry, Container, dữ liệu và kết nối | Có thể đọc |
| Part 02. CLI & Lifecycle | Cú pháp CLI và vòng đời Docker object | Thiết kế đã được duyệt; chưa triển khai |
| Part 03. Storage & Networking | Volume, bind mount, network và kết nối service | Thiết kế đã được duyệt; chưa triển khai |
| Part 04. Dockerfile | Build context, instruction, cache và multi-stage build | Thiết kế đã được duyệt; chưa triển khai |
| Part 05. Docker Compose | Mô tả và phối hợp ứng dụng nhiều service | Thiết kế đã được duyệt; chưa triển khai |
| Part 06. Registry & Delivery | Tag, Digest, Registry và quy trình phân phối Image | Thiết kế đã được duyệt; chưa triển khai |
| Part 07. Production | Bảo mật, tài nguyên, quan sát và vận hành | Thiết kế đã được duyệt; chưa triển khai |
| Part 08. Troubleshooting | Điều tra trạng thái, log, network, storage và build | Thiết kế đã được duyệt; chưa triển khai |

Tài liệu dùng chung hiện có:

- [Docker Glossary](reference/glossary.md) — định nghĩa thống nhất cho các thuật ngữ cốt lõi.
- [Docker Documentation Style Guide](STYLE-GUIDE.md) — quy chuẩn biên tập, cấu trúc và kiểm chứng nội dung.

## Quy ước của repository

- Chapter trong Learning Path dùng số thứ tự để thể hiện vị trí học được khuyến nghị.
- Thuật ngữ kỹ thuật tiếng Anh được giữ nguyên và giải thích bằng tiếng Việt ở lần xuất hiện đầu tiên.
- Định nghĩa sâu liên kết tới Glossary; mỗi mục Glossary có backlink về đúng đoạn đang học.
- Syntax quan trọng được phân tích từ token, scope và giá trị đã resolve đến trạng thái trước và sau khi thực thi.
- Các quan niệm dễ gây hiểu nhầm được nêu rõ, sửa bằng mental model chính xác và đi kèm cách kiểm chứng.
- Diagram dùng Mermaid khi relationship, sequence hoặc state change khó diễn đạt gọn bằng văn xuôi; mỗi diagram đều được giải thích ở phần liền kề.
- Docker Official Documentation là nguồn chính; OCI Specification được dùng khi cần độ chính xác về định dạng Image hoặc runtime.

## Trạng thái xuất bản

| Phạm vi | Trạng thái |
|---|---|
| Slice đầu tiên: book cover, quy chuẩn, glossary và Foundation hoàn chỉnh | Hoàn thành |
| Part 01 hoàn chỉnh | Có thể đọc |
| Parts 02-08 | Thiết kế đã được duyệt; chưa triển khai |

Part 01 hiện là nền tảng hoàn chỉnh để bắt đầu Part 02: CLI & Lifecycle. Các phần chưa có file đích tiếp tục được giữ dưới dạng plain text để không tạo broken link.
