# Compose file keys

> **Loại:** Reference · **Cấp độ:** Beginner đến Intermediate  
> **Mục đích:** Tra nhanh scope, type và ý nghĩa của các key Compose thường dùng.

[Mục lục Compose Reference](README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)

---

## 1. Top-level keys

| Key | Type chính | Ý nghĩa |
|---|---|---|
| `name` | Scalar | Đặt Compose project name trong model. |
| `services` | Mapping | Khai báo các Service workload. |
| `networks` | Mapping | Khai báo Network resource dùng trong project. |
| `volumes` | Mapping | Khai báo named Volume resource. |
| `configs` | Mapping | Khai báo configuration data có thể mount/cấp cho Service. |
| `secrets` | Mapping | Khai báo secret data theo cơ chế Compose/platform hỗ trợ. |

Compose hiện đại không cần top-level `version`. Dùng Specification hiện tại và kiểm tra feature support của Compose CLI đang cài.

## 2. Service keys thường dùng

Scope mẫu: `services.<service-name>.<key>`.

| Key | Type | Tác dụng chính |
|---|---|---|
| `image` | Scalar | Image reference để tạo Container; cũng có thể đặt tên kết quả build. |
| `build` | Scalar hoặc mapping | Build context hoặc build configuration. |
| `command` | Scalar hoặc sequence | Ghi đè command mặc định của Image. |
| `entrypoint` | Scalar hoặc sequence | Ghi đè entrypoint mặc định của Image. |
| `environment` | Mapping hoặc sequence | Đặt environment cho Container process. |
| `env_file` | Scalar/sequence hoặc long form | Nạp environment từ file cho Service. |
| `ports` | Sequence | Publish container port ra host. |
| `expose` | Sequence | Mô tả port nội bộ, không publish host port. |
| `volumes` | Sequence | Mount Volume, bind path hoặc mount type khác vào Container. |
| `networks` | Sequence hoặc mapping | Gắn Service vào Network. |
| `depends_on` | Sequence hoặc mapping | Khai báo startup dependency. |
| `healthcheck` | Mapping | Ghi đè/định nghĩa health probe. |
| `restart` | Scalar | Runtime restart policy theo giá trị được hỗ trợ. |
| `working_dir` | Scalar | Ghi đè working directory runtime. |
| `user` | Scalar | User/UID:GID dùng chạy process. |
| `container_name` | Scalar | Ép tên Container; hạn chế scaling và project isolation. |
| `profiles` | Sequence | Chỉ kích hoạt Service trong profile tương ứng. |

## 3. `build`

### Short syntax

```yaml
services:
  backend:
    build: ./backend
```

`./backend` là build context. Dockerfile mặc định là `Dockerfile` trong context.

### Long syntax

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: runtime
      args:
        JAVA_VERSION: "21"
```

| Key | Ý nghĩa |
|---|---|
| `context` | Build context path/URL. |
| `dockerfile` | Dockerfile được chọn trong quan hệ với context. |
| `target` | Stage đích trong multi-stage Dockerfile. |
| `args` | Giá trị cho Dockerfile `ARG`; không tự là runtime env. |
| `cache_from` / `cache_to` | Nguồn/đích build cache theo builder hỗ trợ. |
| `secrets` | Secret mount dành cho build, khác runtime `secrets`. |

## 4. `ports`

### Short syntax

```text
[HOST_IP:]PUBLISHED:TARGET[/PROTOCOL]
```

```yaml
ports:
  - "127.0.0.1:8080:80/tcp"
```

| Field | Ví dụ | Scope |
|---|---|---|
| `HOST_IP` | `127.0.0.1` | Interface của host. |
| `PUBLISHED` | `8080` | Port trên host. |
| `TARGET` | `80` | Port trong Container. |
| `PROTOCOL` | `tcp` | TCP hoặc UDP theo hỗ trợ. |

### Long syntax

```yaml
ports:
  - name: http
    target: 80
    published: "8080"
    host_ip: 127.0.0.1
    protocol: tcp
```

Không dùng `ports` cho giao tiếp Service-to-Service nếu hai Service đã chung Network; dùng service name và target port.

## 5. `environment`, `env_file` và interpolation

### Mapping syntax

```yaml
environment:
  DB_HOST: database
  DB_PORT: "3306"
```

### Sequence syntax

```yaml
environment:
  - DB_HOST=database
  - DB_PORT=3306
