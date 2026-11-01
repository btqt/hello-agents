# Chương 10 Giao thức giao tiếp của Agent

Trong các chương trước, chúng ta đã xây dựng các Agent đơn thể (monolithic) có đầy đủ chức năng, với khả năng suy luận (reasoning), gọi công cụ (tool calling) và ghi nhớ (memory). Tuy nhiên, khi cố gắng xây dựng các hệ thống AI phức tạp hơn, chúng ta tự nhiên sẽ đặt câu hỏi: <strong>Làm thế nào để Agent tương tác hiệu quả với thế giới bên ngoài? Làm thế nào để nhiều Agent có thể hợp tác với nhau?</strong>

Đây chính là vấn đề cốt lõi mà giao thức giao tiếp của Agent cần giải quyết. Chương này sẽ giới thiệu ba giao thức giao tiếp cho framework HelloAgents: <strong>MCP (Model Context Protocol)</strong> dùng cho giao tiếp chuẩn hóa giữa Agent và công cụ (tool), <strong>A2A (Agent-to-Agent Protocol)</strong> dùng cho hợp tác điểm-điểm (peer-to-peer) giữa các Agent, <strong>ANP (Agent Network Protocol)</strong> dùng để xây dựng mạng lưới Agent quy mô lớn. Ba giao thức này cùng nhau tạo thành lớp hạ tầng (infrastructure) cho giao tiếp của Agent.

Qua chương này, bạn sẽ nắm được triết lý thiết kế và kỹ năng thực hành của các giao thức giao tiếp Agent, hiểu được sự khác biệt trong thiết kế của ba giao thức chủ đạo, và học cách lựa chọn giao thức phù hợp để giải quyết các vấn đề thực tế.

## 10.1 Nền tảng về giao thức giao tiếp của Agent

### 10.1.1 Tại sao cần giao thức giao tiếp

Hãy nhìn lại ReAct Agent mà chúng ta đã xây dựng ở Chương 7, nó đã sở hữu khả năng suy luận và gọi công cụ mạnh mẽ. Hãy xem một kịch bản sử dụng điển hình:

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import CalculatorTool, SearchTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="AI助手", llm=llm)
agent.add_tool(CalculatorTool())
agent.add_tool(SearchTool())

# 智能体可以独立完成任务
response = agent.run("搜索最新的AI新闻，并计算相关公司的市值总和")
```

Agent này hoạt động khá tốt, nhưng nó gặp phải ba giới hạn căn bản. Đầu tiên là <strong>khó khăn trong việc tích hợp công cụ (tool)</strong>: mỗi khi cần truy cập một dịch vụ bên ngoài mới (như GitHub API, cơ sở dữ liệu, hệ thống file), chúng ta đều phải viết một class Tool chuyên biệt. Điều này không chỉ tốn công sức mà các tool do những nhà phát triển khác nhau viết ra còn không thể tương thích với nhau. Thứ hai là <strong>nút thắt cổ chai trong mở rộng năng lực</strong>: năng lực của Agent bị giới hạn trong tập hợp tool đã định nghĩa trước, không thể tự động khám phá (discover) và sử dụng các dịch vụ mới. Cuối cùng là <strong>thiếu khả năng hợp tác</strong>: khi nhiệm vụ phức tạp đến mức cần nhiều Agent chuyên biệt hợp tác (như nhà nghiên cứu + người viết + biên tập viên), chúng ta chỉ có thể điều phối công việc của chúng một cách thủ công.

Hãy xem một ví dụ cụ thể hơn để hiểu rõ những giới hạn này. Giả sử bạn muốn xây dựng một trợ lý nghiên cứu thông minh (research assistant), nó cần:

```python
# 传统方式：手动集成每个服务
class GitHubTool(BaseTool):
    """需要手写GitHub API适配器"""
    def run(self, repo_url):
        # 大量的API调用代码...
        pass

class DatabaseTool(BaseTool):
    """需要手写数据库适配器"""
    def run(self, query):
        # 数据库连接和查询代码...
        pass

class WeatherTool(BaseTool):
    """需要手写天气API适配器"""
    def run(self, location):
        # 天气API调用代码...
        pass

# 每个新服务都需要重复这个过程
agent.add_tool(GitHubTool())
agent.add_tool(DatabaseTool())
agent.add_tool(WeatherTool())
```

Cách làm này tồn tại những vấn đề rõ ràng: code bị lặp lại (mỗi tool đều phải xử lý HTTP request, xử lý lỗi, xác thực (authentication)...), khó bảo trì (khi API thay đổi phải sửa tất cả các tool liên quan), không thể tái sử dụng (tool của nhà phát triển khác không thể dùng trực tiếp), khả năng mở rộng kém (thêm dịch vụ mới cần rất nhiều công sức lập trình).

<strong>Giá trị cốt lõi của giao thức giao tiếp</strong> chính là giải quyết những vấn đề này. Nó cung cấp một bộ quy tắc giao diện (interface) chuẩn hóa, cho phép Agent truy cập nhiều dịch vụ bên ngoài khác nhau theo cùng một cách thống nhất, mà không cần viết adapter riêng cho từng dịch vụ. Điều này giống như giao thức TCP/IP của Internet, giúp các thiết bị khác nhau có thể giao tiếp với nhau mà không cần viết code giao tiếp riêng cho từng loại thiết bị.

Với giao thức giao tiếp, đoạn code trên có thể được đơn giản hóa thành:

```python
from hello_agents.tools import MCPTool

# 连接到MCP服务器，自动获得所有工具
mcp_tool = MCPTool()  # 内置服务器提供基础工具

# 或者连接到专业的MCP服务器
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])
database_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])

# 智能体自动获得所有能力，无需手写适配器
agent.add_tool(mcp_tool)
agent.add_tool(github_mcp)
agent.add_tool(database_mcp)
```

Sự thay đổi mà giao thức giao tiếp mang lại là mang tính nền tảng: <strong>giao diện chuẩn hóa (standardized interface)</strong> giúp các dịch vụ khác nhau cung cấp cách truy cập thống nhất, <strong>khả năng tương tác (interoperability)</strong> giúp các tool của nhiều nhà phát triển khác nhau có thể tích hợp liền mạch, <strong>khám phá động (dynamic discovery)</strong> cho phép Agent khám phá các dịch vụ và năng lực mới trong thời gian chạy (runtime), <strong>khả năng mở rộng (extensibility)</strong> giúp hệ thống dễ dàng thêm các module chức năng mới.

### 10.1.2 So sánh triết lý thiết kế của ba giao thức

Giao thức giao tiếp của Agent không phải là một giải pháp duy nhất, mà là một loạt các tiêu chuẩn được thiết kế cho các tình huống giao tiếp khác nhau. Trong chương này, chúng ta sẽ thực hành với ba giao thức chủ đạo hiện nay trong ngành là MCP, A2A và ANP, dưới đây là một bảng tổng quan so sánh.

<strong>（1）MCP: Cầu nối giữa Agent và công cụ (tool)</strong>

MCP (Model Context Protocol) được đội ngũ Anthropic đề xuất<sup>[1]</sup>, triết lý thiết kế cốt lõi của nó là <strong>chuẩn hóa cách giao tiếp giữa Agent và các tool/resource bên ngoài</strong>. Hãy tưởng tượng Agent của bạn cần truy cập hệ thống file, cơ sở dữ liệu, GitHub, Slack và nhiều dịch vụ khác. Cách làm truyền thống là viết adapter chuyên biệt cho từng dịch vụ, việc này không chỉ tốn nhiều công sức mà còn khó bảo trì. MCP định nghĩa một quy tắc giao thức (protocol specification) thống nhất, giúp mọi dịch vụ đều có thể được truy cập theo cùng một cách.

Triết lý thiết kế của MCP là "chia sẻ ngữ cảnh" (context sharing). Nó không chỉ đơn thuần là một giao thức RPC (Remote Procedure Call), mà quan trọng hơn là nó cho phép Agent và tool chia sẻ thông tin ngữ cảnh (context) phong phú với nhau. Như Hình 10.1 minh họa, khi Agent truy cập một repository mã nguồn, MCP Server không chỉ có thể cung cấp nội dung file mà còn có thể cung cấp cấu trúc code, quan hệ phụ thuộc (dependency), lịch sử commit và các thông tin ngữ cảnh khác, giúp Agent đưa ra quyết định thông minh hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-1.png" alt="" width="85%"/>
  <p>Hình 10.1 Triết lý thiết kế của MCP</p>
</div>

<strong>（2）A2A: Cuộc đối thoại giữa các Agent</strong>

Giao thức A2A (Agent-to-Agent Protocol) được đội ngũ Google đề xuất<sup>2</sup>, triết lý thiết kế cốt lõi của nó là <strong>thực hiện giao tiếp điểm-điểm (peer-to-peer) giữa các Agent</strong>. Khác với MCP tập trung vào giao tiếp giữa Agent và tool, A2A tập trung vào cách các Agent hợp tác với nhau. Thiết kế này cho phép các Agent đối thoại, đàm phán (negotiate) và hợp tác giống như một đội ngũ con người.

Triết lý thiết kế của A2A là "giao tiếp ngang hàng" (peer communication). Như Hình 10.2 minh họa, trong mạng A2A, mỗi Agent vừa là nhà cung cấp dịch vụ (service provider), vừa là người tiêu dùng dịch vụ (service consumer). Agent có thể chủ động khởi tạo request, cũng có thể phản hồi request từ Agent khác. Thiết kế ngang hàng này tránh được nút thắt cổ chai của bộ điều phối trung tâm (centralized coordinator), giúp mạng lưới Agent linh hoạt và dễ mở rộng hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-2.png" alt="" width="85%"/>
  <p>Hình 10.2 Triết lý thiết kế của A2A</p>
</div>

<strong>（3）ANP: Hạ tầng của mạng lưới Agent</strong>

ANP (Agent Network Protocol) là một khung giao thức (protocol framework) mang tính khái niệm<sup>3</sup>, hiện được duy trì bởi cộng đồng mã nguồn mở, chưa có hệ sinh thái hoàn thiện, triết lý thiết kế cốt lõi của nó là <strong>xây dựng hạ tầng cho mạng lưới Agent quy mô lớn</strong>. Nếu MCP giải quyết vấn đề "làm thế nào để truy cập tool", A2A giải quyết vấn đề "làm thế nào để đối thoại với Agent khác", thì ANP giải quyết vấn đề "làm thế nào để khám phá và kết nối Agent trong một mạng lưới quy mô lớn".

Triết lý thiết kế của ANP là "khám phá dịch vụ phi tập trung" (decentralized service discovery). Trong một mạng lưới chứa hàng trăm, hàng nghìn Agent, làm thế nào để một Agent có thể tìm được dịch vụ mà nó cần? Như Hình 10.3 minh họa, ANP cung cấp cơ chế đăng ký (registration), khám phá (discovery) và định tuyến (routing) dịch vụ, giúp Agent có thể khám phá động các dịch vụ khác trong mạng lưới mà không cần cấu hình sẵn tất cả các mối quan hệ kết nối.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-3.png" alt="" width="85%"/>
  <p>Hình 10.3 Triết lý thiết kế của ANP</p>
</div>

Cuối cùng, trong Bảng 10.1, hãy cùng xem một bảng so sánh để hiểu rõ hơn sự khác biệt giữa ba giao thức này:

<div align="center">
  <p>Bảng 10.1 So sánh ba giao thức</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-1.png" alt="" width="85%"/>
</div>

<strong>（4）Làm thế nào để chọn giao thức phù hợp?</strong>

Hiện tại các giao thức vẫn đang ở giai đoạn phát triển sớm, hệ sinh thái của MCP tương đối hoàn thiện hơn, tuy nhiên tính thời sự của các tool phụ thuộc vào người duy trì (maintainer), nên khuyến nghị chọn các tool MCP được bảo chứng bởi các công ty lớn.

Chìa khóa để chọn giao thức nằm ở việc hiểu rõ nhu cầu của bạn:

- Nếu Agent của bạn cần truy cập dịch vụ bên ngoài (file, cơ sở dữ liệu, API), hãy chọn <strong>MCP</strong>
- Nếu bạn cần nhiều Agent hợp tác với nhau để hoàn thành nhiệm vụ, hãy chọn <strong>A2A</strong>
- Nếu bạn muốn xây dựng một hệ sinh thái Agent quy mô lớn, hãy cân nhắc <strong>ANP</strong>

### 10.1.3 Thiết kế kiến trúc giao thức giao tiếp của HelloAgents

Sau khi hiểu được triết lý thiết kế của ba giao thức, hãy cùng xem cách triển khai và sử dụng chúng trong framework HelloAgents. Mục tiêu thiết kế của chúng tôi là: <strong>giúp người học có thể sử dụng các giao thức này theo cách đơn giản nhất, đồng thời vẫn giữ đủ sự linh hoạt để đối phó với các tình huống phức tạp</strong>.

Như Hình 10.4 minh họa, kiến trúc giao thức giao tiếp của HelloAgents sử dụng thiết kế ba lớp, từ tầng dưới lên tầng trên lần lượt là: lớp triển khai giao thức (protocol implementation layer), lớp bọc tool (tool wrapper layer) và lớp tích hợp Agent (agent integration layer).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-4.png" alt="" width="85%"/>
  <p>Hình 10.4 Thiết kế giao thức giao tiếp của HelloAgents</p>
</div>

<strong>（1）Lớp triển khai giao thức</strong>: lớp này chứa phần triển khai cụ thể của ba giao thức. MCP được triển khai dựa trên thư viện FastMCP, cung cấp chức năng client và server; A2A được triển khai dựa trên a2a-sdk chính thức của Google; ANP là phần triển khai nhẹ (lightweight) do chúng tôi tự phát triển, cung cấp chức năng khám phá dịch vụ (service discovery) và quản lý mạng lưới (network management), tất nhiên hiện tại cũng đã có [bản triển khai](https://github.com/agent-network-protocol/AgentConnect) chính thức, nhưng xét đến sự biến đổi sau này, ở đây chúng tôi chỉ mô phỏng khái niệm.

<strong>（2）Lớp bọc tool</strong>: lớp này đóng gói các phần triển khai giao thức thành giao diện Tool thống nhất. MCPTool, A2ATool và ANPTool đều kế thừa từ BaseTool, cung cấp phương thức `run()` nhất quán. Thiết kế này giúp Agent sử dụng các giao thức khác nhau theo cùng một cách.

<strong>（3）Lớp tích hợp Agent</strong>: lớp này là điểm tích hợp giữa Agent và giao thức. Tất cả các Agent (ReActAgent, SimpleAgent, v.v.) đều sử dụng protocol tool thông qua Tool System, không cần quan tâm đến chi tiết giao thức ở tầng dưới.

### 10.1.4 Mục tiêu học tập và trải nghiệm nhanh của chương

Hãy cùng xem trước nội dung học tập của Chương 10:

```
hello_agents/
├── protocols/                          # 通信协议模块
│   ├── mcp/                            # MCP协议实现（Model Context Protocol）
│   │   ├── client.py                   # MCP客户端（支持5种传输方式）
│   │   ├── server.py                   # MCP服务器（FastMCP封装）
│   │   └── utils.py                    # 工具函数（create_context/parse_context）
│   ├── a2a/                            # A2A协议实现（Agent-to-Agent Protocol）
│   │   └── implementation.py           # A2A服务器/客户端（基于a2a-sdk，可选依赖）
│   └── anp/                            # ANP协议实现（Agent Network Protocol）
│       └── implementation.py           # ANP服务发现/注册（概念性实现）
└── tools/builtin/                      # 内置工具模块
    └── protocol_tools.py               # 协议工具包装器（MCPTool/A2ATool/ANPTool）
