# Chương 4 Xây dựng các Paradigm kinh điển của Agent

Trong chương trước, chúng ta đã tìm hiểu sâu về Large Language Model (LLM) với vai trò là "bộ não" của Agent hiện đại. Chúng ta đã hiểu được kiến trúc Transformer bên trong nó, các phương pháp tương tác với nó, cũng như ranh giới năng lực của nó. Bây giờ, đã đến lúc biến những kiến thức lý thuyết đó thành thực hành, tự tay xây dựng Agent.

Năng lực cốt lõi của một Agent hiện đại nằm ở khả năng kết nối năng lực suy luận (reasoning) của LLM với thế giới bên ngoài. Nó có thể tự chủ hiểu ý định của người dùng, phân rã các nhiệm vụ phức tạp, và thông qua việc gọi một loạt các "công cụ" (Tools) như code interpreter, công cụ tìm kiếm, API, v.v., để thu thập thông tin, thực hiện thao tác, và cuối cùng đạt được mục tiêu. Tuy nhiên, Agent không phải là vạn năng, nó cũng phải đối mặt với vấn đề "ảo giác" (hallucination) từ chính mô hình lớn, khả năng rơi vào vòng lặp suy luận trong các nhiệm vụ phức tạp, cũng như việc sử dụng sai công cụ — những điều này cũng tạo nên ranh giới năng lực của Agent.

Để tổ chức tốt hơn quá trình "suy nghĩ" (thinking) và "hành động" (acting) của Agent, giới công nghệ đã xuất hiện nhiều paradigm kiến trúc kinh điển. Trong chương này, chúng ta sẽ tập trung vào ba paradigm tiêu biểu nhất, và từng bước hiện thực hóa chúng từ con số 0:

- **ReAct (Reasoning and Acting):** Một paradigm kết hợp chặt chẽ "suy nghĩ" và "hành động", để Agent vừa nghĩ vừa làm, điều chỉnh động.
- **Plan-and-Solve:** Một paradigm "suy nghĩ kỹ trước khi hành động", Agent trước tiên tạo ra một kế hoạch hành động hoàn chỉnh, sau đó thực hiện nghiêm ngặt.
- **Reflection:** Một paradigm trao cho Agent khả năng "phản tư" (reflection), thông qua tự phê bình và sửa chữa để tối ưu hóa kết quả.

Sau khi hiểu những điều này, bạn có thể hỏi, trên thị trường đã có LangChain, LlamaIndex và nhiều framework xuất sắc khác, tại sao vẫn phải "phát minh lại bánh xe"? Câu trả lời là, mặc dù các framework trưởng thành có lợi thế rõ rệt về hiệu quả kỹ thuật, nhưng việc trực tiếp sử dụng các công cụ trừu tượng hóa cao không có lợi cho việc chúng ta hiểu được cơ chế thiết kế phía sau vận hành như thế nào, hay có lợi ích gì. Thứ hai, quá trình này sẽ phơi bày những thách thức kỹ thuật của dự án. Framework đã xử lý giúp chúng ta nhiều vấn đề, ví dụ như parse định dạng output của model, retry khi tool call thất bại, ngăn Agent rơi vào vòng lặp vô hạn, v.v. Tự tay xử lý những vấn đề này là cách trực tiếp nhất để rèn luyện năng lực thiết kế hệ thống. Cuối cùng, cũng là điểm quan trọng nhất, chỉ khi nắm vững nguyên lý thiết kế, bạn mới thực sự chuyển mình từ một "người dùng" của framework thành một "người sáng tạo" ứng dụng Agent. Khi các thành phần tiêu chuẩn không thể đáp ứng được nhu cầu phức tạp của bạn, bạn sẽ có năng lực tùy biến sâu, thậm chí xây dựng một Agent hoàn toàn mới từ con số 0.

## 4.1 Chuẩn bị môi trường và định nghĩa công cụ cơ bản

Trước khi bắt đầu xây dựng, chúng ta cần thiết lập môi trường phát triển và định nghĩa một số thành phần cơ bản trước. Điều này sẽ giúp chúng ta tránh lặp lại công việc khi hiện thực hóa các paradigm khác nhau sau này, tập trung hơn vào logic cốt lõi.

### 4.1.1 Cài đặt thư viện phụ thuộc

Phần thực hành của cuốn sách này sẽ chủ yếu sử dụng ngôn ngữ Python, khuyến nghị sử dụng Python 3.10 trở lên. Trước tiên, hãy đảm bảo bạn đã cài đặt thư viện `openai` dùng để tương tác với Large Language Model, cũng như thư viện `python-dotenv` dùng để quản lý API key của chúng ta một cách an toàn.

Chạy lệnh sau trong terminal của bạn:

```bash
pip install openai python-dotenv
```

### 4.1.2 Cấu hình API Key

Để code của chúng ta mang tính tổng quát hơn, chúng ta sẽ cấu hình thống nhất các thông tin liên quan đến dịch vụ model (model ID, API key, địa chỉ dịch vụ) trong biến môi trường (environment variable).

1. Tạo một file có tên `.env` trong thư mục gốc của project.
2. Thêm nội dung sau vào file này. Bạn có thể trỏ nó đến dịch vụ chính thức của OpenAI, hoặc bất kỳ dịch vụ local/bên thứ ba nào tương thích với OpenAI API, tùy theo nhu cầu của bạn.
3. Nếu thực sự không biết cách lấy, bạn có thể tham khảo [Cấu hình môi trường](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)

```bash
# .env file
LLM_API_KEY="YOUR-API-KEY"
LLM_MODEL_ID="YOUR-MODEL"
LLM_BASE_URL="YOUR-URL"
```

Code của chúng ta sẽ tự động load các cấu hình này từ file.

### 4.1.3 Đóng gói hàm gọi LLM cơ bản

Để cấu trúc code rõ ràng hơn, dễ tái sử dụng hơn, chúng ta hãy định nghĩa một class LLM client chuyên dụng. Class này sẽ đóng gói toàn bộ chi tiết tương tác với dịch vụ model, giúp logic chính của chúng ta có thể tập trung hơn vào việc xây dựng Agent.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv
from typing import List, Dict

# Load biến môi trường từ file .env
load_dotenv()

class HelloAgentsLLM:
    """
    LLM client tùy chỉnh cho cuốn sách "Hello Agents".
    Dùng để gọi bất kỳ dịch vụ nào tương thích với OpenAI interface, và mặc định sử dụng streaming response.
    """
    def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None):
        """
        Khởi tạo client. Ưu tiên sử dụng tham số truyền vào, nếu không được cung cấp thì load từ biến môi trường.
        """
        self.model = model or os.getenv("LLM_MODEL_ID")
        apiKey = apiKey or os.getenv("LLM_API_KEY")
        baseUrl = baseUrl or os.getenv("LLM_BASE_URL")
        timeout = timeout or int(os.getenv("LLM_TIMEOUT", 60))
        
        if not all([self.model, apiKey, baseUrl]):
            raise ValueError("模型ID、API密钥和服务地址必须被提供或在.env文件中定义。")

        self.client = OpenAI(api_key=apiKey, base_url=baseUrl, timeout=timeout)

    def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str:
        """
        调用大语言模型进行思考，并返回其响应。
        """
        print(f"🧠 正在调用 {self.model} 模型...")
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=temperature,
                stream=True,
            )
            
            # 处理流式响应
            print("✅ 大语言模型响应成功:")
            collected_content = []
            for chunk in response:
                if not chunk.choices:
                    continue
                content = chunk.choices[0].delta.content or ""
                print(content, end="", flush=True)
                collected_content.append(content)
            print()  # 在流式输出结束后换行
            return "".join(collected_content)

        except Exception as e:
            print(f"❌ 调用LLM API时发生错误: {e}")
            return None

# --- 客户端使用示例 ---
if __name__ == '__main__':
    try:
        llmClient = HelloAgentsLLM()
        
        exampleMessages = [
            {"role": "system", "content": "You are a helpful assistant that writes Python code."},
            {"role": "user", "content": "写一个快速排序算法"}
        ]
        
        print("--- 调用LLM ---")
        responseText = llmClient.think(exampleMessages)
        if responseText:
            print("\n\n--- 完整模型响应 ---")
            print(responseText)

    except ValueError as e:
        print(e)


