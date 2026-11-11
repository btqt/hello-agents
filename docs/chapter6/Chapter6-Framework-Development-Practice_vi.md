# Chương 6 Thực hành phát triển Framework

Trong Chương 4, chúng ta đã viết code gốc (native code) để hiện thực hóa luồng làm việc cốt lõi của các loại Agent ReAct, Plan-and-Solve và Reflection. Quá trình này giúp chúng ta hiểu được logic thực thi nội tại của Agent. Sau đó, ở Chương 5, chúng ta chuyển sang góc nhìn của "người sử dụng", trải nghiệm sự tiện lợi và hiệu quả mà các nền tảng low-code mang lại.

Mục tiêu của chương này là khám phá cách sử dụng một số <strong>Agent Framework</strong> phổ biến trong ngành, để xây dựng các ứng dụng Agent đáng tin cậy một cách hiệu quả và có quy chuẩn. Trước tiên chúng ta sẽ tổng quan về các Agent Framework chủ đạo hiện có trên thị trường, sau đó thông qua các case thực hành hoàn chỉnh với một vài framework tiêu biểu, để trải nghiệm mô hình phát triển dựa trên framework (framework-driven development).

## 6.1 Từ triển khai thủ công đến phát triển bằng Framework

Từ việc viết các script dùng một lần đến việc sử dụng một framework trưởng thành, đó là một bước nhảy vọt quan trọng trong tư duy của lĩnh vực kỹ thuật phần mềm. Code chúng ta viết ở Chương 4 chủ yếu nhằm mục đích giảng dạy và giúp hiểu rõ bản chất. Chúng có thể hoàn thành tốt các nhiệm vụ cụ thể, nhưng nếu dùng chúng để xây dựng nhiều ứng dụng Agent khác loại và có logic phức tạp, chúng ta sẽ nhanh chóng gặp phải điểm nghẽn (bottleneck).

Bản chất của một framework là cung cấp một bộ "quy chuẩn" đã được kiểm chứng. Nó trừu tượng hóa (abstract) và đóng gói (encapsulate) những công việc chung, lặp đi lặp lại của tất cả các Agent (như vòng lặp chính, quản lý trạng thái, gọi tool, ghi log, v.v.), giúp chúng ta khi xây dựng một Agent mới có thể tập trung vào logic nghiệp vụ đặc thù của nó, thay vì phần triển khai nền tảng (underlying implementation) mang tính thông dụng.

### 6.1.1 Vì sao cần đến Agent Framework

Trước khi bắt đầu thực hành, trước tiên chúng ta cần làm rõ lý do tại sao phải sử dụng framework. So với việc tự viết độc lập một script Agent, giá trị của việc sử dụng framework chủ yếu được thể hiện ở những khía cạnh sau:

1. <strong>Nâng cao khả năng tái sử dụng code và hiệu quả phát triển</strong>: Đây là giá trị trực tiếp nhất. Một framework tốt sẽ cung cấp một lớp `Agent` cơ sở (base class) hoặc một executor mang tính thông dụng, nó đóng gói vòng lặp cốt lõi (Agent Loop) khi Agent vận hành. Dù là ReAct hay Plan-and-Solve, đều có thể nhanh chóng xây dựng dựa trên các thành phần chuẩn mà framework cung cấp, từ đó tránh được việc lặp lại công sức.
2. <strong>Thực hiện tách rời (decoupling) và khả năng mở rộng cho các thành phần cốt lõi</strong>: Một hệ thống Agent mạnh mẽ nên được cấu thành từ nhiều module lỏng lẻo kết nối với nhau (loosely coupled). Thiết kế của framework sẽ buộc chúng ta phải tách biệt các mối quan tâm khác nhau:
   - <strong>Tầng Model (Model Layer)</strong>: Chịu trách nhiệm tương tác với mô hình ngôn ngữ lớn, có thể dễ dàng thay thế các mô hình khác nhau (OpenAI, Anthropic, mô hình nội bộ (local model)).
   - <strong>Tầng Tool (Tool Layer)</strong>: Cung cấp giao diện định nghĩa, đăng ký và thực thi tool một cách chuẩn hóa, việc thêm tool mới sẽ không ảnh hưởng tới các đoạn code khác.
   - <strong>Tầng Memory (Memory Layer)</strong>: Xử lý bộ nhớ ngắn hạn và dài hạn, có thể chuyển đổi chiến lược memory khác nhau tùy theo nhu cầu (như sliding window, summary memory). Thiết kế theo module hóa (modular) này khiến toàn bộ hệ thống có khả năng mở rộng cực lớn, việc thay thế hoặc nâng cấp bất kỳ thành phần nào cũng trở nên đơn giản.
3. <strong>Chuẩn hóa việc quản lý trạng thái (state management) phức tạp</strong>: Lớp `Memory` mà chúng ta triển khai trong `ReflectionAgent` chỉ là một khởi đầu đơn giản. Trong các ứng dụng Agent chạy lâu dài, thực tế, quản lý trạng thái là một thách thức rất lớn, nó cần xử lý các vấn đề như giới hạn context window, lưu trữ bền vững (persistence) thông tin lịch sử, theo dõi trạng thái hội thoại nhiều lượt (multi-turn). Một framework có thể cung cấp một cơ chế quản lý trạng thái mạnh mẽ và mang tính thông dụng, giúp các nhà phát triển không cần phải xử lý lại những vấn đề phức tạp này mỗi lần.
4. <strong>Đơn giản hóa quá trình quan sát (observability) và debug</strong>: Khi hành vi của Agent trở nên phức tạp, việc hiểu được quá trình ra quyết định của nó trở nên vô cùng quan trọng. Một framework được thiết kế tỉ mỉ có thể tích hợp sẵn khả năng quan sát mạnh mẽ. Ví dụ, thông qua việc đưa vào cơ chế event callback (Callbacks), chúng ta có thể tự động kích hoạt việc ghi log hoặc báo cáo dữ liệu tại các điểm mốc quan trọng trong vòng đời của Agent (như `on_llm_start`, `on_tool_end`, `on_agent_finish`), từ đó dễ dàng theo dõi và debug toàn bộ quỹ đạo vận hành của Agent. Điều này hiệu quả và có hệ thống hơn nhiều so với việc tự thêm thủ công câu lệnh `print` trong code.

Vì vậy, việc chuyển từ triển khai thủ công sang phát triển bằng framework không chỉ là sự thay đổi trong cách tổ chức code, mà còn là con đường tất yếu để xây dựng các ứng dụng Agent phức tạp, đáng tin cậy và dễ bảo trì (maintainable).

### 6.1.2 Lựa chọn và so sánh các Framework chủ đạo

Hệ sinh thái các Agent Framework đang phát triển với tốc độ chưa từng có. Nếu như LangChain và LlamaIndex định hình nên mô hình (paradigm) của thế hệ framework ứng dụng LLM thông dụng đầu tiên, thì thế hệ framework mới hơn lại tập trung sâu hơn vào việc giải quyết các thách thức chuyên sâu trong các lĩnh vực cụ thể, đặc biệt là <strong>hợp tác đa Agent (Multi-Agent Collaboration)</strong> và <strong>điều khiển luồng công việc phức tạp (Complex Workflow Control)</strong>.

Trong phần thực hành tiếp theo của chương này, chúng ta sẽ tập trung vào bốn framework tiêu biểu nhất trong các lĩnh vực tiên phong này: AutoGen, AgentScope, CAMEL và LangGraph. Triết lý thiết kế của chúng hoàn toàn khác nhau, lần lượt đại diện cho các hướng kỹ thuật khác nhau để hiện thực hóa các hệ thống Agent phức tạp, như trong Bảng 6.1.

<div align="center">
  <p>Bảng 6.1 So sánh bốn Agent Framework</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/01.png" alt="" width="90%"/>
</div>


- <strong>AutoGen</strong>: Tư tưởng cốt lõi của AutoGen là hiện thực hóa sự hợp tác thông qua hội thoại (conversation)<sup>[1]</sup>. Nó trừu tượng hóa hệ thống đa Agent thành một nhóm chat (group chat) gồm nhiều Agent "có thể hội thoại" (conversable). Nhà phát triển có thể định nghĩa các vai trò (role) khác nhau (như `Coder`, `ProductManager`, `Tester`), và thiết lập quy tắc tương tác giữa chúng (ví dụ, sau khi `Coder` viết xong code thì `Tester` sẽ tự động tiếp quản). Quá trình giải quyết nhiệm vụ chính là quá trình các Agent này liên tục hội thoại, hợp tác, lặp lại trong nhóm chat thông qua việc truyền tin nhắn tự động hóa, cho đến khi đạt được mục tiêu cuối cùng.
- <strong>AgentScope</strong>: AgentScope là một nền tảng phát triển toàn diện được thiết kế chuyên biệt cho các ứng dụng đa Agent<sup>[2]</sup>. Đặc điểm cốt lõi của nó là <strong>tính dễ sử dụng</strong> và <strong>tính kỹ thuật hóa (engineering)</strong>. Nó cung cấp một giao diện lập trình (programming interface) rất thân thiện, giúp nhà phát triển dễ dàng định nghĩa Agent, xây dựng mạng lưới truyền thông (communication network), và quản lý toàn bộ vòng đời của ứng dụng. <strong>Cơ chế truyền tin nhắn (message passing)</strong> được tích hợp sẵn cùng khả năng hỗ trợ triển khai phân tán (distributed deployment), khiến nó rất phù hợp để xây dựng và vận hành các hệ thống đa Agent phức tạp, quy mô lớn.
- <strong>CAMEL</strong>: CAMEL cung cấp một phương pháp hợp tác mới mẻ có tên gọi <strong>đóng vai (Role-Playing)</strong><sup>[3]</sup>. Ý tưởng cốt lõi của nó là, chúng ta chỉ cần thiết lập vai trò riêng và mục tiêu nhiệm vụ chung cho hai Agent (ví dụ, `Nhà nghiên cứu AI` và `Lập trình viên Python`), chúng sẽ có thể tự chủ tiến hành hội thoại nhiều vòng dưới sự dẫn dắt của "<strong>Inception Prompting</strong>" (prompt khơi mào), khơi gợi và phối hợp lẫn nhau, cùng nhau hoàn thành nhiệm vụ. Điều này giảm đáng kể độ phức tạp trong việc thiết kế luồng hội thoại cho đa Agent.
- <strong>LangGraph</strong>: Là phần mở rộng của hệ sinh thái LangChain, LangGraph đi theo một hướng khác, mô hình hóa quy trình thực thi của Agent thành một <strong>đồ thị (Graph)</strong><sup>[4]</sup>. Trong cấu trúc chuỗi (chain) truyền thống, thông tin chỉ có thể chảy theo một chiều. Trong khi đó, LangGraph định nghĩa mỗi bước thao tác (như gọi LLM, thực thi tool) là một <strong>nút (Node)</strong> trong đồ thị, và dùng <strong>cạnh (Edge)</strong> để định nghĩa logic chuyển tiếp (jump) giữa các node. Thiết kế này vốn dĩ hỗ trợ <strong>vòng lặp (Cycles)</strong>, khiến việc hiện thực hóa các luồng công việc phức tạp mang tính lặp, sửa đổi và tự phản tư (self-reflection) như Reflection trở nên vô cùng đơn giản và trực quan.

Trong các mục tiếp theo, chúng ta sẽ trải nghiệm chuyên sâu mô hình phát triển dựa trên framework cho bốn framework này, thông qua các case thực hành hoàn chỉnh riêng biệt. <strong>Xin lưu ý</strong>, toàn bộ file mã nguồn (source file) của các dự án demo sẽ được đặt trong thư mục `code`, phần nội dung chính chỉ giải thích phần nguyên lý.

## 6.2 Framework một: AutoGen

Như đã đề cập ở trên, triết lý thiết kế của AutoGen bắt nguồn từ "hợp tác được thúc đẩy bởi hội thoại". Nó khéo léo ánh xạ quy trình giải quyết nhiệm vụ phức tạp thành một chuỗi hội thoại tự động hóa giữa các Agent với các vai trò khác nhau. Dựa trên ý tưởng cốt lõi này, framework AutoGen tiếp tục phát triển không ngừng. Chúng ta sẽ lấy phiên bản `0.7.4` làm ví dụ, vì đây là phiên bản mới nhất tính đến hiện tại, đại diện cho một lần tái cấu trúc kiến trúc (architecture refactoring) quan trọng, chuyển từ thiết kế kế thừa lớp (class inheritance) sang kiến trúc dạng tổ hợp (composition) linh hoạt hơn. Để hiểu sâu sắc và áp dụng framework này, trước tiên chúng ta cần giải thích các thành phần cấu thành cốt lõi nhất và cơ chế tương tác hội thoại ở tầng dưới.

### 6.2.1 Cơ chế cốt lõi của AutoGen

Việc phát hành phiên bản `0.7.4` là một cột mốc quan trọng trong sự phát triển của AutoGen, nó đánh dấu một cuộc cách tân căn bản trong thiết kế nền tảng (underlying design) của framework. Lần cập nhật này không đơn thuần là chồng thêm tính năng, mà là một sự suy nghĩ lại toàn diện về kiến trúc tổng thể, nhằm nâng cao tính module hóa, hiệu năng xử lý đồng thời (concurrency) và trải nghiệm của nhà phát triển.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/02.png" alt="" width="90%"/>
  <p>Hình 6.1 Sơ đồ kiến trúc AutoGen</p>
</div>

（1）Sự tiến hóa của cấu trúc framework

Như thể hiện ở Hình 6.1, thay đổi nổi bật nhất của kiến trúc mới là việc đưa vào triết lý thiết kế phân tầng (layering) rõ ràng và ưu tiên bất đồng bộ (async-first).

- <strong>Thiết kế phân tầng:</strong> Framework được tách thành hai module cốt lõi:
  - `autogen-core`: Là nền tảng tầng dưới của framework, đóng gói các chức năng cốt lõi như tương tác với mô hình ngôn ngữ, truyền tin nhắn. Sự tồn tại của nó đảm bảo tính ổn định và khả năng mở rộng trong tương lai của framework.
  - `autogen-agentchat`: Được xây dựng trên nền `core`, cung cấp giao diện cấp cao (high-level interface) dùng để phát triển các ứng dụng Agent dạng hội thoại, giúp đơn giản hóa quy trình phát triển ứng dụng đa Agent. Chiến lược phân tầng này khiến trách nhiệm của từng thành phần rõ ràng, giảm độ kết nối (coupling) của hệ thống.
- <strong>Ưu tiên bất đồng bộ:</strong> Kiến trúc mới chuyển hoàn toàn sang lập trình bất đồng bộ (`async/await`). Trong bối cảnh hợp tác đa Agent, các yêu cầu mạng (network request) (như gọi LLM API) là thao tác tốn thời gian chủ yếu. Mô hình bất đồng bộ cho phép hệ thống xử lý các tác vụ khác trong khi đang chờ phản hồi từ một Agent, từ đó tránh được việc chặn luồng (thread blocking), nâng cao đáng kể khả năng xử lý đồng thời và hiệu suất sử dụng tài nguyên của hệ thống.

（2）Các thành phần Agent cốt lõi

Agent là đơn vị cơ bản để thực thi nhiệm vụ. Trong phiên bản `0.7.4`, thiết kế của Agent trở nên tập trung và module hóa hơn.

- <strong>AssistantAgent (Agent trợ lý):</strong> Đây là người giải quyết nhiệm vụ chính, cốt lõi của nó là đóng gói một mô hình ngôn ngữ lớn (LLM). Nhiệm vụ của nó là tạo ra các phản hồi giàu logic và kiến thức dựa trên lịch sử hội thoại, ví dụ như đề xuất kế hoạch, viết bài hoặc viết code. Thông qua các System Message (thông điệp hệ thống) khác nhau, chúng ta có thể gán cho nó các vai trò "chuyên gia" khác nhau.
- <strong>UserProxyAgent (Agent đại diện người dùng):</strong> Đây là thành phần mang tính đặc trưng độc đáo trong AutoGen. Nó đóng vai trò kép: vừa là "người đại diện" cho người dùng, chịu trách nhiệm khởi tạo nhiệm vụ và truyền đạt ý định; vừa là một "bộ thực thi" (executor) đáng tin cậy, có thể được cấu hình để thực thi code hoặc gọi tool, và phản hồi kết quả cho các Agent khác. Thiết kế này phân tách rõ ràng giữa "suy nghĩ" (do `AssistantAgent` đảm nhiệm) và "hành động".

（3）Từ GroupChatManager đến Team

