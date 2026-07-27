# Part 04 — Dockerfile: từ source code đến Docker Image

> Dockerfile không chỉ là danh sách lệnh để “đóng gói ứng dụng”. Nó là bản mô tả có thứ tự để builder tạo filesystem và cấu hình mặc định của một Docker Image.

> **Loại:** Learning Path · **Cấp độ:** Beginner → Intermediate · **Thời gian:** Khoảng 4–6 giờ

## Bạn sẽ học được gì?

Sau Part này, bạn có thể:

- Phân biệt Dockerfile, build context, build step, stage, layer và Image.
- Đọc cú pháp `FROM`, `RUN`, `COPY`, `WORKDIR`, `ARG`, `ENV`, `USER`, `EXPOSE`, `ENTRYPOINT`, `CMD` và `HEALTHCHECK` theo đúng scope.
- Giải thích cache hoạt động ra sao và vì sao thứ tự instruction ảnh hưởng thời gian build.
- Thiết kế multi-stage build để tách môi trường build khỏi môi trường chạy.
- Mô tả đầy đủ luồng Java: source → bytecode → JAR → JVM → process.
- Viết Dockerfile Spring Boot bằng Gradle hoặc Maven có thể kiểm chứng và dọn dẹp.

## Điều kiện trước khi học

- Hiểu Image và Container trong [Part 01 — Foundations](../01-foundations/README.md).
- Biết chạy các lệnh Docker cơ bản như `docker build`, `docker run`, `docker logs` và `docker rm`.
- Có kiến thức Java/Spring Boot cơ bản nếu thực hành hai tutorial cuối Part.

## Lộ trình chapter

| Chapter | Trọng tâm | Kết quả |
|---|---|---|
| [1. Dockerfile là gì?](01-dockerfile-la-gi.md) | Mental model và build flow | Không nhầm Dockerfile với Image hoặc shell script |
| [2. Build context và `.dockerignore`](02-build-context-va-dockerignore.md) | Phạm vi file builder được đọc | Hiểu dấu `.` cuối `docker build` |
| [3. Các instruction cơ bản](03-cac-instruction-co-ban.md) | Cú pháp, scope và trạng thái | Đọc được Dockerfile phổ biến |
| [4. Layer và build cache](04-layer-va-build-cache.md) | Cache key và thứ tự instruction | Build nhanh hơn mà không hiểu sai layer |
| [5. Multi-stage build](05-multi-stage-build.md) | Tách build/runtime | Không mang tool build vào Image cuối |
| [6. Dockerfile cho Java](06-dockerfile-cho-java.md) | JAR, JDK, runtime và compatibility | Hiểu chính xác artifact Java đang chạy |
| [7. Bảo mật và tối ưu Image](07-bao-mat-va-toi-uu-image.md) | User, secret, base Image, size | Tối ưu có mục tiêu, không chạy theo mẹo |
| [8. Bức tranh build hoàn chỉnh](08-buc-tranh-build-hoan-chinh.md) | Kết nối toàn bộ luồng | Có checklist thiết kế Dockerfile mới |

## Thực hành và tra cứu

- [Tutorial: Dockerize Spring Boot với Gradle](../../tutorials/dockerize-spring-boot-gradle.md)
- [Tutorial: Dockerize Spring Boot với Maven](../../tutorials/dockerize-spring-boot-maven.md)
- [Dockerfile instruction reference](../../reference/dockerfile/instructions.md)
- [`docker build` options reference](../../reference/dockerfile/docker-build-options.md)

## Checklist hoàn thành

- [ ] Tôi giải thích được source và destination của `COPY app.jar app.jar` thuộc hai filesystem khác nhau.
- [ ] Tôi phân biệt được `RUN` với `CMD`, `ARG` với `ENV`, `EXPOSE` với publish port.
- [ ] Tôi biết một Dockerfile không bắt buộc phải có nhiều stage.
- [ ] Tôi giải thích được vì sao builder thường cần JDK nhưng runtime có thể chỉ cần Java runtime.
- [ ] Tôi biết cách kiểm chứng Image bằng `docker image history`, `docker image inspect` và chạy Container thử.

[1. Dockerfile là gì? →](01-dockerfile-la-gi.md)