```

Đối với nội dung của chương này, chủ yếu tập trung vào ứng dụng thực tiễn, mục tiêu học tập là có được năng lực áp dụng các giao thức vào dự án của chính bạn. Và hiện tại các giao thức vẫn đang ở giai đoạn phát triển sớm, nên không cần tốn quá nhiều công sức để "phát minh lại bánh xe" (reinvent the wheel). Trước khi bắt tay vào thực hành, hãy cùng chuẩn bị môi trường phát triển:

```bash
# 安装HelloAgents框架（第10章版本）
pip install "hello-agents[protocol]==0.2.2"

# 安装NodeJS, 可以参考Additional-Chapter中的文档
```

Hãy dùng đoạn code đơn giản nhất để trải nghiệm chức năng cơ bản của ba giao thức:

```python
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 1. MCP：访问工具
mcp_tool = MCPTool()
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(f"MCP计算结果: {result}")  # 输出: 30.0

# 2. ANP：服务发现
anp_tool = ANPTool()
anp_tool.run({
    "action": "register_service",
    "service_id": "calculator",
    "service_type": "math",
    "endpoint": "http://localhost:8080"
})
services = anp_tool.run({"action": "discover_services"})
print(f"发现的服务: {services}")

# 3. A2A：智能体通信
a2a_tool = A2ATool("http://localhost:5000")
print("A2A工具创建成功")
```

Ví dụ đơn giản này minh họa chức năng cốt lõi của ba giao thức. Trong các phần tiếp theo, chúng ta sẽ đi sâu học cách sử dụng chi tiết và best practice của từng giao thức.


## 10.2 Thực hành giao thức MCP

Bây giờ, hãy đi sâu học về MCP, nắm được cách giúp Agent truy cập công cụ (tool) và tài nguyên (resource) bên ngoài.

### 10.2.1 Giới thiệu khái niệm giao thức MCP

<strong>（1）MCP: "Cổng USB-C" của Agent</strong>

Hãy tưởng tượng Agent của bạn có thể cần làm nhiều việc cùng lúc, ví dụ:
- Đọc tài liệu trên hệ thống file cục bộ
- Truy vấn cơ sở dữ liệu PostgreSQL
- Tìm kiếm code trên GitHub
- Gửi tin nhắn Slack
- Truy cập Google Drive

Theo cách truyền thống, bạn cần viết code adapter cho từng dịch vụ, xử lý các API, phương thức xác thực (authentication), xử lý lỗi khác nhau. Điều này không chỉ tốn nhiều công sức mà còn khó bảo trì. Quan trọng hơn, cách triển khai function call của các nền tảng LLM khác nhau có sự khác biệt rất lớn, khi chuyển đổi model cần viết lại rất nhiều code.

Sự xuất hiện của MCP đã thay đổi tất cả điều đó. Nó giống như cách USB-C thống nhất phương thức kết nối của các thiết bị khác nhau, <strong>MCP thống nhất cách Agent tương tác với công cụ (tool) bên ngoài</strong>. Dù bạn dùng Claude, GPT hay model khác, chỉ cần chúng hỗ trợ giao thức MCP, chúng có thể truy cập liền mạch các tool và resource giống nhau.

<strong>（2）Kiến trúc MCP</strong>

Giao thức MCP sử dụng thiết kế kiến trúc ba lớp Host, Client, Servers, hãy cùng tìm hiểu cách các thành phần này phối hợp hoạt động qua kịch bản trong Hình 10.5.

Giả sử bạn đang dùng Claude Desktop để hỏi: "Trên desktop của tôi có những tài liệu nào?"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-5.png" alt="" width="85%"/>
  <p>Hình 10.5 Ví dụ minh họa MCP</p>
</div>

<strong>Trách nhiệm của kiến trúc ba lớp:</strong>

1. <strong>Host (lớp host)</strong>: Claude Desktop đóng vai trò Host, chịu trách nhiệm nhận câu hỏi của người dùng và tương tác với model Claude. Host là giao diện mà người dùng tương tác trực tiếp, nó quản lý toàn bộ luồng hội thoại.

2. <strong>Client (lớp client)</strong>: Khi model Claude quyết định cần truy cập hệ thống file, MCP Client được tích hợp sẵn trong Host sẽ được kích hoạt. Client chịu trách nhiệm thiết lập kết nối với MCP Server phù hợp, gửi request và nhận response.

3. <strong>Server (lớp server)</strong>: MCP Server của hệ thống file được gọi, thực hiện thao tác quét file thực tế, truy cập thư mục desktop, và trả về danh sách tài liệu tìm thấy.

<strong>Luồng tương tác hoàn chỉnh:</strong> Câu hỏi của người dùng → Claude Desktop (Host) → Model Claude phân tích → Cần thông tin file → MCP Client kết nối → MCP Server hệ thống file → Thực thi thao tác → Trả về kết quả → Claude tạo câu trả lời → Hiển thị trên Claude Desktop

Ưu điểm của thiết kế kiến trúc này nằm ở <strong>sự phân tách mối quan tâm (separation of concerns)</strong>: Host tập trung vào trải nghiệm người dùng, Client tập trung vào giao tiếp giao thức, Server tập trung vào triển khai chức năng cụ thể. Nhà phát triển chỉ cần tập trung phát triển MCP Server tương ứng, không cần quan tâm đến chi tiết triển khai của Host và Client.

<strong>（3）Năng lực cốt lõi của MCP</strong>

Như Bảng 10.2 minh họa, giao thức MCP cung cấp ba năng lực cốt lõi, tạo thành một khung truy cập tool hoàn chỉnh:

<div align="center">
  <p>Bảng 10.2 Năng lực cốt lõi của MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-2.png" alt="" width="85%"/>
</div>

Sự khác biệt giữa ba năng lực này là: <strong>Tools mang tính chủ động</strong> (thực thi thao tác), <strong>Resources mang tính bị động</strong> (cung cấp dữ liệu), <strong>Prompts mang tính chỉ dẫn</strong> (cung cấp mẫu/template).

<strong>（4）Luồng làm việc của MCP</strong>

Hãy cùng xem một ví dụ cụ thể để hiểu luồng làm việc hoàn chỉnh của MCP, như Hình 10.6 minh họa:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-6.png" alt="" width="85%"/>
  <p>Hình 10.6 Ví dụ minh họa MCP</p>
</div>

Một câu hỏi then chốt là: <strong>Claude (hoặc LLM khác) quyết định sử dụng tool nào bằng cách nào?</strong>

Khi người dùng đặt câu hỏi, luồng lựa chọn tool hoàn chỉnh như sau:

1. <strong>Giai đoạn khám phá tool</strong>: Sau khi MCP Client kết nối tới Server, trước tiên nó gọi `list_tools()` để lấy thông tin mô tả của tất cả tool khả dụng (bao gồm tên tool, mô tả chức năng, định nghĩa tham số)

2. <strong>Xây dựng ngữ cảnh (context)</strong>: Client chuyển danh sách tool thành định dạng mà LLM có thể hiểu được, thêm vào system prompt. Ví dụ:
   ```
   你可以使用以下工具：
   - read_file(path: str): 读取指定路径的文件内容
   - search_code(query: str, language: str): 在代码库中搜索
   ```

3. <strong>Suy luận của model</strong>: LLM phân tích câu hỏi của người dùng và các tool khả dụng, quyết định có cần gọi tool hay không và gọi tool nào. Quyết định này dựa trên mô tả của tool và ngữ cảnh hội thoại hiện tại

4. <strong>Thực thi tool</strong>: Nếu LLM quyết định dùng tool, Client thực thi tool đã chọn thông qua MCP Server, lấy kết quả

5. <strong>Tổng hợp kết quả</strong>: Kết quả thực thi tool được gửi lại cho LLM, LLM kết hợp với kết quả để tạo câu trả lời cuối cùng

Quá trình này <strong>hoàn toàn tự động hóa</strong>, LLM sẽ dựa vào chất lượng mô tả của tool để quyết định có sử dụng và sử dụng như thế nào. Vì vậy, viết mô tả tool rõ ràng, chính xác là vô cùng quan trọng.

<strong>（5）Sự khác biệt giữa MCP và Function Calling</strong>

Nhiều nhà phát triển sẽ hỏi: <strong>Tôi đã dùng Function Calling rồi, tại sao vẫn cần MCP?</strong> Hãy cùng tìm hiểu sự khác biệt của chúng qua Bảng 10.3.

<div align="center">
  <p>Bảng 10.3 So sánh Function Calling và MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-3.png" alt="" width="85%"/>
</div>

Ở đây chúng ta lấy ví dụ Agent cần truy cập repository GitHub và hệ thống file cục bộ để so sánh chi tiết hai cách triển khai cho cùng một nhiệm vụ

<strong>Cách 1: Sử dụng Function Calling</strong>

```python
# 步骤1：为每个LLM提供商定义函数
# OpenAI格式
openai_tools = [
    {
        "type": "function",
        "function": {
            "name": "search_github",
            "description": "搜索GitHub仓库",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "搜索关键词"}
                },
                "required": ["query"]
            }
        }
    }
]

# Claude格式
claude_tools = [
    {
        "name": "search_github",
        "description": "搜索GitHub仓库",
        "input_schema": {  # 注意：不是parameters
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "搜索关键词"}
            },
            "required": ["query"]
        }
    }
]

# 步骤2：自己实现工具函数
def search_github(query):
    import requests
    response = requests.get(
        "https://api.github.com/search/repositories",
        params={"q": query}
    )
    return response.json()

# 步骤3：处理不同模型的响应格式
# OpenAI的响应
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    result = search_github(**json.loads(tool_call.function.arguments))

# Claude的响应
if response.content[0].type == "tool_use":
    tool_use = response.content[0]
    result = search_github(**tool_use.input)
```

<strong>Cách 2: Sử dụng MCP</strong>

```python
from hello_agents.protocols import MCPClient

# 步骤1：连接到社区提供的MCP服务器（无需自己实现）
github_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-github"
])

fs_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-filesystem", "."
])

# 步骤2：统一的调用方式（与模型无关）
async with github_client:
    # 自动发现工具
    tools = await github_client.list_tools()

    # 调用工具（标准化接口）
    result = await github_client.call_tool(
        "search_repositories",
        {"query": "AI agents"}
    )

