# Docker Documentation First Slice Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Khởi tạo repository tài liệu Docker độc lập và xuất bản slice đầu tiên gồm format Markdown chung, README tổng, README Foundation, Glossary và chapter mẫu `Docker Image` đạt chất lượng production.

**Architecture:** Repository dùng GitHub-Flavored Markdown làm định dạng xuất bản chính, giữ nội dung tương thích với MkDocs trong tương lai nhưng không cài framework ở slice đầu. `03-docker-image.md` là canonical Explanation chapter; các tài liệu sau phải tái sử dụng quy chuẩn thuật ngữ, misconception, diagram, syntax deep dive và navigation được kiểm chứng trong chapter này.

**Tech Stack:** Git, GitHub-Flavored Markdown, Mermaid, Docker CLI 29.6.1+, PowerShell 7+, Docker Official Documentation, OCI Image Specification.

## Global Constraints

- Target repository: `D:\work\Project\Tool_Theory_Project\docker-document`.
- Authoritative source spec: `D:\work\Project\springboot_project\Gearhouse_e_commnerce\docs\superpowers\specs\2026-07-27-docker-documentation-design.md`.
- Source spec SHA-256 at planning time: `890B682E944A9CDA8305EE5A3B82A65607EFD62DF9E2CCE5320F6AE1DA955DEE`.
- Use GitHub-Flavored Markdown; do not add MkDocs, Docusaurus, package managers, or generated HTML in this slice.
- Use lowercase ASCII kebab-case filenames; Vietnamese prose remains fully accented.
- Use exactly one H1 per Markdown file and do not skip heading levels.
- Keep planned chapters as plain-text roadmap entries until their files exist; never create broken links intentionally.
- Use Mermaid instead of third-party hot-linked diagrams.
- Use only `NOTE`, `TIP`, `IMPORTANT`, and `WARNING` callouts unless the spec justifies another type.
- Explanation chapters use numbered titles and numbered major sections, inspired by the reviewed NGINX docs.
- Every compact command or syntax example must explain tokens, scope, state change, likely misconception, and verification.
- Use Docker Official Documentation as the primary source and OCI specifications only where lower-level precision is needed.
- Gearhouse may appear only as an example or case-study reference; reusable theory must stand independently.
- Do not add empty future directories or placeholder files.
- Do not write `TODO`, `TBD`, `FIXME`, or fake links in published Markdown.
- Commit after every content-producing task using the exact commit message listed in that task. Validation-only work must not create an empty commit.

---

## File Map

| File | Responsibility |
|---|---|
| `.gitignore` | Exclude editor, OS, log, and temporary artifacts without hiding documentation source. |
| `README.md` | Book cover, audience, Diataxis navigation, learning roadmap, master table of contents, conventions, and current completion status. |
| `STYLE-GUIDE.md` | Canonical Markdown, terminology, NGINX-inspired visual rhythm, syntax deep-dive, callout, diagram, navigation, and source rules. |
| `learning-path/01-foundations/README.md` | Foundation scope, prerequisites, chapter roadmap, reading order, exclusions, and completion checklist. |
| `learning-path/01-foundations/03-docker-image.md` | Production-quality canonical Explanation chapter for Docker Image. |
| `reference/glossary.md` | Shared beginner-friendly definitions and stable anchors for Docker terminology used by the first slice. |
| `docs/superpowers/specs/2026-07-27-docker-documentation-design.md` | Self-contained copy of the approved design spec in the new repository. |
| `docs/superpowers/plans/2026-07-27-docker-documentation-first-slice.md` | This implementation plan and execution checklist. |

---

### Task 1: Initialize the Dedicated Documentation Repository

**Files:**
- Create: `.gitignore`
- Create: `docs/superpowers/specs/2026-07-27-docker-documentation-design.md`
- Track: `docs/superpowers/plans/2026-07-27-docker-documentation-first-slice.md`

**Interfaces:**
- Consumes: Approved source spec at the absolute path and SHA-256 in Global Constraints.
- Produces: A `main` Git repository containing the authoritative spec and plan for every later task.

- [ ] **Step 1: Verify the target is safe to initialize**

Run:

```powershell
$repo = 'D:\work\Project\Tool_Theory_Project\docker-document'
Resolve-Path $repo
Get-ChildItem -Force $repo
Test-Path (Join-Path $repo '.git')
```

