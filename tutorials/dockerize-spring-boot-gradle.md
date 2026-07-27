# Tutorial — Dockerize Spring Boot với Gradle

> **Mục tiêu:** Build source Spring Boot bằng Gradle trong JDK stage, chuyển executable JAR sang runtime stage, chạy non-root và kiểm chứng bằng HTTP.

> **Loại:** Tutorial · **Cấp độ:** Beginner → Intermediate · **Thời gian:** Khoảng 45 phút<br>
> **Lý thuyết liên quan:** [Dockerfile cho Java](../learning-path/04-dockerfile/06-dockerfile-cho-java.md)

[Mục lục Part 04](../learning-path/04-dockerfile/README.md) · [Tutorial Maven →](dockerize-spring-boot-maven.md)

---

## Kết quả cuối

```text
Source + Gradle Wrapper
-> JDK build stage
-> /workspace/app.jar
-> Java runtime stage
-> Image spring-gradle-demo:1.0
-> Container trả HTTP trên localhost:8080
```

## Điều kiện trước khi bắt đầu

- Docker Engine/Desktop đang chạy.
- Project Spring Boot dùng Gradle Wrapper: có `gradlew`, `gradlew.bat`, `gradle/wrapper/`.
- Project build được bằng Java 21, hoặc bạn thay Image/tag và toolchain cho đúng version.
- Ứng dụng lắng nghe port `8080` và có endpoint kiểm thử, ví dụ `/actuator/health` hoặc `/`.

Ví dụ tree:

```text
spring-app/
├── build.gradle
├── settings.gradle
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
└── src/
```

## Bước 1 — Đặt tên JAR ổn định

Trong `build.gradle` Groovy DSL:

```groovy
tasks.named('bootJar') {
    archiveFileName = 'application.jar'
}
```

Với Kotlin DSL `build.gradle.kts`:

```kotlin
tasks.named<org.springframework.boot.gradle.tasks.bundling.BootJar>("bootJar") {
    archiveFileName.set("application.jar")
}
```

Mục tiêu là output luôn tại `build/libs/application.jar`. Tên ổn định giúp Dockerfile không dùng `build/libs/*.jar`, vì wildcard có thể match nhiều artifact như plain JAR và executable JAR.

Kiểm tra ngoài Docker nếu máy có JDK:

```powershell
.\gradlew.bat bootJar
Get-ChildItem .\build\libs\
```

Bash/macOS/Linux:

```bash
./gradlew bootJar
ls -la build/libs/
```

Quan sát file `application.jar`. Nếu không có, kiểm tra plugin Spring Boot và cấu hình task.

## Bước 2 — Tạo `.dockerignore`

```text
.git
.gradle
.idea
.vscode
build
*.log
.env
```

Build thực hiện bên trong Docker nên local `build/` và `.gradle/` không cần đi vào context. `.env` bị loại để giảm nguy cơ gửi credential/config local vào builder; runtime config sẽ được truyền khi chạy.

## Bước 3 — Tạo Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk AS build
WORKDIR /workspace

COPY gradlew settings.gradle build.gradle ./
COPY gradle/ gradle/
RUN chmod +x gradlew
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew dependencies --no-daemon

COPY src/ src/
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew bootJar --no-daemon \
    && cp build/libs/application.jar /workspace/app.jar

FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /workspace/app.jar app.jar
USER 10001:10001
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

Giải thích theo từng nhóm:

| Dòng | Tác dụng và điều cần chú ý |
|---|---|
| `# syntax=...` | Chọn Dockerfile frontend hỗ trợ cache mount. |
| `FROM ...-jdk AS build` | Builder cần JDK để Gradle chạy, compile, test/process và package. |
| `WORKDIR /workspace` | Tạo/chọn thư mục làm việc của build stage. |
| `COPY gradlew ... ./` | Source thuộc context; destination `./` resolve thành `/workspace/`. |
| `COPY gradle/ gradle/` | Chuyển Gradle Wrapper files; destination là `/workspace/gradle/`. |
| `chmod +x gradlew` | Bảo đảm script có execute bit trong Linux stage. |
| cache mount | Tái sử dụng cache Gradle ở builder, không copy cache sang runtime Image. |
| `COPY src/ src/` | Chỉ source đổi thường xuyên được copy sau dependency step. |
| `bootJar` | Tạo Spring Boot executable JAR. Không dùng `-x test` mặc định để tutorial không âm thầm bỏ test. |
| `cp ... app.jar` | Chuẩn hóa artifact thành `/workspace/app.jar`. |
| `FROM ...-jre` | Runtime stage mới chỉ cần Java runtime cho JAR. |
| `COPY --from=build ...` | Source ở build-stage filesystem; destination resolve thành `/app/app.jar` trong runtime stage. |
| `USER 10001:10001` | Process runtime không chạy root; app không được ghi vào path thiếu permission. |
| `EXPOSE 8080` | Metadata port, không publish ra host. |
| exec `ENTRYPOINT` | JVM là process chính và nhận signal trực tiếp. |

