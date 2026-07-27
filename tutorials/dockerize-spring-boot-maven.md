# Tutorial — Dockerize Spring Boot với Maven

> **Mục tiêu:** Dùng Maven Wrapper trong JDK stage để package Spring Boot executable JAR, sau đó chạy artifact trong Java runtime Image tối giản hơn.

> **Loại:** Tutorial · **Cấp độ:** Beginner → Intermediate · **Thời gian:** Khoảng 45 phút<br>
> **Lý thuyết liên quan:** [Dockerfile cho Java](../learning-path/04-dockerfile/06-dockerfile-cho-java.md)

[← Tutorial Gradle](dockerize-spring-boot-gradle.md) · [Mục lục Part 04](../learning-path/04-dockerfile/README.md)

---

## Điều kiện trước khi bắt đầu

- Docker Engine/Desktop đang chạy.
- Project Spring Boot có Maven Wrapper: `mvnw`, `mvnw.cmd`, `.mvn/wrapper/`.
- Project và runtime dùng Java version tương thích; ví dụ này dùng Java 21.
- Ứng dụng lắng nghe port `8080` và có endpoint kiểm thử.

```text
spring-app/
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .mvn/
│   └── wrapper/
└── src/
```

## Bước 1 — Đặt final name ổn định

Trong `pom.xml`, bên trong `<build>`:

