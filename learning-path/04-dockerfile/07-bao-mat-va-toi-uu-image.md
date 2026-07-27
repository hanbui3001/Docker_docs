# 7. Bảo mật và tối ưu Docker Image

> **Tóm tắt một câu:** Tối ưu Image là giảm nội dung, đặc quyền và biến động không cần thiết trong khi vẫn giữ khả năng vận hành, kiểm chứng và cập nhật.

> **Loại:** Explanation · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 40 phút<br>
> **Nguồn chính:** [Docker build best practices](https://docs.docker.com/build/building/best-practices/)

[← 6. Dockerfile cho Java](06-dockerfile-cho-java.md) · [Mục lục Part 04](README.md) · [8. Bức tranh build hoàn chỉnh →](08-buc-tranh-build-hoan-chinh.md)

---

## 1. Tối ưu cho mục tiêu nào?

“Image nhỏ” chỉ là một mục tiêu. Dockerfile production còn cần:

- Ít package/tool không cần thiết.
- Base Image được duy trì và cập nhật.
- Build có thể tái lập ở mức phù hợp.
- Process không chạy với quyền root nếu không cần.
- Secret không nằm trong layer/config/history.
- Artifact và version có thể truy vết.
- Health, log và shutdown hoạt động đúng.

Đánh đổi phải được nói rõ: Image cực nhỏ có thể thiếu certificate, timezone data hoặc tool chẩn đoán; Image có JDK lớn hơn nhưng hỗ trợ điều tra JVM tốt hơn.

## 2. Chọn base Image

Đánh giá:

- Nguồn/publisher có đáng tin và còn bảo trì không?
- OS/runtime version có phù hợp dependency không?
- Tag có quá trôi không; có cần pin Digest cho quy trình release không?
- Image có certificate, libc và tool runtime cần thiết không?
- Scanner và chính sách tổ chức hỗ trợ ra sao?

Pin Digest tăng tính xác định nội dung:

```dockerfile
FROM eclipse-temurin:21-jre@sha256:<digest>
```

Nhưng Digest pinning chuyển trách nhiệm cập nhật sang bạn; nếu không có automation theo dõi base mới, Image có thể đứng yên ở bản có lỗ hổng.

## 3. Chạy non-root

```dockerfile
WORKDIR /app
COPY --chown=10001:10001 app.jar app.jar
USER 10001:10001
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

`COPY --chown` đặt ownership khi copy trên Linux builder. `USER` đổi identity mặc định cho process. Trước khi đổi user, cần bảo đảm app đọc được JAR và ghi được đúng thư mục cần thiết.

Non-root giảm tác động của một số lỗ hổng nhưng không tạo sandbox tuyệt đối. Container vẫn cần giới hạn capability, filesystem write, network, resource và host mount ở runtime.

## 4. Không đưa secret vào Image

Sai:

```dockerfile
ARG TOKEN
RUN curl -H "Authorization: Bearer $TOKEN" https://private.example/artifact
ENV DB_PASSWORD=secret
```

Tốt hơn cho build secret:

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=repo_token \
    TOKEN="$(cat /run/secrets/repo_token)" \
    && curl -H "Authorization: Bearer ${TOKEN}" https://private.example/artifact
```

Build command:

```bash
docker build --secret id=repo_token,src=repo-token.txt --tag my-app:1.0 .
```

Secret mount chỉ xuất hiện trong build step, không được copy tự động vào output layer. Command vẫn phải tránh ghi token vào file/log.

## 5. Giảm nội dung không cần thiết

- Dùng `.dockerignore` để loại `.git`, IDE files, logs, credential và output thừa.
- Dùng multi-stage để final Image không chứa compiler/build tool/source.
- Chỉ install package cần thiết; dọn package metadata trong cùng `RUN` khi phù hợp.
- Copy artifact cụ thể thay vì toàn project.
- Không giữ shell/debug tool chỉ vì tiện nếu production policy không cần; cân nhắc debug Image riêng.

Kích thước cần đo thay vì đoán:

```bash
docker image ls my-app
docker image history my-app:1.0
```

## 6. Version, label và provenance

`LABEL` thêm metadata:

```dockerfile
LABEL org.opencontainers.image.title="sample-api" \
      org.opencontainers.image.version="1.0.0" \
      org.opencontainers.image.source="https://example.invalid/repository"
```

Label không chứng minh nội dung trung thực; pipeline cần đặt giá trị từ nguồn release đáng tin và có thể bổ sung SBOM, signature, provenance attestation.

**SBOM (Software Bill of Materials)** là danh sách thành phần phần mềm trong artifact/Image. Nó hỗ trợ truy vết dependency và đánh giá lỗ hổng, nhưng scanner result vẫn phụ thuộc database và ngữ cảnh khai thác.

## 7. Process và signal

Ưu tiên exec form:

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

JVM trở thành PID 1 trong Container và nhận signal dừng trực tiếp. Nếu dùng startup script, script nên dùng `exec java ...` để thay shell bằng JVM.

Ứng dụng vẫn cần graceful shutdown — xử lý tín hiệu để ngừng nhận request, hoàn thành công việc đang chạy và giải phóng resource trong timeout.

## 8. Healthcheck có phải luôn nên đặt trong Dockerfile?

Không tuyệt đối. Đặt trong Dockerfile cung cấp default dùng lại; nền tảng deploy có thể muốn health probe khác theo environment. Nếu command healthcheck phụ thuộc `curl`/`wget`, thêm tool chỉ cho probe cũng làm tăng Image và bề mặt tấn công.

Có thể dùng Java/native probe nhẹ, endpoint TCP hoặc định nghĩa ở Compose/orchestrator. Điều quan trọng là probe đo đúng readiness/liveness cần thiết, không chỉ kiểm tra process tồn tại.

## 9. Quan niệm dễ gây hiểu nhầm

### 9.1 “Alpine luôn nhỏ và tốt nhất.”

Sai: musl libc, package availability, DNS/native dependency và khả năng debug có thể tạo trade-off. Chọn theo compatibility và vận hành, không theo tên distribution.

### 9.2 “Non-root làm Container an toàn hoàn toàn.”

Sai: đây là một lớp giảm đặc quyền, không thay thế patching, secret, capability, mount và network controls.

### 9.3 “Image càng ít MB thì chạy càng nhanh.”

Sai: size ảnh hưởng pull/storage/start cold path trong một số trường hợp; hiệu năng ứng dụng phụ thuộc CPU, memory, JVM, I/O và code.

### 9.4 “Xóa secret ở dòng sau là đủ.”

Sai: secret có thể còn trong layer/cache/history trước. Không đưa secret vào layer ngay từ đầu.

## 10. Checklist review Dockerfile

- Base Image có nguồn và version rõ ràng.
- Context nhỏ, `.dockerignore` hợp lý.
- Build/runtime được tách khi có lợi.
- Artifact được chọn xác định, không wildcard mơ hồ.
- Secret dùng secret mount/runtime secret.
- Process chạy user phù hợp và filesystem permission đúng.
- Exec form/signal/shutdown đã được kiểm tra.
- Image được scan, truy vết và cập nhật định kỳ.

## 11. Tóm tắt

- Tối ưu cân bằng size, attack surface, khả năng vận hành và tái lập.
- Multi-stage, non-root và secret mount là công cụ; không phải bảo đảm tuyệt đối.
- Base pinning cần đi cùng quy trình cập nhật.
- Đo Image, kiểm tra process và review runtime policy thay vì áp dụng mẹo máy móc.

## Tài liệu tham khảo

- Docker Docs, [Build best practices](https://docs.docker.com/build/building/best-practices/)
- Docker Docs, [Build secrets](https://docs.docker.com/build/building/secrets/)
- OCI, [Image annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md)

[← 6. Dockerfile cho Java](06-dockerfile-cho-java.md) · [Mục lục Part 04](README.md) · [8. Bức tranh build hoàn chỉnh →](08-buc-tranh-build-hoan-chinh.md)
