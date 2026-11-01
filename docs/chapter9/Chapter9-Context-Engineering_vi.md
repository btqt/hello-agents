# Chương 9 Context Engineering

Trong các chương trước, chúng ta đã giới thiệu hệ thống memory và RAG cho Agent. Tuy nhiên, để Agent có thể "suy nghĩ" và "hành động" ổn định trong các tình huống phức tạp thực tế, chỉ có memory và retrieval là chưa đủ — chúng ta cần một phương pháp kỹ thuật (engineering) để liên tục, có hệ thống xây dựng "context" phù hợp cho model. Đây chính là chủ đề của chương này: Context Engineering. Nó quan tâm đến vấn đề "trước mỗi lần gọi model, làm thế nào để lắp ráp và tối ưu hóa context đầu vào theo cách có thể tái sử dụng, đo lường và tiến hóa được", từ đó nâng cao độ chính xác, độ bền vững (robustness) và hiệu suất<sup>[1][2]</sup>.

Để người đọc có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp gói Python có thể cài đặt trực tiếp. Bạn có thể cài đặt phiên bản tương ứng của chương này bằng lệnh sau:

```bash
pip install "hello-agents[all]==0.2.8"
```

Chương này chủ yếu giới thiệu các khái niệm cốt lõi và thực hành của Context Engineering, đồng thời bổ sung một context builder và hai công cụ đi kèm trong framework HelloAgents:

- **ContextBuilder** (`hello_agents/context/builder.py`): Context builder, triển khai pipeline GSSC (Gather-Select-Structure-Compress), cung cấp giao diện quản lý context thống nhất
- **NoteTool** (`hello_agents/tools/builtin/note_tool.py`): Công cụ ghi chú có cấu trúc, hỗ trợ Agent quản lý memory bền vững (persistent)
- **TerminalTool** (`hello_agents/tools/builtin/terminal_tool.py`): Công cụ terminal, hỗ trợ Agent thao tác hệ thống file và truy xuất context tức thời

Các thành phần này cùng nhau tạo thành một giải pháp Context Engineering hoàn chỉnh, là chìa khóa để thực hiện quản lý tác vụ dài hạn (long-horizon task) và tìm kiếm kiểu Agent (agentic search), sẽ được giới thiệu chi tiết trong các chương sau.

Ngoài việc cài đặt framework, bạn còn cần cấu hình API của LLM trong `.env`. Ví dụ trong chương này chủ yếu sử dụng mô hình ngôn ngữ lớn để quản lý context và ra quyết định thông minh.

Sau khi hoàn tất cấu hình, hãy bắt đầu hành trình học tập của chương này!



## 9.1 Context Engineering là gì

Sau nhiều năm Prompt Engineering trở thành tâm điểm của AI ứng dụng, một thuật ngữ mới bắt đầu bước lên sân khấu: <strong>Context Engineering</strong>. Ngày nay, việc xây dựng hệ thống bằng language model không còn chỉ là tìm đúng cú pháp và cách diễn đạt trong prompt, mà phải trả lời một câu hỏi vĩ mô hơn: <strong>cấu hình context như thế nào thì có khả năng cao nhất khiến model tạo ra hành vi mà chúng ta mong muốn?</strong>

Cái gọi là "context" là tập hợp các tokens được đưa vào khi lấy mẫu (sampling) từ một Large Language Model (LLM). Vấn đề kỹ thuật đặt ra là, dưới các ràng buộc vốn có của LLM, <strong>tối ưu hóa hiệu dụng (utility) của các tokens này</strong>, nhằm đạt được kết quả mong muốn một cách ổn định. Để điều khiển LLM hiệu quả, thường cần phải "suy nghĩ trong context" — nghĩa là: trong mỗi lần gọi, cần xem xét toàn bộ trạng thái mà LLM có thể nhìn thấy, và dự đoán hành vi mà trạng thái đó có thể gây ra.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-1.webp" alt="" width="85%"/>
  <p>Hình 9.1 Prompt engineering vs Context engineering</p>
</div>

Phần này sẽ bàn về Context Engineering đang nổi lên, và đưa ra một mô hình tư duy tinh gọn để xây dựng Agent <strong>có thể điều khiển, hiệu quả</strong>.

<strong>Context Engineering vs. Prompt Engineering</strong>

Như hình 9.1 cho thấy, trong góc nhìn của các nhà cung cấp model tiên tiến hiện nay, Context Engineering là sự tiến hóa tự nhiên của Prompt Engineering. Prompt Engineering quan tâm đến việc viết và tổ chức hướng dẫn cho LLM như thế nào để đạt kết quả tốt hơn (ví dụ cách viết system prompt và chiến lược cấu trúc hóa); còn Context Engineering là <strong>trong giai đoạn suy luận (inference), làm thế nào để hoạch định và duy trì "tập hợp thông tin tối ưu (tokens)"</strong>, trong đó không chỉ bao gồm prompt mà còn bao gồm tất cả thông tin khác sẽ đi vào context window.

Trong giai đoạn đầu của kỹ thuật LLM, prompt thường là công việc chính, vì hầu hết các use case (ngoại trừ trò chuyện hàng ngày) đều cần tối ưu hóa prompt tinh chỉnh cho phân loại hoặc sinh văn bản đơn lượt (single-turn). Đúng như tên gọi, cốt lõi của Prompt Engineering là "làm thế nào để viết prompt hiệu quả", đặc biệt là system prompt. Tuy nhiên, khi chúng ta bắt đầu xây dựng về mặt kỹ thuật các Agent mạnh mẽ hơn, hoạt động trong phạm vi thời gian dài hơn, qua nhiều vòng suy luận, chúng ta cần chiến lược có thể quản lý <strong>toàn bộ trạng thái context</strong> — bao gồm system instructions, tools, MCP (Model Context Protocol), external data, lịch sử tin nhắn, v.v.

Một Agent chạy theo vòng lặp sẽ liên tục tạo ra dữ liệu có thể liên quan đến vòng suy luận tiếp theo, những thông tin này phải được <strong>tinh lọc theo chu kỳ</strong>. Do đó, "nghệ thuật và kỹ thuật" của Context Engineering nằm ở việc, từ "vũ trụ thông tin ứng viên" liên tục mở rộng, <strong>sàng lọc ra những nội dung nào nên đi vào context window hữu hạn</strong>.

## 9.2 Tại sao Context Engineering lại quan trọng

Mặc dù tốc độ của model ngày càng nhanh, quy mô dữ liệu có thể xử lý ngày càng lớn, nhưng chúng ta quan sát thấy: LLM cũng giống như con người, đến một mức độ nhất định sẽ "lơ đãng" hoặc "rối loạn". Các benchmark kiểu needle-in-a-haystack (tìm kim trong đống rơm) đã tiết lộ một hiện tượng: <strong>context rot (sự suy thoái của context)</strong> — khi số lượng tokens trong context window tăng lên, khả năng của model trong việc nhớ lại chính xác thông tin từ context lại giảm xuống.

Đường cong suy giảm của các model khác nhau có thể mượt hơn hoặc dốc hơn, nhưng đặc điểm này gần như xuất hiện ở tất cả các model. Do đó, <strong>context phải được coi là một loại tài nguyên hữu hạn, và có lợi ích biên giảm dần (diminishing returns)</strong>. Giống như con người có dung lượng working memory hữu hạn, LLM cũng có một "ngân sách chú ý" (attention budget). Mỗi token được thêm vào đều tiêu tốn một phần ngân sách này, vì vậy chúng ta càng cần thận trọng hơn trong việc chọn lọc những tokens nào nên được cung cấp cho LLM.

Sự khan hiếm này không phải ngẫu nhiên, mà bắt nguồn từ ràng buộc kiến trúc của LLM. Transformer cho phép mỗi token thiết lập liên kết với <strong>tất cả</strong> các token trong context, về lý thuyết hình thành mối quan hệ attention từng cặp ở cấp độ \(n^2\). Khi độ dài context tăng lên, khả năng mô hình hóa các mối quan hệ từng cặp này của model sẽ bị "kéo mỏng", từ đó tự nhiên tạo ra sự căng thẳng giữa "quy mô context" và "mức độ tập trung chú ý". Ngoài ra, các mẫu attention của model bắt nguồn từ phân bố dữ liệu huấn luyện — chuỗi ngắn thường phổ biến hơn chuỗi dài, do đó model có ít kinh nghiệm hơn và ít tham số chuyên dụng hơn đối với "sự phụ thuộc toàn context".

Các kỹ thuật như position encoding interpolation có thể cho phép model "thích nghi" với chuỗi dài hơn so với lúc huấn luyện trong giai đoạn suy luận, nhưng sẽ hy sinh một phần độ chính xác trong việc hiểu vị trí của token. Nhìn chung, các yếu tố này cùng nhau hình thành một <strong>gradient hiệu suất</strong>, chứ không phải sự sụp đổ kiểu "vách đá": model vẫn mạnh mẽ ở context dài, nhưng so với context ngắn, độ chính xác trong việc truy xuất thông tin và suy luận tầm xa sẽ giảm đi phần nào.

Dựa trên thực tế trên, <strong>Context Engineering có chủ đích</strong> trở thành điều tất yếu để xây dựng Agent bền vững.

### 9.2.1 "Giải phẫu học" của context hiệu quả

Dưới ràng buộc "ngân sách chú ý hữu hạn", mục tiêu của Context Engineering tốt là: <strong>sử dụng càng ít tokens càng tốt, nhưng mật độ tín hiệu cao, để tối đa hóa xác suất đạt được kết quả mong muốn</strong>. Khi triển khai thực tế, chúng tôi đề xuất xây dựng kỹ thuật xoay quanh các thành phần sau:

- <strong>System Prompt</strong>: Ngôn ngữ rõ ràng, trực tiếp, phân cấp thông tin ở mức "vừa đủ". Hai sai lầm cực đoan thường gặp:
  - Hard-code quá mức: viết logic if-else phức tạp, dễ vỡ vào prompt, chi phí bảo trì lâu dài cao và dễ hỏng.
  - Quá mơ hồ: chỉ đưa ra mục tiêu vĩ mô và hướng dẫn tổng quát, thiếu <strong>tín hiệu cụ thể</strong> về đầu ra mong muốn hoặc giả định sai "shared context".
  Đề xuất tổ chức prompt theo phân khu (ví dụ <background_information>, <instructions>, hướng dẫn tool, mô tả đầu ra, v.v.), dùng XML/Markdown để phân tách. Bất kể định dạng nào, mục tiêu theo đuổi là <strong>"tập hợp thông tin tối thiểu cần thiết" có thể vẽ nên đầy đủ hành vi mong muốn</strong> ("tối thiểu" không đồng nghĩa với "ngắn nhất"). Trước tiên hãy dùng model tốt nhất để chạy thử trên prompt tối thiểu, sau đó dựa trên các mẫu lỗi để bổ sung hướng dẫn và ví dụ rõ ràng.

- <strong>Tools (công cụ)</strong>: Tools định nghĩa hợp đồng (contract) giữa Agent và không gian thông tin/hành động, phải thúc đẩy hiệu quả: vừa phải trả về thông tin <strong>thân thiện với token</strong>, vừa phải khuyến khích hành vi Agent hiệu quả. Tools nên:
  - Trách nhiệm đơn nhất, ít chồng chéo lẫn nhau, ngữ nghĩa giao diện rõ ràng;
  - Bền vững trước lỗi (robust to errors);
  - Mô tả tham số đầu vào rõ ràng, không mơ hồ, phát huy tối đa khả năng diễn đạt và suy luận mà model giỏi.
  Mẫu lỗi thường gặp là "bộ công cụ cồng kềnh": ranh giới chức năng mơ hồ, khiến quyết định "chọn công cụ nào" trở nên mập mờ. <strong>Nếu ngay cả kỹ sư con người cũng không chắc dùng tool nào, đừng mong Agent làm tốt hơn</strong>. Việc sàng lọc kỹ lưỡng một "bộ công cụ khả thi tối thiểu (MVTS - Minimum Viable Tool Set)" thường có thể nâng cao đáng kể độ ổn định và khả năng bảo trì trong tương tác dài hạn.

- <strong>Ví dụ (Few-shot)</strong>: Luôn khuyến khích cung cấp ví dụ, nhưng không nên nhồi nhét "liệt kê tất cả các trường hợp biên" vào prompt. Hãy chọn lọc kỹ một nhóm ví dụ <strong>đa dạng và điển hình</strong>, trực tiếp vẽ nên "hành vi mong muốn". Đối với LLM, <strong>một ví dụ tốt đáng giá hơn ngàn lời nói</strong>.

Tư tưởng chỉ đạo tổng thể là: <strong>đủ thông tin nhưng gọn gàng</strong>. Như hình 9.2, là retrieval động khi đi vào runtime.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-2.webp" alt="" width="85%"/>
  <p>Hình 9.2 Calibrating the system prompt</p>
</div>


### 9.2.2 Context Retrieval và Agentic Search

Một định nghĩa ngắn gọn: <strong>Agent = LLM tự chủ gọi tool trong một vòng lặp</strong>. Khi khả năng của model nền tảng được tăng cường, mức độ tự chủ của Agent cũng có thể nâng cao: khả năng tự khám phá không gian vấn đề phức tạp và phục hồi từ lỗi tốt hơn.

Thực hành kỹ thuật đang chuyển dần từ "retrieval một lần trước khi suy luận (embedding retrieval)" sang "<strong>context tức thời (Just-in-time, JIT)</strong>". Cách tiếp cận sau không còn nạp sẵn toàn bộ dữ liệu liên quan, mà duy trì <strong>các tham chiếu nhẹ (lightweight references)</strong> (đường dẫn file, truy vấn lưu trữ, URL, v.v.), nạp dữ liệu cần thiết một cách động thông qua tool khi runtime. Điều này cho phép model viết các truy vấn có mục tiêu, cache các kết quả cần thiết, và sử dụng các lệnh như <code>head</code>/<code>tail</code> để phân tích dữ liệu khối lượng lớn — không cần nhồi toàn bộ dữ liệu vào context cùng lúc. Mô hình nhận thức của nó gần gũi hơn với con người: chúng ta không ghi nhớ thuộc lòng toàn bộ thông tin, mà sử dụng hệ thống file, hộp thư đến, bookmark v.v. làm chỉ mục bên ngoài để trích xuất theo nhu cầu.

Ngoài hiệu quả lưu trữ, <strong>metadata của tham chiếu</strong> bản thân nó cũng có thể giúp tinh chỉnh hành vi: cấp bậc thư mục, quy ước đặt tên, timestamp v.v. đều ngầm truyền đạt "mục đích và tính thời sự". Ví dụ, hàm ý ngữ nghĩa của <code>tests/test_utils.py</code> và <code>src/core/test_utils.py</code> là khác nhau.

Cho phép Agent tự chủ điều hướng và truy xuất còn có thể thực hiện <strong>tiết lộ dần dần (progressive disclosure)</strong>: mỗi bước tương tác đều tạo ra context mới, ngược lại hướng dẫn quyết định của bước tiếp theo — kích thước file gợi ý độ phức tạp, tên gọi gợi ý mục đích, timestamp gợi ý tính liên quan. Agent có thể xây dựng sự hiểu biết theo từng lớp, chỉ giữ lại "tập con cần thiết hiện tại" trong working memory, và bổ sung lưu trữ bền vững bằng cách "ghi chú", từ đó duy trì sự tập trung thay vì "bị kéo sập bởi lượng thông tin khổng lồ".

Điều cần cân nhắc là: khám phá tại runtime thường chậm hơn so với retrieval tính toán trước, và cần thiết kế kỹ thuật "có chủ kiến" để đảm bảo model có đúng tool và heuristic. Nếu thiếu hướng dẫn, Agent có thể sử dụng sai tool, đi vào ngõ cụt hoặc bỏ lỡ thông tin quan trọng, gây lãng phí context.

