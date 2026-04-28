# Roadmap học AI thực dụng để biến nó thành lợi thế

## Mục tiêu của tài liệu này

Tài liệu này được viết theo hướng:

- Học để dùng được ngay trong công việc.
- Lý thuyết vừa đủ để không học mông lung.
- Mỗi tuần đều có đầu ra cụ thể.
- Sau lộ trình, bạn có thể dùng AI để tăng năng suất, build automation nhỏ, và có 1-2 dự án đem đi nói chuyện với sếp hoặc phỏng vấn.

Tài liệu này nghiêng về người làm kỹ thuật, đặc biệt hợp với backend/dev đang làm việc với code, API, SQL, tài liệu, bug, log và quy trình lặp đi lặp lại.

## Kết quả cần đạt sau 8 tuần

- Dùng AI hằng ngày để viết, đọc, tóm tắt, debug và lập kế hoạch làm việc.
- Viết được 1-2 automation nhỏ có ích thật.
- Hiểu các khái niệm cốt lõi: prompt, hallucination, token, context, embeddings, RAG, tool use, eval.
- Làm được 1 project AI thực dụng để đưa vào CV, GitHub hoặc nói chuyện với sếp.
- Biết cách đánh giá đầu ra của AI thay vì tin mù.

## Cách học để không bị ngợp

- Học 60-90 phút mỗi ngày là đủ.
- Mỗi buổi học chia thành 3 phần:
  - 15 phút đọc lý thuyết.
  - 30-45 phút thực hành.
  - 10-15 phút ghi note: hôm nay học được gì, dùng vào việc gì.
- Ưu tiên bài toán thật trong công việc hiện tại.
- Không học quá sâu về toán học, training model, fine-tuning ngay từ đầu.
- Không dùng AI chỉ để chat vui. Phải gắn với đầu ra cụ thể.

## Cần chuẩn bị trước khi bắt đầu

- 1 công cụ chat AI để hỏi đáp và phân tích.
- 1 môi trường code quen tay: IntelliJ, VS Code, hoặc IDE bạn đang dùng.
- GitHub hoặc nơi lưu project cá nhân.
- Có bản về:
  - HTTP/API
  - JSON
  - Git
  - SQL căn bản
- Nếu bạn là dev:
  - giữ Java là stack chính nếu công việc đang dùng Java
  - học thêm Python hoặc TypeScript ở mức đủ dùng để viết script

## Bản đồ lý thuyết cần học

Đây là các lý thuyết cần nắm, theo mức "hiểu để xài", không phải "học để đi thi".

### 1. LLM là gì

- LLM là mô hình dự đoán token tiếp theo dựa trên ngữ cảnh.
- Nó mạnh ở việc sinh văn bản, tóm tắt, phân loại, biến đổi dữ liệu, viết nháp code.
- Nó không "biết thật" theo kiểu con người. Nó có thể trả lời rất trơn tru nhưng vẫn sai.

Cần hiểu:

- `token`: đơn vị xử lý nhỏ của model
- `context window`: lượng thông tin model nhớ trong 1 lần gọi
- `temperature`: mức ngẫu nhiên của đầu ra
- `hallucination`: model tự tạo thông tin không đúng

### 2. Prompting

- Prompt tốt = mục tiêu rõ + đầu vào rõ + ràng buộc rõ + đầu ra mong muốn rõ.
- Prompt không cần hoa mỹ. Chỉ cần đủ thông tin và có cấu trúc.

Khung prompt đơn giản:

1. Nhiệm vụ: "Hãy làm gì?"
2. Ngữ cảnh: "Tài liệu hoặc dữ liệu nào đang có?"
3. Ràng buộc: "Độ dài, định dạng, quy tắc?"
4. Đầu ra: "Muốn nhận về theo mẫu nào?"

### 3. Đánh giá đầu ra

- AI không được phép là "nguồn sự thật".
- Mọi đầu ra quan trọng phải được kiểm lại.

Cần hiểu:

- cách đối chiếu với tài liệu gốc
- cách test code AI viết
- cách yêu cầu model nói rõ giả định
- cách bắt model đưa ra bằng chứng hoặc trích từ input

### 4. Embeddings và RAG

