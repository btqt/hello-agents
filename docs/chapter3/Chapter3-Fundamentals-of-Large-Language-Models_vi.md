# Chương 3: Nền tảng về Large Language Model

Hai chương trước đã lần lượt giới thiệu định nghĩa và lịch sử phát triển của Agent, chương này sẽ tập trung hoàn toàn vào bản thân Large Language Model để giải đáp một câu hỏi then chốt: các Agent hiện đại hoạt động như thế nào? Chúng ta sẽ xuất phát từ định nghĩa cơ bản của mô hình ngôn ngữ, thông qua việc học các nguyên lý này, đặt nền móng vững chắc để hiểu LLM có được kho tri thức phong phú và khả năng suy luận mạnh mẽ như thế nào.

## 3.1 Mô hình ngôn ngữ và kiến trúc Transformer

### 3.1.1 Từ N-gram đến RNN

<strong>Mô hình ngôn ngữ (Language Model, LM)</strong> là cốt lõi của xử lý ngôn ngữ tự nhiên, nhiệm vụ căn bản của nó là tính toán xác suất xuất hiện của một chuỗi từ (tức một câu). Một mô hình ngôn ngữ tốt có thể cho chúng ta biết câu nào là trôi chảy, tự nhiên. Trong hệ thống đa Agent, mô hình ngôn ngữ là nền tảng để Agent hiểu chỉ thị của con người và sinh ra phản hồi. Phần này sẽ điểm lại quá trình phát triển từ các phương pháp thống kê kinh điển đến các mô hình deep learning hiện đại, đặt nền tảng vững chắc cho việc hiểu kiến trúc Transformer ở phần sau.

<strong>(1) Mô hình ngôn ngữ thống kê và ý tưởng N-gram</strong>

Trước khi deep learning nổi lên, phương pháp thống kê là chủ đạo của mô hình ngôn ngữ. Ý tưởng cốt lõi của nó là: xác suất xuất hiện của một câu bằng tích các xác suất có điều kiện của từng từ trong câu đó. Đối với một câu S được tạo thành từ các từ $w_1,w_2,\cdots,w_m$, xác suất P(S) có thể biểu diễn như sau:

$$P(S)=P(w_1,w_2,…,w_m)=P(w_1)⋅P(w_2∣w_1)⋅P(w_3∣w_1,w_2)⋯P(w_m∣w_1,…,w_{m−1})$$

Công thức này được gọi là quy tắc dây chuyền xác suất (chain rule). Tuy nhiên, việc tính trực tiếp công thức này gần như bất khả thi, vì các xác suất có điều kiện như $P(w_m∣w_1,\cdots,w_{m−1})$ quá khó để ước lượng từ kho ngữ liệu (corpus), bởi chuỗi từ $w_1,\cdots,w_{m−1}$ có thể chưa từng xuất hiện trong dữ liệu huấn luyện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-0.png" alt="图片描述" width="90%"/>
  <p>Hình 3.1 Minh họa giả định Markov</p>
</div>

Để giải quyết vấn đề này, các nhà nghiên cứu đã đưa ra <strong>giả định Markov (Markov Assumption)</strong>. Ý tưởng cốt lõi của nó là: chúng ta không cần truy hồi toàn bộ lịch sử của một từ, mà có thể xấp xỉ rằng xác suất xuất hiện của một từ chỉ liên quan đến $n−1$ từ hữu hạn phía trước nó, như minh họa ở Hình 3.1. Mô hình ngôn ngữ được xây dựng dựa trên giả định này được gọi là <strong>mô hình N-gram</strong>. "N" ở đây đại diện cho kích thước cửa sổ ngữ cảnh mà chúng ta xem xét. Hãy cùng xem một vài ví dụ phổ biến nhất để hiểu khái niệm này:

- <strong>Bigram (khi N=2)</strong>: Đây là trường hợp đơn giản nhất, chúng ta giả định rằng sự xuất hiện của một từ chỉ liên quan đến một từ đứng trước nó. Do đó, xác suất có điều kiện phức tạp $P(w_i∣w_1,\cdots,w_{i−1})$ trong quy tắc dây chuyền có thể được xấp xỉ thành dạng dễ tính toán hơn:

$$P(w_{i}∣w_{1},…,w_{i−1})≈P(w_{i}∣w_{i−1})$$

- <strong>Trigram (khi N=3)</strong>: Tương tự, chúng ta giả định rằng sự xuất hiện của một từ chỉ liên quan đến hai từ đứng trước nó:

$$P(w_i∣w_1,…,w_{i−1})≈P(w_i∣w_{i−2},w_{i−1})$$

Các xác suất này có thể được tính thông qua <strong>ước lượng hợp lý cực đại (Maximum Likelihood Estimation, MLE)</strong> trên một kho ngữ liệu lớn. Thuật ngữ này nghe có vẻ phức tạp, nhưng ý tưởng của nó rất trực quan: cái nào có khả năng xuất hiện nhất chính là cái mà chúng ta thấy xuất hiện nhiều lần nhất trong dữ liệu. Ví dụ, đối với mô hình Bigram, chúng ta muốn tính xác suất $P(w_i∣w_{i−1})$ rằng sau khi từ $w_{i−1}$ xuất hiện, từ tiếp theo là $w_i$. Theo ước lượng hợp lý cực đại, xác suất này có thể được ước tính bằng cách đếm đơn giản:

$$P(w_i∣w_{i−1})=\frac{Count(w_{i−1},w_i)}{Count(w_{i−1})}$$

Hàm `Count()` ở đây đại diện cho "đếm số lần":

- $Count(w_{i−1},w_i)$: biểu thị tổng số lần cặp từ $(w_{i−1},w_i)$ xuất hiện liên tiếp trong kho ngữ liệu.
- $Count(w_{i−1})$: biểu thị tổng số lần từ $w_{i−1}$ xuất hiện trong kho ngữ liệu.

Ý nghĩa của công thức là: chúng ta lấy "số lần cặp từ $Count(w_{i−1},w_i)$ xuất hiện" chia cho "tổng số lần từ $Count(w_{i−1})$ xuất hiện", để làm ước lượng xấp xỉ cho $P(w_i∣w_{i−1})$.

Để làm cho quá trình này cụ thể hơn, chúng ta hãy thực hiện một phép tính thủ công. Giả sử chúng ta có một kho ngữ liệu mini chỉ gồm hai câu sau: `datawhale agent learns`, `datawhale agent works`. Mục tiêu của chúng ta là: sử dụng mô hình Bigram (N=2) để ước tính xác suất xuất hiện của câu `datawhale agent learns`. Theo giả định của Bigram, mỗi lần chúng ta sẽ xem xét hai từ liên tiếp (tức một cặp từ).

<strong>Bước một: Tính xác suất của từ đầu tiên</strong> $P(datawhale)$ Đây là số lần `datawhale` xuất hiện chia cho tổng số từ. `datawhale` xuất hiện 2 lần, tổng số từ là 6.

$$P(\text{datawhale}) = \frac{\text{总语料中"datawhale"的数量}}{\text{总语料的词数}} = \frac{2}{6} \approx 0.333$$

<strong>Bước hai: Tính xác suất có điều kiện</strong> $P(agent∣datawhale)$ Đây là số lần cặp từ `datawhale agent` xuất hiện chia cho tổng số lần `datawhale` xuất hiện. `datawhale agent` xuất hiện 2 lần, `datawhale` xuất hiện 2 lần.

$$P(\text{agent}|\text{datawhale}) =  \frac{\text{Count}(\text{datawhale agent})}{\text{Count}(\text{datawhale})} =  \frac{2}{2} = 1$$

<strong>Bước ba: Tính xác suất có điều kiện</strong> $P(learns∣agent)$ Đây là số lần cặp từ `agent learns` xuất hiện chia cho tổng số lần `agent` xuất hiện. `agent learns` xuất hiện 1 lần, `agent` xuất hiện 2 lần.

$$P(\text{learns}|\text{agent}) =  \frac{\text{Count(agent learns)}}{\text{Count(agent)}} =  \frac{1}{2} = 0.5$$

<strong>Cuối cùng: Nhân các xác suất lại với nhau</strong> Vậy, xác suất xấp xỉ của toàn bộ câu là:

$$P(\text{datawhale agent learns}) \approx  P(\text{datawhale}) \cdot  P(\text{agent}|\text{datawhale}) \cdot  P(\text{learns}|\text{agent}) \approx  0.333 \cdot 1 \cdot 0.5 \approx 0.167$$

```Python
import collections

# Kho ngữ liệu ví dụ, nhất quán với kho ngữ liệu trong phần giải thích ở trên
corpus = "datawhale agent learns datawhale agent works"
tokens = corpus.split()
total_tokens = len(tokens)

# --- Bước một: Tính P(datawhale) ---
count_datawhale = tokens.count('datawhale')
p_datawhale = count_datawhale / total_tokens
print(f"第一步: P(datawhale) = {count_datawhale}/{total_tokens} = {p_datawhale:.3f}")

# --- Bước hai: Tính P(agent|datawhale) ---
# Tính bigram trước để dùng cho các bước sau
bigrams = zip(tokens, tokens[1:])
bigram_counts = collections.Counter(bigrams)
count_datawhale_agent = bigram_counts[('datawhale', 'agent')]
# count_datawhale đã được tính ở bước một
p_agent_given_datawhale = count_datawhale_agent / count_datawhale
print(f"第二步: P(agent|datawhale) = {count_datawhale_agent}/{count_datawhale} = {p_agent_given_datawhale:.3f}")

# --- Bước ba: Tính P(learns|agent) ---
count_agent_learns = bigram_counts[('agent', 'learns')]
count_agent = tokens.count('agent')
p_learns_given_agent = count_agent_learns / count_agent
print(f"第三步: P(learns|agent) = {count_agent_learns}/{count_agent} = {p_learns_given_agent:.3f}")

# --- Cuối cùng: Nhân các xác suất lại với nhau ---
p_sentence = p_datawhale * p_agent_given_datawhale * p_learns_given_agent
print(f"最后: P('datawhale agent learns') ≈ {p_datawhale:.3f} * {p_agent_given_datawhale:.3f} * {p_learns_given_agent:.3f} = {p_sentence:.3f}")

>>>
第一步: P(datawhale) = 2/6 = 0.333
第二步: P(agent|datawhale) = 2/2 = 1.000
第三步: P(learns|agent) = 1/2 = 0.500
最后: P('datawhale agent learns') ≈ 0.333 * 1.000 * 0.500 = 0.167
```

Mô hình N-gram tuy đơn giản và hiệu quả, nhưng có hai nhược điểm chí mạng:

1. <strong>Tính thưa thớt dữ liệu (Sparsity)</strong>: Nếu một chuỗi từ chưa từng xuất hiện trong kho ngữ liệu, ước lượng xác suất của nó sẽ bằng 0, điều này rõ ràng là không hợp lý. Mặc dù có thể giảm thiểu bằng kỹ thuật làm mượt (Smoothing), nhưng không thể loại bỏ hoàn toàn.
2. <strong>Khả năng tổng quát hóa kém:</strong> Mô hình không thể hiểu được sự tương đồng ngữ nghĩa giữa các từ. Ví dụ, ngay cả khi mô hình đã thấy `agent learns` nhiều lần trong kho ngữ liệu, nó cũng không thể tổng quát hóa tri thức này sang các từ có ngữ nghĩa tương tự. Khi chúng ta tính xác suất của `robot learns`, nếu từ `robot` chưa từng xuất hiện, hoặc tổ hợp `robot learns` chưa từng xuất hiện, xác suất mà mô hình tính ra cũng sẽ bằng 0. Mô hình không thể hiểu được sự tương đồng về ngữ nghĩa giữa `agent` và `robot`.

<strong>(2) Mô hình ngôn ngữ mạng nơ-ron và word embedding</strong>

Nhược điểm căn bản của mô hình N-gram nằm ở chỗ nó coi mỗi từ là một ký hiệu rời rạc, cô lập. Để khắc phục vấn đề này, các nhà nghiên cứu đã chuyển sang mạng nơ-ron, và đề xuất một ý tưởng: dùng các vector liên tục để biểu diễn từ. Năm 2003, <strong>mô hình ngôn ngữ mạng nơ-ron truyền thẳng (Feedforward Neural Network Language Model)</strong> do Bengio và cộng sự đề xuất là một cột mốc trong lĩnh vực này<sup>[1]</sup>.

Ý tưởng cốt lõi của nó có thể chia thành hai bước:

1. <strong>Xây dựng một không gian ngữ nghĩa</strong>: Tạo ra một không gian vector liên tục nhiều chiều, sau đó ánh xạ mỗi từ trong bộ từ vựng thành một điểm trong không gian đó. Điểm này (tức vector) được gọi là <strong>Word Embedding</strong> hay vector từ. Trong không gian này, các từ gần nhau về ngữ nghĩa thì vector tương ứng của chúng cũng nằm gần nhau trong không gian. Ví dụ, vector của `agent` và `robot` sẽ gần nhau, trong khi vector của `agent` và `apple` sẽ cách xa nhau.
2. <strong>Học ánh xạ từ ngữ cảnh đến từ tiếp theo</strong>: Tận dụng khả năng khớp hàm mạnh mẽ của mạng nơ-ron để học một hàm. Đầu vào của hàm này là các vector từ của $n−1$ từ trước đó, đầu ra là phân phối xác suất xuất hiện của mỗi từ trong bộ từ vựng sau ngữ cảnh hiện tại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-1.png" alt="图片描述" width="90%"/>
  <p>Hình 3.2 Minh họa kiến trúc mô hình ngôn ngữ mạng nơ-ron</p>
</div>

