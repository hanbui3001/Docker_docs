# 2. Cách đọc Compose YAML

> **Tóm tắt một câu:** Muốn hiểu Compose, hãy đọc YAML như một cây dữ liệu có scope: indentation xác định quan hệ cha-con, dấu `-` tạo phần tử danh sách và vị trí của key quyết định object nào sở hữu giá trị.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 35 phút  
> **Nguồn chính:** [Compose file reference](https://docs.docker.com/reference/compose-file/) · [YAML 1.2 specification](https://yaml.org/spec/1.2.2/)

[← 1. Docker Compose là gì?](01-compose-la-gi.md) · [Mục lục Part 05](README.md) · [3. Service, Image và build →](03-service-image-va-build.md)

---

## 1. Vì sao người mới thường đọc sai Compose file?

YAML trông gần với văn bản tự nhiên nhưng vẫn là ngôn ngữ biểu diễn dữ liệu có quy tắc. Hai key cùng chữ có thể mang ý nghĩa khác vì nằm ở hai nhánh khác nhau. Một dấu cách sai có thể chuyển key sang owner khác hoặc làm file không parse được.

Ba câu hỏi nên hỏi khi đọc mọi dòng:

1. Dòng này thuộc node cha nào?
2. Giá trị là mapping, sequence hay scalar?
3. Compose Specification gán ý nghĩa gì cho đường dẫn đầy đủ của key?

## 2. Ba kiểu dữ liệu nền tảng

### 2.1 Mapping: tập key-value

**Mapping** — cấu trúc ánh xạ một key tới một value, tương tự object hoặc map trong ngôn ngữ lập trình.

```yaml
services:
  backend:
    image: demo/backend:1.0
```

`services`, `backend`, `image` đều là key. Value của `image` là scalar; value của `backend` là mapping; value của `services` cũng là mapping.

### 2.2 Sequence: danh sách có thứ tự

**Sequence** — danh sách các phần tử, mỗi phần tử thường bắt đầu bằng `-`.

```yaml
ports:
  - "8080:8080"
  - "9090:9090"
```

Đường dẫn hai phần tử là `ports[0]` và `ports[1]`. Dấu `-` không phải trang trí; nó tạo một item trong sequence.

### 2.3 Scalar: một giá trị đơn

**Scalar** — giá trị đơn như chuỗi, số, boolean hoặc null.

```yaml
image: mysql:8.4
restart: unless-stopped
read_only: true
```

Compose mong đợi type cụ thể ở từng key. Việc YAML parse `true` thành boolean nhưng `"true"` thành chuỗi có thể tạo hành vi khác.

## 3. Indentation là cấu trúc, không phải thẩm mỹ

YAML dùng indentation bằng space để biểu diễn nesting. Không dùng tab.

```yaml
services:
  backend:
    image: demo/backend:1.0
    environment:
      SERVER_PORT: "8080"
```

Cây tương ứng:

```text
root
└── services                         mapping
    └── backend                      mapping
        ├── image                    scalar
        └── environment              mapping
            └── SERVER_PORT          scalar
```

Đường dẫn đầy đủ là `services.backend.environment.SERVER_PORT`. `SERVER_PORT` không phải top-level Compose key; nó là tên biến môi trường thuộc Service `backend`.

Ví dụ sai scope:

```yaml
services:
  backend:
    image: demo/backend:1.0
  environment:
    SERVER_PORT: "8080"
```

Ở đây `environment` ngang hàng với `backend`, nên Compose hiểu nó như tên một Service mới. Parser YAML có thể vẫn chấp nhận file; lỗi nằm ở application model chứ không nhất thiết ở cú pháp YAML.

> [!IMPORTANT]
> “YAML hợp lệ” chưa đồng nghĩa “Compose model hợp lệ”. YAML parser kiểm tra cấu trúc dữ liệu; Compose tiếp tục kiểm tra key, type và ngữ nghĩa theo Compose Specification.

## 4. Ownership: key thuộc về scope nào?

Xét file:

```yaml
services:
  database:
    image: mysql:8.4
    volumes:
      - database-data:/var/lib/mysql

volumes:
  database-data:
```

Hai key `volumes` có owner khác nhau:

| Đường dẫn | Owner | Ý nghĩa |
|---|---|---|
| `services.database.volumes` | Service `database` | Khai báo mount nào được gắn vào Container của Service. |
| `volumes.database-data` | Compose project | Khai báo named Volume resource có logical name `database-data`. |

Nếu bỏ top-level declaration nhưng vẫn dùng named Volume trong Service, Compose thường báo model không hợp lệ vì source chưa được khai báo. Nếu khai báo top-level Volume nhưng không mount vào Service nào, resource có trong model nhưng không cung cấp dữ liệu cho Container.

## 5. Short syntax và long syntax

Compose thường cho phép hai cách biểu diễn cùng nhóm cấu hình.

Short syntax nén nhiều field vào một scalar:

```yaml
ports:
  - "8080:80"
```

Long syntax biểu diễn rõ từng field:

```yaml
ports:
  - target: 80
    published: "8080"
    protocol: tcp
```

Short syntax dễ viết và phù hợp case đơn giản. Long syntax dễ mở rộng, ít mơ hồ hơn và thể hiện ownership từng giá trị. Không nên mặc định short syntax “kém chuẩn”; cả hai đều là cú pháp hợp lệ khi Specification hỗ trợ.

## 6. Quoting: khi nào nên đặt dấu nháy?

YAML có thể tự suy luận type. Với Compose, quoting hữu ích khi scalar chứa dấu `:`, ký hiệu biến, số có ý nghĩa như chuỗi hoặc giá trị có thể bị YAML diễn giải ngoài ý muốn.

```yaml
ports:
  - "8080:80"
environment:
  FEATURE_ENABLED: "true"
  BUILD_NUMBER: "0012"
```

- Port mapping được quote để toàn bộ `8080:80` rõ ràng là một scalar compact syntax.
- Environment của process cuối cùng là chuỗi. Quote giúp giữ `"true"` là chuỗi và giữ số `0` đầu của `"0012"`.

Single quote và double quote không hoàn toàn giống nhau trong YAML. Double quote xử lý escape sequence; single quote gần literal hơn. Compose interpolation dùng cú pháp `$` trước khi giá trị được gửi tới Engine, nên cần xem cả quy tắc Compose chứ không chỉ quy tắc quote của shell.

## 7. Null, giá trị rỗng và key bị bỏ qua

Các dạng sau nhìn gần giống nhưng có thể khác nghĩa:

```yaml
environment:
  API_URL:
  EMPTY_VALUE: ""
  FIXED_VALUE: "https://example.test"
```

- `API_URL:` không có value rõ ràng; với `environment`, Compose có thể tìm giá trị từ môi trường chạy Compose và bỏ biến nếu không resolve được, tùy dạng khai báo.
- `EMPTY_VALUE: ""` chủ động truyền chuỗi rỗng.
- `FIXED_VALUE` truyền chuỗi cụ thể.

Đừng dùng null và empty string thay thế lẫn nhau nếu ứng dụng phân biệt “biến không tồn tại” với “biến tồn tại nhưng rỗng”.

## 8. Relative path được resolve từ đâu?

Path không tự nhiên thuộc về terminal hiện tại trong mọi trường hợp. Compose có quy tắc xác định project directory và base file. Ví dụ:

```yaml
services:
  backend:
    build:
      context: ./backend
    env_file:
      - ./config/backend.env
```

`./backend` và `./config/backend.env` cần được hiểu theo base path của Compose project/file theo quy tắc của Compose CLI, đặc biệt khi dùng nhiều `-f`. Chạy lệnh từ thư mục khác nhưng chỉ nhìn chuỗi `./` có thể dẫn đến kết luận sai.

Dùng lệnh sau để xem model đã resolve:

```bash
docker compose config
```

`config` parse, merge và render model; nó không khởi động Container. Đây là lệnh kiểm tra quan trọng nhất trước khi đoán Compose đã hiểu path hoặc biến như thế nào.

## 9. Top-level `version` có còn cần không?

Compose Specification hiện đại dùng model versionless. Top-level `version` cũ chỉ còn tính tương thích/thông tin trong Compose mới và có thể tạo cảnh báo obsolete.

```yaml
services:
  web:
    image: nginx:alpine
```

Không cần thêm `version: "3.8"` chỉ để file “đúng chuẩn”. Khả năng hỗ trợ feature phụ thuộc Compose implementation đang dùng, không phải việc nâng chuỗi version tùy ý.

## 10. Quan niệm dễ gây hiểu nhầm

### 10.1 “Cùng tên key thì cùng ý nghĩa.”

Sai vì đường dẫn đầy đủ quyết định scope. `services.database.volumes` là mount list; top-level `volumes` là resource declaration.

### 10.2 “Thụt lề chỉ để file đẹp.”

Sai vì indentation xây cây cha-con. Thay indentation có thể đổi owner hoặc type của node.

### 10.3 “File parse được nghĩa là chạy đúng.”

Sai vì còn các lớp validation, interpolation, path resolution và lỗi runtime như port conflict hoặc Image không tồn tại.

### 10.4 “Mọi giá trị YAML cuối cùng giữ nguyên type trong Container.”

Sai vì environment của process là chuỗi, trong khi Compose model có nhiều field boolean, number, duration và mapping với type riêng.

## 11. Quy trình đọc một block Compose

Khi gặp một block mới:

1. Viết đường dẫn đầy đủ, ví dụ `services.backend.healthcheck.test`.
2. Xác định value là mapping, sequence hay scalar.
3. Xác định token compact syntax nếu value là chuỗi nén.
4. Tìm owner và default trong Compose reference.
5. Chạy `docker compose config` để xem giá trị sau resolve.
6. Chỉ sau đó mới kết luận resource hoặc lifecycle nào sẽ thay đổi.

## 12. Tóm tắt

1. Mapping, sequence và scalar là ba khối dữ liệu chính khi đọc Compose YAML.
2. Indentation xác định nesting; đường dẫn đầy đủ xác định ownership.
3. Short syntax nén field vào chuỗi; long syntax mở rõ field.
4. Quote giúp kiểm soát type và compact syntax nhưng không thay thế hiểu biết về interpolation.
5. `docker compose config` cho thấy model sau parse, merge và resolve mà không tạo Container.

## Tài liệu tham khảo

- Docker Docs, [Compose file reference](https://docs.docker.com/reference/compose-file/)
- Docker Docs, [Merge Compose files](https://docs.docker.com/compose/how-tos/multiple-compose-files/merge/)
- YAML, [YAML 1.2.2 Specification](https://yaml.org/spec/1.2.2/)

[← 1. Docker Compose là gì?](01-compose-la-gi.md) · [Mục lục Part 05](README.md) · [3. Service, Image và build →](03-service-image-va-build.md)
