# Chương 7: Xây dựng Agent Framework của riêng bạn

Trong các chương trước, chúng ta đã học các kiến thức nền tảng về Agent, và trải nghiệm sự tiện lợi trong phát triển mà các framework phổ biến mang lại. Bắt đầu từ chương này, chúng ta sẽ bước vào một giai đoạn thách thức hơn nhưng cũng có giá trị hơn: **xây dựng từ đầu, từng bước một, một Agent Framework — HelloAgents**.

Để đảm bảo tính liên tục và khả năng tái tạo của quá trình học tập, HelloAgents sẽ được phát triển theo hướng lặp phiên bản (version iteration). Mỗi chương sẽ bổ sung thêm các module chức năng mới dựa trên nền tảng của chương trước, đồng thời hệ thống hóa và hiện thực hóa các kiến thức liên quan đến Agent. Cuối cùng, chúng ta sẽ sử dụng framework tự xây dựng này để triển khai hiệu quả các case study ứng dụng nâng cao trong các chương tiếp theo của cuốn sách.

## 7.1 Thiết kế kiến trúc tổng thể của Framework

### 7.1.1 Tại sao cần tự xây dựng Agent Framework

Trong bối cảnh công nghệ Agent phát triển nhanh chóng hiện nay, trên thị trường đã có rất nhiều Agent Framework trưởng thành. Vậy tại sao chúng ta vẫn cần xây dựng một framework mới từ đầu?

(1) Sự thay đổi nhanh chóng và những hạn chế của các framework trên thị trường

Lĩnh vực Agent là một lĩnh vực phát triển nhanh chóng, bất cứ lúc nào cũng có thể xuất hiện khái niệm mới. Đối với thiết kế Agent, mỗi framework đều có định vị và cách hiểu riêng, nhưng các kiến thức cốt lõi về Agent là nhất quán.

- **Sự phức tạp do trừu tượng hóa quá mức**: Nhiều framework để theo đuổi tính tổng quát đã đưa vào rất nhiều lớp trừu tượng (abstraction layer) và tùy chọn cấu hình. Lấy LangChain làm ví dụ, cơ chế gọi theo chuỗi (chain) tuy linh hoạt nhưng lại có đường cong học tập (learning curve) rất dốc đối với người mới bắt đầu, thường đòi hỏi phải hiểu rất nhiều khái niệm chỉ để hoàn thành một tác vụ đơn giản.
- **Sự bất ổn định do thay đổi nhanh**: Các framework thương mại hóa, để chiếm lĩnh thị trường, thường xuyên thay đổi API interface. Các nhà phát triển thường xuyên gặp khó khăn khi code không chạy được sau khi nâng cấp phiên bản, chi phí bảo trì luôn ở mức cao.
- **Logic triển khai dạng hộp đen (black box)**: Nhiều framework đóng gói logic cốt lõi quá chặt chẽ, khiến nhà phát triển khó hiểu được cơ chế hoạt động bên trong của Agent, thiếu khả năng tùy biến sâu. Khi gặp vấn đề chỉ có thể dựa vào tài liệu và hỗ trợ từ cộng đồng, đặc biệt nếu cộng đồng không đủ sôi động, có thể một phản hồi (feedback) sẽ mất rất lâu mà không có ai xử lý, ảnh hưởng đến hiệu suất phát triển sau này.
- **Sự phức tạp của các mối quan hệ phụ thuộc (dependency)**: Các framework trưởng thành thường mang theo rất nhiều gói phụ thuộc, kích thước gói cài đặt lớn, khi cần phối hợp sử dụng với code của dự án khác có thể xảy ra xung đột phụ thuộc (dependency conflict).

(2) Sự chuyển đổi năng lực từ người dùng đến người xây dựng

Xây dựng Agent Framework của riêng mình, thực chất là một quá trình chuyển đổi từ "người sử dụng" sang "người xây dựng". Sự chuyển đổi này mang lại giá trị lâu dài.

- **Hiểu sâu nguyên lý hoạt động của Agent**: Thông qua việc tự tay triển khai từng thành phần, nhà phát triển có thể thực sự hiểu được quá trình tư duy của Agent, cơ chế gọi công cụ (tool), cũng như ưu nhược điểm và sự khác biệt giữa các pattern thiết kế khác nhau.
- **Có được quyền kiểm soát hoàn toàn**: Tự xây dựng framework đồng nghĩa với việc có quyền kiểm soát hoàn toàn đối với từng dòng code, có thể tinh chỉnh chính xác theo nhu cầu cụ thể, mà không bị ràng buộc bởi triết lý thiết kế của framework bên thứ ba.
- **Rèn luyện năng lực thiết kế hệ thống**: Quá trình xây dựng framework liên quan đến thiết kế mô-đun (modular design), trừu tượng hóa interface, xử lý lỗi (error handling) và các kỹ năng cốt lõi khác của kỹ thuật phần mềm, những năng lực này có giá trị quan trọng đối với sự phát triển lâu dài của nhà phát triển.

(3) Nhu cầu tùy biến và sự cần thiết của việc nắm vững sâu sắc

Trong ứng dụng thực tế, nhu cầu của các kịch bản khác nhau đối với Agent có sự khác biệt rất lớn, thường cần phải phát triển thêm (second development) trên nền tảng framework tổng quát.

- **Nhu cầu tối ưu hóa theo lĩnh vực đặc thù**: Các lĩnh vực chuyên biệt như tài chính, y tế, giáo dục thường cần template prompt mang tính định hướng, tích hợp công cụ đặc biệt, cũng như các chính sách bảo mật tùy biến.
- **Kiểm soát chính xác hiệu năng và tài nguyên**: Trong môi trường production, thời gian phản hồi, mức sử dụng bộ nhớ, khả năng xử lý đồng thời đều có yêu cầu nghiêm ngặt, giải pháp "một kích cỡ cho tất cả" (one-size-fits-all) của framework tổng quát thường không thể đáp ứng các nhu cầu tinh chỉnh chi tiết.
- **Yêu cầu về tính minh bạch trong học tập và giảng dạy**: Trong bối cảnh giảng dạy của chúng ta, người học mong đợi nhìn thấy rõ ràng từng bước xây dựng Agent, hiểu được cơ chế hoạt động của các paradigm khác nhau, điều này đòi hỏi framework phải có tính quan sát được (observability) và khả năng giải thích (explainability) cao.

### 7.1.2 Triết lý thiết kế của Framework HelloAgents

Xây dựng một Agent Framework mới, điều quan trọng không nằm ở số lượng tính năng, mà nằm ở việc triết lý thiết kế có thực sự giải quyết được những điểm đau (pain point) của các framework hiện có hay không. Thiết kế của framework HelloAgents xoay quanh một câu hỏi cốt lõi: làm thế nào để người học vừa có thể nhanh chóng bắt đầu sử dụng, vừa có thể hiểu sâu nguyên lý hoạt động của Agent?

Khi bạn lần đầu tiếp xúc với bất kỳ framework trưởng thành nào, có thể bạn sẽ bị thu hút bởi các tính năng phong phú của nó, nhưng rất nhanh sẽ phát hiện ra một vấn đề: để hoàn thành một tác vụ đơn giản, thường cần phải hiểu hàng chục khái niệm khác nhau như Chain, Agent, Tool, Memory, Retriever. Mỗi khái niệm đều có lớp trừu tượng riêng, đường cong học tập trở nên cực kỳ dốc. Sự phức tạp này tuy mang lại tính năng mạnh mẽ, nhưng cũng trở thành rào cản đối với người mới bắt đầu. Framework HelloAgents cố gắng tìm điểm cân bằng giữa tính đầy đủ của chức năng và sự thân thiện với việc học tập, hình thành bốn triết lý thiết kế cốt lõi.

(1) Cân bằng giữa tính nhẹ (lightweight) và thân thiện với giảng dạy

Một framework học tập xuất sắc nên có tính dễ đọc (readability) hoàn chỉnh. HelloAgents chia code cốt lõi theo từng chương, dựa trên một nguyên tắc đơn giản: bất kỳ nhà phát triển nào có nền tảng lập trình nhất định đều có thể hiểu hoàn toàn nguyên lý hoạt động của framework trong một khoảng thời gian hợp lý. Về mặt quản lý dependency, framework áp dụng chiến lược tối giản (minimalism). Ngoài SDK chính thức của OpenAI và một vài thư viện cơ bản cần thiết, không đưa vào bất kỳ dependency nặng nào. Khi gặp vấn đề, chúng ta có thể trực tiếp định vị đến code của chính framework, mà không cần phải tìm câu trả lời trong các mối quan hệ phụ thuộc phức tạp.

(2) Lựa chọn thực dụng dựa trên API chuẩn

API của OpenAI đã trở thành chuẩn mực của ngành (industry standard), hầu hết các nhà cung cấp LLM chủ đạo đều đang cố gắng tương thích với interface này. HelloAgents chọn xây dựng trên nền tảng chuẩn này, thay vì phát minh lại một bộ interface trừu tượng mới. Quyết định này chủ yếu xuất phát từ vài động lực. Đầu tiên là đảm bảo tính tương thích, khi bạn đã nắm vững cách sử dụng HelloAgents, khi chuyển sang framework khác hoặc tích hợp vào dự án hiện có, logic gọi API ở tầng dưới hoàn toàn nhất quán. Thứ hai là giảm chi phí học tập. Bạn không cần học mô hình khái niệm mới, vì tất cả các thao tác đều dựa trên interface chuẩn mà bạn đã quen thuộc.

(3) Thiết kế cẩn thận lộ trình học tập tăng dần (progressive learning path)

HelloAgents cung cấp một lộ trình học tập rõ ràng. Chúng tôi sẽ lưu code học tập của mỗi chương thành một phiên bản lịch sử có thể tải về bằng pip, vì vậy không cần lo lắng về chi phí sử dụng code, vì mỗi chức năng cốt lõi sẽ là do chính bạn viết ra. Thiết kế này cho phép bạn tiến lên theo nhu cầu và nhịp độ của riêng mình. Mỗi lần nâng cấp đều diễn ra một cách tự nhiên, không tạo ra bước nhảy về khái niệm hay khoảng trống về nhận thức. Đáng chú ý là, nội dung chương này cũng được hoàn thiện dựa trên nội dung của sáu chương trước. Đồng thời, chương này cũng đặt nền móng cho phần học tập kiến thức nâng cao ở các chương sau.

(4) Sự trừu tượng hóa thống nhất về "công cụ" (Tool): mọi thứ đều là công cụ

Để quán triệt triệt để triết lý nhẹ nhàng và thân thiện với giảng dạy, HelloAgents đã thực hiện một sự đơn giản hóa quan trọng về mặt kiến trúc: ngoài lớp (class) Agent cốt lõi, mọi thứ còn lại đều là Tools. Các module như Memory (bộ nhớ), RAG (Retrieval-Augmented Generation), RL (Reinforcement Learning), MCP (Model Context Protocol) cần được học riêng biệt trong nhiều framework khác, trong HelloAgents đều được trừu tượng hóa thống nhất thành một loại "công cụ". Ý định ban đầu của thiết kế này là loại bỏ các lớp trừu tượng không cần thiết, để người học có thể quay trở về logic cốt lõi trực quan nhất là "Agent gọi công cụ", từ đó thực sự đạt được sự thống nhất giữa việc nhanh chóng bắt đầu và hiểu sâu.

### 7.1.3 Mục tiêu học tập của chương này

Hãy cùng xem trước nội dung học tập cốt lõi của Chương 7:

```
hello-agents/
├── hello_agents/
│   │
│   ├── core/                     # Tầng framework cốt lõi
│   │   ├── agent.py              # Lớp cơ sở Agent
│   │   ├── llm.py                # Interface thống nhất HelloAgentsLLM
│   │   ├── message.py            # Hệ thống message
│   │   ├── config.py             # Quản lý cấu hình (config)
│   │   └── exceptions.py         # Hệ thống exception
│   │
│   ├── agents/                   # Tầng triển khai Agent
│   │   ├── simple_agent.py       # Triển khai SimpleAgent
│   │   ├── react_agent.py        # Triển khai ReActAgent
│   │   ├── reflection_agent.py   # Triển khai ReflectionAgent
│   │   └── plan_solve_agent.py   # Triển khai PlanAndSolveAgent
│   │
│   ├── tools/                    # Tầng hệ thống công cụ (tool)
│   │   ├── base.py               # Lớp cơ sở Tool
│   │   ├── registry.py           # Cơ chế đăng ký công cụ
│   │   ├── chain.py               # Hệ thống quản lý chuỗi công cụ (tool chain)
│   │   ├── async_executor.py     # Bộ thực thi công cụ bất đồng bộ (async)
│   │   └── builtin/              # Bộ công cụ tích hợp sẵn
│   │       ├── calculator.py     # Công cụ tính toán
│   │       └── search.py         # Công cụ tìm kiếm
└──
```

Trước khi bắt đầu viết code cụ thể, chúng ta cần xây dựng một bản thiết kế kiến trúc (architecture blueprint) rõ ràng. Thiết kế kiến trúc của HelloAgents tuân theo nguyên tắc cốt lõi "phân tầng tách rời (layered decoupling), đơn trách nhiệm (single responsibility), interface thống nhất", điều này vừa giữ được tính tổ chức của code, vừa tiện cho việc mở rộng nội dung theo từng chương.

**Bắt đầu nhanh: Cài đặt Framework HelloAgents**

Để người đọc có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp một gói Python có thể cài đặt trực tiếp. Bạn có thể cài đặt phiên bản tương ứng với chương này bằng lệnh sau:

```bash
# Link code framework hello-agents: https://github.com/jjyaoao/helloagents
# Yêu cầu phiên bản Python >=3.10
pip install "hello-agents==0.1.1"
```

Việc học tập trong chương này có thể thực hiện theo hai cách:

1. **Học tập trải nghiệm (experience-based learning)**: Trực tiếp cài đặt framework bằng `pip`, chạy code ví dụ, nhanh chóng trải nghiệm các chức năng khác nhau
2. **Học tập chuyên sâu (deep learning)**: Theo sát nội dung chương này, tự xây dựng từng component từ đầu, hiểu sâu triết lý thiết kế và chi tiết triển khai của framework