Như minh họa ở Hình 3.2, trong kiến trúc này, word embedding được học tự động trong quá trình huấn luyện mô hình. Để hoàn thành nhiệm vụ "dự đoán từ tiếp theo", mô hình sẽ liên tục điều chỉnh vị trí vector của mỗi từ, cuối cùng khiến các vector này chứa đựng thông tin ngữ nghĩa phong phú. Một khi chúng ta đã chuyển đổi từ thành vector, chúng ta có thể dùng các công cụ toán học để đo lường quan hệ giữa chúng. Phương pháp thường dùng nhất là <strong>độ tương đồng cosine (Cosine Similarity)</strong>, nó đo lường độ tương đồng giữa hai vector bằng cách tính giá trị cosine của góc giữa chúng.

$$\text{similarity}(\vec{a}, \vec{b}) = \cos(\theta) = \frac{\vec{a} \cdot \vec{b}}{|\vec{a}| |\vec{b}|}$$

Ý nghĩa của công thức này là:

- Nếu hai vector có hướng hoàn toàn giống nhau, góc là 0°, giá trị cosine là 1, biểu thị tương quan hoàn toàn.
- Nếu hai vector vuông góc, góc là 90°, giá trị cosine là 0, biểu thị không có quan hệ gì.
- Nếu hai vector có hướng hoàn toàn ngược nhau, góc là 180°, giá trị cosine là -1, biểu thị tương quan nghịch hoàn toàn.

Thông qua cách này, vector từ không chỉ có thể nắm bắt các quan hệ đơn giản như "từ đồng nghĩa", mà còn có thể nắm bắt các quan hệ loại suy phức tạp hơn.

Một ví dụ nổi tiếng cho thấy quan hệ ngữ nghĩa mà vector từ nắm bắt được: kết quả của phép toán vector `vector('King') - vector('Man') + vector('Woman')` nằm gần một cách đáng kinh ngạc với vị trí của `vector('Queen')` trong không gian vector. Điều này giống như đang thực hiện một phép tịnh tiến ngữ nghĩa: chúng ta xuất phát từ điểm "King" (Vua), trừ đi vector "Man" (Nam), rồi cộng thêm vector "Woman" (Nữ), cuối cùng đến được vị trí của "Queen" (Nữ hoàng). Điều này chứng minh rằng word embedding có khả năng học được các khái niệm trừu tượng như "giới tính", "hoàng tộc".

```Python
import numpy as np

# Giả sử chúng ta đã học được các vector từ hai chiều đơn giản hóa
embeddings = {
    "king": np.array([0.9, 0.8]),
    "queen": np.array([0.9, 0.2]),
    "man": np.array([0.7, 0.9]),
    "woman": np.array([0.7, 0.3])
}

def cosine_similarity(vec1, vec2):
    dot_product = np.dot(vec1, vec2)
    norm_product = np.linalg.norm(vec1) * np.linalg.norm(vec2)
    return dot_product / norm_product

# king - man + woman
result_vec = embeddings["king"] - embeddings["man"] + embeddings["woman"]

# Tính độ tương đồng giữa vector kết quả và "queen"
sim = cosine_similarity(result_vec, embeddings["queen"])

print(f"king - man + woman 的结果向量: {result_vec}")
print(f"该结果与 'queen' 的相似度: {sim:.4f}")

>>>
king - man + woman 的结果向量: [0.9 0.2]
该结果与 'queen' 的相似度: 1.0000
```

Mô hình ngôn ngữ mạng nơ-ron, thông qua word embedding, đã giải quyết thành công vấn đề khả năng tổng quát hóa kém của mô hình N-gram. Tuy nhiên, nó vẫn còn một hạn chế tương tự N-gram: cửa sổ ngữ cảnh cố định. Nó chỉ có thể xem xét một lượng văn bản trước đó cố định, điều này đặt tiền đề cho mạng nơ-ron hồi quy có khả năng xử lý chuỗi có độ dài tùy ý.

<strong>(3) Mạng nơ-ron hồi quy (RNN) và mạng bộ nhớ dài-ngắn hạn (LSTM)</strong>

Mô hình ngôn ngữ mạng nơ-ron ở phần trước tuy đã đưa vào word embedding để giải quyết vấn đề tổng quát hóa, nhưng nó cũng giống mô hình N-gram, có cửa sổ ngữ cảnh với kích thước cố định. Để dự đoán từ tiếp theo, nó chỉ có thể nhìn thấy n−1 từ trước đó, còn thông tin lịch sử xa hơn sẽ bị bỏ qua. Điều này rõ ràng không phù hợp với cách con người chúng ta hiểu ngôn ngữ. Để phá vỡ giới hạn cửa sổ cố định, <strong>mạng nơ-ron hồi quy (Recurrent Neural Network, RNN)</strong> ra đời, ý tưởng cốt lõi của nó rất trực quan: bổ sung khả năng "ghi nhớ" cho mạng<sup>[2]</sup>.

Như minh họa ở Hình 3.3, thiết kế của RNN đưa vào một vector <strong>trạng thái ẩn (hidden state)</strong>, chúng ta có thể hiểu nó như bộ nhớ ngắn hạn của mạng. Ở mỗi bước xử lý chuỗi, mạng sẽ đọc từ đầu vào hiện tại, kết hợp với ký ức ở thời điểm trước đó (tức trạng thái ẩn của bước thời gian trước), sau đó sinh ra một ký ức mới (tức trạng thái ẩn của bước thời gian hiện tại) truyền cho bước tiếp theo. Quá trình lặp đi lặp lại này khiến thông tin có thể liên tục truyền về sau trong chuỗi.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-2.png" alt="图片描述" width="90%"/>
  <p>Hình 3.3 Minh họa cấu trúc RNN</p>
</div>

Tuy nhiên, RNN tiêu chuẩn tồn tại một vấn đề nghiêm trọng trong thực tế: <strong>vấn đề phụ thuộc dài hạn (Long-term Dependency Problem)</strong>. Trong quá trình huấn luyện, mô hình cần thông qua thuật toán lan truyền ngược (backpropagation) để điều chỉnh trọng số ở các lớp sâu của mạng dựa trên sai số ở đầu ra. Đối với RNN, độ dài của chuỗi chính là độ sâu của mạng. Khi chuỗi rất dài, gradient khi lan truyền ngược từ sau ra trước sẽ trải qua nhiều phép nhân liên tiếp, điều này khiến giá trị gradient nhanh chóng tiến về 0 (<strong>vanishing gradient</strong>) hoặc trở nên cực lớn (<strong>exploding gradient</strong>). Vanishing gradient khiến mô hình không thể học hiệu quả ảnh hưởng của thông tin ở đầu chuỗi đến đầu ra ở cuối chuỗi, tức khó nắm bắt được các quan hệ phụ thuộc khoảng cách xa.

Để giải quyết vấn đề phụ thuộc dài hạn, <strong>mạng bộ nhớ dài-ngắn hạn (Long Short-Term Memory, LSTM)</strong> đã được thiết kế ra<sup>[3]</sup>. LSTM là một loại RNN đặc biệt, đổi mới cốt lõi của nó nằm ở việc đưa vào <strong>trạng thái tế bào (Cell State)</strong> và một bộ <strong>cơ chế cổng (Gating Mechanism)</strong> tinh vi. Trạng thái tế bào có thể được xem như một kênh thông tin độc lập với trạng thái ẩn, cho phép thông tin truyền đi mượt mà hơn giữa các bước thời gian. Cơ chế cổng được cấu thành từ vài mạng nơ-ron nhỏ, chúng có thể học cách chọn lọc cho thông tin đi qua, từ đó kiểm soát việc thêm vào và loại bỏ thông tin trong trạng thái tế bào. Các cổng này bao gồm:

- <strong>Cổng quên (Forget Gate)</strong>: quyết định loại bỏ thông tin nào từ trạng thái tế bào ở thời điểm trước.
- <strong>Cổng đầu vào (Input Gate)</strong>: quyết định lưu thông tin mới nào từ đầu vào hiện tại vào trạng thái tế bào.
- <strong>Cổng đầu ra (Output Gate)</strong>: quyết định xuất thông tin nào ra trạng thái ẩn dựa trên trạng thái tế bào hiện tại.

### 3.1.2 Phân tích kiến trúc Transformer

Ở phần trước, chúng ta đã thấy RNN và LSTM đưa vào cấu trúc hồi quy để xử lý dữ liệu chuỗi, điều này đã giải quyết được phần nào vấn đề nắm bắt các phụ thuộc khoảng cách xa. Tuy nhiên, cách tính toán hồi quy này cũng mang lại một nút thắt cổ chai mới: nó buộc phải xử lý dữ liệu theo thứ tự. Việc tính toán ở bước thời gian thứ t phải chờ bước thời gian thứ t−1 hoàn thành mới có thể bắt đầu. Điều này có nghĩa RNN không thể thực hiện tính toán song song quy mô lớn, hiệu quả thấp khi xử lý chuỗi dài, điều này hạn chế rất lớn khả năng mở rộng quy mô mô hình và tăng tốc độ huấn luyện. Transformer được nhóm của Google đề xuất vào năm 2017<sup>[4]</sup>. Nó hoàn toàn từ bỏ cấu trúc hồi quy, mà hoàn toàn dựa vào một cơ chế gọi là <strong>Attention</strong> để nắm bắt các quan hệ phụ thuộc trong chuỗi, từ đó đạt được tính toán song song đúng nghĩa.

<strong>(1) Cấu trúc tổng thể Encoder-Decoder</strong>

Mô hình Transformer ban đầu được thiết kế cho tác vụ dịch máy đầu-cuối (end-to-end). Như minh họa ở Hình 3.4, về mặt vĩ mô nó tuân theo kiến trúc kinh điển <strong>Encoder-Decoder</strong>.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-3.png" alt="图片描述" width="50%"/>
  <p>Hình 3.4 Sơ đồ kiến trúc tổng thể của Transformer</p>
</div>


Chúng ta có thể hiểu cấu trúc này như một đội ngũ có phân công rõ ràng:

1. <strong>Encoder</strong>: nhiệm vụ là "<strong>hiểu</strong>" toàn bộ câu đầu vào. Nó sẽ đọc tất cả các token đầu vào (khái niệm này sẽ được giới thiệu ở mục 3.2.2), cuối cùng sinh ra cho mỗi token một biểu diễn vector chứa đựng thông tin ngữ cảnh phong phú.
2. <strong>Decoder</strong>: nhiệm vụ là "<strong>sinh</strong>" câu đích. Nó sẽ tham khảo phần văn bản nó đã sinh ra trước đó, đồng thời "tham vấn" kết quả hiểu của encoder, để sinh ra từ tiếp theo.

Để thực sự hiểu được nguyên lý hoạt động của Transformer, cách tốt nhất không gì khác hơn là tự tay cài đặt nó. Trong phần này, chúng ta sẽ áp dụng phương pháp "từ trên xuống" (top-down): trước tiên, chúng ta sẽ xây dựng khung code hoàn chỉnh của Transformer, định nghĩa tất cả các class và method cần thiết. Sau đó, chúng ta sẽ hiện thực hóa chức năng cụ thể của từng class này giống như đang ghép một mảnh ghép hình.

```Python
import torch
import torch.nn as nn
import math

# --- Các module placeholder, sẽ được hiện thực hóa ở các phần sau ---

class PositionalEncoding(nn.Module):
    """
    Module positional encoding
    """
    def forward(self, x):
        pass

class MultiHeadAttention(nn.Module):
    """
    Module cơ chế multi-head attention
    """
    def forward(self, query, key, value, mask):
        pass

class PositionWiseFeedForward(nn.Module):
    """
    Module position-wise feed-forward network
    """
    def forward(self, x):
        pass

# --- Lớp cốt lõi của Encoder ---

class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(EncoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # Sẽ hiện thực hóa
        self.feed_forward = PositionWiseFeedForward() # Sẽ hiện thực hóa
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, mask):
        # Residual connection và layer normalization sẽ được giải thích chi tiết ở mục 3.1.2.4
        # 1. Multi-head self-attention
        attn_output = self.self_attn(x, x, x, mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Feed-forward network
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))

        return x

# --- Lớp cốt lõi của Decoder ---

class DecoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(DecoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # Sẽ hiện thực hóa
        self.cross_attn = MultiHeadAttention() # Sẽ hiện thực hóa
        self.feed_forward = PositionWiseFeedForward() # Sẽ hiện thực hóa
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, encoder_output, src_mask, tgt_mask):
        # 1. Masked multi-head self-attention (đối với chính nó)
        attn_output = self.self_attn(x, x, x, tgt_mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Cross-attention (đối với đầu ra của encoder)
        cross_attn_output = self.cross_attn(x, encoder_output, encoder_output, src_mask)
        x = self.norm2(x + self.dropout(cross_attn_output))

        # 3. Feed-forward network
        ff_output = self.feed_forward(x)
        x = self.norm3(x + self.dropout(ff_output))

        return x
```

<strong>(2) Từ Self-Attention đến Multi-Head Attention</strong>

Bây giờ, chúng ta sẽ lấp đầy module then chốt nhất trong khung sườn: cơ chế attention.

Hãy tưởng tượng chúng ta đang đọc câu: "The agent learns because <strong>it</strong> is intelligent." Khi đọc đến từ in đậm "<strong>it</strong>", để hiểu nó đang chỉ đến ai, bộ não chúng ta sẽ không tự chủ mà tập trung chú ý nhiều hơn vào từ "agent" ở phía trước. Cơ chế <strong>Self-Attention</strong> chính là mô hình hóa toán học cho hiện tượng này. Nó cho phép mô hình, khi xử lý mỗi từ trong chuỗi, có thể xem xét đồng thời tất cả các từ khác trong câu, và gán cho các từ này các "trọng số attention" khác nhau. Từ có trọng số càng cao thì mức độ liên quan với từ hiện tại càng mạnh, thông tin của nó cũng nên chiếm tỷ trọng lớn hơn trong biểu diễn của từ hiện tại.

Để thực hiện quá trình trên, cơ chế self-attention đưa vào ba vai trò có thể học cho mỗi vector token đầu vào:

- <strong>Query (Q)</strong>: đại diện cho token hiện tại, nó đang chủ động "truy vấn" các token khác để lấy thông tin.
- <strong>Key (K)</strong>: đại diện cho "nhãn" hoặc "chỉ mục" của token có thể được truy vấn trong câu.
- <strong>Value (V)</strong>: đại diện cho "nội dung" hoặc "thông tin" mà bản thân token mang theo.