>>>
--- 调用LLM ---
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功:
快速排序是一种非常高效的排序算法...
```



## 4.2 ReAct

Sau khi chuẩn bị xong LLM client, chúng ta sẽ xây dựng paradigm Agent đầu tiên, cũng là paradigm kinh điển nhất <strong>ReAct (Reason + Act)</strong>. ReAct được Shunyu Yao đề xuất vào năm 2022<sup>[1]</sup>, tư tưởng cốt lõi của nó là mô phỏng cách con người giải quyết vấn đề, kết hợp một cách tường minh <strong>Suy luận (Reasoning)</strong> và <strong>Hành động (Acting)</strong>, tạo thành một vòng lặp "suy nghĩ - hành động - quan sát".

### 4.2.1 Quy trình làm việc của ReAct

Trước khi ReAct ra đời, các phương pháp chủ đạo có thể chia thành hai loại: một loại là kiểu "thuần suy nghĩ", như <strong>Chain-of-Thought (chuỗi suy luận)</strong>, nó có thể dẫn dắt model thực hiện suy luận logic phức tạp, nhưng không thể tương tác với thế giới bên ngoài, dễ sinh ra ảo giác về sự thật (fact hallucination); loại còn lại là kiểu "thuần hành động", model trực tiếp output hành động cần thực hiện, nhưng thiếu khả năng lập kế hoạch (planning) và sửa lỗi.

Sự khéo léo của ReAct nằm ở chỗ, nó nhận ra rằng <strong>suy nghĩ và hành động bổ trợ lẫn nhau</strong>. Suy nghĩ chỉ đạo hành động, còn kết quả của hành động lại quay ngược lại điều chỉnh suy nghĩ. Vì vậy, paradigm ReAct dẫn dắt model thông qua một kiểu prompt engineering đặc biệt, khiến output ở mỗi bước đều tuân theo một quỹ đạo cố định:

- <strong>Thought (Suy nghĩ):</strong> Đây là "độc thoại nội tâm" của Agent. Nó sẽ phân tích tình huống hiện tại, phân rã nhiệm vụ, lập kế hoạch cho bước tiếp theo, hoặc phản tư về kết quả bước trước.
- <strong>Action (Hành động):</strong> Đây là hành động cụ thể mà Agent quyết định thực hiện, thường là gọi một công cụ bên ngoài (Tool), ví dụ `Search['华为最新款手机']`.
- <strong>Observation (Quan sát):</strong> Đây là kết quả trả về từ công cụ bên ngoài sau khi thực thi `Action`, ví dụ bản tóm tắt kết quả tìm kiếm hoặc giá trị trả về của API.

Agent sẽ liên tục lặp lại vòng lặp <strong>Thought -> Action -> Observation</strong>, thêm kết quả quan sát mới vào lịch sử, tạo thành một context không ngừng tăng trưởng, cho đến khi nó nhận định trong `Thought` rằng đã tìm ra câu trả lời cuối cùng, rồi output kết quả. Quá trình này tạo nên một hiệu ứng cộng hưởng mạnh mẽ: <strong>suy luận khiến hành động có mục đích hơn, còn hành động thì cung cấp căn cứ thực tế cho suy luận.</strong>

Chúng ta có thể biểu diễn quá trình này một cách hình thức hóa, như Hình 4.1. Cụ thể, tại mỗi bước thời gian $t$, chính sách (policy) của Agent (tức Large Language Model $\pi$) sẽ dựa trên câu hỏi ban đầu $q$ và toàn bộ quỹ đạo lịch sử "hành động - quan sát" của các bước trước đó $((a_1,o_1),\dots,(a_{t-1},o_{t-1}))$, để sinh ra suy nghĩ $th_t$ và hành động $a_t$ hiện tại:

$$\left(th_t,a_t\right)=\pi\left(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1})\right)$$

Sau đó, công cụ $T$ trong môi trường sẽ thực thi hành động $a_t$, và trả về một kết quả quan sát mới $o_t$:

$$o_t = T(a_t)$$

Vòng lặp này diễn ra liên tục, thêm cặp $(a_t,o_t)$ mới vào lịch sử, cho đến khi model trong suy nghĩ $th_t$ nhận định rằng nhiệm vụ đã hoàn thành.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-1.png" alt="ReAct范式中的“思考-行动-观察”协同循环" width="90%"/>
  <p>Hình 4.1 Vòng lặp cộng hưởng "suy nghĩ - hành động - quan sát" trong paradigm ReAct</p>
</div>

Cơ chế này đặc biệt phù hợp với các kịch bản sau:

- <strong>Nhiệm vụ cần kiến thức bên ngoài</strong>: như truy vấn thông tin thời gian thực (thời tiết, tin tức, giá cổ phiếu), tìm kiếm kiến thức chuyên ngành, v.v.
- <strong>Nhiệm vụ cần tính toán chính xác</strong>: giao bài toán cho công cụ máy tính (calculator), tránh lỗi tính toán của LLM.
- <strong>Nhiệm vụ cần tương tác với API</strong>: như thao tác cơ sở dữ liệu, gọi API của một dịch vụ nào đó để hoàn thành chức năng cụ thể.

Vì vậy chúng ta sẽ xây dựng một Agent ReAct có khả năng <strong>sử dụng công cụ bên ngoài</strong>, để trả lời một câu hỏi mà Large Language Model chỉ dựa vào kiến thức bản thân không thể trả lời trực tiếp. Ví dụ: "Chiếc điện thoại mới nhất của Huawei là mẫu nào? Điểm bán hàng chủ đạo của nó là gì?" Câu hỏi này đòi hỏi Agent phải hiểu rằng mình cần lên mạng tìm kiếm, gọi tool để tìm kết quả và tổng hợp câu trả lời.

### 4.2.2 Định nghĩa và hiện thực hóa Tool

Nếu nói Large Language Model là bộ não của Agent, thì <strong>Tool (công cụ)</strong> chính là "tay và chân" giúp nó tương tác với thế giới bên ngoài. Để paradigm ReAct có thể thực sự giải quyết vấn đề đã đặt ra, Agent cần có khả năng gọi Tool bên ngoài.

Nhắm vào mục tiêu đã đặt ra trong mục này — trả lời câu hỏi về "điện thoại mới nhất của Huawei", chúng ta cần cung cấp cho Agent một web search tool. Ở đây chúng ta chọn <strong>SerpApi</strong>, nó cung cấp kết quả tìm kiếm Google có cấu trúc thông qua API, có thể trực tiếp trả về "answer box tóm tắt" hoặc thông tin knowledge graph chính xác.

Trước tiên, cần cài đặt thư viện này:

```bash
pip install google-search-results
```

Đồng thời, bạn cần truy cập [trang chủ SerpApi](https://serpapi.com/) để đăng ký một tài khoản miễn phí, lấy API key của bạn, và thêm nó vào file `.env` trong thư mục gốc project của chúng ta:

```bash
# .env file
# ... (保留之前的LLM配置)
SERPAPI_API_KEY="YOUR_SERPAPI_API_KEY"
```

Tiếp theo, chúng ta sẽ định nghĩa và quản lý tool này thông qua code. Chúng ta sẽ thực hiện từng bước: trước tiên hiện thực hóa logic cốt lõi của tool, sau đó xây dựng một tool manager tổng quát.

（1）Hiện thực hóa logic cốt lõi của search tool

Một tool được định nghĩa tốt nên bao gồm ba yếu tố cốt lõi sau:

1. <strong>Tên (Name)</strong>： Một định danh (identifier) ngắn gọn, duy nhất, để Agent gọi trong `Action`, ví dụ `Search`.
2. <strong>Mô tả (Description)</strong>： Một đoạn mô tả bằng ngôn ngữ tự nhiên rõ ràng, giải thích công dụng của tool này. <strong>Đây là phần quan trọng nhất của toàn bộ cơ chế</strong>, vì Large Language Model sẽ dựa vào đoạn mô tả này để phán đoán khi nào nên dùng tool nào.
3. <strong>Logic thực thi (Execution Logic)</strong>： Hàm hoặc phương thức thực sự thực hiện nhiệm vụ.

Tool đầu tiên của chúng ta là hàm `search`, công dụng của nó là nhận một query string, sau đó trả về kết quả tìm kiếm.

```python
from serpapi import SerpApiClient

def search(query: str) -> str:
    """
    一个基于SerpApi的实战网页搜索引擎工具。
    它会智能地解析搜索结果，优先返回直接答案或知识图谱信息。
    """
    print(f"🔍 正在执行 [SerpApi] 网页搜索: {query}")
    try:
        api_key = os.getenv("SERPAPI_API_KEY")
        if not api_key:
            return "错误:SERPAPI_API_KEY 未在 .env 文件中配置。"

        params = {
            "engine": "google",
            "q": query,
            "api_key": api_key,
            "gl": "cn",  # 国家代码
            "hl": "zh-cn", # 语言代码
        }
        
        client = SerpApiClient(params)
        results = client.get_dict()
        
        # 智能解析:优先寻找最直接的答案
        if "answer_box_list" in results:
            return "\n".join(results["answer_box_list"])
        if "answer_box" in results and "answer" in results["answer_box"]:
            return results["answer_box"]["answer"]
        if "knowledge_graph" in results and "description" in results["knowledge_graph"]:
            return results["knowledge_graph"]["description"]
        if "organic_results" in results and results["organic_results"]:
            # 如果没有直接答案，则返回前三个有机结果的摘要
            snippets = [
                f"[{i+1}] {res.get('title', '')}\n{res.get('snippet', '')}"
                for i, res in enumerate(results["organic_results"][:3])
            ]
            return "\n\n".join(snippets)
        
        return f"对不起，没有找到关于 '{query}' 的信息。"

    except Exception as e:
        return f"搜索时发生错误: {e}"
```

Trong đoạn code trên, trước tiên sẽ kiểm tra xem có tồn tại `answer_box` (answer box tóm tắt của Google) hoặc `knowledge_graph` (knowledge graph) hay không, nếu có thì trực tiếp trả về những câu trả lời chính xác nhất này. Nếu không tồn tại, nó mới quay sang phương án thứ hai, trả về tóm tắt của ba kết quả tìm kiếm thông thường đầu tiên. Cách "phân tích thông minh" này có thể cung cấp input chất lượng cao hơn cho LLM.

（2）Xây dựng một bộ thực thi Tool tổng quát

Khi Agent cần sử dụng nhiều loại tool (ví dụ, ngoài tìm kiếm, còn có thể cần tính toán, truy vấn cơ sở dữ liệu, v.v.), chúng ta cần một manager thống nhất để đăng ký và điều phối các tool này. Vì vậy, chúng ta tạo một class `ToolExecutor`.

```python
from typing import Dict, Any

class ToolExecutor:
    """
    一个工具执行器，负责管理和执行工具。
    """
    def __init__(self):
        self.tools: Dict[str, Dict[str, Any]] = {}

    def registerTool(self, name: str, description: str, func: callable):
        """
        向工具箱中注册一个新工具。
        """
        if name in self.tools:
            print(f"警告:工具 '{name}' 已存在，将被覆盖。")
        self.tools[name] = {"description": description, "func": func}
        print(f"工具 '{name}' 已注册。")

    def getTool(self, name: str) -> callable:
        """
        根据名称获取一个工具的执行函数。
        """
        return self.tools.get(name, {}).get("func")

    def getAvailableTools(self) -> str:
        """
        获取所有可用工具的格式化描述字符串。
        """
        return "\n".join([
            f"- {name}: {info['description']}" 
            for name, info in self.tools.items()
        ])

```

(3) Kiểm thử

Bây giờ, chúng ta sẽ đăng ký tool `search` vào `ToolExecutor`, và mô phỏng một lần gọi, để xác minh toàn bộ quy trình hoạt động bình thường hay không.

```python
# --- 工具初始化与使用示例 ---
if __name__ == '__main__':
    # 1. 初始化工具执行器
    toolExecutor = ToolExecutor()

    # 2. 注册我们的实战搜索工具
    search_description = "一个网页搜索引擎。当你需要回答关于时事、事实以及在你的知识库中找不到的信息时，应使用此工具。"
    toolExecutor.registerTool("Search", search_description, search)
    
    # 3. 打印可用的工具
    print("\n--- 可用的工具 ---")
    print(toolExecutor.getAvailableTools())

    # 4. 智能体的Action调用，这次我们问一个实时性的问题
    print("\n--- 执行 Action: Search['英伟达最新的GPU型号是什么'] ---")
    tool_name = "Search"
    tool_input = "英伟达最新的GPU型号是什么"

    tool_function = toolExecutor.getTool(tool_name)
    if tool_function:
        observation = tool_function(tool_input)
        print("--- 观察 (Observation) ---")
        print(observation)
    else:
        print(f"错误:未找到名为 '{tool_name}' 的工具。")
        
>>>
工具 'Search' 已注册。

--- 可用的工具 ---
- Search: 一个网页搜索引擎。当你需要回答关于时事、事实以及在你的知识库中找不到的信息时，应使用此工具。

--- 执行 Action: Search['英伟达最新的GPU型号是什么'] ---
🔍 正在执行 [SerpApi] 网页搜索: 英伟达最新的GPU型号是什么
--- 观察 (Observation) ---
[1] GeForce RTX 50 系列显卡
GeForce RTX™ 50 系列GPU 搭载NVIDIA Blackwell 架构，为游戏玩家和创作者带来全新玩法。RTX 50 系列具备强大的AI 算力，带来升级体验和更逼真的画面。

[2] 比较GeForce 系列最新一代显卡和前代显卡
比较最新一代RTX 30 系列显卡和前代的RTX 20 系列、GTX 10 和900 系列显卡。查看规格、功能、技术支持等内容。

