# Chương 12 Đánh giá hiệu năng Agent

Trong các chương trước, chúng ta đã xây dựng các chức năng cốt lõi của framework HelloAgents, hiện thực nhiều paradigm Agent khác nhau, hệ thống tool, cơ chế memory và huấn luyện reinforcement learning, v.v. Khi xây dựng hệ thống Agent, chúng ta còn cần giải quyết một vấn đề cốt lõi: <strong>làm thế nào để đánh giá khách quan hiệu năng của Agent?</strong> Cụ thể, chúng ta cần trả lời các câu hỏi sau:

1. Agent có sở hữu năng lực như kỳ vọng hay không?
2. Hiệu suất trên các task khác nhau như thế nào?
3. So với các Agent khác thì ở mức độ nào?

Chương này sẽ bổ sung cho HelloAgents <strong>hệ thống đánh giá hiệu năng (Evaluation System)</strong>. Chúng ta sẽ tìm hiểu sâu nền tảng lý thuyết của việc đánh giá Agent, và hiện thực các công cụ đánh giá.

## 12.1 Nền tảng đánh giá Agent

### 12.1.1 Vì sao cần đánh giá Agent

SimpleAgent hiện tại của chúng ta đã sở hữu năng lực reasoning và gọi tool mạnh mẽ. Hãy cùng xem một kịch bản sử dụng điển hình:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import SearchTool

# Tạo LLM và Agent
llm = HelloAgentsLLM()

# Tạo một system prompt nhấn mạnh việc sử dụng tool
system_prompt = """你是一个AI助手，可以使用搜索工具来获取最新信息。

当需要搜索信息时，请使用以下格式：
[TOOL_CALL:search:搜索关键词]

例如：
- [TOOL_CALL:search:最新AI新闻]
- [TOOL_CALL:search:Python编程教程]

请在回答问题前先使用搜索工具获取最新信息。"""

agent = SimpleAgent(name="AI助手", llm=llm, system_prompt=system_prompt)

# Thêm search tool
agent.add_tool(SearchTool())

# Ví dụ: dùng search tool để trả lời câu hỏi
response = agent.run("最新的AI技术发展趋势是什么？")
print(f"\n回答：{response}")
```

Agent này hoạt động bình thường, nhưng chúng ta đối mặt một vấn đề cốt lõi: làm sao đánh giá khách quan hiệu năng của nó? Khi tối ưu prompt hoặc đổi LLM model, làm sao biết liệu có thực sự cải thiện hay không? Trước khi deploy lên môi trường production, làm sao đảm bảo độ tin cậy của Agent? Những vấn đề này đều cần được giải quyết thông qua đánh giá có hệ thống.

Giá trị cốt lõi của việc đánh giá Agent nằm ở việc cung cấp phương pháp chuẩn hóa để đo lường năng lực của Agent. Thông qua đánh giá, chúng ta có thể lượng hóa hiệu suất của Agent bằng các chỉ số cụ thể, so sánh khách quan ưu nhược điểm giữa các phương án thiết kế khác nhau, kịp thời phát hiện điểm yếu của Agent trong các tình huống cụ thể, và chứng minh độ tin cậy của Agent cho người dùng.

Khác với kiểm thử phần mềm truyền thống, việc đánh giá Agent gặp phải những thách thức riêng biệt. Đầu tiên là tính bất định của output, cùng một câu hỏi có thể có nhiều đáp án đúng, rất khó dùng tiêu chí đúng/sai đơn giản để phán xét. Thứ hai là sự đa dạng của tiêu chuẩn đánh giá, các task khác nhau cần phương pháp đánh giá khác nhau, việc gọi tool cần kiểm tra function signature, task hỏi đáp cần đánh giá độ tương đồng ngữ nghĩa. Cuối cùng là chi phí đánh giá cao, mỗi lần đánh giá cần số lượng lớn API call, chi phí có thể lên tới hàng trăm nghìn đồng (vài trăm tệ) hoặc hơn.

Để đối phó với những thách thức này, giới học thuật và công nghiệp đã đề xuất nhiều <strong>chuẩn đánh giá (Benchmark)</strong> chuẩn hóa. Các benchmark này cung cấp bộ dữ liệu, chỉ số đánh giá và phương pháp chấm điểm thống nhất, giúp chúng ta có thể đánh giá và so sánh các hệ thống Agent khác nhau theo cùng một tiêu chuẩn.

### 12.1.2 Tổng quan các benchmark đánh giá chính

Lĩnh vực đánh giá Agent đã xuất hiện nhiều benchmark có sức ảnh hưởng lớn. Dưới đây giới thiệu một số benchmark và chỉ số đánh giá phổ biến:

<strong>(1) Đánh giá năng lực gọi tool</strong>

Gọi tool (Tool Calling) là một trong những năng lực cốt lõi của Agent. Agent cần hiểu ý định người dùng, lựa chọn tool phù hợp, và xây dựng function call chính xác. Các benchmark liên quan bao gồm:

- <strong>BFCL (Berkeley Function Calling Leaderboard)</strong><sup>[1]</sup>: do UC Berkeley đưa ra, bao gồm hơn 1120 mẫu test, bao phủ bốn category simple, multiple, parallel, irrelevance, sử dụng thuật toán AST matching để đánh giá, quy mô dataset vừa phải, cộng đồng hoạt động tích cực.
- <strong>ToolBench</strong><sup>[2]</sup>: do Đại học Thanh Hoa đưa ra, bao gồm hơn 16000 kịch bản gọi API thực tế, bao phủ các kịch bản sử dụng tool phức tạp trong thế giới thực.
- <strong>API-Bank</strong><sup>[3]</sup>: do Microsoft Research đưa ra, bao gồm 53 API tool thông dụng, tập trung đánh giá khả năng hiểu tài liệu API và gọi API của Agent.

<strong>(2) Đánh giá năng lực tổng quát</strong>

Đánh giá hiệu suất tổng hợp của Agent trong các task thực tế, bao gồm multi-step reasoning, vận dụng tri thức, hiểu đa phương thức (multimodal), v.v.:

- <strong>GAIA (General AI Assistants)</strong><sup>[4]</sup>: do Meta AI và Hugging Face phối hợp đưa ra, bao gồm 466 câu hỏi thực tế, chia thành ba mức độ khó Level 1/2/3, đánh giá năng lực multi-step reasoning, sử dụng tool, xử lý file, duyệt web, sử dụng thuật toán Quasi Exact Match, task mang tính thực tế và tổng hợp cao.
- <strong>AgentBench</strong><sup>[5]</sup>: do Đại học Thanh Hoa đưa ra, bao gồm 8 task thuộc các lĩnh vực khác nhau, đánh giá toàn diện năng lực tổng quát của Agent.
- <strong>WebArena</strong><sup>[6]</sup>: do CMU đưa ra, đánh giá khả năng hoàn thành task và tương tác web của Agent trong môi trường web thực tế.

<strong>(3) Đánh giá hợp tác đa Agent</strong>

Đánh giá năng lực phối hợp làm việc của nhiều Agent:

- <strong>ChatEval</strong><sup>[7]</sup>: đánh giá chất lượng hệ thống đối thoại đa Agent.
- <strong>SOTOPIA</strong><sup>[8]</sup>: đánh giá năng lực tương tác của Agent trong kịch bản xã hội.
- <strong>Kịch bản hợp tác tùy chỉnh</strong>: các task đánh giá được thiết kế theo từng kịch bản ứng dụng cụ thể.

<strong>(4) Các chỉ số đánh giá thông dụng</strong>

Các benchmark khác nhau sử dụng các chỉ số đánh giá khác nhau, phổ biến gồm:

- <strong>Chỉ số độ chính xác</strong>: Accuracy (độ chính xác), Exact Match (khớp chính xác), F1 Score (điểm F1), dùng để đo tính đúng đắn của đáp án.
- <strong>Chỉ số hiệu suất</strong>: Response Time (thời gian phản hồi), Token Usage (lượng Token sử dụng), dùng để đo hiệu suất thực thi.
- <strong>Chỉ số độ bền vững</strong>: Error Rate (tỷ lệ lỗi), Failure Recovery (khả năng phục hồi sau lỗi), dùng để đo khả năng chịu lỗi.
- <strong>Chỉ số hợp tác</strong>: Communication Efficiency (hiệu quả giao tiếp), Task Completion (mức độ hoàn thành task), dùng để đo hiệu quả hợp tác.

### 12.1.3 Thiết kế hệ thống đánh giá của HelloAgents

Xét đến đường cong học tập và tính thực dụng, chương này sẽ tập trung giới thiệu các kịch bản đánh giá sau:

1. <strong>BFCL</strong>: đánh giá năng lực gọi tool
   - Lý do lựa chọn: quy mô dataset vừa phải, chỉ số đánh giá rõ ràng, cộng đồng hoạt động tích cực
   - Kịch bản áp dụng: đánh giá độ chính xác gọi function của Agent

2. <strong>GAIA</strong>: đánh giá năng lực của AI assistant tổng quát
   - Lý do lựa chọn: task mang tính thực tế, phân cấp độ khó, tính tổng hợp cao
   - Kịch bản áp dụng: đánh giá năng lực giải quyết vấn đề tổng hợp của Agent

3. <strong>Đánh giá chất lượng sinh dữ liệu (Data Generation)</strong>: đánh giá chất lượng dữ liệu do LLM sinh ra
   - Lý do lựa chọn: thông qua case này có thể trải nghiệm đầy đủ cách dùng Agent để tạo dữ liệu, và demo hoàn chỉnh việc đánh giá dữ liệu.
   - Kịch bản áp dụng: đánh giá chất lượng dữ liệu huấn luyện, dữ liệu kiểm thử được sinh ra
   - Phương pháp đánh giá: LLM Judge, Win Rate, xác minh thủ công (human verification)

Thông qua ba kịch bản đánh giá này, chúng ta sẽ xây dựng một hệ thống đánh giá hoàn chỉnh, Hình 12.1 minh họa ý tưởng xây dựng hệ thống đánh giá của chúng ta.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-1.png" alt="" width="85%"/>
  <p>Hình 12.1 Sơ đồ kiến trúc hệ thống đánh giá của HelloAgents</p>
</div>



### 12.1.4 Mục tiêu học tập và trải nghiệm nhanh của chương này

Hãy cùng xem trước nội dung học tập của Chương 12:

```
hello_agents/
├── evaluation/                         # Module đánh giá
│   └── benchmarks/                     # Hiện thực các benchmark đánh giá
│       ├── bfcl/                       # Hiện thực đánh giá BFCL
│       │   ├── dataset.py              # Bộ nạp dataset BFCL
│       │   ├── evaluator.py            # Evaluator BFCL (AST matching)
│       │   ├── metrics.py              # Chỉ số chuyên dụng cho BFCL
│       │   └── ast_matcher.py          # Thuật toán AST matching
│       ├── gaia/                       # Hiện thực đánh giá GAIA
│       │   ├── dataset.py              # Bộ nạp dataset GAIA
│       │   ├── evaluator.py            # Evaluator GAIA (Quasi Exact Match)
│       │   ├── metrics.py              # Chỉ số chuyên dụng cho GAIA
│       │   └── quasi_exact_match.py    # Thuật toán Quasi Exact Match
│       └── data_generation/            # Hiện thực đánh giá sinh dữ liệu
│           ├── dataset.py              # Bộ nạp dataset AIME
│           ├── llm_judge.py            # Evaluator LLM Judge
│           └── win_rate.py             # Evaluator Win Rate
└── tools/builtin/                      # Module tool có sẵn
    ├── bfcl_evaluation_tool.py         # Tool đánh giá BFCL
    ├── gaia_evaluation_tool.py         # Tool đánh giá GAIA
    ├── llm_judge_tool.py               # Tool LLM Judge
    └── win_rate_tool.py                # Tool Win Rate
```

Đối với nội dung chương này, mục tiêu học tập là nắm vững khả năng áp dụng các công cụ đánh giá. Hãy chuẩn bị môi trường phát triển trước:

```bash
# Cài đặt framework HelloAgents (phiên bản Chương 12)
pip install "hello-agents[evaluation]==0.2.7"

# Thiết lập biến môi trường
export HF_TOKEN="your_huggingface_token"     # Dùng cho dataset GAIA (sẽ có bước thiết lập ở phần sau)

# Vì package chính thức `bfcl-eval` bắt buộc numpy<=2.0.0, xung đột với phiên bản dependency chính của HelloAgents, nên cần cài đặt riêng
pip install "numpy==1.26.4" bfcl-eval
```

Trong các phần tiếp theo, chúng ta sẽ tìm hiểu sâu cách sử dụng chi tiết và giới thiệu từng phương pháp đánh giá.

## 12.2 BFCL: Đánh giá năng lực gọi tool

### 12.2.1 Giới thiệu benchmark BFCL

BFCL (Berkeley Function Calling Leaderboard) là benchmark đánh giá năng lực function calling do Đại học California, Berkeley đưa ra<sup>[1]</sup>. Trong hệ thống Agent, gọi tool (Tool Calling) là một trong những năng lực cốt lõi. Agent cần hoàn thành các nhiệm vụ sau:

1. <strong>Hiểu yêu cầu task</strong>: trích xuất thông tin quan trọng từ mô tả ngôn ngữ tự nhiên của người dùng
2. <strong>Lựa chọn tool phù hợp</strong>: chọn tool phù hợp nhất từ tập các tool khả dụng
3. <strong>Xây dựng function call</strong>: điền đúng tên function và tham số
4. <strong>Xử lý kịch bản phức tạp</strong>: hỗ trợ gọi nhiều function, gọi song song (parallel) và các kịch bản nâng cao khác

Benchmark BFCL bao gồm bốn category đánh giá với độ khó tăng dần. Bắt đầu từ gọi một function đơn giản nhất (Simple), dần dần tăng lên kịch bản cần gọi nhiều function (Multiple), tiếp đến kịch bản phức tạp cần gọi song song nhiều function (Parallel), cuối cùng là kịch bản cần phán đoán có cần gọi function hay không (Irrelevance). Bốn category này bao phủ các kịch bản gọi tool khác nhau mà Agent có thể gặp phải trong ứng dụng thực tế, như Bảng 12.1 thể hiện:

<div align="center">
  <p>Bảng 12.1 Bốn category đánh giá trong benchmark BFCL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-1.png" alt="" width="85%"/>
</div>
Quy trình đánh giá của BFCL tuân theo quy trình benchmark test chuẩn: đầu tiên nạp dataset và chọn category đánh giá, sau đó chạy Agent để lấy kết quả dự đoán, tiếp theo parse kết quả dự đoán thành cây cú pháp trừu tượng (Abstract Syntax Tree - AST), cuối cùng dùng thuật toán AST matching để phán đoán dự đoán có đúng hay không. Toàn bộ quy trình sẽ duyệt qua tất cả mẫu test, cuối cùng tính ra các chỉ số đánh giá như accuracy và sinh báo cáo đánh giá. Quy trình đánh giá hoàn chỉnh như Hình 12.2 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-2.png" alt="" width="85%"/>
  <p>Hình 12.2 Sơ đồ quy trình đánh giá BFCL</p>
</div>
<strong>(1) Cấu trúc dataset BFCL</strong>

Dataset BFCL sử dụng định dạng JSON, mỗi mẫu test bao gồm các field sau:

```json
{
  "id": "simple_001",
  "question": "What's the weather like in Beijing today?",
  "function": [
    {
      "name": "get_weather",
      "description": "Get the current weather for a location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city name"
          }
        },
        "required": ["location"]
      }
    }
  ],
  "ground_truth": [
    {
      "name": "get_weather",
      "arguments": {
        "location": "Beijing"
      }
    }
  ]
}
```

<strong>Giải thích các field quan trọng:</strong>

- `question`: yêu cầu bằng ngôn ngữ tự nhiên của người dùng
- `function`: danh sách function khả dụng (bao gồm function signature và mô tả)
- `ground_truth`: đáp án chuẩn (function call kỳ vọng)

<strong>(2) Giải thích AST matching</strong>

BFCL sử dụng <strong>AST matching (Abstract Syntax Tree Matching)</strong> làm thuật toán đánh giá cốt lõi, vì vậy dưới đây chúng ta sẽ tìm hiểu chiến lược đánh giá.

BFCL sử dụng cây cú pháp trừu tượng (AST) để so khớp thông minh, thay vì so khớp chuỗi (string matching) đơn giản. Ý tưởng cốt lõi của AST matching là: <strong>parse function call thành cây cú pháp, sau đó so sánh cấu trúc cây và giá trị các node</strong>.

Cho trước function call dự đoán $P$ và đáp án chuẩn $G$, hàm AST matching được định nghĩa như sau:

$$
\text{AST\_Match}(P, G) = \begin{cases}
1 & \text{if } \text{AST}(P) \equiv \text{AST}(G) \\
0 & \text{otherwise}
\end{cases}
$$

trong đó $\text{AST}(x)$ biểu thị việc parse function call thành cây cú pháp trừu tượng, $\equiv$ biểu thị tính tương đương của cây cú pháp.

Hai cây cú pháp tương đương cần thỏa mãn ba điều kiện cốt lõi: tên function phải hoàn toàn trùng khớp (khớp chính xác), tập hợp cặp key-value của tham số bằng nhau (bỏ qua thứ tự), và giá trị của mỗi tham số tương đương về mặt ngữ nghĩa (ví dụ `2+3` tương đương với `5`). Trong quá trình so khớp cụ thể, việc so khớp tên function yêu cầu khớp chuỗi chính xác, ví dụ `get_weather` và `get_temperature` được xem là hai function khác nhau. Việc so khớp tham số thì dùng AST để so sánh thông minh, cho phép thứ tự tham số khác nhau (`f(a=1, b=2)` tương đương `f(b=2, a=1)`), cho phép biểu thức tương đương (`f(x=2+3)` tương đương `f(x=5)`), cũng cho phép cách biểu diễn chuỗi khác nhau (`f(s="hello")` tương đương `f(s='hello')`). Đối với kịch bản gọi nhiều function, thuật toán matching yêu cầu gọi cùng số lượng function, mỗi function call đều phải khớp, nhưng thứ tự gọi có thể khác nhau (dùng set matching).

<strong>Ví dụ AST matching:</strong>

```python
# Ví dụ 1: thứ tự tham số khác nhau (matching thành công)
预测: get_weather(city="Beijing", unit="celsius")
标准: get_weather(unit="celsius", city="Beijing")
结果: ✅ 匹配成功

