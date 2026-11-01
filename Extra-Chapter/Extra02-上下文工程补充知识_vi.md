# Kiến thức bổ sung về Context Engineering

## Giới thiệu

Tại sao context engineering gần đây lại nóng trở lại? Bắt nguồn từ cuộc trò chuyện của Jeff - nhà sáng lập kiêm CEO của Chroma trên [podcast](https://youware.app/project/7529x70z4p) Len Space,
Chroma là ông trùm mã nguồn mở trong lĩnh vực vector database. Ngay cả paper nổi tiếng Voyager cũng sử dụng nó.
Tiêu đề cuộc trò chuyện của CEO Jeff chính là về quan điểm "RAG is dead", trong video đã nói rõ ràng về hạn chế của RAG truyền thống và tầm quan trọng của context engineering hiện nay,

![alt text](./images/Extra02-figures/image-1.png)


Trong chương này, trước tiên chúng ta sẽ giải thích toàn diện khái niệm "context engineering",
và ở cuối bài sẽ bàn luận về quan điểm "RAG is dead"



## Context Engineering là gì?

Chúng ta có thể lấy một ví dụ so sánh, Agent giống như một [hệ điều hành kiểu mới](https://www.youtube.com/watch?si=-aKY-x57ILAmWTdw&t=620&v=LCEmiRjPEtQ&feature=youtu.be&ref=blog.langchain.com). LLM giống như CPU, [context window](https://docs.anthropic.com/en/docs/build-with-claude/context-windows?ref=blog.langchain.com) của nó giống như RAM, đóng vai trò là bộ nhớ làm việc của mô hình. Giống như RAM, [dung lượng của context window](https://lilianweng.github.io/posts/2023-06-23-agent/?ref=blog.langchain.com) của LLM cũng có giới hạn, không thể xử lý context từ mọi nguồn khác nhau. Và context engineering giống như cách hệ điều hành quản lý RAM của CPU, dùng để quản lý context window của LLM, quyết định khi nào cần điền vào nội dung gì. [Karpathy đã tổng kết rất hay](https://x.com/karpathy/status/1937902205765607626?ref=blog.langchain.com):
_"Context engineering là... nghệ thuật và khoa học tinh tế của việc điền vào context window đúng thông tin cần thiết cho bước tiếp theo."_

![llm_context_engineering](https://blog.langchain.com/content/images/2025/07/image-1.png)


















## [Khái niệm Context Engineering](https://blog.langchain.com/context-engineering-for-agents/)

![alt text](./images/Extra02-figures/image-2.png)

Context chính là tất cả những gì mô hình "nhìn thấy", mô hình thực ra không chỉ dựa vào prompt mà chúng ta nhập vào để trả lời câu hỏi, mà còn có các thông tin khác phối hợp để tạo ra câu trả lời. Context engineering là một khái niệm bao quát áp dụng cho một vài loại context khác nhau:

- <strong>Instructions (context chỉ dẫn)</strong>: prompt, memory, few-shot example... thuộc prompt engineering, bao gồm:
  - System prompt: định nghĩa vai trò, quy tắc hành vi và phong cách phản hồi của AI
  - User instructions: mô tả nhiệm vụ cụ thể và yêu cầu
  - Few-shot examples: ví dụ input-output, giúp hiểu định dạng mong đợi
  - Tool descriptions: quy cách và hướng dẫn sử dụng của function hoặc tool
  - Format constraints: yêu cầu định dạng và cấu trúc của output
- <strong>Knowledge (context tri thức)</strong>: sự thật, knowledge base... thuộc RAG, bao gồm:
  - Domain knowledge: thông tin sự thật của một ngành hoặc lĩnh vực cụ thể
  - Memory: sở thích người dùng, lịch sử tương tác và bản ghi hội thoại
  - Knowledge base: thông tin liên quan lấy từ database hoặc knowledge base
  - Real-time data: thông tin trạng thái hiện tại được cập nhật động
- <strong>Tools (context công cụ)</strong>: mô tả công cụ và phản hồi khi gọi công cụ thuộc agent, bao gồm:
  - Function calling results: phản hồi API hoặc kết quả truy vấn
  - Tool execution status: phản hồi thành công, thất bại hoặc lỗi
  - Multi-step tool chain: quan hệ phụ thuộc và truyền dữ liệu giữa các tool
  - Execution history: bản ghi và kết quả của các lần gọi tool






### Ví dụ — trợ lý thông minh của một app du lịch


![alt text](./images/Extra02-figures/image-5.png)


Để phân biệt rõ ràng bốn khái niệm này, chúng ta thiết lập một tình huống thực tế thống nhất, sau đó xem mỗi phương pháp giải quyết vấn đề này như thế nào.

<strong>Tình huống: Trợ lý thông minh của một app du lịch</strong>

<strong>Yêu cầu người dùng:</strong> "Giúp tôi lên kế hoạch cho chuyến du lịch gia đình 3 ngày ở Bắc Kinh. Chúng tôi gồm hai người lớn và một trẻ 5 tuổi, thích lịch sử văn hóa, cũng muốn có một vài hoạt động nhẹ nhàng thú vị. Tổng ngân sách của chúng tôi là 8000 tệ."

---

#### 1. Prompt Engineering

Đây là phương pháp cơ bản nhất, trực tiếp nhất. Cốt lõi của nó là <strong>làm thế nào để đặt một câu hỏi hay cho language model (LLM)</strong>, với kỳ vọng rằng chỉ dựa vào knowledge base nội tại tổng quát của nó cũng có thể đưa ra câu trả lời tốt nhất.

*   <strong>Ý tưởng cốt lõi:</strong> Tối ưu hóa instruction (Prompt) đưa vào mô hình, để nó cho ra kết quả phù hợp hơn với mong đợi.
*   <strong>Cách thức hoạt động:</strong>
    1.  Developer hoặc người dùng xây dựng cẩn thận tất cả yêu cầu thành một prompt chi tiết.
    2.  Gửi prompt này trực tiếp đến một large language model tổng quát (như GPT-4).
    3.  Mô hình hoàn toàn dựa vào kiến thức nội tại của nó tính đến ngày huấn luyện (ví dụ năm 2023) để trả lời.

*   <strong>Ví dụ:</strong>
    ```
    你是一位专业的旅行规划师。请为北京一个为期三天的家庭旅行设计一份详细行程。
    
    # 家庭成员
    - 2个成年人
    - 1个5岁的儿童
    
    # 兴趣偏好
    - 历史文化（故宫、长城等）
    - 轻松有趣的儿童活动
    
    # 预算
    - 总预算不超过8000元人民币，请给出大致的费用估算。
    
    # 输出要求
    - 每日行程安排（上午、下午、晚上）
    - 交通建议
    - 餐饮推荐（包含适合儿童的餐厅）
    - 预算明细
    ```

*   <strong>Hạn chế:</strong>
    *   <strong>Thông tin lỗi thời:</strong> Không thể cung cấp giá vé theo thời gian thực, giờ mở cửa hoặc thông tin giao thông mới nhất.
    *   <strong>Thông tin không chính xác:</strong> Ước tính ngân sách có thể rất thô, vì nó không biết giá khách sạn và vé máy bay hiện tại.
    *   <strong>Thiếu cá nhân hóa:</strong> Không thể gợi ý dựa trên sở thích lịch sử của người dùng.
    *   <strong>"Nói nghiêm túc nhưng toàn nhảm":</strong> Có thể bịa ra một "khu vui chơi trẻ em" hoặc nhà hàng không tồn tại.

---

#### 2. Retrieval-Augmented Generation (RAG)

Để giải quyết vấn đề "kiến thức lỗi thời" của prompt engineering, RAG đưa vào một <strong>knowledge base bên ngoài</strong>.

*   <strong>Ý tưởng cốt lõi:</strong> Trước khi sinh câu trả lời, truy xuất thông tin liên quan từ một database cụ thể, đáng tin cậy, sau đó cung cấp thông tin này cùng với câu hỏi của người dùng cho mô hình.
*   <strong>Cách thức hoạt động:</strong>
    1.  <strong>Chuẩn bị knowledge base:</strong> Chuẩn bị trước một database chứa cẩm nang du lịch mới nhất, giới thiệu địa điểm tham quan, danh sách khách sạn, đánh giá nhà hàng (ví dụ một đống PDF, trang web hoặc bản ghi database).
    2.  <strong>Retrieve (Truy xuất):</strong> Khi người dùng đặt câu hỏi, hệ thống trước tiên tìm kiếm trong knowledge base các đoạn tài liệu liên quan đến "du lịch gia đình Bắc Kinh", "địa điểm lịch sử văn hóa".
    3.  <strong>Augment (Tăng cường):</strong> Ghép thông tin đã truy xuất (ví dụ: "Giá vé mới nhất của Cố Cung là 60 tệ, đóng cửa thứ Hai", "Universal Studios Bắc Kinh là điểm đến gia đình phổ biến") cùng câu hỏi gốc của người dùng thành một prompt mới, phong phú hơn.
    4.  <strong>Generate (Sinh kết quả):</strong> Gửi prompt đã tăng cường này đến LLM, để nó tạo ra hành trình dựa trên các tư liệu "mới" này.

*   <strong>Ví dụ:</strong>
    Hệ thống tìm thấy ba đoạn văn bản trong knowledge base nội bộ: A) Giờ mở cửa và giá vé trên trang chính thức của Cố Cung; B) Một bài blog về "đưa trẻ đi Thiên Đàn"; C) Danh sách "khách sạn thân thiện với gia đình ở Bắc Kinh".
    Sau đó, nó gửi chỉ dẫn đến LLM: "Dựa trên thông tin sau: [nội dung đoạn A, B, C], hãy lên kế hoạch cho chuyến du lịch gia đình 3 ngày ở Bắc Kinh cho người dùng, ngân sách 8000 tệ."

