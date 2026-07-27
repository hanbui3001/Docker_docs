# 6. Logging và observability

> **Tóm tắt một câu:** Observability là khả năng suy ra trạng thái nội bộ từ tín hiệu bên ngoài; với Container, log stream chỉ là một tín hiệu trong tập logs, metrics, traces và metadata.

> **Loại:** Explanation · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Docker logging](https://docs.docker.com/engine/logging/) · [Configure logging drivers](https://docs.docker.com/engine/logging/configure/)

> **Sau chapter này, bạn có thể:**
> - Phân biệt log stream, logging driver, rotation và retention.
> - Phân tích token của `docker logs` và scope của Compose `logging`.
> - Chọn logs, metrics, traces hoặc metadata theo câu hỏi điều tra.
> - Kiểm tra effective logging configuration và tránh rò rỉ dữ liệu nhạy cảm.

> **Thuật ngữ:** **Log** ghi sự kiện. **Metric** là giá trị số theo thời gian. **Trace** nối hành trình của một request qua nhiều thành phần. **Correlation ID** là định danh giúp ghép các bản ghi thuộc cùng một luồng. **Telemetry** là tập tín hiệu được hệ thống phát ra để quan sát, thường gồm logs, metrics và traces.

[← 5. Health và restart](05-health-va-restart.md) · [Mục lục Production](README.md) · [7. Production checklist →](07-production-checklist.md)

---

## 1. Viết log ra stdout và stderr

Docker thu thập output chuẩn của process qua logging driver. Với ứng dụng thông thường, ghi application log ra stdout/stderr giúp `docker logs` và hệ thống collector nhận cùng stream mà không phụ thuộc file path bên trong writable layer.

```bash
docker container logs --since 10m --tail 200 --timestamps api
```

| Token | Scope | Ý nghĩa |
|---|---|---|
| `docker container logs` | Docker CLI | Đọc log stream của một Container đã tồn tại. |
| `--since 10m` | Option của `logs` | Chỉ lấy record từ mười phút gần nhất theo timestamp được driver/runtime cung cấp. |
| `--tail 200` | Option của `logs` | Giới hạn 200 dòng cuối trước khi kết thúc hoặc follow. |
| `--timestamps` | Option của `logs` | Yêu cầu Docker thêm timestamp vào output. |
| `api` | Argument | Container name hoặc ID, không phải Service name trừ khi tên thực tế trùng nhau. |

Lệnh chỉ đọc log, không thay Container state. Thêm `--follow` giữ terminal chờ record mới; `Ctrl+C` dừng client theo dõi, không dừng detached Container.

Không ghi secret, access token hoặc toàn bộ credential vào log. **Redaction** — loại bỏ hoặc che phần nhạy cảm trước khi record rời ứng dụng — phải được thực hiện có chủ đích; collector không thể luôn đoán field nào là secret.

## 2. Structured logging và schema

**Structured log** biểu diễn field có tên thay vì chỉ một câu tự do:

```json
{"timestamp":"2026-07-28T10:30:00Z","level":"ERROR","service":"checkout","version":"1.4.2","request_id":"req-42","event":"payment_timeout","duration_ms":3000}
```

Field ổn định giúp lọc và aggregate tốt hơn. JSON không tự làm log có chất lượng; schema không nhất quán hoặc message thiếu context vẫn khó dùng.

Các field thường hữu ích:

- `timestamp`, `level`, `service`, `version`, `environment`.
- `request_id` hoặc `trace_id` để nối sự kiện.
- `event` ổn định cho machine query; `message` cho con người.
- Error type và stack trace có kiểm soát, không kèm secret payload.

## 3. Logging driver, rotation và retention

Docker Engine dùng **logging driver** — backend nhận stdout/stderr của Container — để lưu hoặc chuyển log. Driver và option phụ thuộc môi trường.

```yaml
services:
  api:
    image: example/api:1.0
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
```

| Đường dẫn | Owner | Ý nghĩa |
|---|---|---|
| `services.api.logging.driver` | Service runtime config | Chọn logging driver cho Container mới. |
| `services.api.logging.options.max-size` | Driver `json-file` | Ngưỡng kích thước mỗi file trước rotation. |
| `services.api.logging.options.max-file` | Driver `json-file` | Số file rotated giữ local theo driver semantics. |

**Rotation** — chuyển file hiện tại sang file cũ và bắt đầu file mới để giới hạn tăng trưởng từng file. **Retention** — chính sách giữ log bao lâu hoặc bao nhiêu dữ liệu trước khi xóa. Rotation local không tự tạo retention tập trung, backup hoặc search dài hạn.

Thay Compose logging config thường cần recreate Container để effective HostConfig đổi; restart object cũ không bảo đảm áp dụng.

## 4. Kiểm tra effective logging configuration

```bash
docker info --format '{{json .LoggingDriver}}'
docker container inspect api --format '{{json .HostConfig.LogConfig}}'
docker system df
```

- Lệnh đầu xem driver mặc định của daemon, không nhất thiết là driver của mọi Container.
- Lệnh thứ hai xem driver/options thực tế của `api`, gồm override từ Compose.
- `docker system df` quan sát disk usage Docker ở mức tổng quan nhưng không thay kiểm tra filesystem host hoặc backend log bên ngoài.

Trước khi recreate, Container có LogConfig cũ. Sau `docker compose up -d`, inspect phải được đọc lại để xác nhận config mới. Nếu driver không hỗ trợ `docker logs`, command có thể không cung cấp output như mong đợi; phải dùng backend tương ứng.

## 5. Logs, metrics và traces trả lời câu hỏi khác nhau

| Tín hiệu | Phù hợp để trả lời |
|---|---|
| Logs | Sự kiện cụ thể nào xảy ra và context là gì? |
| Metrics | Tỷ lệ lỗi, latency, CPU hoặc queue depth thay đổi ra sao? |
| Traces | Request chậm ở Service/span nào? |
| Container metadata | Image, restart count, mounts và network configuration nào đang áp dụng? |

**Span** — một đoạn công việc trong trace, có thời gian bắt đầu/kết thúc và quan hệ cha-con với span khác. Một dashboard CPU không giải thích exception cụ thể; một log đơn lẻ không cho biết lỗi xảy ra ở 0,1% hay 80% request.

Observability tốt kết nối tín hiệu bằng timestamp, Service identity, version, environment và correlation/trace ID. Đồng hồ giữa các host sai lệch có thể làm timeline điều tra khó ghép, nên time synchronization cũng là dependency vận hành.

## 6. Từ câu hỏi tới bằng chứng

| Câu hỏi điều tra | Tín hiệu ưu tiên | Bổ sung |
|---|---|---|
| “API bắt đầu lỗi từ khi nào?” | Error-rate metric theo thời gian | Deploy event và logs quanh mốc tăng |
| “Request `req-42` chậm ở đâu?” | Distributed trace | Logs có cùng trace/request ID |
| “Container có restart vì OOM?” | Container metadata | Host/kernel log và memory metric |
| “Disk đầy do log?” | Host disk metric và driver files/backend | Log rate, rotation config, retention |

Việc chọn tín hiệu theo câu hỏi giảm noise và tránh thói quen “grep mọi thứ” mà không có hypothesis.

## 7. Quan niệm dễ sai

### “Có `docker logs` là đủ observability.”

- **Phân loại:** Sai.
- **Vì sao nghe hợp lý:** Command cho thấy application event ngay tại terminal.
- **Lỗi kỹ thuật:** Log có thể thiếu lịch sử tập trung, metric xu hướng, trace, host signal và deploy context.
- **Cách nói tốt hơn:** `docker logs` là công cụ quan sát cục bộ; observability cần nhiều tín hiệu liên kết.

### “Ghi mọi thứ sẽ giúp debug tốt hơn.”

- **Phân loại:** Sai.
- **Lỗi kỹ thuật:** Noise, chi phí lưu trữ và rò rỉ dữ liệu tăng; tín hiệu quan trọng bị chìm.
- **Cách nói tốt hơn:** Chọn event, level, field và retention theo mục tiêu vận hành.

### “Có rotation nghĩa log đã được lưu an toàn.”

- **Phân loại:** Sai do nhầm rotation với retention/centralization.
- **Lỗi kỹ thuật:** File cũ vẫn có thể bị xóa theo giới hạn local và mất cùng host.
- **Kiểm chứng:** Xác định driver, backend, retention policy và thử truy vấn log của incident cũ.

## 8. Tự kiểm tra mental model

1. Vì sao driver mặc định của daemon chưa chứng minh Container `api` đang dùng driver đó?
2. Rotation và retention khác nhau ở câu hỏi vận hành nào?
3. Muốn biết lỗi ảnh hưởng 1% hay 80% request, log đơn lẻ còn thiếu tín hiệu gì?
4. Vì sao correlation ID hữu ích nhưng không thay distributed trace?

## 9. Tóm tắt

1. Stdout/stderr tạo log stream phù hợp với runtime Container.
2. Logging driver, rotation và retention phải được cấu hình và kiểm chứng có chủ đích.
3. Structured logging cần schema ổn định, context đủ và redaction.
4. Kết hợp logs, metrics, traces và metadata để chẩn đoán bằng bằng chứng.

## Tài liệu tham khảo

- Docker Docs, [View container logs](https://docs.docker.com/engine/logging/)
- Docker Docs, [Configure logging drivers](https://docs.docker.com/engine/logging/configure/)
- Docker Docs, [Configure logging drivers for Compose](https://docs.docker.com/compose/how-tos/logging/)

[← 5. Health và restart](05-health-va-restart.md) · [Mục lục Production](README.md) · [7. Production checklist →](07-production-checklist.md)