# Ví dụ 2: biểu thức tương đương (matching thành công)
预测: calculate(x=2+3)
标准: calculate(x=5)
结果: ✅ 匹配成功

# Ví dụ 3: tên function sai (matching thất bại)
预测: get_temperature(city="Beijing")
标准: get_weather(city="Beijing")
结果: ❌ 匹配失败

# Ví dụ 4: giá trị tham số sai (matching thất bại)
预测: get_weather(city="Shanghai")
标准: get_weather(city="Beijing")
结果: ❌ 匹配失败
```

<strong>(3) Chỉ số đánh giá BFCL</strong>

BFCL sử dụng các chỉ số sau để đánh giá hiệu năng Agent:

<strong>1. Độ chính xác (Accuracy)</strong>

Accuracy là chỉ số cốt lõi nhất, được định nghĩa là tỷ lệ mẫu test khớp AST thành công:

$$
\text{Accuracy} = \frac{1}{N} \sum_{i=1}^{N} \text{AST\_Match}(P_i, G_i)
$$

trong đó:
- $N$ là tổng số mẫu test
- $P_i$ là kết quả dự đoán của mẫu thứ $i$
- $G_i$ là đáp án chuẩn của mẫu thứ $i$
- $\text{AST\_Match}(P_i, G_i) \in \{0, 1\}$ là hàm AST matching

<strong>2. Tỷ lệ khớp AST (AST Match Rate)</strong>

Giống với Accuracy, nhấn mạnh việc sử dụng thuật toán AST matching:

$$
\text{AST Match Rate} = \text{Accuracy}
$$

<strong>3. Accuracy theo category (Category-wise Accuracy)</strong>

Với mỗi category $c \in \{\text{simple}, \text{multiple}, \text{parallel}, \ldots\}$, tính Accuracy của category đó:

$$
\text{Accuracy}_c = \frac{1}{|D_c|} \sum_{i \in D_c} \text{AST\_Match}(P_i, G_i)
$$

trong đó $D_c$ là tập mẫu của category $c$, $|D_c|$ là số mẫu của category đó.

<strong>4. Accuracy có trọng số (Weighted Accuracy)</strong>

Xét đến trọng số độ khó của các category khác nhau:

$$
\text{Weighted Accuracy} = \sum_{c} w_c \cdot \text{Accuracy}_c
$$

trong đó $w_c$ là trọng số của category $c$, thỏa mãn $\sum_c w_c = 1$.

<strong>5. Tỷ lệ lỗi (Error Rate)</strong>

Tỷ lệ mẫu không gọi đúng function:

$$
\text{Error Rate} = 1 - \text{Accuracy} = \frac{1}{N} \sum_{i=1}^{N} (1 - \text{AST\_Match}(P_i, G_i))
$$

<strong>Giải thích chỉ số:</strong>

- <strong>Accuracy = 1.0</strong>: tất cả mẫu đều hoàn toàn đúng
- <strong>Accuracy = 0.8</strong>: 80% mẫu đúng, 20% mẫu sai
- <strong>Accuracy = 0.0</strong>: tất cả mẫu đều sai

<strong>Ví dụ Accuracy theo category:</strong>

```python
# Giả sử kết quả đánh giá
simple_accuracy = 0.95      # Category Simple: 95% đúng
multiple_accuracy = 0.82    # Category Multiple: 82% đúng
parallel_accuracy = 0.68    # Category Parallel: 68% đúng

# Weighted Accuracy (giả sử trọng số bằng nhau)
weighted_accuracy = (0.95 + 0.82 + 0.68) / 3 = 0.817
```

<strong>(4) Công cụ đánh giá chính thức của BFCL</strong>

BFCL cung cấp CLI tool chính thức để đánh giá:

```bash
# Cài đặt công cụ đánh giá BFCL
pip install bfcl

# Chạy đánh giá chính thức
bfcl evaluate \
    --model-result-path ./results.json \
    --test-category simple_python
```

Ưu điểm của việc dùng công cụ đánh giá chính thức là: nó sử dụng thuật toán AST matching chính thức, kết quả đánh giá hoàn toàn nhất quán với bảng xếp hạng (leaderboard), hỗ trợ tất cả category của BFCL v4, và có thể tự động sinh báo cáo đánh giá chi tiết.


### 12.2.2 Lấy dataset BFCL

Dataset BFCL có thể lấy theo các cách sau:

<strong>Cách 1: Clone từ GitHub repository chính thức (khuyến nghị)</strong>

Đây là cách đáng tin cậy nhất, có thể lấy được bộ dataset đầy đủ cùng ground truth:

```bash
# Clone repository BFCL
git clone https://github.com/ShishirPatil/gorilla.git temp_gorilla
cd temp_gorilla/berkeley-function-call-leaderboard

# Xem dataset BFCL v4
ls bfcl_eval/data/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  BFCL_v4_parallel.json  ...

# Xem ground truth
ls bfcl_eval/data/possible_answer/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  ...
```

Lý do khuyến nghị cách này là: nó bao gồm ground truth (đáp án chuẩn) đầy đủ, định dạng dữ liệu hoàn toàn nhất quán với công cụ đánh giá chính thức, có thể dùng trực tiếp script đánh giá chính thức, và hỗ trợ phiên bản mới nhất BFCL v4.

<strong>Cách 2: Dùng HelloAgents để nạp dữ liệu chính thức</strong>

Sau khi clone repository, dùng HelloAgents để nạp dữ liệu:

```python
from hello_agents.evaluation import BFCLDataset

# Nạp dữ liệu chính thức BFCL
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"  # Category BFCL v4
)

# Nạp dữ liệu (bao gồm test data và ground truth)
data = dataset.load()

print(f"✅ 加载了 {len(data)} 个测试样本")
print(f"✅ 加载了 {len(dataset.ground_truth)} 个ground truth")
# Output:
# ✅ 加载了 400 个测试样本
# ✅ 加载了 400 个ground truth
```

Nguyên lý hoạt động của bộ nạp này là: đầu tiên nạp test data từ `bfcl_eval/data/`, sau đó nạp ground truth từ `bfcl_eval/data/possible_answer/`, tiếp theo tự động merge test data và ground truth, cuối cùng giữ nguyên định dạng dữ liệu BFCL gốc. Các category của dataset BFCL v4 có thể xem trong Bảng 12.2.

<div align="center">
  <p>Bảng 12.2 Bốn category đánh giá trong benchmark BFCL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-2.png" alt="" width="85%"/>
</div>

Tất nhiên cũng có thể xem các category khả dụng bằng code:

```python
# Lấy tất cả category được hỗ trợ
categories = dataset.get_available_categories()
print(f"支持的类别: {categories}")
# Output: ['simple_python', 'simple_java', 'simple_javascript', 'multiple', ...]
```

### 12.2.3 Hiện thực đánh giá BFCL trong HelloAgents

Bây giờ hãy xem cách hiện thực đánh giá BFCL trong framework HelloAgents. Chúng ta cung cấp ba cách sử dụng:

<strong>Cách 1: Dùng BFCLEvaluationTool (khuyến nghị)</strong>

Đây là cách đơn giản nhất, một dòng code hoàn thành việc đánh giá, sinh báo cáo và đánh giá chính thức:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# 1. Tạo Agent cần đánh giá
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Tạo BFCL evaluation tool
bfcl_tool = BFCLEvaluationTool()

# 3. Chạy đánh giá (tự động hoàn thành mọi bước)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",  # Category đánh giá
    max_samples=5              # Số mẫu đánh giá (0 nghĩa là tất cả)
)

# 4. Xem kết quả
print(f"准确率: {results['overall_accuracy']:.2%}")
print(f"正确数: {results['correct_samples']}/{results['total_samples']}")
```

<strong>Output khi chạy:</strong>

```
============================================================
BFCL一键评估
============================================================

配置:
   评估类别: simple_python
   样本数量: 5
   智能体: TestAgent

============================================================
步骤1: 运行HelloAgents评估
============================================================
✅ BFCL数据集加载完成
   数据目录: ./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data
   类别: simple_python
   样本数: 400
   Ground truth数: 400

🔧 开始 BFCL 评估...
   进度: 1/5
   进度: 5/5

✅ BFCL 评估完成
   总体准确率: 100.00%
   simple_python: 100.00% (5/5)

📊 评估结果:
   准确率: 100.00%
   正确数: 5/5

============================================================
步骤2: 导出BFCL格式结果
============================================================
✅ BFCL格式结果已导出
   输出文件: ./evaluation_results/bfcl_official/BFCL_v4_simple_python_result.json

============================================================
步骤3: 运行BFCL官方评估
============================================================
✅ 结果文件已复制到: ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 运行命令: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL官方评估结果
============================================================
📊 评估结果汇总:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 最终结果:
   准确率: 100.00%
   正确数: 5/5

============================================================
步骤4: 生成评估报告
============================================================
📄 报告已生成: ./evaluation_reports/bfcl_report_20251011_005938.md

准确率: 100.00%
正确数: 5/5
```

<strong>Báo cáo Markdown được tự động sinh ra:</strong>

Sau khi đánh giá hoàn tất, hệ thống sẽ tự động sinh một báo cáo Markdown chi tiết, bao gồm:

```markdown
# BFCL评估报告
**生成时间**: 2025-10-11 00:59:38

## 📊 评估概览

- **智能体**: TestAgent
- **评估类别**: simple_python
- **总体准确率**: 100.00%
- **正确样本数**: 5/5

## 📈 详细指标

### 分类准确率

- **simple_python**: 100.00% (5/5)

## 📝 样本详情

| 样本ID | 问题 | 预测结果 | 正确答案 | 是否正确 |
|--------|------|----------|----------|----------|
| simple_python_0 | Find the area of a triangle... | [{'name': 'calculate_triangle_area'...}] | [{'function_name': {'base': [10]...}}] | ✅ |
| simple_python_1 | Calculate the factorial of 5... | [{'name': 'calculate_factorial'...}] | [{'function_name': {'number': [5]}}] | ✅ |
...

## 📊 准确率可视化
准确率: ██████████████████████████████████████████████████ 100.00%

## 💡 建议
- ✅ 表现优秀！智能体在工具调用方面表现出色。
```

<strong>Cách 2: Dùng script đánh giá một cú click</strong>

Phù hợp để đánh giá nhanh qua command line, trong code mẫu đi kèm chương này, chúng tôi cung cấp `04_run_bfcl_evaluation.py`, hỗ trợ gọi trực tiếp qua command line:

```bash
# Chạy script đánh giá
python chapter12/04_run_bfcl_evaluation.py --category simple_python --samples 10

# Chỉ định tên model (dùng cho đánh giá chính thức BFCL)
python examples/04_run_bfcl_evaluation.py \
    --category simple_python \
    --samples 10 \
    --model-name "Qwen/Qwen3-8B"
```

Script hỗ trợ ba tham số: `--category` chỉ định category đánh giá (mặc định simple_python), `--samples` chỉ định số mẫu đánh giá (mặc định 5, 0 nghĩa là tất cả), `--model-name` chỉ định tên model dùng cho đánh giá chính thức BFCL (mặc định Qwen/Qwen3-8B).

<strong>Cách 3: Dùng trực tiếp Dataset và Evaluator</strong>

Phù hợp cho kịch bản cần tùy chỉnh quy trình đánh giá:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.evaluation import BFCLDataset, BFCLEvaluator

# 1. Tạo Agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Nạp dataset
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"
)
data = dataset.load()

# 3. Tạo evaluator
evaluator = BFCLEvaluator(
    dataset=dataset,
    category="simple_python",
    evaluation_mode="ast"  # Dùng chế độ AST matching
)

# 4. Chạy đánh giá
results = evaluator.evaluate(agent, max_samples=10)

# 5. Xem kết quả
print(f"准确率: {results['overall_accuracy']:.2%}")
print(f"正确数: {results['correct_samples']}/{results['total_samples']}")

# 6. Export kết quả theo định dạng BFCL (tùy chọn)
evaluator.export_to_bfcl_format(
    results,
    output_path="./evaluation_results/my_results.json"
)
```

Thông qua ba cách trên, chúng ta có thể chọn phương pháp đánh giá phù hợp theo nhu cầu khác nhau. Nếu chỉ muốn nhanh chóng tìm hiểu hiệu suất của Agent, dùng BFCLEvaluationTool để đánh giá một cú click là tiện lợi nhất; nếu cần đánh giá hàng loạt hoặc tích hợp vào quy trình CI/CD, dùng script command line sẽ phù hợp hơn; nếu cần tùy chỉnh sâu quy trình đánh giá hoặc tích hợp vào hệ thống của riêng mình, dùng trực tiếp Dataset và Evaluator mang lại tính linh hoạt lớn nhất.




### 12.2.4 Tích hợp công cụ đánh giá chính thức của BFCL

Ở phần trước, chúng ta đã học cách dùng chức năng đánh giá có sẵn của HelloAgents. Thực ra, `BFCLEvaluationTool` đã <strong>tự động tích hợp công cụ đánh giá chính thức của BFCL</strong>, giúp bạn có được kết quả đánh giá uy tín, có thể so sánh.

Toàn bộ quy trình đánh giá gồm bốn bước: đầu tiên nạp test data từ dataset BFCL v4, sau đó dùng HelloAgents chạy đánh giá để lấy kết quả dự đoán của Agent, tiếp theo export kết quả sang định dạng chính thức của BFCL (JSONL), cuối cùng dùng script đánh giá chính thức để tính điểm cuối cùng. Quy trình này đảm bảo kết quả đánh giá hoàn toàn nhất quán với leaderboard BFCL, như Hình 12.3 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-3.png" alt="" width="85%"/>
  <p>Hình 12.3 Quy trình HelloAgents nạp và đánh giá BFCL</p>
</div>
Khi dùng `BFCLEvaluationTool`, đánh giá chính thức sẽ <strong>tự động chạy</strong> (mặc định bật):

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# Tạo Agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# Tạo evaluation tool
bfcl_tool = BFCLEvaluationTool()

# Chạy đánh giá (tự động chạy đánh giá chính thức)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    # run_official_eval=True  # Mặc định là True, có thể lược bỏ
    model_name="Qwen/Qwen3-8B"  # Tùy chọn, chỉ định tên model
)
```

Tool sẽ tự động thực thi toàn bộ quy trình đánh giá: đầu tiên chạy đánh giá HelloAgents để lấy kết quả dự đoán, sau đó export kết quả sang định dạng BFCL và lưu vào thư mục `evaluation_results/bfcl_official/`, tiếp theo sao chép file kết quả vào thư mục `result/{model_name}/` để phù hợp yêu cầu của công cụ đánh giá chính thức, sau đó chạy lệnh đánh giá chính thức BFCL để tính điểm, cuối cùng hiển thị kết quả đánh giá chính thức và sinh báo cáo đánh giá định dạng Markdown.

<strong>Ví dụ output của đánh giá chính thức:</strong>