[3] GeForce 显卡| NVIDIA
DRIVE AGX. 强大的车载计算能力，适用于AI 驱动的智能汽车系统 · Clara AGX. 适用于创新型医疗设备和成像的AI 计算. 游戏和创作. GeForce. 探索显卡、游戏解决方案、AI ...
```

Đến đây, chúng ta đã trang bị cho Agent tool `Search` kết nối với internet thực, cung cấp nền tảng vững chắc cho vòng lặp ReAct ở phần sau.



### 4.2.3 Hiện thực hóa code Agent ReAct

Bây giờ, chúng ta sẽ lắp ráp tất cả các thành phần độc lập, LLM client và tool executor lại với nhau, xây dựng một Agent ReAct hoàn chỉnh. Chúng ta sẽ đóng gói logic cốt lõi của nó thông qua một class `ReActAgent`. Để dễ hiểu, chúng ta sẽ chia quá trình hiện thực hóa class này thành các phần quan trọng sau để giảng giải.

（1）Thiết kế System Prompt

Prompt là nền tảng của toàn bộ cơ chế ReAct, nó cung cấp chỉ thị thao tác cho Large Language Model. Chúng ta cần thiết kế tỉ mỉ một template, nó sẽ chèn động các tool khả dụng, câu hỏi của người dùng cũng như lịch sử tương tác của các bước trung gian.

```bash
# ReAct 提示词模板
REACT_PROMPT_TEMPLATE = """
请注意，你是一个有能力调用外部工具的智能助手。

可用工具如下:
{tools}

请严格按照以下格式进行回应:

Thought: 你的思考过程，用于分析问题、拆解任务和规划下一步行动。
Action: 你决定采取的行动，必须是以下格式之一:
- `{{tool_name}}[{{tool_input}}]`:调用一个可用工具。
- `Finish[最终答案]`:当你认为已经获得最终答案时。
- 当你收集到足够的信息，能够回答用户的最终问题时，你必须在Action:字段后使用 Finish[最终答案] 来输出最终答案。

现在，请开始解决以下问题:
Question: {question}
History: {history}
"""
```

Template này định nghĩa quy chuẩn tương tác giữa Agent và LLM:

- <strong>Định nghĩa vai trò (Role)</strong>： "Bạn là một trợ lý thông minh có khả năng gọi công cụ bên ngoài", thiết lập vai trò cho LLM.
- <strong>Danh sách Tool (`{tools}`)</strong>： Thông báo cho LLM biết nó có những "tay chân" khả dụng nào.
- <strong>Quy ước định dạng (`Thought`/`Action`)</strong>： Đây là phần quan trọng nhất, nó bắt buộc output của LLM phải có tính cấu trúc, giúp chúng ta có thể parse chính xác ý định của nó thông qua code.
- <strong>Context động (`{question}`/`{history}`)</strong>： Đưa câu hỏi gốc của người dùng và lịch sử tương tác tích lũy liên tục vào, để LLM đưa ra quyết định dựa trên context hoàn chỉnh.

（2）Hiện thực hóa vòng lặp cốt lõi

Cốt lõi của `ReActAgent` là một vòng lặp, nó liên tục "format prompt -> gọi LLM -> thực thi hành động -> tổng hợp kết quả", cho đến khi nhiệm vụ hoàn thành hoặc đạt đến giới hạn số bước tối đa.

```python
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        运行ReAct智能体来回答一个问题。
        """
        self.history = [] # 每次运行时重置历史记录
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- 第 {current_step} 步 ---")

            # 1. 格式化提示词
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. 调用LLM进行思考
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)
            
            if not response_text:
                print("错误:LLM未能返回有效响应。")
                break

            # ... (后续的解析、执行、整合步骤)

```

Phương thức `run` là điểm vào (entry point) của Agent. Vòng lặp `while` của nó tạo nên thân chính của paradigm ReAct, tham số `max_steps` là một van an toàn quan trọng, ngăn Agent rơi vào vòng lặp vô hạn và cạn kiệt tài nguyên.

（3）Hiện thực hóa Output Parser

Thứ LLM trả về là văn bản thuần (plain text), chúng ta cần trích xuất chính xác `Thought` và `Action` từ đó. Việc này được thực hiện thông qua một vài hàm hỗ trợ parse, chúng thường sử dụng regular expression (regex) để hiện thực hóa.

```python
# (这些方法是 ReActAgent 类的一部分)
    def _parse_output(self, text: str):
        """解析LLM的输出，提取Thought和Action。
        """
        # Thought: 匹配到 Action: 或文本末尾
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: 匹配到文本末尾
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """解析Action字符串，提取工具名称和输入。
        """
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`： Phụ trách tách `Thought` và `Action`, hai phần chính, từ response hoàn chỉnh của LLM.
- `_parse_action`： Phụ trách parse tiếp chuỗi `Action`, ví dụ từ `Search[华为最新手机]` trích xuất ra tên tool `Search` và input của tool `华为最新手机`.

(4) Gọi và thực thi Tool

```python
# (这段逻辑在 run 方法的 while 循环内)
            # 3. 解析LLM的输出
            thought, action = self._parse_output(response_text)
            
            if thought:
                print(f"思考: {thought}")

            if not action:
                print("警告:未能解析出有效的Action，流程终止。")
                break

            # 4. 执行Action
            if action.startswith("Finish"):
                # 如果是Finish指令，提取最终答案并结束
                final_answer = re.match(r"Finish\[(.*)\]", action).group(1)
                print(f"🎉 最终答案: {final_answer}")
                return final_answer
            
            tool_name, tool_input = self._parse_action(action)
            if not tool_name or not tool_input:
                # ... 处理无效Action格式 ...
                continue

            print(f"🎬 行动: {tool_name}[{tool_input}]")
            
            tool_function = self.tool_executor.getTool(tool_name)
            if not tool_function:
                observation = f"错误:未找到名为 '{tool_name}' 的工具。"
            else:
                observation = tool_function(tool_input) # 调用真实工具

```

Đoạn code này là trung tâm thực thi của `Action`. Trước tiên nó kiểm tra xem có phải chỉ thị `Finish` hay không, nếu phải thì quy trình kết thúc. Nếu không phải, nó sẽ lấy hàm tool tương ứng thông qua `tool_executor` và thực thi, nhận về `observation`.

(5) Tổng hợp kết quả quan sát

Bước cuối cùng, cũng là chìa khóa tạo thành vòng kín (closed loop), là thêm `Action` bản thân nó và `Observation` sau khi tool thực thi vào lịch sử, cung cấp context mới cho vòng lặp tiếp theo.

```python
# (这段逻辑紧随工具调用之后，在 while 循环的末尾)
            print(f"👀 观察: {observation}")
            
            # 将本轮的Action和Observation添加到历史记录中
            self.history.append(f"Action: {action}")
            self.history.append(f"Observation: {observation}")

        # 循环结束
        print("已达到最大步数，流程终止。")
        return None
```

Thông qua việc thêm `Observation` vào `self.history`, Agent có thể "nhìn thấy" kết quả của hành động bước trước khi sinh prompt ở vòng tiếp theo, và dựa vào đó để tiến hành suy nghĩ và lập kế hoạch cho vòng mới.

（6）Ví dụ chạy thực tế và phân tích

Kết hợp tất cả các phần trên lại, chúng ta có được class `ReActAgent` hoàn chỉnh. Code chạy ví dụ hoàn chỉnh có thể được tìm thấy trong thư mục `code` của repository code đi kèm cuốn sách này.

Dưới đây là một bản ghi chạy thực tế:

```
工具 'Search' 已注册。

--- 第 1 步 ---
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功:
Thought: 要回答这个问题，我需要查找华为最新发布的手机型号及其主要特点。这些信息可能在我的现有知识库之外，因此需要使用搜索引擎来获取最新数据。
Action: Search[华为最新手机型号及主要卖点]
🤔 思考: 要回答这个问题，我需要查找华为最新发布的手机型号及其主要特点。这些信息可能在我的现有知识库之外，因此需要使用搜索引擎来获取最新数据。
🎬 行动: Search[华为最新手机型号及主要卖点]
🔍 正在执行 [SerpApi] 网页搜索: 华为最新手机型号及主要卖点
👀 观察: [1] 华为手机- 华为官网
智能手机 ; Mate 系列. 非凡旗舰 · HUAWEI Mate XTs. 非凡大师 ; Pura 系列. 先锋影像 · HUAWEI Pura 80 Pro+ ; Pocket 系列. 美学新篇. HUAWEI Pocket 2 ; nova 系列. 专业人像.

[2] 2025年华为手机哪一款性价比高？华为手机推荐与市场分析 ...
现在华为手机最大的卖点只剩下鸿蒙HarmonyOS系统，以及饱受争议的品牌信仰。 这里推荐目前值得入手的几款华为系列手机，根据不同预算自行选择:. 华为目前最受欢迎，也是搭载 ...

[3] 2025年华为新款手机哪个性价比高？10款华为新款手机推荐
选华为主要还是要推荐高端手机，Mate 70和Pura 70系列是最新发布的旗舰机型。 HUAWEI Mate 70. 优点是，拍照配置依旧顶级，全焦段覆盖，适合专业摄影，做工出色，户外抗摔 ...

--- 第 2 步 ---
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功:
Thought: 根据搜索结果，华为最新发布的旗舰机型包括Mate 70和Pura 80 Pro+。为了确定最新型号及其主要卖点，我将重点放在这些信息上。从提供的链接来看，Mate 70系列和Pura 80 Pro+都是近期发布的产品，但具体哪一个是“最新”还需要进一步确认。同时，我可以从这些信息中提取出它们的主要
卖点。
Action: Finish[根据最新信息，华为的最新手机可能是HUAWEI Pura 80 Pro+或HUAWEI Mate 70。其中，HUAWEI Mate 70的主要卖点包括顶级的拍照配置，全焦段覆盖，适合专业摄影，做工出色，并且具有良好的户外抗摔性能。而HUAWEI Pura 80 Pro+则强调了先锋影像技术。]
🤔 思考: 根据搜索结果，华为最新发布的旗舰机型包括Mate 70和Pura 80 Pro+。为了确定最新型号及其主要卖点，我将重点放在这些信息上。从提供的链接来看，Mate 70系列和Pura 80 Pro+都是近期发布的产品，但具体哪一个是“最新”还需要进一步确认。同时，我可以从这些信息中提取出它们的主要 
卖点。
🎉 最终答案: 根据最新信息，华为的最新手机可能是HUAWEI Pura 80 Pro+或HUAWEI Mate 70。其中，HUAWEI Mate 70的主要卖点包括顶级的拍照配置，全焦段覆盖，适合专业摄影，做工出色，并且具有良好的户外抗摔性能。而HUAWEI Pura 80 Pro+则强调了先锋影像技术。
```

Từ output ở trên có thể thấy, Agent đã trình bày rõ ràng chuỗi suy nghĩ của nó: trước tiên nó nhận ra kiến thức bản thân không đủ, cần sử dụng search tool; sau đó, nó suy luận và tổng hợp dựa trên kết quả tìm kiếm, và đưa ra câu trả lời cuối cùng chỉ trong hai bước.

Đáng chú ý là, do kiến thức của model và thông tin trên internet không ngừng được cập nhật, kết quả bạn chạy có thể không hoàn toàn giống với kết quả này. Tính đến ngày 8 tháng 9 năm 2025 khi mục này được biên soạn, HUAWEI Mate 70 và HUAWEI Pura 80 Pro+ được nhắc đến trong kết quả tìm kiếm quả thực là dòng điện thoại flagship mới nhất của Huawei vào thời điểm đó. Điều này cho thấy rõ năng lực mạnh mẽ của paradigm ReAct trong việc xử lý các vấn đề mang tính thời sự (timeliness).

### 4.2.4 Đặc điểm, hạn chế và kỹ thuật debug của ReAct

Thông qua việc tự tay hiện thực hóa một Agent ReAct, chúng ta không chỉ nắm vững quy trình làm việc của nó, mà còn có nhận thức sâu sắc hơn về cơ chế nội tại của nó. Bất kỳ paradigm kỹ thuật nào cũng có điểm sáng và điểm cần cải thiện, mục này sẽ tổng kết về ReAct.

（1）Các đặc điểm chính của ReAct

1. <strong>Tính giải thích cao (High Interpretability)</strong>：Một trong những ưu điểm lớn nhất của ReAct chính là tính minh bạch. Thông qua chuỗi `Thought`, chúng ta có thể nhìn thấy rõ ràng "hành trình tâm lý" của Agent ở mỗi bước — tại sao nó chọn tool này, bước tiếp theo nó định làm gì. Điều này vô cùng quan trọng đối với việc hiểu, tin tưởng và debug hành vi của Agent.
2. <strong>Khả năng lập kế hoạch động và sửa lỗi</strong>：Khác với paradigm sinh ra kế hoạch hoàn chỉnh một lần, ReAct "đi một bước, nhìn một bước". Nó điều chỉnh động `Thought` và `Action` tiếp theo dựa trên `Observation` nhận được từ thế giới bên ngoài ở mỗi bước. Nếu kết quả tìm kiếm của bước trước không lý tưởng, nó có thể sửa từ khóa tìm kiếm ở bước tiếp theo, thử lại.
3. <strong>Khả năng phối hợp Tool</strong>：Paradigm ReAct kết hợp một cách tự nhiên năng lực suy luận của Large Language Model với năng lực thực thi của Tool bên ngoài. LLM phụ trách hoạch định chiến lược (lập kế hoạch và suy luận), Tool phụ trách giải quyết vấn đề cụ thể (tìm kiếm, tính toán), hai bên phối hợp làm việc, phá vỡ những hạn chế cố hữu của một LLM đơn lẻ về mặt tính thời sự của kiến thức, độ chính xác tính toán, v.v.

（2）Những hạn chế cố hữu của ReAct

1. <strong>Phụ thuộc mạnh vào năng lực bản thân của LLM</strong>：Sự thành công hay thất bại của quy trình ReAct phụ thuộc rất lớn vào năng lực tổng hợp của LLM nền tảng. Nếu năng lực suy luận logic, năng lực tuân theo chỉ thị (instruction following) hoặc năng lực output có định dạng của LLM không đủ, thì rất dễ sinh ra kế hoạch sai ở bước `Thought`, hoặc sinh ra chỉ thị không đúng định dạng ở bước `Action`, dẫn đến toàn bộ quy trình bị gián đoạn.
2. <strong>Vấn đề hiệu suất thực thi</strong>：Do tính chất tuần tự từng bước, hoàn thành một nhiệm vụ thường cần gọi LLM nhiều lần. Mỗi lần gọi đều đi kèm với độ trễ mạng (network latency) và chi phí tính toán. Đối với các nhiệm vụ phức tạp cần nhiều bước, vòng lặp "suy nghĩ - hành động" tuần tự này có thể dẫn đến tổng thời gian và chi phí cao.
3. <strong>Tính dễ vỡ (fragility) của prompt</strong>：Toàn bộ cơ chế vận hành ổn định dựa trên một prompt template được thiết kế tỉ mỉ. Bất kỳ thay đổi nhỏ nào trong template, thậm chí chỉ là sự khác biệt về cách dùng từ, cũng có thể ảnh hưởng đến hành vi của LLM. Ngoài ra, không phải model nào cũng có thể tuân theo định dạng đã định trước một cách ổn định liên tục, điều này làm tăng tính bất định trong ứng dụng thực tế.
4. <strong>Có thể rơi vào tối ưu cục bộ (local optimum)</strong>：Mô hình quyết định từng bước đồng nghĩa với việc Agent thiếu một kế hoạch toàn cục, dài hạn. Nó có thể vì `Observation` trước mắt mà chọn một con đường trông có vẻ đúng nhưng xét về lâu dài không phải tối ưu, thậm chí trong một số trường hợp rơi vào vòng lặp "giậm chân tại chỗ".

（3）Kỹ thuật debug

Khi Agent ReAct mà bạn xây dựng có hành vi không như mong đợi, bạn có thể bắt đầu debug từ các khía cạnh sau:

- <strong>Kiểm tra prompt hoàn chỉnh</strong>：Trước mỗi lần gọi LLM, hãy in ra prompt hoàn chỉnh đã được format cuối cùng, bao gồm toàn bộ lịch sử. Đây là cách trực tiếp nhất để truy ngược nguồn gốc quyết định của LLM.
- <strong>Phân tích output gốc</strong>：Khi parse output thất bại (ví dụ, regular expression không khớp được `Action`), nhất định phải in ra văn bản gốc, chưa qua xử lý mà LLM trả về. Điều này giúp bạn phán đoán xem là LLM không tuân theo định dạng, hay logic parse của bạn có vấn đề.
- <strong>Xác minh input và output của Tool</strong>：Kiểm tra xem `tool_input` mà Agent sinh ra có đúng định dạng mà hàm tool mong đợi hay không, đồng thời cũng cần đảm bảo định dạng `observation` mà tool trả về là thứ mà Agent có thể hiểu và xử lý.
- <strong>Điều chỉnh ví dụ trong prompt (Few-shot Prompting)</strong>：Nếu model thường xuyên mắc lỗi, có thể thêm một hoặc hai ví dụ thành công hoàn chỉnh về "Thought-Action-Observation" vào prompt, thông qua ví dụ để dẫn dắt model tuân theo chỉ thị của bạn tốt hơn.
- <strong>Thử các model hoặc tham số khác nhau</strong>：Đổi sang một model có năng lực mạnh hơn, hoặc điều chỉnh tham số `temperature` (thường đặt là 0 để đảm bảo tính xác định của output), đôi khi có thể giải quyết trực tiếp vấn đề.

## 4.3 Plan-and-Solve

Sau khi đã nắm vững paradigm Agent theo kiểu phản ứng (reactive), quyết định từng bước ReAct, tiếp theo chúng ta sẽ khám phá một phương pháp có phong cách hoàn toàn khác nhưng cũng mạnh mẽ không kém, <strong>Plan-and-Solve</strong>. Đúng như tên gọi, paradigm này chia việc xử lý nhiệm vụ thành hai giai đoạn rõ ràng: <strong>lập kế hoạch trước (Plan), thực thi sau (Solve)</strong>.

Nếu nói ReAct giống như một thám tử giàu kinh nghiệm, dựa vào manh mối tại hiện trường (Observation) để suy luận từng bước, liên tục điều chỉnh hướng điều tra của mình; thì Plan-and-Solve lại giống như một kiến trúc sư, trước khi khởi công bắt buộc phải vẽ ra bản thiết kế hoàn chỉnh (Plan), sau đó thi công nghiêm ngặt theo bản thiết kế đó (Solve). Thực tế, nhiều công cụ large model với Agent mode mà chúng ta đang sử dụng hiện nay cũng tích hợp mẫu thiết kế (design pattern) này.

### 4.3.1 Nguyên lý hoạt động của Plan-and-Solve

Plan-and-Solve Prompting được Lei Wang đề xuất vào năm 2023<sup>[2]</sup>. Động cơ cốt lõi của nó là để giải quyết vấn đề Chain-of-Thought dễ bị "lệch hướng" khi xử lý các vấn đề nhiều bước, phức tạp.

Khác với ReAct dung hợp suy nghĩ và hành động ở mỗi bước, Plan-and-Solve tách toàn bộ quy trình thành hai giai đoạn cốt lõi, như Hình 4.2:

1. <strong>Giai đoạn lập kế hoạch (Planning Phase)</strong>： Trước tiên, Agent sẽ nhận câu hỏi hoàn chỉnh của người dùng. Nhiệm vụ đầu tiên của nó không phải là trực tiếp đi giải quyết vấn đề hay gọi tool, mà là <strong>phân rã vấn đề, và lập ra một kế hoạch hành động rõ ràng, theo từng bước</strong>. Bản thân kế hoạch này là sản phẩm của một lần gọi Large Language Model.
2. <strong>Giai đoạn thực thi (Solving Phase)</strong>： Sau khi có được kế hoạch hoàn chỉnh, Agent bước vào giai đoạn thực thi. Nó sẽ <strong>thực thi lần lượt, nghiêm ngặt theo các bước trong kế hoạch</strong>. Mỗi bước thực thi có thể là một lần gọi LLM độc lập, hoặc là xử lý kết quả của bước trước, cho đến khi tất cả các bước trong kế hoạch hoàn thành, cuối cùng đưa ra câu trả lời.

Chiến lược "nghĩ trước làm sau" này giúp Agent duy trì tính nhất quán mục tiêu cao hơn khi xử lý các nhiệm vụ phức tạp cần lập kế hoạch dài hạn, tránh lạc hướng trong các bước trung gian.

Chúng ta có thể biểu diễn hình thức hóa quá trình hai giai đoạn này. Trước tiên, model lập kế hoạch $\pi_{\text{plan}}$ dựa trên câu hỏi gốc $q$ để sinh ra một kế hoạch $P = (p_1, p_2, \dots, p_n)$ gồm $n$ bước:

$$
P = \pi_{\text{plan}}(q)
$$

Sau đó, trong giai đoạn thực thi, model thực thi $\pi_{\text{solve}}$ sẽ hoàn thành lần lượt các bước trong kế hoạch. Đối với bước thứ $i$, giải pháp $s_i$ của nó được sinh ra sẽ đồng thời phụ thuộc vào câu hỏi gốc $q$, kế hoạch hoàn chỉnh $P$ cũng như kết quả thực thi của tất cả các bước trước đó $(s_1, \dots, s_{i-1})$：

$$
s_i = \pi_{\text{solve}}(q, P, (s_1, \dots, s_{i-1}))
$$

Câu trả lời cuối cùng chính là kết quả thực thi của bước cuối cùng $s_n$.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-2.png" alt="Plan-and-Solve范式的两阶段工作流" width="90%"/>
  <p>Hình 4.2 Quy trình làm việc hai giai đoạn của paradigm Plan-and-Solve</p>
</div>

Plan-and-Solve đặc biệt phù hợp với những nhiệm vụ phức tạp có tính cấu trúc mạnh, có thể được phân rã rõ ràng, ví dụ:

- <strong>Bài toán ứng dụng nhiều bước</strong>：Cần liệt kê các bước tính toán trước, rồi giải quyết từng bước.
- <strong>Viết báo cáo cần tích hợp nhiều nguồn thông tin</strong>：Cần lập kế hoạch cấu trúc báo cáo trước (mở đầu, nguồn dữ liệu A, nguồn dữ liệu B, tổng kết), rồi điền nội dung từng phần.
- <strong>Nhiệm vụ sinh code</strong>：Cần phác thảo cấu trúc hàm, class và module trước, rồi hiện thực hóa từng phần.

### 4.3.2 Giai đoạn lập kế hoạch

Để làm nổi bật ưu thế của paradigm Plan-and-Solve trong các nhiệm vụ suy luận có cấu trúc, chúng ta sẽ không sử dụng tool, mà thông qua thiết kế prompt, hoàn thành một nhiệm vụ suy luận.

Đặc điểm của loại nhiệm vụ này là, câu trả lời không thể có được chỉ bằng một lần truy vấn hay tính toán, bắt buộc phải phân rã vấn đề thành một chuỗi các bước con liên kết logic trước, sau đó giải quyết lần lượt theo thứ tự. Điều này vừa khéo phát huy năng lực cốt lõi "lập kế hoạch trước, thực thi sau" của Plan-and-Solve.

<strong>Vấn đề mục tiêu của chúng ta là:</strong>"Một cửa hàng trái cây bán được 15 quả táo vào thứ Hai. Số lượng táo bán được vào thứ Ba gấp đôi thứ Hai. Số lượng bán được vào thứ Tư ít hơn thứ Ba 5 quả. Hỏi tổng cộng ba ngày này bán được bao nhiêu quả táo?"

Vấn đề này không quá khó đối với Large Language Model, nhưng nó chứa một chuỗi logic rõ ràng để tham khảo. Trong một số bài toán logic thực tế, nếu large model không thể suy luận ra câu trả lời chính xác với chất lượng cao, có thể tham khảo mẫu thiết kế này để thiết kế Agent của riêng mình hoàn thành nhiệm vụ. Agent cần:

1. <strong>Giai đoạn lập kế hoạch</strong>：Trước tiên, phân rã vấn đề thành ba bước tính toán độc lập (tính doanh số thứ Ba, tính doanh số thứ Tư, tính tổng doanh số).
2. <strong>Giai đoạn thực thi</strong>：Sau đó, thực hiện tính toán từng bước nghiêm ngặt theo kế hoạch, và lấy kết quả của mỗi bước làm input cho bước tiếp theo, cuối cùng tính ra tổng số.

Mục tiêu của giai đoạn lập kế hoạch là để Large Language Model nhận câu hỏi gốc, và output một kế hoạch hành động rõ ràng, theo từng bước. Kế hoạch này phải có tính cấu trúc, để code của chúng ta có thể dễ dàng parse và thực thi lần lượt. Vì vậy, prompt chúng ta thiết kế cần thông báo rõ ràng cho model biết vai trò và nhiệm vụ của nó, đồng thời đưa ra một ví dụ về định dạng output.

````python
PLANNER_PROMPT_TEMPLATE = """
你是一个顶级的AI规划专家。你的任务是将用户提出的复杂问题分解成一个由多个简单步骤组成的行动计划。
请确保计划中的每个步骤都是一个独立的、可执行的子任务，并且严格按照逻辑顺序排列。
你的输出必须是一个Python列表，其中每个元素都是一个描述子任务的字符串。

