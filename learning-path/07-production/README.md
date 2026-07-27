# Part 07. Docker trong Production

[← Part 06. Registry & Delivery](../06-registry-and-delivery/README.md) · [Mục lục sách](../../README.md)

> **Mục tiêu:** Chuyển từ “Container chạy được” sang workload có artifact xác định, ranh giới bảo mật, giới hạn tài nguyên, dữ liệu bền vững và khả năng quan sát.

> **Loại:** Learning Path · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 4-6 giờ  
> **Điều kiện:** Đã hiểu Image, Container, Volume, Network, Dockerfile, Compose và Registry.

## Bạn sẽ học gì?

1. Production readiness thực sự gồm những lớp nào.
2. Vì sao non-root user và quyền tối thiểu làm giảm blast radius.
3. Cách tách configuration và secret khỏi Image.
4. Cách CPU, memory và PID limit ảnh hưởng workload.
5. Vai trò khác nhau của healthcheck, restart policy và monitoring.
6. Cách thiết kế log và bằng chứng chẩn đoán.
7. Cách đánh giá workload trước khi phát hành.

## Thứ tự học

| Chapter | Trọng tâm |
|---|---|
| [1. Production readiness](01-production-readiness.md) | Các lớp cần kiểm soát ngoài trạng thái `running` |
| [2. Runtime security](02-runtime-security.md) | Non-root, capability và read-only filesystem |
| [3. Configuration và secret](03-configuration-va-secret.md) | Tách artifact khỏi dữ liệu theo môi trường |
| [4. Resource limits](04-resource-limits.md) | CPU, memory, PID và hành vi khi vượt giới hạn |
| [5. Health và restart](05-health-va-restart.md) | Healthcheck, dependency và recovery |
| [6. Logging và observability](06-logging-va-observability.md) | Log stream, metadata, metric và correlation |
| [7. Production checklist](07-production-checklist.md) | Kiểm tra có hệ thống trước khi deploy |

## Ranh giới của Part

Part này dùng Docker Engine và Compose để giải thích nguyên tắc vận hành. Nó không biến Compose thành một orchestrator đầy đủ và không thay thế Kubernetes, cloud load balancer, secret manager hay monitoring platform. Mental model ở đây được thiết kế để chuyển tiếp sang các nền tảng đó.

## Hoàn thành khi bạn có thể

- Giải thích vì sao `running` chưa đồng nghĩa production-ready.
- Chỉ ra dữ liệu nào thuộc Image, runtime configuration, secret, Volume và log stream.
- Dự đoán điều gì xảy ra khi process vượt memory limit hoặc process chính thoát.
- Phân biệt healthcheck, restart policy, readiness và monitoring.
- Đọc một Compose file và nêu được các rủi ro vận hành chính.

## Cách học và kiểm chứng

Đọc tuần tự từ chapter 1 đến chapter 6, sau đó dùng chapter 7 như một cổng review. Mỗi cấu hình production nên được kiểm tra ở hai lớp:

1. **Desired configuration** — trạng thái mong muốn được viết trong Dockerfile, Compose file hoặc deployment configuration.
2. **Effective runtime state** — trạng thái thực tế Docker Engine đã áp dụng, được quan sát bằng `docker inspect`, `docker stats`, log và request kiểm tra.

Một file trông đúng chưa đủ nếu runtime bỏ qua key, dùng Image khác, mount sai path hoặc chưa recreate Container sau khi cấu hình đổi.

[← Part 06. Registry & Delivery](../06-registry-and-delivery/README.md) · [Mục lục sách](../../README.md) · [1. Production readiness →](01-production-readiness.md)
