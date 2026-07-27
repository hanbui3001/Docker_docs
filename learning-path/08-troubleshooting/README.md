# Part 08. Docker Troubleshooting

> **Mục tiêu:** Chẩn đoán lỗi Docker bằng trạng thái quan sát được thay vì thử lệnh ngẫu nhiên.

> **Loại:** Learning Path · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 5-7 giờ  
> **Điều kiện:** Đã học Part 01-07 hoặc có kinh nghiệm chạy Container, Volume, Network, Dockerfile và Compose.

## Phương pháp chung

Mỗi playbook dùng năm bước:

```text
Symptom
-> Hypothesis
-> Evidence
-> Correction
-> Verification
```

**Symptom** là điều quan sát được, không phải kết luận. “API không truy cập được” là symptom; “Docker network hỏng” mới chỉ là hypothesis. Mỗi hypothesis cần command hoặc output để xác nhận hay loại trừ.

## Thứ tự học

| Chapter | Trọng tâm |
|---|---|
| [1. Phương pháp chẩn đoán](01-phuong-phap-chan-doan.md) | Điều tra theo layer và giữ bằng chứng |
| [2. Lỗi build Image](02-loi-build-image.md) | Context, Dockerfile, cache, platform và artifact |
| [3. Container không khởi động](03-container-khong-khoi-dong.md) | Exit code, command, log, OOM và restart loop |
| [4. Lỗi port và network](04-loi-port-va-network.md) | Listen address, publish, DNS và network membership |
| [5. Lỗi storage và permission](05-loi-storage-va-permission.md) | Mount scope, ownership và dữ liệu “biến mất” |
| [6. Lỗi Docker Compose](06-loi-docker-compose.md) | Model đã resolve, project state và dependency |
| [7. Disk usage và cleanup](07-disk-usage-va-cleanup.md) | Xác định object chiếm chỗ trước khi prune |
| [8. Diagnostic playbook](08-diagnostic-playbook.md) | Checklist điều tra end-to-end |

## Nguyên tắc an toàn

- Không chạy `prune` hoặc xóa Volume trước khi biết object nào chứa dữ liệu cần giữ.
- Không dùng `--force`, `privileged` hoặc đổi permission toàn cục chỉ để “thử xem”.
- Lưu command, timestamp, Container ID, Image digest và output quan trọng trước khi recreate.
- Chỉ thay một biến đáng kể mỗi lần để biết điều gì thực sự sửa lỗi.

[← Part 07. Production](../07-production/README.md) · [Mục lục sách](../../README.md) · [1. Phương pháp chẩn đoán →](01-phuong-phap-chan-doan.md)
