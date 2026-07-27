# Dockerfile Reference

> Trang tra cứu nhanh cho cú pháp Dockerfile và `docker build`. Nếu bạn đang học từ đầu, bắt đầu tại [Part 04 — Dockerfile](../../learning-path/04-dockerfile/README.md).

## Nội dung

| Trang | Dùng khi |
|---|---|
| [Dockerfile instructions](instructions.md) | Cần cú pháp, scope, default và so sánh instruction |
| [`docker build` options](docker-build-options.md) | Cần chọn Dockerfile/context, Tag, target, build arg, secret hoặc cache behavior |

## Grammar nền tảng

```text
INSTRUCTION arguments
```

- Instruction không phân biệt chữ hoa/thường về parser, nhưng quy ước dùng chữ hoa để tách khỏi argument.
- Dockerfile được xử lý theo thứ tự và mỗi `FROM` mở stage mới.
- Path source/destination phải được đọc cùng scope; đặc biệt source của `COPY` không cùng filesystem với destination.

## Liên kết chính thức

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [`docker build` reference](https://docs.docker.com/reference/cli/docker/buildx/build/)

[Mục lục Part 04](../../learning-path/04-dockerfile/README.md)
