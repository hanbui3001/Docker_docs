# 3. Container không khởi động

> **Tóm tắt một câu:** Container “không chạy” có thể chưa được tạo, đang `created`, đã `exited`, bị OOM kill hoặc mắc restart loop; phải xác định lifecycle state và process exit trước khi sửa command hay Image.

> **Thuật ngữ:** **Exit code** là mã process trả về khi kết thúc. **OOM kill** là việc hệ điều hành/runtime kết thúc process do thiếu memory. **Restart loop** là chu kỳ process thoát rồi restart policy khởi động lại liên tục.

[← 2. Lỗi build Image](02-loi-build-image.md) · [Mục lục Troubleshooting](README.md) · [4. Lỗi port và network →](04-loi-port-va-network.md)

---

## 1. Symptom: “Container không thấy trong `docker ps`”

`docker ps` mặc định chỉ hiện Container đang chạy. Bước đầu tiên là mở rộng phạm vi quan sát:

```bash
docker container ls -a
docker container inspect <container> --format 'status={{.State.Status}} exit={{.State.ExitCode}} error={{.State.Error}} oom={{.State.OOMKilled}} restart={{.RestartCount}}'
```

| Actual state | Hypothesis ban đầu |
|---|---|
| Không có object | `docker run`/Compose create chưa thành công hoặc dùng sai project/context |
| `created` | Runtime chưa start được process, mount hoặc executable |
| `exited (0)` | Process hoàn thành bình thường nhưng không phải long-running service |
| `exited` khác `0` | Application, command hoặc dependency thất bại |
| `restarting` | Restart policy đang che một lỗi lặp lại |
| `OOMKilled=true` | Process vượt memory có thể cấp hoặc host thiếu memory |

Không kết luận “Docker bị lỗi” chỉ từ việc Container không xuất hiện trong danh sách running.

## 2. Hypothesis: command hoặc executable sai

### Evidence

```bash
docker container inspect <container> --format '{{json .Config.Entrypoint}} {{json .Config.Cmd}}'
docker container logs --timestamps --tail 200 <container>
```

So sánh command đã resolve với Dockerfile/Compose mong muốn. Các dấu hiệu thường gặp:

- `executable file not found`: tên executable không tồn tại trong `PATH` hoặc Image.
- `permission denied`: file có nhưng thiếu execute bit, mount bị `noexec`, hoặc user không có quyền.
- `exec format error`: binary/script không đúng platform, thiếu shebang hoặc line ending không phù hợp.
- Process chạy xong và exit `0`: command là job ngắn hạn, không phải server.

Với Image có shell, có thể tạo phiên điều tra bằng command thay thế:

```bash
docker run --rm --entrypoint sh <image> -c 'id; pwd; ls -la; command -v <executable>'
```

`--entrypoint` chỉ dùng để thu bằng chứng. Nó không chứng minh entrypoint gốc đúng.

### Correction

- Sửa `ENTRYPOINT`/`CMD` đúng exec form và đúng executable.
- Copy artifact vào đúng destination, đặt execute permission trong Image khi cần.
- Build cho đúng platform hoặc dùng base Image tương thích.
- Nếu đây là batch job, chấp nhận `exited (0)` thay vì ép process sống bằng lệnh giả.

### Verification

```bash
docker run --name startup-check <image>
docker inspect startup-check --format 'status={{.State.Status}} exit={{.State.ExitCode}}'
docker logs startup-check
```

Tiêu chí pass phụ thuộc workload: service cần ở `running` và health tốt; job cần `exited (0)` cùng output đúng.

## 3. Hypothesis: application tự thoát do config hoặc dependency

### Evidence

```bash
docker logs --timestamps --tail 300 <container>
docker inspect <container> --format '{{json .Config.Env}}'
docker inspect <container> --format '{{json .Mounts}}'
```

Đọc log từ lỗi đầu tiên theo timeline, không chỉ dòng cuối. Kiểm tra biến môi trường, secret path, database hostname, migration và file cấu hình thực tế.

> [!CAUTION]
> Output `.Config.Env` có thể chứa password/token. Không dán nguyên output vào issue công khai; hãy redact secret nhưng giữ tên biến và trạng thái có/không.

### Correction

Sửa đúng input gây lỗi: hostname service thay cho `localhost`, mount đúng config, cung cấp credential qua secret mechanism, hoặc thêm retry/readiness nếu dependency cần thời gian khởi động. Không dùng sleep dài cố định để che race condition.

### Verification

Recreate Container để desired config trở thành actual config, rồi kiểm tra:

```bash
docker container inspect <container> --format '{{.State.Status}}'
docker container logs --since 5m <container>
```

Xác minh cả service behavior, ví dụ health endpoint hoặc truy vấn nghiệp vụ tối thiểu.

## 4. Hypothesis: OOM hoặc resource limit

### Evidence

```bash
docker inspect <container> --format 'oom={{.State.OOMKilled}} exit={{.State.ExitCode}} memory={{.HostConfig.Memory}}'
docker stats --no-stream
docker info
```

Exit code `137` thường liên quan `SIGKILL`, nhưng không tự chứng minh OOM; trường `OOMKilled`, host event và memory metrics mới tăng độ chắc chắn.

### Correction

- Giảm memory thực dùng hoặc leak của application.
- Đặt heap phù hợp với container limit, tính cả native memory.
- Tăng limit khi workload hợp lệ và host còn capacity.

### Verification

Chạy lại cùng workload đủ lâu, quan sát `docker stats`, restart count và OOM state. Một lần start thành công chưa loại trừ memory growth theo thời gian.

## 5. Hypothesis: restart policy che root cause

### Evidence

```bash
docker inspect <container> --format 'policy={{.HostConfig.RestartPolicy.Name}} count={{.RestartCount}} status={{.State.Status}}'
docker logs --timestamps --tail 300 <container>
```

Restart count tăng và log lặp cùng startup error là bằng chứng của loop.

> [!WARNING]
> Tắt restart policy hoặc stop Container có thể làm gián đoạn service. Chụp log, inspect và timeline trước; thực hiện trong phạm vi được phép.

### Correction

Sửa root cause khiến process thoát. Restart policy là cơ chế recovery, không phải cách làm application đúng.

### Verification

Sau recreate, restart count không tiếp tục tăng, health ổn định và log không lặp startup sequence bất thường.

## 6. Quan niệm dễ sai

### “Exit code `0` nghĩa Container khỏe.”

Chỉ nghĩa process kết thúc thành công theo contract của nó. Với web server, việc thoát ngay dù code `0` vẫn trái desired state.

### “Thêm `tail -f /dev/null` sẽ sửa Container bị thoát.”

Lệnh này chỉ giữ một process giả sống và che việc application chính không chạy. Nó có thể hữu ích trong lab điều tra nhưng không phải correction production.

## 7. Tóm tắt

1. Mở `docker ps -a`, đọc state, exit code, OOM và restart count.
2. So command đã resolve với artifact và platform thực tế.
3. Dùng log/config/mount làm evidence trước khi recreate.
4. Verification phải phù hợp loại workload: service, worker hay batch job.

## Tài liệu tham khảo

- Docker CLI, [docker container inspect](https://docs.docker.com/reference/cli/docker/container/inspect/)
- Docker Docs, [Start containers automatically](https://docs.docker.com/engine/containers/start-containers-automatically/)

[← 2. Lỗi build Image](02-loi-build-image.md) · [Mục lục Troubleshooting](README.md) · [4. Lỗi port và network →](04-loi-port-va-network.md)