```
============================================================
步骤3: 运行BFCL官方评估
============================================================

✅ 结果文件已复制到:
   ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 运行命令: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL官方评估结果
============================================================

📊 评估结果汇总:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 最终结果:
   准确率: 100.00%
   正确数: 5/5
```

Nếu bạn muốn tự tay kiểm soát quy trình đánh giá, có thể tắt chế độ tự động chạy đánh giá chính thức:

```python
# Tắt đánh giá chính thức
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    run_official_eval=False  # Tắt đánh giá chính thức
)

# Sau đó tự chạy đánh giá chính thức
import subprocess
subprocess.run([
    "bfcl", "evaluate",
    "--model", "Qwen/Qwen3-8B",
    "--test-category", "simple_python",
    "--partial-eval"
])
```

Bạn cũng có thể tự tạo báo cáo:

```python
# Chạy đánh giá
results = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Tự tạo báo cáo
report = bfcl_tool.generate_report(
    results,
    output_file="./my_reports/custom_report.md"
)

# In nội dung báo cáo
print(report)
```



### 12.2.5 Chi tiết hiện thực các thành phần cốt lõi

Ở các phần trước, chúng ta đã học cách dùng công cụ đánh giá BFCL. Bây giờ hãy tìm hiểu sâu cách các thành phần cốt lõi của hệ thống đánh giá HelloAgents được hiện thực như thế nào. Việc hiểu các chi tiết hiện thực này không chỉ giúp bạn sử dụng hệ thống đánh giá tốt hơn, mà còn giúp bạn tùy chỉnh và mở rộng theo nhu cầu riêng.

<strong>(1) BFCLDataset: Bộ nạp dataset</strong>

BFCLDataset chịu trách nhiệm nạp và quản lý dataset BFCL:

````python
class BFCLDataset:
    """BFCL数据集加载器"""

    def __init__(self, category: str = "simple", local_data_path: Optional[str] = None):
        self.category = category
        self.local_data_path = local_data_path
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """加载数据集"""
        # 优先从本地加载
        if self.local_data_path:
            return self._load_from_local()
        # 否则从Hugging Face加载
        return self._load_from_huggingface()
````
Vì dataset của BFCL nằm ngay trong repository chính thức, nên cách khuyến nghị ở đây là clone trực tiếp về local để đánh giá. Chỉ khi không tìm thấy thì mới nạp từ huggingface.

<strong>(2) BFCLEvaluator: Bộ thực thi đánh giá</strong>

BFCLEvaluator chịu trách nhiệm thực thi quy trình đánh giá. Cốt lõi của nó là phương thức `evaluate()`, phương thức này điều phối toàn bộ quá trình đánh giá:

````python
class BFCLEvaluator:
    """BFCL评估器"""

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """执行评估"""
        results = []

        for item in self.dataset[:max_samples]:
            # 1. 构造提示词
            prompt = self._build_prompt(item)

            # 2. 调用智能体
            response = agent.run(prompt)

            # 3. 提取函数调用
            predicted_calls = self._extract_function_calls(response)

            # 4. 与标准答案对比
            is_correct = self._compare_calls(predicted_calls, item["ground_truth"])

            results.append({
                "id": item["id"],
                "prediction": predicted_calls,
                "ground_truth": item["ground_truth"],
                "is_correct": is_correct
            })

        return {"results": results, "total_samples": len(results)}
````
Thiết kế của evaluator này bao gồm ba điểm cốt lõi: đầu tiên là xây dựng prompt, cần chuyển đổi câu hỏi và định nghĩa function trong dataset thành prompt mà Agent có thể hiểu được; thứ hai là trích xuất function call, cần trích xuất function call từ response của Agent, và hỗ trợ nhiều định dạng (JSON, code block, v.v.); cuối cùng là AST matching, dùng cây cú pháp trừu tượng để so sánh function call, điều này chính xác hơn so khớp chuỗi đơn giản.

Hãy xem hiện thực của việc trích xuất function call:

```python
def _extract_function_calls(self, response: str) -> List[Dict[str, Any]]:
    """从响应中提取函数调用

    支持多种格式：
    1. JSON格式：{"name": "func", "arguments": {...}}
    2. 代码块格式：```python\nfunc(arg1=val1)\n```
    3. 纯文本格式：func(arg1=val1)
    """
    calls = []

    # 尝试JSON解析
    try:
        json_match = re.search(r'\{.*\}', response, re.DOTALL)
        if json_match:
            data = json.loads(json_match.group())
            if isinstance(data, dict) and "name" in data:
                calls.append(data)
            elif isinstance(data, list):
                calls.extend(data)
    except json.JSONDecodeError:
        pass

    # 尝试代码块提取
    code_blocks = re.findall(r'```(?:python)?\n(.*?)\n```', response, re.DOTALL)
    for code in code_blocks:
        # 解析Python函数调用
        parsed_calls = self._parse_python_calls(code)
        calls.extend(parsed_calls)

    return calls
```

<strong>(3) BFCLMetrics: Bộ tính toán chỉ số</strong>

BFCLMetrics chịu trách nhiệm tính toán các chỉ số đánh giá khác nhau:

````python
class BFCLMetrics:
    """BFCL指标计算器"""

    def compute_metrics(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """计算所有指标"""
        return {
            "accuracy": self._compute_accuracy(results),
            "ast_match_rate": self._compute_ast_match_rate(results),
            "parameter_accuracy": self._compute_parameter_accuracy(results),
            "f1_score": self._compute_f1_score(results),
            "category_statistics": self._compute_category_stats(results)
        }
````
<strong>Hiện thực AST matching</strong>:

AST matching là kỹ thuật cốt lõi của đánh giá BFCL. Nó thông minh hơn so khớp chuỗi đơn giản, có thể nhận diện các function call tương đương về ngữ nghĩa:

```python
def _ast_match(self, pred_call: Dict, true_call: Dict) -> bool:
    """使用AST匹配函数调用

    AST匹配的优势：
    1. 忽略参数顺序：func(a=1, b=2) 等价于 func(b=2, a=1)
    2. 识别等价表达式：2+3 等价于 5
    3. 忽略空格和格式差异
    """
    # 1. 函数名必须完全匹配
    if pred_call.get("name") != true_call.get("name"):
        return False

    # 2. 将参数转换为AST节点
    pred_args = self._args_to_ast(pred_call.get("arguments", {}))
    true_args = self._args_to_ast(true_call.get("arguments", {}))

    # 3. 比较AST节点
    return ast.dump(pred_args) == ast.dump(true_args)

def _args_to_ast(self, args: Dict[str, Any]) -> ast.AST:
    """将参数字典转换为AST节点"""
    # 构造一个虚拟的函数调用
    code = f"func({', '.join(f'{k}={repr(v)}' for k, v in args.items())})"
    tree = ast.parse(code)
    return tree.body[0].value  # 返回Call节点
```

<strong>(4) Đóng gói thành tool: BFCLEvaluationTool</strong>

Cuối cùng, chúng ta đóng gói các thành phần này thành một Tool, để nó có thể được Agent gọi trực tiếp:

````python
class BFCLEvaluationTool(Tool):
    """BFCL评估工具"""

    def __init__(self, local_data_path: Optional[str] = None):
        super().__init__(
            name="bfcl_evaluation",
            description="评估智能体的工具调用能力"
        )
        self.dataset = None
        self.evaluator = None
        self.metrics_calculator = BFCLMetrics()

    def run(self, parameters: Dict[str, Any]) -> str:
        """执行评估"""
        # 1. 加载数据集
        self.dataset = BFCLDataset(...)

        # 2. 创建评估器
        self.evaluator = BFCLEvaluator(...)

        # 3. 运行评估
        results = self.evaluator.evaluate(...)

        # 4. 计算指标
        metrics = self.metrics_calculator.compute_metrics(...)

        # 5. 返回JSON结果
        return json.dumps(results, ensure_ascii=False)
````
Thiết kế của tool này tuân theo ba nguyên tắc cốt lõi: đầu tiên kế thừa lớp Tool cơ sở để tuân thủ chuẩn tool của HelloAgents, đảm bảo tích hợp liền mạch với framework; thứ hai là validate tham số nghiêm ngặt, kiểm tra tham số bắt buộc và cung cấp thông báo lỗi thân thiện, nâng cao trải nghiệm người dùng; cuối cùng là format hóa kết quả, trả về chuỗi JSON để tiện parse và hiển thị. Thông qua thiết kế module hóa này, chúng ta hiện thực được một hệ thống đánh giá vừa dễ dùng vừa linh hoạt, người dùng có thể dùng trực tiếp interface Tool ở tầng cao để nhanh chóng hoàn thành đánh giá, cũng có thể đi sâu vào các thành phần tầng thấp để tùy chỉnh nhằm đáp ứng nhu cầu đặc biệt.

### 12.2.6 Đề xuất mở rộng và tối ưu

Thông qua phần học trước, chúng ta đã nắm được cách dùng HelloAgents để đánh giá BFCL. Cần lưu ý rằng, hiện thực hiện tại của chúng ta là phiên bản tái hiện đơn giản dựa trên SimpleAgent, chủ yếu hoàn thành chức năng cơ bản của đánh giá BFCL. Trong ứng dụng thực tế, benchmark BFCL bao gồm nhiều mức độ khó và kịch bản, để đạt điểm cao hơn trên leaderboard, vẫn cần tối ưu và mở rộng thêm.

<strong>(1) Hạn chế của hiện thực hiện tại</strong>

SimpleAgent hiện tại của chúng ta chủ yếu tập trung vào việc xây dựng quy trình đánh giá, vẫn còn không gian cải thiện về năng lực gọi tool. SimpleAgent dùng định dạng gọi tool tùy chỉnh `[TOOL_CALL:tool_name:parameters]`, định dạng này cần LLM chủ động học và sử dụng, trong kịch bản phức tạp có thể không hiệu quả bằng Agent dùng Function Calling gốc (native). Ngoài ra, chúng ta hiện mới chỉ test các category cơ bản như simple_python, với các kịch bản phức tạp hơn như multiple, parallel, irrelevance, vẫn cần tối ưu riêng.

<strong>(2) Hướng nâng cao điểm số BFCL</strong>

Để tiếp tục nâng cao điểm đánh giá BFCL, có thể bắt đầu từ các hướng sau. Đầu tiên là tối ưu năng lực gọi tool của Agent, có thể cân nhắc dùng LLM hỗ trợ native function calling (như GPT-4, Claude, v.v.), hoặc cải tiến prompt để LLM hiểu rõ hơn định dạng gọi tool. Thứ hai là mở rộng thư viện tool, test BFCL liên quan đến nhiều loại function khác nhau, có thể dựa vào đặc điểm của test dataset để hiện thực trước các loại tool thông dụng, nâng cao độ bao phủ tool của Agent. Thứ ba là thiết kế chiến lược khác nhau cho từng mức độ khó, ví dụ ở kịch bản multiple cần Agent có khả năng lập kế hoạch chuỗi gọi tool nhiều bước, ở kịch bản parallel cần nhận diện các lệnh gọi tool có thể thực thi song song, ở kịch bản irrelevance cần phán đoán xem có thực sự cần gọi tool hay không.

<strong>(3) Đề xuất thực hành</strong>

Đối với các nhà phát triển muốn đạt kết quả tốt hơn trên BFCL, khuyến nghị áp dụng các chiến lược thực hành sau. Đầu tiên, bắt đầu từ category simple, đảm bảo việc gọi một function đơn giản hoạt động ổn định, đây là nền tảng cho việc tối ưu sau này. Sau đó, dần dần test các category phức tạp hơn như multiple, parallel, phân tích các case thất bại, tìm ra điểm yếu của Agent. Trong quá trình tối ưu, có thể tham khảo các model đạt điểm cao trên leaderboard BFCL, học hỏi ý tưởng thiết kế và kỹ thuật tối ưu của họ. Đồng thời, khuyến nghị dùng công cụ đánh giá chính thức để verify, đảm bảo kết quả sau tối ưu nhất quán với tiêu chuẩn leaderboard.

Dưới đây tổng hợp một số đề xuất có thể xử lý thêm khi đánh giá:

<strong>1. Đánh giá tăng dần (progressive)</strong>

Bắt đầu từ số mẫu nhỏ, dần dần tăng số mẫu:

```python
# Bước 1: test nhanh (5 mẫu)
results_quick = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Bước 2: test quy mô vừa (50 mẫu)
if results_quick['overall_accuracy'] > 0.8:
    results_medium = bfcl_tool.run(agent, category="simple_python", max_samples=50)

# Bước 3: đánh giá đầy đủ (toàn bộ mẫu)
if results_medium['overall_accuracy'] > 0.8:
    results_full = bfcl_tool.run(agent, category="simple_python", max_samples=0)
```

<strong>2. Đánh giá nhiều category</strong>

Đánh giá các task với độ khó khác nhau:

```python
categories = ["simple_python", "multiple", "parallel", "irrelevance"]

for category in categories:
    print(f"\n评估类别: {category}")
    results = bfcl_tool.run(agent, category=category, max_samples=10)
    print(f"准确率: {results['overall_accuracy']:.2%}")
```

<strong>3. Đánh giá so sánh</strong>

So sánh các Agent với cấu hình khác nhau:

```python
# Cấu hình 1: prompt mặc định
agent1 = SimpleAgent(name="Agent-Default", llm=llm)
results1 = bfcl_tool.run(agent1, category="simple_python", max_samples=10)

# Cấu hình 2: prompt đã tối ưu
agent2 = SimpleAgent(name="Agent-Optimized", llm=llm)
# ... 设置优化的系统提示词 ...
results2 = bfcl_tool.run(agent2, category="simple_python", max_samples=10)

# So sánh kết quả
print(f"默认配置准确率: {results1['overall_accuracy']:.2%}")
print(f"优化配置准确率: {results2['overall_accuracy']:.2%}")
```

Nếu kết quả đánh giá của bạn tốt, có thể cân nhắc submit lên leaderboard chính thức của BFCL!

<strong>Bước 1: Chuẩn bị tài liệu submit</strong>

1. Tài liệu mô tả model
2. File kết quả đánh giá (tất cả category)
3. Cách truy cập model (API hoặc link mã nguồn mở)

<strong>Bước 2: Submit lên GitHub</strong>

Truy cập repository chính thức của BFCL, submit Pull Request theo hướng dẫn:

- Địa chỉ repository: https://github.com/ShishirPatil/gorilla
- Hướng dẫn submit: tham khảo `CONTRIBUTING.md`

<strong>Bước 3: Chờ xét duyệt</strong>

Team BFCL sẽ xét duyệt bài submit của bạn, verify tính chính xác của kết quả. Sau khi được duyệt, model của bạn sẽ xuất hiện trên leaderboard chính thức!



## 12.3 GAIA: Đánh giá năng lực AI assistant tổng quát

### 12.3.1 Giới thiệu benchmark GAIA

GAIA (General AI Assistants) là benchmark đánh giá do Meta AI và Hugging Face phối hợp đưa ra, tập trung đánh giá <strong>năng lực tổng quát</strong> của AI assistant<sup>[2]</sup>. Khác với BFCL tập trung vào gọi tool, GAIA đánh giá hiệu suất tổng hợp của Agent trong các task thực tế.

Triết lý thiết kế của GAIA là: <strong>các vấn đề trong thế giới thực thường cần vận dụng tổng hợp nhiều năng lực</strong>. Một AI assistant xuất sắc không chỉ cần gọi tool, mà còn cần:

- <strong>Multi-step reasoning</strong>: phân rã vấn đề phức tạp thành nhiều vấn đề con
- <strong>Vận dụng tri thức</strong>: tận dụng tri thức nội tại và knowledge base bên ngoài
- <strong>Hiểu đa phương thức (multimodal)</strong>: xử lý nhiều loại input như văn bản, hình ảnh, file
- <strong>Duyệt web</strong>: lấy thông tin mới nhất từ internet
- <strong>Thao tác file</strong>: đọc và xử lý file ở nhiều định dạng khác nhau

<strong>(1) Cấu trúc dataset GAIA</strong>

Sau khi hiểu triết lý đánh giá của GAIA, hãy cùng tìm hiểu sâu cấu trúc cụ thể của dataset GAIA. GAIA bao gồm 466 câu hỏi thực tế được thiết kế kỹ lưỡng, các câu hỏi này được chia thành ba mức độ khó dựa trên độ phức tạp và số bước reasoning cần thiết, từ task reasoning zero-step đơn giản đến task khó cần multi-step reasoning phức tạp, bao phủ toàn diện các kịch bản mà Agent có thể gặp trong ứng dụng thực tế, như Bảng 12.3 thể hiện:

<div align="center">
  <p>Bảng 12.3 Phân bố mức độ khó của dataset GAIA</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-3.png" alt="" width="85%"/>
</div>
Về ví dụ mẫu của dataset GAIA, có thể tham khảo đoạn code dưới đây:

```json
{
  "task_id": "gaia_001",
  "Question": "What is the total population of the top 3 most populous cities in California?",
  "Level": 2,
  "Final answer": "12847521",
  "file_name": "",
  "file_path": "",
  "Annotator Metadata": {
    "Steps": [
      "Search for most populous cities in California",
      "Get population data for top 3 cities",
      "Sum the populations"
    ],
    "Number of steps": 3,
    "How long did this take?": "5 minutes",
    "Tools": ["web_search", "calculator"]
  }
}
```

<strong>Giải thích các field quan trọng:</strong>
- `Question`: mô tả câu hỏi
- `Level`: mức độ khó (1-3)
- `Final answer`: đáp án chuẩn (có thể là số, văn bản hoặc file)
- `file_name/file_path`: file đính kèm (nếu có)
- `Annotator Metadata`: metadata do người gán nhãn cung cấp (các bước reasoning, tool cần dùng, v.v.)

<strong>(2) Giới thiệu Quasi Exact Match</strong>

GAIA dùng thuật toán đánh giá <strong>Quasi Exact Match</strong>, đây là tiêu chuẩn đánh giá do GAIA chính thức định nghĩa. Ý tưởng cốt lõi của thuật toán này là: <strong>trước tiên chuẩn hóa (normalize) đáp án, sau đó mới tiến hành khớp chính xác</strong>.

Cho trước đáp án dự đoán $A_{\text{pred}}$ và đáp án chuẩn $A_{\text{true}}$, hàm Quasi Exact Match được định nghĩa như sau:

$$
\text{Quasi\_Exact\_Match}(A_{\text{pred}}, A_{\text{true}}) = \begin{cases}
1 & \text{if } \mathcal{N}(A_{\text{pred}}) = \mathcal{N}(A_{\text{true}}) \\
0 & \text{otherwise}
\end{cases}
$$

trong đó $\mathcal{N}(\cdot)$ là hàm chuẩn hóa, áp dụng các quy tắc khác nhau tùy theo loại đáp án.

Hàm chuẩn hóa áp dụng các quy tắc khác nhau tùy theo loại đáp án. Đối với loại số, cần loại bỏ dấu phân cách phẩy (`1,000` → `1000`) và ký hiệu đơn vị (`$100` → `100`, `50%` → `50`), ví dụ `"$1,234.56"` được chuẩn hóa thành `"1234.56"`. Đối với loại chuỗi, cần chuyển về chữ thường (`"Apple"` → `"apple"`), loại bỏ mạo từ (`"the apple"` → `"apple"`), loại bỏ khoảng trắng thừa (`"hello  world"` → `"hello world"`) và loại bỏ dấu câu ở cuối (`"hello."` → `"hello"`), ví dụ `"The United States"` được chuẩn hóa thành `"united states"`. Đối với loại danh sách (list), cần tách các phần tử bằng dấu phẩy, áp dụng chuẩn hóa chuỗi cho mỗi phần tử, sắp xếp theo thứ tự chữ cái rồi nối lại, ví dụ `"Paris, London, Berlin"` được chuẩn hóa thành `"berlin,london,paris"`.

<strong>Ví dụ chuẩn hóa:</strong>

```python
# Đáp án dạng số
原始答案: "$1,234.56"
归一化后: "1234.56"

# Đáp án dạng chuỗi
原始答案: "The United States of America"
归一化后: "united states of america"

# Đáp án dạng danh sách
原始答案: "Paris, London, Berlin"
归一化后: "berlin, london, paris"
```

<strong>(3) Chỉ số đánh giá GAIA</strong>

GAIA sử dụng các chỉ số sau để đánh giá hiệu năng Agent:

<strong>1. Tỷ lệ khớp chính xác (Exact Match Rate)</strong>

Exact Match Rate là chỉ số cốt lõi của GAIA, được định nghĩa là tỷ lệ mẫu khớp Quasi Exact Match thành công:

$$
\text{Exact Match Rate} = \frac{1}{N} \sum_{i=1}^{N} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

trong đó:
- $N$ là tổng số mẫu
- $A_{\text{pred},i}$ là đáp án dự đoán của mẫu thứ $i$
- $A_{\text{true},i}$ là đáp án chuẩn của mẫu thứ $i$
- $\text{Quasi\_Exact\_Match}(\cdot, \cdot) \in \{0, 1\}$ là hàm Quasi Exact Match

<strong>2. Accuracy theo level (Level-wise Accuracy)</strong>

Với mỗi mức độ khó $\ell \in \{1, 2, 3\}$, tính Accuracy của level đó:

$$
\text{Accuracy}_\ell = \frac{1}{|D_\ell|} \sum_{i \in D_\ell} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

trong đó $D_\ell$ là tập mẫu của mức độ khó $\ell$, $|D_\ell|$ là số mẫu của level đó.

<strong>3. Tỷ lệ suy giảm theo độ khó (Difficulty Progression Drop Rate)</strong>

Đo mức độ suy giảm hiệu suất của Agent khi độ khó tăng lên:

$$
\text{Drop Rate}_{\ell \to \ell+1} = \frac{\text{Accuracy}_\ell - \text{Accuracy}_{\ell+1}}{\text{Accuracy}_\ell}
$$

- $\text{Drop Rate}_{1 \to 2}$: tỷ lệ suy giảm từ Level 1 sang Level 2
- $\text{Drop Rate}_{2 \to 3}$: tỷ lệ suy giảm từ Level 2 sang Level 3

<strong>4. Số bước reasoning trung bình (Average Reasoning Steps)</strong>

Đánh giá số bước trung bình mà Agent cần để hoàn thành task:

$$
\text{Avg Steps} = \frac{1}{N_{\text{correct}}} \sum_{i \in \text{Correct}} \text{steps}_i
$$

trong đó $N_{\text{correct}}$ là số mẫu trả lời đúng, $\text{steps}_i$ là số bước reasoning của mẫu thứ $i$.

<strong>Giải thích chỉ số:</strong>

- <strong>Exact Match Rate = 1.0</strong>: tất cả mẫu đều hoàn toàn đúng
- <strong>Exact Match Rate = 0.5</strong>: 50% mẫu đúng, 50% mẫu sai
- <strong>Drop Rate = 0.3</strong>: độ khó tăng khiến Accuracy giảm 30%
- <strong>Drop Rate = 0.0</strong>: độ khó tăng không ảnh hưởng đến Accuracy (trường hợp lý tưởng)

<strong>Ví dụ đánh giá:</strong>

Giả sử chúng ta đánh giá 10 mẫu, kết quả có thể tham khảo Bảng 12.4 dưới đây:

<div align="center">
  <p>Bảng 12.4 Phân bố mức độ khó của dataset GAIA</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-4.png" alt="" width="85%"/>
</div>

Nếu muốn tính chỉ số cho case này, có thể tham khảo script Python dưới đây.

```python
# 1. 精确匹配率
total_samples = 10
correct_samples = 7  # 样本1,2,3,5,6,8,9
exact_match_rate = correct_samples / total_samples = 0.70  # 70%

# 2. 分级准确率
level_1_correct = 3  # 样本1,2,3
level_1_total = 3
level_1_accuracy = 3 / 3 = 1.00  # 100%

level_2_correct = 2  # 样本5,6
level_2_total = 3
level_2_accuracy = 2 / 3 = 0.67  # 67%

level_3_correct = 2  # 样本8,9
level_3_total = 4
level_3_accuracy = 2 / 4 = 0.50  # 50%

# 3. 难度递进下降率
drop_rate_1_to_2 = (1.00 - 0.67) / 1.00 = 0.33  # 33%
drop_rate_2_to_3 = (0.67 - 0.50) / 0.67 = 0.25  # 25%

print(f"精确匹配率: {exact_match_rate:.2%}")  # 70.00%
print(f"Level 1准确率: {level_1_accuracy:.2%}")  # 100.00%
print(f"Level 2准确率: {level_2_accuracy:.2%}")  # 66.67%
print(f"Level 3准确率: {level_3_accuracy:.2%}")  # 50.00%
print(f"Level 1→2 下降率: {drop_rate_1_to_2:.2%}")  # 33.00%
print(f"Level 2→3 下降率: {drop_rate_2_to_3:.2%}")  # 25.00%
```

<strong>Phân tích kết quả:</strong>

- <strong>Hiệu suất tổng thể</strong>: Exact Match Rate 70%, hiệu suất tốt
- <strong>Độ nhạy với độ khó</strong>: từ Level 1 xuống Level 2 giảm 33%, cho thấy Agent suy giảm rõ rệt ở task độ khó trung bình
- <strong>Giới hạn năng lực</strong>: Accuracy Level 3 là 50%, cho thấy Agent vẫn còn không gian cải thiện ở task phức tạp

Drop Rate càng lớn, cho thấy năng lực của Agent suy giảm càng rõ khi xử lý task phức tạp.

<strong>(4) System prompt chính thức của GAIA</strong>

GAIA yêu cầu dùng system prompt đặc thù, để đảm bảo output của model tuân theo định dạng đánh giá:

```python
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""
```

GAIA có yêu cầu nghiêm ngặt về định dạng đáp án: đáp án phải theo định dạng `FINAL ANSWER: [答案]`; đối với đáp án dạng số, không dùng dấu phẩy phân cách và ký hiệu đơn vị; đối với đáp án dạng chuỗi, không dùng mạo từ và viết tắt; đối với đáp án dạng danh sách, dùng dấu phẩy phân cách và sắp xếp theo thứ tự chữ cái.

### 12.3.2 Lấy dataset GAIA

<strong>Lưu ý quan trọng</strong>: GAIA là <strong>dataset có hạn chế truy cập (Gated Dataset)</strong>, cần xin quyền truy cập trên HuggingFace trước.

<strong>Bước 1: Xin quyền truy cập</strong>

1. Truy cập https://huggingface.co/datasets/gaia-benchmark/GAIA
2. Nhấn nút "Request access"
3. Điền form đăng ký (thường được duyệt trong vài giây)
4. Lấy HuggingFace Token của bạn: https://huggingface.co/settings/tokens

<strong>Bước 2: Cấu hình biến môi trường</strong>

Thêm HuggingFace Token của bạn vào file `.env`:

```bash
# Cấu hình HuggingFace API
HF_TOKEN=hf_your_token_here
```

<strong>Cách 1: Dùng HelloAgents tự động tải về (khuyến nghị)</strong>

HelloAgents sẽ tự động xử lý việc tải về và cache dataset GAIA:

```python
from hello_agents.evaluation import GAIADataset
import os

# Đảm bảo đã thiết lập HF_TOKEN, nếu đã thiết lập trong .env thì không cần dòng này
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Tự động tải về ./data/gaia/
dataset = GAIADataset(
    dataset_name="gaia-benchmark/GAIA",
    split="validation",  # hoặc "test"
    level=1  # Tùy chọn: 1, 2, 3, None (tất cả)
)
items = dataset.load()

print(f"加载了 {len(items)} 个测试样本")
# Output: 加载了 53 个测试样本 (Level 1)
```

<strong>Nguyên lý hoạt động</strong>:

- Lần chạy đầu tiên, dùng `snapshot_download` để tải toàn bộ dataset về `./data/gaia/`
- Dataset bao gồm 114 file (câu hỏi, hình ảnh, PDF, v.v.)
- Các lần sử dụng sau sẽ nạp trực tiếp từ local, tốc độ nhanh

<strong>Cấu trúc thư mục dataset</strong>:
```
./data/gaia/
├── 2023/
│   ├── validation/
│   │   ├── metadata.jsonl  (165个问题)
│   │   ├── *.png, *.pdf, *.csv, *.xlsx  (附件文件)
│   └── test/
│       ├── metadata.jsonl  (301个问题)
│       └── ... (附件文件)
├── GAIA.py
└── README.md
```

<strong>Cách 2: Tải về thủ công</strong>

Nếu bạn muốn tự tay tải dataset về:

```python
from huggingface_hub import snapshot_download
import os

# Thiết lập Token
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Tải dataset
snapshot_download(
    repo_id="gaia-benchmark/GAIA",
    repo_type="dataset",
    local_dir="./data/gaia",
    token=os.getenv("HF_TOKEN")
)
```

<strong>Xem thống kê dataset</strong>:

```python
# Xem thống kê dataset
stats = dataset.get_statistics()
print(f"总样本数: {stats['total_samples']}")
print(f"级别分布: {stats['level_distribution']}")
# Output:
# 总样本数: 165
# 级别分布: {1: 53, 2: 62, 3: 50}
```


### 12.3.3 Hiện thực đánh giá GAIA trong HelloAgents

Tương tự BFCL, chúng ta cung cấp hai cách đánh giá, khuyến nghị dùng <strong>cách 1</strong>.

<strong>Cách 1: Dùng GAIAEvaluationTool để đánh giá một cú click</strong>

Đây là cách đơn giản nhất, tự động hoàn thành việc tải dataset, thực thi đánh giá, export kết quả và sinh báo cáo:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import GAIAEvaluationTool

# System prompt chính thức của GAIA (từ paper)
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""

# 1. Tạo Agent (dùng system prompt chính thức của GAIA)
llm = HelloAgentsLLM()
agent = SimpleAgent(
    name="TestAgent",
    llm=llm,
    system_prompt=GAIA_SYSTEM_PROMPT  # Quan trọng: dùng prompt chính thức của GAIA
)

# 2. Tạo GAIA evaluation tool
gaia_tool = GAIAEvaluationTool()

# 3. Chạy đánh giá một cú click
results = gaia_tool.run(
    agent=agent,
    level=1,  # Level 1: task đơn giản
    max_samples=5,  # Đánh giá 5 mẫu
    export_results=True,  # Export kết quả định dạng GAIA
    generate_report=True  # Sinh báo cáo đánh giá
)

# 4. Xem kết quả
print(f"精确匹配率: {results['exact_match_rate']:.2%}")
print(f"部分匹配率: {results['partial_match_rate']:.2%}")
print(f"正确数: {results['exact_matches']}/{results['total_samples']}")
```

<strong>Kết quả chạy:</strong>

```
============================================================
GAIA一键评估
============================================================

配置:
   智能体: TestAgent
   难度级别: 1
   样本数量: 5

============================================================
步骤1: 运行HelloAgents评估
============================================================
   正在从HuggingFace下载: gaia-benchmark/GAIA
   📥 下载GAIA数据集...
   ✓ 数据集下载完成
   ✓ 加载了 165 个样本
✅ GAIA数据集加载完成
   数据源: gaia-benchmark/GAIA
   分割: validation
   级别: 1
   样本数: 53

🌟 开始 GAIA 评估...
   样本数量: 5
   进度: 5/5
✅ GAIA 评估完成
   精确匹配率: 80.00%
   部分匹配率: 80.00%

============================================================
步骤2: 导出GAIA格式结果
============================================================
✅ GAIA格式结果已导出
   输出文件: evaluation_results\gaia_official\gaia_level1_result_20251011_012648.jsonl
   样本数: 5
   包含推理轨迹: True
📄 提交说明已生成: evaluation_results\gaia_official\SUBMISSION_GUIDE_20251011_012648.md

============================================================
步骤3: 生成评估报告
============================================================
📄 报告已生成: evaluation_reports\gaia_report_20251011_012648.md

============================================================
🎯 最终结果
============================================================
   精确匹配率: 80.00%
   部分匹配率: 80.00%
   正确数: 4/5
```

Sau khi đánh giá hoàn tất sẽ tự động sinh ba loại file: đầu tiên là file kết quả định dạng GAIA (`evaluation_results/gaia_official/gaia_level1_result_*.jsonl`), dùng định dạng JSONL (mỗi dòng một JSON object), có thể dùng trực tiếp để submit lên leaderboard GAIA; tiếp theo là file hướng dẫn submit (`evaluation_results/gaia_official/SUBMISSION_GUIDE_*.md`), bao gồm các bước submit chi tiết, giải thích định dạng file kết quả và các lưu ý; cuối cùng là báo cáo đánh giá (`evaluation_reports/gaia_report_*.md`), bao gồm tóm tắt kết quả đánh giá, chỉ số chi tiết, chi tiết từng mẫu và biểu đồ trực quan.

<strong>Lưu ý</strong>: nếu bạn thấy kết quả đánh giá sinh ra không như ý (ví dụ Accuracy thấp), đây là hiện tượng bình thường. Mặc dù Level 1 là task reasoning một bước, nhưng vẫn cần Agent sở hữu năng lực gọi tool (như search engine, calculator, v.v.) mới có thể trả lời đúng. SimpleAgent chúng ta đang dùng hiện tại chủ yếu để minh họa quy trình đánh giá, vẫn còn không gian cải thiện về năng lực gọi tool.

<strong>Cách 2: Dùng Dataset + Evaluator (tùy chỉnh linh hoạt)</strong>

Nếu cần kiểm soát chi tiết hơn, có thể dùng trực tiếp các thành phần tầng thấp:

```python
from hello_agents.evaluation import GAIADataset, GAIAEvaluator

# 1. Nạp dataset
dataset = GAIADataset(level=1)
items = dataset.load()
print(f"加载了 {len(items)} 个样本")

# 2. Tạo evaluator
evaluator = GAIAEvaluator(dataset=dataset, level=1)

# 3. Chạy đánh giá
results = evaluator.evaluate(agent, max_samples=5)

# 4. Export kết quả theo định dạng GAIA
evaluator.export_to_gaia_format(
    results,
    "gaia_results.jsonl",
    include_reasoning=True
)
```

Báo cáo đánh giá được sinh ra (`gaia_report_*.md`) có thể tham khảo file dưới đây:

```markdown
# GAIA评估报告

**生成时间**: 2025-10-11 01:26:48

## 📊 评估概览

- **智能体**: TestAgent
- **难度级别**: 1
- **总样本数**: 2
- **精确匹配数**: 1
- **部分匹配数**: 1
- **精确匹配率**: 50.00%
- **部分匹配率**: 50.00%

## 📈 详细指标

### 分级准确率

- **Level 1**: 50.00% 精确 / 50.00% 部分 (1/2)

## 📝 样本详情（前10个）

| 任务ID | 级别 | 预测答案 | 正确答案 | 精确匹配 | 部分匹配 |
|--------|------|----------|----------|----------|----------|
| e1fc63a2-da7a-432f-be78-7c4a95598703 | 1 | 24000 | 17 | ❌ | ❌ |
| 8e867cd7-cff9-4e6c-867a-ff5ddc2550be | 1 | 3 | 3 | ✅ | ✅ |

## 📊 准确率可视化

精确匹配: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%
部分匹配: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%


## 💡 建议

- ⚠️ 表现一般，需要改进。
- 💡 建议检查工具使用和多步推理能力。
```

**Kết quả định dạng GAIA được sinh ra (`gaia_level1_result_*.jsonl`):<strong>

```json
{"task_id": "e1fc63a2-da7a-432f-be78-7c4a95598703", "model_answer": "24000", "reasoning_trace": "24000"}
{"task_id": "8e867cd7-cff9-4e6c-867a-ff5ddc2550be", "model_answer": "3", "reasoning_trace": "3"}
```

### 12.3.4 Submit kết quả lên leaderboard chính thức của GAIA

Sau khi dùng GAIAEvaluationTool để chạy đánh giá, các file cần thiết để submit và hướng dẫn submit chi tiết sẽ được sinh ra trong thư mục `evaluation_results/gaia_official/`.

1. </strong>File kết quả định dạng GAIA**: `gaia_level1_result_*.jsonl`
   ```json
   {"task_id": "xxx", "model_answer": "答案", "reasoning_trace": "推理过程"}
   {"task_id": "yyy", "model_answer": "答案", "reasoning_trace": "推理过程"}
   ```

2. <strong>File hướng dẫn submit</strong>: `SUBMISSION_GUIDE_*.md`

Mở file `SUBMISSION_GUIDE_*.md` được tự động sinh ra, bên trong có hướng dẫn submit đầy đủ:

Cụ thể, mở trình duyệt, truy cập:
```
https://huggingface.co/spaces/gaia-benchmark/leaderboard
```

Như Hình 12.4 thể hiện, điền thông tin vào form submit là xong:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-4.png" alt="" width="85%"/>
  <p>Hình 12.4 Sơ đồ quy trình đánh giá GAIA</p>
</div>

Trước khi submit, có thể kiểm tra thủ công file JSON được sinh ra:

```python
import json

# Đọc file kết quả
with open("evaluation_results/gaia_official/gaia_level1_result_*.jsonl", "r") as f:
    for line in f:
        result = json.loads(line)
        print(f"Task ID: {result['task_id']}")
        print(f"Answer: {result['model_answer']}")
        print(f"Reasoning: {result['reasoning_trace']}")
        print("-" * 50)
```

### 12.3.5 Chi tiết hiện thực các thành phần cốt lõi

Việc hiện thực hệ thống đánh giá GAIA tương tự BFCL, nhưng có một số thiết kế đặc thù cho đánh giá năng lực tổng quát.

<strong>(1) GAIADataset: Bộ nạp dữ liệu hỗ trợ đa phương thức</strong>

Điểm đặc biệt của dataset GAIA là nó bao gồm dữ liệu đa phương thức (văn bản, file, hình ảnh, v.v.):

````python
class GAIADataset:
    """GAIA数据集加载器

    支持从HuggingFace加载GAIA数据集（受限数据集）
    """

    def __init__(
        self,
        level: Optional[int] = None,
        split: str = "validation",
        local_data_dir: Optional[str] = None
    ):
        self.level = level
        self.split = split
        self.local_data_dir = local_data_dir or "./data/gaia"
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """加载数据集"""
        # 从HuggingFace下载
        items = self._load_from_huggingface()

        # 按级别过滤
        if self.level:
            items = [item for item in items if item.get("level") == self.level]

        self.data = items
        return items

    def _load_from_huggingface(self) -> List[Dict[str, Any]]:
        """从HuggingFace下载GAIA数据集"""
        from huggingface_hub import snapshot_download
        import json

        # 下载数据集
        repo_id = "gaia-benchmark/GAIA"
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset",
            local_dir=self.local_data_dir,
            local_dir_use_symlinks=False
        )

        # 加载JSONL文件
        data_file = Path(local_dir) / "2023" / self.split / "metadata.jsonl"
        items = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                item = json.loads(line)
                items.append(self._standardize_item(item))

        return items
````
<strong>(2) GAIAEvaluator: Hiện thực thuật toán đánh giá chính thức của GAIA</strong>

Đánh giá của GAIA dùng thuật toán <strong>Quasi Exact Match</strong>, cần logic chuẩn hóa và so khớp đáp án đặc biệt:

````python
class GAIAEvaluator:
    """GAIA评估器

    实现GAIA官方的准精确匹配（Quasi Exact Match）评估算法
    """

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """执行评估"""
        dataset_items = self.dataset.load()

        if max_samples:
            dataset_items = dataset_items[:max_samples]

        results = []
        for i, item in enumerate(dataset_items, 1):
            # 1. 构造提示词
            prompt = self._build_prompt(item["question"], item)

            # 2. 调用智能体
            response = agent.run(prompt)

            # 3. 提取答案（GAIA格式：FINAL ANSWER: [答案]）
            predicted_answer = self._extract_answer(response)

            # 4. 归一化答案（GAIA官方规则）
            normalized_pred = self._normalize_answer(predicted_answer)
            normalized_truth = self._normalize_answer(item["final_answer"])

            # 5. 准精确匹配
            exact_match = (normalized_pred == normalized_truth)

            results.append({
                "task_id": item["task_id"],
                "predicted": predicted_answer,
                "expected": item["final_answer"],
                "exact_match": exact_match,
                "level": item.get("level", 0)
            })

        return self._format_results(results)
````
GAIA dùng các quy tắc chuẩn hóa đặc thù để xử lý các loại đáp án khác nhau:

```python
def _normalize_answer(self, answer: str) -> str:
    """标准化答案字符串（GAIA官方标准化规则）

    规则：
    1. 数字：移除逗号分隔符和单位符号
    2. 字符串：移除冠词、转小写、移除多余空格
    3. 列表：逗号分隔，按字母顺序排序
    """
    if not answer:
        return ""

    answer = answer.strip()

    # 检查是否是逗号分隔的列表
    if ',' in answer:
        parts = [self._normalize_single_answer(p.strip()) for p in answer.split(',')]
        parts.sort()  # GAIA要求按字母顺序排序
        return ','.join(parts)
    else:
        return self._normalize_single_answer(answer)

def _normalize_single_answer(self, answer: str) -> str:
    """标准化单个答案（不包含逗号的答案）"""
    answer = answer.strip().lower()

    # 移除常见的冠词
    articles = ['the', 'a', 'an']
    words = answer.split()
    if words and words[0] in articles:
        words = words[1:]
        answer = ' '.join(words)

    # 移除货币符号和百分号
    answer = answer.replace('$', '').replace('%', '').replace('€', '').replace('£', '')

    # 移除数字中的逗号分隔符
    answer = re.sub(r'(\d),(\d)', r'\1\2', answer)

    # 移除多余空格
    answer = ' '.join(answer.split())

    # 移除末尾的标点符号
    answer = answer.rstrip('.,;:!?')

    return answer
```

GAIA yêu cầu output của model theo định dạng `FINAL ANSWER: [答案]`:

```python
def _extract_answer(self, response: str) -> str:
    """从响应中提取答案（GAIA格式）

    GAIA要求答案格式为：FINAL ANSWER: [答案]
    """
    # 首先尝试提取GAIA官方格式的答案
    final_answer_pattern = r'FINAL ANSWER:\s*(.+?)(?:\n|$)'
    match = re.search(final_answer_pattern, response, re.IGNORECASE | re.MULTILINE)
    if match:
        answer = match.group(1).strip()
        # 移除可能的方括号
        answer = answer.strip('[]')
        return answer

    # 备用方案：查找其他答案标记
    answer_patterns = [
        r'答案[：:]\s*(.+)',
        r'最终答案[：:]\s*(.+)',
        r'Final answer[：:]\s*(.+)',
        r'Answer[：:]\s*(.+)',
    ]

    for pattern in answer_patterns:
        match = re.search(pattern, response, re.IGNORECASE)
        if match:
            return match.group(1).strip()

    # 如果没有找到标记，返回最后一个非空行
    lines = response.strip().split('\n')
    for line in reversed(lines):
        line = line.strip()
        if line and not line.startswith('#'):
            return line

    return response.strip()
```

Sau khi đánh giá hoàn tất, có thể export sang định dạng JSONL theo yêu cầu chính thức của GAIA:

```python
def export_to_gaia_format(
    self,
    results: Dict[str, Any],
    output_path: Union[str, Path],
    include_reasoning: bool = True
) -> None:
    """导出为GAIA官方格式（JSONL）

    GAIA要求的格式：
    {"task_id": "xxx", "model_answer": "答案", "reasoning_trace": "推理过程"}
    """
    output_path = Path(output_path)
    output_path.parent.mkdir(parents=True, exist_ok=True)

    with open(output_path, 'w', encoding='utf-8') as f:
        for result in results.get("detailed_results", []):
            entry = {
                "task_id": result["task_id"],
                "model_answer": result["predicted"]
            }

            if include_reasoning:
                entry["reasoning_trace"] = result.get("response", result["predicted"])

            f.write(json.dumps(entry, ensure_ascii=False) + '\n')
```

<strong>(3) GAIAEvaluationTool: Tool đánh giá một cú click</strong>

GAIAEvaluationTool đóng gói toàn bộ quy trình đánh giá, cung cấp chức năng đánh giá một cú click:

````python
class GAIAEvaluationTool(Tool):
    """GAIA评估工具

    提供一键评估功能：
    1. 运行HelloAgents评估
    2. 导出GAIA格式结果
    3. 生成评估报告
    4. 生成提交说明
    """

    def run(
        self,
        agent: Any,
        level: Optional[int] = None,
        max_samples: Optional[int] = None,
        local_data_dir: Optional[str] = None,
        export_results: bool = True,
        generate_report: bool = True
    ) -> Dict[str, Any]:
        """执行GAIA一键评估"""
        # 步骤1: 运行HelloAgents评估
        results = self._run_evaluation(agent, level, max_samples, local_data_dir)

        # 步骤2: 导出GAIA格式结果
        if export_results:
            self._export_results(results)

        # 步骤3: 生成评估报告
        if generate_report:
            self.generate_report(results)

        return results
````
GAIAEvaluationTool sẽ tự động sinh báo cáo đánh giá:

```python
def generate_report(
    self,
    results: Dict[str, Any],
    output_file: Optional[Union[str, Path]] = None
) -> str:
    """生成评估报告"""
    report = f"""# GAIA评估报告

**生成时间**: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}

## 📊 评估概览

- **智能体**: {results.get("agent_name", "Unknown")}
- **难度级别**: {results.get("level_filter") or '全部'}
- **总样本数**: {results.get("total_samples", 0)}
- **精确匹配数**: {results.get("exact_matches", 0)}
- **精确匹配率**: {results.get("exact_match_rate", 0):.2%}

## 📈 详细指标

### 分级准确率

{self._format_level_metrics(results.get("level_metrics", {}))}

## 📝 样本详情（前10个）

{self._format_sample_details(results.get("detailed_results", [])[:10])}

## 📊 准确率可视化

{self._format_visualization(results.get("exact_match_rate", 0))}

## 💡 建议

{self._format_suggestions(results.get("exact_match_rate", 0))}
"""

    # 保存报告
    if output_file is None:
        output_dir = Path("./evaluation_reports")
        output_dir.mkdir(parents=True, exist_ok=True)
        output_file = output_dir / f"gaia_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.md"

    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(report)

    return report
```

## 12.4 Đánh giá chất lượng sinh dữ liệu (Data Generation)

Trong phát triển hệ thống AI, dữ liệu huấn luyện chất lượng cao là nền tảng cho hiệu năng hệ thống. Phần này giới thiệu cách dùng framework HelloAgents để đánh giá chất lượng dữ liệu được sinh ra, lấy việc sinh đề toán phong cách AIME (American Invitational Mathematics Examination)<sup>[9]</sup> làm ví dụ.

AIME là kỳ thi toán học có độ khó trung bình do Hiệp hội Toán học Hoa Kỳ (MAA) tổ chức, nằm giữa AMC 10/12 và Olympic Toán học Hoa Kỳ (USAMO). Đề thi AIME có đặc điểm rõ ràng: đáp án mỗi câu là một số nguyên từ 0 đến 999, đề bài bao phủ nhiều lĩnh vực toán học như đại số, hình học, lý thuyết số, tổ hợp, xác suất, cần multi-step reasoning nhưng không liên quan đến lý thuyết cao cấp, độ khó vừa phải (tương đương câu 6-9 của đề AIME). Những đặc điểm này khiến đề AIME trở thành benchmark lý tưởng để đánh giá chất lượng sinh đề toán: định dạng đáp án thống nhất thuận tiện cho đánh giá tự động, độ khó đề vừa phải phù hợp để sinh hàng loạt. Chúng ta dùng dataset `TianHongZXY/aime-1983-2025` trên HuggingFace làm tham chiếu, dataset này bao gồm hơn 900 đề thi AIME thật từ năm 1983 đến 2025, cung cấp nguồn mẫu tham khảo phong phú cho việc sinh và đánh giá của chúng ta.

### 12.4.1 Tổng quan phương pháp đánh giá

Trong đánh giá chất lượng sinh dữ liệu, chúng ta áp dụng ba phương pháp đánh giá bổ trợ lẫn nhau: LLM Judge, Win Rate và chấm điểm thủ công (human scoring). Việc lựa chọn ba phương pháp này có hai lý do quan trọng. Thứ nhất, xét từ góc độ phương pháp luận, đây là các phương án đánh giá tự động thông dụng trong lĩnh vực Agent hiện nay, cũng là cách làm chủ đạo trong nhiều bài báo học thuật, có mức độ công nhận và nền tảng thực tiễn rộng rãi. Thứ hai, xét từ góc độ tính phù hợp, ba phương pháp này tự nhiên phù hợp với kịch bản đánh giá của chúng ta: LLM Judge và Win Rate dùng để đánh giá chất lượng sinh đề (đánh giá đa chiều từ các khía cạnh như tính đúng đắn, độ rõ ràng, mức độ phù hợp của độ khó), còn chấm điểm thủ công dùng để đánh giá chất lượng sinh đáp án (verify tính chính xác của đáp án thông qua chuyên gia con người), sự phân công này rất hợp lý và dễ hiểu.

Dưới đây chúng ta sẽ giới thiệu chi tiết cách hiện thực cụ thể của ba phương pháp đánh giá này. Quy trình hiện thực toàn bộ case như Hình 12.5 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-5.png" alt="" width="85%"/>
  <p>Hình 12.5 Sơ đồ quy trình đánh giá chất lượng sinh dữ liệu</p>
</div>
<strong>(1) Đánh giá LLM Judge</strong>

<strong>Động lực thiết kế</strong>: Trong đánh giá chất lượng sinh dữ liệu, chúng ta cần đánh giá chất lượng nhanh chóng và nhất quán cho số lượng lớn đề được sinh ra. Đánh giá thủ công truyền thống tuy chính xác, nhưng chi phí cao, hiệu suất thấp, khó đáp ứng nhu cầu sinh dữ liệu quy mô lớn. LLM Judge thông qua việc dùng large language model làm giám khảo, có thể tự động đánh giá chất lượng dữ liệu sinh ra từ nhiều chiều, không chỉ nâng cao đáng kể hiệu suất đánh giá, mà còn giữ được tính nhất quán của tiêu chuẩn đánh giá. Quan trọng hơn, LLM Judge có thể cung cấp lý do chấm điểm chi tiết và đề xuất cải thiện, giúp chúng ta hiểu ưu nhược điểm của dữ liệu sinh ra, cung cấp hướng đi cho việc tối ưu sau này.

Trong hiện thực của chúng ta, LLM Judge đánh giá chất lượng đề AIME từ bốn chiều quan trọng:

<div align="center">
  <p>Bảng 12.5 Các chiều đánh giá đề AIME của LLM Judge</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-5.png" alt="" width="85%"/>
</div>

Sau khi có điểm số của bốn chiều, chúng ta cần tổng hợp các điểm số này thành chỉ số đánh giá tổng thể. Chúng ta định nghĩa ba chỉ số quan trọng để đo mức độ chất lượng của đề được sinh ra:

<strong>Chỉ số đánh giá</strong>:

<strong>1. Điểm trung bình (Average Score)</strong>: tính điểm trung bình của tất cả đề trên bốn chiều, phản ánh mức độ chất lượng tổng thể của đề được sinh ra.
$$
\text{Average Score} = \frac{1}{N} \sum_{i=1}^{N} \frac{\sum_{d=1}^{4} S_{i,d}}{4}
$$

<strong>2. Tỷ lệ đạt (Pass Rate)</strong>: thống kê tỷ lệ đề có điểm trung bình đạt từ 3.5 trở lên, phản ánh mức độ bảo đảm chất lượng cơ bản của đề được sinh ra.

$$
\text{Pass Rate} = \frac{|\{i : \text{Score}_i \geq 3.5\}|}{N}
$$

<strong>3. Tỷ lệ xuất sắc (Excellent Rate)</strong>: thống kê tỷ lệ đề có điểm trung bình đạt từ 4.5 trở lên, phản ánh tỷ lệ đề chất lượng cao trong số đề được sinh ra.

$$
\text{Excellent Rate} = \frac{|\{i : \text{Score}_i \geq 4.5\}|}{N}
$$

trong đó:
- $N$ là tổng số đề được đánh giá
- $S_{i,d}$ là điểm của đề thứ $i$ ở chiều thứ $d$ (1-5 điểm)
- $\text{Score}_i$ là điểm trung bình của đề thứ $i$ (trung bình điểm của bốn chiều)

Ba chỉ số này phản ánh chất lượng sinh dữ liệu từ các góc độ khác nhau: Average Score cho biết mức độ tổng thể, Pass Rate đảm bảo chất lượng cơ bản, Excellent Rate đo năng lực tạo ra sản phẩm chất lượng cao.

<strong>(2) Đánh giá Win Rate</strong>

<strong>Động lực thiết kế</strong>: Mặc dù LLM Judge có thể cung cấp điểm số tuyệt đối đa chiều, nhưng chúng ta vẫn cần một chỉ số đánh giá tương đối để đo khoảng cách chất lượng giữa đề được sinh ra và đề thật. Đánh giá Win Rate thông qua so sánh theo cặp (pairwise comparison), để LLM trực tiếp phán đoán xem đề sinh ra và đề thật cái nào tốt hơn, cách so sánh tương đối này phù hợp với thói quen phán đoán của con người hơn so với chấm điểm tuyệt đối, cũng dễ phát hiện ưu nhược điểm tương đối của đề được sinh ra hơn. Trong trường hợp lý tưởng, nếu chất lượng đề được sinh ra gần với đề thật, Win Rate nên ở khoảng 50% (tức đề sinh ra và đề thật mỗi bên thắng 50%). Chỉ số này đơn giản trực quan, có thể nhanh chóng phán đoán mức độ chất lượng tổng thể của hệ thống sinh dữ liệu.

Trong hiện thực của chúng ta, đánh giá Win Rate được tiến hành theo quy trình như Hình 12.6 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-6.png" alt="" width="85%"/>
  <p>Hình 12.6 Sơ đồ quy trình đánh giá chất lượng sinh dữ liệu</p>
</div>

Trong đánh giá so sánh theo cặp, mỗi lần so sánh sẽ cho ra một trong ba kết quả có thể: đề được sinh ra thắng (Win), đề thật thắng (Loss) hoặc hòa (Tie). Chúng ta thống kê tỷ lệ của ba kết quả này để đánh giá chất lượng đề được sinh ra:

<strong>Chỉ số đánh giá</strong>:

<strong>1. Tỷ lệ thắng (Win Rate)</strong>: tỷ lệ đề được sinh ra được phán đoán là tốt hơn, phản ánh ưu thế của đề sinh ra so với đề thật.

$$
\text{Win Rate} = \frac{\text{Wins}}{\text{Total Comparisons}}
$$

<strong>2. Tỷ lệ thua (Loss Rate)</strong>: tỷ lệ đề thật được phán đoán là tốt hơn, phản ánh điểm yếu của đề sinh ra so với đề thật.

$$
\text{Loss Rate} = \frac{\text{Losses}}{\text{Total Comparisons}}
$$

<strong>3. Tỷ lệ hòa (Tie Rate)</strong>: tỷ lệ cả hai được phán đoán là chất lượng tương đương, phản ánh mức độ tương đồng giữa đề sinh ra và đề thật.

$$
\text{Tie Rate} = \frac{\text{Ties}}{\text{Total Comparisons}}
$$

trong đó, Total Comparisons là tổng số lần so sánh, Wins, Losses và Ties lần lượt là số lần đề được sinh ra thắng, thua và hòa. Ba chỉ số này thỏa mãn: Win Rate + Loss Rate + Tie Rate = 100%.

<strong>Kết quả lý tưởng</strong>: Win Rate ≈ 50% (cho thấy chất lượng sinh gần với đề thật). Nếu Win Rate thấp hơn đáng kể so với 50%, cho thấy chất lượng đề sinh ra không bằng đề thật, cần tối ưu chiến lược sinh; nếu Win Rate cao hơn đáng kể so với 50%, có thể cho thấy đề sinh ra đã vượt qua đề thật ở một số khía cạnh nào đó, hoặc tiêu chuẩn đánh giá tồn tại sai lệch.

<strong>(3) Xác minh thủ công (Human Verification)</strong>

<strong>Động lực thiết kế</strong>: Mặc dù LLM Judge và Win Rate có thể tự động đánh giá chất lượng đề, nhưng đối với nội dung cần reasoning logic nghiêm ngặt như đề toán, xác minh thủ công vẫn không thể thiếu. Đặc biệt khi đánh giá chất lượng sinh đáp án, cần chuyên gia con người verify tính chính xác của đáp án, tính đầy đủ của các bước giải và tính chặt chẽ của reasoning toán học. Ngoài ra, xác minh thủ công còn có thể phát hiện các vấn đề mà đánh giá tự động có thể bỏ sót, như tính sáng tạo, tính thú vị của đề, các yếu tố chủ quan khác. Để nâng cao hiệu quả và trải nghiệm của việc xác minh thủ công, chúng ta đã phát triển giao diện Web dựa trên Gradio, để người xác minh có thể dễ dàng duyệt đề, chấm điểm, gán nhãn trạng thái và thêm bình luận, giảm đáng kể rào cản của việc xác minh thủ công.

Trong hiện thực của chúng ta, xác minh thủ công được tiến hành qua các bước sau:

1. Đọc đề bài, đáp án, lời giải
2. Chấm điểm (1-5 điểm): tính đúng đắn, độ rõ ràng, mức độ phù hợp của độ khó, tính đầy đủ
3. Gán nhãn trạng thái:
   - ✅ approved (đã duyệt)
   - ❌ rejected (từ chối)
   - 🔄 needs_revision (cần chỉnh sửa)
4. Thêm bình luận

### 12.4.2 Kiến trúc hệ thống

Hệ thống sinh dữ liệu và đánh giá áp dụng thiết kế module hóa:

```
data_generation/
├── aime_generator.py              # Bộ sinh đề AIME
├── human_verification_ui.py       # Giao diện xác minh thủ công
├── run_complete_evaluation.py     # Quy trình đánh giá hoàn chỉnh
│
├── generated_data/                # Dữ liệu được sinh ra
│   ├── aime_generated_XXXXXX.json
│   └── generation_report_XXXXXX.md
│
└── evaluation_results/            # Kết quả đánh giá
    └── XXXXXX/
        ├── llm_judge/
        ├── win_rate/
        └── comprehensive_report.md
```

Hệ thống bao gồm bốn thành phần cốt lõi: đầu tiên là AIMEGenerator (bộ sinh đề), dùng framework HelloAgents để sinh đề phong cách AIME, hỗ trợ sinh hàng loạt và lưu tiến trình (progress), đồng thời tự động xử lý giới hạn tốc độ API (rate limit); tiếp theo là LLMJudgeTool (công cụ đánh giá LLM Judge), cung cấp đánh giá chất lượng 4 chiều, tự động sinh kết quả JSON và báo cáo Markdown; thứ ba là WinRateTool (công cụ đánh giá Win Rate), thông qua so sánh theo cặp để tính Win Rate, Loss Rate và Tie Rate; cuối cùng là HumanVerificationUI (giao diện xác minh thủ công), dựa trên giao diện Web Gradio, hỗ trợ chấm điểm và gán nhãn trạng thái.

### 12.4.3 Hiện thực bộ sinh đề AIME

```python
class AIMEGenerator:
    """AIME Problem Generator"""

    def __init__(
        self,
        llm: HelloAgentsLLM = None,
        delay_seconds: float = 1.0,
        use_reference_examples: bool = True,
        reference_dataset: str = "TianHongZXY/aime-1983-2025"
    ):
        self.llm = llm or HelloAgentsLLM()
        self.agent = SimpleAgent(
            name="AIME Generator",
            llm=self.llm,
            system_prompt="You are a professional mathematics competition problem designer."
        )
        self.delay_seconds = delay_seconds
        self.use_reference_examples = use_reference_examples

        # Load reference examples from 900+ AIME problems (1983-2025)
        if use_reference_examples:
            dataset = load_dataset(reference_dataset, split="test")
            self.reference_examples = list(dataset)
```
Mục tiêu của chúng ta là sinh ra bộ dữ liệu có phong cách tương tự, vì vậy chọn ngẫu nhiên mẫu tham khảo từ hơn 900 đề AIME thật (1983-2025)

Thiết kế prompt sinh đề (tiếng Anh):

```python
GENERATION_PROMPT = """You are a professional mathematics competition problem designer, skilled in creating AIME (American Invitational Mathematics Examination) style problems.

【Reference Example】(For style reference only, please generate a completely different problem)
Problem: {example_problem}
Answer: {example_answer}

AIME Problem Characteristics:
1. Answer: An integer between 0 and 999
2. Topics: Algebra, Geometry, Number Theory, Combinatorics, Probability, etc.
3. Style: Requires multi-step reasoning, but no advanced theory
4. Difficulty: Medium to hard (similar to AIME problems 6-9)

Please generate a **completely different** AIME-style mathematics problem, including:
1. Problem statement (clear and complete, different from the reference)
2. Answer (an integer between 0 and 999, different from the reference)
3. Detailed solution (including all reasoning steps)
4. Topic classification (Algebra/Geometry/Number Theory/Combinatorics/Probability)

Please output in the following JSON format:
{
    "problem": "Problem statement in English",
    "answer": 123,
    "solution": "Detailed solution steps in English",
    "topic": "Algebra"
}
"""
```
Chúng ta chọn sinh đề bằng tiếng Anh vì bốn lý do quan trọng: đầu tiên là để nhất quán với đề AIME thật (AIME là kỳ thi tiếng Anh, sinh đề bằng tiếng Anh hợp lý hơn), thứ hai là đảm bảo tính công bằng của đánh giá (khi LLM Judge đánh giá, tiếng Anh vs tiếng Anh sẽ công bằng hơn), thứ ba là thuận tiện cho việc quốc tế hóa (đề tiếng Anh có thể được sử dụng rộng rãi hơn), cuối cùng là tránh vấn đề dịch thuật (không cần lo lắng về độ chính xác khi dịch Trung-Anh).

Hiện thực sinh hàng loạt:

```python
def generate_and_save(self, num_problems: int = 30, output_dir: str = "data_generation/generated_data"):
    """Generate and save problems with intelligent delay"""
    # Clean old checkpoints
    for file in os.listdir(output_dir):
        if file.startswith("checkpoint_") and file.endswith(".json"):
            os.remove(os.path.join(output_dir, file))

    # Generate with tqdm progress bar
    with tqdm(total=num_problems, desc="Generating AIME problems", unit="problem") as pbar:
        last_call_time = 0

        for i in range(num_problems):
            # Ensure minimum delay between API calls
            if last_call_time > 0:
                elapsed = time.time() - last_call_time
                if elapsed < self.delay_seconds:
                    wait_time = self.delay_seconds - elapsed
                    time.sleep(wait_time)

            # Generate problem (randomly select reference example)
            start_time = time.time()
            problem = self.generate_single()
            last_call_time = time.time()
            generation_time = last_call_time - start_time

            # Update progress bar
            pbar.set_postfix({
                "topic": problem.get('topic', 'N/A'),
                "answer": problem.get('answer', 'N/A'),
                "time": f"{generation_time:.1f}s"
            })
            pbar.update(1)

    return generated_data_path
```
Hỗ trợ công thức toán LaTeX:

Đề AIME được sinh ra bao gồm công thức toán LaTeX (như `$\frac{a}{b}$`, `$\sqrt{x}$`), cần xử lý đặc biệt khi parse JSON:

```python
def _parse_response(self, response: str) -> Dict[str, Any]:
    """解析LLM响应（支持LaTeX数学公式）"""
    import re

    # 提取JSON部分
    if "```json" in response:
        json_str = response.split("```json")[1].split("```")[0].strip()
    else:
        json_str = response.strip()

    try:
        problem_data = json.loads(json_str)
    except json.JSONDecodeError:
        # 修复LaTeX转义问题：将 \frac 转为 \\frac
        # 正则表达式：找到未转义的反斜杠
        fixed_json_str = re.sub(r'(?<!\\)\\(?!["\\/bfnrtu])', r'\\\\', json_str)
        problem_data = json.loads(fixed_json_str)

    return problem_data
```
Dấu gạch chéo ngược (backslash) trong công thức LaTeX (như `\frac`, `\sqrt`) là ký tự escape bất hợp lệ trong JSON, sẽ khiến việc parse thất bại:
```
Invalid \escape: line 4 column 185 (char 375)
```

Thông qua biểu thức chính quy (regex), thay thế các backslash chưa được escape bằng backslash kép, giúp nó hợp lệ trong JSON.

### 12.4.4 Công cụ đánh giá LLM Judge

Công cụ LLM Judge dùng LLM làm giám khảo, đánh giá đa chiều cho đề được sinh ra.

```python
class LLMJudgeTool(Tool):
    """LLM Judge评估工具"""

    def run(self, params: Dict[str, Any]) -> str:
        """运行LLM Judge评估"""
        # 1. 加载生成数据
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. 加载参考数据（AIME 2025）
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. 创建评估器
        evaluator = LLMJudgeEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. 运行评估
        results = evaluator.evaluate_batch(gen_problems, max_samples=params.get("max_samples"))

        # 5. 保存结果
        evaluator.export_results(results, result_file)

        # 6. 生成报告
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```
**Prompt đánh giá**:

```python
EVALUATION_PROMPT = """请评估以下AIME数学题目的质量。

题目：
{problem}

答案：{answer}

解答：
{solution}

请从以下4个维度评分（1-5分）：

1. <strong>正确性 (Correctness)</strong>：数学逻辑是否正确，答案是否准确
2. <strong>清晰度 (Clarity)</strong>：问题表述是否清晰，解答是否易懂
3. <strong>难度匹配 (Difficulty Match)</strong>：难度是否符合AIME标准（中等偏难）
4. <strong>完整性 (Completeness)</strong>：解答步骤是否完整，是否包含必要的推理

请按以下JSON格式输出：
{
    "correctness": 5,
    "clarity": 4,
    "difficulty_match": 4,
    "completeness": 5,
    "comments": "评价理由"
}
"""
```

**Ví dụ báo cáo đánh giá**:

```markdown
# LLM Judge评估报告

## 总体评分

- <strong>平均总分</strong>: 4.2/5.0
- <strong>通过率</strong>: 85.0% (≥3.5分)
- <strong>优秀率</strong>: 40.0% (≥4.5分)

## 各维度评分

| 维度 | 平均分 | 评级 |
|------|--------|------|
| 正确性 | 4.3/5.0 | 良好 ⭐⭐⭐⭐ |
| 清晰度 | 4.1/5.0 | 良好 ⭐⭐⭐⭐ |
| 难度匹配 | 4.0/5.0 | 良好 ⭐⭐⭐⭐ |
| 完整性 | 4.4/5.0 | 良好 ⭐⭐⭐⭐ |
```

### 12.4.5 Công cụ đánh giá Win Rate

Công cụ Win Rate đánh giá chất lượng dữ liệu sinh ra so với đề thật thông qua so sánh theo cặp.

```python
class WinRateTool(Tool):
    """Win Rate评估工具"""

    def run(self, params: Dict[str, Any]) -> str:
        """运行Win Rate评估"""
        # 1. 加载生成数据
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. 加载参考数据（AIME 2025）
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. 创建评估器
        evaluator = WinRateEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. 运行评估
        results = evaluator.evaluate_win_rate(gen_problems, ref_problems, num_comparisons=params.get("num_comparisons"))

        # 5. 保存结果和报告
        evaluator.export_results(results, result_file)
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```
AIDataset chịu trách nhiệm nạp dữ liệu sinh ra và dữ liệu đề thật AIME, hỗ trợ hai loại dữ liệu:

```python
class AIDataset:
    """AI数据集加载器

    支持两种数据类型：
    1. generated: 生成的数据（JSON格式）
    2. real: AIME真题（从HuggingFace加载）
    """

    def __init__(
        self,
        dataset_type: str = "generated",
        data_path: Optional[str] = None,
        year: Optional[int] = None
    ):
        self.dataset_type = dataset_type
        self.data_path = data_path
        self.year = year  # 仅用于real类型，默认2025

    def load(self) -> List[Dict[str, Any]]:
        """加载数据集"""
        if self.dataset_type == "generated":
            return self._load_generated_data()
        elif self.dataset_type == "real":
            return self._load_real_data()

    def _load_real_data(self) -> List[Dict[str, Any]]:
        """从HuggingFace加载AIME 2025真题"""
        from huggingface_hub import snapshot_download

        # 使用AIME 2025数据集
        repo_id = "math-ai/aime25"

        # 下载数据集
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset"
        )

        # 读取JSONL文件
        data_file = list(Path(local_dir).glob("*.jsonl"))[0]
        data = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                if line.strip():
                    data.append(json.loads(line))

        # 统一数据格式（AIME 2025使用小写字段名）
        problems = []
        for idx, item in enumerate(data):
            problem = {
                "problem_id": item.get("id", f"aime_2025_{idx}"),
                "problem": item.get("problem", ""),
                "answer": item.get("answer", ""),
                "solution": item.get("solution", ""),  # AIME 2025没有solution字段
            }
            problems.append(problem)

        return problems