# 步骤3：任何支持MCP的模型都能使用
# OpenAI、Claude、Llama等都使用相同的MCP客户端
```

Trước hết cần làm rõ rằng, Function Calling và MCP không phải là mối quan hệ cạnh tranh, mà là bổ trợ lẫn nhau. Function Calling là một năng lực cốt lõi của mô hình ngôn ngữ lớn (LLM), nó thể hiện trí thông minh nội tại của model, giúp model hiểu được khi nào cần gọi function, và tạo ra tham số gọi hàm một cách chính xác. Ngược lại, MCP đóng vai trò là một giao thức hạ tầng (infrastructure protocol), nó giải quyết ở tầng kỹ thuật vấn đề làm thế nào để kết nối tool với model, thông qua một phương pháp chuẩn hóa để mô tả và gọi tool.

Chúng ta có thể dùng một phép ẩn dụ đơn giản để hiểu: Function Calling tương đương với việc bạn đã học được kỹ năng "làm thế nào để gọi điện thoại", bao gồm khi nào bấm số, cách giao tiếp với đối phương, khi nào cúp máy. Còn MCP chính là "tiêu chuẩn viễn thông điện thoại" thống nhất toàn cầu, đảm bảo rằng bất kỳ chiếc điện thoại nào cũng có thể gọi thông suốt tới một chiếc điện thoại khác.

Sau khi hiểu được mối quan hệ bổ trợ giữa chúng, hãy cùng xem cách sử dụng giao thức MCP trong HelloAgents.

### 10.2.2 Sử dụng MCP client

HelloAgents triển khai đầy đủ chức năng MCP client dựa trên FastMCP 2.0. Chúng tôi cung cấp cả hai API bất đồng bộ (asynchronous) và đồng bộ (synchronous), để phù hợp với các tình huống sử dụng khác nhau. Với hầu hết ứng dụng, khuyến nghị dùng API bất đồng bộ, nó xử lý tốt hơn các request đồng thời (concurrent) và các thao tác chạy dài. Dưới đây chúng tôi sẽ trình bày một bản demo thao tác được chia nhỏ từng bước.

<strong>（1）Kết nối tới MCP server</strong>

MCP client hỗ trợ nhiều phương thức kết nối, phổ biến nhất là chế độ Stdio (giao tiếp với tiến trình cục bộ qua standard input/output):

```python
import asyncio
from hello_agents.protocols import MCPClient

async def connect_to_server():
    # 方式1：连接到社区提供的文件系统服务器
    # npx会自动下载并运行@modelcontextprotocol/server-filesystem包
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."  # 指定根目录
    ])

    # 使用async with确保连接正确关闭
    async with client:
        # 在这里使用client
        tools = await client.list_tools()
        print(f"可用工具: {[t['name'] for t in tools]}")

    # 方式2：连接到自定义的Python MCP服务器
    client = MCPClient(["python", "my_mcp_server.py"])
    async with client:
        # 使用client...
        pass

# 运行异步函数
asyncio.run(connect_to_server())
```

<strong>（2）Khám phá tool khả dụng</strong>

Sau khi kết nối thành công, bước đầu tiên thường là truy vấn xem server cung cấp những tool nào:

```python
async def discover_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # 获取所有可用工具
        tools = await client.list_tools()

        print(f"服务器提供了 {len(tools)} 个工具：")
        for tool in tools:
            print(f"\n工具名称: {tool['name']}")
            print(f"描述: {tool.get('description', '无描述')}")

            # 打印参数信息
            if 'inputSchema' in tool:
                schema = tool['inputSchema']
                if 'properties' in schema:
                    print("参数:")
                    for param_name, param_info in schema['properties'].items():
                        param_type = param_info.get('type', 'any')
                        param_desc = param_info.get('description', '')
                        print(f"  - {param_name} ({param_type}): {param_desc}")

asyncio.run(discover_tools())

# 输出示例：
# 服务器提供了 5 个工具：
#
# 工具名称: read_file
# 描述: 读取文件内容
# 参数:
#   - path (string): 文件路径
#
# 工具名称: write_file
# 描述: 写入文件内容
# 参数:
#   - path (string): 文件路径
#   - content (string): 文件内容
```

<strong>（3）Gọi tool</strong>

Khi gọi tool, chỉ cần cung cấp tên tool và tham số phù hợp với JSON Schema:

```python
async def use_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # 读取文件
        result = await client.call_tool("read_file", {"path": "my_README.md"})
        print(f"文件内容：\n{result}")

        # 列出目录
        result = await client.call_tool("list_directory", {"path": "."})
        print(f"当前目录文件：{result}")

        # 写入文件
        result = await client.call_tool("write_file", {
            "path": "output.txt",
            "content": "Hello from MCP!"
        })
        print(f"写入结果：{result}")

asyncio.run(use_tools())
```

Ở đây cung cấp một cách an toàn hơn để gọi MCP service, có thể dùng để tham khảo:

```python
async def safe_tool_call():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        try:
            # 尝试读取可能不存在的文件
            result = await client.call_tool("read_file", {"path": "nonexistent.txt"})
            print(result)
        except Exception as e:
            print(f"工具调用失败: {e}")
            # 可以选择重试、使用默认值或向用户报告错误

asyncio.run(safe_tool_call())
```

<strong>（4）Truy cập resource</strong>

Ngoài tool, MCP server còn có thể cung cấp resource (Resources):

```python
# 列出可用资源
resources = client.list_resources()
print(f"可用资源：{[r['uri'] for r in resources]}")

# 读取资源
resource_content = client.read_resource("file:///path/to/resource")
print(f"资源内容：{resource_content}")
```

<strong>（5）Sử dụng prompt template</strong>

MCP server có thể cung cấp các mẫu prompt được định nghĩa sẵn (Prompts):

```python
# 列出可用提示
prompts = client.list_prompts()
print(f"可用提示：{[p['name'] for p in prompts]}")

# 获取提示内容
prompt = client.get_prompt("code_review", {"language": "python"})
print(f"提示内容：{prompt}")
```

<strong>（6）Ví dụ hoàn chỉnh: Sử dụng dịch vụ GitHub MCP</strong>

Hãy cùng xem một ví dụ hoàn chỉnh về cách sử dụng dịch vụ GitHub MCP do cộng đồng cung cấp, chúng ta sẽ dùng MCP Tools đã được đóng gói sẵn:

```python
"""
GitHub MCP 服务示例

注意：需要设置环境变量
    Windows: $env:GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
    Linux/macOS: export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
"""

from hello_agents.tools import MCPTool

# 创建 GitHub MCP 工具
github_tool = MCPTool(
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)

# 1. 列出可用工具
print("📋 可用工具：")
result = github_tool.run({"action": "list_tools"})
print(result)

# 2. 搜索仓库
print("\n🔍 搜索仓库：")
result = github_tool.run({
    "action": "call_tool",
    "tool_name": "search_repositories",
    "arguments": {
        "query": "AI agents language:python",
        "page": 1,
        "perPage": 3
    }
})
print(result)

```

### 10.2.3 Chi tiết các phương thức truyền tải (transport) của MCP

Một đặc tính quan trọng của giao thức MCP là <strong>tính độc lập với tầng truyền tải</strong> (Transport Agnostic). Điều này có nghĩa là bản thân giao thức MCP không phụ thuộc vào một phương thức truyền tải cụ thể nào, nó có thể chạy trên nhiều kênh giao tiếp khác nhau. HelloAgents dựa trên FastMCP 2.0, cung cấp đầy đủ các phương thức truyền tải, giúp bạn chọn chế độ truyền tải phù hợp nhất với tình huống thực tế.

<strong>（1）Tổng quan các phương thức truyền tải</strong>

`MCPClient` của HelloAgents hỗ trợ năm phương thức truyền tải, mỗi phương thức có tình huống sử dụng khác nhau, như Bảng 10.4 minh họa:

<div align="center">
  <p>Bảng 10.4 So sánh các phương thức truyền tải của MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-4.png" alt="" width="85%"/>
</div>

<strong>（2）Ví dụ sử dụng các phương thức truyền tải</strong>

```python
from hello_agents.tools import MCPTool

# 1. Memory Transport - 内存传输（用于测试）
# 不指定任何参数，使用内置演示服务器
mcp_tool = MCPTool()

# 2. Stdio Transport - 标准输入输出传输（本地开发）
# 使用命令列表启动本地服务器
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py"])

# 3. Stdio Transport with Args - 带参数的命令传输
# 可以传递额外参数
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py", "--debug"])

# 4. Stdio Transport - 社区服务器（npx方式）
# 使用npx启动社区MCP服务器
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 5. HTTP/SSE/StreamableHTTP Transport
# 注意：MCPTool主要用于Stdio和Memory传输
# 对于HTTP/SSE等远程传输，建议直接使用MCPClient
```

<strong>（3）Memory Transport - Truyền tải trong bộ nhớ</strong>

Tình huống áp dụng: unit test, phát triển prototype nhanh

```python
from hello_agents.tools import MCPTool

# 使用内置演示服务器（Memory传输）
mcp_tool = MCPTool()

# 列出可用工具
result = mcp_tool.run({"action": "list_tools"})
print(result)

# 调用工具
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(result)
```

<strong>（4）Stdio Transport - Truyền tải qua standard input/output</strong>

Tình huống áp dụng: phát triển cục bộ, debug, server bằng script Python

```python
from hello_agents.tools import MCPTool

# 方式1：使用自定义Python服务器
mcp_tool = MCPTool(server_command=["python", "my_mcp_server.py"])

# 方式2：使用社区服务器（文件系统）
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 列出工具
result = mcp_tool.run({"action": "list_tools"})
print(result)

# 调用工具
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "read_file",
    "arguments": {"path": "README.md"}
})
print(result)
```

<strong>（5）HTTP Transport - Truyền tải qua HTTP</strong>

Tình huống áp dụng: môi trường production, dịch vụ từ xa (remote), kiến trúc microservices

```python
# 注意：MCPTool 主要用于 Stdio 和 Memory 传输
# 对于 HTTP/SSE 等远程传输，建议使用底层的 MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_http_transport():
    # 连接到远程 HTTP MCP 服务器
    client = MCPClient("http://api.example.com/mcp")

    async with client:
        # 获取服务器信息
        tools = await client.list_tools()
        print(f"远程服务器工具: {len(tools)} 个")

        # 调用远程工具
        result = await client.call_tool("process_data", {
            "data": "Hello, World!",
            "operation": "uppercase"
        })
        print(f"远程处理结果: {result}")

# 注意：需要实际的 HTTP MCP 服务器
# asyncio.run(test_http_transport())
```

<strong>（6）SSE Transport - Truyền tải Server-Sent Events</strong>

Tình huống áp dụng: giao tiếp thời gian thực (real-time), xử lý streaming, kết nối dài hạn (long connection)

```python
# 注意：MCPTool 主要用于 Stdio 和 Memory 传输
# 对于 SSE 传输，建议使用底层的 MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_sse_transport():
    # 连接到 SSE MCP 服务器
    client = MCPClient(
        "http://localhost:8080/sse",
        transport_type="sse"
    )

    async with client:
        # SSE 特别适合流式处理
        result = await client.call_tool("stream_process", {
            "input": "大量数据处理请求",
            "stream": True
        })
        print(f"流式处理结果: {result}")

# 注意：需要支持 SSE 的 MCP 服务器
# asyncio.run(test_sse_transport())
```

<strong>（7）StreamableHTTP Transport - Truyền tải HTTP dạng streaming</strong>

Tình huống áp dụng: các tình huống HTTP cần giao tiếp streaming hai chiều

```python
# 注意：MCPTool 主要用于 Stdio 和 Memory 传输
# 对于 StreamableHTTP 传输，建议使用底层的 MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_streamable_http_transport():
    # 连接到 StreamableHTTP MCP 服务器
    client = MCPClient(
        "http://localhost:8080/mcp",
        transport_type="streamable_http"
    )

    async with client:
        # 支持双向流式通信
        tools = await client.list_tools()
        print(f"StreamableHTTP 服务器工具: {len(tools)} 个")

# 注意：需要支持 StreamableHTTP 的 MCP 服务器
# asyncio.run(test_streamable_http_transport())
```

### 10.2.4 Sử dụng MCP tool trong Agent

Ở phần trước chúng ta đã học cách sử dụng MCP client một cách trực tiếp. Nhưng trong ứng dụng thực tế, chúng ta mong muốn Agent <strong>tự động</strong> gọi MCP tool, thay vì viết code gọi thủ công. HelloAgents cung cấp bộ bọc `MCPTool`, giúp tích hợp MCP server một cách liền mạch vào chuỗi tool của Agent.

<strong>（1）Cơ chế tự động mở rộng (auto-expand) của MCP tool</strong>

`MCPTool` của HelloAgents có một đặc tính: <strong>tự động mở rộng</strong> (auto-expand). Khi bạn thêm một MCP tool vào Agent, nó sẽ tự động mở rộng tất cả các tool do MCP server cung cấp thành các tool độc lập, giúp Agent có thể gọi chúng giống như gọi tool thông thường.

<strong>Cách 1: Sử dụng server demo tích hợp sẵn</strong>

Trước đây chúng ta đã triển khai hàm tool cho máy tính (calculator), ở đây chúng ta sẽ chuyển nó thành dịch vụ MCP. Đây là cách sử dụng đơn giản nhất.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="助手", llm=HelloAgentsLLM())

# 无需任何配置，自动使用内置演示服务器
mcp_tool = MCPTool(name="calculator")
agent.add_tool(mcp_tool)
# ✅ MCP工具 'calculator' 已展开为 6 个独立工具

# 智能体可以直接使用展开后的工具
response = agent.run("计算 25 乘以 16")
print(response)  # 输出：25 乘以 16 的结果是 400
```