Trong không ít tình huống, <strong>chiến lược lai (hybrid)</strong> hiệu quả hơn: nạp trước một lượng nhỏ context "giá trị cao" để đảm bảo tốc độ, sau đó cho phép Agent tiếp tục tự chủ khám phá theo nhu cầu. Việc lựa chọn ranh giới phụ thuộc vào tính động của tác vụ và yêu cầu về tính thời sự. Về mặt kỹ thuật, có thể đặt trước các file dạng "mô tả quy ước dự án (như README/hướng dẫn)", đồng thời cung cấp các nguyên hàm (primitive) như <code>glob</code>, <code>grep</code>, để Agent có thể truy xuất tức thời các file cụ thể, từ đó né tránh chi phí chìm (sunk cost) của chỉ mục lỗi thời và cây cú pháp phức tạp.


### 9.2.3 Context Engineering cho tác vụ dài hạn (long-horizon task)

Các tác vụ dài hạn yêu cầu Agent vẫn giữ được tính nhất quán, tính thống nhất của context và định hướng mục tiêu trong chuỗi hành động dài vượt quá context window. Ví dụ như di chuyển codebase lớn, nghiên cứu có hệ thống kéo dài nhiều giờ. Việc kỳ vọng mở rộng vô hạn context window không thể giải quyết triệt để vấn đề "ô nhiễm context" và sự suy giảm độ liên quan, do đó cần các biện pháp kỹ thuật trực tiếp đối phó với những ràng buộc này: <strong>Compaction (nén tổng hợp)</strong>, <strong>Structured note-taking (ghi chú có cấu trúc)</strong> và <strong>Sub-agent architectures (kiến trúc sub-agent)</strong>.

- <strong>Compaction (nén tổng hợp)</strong>
  - Định nghĩa: khi cuộc hội thoại gần đạt giới hạn context, thực hiện tóm tắt độ trung thực cao (high-fidelity), và dùng bản tóm tắt đó khởi động lại một context window mới, để duy trì tính nhất quán dài hạn.
  - Thực hành: để model nén và giữ lại các quyết định kiến trúc, lỗi chưa giải quyết, chi tiết triển khai, loại bỏ output tool trùng lặp và nhiễu; window mới mang theo bản tóm tắt nén + một số ít artifact liên quan cao gần đây nhất (như "một số file đã truy cập gần đây").
  - Khuyến nghị điều chỉnh tham số: trước tiên tối ưu hóa <strong>recall (khả năng gọi lại)</strong> (đảm bảo không bỏ sót thông tin quan trọng), sau đó tối ưu hóa <strong>precision (độ chính xác)</strong> (loại bỏ nội dung dư thừa); một cách nén "chạm nhẹ" an toàn là dọn dẹp "các lần gọi tool và kết quả trong lịch sử sâu".

- <strong>Structured note-taking (ghi chú có cấu trúc)</strong>
  - Định nghĩa: còn gọi là "agentic memory". Agent ghi thông tin quan trọng vào <strong>bộ nhớ bền vững bên ngoài context</strong> theo tần suất cố định, và kéo lại khi cần ở các giai đoạn sau.
  - Giá trị: duy trì trạng thái bền vững và các mối quan hệ phụ thuộc với chi phí context cực thấp. Ví dụ duy trì danh sách TODO, file NOTES.md của dự án, chỉ mục các kết luận/phụ thuộc/điểm nghẽn quan trọng, vẫn giữ được tiến độ và tính nhất quán qua hàng chục lần gọi tool và nhiều lần reset context.
  - Giải thích: cũng hiệu quả trong các tình huống không phải coding (như các tác vụ chiến lược dài hạn, quản lý mục tiêu và đếm thống kê trong game/mô phỏng). Kết hợp với <code>MemoryTool</code> ở Chương 8, có thể dễ dàng triển khai bộ nhớ ngoài kiểu file/vector và truy xuất tại runtime.

- <strong>Sub-agent architectures (kiến trúc sub-agent)</strong>
  - Ý tưởng: Agent chính chịu trách nhiệm lập kế hoạch và tổng hợp ở cấp cao, nhiều sub-agent chuyên biệt tự đào sâu, gọi tool và khám phá trong "context window sạch" riêng, cuối cùng chỉ trả về <strong>bản tóm tắt cô đọng</strong> (thường khoảng 1.000–2.000 tokens).
  - Lợi ích: thực hiện tách biệt mối quan tâm (separation of concerns). Context tìm kiếm cồng kềnh được giữ lại bên trong sub-agent, agent chính tập trung vào tổng hợp và suy luận; phù hợp với các tác vụ nghiên cứu/phân tích phức tạp cần khám phá song song.
  - Kinh nghiệm: các hệ thống nghiên cứu đa Agent công khai cho thấy, mô hình này có lợi thế đáng kể so với baseline đơn Agent trong các tác vụ nghiên cứu phức tạp.

Việc lựa chọn phương pháp có thể tuân theo các quy tắc kinh nghiệm sau:

- <strong>Compaction</strong>: phù hợp với các tác vụ cần tính liên tục của hội thoại dài, nhấn mạnh việc "chuyển tiếp" context.
- <strong>Structured note-taking</strong>: phù hợp với phát triển và nghiên cứu lặp lại (iterative) có các cột mốc/thành quả theo giai đoạn.
- <strong>Sub-agent architectures</strong>: phù hợp với nghiên cứu và phân tích phức tạp, có thể hưởng lợi từ khám phá song song.

Ngay cả khi khả năng của model tiếp tục được cải thiện, "duy trì tính nhất quán và sự tập trung trong tương tác dài hạn" vẫn là thách thức cốt lõi để xây dựng Agent bền vững. Context Engineering thận trọng và có hệ thống sẽ luôn giữ giá trị then chốt của nó về lâu dài.

## 9.3 Thực hành trong Hello-Agents: ContextBuilder

Phần này sẽ giới thiệu chi tiết thực hành Context Engineering trong framework HelloAgents. Chúng ta sẽ đi từ động cơ thiết kế, cấu trúc dữ liệu cốt lõi, chi tiết triển khai cho đến case hoàn chỉnh, từng bước thể hiện cách xây dựng một hệ thống quản lý context cấp sản xuất (production-grade). Triết lý thiết kế của ContextBuilder là "đơn giản và hiệu quả", loại bỏ sự phức tạp không cần thiết, thống nhất sử dụng điểm số "relevance + recency" để lựa chọn, phù hợp với định hướng kỹ thuật về tính module hóa và khả năng bảo trì của Agent.

### 9.3.1 Động cơ và mục tiêu thiết kế

Trước khi xây dựng ContextBuilder, trước tiên chúng ta cần làm rõ mục tiêu thiết kế và giá trị cốt lõi của nó. Một hệ thống quản lý context tốt nên giải quyết các vấn đề then chốt sau:

1. <strong>Điểm vào thống nhất</strong>: trừu tượng hóa "Gather (thu thập) - Select (lựa chọn) - Structure (cấu trúc hóa) - Compress (nén)" thành một pipeline có thể tái sử dụng, giảm mã boilerplate lặp lại trong triển khai Agent. Thiết kế giao diện thống nhất này giúp developer không cần viết lại logic quản lý context trong mỗi Agent.

2. <strong>Hình thái ổn định</strong>: xuất ra template context có khung cố định, thuận tiện cho debug, A/B testing và đánh giá. Chúng tôi áp dụng cấu trúc template được tổ chức theo phân khu:
   - `[Role & Policies]`: xác định rõ vai trò và nguyên tắc hành vi của Agent
   - `[Task]`: tác vụ cụ thể cần hoàn thành hiện tại
   - `[State]`: trạng thái hiện tại và thông tin context của Agent
   - `[Evidence]`: thông tin bằng chứng được truy xuất từ knowledge base bên ngoài
   - `[Context]`: lịch sử hội thoại và memory liên quan
   - `[Output]`: định dạng và yêu cầu đầu ra mong muốn

3. <strong>Bảo vệ ngân sách (budget)</strong>: trong phạm vi token budget, cố gắng giữ lại thông tin giá trị cao, cung cấp chiến lược nén dự phòng cho context vượt giới hạn. Điều này đảm bảo rằng ngay cả trong các tình huống có lượng thông tin khổng lồ, hệ thống vẫn có thể hoạt động ổn định.

4. <strong>Quy tắc tối thiểu</strong>: không đưa vào các chiều phân loại như nguồn/ưu tiên, tránh làm tăng độ phức tạp. Thực tế cho thấy, cơ chế chấm điểm đơn giản dựa trên relevance và recency, trong hầu hết các tình huống đã đủ hiệu quả.

### 9.3.2 Cấu trúc dữ liệu cốt lõi

Việc triển khai ContextBuilder dựa trên hai cấu trúc dữ liệu cốt lõi, chúng định nghĩa cấu hình và đơn vị thông tin của hệ thống.

（1）ContextPacket: gói thông tin ứng viên

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any
from datetime import datetime

@dataclass
class ContextPacket:
    """Gói thông tin ứng viên

    Attributes:
        content: Nội dung thông tin
        timestamp: Dấu thời gian
        token_count: Số lượng token
        relevance_score: Điểm relevance (0.0-1.0)
        metadata: Metadata tùy chọn
    """
    content: str
    timestamp: datetime
    token_count: int
    relevance_score: float = 0.5
    metadata: Optional[Dict[str, Any]] = None

    def __post_init__(self):
        """Xử lý sau khởi tạo"""
        if self.metadata is None:
            self.metadata = {}
        # Đảm bảo relevance score nằm trong phạm vi hợp lệ
        self.relevance_score = max(0.0, min(1.0, self.relevance_score))
```

`ContextPacket` là đơn vị cơ bản của thông tin trong hệ thống. Mỗi thông tin ứng viên đều được đóng gói thành một ContextPacket, bao gồm các thuộc tính cốt lõi như nội dung, timestamp, số lượng token và relevance score. Cấu trúc dữ liệu thống nhất này đơn giản hóa logic lựa chọn và sắp xếp ở các bước sau.

（2）ContextConfig: quản lý cấu hình

```python
@dataclass
class ContextConfig:
    """Cấu hình xây dựng context

    Attributes:
        max_tokens: Số lượng token tối đa
        reserve_ratio: Tỷ lệ dự trữ cho system instructions (0.0-1.0)
        min_relevance: Ngưỡng relevance tối thiểu
        enable_compression: Có bật nén hay không
        recency_weight: Trọng số recency (0.0-1.0)
        relevance_weight: Trọng số relevance (0.0-1.0)
    """
    max_tokens: int = 3000
    reserve_ratio: float = 0.2
    min_relevance: float = 0.1
    enable_compression: bool = True
    recency_weight: float = 0.3
    relevance_weight: float = 0.7

    def __post_init__(self):
        """Xác thực tham số cấu hình"""
        assert 0.0 <= self.reserve_ratio <= 1.0, "reserve_ratio phải nằm trong phạm vi [0, 1]"
        assert 0.0 <= self.min_relevance <= 1.0, "min_relevance phải nằm trong phạm vi [0, 1]"
        assert abs(self.recency_weight + self.relevance_weight - 1.0) < 1e-6, \
            "recency_weight + relevance_weight phải bằng 1.0"
```

`ContextConfig` đóng gói tất cả các tham số có thể cấu hình, giúp hành vi hệ thống có thể điều chỉnh linh hoạt. Đặc biệt đáng chú ý là tham số `reserve_ratio`, nó đảm bảo rằng các thông tin quan trọng như system instructions luôn có đủ không gian, không bị các thông tin khác chiếm chỗ.

### 9.3.3 Giải thích chi tiết pipeline GSSC

Cốt lõi của ContextBuilder là pipeline GSSC (Gather-Select-Structure-Compress), nó phân rã quá trình xây dựng context thành bốn giai đoạn rõ ràng. Hãy cùng tìm hiểu sâu về chi tiết triển khai của từng giai đoạn.

（1）Gather: thu thập thông tin đa nguồn

Giai đoạn đầu tiên là thu thập thông tin ứng viên từ nhiều nguồn khác nhau. Mấu chốt của giai đoạn này nằm ở khả năng chịu lỗi (fault tolerance) và tính linh hoạt.

```python
def _gather(
    self,
    user_query: str,
    conversation_history: Optional[List[Message]] = None,
    system_instructions: Optional[str] = None,
    custom_packets: Optional[List[ContextPacket]] = None
) -> List[ContextPacket]:
    """Thu thập tất cả thông tin ứng viên

    Args:
        user_query: Truy vấn của người dùng
        conversation_history: Lịch sử hội thoại
        system_instructions: System instructions
        custom_packets: Gói thông tin tùy chỉnh

    Returns:
        List[ContextPacket]: Danh sách thông tin ứng viên
    """
    packets = []

    # 1. Thêm system instructions (ưu tiên cao nhất, không tham gia chấm điểm)
    if system_instructions:
        packets.append(ContextPacket(
            content=system_instructions,
            timestamp=datetime.now(),
            token_count=self._count_tokens(system_instructions),
            relevance_score=1.0,  # System instructions luôn được giữ lại
            metadata={"type": "system_instruction", "priority": "high"}
        ))

    # 2. Truy xuất memory liên quan từ hệ thống memory
    if self.memory_tool:
        try:
            memory_results = self.memory_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 10,
                "min_importance": 0.3
            })
            # Phân tích kết quả memory và chuyển thành ContextPacket
            memory_packets = self._parse_memory_results(memory_results, user_query)
            packets.extend(memory_packets)
        except Exception as e:
            print(f"[WARNING] Truy xuất memory thất bại: {e}")

    # 3. Truy xuất knowledge liên quan từ hệ thống RAG
    if self.rag_tool:
        try:
            rag_results = self.rag_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 5,
                "min_score": 0.3
            })
            # Phân tích kết quả RAG và chuyển thành ContextPacket
            rag_packets = self._parse_rag_results(rag_results, user_query)
            packets.extend(rag_packets)
        except Exception as e:
            print(f"[WARNING] Truy xuất RAG thất bại: {e}")

    # 4. Thêm lịch sử hội thoại (chỉ giữ N tin nhắn gần nhất)
    if conversation_history:
        recent_history = conversation_history[-5:]  # Mặc định giữ 5 tin nhắn gần nhất
        for msg in recent_history:
            packets.append(ContextPacket(
                content=f"{msg.role}: {msg.content}",
                timestamp=msg.timestamp if hasattr(msg, 'timestamp') else datetime.now(),
                token_count=self._count_tokens(msg.content),
                relevance_score=0.6,  # Relevance cơ bản của tin nhắn lịch sử
                metadata={"type": "conversation_history", "role": msg.role}
            ))

    # 5. Thêm gói thông tin tùy chỉnh
    if custom_packets:
        packets.extend(custom_packets)

    print(f"[ContextBuilder] Đã thu thập {len(packets)} gói thông tin ứng viên")
    return packets
