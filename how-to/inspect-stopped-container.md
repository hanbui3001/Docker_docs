# Inspect một Container đã dừng

> **Mục tiêu:** Xác định Container dừng vì process hoàn thành, ứng dụng lỗi hay runtime không khởi động được, mà chưa làm mất log và trạng thái hiện tại.

> **Loại:** How-to · **Điều kiện:** Docker Engine đang hoạt động; biết tên hoặc một phần tên Container<br>
> **Đọc nền:** [Container lifecycle](../learning-path/02-cli-and-lifecycle/04-container-lifecycle.md) · [Quan sát và debug Container](../learning-path/02-cli-and-lifecycle/05-quan-sat-va-debug-container.md)

[Mục lục How-to](README.md) · [Chẩn đoán published port →](diagnose-published-port.md)

---

## 1. Vấn đề

Container không xuất hiện trong `docker container ls`, hoặc ứng dụng vừa dừng. Đừng remove hay recreate ngay: stopped Container vẫn giữ exit code, timestamps, configuration, logs và có thể cả dữ liệu trong writable layer.

## 2. Kiểm tra trạng thái hiện tại

Thay `<container>` bằng tên hoặc ID thực tế:

```bash
docker container ls --all --filter name=<container>
```

`--all` mở rộng danh sách sang Container `created`, `exited` và các state khác; `--filter name=...` thu hẹp output. Nếu không có kết quả, kiểm tra lại tên bằng `docker container ls --all` trước khi kết luận object đã bị xóa.

Lấy snapshot state ngắn:

```bash
docker container inspect <container> --format 'Status={{.State.Status}} Exit={{.State.ExitCode}} OOM={{.State.OOMKilled}} Error={{.State.Error}} Started={{.State.StartedAt}} Finished={{.State.FinishedAt}}'
```

Các trường quan trọng:

- `Status=exited`: process chính đã kết thúc.
- `Exit=0`: thường là hoàn thành bình thường; vẫn cần đối chiếu mục đích workload.
- `OOM=true`: process bị kết thúc do thiếu memory theo giới hạn hoặc host pressure.
- `Error`: lỗi runtime khi tạo hoặc khởi động process, có thể khác log ứng dụng.

## 3. Thu thập bằng chứng

Đọc tối đa 200 dòng log gần cuối:

```bash
docker container logs --timestamps --tail 200 <container>
```

Sau đó kiểm tra command, Image và restart policy thực tế:

```bash
docker container inspect <container> --format 'Image={{.Config.Image}} Entrypoint={{json .Config.Entrypoint}} Cmd={{json .Config.Cmd}} Restart={{json .HostConfig.RestartPolicy}}'
```

`{{json ...}}` là format template do Docker CLI xử lý, giúp giữ cấu trúc mảng command. Không đăng toàn bộ output inspect công khai: `.Config.Env`, label hoặc metadata khác có thể chứa thông tin nhạy cảm.

Nếu cần giữ một file trong writable layer trước khi recreate, `docker container cp` hoạt động với cả Container đang chạy và đã dừng:

```bash
docker container cp <container>:/path/to/file ./container-evidence-file
```

Token trước dấu `:` là Container; path sau dấu `:` nằm trong filesystem view của Container; `./container-evidence-file` là destination trên host.

## 4. Thay đổi có kiểm soát

Nếu configuration vẫn đúng và workload nên chạy lại, start cùng object:

```bash
docker container start <container>
```

Nếu command, environment, port hoặc mount sai, không thể sửa đầy đủ create-time configuration bằng `start`. Ghi lại `inspect`, bảo toàn dữ liệu cần thiết, rồi recreate theo [Giữ dữ liệu khi recreate Container](preserve-data-when-recreating-container.md).

## 5. Xác minh

```bash
docker container ls --all --filter name=<container>
docker container inspect <container> --format 'Status={{.State.Status}} Exit={{.State.ExitCode}}'
docker container logs --since 2m <container>
```

`running` chỉ chứng minh process chính chưa thoát. Dùng health status hoặc request thực tế nếu cần xác nhận ứng dụng ready.

## 6. Recovery và cleanup

- Nếu start làm lỗi lặp lại, dùng `docker container stop <container>` nếu process còn chạy, rồi giữ object để điều tra.
- Không dùng `docker container rm --force` trước khi đã lấy log, inspect và file cần bảo toàn.
- Khi chắc chắn object không còn giá trị, dừng rồi xóa có mục tiêu:

```bash
docker container stop <container>
docker container rm <container>
```

> [!WARNING]
> Remove xóa metadata và writable layer của Container. Dữ liệu chưa nằm trong Volume, bind mount hoặc bản sao host sẽ không được khôi phục bằng cách start lại.

## Tài liệu tham khảo

- Docker CLI, [`docker container inspect`](https://docs.docker.com/reference/cli/docker/inspect/)
- Docker CLI, [`docker container logs`](https://docs.docker.com/reference/cli/docker/container/logs/)
- Docker CLI, [`docker container cp`](https://docs.docker.com/reference/cli/docker/container/cp/)

[Mục lục How-to](README.md) · [Chẩn đoán published port →](diagnose-published-port.md)
