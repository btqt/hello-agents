# Chương 11 Agentic RL

## 11.1 Từ huấn luyện LLM đến Agentic RL

Trong các chương trước, chúng ta đã triển khai nhiều paradigm Agent và giao thức giao tiếp khác nhau. Tuy nhiên khi Agent xử lý các tác vụ phức tạp hơn thì hiệu suất lại không tốt, tự nhiên sẽ nảy sinh câu hỏi: <strong>Làm thế nào để Agent có khả năng suy luận (reasoning) mạnh hơn? Làm thế nào để Agent học cách sử dụng tool tốt hơn? Làm thế nào để Agent có thể tự cải thiện (self-improvement)?</strong>

Đây chính là vấn đề cốt lõi mà Agentic RL (huấn luyện Agent dựa trên Reinforcement Learning) cần giải quyết. Chương này sẽ giới thiệu khả năng huấn luyện Reinforcement Learning cho framework HelloAgents, giúp bạn có thể huấn luyện ra những Agent có khả năng reasoning, sử dụng tool và các năng lực nâng cao khác. Chúng ta sẽ bắt đầu từ kiến thức cơ bản về huấn luyện LLM, từng bước đi sâu vào các kỹ thuật thực dụng như Supervised Fine-Tuning (SFT), Group Relative Policy Optimization (GRPO), và cuối cùng xây dựng một pipeline huấn luyện Agent hoàn chỉnh.

### 11.1.1 Từ Reinforcement Learning đến Agentic RL

Trong mục 2.4.2 của Chương 2, chúng ta đã giới thiệu về Agent dựa trên Reinforcement Learning. Reinforcement Learning (RL) là một paradigm học tập tập trung giải quyết các bài toán ra quyết định tuần tự (sequential decision), thông qua việc Agent tương tác trực tiếp với môi trường (environment), "thử và sai" (trial and error) để học cách tối đa hóa lợi ích dài hạn.

Bây giờ, hãy áp dụng framework này vào LLM Agent. Hãy xem xét một Agent giải bài toán toán học, nó cần trả lời câu hỏi như sau:

```
问题: Janet's ducks lay 16 eggs per day. She eats three for breakfast
every morning and bakes muffins for her friends every day with four.
She sells the remainder at the farmers' market daily for $2 per fresh
duck egg. How much in dollars does she make every day at the farmers' market?
```

Bài toán này cần reasoning nhiều bước: trước tiên tính số trứng còn lại mỗi ngày của Janet (16 - 3 - 4 = 9), sau đó tính thu nhập của cô ấy (9 × 2 = 18). Chúng ta có thể ánh xạ tác vụ này vào framework Reinforcement Learning:

- <strong>Agent</strong>: Hệ thống reasoning dựa trên LLM
- <strong>Environment</strong>: Bài toán toán học và hệ thống kiểm chứng
- <strong>State</strong>: Mô tả bài toán hiện tại và các bước reasoning đã có
- <strong>Action</strong>: Sinh ra bước reasoning tiếp theo hoặc đáp án cuối cùng
- <strong>Reward</strong>: Đáp án có đúng hay không (đúng +1, sai 0)

Phương pháp supervised learning truyền thống tồn tại ba hạn chế cốt lõi: một là chất lượng dữ liệu quyết định hoàn toàn chất lượng huấn luyện, model chỉ có thể bắt chước dữ liệu huấn luyện, khó vượt qua; hai là thiếu khả năng khám phá (exploration), chỉ có thể học thụ động theo con đường do con người cung cấp; ba là khó tối ưu hóa mục tiêu dài hạn, không thể tối ưu hóa chính xác các bước trung gian của reasoning nhiều bước.

Reinforcement Learning mang lại khả năng mới. Bằng cách để Agent tự chủ sinh ra nhiều đáp án ứng viên và nhận reward dựa trên tính chính xác, nó có thể học được con đường reasoning nào tốt hơn, bước nào là then chốt, thậm chí phát hiện ra phương pháp giải bài toán tốt hơn cả dữ liệu gán nhãn của con người<sup>[8]</sup>. Đây chính là ý tưởng cốt lõi của Agentic RL: coi LLM như một policy có thể học được, nhúng vào vòng lặp cảm nhận-quyết định-thực thi (perception-decision-execution) của Agent, thông qua Reinforcement Learning tối ưu hóa hiệu suất tác vụ nhiều bước.

### 11.1.2 Bức tranh toàn cảnh về huấn luyện LLM

Trước khi đi sâu vào Agentic RL, chúng ta cần hiểu rõ toàn bộ quy trình huấn luyện LLM. Sự ra đời của một LLM mạnh mẽ (như GPT, Claude, Qwen) thường phải trải qua hai giai đoạn chính: Pretraining (tiền huấn luyện) và Post-training (hậu huấn luyện). Như Hình 11.1 thể hiện, hai giai đoạn này tạo thành con đường tiến hóa hoàn chỉnh của LLM từ "language model" đến "trợ lý hội thoại".

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-1.png" alt="" width="85%"/>
  <p>Hình 11.1 Bức tranh toàn cảnh về huấn luyện LLM</p>
</div>

<strong>Giai đoạn Pretraining</strong> là giai đoạn đầu tiên của huấn luyện LLM, mục tiêu là để model học các quy luật cơ bản của ngôn ngữ và kiến thức thế giới. Giai đoạn này sử dụng lượng dữ liệu văn bản khổng lồ (thường ở mức vài TB), huấn luyện model theo phương thức self-supervised learning: tín hiệu huấn luyện đến từ chính văn bản, ví dụ dự đoán từ tiếp theo dựa vào đoạn văn phía trước. Tác vụ pretraining phổ biến nhất là Causal Language Modeling, còn gọi là Next Token Prediction.

Cho một chuỗi văn bản $x_1, x_2, ..., x_t$, model cần dự đoán từ tiếp theo $x_{t+1}$:

$$
\mathcal{L}_{\text{pretrain}} = -\sum_{t=1}^{T} \log P(x_t | x_1, x_2, ..., x_{t-1}; \theta)
$$

Trong đó $\theta$ là tham số model, $P(x_t | x_1, ..., x_{t-1}; \theta)$ là phân phối xác suất của từ tiếp theo mà model dự đoán, mục tiêu là tối thiểu hóa negative log-likelihood, tức tối đa hóa xác suất dự đoán đúng từ. Ví dụ, cho văn bản "The cat sat on the", model cần dự đoán từ tiếp theo có khả năng cao nhất là "mat". Thông qua việc huấn luyện như vậy trên lượng văn bản khổng lồ, model dần dần học được quy tắc ngữ pháp (thứ tự từ như thế nào là hợp lệ), kiến thức ngữ nghĩa (quan hệ giữa các từ), kiến thức thế giới (thông tin thực tế về thế giới) cũng như khả năng reasoning cơ bản.

Đặc điểm của giai đoạn Pretraining là lượng dữ liệu khổng lồ, chi phí tính toán cao, những gì học được là khả năng hiểu và sinh ngôn ngữ mang tính tổng quát, quá trình huấn luyện thường sử dụng mục tiêu self-supervised được tự động xây dựng từ văn bản chưa gán nhãn.

<strong>Giai đoạn Post-training</strong> thì nhằm giải quyết những thiếu sót của model đã pretrain. Model sau pretraining tuy đã có khả năng ngôn ngữ mạnh mẽ, nhưng nó chỉ là một model "dự đoán từ tiếp theo", không biết cách tuân theo chỉ dẫn (instruction) của con người, sinh ra câu trả lời hữu ích-vô hại-trung thực, từ chối các yêu cầu không phù hợp, cũng như tương tác với con người theo kiểu hội thoại. Giai đoạn post-training chính là để giải quyết những vấn đề này, giúp model căn chỉnh (align) với sở thích và giá trị quan của con người.

Post-training thường bao gồm ba bước. Bước một là <strong>Supervised Fine-Tuning (SFT)</strong><sup>[15]</sup>, mục tiêu là giúp model học cách tuân theo instruction và định dạng hội thoại. Dữ liệu huấn luyện là các cặp (prompt, completion), mục tiêu huấn luyện tương tự như pretraining, vẫn là tối đa hóa xác suất của output đúng:

$$
\mathcal{L}_{\text{SFT}} = -\sum_{i=1}^{N} \log P(y_i | x_i; \theta)
$$

Trong đó $x_i$ là prompt đầu vào, $y_i$ là output mong muốn, $N$ là số lượng mẫu huấn luyện. Đặc điểm của SFT là lượng dữ liệu nhỏ, cần gán nhãn thủ công, có hiệu quả nhanh, chủ yếu học định dạng tác vụ và năng lực cơ bản.

Bước hai là <strong>Reward Modeling (RM)</strong>. Model sau SFT tuy có thể tuân theo instruction, nhưng chất lượng câu trả lời sinh ra lại không đồng đều. Chúng ta cần một cách để đánh giá chất lượng câu trả lời, đây chính là vai trò của Reward Model<sup>[13,14]</sup>. Dữ liệu huấn luyện của Reward Model là dữ liệu so sánh sở thích (preference), chứa hai câu trả lời cho cùng một câu hỏi, một câu tốt hơn (chosen), một câu kém hơn (rejected). Mục tiêu huấn luyện của Reward Model là học sở thích của con người:

$$
\mathcal{L}_{\text{RM}} = -\mathbb{E}_{(x, y_w, y_l)} [\log \sigma(r_\phi(x, y_w) - r_\phi(x, y_l))]
$$

Trong đó $r_\phi(x, y)$ là Reward Model, đầu vào là cặp (prompt, câu trả lời), đầu ra là điểm chất lượng; $y_w$ là câu trả lời tốt hơn (chosen), $y_l$ là câu trả lời kém hơn (rejected), $\sigma$ là hàm sigmoid, mục tiêu là để Reward Model cho câu trả lời tốt hơn điểm số cao hơn.

Bước ba là <strong>Reinforcement Learning Fine-Tuning</strong>. Khi đã có Reward Model, chúng ta có thể dùng Reinforcement Learning để tối ưu hóa language model, giúp nó sinh ra câu trả lời chất lượng cao hơn. Thuật toán kinh điển nhất là PPO (Proximal Policy Optimization)<sup>[1]</sup>, mục tiêu huấn luyện là:

$$
J_{\text{PPO}} = \mathbb{E}_{x, y \sim \pi_\theta} [r_\phi(x, y)] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

Trong đó $\pi_\theta$ là policy hiện tại, tức language model, $\pi_{\text{ref}}$ là policy tham chiếu (reference policy), trong kịch bản này có thể là model SFT, $r_\phi(x, y)$ là điểm số của Reward Model, $D_{KL}$ là KL divergence, mục đích là ngăn model lệch quá xa, $\beta$ là hệ số cân bằng. Ý nghĩa của hàm mục tiêu này là: tối đa hóa reward, đồng thời không lệch quá xa model gốc.

RLHF (Reinforcement Learning from Human Feedback)<sup>[5]</sup> truyền thống cần lượng lớn dữ liệu preference gán nhãn thủ công, chi phí rất cao. Để giảm chi phí, các nhà nghiên cứu đã đề xuất RLAIF (Reinforcement Learning from AI Feedback)<sup>[7]</sup>, dùng một model AI mạnh (như GPT-4) để thay thế người gán nhãn. Quy trình làm việc của RLAIF là: dùng model SFT sinh ra nhiều câu trả lời ứng viên, dùng model AI mạnh để chấm điểm và xếp hạng các câu trả lời, dùng điểm số của AI để huấn luyện Reward Model, dùng Reward Model để thực hiện Reinforcement Learning. Thực nghiệm cho thấy, hiệu quả của RLAIF gần bằng, thậm chí vượt qua RLHF, đồng thời chi phí giảm đáng kể<sup>[11]</sup>.

### 11.1.3 Tư tưởng cốt lõi của Agentic RL

Sau khi hiểu quy trình huấn luyện cơ bản của LLM, hãy cùng xem sự khác biệt giữa Agentic RL và phương pháp huấn luyện truyền thống. Post-training truyền thống (chúng ta gọi là PBRFT: Preference-Based Reinforcement Fine-Tuning) chủ yếu tập trung tối ưu hóa chất lượng hội thoại đơn lượt (single-turn): cho một câu hỏi của người dùng, model sinh ra một câu trả lời, sau đó nhận reward dựa trên chất lượng câu trả lời. Cách làm này phù hợp để tối ưu hóa trợ lý hội thoại, nhưng đối với các tác vụ Agent cần reasoning nhiều bước, sử dụng tool, lập kế hoạch dài hạn thì lại tỏ ra bất lực.

<strong>Agentic RL</strong> là một paradigm mới, nó coi LLM như một policy có thể học được, nhúng vào trong một vòng lặp ra quyết định tuần tự (sequential decision loop). Trong framework này, Agent cần tương tác với thế giới bên ngoài trong môi trường động, thực thi hành động nhiều bước để hoàn thành tác vụ phức tạp, nhận feedback trung gian để chỉ dẫn quyết định tiếp theo, tối ưu hóa tổng reward tích lũy dài hạn thay vì reward đơn bước.

Hãy cùng hiểu sự khác biệt này qua một ví dụ cụ thể. Trong kịch bản PBRFT, người dùng hỏi "hãy giải thích Reinforcement Learning là gì", model sinh ra câu trả lời hoàn chỉnh, sau đó chấm điểm trực tiếp dựa trên chất lượng câu trả lời. Còn trong kịch bản Agentic RL, người dùng yêu cầu "giúp tôi phân tích chất lượng code của repository GitHub này", Agent cần trải qua nhiều bước: trước tiên gọi GitHub API để lấy thông tin repository, thành công lấy được cấu trúc repository và danh sách file, nhận được reward +0.1; sau đó đọc các file code chính, thành công lấy được nội dung code, nhận được reward +0.1; tiếp theo phân tích chất lượng code hợp lý, nhận được reward +0.2; cuối cùng sinh báo cáo phân tích chất lượng cao, nhận được reward +0.6. Tổng reward là tích lũy của tất cả các bước: 1.0.

Có thể thấy, đặc điểm then chốt của Agentic RL là tương tác nhiều bước, mỗi hành động đều làm thay đổi state của môi trường, mỗi bước đều có thể nhận được feedback, tối ưu hóa chất lượng hoàn thành của toàn bộ tác vụ.