- `embedding` là cách biến text thành vector để tìm nội dung liên quan.
- `RAG` là lấy tài liệu liên quan trước, rồi mới đưa cho model trả lời.
- RAG giúp giảm hallucination khi hỏi trên tài liệu riêng.

Cần hiểu:

- chunking
- retrieval
- top-k
- tại sao "chat với tài liệu" không chỉ là copy file vào prompt

### 5. Tool use và function calling

- Model có thể được nối với công cụ bên ngoài:
  - gọi API
  - tìm database
  - đọc file
  - chạy command
- Đây là lúc AI từ "chat" chuyển thành "làm việc".

### 6. Eval, chi phí, độ trễ, bảo mật

- `eval`: đo chất lượng đầu ra bằng tập câu hỏi hoặc bài test
- `cost`: model tính tiền theo token hoặc request
- `latency`: tốc độ phản hồi
- `privacy`: không ném dữ liệu nhạy cảm vào công cụ mà bạn không được phép dùng

## Lộ trình 8 tuần

## Tuần 1: Dùng AI cho việc hằng ngày

### Mục tiêu

- Biết prompt sao cho ra kết quả đúng việc.
- Dùng AI để tóm tắt, viết lại, lập checklist, giải thích tài liệu.

### Lý thuyết cần học

- Prompt có cấu trúc
- Hallucination
- Context window
- Vai trò của example trong prompt

### Việc làm ngay

- Lấy 1 ticket thật trong công việc, nhờ AI:
  - tóm tắt yêu cầu
  - liệt kê rủi ro
  - viết checklist implementation
- Lấy 1 file tài liệu kỹ thuật, nhờ AI:
  - giải thích lại bằng ngôn ngữ dễ hiểu
  - rút ra 5 điểm quan trọng
- Tạo 3 prompt mẫu cho riêng bạn:
  - prompt tóm tắt yêu cầu
  - prompt lập kế hoạch làm task
  - prompt giải thích đoạn code

### Đầu ra cuối tuần

- 1 file note chứa 3 prompt dùng lại được
- 3 ví dụ đã dùng AI vào việc thật

### Dấu hiệu đã hiểu

- Bạn không hỏi kiểu mơ hồ nữa.
- Bạn biết nói rõ đầu vào, ràng buộc, định dạng đầu ra.

## Tuần 2: Dùng AI để đọc code, viết test, viết tài liệu

### Mục tiêu

- Biến AI thành trợ lý đọc code và viết nháp.
- Giảm thời gian đọc code cũ, log cũ, tài liệu cũ.

### Lý thuyết cần học

- Cách chia đoạn code thành phần để hỏi
- Cách yêu cầu AI chỉ ra giả định, điểm mơ hồ, và rủi ro
- Cách review đầu ra code của AI

### Việc làm ngay

- Chọn 1 class trong project, nhờ AI:
  - giải thích luồng xử lý
  - tìm chỗ dễ null pointer, bug logic, duplicate logic
  - đề xuất unit test cases
- Chọn 1 API, nhờ AI:
  - viết mô tả request/response
  - viết tài liệu ngắn cho đồng đội
- Chọn 1 bug cũ, nhờ AI:
  - liệt kê các giả thuyết gây lỗi
  - đề xuất cách thêm log để bắt được lỗi

### Đầu ra cuối tuần

- 1 file tài liệu ngắn về một thành phần code
- 1 danh sách unit test cases có thể dùng ngay

### Dấu hiệu đã hiểu

- Bạn đã biết sử dụng AI để tăng tốc độ đọc hiểu, không phải giao hết phần suy nghĩ cho nó.

## Tuần 3: Dùng AI cho debug, SQL, log, và phân tích hệ thống

### Mục tiêu

- Dùng AI để xử lý việc kỹ thuật gắn với công việc thật.

### Lý thuyết cần học

- Root cause vs symptom
- Cách đưa log cho AI mà vẫn đủ ngữ cảnh
- Cách mô tả schema, bảng, field để AI viết SQL đúng hơn

### Việc làm ngay

- Lấy 1 đoạn log thật, nhờ AI:
  - tóm tắt sự kiện theo thứ tự
  - khoanh vùng điểm lỗi có khả năng cao
  - đề xuất thêm log nào để xác minh
