# Chương 6 Thực hành phát triển Framework

Trong Chương 4, chúng ta đã viết code gốc (native code) để hiện thực hóa luồng làm việc cốt lõi của các loại Agent ReAct, Plan-and-Solve và Reflection. Quá trình này giúp chúng ta hiểu được logic thực thi nội tại của Agent. Sau đó, ở Chương 5, chúng ta chuyển sang góc nhìn của "người sử dụng", trải nghiệm sự tiện lợi và hiệu quả mà các nền tảng low-code mang lại.

Mục tiêu của chương này là khám phá cách sử dụng một số <strong>Agent Framework</strong> phổ biến trong ngành, để xây dựng các ứng dụng Agent đáng tin cậy một cách hiệu quả và có quy chuẩn. Trước tiên chúng ta sẽ tổng quan về các Agent Framework chủ đạo hiện có trên thị trường, sau đó thông qua các case thực hành hoàn chỉnh với một vài framework tiêu biểu, để trải nghiệm mô hình phát triển dựa trên framework (framework-driven development).

## 6.1 Từ triển khai thủ công đến phát triển bằng Framework

Từ việc viết các script dùng một lần đến việc sử dụng một framework trưởng thành, đó là một bước nhảy vọt quan trọng trong tư duy của lĩnh vực kỹ thuật phần mềm. Code chúng ta viết ở Chương 4 chủ yếu nhằm mục đích giảng dạy và giúp hiểu rõ bản chất. Chúng có thể hoàn thành tốt các nhiệm vụ cụ thể, nhưng nếu dùng chúng để xây dựng nhiều ứng dụng Agent khác loại và có logic phức tạp, chúng ta sẽ nhanh chóng gặp phải điểm nghẽn (bottleneck).

Bản chất của một framework là cung cấp một bộ "quy chuẩn" đã được kiểm chứng. Nó trừu tượng hóa (abstract) và đóng gói (encapsulate) những công việc chung, lặp đi lặp lại của tất cả các Agent (như vòng lặp chính, quản lý trạng thái, gọi tool, ghi log, v.v.), giúp chúng ta khi xây dựng một Agent mới có thể tập trung vào logic nghiệp vụ đặc thù của nó, thay vì phần triển khai nền tảng (底层实现) mang tính thông dụng.

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

Việc phát hành phiên bản `0.7.4` là một cột mốc quan trọng trong sự phát triển của AutoGen, nó đánh dấu một cuộc cách tân căn bản trong thiết kế nền tảng (底层设计) của framework. Lần cập nhật này không đơn thuần là chồng thêm tính năng, mà là một sự suy nghĩ lại toàn diện về kiến trúc tổng thể, nhằm nâng cao tính module hóa, hiệu năng xử lý đồng thời (concurrency) và trải nghiệm của nhà phát triển.

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
    system_message = """你是一位经验丰富的产品经理，专门负责软件产品的需求分析和项目规划。

你的核心职责包括：
1. **需求分析**：深入理解用户需求，识别核心功能和边界条件
2. **技术规划**：基于需求制定清晰的技术实现路径
3. **风险评估**：识别潜在的技术风险和用户体验问题
4. **协调沟通**：与工程师和其他团队成员进行有效沟通

当接到开发任务时，请按以下结构进行分析：
1. 需求理解与分析
2. 功能模块划分
3. 技术选型建议
4. 实现优先级排序
5. 验收标准定义

请简洁明了地回应，并在分析完成后说"请工程师开始实现"。"""

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
    """创建软件工程师智能体"""
    system_message = """你是一位资深的软件工程师，擅长 Python 开发和 Web 应用构建。

你的技术专长包括：
1. **Python 编程**：熟练掌握 Python 语法和最佳实践
2. **Web 开发**：精通 Streamlit、Flask、Django 等框架
3. **API 集成**：有丰富的第三方 API 集成经验
4. **错误处理**：注重代码的健壮性和异常处理

当收到开发任务时，请：
1. 仔细分析技术需求
2. 选择合适的技术方案
3. 编写完整的代码实现
4. 添加必要的注释和说明
5. 考虑边界情况和异常处理

请提供完整的可运行代码，并在完成后说"请代码审查员检查"。"""

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
    """创建代码审查员智能体"""
    system_message = """你是一位经验丰富的代码审查专家，专注于代码质量和最佳实践。

你的审查重点包括：
1. **代码质量**：检查代码的可读性、可维护性和性能
2. **安全性**：识别潜在的安全漏洞和风险点
3. **最佳实践**：确保代码遵循行业标准和最佳实践
4. **错误处理**：验证异常处理的完整性和合理性

审查流程：
1. 仔细阅读和理解代码逻辑
2. 检查代码规范和最佳实践
3. 识别潜在问题和改进点
4. 提供具体的修改建议
5. 评估代码的整体质量

请提供具体的审查意见，完成后说"代码审查完成，请用户代理测试"。"""

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
    """创建用户代理智能体"""
    return UserProxyAgent(
        name="UserProxy",
        description="""用户代理，负责以下职责：
1. 代表用户提出开发需求
2. 执行最终的代码实现
3. 验证功能是否符合预期
4. 提供用户反馈和建议

完成测试后请回复 TERMINATE。""",
    )
