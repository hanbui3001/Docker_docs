# Docker Documentation Design

## 1. Purpose

Build a reusable Vietnamese Docker documentation set for current and future projects. Gearhouse is a case study, not the boundary of the documentation. The design covers the documentation architecture and the editorial standards for Foundations, Docker CLI, Dockerfile, Java containerization, and Docker Compose.

The documentation must help a beginner:

- Build an accurate mental model instead of memorizing commands.
- Learn the English terminology used by Docker and its official documentation.
- Recognize explanations that sound reasonable but are technically misleading.
- Move from conceptual learning to tutorials, task-oriented guides, and reference material without duplicating content.

## 2. Documentation Architecture

Use a hybrid structure: a linear learning path for beginners combined with the four Diataxis content types for long-term lookup.

```text
docker/
|-- README.md
|-- learning-path/
|   |-- 01-foundations/
|   |-- 02-cli-and-lifecycle/
|   |-- 03-storage-and-networking/
|   |-- 04-dockerfile/
|   |-- 05-docker-compose/
|   |-- 06-registry-and-delivery/
|   |-- 07-production/
|   `-- 08-troubleshooting/
|-- tutorials/
|-- how-to/
|-- reference/
`-- case-studies/
    `-- gearhouse/
```

Navigation has three levels:

1. The root `README.md` acts as the book cover, introduction, learning roadmap, and master table of contents.
2. Each part has a `README.md` that defines its learning goals, prerequisites, chapter order, scope, and completion checklist.
3. Each chapter owns one focused topic and links to the previous chapter, the part index, and the next chapter.

## 3. Diataxis Boundaries

The learning path provides reading order, but each linked document must keep a clear primary purpose.

| Content type | Primary question | Examples |
|---|---|---|
| Tutorial | How can I learn this by doing? | Run the first Nginx container |
| How-to | How do I complete a specific task? | Inspect logs for a stopped container |
| Explanation | Why does Docker work this way? | What an Image is and how layers relate |
| Reference | What is the exact syntax or option? | `docker image ls` options |

Foundation chapters are explanation-first. They may contain short observational commands, but they must not turn into full tutorials or command references.

Every chapter links to the correct supporting documents instead of duplicating them:

- Complete exercise -> `tutorials/`
- Task-focused procedure -> `how-to/`
- Exact syntax and option table -> `reference/`
- Real project diagnosis and correction -> `case-studies/`

## 4. Foundation Scope

Foundation teaches the concepts that are required to understand all later Docker topics. It explains Docker, its architecture, Image, Container, and their relationship in depth.

Volume, Network, Registry, and Docker Hub appear only in the final ecosystem overview. Their syntax and internal behavior belong to later parts.

```text
learning-path/01-foundations/
|-- README.md
|-- 01-docker-la-gi.md
|-- 02-docker-hoat-dong-nhu-the-nao.md
|-- 03-docker-image.md
|-- 04-docker-container.md
|-- 05-image-va-container.md
`-- 06-buc-tranh-tong-the.md
```

### 4.1 `README.md`

- Explain what Foundation covers and deliberately does not cover.
- List prerequisites and expected learning time.
- Present the six chapters in reading order.
- Provide a completion checklist based on understanding, not memorization.

### 4.2 `01-docker-la-gi.md`

- Start with environment inconsistency and dependency problems.
- Define Docker as a platform for building, distributing, and running containerized applications.
- Explain common use cases and boundaries.
- Compare containers and virtual machines only at an introductory level.

### 4.3 `02-docker-hoat-dong-nhu-the-nao.md`

- Explain Docker Client, Docker Daemon/Engine, Docker objects, and Registry.
- Show the request flow for `docker run`.
- Place Docker Desktop in the architecture without treating it as Docker itself.

### 4.4 `03-docker-image.md`

- Define Image from an intuitive and precise perspective.
- Explain its purpose, filesystem layers, image configuration, metadata, immutability, tag, and digest.
- Show how Images are built, pulled, pushed, stored, and used to create Containers.
- Use this chapter as the canonical Explanation template.

### 4.5 `04-docker-container.md`

- Define a Container as an isolated runtime process, not a small virtual machine.
- Explain its relationship to Image, runtime state, writable layer, and lifecycle.
- Explain what is temporary and what requires persistent storage.