Reinforcement Learning thường dùng framework Markov Decision Process (MDP) để hình thức hóa. MDP được định nghĩa bởi bộ ngũ $(S, A, P, R, \gamma)$: không gian state $S$, không gian action $A$, hàm chuyển trạng thái $P(s'|s,a)$, hàm reward $R(s,a)$ và hệ số chiết khấu (discount factor) $\gamma$. Bảng dưới đây so sánh PBRFT và Agentic RL theo góc nhìn MDP.

<div align="center">
  <p>Bảng 11.1 So sánh PBRFT và Agentic RL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-1.png" alt="" width="85%"/>
</div>

Về state, state $s_0$ của PBRFT chỉ được tạo thành từ prompt của người dùng, phạm vi thời gian $T=1$ (một bước), state không thay đổi, có thể biểu diễn là $s_0 = \text{prompt}$. Trong khi state $s_t$ của Agentic RL chứa quan sát lịch sử (historical observation) và context, phạm vi thời gian $T \gg 1$ (nhiều bước), state tiến hóa theo hành động, có thể biểu diễn là $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$, trong đó $o_t$ là quan sát (observation) của bước thứ $t$ (như kết quả trả về của tool, feedback từ môi trường, v.v.).

Về action, không gian action của PBRFT chỉ có sinh văn bản (text generation), là loại hành động duy nhất, biểu diễn là $a = y \sim \pi_\theta(y|s_0)$. Còn không gian action của Agentic RL chứa nhiều loại như sinh văn bản, gọi tool, thao tác môi trường, biểu diễn là $a_t \in \{a_t^{\text{text}}, a_t^{\text{tool}}\}$, ví dụ $a_t^{\text{text}}$ là sinh quá trình suy nghĩ hoặc câu trả lời, $a_t^{\text{tool}}$ là gọi calculator, search engine, hoặc các tool khác.

Về hàm chuyển trạng thái, PBRFT không có chuyển trạng thái, biểu diễn là $P(s'|s,a) = \delta(s' - s_{\text{terminal}})$. Còn state của Agentic RL thay đổi động theo hành động và môi trường, biểu diễn là $s_{t+1} \sim P(s_{t+1}|s_t, a_t)$, ví dụ sau khi gọi search tool, state sẽ chứa kết quả tìm kiếm.

Về reward, PBRFT chỉ có reward đơn bước $r(s_0, a)$, chỉ được cấp khi tác vụ kết thúc, biểu diễn là $R_{\text{PBRFT}} = r(s_0, y)$, thường do Reward Model cấp: $r(s_0, y) = r_\phi(s_0, y)$. Còn Agentic RL có reward nhiều bước $r(s_t, a_t)$, có thể cấp một phần reward ở các bước trung gian, biểu diễn là:

$$
R_{\text{Agentic}} = \sum_{t=0}^{T} \gamma^t r(s_t, a_t)
$$

Trong đó $\gamma \in [0,1]$ là discount factor, $r(s_t, a_t)$ có thể là sparse reward (chỉ cấp khi tác vụ hoàn thành, ví dụ đáp án đúng +1), dense reward (mỗi bước đều cấp, ví dụ gọi tool thành công +0.1) hoặc hybrid reward kết hợp cả hai.

Về hàm mục tiêu, PBRFT tối đa hóa expected reward đơn bước:

$$
J_{\text{PBRFT}}(\theta) = \mathbb{E}_{s_0, y \sim \pi_\theta} [r(s_0, y)]
$$

Còn Agentic RL tối đa hóa cumulative discounted reward:

$$
J_{\text{Agentic}}(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

Trong đó $\tau = (s_0, a_0, s_1, a_1, ..., s_T)$ là trajectory hoàn chỉnh.

Sự chuyển đổi này không chỉ đơn thuần là khác biệt về chi tiết kỹ thuật, mà là sự chuyển đổi căn bản về tư duy. Tư duy PBRFT tập trung vào "làm thế nào để model sinh ra một câu trả lời đơn lẻ tốt hơn", tối ưu hóa chất lượng câu trả lời, quan tâm đến cách diễn đạt ngôn ngữ, ra quyết định đơn bước. Còn tư duy Agentic RL tập trung vào "làm thế nào để Agent hoàn thành tác vụ phức tạp", tối ưu hóa mức độ hoàn thành tác vụ, quan tâm đến chiến lược hành động, lập kế hoạch nhiều bước. Sự chuyển đổi này khiến LLM tiến hóa từ "trợ lý hội thoại" thành "Agent tự chủ", có khả năng chủ động tìm kiếm thông tin, biết khi nào, cách nào sử dụng tool bên ngoài, vì mục tiêu cuối cùng mà sẵn sàng thực hiện các bước trung gian có vẻ "vòng vo", và học hỏi từ sai lầm.

Mục tiêu của Agentic RL là trao cho LLM Agent sáu năng lực cốt lõi, như Hình 11.2 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-2.png" alt="" width="85%"/>
  <p>Hình 11.2 Sáu năng lực cốt lõi của Agentic RL</p>
</div>

<strong>Reasoning</strong> là quá trình suy ra kết luận một cách logic từ thông tin đã cho, là năng lực cốt lõi của Agent. Phương pháp CoT prompting truyền thống phụ thuộc vào các ví dụ few-shot, khả năng tổng quát hóa (generalization) hạn chế; SFT chỉ có thể bắt chước các mẫu reasoning trong dữ liệu huấn luyện, khó sáng tạo. Ưu điểm của Reinforcement Learning là thông qua thử-sai để học các chiến lược reasoning hiệu quả, phát hiện ra các con đường reasoning không có trong dữ liệu huấn luyện, học được khi nào cần suy nghĩ sâu, khi nào có thể trả lời nhanh. Tác vụ reasoning có thể được mô hình hóa như bài toán ra quyết định tuần tự, cho bài toán $q$, Agent cần sinh ra chuỗi reasoning (reasoning chain) $c = (c_1, c_2, ..., c_n)$ và đáp án cuối cùng $a$. Hàm reward thường được thiết kế là $r(q, c, a) = 1$ nếu $a = a^*$ ngược lại $0$, mục tiêu huấn luyện là $\max_\theta \mathbb{E}_{q, (c,a) \sim \pi_\theta} [r(q, c, a)]$. Thông qua cách này, model học được cách sinh ra chuỗi reasoning chất lượng cao, chứ không chỉ đơn thuần ghi nhớ đáp án.

<strong>Tool Use</strong> là khả năng Agent gọi tool bên ngoài để hoàn thành tác vụ. Trong tác vụ tool use, không gian action được mở rộng thành $a_t \in \{a_t^{\text{think}}, a_t^{\text{tool}}\}$, trong đó $a_t^{\text{think}}$ là sinh quá trình suy nghĩ, $a_t^{\text{tool}} = (\text{tool\_name}, \text{arguments})$ là gọi tool. Reinforcement Learning giúp Agent học khi nào cần dùng tool, chọn tool nào, cách kết hợp nhiều tool. Ví dụ, khi giải bài toán toán học, Agent cần học khi nào dùng calculator, khi nào dùng code interpreter, khi nào reasoning trực tiếp.

<strong>Memory</strong> là khả năng Agent duy trì và tái sử dụng thông tin quá khứ, vô cùng quan trọng với các tác vụ dài hạn. Context window của LLM có hạn, chiến lược retrieval tĩnh (như RAG) không thể tối ưu hóa theo tác vụ. Reinforcement Learning giúp Agent học chiến lược quản lý memory: quyết định thông tin nào đáng nhớ, khi nào cập nhật memory, khi nào xóa thông tin lỗi thời. Điều này tương tự như working memory của con người, chúng ta chủ động quản lý thông tin trong não bộ, giữ lại cái quan trọng, quên đi cái không liên quan.

<strong>Planning</strong> là khả năng lập chuỗi hành động để đạt được mục tiêu. CoT truyền thống là tư duy tuyến tính, không thể quay lui (backtrack); prompt engineering sử dụng template planning tĩnh, khó thích ứng với tình huống mới. Reinforcement Learning giúp Agent học planning động: thông qua thử-sai phát hiện chuỗi hành động hiệu quả, học cách cân bằng lợi ích ngắn hạn và dài hạn. Ví dụ, trong tác vụ nhiều bước, Agent có thể cần thực hiện trước một số bước có vẻ "vòng vo", như thu thập thông tin, mới có thể hoàn thành tác vụ cuối cùng.

<strong>Self-Improvement</strong> là khả năng Agent nhìn lại output của chính mình, sửa lỗi và tối ưu hóa chiến lược. Reinforcement Learning giúp Agent học tự phản tư (self-reflect): nhận diện lỗi của bản thân, phân tích nguyên nhân thất bại, điều chỉnh chiến lược. Năng lực này khiến Agent có thể tự cải thiện liên tục mà không cần can thiệp của con người, tương tự như việc con người "học từ sai lầm".

<strong>Perception</strong> là khả năng hiểu thông tin đa phương thức (multimodal). Ví dụ, Reinforcement Learning có thể nâng cao khả năng visual reasoning, giúp model học cách sử dụng visual tool, học visual planning. Điều này khiến Agent không chỉ hiểu văn bản, mà còn có thể hiểu và thao tác với thế giới hình ảnh.

### 11.1.4 Thiết kế Agentic RL của HelloAgents

Sau khi hiểu tư tưởng cốt lõi của Agentic RL, hãy cùng xem cách triển khai các năng lực này trong framework HelloAgents.

Về lựa chọn công nghệ, chúng ta tích hợp framework TRL (Transformer Reinforcement Learning)<sup>[9]</sup>, model được chọn là Qwen3-0.6B<sup>[10]</sup>. TRL là thư viện Reinforcement Learning của Hugging Face, ổn định, chức năng đầy đủ, dễ tích hợp. Qwen3-0.6B là small language model của Alibaba Cloud, với 0.6B tham số phù hợp để huấn luyện trên GPU phổ thông, hiệu suất tốt và mã nguồn mở miễn phí.

Module Agentic RL của HelloAgents áp dụng kiến trúc bốn tầng, như Hình 11.3 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-3.png" alt="" width="85%"/>
  <p>Hình 11.3 Kiến trúc Agentic RL của HelloAgents</p>
</div>

Tầng dưới cùng là <strong>tầng Dataset</strong>, chứa class <code>GSM8KDataset</code>, hàm <code>create_sft_dataset()</code> và hàm <code>create_rl_dataset()</code>, chịu trách nhiệm tải và chuyển đổi định dạng dữ liệu. Tầng thứ hai là <strong>tầng Reward Function</strong>, chứa base class <code>MathRewardFunction</code>, <code>AccuracyReward</code> (reward độ chính xác), <code>LengthPenaltyReward</code> (penalty độ dài), <code>StepReward</code> (reward theo bước), cùng các hàm tiện lợi <code>create_*_reward()</code>, chịu trách nhiệm định nghĩa thế nào là hành vi tốt. Tầng thứ ba là <strong>tầng Trainer</strong>, chứa <code>SFTTrainerWrapper</code> và <code>GRPOTrainerWrapper</code>, chịu trách nhiệm logic huấn luyện cụ thể và hỗ trợ LoRA. Tầng trên cùng là <strong>tầng Interface thống nhất</strong>, cung cấp <code>RLTrainingTool</code> - công cụ huấn luyện thống nhất, hỗ trợ bốn thao tác: <code>action="train"</code> (huấn luyện model), <code>action="load_dataset"</code> (tải dataset), <code>action="create_reward"</code> (tạo reward function), <code>action="evaluate"</code> (đánh giá model).

### 11.1.5 Ví dụ bắt đầu nhanh

Trước khi đi sâu học tập, hãy trải nghiệm nhanh toàn bộ quy trình huấn luyện. Do phần lý thuyết của chương này khá nhiều, phần thực hành lại có nhiều chỗ cần debug phức tạp, nên chúng ta không tập trung vào việc xây dựng tool mà tập trung học cách áp dụng. Trước tiên cài đặt framework HelloAgents:

```bash
# 安装HelloAgents框架(第11章版本)
pip install "hello-agents[rl]==0.2.5"

# 或者从源码安装
cd HelloAgents
pip install -e ".[rl]"
```

Sau đó chạy ví dụ huấn luyện nhanh:

```python
import sys
import json

from hello_agents.tools import RLTrainingTool

# 创建RL训练工具
rl_tool = RLTrainingTool()

# 1. 快速测试:SFT训练(10个样本，1个epoch)
sft_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/quick_test_sft",
    "max_samples": 10,      # 只用10个样本快速测试
    "num_epochs": 1,        # 只训练1轮
    "batch_size": 2,
    "use_lora": True        # 使用LoRA加速训练
})

sft_result = json.loads(sft_result_str)
print(f"\n✓ SFT训练完成,模型保存在: {sft_result['output_dir']}")

# 2. GRPO训练(5个样本,1个epoch)
grpo_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",  # 使用基础模型
    "output_dir": "./models/quick_test_grpo",
    "max_samples": 5,       # 只用5个样本快速测试
    "num_epochs": 1,
    "batch_size": 2,        # 必须能被num_generations(8)整除,使用2
    "use_lora": True
})

grpo_result = json.loads(grpo_result_str)
print(f"\n✓ GRPO训练完成,模型保存在: {grpo_result['output_dir']}")

# 3. 评估模型
eval_result_str = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/quick_test_grpo",
    "max_samples": 10,      # 在10个测试样本上评估
    "use_lora": True
})

eval_result = json.loads(eval_result_str)
print(f"\n✓ 评估完成:")
print(f"  - 准确率: {eval_result['accuracy']}")
print(f"  - 平均奖励: {eval_result['average_reward']}")
print(f"  - 测试样本数: {eval_result['num_samples']}")

print("\n" + "=" * 50)
print("🎉 恭喜!你已经完成了第一个Agentic RL模型的训练!")
print("=" * 50)
print(f"\n模型路径:")
print(f"  SFT模型: {sft_result['output_dir']}")
print(f"  GRPO模型: {grpo_result['output_dir']}")
```

Ví dụ nhanh này thể hiện quy trình huấn luyện hoàn chỉnh: huấn luyện SFT giúp model học định dạng reasoning cơ bản và kiểu hội thoại, huấn luyện GRPO thông qua Reinforcement Learning tối ưu hóa chiến lược reasoning nâng cao độ chính xác, đánh giá model trên test set để đánh giá hiệu quả huấn luyện. Ngoài ra, độ chính xác thấp sau khi chạy xong là hiện tượng bình thường, vì hiện tại model mới chỉ thấy 0.7% số mẫu huấn luyện, và chỉ chạy một epoch.

## 11.2 Dataset và Reward Function

Dataset và reward function là hai nền tảng của huấn luyện Reinforcement Learning. Dataset định nghĩa tác vụ mà Agent cần học, reward function định nghĩa thế nào là hành vi tốt. Trong mục này, chúng ta sẽ học cách chuẩn bị dữ liệu huấn luyện và thiết kế reward function.

### 11.2.1 Dataset reasoning toán học GSM8K

Toán học reasoning là tác vụ lý tưởng để đánh giá khả năng reasoning của LLM. Trước tiên, bài toán toán học có đáp án đúng rõ ràng, có thể đánh giá tự động, không cần gán nhãn thủ công hay reward model phức tạp. Thứ hai, giải bài toán toán học cần phân rã bài toán, suy luận từng bước, đây chính là kịch bản điển hình của reasoning nhiều bước. Cuối cùng, khả năng reasoning học được có thể chuyển giao sang các lĩnh vực khác, có khả năng tổng quát hóa rất tốt. Ngược lại, các tác vụ hỏi đáp mở (như "làm thế nào để học lập trình?") thì chất lượng đáp án khó đánh giá khách quan, cần lượng lớn gán nhãn thủ công.

GSM8K (Grade School Math 8K)<sup>[4]</sup> là một dataset bài toán toán học tiểu học chất lượng cao. Như Bảng 11.2 thể hiện, dataset chứa 7.473 mẫu huấn luyện và 1.319 mẫu kiểm tra, độ khó ở mức toán tiểu học (lớp 2-8), dạng bài là bài toán ứng dụng, cần 2-8 bước reasoning mới ra được đáp án.

<div align="center">
  <p>Bảng 11.2 Thống kê dataset GSM8K</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-2.png" alt="" width="85%"/>
</div>
Hãy xem một bài toán GSM8K điển hình:

```
问题: Natalia sold clips to 48 of her friends in April, and then she sold half 
      as many clips in May. How many clips did Natalia sell altogether in April 
      and May?

答案: Natalia sold 48/2 = <<48/2=24>>24 clips in May.
      Natalia sold 48+24 = <<48+24=72>>72 clips altogether in April and May.
      #### 72

最终答案: 72
```

Bài toán này cần hai bước reasoning: trước tiên tính số lượng bán được trong tháng 5 (một nửa của 48), sau đó tính tổng số (tháng 4 + tháng 5). Trong đáp án, `<<48/2=24>>` là ký hiệu đánh dấu bước tính toán trung gian, `#### 72` đánh dấu đáp án cuối cùng.

Dataset GSM8K cần được chuyển đổi sang các định dạng khác nhau, để phù hợp với các phương pháp huấn luyện khác nhau, như Hình 11.4 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-4.png" alt="" width="85%"/>
  <p>Hình 11.4 Chuyển đổi định dạng dữ liệu GSM8K</p>
</div>


Định dạng gốc đến trực tiếp từ dataset, chứa question (câu hỏi) và answer (đáp án, bao gồm các bước giải), phù hợp để con người đọc. Định dạng SFT dùng cho supervised fine-tuning, chuyển câu hỏi thành prompt dạng hội thoại, lấy toàn bộ lời giải làm completion. Ví dụ:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "completion": "Let me solve this step by step.\n\nStep 1: ...\n\nFinal Answer: 72<|im_end|>"
}
```

Điểm then chốt là sử dụng chat template của model (như ký hiệu `<|im_start|>` của Qwen), prompt chứa câu hỏi của người dùng, completion chứa toàn bộ quá trình giải và đáp án. Như vậy model có thể học cách định dạng output, cách reasoning từng bước.

Định dạng RL dùng cho Reinforcement Learning, chỉ cung cấp câu hỏi và đáp án đúng, không cung cấp quá trình giải. Ví dụ:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "ground_truth": "72"
}
```

Điểm then chốt là prompt giống với SFT, nhưng ground_truth chỉ chứa đáp án cuối cùng (dùng để tính reward), model cần tự sinh ra toàn bộ quá trình reasoning. Thiết kế này buộc model phải học cách tự reasoning, thay vì đơn thuần ghi nhớ đáp án.

Như Bảng 11.3 thể hiện, ba định dạng mỗi loại có công dụng riêng.

<div align="center">
  <p>Bảng 11.3 So sánh các định dạng dữ liệu</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-3.png" alt="" width="85%"/>
</div>
HelloAgents cung cấp hàm tải dataset tiện lợi. Hãy dùng code để tải và xem dataset:

```python
from hello_agents.tools import RLTrainingTool
import json

# 创建工具
rl_tool = RLTrainingTool()

# 1. 加载SFT格式数据集
sft_result = rl_tool.run({
    "action": "load_dataset",
    "format": "sft",
    "max_samples": 5  # 只加载5个样本查看
})
sft_data = json.loads(sft_result)

print(f"数据集大小: {sft_data['dataset_size']}")
print(f"数据格式: {sft_data['format']}")
print(f"样本字段: {sft_data['sample_keys']}")

# 2. 加载RL格式数据集
rl_result = rl_tool.run({
    "action": "load_dataset",
    "format": "rl",
    "max_samples": 5
})
rl_data = json.loads(rl_result)

print(f"数据集大小: {rl_data['dataset_size']}")
print(f"数据格式: {rl_data['format']}")
print(f"样本字段: {rl_data['sample_keys']}")
```

Có thể thấy, định dạng SFT chứa toàn bộ quá trình giải, dùng cho supervised learning; định dạng RL chỉ chứa đáp án cuối cùng, model cần tự sinh ra quá trình reasoning. Tham số `max_samples` kiểm soát số lượng mẫu được tải, giúp kiểm thử nhanh.

### 11.2.2 Thiết kế Reward Function

Reward function là cốt lõi của Reinforcement Learning, nó định nghĩa thế nào là "hành vi tốt". Một reward function tốt có thể dẫn dắt Agent học được policy đúng đắn, còn một reward function tồi có thể khiến huấn luyện thất bại hoặc học ra hành vi sai.

Trong Reinforcement Learning, reward function $r(s, a)$ hoặc $r(s, a, s')$ gán một giá trị reward số cho mỗi hành động của Agent. Mục tiêu của Agent là tối đa hóa cumulative reward:

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

Đối với tác vụ toán học reasoning, chúng ta có thể đơn giản hóa thành:

$$
r(q, a) = f(a, a^*)
$$

Trong đó $q$ là câu hỏi, $a$ là đáp án model sinh ra, $a^*$ là đáp án đúng, $f$ là hàm đánh giá.

Thiết kế reward function ảnh hưởng trực tiếp đến hiệu quả huấn luyện. Reward function tốt cần có khả năng định nghĩa rõ ràng thế nào là thành công, có khả năng cung cấp tín hiệu gradient, không gây ra phương sai (variance) quá lớn, dễ điều chỉnh và kết hợp. Reward function tồi có thể chỉ cấp reward khi tác vụ kết thúc, các bước trung gian không có feedback, tồn tại reward hacking khiến Agent tìm ra cách "gian lận" để nhận reward cao, nhiều mục tiêu mâu thuẫn lẫn nhau, phương sai quá lớn khiến huấn luyện không hội tụ.

HelloAgents cung cấp ba loại reward function tích hợp sẵn, có thể dùng riêng lẻ hoặc kết hợp, như Hình 11.5 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-5.png" alt="" width="85%"/>
  <p>Hình 11.5 Thiết kế Reward Function</p>
</div>
<strong>（1）Accuracy Reward</strong>

Accuracy Reward (AccuracyReward) là reward function cơ bản nhất, nó chỉ quan tâm đáp án có đúng hay không. Định nghĩa toán học là:

$$
r_{\text{acc}}(a, a^*) = \begin{cases}
1 & \text{if } a = a^* \\
0 & \text{otherwise}
\end{cases}
$$

Trong đó $a$ là đáp án model sinh ra, $a^*$ là đáp án đúng. Đây là reward function nhị phân, đáp án đúng được 1 điểm, sai được 0 điểm.

Khi triển khai cần xử lý việc trích xuất và so sánh đáp án. Output của model có thể chứa lượng lớn văn bản, chúng ta cần trích xuất ra đáp án cuối cùng. Các phương pháp trích xuất phổ biến bao gồm: tìm số sau "Final Answer:", tìm số sau ký hiệu "####", dùng regular expression để trích xuất số cuối cùng. Khi so sánh đáp án cần xử lý độ chính xác số (như 72.0 và 72 nên được coi là giống nhau), chuyển đổi đơn vị (như 1000 và 1k), khác biệt định dạng (như "72" và "seventy-two").

Ví dụ sử dụng:

```python
from hello_agents.tools import RLTrainingTool
import json
rl_tool = RLTrainingTool()

# 创建准确率奖励函数
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
reward_data = json.loads(reward_result)

print(f"奖励类型: {reward_data['reward_type']}")
print(f"描述: {reward_data['description']}")

# 注意: RLTrainingTool的create_reward操作返回的是配置信息,
# 实际的奖励函数会在训练时自动创建和使用
```

Output:

```json
预测: 72, 真实: 72, 奖励: 1.0
预测: 72.0, 真实: 72, 奖励: 1.0
预测: 73, 真实: 72, 奖励: 0.0
```

Ưu điểm của Accuracy Reward là đơn giản trực tiếp, dễ hiểu và dễ triển khai, phù hợp với tác vụ có đáp án đúng rõ ràng. Nhược điểm là reward thưa (sparse reward), chỉ khi đáp án hoàn toàn đúng mới có reward, không thể phân biệt "gần đúng" và "hoàn toàn sai", có thể dẫn đến thiếu feedback hiệu quả ở giai đoạn đầu huấn luyện.

<strong>（2）Length Penalty</strong>

Length Penalty (LengthPenaltyReward) khuyến khích model sinh ra câu trả lời ngắn gọn, tránh dài dòng. Định nghĩa toán học là:

$$
r_{\text{length}}(a, a^*, l) = r_{\text{acc}}(a, a^*) - \alpha \cdot \max(0, l - l_{\text{target}})
$$

Trong đó $l$ là độ dài văn bản sinh ra (số ký tự hoặc số token), $l_{\text{target}}$ là độ dài mục tiêu, $\alpha$ là hệ số penalty (mặc định 0.001). Chỉ khi đáp án đúng mới áp dụng length penalty, tránh model sinh ra đáp án ngắn nhưng sai để giảm penalty.

Ý tưởng thiết kế là: nếu đáp án sai, reward bằng 0 (bất kể độ dài); nếu đáp án đúng và độ dài hợp lý, reward bằng 1; nếu đáp án đúng nhưng quá dài, reward bằng $1 - \alpha \cdot (l - l_{\text{target}})$. Ví dụ, độ dài mục tiêu 200 ký tự, độ dài thực tế 500 ký tự, hệ số penalty 0.001, thì reward bằng $1 - 0.001 \times (500 - 200) = 0.7$.

Ví dụ sử dụng:

```python
# 创建长度惩罚奖励函数
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,      # 最大长度
    "penalty_weight": 0.001  # 惩罚权重
})
reward_data = json.loads(reward_result)

print(f"奖励类型: {reward_data['reward_type']}")
print(f"描述: {reward_data['description']}")
print(f"最大长度: {reward_data['max_length']}")
print(f"惩罚权重: {reward_data['penalty_weight']}")
```

Output:

```
预测: 72, 真实: 72, 长度: 50, 奖励: 1.000
预测: 72, 真实: 72, 长度: 200, 奖励: 1.000
预测: 72, 真实: 72, 长度: 500, 奖励: 0.700
预测: 73, 真实: 72, 长度: 50, 奖励: 0.000
```

Ưu điểm của Length Penalty là khuyến khích diễn đạt ngắn gọn, tránh model sinh ra nội dung dư thừa, có thể kiểm soát chi phí inference (output ngắn hơn đồng nghĩa với tiêu thụ token ít hơn). Nhược điểm là có thể ức chế reasoning chi tiết, cần điều chỉnh cẩn thận hệ số penalty, độ dài tối ưu của các tác vụ khác nhau chênh lệch rất lớn.

<strong>（3）Step Reward</strong>

Step Reward (StepReward) khuyến khích model sinh ra các bước reasoning rõ ràng, nâng cao khả năng diễn giải. Định nghĩa toán học là:

$$
r_{\text{step}}(a, a^*, s) = r_{\text{acc}}(a, a^*) + \beta \cdot s
$$

Trong đó $s$ là số lượng bước reasoning được phát hiện, $\beta$ là hệ số step reward (mặc định 0.1). Tương tự, chỉ khi đáp án đúng mới cấp step reward.

Phương pháp phát hiện bước bao gồm: tìm các ký hiệu đánh dấu như "Step 1:", "Step 2:", đếm số ký tự xuống dòng, dùng regular expression để khớp mẫu reasoning. Ví dụ, một đáp án đúng chứa 3 bước rõ ràng sẽ có reward là $1 + 0.1 \times 3 = 1.3$.

Ví dụ sử dụng:

```python
# 创建步骤奖励函数
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1  # 每个步骤奖励0.1
})
reward_data = json.loads(reward_result)

print(f"奖励类型: {reward_data['reward_type']}")
print(f"描述: {reward_data['description']}")
print(f"步骤奖励: {reward_data['step_bonus']}")
```

Output:

```
预测: 72, 真实: 72, 步骤: 0, 奖励: 1.00
预测: 72, 真实: 72, 步骤: 2, 奖励: 1.20
预测: 72, 真实: 72, 步骤: 5, 奖励: 1.50
预测: 73, 真实: 72, 步骤: 5, 奖励: 0.00
```

Ưu điểm của Step Reward là khuyến khích reasoning có thể diễn giải được, đáp án sinh ra dễ kiểm chứng và debug hơn, giúp model học cách tư duy có hệ thống. Nhược điểm là có thể khiến model sinh ra các bước dư thừa để nhận thêm reward, cần cân bằng giữa số lượng bước và chất lượng đáp án, việc phát hiện bước có thể không chính xác.

Trong ứng dụng thực tế, chúng ta thường kết hợp nhiều reward function, để cân bằng các mục tiêu khác nhau. Các chiến lược kết hợp phổ biến bao gồm:

<strong>Accuracy + Length Penalty</strong>: khuyến khích đáp án ngắn gọn và đúng, phù hợp với hệ thống hội thoại, hệ thống hỏi đáp. Công thức là:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}})
$$

