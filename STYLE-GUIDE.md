# Docker Documentation Style Guide

> Quy chuẩn biên tập cho toàn bộ tài liệu trong repository. Mục tiêu là tạo
> nội dung dễ học với người mới, chính xác về kỹ thuật và nhất quán khi tra cứu.

## 1. Nguyên tắc chung

- Viết nội dung theo hướng Vietnamese-first: giải thích bằng tiếng Việt đầy đủ dấu, rõ ràng và phù hợp với người mới.
- Giữ nguyên thuật ngữ kỹ thuật tiếng Anh mà người học sẽ gặp trong tài liệu chính thức, câu lệnh, log và thông báo lỗi; giải thích ngắn bằng tiếng Việt ở lần xuất hiện đầu tiên.
- Dùng Docker Official Documentation làm nguồn chính. Chỉ dùng OCI Specification khi cần độ chính xác ở mức định dạng hoặc runtime; nguồn thứ cấp không được lấn át nguồn chính thức.
- Ưu tiên mental model chính xác hơn việc ghi nhớ câu lệnh. Mỗi khái niệm quan trọng cần đi từ cách hiểu trực quan đến định nghĩa chính xác, cơ chế hoạt động và ví dụ có thể quan sát.
- Không dùng một thuật ngữ nâng cao chưa được giải thích để định nghĩa một thuật ngữ khác. Mọi phép so sánh hoặc ẩn dụ phải nêu rõ giới hạn.
- Gearhouse chỉ là case study. Lý thuyết dùng chung không được phụ thuộc vào cấu trúc riêng của dự án này.

## 2. Ranh giới Diataxis

Mỗi tài liệu có đúng một mục đích chính. Khi cần nội dung thuộc loại khác, hãy liên kết đến tài liệu phù hợp thay vì trộn nhiều mục đích hoặc lặp lại nội dung.

| Loại | Câu hỏi của người đọc | Luồng bắt buộc | Không được trộn |
|---|---|---|---|
| Tutorial | "Tôi học điều này bằng cách thực hành như thế nào?" | prerequisites -> steps -> observation -> verification -> cleanup | Không biến thành bài giải thích lý thuyết dài, danh mục tùy chọn đầy đủ hoặc quy trình xử lý một sự cố riêng lẻ. |
| How-to | "Tôi hoàn thành một tác vụ cụ thể như thế nào?" | problem -> current-state check -> change -> verification -> recovery | Không dạy từ đầu như Tutorial, diễn giải toàn bộ mental model hoặc liệt kê mọi cú pháp liên quan. |
| Explanation | "Tại sao Docker hoạt động theo cách này?" | problem -> intuition -> precise model -> mechanism -> misconception | Không biến thành dự án từng bước, runbook vận hành hoặc trang tra cứu tùy chọn. |
| Reference | "Cú pháp hoặc tùy chọn chính xác là gì?" | quick table -> exact syntax -> options -> examples -> related entries | Không dẫn dắt theo bài học dài, kể chuyện theo tình huống hoặc thay thế Tutorial và How-to. |

Các luồng chính phải được giữ nguyên khi lập dàn ý:

```text
Tutorial: prerequisites -> steps -> observation -> verification -> cleanup
How-to: problem -> current-state check -> change -> verification -> recovery
Explanation: problem -> intuition -> precise model -> mechanism -> misconception
Reference: quick table -> exact syntax -> options -> examples -> related entries
```

Explanation có thể chứa hai đến bốn lệnh quan sát ngắn, nhưng mỗi lệnh phải chứng minh một nhận định lý thuyết. Một quy trình thực hành hoàn chỉnh phải chuyển sang Tutorial; cú pháp đầy đủ phải chuyển sang Reference.

## 3. Nhịp trình bày

Chapter nội dung áp dụng nhịp trình bày lấy cảm hứng từ tài liệu NGINX nhưng vẫn ưu tiên độ chính xác và khả năng đọc trên GitHub:

1. Dùng H1 có số thứ tự cho tên chapter và đánh số các section chính.
2. Đặt khối mental model, metadata và nguồn chính gần đầu trang.
3. Dùng đường phân cách ngang giữa các phase đọc lớn, không chèn sau mọi section.
4. Đặt ví dụ cụ thể ngay sau phần giải thích trừu tượng mà ví dụ đó làm sáng tỏ.
5. Dùng bảng ngắn cho so sánh, ánh xạ và tra cứu nhanh; không ép văn xuôi tuần tự vào bảng.
6. Kết thúc bằng phần tóm tắt ngắn, hướng học tiếp, nguồn tham khảo và navigation lặp lại.

