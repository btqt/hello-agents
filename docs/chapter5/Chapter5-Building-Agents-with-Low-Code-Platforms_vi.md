# Chương 5: Xây dựng Agent dựa trên nền tảng Low-code

Trong chương trước, bằng cách viết code Python, chúng ta đã triển khai từ đầu nhiều workflow Agent kinh điển như ReAct, Plan-and-Solve và Reflection. Quá trình này đã giúp chúng ta xây dựng được nền tảng kỹ thuật vững chắc, hiểu sâu sắc về cơ chế vận hành bên trong của Agent. Tuy nhiên, đối với một lĩnh vực phát triển nhanh chóng, mô hình phát triển thuần code không phải lúc nào cũng là lựa chọn hiệu quả nhất, đặc biệt là trong các tình huống cần nhanh chóng xác thực ý tưởng, hoặc khi các nhà phát triển không chuyên nghiệp muốn tham gia xây dựng.

## 5.1 Sự trỗi dậy của việc xây dựng theo hướng nền tảng (Platform-based)

Cùng với sự trưởng thành của công nghệ, chúng ta thấy ngày càng nhiều năng lực đang được "nền tảng hóa". Giống như việc phát triển website đã tiến hóa từ viết tay HTML/CSS/JS sang sử dụng các nền tảng dựng web như WordPress, Wix, việc xây dựng Agent cũng đang đón nhận làn sóng nền tảng hóa. Chương này sẽ tập trung vào cách tận dụng các nền tảng low-code trực quan, theo mô-đun để nhanh chóng, trực quan xây dựng, debug và triển khai các ứng dụng Agent, chuyển trọng tâm của chúng ta từ "chi tiết triển khai" sang "logic nghiệp vụ".

### 5.1.1 Vì sao cần nền tảng Low-code

"Tái tạo lại bánh xe" (reinventing the wheel) rất quan trọng đối với việc học chuyên sâu, nhưng trong thực tiễn theo đuổi hiệu suất kỹ thuật và sự đổi mới, chúng ta thường cần đứng trên vai những người khổng lồ. Mặc dù ở Chương 4 chúng ta đã đóng gói các class có thể tái sử dụng như `ReActAgent`, `PlanAndSolveAgent`, nhưng khi logic nghiệp vụ trở nên phức tạp, chi phí bảo trì và chu kỳ phát triển của mã thuần (pure code) sẽ tăng vọt. Sự xuất hiện của các nền tảng low-code chính là để giải quyết những điểm nghẽn (pain points) này.

Giá trị cốt lõi của nó chủ yếu thể hiện ở các khía cạnh sau:

1. <strong>Hạ thấp rào cản kỹ thuật</strong>: Nền tảng low-code đóng gói các chi tiết kỹ thuật phức tạp (như gọi API, quản lý trạng thái, kiểm soát đồng thời) thành từng "node" hoặc "module" dễ hiểu. Người dùng không cần thông thạo lập trình, chỉ cần kéo thả, kết nối các node này là có thể xây dựng nên những workflow mạnh mẽ. Điều này giúp cho product manager, designer, chuyên gia nghiệp vụ và những nhân sự không chuyên về kỹ thuật khác cũng có thể tham gia vào việc thiết kế và sáng tạo Agent, mở rộng đáng kể ranh giới của sự đổi mới.
2. <strong>Nâng cao hiệu quả phát triển</strong>: Đối với các nhà phát triển chuyên nghiệp, nền tảng cũng mang lại sự cải thiện hiệu quả to lớn. Ở giai đoạn đầu của dự án, khi cần nhanh chóng xác thực một ý tưởng hoặc dựng một Prototype, việc sử dụng nền tảng low-code có thể hoàn thành công việc vốn cần nhiều ngày code chỉ trong vài giờ, thậm chí vài phút. Nhà phát triển có thể dồn nhiều tâm sức hơn vào việc hệ thống hóa logic nghiệp vụ và tối ưu hóa prompt engineering, thay vì triển khai kỹ thuật ở tầng thấp.
3. <strong>Cung cấp khả năng trực quan hóa và khả năng quan sát (observability) tốt hơn</strong>: So với việc in log ra terminal, nền tảng đồ họa vốn dĩ cung cấp khả năng trực quan hóa end-to-end đối với quỹ đạo vận hành của Agent. Bạn có thể thấy rõ dữ liệu luân chuyển giữa từng node như thế nào, khâu nào tốn thời gian nhất, công cụ nào gọi bị lỗi. Trải nghiệm debug trực quan này là điều mà phát triển thuần code khó có thể sánh kịp.
4. <strong>Chuẩn hóa và tích lũy best practice</strong>: Các nền tảng low-code tốt thường tích hợp sẵn nhiều best practice trong ngành. Ví dụ, chúng cung cấp sẵn template ReAct, công cụ truy xuất knowledge base được tối ưu hóa, quy chuẩn tích hợp công cụ được chuẩn hóa, v.v. Điều này không chỉ giúp nhà phát triển tránh "dính bẫy", mà còn khiến việc hợp tác nhóm trở nên trôi chảy hơn, vì tất cả mọi người đều phát triển dựa trên cùng một bộ chuẩn và component.

Nói ngắn gọn, nền tảng low-code không phải để thay thế code, mà là cung cấp một tầng trừu tượng cao hơn. Nó giúp chúng ta thoát khỏi việc triển khai tầng thấp rườm rà, tập trung hơn vào chính logic "suy nghĩ" và "hành động" của Agent, từ đó biến ý tưởng thành hiện thực nhanh hơn và tốt hơn.

### 5.1.2 Lựa chọn nền tảng Low-code

Hiện tại, thị trường nền tảng low-code cho Agent và ứng dụng LLM đang trong giai đoạn "trăm hoa đua nở", mỗi nền tảng đều có vị thế và ưu thế riêng. Việc chọn nền tảng nào thường phụ thuộc vào nhu cầu cốt lõi, nền tảng kỹ thuật và mục tiêu cuối cùng của dự án của bạn. Trong phần nội dung tiếp theo của chương này, chúng ta sẽ giới thiệu và thực hành với một số nền tảng tiêu biểu: Coze, Dify, FastGPT và n8n. Trước khi đi vào chi tiết, chúng ta hãy cùng điểm qua một cách khái quát về chúng.

<strong>Coze</strong>

- <strong>Định vị cốt lõi</strong>: Coze<sup>[1]</sup> do ByteDance ra mắt, chủ trương trải nghiệm xây dựng Agent không cần code/low-code, giúp người dùng không có nền tảng lập trình cũng có thể dễ dàng tạo ra Agent.
- <strong>Phân tích đặc điểm</strong>: Coze sở hữu giao diện trực quan cực kỳ thân thiện, người dùng có thể như đang xếp lego, thông qua kéo thả plugin, cấu hình knowledge base và thiết lập workflow để tạo ra Agent. Nền tảng tích hợp sẵn kho plugin cực kỳ phong phú, và hỗ trợ xuất bản một chạm (one-click publish) lên nhiều nền tảng chủ lưu như Douyin, Feishu, WeChat Official Account, giúp đơn giản hóa đáng kể quy trình phân phối.
- <strong>Đối tượng phù hợp</strong>: Người dùng mới làm quen với ứng dụng AI, product manager, nhân viên vận hành, và các cá nhân sáng tạo muốn nhanh chóng biến ý tưởng thành sản phẩm có thể tương tác.

<strong>Dify</strong>

- <strong>Định vị cốt lõi</strong>: Dify là một nền tảng phát triển và vận hành ứng dụng LLM mã nguồn mở, toàn diện<sup>[2]</sup>, nhằm cung cấp giải pháp trọn gói từ xây dựng Prototype đến triển khai production cho nhà phát triển.
- <strong>Phân tích đặc điểm</strong>: Nó kết hợp giữa dịch vụ backend và tư duy vận hành mô hình, hỗ trợ Agent workflow, RAG Pipeline, gán nhãn dữ liệu và fine-tuning cùng nhiều năng lực khác. Đối với các ứng dụng cấp doanh nghiệp đòi hỏi tính chuyên nghiệp, ổn định, có thể mở rộng, Dify cung cấp một nền tảng vững chắc.
- <strong>Đối tượng phù hợp</strong>: Nhà phát triển có nền tảng kỹ thuật nhất định, các nhóm cần xây dựng ứng dụng AI cấp doanh nghiệp có khả năng mở rộng.

<strong>FastGPT</strong>

- <strong>Định vị cốt lõi</strong>: FastGPT là một nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở dựa trên LLM<sup>[3]</sup>, tập trung cung cấp giải pháp RAG (Retrieval-Augmented Generation) dễ sử dụng và khả năng sắp xếp workflow trực quan.
- <strong>Phân tích đặc điểm</strong>: Ưu thế cốt lõi nhất của FastGPT nằm ở việc tối ưu hóa đến mức cực hạn cho kịch bản hỏi đáp knowledge base. Nó cung cấp một chuỗi RAG hoàn chỉnh từ nhập dữ liệu, tự động phân đoạn văn bản (chunking), lưu trữ vector hóa đến truy xuất thông minh, và hỗ trợ sắp xếp các luồng hội thoại cũng như workflow Agent phức tạp thông qua giao diện trực quan (module Flow). Nền tảng được thiết kế trung lập về mô hình, có thể linh hoạt kết nối với OpenAI, Claude, Tongyi Qianwen và nhiều mô hình lớn chủ lưu trong và ngoài nước khác, đồng thời cung cấp API hoàn chỉnh và chợ plugin, thuận tiện cho việc tích hợp nhanh với các hệ thống hiện có như WeCom, DingTalk, Feishu.
- <strong>Đối tượng phù hợp</strong>: Các nhà phát triển và đội nhóm doanh nghiệp vừa và nhỏ muốn nhanh chóng xây dựng trợ lý chăm sóc khách hàng thông minh, trợ lý tri thức nội bộ doanh nghiệp, chatbot hỏi đáp tài liệu dựa trên knowledge base riêng, cũng như những người đam mê kỹ thuật quan tâm đến RAG nhưng muốn hạ thấp rào cản triển khai.

<strong>n8n</strong>

- <strong>Định vị cốt lõi</strong>: n8n về bản chất là một công cụ tự động hóa workflow mã nguồn mở<sup>[4]</sup>, chứ không phải một nền tảng LLM thuần túy. Những năm gần đây, nó đã tích cực tích hợp năng lực AI.
- <strong>Phân tích đặc điểm</strong>: Thế mạnh của n8n nằm ở khả năng "kết nối". Nó sở hữu hàng trăm node dựng sẵn, có thể dễ dàng kết nối các loại dịch vụ SaaS, database, API thành các quy trình nghiệp vụ tự động hóa phức tạp. Bạn có thể nhúng node LLM vào trong quy trình này, khiến nó trở thành một mắt xích trong toàn bộ chuỗi tự động hóa. Mặc dù về mức độ chuyên sâu của tính năng LLM không bằng hai nền tảng trên, nhưng năng lực tự động hóa tổng quát của nó là độc nhất vô nhị. Tuy nhiên, đường cong học tập (learning curve) của nó cũng tương đối dốc.
- <strong>Đối tượng phù hợp</strong>: Các nhà phát triển và doanh nghiệp cần tích hợp sâu năng lực AI vào quy trình nghiệp vụ hiện có, thực hiện tự động hóa có mức độ tùy biến cao.

Trong các mục tiếp theo, chúng ta sẽ lần lượt trải nghiệm thực tế các nền tảng này, thông qua thao tác thực tế để cảm nhận trực quan hơn sức hút riêng của từng nền tảng.

## 5.2 Nền tảng một: Coze ([coze.com](https://www.coze.com/home))

Coze (扣子) là một công cụ tạo Agent AI cực kỳ "ngầu"! Đây cũng là nền tảng Agent được sử dụng rộng rãi nhất trên thị trường hiện nay. Nền tảng này với giao diện trực quan và các module tính năng phong phú, giúp người dùng dễ dàng tạo ra các loại ứng dụng Agent khác nhau, ví dụ như chatbot có thể trò chuyện cùng bạn, máy tự động viết truyện, thậm chí trực tiếp giúp bạn biến câu chuyện thành MV phim! Một điểm sáng lớn của nó nằm ở khả năng tích hợp hệ sinh thái mạnh mẽ. Agent sau khi phát triển xong có thể xuất bản một chạm lên WeChat, Feishu, Doubao và các nền tảng chủ lưu khác, thực hiện triển khai liền mạch xuyên nền tảng. Đối với người dùng doanh nghiệp, Coze còn cung cấp API linh hoạt, hỗ trợ tích hợp năng lực Agent vào hệ thống nghiệp vụ hiện có, hiện thực hóa việc xây dựng ứng dụng AI theo kiểu "xếp hình".

### 5.2.1 Các module tính năng của Coze

(1) Khái quát giao diện nền tảng

Giới thiệu bố cục tổng thể: Gần đây Coze lại một lần nữa cập nhật giao diện UI của mình, như Hình 5.1. Hiện tại thanh bên ngoài cùng bên trái là khu vực làm việc trang chủ của nền tảng Coze, bao gồm phát triển dự án cốt lõi, kho tài nguyên (resource library), đánh giá hiệu quả (effect evaluation) và cấu hình không gian (space configuration). Khu vực phía dưới là không gian tài liệu đi kèm cho việc phát triển trên Coze, bao gồm sao chép template chính thức bằng một chạm, chợ plugin — ưu thế lớn nhất và phong phú nhất của Coze, cộng đồng Agent lớn nhất với vô vàn lựa chọn, quản lý API dùng để test API, cùng với tài liệu hướng dẫn chi tiết và quản lý chung dành cho doanh nghiệp. Khu vực bên phải có bốn template, trên cùng là thông báo cập nhật mới nhất của Coze, cho bạn biết tiến triển mới nhất của Coze để tiện theo dõi công cụ và tính năng mới. Tiếp theo phía dưới là hướng dẫn cho người mới, click vào là tài liệu hướng dẫn cho người mới, giúp bạn bắt đầu xây dựng Agent trong chớp mắt. Kế đến là mục bạn đang theo dõi và Agent được đề xuất, tại đây bạn cũng có thể theo dõi những nhà phát triển AI yêu thích và lưu lại Agent của họ để sử dụng cho riêng mình.

<div align="center">
  <img src="images/coze-01.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.1 Sơ đồ tổng thể nền tảng Agent thông minh Coze</p>
</div>

(2) Giới thiệu tính năng cốt lõi

Trước tiên, chúng ta click vào dấu cộng ở thanh bên trái là sẽ thấy lối vào để tạo Agent, hiện tại có hai loại ứng dụng AI, một là tạo Agent, hai là gọi là Application. Trong đó Agent lại chia thành chế độ tự lên kế hoạch cho Agent đơn (single-agent autonomous planning mode), chế độ luồng hội thoại cho Agent đơn (single-agent dialogue flow mode) và chế độ đa Agent (multi-agent mode). Ứng dụng AI cũng chia thành hai loại, không chỉ có thể thiết kế giao diện người dùng cho desktop/web, mà còn có thể dễ dàng dựng giao diện cho mini-app và H5, như Hình 5.2.

<div align="center">
  <img src="images/coze-02.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.2 Lối vào tạo Agent của Coze</p>
</div>

Không gian dự án (project space) là kho Agent của bạn, nơi lưu trữ tất cả các Agent/Application mà bạn đã phát triển hoặc sao chép, cũng là nơi bạn thường xuyên ghé thăm nhất khi phát triển Agent trên Coze, như Hình 5.3.
<div align="center">
  <img src="images/coze-03.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.3 Không gian dự án Agent của Coze</p>
</div>
Kho tài nguyên (resource library) là vũ khí cốt lõi để bạn phát triển Agent trên Coze, kho tài nguyên sẽ lưu trữ workflow, knowledge base, card, kho prompt và một loạt công cụ phát triển Agent khác của bạn. Agent bạn tạo ra mạnh đến đâu trước tiên phụ thuộc vào năng lực của mô hình, nhưng quan trọng nhất vẫn là bạn "trang bị kỹ năng" cho Agent như thế nào. Mô hình quyết định giới hạn dưới của Agent, nhưng kho tài nguyên của Coze mang lại giới hạn trên vô tận cho năng lực của Agent, cho phép bạn phát triển Agent theo ý tưởng, trí tưởng tượng và sự sáng tạo của riêng mình, như Hình 5.4.
<div align="center">
  <img src="images/coze-04.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.4 Kho tài nguyên Agent của Coze</p>
</div>
Cấu hình không gian (space configuration) bao gồm kênh quản lý thống nhất cho Agent, plugin, workflow và kênh xuất bản, cùng với quản lý mô hình — nơi bạn có thể xem các mô hình lớn mà mình đang gọi, như Hình 5.5.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-05.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.5 Kênh xuất bản Agent của Coze</p>
</div>
Nếu để tôi tổng kết đơn giản về việc phát triển Agent trên Coze, tôi sẽ ví nó như các thành phần của một trò chơi, các phần phối hợp với nhau tạo nên từng Agent xuất sắc, giống hệt như đang "chơi game", mỗi lần hoàn thành một Agent giống như vừa đánh bại một con boss và thu hoạch đầy ắp, dù là "kinh nghiệm" hay "trang bị".