### 4.6 `05-image-va-container.md`

- Compare Image and Container directly.
- Explain one-Image-to-many-Containers behavior.
- Resolve lifecycle questions such as stopping, removing, and recreating Containers.
- Focus on correcting the learner's combined mental model instead of repeating both definitions.

### 4.7 `06-buc-tranh-tong-the.md`

- Connect Dockerfile, Image, Registry, Container, Volume, and Network in one flow.
- Introduce non-foundation concepts only enough for recognition.
- Link each concept to its later learning-path part.

## 5. Canonical Explanation Chapter

`03-docker-image.md` defines the editorial standard for other Explanation chapters.

Each chapter should contain the following sections when relevant:

1. Learning outcomes.
2. The problem that motivates the concept.
3. An intuitive first explanation.
4. A precise technical definition.
5. Local terminology notes.
6. Practical purpose and use cases.
7. Internal structure or mechanism.
8. Relationship with other Docker objects.
9. A diagram or process flow.
10. Short observational commands.
11. Common misconceptions.
12. Mental-model questions.
13. A concise summary.
14. Links to the next learning material.
15. Authoritative sources.

Sections are omitted when they do not help the topic. The template is an editorial checklist, not a requirement to manufacture unnecessary content.

## 6. Explanation Depth

The final prose must be substantially deeper than an outline. Important concepts follow four layers:

```text
Intuitive understanding
-> precise definition
-> operating mechanism
-> observable example
```

For example, "Image is immutable" must explain:

- What immutable means in Vietnamese.
- That existing image layers are not edited in place.
- That rebuilding creates a new Image and may reuse unchanged layers.
- That a mutable tag can point to a different Image without mutating the old Image.
- How commands or inspection output can demonstrate the distinction.

Expected reading depth varies by topic. A central topic such as Image may require roughly 2,500-4,000 words, several diagrams, and multiple examples. A comparison chapter can be shorter. Word count is not a target; complete understanding within the defined scope is the target.

## 7. Terminology Strategy

Keep the original English term because learners will encounter it in Docker documentation, commands, logs, and error messages. Explain it in Vietnamese at first use.

Use three layers of terminology support:

1. **First occurrence:** give a short Vietnamese explanation in the sentence.
2. **Local terminology box:** explain three to six difficult terms used by the current section.
3. **Central glossary:** provide a reusable definition in `reference/glossary.md`.

Definitions follow this order:

```text
Quick meaning
-> precise Docker meaning
-> example or analogy
-> limitation of the analogy
```

Editorial rules:

- Do not use an undefined advanced term to explain another term.
- Do not repeatedly redefine a term within the same chapter.
- Use bold text for conceptual terms.
- Use inline code for commands, options, file names, and literal values.
- Link to the glossary when a deeper definition is useful.

Initial glossary topics include Image, Container, Instance, Filesystem, Filesystem Layer, Metadata, Writable Layer, Daemon, Registry, Repository, Tag, Digest, and Build Context.

## 8. Misconception Design

Misconceptions belong in Explanation because they correct the learner's mental model.

Use the combined presentation pattern:

- Add a short warning next to a concept when misunderstanding it would immediately distort the explanation.
- Analyze two to four important misconceptions in a dedicated section near the end of the chapter.

Each detailed misconception contains:

1. The plausible statement.
2. Classification: incorrect, partially correct, context-dependent, outdated, or useful analogy with limitations.
3. Why the statement sounds correct.
4. The exact technical flaw.
5. A more accurate formulation.
6. An observation, command, or diagram that can verify the correction.
7. A source when directly quoting community content.

Do not invent obviously foolish claims. Select statements that a beginner or working developer could reasonably believe.

Community statements should normally be rewritten as anonymous misconception patterns. Direct quotations require a link, sufficient context, and comparison with an authoritative source. The documentation must critique the idea, not the person.

## 9. Diagrams

Use diagrams only when they clarify a relationship, lifecycle, hierarchy, or process that is harder to understand in prose.

Preferred diagram types:

- Concept flow: Dockerfile -> Image -> Container.
- Sequence: user -> Docker Client -> Daemon -> Registry.
- Layer stack: Image filesystem layers plus Container writable layer.
- Lifecycle: created -> running -> stopped -> removed.
- One-to-many relationship: one Image creating multiple Containers.

