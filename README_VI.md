<div align="right">
  <a href="./README_EN_vi.md">English</a> | Tiếng Việt
</div>

<div align='center'>
  <img src="./docs/images/hello-agents.png" alt="alt text" width="100%">
  <h1>Hello-Agents</h1>
  <h3>🤖 《Xây dựng Agent từ đầu》</h3>
  <div align="center">
  <a href="https://trendshift.io/repositories/15520" target="_blank">
    <img src="https://trendshift.io/api/badge/repositories/15520" alt="datawhalechina%2Fhello-agents | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/>
  </a>
  </div>
  <p><em>Từ lý thuyết nền tảng đến ứng dụng thực tế, nắm vững toàn diện việc thiết kế và triển khai hệ thống Agent</em></p>
  <img src="https://img.shields.io/github/stars/datawhalechina/Hello-Agents?style=flat&logo=github" alt="GitHub stars"/>
  <img src="https://img.shields.io/github/forks/datawhalechina/Hello-Agents?style=flat&logo=github" alt="GitHub forks"/>
  <img src="https://img.shields.io/badge/language-Chinese-brightgreen?style=flat" alt="Language"/>
  <a href="https://github.com/datawhalechina/Hello-Agents"><img src="https://img.shields.io/badge/GitHub-Project-blue?style=flat&logo=github" alt="GitHub Project"></a>
  <a href="https://datawhalechina.github.io/hello-agents/"><img src="https://img.shields.io/badge/在线阅读-Online%20Reading-green?style=flat&logo=gitbook" alt="Online Reading"></a>
</div>

---

## 🎯 Giới thiệu dự án

&emsp;&emsp;Nếu nói năm 2024 là năm mở đầu của "cuộc chiến trăm mô hình", thì năm 2025 chắc chắn đã mở ra "năm của Agent". Trọng tâm công nghệ đang chuyển dịch từ việc huấn luyện các mô hình nền tảng (foundation model) lớn hơn, sang việc xây dựng các ứng dụng Agent thông minh hơn. Tuy nhiên, các tutorial mang tính hệ thống và chú trọng thực hành hiện vẫn vô cùng khan hiếm. Vì vậy, chúng tôi đã khởi xướng dự án Hello-Agents, với mong muốn mang đến cho cộng đồng một cuốn hướng dẫn xây dựng hệ thống Agent từ con số 0, cân bằng giữa lý thuyết và thực hành.

&emsp;&emsp;Hello-Agents là <strong>tutorial học tập Agent mang tính hệ thống</strong> của cộng đồng Datawhale. Hiện nay, việc xây dựng Agent chủ yếu chia thành hai trường phái: một là các Agent kiểu kỹ thuật phần mềm (software engineering) như Dify, Coze, n8n, bản chất là phát triển phần mềm theo hướng quy trình (process-driven), trong đó LLM chỉ đóng vai trò backend xử lý dữ liệu; trường phái còn lại là Agent AI-native, tức là Agent thực sự được điều khiển bởi AI. Tutorial này nhằm dẫn dắt mọi người hiểu sâu và xây dựng trường phái thứ hai — Agent AI Native thực thụ. Tutorial sẽ đưa bạn xuyên qua lớp vỏ bề ngoài của các framework, xuất phát từ nguyên lý cốt lõi của Agent, đi sâu vào kiến trúc cốt lõi của nó, hiểu rõ các paradigm kinh điển, và cuối cùng tự tay xây dựng ứng dụng multi-agent của riêng bạn. Chúng tôi tin rằng cách học tốt nhất chính là thực hành. Hy vọng cuốn tutorial này sẽ trở thành điểm khởi đầu cho bạn khám phá thế giới Agent, giúp bạn chuyển mình từ một "người sử dụng" Large Language Model thành một "người xây dựng" hệ thống Agent.

## 📚 Bắt đầu nhanh