<strong>Các tool sau khi tự động mở rộng</strong>:

- `calculator_add` - máy tính phép cộng
- `calculator_subtract` - máy tính phép trừ
- `calculator_multiply` - máy tính phép nhân
- `calculator_divide` - máy tính phép chia
- `calculator_greet` - lời chào thân thiện
- `calculator_get_system_info` - lấy thông tin hệ thống

Khi Agent gọi, chỉ cần cung cấp tham số, ví dụ: `[TOOL_CALL:calculator_multiply:a=25,b=16]`, hệ thống sẽ tự động xử lý chuyển đổi kiểu dữ liệu (type conversion) và gọi MCP.

<strong>Cách 2: Kết nối tới MCP server bên ngoài</strong>

Trong dự án thực tế, bạn cần kết nối tới các MCP server có chức năng mạnh mẽ hơn. Các server này có thể là:
- <strong>Server chính thức do cộng đồng cung cấp</strong> (như hệ thống file, GitHub, cơ sở dữ liệu, v.v.)
- <strong>Server tùy chỉnh do chính bạn viết</strong> (đóng gói business logic)

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="文件助手", llm=HelloAgentsLLM())

# 示例1：连接到社区提供的文件系统服务器
fs_tool = MCPTool(
    name="filesystem",  # 指定唯一名称
    description="访问本地文件系统",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
agent.add_tool(fs_tool)

# 示例2：连接到自定义的 Python MCP 服务器
# 关于如何编写自定义MCP服务器，请参考10.5章节
custom_tool = MCPTool(
    name="custom_server",  # 使用不同的名称
    description="自定义业务逻辑服务器",
    server_command=["python", "my_mcp_server.py"]
)
agent.add_tool(custom_tool)

# Agent现在可以自动使用这些工具！
response = agent.run("请读取my_README.md文件，并总结其中的主要内容")
print(response)
```

Khi sử dụng nhiều MCP server, bạn nhất định phải chỉ định `name` khác nhau cho mỗi MCPTool, `name` này sẽ được thêm làm tiền tố (prefix) vào tên tool được mở rộng, để tránh xung đột (conflict). Ví dụ: `name="fs"` sẽ mở rộng thành `fs_read_file`, `fs_write_file`, v.v. Nếu bạn cần viết MCP server của riêng mình để đóng gói business logic cụ thể, vui lòng tham khảo nội dung Mục 10.5.

<strong>（2）Nguyên lý hoạt động của cơ chế tự động mở rộng MCP tool</strong>

Hiểu được cơ chế tự động mở rộng sẽ giúp bạn sử dụng MCP tool tốt hơn. Hãy cùng tìm hiểu sâu cách nó hoạt động:

```python
# 用户代码
fs_tool = MCPTool(name="fs", server_command=[...])
agent.add_tool(fs_tool)

# 内部发生的事情：
# 1. MCPTool连接到服务器，发现14个工具
# 2. 为每个工具创建包装器：
#    - fs_read_text_file (参数: path, tail, head)
#    - fs_write_file (参数: path, content)
#    - ...
# 3. 注册到Agent的工具注册表

# Agent调用
response = agent.run("读取README.md")

# Agent内部：
# 1. 识别需要调用 fs_read_text_file
# 2. 生成参数：path=README.md
# 3. 包装器转换为MCP格式：
#    {"action": "call_tool", "tool_name": "read_text_file", "arguments": {"path": "README.md"}}
# 4. 调用MCP服务器
# 5. 返回文件内容
```

Hệ thống sẽ tự động chuyển đổi kiểu dữ liệu dựa trên định nghĩa tham số của tool:

```python
# Agent调用计算器
agent.run("计算 25 乘以 16")

# Agent生成：a=25,b=16 (字符串)
# 系统自动转换为：{"a": 25.0, "b": 16.0} (数字)
# MCP服务器接收到正确的数字类型
```

<strong>（3）Case thực chiến: Trợ lý tài liệu thông minh</strong>

Hãy cùng xây dựng một trợ lý tài liệu thông minh hoàn chỉnh, ở đây chúng ta sẽ dùng một ví dụ điều phối đa Agent (multi-agent orchestration) đơn giản để minh họa:

```python
"""
多Agent协作的智能文档助手

使用两个SimpleAgent分工协作：
- Agent1：GitHub搜索专家
- Agent2：文档生成专家
"""
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool
from dotenv import load_dotenv

# 加载.env文件中的环境变量
load_dotenv(dotenv_path="../HelloAgents/.env")

print("="*70)
print("多Agent协作的智能文档助手")
print("="*70)

# ============================================================
# Agent 1: GitHub搜索专家
# ============================================================
print("\n【步骤1】创建GitHub搜索专家...")

github_searcher = SimpleAgent(
    name="GitHub搜索专家",
    llm=HelloAgentsLLM(),
    system_prompt="""你是一个GitHub搜索专家。
你的任务是搜索GitHub仓库并返回结果。
请返回清晰、结构化的搜索结果，包括：
- 仓库名称
- 简短描述

保持简洁，不要添加额外的解释。"""
)

# 添加GitHub工具
github_tool = MCPTool(
    name="gh",
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)
github_searcher.add_tool(github_tool)

# ============================================================
# Agent 2: 文档生成专家
# ============================================================
print("\n【步骤2】创建文档生成专家...")

document_writer = SimpleAgent(
    name="文档生成专家",
    llm=HelloAgentsLLM(),
    system_prompt="""你是一个文档生成专家。
你的任务是根据提供的信息生成结构化的Markdown报告。

报告应该包括：
- 标题
- 简介
- 主要内容（分点列出，包括项目名称、描述等）
- 总结

请直接输出完整的Markdown格式报告内容，不要使用工具保存。"""
)

# 添加文件系统工具
fs_tool = MCPTool(
    name="fs",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
document_writer.add_tool(fs_tool)

# ============================================================
# 执行任务
# ============================================================
print("\n" + "="*70)
print("开始执行任务...")
print("="*70)

try:
    # 步骤1：GitHub搜索
    print("\n【步骤3】Agent1 搜索GitHub...")
    search_task = "搜索关于'AI agent'的GitHub仓库，返回前5个最相关的结果"
    
    search_results = github_searcher.run(search_task)
    
    print("\n搜索结果:")
    print("-" * 70)
    print(search_results)
    print("-" * 70)
    
    # 步骤2：生成报告
    print("\n【步骤4】Agent2 生成报告...")
    report_task = f"""
根据以下GitHub搜索结果，生成一份Markdown格式的研究报告：

{search_results}

报告要求：
1. 标题：# AI Agent框架研究报告
2. 简介：说明这是关于AI Agent的GitHub项目调研
3. 主要发现：列出找到的项目及其特点（包括名称、描述等）
4. 总结：总结这些项目的共同特点

请直接输出完整的Markdown格式报告。
"""

    report_content = document_writer.run(report_task)

    print("\n报告内容:")
    print("=" * 70)
    print(report_content)
    print("=" * 70)

    # 步骤3：保存报告
    print("\n【步骤5】保存报告到文件...")
    import os
    try:
        with open("report.md", "w", encoding="utf-8") as f:
            f.write(report_content)
        print("✅ 报告已保存到 report.md")

        # 验证文件
        file_size = os.path.getsize("report.md")
        print(f"✅ 文件大小: {file_size} 字节")
    except Exception as e:
        print(f"❌ 保存失败: {e}")
    
    print("\n" + "="*70)
    print("任务完成！")
    print("="*70)
    
except Exception as e:
    print(f"\n❌ 错误: {e}")
    import traceback
    traceback.print_exc()

```

Trong quá trình này, `github_searcher` sẽ gọi `gh_search_repositories` để tìm kiếm dự án GitHub. Kết quả thu được sẽ được trả về cho `document_writer` làm input, tiếp tục hướng dẫn việc sinh báo cáo, cuối cùng lưu báo cáo vào `report.md`.

### 10.2.5 Hệ sinh thái cộng đồng của MCP

Một ưu điểm lớn của giao thức MCP là <strong>hệ sinh thái cộng đồng phong phú</strong>. Anthropic và các nhà phát triển cộng đồng đã tạo ra một lượng lớn MCP server có sẵn, bao phủ hệ thống file, cơ sở dữ liệu, dịch vụ API và nhiều tình huống khác. Điều này có nghĩa là bạn không cần viết adapter tool từ đầu, có thể trực tiếp sử dụng các server đã được kiểm chứng này.

Dưới đây là ba kho tài nguyên của cộng đồng MCP:

1. <strong>Awesome MCP Servers</strong> (https://github.com/punkpeye/awesome-mcp-servers)
   - Danh sách MCP server tuyển chọn do cộng đồng duy trì
   - Bao gồm nhiều server bên thứ ba
   - Được phân loại theo chức năng, dễ tra cứu

2. <strong>MCP Servers Website</strong> (https://mcpservers.org/)
   - Website danh mục MCP server chính thức
   - Cung cấp chức năng tìm kiếm và lọc
   - Bao gồm hướng dẫn sử dụng và ví dụ

3. <strong>Official MCP Servers</strong> (https://github.com/modelcontextprotocol/servers)
   - Server do Anthropic chính thức duy trì
   - Chất lượng cao nhất, tài liệu hoàn thiện nhất
   - Bao gồm triển khai của các dịch vụ thông dụng

Bảng 10.5 và 10.6 trình bày các MCP server chính thức thông dụng và MCP server phổ biến của cộng đồng:

<div align="center">
  <p>Bảng 10.5 Các MCP server chính thức thông dụng</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-5.png" alt="" width="85%"/>
</div>

<div align="center">
  <p>Bảng 10.6 Các MCP server phổ biến của cộng đồng</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-6.png" alt="" width="85%"/>
</div>

Dưới đây là một số case thú vị TODO để bạn tham khảo:

1. <strong>Tự động hóa kiểm thử web (Playwright)</strong>
   
   ```python
   # Agent可以自动：
   # - 打开浏览器访问网站
   # - 填写表单并提交
   # - 截图验证结果
   # - 生成测试报告
   playwright_tool = MCPTool(
       name="playwright",
       server_command=["npx", "-y", "@playwright/mcp"]
   )
   ```
   
2. <strong>Trợ lý ghi chú thông minh (Obsidian + Perplexity)</strong>
   ```python
   # Agent可以：
   # - 搜索最新技术资讯（Perplexity）
   # - 整理成结构化笔记
   # - 保存到Obsidian知识库
   # - 自动建立笔记间的链接
   ```

3. <strong>Tự động hóa quản lý dự án (Jira + GitHub)</strong>
   ```python
   # Agent可以：
   # - 从GitHub Issue创建Jira任务
   # - 同步代码提交到Jira
   # - 自动更新Sprint进度
   # - 生成项目报告
   ```

5. <strong>Luồng tạo nội dung (YouTube + Notion + Spotify)</strong>
   
   ```python
   # Agent可以：
   # - 获取YouTube视频字幕
   # - 生成内容摘要
   # - 保存到Notion数据库
   # - 播放背景音乐（Spotify）
   ```

Qua nội dung phần này, hy vọng bạn có thể khám phá thêm nhiều case triển khai MCP khác, cũng rất hoan nghênh bạn đóng góp cho Helloagents! Tiếp theo, hãy cùng học về giao thức A2A.

## 10.3 Thực hành giao thức A2A

A2A (Agent-to-Agent) là một giao thức hỗ trợ giao tiếp và hợp tác trực tiếp giữa các Agent.

### 10.3.1 Động lực thiết kế giao thức

Giao thức MCP giải quyết vấn đề tương tác giữa Agent và tool, còn giao thức A2A giải quyết vấn đề hợp tác giữa các Agent. Trong một nhiệm vụ cần nhiều Agent (như nhà nghiên cứu, người viết, biên tập viên) hợp tác, chúng cần giao tiếp, ủy thác (delegate) nhiệm vụ, đàm phán năng lực và đồng bộ trạng thái (state).

Phương án bộ điều phối trung tâm (central coordinator) truyền thống (cấu trúc hình sao - star topology) tồn tại ba vấn đề chính:

- <strong>Điểm lỗi đơn (single point of failure)</strong>: Khi bộ điều phối gặp sự cố sẽ khiến toàn bộ hệ thống tê liệt.
- <strong>Nút thắt cổ chai hiệu năng (performance bottleneck)</strong>: Mọi giao tiếp đều phải đi qua node trung tâm, giới hạn khả năng xử lý đồng thời (concurrency).
- <strong>Khó mở rộng</strong>: Thêm hoặc sửa đổi Agent cần phải thay đổi logic trung tâm.

Giao thức A2A sử dụng kiến trúc điểm-điểm (P2P) (cấu trúc hình lưới - mesh topology), cho phép các Agent giao tiếp trực tiếp, từ gốc giải quyết các vấn đề nêu trên. Cốt lõi của nó là hai khái niệm trừu tượng <strong>Task (nhiệm vụ)</strong> và <strong>Artifact (sản phẩm đầu ra)</strong>, đây cũng là điểm khác biệt lớn nhất giữa nó và MCP, như Bảng 10.7 minh họa.

<div align="center">
  <p>Bảng 10.7 Các khái niệm cốt lõi của A2A</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-7.png" alt="" width="85%"/>
</div>

Để quản lý được quá trình hợp tác, A2A định nghĩa một vòng đời (lifecycle) chuẩn hóa cho Task, bao gồm các trạng thái tạo (created), đàm phán (negotiating), ủy thác (delegated), đang thực thi (executing), hoàn thành (completed), thất bại (failed), có thể thấy ở Hình 10.7.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-7.png" alt="" width="85%"/>
  <p>Hình 10.7 Vòng đời của Task trong A2A</p>
</div>


Cơ chế này giúp Agent có thể đàm phán nhiệm vụ, theo dõi tiến độ và xử lý ngoại lệ.

Vòng đời của request A2A là một chuỗi tuần tự, mô tả chi tiết bốn bước chính mà request tuân theo: khám phá Agent (agent discovery), xác thực danh tính (authentication), gửi API message và gửi API message dạng stream. Hình 10.8 dưới đây tham khảo từ sơ đồ luồng của trang chủ chính thức, dùng để minh họa luồng thao tác, mô tả tương tác giữa client, A2A server và server xác thực.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-8.png" alt="" width="85%"/>
  <p>Hình 10.8 Vòng đời của request A2A</p>
</div>

### 10.3.2 Thực hành sử dụng giao thức A2A

Phần lớn các bản triển khai A2A hiện có đều là `Sample Code`, và dù có triển khai bằng Python thì cũng khá rườm rà, vì vậy ở đây chúng ta chỉ sử dụng cách mô phỏng triết lý giao thức, thông qua A2A-SDK để kế thừa một phần chức năng đã triển khai.

<strong>（2）Tạo một A2A Agent đơn giản</strong>

Hãy cùng tạo một A2A Agent, vẫn dùng ví dụ máy tính (calculator) để minh họa:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_calculator_agent():
    """创建一个计算器智能体"""
    if not A2A_AVAILABLE:
        print("❌ A2A SDK 未安装，请运行: pip install a2a-sdk")
        return None

    print("🧮 创建计算器智能体")

    # 创建 A2A 服务器
    calculator = A2AServer(
        name="calculator-agent",
        description="专业的数学计算智能体",
        version="1.0.0",
        capabilities={
            "math": ["addition", "subtraction", "multiplication", "division"],
            "advanced": ["power", "sqrt", "factorial"]
        }
    )

    # 添加基础计算技能
    @calculator.skill("add")
    def add_numbers(query: str) -> str:
        """加法计算"""
        try:
            # 简单解析 "计算 5 + 3" 格式
            parts = query.replace("计算", "").replace("加", "+").replace("加上", "+")
            if "+" in parts:
                numbers = [float(x.strip()) for x in parts.split("+")]
                result = sum(numbers)
                return f"计算结果: {' + '.join(map(str, numbers))} = {result}"
            else:
                return "请使用格式: 计算 5 + 3"
        except Exception as e:
            return f"计算错误: {e}"

    @calculator.skill("multiply")
    def multiply_numbers(query: str) -> str:
        """乘法计算"""
        try:
            parts = query.replace("计算", "").replace("乘以", "*").replace("×", "*")
            if "*" in parts:
                numbers = [float(x.strip()) for x in parts.split("*")]
                result = 1
                for num in numbers:
                    result *= num
                return f"计算结果: {' × '.join(map(str, numbers))} = {result}"
            else:
                return "请使用格式: 计算 5 * 3"
        except Exception as e:
            return f"计算错误: {e}"

    @calculator.skill("info")
    def get_info(query: str) -> str:
        """获取智能体信息"""
        return f"我是 {calculator.name}，可以进行基础数学计算。支持的技能: {list(calculator.skills.keys())}"

    print(f"✅ 计算器智能体创建成功，支持技能: {list(calculator.skills.keys())}")
    return calculator

# 创建智能体
calc_agent = create_calculator_agent()
if calc_agent:
    # 测试技能
    print("\n🧪 测试智能体技能:")
    test_queries = [
        "获取信息",
        "计算 10 + 5",
        "计算 6 * 7"
    ]

    for query in test_queries:
        if "信息" in query:
            result = calc_agent.skills["info"](query)
        elif "+" in query:
            result = calc_agent.skills["add"](query)
        elif "*" in query or "×" in query:
            result = calc_agent.skills["multiply"](query)
        else:
            result = "未知查询类型"

        print(f"  📝 查询: {query}")
        print(f"  🤖 回复: {result}")
        print()
```

<strong>（2）Tùy chỉnh A2A Agent</strong>

Bạn cũng có thể tạo A2A Agent của riêng mình, ở đây chỉ trình bày một ví dụ minh họa đơn giản:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_custom_agent():
    """创建自定义智能体"""
    if not A2A_AVAILABLE:
        print("请先安装 A2A SDK: pip install a2a-sdk")
        return None

    # 创建智能体
    agent = A2AServer(
        name="my-custom-agent",
        description="我的自定义智能体",
        capabilities={"custom": ["skill1", "skill2"]}
    )

    # 添加技能
    @agent.skill("greet")
    def greet_user(name: str) -> str:
        """问候用户"""
        return f"你好，{name}！我是自定义智能体。"

    @agent.skill("calculate")
    def simple_calculate(expression: str) -> str:
        """简单计算"""
        try:
            # 安全的计算（仅支持基本运算）
            allowed_chars = set('0123456789+-*/(). ')
            if all(c in allowed_chars for c in expression):
                result = eval(expression)
                return f"计算结果: {expression} = {result}"
            else:
                return "错误: 只支持基本数学运算"
        except Exception as e:
            return f"计算错误: {e}"

    return agent

# 创建并测试自定义智能体
custom_agent = create_custom_agent()
if custom_agent:
    # 测试技能
    print("测试问候技能:")
    result1 = custom_agent.skills["greet"]("张三")
    print(result1)

    print("\n测试计算技能:")
    result2 = custom_agent.skills["calculate"]("10 + 5 * 2")
    print(result2)
```

### 10.3.3 Sử dụng A2A tool của HelloAgents

HelloAgents cung cấp giao diện A2A tool thống nhất.

<strong>（1）Tạo A2A Agent server</strong>

Trước tiên, hãy cùng tạo một Agent server:

```python
from hello_agents.protocols import A2AServer
import threading
import time

# 创建研究员Agent服务
researcher = A2AServer(
    name="researcher",
    description="负责搜索和分析资料的Agent",
    version="1.0.0"
)

# 定义技能
@researcher.skill("research")
def handle_research(text: str) -> str:
    """处理研究请求"""
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    
    # 实际的研究逻辑（这里简化）
    result = {
        "topic": topic,
        "findings": f"关于{topic}的研究结果...",
        "sources": ["来源1", "来源2", "来源3"]
    }
    return str(result)

# 在后台启动服务
def start_server():
    researcher.run(host="localhost", port=5000)

if __name__ == "__main__":
    server_thread = threading.Thread(target=start_server, daemon=True)
    server_thread.start()
    
    print("✅ 研究员Agent服务已启动在 http://localhost:5000")
    
    # 保持程序运行
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\n服务已停止")
```

<strong>（2）Tạo A2A Agent client</strong>

Bây giờ, hãy cùng tạo một client để giao tiếp với server:

```python
from hello_agents.protocols import A2AClient

# 创建客户端连接到研究员Agent
client = A2AClient("http://localhost:5000")

# 发送研究请求
response = client.execute_skill("research", "research AI在医疗领域的应用")
print(f"收到响应：{response.get('result')}")

# 输出：
# 收到响应：{'topic': 'AI在医疗领域的应用', 'findings': '关于AI在医疗领域的应用的研究结果...', 'sources': ['来源1', '来源2', '来源3']}
```

<strong>（3）Tạo mạng lưới Agent</strong>

Đối với sự hợp tác của nhiều Agent, chúng ta có thể để nhiều Agent kết nối với nhau:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 1. 创建多个Agent服务
researcher = A2AServer(
    name="researcher",
    description="研究员"
)

@researcher.skill("research")
def do_research(text: str) -> str:
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    return str({"topic": topic, "findings": f"{topic}的研究结果"})

writer = A2AServer(
    name="writer",
    description="撰写员"
)

@writer.skill("write")
def write_article(text: str) -> str:
    import re
    match = re.search(r'write\s+(.+)', text, re.IGNORECASE)
    content = match.group(1).strip() if match else text
    
    # 尝试解析研究数据
    try:
        data = eval(content)
        topic = data.get("topic", "未知主题")
        findings = data.get("findings", "无研究结果")
    except:
        topic = "未知主题"
        findings = content
    
    return f"# {topic}\n\n基于研究：{findings}\n\n文章内容..."

editor = A2AServer(
    name="editor",
    description="编辑"
)

@editor.skill("edit")
def edit_article(text: str) -> str:
    import re
    match = re.search(r'edit\s+(.+)', text, re.IGNORECASE)
    article = match.group(1).strip() if match else text
    
    result = {
        "article": article + "\n\n[已编辑优化]",
        "feedback": "文章质量良好",
        "approved": True
    }
    return str(result)

# 2. 启动所有服务
threading.Thread(target=lambda: researcher.run(port=5000), daemon=True).start()
threading.Thread(target=lambda: writer.run(port=5001), daemon=True).start()
threading.Thread(target=lambda: editor.run(port=5002), daemon=True).start()
time.sleep(2)  # 等待服务启动

# 3. 创建客户端连接到各个Agent
researcher_client = A2AClient("http://localhost:5000")
writer_client = A2AClient("http://localhost:5001")
editor_client = A2AClient("http://localhost:5002")

# 4. 协作流程
def create_content(topic):
    # 步骤1：研究
    research = researcher_client.execute_skill("research", f"research {topic}")
    research_data = research.get('result', '')
    
    # 步骤2：撰写
    article = writer_client.execute_skill("write", f"write {research_data}")
    article_content = article.get('result', '')
    
    # 步骤3：编辑
    final = editor_client.execute_skill("edit", f"edit {article_content}")
    return final.get('result', '')

# 使用
result = create_content("AI在医疗领域的应用")
print(f"\n最终结果：\n{result}")
```

### 10.3.4 Sử dụng A2A tool trong Agent

Bây giờ hãy cùng xem cách tích hợp A2A vào Agent của HelloAgents.

<strong>（1）Sử dụng bộ bọc A2ATool</strong>

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 假设已经有一个研究员Agent服务运行在 http://localhost:5000

# 创建协调者Agent
coordinator = SimpleAgent(name="协调者", llm=llm)

# 添加A2A工具，连接到研究员Agent
researcher_tool = A2ATool(
    name="researcher",
    description="研究员Agent，可以搜索和分析资料",
    agent_url="http://localhost:5000"
)
coordinator.add_tool(researcher_tool)

# 协调者可以调用研究员Agent
response = coordinator.run("请让研究员帮我研究AI在教育领域的应用")
print(response)
```

<strong>（2）Case thực chiến: Hệ thống chăm sóc khách hàng thông minh</strong>

Hãy cùng xây dựng một hệ thống chăm sóc khách hàng thông minh hoàn chỉnh, gồm ba Agent:
- <strong>Lễ tân (Receptionist)</strong>: Phân tích loại vấn đề của khách hàng
- <strong>Chuyên gia kỹ thuật</strong>: Trả lời câu hỏi kỹ thuật
- <strong>Tư vấn bán hàng</strong>: Trả lời câu hỏi bán hàng

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from hello_agents.protocols import A2AServer
import threading
import time
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. 创建技术专家Agent服务
tech_expert = A2AServer(
    name="tech_expert",
    description="技术专家，回答技术问题"
)

@tech_expert.skill("answer")
def answer_tech_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    # 实际应用中，这里会调用LLM或知识库
    return f"技术回答：关于'{question}'，我建议您查看我们的技术文档..."

# 2. 创建销售顾问Agent服务
sales_advisor = A2AServer(
    name="sales_advisor",
    description="销售顾问，回答销售问题"
)

@sales_advisor.skill("answer")
def answer_sales_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    return f"销售回答：关于'{question}'，我们有特别优惠..."

# 3. 启动服务
threading.Thread(target=lambda: tech_expert.run(port=6000), daemon=True).start()
threading.Thread(target=lambda: sales_advisor.run(port=6001), daemon=True).start()
time.sleep(2)

# 4. 创建接待员Agent（使用HelloAgents的SimpleAgent）
receptionist = SimpleAgent(
    name="接待员",
    llm=llm,
    system_prompt="""你是客服接待员，负责：
1. 分析客户问题类型（技术问题 or 销售问题）
2. 将问题转发给相应的专家
3. 整理专家的回答并返回给客户

请保持礼貌和专业。"""
)

# 添加技术专家工具
tech_tool = A2ATool(
    agent_url="http://localhost:6000",
    name="tech_expert",
    description="技术专家，回答技术相关问题"
)
receptionist.add_tool(tech_tool)

# 添加销售顾问工具
sales_tool = A2ATool(
    agent_url="http://localhost:6001",
    name="sales_advisor",
    description="销售顾问，回答价格、购买相关问题"
)
receptionist.add_tool(sales_tool)

# 5. 处理客户咨询
def handle_customer_query(query):
    print(f"\n客户咨询：{query}")
    print("=" * 50)
    response = receptionist.run(query)
    print(f"\n客服回复：{response}")
    print("=" * 50)

# 测试不同类型的问题
if __name__ == "__main__":
    handle_customer_query("你们的API如何调用？")
    handle_customer_query("企业版的价格是多少？")
    handle_customer_query("如何集成到我的Python项目中？")
```

<strong>（3）Cách dùng nâng cao: Đàm phán giữa các Agent</strong>

Giao thức A2A còn hỗ trợ cơ chế đàm phán (negotiation) giữa các Agent:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 创建两个需要协商的Agent
agent1 = A2AServer(
    name="agent1",
    description="Agent 1"
)

@agent1.skill("propose")
def handle_proposal(text: str) -> str:
    """处理协商提案"""
    import re
    
    # 解析提案
    match = re.search(r'propose\s+(.+)', text, re.IGNORECASE)
    proposal_str = match.group(1).strip() if match else text
    
    try:
        proposal = eval(proposal_str)
        task = proposal.get("task")
        deadline = proposal.get("deadline")
        
        # 评估提案
        if deadline >= 7:  # 至少需要7天
            result = {"accepted": True, "message": "接受提案"}
        else:
            result = {
                "accepted": False,
                "message": "时间太紧",
                "counter_proposal": {"deadline": 7}
            }
        return str(result)
    except:
        return str({"accepted": False, "message": "无效的提案格式"})

agent2 = A2AServer(
    name="agent2",
    description="Agent 2"
)

@agent2.skill("negotiate")
def negotiate_task(text: str) -> str:
    """发起协商"""
    import re
    
    # 解析任务和截止日期
    match = re.search(r'negotiate\s+task:(.+?)\s+deadline:(\d+)', text, re.IGNORECASE)
    if match:
        task = match.group(1).strip()
        deadline = int(match.group(2))
        
        # 向agent1发送提案
        proposal = {"task": task, "deadline": deadline}
        return str({"status": "negotiating", "proposal": proposal})
    else:
        return str({"status": "error", "message": "无效的协商请求"})

# 启动服务
threading.Thread(target=lambda: agent1.run(port=7000), daemon=True).start()
threading.Thread(target=lambda: agent2.run(port=7001), daemon=True).start()
```

## 10.4 Thực hành giao thức ANP

Sau khi giao thức MCP giải quyết việc gọi tool, giao thức A2A giải quyết việc hợp tác điểm-điểm giữa Agent, giao thức ANP tập trung giải quyết vấn đề quản lý Agent trong môi trường mạng lưới mở (open network) quy mô lớn.

Trong Mục 10.2 và 10.3, chúng ta đã học về MCP (truy cập tool) và A2A (hợp tác Agent). Bây giờ, hãy cùng học về giao thức ANP (Agent Network Protocol), nó tập trung vào việc xây dựng <strong>mạng lưới Agent quy mô lớn, mở (open)</strong>.

### 10.4.1 Mục tiêu của giao thức

Khi trong một mạng lưới tồn tại một lượng lớn Agent với chức năng khác nhau (ví dụ: xử lý ngôn ngữ tự nhiên, nhận diện hình ảnh, phân tích dữ liệu, v.v.), hệ thống sẽ đối mặt với một loạt thách thức:

- <strong>Khám phá dịch vụ (service discovery)</strong>: Khi một nhiệm vụ mới xuất hiện, làm thế nào để nhanh chóng tìm được Agent có thể xử lý nhiệm vụ đó?
- <strong>Định tuyến thông minh (intelligent routing)</strong>: Nếu nhiều Agent đều có thể xử lý cùng một nhiệm vụ, làm thế nào để chọn ra Agent phù hợp nhất (dựa trên tải, chi phí, v.v.) và phân công nhiệm vụ cho nó?
- <strong>Mở rộng động (dynamic scaling)</strong>: Làm thế nào để Agent mới gia nhập mạng lưới có thể được các thành viên khác khám phá và gọi tới?

Mục tiêu thiết kế của ANP là cung cấp một bộ cơ chế chuẩn hóa, để giải quyết các vấn đề về khám phá dịch vụ, lựa chọn định tuyến và khả năng mở rộng mạng lưới nêu trên.

Để đạt được mục tiêu thiết kế của mình, ANP định nghĩa một số khái niệm cốt lõi như sau, như Bảng 10.8 minh họa:

<div align="center">
  <p>Bảng 10.8 Các khái niệm cốt lõi của ANP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-8.png" alt="" width="85%"/>
</div>

Chúng tôi cũng mượn [hướng dẫn nhập môn](https://github.com/agent-network-protocol/AgentNetworkProtocol/blob/main/docs/chinese/ANP入门指南.md) chính thức để giới thiệu thiết kế kiến trúc của ANP, như Hình 10.9 minh họa

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-9.png" alt="" width="85%"/>
  <p>Hình 10.9 Luồng tổng thể của ANP</p>
</div>


Trong sơ đồ luồng này, chủ yếu bao gồm các bước sau:

<strong>1. Khám phá và khớp dịch vụ:</strong> Trước tiên, Agent A thông qua một dịch vụ khám phá (discovery service) công khai, dựa trên mô tả ngữ nghĩa (semantic) hoặc chức năng để truy vấn, nhằm xác định Agent B phù hợp với nhu cầu nhiệm vụ của nó. Dịch vụ khám phá này xây dựng chỉ mục (index) bằng cách crawl trước các endpoint chuẩn mà mỗi Agent công khai (`.well-known/agent-descriptions`), từ đó thực hiện việc khớp động giữa bên có nhu cầu dịch vụ và bên cung cấp dịch vụ.

<strong>2. Xác thực danh tính dựa trên DID:</strong> Khi bắt đầu tương tác, Agent A dùng private key của mình để ký (sign) vào request có chứa DID của chính nó. Sau khi Agent B nhận được, nó phân giải (resolve) DID đó để lấy public key tương ứng, và dùng nó để xác minh tính xác thực của chữ ký cũng như tính toàn vẹn của request, từ đó thiết lập giao tiếp đáng tin cậy giữa hai bên.

<strong>3. Thực thi dịch vụ chuẩn hóa:</strong> Sau khi xác thực danh tính thành công, Agent B phản hồi request, hai bên tiến hành trao đổi dữ liệu hoặc gọi dịch vụ (như đặt chỗ, truy vấn, v.v.) theo giao diện chuẩn và định dạng dữ liệu đã được định nghĩa trước. Luồng tương tác chuẩn hóa là nền tảng để đạt được khả năng tương tác (interoperability) xuyên nền tảng, xuyên hệ thống.

Tóm lại, cốt lõi của cơ chế này là sử dụng DID để xây dựng một nền tảng tin cậy (trust) phi tập trung, và dựa vào giao thức mô tả chuẩn hóa để thực hiện khám phá dịch vụ động. Phương pháp này giúp các Agent có thể hình thành mạng lưới hợp tác trên Internet một cách an toàn, hiệu quả mà không cần điều phối trung tâm.

### 10.4.2 Sử dụng khám phá dịch vụ (service discovery) của ANP

<strong>（1）Tạo trung tâm khám phá dịch vụ</strong>

```python
from hello_agents.protocols import ANPDiscovery, register_service

# 创建服务发现中心
discovery = ANPDiscovery()

# 注册Agent服务
register_service(
    discovery=discovery,
    service_id="nlp_agent_1",
    service_name="NLP处理专家A",
    service_type="nlp",
    capabilities=["text_analysis", "sentiment_analysis", "ner"],
    endpoint="http://localhost:8001",
    metadata={"load": 0.3, "price": 0.01, "version": "1.0.0"}
)

register_service(
    discovery=discovery,
    service_id="nlp_agent_2",
    service_name="NLP处理专家B",
    service_type="nlp",
    capabilities=["text_analysis", "translation"],
    endpoint="http://localhost:8002",
    metadata={"load": 0.7, "price": 0.02, "version": "1.1.0"}
)

print("✅ 服务注册完成")
```

<strong>（2）Khám phá dịch vụ</strong>

```python
from hello_agents.protocols import discover_service

# 按类型查找
nlp_services = discover_service(discovery, service_type="nlp")
print(f"找到 {len(nlp_services)} 个NLP服务")

# 选择负载最低的服务
best_service = min(nlp_services, key=lambda s: s.metadata.get("load", 1.0))
print(f"最佳服务：{best_service.service_name} (负载: {best_service.metadata['load']})")
```

<strong>（3）Xây dựng mạng lưới Agent</strong>

```python
from hello_agents.protocols import ANPNetwork

# 创建网络
network = ANPNetwork(network_id="ai_cluster")

# 添加节点
for service in discovery.list_all_services():
    network.add_node(service.service_id, service.endpoint)

# 建立连接（根据能力匹配）
network.connect_nodes("nlp_agent_1", "nlp_agent_2")

stats = network.get_network_stats()
print(f"✅ 网络构建完成，共 {stats['total_nodes']} 个节点")
```

### 10.4.3 Case thực chiến

Hãy cùng xây dựng một hệ thống lập lịch nhiệm vụ phân tán (distributed task scheduling) hoàn chỉnh:

```python
from hello_agents.protocols import ANPDiscovery, register_service
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin import ANPTool
import random
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. 创建服务发现中心
discovery = ANPDiscovery()

# 2. 注册多个计算节点
for i in range(10):
    register_service(
        discovery=discovery,
        service_id=f"compute_node_{i}",
        service_name=f"计算节点{i}",
        service_type="compute",
        capabilities=["data_processing", "ml_training"],
        endpoint=f"http://node{i}:8000",
        metadata={
            "load": random.uniform(0.1, 0.9),
            "cpu_cores": random.choice([4, 8, 16]),
            "memory_gb": random.choice([16, 32, 64]),
            "gpu": random.choice([True, False])
        }
    )

print(f"✅ 注册了 {len(discovery.list_all_services())} 个计算节点")

# 3. 创建任务调度Agent
scheduler = SimpleAgent(
    name="任务调度器",
    llm=llm,
    system_prompt="""你是一个智能任务调度器，负责：
1. 分析任务需求
2. 选择最合适的计算节点
3. 分配任务

选择节点时考虑：负载、CPU核心数、内存、GPU等因素。"""
)

# 添加ANP工具
anp_tool = ANPTool(
    name="service_discovery",
    description="服务发现工具，可以查找和选择计算节点",
    discovery=discovery
)
scheduler.add_tool(anp_tool)

# 4. 智能任务分配
def assign_task(task_description):
    print(f"\n任务：{task_description}")
    print("=" * 50)

    # 让Agent智能选择节点
    response = scheduler.run(f"""
    请为以下任务选择最合适的计算节点：
    {task_description}

    要求：
    1. 列出所有可用节点
    2. 分析每个节点的特点
    3. 选择最合适的节点
    4. 说明选择理由
    """)

    print(response)
    print("=" * 50)

# 测试不同类型的任务
assign_task("训练一个大型深度学习模型，需要GPU支持")
assign_task("处理大量文本数据，需要高内存")
assign_task("运行轻量级数据分析任务")
```

Đây là một ví dụ về cân bằng tải (load balancing)

```python
from hello_agents.protocols import ANPDiscovery, register_service
import random

# 创建服务发现中心
discovery = ANPDiscovery()

# 注册多个相同类型的服务
for i in range(5):
    register_service(
        discovery=discovery,
        service_id=f"api_server_{i}",
        service_name=f"API服务器{i}",
        service_type="api",
        capabilities=["rest_api"],
        endpoint=f"http://api{i}:8000",
        metadata={"load": random.uniform(0.1, 0.9)}
    )

# 负载均衡函数
def get_best_server():
    """选择负载最低的服务器"""
    servers = discovery.discover_services(service_type="api")
    if not servers:
        return None

    best = min(servers, key=lambda s: s.metadata.get("load", 1.0))
    return best

# 模拟请求分配
for i in range(10):
    server = get_best_server()
    print(f"请求 {i+1} -> {server.service_name} (负载: {server.metadata['load']:.2f})")

    # 更新负载（模拟）
    server.metadata["load"] += 0.1
```

## 10.5 Xây dựng MCP server tùy chỉnh

Trong các phần trước, chúng ta đã học cách sử dụng các dịch vụ MCP có sẵn. Và cũng đã hiểu được đặc điểm của các giao thức khác nhau. Bây giờ, hãy cùng học cách xây dựng MCP server của riêng bạn.

### 10.5.1 Tạo MCP server đầu tiên của bạn

<strong>（1）Tại sao cần xây dựng MCP server tùy chỉnh?</strong>

Mặc dù có thể dùng trực tiếp các dịch vụ MCP công khai, nhưng trong nhiều tình huống ứng dụng thực tế, cần xây dựng MCP server tùy chỉnh để đáp ứng nhu cầu cụ thể.

Các động lực chính bao gồm những điểm sau:

- <strong>Đóng gói business logic</strong>: Đóng gói các quy trình nghiệp vụ hoặc thao tác phức tạp đặc thù trong doanh nghiệp thành các tool MCP chuẩn hóa, để Agent gọi một cách thống nhất.
- <strong>Truy cập dữ liệu riêng tư (private data)</strong>: Tạo một giao diện hoặc proxy an toàn, có kiểm soát, dùng để truy cập cơ sở dữ liệu nội bộ, API hoặc các nguồn dữ liệu riêng tư khác không thể công khai ra Internet.
- <strong>Tối ưu hóa hiệu năng chuyên biệt</strong>: Tối ưu hóa sâu cho các tình huống ứng dụng gọi tần suất cao hoặc yêu cầu độ trễ phản hồi (latency) khắt khe.
- <strong>Mở rộng chức năng tùy chỉnh</strong>: Triển khai các chức năng cụ thể mà dịch vụ MCP chuẩn chưa cung cấp, ví dụ tích hợp model thuật toán độc quyền hoặc kết nối với thiết bị phần cứng cụ thể.

<strong>（2）Case giảng dạy: MCP server tra cứu thời tiết</strong>

Hãy bắt đầu từ một server tra cứu thời tiết đơn giản, học từng bước cách phát triển MCP server:

```python
#!/usr/bin/env python3
"""天气查询 MCP 服务器"""

import json
import requests
import os
from datetime import datetime
from typing import Dict, Any
from hello_agents.protocols import MCPServer

# 创建 MCP 服务器
weather_server = MCPServer(name="weather-server", description="真实天气查询服务")

CITY_MAP = {
    "北京": "Beijing", "上海": "Shanghai", "广州": "Guangzhou",
    "深圳": "Shenzhen", "杭州": "Hangzhou", "成都": "Chengdu",
    "重庆": "Chongqing", "武汉": "Wuhan", "西安": "Xi'an",
    "南京": "Nanjing", "天津": "Tianjin", "苏州": "Suzhou"
}


def get_weather_data(city: str) -> Dict[str, Any]:
    """从 wttr.in 获取天气数据"""
    city_en = CITY_MAP.get(city, city)
    url = f"https://wttr.in/{city_en}?format=j1"
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    current = data["current_condition"][0]

    return {
        "city": city,
        "temperature": float(current["temp_C"]),
        "feels_like": float(current["FeelsLikeC"]),
        "humidity": int(current["humidity"]),
        "condition": current["weatherDesc"][0]["value"],
        "wind_speed": round(float(current["windspeedKmph"]) / 3.6, 1),
        "visibility": float(current["visibility"]),
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }


# 定义工具函数
def get_weather(city: str) -> str:
    """获取指定城市的当前天气"""
    try:
        weather_data = get_weather_data(city)
        return json.dumps(weather_data, ensure_ascii=False, indent=2)
    except Exception as e:
        return json.dumps({"error": str(e), "city": city}, ensure_ascii=False)


def list_supported_cities() -> str:
    """列出所有支持的中文城市"""
    result = {"cities": list(CITY_MAP.keys()), "count": len(CITY_MAP)}
    return json.dumps(result, ensure_ascii=False, indent=2)


def get_server_info() -> str:
    """获取服务器信息"""
    info = {
        "name": "Weather MCP Server",
        "version": "1.0.0",
        "tools": ["get_weather", "list_supported_cities", "get_server_info"]
    }
    return json.dumps(info, ensure_ascii=False, indent=2)


# 注册工具到服务器
weather_server.add_tool(get_weather)
weather_server.add_tool(list_supported_cities)
weather_server.add_tool(get_server_info)


if __name__ == "__main__":
    weather_server.run()
```

<strong>（3）Kiểm thử MCP server tùy chỉnh</strong>

Sau đó tạo script kiểm thử:

```python
#!/usr/bin/env python3
"""测试天气查询 MCP 服务器"""

import asyncio
import json
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'HelloAgents'))
from hello_agents.protocols.mcp.client import MCPClient


