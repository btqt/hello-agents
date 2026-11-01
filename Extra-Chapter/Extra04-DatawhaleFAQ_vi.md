# FAQ các câu hỏi thường gặp về Hello-Agents Datawhale

> Bài viết này được tổng hợp dựa trên QA của buổi livestream ngày 2024-12-01, cũng như các câu hỏi thu thập được trong quá trình xây dựng khóa học đợt đầu, dùng làm \
> tài liệu đọc mở rộng cho khóa học《Hello-Agents Nhập môn từ số 0》của Datawhale.\
> Khuyến nghị học cùng với tài liệu khóa học chính:
> - 🔗 [Tài liệu khóa học](https://datawhalechina.github.io/hello-agents/#/)
> - ⌛️ Video khóa học (sắp ra mắt)

---

## 1. Kiến trúc đa Agent và điều phối song song

<strong>Q1. Hệ thống đa Agent làm thế nào để thực hiện "đa luồng song song"? Các bước có thể chạy song song do Agent lập kế hoạch nhiệm vụ tách ra, làm thế nào để nhiều Agent thực thi tự nhận nhiệm vụ và tự động xử lý dependency? Có framework có sẵn nào không?</strong>

- Tóm tắt ý chính:
  - Trước tiên để "Agent lập kế hoạch nhiệm vụ" tách dependency của nhiệm vụ, hình thành các sub-task có thể song song và có dependency.
  - Tầng thực thi có thể thiết kế thành nhiều Agent chuyên trách, mỗi Agent chỉ xử lý sub-task mà nó phụ trách.
  - Điều phối song song thường được thực hiện thông qua queue / polling API..., đa số kịch bản cần kết hợp với business để tùy chỉnh, không có giải pháp one-click hoàn toàn tổng quát.
- Hướng dẫn trong khóa học: các chương liên quan đến paradigm đa Agent và kiến trúc hệ thống (phần paradigm kinh điển + thực hành framework).

---

## 2. Hệ sinh thái framework và giao thức giao tiếp

### 2.1 Các framework phổ biến và định vị của Hello-Agents

<strong>Q2. Hiện nay có những framework Agent phổ biến nào? Hello-Agents chủ yếu giải quyết vấn đề gì?</strong>

- Tóm tắt ý chính:
  - So sánh hệ thống các framework phổ biến và nhịp độ cập nhật được thảo luận tập trung ở Chương 6 của khóa học, ở đây không liệt kê lại.
  - <strong>Định vị của Hello-Agents</strong>: lấy giảng dạy và học tập làm trọng tâm, nhấn mạnh "cấu trúc rõ ràng + có thể triển khai thực tế + dễ áp dụng linh hoạt", giúp người mới xây dựng khung kiến thức và thực hành Agent hoàn chỉnh.
- Hướng dẫn trong khóa học: Chương 6 "Thực hành phát triển framework".

<strong>Q7. Hello-Agents trông có vẻ đầy đủ chức năng, nếu muốn dùng cho production thì cần bổ sung thêm những khả năng nào?</strong>

- Tóm tắt ý chính:
  - Bản thân framework thiên về "giảng dạy + có thể dùng được", để thực sự triển khai production vẫn cần kết hợp với business để phát triển thêm.
  - Các điểm tăng cường cốt lõi thường ở:
    - Mô hình hóa kiến thức nghiệp vụ và hiểu biết kịch bản;
    - Cơ chế log, giám sát, đánh giá và rollback bền vững hơn;
    - Tối ưu hiệu năng và kiểm soát chi phí.
- Hướng dẫn trong khóa học: phần framework + chương thực hành dự án.

### 2.2 Sự kết hợp giữa Hello-Agents với LangGraph / các framework khác

<strong>Q4. Hello-Agents sử dụng kết hợp với LangGraph như thế nào? Ai gọi ai?</strong>

<strong>Q11 / Q15. Muốn biết làm thế nào để dùng A2A kết hợp Hello-Agents và LangGraph?</strong>

- Tóm tắt ý chính (gộp câu trả lời):
  - Có thể dùng <strong>giao thức A2A + Agent Card</strong>, để "Agent trong một framework" được expose thành "khả năng từ xa" mà framework khác có thể gọi.
  - Tương tự như Function Calling: node của LangGraph có thể coi Agent trong Hello-Agents như một "hàm từ xa" để gọi, và ngược lại cũng vậy.
- Hướng dẫn trong khóa học: Chương 10 "Giao thức giao tiếp Agent". Chương 6 "Thực hành phát triển framework".

<strong>Q17. Trong quá trình học có giới thiệu DeepResearch và các framework open-source / có sẵn khác không?</strong>

- Tóm tắt ý chính:
  - DeepResearch thuộc nội dung Workflow, sẽ được bổ sung ở các phần sau.
  - Nội dung liên quan đến framework tập trung ở <strong>Chương 5, Chương 6</strong>, sẽ đề cập từ "framework đơn lẻ" đến "hệ sinh thái đa framework".
- Hướng dẫn trong khóa học: Chương 5, Chương 6 (framework và các case ứng dụng).

<strong>Q18. Khi viết Agent bằng LangGraph cảm giác giống "kịch bản workflow của large model" hơn, để thực sự triển khai thành dự án cần cân nhắc điều gì? Khóa học có đề cập không?</strong>

- Tóm tắt ý chính:
  - Từ "kịch bản" đến "dự án" cần cân nhắc thêm: ranh giới module, quản lý cấu hình, giám sát và đánh giá, khôi phục lỗi, cũng như các vấn đề kỹ thuật phối hợp nhóm...
  - <strong>Phần 4</strong> của khóa học hướng dẫn mọi người xây dựng một dự án hoàn chỉnh từ số 0, có thể đối chiếu thực hành.
- Hướng dẫn trong khóa học: Phần 4 "Xây dựng dự án".

### 2.3 Giao thức giao tiếp: A2A & ANP

<strong>Q5. Phần A2A và ANP giảng hơi nhanh, có thể giải thích chi tiết hơn và kèm ví dụ code không?</strong>

- Tóm tắt ý chính:
  - Khóa học sẽ triển khai dần theo trình tự "động cơ → trừu tượng hóa → trường của giao thức → ví dụ code".
  - Khuyến nghị đọc thêm chương "Giao thức giao tiếp Agent" trong 《Động thủ học phát triển ứng dụng Agent》của Datawhale (giải thích bởi tác giả cộng đồng ANP).
- Hướng dẫn trong khóa học: Chương 10; có thể xem link trong phần "Đọc mở rộng" của README trỏ đến tài liệu chính thức của Datawhale.

<strong>Q6. Khi thực hành thì nên dùng triển khai local hay dùng API trực tiếp?</strong>

- Tóm tắt ý chính:
  - <strong>Chương 7</strong> của khóa học đưa ra cả hai hướng: triển khai local và API trên cloud.
  - Giai đoạn học tập: ưu tiên API (ngưỡng vào thấp), khi cần kiểm soát chi phí hoặc triển khai offline thì mới cân nhắc local.
  - Hướng dẫn trong khóa học: Chương 10

Hướng dẫn nội dung liên quan trong khóa học:
- 🔗 Chương 5 "Xây dựng Agent dựa trên nền tảng low-code". Nhấp để chuyển đến: [Chương 5](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter5/第五章%20基于低代码平台的智能体搭建.md)
- 🔗 Chương 6 "Thực hành phát triển framework". Nhấp để chuyển đến: [Chương 6](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter6/第六章%20框架开发实践.md)
- 🔗 Chương 7 "Xây dựng framework Agent của riêng bạn". Nhấp để chuyển đến: [Chương 7](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter7/第七章%20构建你的Agent框架.md)
- 🔗 Chương 10 "Giao thức giao tiếp Agent". Nhấp để chuyển đến: [Chương 10](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter10/第十章%20智能体通信协议.md)


---

## 3. Định vị khóa học, lộ trình học và đối tượng phù hợp

<strong>Q9. Các khóa học tiếp theo sẽ đăng trên những nền tảng nào?</strong>

- Tóm tắt ý chính:
  - Repo Github (code và tài liệu khóa học)
  - Bilibili của Datawhale (video)
  - Trang chủ chính thức của Datawhale (cổng khóa học dạng văn bản - hình ảnh)

<strong>Q10. Người hoàn toàn chưa có nền tảng gì có phù hợp để học không?</strong>

- Tóm tắt ý chính:
  - Có thể học được, nhưng cần "học chậm hơn một chút + xem kỹ phần phân tích code".
  - Các bạn chưa có nền tảng Python cần bỏ thêm công sức để bổ sung ngữ pháp, bản thân khóa học không phải độ khó hoàn toàn "không cần lập trình".
  - Về Python khuyến nghị khóa học độc quyền của Datawhale: 🔗 [Học Python bằng cách thông minh](https://datawhalechina.github.io/learn-python-the-smart-way-v2/)
  - Dựa trên nền tảng Python, khóa học này khuyến nghị lộ trình học như sau:
    1. Cấu hình môi trường, lời nói đầu
    2. 🔗 [Chương 1 Làm quen với Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter1/第一章%20初识智能体.md)
    3. 🔗 [Chương 2 Lịch sử phát triển Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter2/第二章%20智能体发展史.md)
    4. 🔗 [Chương 3 Kiến thức nền tảng Large Language Model](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter3/第三章%20大语言模型基础.md)
    5. 🔗 [Chương 4 Xây dựng các paradigm kinh điển của Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter4/第四章%20智能体经典范式构建.md)
    6. 🔗 [Chương 5 Xây dựng Agent dựa trên nền tảng low-code](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter5/第五章%20基于低代码平台的智能体搭建.md)
    7. 🔗 [Chương 6 Thực hành phát triển ứng dụng framework](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter6/第六章%20框架开发实践.md)

<strong>Q12. Mục 1.1 và 2.1 của khóa học trông có vẻ hơi giống nhau, trọng tâm của mỗi phần là gì?</strong>

- Tóm tắt ý chính:
  - <strong>2.1</strong>: hệ thống hóa Agent theo dòng thời gian phát triển, và mở rộng đến các case như hệ chuyên gia, thiên về "mạch lịch sử + góc nhìn toàn cảnh".
  - <strong>1.1</strong>: là phần mở đầu dẫn nhập, giới thiệu đơn giản các khái niệm cơ bản và bối cảnh.

<strong>Q13. Các bạn đã đi làm thì học như thế nào? Học xong thì ứng dụng vào công việc ra sao? Khác gì với các công cụ workflow như n8n?</strong>

- Tóm tắt ý chính:
  - Gợi ý học tập: kết hợp với kịch bản nghiệp vụ của bản thân, ưu tiên làm một ứng dụng Agent "nhỏ nhưng hoàn chỉnh", kể cả chỉ để thay thế một đoạn nhỏ trong quy trình công việc.
  - Khác biệt cốt lõi so với workflow (n8n...):
    - Workflow: quy trình và nhánh về cơ bản cố định, dùng để giải quyết nhiệm vụ "cấu trúc tương đối xác định".
    - Agent: phù hợp với nhiệm vụ phức tạp hơn, không chắc chắn hơn, có thể tự chủ đưa ra quyết định ở mức độ nhất định trong quá trình chạy (gọi tool, lập kế hoạch sub-task...).
- Hướng dẫn trong khóa học: chương khái niệm + phần paradigm kinh điển và thực hành dự án.

<strong>Q14. Khóa học có những chương nào chuyên về test và đánh giá Agent?</strong>

- Tóm tắt ý chính:
  - Có chương đánh giá riêng (như Chương 12), cũng có các case thực hành "đánh giá khép kín" trong các chương khác.
- Hướng dẫn trong khóa học: chương riêng về test và đánh giá + thực hành dự án liên quan.
- 🔗 [Chương 12](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter12/第十二章%20智能体性能评估.md)

<strong>Q16. Làm thí nghiệm trong khóa học thì cấu hình phần cứng tối thiểu là gì? Có cần card đồ họa không?</strong>

- Tóm tắt ý chính:
  - <strong>Chương Agentic RL</strong>: khuyến nghị môi trường GPU với VRAM ≥ 4G sẽ trải nghiệm tốt hơn.
  - Các chương khác dùng API là có thể hoàn thành toàn bộ quá trình học, không bắt buộc phải có GPU local.
- Hướng dẫn trong khóa học: phần giải thích chương Agentic RL liên quan.

<strong>Q20. Sinh viên đang đi học có phù hợp học khóa này không?</strong>

- Tóm tắt ý chính:
  - Rất phù hợp, hãy coi nó như một dự án nhập môn cho hướng giao thoa "AI + Software Engineering".
  - Khuyến nghị kết hợp với đồ án tốt nghiệp / dự án nghiên cứu nhỏ của bản thân, nâng cao giá trị thực hành.
  - Kết quả đồ án tốt nghiệp có thể gửi vào dự án này thông qua hình thức submit PR:
    - 🔗 [Co-creation-projects](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects).

---

## 4. Kiến thức và công cụ: RAG / KAG / Đồ thị tri thức / RL...

<strong>Q3. Xây dựng Agent có nhất thiết phải dùng RAG không?</strong>

- Tóm tắt ý chính:
  - RAG không phải là lựa chọn bắt buộc của Agent, mà là một loại "công cụ tri thức bên ngoài" phổ biến.
  - <strong>Chương 8</strong> của khóa học sẽ hiện thực hóa một phương án RAG từ số 0, giúp hiểu sự khác biệt về thiết kế hệ thống giữa "có RAG / không có RAG".
- Hướng dẫn trong khóa học: Chương 8.
- 🔗 [Chương 8 Memory và Retrieval](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter8/第八章%20记忆与检索.md)

<strong>Q21. KAG / đồ thị tri thức có liên quan gì đến Agent?</strong>

- Tóm tắt ý chính:
  - Có thể hiểu KAG / đồ thị tri thức như một loại "công cụ" hoặc "nền tảng tri thức" của Agent:
    - Agent chịu trách nhiệm ra quyết định và gọi;
    - Đồ thị tri thức cung cấp tri thức có cấu trúc và khả năng retrieval.

<strong>Q22. RL, LLM, RLHF theo tiêu chuẩn phân loại "Agent học tập và Agent dựa trên LLMs" ở Chương 1, thì lần lượt thuộc nhóm nào?</strong>

- Tóm tắt ý chính:
  - RL, LLM, RLHF đều giống như là <strong>thành phần cấu thành hoặc kỹ thuật hiện thực hóa</strong> của Agent, chứ không phải là một "loại Agent" riêng biệt.
  - Ví dụ: LLM có thể đóng vai trò là bộ não của Agent; RL / RLHF có thể dùng để huấn luyện hoặc fine-tune policy của Agent.

---

## 5. Hiệu năng và quản lý context

<strong>Q19. System prompt của nhiệm vụ phức tạp rất dài, mỗi lần gọi tốn nhiều token, khiến phản hồi chậm lại; context càng dài thì vấn đề này càng nghiêm trọng, nên cân bằng như thế nào?</strong>

- Tóm tắt ý chính:
  - Điểm then chốt nằm ở "cắt tỉa và quản lý context": lịch sử hội thoại, kết quả gọi tool, system prompt... cần giữ lại theo tầng, theo cấp độ.
  - Có thể giảm áp lực token thông qua "tóm tắt + module memory + context dạng retrieval".
  - <strong>Chương 9</strong> của khóa học chuyên nói về chiến lược xử lý context.
- Hướng dẫn trong khóa học: Chương 9.
- 🔗 [Chương 9 Context Engineering](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter9/第九章%20上下文工程.md)

---

## 6. Dự án và phát triển sự nghiệp

<strong>Q8. Các dự án case trong khóa học có thể viết vào CV không?</strong>

- Tóm tắt ý chính:
  - Có thể, với điều kiện bạn thực sự hiểu và có thể giải thích rõ ràng:
    - Dự án giải quyết vấn đề gì;
    - Đã sử dụng thiết kế Agent nào;
    - Đã làm những công việc cụ thể gì về gọi tool / đánh giá / triển khai.
  - Khuyến nghị dùng cấu trúc "vấn đề - giải pháp - kết quả" để mô tả các dự án này trong CV, và đính kèm link Github.

---

## 7. Các vấn đề liên quan đến cấu hình môi trường, sử dụng model, gọi API

<strong>Q17. API trong khóa học được thiết lập như thế nào, có trường hợp gọi bị lỗi</strong>

- Tóm tắt ý chính:
  - Các API model mà dự án khóa học hỗ trợ:
    - [Silicon Flow Inference API](https://modelscope.cn/models);
    - [Deepseek API](https://platform.deepseek.com/usage);
    - [OpenAI API](https://platform.openai.com/docs/quickstart);
    - Khác ...
  - Quy trình cấu hình, lấy API_KEY, MODEL_ID, BASE_URL thiết lập trong file biến môi trường `.env`.
  - Cách lấy API model của cộng đồng ModelScope: https://www.modelscope.cn/models/Qwen/Qwen3-VL-8B-Instruct
    - Nhấp vào thư viện model, tìm model hỗ trợ API-Inference, nhấp vào trang chi tiết model, tìm API-Inference
    - ![alt text](./images/Extra04-figures/3f1b68eedc9d9e556fbb51358bf49f9d.png)
    - ![alt text](./images/Extra04-figures/e7dd177f-4867-4af0-bd0e-03771a3a040e.png)


<strong>Q18. Khi hiện thực workflow ReAct, công cụ tìm kiếm web serpApi này có phương án thay thế nào không?</strong>

- Tóm tắt ý chính:
  - Cần truy cập mạng quốc tế, nếu không truy cập được thì đổi phương án khác;
  - Có thể cân nhắc các search engine khác, như: duckduckgo, googlesearch...

<strong>Q21. Model reasoning đang dùng chỉ hỗ trợ streaming output, không thể vào vòng lặp tiếp theo của Agent</strong>

- Tóm tắt ý chính:
  - Các model reasoning như DeepSeek, Qwen... mặc định chỉ cung cấp streaming API, cần tự thực hiện ghép nối đúng cách thay vì chỉ đơn thuần ghép chuỗi string (cách ghép nối cụ thể có thể tham khảo Chương 7)

<strong>Q23. Liên kết memory với knowledge base, không biết nên hiểu như thế nào.</strong>

- Tóm tắt ý chính:
  - Tôi hiểu là: lấy ví dụ, thông thường agent có short-term memory và long-term memory, short-term memory tương đương với những việc chúng ta làm trong một ngày, chúng ta có thể trực tiếp dùng dưới dạng context làm input đưa vào mô hình. Nhưng long-term memory thì tương đương với việc chúng ta ghi chú, chúng ta không thể đưa quá nhiều thông tin vào context một lúc, vì vậy chúng ta làm một tool để cung cấp cho model gọi, model có thể sinh ra một query rồi thông qua cách của RAG để gọi vào knowledge base của chúng ta, đó cũng chính là long-term memory.

<strong>Q24. Mirror nguồn Thanh Hoa (Tsinghua) báo lỗi request, 403</strong>

- Tóm tắt ý chính:
  - Do nguyên nhân mạng, đổi sang mirror USTC (Trung Khoa Đại):
    - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
    - https://mirrors.ustc.edu.cn/anaconda/pkgs/free/

<strong>Q25. Gọi model API báo lỗi 401</strong>

- Tóm tắt ý chính:
  - Số dư tài khoản model không đủ, cần nạp thêm tiền.

**Q27. Model open-source Hugging Face báo lỗi Connection aborted.**

- Tóm tắt ý chính:
  - Do nguyên nhân mạng, khuyến nghị cấu hình bằng HF_ENDPOINT
    - Ví dụ chi tiết xem code/chapter3/Qwen.py
    - https://hf-mirror.com
   
Nếu báo lỗi (MaxRetryError("HTTPSConnectionPool(host='huggingface.co', port=443)" này
có thể thêm vào trong code
```
import os
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"
```
hoặc trực tiếp ở command line
(Linux\mac)
```bash
export HF_ENDPOINT="https://hf-mirror.com"
```
(win's powshell)
```bash
$env:HF_ENDPOINT = "https://hf-mirror.com"
```


## 8. Vấn đề về kiến thức toán học nền tảng

<strong>Q22. Trong công thức xác suất, hiểu P(w_2∣w_1) như thế nào</strong>

- Tóm tắt ý chính:
  - Xác suất có điều kiện trong xác suất học, tức là xác suất w_2 xảy ra với điều kiện w_1 đã xảy ra

## 9. Các vấn đề khác

### 9.1 Liên quan đến đồ án tốt nghiệp

<strong>Q26. Sau khi nộp đồ án tốt nghiệp, làm thế nào để trình bày trong CV và repo cá nhân?</strong>

- Tóm tắt ý chính:
  - Xem chi tiết mục 5 trong [Chương 16](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter16/第十六章%20毕业设计.md)
  - Trong CV khuyến nghị viết như sau (ví dụ):
    -「Trợ lý lập kế hoạch hành trình du lịch thông minh dựa trên framework Hello-Agents」
      - Phụ trách: thiết kế vai trò Agent, điều phối gọi tool, đánh giá RAG retrieval và hội thoại
      - Kết quả: tự động sinh ra phương án hành trình nhiều ngày, hỗ trợ ràng buộc ngân sách và sở thích cá nhân hóa
      - Link: `https://github.com/<your-id>/hello-agents/tree/main/projects/<your-folder>`
  - Khi phỏng vấn cần trình bày rõ trọng tâm:
    - Tại sao lại thiết kế cấu trúc Agent như vậy;
    - Đã sử dụng phương pháp đánh giá nào;
    - Gặp phải những vấn đề gì (ví dụ context quá dài, chi phí gọi...) và bạn đã ứng phó như thế nào.

---
</content>
