# Tổng hợp câu hỏi phỏng vấn LLM & VLM & Agent

Tài liệu này được tổng hợp trong quá trình chuẩn bị phỏng vấn mùa tuyển dụng thu 2025, là bộ sưu tập các câu hỏi phỏng vấn "bát cổ" (kiến thức nền tảng kinh điển).

Chủ thớt chủ yếu ứng tuyển các vị trí: Kỹ sư thuật toán Large Language Model, Kỹ sư Agent, Kỹ sư phát triển AI, Kỹ sư đánh giá thuật toán..., các công ty phỏng vấn chủ yếu là các công ty internet lớn và vừa trong nước. Do đó, chiều sâu và độ rộng của các câu hỏi trong tài liệu này đều xoay quanh yêu cầu của các vị trí trên, nội dung bao phủ toàn bộ technical stack từ lý thuyết cốt lõi LLM/VLM, đến phát triển ứng dụng RAG/Agent, rồi đến kỹ thuật alignment RLHF và đánh giá model/Agent. Tất cả câu hỏi đều được tổng hợp từ những trải nghiệm phỏng vấn kỹ thuật online thực tế nhiều lần.

【Gợi ý sử dụng】
Tài liệu này chỉ dùng cho mục đích học tập và tham khảo. Để đạt hiệu quả tốt nhất, khuyến nghị mạnh mẽ bạn nên tự suy nghĩ độc lập từng câu hỏi, thử xây dựng câu trả lời của riêng mình trước, sau đó mới đối chiếu với hướng tham khảo mà tài liệu cung cấp để bổ sung những chỗ còn thiếu sót. Biết nó là gì, còn phải biết tại sao nó lại như vậy. Học thuộc lòng trực tiếp là cách làm có hiệu quả thấp nhất.

Chúc mọi người tìm việc thuận lợi, đều nhận được Offer mơ ước!

---

### 1. Kiến thức nền tảng LLM

1.  Hãy giải thích chi tiết cơ chế self-attention trong mô hình Transformer hoạt động như thế nào? Tại sao nó phù hợp hơn RNN khi xử lý chuỗi dài?
2.  Position encoding là gì? Trong Transformer, tại sao nó là bắt buộc? Hãy liệt kê ít nhất hai cách triển khai.
3.  Hãy giới thiệu chi tiết về RoPE, so với absolute position encoding thì ưu nhược điểm của nó lần lượt là gì?
4.  Bạn có biết sự khác biệt giữa MHA, MQA, GQA không? Hãy giải thích chi tiết.
5.  Hãy so sánh một vài kiến trúc LLM phổ biến, ví dụ Encoder-Only, Decoder-Only, và Encoder-Decoder, và nói rõ loại nhiệm vụ nào mỗi kiến trúc giỏi nhất.
6.  Scaling Laws là gì? Nó tiết lộ mối quan hệ gì giữa hiệu năng mô hình, lượng tính toán và lượng dữ liệu? Điều này có ý nghĩa chỉ đạo gì cho việc nghiên cứu phát triển LLM?
7.  Trong giai đoạn inference của LLM, có những chiến lược decoding phổ biến nào? Hãy giải thích nguyên lý cũng như ưu nhược điểm của Greedy Search, Beam Search, Top-K Sampling và Nucleus Sampling (Top-P).
8.  Tokenization là gì? Hãy so sánh hai thuật toán tách subword phổ biến là BPE và WordPiece.
9.  Bạn nghĩ sự khác biệt lớn nhất giữa NLP và LLM là gì? Hai cái này có điểm gì chung và khác nhau?
10. Regularization L1 và L2 lần lượt là gì, trường hợp nào thì phù hợp để sử dụng?
11. "Khả năng xuất hiện" (Emergent Abilities) là một hiện tượng được quan tâm nhiều ở các mô hình lớn, bạn hiểu khái niệm này như thế nào? Nó thường xuất hiện khi quy mô mô hình đạt đến mức độ nào?
12. Bạn có hiểu về activation function không, bạn biết những activation function nào thường dùng trong LLM? Tại sao lại chọn nó?
13. Mixture of Experts (MoE) đã mở rộng hiệu quả quy mô tham số của mô hình mà không làm tăng đáng kể chi phí inference như thế nào? Hãy trình bày tóm tắt nguyên lý hoạt động của nó.
14. Khi huấn luyện một LLM ở quy mô trăm tỷ hoặc nghìn tỷ tham số, bạn sẽ gặp phải những thách thức kỹ thuật và thuật toán chủ yếu nào? (Ví dụ: bộ nhớ GPU, giao tiếp, sự không ổn định trong huấn luyện...)
15. Bạn đã tìm hiểu những open-source framework nào? Bạn đã đọc qua paper của Qwen, Deepseek chưa, hãy nói về những điểm sáng tạo chính của chúng thể hiện ở đâu?
16. Gần đây bạn đã đọc những paper tiên phong nào về LLM, hãy trao đổi về phương pháp liên quan, nhắm vào vấn đề gì, đề xuất phương pháp gì, có những thí nghiệm so sánh nào?