- Lấy 1 bài toán SQL thật, nhờ AI:
  - viết câu query đầu tiên
  - giải thích join/group by
  - so sánh 2 cách viết query
- Lấy 1 flow nghiệp vụ, nhờ AI vẽ lại:
  - input
  - xử lý
  - output
  - điểm dễ vỡ

### Đầu ra cuối tuần

- 1 tài liệu "debug playbook" nhỏ cho 1 lỗi hoặc 1 flow
- 1 bộ prompt để phân tích log và SQL

### Dấu hiệu đã hiểu

- Bạn bắt đầu tiết kiệm được thời gian thật trong công việc.

## Tuần 4: Viết automation nhỏ bằng script

### Mục tiêu

- Chuyển từ "hỏi AI" sang "dùng AI để tạo ra thứ chạy được".

### Lý thuyết cần học

- Input -> xử lý -> output
- Script là gì, pipeline là gì
- JSON, CSV, file I/O cơ bản
- Khi nào dùng script thay vì ngồi copy paste

### Việc làm ngay

Chọn 1 bài toán lặp đi lặp lại, ví dụ:

- tóm tắt log thành báo cáo ngắn
- chuyển file JSON sang CSV
- tổng hợp ticket thành bảng tổng kết
- sinh test case từ input mô tả API
- rút trích thông tin từ file text hoặc tài liệu nội bộ

Làm theo quy trình:

1. Viết tay 1 lần bằng AI hỗ trợ.
2. Chốt input và output.
3. Viết script nhỏ.
4. Chạy thử trên dữ liệu thật.
5. Sửa cho dùng được nhiều lần.

### Đầu ra cuối tuần

- 1 script có ích thật
- 1 README 5-10 dòng nói script dùng để làm gì

### Dấu hiệu đã hiểu

- Bạn đã có "sản phẩm nhỏ", không còn chỉ là hỏi đáp.

## Tuần 5: Học AI API và cách tích hợp vào ứng dụng

### Mục tiêu

- Hiểu cách gọi model qua API.
- Biết cách đưa prompt vào code.

### Lý thuyết cần học

- API key
- Request/response JSON
- System prompt, user prompt, output format
- Retry, timeout, rate limit
- Cost theo token

### Việc làm ngay

- Viết 1 app nhỏ:
  - nhận input text
  - gọi model
  - trả về output có cấu trúc
- Thử 2 use case:
  - tóm tắt nội dung
  - phân loại ticket/email/log
- Bắt model trả về JSON có schema cố định.

### Đầu ra cuối tuần

- 1 app CLI hoặc web nhỏ gọi được model
- 1 file ghi lại:
  - prompt đã dùng
  - input/output mẫu
  - chi phí ước lượng

### Dấu hiệu đã hiểu

- Bạn không còn xem AI như hộp đen. Bạn biết cách nối nó vào code.

## Tuần 6: Học embeddings và build chat với tài liệu

### Mục tiêu

- Hiểu tại sao muốn hỏi trên dữ liệu riêng thì cần retrieval.
- Làm 1 bản RAG tối giản.

### Lý thuyết cần học

- Embeddings
- Chunking
- Similarity search
- Top-k retrieval
- Tại sao chunk quá dài hoặc quá ngắn đều có vấn đề

### Việc làm ngay

- Chọn 5-20 file tài liệu nội bộ hoặc tài liệu học tập
- Tách nội dung thành chunk
- Tìm cách lấy chunk liên quan nhất cho 1 câu hỏi
- Đưa chunk tìm được vào prompt rồi trả lời
- Yêu cầu model trả lời kèm trích đoạn đã dùng

### Đầu ra cuối tuần

- 1 chatbot hỏi đáp trên bộ tài liệu nhỏ
- 1 note mô tả:
  - tài liệu nguồn
  - cách chunk
  - giới hạn của hệ thống

### Dấu hiệu đã hiểu

- Bạn hiểu RAG là "tìm trước, trả lời sau", không phải phép màu.

## Tuần 7: Tool use, guardrails, và đánh giá

### Mục tiêu

- Làm AI ổn định hơn, dùng được cho việc thật hơn.

### Lý thuyết cần học

- Tool use/function calling
- Output schema
- Guardrails
- Eval
- Human-in-the-loop