Tutorial đặt tên bước theo `Bước 1`, `Bước 2`, ... và luôn chỉ rõ điều cần quan sát, cách xác minh kết quả, lỗi thường gặp và cleanup. How-to giữ luồng ngắn: vấn đề, kiểm tra trạng thái hiện tại, thay đổi, xác minh và recovery.

Không dùng:

- Chữ đậm quá mức (`excessive bold text`) làm mất phân cấp thị giác.
- Khẳng định tuyệt đối không được hỗ trợ (`unsupported absolute claims`) bởi nguồn chính thức hoặc bằng chứng kỹ thuật.
- Hình ảnh hot-link từ bên thứ ba (`third-party hot-linked images`). Tài sản raster bắt buộc phải được lưu trong repository.
- Câu lệnh đứng riêng mà không giải thích cú pháp, phần output cần đọc hoặc điều nó chứng minh.
- Nhiều mục đích Diataxis trong cùng một tài liệu.

## 4. Khung Explanation chuẩn

Dùng khung sau làm checklist có thể sao chép. Chỉ giữ section thực sự giúp giải thích chủ đề; không tạo nội dung gượng ép chỉ để lấp đầy khung.

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

Link về mục lục phần luôn hiện diện. Chỉ thêm link chapter trước và chapter sau khi các file đích đã tồn tại; không tạo link giả cho nội dung mới nằm trong roadmap. Navigation hợp lệ phải xuất hiện ở cả đầu và cuối chapter.

Một Explanation hoàn chỉnh nên đi theo bốn lớp chiều sâu:

```text
Intuitive understanding
-> precise definition
-> operating mechanism
-> observable example
```

Phần quan niệm dễ gây hiểu nhầm chỉ chọn các nhận định mà người mới hoặc developer có thể tin một cách hợp lý. Mỗi nhận định cần có phân loại, lý do nghe có vẻ đúng, lỗi kỹ thuật chính xác, cách diễn đạt tốt hơn và bằng chứng kiểm tra.

## 5. Syntax Deep-Dive

Mọi Dockerfile instruction, Compose key, CLI option, path hoặc compact syntax quan trọng phải theo thứ tự:

```text
Syntax -> syntax tree -> token table -> resolved values -> before/after state
-> similar syntax -> mistakes -> misconceptions -> verification
```

Yêu cầu cụ thể:

1. Bắt đầu bằng cú pháp chính thức hoặc dạng tổng quát.
2. Vẽ hoặc mô tả cây cú pháp và giải thích từng token, key, index, dấu phân cách.
3. Xác định parser và scope sở hữu từng giá trị.
4. Resolve path tương đối, biến nội suy và giá trị mặc định thành ví dụ cụ thể.
5. Mô tả filesystem, resource hoặc lifecycle state trước và sau khi thực thi.
6. So sánh với cú pháp trông tương tự nhưng có hành vi khác.
7. Nêu lỗi hợp lý, misconception và tác động của giá trị bị bỏ qua.
8. Kết thúc bằng lệnh hoặc output có thể xác minh lời giải thích.

Ví dụ bắt buộc:

```dockerfile
WORKDIR /app
COPY app.jar app.jar
```

| Token | Scope | Giá trị đã resolve |
|---|---|---|
| `WORKDIR` | Image đang được build | Đặt working directory cho các instruction tiếp theo thành `/app`. |
| `app.jar` thứ nhất | Build context | Source path trỏ đến file `app.jar` trong build context. |
| `app.jar` thứ hai | Image filesystem | Destination path tương đối với `WORKDIR`, nên resolve thành `/app/app.jar`. |

Hai chuỗi `app.jar` giống nhau về chữ nhưng không trỏ đến cùng một vị trí. Giá trị thứ nhất thuộc filesystem được gửi vào builder dưới dạng build context; giá trị thứ hai thuộc filesystem của Image đang được tạo. Trước `COPY`, Image chưa có file đích do instruction này tạo. Sau `COPY`, nội dung từ source trong build context xuất hiện tại `/app/app.jar` trong Image filesystem.

## 6. Thuật ngữ