<strong>Accuracy + Step Reward</strong>: khuyến khích quá trình reasoning chi tiết, phù hợp với kịch bản giáo dục, AI có thể diễn giải. Công thức là:

$$
r = r_{\text{acc}} + \beta \cdot s
$$

<strong>Cân bằng ba yếu tố</strong>: tối ưu hóa toàn diện chất lượng đáp án, tính ngắn gọn và khả năng diễn giải. Công thức là:
$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}}) + \beta \cdot s
$$

Cần điều chỉnh cẩn thận trọng số $\alpha$ và $\beta$, tránh để một mục tiêu nào đó chiếm ưu thế quá mức.

Ví dụ sử dụng:

```python
# 组合奖励函数:准确率 + 长度惩罚 + 步骤奖励
# 注意: RLTrainingTool目前支持单一奖励类型
# 组合奖励需要在训练配置中通过reward_fn参数指定
# 这里展示如何配置不同类型的奖励函数

# 准确率奖励
accuracy_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
print("准确率奖励:", json.loads(accuracy_result)['description'])

# 长度惩罚奖励
length_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,
    "penalty_weight": 0.001
})
print("长度惩罚奖励:", json.loads(length_result)['description'])

# 步骤奖励
step_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1
})
print("步骤奖励:", json.loads(step_result)['description'])
```

Output:

```
组合奖励: 1.200
  - 准确率: 1.0
  - 长度惩罚: -0.100
  - 步骤奖励: +0.3
```

Như Bảng 11.4 thể hiện, các reward function khác nhau phù hợp với các kịch bản ứng dụng khác nhau.

<div align="center">
  <p>Bảng 11.4 So sánh các Reward Function</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-4.png" alt="" width="85%"/>
</div>

### 11.2.3 Tùy chỉnh Dataset và Reward Function

Mặc dù HelloAgents cung cấp sẵn dataset GSM8K và các reward function thông dụng, nhưng trong ứng dụng thực tế, bạn có thể cần sử dụng dataset của riêng mình hoặc thiết kế reward function đặc thù. Mục này sẽ giới thiệu cách mở rộng framework.

Trước khi sử dụng dataset tùy chỉnh, cần hiểu rõ yêu cầu dữ liệu của hai định dạng huấn luyện:

<strong>Định dạng SFT</strong>: dùng cho supervised fine-tuning, cần chứa các field sau:
- `prompt`: prompt đầu vào (chứa message system và user)
- `completion`: output mong muốn
- `text`: văn bản hội thoại hoàn chỉnh (tùy chọn)

<strong>Định dạng RL</strong>: dùng cho Reinforcement Learning, cần chứa các field sau:
- `question`: câu hỏi gốc
- `prompt`: prompt đầu vào (chứa message system và user)
- `ground_truth`: đáp án đúng
- `full_answer`: đáp án hoàn chỉnh (chứa quá trình reasoning)

<strong>（1）Dùng format_math_dataset để chuyển đổi</strong>

Phương pháp đơn giản nhất là chuẩn bị dữ liệu gốc chứa field `question` và `answer`, sau đó dùng hàm `format_math_dataset()` để tự động chuyển đổi:

```python
from datasets import Dataset
from hello_agents.rl import format_math_dataset

# 1. 准备原始数据
custom_data = [
    {
        "question": "What is 2+2?",
        "answer": "2+2=4. #### 4"
    },
    {
        "question": "What is 5*3?",
        "answer": "5*3=15. #### 15"
    },
    {
        "question": "What is 10+7?",
        "answer": "10+7=17. #### 17"
    }
]

# 2. 转换为Dataset对象
raw_dataset = Dataset.from_list(custom_data)

# 3. 转换为SFT格式
sft_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="sft",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"SFT数据集: {len(sft_dataset)}个样本")
print(f"字段: {sft_dataset.column_names}")

# 4. 转换为RL格式
rl_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="rl",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"RL数据集: {len(rl_dataset)}个样本")
print(f"字段: {rl_dataset.column_names}")
```

<strong>（2）Truyền trực tiếp dataset tùy chỉnh</strong>

Khi dùng RLTrainingTool, có thể truyền trực tiếp dataset tùy chỉnh qua tham số `custom_dataset`:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# SFT训练
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_sft",
    "num_epochs": 3,
    "batch_size": 4,
    "use_lora": True,
    "custom_dataset": sft_dataset  # 直接传入自定义数据集
})

# GRPO训练
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    "custom_dataset": rl_dataset  # 直接传入自定义数据集
})
```

（3）Đăng ký dataset tùy chỉnh (khuyến nghị)

Đối với dataset cần sử dụng nhiều lần, khuyến nghị dùng cách đăng ký:

```python
# 1. 注册数据集
rl_tool.register_dataset("my_math_dataset", rl_dataset)

# 2. 使用注册的数据集
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",  # 使用注册的数据集名称
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "use_lora": True
})
```

Reward function dùng để đánh giá chất lượng đáp án do model sinh ra. Reward function tùy chỉnh cần tuân theo chữ ký (signature) sau:

```python
from typing import List
import re

def custom_reward_function(
    completions: List[str],
    **kwargs
) -> List[float]:
    """
    自定义奖励函数

    Args:
        completions: 模型生成的完成文本列表
        **kwargs: 其他参数,通常包含:
            - ground_truth: 正确答案列表
            - 其他数据集字段

    Returns:
        奖励值列表(每个值在0.0-1.0之间)
    """
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        reward = 0.0

        # 提取答案
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                # 根据误差给予不同奖励
                if error < 0.01:
                    reward = 1.0  # 完全正确
                elif error < 1.0:
                    reward = 0.8  # 非常接近
                elif error < 5.0:
                    reward = 0.5  # 接近

                # 额外奖励:鼓励展示推理步骤
                if "step" in completion.lower() or "=" in completion:
                    reward += 0.1

            except ValueError:
                reward = 0.0

        rewards.append(min(reward, 1.0))  # 限制最大值为1.0

    return rewards
```

Có hai cách để sử dụng reward function tùy chỉnh:

<strong>（1）Truyền trực tiếp</strong>

```python
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "custom_dataset": rl_dataset,
    "custom_reward": custom_reward_function  # 直接传入奖励函数
})
```

<strong>（2）Đăng ký sử dụng (khuyến nghị)</strong>

```python
# 1. 注册奖励函数
rl_tool.register_reward_function("my_reward", custom_reward_function)

# 2. 使用注册的奖励函数
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",
    "output_dir": "./models/custom_grpo"
    # 奖励函数会自动使用与dataset同名的注册函数
})
```

Dưới đây là một ví dụ hoàn chỉnh về dataset và reward function tùy chỉnh:

```python
from datasets import Dataset
from hello_agents.tools import RLTrainingTool
from hello_agents.rl import format_math_dataset
import re
from typing import List

# 1. 准备自定义数据
custom_data = [
    {"question": "What is 2+2?", "answer": "2+2=4. #### 4"},
    {"question": "What is 5+3?", "answer": "5+3=8. #### 8"},
    {"question": "What is 10+7?", "answer": "10+7=17. #### 17"}
]

# 2. 转换为训练格式
raw_dataset = Dataset.from_list(custom_data)
rl_dataset = format_math_dataset(raw_dataset, format_type="rl")