```

Việc triển khai này thể hiện một số cân nhắc thiết kế quan trọng:

- <strong>Cơ chế chịu lỗi</strong>: mỗi lệnh gọi nguồn dữ liệu bên ngoài đều được bọc bởi try-except, đảm bảo lỗi của một nguồn đơn lẻ không ảnh hưởng đến toàn bộ quy trình
- <strong>Xử lý ưu tiên</strong>: system instructions được đánh dấu là ưu tiên cao, đảm bảo luôn được giữ lại
- <strong>Giới hạn lịch sử</strong>: lịch sử hội thoại chỉ giữ lại vài tin nhắn gần nhất, tránh context window bị thông tin lịch sử chiếm dụng

（2）Select: lựa chọn thông tin thông minh

Giai đoạn thứ hai là chấm điểm và lựa chọn thông tin ứng viên dựa trên relevance và recency. Đây là cốt lõi của toàn bộ pipeline, quyết định trực tiếp chất lượng của context cuối cùng.

```python
def _select(
    self,
    packets: List[ContextPacket],
    user_query: str,
    available_tokens: int
) -> List[ContextPacket]:
    """Lựa chọn các gói thông tin liên quan nhất

    Args:
        packets: Danh sách gói thông tin ứng viên
        user_query: Truy vấn của người dùng (để tính relevance)
        available_tokens: Số lượng token khả dụng

    Returns:
        List[ContextPacket]: Danh sách gói thông tin được chọn
    """
    # 1. Tách system instructions và các thông tin khác
    system_packets = [p for p in packets if p.metadata.get("type") == "system_instruction"]
    other_packets = [p for p in packets if p.metadata.get("type") != "system_instruction"]

    # 2. Tính số token chiếm dụng bởi system instructions
    system_tokens = sum(p.token_count for p in system_packets)
    remaining_tokens = available_tokens - system_tokens

    if remaining_tokens <= 0:
        print("[WARNING] System instructions đã chiếm hết toàn bộ token budget")
        return system_packets

    # 3. Tính điểm tổng hợp cho các thông tin khác
    scored_packets = []
    for packet in other_packets:
        # Tính relevance score (nếu chưa được tính)
        if packet.relevance_score == 0.5:  # Giá trị mặc định, cần tính lại
            relevance = self._calculate_relevance(packet.content, user_query)
            packet.relevance_score = relevance

        # Tính recency score
        recency = self._calculate_recency(packet.timestamp)

        # Điểm tổng hợp = trọng số relevance × relevance + trọng số recency × recency
        combined_score = (
            self.config.relevance_weight * packet.relevance_score +
            self.config.recency_weight * recency
        )

        # Lọc bỏ thông tin dưới ngưỡng relevance tối thiểu
        if packet.relevance_score >= self.config.min_relevance:
            scored_packets.append((combined_score, packet))

    # 4. Sắp xếp giảm dần theo điểm
    scored_packets.sort(key=lambda x: x[0], reverse=True)

    # 5. Lựa chọn tham lam (greedy): điền từ điểm cao đến thấp, cho đến khi đạt giới hạn token
    selected = system_packets.copy()
    current_tokens = system_tokens

    for score, packet in scored_packets:
        if current_tokens + packet.token_count <= available_tokens:
            selected.append(packet)
            current_tokens += packet.token_count
        else:
            # Token budget đã đầy, dừng lựa chọn
            break

    print(f"[ContextBuilder] Đã chọn {len(selected)} gói thông tin, tổng cộng {current_tokens} tokens")
    return selected

def _calculate_relevance(self, content: str, query: str) -> float:
    """Tính relevance giữa nội dung và truy vấn

    Sử dụng thuật toán overlap từ khóa đơn giản. Trong môi trường production, có thể thay bằng tính toán vector similarity.

    Args:
        content: Văn bản nội dung
        query: Văn bản truy vấn

    Returns:
        float: Relevance score (0.0-1.0)
    """
    # Tách từ (triển khai đơn giản, có thể dùng tokenizer phức tạp hơn)
    content_words = set(content.lower().split())
    query_words = set(query.lower().split())

    if not query_words:
        return 0.0

    # Độ tương đồng Jaccard
    intersection = content_words & query_words
    union = content_words | query_words

    return len(intersection) / len(union) if union else 0.0

def _calculate_recency(self, timestamp: datetime) -> float:
    """Tính điểm recency theo thời gian

    Sử dụng mô hình suy giảm theo hàm mũ (exponential decay), giữ điểm cao trong 24 giờ, sau đó giảm dần.

    Args:
        timestamp: Dấu thời gian của thông tin

    Returns:
        float: Recency score (0.0-1.0)
    """
    import math

    age_hours = (datetime.now() - timestamp).total_seconds() / 3600

    # Suy giảm hàm mũ: giữ điểm cao trong 24 giờ, sau đó giảm dần
    decay_factor = 0.1  # Hệ số suy giảm
    recency_score = math.exp(-decay_factor * age_hours / 24)

    return max(0.1, min(1.0, recency_score))  # Giới hạn trong phạm vi [0.1, 1.0]
```

Thuật toán cốt lõi của giai đoạn lựa chọn thể hiện một số cân nhắc kỹ thuật quan trọng:

- <strong>Cơ chế chấm điểm</strong>: áp dụng tổ hợp trọng số của relevance và recency, trọng số có thể cấu hình
- <strong>Thuật toán tham lam</strong>: điền từ điểm cao đến thấp, đảm bảo lựa chọn thông tin giá trị nhất trong ngân sách hữu hạn
- <strong>Cơ chế lọc</strong>: lọc bỏ thông tin chất lượng thấp thông qua tham số `min_relevance`

（3）Structure: cấu trúc hóa đầu ra

Giai đoạn thứ ba là tổ chức thông tin được chọn thành template context có cấu trúc.

```python
def _structure(self, selected_packets: List[ContextPacket], user_query: str) -> str:
    """Tổ chức các gói thông tin được chọn thành template context có cấu trúc

    Args:
        selected_packets: Danh sách gói thông tin được chọn
        user_query: Truy vấn của người dùng

    Returns:
        str: Chuỗi context có cấu trúc
    """
    # Nhóm theo loại
    system_instructions = []
    evidence = []
    context = []

    for packet in selected_packets:
        packet_type = packet.metadata.get("type", "general")

        if packet_type == "system_instruction":
            system_instructions.append(packet.content)
        elif packet_type in ["rag_result", "knowledge"]:
            evidence.append(packet.content)
        else:
            context.append(packet.content)

    # Xây dựng template có cấu trúc
    sections = []

    # [Role & Policies]
    if system_instructions:
        sections.append("[Role & Policies]\n" + "\n".join(system_instructions))

    # [Task]
    sections.append(f"[Task]\n{user_query}")

    # [Evidence]
    if evidence:
        sections.append("[Evidence]\n" + "\n---\n".join(evidence))

    # [Context]
    if context:
        sections.append("[Context]\n" + "\n".join(context))

    # [Output]
    sections.append("[Output]\nHãy dựa trên thông tin trên để cung cấp câu trả lời chính xác, có căn cứ.")

    return "\n\n".join(sections)
```

Giai đoạn cấu trúc hóa tổ chức các gói thông tin rời rạc thành các phân khu rõ ràng, thiết kế này có một số ưu điểm:

- <strong>Khả năng đọc hiểu</strong>: các phân khu rõ ràng giúp cả con người và model dễ hiểu cấu trúc context hơn
- <strong>Khả năng debug</strong>: việc định vị vấn đề dễ dàng hơn, có thể nhanh chóng xác định phân khu nào có vấn đề về thông tin
- <strong>Khả năng mở rộng</strong>: thêm nguồn thông tin mới chỉ cần tạo phân khu mới

（4）Compress: nén dự phòng

Giai đoạn thứ tư là xử lý nén đối với context vượt giới hạn.

```python
def _compress(self, context: str, max_tokens: int) -> str:
    """Nén context vượt giới hạn

    Args:
        context: Context gốc
        max_tokens: Giới hạn token tối đa

    Returns:
        str: Context sau khi nén
    """
    current_tokens = self._count_tokens(context)

    if current_tokens <= max_tokens:
        return context  # Không cần nén

    print(f"[ContextBuilder] Context vượt giới hạn ({current_tokens} > {max_tokens}), thực hiện nén")

    # Nén theo phân khu: giữ nguyên tính toàn vẹn của cấu trúc
    sections = context.split("\n\n")
    compressed_sections = []
    current_total = 0

    for section in sections:
        section_tokens = self._count_tokens(section)

        if current_total + section_tokens <= max_tokens:
            # Giữ nguyên toàn bộ
            compressed_sections.append(section)
            current_total += section_tokens
        else:
            # Giữ lại một phần
            remaining_tokens = max_tokens - current_total
            if remaining_tokens > 50:  # Giữ lại ít nhất 50 tokens
                # Cắt bớt đơn giản (môi trường production có thể dùng LLM để tóm tắt)
                truncated = self._truncate_text(section, remaining_tokens)
                compressed_sections.append(truncated + "\n[... nội dung đã được nén ...]")
            break

    compressed_context = "\n\n".join(compressed_sections)
    final_tokens = self._count_tokens(compressed_context)
    print(f"[ContextBuilder] Nén hoàn tất: {current_tokens} -> {final_tokens} tokens")

    return compressed_context

def _truncate_text(self, text: str, max_tokens: int) -> str:
    """Cắt văn bản xuống số lượng token chỉ định

    Args:
        text: Văn bản gốc
        max_tokens: Số lượng token tối đa

    Returns:
        str: Văn bản sau khi cắt
    """
    # Triển khai đơn giản: ước lượng theo tỷ lệ ký tự
    # Môi trường production nên sử dụng tokenizer chính xác
    char_per_token = len(text) / self._count_tokens(text) if self._count_tokens(text) > 0 else 4
    max_chars = int(max_tokens * char_per_token)

    return text[:max_chars]

def _count_tokens(self, text: str) -> int:
    """Ước lượng số lượng token của văn bản

    Args:
        text: Nội dung văn bản

    Returns:
        int: Số lượng token
    """
    # Ước lượng đơn giản: 1 ký tự tiếng Trung ≈ 1 token, 1 từ tiếng Anh ≈ 1.3 token
    # Môi trường production nên sử dụng tokenizer thực tế
    chinese_chars = sum(1 for ch in text if '\u4e00' <= ch <= '\u9fff')
    english_words = len([w for w in text.split() if w])

    return int(chinese_chars + english_words * 1.3)
```

Thiết kế của giai đoạn nén thể hiện nguyên tắc "giữ nguyên tính toàn vẹn của cấu trúc", ngay cả khi token budget căng thẳng, vẫn cố gắng giữ lại thông tin then chốt của mỗi phân khu.

### 9.3.4 Ví dụ sử dụng hoàn chỉnh

Bây giờ hãy thông qua một ví dụ hoàn chỉnh, thể hiện cách sử dụng ContextBuilder trong dự án thực tế.

（1）Sử dụng cơ bản

```python
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool
from hello_agents.core.message import Message
from datetime import datetime

# 1. Khởi tạo tool
memory_tool = MemoryTool(user_id="user123")
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")

# 2. Tạo ContextBuilder
config = ContextConfig(
    max_tokens=3000,
    reserve_ratio=0.2,
    min_relevance=0.2,
    enable_compression=True
)

builder = ContextBuilder(
    memory_tool=memory_tool,
    rag_tool=rag_tool,
    config=config
)

# 3. Chuẩn bị lịch sử hội thoại
conversation_history = [
    Message(content="Tôi đang phát triển một công cụ phân tích dữ liệu", role="user", timestamp=datetime.now()),
    Message(content="Tuyệt vời! Công cụ phân tích dữ liệu thường cần xử lý lượng lớn dữ liệu. Bạn dự định dùng công nghệ gì?", role="assistant", timestamp=datetime.now()),
    Message(content="Tôi dự định dùng Python và Pandas, đã hoàn thành module đọc CSV", role="user", timestamp=datetime.now()),
    Message(content="Lựa chọn tốt! Pandas rất mạnh trong xử lý dữ liệu. Tiếp theo bạn có thể cần cân nhắc làm sạch và chuyển đổi dữ liệu.", role="assistant", timestamp=datetime.now()),
]

# 4. Thêm một số memory
memory_tool.run({
    "action": "add",
    "content": "Người dùng đang phát triển công cụ phân tích dữ liệu, sử dụng Python và Pandas",
    "memory_type": "semantic",
    "importance": 0.8
})

memory_tool.run({
    "action": "add",
    "content": "Đã hoàn thành phát triển module đọc CSV",
    "memory_type": "episodic",
    "importance": 0.7
})

# 5. Xây dựng context
context = builder.build(
    user_query="Làm thế nào để tối ưu hóa mức sử dụng bộ nhớ của Pandas?",
    conversation_history=conversation_history,
    system_instructions="Bạn là một chuyên gia tư vấn kỹ thuật dữ liệu Python dày dạn kinh nghiệm. Câu trả lời của bạn cần: 1) Đưa ra đề xuất cụ thể, khả thi 2) Giải thích nguyên lý kỹ thuật 3) Đưa ra ví dụ code"
)

print("=" * 80)
print("Context đã xây dựng:")
print("=" * 80)
print(context)
print("=" * 80)
```

（2）Hiển thị kết quả chạy

Sau khi chạy đoạn code trên, bạn sẽ thấy đầu ra context có cấu trúc như sau:

```
================================================================================
构建的上下文:
================================================================================
[Role & Policies]
Bạn là một chuyên gia tư vấn kỹ thuật dữ liệu Python dày dạn kinh nghiệm. Câu trả lời của bạn cần: 1) Đưa ra đề xuất cụ thể, khả thi 2) Giải thích nguyên lý kỹ thuật 3) Đưa ra ví dụ code

[Task]
Làm thế nào để tối ưu hóa mức sử dụng bộ nhớ của Pandas?

[Evidence]
Các chiến lược cốt lõi để tối ưu hóa bộ nhớ Pandas bao gồm:
1. Sử dụng kiểu dữ liệu phù hợp (như category thay cho object)
2. Đọc file lớn theo khối (chunk)
3. Sử dụng tham số chunksize
---
Tối ưu hóa kiểu dữ liệu có thể giảm đáng kể mức sử dụng bộ nhớ. Ví dụ, hạ int64 xuống int32 có thể tiết kiệm 50% bộ nhớ.

[Context]
user: Tôi đang phát triển một công cụ phân tích dữ liệu
assistant: Tuyệt vời! Công cụ phân tích dữ liệu thường cần xử lý lượng lớn dữ liệu. Bạn dự định dùng công nghệ gì?
user: Tôi dự định dùng Python và Pandas, đã hoàn thành module đọc CSV
assistant: Lựa chọn tốt! Pandas rất mạnh trong xử lý dữ liệu. Tiếp theo bạn có thể cần cân nhắc làm sạch và chuyển đổi dữ liệu.
记忆: Người dùng đang phát triển công cụ phân tích dữ liệu, sử dụng Python và Pandas
记忆: Đã hoàn thành phát triển module đọc CSV

[Output]
Hãy dựa trên thông tin trên để cung cấp câu trả lời chính xác, có căn cứ.
================================================================================
```

Context có cấu trúc này bao gồm tất cả thông tin cần thiết:

- <strong>[Role & Policies]</strong>: xác định rõ vai trò của AI và yêu cầu trả lời
- <strong>[Task]</strong>: diễn đạt rõ ràng câu hỏi của người dùng
- <strong>[Evidence]</strong>: knowledge liên quan được truy xuất từ hệ thống RAG
- <strong>[Context]</strong>: lịch sử hội thoại và memory liên quan, cung cấp đầy đủ thông tin nền
- <strong>[Output]</strong>: hướng dẫn LLM cách tổ chức câu trả lời

（3）Tích hợp với Agent

Cuối cùng, hãy thể hiện cách tích hợp ContextBuilder vào Agent:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool

class ContextAwareAgent(SimpleAgent):
    """Agent có khả năng nhận biết context (context-aware)"""

    def __init__(self, name: str, llm: HelloAgentsLLM, **kwargs):
        super().__init__(name=name, llm=llm, system_prompt=kwargs.get("system_prompt", ""))

        # Khởi tạo context builder
        self.memory_tool = MemoryTool(user_id=kwargs.get("user_id", "default"))
        self.rag_tool = RAGTool(knowledge_base_path=kwargs.get("knowledge_base_path", "./kb"))

        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str) -> str:
        """Chạy Agent, tự động xây dựng context được tối ưu hóa"""

        # 1. Sử dụng ContextBuilder để xây dựng context được tối ưu hóa
        optimized_context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self.system_prompt
        )

        # 2. Sử dụng context đã tối ưu để gọi LLM
        messages = [
            {"role": "system", "content": optimized_context},
            {"role": "user", "content": user_input}
        ]
        response = self.llm.invoke(messages)

        # 3. Cập nhật lịch sử hội thoại
        from hello_agents.core.message import Message
        from datetime import datetime

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 4. Ghi lại các tương tác quan trọng vào hệ thống memory
        self.memory_tool.run({
            "action": "add",
            "content": f"Q: {user_input}\nA: {response[:200]}...",  # Tóm tắt
            "memory_type": "episodic",
            "importance": 0.6
        })

        return response

# Ví dụ sử dụng
agent = ContextAwareAgent(
    name="Chuyên gia tư vấn phân tích dữ liệu",
    llm=HelloAgentsLLM(),
    system_prompt="Bạn là một chuyên gia tư vấn kỹ thuật dữ liệu Python dày dạn kinh nghiệm.",
    user_id="user123",
    knowledge_base_path="./data_science_kb"
)

response = agent.run("Làm thế nào để tối ưu hóa mức sử dụng bộ nhớ của Pandas?")
print(response)
```