Every diagram must:

- Have an explicit starting point and direction.
- Use a small, consistent vocabulary.
- Be explained in the surrounding prose.
- Avoid decorative colors or nodes that do not add meaning.
- Remain readable in GitHub Markdown.

## 10. Observational Commands

Explanation chapters may use two to four short commands to make an abstract statement observable.

For each command, explain:

- What request it sends to Docker.
- Which part of the output matters.
- What theoretical claim the output demonstrates.

Do not include a full step-by-step project in an Explanation chapter. Link to a Tutorial for that experience and to Reference for complete syntax.

## 11. Accuracy and Sources

Use Docker Official Documentation as the primary source. Use the OCI Image Specification when a precise image-format definition is necessary.

Secondary articles and forum discussions may help identify confusing explanations, but they do not override primary sources.

Before publishing a chapter:

- Check definitions against current official documentation.
- Check commands against the installed/current Docker CLI syntax where practical.
- Ensure analogies include their limits.
- Ensure diagrams agree with the prose.
- Ensure examples do not imply that Gearhouse is the only supported project shape.

## 12. Syntax Deep-Dive Standard

Code must never be treated as self-explanatory. Every important Dockerfile instruction, Compose key, CLI option, path, and compact syntax follows the same analysis pattern:

1. Show the official or general syntax.
2. Draw or describe the syntax tree.
3. Explain every token, key, index, and separator.
4. Identify the scope that owns the value.
5. Resolve relative paths and interpolated values to concrete examples.
6. Show the filesystem, resource, or lifecycle state before and after execution.
7. Explain defaults and omitted values.
8. Compare the syntax with similar-looking alternatives.
9. Identify plausible mistakes and misconceptions.
10. Provide a command or output that verifies the explanation.

For example, this Dockerfile instruction:

```dockerfile
WORKDIR /app
COPY app.jar app.jar
```

must explain that the first `app.jar` is a source path in the build context while the second is a destination path in the Image filesystem. Because `WORKDIR` is `/app`, the destination resolves to `/app/app.jar`. The two strings have the same name but refer to different filesystems and locations.

For multi-stage builds:

```dockerfile
COPY --from=build /workspace/build/libs/app.jar app.jar
```

the source belongs to the filesystem of the `build` stage, while the destination belongs to the current runtime stage and resolves relative to its `WORKDIR`.

The same standard applies to compact Compose syntax such as:

```yaml
ports:
  - "8080:80"

volumes:
  - database-data:/var/lib/mysql
```

The documentation must expand these values into their source, destination, scope, generated resource, and runtime data flow.

## 13. Terminal Command Standard

Terminal documentation distinguishes global options, command options, arguments, service names, Container names, paths, and commands executed inside a Container.

The standard Docker CLI pattern is:

```text
docker [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [ARGUMENTS]
```

The standard Compose CLI pattern is:

```text
docker compose [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [SERVICE...]
```

Each command explanation contains:

1. The problem the command solves.
2. Official syntax notation.
3. A token-by-token breakdown.
4. Which parser owns each option.
5. The state transition or resource mutation.
6. Similar commands and their behavioral differences.
7. Destructive options and recovery implications.
8. A verification command.
9. PowerShell and Bash variants when multiline or environment syntax differs.