```

Thông qua bốn hàm định nghĩa độc lập này, chúng ta không chỉ xây dựng được một "đội ngũ ảo" đầy đủ chức năng, mà còn thể hiện rằng việc "prompt engineering" thông qua System Message chính là khâu cốt lõi để thiết kế một ứng dụng đa Agent hiệu quả.

（3）Định nghĩa luồng hợp tác nhóm

Trong case này, quy trình phát triển phần mềm tương đối cố định (yêu cầu -> code -> review -> test), do đó `RoundRobinGroupChat` (nhóm chat quay vòng) là lựa chọn lý tưởng. Chúng ta thêm bốn Agent vào danh sách người tham gia theo thứ tự logic nghiệp vụ.

```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# 定义团队聊天和协作规则
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
    # ... 初始化客户端和智能体 ...
    
    # 定义任务描述
    task = """我们需要开发一个比特币价格显示应用，具体要求如下：
            核心功能：
            - 实时显示比特币当前价格（USD）
            - 显示24小时价格变化趋势（涨跌幅和涨跌额）
            - 提供价格刷新功能

            技术要求：
            - 使用 Streamlit 框架创建 Web 应用
            - 界面简洁美观，用户友好
            - 添加适当的错误处理和加载状态

            请团队协作完成这个任务，从需求分析到最终实现。"""
    
    # 异步执行团队协作，并流式输出对话过程
    result = await Console(team_chat.run_stream(task=task))
    return result

# 主程序入口
if __name__ == "__main__":
    result = asyncio.run(run_software_development_team())
```

Khi chương trình được chạy, `task` được truyền vào `team_chat` như một tin nhắn khởi đầu, Product Manager với tư cách người tham gia đầu tiên sẽ nhận được tin nhắn này, sau đó toàn bộ luồng hợp tác tự động hóa bắt đầu diễn ra.

（5）Hiệu quả hợp tác dự kiến

Khi chúng ta chạy đội ngũ phát triển phần mềm này, có thể quan sát được một luồng hợp tác hoàn chỉnh:

```bash
🔧 正在初始化模型客户端...
👥 正在创建智能体团队...
🚀 启动 AutoGen 软件开发团队协作...
============================================================
---------- TextMessage (user) ----------
我们需要开发一个比特币价格显示应用，具体要求如下：
...
请团队协作完成这个任务，从需求分析到最终实现。
---------- TextMessage (ProductManager) ----------
### 1. 需求理解与分析
...
请工程师开始实现。
---------- TextMessage (Engineer) ----------
### 技术方案实施
...
请代码审查员检查。
---------- TextMessage (CodeReviewer) ----------
### 代码审查
...
代码审查完成，请用户代理测试。    
---------- TextMessage (UserProxy) ----------
已经完成需求
---------- TextMessage (ProductManager) ----------
太好了，感谢您的反馈！如果在使用过程中有任何问题，或者有其他功能需求和改进建议，请随时告知我们。我们会持续提供支持和改进。期待您对我们的应用
有愉快的使用体验！
---------- TextMessage (Engineer) ----------
很高兴听到项目顺利完成。如果您或用户有任何问题或者需要帮助，请随时联系我们。感谢您对我们工作的支持，让我们一起确保应用稳定运行并不断优化用户
体验！
---------- TextMessage (CodeReviewer) ----------
非常感谢大家的努力与协作，使得项目能够顺利完成。未来若有更多技术支持的需求或者需要改进的地方，我们愿意为项目的持续优化贡
献力量。期待用户能够享受到流畅的体验，同时也欢迎提出更多的反馈与建议。再次感谢团队的合作！
---------- TextMessage (UserProxy) ----------
Enter your response: TERMINATE
============================================================
✅ 团队协作完成！

📋 协作结果摘要：
- 参与智能体数量：4个
- 任务完成状态：成功
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