Thông qua cách này, ContextBuilder trở thành "bộ não quản lý context" của Agent, tự động xử lý việc thu thập, sàng lọc và tổ chức thông tin, giúp Agent luôn có thể suy luận và sinh ra kết quả dưới context tối ưu.

### 9.3.5 Best practice và đề xuất tối ưu hóa

Khi áp dụng ContextBuilder trong thực tế, các best practice sau đây đáng được chú ý:

1. <strong>Điều chỉnh động token budget</strong>: điều chỉnh động `max_tokens` dựa trên độ phức tạp của tác vụ, tác vụ đơn giản dùng budget nhỏ, tác vụ phức tạp tăng budget.

2. <strong>Tối ưu hóa tính toán relevance</strong>: trong môi trường production, thay thế overlap từ khóa đơn giản bằng tính toán vector similarity, nâng cao chất lượng retrieval.

3. <strong>Cơ chế cache</strong>: đối với system instructions và nội dung knowledge base không đổi, có thể triển khai cơ chế cache, tránh tính toán lặp lại.

4. <strong>Giám sát và log</strong>: ghi lại thông tin thống kê của mỗi lần xây dựng context (số lượng thông tin được chọn, tỷ lệ sử dụng token, v.v.), thuận tiện cho việc tối ưu hóa sau này.

5. <strong>A/B testing</strong>: đối với các tham số quan trọng (như trọng số relevance, trọng số recency), tìm cấu hình tối ưu thông qua A/B testing.



## 9.4 NoteTool: Ghi chú có cấu trúc

NoteTool là thành phần memory ngoài có cấu trúc được cung cấp cho các "tác vụ dài hạn". Nó sử dụng file Markdown làm vật mang, phần đầu sử dụng metadata YAML để ghi lại thông tin then chốt, phần nội dung dùng để ghi lại trạng thái, kết luận, điểm nghẽn và các mục hành động (action item), v.v. Thiết kế này kết hợp khả năng đọc hiểu của con người, tính thân thiện với version control và dễ dàng đưa trở lại context, là công cụ quan trọng để xây dựng Agent dài hạn.

### 9.4.1 Triết lý thiết kế và tình huống ứng dụng

Trước khi đi sâu vào chi tiết triển khai, trước tiên hãy hiểu triết lý thiết kế và các tình huống ứng dụng điển hình của NoteTool.

（1）Tại sao cần NoteTool?

Trong Chương 8, chúng ta đã giới thiệu MemoryTool, nó cung cấp khả năng quản lý memory mạnh mẽ. Tuy nhiên, MemoryTool chủ yếu quan tâm đến <strong>memory kiểu hội thoại</strong> — working memory ngắn hạn, episodic memory và semantic memory. Đối với các <strong>tác vụ kiểu dự án</strong> cần theo dõi lâu dài, quản lý có cấu trúc, chúng ta cần một phương thức ghi chép nhẹ hơn, thân thiện với con người hơn.

NoteTool lấp đầy khoảng trống này, nó cung cấp:

- <strong>Ghi chép có cấu trúc</strong>: sử dụng định dạng Markdown + YAML, vừa phù hợp để máy phân tích, vừa thuận tiện cho con người đọc và chỉnh sửa
- <strong>Thân thiện với version control</strong>: định dạng văn bản thuần, hỗ trợ tự nhiên cho các hệ thống version control như Git
- <strong>Chi phí thấp</strong>: không cần thao tác database phức tạp, phù hợp để theo dõi trạng thái nhẹ
- <strong>Phân loại linh hoạt</strong>: tổ chức ghi chú linh hoạt thông qua `type` và `tags`, hỗ trợ truy xuất đa chiều

（2）Tình huống ứng dụng điển hình

NoteTool đặc biệt phù hợp với các tình huống sau:

<strong>Tình huống 1: Theo dõi dự án dài hạn</strong>

Hãy tưởng tượng một Agent đang hỗ trợ hoàn thành tác vụ refactor một codebase lớn, điều này có thể mất vài ngày thậm chí vài tuần. NoteTool có thể ghi lại:

- `task_state`: trạng thái và tiến độ tác vụ ở giai đoạn hiện tại
- `conclusion`: các kết luận then chốt sau mỗi giai đoạn
- `blocker`: các vấn đề và điểm nghẽn gặp phải
- `action`: kế hoạch hành động tiếp theo

```python
# Ghi lại trạng thái tác vụ
notes.run({
    "action": "create",
    "title": "Dự án refactor - Giai đoạn 1",
    "content": "Đã hoàn thành refactor tầng data model, độ bao phủ test đạt 85%. Tiếp theo sẽ refactor tầng business logic.",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

# Ghi lại điểm nghẽn
notes.run({
    "action": "create",
    "title": "Vấn đề xung đột dependency",
    "content": "Phát hiện một số thư viện bên thứ ba không tương thích phiên bản, cần giải quyết. Phạm vi ảnh hưởng: 3 module ở tầng business logic.",
    "note_type": "blocker",
    "tags": ["dependency", "urgent"]
})
```

<strong>Tình huống 2: Quản lý tác vụ nghiên cứu</strong>

Một trợ lý nghiên cứu thông minh khi thực hiện literature review, có thể sử dụng NoteTool để ghi lại:

- Quan điểm cốt lõi của mỗi bài báo (`conclusion`)
- Các chủ đề cần nghiên cứu sâu thêm (`action`)
- Các tài liệu tham khảo quan trọng (`reference`)

<strong>Tình huống 3: Kết hợp với ContextBuilder</strong>

Trước mỗi vòng hội thoại, Agent có thể thông qua thao tác `search` hoặc `list` để truy xuất ghi chú liên quan, và đưa vào context:

```python
# Trong phương thức run của Agent
def run(self, user_input: str) -> str:
    # 1. Truy xuất ghi chú liên quan
    relevant_notes = self.note_tool.run({
        "action": "search",
        "query": user_input,
        "limit": 3
    })

    # 2. Chuyển nội dung ghi chú thành ContextPacket
    note_packets = []
    for note in relevant_notes:
        note_packets.append(ContextPacket(
            content=note['content'],
            timestamp=note['updated_at'],
            token_count=self._count_tokens(note['content']),
            relevance_score=0.7,
            metadata={"type": "note", "note_type": note['type']}
        ))

    # 3. Truyền ghi chú vào khi xây dựng context
    context = self.context_builder.build(
        user_query=user_input,
        custom_packets=note_packets,
        ...
    )
```

### 9.4.2 Giải thích chi tiết định dạng lưu trữ

NoteTool áp dụng định dạng lai Markdown + YAML, thiết kế này cân bằng giữa tính cấu trúc và khả năng đọc hiểu.

（1）Định dạng file ghi chú

Mỗi ghi chú là một file `.md` độc lập, có định dạng như sau:

```markdown
---
id: note_20250119_153000_0
title: Tiến độ dự án - Giai đoạn 1
type: task_state
tags: [refactoring, phase1, backend]
created_at: 2025-01-19T15:30:00
updated_at: 2025-01-19T15:30:00
---

# Tiến độ dự án - Giai đoạn 1

## Tình trạng hoàn thành

Đã hoàn thành refactor tầng data model, các thay đổi chính bao gồm:

1. Thống nhất quy ước đặt tên của entity class
2. Giới thiệu type hint, nâng cao khả năng bảo trì code
3. Tối ưu hóa hiệu suất truy vấn database

## Độ bao phủ test

- Độ bao phủ unit test: 85%
- Độ bao phủ integration test: 70%

## Kế hoạch tiếp theo

1. Refactor tầng business logic
2. Giải quyết vấn đề xung đột dependency
3. Nâng độ bao phủ integration test lên 85%
```

Ưu điểm của định dạng này:

- <strong>Metadata YAML</strong>: máy có thể phân tích, hỗ trợ trích xuất và truy xuất field chính xác
- <strong>Phần nội dung Markdown</strong>: con người có thể đọc, hỗ trợ định dạng phong phú (tiêu đề, danh sách, code block, v.v.)
- <strong>Tên file chính là ID</strong>: đơn giản hóa quản lý, tên file của mỗi ghi chú chính là định danh duy nhất của nó

（2）File index

NoteTool duy trì một file `notes_index.json`, dùng để truy xuất và quản lý ghi chú nhanh chóng:

```json
{
  "note_20250119_153000_0": {
    "id": "note_20250119_153000_0",
    "title": "Tiến độ dự án - Giai đoạn 1",
    "type": "task_state",
    "tags": ["refactoring", "phase1", "backend"],
    "created_at": "2025-01-19T15:30:00",
    "updated_at": "2025-01-19T15:30:00",
    "file_path": "./notes/note_20250119_153000_0.md"
  }
}
```

Vai trò của file index này:

- <strong>Truy xuất nhanh</strong>: không cần mở từng file, tìm kiếm trực tiếp từ index
- <strong>Quản lý metadata</strong>: quản lý tập trung metadata của tất cả ghi chú
- <strong>Kiểm tra tính toàn vẹn</strong>: có thể phát hiện file bị thiếu hoặc hỏng

### 9.4.3 Giải thích chi tiết các thao tác cốt lõi

NoteTool cung cấp bảy thao tác cốt lõi, bao phủ toàn bộ vòng đời quản lý ghi chú.

（1）create: tạo ghi chú

```python
def _create_note(
    self,
    title: str,
    content: str,
    note_type: str = "general",
    tags: Optional[List[str]] = None
) -> str:
    """Tạo ghi chú

    Args:
        title: Tiêu đề ghi chú
        content: Nội dung ghi chú (định dạng Markdown)
        note_type: Loại ghi chú (task_state/conclusion/blocker/action/reference/general)
        tags: Danh sách tag

    Returns:
        str: ID ghi chú
    """
    from datetime import datetime

    # 1. Sinh ID duy nhất
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    note_id = f"note_{timestamp}_{len(self.index)}"

    # 2. Xây dựng metadata
    metadata = {
        "id": note_id,
        "title": title,
        "type": note_type,
        "tags": tags or [],
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }

    # 3. Xây dựng nội dung file Markdown hoàn chỉnh
    md_content = self._build_markdown(metadata, content)

    # 4. Lưu vào file
    file_path = os.path.join(self.workspace, f"{note_id}.md")
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 5. Cập nhật index
    metadata["file_path"] = file_path
    self.index[note_id] = metadata
    self._save_index()

    return note_id

def _build_markdown(self, metadata: Dict, content: str) -> str:
    """Xây dựng nội dung file Markdown (YAML + nội dung)"""
    import yaml

    # Metadata YAML ở đầu
    yaml_header = yaml.dump(metadata, allow_unicode=True, sort_keys=False)

    # Kết hợp định dạng
    return f"---\n{yaml_header}---\n\n{content}"
```

Ví dụ sử dụng:

```python
from hello_agents.tools import NoteTool

notes = NoteTool(workspace="./project_notes")

note_id = notes.run({
    "action": "create",
    "title": "Dự án refactor - Giai đoạn 1",
    "content": """## Tình trạng hoàn thành
Đã hoàn thành refactor tầng data model, độ bao phủ test đạt 85%.

## Bước tiếp theo
Refactor tầng business logic""",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

print(f"✅ Tạo ghi chú thành công, ID: {note_id}")
```

（2）read: đọc ghi chú

```python
def _read_note(self, note_id: str) -> Dict:
    """Đọc nội dung ghi chú

    Args:
        note_id: ID ghi chú

    Returns:
        Dict: Dictionary chứa metadata và nội dung
    """
    if note_id not in self.index:
        raise ValueError(f"Ghi chú không tồn tại: {note_id}")

    file_path = self.index[note_id]["file_path"]

    # Đọc file
    with open(file_path, 'r', encoding='utf-8') as f:
        raw_content = f.read()

    # Phân tích metadata YAML và nội dung Markdown
    metadata, content = self._parse_markdown(raw_content)

    return {
        "metadata": metadata,
        "content": content
    }

def _parse_markdown(self, raw_content: str) -> Tuple[Dict, str]:
    """Phân tích file Markdown (tách YAML và nội dung)"""
    import yaml

    # Tìm ký tự phân cách YAML
    parts = raw_content.split('---\n', 2)

    if len(parts) >= 3:
        # Có metadata YAML ở đầu
        yaml_str = parts[1]
        content = parts[2].strip()
        metadata = yaml.safe_load(yaml_str)
    else:
        # Không có metadata, toàn bộ là nội dung
        metadata = {}
        content = raw_content.strip()

    return metadata, content
```

（3）update: cập nhật ghi chú

```python
def _update_note(
    self,
    note_id: str,
    title: Optional[str] = None,
    content: Optional[str] = None,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> str:
    """Cập nhật ghi chú

    Args:
        note_id: ID ghi chú
        title: Tiêu đề mới (tùy chọn)
        content: Nội dung mới (tùy chọn)
        note_type: Loại mới (tùy chọn)
        tags: Tag mới (tùy chọn)

    Returns:
        str: Thông báo kết quả thao tác
    """
    if note_id not in self.index:
        raise ValueError(f"Ghi chú không tồn tại: {note_id}")

    # 1. Đọc ghi chú hiện có
    note = self._read_note(note_id)
    metadata = note["metadata"]
    old_content = note["content"]

    # 2. Cập nhật các field
    if title:
        metadata["title"] = title
    if note_type:
        metadata["type"] = note_type
    if tags is not None:
        metadata["tags"] = tags
    if content is not None:
        old_content = content

    # Cập nhật timestamp
    from datetime import datetime
    metadata["updated_at"] = datetime.now().isoformat()

    # 3. Xây dựng lại và lưu
    md_content = self._build_markdown(metadata, old_content)
    file_path = metadata["file_path"]

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 4. Cập nhật index
    self.index[note_id] = metadata
    self._save_index()

    return f"✅ Ghi chú đã được cập nhật: {metadata['title']}"
```

（4）search: tìm kiếm ghi chú

```python
def _search_notes(
    self,
    query: str,
    limit: int = 10,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> List[Dict]:
    """Tìm kiếm ghi chú

    Args:
        query: Từ khóa tìm kiếm
        limit: Giới hạn số lượng trả về
        note_type: Lọc theo loại (tùy chọn)
        tags: Lọc theo tag (tùy chọn)

    Returns:
        List[Dict]: Danh sách ghi chú khớp
    """
    results = []
    query_lower = query.lower()

    for note_id, metadata in self.index.items():
        # Lọc theo loại
        if note_type and metadata.get("type") != note_type:
            continue

        # Lọc theo tag
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        # Đọc nội dung ghi chú
        try:
            note = self._read_note(note_id)
            content = note["content"]
            title = metadata.get("title", "")

            # Tìm kiếm trong tiêu đề và nội dung
            if query_lower in title.lower() or query_lower in content.lower():
                results.append({
                    "note_id": note_id,
                    "title": title,
                    "type": metadata.get("type"),
                    "tags": metadata.get("tags", []),
                    "content": content,
                    "updated_at": metadata.get("updated_at")
                })
        except Exception as e:
            print(f"[WARNING] Đọc ghi chú {note_id} thất bại: {e}")
            continue

    # Sắp xếp theo thời gian cập nhật
    results.sort(key=lambda x: x["updated_at"], reverse=True)

    return results[:limit]
```