Expected: resolved path is exactly the target repository, only planning directories created by this session are present, and `.git` returns `False`.

- [ ] **Step 2: Initialize Git with `main` as the default branch**

Run:

```powershell
git init -b main 'D:\work\Project\Tool_Theory_Project\docker-document'
git -C 'D:\work\Project\Tool_Theory_Project\docker-document' status --short
```

Expected: Git reports an empty repository on branch `main`; the plan file is untracked.

- [ ] **Step 3: Create the repository `.gitignore`**

Create `.gitignore` with exactly:

```gitignore
.idea/
.tmp/
.DS_Store
Thumbs.db
*.log
```

Do not ignore `.vscode/`, Markdown files, diagrams, or documentation configuration because those may become shared project assets later.

- [ ] **Step 4: Copy the approved spec into the new repository**

Run:

```powershell
$source = 'D:\work\Project\springboot_project\Gearhouse_e_commnerce\docs\superpowers\specs\2026-07-27-docker-documentation-design.md'
$destination = 'D:\work\Project\Tool_Theory_Project\docker-document\docs\superpowers\specs\2026-07-27-docker-documentation-design.md'
New-Item -ItemType Directory -Force (Split-Path $destination) | Out-Null
Copy-Item -LiteralPath $source -Destination $destination
```

- [ ] **Step 5: Verify the copied spec is byte-identical**

Run:

```powershell
$sourceHash = (Get-FileHash 'D:\work\Project\springboot_project\Gearhouse_e_commnerce\docs\superpowers\specs\2026-07-27-docker-documentation-design.md' -Algorithm SHA256).Hash
$targetHash = (Get-FileHash 'D:\work\Project\Tool_Theory_Project\docker-document\docs\superpowers\specs\2026-07-27-docker-documentation-design.md' -Algorithm SHA256).Hash
$sourceHash
$targetHash
$sourceHash -eq $targetHash
```

Expected: both hashes equal the Global Constraints hash and the final expression returns `True`.

- [ ] **Step 6: Commit repository governance files**

Run:

```powershell
git add .gitignore docs/superpowers/specs/2026-07-27-docker-documentation-design.md docs/superpowers/plans/2026-07-27-docker-documentation-first-slice.md
git diff --cached --check
git commit -m "docs: initialize Docker documentation repository"
```

Expected: one root commit containing only `.gitignore`, the approved spec, and this plan.

---

### Task 2: Create the Markdown Style Guide

**Files:**
- Create: `STYLE-GUIDE.md`

**Interfaces:**
- Consumes: Presentation rules from spec sections 5-13 and 22.
- Produces: A reusable editorial contract referenced by every content task.

- [ ] **Step 1: Create the style guide title and purpose**

Start the file with:

```markdown
# Docker Documentation Style Guide

> Quy chuẩn biên tập cho toàn bộ tài liệu trong repository. Mục tiêu là tạo
> nội dung dễ học với người mới, chính xác về kỹ thuật và nhất quán khi tra cứu.

## 1. Nguyên tắc chung
```

Under section 1, state that content is Vietnamese-first, keeps original English technical terms, uses official sources, and prefers accurate mental models over memorized commands.

- [ ] **Step 2: Document the four Diataxis formats**

Add a table with rows for Tutorial, How-to, Explanation, and Reference. Include each type's user question, required flow, and prohibited content mixing.

Use these exact primary flows:

```text
Tutorial: prerequisites -> steps -> observation -> verification -> cleanup
How-to: problem -> current-state check -> change -> verification -> recovery
Explanation: problem -> intuition -> precise model -> mechanism -> misconception
Reference: quick table -> exact syntax -> options -> examples -> related entries
```

- [ ] **Step 3: Document the NGINX-inspired visual rhythm**

Require numbered H1 titles, numbered major sections, a primary-source block near the top, horizontal rules between major reading phases, nearby concrete examples, concise comparison tables, and bottom summaries.

Explicitly prohibit excessive bold text, unsupported absolute claims, and third-party hot-linked images.

- [ ] **Step 4: Add the canonical Explanation skeleton**

Include this copyable skeleton with no empty placeholder markers:

