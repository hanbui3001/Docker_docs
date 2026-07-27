# Docker Glossary

> Từ điển thuật ngữ dùng chung cho bộ tài liệu. Mỗi định nghĩa bắt đầu bằng
> cách hiểu nhanh, sau đó mới đi tới ý nghĩa kỹ thuật chính xác.

[← Về mục lục chính](../README.md)

---

## Build context

**Cách hiểu nhanh:** Build context là tập hợp dữ liệu mà Docker builder được phép dùng khi tạo Image.

**Định nghĩa chính xác:** Build context là tập hợp file và thư mục được gửi hoặc cung cấp cho builder khi chạy một lệnh build. Context có thể đến từ thư mục cục bộ, Git repository, URL hoặc đầu vào chuẩn, tùy cách gọi builder; file `.dockerignore` có thể loại bớt nội dung trước khi builder xử lý.

Các source path cục bộ của `COPY` và `ADD` được resolve trong Build context, không phải tùy ý trên toàn bộ Filesystem của máy gọi lệnh. Riêng `ADD` cũng chấp nhận một số nguồn từ xa như URL hoặc Git repository; các nguồn đó không được resolve như path cục bộ. Dockerfile có thể nằm trong context hoặc được chỉ định riêng, nhưng quyền truy cập source cục bộ vẫn bị giới hạn bởi context đã chọn.

**Ví dụ:** Trong `docker build -t demo .`, dấu `.` chọn thư mục hiện tại làm Build context; `COPY app.jar /app/app.jar` chỉ thành công khi `app.jar` có trong context và không bị `.dockerignore` loại bỏ.