Ba vector này đều được tính bằng cách nhân vector word embedding gốc với ba ma trận trọng số có thể học khác nhau ($W^Q,W^K,W^V$). Toàn bộ quá trình tính toán có thể chia thành các bước sau, chúng ta có thể hình dung nó như một kỳ thi mở (open-book) hiệu quả:

- Chuẩn bị "đề thi" và "tài liệu": đối với mỗi từ trong câu, đều sinh ra vector $Q,K,V$ của nó thông qua ma trận trọng số.
- Tính điểm liên quan: để tính biểu diễn mới của từ $A$, dùng vector $Q$ của từ $A$ để thực hiện phép tích vô hướng (dot product) với vector $K$ của tất cả các từ trong câu (bao gồm cả $A$). Điểm số này phản ánh mức độ quan trọng của các từ khác đối với việc hiểu từ $A$.
- Ổn định hóa và chuẩn hóa: chia tất cả các điểm số thu được cho một hệ số tỷ lệ $\sqrt{d_{k}}$ ($d_{k}$ là số chiều của vector $K$), để tránh gradient quá nhỏ, sau đó dùng hàm Softmax để chuyển các điểm số thành các trọng số có tổng bằng 1, tức quá trình chuẩn hóa.
- Tổng có trọng số: nhân các trọng số thu được ở bước trên với vector $V$ tương ứng của mỗi từ, sau đó cộng tất cả các kết quả lại. Vector cuối cùng thu được chính là biểu diễn mới của từ $A$ sau khi đã kết hợp thông tin ngữ cảnh toàn cục.

Quá trình này có thể được khái quát bằng một công thức ngắn gọn:

$$\text{Attention}(Q,K,V)=\text{softmax}\left(\frac{QK^{T}}{\sqrt{d_{k}}}\right)V$$

Nếu chỉ thực hiện một lần tính toán attention như trên (tức single-head), mô hình có thể chỉ học được cách chú ý đến một loại quan hệ. Ví dụ, khi xử lý "it", có thể chỉ học được cách chú ý đến chủ ngữ. Nhưng quan hệ trong ngôn ngữ rất phức tạp, chúng ta mong muốn mô hình có thể đồng thời chú ý đến nhiều loại quan hệ (như quan hệ chỉ định, quan hệ thì, quan hệ phụ thuộc, v.v.). Cơ chế multi-head attention ra đời từ đó. Ý tưởng của nó rất đơn giản: biến một lần làm xong thành chia thành nhiều nhóm, làm riêng, rồi hợp lại.

Nó chia vector Q, K, V gốc thành h phần theo chiều (h chính là số "head"), mỗi phần độc lập thực hiện một lần tính toán single-head attention. Điều này giống như việc để h "chuyên gia" khác nhau xem xét câu từ các góc độ khác nhau, mỗi chuyên gia đều có thể nắm bắt một loại quan hệ đặc trưng khác nhau. Cuối cùng, ghép (concatenate) "ý kiến" (tức vector đầu ra) của h chuyên gia này lại, rồi thực hiện một phép biến đổi tuyến tính để tích hợp, ta sẽ thu được đầu ra cuối cùng.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-4.png" alt="图片描述" width="50%"/>
  <p>Hình 3.5 Cơ chế Multi-Head Attention</p>
</div>


Như minh họa ở Hình 3.5, thiết kế này cho phép mô hình đồng thời chú ý đến thông tin từ các vị trí khác nhau, các không gian con biểu diễn khác nhau, tăng cường mạnh mẽ khả năng biểu đạt của mô hình. Dưới đây là một cách hiện thực hóa đơn giản của multi-head attention để tham khảo.

```Python
class MultiHeadAttention(nn.Module):
    """
    Module cơ chế multi-head attention
    """
    def __init__(self, d_model, num_heads):
        super(MultiHeadAttention, self).__init__()
        assert d_model % num_heads == 0, "d_model 必须能被 num_heads 整除"

        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Định nghĩa các lớp biến đổi tuyến tính cho Q, K, V và đầu ra
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)

    def scaled_dot_product_attention(self, Q, K, V, mask=None):
        # 1. Tính điểm attention (QK^T)
        attn_scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)

        # 2. Áp dụng mask (nếu có)
        if mask is not None:
            # Đặt các vị trí có giá trị mask bằng 0 thành một số âm rất nhỏ, như vậy sau softmax sẽ gần bằng 0
            attn_scores = attn_scores.masked_fill(mask == 0, -1e9)

        # 3. Tính trọng số attention (Softmax)
        attn_probs = torch.softmax(attn_scores, dim=-1)

        # 4. Tổng có trọng số (trọng số * V)
        output = torch.matmul(attn_probs, V)
        return output

    def split_heads(self, x):
        # Biến đổi hình dạng của x từ (batch_size, seq_length, d_model)
        # thành (batch_size, num_heads, seq_length, d_k)
        batch_size, seq_length, d_model = x.size()
        return x.view(batch_size, seq_length, self.num_heads, self.d_k).transpose(1, 2)

    def combine_heads(self, x):
        # Biến đổi hình dạng của x từ (batch_size, num_heads, seq_length, d_k)
        # trở lại (batch_size, seq_length, d_model)
        batch_size, num_heads, seq_length, d_k = x.size()
        return x.transpose(1, 2).contiguous().view(batch_size, seq_length, self.d_model)

    def forward(self, Q, K, V, mask=None):
        # 1. Thực hiện biến đổi tuyến tính cho Q, K, V
        Q = self.split_heads(self.W_q(Q))
        K = self.split_heads(self.W_k(K))
        V = self.split_heads(self.W_v(V))

        # 2. Tính scaled dot-product attention
        attn_output = self.scaled_dot_product_attention(Q, K, V, mask)

        # 3. Kết hợp đầu ra của các head và thực hiện biến đổi tuyến tính cuối cùng
        output = self.W_o(self.combine_heads(attn_output))
        return output
```

<strong>(3) Mạng Feed-Forward</strong>

Trong mỗi lớp Encoder và Decoder, sau lớp con multi-head attention đều có một <strong>Position-wise Feed-Forward Network (FFN)</strong>. Nếu nói tác dụng của lớp attention là "tổng hợp động" thông tin liên quan từ toàn bộ chuỗi, thì tác dụng của feed-forward network là trích xuất các đặc trưng bậc cao hơn từ thông tin đã tổng hợp này.

Điểm mấu chốt của cái tên này nằm ở "theo vị trí" (position-wise). Nó có nghĩa là mạng feed-forward này sẽ tác động độc lập lên mỗi vector token trong chuỗi. Nói cách khác, đối với một chuỗi có độ dài `seq_len`, FFN này thực tế sẽ được gọi `seq_len` lần, mỗi lần xử lý một token. Điều quan trọng là, tất cả các vị trí dùng chung một bộ trọng số mạng. Thiết kế này vừa duy trì khả năng xử lý độc lập cho từng vị trí, vừa giảm đáng kể số lượng tham số của mô hình. Cấu trúc của mạng này rất đơn giản, gồm hai phép biến đổi tuyến tính và một hàm kích hoạt ReLU:

$$\mathrm{FFN}(x)=\max\left(0, xW_{1}+b_{1}\right) W_{2}+b_{2}$$

Trong đó, $x$ là đầu ra của lớp con attention. $W_1,b_1,W_2,b_2$ là các tham số có thể học. Thông thường, số chiều đầu ra `d_ff` của lớp tuyến tính đầu tiên sẽ lớn hơn nhiều so với số chiều đầu vào `d_model` (ví dụ `d_ff = 4 * d_model`), sau khi kích hoạt ReLU sẽ được ánh xạ trở lại số chiều `d_model` qua lớp tuyến tính thứ hai. Mô hình "mở rộng trước, thu nhỏ sau" này được cho là giúp mô hình học được biểu diễn đặc trưng phong phú hơn.

Trong khung sườn PyTorch của chúng ta, chúng ta có thể dùng đoạn code sau để hiện thực hóa module này:

```Python
class PositionWiseFeedForward(nn.Module):
    """
    Module position-wise feed-forward network
    """
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(PositionWiseFeedForward, self).__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.dropout = nn.Dropout(dropout)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.relu = nn.ReLU()

    def forward(self, x):
        # Hình dạng x: (batch_size, seq_len, d_model)
        x = self.linear1(x)
        x = self.relu(x)
        x = self.dropout(x)
        x = self.linear2(x)
        # Hình dạng đầu ra cuối cùng: (batch_size, seq_len, d_model)
        return x
```

<strong>(4) Residual Connection và Layer Normalization</strong>

Trong mỗi lớp encoder và decoder của Transformer, tất cả các module con (như multi-head attention và feed-forward network) đều được bao bọc bởi một phép toán `Add & Norm`. Sự kết hợp này nhằm đảm bảo Transformer có thể được huấn luyện ổn định.

Phép toán này gồm hai phần:

- <strong>Residual Connection (Add)</strong>: phép toán này cộng trực tiếp đầu vào `x` của module con vào đầu ra `Sublayer(x)` của module con đó. Cấu trúc này giải quyết vấn đề <strong>vanishing gradient</strong> trong mạng nơ-ron sâu. Khi lan truyền ngược, gradient có thể vòng qua module con để truyền trực tiếp về phía trước, từ đó đảm bảo mô hình vẫn có thể được huấn luyện hiệu quả ngay cả khi mạng có nhiều lớp. Công thức của nó có thể biểu diễn là: $\text{Output} = x + \text{Sublayer}(x)$.
- <strong>Layer Normalization (Norm)</strong>: phép toán này chuẩn hóa tất cả các đặc trưng của một mẫu đơn, khiến giá trị trung bình của nó bằng 0, phương sai bằng 1. Điều này giải quyết vấn đề <strong>Internal Covariate Shift</strong> trong quá trình huấn luyện mô hình, giữ cho phân phối đầu vào của mỗi lớp ổn định, từ đó tăng tốc độ hội tụ của mô hình và nâng cao độ ổn định của huấn luyện.

<strong>3.1.2.5 Positional Encoding</strong>

Chúng ta đã biết, cốt lõi của Transformer là cơ chế self-attention, nó nắm bắt các quan hệ phụ thuộc bằng cách tính quan hệ giữa hai token bất kỳ trong chuỗi. Tuy nhiên, cách tính toán này có một vấn đề cố hữu: bản thân nó không chứa bất kỳ thông tin nào về thứ tự hay vị trí của token. Đối với self-attention, chuỗi "agent learns" và "learns agent" là hoàn toàn tương đương nhau, vì nó chỉ quan tâm đến quan hệ giữa các token, mà bỏ qua trật tự sắp xếp của chúng. Để giải quyết vấn đề này, Transformer đưa vào <strong>Positional Encoding</strong>.

Ý tưởng cốt lõi của positional encoding là, cộng thêm một "vector vị trí" có thể đại diện cho thông tin vị trí tuyệt đối và tương đối vào mỗi vector embedding token trong chuỗi đầu vào. Vector vị trí này không được học mà được tính trực tiếp thông qua một công thức toán học cố định. Như vậy, ngay cả khi embedding của bản thân hai token (ví dụ, hai token đều là `agent`) giống hệt nhau, nhưng do vị trí của chúng trong câu khác nhau, vector cuối cùng được đưa vào mô hình Transformer sẽ trở nên duy nhất vì được cộng thêm các positional encoding khác nhau. Positional encoding được đề xuất trong bài báo gốc sử dụng hàm sin và cos để sinh ra, công thức như sau:

$$PE_{(pos,2i)}=\sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)，$$

$$PE_{(pos,2i+1)}=\cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

Trong đó:

- $pos$ là vị trí của token trong chuỗi (ví dụ, $0$, $1$, $2$, ...)
- $i$ là chỉ số chiều trong vector vị trí (từ $0$ đến $d_{\text{model}}/2$)
- $d_{\text{model}}$ là số chiều của vector word embedding (nhất quán với mô hình chúng ta đã định nghĩa)

Bây giờ, chúng ta sẽ hiện thực hóa module `PositionalEncoding`, hoàn thành phần cuối cùng của khung code Transformer.

```Python
class PositionalEncoding(nn.Module):
    """
    Thêm positional encoding vào vector embedding của chuỗi đầu vào.
    """
    def __init__(self, d_model: int, dropout: float = 0.1, max_len: int = 5000):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)

        # Tạo một ma trận positional encoding đủ dài
        position = torch.arange(max_len).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2) * (-math.log(10000.0) / d_model))

        # Kích thước pe (positional encoding) là (max_len, d_model)
        pe = torch.zeros(max_len, d_model)

        # Chiều chẵn dùng sin, chiều lẻ dùng cos
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)

        # Đăng ký pe làm buffer, như vậy nó sẽ không được xem là tham số mô hình, nhưng sẽ di chuyển cùng mô hình (ví dụ to(device))
        self.register_buffer('pe', pe.unsqueeze(0))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x.size(1) là độ dài chuỗi của đầu vào hiện tại
        # Cộng positional encoding vào vector đầu vào
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)
```

Phần này chủ yếu nhằm giúp hiểu cấu trúc vĩ mô của Transformer và chi tiết hoạt động của từng module bên trong. Vì mục đích bổ sung hệ thống tri thức về mô hình lớn trong việc học Agent, nên chúng ta sẽ không tiếp tục đi sâu hiện thực hóa thêm. Đến đây, chúng ta đã đặt được nền tảng kiến trúc vững chắc để hiểu các Large Language Model hiện đại. Trong phần tiếp theo, chúng ta sẽ khám phá kiến trúc Decoder-Only, xem nó đã tiến hóa từ ý tưởng của Transformer như thế nào.

### 3.1.3 Kiến trúc Decoder-Only