```markdown
# 3. Tên chapter

> **Tóm tắt một câu:** Mental model chính xác của chapter.

> **Loại:** Explanation · **Cấp độ:** Beginner · **Thời gian:** Khoảng 25 phút<br>
> **Nguồn chính:** Link tài liệu chính thức

[Mục lục phần](README.md)

---

## 1. Vấn đề cần giải quyết
## 2. Hiểu nhanh
## 3. Định nghĩa chính xác
## 4. Cơ chế hoạt động
## 5. Ví dụ quan sát
## 6. Quan niệm dễ gây hiểu nhầm
## 7. Tự kiểm tra mental model
## 8. Tóm tắt
## 9. Học tiếp
## Tài liệu tham khảo
```

Immediately explain that previous and next links are added only when their target files already exist. The part index link is always present.

- [ ] **Step 5: Add the Syntax Deep-Dive template**

Document the required order:

```text
Syntax -> syntax tree -> token table -> resolved values -> before/after state
-> similar syntax -> mistakes -> misconceptions -> verification
```

Include the `WORKDIR /app` plus `COPY app.jar app.jar` example and explain why the two `app.jar` values belong to different filesystems.

- [ ] **Step 6: Add terminology, callout, code, diagram, naming, and navigation rules**

Include:

- First-use format: `**Writable layer** — lớp cho phép Container ghi dữ liệu trong quá trình chạy.`
- Allowed callouts: `NOTE`, `TIP`, `IMPORTANT`, `WARNING`.
- Required fence labels: `bash`, `powershell`, `dockerfile`, `yaml`, `mermaid`, `text`.
- Lowercase ASCII kebab-case filenames.
- One H1, no skipped heading levels, navigation at top and bottom.
- Relative repository links only for internal content.

- [ ] **Step 7: Verify the style guide**

Run:

```powershell
$file = 'STYLE-GUIDE.md'
(Select-String -Path $file -Pattern '^# ').Count
Select-String -Path $file -Pattern '^## '
Select-String -Path $file -Pattern '\b(TODO|TBD|FIXME)\b'
```

Expected: exactly one H1, all required H2 sections are listed, and the placeholder scan returns no output.

- [ ] **Step 8: Commit the style guide**

```powershell
git add STYLE-GUIDE.md
git diff --cached --check
git commit -m "docs: define Markdown editorial style"
```

---

### Task 3: Create the Shared Docker Glossary

**Files:**
- Create: `reference/glossary.md`

**Interfaces:**
- Consumes: Terminology strategy from the spec and `STYLE-GUIDE.md`.
- Produces: Stable anchors consumed by the Foundation README and Image chapter.

- [ ] **Step 1: Create the glossary introduction and navigation**

Use:

```markdown
# Docker Glossary

> Từ điển thuật ngữ dùng chung cho bộ tài liệu. Mỗi định nghĩa bắt đầu bằng
> cách hiểu nhanh, sau đó mới đi tới ý nghĩa kỹ thuật chính xác.

[← Về mục lục chính](../README.md)

---
```

- [ ] **Step 2: Add the initial terms in alphabetical order**

Create complete H2 entries for:

```text
Build context
Container
Daemon
Digest
Dockerfile
Filesystem
Filesystem layer
Image
Image configuration
Instance
Metadata
Registry
Repository
Tag
Writable layer
```

Every term must contain:

1. `**Cách hiểu nhanh:**` one beginner-friendly sentence.
2. `**Định nghĩa chính xác:**` one or two technically precise paragraphs.
3. `**Ví dụ:**` a Docker-specific example.
4. `**Liên quan:**` relative anchor links to two or three related glossary terms.

- [ ] **Step 3: Add an important Tag versus Digest cross-reference**

Under `Tag`, state that a tag is a mutable human-readable reference. Under `Digest`, state that it identifies specific content. Neither definition may claim that equal tags guarantee equal Image content over time.

- [ ] **Step 4: Verify glossary anchors and completeness**

Run:

```powershell
$file = 'reference/glossary.md'
$required = @(
  'Build context','Container','Daemon','Digest','Dockerfile','Filesystem',
  'Filesystem layer','Image','Image configuration','Instance','Metadata',
  'Registry','Repository','Tag','Writable layer'
)
$headings = Select-String -Path $file -Pattern '^## ' | ForEach-Object { $_.Line.Substring(3) }
Compare-Object $required $headings
```