问题: {question}

请严格按照以下格式输出你的计划,```python与```作为前后缀是必要的:
```python
["步骤1", "步骤2", "步骤3", ...]
```
"""
````

Prompt này đảm bảo chất lượng và tính ổn định của output thông qua các điểm sau:
- <strong>Thiết lập vai trò (Role setting)</strong>： "Chuyên gia lập kế hoạch AI hàng đầu", kích hoạt năng lực chuyên môn của model.
- <strong>Mô tả nhiệm vụ (Task description)</strong>： Định nghĩa rõ ràng mục tiêu "phân rã vấn đề".
- <strong>Ràng buộc định dạng (Format constraint)</strong>： Bắt buộc output dưới dạng chuỗi theo định dạng Python list, điều này đơn giản hóa rất nhiều công việc parse của code sau này, khiến nó ổn định và đáng tin cậy hơn so với việc parse ngôn ngữ tự nhiên.

Tiếp theo, chúng ta sẽ đóng gói logic prompt này thành một class `Planner`, class này cũng chính là bộ lập kế hoạch (planner) của chúng ta.

```python
# 假定 llm_client.py 中的 HelloAgentsLLM 类已经定义好
# from llm_client import HelloAgentsLLM

class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        根据用户问题生成一个行动计划。
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)
        
        # 为了生成计划，我们构建一个简单的消息列表
        messages = [{"role": "user", "content": prompt}]
        
        print("--- 正在生成计划 ---")
        # 使用流式输出来获取完整的计划
        response_text = self.llm_client.think(messages=messages) or ""
        
        print(f"✅ 计划已生成:\n{response_text}")
        
        # 解析LLM输出的列表字符串
        try:
            # 找到```python和```之间的内容
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # 使用ast.literal_eval来安全地执行字符串，将其转换为Python列表
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ 解析计划时出错: {e}")
            print(f"原始响应: {response_text}")
            return []
        except Exception as e:
            print(f"❌ 解析计划时发生未知错误: {e}")
            return []
```

### 4.3.3 Executor và quản lý trạng thái (state management)

Sau khi bộ lập kế hoạch (`Planner`) đã sinh ra bản thiết kế hành động rõ ràng, chúng ta cần một bộ thực thi (`Executor`) để hoàn thành lần lượt các nhiệm vụ trong kế hoạch. Executor không chỉ phụ trách gọi Large Language Model để giải quyết từng vấn đề con, mà còn gánh vác một vai trò vô cùng quan trọng: <strong>quản lý trạng thái (State Management)</strong>. Nó phải ghi lại kết quả thực thi của mỗi bước, và cung cấp nó làm context cho các bước tiếp theo, đảm bảo thông tin lưu thông thông suốt trong toàn bộ chuỗi nhiệm vụ.

Prompt của Executor khác với Planner. Mục tiêu của nó không phải là phân rã vấn đề, mà là <strong>tập trung giải quyết bước hiện tại này, dựa trên context đã có</strong>. Vì vậy, prompt cần bao gồm các thông tin quan trọng sau:

- <strong>Câu hỏi gốc</strong>： Đảm bảo model luôn hiểu mục tiêu cuối cùng.
- <strong>Kế hoạch hoàn chỉnh</strong>： Để model hiểu vị trí của bước hiện tại trong toàn bộ nhiệm vụ.
- <strong>Các bước lịch sử và kết quả</strong>： Cung cấp công việc đã hoàn thành cho đến nay, làm input trực tiếp cho bước hiện tại.
- <strong>Bước hiện tại</strong>： Chỉ rõ ràng cho model biết bây giờ cần giải quyết nhiệm vụ cụ thể nào.

```python
EXECUTOR_PROMPT_TEMPLATE = """
你是一位顶级的AI执行专家。你的任务是严格按照给定的计划，一步步地解决问题。
你将收到原始问题、完整的计划、以及到目前为止已经完成的步骤和结果。
请你专注于解决“当前步骤”，并仅输出该步骤的最终答案，不要输出任何额外的解释或对话。