```xml
<build>
    <finalName>application</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

Sau `package`, executable artifact mong đợi là `target/application.jar`. Spring Boot plugin cần cấu hình/repackage đúng theo project parent/plugin management.

Kiểm tra ngoài Docker nếu máy có JDK:

```powershell
.\mvnw.cmd package
Get-ChildItem .\target\
```

Bash/macOS/Linux:

```bash
./mvnw package
ls -la target/
```

Không thêm `-DskipTests` ở bước đầu. Tutorial cần chứng minh package đi qua test theo lifecycle project.

## Bước 2 — Tạo `.dockerignore`

```text
.git
.idea
.vscode
.mvn/timing.properties
target
*.log
.env
```

Không ignore toàn bộ `.mvn/` vì Maven Wrapper cần `.mvn/wrapper`. Local `target/` bị loại vì artifact sẽ được build lại trong stage.

## Bước 3 — Tạo Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk AS build
WORKDIR /workspace

COPY mvnw pom.xml ./
COPY .mvn/ .mvn/
RUN chmod +x mvnw
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw dependency:go-offline --batch-mode

COPY src/ src/
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw package --batch-mode \
    && cp target/application.jar /workspace/app.jar

FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /workspace/app.jar app.jar
USER 10001:10001
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

Giải thích:

| Dòng/nhóm | Tác dụng |
|---|---|
| `FROM ...-jdk AS build` | Maven chạy trên Java và cần compiler/toolchain để compile/package. |
| `COPY mvnw pom.xml ./` | Copy Wrapper script và dependency descriptor trước source để tăng cơ hội cache. |
| `COPY .mvn/ .mvn/` | Source context `.mvn/`; destination `/workspace/.mvn/`. |
| `chmod +x mvnw` | Bảo đảm Linux có thể chạy Wrapper script. |
| cache `/root/.m2` | Builder tái sử dụng Maven local repository giữa build. |
| `dependency:go-offline` | Cố gắng resolve plugin/dependency sớm; project phức tạp vẫn có thể cần artifact ở phase sau. |
| `package --batch-mode` | Chạy lifecycle đến package; `--batch-mode` giảm interactive/color noise trong CI. |
| `cp target/application.jar ...` | Chọn artifact xác định và chuẩn hóa path giao giữa stage. |
| runtime stage | Chỉ mang Java runtime và `/app/app.jar`, không mang Maven/source/`.m2`. |

Nếu pipeline đã chạy test ở job bắt buộc riêng, có thể chủ động dùng:

```dockerfile
RUN ./mvnw package --batch-mode -DskipTests
```

Nhưng phải ghi rõ build Image này không tự chạy test. `-DskipTests` không nên xuất hiện vô thức trong template.

## Bước 4 — Build Image

```bash
docker build --progress=plain --tag spring-maven-demo:1.0 .
```

Quan sát Maven test/package thành công và runtime stage copy đúng `/workspace/app.jar`.

Nếu `target/application.jar` không tồn tại:

1. Kiểm tra `<finalName>` có nằm đúng trong `<build>`.
2. Kiểm tra Spring Boot Maven Plugin có chạy `repackage`.
3. Xem danh sách file `target/` trong build log hoặc thêm lệnh debug tạm `RUN ls -la target`.
4. Không sửa thành `target/*.jar` trước khi hiểu vì sao có nhiều artifact.

## Bước 5 — Inspect Image

```bash
docker image inspect spring-maven-demo:1.0 --format 'User={{.Config.User}} WorkDir={{.Config.WorkingDir}} Entrypoint={{json .Config.Entrypoint}}'
docker image history spring-maven-demo:1.0
```

Image config cần cho thấy user non-root, `/app` và Java entrypoint. History của final output không nên cho thấy source/Maven repository được copy vào runtime stage.

## Bước 6 — Chạy và xác minh

```bash
docker run --detach --name spring-maven-demo --publish 8080:8080 spring-maven-demo:1.0
docker logs spring-maven-demo
curl http://localhost:8080/actuator/health
```

Nếu không có Actuator, gọi endpoint hiện có:

```bash
curl http://localhost:8080/
```

Kiểm tra runtime:

```bash
docker exec spring-maven-demo sh -c 'id && ls -l /app/app.jar && java -version'
```

Bạn cần thấy UID khác `0`, artifact tồn tại, Java major version phù hợp và HTTP response đúng.

## Bước 7 — Kiểm tra JAR

Stage runtime thường có tool `jar` hay không tùy distribution. Cách ổn định hơn là kiểm tra trong build stage:

```bash
docker build --target build --tag spring-maven-build:debug .
docker run --rm spring-maven-build:debug jar --list --file /workspace/app.jar
```

`--target build` xuất Image tại stage `build`; command sau override command mặc định và liệt kê JAR. Đây là debug artifact, không phải runtime Image production.

Cleanup debug Image:

```bash
docker image rm spring-maven-build:debug
```

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Hướng xử lý |
|---|---|---|
| Wrapper download thất bại | Network/proxy/certificate | Cấu hình Maven settings/proxy bằng cơ chế secret/config phù hợp |
| Multiple JARs | Original JAR và repackaged JAR cùng tồn tại | Dùng `finalName` và source path cụ thể |
| Test fail trong `package` | Code/test/config lỗi | Sửa test hoặc tách pipeline rõ ràng; không bỏ test vô thức |
| App không start | Thiếu runtime config hoặc JAR không executable | Đọc log, kiểm tra manifest/plugin repackage |
| Permission denied | Runtime ghi vào path không được cấp quyền | Chuẩn bị/chown directory hoặc mount đúng |

## Cleanup

> [!WARNING]
> Cleanup xóa Container demo và writable layer của nó. Chỉ chạy khi không có dữ liệu cần giữ.

```bash
docker rm --force spring-maven-demo
docker image rm spring-maven-demo:1.0
```

Kiểm tra hai object không còn:

```bash
docker container inspect spring-maven-demo
docker image inspect spring-maven-demo:1.0
```

## Hoàn thành khi

- Maven Wrapper package executable JAR trong JDK stage.
- Runtime stage chỉ nhận artifact có tên ổn định.
- Image config và user được inspect đúng.
- HTTP endpoint trả kết quả mong đợi.
- Mọi Container/Image demo đã cleanup.

[← Tutorial Gradle](dockerize-spring-boot-gradle.md) · [Mục lục Part 04](../learning-path/04-dockerfile/README.md)