*   <strong>Hạn chế:</strong>
    *   <strong>Phản hồi bị động:</strong> Nó chỉ có thể trả lời dựa trên thông tin bạn cung cấp, không thể chủ động thực hiện nhiệm vụ. Nó không thể "tra cứu" vé máy bay, chỉ có thể dùng thông tin vé máy bay "có sẵn" trong database của bạn.
    *   <strong>Tương tác một chiều:</strong> Hoàn thành một lần retrieval và generation là kết thúc, không thể thực hiện suy luận và hành động nhiều bước.
    *   <strong>Phụ thuộc vào knowledge base:</strong> Hiệu quả tốt hay xấu phụ thuộc nghiêm trọng vào chất lượng và tần suất cập nhật của knowledge base.

---

#### 3. Agent (Tác tử thông minh)

Agent giúp AI tiến hóa từ một "robot hỏi đáp" thành <strong>một "người hành động" có thể suy nghĩ, có thể sử dụng công cụ</strong>.

*   <strong>Ý tưởng cốt lõi:</strong> Trao cho mô hình một vòng lặp "suy nghĩ - hành động" (Reasoning-Action Loop), để nó có thể tự chủ lập kế hoạch các bước, sử dụng công cụ bên ngoài (như API) để hoàn thành nhiệm vụ phức tạp.
*   <strong>Cách thức hoạt động:</strong>
    1.  <strong>Suy nghĩ và lập kế hoạch:</strong> LLM (đóng vai trò là bộ não của Agent) sau khi nhận được yêu cầu người dùng, sẽ suy nghĩ trước: "Để hoàn thành nhiệm vụ này, tôi cần: 1. Tra giá vé máy bay và khách sạn; 2. Tra giá vé tham quan; 3. Lên kế hoạch lộ trình; 4. Tổng hợp thành hành trình."
    2.  <strong>Chọn công cụ (Action):</strong> Nó quyết định dùng công cụ đầu tiên: `search_flight_api(from="上海", to="北京", date="...")`.
    3.  <strong>Quan sát kết quả (Observation):</strong> API trả về giá vé máy bay: 5000 tệ.
    4.  <strong>Suy nghĩ lại:</strong> "Vé máy bay tốn 5000, ngân sách còn lại 3000. Tôi cần tìm khách sạn có giá dưới 800 tệ mỗi đêm."
    5.  <strong>Hành động tiếp:</strong> Sử dụng công cụ `search_hotel_api(city="北京", price_max=800, family_friendly=true)`.
    6.  Vòng lặp này sẽ tiếp tục cho đến khi nó thu thập đủ thông tin cần thiết, cuối cùng hoàn thành kế hoạch.