Expected: no output from `Compare-Object`.

- [ ] **Step 5: Commit the glossary**

```powershell
git add reference/glossary.md
git diff --cached --check
git commit -m "docs: add foundational Docker glossary"
```

---

### Task 4: Create the Foundation Part README

**Files:**
- Create: `learning-path/01-foundations/README.md`

**Interfaces:**
- Consumes: Root architecture from the spec and glossary anchors from Task 3.
- Produces: The Foundation landing page linked by the root README and Image chapter.

- [ ] **Step 1: Write the part identity and scope**

Use title `# Part 01. Docker Foundations` and a one-sentence summary stating that the part builds the mental model required by every later Docker topic.

Add compact metadata:

```text
Loại: Learning path index
Cấp độ: Beginner
Điều kiện: Không yêu cầu kiến thức Docker trước đó
```

- [ ] **Step 2: Define what Foundation covers and excludes**

Cover Docker, architecture, Image, Container, their relationship, and an ecosystem overview. State explicitly that Volume, Network, Registry delivery, Dockerfile syntax, Compose syntax, and production operations are introduced only briefly or deferred to later parts.

- [ ] **Step 3: Add the chapter roadmap without broken links**

Use a status table. At this task the Image chapter does not exist yet, so keep it as plain text:

```markdown
| Chapter | Trạng thái |
|---|---|
| 1. Docker là gì? | Đã lên kế hoạch |
| 2. Docker hoạt động như thế nào? | Đã lên kế hoạch |
| 3. Docker Image | Đang triển khai |
| 4. Docker Container | Đã lên kế hoạch |
| 5. Image và Container | Đã lên kế hoạch |
| 6. Bức tranh tổng thể | Đã lên kế hoạch |
```

- [ ] **Step 4: Add reading guidance and completion checklist**

Explain that Image is available first because it validates the canonical Explanation format, not because learners should permanently skip chapters 1 and 2. Add a checklist based on explaining concepts in the learner's own words, not memorizing definitions.

- [ ] **Step 5: Add navigation**

Top and bottom navigation must link only to the repository root. Do not link to any unimplemented chapter.

- [ ] **Step 6: Verify links and commit**

Run:

```powershell
Select-String -Path 'learning-path/01-foundations/README.md' -Pattern '\]\([^)]*\.md\)'
```

Expected: every listed Markdown link resolves to an existing file; no Image chapter link exists yet.

Commit:

```powershell
git add learning-path/01-foundations/README.md
git diff --cached --check
git commit -m "docs: add Foundation learning roadmap"
```

---

### Task 5: Create the Root Book README

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: Foundation README from Task 4, glossary from Task 3, and the approved eight-part roadmap.
- Produces: Repository homepage and master navigation entry.

- [ ] **Step 1: Write the book cover**

Use:

```markdown
# Docker: Từ nền tảng đến vận hành

> Tài liệu Docker bằng tiếng Việt, tập trung vào mental model, syntax,
> cách kiểm chứng và những quan niệm dễ gây hiểu nhầm.
```

Do not add marketing badges or framework badges.

- [ ] **Step 2: Add audience, outcomes, and usage modes**

Include a table mapping these needs:

| Nhu cầu | Khu vực |
|---|---|
| Học từ đầu | Learning Path |
| Thực hành có hướng dẫn | Tutorials |
| Hoàn thành một công việc cụ thể | How-to |
| Tra cứu syntax | Reference |
| Xem lỗi và cách sửa thực tế | Case Studies |

- [ ] **Step 3: Add the eight-part Mermaid roadmap**

Use one left-to-right Mermaid flow covering Foundations, CLI & Lifecycle, Storage & Networking, Dockerfile, Docker Compose, Registry & Delivery, Production, and Troubleshooting.

Add prose explaining the progression: understand, control, persist/connect, package, coordinate, distribute, operate, diagnose.

- [ ] **Step 4: Add the master table of contents**

Link to:

- `learning-path/01-foundations/README.md`
- `reference/glossary.md`
- `STYLE-GUIDE.md`

List the Docker Image chapter and Parts 02-08 as unlinked roadmap entries. Use `Đang triển khai` for Docker Image and `Thiết kế đã được duyệt; chưa triển khai` for Parts 02-08.