Lần xuất hiện đầu tiên của thuật ngữ quan trọng dùng mẫu:

**Writable layer** — lớp cho phép Container ghi dữ liệu trong quá trình chạy.

Quy tắc thuật ngữ:

- Giữ English term và giải thích bằng tiếng Việt ngay trong câu đầu tiên.
- Dùng chữ đậm cho khái niệm mới được giới thiệu; không tiếp tục bôi đậm mọi lần xuất hiện sau đó.
- Dùng inline code cho câu lệnh, option, key, path, filename và literal value.
- Với ba đến sáu thuật ngữ khó trong một section, dùng `NOTE` hoặc `IMPORTANT` theo mục 7, hoặc dùng bảng thuật ngữ thông thường thay vì tạo loại callout mới.
- Khi cần định nghĩa sâu hơn, liên kết đến glossary trung tâm bằng relative repository link.
- Một định nghĩa đầy đủ theo thứ tự: cách hiểu nhanh, nghĩa Docker chính xác, ví dụ hoặc ẩn dụ, rồi giới hạn của ẩn dụ.
- Không dùng một danh sách thuật ngữ cố định cho mọi chapter. Mỗi chapter tự nhận diện các từ mới, khó hoặc có nghĩa chuyên biệt trong đúng chủ đề đang học; giải thích tại chỗ trước, rồi chỉ đưa vào Glossary khi thuật ngữ có giá trị tái sử dụng ở nhiều chapter.
- Một từ tiếng Anh phổ biến không bắt buộc có hộp chú thích riêng nếu câu chứa nó đã giải thích đủ rõ. Ngược lại, một từ tưởng quen nhưng có nghĩa Docker-specific phải được làm rõ ngay tại lần dùng đầu tiên.

Khi một thuật ngữ dẫn người đọc sang Glossary, chapter phải đặt một source anchor ngay tại lần xuất hiện đó. Glossary thêm mục **Quay lại nơi đang học** ở cuối định nghĩa và trỏ về source anchor, để người đọc trở lại đúng đoạn thay vì đầu chapter.

Quy ước anchor:

```markdown
<a id="back-03-docker-image-filesystem"></a>
**[Filesystem](../../reference/glossary.md#filesystem)** — cây thư mục và file mà môi trường nhìn thấy.
```

Backlink trong Glossary:

```markdown
**Quay lại nơi đang học:** [3. Docker Image](../learning-path/01-foundations/03-docker-image.md#back-03-docker-image-filesystem)
```

- Dùng mẫu `back-<chapter-slug>-<term-slug>` và giữ anchor duy nhất trong mỗi file.
- Nếu nhiều chapter cùng dẫn tới một thuật ngữ, Glossary liệt kê các backlink trên cùng một dòng, phân cách bằng `·`.
- Backlink chỉ trỏ tới file và anchor đã tồn tại; không dùng JavaScript history vì GitHub Markdown không bảo đảm hỗ trợ.

## 7. Callout

Chỉ dùng bốn loại GitHub callout sau:

```markdown
> [!NOTE]
> Ngữ cảnh hỗ trợ cần thiết.

> [!TIP]
> Cải tiến thực tế hoặc cách xác minh nhanh hơn.

> [!IMPORTANT]
> Khái niệm phải được hiểu chính xác.

> [!WARNING]
> Rủi ro lỗi, bảo mật hoặc mất dữ liệu.
```

Tên hợp lệ là `NOTE`, `TIP`, `IMPORTANT`, `WARNING`. Callout phải là ngoại lệ; nội dung giải thích thông thường vẫn viết dưới dạng văn xuôi. Dùng `WARNING` trước lệnh phá hủy dữ liệu và nêu rõ ảnh hưởng cùng khả năng recovery.

## 8. Code và lệnh terminal