One-line commands are the cross-platform default. Multiline examples use `\` for Bash and a backtick for PowerShell. The PowerShell example must warn that trailing spaces after a backtick break line continuation.

## 14. Part 02: Docker CLI and Lifecycle

```text
learning-path/02-cli-and-lifecycle/
|-- README.md
|-- 01-cach-doc-lenh-docker.md
|-- 02-lenh-quan-ly-image.md
|-- 03-tao-va-chay-container.md
|-- 04-container-lifecycle.md
|-- 05-quan-sat-va-debug-container.md
`-- 06-don-dep-tai-nguyen.md

reference/commands/
|-- README.md
|-- docker-pull.md
|-- docker-run.md
|-- docker-ps.md
|-- docker-logs.md
`-- docker-exec.md
```

### 14.1 Command grammar

The first chapter teaches the object-action mental model:

```text
docker + object + action + options + arguments
```

Examples use the explicit modern form as the canonical teaching syntax:

```bash
docker image ls
docker container ls
docker container stop web
```

Common aliases remain visible in notes:

```bash
docker images
docker ps
docker stop web
```

The learner must understand options, arguments, optional notation (`[]`), repeated arguments (`...`), option placement, and how to use layered help:

```bash
docker --help
docker container --help
docker container run --help
```

### 14.2 Command groups

| Group | Core commands |
|---|---|
| Engine checks | `version`, `info` |
| Image | `pull`, `image ls`, `image inspect`, `image history`, `build`, `tag`, `push`, `image rm` |
| Container creation | `create`, `run` |
| Lifecycle | `start`, `stop`, `restart`, `kill`, `container rm` |
| Observation | `container ls`, `logs`, `inspect`, `stats`, `top`, `port` |
| Interaction | `exec`, `attach`, `cp` |
| Registry | `login`, `logout`, `pull`, `push` |
| Disk usage | `system df` |
| Cleanup | `container prune`, `image prune`, `builder prune`, `system prune` |

The quick command inventory belongs in `reference/commands/README.md` as a CLI cheat sheet. Learning-path chapters explain how commands relate to Docker object state and lifecycle.

### 14.3 Required accuracy notes

- `docker container run` creates a new Container and starts it; it does not merely restart an existing Container.
- `docker image prune` removes dangling Images by default. `-a` expands the scope to unused Images.
- `docker system prune -a` does not remove running Containers or named Volumes by default.
- `docker container rm` normally removes a stopped Container; `--force` is a separate destructive behavior.
- `docker container exec` requires a running Container.
- `bash` is not present in every Image; `sh` may be the available shell.
- Passwords passed directly with `--env` may appear in shell history and Container metadata.

## 15. Part 04: Dockerfile

```text
learning-path/04-dockerfile/
|-- README.md
|-- 01-dockerfile-la-gi.md
|-- 02-build-context-va-dockerignore.md
|-- 03-cac-instruction-co-ban.md
|-- 04-layer-va-build-cache.md
|-- 05-multi-stage-build.md
|-- 06-dockerfile-cho-java.md
|-- 07-bao-mat-va-toi-uu-image.md
`-- 08-buc-tranh-build-hoan-chinh.md

tutorials/
|-- dockerize-spring-boot-gradle.md
`-- dockerize-spring-boot-maven.md

reference/dockerfile/
|-- README.md
|-- instructions.md
`-- docker-build-options.md
```

### 15.1 Dockerfile mental model

The documentation explains this build flow:

```text
Dockerfile + build context
-> BuildKit
-> filesystem layers and image configuration
-> Docker Image
-> Container
```

It explicitly distinguishes:

| Term | Meaning |
|---|---|
| Instruction | A directive such as `FROM`, `RUN`, or `COPY` |
| Stage | A build phase that begins with `FROM` |
| Layer | A content or configuration component of an Image |
| Build step | BuildKit processing an instruction |

A Dockerfile does not have a mandatory list of stages. A file with one `FROM` is a valid single-stage build. Each additional `FROM` begins another stage.

### 15.2 Build context

The chapter must explain the final dot in:

```bash
docker build -t my-app:1.0 .
```

The dot identifies the build context, not the Dockerfile itself. `COPY` cannot freely read files outside that context. `.dockerignore` reduces context size and prevents files such as `.git`, local build output, logs, and environment files from being sent to the builder.

### 15.3 Instruction groups

| Purpose | Instructions |
|---|---|
| Base | `FROM` |
| Build filesystem | `RUN`, `COPY`, `ADD` |
| Build variables | `ARG` |
| Runtime configuration | `ENV`, `WORKDIR`, `USER`, `EXPOSE`, `HEALTHCHECK` |
| Process startup | `ENTRYPOINT`, `CMD` |
| Multi-stage transfer | `COPY --from` |

Each instruction follows the Syntax Deep-Dive Standard. Important comparisons include `RUN` versus `CMD`, `CMD` versus `ENTRYPOINT`, `COPY` versus `ADD`, `ARG` versus `ENV`, and `EXPOSE` versus published ports.

### 15.4 Layer cache