*   <strong>Ví dụ:</strong>
    Trợ lý này sẽ hoạt động giống như một trợ lý con người thực thụ:
    *   "Được rồi, tôi đang tra cứu cho bạn... Tôi thấy vé máy bay đi Bắc Kinh thứ Sáu tuần sau tốn khoảng 5000 tệ."
    *   "Xét theo ngân sách, tôi đã lọc cho bạn vài khách sạn gia đình có đánh giá tốt và giá 600-800 tệ/đêm."
    *   "Vé Cố Cung đã được tra cứu qua `ticket_api`, trẻ em miễn phí. Tôi đã thêm thông tin này vào hành trình."

*   <strong>Hạn chế:</strong>
    *   <strong>Phức tạp và không ổn định:</strong> Đường đi hành vi của Agent không cố định, có thể mắc lỗi (như rơi vào vòng lặp, dùng sai công cụ), khó debug và kiểm soát.
    *   <strong>Chi phí cao:</strong> Mỗi bước suy nghĩ và gọi công cụ có thể là một lần gọi LLM API, chi phí khá cao.

---

#### 4. Context Engineering

Context engineering là <strong>một bộ môn vĩ mô hơn, nghiêm ngặt hơn</strong>, nó tập trung vào <strong>cách xây dựng "context window" tối ưu nhất cho mô hình (dù là RAG đơn giản hay Agent phức tạp)</strong>. Nó là sự tối ưu hóa và nâng tầm của tất cả các phương pháp nêu trên.

*   <strong>Ý tưởng cốt lõi:</strong> Thiết kế và sắp xếp cẩn thận tất cả thông tin đi vào context của mô hình (instruction, dữ liệu đã retrieve, lịch sử hội thoại, output của tool...), để đạt được output hiệu quả nhất, đáng tin cậy nhất. Đây là một khoa học về "cho ăn gì" và "cho ăn như thế nào".
*   <strong>Cách thức hoạt động:</strong>
    Nó không phải là một loại hệ thống độc lập, mà là phương pháp luận để tối ưu RAG và Agent. Quay lại ví dụ lập kế hoạch du lịch:
    1.  <strong>Giai đoạn thu thập (Gather):</strong>
        *   <strong>Truy xuất song song:</strong> Không chỉ truy xuất từ kho cẩm nang du lịch (RAG), nó còn đồng thời:
            *   Gọi `weather_api` để tra thời tiết Bắc Kinh những ngày tới.
            *   Gọi `events_api` để tra xem có triển lãm hoặc hoạt động dành cho trẻ em đặc biệt nào không.
            *   Truy xuất từ database hồ sơ người dùng (CRM) thông tin "người dùng này lần du lịch trước đã đặt vé bảo tàng".
            *   Thực hiện tìm kiếm đa chiều cho câu hỏi mơ hồ của người dùng "hoạt động nhẹ nhàng thú vị", bao gồm "khu vui chơi Bắc Kinh", "bảo tàng khoa học kỹ thuật Bắc Kinh", "biểu diễn phù hợp trẻ em".
    2.  <strong>Giai đoạn chọn lọc và nén (Glean & Compact):</strong>
        *   <strong>Xếp hạng lại:</strong> Nó phát hiện dự báo thời tiết cho thấy ngày thứ hai có mưa, vì vậy hạ thấp độ ưu tiên của Vạn Lý Trường Thành ngoài trời, nâng trọng số gợi ý của bảo tàng khoa học trong nhà.
        *   <strong>Nén:</strong> Nó sẽ không đưa cả một bài đánh giá khách sạn dài dòng cho mô hình, mà trích xuất thông tin then chốt: "Khách sạn này có khu vui chơi trẻ em, cung cấp nôi cho bé."
        *   <strong>Định dạng:</strong> Nó tích hợp tất cả thông tin lộn xộn đã thu thập được (thời tiết, vé máy bay, sở thích người dùng, giới thiệu địa điểm) thành một đối tượng JSON có cấu trúc cao, gọn gàng rõ ràng.
    3.  <strong>Bàn giao cuối cùng:</strong> Cuối cùng, nó giao gói context "hoàn hảo" này cho bộ não của Agent (LLM), chỉ dẫn có thể là: "Hãy dựa trên dữ liệu có cấu trúc đã được xác thực, đã được sắp xếp này `[JSON object]`, tạo ra hành trình cuối cùng cho người dùng."