Khi nhiệm vụ cần nhiều Agent hợp tác, cần có một cơ chế để điều phối luồng hội thoại. Trong các phiên bản trước đây, `GroupChatManager` đảm nhận trách nhiệm này. Còn trong kiến trúc mới, khái niệm `Team` hay nhóm chat linh hoạt hơn đã được đưa vào, ví dụ như `RoundRobinGroupChat`.

- <strong>Nhóm chat quay vòng (RoundRobinGroupChat):</strong> Đây là một cơ chế điều phối hội thoại rõ ràng, theo trình tự (sequential). Nó sẽ khiến các Agent tham gia phát biểu lần lượt theo một thứ tự được định nghĩa trước. Mô hình này rất phù hợp với các nhiệm vụ có quy trình cố định, ví dụ một quy trình phát triển phần mềm điển hình: Product Manager đề xuất yêu cầu trước, sau đó Engineer viết code, cuối cùng Code Reviewer tiến hành kiểm tra.
- <strong>Luồng công việc:</strong>
  1. Đầu tiên, tạo một instance `RoundRobinGroupChat`, và thêm tất cả các Agent tham gia hợp tác (như Product Manager, Engineer, v.v.) vào đó.
  2. Khi một nhiệm vụ bắt đầu, nhóm chat sẽ lần lượt kích hoạt các Agent tương ứng theo thứ tự được thiết lập trước.
  3. Agent được chọn phản hồi dựa trên ngữ cảnh (context) hội thoại hiện tại.
  4. Nhóm chat sẽ thêm câu trả lời mới vào lịch sử hội thoại, và kích hoạt Agent tiếp theo.
  5. Quá trình này tiếp diễn cho tới khi đạt số vòng hội thoại tối đa (max turns) hoặc thỏa mãn điều kiện dừng (termination condition) được thiết lập trước.

Thông qua cách này, AutoGen đã đơn giản hóa mối quan hệ hợp tác phức tạp thành một "cuộc họp bàn tròn" tự động hóa, luồng công việc rõ ràng và dễ quản lý. Nhà phát triển chỉ cần định nghĩa vai trò và thứ tự phát biểu của từng thành viên trong nhóm, phần còn lại của luồng hợp tác sẽ được cơ chế nhóm chat tự động vận hành.

Trong mục tiếp theo, chúng ta sẽ tự tay trải nghiệm cách định nghĩa các Agent với vai trò khác nhau dưới kiến trúc mới, và tổ chức chúng trong một nhóm chat được điều phối bởi `RoundRobinGroupChat`, để hợp tác hoàn thành một nhiệm vụ lập trình thực tế, thông qua việc xây dựng một ví dụ mô phỏng đội ngũ phát triển phần mềm.

### 6.2.2 Đội ngũ phát triển phần mềm

Sau khi đã hiểu các thành phần cốt lõi và cơ chế hội thoại của AutoGen, mục này sẽ thông qua một case thực hành hoàn chỉnh để trình bày cụ thể cách áp dụng các tính năng mới này. Chúng ta sẽ xây dựng một đội ngũ phát triển phần mềm mô phỏng, đội ngũ này gồm nhiều Agent có kỹ năng chuyên môn khác nhau, chúng sẽ hợp tác hoàn thành một nhiệm vụ phát triển phần mềm thực tế.

（1）Mục tiêu nghiệp vụ

Mục tiêu của chúng ta là phát triển một ứng dụng Web có chức năng rõ ràng: <strong>hiển thị giá Bitcoin hiện tại theo thời gian thực</strong>. Nhiệm vụ này tuy nhỏ nhưng bao quát đầy đủ các khâu điển hình của phát triển phần mềm: từ phân tích yêu cầu, lựa chọn công nghệ, triển khai code cho tới review code và kiểm thử (test) cuối cùng. Điều này khiến nó trở thành kịch bản lý tưởng để kiểm chứng luồng hợp tác tự động hóa của AutoGen.

（2）Vai trò trong đội ngũ Agent

Để mô phỏng quy trình phát triển phần mềm thực tế, chúng ta thiết kế bốn vai trò Agent với trách nhiệm rõ ràng:

- <strong>ProductManager (Product Manager):</strong> Chịu trách nhiệm chuyển hóa yêu cầu mơ hồ của người dùng thành kế hoạch phát triển rõ ràng, có thể thực thi.
- <strong>Engineer (Kỹ sư):</strong> Dựa trên kế hoạch phát triển, chịu trách nhiệm viết code ứng dụng cụ thể.
- <strong>CodeReviewer (Người review code):</strong> Chịu trách nhiệm review code mà Engineer gửi lên, đảm bảo chất lượng, khả năng đọc hiểu (readability) và tính vững chắc (robustness) của nó.
- <strong>UserProxy (Agent đại diện người dùng):</strong> Đại diện cho người dùng cuối, khởi tạo nhiệm vụ ban đầu, và chịu trách nhiệm thực thi cũng như xác minh code cuối cùng được bàn giao.

Việc phân chia vai trò này là một bước then chốt trong thiết kế hệ thống đa Agent, nó phân rã một nhiệm vụ phức tạp thành nhiều nhiệm vụ con được xử lý bởi các "chuyên gia" trong từng lĩnh vực.

### 6.2.3 Triển khai code cốt lõi

Dưới đây, chúng ta sẽ phân tích từng bước code cốt lõi của đội ngũ tự động hóa này.

（1）Cấu hình model client

Tất cả các Agent dựa trên LLM đều cần một model client để tương tác với mô hình ngôn ngữ. AutoGen `0.7.4` cung cấp `OpenAIChatCompletionClient` được chuẩn hóa, nó có thể dễ dàng kết nối với bất kỳ dịch vụ mô hình nào tương thích với tiêu chuẩn OpenAI API (bao gồm dịch vụ chính thức của OpenAI, Azure OpenAI và các dịch vụ mô hình nội bộ như Ollama).

Chúng ta tạo và cấu hình model client thông qua một hàm (function) độc lập, và quản lý API Key cùng địa chỉ dịch vụ (service URL) thông qua biến môi trường (environment variable), đây là một thực hành kỹ thuật (engineering practice) tốt, giúp tăng tính linh hoạt và bảo mật cho code.

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

def create_openai_model_client():
    """Tạo và cấu hình model client OpenAI"""
    return OpenAIChatCompletionClient(
        model=os.getenv("LLM_MODEL_ID", "gpt-4o"),
        api_key=os.getenv("LLM_API_KEY"),
        base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1")
    )
```

（2）Định nghĩa vai trò Agent

Cốt lõi của việc định nghĩa Agent nằm ở việc viết System Message (thông điệp hệ thống) chất lượng cao. System Message giống như "quy tắc hành vi" và "kho tri thức chuyên môn" được thiết lập cho Agent, nó quy định chính xác vai trò, trách nhiệm, quy trình làm việc, thậm chí cả cách Agent tương tác với các Agent khác. Một System Message được thiết kế tỉ mỉ là chìa khóa để đảm bảo hệ thống đa Agent có thể hợp tác hiệu quả, chính xác. Trong đội ngũ phát triển phần mềm của chúng ta, mỗi vai trò được tạo bởi một hàm độc lập để đóng gói định nghĩa của nó.

<strong>Product Manager (ProductManager)</strong>

Product Manager chịu trách nhiệm khởi động toàn bộ quy trình. System Message của nó không chỉ định nghĩa trách nhiệm, mà còn quy chuẩn hóa cấu trúc đầu ra của nó, đồng thời bao gồm chỉ dẫn rõ ràng để dẫn dắt hội thoại chuyển sang khâu tiếp theo (Engineer).

```python
def create_product_manager(model_client):
    """Tạo Agent Product Manager"""
    system_message = """Bạn là một Product Manager giàu kinh nghiệm, chuyên phụ trách phân tích yêu cầu và lập kế hoạch dự án cho các sản phẩm phần mềm.

Trách nhiệm cốt lõi của bạn bao gồm:
1. **Phân tích yêu cầu**: Hiểu sâu sắc nhu cầu người dùng, xác định các tính năng cốt lõi và điều kiện biên
2. **Lập kế hoạch kỹ thuật**: Xây dựng lộ trình triển khai kỹ thuật rõ ràng dựa trên yêu cầu
3. **Đánh giá rủi ro**: Nhận diện các rủi ro kỹ thuật tiềm ẩn và vấn đề về trải nghiệm người dùng
4. **Điều phối và giao tiếp**: Giao tiếp hiệu quả với các kỹ sư và thành viên khác trong nhóm

Khi nhận được nhiệm vụ phát triển, hãy phân tích theo cấu trúc sau:
1. Hiểu và phân tích yêu cầu
2. Phân chia module chức năng
3. Đề xuất lựa chọn công nghệ
4. Sắp xếp thứ tự ưu tiên triển khai
5. Định nghĩa tiêu chí nghiệm thu

Vui lòng phản hồi ngắn gọn, rõ ràng và sau khi hoàn tất phân tích hãy nói "Mời Engineer bắt đầu triển khai"."""

    return AssistantAgent(
        name="ProductManager",
        model_client=model_client,
        system_message=system_message,
    )
```

<strong>Engineer (Kỹ sư)</strong>

System Message của Engineer tập trung vào việc triển khai kỹ thuật. Nó liệt kê các sở trường kỹ thuật của Engineer, và quy định các bước hành động cụ thể sau khi nhận được nhiệm vụ, đồng thời cũng bao gồm chỉ dẫn dẫn dắt luồng công việc chuyển sang Code Reviewer.

```python
def create_engineer(model_client):
    """Tạo Agent kỹ sư phần mềm"""
    system_message = """Bạn là một kỹ sư phần mềm cấp cao, thành thạo phát triển Python và xây dựng ứng dụng Web.

Chuyên môn kỹ thuật của bạn bao gồm:
1. **Lập trình Python**: Thành thạo cú pháp Python và các best practice
2. **Phát triển Web**: Tinh thông các framework như Streamlit, Flask, Django
3. **Tích hợp API**: Có nhiều kinh nghiệm tích hợp API bên thứ ba
4. **Xử lý lỗi**: Chú trọng tính vững chắc (robustness) của code và xử lý ngoại lệ

Khi nhận được nhiệm vụ phát triển, vui lòng:
1. Phân tích kỹ các yêu cầu kỹ thuật
2. Lựa chọn giải pháp kỹ thuật phù hợp
3. Viết mã nguồn triển khai hoàn chỉnh
4. Thêm các chú thích (comment) và giải thích cần thiết
5. Xem xét các trường hợp biên (edge case) và xử lý ngoại lệ

Vui lòng cung cấp code hoàn chỉnh có thể chạy được, và sau khi hoàn thành hãy nói "Mời Code Reviewer kiểm tra"."""

    return AssistantAgent(
        name="Engineer",
        model_client=model_client,
        system_message=system_message,
    )
```

<strong>Code Reviewer (CodeReviewer)</strong>

Định nghĩa của Code Reviewer tập trung vào chất lượng, tính bảo mật và tính chuẩn mực của code. System Message của nó liệt kê chi tiết các trọng tâm và quy trình review, đảm bảo khâu kiểm soát chất lượng trước khi bàn giao code.

```python
def create_code_reviewer(model_client):
    """Tạo Agent Code Reviewer"""
    system_message = """Bạn là một chuyên gia review code giàu kinh nghiệm, tập trung vào chất lượng code và các best practice.

Trọng tâm kiểm tra của bạn bao gồm:
1. **Chất lượng code**: Kiểm tra khả năng đọc hiểu, khả năng bảo trì và hiệu năng của code
2. **Bảo mật**: Nhận diện các lỗ hổng bảo mật và điểm rủi ro tiềm ẩn
3. **Best Practice**: Đảm bảo code tuân thủ các tiêu chuẩn ngành và best practice
4. **Xử lý lỗi**: Xác minh tính đầy đủ và hợp lý của việc xử lý ngoại lệ

Quy trình review:
1. Đọc kỹ và hiểu logic của code
2. Kiểm tra quy chuẩn code và các best practice
3. Nhận diện các vấn đề tiềm ẩn và điểm cần cải thiện
4. Đưa ra các đề xuất chỉnh sửa cụ thể
5. Đánh giá chất lượng tổng thể của code

Vui lòng đưa ra ý kiến review cụ thể, sau khi hoàn tất hãy nói "Review code hoàn tất, mời User Proxy kiểm thử"."""

    return AssistantAgent(
        name="CodeReviewer",
        model_client=model_client,
        system_message=system_message,
    )
```

<strong>Agent đại diện người dùng (UserProxy)</strong>

`UserProxyAgent` là một Agent đặc biệt, nó không dựa vào LLM để phản hồi, mà đóng vai trò đại diện cho người dùng trong hệ thống. Trường `description` của nó mô tả rõ ràng trách nhiệm của nó, đặc biệt quan trọng là, nó chịu trách nhiệm phát ra lệnh `TERMINATE` sau khi nhiệm vụ hoàn thành cuối cùng, để kết thúc toàn bộ luồng hợp tác một cách bình thường.

```python
def create_user_proxy():
    """Tạo Agent đại diện người dùng (UserProxy)"""
    return UserProxyAgent(
        name="UserProxy",
        description="""Đại diện người dùng (User Proxy), đảm nhận các trách nhiệm sau:
1. Thay mặt người dùng đưa ra yêu cầu phát triển
2. Thực thi mã nguồn triển khai cuối cùng
3. Xác minh xem tính năng có đáp ứng kỳ vọng hay không
4. Cung cấp phản hồi và đề xuất từ phía người dùng

Sau khi hoàn tất kiểm thử vui lòng phản hồi TERMINATE.""",
    )
```

Thông qua bốn hàm định nghĩa độc lập này, chúng ta không chỉ xây dựng được một "đội ngũ ảo" đầy đủ chức năng, mà còn thể hiện rằng việc "prompt engineering" thông qua System Message chính là khâu cốt lõi để thiết kế một ứng dụng đa Agent hiệu quả.

（3）Định nghĩa luồng hợp tác nhóm

Trong case này, quy trình phát triển phần mềm tương đối cố định (yêu cầu -> code -> review -> test), do đó `RoundRobinGroupChat` (nhóm chat quay vòng) là lựa chọn lý tưởng. Chúng ta thêm bốn Agent vào danh sách người tham gia theo thứ tự logic nghiệp vụ.

```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# Định nghĩa quy tắc chat nhóm và hợp tác của đội ngũ
team_chat = RoundRobinGroupChat(
    participants=[
        product_manager,
        engineer,
        code_reviewer,
        user_proxy
    ],
    termination_condition=TextMentionTermination("TERMINATE"),
    max_turns=20,
)
```

- <strong>Thứ tự người tham gia:</strong> Thứ tự của danh sách `participants` quyết định trình tự phát biểu trước sau của các Agent.
- <strong>Điều kiện dừng:</strong> `termination_condition` là yếu tố then chốt kiểm soát thời điểm luồng hợp tác kết thúc. Ở đây chúng ta thiết lập rằng, khi bất kỳ tin nhắn nào chứa từ khóa "TERMINATE", hội thoại sẽ kết thúc. Trong thiết kế của chúng ta, lệnh này được phát ra bởi `UserProxy` sau khi hoàn tất kiểm thử cuối cùng.
- <strong>Số vòng tối đa:</strong> `max_turns` là một van an toàn (safety valve), dùng để ngăn hội thoại rơi vào vòng lặp vô hạn, tránh tiêu hao tài nguyên không cần thiết.

（4）Khởi động và vận hành

Do AutoGen `0.7.4` áp dụng kiến trúc bất đồng bộ, toàn bộ việc khởi động và vận hành của luồng hợp tác đều được hoàn thành trong một hàm bất đồng bộ (async function), và cuối cùng được thực thi thông qua `asyncio.run()`.

```python
async def run_software_development_team():
    # ... Khởi tạo client và các Agent ...
    
    # Định nghĩa mô tả nhiệm vụ
    task = """Chúng ta cần phát triển một ứng dụng hiển thị giá Bitcoin với các yêu cầu cụ thể sau:
            Chức năng cốt lõi:
            - Hiển thị giá Bitcoin hiện tại theo thời gian thực (USD)
            - Hiển thị xu hướng biến động giá trong 24 giờ (tỷ lệ phần trăm và mức tăng/giảm)
            - Cung cấp tính năng làm mới (refresh) giá

            Yêu cầu kỹ thuật:
            - Sử dụng framework Streamlit để tạo ứng dụng Web
            - Giao diện đơn giản, đẹp mắt, thân thiện với người dùng
            - Bổ sung xử lý lỗi và trạng thái đang tải (loading) phù hợp

            Vui lòng phối hợp cùng nhau để hoàn thành nhiệm vụ này, từ bước phân tích yêu cầu cho đến triển khai cuối cùng."""
    
    # Thực thi bất đồng bộ quá trình hợp tác của đội ngũ và xuất luồng (stream) hội thoại
    result = await Console(team_chat.run_stream(task=task))
    return result