# 原始问题:
{question}

# 完整计划:
{plan}

# 历史步骤与结果:
{history}

# 当前步骤:
{current_step}

请仅输出针对“当前步骤”的回答:
"""
```

Chúng ta sẽ đóng gói logic thực thi vào class `Executor`. Class này sẽ lặp qua kế hoạch, gọi LLM, và duy trì một lịch sử (trạng thái).

```python
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        根据计划，逐步执行并解决问题。
        """
        history = "" # 用于存储历史步骤和结果的字符串
        
        print("\n--- 正在执行计划 ---")
        
        for i, step in enumerate(plan):
            print(f"\n-> 正在执行步骤 {i+1}/{len(plan)}: {step}")
            
            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "无", # 如果是第一步，则历史为空
                current_step=step
            )
            
            messages = [{"role": "user", "content": prompt}]
            
            response_text = self.llm_client.think(messages=messages) or ""
            
            # 更新历史记录，为下一步做准备
            history += f"步骤 {i+1}: {step}\n结果: {response_text}\n\n"
            
            print(f"✅ 步骤 {i+1} 已完成，结果: {response_text}")

        # 循环结束后，最后一步的响应就是最终答案
        final_answer = response_text
        return final_answer
```

Bây giờ chúng ta đã xây dựng xong `Planner` phụ trách "lập kế hoạch" và `Executor` phụ trách "thực thi". Bước cuối cùng là tích hợp hai thành phần này vào một Agent thống nhất `PlanAndSolveAgent`, và trao cho nó năng lực giải quyết vấn đề hoàn chỉnh. Chúng ta sẽ tạo một class chính `PlanAndSolveAgent`, trách nhiệm của nó rất rõ ràng: nhận một LLM client, khởi tạo planner và executor nội bộ, và cung cấp một phương thức `run` đơn giản để khởi động toàn bộ quy trình.

```python
class PlanAndSolveAgent:
    def __init__(self, llm_client):
        """
        初始化智能体，同时创建规划器和执行器实例。
        """
        self.llm_client = llm_client
        self.planner = Planner(self.llm_client)
        self.executor = Executor(self.llm_client)

    def run(self, question: str):
        """
        运行智能体的完整流程:先规划，后执行。
        """
        print(f"\n--- 开始处理问题 ---\n问题: {question}")
        
        # 1. 调用规划器生成计划
        plan = self.planner.plan(question)
        
        # 检查计划是否成功生成
        if not plan:
            print("\n--- 任务终止 --- \n无法生成有效的行动计划。")
            return

        # 2. 调用执行器执行计划
        final_answer = self.executor.execute(question, plan)
        
        print(f"\n--- 任务完成 ---\n最终答案: {final_answer}")