> [!NOTE]
> `./gradlew dependencies` không phải project nào cũng có cùng hành vi cache tối ưu. Nếu task/plugin của project không resolve đủ dependency, có thể dùng task phù hợp hơn hoặc để `bootJar` thực hiện với cache mount. Nguyên tắc cần giữ là copy file dependency ổn định trước source khi điều đó đúng với build graph.

## Bước 4 — Build Image

PowerShell và Bash đều dùng được lệnh một dòng:

```bash
docker build --progress=plain --tag spring-gradle-demo:1.0 .
```

Quan sát:

- Context không chứa local `build/`.
- Stage `build` chạy Gradle và tạo `application.jar`.
- Stage `runtime` chỉ copy `/workspace/app.jar`.
- Output cuối gắn Tag `spring-gradle-demo:1.0`.

Nếu lỗi `gradlew: not found` hoặc ký tự `^M`, kiểm tra line ending CRLF. Có thể cấu hình Git giữ `gradlew` với LF.

## Bước 5 — Inspect Image

```bash
docker image inspect spring-gradle-demo:1.0 --format 'User={{.Config.User}} WorkDir={{.Config.WorkingDir}} Entrypoint={{json .Config.Entrypoint}}'
docker image history spring-gradle-demo:1.0
```

Kết quả mong đợi:

- User là `10001:10001`.
- Working directory là `/app`.
- Entrypoint chứa `java`, `-jar`, `/app/app.jar`.
- History final Image không có Gradle source-copy instruction từ stage build như nội dung runtime.

## Bước 6 — Chạy Container

```bash
docker run --detach --name spring-gradle-demo --publish 8080:8080 spring-gradle-demo:1.0
```

| Token | Ý nghĩa |
|---|---|
| `--detach` | Chạy nền và in Container ID. |
| `--name spring-gradle-demo` | Đặt tên Container để inspect/log/cleanup. |
| `--publish 8080:8080` | Host port `8080` → Container port `8080`. |
| `spring-gradle-demo:1.0` | Image reference. |

## Bước 7 — Quan sát và xác minh

```bash
docker container ps --filter name=spring-gradle-demo
docker logs spring-gradle-demo
curl http://localhost:8080/actuator/health
```

Nếu project không có Actuator:

```bash
curl http://localhost:8080/
```

Xác minh user và JAR bên trong Container:

```bash
docker exec spring-gradle-demo sh -c 'id && ls -l /app/app.jar && java -version'
```

Base Image có thể có `sh` nhưng không bảo đảm có `bash`. Output cần cho thấy UID không phải `0`, JAR tồn tại và Java runtime đúng major version.

## Lỗi thường gặp

| Lỗi | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| JAR không tồn tại | Chưa cấu hình `bootJar` name hoặc task khác output | Kiểm tra `build/libs`, sửa `archiveFileName` |
| `UnsupportedClassVersionError` | Runtime Java thấp hơn bytecode target | Đồng bộ JDK/toolchain/runtime |
| Container exit code 1 | App thiếu config/dependency hoặc entrypoint sai | Đọc `docker logs`, inspect command |
| Không truy cập được port | App dùng port khác hoặc bind sai | Kiểm tra log, `server.port`, mapping |
| Permission denied | App ghi vào `/app` dưới UID 10001 | Tạo/chown writable directory hoặc mount phù hợp |

## Cleanup

> [!WARNING]
> Các lệnh sau xóa Container và Image demo. Không dùng tên này nếu bạn đã gắn dữ liệu cần giữ vào Container khác cùng tên.

```bash
docker rm --force spring-gradle-demo
docker image rm spring-gradle-demo:1.0
```

Kiểm tra cleanup:

```bash
docker container inspect spring-gradle-demo
docker image inspect spring-gradle-demo:1.0
```

Hai lệnh phải báo object không tồn tại.

## Hoàn thành khi

- Image build thành công từ source bằng Gradle Wrapper.
- Artifact runtime là `/app/app.jar` với tên ổn định.
- Container chạy non-root và endpoint trả response mong đợi.
- Bạn giải thích được source/destination của cả hai loại `COPY`.
- Container và Image demo đã được dọn.

[Mục lục Part 04](../learning-path/04-dockerfile/README.md) · [Tutorial Maven →](dockerize-spring-boot-maven.md)