- Workflow: Bản đồ lộ trình thông quan màn chơi
- Luồng hội thoại (Dialogue flow): Hội thoại thông quan với NPC
- Plugin: Thẻ kỹ năng nhân vật
- Knowledge base: Bách khoa toàn thư trong game
- Card: Ô vật phẩm nhanh
- Prompt: Phím di chuyển của nhân vật
- Database: "Lưu trữ đám mây"
- Quản lý xuất bản: Người kiểm duyệt màn chơi
- Quản lý mô hình: Kho nhân vật game hay còn gọi là hệ thống chỉnh sửa khuôn mặt
- Đánh giá hiệu quả: Hệ thống chấm điểm vượt ải

### 5.2.2 Xây dựng trợ lý "Bản tin AI hằng ngày"

<strong>Mô tả case study:</strong> Thực hành này nhằm phân tích sâu khả năng tích hợp plugin của nền tảng Coze, hướng dẫn người đọc xây dựng từ đầu một Agent "Bản tin AI hằng ngày" mạnh mẽ. Agent này có thể tự động thu thập tin tức nổi bật nhất trong ngày, các bài báo học thuật và động thái dự án mã nguồn mở trong lĩnh vực AI từ nhiều nguồn thông tin (bao gồm 36Kr, Huxiu, ITHome, InfoQ, GitHub, arXiv), rồi tích hợp chúng thành một bản tin sinh động, súc tích, chuyên nghiệp và có cấu trúc rõ ràng.

Thông qua case study này, bạn sẽ nắm được một cách hệ thống các kỹ năng cốt lõi sau:

* <strong>Tổng hợp thông tin đa nguồn:</strong> Tận dụng hệ sinh thái plugin của Coze, hiện thực hóa việc tích hợp liền mạch dữ liệu xuyên nền tảng, xuyên loại hình.
* <strong>Định nghĩa hành vi Agent:</strong> Thông qua thiết lập vai trò (role) và prompt engineering, kiểm soát chính xác việc thực thi nhiệm vụ và sinh nội dung của Agent, đảm bảo output phù hợp với tiêu chuẩn chuyên nghiệp đã đặt ra.
* <strong>Xây dựng workflow tự động hóa:</strong> Học cách kết nối nhiều bước như lấy dữ liệu, xử lý nội dung và định dạng output thành một workflow hiệu quả, tự động hóa.

<strong>Lưu ý: Cập nhật lối vào nền tảng Coze</strong>

Do nền tảng Coze liên tục cập nhật, lối vào giao diện của phiên bản hiện tại đã có thay đổi so với thời điểm viết hướng dẫn.