---

### 2. Kiến thức nền tảng VLM

1.  Thách thức cốt lõi của mô hình đa phương thức lớn (như VLM) là gì? Tức là làm thế nào để đạt được sự alignment và fusion hiệu quả giữa các thông tin thuộc các modality khác nhau (như hình ảnh và ngôn ngữ)?
2.  Hãy giải thích nguyên lý hoạt động của mô hình CLIP. Nó kết nối hình ảnh và văn bản thông qua contrastive learning như thế nào?
3.  Những mô hình như LLaVA hay MiniGPT-4 kết nối một Vision Encoder đã được pretrain với một Large Language Model (LLM) như thế nào? Hãy mô tả thiết kế kiến trúc then chốt của chúng.
4.  Visual instruction tuning là gì? Tại sao nói đây là bước then chốt để VLM có khả năng hội thoại và tuân theo instruction tốt?
5.  Khi xử lý dữ liệu đa phương thức như video, so với hình ảnh tĩnh, VLM cần giải quyết thêm những vấn đề gì? (Ví dụ: làm thế nào để biểu diễn thông tin thời gian?)
6.  Hãy giải thích ý nghĩa của Grounding trong lĩnh vực VLM. Chúng ta đánh giá một VLM có thể ánh xạ chính xác mô tả văn bản đến vùng cụ thể trong hình ảnh hay không bằng cách nào?
7.  Hãy so sánh ít nhất các mô hình kiến trúc VLM khác nhau (như shared encoder so với cross-modal attention fusion), và phân tích ưu nhược điểm của chúng.
8.  Trong ứng dụng của VLM, làm thế nào để xử lý ảnh đầu vào có độ phân giải cao? Điều này sẽ mang lại những thách thức gì về tính toán và thiết kế mô hình?
9.  Khi VLM sinh nội dung, nó cũng sẽ gặp phải vấn đề "ảo giác" (Hallucination), nhưng biểu hiện của nó khác với LLM thuần văn bản như thế nào? Hãy cho ví dụ minh họa.
10. Ngoài mô tả hình ảnh và visual question answering (VQA), bạn còn có thể liệt kê những hướng ứng dụng tiên phong hoặc tiềm năng nào khác của VLM?
11. Bạn đã từng fine-tune liên quan đến VLM chưa? Model nào?

---

### 3. Kiến thức nền tảng RLHF