```

### Service env file

```yaml
env_file:
  - ./config/backend.env
```

`env_file` tạo Container environment. `.env`/CLI `--env-file` thường cấp nguồn cho Compose interpolation. Hai cơ chế không đồng nghĩa.

### Interpolation forms

| Form | Kết quả |
|---|---|
| `${VAR}` | Giá trị VAR. |
| `${VAR:-default}` | Default nếu unset hoặc rỗng. |
| `${VAR-default}` | Default nếu unset. |
| `${VAR:?error}` | Fail nếu unset hoặc rỗng. |
| `${VAR?error}` | Fail nếu unset. |
| `$$` | Dấu `$` literal. |

Kiểm tra bằng `docker compose config --environment` và `docker compose config`.

## 6. `volumes`

### Short syntax

```text
SOURCE:TARGET[:MODE]
```

```yaml
services:
  database:
    volumes:
      - database-data:/var/lib/mysql
      - ./config:/etc/example:ro

volumes:
  database-data:
```

- `database-data` không bắt đầu bằng path nên là named Volume logical source.
- `./config` là host path nên là bind mount source.
- Target luôn là path trong Container.
- `ro` đặt read-only; mặc định thường read-write.

### Long syntax

```yaml
volumes:
  - type: volume
    source: database-data
    target: /var/lib/mysql
    read_only: false
```

Top-level Volume options:

| Key | Ý nghĩa |
|---|---|
| `name` | Runtime Volume name cụ thể. |
| `external` | Resource phải tồn tại ngoài project lifecycle. |
| `driver` | Volume driver. |
| `driver_opts` | Options cho driver. |
| `labels` | Metadata labels. |

## 7. `networks`

```yaml
services:
  backend:
    networks:
      - private

networks:
  private:
    driver: bridge
```

Service-level `networks` là attachment; top-level `networks` là declaration.

Long attachment có thể thêm alias:

```yaml
services:
  backend:
    networks:
      private:
        aliases:
          - api
```

Alias là thêm DNS name trong Network đó; service name vẫn là lựa chọn mặc định ổn định hơn khi không cần alias.

## 8. `healthcheck`

```yaml
healthcheck:
  test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
  interval: 10s
  timeout: 3s
  retries: 5
  start_period: 20s
```

| Key | Type | Ý nghĩa |
|---|---|---|
| `test` | Sequence hoặc scalar | Probe command. |
| `interval` | Duration | Khoảng giữa probe. |
| `timeout` | Duration | Giới hạn một probe. |
| `retries` | Integer | Failure liên tiếp trước unhealthy. |
| `start_period` | Duration | Grace period cho startup. |
| `start_interval` | Duration | Nhịp probe trong start period nếu phiên bản hỗ trợ. |

`CMD` chạy trực tiếp; `CMD-SHELL` chạy qua shell; `NONE` tắt healthcheck kế thừa.

## 9. `depends_on`

### Short syntax

```yaml
depends_on:
  - database
```

Chờ dependency được start theo flow Compose, không chờ healthy.

### Long syntax

```yaml
depends_on:
  database:
    condition: service_healthy
    restart: true
    required: true
```

| Condition | Ý nghĩa |
|---|---|
| `service_started` | Dependency đã start. |
| `service_healthy` | Dependency healthcheck đạt healthy. |
| `service_completed_successfully` | Dependency job kết thúc thành công. |

`depends_on.restart` không phải Service restart policy. Dependency không thay thế retry/reconnect runtime.

## 10. Multiple files và merge

```bash
docker compose -f compose.yaml -f compose.dev.yaml config
```

File sau override/merge file trước theo quy tắc từng field. Relative path có base resolution rules; luôn dùng `config` để xem kết quả. Không giả định mọi sequence chỉ được nối hoặc mọi mapping chỉ được thay toàn bộ.

## 11. Kiểm tra model

```bash
docker compose config
docker compose config --services
docker compose config --images
docker compose config --environment
```

`config` có thể in secret đã resolve. Bảo vệ output và log.

## Tài liệu liên quan

- [Compose commands](compose-commands.md)
- [2. Cách đọc Compose YAML](../../learning-path/05-docker-compose/02-cach-doc-compose-yaml.md)
- [7. Healthcheck và dependency](../../learning-path/05-docker-compose/07-healthcheck-va-dependency.md)
- Docker Docs, [Compose file reference](https://docs.docker.com/reference/compose-file/)

[Mục lục Compose Reference](README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)