（5）list: liệt kê ghi chú

```python
def _list_notes(
    self,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None,
    limit: int = 20
) -> List[Dict]:
    """Liệt kê ghi chú (sắp xếp giảm dần theo thời gian cập nhật)

    Args:
        note_type: Lọc theo loại (tùy chọn)
        tags: Lọc theo tag (tùy chọn)
        limit: Giới hạn số lượng trả về

    Returns:
        List[Dict]: Danh sách metadata ghi chú
    """
    results = []

    for note_id, metadata in self.index.items():
        # Lọc theo loại
        if note_type and metadata.get("type") != note_type:
            continue

        # Lọc theo tag
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        results.append(metadata)

    # Sắp xếp theo thời gian cập nhật
    results.sort(key=lambda x: x.get("updated_at", ""), reverse=True)

    return results[:limit]
```

（6）summary: tóm tắt ghi chú

```python
def _summary(self) -> Dict[str, Any]:
    """Sinh thống kê tóm tắt ghi chú

    Returns:
        Dict: Thông tin thống kê
    """
    total_count = len(self.index)

    # Thống kê theo loại
    type_counts = {}
    for metadata in self.index.values():
        note_type = metadata.get("type", "general")
        type_counts[note_type] = type_counts.get(note_type, 0) + 1

    # Ghi chú được cập nhật gần nhất
    recent_notes = sorted(
        self.index.values(),
        key=lambda x: x.get("updated_at", ""),
        reverse=True
    )[:5]

    return {
        "total_notes": total_count,
        "type_distribution": type_counts,
        "recent_notes": [
            {
                "id": note["id"],
                "title": note.get("title", ""),
                "type": note.get("type"),
                "updated_at": note.get("updated_at")
            }
            for note in recent_notes
        ]
    }
```

（7）delete: xóa ghi chú

```python
def _delete_note(self, note_id: str) -> str:
    """Xóa ghi chú

    Args:
        note_id: ID ghi chú

    Returns:
        str: Thông báo kết quả thao tác
    """
    if note_id not in self.index:
        raise ValueError(f"Ghi chú không tồn tại: {note_id}")

    # 1. Xóa file
    file_path = self.index[note_id]["file_path"]
    if os.path.exists(file_path):
        os.remove(file_path)

    # 2. Loại bỏ khỏi index
    title = self.index[note_id].get("title", note_id)
    del self.index[note_id]
    self._save_index()

    return f"✅ Ghi chú đã được xóa: {title}"
```

### 9.4.4 Tích hợp sâu với ContextBuilder

Sức mạnh thực sự của NoteTool nằm ở việc phối hợp sử dụng với ContextBuilder. Hãy cùng thể hiện cách tích hợp này qua một case hoàn chỉnh.

（1）Thiết lập tình huống

Giả sử chúng ta đang xây dựng một trợ lý dự án dài hạn, nó cần:

1. Ghi lại tiến độ theo giai đoạn của dự án
2. Theo dõi các vấn đề cần giải quyết
3. Mỗi khi hội thoại, tự động rà soát lại các ghi chú liên quan
4. Dựa trên ghi chú lịch sử để đưa ra đề xuất nhất quán

（2）Ví dụ triển khai

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, RAGTool, NoteTool
from datetime import datetime

class ProjectAssistant(SimpleAgent):
    """Trợ lý dự án dài hạn, tích hợp NoteTool và ContextBuilder"""

    def __init__(self, name: str, project_name: str, **kwargs):
        super().__init__(name=name, llm=HelloAgentsLLM(), **kwargs)

        self.project_name = project_name

        # Khởi tạo tool
        self.memory_tool = MemoryTool(user_id=project_name)
        self.rag_tool = RAGTool(knowledge_base_path=f"./{project_name}_kb")
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")

        # Khởi tạo context builder
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str, note_as_action: bool = False) -> str:
        """Chạy trợ lý, tự động tích hợp ghi chú"""

        # 1. Truy xuất ghi chú liên quan từ NoteTool
        relevant_notes = self._retrieve_relevant_notes(user_input)

        # 2. Chuyển ghi chú thành ContextPacket
        note_packets = self._notes_to_packets(relevant_notes)

        # 3. Xây dựng context được tối ưu hóa
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(),
            custom_packets=note_packets
        )

        # 4. Gọi LLM
        response = self.llm.invoke(context)

        # 5. Nếu cần, ghi lại tương tác thành ghi chú
        if note_as_action:
            self._save_as_note(user_input, response)

        # 6. Cập nhật lịch sử hội thoại
        self._update_history(user_input, response)

        return response

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Truy xuất ghi chú liên quan"""
        try:
            # Ưu tiên truy xuất ghi chú loại blocker và action
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # Tìm kiếm tổng quát
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Hợp nhất và loại bỏ trùng lặp
            all_notes = {note['note_id']: note for note in blockers + search_results}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] Truy xuất ghi chú thất bại: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Chuyển ghi chú thành context packet"""
        packets = []

        for note in notes:
            content = f"[Ghi chú:{note['title']}]\n{note['content']}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note['updated_at']),
                token_count=len(content) // 4,  # Ước lượng đơn giản
                relevance_score=0.75,  # Ghi chú có relevance tương đối cao
                metadata={
                    "type": "note",
                    "note_type": note['type'],
                    "note_id": note['note_id']
                }
            ))

        return packets

    def _save_as_note(self, user_input: str, response: str):
        """Lưu tương tác thành ghi chú"""
        try:
            # Phán đoán nên lưu thành loại ghi chú nào
            if "问题" in user_input or "阻塞" in user_input:
                note_type = "blocker"
            elif "计划" in user_input or "下一步" in user_input:
                note_type = "action"
            else:
                note_type = "conclusion"

            self.note_tool.run({
                "action": "create",
                "title": f"{user_input[:30]}...",
                "content": f"## Vấn đề\n{user_input}\n\n## Phân tích\n{response}",
                "note_type": note_type,
                "tags": [self.project_name, "auto_generated"]
            })

        except Exception as e:
            print(f"[WARNING] Lưu ghi chú thất bại: {e}")

    def _build_system_instructions(self) -> str:
        """Xây dựng system instructions"""
        return f"""Bạn là trợ lý dài hạn của dự án {self.project_name}.

Trách nhiệm của bạn:
1. Dựa trên ghi chú lịch sử để đưa ra đề xuất nhất quán
2. Theo dõi tiến độ dự án và các vấn đề cần giải quyết
3. Khi trả lời, trích dẫn các ghi chú lịch sử liên quan
4. Đưa ra đề xuất bước tiếp theo cụ thể, khả thi

Lưu ý:
- Ưu tiên quan tâm đến các vấn đề được đánh dấu blocker
- Khi đưa ra đề xuất, nêu rõ nguồn căn cứ (ghi chú, memory hoặc knowledge base)
- Duy trì nhận thức về tiến độ tổng thể của dự án"""

    def _update_history(self, user_input: str, response: str):
        """Cập nhật lịch sử hội thoại"""
        from hello_agents.core.message import Message

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Giới hạn độ dài lịch sử
        if len(self.conversation_history) > 10:
            self.conversation_history = self.conversation_history[-10:]

# Ví dụ sử dụng
assistant = ProjectAssistant(
    name="Trợ lý dự án",
    project_name="data_pipeline_refactoring"
)

# Tương tác lần đầu: ghi lại trạng thái dự án
response = assistant.run(
    "Chúng tôi đã hoàn thành refactor tầng data model, độ bao phủ test đạt 85%. Bước tiếp theo dự định refactor tầng business logic.",
    note_as_action=True
)

# Tương tác lần hai: đặt câu hỏi
response = assistant.run(
    "Khi refactor tầng business logic, tôi gặp vấn đề xung đột phiên bản dependency, nên giải quyết thế nào?"
)

# Xem tóm tắt ghi chú
summary = assistant.note_tool.run({"action": "summary"})
print(summary)
```

（3）Hiển thị kết quả chạy

```bash
[ContextBuilder] 汇集了 8 个候选信息包
[ContextBuilder] 选择了 7 个信息包,共 3500 tokens

✅ Trợ lý trả lời:

Tôi nhận thấy ghi chú trước đây của bạn có đề cập đến vấn đề này. Theo ghi chú [Dự án refactor - Giai đoạn 1], độ bao phủ test hiện tại của bạn đã đạt 85%, đây là một nền tảng rất tốt.

Về vấn đề xung đột phiên bản dependency, tôi đề xuất:

1. **Sử dụng virtual environment cách ly**: tạo virtual environment độc lập cho tầng business logic, tránh xung đột dependency với các module khác
2. **Khóa phiên bản (lock version)**: chỉ định rõ phiên bản chính xác của tất cả dependency trong requirements.txt
3. **Sử dụng pipdeptree**: phân tích cây dependency, tìm ra nguồn gốc xung đột

Vấn đề này tôi sẽ đánh dấu là blocker, đề xuất ưu tiên giải quyết.

[Nguồn căn cứ: ghi chú note_20250119_153000_0, knowledge base của dự án]

---

📋 Tóm tắt ghi chú:
{
  "total_notes": 2,
  "type_distribution": {
    "action": 1,
    "blocker": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_154500_1",
      "title": "Khi refactor tầng business logic, tôi gặp vấn đề xung đột phiên bản dependency...",
      "type": "blocker",
      "updated_at": "2025-01-19T15:45:00"
    },
    {
      "id": "note_20250119_153000_0",
      "title": "Chúng tôi đã hoàn thành refactor tầng data model...",
      "type": "action",
      "updated_at": "2025-01-19T15:30:00"
    }
  ]
}
```

### 9.4.5 Best practice

Khi sử dụng NoteTool trong thực tế, các best practice sau đây sẽ giúp bạn xây dựng Agent dài hạn mạnh mẽ hơn:

1. <strong>Phân loại ghi chú hợp lý</strong>:
   - `task_state`: ghi lại tiến độ và trạng thái theo giai đoạn
   - `conclusion`: ghi lại các kết luận và phát hiện quan trọng
   - `blocker`: ghi lại vấn đề nghẽn, ưu tiên cao nhất
   - `action`: ghi lại kế hoạch hành động tiếp theo
   - `reference`: ghi lại tài liệu tham khảo quan trọng

2. <strong>Dọn dẹp và lưu trữ (archive) định kỳ</strong>:
   - Đối với blocker đã giải quyết, cập nhật thành conclusion
   - Đối với action đã lỗi thời, kịp thời xóa hoặc cập nhật
   - Sử dụng tags để quản lý phiên bản, như `["v1.0", "completed"]`

3. <strong>Phối hợp với ContextBuilder</strong>:
   - Truy xuất ghi chú liên quan trước mỗi vòng hội thoại
   - Thiết lập relevance score khác nhau theo loại ghi chú (blocker > action > conclusion)
   - Giới hạn số lượng ghi chú, tránh quá tải context

4. <strong>Hợp tác người-máy (human-in-the-loop)</strong>:
   - Ghi chú là định dạng Markdown mà con người có thể đọc, hỗ trợ chỉnh sửa thủ công
   - Sử dụng Git để quản lý phiên bản, theo dõi sự tiến hóa của ghi chú
   - Ở các giai đoạn then chốt, con người nên kiểm duyệt thủ công ghi chú do Agent tạo ra

5. <strong>Quy trình làm việc tự động hóa</strong>:
   - Định kỳ sinh báo cáo tóm tắt ghi chú
   - Dựa trên ghi chú tự động sinh tài liệu tiến độ dự án
   - Đồng bộ nội dung ghi chú sang các hệ thống khác (như Notion, Confluence)

## 9.5 TerminalTool: Truy cập hệ thống file tức thời

Trong các chương trước, chúng ta đã giới thiệu MemoryTool và RAGTool, chúng lần lượt cung cấp khả năng memory hội thoại và truy xuất knowledge. Tuy nhiên, trong nhiều tình huống thực tế, Agent cần <strong>truy cập và khám phá hệ thống file tức thời</strong> — xem log file, phân tích cấu trúc codebase, truy xuất file cấu hình, v.v. Đây chính là nơi TerminalTool phát huy tác dụng.

TerminalTool cung cấp cho Agent <strong>khả năng thực thi command-line an toàn</strong>, hỗ trợ các lệnh xử lý văn bản và hệ thống file thường dùng, đồng thời đảm bảo an toàn hệ thống thông qua cơ chế bảo mật nhiều lớp. Thiết kế này hiện thực hóa triết lý "context tức thời (Just-in-time, JIT)" đã đề cập ở mục 9.2.2 — Agent không cần nạp sẵn tất cả các file, mà khám phá và truy xuất theo nhu cầu.

### 9.5.1 Triết lý thiết kế và cơ chế bảo mật

（1）Tại sao cần TerminalTool?

Khi xây dựng Agent dài hạn, chúng ta thường gặp các tình huống sau:

<strong>Tình huống 1: Khám phá codebase</strong>

Một trợ lý phát triển cần giúp người dùng hiểu cấu trúc của một codebase lớn:

```python
# Cách truyền thống: lập chỉ mục sẵn tất cả file (chi phí cao, có thể lỗi thời)
rag_tool.add_document("./project/**/*.py")  # Tốn thời gian, chiếm nhiều dung lượng lưu trữ

# Cách của TerminalTool: khám phá tức thời
terminal.run({"command": "find . -name '*.py' -type f"})  # Nhanh, thời gian thực
terminal.run({"command": "grep -r 'class UserService' ."})  # Định vị chính xác
terminal.run({"command": "head -n 50 src/services/user.py"})  # Xem theo nhu cầu
```

<strong>Tình huống 2: Phân tích log file</strong>

Một trợ lý vận hành (ops) cần phân tích log của ứng dụng:

```python
# Kiểm tra kích thước file log
terminal.run({"command": "ls -lh /var/log/app.log"})

# Xem log lỗi mới nhất
terminal.run({"command": "tail -n 100 /var/log/app.log | grep ERROR"})

# Thống kê phân bố loại lỗi
terminal.run({"command": "grep ERROR /var/log/app.log | cut -d':' -f3 | sort | uniq -c"})
```

<strong>Tình huống 3: Xem trước (preview) file dữ liệu</strong>

Một trợ lý phân tích dữ liệu cần nhanh chóng hiểu cấu trúc của file dữ liệu:

```python
# Xem vài dòng đầu của file CSV
terminal.run({"command": "head -n 5 data/sales.csv"})

# Đếm số dòng
terminal.run({"command": "wc -l data/*.csv"})