1.  So với SFT truyền thống, RLHF nhằm giải quyết những vấn đề cốt lõi nào trong mô hình ngôn ngữ? Tại sao nói bản thân SFT không đủ để đạt được mục tiêu "alignment" mà chúng ta mong muốn?
2.  Hãy trình bày chi tiết ba giai đoạn cốt lõi của quy trình RLHF kinh điển. Ở mỗi giai đoạn, đầu vào là gì, đầu ra là gì, và mục tiêu then chốt của giai đoạn đó là gì?
3.  Trong giai đoạn huấn luyện RM, chúng ta thường thu thập dữ liệu so sánh theo cặp, thay vì để người gán nhãn trực tiếp chấm một điểm số tuyệt đối cho câu trả lời. Bạn nghĩ ưu điểm chính và nhược điểm tiềm ẩn của cách làm này lần lượt là gì?
4.  Thiết kế của reward model là vô cùng quan trọng. Kiến trúc mô hình của nó thường được chọn như thế nào? Nó có quan hệ gì với LLM mà cuối cùng chúng ta muốn tối ưu? Khi huấn luyện reward model, loss function thường dùng là gì? Hãy giải thích nguyên lý toán học đằng sau nó (ví dụ, có thể kết hợp mô hình Bradley-Terry để giải thích).
5.  Ở giai đoạn thứ ba của RLHF, PPO là thuật toán reinforcement learning phổ biến nhất. Tại sao lại chọn PPO, mà không phải các thuật toán policy gradient đơn giản hơn khác (như REINFORCE) hoặc các thuật toán nhóm Q-learning? Thành phần phạt KL divergence trong PPO đóng vai trò then chốt gì?
6.  Nếu trong quá trình huấn luyện PPO, hệ số β của thành phần phạt KL divergence được đặt quá lớn hoặc quá nhỏ, sẽ lần lượt dẫn đến những vấn đề gì? Bạn sẽ điều chỉnh siêu tham số này thông qua thí nghiệm và quan sát như thế nào?
7.  "Reward Hacking" là gì? Hãy kết hợp một tình huống ứng dụng LLM cụ thể để đưa ra ví dụ, và thảo luận một vài chiến lược giảm thiểu khả thi.
8.  Quy trình RLHF phức tạp và không ổn định. Những năm gần đây đã xuất hiện một số phương án thay thế, ví dụ DPO. Hãy giải thích ý tưởng cốt lõi của DPO, và so sánh sự khác biệt cũng như ưu thế chính của nó so với RLHF truyền thống (dựa trên PPO).
9.  Hãy tưởng tượng, mô hình RLHF bạn huấn luyện xong thể hiện xuất sắc trong offline evaluation, điểm reward model rất cao, nhưng sau khi lên production thì người dùng phản hồi câu trả lời của nó ngày càng trở nên "khuôn mẫu", nịnh hót, và thiếu thông tin. Bạn nghĩ nguyên nhân có thể là gì? Bạn sẽ bắt đầu phân tích và giải quyết vấn đề này từ những khía cạnh nào?
10. Bạn có biết GRPO của Deepseek không, sự khác biệt chính của nó với PPO là gì? Ưu nhược điểm là gì?
11. Bạn đã nghe về GSPO và DAPO chưa? Chúng khác gì với GRPO?
12. Làm thế nào để giải quyết vấn đề credit assignment? Reward ở cấp độ token và cấp độ sequence khác nhau như thế nào?
13. Ngoài phản hồi của con người, chúng ta còn có thể tận dụng phản hồi của chính AI để làm alignment, tức là RLAIF. Hãy nói về hiểu biết của bạn về RLAIF, tiềm năng và rủi ro của nó lần lượt là gì?

---

### 4. Agent

1.  Bạn định nghĩa một Agent dựa trên LLM như thế nào? Nó thường bao gồm những thành phần cốt lõi nào?
2.  Hãy giải thích chi tiết framework ReAct. Nó kết hợp chuỗi tư duy (Chain-of-Thought) và hành động như thế nào để hoàn thành nhiệm vụ phức tạp?
3.  Trong thiết kế Agent, "khả năng lập kế hoạch" (Planning) là vô cùng quan trọng. Hãy nói về những phương pháp phổ biến hiện nay để trao cho LLM khả năng lập kế hoạch? (Ví dụ CoT, ToT, GoT...)
4.  Memory là một module then chốt của Agent. Làm thế nào để thiết kế hệ thống bộ nhớ ngắn hạn và dài hạn cho Agent? Có thể dựa vào những công cụ hoặc kỹ thuật bên ngoài nào?
5.  Tool Use là con đường hiệu quả để mở rộng khả năng của Agent. Hãy giải thích LLM học cách gọi API hoặc công cụ bên ngoài như thế nào? (Có thể giải thích từ góc độ Function Calling)
6.  Hãy so sánh hai framework phát triển Agent phổ biến, như LangChain và LlamaIndex. Các kịch bản ứng dụng cốt lõi của chúng khác nhau như thế nào?
7.  Khi xây dựng một Agent phức tạp, bạn nghĩ thách thức chủ yếu nhất là gì?
8.  Hệ thống đa Agent (Multi-agent System) là gì? Để nhiều LLM Agent cùng phối hợp làm việc có ưu thế gì so với một Agent đơn lẻ? Và sẽ đưa ra những độ phức tạp mới nào?
9.  Khi một Agent cần thực hiện nhiệm vụ trong môi trường thực tế hoặc mô phỏng (như robot, game), nó khác biệt bản chất gì so với Agent thuần túy dựa trên công cụ phần mềm?
10. Làm thế nào để đảm bảo hành vi của một Agent là an toàn, có thể kiểm soát và phù hợp với ý định của con người? Trong thiết kế Agent, có những phương pháp bảo đảm alignment nào?
11. Bạn có hiểu về framework A2A không? Sự khác biệt giữa nó và framework Agent thông thường ở đâu, hãy chọn một điểm khác biệt then chốt nhất để minh họa.
12. Bạn đã dùng qua những Agent framework nào? Bạn chọn lựa như thế nào? Chỉ số đánh giá cuối cùng cho kịch bản của bạn là gì?
13. Bạn đã từng fine-tune khả năng của Agent chưa? Dataset được thu thập như thế nào?