### Đọc online
**[🌐 Truy cập quốc tế](https://datawhalechina.github.io/hello-agents/)** | **[🚀 Tăng tốc trong nước (Trung Quốc)](https://hello-agents.datawhale.cc)** - Không cần tải xuống, học mọi lúc mọi nơi

### Đọc offline
Nếu bạn muốn đọc hoặc đóng góp nội dung offline, vui lòng tham khảo hướng dẫn học tập bên dưới.

### ✨ Bạn sẽ nhận được gì?

- 📖 <strong>Datawhale mã nguồn mở miễn phí</strong> Học hoàn toàn miễn phí toàn bộ nội dung của dự án này, cùng phát triển với cộng đồng
- 🔍 <strong>Hiểu nguyên lý cốt lõi</strong> Hiểu sâu về khái niệm, lịch sử và các paradigm kinh điển của Agent
- 🏗️ <strong>Tự tay triển khai</strong> Nắm vững cách sử dụng các nền tảng low-code phổ biến và các framework code cho Agent
- 🛠️ <strong>Framework tự phát triển [HelloAgents](https://github.com/jjyaoao/helloagents)</strong> Xây dựng từ đầu một framework Agent của riêng bạn dựa trên API gốc của OpenAI
- ⚙️ <strong>Nắm vững các kỹ năng nâng cao</strong> Từng bước triển khai các kỹ thuật mang tính hệ thống như Context Engineering, Memory, protocol, đánh giá (evaluation)
- 🤝 <strong>Huấn luyện mô hình</strong> Nắm vững Agentic RL, thực hành toàn bộ quy trình huấn luyện LLM từ SFT đến GRPO
- 🚀 <strong>Dẫn dắt bằng case thực tế</strong> Phát triển thực chiến trợ lý du lịch thông minh, thị trấn cyber (cyber town) và các dự án tổng hợp khác
- 📖 <strong>Phỏng vấn xin việc</strong> Học các câu hỏi phỏng vấn liên quan đến việc tìm việc làm Agent

## 📖 Danh mục nội dung

| Chương                                                                                       | Nội dung chính                                | Trạng thái |
| --------------------------------------------------------------------------------------------- | ---------------------------------------------- | ---- |
| [Lời nói đầu](./docs/前言_vi.md)                                                               | Nguồn gốc, bối cảnh của dự án và lời khuyên cho độc giả | ✅    |
| <strong>Phần 1: Nền tảng về Agent và mô hình ngôn ngữ</strong>                                 |                                                 |      |
| [Chương 1 Làm quen với Agent](./docs/chapter1/第一章%20初识智能体_vi.md)                        | Định nghĩa, phân loại, paradigm và ứng dụng của Agent | ✅    |
| [Chương 2 Lịch sử phát triển của Agent](./docs/chapter2/第二章%20智能体发展史_vi.md)            | Quá trình tiến hóa của Agent từ chủ nghĩa biểu tượng đến LLM | ✅    |
| [Chương 3 Nền tảng mô hình ngôn ngữ lớn](./docs/chapter3/第三章%20大语言模型基础_vi.md)         | Transformer, prompt, các LLM chủ đạo và hạn chế của chúng | ✅    |
| <strong>Phần 2: Xây dựng Agent LLM của bạn</strong>                                            |                                                 |      |
| [Chương 4 Xây dựng các paradigm kinh điển của Agent](./docs/chapter4/第四章%20智能体经典范式构建_vi.md) | Triển khai từng bước ReAct, Plan-and-Solve, Reflection | ✅    |
| [Chương 5 Xây dựng Agent dựa trên nền tảng low-code](./docs/chapter5/第五章%20基于低代码平台的智能体搭建_vi.md) | Tìm hiểu cách sử dụng các nền tảng Agent low-code như Coze, Dify, n8n | ✅    |
| [Chương 6 Thực hành phát triển với framework](./docs/chapter6/第六章%20框架开发实践_vi.md)      | Ứng dụng các framework chủ đạo như AutoGen, AgentScope, LangGraph | ✅    |
| [Chương 7 Xây dựng framework Agent của bạn](./docs/chapter7/第七章%20构建你的Agent框架_vi.md)   | Xây dựng framework Agent từ con số 0            | ✅    |
| <strong>Phần 3: Mở rộng kiến thức nâng cao</strong>                                            |                                                 |      |
| [Chương 8 Memory và truy xuất](./docs/chapter8/第八章%20记忆与检索_vi.md)                       | Hệ thống Memory, RAG, lưu trữ                   | ✅    |
| [Chương 9 Context Engineering](./docs/chapter9/第九章%20上下文工程_vi.md)                       | "Hiểu ngữ cảnh" trong tương tác liên tục         | ✅    |
| [Chương 10 Giao thức giao tiếp của Agent](./docs/chapter10/第十章%20智能体通信协议_vi.md)        | Phân tích các giao thức MCP, A2A, ANP            | ✅    |
| [Chương 11 Agentic-RL](./docs/chapter11/第十一章%20Agentic-RL_vi.md)                           | Thực hành huấn luyện LLM từ SFT đến GRPO        | ✅    |
| [Chương 12 Đánh giá hiệu năng Agent](./docs/chapter12/第十二章%20智能体性能评估_vi.md)           | Các chỉ số cốt lõi, benchmark và framework đánh giá | ✅    |
| <strong>Phần 4: Case tổng hợp nâng cao</strong>                                                |                                                 |      |
| [Chương 13 Trợ lý du lịch thông minh](./docs/chapter13/第十三章%20智能旅行助手_vi.md)            | Ứng dụng thực tế của MCP và sự phối hợp multi-agent | ✅    |
| [Chương 14 Agent nghiên cứu chuyên sâu tự động hóa](./docs/chapter14/第十四章%20自动化深度研究智能体_vi.md) | Tái hiện và phân tích DeepResearch Agent         | ✅    |
| [Chương 15 Xây dựng thị trấn cyber](./docs/chapter15/第十五章%20构建赛博小镇_vi.md)              | Sự kết hợp giữa Agent và game, mô phỏng động lực xã hội | ✅    |
| <strong>Phần 5: Đồ án tốt nghiệp và triển vọng tương lai</strong>                              |                                                 |      |
| [Chương 16 Đồ án tốt nghiệp](./docs/chapter16/第十六章%20毕业设计_vi.md)                        | Xây dựng ứng dụng multi-agent hoàn chỉnh của riêng bạn | ✅    |

### Tuyển chọn đóng góp cộng đồng (Community Blog)

&emsp;&emsp;Hoan nghênh mọi người đóng góp những góc nhìn độc đáo, tổng kết thực hành của mình trong quá trình học Hello-Agents hoặc các công nghệ liên quan đến Agent, dưới hình thức PR vào mục tuyển chọn cộng đồng. Nếu là nội dung độc lập với phần chính văn, bạn cũng có thể gửi bài vào Extra-Chapter! <strong>Mong chờ lần đóng góp đầu tiên của bạn!</strong>

| Tuyển chọn cộng đồng                                                                                                                            | Tóm tắt nội dung                  |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [00-Đồ án tốt nghiệp đồng sáng tạo](https://github.com/datawhalechina/hello-agents/blob/main/Co-creation-projects)                             | Dự án đồ án tốt nghiệp đồng sáng tạo cộng đồng      |
| [01-Tổng hợp câu hỏi phỏng vấn Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra01-面试问题总结.md)                          | Các câu hỏi phỏng vấn liên quan đến vị trí Agent    |
| [01-Đáp án câu hỏi phỏng vấn Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra01-参考答案.md)                              | Đáp án cho các câu hỏi phỏng vấn liên quan          |
| [02-Bổ sung nội dung Context Engineering](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra02-上下文工程补充知识.md)                 | Mở rộng nội dung Context Engineering        |
| [03-Hướng dẫn tạo Agent trên Dify chi tiết từng bước](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra03-Dify智能体创建保姆级操作流程.md) | Hướng dẫn chi tiết từng bước tạo Agent trên Dify  |
| [04-Câu hỏi thường gặp về khóa học Hello-Agents](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra04-DatawhaleFAQ.md)                 | Câu hỏi thường gặp về khóa học Datawhale     |
| [05-So sánh Agent Skills và MCP](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra05-AgentSkills解读.md)             | So sánh công nghệ Agent Skills và MCP |
| [06-Giới thiệu và thực hành GUI Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra06-GUIAgent科普与实战.md)                | Giới thiệu kiến thức và thực hành GUI Agent trong nhiều tình huống |
| [07-Cấu hình môi trường](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)                | Cấu hình môi trường |
| [08-Cách viết Skill tốt](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra08-如何写出好的Skill.md) | Best practice khi viết Skill |
| [09-Chia sẻ kinh nghiệm và lỗi thường gặp khi phát triển ứng dụng Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra09-Agent应用开发实践踩坑与经验分享.md) | Tổng kết kinh nghiệm và lỗi thường gặp khi phát triển ứng dụng Code Agent |
| [10-Agent Self-Evolution (Agent tự tiến hóa)](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra10-Agent自进化.md) | Bốn loại vòng lặp (closed-loop) tự tiến hóa của Agent và các dự án tiêu biểu |
| [11-Giới thiệu và thực hành Web Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra11-WebAgent科普与实战.md) | Nguyên lý Web Agent, thực hành chống anti-crawl và tích hợp với HelloAgents |
| [12-Thực hành post-training cho trợ lý du lịch](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra12-旅行助手后训练实战.md) | Hoàn thiện Demo trợ lý du lịch thành một Planner có thể sử dụng thực tế |
| [13-Đồng sáng tạo ghi hình khóa học video Hello-Agents](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra13-Hello-Agents视频课录制共创.md) | Tài nguyên đồng sáng tạo ghi hình khóa học video |

### Tải xuống bản PDF

&emsp;&emsp;*<strong>Tutorial PDF của Hello-Agents hoàn toàn mã nguồn mở và miễn phí. Để ngăn chặn các tài khoản marketing đóng dấu watermark rồi bán lại cho những người mới bắt đầu học hệ thống multi-agent, chúng tôi đã chủ động thêm watermark logo mã nguồn mở của Datawhale vào file PDF (không ảnh hưởng đến việc đọc), mong bạn thông cảm~</strong>*

> *Hello-Agents PDF : https://github.com/datawhalechina/hello-agents/releases/latest/*  
> *Địa chỉ tải PDF Hello-Agents trong nước (Trung Quốc) : https://www.datawhale.cn/learn/summary/239* 

## 💡 Cách học tập

&emsp;&emsp;Chào mừng bạn, người xây dựng hệ thống thông minh trong tương lai! Trước khi bắt đầu hành trình đầy hứng khởi này, hãy để chúng tôi đưa ra cho bạn một vài chỉ dẫn rõ ràng.

&emsp;&emsp;Nội dung dự án này chú trọng cả lý thuyết lẫn thực hành, nhằm giúp bạn nắm vững một cách có hệ thống toàn bộ quy trình thiết kế và phát triển từ một Agent đơn lẻ đến hệ thống multi-agent. Vì vậy, dự án đặc biệt phù hợp với những <strong>AI developer, kỹ sư phần mềm, sinh viên</strong> đã có nền tảng lập trình nhất định, cũng như các <strong>người tự học</strong> có niềm đam mê mãnh liệt với công nghệ AI tiên tiến. Trước khi học dự án này, chúng tôi mong bạn có khả năng lập trình Python cơ bản, và có hiểu biết khái niệm cơ bản về Large Language Model (ví dụ: biết cách gọi một LLM thông qua API). Trọng tâm của dự án là ứng dụng và xây dựng, nên bạn không cần có nền tảng sâu về thuật toán hay huấn luyện mô hình.

&emsp;&emsp;Dự án được chia thành năm phần lớn, mỗi phần là một bậc thang vững chắc dẫn tới giai đoạn tiếp theo:

- <strong>Phần 1: Nền tảng về Agent và mô hình ngôn ngữ</strong> (Chương 1 ~ Chương 3), chúng ta sẽ bắt đầu từ định nghĩa, phân loại và lịch sử phát triển của Agent, giúp bạn hiểu rõ ngọn ngành khái niệm "Agent". Sau đó, chúng ta sẽ nhanh chóng củng cố kiến thức cốt lõi về Large Language Model, đặt nền móng lý thuyết vững chắc cho hành trình thực hành của bạn.

- <strong>Phần 2: Xây dựng Agent LLM của bạn</strong> (Chương 4 ~ Chương 7), đây là điểm khởi đầu cho việc thực hành của bạn. Bạn sẽ tự tay triển khai các paradigm kinh điển như ReAct, trải nghiệm sự tiện lợi của các nền tảng low-code như Coze, và nắm vững cách ứng dụng các framework chủ đạo như Langgraph. Cuối cùng, chúng tôi sẽ dẫn dắt bạn xây dựng từ con số 0 một framework Agent của riêng mình, giúp bạn vừa có khả năng "dùng bánh xe có sẵn" vừa có khả năng "tự chế tạo bánh xe".

- <strong>Phần 3: Mở rộng kiến thức nâng cao</strong> (Chương 8 ~ Chương 12), trong phần này, Agent của bạn sẽ "học được" cách suy nghĩ và phối hợp. Chúng ta sẽ sử dụng framework tự phát triển ở Phần 2, đi sâu khám phá các công nghệ cốt lõi như Memory và retrieval, Context Engineering, huấn luyện Agent, đồng thời học về các giao thức giao tiếp giữa các multi-agent. Cuối cùng, bạn sẽ nắm vững phương pháp chuyên nghiệp để đánh giá hiệu năng hệ thống Agent.

- <strong>Phần 4: Case tổng hợp nâng cao</strong> (Chương 13 ~ Chương 15), đây là nơi giao thoa giữa lý thuyết và thực hành. Bạn sẽ vận dụng tổng hợp những gì đã học, tự tay xây dựng trợ lý du lịch thông minh, Agent nghiên cứu chuyên sâu tự động hóa, thậm chí một thị trấn cyber mô phỏng động lực xã hội, rèn luyện khả năng xây dựng của bạn qua những dự án chân thực và thú vị.

- <strong>Phần 5: Đồ án tốt nghiệp và triển vọng tương lai</strong> (Chương 16), tại điểm cuối của hành trình, bạn sẽ bước vào đồ án tốt nghiệp, xây dựng một ứng dụng multi-agent hoàn chỉnh của riêng bạn, kiểm tra toàn diện kết quả học tập. Chúng ta cũng sẽ cùng bạn nhìn về tương lai của Agent, khám phá những hướng đi tiên phong đầy hứng khởi.


&emsp;&emsp;Agent là một lĩnh vực phát triển cực nhanh và phụ thuộc rất nhiều vào thực hành. Để đạt được hiệu quả học tập tốt nhất, chúng tôi đã cung cấp toàn bộ code đi kèm trong thư mục `code` của dự án, rất khuyến khích bạn <strong>kết hợp lý thuyết với thực hành</strong>. Hãy nhớ tự tay chạy, debug, thậm chí chỉnh sửa từng đoạn code được cung cấp trong dự án. Hoan nghênh bạn theo dõi Datawhale cũng như các cộng đồng liên quan đến Agent khác, khi gặp vấn đề, bạn có thể đặt câu hỏi bất cứ lúc nào trong khu vực issue của dự án này.

&emsp;&emsp;Bây giờ, bạn đã sẵn sàng bước vào thế giới kỳ diệu của Agent chưa? Hãy cùng lên đường ngay thôi!

## Kế hoạch tiếp theo

- Các khóa học video sẽ lần lượt được phát hành (sẽ chi tiết hơn, các buổi thực hành sẽ dẫn dắt mọi người từ ý tưởng thiết kế đến triển khai, vừa trao "con cá" vừa trao "cần câu")
- Framework HelloAgents đã cập nhật lên phiên bản V1.0.0, sẽ tiếp tục được hoàn thiện, bổ sung thêm nhiều công cụ và tính năng hữu ích, nhẹ (lightweight), đồng thời tương thích với phiên bản học tập.
- Tháng 5 năm 2026 sẽ cập nhật lộ trình học tập Agent mới nhất, [nhấn vào đây](https://github.com/datawhalechina/Agent-Learning-Hub) để xem
- Cảm ơn mọi người đã góp sức giúp dự án đạt 70,000 Star (7W Star)! Hiện tại chúng tôi cung cấp một bảng khảo sát để mọi người điền những nội dung Agent mà mình muốn học. Tác phẩm tiếp theo 《Huấn luyện Agent từ con số 0》 sẽ giúp mỗi người học nắm vững khả năng huấn luyện mô hình Agent cho các kịch bản tùy chỉnh (custom scenario) từ 0 đến 1.
<div align='center'>
    <img src="./读者反馈问卷.png" alt="Phiếu khảo sát phản hồi độc giả" width="30%">
    <p>Quét mã QR để điền ý kiến phản hồi, góp sức cùng phát triển dự án mới</p>
</div>

## 🤝 Cách đóng góp

Chúng tôi là một cộng đồng mã nguồn mở cởi mở, hoan nghênh mọi hình thức đóng góp!

- 🐛 <strong>Báo cáo Bug</strong> - Nếu phát hiện vấn đề về nội dung hoặc code, vui lòng gửi Issue
- 💡 <strong>Đề xuất ý kiến</strong> - Nếu có ý tưởng hay cho dự án, hãy mở một cuộc thảo luận
- 📝 <strong>Hoàn thiện nội dung</strong> - Giúp cải thiện tutorial, gửi Pull Request của bạn
- ✍️ <strong>Chia sẻ thực hành</strong> - Chia sẻ ghi chú học tập và dự án của bạn trong mục "Tuyển chọn đóng góp cộng đồng"

## 🙏 Lời cảm ơn

### Người đóng góp cốt lõi
- [陈思州 - Trưởng dự án](https://github.com/jjyaoao) (Thành viên Datawhale, viết và hiệu đính toàn văn) <a href="https://www.xiaohongshu.com/user/profile/67b9cc34000000000e013517" target="_blank"><img alt="Static Badge" src="https://img.shields.io/badge/Rednote-小红书-e93c49"></a>
- [孙韬 - Đồng sáng lập](https://github.com/fengju0213) (Thành viên Datawhale, CAMEL-AI, nội dung và hiệu đính Chương 9)  
- [姜舒凡 - Đồng sáng lập](https://github.com/Tsumugii24)（Thành viên Datawhale, thiết kế bài tập chương và hiệu đính）
- [黄佩林 - Thành viên tiềm năng của Datawhale](https://github.com/HeteroCat) (Kỹ sư phát triển Agent, người đóng góp nội dung Chương 5)
- [曾鑫民 - Kỹ sư Agent](https://github.com/fancyboi999) (Công ty Nowcoder, phát triển case Chương 14)
- [胡昊 - Người tổ chức vận hành](https://github.com/ACGpp) (Thành viên Datawhale)
- [朱信忠 - Chuyên gia hướng dẫn](https://xinzhongzhu.github.io/) (Nhà khoa học trưởng của Datawhale - Giáo sư tại Viện nghiên cứu Trí tuệ nhân tạo Hàng Châu, Đại học Sư phạm Chiết Giang)
### Người đóng góp Extra-Chapter
- [WH](https://github.com/WHQAQ11) (Người đóng góp nội dung)
- [周奥杰 - Đội ngũ đóng góp DW](https://github.com/thunderbolt-fire) (Đại học Giao thông Tây An, đóng góp nội dung Extra02)
- [张宸旭 - Nhà phát triển cá nhân](https://github.com/Tasselszcx)(Imperial College London, đóng góp nội dung Extra03)
- [黄宏晗 - Đội ngũ đóng góp DW](https://github.com/XiaoMa-PM) (Đại học Thâm Quyến, đóng góp nội dung Extra04)
- [王大鹏 - Thành viên Datawhale](https://github.com/ditingdapeng) (Kỹ sư R&D cao cấp, đóng góp nội dung Extra08)
- [尤逸晖 - Nhà phát triển cá nhân](https://github.com/YYHDBL) (Đại học Thông tin Kỹ thuật Nam Kinh, đóng góp nội dung Extra09)
- [殷鑫 - Nhà phát triển cá nhân](https://github.com/vinci-grape) (Đại học Chiết Giang, đóng góp nội dung Extra10)
- [Pranav Janakiraman - Nhà phát triển cá nhân](https://github.com/pranavjana) (TinyFish, đóng góp nội dung Extra11)
- [王雨飞 - Nhà phát triển cá nhân](https://github.com/nameless0120) (Đại học Bưu điện Bắc Kinh，đóng góp nội dung Extra12)

### Người đóng góp khóa học video

- 范丽（Giảng giải video Chương 4: ReAct, Plan-and-Solve và Reflection）
- 西芹百合（Giảng giải video Chương 5: Coze）
- 长庚（Thực hành theo video Chương 5 - Mục 2: Coze）
- 高鸿飞（Thực hành theo video Chương 6 - Mục 5: LangGraph）
- [柠檬酸（不吃柠檬🍋）](https://github.com/HnuLemons)（Giảng giải video và slide bài giảng Chương 8 - Mục 1, 2）
- [楚客（秦川楚客）](https://www.xiaohongshu.com/user/profile/643f4844000000002a009e81)（Giảng giải và thực hành Context Engineering Chương 9）
- 潘钲日（Giảng giải và thực hành Chương 14: Agent nghiên cứu chuyên sâu tự động hóa）

Liên kết video và tài liệu đi kèm xem tại [Đồng sáng tạo ghi hình khóa học video Hello-Agents](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra13-Hello-Agents视频课录制共创.md)。

### Lời cảm ơn đặc biệt
- Cảm ơn [@Sm1les](https://github.com/Sm1les) đã giúp đỡ và hỗ trợ dự án này
- Cảm ơn tất cả các nhà phát triển đã đóng góp cho dự án này ❤️

<div align=center style="margin-top: 30px;">
  <a href="https://github.com/datawhalechina/Hello-Agents/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=datawhalechina/Hello-Agents" />
  </a>
</div>

## Star History

<div align='center'>
    <img src="./docs/images/0920-hello-agents-stats.png" alt="Datawhale" width="90%">
</div>

<div align="center">
  <p>⭐ Nếu dự án này hữu ích với bạn, hãy cho chúng tôi một Star!</p>
</div>

## Nhóm trao đổi độc giả

<div align='center'>
    <img src="./读者群二维码.png" alt="Mã QR nhóm độc giả" width="30%">
    <p>Quét mã QR để tham gia nhóm trao đổi độc giả, cùng trao đổi thảo luận với nhiều người học khác</p>
</div>

## Về Datawhale

<div align='center'>
    <img src="./docs/images/datawhale.png" alt="Datawhale" width="30%">
    <p>Quét mã QR để theo dõi tài khoản công khai (official account) Datawhale, nhận thêm nhiều nội dung mã nguồn mở chất lượng</p>
</div>

## 🎓 Trích dẫn

Nếu Hello-Agents hữu ích cho nghiên cứu hoặc công việc của bạn, hãy trích dẫn:

```bibtex
@misc{hello_agents2025,
  title  = {Hello-Agents: Building an AI Agent from Scratch},
  author = {Sizhou Chen and Tao Sun and Shufan Jiang and Peilin Huang and Xinmin Zeng and Hao Hu and Xinzhong Zhu and all Hello-Agents contributors},
  year   = {2025},
  url    = {https://github.com/datawhalechina/Hello-Agents},
  note   = {GitHub repository}
}
```

---

## 📜 Giấy phép mã nguồn mở

Tác phẩm này được cấp phép theo [Giấy phép quốc tế Creative Commons Attribution-NonCommercial-ShareAlike 4.0 (CC BY-NC-SA 4.0)](http://creativecommons.org/licenses/by-nc-sa/4.0/).