# Điểm vào (entry point) chương trình chính
if __name__ == "__main__":
    result = asyncio.run(run_software_development_team())
```

Khi chương trình được chạy, `task` được truyền vào `team_chat` như một tin nhắn khởi đầu, Product Manager với tư cách người tham gia đầu tiên sẽ nhận được tin nhắn này, sau đó toàn bộ luồng hợp tác tự động hóa bắt đầu diễn ra.

（5）Hiệu quả hợp tác dự kiến

Khi chúng ta chạy đội ngũ phát triển phần mềm này, có thể quan sát được một luồng hợp tác hoàn chỉnh:

```bash
🔧 Đang khởi tạo model client...
👥 Đang tạo đội ngũ Agent...
🚀 Khởi động hợp tác đội ngũ phát triển phần mềm AutoGen...
============================================================
---------- TextMessage (user) ----------
Chúng ta cần phát triển một ứng dụng hiển thị giá Bitcoin với các yêu cầu cụ thể sau:
...
Vui lòng phối hợp cùng nhau để hoàn thành nhiệm vụ này, từ bước phân tích yêu cầu cho đến triển khai cuối cùng.
---------- TextMessage (ProductManager) ----------
### 1. Hiểu và phân tích yêu cầu
...
Mời Engineer bắt đầu triển khai.
---------- TextMessage (Engineer) ----------
### Triển khai giải pháp kỹ thuật
...
Mời Code Reviewer kiểm tra.
---------- TextMessage (CodeReviewer) ----------
### Review code
...
Review code hoàn tất, mời User Proxy kiểm thử.
---------- TextMessage (UserProxy) ----------
Đã hoàn thành yêu cầu
---------- TextMessage (ProductManager) ----------
Tuyệt vời, cảm ơn phản hồi của bạn! Nếu có bất kỳ vấn đề nào trong quá trình sử dụng, hoặc có các yêu cầu tính năng và đề xuất cải tiến khác, xin vui lòng cho chúng tôi biết bất cứ lúc nào. Chúng tôi sẽ tiếp tục hỗ trợ và cải thiện. Chúc bạn có trải nghiệm thú vị với ứng dụng của chúng tôi!
---------- TextMessage (Engineer) ----------
Rất vui khi nghe dự án đã hoàn thành thuận lợi. Nếu bạn hoặc người dùng có bất kỳ câu hỏi nào hoặc cần hỗ trợ, xin vui lòng liên hệ với chúng tôi bất cứ lúc nào. Cảm ơn sự ủng hộ của bạn đối với công việc của chúng tôi, hãy cùng nhau đảm bảo ứng dụng vận hành ổn định và không ngừng tối ưu hóa trải nghiệm người dùng!
---------- TextMessage (CodeReviewer) ----------
Cảm ơn rất nhiều vì sự nỗ lực và hợp tác của mọi người để dự án có thể hoàn thành thuận lợi. Trong tương lai, nếu có thêm nhu cầu hỗ trợ kỹ thuật hoặc điểm nào cần cải thiện, chúng tôi sẵn sàng đóng góp để tiếp tục tối ưu hóa dự án. Mong rằng người dùng sẽ có trải nghiệm mượt mà, đồng thời rất hoan nghênh các phản hồi và góp ý thêm. Một lần nữa cảm ơn sự hợp tác của cả đội ngũ!
---------- TextMessage (UserProxy) ----------
Enter your response: TERMINATE
============================================================
✅ Hợp tác đội ngũ hoàn tất!

📋 Tóm tắt kết quả hợp tác:
- Số lượng Agent tham gia: 4
- Trạng thái hoàn thành nhiệm vụ: Thành công
```

Toàn bộ quá trình hợp tác thể hiện rõ ưu điểm của framework AutoGen: <strong>hợp tác tự nhiên được thúc đẩy bởi hội thoại</strong>, <strong>phân công chuyên môn hóa vai trò</strong>, <strong>quản lý tự động hóa quy trình</strong> và <strong>vòng khép kín phát triển hoàn chỉnh</strong>.

### 6.2.4 Phân tích ưu điểm và hạn chế của AutoGen

Bất kỳ framework kỹ thuật nào cũng có kịch bản áp dụng đặc thù và sự đánh đổi (trade-off) trong thiết kế của riêng nó. Trong mục này, chúng ta sẽ phân tích một cách khách quan ưu điểm cốt lõi của AutoGen cũng như những hạn chế và thách thức mà nó có thể gặp phải trong ứng dụng thực tế.

（1）Ưu điểm

- Như case đã thể hiện, chúng ta không cần phải thiết kế state machine hay logic kiểm soát luồng phức tạp cho đội ngũ Agent, mà ánh xạ một cách tự nhiên toàn bộ quy trình phát triển phần mềm thành hội thoại giữa Product Manager, Engineer và Reviewer. Cách làm này gần gũi hơn với mô hình hợp tác của đội ngũ con người, giảm đáng kể ngưỡng (threshold) để mô hình hóa các nhiệm vụ phức tạp. Nhà phát triển có thể tập trung nhiều tâm sức hơn vào việc định nghĩa "ai (vai trò)" và "làm gì (trách nhiệm)", thay vì "làm như thế nào (kiểm soát quy trình)".
- Framework cho phép gán vai trò chuyên môn hóa cao cho mỗi Agent thông qua System Message. Trong case này, `ProductManager` tập trung vào yêu cầu, còn `CodeReviewer` thì tập trung vào chất lượng. Một Agent được thiết kế tỉ mỉ có thể được tái sử dụng trong nhiều dự án khác nhau, dễ bảo trì và mở rộng.
- Đối với các nhiệm vụ theo quy trình, cơ chế như `RoundRobinGroupChat` cung cấp một luồng hợp tác rõ ràng, có thể dự đoán được. Đồng thời, thiết kế của `UserProxyAgent` cung cấp một giao diện tự nhiên cho "con người tham gia vòng lặp" (Human-in-the-loop). Nó vừa có thể là người khởi tạo nhiệm vụ, vừa có thể là người giám sát quy trình và người nghiệm thu cuối cùng. Thiết kế này đảm bảo hệ thống tự động hóa luôn nằm dưới sự giám sát của con người.

（2）Hạn chế

- Mặc dù `RoundRobinGroupChat` cung cấp một quy trình theo trình tự, nhưng hội thoại dựa trên LLM về bản chất mang tính bất định (uncertainty). Agent có thể tạo ra phản hồi lệch khỏi kỳ vọng, khiến hội thoại đi vào các nhánh ngoài ý muốn, thậm chí rơi vào vòng lặp.
- Khi kết quả làm việc của đội ngũ Agent không đạt kỳ vọng, quá trình debug có thể rất khó khăn. Khác với chương trình truyền thống, những gì chúng ta nhận được không phải là một stack trace lỗi rõ ràng, mà là một chuỗi dài lịch sử hội thoại. Đây được gọi là bài toán khó "debug kiểu hội thoại".

（3）Bổ sung cấu hình cho các mô hình không phải của OpenAI

Nếu bạn muốn sử dụng các mô hình không thuộc dòng OpenAI (như DeepSeek, Qwen, v.v.), trong phiên bản 0.7.4 cần truyền thêm dictionary thông tin mô hình vào tham số của `OpenAIChatCompletionClient`. Lấy DeepSeek làm ví dụ:

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    model_info={
        "function_calling": True,
        "max_tokens": 4096,
        "context_length": 32768,
        "vision": False,
        "json_output": True,
        "family": "deepseek",
        "structured_output": True,
    }
)
```

Dictionary `model_info` này giúp AutoGen hiểu được ranh giới năng lực (capability boundary) của mô hình, từ đó thích ứng tốt hơn với các dịch vụ mô hình khác nhau.



## 6.3 Framework hai: AgentScope

Nếu như triết lý thiết kế của AutoGen là "hợp tác được thúc đẩy bởi hội thoại", thì AgentScope lại đại diện cho một hướng kỹ thuật khác: <strong>nền tảng đa Agent ưu tiên kỹ thuật hóa (engineering-first)</strong>. AgentScope được phát triển bởi Alibaba DAMO Academy, được thiết kế chuyên biệt để xây dựng các ứng dụng đa Agent quy mô lớn, độ tin cậy cao. Nó không chỉ cung cấp một giao diện lập trình trực quan, dễ sử dụng, mà quan trọng hơn là đã tích hợp sẵn các đặc tính cấp doanh nghiệp (enterprise-grade) như triển khai phân tán, khả năng khôi phục khi lỗi (fault recovery), khả năng quan sát (observability), khiến nó đặc biệt phù hợp để xây dựng các ứng dụng cần vận hành ổn định lâu dài trong môi trường production.

### 6.3.1 Thiết kế của AgentScope

So với AutoGen, điểm khác biệt cốt lõi của AgentScope nằm ở <strong>thiết kế kiến trúc hướng tin nhắn (message-driven)</strong> và <strong>thực hành kỹ thuật ở cấp độ công nghiệp</strong>. Nếu AutoGen giống như một "studio hội thoại" linh hoạt, thì AgentScope chính là một "hệ điều hành Agent" hoàn chỉnh, cung cấp cho nhà phát triển sự hỗ trợ xuyên suốt toàn bộ vòng đời từ phát triển, kiểm thử cho tới triển khai. Khác với thiết kế kiểu kế thừa (inheritance) mà nhiều framework áp dụng, AgentScope lựa chọn <strong>kiến trúc tổ hợp (composition)</strong> và <strong>mô hình hướng tin nhắn (message-driven)</strong>. Thiết kế này không chỉ tăng cường mức độ module hóa của hệ thống, mà còn đặt nền móng cho hiệu năng xử lý đồng thời xuất sắc và khả năng phân tán của nó.

（1）Hệ thống kiến trúc phân tầng

Như Hình 6.2 thể hiện, AgentScope áp dụng thiết kế phân tầng module hóa rõ ràng, từ các thành phần cơ sở ở tầng dưới cho tới việc điều phối (orchestration) ứng dụng ở tầng trên, hình thành nên một hệ sinh thái phát triển Agent hoàn chỉnh.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/03.png" alt="" width="90%"/>
  <p>Hình 6.2 Sơ đồ kiến trúc AgentScope</p>
</div>

Trong kiến trúc này, tầng thấp nhất là <strong>tầng thành phần cơ sở (Foundational Components)</strong>, nó cung cấp các khối xây dựng (building block) cốt lõi cho toàn bộ framework. Thành phần `Message` định nghĩa một định dạng tin nhắn thống nhất, hỗ trợ từ tương tác văn bản đơn giản cho tới nội dung đa phương thức (multimodal) phức tạp; thành phần `Memory` cung cấp khả năng quản lý bộ nhớ ngắn hạn và dài hạn; tầng `Model API` trừu tượng hóa việc gọi các mô hình ngôn ngữ lớn khác nhau; còn thành phần `Tool` thì đóng gói khả năng tương tác của Agent với thế giới bên ngoài.

Phía trên các thành phần cơ sở, <strong>tầng hạ tầng cấp Agent (Agent-level Infrastructure)</strong> cung cấp các mức trừu tượng cao hơn. Tầng này không chỉ bao gồm nhiều Agent được xây dựng sẵn (prebuilt) (như Agent duyệt web, Agent nghiên cứu chuyên sâu), mà còn hiện thực hóa mô hình ReAct kinh điển, hỗ trợ agent hook, gọi tool song song (parallel tool calling), quản lý trạng thái và các tính năng cao cấp khác. Đặc biệt đáng chú ý là, tầng này hỗ trợ nguyên bản (native) <strong>thực thi bất đồng bộ và điều khiển theo thời gian thực</strong>, đây là một ưu thế quan trọng của AgentScope so với các framework khác.

<strong>Tầng hợp tác đa Agent (Multi-Agent Cooperation)</strong> là nơi đặt sự sáng tạo cốt lõi của AgentScope. `MsgHub` với vai trò là trung tâm tin nhắn, chịu trách nhiệm định tuyến (routing) tin nhắn giữa các Agent và quản lý trạng thái; còn hệ thống `Pipeline` thì cung cấp khả năng điều phối luồng công việc linh hoạt, hỗ trợ nhiều chế độ thực thi như tuần tự, đồng thời. Thiết kế này giúp nhà phát triển dễ dàng xây dựng các kịch bản hợp tác đa Agent phức tạp.

<strong>Tầng phát triển và triển khai (Deployment & Development)</strong> ở trên cùng thể hiện sự coi trọng của AgentScope đối với việc kỹ thuật hóa. `AgentScope Runtime` cung cấp một môi trường runtime cấp production, còn `AgentScope Studio` thì cung cấp cho nhà phát triển một bộ công cụ phát triển trực quan (visualization) hoàn chỉnh.

（2）Hướng tin nhắn (Message-driven)

Sự sáng tạo cốt lõi của AgentScope nằm ở <strong>kiến trúc hướng tin nhắn (message-driven architecture)</strong> của nó. Trong kiến trúc này, toàn bộ tương tác giữa các Agent được trừu tượng hóa thành việc gửi và nhận <strong>tin nhắn (message)</strong>, thay vì lời gọi hàm (function call) truyền thống.

```python
from agentscope.message import Msg

# Cấu trúc chuẩn của tin nhắn
message = Msg(
    name="Alice",           # Tên người gửi
    content="Hello, Bob!",  # Nội dung tin nhắn
    role="user",            # Loại vai trò
    metadata={              # Thông tin metadata
        "timestamp": "2024-01-15T10:30:00Z",
        "message_type": "text",
        "priority": "normal"
    }
)
```

Việc coi tin nhắn là đơn vị cơ bản của tương tác mang lại một vài ưu điểm then chốt:

- <strong>Tách rời bất đồng bộ (asynchronous decoupling)</strong>: Bên gửi và bên nhận tin nhắn được tách rời về mặt thời gian, không cần phải chờ đợi lẫn nhau, vốn dĩ hỗ trợ tốt các kịch bản xử lý đồng thời cao.
- <strong>Trong suốt về vị trí (location transparency)</strong>: Agent không cần quan tâm liệu một Agent khác đang ở tiến trình (process) nội bộ hay trên server từ xa, hệ thống tin nhắn sẽ tự động xử lý việc định tuyến.
- <strong>Khả năng quan sát (observability)</strong>: Mỗi tin nhắn đều có thể được ghi lại, truy vết (trace) và phân tích, giúp đơn giản hóa đáng kể việc debug và giám sát các hệ thống phức tạp.
- <strong>Độ tin cậy (reliability)</strong>: Tin nhắn có thể được lưu trữ bền vững và thử lại (retry), ngay cả khi hệ thống gặp sự cố, vẫn có thể đảm bảo tính nhất quán cuối cùng (eventual consistency) của tương tác, nâng cao khả năng chịu lỗi của hệ thống.

（3）Quản lý vòng đời Agent

Trong AgentScope, mỗi Agent đều có vòng đời rõ ràng (khởi tạo, chạy, tạm dừng, hủy, v.v.), và được hiện thực hóa dựa trên một lớp cơ sở (base class) thống nhất là `AgentBase`. Nhà phát triển thường chỉ cần quan tâm đến phương thức `reply` cốt lõi của nó.

```python
from agentscope.agents import AgentBase

class CustomAgent(AgentBase):
    def __init__(self, name: str, **kwargs):
        super().__init__(name=name, **kwargs)
        # Logic khởi tạo Agent
    
    def reply(self, x: Msg) -> Msg:
        # Logic phản hồi cốt lõi của Agent
        response = self.model(x.content)
        return Msg(name=self.name, content=response, role="assistant")
    
    def observe(self, x: Msg) -> None:
        # Logic quan sát của Agent (tùy chọn)
        self.memory.add(x)
```

Mô hình thiết kế này tách biệt logic nội bộ của Agent khỏi việc giao tiếp (communication) bên ngoài, nhà phát triển chỉ cần định nghĩa cách Agent "suy nghĩ và phản hồi" trong phương thức `reply` là đủ.

（4）Cơ chế truyền tin nhắn

AgentScope tích hợp sẵn một <strong>trung tâm tin nhắn (MsgHub)</strong>, đây là bộ phận trung tâm của toàn bộ kiến trúc hướng tin nhắn. MsgHub không chỉ chịu trách nhiệm định tuyến và phân phối tin nhắn, mà còn tích hợp các chức năng cao cấp như lưu trữ bền vững và giao tiếp phân tán, nó có những đặc điểm sau.