- [ ] **Step 5: Add repository conventions and status**

Explain numbered chapters, English term preservation, syntax deep dives, misconceptions, diagrams, and official sources. Add a completion table that accurately states only the first slice is in progress.

- [ ] **Step 6: Verify and commit**

Run:

```powershell
Test-Path 'learning-path/01-foundations/README.md'
Test-Path 'reference/glossary.md'
Test-Path 'STYLE-GUIDE.md'
Select-String -Path README.md -Pattern '```mermaid'
```

Expected: all paths return `True` and one Mermaid block is found.

Commit:

```powershell
git add README.md
git diff --cached --check
git commit -m "docs: add Docker book landing page"
```

---

### Task 6: Write the Canonical Docker Image Chapter

**Files:**
- Create: `learning-path/01-foundations/03-docker-image.md`
- Modify: `README.md`
- Modify: `learning-path/01-foundations/README.md`

**Interfaces:**
- Consumes: Style guide, Foundation navigation, and glossary anchors.
- Produces: The canonical Explanation format reused by later Docker, Container, Dockerfile, and Compose theory chapters.

- [ ] **Step 1: Verify primary sources before drafting**

Open and read these sources completely enough to validate the sections they support:

```text
https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/
https://docs.docker.com/get-started/docker-concepts/building-images/understanding-image-layers/
https://docs.docker.com/reference/cli/docker/image/
https://github.com/opencontainers/image-spec/blob/main/spec.md
```

Record only claims supported by current official behavior. Secondary articles may identify misconceptions but must not override these sources.

- [ ] **Step 2: Write the chapter header and navigation**

Use H1 `# 3. Docker Image`, a one-sentence mental model, compact metadata, primary-source links, and navigation to the Foundation README. Because chapters 2 and 4 do not exist, do not create previous/next links to them; use only `Mục lục Foundation` at this stage.

- [ ] **Step 3: Write section 1, the motivating problem**

Explain why Spring Boot source code alone is insufficient to reproduce a running environment: Java runtime version, dependencies, resources, filesystem content, and startup configuration also matter.

Avoid claiming that every Image contains a complete operating system.

- [ ] **Step 4: Write section 2, intuitive and precise definitions**

Include:

- A beginner analogy with its limitation.
- A precise definition of Image as read-only content plus configuration used to create Containers.
- First-use explanations and glossary links for Image, filesystem, filesystem layer, metadata, image configuration, and instance.

- [ ] **Step 5: Write section 3, practical purposes**

Cover reproducibility, distribution, versioned deployment inputs, local/CI consistency, and one-Image-to-many-Containers behavior. Do not turn the section into Registry or CI/CD instruction.

- [ ] **Step 6: Write section 4, Image anatomy**

Explain filesystem layers and image configuration separately. Include a Mermaid layer-stack diagram and prose explaining every level.

Explain that not every Dockerfile instruction creates a filesystem layer and defer full Dockerfile cache mechanics to Part 04.

- [ ] **Step 7: Write section 5, immutability, tag, and digest**

Explain that existing Image content is not edited in place, rebuilding creates new content, tags can move, and digests identify specific content.

Include a misconception warning against the sentence `Docker Image hoàn toàn không bao giờ thay đổi` without qualification.

- [ ] **Step 8: Write section 6, Image lifecycle and flow**

Add a Mermaid flow showing:

```text
Dockerfile -> docker build -> local Image
Registry -> docker pull -> local Image
local Image -> docker run -> Container
local Image -> docker push -> Registry
```

Explain each arrow in prose and keep Dockerfile/Registry mechanics introductory.

- [ ] **Step 9: Write section 7, observational commands with syntax deep dives**

Use these commands:

```bash
docker image pull nginx:alpine
docker image ls
docker image inspect nginx:alpine
docker image history nginx:alpine
docker container run --name image-demo --detach nginx:alpine
docker container rm --force image-demo
```

For every command, explain:

- Object and action.
- Options and arguments.
- Output fields to observe.
- What theoretical statement the command demonstrates.
- Cleanup behavior.

Do not expose the Nginx application as a tutorial; the Container exists only to demonstrate that one Image can create a runtime instance.

- [ ] **Step 10: Write section 8, Image and Container relationship**