# Xem tên cột
terminal.run({"command": "head -n 1 data/sales.csv | tr ',' '\n'"})
```

Điểm chung của các tình huống này là: <strong>cần truy cập hệ thống file thời gian thực, nhẹ, chứ không phải lập chỉ mục và vector hóa trước</strong>. TerminalTool chính là được thiết kế cho luồng công việc "kiểu khám phá" này.

（2）Giải thích chi tiết cơ chế bảo mật

Cho phép Agent thực thi lệnh là một năng lực mạnh mẽ nhưng nguy hiểm. TerminalTool đảm bảo an toàn hệ thống thông qua cơ chế bảo mật nhiều lớp:

<strong>Lớp 1: Danh sách trắng (whitelist) lệnh</strong>

Chỉ cho phép các lệnh chỉ đọc an toàn, hoàn toàn cấm mọi thao tác có thể sửa đổi hệ thống:

```python
ALLOWED_COMMANDS = {
    # Liệt kê và thông tin file
    'ls', 'dir', 'tree',
    # Xem nội dung file
    'cat', 'head', 'tail', 'less', 'more',
    # Tìm kiếm file
    'find', 'grep', 'egrep', 'fgrep',
    # Xử lý văn bản
    'wc', 'sort', 'uniq', 'cut', 'awk', 'sed',
    # Thao tác thư mục
    'pwd', 'cd',
    # Thông tin file
    'file', 'stat', 'du', 'df',
    # Khác
    'echo', 'which', 'whereis',
}
```

Nếu Agent cố gắng thực thi lệnh ngoài danh sách trắng, sẽ bị từ chối ngay lập tức:

```python
terminal.run({"command": "rm -rf /"})
# ❌ 不允许的命令: rm
# 允许的命令: cat, cd, cut, dir, du, ...
```

<strong>Lớp 2: Giới hạn thư mục làm việc (sandbox)</strong>

TerminalTool chỉ có thể truy cập thư mục làm việc được chỉ định và các thư mục con của nó, không thể truy cập các phần khác của hệ thống:

```python
# Chỉ định thư mục làm việc khi khởi tạo
terminal = TerminalTool(workspace="./project")

# Cho phép: truy cập file trong thư mục làm việc
terminal.run({"command": "cat ./src/main.py"})  # ✅

# Cấm: truy cập file ngoài thư mục làm việc
terminal.run({"command": "cat /etc/passwd"})  # ❌ 不允许访问工作目录外的路径

# Cấm: thoát ra ngoài thông qua ..
terminal.run({"command": "cd ../../../etc"})  # ❌ 不允许访问工作目录外的路径
```

Cơ chế sandbox này đảm bảo rằng ngay cả khi hành vi của Agent xuất hiện bất thường, cũng không thể ảnh hưởng đến các phần khác của hệ thống.

<strong>Lớp 3: Kiểm soát timeout</strong>

Mỗi lệnh đều có giới hạn thời gian thực thi, ngăn chặn vòng lặp vô hạn hoặc cạn kiệt tài nguyên:

```python
terminal = TerminalTool(
    workspace="./project",
    timeout=30  # Timeout 30 giây
)

# Nếu lệnh thực thi vượt quá 30 giây
terminal.run({"command": "find / -name '*.log'"})
# ❌ 命令执行超时（超过 30 秒）
```

<strong>Lớp 4: Giới hạn kích thước output</strong>

Giới hạn kích thước output của lệnh, ngăn chặn tràn bộ nhớ:

```python
terminal = TerminalTool(
    workspace="./project",
    max_output_size=10 * 1024 * 1024  # 10MB
)

# Nếu output vượt quá 10MB
terminal.run({"command": "cat huge_file.log"})
# ... (前10MB的内容) ...
# ⚠️ 输出被截断（超过 10485760 字节）
```

Thông qua bốn lớp cơ chế bảo mật này, TerminalTool trong khi cung cấp năng lực mạnh mẽ, vẫn đảm bảo an toàn hệ thống ở mức tối đa.

### 9.5.2 Giải thích chi tiết chức năng cốt lõi

Việc triển khai TerminalTool tập trung vào hai chức năng cốt lõi: thực thi lệnh và điều hướng thư mục.

（1）Thực thi lệnh

Phương thức cốt lõi `_execute_command` chịu trách nhiệm thực thi lệnh thực tế:

```python
def _execute_command(self, command: str) -> str:
    """Thực thi lệnh"""
    try:
        # Thực thi lệnh trong thư mục hiện tại
        result = subprocess.run(
            command,
            shell=True,
            cwd=str(self.current_dir),  # Thực thi trong thư mục làm việc hiện tại
            capture_output=True,
            text=True,
            timeout=self.timeout,
            env=os.environ.copy()
        )

        # Hợp nhất standard output và standard error
        output = result.stdout
        if result.stderr:
            output += f"\n[stderr]\n{result.stderr}"

        # Kiểm tra kích thước output
        if len(output) > self.max_output_size:
            output = output[:self.max_output_size]
            output += f"\n\n⚠️ 输出被截断（超过 {self.max_output_size} 字节）"

        # Thêm thông tin mã trả về
        if result.returncode != 0:
            output = f"⚠️ 命令返回码: {result.returncode}\n\n{output}"

        return output if output else "✅ 命令执行成功（无输出）"

    except subprocess.TimeoutExpired:
        return f"❌ 命令执行超时（超过 {self.timeout} 秒）"
    except Exception as e:
        return f"❌ 命令执行失败: {e}"
```

Các điểm then chốt của triển khai này:

- <strong>Nhận biết thư mục hiện tại</strong>: sử dụng tham số `cwd` để thực thi lệnh trong đúng thư mục
- <strong>Xử lý lỗi</strong>: bắt và hợp nhất standard error, cung cấp thông tin chẩn đoán đầy đủ
- <strong>Kiểm tra mã trả về</strong>: mã trả về khác không sẽ được đánh dấu là cảnh báo
- <strong>Thiết kế chịu lỗi</strong>: timeout và ngoại lệ đều được xử lý thỏa đáng, không khiến Agent bị crash

（2）Điều hướng thư mục

Xử lý đặc biệt của lệnh `cd` hỗ trợ Agent điều hướng trong hệ thống file:

```python
def _handle_cd(self, parts: List[str]) -> str:
    """Xử lý lệnh cd"""
    if not self.allow_cd:
        return "❌ cd 命令已禁用"

    if len(parts) < 2:
        # cd không tham số, trả về thư mục hiện tại
        return f"当前目录: {self.current_dir}"

    target_dir = parts[1]

    # Xử lý đường dẫn tương đối
    if target_dir == "..":
        new_dir = self.current_dir.parent
    elif target_dir == ".":
        new_dir = self.current_dir
    elif target_dir == "~":
        new_dir = self.workspace
    else:
        new_dir = (self.current_dir / target_dir).resolve()

    # Kiểm tra xem có nằm trong thư mục làm việc không
    try:
        new_dir.relative_to(self.workspace)
    except ValueError:
        return f"❌ 不允许访问工作目录外的路径: {new_dir}"

    # Kiểm tra thư mục có tồn tại không
    if not new_dir.exists():
        return f"❌ 目录不存在: {new_dir}"

    if not new_dir.is_dir():
        return f"❌ 不是目录: {new_dir}"

    # Cập nhật thư mục hiện tại
    self.current_dir = new_dir
    return f"✅ 切换到目录: {self.current_dir}"
```

Thiết kế này hỗ trợ Agent thực hiện khám phá hệ thống file nhiều bước:

```python
# Bước 1: xem cấu trúc dự án
terminal.run({"command": "ls -la"})

# Bước 2: vào thư mục source code
terminal.run({"command": "cd src"})

# Bước 3: tìm file cụ thể
terminal.run({"command": "find . -name '*service*.py'"})

# Bước 4: xem nội dung file
terminal.run({"command": "cat user_service.py"})
```

### 9.5.3 Các mẫu sử dụng điển hình

TerminalTool hỗ trợ nhiều mẫu thao tác hệ thống file thường gặp.

（1）Điều hướng kiểu khám phá

Agent có thể từng bước khám phá codebase giống như một developer con người:

```python
from hello_agents.tools import TerminalTool

terminal = TerminalTool(workspace="./my_project")

# Bước 1: xem thư mục gốc của dự án
print(terminal.run({"command": "ls -la"}))
"""
total 24
drwxr-xr-x  6 user  staff   192 Jan 19 16:00 .
drwxr-xr-x  5 user  staff   160 Jan 19 15:30 ..
-rw-r--r--  1 user  staff  1234 Jan 19 15:30 README.md
drwxr-xr-x  4 user  staff   128 Jan 19 15:30 src
drwxr-xr-x  3 user  staff    96 Jan 19 15:30 tests
-rw-r--r--  1 user  staff   456 Jan 19 15:30 requirements.txt
"""

# Bước 2: xem cấu trúc thư mục source code
terminal.run({"command": "cd src"})
print(terminal.run({"command": "tree"}))

# Bước 3: tìm kiếm mẫu cụ thể
print(terminal.run({"command": "grep -r 'def process' ."}))
```

（2）Phân tích file dữ liệu

Nhanh chóng hiểu cấu trúc và nội dung của file dữ liệu:

```python
terminal = TerminalTool(workspace="./data")

# Xem vài dòng đầu của file CSV
print(terminal.run({"command": "head -n 5 sales_2024.csv"}))
"""
date,product,quantity,revenue
2024-01-01,Widget A,150,4500.00
2024-01-01,Widget B,200,8000.00
2024-01-02,Widget A,180,5400.00
2024-01-02,Widget C,120,3600.00
"""

# Đếm tổng số dòng
print(terminal.run({"command": "wc -l *.csv"}))
"""
  10234 sales_2024.csv
   8567 sales_2023.csv
  18801 total
"""

# Trích xuất và thống kê danh mục sản phẩm
print(terminal.run({"command": "tail -n +2 sales_2024.csv | cut -d',' -f2 | sort | uniq -c"}))
"""
  3456 Widget A
  4123 Widget B
  2655 Widget C
"""
```

（3）Phân tích log file

Phân tích log ứng dụng thời gian thực, nhanh chóng định vị vấn đề:

```python
terminal = TerminalTool(workspace="/var/log")

# Xem log lỗi mới nhất
print(terminal.run({"command": "tail -n 50 app.log | grep ERROR"}))

# Thống kê phân bố loại lỗi
print(terminal.run({"command": "grep ERROR app.log | awk '{print $4}' | sort | uniq -c | sort -rn"}))
"""
  245 DatabaseConnectionError
  123 TimeoutException
   67 ValidationError
   34 AuthenticationError
"""

# Tìm log trong khoảng thời gian cụ thể
print(terminal.run({"command": "grep '2024-01-19 15:' app.log | tail -n 20"}))
```

（4）Phân tích codebase

Hỗ trợ code review và hiểu codebase:

```python
terminal = TerminalTool(workspace="./codebase")