Chúng tôi khuyến nghị áp dụng lộ trình học tập "trải nghiệm trước, triển khai sau". Trong chương này, chúng tôi cung cấp các file test hoàn chỉnh, bạn có thể viết lại (rewrite) các hàm cốt lõi và chạy test, để kiểm tra xem triển khai của bạn có chính xác hay không. Cách học này vừa đảm bảo tính thực hành, vừa đảm bảo hiệu quả học tập. Nếu bạn muốn hiểu sâu hơn về chi tiết triển khai của framework, hoặc muốn tham gia vào quá trình phát triển framework, có thể truy cập [Repository GitHub](https://github.com/jjyaoao/helloagents) này.

Trước khi bắt đầu, hãy dùng 30 giây để trải nghiệm xây dựng một Agent đơn giản với Hello-agents!

```python
# Cấu hình API của large model trong file .env cùng cấp, có thể tham khảo file .env.example đi kèm thư mục code, hoặc tái sử dụng file .env từ các case ở các chương trước.
from hello_agents import SimpleAgent, HelloAgentsLLM
from dotenv import load_dotenv

# Tải biến môi trường (environment variables)
load_dotenv()

# Tạo instance LLM - framework tự động detect provider
llm = HelloAgentsLLM()

# Hoặc chỉ định provider thủ công (tùy chọn)
# llm = HelloAgentsLLM(provider="modelscope")

# Tạo SimpleAgent
agent = SimpleAgent(
    name="Trợ lý AI",
    llm=llm,
    system_prompt="Bạn là một trợ lý AI hữu ích"
)

# Hội thoại cơ bản
response = agent.run("Xin chào! Hãy giới thiệu về bản thân")
print(response)

# Thêm chức năng công cụ (tool) (tùy chọn)
from hello_agents.tools import CalculatorTool
calculator = CalculatorTool()
# Cần triển khai MySimpleAgent ở mục 7.4.1 để có thể gọi, các chương sau sẽ hỗ trợ cách gọi này
# agent.add_tool(calculator)

# Bây giờ có thể sử dụng công cụ
response = agent.run("Hãy giúp tôi tính 2 + 3 * 4")
print(response)

# Xem lịch sử hội thoại
print(f"Số lượng message trong lịch sử: {len(agent.get_history())}")
```



## 7.2 Mở rộng HelloAgentsLLM

Nội dung phần này sẽ lặp lại và nâng cấp dựa trên nền tảng `HelloAgentsLLM` đã tạo ở mục 4.1.3. Chúng ta sẽ cải tạo client cơ bản này thành một trung tâm gọi model (model invocation hub) có khả năng thích ứng cao hơn. Lần nâng cấp này chủ yếu xoay quanh ba mục tiêu sau:

1. **Hỗ trợ đa provider**: Thực hiện chuyển đổi liền mạch (seamless switching) giữa nhiều nhà cung cấp dịch vụ LLM phổ biến như OpenAI, ModelScope, Zhipu AI, tránh việc framework bị ràng buộc với một nhà cung cấp cụ thể.
2. **Tích hợp model local**: Đưa vào hai giải pháp triển khai local hiệu năng cao là VLLM và Ollama, như một bổ sung cấp production (production-grade) cho giải pháp Hugging Face Transformers ở mục 3.2.3, đáp ứng nhu cầu về bảo mật dữ liệu (data privacy) và kiểm soát chi phí.
3. **Cơ chế tự động detect**: Xây dựng một cơ chế nhận diện tự động, giúp framework có thể suy luận thông minh loại dịch vụ LLM đang sử dụng dựa trên thông tin môi trường, đơn giản hóa quá trình cấu hình của người dùng.

### 7.2.1 Hỗ trợ đa provider

Lớp `HelloAgentsLLM` mà chúng ta đã định nghĩa trước đây, đã có thể kết nối với bất kỳ dịch vụ nào tương thích với interface OpenAI thông qua hai tham số cốt lõi `api_key` và `base_url`. Về mặt lý thuyết, điều này đảm bảo tính tổng quát, nhưng trong ứng dụng thực tế, các nhà cung cấp dịch vụ khác nhau có sự khác biệt về cách đặt tên biến môi trường, địa chỉ API mặc định và model được đề xuất. Nếu mỗi lần chuyển đổi nhà cung cấp dịch vụ đều cần người dùng tự tra cứu và sửa code thủ công, sẽ ảnh hưởng rất lớn đến hiệu suất phát triển. Để giải quyết vấn đề này, chúng ta đưa vào `provider`. Ý tưởng cải tiến là: để `HelloAgentsLLM` xử lý nội bộ các chi tiết cấu hình của từng nhà cung cấp dịch vụ khác nhau, từ đó mang lại cho người dùng một trải nghiệm gọi thống nhất, đơn giản. Chi tiết triển khai cụ thể chúng ta sẽ trình bày kỹ ở mục 7.2.3 "Cơ chế tự động detect", tại đây, trước tiên chúng ta tập trung vào cách tận dụng cơ chế này để mở rộng framework.

Dưới đây, chúng ta sẽ trình diễn cách kế thừa (inherit) `HelloAgentsLLM` để bổ sung hỗ trợ cho nền tảng ModelScope. Chúng tôi mong muốn người đọc không chỉ học cách "sử dụng" framework, mà còn nắm vững cách "mở rộng" framework. Việc sửa trực tiếp source code của thư viện đã cài đặt là một cách làm không được khuyến khích, vì nó sẽ khiến việc nâng cấp thư viện sau này trở nên khó khăn.

(1) Tạo lớp LLM tùy biến (custom) và kế thừa

Giả sử trong thư mục dự án của chúng ta có một file `my_llm.py`. Đầu tiên chúng ta import lớp cơ sở `HelloAgentsLLM` từ thư viện `hello_agents`, sau đó tạo một lớp mới tên là `MyLLM` kế thừa nó.

```python
# my_llm.py
import os
from typing import Optional
from openai import OpenAI
from hello_agents import HelloAgentsLLM

class MyLLM(HelloAgentsLLM):
    """
    Một LLM client tùy biến, thông qua kế thừa đã bổ sung hỗ trợ cho ModelScope.
    """
    pass # Tạm thời để trống
```

(2) Viết lại (override) phương thức `__init__` để hỗ trợ nhà cung cấp mới

Tiếp theo, chúng ta viết lại phương thức `__init__` trong lớp `MyLLM`. Mục tiêu của chúng ta là: khi người dùng truyền vào `provider="modelscope"`, thực thi logic tùy biến của chúng ta; ngược lại, thì gọi logic gốc của lớp cha `HelloAgentsLLM`, để nó có thể tiếp tục hỗ trợ các nhà cung cấp tích hợp sẵn khác như OpenAI.

```python
class MyLLM(HelloAgentsLLM):
    def __init__(
        self,
        model: Optional[str] = None,
        api_key: Optional[str] = None,
        base_url: Optional[str] = None,
        provider: Optional[str] = "auto",
        **kwargs
    ):
        # Kiểm tra xem provider có phải là 'modelscope' mà chúng ta muốn xử lý không
        if provider == "modelscope":
            print("Đang sử dụng ModelScope Provider tùy biến")
            self.provider = "modelscope"
            
            # Phân giải (resolve) credential của ModelScope
            self.api_key = api_key or os.getenv("MODELSCOPE_API_KEY")
            self.base_url = base_url or "https://api-inference.modelscope.cn/v1/"
            
            # Xác thực xem credential có tồn tại hay không
            if not self.api_key:
                raise ValueError("ModelScope API key not found. Please set MODELSCOPE_API_KEY environment variable.")

            # Thiết lập model mặc định và các tham số khác
            self.model = model or os.getenv("LLM_MODEL_ID") or "Qwen/Qwen2.5-VL-72B-Instruct"
            self.temperature = kwargs.get('temperature', 0.7)
            self.max_tokens = kwargs.get('max_tokens')
            self.timeout = kwargs.get('timeout', 60)
            
            # Sử dụng các tham số đã lấy được để tạo instance OpenAI client
            self._client = OpenAI(api_key=self.api_key, base_url=self.base_url, timeout=self.timeout)

        else:
            # Nếu không phải modelscope, thì hoàn toàn sử dụng logic gốc của lớp cha để xử lý
            super().__init__(model=model, api_key=api_key, base_url=base_url, provider=provider, **kwargs)

```

Đoạn code này thể hiện tư tưởng "override": chúng ta chặn (intercept) trường hợp `provider="modelscope"` và xử lý đặc biệt, đối với tất cả các trường hợp còn lại, thì chuyển giao lại cho lớp cha thông qua `super().__init__(...)`, giữ nguyên toàn bộ chức năng của framework gốc.

(3) Sử dụng lớp `MyLLM` tùy biến

Bây giờ, chúng ta có thể sử dụng lớp `MyLLM` của riêng mình giống như sử dụng `HelloAgentsLLM` gốc trong logic nghiệp vụ của dự án.

Trước tiên, cấu hình API key của ModelScope trong file `.env`:

```bash
# .env file
MODELSCOPE_API_KEY="your-modelscope-api-key"
```

Sau đó, import và sử dụng `MyLLM` trong chương trình chính:

```python
# my_main.py
from dotenv import load_dotenv
from my_llm import MyLLM # Chú ý: ở đây import lớp của chính chúng ta

# Tải biến môi trường
load_dotenv()

# Khởi tạo client đã được viết lại của chúng ta, và chỉ định provider
llm = MyLLM(provider="modelscope") 

# Chuẩn bị message
messages = [{"role": "user", "content": "Xin chào, hãy giới thiệu về bản thân bạn."}]

# Thực hiện lời gọi, phương thức think và các phương thức khác đều đã được kế thừa từ lớp cha, không cần viết lại
response_stream = llm.think(messages)

# In ra response
print("ModelScope Response:")
for chunk in response_stream:
    # chunk đã được in ra trong thư viện my_llm, ở đây chỉ cần pass
    # print(chunk, end="", flush=True)
    pass
```

Thông qua các bước trên, chúng ta đã thành công mở rộng thêm chức năng mới cho thư viện `hello-agents` mà không cần sửa source code của nó. Phương pháp này không chỉ đảm bảo code gọn gàng, dễ bảo trì, mà còn đảm bảo chức năng tùy biến của chúng ta sẽ không bị mất đi khi nâng cấp thư viện `hello-agents` trong tương lai.

### 7.2.2 Gọi model local

Ở mục 3.2.3, chúng ta đã học cách sử dụng thư viện Hugging Face Transformers để chạy trực tiếp model mã nguồn mở (open-source) trên local. Phương pháp này rất phù hợp để nhập môn và kiểm chứng chức năng, nhưng cách triển khai ở tầng dưới của nó có hiệu năng hạn chế khi xử lý các request với mức độ đồng thời (concurrency) cao, thường không phải là lựa chọn hàng đầu cho môi trường production.

Để đạt được dịch vụ inference model hiệu năng cao, cấp production trên local, cộng đồng đã xuất hiện các công cụ xuất sắc như VLLM và Ollama. Chúng thông qua các kỹ thuật như continuous batching, PagedAttention, đã nâng cao đáng kể throughput và hiệu suất vận hành của model, đồng thời đóng gói model thành dịch vụ API tương thích với chuẩn OpenAI. Điều này có nghĩa là, chúng ta có thể tích hợp chúng một cách liền mạch vào `HelloAgentsLLM`.

**VLLM**

VLLM là một thư viện Python hiệu năng cao được thiết kế cho inference LLM. Thông qua các kỹ thuật tiên tiến như PagedAttention, nó có thể đạt được throughput cao hơn gấp nhiều lần so với triển khai Transformers tiêu chuẩn. Dưới đây là các bước đầy đủ để triển khai một dịch vụ VLLM trên local:

Trước tiên, cần cài đặt VLLM dựa theo môi trường phần cứng của bạn (đặc biệt là phiên bản CUDA). Khuyến nghị tuân theo [tài liệu chính thức](https://docs.vllm.ai/en/latest/getting_started/installation.html) của họ để cài đặt, nhằm tránh vấn đề không tương thích phiên bản.

```python
pip install vllm
```

Sau khi cài đặt xong, sử dụng lệnh sau để khởi động một dịch vụ API tương thích với OpenAI. VLLM sẽ tự động tải weight của model được chỉ định từ Hugging Face Hub (nếu local chưa có). Chúng ta vẫn lấy model Qwen1.5-0.5B-Chat làm ví dụ:

```
# Khởi động dịch vụ VLLM, và tải model Qwen1.5-0.5B-Chat
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-0.5B-Chat \
    --host 0.0.0.0 \
    --port 8000
```

Sau khi dịch vụ khởi động, sẽ cung cấp API tương thích với OpenAI tại địa chỉ `http://localhost:8000/v1`.

**Ollama**

Ollama tiếp tục đơn giản hóa việc quản lý và triển khai model local, nó đóng gói các bước tải model, cấu hình và khởi động dịch vụ vào một lệnh duy nhất, rất phù hợp để nhanh chóng bắt đầu sử dụng. Truy cập [trang chủ chính thức](https://ollama.com) của Ollama để tải và cài đặt client phù hợp với hệ điều hành của bạn.

Sau khi cài đặt, mở terminal, thực thi lệnh sau để tải và chạy một model (lấy Llama 3 làm ví dụ). Ollama sẽ tự động xử lý việc tải model, đóng gói dịch vụ và cấu hình tăng tốc phần cứng.

```
# Lần chạy đầu tiên sẽ tự động tải model, sau đó sẽ trực tiếp khởi động dịch vụ
ollama run llama3
```

Khi bạn thấy prompt tương tác của model trên terminal, điều đó có nghĩa là dịch vụ đã khởi động thành công ở chế độ nền (background). Mặc định Ollama sẽ expose API tương thích OpenAI tại địa chỉ `http://localhost:11434/v1`.

**Tích hợp vào `HelloAgentsLLM`**

Vì VLLM và Ollama đều tuân theo API chuẩn của ngành, nên quá trình tích hợp chúng vào `HelloAgentsLLM` rất đơn giản. Chúng ta chỉ cần coi chúng như một `provider` mới khi khởi tạo client.

Ví dụ, kết nối với dịch vụ **VLLM** đang chạy trên local:

```python
llm_client = HelloAgentsLLM(
    provider="vllm",
    model="Qwen/Qwen1.5-0.5B-Chat", # Cần nhất quán với model được chỉ định khi khởi động dịch vụ
    base_url="http://localhost:8000/v1",
    api_key="vllm" # Dịch vụ local thường không cần API Key thật, có thể điền bất kỳ chuỗi không rỗng nào
)
```

Hoặc, thông qua việc thiết lập biến môi trường và để client tự động detect, thực hiện việc không cần sửa code:

```bash
# Thiết lập trong file .env
LLM_BASE_URL="http://localhost:8000/v1"
LLM_API_KEY="vllm"

# Trong code Python chỉ cần khởi tạo trực tiếp
llm_client = HelloAgentsLLM() # Sẽ tự động detect là vllm
```

Tương tự, việc kết nối với dịch vụ **Ollama** trên local cũng đơn giản như vậy:

```python
llm_client = HelloAgentsLLM(
    provider="ollama",
    model="llama3", # Cần nhất quán với model được chỉ định bởi `ollama run`
    base_url="http://localhost:11434/v1",
    api_key="ollama" # Dịch vụ local cũng không cần Key thật
)
```

Thông qua thiết kế thống nhất này, code Agent cốt lõi của chúng ta không cần bất kỳ thay đổi nào, vẫn có thể tự do chuyển đổi giữa API đám mây (cloud) và model local. Điều này mang lại sự linh hoạt rất lớn cho việc phát triển, triển khai, kiểm soát chi phí cũng như bảo vệ quyền riêng tư dữ liệu (data privacy) của các ứng dụng sau này.

### 7.2.3 Cơ chế tự động detect

Để giảm thiểu tối đa gánh nặng cấu hình cho người dùng và tuân theo nguyên tắc "convention over configuration", `HelloAgentsLLM` đã thiết kế nội bộ hai phương thức hỗ trợ cốt lõi: `_auto_detect_provider` và `_resolve_credentials`. Chúng phối hợp hoạt động với nhau, `_auto_detect_provider` chịu trách nhiệm suy luận nhà cung cấp dịch vụ dựa trên thông tin môi trường, còn `_resolve_credentials` thì dựa trên kết quả suy luận để hoàn thành việc cấu hình tham số cụ thể.

Phương thức `_auto_detect_provider` chịu trách nhiệm, dựa trên thông tin môi trường, theo thứ tự ưu tiên dưới đây, thử tự động suy luận nhà cung cấp dịch vụ:

1. **Ưu tiên cao nhất: Kiểm tra biến môi trường của nhà cung cấp dịch vụ cụ thể** Đây là căn cứ phán đoán trực tiếp và đáng tin cậy nhất. Framework sẽ lần lượt kiểm tra xem các biến môi trường như `MODELSCOPE_API_KEY`, `OPENAI_API_KEY`, `ZHIPU_API_KEY` có tồn tại hay không. Một khi phát hiện bất kỳ biến nào, sẽ lập tức xác định nhà cung cấp dịch vụ tương ứng.

2. **Ưu tiên thứ hai: Phán đoán dựa trên `base_url`** Nếu người dùng không thiết lập key của nhà cung cấp dịch vụ cụ thể, nhưng đã thiết lập `LLM_BASE_URL` chung, framework sẽ chuyển sang phân tích URL này.

   - **Khớp domain**: Thông qua kiểm tra xem URL có chứa chuỗi đặc trưng như `"api-inference.modelscope.cn"`, `"api.openai.com"` hay không để nhận diện nhà cung cấp dịch vụ đám mây.

   - **Khớp port**: Thông qua kiểm tra xem URL có chứa các port chuẩn của dịch vụ local như `:11434` (Ollama), `:8000` (VLLM) hay không để nhận diện phương án triển khai local.

3. **Phán đoán bổ trợ: Phân tích định dạng (format) của API key** Trong một số trường hợp, nếu hai cách trên đều không thể xác định, framework sẽ thử phân tích định dạng của biến môi trường chung `LLM_API_KEY`. Ví dụ, API key của một số nhà cung cấp dịch vụ có tiền tố (prefix) cố định hoặc định dạng mã hóa (encoding) đặc trưng. Tuy nhiên, vì cách này có thể tồn tại sự mơ hồ (ambiguity) (ví dụ định dạng key của nhiều nhà cung cấp dịch vụ tương tự nhau), nên nó có mức độ ưu tiên thấp hơn, chỉ được dùng như một biện pháp bổ trợ.

Một phần code cốt lõi của nó như sau:

```python
def _auto_detect_provider(self, api_key: Optional[str], base_url: Optional[str]) -> str:
    """
    Tự động detect LLM provider
    """
    # 1. Kiểm tra biến môi trường của provider cụ thể (ưu tiên cao nhất)
    if os.getenv("MODELSCOPE_API_KEY"): return "modelscope"
    if os.getenv("OPENAI_API_KEY"): return "openai"
    if os.getenv("ZHIPU_API_KEY"): return "zhipu"
    # ... kiểm tra biến môi trường của các provider khác

    # Lấy biến môi trường chung
    actual_api_key = api_key or os.getenv("LLM_API_KEY")
    actual_base_url = base_url or os.getenv("LLM_BASE_URL")

    # 2. Phán đoán dựa trên base_url
    if actual_base_url:
        base_url_lower = actual_base_url.lower()
        if "api-inference.modelscope.cn" in base_url_lower: return "modelscope"
        if "open.bigmodel.cn" in base_url_lower: return "zhipu"
        if "localhost" in base_url_lower or "127.0.0.1" in base_url_lower:
            if ":11434" in base_url_lower: return "ollama"
            if ":8000" in base_url_lower: return "vllm"
            return "local" # Các port local khác

    # 3. Phán đoán bổ trợ dựa trên định dạng API key
    if actual_api_key:
        if actual_api_key.startswith("ms-"): return "modelscope"
        # ... phán đoán định dạng key của các provider khác

    # 4. Mặc định trả về 'auto', sử dụng cấu hình chung
    return "auto"
```

Một khi `provider` đã được xác định (dù là do người dùng chỉ định hay tự động detect), phương thức `_resolve_credentials` sẽ tiếp quản việc xử lý cấu hình khác biệt của nhà cung cấp dịch vụ. Nó sẽ dựa vào giá trị của `provider`, chủ động tìm kiếm biến môi trường tương ứng, và thiết lập `base_url` mặc định cho nó. Một phần triển khai cốt lõi của nó như sau:

```python
def _resolve_credentials(self, api_key: Optional[str], base_url: Optional[str]) -> tuple[str, str]:
    """Phân giải API key và base_url dựa trên provider"""
    if self.provider == "openai":
        resolved_api_key = api_key or os.getenv("OPENAI_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api.openai.com/v1"
        return resolved_api_key, resolved_base_url

    elif self.provider == "modelscope":
        resolved_api_key = api_key or os.getenv("MODELSCOPE_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api-inference.modelscope.cn/v1/"
        return resolved_api_key, resolved_base_url
    
    # ... logic của các provider khác
```

Hãy cùng cảm nhận sự tiện lợi mà cơ chế tự động detect mang lại thông qua một ví dụ đơn giản. Giả sử một người dùng muốn sử dụng dịch vụ Ollama trên local, anh ta chỉ cần cấu hình như sau trong file `.env`:

```bash
LLM_BASE_URL="http://localhost:11434/v1"
LLM_MODEL_ID="llama3"
```

Anh ta hoàn toàn không cần cấu hình `LLM_API_KEY` hoặc chỉ định `provider` trong code. Sau đó, trong code Python, anh ta chỉ cần đơn giản khởi tạo `HelloAgentsLLM`:

```python
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM

load_dotenv()

# Không cần truyền provider, framework sẽ tự động detect
llm = HelloAgentsLLM() 
# Log nội bộ của framework sẽ hiển thị đã detect được provider là 'ollama'

# Cách gọi sau đó hoàn toàn không đổi
messages = [{"role": "user", "content": "Xin chào!"}]
for chunk in llm.think(messages):
    print(chunk, end="")

```

Trong quá trình này, phương thức `_auto_detect_provider` thông qua việc phân tích `"localhost"` và `:11434` trong `LLM_BASE_URL`, đã suy luận thành công `provider` là `"ollama"`. Sau đó, phương thức `_resolve_credentials` sẽ thiết lập các tham số mặc định chính xác cho Ollama.

So với triển khai cơ bản ở mục 4.1.3, HelloAgentsLLM hiện tại có những ưu điểm nổi bật sau:

<div align="center">
  <p>Bảng 7.1 So sánh tính năng giữa các phiên bản HelloAgentLLM</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-01.png" alt="" width="90%"/>
</div>

Như Bảng 7.1 ở trên đã thể hiện, sự tiến hóa này thể hiện một nguyên tắc quan trọng trong thiết kế framework: **bắt đầu từ sự đơn giản, dần dần hoàn thiện**. Chúng ta vừa giữ cho interface đơn giản, vừa tăng cường tính đầy đủ của chức năng.



## 7.3 Triển khai Interface của Framework

Ở mục trước, chúng ta đã xây dựng thành phần cốt lõi `HelloAgentsLLM`, giải quyết vấn đề then chốt là giao tiếp với large language model. Tuy nhiên, nó vẫn cần một loạt interface và component đi kèm để xử lý dòng dữ liệu (data flow), quản lý cấu hình, ứng phó với exception, và cung cấp một cấu trúc rõ ràng, thống nhất cho việc xây dựng ứng dụng ở tầng trên. Mục này sẽ trình bày ba file cốt lõi sau:

- `message.py`: Định nghĩa định dạng message thống nhất trong framework, đảm bảo việc truyền tải thông tin giữa Agent và model được chuẩn hóa.
- `config.py`: Cung cấp một giải pháp quản lý cấu hình (config) tập trung, giúp hành vi của framework dễ dàng điều chỉnh và mở rộng.
- `agent.py`: Định nghĩa lớp cơ sở trừu tượng (`Agent`) cho tất cả các Agent, cung cấp interface và quy chuẩn thống nhất cho việc triển khai các loại Agent khác nhau sau này.

### 7.3.1 Lớp Message

Trong tương tác giữa Agent và large language model, lịch sử hội thoại (conversation history) là ngữ cảnh (context) cực kỳ quan trọng. Để quản lý thông tin này một cách chuẩn hóa, chúng ta thiết kế một lớp `Message` đơn giản. Ở chương về context engineering sau này, nó sẽ được mở rộng thêm.

```python
"""Hệ thống message"""
from typing import Optional, Dict, Any, Literal
from datetime import datetime
from pydantic import BaseModel

# Định nghĩa type cho role của message, giới hạn giá trị
MessageRole = Literal["user", "assistant", "system", "tool"]

class Message(BaseModel):
    """Lớp Message"""
    
    content: str
    role: MessageRole
    timestamp: datetime = None
    metadata: Optional[Dict[str, Any]] = None
    
    def __init__(self, content: str, role: MessageRole, **kwargs):
        super().__init__(
            content=content,
            role=role,
            timestamp=kwargs.get('timestamp', datetime.now()),
            metadata=kwargs.get('metadata', {})
        )
    
    def to_dict(self) -> Dict[str, Any]:
        """Chuyển sang định dạng dict (định dạng OpenAI API)"""
        return {
            "role": self.role,
            "content": self.content
        }
    
    def __str__(self) -> str:
        return f"[{self.role}] {self.content}"
```

Thiết kế của lớp này có vài điểm then chốt. Đầu tiên, chúng ta thông qua `typing.Literal` giới hạn nghiêm ngặt giá trị của trường `role` chỉ còn bốn loại `"user"`, `"assistant"`, `"system"`, `"tool"`, điều này tương ứng trực tiếp với quy chuẩn của OpenAI API, đảm bảo tính an toàn kiểu dữ liệu (type safety). Ngoài hai trường cốt lõi `content` và `role`, chúng ta còn bổ sung thêm `timestamp` và `metadata`, dành chỗ cho việc ghi log và mở rộng chức năng trong tương lai. Cuối cùng, phương thức `to_dict()` là một trong những chức năng cốt lõi của nó, chịu trách nhiệm chuyển đổi object `Message` được sử dụng nội bộ thành định dạng dict tương thích với OpenAI API, thể hiện nguyên tắc thiết kế "phong phú bên trong, tương thích bên ngoài".

### 7.3.2 Lớp Config

Trách nhiệm của lớp `Config` là tập trung hóa các tham số cấu hình được hardcode trong code, và hỗ trợ đọc từ biến môi trường.

```python
"""Quản lý cấu hình"""
import os
from typing import Optional, Dict, Any
from pydantic import BaseModel

class Config(BaseModel):
    """Lớp cấu hình của HelloAgents"""
    
    # Cấu hình LLM
    default_model: str = "gpt-3.5-turbo"
    default_provider: str = "openai"
    temperature: float = 0.7
    max_tokens: Optional[int] = None
    
    # Cấu hình hệ thống
    debug: bool = False
    log_level: str = "INFO"
    
    # Cấu hình khác
    max_history_length: int = 100
    
    @classmethod
    def from_env(cls) -> "Config":
        """Tạo config từ biến môi trường"""
        return cls(
            debug=os.getenv("DEBUG", "false").lower() == "true",
            log_level=os.getenv("LOG_LEVEL", "INFO"),
            temperature=float(os.getenv("TEMPERATURE", "0.7")),
            max_tokens=int(os.getenv("MAX_TOKENS")) if os.getenv("MAX_TOKENS") else None,
        )
    
    def to_dict(self) -> Dict[str, Any]:
        """Chuyển sang dict"""
        return self.dict()
```

Đầu tiên, chúng ta chia các mục cấu hình theo logic thành `Cấu hình LLM`, `Cấu hình hệ thống`, v.v., giúp cấu trúc rõ ràng một cách trực quan. Thứ hai, mỗi mục cấu hình đều có giá trị mặc định hợp lý, đảm bảo framework vẫn hoạt động được ngay cả khi không có cấu hình (zero-config). Cốt lõi nhất là classmethod `from_env()`, nó cho phép người dùng ghi đè (override) cấu hình mặc định thông qua việc thiết lập biến môi trường, mà không cần sửa code, điều này đặc biệt hữu ích khi triển khai ở các môi trường khác nhau.

### 7.3.3 Lớp cơ sở trừu tượng Agent

Lớp `Agent` là lớp trừu tượng ở tầng cao nhất của toàn bộ framework. Nó định nghĩa hành vi và thuộc tính chung mà một Agent cần có, nhưng không quan tâm đến cách triển khai cụ thể. Chúng ta thực hiện nó thông qua module `abc` (Abstract Base Classes) của Python, điều này bắt buộc tất cả các triển khai Agent cụ thể (như `SimpleAgent`, `ReActAgent` ở các chương sau) đều phải tuân theo cùng một "interface".

```python
"""Lớp cơ sở Agent"""
from abc import ABC, abstractmethod
from typing import Optional, Any
from .message import Message
from .llm import HelloAgentsLLM
from .config import Config

class Agent(ABC):
    """Lớp cơ sở Agent"""
    
    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None
    ):
        self.name = name
        self.llm = llm
        self.system_prompt = system_prompt
        self.config = config or Config()
        self._history: list[Message] = []
    
    @abstractmethod
    def run(self, input_text: str, **kwargs) -> str:
        """Chạy Agent"""
        pass
    
    def add_message(self, message: Message):
        """Thêm message vào lịch sử"""
        self._history.append(message)
    
    def clear_history(self):
        """Xóa lịch sử"""
        self._history.clear()
    
    def get_history(self) -> list[Message]:
        """Lấy lịch sử"""
        return self._history.copy()
    
    def __str__(self) -> str:
        return f"Agent(name={self.name}, provider={self.llm.provider})"
```

Thiết kế của lớp này thể hiện nguyên tắc trừu tượng hóa trong lập trình hướng đối tượng (OOP). Đầu tiên, nó thông qua việc kế thừa `ABC` được định nghĩa là một lớp trừu tượng không thể khởi tạo (instantiate) trực tiếp. Constructor `__init__` của nó định nghĩa rõ ràng các dependency cốt lõi của Agent: tên, instance LLM, system prompt và config. Phần quan trọng nhất là phương thức `run` được decorate bằng `@abstractmethod`, nó bắt buộc tất cả các lớp con phải triển khai phương thức này, từ đó đảm bảo tất cả các Agent đều có entry point (điểm vào) thực thi thống nhất. Ngoài ra, lớp cơ sở còn cung cấp các phương thức quản lý lịch sử chung, các phương thức này phối hợp hoạt động với lớp `Message`, thể hiện mối liên kết giữa các component.

Đến đây, chúng ta đã hoàn thành việc thiết kế và triển khai các component cơ sở cốt lõi của framework `HelloAgents`.

## 7.4 Hiện thực hóa các Paradigm Agent trong Framework

Nội dung phần này sẽ tái cấu trúc (refactor) theo hướng framework hóa, dựa trên ba paradigm Agent kinh điển (ReAct, Plan-and-Solve, Reflection) đã được xây dựng ở Chương 4, và bổ sung thêm SimpleAgent như một paradigm hội thoại cơ bản. Chúng ta sẽ cải tạo các triển khai Agent độc lập này thành các component framework dựa trên kiến trúc thống nhất. Lần tái cấu trúc này chủ yếu xoay quanh ba mục tiêu cốt lõi sau:

1. **Nâng cao tính hệ thống của kỹ thuật prompt**: Tối ưu hóa chuyên sâu các prompt ở Chương 4, chuyển từ định hướng tác vụ cụ thể sang thiết kế tổng quát hóa, đồng thời tăng cường ràng buộc định dạng (format) và định nghĩa vai trò (role).
2. **Chuẩn hóa thống nhất interface và định dạng**: Xây dựng lớp cơ sở Agent thống nhất và interface chạy được chuẩn hóa, tất cả các Agent đều tuân theo cùng tham số khởi tạo, chữ ký phương thức (method signature) và cơ chế quản lý lịch sử.
3. **Khả năng tùy biến có thể cấu hình cao**: Hỗ trợ người dùng tự định nghĩa template prompt, tham số cấu hình và chiến lược thực thi.

### 7.4.1 SimpleAgent

SimpleAgent là triển khai Agent cơ bản nhất, nó minh họa cách xây dựng một Agent hội thoại hoàn chỉnh trên nền tảng framework. Chúng ta sẽ kế thừa lớp `SimpleAgent` đã có sẵn trong framework và viết lại các phương thức cốt lõi của nó, để triển khai một phiên bản có thể mở rộng. Trước tiên, tạo một file `my_simple_agent.py` trong thư mục dự án của bạn:

```python
# my_simple_agent.py
from typing import Optional, Iterator
from hello_agents import SimpleAgent, HelloAgentsLLM, Config, Message,ToolRegistry

class MySimpleAgent(SimpleAgent):
    """
    Agent hội thoại đơn giản được viết lại
    Minh họa cách xây dựng Agent tùy biến dựa trên lớp cơ sở của framework
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        tool_registry: Optional['ToolRegistry'] = None,
        enable_tool_calling: bool = True
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.enable_tool_calling = enable_tool_calling and tool_registry is not None
        print(f"✅ {name} khởi tạo hoàn tất, gọi tool: {'bật' if self.enable_tool_calling else 'tắt'}")
```

Tiếp theo, chúng ta cần viết lại phương thức `run`. SimpleAgent hỗ trợ chức năng gọi tool tùy chọn, cũng để tiện cho việc mở rộng ở các chương sau:

```python
# Tiếp tục thêm vào my_simple_agent.py
import re

class MySimpleAgent(SimpleAgent):
    # ... phương thức __init__ ở trên

    def run(self, input_text: str, max_tool_iterations: int = 3, **kwargs) -> str:
        """
        Phương thức run được viết lại - triển khai logic hội thoại đơn giản, hỗ trợ gọi tool tùy chọn
        """
        print(f"🤖 {self.name} đang xử lý: {input_text}")

        # Xây dựng danh sách message
        messages = []

        # Thêm system message (có thể chứa thông tin tool)
        enhanced_system_prompt = self._get_enhanced_system_prompt()
        messages.append({"role": "system", "content": enhanced_system_prompt})

        # Thêm các message lịch sử
        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        # Thêm message hiện tại của user
        messages.append({"role": "user", "content": input_text})

        # Nếu không bật tool calling, sử dụng logic hội thoại đơn giản
        if not self.enable_tool_calling:
            response = self.llm.invoke(messages, **kwargs)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(response, "assistant"))
            print(f"✅ {self.name} phản hồi hoàn tất")
            return response

        # Logic hỗ trợ gọi tool nhiều vòng
        return self._run_with_tools(messages, input_text, max_tool_iterations, **kwargs)

    def _get_enhanced_system_prompt(self) -> str:
        """Xây dựng system prompt nâng cao, chứa thông tin tool"""
        base_prompt = self.system_prompt or "Bạn là một trợ lý AI hữu ích."

        if not self.enable_tool_calling or not self.tool_registry:
            return base_prompt

        # Lấy mô tả tool
        tools_description = self.tool_registry.get_tools_description()
        if not tools_description or tools_description == "暂无可用工具":
            return base_prompt

        tools_section = "\n\n## Công cụ khả dụng\n"
        tools_section += "Bạn có thể sử dụng các công cụ sau để giúp trả lời câu hỏi:\n"
        tools_section += tools_description + "\n"

        tools_section += "\n## Định dạng gọi tool\n"
        tools_section += "Khi cần sử dụng tool, vui lòng dùng định dạng sau:\n"
        tools_section += "`[TOOL_CALL:{tool_name}:{parameters}]`\n"
        tools_section += "Ví dụ: `[TOOL_CALL:search:Lập trình Python]` hoặc `[TOOL_CALL:memory:recall=thông tin user]`\n\n"
        tools_section += "Kết quả gọi tool sẽ tự động được chèn vào hội thoại, sau đó bạn có thể tiếp tục trả lời dựa trên kết quả đó.\n"

        return base_prompt + tools_section
```

Bây giờ chúng ta triển khai logic cốt lõi của việc gọi tool:

```python
# Tiếp tục thêm vào my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... các phương thức ở trên

    def _run_with_tools(self, messages: list, input_text: str, max_tool_iterations: int, **kwargs) -> str:
        """Logic chạy hỗ trợ gọi tool"""
        current_iteration = 0
        final_response = ""

        while current_iteration < max_tool_iterations:
            # Gọi LLM
            response = self.llm.invoke(messages, **kwargs)

            # Kiểm tra xem có lời gọi tool nào không
            tool_calls = self._parse_tool_calls(response)

            if tool_calls:
                print(f"🔧 Phát hiện {len(tool_calls)} lời gọi tool")
                # Thực thi tất cả các lời gọi tool và thu thập kết quả
                tool_results = []
                clean_response = response

                for call in tool_calls:
                    result = self._execute_tool_call(call['tool_name'], call['parameters'])
                    tool_results.append(result)
                    # Xóa dấu hiệu lời gọi tool khỏi response
                    clean_response = clean_response.replace(call['original'], "")

                # Xây dựng message chứa kết quả tool
                messages.append({"role": "assistant", "content": clean_response})

                # Thêm kết quả tool
                tool_results_text = "\n\n".join(tool_results)
                messages.append({"role": "user", "content": f"Kết quả thực thi tool:\n{tool_results_text}\n\nVui lòng đưa ra câu trả lời hoàn chỉnh dựa trên các kết quả này."})

                current_iteration += 1
                continue

            # Không có lời gọi tool, đây là câu trả lời cuối cùng
            final_response = response
            break

        # Nếu vượt quá số lần lặp tối đa, lấy câu trả lời cuối cùng
        if current_iteration >= max_tool_iterations and not final_response:
            final_response = self.llm.invoke(messages, **kwargs)

        # Lưu vào lịch sử
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(final_response, "assistant"))
        print(f"✅ {self.name} phản hồi hoàn tất")

        return final_response

    def _parse_tool_calls(self, text: str) -> list:
        """Phân tích (parse) lời gọi tool trong text"""
        pattern = r'\[TOOL_CALL:([^:]+):([^\]]+)\]'
        matches = re.findall(pattern, text)

        tool_calls = []
        for tool_name, parameters in matches:
            tool_calls.append({
                'tool_name': tool_name.strip(),
                'parameters': parameters.strip(),
                'original': f'[TOOL_CALL:{tool_name}:{parameters}]'
            })

        return tool_calls

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """Thực thi lời gọi tool"""
        if not self.tool_registry:
            return f"❌ Lỗi: chưa cấu hình tool registry"

        try:
            # Phân tích tham số thông minh
            if tool_name == 'calculator':
                # Tool calculator truyền thẳng expression vào
                result = self.tool_registry.execute_tool(tool_name, parameters)
            else:
                # Các tool khác sử dụng phân tích tham số thông minh
                param_dict = self._parse_tool_parameters(tool_name, parameters)
                tool = self.tool_registry.get_tool(tool_name)
                if not tool:
                    return f"❌ Lỗi: không tìm thấy tool '{tool_name}'"
                result = tool.run(param_dict)

            return f"🔧 Kết quả thực thi tool {tool_name}:\n{result}"

        except Exception as e:
            return f"❌ Gọi tool thất bại: {str(e)}"

    def _parse_tool_parameters(self, tool_name: str, parameters: str) -> dict:
        """Phân tích tham số tool một cách thông minh"""
        param_dict = {}

        if '=' in parameters:
            # Định dạng: key=value hoặc action=search,query=Python
            if ',' in parameters:
                # Nhiều tham số: action=search,query=Python,limit=3
                pairs = parameters.split(',')
                for pair in pairs:
                    if '=' in pair:
                        key, value = pair.split('=', 1)
                        param_dict[key.strip()] = value.strip()
            else:
                # Một tham số duy nhất: key=value
                key, value = parameters.split('=', 1)
                param_dict[key.strip()] = value.strip()
        else:
            # Truyền tham số trực tiếp, suy luận thông minh theo loại tool
            if tool_name == 'search':
                param_dict = {'query': parameters}
            elif tool_name == 'memory':
                param_dict = {'action': 'search', 'query': parameters}
            else:
                param_dict = {'input': parameters}

        return param_dict
```

Chúng ta cũng có thể bổ sung chức năng phản hồi streaming và các phương thức tiện ích (convenience method) cho Agent tùy biến:

```python
# Tiếp tục thêm vào my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... các phương thức ở trên

    def stream_run(self, input_text: str, **kwargs) -> Iterator[str]:
        """
        Phương thức chạy streaming tùy biến
        """
        print(f"🌊 {self.name} bắt đầu xử lý streaming: {input_text}")

        messages = []

        if self.system_prompt:
            messages.append({"role": "system", "content": self.system_prompt})

        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        messages.append({"role": "user", "content": input_text})

        # Gọi LLM dạng streaming
        full_response = ""
        print("📝 Phản hồi thời gian thực: ", end="")
        for chunk in self.llm.stream_invoke(messages, **kwargs):
            full_response += chunk
            yield chunk

        print()  # Xuống dòng

        # Lưu toàn bộ hội thoại vào lịch sử
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(full_response, "assistant"))
        print(f"✅ {self.name} phản hồi streaming hoàn tất")

    def add_tool(self, tool) -> None:
        """Thêm tool vào Agent (phương thức tiện ích)"""
        if not self.tool_registry:
            from hello_agents import ToolRegistry
            self.tool_registry = ToolRegistry()
            self.enable_tool_calling = True

        self.tool_registry.register_tool(tool)
        print(f"🔧 Tool '{tool.name}' đã được thêm")

    def has_tools(self) -> bool:
        """Kiểm tra xem có tool khả dụng hay không"""
        return self.enable_tool_calling and self.tool_registry is not None
    
    def remove_tool(self, tool_name: str) -> bool:
        """Xóa tool (phương thức tiện ích)"""
        if self.tool_registry:
            self.tool_registry.unregister(tool_name)
            return True
        return False
    
    def list_tools(self) -> list:
        """Liệt kê tất cả tool khả dụng"""
        if self.tool_registry:
            return self.tool_registry.list_tools()
        return []
```

Tạo một file test `test_simple_agent.py`:

```python
# test_simple_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM, ToolRegistry
from hello_agents.tools import CalculatorTool
from my_simple_agent import MySimpleAgent

# Tải biến môi trường
load_dotenv()

# Tạo instance LLM
llm = HelloAgentsLLM()

# Test 1: Agent hội thoại cơ bản (không có tool)
print("=== Test 1: Hội thoại cơ bản ===")
basic_agent = MySimpleAgent(
    name="Trợ lý cơ bản",
    llm=llm,
    system_prompt="Bạn là một trợ lý AI thân thiện, vui lòng trả lời câu hỏi một cách ngắn gọn, rõ ràng."
)

response1 = basic_agent.run("Xin chào, hãy giới thiệu về bản thân bạn")
print(f"Phản hồi hội thoại cơ bản: {response1}\n")

# Test 2: Agent có tích hợp tool
print("=== Test 2: Hội thoại tăng cường với tool ===")
tool_registry = ToolRegistry()
calculator = CalculatorTool()
tool_registry.register_tool(calculator)

enhanced_agent = MySimpleAgent(
    name="Trợ lý tăng cường",
    llm=llm,
    system_prompt="Bạn là một trợ lý thông minh, có thể sử dụng tool để giúp đỡ người dùng.",
    tool_registry=tool_registry,
    enable_tool_calling=True
)

response2 = enhanced_agent.run("Hãy giúp tôi tính 15 * 8 + 32")
print(f"Phản hồi tăng cường bởi tool: {response2}\n")

# Test 3: Phản hồi streaming
print("=== Test 3: Phản hồi streaming ===")
print("Phản hồi streaming: ", end="")
for chunk in basic_agent.stream_run("Hãy giải thích trí tuệ nhân tạo là gì"):
    pass  # Nội dung đã được in thời gian thực trong stream_run

# Test 4: Thêm tool động
print("\n=== Test 4: Quản lý tool động ===")
print(f"Trước khi thêm tool: {basic_agent.has_tools()}")
basic_agent.add_tool(calculator)
print(f"Sau khi thêm tool: {basic_agent.has_tools()}")
print(f"Tool khả dụng: {basic_agent.list_tools()}")

# Xem lịch sử hội thoại
print(f"\nLịch sử hội thoại: {len(basic_agent.get_history())} message")
```

Trong mục này, thông qua việc kế thừa lớp cơ sở `Agent`, chúng ta đã xây dựng thành công một Agent hội thoại cơ bản `MySimpleAgent` có đầy đủ chức năng và tuân theo quy chuẩn của framework. Nó không chỉ hỗ trợ hội thoại cơ bản, mà còn có khả năng gọi tool tùy chọn, phản hồi streaming và các phương thức quản lý tool tiện ích.

### 7.4.2 ReActAgent

ReActAgent được framework hóa trong khi vẫn giữ nguyên logic cốt lõi, đã nâng cao tính tổ chức và khả năng bảo trì của code, chủ yếu thông qua việc tối ưu hóa prompt và tích hợp với hệ thống tool của framework.

(1) Cải tiến template prompt

Giữ nguyên yêu cầu định dạng ban đầu, nhấn mạnh "mỗi lần chỉ được thực thi một bước", tránh gây rối loạn, và làm rõ hai kịch bản sử dụng của Action.

```python
MY_REACT_PROMPT = """Bạn là một trợ lý AI có khả năng suy luận (reasoning) và hành động (action). Bạn có thể phân tích vấn đề thông qua suy nghĩ, sau đó gọi tool phù hợp để lấy thông tin, cuối cùng đưa ra câu trả lời chính xác.

## Công cụ khả dụng
{tools}

## Quy trình làm việc
Vui lòng tuân thủ nghiêm ngặt định dạng sau để phản hồi, mỗi lần chỉ được thực thi một bước:

Thought: Phân tích vấn đề hiện tại, suy nghĩ cần thông tin gì hoặc cần thực hiện hành động gì.
Action: Chọn một hành động, định dạng phải là một trong các loại sau:
- `{{tool_name}}[{{tool_input}}]` - Gọi tool được chỉ định
- `Finish[câu trả lời cuối cùng]` - Khi bạn có đủ thông tin để đưa ra câu trả lời cuối cùng

## Lưu ý quan trọng
1. Mỗi lần phản hồi phải chứa cả hai phần Thought và Action
2. Định dạng gọi tool phải tuân thủ nghiêm ngặt: tên_tool[tham_số]
3. Chỉ khi bạn chắc chắn có đủ thông tin để trả lời câu hỏi, mới sử dụng Finish
4. Nếu thông tin trả về từ tool không đủ, hãy tiếp tục sử dụng tool khác hoặc tham số khác của cùng một tool

## Nhiệm vụ hiện tại
**Question:** {question}

## Lịch sử thực thi
{history}

Bây giờ hãy bắt đầu suy luận và hành động của bạn:
"""
```

(2) Triển khai hoàn chỉnh việc viết lại ReActAgent

Tạo file `my_react_agent.py` để viết lại ReActAgent:

```python
# my_react_agent.py
import re
from typing import Optional, List, Tuple
from hello_agents import ReActAgent, HelloAgentsLLM, Config, Message, ToolRegistry

class MyReActAgent(ReActAgent):
    """
    ReAct Agent được viết lại - Agent kết hợp suy luận và hành động
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        tool_registry: ToolRegistry,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        max_steps: int = 5,
        custom_prompt: Optional[str] = None
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.max_steps = max_steps
        self.current_history: List[str] = []
        self.prompt_template = custom_prompt if custom_prompt else MY_REACT_PROMPT
        print(f"✅ {name} khởi tạo hoàn tất, số bước tối đa: {max_steps}")
```

Ý nghĩa của các tham số khởi tạo của nó như sau:

- `name`: Tên của Agent.
- `llm`: Instance của `HelloAgentsLLM`, chịu trách nhiệm giao tiếp với large language model.
- `tool_registry`: Instance của `ToolRegistry`, dùng để quản lý và thực thi các tool khả dụng của Agent.
- `system_prompt`: System prompt, dùng để thiết lập vai trò và quy tắc hành vi của Agent.
- `config`: Object cấu hình, dùng để truyền các thiết lập ở cấp framework.
- `max_steps`: Số bước thực thi tối đa của vòng lặp ReAct, ngăn chặn vòng lặp vô hạn.
- `custom_prompt`: Template prompt tùy biến, dùng để thay thế prompt ReAct mặc định.

ReActAgent được framework hóa chia quy trình thực thi thành các bước rõ ràng:

```python
def run(self, input_text: str, **kwargs) -> str:
    """Chạy ReAct Agent"""
    self.current_history = []
    current_step = 0

    print(f"\n🤖 {self.name} bắt đầu xử lý vấn đề: {input_text}")

    while current_step < self.max_steps:
        current_step += 1
        print(f"\n--- Bước {current_step} ---")

        # 1. Xây dựng prompt
        tools_desc = self.tool_registry.get_tools_description()
        history_str = "\n".join(self.current_history)
        prompt = self.prompt_template.format(
            tools=tools_desc,
            question=input_text,
            history=history_str
        )

        # 2. Gọi LLM
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm.invoke(messages, **kwargs)

        # 3. Phân tích output
        thought, action = self._parse_output(response_text)

        # 4. Kiểm tra điều kiện hoàn thành
        if action and action.startswith("Finish"):
            final_answer = self._parse_action_input(action)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(final_answer, "assistant"))
            return final_answer

        # 5. Thực thi lời gọi tool
        if action:
            tool_name, tool_input = self._parse_action(action)
            observation = self.tool_registry.execute_tool(tool_name, tool_input)
            self.current_history.append(f"Action: {action}")
            self.current_history.append(f"Observation: {observation}")

    # Đạt đến số bước tối đa
    final_answer = "Xin lỗi, tôi không thể hoàn thành nhiệm vụ này trong số bước giới hạn."
    self.add_message(Message(input_text, "user"))
    self.add_message(Message(final_answer, "assistant"))
    return final_answer
```

Thông qua quá trình tái cấu trúc trên, chúng ta đã tích hợp thành công paradigm ReAct vào framework. Cải tiến cốt lõi nằm ở việc tận dụng interface `ToolRegistry` thống nhất, và thông qua một template prompt có thể cấu hình, định dạng nghiêm ngặt hơn, nâng cao tính ổn định của vòng lặp suy nghĩ-hành động (think-act loop) mà Agent thực thi. Đối với case test của ReAct, do cần gọi tool, nên code test sẽ được cung cấp thống nhất ở cuối bài.

### 7.4.3 ReflectionAgent

Vì các loại Agent này đã được triển khai logic cốt lõi ở Chương 4, nên ở đây chỉ đưa ra Prompt tương ứng. Khác với prompt chuyên biệt cho việc sinh code ở Chương 4, phiên bản được framework hóa áp dụng thiết kế tổng quát hóa, giúp nó phù hợp với nhiều kịch bản khác nhau như sinh văn bản, phân tích, sáng tác, và hỗ trợ tùy biến chuyên sâu thông qua tham số `custom_prompts`.

```python
DEFAULT_PROMPTS = {
    "initial": """
Vui lòng hoàn thành nhiệm vụ theo yêu cầu sau:

Nhiệm vụ: {task}

Vui lòng đưa ra một câu trả lời hoàn chỉnh, chính xác.
""",
    "reflect": """
Vui lòng xem xét kỹ lưỡng câu trả lời dưới đây, và chỉ ra những vấn đề hoặc không gian cải thiện có thể có:

# Nhiệm vụ gốc:
{task}

# Câu trả lời hiện tại:
{content}

Vui lòng phân tích chất lượng của câu trả lời này, chỉ ra những điểm chưa đủ tốt, và đưa ra đề xuất cải thiện cụ thể.
Nếu câu trả lời đã rất tốt, vui lòng trả lời "không cần cải thiện".
""",
    "refine": """
Vui lòng cải thiện câu trả lời của bạn dựa trên phản hồi:

# Nhiệm vụ gốc:
{task}

# Câu trả lời ở vòng trước:
{last_attempt}

# Ý kiến phản hồi:
{feedback}

Vui lòng đưa ra câu trả lời đã được cải thiện.
"""
}
```

Bạn có thể thử dựa trên code ở Chương 4, cũng như triển khai ReAct ở trên, để tự xây dựng `MyReflectionAgent` của riêng mình. Dưới đây cung cấp một đoạn code test để kiểm chứng ý tưởng.

```python
# test_reflection_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM
from my_reflection_agent import MyReflectionAgent

load_dotenv()
llm = HelloAgentsLLM()

# Sử dụng prompt tổng quát mặc định
general_agent = MyReflectionAgent(name="Trợ lý phản tư của tôi", llm=llm)

# Sử dụng prompt sinh code tùy biến (tương tự Chương 4)
code_prompts = {
    "initial": "Bạn là chuyên gia Python, vui lòng viết hàm: {task}",
    "reflect": "Vui lòng xem xét hiệu suất thuật toán của code:\nNhiệm vụ: {task}\nCode: {content}",
    "refine": "Vui lòng tối ưu hóa code dựa trên phản hồi:\nNhiệm vụ: {task}\nPhản hồi: {feedback}"
}
code_agent = MyReflectionAgent(
    name="Trợ lý sinh code của tôi",
    llm=llm,
    custom_prompts=code_prompts
)

# Thử nghiệm sử dụng
result = general_agent.run("Viết một bài báo ngắn về lịch sử phát triển của trí tuệ nhân tạo")
print(f"Kết quả cuối cùng: {result}")
```

### 7.4.4 PlanAndSolveAgent

Khác với output dạng văn bản tự do của Planner ở Chương 4, phiên bản được framework hóa bắt buộc Planner phải output kế hoạch theo định dạng list của Python, và cung cấp cơ chế xử lý exception hoàn chỉnh, đảm bảo các bước sau có thể thực thi một cách ổn định. Prompt Plan-and-Solve được framework hóa:

````bash
# Template prompt Planner mặc định
DEFAULT_PLANNER_PROMPT = """
Bạn là một chuyên gia lập kế hoạch (planning) AI hàng đầu. Nhiệm vụ của bạn là phân rã vấn đề phức tạp mà người dùng đưa ra thành một kế hoạch hành động gồm nhiều bước đơn giản.
Vui lòng đảm bảo mỗi bước trong kế hoạch đều là một nhiệm vụ con độc lập, có thể thực thi được, và được sắp xếp nghiêm ngặt theo thứ tự logic.
Output của bạn phải là một list Python, trong đó mỗi phần tử là một chuỗi mô tả một nhiệm vụ con.

Vấn đề: {question}

Vui lòng output kế hoạch của bạn nghiêm ngặt theo định dạng sau:
```python
["Bước 1", "Bước 2", "Bước 3", ...]
```
"""

# Template prompt Executor mặc định
DEFAULT_EXECUTOR_PROMPT = """
Bạn là một chuyên gia thực thi (execution) AI hàng đầu. Nhiệm vụ của bạn là giải quyết vấn đề từng bước một, nghiêm ngặt theo kế hoạch đã cho.
Bạn sẽ nhận được vấn đề gốc, kế hoạch hoàn chỉnh, cũng như các bước và kết quả đã hoàn thành cho đến thời điểm hiện tại.
Vui lòng tập trung giải quyết "bước hiện tại", và chỉ output câu trả lời cuối cùng của bước đó, không output bất kỳ giải thích hay đối thoại thừa nào.

# Vấn đề gốc:
{question}

# Kế hoạch hoàn chỉnh:
{plan}

# Các bước và kết quả lịch sử:
{history}

# Bước hiện tại:
{current_step}

Vui lòng chỉ output câu trả lời cho "bước hiện tại":
"""
````

Mục này vẫn cung cấp một file test tổng hợp `test_plan_solve_agent.py`, bạn có thể tự thiết kế và triển khai.

```python
# test_plan_solve_agent.py
from dotenv import load_dotenv
from hello_agents.core.llm import HelloAgentsLLM
from my_plan_solve_agent import MyPlanAndSolveAgent

# Tải biến môi trường
load_dotenv()

# Tạo instance LLM
llm = HelloAgentsLLM()

# Tạo PlanAndSolveAgent tùy biến
agent = MyPlanAndSolveAgent(
    name="Trợ lý lập kế hoạch và thực thi của tôi",
    llm=llm
)

# Thử nghiệm vấn đề phức tạp
question = "Một cửa hàng trái cây bán được 15 quả táo vào thứ Hai. Số lượng táo bán được vào thứ Ba gấp đôi thứ Hai. Số lượng bán được vào thứ Tư ít hơn thứ Ba 5 quả. Hỏi tổng cộng ba ngày này bán được bao nhiêu quả táo?"

result = agent.run(question)
print(f"\nKết quả cuối cùng: {result}")

# Xem lịch sử hội thoại
print(f"Lịch sử hội thoại: {len(agent.get_history())} message")
```

Cuối cùng có thể bổ sung một prompt mới, bạn có thể thử triển khai việc tải `custom_prompt` tùy biến.

```python
# Tạo prompt tùy biến chuyên dùng cho vấn đề toán học
math_prompts = {
    "planner": """
Bạn là chuyên gia lập kế hoạch cho vấn đề toán học. Vui lòng phân rã vấn đề toán học thành các bước tính toán:

Vấn đề: {question}

Định dạng output:
python
["Bước tính toán 1", "Bước tính toán 2", "Tính tổng"]

""",
    "executor": """
Bạn là chuyên gia tính toán. Vui lòng tính toán bước hiện tại:

Vấn đề: {question}
Kế hoạch: {plan}
Lịch sử: {history}
Bước hiện tại: {current_step}

Vui lòng chỉ output kết quả số:
"""
}

# Sử dụng prompt tùy biến để tạo Agent chuyên về toán học
math_agent = MyPlanAndSolveAgent(
    name="Trợ lý tính toán toán học",
    llm=llm,
    custom_prompts=math_prompts
)

# Thử nghiệm vấn đề toán học
math_result = math_agent.run(question)
print(f"Kết quả của Agent chuyên dụng toán học: {math_result}")
```

Như Bảng 7.2 đã thể hiện, thông qua việc tái cấu trúc theo hướng framework hóa này, chúng ta không chỉ giữ được chức năng cốt lõi của các paradigm Agent ở Chương 4, mà còn nâng cao đáng kể tính tổ chức, khả năng bảo trì và khả năng mở rộng của code. Tất cả các Agent hiện tại đều chia sẻ một kiến trúc cơ sở thống nhất, đồng thời vẫn giữ được đặc điểm và ưu thế riêng của từng loại.

<div align="center">
  <p>Bảng 7.2 So sánh triển khai Agent giữa các chương</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-02.png" alt="" width="90%"/>
</div>

### 7.4.5 FunctionCallAgent

FunctionCallAgent là Agent được hello-agents đưa vào sau phiên bản 0.2.8, nó là một Agent dựa trên cơ chế function calling (gọi hàm) nguyên bản của OpenAI, minh họa cách sử dụng cơ chế function calling của OpenAI để xây dựng Agent.
Nó hỗ trợ các chức năng sau:

- _build_tool_schemas: Xây dựng schema function calling của OpenAI thông qua description của tool
- _extract_message_content: Trích xuất văn bản từ response của OpenAI
- _parse_function_call_arguments: Phân tích tham số dạng chuỗi JSON mà model trả về
- _convert_parameter_types: Chuyển đổi kiểu dữ liệu của tham số

Những chức năng này giúp nó có được khả năng Function Calling nguyên bản của OpenAI, so với cách dùng prompt để ràng buộc, nó có tính bền vững (robustness) cao hơn.
```python
def _invoke_with_tools(self, messages: list[dict[str, Any]], tools: list[dict[str, Any]], tool_choice: Union[str, dict], **kwargs):
        """Gọi OpenAI client ở tầng dưới để thực thi function calling"""
        client = getattr(self.llm, "_client", None)
        if client is None:
            raise RuntimeError("HelloAgentsLLM 未正确初始化客户端，无法执行函数调用。")

        client_kwargs = dict(kwargs)
        client_kwargs.setdefault("temperature", self.llm.temperature)
        if self.llm.max_tokens is not None:
            client_kwargs.setdefault("max_tokens", self.llm.max_tokens)

        return client.chat.completions.create(
            model=self.llm.model,
            messages=messages,
            tools=tools,
            tool_choice=tool_choice,
            **client_kwargs,
        )

#Logic nội bộ là đóng gói lại function call nguyên bản của Openai
#Ví dụ function call nguyên bản của OpenAI
from openai import OpenAI
client = OpenAI()

tools = [
  {
    "type": "function",
    "function": {
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city and state, e.g. San Francisco, CA",
          },
          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
        },
        "required": ["location"],
      },
    }
  }
]
messages = [{"role": "user", "content": "What's the weather like in Boston today?"}]
completion = client.chat.completions.create(
  model="gpt-5",
  messages=messages,
  tools=tools,
  tool_choice="auto"
)

print(completion)
```

## 7.5 Hệ thống công cụ (Tool)

Nội dung phần này sẽ dựa trên kiến trúc nền tảng Agent đã xây dựng ở các phần trước, đi sâu vào thiết kế và triển khai hệ thống công cụ (tool system). Chúng ta sẽ bắt đầu từ việc xây dựng hạ tầng cơ sở, dần dần đi sâu vào thiết kế phát triển tùy biến. Mục tiêu học tập của phần này xoay quanh ba khía cạnh cốt lõi sau:

1. **Trừu tượng hóa và quản lý tool thống nhất**: Xây dựng lớp cơ sở Tool chuẩn hóa và cơ chế đăng ký ToolRegistry, cung cấp hạ tầng thống nhất cho việc phát triển, đăng ký, khám phá (discovery) và thực thi tool.

2. **Phát triển tool theo hướng thực chiến**: Lấy công cụ tính toán toán học làm case study, minh họa cách thiết kế và triển khai tool tùy biến, giúp người đọc nắm vững toàn bộ quy trình phát triển tool.

3. **Chiến lược tích hợp và tối ưu hóa nâng cao**: Thông qua thiết kế tool tìm kiếm đa nguồn (multi-source search), minh họa cách tích hợp nhiều dịch vụ bên ngoài, thực hiện lựa chọn backend thông minh, hợp nhất kết quả và xử lý chịu lỗi (fault tolerance), thể hiện tư duy thiết kế của hệ thống tool trong các kịch bản phức tạp.

### 7.5.1 Thiết kế lớp cơ sở Tool và cơ chế đăng ký

Khi xây dựng một hệ thống tool có khả năng mở rộng, trước tiên chúng ta cần xây dựng một bộ hạ tầng cơ sở chuẩn hóa. Hạ tầng này bao gồm lớp cơ sở Tool, registry ToolRegistry, cũng như cơ chế quản lý tool.

(1) Thiết kế trừu tượng của lớp cơ sở Tool

Lớp cơ sở Tool là sự trừu tượng hóa cốt lõi của toàn bộ hệ thống tool, nó định nghĩa quy chuẩn interface mà tất cả các tool phải tuân theo:

````python
class Tool(ABC):
    """Lớp cơ sở Tool"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description

    @abstractmethod
    def run(self, parameters: Dict[str, Any]) -> str:
        """Thực thi tool"""
        pass

    @abstractmethod
    def get_parameters(self) -> List[ToolParameter]:
        """Lấy định nghĩa tham số của tool"""
        pass
````
Thiết kế này thể hiện tư tưởng cốt lõi của thiết kế hướng đối tượng: thông qua interface `run` thống nhất, tất cả các tool đều có thể được thực thi theo cách nhất quán, nhận tham số dạng dict và trả về kết quả dạng string, đảm bảo tính nhất quán của framework. Đồng thời, tool có khả năng tự mô tả (self-describing), thông qua phương thức `get_parameters` có thể thông báo rõ ràng cho bên gọi biết mình cần tham số gì, cơ chế introspection này tạo nền tảng cho việc tự động sinh tài liệu (documentation) và xác thực tham số. Còn thiết kế metadata như name và description, thì giúp hệ thống tool có khả năng khám phá (discoverability) và dễ hiểu tốt.

(2) Hệ thống định nghĩa tham số ToolParameter

Để hỗ trợ việc xác thực tham số phức tạp và sinh tài liệu, chúng ta thiết kế lớp ToolParameter:

````python
class ToolParameter(BaseModel):
    """Định nghĩa tham số của tool"""
    name: str
    type: str
    description: str
    required: bool = True
    default: Any = None
````
Thiết kế này giúp tool có thể mô tả chính xác nhu cầu tham số của mình, hỗ trợ kiểm tra kiểu dữ liệu (type checking), thiết lập giá trị mặc định và tự động sinh tài liệu.

(3) Triển khai registry ToolRegistry

ToolRegistry là trung tâm quản lý của hệ thống tool, nó cung cấp các chức năng cốt lõi như đăng ký, khám phá, thực thi tool, trong mục này chúng ta chủ yếu sử dụng các chức năng sau:

````python
class ToolRegistry:
    """Registry tool của HelloAgents"""

    def __init__(self):
        self._tools: dict[str, Tool] = {}
        self._functions: dict[str, dict[str, Any]] = {}

    def register_tool(self, tool: Tool):
        """Đăng ký object Tool"""
        if tool.name in self._tools:
            print(f"⚠️ Cảnh báo: tool '{tool.name}' đã tồn tại, sẽ bị ghi đè.")
        self._tools[tool.name] = tool
        print(f"✅ Tool '{tool.name}' đã được đăng ký.")
        
    def register_function(self, name: str, description: str, func: Callable[[str], str]):
        """
        Đăng ký trực tiếp function làm tool (cách đơn giản)

        Args:
            name: tên tool
            description: mô tả tool
            func: hàm tool, nhận tham số dạng chuỗi, trả về kết quả dạng chuỗi
        """
        if name in self._functions:
            print(f"⚠️ Cảnh báo: tool '{name}' đã tồn tại, sẽ bị ghi đè.")

        self._functions[name] = {
            "description": description,
            "func": func
        }
        print(f"✅ Tool '{name}' đã được đăng ký.")
````
ToolRegistry hỗ trợ hai cách đăng ký:

1. **Đăng ký object Tool**: Phù hợp với tool phức tạp, hỗ trợ định nghĩa và xác thực tham số đầy đủ
2. **Đăng ký function trực tiếp**: Phù hợp với tool đơn giản, tích hợp nhanh với function có sẵn

(4) Cơ chế khám phá và quản lý tool

Registry cung cấp các chức năng quản lý tool phong phú:

````python
def get_tools_description(self) -> str:
    """Lấy chuỗi mô tả đã được định dạng của tất cả tool khả dụng"""
    descriptions = []

    # Mô tả object Tool
    for tool in self._tools.values():
        descriptions.append(f"- {tool.name}: {tool.description}")

    # Mô tả tool dạng function
    for name, info in self._functions.items():
        descriptions.append(f"- {name}: {info['description']}")

    return "\n".join(descriptions) if descriptions else "暂无可用工具"
````
Chuỗi mô tả được sinh ra bởi phương thức này có thể trực tiếp dùng để xây dựng prompt cho Agent, giúp Agent biết được các tool khả dụng.

````python
def to_openai_schema(self) -> Dict[str, Any]:
        """Chuyển sang định dạng schema function calling của OpenAI

        Dùng cho FunctionCallAgent, giúp tool có thể được sử dụng bởi function calling nguyên bản của OpenAI

        Returns:
            Schema tuân theo chuẩn function calling của OpenAI
        """
        parameters = self.get_parameters()

        # Xây dựng properties
        properties = {}
        required = []

        for param in parameters:
            # Định nghĩa thuộc tính cơ bản
            prop = {
                "type": param.type,
                "description": param.description
            }

            # Nếu có giá trị mặc định, thêm vào description (schema OpenAI không hỗ trợ trường default)
            if param.default is not None:
                prop["description"] = f"{param.description} (mặc định: {param.default})"

            # Nếu là kiểu array, thêm định nghĩa items
            if param.type == "array":
                prop["items"] = {"type": "string"}  # Mặc định là mảng chuỗi

            properties[param.name] = prop

            # Thu thập các tham số bắt buộc
            if param.required:
                required.append(param.name)

        return {
            "type": "function",
            "function": {
                "name": self.name,
                "description": self.description,
                "parameters": {
                    "type": "object",
                    "properties": properties,
                    "required": required
                }
            }
        }
````
Schema được sinh ra bởi phương thức này có thể trực tiếp dùng cho việc gọi tool trong SDK nguyên bản của OpenAI.

### 7.5.2 Phát triển tool tùy biến

Có được hạ tầng cơ sở rồi, hãy cùng xem cách phát triển một tool tùy biến hoàn chỉnh. Công cụ tính toán toán học là một ví dụ tốt, vì nó đơn giản, trực quan, cách trực tiếp nhất là sử dụng chức năng đăng ký function của ToolRegistry.

Hãy cùng tạo một công cụ tính toán toán học tùy biến. Trước tiên, tạo file `my_calculator_tool.py` trong thư mục dự án của bạn:

```python
# my_calculator_tool.py
import ast
import operator
import math
from hello_agents import ToolRegistry

def my_calculate(expression: str) -> str:
    """Hàm tính toán toán học đơn giản"""
    if not expression.strip():
        return "Biểu thức tính toán không được để trống"

    # Các phép toán cơ bản được hỗ trợ
    operators = {
        ast.Add: operator.add,      # +
        ast.Sub: operator.sub,      # -
        ast.Mult: operator.mul,     # *
        ast.Div: operator.truediv,  # /
    }

    # Các hàm cơ bản được hỗ trợ
    functions = {
        'sqrt': math.sqrt,
        'pi': math.pi,
    }

    try:
        node = ast.parse(expression, mode='eval')
        result = _eval_node(node.body, operators, functions)
        return str(result)
    except:
        return "Tính toán thất bại, vui lòng kiểm tra lại định dạng biểu thức"

def _eval_node(node, operators, functions):
    """Tính toán giá trị biểu thức đơn giản hóa"""
    if isinstance(node, ast.Constant):
        return node.value
    elif isinstance(node, ast.BinOp):
        left = _eval_node(node.left, operators, functions)
        right = _eval_node(node.right, operators, functions)
        op = operators.get(type(node.op))
        return op(left, right)
    elif isinstance(node, ast.Call):
        func_name = node.func.id
        if func_name in functions:
            args = [_eval_node(arg, operators, functions) for arg in node.args]
            return functions[func_name](*args)
    elif isinstance(node, ast.Name):
        if node.id in functions:
            return functions[node.id]

def create_calculator_registry():
    """Tạo tool registry có chứa calculator"""
    registry = ToolRegistry()

    # Đăng ký function calculator
    registry.register_function(
        name="my_calculator",
        description="Công cụ tính toán toán học đơn giản, hỗ trợ phép toán cơ bản (+,-,*,/) và hàm sqrt",
        func=my_calculate
    )

    return registry
```

Tool không chỉ hỗ trợ bốn phép toán cơ bản, mà còn bao gồm các hàm và hằng số toán học thường dùng, đáp ứng nhu cầu của phần lớn các kịch bản tính toán. Bạn cũng có thể tự mở rộng file này, tạo ra một hàm tính toán hoàn thiện hơn. Chúng tôi cung cấp một file test `test_my_calculator.py` để giúp bạn kiểm chứng việc triển khai của mình:

```python
# test_my_calculator.py
from dotenv import load_dotenv
from my_calculator_tool import create_calculator_registry

# Tải biến môi trường
load_dotenv()

def test_calculator_tool():
    """Test tool calculator tùy biến"""

    # Tạo registry có chứa calculator
    registry = create_calculator_registry()

    print("🧪 Test tool calculator tùy biến\n")

    # Các test case đơn giản
    test_cases = [
        "2 + 3",           # Phép cộng cơ bản
        "10 - 4",          # Phép trừ cơ bản
        "5 * 6",           # Phép nhân cơ bản
        "15 / 3",          # Phép chia cơ bản
        "sqrt(16)",        # Căn bậc hai
    ]

    for i, expression in enumerate(test_cases, 1):
        print(f"Test {i}: {expression}")
        result = registry.execute_tool("my_calculator", expression)
        print(f"Kết quả: {result}\n")

def test_with_simple_agent():
    """Test tích hợp với SimpleAgent"""
    from hello_agents import HelloAgentsLLM

    # Tạo LLM client
    llm = HelloAgentsLLM()

    # Tạo registry có chứa calculator
    registry = create_calculator_registry()

    print("🤖 Test tích hợp với SimpleAgent:")

    # Mô phỏng kịch bản SimpleAgent sử dụng tool
    user_question = "Hãy giúp tôi tính sqrt(16) + 2 * 3"

    print(f"Câu hỏi của user: {user_question}")

    # Sử dụng tool để tính toán
    calc_result = registry.execute_tool("my_calculator", "sqrt(16) + 2 * 3")
    print(f"Kết quả tính toán: {calc_result}")

    # Xây dựng câu trả lời cuối cùng
    final_messages = [
        {"role": "user", "content": f"Kết quả tính toán là {calc_result}, vui lòng trả lời câu hỏi của user bằng ngôn ngữ tự nhiên: {user_question}"}
    ]

    print("\n🎯 Câu trả lời của SimpleAgent:")
    response = llm.think(final_messages)
    for chunk in response:
        print(chunk, end="", flush=True)
    print("\n")

if __name__ == "__main__":
    test_calculator_tool()
    test_with_simple_agent()
```

Thông qua case study công cụ tính toán toán học đơn giản hóa này, chúng ta đã học được cách nhanh chóng phát triển một tool tùy biến: viết một hàm tính toán đơn giản, đăng ký thông qua ToolRegistry, sau đó tích hợp sử dụng với SimpleAgent. Để quan sát trực quan hơn, ở đây cung cấp Hình 7.1, giúp bạn hiểu rõ logic vận hành của code.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/01.png" alt="" width="90%"/>
  <p>Hình 7.1 Luồng công việc vận hành của SimpleAgent dựa trên Helloagents</p>
</div>

### 7.5.3 Công cụ tìm kiếm đa nguồn (multi-source search)

Trong ứng dụng thực tế, chúng ta thường cần tích hợp nhiều dịch vụ bên ngoài để cung cấp chức năng mạnh mẽ hơn. Công cụ tìm kiếm là một ví dụ điển hình, nó tích hợp nhiều search engine, có thể cung cấp thông tin thực tế đầy đủ hơn. Ở Chương 1 chúng ta đã sử dụng API search của Tavily, ở Chương 4 chúng ta đã sử dụng API search của SerpApi. Vì vậy lần này chúng ta sẽ sử dụng cả hai API này để triển khai chức năng tìm kiếm đa nguồn. Nếu chưa cài đặt dependency Python tương ứng, bạn có thể chạy script sau:

```bash
pip install "hello-agents[search]==0.1.1"
```

(1) Thiết kế interface thống nhất cho tool tìm kiếm

SearchTool tích hợp sẵn trong framework HelloAgents minh họa cách thiết kế một tool tìm kiếm đa nguồn nâng cao:

````python
class SearchTool(Tool):
    """
    Công cụ tìm kiếm lai (hybrid) thông minh

    Hỗ trợ nhiều backend search engine, tự động lựa chọn nguồn tìm kiếm tốt nhất:
    1. Chế độ lai (hybrid) - tự động lựa chọn TAVILY hoặc SERPAPI
    2. Tavily API (tavily) - tìm kiếm AI chuyên nghiệp
    3. SerpApi (serpapi) - tìm kiếm Google truyền thống
    """

    def __init__(self, backend: str = "hybrid", tavily_key: Optional[str] = None, serpapi_key: Optional[str] = None):
        super().__init__(
            name="search",
            description="Một search engine web thông minh. Hỗ trợ chế độ tìm kiếm lai, tự động lựa chọn nguồn tìm kiếm tốt nhất."
        )
        self.backend = backend
        self.tavily_key = tavily_key or os.getenv("TAVILY_API_KEY")
        self.serpapi_key = serpapi_key or os.getenv("SERPAPI_API_KEY")
        self.available_backends = []
        self._setup_backends()
````
Tư tưởng cốt lõi của thiết kế này là tự động lựa chọn backend tìm kiếm tốt nhất dựa trên API key và thư viện dependency khả dụng.

(2) Chiến lược tích hợp giữa nguồn tìm kiếm TAVILY và SERPAPI

Framework triển khai logic lựa chọn backend thông minh:

````python
def _search_hybrid(self, query: str) -> str:
    """Tìm kiếm lai - tự động lựa chọn nguồn tìm kiếm tốt nhất"""
    # Ưu tiên sử dụng Tavily (tìm kiếm được tối ưu bởi AI)
    if "tavily" in self.available_backends:
        try:
            return self._search_tavily(query)
        except Exception as e:
            print(f"⚠️ Tìm kiếm Tavily thất bại: {e}")
            # Nếu Tavily thất bại, thử dùng SerpApi
            if "serpapi" in self.available_backends:
                print("🔄 Chuyển sang tìm kiếm SerpApi")
                return self._search_serpapi(query)

    # Nếu Tavily không khả dụng, dùng SerpApi
    elif "serpapi" in self.available_backends:
        try:
            return self._search_serpapi(query)
        except Exception as e:
            print(f"⚠️ Tìm kiếm SerpApi thất bại: {e}")

    # Nếu không có nguồn nào khả dụng, nhắc nhở user cấu hình API
    return "❌ Không có nguồn tìm kiếm nào khả dụng, vui lòng cấu hình biến môi trường TAVILY_API_KEY hoặc SERPAPI_API_KEY"
````
Thiết kế này thể hiện tư tưởng cốt lõi của hệ thống có tính sẵn sàng cao (high availability): thông qua cơ chế giảm cấp (degradation mechanism), hệ thống có thể từ nguồn tìm kiếm tối ưu nhất, dần dần giảm cấp xuống phương án thay thế khả dụng. Khi tất cả các nguồn tìm kiếm đều không khả dụng, sẽ nhắc nhở rõ ràng để user cấu hình đúng API key.

(3) Chuẩn hóa định dạng kết quả tìm kiếm

Định dạng kết quả trả về của các search engine khác nhau là khác nhau, framework xử lý thông qua phương thức định dạng thống nhất:

````python
def _search_tavily(self, query: str) -> str:
    """Sử dụng Tavily để tìm kiếm"""
    response = self.tavily_client.search(
        query=query,
        search_depth="basic",
        include_answer=True,
        max_results=3
    )

    result = f"🎯 Kết quả tìm kiếm AI Tavily: {response.get('answer', '未找到直接答案')}\n\n"

    for i, item in enumerate(response.get('results', [])[:3], 1):
        result += f"[{i}] {item.get('title', '')}\n"
        result += f"    {item.get('content', '')[:200]}...\n"
        result += f"    Nguồn: {item.get('url', '')}\n\n"

    return result
````

Dựa trên tư tưởng thiết kế của framework, chúng ta có thể tạo ra công cụ tìm kiếm nâng cao của riêng mình. Lần này chúng ta sử dụng cách tiếp cận bằng class để minh họa một phương pháp triển khai khác, tạo file `my_advanced_search.py`:

```python
# my_advanced_search.py
import os
from typing import Optional, List, Dict, Any
from hello_agents import ToolRegistry

class MyAdvancedSearchTool:
    """
    Lớp công cụ tìm kiếm nâng cao tùy biến
    Minh họa pattern thiết kế tích hợp đa nguồn và lựa chọn thông minh
    """

    def __init__(self):
        self.name = "my_advanced_search"
        self.description = "Công cụ tìm kiếm thông minh, hỗ trợ nhiều nguồn tìm kiếm, tự động lựa chọn kết quả tốt nhất"
        self.search_sources = []
        self._setup_search_sources()

    def _setup_search_sources(self):
        """Thiết lập các nguồn tìm kiếm khả dụng"""
        # Kiểm tra tính khả dụng của Tavily
        if os.getenv("TAVILY_API_KEY"):
            try:
                from tavily import TavilyClient
                self.tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
                self.search_sources.append("tavily")
                print("✅ Nguồn tìm kiếm Tavily đã được kích hoạt")
            except ImportError:
                print("⚠️ Thư viện Tavily chưa được cài đặt")

        # Kiểm tra tính khả dụng của SerpApi
        if os.getenv("SERPAPI_API_KEY"):
            try:
                import serpapi
                self.search_sources.append("serpapi")
                print("✅ Nguồn tìm kiếm SerpApi đã được kích hoạt")
            except ImportError:
                print("⚠️ Thư viện SerpApi chưa được cài đặt")

        if self.search_sources:
            print(f"🔧 Nguồn tìm kiếm khả dụng: {', '.join(self.search_sources)}")
        else:
            print("⚠️ Không có nguồn tìm kiếm nào khả dụng, vui lòng cấu hình API key")

    def search(self, query: str) -> str:
        """Thực hiện tìm kiếm thông minh"""
        if not query.strip():
            return "❌ Lỗi: câu truy vấn tìm kiếm không được để trống"

        # Kiểm tra xem có nguồn tìm kiếm khả dụng không
        if not self.search_sources:
            return """❌ Không có nguồn tìm kiếm nào khả dụng, vui lòng cấu hình một trong các API key sau:

1. Tavily API: thiết lập biến môi trường TAVILY_API_KEY
   Địa chỉ đăng ký: https://tavily.com/

2. SerpAPI: thiết lập biến môi trường SERPAPI_API_KEY
   Địa chỉ đăng ký: https://serpapi.com/

Sau khi cấu hình hãy chạy lại chương trình."""

        print(f"🔍 Bắt đầu tìm kiếm thông minh: {query}")

        # Thử nhiều nguồn tìm kiếm, trả về kết quả tốt nhất
        for source in self.search_sources:
            try:
                if source == "tavily":
                    result = self._search_with_tavily(query)
                    if result and "未找到" not in result:
                        return f"📊 Kết quả tìm kiếm AI Tavily:\n\n{result}"

                elif source == "serpapi":
                    result = self._search_with_serpapi(query)
                    if result and "未找到" not in result:
                        return f"🌐 Kết quả tìm kiếm Google SerpApi:\n\n{result}"

            except Exception as e:
                print(f"⚠️ Tìm kiếm {source} thất bại: {e}")
                continue

        return "❌ Tất cả các nguồn tìm kiếm đều thất bại, vui lòng kiểm tra kết nối mạng và cấu hình API key"

    def _search_with_tavily(self, query: str) -> str:
        """Sử dụng Tavily để tìm kiếm"""
        response = self.tavily_client.search(query=query, max_results=3)

        if response.get('answer'):
            result = f"💡 Câu trả lời trực tiếp từ AI: {response['answer']}\n\n"
        else:
            result = ""

        result += "🔗 Kết quả liên quan:\n"
        for i, item in enumerate(response.get('results', [])[:3], 1):
            result += f"[{i}] {item.get('title', '')}\n"
            result += f"    {item.get('content', '')[:150]}...\n\n"

        return result

    def _search_with_serpapi(self, query: str) -> str:
        """Sử dụng SerpApi để tìm kiếm"""
        import serpapi

        search = serpapi.GoogleSearch({
            "q": query,
            "api_key": os.getenv("SERPAPI_API_KEY"),
            "num": 3
        })

        results = search.get_dict()

        result = "🔗 Kết quả tìm kiếm Google:\n"
        if "organic_results" in results:
            for i, res in enumerate(results["organic_results"][:3], 1):
                result += f"[{i}] {res.get('title', '')}\n"
                result += f"    {res.get('snippet', '')}\n\n"

        return result

def create_advanced_search_registry():
    """Tạo registry chứa công cụ tìm kiếm nâng cao"""
    registry = ToolRegistry()

    # Tạo instance công cụ tìm kiếm
    search_tool = MyAdvancedSearchTool()

    # Đăng ký phương thức của công cụ tìm kiếm như một function
    registry.register_function(
        name="advanced_search",
        description="Công cụ tìm kiếm nâng cao, tích hợp nhiều nguồn tìm kiếm Tavily và SerpAPI, cung cấp kết quả tìm kiếm toàn diện hơn",
        func=search_tool.search
    )

    return registry
```

Tiếp theo có thể test tool tự viết của chúng ta, tạo file `test_advanced_search.py`:

```python
# test_advanced_search.py
from dotenv import load_dotenv
from my_advanced_search import create_advanced_search_registry, MyAdvancedSearchTool

# Tải biến môi trường
load_dotenv()

def test_advanced_search():
    """Test công cụ tìm kiếm nâng cao"""

    # Tạo registry chứa công cụ tìm kiếm nâng cao
    registry = create_advanced_search_registry()

    print("🔍 Test công cụ tìm kiếm nâng cao\n")

    # Các query thử nghiệm
    test_queries = [
        "Lịch sử của ngôn ngữ lập trình Python",
        "Những phát triển mới nhất của trí tuệ nhân tạo",
        "Xu hướng công nghệ năm 2024"
    ]

    for i, query in enumerate(test_queries, 1):
        print(f"Test {i}: {query}")
        result = registry.execute_tool("advanced_search", query)
        print(f"Kết quả: {result}\n")
        print("-" * 60 + "\n")

def test_api_configuration():
    """Test kiểm tra cấu hình API"""
    print("🔧 Test kiểm tra cấu hình API:")

    # Trực tiếp tạo instance công cụ tìm kiếm
    search_tool = MyAdvancedSearchTool()

    # Nếu chưa cấu hình API, sẽ hiển thị nhắc nhở cấu hình
    result = search_tool.search("Thuật toán học máy")
    print(f"Kết quả tìm kiếm: {result}")

def test_with_agent():
    """Test tích hợp với Agent"""
    print("\n🤖 Test tích hợp với Agent:")
    print("Công cụ tìm kiếm nâng cao đã sẵn sàng, có thể tích hợp sử dụng với Agent")

    # Hiển thị mô tả tool
    registry = create_advanced_search_registry()
    tools_desc = registry.get_tools_description()
    print(f"Mô tả tool:\n{tools_desc}")

if __name__ == "__main__":
    test_advanced_search()
    test_api_configuration()
    test_with_agent()
```

Thông qua thực hành thiết kế công cụ tìm kiếm nâng cao này, chúng ta đã học được cách sử dụng class để xây dựng hệ thống tool phức tạp. So với cách dùng function, cách dùng class phù hợp hơn với các tool cần duy trì trạng thái (state) (như API client, thông tin cấu hình).

### 7.5.4 Các đặc tính nâng cao của hệ thống tool

Sau khi đã nắm vững việc phát triển tool cơ bản và tích hợp đa nguồn, hãy cùng khám phá các đặc tính nâng cao của hệ thống tool. Những đặc tính này giúp hệ thống tool hoạt động ổn định trong môi trường production phức tạp, và cung cấp khả năng mạnh mẽ hơn cho Agent.

(1) Cơ chế gọi tool theo chuỗi (chain)

Trong ứng dụng thực tế, Agent thường cần kết hợp sử dụng nhiều tool để hoàn thành nhiệm vụ phức tạp. Chúng ta có thể thiết kế một trình quản lý chuỗi tool (tool chain manager) để hỗ trợ kịch bản này, ở đây mượn khái niệm về đồ thị (graph) đã đề cập ở Chương 6:

```python
# tool_chain_manager.py
from typing import List, Dict, Any, Optional
from hello_agents import ToolRegistry

class ToolChain:
    """Chuỗi tool - hỗ trợ thực thi tuần tự nhiều tool"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
        self.steps: List[Dict[str, Any]] = []

    def add_step(self, tool_name: str, input_template: str, output_key: str = None):
        """
        Thêm bước thực thi tool

        Args:
            tool_name: tên tool
            input_template: template input, hỗ trợ thay thế biến
            output_key: tên key của kết quả output, dùng để tham chiếu ở các bước sau
        """
        self.steps.append({
            "tool_name": tool_name,
            "input_template": input_template,
            "output_key": output_key or f"step_{len(self.steps)}_result"
        })

    def execute(self, registry: ToolRegistry, initial_input: str, context: Dict[str, Any] = None) -> str:
        """Thực thi chuỗi tool"""
        context = context or {}
        context["input"] = initial_input

        print(f"🔗 Bắt đầu thực thi chuỗi tool: {self.name}")

        for i, step in enumerate(self.steps, 1):
            tool_name = step["tool_name"]
            input_template = step["input_template"]
            output_key = step["output_key"]

            # Thay thế biến trong template
            try:
                tool_input = input_template.format(**context)
            except KeyError as e:
                return f"❌ Thực thi chuỗi tool thất bại: không tìm thấy biến template {e}"

            print(f"  Bước {i}: sử dụng {tool_name} xử lý '{tool_input[:50]}...'")

            # Thực thi tool
            result = registry.execute_tool(tool_name, tool_input)
            context[output_key] = result

            print(f"  ✅ Bước {i} hoàn tất, độ dài kết quả: {len(result)} ký tự")

        # Trả về kết quả của bước cuối cùng
        final_result = context[self.steps[-1]["output_key"]]
        print(f"🎉 Chuỗi tool '{self.name}' thực thi hoàn tất")
        return final_result

class ToolChainManager:
    """Trình quản lý chuỗi tool"""

    def __init__(self, registry: ToolRegistry):
        self.registry = registry
        self.chains: Dict[str, ToolChain] = {}

    def register_chain(self, chain: ToolChain):
        """Đăng ký chuỗi tool"""
        self.chains[chain.name] = chain
        print(f"✅ Chuỗi tool '{chain.name}' đã được đăng ký")

    def execute_chain(self, chain_name: str, input_data: str, context: Dict[str, Any] = None) -> str:
        """Thực thi chuỗi tool được chỉ định"""
        if chain_name not in self.chains:
            return f"❌ Chuỗi tool '{chain_name}' không tồn tại"

        chain = self.chains[chain_name]
        return chain.execute(self.registry, input_data, context)

    def list_chains(self) -> List[str]:
        """Liệt kê tất cả chuỗi tool"""
        return list(self.chains.keys())

# Ví dụ sử dụng
def create_research_chain() -> ToolChain:
    """Tạo một chuỗi tool nghiên cứu: tìm kiếm -> tính toán -> tổng hợp"""
    chain = ToolChain(
        name="research_and_calculate",
        description="Tìm kiếm thông tin và thực hiện tính toán liên quan"
    )

    # Bước 1: tìm kiếm thông tin
    chain.add_step(
        tool_name="search",
        input_template="{input}",
        output_key="search_result"
    )

    # Bước 2: tính toán dựa trên kết quả tìm kiếm (nếu cần)
    chain.add_step(
        tool_name="my_calculator",
        input_template="Hãy tính toán các giá trị liên quan dựa trên thông tin sau: {search_result}",
        output_key="calculation_result"
    )

    return chain
```

(2) Hỗ trợ thực thi tool bất đồng bộ (async)

Đối với các thao tác tool tốn thời gian, chúng ta có thể cung cấp khả năng thực thi bất đồng bộ:

```python
# async_tool_executor.py
import asyncio
import concurrent.futures
from typing import Dict, Any, List, Callable
from hello_agents import ToolRegistry

class AsyncToolExecutor:
    """Bộ thực thi tool bất đồng bộ"""

    def __init__(self, registry: ToolRegistry, max_workers: int = 4):
        self.registry = registry
        self.executor = concurrent.futures.ThreadPoolExecutor(max_workers=max_workers)

    async def execute_tool_async(self, tool_name: str, input_data: str) -> str:
        """Thực thi bất đồng bộ một tool đơn"""
        loop = asyncio.get_event_loop()

        def _execute():
            return self.registry.execute_tool(tool_name, input_data)

        result = await loop.run_in_executor(self.executor, _execute)
        return result

    async def execute_tools_parallel(self, tasks: List[Dict[str, str]]) -> List[str]:
        """Thực thi song song nhiều tool"""
        print(f"🚀 Bắt đầu thực thi song song {len(tasks)} tác vụ tool")

        # Tạo các task bất đồng bộ
        async_tasks = []
        for task in tasks:
            tool_name = task["tool_name"]
            input_data = task["input_data"]
            async_task = self.execute_tool_async(tool_name, input_data)
            async_tasks.append(async_task)

        # Chờ tất cả task hoàn thành
        results = await asyncio.gather(*async_tasks)

        print(f"✅ Tất cả các tác vụ tool đã thực thi xong")
        return results

    def __del__(self):
        """Dọn dẹp tài nguyên"""
        if hasattr(self, 'executor'):
            self.executor.shutdown(wait=True)

# Ví dụ sử dụng
async def test_parallel_execution():
    """Test thực thi tool song song"""
    from hello_agents import ToolRegistry

    registry = ToolRegistry()
    # Giả sử đã đăng ký tool tìm kiếm và tính toán

    executor = AsyncToolExecutor(registry)

    # Định nghĩa các task song song
    tasks = [
        {"tool_name": "search", "input_data": "Lập trình Python"},
        {"tool_name": "search", "input_data": "Học máy"},
        {"tool_name": "my_calculator", "input_data": "2 + 2"},
        {"tool_name": "my_calculator", "input_data": "sqrt(16)"},
    ]

    # Thực thi song song
    results = await executor.execute_tools_parallel(tasks)

    for i, result in enumerate(results):
        print(f"Kết quả tác vụ {i+1}: {result[:100]}...")
```

Dựa trên kinh nghiệm thiết kế và triển khai ở trên, chúng ta có thể tổng kết được triết lý cốt lõi của việc phát triển hệ thống tool: ở tầng thiết kế, mỗi tool đều nên tuân theo nguyên tắc đơn trách nhiệm (single responsibility), tập trung vào chức năng cụ thể trong khi vẫn giữ tính thống nhất của interface, và coi việc xử lý exception hoàn thiện cùng việc xác thực input ưu tiên bảo mật (security-first) là yêu cầu cơ bản. Về mặt tối ưu hóa hiệu năng, tận dụng thực thi bất đồng bộ để nâng cao khả năng xử lý đồng thời, đồng thời quản lý hợp lý các kết nối bên ngoài và tài nguyên hệ thống.



## 7.6 Tổng kết chương

Trước khi tổng kết chính thức, chúng tôi muốn thông báo một tin tốt: đối với tất cả các phương thức và chức năng được triển khai trong chương này, đều đã cung cấp đầy đủ case test trong Repository GitHub. Bạn có thể truy cập [link này](https://github.com/datawhalechina/hello-agents/tree/main/code/chapter7) để xem và chạy các đoạn code test này. Thư mục đó bao gồm phần minh họa của bốn paradigm Agent, test tích hợp hệ thống tool, ví dụ sử dụng các tính năng nâng cao, cũng như trải nghiệm Agent dạng tương tác. Nếu bạn muốn kiểm chứng xem triển khai của mình có chính xác hay không, hoặc muốn hiểu sâu hơn về cách sử dụng thực tế của framework, những case test này sẽ là tài liệu tham khảo có giá trị.

Nhìn lại chương này, chúng ta đã hoàn thành một nhiệm vụ đầy thách thức: từng bước xây dựng một Agent Framework cơ bản — HelloAgents. Quá trình này luôn tuân theo nguyên tắc cốt lõi "phân tầng tách rời, đơn trách nhiệm, interface thống nhất".

Trong việc triển khai cụ thể của framework, chúng ta đã một lần nữa triển khai bốn paradigm Agent kinh điển. Từ mô hình hội thoại cơ bản của SimpleAgent, đến sự kết hợp suy luận và hành động của ReActAgent; từ sự tự phản tư và tối ưu hóa lặp lại của ReflectionAgent, đến sự phân rã kế hoạch và thực thi từng bước của PlanAndSolveAgent. Còn hệ thống tool, với vai trò là cốt lõi mở rộng năng lực của Agent, thì quá trình xây dựng của nó là một thực hành kỹ thuật hoàn chỉnh.

Quan trọng hơn, việc xây dựng Chương 7 không phải là điểm kết thúc, mà là cung cấp nền tảng kỹ thuật cần thiết cho việc học tập chuyên sâu hơn sau này. Ngay từ đầu quá trình thiết kế, chúng tôi đã cân nhắc đầy đủ về khả năng mở rộng của nội dung sau này, dành sẵn các interface và điểm mở rộng cần thiết cho việc triển khai các tính năng nâng cao. Interface LLM thống nhất, hệ thống message chuẩn hóa, cơ chế đăng ký tool mà chúng ta đã xây dựng, cùng nhau tạo thành một nền tảng kỹ thuật hoàn chỉnh. Điều này giúp chúng ta, ở các chương sau, có thể tự tin hơn khi học các chủ đề nâng cao: hệ thống Memory và RAG ở Chương 8 sẽ mở rộng ranh giới năng lực của Agent dựa trên nền tảng này; Context Engineering ở Chương 9 sẽ đi sâu vào cơ chế xử lý message mà chúng ta đã xây dựng; Agent Protocol ở Chương 10 thì cần mở rộng thêm tool mới.

Tiếp theo, chúng ta sẽ cùng nhau khám phá cách thêm hệ thống RAG và cơ chế Memory vào framework, hãy cùng đón chờ Chương 8!


## Bài tập

1. Chương này đã xây dựng framework `HelloAgents`, và trình bày "tại sao cần tự xây dựng Agent Framework". Hãy phân tích:

   - Ở mục 7.1.1 đã đề cập đến bốn hạn chế chính của các framework phổ biến hiện nay. Kết hợp với kinh nghiệm thực tế sử dụng một framework nào đó mà bạn đã dùng trong [bài tập Chương 6](../chapter6/第六章%20框架开发实践_vi.md#习题) hoặc dự án thực tế, hãy giải thích những vấn đề này ảnh hưởng như thế nào đến hiệu suất phát triển.
   - `HelloAgents` đề xuất triết lý thiết kế "mọi thứ đều là tool", trừu tượng hóa các module như `Memory`, `RAG`, `MCP` thành tool. Thiết kế này có ưu điểm gì? Có tồn tại hạn chế nào không? Hãy cho ví dụ minh họa.
   - So sánh code Agent được triển khai từ đầu ở Chương 4 và triển khai được framework hóa ở chương này, việc framework hóa mang lại những cải tiến cụ thể nào? Nếu để bạn thiết kế một framework, bạn sẽ ưu tiên xem xét những nguyên tắc thiết kế nào?

2. Ở mục 7.2, chúng ta đã mở rộng `HelloAgentsLLM` để hỗ trợ nhiều nhà cung cấp model và gọi model local.

   > <strong>Gợi ý</strong>: đây là một bài tập thực hành, nên thực sự thao tác

   - Tham khảo ví dụ ở mục 7.2.1, hãy thử bổ sung hỗ trợ cho một nhà cung cấp model mới (như `Gemini`, `Anthropic`, `Kimi`) cho `HelloAgentsLLM`. Yêu cầu triển khai theo cách kế thừa, và có khả năng tự động detect biến môi trường của nhà cung cấp đó.
   - Ở mục 7.2.3 đã giới thiệu ba mức ưu tiên của cơ chế tự động detect. Hãy phân tích: nếu cùng lúc thiết lập `OPENAI_API_KEY` và `LLM_BASE_URL="http://localhost:11434/v1"`, framework cuối cùng sẽ lựa chọn nhà cung cấp nào? Thiết kế mức ưu tiên này có hợp lý hay không?
   - Ngoài `VLLM` và `Ollama` được giới thiệu trong chương này, còn có các phương án triển khai model local khác như `SGLang`. Hãy tìm hiểu trước thông tin cơ bản và đặc điểm của `SGLang`, sau đó so sánh ưu nhược điểm của `VLLM`, `SGLang` và `Ollama` về mặt tính dễ sử dụng, mức sử dụng tài nguyên, tốc độ inference, độ chính xác inference.

3. Ở mục 7.3, chúng ta đã triển khai lớp `Message`, lớp `Config` và lớp cơ sở `Agent`. Hãy phân tích:

   - Lớp `Message` sử dụng `BaseModel` của `Pydantic` để xác thực dữ liệu. Thiết kế này có những ưu điểm gì trong ứng dụng thực tế?
   - Lớp cơ sở `Agent` định nghĩa hai phương thức `run` và `_execute`, trong đó `run` là interface công khai, `_execute` là phương thức trừu tượng. Pattern thiết kế này được gọi là gì? Có lợi ích gì?
   - Trong lớp `Config`, chúng ta đã sử dụng singleton pattern. Hãy giải thích singleton pattern là gì, tại sao việc quản lý config cần sử dụng singleton pattern? Nếu không sử dụng singleton sẽ dẫn đến vấn đề gì?

4. Ở mục 7.4, chúng ta đã thực hành việc framework hóa bốn paradigm `Agent`.

   > <strong>Gợi ý</strong>: đây là một bài tập thực hành, nên thực sự thao tác

   - So sánh `ReActAgent` được triển khai từ đầu ở Chương 4 và `ReActAgent` được framework hóa ở chương này, hãy liệt kê 3 điểm cải tiến cụ thể, và giải thích những cải tiến này nâng cao khả năng bảo trì và mở rộng của code như thế nào.
   - `ReflectionAgent` triển khai vòng lặp "thực thi - phản tư - tối ưu hóa". Hãy mở rộng triển khai này, thêm cơ chế "chấm điểm chất lượng": sau mỗi lần phản tư, để `LLM` chấm điểm cho output của phiên bản hiện tại, chỉ khi điểm số thấp hơn ngưỡng (threshold) mới tiếp tục tối ưu hóa, ngược lại thì kết thúc sớm.
   - Hãy thiết kế và triển khai một paradigm `Agent` mới là `Tree-of-Thought Agent`, yêu cầu kế thừa lớp cơ sở `Agent`, nó có khả năng sinh ra nhiều đường suy nghĩ (thinking path) khả dĩ ở mỗi bước, sau đó lựa chọn đường đi tối ưu để tiếp tục.

5. Ở mục 7.5, chúng ta đã xây dựng hệ thống tool. Hãy suy nghĩ về các câu hỏi sau:

   - Lớp `BaseTool` định nghĩa phương thức trừu tượng `execute`, tất cả các tool đều phải triển khai phương thức này. Hãy giải thích tại sao phải bắt buộc tất cả các tool triển khai một interface thống nhất? Nếu một tool nào đó cần trả về nhiều giá trị (như tool tìm kiếm trả về title, tóm tắt, link), nên thiết kế như thế nào?
   - Ở mục 7.5.3 đã triển khai chuỗi tool (`ToolChain`). Hãy thiết kế một kịch bản ứng dụng thực tế, cần nối tiếp ít nhất 3 tool, và vẽ sơ đồ luồng thực thi của chuỗi tool.
   - Bộ thực thi tool bất đồng bộ (`AsyncToolExecutor`) sử dụng thread pool để thực thi tool song song. Hãy phân tích: trong trường hợp nào thì việc thực thi tool song song mang lại cải thiện hiệu năng?

6. Khả năng mở rộng của framework là một yếu tố quan trọng cần cân nhắc trong thiết kế. Bây giờ bạn cần mở rộng framework `HelloAgents`, triển khai một số tính năng và đặc tính mới thú vị cho nó.

   - Trước tiên hãy thêm chức năng "output dạng streaming" cho `HelloAgents`, giúp `Agent` có thể trả về kết quả trung gian theo thời gian thực khi sinh response (tương tự hiệu ứng gõ chữ của giao diện người dùng `ChatGPT`). Hãy thiết kế phương án triển khai cho chức năng này, giải thích cần sửa đổi những lớp và phương thức nào.
   - Sau đó hãy thêm chức năng "quản lý hội thoại nhiều vòng" cho framework, có thể tự động quản lý lịch sử hội thoại, hỗ trợ phân nhánh (branch) và quay lui (backtrack) hội thoại, bạn sẽ thiết kế như thế nào? Cần thêm những lớp nào? Làm thế nào để tích hợp với hệ thống `Message` hiện có?
   - Cuối cùng hãy thiết kế một "hệ thống plugin" cho `HelloAgents`, cho phép nhà phát triển bên thứ ba mở rộng chức năng của framework thông qua plugin (như thêm loại `Agent` mới, loại tool mới, v.v.), mà không cần sửa đổi code cốt lõi của framework. Yêu cầu vẽ sơ đồ kiến trúc của hệ thống plugin và giải thích các interface then chốt.