### Việc làm ngay

- Thêm 1 tool hoặc 1 bước xử lý vào app tuần 5 hoặc 6:
  - đọc file
  - gọi API nội bộ
  - tìm DB
  - parse JSON
- Tạo 10 câu test để đánh giá:
  - câu nào trả lời đúng
  - câu nào trả lời sai
  - trường hợp nào phải từ chối
- Thêm quy tắc:
  - nếu không đủ dữ liệu thì nói không biết
  - nếu không tìm thấy tài liệu thì báo rõ

### Đầu ra cuối tuần

- 1 bộ eval mini
- 1 hệ thống AI có hành vi ổn định hơn bản đầu

### Dấu hiệu đã hiểu

- Bạn biết "làm AI dùng được" không phải chỉ prompt hay, mà còn là test và ràng buộc.

## Tuần 8: Làm 1 project đủ sức đem đi khoe

### Mục tiêu

- Đóng gói kiến thức thành project có giá trị trình bày.

### Gợi ý project để chọn

- Trợ lý hỏi đáp tài liệu nội bộ cho team
- Công cụ tóm tắt log và gợi ý nguyên nhân lỗi
- Công cụ phân loại ticket hỗ trợ
- Công cụ sinh test case từ mô tả API
- Công cụ tổng hợp báo cáo từ CSV/JSON/email

### Khung làm project

1. Nêu bài toán.
2. Input.
3. Xử lý bằng AI.
4. Output.
5. Giới hạn.
6. Cách đo hiệu quả.

### Việc làm ngay

- Chọn 1 bài toán thật
- Viết README rõ:
  - bài toán
  - cách chạy
  - công nghệ dùng
  - hình ảnh hoặc output mẫu
  - điều gì đã học được
- Nếu được, quay GIF hoặc demo ngắn.

### Đầu ra cuối tuần

- 1 project có README tốt
- 1 đoạn mô tả 5-7 dòng để đưa vào CV hoặc LinkedIn

### Dấu hiệu đã hiểu

- Bạn đã có thứ để chứng minh năng lực, không còn học chay.

## 4 chủ đề nâng cao chỉ học sau khi xong nền tảng

- Multi-agent/agent workflow
- Fine-tuning
- Vector database tối ưu
- Bảo mật, phân quyền, audit, quản lý prompt ở quy mô lớn

Không học sớm nếu bạn chưa:

- dùng AI hằng ngày ổn định
- viết được app gọi model
- build được 1 project RAG hoặc automation nhỏ

## 3 project để ưu tiên nếu bạn muốn để xin việc hoặc tăng giá trị ở công ty

### 1. AI log assistant

Giá trị:

- Giảm thời gian đọc log
- Phù hợp với dev, backend, on-call

Cần học:

- prompt cho log
- classify issue
- summary
- output JSON

### 2. AI doc assistant cho tài liệu nội bộ

Giá trị:

- Dễ nói chuyện với sếp vì nó gắn với productivity
- Giúp bạn học được RAG

Cần học:

- chunking
- retrieval
- citation
- eval

### 3. AI test case generator cho API

Giá trị:

- Gắn với công việc dev và QA
- Dễ giải thích ROI

Cần học:

- OpenAPI/request/response
- edge cases
- output schema
- review đầu ra

## Prompt khung để dùng ngay

### 1. Prompt học lý thuyết

```text
Hãy giải thích cho tôi khái niệm [tên khái niệm] theo mức đủ để dùng được trong công việc.
Bố cục:
1. Định nghĩa ngắn gọn
2. Tại sao nó quan trọng
3. Ví dụ thực tế
4. Lỗi thường gặp
5. Bài tập 15 phút để tôi tự thử
Dùng ngôn ngữ dễ hiểu, tránh học thuật không cần thiết.
```

### 2. Prompt đọc code

```text
Đây là đoạn code của tôi. Hãy:
1. Tóm tắt nó đang làm gì
2. Chỉ ra các điểm dễ lỗi hoặc khó bảo trì
3. Đề xuất unit test cases
4. Nếu cần refactor, hãy đưa ra hướng refactor ít rủi ro nhất
Chỉ đưa ra nhận xét dựa trên code đã thấy. Nếu thiếu ngữ cảnh thì nói rõ.
```