# 3. 定义自定义奖励函数
def tolerant_reward(completions: List[str], **kwargs) -> List[float]:
    """带容差的奖励函数"""
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                if error < 0.01:
                    reward = 1.0
                elif error < 5.0:
                    reward = 0.5
                else:
                    reward = 0.0
            except ValueError:
                reward = 0.0
        else:
            reward = 0.0

        rewards.append(reward)

    return rewards

# 4. 创建工具并注册
rl_tool = RLTrainingTool()
rl_tool.register_dataset("my_dataset", rl_dataset)
rl_tool.register_reward_function("my_dataset", tolerant_reward)

# 5. 训练
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "dataset": "my_dataset",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True
})
```

## 11.3 Huấn luyện SFT

Supervised Fine-Tuning (SFT) là bước đầu tiên của huấn luyện Reinforcement Learning, cũng là nền tảng quan trọng nhất. SFT giúp model học định dạng cơ bản của tác vụ, kiểu hội thoại và khả năng reasoning sơ bộ. Nếu không có nền tảng SFT, việc trực tiếp thực hiện Reinforcement Learning thường sẽ thất bại, vì model thậm chí còn không biết định dạng output cơ bản.

### 11.3.1 Tại sao cần SFT

Trước khi bắt đầu Reinforcement Learning, chúng ta cần thực hiện huấn luyện SFT trước. Điều này là vì model đã pretrain tuy có khả năng ngôn ngữ mạnh mẽ, nhưng nó không biết cách hoàn thành một tác vụ cụ thể. Mục tiêu huấn luyện của model pretrain là dự đoán từ tiếp theo, chứ không phải giải bài toán hay sử dụng tool. Định dạng output của model pretrain là văn bản tự do, trong khi chúng ta cần output có cấu trúc (như "Step 1: ..., Step 2: ..., Final Answer: ..."). Model pretrain chưa từng thấy dữ liệu liên quan đến tác vụ, không biết thế nào là quá trình reasoning "tốt".

Vai trò của SFT là dạy cho model các quy tắc cơ bản của tác vụ. Trước tiên, học định dạng output, giúp model biết cách tổ chức đáp án (như dùng ký hiệu "Step 1", "Final Answer"). Thứ hai, học mẫu reasoning, thông qua ví dụ học cách phân rã bài toán, suy luận từng bước. Thứ ba, xây dựng năng lực cơ sở (baseline), cung cấp một điểm xuất phát hợp lý cho Reinforcement Learning tiếp theo. Cuối cùng, giảm không gian khám phá (exploration space), Reinforcement Learning không cần bắt đầu từ con số không, có thể tối ưu hóa dựa trên nền tảng SFT.

Hãy cùng hiểu tầm quan trọng của SFT qua một thí nghiệm so sánh. Giả sử chúng ta dùng trực tiếp model pretrain để giải bài toán GSM8K:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# 加载预训练模型
model_name = "Qwen/Qwen3-0.6B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# 测试问题
question = """Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?"""

# 构造输入
prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
inputs = tokenizer(prompt, return_tensors="pt")

# 生成回答
outputs = model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("预训练模型的回答:")
print(response)
```

Output của model pretrain có thể là:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
I can help you with that. Let me think about this problem. Natalia sold clips to her friends. In April she sold 48 clips. In May she sold half as many. So we need to find out how many she sold in May and then add them together. But I'm not sure how to calculate half of 48. Maybe I should use a calculator? Or maybe I can just estimate...
```

Có thể thấy, model pretrain tuy hiểu được câu hỏi, nhưng output rất dài dòng, thiếu cấu trúc, không có đáp án rõ ràng, quá trình reasoning hỗn loạn. Output như vậy không thể dùng cho Reinforcement Learning, vì chúng ta không thể trích xuất đáp án, không thể đánh giá chất lượng, không thể cung cấp tín hiệu reward hiệu quả.

Bây giờ hãy xem model sau SFT:

```python
# 加载SFT模型(假设已经训练好)
sft_model_path = "./models/sft_model"
sft_model = AutoModelForCausalLM.from_pretrained(sft_model_path)

# 使用相同的问题
outputs = sft_model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("SFT模型的回答:")
print(response)
```

Output của model SFT:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
Let me solve this step by step.

Step 1: Calculate clips sold in May
Natalia sold half as many clips in May as in April.
Clips in May = 48 / 2 = 24

Step 2: Calculate total clips
Total = April + May = 48 + 24 = 72

Final Answer: 72<|im_end|>
```

Có thể thấy, output của model SFT có cấu trúc rõ ràng (dùng ký hiệu "Step 1", "Step 2", "Final Answer"), reasoning chính xác, đáp án rõ ràng, định dạng thống nhất. Output như vậy có thể dùng cho Reinforcement Learning, vì chúng ta có thể trích xuất đáp án, tính reward, tối ưu hóa policy.

Như Hình 11.6 thể hiện, SFT là cầu nối từ model pretrain đến Reinforcement Learning.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-6.png" alt="" width="85%"/>
  <p>Hình 11.6 Vai trò của SFT trong quy trình huấn luyện</p>
</div>

### 11.3.2 LoRA: Fine-tuning hiệu quả về tham số

Fine-tune trực tiếp toàn bộ model cần lượng lớn tài nguyên tính toán và VRAM. Đối với Qwen3-0.6B (0.6B tham số), full fine-tuning cần khoảng 12GB VRAM (FP16) hoặc 24GB VRAM (FP32). Đối với các model lớn hơn (như 7B, 13B), full fine-tuning hầu như không thể thực hiện trên GPU tiêu dùng (consumer GPU).

LoRA (Low-Rank Adaptation)<sup>[3]</sup> là một phương pháp fine-tuning hiệu quả về tham số (parameter-efficient fine-tuning), nó chỉ huấn luyện một lượng nhỏ tham số bổ sung, trong khi vẫn giữ nguyên (đóng băng/freeze) tham số của model gốc. Ý tưởng cốt lõi của LoRA là: sự thay đổi tham số khi fine-tune model có thể được biểu diễn bằng ma trận hạng thấp (low-rank matrix).

Giả sử ma trận trọng số của model gốc là $W \in \mathbb{R}^{d \times k}$, trọng số sau fine-tune là $W' = W + \Delta W$. LoRA giả định rằng $\Delta W$ có thể được phân rã thành tích của hai ma trận hạng thấp:

$$
\Delta W = BA
$$

Trong đó $B \in \mathbb{R}^{d \times r}$, $A \in \mathbb{R}^{r \times k}$, $r \ll \min(d, k)$ là rank (hạng).

Khi forward propagation, output là:

$$
h = Wx + \Delta Wx = Wx + BAx
$$

Tham số model gốc $W$ được giữ nguyên (freeze), chỉ huấn luyện $B$ và $A$.

So sánh số lượng tham số: số lượng tham số của model gốc là $d \times k$, số lượng tham số của LoRA là $d \times r + r \times k = r(d + k)$. Khi $r \ll \min(d, k)$, số lượng tham số của LoRA nhỏ hơn rất nhiều so với model gốc. Ví dụ, với trường hợp $d=4096, k=4096, r=8$, số lượng tham số của model gốc là $4096 \times 4096 = 16.777.216$, số lượng tham số của LoRA là $8 \times (4096 + 4096) = 65.536$, số lượng tham số giảm 256 lần!

Do đó có thể tổng kết ưu điểm của LoRA: giảm đáng kể mức tiêu thụ VRAM, tốc độ huấn luyện nhanh hơn, dễ triển khai (deploy), ngăn ngừa overfitting. Tuy nhiên hiệu quả huấn luyện thường kém hơn một chút so với full fine-tuning.

Như Bảng 11.5 thể hiện, hiệu quả so sánh của LoRA ở các quy mô model khác nhau.

<div align="center">
  <p>Bảng 11.5 So sánh LoRA và full fine-tuning</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-5.png" alt="" width="85%"/>
</div>

Các hyperparameter then chốt của LoRA bao gồm: rank (r), kiểm soát hạng của ma trận LoRA, càng lớn thì khả năng biểu diễn càng mạnh, nhưng số lượng tham số cũng càng nhiều, giá trị điển hình là 4-64, mặc định 8; Alpha ($\alpha$), hệ số scale của LoRA, cập nhật thực tế là $\Delta W = \frac{\alpha}{r} BA$, kiểm soát mức độ ảnh hưởng của LoRA, giá trị điển hình bằng rank; target_modules (module mục tiêu), chỉ định những layer nào áp dụng LoRA, thường chọn attention layer (q_proj, k_proj, v_proj, o_proj), cũng có thể bao gồm MLP layer (gate_proj, up_proj, down_proj).

### 11.3.3 Thực hành huấn luyện SFT

Bây giờ hãy dùng HelloAgents để thực hiện huấn luyện SFT. Quy trình huấn luyện hoàn chỉnh bao gồm: chuẩn bị dataset, cấu hình LoRA, thiết lập tham số huấn luyện, bắt đầu huấn luyện, lưu model.

Ví dụ huấn luyện cơ bản:

```python
from hello_agents.tools import RLTrainingTool

# 创建训练工具
rl_tool = RLTrainingTool()

# SFT训练
result = rl_tool.run({
    # 训练配置
    "action": "train",
    "algorithm": "sft",
    
    # 模型配置
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_model",
    
    # 数据配置
    "max_samples": 100,     # 使用100个样本快速测试
    
    # 训练参数
    "num_epochs": 3,        # 训练3轮
    "batch_size": 4,        # 批次大小
    "learning_rate": 5e-5,  # 学习率
    
    # LoRA配置
    "use_lora": True,       # 使用LoRA
    "lora_rank": 8,         # LoRA秩
    "lora_alpha": 16,       # LoRA alpha
})

print(f"\n✓ 训练完成!")
print(f"  - 模型保存路径: {result['model_path']}")
print(f"  - 训练样本数: {result['num_samples']}")
print(f"  - 训练轮数: {result['num_epochs']}")
print(f"  - 最终损失: {result['final_loss']:.4f}")
```

Nếu loss giảm dần trong quá trình huấn luyện, nghĩa là model đang học.

<strong>（1）Giải thích chi tiết tham số huấn luyện</strong>

Hãy tìm hiểu chi tiết ý nghĩa và gợi ý điều chỉnh của từng tham số huấn luyện.

<strong>Tham số dữ liệu</strong>:

- `max_samples`: Số lượng mẫu huấn luyện được sử dụng. Khi kiểm thử nhanh có thể dùng 100-1000 mẫu, huấn luyện đầy đủ khuyến nghị dùng toàn bộ dữ liệu (7473 mẫu). Dữ liệu nhiều hơn thường mang lại hiệu quả tốt hơn, nhưng thời gian huấn luyện cũng lâu hơn.
- `split`: Cách chia dataset, mặc định "train". Có thể đặt thành "train[:1000]" để chỉ dùng 1000 mẫu đầu tiên.

<strong>Tham số huấn luyện</strong>:

- `num_epochs`: Số epoch huấn luyện. 1 epoch nghĩa là duyệt qua toàn bộ dataset một lần. Quá ít (1-2 epoch) có thể underfitting, quá nhiều (>10 epoch) có thể overfitting. Khuyến nghị bắt đầu từ 3 epoch, quan sát đường cong loss để điều chỉnh.
- `batch_size`: Số lượng mẫu dùng cho mỗi lần cập nhật. Càng lớn thì huấn luyện càng ổn định, nhưng mức tiêu thụ VRAM càng cao. Khuyến nghị điều chỉnh theo VRAM: VRAM 4GB dùng batch_size=1-2, VRAM 8GB dùng batch_size=4-8, VRAM 16GB dùng batch_size=8-16.
- `learning_rate`: Learning rate, kiểm soát bước cập nhật tham số. Quá nhỏ (1e-6) hội tụ chậm, quá lớn (1e-3) có thể không hội tụ. SFT khuyến nghị 5e-5, LoRA có thể lớn hơn một chút (1e-4).

<strong>Tham số LoRA</strong>:

- `use_lora`: Có dùng LoRA hay không. Khuyến nghị luôn bật, trừ khi có đủ VRAM.
- `lora_rank`: Rank của LoRA, kiểm soát khả năng biểu diễn. 4-8 phù hợp cho tác vụ nhỏ, 16-32 phù hợp cho tác vụ phức tạp, 64 phù hợp cho fine-tuning quy mô lớn.
- `lora_alpha`: Hệ số scale của LoRA, thường đặt bằng 2 lần rank. rank=8 thì alpha=16; rank=16 thì alpha=32.

<strong>Tham số optimizer</strong>:

- `optimizer`: Loại optimizer, mặc định "adamw". AdamW là lựa chọn phổ biến nhất, cũng có thể thử "sgd" hoặc "adafactor" v.v.
- `weight_decay`: Weight decay, ngăn ngừa overfitting. Mặc định 0.01, có thể thử 0.001-0.1.
- `warmup_ratio`: Tỷ lệ warmup của learning rate. Trong số bước warmup_ratio đầu tiên, learning rate tăng tuyến tính, sau đó giảm dần tuyến tính. Mặc định 0.1 (warmup trong 10% số bước đầu).

<strong>（2）Ví dụ huấn luyện hoàn chỉnh</strong>

Hãy thực hiện một lần huấn luyện SFT hoàn chỉnh, sử dụng toàn bộ dữ liệu và best practice:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 完整SFT训练
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",

    # 模型配置
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_full",

    # 数据配置
    "max_samples": None,    # 使用全部数据(7473个样本)

    # 训练参数
    "num_epochs": 3,
    "batch_size": 8,
    "learning_rate": 5e-5,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01,

    # LoRA配置
    "use_lora": True,
    "lora_rank": 16,        # 使用更大的rank
    "lora_alpha": 32,
    "lora_target_modules": ["q_proj", "k_proj", "v_proj", "o_proj"],

    # 其他配置
    "save_steps": 500,      # 每500步保存一次
    "logging_steps": 100,   # 每100步记录一次
    "eval_steps": 500,      # 每500步评估一次
})

print(f"训练完成! 模型保存在: {result['model_path']}")
```

Cấu hình này phù hợp để huấn luyện trên GPU có 8GB VRAM, thời gian dự kiến 30-60 phút.

<strong>（3）Giám sát và debug huấn luyện</strong>

Trong quá trình huấn luyện, chúng ta cần giám sát ba chỉ số then chốt. Loss nên giảm dần, nếu không giảm có thể là learning rate quá nhỏ hoặc dữ liệu có vấn đề, nếu giảm rồi lại tăng thì có thể là learning rate quá lớn hoặc xuất hiện overfitting. Gradient Norm nên nằm trong khoảng hợp lý 0.1-10, quá lớn (>100) cho thấy xuất hiện gradient exploding cần giảm learning rate, quá nhỏ (<0.01) cho thấy gradient vanishing cần kiểm tra cấu hình model. Learning Rate nên biến đổi theo chiến lược warmup, 10% số bước đầu tăng tuyến tính, sau đó giảm dần tuyến tính về 0.

Các vấn đề thường gặp trong huấn luyện và giải pháp: khi thiếu VRAM có thể giảm batch_size hoặc max_length, dùng gradient accumulation hoặc model nhỏ hơn; khi tốc độ huấn luyện chậm có thể tăng batch_size, giảm tần suất logging, hoặc dùng mixed precision training; khi loss không giảm có thể tăng learning rate, kiểm tra định dạng dữ liệu, hoặc tăng số epoch huấn luyện; khi overfitting có thể tăng weight_decay, giảm số epoch huấn luyện, hoặc dùng nhiều dữ liệu hơn.

### 11.3.4 Đánh giá model

Sau khi huấn luyện xong, chúng ta cần đánh giá hiệu quả của model. Các chỉ số đánh giá bao gồm:

- <strong>Accuracy</strong>: Tỷ lệ đáp án hoàn toàn đúng, chỉ số trực tiếp nhất, phạm vi 0-1, càng cao càng tốt.

- <strong>Average Reward</strong>: Reward trung bình của tất cả các mẫu, xem xét tổng hợp các yếu tố như accuracy, độ dài, số bước, phạm vi phụ thuộc vào thiết kế reward function.

- <strong>Reasoning Quality</strong>: Độ rõ ràng và tính logic của quá trình reasoning, cần đánh giá thủ công hoặc dùng model đánh giá chuyên dụng.

Dùng HelloAgents để đánh giá model:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 评估SFT模型
eval_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,     # 在100个测试样本上评估
    "use_lora": True,
})

eval_data = json.loads(eval_result)
print(f"\n评估结果:")
print(f"  - 准确率: {eval_data['accuracy']}")
print(f"  - 平均奖励: {eval_data['average_reward']}")
print(f"  - 测试样本数: {eval_data['num_samples']}")
```

Đối với một small model như Qwen3-0.6B, việc đạt độ chính xác 40-50% trên GSM8K sau SFT là bình thường. Thông qua Reinforcement Learning, chúng ta có thể nâng cao hơn nữa lên 60-70%.

Để hiểu rõ hơn hiệu quả của SFT, chúng ta có thể so sánh model ở các giai đoạn khác nhau:

```python
# 评估预训练模型(未经SFT)
base_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "Qwen/Qwen3-0.6B",
    "max_samples": 100,
    "use_lora": False,
})
base_data = json.loads(base_result)

# 评估SFT模型
sft_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,
    "use_lora": True,
})
sft_data = json.loads(sft_result)

# 对比结果
print("模型对比:")
print(f"预训练模型准确率: {base_data['accuracy']}")
print(f"SFT模型准确率: {sft_data['accuracy']}"
```

Trong mục này, chúng ta đã học về tầm quan trọng của SFT (học định dạng, xây dựng baseline), nguyên lý LoRA (phân rã hạng thấp, hiệu quả về tham số), thực hành huấn luyện SFT (cấu hình tham số, giám sát huấn luyện), đánh giá model (accuracy, phân tích so sánh).

## 11.4 Huấn luyện GRPO

Sau khi hoàn thành huấn luyện SFT, chúng ta đã có được một model có khả năng sinh ra đáp án có cấu trúc. Nhưng model SFT mới chỉ học được cách "bắt chước" quá trình reasoning trong dữ liệu huấn luyện, chưa thực sự học được cách "suy nghĩ". Reinforcement Learning có thể giúp model thông qua thử-sai để tối ưu hóa chiến lược reasoning, từ đó vượt qua chất lượng của dữ liệu huấn luyện.

### 11.4.1 Từ PPO đến GRPO

Trong lĩnh vực Reinforcement Learning, PPO (Proximal Policy Optimization)<sup>[1]</sup> là một trong những thuật toán kinh điển nhất. PPO giới hạn mức độ cập nhật policy, đảm bảo tính ổn định của huấn luyện. Nhưng PPO tồn tại một số vấn đề khi huấn luyện LLM: cần huấn luyện Value Model, làm tăng độ phức tạp huấn luyện và mức tiêu thụ VRAM; cần duy trì đồng thời bốn model (Policy Model, Reference Model, Value Model, Reward Model), việc triển khai kỹ thuật phức tạp; huấn luyện không ổn định, dễ xuất hiện reward collapse hoặc policy degradation.

GRPO (Group Relative Policy Optimization)<sup>[2]</sup> là một biến thể đơn giản hóa của PPO, được thiết kế chuyên biệt cho LLM. Ý tưởng cốt lõi của GRPO là: không cần Value Model, dùng reward tương đối trong nhóm (group) thay cho reward tuyệt đối; đơn giản hóa quy trình huấn luyện, chỉ cần Policy Model và Reference Model; nâng cao tính ổn định của huấn luyện, giảm rủi ro reward collapse.

Hãy dùng công thức toán học để hiểu nguyên lý của GRPO. Hàm mục tiêu của PPO là:

$$
J_{\text{PPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \min\left( \frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)} A(s,a), \text{clip}\left(\frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)}, 1-\epsilon, 1+\epsilon\right) A(s,a) \right) \right]
$$

Trong đó $A(s,a)$ là advantage function, cần Value Model để ước lượng:

$$
A(s,a) = Q(s,a) - V(s) = r(s,a) + \gamma V(s') - V(s)
$$

Hàm mục tiêu của GRPO được đơn giản hóa thành:

$$
J_{\text{GRPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \cdot (r(s,a) - \bar{r}_{\text{group}}) \right] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

Trong đó $\bar{r}_{\text{group}}$ là reward trung bình trong nhóm, $\beta$ là hệ số penalty của KL divergence. Khác biệt then chốt ở chỗ: GRPO dùng $r(s,a) - \bar{r}_{\text{group}}$ thay cho advantage function $A(s,a)$, không cần Value Model; GRPO dùng reward tương đối trong nhóm, giảm phương sai của reward; GRPO thêm KL divergence penalty, ngăn policy lệch quá xa.

Như Hình 11.7 thể hiện, so sánh quy trình huấn luyện của PPO và GRPO.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-7.png" alt="" width="85%"/>
  <p>Hình 11.7 Quy trình huấn luyện PPO vs GRPO</p>
</div>

Có thể thấy, GRPO đã bỏ qua việc huấn luyện Value Model, đơn giản hóa đáng kể quy trình.

Như Bảng 11.6 thể hiện, so sánh chi tiết giữa PPO và GRPO.

<div align="center">
  <p>Bảng 11.6 So sánh PPO vs GRPO</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-6.png" alt="" width="85%"/>
</div>



Đối với huấn luyện LLM, GRPO là lựa chọn tốt hơn, vì nó đơn giản hơn, ổn định hơn, mức tiêu thụ VRAM thấp hơn.

### 11.4.2 Thực hành huấn luyện GRPO

Bây giờ hãy dùng HelloAgents để thực hiện huấn luyện GRPO. Tiền đề của huấn luyện GRPO là đã hoàn thành huấn luyện SFT, vì GRPO cần một policy khởi tạo hợp lý.

Ví dụ huấn luyện GRPO cơ bản:

```python
from hello_agents.tools import RLTrainingTool

# 创建训练工具
rl_tool = RLTrainingTool()

# GRPO训练
result = rl_tool.run({
    # 训练配置
    "action": "train",
    "algorithm": "grpo",
    
    # 模型配置
    "model_name": "./models/sft_full",  # 从SFT模型开始
    "output_dir": "./models/grpo_model",
    
    # 数据配置
    "max_samples": 100,     # 使用100个样本快速测试
    
    # 训练参数
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,  # GRPO学习率通常比SFT小
    
    # GRPO特定参数
    "num_generations": 4,   # 每个问题生成4个答案
    "kl_coef": 0.05,        # KL散度惩罚系数
    
    # LoRA配置
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,
    
    # 奖励函数配置
    "reward_type": "accuracy",  # 使用准确率奖励
})

print(f"\n✓ 训练完成!")
print(f"  - 模型保存路径: {result['model_path']}")
print(f"  - 训练样本数: {result['num_samples']}")
print(f"  - 训练轮数: {result['num_epochs']}")
print(f"  - 平均奖励: {result['average_reward']:.4f}")
```

Nếu average reward tăng dần trong quá trình huấn luyện GRPO, KL divergence giữ trong phạm vi hợp lý, nghĩa là huấn luyện đang diễn ra bình thường.

GRPO có một số tham số đặc thù cần hiểu và điều chỉnh.

<strong>Tham số sinh (generation)</strong>:

- `num_generations`: Mỗi câu hỏi sinh ra bao nhiêu đáp án. Càng nhiều càng tốt, nhưng chi phí tính toán cũng càng cao. Giá trị điển hình là 4-8. Mục đích của việc sinh nhiều đáp án là để tính reward tương đối trong nhóm, tăng tính đa dạng của tín hiệu huấn luyện.
- `max_new_tokens`: Số token tối đa được sinh cho mỗi đáp án. Quá ít có thể cắt cụt đáp án, quá nhiều lãng phí tính toán. Khuyến nghị 256-512.
- `temperature`: Temperature khi sinh, kiểm soát tính ngẫu nhiên. 0 nghĩa là greedy decoding, 1 nghĩa là standard sampling. GRPO khuyến nghị 0.7-1.0, giữ một mức độ exploration nhất định.

<strong>Tham số tối ưu hóa</strong>:

- `learning_rate`: Learning rate của GRPO thường nhỏ hơn SFT, vì chúng ta không muốn lệch quá xa so với model SFT. Khuyến nghị 1e-6 đến 1e-5; trên small model, learning rate quá lớn (như 5e-5) có thể dẫn đến policy collapse.
- `kl_coef`: Hệ số penalty của KL divergence, kiểm soát mức độ cập nhật policy. Quá nhỏ (0.01) có thể khiến policy lệch quá xa, quá lớn (0.5) có thể hạn chế việc học. Khuyến nghị 0.05-0.1.
- `clip_range`: Phạm vi clip tỷ lệ policy, tương tự epsilon của PPO. Khuyến nghị 0.2.

<strong>Tham số reward</strong>:

- `reward_type`: Loại reward function, có thể là "accuracy", "length_penalty", "step" hoặc "combined".
- `reward_config`: Cấu hình bổ sung của reward function, như độ dài mục tiêu của length penalty, hệ số của step reward, v.v.

Hãy thực hiện một lần huấn luyện GRPO hoàn chỉnh, sử dụng toàn bộ dữ liệu và best practice:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 完整GRPO训练
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",

    # 模型配置
    "model_name": "./models/sft_full",
    "output_dir": "./models/grpo_full",
    
    # 数据配置
    "max_samples": None,    # 使用全部数据
    
    # 训练参数
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,
    "warmup_ratio": 0.1,
    
    # GRPO特定参数
    "num_generations": 4,
    "max_new_tokens": 512,
    "temperature": 0.8,
    "kl_coef": 0.05,
    "clip_range": 0.2,
    
    # LoRA配置
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,
    
    # 奖励函数配置
    "reward_type": "combined",
    "reward_config": {
        "components": [
            {"type": "accuracy", "weight": 1.0},
            {"type": "length_penalty", "weight": 0.5, "target_length": 200},
            {"type": "step", "weight": 0.3, "step_bonus": 0.1}
        ]
    },
    
    # 其他配置
    "save_steps": 500,
    "logging_steps": 100,
})

print(f"训练完成! 模型保存在: {result['model_path']}")
```

### 11.4.3 Phân tích quá trình huấn luyện GRPO

Hãy cùng đi sâu hiểu quá trình huấn luyện GRPO, xem mỗi bước xảy ra chuyện gì.

<strong>（1）Vòng lặp huấn luyện</strong>

Vòng lặp huấn luyện của GRPO bao gồm các bước sau:

1. <strong>Giai đoạn Rollout (sampling)</strong>: Với mỗi câu hỏi, dùng policy hiện tại để sinh ra nhiều đáp án (`num_generations` đáp án). Các đáp án này tạo thành một "nhóm" (group), dùng để tính relative reward.

2. <strong>Tính Reward</strong>: Tính reward $r_i$ cho mỗi đáp án sinh ra. Reward có thể là accuracy, length penalty, step reward hoặc sự kết hợp của chúng.

3. <strong>Relative Reward</strong>: Tính reward trung bình trong nhóm $\bar{r} = \frac{1}{N}\sum_{i=1}^{N} r_i$, sau đó tính relative reward $\hat{r}_i = r_i - \bar{r}$. Lợi ích của cách làm này là giảm phương sai reward, giúp huấn luyện ổn định hơn.

4. <strong>Cập nhật Policy</strong>: Dùng relative reward để cập nhật policy, đồng thời thêm KL divergence penalty, ngăn policy lệch quá xa reference model.

5. <strong>Lặp lại</strong>: Lặp lại các bước trên, cho đến khi hoàn thành tất cả các epoch huấn luyện.

Hãy cùng hiểu qua một ví dụ cụ thể:

```python
# 假设我们有一个问题
question = "What is 48 + 24?"

# 生成4个答案
answers = [
    "48 + 24 = 72. Final Answer: 72",      # 正确
    "48 + 24 = 72. Final Answer: 72",      # 正确
    "48 + 24 = 70. Final Answer: 70",      # 错误
    "Let me think... 72. Final Answer: 72" # 正确但冗长
]

# 计算奖励(假设使用准确率 + 长度惩罚)
rewards = [1.0, 1.0, 0.0, 0.8]  # 第4个答案因为冗长被惩罚

# 计算组内平均奖励
avg_reward = (1.0 + 1.0 + 0.0 + 0.8) / 4 = 0.7

# 计算相对奖励
relative_rewards = [
    1.0 - 0.7 = 0.3,   # 正确且简洁,相对奖励为正
    1.0 - 0.7 = 0.3,   # 正确且简洁,相对奖励为正
    0.0 - 0.7 = -0.7,  # 错误,相对奖励为负
    0.8 - 0.7 = 0.1    # 正确但冗长,相对奖励较小
]

# 策略更新:增加前两个答案的概率,减少第三个答案的概率
```

Có thể thấy, cơ chế relative reward khuyến khích model sinh ra đáp án "tốt hơn mức trung bình", thay vì đơn thuần theo đuổi reward cao. Như vậy có thể giảm phương sai reward, nâng cao tính ổn định của huấn luyện.

<strong>（2）KL Divergence Penalty</strong>

KL divergence penalty là thành phần then chốt của GRPO, nó ngăn policy lệch quá xa reference model. KL divergence được định nghĩa là:

$$
D_{KL}(\pi_\theta || \pi_{\text{ref}}) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \log \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \right]
$$

Trong thực tế, chúng ta tính KL divergence của mỗi token, sau đó cộng lại:

$$
D_{KL} = \sum_{t=1}^{T} \log \frac{\pi_\theta(a_t|s, a_{<t})}{\pi_{\text{ref}}(a_t|s, a_{<t})}
$$

KL divergence càng lớn, nghĩa là policy hiện tại khác biệt càng nhiều so với reference model. Thông qua việc thêm penalty term $-\beta \cdot D_{KL}$, chúng ta giới hạn mức độ cập nhật policy, tránh "quên" kiến thức đã học ở giai đoạn SFT.

Việc lựa chọn `kl_coef` ($\beta$) rất quan trọng:

- Quá nhỏ (0.01): policy có thể lệch quá xa, dẫn đến định dạng output hỗn loạn hoặc chất lượng giảm sút
- Quá lớn (0.5): việc cập nhật policy bị hạn chế, học chậm, khó vượt qua model SFT
- Khuyến nghị (0.05-0.1): cân bằng giữa exploration và tính ổn định

<strong>（3）Giám sát huấn luyện</strong>

Trong quá trình huấn luyện GRPO, chúng ta cần giám sát các chỉ số sau:

- <strong>Average Reward</strong>: nên tăng dần. Nếu reward không tăng, có thể là learning rate quá nhỏ, KL penalty quá lớn, thiết kế reward function không hợp lý. Nếu reward tăng rồi lại giảm, có thể là overfitting hoặc reward collapse.

- <strong>KL Divergence</strong>: nên giữ trong phạm vi hợp lý (0.01-0.1). Nếu KL divergence quá lớn (>0.5), nghĩa là policy lệch quá xa, cần tăng kl_coef hoặc giảm learning rate. Nếu KL divergence quá nhỏ (<0.001), nghĩa là policy gần như không cập nhật, cần giảm kl_coef hoặc tăng learning rate.

- <strong>Accuracy</strong>: nên tăng dần. Đây là chỉ số trực quan nhất, phản ánh năng lực thực tế của model.

- <strong>Generation Quality</strong>: cần kiểm tra thủ công các đáp án được sinh ra, đảm bảo định dạng đúng, reasoning rõ ràng.

HelloAgents tích hợp hai công cụ giám sát huấn luyện phổ biến: Weights & Biases (wandb) và TensorBoard.

<strong>Cách 1: Dùng Weights & Biases (khuyến nghị)</strong>

Weights & Biases là nền tảng theo dõi thực nghiệm machine learning phổ biến nhất hiện nay, cung cấp khả năng visualize và quản lý thực nghiệm mạnh mẽ.

```python
import os

# 1. 设置wandb(需要先注册账号: https://wandb.ai)
os.environ["WANDB_PROJECT"] = "hello-agents-grpo"  # 项目名称
os.environ["WANDB_LOG_MODEL"] = "false"            # 不上传模型文件

# 2. 在训练配置中启用wandb
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_monitored",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    # wandb会自动记录所有训练指标
})