async def test_weather_server():
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    client = MCPClient(["python", server_script])

    try:
        async with client:
            # 测试1: 获取服务器信息
            info = json.loads(await client.call_tool("get_server_info", {}))
            print(f"服务器: {info['name']} v{info['version']}")

            # 测试2: 列出支持的城市
            cities = json.loads(await client.call_tool("list_supported_cities", {}))
            print(f"支持城市: {cities['count']} 个")

            # 测试3: 查询北京天气
            weather = json.loads(await client.call_tool("get_weather", {"city": "北京"}))
            if "error" not in weather:
                print(f"\n北京天气: {weather['temperature']}°C, {weather['condition']}")

            # 测试4: 查询深圳天气
            weather = json.loads(await client.call_tool("get_weather", {"city": "深圳"}))
            if "error" not in weather:
                print(f"深圳天气: {weather['temperature']}°C, {weather['condition']}")

            print("\n✅ 所有测试完成！")

    except Exception as e:
        print(f"❌ 测试失败: {e}")


if __name__ == "__main__":
    asyncio.run(test_weather_server())
```

<strong>（4）Sử dụng MCP server tùy chỉnh trong Agent</strong>

```python
"""在 Agent 中使用天气 MCP 服务器"""

import os
from dotenv import load_dotenv
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