- <strong>Định tuyến tin nhắn linh hoạt</strong>: Hỗ trợ nhiều phương thức giao tiếp như point-to-point, broadcast, multicast, có thể xây dựng mạng lưới tương tác linh hoạt, phức tạp.
- <strong>Lưu trữ bền vững tin nhắn (message persistence)</strong>: Có khả năng tự động lưu toàn bộ tin nhắn vào database (như SQLite, MongoDB), đảm bảo trạng thái của các nhiệm vụ chạy dài hạn có thể được khôi phục.
- <strong>Hỗ trợ phân tán nguyên bản (native distributed support)</strong>: Đây là tính năng mang tính biểu tượng của AgentScope. Agent có thể được triển khai trên các tiến trình hoặc server khác nhau, `MsgHub` sẽ tự động xử lý việc giao tiếp xuyên node (cross-node) thông qua RPC (Remote Procedure Call), hoàn toàn trong suốt với nhà phát triển.

Những khả năng kỹ thuật hóa được cung cấp bởi kiến trúc tầng dưới này, khiến AgentScope có ưu thế hơn so với các framework hướng hội thoại truyền thống khi xử lý các kịch bản ứng dụng phức tạp cần xử lý đồng thời cao, độ tin cậy cao. Tất nhiên, điều này cũng đòi hỏi nhà phát triển phải hiểu và thích ứng với mô hình lập trình bất đồng bộ hướng tin nhắn.

Trong mục tiếp theo, chúng ta sẽ thông qua một case thực hành cụ thể, trò chơi Ma sói Tam Quốc, để trải nghiệm chuyên sâu năng lực của framework AgentScope, đặc biệt là ưu thế của nó trong việc xử lý tương tác đồng thời.

### 6.3.2 Trò chơi Ma sói Tam Quốc

Để hiểu sâu sắc kiến trúc hướng tin nhắn và năng lực hợp tác đa Agent của AgentScope, chúng ta sẽ xây dựng trò chơi "Ma sói Tam Quốc" kết hợp yếu tố văn hóa cổ điển Trung Hoa. Case này không chỉ thể hiện ưu thế của AgentScope trong việc xử lý tương tác đa Agent phức tạp, mà quan trọng hơn, nó minh họa cách phát huy sức mạnh của kiến trúc hướng tin nhắn trong một kịch bản cần <strong>hợp tác thời gian thực</strong>, <strong>đóng vai (role-playing)</strong> và <strong>đấu trí chiến lược (strategic game)</strong>. Khác với Ma sói truyền thống, "Ma sói Tam Quốc" của chúng ta đưa các nhân vật kinh điển như Lưu Bị, Quan Vũ, Gia Cát Lượng vào trò chơi, mỗi Agent không chỉ cần hoàn thành nhiệm vụ cơ bản của Ma sói (như sói giết người, tiên tri kiểm tra, dân làng suy luận), mà còn phải thể hiện được đặc điểm tính cách và mô thức hành vi của nhân vật Tam Quốc tương ứng. Thiết kế này cho phép chúng ta quan sát được cách AgentScope xử lý <strong>việc mô hình hóa vai trò đa tầng</strong>.

（1）Thiết kế kiến trúc và các thành phần cốt lõi

Thiết kế hệ thống của case này tuân theo nguyên tắc phân tầng tách rời (decoupling), chia logic trò chơi thành ba tầng độc lập, mỗi tầng ánh xạ tới một hoặc nhiều thành phần cốt lõi của AgentScope:

- <strong>Tầng điều khiển trò chơi (Game Control Layer)</strong>: Lớp `ThreeKingdomsWerewolfGame` đóng vai trò là bộ điều khiển chính của trò chơi, chịu trách nhiệm duy trì trạng thái toàn cục (như danh sách người chơi còn sống, giai đoạn hiện tại của trò chơi), thúc đẩy luồng trò chơi (gọi giai đoạn ban đêm, giai đoạn ban ngày) cũng như phân định thắng thua.
- <strong>Tầng tương tác Agent (Agent Interaction Layer)</strong>: Hoàn toàn được điều khiển bởi `MsgHub`. Toàn bộ giao tiếp giữa các Agent, dù là sự thương lượng bí mật giữa những người sói hay cuộc tranh luận công khai vào ban ngày, đều được định tuyến và phân phối thông qua trung tâm tin nhắn.
- <strong>Tầng mô hình hóa vai trò (Role Modeling Layer)</strong>: Mỗi người chơi là một instance dựa trên `DialogAgent`. Chúng ta thông qua các system prompt được thiết kế tỉ mỉ, để gán cho mỗi Agent một danh tính kép gồm "vai trò trong trò chơi" và "nhân cách Tam Quốc".

（2）Luồng trò chơi hướng tin nhắn

Thiết kế cốt lõi nhất của case này là dùng <strong>hướng tin nhắn</strong> để thay thế <strong>state machine</strong> trong việc quản lý luồng trò chơi. Trong triển khai truyền thống, việc chuyển đổi giai đoạn trò chơi thường được điều khiển bởi một state machine tập trung (centralized). Còn trong mô hình của AgentScope, luồng trò chơi được mô hình hóa một cách tự nhiên thành một chuỗi các mẫu hình (pattern) tương tác tin nhắn được định nghĩa sẵn.

Ví dụ, việc hiện thực hóa giai đoạn sói không phải là một lời gọi hàm đơn giản, mà thông qua `MsgHub` để tạo động một kênh giao tiếp riêng tư tạm thời, chỉ bao gồm những người chơi sói:

```python
async def werewolf_phase(self, round_num: int):
    """Giai đoạn Ma sói - minh họa mô hình hợp tác hướng tin nhắn"""
    if not self.werewolves:
        return None
        
    # Thiết lập kênh giao tiếp dành riêng cho Ma sói thông qua trung tâm tin nhắn
    async with MsgHub(
        self.werewolves,
        enable_auto_broadcast=True,
        announcement=await self.moderator.announce(
            f"Các Ma sói, hãy thảo luận mục tiêu giết tối nay. Người chơi còn sống: {format_player_list(self.alive_players)}"
        ),
    ) as werewolves_hub:
        # Giai đoạn thảo luận: Ma sói trao đổi chiến lược thông qua tin nhắn
        for _ in range(MAX_DISCUSSION_ROUND):
            for wolf in self.werewolves:
                await wolf(structured_model=DiscussionModelCN)
        
        # Giai đoạn bỏ phiếu: thu thập và thống kê quyết định giết của Ma sói
        werewolves_hub.set_auto_broadcast(False)
        kill_votes = await fanout_pipeline(
            self.werewolves,
            msg=await self.moderator.announce("Vui lòng chọn mục tiêu cần giết"),
            structured_model=WerewolfKillModelCN,
            enable_gather=False,
        )
```

Ưu điểm của thiết kế này là, logic trò chơi được biểu đạt một cách rõ ràng là "trong ngữ cảnh cụ thể, tiến hành trao đổi tin nhắn theo mô thức nào", thay vì một chuỗi chuyển đổi trạng thái cứng nhắc. Các giai đoạn như thảo luận ban ngày (broadcast toàn bộ), tiên tri kiểm tra (yêu cầu point-to-point) cũng đều tuân theo cùng một mô hình thiết kế.

（3）Dùng Structured Output để ràng buộc luật chơi

Một thách thức then chốt của trò chơi Ma sói là làm sao đảm bảo hành vi của Agent tuân thủ luật chơi. <strong>Cơ chế Structured Output</strong> của AgentScope cung cấp giải pháp cho vấn đề này. Chúng ta định nghĩa các data model (mô hình dữ liệu) nghiêm ngặt cho các hành vi trò chơi khác nhau:

```python
class DiscussionModelCN(BaseModel):
    """Định dạng đầu ra của giai đoạn thảo luận"""
    reach_agreement: bool = Field(
        description="Đã đạt được sự thống nhất ý kiến hay chưa",
        default=False
    )
    confidence_level: int = Field(
        description="Mức độ tự tin vào suy luận hiện tại (1-10)",
        ge=1, le=10,
        default=5
    )
    key_evidence: Optional[str] = Field(
        description="Bằng chứng then chốt ủng hộ quan điểm của bạn",
        default=None
    )

class WitchActionModelCN(BaseModel):
    """Định dạng đầu ra cho hành động của Phù thủy"""
    use_antidote: bool = Field(description="Có sử dụng thuốc giải hay không")
    use_poison: bool = Field(description="Có sử dụng thuốc độc hay không")
    target_name: Optional[str] = Field(description="Tên người chơi mục tiêu của thuốc độc")
```

Thông qua cách này, chúng ta không chỉ đảm bảo được <strong>tính nhất quán về định dạng (format consistency)</strong> trong đầu ra của Agent, mà quan trọng hơn là hiện thực hóa được <strong>việc ràng buộc tự động hóa luật chơi</strong>. Ví dụ, Agent phù thủy không thể vừa dùng thuốc giải vừa dùng thuốc độc cho cùng một mục tiêu trong cùng lúc, tiên tri mỗi đêm chỉ có thể kiểm tra một người chơi, những ràng buộc này đều được tự động thực thi thông qua định nghĩa trường (field) của data model và logic kiểm chứng (validation).

（4）Thách thức kép trong việc mô hình hóa vai trò

Trong case này, thách thức kỹ thuật thú vị nhất là làm sao để Agent có thể đóng tốt vai trò ở cả hai tầng: <strong>vai trò chức năng trong trò chơi</strong> (sói, tiên tri, v.v.) và <strong>vai trò nhân cách văn hóa</strong> (Lưu Bị, Tào Tháo, v.v.). Chúng ta giải quyết vấn đề này thông qua prompt engineering:

```python
def get_role_prompt(role: str, character: str) -> str:
    """Lấy prompt vai trò - kết hợp luật chơi và tính cách nhân vật"""
    base_prompt = f"""Bạn là {character}, đóng vai {role} trong trò chơi Ma sói Tam Quốc này.

Quy tắc quan trọng:
1. Bạn chỉ có thể tham gia trò chơi thông qua hội thoại và suy luận
2. Không cố gắng gọi bất kỳ công cụ hay hàm bên ngoài nào
3. Trả lời nghiêm ngặt theo đúng định dạng JSON được yêu cầu

Đặc điểm vai trò:
"""
    
    if role == "Ma sói":
        return base_prompt + f"""
- Bạn thuộc phe Ma sói, mục tiêu là tiêu diệt toàn bộ người tốt
- Ban đêm có thể thương lượng với các Ma sói khác về mục tiêu giết
- Ban ngày phải che giấu thân phận, đánh lạc hướng người tốt
- Nói chuyện và hành động theo đúng tính cách của {character}
"""
```

Thiết kế này giúp chúng ta quan sát được một hiện tượng thú vị: các nhân vật Tam Quốc khác nhau khi đóng cùng một vai trò trong trò chơi sẽ thể hiện chiến lược và phong cách lời nói hoàn toàn khác nhau. Ví dụ, "Tào Tháo" khi đóng vai sói có thể tỏ ra xảo quyệt và giỏi ngụy trang hơn, còn "Trương Phi" khi đóng vai sói thì có thể tỏ ra trực diện và bốc đồng hơn.

（5）Xử lý đồng thời và cơ chế chịu lỗi (fault tolerance)

Kiến trúc bất đồng bộ của AgentScope đóng vai trò quan trọng trong trò chơi đa Agent này. Trò chơi thường xuyên xuất hiện các kịch bản cần <strong>thu thập quyết định của nhiều Agent cùng lúc</strong>, ví dụ như giai đoạn bỏ phiếu:

```python
# Thu thập song song quyết định bỏ phiếu của tất cả người chơi
vote_msgs = await fanout_pipeline(
    self.alive_players,
    await self.moderator.announce("Vui lòng bỏ phiếu chọn người chơi muốn loại"),
    structured_model=get_vote_model_cn(self.alive_players),
    enable_gather=False,
)
```

`fanout_pipeline` cho phép chúng ta gửi cùng một tin nhắn đến tất cả các Agent song song, và thu thập phản hồi của chúng một cách bất đồng bộ. Điều này không chỉ nâng cao hiệu quả thực thi của trò chơi, mà quan trọng hơn là mô phỏng được kịch bản "bỏ phiếu đồng thời" trong trò chơi Ma sói thực tế. Đồng thời, chúng ta bổ sung xử lý chịu lỗi (fault tolerance) tại các khâu then chốt:

```python
try:
    response = await wolf(
        "Vui lòng phân tích tình hình hiện tại và bày tỏ quan điểm của bạn.",
        structured_model=DiscussionModelCN
    )
except Exception as e:
    print(f"⚠️ Lỗi khi {wolf.name} thảo luận: {e}")
    # Tạo phản hồi mặc định, đảm bảo trò chơi tiếp tục diễn ra
    default_response = DiscussionModelCN(
        reach_agreement=False,
        confidence_level=5,
        key_evidence="Tạm thời không thể phân tích"
    )
```

Thiết kế này đảm bảo rằng ngay cả khi một Agent nào đó xảy ra lỗi ngoại lệ, toàn bộ luồng trò chơi vẫn có thể tiếp tục diễn ra.

（6）Kết quả đầu ra và tổng kết case

Để cảm nhận trực quan hơn về cơ chế vận hành của AgentScope, dưới đây là một đoạn log vận hành thực tế trích từ giai đoạn ban đêm của trò chơi, thể hiện quá trình hai Agent sói đóng vai "Tôn Quyền" và "Chu Du" thương lượng bí mật và thực hiện việc giết người.