```

Thiết kế class `PlanAndSolveAgent` này thể hiện nguyên tắc "composition over inheritance" (ưu tiên kết hợp hơn kế thừa). Bản thân nó không chứa logic phức tạp, mà đóng vai trò là một người điều phối (Orchestrator), gọi các thành phần nội bộ của nó một cách rõ ràng để hoàn thành nhiệm vụ.

### 4.3.4 Ví dụ chạy thực tế và phân tích

Code hoàn chỉnh cũng tham khảo thư mục `code` trong repository code đi kèm cuốn sách này, ở đây chỉ trình diễn kết quả cuối cùng.

````bash
--- 开始处理问题 ---
问题: 一个水果店周一卖出了15个苹果。周二卖出的苹果数量是周一的两倍。周三卖出的数量比周二少了5个。请问这三天总共卖出了多少个苹果？
--- 正在生成计划 ---
🧠 正在调用 xxxx 模型...
✅ 大语言模型响应成功:
```python
["计算周一卖出的苹果数量： 15个", "计算周二卖出的苹果数量： 周一数量 × 2 = 15 × 2 = 30个", "计算周三卖出的苹果数量： 周二数量 - 5 = 30 - 5 = 25个", "计算三天总销量： 周一 + 周二 + 周三 = 15 + 30 + 25 = 70个"]
```
✅ 计划已生成:
```python
["计算周一卖出的苹果数量： 15个", "计算周二卖出的苹果数量： 周一数量 × 2 = 15 × 2 = 30个", "计算周三卖出的苹果数量： 周二数量 - 5 = 30 - 5 = 25个", "计算三天总销量： 周一 + 周二 + 周三 = 15 + 30 + 25 = 70个"]
```

--- 正在执行计划 ---

-> 正在执行步骤 1/4: 计算周一卖出的苹果数量: 15个
🧠 正在调用 xxxx 模型...
✅ 大语言模型响应成功:
15
✅ 步骤 1 已完成，结果: 15

-> 正在执行步骤 2/4: 计算周二卖出的苹果数量: 周一数量 × 2 = 15 × 2 = 30个
🧠 正在调用 xxxx 模型...
✅ 大语言模型响应成功:
30
✅ 步骤 2 已完成，结果: 30

-> 正在执行步骤 3/4: 计算周三卖出的苹果数量: 周二数量 - 5 = 30 - 5 = 25个
🧠 正在调用 xxxx 模型...
✅ 大语言模型响应成功:
25
✅ 步骤 3 已完成，结果: 25

-> 正在执行步骤 4/4: 计算三天总销量: 周一 + 周二 + 周三 = 15 + 30 + 25 = 70个
🧠 正在调用 xxxx 模型...
✅ 大语言模型响应成功:
70
✅ 步骤 4 已完成，结果: 70

--- 任务完成 ---
最终答案: 70
````

Từ log output ở trên, chúng ta có thể thấy rõ ràng quy trình làm việc của paradigm Plan-and-Solve:

1.  <strong>Giai đoạn lập kế hoạch</strong>： Agent trước tiên gọi `Planner`, phân rã thành công bài toán ứng dụng phức tạp thành một Python list gồm bốn bước logic. Kế hoạch có cấu trúc này đặt nền tảng cho việc thực thi sau đó.
2.  <strong>Giai đoạn thực thi</strong>： `Executor` thực thi lần lượt từng bước, nghiêm ngặt theo kế hoạch đã sinh ra. Ở mỗi bước, nó đều lấy kết quả lịch sử làm context, đảm bảo thông tin được truyền đạt chính xác (ví dụ, bước 2 sử dụng đúng kết quả "15" của bước 1, bước 3 cũng sử dụng đúng kết quả "30" của bước 2).
3.  <strong>Kết quả</strong>：Toàn bộ quá trình logic rõ ràng, các bước minh bạch, cuối cùng Agent tính ra chính xác đáp án "70 quả".

## 4.4 Reflection

Trong các paradigm ReAct và Plan-and-Solve mà chúng ta đã hiện thực hóa, một khi Agent hoàn thành nhiệm vụ, quy trình làm việc của nó coi như kết thúc. Tuy nhiên, câu trả lời ban đầu mà chúng sinh ra, dù là quỹ đạo hành động hay kết quả cuối cùng, đều có thể tồn tại sai sót hoặc chỗ cần cải thiện. Tư tưởng cốt lõi của cơ chế Reflection, chính là đưa vào Agent một <strong>vòng lặp tự sửa chữa sau khi thực hiện (post-hoc)</strong>, giúp nó có thể, giống như con người, xem xét lại công việc của mình, phát hiện thiếu sót, và tiến hành tối ưu hóa lặp lại (iteration).

### 4.4.1 Tư tưởng cốt lõi của cơ chế Reflection

Cơ chế Reflection lấy cảm hứng từ quá trình học tập của con người: chúng ta hoàn thành bản nháp rồi sẽ rà soát lại, giải xong bài toán rồi sẽ kiểm tra lại. Tư tưởng này được thể hiện trong nhiều nghiên cứu, ví dụ framework Reflexion do Shinn, Noah đề xuất năm 2023<sup>[3]</sup>. Quy trình làm việc cốt lõi của nó có thể tóm gọn thành một vòng lặp ba bước đơn giản: <strong>Thực thi -> Phản tư -> Tối ưu hóa</strong>.

1. <strong>Thực thi (Execution)</strong>：Trước tiên, Agent sử dụng các phương pháp mà chúng ta đã quen thuộc (như ReAct hoặc Plan-and-Solve) để cố gắng hoàn thành nhiệm vụ, sinh ra một giải pháp hoặc quỹ đạo hành động sơ bộ. Đây có thể coi là "bản nháp".
2. <strong>Phản tư (Reflection)</strong>：Tiếp theo, Agent bước vào giai đoạn phản tư. Nó sẽ gọi một instance Large Language Model độc lập, hoặc mang prompt đặc biệt, để đóng vai một "người đánh giá" (reviewer). "Người đánh giá" này sẽ xem xét "bản nháp" được sinh ra ở bước một, và đánh giá từ nhiều khía cạnh, ví dụ:
   - <strong>Lỗi sự thật (Factual error)</strong>：Có tồn tại nội dung trái ngược với常识 (common sense) hoặc sự thật đã biết không?
   - <strong>Lỗ hổng logic (Logical gap)</strong>：Quá trình suy luận có chỗ nào không mạch lạc hoặc mâu thuẫn không?
   - <strong>Vấn đề hiệu quả (Efficiency issue)</strong>：Có con đường trực tiếp hơn, ngắn gọn hơn để hoàn thành nhiệm vụ không?
   - <strong>Thông tin bị bỏ sót (Missing information)</strong>：Có bỏ qua một số ràng buộc hoặc khía cạnh quan trọng nào của vấn đề không？ Dựa trên đánh giá, nó sẽ sinh ra một đoạn <strong>phản hồi (Feedback)</strong> có cấu trúc, chỉ ra vấn đề cụ thể nằm ở đâu và đề xuất cải thiện.
3. <strong>Tối ưu hóa (Refinement)</strong>：Cuối cùng, Agent lấy "bản nháp" và "phản hồi" làm context mới, gọi lại Large Language Model, yêu cầu nó sửa chữa bản nháp dựa trên nội dung phản hồi, sinh ra một "bản chỉnh sửa" hoàn thiện hơn.

Như Hình 4.3, vòng lặp này có thể lặp lại nhiều lần, cho đến khi giai đoạn phản tư không còn phát hiện vấn đề mới, hoặc đạt đến giới hạn số lần lặp đã định trước. Chúng ta có thể biểu diễn hình thức hóa quá trình tối ưu hóa lặp lại này. Giả sử $O_i$ là output được sinh ra ở lần lặp thứ $i$ ($O_0$ là output ban đầu), model phản tư $\pi_{\text{reflect}}$ sẽ sinh ra phản hồi $F_i$ đối với $O_i$:
$$
F_i = \pi_{\text{reflect}}(\text{Task}, O_i)
$$
Sau đó, model tối ưu hóa $\pi_{\text{refine}}$ sẽ kết hợp nhiệm vụ gốc, output phiên bản trước đó cũng như phản hồi, sinh ra output phiên bản mới $O_{i+1}$:
$$
O_{i+1} = \pi_{\text{refine}}(\text{Task}, O_i, F_i)
$$



<div align="center">
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-3.png" alt="Reflection机制中的“执行-反思-优化”迭代循环" width="70%"/>
<p>Hình 4.3 Vòng lặp lặp lại "thực thi - phản tư - tối ưu hóa" trong cơ chế Reflection</p>
</div>



So với hai paradigm trước, giá trị của Reflection nằm ở:

- Nó cung cấp cho Agent một vòng lặp tự sửa lỗi nội bộ (internal error-correction loop), khiến nó không còn hoàn toàn phụ thuộc vào phản hồi từ tool bên ngoài (Observation của ReAct), từ đó có thể sửa chữa các lỗi logic và chiến lược ở tầng cao hơn.
- Nó biến việc thực thi nhiệm vụ một lần duy nhất, thành một quá trình tối ưu hóa liên tục, nâng cao đáng kể tỷ lệ thành công và chất lượng câu trả lời cuối cùng của các nhiệm vụ phức tạp.
- Nó xây dựng cho Agent một <strong>"bộ nhớ ngắn hạn" (short-term memory)</strong> tạm thời. Toàn bộ quỹ đạo "thực thi - phản tư - tối ưu hóa" tạo thành một bản ghi kinh nghiệm quý giá, Agent không chỉ biết câu trả lời cuối cùng, mà còn nhớ cách nó đã lặp đi lặp lại từ bản nháp có khuyết điểm đến phiên bản cuối cùng như thế nào. Hơn nữa, hệ thống bộ nhớ này còn có thể là <strong>đa phương thức (multimodal)</strong>, cho phép Agent phản tư và sửa chữa các output ngoài văn bản (như code, hình ảnh, v.v.), đặt nền móng cho việc xây dựng các Agent đa phương thức mạnh mẽ hơn.

### 4.4.2 Thiết lập bài toán và thiết kế module bộ nhớ

Để thể hiện cơ chế Reflection trong thực hành, chúng ta sẽ đưa vào cơ chế quản lý bộ nhớ (memory management), vì reflection thường tương ứng với việc lưu trữ và trích xuất thông tin, nếu context đủ dài, muốn để "người đánh giá" trực tiếp lấy toàn bộ thông tin rồi tiến hành phản tư thường sẽ truyền vào rất nhiều thông tin dư thừa. Bước thực hành này chúng ta chủ yếu hoàn thành <strong>sinh code và tối ưu hóa lặp lại</strong>.

Nhiệm vụ mục tiêu của bước này là: "Viết một hàm Python, tìm tất cả các số nguyên tố (prime numbers) từ 1 đến n."

Nhiệm vụ này là kịch bản tuyệt vời để kiểm chứng cơ chế Reflection:

1. <strong>Tồn tại con đường tối ưu rõ ràng</strong>：Code mà Large Language Model sinh ra lần đầu rất có thể là một cài đặt đệ quy đơn giản nhưng hiệu suất thấp.
2. <strong>Điểm phản tư rõ ràng</strong>：Có thể phát hiện qua phản tư rằng nó có vấn đề "độ phức tạp thời gian quá cao" hoặc "tồn tại tính toán lặp lại".
3. <strong>Hướng tối ưu rõ ràng</strong>：Có thể dựa vào phản hồi, tối ưu hóa nó thành phiên bản lặp (iterative) hiệu quả hơn hoặc phiên bản sử dụng memoization.

Cốt lõi của Reflection nằm ở sự lặp lại (iteration), còn tiền đề của sự lặp lại là có thể nhớ được những lần thử trước đó và phản hồi đã nhận được. Vì vậy, một module "bộ nhớ ngắn hạn" là thứ bắt buộc phải có để hiện thực hóa paradigm này. Module bộ nhớ này sẽ phụ trách lưu trữ quỹ đạo hoàn chỉnh của mỗi vòng lặp "thực thi - phản tư".

```python
from typing import List, Dict, Any, Optional

class Memory:
    """
    一个简单的短期记忆模块，用于存储智能体的行动与反思轨迹。
    """

    def __init__(self):
        """
        初始化一个空列表来存储所有记录。
        """
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        """
        向记忆中添加一条新记录。

        参数:
        - record_type (str): 记录的类型 ('execution' 或 'reflection')。
        - content (str): 记录的具体内容 (例如，生成的代码或反思的反馈)。
        """
        record = {"type": record_type, "content": content}
        self.records.append(record)
        print(f"📝 记忆已更新，新增一条 '{record_type}' 记录。")

    def get_trajectory(self) -> str:
        """
        将所有记忆记录格式化为一个连贯的字符串文本，用于构建提示词。
        """
        trajectory_parts = []
        for record in self.records:
            if record['type'] == 'execution':
                trajectory_parts.append(f"--- 上一轮尝试 (代码) ---\n{record['content']}")
            elif record['type'] == 'reflection':
                trajectory_parts.append(f"--- 评审员反馈 ---\n{record['content']}")
        
        return "\n\n".join(trajectory_parts)

    def get_last_execution(self) -> Optional[str]:
        """
        获取最近一次的执行结果 (例如，最新生成的代码)。
        如果不存在，则返回 None。
        """
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return None
```

Thiết kế class `Memory` này khá đơn giản, phần chính của nó như sau:

- Sử dụng một list `records` để lưu trữ tuần tự mỗi lần hành động và phản tư.
- Phương thức `add_record` phụ trách thêm mục mới vào bộ nhớ.
- Phương thức `get_trajectory` là cốt lõi, nó "serialize" quỹ đạo bộ nhớ thành một đoạn văn bản, có thể trực tiếp chèn vào prompt sau đó, cung cấp context hoàn chỉnh cho việc phản tư và tối ưu hóa của model.
- `get_last_execution` giúp chúng ta thuận tiện lấy "bản nháp" mới nhất để phản tư.



### 4.4.3 Hiện thực hóa code Agent Reflection

Với module `Memory` làm nền tảng, bây giờ chúng ta có thể bắt tay vào xây dựng logic cốt lõi của `ReflectionAgent`. Toàn bộ quy trình làm việc của Agent sẽ xoay quanh vòng lặp "thực thi - phản tư - tối ưu hóa" đã thảo luận trước đó, và thông qua các prompt được thiết kế tỉ mỉ để dẫn dắt Large Language Model đóng các vai trò khác nhau.

（1）Thiết kế prompt

Khác với các paradigm trước, cơ chế Reflection cần nhiều prompt với vai trò khác nhau phối hợp làm việc.

1. <strong>Prompt thực thi ban đầu (Execution Prompt)</strong> ：Đây là prompt cho lần thử giải quyết vấn đề đầu tiên của Agent, nội dung tương đối trực tiếp, chỉ yêu cầu model hoàn thành nhiệm vụ được chỉ định.

```bash
INITIAL_PROMPT_TEMPLATE = """
你是一位资深的Python程序员。请根据以下要求，编写一个Python函数。
你的代码必须包含完整的函数签名、文档字符串，并遵循PEP 8编码规范。

要求: {task}

请直接输出代码，不要包含任何额外的解释。
"""
```

2. <strong>Prompt phản tư (Reflection Prompt)</strong> ：Prompt này chính là linh hồn của cơ chế Reflection. Nó chỉ thị cho model đóng vai "người đánh giá code" (code reviewer), tiến hành phân tích phê phán đối với code được sinh ra ở vòng trước, và cung cấp phản hồi cụ thể, có thể hành động được.

````bash
REFLECT_PROMPT_TEMPLATE = """
你是一位极其严格的代码评审专家和资深算法工程师，对代码的性能有极致的要求。
你的任务是审查以下Python代码，并专注于找出其在<strong>算法效率</strong>上的主要瓶颈。

# 原始任务:
{task}

# 待审查的代码:
```python
{code}
```

请分析该代码的时间复杂度，并思考是否存在一种<strong>算法上更优</strong>的解决方案来显著提升性能。
如果存在，请清晰地指出当前算法的不足，并提出具体的、可行的改进算法建议（例如，使用筛法替代试除法）。
如果代码在算法层面已经达到最优，才能回答“无需改进”。

请直接输出你的反馈，不要包含任何额外的解释。
"""
````

3. <strong>Prompt tối ưu hóa (Refinement Prompt)</strong> ：Sau khi nhận được phản hồi, prompt này sẽ dẫn dắt model sửa chữa và tối ưu hóa code gốc dựa trên nội dung phản hồi.

````bash

REFINE_PROMPT_TEMPLATE = """
你是一位资深的Python程序员。你正在根据一位代码评审专家的反馈来优化你的代码。

# 原始任务:
{task}

# 你上一轮尝试的代码:
{last_code_attempt}
评审员的反馈：
{feedback}

请根据评审员的反馈，生成一个优化后的新版本代码。
你的代码必须包含完整的函数签名、文档字符串，并遵循PEP 8编码规范。
请直接输出优化后的代码，不要包含任何额外的解释。
"""
````

（2）Đóng gói và hiện thực hóa Agent

Bây giờ, chúng ta sẽ tích hợp bộ prompt này cùng module `Memory` vào class `ReflectionAgent`.

```python
# 假设 llm_client.py 和 memory.py 已定义
# from llm_client import HelloAgentsLLM
# from memory import Memory