### 3. Prompt phân tích log

```text
Đây là log của 1 sự cố. Hãy:
1. Sắp xếp sự kiện theo thứ tự
2. Chỉ ra điểm bất thường
3. Đưa ra 3 giả thuyết gây lỗi có khả năng nhất
4. Đề xuất log/metric cần thêm để xác minh
5. Cho tôi 1 bản tóm tắt 5 dòng để gửi team
Không được khẳng định chắc chắn nếu không đủ bằng chứng.
```

### 4. Prompt lập kế hoạch build mini project

```text
Tôi muốn làm project AI về [mô tả bài toán].
Hãy giúp tôi:
1. Chốt phạm vi nhỏ nhất có thể demo trong 1 tuần
2. Liệt kê lý thuyết tôi cần học trước
3. Đề xuất stack để làm nhanh
4. Chia task theo ngày
5. Chỉ ra rủi ro chính và cách giảm rủi ro
Ưu tiên hướng đơn giản, ít hạ tầng, dễ chạy được nhất.
```

## Lịch học mẫu 1 tuần

Nếu bạn chỉ có 1 giờ mỗi ngày:

- Thứ 2: học lý thuyết mới + viết note 10 dòng
- Thứ 3: thực hành prompt trên bài toán thật
- Thứ 4: làm 1 output nhỏ có thể dùng lại
- Thứ 5: đọc lỗi và sửa đầu ra AI
- Thứ 6: đóng gói thành script hoặc tài liệu
- Thứ 7: tổng kết, ghi ROI, chốt bài học
- Chủ nhật: nghỉ hoặc xem lại phần chưa rõ

## Những thứ không nên làm lúc mới học

- Nhảy vào training model từ đầu
- Học quá sâu về toán học nếu chưa có use case
- Vung tiền cho hạ tầng phức tạp quá sớm
- Tin output AI mà không test
- Làm dự án quá to đến mức bỏ giữa chừng

## Cách biết mình đang tiến bộ

Sau mỗi tuần, tự hỏi 5 câu:

1. Tuần này AI đã giúp mình tiết kiệm việc gì?
2. Mình đã tạo ra output nào dùng lại được?
3. Mình có case nào AI trả lời sai và mình bắt được không?
4. Mình đã học thêm 1 khái niệm lý thuyết nào và dùng vào đâu?
5. Nếu sếp hỏi "học AI để làm gì", mình đã trả lời được bằng một ví dụ cụ thể chưa?

## Kế hoạch bắt đầu ngay hôm nay

Nếu muốn vào việc ngay, làm theo thứ tự này:

1. Tạo 1 file note riêng tên `ai-learning-log.md`.
2. Viết ra 3 bài toán lặp đi lặp lại trong công việc của bạn.
3. Chọn 1 bài toán nhỏ nhất trong 3 bài toán đó.
4. Dùng prompt để tóm tắt, lập kế hoạch, và tạo output đầu tiên.
5. Cuối ngày ghi lại:
   - đã dùng prompt gì
   - ra kết quả gì
   - sai ở đâu
   - lần sau sửa gì

## Mục tiêu sau 30 ngày

- Có 1 bộ prompt dùng thường xuyên
- Có 1 script hoặc 1 công cụ nhỏ
- Có 1 tài liệu hoặc 1 README do chính bạn viết với AI hỗ trợ
- Có 1 cách giải thích rõ ràng với sếp rằng AI đang giúp bạn tăng năng suất ra sao

## Mục tiêu sau 60-90 ngày

- Có 1 project có thể khoe
- Có 1 số đo hiệu quả thực tế:
  - giảm thời gian đọc log
  - giảm thời gian viết test case
  - giảm thời gian đọc tài liệu
- Có khả năng tự build 1 AI utility nhỏ từ đầu đến cuối

## Câu chốt

Học AI theo cách để sống được với nó là:

- dùng nó vào công việc thật
- biết nó mạnh ở đâu, yếu ở đâu
- tạo ra thứ có thể chạy, có thể đo, có thể giải thích giá trị

Nếu bạn đi hết lộ trình này, bạn sẽ không còn ở vị trí "người bị AI đe dọa". Bạn sẽ ở vị trí "người biết dùng AI để tăng giá trị của mình".