```
🎮 Chào mừng đến với Ma sói Tam Quốc!

=== Khởi tạo trò chơi ===
Quản trò: 📢 [Tôn Quyền] Bạn đóng vai Ma sói trong trò chơi Ma sói Tam Quốc này, nhân vật của bạn là Tôn Quyền. Ban đêm có thể giết một người chơi
Quản trò: 📢 [Chu Du] Bạn đóng vai Ma sói trong trò chơi Ma sói Tam Quốc này, nhân vật của bạn là Chu Du. Ban đêm có thể giết một người chơi
...

Quản trò: 📢 Trò chơi Ma sói Tam Quốc bắt đầu! Người tham gia: Tôn Quyền, Chu Du, Tào Tháo, Trương Phi, Tư Mã Ý, Triệu Vân
✅ Thiết lập trò chơi hoàn tất, tổng cộng 6 người chơi

=== Vòng 1 ===
🌙 Đêm thứ 1 buông xuống, trời tối xin mời nhắm mắt...

[Giai đoạn Ma sói]
Quản trò: 📢 🐺 Ma sói xin mời mở mắt, chọn mục tiêu sẽ giết tối nay...
Quản trò: 📢 Các Ma sói, hãy thảo luận mục tiêu giết tối nay. Người chơi còn sống: Tôn Quyền, Chu Du, Tào Tháo, Trương Phi, Tư Mã Ý, Triệu Vân

Tôn Quyền: Tối nay chúng ta nên trừ khử Chu Du, người này mưu trí hơn người, là mối đe dọa rất lớn đối với chúng ta.
Chu Du: Tôn Quyền, lời ngài nói có lý. Nhưng Chu Du tuy mưu trí, chưa chắc đã là mối đe dọa lớn nhất đêm nay. Thế lực của Tào Tháo rất lớn mạnh, nếu không trừ khử sớm, e rằng sẽ bất lợi cho chúng ta.
Tôn Quyền: Tào Tháo quả thực là một mối đe dọa, nhưng nếu Chu Du còn sống, hắn có thể nhìn thấu mưu kế của chúng ta. Chi bằng giải quyết mối họa trước mắt trước.
Chu Du: Tôn Quyền, mối lo ngại của ngài không phải không có lý. Nhưng nếu Tào Tháo trở thành kẻ thù của chúng ta, hắn có thể liên kết các thế lực khác để tạo ra mối đe dọa lớn hơn nữa.
Tôn Quyền: Ngài nói cũng có lý, sự liên kết của Tào Tháo quả thực rất phiền phức. Vậy chúng ta hãy đối phó với Tào Tháo trước đi.
Chu Du: Rất tốt, Tôn Quyền. Tào Tháo mới chính là mục tiêu hàng đầu của chúng ta đêm nay.

Quản trò: 📢 Vui lòng chọn mục tiêu cần giết
Tôn Quyền: Tôi đồng ý, Tào Tháo nhất định phải bị trừ khử.
Chu Du: Tôi đồng ý, Tào Tháo là mục tiêu chúng ta cần giải quyết đêm nay.

[Giai đoạn Tiên tri]
Quản trò: 📢 🔮 Tiên tri xin mời mở mắt, chọn người chơi muốn kiểm tra...
Tào Tháo: Tôi muốn kiểm tra Tôn Quyền.
Quản trò: 📢 Kết quả kiểm tra: Tôn Quyền là Ma sói

[Giai đoạn Phù thủy]
Quản trò: 📢 🧙‍♀️ Phù thủy xin mời mở mắt...
Quản trò: 📢 Đêm nay Tào Tháo bị Ma sói giết
Trương Phi: Đêm qua ta đã dùng thuốc giải cứu Tào Tháo, hiện tại thuốc giải đã dùng hết.
Quản trò: 📢 Bạn đã dùng thuốc giải cứu Tào Tháo

Quản trò: 📢 Đêm qua bình yên vô sự, không ai chết.

[Giai đoạn thảo luận ban ngày]
Quản trò: 📢 ☀️ Ngày thứ 1 trời sáng rồi, xin mời mọi người mở mắt...
Quản trò: 📢 Bây giờ bắt đầu thảo luận tự do. Người chơi còn sống: Tôn Quyền, Chu Du, Tào Tháo, Trương Phi, Tư Mã Ý, Triệu Vân

Tôn Quyền: Chư vị, thế lực của Tào Tháo rất lớn, là mối đe dọa tiềm tàng đối với tất cả chúng ta. Tối nay tôi đề nghị chúng ta tập trung lực lượng đối phó với hắn.
Chu Du: Tôn Quyền nói rất chí lý, Tào Tháo không chỉ bản thân hùng mạnh, mà còn có thể kết minh với những người chơi khác, gây ra mối đe dọa lớn hơn cho chúng ta.
Tào Tháo: Đêm qua tôi đã kiểm tra Tôn Quyền, vốn tưởng hắn là người tốt, nhưng kết quả quản trò đưa ra lại là Ma sói. Điều này chứng tỏ có Ma sói đang nói dối.
Trương Phi: Đêm qua ta quả thực đã cứu Tào Tháo, chứng tỏ hắn bị Ma sói tấn công. Nhưng kết quả Tào Tháo kiểm tra Tôn Quyền lại khiến người ta nghi ngờ.
Tư Mã Ý: Kết quả kiểm tra của Tào Tháo và hành động cứu người của Trương Phi dường như có sự mâu thuẫn, chúng ta cần thêm thông tin để phán đoán xem ai là Ma sói.
Triệu Vân: Tình hình quả thực phức tạp, chúng ta cần phân tích kỹ phát ngôn của các bên.

[Giai đoạn bỏ phiếu]
Quản trò: 📢 Vui lòng bỏ phiếu chọn người chơi muốn loại

Tôn Quyền: Mối đe dọa từ Tào Tháo vẫn rất lớn, tôi giữ nguyên quan điểm nên bỏ phiếu cho hắn.
Chu Du: Dựa trên tình hình đêm qua, tôi cho rằng chúng ta nên bỏ phiếu cho Tào Tháo trước, mối đe dọa từ hắn là lớn nhất.
Tào Tháo: Tôi chọn bỏ phiếu cho Tôn Quyền. Theo phản hồi từ quản trò, Tôn Quyền quả thực là Ma sói.
Trương Phi: Tôi giữ vững sự thật là đêm qua ta đã cứu Tào Tháo, nhưng việc Tôn Quyền bị kiểm tra ra là Ma sói khiến ta cảm thấy bối rối.
Tư Mã Ý: Chúng ta cần thêm thông tin để phán đoán xem ai là Ma sói.

[Trò chơi tiếp tục...]
```

### 6.3.3 Phân tích ưu điểm và hạn chế của AgentScope

Thông qua case "Ma sói Tam Quốc" này, chúng ta đã trải nghiệm sâu sắc ưu điểm cốt lõi của framework AgentScope. Framework này lấy kiến trúc hướng tin nhắn làm cốt lõi, ánh xạ một cách tinh tế luồng trò chơi phức tạp thành một chuỗi các sự kiện truyền tin nhắn đồng thời, bất đồng bộ, từ đó tránh được sự cứng nhắc và phức tạp của state machine truyền thống. Kết hợp với năng lực Structured Output mạnh mẽ, chúng ta chuyển hóa trực tiếp luật chơi thành ràng buộc ở tầng code, nâng cao đáng kể tính ổn định và khả năng dự đoán của hệ thống. Mô hình thiết kế này không chỉ thể hiện ưu thế về hiệu năng của tính đồng thời nguyên bản (native concurrency), mà còn đảm bảo rằng ngay cả khi một Agent đơn lẻ gặp lỗi ngoại lệ, toàn bộ luồng vẫn có thể vận hành ổn định nhờ cơ chế chịu lỗi.

Tuy nhiên, ưu thế kỹ thuật hóa của AgentScope cũng kéo theo một chi phí phức tạp nhất định. Kiến trúc hướng tin nhắn của nó tuy mạnh mẽ nhưng đòi hỏi nhà phát triển phải có trình độ kỹ thuật cao hơn, cần hiểu các khái niệm như lập trình bất đồng bộ, giao tiếp phân tán. Đối với các kịch bản hội thoại đa Agent đơn giản, kiến trúc này có thể trở nên quá phức tạp, tồn tại nguy cơ "kỹ thuật hóa quá mức" (over-engineering). Ngoài ra, với tư cách là một framework tương đối mới, hệ sinh thái và tài nguyên cộng đồng của nó vẫn cần được hoàn thiện thêm. Vì vậy, AgentScope phù hợp hơn với việc xây dựng các hệ thống đa Agent quy mô lớn, độ tin cậy cao ở cấp production, còn đối với việc phát triển prototype nhanh hay các kịch bản ứng dụng đơn giản, việc lựa chọn một framework nhẹ (lightweight) hơn có thể phù hợp hơn.



## 6.4 Framework ba: CAMEL

Khác với các framework toàn diện như AutoGen và AgentScope, mục tiêu cốt lõi ban đầu của CAMEL là khám phá cách để hai Agent tự chủ hợp tác giải quyết nhiệm vụ phức tạp thông qua "đóng vai" (role-playing) với sự can thiệp tối thiểu của con người.

### 6.4.1 Sự hợp tác tự chủ của CAMEL

Nền tảng để CAMEL hiện thực hóa sự hợp tác tự chủ là hai khái niệm cốt lõi: <strong>Đóng vai (Role-Playing)</strong> và <strong>Prompt khơi mào (Inception Prompting)</strong>.

（1）Đóng vai (Role-Playing)

Trong thiết kế ban đầu của CAMEL, một nhiệm vụ thường được hoàn thành bởi hai Agent hợp tác. Hai Agent này được gán các "vai trò" bổ sung cho nhau, được định nghĩa rõ ràng. Một Agent đóng vai <strong>"Người dùng AI" (AI User)</strong>, chịu trách nhiệm đề xuất yêu cầu, ra lệnh và hình dung các bước nhiệm vụ; một Agent khác đóng vai <strong>"Trợ lý AI" (AI Assistant)</strong>, chịu trách nhiệm thực thi các thao tác cụ thể và đưa ra giải pháp theo chỉ dẫn.

Ví dụ, trong nhiệm vụ "phát triển một công cụ phân tích chiến lược giao dịch chứng khoán":

- Vai trò <strong>Người dùng AI</strong> có thể là một "nhà giao dịch chứng khoán dày dạn kinh nghiệm". Người này am hiểu thị trường, am hiểu chiến lược, nhưng không biết lập trình.
- Vai trò <strong>Trợ lý AI</strong> thì là một "lập trình viên Python xuất sắc". Người này tinh thông lập trình, nhưng không biết gì về giao dịch chứng khoán.

Thông qua thiết lập này, quá trình giải quyết nhiệm vụ được chuyển hóa một cách tự nhiên thành một cuộc hội thoại giữa hai "chuyên gia liên ngành". Nhà giao dịch đề xuất yêu cầu chuyên môn, lập trình viên chuyển hóa nó thành code triển khai, cả hai hợp tác hoàn thành một nhiệm vụ phức tạp mà bất kỳ bên nào cũng không thể tự mình hoàn thành.

（2）Prompt khơi mào (Inception Prompting)

Chỉ thiết lập vai trò thôi là chưa đủ, làm sao để đảm bảo hai AI luôn "ở đúng vai trò của mình" khi không có sự giám sát liên tục của con người, và tiến về mục tiêu chung một cách hiệu quả? Đây chính là nơi kỹ thuật cốt lõi nhất của CAMEL, Prompt khơi mào, phát huy tác dụng. "Prompt khơi mào" là một đoạn chỉ dẫn khởi đầu (System Prompt) được thiết kế tỉ mỉ, có cấu trúc, được tiêm vào riêng cho hai Agent trước khi hội thoại bắt đầu. Chỉ dẫn này giống như một "cương lĩnh hành động" được cấy vào Agent, nó thường bao gồm các phần then chốt sau:

- <strong>Xác định rõ vai trò của bản thân</strong>: Ví dụ, "Bạn là một nhà giao dịch chứng khoán dày dạn kinh nghiệm..."
- <strong>Thông báo vai trò của người cộng tác</strong>: Ví dụ, "Bạn đang hợp tác với một lập trình viên Python xuất sắc..."
- <strong>Định nghĩa mục tiêu chung</strong>: Ví dụ, "Mục tiêu chung của hai bạn là phát triển một công cụ phân tích chiến lược giao dịch chứng khoán."
- <strong>Thiết lập ràng buộc hành vi và giao thức giao tiếp</strong>: Đây là khâu then chốt nhất. Ví dụ, chỉ dẫn sẽ yêu cầu AI Người dùng "mỗi lần chỉ đề xuất một bước rõ ràng, cụ thể", và yêu cầu AI Trợ lý "không hỏi thêm chi tiết trước khi hoàn thành bước trước đó", đồng thời quy định cả hai bên cần sử dụng một cờ hiệu (flag) đặc biệt (như `<SOLUTION>`) ở cuối phản hồi để đánh dấu nhiệm vụ đã hoàn thành.

Các điều kiện ràng buộc này đảm bảo hội thoại không lệch khỏi chủ đề, không rơi vào vòng lặp vô ích, mà tiến triển theo một cách có cấu trúc cao, hướng tới nhiệm vụ (task-driven), như trong Hình 6.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/04.png" alt="" width="90%"/>
  <p>Hình 6.3 CAMEL tạo robot giao dịch chứng khoán</p>
</div>

Trong mục tiếp theo, chúng ta sẽ trải nghiệm quá trình này thông qua một ví dụ cụ thể.

### 6.4.2 Ebook phổ cập kiến thức AI

Để hiểu năng lực đóng vai của framework CAMEL, chúng ta sẽ xây dựng một case hợp tác có giá trị thực tiễn: để một AI nhà tâm lý học và một AI nhà văn hợp tác, cùng sáng tác một cuốn ebook ngắn về "tâm lý học của sự trì hoãn" (procrastination). Case này thể hiện ưu thế cốt lõi của CAMEL, cho phép hai Agent phát huy sở trường trong lĩnh vực chuyên môn riêng của mình, hợp tác hoàn thành một nhiệm vụ sáng tạo phức tạp mà một Agent đơn lẻ khó có thể đảm nhiệm.

（1）Thiết lập nhiệm vụ

<strong>Bối cảnh</strong>: Sáng tác một cuốn ebook phổ cập kiến thức tâm lý học về sự trì hoãn hướng tới độc giả phổ thông, yêu cầu vừa có tính khoa học nghiêm ngặt, vừa có khả năng đọc hiểu tốt.

<strong>Vai trò Agent</strong>:

- <strong>Nhà tâm lý học (Psychologist)</strong>: Có nền tảng lý thuyết tâm lý học sâu sắc, am hiểu các lĩnh vực liên quan như khoa học hành vi nhận thức (cognitive behavioral science), khoa học thần kinh (neuroscience), có thể cung cấp kiến giải học thuật chuyên nghiệp và hỗ trợ bằng các nghiên cứu thực chứng (empirical research).
- <strong>Nhà văn (Writer)</strong>: Có kỹ thuật viết và khả năng trần thuật xuất sắc, giỏi chuyển hóa các khái niệm học thuật phức tạp thành văn phong sinh động, dễ hiểu, chú trọng trải nghiệm của người đọc và tính dễ đọc của nội dung.

（2）Định nghĩa nhiệm vụ hợp tác

Trước tiên, chúng ta cần làm rõ mục tiêu chung của hai chuyên gia AI. Chúng ta định nghĩa nhiệm vụ này thông qua một chuỗi `task_prompt` có nội dung chi tiết.

```python
from colorama import Fore
from camel.societies import RolePlaying
from camel.utils import print_text_animated
from camel.models import ModelFactory
from camel.types import ModelPlatformType
from dotenv import load_dotenv
import os

load_dotenv()
LLM_API_KEY = os.getenv("LLM_API_KEY")
LLM_BASE_URL = os.getenv("LLM_BASE_URL")
LLM_MODEL = os.getenv("LLM_MODEL")

# Tạo model, ở đây lấy Qwen làm ví dụ, gọi API nền tảng mô hình lớn Bailian (Alibaba Cloud)
model = ModelFactory.create(
    model_platform=ModelPlatformType.QWEN,
    model_type=LLM_MODEL,
    url=LLM_BASE_URL,
    api_key=LLM_API_KEY
)

# Định nghĩa nhiệm vụ hợp tác
task_prompt = """
Sáng tác một cuốn ebook ngắn về "Tâm lý học của sự trì hoãn", độc giả mục tiêu là đại chúng quan tâm đến tâm lý học.
Yêu cầu:
1. Nội dung khoa học, nghiêm ngặt, dựa trên nghiên cứu thực chứng
2. Ngôn ngữ bình dị dễ hiểu, tránh quá nhiều thuật ngữ chuyên ngành
3. Bao gồm các lời khuyên cải thiện thiết thực và phân tích case study
4. Độ dài khống chế trong khoảng 8.000-10.000 từ
5. Cấu trúc rõ ràng, bao gồm phần mở đầu, các chương cốt lõi và phần tổng kết
"""

print(Fore.YELLOW + f"Nhiệm vụ hợp tác:\n{task_prompt}\n")
```

`task_prompt` chính là "bản thuyết minh nhiệm vụ" của toàn bộ sự hợp tác. Nó không chỉ là mục tiêu chúng ta cần hoàn thành, mà còn được CAMEL sử dụng ở phía sau hậu trường để tạo ra "Prompt khơi mào", đảm bảo hội thoại của hai Agent luôn xoay quanh mục tiêu cốt lõi này.

（3）Khởi tạo "xã hội" đóng vai

Tiếp theo, chúng ta tạo một instance phiên `RolePlaying`. Đây là thao tác cốt lõi của CAMEL, nó dựa trên vai trò và nhiệm vụ mà chúng ta cung cấp, nhanh chóng xây dựng một "xã hội" hợp tác song-Agent.

```python
# Khởi tạo phiên đóng vai
# Nhà văn AI đóng vai "user", chịu trách nhiệm đề xuất cấu trúc viết và các yêu cầu
# Nhà tâm lý học AI đóng vai "assistant", chịu trách nhiệm cung cấp kiến thức chuyên môn và nội dung
role_play_session = RolePlaying(
    assistant_role_name="Nhà tâm lý học",
    user_role_name="Nhà văn",
    task_prompt=task_prompt,
    model=model,
    with_task_specify=False, # Trong ví dụ này, chúng ta sử dụng trực tiếp task_prompt đã cho
)

print(Fore.CYAN + f"Mô tả nhiệm vụ cụ thể:\n{role_play_session.task_prompt}\n")
```

`RolePlaying` là một API cấp cao mà CAMEL cung cấp, nó đóng gói prompt engineering phức tạp. Chúng ta chỉ cần truyền vào tên của hai vai trò và nhiệm vụ. Trong thiết kế của CAMEL, vai trò `user` là "người thúc đẩy" và "bên có nhu cầu" của hội thoại, còn vai trò `assistant` là "người thực thi" và "bên cung cấp giải pháp". Do đó, chúng ta gán "nhà văn" chịu trách nhiệm lên kế hoạch cấu trúc cho `user_role_name`, gán "nhà tâm lý học" chịu trách nhiệm cung cấp kiến thức chuyên môn cho `assistant_role_name`.

（4）Khởi động và chạy hội thoại tự động hóa

Cuối cùng, chúng ta viết một vòng lặp để thúc đẩy toàn bộ quá trình hội thoại, để hai chuyên gia AI bắt đầu quá trình hợp tác tự động hóa của mình.