*   <strong>Ví dụ:</strong>
    Sản phẩm đầu ra của context engineering không phải là hành trình đưa trực tiếp cho người dùng, mà là một "bản đồ tác chiến" được tối ưu hóa dành cho mô hình xem. Vì đã được context engineering tối ưu hóa, công việc của Agent trở nên cực kỳ đơn giản và hiệu quả, nó không cần phải tự mình thử-sai từng bước một cách vất vả, mà dựa trên một bản tóm tắt hoàn hảo để trực tiếp thực hiện việc sinh kế hoạch cuối cùng.

#### Tổng kết so sánh

| Khái niệm | Ý tưởng cốt lõi | Cách thức hoạt động | Hạn chế |
| :--- | :--- | :--- | :--- |
| <strong>Prompt Engineering</strong> | Hỏi đúng câu hỏi | Thiết kế cẩn thận một Prompt hoàn hảo | Kiến thức lỗi thời, không thể tương tác với thế giới bên ngoài |
| <strong>RAG</strong> | Cung cấp tài liệu tham khảo | Truy xuất thông tin liên quan từ knowledge base trước khi trả lời | Phản hồi bị động, không thể thực hiện nhiệm vụ, phụ thuộc knowledge base |
| <strong>Agent</strong> | Trao khả năng hành động | Sử dụng công cụ, hoàn thành nhiệm vụ thông qua vòng lặp "suy nghĩ - hành động" | Phức tạp, không ổn định, chi phí cao |
| <strong>Context Engineering</strong> | Tạo ra input hoàn hảo | Thu thập, chọn lọc, nén, định dạng mang tính hệ thống tất cả thông tin, cung cấp context tối ưu cho mô hình | Là một phương pháp luận/bộ môn, không phải hệ thống cụ thể, triển khai phức tạp |

Nói đơn giản, chúng là sự tiến hóa về khả năng:
*   <strong>Prompt Engineering</strong> là một <strong>người đối thoại</strong>.
*   <strong>RAG</strong> là một <strong>người đối thoại</strong> mang theo cuốn sách để tra cứu.
*   <strong>Agent</strong> là một <strong>trợ lý</strong> có thể gọi điện thoại, lên mạng tra tài liệu, giúp bạn đặt vé.
*   <strong>Context Engineering</strong> là <strong>tổng tham mưu</strong> đứng sau trợ lý này, chịu trách nhiệm thu thập và sắp xếp trước mọi thông tin tình báo, đảm bảo trợ lý có thể đưa ra quyết định sáng suốt nhất.



## Tại sao xuất hiện Context Engineer?

![alt text](./images/Extra02-figures/image-3.png)


Khi LLM ngày càng giỏi hơn về mặt suy luận và gọi công cụ, mọi người ngày càng quan tâm nhiều hơn đến Agent. Agent đan xen việc gọi LLM và gọi công cụ, thường được dùng cho các tác vụ chạy trong thời gian dài. Agent sử dụng phản hồi từ công cụ để quyết định bước tiếp theo.


Tuy nhiên, các tác vụ chạy trong thời gian dài và phản hồi tích lũy từ việc gọi công cụ đồng nghĩa với việc Agent thường sử dụng một lượng lớn token. Điều này có thể dẫn đến nhiều vấn đề: có thể vượt quá kích thước context window, tăng chi phí/độ trễ hoặc làm giảm hiệu năng của Agent.

Khi context window ngày càng dài hơn, chúng ta ban đầu nghĩ rằng chỉ cần "ném toàn bộ lịch sử hội thoại và tài liệu vào mô hình" là có thể giải quyết vấn đề memory. Nhưng thực nghiệm cho thấy, thực tế phức tạp hơn nhiều so với tưởng tượng. Khi độ dài context tăng lên, mô hình ngày càng khó duy trì tính chính xác và nhất quán của thông tin, biểu hiện giống như "<strong>bộ nhớ bị mục rữa</strong>".