Ở phần trước, chúng ta đã tự tay xây dựng một mô hình Transformer hoàn chỉnh, nó có thể thể hiện xuất sắc trong nhiều kịch bản đầu-cuối. Nhưng khi nhiệm vụ chuyển sang xây dựng một mô hình tổng quát để trò chuyện với con người, sáng tác, làm bộ não của Agent, có lẽ chúng ta không cần một cấu trúc phức tạp đến vậy.

Triết lý thiết kế của Transformer là "hiểu trước, sinh sau". Encoder có nhiệm vụ hiểu sâu toàn bộ câu đầu vào, hình thành một ký ức ngữ cảnh chứa thông tin toàn cục, sau đó decoder dựa trên ký ức này để sinh ra bản dịch. Nhưng khi OpenAI phát triển <strong>GPT (Generative Pre-trained Transformer)</strong>, họ đã đề xuất một ý tưởng đơn giản hơn<sup>[5]</sup>: <strong>nhiệm vụ cốt lõi của ngôn ngữ, chẳng phải là dự đoán từ có khả năng xuất hiện tiếp theo nhất hay sao?</strong>

Dù là trả lời câu hỏi, viết truyện hay sinh code, về bản chất đều là thêm nội dung hợp lý nhất vào sau một chuỗi văn bản đã có, từng từ một. Dựa trên ý tưởng này, GPT đã thực hiện một đơn giản hóa táo bạo: <strong>nó hoàn toàn từ bỏ encoder, chỉ giữ lại phần decoder.</strong> Đây chính là nguồn gốc của kiến trúc <strong>Decoder-Only</strong>.

Phương thức hoạt động của kiến trúc Decoder-Only được gọi là <strong>tự hồi quy (Autoregressive)</strong>. Thuật ngữ nghe có vẻ chuyên môn này thực ra mô tả một quá trình rất đơn giản:

1. Đưa cho mô hình một đoạn văn bản khởi đầu (ví dụ "Datawhale Agent is").
2. Mô hình dự đoán từ có khả năng xuất hiện tiếp theo nhất (ví dụ "a").
3. Mô hình thêm từ "a" vừa sinh ra vào cuối văn bản đầu vào, hình thành đầu vào mới ("Datawhale Agent is a").
4. Mô hình dựa trên đầu vào mới này, lại dự đoán từ tiếp theo (ví dụ "powerful").
5. Liên tục lặp lại quá trình này, cho đến khi sinh ra câu hoàn chỉnh hoặc đạt điều kiện dừng.

Mô hình giống như đang chơi trò chơi "nối chữ" (word chain), nó liên tục "nhìn lại" nội dung mình đã viết, rồi suy nghĩ chữ tiếp theo nên viết là gì.

Bạn có thể hỏi, khi huấn luyện mô hình thường nhận được toàn bộ văn bản cùng một lúc, vậy làm sao nó đảm bảo rằng khi học cách dự đoán từ thứ `t+1`, nó không "nhìn trộm" đáp án ở phía sau?

Câu trả lời chính là <strong>Masked Self-Attention</strong>. Trong kiến trúc Decoder-Only, cơ chế này trở nên vô cùng quan trọng. Nguyên lý hoạt động của nó rất khéo léo:

Trong giai đoạn huấn luyện, dù một đoạn văn bản có thể được đưa vào mô hình song song, nhưng sau khi cơ chế self-attention tính ra ma trận điểm attention (tức điểm chú ý của mỗi từ đối với tất cả các từ khác), và trước khi thực hiện chuẩn hóa Softmax, mô hình sẽ áp dụng một "causal mask" (mặt nạ nhân quả). Mặt nạ này sẽ thay thế điểm số tương ứng với tất cả các token nằm sau vị trí hiện tại bằng một số âm rất lớn. Khi ma trận có các điểm số âm vô cùng này đi qua hàm Softmax, xác suất ở các vị trí đó sẽ trở thành 0. Như vậy, khi tính biểu diễn tại vị trí bất kỳ, mô hình đã bị ngăn chặn về mặt toán học không cho chú ý đến thông tin ở phía sau nó.

Ở giai đoạn sinh (generation), mọi thứ trực tiếp hơn: các token trong tương lai còn chưa được sinh ra, mô hình chỉ có thể lấy nội dung đã sinh ra làm ngữ cảnh đầu vào, và dự đoán từ tiếp theo từng bước một. Masked self-attention đảm bảo rằng cách học khi huấn luyện nhất quán với cách sử dụng tự hồi quy khi sinh, từ đó khiến mô hình luôn chỉ phụ thuộc vào thông tin trước vị trí hiện tại.

<strong>Ưu điểm của kiến trúc Decoder-Only</strong>

Kiến trúc tưởng chừng đơn giản này lại mang đến thành công to lớn, ưu điểm của nó nằm ở:

- <strong>Mục tiêu huấn luyện thống nhất</strong>: nhiệm vụ duy nhất của mô hình là "dự đoán từ tiếp theo", mục tiêu đơn giản này rất phù hợp để tiền huấn luyện (pretrain) trên lượng dữ liệu văn bản không gán nhãn khổng lồ.
- <strong>Cấu trúc đơn giản, dễ mở rộng</strong>: ít thành phần hơn đồng nghĩa với việc dễ mở rộng quy mô hơn. Các mô hình khổng lồ ngày nay như GPT-4, Llama có hàng trăm tỷ thậm chí hàng nghìn tỷ tham số, đều dựa trên kiến trúc đơn giản này.
- <strong>Phù hợp tự nhiên với tác vụ sinh</strong>: phương thức hoạt động tự hồi quy của nó khớp hoàn hảo với tất cả các tác vụ sinh (đối thoại, viết lách, sinh code, v.v.), đây cũng là lý do cốt lõi khiến nó trở thành nền tảng để xây dựng Agent tổng quát.

Tóm lại, kiến trúc Decoder-Only tiến hóa từ decoder của Transformer, thông qua phương pháp đơn giản "dự đoán từ tiếp theo", đã mở ra kỷ nguyên Large Language Model mà chúng ta đang sống hiện nay.

## 3.2 Tương tác với Large Language Model

### 3.2.1 Prompt Engineering

Nếu chúng ta ví Large Language Model như một "bộ não" có năng lực cực mạnh, thì <strong>Prompt</strong> chính là ngôn ngữ chúng ta dùng để giao tiếp với "bộ não" này. Prompt Engineering là nghiên cứu cách thiết kế các prompt chính xác, từ đó dẫn dắt mô hình tạo ra phản hồi đúng như kỳ vọng của chúng ta. Đối với việc xây dựng Agent, một prompt được thiết kế tỉ mỉ có thể khiến việc hợp tác phân công giữa các Agent trở nên hiệu quả.


<strong>(1) Tham số sampling của mô hình</strong>

Khi sử dụng mô hình lớn, bạn sẽ thường thấy các tham số có thể cấu hình như `Temperature`, bản chất của nó là thông qua việc điều chỉnh chiến lược sampling đối với "phân phối xác suất", khiến đầu ra khớp với yêu cầu của kịch bản cụ thể, cấu hình tham số phù hợp có thể nâng cao hiệu năng của Agent trong kịch bản cụ thể.

Phân phối xác suất truyền thống được tính bằng công thức Softmax: $p_i = \frac{e^{z_i}}{\sum_{j=1}^k e^{z_j}}$, bản chất của tham số sampling chính là dựa trên nền tảng này, "điều chỉnh lại" hoặc "cắt bớt" phân phối theo các chiến lược khác nhau, từ đó thay đổi token tiếp theo mà mô hình lớn sinh ra.

`Temperature`: nhiệt độ là tham số then chốt kiểm soát "tính ngẫu nhiên" và "tính xác định" của đầu ra mô hình. Nguyên lý của nó là đưa vào hệ số nhiệt độ $T\gt0$, viết lại Softmax thành $p_i^{(T)} = \frac{e^{z_i / T}}{\sum_{j=1}^k e^{z_j / T}}$.

Khi T nhỏ đi, phân phối "dốc hơn", trọng số các mục có xác suất cao được khuếch đại thêm, sinh ra văn bản "bảo thủ" hơn và có tỷ lệ lặp lại cao hơn. Khi T lớn lên, phân phối "bằng phẳng hơn", trọng số các mục có xác suất thấp được nâng lên, sinh ra nội dung "đa dạng" hơn nhưng có thể xuất hiện sự thiếu mạch lạc.

- Nhiệt độ thấp (0 $\leqslant$ Temperature $\lt$ 0.3) khi đầu ra "chính xác, xác định" hơn. Kịch bản áp dụng: tác vụ thực tế: như hỏi đáp, tính toán dữ liệu, sinh code; kịch bản nghiêm ngặt: diễn giải điều khoản pháp lý, soạn thảo tài liệu kỹ thuật, giải thích khái niệm học thuật, v.v.

- Nhiệt độ trung bình (0.3 $\leqslant$ Temperature $\lt$ 0.7): đầu ra "cân bằng, tự nhiên". Kịch bản áp dụng: đối thoại thường ngày: như tương tác chăm sóc khách hàng, chatbot; sáng tác thông thường: như soạn email, viết nội dung sản phẩm, sáng tác truyện đơn giản.

- Nhiệt độ cao (0.7 $\leqslant$ Temperature $\lt$ 2): đầu ra "sáng tạo, phát tán". Kịch bản áp dụng: tác vụ sáng tạo: như sáng tác thơ, xây dựng ý tưởng truyện khoa học viễn tưởng, brainstorm slogan quảng cáo, khơi gợi cảm hứng nghệ thuật; tư duy phát tán.

`Top-k`: nguyên lý của nó là sắp xếp tất cả token theo xác suất từ cao đến thấp, lấy k token xếp hạng đầu tiên tạo thành "tập ứng viên", sau đó thực hiện "chuẩn hóa" (normalize) xác suất của k token đã được lọc ra: $ \hat{p}_i = \frac{p_i}{\sum_{j \in \text{tập ứng viên}} p_j}$

- Sự khác biệt và liên hệ với temperature sampling: temperature sampling thông qua nhiệt độ T điều chỉnh phân phối xác suất của tất cả token (làm mượt hoặc làm dốc), không thay đổi số lượng token ứng viên (vẫn xem xét toàn bộ N token). Top-k sampling thông qua giá trị k giới hạn số lượng token ứng viên (chỉ giữ lại k token có xác suất cao nhất), rồi sampling từ đó. Khi k=1, đầu ra hoàn toàn xác định, suy biến thành "greedy sampling".

`Top-p`: nguyên lý của nó là sắp xếp tất cả token theo xác suất từ cao đến thấp, bắt đầu từ token đầu tiên sau khi sắp xếp, cộng dồn xác suất từng bước, cho đến khi tổng tích lũy lần đầu đạt hoặc vượt qua ngưỡng p: $\sum_{i \in S} p_{(i)} \geq p$, lúc này tất cả token được bao gồm trong quá trình cộng dồn tạo thành "tập hạt nhân" (nucleus set), cuối cùng thực hiện chuẩn hóa cho tập hạt nhân này.

- Sự khác biệt và liên hệ với Top-k: so với Top-k có kích thước cắt cố định, Top-p có thể thích ứng động với đặc tính "đuôi dài" (long-tail) của các phân phối khác nhau, khả năng thích ứng tốt hơn với các trường hợp cực đoan khi phân phối xác suất không đồng đều.


Trong sinh văn bản, khi Top-p, Top-k và hệ số nhiệt độ được thiết lập đồng thời, các tham số này sẽ phối hợp làm việc theo cách lọc phân tầng, thứ tự ưu tiên của chúng là: điều chỉnh nhiệt độ → Top-k → Top-p. Điều chỉnh nhiệt độ tác động lên độ dốc tổng thể của phân phối, Top-k sẽ giữ lại trước k ứng viên có xác suất cao nhất, sau đó Top-p sẽ chọn ra từ kết quả của Top-k tập hợp nhỏ nhất có xác suất tích lũy ≥ p làm tập ứng viên cuối cùng. Tuy nhiên, thông thường chỉ cần chọn một trong hai Top-k và Top-p là đủ, nếu thiết lập đồng thời, tập ứng viên thực tế là giao của cả hai.
Cần lưu ý rằng, nếu đặt nhiệt độ bằng 0, thì Top-k và Top-p sẽ trở nên không còn ý nghĩa, vì token có khả năng xuất hiện cao nhất sẽ trở thành token được dự đoán tiếp theo; nếu đặt Top-k bằng 1, nhiệt độ và Top-p cũng sẽ trở nên không còn ý nghĩa, vì chỉ có một token vượt qua tiêu chuẩn Top-k, nó sẽ là token được dự đoán tiếp theo.



<strong>(2) Zero-shot, One-shot và Few-shot Prompting</strong>

Dựa trên số lượng ví dụ (Exemplar) chúng ta cung cấp cho mô hình, prompt có thể được chia thành ba loại. Để hiểu rõ hơn, hãy lấy một tác vụ phân loại cảm xúc (sentiment classification) làm ví dụ, mục tiêu là để mô hình phán đoán sắc thái cảm xúc của một đoạn văn bản (như tích cực, tiêu cực hoặc trung tính).

<strong>Zero-shot Prompting</strong> nghĩa là chúng ta không đưa cho mô hình bất kỳ ví dụ nào, trực tiếp để nó hoàn thành tác vụ dựa trên chỉ thị. Điều này nhờ vào khả năng tổng quát hóa mạnh mẽ mà mô hình có được sau khi tiền huấn luyện trên lượng dữ liệu khổng lồ.

Ví dụ: Chúng ta trực tiếp đưa ra chỉ thị cho mô hình, yêu cầu nó hoàn thành tác vụ phân loại cảm xúc.

```Python
文本:Datawhale的AI Agent课程非常棒！
情感:正面
```

<strong>One-shot Prompting</strong> Chúng ta cung cấp cho mô hình một ví dụ hoàn chỉnh, thể hiện cho nó định dạng tác vụ và phong cách đầu ra mong muốn.

Ví dụ: Chúng ta đưa trước cho mô hình một cặp "câu hỏi - câu trả lời" hoàn chỉnh làm mẫu, sau đó đưa ra câu hỏi mới của chúng ta.

