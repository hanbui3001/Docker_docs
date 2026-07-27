# Docker Glossary

> Từ điển thuật ngữ dùng chung cho bộ tài liệu. Mỗi định nghĩa bắt đầu bằng
> cách hiểu nhanh, sau đó mới đi tới ý nghĩa kỹ thuật chính xác.

[← Về mục lục chính](../README.md)

---

## Argument

**Cách hiểu nhanh:** Argument là giá trị đầu vào chỉ rõ command sẽ tác động lên đối tượng nào hoặc dùng dữ liệu nào.

**Định nghĩa chính xác:** Trong Docker CLI, Argument là token vị trí được parser đọc sau command và option, chẳng hạn Image reference, Container name hoặc path. Argument khác option ở chỗ nó thường không bắt đầu bằng `-` hoặc `--`, và ý nghĩa của nó phụ thuộc command đang được gọi. Với `docker run`, các token sau Image reference còn có thể trở thành command cùng argument chạy bên trong Container.

**Ví dụ:** Trong `docker container logs web`, `web` là Argument chọn Container; trong `docker run nginx:1.27 nginx -g "daemon off;"`, `nginx:1.27` là Image argument, còn các token phía sau là command và argument ghi đè command mặc định của Image.

**Liên quan:** [Container](#container), [Image](#image), [Service](#service)

**Học sâu:** [1. Cách đọc lệnh Docker](../learning-path/02-cli-and-lifecycle/01-cach-doc-lenh-docker.md)

## Bind mount

**Cách hiểu nhanh:** Bind mount đưa trực tiếp một file hoặc thư mục có sẵn trên host vào Filesystem của Container.

**Định nghĩa chính xác:** Bind mount ánh xạ một host path cụ thể tới một destination path trong Container thông qua mount do runtime tạo. Dữ liệu và quyền sở hữu vẫn gắn với Filesystem của host; Docker không quản lý vòng đời nội dung đó như một Volume. Bind mount phụ thuộc cấu trúc path, quyền truy cập và hành vi chia sẻ file của máy chạy Container.

**Ví dụ:** `docker run --mount type=bind,src="$PWD/config",dst=/app/config,readonly my-app:1.0` cho Container đọc thư mục `config` hiện tại tại `/app/config` mà không copy nội dung vào Image.

**Liên quan:** [Filesystem](#filesystem), [tmpfs](#tmpfs), [Volume](#volume)

**Học sâu:** [3. Bind mount](../learning-path/03-storage-and-networking/03-bind-mount.md)

## Build cache

**Cách hiểu nhanh:** Build cache là các kết quả build trước mà builder có thể tái sử dụng để bỏ qua công việc không cần lặp lại.

**Định nghĩa chính xác:** Build cache lưu kết quả trung gian của build step theo instruction, trạng thái cha và các input liên quan. Khi cache key vẫn hợp lệ, BuildKit có thể dùng lại kết quả thay vì thực thi step. Cache là dữ liệu của builder và không đồng nghĩa với Filesystem layer đang có trong final Image; thứ tự instruction quyết định phạm vi step phía sau bị invalidated khi input thay đổi.

**Ví dụ:** Copy `build.gradle` trước `src/` cho phép step resolve dependency được cache lại khi chỉ source code thay đổi, miễn các input khác của step vẫn giữ nguyên.

**Liên quan:** [Build context](#build-context), [BuildKit](#buildkit), [Filesystem layer](#filesystem-layer)

**Học sâu:** [4. Layer và build cache](../learning-path/04-dockerfile/04-layer-va-build-cache.md)

## Build context

**Cách hiểu nhanh:** Build context là tập hợp dữ liệu mà Docker builder được phép dùng khi tạo Image.

**Định nghĩa chính xác:** Build context là tập hợp file và thư mục được gửi hoặc cung cấp cho builder khi chạy một lệnh build. Context có thể đến từ thư mục cục bộ, Git repository, URL hoặc đầu vào chuẩn, tùy cách gọi builder; file `.dockerignore` có thể loại bớt nội dung trước khi builder xử lý.

Các source path cục bộ của `COPY` và `ADD` được resolve trong Build context, không phải tùy ý trên toàn bộ Filesystem của máy gọi lệnh. Riêng `ADD` cũng chấp nhận một số nguồn từ xa như URL hoặc Git repository; các nguồn đó không được resolve như path cục bộ. Dockerfile có thể nằm trong context hoặc được chỉ định riêng, nhưng quyền truy cập source cục bộ vẫn bị giới hạn bởi context đã chọn.

**Ví dụ:** Trong `docker build -t demo .`, dấu `.` chọn thư mục hiện tại làm Build context; `COPY app.jar /app/app.jar` chỉ thành công khi `app.jar` có trong context và không bị `.dockerignore` loại bỏ.

**Liên quan:** [Dockerfile](#dockerfile), [Filesystem](#filesystem), [Image](#image)

**Quay lại nơi đang học:** [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-build-context) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-build-context) · [6. Bức tranh tổng thể](../learning-path/01-foundations/06-buc-tranh-tong-the.md#back-06-buc-tranh-tong-the-build-context)

## Build stage

**Cách hiểu nhanh:** Build stage là một giai đoạn độc lập trong Dockerfile, bắt đầu bằng một instruction `FROM`.

**Định nghĩa chính xác:** Mỗi `FROM` tạo một stage có base Image, Filesystem và Image configuration đang được xây dựng riêng. Stage có thể được đặt tên bằng `AS`, dùng làm source cho `COPY --from`, hoặc được chọn làm output bằng `--target`. Nội dung của stage trước không tự xuất hiện trong stage sau; chỉ dữ liệu được chuyển rõ ràng mới đi qua ranh giới stage.

**Ví dụ:** `FROM eclipse-temurin:21-jdk AS build` mở stage `build`; `COPY --from=build /workspace/app.jar /app/app.jar` lấy artifact từ Filesystem của stage đó vào stage hiện tại.

**Liên quan:** [Build cache](#build-cache), [Dockerfile](#dockerfile), [Filesystem](#filesystem)

**Học sâu:** [5. Multi-stage build](../learning-path/04-dockerfile/05-multi-stage-build.md)

## BuildKit

**Cách hiểu nhanh:** BuildKit là builder hiện đại thực thi build graph và tạo output Image hoặc artifact.

**Định nghĩa chính xác:** BuildKit là backend build của Docker hỗ trợ thực thi graph có phụ thuộc, cache nâng cao, parallelism, secret/SSH mount, nhiều loại build context và output. Dockerfile frontend chuyển Dockerfile thành định nghĩa build mà BuildKit giải quyết. BuildKit quản lý cache và intermediate result riêng; các dữ liệu này không tự trở thành nội dung final Image.

**Ví dụ:** `RUN --mount=type=cache,target=/root/.gradle ./gradlew bootJar` dùng cache mount của BuildKit để tái sử dụng Gradle cache giữa các lần build.

**Liên quan:** [Build cache](#build-cache), [Build context](#build-context), [Dockerfile](#dockerfile)

**Học sâu:** [1. Dockerfile là gì?](../learning-path/04-dockerfile/01-dockerfile-la-gi.md)

## Capability

**Cách hiểu nhanh:** Capability là một phần quyền đặc biệt của Linux được tách nhỏ khỏi quyền root toàn phần.

**Định nghĩa chính xác:** Linux capability chia các đặc quyền kernel thành các đơn vị như thay đổi network configuration hoặc bind port đặc quyền. Container process nhận một tập capability mặc định do runtime cấp và có thể bị thêm hoặc loại bằng runtime option. Chạy non-root và drop capability là hai lớp kiểm soát khác nhau; một process non-root vẫn có thể được cấp capability cụ thể.

**Ví dụ:** `docker run --cap-drop ALL --cap-add NET_BIND_SERVICE my-web:1.0` loại toàn bộ capability mặc định rồi chỉ thêm quyền bind port đặc quyền nếu ứng dụng thật sự cần.

**Liên quan:** [Container](#container), [Resource limit](#resource-limit), [Secret](#secret)

**Học sâu:** [2. Runtime security](../learning-path/07-production/02-runtime-security.md)

## Compose project

**Cách hiểu nhanh:** Compose project là một nhóm tài nguyên Compose được quản lý cùng nhau dưới một project name.

**Định nghĩa chính xác:** Docker Compose resolve Compose model rồi tạo và gắn nhãn các Container, Network, Volume cùng tài nguyên liên quan vào một project. Project name tạo namespace giúp nhiều bản triển khai của cùng Compose file có thể cùng tồn tại. Tên này có thể đến từ option `--project-name`, biến môi trường, trường cấu hình hoặc tên thư mục theo precedence của Compose.

**Ví dụ:** `docker compose --project-name demo up -d` tạo các tài nguyên thuộc project `demo`; `docker compose --project-name demo down` nhắm đúng nhóm đó.

**Liên quan:** [Network](#network), [Service](#service), [Volume](#volume)

**Học sâu:** [1. Docker Compose là gì?](../learning-path/05-docker-compose/01-compose-la-gi.md)

## Container

**Cách hiểu nhanh:** Container là một môi trường chạy cụ thể được tạo từ Image.

**Định nghĩa chính xác:** Container là một Docker object có cấu hình runtime, trạng thái vòng đời và Filesystem riêng được khởi tạo từ một Image. Khi chạy, nó bao quanh một hoặc nhiều process được cô lập bằng các cơ chế của hệ điều hành; khi dừng, Container vẫn có thể tồn tại cho đến khi bị xóa.

Container dùng các Filesystem layer chỉ đọc của Image và thêm Writable layer riêng. Việc tạo hoặc thay đổi Container không sửa nội dung Image nguồn, nên một Image có thể tạo nhiều Container độc lập.

**Ví dụ:** `docker run --name web nginx:1.27` tạo Container `web` từ Image được tham chiếu bởi `nginx:1.27` rồi khởi động process mặc định của Image.

**Liên quan:** [Image](#image), [Instance](#instance), [Writable layer](#writable-layer)

**Quay lại nơi đang học:** [1. Docker là gì?](../learning-path/01-foundations/01-docker-la-gi.md#back-01-docker-la-gi-container) · [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-container) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-container) · [4. Docker Container](../learning-path/01-foundations/04-docker-container.md#back-04-docker-container-container) · [5. Image và Container](../learning-path/01-foundations/05-image-va-container.md#back-05-image-va-container-container) · [6. Bức tranh tổng thể](../learning-path/01-foundations/06-buc-tranh-tong-the.md#back-06-buc-tranh-tong-the-container)

## Daemon

**Cách hiểu nhanh:** Daemon là tiến trình nền nhận yêu cầu Docker và quản lý các Docker object.

**Định nghĩa chính xác:** Docker daemon, thường là tiến trình `dockerd`, cung cấp Docker API và thực hiện các thao tác như build Image, tạo hoặc chạy Container, quản lý network, volume và trao đổi nội dung với Registry. Docker CLI là client gửi yêu cầu đến API này; client và Daemon có thể ở cùng máy hoặc ở hai máy khác nhau.

Trong Docker Desktop, Daemon thường chạy bên trong môi trường Linux do Docker Desktop quản lý thay vì trực tiếp như một process Linux trên hệ điều hành máy người dùng.

**Ví dụ:** Khi chạy `docker container ls`, CLI gửi yêu cầu liệt kê Container đến Daemon đang được Docker context hiện tại chọn.

**Liên quan:** [Container](#container), [Image](#image), [Registry](#registry)

**Quay lại nơi đang học:** [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-daemon) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-daemon)

## Digest

**Cách hiểu nhanh:** Digest là mã băm dùng để tham chiếu đúng một nội dung cụ thể.

**Định nghĩa chính xác:** Digest là định danh content-addressable gồm thuật toán và giá trị băm, thường có dạng `sha256:<hex>`. Trong một Image reference, Digest xác định chính xác manifest hoặc image index có nội dung tạo ra giá trị băm đó; nếu nội dung của object thay đổi thì Digest cũng thay đổi.

Digest gắn với object được băm, vì vậy Digest của image index đa nền tảng và Digest của manifest cho một nền tảng là hai định danh khác nhau. Khác với Tag có thể được chuyển sang nội dung mới, một tham chiếu theo Digest giữ nguyên nội dung được xác định bởi Digest đó.

**Ví dụ:** `docker pull nginx@sha256:<digest>` yêu cầu nội dung `nginx` có manifest hoặc image index khớp chính xác với Digest đã nêu.

**Liên quan:** [Image](#image), [Repository](#repository), [Tag](#tag)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-digest)

## Dockerfile

**Cách hiểu nhanh:** Dockerfile là file văn bản mô tả cách builder tạo một Image.

**Định nghĩa chính xác:** Dockerfile chứa chuỗi instruction như `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `ENTRYPOINT` và `CMD` để builder tạo Image filesystem cùng Image configuration. Builder xử lý các instruction theo thứ tự và có thể tái sử dụng kết quả đã cache khi input liên quan không đổi.

Dockerfile không phải là shell script: mỗi instruction có cú pháp và ngữ nghĩa riêng, dù một số instruction như dạng shell của `RUN` có thể gọi shell bên trong môi trường build.

**Ví dụ:** `FROM eclipse-temurin:21-jre` chọn base Image, còn `COPY app.jar /app/app.jar` chép artifact từ Build context vào Image filesystem.

**Liên quan:** [Build context](#build-context), [Image](#image), [Image configuration](#image-configuration)

**Quay lại nơi đang học:** [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-dockerfile) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-dockerfile) · [6. Bức tranh tổng thể](../learning-path/01-foundations/06-buc-tranh-tong-the.md#back-06-buc-tranh-tong-the-dockerfile)

## Filesystem

**Cách hiểu nhanh:** Filesystem là cây thư mục và file mà một môi trường có thể nhìn thấy tại các đường dẫn như `/app/app.jar`.

**Định nghĩa chính xác:** Filesystem biểu diễn dữ liệu theo cấu trúc đường dẫn, thư mục, file và các thuộc tính liên quan. Trong Docker, cần phân biệt Filesystem của máy cung cấp Build context, Filesystem tạm thời trong quá trình build, Image filesystem được ghép từ các Filesystem layer và Filesystem của Container khi có thêm Writable layer cùng các mount.

Hai đường dẫn có cùng chuỗi ký tự nhưng thuộc hai Filesystem khác nhau không trỏ đến cùng một dữ liệu. Vì vậy source của `COPY` và destination trong Image phải được hiểu theo đúng scope của từng instruction.

**Ví dụ:** Với `COPY app.jar /app/app.jar`, `app.jar` được đọc từ Filesystem của Build context, còn `/app/app.jar` được tạo trong Filesystem của Image đang build.

**Liên quan:** [Build context](#build-context), [Filesystem layer](#filesystem-layer), [Writable layer](#writable-layer)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-filesystem) · [4. Docker Container](../learning-path/01-foundations/04-docker-container.md#back-04-docker-container-filesystem)

## Filesystem layer

**Cách hiểu nhanh:** Filesystem layer là một lớp thay đổi được xếp cùng các lớp khác để tạo nên Image filesystem.

**Định nghĩa chính xác:** Filesystem layer là một tập thay đổi content-addressable đối với cây Filesystem, có thể thêm, sửa hoặc đánh dấu xóa đường dẫn so với các layer phía dưới. Image manifest tham chiếu các layer theo thứ tự; runtime ghép chúng thành một góc nhìn Filesystem thống nhất cho Container.

Layer của Image được dùng như dữ liệu chỉ đọc và có thể được nhiều Image hoặc Container tái sử dụng. Không phải mọi Dockerfile instruction đều tạo Filesystem layer mới: instruction chỉ thay đổi Image configuration có thể không thêm thay đổi Filesystem.

**Ví dụ:** Một instruction `COPY app.jar /app/app.jar` có thể tạo layer chứa file `/app/app.jar`, trong khi `CMD ["java", "-jar", "/app/app.jar"]` chỉ đặt cấu hình chạy mặc định.

**Liên quan:** [Filesystem](#filesystem), [Image](#image), [Writable layer](#writable-layer)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-filesystem-layer)

## Healthcheck

**Cách hiểu nhanh:** Healthcheck là phép kiểm tra định kỳ để Docker xác định ứng dụng trong Container đang healthy hay unhealthy.

**Định nghĩa chính xác:** Healthcheck định nghĩa command chạy bên trong Container cùng interval, timeout, start period và số lần retry. Exit code của command được Docker dùng để cập nhật health status tách biệt với lifecycle state `running` hoặc `exited`. Healthcheck không tự bảo đảm dependency sẵn sàng và Docker Engine không mặc định restart Container chỉ vì trạng thái chuyển thành `unhealthy`.

**Ví dụ:** Compose có thể dùng `test: ["CMD", "curl", "--fail", "http://localhost:8080/actuator/health"]` với `interval: 30s` và `retries: 3` để kiểm tra endpoint nội bộ của Service.

**Liên quan:** [Restart policy](#restart-policy), [Service](#service), [Container](#container)

**Học sâu:** [7. Healthcheck và dependency](../learning-path/05-docker-compose/07-healthcheck-va-dependency.md)

## Image

**Cách hiểu nhanh:** Image là gói mẫu chỉ đọc dùng để tạo Container.

**Định nghĩa chính xác:** Image là tập nội dung được định danh theo nội dung. Một Image reference có thể resolve tới image index đa nền tảng hoặc manifest của một nền tảng; manifest tiếp tục tham chiếu Image configuration và các Filesystem layer theo Digest. Các thành phần này cung cấp Image filesystem cùng những giá trị runtime mặc định như command, environment, working directory và user để runtime tạo Container.

Nội dung đã được định danh bằng Digest không bị sửa tại chỗ. Build lại có thể tạo Image mới và tái sử dụng các layer không đổi; một Tag có thể được cập nhật để trỏ đến Image mới mà không làm thay đổi Image cũ.

**Ví dụ:** `docker image pull nginx:1.27` tải Image mà Tag `1.27` của Repository `nginx` đang tham chiếu tại thời điểm pull.

**Liên quan:** [Container](#container), [Filesystem layer](#filesystem-layer), [Image configuration](#image-configuration)

**Quay lại nơi đang học:** [1. Docker là gì?](../learning-path/01-foundations/01-docker-la-gi.md#back-01-docker-la-gi-image) · [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-image) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-image) · [4. Docker Container](../learning-path/01-foundations/04-docker-container.md#back-04-docker-container-image) · [5. Image và Container](../learning-path/01-foundations/05-image-va-container.md#back-05-image-va-container-image) · [6. Bức tranh tổng thể](../learning-path/01-foundations/06-buc-tranh-tong-the.md#back-06-buc-tranh-tong-the-image)

## Image configuration

**Cách hiểu nhanh:** Image configuration là phần cấu hình mô tả cách Image nên được dùng khi tạo Container.

**Định nghĩa chính xác:** Image configuration là JSON object content-addressable được Image manifest tham chiếu. Nó ghi lại thông tin như nền tảng, chuỗi Diff ID của root filesystem, lịch sử build và các giá trị runtime mặc định gồm environment, entrypoint, command, working directory, user, label và một số khai báo liên quan.

Image configuration không chứa byte của các file trong Image filesystem; dữ liệu file nằm trong các Filesystem layer. Runtime kết hợp cấu hình này với tùy chọn do người dùng cung cấp khi tạo Container, và tùy chọn runtime có thể ghi đè một số giá trị mặc định.

**Ví dụ:** `CMD ["java", "-jar", "/app/app.jar"]` đặt command mặc định trong Image configuration; `docker run <image> sh` có thể thay command đó khi tạo Container.

**Liên quan:** [Dockerfile](#dockerfile), [Image](#image), [Metadata](#metadata)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-image-configuration)

## Image index

**Cách hiểu nhanh:** Image index là danh sách trỏ tới nhiều Image manifest, thường để một Image reference hỗ trợ nhiều platform.

**Định nghĩa chính xác:** Theo OCI Image Specification, image index là object JSON content-addressable chứa các descriptor tham chiếu manifest khác, kèm thông tin platform như operating system và architecture. Khi pull một multi-platform Image, client/runtime chọn manifest phù hợp với platform mục tiêu từ index. Docker manifest list là khái niệm tương đương thường gặp trong hệ sinh thái Docker.

**Ví dụ:** Tag `my-app:1.0` có thể trỏ tới một Image index chứa manifest riêng cho `linux/amd64` và `linux/arm64`; hai manifest có Digest và layer phù hợp từng kiến trúc.

**Liên quan:** [Digest](#digest), [Manifest](#manifest), [Multi-platform Image](#multi-platform-image)

**Học sâu:** [5. Multi-platform Image](../learning-path/06-registry-and-delivery/05-multi-platform-image.md)

## Instance

**Cách hiểu nhanh:** Instance là một bản thể cụ thể được tạo từ một khuôn hoặc định nghĩa dùng lại được.

**Định nghĩa chính xác:** Trong cách giải thích Docker, Instance mô tả quan hệ giữa Image và Container: mỗi Container là một lần hiện thực hóa cụ thể từ Image, có tên hoặc ID, cấu hình runtime, trạng thái và Writable layer riêng. Nhiều Instance có thể dùng chung nội dung Image mà không chia sẻ các thay đổi trong Writable layer.

`Instance` là thuật ngữ khái niệm, không phải một loại Docker object riêng hay một nhóm lệnh độc lập trong Docker CLI.

**Ví dụ:** Chạy `docker run --name web-1 nginx:1.27` và `docker run --name web-2 nginx:1.27` tạo hai Container instance khác nhau từ cùng một Image reference.

**Liên quan:** [Container](#container), [Image](#image), [Writable layer](#writable-layer)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-instance) · [4. Docker Container](../learning-path/01-foundations/04-docker-container.md#back-04-docker-container-instance)

## JAR

**Cách hiểu nhanh:** JAR là file archive dùng để đóng gói bytecode, resource và metadata của ứng dụng hoặc thư viện Java.

**Định nghĩa chính xác:** JAR, viết tắt của Java Archive, là định dạng dựa trên ZIP. Tùy packaging, nó có thể là library JAR, thin JAR, fat/uber JAR hoặc Spring Boot executable JAR. Một executable JAR cần metadata và layout phù hợp để `java -jar` xác định launcher; JAR không tự chứa JVM đầy đủ.

**Ví dụ:** `build/libs/application.jar` có thể chứa `META-INF/MANIFEST.MF`, class ứng dụng trong `BOOT-INF/classes` và dependency trong `BOOT-INF/lib` khi được tạo bởi Spring Boot `bootJar`.

**Liên quan:** [JDK](#jdk), [JVM](#jvm), [Filesystem layer](#filesystem-layer)

**Học sâu:** [6. Dockerfile cho Java](../learning-path/04-dockerfile/06-dockerfile-cho-java.md)

## JDK

**Cách hiểu nhanh:** JDK là bộ công cụ phát triển Java dùng để compile, kiểm tra và đóng gói ứng dụng.

**Định nghĩa chính xác:** JDK, viết tắt của Java Development Kit, là distribution chứa Java runtime cùng compiler `javac` và các công cụ phát triển/quan sát tùy nhà cung cấp. Trong multi-stage build Java, build stage thường cần JDK để Gradle hoặc Maven compile source, chạy annotation processor, test và tạo JAR; runtime stage có thể dùng distribution chỉ chứa thành phần cần chạy ứng dụng.

**Ví dụ:** `FROM eclipse-temurin:21-jdk AS build` cung cấp Java 21 toolchain cho Gradle build trước khi artifact được copy sang runtime stage.

**Liên quan:** [Build stage](#build-stage), [JAR](#jar), [JVM](#jvm)

**Học sâu:** [6. Dockerfile cho Java](../learning-path/04-dockerfile/06-dockerfile-cho-java.md)

## JVM

**Cách hiểu nhanh:** JVM là máy ảo thực thi Java bytecode và quản lý process Java lúc runtime.

**Định nghĩa chính xác:** JVM, viết tắt của Java Virtual Machine, nạp và thực thi class file, quản lý memory, garbage collection, thread và tương tác runtime với hệ điều hành. JVM phải hỗ trợ class file version mà build tạo ra; chạy bytecode Java mới trên JVM cũ có thể gây `UnsupportedClassVersionError`. JVM là thành phần runtime, không phải JAR hoặc build tool.

**Ví dụ:** `ENTRYPOINT ["java", "-jar", "/app/app.jar"]` dùng Java launcher để khởi tạo JVM và chạy executable JAR trong Container.

**Liên quan:** [JAR](#jar), [JDK](#jdk), [Resource limit](#resource-limit)

**Học sâu:** [6. Dockerfile cho Java](../learning-path/04-dockerfile/06-dockerfile-cho-java.md)

## Logging driver

**Cách hiểu nhanh:** Logging driver là cơ chế Docker dùng để thu nhận và chuyển log từ stdout/stderr của Container.

**Định nghĩa chính xác:** Docker logging driver xác định nơi log stream của Container được lưu hoặc gửi, chẳng hạn local storage, `json-file`, syslog, journald hoặc dịch vụ logging bên ngoài. Driver và option được chọn ở Daemon hoặc khi tạo Container. `docker logs` chỉ hoạt động đầy đủ với driver hỗ trợ đọc log qua Docker API; log application ghi trực tiếp vào file bên trong Container không tự đi qua logging driver.

**Ví dụ:** `docker run --log-driver local --log-opt max-size=10m my-app:1.0` dùng local driver và giới hạn kích thước segment theo option của driver.

**Liên quan:** [Container](#container), [Daemon](#daemon), [Resource limit](#resource-limit)

**Học sâu:** [6. Logging và observability](../learning-path/07-production/06-logging-va-observability.md)

## Manifest

**Cách hiểu nhanh:** Manifest là bản mô tả nối một Image cho một platform với configuration và các layer của nó.

**Định nghĩa chính xác:** Image manifest là object JSON content-addressable tham chiếu một Image configuration descriptor và danh sách Filesystem layer descriptor theo thứ tự. Manifest thường đại diện cho một platform cụ thể; Image index có thể trỏ tới nhiều manifest. Digest của manifest xác định chính xác nội dung cấu trúc đó, không phải chỉ riêng một layer hay Tag.

**Ví dụ:** Manifest cho `linux/amd64` tham chiếu configuration ghi architecture `amd64` và các layer chứa filesystem tương ứng của Image.

**Liên quan:** [Digest](#digest), [Image configuration](#image-configuration), [Image index](#image-index)

**Học sâu:** [5. Multi-platform Image](../learning-path/06-registry-and-delivery/05-multi-platform-image.md)

## Metadata

**Cách hiểu nhanh:** Metadata là dữ liệu mô tả hoặc cấu hình cho Docker object thay vì nội dung file ứng dụng thông thường.

**Định nghĩa chính xác:** Metadata là tên gọi chung cho thông tin mô tả, nhận diện hoặc điều khiển cách một Docker object được xử lý. Với Image, Metadata có thể gồm label, lịch sử, kiến trúc, hệ điều hành và các giá trị trong Image configuration; với Container, nó còn có thể gồm tên, trạng thái, cấu hình network và thời điểm tạo.

Metadata không đồng nghĩa với một file duy nhất hoặc một vị trí lưu trữ duy nhất. Một số Metadata chỉ dùng để mô tả, còn một số giá trị như command mặc định hoặc environment có ảnh hưởng trực tiếp đến Container được tạo.

**Ví dụ:** `LABEL org.opencontainers.image.version="1.0.0"` thêm Metadata dạng label vào Image configuration và có thể được xem bằng `docker image inspect`.

**Liên quan:** [Dockerfile](#dockerfile), [Image](#image), [Image configuration](#image-configuration)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-metadata)

## Multi-platform Image

**Cách hiểu nhanh:** Multi-platform Image là một Image reference có thể resolve tới nội dung phù hợp cho nhiều hệ điều hành hoặc kiến trúc CPU.

**Định nghĩa chính xác:** Một multi-platform Image thường được phân phối bằng Image index hoặc manifest list chứa descriptor tới manifest riêng cho từng platform. Khi pull/run, client và runtime chọn variant phù hợp như `linux/amd64` hoặc `linux/arm64`. Các variant có thể chia sẻ một số layer nhưng không bắt buộc có cùng Digest hoặc byte content.

**Ví dụ:** `docker buildx build --platform linux/amd64,linux/arm64 --tag registry.example/app:1.0 --push .` build và push các platform variant dưới một Image reference chung.

**Liên quan:** [Image](#image), [Image index](#image-index), [Manifest](#manifest)

**Học sâu:** [5. Multi-platform Image](../learning-path/06-registry-and-delivery/05-multi-platform-image.md)

## Network

**Cách hiểu nhanh:** Docker Network là miền kết nối cho phép Container giao tiếp theo topology và chính sách đã chọn.

**Định nghĩa chính xác:** Network là Docker object hoặc network attachment do driver quản lý, cung cấp địa chỉ, route, isolation và trong nhiều trường hợp DNS/service discovery cho Container được kết nối. Bridge Network do người dùng tạo cho phép Container giao tiếp bằng tên trong cùng Network; Network không đồng nghĩa với việc port đã được publish ra host.

**Ví dụ:** `docker network create app-net` rồi chạy hai Container với `--network app-net` cho phép chúng giao tiếp trong Network đó bằng Container name hoặc alias phù hợp.

**Liên quan:** [Port publishing](#port-publishing), [Service](#service), [Container](#container)

**Học sâu:** [5. Docker Network](../learning-path/03-storage-and-networking/05-docker-network.md)

## Port publishing

**Cách hiểu nhanh:** Port publishing tạo đường chuyển tiếp từ địa chỉ/port trên host tới port của Container.

**Định nghĩa chính xác:** Khi tạo Container, publish rule ánh xạ host IP và host port tới Container port/protocol thông qua network stack do Docker quản lý. Nó là runtime configuration và khác `EXPOSE`, vốn chỉ là Image metadata. Nếu bỏ host IP, rule có thể bind trên nhiều interface theo cấu hình nền tảng, làm service có thể truy cập từ ngoài máy.

**Ví dụ:** `docker run --publish 127.0.0.1:8080:80 nginx:1.27` chỉ bind host loopback port `8080` và chuyển traffic tới port `80` của Container.

**Liên quan:** [Network](#network), [Service](#service), [Container](#container)

**Học sâu:** [6. Port publishing](../learning-path/03-storage-and-networking/06-port-publishing.md)

## Provenance

**Cách hiểu nhanh:** Provenance là bằng chứng mô tả artifact được build từ đâu, bằng quy trình nào và với đầu vào nào.

**Định nghĩa chính xác:** Build provenance là metadata hoặc attestation ghi nhận thông tin như source repository, builder, build parameters và material dùng để tạo Image. Provenance hỗ trợ truy vết supply chain và xác minh policy, nhưng không tự chứng minh source an toàn hoặc artifact không có lỗ hổng; mức độ tin cậy phụ thuộc builder, chữ ký và hệ thống xác minh.

**Ví dụ:** Pipeline BuildKit có thể xuất provenance attestation cùng Image để hệ thống release kiểm tra commit và builder trước khi deploy.

**Liên quan:** [Digest](#digest), [SBOM](#sbom), [Image](#image)

**Học sâu:** [6. Delivery flow](../learning-path/06-registry-and-delivery/06-delivery-flow.md)

## Registry

**Cách hiểu nhanh:** Registry là dịch vụ lưu trữ và phân phối Image qua mạng.

**Định nghĩa chính xác:** Registry triển khai API để client push, pull và tra cứu manifest, image index, layer cùng các blob content-addressable khác. Registry tổ chức các tham chiếu dưới các Repository, có thể yêu cầu xác thực và thường áp dụng chính sách quyền truy cập hoặc lưu giữ nội dung.

Registry là dịch vụ, còn Repository là không gian tên cho một nhóm nội dung trong dịch vụ đó. Docker Hub là một Registry công cộng phổ biến, nhưng tổ chức cũng có thể vận hành Registry riêng.

**Ví dụ:** Trong `docker pull registry.example.com/team/api:1.0`, `registry.example.com` xác định Registry mà Docker client liên hệ.

**Liên quan:** [Digest](#digest), [Repository](#repository), [Tag](#tag)

**Quay lại nơi đang học:** [1. Docker là gì?](../learning-path/01-foundations/01-docker-la-gi.md#back-01-docker-la-gi-registry) · [2. Docker hoạt động như thế nào?](../learning-path/01-foundations/02-docker-hoat-dong-nhu-the-nao.md#back-02-docker-hoat-dong-nhu-the-nao-registry) · [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-registry) · [6. Bức tranh tổng thể](../learning-path/01-foundations/06-buc-tranh-tong-the.md#back-06-buc-tranh-tong-the-registry)

## Repository

**Cách hiểu nhanh:** Repository là tên nhóm các phiên bản và tham chiếu liên quan của một Image trong Registry.

**Định nghĩa chính xác:** Repository là một namespace có tên trong Registry, chứa các manifest hoặc image index có thể được tham chiếu bằng Tag hoặc Digest. Một Repository thường đại diện cho cùng một ứng dụng hoặc thành phần qua nhiều phiên bản và nền tảng, nhưng Registry không bắt buộc mọi nội dung trong đó phải có hành vi giống nhau.

Repository trong ngữ cảnh này không phải Git repository. Trong một Image reference đầy đủ, phần Repository nằm sau hostname Registry và trước Tag hoặc Digest.

**Ví dụ:** Trong `registry.example.com/team/api:1.0`, `team/api` là Repository, còn `1.0` là Tag nằm trong Repository đó.

**Liên quan:** [Image](#image), [Registry](#registry), [Tag](#tag)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-repository)

## Resource limit

**Cách hiểu nhanh:** Resource limit giới hạn lượng CPU, memory hoặc tài nguyên khác mà Container được phép sử dụng.

**Định nghĩa chính xác:** Resource limit là runtime configuration được Docker chuyển thành cơ chế kiểm soát của kernel như cgroup trên Linux. Memory limit có thể dẫn tới OOM kill khi process vượt khả năng cấp phát; CPU limit hoặc quota điều tiết thời gian CPU thay vì bảo đảm hiệu năng. Limit khác reservation/request của orchestrator và không thay thế việc cấu hình application runtime như JVM heap.

**Ví dụ:** `docker run --memory 512m --cpus 1.5 my-app:1.0` giới hạn Container ở khoảng 512 MiB memory và tối đa tương đương 1,5 CPU theo cơ chế quota.

**Liên quan:** [Container](#container), [JVM](#jvm), [Restart policy](#restart-policy)

**Học sâu:** [4. Resource limits](../learning-path/07-production/04-resource-limits.md)

## Restart policy

**Cách hiểu nhanh:** Restart policy quy định khi nào Docker nên khởi động lại Container sau khi process dừng hoặc Daemon restart.

**Định nghĩa chính xác:** Restart policy là cấu hình runtime như `no`, `on-failure`, `always` hoặc `unless-stopped`. Docker đánh giá policy dựa trên cách Container dừng, exit status và trạng thái Daemon; policy không sửa nguyên nhân ứng dụng crash và không đồng nghĩa với healthcheck remediation. Hành vi chi tiết cần được hiểu cùng nền tảng và orchestrator đang dùng.

**Ví dụ:** `docker run --restart on-failure:3 my-app:1.0` cho Docker thử khởi động lại tối đa ba lần khi process kết thúc với exit code khác `0`.

**Liên quan:** [Container](#container), [Healthcheck](#healthcheck), [Resource limit](#resource-limit)

**Học sâu:** [5. Health và restart](../learning-path/07-production/05-health-va-restart.md)

## SBOM

**Cách hiểu nhanh:** SBOM là danh sách có cấu trúc về các thành phần phần mềm có trong artifact hoặc Image.

**Định nghĩa chính xác:** SBOM, viết tắt của Software Bill of Materials, ghi nhận package, library, version và quan hệ dependency theo format như SPDX hoặc CycloneDX. SBOM hỗ trợ inventory, vulnerability matching và license review, nhưng không tự phát hiện mọi mã độc hoặc chứng minh component được sử dụng an toàn. Chất lượng phụ thuộc khả năng scanner nhìn thấy package và metadata build.

**Ví dụ:** Pipeline có thể tạo SBOM cho `registry.example/app@sha256:...` rồi lưu hoặc gắn attestation để hệ thống security tra dependency bị ảnh hưởng bởi CVE.

**Liên quan:** [Digest](#digest), [Provenance](#provenance), [Image](#image)

**Học sâu:** [6. Delivery flow](../learning-path/06-registry-and-delivery/06-delivery-flow.md)

## Secret

**Cách hiểu nhanh:** Secret là dữ liệu nhạy cảm như password, token hoặc private key cần được cấp có kiểm soát.

**Định nghĩa chính xác:** Trong workflow Docker, Secret là dữ liệu không nên được ghi cố định vào Dockerfile, Image layer, `ARG`, `ENV`, Compose file hoặc command history. BuildKit secret mount cung cấp secret tạm cho một build step; runtime secret có thể được inject bằng file, secret store hoặc cơ chế của orchestrator. Cách cấp secret phải giới hạn phạm vi, thời gian tồn tại và quyền đọc.

**Ví dụ:** `docker build --secret id=repo_token,src=token.txt .` kết hợp `RUN --mount=type=secret,id=repo_token ...` để dùng token trong build mà không copy file đó vào final Image.

**Liên quan:** [BuildKit](#buildkit), [Container](#container), [Provenance](#provenance)

**Học sâu:** [3. Configuration và secret](../learning-path/07-production/03-configuration-va-secret.md)

## Service

**Cách hiểu nhanh:** Service trong Docker Compose là định nghĩa một loại workload mà Compose dùng để tạo Container với cấu hình chung.

**Định nghĩa chính xác:** Compose Service là entry dưới top-level `services`, mô tả Image hoặc build, command, environment, mount, Network, port, dependency và các thuộc tính runtime khác. Service là model khai báo, không phải chính Container; Compose có thể tạo lại hoặc tạo nhiều Container từ cùng Service tùy lệnh và nền tảng hỗ trợ.

**Ví dụ:** `services.backend` có thể build Image ứng dụng, nối vào `app-network`, mount Volume và publish port; Container tạo ra nhận label liên kết với Service `backend` và Compose project tương ứng.

**Liên quan:** [Compose project](#compose-project), [Network](#network), [Volume](#volume)

**Học sâu:** [3. Service, Image và build](../learning-path/05-docker-compose/03-service-image-va-build.md)

## Tag

**Cách hiểu nhanh:** Tag là nhãn dễ đọc mà con người dùng để tham chiếu một Image trong Repository.

**Định nghĩa chính xác:** Tag là một tham chiếu dạng tên, có thể thay đổi, ánh xạ một tên như `1.0` hoặc `latest` tới một manifest hoặc image index trong Repository. Registry có thể cập nhật cùng Tag để trỏ tới nội dung mới, nên Tag không phải định danh nội dung bất biến.

Hai lần dùng cùng một Tag ở hai thời điểm khác nhau không bảo đảm nhận được cùng nội dung Image. Khi cần cố định chính xác nội dung, dùng tham chiếu theo Digest phù hợp thay vì chỉ dựa vào Tag.

**Ví dụ:** Sau khi push bản build mới dưới tên `myapp:latest`, Tag `latest` có thể trỏ tới Digest khác với lần pull trước đó.

**Liên quan:** [Digest](#digest), [Image](#image), [Repository](#repository)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-tag)

## tmpfs

**Cách hiểu nhanh:** tmpfs là mount lưu dữ liệu tạm trong memory-backed filesystem thay vì Writable layer hoặc ổ đĩa bền vững.

**Định nghĩa chính xác:** Trên Linux, tmpfs mount tạo Filesystem tạm do kernel quản lý, thường dùng memory và có thể dùng swap tùy hệ thống. Dữ liệu không được ghi vào Image hay Writable layer và mất khi Container dừng/bị xóa hoặc mount biến mất. tmpfs phù hợp dữ liệu tạm hoặc nhạy cảm không cần persistence, nhưng vẫn tiêu thụ memory và cần Resource limit phù hợp.

**Ví dụ:** `docker run --tmpfs /run:size=64m,noexec my-app:1.0` mount tmpfs tại `/run` với giới hạn option được runtime hỗ trợ.

**Liên quan:** [Bind mount](#bind-mount), [Resource limit](#resource-limit), [Volume](#volume)

**Học sâu:** [4. tmpfs và lựa chọn storage](../learning-path/03-storage-and-networking/04-tmpfs-va-lua-chon-storage.md)

## Volume

**Cách hiểu nhanh:** Volume là vùng dữ liệu bền vững do Docker quản lý độc lập với vòng đời một Container cụ thể.

**Định nghĩa chính xác:** Volume là Docker storage object được volume driver tạo và mount vào một hoặc nhiều Container. Dữ liệu của named Volume không bị xóa khi Container bị xóa, trừ khi Volume cũng được remove. So với Bind mount, Docker quản lý tên, location và lifecycle object; location vật lý phụ thuộc Daemon, driver và nền tảng nên application không nên dựa vào host path nội bộ.

**Ví dụ:** `docker volume create database-data` rồi `docker run --mount type=volume,src=database-data,dst=/var/lib/postgresql/data postgres` giữ dữ liệu database ngoài Writable layer của Container.

**Liên quan:** [Bind mount](#bind-mount), [Compose project](#compose-project), [Writable layer](#writable-layer)

**Học sâu:** [2. Docker Volume](../learning-path/03-storage-and-networking/02-docker-volume.md)

## Writable layer

**Cách hiểu nhanh:** Writable layer là lớp riêng cho phép một Container ghi thay đổi lên Filesystem trong vòng đời của nó.

**Định nghĩa chính xác:** Khi tạo Container, runtime đặt một Writable layer lên trên các Filesystem layer chỉ đọc của Image. Các thao tác tạo, sửa hoặc xóa file mà không đi qua mount được ghi nhận tại layer này theo cơ chế copy-on-write, trong khi layer của Image bên dưới vẫn không đổi.

Mỗi Container có Writable layer riêng. Dữ liệu trong layer này bị xóa cùng Container, vì vậy dữ liệu cần tồn tại độc lập với vòng đời Container nên được đặt trong volume, bind mount hoặc hệ thống lưu trữ bên ngoài phù hợp.

**Ví dụ:** Nếu process trong Container ghi `/tmp/result.txt` mà `/tmp` không phải mount, file đó nằm trong Writable layer và biến mất khi Container bị xóa.

**Liên quan:** [Container](#container), [Filesystem](#filesystem), [Filesystem layer](#filesystem-layer)

**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-writable-layer) · [4. Docker Container](../learning-path/01-foundations/04-docker-container.md#back-04-docker-container-writable-layer) · [5. Image và Container](../learning-path/01-foundations/05-image-va-container.md#back-05-image-va-container-writable-layer)