![alt text](./images/Extra02-figures/image-4.png)

Những hiện tượng này được gọi là Context Rot trong nghiên cứu của Chroma — tức là hiệu năng của mô hình bị "ăn mòn" trong ngữ cảnh dài. Đây chính là nguyên nhân gốc rễ khiến vai trò Context Engineer ra đời: cần có người để chống lại và khắc phục hiện tượng "ngữ cảnh bị mục rữa" này, thông qua việc cắt tỉa, nén, tái cấu trúc và retrieval-augmentation, giúp mô hình duy trì hiệu suất đáng tin cậy trong tài nguyên attention có hạn.


## Thách thức của Context

Thách thức của context chủ yếu tồn tại ở [bốn khía cạnh](https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html?ref=blog.langchain.com), lần lượt được mô tả là:

- Context Poisoning (Ô nhiễm context) - khi hallucination lọt vào context
- Context Distraction (Phân tán context) - khi context lấn át dữ liệu huấn luyện
- Context Confusion (Nhầm lẫn context) - khi context thừa ảnh hưởng đến phản hồi
- Context Clash (Xung đột context) - khi các phần của context không nhất quán với nhau







### Context Poisoning: When a Hallucination Makes It into the Context

Context Poisoning (ô nhiễm context) đề cập đến việc hallucination (tức thông tin sai lệch hoặc hư cấu do mô hình sinh ra) hoặc lỗi khác lọt vào context window, và bị tham chiếu lặp đi lặp lại, từ đó nhúng thông tin sai lệch vào, khiến hiệu năng của agent đi chệch hướng. Tình huống này sẽ "đầu độc" các phần then chốt, như mục tiêu hoặc tóm tắt, khiến mô hình chấp nhất vào các mục tiêu bất khả thi hoặc không liên quan, dẫn đến hành vi lặp lại, vô nghĩa.

### Context Distraction: When the Context Overwhelms the Training

Context Distraction (phân tán context) xảy ra khi context tăng trưởng quá dài (ví dụ vượt quá 100.000 token), khiến mô hình phụ thuộc quá mức vào chi tiết lịch sử, mà bỏ qua kiến thức pretrain của nó hoặc khả năng sinh ra giải pháp mới mẻ. Điều này sẽ dẫn đến hành động lặp đi lặp lại thay vì giải quyết vấn đề một cách sáng tạo, và hiệu năng đã suy giảm trước cả khi context window đầy.


Khi đối mặt với input hàng trăm nghìn token, mô hình không thể ghi nhớ đều đặn mọi thông tin giống như ổ cứng. Thực nghiệm phát hiện, input tinh gọn (chỉ vài trăm token) lại cho hiệu năng tốt hơn so với input đầy đủ (hàng trăm nghìn token). Kết quả nghiên cứu cho thấy, hiệu năng của mô hình trên phiên bản tinh gọn vượt trội đáng kể so với phiên bản đầy đủ. Điều này cho thấy khi input quá dài, nhiễu quá nhiều, dù là mô hình tiên tiến nhất cũng khó nắm bắt thông tin then chốt.

### Context Confusion: When Superfluous Context Influences the Response

Context Confusion (nhầm lẫn context) chỉ việc thông tin không liên quan hoặc dư thừa (như định nghĩa tool trùng lặp) được đưa vào context, buộc mô hình phải cân nhắc nó, từ đó tạo ra phản hồi không tối ưu. Ngay cả khi nội dung thêm vào vô hại, nó cũng làm loãng sự tập trung và giảm chất lượng.
Trong hội thoại và tài liệu thực tế, thường tồn tại "nhiễu" có ngữ nghĩa tương tự nhưng không liên quan. Với context ngắn, mô hình có thể phân biệt được, nhưng với context dài thì dễ bị đánh lừa hơn. Điều này đòi hỏi phải có người thực hiện việc chọn lọc và khử nhiễu cho context, để mô hình tập trung vào thông tin thực sự liên quan. Trong context dài, mô hình không chỉ cần tìm ra thông tin liên quan, mà còn phải phân biệt được "cái nào mới là needle đúng, cái nào chỉ là yếu tố gây nhiễu".

### Context Clash: When Parts of the Context Disagree

Context Clash (xung đột context) là dạng nghiêm trọng hơn của sự nhầm lẫn, chỉ việc thông tin trong context mâu thuẫn lẫn nhau (như tool mới hoặc sự thật mới mâu thuẫn với nội dung hiện có), từ đó phá vỡ quá trình suy luận, thường do mô hình bị khóa chặt vào giả định ban đầu. Điều này gây phá hoại nhiều hơn so với việc chỉ đơn thuần không liên quan: "This is a more problematic version of Context Confusion: the bad context here isn't irrelevant, it directly conflicts with other information in the prompt." Trong tương tác nhiều bước, lỗi ban đầu sẽ lan truyền, mô hình phụ thuộc vào tiền đề có khiếm khuyết.