```Python
文本:这家餐厅的服务太慢了。
情感:负面

文本:Datawhale的AI Agent课程非常棒！
情感:
```

Mô hình sẽ bắt chước định dạng ví dụ đã cho, hoàn thành đoạn văn bản thứ hai với kết quả "tích cực".

<strong>Few-shot Prompting</strong> Chúng ta cung cấp nhiều ví dụ, điều này giúp mô hình hiểu chính xác hơn chi tiết, ranh giới và những sắc thái tinh tế của tác vụ, từ đó đạt được hiệu năng tốt hơn.

Ví dụ: Chúng ta cung cấp nhiều ví dụ bao quát các tình huống khác nhau, để mô hình có hiểu biết toàn diện hơn về tác vụ.

```Python
文本:这家餐厅的服务太慢了。
情感:负面

文本:这部电影的情节很平淡。
情感:中性

文本:Datawhale的AI Agent课程非常棒！
情感:
```

Mô hình sẽ tổng hợp tất cả các ví dụ, phân loại chính xác hơn cảm xúc của câu cuối cùng là "tích cực".

<strong>(3) Ảnh hưởng của Instruction Tuning</strong>

Các mô hình GPT thời kỳ đầu (như GPT-3) chủ yếu là mô hình "hoàn thành văn bản" (text completion), chúng giỏi tiếp tục viết dựa trên văn bản phía trước, nhưng không nhất thiết hiểu và thực hiện tốt chỉ thị của con người.

<strong>Instruction Tuning</strong> là một kỹ thuật fine-tuning, nó sử dụng lượng lớn dữ liệu có định dạng "chỉ thị - câu trả lời" để tiếp tục huấn luyện mô hình đã được tiền huấn luyện. Sau khi instruction tuning, mô hình có thể hiểu và tuân theo chỉ thị của người dùng tốt hơn. Tất cả các mô hình chúng ta sử dụng hàng ngày trong công việc và học tập (như `ChatGPT`, `DeepSeek`, `Qwen`) đều là các mô hình đã qua instruction tuning trong dòng mô hình của chúng.

- <strong>Prompt đối với mô hình "hoàn thành văn bản" (bạn cần dùng few-shot prompting để "dạy" mô hình làm gì):</strong>

```Plain
这是一段将英文翻译成中文的程序。
英文:Hello
中文:你好
英文:How are you?
中文:
```

- <strong>Prompt đối với mô hình "đã instruction tuning" (bạn có thể trực tiếp đưa ra chỉ thị):</strong>

```Plain
请将下面的英文翻译成中文:
How are you?
```

Sự xuất hiện của instruction tuning đã đơn giản hóa rất nhiều cách chúng ta tương tác với mô hình, khiến các chỉ thị bằng ngôn ngữ tự nhiên trực tiếp, rõ ràng trở nên khả thi.

<strong>(4) Các kỹ thuật prompt cơ bản</strong>

<strong>Role-playing</strong> Bằng cách gán cho mô hình một vai trò cụ thể, chúng ta có thể dẫn dắt phong cách trả lời, giọng điệu và phạm vi tri thức của nó, khiến đầu ra của nó phù hợp hơn với yêu cầu của kịch bản cụ thể.

```Plain
# 案例
你现在是一位资深的Python编程专家。请解释一下Python中的GIL（全局解释器锁）是什么，要让一个初学者也能听懂。
```

<strong>In-context Example</strong> Điều này nhất quán với ý tưởng của few-shot prompting, thông qua việc cung cấp các ví dụ đầu vào-đầu ra rõ ràng trong prompt, để "dạy" mô hình cách xử lý yêu cầu của chúng ta, đặc biệt hữu ích khi xử lý các tác vụ có định dạng phức tạp hoặc phong cách đặc thù.

```Plain
# 案例
我需要你从产品评论中提取产品名称和用户情感。请严格按照下面的JSON格式输出。

评论:这款"星尘"笔记本电脑的屏幕显示效果惊人，但我不太喜欢它的键盘手感。
输出:{"product_name": "星尘笔记本电脑", "sentiment": "混合"}

评论:我刚买的"声动"耳机音质很棒，续航也超出了我的预期！
输出:
```

<strong>(5) Chain-of-Thought</strong>

Đối với các bài toán phức tạp cần suy luận logic, tính toán hoặc tư duy nhiều bước, việc để mô hình trực tiếp đưa ra câu trả lời thường dễ bị sai. <strong>Chain-of-Thought (CoT)</strong> là một kỹ thuật prompt mạnh mẽ, nó thông qua việc dẫn dắt mô hình "suy nghĩ từng bước một", nâng cao khả năng suy luận của mô hình trong các tác vụ phức tạp.

Chìa khóa để thực hiện CoT là thêm vào prompt một câu dẫn dắt đơn giản, như "hãy suy nghĩ từng bước" hoặc "Let's think step by step".

```Plain
# 思维链提示
一个篮球队在一个赛季的80场比赛中赢了60%。在接下来的赛季中，他们打了15场比赛，赢了12场。两个赛季的总胜率是多少？
请一步一步地思考并解答。

>>>
（模型可能会输出）
好的，我们来一步步计算。
第一步:计算第一个赛季赢得的比赛数。
80场 * 60% = 48场。
第二步:计算两个赛季的总比赛数和总胜利数。
总比赛数 = 80 + 15 = 95场。
总胜利数 = 48 + 12 = 60场。
第三步:计算总胜率。
总胜率 = (总胜利数 / 总比赛数) * 100% = (60 / 95) * 100% ≈ 63.16%。
所以，两个赛季的总胜率约为63.16%。
```

Thông qua việc trình bày tường minh quá trình suy luận của nó, mô hình không chỉ dễ dàng đưa ra câu trả lời đúng hơn, mà còn khiến câu trả lời của nó trở nên đáng tin cậy hơn, dễ dàng cho chúng ta kiểm tra và sửa lỗi hơn.

### 3.2.2 Text Tokenization

Chúng ta biết rằng, máy tính về bản chất chỉ có thể hiểu con số. Do đó, trước khi đưa văn bản ngôn ngữ tự nhiên vào Large Language Model, cần phải chuyển đổi nó thành định dạng số mà mô hình có thể xử lý. Quá trình chuyển đổi chuỗi văn bản thành chuỗi số này được gọi là <strong>Tokenization (phân tách từ/token hóa)</strong>. Tác dụng của <strong>Tokenizer</strong> là định nghĩa một bộ quy tắc, chia văn bản gốc thành các đơn vị nhỏ nhất, chúng ta gọi đó là <strong>Token</strong>.

<strong>3.2.2.1 Vì sao cần Tokenization</strong>

Các tác vụ xử lý ngôn ngữ tự nhiên thời kỳ đầu có thể áp dụng các chiến lược tokenization đơn giản:

- <strong>Tokenization theo từ (Word-based)</strong>: trực tiếp dùng khoảng trắng hoặc dấu câu để chia câu thành các từ. Phương pháp này rất trực quan, nhưng cũng đối mặt với thách thức:
  - Bùng nổ bộ từ vựng và từ chưa biết: lượng từ vựng của một ngôn ngữ là rất lớn, nếu mỗi từ đều là một token độc lập, bộ từ vựng sẽ trở nên khó quản lý. Tệ hơn, mô hình sẽ không thể xử lý bất kỳ từ nào chưa xuất hiện trong bộ từ vựng (ví dụ "DatawhaleAgent"), hiện tượng này được gọi là "từ chưa biết" (Out-Of-Vocabulary, OOV).
  - Thiếu liên kết ngữ nghĩa: mô hình khó nắm bắt quan hệ ngữ nghĩa giữa các từ có hình thái gần nhau. Ví dụ, "look", "looks" và "looking" sẽ bị coi là ba token hoàn toàn khác nhau, dù chúng có ý nghĩa cốt lõi chung. Tương tự, các từ hiếm trong dữ liệu huấn luyện do xuất hiện ít lần nên ngữ nghĩa của chúng cũng khó được mô hình học đầy đủ.
- <strong>Tokenization theo ký tự (Character-based)</strong>: chia văn bản thành các ký tự đơn lẻ. Phương pháp này có bộ từ vựng rất nhỏ (ví dụ chữ cái tiếng Anh, chữ số và dấu câu), không tồn tại vấn đề OOV. Nhưng nhược điểm của nó là, hầu hết các ký tự đơn lẻ không mang ngữ nghĩa độc lập, mô hình cần tốn nhiều công sức hơn để học cách kết hợp các ký tự thành từ có ý nghĩa, dẫn đến hiệu suất học tập thấp.

Để cân bằng giữa kích thước bộ từ vựng và khả năng biểu đạt ngữ nghĩa, các Large Language Model hiện đại phổ biến áp dụng thuật toán <strong>Subword Tokenization</strong>. Ý tưởng cốt lõi của nó là: giữ nguyên các từ phổ biến (như "agent") thành một token hoàn chỉnh, đồng thời tách các từ ít phổ biến (như "Tokenization") thành nhiều đoạn subword có ý nghĩa (như "Token" và "ization"). Như vậy vừa kiểm soát được kích thước bộ từ vựng, vừa cho phép mô hình hiểu và sinh ra từ mới thông qua việc kết hợp các subword.

<strong>3.2.2.2 Phân tích thuật toán Byte-Pair Encoding</strong>

Byte-Pair Encoding (BPE) là một trong những thuật toán subword tokenization chủ đạo nhất<sup>[6]</sup>, dòng mô hình GPT đã áp dụng thuật toán này. Ý tưởng cốt lõi của nó rất ngắn gọn, có thể hiểu là một quá trình hợp nhất "tham lam" (greedy):

1. <strong>Khởi tạo</strong>: khởi tạo bộ từ vựng bằng tất cả các ký tự cơ bản xuất hiện trong kho ngữ liệu.
2. <strong>Hợp nhất lặp</strong>: trên kho ngữ liệu, thống kê tần suất xuất hiện của tất cả các cặp token liền kề, tìm ra cặp có tần suất cao nhất, hợp nhất chúng thành một token mới, và thêm vào bộ từ vựng.
3. <strong>Lặp lại</strong>: lặp lại bước 2, cho đến khi kích thước bộ từ vựng đạt ngưỡng đặt trước.

<strong>Minh họa ví dụ:</strong> Giả sử kho ngữ liệu mini của chúng ta là `{"hug": 1, "pug": 1, "pun": 1, "bun": 1}`, và chúng ta muốn xây dựng một bộ từ vựng có kích thước 10. Quá trình huấn luyện BPE có thể được biểu diễn bằng Bảng 3.1 dưới đây:

<div align="center">
  <p>Bảng 3.1 Ví dụ quá trình hợp nhất của thuật toán BPE</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-5.png" alt="图片描述" width="90%"/>
</div>

Sau khi huấn luyện kết thúc, kích thước bộ từ vựng đạt 10, chúng ta thu được quy tắc tokenization mới. Bây giờ, đối với một từ chưa từng thấy "bug", tokenizer sẽ trước tiên tìm xem "bug" có trong bộ từ vựng không, phát hiện không có; sau đó tìm "bu", phát hiện không có; cuối cùng tìm "b" và "ug", phát hiện cả hai đều có, vậy nó sẽ tách thành `['b', 'ug']`.

Dưới đây chúng ta dùng một đoạn code Python đơn giản để mô phỏng quá trình trên:

```Python
import re, collections

def get_stats(vocab):
    """Thống kê tần suất cặp token"""
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    """Hợp nhất cặp token"""
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

# Chuẩn bị kho ngữ liệu, mỗi từ thêm </w> ở cuối để biểu thị kết thúc, và đã được tách sẵn thành ký tự
vocab = {'h u g </w>': 1, 'p u g </w>': 1, 'p u n </w>': 1, 'b u n </w>': 1}
num_merges = 4 # Đặt số lần hợp nhất

for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    best = max(pairs, key=pairs.get)
    vocab = merge_vocab(best, vocab)
    print(f"第{i+1}次合并: {best} -> {''.join(best)}")
    print(f"新词表（部分）: {list(vocab.keys())}")
    print("-" * 20)

>>>
第1次合并: ('u', 'g') -> ug
新词表（部分）: ['h ug </w>', 'p ug </w>', 'p u n </w>', 'b u n </w>']
--------------------
第2次合并: ('ug', '</w>') -> ug</w>
新词表（部分）: ['h ug</w>', 'p ug</w>', 'p u n </w>', 'b u n </w>']
--------------------
第3次合并: ('u', 'n') -> un
新词表（部分）: ['h ug</w>', 'p ug</w>', 'p un </w>', 'b un </w>']
--------------------
第4次合并: ('un', '</w>') -> un</w>
新词表（部分）: ['h ug</w>', 'p ug</w>', 'p un</w>', 'b un</w>']
--------------------
```

Đoạn code này thể hiện rõ ràng cách thuật toán BPE, thông qua việc lặp đi lặp lại hợp nhất cặp token liền kề có tần suất cao nhất, dần dần xây dựng và mở rộng bộ từ vựng.

Nhiều thuật toán sau này đều được tối ưu hóa dựa trên nền tảng BPE. Trong đó, WordPiece và SentencePiece do Google phát triển là hai thuật toán có ảnh hưởng lớn nhất.

- <strong>WordPiece</strong>: thuật toán được mô hình BERT của Google áp dụng<sup>[7]</sup>. Nó rất giống BPE, nhưng tiêu chuẩn hợp nhất token không phải "tần suất cao nhất", mà là "khả năng tối đa hóa xác suất mô hình ngôn ngữ của kho ngữ liệu". Nói đơn giản, nó sẽ ưu tiên hợp nhất các cặp token giúp "độ trôi chảy" của toàn bộ kho ngữ liệu tăng lên nhiều nhất.
- <strong>SentencePiece</strong>: một công cụ tokenization mã nguồn mở của Google<sup>[8]</sup>, dòng mô hình Llama đã áp dụng thuật toán này. Đặc điểm lớn nhất của nó là coi khoảng trắng cũng là một ký tự bình thường (thường được biểu thị bằng dấu gạch dưới `_`). Điều này khiến quá trình tokenization và giải mã hoàn toàn có thể đảo ngược, và không phụ thuộc vào ngôn ngữ cụ thể (ví dụ, nó không cần biết tiếng Trung không dùng khoảng trắng để tách từ).