```
Chúng ta chọn chỉ dùng dataset AIME 2025 vì bốn lý do: đầu tiên là tính thời sự của dữ liệu (2025 là dữ liệu kỳ thi AIME mới nhất), thứ hai là đơn giản hóa việc bảo trì (chỉ bảo trì một dataset, code gọn gàng hơn), thứ ba là định dạng thống nhất (định dạng JSONL, tên field thống nhất chữ thường), cuối cùng là tính đại diện đầy đủ (30 đề đã đủ để đánh giá chất lượng sinh dữ liệu).

**Prompt so sánh**:

```python
COMPARISON_PROMPT = """请比较以下两个AIME数学题目的质量，判断哪个更好。

【题目A - 生成题目】
问题：{problem_a}
答案：{answer_a}
解答：{solution_a}

【题目B - AIME真题】
问题：{problem_b}
答案：{answer_b}
解答：{solution_b}

请从以下方面比较：
1. 数学逻辑的严谨性
2. 问题表述的清晰度
3. 难度的合理性
4. 解答的完整性

请按以下JSON格式输出：
{
    "winner": "A" 或 "B" 或 "Tie",
    "reason": "判断理由"
}
"""
```

**Ví dụ báo cáo đánh giá**:

```markdown
# Win Rate评估报告

## 胜率统计