Thiếu độ tin cậy kiểu "máy tính"
Chúng ta mong muốn LLM có output chất lượng nhất quán, nhưng ngay cả tác vụ sao chép đơn giản nhất, mô hình cũng sẽ mắc lỗi khi input dài. Nó không phải là bộ xử lý ký hiệu từng chữ từng vị trí, mà là bộ sinh ngôn ngữ dựa trên xác suất. Vì vậy không thể kỳ vọng nó xử lý context dài một cách chính xác như database hay máy tính, mà phải nhờ vào thiết kế có cấu trúc để bù đắp.




Do đó, quản lý context window hiệu quả và context engineering là không thể thiếu.







## Chiến lược Context Engineering

Phần trước đã đề cập context phải đối mặt với nhiều thách thức như vậy, vậy làm thế nào để vượt qua chúng? Điều này cần dựa vào context engineering. Trong đó, chiến lược của context engineering chủ yếu chia thành bốn loại: Write (ghi/lưu trữ), Select (chọn lọc), Compress (nén) và Isolate (cách ly).

![alt text](./images/Extra02-figures/image-6.png)

### Write Context (Ghi context)

<strong>Write context</strong> nghĩa là lưu trữ nó bên ngoài context window để giúp Agent thực hiện nhiệm vụ.
Chủ yếu chia thành hai loại:
- <strong>Bảng nháp tạm thời (Scratchpad)</strong>
Một không gian làm việc tạm thời, ghi lại quá trình suy luận trung gian của mô hình, làm cho quá trình tư duy trở nên có thể quan sát được. Ghi chú thông qua "bảng nháp tạm thời" là một phương pháp lưu giữ thông tin lâu dài khi Agent thực hiện nhiệm vụ. Ý tưởng là lưu thông tin bên ngoài context window, để Agent có thể sử dụng.
- <strong>Memory (Bộ nhớ)</strong>
Agent kết hợp context mới phát sinh (new context) với các memory sẵn có (existing memories), sau khi xử lý sẽ viết thành memory được cập nhật (updated memory)

![alt text](./images/Extra02-figures/image-8.png)



### Select Context (Chọn lọc context)

Khi lượng thông tin ngày càng lớn, cách chọn lọc còn quan trọng hơn cách lưu trữ. Select context chính là ở mỗi lần gọi mô hình, chọn ra phần thực sự liên quan từ tất cả nguồn thông tin sẵn có để đưa vào context window.

Các loại context cụ thể có thể chọn lọc bao gồm:

- <strong>Scratchpad (Bảng nháp tạm thời)</strong>: tức bảng nháp tạm thời đã đề cập ở trên, đóng vai trò không gian "bộ nhớ làm việc" của mô hình, dùng để ghi lại quá trình suy luận, kết quả trung gian và các bước tư duy. Trong nhiệm vụ nhiều bước, mô hình có thể ghi trạng thái suy luận hiện tại, sub-task đã hoàn thành, vấn đề đang chờ xử lý... vào scratchpad, tiện cho việc tham khảo và điều chỉnh chiến lược ở các bước sau.

- <strong>Memory (Bộ nhớ)</strong>: bao gồm hai tầng short-term memory và long-term memory. Short-term memory lưu lịch sử hội thoại và thông tin context trong phiên hiện tại, đảm bảo tính liên tục của hội thoại; long-term memory thì lưu trữ thông tin bền vững xuyên suốt nhiều phiên như sở thích người dùng, pattern tương tác lịch sử, cài đặt cá nhân hóa, giúp mô hình cung cấp trải nghiệm dịch vụ cá nhân hóa và nhất quán hơn.

- <strong>Tools (Công cụ)</strong>: trong hệ thống Agent, bản thân tool cũng là một loại context. Khi mô hình gọi API, plugin hoặc function bên ngoài, nó phải hiểu mô tả của tool (bao gồm giải thích chức năng, yêu cầu tham số, định dạng trả về...), và chọn đúng tool trong tình huống phù hợp. Kết quả phản hồi sau khi gọi tool cũng sẽ trở thành context mới, chỉ dẫn quyết định của mô hình ở bước tiếp theo. Tính khả dụng, trạng thái thực thi, lịch sử gọi của tool đều là thông tin context quan trọng.

- <strong>Knowledge (Tri thức)</strong>: chủ yếu chỉ knowledge base bên ngoài trong RAG (Retrieval-Augmented Generation). Bao gồm dữ liệu có cấu trúc (như bảng database), tài liệu phi cấu trúc (như tài liệu kỹ thuật, hướng dẫn sản phẩm), kết quả semantic retrieval từ vector database... Những tri thức bên ngoài này bù đắp cho hạn chế về tính thời sự và độ bao phủ kiến thức của dữ liệu huấn luyện mô hình, thông qua việc truy xuất động thông tin liên quan để tăng cường độ chính xác và chuyên nghiệp của câu trả lời mô hình.

### Compress Context (Nén context)




![alt text](./images/Extra02-figures/image-9.png)


Compress context liên quan đến việc chỉ giữ lại các token cần thiết để thực hiện nhiệm vụ, thông qua giảm thông tin dư thừa để tối ưu hóa hiệu quả sử dụng context window.

#### Tóm tắt context