<strong>3.2.2.3 Ý nghĩa của Tokenizer đối với nhà phát triển</strong>

Hiểu chi tiết thuật toán tokenization không phải là mục đích, nhưng với tư cách là nhà phát triển Agent, việc hiểu ảnh hưởng thực tế của tokenizer rất quan trọng, điều này liên quan trực tiếp đến hiệu năng, chi phí và độ ổn định của Agent:

- <strong>Giới hạn context window</strong>: context window của mô hình (như 8K, 128K) được tính bằng <strong>số lượng Token</strong>, chứ không phải số ký tự hay số từ. Cùng một đoạn văn, ở các ngôn ngữ khác nhau (như tiếng Trung và tiếng Anh) hoặc các tokenizer khác nhau, số lượng Token có thể chênh lệch rất lớn. Quản lý chính xác độ dài đầu vào, tránh vượt quá giới hạn context là nền tảng để xây dựng Agent có bộ nhớ dài hạn.
- <strong>Chi phí API</strong>: hầu hết các API mô hình đều tính phí theo số lượng Token. Hiểu được văn bản của bạn sẽ được tokenize như thế nào là bước then chốt để ước tính và kiểm soát chi phí vận hành Agent.
- <strong>Các hiện tượng bất thường trong hiệu năng mô hình</strong>: đôi khi nguyên nhân của những biểu hiện kỳ lạ của mô hình nằm ở tokenization. Ví dụ, mô hình có thể rất giỏi tính `2 + 2`, nhưng với `2+2` (không có khoảng trắng) lại có thể bị sai, vì cái sau có thể bị tokenizer coi là một token độc lập, hiếm gặp. Tương tự, một từ do viết hoa/viết thường chữ cái đầu khác nhau, cũng có thể bị tách thành chuỗi Token hoàn toàn khác, từ đó ảnh hưởng đến khả năng hiểu của mô hình. Khi thiết kế prompt và phân tích đầu ra của mô hình, cân nhắc đến những "cạm bẫy" này giúp nâng cao độ bền vững (robustness) của Agent.

### 3.2.3 Gọi Large Language Model mã nguồn mở

Ở chương đầu tiên của cuốn sách, chúng ta đã thông qua API để tương tác với Large Language Model, nhằm thúc đẩy Agent của chúng ta hoạt động. Đây là một cách nhanh chóng, tiện lợi, nhưng không phải là cách duy nhất. Đối với nhiều kịch bản cần xử lý dữ liệu nhạy cảm, muốn chạy offline hoặc muốn kiểm soát chi phí một cách tinh tế, việc triển khai Large Language Model trực tiếp cục bộ (local) trở nên vô cùng quan trọng.

<strong>Hugging Face Transformers</strong> là một thư viện mã nguồn mở mạnh mẽ, nó cung cấp giao diện chuẩn hóa để tải và sử dụng hàng vạn mô hình đã được tiền huấn luyện. Chúng ta sẽ dùng nó để hoàn thành thực hành lần này.

<strong>Cấu hình môi trường và lựa chọn mô hình</strong>: để đa số độc giả đều có thể chạy thuận lợi trên máy tính cá nhân, chúng ta đặc biệt chọn một mô hình quy mô nhỏ nhưng mạnh mẽ: `Qwen/Qwen1.5-0.5B-Chat`. Đây là một mô hình đối thoại mã nguồn mở có khoảng 500 triệu tham số do Alibaba DAMO Academy phát triển, nó có kích thước nhỏ, hiệu năng xuất sắc, rất phù hợp cho việc học nhập môn và triển khai cục bộ.

Trước tiên, hãy đảm bảo bạn đã cài đặt các thư viện cần thiết:

```Plain
pip install transformers torch
```

Trong thư viện `transformers`, chúng ta thường dùng hai class `AutoModelForCausalLM` và `AutoTokenizer` để tự động tải trọng số và tokenizer khớp với mô hình. Đoạn code dưới đây sẽ tự động tải về các tệp mô hình và cấu hình tokenizer cần thiết từ Hugging Face Hub, quá trình này có thể mất một chút thời gian, tùy thuộc vào tốc độ mạng của bạn.

```Python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

# Chỉ định model ID
model_id = "Qwen/Qwen1.5-0.5B-Chat"

# Thiết lập device, ưu tiên dùng GPU
device = "cuda" if torch.cuda.is_available() else "cpu"
print(f"Using device: {device}")

# Tải tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_id)

# Tải mô hình, và chuyển nó đến device đã chỉ định
model = AutoModelForCausalLM.from_pretrained(model_id).to(device)

print("模型和分词器加载完成！")
```

Chúng ta cùng tạo một prompt đối thoại, mô hình Qwen1.5-Chat tuân theo một template đối thoại đặc thù. Sau đó, có thể dùng `tokenizer` đã tải ở bước trên để chuyển đổi prompt văn bản thành ID số (tức Token ID) mà mô hình có thể hiểu.

```Python
# Chuẩn bị đầu vào đối thoại
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "你好，请介绍你自己。"}
]

# Dùng template của tokenizer để định dạng đầu vào
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# Mã hóa (encode) văn bản đầu vào
model_inputs = tokenizer([text], return_tensors="pt").to(device)

print("编码后的输入文本:")
print(model_inputs)

>>>
{'input_ids': tensor([[151644, 8948, 198, 2610, 525, 264,  10950, 17847, 13,151645, 198, 151644, 872, 198, 108386, 37945, 100157, 107828,1773, 151645, 198, 151644, 77091, 198]], device='cuda:0'), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]],
       device='cuda:0')}
```

Bây giờ có thể gọi phương thức `generate()` của mô hình để sinh ra câu trả lời. Mô hình sẽ xuất ra một chuỗi Token ID, đại diện cho câu trả lời của nó.

Cuối cùng, chúng ta cần dùng phương thức `decode()` của tokenizer, để dịch các ID số này trở lại thành văn bản mà con người có thể đọc được.

```Python
# Dùng mô hình để sinh câu trả lời
# max_new_tokens kiểm soát số lượng Token mới tối đa mà mô hình có thể sinh ra
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)

# Cắt bỏ phần đầu vào khỏi Token ID đã sinh ra
# Như vậy chúng ta chỉ giải mã phần mới mà mô hình sinh ra
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

# Giải mã Token ID đã sinh ra
response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]

print("\n模型的回答:")
print(response)

>>>
我叫通义千问，是由阿里云研发的预训练语言模型，可以回答问题、创作文字，还能表达观点、撰写代码。我主要的功能是在多个领域提
供帮助，包括但不限于:语言理解、文本生成、机器翻译、问答系统等。有什么我可以帮到你的吗？
```

Sau khi bạn chạy xong tất cả code, bạn sẽ thấy trên máy tính cục bộ của mình phần giới thiệu về mô hình Qwen do mô hình sinh ra. Chúc mừng bạn, bạn đã triển khai và chạy thành công một Large Language Model mã nguồn mở trên máy cục bộ!

### 3.2.4 Lựa chọn mô hình

Ở phần trước, chúng ta đã chạy thành công một mô hình ngôn ngữ mã nguồn mở quy mô nhỏ trên máy cục bộ. Điều này tự nhiên dẫn đến một câu hỏi then chốt đối với nhà phát triển Agent: trong bối cảnh hàng trăm mô hình đang nở rộ hiện nay, chúng ta nên chọn mô hình phù hợp nhất cho một tác vụ cụ thể như thế nào?

Việc lựa chọn mô hình ngôn ngữ không đơn thuần là theo đuổi "lớn nhất, mạnh nhất", mà là một quá trình ra quyết định cân nhắc giữa hiệu năng, chi phí, tốc độ và phương thức triển khai. Phần này trước tiên sẽ điểm qua một số yếu tố cân nhắc then chốt khi lựa chọn mô hình, sau đó điểm qua các mô hình closed-source và open-source chủ đạo hiện nay.

Do công nghệ Large Language Model đang trong giai đoạn phát triển với tốc độ cao, các mô hình mới, phiên bản mới xuất hiện liên tục, tốc độ lặp lại cực nhanh. Phần này khi viết cố gắng cung cấp một cái nhìn tổng quan về các mô hình chủ đạo hiện nay và các cân nhắc lựa chọn, nhưng xin độc giả lưu ý rằng, các phiên bản mô hình cụ thể và dữ liệu hiệu năng được đề cập trong bài có thể thay đổi theo thời gian, và chỉ liệt kê một phần công việc, không đầy đủ. Chúng tôi chú trọng hơn vào việc giới thiệu các đặc điểm công nghệ cốt lõi, xu hướng phát triển cũng như các nguyên tắc lựa chọn chung trong phát triển Agent.

<strong>3.2.4.1 Các cân nhắc then chốt khi lựa chọn mô hình</strong>

Khi lựa chọn Large Language Model cho Agent của bạn, có thể đánh giá tổng hợp từ các khía cạnh sau:

- <strong>Hiệu năng và năng lực</strong>: đây là cân nhắc cốt lõi nhất. Các mô hình khác nhau giỏi ở các tác vụ khác nhau, có mô hình giỏi suy luận logic và sinh code, có mô hình lại vượt trội hơn trong sáng tác sáng tạo hoặc dịch đa ngôn ngữ. Bạn có thể tham khảo một số bảng xếp hạng benchmark công khai (như LMSys Chatbot Arena Leaderboard) để đánh giá năng lực tổng hợp của mô hình.
- <strong>Chi phí</strong>: đối với mô hình closed-source, chi phí chủ yếu thể hiện ở phí gọi API, thường tính theo số lượng Token. Đối với mô hình open-source, chi phí thể hiện ở phần cứng cần thiết để triển khai cục bộ (GPU, bộ nhớ) và vận hành. Cần lựa chọn dựa trên lượng sử dụng dự kiến và ngân sách của ứng dụng.
- <strong>Tốc độ (độ trễ)</strong>: đối với Agent cần tương tác thời gian thực (như chăm sóc khách hàng, NPC trong game), tốc độ phản hồi của mô hình vô cùng quan trọng. Một số mô hình nhẹ hoặc đã được tối ưu hóa (như GPT-3.5 Turbo, Claude 3.5 Sonnet) có hiệu năng về độ trễ tốt hơn.
- <strong>Context window</strong>: giới hạn số lượng Token tối đa mà mô hình có thể xử lý trong một lần. Đối với Agent cần hiểu tài liệu dài, phân tích codebase hoặc duy trì ký ức đối thoại dài hạn, việc lựa chọn một mô hình có context window lớn hơn (như 128K Token trở lên) là cần thiết.
- <strong>Phương thức triển khai</strong>: dùng API là cách đơn giản, tiện lợi nhất, nhưng dữ liệu cần gửi cho bên thứ ba, và bị giới hạn bởi điều khoản của nhà cung cấp dịch vụ. Triển khai cục bộ có thể đảm bảo quyền riêng tư dữ liệu và mức độ tự chủ cao nhất, nhưng yêu cầu cao hơn về công nghệ và phần cứng.
- <strong>Hệ sinh thái và chuỗi công cụ</strong>: mức độ phổ biến của một mô hình cũng quyết định độ trưởng thành của hệ sinh thái xung quanh nó. Các mô hình chủ đạo thường có cộng đồng hỗ trợ phong phú hơn, tutorial, mô hình đã tiền huấn luyện, công cụ fine-tuning và framework phát triển tương thích (như LangChain, LlamaIndex, Hugging Face Transformers), điều này có thể tăng tốc đáng kể tiến trình phát triển, giảm độ khó phát triển. Lựa chọn một mô hình có cộng đồng năng động và chuỗi công cụ hoàn thiện, có thể dễ dàng tìm thấy giải pháp và tài nguyên hơn khi gặp vấn đề.
- <strong>Khả năng fine-tuning và tùy chỉnh</strong>: đối với Agent cần xử lý dữ liệu lĩnh vực cụ thể hoặc thực hiện tác vụ cụ thể, khả năng fine-tuning của mô hình vô cùng quan trọng. Một số mô hình cung cấp giao diện và công cụ fine-tuning tiện lợi, cho phép nhà phát triển dùng dữ liệu của riêng mình để huấn luyện tùy chỉnh mô hình, từ đó nâng cao đáng kể hiệu năng và độ chính xác của mô hình trong kịch bản cụ thể. Mô hình open-source thường cung cấp sự linh hoạt lớn hơn ở khía cạnh này.
- <strong>Tính an toàn và đạo đức</strong>: cùng với việc ứng dụng rộng rãi của Large Language Model, các rủi ro an toàn và vấn đề đạo đức tiềm ẩn của nó cũng ngày càng nổi bật. Khi lựa chọn mô hình, cần xem xét biểu hiện của nó ở các khía cạnh như thiên kiến (bias), độc hại (toxicity), ảo giác (hallucination), cũng như sự đầu tư của nhà cung cấp dịch vụ hoặc cộng đồng open-source vào an toàn mô hình và AI có trách nhiệm. Đối với các ứng dụng hướng đến công chúng hoặc liên quan đến thông tin nhạy cảm, tính an toàn và tuân thủ đạo đức của mô hình là cân nhắc không thể bỏ qua.

<strong>3.2.4.2 Tổng quan các mô hình closed-source</strong>

Mô hình closed-source thường đại diện cho tiền tuyến công nghệ AI hiện nay, và cung cấp dịch vụ API ổn định, dễ sử dụng, là lựa chọn hàng đầu để xây dựng Agent hiệu năng cao.

