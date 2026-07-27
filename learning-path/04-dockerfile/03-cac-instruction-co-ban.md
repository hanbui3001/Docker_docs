# 3. Các instruction Dockerfile cơ bản

> **Tóm tắt một câu:** Mỗi instruction có parser, scope và thời điểm tác động riêng; đọc đúng Dockerfile cần biết giá trị thuộc build stage, Image configuration hay process runtime.

> **Loại:** Explanation · **Cấp độ:** Beginner → Intermediate · **Thời gian:** Khoảng 50 phút<br>
> **Nguồn chính:** [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

[← 2. Build context và `.dockerignore`](02-build-context-va-dockerignore.md) · [Mục lục Part 04](README.md) · [4. Layer và build cache →](04-layer-va-build-cache.md)

---

## 1. Bản đồ instruction

| Mục đích | Instruction thường dùng |
|---|---|
| Chọn nền/mở stage | `FROM` |
| Thay đổi filesystem khi build | `RUN`, `COPY`, `ADD` |
| Biến build/runtime | `ARG`, `ENV` |
| Cấu hình thư mục và quyền | `WORKDIR`, `USER` |
| Mô tả network/health | `EXPOSE`, `HEALTHCHECK` |
| Chọn process khởi động | `ENTRYPOINT`, `CMD` |
| Chuyển artifact giữa stage | `COPY --from` |

## 2. `FROM`: bắt đầu một stage

Cú pháp:

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>|@<digest>] [AS <name>]
```

Ví dụ:

```dockerfile
FROM eclipse-temurin:21-jdk AS build
```

| Token | Ý nghĩa |
|---|---|
| `FROM` | Bắt đầu stage mới. |
| `eclipse-temurin` | Repository của base Image. |
| `21-jdk` | Tag; tên tham chiếu có thể được publisher cập nhật. |
| `AS build` | Đặt tên stage là `build` để `COPY --from=build` tham chiếu ổn định. |

Trước `FROM`, chưa có filesystem stage. Sau `FROM`, stage bắt đầu từ filesystem và configuration của base Image. `FROM scratch` là trường hợp đặc biệt bắt đầu từ filesystem trống.

## 3. `WORKDIR`: đặt thư mục làm việc

```dockerfile
WORKDIR /app
```

`WORKDIR` không chỉ tương đương `cd` tạm thời. Nó đặt working directory cho instruction sau như `RUN`, `COPY`, `ADD`, `CMD`, `ENTRYPOINT`, đồng thời ghi mặc định vào Image configuration.

```dockerfile
WORKDIR /app
WORKDIR data
```

`data` là path tương đối nên dòng hai resolve thành `/app/data`. Builder tạo thư mục nếu chưa tồn tại.

> [!TIP]
> Dùng absolute path cho `WORKDIR` đầu tiên để Dockerfile không phụ thuộc working directory được thừa kế từ base Image.

## 4. `COPY`: chuyển file giữa hai scope

Cú pháp phổ biến:

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--from=<stage>] <src>... <dest>
```

Ví dụ bắt buộc phải đọc theo scope:

```dockerfile
WORKDIR /app
COPY app.jar app.jar
```

Cây cú pháp:

```text
COPY
├── source: app.jar
│   └── scope: build context
└── destination: app.jar
    └── scope: current stage filesystem
        └── resolved: /app/app.jar
```

| Token | Scope | Giá trị đã resolve |
|---|---|---|
| `COPY` | Builder | Tạo filesystem change trong stage hiện tại. |
| `app.jar` thứ nhất | Build context | File `<context-root>/app.jar`. |
| `app.jar` thứ hai | Image filesystem | `/app/app.jar` do destination tương đối với `WORKDIR`. |

Trước `COPY`, source ở bên ngoài Image; destination có thể chưa tồn tại. Sau `COPY`, nội dung source xuất hiện tại `/app/app.jar` trong stage. Hai tên giống nhau nhưng khác filesystem, khác vai trò và có thể khác path tuyệt đối.

Nếu destination kết thúc bằng `/`, Docker coi đó là directory:

```dockerfile
COPY app.jar /app/
```

Kết quả vẫn là `/app/app.jar`. Ngược lại, `COPY app.jar /app` có thể tạo file tên `/app` nếu path đó chưa là directory; dấu `/` cuối làm ý định rõ hơn.

### `COPY` hay `ADD`?

`ADD` có thêm hành vi như giải nén tar local và hỗ trợ một số source URL/Git tùy builder. Với copy file thông thường, ưu tiên `COPY` vì hành vi hẹp và dễ đọc. “`ADD` mạnh hơn” không có nghĩa “luôn tốt hơn”.

## 5. `RUN`: thực thi tại build time

Shell form:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

Exec form:

```dockerfile
RUN ["/bin/sh", "-c", "apt-get update && apt-get install -y curl"]
```

`RUN` tạo một build-time process bên trong môi trường stage, rồi ghi thay đổi filesystem còn lại vào kết quả build. Process đó kết thúc trước khi Image được tạo; nó không chạy lại khi Container start.

Trong shell form, shell xử lý `&&`, biến và glob. Exec form truyền mảng argument trực tiếp cho executable đã nêu; không tự có shell expansion.

### `RUN` khác `CMD`

| `RUN` | `CMD` |
|---|---|
| Thực thi lúc build | Chỉ ghi command/argument mặc định cho runtime |
| Có thể thay đổi filesystem Image | Thường thay đổi Image configuration |
| Kết thúc trước khi build xong | Được dùng khi Container start |