load_dotenv()


def create_weather_assistant():
    """创建天气助手"""
    llm = HelloAgentsLLM()

    assistant = SimpleAgent(
        name="天气助手",
        llm=llm,
        system_prompt="""你是天气助手，可以查询城市天气。
使用 mcp_get_weather 工具查询天气，支持中文城市名。
"""
    )

    # 添加天气 MCP 工具
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    weather_tool = MCPTool(server_command=["python", server_script])

    # 显式展开并注册 MCP 子工具
    expanded_tools = weather_tool.get_expanded_tools()
    if not expanded_tools:
        raise RuntimeError("未发现天气 MCP 子工具，请检查服务脚本、依赖和启动日志。")

    for expanded_tool in expanded_tools:
        assistant.add_tool(expanded_tool)

    return assistant


def demo():
    """演示"""
    assistant = create_weather_assistant()

    print("\n查询北京天气：")
    response = assistant.run("北京今天天气怎么样？")
    print(f"回答: {response}\n")


def interactive():
    """交互模式"""
    assistant = create_weather_assistant()

    while True:
        user_input = input("\n你: ").strip()
        if user_input.lower() in ['quit', 'exit']:
            break
        response = assistant.run(user_input)
        print(f"助手: {response}")


if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1 and sys.argv[1] == "demo":
        demo()
    else:
        interactive()
