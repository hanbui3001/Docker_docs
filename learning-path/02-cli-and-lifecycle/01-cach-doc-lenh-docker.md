# 1. Cách đọc lệnh Docker

> **Tóm tắt một câu:** Một lệnh Docker là yêu cầu có cấu trúc; đọc đúng object, action, option và argument giúp bạn dự đoán chính xác tài nguyên nào sẽ được đọc hoặc thay đổi.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 30 phút<br>
> **Nguồn chính:** [Use the Docker command line](https://docs.docker.com/engine/cli/) · [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)

> **Sau chapter này, bạn có thể:**
> - Đọc cú pháp tổng quát và notation trong trang help.
> - Phân biệt global option, command option và argument.
> - Nhận ra modern object-action form cùng các alias cũ.
> - Viết lệnh nhiều dòng đúng trong Bash và PowerShell.

[Mục lục Part 02](README.md) · [2. Lệnh quản lý Image →](02-lenh-quan-ly-image.md)

---

## 1. Vấn đề cần giải quyết

Người mới thường ghi nhớ `docker ps`, `docker run`, `docker images` như các chuỗi rời rạc. Cách học này hoạt động khi ví dụ giống hệt tài liệu, nhưng dễ vỡ khi cần thêm option, đổi object hoặc đọc lỗi `unknown flag`.

Docker CLI có grammar, tức quy tắc tổ chức câu lệnh. Khi hiểu grammar, bạn không cần nhớ mọi biến thể: bạn có thể phân tích ai sở hữu option, argument đang chọn object nào và lệnh có làm thay đổi state hay chỉ đọc dữ liệu.

## 2. Mental model object–action

Cách viết rõ nghĩa nhất là:

```text
docker [GLOBAL OPTIONS] OBJECT ACTION [COMMAND OPTIONS] [ARGUMENTS]
```

Ví dụ:

```bash
docker container stop --time 20 web
```

```text
docker
└── container
    └── stop
        ├── --time 20
        └── web
```

| Token | Loại | Parser/scope sở hữu | Ý nghĩa |
|---|---|---|---|
| `docker` | Executable | Hệ điều hành/shell | Khởi chạy Docker CLI. |
| `container` | Object group | Docker CLI | Chọn nhóm Container. |
| `stop` | Action | Nhóm `container` | Yêu cầu dừng Container. |
| `--time 20` | Command option | Lệnh `container stop` | Cho process chính tối đa 20 giây để kết thúc trật tự. |
| `web` | Argument | Lệnh `container stop` | Tên hoặc ID của Container mục tiêu. |

Trước lệnh, `web` phải tồn tại và thường đang `running`. Sau khi dừng thành công, object vẫn tồn tại nhưng trạng thái chuyển sang `exited`. `stop` không xóa Container và không xóa Image.

## 3. Global option và command option

**Global option** là tùy chọn áp dụng cho Docker CLI trước khi chọn command, ví dụ `--context` hoặc `--debug`:

```bash
docker --context production container ls
```

`--context production` thuộc CLI cấp cao nhất, nên nó xuất hiện trước `container ls`. Nó chọn Docker endpoint đã cấu hình tên `production`; `container ls` sau đó chạy trên endpoint ấy.

**Command option** thuộc một action cụ thể:

```bash
docker container ls --all
```

`--all` thuộc `container ls`, yêu cầu hiện cả Container không chạy. Đặt option sai scope có thể tạo lỗi:

```bash
docker --all container ls
```

Docker CLI cố đọc `--all` như global option, nhưng global parser không sở hữu flag đó nên báo `unknown flag`.

> [!IMPORTANT]
> Vị trí option không chỉ là thẩm mỹ. Nó giúp xác định parser nào đọc option và option tác động lên phần nào của yêu cầu.

## 4. Argument, value và command bên trong Container

Argument thường chọn tài nguyên mục tiêu. Trong lệnh sau, `nginx:alpine` là Image argument, còn `nginx -g 'daemon off;'` là command và argument dành cho process bên trong Container:

```bash
docker container run nginx:alpine nginx -g 'daemon off;'
```

Docker CLI phân chia theo vị trí:

```text
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

| Phần | Giá trị đã resolve | Ai sử dụng? |
|---|---|---|
| `[OPTIONS]` | Không có trong ví dụ | Docker CLI/daemon khi tạo Container. |
| `IMAGE` | `nginx:alpine` | Docker daemon dùng để tìm Image nguồn. |
| `[COMMAND]` | `nginx` | Runtime dùng làm executable chính. |
| `[ARG...]` | `-g`, `daemon off;` | Process `nginx` nhận khi khởi động. |

Option của Docker phải đứng trước `IMAGE` trong mental model an toàn. Sau `IMAGE`, token có thể trở thành command của ứng dụng, không còn là Docker option.

## 5. Cách đọc notation trong help

```text
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

| Ký hiệu | Cách đọc |
|---|---|
| Chữ không nằm trong `[]` | Bắt buộc, ví dụ `IMAGE`. |
| `[OPTIONS]` | Có thể bỏ qua hoặc thêm option hợp lệ. |
| `[COMMAND]` | Command override là tùy chọn. |
| `[ARG...]` | Có thể có không, một hoặc nhiều argument. |
| `A\|B` trong một số tài liệu | Chọn một trong các dạng được nêu. |

Chữ hoa như `IMAGE` là placeholder, không phải literal cần gõ nguyên. Bạn thay bằng giá trị thực như `nginx:alpine`.

## 6. Help theo từng lớp

```bash
docker --help
docker container --help
docker container run --help
```

- `docker --help` liệt kê command và global option.
- `docker container --help` liệt kê action của object Container.
- `docker container run --help` hiển thị syntax và option của action `run`.

Help cục bộ thường đáng tin hơn bài nhớ lệnh cũ vì nó phản ánh CLI đang cài. Tuy nhiên một option hiện trong CLI chưa bảo đảm daemon từ xa quá cũ hỗ trợ đầy đủ; `docker version` giúp so sánh Client và Server.

## 7. Modern form và alias

Docker hỗ trợ cả object-action form rõ nghĩa và nhiều command alias lịch sử:

| Dạng rõ nghĩa | Alias thường gặp | Nhận xét |
|---|---|---|
| `docker image ls` | `docker images` | Cùng mục đích liệt kê Image. |
| `docker container ls` | `docker ps` | `ps` là tên quen thuộc từ Unix. |
| `docker container stop web` | `docker stop web` | Alias bỏ object group. |
| `docker container rm web` | `docker rm web` | Cần đọc context để biết đang xóa Container. |

Tài liệu này dùng dạng rõ nghĩa làm canonical syntax vì `image`, `container`, `volume`, `network` tạo hệ thống dễ học. Alias vẫn được nêu để bạn đọc được tutorial, script và trao đổi cộng đồng.

## 8. Bash và PowerShell

Lệnh một dòng thường dùng giống nhau:

```bash
docker container run --name web --detach --publish 8080:80 nginx:alpine
```

Khi xuống dòng, Bash dùng dấu `\`:

```bash
docker container run \
  --name web \
  --detach \
  --publish 8080:80 \
  nginx:alpine
```

PowerShell dùng backtick:

```powershell
docker container run `
  --name web `
  --detach `
  --publish 8080:80 `
  nginx:alpine
```

> [!WARNING]
> Trong PowerShell, backtick phải là ký tự cuối dòng. Khoảng trắng sau backtick làm line continuation hỏng và dòng tiếp theo có thể bị chạy như command riêng.

Nối lệnh cũng khác. Bash và PowerShell 7 hỗ trợ `&&` để chỉ chạy lệnh sau khi lệnh trước thành công. Windows PowerShell 5.1 không hỗ trợ `&&`; dùng kiểm tra `$LASTEXITCODE`:

```powershell
docker container stop web
if ($LASTEXITCODE -eq 0) { docker container rm web }
```

## 9. Quan niệm dễ gây hiểu nhầm

### 9.1 “Option đặt ở đâu cũng được”

- **Phân loại:** Sai theo scope parser.
- **Vì sao nghe hợp lý:** Một số CLI linh hoạt về thứ tự option.
- **Lỗi kỹ thuật:** Docker có global parser và parser riêng cho từng command; sau `IMAGE`, token của `run` còn có thể thuộc process bên trong.
- **Cách nói tốt hơn:** Đặt global option trước command, command option trong vị trí syntax của action và command ứng dụng sau Image.
- **Kiểm chứng:** So sánh `docker container ls --all` với `docker --all container ls`.

### 9.2 “`docker ps` là một hệ lệnh khác `docker container ls`”

- **Phân loại:** Sai; đây là alias lịch sử và object-action form.
- **Lỗi kỹ thuật:** Chúng cùng truy vấn danh sách Container; sự khác nhau chủ yếu là cách biểu đạt.
- **Cách nói tốt hơn:** Học object-action để hiểu cấu trúc, đồng thời nhận diện alias để đọc nội dung cũ.

### 9.3 “Command thành công nghĩa là ứng dụng bên trong khỏe mạnh”

- **Phân loại:** Sai do trộn trạng thái yêu cầu CLI với trạng thái ứng dụng.
- **Lỗi kỹ thuật:** `docker container start` có thể thành công khi process vừa khởi động nhưng ứng dụng sau đó crash hoặc chưa sẵn sàng phục vụ.
- **Kiểm chứng:** Dùng `docker container ls --all`, `docker container logs` và health status nếu Image có healthcheck.

## 10. Tự kiểm tra mental model

1. Trong `docker --context prod container ls --all`, option nào là global và option nào thuộc command?
2. Vì sao `docker container run alpine --name demo` không nên được hiểu là đặt tên Container?
3. `[ARG...]` cho biết điều gì về số lượng argument?
4. Khi nào nên dùng alias, khi nào object-action form giúp giảm nhầm lẫn?

## 11. Tóm tắt

1. Docker CLI có grammar; command không phải chuỗi thần chú.
2. Object chọn nhóm tài nguyên, action chọn hành động, option điều chỉnh hành vi và argument chọn mục tiêu.
3. Scope và vị trí quyết định parser nào sở hữu option.
4. Modern object-action form dễ học; alias vẫn phổ biến.
5. Bash và PowerShell khác nhau ở continuation, quoting và một số toán tử nối lệnh.

## 12. Học tiếp

Đọc [2. Lệnh quản lý Image](02-lenh-quan-ly-image.md) để áp dụng grammar vào object đầu tiên. Khi cần tra nhanh cú pháp, mở [Docker CLI quick reference](../../reference/commands/README.md).

## Tài liệu tham khảo

- Docker Docs, [Use the Docker command line](https://docs.docker.com/engine/cli/)
- Docker Docs, [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
- Docker Docs, [`docker container run`](https://docs.docker.com/reference/cli/docker/container/run/)

[Mục lục Part 02](README.md) · [2. Lệnh quản lý Image →](02-lenh-quan-ly-image.md)