The documentation explains why instruction order affects cache reuse. Dependency descriptors are copied before frequently changing source code when doing so allows the dependency-resolution step to be reused.

Misconceptions include:

- Every instruction creates a filesystem layer.
- Fewer layers always produce a better Image.
- `--no-cache` makes the resulting application run faster.
- Changing instruction order has no effect on build performance.

### 15.5 Multi-stage build

The core model is:

```text
Build stage: build tools + JDK + source -> JAR
Runtime stage: Java runtime + JAR -> application process
```

The chapter explains `AS build`, named stages, `COPY --from`, filesystem isolation between stages, what remains in build cache, and why build tools do not automatically appear in the final runtime Image.

## 16. Java, JAR, JDK, and Runtime Model

The Java chapter teaches the complete artifact flow before presenting a production Dockerfile:

```text
Source `.java`
-> Java compiler
-> bytecode `.class`
-> Gradle or Maven packaging
-> executable JAR
-> JVM/runtime
-> Spring Boot process
```

### 16.1 JAR

The chapter defines JAR as Java Archive, a ZIP-based Java packaging format that may contain bytecode, resources, metadata, and dependencies depending on packaging style.

It distinguishes:

- Library JAR.
- Thin JAR.
- Fat or Uber JAR.
- Spring Boot executable JAR.

The Spring Boot layout explains `META-INF/MANIFEST.MF`, `BOOT-INF/classes`, `BOOT-INF/lib`, Spring Boot Loader, `Main-Class`, and `Start-Class`.

The learner uses commands such as:

```bash
jar --list --file build/libs/application.jar
unzip -p build/libs/application.jar META-INF/MANIFEST.MF
```

The chapter corrects these misconceptions:

- A JAR contains Java itself.
- Every JAR is executable with `java -jar`.
- A JAR contains only compiled code.
- Renaming a JAR changes its internal application behavior.
- Dependencies inside one copied JAR automatically become separate Docker layers.

### 16.2 Gradle and Maven packaging

Gradle content distinguishes `jar`, `bootJar`, `assemble`, and `build`. Maven content explains the lifecycle through `package` and Spring Boot `repackage`.

The documentation explains the consequences of skipping tests with `-x test` or `-DskipTests` rather than presenting those flags as neutral boilerplate.

### 16.3 Why build uses a JDK

The build stage may need the compiler, annotation processors, resource processing, testing tools, and packaging utilities. Gearhouse uses Lombok and MapStruct annotation processors, which makes the compile-time toolchain especially visible.

### 16.4 Why runtime often uses a runtime-only distribution

A typical Spring Boot executable JAR needs the JVM, the `java` launcher, runtime libraries, and the application artifact. It normally does not need Gradle, Maven, source code, dependency caches, or `javac`.

The documentation avoids an absolute rule. A JDK runtime can be justified when operations require tools such as `jcmd`, `jstack`, or other development-kit components. Runtime-only Images reduce size and attack surface, but do not automatically make execution faster or guarantee security.

### 16.5 Version compatibility

The build toolchain, bytecode target, and runtime version must be compatible. Building Java 21 bytecode and running it on Java 17 may produce `UnsupportedClassVersionError`. The reverse direction is often more compatible but still requires dependency and behavior checks.

### 16.6 Canonical Java Dockerfile patterns

The Gradle and Maven tutorials use multi-stage builds, stable artifact names, explicit Java versions, cache-aware copy ordering, and an explanation for every line. Wildcards such as `*.jar` are discussed as a trade-off because multiple matching artifacts can cause ambiguity.

## 17. Part 05: Docker Compose

```text
learning-path/05-docker-compose/
|-- README.md
|-- 01-docker-compose-la-gi.md
|-- 02-cau-truc-compose-file.md
|-- 03-service-image-va-build.md
|-- 04-environment-va-configuration.md
|-- 05-port-va-network.md
|-- 06-volume-va-persistent-data.md
|-- 07-dependency-va-healthcheck.md
|-- 08-compose-cli-va-lifecycle.md
|-- 09-profile-override-va-multi-environment.md
`-- 10-best-practices-va-misconceptions.md