1. <strong>Dòng OpenAI GPT</strong>: từ GPT-3 mở ra kỷ nguyên mô hình lớn, đến ChatGPT đưa vào RLHF (Reinforcement Learning from Human Feedback), đạt được sự thống nhất với ý định con người, rồi đến GPT-4 mở ra kỷ nguyên đa phương thức (multimodal), OpenAI liên tục dẫn đầu sự phát triển của ngành. GPT-5 mới nhất càng nâng cao khả năng đa phương thức và mức độ trí tuệ tổng quát lên tầm cao mới, có thể xử lý liền mạch đầu vào văn bản, âm thanh và hình ảnh, và sinh ra đầu ra tương ứng, tốc độ phản hồi và tính tự nhiên của nó cũng được nâng cao đáng kể, đặc biệt thể hiện xuất sắc trong đối thoại thoại thời gian thực.
2. <strong>Dòng Google Gemini</strong>: dòng mô hình Gemini do Google DeepMind ra mắt là đại diện cho đa phương thức nguyên sinh (native multimodal), đặc điểm cốt lõi của nó là có thể xử lý thống nhất dữ liệu đa phương thức như văn bản, code, âm thanh/video và hình ảnh, và có lợi thế trong xử lý khối lượng thông tin khổng lồ nhờ context window siêu dài. Gemini Ultra là mô hình mạnh nhất của nó, phù hợp với các tác vụ cực kỳ phức tạp; Gemini Pro phù hợp với phạm vi tác vụ rộng, cung cấp hiệu năng và hiệu suất cao; Gemini Nano được tối ưu hóa cho triển khai trên thiết bị (on-device). Dòng mô hình Gemini 2.5 mới nhất, như Gemini 2.5 Pro và Gemini 2.5 Flash, càng nâng cao khả năng suy luận và context window, đặc biệt Gemini 2.5 Flash với tốc độ suy luận nhanh hơn và hiệu quả chi phí, phù hợp với các kịch bản cần phản hồi nhanh.
3. <strong>Dòng Anthropic Claude</strong>: Anthropic là một công ty tập trung vào an toàn AI và AI có trách nhiệm, dòng mô hình Claude của họ đặt an toàn AI lên hàng đầu ngay từ khi thiết kế, nổi tiếng với độ tin cậy trong xử lý tài liệu dài, giảm đầu ra có hại, tuân theo chỉ thị, được các ứng dụng cấp doanh nghiệp ưa chuộng. Dòng Claude 3 bao gồm Claude 3 Opus (thông minh nhất, hiệu năng mạnh nhất), Claude 3 Sonnet (lựa chọn cân bằng giữa hiệu năng và tốc độ) và Claude 3 Haiku (mô hình nhanh nhất, gọn nhẹ nhất, phù hợp cho tương tác gần thời gian thực). Dòng mô hình Claude 4 mới nhất, như Claude 4 Opus, đã đạt được tiến bộ đáng kể về trí tuệ tổng quát, suy luận phức tạp và sinh code, càng nâng cao khả năng xử lý context dài và tác vụ đa phương thức.
4. <strong>Các mô hình chủ đạo trong nước (Trung Quốc)</strong>: Trung Quốc đã xuất hiện nhiều mô hình closed-source có tính cạnh tranh trong lĩnh vực Large Language Model, đại diện bởi các mô hình nội địa như Baidu Ernie Bot (文心一言), Tencent Hunyuan (混元), Huawei Pangu-α (盘古), iFlytek SparkDesk (讯飞星火) và Moonshot AI (月之暗面), có lợi thế tự nhiên trong xử lý tiếng Trung, và hỗ trợ sâu cho các ngành công nghiệp nội địa.

<strong>3.2.4.3 Tổng quan các mô hình open-source</strong>

Mô hình open-source cung cấp cho nhà phát triển mức độ linh hoạt, minh bạch và tự chủ cao nhất, tạo ra một hệ sinh thái cộng đồng thịnh vượng. Chúng cho phép nhà phát triển triển khai cục bộ, thực hiện fine-tuning tùy chỉnh, và nắm hoàn toàn quyền kiểm soát mô hình.

- <strong>Dòng Meta Llama</strong>: dòng Llama do Meta ra mắt là một cột mốc quan trọng của Large Language Model mã nguồn mở. Dòng này nhờ hiệu năng tổng hợp xuất sắc, giấy phép mở và hỗ trợ cộng đồng mạnh mẽ, đã trở thành nền tảng của nhiều dự án phái sinh và nghiên cứu. Dòng Llama 4 được phát hành vào tháng 4 năm 2025, là lô mô hình đầu tiên của Meta áp dụng kiến trúc Mixture-of-Experts (MoE), kiến trúc này nâng cao đáng kể hiệu suất tính toán bằng cách chỉ kích hoạt phần mô hình cần thiết để xử lý tác vụ cụ thể. Dòng này gồm ba mô hình có định vị rõ ràng: Llama 4 Scout hỗ trợ context window 10 triệu token, được thiết kế chuyên biệt cho phân tích tài liệu dài và triển khai trên di động. Llama 4 Maverick tập trung vào khả năng đa phương thức, thể hiện xuất sắc ở khả năng viết code, suy luận phức tạp và hỗ trợ đa ngôn ngữ. Llama 4 Behemoth vượt qua các đối thủ cạnh tranh ở nhiều benchmark STEM, là mô hình mạnh nhất hiện nay của Meta.
- <strong>Dòng Mistral AI</strong>: Mistral AI đến từ Pháp nổi tiếng với thiết kế mô hình "kích thước nhỏ, hiệu năng cao". Mô hình mới nhất của họ, Mistral Medium 3.1, được phát hành vào tháng 8 năm 2025, có độ chính xác và tốc độ phản hồi được cải thiện đáng kể trong các tác vụ như sinh code, suy luận STEM và hỏi đáp liên ngành, hiệu năng benchmark vượt trội hơn các mô hình cùng hạng như Claude Sonnet 3.7 và Llama 4 Maverick. Nó có khả năng đa phương thức nguyên sinh, có thể đồng thời xử lý đầu vào hỗn hợp hình ảnh và văn bản, đồng thời được tích hợp sẵn "lớp thích ứng giọng điệu" (tone adaptation layer), giúp doanh nghiệp dễ dàng hơn trong việc tạo ra đầu ra phù hợp với phong cách thương hiệu.
- <strong>Lực lượng open-source trong nước (Trung Quốc)</strong>: các nhà sản xuất và tổ chức nghiên cứu trong nước cũng đang tích cực đón nhận open-source, ví dụ dòng <strong>Qwen (通义千问)</strong> của Alibaba và dòng <strong>ChatGLM</strong> do Đại học Thanh Hoa hợp tác với Zhipu AI phát triển, chúng cung cấp khả năng xử lý tiếng Trung mạnh mẽ, và xây dựng được cộng đồng năng động xung quanh mình.

Đối với nhà phát triển Agent, mô hình closed-source cung cấp sự tiện lợi "dùng ngay, không cần chỉnh sửa" (out-of-the-box), còn mô hình open-source lại mang đến cho chúng ta sự tự do tùy chỉnh "theo ý muốn". Hiểu được đặc điểm và các mô hình đại diện của hai phe phái này là bước đầu tiên để đưa ra lựa chọn công nghệ sáng suốt cho dự án Agent của chúng ta.

## 3.3 Scaling Laws và hạn chế của Large Language Model

Large Language Model (LLM) đã đạt được những tiến bộ đáng kinh ngạc trong những năm gần đây, ranh giới năng lực của nó không ngừng được mở rộng, các kịch bản ứng dụng ngày càng phong phú. Tuy nhiên, đằng sau những thành tựu này, không thể thiếu sự hiểu biết sâu sắc về mối quan hệ giữa quy mô mô hình, lượng dữ liệu và tài nguyên tính toán, tức <strong>Scaling Laws</strong>. Đồng thời, là một công nghệ mới nổi, LLM cũng đối mặt với nhiều thách thức và hạn chế. Phần này sẽ đi sâu khám phá các khái niệm cốt lõi này, nhằm giúp độc giả hiểu toàn diện ranh giới năng lực của LLM, từ đó phát huy điểm mạnh, tránh điểm yếu khi xây dựng Agent.

### 3.3.1 Scaling Laws

<strong>Scaling Laws</strong> là một trong những phát hiện quan trọng nhất trong lĩnh vực Large Language Model những năm gần đây. Nó tiết lộ rằng hiệu năng mô hình có mối quan hệ lũy thừa (power-law) có thể dự đoán được với số lượng tham số mô hình, lượng dữ liệu huấn luyện và tài nguyên tính toán. Phát hiện này cung cấp định hướng lý thuyết cho sự phát triển liên tục của Large Language Model, làm rõ logic nền tảng rằng việc tăng đầu tư tài nguyên có thể nâng cao hiệu năng mô hình một cách có hệ thống.

Nghiên cứu phát hiện, trong hệ tọa độ log-log, hiệu năng mô hình (thường được đo bằng Loss) thể hiện mối quan hệ lũy thừa mượt mà với cả ba yếu tố: số lượng tham số, lượng dữ liệu và lượng tính toán<sup>[9]</sup>. Nói đơn giản, chỉ cần chúng ta liên tục tăng ba yếu tố này theo tỷ lệ, hiệu năng mô hình sẽ được nâng cao một cách có thể dự đoán được, mượt mà, mà không xuất hiện nút thắt cổ chai rõ rệt. Phát hiện này cung cấp định hướng rõ ràng cho việc thiết kế và huấn luyện mô hình lớn: trong phạm vi tài nguyên cho phép, mở rộng quy mô mô hình và lượng dữ liệu huấn luyện càng nhiều càng tốt.

Các nghiên cứu ban đầu chú trọng hơn vào việc tăng số lượng tham số mô hình, nhưng "định luật Chinchilla" do DeepMind đề xuất năm 2022 đã có một sửa đổi quan trọng đối với điều này<sup>[10]</sup>. Định luật này chỉ ra rằng, trong một ngân sách tính toán cho trước, để đạt được hiệu năng tối ưu, <strong>tồn tại một tỷ lệ tối ưu giữa số lượng tham số mô hình và lượng dữ liệu huấn luyện</strong>. Cụ thể, mô hình tối ưu nên nhỏ hơn so với quan niệm phổ biến trước đây, nhưng cần được huấn luyện với lượng dữ liệu nhiều hơn rất nhiều. Ví dụ, một mô hình Chinchilla với 70 tỷ tham số, do được huấn luyện với lượng dữ liệu gấp 4 lần GPT-3 (175 tỷ tham số), hiệu năng của nó lại vượt qua GPT-3. Phát hiện này đã sửa chữa nhận thức phiến diện "càng lớn càng tốt", nhấn mạnh tầm quan trọng của hiệu quả dữ liệu, và định hướng cho việc thiết kế nhiều mô hình lớn hiệu quả sau này (như dòng Llama).

Sản phẩm đáng kinh ngạc nhất của Scaling Laws là "sự xuất hiện (emergence) của năng lực". Cái gọi là sự xuất hiện năng lực, nghĩa là khi quy mô mô hình đạt đến một ngưỡng nhất định, nó sẽ đột ngột thể hiện các năng lực hoàn toàn mới mà hoàn toàn không tồn tại hoặc thể hiện kém ở các mô hình quy mô nhỏ. Ví dụ, các năng lực như <strong>Chain-of-Thought</strong>, <strong>Instruction Following</strong>, suy luận đa bước, sinh code, đều chỉ xuất hiện rõ rệt sau khi số lượng tham số mô hình đạt đến mức hàng chục tỷ thậm chí hàng trăm tỷ. Hiện tượng này cho thấy, Large Language Model không chỉ đơn thuần là ghi nhớ và lặp lại, mà chúng có thể đã hình thành một năng lực trừu tượng hóa và suy luận sâu sắc hơn trong quá trình học. Đối với nhà phát triển Agent, sự xuất hiện của năng lực có nghĩa là việc lựa chọn một mô hình có quy mô đủ lớn là tiền đề để đạt được khả năng ra quyết định và lập kế hoạch tự chủ phức tạp.

### 3.3.2 Ảo giác mô hình

<strong>Ảo giác mô hình (Hallucination)</strong> thường chỉ việc nội dung do Large Language Model sinh ra mâu thuẫn với thực tế khách quan, đầu vào của người dùng hoặc thông tin ngữ cảnh, hoặc sinh ra các sự thật, thực thể hay sự kiện không tồn tại. Bản chất của ảo giác là mô hình "bịa đặt" thông tin một cách quá tự tin trong quá trình sinh, thay vì truy xuất hoặc suy luận chính xác. Dựa trên hình thức biểu hiện, ảo giác có thể được chia thành nhiều loại<sup>[11]</sup>, ví dụ:

- <strong>Ảo giác thực tế (Factual Hallucinations)</strong>: mô hình sinh ra thông tin không khớp với sự thật trong thế giới thực.
- <strong>Ảo giác về tính trung thực (Faithfulness Hallucinations)</strong>: trong các tác vụ như tóm tắt văn bản, dịch thuật, nội dung sinh ra không phản ánh trung thực ý nghĩa của văn bản nguồn.
- <strong>Ảo giác nội tại (Intrinsic Hallucinations)</strong>: nội dung mô hình sinh ra mâu thuẫn trực tiếp với thông tin đầu vào.

Sự xuất hiện của ảo giác là kết quả của nhiều yếu tố cùng tác động. Trước tiên, dữ liệu huấn luyện có thể chứa thông tin sai lệch hoặc mâu thuẫn. Tiếp theo, cơ chế sinh tự hồi quy của mô hình quyết định rằng nó chỉ đang dự đoán token có khả năng xuất hiện cao nhất tiếp theo, mà không có module kiểm tra sự thật tích hợp sẵn. Cuối cùng, khi đối mặt với tác vụ cần suy luận phức tạp, mô hình có thể mắc lỗi trong chuỗi logic, từ đó "bịa đặt" ra kết luận sai. Ví dụ: một Agent lập kế hoạch du lịch, có thể đề xuất cho bạn một địa điểm du lịch không tồn tại trong thực tế, hoặc đặt một vé máy bay với số hiệu chuyến bay sai.

