# Docker Compose Reference

[← Mục lục sách](../../README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)

Thư mục này dùng để tra cứu cú pháp chính xác sau khi đã có mental model nền tảng. Nội dung được nhóm theo object và command, không thay thế các chapter giải thích tuần tự.

## Trang tra cứu

| Trang | Dùng khi nào? |
|---|---|
| [Compose file keys](compose-file-keys.md) | Cần biết key nằm ở scope nào, nhận type gì và ảnh hưởng resource nào. |
| [Compose commands](compose-commands.md) | Cần chọn lệnh theo mục tiêu và hiểu state transition. |

## Ngữ pháp cốt lõi

```text
compose.yaml
└── top-level resources
    ├── services
    ├── networks
    ├── volumes
    ├── configs
    └── secrets
```

```text
docker compose [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [SERVICE...]
```

## Quy tắc tra cứu

1. Xác định đường dẫn đầy đủ của key, ví dụ `services.backend.volumes`, không chỉ tìm chữ `volumes`.
2. Xác định value là mapping, sequence hay scalar.
3. Với short syntax, tách từng token trước khi suy luận.
4. Chạy `docker compose config` để xem model đã resolve.
5. Với command, phân biệt resource được giữ, dừng, recreate hay xóa.

## Nguồn chính

- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Docker Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
- [Compose Specification](https://compose-spec.io/compose-spec/)

[← Mục lục sách](../../README.md) · [Learning Path: Docker Compose](../../learning-path/05-docker-compose/README.md)