- Mọi fenced code block phải có language label. Các label bắt buộc theo loại nội dung là `bash`, `powershell`, `dockerfile`, `yaml`, `mermaid`, `text`; dùng `markdown` khi minh họa chính Markdown.
- Ưu tiên lệnh một dòng để dùng được trên nhiều shell.
- Khi cú pháp multiline khác nhau, cung cấp riêng bản Bash dùng `\` và PowerShell dùng backtick. Cảnh báo rằng khoảng trắng sau backtick sẽ làm hỏng line continuation trong PowerShell.
- Phân biệt rõ global option, command option, argument, service name, Container name, path và command chạy bên trong Container.
- Với mỗi lệnh quan trọng, giải thích vấn đề được giải quyết, cú pháp chính thức, từng token, parser sở hữu option, state transition, lệnh tương tự, rủi ro recovery và verification.

Cú pháp CLI nền tảng:

```text
docker [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [ARGUMENTS]
docker compose [GLOBAL OPTIONS] COMMAND [COMMAND OPTIONS] [SERVICE...]
```

Không coi code là tự giải thích. Ngay sau code block phải có văn bản hoặc bảng chỉ rõ phần người đọc cần quan sát và kết luận kỹ thuật có thể rút ra.

## 9. Diagram và hình ảnh

- Dùng Mermaid cho diagram nằm trong repository.
- Chỉ thêm diagram khi nó làm rõ relationship, sequence, hierarchy hoặc state change khó diễn đạt bằng văn xuôi.
- Mỗi diagram phải có điểm bắt đầu, hướng đọc rõ, vocabulary nhỏ và nhất quán.
- Giải thích diagram trong phần văn xuôi liền kề; không để người đọc tự suy đoán ý nghĩa node hoặc arrow.
- Tránh màu trang trí và node không mang thông tin. Diagram phải đọc được trong GitHub Markdown.
- Hình raster cần thiết phải được lưu trong repository với tên lowercase ASCII kebab-case; không hot-link ảnh bên thứ ba.

## 10. Tên file, heading, link và navigation

- Filename dùng lowercase ASCII kebab-case; file trong learning path có numeric prefix, ví dụ `03-docker-image.md`.
- Mỗi Markdown file có đúng một H1 và không bỏ qua heading level.
- Chapter nội dung dùng H1 và các major section có số thứ tự. Các heading con tiếp tục đúng hierarchy thay vì chọn level theo kích thước hiển thị.
- Navigation ở đầu và cuối chapter. Mục lục phần luôn có link; link previous và next chỉ xuất hiện khi file đích đã tồn tại.
- Internal content chỉ dùng relative repository links. Không dùng đường dẫn tuyệt đối, URI cục bộ hoặc link phụ thuộc máy của tác giả.
- Roadmap chưa có file đích phải hiển thị dưới dạng plain text, không tạo broken link có chủ ý.
- Link ngoài chỉ dùng cho nguồn và tài nguyên thực sự cần thiết; text của link phải mô tả được đích đến.

## 11. Nguồn và kiểm tra độ chính xác

Khối nguồn chính đặt gần đầu chapter và trỏ đến Docker Official Documentation. Dùng OCI Image Specification khi cần giải thích chính xác cấu trúc Image; bài viết và thảo luận cộng đồng chỉ hỗ trợ phát hiện điểm dễ nhầm.

Trước khi xuất bản:

1. Đối chiếu định nghĩa với tài liệu chính thức hiện hành.
2. Kiểm tra cú pháp lệnh với Docker CLI hiện tại khi có thể.
3. Xác nhận ví dụ, output và diagram không mâu thuẫn với văn xuôi.
4. Nêu giới hạn của analogy và tránh absolute claim không có bằng chứng.
5. Khi trích dẫn trực tiếp nội dung cộng đồng, cung cấp link, đủ ngữ cảnh và đối chiếu với nguồn có thẩm quyền; phê bình ý tưởng, không phê bình người viết.

## 12. Checklist trước khi xuất bản

- Tài liệu có một mục đích Diataxis rõ ràng và giữ đúng luồng của loại đó.
- Người mới có thể diễn đạt mental model bằng lời của mình thay vì chỉ lặp lại định nghĩa.
- English term được giải thích bằng tiếng Việt ở lần đầu và không bị định nghĩa lặp lại vô ích.
- Code compact được phân tích theo Syntax Deep-Dive và có verification.
- Command quan sát chứng minh lý thuyết; command phá hủy có warning và recovery.
- Diagram được giải thích, có hướng đọc và khớp với prose.
- Mọi code fence có language label; heading không nhảy level; file có đúng một H1.
- Navigation xuất hiện ở đầu và cuối; mọi internal link là relative và trỏ đến file đã tồn tại.
- Chapter có nguồn chính thức, tóm tắt cuối trang và không chứa placeholder biên tập.
- Vietnamese prose giữ đầy đủ dấu; định dạng GitHub-Flavored Markdown hiển thị rõ ràng.