---

### 5. RAG

1.  Hãy giải thích nguyên lý hoạt động của RAG. So với việc fine-tune trực tiếp LLM, RAG chủ yếu giải quyết vấn đề gì? Có những ưu thế nào?
2.  Một pipeline RAG hoàn chỉnh bao gồm những bước then chốt nào? Hãy mô tả chi tiết toàn bộ quá trình từ chuẩn bị dữ liệu đến sinh kết quả cuối cùng.
3.  Khi xây dựng knowledge base, chiến lược chia nhỏ văn bản (chunking) là vô cùng quan trọng. Bạn sẽ chọn kích thước chunk và độ dài overlap phù hợp như thế nào? Đằng sau đó có sự đánh đổi gì?
4.  Làm thế nào để chọn một embedding model phù hợp? Có những chỉ số nào để đánh giá một Embedding model tốt hay không?
5.  Ngoài vector retrieval cơ bản, bạn còn biết những kỹ thuật nào có thể nâng cao chất lượng retrieval của RAG?
6.  Hãy giải thích vấn đề "Lost in the Middle". Nó mô tả hiện tượng gì trong RAG? Có phương pháp nào để giảm thiểu vấn đề này?
7.  Làm thế nào để đánh giá toàn diện hiệu năng của một hệ thống RAG? Hãy đề xuất chỉ số đánh giá lần lượt từ hai giai đoạn retrieval và generation.
8.  Trong trường hợp nào, bạn sẽ chọn sử dụng graph database hoặc knowledge graph để tăng cường hoặc thay thế retrieval bằng vector database truyền thống?
9.  Quy trình RAG truyền thống là "retrieval trước, generation sau", bạn có hiểu một số RAG paradigm phức tạp hơn không, ví dụ như thực hiện nhiều lần retrieval hoặc adaptive retrieval trong quá trình generation?
10. Hệ thống RAG khi triển khai thực tế có thể gặp phải những thách thức nào?
11. Bạn có hiểu về search system không? Nó khác gì với RAG?
12. Bạn biết hoặc đã dùng qua những open-source RAG framework nào như Ragflow? Làm thế nào để chọn kịch bản phù hợp?

---

### 6. Đánh giá Model và đánh giá Agent

1.  Tại sao các chỉ số đánh giá NLP truyền thống (như BLEU, ROUGE) lại có nhiều hạn chế khi đánh giá chất lượng sinh nội dung của LLM hiện đại?
2.  Hãy giới thiệu một vài bộ benchmark tổng hợp của LLM đang được sử dụng rộng rãi trong ngành hiện nay, và nói rõ trọng tâm của từng bộ. (Ví dụ: MMLU, Big-Bench, HumanEval)
3.  "LLM-as-a-Judge" là gì? Sử dụng LLM để đánh giá output của một LLM khác có những ưu điểm và thiên lệch (bias) tiềm ẩn nào?
4.  Làm thế nào để thiết kế một phương án đánh giá nhằm đo lường khả năng cụ thể của LLM, ví dụ như "mức độ chính xác về sự thật/hallucination", "khả năng suy luận" hoặc "tính an toàn"?
5.  Tại sao đánh giá một Agent lại khó khăn và phức tạp hơn so với đánh giá một LLM nền tảng? Các chiều đánh giá có gì khác nhau?
6.  Bạn hiểu những benchmark chuyên dùng để đánh giá khả năng Agent nào? Những benchmark này thường xây dựng môi trường và nhiệm vụ test như thế nào?
7.  Khi đánh giá kết quả hoàn thành nhiệm vụ của một Agent, ngoài tính đúng đắn của kết quả cuối cùng, còn những chỉ số về quá trình nào đáng để quan tâm? (Ví dụ: hiệu suất, chi phí, tính bền vững)
8.  Red teaming là gì? Nó đóng vai trò gì trong việc phát hiện lỗ hổng an toàn và thiên lệch của LLM và Agent?
9.  Khi thực hiện đánh giá thủ công (human evaluation), làm thế nào để thiết kế tiêu chí và quy trình đánh giá hợp lý, nhằm đảm bảo tính khách quan và nhất quán của kết quả đánh giá?
10. Làm thế nào để giám sát và đánh giá liên tục hiệu năng của một ứng dụng LLM hoặc dịch vụ Agent đã triển khai lên production, nhằm ứng phó với sự suy giảm hiệu năng hoặc hành vi drift có thể xảy ra?

