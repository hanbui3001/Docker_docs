# 4. Tag, digest và versioning

> **Tóm tắt một câu:** Tag là con trỏ dễ đọc có thể di chuyển; digest là định danh theo content không đổi, nên delivery an toàn thường dùng tag để con người hiểu và digest để máy khóa artifact.

> **Loại:** Explanation · **Cấp độ:** Beginner → Intermediate · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Image digests](https://docs.docker.com/dhi/core-concepts/digests/)

[← 3. Login, tag, push và pull](03-login-tag-push-pull.md) · [Mục lục Registry & Delivery](README.md) · [5. Multi-platform image →](05-multi-platform-image.md)

---

## 1. Hai kiểu định danh

```text
registry.example.com/team/api:1.4.2
registry.example.com/team/api@sha256:abc123...
```

Tag `1.4.2` là tên do publisher quản lý. Digest được tính từ bytes của manifest/index tương ứng. Cùng content cho cùng digest theo thuật toán; đổi content làm digest đổi.

**Mutable tag** là tag có thể được cập nhật để trỏ sang artifact khác. **Immutable digest** là định danh content; Registry có thể áp policy không cho sửa tag, nhưng bản chất tag vẫn là lớp tên còn digest gắn với object.

## 2. Vì sao tag vẫn cần thiết?

Digest khó đọc và không tự nói version nghiệp vụ. Tag hỗ trợ:

- Semantic version như `1.4.2`.
- Channel như `stable`, `edge`.
- Liên kết commit như `git-a1b2c3d`.
- Môi trường tạm như `pr-128`.

Nhưng mỗi loại tag có độ ổn định khác nhau. `1.4.2` thường được kỳ vọng không di chuyển; `stable` chủ đích di chuyển; `latest` chỉ là tên mặc định.

## 3. Versioning policy thực tế

Một lần build có thể phát hành nhiều tag cùng trỏ một digest:

```text
api:1.4.2       ┐
api:1.4         ├──> sha256:abc...
api:stable      ┘
```

Khi phát hành `1.4.3`, `stable` và có thể `1.4` di chuyển, còn `1.4.2` nên giữ nguyên nếu policy xem version patch là immutable.

> [!IMPORTANT]
> Không build lại riêng cho từng môi trường. Tốt hơn là build một artifact, promote cùng digest qua test/staging/production và cung cấp runtime configuration riêng.

## 4. Pinning và update

**Pinning** là khóa dependency/deployment vào digest cụ thể để kết quả resolve không thay đổi ngoài ý muốn. Đổi lại, hệ thống không tự nhận security update khi tag di chuyển; cần quy trình chủ động cập nhật digest.

Tag-only dễ nhận update nhưng giảm reproducibility. Digest-only chính xác nhưng khó đọc. Cách kết hợp:

```text
api:1.4.2@sha256:abc123...
```

## 5. Digest nào?

Với multi-platform image, tag có thể trỏ tới image index digest; index lại trỏ tới manifest digest cho `linux/amd64`, `linux/arm64`, v.v. Vì thế hai máy khác platform có thể tải layer khác nhưng cùng resolve qua một index reference.

Khi kiểm chứng cần ghi rõ đang so sánh index digest hay platform-specific manifest digest. Chapter tiếp theo đi sâu vào cấu trúc này.

## 6. Quan niệm dễ gây hiểu nhầm

### “Tag version có số nên tự động immutable.”

Registry vẫn có thể cho phép push đè `1.4.2`. Immutability cần policy và kỷ luật phát hành.

### “Digest bảo đảm Image an toàn.”

Digest bảo đảm identity/integrity của content đã chọn, không chứng minh content không có lỗ hổng hay đến từ publisher đáng tin. Cần signature, provenance, scanning và access control tùy threat model.

### “Dùng `latest` luôn lấy Image mới nhất.”

Chỉ khi publisher cập nhật tag đó và client thực sự pull/resolve lại. Local cache và deployment pull policy cũng ảnh hưởng.

## 7. Tự kiểm tra

1. Vì sao rollback bằng digest đáng tin hơn rollback chỉ bằng `stable`?
2. Tag immutable là đặc tính kỹ thuật tự nhiên hay policy Registry?
3. Pin digest tạo ra trách nhiệm cập nhật nào?

## 8. Tóm tắt

Tag phục vụ giao tiếp và release channel; digest phục vụ identity và reproducibility. Chính sách tốt nêu rõ tag nào được di chuyển, tag nào bất biến và digest nào được deploy.

## Tài liệu tham khảo

- Docker Docs, [Image digests](https://docs.docker.com/dhi/core-concepts/digests/)
- OCI, [Image Manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md)

[← 3. Login, tag, push và pull](03-login-tag-push-pull.md) · [Mục lục Registry & Delivery](README.md) · [5. Multi-platform image →](05-multi-platform-image.md)