# 训练完成后,访问 https://wandb.ai 查看训练曲线
```

wandb sẽ tự động ghi lại các chỉ số sau:
- `train/reward`: average reward
- `train/kl`: KL divergence
- `train/loss`: training loss
- `train/learning_rate`: learning rate
- `train/epoch`: số epoch huấn luyện

<strong>Cách 2: Dùng TensorBoard</strong>

TensorBoard là công cụ visualize do TensorFlow cung cấp, cũng hỗ trợ huấn luyện PyTorch.

```python
# 1. 训练时会自动在output_dir下创建tensorboard日志
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_tb",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 2. 启动TensorBoard查看训练曲线
# 在命令行运行:
# tensorboard --logdir=./models/grpo_tb
# 然后访问 http://localhost:6006
```

<strong>Cách 3: Giám sát offline (không cần công cụ bên ngoài)</strong>

Nếu không muốn dùng wandb hoặc TensorBoard, cũng có thể giám sát thông qua log huấn luyện:

```python
# 训练过程会打印详细日志
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_simple",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 日志示例:
# Epoch 1/2 | Step 100/500 | Reward: 0.45 | KL: 0.023 | Loss: 1.234
# Epoch 1/2 | Step 200/500 | Reward: 0.52 | KL: 0.031 | Loss: 1.156
# ...
```

Trong quá trình huấn luyện GRPO, có thể gặp một số vấn đề. Khi reward không tăng, có thể là learning rate quá nhỏ hoặc KL penalty quá lớn hạn chế việc cập nhật policy, cũng có thể là thiết kế reward function không hợp lý hoặc chất lượng model SFT quá kém, lúc này có thể tăng learning rate (từ 1e-5 lên 5e-5), giảm kl_coef (từ 0.1 xuống 0.05), kiểm tra lại reward function hoặc huấn luyện lại model SFT.

Khi KL divergence bùng nổ (vượt quá 0.5 thậm chí 1.0) dẫn đến định dạng đáp án sinh ra hỗn loạn, thường là learning rate quá lớn hoặc KL penalty quá nhỏ, hoặc reward function quá tích cực (quá "aggressive"), có thể giảm learning rate (từ 5e-5 xuống 1e-5), tăng kl_coef (từ 0.05 lên 0.1), điều chỉnh reward function hoặc dùng gradient clipping.

Khi chất lượng sinh ra giảm sút (accuracy tăng nhưng định dạng hỗn loạn, reasoning không rõ ràng), có thể là reward function chỉ quan tâm accuracy mà bỏ qua các chỉ số chất lượng khác, hoặc KL penalty quá nhỏ khiến model lệch quá xa SFT, hoặc xuất hiện overfitting, lúc này nên dùng combined reward function để đồng thời tối ưu hóa nhiều chỉ số, tăng kl_coef để giữ tính nhất quán, giảm số epoch huấn luyện hoặc tăng dữ liệu huấn luyện.

Mức tiêu thụ VRAM của huấn luyện GRPO cao hơn SFT, vì cần sinh đồng thời nhiều đáp án và lưu trữ output của reference model, dễ xảy ra OOM. Có thể giảm num_generations (từ 8 xuống 4), batch_size (từ 4 xuống 2) hoặc max_new_tokens (từ 512 xuống 256), hoặc dùng gradient checkpointing và mixed precision training để giảm bớt.

## 11.5 Đánh giá và phân tích model

Sau khi huấn luyện xong, chúng ta cần đánh giá toàn diện hiệu suất của model, không chỉ nhìn vào mỗi chỉ số accuracy, mà còn cần phân tích sâu chất lượng reasoning, mẫu lỗi (error pattern), khả năng tổng quát hóa của model. Mục này sẽ giới thiệu cách đánh giá và phân tích model Agentic RL một cách có hệ thống.

### 11.5.1 Hệ thống chỉ số đánh giá

Một hệ thống đánh giá tốt nên đa chiều, đo lường năng lực của model từ nhiều góc độ khác nhau. Chúng ta chia chỉ số đánh giá thành ba loại: chỉ số độ chính xác, chỉ số hiệu suất, chỉ số chất lượng.

<strong>（1）Chỉ số độ chính xác</strong>

Chỉ số độ chính xác đo lường model có thể đưa ra đáp án đúng hay không.

<strong>Accuracy</strong>: chỉ số cơ bản nhất, tỷ lệ đáp án hoàn toàn đúng. Công thức tính:
$$
\text{Accuracy} = \frac{\text{正确答案数}}{\text{总问题数}}
$$

Ưu điểm là đơn giản trực quan, dễ hiểu và dễ so sánh. Nhược điểm là không thể phân biệt "gần đúng" và "hoàn toàn sai", với tác vụ phức tạp có thể quá thô.

<strong>Top-K Accuracy</strong>: sinh K đáp án, chỉ cần một đáp án đúng là tính đúng. Công thức tính:
$$
\text{Accuracy@K} = \frac{\text{至少有一个正确答案的问题数}}{\text{总问题数}}
$$

Chỉ số này phản ánh "tiềm năng" của model, tức là thông qua nhiều lần sampling có thể tìm ra đáp án đúng hay không.

<strong>Numerical Error (sai số số học)</strong>: đối với bài toán toán học, có thể tính sai số giữa giá trị dự đoán và giá trị thực. Công thức tính:

$$
\text{Error} = \frac{1}{N} \sum_{i=1}^{N} |y_i - \hat{y}_i|
$$

Chỉ số này có thể phân biệt "gần đúng" (như dự đoán 72.5, thực tế 72) và "hoàn toàn sai" (như dự đoán 100, thực tế 72).

<strong>（2）Chỉ số hiệu suất</strong>

Chỉ số hiệu suất đo lường chi phí sinh ra đáp án của model.

<strong>Average Length</strong>: số token trung bình của đáp án sinh ra. Công thức tính:

$$
\text{Avg Length} = \frac{1}{N} \sum_{i=1}^{N} |y_i|
$$

Đáp án ngắn hơn đồng nghĩa với chi phí inference thấp hơn và tốc độ phản hồi nhanh hơn.

<strong>Reasoning Steps</strong>: số lượng bước reasoning chứa trong đáp án. Công thức tính:

$$
\text{Avg Steps} = \frac{1}{N} \sum_{i=1}^{N} s_i
$$

Số bước phù hợp (2-5 bước) cho thấy model có thể phân rã bài toán một cách có hệ thống, quá nhiều bước có thể cho thấy reasoning dư thừa.

<strong>Inference Time</strong>: thời gian cần thiết để sinh ra một đáp án. Chỉ số này rất quan trọng trong triển khai thực tế, ảnh hưởng đến trải nghiệm người dùng.

<strong>（3）Chỉ số chất lượng</strong>

Chỉ số chất lượng đo lường khả năng đọc hiểu và diễn giải của đáp án.

<strong>Format Correctness</strong>: đáp án có tuân theo định dạng mong muốn hay không (như chứa ký hiệu "Step 1", "Final Answer" v.v.). Công thức tính:
$$
\text{Format Correctness} = \frac{\text{格式正确的答案数}}{\text{总答案数}}
$$

Định dạng đúng là yêu cầu cơ bản, đáp án có định dạng hỗn loạn dù kết quả đúng cũng khó sử dụng.

<strong>Reasoning Coherence</strong>: các bước reasoning có logic nhất quán hay không. Chỉ số này thường cần đánh giá thủ công hoặc dùng model đánh giá chuyên dụng.

<strong>Explainability</strong>: đáp án có dễ hiểu và dễ kiểm chứng hay không. Đáp án chứa các bước rõ ràng có khả năng diễn giải tốt hơn đáp án chỉ đưa ra kết quả trực tiếp.

Như Bảng 11.7 thể hiện, so sánh các chỉ số khác nhau.

<div align="center">
  <p>Bảng 11.7 So sánh các chỉ số đánh giá</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-7.png" alt="" width="85%"/>
</div>


### 11.5.2 Thực hành đánh giá

HelloAgents cung cấp chức năng đánh giá toàn diện, có thể tính nhiều chỉ số cùng một lúc.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 全面评估
print("=" * 50)
print("全面评估GRPO模型")
print("=" * 50)

result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    
    # 评估配置
    "metrics": [
        "accuracy",           # 准确率
        "accuracy_at_k",      # Top-K准确率
        "average_length",     # 平均长度
        "average_steps",      # 平均步骤数
        "format_correctness", # 格式正确率
    ],
    "k": 3,  # Top-3准确率
})

# 解析结果
eval_data = json.loads(result)

# 打印结果
print(f"\n评估结果:")
print(f"  准确率: {eval_data['accuracy']}")
print(f"  平均奖励: {eval_data['average_reward']}")
print(f"  测试样本数: {eval_data['num_samples']}")
```

Chúng ta có thể so sánh hiệu suất của model pretrain, model SFT, model GRPO:

```python
# 评估三个模型
models = [
    ("预训练模型", "Qwen/Qwen3-0.6B", False),
    ("SFT模型", "./models/sft_full", True),
    ("GRPO模型", "./models/grpo_full", True),
]

results = []
for name, path, use_lora in models:
    print(f"\n评估{name}...")
    result = rl_tool.run({
        "action": "evaluate",
        "model_path": path,
        "max_samples": 200,
        "use_lora": use_lora,
        "metrics": ["accuracy", "average_length", "format_correctness"],
    })
    results.append((name, result))

# 打印对比表格
print("\n" + "=" * 70)
print(f"{'模型':<15} {'准确率':<12} {'平均长度':<15} {'格式正确率':<12}")
print("=" * 70)
for name, result in results:
    print(f"{name:<15} {result['accuracy']:<12.2%} {result['average_length']:<15.1f} {result['format_correctness']:<12.2%}")
print("=" * 70)
```

### 11.5.3 Phân tích lỗi

Chỉ biết accuracy là chưa đủ, chúng ta cần phân tích sâu xem model dễ mắc lỗi ở loại bài toán nào, từ đó định hướng cải thiện tiếp theo. Lỗi của model có thể chia thành bốn loại: lỗi tính toán (các bước reasoning đúng nhưng tính toán sai, như "48/2=25", cho thấy khả năng tính toán số học chưa đủ), lỗi reasoning (logic reasoning sai dẫn đến hướng giải sai, như cộng trước chia sau thay vì chia trước cộng sau, cho thấy khả năng reasoning logic chưa đủ), lỗi hiểu (không hiểu đúng câu hỏi, như câu hỏi hỏi "tổng cộng" nhưng chỉ tính một phần, cho thấy khả năng hiểu ngôn ngữ chưa đủ), lỗi định dạng (đáp án đúng nhưng định dạng không đúng yêu cầu, như thiếu ký hiệu "Final Answer:", cho thấy việc học định dạng chưa đủ).

Ví dụ phân tích lỗi:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 评估并收集错误样本
result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    "return_details": True,  # 返回详细结果
})

# 分析错误样本
errors = result['errors']  # 错误样本列表
print(f"总错误数: {len(errors)}")

# 按错误类型分类
error_types = {
    "计算错误": 0,
    "推理错误": 0,
    "理解错误": 0,
    "格式错误": 0,
}

for error in errors:
    question = error['question']
    prediction = error['prediction']
    ground_truth = error['ground_truth']
    
    # 简单的错误分类逻辑(实际应用中可能需要更复杂的分析)
    if "Final Answer:" not in prediction:
        error_types["格式错误"] += 1
    elif "Step" in prediction:
        # 有推理步骤,可能是计算或推理错误
        # 这里需要更细致的分析
        error_types["计算错误"] += 1
    else:
        error_types["理解错误"] += 1

# 打印错误分布
print("\n错误类型分布:")
for error_type, count in error_types.items():
    percentage = count / len(errors) * 100
    print(f"  {error_type}: {count} ({percentage:.1f}%)")
```

Output ví dụ:

```bash
总错误数: 76

错误类型分布:
  计算错误: 32 (42.1%)
  推理错误: 18 (23.7%)
  理解错误: 22 (28.9%)
  格式错误: 4 (5.3%)
```

Có thể thấy, lỗi tính toán là loại lỗi chủ yếu (42.1%), cho thấy khả năng tính toán số học của model cần được tăng cường. Lỗi định dạng rất ít (5.3%), cho thấy hiệu quả huấn luyện SFT tốt. Chúng ta cũng có thể phân tích hiệu suất của model trên các bài toán có độ khó khác nhau:

```python
# 按推理步骤数分组
step_groups = {
    "简单(1-2步)": [],
    "中等(3-4步)": [],
    "困难(5+步)": [],
}

for sample in result['details']:
    steps = sample['ground_truth_steps']  # 真实答案的步骤数
    correct = sample['correct']
    
    if steps <= 2:
        step_groups["简单(1-2步)"].append(correct)
    elif steps <= 4:
        step_groups["中等(3-4步)"].append(correct)
    else:
        step_groups["困难(5+步)"].append(correct)

# 计算每组的准确率
print("\n不同难度的准确率:")
for group_name, results in step_groups.items():
    if len(results) > 0:
        accuracy = sum(results) / len(results)
        print(f"  {group_name}: {accuracy:.2%} ({len(results)}个样本)")
```

Output ví dụ:

```bash
不同难度的准确率:
  简单(1-2步): 78.50% (85个样本)
  中等(3-4步): 58.30% (96个样本)
  困难(5+步): 31.60% (19个样本)
```

Có thể thấy, model thể hiện tốt trên bài toán đơn giản (78.5%), nhưng kém hơn trên bài toán khó (31.6%). Điều này cho thấy khả năng reasoning nhiều bước của model vẫn cần được nâng cao

### 11.5.4 Hướng cải thiện

Dựa trên kết quả đánh giá và phân tích, chúng ta có thể xác định hướng cải thiện model, như Hình 11.8 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-8.png" alt="" width="85%"/>
  <p>Hình 11.8 Quy trình cải tiến lặp lại của model</p>
</div>

Đây là một quá trình lặp lại liên tục: huấn luyện model → đánh giá hiệu suất → phân tích lỗi → xác định vấn đề → chọn hướng cải thiện → huấn luyện lại. Thông qua nhiều lần lặp lại, hiệu suất của model sẽ không ngừng nâng cao.

## 11.6 Thực hành quy trình huấn luyện hoàn chỉnh

Trong các mục trước, chúng ta đã lần lượt học về chuẩn bị dữ liệu, huấn luyện SFT, huấn luyện GRPO và đánh giá model. Bây giờ, hãy tích hợp những kiến thức này lại, hoàn thành một quy trình huấn luyện Agentic RL end-to-end.

### 11.6.1 Quy trình huấn luyện end-to-end

Một quy trình huấn luyện Agentic RL hoàn chỉnh bao gồm các giai đoạn sau: chuẩn bị dữ liệu, huấn luyện SFT, đánh giá SFT, huấn luyện GRPO, đánh giá GRPO, triển khai model. Như Hình 11.9 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-9.png" alt="" width="85%"/>
  <p>Hình 11.9 Quy trình huấn luyện end-to-end</p>
</div>

Hãy dùng một script hoàn chỉnh để triển khai quy trình này:

```python
"""
完整的Agentic RL训练流程
从数据准备到模型部署的端到端示例
"""

from hello_agents.tools import RLTrainingTool
import json
from datetime import datetime

