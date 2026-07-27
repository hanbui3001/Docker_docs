# 7. Production checklist

> **Tóm tắt một câu:** Checklist biến các nguyên tắc vận hành thành câu hỏi kiểm chứng được trước deploy, nhưng không thay thế review kiến trúc, load test và ownership rõ ràng.

> **Loại:** Explanation/Review gate · **Cấp độ:** Intermediate · **Thời gian:** Khoảng 35 phút  
> **Nguồn chính:** [Compose production considerations](https://docs.docker.com/compose/production/) · [Docker Engine security](https://docs.docker.com/engine/security/)

> **Sau chapter này, bạn có thể:**
> - Review workload theo artifact, config, security, resources, data và observability.
> - Gắn mỗi mục quan trọng với owner, bằng chứng và thời điểm kiểm tra.
> - Phân biệt “đã khai báo” với “đã áp dụng và hoạt động đúng”.
> - Nhận ra khi yêu cầu production vượt giới hạn của Compose single-host.

[← 6. Logging và observability](06-logging-va-observability.md) · [Mục lục Production](README.md) · [Part 08. Troubleshooting →](../08-troubleshooting/README.md)

---

## 1. Một ô checklist cần những gì?

Một mục chỉ đáng tin khi có bốn thành phần:

```text
Requirement -> Owner -> Evidence -> Freshness
```

- **Requirement** — điều cần đúng, ví dụ Container có memory limit.
- **Owner** — người/nhóm chịu trách nhiệm khi bằng chứng sai hoặc hết hạn.
- **Evidence** — command output, test result, dashboard, backup restore record hoặc tài liệu kiến trúc.
- **Freshness** — thời điểm bằng chứng còn đại diện cho release hiện tại.

Ví dụ “đã đặt memory limit” chưa hoàn thành nếu chỉ trỏ vào Compose file. Bằng chứng mạnh hơn là resolved model, runtime inspect và load test của Image/version sắp deploy.

## 2. Artifact và delivery

- [ ] Image có version rõ ràng; deploy quan trọng có thể đối chiếu digest.
- [ ] Có liên kết từ Image/version tới source revision và build pipeline.
- [ ] Base Image và dependency có nguồn tin cậy, có quy trình cập nhật.
- [ ] Có rollback artifact đã kiểm chứng, không phụ thuộc build lại khẩn cấp.

Kiểm chứng mẫu:

```bash
docker image inspect registry.example.com/team/api:1.4.2 --format 'Id={{.Id}} Digests={{json .RepoDigests}}'
docker container inspect api --format 'RuntimeImage={{.Image}}'
```

Lệnh đầu kiểm tra Image local và repository digest đã biết; lệnh sau cho biết Image ID Container dùng lúc create. Muốn chứng minh Registry remote hoặc deployment khác, cần thêm registry/deployment evidence phù hợp; inspect local không tự đại diện toàn hệ thống.

## 3. Runtime configuration và secret

- [ ] Config theo môi trường không bị đóng cứng vào Image.
- [ ] Secret không nằm trong Dockerfile, repository, command line hoặc log.
- [ ] Owner, rotation, expiration và recovery của secret được xác định.
- [ ] `docker compose config` được review mà không để output nhạy cảm rò rỉ vào log.
- [ ] Cách ứng dụng reload config/secret đã rõ: dynamic reload hay restart/recreate.

```bash
docker compose config --services
docker compose config --images
docker container inspect api --format 'EnvCount={{len .Config.Env}} Mounts={{json .Mounts}}'
```

Không in toàn bộ `.Config.Env` trong môi trường có secret. Review nên kiểm tra tên/source và policy mà không phát tán giá trị.

## 4. Security boundary

- [ ] Process chạy non-root khi ứng dụng cho phép.
- [ ] File và mount có owner/mode tối thiểu cần thiết.
- [ ] Không dùng `privileged` hoặc Docker socket nếu không có lý do được review.
- [ ] Capability được giảm; root filesystem read-only khi khả thi.
- [ ] Port chỉ publish trên interface cần thiết.

```bash
docker container inspect api --format 'User={{json .Config.User}} Privileged={{.HostConfig.Privileged}} Readonly={{.HostConfig.ReadonlyRootfs}} CapAdd={{json .HostConfig.CapAdd}} CapDrop={{json .HostConfig.CapDrop}}'
docker container exec api id
```

Image `USER` và effective identity có thể khác do runtime override. Cần đọc cả metadata lẫn process thực tế.

## 5. Resource và capacity

- [ ] Có CPU, memory và PID limit dựa trên đo đạc.
- [ ] JVM/runtime memory setting chừa headroom cho native memory.
- [ ] Load test bao gồm peak, dependency chậm và restart.
- [ ] Alert phân biệt saturation, throttling và application error.
- [ ] Tổng capacity của host có headroom cho daemon, kernel và failure/restart spike.

```bash
docker container inspect api --format 'Memory={{.HostConfig.Memory}} NanoCpus={{.HostConfig.NanoCpus}} Pids={{.HostConfig.PidsLimit}} OOM={{.State.OOMKilled}}'
docker stats --no-stream api
```

`stats --no-stream` chỉ là snapshot. Bằng chứng capacity cần metric lịch sử và load profile đại diện, không chỉ output tại thời điểm nhàn rỗi.

## 6. Data và network

- [ ] Dữ liệu cần bền không nằm duy nhất trong writable layer.
- [ ] Volume/bind mount được backup và restore thử nghiệm.
- [ ] Host path, permission và ownership được kiểm tra trên môi trường đích.
- [ ] Service dùng DNS name/network đúng scope, không giả định `localhost` là Container khác.
- [ ] Timeout và retry có backoff/jitter; không retry vô hạn.
- [ ] Backup nằm ngoài failure domain của dữ liệu nguồn và có retention phù hợp.

```bash
docker container inspect api --format 'Mounts={{json .Mounts}} Networks={{json .NetworkSettings.Networks}}'
docker compose exec api getent hosts database
```

Lệnh DNS phụ thuộc utility `getent` có trong Image. Backup chỉ được coi là dùng được sau restore test vào môi trường an toàn và kiểm tra dữ liệu/application behavior.

## 7. Health, logs và recovery

- [ ] Healthcheck dùng command có thật trong Image và có timing hợp lý.
- [ ] Restart policy phù hợp; restart loop có alert và log đủ để điều tra.
- [ ] Log ra stdout/stderr, có rotation và không chứa secret.
- [ ] Version, Service, environment và correlation ID xuất hiện trong telemetry.
- [ ] Runbook nêu cách xem state, log, network, mounts và rollback.
- [ ] Recovery có bước verification bằng request, metric hoặc data check, không dừng ở “đã restart”.

```bash
docker container inspect api --format 'Health={{json .State.Health}} Restart={{json .HostConfig.RestartPolicy}} Count={{.RestartCount}}'
docker container inspect api --format 'LogConfig={{json .HostConfig.LogConfig}}'
docker container logs --since 10m --tail 200 --timestamps api
```

Health output và logs có thể chứa dữ liệu nhạy cảm. Chỉ chia sẻ phần cần thiết và áp dụng redaction.

## 8. Release gate tối thiểu

Trước deploy, ghi lại một bảng ngắn thay vì chỉ tick:

| Mục | Owner | Evidence | Kết quả |
|---|---|---|---|
| Runtime Image | Release team | Digest/Container Image ID | Pass/Fail |
| Security context | Service owner | Inspect + `id` | Pass/Fail |
| Resource limits | Platform owner | HostConfig + load test | Pass/Fail |
| Persistent data | Data owner | Restore record | Pass/Fail |
| Health/recovery | Service owner | Failure test + metrics | Pass/Fail |
| Logging | Operations | Driver/rotation/query test | Pass/Fail |

**Release gate** — điều kiện phải đạt trước khi cho phép phát hành. Gate nên có tiêu chí fail rõ và người có quyền quyết định exception; nếu mọi failure đều được bỏ qua, gate chỉ còn hình thức.

## 9. Compose có giới hạn gì?

Compose rất phù hợp cho local development, test, single-host deployment và mô tả stack có quy mô phù hợp. Nó không tự cung cấp multi-host scheduling, rolling update phức tạp, autoscaling, distributed secret management hay traffic management như một orchestrator đầy đủ.

**Failure domain** của single-host Compose thường bao gồm chính host đó: host mất điện, disk lỗi hoặc daemon hỏng có thể ảnh hưởng toàn stack. Restart policy trên cùng host không tạo high availability nếu host biến mất.

Không phải mọi production workload đều cần Kubernetes; nhưng dùng Compose phải là quyết định dựa trên availability, scale, team operation và failure domain, không phải vì YAML đã chạy được ở local.

## 10. Quan niệm dễ sai

### “Tick đủ checklist nghĩa release chắc chắn không lỗi.”

- **Phân loại:** Sai.
- **Vì sao nghe hợp lý:** Checklist bao phủ các failure mode đã biết.
- **Lỗi kỹ thuật:** Checklist không dự đoán mọi tương tác, tải hoặc lỗi mới; bằng chứng cũng có thể cũ.
- **Cách nói tốt hơn:** Checklist giảm lỗi có thể phòng tránh và làm risk rõ hơn, không xóa uncertainty.

### “Compose chạy ổn trên một host nên đã có high availability.”

- **Phân loại:** Sai.
- **Lỗi kỹ thuật:** Mọi Container có thể cùng nằm trong một host failure domain.
- **Kiểm chứng:** Mô phỏng mất host/daemon và xác định Service còn endpoint ở đâu.

## 11. Bài kiểm tra cuối Part

Hãy chọn một Service thật và trả lời bằng bằng chứng:

1. Image digest hoặc Image ID đang chạy là gì?
2. Process chạy UID nào?
3. Memory/CPU/PID limit thực tế là bao nhiêu?
4. Dữ liệu nào biến mất khi recreate và restore gần nhất khi nào?
5. Healthcheck kiểm tra điều gì và tool kiểm tra có trong Image không?
6. Khi dependency mất 30 giây, ứng dụng phản ứng ra sao?
7. Log đi qua driver/backend nào, rotate/retain thế nào và có correlation ID không?
8. Nếu host mất hoàn toàn, recovery target và owner là ai?

Nếu câu trả lời chỉ là “chắc là”, workload chưa có đủ bằng chứng vận hành.

## 12. Tóm tắt

1. Checklist phải gắn với requirement, owner, evidence và freshness.
2. Desired configuration phải được đối chiếu với effective runtime state.
3. Production readiness là thuộc tính của toàn hệ thống, không chỉ Dockerfile.
4. Part 08 sẽ dùng chính các lớp này làm thứ tự điều tra khi hệ thống sai lệch.

## Tài liệu tham khảo

- Docker Docs, [Production considerations](https://docs.docker.com/compose/production/)
- Docker Docs, [Engine security](https://docs.docker.com/engine/security/)
- Docker Docs, [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)

[← 6. Logging và observability](06-logging-va-observability.md) · [Mục lục Production](README.md) · [Part 08. Troubleshooting →](../08-troubleshooting/README.md)
