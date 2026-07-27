# 4. CPU, memory và PID limits

> **Tóm tắt một câu:** Resource limit không làm workload nhanh hơn; nó xác định mức tài nguyên được phép cạnh tranh và hành vi khi workload vượt ranh giới.

> **Loại:** Explanation · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 30 phút  
> **Nguồn chính:** [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/) · [Compose services](https://docs.docker.com/reference/compose-file/services/)

> **Sau chapter này, bạn có thể:**
> - Phân biệt memory limit, CPU quota và PID limit theo failure behavior.
> - Phân tích token của `docker run` resource options và Compose service keys.
> - Kiểm tra limit thực tế trên Container và nhận diện OOM/throttling.
> - Giải thích vì sao runtime memory setting phải chừa headroom.

> **Thuật ngữ:** **Limit** là trần sử dụng. **Reservation** là lượng tài nguyên mong muốn được dành hoặc dùng để lập lịch ở nền tảng hỗ trợ. **OOM** là tình trạng thiếu memory; **OOM kill** là kernel kết thúc process để giải phóng memory. **Throttling** là trì hoãn thời gian chạy khi workload vượt CPU quota thay vì cấp thêm CPU.

[← 3. Configuration và secret](03-configuration-va-secret.md) · [Mục lục Production](README.md) · [5. Health và restart →](05-health-va-restart.md)

---

## 1. Nếu không đặt limit

Container không tự có một phần RAM nhỏ cố định. Nếu không cấu hình, workload có thể sử dụng tài nguyên host trong phạm vi runtime và hệ điều hành cho phép, cạnh tranh với Container khác và Docker Engine.

```bash
docker container run --name api --memory 512m --cpus 1.5 --pids-limit 200 example/api:1.0
```

| Token | Parser/scope | Giá trị đã hiểu |
|---|---|---|
| `docker container run` | Docker CLI | Tạo rồi start một Container mới. |
| `--name api` | Option của `run` | Đặt Container name `api`. |
| `--memory 512m` | HostConfig runtime | Memory limit 512 MiB. |
| `--cpus 1.5` | CPU quota shorthand | Thời lượng CPU tương đương khoảng 1,5 CPU, không phải ownership core riêng. |
| `--pids-limit 200` | Linux PID cgroup/runtime | Giới hạn số process/task Container có thể tạo. |
| `example/api:1.0` | Argument | Image reference dùng tạo Container. |

Trước lệnh, Container `api` chưa tồn tại. Sau lệnh, runtime tạo Container với resource constraints trong HostConfig; nếu tên đã tồn tại, `run` thất bại thay vì cập nhật object cũ.

## 2. Memory limit và OOM

Khi process tiếp tục yêu cầu memory vượt giới hạn và không thể thu hồi đủ, kernel có thể OOM-kill process. Container thường chuyển sang `exited` nếu process chính bị kill.

```bash
docker container inspect api --format 'Limit={{.HostConfig.Memory}} OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'
```

- `.HostConfig.Memory` là limit byte đã áp dụng; `536870912` tương ứng 512 MiB.
- `.State.OOMKilled` cho biết process chính có bị đánh dấu OOM kill trong lần chạy được ghi nhận hay không.
- `.State.ExitCode` hỗ trợ chẩn đoán nhưng không nên dùng một mình để khẳng định nguyên nhân.

Application memory setting phải phù hợp limit. Ví dụ JVM cần chừa memory cho heap, metaspace, thread stacks, native buffers và chính runtime; đặt heap bằng toàn bộ Container limit dễ gây OOM ngoài heap.

**Headroom** — phần tài nguyên chừa lại ngoài mức sử dụng chính để hấp thụ native memory, spike và overhead. Headroom không phải một tỷ lệ cố định; cần đo bằng load test và runtime metrics.

## 3. CPU limit là throttling, không phải memory kill

Khi workload muốn dùng nhiều CPU hơn quota, scheduler thường throttle: trì hoãn thời gian chạy để giữ mức sử dụng trong quota. Service có thể chậm và latency tăng nhưng process không nhất thiết bị kill. Vì vậy cần nhìn cả CPU usage, throttling metric và response latency.

```bash
docker stats api
```

`stats` stream CPU, memory, network, block I/O và PID tại thời điểm quan sát. Nó không thay metric có lịch sử; một snapshot thấp sau incident không chứng minh trước đó không bị saturation.

**Saturation** — trạng thái tài nguyên hoặc hàng đợi đã gần/hết khả năng phục vụ thêm công việc, thường thể hiện qua latency hoặc queue tăng.

## 4. PID limit bảo vệ loại failure khác

PID limit giảm tác động của process leak, thread explosion hoặc fork bomb. Vượt limit thường làm việc tạo process/thread mới thất bại; Container không nhất thiết dừng ngay.

```bash
docker container exec api sh -c 'ps -e | wc -l'
docker container inspect api --format 'PidsLimit={{.HostConfig.PidsLimit}}'
```

Lệnh đầu phụ thuộc `sh`, `ps`, `wc` có trong Image và chỉ cho snapshot process. Lệnh inspect đọc limit runtime đã áp dụng.

## 5. Compose syntax và scope

```yaml
services:
  api:
    image: example/api:1.0
    mem_limit: 512m
    cpus: 1.5
    pids_limit: 200
```

| Đường dẫn | Owner | Runtime effect mong muốn |
|---|---|---|
| `services.api.mem_limit` | Service `api` | Memory limit của Container. |
| `services.api.cpus` | Service `api` | CPU quota shorthand. |
| `services.api.pids_limit` | Service `api` | PID limit. |

Compose và deployment platform còn có `deploy.resources`; mức áp dụng phụ thuộc implementation và cách triển khai. Không trộn reservation với hard limit và không chỉ tin file YAML trông đúng.

```bash
docker compose config
docker compose up -d api
docker compose ps -q api
docker container inspect <container-id> --format 'Memory={{.HostConfig.Memory}} NanoCpus={{.HostConfig.NanoCpus}} Pids={{.HostConfig.PidsLimit}}'
```

`config` kiểm tra resolved model. `up` reconcile Service và có thể recreate Container. `ps -q` trả Container ID; inspect xác nhận effective HostConfig sau create.

## 6. Limit phải đi cùng capacity planning

Limit quá cao không bảo vệ host; limit quá thấp tạo OOM hoặc throttling dưới tải hợp lệ. Quy trình tối thiểu:

```text
Đo baseline -> load test -> chọn limit + headroom
-> deploy -> quan sát OOM/throttling/latency
-> điều chỉnh có version và rollback
```

**Capacity planning** — ước lượng tài nguyên cần thiết dựa trên tải, mục tiêu chất lượng và headroom, thay vì chọn số tùy ý. Với nhiều Container trên một host, tổng limit có thể vượt capacity vật lý; limit riêng lẻ không tự giải quyết overcommit.

## 7. Quan niệm dễ sai

### “Đặt memory limit làm ứng dụng dùng ít memory hơn một cách an toàn.”

- **Phân loại:** Sai.
- **Vì sao nghe hợp lý:** Runtime cưỡng chế một con số tối đa.
- **Lỗi kỹ thuật:** Limit tạo ranh giới; ứng dụng vẫn có thể vượt và bị OOM-kill.
- **Cách nói tốt hơn:** Kết hợp limit với runtime tuning, load test và monitoring.

### “`--cpus 2` nghĩa Container sở hữu riêng hai core.”

- **Phân loại:** Thường sai.
- **Lỗi kỹ thuật:** Đây chủ yếu là quota thời gian CPU; CPU pinning/affinity là cơ chế khác.
- **Kiểm chứng:** Đọc HostConfig và metric throttling thay vì suy luận từ tên option.

### “Compose file có limit thì Container hiện tại đã nhận limit.”

- **Phân loại:** Sai nếu chưa reconcile hoặc key không được implementation áp dụng.
- **Kiểm chứng:** Chạy `docker compose config`, `up` rồi đọc `.HostConfig` của Container mới.

## 8. Tự kiểm tra mental model

1. OOM và CPU throttling khác nhau ở phản ứng của kernel/runtime như thế nào?
2. Vì sao heap 512 MiB không phù hợp với Container limit đúng 512 MiB?
3. PID limit bảo vệ failure mode nào mà memory limit không mô tả trực tiếp?

## 9. Tóm tắt

1. Limit bảo vệ host và workload lân cận nhưng cần capacity planning.
2. Memory vượt trần có thể dẫn đến OOM kill; CPU vượt quota thường bị throttle.
3. PID limit kiểm soát số process/task, không thay memory hoặc CPU limit.
4. Kiểm tra cấu hình thực tế bằng inspect và hành vi bằng metrics/load test.

## Tài liệu tham khảo

- Docker Docs, [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
- Docker CLI, [docker container run](https://docs.docker.com/reference/cli/docker/container/run/)
- Docker Docs, [Compose services](https://docs.docker.com/reference/compose-file/services/)

[← 3. Configuration và secret](03-configuration-va-secret.md) · [Mục lục Production](README.md) · [5. Health và restart →](05-health-va-restart.md)