| 指标 | 数值 | 百分比 |
|------|------|--------|
| 生成数据胜出 | 9次 | 45.0% |
| AIME真题胜出 | 8次 | 40.0% |
| 平局 | 3次 | 15.0% |

<strong>Win Rate</strong>: 45.0%

✅ <strong>良好</strong>: 生成数据质量接近参考数据（差距<10%）。
```

### 12.4.6 Giao diện xác minh thủ công

Dùng Gradio để tạo giao diện Web, hỗ trợ xác minh thủ công cho đề được sinh ra.

```python
class HumanVerificationUI:
    """人工验证界面"""

    def launch(self, share: bool = False):
        """启动Gradio界面"""
        with gr.Blocks(title="AIME题目人工验证") as demo:
            gr.Markdown("# 🎯 AIME题目人工验证系统")

            with gr.Row():
                with gr.Column(scale=2):
                    # 题目显示区域
                    problem_text = gr.Textbox(label="问题描述", lines=5, interactive=False)
                    answer_text = gr.Textbox(label="答案", interactive=False)
                    solution_text = gr.Textbox(label="解答过程", lines=10, interactive=False)

                with gr.Column(scale=1):
                    # 评分区域
                    correctness_slider = gr.Slider(1, 5, value=3, step=1, label="正确性")
                    clarity_slider = gr.Slider(1, 5, value=3, step=1, label="清晰度")
                    difficulty_slider = gr.Slider(1, 5, value=3, step=1, label="难度匹配")
                    completeness_slider = gr.Slider(1, 5, value=3, step=1, label="完整性")

                    # 状态选择
                    status_radio = gr.Radio(
                        choices=["approved", "rejected", "needs_revision"],
                        value="approved",
                        label="状态"
                    )

                    # 验证按钮
                    verify_btn = gr.Button("✅ 提交验证", variant="primary")

            demo.launch(share=share, server_name="127.0.0.1", server_port=7860)