class AgenticRLPipeline:
    """Agentic RL训练流水线"""
    
    def __init__(self, config_path="config.json"):
        """
        初始化训练流水线
        
        Args:
            config_path: 配置文件路径
        """
        self.rl_tool = RLTrainingTool()
        self.config = self.load_config(config_path)
        self.results = {}
        
    def load_config(self, config_path):
        """加载配置文件"""
        with open(config_path, 'r') as f:
            return json.load(f)
    
    def log(self, message):
        """记录日志"""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{timestamp}] {message}")
    
    def stage1_prepare_data(self):
        """阶段1: 数据准备"""
        self.log("=" * 50)
        self.log("阶段1: 数据准备")
        self.log("=" * 50)

        # 加载并检查数据集
        result = self.rl_tool.run({
            "action": "load_dataset",
            "format": "sft",
            "max_samples": self.config["data"]["max_samples"],
        })

        # 解析JSON结果
        dataset_info = json.loads(result)

        self.log(f"✓ 数据集加载完成")
        self.log(f"  - 样本数: {dataset_info['dataset_size']}")
        self.log(f"  - 格式: {dataset_info['format']}")
        self.log(f"  - 数据列: {', '.join(dataset_info['sample_keys'])}")

        self.results["data"] = dataset_info

        return dataset_info
    
    def stage2_sft_training(self):
        """阶段2: SFT训练"""
        self.log("\n" + "=" * 50)
        self.log("阶段2: SFT训练")
        self.log("=" * 50)

        sft_config = self.config["sft"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "sft",
            "model_name": self.config["model"]["base_model"],
            "output_dir": sft_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": sft_config["num_epochs"],
            "batch_size": sft_config["batch_size"],
            "use_lora": True,
            # 训练监控配置
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # 解析JSON结果
        result_data = json.loads(result)

        self.log(f"✓ SFT训练完成")
        self.log(f"  - 模型路径: {result_data['output_dir']}")
        self.log(f"  - 状态: {result_data['status']}")

        self.results["sft_training"] = result_data

        return result_data["output_dir"]
    
    def stage3_sft_evaluation(self, model_path):
        """阶段3: SFT评估"""
        self.log("\n" + "=" * 50)
        self.log("阶段3: SFT评估")
        self.log("=" * 50)
        
        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ SFT评估完成")
        self.log(f"  - 准确率: {eval_data['accuracy']}")
        self.log(f"  - 平均奖励: {eval_data['average_reward']}")

        self.results["sft_evaluation"] = eval_data

        return eval_data
    
    def stage4_grpo_training(self, sft_model_path):
        """阶段4: GRPO训练"""
        self.log("\n" + "=" * 50)
        self.log("阶段4: GRPO训练")
        self.log("=" * 50)

        grpo_config = self.config["grpo"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "grpo",
            "model_name": sft_model_path,
            "output_dir": grpo_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": grpo_config["num_epochs"],
            "batch_size": grpo_config["batch_size"],
            "use_lora": True,
            # 训练监控配置
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # 解析JSON结果
        result_data = json.loads(result)

        self.log(f"✓ GRPO训练完成")
        self.log(f"  - 模型路径: {result_data['output_dir']}")
        self.log(f"  - 状态: {result_data['status']}")

        self.results["grpo_training"] = result_data

        return result_data["output_dir"]
    
    def stage5_grpo_evaluation(self, model_path):
        """阶段5: GRPO评估"""
        self.log("\n" + "=" * 50)
        self.log("阶段5: GRPO评估")
        self.log("=" * 50)
        
        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ GRPO评估完成")
        self.log(f"  - 准确率: {eval_data['accuracy']}")
        self.log(f"  - 平均奖励: {eval_data['average_reward']}")

        self.results["grpo_evaluation"] = eval_data

        return eval_data
    
    def stage6_save_results(self):
        """阶段6: 保存结果"""
        self.log("\n" + "=" * 50)
        self.log("阶段6: 保存结果")
        self.log("=" * 50)
        
        # 保存训练结果
        results_path = "training_results.json"
        with open(results_path, 'w') as f:
            json.dump(self.results, f, indent=2)
        
        self.log(f"✓ 结果已保存到: {results_path}")
    
    def run(self):
        """运行完整流程"""
        try:
            # 阶段1: 数据准备
            self.stage1_prepare_data()
            
            # 阶段2: SFT训练
            sft_model_path = self.stage2_sft_training()
            
            # 阶段3: SFT评估
            self.stage3_sft_evaluation(sft_model_path)
            
            # 阶段4: GRPO训练
            grpo_model_path = self.stage4_grpo_training(sft_model_path)
            
            # 阶段5: GRPO评估
            self.stage5_grpo_evaluation(grpo_model_path)
            
            # 阶段6: 保存结果
            self.stage6_save_results()
            
            self.log("\n" + "=" * 50)
            self.log("✓ 训练流程完成!")
            self.log("=" * 50)
            
        except Exception as e:
            self.log(f"\n✗ 训练失败: {str(e)}")
            raise

# 使用示例
if __name__ == "__main__":
    # 创建配置文件
    config = {
        "model": {
            "base_model": "Qwen/Qwen3-0.6B"
        },
        "data": {
            "max_samples": 1000  # 使用1000个样本
        },
        "sft": {
            "output_dir": "./models/sft_model",
            "num_epochs": 3,
            "batch_size": 8,
        },
        "grpo": {
            "output_dir": "./models/grpo_model",
            "num_epochs": 3,
            "batch_size": 4,
        },
        "eval": {
            "max_samples": 200,
            "sft_accuracy_threshold": 0.40  # SFT准确率阈值
        },
        "monitoring": {
            "use_wandb": False,  # 是否使用Wandb
            "use_tensorboard": True,  # 是否使用TensorBoard
            "wandb_project": "agentic-rl-pipeline"  # Wandb项目名
        }
    }
    
    # 保存配置
    with open("config.json", 'w') as f:
        json.dump(config, f, indent=2)
    
    # 运行训练流程
    pipeline = AgenticRLPipeline("config.json")
    pipeline.run()
```

Chạy script này, bạn sẽ thấy toàn bộ quá trình huấn luyện.

Một số gợi ý khi chạy:

<strong>Bắt đầu từ quy mô nhỏ</strong>: đừng dùng ngay toàn bộ dữ liệu để huấn luyện ngay từ đầu. Hãy dùng 100-1000 mẫu để lặp nhanh trước, xác nhận quy trình và tham số, sau khi xác nhận hiệu quả rồi mới mở rộng quy mô. Cách này có thể tiết kiệm rất nhiều thời gian và tài nguyên tính toán.

<strong>Kiểm tra chất lượng dữ liệu</strong>: trước khi huấn luyện cần kiểm tra chất lượng dữ liệu, đảm bảo định dạng đúng, đáp án chính xác, không có mẫu trùng lặp. Có thể dùng code sau:

```python
def check_data_quality(dataset):
    """检查数据质量"""
    issues = []

    # 检查必需字段
    required_fields = ["prompt", "completion"]
    for field in required_fields:
        if field not in dataset.column_names:
            issues.append(f"缺少字段: {field}")

    # 检查空值
    for i, sample in enumerate(dataset):
        if not sample["prompt"] or not sample["completion"]:
            issues.append(f"样本{i}包含空值")

    # 检查重复
    prompts = [s["prompt"] for s in dataset]
    duplicates = len(prompts) - len(set(prompts))
    if duplicates > 0:
        issues.append(f"发现{duplicates}个重复样本")

    return issues

# 使用
issues = check_data_quality(dataset)
if issues:
    print("数据质量问题:")
    for issue in issues:
        print(f"  - {issue}")
else:
    print("✓ 数据质量检查通过")
```

<strong>Data Augmentation</strong>: nếu lượng dữ liệu không đủ, có thể cân nhắc data augmentation, như viết lại câu hỏi (giữ nguyên đáp án), sinh câu hỏi tương tự, back-translation (translate back). Nhưng cần chú ý giữ chất lượng dữ liệu, tránh đưa vào nhiễu (noise).

### 11.6.2 Hyperparameter Tuning

Hyperparameter tuning là chìa khóa để nâng cao hiệu suất model. Dưới đây là một số chiến lược tuning thường dùng.

<strong>（1）Grid Search</strong>

Grid Search là phương pháp tuning đơn giản nhất, duyệt qua tất cả các tổ hợp tham số, chọn ra tổ hợp tốt nhất.

```python
# 定义参数网格
param_grid = {
    "learning_rate": [1e-5, 5e-5, 1e-4],
    "lora_rank": [8, 16, 32],
    "kl_coef": [0.05, 0.1, 0.2],
}

best_accuracy = 0
best_params = None

# 遍历所有组合
for lr in param_grid["learning_rate"]:
    for rank in param_grid["lora_rank"]:
        for kl in param_grid["kl_coef"]:
            print(f"测试参数: lr={lr}, rank={rank}, kl={kl}")

            # 训练模型
            result = rl_tool.run({
                "action": "train",
                "algorithm": "grpo",
                "learning_rate": lr,
                "lora_rank": rank,
                "kl_coef": kl,
                # 其他参数...
            })

            # 评估模型
            eval_result = rl_tool.run({
                "action": "evaluate",
                "model_path": result["model_path"],
            })

            # 更新最佳参数
            if eval_result["accuracy"] > best_accuracy:
                best_accuracy = eval_result["accuracy"]
                best_params = {"lr": lr, "rank": rank, "kl": kl}

print(f"最佳参数: {best_params}")
print(f"最佳准确率: {best_accuracy:.2%}")
```

Ưu điểm của Grid Search là đơn giản trực tiếp, có thể tìm được tối ưu toàn cục. Nhược điểm là chi phí tính toán cao, không khả thi khi số lượng tham số nhiều.

<strong>（2）Random Search</strong>

Random Search lấy mẫu ngẫu nhiên tổ hợp tham số, hiệu quả hơn Grid Search.

```python
import random

# 定义参数范围
param_ranges = {
    "learning_rate": (1e-6, 1e-4),  # 对数均匀分布
    "lora_rank": [4, 8, 16, 32, 64],
    "kl_coef": (0.01, 0.5),
}

best_accuracy = 0
best_params = None

# 随机采样N次
N = 10
for i in range(N):
    # 随机采样参数
    lr = 10 ** random.uniform(-6, -4)  # 对数均匀
    rank = random.choice(param_ranges["lora_rank"])
    kl = random.uniform(0.01, 0.5)

    print(f"[{i+1}/{N}] 测试参数: lr={lr:.2e}, rank={rank}, kl={kl:.3f}")

    # 训练和评估(同上)
    # ...

print(f"最佳参数: {best_params}")
print(f"最佳准确率: {best_accuracy:.2%}")
```

Ưu điểm của Random Search là hiệu quả cao, phù hợp với trường hợp không gian tham số lớn. Nhược điểm là có thể bỏ lỡ nghiệm tối ưu.

<strong>（3）Bayesian Optimization</strong>

Bayesian Optimization dùng mô hình xác suất để dẫn dắt việc tìm kiếm, thông minh hơn. Có thể dùng thư viện Optuna:

```python
import optuna

def objective(trial):
    """优化目标函数"""
    # 采样参数
    lr = trial.suggest_loguniform("learning_rate", 1e-6, 1e-4)
    rank = trial.suggest_categorical("lora_rank", [8, 16, 32])
    kl = trial.suggest_uniform("kl_coef", 0.01, 0.5)

    # 训练模型
    result = rl_tool.run({
        "action": "train",
        "algorithm": "grpo",
        "learning_rate": lr,
        "lora_rank": rank,
        "kl_coef": kl,
        # 其他参数...
    })

    # 评估模型
    eval_result = rl_tool.run({
        "action": "evaluate",
        "model_path": result["model_path"],
    })

    return eval_result["accuracy"]

# 创建研究
study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=20)

# 打印最佳参数
print(f"最佳参数: {study.best_params}")
print(f"最佳准确率: {study.best_value:.2%}")
```

Ưu điểm của Bayesian Optimization là hiệu quả mẫu cao, có thể nhanh chóng tìm ra tham số tốt. Nhược điểm là triển khai phức tạp, cần thêm thư viện bổ sung.

Như Bảng 11.8 thể hiện, so sánh các phương pháp tuning khác nhau.

<div align="center">
  <p>Bảng 11.8 So sánh các phương pháp Hyperparameter Tuning</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-8.png" alt="" width="85%"/>
</div>

### 11.6.3 Distributed Training

Khi lượng dữ liệu và quy mô model tăng lên, huấn luyện single-GPU sẽ trở nên rất chậm. Lúc này chúng ta cần dùng distributed training để tăng tốc quá trình huấn luyện. HelloAgents dựa trên TRL và Hugging Face Accelerate, hỗ trợ sẵn distributed training đa GPU và đa node.

<strong>Gợi ý lựa chọn phương án</strong>:

- <strong>Nhiều GPU trên một máy (2-8 GPU)</strong>: dùng DDP, đơn giản và hiệu quả
- <strong>Model lớn (>7B)</strong>: dùng DeepSpeed ZeRO-2 hoặc ZeRO-3
- <strong>Cụm nhiều node</strong>: dùng DeepSpeed ZeRO-3 + Offload

<strong>（1）Cấu hình Accelerate</strong>

Trước tiên cần tạo file cấu hình Accelerate. Chạy lệnh sau:

```bash
accelerate config
```

Theo gợi ý để chọn cấu hình:

```
In which compute environment are you running?
> This machine

Which type of machine are you using?
> multi-GPU

How many different machines will you use?
> 1

Do you wish to optimize your script with torch dynamo?
> NO

Do you want to use DeepSpeed?
> YES

Which DeepSpeed config file do you want to use?
> ZeRO-2

How many GPU(s) should be used for distributed training?
> 4
```

Cách này sẽ sinh ra file cấu hình tại `~/.cache/huggingface/accelerate/default_config.yaml`.

<strong>（2）Huấn luyện với DDP</strong>

<strong>Data Parallel (DDP)</strong> là phương án distributed đơn giản nhất, mỗi GPU giữ một bản sao model hoàn chỉnh, dữ liệu được chia nhỏ ra các GPU.

<strong>File cấu hình Accelerate</strong> (`multi_gpu_ddp.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_GPU
num_processes: 4  # GPU数量
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
```

<strong>Script huấn luyện</strong> (không cần sửa đổi):

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 训练代码完全不变
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_ddp",
    "num_epochs": 3,
    "batch_size": 4,  # 每个GPU的batch size
    "use_lora": True,
})
```

<strong>Khởi chạy huấn luyện</strong>:

```bash
# 使用配置文件
accelerate launch --config_file multi_gpu_ddp.yaml train_script.py

# 或者直接指定参数
accelerate launch --num_processes 4 --mixed_precision fp16 train_script.py
```

</strong>（3）Huấn luyện với DeepSpeed ZeRO</strong>

</strong>DeepSpeed ZeRO</strong> thông qua việc phân mảnh (shard) optimizer state, gradient và tham số model, giảm đáng kể mức tiêu thụ VRAM, hỗ trợ model và batch size lớn hơn.

</strong>File cấu hình ZeRO-2</strong> (`deepspeed_zero2.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: none
  offload_param_device: none
  zero3_init_flag: false
  zero_stage: 2  # ZeRO-2
```

</strong>File cấu hình ZeRO-3</strong> (`deepspeed_zero3.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: cpu  # 优化器状态卸载到CPU
  offload_param_device: cpu      # 参数卸载到CPU
  zero3_init_flag: true
  zero_stage: 3  # ZeRO-3
```

<strong>Khởi chạy huấn luyện</strong>:

```bash
# ZeRO-2
accelerate launch --config_file deepspeed_zero2.yaml train_script.py

# ZeRO-3
accelerate launch --config_file deepspeed_zero3.yaml train_script.py
```

Như Bảng 11.9 thể hiện, đây là so sánh mức tiêu thụ VRAM khi huấn luyện model Qwen3-0.6B theo các cách khác nhau:

<div align="center">
  <p>Bảng 11.9 So sánh mức tiêu thụ VRAM (model Qwen3-0.6B)</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-9.png" alt="" width="85%"/>
</div>

<strong>（4）Huấn luyện đa node</strong>

Đối với huấn luyện quy mô siêu lớn, có thể dùng nhiều node (máy).

<strong>Cấu hình node chính</strong> (`multi_node_main.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 16  # 4节点 x 4GPU
machine_rank: 0    # 主节点
num_machines: 4
main_process_ip: 192.168.1.100  # 主节点IP
main_process_port: 29500
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  zero_stage: 3
  offload_optimizer_device: cpu
  offload_param_device: cpu
```

<strong>Cấu hình node phụ (worker)</strong> (sửa `machine_rank` thành 1, 2, 3):

```yaml
machine_rank: 1  # 工作节点1
# 其他配置相同
```

<strong>Khởi chạy huấn luyện</strong>:

```bash
# 在主节点上
accelerate launch --config_file multi_node_main.yaml train_script.py

# 在工作节点1上
accelerate launch --config_file multi_node_worker1.yaml train_script.py

# 在工作节点2上
accelerate launch --config_file multi_node_worker2.yaml train_script.py

# 在工作节点3上
accelerate launch --config_file multi_node_worker3.yaml train_script.py
```

<strong>（5）Best practice cho Distributed Training</strong>

<strong>1. Điều chỉnh Batch Size</strong>

Khi distributed training, tổng batch size = `per_device_batch_size × num_gpus × gradient_accumulation_steps`

```python
# 单GPU: batch_size=4, gradient_accumulation=4, 总batch=16
# 4GPU DDP: batch_size=4, gradient_accumulation=1, 总batch=16 (保持一致)
```

<strong>2. Scale Learning Rate</strong>

Dùng quy tắc scale tuyến tính: `lr_new = lr_base × sqrt(total_batch_size_new / total_batch_size_base)`

```python
# 基准: 单GPU, batch=16, lr=5e-5
# 4GPU: batch=64, lr=5e-5 × sqrt(64/16) = 1e-4
```

<strong>3. Giám sát và Debug</strong>

```python
# 启用详细日志
export ACCELERATE_LOG_LEVEL=INFO

# 启用NCCL调试(多节点)
export NCCL_DEBUG=INFO

# 检查GPU利用率
watch -n 1 nvidia-smi
```

### 11.6.4 Triển khai Production

Sau khi huấn luyện xong, chúng ta cần triển khai model lên môi trường production. Dưới đây là một số gợi ý triển khai.

<strong>（1）Export model</strong>

Hợp nhất (merge) trọng số LoRA vào model gốc, thuận tiện cho việc triển khai:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# 加载基础模型
base_model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-0.6B")

# 加载LoRA权重
model = PeftModel.from_pretrained(base_model, "./models/grpo_model")

# 合并权重
merged_model = model.merge_and_unload()

# 保存合并后的模型
merged_model.save_pretrained("./models/merged_model")

# 保存tokenizer
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen3-0.6B")
tokenizer.save_pretrained("./models/merged_model")

print("✓ 模型已导出到: ./models/merged_model")
```

<strong>（2）Tối ưu hóa Inference</strong>

Dùng kỹ thuật quantization và tối ưu hóa để tăng tốc inference:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# 加载模型(使用8-bit量化)
model = AutoModelForCausalLM.from_pretrained(
    "./models/merged_model",
    load_in_8bit=True,  # 8-bit量化
    device_map="auto",  # 自动分配设备
)

tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

# 推理
def generate_answer(question):
    prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    outputs = model.generate(
        **inputs,
        max_new_tokens=512,
        temperature=0.7,
        do_sample=True,
    )

    response = tokenizer.decode(outputs[0], skip_special_tokens=False)
    return response

# 测试
question = "What is 48 + 24?"
answer = generate_answer(question)
print(answer)
```

<strong>（3）API Service</strong>

Dùng FastAPI để tạo inference service:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import AutoModelForCausalLM, AutoTokenizer

app = FastAPI()

# 加载模型
model = AutoModelForCausalLM.from_pretrained("./models/merged_model")
tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

class Question(BaseModel):
    text: str
    max_tokens: int = 512

class Answer(BaseModel):
    text: str
    confidence: float

@app.post("/generate", response_model=Answer)
def generate(question: Question):
    """生成答案"""
    prompt = f"<|im_start|>user\n{question.text}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt")

    outputs = model.generate(
        **inputs,
        max_new_tokens=question.max_tokens,
        temperature=0.7,
        return_dict_in_generate=True,
        output_scores=True,
    )

    response = tokenizer.decode(outputs.sequences[0], skip_special_tokens=False)

    # 计算置信度(简化版)
    confidence = 0.8  # 实际应该基于输出概率计算

    return Answer(text=response, confidence=confidence)

# 运行: uvicorn api:app --host 0.0.0.0 --port 8000
```



## 11.8 Tổng kết chương

Trong chương này, chúng ta đã học một cách có hệ thống lý thuyết và thực hành của Agentic RL, từ khái niệm cơ bản đến quy trình huấn luyện hoàn chỉnh, từ chuẩn bị dữ liệu đến triển khai model. Hãy cùng ôn lại nội dung chính của chương này.

<strong>（1）Bản chất của Agentic RL</strong>

Agentic RL là việc coi LLM như một policy có thể học được, nhúng vào vòng lặp cảm nhận-quyết định-thực thi của Agent, thông qua Reinforcement Learning để tối ưu hóa hiệu suất của Agent trong các tác vụ nhiều bước. Khác biệt cốt lõi của nó so với PBRFT (Preference-Based Reinforcement Fine-Tuning) truyền thống nằm ở:

- <strong>Bản chất tác vụ</strong>: từ tối ưu hóa hội thoại đơn lượt mở rộng thành ra quyết định tuần tự nhiều bước
- <strong>Không gian state</strong>: từ prompt tĩnh mở rộng thành state môi trường tiến hóa động
- <strong>Không gian action</strong>: từ sinh văn bản thuần túy mở rộng thành văn bản + tool + thao tác môi trường
- <strong>Thiết kế reward</strong>: từ đánh giá chất lượng đơn bước mở rộng thành lợi ích tích lũy dài hạn
- <strong>Mục tiêu tối ưu hóa</strong>: từ chất lượng phản hồi ngắn hạn mở rộng thành thành công của tác vụ dài hạn

<strong>（2）Sáu năng lực cốt lõi</strong>

Agentic RL nhằm nâng cao sáu năng lực cốt lõi của Agent:

1. <strong>Reasoning</strong>: suy luận logic nhiều bước, học chiến lược reasoning
2. <strong>Tool Use</strong>: gọi API/tool, học khi nào dùng, dùng như thế nào
3. <strong>Memory</strong>: duy trì thông tin dài hạn, học quản lý memory
4. <strong>Planning</strong>: lập kế hoạch chuỗi hành động, học planning động
5. <strong>Self-Improvement</strong>: tự phản tư và tối ưu hóa, học từ sai lầm
6. <strong>Perception</strong>: hiểu đa phương thức, visual reasoning và sử dụng visual tool

<strong>（3）Quy trình huấn luyện</strong>

Quy trình huấn luyện Agentic RL hoàn chỉnh bao gồm:

1. <strong>Pretraining</strong>: học kiến thức ngôn ngữ trên văn bản quy mô lớn (thường dùng model đã pretrain sẵn có)
2. <strong>Supervised Fine-Tuning (SFT)</strong>: học định dạng tác vụ và khả năng reasoning cơ bản
3. <strong>Reinforcement Learning (RL)</strong>: thông qua thử-sai tối ưu hóa chiến lược reasoning, vượt qua chất lượng dữ liệu huấn luyện

Trong đó, SFT là nền tảng, RL là bước nâng cao. Không có nền tảng SFT, RL rất khó thành công; không có tối ưu hóa của RL, model chỉ có thể bắt chước dữ liệu huấn luyện.

Nếu bạn muốn học sâu hơn về Agentic RL, khuyến nghị theo lộ trình sau:

Giai đoạn cơ bản

1. <strong>Kiến thức nền tảng Reinforcement Learning</strong>: học các khái niệm cơ bản như MDP, policy gradient, PPO
2. <strong>Kiến thức nền tảng LLM</strong>: tìm hiểu các kỹ thuật như Transformer, pretraining, fine-tuning
3. <strong>Thực hành HelloAgents</strong>: chạy code ví dụ trong chương này, hiểu toàn bộ quy trình

Giai đoạn nâng cao

1. <strong>Đi sâu vào TRL</strong>: học cách triển khai thư viện TRL, hiểu chi tiết các thuật toán như SFT và GRPO
2. <strong>Dataset tùy chỉnh</strong>: dùng dataset của riêng mình để huấn luyện model
3. <strong>Reward function tùy chỉnh</strong>: thiết kế reward function phù hợp với tác vụ của riêng bạn
4. <strong>Hyperparameter tuning</strong>: điều chỉnh hyperparameter một cách có hệ thống, nâng cao hiệu suất model

Giai đoạn cao cấp

1. <strong>Reasoning nhiều bước</strong>: nghiên cứu các tác vụ reasoning chuỗi dài
2. <strong>Học sử dụng tool</strong>: giúp Agent học cách sử dụng tool
3. <strong>Multi-Agent</strong>: nghiên cứu hợp tác đa Agent
4. <strong>Các bài báo tiên phong</strong>: đọc các bài báo nghiên cứu mới nhất, theo dõi tiến bộ của lĩnh vực



Hy vọng chương này có thể giúp bạn hiểu và nắm vững kỹ thuật Agentic RL, áp dụng những kiến thức này trong dự án của riêng mình, xây dựng nên những hệ thống Agent thông minh hơn!



## Tài liệu tham khảo

[1] Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal Policy Optimization Algorithms. *arXiv preprint arXiv:1707.06347*.

[2] Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Zhang, M., ... & Guo, D. (2024). DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models. *arXiv preprint arXiv:2402.03300*.

[3] Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... & Chen, W. (2021). LoRA: Low-Rank Adaptation of Large Language Models. *arXiv preprint arXiv:2106.09685*.

[4] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., ... & Schulman, J. (2021). Training Verifiers to Solve Math Word Problems. *arXiv preprint arXiv:2110.14168*.

[5] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., ... & Lowe, R. (2022). Training language models to follow instructions with human feedback. *Advances in Neural Information Processing Systems*, 35, 27730-27744.

[6] Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C. (2023). Direct Preference Optimization: Your Language Model is Secretly a Reward Model. *arXiv preprint arXiv:2305.18290*.

[7] Lee, H., Phatale, S., Mansoor, H., Lu, K., Mesnard, T., Bishop, C., ... & Rastogi, A. (2023). RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback. *arXiv preprint arXiv:2309.00267*.

[8] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., ... & Zhou, D. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. *Advances in Neural Information Processing Systems*, 35, 24824-24837.

[9] von Werra, L., Belkada, Y., Tunstall, L., Beeching, E., Thrush, T., Lambert, N., & Huang, S. (2020). TRL: Transformer Reinforcement Learning. *GitHub repository*. https://github.com/huggingface/trl

[10] Qwen Team. (2025). Qwen3 Technical Report. *arXiv preprint arXiv:2505.09388*.

[11] Bai, Y., Jones, A., Ndousse, K., Askell, A., Chen, A., DasSarma, N., ... & Kaplan, J. (2022). Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback. *arXiv preprint arXiv:2204.05862*.

[12] Wang, X., Wei, J., Schuurmans, D., Le, Q., Chi, E., Narang, S., ... & Zhou, D. (2022). Self-Consistency Improves Chain of Thought Reasoning in Language Models. *arXiv preprint arXiv:2203.11171*.

[13] Christiano, P. F., Leike, J., Brown, T., Martic, M., Legg, S., & Amodei, D. (2017). Deep Reinforcement Learning from Human Preferences. *Advances in Neural Information Processing Systems*, 30.

[14] Stiennon, N., Ouyang, L., Wu, J., Ziegler, D., Lowe, R., Voss, C., ... & Christiano, P. F. (2020). Learning to summarize with human feedback. *Advances in Neural Information Processing Systems*, 33, 3008-3021.

[15] Ziegler, D. M., Stiennon, N., Wu, J., Brown, T. B., Radford, A., Amodei, D., ... & Irving, G. (2019). Fine-Tuning Language Models from Human Preferences. *arXiv preprint arXiv:1909.08593*.

## Bài tập

> <strong>Gợi ý</strong>: một số bài tập không có đáp án chuẩn, trọng tâm là rèn luyện khả năng hiểu và thực hành toàn diện của người học về Agentic RL và huấn luyện Agent.

1. Chương này đã giới thiệu quá trình tiến hóa từ huấn luyện LLM đến Agentic RL. Hãy phân tích:

   - Trong Bảng 11.1 của mục 11.1.3, đã so sánh sự khác biệt giữa PBRFT (Preference-Based Reinforcement Fine-Tuning) và Agentic RL dưới framework MDP. Hãy giải thích sâu hơn: tại sao không gian state $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$ của Agentic RL chứa quan sát lịch sử, trong khi state $s_0 = \text{prompt}$ của PBRFT chỉ chứa prompt ban đầu? Sự khác biệt này ảnh hưởng thế nào đến quá trình huấn luyện và hiệu quả cuối cùng?
   - Giả sử bạn muốn huấn luyện một "trợ lý debug code thông minh", nó cần: (1) phân tích code để tìm ra bug; (2) tra cứu tài liệu để hiểu cách dùng API; (3) sửa code; (4) chạy test để kiểm chứng kết quả sửa lỗi. Hãy ánh xạ tác vụ này vào framework Reinforcement Learning, định nghĩa rõ không gian state, không gian action, reward function và hàm chuyển trạng thái.
   - Trong mục 11.1.1 có đề cập supervised learning truyền thống tồn tại hạn chế "khó tối ưu hóa mục tiêu dài hạn". Hãy thiết kế một tác vụ reasoning nhiều bước cụ thể (như chứng minh toán học, giải bài toán phức tạp), thể hiện tại sao supervised learning khó tối ưu hóa các bước trung gian, trong khi Reinforcement Learning có thể giải quyết vấn đề này thông qua delayed reward.

2. SFT (Supervised Fine-Tuning) và GRPO (Group Relative Policy Optimization) là hai phương pháp huấn luyện cốt lõi của chương này. Dựa trên nội dung mục 11.2 và 11.3, hãy suy nghĩ sâu hơn:

   > <strong>Gợi ý</strong>: đây là bài tập thực hành, khuyến nghị thực hành thực tế

   - Trong code huấn luyện SFT ở mục 11.2.4, chúng ta đã dùng kỹ thuật LoRA (Low-Rank Adaptation) để giảm số tham số huấn luyện. Hãy phân tích: ý tưởng cốt lõi của LoRA là gì? Tại sao nó có thể dùng một lượng nhỏ tham số (như 0.16%) để đạt hiệu quả gần với full-parameter fine-tuning? Trong trường hợp nào nên chọn LoRA thay vì full-parameter fine-tuning?
   - Thuật toán GRPO (mục 11.3) so với thuật toán PPO truyền thống có ưu điểm gì? Hãy so sánh quy trình huấn luyện của hai thuật toán, phân tích cách GRPO đơn giản hóa quy trình huấn luyện và nâng cao tính ổn định thông qua "group relative reward". Nếu muốn áp dụng GRPO vào các tác vụ khác (như sinh code, tối ưu hóa hội thoại), cần điều chỉnh những gì?
   - Dựa trên code ở mục 11.2.5, hãy mở rộng quy trình huấn luyện SFT, thêm các chức năng sau: (1) hỗ trợ huấn luyện dữ liệu hội thoại nhiều lượt; (2) thêm chiến lược data augmentation (như viết lại đồng nghĩa, điều chỉnh độ khó); (3) triển khai giám sát trực quan quá trình huấn luyện (như đường cong loss, đánh giá chất lượng mẫu).

3. Thiết kế reward function là thách thức cốt lõi của Agentic RL. Dựa trên nội dung mục 11.3.3, hãy hoàn thành các bài thực hành mở rộng sau:

   > <strong>Gợi ý</strong>: đây là bài tập thực hành, khuyến nghị thực hành thực tế

   - Trong mục 11.3.3, chúng ta đã thiết kế một binary reward đơn giản (đúng +1, sai 0) cho bài toán toán học GSM8K. Hãy thiết kế một reward function tinh vi hơn, có thể: (1) cấp một phần reward cho đáp án đúng một phần; (2) chấm điểm tính hợp lý của quá trình reasoning; (3) phạt các con đường giải quá dài dòng hoặc kém hiệu quả. Reward function này nên được triển khai như thế nào?
   - Thiết kế reward function thường cần kiến thức chuyên môn (domain knowledge). Hãy thiết kế reward function cho ba tác vụ Agent khác nhau sau: (1) trợ lý sinh code (cần xem xét tính đúng đắn của code, khả năng đọc hiểu, hiệu suất); (2) Agent hội thoại chăm sóc khách hàng (cần xem xét tỷ lệ giải quyết vấn đề, mức độ hài lòng của người dùng, thời gian phản hồi); (3) game AI (cần xem xét tỷ lệ thắng, tính đa dạng của chiến lược, độ bền vững trước đối kháng).
   - Trong ứng dụng thực tế, reward function có thể tồn tại vấn đề "reward hacking": Agent tìm ra con đường tắt để nhận reward cao, nhưng thực sự không hoàn thành tác vụ. Hãy đưa ra ví dụ minh họa hiện tượng này, và thiết kế cơ chế phòng vệ để tránh reward hacking.

4. Trong case study "huấn luyện Agent reasoning toán học" ở mục 11.4, chúng ta đã thấy quy trình huấn luyện hoàn chỉnh. Hãy phân tích sâu hơn:

   - Case study đã dùng dataset GSM8K để huấn luyện và đánh giá. Hãy phân tích: đặc điểm của dataset này là gì? Nó phù hợp để huấn luyện loại khả năng reasoning nào? Nếu muốn huấn luyện một Agent có thể xử lý các bài toán toán học phức tạp hơn (như toán cao cấp, chứng minh toán học), nên mở rộng dataset và phương pháp huấn luyện như thế nào?
   - Trong kết quả huấn luyện ở mục 11.4.3, chúng ta quan sát thấy accuracy trên training set tăng lên, nhưng có thể tồn tại rủi ro overfitting. Hãy thiết kế một phương án "đánh giá khả năng tổng quát hóa": làm thế nào để kiểm tra xem model có thực sự học được reasoning toán học, hay chỉ ghi nhớ dữ liệu huấn luyện? Làm thế nào để nâng cao khả năng tổng quát hóa thông qua regularization, data augmentation?
   - Huấn luyện trong case study là offline (dùng dataset đã thu thập sẵn). Hãy thiết kế một phương án "online learning": Agent liên tục thu thập feedback của người dùng trong quá trình sử dụng thực tế, và tự động cập nhật model. Phương án này cần xem xét những thách thức kỹ thuật nào (như kiểm soát chất lượng dữ liệu, catastrophic forgetting, đảm bảo an toàn)?

5. Một ứng dụng quan trọng của Agentic RL là giúp Agent học cách sử dụng tool. Hãy suy nghĩ:

   - Trong mục 11.1.3 có đề cập Agentic RL phù hợp để tối ưu hóa các tác vụ "cần reasoning nhiều bước, sử dụng tool, lập kế hoạch dài hạn". Hãy thiết kế một phương án huấn luyện "học sử dụng tool": cho một tập hợp tool (như search engine, calculator, code executor), làm thế nào để huấn luyện Agent học cách chọn tool phù hợp vào đúng thời điểm? Reward function nên được thiết kế như thế nào?
   - Việc sử dụng tool thường liên quan đến các mối quan hệ phụ thuộc phức tạp (như "phải gọi tool A để lấy thông tin trước, mới có thể gọi tool B"). Hãy thiết kế một phương án "Hierarchical Reinforcement Learning": high-level policy chịu trách nhiệm lập kế hoạch tác vụ, low-level policy chịu trách nhiệm gọi tool. Cấu trúc phân tầng này nên được huấn luyện như thế nào? Làm thế nào để phối hợp mục tiêu tối ưu hóa giữa high-level và low-level?
   - Trong ứng dụng thực tế, số lượng tool có thể rất nhiều (như hơn 50 API), huấn luyện trực tiếp có thể gặp vấn đề "hiệu quả exploration thấp". Hãy thiết kế một phương án "curriculum learning": bắt đầu huấn luyện từ tác vụ đơn giản (dùng ít tool), sau đó tăng dần độ khó của tác vụ và số lượng tool. Phương án này nên thiết kế thứ tự curriculum như thế nào? Làm thế nào để đánh giá Agent đã sẵn sàng bước vào giai đoạn tiếp theo hay chưa?