```python
# Bắt đầu hội thoại hợp tác
chat_turn_limit, n = 30, 0
# Gọi init_chat() để nhận tin nhắn hội thoại ban đầu do AI tạo ra
input_msg = role_play_session.init_chat()

while n < chat_turn_limit:
    n += 1
    # Phương thức step() thúc đẩy một vòng hội thoại hoàn chỉnh, Người dùng AI và Trợ lý AI mỗi bên phát biểu một lần
    assistant_response, user_response = role_play_session.step(input_msg)
    
    # Kiểm tra xem có tin nhắn trả về không, tránh việc hội thoại kết thúc sớm
    if assistant_response.msg is None or user_response.msg is None:
        break
    
    print_text_animated(Fore.BLUE + f"Nhà văn (AI User):\n\n{user_response.msg.content}\n")
    print_text_animated(Fore.GREEN + f"Nhà tâm lý học (AI Assistant):\n\n{assistant_response.msg.content}\n")
    
    # Kiểm tra cờ hiệu hoàn thành nhiệm vụ
    if "<CAMEL_TASK_DONE>" in user_response.msg.content or "<CAMEL_TASK_DONE>" in assistant_response.msg.content:
        print(Fore.MAGENTA + "✅ Sáng tác ebook hoàn tất!")
        break
    
    # Lấy phản hồi của trợ lý làm đầu vào cho vòng hội thoại tiếp theo
    input_msg = assistant_response.msg

print(Fore.YELLOW + f"Tổng cộng đã thực hiện {n} vòng hội thoại hợp tác")
```

Vòng lặp `while` này chính là cốt lõi của sự hợp tác tự động hóa. Hội thoại được tự động khởi động bởi phương thức `init_chat()` dựa trên nhiệm vụ và vai trò, không cần viết tay lời mở đầu. Mỗi bước trong vòng lặp đều thông qua việc gọi `step()` để thúc đẩy một vòng tương tác hoàn chỉnh (nhà văn đề xuất yêu cầu, nhà tâm lý học cung cấp nội dung), và lấy đầu ra của nhà tâm lý học ở vòng trước làm đầu vào cho vòng tiếp theo, hình thành chuỗi sáng tạo liên hoàn. Toàn bộ quá trình sẽ tiếp tục cho đến khi đạt số vòng hội thoại tối đa được thiết lập trước, hoặc khi một trong hai Agent xuất ra cờ hiệu hoàn thành nhiệm vụ `<CAMEL_TASK_DONE>` thì sẽ tự động kết thúc.

（5）Trình bày luồng hợp tác

Khi thực thi đoạn code trên, chúng ta không chỉ nhận được một chuỗi dài các câu hỏi-đáp đơn điệu, mà có thể quan sát được một luồng hợp tác có cấu trúc cao, giống như một đội ngũ chuyên gia con người, đang tự động diễn ra. Toàn bộ quá trình sáng tạo được chia tự nhiên thành vài giai đoạn:

<strong>Giai đoạn một (khoảng vòng 1-5): Xây dựng khung sườn và thống nhất mục tiêu</strong> Ở giai đoạn đầu của hội thoại, Agent "nhà văn" trước tiên sẽ đóng vai người dẫn dắt chủ đạo, đề xuất ý tưởng sơ bộ về cấu trúc tổng thể và cách sắp xếp chương của ebook. Sau đó, "nhà tâm lý học" sẽ xem xét và bổ sung cho khung sườn này từ góc độ chuyên môn của mình, đảm bảo các module học thuật cốt lõi (như nền tảng lý thuyết, khái niệm then chốt) không bị bỏ sót, từ đó đạt được sự đồng thuận về sản phẩm cuối cùng ngay từ đầu quá trình hợp tác.

<strong>Giai đoạn hai (khoảng vòng 6-20): Sinh nội dung cốt lõi và chuyển dịch tri thức</strong> Đây là giai đoạn sáng tạo nội dung hiệu quả nhất. Mô hình hợp tác sẽ chuyển thành một vòng lặp "yêu cầu-phản hồi" ổn định:

- <strong>Nhà tâm lý học</strong>: Chịu trách nhiệm cung cấp kiến thức chuyên môn "cứng", như giải thích khoa học về các khái niệm cốt lõi như "lý thuyết chiết khấu thời gian" (temporal discounting), "khiếm khuyết chức năng điều hành" (executive function deficit), và trích dẫn các nghiên cứu thực nghiệm liên quan để hỗ trợ quan điểm.
- <strong>Nhà văn</strong>: Thì phát huy vai trò "thông dịch viên" của mình, chuyển hóa những khái niệm học thuật nghiêm ngặt nhưng có thể khó hiểu này thành những phép ẩn dụ sinh động, hình tượng và những ví dụ gần gũi với đời sống. Ví dụ, có thể ví khái niệm "thiên kiến hiện tại" (present bias) trong não bộ như "một đứa trẻ bướng bỉnh chỉ quan tâm đến viên kẹo trước mắt mà không đoái hoài tới sức khỏe lâu dài".

<strong>Giai đoạn ba (khoảng vòng 21-25): Tối ưu hóa lặp lại và đảm bảo chất lượng</strong> Khi nội dung chính của cuốn sách đã hoàn thành, trọng tâm của hội thoại sẽ chuyển sang việc trau chuốt và hoàn thiện văn bản đã có. Lúc này, vai trò của hai Agent sẽ có sự thay đổi tinh tế:

- <strong>Nhà văn</strong>: Chú trọng nhiều hơn vào việc xem xét tính trôi chảy tổng thể của bài viết, sự liên kết logic và phong cách ngôn ngữ, đề xuất chỉnh sửa từ góc độ "trải nghiệm người đọc".
- <strong>Nhà tâm lý học</strong>: Lại đóng vai "người kiểm tra sự thật" (fact checker) một lần nữa, đảm bảo rằng tính chính xác khoa học của tri thức cốt lõi không bị mất đi trong quá trình chuyển dịch và trau chuốt, đồng thời bổ sung các hỗ trợ nghiên cứu thực chứng mạnh mẽ hơn cho một số quan điểm.

<strong>Giai đoạn bốn (giai đoạn kết thúc): Tổng kết và nâng cao</strong> Trong vài vòng hội thoại cuối cùng, hai bên sẽ hợp tác hoàn thành phần tổng kết các lời khuyên thực tiễn và nhìn lại toàn bộ cuốn sách, đảm bảo ebook có một kết thúc rõ ràng, mạnh mẽ, để lại ấn tượng sâu sắc và cung cấp giá trị thực tiễn cho người đọc.

```
Nhiệm vụ hợp tác:
Sáng tác một cuốn ebook ngắn về "Tâm lý học của sự trì hoãn", độc giả mục tiêu là đại chúng quan tâm đến tâm lý học.
Yêu cầu:
1. Nội dung khoa học, nghiêm ngặt, dựa trên nghiên cứu thực chứng
2. Ngôn ngữ bình dị dễ hiểu, tránh quá nhiều thuật ngữ chuyên ngành
3. Bao gồm các lời khuyên cải thiện thiết thực và phân tích case study
4. Độ dài khống chế trong khoảng 8.000-10.000 từ
5. Cấu trúc rõ ràng, bao gồm phần mở đầu, các chương cốt lõi và phần tổng kết

Mô tả nhiệm vụ cụ thể:
Viết một cuốn ebook ngắn dài 8.000–10.000 từ với tựa đề "Tâm lý học của sự trì hoãn" cho độc giả phổ thông: dựa trên thực chứng, bình dị dễ hiểu. Cấu trúc: Mở đầu, Nguyên nhân (nhận thức/cảm xúc/phần thưởng), Động lực và ra quyết định, Hình thành thói quen và can thiệp, Chiến lược thực tiễn và bài tập, Ba tình huống phân tích, Tổng kết và tài nguyên. Mỗi chương đều có trích dẫn nghiên cứu và các bước có thể thực hành.

Nhà văn:
Instruction: Hãy viết một bản thảo dài 400–600 từ cho chương "Mở đầu" của cuốn ebook...
Input: None

Nhà tâm lý học:
Solution:
Bản thảo: Sự trì hoãn là hành vi và khuynh hướng nội tại liên tục chần chừ hoặc né tránh một nhiệm vụ dù biết rõ rằng cần phải hoàn thành nó. Nó có thể là một vấn đề quản lý thời gian thỉnh thoảng xảy ra...

Next request.

Nhà văn:
Instruction: Hãy chỉnh sửa bản thảo mở đầu dưới đây thành một đoạn văn bản dài 450–550 từ...
Input: Bản thảo: Sự trì hoãn là hành vi và khuynh hướng nội tại liên tục chần chừ hoặc né tránh một nhiệm vụ...
.....
```

### 6.4.3 Phân tích ưu điểm và hạn chế của CAMEL

Thông qua case sáng tạo ebook ở trên, chúng ta đã trải nghiệm sâu sắc mô hình đóng vai độc đáo của framework CAMEL. Bây giờ hãy phân tích một cách khách quan ưu điểm và hạn chế của triết lý thiết kế này, để có thể đưa ra lựa chọn kỹ thuật sáng suốt trong các dự án thực tế.

（1）Ưu điểm

Ưu điểm lớn nhất của CAMEL nằm ở triết lý thiết kế "kiến trúc nhẹ, prompt nặng" (light architecture, heavy prompting). So với việc quản lý hội thoại phức tạp của AutoGen và kiến trúc phân tán của AgentScope, CAMEL chỉ cần thông qua prompt khởi đầu được thiết kế tỉ mỉ đã có thể hiện thực hóa sự hợp tác Agent chất lượng cao. Hành vi hợp tác nảy sinh tự nhiên (emergent) này thường linh hoạt và hiệu quả hơn so với luồng công việc được hard-code.