```

```
🔗 连接到 MCP 服务器...
✅ 连接成功！
🔌 连接已断开
✅ 工具 'mcp_get_weather' 已注册。
✅ 工具 'mcp_list_supported_cities' 已注册。
✅ 工具 'mcp_get_server_info' 已注册。

你: 我想查询北京的天气
🔗 连接到 MCP 服务器...
✅ 连接成功！
🔌 连接已断开
助手: 当前北京的天气情况如下：

- 温度：10.0°C
- 体感温度：9.0°C
- 湿度：94%
- 天气状况：小雨
- 风速：1.7米/秒
- 能见度：10.0公里
- 时间戳：2025年10月9日 13:46:40

请注意携带雨具，并根据天气变化适当调整着装。
```

### 10.5.2 Phát hành MCP server

Chúng ta đã tạo một MCP server tra cứu thời tiết thực tế. Bây giờ, hãy cùng phát hành nó lên nền tảng Smithery, để các nhà phát triển trên toàn thế giới đều có thể sử dụng dịch vụ của chúng ta.

 （1）Smithery là gì?

[Smithery](https://smithery.ai/) là nền tảng phát hành chính thức của MCP server, tương tự như PyPI của Python hay npm của Node.js. Thông qua Smithery, người dùng có thể:

- 🔍 Khám phá và tìm kiếm MCP server
- 📦 Cài đặt MCP server chỉ với một cú click
- 📊 Xem thống kê sử dụng và đánh giá của server
- 🔄 Tự động nhận bản cập nhật của server

（2）Chuẩn bị phát hành
Trước tiên, chúng ta cần sắp xếp dự án theo định dạng phát hành chuẩn, thư mục này đã được chuẩn bị sẵn trong thư mục `code`, bạn có thể tham khảo:

```
weather-mcp-server/
├── README.md           # 项目说明文档
├── LICENSE            # 开源许可证
├── Dockerfile         # Docker 构建配置（推荐）
├── pyproject.toml     # Python 项目配置（必需）
├── requirements.txt   # Python 依赖
├── smithery.yaml      # Smithery 配置文件（必需）
└── server.py          # MCP 服务器主文件
```

Cần lưu ý rằng, `smithery.yaml` là file cấu hình của nền tảng Smithery:
```yaml
name: weather-mcp-server
displayName: Weather MCP Server
description: Real-time weather query MCP server based on HelloAgents framework
version: 1.0.0
author: HelloAgents Team
homepage: https://github.com/yourusername/weather-mcp-server
license: MIT
categories:
  - weather
  - data
tags:
  - weather
  - real-time
  - helloagents
  - wttr
runtime: container
build:
  dockerfile: Dockerfile
  dockerBuildPath: .
startCommand:
  type: http
tools:
  - name: get_weather
    description: Get current weather for a city
  - name: list_supported_cities
    description: List all supported cities
  - name: get_server_info
    description: Get server information
```

Giải thích cấu hình:

- `name`: Định danh duy nhất của server (chữ thường, phân cách bằng dấu gạch nối)
- `displayName`: Tên hiển thị
- `description`: Mô tả ngắn gọn
- `version`: Số phiên bản (tuân theo versioning ngữ nghĩa - semantic version)
- `runtime`: Môi trường chạy (python/node)
- `entrypoint`: File entrypoint
- `tools`: Danh sách tool

`pyproject.toml` là file cấu hình chuẩn của dự án Python, Smithery yêu cầu bắt buộc phải có file này, vì sau đó sẽ đóng gói thành một server:

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "weather-mcp-server"
version = "1.0.0"
description = "Real-time weather query MCP server based on HelloAgents framework"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "HelloAgents Team", email = "xxx"}
]
requires-python = ">=3.10"
dependencies = [
    "hello-agents>=0.2.1",
    "requests>=2.31.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/weather-mcp-server"
Repository = "https://github.com/yourusername/weather-mcp-server"
"Bug Tracker" = "https://github.com/yourusername/weather-mcp-server/issues"

[tool.setuptools]
py-modules = ["server"]
```


Giải thích cấu hình:

- `[build-system]`: Chỉ định công cụ build (setuptools)
- `[project]`: Metadata của dự án
  - `name`: Tên dự án
  - `version`: Số phiên bản (tuân theo semantic version)
  - `dependencies`: Danh sách dependency của dự án
  - `requires-python`: Yêu cầu phiên bản Python
- `[project.urls]`: Các link liên quan tới dự án
- `[tool.setuptools]`: Cấu hình setuptools

Mặc dù Smithery sẽ tự động tạo Dockerfile, nhưng việc cung cấp Dockerfile tùy chỉnh có thể đảm bảo việc triển khai (deploy) thành công:

```dockerfile
# Multi-stage build for weather-mcp-server
FROM python:3.12-slim-bookworm as base

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# Copy project files
COPY pyproject.toml requirements.txt ./
COPY server.py ./

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Set environment variables
ENV PYTHONUNBUFFERED=1
ENV PORT=8081

# Expose port (Smithery uses 8081)
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import sys; sys.exit(0)"

# Run the MCP server
CMD ["python", "server.py"]
```

Giải thích cấu hình Dockerfile:

- <strong>Image cơ sở</strong>: `python:3.12-slim-bookworm` - image Python nhẹ (lightweight)
- <strong>Thư mục làm việc</strong>: `/app` - thư mục gốc của ứng dụng
- <strong>Port</strong>: `8081` - port chuẩn của nền tảng Smithery
- <strong>Lệnh khởi động</strong>: `python server.py` - chạy MCP server

Ở đây, chúng ta cần fork repository `hello-agents`, lấy source code trong `code`, và dùng GitHub của chính mình để tạo một repository tên là `weather-mcp-server`, đổi `yourusername` thành username GitHub của bạn.

（3）Nộp lên Smithery