Khi truy cập [trang chủ Coze trong nước](https://www.coze.cn/home), trang có thể tự động chuyển hướng đến trang chủ phiên bản mới https://www.coze.cn . Trong giao diện phiên bản mới, có thể truy cập vào không gian làm việc (workspace) phiên bản cũ thông qua lối vào như sau để tiếp tục xây dựng Agent, như Hình 5.6.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-06.jpg" alt="Lối vào không gian làm việc phiên bản cũ từ trang chủ phiên bản mới của Coze" width="90%"/> 
  <p>Hình 5.6 Lối vào không gian làm việc phiên bản cũ từ trang chủ phiên bản mới của Coze</p>
</div>

Nếu trang không hiển thị lối vào tương tự, bạn cũng có thể truy cập trực tiếp [https://www.coze.com/space/](https://www.coze.com/space/) để vào workspace và tạo Agent.

<strong>Bước 1: Tạo workflow</strong>

Trong giao diện kho tài nguyên, chọn `+资源` (+ Tài nguyên) — `工作流` (Workflow), tạo workflow, như Hình 5.7.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-07.png" alt="Trang tạo workflow" width="90%"/> 
  <p>Hình 5.7 Trang tạo workflow</p> 
</div>

<strong>Bước 2: Thêm và cấu hình plugin nguồn thông tin</strong>

Nhiệm vụ hàng đầu để xây dựng Agent "Bản tin AI hằng ngày" là kết nối cho nó các nguồn thông tin phong phú và đáng tin cậy. Trong nền tảng Coze, việc này được hiện thực hóa thông qua việc thêm và cấu hình các plugin tương ứng.

1. <strong>Tích hợp plugin:</strong> Trong kho plugin của Coze, tìm kiếm và thêm các plugin cần thiết. Ví dụ, thông qua plugin <strong>RSS</strong> để đăng ký nguồn RSS của các nền tảng truyền thông (như Hình 5.8), thông qua plugin <strong>GitHub</strong> để theo dõi các dự án mã nguồn mở (như Hình 5.9), và thông qua plugin <strong>arXiv</strong> để lấy các kết quả nghiên cứu học thuật mới nhất (như Hình 5.10).

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-08.png" alt="Plugin nguồn RSS của các nền tảng truyền thông" width="90%"/> 
  <p>Hình 5.8 Plugin nguồn RSS của các nền tảng truyền thông</p> 
</div>

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-09.png" alt="Plugin GitHub" width="90%"/> 
  <p>Hình 5.9 Plugin GitHub</p> 
</div>

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-10.png" alt="Plugin Arxiv" width="90%"/> 
  <p>Hình 5.10 Plugin Arxiv</p> 
</div>
2. <strong>Cấu hình cá nhân hóa:</strong> Cấu hình chi tiết cho từng plugin để đảm bảo nó có thể lấy chính xác dữ liệu cần thiết. Ví dụ, trong plugin RSS, nhập liên kết đăng ký RSS cụ thể của các trang như 36Kr, Huxiu; trong plugin GitHub, thiết lập số lượng truy vấn từ khóa cần theo dõi cùng cài đặt cập nhật mới nhất; trong plugin arXiv, định nghĩa từ khóa lĩnh vực quan tâm như "LLM", "AI", định nghĩa số lượng cùng cài đặt cập nhật mới nhất.

```
Cấu hình liên kết RSS

url: 36Kr: https://www.36kr.com/feed
url: Huxiu: https://rss.huxiu.com/
url: ITHome: http://www.ithome.com/rss/
url: InfoQ: https://feed.infoq.com/ai-ml-data-eng/

Cấu hình plugin GitHub

q:AI
per_page:10
sort:updated

Cấu hình plugin Arxiv

count: 5
search_query: AI
sort_by: 2
```

3. <strong>Sắp xếp kết nối:</strong> Trong giao diện sắp xếp trực quan (visual orchestration) của Agent, đưa các plugin nguồn thông tin đã cấu hình (ví dụ `rss_24Hbj`, `searchRepository`, `arxiv`, v.v.) làm các node đầu vào dữ liệu, và kết nối chúng với các module xử lý logic tiếp theo (ví dụ module <strong>Mô hình lớn</strong>), để xây dựng lộ trình xử lý dữ liệu hoàn chỉnh, như Hình 5.11.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-11.png" alt="Sơ đồ sắp xếp bản tin AI hằng ngày" width="90%"/> 
  <p>Hình 5.11 Sơ đồ sắp xếp bản tin AI hằng ngày</p> 
</div>

<strong>Bước 3: Thiết lập vai trò và prompt cho Agent</strong>

Thiết lập vai trò (role) và viết prompt là khâu cốt lõi định nghĩa hành vi và chất lượng output của Agent. Bước này nhằm chuyển hóa các chỉ thị trừu tượng thành nhiệm vụ cụ thể mà Agent có thể hiểu và thực thi.

Trong nền tảng Coze, có thể định nghĩa logic hành vi của Agent thông qua cấu hình System Prompt và User Prompt, như Hình 5.12.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-12.png" alt="Trang cấu hình prompt" width="90%"/> 
  <p>Hình 5.12 Trang cấu hình prompt</p> 
</div>

(1) Thiết lập vai trò

Chúng ta thiết lập Agent thành <strong>một biên tập viên truyền thông công nghệ dày dạn kinh nghiệm và có uy tín</strong>. Vai trò này trao cho Agent một định vị chuyên môn rõ ràng, giúp nó, trong quá trình sáng tạo nội dung tiếp theo, có thể mô phỏng theo tư duy của một biên tập viên chuyên nghiệp, thực hiện việc sàng lọc, tổng hợp và khái quát thông tin một cách hiệu quả.

(2) Viết và cấu trúc hóa prompt

Prompt là sổ tay hướng dẫn để Agent thực thi nhiệm vụ. Chúng ta chia nó thành <strong>System Prompt và User Prompt</strong>, để đảm bảo tính rõ ràng, đầy đủ và khả năng kiểm soát của chỉ thị.
<strong>System Prompt</strong>

```
# Vai trò
Bạn là một biên tập viên truyền thông công nghệ dày dạn kinh nghiệm và có uy tín, giỏi tổng hợp và sáng tạo một cách hiệu quả, chính xác các bản tin công nghệ có tính chuyên môn cao, đặc biệt có năng lực phân tích và tổng hợp chuyên sâu về các động thái kỹ thuật trong lĩnh vực AI, các kết quả nghiên cứu học thuật tiên phong và các dự án mã nguồn mở nổi bật.
## Workflow
### Định dạng đầu ra bản tin hằng ngày
1. Phần đầu bản tin phải ghi nổi bật "Bản tin AI hằng ngày", "by@jasonhuang" và ngày hôm đó, ví dụ: "Bản tin AI hằng ngày | 24/09/2025 | by@jasonhuang".
2. <!!!important!!!> Dựa theo nội dung khác nhau của từng tin tức công nghệ AI, từng bài báo học thuật AI, từng dự án mã nguồn mở AI, hãy thêm một biểu tượng Emoji riêng ở đầu tiêu đề của mục đó.
3. Mọi nội dung đầu ra phải liên quan mật thiết đến các chủ đề kỹ thuật như AI, LLM, AIGC, mô hình lớn, kiên quyết loại bỏ mọi thông tin không liên quan, nội dung quảng cáo và tiếp thị.
4. Bắt buộc phải cung cấp đường dẫn gốc tương ứng cho từng mục (bao gồm tin tức công nghệ AI, bài báo học thuật AI, dự án mã nguồn mở AI).
5. Với mỗi tin tức hoặc dự án được xuất ra, hãy viết một mô tả khái quát ngắn gọn và chính xác.
```
System Prompt dùng để định nghĩa quy tắc hành vi dài hạn và chuẩn định dạng output của Agent. User Prompt dùng để định nghĩa chỉ thị nhiệm vụ cụ thể và nguồn dữ liệu.
<strong>User Prompt</strong>

```
- **Trích xuất và tổng hợp thông tin:** Từ các nguồn đầu vào `{{articles}}`, `{{articles1}}`, `{{articles2}}` và `{{articles3}}`, hãy sàng lọc và trích xuất tiêu đề bài viết cùng đường dẫn tương ứng liên quan đến các chủ đề như AI, mô hình lớn, AIGC, LLM, v.v., rồi sắp xếp thành module **"Tin tức công nghệ AI"**.
- **Tóm tắt bài báo học thuật:** Từ nguồn đầu vào `{{arxiv}}`, dựa theo các trường `arxiv_title` và `arxiv_link`, hãy tóm tắt và sắp xếp nội dung các bài báo mới nhất, tạo thành module **"Bài báo học thuật AI"**.
- **Sàng lọc dự án mã nguồn mở:** Từ nguồn đầu vào `{{GitHub}}`, hãy sàng lọc **5 dự án mã nguồn mở AI** được chú ý nhất và có tầm ảnh hưởng lớn nhất. Trích xuất tiêu đề cùng đường dẫn tương ứng của các dự án này, sắp xếp thành module **"Dự án mã nguồn mở AI"**.
# Lưu ý (Attention)
- Tuân thủ nghiêm ngặt định dạng đầu ra bản tin hằng ngày đã định nghĩa trong System Prompt.
- Tổng lượng nội dung đầu ra phải là: **10 tin tức công nghệ AI, 5 bài báo học thuật AI, 5 dự án mã nguồn mở AI**.
```

<strong>Bước 4: Kiểm thử, debug và xuất bản đa kênh</strong>

Sau khi hoàn thành việc xây dựng logic cốt lõi của Agent, cần phải tiến hành kiểm thử (test) và debug nghiêm ngặt để đảm bảo output đúng như kỳ vọng.

<strong>Chạy thử (Preview):</strong> Chạy Agent trong giao diện preview của nền tảng Coze, quan sát nội dung bản tin được sinh ra.

```
# Bản tin AI hằng ngày by@jasonhuang 2025-09-24

## 🚀 Tin tức công nghệ AI

🤖 **Zhiyuan Robotics (AgiBot) mã nguồn mở hoàn toàn mô hình nền tảng hiện thân đa dụng GO-1**
Liên kết: https://36kr.com/p/3479085489708163?f=rss
Khái quát: Zhiyuan Robotics công bố mã nguồn mở hoàn toàn mô hình nền tảng hiện thân đa dụng GO-1, cung cấp năng lực AI nền tảng mạnh mẽ cho lĩnh vực robot.

🔬 **Microsoft khắc phục nút thắt tản nhiệt chip trung tâm dữ liệu: Vi lưu (Microfluidics) + AI làm mát chính xác**
Liên kết: https://www.ithome.com/0/885/391.htm
Khái quát: Microsoft kết hợp công nghệ vi lưu và thuật toán AI để kiểm soát chính xác nhiệt độ chip tại trung tâm dữ liệu, nâng cao hiệu suất năng lượng.
......

## 📚 Bài báo học thuật AI

🧪 **Lyra: Generative 3D Scene Reconstruction via Video Diffusion Model Self-Distillation**
Liên kết: http://arxiv.org/pdf/2509.19296v1
Khái quát: Đề xuất framework sáng tạo giúp tái tạo cảnh 3D thông qua tự chưng cất (self-distillation) mô hình khuếch tán video, không cần dữ liệu huấn luyện đa góc nhìn.

📊 **The ICML 2023 Ranking Experiment: Examining Author Self-Assessment in ML/AI Peer Review**
Liên kết: http://arxiv.org/pdf/2408.13430v3
Khái quát: Nghiên cứu tính hiệu quả của việc tự đánh giá từ tác giả trong quy trình bình duyệt tại hội nghị machine learning, đề xuất phương pháp cải thiện cơ chế bình duyệt.
......

## 💻 Dự án mã nguồn mở AI

🤖 **llmling-agent - Framework workflow đa Agent**
Liên kết: https://github.com/phil65/llmling-agent
Khái quát: Framework tương tác đa Agent hỗ trợ cấu hình YAML và lập trình, tích hợp hỗ trợ giao thức MCP và ACP.

🚌 **College_EV_AI_Transportation - Hệ thống giao thông điện AI trong khuôn viên trường**
Liên kết: https://github.com/LuisMc2005v/College_EV_AI_Transportation
Khái quát: Hệ thống tối ưu hóa giao thông điện trong khuôn viên trường dựa trên AI, thực hiện theo dõi thời gian thực và dịch vụ đi chung xe hiệu quả.
......
```

Kiểm tra kỹ lưỡng độ chính xác nội dung, tính đầy đủ về định dạng cũng như phong cách ngôn ngữ của bản tin. Nếu phát hiện phần nào không đúng kỳ vọng, cần quay lại khâu prompt hoặc cấu hình plugin để điều chỉnh chi tiết. Ví dụ, nếu nội dung chưa đủ súc tích, có thể sửa yêu cầu khái quát trong prompt; nếu dữ liệu lấy về không chính xác, cần kiểm tra lại tham số cấu hình plugin.

Xuất bản đa kênh: Coze cung cấp khả năng xuất bản Agent một chạm lên nhiều nền tảng ứng dụng chủ lưu (như WeChat, Doubao, Feishu, v.v.), mở rộng đáng kể các tình huống ứng dụng của Agent, như Hình 5.13.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-13.png" alt="Các kênh xuất bản đa dạng của nền tảng Coze" width="60%"/> 
  <p>Hình 5.13 Các kênh xuất bản đa dạng của nền tảng Coze</p>
</div>

Sau khi Agent được xuất bản, chúng ta có thể thấy Agent AI vừa tạo trong cửa hàng Coze, đồng thời cũng có thể tích hợp nó vào ứng dụng AI để phục vụ người dùng, như Hình 5.14 và Hình 5.15. Tại đây cũng xin đính kèm [liên kết trải nghiệm Agent tin tức AI hằng ngày](https://www.coze.cn/store/agent/7506052197071962153?bot_id=true&bid=6hkt3je8o2g16)

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-14.png" alt="Agent AI - Tin tức AI hằng ngày" width="90%"/> 
  <p>Hình 5.14 Agent AI - Tin tức AI hằng ngày</p>
</div>

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-15.png" alt="Tin tức AI hằng ngày trong ứng dụng AI" width="90%"/> 
  <p>Hình 5.15 Tin tức AI hằng ngày trong ứng dụng AI</p>
</div>

<strong>Cấu hình xuất bản:</strong> Nếu muốn xuất bản Agent của riêng mình, trước khi xuất bản còn cần cấu hình tên gọi, avatar và lời chào phù hợp cho Agent, để mang lại trải nghiệm thân thiện hơn cho người dùng, như Hình 5.16 và Hình 5.17.

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-16.png" alt="Cấu hình thông tin cơ bản cho Agent" width="50%"/> 
  <p>Hình 5.16 Cấu hình thông tin cơ bản cho Agent</p>
</div>

<div align="center"> 
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-17.png" alt="Cấu hình lời mở đầu và câu hỏi gợi ý cho Agent" width="50%"/> 
  <p>Hình 5.17 Cấu hình lời mở đầu và câu hỏi gợi ý cho Agent</p>
</div>

### 5.2.3 Phân tích ưu điểm và hạn chế của Coze

<strong>Ưu điểm:</strong>

* <strong>Hệ sinh thái plugin mạnh mẽ:</strong> Ưu thế cốt lõi của nền tảng Coze nằm ở kho plugin phong phú, điều này giúp Agent dễ dàng kết nối với các dịch vụ và nguồn dữ liệu bên ngoài, từ đó đạt được khả năng mở rộng tính năng ở mức cao.
* <strong>Sắp xếp trực quan, dễ hiểu:</strong> Nền tảng cung cấp giao diện sắp xếp workflow trực quan với rào cản thấp, người dùng không cần có kiến thức lập trình sâu, vẫn có thể xây dựng workflow phức tạp bằng cách "kéo thả", giảm đáng kể độ khó phát triển.
* <strong>Kiểm soát prompt linh hoạt:</strong> Thông qua việc thiết lập vai trò chính xác và viết prompt, người dùng có thể kiểm soát hành vi và sinh nội dung của Agent ở mức độ chi tiết, đạt được output có tính tùy biến cao. Hơn nữa còn hỗ trợ quản lý prompt và template, giúp nhà phát triển rất thuận tiện trong việc xây dựng Agent.
* <strong>Triển khai đa nền tảng tiện lợi:</strong> Hỗ trợ xuất bản cùng một Agent lên các nền tảng ứng dụng khác nhau, hiện thực hóa việc tích hợp và ứng dụng liền mạch xuyên nền tảng. Hơn nữa Coze vẫn không ngừng tích hợp thêm các nền tảng mới vào hệ sinh thái của mình, ngày càng nhiều hãng điện thoại và nhà sản xuất phần cứng đang lần lượt hỗ trợ xuất bản Agent của Coze.

<strong>Hạn chế:</strong>

* <strong>Không hỗ trợ MCP:</strong> Tôi cho rằng đây là điểm chí mạng nhất, mặc dù chợ plugin của Coze cực kỳ phong phú và hấp dẫn. Nhưng việc không hỗ trợ MCP có thể trở thành gông cùm hạn chế sự phát triển của nó, nếu được mở khóa thì đó sẽ là một át chủ bài khác.
* <strong>Độ phức tạp khi cấu hình một số plugin cao:</strong> Đối với các plugin cần API Key hoặc tham số nâng cao khác, người dùng có thể cần có nền tảng kỹ thuật nhất định mới hoàn thành cấu hình chính xác. Việc sắp xếp workflow phức tạp cũng không phải điều mà người không có nền tảng gì cũng nắm được, cần có chút kiến thức về JS hoặc Python.
* <strong>Không thể trực tiếp import file JSON sắp xếp (orchestration):</strong> Trước đây Coze không có tính năng export/import, nhưng hiện tại phiên bản trả phí đã có thể export/import, tuy nhiên những gì export/import không phải là file JSON giống như Dify, n8n, mà là một file zip. Tức là bạn chỉ có thể export từ Coze rồi import lại vào Coze. Tuy nhiên nếu muốn "lách luật", bạn cũng có thể chọn copy orchestration, trong giao diện orchestration nhấn Ctrl+A chọn tất cả rồi Ctrl+C copy orchestration, sau đó dán vào một workflow trống khác hoặc workflow khác.

## 5.3 Nền tảng hai: Dify

### 5.3.1 Giới thiệu và hệ sinh thái của Dify

Dify là một nền tảng phát triển ứng dụng mô hình ngôn ngữ lớn (LLM) mã nguồn mở, kết hợp giữa tư duy Backend-as-a-Service (BaaS) và LLMOps, cung cấp hỗ trợ toàn bộ chu trình từ thiết kế Prototype đến triển khai production, như Hình 5.18. Nó áp dụng kiến trúc mô-đun phân tầng, chia thành tầng dữ liệu, tầng phát triển, tầng sắp xếp (orchestration) và tầng cơ sở, các tầng tách rời nhau giúp dễ mở rộng.

Dify có tính trung lập cao đối với mô hình và khả năng tương thích mạnh: dù là mô hình mã nguồn mở hay thương mại, người dùng đều có thể kết nối chúng chỉ với cấu hình đơn giản, và gọi năng lực suy luận của chúng thông qua giao diện thống nhất. Nó tích hợp sẵn hỗ trợ cho hàng trăm LLM mã nguồn mở hoặc độc quyền, bao gồm các mô hình GPT, Deepseek, Llama, cũng như bất kỳ mô hình nào tương thích với OpenAI API.

Đồng thời, Dify hỗ trợ cả triển khai local (chính thức cung cấp khởi động một chạm bằng Docker Compose) và triển khai cloud. Người dùng có thể chọn tự triển khai Dify trong môi trường local/riêng tư (đảm bảo quyền riêng tư dữ liệu), hoặc sử dụng dịch vụ SaaS cloud chính thức (mô hình kinh doanh sẽ được trình bày chi tiết ở phần dưới). Sự linh hoạt trong triển khai này khiến nó phù hợp với môi trường mạng nội bộ doanh nghiệp có yêu cầu bảo mật cao, hoặc nhóm nhà phát triển cần sự tiện lợi trong vận hành.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-01.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.18 Trang chủ chính thức của Dify</p>
</div>

Hệ sinh thái plugin Marketplace: Dify Marketplace cung cấp tính năng quản lý plugin tập trung và triển khai một chạm, giúp nhà phát triển có thể khám phá, mở rộng hoặc đóng góp plugin, mang lại nhiều khả năng hơn cho cộng đồng, như Hình 5.19.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-02.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.19 Hệ sinh thái plugin Dify Marketplace</p>
</div>
Marketplace bao gồm:

- Mô hình (Models)
- Công cụ (Tools)
- Chiến lược Agent (Agent Strategies)
- Mở rộng (Extensions)
- Gói (Bundles)

Hiện tại, Dify Marketplace đã có hơn 8677 plugin, bao gồm nhiều tính năng và kịch bản ứng dụng khác nhau. Trong đó, các plugin được chính thức đề xuất bao gồm:

- Google Search: langgenius/google
- Azure OpenAI: langgenius/azure_openai
- Notion: langgenius/notion
- DuckDuckGo: langgenius/duckduckgo

Dify cung cấp hỗ trợ phát triển mạnh mẽ cho các nhà phát triển plugin, bao gồm tính năng remote debug, có thể phối hợp liền mạch với các IDE phổ biến, chỉ cần thiết lập môi trường tối thiểu. Nhà phát triển có thể kết nối với dịch vụ SaaS của Dify, đồng thời chuyển tiếp mọi thao tác plugin về môi trường local để kiểm thử, cách tiếp cận thân thiện với nhà phát triển này nhằm trao quyền cho người tạo plugin và đẩy nhanh sự đổi mới trong hệ sinh thái Dify. Đây cũng là lý do Dify có thể trở thành một trong những nền tảng Agent thành công nhất hiện nay, bởi vì mô hình nào cũng có thể kết nối được, prompt và orchestration đều có thể sao chép được, nhưng việc có hay không có công cụ/plugin, và chúng có phong phú hay không, sẽ trực tiếp quyết định việc Agent của bạn có thể đạt hiệu quả tốt hơn hoặc có những tính năng mạnh mẽ bất ngờ hay không.

### 5.3.2 Xây dựng một "Siêu trợ lý Agent cá nhân"

> **✨✨ Hướng dẫn thao tác chi tiết**: Vui lòng tham khảo **[Quy trình thao tác chi tiết từng bước tạo Agent Dify](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra03-Dify智能体创建保姆级操作流程.md)**

Trong case study của Coze ở mục trước, chúng ta đã xây dựng một Agent bản tin AI hằng ngày. Mặc dù tính năng rõ ràng, nhưng khả năng sinh bản tin đơn nhất của nó hơi hạn chế. Mục này sẽ sử dụng Dify để xây dựng một "Siêu trợ lý Agent cá nhân" toàn diện về tính năng, bao gồm nhiều tình huống như hỏi đáp hằng ngày, tối ưu hóa văn bản, sinh nội dung đa phương thức (multimodal), phân tích dữ liệu. Trước khi bắt đầu, chúng ta hãy cùng tìm hiểu sơ lược về giao diện chính và các module tính năng của Dify.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-14.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.20 Trang chủ xây dựng Agent của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-18.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.21 Kho template chính thức của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-15.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.22 Knowledge base của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-16.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.23 Chợ plugin của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-17.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.24 Cấu hình mô hình lớn của Dify</p>
</div>

<strong>(1) Tạo plugin và cấu hình MCP</strong>

Trước khi xây dựng Agent, cần hoàn thành việc cài đặt plugin cần thiết và cấu hình MCP. Như Hình 5.25, đây là các plugin cốt lõi cần thiết cho case study này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-19.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.25 Cấu hình cài đặt plugin của Dify</p>
</div>

Các plugin được đánh dấu trong khung đỏ trong hình cần được tìm kiếm và cài đặt từ chợ plugin của Dify. Người dùng có thể click xem chi tiết để hiểu tính năng cụ thể của từng plugin.

Tiếp theo cấu hình MCP (Model Context Protocol). Nguyên lý chi tiết về MCP sẽ không được trình bày ở đây, chúng ta tập trung trình diễn cách sử dụng dịch vụ MCP được triển khai trên cloud. Case study này sử dụng chợ MCP của cộng đồng ModelScope (魔搭) trong nước để trình diễn, như Hình 5.26.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-20.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.26 Chợ MCP của cộng đồng ModelScope</p>
</div>

Mở chợ MCP của cộng đồng ModelScope, chọn loại hosted. Lấy MCP Amap (高德) làm ví dụ, sau khi vào trang chủ của nó, chọn chế độ SSE ở bên phải và click vào cấu hình kết nối, là có thể sinh ra JSON cấu hình MCP riêng, như Hình 5.27. MCP hỗ trợ nhiều chế độ giao tiếp, nhưng khi sử dụng trong Dify, giao tiếp theo chế độ SSE mượt mà và ổn định hơn, do đó khuyến nghị chọn chế độ SSE.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-21.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.27 Ví dụ cấu hình MCP Amap</p>
</div>

<strong>(2) Thiết kế Agent và trình diễn hiệu quả</strong>

Case study này sẽ tạo ra một trợ lý cá nhân toàn diện, bao gồm các module tính năng sau:

- Hỏi đáp đời sống hằng ngày
- Tối ưu hóa văn bản
- Sinh nội dung đa phương thức (hình ảnh, video)
- Truy vấn và phân tích trực quan hóa dữ liệu
- Tích hợp công cụ MCP (bản đồ Amap, gợi ý ẩm thực, tin tức thời sự)

Kiến trúc sắp xếp (orchestration) tổng thể của Agent như Hình 5.28.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-12.png" alt="Mô tả hình ảnh" width="60%"/>
  <p>Hình 5.28 Sắp xếp Agent</p>
</div>

Đối với kiến trúc đa Agent, chúng ta sử dụng bộ phân loại câu hỏi (question classifier) để định tuyến thông minh. Trong bộ phân loại, định nghĩa tính năng cốt lõi và phạm vi nhiệm vụ cho từng Agent, đảm bảo yêu cầu của người dùng được phân phối chính xác tới module xử lý tương ứng.

<strong>Module trợ lý hằng ngày</strong>

Đây là một module hội thoại cơ bản, cấu hình LLM và công cụ thời gian (time tool), đóng vai trò dịch vụ hỏi đáp tổng quát mang tính dự phòng (fallback).

Cấu hình prompt:

```
# Role: Chuyên gia tư vấn vấn đề hằng ngày

## Profile
- language: Tiếng Việt
- description: Chuyên trả lời các câu hỏi thông thường trong đời sống hằng ngày của người dùng, cung cấp lời khuyên và giải đáp thiết thực, chính xác, dễ hiểu
- background: Có kinh nghiệm sống phong phú và kho tàng kiến thức rộng rãi, giỏi đơn giản hóa các vấn đề phức tạp
- personality: Thân thiện gần gũi, kiên nhẫn tỉ mỉ, thực tế đáng tin cậy
- expertise: Đời sống hằng ngày, chăm sóc sức khỏe, quản lý gia đình, quan hệ giao tiếp, mẹo vặt thực tế


## Skills

1. Năng lực phân tích vấn đề
   - Hiểu nhanh: Nhanh chóng nắm bắt điểm cốt lõi trong câu hỏi của người dùng
   - Nhận diện phân loại: Xác định chính xác lĩnh vực đời sống mà vấn đề thuộc về
   - Khai thác nhu cầu: Hiểu sâu nhu cầu tiềm ẩn của người dùng
   - Sắp xếp ưu tiên: Đánh giá hợp lý mức độ quan trọng và khẩn cấp của vấn đề

2. Năng lực đưa ra giải đáp
   - Tích hợp kiến thức: Vận dụng tổng hợp kiến thức đa lĩnh vực để giải đáp
   - Lập phương án: Đưa ra giải pháp cụ thể và khả thi
   - Phân rã từng bước: Chia nhỏ vấn đề phức tạp thành các bước đơn giản
   - Phương án thay thế: Chuẩn bị nhiều phương án dự phòng để người dùng lựa chọn

3. Năng lực giao tiếp và diễn đạt
   - Ngôn ngữ gần gũi: Sử dụng ngôn từ đời thường đơn giản, dễ hiểu
   - Logic rõ ràng: Tổ chức nội dung câu trả lời mạch lạc, có trình tự
   - Minh họa bằng ví dụ: Giúp người dùng dễ hiểu thông qua các trường hợp cụ thể
   - Nhấn mạnh trọng tâm: Làm nổi bật thông tin then chốt và các điểm cần lưu ý

## Rules

1. Nguyên tắc trả lời:
   - Ưu tiên tính thiết thực: Đảm bảo lời khuyên đưa ra có tính khả thi cao
   - Đảm bảo tính chính xác: Đưa ra câu trả lời dựa trên thông tin đáng tin cậy và kiến thức thường thức
   - Trung lập khách quan: Tránh thiên kiến cá nhân và suy đoán chủ quan
   - Tư vấn vừa phải: Cung cấp mức độ giải đáp phù hợp với độ phức tạp của vấn đề

2. Quy tắc ứng xử:
   - Phản hồi kịp thời: Trả lời nhanh chóng câu hỏi của người dùng
   - Kiên nhẫn tỉ mỉ: Giữ sự kiên nhẫn với các câu hỏi lặp lại hoặc đơn giản
   - Chủ động dẫn dắt: Khuyến khích người dùng cung cấp thêm thông tin bối cảnh
   - Cải thiện liên tục: Tối ưu hóa chất lượng câu trả lời dựa trên phản hồi


## Workflows

- Mục tiêu: Cung cấp cho người dùng giải pháp thiết thực, đáng tin cậy cho các vấn đề hằng ngày
- Bước 1: Đọc kỹ và thấu hiểu vấn đề hằng ngày mà người dùng nêu ra
- Bước 2: Phân tích loại vấn đề và nhu cầu tiềm ẩn của người dùng
- Bước 3: Dựa trên kiến thức thường thức và kinh nghiệm để đưa ra gợi ý cụ thể, khả thi
- Bước 4: Tổ chức nội dung câu trả lời bằng ngôn ngữ mộc mạc, dễ hiểu
- Bước 5: Kiểm tra tính thiết thực và tính an toàn của câu trả lời


## Initialization
Với vai trò là chuyên gia tư vấn vấn đề hằng ngày, bạn bắt buộc phải tuân thủ các Rules nêu trên và thực thi nhiệm vụ theo Workflows.
```

Trình diễn hiệu quả như Hình 5.29:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-03.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.29 Trợ lý hằng ngày</p>
</div>

<strong>Module tối ưu hóa văn bản</strong>

Theo báo cáo dữ liệu của OpenAI, hơn 60% người dùng sử dụng ChatGPT cho các tác vụ liên quan đến tối ưu hóa văn bản, bao gồm chỉnh sửa, sửa đổi, mở rộng, rút gọn. Do đó, tối ưu hóa văn bản là một kịch bản nhu cầu tần suất cao, chúng ta sẽ lấy nó làm module tính năng cốt lõi thứ hai.

Cấu hình prompt:

```
# I. Thiết lập vai trò (Role)
Bạn là một chuyên gia tối ưu hóa nội dung (copywriting) chuyên nghiệp, sở hữu kinh nghiệm phong phú trong việc viết và tối ưu hóa nội dung marketing, giỏi nâng cao sức hút, tỷ lệ chuyển đổi và khả năng đọc của văn bản. Góc nhìn của bạn đứng từ phía đối tượng mục tiêu và mục tiêu marketing, phạm vi chuyên môn giới hạn trong lĩnh vực tối ưu hóa văn bản, không liên quan đến triển khai kỹ thuật hay phát triển sản phẩm.

# II. Bối cảnh (Background)
Người dùng cung cấp một đoạn văn bản gốc và cần bạn tối ưu hóa để nâng cao hiệu quả tổng thể. Thông tin bối cảnh bao gồm: văn bản có thể được sử dụng trong các kịch bản như marketing, quảng bá thương hiệu hoặc truyền đạt thông tin, nhưng mục đích sử dụng cụ thể chưa được mô tả chi tiết. Điều kiện đã biết là người dùng mong muốn văn bản thu hút hơn, rõ ràng hơn hoặc có sức thuyết phục hơn, nhưng chưa cung cấp nội dung văn bản gốc, vì vậy bạn cần làm việc dựa trên các nguyên tắc tối ưu hóa chung.

# III. Mục tiêu nhiệm vụ (Task)
- Phân tích và tối ưu hóa cấu trúc, ngôn ngữ và phong cách của văn bản để phù hợp hơn với sở thích của đối tượng mục tiêu.
- Nâng cao sức hút, khả năng đọc và tiềm năng chuyển đổi của văn bản, đảm bảo truyền đạt thông tin rõ ràng.
- Điều chỉnh dựa theo các nguyên tắc tối ưu hóa phổ biến (như tính súc tích, cộng hưởng cảm xúc, lời kêu gọi hành động - CTA, v.v.), không viết lại toàn bộ nội dung trừ khi cần thiết.
- Trên cơ sở giữ nguyên thông tin cốt lõi, mở rộng và làm phong phú nội dung văn bản một cách hợp lý để cung cấp phiên bản tối ưu hóa toàn diện hơn.

# IV. Giới hạn và lưu ý (Limit)
- Tránh làm thay đổi thông tin cốt lõi hoặc ý định của văn bản gốc, trừ khi người dùng yêu cầu rõ ràng.
- Không thêm nội dung hư cấu hoặc không liên quan, đảm bảo việc tối ưu hóa dựa trên logic và các best practice.
- Tránh sử dụng thuật ngữ quá mang tính kỹ thuật hoặc chuyên ngành sâu, trừ khi đối tượng mục tiêu là người trong ngành.
- Không can thiệp vào việc tối ưu hóa hình ảnh, bố cục hoặc các yếu tố phi văn bản khác.

# V. Yêu cầu định dạng đầu ra (Example)
Đầu ra phải là văn bản đã được tối ưu hóa với cấu trúc rõ ràng, ngôn từ trôi chảy và nội dung chi tiết. Ví dụ:
- Nếu văn bản gốc là "Sản phẩm của chúng tôi rất tốt, hãy mau mua đi"
Phiên bản sau khi tối ưu hóa có thể là: "Trong thời đại tràn ngập sự lựa chọn này, điều thực sự chạm đến trái tim người dùng chưa bao giờ là những lời quảng cáo hoa mỹ, mà là một sản phẩm chất lượng vượt qua được thử thách của thời gian và trải nghiệm thực tế. Sản phẩm của chúng tôi chính là như vậy. Nó không chỉ chú trọng đến từng chi tiết và chất lượng trong thiết kế, mà còn không ngừng được trau chuốt và đổi mới về tính năng, chỉ để mang lại trải nghiệm sử dụng tốt hơn cho mỗi khách hàng. Từ cảm giác tinh tế ở diện mạo bên ngoài đến sự ổn định trong hiệu năng vận hành, chúng tôi luôn kiên định với tiêu chuẩn cao và yêu cầu khắt khe, nỗ lực để mỗi khách hàng lựa chọn chúng tôi đều cảm nhận được sự hài lòng vượt mong đợi.
Chúng tôi hiểu sâu sắc rằng, việc mua một sản phẩm không đơn thuần chỉ là một giao dịch tiêu dùng, mà còn là sự lựa chọn cho một phong cách sống. Vì vậy, từ khâu chọn lọc chất liệu, quy trình chế tác cho đến dịch vụ hậu mãi, chúng tôi đều đặt trọn tâm huyết và sự chuyên nghiệp vào từng công đoạn, tận tâm nâng niu từng trải nghiệm của bạn. Dù bạn là người theo đuổi tính thực dụng, đề cao chất lượng, hay mong muốn dấu ấn cá nhân khác biệt, sản phẩm của chúng tôi đều có thể mang đến cho bạn giải pháp lý tưởng.
Ngay lúc này, hãy để hành động chứng minh tất cả. Một sản phẩm thực sự tốt không cần quá nhiều lời tô vẽ, bản thân nó đã là người đại diện thuyết phục nhất. Hãy hành động ngay hôm nay, lựa chọn chúng tôi, để chất lượng thay đổi cuộc sống và mang đến cho bạn trải nghiệm khác biệt từ giây phút này!"
- Đầu ra phải trình bày trực tiếp nội dung đã tối ưu hóa, không cần giải thích hay chú thích thêm trừ khi người dùng yêu cầu. Hãy đảm bảo nội dung văn bản sau tối ưu hóa phong phú và hoàn chỉnh hơn, đồng thời văn bản sau khi tối ưu hóa phải vượt quá 500 từ.
```

Trình diễn hiệu quả như Hình 5.30:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-04.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.30 Trợ lý văn bản</p>
</div>

<strong>Module sinh nội dung đa phương thức</strong>

Sinh ảnh và video là một kịch bản ứng dụng tần suất cao khác. Cùng với sự tiến hóa của các mô hình như Doubao sinh ảnh, Google Imagen, cũng như sự đột phá của các công nghệ sinh video như Kling (可灵), Google Veo 3, Seedance 2.0, chất lượng sinh nội dung đa phương thức đã đạt đến mức có thể sử dụng thực tế.

Case study này sử dụng plugin Jimeng (即梦) để hiện thực hóa việc sinh ảnh và video. Các bước cấu hình như sau:

1. Thêm plugin sinh ảnh/sinh video Jimeng vào workflow
2. Cấu hình tham số (như tỷ lệ ảnh 1:1, chọn mô hình seedream4.5/5.0 và seedance1.5/2.0)
3. Xuất file đã sinh ra

Ở đây chúng ta sử dụng plugin Jimeng để gọi mô hình mới nhất, tức seedream5.0 và seedance2.0. Chất lượng ảnh và video đều được cải thiện đáng kể. Đồng thời ở đây chúng ta đã sử dụng vòng lặp (loop) để lấy kết quả tác vụ video, cũng như bổ sung case study về lấy tham số và phán đoán điều kiện, nội dung cụ thể có thể xem tại [workflow hoàn chỉnh](code/chapter5/HelloAgent_difyCase.yml).

Cấu hình và hiệu quả sinh ảnh như Hình 5.31 và Hình 5.32.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-13.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.31 Thiết lập sinh ảnh</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-05.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.32 Trợ lý sinh ảnh</p>
</div>

Hiệu quả sinh video như Hình 5.33.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-06.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.33 Trợ lý sinh video</p>

<strong>Module truy vấn và phân tích dữ liệu</strong>

Xử lý dữ liệu là một trong những năng lực quan trọng của Agent. Module này trình diễn cách kết nối database trong Dify, hiện thực hóa truy vấn dữ liệu và phân tích trực quan hóa.

Trước tiên cài đặt plugin công cụ truy vấn dữ liệu, case study này sử dụng plugin `rookie-text2data`. Điểm then chốt của truy vấn dữ liệu là cung cấp cho LLM cấu trúc bảng và thông tin trường dữ liệu rõ ràng, giúp nó có thể sinh câu lệnh SQL truy vấn chính xác. Các cách làm phổ biến bao gồm:

- Trực tiếp cung cấp câu lệnh DDL của bảng dữ liệu
- Cung cấp mô tả quan hệ tương ứng giữa tên bảng và tên trường

Cấu hình thông tin kết nối database (địa chỉ IP, tên database, port, tài khoản, mật khẩu, v.v.), như Hình 5.34. Kết quả truy vấn cần được sắp xếp thông qua node LLM, chuyển thành output ngôn ngữ tự nhiên dễ hiểu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-22.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.34 Cấu hình database</p>
</div>

Cấu hình prompt:

```
# I. Thiết lập vai trò (Role)
Bạn là một chuyên viên truy vấn dữ liệu chuyên nghiệp, giỏi sắp xếp và tổng hợp dữ liệu, có tư duy logic rõ ràng và khả năng diễn đạt súc tích.

# II. Bối cảnh (Background)
Người dùng cung cấp dữ liệu thô được truy vấn từ cơ sở dữ liệu; dữ liệu này có thể gặp các vấn đề như định dạng không đồng nhất, thiếu trường thông tin, bản ghi trùng lặp, cần được xử lý và sắp xếp chuyên nghiệp trước khi hiển thị một cách hiệu quả.

# III. Mục tiêu nhiệm vụ (Task)
1. Tổng hợp và sắp xếp lại dữ liệu thô
2. Phân loại và sắp xếp dữ liệu theo logic chính xác
3. Trình bày dữ liệu sao cho làm nổi bật thông tin then chốt và các insight dữ liệu
4. Cung cấp cách hiển thị dữ liệu dễ hiểu

# IV. Giới hạn và lưu ý (Limit)
1. Không được tùy ý xóa bỏ dữ liệu quan trọng
2. Tránh sử dụng các thuật ngữ thống kê quá phức tạp hoặc quá chuyên sâu
3. Không được làm sai lệch giá trị thực của dữ liệu thô
4. Tránh hiển thị quá nhiều thông tin dư thừa, giữ cho nội dung ngắn gọn, rõ ràng
5. Không được làm rò rỉ dữ liệu nhạy cảm hoặc thông tin riêng tư cá nhân

# V. Yêu cầu định dạng đầu ra (Example)
Tổng quan dữ liệu: Chỉ cần giải thích ngắn gọn nội dung dữ liệu
```

Trình diễn hiệu quả như Hình 5.35:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-07.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.35 Trợ lý truy vấn dữ liệu</p>
</div>

Cấu hình prompt:

```
# I. Thiết lập vai trò (Role)
Bạn là một chuyên viên phân tích dữ liệu chuyên nghiệp, có năng lực tổng hợp, làm sạch và trực quan hóa dữ liệu, có thể trích xuất thông tin then chốt từ dữ liệu thô và chuyển hóa thành phần trình bày trực quan dễ hiểu.

# II. Bối cảnh (Background)
Người dùng đã truy vấn một tập dữ liệu thô từ cơ sở dữ liệu; dữ liệu này có thể chứa nhiều trường, có giá trị bị thiếu hoặc định dạng không nhất quán, cần được xử lý và sắp xếp trước khi tạo biểu đồ trực quan hóa.

# III. Mục tiêu nhiệm vụ (Task)
# Quy trình làm việc
1. Phân tích dữ liệu
Phân tích, sắp xếp và tổng kết dữ liệu theo các quy tắc hợp lý
2. Phân tích & Trực quan hóa
Tạo ít nhất 1 biểu đồ (chọn một hoặc nhiều trong số biểu đồ cột / biểu đồ đường / biểu đồ tròn)
Các công cụ có thể gọi: "generate_pie_chart" | "generate_column_chart" | "generate_line_chart"

# IV. Giới hạn và lưu ý (Limit)
1. Tránh sử dụng các loại biểu đồ quá phức tạp, đảm bảo kết quả trực quan hóa dễ hiểu
2. Không được bỏ qua các vấn đề về chất lượng dữ liệu, bắt buộc phải thực hiện làm sạch dữ liệu cần thiết
3. Tránh sử dụng quá nhiều màu sắc hoặc thành phần trong biểu đồ trực quan hóa, giữ sự tối giản và rõ ràng
4. Không được bỏ sót phần chú thích và giải thích cho các dữ liệu then chốt
5. Bắt buộc phải thực hiện tổng kết và tạo biểu đồ, bất kể lượng dữ liệu nhiều hay ít

# V. Yêu cầu định dạng đầu ra (Example)
Vui lòng xuất kết quả theo định dạng sau:
1. Tổng kết khái quát dữ liệu (không xuất tên trường dữ liệu, không chia gạch đầu dòng, chỉ cần một đoạn văn ngắn)
2. Hiển thị biểu đồ đã tạo
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-08.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.36 Trợ lý phân tích dữ liệu</p>
</div>

Điểm khác biệt duy nhất của trợ lý phân tích dữ liệu là chúng ta đã bổ sung công cụ trực quan hóa dữ liệu, tức là các plugin công cụ sinh biểu đồ BI "generate_pie_chart" | "generate_column_chart" | "generate_line_chart", phần này tin rằng mọi người đã cài đặt theo yêu cầu ở trên nên có thể trực tiếp thêm và kích hoạt sử dụng, và chỉ cần bổ sung mô tả tương ứng giống như prompt ở trên là được.

<strong>Tích hợp công cụ MCP</strong>

Cuối cùng là ứng dụng tích hợp công cụ MCP. Ở phần trước chúng ta đã hoàn thành cấu hình MCP, bây giờ sẽ tích hợp nó vào Agent. Các bước cấu hình như sau:

1. Chọn chiến lược Agent hỗ trợ gọi MCP
2. Chọn chế độ ReAct
3. Cấu hình dịch vụ MCP (lưu ý xóa tiền tố `mcp-server`, chọn chế độ SSE)
4. Điền prompt tương ứng

Giao diện cấu hình như Hình 5.37.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-23.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.37 Cấu hình MCP của Agent</p>
</div>

Hiệu quả của trợ lý Amap, trợ lý ẩm thực và trợ lý tin tức lần lượt như Hình 5.38, Hình 5.39 và Hình 5.40.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-09.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.38 Trợ lý Amap</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-10.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.39 Trợ lý ẩm thực</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-11.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.40 Trợ lý tin tức</p>
</div>

Đến đây, chúng ta đã hoàn thành một "Siêu trợ lý Agent cá nhân" toàn diện về tính năng. Trợ lý này bao quát nhiều khía cạnh cuộc sống: khi cần quần áo mới, có thể nhờ Doubao sinh thiết kế; trước khi ra ngoài, có thể nhờ trợ lý Amap lên kế hoạch lộ trình; không biết ăn gì, có thể lấy gợi ý ẩm thực; muốn hiểu tình hình học tập, có thể tiến hành phân tích dữ liệu. Agent này có thể xử lý nhiều loại công việc và tác vụ đời sống, mong chờ mọi người xây dựng được nhiều trợ lý Agent cá nhân sáng tạo hơn nữa.

### 5.3.3 Phân tích ưu điểm và hạn chế của Dify

Là một nền tảng phát triển ứng dụng AI hàng đầu, Dify thể hiện ưu thế đáng kể ở nhiều khía cạnh:

1. Ưu điểm cốt lõi

- Trải nghiệm phát triển full-stack: Dify tích hợp RAG pipeline, AI workflow, quản lý mô hình và nhiều tính năng khác vào một nền tảng duy nhất, mang lại trải nghiệm phát triển trọn gói
- Cân bằng giữa low-code và khả năng mở rộng cao: Dify đạt được sự cân bằng tốt giữa sự tiện lợi của phát triển low-code và tính linh hoạt của phát triển chuyên nghiệp
- Bảo mật và tuân thủ cấp doanh nghiệp: Dify cung cấp mã hóa AES-256, kiểm soát quyền RBAC và audit log, đáp ứng các yêu cầu bảo mật và tuân thủ nghiêm ngặt
- Khả năng tích hợp công cụ phong phú: Dify hỗ trợ mở rộng với hơn 9000 công cụ và API, cung cấp khả năng mở rộng tính năng rộng rãi
- Cộng đồng mã nguồn mở năng động: Dify sở hữu cộng đồng mã nguồn mở năng động, cung cấp nguồn tài liệu học tập và hỗ trợ phong phú

2. Hạn chế chính

- Đường cong học tập (learning curve) tương đối dốc: Đối với người dùng hoàn toàn không có nền tảng kỹ thuật, vẫn tồn tại đường cong học tập nhất định
- Nút thắt hiệu năng (performance bottleneck): Có thể gặp thách thức về hiệu năng trong kịch bản đồng thời cao, cần tối ưu hóa phù hợp. Các component dịch vụ backend cốt lõi của hệ thống Dify được triển khai bằng Python, so với các ngôn ngữ như C++, Golang, Rust thì hiệu năng tương đối kém hơn
- Hỗ trợ đa phương thức chưa đủ: Hiện tại chủ yếu xử lý văn bản, hỗ trợ cho hình ảnh, video, HTML còn hạn chế
- Chi phí bản Enterprise khá cao: Giá bản Enterprise của Dify tương đối cao, có thể vượt quá ngân sách của các nhóm nhỏ
- Vấn đề tương thích API: Định dạng API của Dify không tương thích với OpenAI, có thể hạn chế việc tích hợp với một số hệ thống bên thứ ba

## 5.4 Nền tảng ba: FastGPT

### 5.4.1 Giới thiệu và tính năng cốt lõi của FastGPT

FastGPT là một nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở dựa trên LLM, định vị cốt lõi là "động cơ sản xuất AI cấp doanh nghiệp" (enterprise-grade AI productivity engine), tập trung cung cấp giải pháp RAG (Retrieval-Augmented Generation) dễ sử dụng và khả năng sắp xếp workflow trực quan. Khác với trải nghiệm không cần code của Coze và năng lực phát triển full-stack của Dify, FastGPT coi hỏi đáp knowledge base là công dân hạng nhất (first-class citizen), tối ưu hóa chuyên sâu cho toàn bộ chuỗi "nhập dữ liệu — phân đoạn thông minh — truy xuất vector — sinh hội thoại".

Vào trang chủ chính thức của FastGPT, điều đầu tiên đập vào mắt là tuyên ngôn sản phẩm ngắn gọn mà mạnh mẽ — "Động cơ sản xuất AI cấp doanh nghiệp", nhấn mạnh việc xây dựng Agent AI cấp doanh nghiệp an toàn, có thể kiểm soát, như Hình 5.41.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-01.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.41 Trang chủ chính thức của FastGPT</p>
</div>

Sau khi đăng nhập vào nền tảng, có thể thấy bố cục workspace rõ ràng. Thanh điều hướng bên trái chia các tính năng cốt lõi thành bốn module lớn: cổng hội thoại (dialogue portal), workspace, knowledge base và tài khoản. Trong đó module Agent lại chia thành ba loại: workflow, Agent hội thoại và Agent hội thoại V2 (Beta), giúp người dùng tiện lựa chọn chế độ xây dựng phù hợp với kịch bản nghiệp vụ. Khu vực chính cung cấp lối vào nhanh "tạo mới từ template", tích hợp sẵn các template chính thức như Sales Coach Master, trợ lý dịch tài liệu, bản tin phân tích xu hướng ngành; phía dưới là danh sách Agent do người dùng tự tạo, như Hình 5.42.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-02.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.42 Giao diện chính của nền tảng FastGPT</p>
</div>

Về tài khoản và gói dịch vụ, FastGPT cung cấp bản miễn phí để nhà phát triển cá nhân trải nghiệm. Bản miễn phí bao gồm 100 điểm, 600 lượt index knowledge base, 1 thành viên nhóm, 10 Agent, 3 knowledge base, lưu trữ lịch sử hội thoại 30 ngày, tốc độ gọi 30 QPM, cùng quyền tải lên tối đa 5 file 50MB mỗi lần, như Hình 5.43. Đối với doanh nghiệp vừa và nhỏ cùng các nhóm, nền tảng cũng cung cấp gói nâng cấp trả phí để đáp ứng nhu cầu đồng thời và lưu trữ cao hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-03.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.43 Gói và mức sử dụng bản miễn phí của FastGPT</p>
</div>

Năng lực cạnh tranh cốt lõi nhất của FastGPT nằm ở knowledge base mạnh mẽ của nó. Nền tảng hỗ trợ import nhiều định dạng file, bao gồm các loại tài liệu phổ biến như Word, Markdown, PDF. Như Hình 5.44, trong "test knowledge base tổng quát", chúng ta có thể tải lên nhiều file như Giới thiệu Deep Learning, Nhập môn Machine Learning, nội dung chính hồ sơ mời thầu, hệ thống sẽ tự động phân đoạn file và xây dựng index, trạng thái hiển thị "đã sẵn sàng" là có thể được truy xuất và trích dẫn trong hội thoại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-12.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.44 Quản lý file knowledge base của FastGPT</p>
</div>

Ở tầng xử lý file, FastGPT cung cấp cấu hình tham số chi tiết. Như Hình 5.45, người dùng có thể chọn một trong hai cách xử lý "lưu trữ theo chunk" hoặc "trích xuất cặp hỏi-đáp", thiết lập điều kiện phân đoạn (ví dụ độ dài văn bản gốc lớn hơn 1000 ký tự thì kích hoạt phân đoạn), và bật nhiều tùy chọn tăng cường index, bao gồm đưa tiêu đề vào index, tự động sinh index bổ sung và tự động index hình ảnh. Đối với các tài liệu chứa nhiều nội dung văn bản-hình ảnh xen kẽ (như giáo trình, báo cáo nghiên cứu), tính năng tự động index hình ảnh đặc biệt quan trọng, nó giúp LLM hiểu và trích dẫn thông tin thị giác trong tài liệu khi trả lời.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-14.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.45 Thiết lập tham số xử lý dữ liệu knowledge base</p>
</div>

Sau khi tải lên xong, người dùng có thể xem nội dung cụ thể của file sau khi đã được phân đoạn. Như Hình 5.46, lấy ví dụ "sách điện tử tiếng Anh lớp 4 tập đầy đủ.pdf", nền tảng hiển thị preview văn bản của từng chunk, đồng thời panel metadata bên phải hiển thị các thông tin quan trọng như kích thước file (62MB), độ dài văn bản gốc (37797 ký tự), chế độ xử lý (lưu trữ theo chunk), trạng thái index hình ảnh. Cách hiển thị chunk minh bạch này giúp nhà phát triển tiện debug và tối ưu hóa knowledge base.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-13.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.46 Chi tiết phân đoạn file knowledge base và metadata</p>
</div>

Ngoài knowledge base, FastGPT trong khía cạnh tích hợp công cụ cũng bám sát xu hướng hệ sinh thái. Nền tảng hỗ trợ sẵn công cụ MCP (Model Context Protocol), người dùng có thể quản lý tập trung các dịch vụ MCP khác nhau trong module "Công cụ của tôi". Như Hình 5.47, trong thư mục "AI tài chính", chúng ta đã cấu hình nhiều công cụ MCP như tổng hợp xu hướng tiếng Trung, MCP cổ phiếu thời gian thực, MCP quỹ Qieman (且慢), Minimax-MCP, công cụ vẽ biểu đồ BI, những công cụ này sẽ trao cho Agent năng lực gọi dữ liệu thời gian thực và dịch vụ chuyên nghiệp bên ngoài.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-04.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.47 Quản lý công cụ MCP của FastGPT</p>
</div>

### 5.4.2 Xây dựng "Trợ lý tư vấn đầu tư thông minh"

**Mô tả case study:** Case study này sẽ dựa trên nền tảng FastGPT, kết hợp knowledge base, công cụ MCP và sắp xếp workflow, xây dựng một "Trợ lý tư vấn đầu tư thông minh" chuyên nghiệp. Trợ lý này có thể trả lời các câu hỏi lý thuyết đầu tư tài chính, truy vấn giá cổ phiếu thời gian thực, thực hiện đánh giá hồ sơ rủi ro người dùng, và sinh báo cáo phân tích chiến lược đầu tư được cá nhân hóa. Thông qua case study này, bạn sẽ nắm được mô hình phát triển cốt lõi của FastGPT: xây dựng knowledge base, tích hợp công cụ MCP, sắp xếp workflow trực quan và thiết kế tương tác hội thoại nhiều lượt.

**Bước 1: Cấu hình công cụ MCP**

Một trong những năng lực cốt lõi của trợ lý tư vấn đầu tư thông minh là lấy dữ liệu tài chính thời gian thực. Trong FastGPT, chúng ta hiện thực hóa tính năng này bằng cách tích hợp công cụ MCP.

Case study này cần hai loại dịch vụ MCP sau:

1. **Truy vấn giá cổ phiếu thời gian thực**: dùng để lấy giá, biến động tăng giảm, khối lượng giao dịch thời gian thực của cổ phiếu riêng lẻ.
2. **Dữ liệu tài chính và sinh biểu đồ**: dùng để lấy dữ liệu tài chính vĩ mô cũng như sinh biểu đồ trực quan hóa.

Như Hình 5.48, chúng ta có thể tìm thấy "MCP Server biểu đồ trực quan hóa" trong chợ MCP của cộng đồng ModelScope. Dịch vụ này được phát triển bằng TypeScript, tương thích với giao thức MCP, cung cấp năng lực sinh nhiều loại biểu đồ như biểu đồ vùng, biểu đồ cột, biểu đồ tròn, có thể chuyển hóa dữ liệu khô khan thành kết quả trực quan hóa dễ hiểu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-05.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.48 MCP Server biểu đồ trực quan hóa của cộng đồng ModelScope</p>
</div>

Ngoài ra, như Hình 5.49, nền tảng Alibaba Cloud Bailian (百炼) cũng cung cấp nhiều dịch vụ MCP chính thức phong phú. Trong trang quản lý MCP, chúng ta có thể tìm thấy các dịch vụ MCP tài chính như "Tin tức đầu tư hôm nay - tài chính..." và "Qieman" (且慢), cũng như các công cụ truy vấn giá cổ phiếu thời gian thực, sinh video Wanxiang (万相). Sau khi thêm các dịch vụ này vào kho công cụ MCP của FastGPT, Agent có thể gọi chúng theo nhu cầu trong hội thoại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-06.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.49 Quản lý MCP của Alibaba Cloud Bailian</p>
</div>

Trong giao diện cấu hình công cụ MCP của FastGPT, điền địa chỉ dịch vụ và thông tin xác thực tương ứng, là có thể hoàn thành việc tích hợp công cụ. Mỗi công cụ MCP đều có thể thiết lập mô tả và tham số gọi riêng, giúp Agent hiểu công dụng của từng công cụ khi ra quyết định.

**Bước 2: Thiết kế workflow tư vấn đầu tư thông minh**

Sau khi hoàn thành cấu hình công cụ, bước vào khâu sắp xếp workflow cốt lõi. FastGPT cung cấp giao diện sắp xếp Flow trực quan, người dùng có thể xây dựng luồng hội thoại phức tạp bằng cách kéo thả node, kết nối đường nối.

Như Hình 5.50, workflow hoàn chỉnh của "Trợ lý tư vấn đầu tư thông minh" bao gồm nhiều nhánh xử lý: nhận diện ý định người dùng, truy xuất knowledge base, thu thập bảng câu hỏi rủi ro, gọi công cụ MCP, sinh báo cáo. Toàn bộ workflow thể hiện cấu trúc mô-đun rõ ràng, dữ liệu luân chuyển có trật tự giữa các node khác nhau. Cách sắp xếp trực quan này giúp nhà phát triển hiểu và debug trực quan lộ trình ra quyết định của Agent.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-07.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.50 Sắp xếp workflow trợ lý tư vấn đầu tư thông minh</p>
</div>

Logic cốt lõi của workflow như sau:

1. **Node nhận diện ý định**: Trước tiên phán đoán loại input của người dùng. Nếu là tư vấn khái niệm tài chính, đi vào nhánh truy xuất knowledge base; nếu là truy vấn cổ phiếu, đi vào nhánh gọi công cụ MCP; nếu là chẩn đoán đầu tư, thì đi vào quy trình thu thập bảng câu hỏi rủi ro.
2. **Chuyên viên giáo dục kiến thức đầu tư**: Kết nối tới knowledge base tài chính đã được xây dựng sẵn, truy xuất các lý thuyết đầu tư, giải thích khái niệm và phân tích case liên quan.
3. **Chuyên gia phân tích đánh giá rủi ro**: Thông qua node nhập form, hướng dẫn người dùng hoàn thành bảng câu hỏi đánh giá rủi ro, bao gồm các chiều như tuổi tác, kinh nghiệm đầu tư, mức thu nhập hàng tháng, khả năng chịu rủi ro và mục tiêu đầu tư, sau đó truyền dữ liệu cho node LLM tiếp theo để phân tích môi trường thị trường, cơ bản (fundamentals), cảm xúc tin tức, tổng hợp hồ sơ người dùng, dữ liệu thị trường và thông tin tin tức, gọi LLM để sinh báo cáo phân tích chiến lược đầu tư có cấu trúc.
4. **Chuyên viên tình báo tin tức thị trường**: Căn cứ nhu cầu người dùng gọi MCP cổ phiếu thời gian thực hoặc MCP sinh biểu đồ, lấy dữ liệu bên ngoài.
5. **Chuyên viên tư vấn tổng quát**: Căn cứ câu hỏi tư vấn của người dùng, trả lời một số nội dung đơn giản, ví dụ như "xin chào", "bạn khỏe không".

**Bước 3: Cấu hình prompt và knowledge base**

Trong FastGPT, việc cấu hình prompt (System Prompt) cũng quan trọng không kém. Đối với trợ lý tư vấn đầu tư thông minh, chúng ta cần thiết lập cho nó vai trò một cố vấn đầu tư tài chính chuyên nghiệp:

```
# I. Thiết lập vai trò (Role)
Bạn là một cố vấn đầu tư tài chính chuyên nghiệp, có kinh nghiệm phong phú về đánh giá rủi ro và quản lý danh mục đầu tư. Nền tảng chuyên môn của bạn bao gồm tài chính học, tài chính hành vi và tâm lý học đầu tư, có khả năng phân tích mức độ chịu đựng rủi ro của người dùng từ nhiều chiều cạnh. Giọng văn của bạn cần chuyên nghiệp, trung lập và dễ hiểu, tránh sử dụng các thuật ngữ tài chính quá phức tạp, đảm bảo nhà đầu tư phổ thông cũng có thể dễ dàng hiểu được phân tích của bạn.

# II. Bối cảnh (Background)
Người dùng sẽ cung cấp các thông tin sau: độ tuổi, kinh nghiệm đầu tư, mức thu nhập hàng tháng, mức thua lỗ tối đa có thể chấp nhận, mục tiêu đầu tư. Những thông tin này là cơ sở để tiến hành đánh giá rủi ro; bạn cần dựa trên các dữ liệu này, kết hợp với các nguyên tắc tài chính phổ quát (như lý thuyết vòng đời, nguyên tắc tương xứng giữa rủi ro và lợi nhuận) để phân tích. Giới hạn của kịch bản là bạn không thể tiếp cận các thông tin cá nhân khác của người dùng (như tổng tài sản, tình trạng gia đình), vì vậy phân tích cần tập trung vào thông tin được cung cấp, tránh suy diễn quá mức.

# III. Mục tiêu nhiệm vụ (Task)
Dựa trên độ tuổi, kinh nghiệm đầu tư, mức thu nhập hàng tháng, mức thua lỗ tối đa có thể chấp nhận và mục tiêu đầu tư do người dùng cung cấp, thực hiện đánh giá rủi ro tổng hợp.
Xuất ra kết quả đánh giá mức độ rủi ro rõ ràng (ví dụ: Thận trọng, Ổn định, Tích cực, v.v.) và giải thích ngắn gọn lý do.
Đảm bảo logic phân tích mạch lạc, dễ để người dùng hiểu và áp dụng.

# IV. Giới hạn và lưu ý (Limit)
Tránh đưa ra khuyến nghị sản phẩm tài chính cụ thể (như tên cổ phiếu hoặc quỹ đầu tư cụ thể), chỉ thảo luận về các loại tài sản mang tính tổng quát.
Không được đưa ra bất kỳ cam kết hay dự báo bảo đảm lợi nhuận nào, cần nhấn mạnh rằng đầu tư luôn có rủi ro.
Tránh sử dụng ngôn ngữ quá mang tính kỹ thuật hoặc hàn lâm, đảm bảo nội dung đầu ra thân thiện với nhà đầu tư không chuyên.
Không mở rộng thông tin người dùng dựa trên giả định hoặc suy đoán, chỉ sử dụng độ tuổi, kinh nghiệm đầu tư, thu nhập hàng tháng, khả năng chịu lỗ tối đa và mục tiêu đầu tư được cung cấp để phân tích.
Trong nội dung đầu ra không được chứa bất kỳ diễn đạt nào mang tính phân biệt đối xử, thiên kiến hoặc quá chủ quan; cần giữ thái độ khách quan, trung lập.

# V. Yêu cầu định dạng đầu ra (Example)
Đầu ra cần được tổ chức theo cấu trúc sau:
**Đánh giá mức độ rủi ro**: [Ví dụ: Ổn định]
**Lý do đánh giá**: Giải thích ngắn gọn phân tích dựa trên độ tuổi, kinh nghiệm đầu tư, thu nhập, khả năng chịu lỗ và mục tiêu đầu tư của người dùng.
**Cảnh báo rủi ro**: Nhắc lại rủi ro đầu tư, khuyến khích người dùng tự điều chỉnh theo tình hình thực tế của bản thân.
```

Đồng thời, chúng ta cần cấu hình knowledge base tài chính cho trợ lý. Tải lên knowledge base các tài liệu như kiến thức nền tảng đầu tư, phân tích báo cáo tài chính, giải đọc chỉ số kinh tế vĩ mô, thực hiện quy trình phân đoạn và index theo Hình 5.44~Hình 5.46. Như vậy, khi người dùng hỏi những câu hỏi mang tính khái niệm như "tỷ lệ P/E và tỷ lệ P/B khác nhau như thế nào", Agent có thể truy xuất được định nghĩa chính xác và phân tích so sánh từ knowledge base, thay vì hoàn toàn dựa vào kiến thức pretrain của LLM, từ đó giảm hiệu quả rủi ro ảo giác (hallucination).

**Bước 4: Kiểm thử và trình diễn hiệu quả**

Sau khi hoàn thành cấu hình workflow và prompt, chúng ta có thể kiểm thử trong giao diện hội thoại của FastGPT. Như Hình 5.51, lời mở đầu của trợ lý tư vấn đầu tư thông minh giới thiệu rõ ràng ba đặc điểm tính năng chính của nó: thông thạo lý thuyết đầu tư tài chính, cung cấp tin tức và dữ liệu thị trường thời gian thực, cung cấp gợi ý phân bổ tài sản dựa trên đánh giá hồ sơ rủi ro. Phía dưới giao diện còn cung cấp nút thao tác nhanh, giúp người dùng kích hoạt các tác vụ phổ biến chỉ bằng một chạm.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-08.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.51 Giao diện hội thoại trợ lý tư vấn đầu tư thông minh</p>
</div>

Khi người dùng click "Tiến hành đánh giá tài sản để lấy ý kiến đầu tư", trợ lý sẽ lần lượt mở bảng câu hỏi đánh giá rủi ro, thu thập thông tin về tuổi tác, kinh nghiệm đầu tư, mức thu nhập hàng tháng, khả năng chịu lỗ tối đa và mục tiêu đầu tư của người dùng. Dựa trên những thông tin này, trợ lý sẽ sinh ra một báo cáo phân tích chiến lược đầu tư hoàn chỉnh.

Như Hình 5.52, báo cáo bao gồm các module cốt lõi sau:

- **Đánh giá rủi ro người dùng**: Dựa trên kết quả bảng câu hỏi, phân tích mức độ chịu rủi ro của người dùng (ví dụ: nhóm ổn định).
- **Gợi ý tỷ lệ phân bổ tài sản**: Thể hiện dưới dạng biểu đồ tròn trực quan tỷ lệ phân bổ các loại tài sản lớn như cổ phiếu, trái phiếu, tiền mặt (ví dụ cổ phiếu 45%, trái phiếu 40%, tiền mặt 15%).
- **Phân tích cơ bản (fundamentals) thị trường**: Kết hợp môi trường kinh tế vĩ mô hiện tại và xu hướng ngành, đưa ra nhận định thị trường.
- **Chiến lược tái cân bằng (rebalancing)**: Cung cấp phương án gợi ý điều chỉnh danh mục định kỳ, bao gồm chu kỳ điều chỉnh và điều kiện kích hoạt.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-09.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.52 Báo cáo phân tích chiến lược đầu tư</p>
</div>

Đối với kịch bản truy vấn dữ liệu thời gian thực, như Hình 5.53, khi người dùng hỏi "truy vấn giá cổ phiếu hiện tại của Kweichow Moutai", Agent sẽ tự động gọi công cụ MCP (`get_stock_quote_realtime`) để lấy dữ liệu giá thời gian thực. Kết quả trả về bao gồm tiêu đề, nguồn dữ liệu, các điểm chính (giá mở cửa, giá cao nhất, biên độ giá trong ngày, khối lượng giao dịch, tổng giá trị vốn hóa, giá trị vốn hóa lưu hành, v.v.), cùng với phân tích tác động tiềm năng và hành động đề xuất. Output có cấu trúc, chuyên nghiệp như vậy thể hiện giá trị thực tế của năng lực gọi công cụ (tool calling) của Agent.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-10.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.53 Truy vấn giá cổ phiếu thời gian thực</p>
</div>

Về mặt giải thích khái niệm, như Hình 5.54, khi người dùng hỏi "tỷ lệ P/E và tỷ lệ P/B khác nhau như thế nào", trợ lý dựa trên sự hiểu biết từ knowledge base và LLM, đưa ra phân tích so sánh có hệ thống: xuất phát từ định nghĩa, giải thích chi tiết cách tính P/E Ratio (tỷ lệ giá trên lợi nhuận) và P/B Ratio (tỷ lệ giá trên giá trị sổ sách); so sánh theo bốn chiều (cơ sở tính toán, ngành áp dụng, thông tin phản ánh, hạn chế); cuối cùng đưa ra gợi ý ứng dụng thực tế, giải thích khi nào nên tập trung vào P/E, khi nào nên tập trung vào P/B. Output có tầng lớp rõ ràng, logic chặt chẽ như vậy chính là ưu thế điển hình của LLM được tăng cường bởi RAG trong hỏi đáp lĩnh vực chuyên sâu (vertical domain).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-11.png" alt="Mô tả hình ảnh" width="50%"/>
  <p>Hình 5.54 Giải thích khái niệm P/E và P/B</p>
</div>

### 5.4.3 Phân tích ưu điểm và hạn chế của FastGPT

Thông qua thực hành xây dựng "Trợ lý tư vấn đầu tư thông minh" ở trên, chúng ta có thể hình thành một nhận thức tương đối toàn diện về nền tảng FastGPT.

**Ưu điểm:**

- **Trải nghiệm knowledge base đến mức cực hạn**: Ưu thế cốt lõi nhất của FastGPT nằm ở việc chăm chút sâu cho chuỗi RAG. Từ tải file, phân đoạn thông minh, tăng cường index, nhận diện hình ảnh đến truy xuất, mỗi khâu đều cung cấp tùy chọn cấu hình chi tiết và giao diện debug minh bạch. Đối với các kịch bản cần xây dựng hệ thống hỏi đáp dựa trên knowledge base riêng (như trợ lý tri thức doanh nghiệp, chăm sóc khách hàng thông minh, tư vấn lĩnh vực chuyên sâu), FastGPT cung cấp trải nghiệm tuyệt vời, dùng ngay (out-of-the-box).
- **Hỗ trợ MCP gốc (native)**: Khác với Coze, FastGPT hỗ trợ sẵn giao thức MCP, có thể kết nối liền mạch với số lượng lớn dịch vụ MCP trong các hệ sinh thái như cộng đồng ModelScope, Alibaba Cloud Bailian. Điều này giúp năng lực mở rộng công cụ của Agent không còn bị giới hạn bởi kho plugin tích hợp sẵn của nền tảng, nhà phát triển có thể tự do kết nối với bất kỳ công cụ bên thứ ba nào tuân theo chuẩn MCP.
- **Thiết kế trung lập về mô hình**: FastGPT hỗ trợ linh hoạt kết nối với nhiều mô hình lớn chủ lưu trong và ngoài nước như OpenAI, Claude, Tongyi Qianwen, DeepSeek, người dùng có thể tự do chuyển đổi mô hình nền tảng theo nhu cầu nghiệp vụ và cân nhắc chi phí, tránh rủi ro bị ràng buộc bởi một nhà cung cấp mô hình duy nhất.
- **Sắp xếp workflow trực quan**: Module Flow cung cấp giao diện sắp xếp theo node trực quan, các logic đa nhánh phức tạp (như nhận diện ý định, thu thập bảng câu hỏi, sinh báo cáo trong case study này) có thể được xây dựng nhanh chóng bằng cách kéo thả, hạ thấp rào cản tiếp cận cho người không chuyên về phát triển.

**Hạn chế:**

- **Hệ sinh thái template tương đối yếu**: So với chợ plugin phong phú của Coze và Marketplace với hơn 8000 plugin của Dify, số lượng template chính thức và công cụ tích hợp sẵn của FastGPT tương đối hạn chế. Mặc dù giao thức MCP đã phần nào bù đắp cho khoảng trống này, nhưng đối với người dùng không chuyên về kỹ thuật, việc tìm kiếm và cấu hình dịch vụ MCP phù hợp vẫn tồn tại rào cản nhất định.
- **Hạn mức bản miễn phí khá eo hẹp**: Bản miễn phí chỉ cung cấp 100 điểm và tốc độ gọi 30 QPM, đối với nhà phát triển cần kiểm thử và lặp lại thường xuyên, hạn mức tiêu hao khá nhanh. Giới hạn về số lượng index knowledge base và số lượng thành viên nhóm cũng khiến bản miễn phí khó có thể hỗ trợ cho nhóm có quy mô lớn hơn một chút.
- **Cộng đồng và tài liệu vẫn đang hoàn thiện**: Là một dự án mã nguồn mở tương đối trẻ, mức độ năng động của cộng đồng và độ hoàn thiện của tài liệu tiếng Anh của FastGPT vẫn còn khoảng cách nhất định so với các nền tảng trưởng thành như Dify, n8n. Khi gặp vấn đề ở các trường hợp biên (edge case), có thể cần phải đào sâu vào source code hoặc cầu cứu cộng đồng.

Nhìn chung, FastGPT là một nền tảng có tính cạnh tranh cực cao trong lĩnh vực hỏi đáp knowledge base. Nếu nhu cầu cốt lõi của bạn là xây dựng hệ thống hỏi đáp thông minh dựa trên tài liệu riêng, và mong muốn có khả năng kiểm soát chi tiết toàn bộ quy trình RAG, FastGPT là một lựa chọn rất đáng để thử. Còn đối với các kịch bản cần hệ sinh thái plugin cực mạnh hoặc tự động hóa quy trình nghiệp vụ phức tạp, có thể kết hợp bổ sung với Dify hoặc n8n.

## 5.5 Nền tảng bốn: n8n

Như chúng ta đã giới thiệu trước đây, danh tính cốt lõi của n8n là một nền tảng tự động hóa workflow tổng quát, chứ không phải một công cụ xây dựng ứng dụng LLM thuần túy. Hiểu rõ điều này là chìa khóa để nắm vững n8n. Khi sử dụng n8n để xây dựng ứng dụng thông minh, thực chất chúng ta đang thiết kế một quy trình tự động hóa lớn hơn, và LLM chỉ là một (hoặc nhiều) "node xử lý" mạnh mẽ trong quy trình đó.

### 5.5.1 Node và Workflow của n8n

Thế giới của n8n được cấu thành từ hai khái niệm cơ bản nhất: <strong>Node (node)</strong> và <strong>Workflow (workflow)</strong>.

- <strong>Node</strong>: Node là đơn vị nhỏ nhất thực thi thao tác cụ thể trong workflow. Bạn có thể tưởng tượng nó như một "khối lego" có chức năng riêng biệt. n8n cung cấp sẵn hàng trăm node dựng sẵn, bao quát các thao tác phổ biến từ gửi email, đọc/ghi database, gọi API đến xử lý file. Mỗi node đều có input và output, và cung cấp giao diện cấu hình đồ họa. Node có thể được chia thành hai loại lớn:
  - <strong>Trigger Node</strong>: Đây là điểm khởi đầu của toàn bộ workflow, chịu trách nhiệm kích hoạt quy trình. Ví dụ, "khi nhận được một email Gmail mới", "kích hoạt định kỳ mỗi giờ" hoặc "khi nhận được một request Webhook". Một workflow phải có và chỉ có đúng một trigger node.
  - <strong>Regular Node</strong>: Chịu trách nhiệm xử lý dữ liệu và logic cụ thể. Ví dụ, "đọc bảng Google Sheets", "gọi mô hình OpenAI" hoặc "thêm một bản ghi vào database".
- <strong>Workflow</strong>: Workflow là một sơ đồ quy trình tự động hóa được kết nối từ nhiều node. Nó định nghĩa đường đi hoàn chỉnh của dữ liệu bắt đầu từ trigger node, được truyền và xử lý từng bước qua các node khác nhau, và cuối cùng hoàn thành nhiệm vụ đã định sẵn như thế nào. Dữ liệu được truyền giữa các node theo định dạng JSON có cấu trúc, điều này cho phép chúng ta kiểm soát chính xác input và output của từng khâu.

Sức mạnh thực sự của n8n nằm ở khả năng "kết nối" mạnh mẽ của nó. Nó có thể kết nối các ứng dụng và dịch vụ vốn tách biệt (như CRM nội bộ doanh nghiệp, các nền tảng mạng xã hội bên ngoài, database của bạn và LLM) lại với nhau, hiện thực hóa việc tự động hóa quy trình nghiệp vụ end-to-end mà trước đây cần code phức tạp mới hoàn thành được. Trong phần thực hành tiếp theo, chúng ta sẽ tự tay trải nghiệm cách tận dụng hệ thống node và workflow này để xây dựng một ứng dụng tự động hóa tích hợp năng lực AI.

### 5.5.2 Xây dựng trợ lý email thông minh

Về cấu hình môi trường và cách sử dụng cơ bản nhất của n8n, đã có tài liệu được biên soạn trong thư mục `Additional-Chapter` của dự án, ở đây sẽ không giới thiệu thêm. Ở mục trước, chúng ta đã tìm hiểu khái niệm cơ bản của n8n. Case study này sẽ thể hiện rõ ràng sự khác biệt cốt lõi giữa Agent AI hiện đại và workflow tự động hóa truyền thống. Quy trình truyền thống mang tính tuyến tính, còn Agent mà chúng ta sắp xây dựng sẽ có thể nhận email của người dùng, thông qua một <strong>node AI Agent</strong> cốt lõi để "suy nghĩ", tự chủ hiểu ý định người dùng, và tự ra quyết định, lựa chọn trong nhiều "công cụ" khả dụng, cuối cùng tự động sinh và gửi phản hồi có độ liên quan cao.

Toàn bộ quy trình mô phỏng một logic ra quyết định cao cấp hơn: `Nhận email -> AI Agent (Suy nghĩ -> Ra quyết định -> Gọi công cụ) -> Phản hồi`, như Hình 5.55.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-01.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.55 Sơ đồ kiến trúc Agent email thông minh tích hợp</p>
</div>

Khác với cách truyền thống là tách công cụ thành nhiều sub-workflow, node `AI Agent` của n8n cho phép chúng ta tích hợp các component như LLM, Memory (bộ nhớ), Tools (công cụ) vào trong một giao diện thống nhất, giúp đơn giản hóa đáng kể quá trình xây dựng.

Toàn bộ quá trình xây dựng chia thành hai bước cốt lõi:

1. <strong>Chuẩn bị "bộ nhớ" cho Agent</strong>: Tạo một quy trình độc lập, nạp knowledge base riêng cho Agent.
2. <strong>Xây dựng thân chính của Agent</strong>: Tạo workflow chính nhận email, suy nghĩ và phản hồi.

### 5.5.3 Xây dựng Knowledge Base riêng cho Agent

Để Agent có thể trả lời các câu hỏi về lĩnh vực cụ thể (ví dụ thông tin cá nhân hoặc tài liệu dự án của bạn), chúng ta cần chuẩn bị trước cho nó một "bộ não bên ngoài" — một knowledge base dạng vector.

Trong n8n, chúng ta có thể sử dụng node `Simple Vector Store` để nhanh chóng xây dựng một knowledge base trong bộ nhớ (in-memory). Quy trình chuẩn bị này thường chỉ cần chạy một lần khi cập nhật tri thức.

<strong>(1) Định nghĩa nguồn tri thức</strong>

Trước tiên, chúng ta sử dụng node `Code` để lưu trữ văn bản tri thức gốc. Đây là một cách đơn giản, nhanh chóng, trong dự án thực tế dữ liệu cũng có thể đến từ file, database, v.v.

- <strong>Node</strong>: `Code`
- <strong>Nội dung</strong>: Viết tri thức của bạn theo định dạng JSON.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-02.png" alt="Ảnh chụp màn hình node Code đã điền văn bản JSON knowledge base" width="90%"/>
  <p>Hình 5.56 Định nghĩa nguồn tri thức trong node Code</p>
</div>

```javascript
return [
  {
    "doc_id": "work-schedule-001",
    "content": "Thời gian làm việc của tôi là từ thứ Hai đến thứ Sáu, 9 giờ sáng đến 5 giờ chiều. Múi giờ là Giờ chuẩn miền Đông Úc (AEST)."
  },
  {
    "doc_id": "off-hours-policy-001",
    "content": "Ngoài giờ làm việc (bao gồm cuối tuần và các ngày nghỉ lễ), tôi không thể trả lời email ngay lập tức."
  },
  {
    "doc_id": "auto-reply-instruction-001",
    "content": "Nếu email được nhận ngoài giờ làm việc, trợ lý AI cần thông báo cho người gửi rằng email đã được tiếp nhận, và tôi sẽ xử lý cũng như phản hồi sớm nhất trong khung giờ từ 9 giờ đến 5 giờ của ngày làm việc tiếp theo."
  }
];
```

<strong>(2) Vector hóa văn bản (Embeddings)</strong>

Máy tính không thể trực tiếp hiểu văn bản, cần chuyển nó thành vector. Chúng ta sử dụng node `Embeddings` để hoàn thành công việc "phiên dịch" này.

- <strong>Node</strong>: `Embeddings Google Gemini`, chọn model là `gemini-embedding-exp-03-07`. Ở đây sử dụng Google API để trình diễn, nếu không biết cách lấy Google API có thể tham khảo [tài liệu chính thức](https://gemini-api.apifox.cn/).
- <strong>Cấu hình</strong>: Kết nối nó phía sau node `Code`, nó sẽ tự động chuyển văn bản đầu vào từ upstream thành dữ liệu vector.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-03.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.57 Vector hóa dữ liệu trong Code</p>
</div>

<strong>(3) Lưu vào Vector Store</strong>

Cuối cùng, chúng ta lưu tri thức đã vector hóa vào database trong bộ nhớ, như Hình 5.58.

- <strong>Node</strong>: `Simple Vector Store`
- <strong>Cấu hình</strong>:
  - <strong>Operation Mode</strong>: `Insert Documents` (chế độ ghi).
  - <strong>Memory Key</strong>: Đặt một tên duy nhất cho knowledge base này, ví dụ `my-dailytime`. Key này tương đương "tên bảng" của database, sau này Agent sẽ dùng nó để tìm thông tin.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-04.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.58 Lưu dữ liệu trong Code vào Vector Store</p>
</div>

Sau khi hoàn thành cấu hình, <strong>thực thi thủ công một lần</strong> quy trình này. Thành công rồi, tri thức riêng của bạn đã được nạp vào bộ nhớ của n8n, như Hình 5.59.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-05.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.59 Workflow nạp knowledge base hoàn chỉnh</p>
</div>

### 5.5.4 Tạo Workflow chính của Agent

Đã có công cụ, bây giờ chúng ta bắt đầu xây dựng quy trình chính của Agent. Nó sẽ chịu trách nhiệm nhận email, suy nghĩ và ra quyết định, và gọi công cụ chúng ta vừa tạo vào đúng thời điểm, cuối cùng thực hiện việc phản hồi email.

(1) Cấu hình Gmail Trigger

Tạo một workflow mới, đặt tên là `Agent: Customer Support`. Sử dụng node `Gmail` làm trigger, thiết lập <strong>Event</strong> của nó thành `Message Received`, và cấu hình tài khoản email của bạn. Như vậy, mỗi khi có email mới vào hộp thư đến, workflow này sẽ tự động được kích hoạt, như Hình 5.60.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-06.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.60 Sơ đồ tạo mới node Gmail</p>
</div>

Quá trình cấu hình có thể tham khảo [tài liệu chính thức của n8n](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/?utm_source=n8n_app&utm_medium=credential_settings&utm_campaign=create_new_credentials_modal#enable-apis). API Gmail được [cấu hình](https://console.cloud.google.com/apis/library/gmail.googleapis.com?project=apt-entropy-471905-b9) tại đây, cần tạo credential, chọn loại Web Application, cuối cùng sẽ nhận được Client ID và Client Secret cần thiết. Và cần thêm OAuth Redirect URL mà n8n vừa cung cấp vào mục Authorized redirect URIs. Đồng thời, cũng cần thêm địa chỉ email của bạn vào mục Add users của [Audience](https://console.cloud.google.com/auth/audience?project=apt-entropy-471905-b9). Trang cấu hình hoàn chỉnh như Hình 5.61.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-07.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.61 Hình ảnh tài khoản Gmail đã nạp thành công</p>
</div>

Bây giờ chúng ta có thể click `Fetch Test Event` để lấy email, như Hình 5.62!

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-08.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.62 Hình ảnh lấy email thời gian thực</p>
</div>

(2) Cấu hình node AI Agent

Đây là "bộ não" của toàn bộ workflow. Kéo một node `AI Agent` ra từ menu node, và cấu hình như sau:

- <strong>Chat Model</strong>: Kết nối với LLM bạn chọn, ví dụ `Google Gemini Chat Model`. Đây là "lõi tư duy" của Agent.
- <strong>Memory</strong>: Kết nối một node `Simple Memory`. Điều này giúp Agent nhớ được lịch sử hội thoại trước đó khi xử lý nhiều email qua lại trong cùng một luồng email (thread).
- <strong>Tools</strong>: Chúng ta có thể kết nối nhiều công cụ vào đây. Trong case study này, chúng ta kết nối hai công cụ:
  1. `SerpAPI`: Đây là API chúng ta đã sử dụng trong case study ở Chương 4, giúp Agent có khả năng tìm kiếm thông tin công khai trên mạng.
  2. `Simple Vector Store`: Giúp Agent có khả năng truy vấn knowledge base riêng mà chúng ta đã tạo ở phần đầu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-09.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.63 Hình ảnh thiết lập node AI Agent</p>
</div>

Đây là bước đầu tiên trong việc "suy nghĩ" của Agent. Thêm một node `Gemini` (hoặc node LLM khác), chế độ thiết lập là `Chat`. Mục tiêu của chúng ta là để nó phân tích nội dung email, phán đoán ý định người dùng. Việc thiết kế Prompt cực kỳ quan trọng, một chỉ thị rõ ràng sẽ giúp LLM hoàn thành nhiệm vụ chính xác hơn. Chúng ta đưa nội dung và chủ đề email (`{{ $json.snippet }}{{ $json.Subject }}`) làm biến truyền vào Prompt, nếu không có API có thể vào [Google AI Studio](https://aistudio.google.com/prompts/new_chat) click Get API key để tạo một API key khả dụng.

Trong đó, đối với node AI Agent, phần chính chúng ta cần điền là `User Message` và `System Message`, như Hình 5.64.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-10.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.64 Hình ảnh giải thích chi tiết node AI Agent</p>
</div>

Dưới đây là Prompt mà case study của chúng ta sử dụng:

```json
# Prompt (User Message)
# Thông tin ngữ cảnh
- Thời gian hiện tại: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Giờ Sydney, Úc)
- Người gửi: {{ $json.From }}
- Chủ đề: {{ $json.Subject }}
- Nội dung email: {{ $json.snippet }}

# System Message
# Vai trò và mục tiêu
Bạn là một trợ lý email AI chuyên nghiệp, hiệu quả và sẵn sàng trực 24/7. Nhiệm vụ của bạn là: sử dụng thông tin công khai để cố gắng trả lời mọi câu hỏi trong email sớm nhất có thể, và căn cứ vào lịch làm việc của tôi để đính kèm lời nhắc trạng thái ngữ cảnh ở đầu thư phản hồi.

# Thông tin ngữ cảnh
- Thời gian hiện tại: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Giờ Sydney, Úc)
- Thông tin email nằm trong dữ liệu đầu vào.

# Công cụ khả dụng
- Simple Vector Store2: Dùng để truy vấn thời gian làm việc chính xác của tôi (ví dụ: thứ Hai đến thứ Sáu, 9 giờ sáng đến 5 giờ chiều).
- SerpAPI: **[Nguồn thông tin chính]** Ưu tiên sử dụng công cụ này để tìm kiếm trên Internet nhằm trả lời các câu hỏi cụ thể trong email.

# Các bước thực thi
1.  **Phân tích vấn đề**: Trước tiên, đọc kỹ nội dung email và trích xuất câu hỏi cốt lõi của người gửi.

2.  **Thu thập thông tin song song**: Thực hiện đồng thời hai thao tác sau để thu thập thông tin:
    a. Sử dụng công cụ `SerpAPI` để tìm kiếm câu trả lời cho câu hỏi của người gửi trên Internet.
    b. Sử dụng công cụ `Simple Vector Store2` để lấy thời gian làm việc chính xác mà tôi đã thiết lập.

3.  **Soạn thảo phản hồi cốt lõi**: Dựa trên thông tin thu thập được từ `SerpAPI`, trả lời rõ ràng và trực tiếp câu hỏi của người gửi; phần này sẽ đóng vai trò là nội dung chính của email phản hồi.

4.  **Thêm tiền tố trạng thái và tích hợp**:
    a. So sánh "Thời gian hiện tại" với thời gian làm việc của tôi lấy được từ công cụ.
    b. **Nếu hiện tại là "ngoài giờ làm việc"**: Tạo một đoạn tiền tố nhắc trạng thái. Đoạn tiền tố này **bắt buộc phải chứa** thời gian làm việc cụ thể lấy được từ `Simple Vector Store2`.
        * **Ví dụ tiền tố**: "Xin chào, cảm ơn bạn đã gửi thư. Bạn đã liên hệ với tôi vào ngoài giờ làm việc (thời gian làm việc của tôi là: [chèn thời gian làm việc truy vấn được vào đây]). Tôi sẽ trực tiếp xem xét email này vào ngày làm việc tiếp theo. Trong thời gian chờ đợi, đây là câu trả lời sơ bộ được tìm thấy dựa trên thông tin công khai:**<br><br>---<br><br>**"
    c. **Nếu hiện tại là "trong giờ làm việc"**: Chỉ cần dùng lời chào đơn giản là được.
        * **Ví dụ tiền tố**: "Xin chào, đối với câu hỏi bạn nêu ra, xin giải đáp như sau:**<br><br>---<br><br>**"
    d. Ghép tiền tố vừa tạo với phần phản hồi cốt lõi bạn đã soạn thảo (kết quả của bước 3) lại với nhau để tạo thành nội dung email hoàn chỉnh cuối cùng.

5.  **Định dạng đầu ra**: Bạn bắt buộc phải xuất nội dung email tạo thành cuối cùng theo một định dạng JSON nghiêm ngặt. Định dạng như sau, không thêm bất kỳ giải thích hay văn bản nào khác:
    {
      "shouldReply": true,
      "subject": "Re: [Tiêu đề email gốc]",
      "body": "[Đây là toàn bộ nội dung email phản hồi hoàn chỉnh đã được ghép lại, **mọi ký tự xuống dòng bắt buộc phải sử dụng thẻ HTML <br>**]"
    }

# Quy tắc và giới hạn
- **Luôn ưu tiên cố gắng trả lời**: Dù vào bất kỳ lúc nào, nhiệm vụ hàng đầu của bạn là sử dụng `SerpAPI` để cung cấp phản hồi có giá trị cho người dùng.
- **Bắt buộc phải khai báo trạng thái**: Nếu phản hồi ngoài giờ làm việc, bắt buộc phải nêu rõ ở phần đầu email và đính kèm thời gian làm việc chính xác của tôi.
- **Nguồn thông tin phải chính xác**: Thời gian làm việc phải tuân thủ nghiêm ngặt theo kết quả của `Simple Vector Store2`; câu trả lời cho các thắc mắc chủ yếu lấy từ `SerpAPI`, không được bịa đặt thông tin.
- **Định dạng đầu ra**: **Trong JSON đầu ra cuối cùng, mọi ký tự xuống dòng bên trong trường `body` bắt buộc phải dùng thẻ `<br>`, không được dùng `\n`.**
```

(3) Cấu hình công cụ của Agent

Đối với công cụ `Simple Vector Store`, chúng ta cần thực hiện cấu hình quan trọng, để đảm bảo nó có thể "đọc" chính xác tri thức mà chúng ta đã lưu trước đó:

- <strong>Operation Mode</strong>: `Retrieve Documents (As Tool for AI Agent)` (chế độ đọc dùng làm công cụ).
- <strong>Memory Key</strong>: Phải điền Key <strong>hoàn toàn giống</strong> với phần đầu, tức `my-dailytime`.
- <strong>Embeddings</strong>: Phải sử dụng model `Embeddings Google Gemini` <strong>hoàn toàn giống</strong> với phần đầu.

Chỉ khi `Memory Key` và model `Embeddings` hoàn toàn nhất quán, Agent mới có thể dùng đúng "chìa khóa" và "ngôn ngữ" để truy cập knowledge base, như Hình 5.65.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-11.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.65 Cấu hình công cụ Simple Vector Store</p>
</div>

Tham số Description chính là định nghĩa mô tả cho công cụ khi node AI Agent gọi nó, ở đây cũng đưa ra Prompt tương ứng:

```json
Đây là công cụ Simple Vector Store2, dùng để truy vấn thông tin cá nhân của tôi, đặc biệt là thời gian làm việc và chính sách phản hồi email của tôi. Khi cần phán đoán hiện tại có phải là giờ làm việc hay không, hoặc cần thông báo cho đối phương khi nào tôi sẽ trả lời email, bắt buộc phải sử dụng công cụ này.
```

Đối với Memory, điều duy nhất cần lưu ý là ở đây chúng ta sử dụng tên luồng (thread) của mỗi hộp email làm định danh duy nhất, để đảm bảo tính duy nhất khi lưu trữ, Key thiết lập là `{{ $('Gmail').item.json.threadId }}`

(4) Gửi phản hồi cuối cùng

Bước cuối cùng là thực thi. Kết nối output của node `AI Agent` tới một node `Gmail`, thiết lập <strong>Operation</strong> là `Send`. Sử dụng biểu thức (expression) của n8n, liên kết người nhận, chủ đề và nội dung email lần lượt với các trường tương ứng trong dữ liệu JSON output của `AI Agent`, là có thể hiện thực hóa việc tự động phản hồi email, như Hình 5.66.

- <strong>To</strong>: `{{ $('Gmail').item.json.From }}` (hoặc trường người gửi khác trong trigger)
- <strong>Subject</strong>: `Re:  {{ $('Gmail').item.json.Subject }}`
- <strong>Message</strong>: `{{ $json.output }}`

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-12.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.66 Hình ảnh minh họa công cụ phản hồi cuối cùng</p>
</div>

Và khi gửi thành công, cũng có thể nhận được email phản hồi thực tế trong hộp thư cá nhân, như Hình 5.67.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-13.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 5.67 Định dạng email phản hồi trong hộp thư cá nhân</p>
</div>

Đến đây, một trợ lý chăm sóc khách hàng thông minh tích hợp dựa trên node `AI Agent` đã được xây dựng xong, bạn có thể gửi một email kiểm thử để đánh giá thành quả hoạt động của nó. Kiến trúc này có khả năng mở rộng cực kỳ cao. Trong tương lai, bạn có thể trực tiếp thêm nhiều công cụ hơn vào node `AI Agent` (như lịch (calendar), database, CRM, v.v.), chỉ cần trong Prompt dạy Agent cách sử dụng chúng, là có thể không ngừng trao cho Agent của bạn năng lực mạnh mẽ hơn.

### 5.5.5 Phân tích ưu điểm và hạn chế của n8n

Thông qua thực hành xây dựng trợ lý email thông minh từ đầu đến cuối ở trên, chúng ta đã có cảm nhận trực quan về mô hình hoạt động của n8n. Là một nền tảng tự động hóa low-code mạnh mẽ, n8n thể hiện xuất sắc trong việc trao quyền phát triển ứng dụng Agent, nhưng nó cũng không phải vạn năng. Như Bảng 5.1, chúng ta sẽ phân tích khách quan ưu điểm và hạn chế tiềm ẩn của nó.

<div align="center">
  <p>Bảng 5.1 Tổng kết ưu điểm và hạn chế của nền tảng n8n</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-14.png" alt="Mô tả hình ảnh" width="90%"/>
</div>

Trước tiên, ưu điểm nổi bật nhất của n8n nằm ở <strong>hiệu quả phát triển</strong>. Nó trừu tượng hóa logic phức tạp thành workflow trực quan dễ hiểu, dù là việc nhận email, ra quyết định của AI, hay việc gọi công cụ và phản hồi cuối cùng, toàn bộ luồng dữ liệu và chuỗi xử lý đều hiện rõ trên canvas. Đặc tính low-code này hạ thấp đáng kể rào cản kỹ thuật, giúp nhà phát triển nhanh chóng xây dựng và xác thực logic cốt lõi của Agent, rút ngắn đáng kể khoảng cách từ ý tưởng đến Prototype.

Thứ hai, nền tảng <strong>mạnh mẽ về tính năng và tích hợp cao</strong>. n8n sở hữu kho node tích hợp sẵn phong phú, có thể dễ dàng kết nối hàng trăm dịch vụ phổ biến như Gmail, Google Gemini. Quan trọng hơn, node `AI Agent` tiên tiến của nó tích hợp cao độ việc quản lý model, memory và tools, giúp chúng ta chỉ cần một node là có thể hiện thực hóa việc ra quyết định tự chủ phức tạp, điều này thanh thoát và mạnh mẽ hơn nhiều so với cách định tuyến thủ công nhiều node truyền thống. Đồng thời, đối với các kịch bản mà tính năng tích hợp sẵn không thể bao quát, node `Code` cũng cung cấp sự linh hoạt để viết code tùy chỉnh, đảm bảo giới hạn trên của tính năng.

Cuối cùng, ở tầng <strong>triển khai và vận hành</strong>, n8n hỗ trợ <strong>triển khai riêng tư (private deployment)</strong>, và cũng là một trong những giải pháp Agent private hiện nay tương đối đơn giản và có thể triển khai phiên bản đầy đủ của dự án, điều này cực kỳ quan trọng đối với doanh nghiệp coi trọng bảo mật dữ liệu và quyền riêng tư. Chúng ta có thể triển khai toàn bộ dịch vụ trên server của riêng mình, đảm bảo các thông tin nhạy cảm như email nội bộ, dữ liệu khách hàng không rời khỏi môi trường của mình, điều này mang lại nền tảng vững chắc cho tính tuân thủ của ứng dụng Agent.

Tất nhiên, mỗi công cụ đều có sự đánh đổi của nó. Trong khi tận hưởng sự tiện lợi mà n8n mang lại, chúng ta cũng phải nhận thức được hạn chế của nó.

Đằng sau <strong>hiệu quả phát triển</strong>, là <strong>việc debug và xử lý lỗi tương đối rườm rà</strong>. Khi workflow trở nên phức tạp, một khi xảy ra lỗi định dạng dữ liệu, nhà phát triển có thể cần kiểm tra input/output của từng node để định vị vấn đề, điều này đôi khi không trực tiếp bằng việc đặt breakpoint trong code.

Về mặt tính năng, hạn chế lớn nhất thể hiện ở <strong>tính không bền vững (non-persistent) của bộ nhớ tích hợp sẵn</strong>. `Simple Memory` và `Simple Vector Store` mà chúng ta sử dụng trong case study đều dựa trên bộ nhớ (in-memory), nghĩa là một khi dịch vụ n8n khởi động lại, toàn bộ lịch sử hội thoại và knowledge base sẽ mất hết. Điều này là chí mạng đối với ứng dụng trong môi trường production. Do đó, khi triển khai thực tế, phải thay thế bằng database bền vững bên ngoài như Redis, Pinecone, điều này cũng sẽ làm tăng thêm chi phí cấu hình và bảo trì.

Ngoài ra, về mặt <strong>triển khai vận hành</strong> và hợp tác nhóm, <strong>kiểm soát phiên bản (version control) và hợp tác nhiều người của n8n chưa trưởng thành bằng code truyền thống</strong>. Mặc dù có thể export workflow ra file JSON để quản lý, nhưng so sánh thay đổi (diff) của nó không rõ ràng bằng `git diff` của code, nhiều người cùng sửa một workflow cũng dễ phát sinh xung đột (conflict).

Cuối cùng là về <strong>hiệu năng</strong>, n8n hoàn toàn có thể đáp ứng đại đa số các tác vụ tự động hóa doanh nghiệp và Agent tần suất trung-thấp. Nhưng đối với các kịch bản cần xử lý request với độ đồng thời cực cao, cơ chế điều phối node của nó có thể mang lại một số chi phí hiệu năng nhất định, so với dịch vụ triển khai bằng code thuần có thể hơi kém hơn một chút.

## 5.6 Tổng kết chương

Chương này đã giới thiệu một cách hệ thống về lý niệm, phương pháp và thực hành xây dựng ứng dụng Agent dựa trên nền tảng low-code, đánh dấu sự chuyển đổi quan trọng của chúng ta từ "viết code thủ công" sang "phát triển theo hướng nền tảng".

Ở mục đầu tiên, chúng ta đã trình bày bối cảnh và giá trị của sự trỗi dậy nền tảng low-code. So với việc triển khai Agent thuần code ở Chương 4, nền tảng low-code thông qua cách đồ họa, theo mô-đun, đã hạ thấp đáng kể rào cản kỹ thuật, nâng cao hiệu quả phát triển, và cung cấp trải nghiệm debug trực quan tốt hơn. Tầng "trừu tượng hóa cao hơn" này giúp nhà phát triển có thể tập trung tâm sức vào logic nghiệp vụ và prompt engineering, thay vì chi tiết triển khai ở tầng thấp.

Tiếp theo, chúng ta đã thực hành chuyên sâu với bốn nền tảng tiêu biểu, mỗi nền tảng có đặc điểm riêng:

**Coze** nổi bật với trải nghiệm thân thiện không cần code và hệ sinh thái plugin phong phú. Thông qua case study "Bản tin AI hằng ngày", chúng ta đã trải nghiệm cách nhanh chóng tích hợp thông tin đa nguồn bằng cấu hình kéo thả, và xuất bản một chạm lên nhiều nền tảng chủ lưu. Coze đặc biệt phù hợp với người dùng không có nền tảng kỹ thuật và các kịch bản cần nhanh chóng xác thực ý tưởng, nhưng hạn chế về việc không hỗ trợ MCP và không thể export file cấu hình chuẩn hóa cũng đáng lưu ý.

**Dify** với tư cách một nền tảng mã nguồn mở cấp doanh nghiệp, đã thể hiện năng lực phát triển full-stack. Case study "Siêu trợ lý Agent cá nhân" bao quát nhiều module như hỏi đáp hằng ngày, tối ưu hóa văn bản, sinh nội dung đa phương thức, phân tích dữ liệu và tích hợp công cụ MCP, thể hiện đầy đủ khả năng sắp xếp mạnh mẽ của Dify trong các kịch bản nghiệp vụ phức tạp. Chợ plugin phong phú (8000+), phương thức triển khai linh hoạt và đặc tính bảo mật cấp doanh nghiệp, khiến nó trở thành lựa chọn lý tưởng cho nhà phát triển chuyên nghiệp và đội nhóm doanh nghiệp. Tuy nhiên, đường cong học tập tương đối dốc và thách thức về hiệu năng trong kịch bản đồng thời cao cũng cần được cân nhắc.

**FastGPT** thì dựa vào trải nghiệm knowledge base RAG đến mức cực hạn, trở thành đối thủ cạnh tranh mạnh mẽ trong kịch bản hỏi đáp lĩnh vực chuyên sâu. Thông qua case study "Trợ lý tư vấn đầu tư thông minh", chúng ta đã trải nghiệm mô hình phát triển hoàn chỉnh từ xây dựng knowledge base, tích hợp công cụ MCP đến sắp xếp workflow trực quan. Việc FastGPT kiểm soát chi tiết các khía cạnh như phân đoạn file, tăng cường index, nhận diện hình ảnh, giúp nó có ưu thế độc đáo trong các kịch bản như trợ lý tri thức doanh nghiệp, chăm sóc khách hàng thông minh. Nhưng hệ sinh thái template tương đối yếu, hạn mức bản miễn phí có hạn, cũng hạn chế khả năng phát huy của nó trong các kịch bản nghiệp vụ phức tạp hơn.

**n8n** thì mở ra một con đường khác với khả năng "kết nối" độc đáo của mình. Thông qua case study "Trợ lý email thông minh", chúng ta đã thấy cách nhúng năng lực AI liền mạch vào quy trình tự động hóa nghiệp vụ phức tạp. Node AI Agent của n8n tích hợp cao độ model, memory và tools, phối hợp với hàng trăm node dựng sẵn, có thể hiện thực hóa phương án tự động hóa với mức độ tùy biến cao. Đặc tính hỗ trợ triển khai riêng tư của nó đặc biệt quan trọng đối với doanh nghiệp coi trọng bảo mật dữ liệu. Nhưng tính không bền vững của bộ nhớ tích hợp sẵn và sự chưa trưởng thành của kiểm soát phiên bản, cần được xử lý kỹ thuật bổ sung trong môi trường production.

Thông qua thực hành so sánh bốn nền tảng, chúng ta có thể rút ra các gợi ý lựa chọn sau:

- **Xác thực Prototype nhanh, người dùng không chuyên kỹ thuật**: Ưu tiên chọn Coze
- **Ứng dụng cấp doanh nghiệp, logic nghiệp vụ phức tạp, sinh nội dung đa phương thức**: Ưu tiên chọn Dify
- **Hệ thống hỏi đáp dựa trên knowledge base riêng, chăm sóc khách hàng thông minh**: Ưu tiên chọn FastGPT
- **Tích hợp nghiệp vụ sâu, quy trình tự động hóa tổng quát**: Ưu tiên chọn n8n

Điều đáng nhấn mạnh là, nền tảng low-code không nhằm thay thế việc phát triển bằng code, mà cung cấp một lựa chọn bổ sung. Trong dự án thực tế, chúng ta hoàn toàn có thể linh hoạt chuyển đổi theo nhu cầu của từng giai đoạn khác nhau: dùng nền tảng low-code để nhanh chóng xác thực ý tưởng, dùng code để hiện thực hóa việc kiểm soát chi tiết; dùng nền tảng để xử lý quy trình chuẩn hóa, dùng code để xử lý logic đặc thù. Tư duy "phát triển hỗn hợp" (hybrid development) này mới chính là best practice trong việc công nghiệp hóa Agent.

Chương tiếp theo, chúng ta sẽ tiếp tục khám phá các framework Agent ở tầng sâu hơn, giúp người đọc xây dựng được những ứng dụng đáng tin cậy và thú vị hơn nữa.

## Bài tập

1. Chương này đã giới thiệu bốn nền tảng low-code mỗi nền tảng có đặc điểm riêng: `Coze`, `Dify`, `FastGPT` và `n8n`. Hãy phân tích:

   - Bốn nền tảng này khác nhau như thế nào về định vị cốt lõi và lý niệm thiết kế? Chúng lần lượt giải quyết những điểm nghẽn (pain points) nào trong việc phát triển Agent?
   - Phát triển low-code và phát triển thuần code đều có ưu nhược điểm riêng, ngoài ra, cũng có mô hình "phát triển hỗn hợp" (hybrid development) trong đó một phần tính năng dùng nền tảng, một phần tính năng dùng code. Hãy suy nghĩ xem ba mô hình phát triển này lần lượt phù hợp với những kịch bản nào? Vui lòng cho ví dụ minh họa.
2. Trong case study `Coze` ở mục 5.2, chúng ta đã xây dựng một Agent "Bản tin AI hằng ngày". Hãy mở rộng suy nghĩ dựa trên case study này:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thao tác thực tế
   >

   - Việc sinh bản tin hiện tại được kích hoạt một cách bị động (người dùng chủ động hỏi). Làm thế nào để cải tạo Agent này, giúp nó có thể tự động sinh bản tin vào 8 giờ sáng mỗi ngày và đẩy tới nhóm Feishu hoặc WeChat Official Account chỉ định?
   - Chất lượng bản tin phụ thuộc rất lớn vào thiết kế prompt. Hãy thử tối ưu hóa prompt ở mục 5.2.2, giúp bản tin được sinh ra chuyên nghiệp hơn, có cấu trúc rõ ràng hơn, hoặc bổ sung các tính năng mới như "phân tích điểm nóng", "dự báo xu hướng".
   - Việc `Coze` hiện tại không hỗ trợ giao thức `MCP` được coi là một hạn chế quan trọng (trong quá trình viết bài tập này, `feature-mcp` tuy đã nằm trong [`Coze Studio Q4 2025 Product Roadmap`](https://github.com/coze-dev/coze-studio/issues/2218) nhưng vẫn chưa được hiện thực hóa). Hãy trình bày ngắn gọn, `MCP` là giao thức gì? Vì sao nó quan trọng? Nếu `Coze` trong tương lai hỗ trợ `MCP`, sẽ mang lại những khả năng mới nào?
3. Trong case study `Dify` ở mục 5.3, chúng ta đã xây dựng một "Siêu trợ lý Agent cá nhân" toàn diện về tính năng. Hãy phân tích sâu hơn:

   - Case study đã sử dụng "bộ phân loại câu hỏi" (question classifier) để định tuyến thông minh, phân phối các loại yêu cầu khác nhau đến các sub-agent khác nhau. Kiến trúc đa Agent này có ưu điểm gì? Nếu không dùng bộ phân loại, mà để một Agent đơn lẻ xử lý tất cả nhiệm vụ, sẽ gặp vấn đề gì?
   - Module truy vấn dữ liệu cần cung cấp thông tin cấu trúc bảng rõ ràng cho LLM. Nếu database có 50 bảng, mỗi bảng có 20 trường, việc trực tiếp đưa tất cả câu lệnh `DDL` vào prompt sẽ khiến context quá dài. Hãy thiết kế một phương án thông minh hơn để giải quyết vấn đề này.
   - `Dify` hỗ trợ cả hai chế độ triển khai local và cloud. Hãy so sánh sự khác biệt giữa hai chế độ này về bảo mật dữ liệu, chi phí, hiệu năng, độ khó bảo trì, và nêu rõ kịch bản phù hợp cho từng chế độ.
4. Trong case study `n8n` ở mục 5.5, chúng ta đã xây dựng một "Trợ lý email thông minh". Hãy suy nghĩ về các vấn đề sau:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thao tác thực tế
   >

   - `Simple Vector Store` và `Simple Memory` được sử dụng trong case study đều dựa trên bộ nhớ (in-memory), dữ liệu sẽ mất sau khi dịch vụ khởi động lại. Hãy tra cứu tài liệu `n8n`, thử thay thế bằng phương án lưu trữ bền vững (như `Pinecone`, `Redis`), và trình bày quá trình cấu hình.
   - Trợ lý email hiện tại chỉ có thể xử lý email dạng văn bản. Nếu email người dùng gửi có chứa file đính kèm (như tài liệu `PDF`, hình ảnh), bạn sẽ mở rộng workflow này như thế nào, để Agent có thể hiểu nội dung file đính kèm và đưa ra phản hồi tương ứng?
   - Ưu thế cốt lõi của `n8n` nằm ở khả năng "kết nối". Hãy thiết kế một kịch bản tự động hóa phức tạp hơn: khi khách hàng đặt hàng trên nền tảng thương mại điện tử, tự động kích hoạt một chuỗi thao tác (gửi email xác nhận, cập nhật database tồn kho, thông báo hệ thống logistics, ghi lại thông tin khách hàng trong `CRM`). Hãy vẽ sơ đồ kết nối node của workflow và trình bày các cấu hình then chốt.
5. Prompt engineering cũng quan trọng không kém trong nền tảng low-code. Chương này đã trình bày nhiều case study thiết kế prompt trên các nền tảng khác nhau. Hãy phân tích:

   - So sánh thiết kế prompt ở mục 5.2.2 (`Coze`), mục 5.3.2 (`Dify`) và mục 5.5.4 (`n8n`), chúng khác nhau như thế nào về cấu trúc, phong cách và trọng tâm? Những khác biệt này có liên quan đến đặc tính của từng nền tảng hay không?
   - Trong "module tối ưu hóa văn bản" của `Dify`, prompt yêu cầu output "vượt quá 500 từ". Yêu cầu cứng nhắc về độ dài output này có hợp lý không? Trường hợp nào nên giới hạn độ dài output, trường hợp nào nên để mô hình tự do phát huy?
6. Công cụ và plugin là phương thức mở rộng năng lực cốt lõi của nền tảng low-code. Hãy suy nghĩ:

   - `Coze` sở hữu kho plugin phong phú, `Dify` sở hữu Marketplace với hơn 8000 plugin, `FastGPT` hỗ trợ sẵn giao thức MCP, `n8n` sở hữu hàng trăm node dựng sẵn. Nếu cả ba nền tảng này đều không có công cụ cụ thể bạn cần (như "kết nối API hệ thống nội bộ công ty"), bạn sẽ giải quyết như thế nào?
   - Trong mục 5.3.2, chúng ta đã sử dụng giao thức `MCP` để tích hợp các dịch vụ như bản đồ Amap, gợi ý ẩm thực. Hãy nghiên cứu và trình bày: giao thức `MCP` khác với `RESTful API` truyền thống và `Tool Calling` ở những điểm nào? Vì sao `MCP` được coi là "tiêu chuẩn mới" của việc gọi công cụ trong Agent?
   - Giả sử bạn cần phát triển một plugin tùy chỉnh cho `Dify`, giúp nó có thể gọi hệ thống knowledge base nội bộ của công ty bạn. Hãy tra cứu tài liệu phát triển plugin của `Dify`, tóm tắt quy trình phát triển và các điểm kỹ thuật then chốt.
7. Lựa chọn nền tảng là một trong những quyết định then chốt cho sự thành công của sản phẩm Agent. Giả sử bạn là trưởng nhóm kỹ thuật của một công ty khởi nghiệp, công ty dự định phát triển ba ứng dụng AI sau, hãy chọn nền tảng phù hợp nhất (`Coze`, `Dify`, `FastGPT`, `n8n` hoặc phát triển thuần code) cho mỗi ứng dụng, và trình bày chi tiết lý do:

   <strong>Ứng dụng A</strong>: Mini-app "Trợ lý viết lách AI" hướng đến người dùng cuối (C-end), cần nhanh chóng ra mắt để xác thực nhu cầu thị trường, ngân sách hạn chế, đội nhóm chỉ có 1 kỹ sư frontend và 1 product manager.

   <strong>Ứng dụng B</strong>: Hệ thống "Kiểm tra hợp đồng thông minh" hướng đến khách hàng doanh nghiệp, cần xử lý tài liệu pháp lý nhạy cảm, yêu cầu dữ liệu không được rời khỏi môi trường riêng tư của khách hàng, cần tích hợp sâu với hệ thống OA, hệ thống quản lý tài liệu hiện có của khách hàng.

   <strong>Ứng dụng C</strong>: Công cụ "Nâng cao hiệu suất R&D" dùng nội bộ, cần tự động hóa xử lý nhiều khâu quy trình R&D như code review, sinh báo cáo kiểm thử, theo dõi Bug, đồng bộ tiến độ dự án, đội nhóm có thực lực kỹ thuật mạnh.

   Đối với mỗi ứng dụng, vui lòng phân tích từ các khía cạnh sau (bao gồm nhưng không giới hạn):

   > <strong>Gợi ý</strong>: năng lực nền tảng có đáp ứng được nhu cầu không, bao lâu thì có thể ra mắt, chi phí phát triển, chi phí vận hành, độ khó lặp lại (iterate) về sau, không gian mở rộng tính năng trong tương lai
   >

   - Tính khả thi kỹ thuật
   - Hiệu quả phát triển
   - Kiểm soát chi phí
   - Khả năng bảo trì
   - Khả năng mở rộng
   - Bảo mật dữ liệu và tính tuân thủ

## Tài liệu tham khảo

[1] Coze - Nền tảng phát triển ứng dụng AI thế hệ mới. https://www.coze.cn/

[2] Dify - Nền tảng phát triển ứng dụng LLM mã nguồn mở. https://dify.ai/

[3] FastGPT - Nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở. https://fastgpt.io/en/

[4] n8n - Công cụ tự động hóa workflow. https://n8n.io/