# Đếm số dòng code
print(terminal.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"}))

# Tìm tất cả comment TODO
print(terminal.run({"command": "grep -rn 'TODO' --include='*.py'"}))

# Tìm định nghĩa của hàm cụ thể
print(terminal.run({"command": "grep -rn 'def process_data' --include='*.py'"}))

# Xem triển khai của hàm
print(terminal.run({"command": "sed -n '/def process_data/,/^def /p' src/processor.py | head -n -1"}))
```

### 9.5.4 Phối hợp với các tool khác

Sức mạnh thực sự của TerminalTool nằm ở việc phối hợp sử dụng với MemoryTool, NoteTool và ContextBuilder.

（1）Phối hợp với MemoryTool

Thông tin mà TerminalTool phát hiện có thể được lưu vào hệ thống memory:

```python
# Sử dụng TerminalTool để phát hiện cấu trúc dự án
structure = terminal.run({"command": "tree -L 2 src"})

# Lưu vào semantic memory
memory_tool.run({
    "action": "add",
    "content": f"项目结构:\n{structure}",
    "memory_type": "semantic",
    "importance": 0.8,
    "metadata": {"type": "project_structure"}
})
```

（2）Phối hợp với NoteTool

Những phát hiện quan trọng có thể được ghi lại thành ghi chú có cấu trúc:

```python
# Phát hiện một điểm nghẽn hiệu suất
log_analysis = terminal.run({"command": "grep 'slow query' app.log | tail -n 10"})

# Ghi lại thành ghi chú blocker
note_tool.run({
    "action": "create",
    "title": "Vấn đề truy vấn chậm của database",
    "content": f"## Mô tả vấn đề\nPhát hiện nhiều truy vấn chậm, ảnh hưởng hiệu suất hệ thống\n\n## Phân tích log\n```\n{log_analysis}\n```\n\n## Bước tiếp theo\n1. Phân tích SQL truy vấn chậm\n2. Thêm index\n3. Tối ưu hóa logic truy vấn",
    "note_type": "blocker",
    "tags": ["performance", "database"]
})
```

（3）Phối hợp với ContextBuilder

Output của TerminalTool có thể trở thành một phần của context:

```python
# Khám phá codebase
code_structure = terminal.run({"command": "ls -R src"})
recent_changes = terminal.run({"command": "git log --oneline -10"})

# Chuyển thành ContextPacket
from hello_agents.context import ContextPacket
from datetime import datetime

packets = [
    ContextPacket(
        content=f"代码库结构:\n{code_structure}",
        timestamp=datetime.now(),
        token_count=len(code_structure) // 4,
        relevance_score=0.7,
        metadata={"type": "code_structure", "source": "terminal"}
    ),
    ContextPacket(
        content=f"最近提交:\n{recent_changes}",
        timestamp=datetime.now(),
        token_count=len(recent_changes) // 4,
        relevance_score=0.8,
        metadata={"type": "git_history", "source": "terminal"}
    )
]

# Bao gồm các thông tin này khi xây dựng context
context = context_builder.build(
    user_query="如何重构用户服务模块?",
    custom_packets=packets
)
```

## 9.6 Thực chiến Agent dài hạn: trợ lý bảo trì codebase

Bây giờ, hãy tích hợp ContextBuilder, NoteTool và TerminalTool lại với nhau, xây dựng một Agent dài hạn hoàn chỉnh — <strong>trợ lý bảo trì codebase</strong>. Trợ lý này có thể:

1. Khám phá và hiểu cấu trúc codebase
2. Ghi lại các vấn đề và điểm cải thiện đã phát hiện
3. Theo dõi tác vụ refactor dài hạn
4. Duy trì tính nhất quán dưới giới hạn context window

### 9.6.1 Thiết lập tình huống và phân tích yêu cầu

<strong>Tình huống nghiệp vụ</strong>

Giả sử chúng ta đang bảo trì một ứng dụng web Python cỡ trung, codebase này bao gồm khoảng 50 file Python, được xây dựng bằng framework Flask, bao phủ nhiều module như data model, business logic, API interface, đồng thời tồn tại một số technical debt cần được dọn dẹp dần. Trong tình huống như vậy, chúng ta cần một trợ lý thông minh để giúp chúng ta khám phá codebase, hiểu cấu trúc dự án, mối quan hệ dependency và phong cách code; nhận diện các vấn đề trong code, như code trùng lặp, độ phức tạp quá cao, thiếu test; theo dõi tiến độ tác vụ, ghi lại các mục cần làm, công việc đã hoàn thành và các điểm nghẽn gặp phải; và đưa ra đề xuất refactor nhất quán dựa trên context lịch sử.

<strong>Thách thức và giải pháp</strong>

Tình huống này đối mặt với một số thách thức điển hình của tác vụ dài hạn. Đầu tiên là vấn đề lượng thông tin vượt quá context window, toàn bộ codebase có thể bao gồm hàng vạn dòng code, không thể nạp vào context window cùng lúc, chúng ta giải quyết vấn đề này bằng cách sử dụng TerminalTool để khám phá code tức thời, theo nhu cầu, chỉ xem file cụ thể khi cần thiết. Thứ hai là thách thức quản lý trạng thái xuyên phiên (cross-session), tác vụ refactor có thể kéo dài nhiều ngày, cần duy trì tiến độ qua nhiều phiên, chúng ta sử dụng NoteTool để ghi lại tiến độ theo giai đoạn, các mục cần làm và quyết định then chốt để đối phó. Cuối cùng là vấn đề chất lượng và độ liên quan của context, mỗi lần hội thoại cần rà soát lại thông tin lịch sử liên quan, nhưng không được bị thông tin không liên quan nhấn chìm, chúng ta thông qua ContextBuilder để sàng lọc và tổ chức context một cách thông minh, đảm bảo mật độ tín hiệu cao.

### 9.6.2 Thiết kế kiến trúc hệ thống

Trợ lý bảo trì codebase của chúng ta áp dụng kiến trúc ba lớp, như hình 9.3 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-3.png" alt="" width="85%"/>
  <p>Hình 9.3 Kiến trúc ba lớp của trợ lý bảo trì codebase</p>
</div>


### 9.6.3 Triển khai cốt lõi

Bây giờ hãy triển khai class cốt lõi của hệ thống này:

```python
from typing import Dict， Any, List, Optional
from datetime import datetime
import json

from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, NoteTool, TerminalTool
from hello_agents.core.message import Message


class CodebaseMaintainer:
    """Trợ lý bảo trì codebase - ví dụ Agent dài hạn

    Tích hợp ContextBuilder + NoteTool + TerminalTool + MemoryTool
    Thực hiện quản lý tác vụ bảo trì codebase xuyên phiên
    """

    def __init__(
        self,
        project_name: str,
        codebase_path: str,
        llm: Optional[HelloAgentsLLM] = None
    ):
        self.project_name = project_name
        self.codebase_path = codebase_path
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Khởi tạo LLM
        self.llm = llm or HelloAgentsLLM()

        # Khởi tạo tool
        self.memory_tool = MemoryTool(user_id=project_name)
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")
        self.terminal_tool = TerminalTool(workspace=codebase_path, timeout=60)

        # Khởi tạo context builder
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=None,  # Case này không sử dụng RAG
            config=ContextConfig(
                max_tokens=4000,
                reserve_ratio=0.15,
                min_relevance=0.2,
                enable_compression=True
            )
        )

        # Lịch sử hội thoại
        self.conversation_history: List[Message] = []

        # Thông tin thống kê
        self.stats = {
            "session_start": datetime.now(),
            "commands_executed": 0,
            "notes_created": 0,
            "issues_found": 0
        }

        print(f"✅ 代码库维护助手已初始化: {project_name}")
        print(f"📁 工作目录: {codebase_path}")
        print(f"🆔 会话ID: {self.session_id}")

    def run(self, user_input: str, mode: str = "auto") -> str:
        """Chạy trợ lý

        Args:
            user_input: Đầu vào của người dùng
            mode: Chế độ chạy
                - "auto": tự động quyết định có dùng tool hay không
                - "explore": thiên về khám phá code
                - "analyze": thiên về phân tích vấn đề
                - "plan": thiên về lập kế hoạch tác vụ

        Returns:
            str: Câu trả lời của trợ lý
        """
        print(f"\n{'='*80}")
        print(f"👤 用户: {user_input}")
        print(f"{'='*80}\n")

        # Bước 1: thực hiện tiền xử lý theo chế độ
        pre_context = self._preprocess_by_mode(user_input, mode)

        # Bước 2: truy xuất ghi chú liên quan
        relevant_notes = self._retrieve_relevant_notes(user_input)
        note_packets = self._notes_to_packets(relevant_notes)

        # Bước 3: xây dựng context được tối ưu hóa
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(mode),
            custom_packets=note_packets + pre_context
        )

        # Bước 4: gọi LLM
        print("🤖 正在思考...")
        response = self.llm.invoke(context)

        # Bước 5: hậu xử lý
        self._postprocess_response(user_input, response)

        # Bước 6: cập nhật lịch sử hội thoại
        self._update_history(user_input, response)

        print(f"\n🤖 助手: {response}\n")
        print(f"{'='*80}\n")

        return response

    def _preprocess_by_mode(
        self,
        user_input: str,
        mode: str
    ) -> List[ContextPacket]:
        """Thực hiện tiền xử lý theo chế độ, thu thập thông tin liên quan"""
        packets = []

        if mode == "explore" or mode == "auto":
            # Chế độ khám phá: tự động xem cấu trúc dự án
            print("🔍 探索代码库结构...")

            structure = self.terminal_tool.run({"command": "find . -type f -name '*.py' | head -n 20"})
            self.stats["commands_executed"] += 1

            packets.append(ContextPacket(
                content=f"[代码库结构]\n{structure}",
                timestamp=datetime.now(),
                token_count=len(structure) // 4,
                relevance_score=0.6,
                metadata={"type": "code_structure", "source": "terminal"}
            ))

        if mode == "analyze":
            # Chế độ phân tích: kiểm tra độ phức tạp code và vấn đề
            print("📊 分析代码质量...")

            # Thống kê số dòng code
            loc = self.terminal_tool.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"})

            # Tìm TODO và FIXME
            todos = self.terminal_tool.run({"command": "grep -rn 'TODO\\|FIXME' --include='*.py' | head -n 10"})

            self.stats["commands_executed"] += 2

            packets.append(ContextPacket(
                content=f"[代码统计]\n{loc}\n\n[待办事项]\n{todos}",
                timestamp=datetime.now(),
                token_count=(len(loc) + len(todos)) // 4,
                relevance_score=0.7,
                metadata={"type": "code_analysis", "source": "terminal"}
            ))

        if mode == "plan":
            # Chế độ lập kế hoạch: nạp các ghi chú gần đây
            print("📋 加载任务规划...")

            task_notes = self.note_tool.run({
                "action": "list",
                "note_type": "task_state",
                "limit": 3
            })

            if task_notes:
                content = "\n".join([f"- {note['title']}" for note in task_notes])
                packets.append(ContextPacket(
                    content=f"[当前任务]\n{content}",
                    timestamp=datetime.now(),
                    token_count=len(content) // 4,
                    relevance_score=0.8,
                    metadata={"type": "task_plan", "source": "notes"}
                ))

        return packets

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Truy xuất ghi chú liên quan"""
        try:
            # Ưu tiên truy xuất blocker
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # Tìm kiếm ghi chú liên quan
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Hợp nhất loại bỏ trùng lặp
            all_notes = {note.get('note_id') or note.get('id'): note for note in (blockers or []) + (search_results or [])}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] 笔记检索失败: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Chuyển ghi chú thành context packet"""
        packets = []

        for note in notes:
            # Thiết lập relevance score khác nhau theo loại ghi chú
            relevance_map = {
                "blocker": 0.9,
                "action": 0.8,
                "task_state": 0.75,
                "conclusion": 0.7
            }

            note_type = note.get('type', 'general')
            relevance = relevance_map.get(note_type, 0.6)

            content = f"[笔记:{note.get('title', 'Untitled')}]\n类型: {note_type}\n\n{note.get('content', '')}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note.get('updated_at', datetime.now().isoformat())),
                token_count=len(content) // 4,
                relevance_score=relevance,
                metadata={
                    "type": "note",
                    "note_type": note_type,
                    "note_id": note.get('note_id') or note.get('id')
                }
            ))

        return packets

    def _build_system_instructions(self, mode: str) -> str:
        """Xây dựng system instructions"""
        base_instructions = f"""你是 {self.project_name} 项目的代码库维护助手。

你的核心能力:
1. 使用 TerminalTool 探索代码库(ls, cat, grep, find等)
2. 使用 NoteTool 记录发现和任务
3. 基于历史笔记提供连贯的建议

当前会话ID: {self.session_id}
"""

        mode_specific = {
            "explore": """
当前模式: 探索代码库

你应该:
- 主动使用 terminal 命令了解代码结构
- 识别关键模块和文件
- 记录项目架构到笔记
""",
            "analyze": """
当前模式: 分析代码质量

你应该:
- 查找代码问题(重复、复杂度、TODO等)
- 评估代码质量
- 将发现的问题记录为 blocker 或 action 笔记
""",
            "plan": """
当前模式: 任务规划

你应该:
- 回顾历史笔记和任务
- 制定下一步行动计划
- 更新任务状态笔记
""",
            "auto": """
当前模式: 自动决策

你应该:
- 根据用户需求灵活选择策略
- 在需要时使用工具
- 保持回答的专业性和实用性
"""
        }

        return base_instructions + mode_specific.get(mode, mode_specific["auto"])

    def _postprocess_response(self, user_input: str, response: str):
        """Hậu xử lý: phân tích câu trả lời, tự động ghi lại thông tin quan trọng"""

        # Nếu phát hiện vấn đề, tự động tạo ghi chú blocker
        if any(keyword in response.lower() for keyword in ["问题", "bug", "错误", "阻塞"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"发现问题: {user_input[:30]}...",
                    "content": f"## 用户输入\n{user_input}\n\n## 问题分析\n{response[:500]}...",
                    "note_type": "blocker",
                    "tags": [self.project_name, "auto_detected", self.session_id]
                })
                self.stats["notes_created"] += 1
                self.stats["issues_found"] += 1
                print("📝 已自动创建问题笔记")
            except Exception as e:
                print(f"[WARNING] 创建笔记失败: {e}")

        # Nếu là lập kế hoạch tác vụ, tự động tạo ghi chú action
        elif any(keyword in user_input.lower() for keyword in ["计划", "下一步", "任务", "todo"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"任务规划: {user_input[:30]}...",
                    "content": f"## 讨论\n{user_input}\n\n## 行动计划\n{response[:500]}...",
                    "note_type": "action",
                    "tags": [self.project_name, "planning", self.session_id]
                })
                self.stats["notes_created"] += 1
                print("📝 已自动创建行动计划笔记")
            except Exception as e:
                print(f"[WARNING] 创建笔记失败: {e}")

    def _update_history(self, user_input: str, response: str):
        """Cập nhật lịch sử hội thoại"""
        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Giới hạn độ dài lịch sử (giữ lại 10 vòng hội thoại gần nhất)
        if len(self.conversation_history) > 20:
            self.conversation_history = self.conversation_history[-20:]

    # === Phương thức tiện ích ===

    def explore(self, target: str = ".") -> str:
        """Khám phá codebase"""
        return self.run(f"请探索 {target} 的代码结构", mode="explore")

    def analyze(self, focus: str = "") -> str:
        """Phân tích chất lượng code"""
        query = f"请分析代码质量" + (f",重点关注{focus}" if focus else "")
        return self.run(query, mode="analyze")

    def plan_next_steps(self) -> str:
        """Lập kế hoạch bước tiếp theo"""
        return self.run("根据当前进度,规划下一步任务", mode="plan")

    def execute_command(self, command: str) -> str:
        """Thực thi lệnh terminal"""
        result = self.terminal_tool.run({"command": command})
        self.stats["commands_executed"] += 1
        return result

    def create_note(
        self,
        title: str,
        content: str,
        note_type: str = "general",
        tags: List[str] = None
    ) -> str:
        """Tạo ghi chú"""
        result = self.note_tool.run({
            "action": "create",
            "title": title,
            "content": content,
            "note_type": note_type,
            "tags": tags or [self.project_name]
        })
        self.stats["notes_created"] += 1
        return result

    def get_stats(self) -> Dict[str, Any]:
        """Lấy thông tin thống kê"""
        duration = (datetime.now() - self.stats["session_start"]).total_seconds()

        # Lấy tóm tắt ghi chú
        try:
            note_summary = self.note_tool.run({"action": "summary"})
        except:
            note_summary = {}

        return {
            "session_info": {
                "session_id": self.session_id,
                "project": self.project_name,
                "duration_seconds": duration
            },
            "activity": {
                "commands_executed": self.stats["commands_executed"],
                "notes_created": self.stats["notes_created"],
                "issues_found": self.stats["issues_found"]
            },
            "notes": note_summary
        }

    def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
        """Sinh báo cáo phiên làm việc"""
        report = self.get_stats()

        if save_to_file:
            report_file = f"maintainer_report_{self.session_id}.json"
            with open(report_file, 'w', encoding='utf-8') as f:
                json.dump(report, f, ensure_ascii=False, indent=2, default=str)
            report["report_file"] = report_file
            print(f"📄 报告已保存: {report_file}")

        return report
```

### 9.6.4 Ví dụ sử dụng hoàn chỉnh

Bây giờ hãy thông qua một tình huống sử dụng hoàn chỉnh, thể hiện luồng công việc của Agent dài hạn này:

```python
# ========== Khởi tạo trợ lý ==========

from hello_agents import HelloAgentsLLM

maintainer = CodebaseMaintainer(
    project_name="my_flask_app",
    codebase_path="./my_flask_app",
    llm=HelloAgentsLLM()
)


# ========== Ngày 1: Khám phá codebase ==========
# 1. Khám phá sơ bộ
response = maintainer.explore()
"""
🔍 探索代码库结构...
🤖 正在思考...

🤖 助手: 我已经扫描了代码库,这是一个 Flask Web 应用,主要包含以下模块:

📁 项目结构:
- app/
  ├── models/      # 数据模型(User, Product, Order等)
  ├── routes/      # API路由
  ├── services/    # 业务逻辑
  └── utils/       # 工具函数
- tests/           # 测试文件
- migrations/      # 数据库迁移
- config.py        # 配置文件

总计约 3,500 行 Python 代码。

建议下一步:
1. 查看核心模型定义
2. 检查路由和API设计
3. 评估测试覆盖率
"""

# 2. Phân tích sâu một module
response = maintainer.run("请分析 app/models/ 目录下的数据模型设计")
"""
🤖 助手: 我查看了 app/models/ 目录,发现以下数据模型:

1. User (user.py)
   - 字段: id, username, email, password_hash, created_at
   - 关系: 一对多 Order
   - 问题: ⚠️ 缺少索引,email 字段未设置唯一约束

2. Product (product.py)
   - 字段: id, name, description, price, stock
   - 问题: ✅ 设计合理

3. Order (order.py)
   - 字段: id, user_id, product_id, quantity, status
   - 问题: ⚠️ 缺少创建时间字段,不利于数据分析

建议:
1. 为 User.email 添加唯一约束
2. 为 Order 添加 created_at 和 updated_at 字段
3. 考虑添加软删除支持

我已将这些问题记录为 blocker 笔记。
"""
# [Đã tự động tạo ghi chú blocker]


# ========== Ngày 2: Phân tích chất lượng code ==========

# 1. Phân tích chất lượng tổng thể
response = maintainer.analyze()
"""
📊 分析代码质量...
🤖 正在思考...

🤖 助手: 代码质量分析报告:

📊 代码统计:
- 总行数: 3,542 行
- 平均文件长度: 约 150 行
- 最大文件: services/order_service.py (456 行)

⚠️ 发现的问题:

1. 代码重复 (HIGH):
   - user_service.py 和 product_service.py 有相似的 CRUD 逻辑
   - 建议: 提取基类 BaseService

2. 复杂度过高 (MEDIUM):
   - order_service.py::process_order 方法有 8 层嵌套
   - 建议: 拆分为多个小方法

3. 缺少测试 (HIGH):
   - 测试覆盖率仅 45%
   - services/ 目录完全没有测试

4. TODO/FIXME (12 个):
   - 大部分是"稍后优化"类的注释
   - 建议: 清理或转化为正式任务

我已将这些问题记录为笔记,并标记优先级。
"""
# [Đã tự động tạo nhiều ghi chú blocker]

# 2. Xem vấn đề cụ thể
response = maintainer.run("请查看 order_service.py 的 process_order 方法,给出重构建议")
"""
🤖 助手: 我查看了 process_order 方法的代码:

【原始代码】
def process_order(self, order_id):
    order = Order.query.get(order_id)
    if order:
        if order.status == 'pending':
            product = Product.query.get(order.product_id)
            if product:
                if product.stock >= order.quantity:
                    # ... 8 层嵌套继续 ...

【问题分析】
1. 过深的嵌套导致可读性差
2. 缺少错误处理
3. 业务逻辑与数据访问混在一起

【重构建议】
def process_order(self, order_id):
    # 1. 早返回模式,减少嵌套
    order = self._get_order(order_id)
    if not order or order.status != 'pending':
        return False

    # 2. 提取方法,单一职责
    if not self._check_stock(order):
        return False

    # 3. 使用事务确保一致性
    with db.transaction():
        self._update_stock(order)
        self._update_order_status(order, 'processing')

    return True

【下一步】
1. 我已创建重构计划笔记
2. 建议先编写测试用例
3. 逐步重构,保持功能不变
"""
# [Đã tự động tạo ghi chú action]