**Liên quan:** [Dockerfile](#dockerfile), [Filesystem](#filesystem), [Image](#image)

## Container

**Cách hiểu nhanh:** Container là một môi trường chạy cụ thể được tạo từ Image.

**Định nghĩa chính xác:** Container là một Docker object có cấu hình runtime, trạng thái vòng đời và Filesystem riêng được khởi tạo từ một Image. Khi chạy, nó bao quanh một hoặc nhiều process được cô lập bằng các cơ chế của hệ điều hành; khi dừng, Container vẫn có thể tồn tại cho đến khi bị xóa.

Container dùng các Filesystem layer chỉ đọc của Image và thêm Writable layer riêng. Việc tạo hoặc thay đổi Container không sửa nội dung Image nguồn, nên một Image có thể tạo nhiều Container độc lập.

**Ví dụ:** `docker run --name web nginx:1.27` tạo Container `web` từ Image được tham chiếu bởi `nginx:1.27` rồi khởi động process mặc định của Image.

**Liên quan:** [Image](#image), [Instance](#instance), [Writable layer](#writable-layer)

## Daemon

**Cách hiểu nhanh:** Daemon là tiến trình nền nhận yêu cầu Docker và quản lý các Docker object.

**Định nghĩa chính xác:** Docker daemon, thường là tiến trình `dockerd`, cung cấp Docker API và thực hiện các thao tác như build Image, tạo hoặc chạy Container, quản lý network, volume và trao đổi nội dung với Registry. Docker CLI là client gửi yêu cầu đến API này; client và Daemon có thể ở cùng máy hoặc ở hai máy khác nhau.

Trong Docker Desktop, Daemon thường chạy bên trong môi trường Linux do Docker Desktop quản lý thay vì trực tiếp như một process Linux trên hệ điều hành máy người dùng.

**Ví dụ:** Khi chạy `docker container ls`, CLI gửi yêu cầu liệt kê Container đến Daemon đang được Docker context hiện tại chọn.

**Liên quan:** [Container](#container), [Image](#image), [Registry](#registry)

## Digest

**Cách hiểu nhanh:** Digest là mã băm dùng để tham chiếu đúng một nội dung cụ thể.

**Định nghĩa chính xác:** Digest là định danh content-addressable gồm thuật toán và giá trị băm, thường có dạng `sha256:<hex>`. Trong một Image reference, Digest xác định chính xác manifest hoặc image index có nội dung tạo ra giá trị băm đó; nếu nội dung của object thay đổi thì Digest cũng thay đổi.

Digest gắn với object được băm, vì vậy Digest của image index đa nền tảng và Digest của manifest cho một nền tảng là hai định danh khác nhau. Khác với Tag có thể được chuyển sang nội dung mới, một tham chiếu theo Digest giữ nguyên nội dung được xác định bởi Digest đó.

**Ví dụ:** `docker pull nginx@sha256:<digest>` yêu cầu nội dung `nginx` có manifest hoặc image index khớp chính xác với Digest đã nêu.

**Liên quan:** [Image](#image), [Repository](#repository), [Tag](#tag)

## Dockerfile

**Cách hiểu nhanh:** Dockerfile là file văn bản mô tả cách builder tạo một Image.

**Định nghĩa chính xác:** Dockerfile chứa chuỗi instruction như `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `ENTRYPOINT` và `CMD` để builder tạo Image filesystem cùng Image configuration. Builder xử lý các instruction theo thứ tự và có thể tái sử dụng kết quả đã cache khi input liên quan không đổi.

Dockerfile không phải là shell script: mỗi instruction có cú pháp và ngữ nghĩa riêng, dù một số instruction như dạng shell của `RUN` có thể gọi shell bên trong môi trường build.

**Ví dụ:** `FROM eclipse-temurin:21-jre` chọn base Image, còn `COPY app.jar /app/app.jar` chép artifact từ Build context vào Image filesystem.

**Liên quan:** [Build context](#build-context), [Image](#image), [Image configuration](#image-configuration)

## Filesystem

**Cách hiểu nhanh:** Filesystem là cây thư mục và file mà một môi trường có thể nhìn thấy tại các đường dẫn như `/app/app.jar`.

**Định nghĩa chính xác:** Filesystem biểu diễn dữ liệu theo cấu trúc đường dẫn, thư mục, file và các thuộc tính liên quan. Trong Docker, cần phân biệt Filesystem của máy cung cấp Build context, Filesystem tạm thời trong quá trình build, Image filesystem được ghép từ các Filesystem layer và Filesystem của Container khi có thêm Writable layer cùng các mount.

Hai đường dẫn có cùng chuỗi ký tự nhưng thuộc hai Filesystem khác nhau không trỏ đến cùng một dữ liệu. Vì vậy source của `COPY` và destination trong Image phải được hiểu theo đúng scope của từng instruction.

**Ví dụ:** Với `COPY app.jar /app/app.jar`, `app.jar` được đọc từ Filesystem của Build context, còn `/app/app.jar` được tạo trong Filesystem của Image đang build.

**Liên quan:** [Build context](#build-context), [Filesystem layer](#filesystem-layer), [Writable layer](#writable-layer)

## Filesystem layer

**Cách hiểu nhanh:** Filesystem layer là một lớp thay đổi được xếp cùng các lớp khác để tạo nên Image filesystem.

**Định nghĩa chính xác:** Filesystem layer là một tập thay đổi content-addressable đối với cây Filesystem, có thể thêm, sửa hoặc đánh dấu xóa đường dẫn so với các layer phía dưới. Image manifest tham chiếu các layer theo thứ tự; runtime ghép chúng thành một góc nhìn Filesystem thống nhất cho Container.

Layer của Image được dùng như dữ liệu chỉ đọc và có thể được nhiều Image hoặc Container tái sử dụng. Không phải mọi Dockerfile instruction đều tạo Filesystem layer mới: instruction chỉ thay đổi Image configuration có thể không thêm thay đổi Filesystem.

**Ví dụ:** Một instruction `COPY app.jar /app/app.jar` có thể tạo layer chứa file `/app/app.jar`, trong khi `CMD ["java", "-jar", "/app/app.jar"]` chỉ đặt cấu hình chạy mặc định.

**Liên quan:** [Filesystem](#filesystem), [Image](#image), [Writable layer](#writable-layer)

## Image

**Cách hiểu nhanh:** Image là gói mẫu chỉ đọc dùng để tạo Container.

**Định nghĩa chính xác:** Image là tập nội dung được định danh theo nội dung. Một Image reference có thể resolve tới image index đa nền tảng hoặc manifest của một nền tảng; manifest tiếp tục tham chiếu Image configuration và các Filesystem layer theo Digest. Các thành phần này cung cấp Image filesystem cùng những giá trị runtime mặc định như command, environment, working directory và user để runtime tạo Container.

Nội dung đã được định danh bằng Digest không bị sửa tại chỗ. Build lại có thể tạo Image mới và tái sử dụng các layer không đổi; một Tag có thể được cập nhật để trỏ đến Image mới mà không làm thay đổi Image cũ.

**Ví dụ:** `docker image pull nginx:1.27` tải Image mà Tag `1.27` của Repository `nginx` đang tham chiếu tại thời điểm pull.

**Liên quan:** [Container](#container), [Filesystem layer](#filesystem-layer), [Image configuration](#image-configuration)

## Image configuration

**Cách hiểu nhanh:** Image configuration là phần cấu hình mô tả cách Image nên được dùng khi tạo Container.

**Định nghĩa chính xác:** Image configuration là JSON object content-addressable được Image manifest tham chiếu. Nó ghi lại thông tin như nền tảng, chuỗi Diff ID của root filesystem, lịch sử build và các giá trị runtime mặc định gồm environment, entrypoint, command, working directory, user, label và một số khai báo liên quan.

Image configuration không chứa byte của các file trong Image filesystem; dữ liệu file nằm trong các Filesystem layer. Runtime kết hợp cấu hình này với tùy chọn do người dùng cung cấp khi tạo Container, và tùy chọn runtime có thể ghi đè một số giá trị mặc định.

**Ví dụ:** `CMD ["java", "-jar", "/app/app.jar"]` đặt command mặc định trong Image configuration; `docker run <image> sh` có thể thay command đó khi tạo Container.

**Liên quan:** [Dockerfile](#dockerfile), [Image](#image), [Metadata](#metadata)

## Instance

**Cách hiểu nhanh:** Instance là một bản thể cụ thể được tạo từ một khuôn hoặc định nghĩa dùng lại được.

**Định nghĩa chính xác:** Trong cách giải thích Docker, Instance mô tả quan hệ giữa Image và Container: mỗi Container là một lần hiện thực hóa cụ thể từ Image, có tên hoặc ID, cấu hình runtime, trạng thái và Writable layer riêng. Nhiều Instance có thể dùng chung nội dung Image mà không chia sẻ các thay đổi trong Writable layer.

`Instance` là thuật ngữ khái niệm, không phải một loại Docker object riêng hay một nhóm lệnh độc lập trong Docker CLI.

**Ví dụ:** Chạy `docker run --name web-1 nginx:1.27` và `docker run --name web-2 nginx:1.27` tạo hai Container instance khác nhau từ cùng một Image reference.

**Liên quan:** [Container](#container), [Image](#image), [Writable layer](#writable-layer)

## Metadata

**Cách hiểu nhanh:** Metadata là dữ liệu mô tả hoặc cấu hình cho Docker object thay vì nội dung file ứng dụng thông thường.

**Định nghĩa chính xác:** Metadata là tên gọi chung cho thông tin mô tả, nhận diện hoặc điều khiển cách một Docker object được xử lý. Với Image, Metadata có thể gồm label, lịch sử, kiến trúc, hệ điều hành và các giá trị trong Image configuration; với Container, nó còn có thể gồm tên, trạng thái, cấu hình network và thời điểm tạo.

Metadata không đồng nghĩa với một file duy nhất hoặc một vị trí lưu trữ duy nhất. Một số Metadata chỉ dùng để mô tả, còn một số giá trị như command mặc định hoặc environment có ảnh hưởng trực tiếp đến Container được tạo.

**Ví dụ:** `LABEL org.opencontainers.image.version="1.0.0"` thêm Metadata dạng label vào Image configuration và có thể được xem bằng `docker image inspect`.

**Liên quan:** [Dockerfile](#dockerfile), [Image](#image), [Image configuration](#image-configuration)

## Registry

**Cách hiểu nhanh:** Registry là dịch vụ lưu trữ và phân phối Image qua mạng.

**Định nghĩa chính xác:** Registry triển khai API để client push, pull và tra cứu manifest, image index, layer cùng các blob content-addressable khác. Registry tổ chức các tham chiếu dưới các Repository, có thể yêu cầu xác thực và thường áp dụng chính sách quyền truy cập hoặc lưu giữ nội dung.

Registry là dịch vụ, còn Repository là không gian tên cho một nhóm nội dung trong dịch vụ đó. Docker Hub là một Registry công cộng phổ biến, nhưng tổ chức cũng có thể vận hành Registry riêng.

**Ví dụ:** Trong `docker pull registry.example.com/team/api:1.0`, `registry.example.com` xác định Registry mà Docker client liên hệ.

**Liên quan:** [Digest](#digest), [Repository](#repository), [Tag](#tag)

## Repository

**Cách hiểu nhanh:** Repository là tên nhóm các phiên bản và tham chiếu liên quan của một Image trong Registry.

**Định nghĩa chính xác:** Repository là một namespace có tên trong Registry, chứa các manifest hoặc image index có thể được tham chiếu bằng Tag hoặc Digest. Một Repository thường đại diện cho cùng một ứng dụng hoặc thành phần qua nhiều phiên bản và nền tảng, nhưng Registry không bắt buộc mọi nội dung trong đó phải có hành vi giống nhau.

Repository trong ngữ cảnh này không phải Git repository. Trong một Image reference đầy đủ, phần Repository nằm sau hostname Registry và trước Tag hoặc Digest.

**Ví dụ:** Trong `registry.example.com/team/api:1.0`, `team/api` là Repository, còn `1.0` là Tag nằm trong Repository đó.

**Liên quan:** [Image](#image), [Registry](#registry), [Tag](#tag)

## Tag

**Cách hiểu nhanh:** Tag là nhãn dễ đọc mà con người dùng để tham chiếu một Image trong Repository.

**Định nghĩa chính xác:** Tag là một tham chiếu dạng tên, có thể thay đổi, ánh xạ một tên như `1.0` hoặc `latest` tới một manifest hoặc image index trong Repository. Registry có thể cập nhật cùng Tag để trỏ tới nội dung mới, nên Tag không phải định danh nội dung bất biến.

Hai lần dùng cùng một Tag ở hai thời điểm khác nhau không bảo đảm nhận được cùng nội dung Image. Khi cần cố định chính xác nội dung, dùng tham chiếu theo Digest phù hợp thay vì chỉ dựa vào Tag.

**Ví dụ:** Sau khi push bản build mới dưới tên `myapp:latest`, Tag `latest` có thể trỏ tới Digest khác với lần pull trước đó.

**Liên quan:** [Digest](#digest), [Image](#image), [Repository](#repository)

## Writable layer

**Cách hiểu nhanh:** Writable layer là lớp riêng cho phép một Container ghi thay đổi lên Filesystem trong vòng đời của nó.

**Định nghĩa chính xác:** Khi tạo Container, runtime đặt một Writable layer lên trên các Filesystem layer chỉ đọc của Image. Các thao tác tạo, sửa hoặc xóa file mà không đi qua mount được ghi nhận tại layer này theo cơ chế copy-on-write, trong khi layer của Image bên dưới vẫn không đổi.

Mỗi Container có Writable layer riêng. Dữ liệu trong layer này bị xóa cùng Container, vì vậy dữ liệu cần tồn tại độc lập với vòng đời Container nên được đặt trong volume, bind mount hoặc hệ thống lưu trữ bên ngoài phù hợp.

**Ví dụ:** Nếu process trong Container ghi `/tmp/result.txt` mà `/tmp` không phải mount, file đó nằm trong Writable layer và biến mất khi Container bị xóa.

**Liên quan:** [Container](#container), [Filesystem](#filesystem), [Filesystem layer](#filesystem-layer)