<strong>Tóm tắt hội thoại:</strong>
Trong tương tác nhiều vòng kéo dài, giữ lại toàn bộ lịch sử hội thoại sẽ nhanh chóng tiêu tốn context window. Thông qua kỹ thuật tóm tắt hội thoại, có thể nén các vòng hội thoại trước đó thành dạng tóm tắt ngắn gọn, giữ lại thông tin then chốt (như sở thích người dùng, quyết định quan trọng, vấn đề chưa giải quyết...), đồng thời loại bỏ phần chào hỏi và nội dung lặp lại dư thừa. Như vậy vừa có thể duy trì tính liên tục của hội thoại, vừa để lại đủ không gian cho tương tác mới.

<strong>Tóm tắt tool:</strong>
Việc gọi tool thường trả về lượng lớn dữ liệu thô (như toàn bộ phản hồi API, kết quả truy vấn database...). Thông qua tóm tắt tool, có thể trích xuất và giữ lại các trường kết quả liên quan nhất, lọc bỏ metadata, thông tin debug... không cần thiết. Ví dụ, weather API có thể trả về các tham số khí tượng chi tiết, nhưng sau khi tóm tắt chỉ giữ lại thông tin cốt lõi như nhiệt độ, tình trạng thời tiết, giảm đáng kể tiêu hao token.

#### Cắt tỉa context

<strong>Cắt tỉa dựa trên quy tắc:</strong>
Có thể dùng heuristic hard-coded để chủ động xóa context lỗi thời hoặc ưu tiên thấp. Các chiến lược phổ biến bao gồm:
- Xóa tin nhắn cũ hơn khỏi lịch sử hội thoại, giữ lại N vòng hội thoại gần nhất
- Xóa bản ghi sub-task đã hoàn thành, chỉ giữ lại thông tin liên quan đến nhiệm vụ hiện tại
- Xóa dữ liệu tạm thời đã hết hạn hoặc kết quả gọi tool đã mất hiệu lực

<strong>Cắt tỉa thông minh:</strong>
Phương pháp nâng cao hơn có thể dựa trên điểm số liên quan (relevance score) để chọn lọc động giữ lại đoạn context nào. Thông qua tính toán độ tương đồng ngữ nghĩa hoặc chấm điểm tầm quan trọng, ưu tiên giữ lại thông tin liên quan nhất đến nhiệm vụ hiện tại, tự động loại bỏ nội dung lịch sử có độ liên quan thấp.


### Isolate Context (Cách ly context)

Isolate context liên quan đến việc tách context ra để giúp Agent thực hiện nhiệm vụ.

#### Kiến trúc đa Agent

![alt text](./images/Extra02-figures/image-10.png)

<strong>Tách biệt mối quan tâm (Separation of Concerns):</strong>
Tách nhiệm vụ lớn phức tạp thành nhiều sub-task độc lập, mỗi sub-task do một Agent chuyên trách đảm nhận. Thiết kế này tuân theo nguyên tắc single responsibility, giúp mỗi Agent tập trung vào một lĩnh vực cụ thể, nâng cao khả năng bảo trì và mở rộng của toàn hệ thống.

<strong>Đặc tính cách ly của Agent:</strong>
Mỗi sub-agent sở hữu tài nguyên và cấu hình độc lập:
- <strong>Bộ công cụ chuyên dụng</strong>: mỗi Agent chỉ có thể truy cập tool cụ thể cần thiết để hoàn thành nhiệm vụ của nó, tránh khó khăn lựa chọn do tool quá nhiều
- <strong>System instruction độc lập</strong>: system prompt được tùy chỉnh cho nhiệm vụ cụ thể, làm rõ định vị vai trò và quy tắc hành vi của Agent
- <strong>Context window cách ly</strong>: mỗi Agent duy trì không gian context riêng, không ảnh hưởng lẫn nhau, tránh thông tin không liên quan làm ô nhiễm

<strong>Cơ chế phối hợp giữa các Agent:</strong>
Nhiều Agent giao tiếp và truyền dữ liệu với nhau thông qua interface rõ ràng, Agent điều phối chính hoặc lớp routing chịu trách nhiệm phân bổ nhiệm vụ và tổng hợp kết quả, hình thành workflow phối hợp.

#### Cách ly môi trường thực thi

![alt text](./images/Extra02-figures/image-11.png)

<strong>Tách biệt context và thực thi:</strong>
Tách môi trường thực thi code ra khỏi context window của LLM, LLM không cần tiếp xúc trực tiếp với toàn bộ dữ liệu output thô của mọi tool.

<strong>Thiết kế lớp xử lý:</strong>
Thêm một lớp xử lý giữa việc thực thi tool và LLM:
- Tool thực thi trong môi trường sandbox độc lập, tạo ra output thô
- Lớp xử lý lọc, chuyển đổi và tóm tắt kết quả thô
- Chỉ truyền thông tin then chốt đã được tinh lọc vào context của LLM

Cách cách ly này vừa nâng cao tính an toàn, vừa giảm tiêu hao token, giúp LLM có thể tập trung vào quyết định cấp cao thay vì xử lý chi tiết cấp thấp.




### Tổng kết