# ========== Ngày 3: Lập kế hoạch tác vụ refactor ==========

# 1. Rà soát tiến độ
response = maintainer.plan_next_steps()
"""
📋 加载任务规划...
🤖 正在思考...

🤖 助手: 基于前两天的探索和分析,我整理了重构任务优先级:

🔴 高优先级 (本周完成):
1. [blocker] User.email 添加唯一约束
   - 风险: 低,只是数据库迁移
   - 工作量: 1小时

2. [blocker] 提取 BaseService 减少代码重复
   - 风险: 中,影响多个服务类
   - 工作量: 半天

3. [blocker] 重构 process_order 方法
   - 风险: 高,核心业务逻辑
   - 工作量: 1天(包括编写测试)

🟡 中优先级 (下周):
4. 为 services/ 添加单元测试
5. 清理 TODO 注释
6. 为 Order 添加时间字段

🟢 低优先级 (待定):
7. 性能优化
8. 文档更新

建议:
- 本周先解决数据模型问题(任务1、6)
- 下周进行代码重构(任务2、3)
- 持续增加测试覆盖率(任务4)

我已更新任务状态笔记,您可以随时查看进度。
"""

# 2. Tạo thủ công kế hoạch refactor chi tiết
maintainer.create_note(
    title="本周重构计划 - Week 1",
    content="""## 目标
完成数据模型层的优化

## 任务清单
- [ ] 为 User.email 添加唯一约束
- [ ] 为 Order 添加 created_at, updated_at 字段
- [ ] 编写数据库迁移脚本
- [ ] 更新相关测试用例

## 时间安排
- 周一: 设计迁移脚本
- 周二-周三: 执行迁移并测试
- 周四: 更新测试用例
- 周五: Code Review

## 风险
- 数据库迁移可能影响线上环境,需要在非高峰期执行
- 现有数据中可能存在重复email,需要先清理
""",
    note_type="task_state",
    tags=["refactoring", "week1", "high_priority"]
)

print("✅ 已创建详细的重构计划")


# ========== Một tuần sau: kiểm tra tiến độ ==========

# Xem tóm tắt ghi chú
summary = maintainer.note_tool.run({"action": "summary"})
print("📊 笔记摘要:")
print(json.dumps(summary, indent=2, ensure_ascii=False))
"""
{
  "total_notes": 8,
  "type_distribution": {
    "blocker": 3,
    "action": 2,
    "task_state": 2,
    "conclusion": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_160000_7",
      "title": "本周重构计划 - Week 1",
      "type": "task_state",
      "updated_at": "2025-01-19T16:00:00"
    },
    ...
  ]
}
"""

# Sinh báo cáo đầy đủ
report = maintainer.generate_report()
print("\n📄 会话报告:")
print(json.dumps(report, indent=2, ensure_ascii=False))
"""
{
  "session_info": {
    "session_id": "session_20250119_150000",
    "project": "my_flask_app",
    "duration_seconds": 172800  # 2天
  },
  "activity": {
    "commands_executed": 24,
    "notes_created": 8,
    "issues_found": 3
  },
  "notes": { ... }
}
"""
```

### 9.6.5 Phân tích hiệu quả hoạt động

Thông qua case hoàn chỉnh này, chúng ta có thể thấy một số đặc tính then chốt của Agent dài hạn. Đầu tiên là tính nhất quán xuyên phiên, Agent thông qua NoteTool duy trì được tính nhất quán của tác vụ qua nhiều ngày, nhiều phiên, các vấn đề được khám phá ở ngày đầu tiên tự động được xem xét khi phân tích ở ngày thứ hai, khi lập kế hoạch ở ngày thứ ba có thể tổng hợp tất cả phát hiện của hai ngày trước, khi kiểm tra sau một tuần toàn bộ lịch sử đều được giữ lại đầy đủ. Thứ hai là quản lý context thông minh, ContextBuilder đảm bảo mỗi lần hội thoại đều có context chất lượng cao, tự động tổng hợp các ghi chú liên quan (đặc biệt loại blocker), điều chỉnh động chiến lược tiền xử lý theo chế độ hội thoại, lựa chọn thông tin liên quan nhất trong token budget.

Đặc tính thứ ba là truy cập hệ thống file tức thời, TerminalTool hỗ trợ khám phá code linh hoạt, không cần lập chỉ mục trước toàn bộ codebase, có thể xem ngay nội dung file cụ thể, hỗ trợ xử lý văn bản phức tạp (grep, awk, v.v.). Thứ tư là quản lý knowledge tự động hóa, hệ thống tự động hóa quản lý knowledge đã phát hiện, khi phát hiện vấn đề tự động tạo ghi chú blocker, khi thảo luận kế hoạch tự động tạo ghi chú action, thông tin then chốt tự động lưu vào hệ thống memory. Cuối cùng là hợp tác người-máy, hệ thống này hỗ trợ mô hình hợp tác người-máy linh hoạt, Agent có thể tự động hóa hoàn thành việc khám phá và phân tích, con người có thể can thiệp và hướng dẫn thông qua hệ thống ghi chú, hỗ trợ tạo thủ công các ghi chú kế hoạch chi tiết.

Khung cơ bản này có thể được mở rộng thêm, ví dụ tích hợp RAGTool để xây dựng chỉ mục vector cho codebase kết hợp semantic retrieval, tách thành các vai trò chuyên biệt như explorer, analyzer, planner để thực hiện hợp tác đa Agent, tích hợp công cụ test để tự động xác minh kết quả refactor, thông qua TerminalTool thực thi lệnh git để theo dõi thay đổi code, hoặc sử dụng Gradio/Streamlit để xây dựng giao diện trực quan.

## 9.7 Tổng kết chương

Trong chương này, chúng ta đã đi sâu khám phá nền tảng lý thuyết và thực hành kỹ thuật của Context Engineering:

### Về mặt lý thuyết

1. <strong>Bản chất của Context Engineering</strong>: sự tiến hóa từ "Prompt Engineering" đến "Context Engineering", cốt lõi là quản lý ngân sách chú ý hữu hạn
2. <strong>Context rot</strong>: hiểu về sự suy giảm hiệu suất do context dài gây ra, nhận thức rằng context là tài nguyên khan hiếm
3. <strong>Ba chiến lược lớn</strong>: compaction, structured note-taking, sub-agent architectures

### Về mặt thực hành kỹ thuật

1. <strong>ContextBuilder</strong>: triển khai pipeline GSSC, cung cấp giao diện quản lý context thống nhất
2. <strong>NoteTool</strong>: định dạng lai Markdown+YAML, hỗ trợ memory dài hạn có cấu trúc
3. <strong>TerminalTool</strong>: công cụ command-line an toàn, hỗ trợ truy cập hệ thống file tức thời
4. <strong>Agent dài hạn</strong>: tích hợp ba công cụ lớn, xây dựng trợ lý bảo trì codebase xuyên phiên

### Thu hoạch cốt lõi

- <strong>Thiết kế phân tầng</strong>: truy cập tức thời (TerminalTool) + memory phiên (MemoryTool) + ghi chú bền vững (NoteTool)
- <strong>Sàng lọc thông minh</strong>: cơ chế chấm điểm dựa trên relevance và recency
- <strong>An toàn là trên hết</strong>: cơ chế bảo mật nhiều lớp đảm bảo hệ thống ổn định
- <strong>Hợp tác người-máy</strong>: cân bằng giữa tự động hóa và khả năng kiểm soát

Thông qua việc học chương này, bạn không chỉ nắm vững kỹ thuật cốt lõi của Context Engineering, mà quan trọng hơn là hiểu được cách xây dựng hệ thống Agent có thể duy trì tính nhất quán và hiệu quả trong phạm vi thời gian dài. Những kỹ năng này sẽ trở thành nền tảng quan trọng để bạn xây dựng các ứng dụng Agent cấp sản xuất.

Trong chương tiếp theo, chúng ta sẽ khám phá giao thức communication của Agent, tìm hiểu cách để Agent tương tác rộng rãi hơn với thế giới bên ngoài.

## Bài tập

> <strong>Gợi ý</strong>: một số bài tập không có đáp án chuẩn, trọng tâm là rèn luyện khả năng hiểu biết và thực hành tổng hợp của người học về Context Engineering và quản lý tác vụ dài hạn.

1. Chương này đã giới thiệu sự khác biệt giữa Context Engineering và Prompt Engineering. Hãy phân tích:

   - Ở mục 9.1 có đề cập "context phải được coi là một loại tài nguyên hữu hạn, và có lợi ích biên giảm dần". Hãy giải thích hiện tượng "context rot" là gì? Tại sao ngay cả khi model hỗ trợ context window 100K thậm chí 200K, chúng ta vẫn cần quản lý context một cách thận trọng?
   - Giả sử bạn cần xây dựng một "trợ lý code review", cần phân tích một codebase gồm 50 file. Hãy so sánh hai chiến lược: (1) nạp toàn bộ nội dung tất cả file vào context cùng lúc; (2) sử dụng context JIT (Just-in-time), truy xuất file theo nhu cầu thông qua tool. Hãy phân tích ưu nhược điểm và tình huống áp dụng của mỗi chiến lược.
   - Ở mục 9.2.1 có đề cập hai sai lầm cực đoan của system prompt: "hard-code quá mức" và "quá mơ hồ". Hãy đưa ra một ví dụ thực tế cho mỗi trường hợp, và giải thích cách tìm điểm cân bằng phù hợp.

2. Pipeline GSSC (Gather-Select-Structure-Compress) là kỹ thuật cốt lõi của chương này. Hãy suy nghĩ sâu hơn:

   > <strong>Gợi ý</strong>: đây là bài tập thực hành, khuyến khích thao tác thực tế

   - Trong triển khai ContextBuilder ở mục 9.3, bốn giai đoạn có trách nhiệm khác nhau. Hãy phân tích: nếu một giai đoạn nào đó bị lỗi (như giai đoạn Select chọn thông tin không liên quan, hoặc giai đoạn Compress nén quá mức dẫn đến mất thông tin), sẽ gây ảnh hưởng gì đến hiệu suất cuối cùng của Agent?
   - Dựa trên code ở mục 9.3.4, hãy thêm một chức năng "đánh giá chất lượng context" cho ContextBuilder: sau mỗi lần xây dựng context, tự động đánh giá mật độ thông tin, mức độ liên quan và tính toàn vẹn của context, và đưa ra đề xuất tối ưu hóa.
   - Giai đoạn "nén" trong pipeline GSSC sử dụng LLM để tóm tắt thông minh. Hãy suy nghĩ: trong trường hợp nào thì chiến lược cắt bớt (truncation) hoặc sliding window đơn giản có thể phù hợp hơn tóm tắt bằng LLM? Hãy thiết kế một chiến lược nén lai, kết hợp ưu điểm của nhiều phương pháp nén.

3. NoteTool và TerminalTool là các công cụ then chốt để hỗ trợ tác vụ dài hạn. Dựa trên nội dung mục 9.4 và 9.5, hãy hoàn thành các thực hành mở rộng sau:

   > <strong>Gợi ý</strong>: đây là bài tập thực hành, khuyến khích thao tác thực tế

   - NoteTool sử dụng hệ thống ghi chú phân tầng (ghi chú dự án, ghi chú tác vụ, ghi chú tạm thời). Hãy thiết kế một cơ chế "tự động sắp xếp ghi chú": khi ghi chú tạm thời tích lũy đến một số lượng nhất định, Agent có thể tự động phân tích các ghi chú này, nâng cấp thông tin quan trọng thành ghi chú tác vụ hoặc ghi chú dự án, và dọn dẹp nội dung dư thừa.
   - TerminalTool cung cấp khả năng thao tác hệ thống file, nhưng ở mục 9.5.2 đã nhấn mạnh thiết kế về tính bảo mật. Hãy phân tích: cơ chế bảo mật hiện tại (xác thực đường dẫn, danh sách trắng lệnh, kiểm tra quyền) có đủ không? Nếu Agent cần truy cập file nhạy cảm hoặc thực thi thao tác nguy hiểm, nên thiết kế quy trình "phê duyệt hợp tác người-máy" như thế nào?
   - Kết hợp NoteTool và TerminalTool, hãy thiết kế một "trợ lý refactor code thông minh": có thể phân tích cấu trúc codebase, ghi lại kế hoạch refactor, từng bước thực hiện thao tác refactor, và theo dõi tiến độ cũng như các vấn đề gặp phải trong ghi chú. Hãy vẽ sơ đồ luồng công việc hoàn chỉnh.

4. Trong case "quản lý tác vụ dài hạn" ở mục 9.6, chúng ta đã thấy giá trị của Context Engineering trong ứng dụng thực tế. Hãy phân tích sâu hơn:

   - Case này sử dụng chiến lược "quản lý context phân tầng": truy cập tức thời (TerminalTool) + memory phiên (MemoryTool) + ghi chú bền vững (NoteTool). Hãy phân tích: ba lớp này nên phối hợp với nhau như thế nào? Thông tin nào nên đặt ở lớp nào? Làm thế nào để tránh dư thừa và không nhất quán thông tin?
   - Giả sử quá trình thực thi tác vụ bị gián đoạn (như hệ thống crash, mất kết nối mạng), Agent cần khôi phục trạng thái từ ghi chú và tiếp tục thực thi. Hãy thiết kế một cơ chế "phục hồi từ điểm gián đoạn": làm thế nào để ghi lại đủ thông tin trạng thái trong ghi chú? Làm thế nào để xác minh trạng thái sau khi khôi phục là chính xác?
   - Tác vụ dài hạn thường liên quan đến việc thực thi song song hoặc tuần tự của nhiều tác vụ con. Hãy thiết kế một hệ thống "quản lý phụ thuộc tác vụ": có thể biểu đạt mối quan hệ phụ thuộc giữa các tác vụ (như "tác vụ B phải thực thi sau khi tác vụ A hoàn thành"), và tự động lập lịch thứ tự thực thi tác vụ. Hệ thống này nên tích hợp với NoteTool như thế nào?

5. Chương này nhiều lần đề cập đến khái niệm "progressive disclosure" (tiết lộ dần dần). Hãy suy nghĩ:

   - Ở mục 9.2.2, progressive disclosure được mô tả là "mỗi bước tương tác đều tạo ra context mới, ngược lại hướng dẫn quyết định của bước tiếp theo". Hãy thiết kế một tình huống ứng dụng cụ thể (như viết bài báo học thuật, debug vấn đề phức tạp), thể hiện cách progressive disclosure giúp Agent hoàn thành tác vụ hiệu quả hơn.
   - Một rủi ro tiềm ẩn của progressive disclosure là "hiệu quả khám phá thấp": Agent có thể lãng phí thời gian vào các chi tiết không quan trọng, hoặc bỏ lỡ thông tin then chốt. Hãy thiết kế một cơ chế "hướng dẫn khám phá": thông qua các quy tắc heuristic hoặc chiến lược metacognitive, giúp Agent quyết định thông minh hơn "bước tiếp theo nên khám phá gì".
   - So sánh "progressive disclosure" với cách tiếp cận truyền thống "nạp toàn bộ context một lần": trong loại tác vụ nào thì cách đầu có lợi thế rõ rệt? Trong loại tác vụ nào thì cách sau có thể phù hợp hơn? Hãy đưa ra ít nhất 3 ví dụ tác vụ thuộc các loại khác nhau.

## Tài liệu tham khảo

[1] Anthropic. Effective Context Engineering for AI Agents. `https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents`

[2] David Kim. Context-Engineering (GitHub). `https://github.com/davidkimai/Context-Engineering`
