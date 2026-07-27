# Part 05. Docker Compose

[← Mục lục sách](../../README.md)

Phần này xây dựng mental model để đọc, viết và vận hành một ứng dụng nhiều Container bằng Docker Compose. Trọng tâm không phải học thuộc YAML, mà là hiểu Compose biến file khai báo thành Service, Container, Network và Volume như thế nào.

Loại: Learning path index  
Cấp độ: Beginner đến Intermediate  
Điều kiện: Đã hiểu Image, Container, Dockerfile, Volume và Network ở mức cơ bản  
Thời gian dự kiến: Khoảng 5-6 giờ cho tám chapter, chưa tính thời gian tự thử lệnh.

## Phạm vi của phần

Sau Part 05, bạn có thể:

- Phân biệt Dockerfile, Compose file, Service và Container.
- Đọc cây YAML theo đúng scope sở hữu từng key.
- Hiểu quan hệ giữa `image`, `build`, build context và Dockerfile.
- Phân tích chính xác short/long syntax của Port và Volume.
- Phân biệt Compose interpolation với environment thực sự đi vào Container.
- Mô hình hóa startup dependency bằng `depends_on` và `healthcheck` mà không nhầm nó với khả năng tự phục hồi lúc runtime.
- Dùng Compose CLI để tạo, quan sát, dừng, khởi động lại, recreate và xóa tài nguyên có chủ đích.

## Lộ trình chapter

| Chapter | Câu hỏi chính |
|---|---|
| [1. Docker Compose là gì?](01-compose-la-gi.md) | Compose giải quyết vấn đề gì và tạo ra những object nào? |
| [2. Cách đọc Compose YAML](02-cach-doc-compose-yaml.md) | Indentation, mapping, sequence, scalar và scope hoạt động ra sao? |
| [3. Service, Image và build](03-service-image-va-build.md) | Service khác Container thế nào; `image` và `build` phối hợp ra sao? |
| [4. Ports và Network](04-ports-va-network.md) | Giao tiếp nội bộ khác publish ra host ở đâu? |
| [5. Environment và interpolation](05-environment-va-interpolation.md) | Biến nào được Compose thay thế, biến nào đi vào process? |
| [6. Volume trong Compose](06-volume-trong-compose.md) | Hai key `volumes` khác scope và vòng đời như thế nào? |
| [7. Healthcheck và dependency](07-healthcheck-va-dependency.md) | “Đã start” khác “đã sẵn sàng” ở đâu? |
| [8. Compose CLI và lifecycle](08-compose-cli-va-lifecycle.md) | `up`, `stop`, `down`, `run`, `exec` thay đổi trạng thái nào? |

## Cách đọc

Người mới nên đọc tuần tự. Chapter 2 cung cấp ngữ pháp YAML; các chapter 3-7 lần lượt áp dụng ngữ pháp đó vào từng nhóm cấu hình; chapter 8 kết nối file khai báo với lifecycle thật của project.

Mỗi thuật ngữ mới được giải thích ngay tại lần xuất hiện đầu tiên. Khi cần tra cú pháp nhanh thay vì đọc bài giải thích, dùng:

- [Compose file keys](../../reference/compose/compose-file-keys.md)
- [Compose commands](../../reference/compose/compose-commands.md)

## Ví dụ xuyên suốt

Các chapter dùng một mô hình nhỏ gồm `backend` và `database`:

```yaml
services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      DB_HOST: database
    depends_on:
      database:
        condition: service_healthy

  database:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: example-only
    volumes:
      - database-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  database-data:
```

Giá trị mật khẩu trên chỉ dùng để đọc cú pháp. Tài liệu không khuyến nghị commit secret thật vào Compose file.

## Checklist hoàn thành

Bạn hoàn thành Part 05 khi có thể giải thích bằng lời của mình:

- Vì sao Service không đồng nghĩa với đúng một Container.
- Vì sao hai key cùng tên như `volumes` có thể mang vai trò khác nhau dựa trên vị trí trong cây YAML.
- `"8080:80"` và `database-data:/var/lib/mysql` được parser tách thành những phần nào.
- `.env`, `env_file` và `environment` khác nhau ở nguồn, thời điểm và đích đến.
- `service_healthy` bảo đảm điều gì lúc startup và không bảo đảm điều gì sau đó.
- Khi nào dùng `up`, `stop`, `down`, `exec`, `run` và `config`.

[← Mục lục sách](../../README.md) · [1. Docker Compose là gì? →](01-compose-la-gi.md)