Bốn hành động của context engineering — Write, Select, Compress, Isolate — không phải là các kỹ thuật rời rạc, mà là một bộ phương pháp có hệ thống.
Chúng lần lượt giải quyết các vấn đề mất thông tin, dư thừa thông tin, quá tải thông tin và xung đột thông tin.
Khi bốn chiến lược này được thực hiện một cách có hệ thống, Agent mới có thể vận hành ổn định trong môi trường phức tạp.


## Triển khai Context Engineering


Sử dụng LangSmith và LangGraph để thực hiện context engineering, phần nội dung cụ thể này có thể tham khảo Chương 9.



## Tổng kết và suy ngẫm: RAG is Dead?

![alt text](./images/Extra02-figures/image.png)


Jeff chủ yếu phê phán việc RAG truyền thống ghép ba khái niệm khác nhau "Retrieval (truy xuất), Augmented (tăng cường), Generation (sinh)" vào với nhau một cách gượng ép, dẫn đến sự lộn xộn về mặt khái niệm và sự mơ hồ trong thực hành. Nhìn lại RAG từ góc độ context engineering, có thể tách nó thành các bước rõ ràng hơn:

<strong>RAG truyền thống so với góc nhìn Context Engineering (Advanced RAG):</strong>

| Giai đoạn | RAG truyền thống | Phương pháp Context Engineering |
|------|---------|----------------|
| <strong>Retrieval</strong> | Tìm kiếm độ tương đồng vector đơn giản | Hybrid retrieval: kết hợp vector retrieval, khớp từ khóa, re-ranking và nhiều chiến lược khác |
| <strong>Filtering</strong> | Thường thiếu hoặc sơ sài | Lọc thông minh: loại bỏ nội dung dư thừa, lỗi thời hoặc không liên quan đến nhiệm vụ |
| <strong>Ranking</strong> | Dựa trên một điểm số tương đồng duy nhất | Xếp hạng đa chiều: cân nhắc độ liên quan, độ mới, độ tin cậy..., ưu tiên đưa thông tin quan trọng nhất vào |
| <strong>Evaluation</strong> | Thiếu đánh giá có hệ thống | Xây dựng golden dataset, định lượng đánh giá chất lượng retrieval, độ chính xác câu trả lời và hiệu quả sử dụng context |

<strong>Cải tiến cốt lõi:</strong>
- <strong>Đa dạng hóa chiến lược retrieval</strong>: không còn phụ thuộc vào vector retrieval đơn lẻ, mà kết hợp sử dụng dense retrieval, sparse retrieval, semantic re-ranking... tùy theo đặc điểm nhiệm vụ
- <strong>Ưu tiên chất lượng context</strong>: nhấn mạnh rằng những gì đưa vào LLM không phải là "càng nhiều càng tốt", mà là "càng chính xác càng tốt", thông qua lọc và xếp hạng để đảm bảo chất lượng cao của context
- <strong>Tối ưu hóa vòng lặp khép kín</strong>: thông qua evaluation dataset liên tục lặp lại tối ưu chiến lược retrieval, quy tắc lọc và thuật toán xếp hạng, hình thành quy trình kỹ thuật có thể đo lường, có thể cải thiện

Góc nhìn này biến RAG từ một quy trình hộp đen thành một vấn đề context engineering có thể tách rời, có thể tối ưu hóa, khiến nó có tính khả thi và khả năng mở rộng cao hơn.

Vì vậy, context engineering vừa là một thực hành kỹ thuật có hệ thống, vừa là một nghệ thuật đòi hỏi sự cân nhắc đánh đổi. Nó yêu cầu chúng ta phán đoán chính xác 4 câu hỏi sau giữa biển thông tin khổng lồ:

- <strong>Write (Ghi)</strong> —— Thông tin nào nên được đưa vào context?
- <strong>Select (Chọn)</strong> —— Nội dung nào liên quan nhất và cần thiết?
- <strong>Compress (Nén)</strong> —— Nội dung nào có thể tóm tắt hoặc đơn giản hóa?
- <strong>Isolate (Cách ly)</strong> —— Nội dung nào cần tách ra không gian độc lập?

Chỉ khi hiểu rõ những câu hỏi này, mới có thể thực hiện context engineering hiệu quả, đạt được sự kết hợp hoàn hảo giữa nghệ thuật và kỹ thuật.

![alt text](./images/Extra02-figures/image-12.png)


## Tài liệu tham khảo


沧海九粟. 上下文工程：优化 Agent 效能的关键技术[EB/OL]. (2025-07-10)[2025-10-21]. https://www.bilibili.com/video/BV1w3GNzeEHb/?spm_id_from=333.1387.upload.video_card.click&vd_source=0f47ed6b43bae0b240e774a8fd72e3e4


Drew Breunig. How Long Contexts Fail[EB/OL]. (2025-06-22)[2025-10-21]. https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html?ref=blog.langchain.com


Latent.Space, Jeff Huber, Swyx. RAG is Dead, Context Engineering is King[EB/OL]. (2025-08-19)[2025-10-21]. https://www.latent.space/p/chroma

万字拆解. RAG已死吗？上下文工程（context engineer）为何为王？[EB/OL]. (2025-09-03)[2025-10-21]. https://www.woshipm.com/ai/6264065.html
</content>