tutorials/
|-- compose-spring-boot-mysql.md
`-- compose-spring-boot-mysql-redis.md

reference/compose/
|-- compose-file-keys.md
`-- compose-cli.md
```

### 17.1 Compose mental model

Dockerfile defines how to build an Image. Compose defines an application model containing services, runtime configuration, Networks, Volumes, dependencies, and related resources.

The core flow is:

```text
compose.yaml
-> Docker Compose CLI
-> resolved Compose model
-> Docker Engine API
-> Networks, Volumes, and Containers
```

The documentation explains YAML mappings, sequences, scalars, indentation, quoting, and path notation such as:

```text
services.backend.image
services.backend.ports[0]
services.backend.environment.SPRING_PROFILES_ACTIVE
volumes.database-data
```

The obsolete top-level `version:` key is not added to new examples.

### 17.2 Service, Image, and build

A service is desired configuration, not exactly one Container. Compose may create multiple Containers from a scalable service. Service names also provide network DNS names within shared Compose Networks.

The documentation distinguishes:

- `image:`: select or name an Image.
- `build:`: define how Compose builds an Image.
- `build.context`: select the build context.
- `build.dockerfile`: select a Dockerfile within that context.

### 17.3 Environment and interpolation

The chapter distinguishes Compose interpolation from Container environment creation:

```text
Shell or `.env`
-> Compose interpolation
-> resolved Compose model
-> `environment` or `env_file`
-> Container process environment
```

It explains precedence, relative file paths, quoting, secrets exposure, and why `.env` and `env_file` are not the same mechanism.

### 17.4 Ports and Networks

Short port syntax is expanded as:

```text
HOST_PORT:CONTAINER_PORT
```

The documentation explains that service-to-service communication uses service DNS and Container ports, not host-published ports. Inside the backend Container, `localhost` refers to the backend Container itself; MySQL is reached through a service name such as `database:3306`.

### 17.5 Volume syntax and lifecycle

The Compose file can contain two `volumes` keys with different scopes:

```text
services.database.volumes
-> mounts storage into a service Container

top-level volumes.database-data
-> declares the named Volume object in the Compose model
```

Short syntax is expanded as:

```text
SOURCE:TARGET[:MODE]
```

For:

```yaml
volumes:
  - database-data:/var/lib/mysql
```

`database-data` is the named Volume source and `/var/lib/mysql` is the target path inside the Container. Compose usually prefixes the runtime Volume name with the project name.

The chapter distinguishes named Volumes, Bind Mounts, anonymous Volumes, long syntax, read-only mounts, `external: true`, and `name:` overrides. It explains mount visibility, Container writable layers, Volume persistence, and the destructive behavior of `docker compose down --volumes`.

The MySQL example explains that initialization environment variables commonly affect an empty data directory. Changing `MYSQL_ROOT_PASSWORD` later does not necessarily alter credentials already stored in an existing Volume.

### 17.6 Healthcheck syntax

The healthcheck chapter explains this tree:

```text
services.database.healthcheck
|-- test
|-- interval
|-- timeout
|-- retries
`-- start_period
```

It distinguishes `CMD` from `CMD-SHELL`, explains shell features and executable availability, and states that Docker interprets exit code `0` as success and nonzero results as failure. A running Container can still have health state `starting` or `unhealthy`.

The timing fields are explained independently:

- `interval`: frequency between checks.
- `timeout`: maximum duration of one check.
- `retries`: consecutive failures required for unhealthy state.
- `start_period`: startup grace period.

Healthchecks observe status; they do not automatically repair services or replace application retry logic.

### 17.7 Dependency syntax

Short `depends_on` expresses startup ordering. Long syntax supports:

- `service_started`.
- `service_healthy`.
- `service_completed_successfully`.
- Optional `required` and Compose-managed `restart` behavior where supported.

The chapter makes clear that startup dependency management does not provide runtime resilience. A backend must still handle database disconnects, timeouts, retries, and reconnect behavior after startup.

## 18. Compose CLI and Lifecycle

The canonical Compose command pattern is:

```text
docker compose [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [SERVICE...]
```

This example is analyzed token by token:

```bash
docker compose -f server/docker-compose.yaml -p gearhouse up -d --build backend
```

- `-f` and `-p` are global Compose options.
- `up` is the command.
- `-d` and `--build` belong to `up`.
- `backend` is a service argument.