Đáng chú ý là, framework CAMEL đang trải qua sự phát triển và tiến hóa nhanh chóng. Từ [GitHub repository](https://github.com/camel-ai/camel) của nó có thể thấy, CAMEL đã không còn đơn thuần là một framework hợp tác song-Agent đơn giản, mà hiện đã sở hữu:

- <strong>Năng lực đa phương thức (multimodal)</strong>: Hỗ trợ sự hợp tác của Agent trên nhiều phương thức như văn bản, hình ảnh, âm thanh.
- <strong>Tích hợp Tool</strong>: Đã tích hợp sẵn một kho tool phong phú, bao gồm tìm kiếm, tính toán, thực thi code, v.v.
- <strong>Thích ứng mô hình</strong>: Hỗ trợ nhiều backend LLM như OpenAI, Anthropic, Google, mô hình mã nguồn mở (open-source model).
- <strong>Liên kết hệ sinh thái</strong>: Đã hiện thực hóa khả năng tương tác (interoperability) với các framework chủ đạo khác như LangChain, CrewAI, AutoGen.

（2）Hạn chế chính

1. Sự phụ thuộc cao vào Prompt Engineering

Sự thành công của CAMEL phần lớn phụ thuộc vào chất lượng của prompt khởi đầu. Điều này mang lại một vài thách thức:

- <strong>Ngưỡng thiết kế prompt</strong>: Cần hiểu sâu sắc về lĩnh vực mục tiêu và đặc tính hành vi của LLM.
- <strong>Độ phức tạp khi debug</strong>: Khi hiệu quả hợp tác không tốt, rất khó xác định liệu vấn đề nằm ở định nghĩa vai trò, mô tả nhiệm vụ hay quy tắc tương tác.
- <strong>Thách thức về tính nhất quán</strong>: Các LLM khác nhau có thể hiểu cùng một prompt theo cách khác nhau.

2. Hạn chế về quy mô hợp tác

Mặc dù CAMEL thể hiện xuất sắc trong việc hợp tác song-Agent, nhưng khi xử lý các kịch bản đa Agent quy mô lớn thì gặp phải thách thức:

- <strong>Quản lý hội thoại</strong>: Thiếu cơ chế định tuyến hội thoại phức tạp giống như AutoGen.
- <strong>Đồng bộ trạng thái</strong>: Không có khả năng quản lý trạng thái phân tán giống như AgentScope.
- <strong>Giải quyết xung đột</strong>: Khi nhiều Agent có ý kiến bất đồng, thiếu cơ chế trọng tài (arbitration) hiệu quả.

3. Ranh giới về khả năng áp dụng cho nhiệm vụ

CAMEL đặc biệt phù hợp với các nhiệm vụ cần sự hợp tác sâu và tư duy sáng tạo, nhưng trong một số kịch bản thì có thể không phải là lựa chọn tối ưu:

- <strong>Kiểm soát quy trình nghiêm ngặt</strong>: Đối với các nhiệm vụ cần kiểm soát bước chính xác, cấu trúc đồ thị của LangGraph phù hợp hơn.
- <strong>Xử lý đồng thời quy mô lớn</strong>: Kiến trúc hướng tin nhắn của AgentScope có ưu thế hơn trong các kịch bản xử lý đồng thời cao.
- <strong>Cây quyết định phức tạp</strong>: Mô hình nhóm chat của AutoGen linh hoạt hơn trong các kịch bản ra quyết định đa phương.

Nhìn chung, CAMEL đại diện cho một mô hình hợp tác đa Agent độc đáo và thanh lịch. Nó thông qua thiết kế đóng vai "lấy con người làm trung tâm" (human-centric), chuyển hóa bài toán kỹ thuật hệ thống phức tạp thành mô hình hợp tác giữa con người với con người trực quan. Cùng với hệ sinh thái và tính năng không ngừng được hoàn thiện và mở rộng, CAMEL đang trở thành một trong những lựa chọn quan trọng để xây dựng các hệ thống hợp tác thông minh.

## 6.5 Framework bốn: LangGraph

### 6.5.1 Cấu trúc tổng quan của LangGraph

Là phần mở rộng quan trọng của hệ sinh thái LangChain, LangGraph đại diện cho một hướng đi hoàn toàn mới trong thiết kế Agent Framework. Khác với các framework dựa trên "hội thoại" được giới thiệu ở trên (như AutoGen và CAMEL), LangGraph mô hình hóa quy trình thực thi của Agent thành một dạng <strong>state machine</strong>, và biểu diễn nó dưới dạng <strong>đồ thị có hướng (Directed Graph)</strong>. Trong mô hình này, <strong>nút (Nodes)</strong> của đồ thị đại diện cho một bước tính toán cụ thể (như gọi LLM, thực thi tool), còn <strong>cạnh (Edges)</strong> thì định nghĩa logic chuyển tiếp (jump) từ node này sang node khác. Tính cách mạng trong thiết kế này nằm ở chỗ nó vốn dĩ hỗ trợ vòng lặp, khiến việc xây dựng các luồng công việc Agent phức tạp có thể lặp lại, phản tư và tự sửa chữa trở nên trực quan và đơn giản chưa từng có.

Để hiểu LangGraph, trước tiên chúng ta cần nắm vững ba thành phần cơ bản cấu thành nên nó.

<strong>Đầu tiên, là trạng thái toàn cục (State)</strong>. Toàn bộ quá trình thực thi của đồ thị xoay quanh một đối tượng trạng thái (state object) được chia sẻ chung. Trạng thái này thường được định nghĩa là một `TypedDict` của Python, nó có thể chứa bất kỳ thông tin nào bạn cần theo dõi, như lịch sử hội thoại, kết quả trung gian, số lần lặp, v.v. Tất cả các node đều có thể đọc và cập nhật trạng thái trung tâm này.

```python
from typing import TypedDict, List

# Định nghĩa cấu trúc dữ liệu của trạng thái toàn cục
class AgentState(TypedDict):
    messages: List[str]      # Lịch sử hội thoại
    current_task: str        # Nhiệm vụ hiện tại
    final_answer: str        # Câu trả lời cuối cùng
    # ... bất kỳ trạng thái nào khác cần theo dõi
```

<strong>Thứ hai, là Node</strong>. Mỗi node đều là một hàm Python nhận trạng thái hiện tại làm đầu vào, và trả về một trạng thái đã được cập nhật làm đầu ra. Node là đơn vị thực thi công việc cụ thể.

```python
# Định nghĩa một hàm node "Planner" (người lập kế hoạch)
def planner_node(state: AgentState) -> AgentState:
    """Lập kế hoạch dựa trên nhiệm vụ hiện tại và cập nhật trạng thái."""
    current_task = state["current_task"]
    # ... Gọi LLM để tạo kế hoạch ...
    plan = f"Kế hoạch được tạo cho nhiệm vụ '{current_task}'..."
    
    # Thêm tin nhắn mới vào trạng thái
    state["messages"].append(plan)
    return state

# Định nghĩa một hàm node "Executor" (người thực thi)
def executor_node(state: AgentState) -> AgentState:
    """Thực thi kế hoạch mới nhất và cập nhật trạng thái."""
    latest_plan = state["messages"][-1]
    # ... Thực thi kế hoạch và nhận kết quả ...
    result = f"Kết quả thực thi kế hoạch '{latest_plan}'..."
    
    state["messages"].append(result)
    return state
```

<strong>Cuối cùng, là Edge (cạnh)</strong>. Edge chịu trách nhiệm kết nối các node, định nghĩa hướng đi của luồng công việc. Edge đơn giản nhất là edge thông thường, nó chỉ định rằng đầu ra của một node luôn chảy sang một node cố định khác. Còn tính năng mạnh mẽ nhất của LangGraph nằm ở <strong>Conditional Edge (cạnh có điều kiện)</strong>. Nó thông qua một hàm để phán đoán trạng thái hiện tại, sau đó quyết định động xem bước tiếp theo nên nhảy tới node nào. Đây chính là chìa khóa để hiện thực hóa vòng lặp và các nhánh logic phức tạp.

```python
def should_continue(state: AgentState) -> str:
    """Hàm điều kiện: quyết định định tuyến bước tiếp theo dựa trên trạng thái."""
    # Giả sử nếu có ít hơn 3 tin nhắn thì cần tiếp tục lập kế hoạch
    if len(state["messages"]) < 3:
        # Chuỗi trả về cần khớp với khóa được định nghĩa khi thêm cạnh có điều kiện
        return "continue_to_planner"
    else:
        state["final_answer"] = state["messages"][-1]
        return "end_workflow"
```

Sau khi đã định nghĩa State, Node và Edge, chúng ta có thể lắp ráp chúng thành một luồng công việc có thể thực thi, giống như lắp ráp các khối lego.

```python
from langgraph.graph import StateGraph, END

# Khởi tạo đồ thị trạng thái và gắn cấu trúc trạng thái đã định nghĩa
workflow = StateGraph(AgentState)

# Thêm các hàm node vào đồ thị
workflow.add_node("planner", planner_node)
workflow.add_node("executor", executor_node)

# Thiết lập điểm vào của đồ thị
workflow.set_entry_point("planner")

# Thêm cạnh thông thường, kết nối planner và executor
workflow.add_edge("planner", "executor")

# Thêm cạnh có điều kiện, thực hiện định tuyến động
workflow.add_conditional_edges(
    # Node bắt đầu
    "executor",
    # Hàm phán đoán
    should_continue,
    # Ánh xạ định tuyến: ánh xạ giá trị trả về của hàm phán đoán tới node đích
    {
        "continue_to_planner": "planner", # Nếu trả về "continue_to_planner", nhảy lại node planner
        "end_workflow": END               # Nếu trả về "end_workflow", kết thúc luồng
    }
)

# Biên dịch đồ thị, tạo ứng dụng có thể thực thi
app = workflow.compile()

# Chạy đồ thị
inputs = {"current_task": "Phân tích tin tức ngành AI gần đây", "messages": []}
for event in app.stream(inputs):
    print(event)
```

### 6.5.2 Trợ lý hỏi đáp ba bước
Sau khi hiểu các khái niệm cốt lõi của LangGraph, chúng ta sẽ củng cố kiến thức đã học thông qua một case thực hành. Chúng ta sẽ xây dựng một trợ lý hội thoại hỏi đáp đơn giản hóa, nó sẽ tuân theo một quy trình ba bước rõ ràng, cố định để trả lời câu hỏi của người dùng:

1. <strong>Hiểu (Understand)</strong>: Trước tiên, phân tích ý định truy vấn (query intent) của người dùng.
2. <strong>Tìm kiếm (Search)</strong>: Sau đó, mô phỏng việc tìm kiếm thông tin liên quan đến ý định đó.
3. <strong>Trả lời (Answer)</strong>: Cuối cùng, dựa trên ý định và thông tin tìm kiếm được, sinh ra câu trả lời cuối cùng.

Case này sẽ thể hiện rõ ràng cách định nghĩa state, tạo node và kết nối chúng tuyến tính thành một luồng công việc hoàn chỉnh. Chúng ta sẽ phân giải code thành bốn bước cốt lõi: định nghĩa state, tạo node, xây dựng đồ thị, và chạy ứng dụng.

（1）Định nghĩa trạng thái toàn cục

Trước tiên, chúng ta cần định nghĩa một trạng thái toàn cục xuyên suốt luồng công việc. <strong>Đây là một cấu trúc dữ liệu được chia sẻ, nó được truyền giữa các node của đồ thị, đóng vai trò là ngữ cảnh lưu trữ bền vững (persistent context) của luồng công việc.</strong> Mỗi node đều có thể đọc dữ liệu trong cấu trúc này, và cập nhật nó.

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class SearchState(TypedDict):
    messages: Annotated[list, add_messages]
    user_query: str      # Tóm tắt nhu cầu người dùng sau khi được LLM hiểu
    search_query: str    # Câu truy vấn tìm kiếm đã tối ưu cho Tavily API
    search_results: str  # Kết quả trả về từ tìm kiếm Tavily
    final_answer: str    # Câu trả lời cuối cùng được sinh ra
    step: str            # Đánh dấu bước hiện tại
```

Chúng ta tạo `SearchState` là một `TypedDict`, định nghĩa một schema dữ liệu rõ ràng cho đối tượng trạng thái. Một thiết kế then chốt là việc đồng thời bao gồm hai trường `user_query` và `search_query`. Điều này cho phép Agent trước tiên tối ưu hóa câu hỏi ngôn ngữ tự nhiên của người dùng thành các từ khóa tinh gọn hơn, phù hợp hơn với công cụ tìm kiếm, từ đó nâng cao đáng kể chất lượng kết quả tìm kiếm.

（2）Định nghĩa các node của luồng công việc

Sau khi định nghĩa cấu trúc state, bước tiếp theo là tạo các node cấu thành nên luồng công việc của chúng ta. Trong LangGraph, mỗi node đều là một hàm Python thực thi một nhiệm vụ cụ thể. Các hàm này nhận đối tượng trạng thái hiện tại làm đầu vào, và trả về một dictionary chứa các trường đã được cập nhật.

Trước khi bắt đầu định nghĩa node, chúng ta hoàn thiện việc thiết lập khởi tạo dự án, bao gồm việc nạp biến môi trường và khởi tạo mô hình ngôn ngữ lớn.

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from tavily import TavilyClient

# Nạp các biến môi trường từ file .env
load_dotenv()

# Khởi tạo mô hình
# Chúng ta sẽ sử dụng instance llm này để điều khiển trí thông minh cho tất cả các node
llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL_ID", "gpt-4o-mini"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1"),
    temperature=0.7
)
# Khởi tạo Tavily client
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
```

Bây giờ, chúng ta lần lượt tạo ba node cốt lõi.

（1） Node hiểu và truy vấn

Node này là bước đầu tiên của luồng công việc, trách nhiệm của node này là hiểu ý định của người dùng, và tạo ra một câu truy vấn tìm kiếm được tối ưu hóa cho nó.

```python
def understand_query_node(state: SearchState) -> dict:
    """Bước 1: Hiểu truy vấn người dùng và tạo từ khóa tìm kiếm"""
    user_message = state["messages"][-1].content
    
    understand_prompt = f"""Phân tích truy vấn của người dùng: "{user_message}"
Vui lòng hoàn thành hai nhiệm vụ:
1. Tóm tắt ngắn gọn điều người dùng muốn tìm hiểu
2. Tạo các từ khóa phù hợp nhất cho công cụ tìm kiếm (tiếng Việt hoặc tiếng Anh đều được, cần chính xác)

Định dạng:
Hiểu ý định: [Tóm tắt nhu cầu người dùng]
Từ khóa tìm kiếm: [Các từ khóa tìm kiếm tối ưu nhất]"""

    response = llm.invoke([SystemMessage(content=understand_prompt)])
    response_text = response.content
    
    # Phân tích đầu ra của LLM, trích xuất từ khóa tìm kiếm
    search_query = user_message # Mặc định sử dụng truy vấn gốc
    if "Từ khóa tìm kiếm:" in response_text:
        search_query = response_text.split("Từ khóa tìm kiếm:")[1].strip()
    
    return {
        "user_query": response_text,
        "search_query": search_query,
        "step": "understood",
        "messages": [AIMessage(content=f"Tôi sẽ tìm kiếm giúp bạn: {search_query}")]
    }
```

Node này thông qua một prompt có cấu trúc, yêu cầu LLM đồng thời hoàn thành hai nhiệm vụ "hiểu ý định" và "sinh từ khóa", và cập nhật từ khóa tìm kiếm chuyên dụng được phân tích vào trường `search_query` của state, chuẩn bị cho việc tìm kiếm chính xác ở bước tiếp theo.

（2）Node tìm kiếm

Node này chịu trách nhiệm thực thi khả năng "sử dụng tool" của Agent, nó sẽ gọi Tavily API để thực hiện tìm kiếm internet thực sự, và có sẵn chức năng xử lý lỗi cơ bản.

```python
def tavily_search_node(state: SearchState) -> dict:
    """Bước 2: Sử dụng Tavily API để tìm kiếm thực tế"""
    search_query = state["search_query"]
    try:
        print(f"🔍 Đang tìm kiếm: {search_query}")
        response = tavily_client.search(
            query=search_query, search_depth="basic", max_results=5, include_answer=True
        )
        # ... (Xử lý và định dạng kết quả tìm kiếm) ...
        search_results = ... # Chuỗi kết quả đã được định dạng
        
        return {
            "search_results": search_results,
            "step": "searched",
            "messages": [AIMessage(content="✅ Tìm kiếm hoàn tất! Đang tổng hợp câu trả lời...")]
        }
    except Exception as e:
        # ... (Xử lý lỗi) ...
        return {
            "search_results": f"Tìm kiếm thất bại: {e}",
            "step": "search_failed",
            "messages": [AIMessage(content="❌ Tìm kiếm gặp sự cố...")]
        }
```

Node này thông qua `tavily_client.search` để thực hiện một lời gọi API thực sự. Nó được bọc trong khối `try...except`, dùng để bắt các ngoại lệ có thể xảy ra. Nếu việc tìm kiếm thất bại, nó sẽ cập nhật trạng thái `step` thành `"search_failed"`, trạng thái này sẽ được node tiếp theo sử dụng để kích hoạt phương án dự phòng (fallback).

（3）Node trả lời

Node trả lời cuối cùng có thể lựa chọn chiến lược trả lời khác nhau dựa trên việc tìm kiếm ở bước trước có thành công hay không, có một mức độ linh hoạt nhất định.

```python
def generate_answer_node(state: SearchState) -> dict:
    """Bước 3: Tạo câu trả lời cuối cùng dựa trên kết quả tìm kiếm"""
    if state["step"] == "search_failed":
        # Nếu tìm kiếm thất bại, thực thi chiến lược dự phòng, trả lời dựa trên kiến thức của chính LLM
        fallback_prompt = f"Search API tạm thời không khả dụng, vui lòng trả lời câu hỏi của người dùng dựa trên kiến thức của bạn:\nCâu hỏi người dùng: {state['user_query']}"
        response = llm.invoke([SystemMessage(content=fallback_prompt)])
    else:
        # Tìm kiếm thành công, tạo câu trả lời dựa trên kết quả tìm kiếm
        answer_prompt = f"""Dựa trên các kết quả tìm kiếm dưới đây, hãy cung cấp câu trả lời đầy đủ và chính xác cho người dùng:
Câu hỏi người dùng: {state['user_query']}
Kết quả tìm kiếm:\n{state['search_results']}
Vui lòng tổng hợp kết quả tìm kiếm, cung cấp câu trả lời chính xác và hữu ích..."""
        response = llm.invoke([SystemMessage(content=answer_prompt)])
    
    return {
        "final_answer": response.content,
        "step": "completed",
        "messages": [AIMessage(content=response.content)]
    }
```

Node này thực thi logic có điều kiện bằng cách kiểm tra giá trị của `state["step"]`. Nếu tìm kiếm thất bại, nó sẽ tận dụng kiến thức nội tại của LLM để trả lời và thông báo tình huống cho người dùng. Nếu tìm kiếm thành công, nó sẽ sử dụng prompt chứa kết quả tìm kiếm theo thời gian thực, để sinh ra một câu trả lời có tính thời sự và có căn cứ.

（4）Xây dựng đồ thị

Chúng ta kết nối tất cả các node lại với nhau.

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver

def create_search_assistant():
    workflow = StateGraph(SearchState)
    
    # Thêm các node
    workflow.add_node("understand", understand_query_node)
    workflow.add_node("search", tavily_search_node)
    workflow.add_node("answer", generate_answer_node)
    
    # Thiết lập quy trình tuyến tính
    workflow.add_edge(START, "understand")
    workflow.add_edge("understand", "search")
    workflow.add_edge("search", "answer")
    workflow.add_edge("answer", END)
    
    # Biên dịch đồ thị
    memory = InMemorySaver()
    app = workflow.compile(checkpointer=memory)
    return app
```

（5）Trình bày kết quả chạy case 

Sau khi chạy script này, bạn có thể đặt một vài câu hỏi cần thông tin theo thời gian thực, ví dụ như case trong Chương 1 của chúng ta: `Ngày mai tôi sẽ đi Bắc Kinh, thời tiết thế nào? Có điểm tham quan nào phù hợp không?`

Bạn sẽ thấy terminal hiển thị rõ ràng quá trình "suy nghĩ" của Agent:

```
🔍 Trợ lý tìm kiếm thông minh đã khởi động!
Tôi sẽ sử dụng Tavily API để tìm kiếm thông tin mới nhất và chính xác nhất cho bạn
Hỗ trợ nhiều loại câu hỏi: tin tức, công nghệ, hỏi đáp kiến thức, v.v.
(Nhập 'quit' để thoát)

🤔 Bạn muốn tìm hiểu điều gì: Ngày mai tôi sẽ đi Bắc Kinh, thời tiết thế nào? Có điểm tham quan nào phù hợp không?

============================================================
🧠 Giai đoạn hiểu ý định: Tôi hiểu nhu cầu của bạn: Hiểu ý định: Người dùng muốn biết tình hình thời tiết ngày mai tại Bắc Kinh cũng như gợi ý các điểm tham quan phù hợp.  
Từ khóa tìm kiếm: Bắc Kinh ngày mai thời tiết gợi ý điểm tham quan Beijing weather tomorrow attractions
🔍 Đang tìm kiếm: Bắc Kinh ngày mai thời tiết gợi ý điểm tham quan Beijing weather tomorrow attractions
🔍 Giai đoạn tìm kiếm: ✅ Tìm kiếm hoàn tất! Đã tìm thấy thông tin liên quan, đang tổng hợp câu trả lời cho bạn...

💡 Câu trả lời cuối cùng:
Dự báo thời tiết Bắc Kinh ngày mai (ngày 17 tháng 9 năm 2025) cho thấy trời dự kiến sẽ nhiều mây, nhiệt độ dao động từ 17°C (62°F) đến 25°C (77°F). Thời tiết ôn hòa như vậy rất thích hợp cho các hoạt động ngoài trời.

### Gợi ý các điểm tham quan phù hợp:
1. **Vạn Lý Trường Thành**: Là một trong những di tích lịch sử nổi tiếng nhất của Trung Quốc, Vạn Lý Trường Thành là nơi nhất định phải đến. Bạn có thể chọn tham quan các đoạn phổ biến như Bát Đạt Lĩnh hoặc Mộ Điền Dục.

2. **Cố Cung (Tử Cấm Thành)**: Cố Cung là hoàng cung của hai triều đại Minh và Thanh, sở hữu lịch sử và văn hóa phong phú, phù hợp cho những du khách yêu thích lịch sử Trung Quốc.

3. **Quảng trường Thiên An Môn**: Đây là một trong những biểu tượng của Trung Quốc, trên quảng trường có nhiều công trình kiến trúc và đài tưởng niệm quan trọng, rất thích hợp để chụp ảnh lưu niệm.

4. **Di Hòa Viên**: Một khu vườn hoàng gia vô cùng xinh đẹp, thích hợp để tản bộ và thưởng ngoạn phong cảnh thiên nhiên, đặc biệt là hồ nước và các công trình cổ kính.

5. **Khu nghệ thuật 798**: Nếu bạn quan tâm đến nghệ thuật hiện đại, Khu nghệ thuật 798 là một nơi hội tụ giữa nghệ thuật, văn hóa và sáng tạo, rất thích hợp để khám phá và chụp ảnh.

### Mẹo nhỏ:
- Vì ngày mai thời tiết tốt, bạn nên lên kế hoạch lộ trình di chuyển trước, đồng thời chuẩn bị một ít nước uống và đồ ăn nhẹ để duy trì đủ thể lực trong suốt chuyến tham quan.
- Do sự thay đổi của thời tiết có thể ảnh hưởng đến trải nghiệm tham quan, bạn nên theo dõi cập nhật thời tiết theo thời gian thực.

Hy vọng những thông tin này sẽ giúp bạn sắp xếp một chuyến đi Bắc Kinh thật vui vẻ! Nếu bạn cần thêm thông tin về các điểm tham quan hoặc lời khuyên du lịch, xin đừng ngần ngại hỏi bất cứ lúc nào.

============================================================

🤔 Bạn muốn tìm hiểu điều gì:
```

Và nó là một trợ lý có thể tương tác liên tục, bạn cũng có thể tiếp tục đặt câu hỏi cho nó.

### 6.5.3 Phân tích ưu điểm và hạn chế của LangGraph

Bất kỳ framework kỹ thuật nào cũng có kịch bản áp dụng đặc thù và sự đánh đổi trong thiết kế của riêng nó. Trong mục này, chúng ta sẽ phân tích một cách khách quan ưu điểm cốt lõi của LangGraph cũng như những hạn chế mà nó có thể gặp phải trong ứng dụng thực tế.

（1）Ưu điểm

- Như case trợ lý tìm kiếm thông minh của chúng ta đã thể hiện, LangGraph định nghĩa một cách tường minh (explicit) toàn bộ luồng hỏi đáp theo thời gian thực thành một "sơ đồ quy trình" (flowchart) được cấu thành từ state, node và edge. Ưu điểm lớn nhất của thiết kế này là <strong>tính kiểm soát được và khả năng dự đoán cao</strong>. Nhà phát triển có thể lên kế hoạch chính xác cho từng bước hành vi của Agent, điều này vô cùng quan trọng đối với việc xây dựng các ứng dụng cấp production cần độ tin cậy và khả năng kiểm toán (auditability) cao. Tính năng mạnh mẽ nhất của nó nằm ở việc <strong>hỗ trợ nguyên bản vòng lặp (Cycles)</strong>. Thông qua conditional edge, chúng ta có thể dễ dàng xây dựng vòng lặp "phản tư-sửa chữa", ví dụ trong case của chúng ta, nếu tìm kiếm thất bại, có thể thiết kế một đường dẫn quay lại (fallback) phương án dự phòng. Đây là chìa khóa để xây dựng Agent có khả năng tự tối ưu hóa và có khả năng chịu lỗi.

- Ngoài ra, do mỗi node đều là một hàm Python độc lập, điều này mang lại <strong>tính module hóa cao</strong>. Đồng thời, việc chèn thêm một node chờ con người xem xét (human review) vào trong quy trình cũng trở nên rất đơn giản, cung cấp nền tảng vững chắc để hiện thực hóa "con người tham gia vòng lặp" (Human-in-the-loop) đáng tin cậy.

（2）Hạn chế

- So với các framework dựa trên hội thoại, LangGraph đòi hỏi nhà phát triển phải viết nhiều <strong>code soạn sẵn (Boilerplate)</strong> hơn ở giai đoạn đầu. Việc định nghĩa state, node, edge và một loạt thao tác khác khiến quá trình phát triển trở nên rườm rà hơn đối với các nhiệm vụ đơn giản. Nhà phát triển cần suy nghĩ nhiều hơn về "cách kiểm soát luồng công việc (how)", thay vì chỉ đơn thuần "làm gì (what)". Do luồng công việc được định nghĩa trước, hành vi của LangGraph tuy có thể kiểm soát được, nhưng cũng thiếu đi sự tương tác mang tính "nảy sinh" (emergent) năng động như các Agent dạng hội thoại. Điểm mạnh của nó nằm ở việc thực thi một quy trình xác định, đáng tin cậy, chứ không phải mô phỏng sự hợp tác mang tính xã hội, mở và khó dự đoán.

- Quá trình debug cũng tồn tại thách thức tương tự. Mặc dù quy trình rõ ràng hơn so với lịch sử hội thoại, nhưng vấn đề có thể nằm ở nhiều khâu: lỗi logic bên trong một node nào đó, dữ liệu state bị biến đổi bất thường khi truyền giữa các node, hoặc phán đoán điều kiện chuyển tiếp của edge bị sai. Điều này đòi hỏi nhà phát triển phải có sự hiểu biết toàn cục về cơ chế vận hành của toàn bộ đồ thị.

## 6.6 Tổng kết chương

Trong chương này, chúng ta đã cảm nhận một số Agent Framework tiên phong nhất hiện nay, thông qua hình thức các case để trải nghiệm thực hành.

Chúng ta thấy rằng, mỗi framework đều có tư duy riêng của mình trong việc xây dựng Agent:

- <strong>AutoGen</strong> trừu tượng hóa sự hợp tác phức tạp thành một "cuộc nhóm chat" có thể tự động diễn ra với nhiều vai trò tham gia, cốt lõi của nó nằm ở việc "hợp tác được thúc đẩy bởi hội thoại".
- <strong>AgentScope</strong> thì tập trung vào tính vững chắc và khả năng mở rộng của ứng dụng cấp công nghiệp, cung cấp nền tảng kỹ thuật vững chắc để xây dựng các hệ thống đa Agent xử lý đồng thời cao, phân tán.
- <strong>CAMEL</strong> với mô hình "đóng vai" và "prompt khơi mào" nhẹ nhàng của mình, thể hiện cách dùng lượng code tối thiểu để khơi gợi sự hợp tác sâu sắc, tự chủ giữa hai Agent chuyên gia.
- <strong>LangGraph</strong> thì quay trở về mô hình "state machine" ở tầng thấp hơn, thông qua cấu trúc đồ thị tường minh, trao cho nhà phát triển quyền kiểm soát chính xác đối với luồng công việc, đặc biệt là khả năng vòng lặp của nó, mở đường cho việc xây dựng các Agent có thể phản tư, có thể sửa chữa.

Thông qua việc phân tích sâu các framework này, chúng ta có thể rút ra một sự đánh đổi mang tính thiết kế: <strong>sự lựa chọn giữa "hợp tác nảy sinh" (emergent collaboration) và "kiểm soát tường minh" (explicit control)</strong>. AutoGen và CAMEL phụ thuộc nhiều hơn vào việc định nghĩa "vai trò" và "mục tiêu" của Agent, để cho hành vi hợp tác phức tạp "nảy sinh" từ các quy tắc hội thoại đơn giản, cách làm này gần gũi hơn với mô hình tương tác của con người, nhưng đôi khi khó dự đoán và debug. Trong khi đó, LangGraph yêu cầu nhà phát triển định nghĩa rõ ràng từng bước và điều kiện chuyển tiếp, hy sinh một phần sự bất ngờ của tính "nảy sinh" để đổi lấy độ tin cậy, khả năng kiểm soát và khả năng quan sát cao. Đồng thời, AgentScope lại tiết lộ một chiều kích quan trọng không kém: <strong>kỹ thuật hóa (engineering)</strong>. Bất kể chúng ta lựa chọn mô hình hợp tác nào, để đưa nó từ prototype thử nghiệm sang ứng dụng production, đều phải đối mặt với các thách thức kỹ thuật như xử lý đồng thời, chịu lỗi, triển khai phân tán. AgentScope chính là được sinh ra để giải quyết những vấn đề này, nó đại diện cho bước nhảy vọt then chốt từ "có thể chạy được" sang "có thể phục vụ ổn định".

Tóm lại, Agent không chỉ có một cách xây dựng duy nhất. Hiểu sâu sắc triết lý thiết kế của các framework được khám phá trong chương này, sẽ giúp chúng ta không chỉ trở thành một "người sử dụng công cụ" tốt hơn, mà còn có thể hiểu được các ưu nhược điểm và sự đánh đổi khác nhau trong thiết kế framework.

Trong chương tiếp theo, chúng ta sẽ bước vào nội dung cốt lõi của giáo trình này, bắt đầu từ con số không, tự tay xây dựng một Agent Framework của riêng chúng ta, kết hợp nhuần nhuyễn toàn bộ lý thuyết và thực hành.


## Bài tập

1. Chương này giới thiệu bốn Agent Framework có đặc trưng riêng: `AutoGen`, `AgentScope`, `CAMEL` và `LangGraph`. Hãy phân tích:

   - Trong Bảng 6.1 ở mục 6.1.2, đã so sánh nhiều chiều kích của bốn framework này. Hãy chọn ra hai framework mà bạn quen thuộc nhất, so sánh sâu hơn theo ba chiều kích "mô hình hợp tác", "cách thức kiểm soát", "kịch bản áp dụng".
   - Chương này có đề cập đến sự đánh đổi giữa "hợp tác nảy sinh" và "kiểm soát tường minh", hãy trình bày cách hiểu của bạn về ý nghĩa của hai triết lý thiết kế này.
   
2. Trong case `AutoGen` ở mục 6.2, chúng ta đã xây dựng một "đội ngũ phát triển phần mềm". Hãy dựa trên case này để mở rộng suy nghĩ:

   > <strong>Gợi ý</strong>: Đây là một bài tập thực hành, khuyến khích bạn tự tay thực hiện

   - Đội ngũ hiện tại sử dụng mô hình `RoundRobinGroupChat` (nhóm chat quay vòng), các Agent phát biểu theo thứ tự cố định. Nếu yêu cầu thay đổi, code của Engineer cần được trả lại cho Product Manager xem xét lại, thì nên sửa đổi luồng hợp tác như thế nào? Hãy thiết kế một cơ chế hỗ trợ "quay lui động" (dynamic rollback).
   - Trong case, chúng ta đã định nghĩa vai trò và trách nhiệm cho mỗi Agent thông qua `System Message`. Hãy thử thêm một vai trò mới "Kỹ sư kiểm thử" (`Quality Assurance`) cho đội ngũ này, và thiết kế System Message của nó, để nó có thể thực hiện kiểm thử tự động hóa sau khi review code.
   - Sự hợp tác dạng hội thoại của `AutoGen` tồn tại khả năng bất ổn định, có thể khiến hội thoại lệch khỏi chủ đề hoặc rơi vào vòng lặp. Hãy suy nghĩ: làm sao để thiết kế một cơ chế "giám sát chất lượng hội thoại", can thiệp kịp thời khi phát hiện bất thường?

3. Trong case `AgentScope` ở mục 6.3, chúng ta đã hiện thực hóa trò chơi "Ma sói Tam Quốc". Hãy phân tích sâu:

   - Case sử dụng `MsgHub` (trung tâm tin nhắn) để quản lý giao tiếp giữa các Agent. Hãy giải thích ưu thế của kiến trúc hướng tin nhắn so với lời gọi hàm truyền thống là gì? Trong những kịch bản nào thì kiến trúc này đặc biệt có giá trị?
   - Trò chơi sử dụng Structured Output (như `DiscussionModelCN`, `WitchActionModelCN`) để ràng buộc hành vi của Agent. Hãy thiết kế một vai trò mới trong trò chơi là "Thợ săn" (Hunter), và định nghĩa model Structured Output tương ứng cho nó, bao gồm định nghĩa trường và quy tắc kiểm chứng.
   - `AgentScope` hỗ trợ triển khai phân tán, nghĩa là các Agent khác nhau có thể chạy trên các server khác nhau. Hãy suy nghĩ: trong kịch bản trò chơi thời gian thực như "Ma sói Tam Quốc", việc triển khai phân tán sẽ mang lại những thách thức kỹ thuật nào? Làm sao để đảm bảo tính tuần tự và tính nhất quán của tin nhắn?

4. Trong case `CAMEL` ở mục 6.4, chúng ta đã để nhà tâm lý học và nhà văn hợp tác sáng tác ebook.

   - Trong case, sự hợp tác sẽ bị buộc kết thúc khi phát hiện cờ hiệu `<CAMEL_TASK_DONE>`. Nhưng nếu hai Agent có ý kiến bất đồng (một bên cho rằng có thể kết thúc, một bên cho rằng không nên kết thúc), không thể đạt được sự đồng thuận thì sao? Hãy thiết kế một cơ chế tương thích để "giải quyết xung đột".
   - `CAMEL` ban đầu được thiết kế cho sự hợp tác song-Agent, nhưng hiện nay đã mở rộng để hỗ trợ đa Agent. Hãy tham khảo tài liệu mới nhất của `CAMEL`, tìm hiểu module hợp tác đa Agent [`workforce`](https://docs.camel-ai.org/key_modules/workforce) của nó, và kết hợp với sơ đồ kiến trúc để trình bày sự khác biệt của nó so với mô hình nhóm chat của `AutoGen`.
   
5. Trong case `LangGraph` ở mục 6.5, chúng ta đã xây dựng một "trợ lý hỏi đáp ba bước". Hãy phân tích:

   - `LangGraph` mô hình hóa luồng Agent thành state machine và đồ thị có hướng. Hãy vẽ cấu trúc đồ thị của luồng "hiểu-tìm kiếm-trả lời" trong case, đánh dấu node, edge và điều kiện chuyển đổi trạng thái.
   - Trợ lý hiện tại là một luồng công việc tuyến tính. Hãy mở rộng case này, thêm một node "phản tư" (reflection): nếu câu trả lời được sinh ra có chất lượng thấp (ví dụ quá ngắn gọn hoặc thiếu chi tiết), hệ thống nên tìm kiếm lại hoặc sinh lại câu trả lời. Hãy thiết kế logic conditional edge cho cơ chế vòng lặp này.
   - Ưu điểm của `LangGraph` nằm ở việc hỗ trợ nguyên bản vòng lặp. Hãy thiết kế một kịch bản ứng dụng phức tạp hơn, tận dụng triệt để tính năng này: ví dụ vòng lặp "sinh code - kiểm thử - sửa lỗi", vòng lặp "viết luận văn - xem xét - chỉnh sửa", v.v. Yêu cầu vẽ ra cấu trúc đồ thị hoàn chỉnh và trình bày chức năng của các node then chốt.

6. Lựa chọn framework là một trong những quyết định then chốt trong quá trình phát triển sản phẩm Agent. Giả sử bạn là kiến trúc sư kỹ thuật (technical architect) của một công ty `AI`, công ty có kế hoạch phát triển ba ứng dụng sản phẩm Agent sau đây, hãy chọn framework phù hợp nhất cho từng ứng dụng (`AutoGen`, `AgentScope`, `CAMEL`, `LangGraph` hoặc tự phát triển từ đầu không dùng framework), và trình bày chi tiết lý do:

   <strong>Ứng dụng A</strong>: Hệ thống chăm sóc khách hàng thông minh, cần xử lý lượng lớn yêu cầu đồng thời từ người dùng (1000+ mỗi giây), yêu cầu thời gian phản hồi dưới 2 giây, hệ thống cần vận hành ổn định 24/7, và hỗ trợ mở rộng theo chiều ngang (horizontal scaling).

   <strong>Ứng dụng B</strong>: Nền tảng hỗ trợ viết bài nghiên cứu khoa học, cần một "Agent nhà nghiên cứu" và một "Agent viết bài" hợp tác chuyên sâu, cùng hoàn thành việc tổng quan tài liệu (literature review), thiết kế thí nghiệm, phân tích dữ liệu và soạn thảo bài báo. Yêu cầu Agent có thể thảo luận chuyên sâu nhiều vòng, tự chủ thúc đẩy nhiệm vụ.

   <strong>Ứng dụng C</strong>: Hệ thống phê duyệt kiểm soát rủi ro tài chính (risk control), cần xử lý đơn xin vay theo quy trình nghiêm ngặt: kiểm tra hồ sơ → đánh giá rủi ro → tính toán hạn mức → kiểm tra tuân thủ (compliance) → xem xét thủ công (human review) → quyết định cuối cùng. Mỗi khâu đều có tiêu chuẩn phán đoán và logic nhánh rõ ràng, yêu cầu quy trình có thể truy vết, có thể kiểm toán.


## Tài liệu tham khảo

[1] Wu Q, Bansal G, Zhang J, et al. Autogen: Enabling next-gen LLM applications via multi-agent conversations[C]//First Conference on Language Modeling. 2024.

[2] Gao D, Li Z, Pan X, et al. Agentscope: A flexible yet robust multi-agent platform[J]. arXiv preprint arXiv:2402.14034, 2024.

[3] Li G, Hammoud H, Itani H, et al. Camel: Communicative agents for" mind" exploration of large language model society[J]. Advances in Neural Information Processing Systems, 2023, 36: 51991-52008.

[4] LangChain. LangGraph [EB/OL]. (2024). https://github.com/langchain-ai/langgraph.

[5] Microsoft. AutoGen - UserProxyAgent [EB/OL]. (2024). https://microsoft.github.io/autogen/stable/reference/python/autogen_agentchat.agents.html#autogen_agentchat.agents.UserProxyAgent.