Mở trình duyệt, truy cập [https://smithery.ai/](https://smithery.ai/). Đăng nhập Smithery bằng tài khoản GitHub. Nhấn nút "Publish Server" trên trang, nhập URL repository GitHub của bạn: `https://github.com/yourusername/weather-mcp-server`, sau đó chờ phát hành.

Sau khi phát hành xong, bạn sẽ thấy một trang tương tự như thế này, như Hình 10.10 minh họa:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-10.png" alt="" width="85%"/>
  <p>Hình 10.10 Trang phát hành thành công của Smithery </p>
</div>



Khi server đã phát hành thành công, người dùng có thể sử dụng bằng các cách sau:

Cách 1: Thông qua Smithery CLI

```bash
# 安装 Smithery CLI
npm install -g @smithery/cli

# 安装你的服务器
smithery install weather-mcp-server
```

Cách 2: Cấu hình trong Claude Desktop

```json
{
  "mcpServers": {
    "weather": {
      "command": "smithery",
      "args": ["run", "weather-mcp-server"]
    }
  }
}
```

Cách 3: Sử dụng trong HelloAgents

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin.protocol_tools import MCPTool

agent = SimpleAgent(name="天气助手", llm=HelloAgentsLLM())

# 使用 Smithery 安装的服务器
weather_tool = MCPTool(
    server_command=["smithery", "run", "weather-mcp-server"]
)
agent.add_tool(weather_tool)

response = agent.run("北京今天天气怎么样？")
```

Tất nhiên, đây chỉ là ví dụ minh họa, còn nhiều cách sử dụng khác bạn có thể tự khám phá, Hình 10.11 dưới đây cho thấy thông tin sẽ xuất hiện khi MCP tool được phát hành thành công, hiển thị tên dịch vụ "天气" (thời tiết), định danh duy nhất `@jjyaoao/weather-mcp-server`, cũng như thông tin trạng thái. Khu vực Tools chính là các phương thức chúng ta vừa triển khai, khu vực Connect cung cấp thông tin kỹ thuật cần thiết để kết nối và sử dụng dịch vụ này, bao gồm <strong>URL truy cập</strong> của dịch vụ và các <strong>đoạn code cấu hình</strong> cho nhiều ngôn ngữ/môi trường khác nhau. Nếu muốn tìm hiểu sâu hơn, bạn có thể nhấn vào [link](https://smithery.ai/server/@jjyaoao/weather-mcp-server) này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-11.png" alt="" width="85%"/>
  <p>Hình 10.11 MCP tool phát hành thành công trên Smithery </p>
</div>

Bây giờ là lúc để bạn tạo ra MCP server của riêng mình!



## 10.6 Tổng kết chương

Chương này đã giới thiệu một cách hệ thống ba giao thức giao tiếp cốt lõi của Agent: MCP, A2A và ANP, đồng thời khám phá triết lý thiết kế, tình huống ứng dụng và phương pháp thực hành của chúng.

<strong>Định vị giao thức:</strong>

- <strong>MCP (Model Context Protocol)</strong>: Đóng vai trò cầu nối giữa Agent và tool, cung cấp giao diện truy cập tool thống nhất, phù hợp để tăng cường năng lực cho một Agent đơn lẻ.
- <strong>A2A (Agent-to-Agent Protocol)</strong>: Đóng vai trò hệ thống đối thoại giữa các Agent, hỗ trợ giao tiếp trực tiếp và đàm phán nhiệm vụ, phù hợp cho sự hợp tác chặt chẽ của các đội ngũ quy mô nhỏ.
- <strong>ANP (Agent Network Protocol)</strong>: Đóng vai trò "Internet" của Agent, cung cấp cơ chế khám phá dịch vụ, định tuyến và cân bằng tải, phù hợp để xây dựng mạng lưới Agent quy mô lớn, mở.

<strong>Phương án tích hợp của HelloAgents</strong>

Trong framework `HelloAgents`, ba giao thức này được trừu tượng hóa thống nhất thành Tool, thực hiện tích hợp liền mạch, cho phép nhà phát triển linh hoạt thêm các năng lực giao tiếp ở nhiều tầng khác nhau cho Agent:

```python
# 统一的Tool接口
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 所有协议都可以作为Tool添加到Agent
agent.add_tool(MCPTool(...))
agent.add_tool(A2ATool(...))
agent.add_tool(ANPTool(...))
```

<strong>Tổng kết kinh nghiệm thực chiến</strong>

- Ưu tiên tận dụng các dịch vụ MCP của cộng đồng đã hoàn thiện, để giảm bớt việc phát triển lặp lại không cần thiết.
- Chọn giao thức phù hợp theo quy mô hệ thống: tình huống hợp tác quy mô nhỏ khuyến nghị dùng A2A, tình huống mạng lưới quy mô lớn nên dùng ANP.

Sau khi hoàn thành chương này, khuyến nghị bạn:

1. <strong>Thực hành tay</strong>：
   - Xây dựng MCP server của riêng bạn
   - Tận dụng giao thức để tạo hệ thống hợp tác đa Agent
   - Chiến lược kết hợp MCP, A2A và ANP
2. <strong>Học sâu hơn</strong>：
   - Đọc tài liệu chính thức của MCP: https://modelcontextprotocol.io
   - Đọc tài liệu chính thức của A2A: https://a2a-protocol.org/latest/
   - Đọc tài liệu chính thức của ANP: https://agent-network-protocol.com/guide/
3. <strong>Tham gia cộng đồng</strong>：
   - Đóng góp dịch vụ MCP mới cho cộng đồng
   - Chia sẻ case triển khai Agent do cá nhân phát triển
   - Tham gia thảo luận tiêu chuẩn kỹ thuật của các giao thức liên quan, cũng có thể đặt câu hỏi ở Issue hoặc trực tiếp giúp Helloagents hỗ trợ example case mới

<strong>Chúc mừng bạn đã hoàn thành Chương 10!</strong>

Bây giờ bạn đã nắm được kiến thức cốt lõi về giao thức giao tiếp của Agent. Hãy tiếp tục cố gắng! 🚀

## Bài tập

> <strong>Gợi ý</strong>：Một số bài tập không có đáp án chuẩn, trọng tâm là rèn luyện khả năng hiểu biết toàn diện và năng lực thực hành của người học về giao thức giao tiếp của Agent.

1. Chương này đã giới thiệu ba giao thức giao tiếp của Agent: MCP, A2A và ANP. Hãy phân tích:

   - Trong Mục 10.1.2 đã so sánh triết lý thiết kế của ba giao thức. Hãy phân tích sâu hơn: tại sao MCP nhấn mạnh "chia sẻ ngữ cảnh" (context sharing), A2A nhấn mạnh "hợp tác kiểu đối thoại" (conversational collaboration), còn ANP nhấn mạnh "topology mạng lưới" (network topology)? Các triết lý thiết kế này lần lượt giải quyết vấn đề cốt lõi nào?
   - Giả sử bạn muốn xây dựng một "hệ thống chăm sóc khách hàng thông minh", cần các chức năng sau: (1) truy cập cơ sở dữ liệu khách hàng và hệ thống đơn hàng; (2) nhiều Agent chăm sóc khách hàng chuyên biệt hợp tác xử lý vấn đề phức tạp; (3) hỗ trợ lượng lớn request đồng thời (concurrent) từ người dùng. Hãy chọn giao thức phù hợp nhất cho mỗi chức năng, và giải thích lý do.
   - Ba giao thức có thể kết hợp sử dụng với nhau không? Hãy thiết kế một tình huống ứng dụng thực tế, minh họa cách sử dụng đồng thời MCP, A2A và ANP để xây dựng một hệ thống Agent hoàn chỉnh. Vẽ sơ đồ kiến trúc hệ thống và giải thích trách nhiệm của từng giao thức.

2. MCP (Model Context Protocol) là giao thức chuẩn cho giao tiếp giữa Agent và tool. Dựa trên nội dung Mục 10.2, hãy suy nghĩ sâu hơn:

   > <strong>Gợi ý</strong>：Đây là một bài tập thực hành tay, khuyến nghị thao tác thực tế

   - Trong phần triển khai MCP server ở Mục 10.2.3, chúng ta đã định nghĩa các phương thức cốt lõi như `list_tools`, `call_tool`. Hãy mở rộng bản triển khai này, thêm một MCP server mới, cung cấp các tool sau: (1) tool truy vấn cơ sở dữ liệu; (2) tool trực quan hóa dữ liệu (data visualization); (3) tool tạo báo cáo. Yêu cầu các tool có thể hợp tác với nhau để hoàn thành nhiệm vụ phân tích dữ liệu phức tạp.
   - Giao thức MCP hỗ trợ hai khái niệm quan trọng "Resource" và "Prompt", nhưng chương này chủ yếu tập trung vào "Tool". Hãy tham khảo tài liệu chính thức của MCP, tìm hiểu mục đích thiết kế của Resource và Prompt, và thiết kế một tình huống ứng dụng, minh họa cách tận dụng cả ba khái niệm cốt lõi này để xây dựng một hệ thống Agent mạnh mẽ hơn.
   - MCP sử dụng JSON-RPC 2.0 làm giao thức giao tiếp ở tầng dưới, giao tiếp liên tiến trình (inter-process) qua stdio. Hãy phân tích: thiết kế này có ưu điểm và hạn chế gì? Nếu cần hỗ trợ MCP server từ xa (remote) (truy cập qua HTTP/WebSocket), nên mở rộng bản triển khai hiện tại như thế nào?

3. A2A (Agent-to-Agent Protocol) hỗ trợ hợp tác kiểu đối thoại giữa các Agent. Dựa trên nội dung Mục 10.3, hãy hoàn thành các bài thực hành mở rộng sau:

   > <strong>Gợi ý</strong>：Đây là một bài tập thực hành tay, khuyến nghị thao tác thực tế

   - Trong case "đội ngũ nghiên cứu" ở Mục 10.3.4, nhà nghiên cứu và người viết hợp tác hoàn thành việc viết bài luận thông qua giao thức A2A. Hãy mở rộng case này, thêm Agent thứ ba "người phản biện" (Reviewer), nó có thể đánh giá chất lượng bài luận và đưa ra đề xuất chỉnh sửa. Hãy thiết kế luồng hợp tác giữa ba Agent, và triển khai code hoàn chỉnh.
   - Giao thức A2A định nghĩa các loại message như `task`, `task_result`. Hãy phân tích: nếu trong quá trình hợp tác xuất hiện xung đột (conflict) (ví dụ hai Agent có ý kiến khác nhau về cùng một vấn đề), nên thiết kế cơ chế giải quyết xung đột như thế nào? Hãy mở rộng giao thức A2A, thêm các loại message như "đàm phán" (negotiation) và "biểu quyết" (voting).
   - So sánh giao thức A2A với các framework đa Agent như AutoGen, CAMEL đã giới thiệu ở Chương 6: A2A với vai trò là một giao thức chuẩn, có mối quan hệ như thế nào với các framework này? Chúng có thể thay thế lẫn nhau không? Hãy thiết kế một phương án, giúp Agent dựa trên giao thức A2A có thể giao tiếp với Agent trong framework AutoGen.

4. ANP (Agent Network Protocol) hỗ trợ mạng lưới Agent quy mô lớn. Dựa trên nội dung Mục 10.4, hãy phân tích sâu:

   - Trong Mục 10.4.2 đã giới thiệu thiết kế topology mạng lưới của ANP, bao gồm cấu trúc hình sao (star), hình lưới (mesh), phân lớp (hierarchical). Hãy phân tích: nên chọn cấu trúc topology nào trong tình huống nào? Nếu quy mô mạng lưới mở rộng từ 10 Agent lên 1000 Agent, cấu trúc topology nên phát triển như thế nào?
   - Giao thức ANP hỗ trợ cơ chế "định tuyến" (routing) và "khám phá" (discovery), giúp Agent có thể động tìm được đối tác hợp tác phù hợp. Hãy thiết kế một "thuật toán định tuyến thông minh" (intelligent routing algorithm): dựa trên loại nhiệm vụ, năng lực của Agent, tải của mạng lưới và các yếu tố khác, tự động chọn đường định tuyến message tối ưu.
   - Trong case "thành phố thông minh" ở Mục 10.4.4, nhiều Agent hợp tác quản lý hệ thống thành phố. Hãy suy nghĩ: nếu một Agent then chốt (như Agent quản lý giao thông) gặp sự cố, toàn bộ hệ thống nên ứng phó như thế nào? Hãy thiết kế một "cơ chế chịu lỗi" (fault tolerance), bao gồm chức năng phát hiện sự cố, chuyển đổi dự phòng (backup switching), khôi phục trạng thái.

5. Tính bảo mật và bảo vệ quyền riêng tư của giao thức giao tiếp Agent là vấn đề then chốt trong ứng dụng thực tế. Hãy suy nghĩ:

   - Trong phần triển khai MCP client ở Mục 10.2.4, Agent có thể gọi bất kỳ tool nào mà MCP server cung cấp. Hãy phân tích: thiết kế này tồn tại rủi ro bảo mật gì? Nếu MCP server cung cấp các thao tác nguy hiểm (như xóa file, thực thi lệnh hệ thống), nên thiết kế cơ chế kiểm soát quyền (permission control) như thế nào?
   - Giao thức A2A và ANP liên quan đến giao tiếp giữa nhiều Agent, có thể chứa thông tin nhạy cảm (như dữ liệu riêng tư người dùng, bí mật kinh doanh). Hãy thiết kế một phương án "mã hóa đầu cuối" (end-to-end encryption): đảm bảo message không bị nghe lén hay giả mạo trong quá trình truyền tải, đồng thời hỗ trợ xác thực danh tính và kiểm soát truy cập của Agent.
   - Trong mạng lưới Agent quy mô lớn, các Agent độc hại (malicious) có thể gửi thông tin giả mạo, phát động tấn công từ chối dịch vụ (DoS) hoặc đánh cắp dữ liệu của Agent khác. Hãy thiết kế một "hệ thống đánh giá độ tin cậy" (trust evaluation system): dựa trên hành vi lịch sử, chất lượng hợp tác, đánh giá cộng đồng và các yếu tố khác của Agent, đánh giá động độ tin cậy của từng Agent, và điều chỉnh chiến lược giao tiếp dựa trên đó.

## Tài liệu tham khảo

[1] Anthropic. (2024). *Model Context Protocol*. Retrieved October 7, 2025, from https://modelcontextprotocol.io/

[2] The A2A Project. (2025). *A2A Protocol: An open protocol for agent-to-agent communication*. Retrieved October 7, 2025, from https://a2a-protocol.org/

[3] Chang, G., Lin, E., Yuan, C., Cai, R., Chen, B., Xie, X., & Zhang, Y. (2025). *Agent Network Protocol technical white paper*. arXiv. https://doi.org/10.48550/arXiv.2508.00007