class ReflectionAgent:
    def __init__(self, llm_client, max_iterations=3):
        self.llm_client = llm_client
        self.memory = Memory()
        self.max_iterations = max_iterations

    def run(self, task: str):
        print(f"\n--- 开始处理任务 ---\n任务: {task}")

        # --- 1. 初始执行 ---
        print("\n--- 正在进行初始尝试 ---")
        initial_prompt = INITIAL_PROMPT_TEMPLATE.format(task=task)
        initial_code = self._get_llm_response(initial_prompt)
        self.memory.add_record("execution", initial_code)

        # --- 2. 迭代循环:反思与优化 ---
        for i in range(self.max_iterations):
            print(f"\n--- 第 {i+1}/{self.max_iterations} 轮迭代 ---")

            # a. 反思
            print("\n-> 正在进行反思...")
            last_code = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT_TEMPLATE.format(task=task, code=last_code)
            feedback = self._get_llm_response(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            # b. 检查是否需要停止
            if "无需改进" in feedback:
                print("\n✅ 反思认为代码已无需改进，任务完成。")
                break

            # c. 优化
            print("\n-> 正在进行优化...")
            refine_prompt = REFINE_PROMPT_TEMPLATE.format(
                task=task,
                last_code_attempt=last_code,
                feedback=feedback
            )
            refined_code = self._get_llm_response(refine_prompt)
            self.memory.add_record("execution", refined_code)
        
        final_code = self.memory.get_last_execution()
        print(f"\n--- 任务完成 ---\n最终生成的代码:\n```python\n{final_code}\n```")
        return final_code

    def _get_llm_response(self, prompt: str) -> str:
        """一个辅助方法，用于调用LLM并获取完整的流式响应。"""
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm_client.think(messages=messages) or ""
        return response_text

```

### 4.4.4 Ví dụ chạy thực tế và phân tích

Code hoàn chỉnh cũng tham khảo thư mục `code` trong repository code đi kèm cuốn sách này, ở đây cung cấp một ví dụ output.

````python
--- 开始处理任务 ---
任务： 编写一个Python函数，找出1到n之间所有的素数 (prime numbers)。

--- 正在进行初始尝试 ---
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功：
```python
def find_primes(n):
    ...
    return primes
```
📝 记忆已更新，新增一条 'execution' 记录。   

--- 第 1/2 轮迭代 ---

-> 正在进行反思...
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功：
当前代码的时间复杂度为O(n * sqrt(n))。虽然对于较小的n值，这种实现是可以接受的，但当n非常大时，性能会显著下降。主要瓶颈在于每个数都需要进行试除法检查，这导致了较高的时间开销。

建议使用埃拉托斯特尼筛法（Sieve of Eratosthenes），该算法的时间复杂度为O(n log(log n))，能够显著提高查找素数的效率。

改进后的代码如下：
```python
def find_primes(n):
    ...
    return primes
```
📝 记忆已更新，新增一条 'reflection' 记录。

-> 正在进行优化...
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功：
```python
def find_primes(n):
    ...
    return primes
```
📝 记忆已更新，新增一条 'execution' 记录。

--- 第 2/2 轮迭代 ---

-> 正在进行反思...
🧠 正在调用 xxxxxx 模型...
✅ 大语言模型响应成功：
当前代码使用了Eratosthenes筛法，时间复杂度为O(n log log n)，空间复杂度为O(n)。此算法在寻找1到n之间的所有素数时已经非常高效，通常情况下无需进一步优化。但在某些特定场景下，可以考虑以下改进：

1. <strong>分段筛法（Segmented Sieve）</strong>：适用于n非常大但内存有限的情况。将区间分成多个小段，每段分别用筛法处理，减少内存使用。
2. <strong>奇数筛法（Odd Number Sieve）</strong>：除了2以外，所有素数都是奇数。可以在初始化`is_prime`数组时只标记奇数，这样可以将空间复杂度降低一半，同时减少一些不必要的计算。

然而，这些改进对于大多数应用场景来说并不是必需的，因为标准的Eratosthenes筛法已经足够高效。因此，在一般情况下，<strong>无需改进</strong>。
📝 记忆已更新，新增一条 'reflection' 记录。

✅ 反思认为代码已无需改进，任务完成。

--- 任务完成 ---
最终生成的代码：
```python
def find_primes(n):
    """
    Finds all prime numbers between 1 and n using the Sieve of Eratosthenes algorithm.

    :param n: The upper limit of the range to find prime numbers.
    :return: A list of all prime numbers between 1 and n.
    """
    if n < 2:
        return []

    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2
    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n + 1, p):
                is_prime[i] = False
        p += 1

    primes = [num for num in range(2, n + 1) if is_prime[num]]
    return primes