# 消息的标准结构
message = Msg(
    name="Alice",           # 发送者名称
    content="Hello, Bob!",  # 消息内容
    role="user",           # 角色类型
    metadata={             # 元数据信息
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
        # 智能体初始化逻辑
    
    def reply(self, x: Msg) -> Msg:
        # 智能体的核心响应逻辑
        response = self.model(x.content)
        return Msg(name=self.name, content=response, role="assistant")
    
    def observe(self, x: Msg) -> None:
        # 智能体的观察逻辑（可选）
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

Để hiểu sâu sắc kiến trúc hướng tin nhắn và năng lực hợp tác đa Agent của AgentScope, chúng ta sẽ xây dựng trò chơi "Ma sói Tam Quốc" (三国狼人杀) kết hợp yếu tố văn hóa cổ điển Trung Hoa. Case này không chỉ thể hiện ưu thế của AgentScope trong việc xử lý tương tác đa Agent phức tạp, mà quan trọng hơn, nó minh họa cách phát huy sức mạnh của kiến trúc hướng tin nhắn trong một kịch bản cần <strong>hợp tác thời gian thực</strong>, <strong>đóng vai (role-playing)</strong> và <strong>đấu trí chiến lược (strategic game)</strong>. Khác với Ma sói truyền thống, "Ma sói Tam Quốc" của chúng ta đưa các nhân vật kinh điển như Lưu Bị, Quan Vũ, Gia Cát Lượng vào trò chơi, mỗi Agent không chỉ cần hoàn thành nhiệm vụ cơ bản của Ma sói (như sói giết người, tiên tri kiểm tra, dân làng suy luận), mà còn phải thể hiện được đặc điểm tính cách và mô thức hành vi của nhân vật Tam Quốc tương ứng. Thiết kế này cho phép chúng ta quan sát được cách AgentScope xử lý <strong>việc mô hình hóa vai trò đa tầng</strong>.

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
    """狼人阶段 - 展示消息驱动的协作模式"""
    if not self.werewolves:
        return None
        
    # 通过消息中心建立狼人专属通信频道
    async with MsgHub(
        self.werewolves,
        enable_auto_broadcast=True,
        announcement=await self.moderator.announce(
            f"狼人们，请讨论今晚的击杀目标。存活玩家：{format_player_list(self.alive_players)}"
        ),
    ) as werewolves_hub:
        # 讨论阶段：狼人通过消息交换策略
        for _ in range(MAX_DISCUSSION_ROUND):
            for wolf in self.werewolves:
                await wolf(structured_model=DiscussionModelCN)
        
        # 投票阶段：收集并统计狼人的击杀决策
        werewolves_hub.set_auto_broadcast(False)
        kill_votes = await fanout_pipeline(
            self.werewolves,
            msg=await self.moderator.announce("请选择击杀目标"),
            structured_model=WerewolfKillModelCN,
            enable_gather=False,
        )
```

Ưu điểm của thiết kế này là, logic trò chơi được biểu đạt một cách rõ ràng là "trong ngữ cảnh cụ thể, tiến hành trao đổi tin nhắn theo mô thức nào", thay vì một chuỗi chuyển đổi trạng thái cứng nhắc. Các giai đoạn như thảo luận ban ngày (broadcast toàn bộ), tiên tri kiểm tra (yêu cầu point-to-point) cũng đều tuân theo cùng một mô hình thiết kế.

（3）Dùng Structured Output để ràng buộc luật chơi

Một thách thức then chốt của trò chơi Ma sói là làm sao đảm bảo hành vi của Agent tuân thủ luật chơi. <strong>Cơ chế Structured Output</strong> của AgentScope cung cấp giải pháp cho vấn đề này. Chúng ta định nghĩa các data model (mô hình dữ liệu) nghiêm ngặt cho các hành vi trò chơi khác nhau:

```python
class DiscussionModelCN(BaseModel):
    """讨论阶段的输出格式"""
    reach_agreement: bool = Field(
        description="是否已达成一致意见",
        default=False
    )
    confidence_level: int = Field(
        description="对当前推理的信心程度(1-10)",
        ge=1, le=10,
        default=5
    )
    key_evidence: Optional[str] = Field(
        description="支持你观点的关键证据",
        default=None
    )

class WitchActionModelCN(BaseModel):
    """女巫行动的输出格式"""
    use_antidote: bool = Field(description="是否使用解药")
    use_poison: bool = Field(description="是否使用毒药")
    target_name: Optional[str] = Field(description="毒药目标玩家姓名")
```

Thông qua cách này, chúng ta không chỉ đảm bảo được <strong>tính nhất quán về định dạng (format consistency)</strong> trong đầu ra của Agent, mà quan trọng hơn là hiện thực hóa được <strong>việc ràng buộc tự động hóa luật chơi</strong>. Ví dụ, Agent phù thủy không thể vừa dùng thuốc giải vừa dùng thuốc độc cho cùng một mục tiêu trong cùng lúc, tiên tri mỗi đêm chỉ có thể kiểm tra một người chơi, những ràng buộc này đều được tự động thực thi thông qua định nghĩa trường (field) của data model và logic kiểm chứng (validation).

（4）Thách thức kép trong việc mô hình hóa vai trò

Trong case này, thách thức kỹ thuật thú vị nhất là làm sao để Agent có thể đóng tốt vai trò ở cả hai tầng: <strong>vai trò chức năng trong trò chơi</strong> (sói, tiên tri, v.v.) và <strong>vai trò nhân cách văn hóa</strong> (Lưu Bị, Tào Tháo, v.v.). Chúng ta giải quyết vấn đề này thông qua prompt engineering:

```python
def get_role_prompt(role: str, character: str) -> str:
    """获取角色提示词 - 融合游戏规则与人物性格"""
    base_prompt = f"""你是{character}，在这场三国狼人杀游戏中扮演{role}。

重要规则：
1. 你只能通过对话和推理参与游戏
2. 不要尝试调用任何外部工具或函数
3. 严格按照要求的JSON格式回复

角色特点：
"""
    
    if role == "狼人":
        return base_prompt + f"""
- 你是狼人阵营，目标是消灭所有好人
- 夜晚可以与其他狼人协商击杀目标
- 白天要隐藏身份，误导好人
- 以{character}的性格说话和行动
"""
```

Thiết kế này giúp chúng ta quan sát được một hiện tượng thú vị: các nhân vật Tam Quốc khác nhau khi đóng cùng một vai trò trong trò chơi sẽ thể hiện chiến lược và phong cách lời nói hoàn toàn khác nhau. Ví dụ, "Tào Tháo" khi đóng vai sói có thể tỏ ra xảo quyệt và giỏi ngụy trang hơn, còn "Trương Phi" khi đóng vai sói thì có thể tỏ ra trực diện và bốc đồng hơn.

（5）Xử lý đồng thời và cơ chế chịu lỗi (fault tolerance)

Kiến trúc bất đồng bộ của AgentScope đóng vai trò quan trọng trong trò chơi đa Agent này. Trò chơi thường xuyên xuất hiện các kịch bản cần <strong>thu thập quyết định của nhiều Agent cùng lúc</strong>, ví dụ như giai đoạn bỏ phiếu:

```python
# 并行收集所有玩家的投票决策
vote_msgs = await fanout_pipeline(
    self.alive_players,
    await self.moderator.announce("请投票选择要淘汰的玩家"),
    structured_model=get_vote_model_cn(self.alive_players),
    enable_gather=False,
)
```

`fanout_pipeline` cho phép chúng ta gửi cùng một tin nhắn đến tất cả các Agent song song, và thu thập phản hồi của chúng một cách bất đồng bộ. Điều này không chỉ nâng cao hiệu quả thực thi của trò chơi, mà quan trọng hơn là mô phỏng được kịch bản "bỏ phiếu đồng thời" trong trò chơi Ma sói thực tế. Đồng thời, chúng ta bổ sung xử lý chịu lỗi (fault tolerance) tại các khâu then chốt:

```python
try:
    response = await wolf(
        "请分析当前局势并表达你的观点。",
        structured_model=DiscussionModelCN
    )
except Exception as e:
    print(f"⚠️ {wolf.name} 讨论时出错: {e}")
    # 创建默认响应，确保游戏继续进行
    default_response = DiscussionModelCN(
        reach_agreement=False,
        confidence_level=5,
        key_evidence="暂时无法分析"
    )
```

Thiết kế này đảm bảo rằng ngay cả khi một Agent nào đó xảy ra lỗi ngoại lệ, toàn bộ luồng trò chơi vẫn có thể tiếp tục diễn ra.

（6）Kết quả đầu ra và tổng kết case

Để cảm nhận trực quan hơn về cơ chế vận hành của AgentScope, dưới đây là một đoạn log vận hành thực tế trích từ giai đoạn ban đêm của trò chơi, thể hiện quá trình hai Agent sói đóng vai "Tôn Quyền" và "Chu Du" thương lượng bí mật và thực hiện việc giết người.

```
🎮 欢迎来到三国狼人杀！

=== 游戏初始化 ===
游戏主持人: 📢 【孙权】你在这场三国狼人杀中扮演狼人，你的角色是孙权。夜晚可以击杀一名玩家
游戏主持人: 📢 【周瑜】你在这场三国狼人杀中扮演狼人，你的角色是周瑜。夜晚可以击杀一名玩家
...

游戏主持人: 📢 三国狼人杀游戏开始！参与者：孙权、周瑜、曹操、张飞、司马懿、赵云
✅ 游戏设置完成，共6名玩家

=== 第1轮游戏 ===
🌙 第1夜降临，天黑请闭眼...

【狼人阶段】
游戏主持人: 📢 🐺 狼人请睁眼，选择今晚要击杀的目标...
游戏主持人: 📢 狼人们，请讨论今晚的击杀目标。存活玩家：孙权、周瑜、曹操、张飞、司马懿、赵云

孙权: 今晚我们应该除掉周瑜，此人智谋过人，对我们威胁很大。
周瑜: 孙权，你言之有理。但周瑜虽智，却未必是今晚的最大威胁。曹操势力庞大，若不尽早除去，恐对我们不利。
孙权: 曹操的确是个威胁，但周瑜若活着，他能够识破我们的计谋。不如先解决眼前的隐患。
周瑜: 孙权，你的顾虑不无道理。但曹操若与我们为敌，他可以联合其他势力对我们构成更大的威胁。
孙权: 你说的也有道理，曹操的联合确实麻烦。那我们就先对付曹操吧。
周瑜: 很好，孙权。曹操才是我们今晚首要的目标。

游戏主持人: 📢 请选择击杀目标
孙权: 我同意，曹操必须被除掉。
周瑜: 我同意，曹操是我们今晚要解决的目标。

【预言家阶段】
游戏主持人: 📢 🔮 预言家请睁眼，选择要查验的玩家...
曹操: 我要查验孙权。
游戏主持人: 📢 查验结果：孙权是狼人

【女巫阶段】
游戏主持人: 📢 🧙‍♀️ 女巫请睁眼...
游戏主持人: 📢 今晚曹操被狼人击杀
张飞: 我昨晚使用了解药救了曹操，现在解药已经用掉了。
游戏主持人: 📢 你使用解药救了曹操

游戏主持人: 📢 昨夜平安无事，无人死亡。

【白天讨论阶段】
游戏主持人: 📢 ☀️ 第1天天亮了，请大家睁眼...
游戏主持人: 📢 现在开始自由讨论。存活玩家：孙权、周瑜、曹操、张飞、司马懿、赵云

孙权: 诸位，曹操势力庞大，对我们都是潜在的威胁。今晚我建议我们集中力量对付他。
周瑜: 孙权所言极是，曹操不仅自身强大，还可能与其他玩家结盟，对我们构成更大的威胁。
曹操: 我昨晚查验了孙权，本以为他是好人，但游戏主持人给出的结果却是狼人。这说明有狼人在说谎。
张飞: 我昨晚确实救了曹操，说明他是被狼人袭击的。但曹操查验孙权的结果令人怀疑。
司马懿: 曹操的查验结果和张飞的救人行动似乎存在矛盾，我们需要更多的信息来判断谁是狼人。
赵云: 情况确实复杂，我们需要仔细分析各方的发言。

【投票阶段】
游戏主持人: 📢 请投票选择要淘汰的玩家

孙权: 曹操的威胁依然很大，我坚持认为应该投票给他。
周瑜: 基于昨晚的情况，我认为我们应该先投票给曹操，他的威胁最大。
曹操: 我选择投票给孙权。根据游戏主持人的反馈，孙权确实是狼人。
张飞: 我坚持昨晚救了曹操的事实，但孙权被查出是狼人这一点让我感到困惑。
司马懿: 我们需要更多的信息来判断谁是狼人。

[游戏继续...]
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

#创建模型,在这里以Qwen为例,调用的百炼大模型平台API
model = ModelFactory.create(
    model_platform=ModelPlatformType.QWEN,
    model_type=LLM_MODEL,
    url=LLM_BASE_URL,
    api_key=LLM_API_KEY
)

# 定义协作任务
task_prompt = """
创作一本关于"拖延症心理学"的短篇电子书，目标读者是对心理学感兴趣的普通大众。
要求：
1. 内容科学严谨，基于实证研究
2. 语言通俗易懂，避免过多专业术语
3. 包含实用的改善建议和案例分析
4. 篇幅控制在8000-10000字
5. 结构清晰，包含引言、核心章节和总结
"""

print(Fore.YELLOW + f"协作任务:\n{task_prompt}\n")
```

`task_prompt` chính là "bản thuyết minh nhiệm vụ" của toàn bộ sự hợp tác. Nó không chỉ là mục tiêu chúng ta cần hoàn thành, mà còn được CAMEL sử dụng ở phía sau hậu trường để tạo ra "Prompt khơi mào", đảm bảo hội thoại của hai Agent luôn xoay quanh mục tiêu cốt lõi này.

（3）Khởi tạo "xã hội" đóng vai

Tiếp theo, chúng ta tạo một instance phiên `RolePlaying`. Đây là thao tác cốt lõi của CAMEL, nó dựa trên vai trò và nhiệm vụ mà chúng ta cung cấp, nhanh chóng xây dựng một "xã hội" hợp tác song-Agent.

```python
# 初始化角色扮演会话
# AI 作家作为 "user"，负责提出写作结构和要求
# AI 心理学家作为 "assistant"，负责提供专业知识和内容
role_play_session = RolePlaying(
    assistant_role_name="心理学家",
    user_role_name="作家",
    task_prompt=task_prompt,
    model=model,
    with_task_specify=False, # 在本例中，我们直接使用给定的task_prompt
)

print(Fore.CYAN + f"具体任务描述:\n{role_play_session.task_prompt}\n")
```

`RolePlaying` là một API cấp cao mà CAMEL cung cấp, nó đóng gói prompt engineering phức tạp. Chúng ta chỉ cần truyền vào tên của hai vai trò và nhiệm vụ. Trong thiết kế của CAMEL, vai trò `user` là "người thúc đẩy" và "bên có nhu cầu" của hội thoại, còn vai trò `assistant` là "người thực thi" và "bên cung cấp giải pháp". Do đó, chúng ta gán "nhà văn" chịu trách nhiệm lên kế hoạch cấu trúc cho `user_role_name`, gán "nhà tâm lý học" chịu trách nhiệm cung cấp kiến thức chuyên môn cho `assistant_role_name`.

（4）Khởi động và chạy hội thoại tự động hóa

Cuối cùng, chúng ta viết một vòng lặp để thúc đẩy toàn bộ quá trình hội thoại, để hai chuyên gia AI bắt đầu quá trình hợp tác tự động hóa của mình.

```python
# 开始协作对话
chat_turn_limit, n = 30, 0
# 调用 init_chat() 来获得由 AI 生成的初始对话消息
input_msg = role_play_session.init_chat()

while n < chat_turn_limit:
    n += 1
    # step() 方法驱动一轮完整的对话，AI 用户和 AI 助理各发言一次
    assistant_response, user_response = role_play_session.step(input_msg)
    
    # 检查是否有消息返回，防止对话提前终止
    if assistant_response.msg is None or user_response.msg is None:
        break
    
    print_text_animated(Fore.BLUE + f"作家 (AI User):\n\n{user_response.msg.content}\n")
    print_text_animated(Fore.GREEN + f"心理学家 (AI Assistant):\n\n{assistant_response.msg.content}\n")
    
    # 检查任务完成标志
    if "<CAMEL_TASK_DONE>" in user_response.msg.content or "<CAMEL_TASK_DONE>" in assistant_response.msg.content:
        print(Fore.MAGENTA + "✅ 电子书创作完成！")
        break
    
    # 将助理的回复作为下一轮对话的输入
    input_msg = assistant_response.msg

print(Fore.YELLOW + f"总共进行了 {n} 轮协作对话")
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
协作任务:
创作一本关于"拖延症心理学"的短篇电子书，目标读者是对心理学感兴趣的普通大众。
要求：
1. 内容科学严谨，基于实证研究
2. 语言通俗易懂，避免过多专业术语
3. 包含实用的改善建议和案例分析
4. 篇幅控制在8000-10000字
5. 结构清晰，包含引言、核心章节和总结

具体任务描述:
为普通大众撰写8000–10000字短篇电子书《拖延症心理学》：实证为本、通俗易懂。结构：引言、成因（认知/情绪/奖励）、动机与决策、习惯形成与干预、实
用策略与练习、三则案例分析、总结与资源。每章含研究引用与可操作步骤。

作家:
Instruction: 请为电子书的"引言"章节撰写一段400–600字的中文草稿...
Input: None

心理学家:
Solution:
草稿：拖延，是指明知应当完成某项任务却反复推迟或回避的行为与内在倾向。它既可以是偶发的时间管理问题...

Next request.

作家:
Instruction: 请把下面的引言草稿修订为一段450–550字的中文文本...
Input: 草稿：拖延，是指明知应当完成某项任务却反复推迟或回避的行为...
.....
```

### 6.4.3 Phân tích ưu điểm và hạn chế của CAMEL

Thông qua case sáng tạo ebook ở trên, chúng ta đã trải nghiệm sâu sắc mô hình đóng vai độc đáo của framework CAMEL. Bây giờ hãy phân tích một cách khách quan ưu điểm và hạn chế của triết lý thiết kế này, để có thể đưa ra lựa chọn kỹ thuật sáng suốt trong các dự án thực tế.

（1）Ưu điểm

Ưu điểm lớn nhất của CAMEL nằm ở triết lý thiết kế "kiến trúc nhẹ, prompt nặng" (轻架构、重提示). So với việc quản lý hội thoại phức tạp của AutoGen và kiến trúc phân tán của AgentScope, CAMEL chỉ cần thông qua prompt khởi đầu được thiết kế tỉ mỉ đã có thể hiện thực hóa sự hợp tác Agent chất lượng cao. Hành vi hợp tác nảy sinh tự nhiên (emergent) này thường linh hoạt và hiệu quả hơn so với luồng công việc được hard-code.

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

# 定义全局状态的数据结构
class AgentState(TypedDict):
    messages: List[str]      # 对话历史
    current_task: str        # 当前任务
    final_answer: str        # 最终答案
    # ... 任何其他需要追踪的状态
```

<strong>Thứ hai, là Node</strong>. Mỗi node đều là một hàm Python nhận trạng thái hiện tại làm đầu vào, và trả về một trạng thái đã được cập nhật làm đầu ra. Node là đơn vị thực thi công việc cụ thể.

```python
# 定义一个"规划者"节点函数
def planner_node(state: AgentState) -> AgentState:
    """根据当前任务制定计划，并更新状态。"""
    current_task = state["current_task"]
    # ... 调用LLM生成计划 ...
    plan = f"为任务 '{current_task}' 生成的计划..."
    
    # 将新消息追加到状态中
    state["messages"].append(plan)
    return state

# 定义一个"执行者"节点函数
def executor_node(state: AgentState) -> AgentState:
    """执行最新计划，并更新状态。"""
    latest_plan = state["messages"][-1]
    # ... 执行计划并获得结果 ...
    result = f"执行计划 '{latest_plan}' 的结果..."
    
    state["messages"].append(result)
    return state
```

<strong>Cuối cùng, là Edge (cạnh)</strong>. Edge chịu trách nhiệm kết nối các node, định nghĩa hướng đi của luồng công việc. Edge đơn giản nhất là edge thông thường, nó chỉ định rằng đầu ra của một node luôn chảy sang một node cố định khác. Còn tính năng mạnh mẽ nhất của LangGraph nằm ở <strong>Conditional Edge (cạnh có điều kiện)</strong>. Nó thông qua một hàm để phán đoán trạng thái hiện tại, sau đó quyết định động xem bước tiếp theo nên nhảy tới node nào. Đây chính là chìa khóa để hiện thực hóa vòng lặp và các nhánh logic phức tạp.

```python
def should_continue(state: AgentState) -> str:
    """条件函数：根据状态决定下一步路由。"""
    # 假设如果消息少于3条，则需要继续规划
    if len(state["messages"]) < 3:
        # 返回的字符串需要与添加条件边时定义的键匹配
        return "continue_to_planner"
    else:
        state["final_answer"] = state["messages"][-1]
        return "end_workflow"
```

Sau khi đã định nghĩa State, Node và Edge, chúng ta có thể lắp ráp chúng thành một luồng công việc có thể thực thi, giống như lắp ráp các khối lego.

```python
from langgraph.graph import StateGraph, END

# 初始化一个状态图，并绑定我们定义的状态结构
workflow = StateGraph(AgentState)

# 将节点函数添加到图中
workflow.add_node("planner", planner_node)
workflow.add_node("executor", executor_node)

# 设置图的入口点
workflow.set_entry_point("planner")

# 添加常规边，连接 planner 和 executor
workflow.add_edge("planner", "executor")

# 添加条件边，实现动态路由
workflow.add_conditional_edges(
    # 起始节点
    "executor",
    # 判断函数
    should_continue,
    # 路由映射：将判断函数的返回值映射到目标节点
    {
        "continue_to_planner": "planner", # 如果返回"continue_to_planner"，则跳回planner节点
        "end_workflow": END               # 如果返回"end_workflow"，则结束流程
    }
)

# 编译图，生成可执行的应用
app = workflow.compile()

# 运行图
inputs = {"current_task": "分析最近的AI行业新闻", "messages": []}
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
    user_query: str      # 经过LLM理解后的用户需求总结
    search_query: str    # 优化后用于Tavily API的搜索查询
    search_results: str  # Tavily搜索返回的结果
    final_answer: str    # 最终生成的答案
    step: str            # 标记当前步骤
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

# 加载 .env 文件中的环境变量
load_dotenv()

# 初始化模型
# 我们将使用这个 llm 实例来驱动所有节点的智能
llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL_ID", "gpt-4o-mini"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1"),
    temperature=0.7
)
# 初始化Tavily客户端
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
```

Bây giờ, chúng ta lần lượt tạo ba node cốt lõi.

（1） Node hiểu và truy vấn

Node này là bước đầu tiên của luồng công việc, trách nhiệm của node này là hiểu ý định của người dùng, và tạo ra một câu truy vấn tìm kiếm được tối ưu hóa cho nó.

```python
def understand_query_node(state: SearchState) -> dict:
    """步骤1：理解用户查询并生成搜索关键词"""
    user_message = state["messages"][-1].content
    
    understand_prompt = f"""分析用户的查询："{user_message}"
请完成两个任务：
1. 简洁总结用户想要了解什么
2. 生成最适合搜索引擎的关键词（中英文均可，要精准）

格式：
理解：[用户需求总结]
搜索词：[最佳搜索关键词]"""

    response = llm.invoke([SystemMessage(content=understand_prompt)])
    response_text = response.content
    
    # 解析LLM的输出，提取搜索关键词
    search_query = user_message # 默认使用原始查询
    if "搜索词：" in response_text:
        search_query = response_text.split("搜索词：")[1].strip()
    
    return {
        "user_query": response_text,
        "search_query": search_query,
        "step": "understood",
        "messages": [AIMessage(content=f"我将为您搜索：{search_query}")]
    }
```

Node này thông qua một prompt có cấu trúc, yêu cầu LLM đồng thời hoàn thành hai nhiệm vụ "hiểu ý định" và "sinh từ khóa", và cập nhật từ khóa tìm kiếm chuyên dụng được phân tích vào trường `search_query` của state, chuẩn bị cho việc tìm kiếm chính xác ở bước tiếp theo.

（2）Node tìm kiếm

Node này chịu trách nhiệm thực thi khả năng "sử dụng tool" của Agent, nó sẽ gọi Tavily API để thực hiện tìm kiếm internet thực sự, và có sẵn chức năng xử lý lỗi cơ bản.

```python
def tavily_search_node(state: SearchState) -> dict:
    """步骤2：使用Tavily API进行真实搜索"""
    search_query = state["search_query"]
    try:
        print(f"🔍 正在搜索: {search_query}")
        response = tavily_client.search(
            query=search_query, search_depth="basic", max_results=5, include_answer=True
        )
        # ... (处理和格式化搜索结果) ...
        search_results = ... # 格式化后的结果字符串
        
        return {
            "search_results": search_results,
            "step": "searched",
            "messages": [AIMessage(content="✅ 搜索完成！正在整理答案...")]
        }
    except Exception as e:
        # ... (处理错误) ...
        return {
            "search_results": f"搜索失败：{e}",
            "step": "search_failed",
            "messages": [AIMessage(content="❌ 搜索遇到问题...")]
        }
```

Node này thông qua `tavily_client.search` để thực hiện một lời gọi API thực sự. Nó được bọc trong khối `try...except`, dùng để bắt các ngoại lệ có thể xảy ra. Nếu việc tìm kiếm thất bại, nó sẽ cập nhật trạng thái `step` thành `"search_failed"`, trạng thái này sẽ được node tiếp theo sử dụng để kích hoạt phương án dự phòng (fallback).

（3）Node trả lời

Node trả lời cuối cùng có thể lựa chọn chiến lược trả lời khác nhau dựa trên việc tìm kiếm ở bước trước có thành công hay không, có một mức độ linh hoạt nhất định.

```python
def generate_answer_node(state: SearchState) -> dict:
    """步骤3：基于搜索结果生成最终答案"""
    if state["step"] == "search_failed":
        # 如果搜索失败，执行回退策略，基于LLM自身知识回答
        fallback_prompt = f"搜索API暂时不可用，请基于您的知识回答用户的问题：\n用户问题：{state['user_query']}"
        response = llm.invoke([SystemMessage(content=fallback_prompt)])
    else:
        # 搜索成功，基于搜索结果生成答案
        answer_prompt = f"""基于以下搜索结果为用户提供完整、准确的答案：
用户问题：{state['user_query']}
搜索结果：\n{state['search_results']}
请综合搜索结果，提供准确、有用的回答..."""
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
    
    # 添加节点
    workflow.add_node("understand", understand_query_node)
    workflow.add_node("search", tavily_search_node)
    workflow.add_node("answer", generate_answer_node)
    
    # 设置线性流程
    workflow.add_edge(START, "understand")
    workflow.add_edge("understand", "search")
    workflow.add_edge("search", "answer")
    workflow.add_edge("answer", END)
    
    # 编译图
    memory = InMemorySaver()
    app = workflow.compile(checkpointer=memory)
    return app
```

（5）Trình bày kết quả chạy case 

Sau khi chạy script này, bạn có thể đặt một vài câu hỏi cần thông tin theo thời gian thực, ví dụ như case trong Chương 1 của chúng ta: `明天我要去北京，天气怎么样？有合适的景点吗`

Bạn sẽ thấy terminal hiển thị rõ ràng quá trình "suy nghĩ" của Agent:

```
🔍 智能搜索助手启动！
我会使用Tavily API为您搜索最新、最准确的信息
支持各种问题：新闻、技术、知识问答等
(输入 'quit' 退出)

🤔 您想了解什么: 明天我要去北京，天气怎么样？有合适的景点吗

============================================================
🧠 理解阶段: 我理解您的需求：理解：用户想了解明天北京的天气情况以及合适的景点推荐。  
搜索词：北京 明天 天气 景点推荐 Beijing weather tomorrow attractions
🔍 正在搜索: 北京 明天 天气 景点推荐 Beijing weather tomorrow attractions
🔍 搜索阶段: ✅ 搜索完成！找到了相关信息，正在为您整理答案...

💡 最终回答:
明天（2025年9月17日）北京的天气预报显示，预计将是多云，气温范围在17°C（62°F）到25°C（77°F）之间。这种温和的天气非常适合户外活动。

### 合适的景点推荐：
1. **长城**：作为中国最著名的历史遗址之一，长城是必游之地。你可以选择八达岭或慕田峪这些较为受欢迎的段落进行游览。

2. **故宫**：故宫是明清两代的皇宫，拥有丰富的历史和文化，适合对中国历史感兴趣的游客。

3. **天安门广场**：这是中国的象征之一，广场上有许多重要的建筑和纪念碑，适合拍照留念。

4. **颐和园**：一个非常美丽的皇家园林，适合漫步和欣赏自然风光，尤其是湖泊和古建筑。

5. **798艺术区**：如果你对现代艺术感兴趣，798艺术区是一个集艺术、文化和创意于一体的地方，适合探索和拍摄。

### 小贴士：
- 由于明天天气良好，建议提前规划出行路线，并准备一些水和小吃，以便在游览时保持充足的体力。
- 由于天气变化可能会影响游览体验，建议查看实时天气更新。

希望这些信息能帮助你安排一个愉快的北京之旅！如果你需要更多关于景点的信息或者旅行建议，欢迎随时询问。

============================================================

🤔 您想了解什么:
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
   - Trò chơi sử dụng Structured Output (như `DiscussionModelCN`, `WitchActionModelCN`) để ràng buộc hành vi của Agent. Hãy thiết kế một vai trò mới trong trò chơi là "Thợ săn" (猎人), và định nghĩa model Structured Output tương ứng cho nó, bao gồm định nghĩa trường và quy tắc kiểm chứng.
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