```
**Cách sử dụng**:

```bash
# Khởi động giao diện xác minh thủ công
python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_XXXXXX.json

# Mở trình duyệt truy cập
http://127.0.0.1:7860
```

Kết quả cuối cùng có thể tham khảo Hình 12.7, đối với tính đúng đắn của đề bài, tốt nhất nên gán nhãn Review thủ công:
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-7.png" alt="" width="85%"/>
  <p>Hình 12.7 Trang xác minh thủ công đề thi AIME</p>
</div>

**Quy trình xác minh**:

1. Mở giao diện xác minh trên trình duyệt
2. Đọc đề bài, đáp án, lời giải
3. Chấm điểm theo 4 chiều (1-5 điểm)
4. Chọn trạng thái xác minh (approved/rejected/needs_revision)
5. Thêm bình luận (tùy chọn)
6. Nhấn "提交验证" (Submit xác minh)
7. Xem đề tiếp theo

**Lưu kết quả xác minh**:

Kết quả xác minh được tự động lưu thành `<data_path>_verifications.json`:

```json
{
  "gen_aime_1": {
    "problem_id": "gen_aime_1",
    "scores": {
      "correctness": 5,
      "clarity": 4,
      "difficulty_match": 4,
      "completeness": 5
    },
    "total_score": 4.5,
    "status": "approved",
    "comments": "题目质量很好，逻辑严谨",
    "verified_at": "2025-01-10T12:00:00"
  }
}
```

### 12.4.7 Quy trình đánh giá hoàn chỉnh

Tích hợp tất cả phương pháp đánh giá vào một quy trình hoàn chỉnh.

```python
def run_complete_evaluation(
    num_problems: int = 30,
    delay_seconds: float = 3.0
):
    """
    运行完整评估流程

    Args:
        num_problems: 生成题目数量
        delay_seconds: 每次生成之间的延迟（秒），避免API速率限制
    """
    # 步骤1: 生成AIME题目
    generator = AIMEGenerator(delay_seconds=delay_seconds)
    generated_data_path = generator.generate_and_save(
        num_problems=num_problems,
        output_dir="data_generation/generated_data"
    )

    # 步骤2: 评估
    # 创建评估结果目录
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    evaluation_dir = f"data_generation/evaluation_results/{timestamp}"
    os.makedirs(evaluation_dir, exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "llm_judge"), exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "win_rate"), exist_ok=True)

    # 创建LLM
    llm = HelloAgentsLLM()

    # 步骤2.1: LLM Judge评估
    llm_judge_result = None
    try:
        llm_judge_tool = LLMJudgeTool(llm=llm)
        llm_judge_result_json = llm_judge_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "max_samples": num_problems,
            "output_dir": os.path.join(evaluation_dir, "llm_judge"),
            "judge_model": "gpt-4o"
        })
        llm_judge_result = json.loads(llm_judge_result_json)
    except Exception as e:
        print(f"❌ LLM Judge评估失败: {e}")

    # 步骤2.2: Win Rate评估
    win_rate_result = None
    try:
        win_rate_tool = WinRateTool(llm=llm)
        win_rate_result_json = win_rate_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "num_comparisons": min(num_problems, 20),
            "output_dir": os.path.join(evaluation_dir, "win_rate"),
            "judge_model": "gpt-4o"
        })
        win_rate_result = json.loads(win_rate_result_json)
    except Exception as e:
        print(f"❌ Win Rate评估失败: {e}")

    # 步骤3: 生成综合报告
    comprehensive_report_path = None
    if llm_judge_result or win_rate_result:
        comprehensive_report_path = os.path.join(evaluation_dir, "comprehensive_report.md")
        report = generate_comprehensive_report(
            generated_data_path,
            llm_judge_result,
            win_rate_result
        )
        with open(comprehensive_report_path, 'w', encoding='utf-8') as f:
            f.write(report)

    return {
        "generated_data_path": generated_data_path,
        "llm_judge_result": llm_judge_result,
        "win_rate_result": win_rate_result,
        "comprehensive_report_path": comprehensive_report_path
    }
```
**Cách chạy**:

```bash
# Cách dùng cơ bản (mặc định delay 3 giây)
python data_generation/run_complete_evaluation.py 30

# Tùy chỉnh delay (khuyến nghị 3-5 giây, tránh rate limit của API)
python data_generation/run_complete_evaluation.py 30 3.0

# Giải thích tham số:
# - 30: số lượng đề được sinh ra
# - 3.0: thời gian delay giữa mỗi lần sinh (giây)

# Giải thích:
# - Giai đoạn sinh: chọn ngẫu nhiên mẫu tham khảo từ hơn 900 đề AIME thật (1983-2025)
# - Giai đoạn đánh giá: so sánh chất lượng với đề thật AIME năm 2025
# - Nguồn dataset: math-ai/aime25 (định dạng JSONL)
```

**Ví dụ output**:

```
================================================================================
🚀 AIME数据生成与评估完整流程
================================================================================

配置信息:
  - 生成题目数量: 30
  - API延迟: 3.0秒/题
  - 生成参考数据: TianHongZXY/aime-1983-2025（900+道题）
  - 评估参考: AIME 2025真题

================================================================================
📝 步骤1: 生成AIME题目
================================================================================
📚 加载AIME真题数据集: TianHongZXY/aime-1983-2025
   ✓ 已加载 963 道参考题目

🎯 开始生成AIME题目
   目标数量: 30
   生成模型: gpt-4o
   延迟设置: 3.0秒/题

生成AIME题目:  100%|██████████| 30/30 [01:30<00:00, 3.00s/题, 主题=Algebra, 答案=123, 耗时=3.0s]

✅ 步骤1完成！生成数据保存在: data_generation/generated_data/aime_generated_20250110_120000.json

🎯 步骤2.1: LLM Judge评估 (vs AIME 2025)

✅ LLM Judge评估完成！
   平均总分: 4.2/5.0
   通过率: 85.0%

🏆 步骤2.2: Win Rate评估 (vs AIME 2025)

✅ Win Rate评估完成！
   Win Rate: 45.0%

================================================================================
📊 步骤3: 生成综合报告
================================================================================

✅ 综合报告已保存: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

================================================================================
🎉 完整评估流程完成！
================================================================================

📁 输出文件:
   - 生成数据: data_generation/generated_data/aime_generated_20250110_120000.json
   - 评估结果目录: data_generation/evaluation_results/20250110_120000
   - LLM Judge报告: data_generation/evaluation_results/20250110_120000/llm_judge/llm_judge_report_20250110_120000.md
   - Win Rate报告: data_generation/evaluation_results/20250110_120000/win_rate/win_rate_report_20250110_120000.md
   - 综合报告: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

💡 下一步:
   1. 查看综合报告: data_generation/evaluation_results/20250110_120000/comprehensive_report.md
   2. 运行人工验证: python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_20250110_120000.json
```

### 12.4.8 Báo cáo đánh giá tổng hợp

Hệ thống tự động sinh báo cáo đánh giá tổng hợp, tổng hợp tất cả kết quả đánh giá. Dưới đây là ví dụ báo cáo:

```markdown
# AIME数据生成与评估综合报告

## 1. 基本信息

- <strong>生成时间</strong>: 2025-01-10 12:00:00
- <strong>生成题目数量</strong>: 30
- <strong>参考AIME年份</strong>: 2025

## 2. 数据生成统计

### 主题分布

| 主题 | 数量 | 占比 |
|------|------|------|
| 代数 | 10 | 33.3% |
| 几何 | 8 | 26.7% |
| 数论 | 7 | 23.3% |
| 组合 | 3 | 10.0% |
| 概率 | 2 | 6.7% |

## 3. LLM Judge评估结果

### 总体评分

- <strong>平均总分</strong>: 4.2/5.0
- <strong>通过率</strong>: 85.0% (≥3.5分)
- <strong>优秀率</strong>: 40.0% (≥4.5分)

### 各维度评分

| 维度 | 平均分 | 评级 |
|------|--------|------|
| 正确性 | 4.3/5.0 | 良好 ⭐⭐⭐⭐ |
| 清晰度 | 4.1/5.0 | 良好 ⭐⭐⭐⭐ |
| 难度匹配 | 4.0/5.0 | 良好 ⭐⭐⭐⭐ |
| 完整性 | 4.4/5.0 | 良好 ⭐⭐⭐⭐ |

## 4. Win Rate评估结果

### 胜率统计

| 指标 | 数值 | 百分比 |
|------|------|--------|
| 生成数据胜出 | 9次 | 45.0% |
| AIME真题胜出 | 8次 | 40.0% |
| 平局 | 3次 | 15.0% |

<strong>Win Rate</strong>: 45.0%

✅ <strong>良好</strong>: 生成数据质量接近参考数据（差距<10%）。

## 5. 综合结论

基于LLM Judge和Win Rate两种评估方法的结果：

1. <strong>LLM Judge评估</strong>: 生成数据的平均质量为 <strong>4.2/5.0</strong>
2. <strong>Win Rate评估</strong>: 生成数据相对于AIME 2025真题的胜率为 <strong>45.0%</strong>

✅ <strong>结论</strong>: 生成数据质量<strong>优秀</strong>，达到或超过AIME真题水平。可以用于实际应用。

## 6. 改进建议

- ✅ 继续保持当前的生成策略
- ✅ 可以考虑增加生成数量
- ✅ 建议进行人工验证以确保质量

## 7. 下一步行动

1. <strong>人工验证</strong>: 运行 `python data_generation/human_verification_ui.py <data_path>` 进行人工验证
2. <strong>查看详细结果</strong>:
   - LLM Judge详细报告
   - Win Rate详细报告
