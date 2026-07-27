# Complete Docker Learning Path Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Hoàn thiện Part 02 đến Part 08 của bộ tài liệu Docker tiếng Việt theo chuẩn nội dung, điều hướng và trình bày đã được Part 01 thiết lập.

**Architecture:** Mỗi Part là một đơn vị học tập độc lập gồm README định hướng và các chapter tập trung vào một chủ đề. Nội dung Learning Path giải thích mental model và cơ chế; tutorial, how-to và reference chỉ được tạo khi chúng bổ sung đúng vai trò Diátaxis, không sao chép nguyên chapter.

**Tech Stack:** GitHub Flavored Markdown, Mermaid, Docker CLI, Dockerfile, Docker Compose.

## Global Constraints

- Viết trực tiếp trong `D:\work\Project\Tool_Theory_Project\docker-document`; không stage, commit, push hay cấu hình Git.
- Giữ tiếng Anh cho technical term và giải thích bằng tiếng Việt tại lần xuất hiện đầu tiên hoặc trong hộp thuật ngữ cục bộ.
- Chỉ thêm thuật ngữ mới có giá trị tái sử dụng vào `reference/glossary.md`; không dùng danh sách cố định.
- Mỗi syntax quan trọng phải giải thích token, scope, giá trị resolve, trạng thái trước/sau, lỗi thường gặp và cách kiểm chứng.
- Dùng Mermaid khi quan hệ, lifecycle hoặc data flow khó hiểu nếu chỉ diễn đạt bằng văn bản.
- Mỗi chapter có learning outcomes, misconception hợp lý, tóm tắt và điều hướng Previous/Part/Next.
- Ví dụ phải dùng được cho nhiều dự án; Gearhouse chỉ là case study.
- Kiểm tra cuối chỉ gồm UTF-8, H1, code fence, relative link/anchor và placeholder.

---

### Task 1: Part 02 — Docker CLI & Lifecycle

**Files:** Tạo `learning-path/02-cli-and-lifecycle/` và `reference/commands/`; cập nhật mục lục gốc.

- [ ] Viết command grammar, Image commands, tạo/chạy Container, lifecycle, quan sát/debug và cleanup.
- [ ] Tạo CLI quick reference và các reference page cho `pull`, `run`, `ps`, `logs`, `exec`.
- [ ] Phân biệt PowerShell/Bash và cảnh báo rõ các lệnh destructive.

### Task 2: Part 03 — Storage & Networking

**Files:** Tạo `learning-path/03-storage-and-networking/`.

- [ ] Giải thích writable layer, Volume, bind mount, tmpfs và tiêu chí lựa chọn.
- [ ] Giải thích network namespace, bridge network, port publishing, DNS/service discovery và luồng request.
- [ ] Thêm ví dụ kiểm chứng dữ liệu tồn tại và giao tiếp giữa Container.

### Task 3: Part 04 — Dockerfile

**Files:** Tạo `learning-path/04-dockerfile/`, tutorial Java và `reference/dockerfile/`.

- [ ] Viết build context, instruction, layer/cache, multi-stage build, Java/JAR/JDK/runtime, security và build flow hoàn chỉnh.
- [ ] Giải thích sâu source/destination của `COPY`, exec/shell form, `CMD`/`ENTRYPOINT`, và JDK so với runtime Image.
- [ ] Tạo tutorial Gradle và Maven có verification/cleanup.

### Task 4: Part 05 — Docker Compose

**Files:** Tạo `learning-path/05-docker-compose/` và `reference/compose/`.

- [ ] Viết Compose model, YAML, service/build/image, ports/network, environment, Volume, healthcheck, dependency và lifecycle CLI.
- [ ] Phân tích sâu short/long syntax, interpolation, `depends_on` conditions và healthcheck token.
- [ ] Tạo reference cho Compose keys và commands.

### Task 5: Part 06 — Registry & Delivery

**Files:** Tạo `learning-path/06-registry-and-delivery/`.

- [ ] Giải thích Image reference, Registry/Repository, authentication, tag/digest, push/pull và multi-platform Image.
- [ ] Trình bày delivery flow từ build đến deploy, versioning và kiểm chứng artifact.

### Task 6: Part 07 — Production

**Files:** Tạo `learning-path/07-production/`.

- [ ] Viết runtime security, user/permission, configuration/secrets, resource limits, health/restart, logs/observability và checklist production.
- [ ] Giải thích trade-off và giới hạn của Compose trong production.

### Task 7: Part 08 — Troubleshooting

**Files:** Tạo `learning-path/08-troubleshooting/` và các how-to chẩn đoán quan trọng.

- [ ] Dạy phương pháp chẩn đoán theo layer: CLI -> object state -> process -> network -> storage -> host.
- [ ] Viết playbook cho build, startup, port/network, storage/permission, Compose và disk pressure.
- [ ] Mỗi lỗi có symptom, hypothesis, evidence command, correction và verification.

### Task 8: Integration

**Files:** Sửa `README.md`, `reference/glossary.md` và các liên kết điều hướng.

- [ ] Cập nhật book roadmap và đánh dấu toàn bộ Part đã có nội dung.
- [ ] Bổ sung các thuật ngữ mới quan trọng phát sinh trong Part 02–08 cùng backlink phù hợp.
- [ ] Chạy kiểm tra Markdown tối thiểu và sửa lỗi thật sự ảnh hưởng việc đọc.