Add a one-to-many Mermaid diagram. State that Image has no running process or Container lifecycle state, while each Container has its own runtime state and writable layer.

- [ ] **Step 11: Write section 9, detailed misconceptions**

Analyze at least these four plausible claims using classification, why it sounds correct, technical flaw, corrected wording, and verification:

1. `Image là Container chưa chạy.`
2. `Image chỉ là một file ZIP chứa ứng dụng.`
3. `Hai Image có cùng tag chắc chắn là cùng một Image.`
4. `Mỗi Container cần một Image riêng.`

Add `Xóa Container cũng xóa Image` only if the chapter remains within the target reading length.

- [ ] **Step 12: Write self-check, summary, next reading, and sources**

Add five mental-model questions that cannot be answered by copying a single sentence. End with five concise summary points, links to the Foundation README and Glossary, official sources, and bottom navigation.

- [ ] **Step 13: Verify reading depth and internal consistency**

Run:

```powershell
$file = 'learning-path/01-foundations/03-docker-image.md'
$wordCount = ((Get-Content -Raw $file) -split '\s+' | Where-Object { $_ }).Count
$wordCount
(Select-String -Path $file -Pattern '^# ').Count
(Select-String -Path $file -Pattern '```mermaid').Count
Select-String -Path $file -Pattern '\b(TODO|TBD|FIXME)\b'
```

Expected:

- Word count is between 2,500 and 4,500 words; use the upper tolerance because Vietnamese tokenization and code explanations vary.
- Exactly one H1.
- At least three Mermaid diagrams.
- No placeholder scan output.

- [ ] **Step 14: Verify Docker command examples against the installed CLI and Engine**

Run:

```powershell
docker version
docker image pull nginx:alpine
docker image inspect nginx:alpine --format '{{.Id}}'
docker container run --name image-demo --detach nginx:alpine
docker container inspect image-demo --format '{{.Image}}|{{.State.Status}}'
docker container rm --force image-demo
```

Expected: Docker Client and Server respond; pull succeeds; Image ID is printed; Container status is `running`; cleanup succeeds. If `docker version` cannot reach the Server, start Docker Desktop and rerun this step before claiming command verification.

- [ ] **Step 15: Publish the chapter links and commit**

Update both navigation indexes only after the chapter file exists:

- In `learning-path/01-foundations/README.md`, replace the plain Image row with `[3. Docker Image](03-docker-image.md)` and status `Có thể đọc`.
- In `README.md`, replace the plain Image roadmap entry with a relative link to `learning-path/01-foundations/03-docker-image.md` and mark it `Có thể đọc`.
- Confirm both links resolve before staging.

Commit:

```powershell
git add README.md learning-path/01-foundations/README.md learning-path/01-foundations/03-docker-image.md
git diff --cached --check
git commit -m "docs: add canonical Docker Image chapter"
```

---

### Task 7: Run Cross-Document Quality Verification

**Files:**
- Modify only if verification reveals a concrete defect: `README.md`, `STYLE-GUIDE.md`, `reference/glossary.md`, `learning-path/01-foundations/README.md`, `learning-path/01-foundations/03-docker-image.md`

**Interfaces:**
- Consumes: All first-slice documents.
- Produces: A link-clean, placeholder-free, internally consistent first slice ready for user review.

- [ ] **Step 1: Verify required files and repository cleanliness**

Run:

```powershell
$required = @(
  'README.md',
  'STYLE-GUIDE.md',
  'reference/glossary.md',
  'learning-path/01-foundations/README.md',
  'learning-path/01-foundations/03-docker-image.md',
  'docs/superpowers/specs/2026-07-27-docker-documentation-design.md',
  'docs/superpowers/plans/2026-07-27-docker-documentation-first-slice.md'
)
$required | ForEach-Object { [pscustomobject]@{ Path = $_; Exists = Test-Path $_ } }
git status --short
```

Expected: all `Exists` values are `True`; worktree is clean before final validation fixes.

- [ ] **Step 2: Scan every published Markdown file for placeholders and forbidden local links**

Run:

```powershell
$published = @(
  'README.md',
  'STYLE-GUIDE.md',
  'reference/glossary.md',
  'learning-path/01-foundations/README.md',
  'learning-path/01-foundations/03-docker-image.md'
)
Select-String -Path $published -Pattern '\b(TODO|TBD|FIXME)\b|file://|vscode://'
```

Expected: no output.

- [ ] **Step 3: Verify one H1 and balanced code fences**

Run:

```powershell
$published | ForEach-Object {
  $h1 = (Select-String -Path $_ -Pattern '^# ').Count
  $fences = (Select-String -Path $_ -Pattern '^```').Count
  [pscustomobject]@{ Path = $_; H1 = $h1; FenceCount = $fences; FencesBalanced = ($fences % 2 -eq 0) }
}
```

Expected: every file has `H1 = 1` and `FencesBalanced = True`.

- [ ] **Step 4: Verify relative Markdown link targets**

Run this PowerShell validation from the repository root:

```powershell
$errors = @()
$files = Get-ChildItem -Recurse -File -Filter '*.md' | Where-Object { $_.FullName -notmatch '\\.git\\' }
foreach ($file in $files) {
  $content = Get-Content -Raw $file.FullName
  $matches = [regex]::Matches($content, '\[[^\]]+\]\(([^)]+)\)')
  foreach ($match in $matches) {
    $target = $match.Groups[1].Value.Trim()
    if ($target -match '^(https?://|mailto:|#)' -or $target -eq '...') { continue }
    $pathOnly = ($target -split '#', 2)[0]
    if ([string]::IsNullOrWhiteSpace($pathOnly)) { continue }
    $resolved = [System.IO.Path]::GetFullPath((Join-Path $file.DirectoryName $pathOnly))
    if (-not (Test-Path -LiteralPath $resolved)) {
      $errors += "$($file.FullName): missing link target $target"
    }
  }
}
$errors
if ($errors.Count -gt 0) { exit 1 }
```

Expected: no output and exit code `0`.

- [ ] **Step 5: Verify official source URLs respond**

Run:

```powershell
$urls = @(
  'https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/',
  'https://docs.docker.com/get-started/docker-concepts/building-images/understanding-image-layers/',
  'https://docs.docker.com/reference/cli/docker/image/',
  'https://github.com/opencontainers/image-spec/blob/main/spec.md'
)
$urls | ForEach-Object {
  $response = Invoke-WebRequest -Uri $_ -Method Head -MaximumRedirection 5 -UseBasicParsing
  [pscustomobject]@{ Url = $_; Status = $response.StatusCode }
}
```

Expected: every URL returns a successful `2xx` status after redirects. If a server rejects `HEAD`, rerun that URL with `GET` and record the successful status.

- [ ] **Step 6: Review the first slice against the spec checklist**

Confirm explicitly:

- Root README acts as book cover and master TOC.
- Foundation README does not teach deferred subjects deeply.
- Image chapter follows Explanation, not Tutorial.
- English terms are introduced in Vietnamese and linked to Glossary.
- Every diagram is explained in prose.
- Misconceptions are plausible and evidence-based.
- Command examples explain syntax and state changes.
- No dangerous command appears without a warning and cleanup explanation.
- NGINX-inspired numbering and visual rhythm are present without copying unsupported claims or excessive formatting.

- [ ] **Step 7: Fix only concrete validation defects and commit if needed**

If Steps 1-6 required edits:

```powershell
git add README.md STYLE-GUIDE.md reference/glossary.md learning-path/01-foundations/README.md learning-path/01-foundations/03-docker-image.md
git diff --cached --check
git commit -m "docs: polish first Docker documentation slice"
```

If no edits were required, do not create an empty commit.

- [ ] **Step 8: Record final evidence for handoff**

Run:

```powershell
git log --oneline --decorate -10
git status --short
Get-ChildItem -Recurse -File |
  Where-Object { $_.FullName -notmatch '\\.git\\' } |
  Select-Object FullName,Length
```

Expected: task commits are visible, worktree is clean, and the file list matches the File Map.

---

## Out of Scope for This Plan

- Remaining Foundation chapters.
- Docker CLI chapter implementation and command cheat sheet.
- Dockerfile and Java/JAR chapters.
- Docker Compose chapters and tutorials.
- Gearhouse case-study content.
- MkDocs, Docusaurus, custom CSS, generated website, CI deployment, or GitHub Pages.
- Publishing or pushing the new repository to a remote.

Each of those items requires a later implementation plan after the canonical Image chapter and editorial format have been reviewed.