3. <strong>数据使用</strong>: 如果质量满意，可以将生成的数据用于训练或测试
```

Dựa trên kinh nghiệm sử dụng thực tế, tổng hợp các điểm sau:

Về mặt sinh dữ liệu, nên dùng thời gian delay phù hợp (2-3 giây) để tránh rate limit của API, bật tính năng lưu checkpoint để tránh mất mát khi bị gián đoạn, test với batch nhỏ trước (10 đề) để xác nhận không có vấn đề rồi mới sinh hàng loạt lớn, và định kỳ kiểm tra chất lượng sinh để kịp thời điều chỉnh prompt. Về chiến lược đánh giá, khuyến nghị kết hợp hai phương pháp LLM Judge và Win Rate, trong đó LLM Judge dùng cho đánh giá chất lượng tuyệt đối, Win Rate dùng cho so sánh chất lượng tương đối, xác minh thủ công dùng để kiểm soát chất lượng cuối cùng. Về tiêu chuẩn chất lượng, khuyến nghị điểm trung bình LLM Judge đạt từ 4.0/5.0 trở lên, Win Rate đạt từ 45% trở lên (gần với 50%), Pass Rate đạt từ 80% trở lên, tỷ lệ thông qua xác minh thủ công đạt từ 90% trở lên. Trong quá trình tối ưu lặp lại, nên điều chỉnh prompt sinh đề dựa trên kết quả đánh giá, phân tích các vấn đề chung của đề điểm thấp, tham khảo ưu điểm của đề điểm cao, liên tục cải thiện chiến lược sinh.

Thông qua việc học phần này, chúng ta đã nắm được cách dùng framework HelloAgents để đánh giá chất lượng sinh dữ liệu, bao gồm ba phương pháp: đánh giá LLM Judge, đánh giá Win Rate và xác minh thủ công. Hệ thống đánh giá hoàn chỉnh này có thể đảm bảo chất lượng cao của dữ liệu sinh ra, cung cấp hỗ trợ dữ liệu đáng tin cậy cho việc huấn luyện và kiểm thử hệ thống AI.

Đối với đánh giá LLM Judge và Win Rate, HelloAgents cũng đã tích hợp thành tool, và cung cấp code mẫu hoàn chỉnh. Nếu bạn quan tâm đến chi tiết hiện thực cụ thể của hai phương pháp đánh giá này, cũng có thể tham khảo code mẫu.




## 12.5 Tổng kết chương

Trong chương này, chúng ta đã xây dựng một hệ thống đánh giá hiệu năng hoàn chỉnh cho framework HelloAgents. Hãy cùng ôn lại những nội dung cốt lõi đã học:

<strong>(1) Tổng quan hệ thống đánh giá</strong>

Chúng ta đã xây dựng một hệ thống đánh giá ba tầng, bao phủ toàn diện các chiều năng lực khác nhau của Agent. Đầu tiên là đánh giá năng lực gọi tool (BFCL), tập trung đánh giá độ chính xác gọi function của Agent, bao gồm bốn category simple, multiple, parallel, irrelevance, dùng kỹ thuật AST matching để đánh giá chính xác. Tiếp theo là đánh giá năng lực tổng quát (GAIA), đánh giá năng lực giải quyết vấn đề tổng hợp của Agent, bao gồm ba mức độ khó với tổng cộng 466 câu hỏi thực tế, tập trung vào các năng lực multi-step reasoning, sử dụng tool, xử lý file. Thứ ba là đánh giá chất lượng sinh dữ liệu (AIME), đánh giá chất lượng dữ liệu được LLM sinh ra, dùng hai phương pháp LLM Judge và Win Rate, hỗ trợ xác minh thủ công và sinh báo cáo tổng hợp, đảm bảo dữ liệu sinh ra đạt tiêu chuẩn chất lượng của dữ liệu tham khảo.

<strong>(2) Các điểm kỹ thuật cốt lõi</strong>

Về mặt hiện thực kỹ thuật, chúng ta áp dụng sáu điểm kỹ thuật cốt lõi. Đầu tiên là thiết kế module hóa, hệ thống đánh giá áp dụng kiến trúc ba tầng: tầng dữ liệu (Dataset chịu trách nhiệm nạp và quản lý dữ liệu), tầng đánh giá (Evaluator chịu trách nhiệm thực thi quy trình đánh giá) và tầng chỉ số (Metrics chịu trách nhiệm tính toán các chỉ số đánh giá). Thứ hai là đóng gói thành tool, tất cả chức năng đánh giá đều được đóng gói thành Tool, có thể được Agent gọi trực tiếp, tích hợp vào workflow hoặc sử dụng qua interface thống nhất. Thứ ba là kỹ thuật AST matching, dùng cây cú pháp trừu tượng để so khớp function call, thông minh hơn so khớp chuỗi đơn giản, có thể bỏ qua thứ tự tham số, nhận diện biểu thức tương đương và bỏ qua khác biệt định dạng. Thứ tư là hỗ trợ đa phương thức, đánh giá GAIA hỗ trợ dữ liệu đa phương thức như câu hỏi văn bản, file đính kèm và input hình ảnh. Thứ năm là đánh giá LLM Judge, dùng LLM làm giám khảo để đánh giá chất lượng dữ liệu sinh ra, cung cấp chấm điểm đa chiều (tính đúng đắn, độ rõ ràng, mức độ phù hợp của độ khó, tính đầy đủ), quy trình đánh giá tự động, báo cáo đánh giá chi tiết, và hỗ trợ tùy chỉnh chiều đánh giá và tiêu chuẩn. Thứ sáu là đánh giá so sánh Win Rate, thông qua so sánh theo cặp để đánh giá chất lượng sinh (dữ liệu sinh ra vs dữ liệu tham khảo), do LLM phán đoán cái nào tốt hơn và tính thống kê tỷ lệ thắng, gần 50% nghĩa là chất lượng tương đương.

<strong>(3) Hướng mở rộng</strong>

Dựa trên hệ thống đánh giá của chương này, bạn có thể mở rộng theo bốn hướng. Đầu tiên là thêm benchmark đánh giá mới, có thể tham khảo mô hình hiện thực của BFCL và GAIA, hiện thực ba thành phần Dataset, Evaluator, Metrics, và đóng gói thành Tool để sử dụng. Thứ hai là tùy chỉnh chỉ số đánh giá, thêm phương thức tính chỉ số mới trong lớp Metrics, thiết kế chỉ số theo kịch bản ứng dụng cụ thể. Thứ ba là tích hợp vào quy trình CI/CD, tự động chạy đánh giá khi commit code, thiết lập ngưỡng hiệu năng để ngăn chặn suy giảm hiệu năng, sinh báo cáo đánh giá và lưu trữ. Thứ tư là mở rộng đánh giá sinh dữ liệu, hỗ trợ thêm nhiều loại dữ liệu (code, đối thoại, tài liệu, v.v.), thêm nhiều chiều đánh giá (tính sáng tạo, tính đa dạng, v.v.), tích hợp thêm nhiều dataset tham khảo, hỗ trợ đánh giá so sánh đa model.

<strong>Chúc mừng bạn đã hoàn thành Chương 12!</strong> 🎉

Đánh giá là khâu quan trọng trong phát triển Agent, nó giúp chúng ta có thể:

- Đo lường khách quan năng lực của Agent
- Phát hiện và sửa các vấn đề
- Liên tục cải thiện hệ thống

Trong chương tiếp theo, chúng ta sẽ khám phá cách áp dụng framework HelloAgents vào các dự án thực tế.

<strong>Tiếp tục cố gắng nhé!</strong> 💪

## Bài tập

> <strong>Gợi ý</strong>: Một số bài tập không có đáp án chuẩn, trọng tâm là rèn luyện cho người học sự hiểu biết toàn diện và năng lực thực hành về đánh giá hiệu năng Agent.

1. Chương này đã giới thiệu nhiều benchmark đánh giá Agent. Hãy phân tích:

   - Ở mục 12.1.2 đã giới thiệu các benchmark BFCL, GAIA, AgentBench, v.v. Hãy so sánh BFCL và GAIA: chúng lần lượt đánh giá những năng lực cốt lõi nào của Agent? Vì sao BFCL dùng thuật toán AST matching, còn GAIA dùng Quasi Exact Match? Hai phương pháp đánh giá này có ưu nhược điểm gì?
   - Giả sử bạn cần xây dựng một "hệ thống chăm sóc khách hàng thông minh", cần đánh giá các năng lực sau: (1) độ chính xác hiểu ý định người dùng; (2) tính đúng đắn khi gọi API backend; (3) tính thân thiện và chuyên nghiệp của câu trả lời; (4) tính bền vững khi xử lý tình huống bất thường. Hãy chọn hoặc thiết kế chỉ số và phương pháp đánh giá phù hợp cho mỗi năng lực.
   - Ở mục 12.1.1 có đề cập, đánh giá Agent đối mặt với ba thách thức lớn là "tính bất định của output", "sự đa dạng của tiêu chuẩn đánh giá", "chi phí đánh giá cao". Hãy đề xuất giải pháp cụ thể cho từng thách thức, và phân tích tính khả thi cũng như hạn chế của giải pháp.

2. BFCL (Berkeley Function Calling Leaderboard) là benchmark quan trọng để đánh giá năng lực gọi tool. Dựa trên nội dung mục 12.2, hãy suy nghĩ sâu hơn:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thực hiện trực tiếp

   - Trong thuật toán AST matching ở mục 12.2.3, chúng ta so sánh cây cú pháp trừu tượng để phán đoán function call có đúng hay không. Hãy phân tích: vì sao AST matching phù hợp hơn so khớp chuỗi đơn giản? Trong trường hợp nào AST matching có thể cho kết quả sai (false positive hoặc false negative)? Làm thế nào cải tiến thuật toán AST matching để nâng cao độ chính xác?
   - Dataset BFCL bao gồm bốn category simple, multiple, parallel, irrelevance. Hãy thiết kế 2-3 mẫu test mới cho mỗi category, yêu cầu có thể test các trường hợp biên hoặc kịch bản dễ gây lỗi của Agent trong category đó.
   - Dựa trên code ở mục 12.2.4, hãy mở rộng BFCL evaluator, thêm các chức năng sau: (1) hỗ trợ đánh giá thứ tự thực thi của các lệnh gọi tool (đối với nhiều lệnh gọi tool có quan hệ phụ thuộc); (2) đánh giá hiệu suất gọi tool (ví dụ có dùng số lần gọi ít nhất hay không); (3) sinh báo cáo phân tích lỗi chi tiết (ví dụ loại lỗi nào phổ biến nhất).

3. GAIA (General AI Assistants) đánh giá năng lực tổng hợp của Agent. Dựa trên nội dung mục 12.3, hãy hoàn thành các bài thực hành mở rộng sau:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thực hiện trực tiếp

   - Ở mục 12.3.2 đã giới thiệu ba mức độ khó của GAIA (Level 1/2/3). Hãy phân tích: ba mức này khác nhau như thế nào về độ phức tạp của task, năng lực cần thiết, tiêu chuẩn đánh giá? Nếu phải thiết kế Level 4 (độ khó siêu cao), nên bao gồm loại task nào?
   - GAIA dùng thuật toán "Quasi Exact Match" để đánh giá tính đúng đắn của đáp án. Hãy phân tích: phương pháp này xử lý tính đa dạng của đáp án như thế nào (ví dụ "42", "bốn mươi hai", "42.0" đều nên được xem là đúng)? Trong trường hợp nào Quasi Exact Match có thể không đủ dùng? Hãy thiết kế một thuật toán so khớp đáp án thông minh hơn, có thể xử lý các đáp án tương đương về ngữ nghĩa.
   - Dựa trên code ở mục 12.3.4, hãy hiện thực một "bộ đánh giá GAIA tùy chỉnh": chọn một lĩnh vực cụ thể (như y tế, pháp lý, tài chính), thiết kế 10 câu hỏi thực tế, và hiện thực quy trình đánh giá hoàn chỉnh. Yêu cầu các câu hỏi bao phủ nhiều mức độ khó khác nhau, và cung cấp đáp án chuẩn cùng tiêu chuẩn chấm điểm.

4. LLM Judge là phương pháp mới nổi dùng large language model để đánh giá. Dựa trên nội dung mục 12.4, hãy phân tích sâu:

   - Ở mục 12.4.2, chúng ta dùng GPT-4 làm giám khảo để đánh giá chất lượng câu trả lời của Agent. Hãy phân tích: LLM Judge có ưu thế gì so với so khớp quy tắc hoặc tính chỉ số truyền thống? Nó tồn tại những thiên lệch (bias) hoặc hạn chế tiềm ẩn nào (như thiên vị với một số phong cách trả lời nhất định, nhạy cảm với độ dài)?
   - Việc thiết kế tiêu chuẩn chấm điểm của LLM Judge rất quan trọng. Hãy thiết kế tiêu chuẩn chấm điểm chi tiết (bao gồm chiều đánh giá, trọng số, ví dụ) cho ba kịch bản đánh giá khác nhau sau: (1) đánh giá chất lượng sinh code; (2) đánh giá chất lượng viết sáng tạo; (3) đánh giá chất lượng tài liệu kỹ thuật.
   - Ở mục 12.4.3 có đề cập, có thể dùng nhiều LLM Judge để đánh giá theo kiểu "hội đồng giám khảo". Hãy thiết kế một "hệ thống đánh giá đa giám khảo": dùng 3-5 LLM khác nhau (như GPT-4, Claude, Qwen) làm giám khảo, làm thế nào để tổng hợp điểm số của chúng? Làm thế nào xử lý bất đồng giữa các giám khảo? Làm thế nào phát hiện và lọc bỏ điểm số bất thường?

5. Việc áp dụng thực tế của đánh giá Agent cần xem xét nhiều khía cạnh. Hãy suy nghĩ:

   - Trong dự án thực tế, việc đánh giá thường cần cân bằng giữa "chi phí đánh giá" và "chất lượng đánh giá". Hãy thiết kế một "chiến lược đánh giá phân tầng": (1) đánh giá nhanh (chi phí thấp, dùng cho iteration phát triển hàng ngày); (2) đánh giá chuẩn (chi phí trung bình, dùng trước khi release phiên bản); (3) đánh giá toàn diện (chi phí cao, dùng cho cập nhật lớn hoặc phát hành ra ngoài). Mỗi tầng nên bao gồm những hạng mục đánh giá nào? Thiết kế quy trình đánh giá như thế nào?
   - Hiệu năng của Agent có thể thay đổi theo thời gian (ví dụ API bên ngoài phụ thuộc thay đổi, nhu cầu người dùng thay đổi). Hãy thiết kế một "hệ thống đánh giá liên tục": có thể định kỳ tự động chạy đánh giá, giám sát xu hướng thay đổi hiệu năng của Agent, và kịp thời cảnh báo khi hiệu năng suy giảm. Hệ thống này nên bao gồm những thành phần nào? Thiết kế quy tắc cảnh báo như thế nào?
   - Kết quả đánh giá cần được trình bày rõ ràng cho các đối tượng khác nhau (như developer, product manager, người dùng). Hãy thiết kế một "hệ thống sinh báo cáo đánh giá": có thể tự động sinh báo cáo với mức độ chi tiết khác nhau tùy theo đối tượng. Báo cáo cho developer nên bao gồm những chi tiết kỹ thuật nào? Báo cáo cho product manager nên nhấn mạnh những chỉ số kinh doanh nào? Báo cáo cho người dùng nên được đơn giản hóa và trực quan hóa như thế nào?

## Tài liệu tham khảo

[1] Patil, S. G., Zhang, T., Wang, X., & Gonzalez, J. E. (2023). Gorilla: Large Language Model Connected with Massive APIs. arXiv preprint arXiv:2305.15334.

[2] Qin, Y., Liang, S., Ye, Y., Zhu, K., Yan, L., Lu, Y., ... & Sun, M. (2023). ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs. arXiv preprint arXiv:2307.16789.

[3] Li, M., Zhao, Y., Yu, B., Song, F., Li, H., Yu, H., ... & Li, Y. (2023). Api-bank: A comprehensive benchmark for tool-augmented llms. arXiv preprint arXiv:2304.08244.

[4] Mialon, G., Dessì, R., Lomeli, M., Nalmpantis, C., Pasunuru, R., Raileanu, R., ... & Scialom, T. (2023). GAIA: a benchmark for General AI Assistants. arXiv preprint arXiv:2311.12983.

[5] Liu, X., Yu, H., Zhang, H., Xu, Y., Lei, X., Lai, H., ... & Zhang, D. (2023). AgentBench: Evaluating LLMs as Agents. arXiv preprint arXiv:2308.03688.

[6] Zhou, S., Xu, F. F., Zhu, H., Zhou, X., Lo, R., Sridhar, A., ... & Neubig, G. (2023). WebArena: A Realistic Web Environment for Building Autonomous Agents. arXiv preprint arXiv:2307.13854.

[7] Chan, C. M., Chen, W., Su, Y., Yu, J., Xue, W., Zhang, S., ... & Liu, Z. (2023). ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate. arXiv preprint arXiv:2308.07201.

[8] Zhou, X., Zhu, H., Mathur, L., Zhang, R., Yu, H., Qi, Z., ... & Neubig, G. (2023). SOTOPIA: Interactive Evaluation for Social Intelligence in Language Agents. arXiv preprint arXiv:2310.11667.

[9] Mathematical Association of America. (2024). American Invitational Mathematics Examination (AIME). Retrieved from https://www.maa.org/math-competitions/invitational-competitions/aime