## 6. `ARG` và `ENV`: cùng giống biến nhưng khác vòng đời

```dockerfile
ARG APP_VERSION=dev
ENV APP_VERSION=$APP_VERSION
```

| Đặc điểm | `ARG` | `ENV` |
|---|---|---|
| Scope chính | Build stage | Image configuration và process runtime |
| Truyền từ CLI | `--build-arg` | Thường truyền lúc run bằng `--env`; Dockerfile đặt default |
| Có trong environment Container mặc định | Không, trừ khi chuyển sang `ENV` | Có |
| Phù hợp secret | Không | Không |

`ARG` khai báo trước `FROM` có scope đặc biệt để nội suy trong `FROM`; muốn dùng lại trong stage thường phải khai báo `ARG` sau `FROM`.

> [!WARNING]
> Không đưa password/token vào `ARG` hoặc `ENV`. Giá trị có thể xuất hiện trong metadata, history, cache hoặc cấu hình Container. Dùng BuildKit secret mount cho secret lúc build và secret manager/runtime injection khi chạy.

## 7. `USER`, `EXPOSE` và `HEALTHCHECK`

### `USER`

```dockerfile
USER 10001:10001
```

Đặt user/group mặc định cho `RUN` tiếp theo và process runtime. Numeric ID tránh phụ thuộc việc tên user có resolve giống nhau hay không, nhưng Image vẫn cần permission filesystem phù hợp.

### `EXPOSE`

```dockerfile
EXPOSE 8080
```

Đây là metadata mô tả Container dự kiến lắng nghe port `8080`. Nó không publish port ra host. Publish xảy ra bằng `docker run --publish 8080:8080 ...` hoặc Compose `ports`.

### `HEALTHCHECK`

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:8080/actuator/health || exit 1
```

| Thành phần | Ý nghĩa |
|---|---|
| `--interval=30s` | Khoảng thời gian giữa các lần kiểm tra sau khi bắt đầu. |
| `--timeout=3s` | Một lần kiểm tra quá 3 giây bị coi thất bại. |
| `--retries=3` | Số thất bại liên tiếp để chuyển sang `unhealthy`. |
| `CMD ...` | Command chạy bên trong Container; tool như `wget` phải có trong Image. |

Healthcheck exit `0` nghĩa healthy, `1` nghĩa unhealthy, `2` được dành riêng. Healthcheck không tự restart Container trong Docker Engine thông thường; orchestrator hoặc policy bên ngoài mới quyết định phản ứng.

## 8. `ENTRYPOINT` và `CMD`

Mẫu exec form phổ biến:

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
CMD ["--spring.profiles.active=prod"]
```

Docker kết hợp chúng thành process argument mặc định:

```text
java -jar /app/app.jar --spring.profiles.active=prod
```

`ENTRYPOINT` thường giữ executable cốt lõi; `CMD` cung cấp argument mặc định dễ override:

```bash
docker run my-app:1.0 --spring.profiles.active=dev
```

Argument sau Image thay `CMD`, không tự thay `ENTRYPOINT`. `--entrypoint` mới thay entrypoint.

Exec form giữ ranh giới argument rõ và để process ứng dụng nhận signal trực tiếp. Shell form như `ENTRYPOINT java -jar app.jar` chạy qua shell, có thể ảnh hưởng signal handling và cách argument được nối.

## 9. Quan niệm dễ gây hiểu nhầm

### 9.1 “`RUN`, `CMD`, `ENTRYPOINT` đều là cách chạy command giống nhau.”

- `RUN` thuộc build time.
- `ENTRYPOINT` và `CMD` thuộc Image runtime configuration.
- `CMD` dễ bị argument sau Image thay thế; `ENTRYPOINT` thường giữ executable.

### 9.2 “`EXPOSE 8080` làm localhost:8080 truy cập được.”

- `EXPOSE` không tạo host-port mapping.
- Cần `--publish HOST:CONTAINER` và ứng dụng phải lắng nghe đúng interface/port.

### 9.3 “`ARG` an toàn cho secret vì không có trong Container environment.”

- Không có trong environment runtime không đồng nghĩa không để lại dấu vết build.
- Secret cần secret mount, không dùng variable thông thường.

## 10. Tự kiểm tra mental model

1. Trong `COPY app.jar app.jar`, hai path thuộc scope nào?
2. Vì sao `RUN java -version` không khiến Java chạy khi Container start?
3. Khi nào argument sau Image thay `CMD`, khi nào thay `ENTRYPOINT`?
4. `EXPOSE` khác `--publish` ở state nào được thay đổi?

## 11. Tóm tắt

- `FROM` mở stage; `COPY`/`RUN` tạo nội dung; instruction cấu hình đặt mặc định runtime.
- `ARG` phục vụ build, `ENV` tồn tại trong Image/runtime; cả hai không phải nơi giữ secret.
- `ENTRYPOINT` và `CMD` nên được thiết kế theo cách process nhận signal và argument rõ ràng.
- Cú pháp chỉ đúng khi đọc cùng parser, scope, path resolve và trạng thái trước/sau.

## Tài liệu tham khảo

- Docker Docs, [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- Docker Docs, [Build secrets](https://docs.docker.com/build/building/secrets/)

[← 2. Build context và `.dockerignore`](02-build-context-va-dockerignore.md) · [Mục lục Part 04](README.md) · [4. Layer và build cache →](04-layer-va-build-cache.md)