Ngoài ra, Large Language Model còn đối mặt với thách thức như tính thời sự của tri thức không đủ và thiên kiến tồn tại trong dữ liệu huấn luyện. Năng lực của Large Language Model đến từ dữ liệu huấn luyện của nó. Điều này có nghĩa là tri thức mà mô hình nắm giữ là tài liệu mới nhất tại thời điểm thu thập dữ liệu huấn luyện. Đối với các sự kiện xảy ra sau thời điểm này, các khái niệm mới xuất hiện hoặc sự thật mới nhất, mô hình sẽ không thể cảm nhận hoặc trả lời chính xác. Đồng thời, dữ liệu huấn luyện thường chứa đựng nhiều thiên kiến và định kiến khác nhau của xã hội loài người. Khi mô hình học trên dữ liệu này, nó không thể tránh khỏi việc hấp thụ và phản ánh những thiên kiến này<sup>[12]</sup>.

Để nâng cao độ tin cậy của Large Language Model, các nhà nghiên cứu và nhà phát triển đang tích cực khám phá nhiều phương pháp phát hiện và giảm thiểu ảo giác:

1. <strong>Ở cấp độ dữ liệu</strong>: thông qua làm sạch dữ liệu chất lượng cao, đưa vào tri thức thực tế cũng như Reinforcement Learning from Human Feedback (RLHF)<sup>[13]</sup>, giảm ảo giác từ gốc.
2. <strong>Ở cấp độ mô hình</strong>: khám phá kiến trúc mô hình mới, hoặc để mô hình có thể biểu đạt mức độ không chắc chắn của nó đối với nội dung sinh ra.
3. <strong>Ở cấp độ suy luận và sinh</strong>:
   1. <strong>Retrieval-Augmented Generation (RAG)</strong><sup>[14]</sup>: đây là một trong những phương pháp hiệu quả hiện nay để giảm thiểu ảo giác. Hệ thống RAG truy xuất thông tin liên quan từ kho tri thức bên ngoài (như cơ sở dữ liệu tài liệu, trang web) trước khi sinh, sau đó dùng thông tin truy xuất được làm ngữ cảnh, dẫn dắt mô hình sinh ra câu trả lời dựa trên sự thật.
   2. <strong>Suy luận và xác thực đa bước</strong>: dẫn dắt mô hình thực hiện suy luận nhiều bước, và tự kiểm tra hoặc xác thực bên ngoài ở mỗi bước.
   3. <strong>Đưa vào công cụ bên ngoài</strong>: cho phép mô hình gọi các công cụ bên ngoài (như công cụ tìm kiếm, máy tính, trình thông dịch code) để lấy thông tin thời gian thực hoặc thực hiện tính toán chính xác.

Mặc dù vấn đề ảo giác khó có thể loại bỏ hoàn toàn trong ngắn hạn, nhưng thông qua các chiến lược trên, có thể giảm đáng kể tần suất xuất hiện và ảnh hưởng của nó, nâng cao độ tin cậy và tính thực dụng của Large Language Model trong ứng dụng thực tế.

## 3.4 Tổng kết chương

Chương này giới thiệu các kiến thức nền tảng cần thiết để xây dựng Agent, tập trung xoay quanh Large Language Model (LLM) với tư cách là thành phần cốt lõi của nó. Nội dung bắt đầu từ sự phát triển ban đầu của mô hình ngôn ngữ, giải thích chi tiết kiến trúc Transformer, và giới thiệu các phương pháp tương tác với LLM. Cuối cùng, chương này đã điểm qua hệ sinh thái mô hình chủ đạo hiện nay, các quy luật phát triển và những hạn chế cố hữu của nó.

<strong>Điểm lại các kiến thức cốt lõi:</strong>

- <strong>Sự tiến hóa mô hình và kiến trúc cốt lõi</strong>: chương này đã truy ngược lại quá trình phát triển từ mô hình ngôn ngữ thống kê (N-gram) đến mô hình mạng nơ-ron (RNN, LSTM), rồi đến kiến trúc Transformer đặt nền móng cho LLM hiện đại. Thông qua việc hiện thực hóa code theo phương pháp "từ trên xuống", chương này đã phân rã các thành phần cốt lõi của Transformer, và trình bày vai trò then chốt của cơ chế self-attention trong tính toán song song và nắm bắt các phụ thuộc khoảng cách xa.
- <strong>Phương thức tương tác với mô hình</strong>: chương này giới thiệu hai khâu cốt lõi khi tương tác với LLM: Prompt Engineering và Text Tokenization. Cái trước dùng để dẫn dắt hành vi của mô hình, cái sau là nền tảng để hiểu cách xử lý đầu vào của mô hình. Thông qua thực hành triển khai và chạy mô hình open-source cục bộ, kiến thức lý thuyết đã được áp dụng vào thao tác thực tế.
- <strong>Hệ sinh thái mô hình và lựa chọn mô hình</strong>: chương này đã hệ thống hóa các yếu tố then chốt cần cân nhắc khi lựa chọn mô hình cho Agent, và điểm qua đặc điểm cùng định vị của các mô hình closed-source đại diện bởi OpenAI GPT, Google Gemini và các mô hình open-source đại diện bởi Llama, Mistral.
- <strong>Quy luật và hạn chế</strong>: chương này đã khám phá Scaling Laws thúc đẩy sự nâng cao năng lực của LLM, trình bày nguyên lý cơ bản đằng sau nó. Đồng thời, chương này cũng phân tích các hạn chế cố hữu của mô hình như ảo giác thực tế, tri thức lỗi thời, điều này vô cùng quan trọng đối với việc xây dựng Agent đáng tin cậy, bền vững.

<strong>Từ nền tảng LLM đến xây dựng Agent:</strong>

Nền tảng LLM trong chương này chủ yếu nhằm giúp mọi người hiểu rõ hơn về sự ra đời và quá trình phát triển của mô hình lớn, trong đó cũng chứa đựng một phần tư duy về thiết kế Agent. Ví dụ, cách thiết kế prompt hiệu quả để dẫn dắt việc lập kế hoạch và ra quyết định của Agent, cách lựa chọn mô hình phù hợp dựa trên yêu cầu tác vụ, cũng như cách thêm cơ chế xác thực vào workflow của Agent để né tránh ảo giác của mô hình, các vấn đề này đều có giải pháp được xây dựng trên nền tảng của chương này. Chúng ta hiện đã sẵn sàng chuyển từ lý thuyết sang thực hành. Trong chương tiếp theo, chúng ta sẽ bắt đầu khám phá việc xây dựng các mô hình kinh điển (paradigm) của Agent, áp dụng kiến thức đã học trong chương này vào thiết kế Agent thực tế.



## Bài tập

1. Trong xử lý ngôn ngữ tự nhiên, mô hình ngôn ngữ đã trải qua quá trình tiến hóa từ thống kê đến mạng nơ-ron.

   - Hãy sử dụng kho ngữ liệu mini được cung cấp trong chương này (`datawhale agent learns`, `datawhale agent works`), tính xác suất của câu `agent works` trong mô hình Bigram
   - Giả định cốt lõi của mô hình N-gram là giả định Markov. Hãy giải thích ý nghĩa của giả định này, và mô hình N-gram tồn tại những hạn chế căn bản nào?
   - Mô hình ngôn ngữ mạng nơ-ron (RNN/LSTM) và Transformer lần lượt khắc phục hạn chế của mô hình N-gram như thế nào? Ưu điểm riêng của mỗi loại là gì?

2. Kiến trúc Transformer<sup>[4]</sup> là nền tảng của Large Language Model hiện đại. Trong đó:

   > <strong>Gợi ý</strong>: có thể kết hợp với phần hiện thực hóa code ở mục 3.1.2 của chương này để hỗ trợ hiểu bài

   - Ý tưởng cốt lõi của cơ chế Self-Attention là gì?
   - Vì sao Transformer có thể xử lý song song chuỗi, trong khi RNN buộc phải xử lý tuần tự? Positional Encoding đóng vai trò gì trong đó?
   - Kiến trúc Decoder-Only khác gì so với kiến trúc Encoder-Decoder hoàn chỉnh? Vì sao các Large Language Model chủ đạo hiện nay đều áp dụng kiến trúc Decoder-Only?

3. Thuật toán subword tokenization văn bản là một công nghệ then chốt của Large Language Model, có nhiệm vụ chuyển đổi văn bản thành chuỗi token mà mô hình có thể xử lý. Vậy vì sao không thể trực tiếp dùng "ký tự" hoặc "từ" làm đơn vị đầu vào của mô hình? Thuật toán BPE (Byte Pair Encoding) đã giải quyết vấn đề gì?

4. Mục 3.2.3 của chương này đã giới thiệu cách triển khai Large Language Model mã nguồn mở cục bộ. Hãy hoàn thành thực hành và phân tích sau:

   > <strong>Gợi ý</strong>: đây là một bài tập thực hành, nên thao tác thực tế

   - Theo hướng dẫn của chương này, triển khai cục bộ một mô hình open-source nhẹ (khuyến nghị [Qwen3-0.6B](https://modelscope.cn/models/Qwen/Qwen3-0.6B)), thử điều chỉnh các tham số sampling và quan sát ảnh hưởng của chúng đến đầu ra
   - Chọn một tác vụ cụ thể (như phân loại văn bản, trích xuất thông tin, sinh code, v.v.), thiết kế và so sánh hiệu quả của các chiến lược prompt khác nhau (như Zero-shot, Few-shot, Chain-of-Thought) đối với kết quả đầu ra
   - So sánh mô hình closed-source và open-source từ các khía cạnh hiệu năng, chi phí, khả năng kiểm soát, quyền riêng tư
   - Nếu bạn muốn xây dựng một Agent chăm sóc khách hàng cấp doanh nghiệp, bạn sẽ chọn loại mô hình nào? Cần cân nhắc những yếu tố nào?

5. Ảo giác mô hình (Hallucination)<sup>[11]</sup> là một trong những hạn chế then chốt hiện nay của Large Language Model. Chương này đã giới thiệu các phương pháp giảm thiểu ảo giác (như Retrieval-Augmented Generation, suy luận đa bước, gọi công cụ bên ngoài)

   - Hãy chọn một trong số đó, trình bày nguyên lý hoạt động và kịch bản áp dụng của nó
   - Hãy tìm hiểu các nghiên cứu và bài báo tiên tiến, xem còn có phương pháp nào khác để giảm thiểu ảo giác của mô hình, chúng có những cải tiến và ưu điểm gì?
   
6. Giả sử bạn muốn thiết kế một Agent hỗ trợ đọc bài báo khoa học, nó có thể giúp nhà nghiên cứu nhanh chóng đọc và hiểu các bài báo học thuật, bao gồm: tóm tắt nội dung cốt lõi của nghiên cứu, trả lời câu hỏi về bài báo, trích xuất thông tin then chốt, so sánh quan điểm của nhiều bài báo khác nhau. Hãy trả lời:

   - Bạn sẽ chọn mô hình nào làm mô hình nền tảng khi thiết kế Agent? Cần cân nhắc những yếu tố nào khi lựa chọn?
   - Làm sao để thiết kế prompt dẫn dắt mô hình hiểu bài báo học thuật tốt hơn? Bài báo học thuật thường rất dài, có thể vượt quá giới hạn context window của mô hình, bạn sẽ giải quyết vấn đề này như thế nào?
   - Nghiên cứu học thuật đòi hỏi sự nghiêm ngặt, điều này có nghĩa là chúng ta cần đảm bảo thông tin do Agent sinh ra là chính xác, khách quan, trung thành với văn bản gốc. Bạn cho rằng nên thêm những thiết kế nào vào hệ thống để đáp ứng tốt hơn yêu cầu này?



## Tài liệu tham khảo

[1] Bengio, Y., Ducharme, R., Vincent, P., & Jauvin, C. (2003). A neural probabilistic language model. *Journal of Machine Learning Research*, 3, 1137-1155.

[2] Elman, J. L. (1990). Finding structure in time. *Cognitive Science*, 14(2), 179-211.

[3] Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. *Neural Computation*, 9(8), 1735-1780.

[4] Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., ... & Polosukhin, I. (2017). Attention is all you need. In *Advances in neural information processing systems* (pp. 5998-6008).

[5] Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). Improving language understanding by generative pre-training. OpenAI.

[6] Gage, P. (1994). A new algorithm for data compression. *C Users Journal*, *12*(2), 23-38.

[7] Schuster, M., & Nakajima, K. (2012, March). Japanese and korean voice search. In *2012 IEEE international conference on acoustics, speech and signal processing (ICASSP)* (pp. 5149-5152). IEEE.

[8] Kudo, T., & Richardson, J. (2018). SentencePiece: A simple and language independent subword tokenizer and detokenizer for neural text processing. *arXiv preprint arXiv:1808.06226*.

[9] Kaplan, J., McCandlish, S., Henighan, T., Brown, T. B., Chess, B., Child, R., ... & Amodei, D. (2020). Scaling Laws for Neural Language Models. arXiv preprint arXiv:2001.08361.

[10] Hoffmann, J., Borgeaud, E., Mensch, A., Buchatskaya, E., Cai, T., Rutherford, R., ... & Sifre, L. (2022). Training Compute-Optimal Large Language Models. arXiv preprint arXiv:2203.07678.

[11] Huang, L., Yu, W., Ma, W., Zhong, W., Feng, Z., Wang, H., ... & Liu, T. (2023). A Survey on Hallucination in Large Language Models: Principles, Taxonomy, Challenges, and Open Questions. *arXiv preprint arXiv:2311.05232*.

[12] Bender, E. M., Gebru, T., McMillan-Major, A., & Mitchell, M. (2021). On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? .

[13] Christiano, P., Leike, J., Brown, T. B., Martic, M., Legg, S., & Amodei, D. (2017). Deep reinforcement learning from human preferences. *arXiv preprint arXiv:1706.03741*.

[14] Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goswami, N., ... & Kiela, D. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. In *Advances in neural information processing systems* (pp. 9459-9474).