---

### 7. Triển vọng và phát triển của LLM

1.  Bạn nghĩ LLM hiện tại còn cách Artificial General Intelligence (AGI) bao xa? Khả năng thiếu sót then chốt nhất là gì?
2.  Từ GPT-4 đến các mô hình tương lai, bạn nghĩ sự hợp nhất đa phương thức sẽ đi về hướng nào? Chỉ đơn thuần là kết hợp văn bản, hình ảnh, hay sẽ mở rộng đến nhiều chiều giác quan hơn?
3.  Bạn nhìn nhận thế nào về sự cạnh tranh và cùng tồn tại giữa hệ sinh thái mô hình open-source và closed-source? Ưu thế riêng của chúng là gì, trong tương lai sẽ phát triển ra sao?
4.  Khi khả năng của mô hình tăng lên, "world model" hay khả năng mô phỏng nội tại của LLM cũng được quan tâm nhiều. Bạn hiểu khái niệm này như thế nào? Nó có ý nghĩa gì đối với việc đạt được khả năng suy luận và lập kế hoạch ở cấp độ cao hơn?
5.  "Dữ liệu" là nhiên liệu để huấn luyện LLM. Bạn nghĩ dữ liệu tổng hợp nhân tạo (synthetic data) chất lượng cao sẽ đóng vai trò gì trong huấn luyện mô hình tương lai?
6.  Embodied AI, tức là sự kết hợp giữa LLM và robot, được coi là làn sóng tiếp theo của AI. Bạn nghĩ LLM sẽ trao quyền cho robot như thế nào, và sẽ mang lại những thách thức gì?
7.  Cá nhân hóa là hướng đi quan trọng của ứng dụng LLM. Trong quá trình hiện thực hóa Agent hoặc trợ lý được cá nhân hóa cao, chúng ta nên cân bằng hiệu quả, quyền riêng tư và an toàn như thế nào?
8.  Bạn nghĩ kiến trúc Transformer có thống trị lĩnh vực này lâu dài không? Hay bạn nhìn thấy tiềm năng của các kiến trúc mới như state space model (SSM, như Mamba)?
9.  Nhìn về 3-5 năm tới, bạn nghĩ công nghệ LLM và Agent có khả năng tạo ra ứng dụng đột phá đầu tiên ở ngành hoặc lĩnh vực nào? Tại sao?

---

### 8. Khác

1.  Bạn nghĩ nút thắt cổ chai lớn nhất hiện đang hạn chế khả năng và sự phổ cập của Agent là gì? (Ví dụ: khả năng của mô hình, chi phí, độ tin cậy, hay yếu tố khác?)
2.  Trong nửa năm qua, bài paper hoặc dự án open-source nào về Agent để lại ấn tượng sâu sắc nhất với bạn? Tại sao?
3.  Bạn nhìn nhận thế nào về "khả năng xuất hiện" (emergent abilities) trong lĩnh vực Agent? Chúng ta nên theo đuổi mô hình nền tảng mạnh mẽ hơn, hay kiến trúc Agent tinh xảo hơn?
4.  Bạn nghĩ trong 1-2 năm tới, công nghệ Agent có khả năng đạt được thương mại hóa quy mô lớn đầu tiên ở ngành hoặc tình huống nào?
5.  Nếu để bạn tự do khám phá, bạn muốn tạo ra một Agent như thế nào để giải quyết vấn đề gì?
6.  Đối với người mới bắt đầu muốn bước vào lĩnh vực Agent, bạn sẽ cho họ lời khuyên gì? Nên tập trung học những kỹ thuật nào?
7.  Tóm lại, bạn nghĩ một kỹ sư AI Agent xuất sắc nên có những phẩm chất cốt lõi nào?
8.  Bạn có thường dùng AI không, dùng để làm gì? Nếu tôi muốn sử dụng AI, ví dụ trong lĩnh vực coding, bạn có lời khuyên gì cho tôi?
</content>