```
````

Ví dụ chạy thực tế này cho thấy cơ chế Reflection đã thúc đẩy Agent tiến hành tối ưu hóa chuyên sâu như thế nào:

1. <strong>"Phê bình" hiệu quả là tiền đề của tối ưu hóa</strong>：Trong vòng phản tư đầu tiên, do chúng ta sử dụng prompt "cực kỳ nghiêm khắc" và "tập trung vào hiệu quả thuật toán", Agent không hài lòng với code phiên bản đầu tiên tuy đúng về mặt chức năng, mà chỉ ra chính xác nút thắt cổ chai về độ phức tạp thời gian `O(n * sqrt(n))` của nó, và đề xuất cải thiện ở tầng thuật toán — thuật toán sàng Eratosthenes (sàng nguyên tố).
2. <strong>Cải thiện lặp lại (Iterative improvement)</strong>：Sau khi nhận được phản hồi rõ ràng, Agent đã thành công hiện thực hóa thuật toán sàng hiệu quả hơn ở giai đoạn tối ưu hóa, giảm độ phức tạp thuật toán xuống `O(n log log n)`, hoàn thành lần lặp tự thân có ý nghĩa đầu tiên.
3. <strong>Hội tụ và kết thúc (Convergence & Termination)</strong>：Trong vòng phản tư thứ hai, đối mặt với thuật toán sàng đã hiệu quả, Agent thể hiện kiến thức sâu sắc hơn. Nó không chỉ khẳng định hiệu quả của thuật toán hiện tại, thậm chí còn đề cập đến các hướng tối ưu cao cấp hơn như sàng phân đoạn (segmented sieve), nhưng cuối cùng đưa ra phán đoán chính xác "trong trường hợp thông thường không cần cải thiện". Phán đoán này đã kích hoạt điều kiện dừng của chúng ta, giúp quá trình tối ưu hóa hội tụ.

Ví dụ này đã chứng minh đầy đủ rằng, một cơ chế Reflection được thiết kế tốt, giá trị của nó không chỉ nằm ở việc sửa lỗi, mà quan trọng hơn là <strong>thúc đẩy giải pháp đạt được sự nâng cấp theo bậc thang về chất lượng và hiệu quả</strong>, điều này khiến nó trở thành một trong những kỹ thuật then chốt để xây dựng các Agent phức tạp, chất lượng cao.

### 4.4.5 Phân tích chi phí - lợi ích của cơ chế Reflection

Mặc dù cơ chế Reflection thể hiện xuất sắc trong việc nâng cao chất lượng giải quyết nhiệm vụ, nhưng việc có được năng lực này không phải không có cái giá phải trả. Trong ứng dụng thực tế, chúng ta cần cân nhắc giữa lợi ích mà nó mang lại và chi phí tương ứng.

（1）Chi phí chính

1. <strong>Chi phí gọi model tăng lên</strong>：Đây là chi phí trực tiếp nhất. Mỗi vòng lặp cần gọi thêm ít nhất hai lần Large Language Model (một lần cho phản tư, một lần cho tối ưu hóa). Nếu lặp nhiều vòng, chi phí gọi API và tiêu hao tài nguyên tính toán sẽ tăng theo cấp số nhân.

2. <strong>Độ trễ nhiệm vụ tăng đáng kể</strong>：Reflection là một quá trình tuần tự, việc tối ưu hóa mỗi vòng phải chờ phản tư của vòng trước hoàn thành. Điều này khiến tổng thời gian của nhiệm vụ kéo dài đáng kể, không phù hợp với các kịch bản yêu cầu tính thời gian thực cao.

3. <strong>Độ phức tạp của prompt engineering tăng lên</strong>：Như ví dụ của chúng ta đã cho thấy, thành công của Reflection phụ thuộc rất lớn vào prompt chất lượng cao, có mục tiêu rõ ràng. Thiết kế và debug các prompt hiệu quả cho các giai đoạn khác nhau như "thực thi", "phản tư", "tối ưu hóa", cần đầu tư nhiều công sức phát triển hơn.

（2）Lợi ích cốt lõi

1. <strong>Bước nhảy vọt về chất lượng giải pháp</strong>：Lợi ích lớn nhất nằm ở chỗ, nó có thể tối ưu hóa lặp lại một giải pháp ban đầu "đạt yêu cầu" thành một giải pháp cuối cùng "xuất sắc". Sự nâng cấp từ đúng về chức năng đến hiệu quả về hiệu suất, từ logic thô sơ đến logic chặt chẽ này, là vô cùng quan trọng trong nhiều nhiệm vụ then chốt.

2. <strong>Tăng cường tính bền vững và độ tin cậy</strong>：Thông qua vòng lặp tự sửa lỗi nội bộ, Agent có thể phát hiện và sửa chữa các lỗ hổng logic, lỗi sự thật hoặc xử lý trường hợp biên không đúng có thể tồn tại trong giải pháp ban đầu, từ đó nâng cao đáng kể độ tin cậy của kết quả cuối cùng.

Tóm lại, cơ chế Reflection là một chiến lược điển hình "đổi chi phí lấy chất lượng". Nó rất phù hợp với những kịch bản <strong>có yêu cầu cực cao về chất lượng, độ chính xác và độ tin cậy của kết quả cuối cùng, đồng thời yêu cầu về tính thời gian thực của việc hoàn thành nhiệm vụ tương đối nới lỏng</strong>. Ví dụ:

- Sinh code nghiệp vụ hoặc báo cáo kỹ thuật then chốt.
- Tiến hành suy luận logic phức tạp trong nghiên cứu khoa học.
- Hệ thống hỗ trợ quyết định cần phân tích và lập kế hoạch sâu.

Ngược lại, nếu kịch bản ứng dụng cần phản hồi nhanh, hoặc một câu trả lời "đại khái đúng" đã là đủ, thì sử dụng paradigm ReAct hoặc Plan-and-Solve nhẹ nhàng hơn có thể là lựa chọn có tính kinh tế cao hơn.

## 4.5 Tổng kết chương

Trong chương này, dựa trên nền tảng kiến thức về Large Language Model đã nắm vững ở Chương 3, chúng ta đã thông qua cách "tự tay làm lại bánh xe", mã hóa từ con số 0 hiện thực hóa ba paradigm xây dựng Agent kinh điển trong ngành: ReAct, Plan-and-Solve và Reflection. Chúng ta không chỉ khám phá nguyên lý hoạt động cốt lõi của chúng, mà còn thông qua các ví dụ thực hành cụ thể, hiểu sâu hơn về ưu điểm, hạn chế và kịch bản ứng dụng phù hợp của mỗi loại.

<strong>Điểm lại các kiến thức cốt lõi:</strong>

1. ReAct：Chúng ta đã xây dựng một Agent ReAct có thể tương tác với thế giới bên ngoài. Thông qua vòng lặp động "suy nghĩ - hành động - quan sát", nó đã thành công sử dụng search engine để trả lời các vấn đề mang tính thời sự mà kiến thức bản thân không thể bao phủ. Ưu thế cốt lõi của nó nằm ở <strong>khả năng thích ứng với môi trường</strong> và <strong>khả năng sửa lỗi động</strong>, khiến nó trở thành lựa chọn hàng đầu để xử lý các nhiệm vụ mang tính khám phá, cần input từ tool bên ngoài.
2. Plan-and-Solve：Chúng ta đã hiện thực hóa một Agent Plan-and-Solve lập kế hoạch trước, thực thi sau, và sử dụng nó để giải quyết bài toán ứng dụng toán học cần suy luận nhiều bước. Nó phân rã nhiệm vụ phức tạp thành các bước rõ ràng, sau đó thực thi lần lượt. Ưu thế cốt lõi của nó nằm ở <strong>tính cấu trúc</strong> và <strong>tính ổn định</strong>, đặc biệt phù hợp để xử lý các nhiệm vụ có đường logic xác định, suy luận nội bộ dày đặc.
3. Reflection (tự phản tư và lặp lại)：Chúng ta đã xây dựng một Agent Reflection có khả năng tự tối ưu hóa. Thông qua việc đưa vào vòng lặp "thực thi - phản tư - tối ưu hóa", nó đã thành công tối ưu hóa một phương án code ban đầu có hiệu suất thấp, thành một phiên bản hiệu năng cao hơn về mặt thuật toán. Giá trị cốt lõi của nó nằm ở khả năng <strong>nâng cao đáng kể chất lượng giải pháp</strong>, phù hợp với các kịch bản có yêu cầu cực cao về độ chính xác và độ tin cậy của kết quả.

Ba paradigm được thảo luận trong chương này đại diện cho ba chiến lược giải quyết vấn đề khác nhau của Agent, như Bảng 4.1. Trong ứng dụng thực tế, việc lựa chọn loại nào phụ thuộc vào nhu cầu cốt lõi của nhiệm vụ:

<div align="center">
<p>Bảng 4.1 Chiến lược lựa chọn các Agent Loop khác nhau</p>
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-4.png" alt="" width="70%"/>
</div>

Đến đây, chúng ta đã nắm vững các kỹ thuật cốt lõi để xây dựng một Agent đơn lẻ. Để chuyển tiếp kiến thức, cũng như đi sâu hơn vào ứng dụng thực tế. Mục tiếp theo chúng ta sẽ khám phá cách sử dụng các nền tảng low-code khác nhau cũng như phương án xây dựng agent bằng light-code.

## Bài tập

> <strong>Gợi ý</strong>：Một số bài tập không có đáp án chuẩn, trọng tâm là rèn luyện sự hiểu biết toàn diện và năng lực thực hành của người học về thiết kế paradigm Agent.

1. Chương này đã giới thiệu ba paradigm Agent kinh điển: `ReAct`, `Plan-and-Solve` và `Reflection`. Hãy phân tích:

   - Ba paradigm này có sự khác biệt bản chất gì trong cách tổ chức "suy nghĩ" và "hành động"?
   - Nếu muốn thiết kế một "trợ lý điều khiển nhà thông minh" (cần điều khiển nhiều thiết bị như đèn, điều hòa, rèm cửa, v.v., và tự động điều chỉnh theo thói quen người dùng), bạn sẽ chọn paradigm nào làm kiến trúc nền tảng? Tại sao?
   - Có thể kết hợp sử dụng ba paradigm này không? Nếu có thể, hãy thử thiết kế một kiến trúc Agent paradigm lai (hybrid), và giải thích kịch bản ứng dụng phù hợp của nó.

2. Trong hiện thực hóa `ReAct` ở mục 4.2, chúng ta đã sử dụng regular expression để parse output của Large Language Model (như `Thought` và `Action`). Hãy suy nghĩ:

   - Phương pháp parse hiện tại tồn tại những điểm yếu tiềm ẩn nào? Trong trường hợp nào có thể thất bại?
   - Ngoài regular expression, còn có những phương án parse output bền vững hơn nào?
   - Hãy thử sửa code trong chương này, sử dụng một định dạng output đáng tin cậy hơn, và so sánh ưu nhược điểm của hai phương án.

3. Tool calling là một trong những năng lực cốt lõi của Agent hiện đại. Dựa trên thiết kế `ToolExecutor` ở mục 4.2.2, hãy hoàn thành các thực hành mở rộng sau:

   > <strong>Gợi ý</strong>：Đây là câu hỏi thực hành, khuyến nghị tự viết code thực tế

   - Thêm một tool "máy tính" cho Agent ReAct, để nó có thể xử lý các bài toán tính toán phức tạp (như "tính kết quả của `(123 + 456) × 789/ 12 = ?`")
   - Thiết kế và hiện thực hóa một cơ chế xử lý "chọn tool thất bại": khi Agent nhiều lần gọi sai tool hoặc cung cấp tham số sai, hệ thống nên dẫn dắt nó sửa chữa như thế nào?
   - Suy nghĩ: nếu số lượng tool có thể gọi tăng lên $50$ thậm chí $100$, cách mô tả tool hiện tại liệu còn hiệu quả không? Khi số lượng tool có thể gọi tăng đáng kể theo nhu cầu nghiệp vụ, từ góc độ kỹ thuật nên tối ưu cơ chế tổ chức và truy xuất (retrieval) tool như thế nào?

4. Paradigm `Plan-and-Solve` phân rã nhiệm vụ thành hai giai đoạn "lập kế hoạch" và "thực thi". Hãy phân tích sâu:

   - Trong hiện thực hóa ở mục 4.3, kế hoạch được sinh ra ở giai đoạn lập kế hoạch là "tĩnh" (sinh một lần, không thể sửa đổi). Nếu trong quá trình thực thi phát hiện một bước nào đó không thể hoàn thành hoặc kết quả không như mong đợi, nên thiết kế một cơ chế "lập kế hoạch lại động" (dynamic re-planning) như thế nào?
   - So sánh `Plan-and-Solve` với `ReAct`: khi xử lý nhiệm vụ "đặt một chuyến công tác từ Bắc Kinh đến Thượng Hải (bao gồm vé máy bay, khách sạn, thuê xe)", paradigm nào phù hợp hơn? Tại sao?
   - Hãy thử thiết kế một hệ thống "lập kế hoạch phân tầng" (hierarchical planning): trước tiên sinh một kế hoạch trừu tượng ở tầng cao, sau đó với mỗi bước ở tầng cao lại sinh kế hoạch con chi tiết. Thiết kế này có ưu thế gì?

5. Cơ chế `Reflection` thông qua vòng lặp "thực thi - phản tư - tối ưu hóa" để nâng cao chất lượng output. Hãy suy nghĩ:

   - Trong ví dụ sinh code ở mục 4.4, các giai đoạn khác nhau sử dụng cùng một model. Nếu sử dụng hai model khác nhau (ví dụ, dùng một model mạnh hơn để phản tư, dùng một model nhanh hơn để thực thi), sẽ mang lại ảnh hưởng gì?
   - Điều kiện dừng của cơ chế `Reflection` là "phản hồi có chứa <strong>không cần cải thiện</strong>" hoặc "đạt đến số lần lặp tối đa". Thiết kế này có hợp lý không? Có thể thiết kế một điều kiện dừng thông minh hơn không?
   - Giả sử bạn muốn xây dựng một "trợ lý viết luận văn học thuật", nó có thể sinh bản nháp và không ngừng tối ưu nội dung luận văn. Hãy thiết kế một cơ chế Reflection đa chiều, phản tư và cải thiện từ nhiều góc độ như tính logic của đoạn văn, tính sáng tạo của phương pháp, cách diễn đạt ngôn ngữ, quy chuẩn trích dẫn, v.v.

6. Prompt engineering là kỹ thuật then chốt ảnh hưởng đến hiệu quả cuối cùng của Agent. Chương này đã trình bày nhiều prompt template được thiết kế tỉ mỉ. Hãy phân tích:

   - So sánh prompt `ReAct` ở mục 4.2.3 và prompt `Plan-and-Solve` ở mục 4.3.2, chúng rõ ràng có sự khác biệt rõ rệt về thiết kế cấu trúc, những khác biệt này phục vụ cho logic cốt lõi của từng paradigm như thế nào?
   - Trong prompt `Reflection` ở mục 4.4.3, chúng ta đã sử dụng thiết lập vai trò như "bạn là một chuyên gia đánh giá code cực kỳ nghiêm khắc". Hãy thử sửa thiết lập vai trò này (ví dụ đổi thành "bạn là một người bảo trì dự án mã nguồn mở chú trọng tính dễ đọc của code"), quan sát sự thay đổi của kết quả output, và tổng kết ảnh hưởng của việc thiết lập vai trò đến hành vi của Agent.
   - Thêm ví dụ `few-shot` vào prompt thường có thể nâng cao đáng kể khả năng tuân theo định dạng cụ thể của model. Hãy thử thêm ví dụ `few-shot` cho một Agent nào đó trong chương này, và so sánh hiệu quả của nó.

7. Một công ty khởi nghiệp thương mại điện tử hiện muốn sử dụng "Agent chăm sóc khách hàng" để thay thế nhân viên thật nhằm giảm chi phí tăng hiệu quả, nó cần có các chức năng sau:

   a. Hiểu lý do yêu cầu hoàn tiền của người dùng

   b. Truy vấn thông tin đơn hàng và trạng thái vận chuyển của người dùng

   c. Dựa trên chính sách công ty để phán đoán thông minh xem có nên phê duyệt hoàn tiền hay không

   d. Sinh một email phản hồi lịch sự và gửi đến hộp thư của người dùng

   e. Nếu phán đoán quyết định có tính tranh cãi nhất định (độ tự tin thấp hơn ngưỡng), có thể tự phản tư và đưa ra đề xuất thận trọng hơn

   Lúc này với vai trò là người phụ trách sản phẩm này:
   - Bạn sẽ chọn paradigm nào (hoặc sự kết hợp của các paradigm nào) trong chương này làm kiến trúc cốt lõi của hệ thống?
   - Hệ thống này cần những tool nào? Hãy liệt kê ít nhất 3 tool cùng mô tả chức năng của chúng.
   - Làm thế nào để thiết kế prompt nhằm đảm bảo quyết định của Agent vừa phù hợp lợi ích công ty, vừa duy trì sự thân thiện với người dùng?
   - Sản phẩm này sau khi ra mắt có thể đối mặt với những rủi ro và thách thức nào? Làm thế nào để giảm thiểu những rủi ro này thông qua biện pháp kỹ thuật?

## Tài liệu tham khảo

[1] Yao S, Zhao J, Yu D, et al. React: Synergizing reasoning and acting in language models[C]//International Conference on Learning Representations (ICLR). 2023.

[2] Wang L, Xu W, Lan Y, et al. Plan-and-solve prompting: Improving zero-shot chain-of-thought reasoning by large language models[J]. arXiv preprint arXiv:2305.04091, 2023.

[3] Shinn N, Cassano F, Gopinath A, et al. Reflexion: Language agents with verbal reinforcement learning[J]. Advances in Neural Information Processing Systems, 2023, 36: 8634-8652.