### 18.1 Command groups

| Purpose | Commands |
|---|---|
| Resolve and validate | `config`, `config --services`, `config --images` |
| Build and distribute | `build`, `pull`, `push`, `images` |
| Create and start | `create`, `up`, `start` |
| Observe | `ps`, `logs`, `top`, `events` |
| Interact | `exec`, `run`, `cp` |
| Stop and remove | `stop`, `restart`, `kill`, `rm`, `down` |

### 18.2 Required comparisons

- `up` creates or updates resources; `start` starts existing Containers.
- `stop` preserves Containers; `down` removes project Containers and Networks.
- `exec` runs inside an existing Container; `run` creates a one-off Container.
- `build` only builds Images; `up --build` builds and then applies the application model.
- `restart` does not reliably apply changed Compose configuration; `up` is used to reconcile changes.

Destructive options such as `down --volumes`, forced removal, and Image removal require explicit warnings.

## 19. Gearhouse Case Studies

Gearhouse examples are evaluated as trade-offs rather than automatically labeled correct or incorrect.

### 19.1 Dockerfile case study

Positive elements include multi-stage build, JDK builder plus JRE runtime, executable `ENTRYPOINT`, and excluding source/build tools from the runtime Image.

Review topics include:

- Base Image Gradle versus Gradle Wrapper.
- Dependency cache opportunities.
- `clean` and cache behavior.
- Skipping tests with `-x test`.
- Wildcard JAR copy.
- Runtime user privileges.
- The difference between `EXPOSE` and port publishing.

### 19.2 Compose case study

Positive elements include named Volumes, service naming, a shared Network, `env_file`, and modern versionless Compose syntax.

Review topics include:

- Hard-coded MySQL root password.
- Publishing MySQL and Redis ports to the host.
- `container_name` and scaling/name-collision trade-offs.
- Missing healthchecks and readiness modeling.
- Backend `image` versus `build` as deployment and development choices.
- Service DNS names instead of `localhost` for Container-to-Container traffic.

Each case study uses:

```text
Current state
-> intended goal
-> what is correct
-> what is risky or context-dependent
-> improved version
-> verification evidence
```

## 20. Quality Checklist

A chapter is ready when:

- A beginner can state the concept in their own words.
- English technical terms are introduced without blocking comprehension.
- The intuitive explanation and precise definition do not contradict each other.
- Important decisions answer: what, why, mechanism, failure mode, trade-off, and verification.
- Code is explained token by token where compact syntax could confuse a beginner.
- Relative paths are resolved to concrete filesystem locations.
- The chapter stays inside its declared Diataxis scope.
- Misconceptions are plausible and corrected with evidence.
- Commands demonstrate theory instead of becoming an unrelated tutorial.
- PowerShell and Bash differences are handled when relevant.
- Diagrams are explained and technically consistent.
- Destructive commands include data-loss and recovery warnings.
- Links point to the correct Diataxis content type.
- Primary sources are listed.
- Navigation to the previous chapter, part index, and next chapter is present.

## 21. Implementation Strategy

Implementation proceeds in reviewed slices rather than generating the entire book at once.

### 21.1 First slice

Create the navigation foundation and one production-quality Explanation chapter:

```text
docker/
|-- README.md
|-- learning-path/
|   `-- 01-foundations/
|       |-- README.md
|       `-- 03-docker-image.md
`-- reference/
    `-- glossary.md
```

The root and part README files may list planned chapters as roadmap entries, but they must not link to files until those files exist.

`03-docker-image.md` is completed and reviewed first. It validates the Explanation template, terminology system, misconceptions, diagrams, observational commands, and source quality.

### 21.2 Later slices

After the canonical Image chapter is approved:

1. Complete the remaining Foundation chapters.
2. Implement the CLI grammar chapter and command cheat sheet.
3. Implement Dockerfile fundamentals and Java artifact chapters.
4. Implement Docker Compose fundamentals, YAML syntax, and CLI lifecycle.
5. Add Gradle/Maven and Compose tutorials.
6. Add Gearhouse case studies only after the reusable theory is stable.

Parts 03, 06, 07, and 08 remain in the approved book roadmap. Their detailed chapter designs will use the same design-and-review cycle before implementation, rather than being improvised while writing.

