# Part 02. Docker CLI & Container Lifecycle

[← Part 01. Docker Foundations](../01-foundations/README.md) · [Mục lục sách](../../README.md) · [1. Cách đọc lệnh Docker →](01-cach-doc-lenh-docker.md)

Phần này biến mental model của Foundation thành khả năng thao tác có chủ đích: đọc đúng một câu lệnh, biết object nào bị tác động, dự đoán trạng thái trước và sau, rồi kiểm chứng bằng output của Docker.

Loại: Learning path index  
Cấp độ: Beginner  
Điều kiện: Đã phân biệt được Image và Container  
Thời gian dự kiến: Khoảng 4 giờ, chưa tính thời gian tự chạy ví dụ.

## Phạm vi

- Cấu trúc `docker + object + action + options + arguments`.
- Lệnh quản lý Image và ý nghĩa của local Image store.
- Sự khác nhau giữa `create`, `run`, `start` và command chạy trong Container.
- Lifecycle từ `created` đến `running`, `exited` và `removed`.
- Quan sát bằng `ps`, `inspect`, `logs`, `stats`, `top`, `port`; tương tác bằng `exec`.
- Cleanup theo đúng object và phạm vi, bao gồm cảnh báo dữ liệu với `prune` và `rm --force`.
- Điểm khác giữa Bash và PowerShell khi nối lệnh, xuống dòng và truyền biến.

Part này chưa dạy chi tiết Volume, Network, Dockerfile hay Compose. Những object đó chỉ xuất hiện khi cần giải thích tác động của một CLI option.

## Lộ trình chapter

| Chapter | Câu hỏi chính |
|---|---|
| [1. Cách đọc lệnh Docker](01-cach-doc-lenh-docker.md) | Mỗi token thuộc về parser và scope nào? |
| [2. Lệnh quản lý Image](02-lenh-quan-ly-image.md) | Pull, list, inspect, history và remove thay đổi gì? |
| [3. Tạo và chạy Container](03-tao-va-chay-container.md) | `run` kết hợp những bước nào và option được áp dụng khi nào? |
| [4. Container lifecycle](04-container-lifecycle.md) | `start`, `stop`, `restart`, `kill`, `rm` khác nhau ra sao? |
| [5. Quan sát và debug Container](05-quan-sat-va-debug-container.md) | Chọn `ps`, `inspect`, `logs` hay `exec` dựa trên loại bằng chứng nào? |
| [6. Dọn dẹp tài nguyên](06-don-dep-tai-nguyen.md) | Xóa đúng object mà không mở rộng phạm vi ngoài ý muốn như thế nào? |

## Reference đi kèm

- [Docker CLI quick reference](../../reference/commands/README.md)
- [`docker image pull`](../../reference/commands/docker-pull.md)
- [`docker container run`](../../reference/commands/docker-run.md)
- [`docker container ls`](../../reference/commands/docker-ps.md)
- [`docker container logs`](../../reference/commands/docker-logs.md)
- [`docker container exec`](../../reference/commands/docker-exec.md)

## Cách học hiệu quả

Đừng chỉ chép lệnh. Với mỗi ví dụ, hãy tự trả lời bốn câu: object mục tiêu là gì, option thuộc lệnh nào, trạng thái nào đổi, và dùng lệnh nào để chứng minh kết quả. Nếu không trả lời được, hãy quay lại bảng token thay vì thử thêm option ngẫu nhiên.

## Checklist hoàn thành

- Đọc được notation có `[]`, `...`, option và argument.
- Giải thích được vì sao `run` tạo mới còn `start` dùng lại Container cũ.
- Phân biệt trạng thái Container với trạng thái process chính.
- Chọn đúng công cụ quan sát trước khi sửa lỗi.
- Mô tả chính xác phạm vi của từng lệnh cleanup và dữ liệu nào có thể mất.

[← Part 01. Docker Foundations](../01-foundations/README.md) · [Mục lục sách](../../README.md) · [1. Cách đọc lệnh Docker →](01-cach-doc-lenh-docker.md)