## 22. Target Repository and Presentation Format

The documentation is implemented in a dedicated repository, separate from Gearhouse:

```text
D:\work\Project\Tool_Theory_Project\docker-document
```

Gearhouse remains an external case-study source. Reusable Docker theory must not depend on the Gearhouse repository structure.

### 22.1 Rendering strategy

The first implementation is GitHub Markdown-first and MkDocs-ready:

- Standard GitHub-Flavored Markdown is the publishing baseline.
- Mermaid is used for repository-native diagrams.
- GitHub callouts are used for notes and warnings.
- No documentation framework or generated site is required in the first slice.
- A later MkDocs Material layer must be possible without rewriting chapter content.

### 22.2 NGINX-inspired visual rhythm

The presentation borrows these useful patterns from the reviewed NGINX documentation:

- Numbered chapter titles and numbered major sections.
- A primary source block near the top of each chapter.
- Horizontal rules between major reading phases.
- Concrete examples immediately after abstract explanations.
- Tables for exact comparisons and command lookup.
- Tutorial steps expressed as `Bước 1`, `Bước 2`, and so on.
- How-to flow expressed as problem, change, validation, and recovery.

The documentation does not copy these weaker patterns:

- Unsupported absolute claims.
- Excessive bold text.
- Hot-linked third-party images.
- Commands without syntax or output explanation.
- Mixed Diataxis purposes in one document.

### 22.3 Explanation chapter presentation

Explanation chapters use this visual order when relevant:

```text
Numbered H1 title
-> one-sentence mental model
-> compact metadata and primary source
-> previous/index/next navigation
-> problem
-> intuitive explanation
-> precise definition
-> terminology callout
-> mechanism and diagrams
-> observational commands
-> misconceptions
-> mental-model questions
-> summary
-> next reading and sources
-> repeated navigation
```

The compact metadata line includes document type, level, estimated reading time, and prerequisites without turning the page into a dashboard.

### 22.4 Tutorial presentation

Tutorials follow a visible step sequence:

```text
Prerequisites
-> Step action
-> command
-> explanation
-> output to observe
-> verification
-> next step
-> final result
-> common failures and cleanup
```

### 22.5 How-to presentation

How-to documents remain short and task-oriented:

```text
Problem
-> current-state check
-> exact change
-> verification
-> rollback or recovery
-> common failures
```

### 22.6 Reference presentation

Reference pages begin with a compact table, followed by exact syntax sections. They prioritize scanning and accuracy over narrative teaching.

### 22.7 Markdown conventions

- Use exactly one H1 per file.
- Do not skip heading levels.
- Use lowercase ASCII kebab-case filenames with numeric prefixes in learning paths.
- Keep Vietnamese prose fully accented.
- Use bold text for newly introduced concepts.
- Use inline code for commands, options, keys, paths, filenames, and literal values.
- Always label fenced code blocks with a language.
- Prefer one-line commands; show separate Bash and PowerShell multiline forms when continuation syntax differs.
- Use tables only for comparisons, mappings, and quick reference.
- Use diagrams only when they clarify a relationship, sequence, hierarchy, or state change.
- Store required raster assets in the repository instead of hot-linking third-party images.
- Put previous/index/next navigation at both the top and bottom of a chapter.

### 22.8 Callout vocabulary

Use only the common callout types unless a strong reason exists:

```md
> [!NOTE]
> Supporting context.

> [!TIP]
> A practical improvement or verification shortcut.

> [!IMPORTANT]
> A concept that must be understood correctly.

> [!WARNING]
> A failure, security, or data-loss risk.
```

Callouts must remain exceptional. Normal explanations stay in normal prose.

### 22.9 First-slice repository files

The first slice creates:

```text
docker-document/
|-- README.md
|-- STYLE-GUIDE.md
|-- learning-path/
|   `-- 01-foundations/
|       |-- README.md
|       `-- 03-docker-image.md
|-- reference/
|   `-- glossary.md
`-- docs/
    `-- superpowers/
        |-- specs/
        |   `-- 2026-07-27-docker-documentation-design.md
        `-- plans/
```

Empty future directories are not committed. The root README lists future roadmap entries as plain text until their target files exist.
