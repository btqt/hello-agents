# Chương 1 Làm quen với Agent

Chào mừng bạn đến với thế giới của Agent! Trong làn sóng trí tuệ nhân tạo (AI) đang càn quét toàn cầu hiện nay, <strong>Agent (智能体)</strong> đã trở thành một trong những khái niệm cốt lõi thúc đẩy sự biến đổi công nghệ và đổi mới ứng dụng. Dù chí hướng của bạn là trở thành nhà nghiên cứu, kỹ sư trong lĩnh vực AI, hay chỉ đơn giản là một người quan sát mong muốn hiểu sâu về công nghệ tiên phong, việc nắm vững bản chất của Agent đều sẽ là một mảnh ghép không thể thiếu trong hệ thống kiến thức của bạn.

Vì vậy, trong chương này, hãy cùng quay về điểm khởi đầu và cùng nhau thảo luận một vài câu hỏi: Agent là gì? Nó có những loại chính nào? Và nó tương tác với thế giới xung quanh chúng ta ra sao? Thông qua những thảo luận này, hy vọng sẽ đặt nền móng vững chắc cho việc học tập và khám phá của bạn trong tương lai.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-0.png" alt="图片描述" width="90%"/>
  <p>Hình 1.1 Vòng lặp tương tác cơ bản giữa Agent và môi trường</p>
</div>

## 1.1 Agent là gì?

Khi khám phá bất kỳ khái niệm phức tạp nào, tốt nhất chúng ta nên bắt đầu từ một định nghĩa ngắn gọn. Trong lĩnh vực trí tuệ nhân tạo, Agent được định nghĩa là bất kỳ thực thể nào có khả năng cảm nhận <strong>môi trường (Environment)</strong> mà nó đang hoạt động thông qua <strong>cảm biến (Sensors)</strong>, và <strong>tự chủ</strong> thực hiện <strong>hành động (Action)</strong> thông qua <strong>bộ chấp hành (Actuators)</strong> nhằm đạt được một mục tiêu cụ thể.

Định nghĩa này bao hàm bốn yếu tố cơ bản tạo nên sự tồn tại của Agent. Môi trường là thế giới bên ngoài mà Agent đang hoạt động. Đối với một chiếc xe tự lái, môi trường là giao thông đường bộ biến đổi liên tục; đối với một thuật toán giao dịch, môi trường lại là thị trường tài chính thay đổi từng giây. Agent không tách biệt khỏi môi trường, nó liên tục cảm nhận trạng thái của môi trường thông qua các cảm biến của mình. Camera, micro, radar hay các luồng dữ liệu trả về từ các loại <strong>giao diện lập trình ứng dụng (Application Programming Interface, API)</strong> khác nhau, đều là sự mở rộng của khả năng cảm nhận này.

Sau khi có được thông tin, Agent cần thực hiện hành động để tác động lên môi trường, nó thay đổi trạng thái của môi trường thông qua bộ chấp hành. Bộ chấp hành có thể là thiết bị vật lý (như cánh tay robot, vô lăng) hoặc công cụ ảo (như thực thi một đoạn code, gọi một dịch vụ).

Tuy nhiên, điều thực sự mang lại "trí tuệ" cho Agent chính là <strong>tính tự chủ (Autonomy)</strong> của nó. Agent không chỉ là một chương trình phản ứng thụ động trước các kích thích bên ngoài hay thực thi nghiêm ngặt các chỉ thị được định sẵn, nó có khả năng đưa ra quyết định độc lập dựa trên nhận thức và trạng thái nội tại của mình để đạt được mục tiêu thiết kế. Vòng khép kín từ cảm nhận đến hành động này tạo thành nền tảng cho mọi hành vi của Agent, như minh họa trong Hình 1.1.


### 1.1.1 Góc nhìn truyền thống về Agent

Trước khi làn sóng <strong>mô hình ngôn ngữ lớn (Large Language Model, LLM)</strong> hiện nay xuất hiện, các nhà tiên phong của trí tuệ nhân tạo đã khám phá và xây dựng khái niệm "Agent" trong suốt nhiều thập kỷ. Những mô hình (paradigm) mà ngày nay chúng ta gọi là "Agent truyền thống" không phải là một khái niệm tĩnh đơn lẻ, mà đã trải qua một lộ trình phát triển rõ ràng từ đơn giản đến phức tạp, từ phản ứng bị động đến học tập chủ động.

Điểm khởi đầu của quá trình phát triển này là những <strong>Agent phản xạ đơn giản (Simple Reflex Agent)</strong> có cấu trúc đơn giản nhất. Lõi quyết định của chúng được tạo thành từ các quy tắc "điều kiện-hành động" được kỹ sư thiết kế rõ ràng, như minh họa trong Hình 1.2. Chiếc máy điều nhiệt tự động kinh điển chính là một ví dụ như vậy: nếu nhiệt độ phòng mà cảm biến cảm nhận được cao hơn giá trị cài đặt, thì hệ thống làm lạnh sẽ được kích hoạt.

Loại Agent này hoàn toàn phụ thuộc vào đầu vào cảm nhận hiện tại, không có khả năng ghi nhớ hay dự đoán. Nó giống như một bản năng số hóa, đáng tin cậy và hiệu quả, nhưng cũng vì vậy mà không thể đối phó với các nhiệm vụ phức tạp cần hiểu ngữ cảnh. Hạn chế này đặt ra một câu hỏi then chốt: nếu trạng thái hiện tại của môi trường không đủ làm căn cứ ra quyết định, thì Agent nên làm gì?

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-1.png" alt="图片描述" width="90%"/>
  <p>Hình 1.2 Sơ đồ logic quyết định của Agent phản xạ đơn giản</p>
</div>

Để trả lời câu hỏi này, các nhà nghiên cứu đã đưa ra khái niệm "trạng thái", từ đó phát triển ra <strong>Agent phản xạ dựa trên mô hình (Model-Based Reflex Agent)</strong>. Loại Agent này sở hữu một <strong>mô hình thế giới (World Model)</strong> nội tại, dùng để theo dõi và hiểu những khía cạnh của môi trường mà nó không thể cảm nhận trực tiếp. Nó cố gắng trả lời câu hỏi: "Thế giới hiện đang như thế nào?". Ví dụ, một chiếc xe tự lái đang di chuyển trong hầm, ngay cả khi camera tạm thời không thể cảm nhận được xe phía trước, mô hình nội tại của nó vẫn duy trì phán đoán về sự tồn tại, tốc độ và vị trí ước tính của chiếc xe đó. Mô hình nội tại này mang lại cho Agent "trí nhớ" sơ khai, khiến việc ra quyết định của nó không còn chỉ phụ thuộc vào cảm nhận tức thời, mà dựa trên một sự hiểu biết về trạng thái thế giới mạch lạc và đầy đủ hơn.

Tuy nhiên, chỉ hiểu thế giới thôi là chưa đủ, Agent còn cần có mục tiêu rõ ràng. Điều này đã thúc đẩy sự phát triển của <strong>Agent dựa trên mục tiêu (Goal-Based Agent)</strong>. Khác với hai loại trên, hành vi của nó không còn là phản ứng thụ động trước môi trường, mà là chủ động, có tính tiên liệu lựa chọn những hành động có thể dẫn đến một trạng thái tương lai cụ thể nào đó. Câu hỏi mà loại Agent này cần trả lời là: "Tôi nên làm gì để đạt được mục tiêu?". Ví dụ kinh điển là hệ thống định vị GPS: mục tiêu của bạn là đến công ty, Agent sẽ dựa trên dữ liệu bản đồ (mô hình thế giới), thông qua thuật toán tìm kiếm (như thuật toán A*) để lập kế hoạch (Planning) một tuyến đường tối ưu. Năng lực cốt lõi của loại Agent này thể hiện ở khả năng cân nhắc và lập kế hoạch cho tương lai.

Xa hơn nữa, mục tiêu trong thế giới thực thường không chỉ có một. Chúng ta không chỉ muốn đến công ty, mà còn muốn thời gian ngắn nhất, quãng đường tiết kiệm nhiên liệu nhất và tránh được tắc đường. Khi cần cân nhắc giữa nhiều mục tiêu, <strong>Agent dựa trên độ thỏa dụng (Utility-Based Agent)</strong> sẽ xuất hiện. Nó gán cho mỗi trạng thái thế giới khả dĩ một giá trị thỏa dụng (utility), giá trị này đại diện cho mức độ hài lòng. Mục tiêu cốt lõi của Agent không còn đơn thuần là đạt được một trạng thái cụ thể nào đó, mà là tối đa hóa độ thỏa dụng kỳ vọng. Nó cần trả lời một câu hỏi phức tạp hơn: "Hành vi nào sẽ mang lại cho tôi kết quả hài lòng nhất?". Kiến trúc này giúp Agent học được cách cân nhắc giữa các mục tiêu xung đột lẫn nhau, khiến cho quyết định của nó tiệm cận hơn với sự lựa chọn lý trí của con người.

Đến đây, mặc dù các Agent mà chúng ta thảo luận có chức năng ngày càng phức tạp, nhưng logic quyết định cốt lõi của chúng, dù là quy tắc, mô hình hay hàm thỏa dụng, vẫn phụ thuộc vào tri thức tiên nghiệm của các nhà thiết kế con người. Nếu Agent có thể không phụ thuộc vào các thiết lập sẵn, mà tự học thông qua tương tác với môi trường thì sao?

Đây chính là tư tưởng cốt lõi của <strong>Agent học tập (Learning Agent)</strong>, và <strong>học tăng cường (Reinforcement Learning, RL)</strong> là con đường tiêu biểu nhất để hiện thực hóa tư tưởng này. Một Agent học tập bao gồm một thành phần hiệu năng (performance element) (tức các loại Agent mà chúng ta đã thảo luận ở trên) và một thành phần học tập (learning element). Thành phần học tập liên tục điều chỉnh chiến lược quyết định của thành phần hiệu năng bằng cách quan sát kết quả do hành động của thành phần hiệu năng trong môi trường mang lại.

Hãy tưởng tượng một AI đang học chơi cờ. Ban đầu nó có thể chỉ đi quân một cách ngẫu nhiên, khi cuối cùng nó thắng một ván, hệ thống sẽ trao cho nó một phần thưởng dương. Thông qua số lượng lớn các ván tự đấu (self-play), thành phần học tập sẽ dần dần phát hiện ra nước đi nào có khả năng dẫn đến chiến thắng cuối cùng cao hơn. AlphaGo Zero là một thành tựu mang tính cột mốc của tư tưởng này. Trong trò chơi cờ vây phức tạp, nó đã khám phá ra nhiều chiến lược hiệu quả vượt ra ngoài tri thức sẵn có của con người thông qua học tăng cường.

Từ chiếc máy điều nhiệt đơn giản, đến chiếc xe có mô hình nội tại, rồi đến hệ thống định vị có khả năng lập kế hoạch tuyến đường, người ra quyết định biết cân nhắc lợi hại, và cuối cùng là người học có thể tự tiến hóa thông qua kinh nghiệm. Con đường phát triển này cho thấy mạch phát triển mà trí tuệ nhân tạo truyền thống đã trải qua trên con đường xây dựng trí tuệ máy móc. Chúng đã đặt nền móng vững chắc và cần thiết để chúng ta hiểu được các mô hình Agent tiên tiến hơn ngày nay.

### 1.1.2 Mô hình mới được dẫn dắt bởi mô hình ngôn ngữ lớn

Sự xuất hiện của các mô hình ngôn ngữ lớn, tiêu biểu là <strong>GPT (Generative Pre-trained Transformer)</strong>, đang thay đổi đáng kể phương pháp xây dựng và ranh giới năng lực của Agent. Agent LLM được dẫn dắt bởi mô hình ngôn ngữ lớn có cơ chế quyết định cốt lõi khác biệt về bản chất so với Agent truyền thống, từ đó mang lại cho nó một loạt đặc tính hoàn toàn mới.

Sự chuyển đổi này có thể thấy rõ qua việc so sánh hai loại Agent ở nhiều khía cạnh như cơ chế cốt lõi, nguồn tri thức, phương thức tương tác, như minh họa trong Bảng 1.1. Nói ngắn gọn, năng lực của Agent truyền thống bắt nguồn từ việc lập trình tường minh và xây dựng tri thức của kỹ sư, hành vi của nó có tính xác định và có giới hạn; còn Agent LLM thì thông qua tiền huấn luyện (pre-training) trên lượng dữ liệu khổng lồ, có được mô hình thế giới ngầm định và năng lực nổi trội (emergent capability) mạnh mẽ, giúp nó có thể ứng phó với các nhiệm vụ phức tạp theo cách linh hoạt và tổng quát hơn.

<div align="center">
  <p>Bảng 1.1 So sánh cốt lõi giữa Agent truyền thống và Agent được dẫn dắt bởi LLM</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-2.png" alt="图片描述" width="90%"/>
</div>

Sự khác biệt này khiến cho Agent LLM có thể xử lý trực tiếp các chỉ thị ngôn ngữ tự nhiên ở cấp độ cao, mơ hồ và chứa nhiều thông tin ngữ cảnh. Hãy lấy một "trợ lý du lịch thông minh" làm ví dụ để minh họa.

Trước khi Agent LLM xuất hiện, việc lên kế hoạch du lịch thường đồng nghĩa với việc người dùng phải chuyển đổi thủ công giữa nhiều ứng dụng chuyên dụng (như thời tiết, bản đồ, trang đặt chỗ), và tự mình đóng vai trò tích hợp thông tin và ra quyết định. Còn một Agent LLM thì có thể tích hợp toàn bộ quy trình này lại. Khi nhận được một chỉ thị mơ hồ như "lên kế hoạch cho một chuyến du lịch Hạ Môn", cách thức làm việc của nó thể hiện những điểm sau:

- <strong>Lập kế hoạch và suy luận</strong>: Agent trước tiên sẽ phân rã mục tiêu cấp cao này thành một chuỗi các nhiệm vụ con hợp logic, ví dụ: `[xác nhận sở thích du lịch] -> [tra cứu thông tin điểm đến] -> [lập bản thảo hành trình] -> [đặt vé và chỗ ở]`. Đây là một quá trình lập kế hoạch nội tại, được dẫn dắt bởi mô hình.
- <strong>Sử dụng công cụ (Tool use)</strong>: Khi thực thi kế hoạch, Agent nhận ra khoảng trống thông tin, sẽ chủ động gọi các công cụ bên ngoài để bổ sung. Ví dụ, nó sẽ gọi giao diện (API) tra cứu thời tiết để lấy thời tiết thực tế, và dựa trên thông tin "dự báo có mưa", nó sẽ có xu hướng đề xuất các hoạt động trong nhà trong kế hoạch tiếp theo.
- <strong>Điều chỉnh động</strong>: Trong quá trình tương tác, Agent sẽ coi phản hồi của người dùng (như "khách sạn này vượt ngân sách") là ràng buộc mới, và dựa vào đó điều chỉnh hành động tiếp theo, tìm kiếm lại và đề xuất các lựa chọn phù hợp với yêu cầu mới. Toàn bộ quy trình "<strong>tra thời tiết → điều chỉnh hành trình → đặt khách sạn</strong>" thể hiện khả năng điều chỉnh hành vi của nó một cách động theo ngữ cảnh.

Tóm lại, chúng ta đang chuyển từ việc phát triển các công cụ tự động hóa chuyên dụng sang xây dựng các hệ thống có thể tự chủ giải quyết vấn đề. Cốt lõi không còn là viết code, mà là dẫn dắt một "bộ não" tổng quát để lập kế hoạch, hành động và học tập.

### 1.1.3 Các loại Agent

Tiếp nối việc ôn lại quá trình phát triển của Agent ở trên, mục này sẽ phân loại Agent theo ba khía cạnh bổ sung cho nhau.

(1) <strong>Phân loại dựa trên kiến trúc quyết định nội tại</strong>

Khía cạnh phân loại thứ nhất là dựa trên mức độ phức tạp của kiến trúc quyết định nội tại của Agent, góc nhìn này được đề xuất một cách có hệ thống trong cuốn sách *Artificial Intelligence: A Modern Approach*<sup>[1]</sup>. Như đã trình bày ở mục 1.1.1, chính lộ trình phát triển của Agent truyền thống đã tạo thành bậc thang phân loại kinh điển nhất, nó bao gồm từ Agent <strong>phản ứng</strong> đơn giản, đến Agent <strong>dựa trên mô hình</strong> có đưa vào mô hình nội tại, rồi đến các Agent <strong>dựa trên mục tiêu</strong> và <strong>dựa trên độ thỏa dụng</strong> có tính tiên liệu hơn. Ngoài ra, <strong>khả năng học tập</strong> là một siêu năng lực (meta-capability) có thể được gán cho tất cả các loại kể trên, giúp chúng tự cải thiện thông qua kinh nghiệm.

(2) <strong>Phân loại dựa trên thời gian và tính phản ứng</strong>

Ngoài độ phức tạp của kiến trúc nội tại, còn có thể phân loại Agent theo khía cạnh thời gian xử lý quyết định. Góc nhìn này quan tâm đến việc Agent hành động ngay lập tức sau khi nhận được thông tin, hay sẽ trải qua quá trình lập kế hoạch cân nhắc kỹ lưỡng rồi mới hành động. Điều này tiết lộ một sự đánh đổi cốt lõi trong thiết kế Agent: sự cân bằng giữa <strong>tính phản ứng (Reactivity)</strong> theo đuổi tốc độ và <strong>tính cân nhắc (Deliberation)</strong> theo đuổi lời giải tối ưu, như minh họa trong Hình 1.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-3.png" alt="图片描述" width="90%"/>
  <p>Hình 1.3 Mối quan hệ giữa thời gian quyết định và chất lượng quyết định của Agent</p>
</div>

- <strong>Agent phản ứng (Reactive Agents)</strong>

Loại Agent này phản hồi gần như tức thời trước các kích thích của môi trường, độ trễ quyết định cực thấp. Chúng thường tuân theo ánh xạ trực tiếp từ cảm nhận đến hành động, không thực hiện hoặc chỉ thực hiện rất ít việc lập kế hoạch cho tương lai. Các Agent <strong>phản xạ đơn giản</strong> và <strong>dựa trên mô hình</strong> đã nêu ở trên đều thuộc loại này.

Ưu điểm cốt lõi của nó nằm ở <strong>tốc độ nhanh, chi phí tính toán thấp</strong>, điều này cực kỳ quan trọng trong các môi trường động cần ra quyết định nhanh. Ví dụ, hệ thống túi khí an toàn của xe phải phản ứng trong vòng mili giây sau khi xảy ra va chạm, bất kỳ độ trễ nào cũng có thể dẫn đến hậu quả nghiêm trọng; tương tự, các robot giao dịch tần suất cao cũng phải dựa vào quyết định phản ứng để nắm bắt các cơ hội thị trường thoáng qua. Tuy nhiên, cái giá của tốc độ này là sự "thiển cận", do thiếu lập kế hoạch dài hạn, Agent phản ứng dễ rơi vào tối ưu cục bộ, khó hoàn thành các nhiệm vụ phức tạp cần phối hợp nhiều bước.

- <strong>Agent lập kế hoạch (Deliberative Agents)</strong>

Trái ngược với Agent phản ứng, Agent lập kế hoạch (hay còn gọi là Agent cân nhắc) sẽ suy nghĩ và lập kế hoạch phức tạp trước khi hành động. Chúng không phản ứng ngay lập tức trước cảm nhận, mà sẽ sử dụng mô hình thế giới nội tại của mình trước, khám phá một cách có hệ thống các khả năng khác nhau trong tương lai, đánh giá hậu quả của các chuỗi hành động khác nhau, nhằm tìm ra con đường tốt nhất để đạt được mục tiêu. Agent <strong>dựa trên mục tiêu</strong> và <strong>dựa trên độ thỏa dụng</strong> là các Agent lập kế hoạch điển hình.

Có thể ví quá trình ra quyết định của nó như một kỳ thủ cờ. Anh ta không chỉ nhìn vào nước đi trước mắt, mà sẽ dự đoán các phản ứng có thể có của đối thủ, và lập kế hoạch cho vài nước, thậm chí hàng chục nước đi tiếp theo. Khả năng cân nhắc kỹ lưỡng này giúp nó có thể xử lý các nhiệm vụ phức tạp, cần tầm nhìn dài hạn, ví dụ như lập một kế hoạch kinh doanh hay lên kế hoạch cho một chuyến du lịch dài ngày. Ưu điểm của chúng nằm ở tính chiến lược và tầm nhìn xa của quyết định. Tuy nhiên, mặt trái của ưu điểm này là chi phí thời gian và tính toán cao. Trong môi trường biến đổi từng giây, khi Agent lập kế hoạch vẫn đang cân nhắc kỹ lưỡng, thời điểm tốt nhất để hành động có thể đã trôi qua từ lâu.

- <strong>Agent lai (Hybrid Agents)</strong>

Các nhiệm vụ phức tạp trong thế giới thực thường vừa cần phản ứng tức thời, vừa cần lập kế hoạch dài hạn. Ví dụ, trợ lý du lịch thông minh mà chúng ta đã đề cập trước đó, vừa cần điều chỉnh đề xuất dựa trên phản hồi tức thời của người dùng (như "khách sạn này quá đắt") (tính phản ứng), vừa cần lập kế hoạch cho một hành trình du lịch hoàn chỉnh kéo dài nhiều ngày (tính lập kế hoạch). Vì vậy, Agent lai ra đời, nhằm kết hợp ưu điểm của cả hai, đạt được sự cân bằng giữa phản ứng và lập kế hoạch.

Một kiến trúc lai kinh điển là thiết kế phân tầng: tầng dưới là một mô-đun phản ứng nhanh, xử lý các tình huống khẩn cấp và hành động cơ bản; tầng trên là một mô-đun lập kế hoạch cẩn trọng, chịu trách nhiệm xây dựng mục tiêu dài hạn. Còn Agent LLM hiện đại thì thể hiện một mô hình lai linh hoạt hơn. Chúng thường hoạt động trong một vòng lặp "suy nghĩ - hành động - quan sát", khéo léo kết hợp cả hai chế độ làm một:

- <strong>Lập kế hoạch (Reasoning)</strong>: Trong giai đoạn "suy nghĩ", LLM phân tích tình huống hiện tại, lập kế hoạch cho hành động hợp lý tiếp theo. Đây là một quá trình cân nhắc (deliberation).
- <strong>Phản ứng (Acting & Observing)</strong>: Trong giai đoạn "hành động" và "quan sát", Agent tương tác với công cụ hoặc môi trường bên ngoài, và nhận được phản hồi ngay lập tức. Đây là một quá trình phản ứng.

Bằng cách này, Agent phân rã một nhiệm vụ lớn cần lập kế hoạch dài hạn thành một chuỗi các vi vòng lặp "lập kế hoạch - phản ứng". Điều này giúp nó vừa có thể linh hoạt ứng phó với các thay đổi tức thời của môi trường, vừa có thể hoàn thành các mục tiêu dài hạn phức tạp thông qua các bước mạch lạc.

<strong>(3) Phân loại dựa trên biểu diễn tri thức</strong>

Đây là một khía cạnh phân loại căn bản hơn, nó tìm hiểu xem tri thức mà Agent dùng để ra quyết định rốt cuộc tồn tại dưới hình thức nào trong "tư duy" của nó. Vấn đề này là trọng tâm của một cuộc tranh luận kéo dài hơn nửa thế kỷ trong lĩnh vực trí tuệ nhân tạo, và đã định hình nên hai nền văn hóa AI hoàn toàn khác biệt.

- <strong>AI phi biểu tượng (Sub-symbolic AI)</strong>

Chủ nghĩa phi biểu tượng, hay còn gọi là chủ nghĩa kết nối (connectionism), cho rằng tri thức không phải là các quy tắc tường minh, mà được phân bố ngầm trong một mạng lưới phức tạp gồm vô số nơ-ron, là các mẫu thống kê được học từ lượng dữ liệu khổng lồ. Mạng nơ-ron (neural network) và học sâu (deep learning) là đại diện của nó.

Như minh họa ở phía bên trái Hình 1.4, AI phi biểu tượng giống như một đứa trẻ đang tập nói. Nó không nhận biết con mèo thông qua việc học các quy tắc như "con mèo có bốn chân, lông mượt, kêu meo meo", mà sau khi nhìn thấy hàng nghìn bức ảnh về mèo, mạng nơ-ron trong não nó có thể nhận ra các mẫu hình ảnh của khái niệm "con mèo". Sức mạnh của phương pháp này nằm ở khả năng nhận dạng mẫu (pattern recognition) và tính bền vững (robustness) trước dữ liệu nhiễu. Nó có thể dễ dàng xử lý dữ liệu phi cấu trúc như hình ảnh, âm thanh, trong khi loại nhiệm vụ này thường rất khó khăn đối với các hệ thống dựa vào quy tắc rõ ràng.

Tuy nhiên, khả năng trực giác mạnh mẽ này cũng đi kèm với tính không minh bạch. Hệ thống phi biểu tượng thường được xem là một <strong>hộp đen (Black Box)</strong>. Nó có thể nhận dạng con mèo trong ảnh với độ chính xác đáng kinh ngạc, nhưng nếu bạn hỏi nó "tại sao bạn cho rằng đây là con mèo?", rất có thể nó không thể đưa ra một lời giải thích hợp logic. Ngoài ra, nó hoạt động không tốt trong các nhiệm vụ suy luận logic thuần túy, đôi khi còn tạo ra những "ảo giác" (hallucination) nghe có vẻ hợp lý nhưng thực tế lại sai.

- <strong>AI biểu tượng (Symbolic AI)</strong>

Khác với AI phi biểu tượng, chủ nghĩa biểu tượng thường được gọi là trí tuệ nhân tạo truyền thống, niềm tin cốt lõi của nó là: trí tuệ bắt nguồn từ các thao tác logic trên ký hiệu (symbol). Ký hiệu ở đây là các thực thể mà con người có thể đọc hiểu (như từ ngữ, khái niệm), thao tác thì tuân theo các quy tắc logic nghiêm ngặt, như minh họa ở giữa Hình 1.4. Điều này giống như một người thủ thư tỉ mỉ, sắp xếp tri thức thế giới thành các kho quy tắc rõ ràng và đồ thị tri thức (knowledge graph).

Ưu điểm chính của nó là tính minh bạch và có thể giải thích được. Do các bước suy luận rõ ràng, quá trình ra quyết định của nó có thể được truy vết đầy đủ, điều này cực kỳ quan trọng trong các lĩnh vực rủi ro cao như tài chính, y tế. Tuy nhiên, "gót chân Achilles" của nó nằm ở sự mong manh: nó phụ thuộc vào một hệ thống quy tắc hoàn chỉnh, nhưng trong thế giới thực đầy mơ hồ và ngoại lệ, bất kỳ tình huống mới nào chưa được bao phủ đều có thể khiến hệ thống thất bại, đây chính là cái gọi là "nút thắt cổ chai trong việc thu nhận tri thức".

- <strong>AI thần kinh-biểu tượng (Neuro-Symbolic AI)</strong>

Trong một thời gian dài, hai phe phái phi biểu tượng và biểu tượng phát triển như hai đường thẳng song song, mỗi bên theo hướng riêng của mình. Để khắc phục hạn chế của hai mô hình nói trên, một tư tưởng "đại hòa giải" bắt đầu nổi lên, đó chính là AI thần kinh-biểu tượng, còn được gọi là chủ nghĩa lai thần kinh-biểu tượng. Mục tiêu của nó là kết hợp ưu điểm của cả hai mô hình, tạo ra một Agent lai vừa có thể học từ dữ liệu như mạng nơ-ron, vừa có thể suy luận logic như hệ thống biểu tượng. Nói cách khác, nó cố gắng kết hợp khả năng nhận dạng mẫu sở trường của chủ nghĩa phi biểu tượng với khả năng suy luận logic sở trường của chủ nghĩa biểu tượng. Lý thuyết hệ thống kép (dual-system theory) mà người đoạt giải Nobel Kinh tế Daniel Kahneman đề xuất trong cuốn sách *Thinking, Fast and Slow* đã cung cấp cho chúng ta một phép ví von tuyệt vời để hiểu về chủ nghĩa thần kinh-biểu tượng<sup>[2]</sup>. Theo thứ tự trình bày từ trái sang phải trong Hình 1.4, mối tương quan này có thể tóm tắt như sau:

- <strong>Hệ thống 1 (System 1)</strong> là kiểu tư duy nhanh, dựa trên trực giác, xử lý song song, tương tự như khả năng nhận dạng mẫu mạnh mẽ của AI phi biểu tượng.
- <strong>Hệ thống 2 (System 2)</strong> là tư duy cân nhắc chậm, có hệ thống, dựa trên logic, giống hệt với quá trình suy luận của AI biểu tượng.
- <strong>AI thần kinh-biểu tượng</strong> thì kết hợp khả năng nhận dạng mẫu của Hệ thống 1 với khả năng suy luận logic của Hệ thống 2, giúp hai hệ thống phối hợp hoạt động với nhau.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-4.png" alt="亚符号主义、符号主义与神经符号主义的关系示意图" width="90%"/>
  <p>Hình 1.4 Mô hình biểu diễn tri thức của chủ nghĩa phi biểu tượng, chủ nghĩa biểu tượng và chủ nghĩa lai thần kinh-biểu tượng</p>
</div>

Trí tuệ của con người thực ra bắt nguồn từ sự phối hợp hoạt động của hai hệ thống này. Tương tự, một AI thực sự bền vững cũng cần có cả hai ưu điểm này. Agent được dẫn dắt bởi mô hình ngôn ngữ lớn là một ví dụ thực hành xuất sắc của chủ nghĩa thần kinh-biểu tượng: lõi của nó là một mạng nơ-ron khổng lồ, mang lại cho nó khả năng nhận dạng mẫu và sinh ngôn ngữ; trong quá trình hoạt động, nó lại sinh ra một chuỗi các bước trung gian có cấu trúc, như suy nghĩ, kế hoạch hay lệnh gọi API, đây đều là những ký hiệu rõ ràng, có thể thao tác được. Bằng cách này, nó kết hợp khả năng nhận dạng mẫu dựa trên mạng nơ-ron với khả năng suy luận logic dựa trên ký hiệu.



## 1.2 Cấu trúc và nguyên lý vận hành của Agent

### 1.2.1 Định nghĩa môi trường nhiệm vụ (Task Environment)

Để hiểu cách Agent vận hành, trước tiên chúng ta phải hiểu <strong>môi trường nhiệm vụ (Task Environment)</strong> mà nó đang hoạt động. Trong lĩnh vực trí tuệ nhân tạo, người ta thường sử dụng <strong>mô hình PEAS</strong> để mô tả chính xác một môi trường nhiệm vụ, tức là phân tích <strong>thước đo hiệu năng (Performance), môi trường (Environment), bộ chấp hành (Actuators) và cảm biến (Sensors)</strong> của nó. Lấy trợ lý du lịch thông minh đã đề cập ở trên làm ví dụ, Bảng 1.2 dưới đây cho thấy cách vận dụng mô hình PEAS để quy định môi trường nhiệm vụ của nó.

<div align="center">
  <p>Bảng 1.2 Mô tả PEAS của trợ lý du lịch thông minh</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-6.png" alt="图片描述" width="90%"/>
</div>


Trong thực tiễn, môi trường số (digital environment) mà Agent LLM hoạt động thể hiện một số đặc tính phức tạp, những đặc tính này ảnh hưởng trực tiếp đến thiết kế của Agent.

Trước hết, môi trường thường là <strong>có thể quan sát một phần (partially observable)</strong>. Ví dụ, khi trợ lý du lịch tra cứu chuyến bay, nó không thể lấy được toàn bộ thông tin ghế ngồi thời gian thực của tất cả các hãng hàng không cùng một lúc. Nó chỉ có thể thấy phần dữ liệu mà API trả về thông qua việc gọi API đặt vé máy bay, điều này yêu cầu Agent phải có khả năng ghi nhớ (nhớ các tuyến bay đã tra cứu) và khám phá (thử các ngày tra cứu khác nhau).

Thứ hai, kết quả của hành động cũng không phải lúc nào cũng xác định. Dựa theo khả năng dự đoán của kết quả, môi trường có thể chia thành <strong>xác định (deterministic)</strong> và <strong>ngẫu nhiên (stochastic)</strong>. Môi trường nhiệm vụ của trợ lý du lịch là một môi trường ngẫu nhiên điển hình. Khi nó tìm kiếm giá vé, giá vé máy bay và số lượng vé còn lại trả về bởi hai lần gọi liên tiếp đều có thể khác nhau, điều này yêu cầu Agent phải có khả năng xử lý sự không chắc chắn, theo dõi thay đổi và ra quyết định kịp thời.

Ngoài ra, trong môi trường còn có thể tồn tại các tác nhân khác, từ đó hình thành môi trường <strong>đa Agent (Multi-agent)</strong>. Đối với trợ lý du lịch, hành vi đặt chỗ của các người dùng khác, các kịch bản tự động hóa khác, thậm chí hệ thống điều chỉnh giá động của hãng hàng không, đều là các "Agent" khác trong môi trường. Hành động của chúng (ví dụ, đặt mất vé khuyến mãi cuối cùng) sẽ trực tiếp thay đổi trạng thái của môi trường mà trợ lý du lịch đang hoạt động, điều này đặt ra yêu cầu cao hơn về khả năng phản hồi nhanh và lựa chọn chiến lược của Agent.

Cuối cùng, hầu như mọi nhiệm vụ đều diễn ra trong môi trường <strong>liên tục (sequential)</strong> và <strong>động (dynamic)</strong>. "Liên tục" có nghĩa là hành động hiện tại sẽ ảnh hưởng đến tương lai; còn "động" có nghĩa là bản thân môi trường có thể thay đổi trong khi Agent đang ra quyết định. Điều này yêu cầu vòng lặp "cảm nhận - suy nghĩ - hành động - quan sát" của Agent phải có khả năng thích ứng nhanh chóng, linh hoạt với một thế giới không ngừng biến đổi.

### 1.2.2 Cơ chế vận hành của Agent

Sau khi định nghĩa môi trường nhiệm vụ mà Agent hoạt động, chúng ta hãy cùng tìm hiểu cơ chế vận hành cốt lõi của nó. Agent không hoàn thành nhiệm vụ trong một lần, mà tương tác với môi trường thông qua một vòng lặp liên tục, cơ chế cốt lõi này được gọi là <strong>Vòng lặp Agent (Agent Loop)</strong>. Như minh họa trong Hình 1.5, vòng lặp này mô tả quá trình tương tác động giữa Agent và môi trường, tạo thành nền tảng cho hành vi tự chủ của nó.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-5.png" alt="图片描述" width="90%"/>
  <p>Hình 1.5 Vòng lặp cơ bản của tương tác giữa Agent và môi trường</p>
</div>

Vòng lặp này chủ yếu bao gồm các giai đoạn liên quan với nhau sau:

1. <strong>Cảm nhận (Perception)</strong>: Đây là điểm khởi đầu của vòng lặp. Agent tiếp nhận thông tin đầu vào từ môi trường thông qua các cảm biến của nó (ví dụ, cổng lắng nghe của API, giao diện nhập liệu của người dùng). Những thông tin này, tức là <strong>quan sát (Observation)</strong>, có thể là chỉ thị ban đầu của người dùng, cũng có thể là phản hồi về sự thay đổi trạng thái môi trường do hành động ở bước trước gây ra.
2. <strong>Suy nghĩ (Thought)</strong>: Sau khi nhận được thông tin quan sát, Agent bước vào giai đoạn quyết định cốt lõi của nó. Đối với Agent LLM, đây thường là một quá trình suy luận nội tại được dẫn dắt bởi mô hình ngôn ngữ lớn. Như hình minh họa, giai đoạn "suy nghĩ" có thể được chia nhỏ hơn thành hai khâu then chốt:
   - <strong>Lập kế hoạch (Planning)</strong>: Agent dựa trên quan sát hiện tại và trí nhớ nội tại của mình, cập nhật sự hiểu biết về nhiệm vụ và môi trường, và xây dựng hoặc điều chỉnh một kế hoạch hành động. Việc này có thể bao gồm việc phân rã mục tiêu phức tạp thành một chuỗi các nhiệm vụ con cụ thể hơn.
   - <strong>Lựa chọn công cụ (Tool Selection)</strong>: Dựa trên kế hoạch hiện tại, Agent lựa chọn công cụ phù hợp nhất để thực thi bước tiếp theo từ kho công cụ khả dụng của mình, và xác định các tham số cụ thể cần thiết để gọi công cụ đó.
3. <strong>Hành động (Action)</strong>: Sau khi ra quyết định xong, Agent thực thi hành động cụ thể thông qua bộ chấp hành (Actuators) của mình. Việc này thường thể hiện qua việc gọi một công cụ đã chọn (như trình thông dịch code, API công cụ tìm kiếm), từ đó tác động lên môi trường, với ý định thay đổi trạng thái của môi trường.

Hành động không phải là điểm kết thúc của vòng lặp. Hành động của Agent sẽ gây ra <strong>sự thay đổi trạng thái (State Change)</strong> của <strong>môi trường (Environment)</strong>, môi trường sau đó sẽ tạo ra một <strong>quan sát (Observation)</strong> mới làm phản hồi kết quả. Quan sát mới này lại sẽ được hệ thống cảm nhận của Agent thu nhận trong vòng lặp tiếp theo, tạo thành một vòng khép kín liên tục "cảm nhận - suy nghĩ - hành động - quan sát". Chính thông qua việc liên tục lặp lại vòng lặp này, Agent từng bước thúc đẩy nhiệm vụ, tiến hóa từ trạng thái ban đầu đến trạng thái mục tiêu.

### 1.2.3 Cảm nhận và hành động của Agent

Trong thực tiễn kỹ thuật, để LLM có thể dẫn dắt hiệu quả vòng lặp này, chúng ta cần một <strong>giao thức tương tác (Interaction Protocol)</strong> rõ ràng để quy định việc trao đổi thông tin giữa nó và môi trường.

Trong nhiều framework Agent hiện đại, giao thức này được thể hiện qua việc định nghĩa cấu trúc cho mỗi đầu ra của Agent. Đầu ra của Agent không còn là một câu trả lời ngôn ngữ tự nhiên đơn lẻ, mà là một đoạn văn bản tuân theo định dạng cụ thể, trong đó thể hiện rõ ràng quá trình suy luận nội tại và quyết định cuối cùng của nó.

Cấu trúc này thường bao gồm hai phần cốt lõi:

- <strong>Thought (Suy nghĩ)</strong>: Đây là "ảnh chụp nhanh" của quyết định nội tại của Agent. Nó trình bày bằng ngôn ngữ tự nhiên cách Agent phân tích tình huống hiện tại, nhìn lại kết quả quan sát ở bước trước, tự phản tư và phân rã vấn đề, và cuối cùng lập kế hoạch cho hành động cụ thể tiếp theo.
- <strong>Action (Hành động)</strong>: Đây là thao tác cụ thể mà Agent quyết định tác động lên môi trường sau khi suy nghĩ, thường được biểu diễn dưới dạng lệnh gọi hàm (function call).

Ví dụ, một Agent đang lập kế hoạch du lịch có thể sinh ra đầu ra được định dạng như sau:

```Bash
Thought: 用户想知道北京的天气。我需要调用天气查询工具。
Action: get_weather("北京")
```

Trường `Action` ở đây tạo thành chỉ thị gửi ra thế giới bên ngoài. Một <strong>bộ phân tích cú pháp (Parser)</strong> bên ngoài sẽ bắt lấy chỉ thị này, và gọi hàm `get_weather` tương ứng.

Sau khi hành động được thực thi, môi trường sẽ trả về một kết quả. Ví dụ, hàm `get_weather` có thể trả về một đối tượng JSON chứa dữ liệu thời tiết chi tiết. Tuy nhiên, dữ liệu máy đọc được thô (raw, như JSON) thường chứa thông tin dư thừa mà LLM không cần quan tâm, và định dạng của nó cũng không phù hợp với thói quen xử lý ngôn ngữ tự nhiên của LLM.

Vì vậy, một trách nhiệm quan trọng của hệ thống cảm nhận là đóng vai trò cảm biến: xử lý và đóng gói đầu ra thô này thành một đoạn văn bản ngôn ngữ tự nhiên ngắn gọn, rõ ràng, tức là quan sát.

```Bash
Observation: 北京当前天气为晴，气温25摄氏度，微风。
```

Đoạn văn bản `Observation` này sẽ được phản hồi lại cho Agent, làm thông tin đầu vào chính cho vòng lặp tiếp theo, để nó tiến hành một vòng `Thought` và `Action` mới.

Tóm lại, thông qua vòng lặp chặt chẽ được tạo thành từ Thought, Action, Observation này, Agent LLM có thể kết hợp hiệu quả khả năng suy luận ngôn ngữ nội tại với thông tin thực tế của môi trường bên ngoài và khả năng thao tác công cụ.

## 1.3 Thực hành: Xây dựng Agent đầu tiên trong 5 phút

Trong các mục trước, chúng ta đã học về môi trường nhiệm vụ của Agent, cơ chế vận hành cốt lõi, cũng như mô hình tương tác `Thought-Action-Observation`. Kiến thức lý thuyết tuy quan trọng, nhưng cách học tốt nhất vẫn là tự tay thực hành. Trong mục này, chúng tôi sẽ hướng dẫn bạn sử dụng vài dòng code Python đơn giản để xây dựng từ đầu một trợ lý du lịch thông minh có thể hoạt động được. Quá trình này sẽ tuân theo vòng lặp lý thuyết mà chúng ta vừa học, giúp bạn cảm nhận trực quan cách một Agent "suy nghĩ" và tương tác với các "công cụ" bên ngoài như thế nào. Hãy cùng bắt đầu nào!

Trong ví dụ này, mục tiêu của chúng ta là xây dựng một trợ lý du lịch thông minh có thể xử lý nhiệm vụ theo từng bước. Nhiệm vụ của người dùng cần giải quyết được định nghĩa là: "Xin chào, hãy giúp tôi tra cứu thời tiết hôm nay ở Bắc Kinh, sau đó dựa trên thời tiết để gợi ý một địa điểm du lịch phù hợp." Để hoàn thành nhiệm vụ này, Agent phải thể hiện khả năng lập kế hoạch logic rõ ràng. Nó cần gọi công cụ tra cứu thời tiết trước, và lấy kết quả quan sát thu được làm căn cứ cho bước tiếp theo. Trong vòng lặp tiếp theo, nó lại gọi công cụ gợi ý địa điểm du lịch, từ đó đưa ra đề xuất cuối cùng.

### 1.3.1 Chuẩn bị

Để có thể truy cập API trên mạng từ chương trình Python, chúng ta cần một thư viện HTTP. `requests` là lựa chọn phổ biến nhất, dễ sử dụng nhất trong cộng đồng Python. `tavily-python` là một client API tìm kiếm AI mạnh mẽ, dùng để lấy kết quả tìm kiếm trên mạng theo thời gian thực, bạn có thể đăng ký trên [trang chủ](https://www.tavily.com/) để lấy API. `openai` là SDK Python chính thức do OpenAI cung cấp, dùng để gọi các dịch vụ mô hình ngôn ngữ lớn như GPT. Vui lòng cài đặt chúng trước bằng lệnh sau:

```bash
pip install requests tavily-python openai
```

(1) Mẫu chỉ thị (Prompt template)

Chìa khóa để dẫn dắt một LLM thực sự nằm ở <strong>kỹ thuật tạo Prompt (Prompt Engineering)</strong>. Chúng ta cần thiết kế một "mẫu chỉ thị", cho LLM biết nó nên đóng vai trò gì, có những công cụ nào, và cách định dạng suy nghĩ và hành động của nó. Đây là "sách hướng dẫn" của Agent chúng ta, nó sẽ được truyền cho LLM dưới dạng `system_prompt`.

```
AGENT_SYSTEM_PROMPT = """
你是一个智能旅行助手。你的任务是分析用户的请求，并使用可用工具一步步地解决问题。

# 可用工具:
- `get_weather(city: str)`: 查询指定城市的实时天气。
- `get_attraction(city: str, weather: str)`: 根据城市和天气搜索推荐的旅游景点。

# 输出格式要求:
你的每次回复必须严格遵循以下格式，包含一对Thought和Action：

Thought: [你的思考过程和下一步计划]
Action: [你要执行的具体行动]

Action的格式必须是以下之一：
1. 调用工具：function_name(arg_name="arg_value")
2. 结束任务：Finish[最终答案]

# 重要提示:
- 每次只输出一对Thought-Action
- Action必须在同一行，不要换行
- 当收集到足够信息可以回答用户问题时，必须使用 Action: Finish[最终答案] 格式结束

请开始吧！
"""
```

(2) Công cụ 1: Tra cứu thời tiết thực tế

Chúng ta sẽ sử dụng dịch vụ tra cứu thời tiết miễn phí `wttr.in`, nó có thể trả về dữ liệu thời tiết của thành phố được chỉ định dưới định dạng JSON. Dưới đây là code triển khai công cụ này:

```python
import requests

def get_weather(city: str) -> str:
    """
    通过调用 wttr.in API 查询真实的天气信息。
    """
    # API端点，我们请求JSON格式的数据
    url = f"https://wttr.in/{city}?format=j1"
    
    try:
        # 发起网络请求
        response = requests.get(url)
        # 检查响应状态码是否为200 (成功)
        response.raise_for_status() 
        # 解析返回的JSON数据
        data = response.json()
        
        # 提取当前天气状况
        current_condition = data['current_condition'][0]
        weather_desc = current_condition['weatherDesc'][0]['value']
        temp_c = current_condition['temp_C']
        
        # 格式化成自然语言返回
        return f"{city}当前天气:{weather_desc}，气温{temp_c}摄氏度"
        
    except requests.exceptions.RequestException as e:
        # 处理网络错误
        return f"错误:查询天气时遇到网络问题 - {e}"
    except (KeyError, IndexError) as e:
        # 处理数据解析错误
        return f"错误:解析天气数据失败，可能是城市名称无效 - {e}"
```

(3) Công cụ 2: Tìm kiếm và gợi ý địa điểm du lịch

Chúng ta sẽ định nghĩa một công cụ mới `get_attraction`, nó sẽ dựa trên thành phố và tình trạng thời tiết để tìm kiếm địa điểm phù hợp trên Internet:

```python
import os
from tavily import TavilyClient

def get_attraction(city: str, weather: str) -> str:
    """
    根据城市和天气，使用Tavily Search API搜索并返回优化后的景点推荐。
    """
    # 1. 从环境变量中读取API密钥
    api_key = os.environ.get("TAVILY_API_KEY")
    if not api_key:
        return "错误:未配置TAVILY_API_KEY环境变量。"

    # 2. 初始化Tavily客户端
    tavily = TavilyClient(api_key=api_key)
    
    # 3. 构造一个精确的查询
    query = f"'{city}' 在'{weather}'天气下最值得去的旅游景点推荐及理由"
    
    try:
        # 4. 调用API，include_answer=True会返回一个综合性的回答
        response = tavily.search(query=query, search_depth="basic", include_answer=True)
        
        # 5. Tavily返回的结果已经非常干净，可以直接使用
        # response['answer'] 是一个基于所有搜索结果的总结性回答
        if response.get("answer"):
            return response["answer"]
        
        # 如果没有综合性回答，则格式化原始结果
        formatted_results = []
        for result in response.get("results", []):
            formatted_results.append(f"- {result['title']}: {result['content']}")
        
        if not formatted_results:
             return "抱歉，没有找到相关的旅游景点推荐。"

        return "根据搜索，为您找到以下信息:\n" + "\n".join(formatted_results)

    except Exception as e:
        return f"错误:执行Tavily搜索时出现问题 - {e}"
```

Cuối cùng, chúng ta đặt tất cả các hàm công cụ vào một dictionary để vòng lặp chính gọi:

```python
# 将所有工具函数放入一个字典，方便后续调用
available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
}
```



### 1.3.2 Kết nối với mô hình ngôn ngữ lớn

Hiện nay, nhiều nhà cung cấp dịch vụ LLM (bao gồm OpenAI, Azure, cũng như nhiều framework dịch vụ mô hình mã nguồn mở như Ollama, vLLM, v.v.) đều tuân theo đặc tả giao diện tương tự như OpenAI API. Việc chuẩn hóa này mang lại sự tiện lợi lớn cho các nhà phát triển. Khả năng ra quyết định tự chủ của Agent bắt nguồn từ LLM. Chúng ta sẽ triển khai một client tổng quát `OpenAICompatibleClient`, nó có thể kết nối đến bất kỳ dịch vụ LLM nào tương thích với đặc tả giao diện OpenAI.

```python
from openai import OpenAI

class OpenAICompatibleClient:
    """
    一个用于调用任何兼容OpenAI接口的LLM服务的客户端。
    """
    def __init__(self, model: str, api_key: str, base_url: str):
        self.model = model
        self.client = OpenAI(api_key=api_key, base_url=base_url)

    def generate(self, prompt: str, system_prompt: str) -> str:
        """调用LLM API来生成回应。"""
        print("正在调用大语言模型...")
        try:
            messages = [
                {'role': 'system', 'content': system_prompt},
                {'role': 'user', 'content': prompt}
            ]
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=False
            )
            answer = response.choices[0].message.content
            print("大语言模型响应成功。")
            return answer
        except Exception as e:
            print(f"调用LLM API时发生错误: {e}")
            return "错误:调用语言模型服务时出错。"
```

Để khởi tạo lớp này, bạn cần cung cấp ba thông tin: `API_KEY`, `BASE_URL` và `MODEL_ID`, giá trị cụ thể phụ thuộc vào nhà cung cấp dịch vụ mà bạn sử dụng (như OpenAI chính thức, Azure, hoặc mô hình cục bộ Ollama, v.v.), nếu tạm thời chưa có cách lấy được, bạn có thể tham khảo [Cấu hình môi trường](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置_vi.md).

### 1.3.3 Thực thi vòng lặp hành động

Vòng lặp chính dưới đây sẽ tích hợp tất cả các thành phần, và thông qua Prompt đã được định dạng để dẫn dắt LLM ra quyết định.

```python
import re

# --- 1. 配置LLM客户端 ---
# 请根据您使用的服务，将这里替换成对应的凭证和地址
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
TAVILY_API_KEY="YOUR_Tavily_KEY"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"

llm = OpenAICompatibleClient(
    model=MODEL_ID,
    api_key=API_KEY,
    base_url=BASE_URL
)

# --- 2. 初始化 ---
user_prompt = "你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。"
prompt_history = [f"用户请求: {user_prompt}"]

print(f"用户输入: {user_prompt}\n" + "="*40)

# --- 3. 运行主循环 ---
for i in range(5): # 设置最大循环次数
    print(f"--- 循环 {i+1} ---\n")
    
    # 3.1. 构建Prompt
    full_prompt = "\n".join(prompt_history)
    
    # 3.2. 调用LLM进行思考
    llm_output = llm.generate(full_prompt, system_prompt=AGENT_SYSTEM_PROMPT)
    # 模型可能会输出多余的Thought-Action，需要截断
    match = re.search(r'(Thought:.*?Action:.*?)(?=\n\s*(?:Thought:|Action:|Observation:)|\Z)', llm_output, re.DOTALL)
    if match:
        truncated = match.group(1).strip()
        if truncated != llm_output.strip():
            llm_output = truncated
            print("已截断多余的 Thought-Action 对")
    print(f"模型输出:\n{llm_output}\n")
    prompt_history.append(llm_output)
    
    # 3.3. 解析并执行行动
    action_match = re.search(r"Action: (.*)", llm_output, re.DOTALL)
    if not action_match:
        observation = "错误: 未能解析到 Action 字段。请确保你的回复严格遵循 'Thought: ... Action: ...' 的格式。"
        observation_str = f"Observation: {observation}"
        print(f"{observation_str}\n" + "="*40)
        prompt_history.append(observation_str)
        continue
    action_str = action_match.group(1).strip()

    if action_str.startswith("Finish"):
        final_answer = re.match(r"Finish\[(.*)\]", action_str).group(1)
        print(f"任务完成，最终答案: {final_answer}")
        break
    
    tool_name = re.search(r"(\w+)\(", action_str).group(1)
    args_str = re.search(r"\((.*)\)", action_str).group(1)
    kwargs = dict(re.findall(r'(\w+)="([^"]*)"', args_str))

    if tool_name in available_tools:
        observation = available_tools[tool_name](**kwargs)
    else:
        observation = f"错误:未定义的工具 '{tool_name}'"

    # 3.4. 记录观察结果
    observation_str = f"Observation: {observation}"
    print(f"{observation_str}\n" + "="*40)
    prompt_history.append(observation_str)
```

Thông qua các bước trên, chúng ta đã xây dựng được một Agent hoàn chỉnh, được dẫn dắt bởi LLM thực sự. Cốt lõi của nó nằm ở sự kết hợp giữa "công cụ" và "kỹ thuật tạo Prompt", đây chính là tinh hoa thiết kế của các framework Agent chủ đạo hiện nay (như LangChain, LlamaIndex, v.v.).

### 1.3.4 Phân tích ví dụ chạy thử

Đầu ra dưới đây thể hiện đầy đủ một quy trình thực thi Agent thành công. Thông qua phân tích ba vòng lặp này, chúng ta có thể thấy rõ ràng các năng lực cốt lõi mà Agent sử dụng để giải quyết vấn đề.

```bash
用户输入: 你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。
========================================
--- 循环 1 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 首先需要获取北京今天的天气情况，之后再根据天气情况来推荐旅游景点。
Action: get_weather(city="北京")

Observation: 北京当前天气:Sunny，气温26摄氏度
========================================      
--- 循环 2 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 现在已经知道了北京今天的天气是晴朗且温度适中，接下来可以基于这个信息来推荐一个适合的旅游景点了。
Action: get_attraction(city="北京", weather="Sunny")

Observation: 北京在晴天最值得去的旅游景点是颐和园，因其美丽的湖景和古建筑。另一个推荐是长城，因其壮观的景观和历史意义。
========================================
--- 循环 3 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 已经获得了两个适合晴天游览的景点建议，现在可以根据这些信息给用户提供满意的答复。
Action: Finish[今天北京的天气是晴朗的，气温26摄氏度，非常适合外出游玩。我推荐您去颐和园欣赏美丽的湖景和古建筑，或者前往长城体验其壮观的景观和深厚的历史意义。希望您有一个愉快的旅行！]

任务完成，最终答案: 今天北京的天气是晴朗的，气温26摄氏度，非常适合外出游玩。我推荐您去颐和园欣赏美丽的湖景和古建筑，或者前往长城体验其壮观的景观和深厚的历史意义。希望您有一个愉快的旅行！
```

Ví dụ trợ lý du lịch đơn giản này đã tập trung minh họa bốn năng lực cơ bản mà Agent dựa trên mô hình `Thought-Action-Observation` sở hữu: phân rã nhiệm vụ, gọi công cụ, hiểu ngữ cảnh và tổng hợp kết quả. Chính nhờ sự lặp lại liên tục của vòng lặp này mà Agent có thể chuyển hóa một ý định mơ hồ của người dùng thành một chuỗi các bước cụ thể, có thể thực thi, và cuối cùng đạt được mục tiêu.

## 1.4 Các mô hình hợp tác trong ứng dụng Agent

Trong mục trước, chúng ta đã hiểu sâu hơn về vòng lặp vận hành nội tại của Agent thông qua việc tự tay xây dựng một Agent. Tuy nhiên, trong các kịch bản ứng dụng rộng hơn, vai trò của chúng ta đang ngày càng chuyển đổi thành người sử dụng và người cộng tác. Dựa trên vai trò và mức độ tự chủ của Agent trong nhiệm vụ, mô hình hợp tác của nó chủ yếu được chia thành hai loại: một là làm công cụ hiệu quả, thâm nhập sâu vào quy trình làm việc của chúng ta; hai là làm cộng tác viên tự chủ, hợp tác với các Agent khác để hoàn thành các mục tiêu phức tạp.

### 1.4.1 Agent với vai trò công cụ của nhà phát triển

Trong mô hình này, Agent được tích hợp sâu vào quy trình làm việc của nhà phát triển, đóng vai trò một công cụ hỗ trợ mạnh mẽ. Nó tăng cường chứ không thay thế vai trò của nhà phát triển, thông qua việc tự động hóa các nhiệm vụ rườm rà, lặp đi lặp lại, giúp nhà phát triển có thể tập trung hơn vào công việc cốt lõi mang tính sáng tạo. Phương thức phối hợp người-máy này đã nâng cao đáng kể hiệu quả và chất lượng của phát triển phần mềm.

Hiện nay, trên thị trường đã xuất hiện nhiều công cụ hỗ trợ lập trình AI xuất sắc, chúng tuy đều có thể nâng cao hiệu quả phát triển, nhưng lại có con đường triển khai và trọng tâm chức năng riêng:

- <strong>GitHub Copilot</strong>: Là một trong những sản phẩm có ảnh hưởng lớn nhất trong lĩnh vực này, Copilot được GitHub và OpenAI hợp tác phát triển. Nó được tích hợp sâu vào các trình soạn thảo chủ đạo như Visual Studio Code, nổi tiếng với khả năng tự động hoàn thành code mạnh mẽ. Khi nhà phát triển viết code, Copilot có thể gợi ý theo thời gian thực cả một dòng, thậm chí cả một khối hàm. Trong những năm gần đây, nó cũng mở rộng khả năng lập trình hội thoại thông qua Copilot Chat, cho phép nhà phát triển giải quyết vấn đề lập trình thông qua chat ngay trong trình soạn thảo.
- <strong>Claude Code</strong>: Claude Code là trợ lý lập trình AI do Anthropic phát triển, nhằm giúp nhà phát triển hoàn thành nhiệm vụ code hiệu quả trong terminal thông qua chỉ thị ngôn ngữ tự nhiên. Nó có thể hiểu cấu trúc toàn bộ codebase, thực hiện các thao tác chỉnh sửa code, kiểm thử và debug, hỗ trợ toàn bộ quy trình phát triển từ mô tả chức năng đến hiện thực hóa code. Claude Code còn cung cấp chế độ không tương tác (headless), phù hợp với CI, pre-commit hooks, build script và các kịch bản tự động hóa khác, mang lại trải nghiệm lập trình command-line mạnh mẽ cho nhà phát triển.
- <strong>Trae</strong>: Là một công cụ lập trình AI mới nổi, Trae tập trung cung cấp dịch vụ sinh code và tối ưu hóa thông minh cho nhà phát triển. Nó phân tích các mẫu code thông qua kỹ thuật học sâu, có thể cung cấp gợi ý code chính xác và phương án tái cấu trúc (refactoring) tự động cho nhà phát triển. Đặc trưng của Trae là thiết kế nhẹ và khả năng phản hồi nhanh, đặc biệt phù hợp với các kịch bản cần lặp lại thường xuyên và phát triển nguyên mẫu (prototype) nhanh.
- <strong>Cursor</strong>: Khác với các công cụ nói trên chủ yếu tồn tại dưới dạng plugin hoặc tính năng tích hợp, Cursor lại chọn một con đường tích hợp hơn, bản thân nó chính là một trình soạn thảo code AI-native. Nó không phải là thêm chức năng AI vào trình soạn thảo có sẵn, mà ngay từ đầu đã coi tương tác AI là trọng tâm thiết kế. Ngoài việc sở hữu khả năng sinh code và chat hàng đầu, nó còn chú trọng hơn vào việc giúp AI hiểu ngữ cảnh của toàn bộ codebase, từ đó thực hiện hỏi đáp, tái cấu trúc và debug ở tầng sâu hơn.

Dĩ nhiên vẫn còn nhiều công cụ xuất sắc khác chưa được liệt kê, nhưng chúng cùng chỉ ra một xu hướng rõ ràng: AI đang thâm nhập sâu vào toàn bộ vòng đời phát triển phần mềm, thông qua việc xây dựng quy trình phối hợp người-máy hiệu quả, đang định hình lại sâu sắc ranh giới hiệu quả và mô hình phát triển của kỹ thuật phần mềm.

### 1.4.2 Agent với vai trò cộng tác viên tự chủ

Khác với việc làm công cụ hỗ trợ con người, mô hình tương tác thứ hai nâng mức độ tự động hóa của Agent lên một tầm cao hoàn toàn mới, đó là cộng tác viên tự chủ. Trong mô hình này, chúng ta không còn hướng dẫn AI từng bước một để hoàn thành mọi việc, mà giao phó một mục tiêu cấp cao cho nó. Agent sẽ giống như một thành viên dự án thực thụ, độc lập lập kế hoạch, suy luận, thực thi và phản tư, cho đến khi giao nộp thành quả cuối cùng. Sự chuyển đổi từ trợ lý sang cộng tác viên này đã đưa Agent LLM tiến sâu hơn vào tầm nhìn của công chúng. Nó đánh dấu mối quan hệ của chúng ta với AI đã tiến hóa từ "ra lệnh - thực thi" sang "mục tiêu - ủy thác". Agent không còn là một công cụ bị động, mà là một người theo đuổi mục tiêu chủ động.

Hiện nay, ý tưởng hiện thực hóa sự hợp tác tự chủ này đang nở rộ, xuất hiện rất nhiều framework và sản phẩm xuất sắc, từ BabyAGI, AutoGPT ở giai đoạn đầu, đến các framework ngày càng hoàn thiện hiện nay như CrewAI, AutoGen, MetaGPT, LangGraph, cùng nhau thúc đẩy sự phát triển nhanh chóng của lĩnh vực này. Mặc dù cách triển khai cụ thể rất khác nhau, nhưng mô hình kiến trúc của chúng về cơ bản có thể được quy về một vài hướng chủ đạo:

1. <strong>Vòng lặp tự chủ đơn Agent</strong>: Đây là mô hình điển hình ở giai đoạn đầu, như mô hình mà <strong>AgentGPT</strong> đại diện. Cốt lõi của nó là một Agent tổng quát thông qua vòng khép kín "suy nghĩ - lập kế hoạch - thực thi - phản tư", liên tục tự nhắc nhở (self-prompting) và lặp lại để hoàn thành một mục tiêu cấp cao mở (open-ended).
2. <strong>Hợp tác đa Agent</strong>: Đây là hướng khám phá chủ đạo nhất hiện nay, nhằm giải quyết các vấn đề phức tạp thông qua việc mô phỏng mô hình hợp tác của đội nhóm con người. Nó có thể được chia nhỏ hơn thành các mô hình khác nhau: <strong>Hội thoại nhập vai</strong>: như framework <strong>CAMEL</strong>, thông qua việc đặt vai trò và giao thức giao tiếp rõ ràng cho hai Agent (ví dụ, "lập trình viên" và "quản lý sản phẩm"), để chúng cùng nhau hoàn thành nhiệm vụ trong một cuộc hội thoại có cấu trúc. <strong>Quy trình làm việc có tổ chức</strong>: như <strong>MetaGPT</strong> và <strong>CrewAI</strong>, chúng mô phỏng một "đội nhóm ảo" phân công rõ ràng (như công ty phần mềm hoặc nhóm tư vấn). Mỗi Agent đều có trách nhiệm và quy trình làm việc (SOP) được định sẵn, thông qua cách thức phân cấp hoặc tuần tự để hợp tác, tạo ra thành quả phức tạp chất lượng cao (như một codebase hoàn chỉnh hay báo cáo nghiên cứu). <strong>AutoGen</strong> và <strong>AgentScope</strong> thì cung cấp mô hình hội thoại linh hoạt hơn, cho phép nhà phát triển tùy chỉnh mạng lưới tương tác phức tạp giữa các Agent.
3. <strong>Kiến trúc luồng điều khiển nâng cao</strong>: Các framework như <strong>LangGraph</strong>, chú trọng hơn vào việc cung cấp nền tảng kỹ thuật cơ sở mạnh mẽ hơn cho Agent. Nó mô hình hóa quá trình thực thi của Agent thành đồ thị trạng thái (State Graph), từ đó có thể hiện thực hóa một cách linh hoạt và đáng tin cậy hơn các luồng phức tạp như vòng lặp, phân nhánh, quay lui và sự can thiệp của con người.

Các mô hình kiến trúc khác nhau này cùng nhau thúc đẩy Agent tự chủ đi từ ý tưởng lý thuyết đến ứng dụng thực tế rộng rãi hơn, giúp nó có khả năng ứng phó với các nhiệm vụ thế giới thực ngày càng phức tạp. Trong các chương tiếp theo, chúng ta cũng sẽ cảm nhận được sự khác biệt và ưu điểm giữa các loại framework khác nhau.

### 1.4.3 Sự khác biệt giữa Workflow và Agent

Sau khi hiểu hai mô hình "công cụ" và "cộng tác viên" của Agent, chúng ta cần thảo luận về sự khác biệt giữa Workflow và Agent, mặc dù cả hai đều nhằm mục đích tự động hóa nhiệm vụ, nhưng logic nền tảng, đặc trưng cốt lõi và kịch bản áp dụng của chúng lại hoàn toàn khác nhau.

Nói một cách đơn giản, <strong>Workflow là để AI thực thi chỉ thị theo từng bước đã định sẵn, còn Agent thì trao cho AI quyền tự do để tự chủ đạt được mục tiêu.</strong>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-18.png" alt="图片描述" width="90%"/>
  <p>Hình 1.6 Sự khác biệt giữa Workflow và Agent</p>
</div>

Như minh họa trong Hình 1.6, Workflow là một mô hình tự động hóa truyền thống, cốt lõi của nó là <strong>việc sắp xếp được định nghĩa trước, có cấu trúc đối với một chuỗi nhiệm vụ hoặc bước thực hiện</strong>. Về bản chất, nó là một sơ đồ quy trình chính xác, tĩnh, quy định rõ trong điều kiện nào, theo thứ tự nào thì thực hiện thao tác gì. Một ví dụ điển hình: quy trình phê duyệt hoàn ứng chi phí của một doanh nghiệp. Nhân viên nộp đơn hoàn ứng (kích hoạt) -> nếu số tiền nhỏ hơn 500 nghìn đồng, trực tiếp do trưởng phòng phê duyệt -> nếu số tiền lớn hơn 500 nghìn đồng, trước tiên do trưởng phòng phê duyệt, sau đó chuyển đến giám đốc tài chính phê duyệt -> sau khi phê duyệt, thông báo cho phòng tài chính giải ngân. Mỗi bước, mỗi điều kiện phán đoán trong toàn bộ quá trình này đều được thiết lập sẵn một cách chính xác.

Khác với Workflow, Agent dựa trên mô hình ngôn ngữ lớn là một <strong>hệ thống có tính tự chủ, hướng đến mục tiêu</strong>. Nó không chỉ đơn thuần thực thi chỉ thị đã định sẵn, mà có thể hiểu môi trường ở một mức độ nhất định, tiến hành suy luận, lập kế hoạch, và chủ động thực hiện hành động để đạt được mục tiêu cuối cùng. LLM đóng vai trò "bộ não" trong đó. Một ví dụ điển hình chính là trợ lý du lịch thông minh mà chúng ta đã viết ở mục 1.3. Khi chúng ta đưa ra một chỉ thị mới cho nó, ví dụ: <strong>"Xin chào, hãy giúp tôi tra cứu thời tiết hôm nay ở Bắc Kinh, sau đó dựa trên thời tiết để gợi ý một địa điểm du lịch phù hợp."</strong> Quá trình xử lý của nó thể hiện đầy đủ tính tự chủ của nó:

1. <strong>Lập kế hoạch và gọi công cụ:</strong> Agent trước tiên sẽ phân rã nhiệm vụ thành hai bước: ① tra cứu thời tiết; ② gợi ý địa điểm dựa trên thời tiết. Sau đó, nó sẽ tự chủ lựa chọn và gọi "API tra cứu thời tiết", và truyền "Bắc Kinh" làm tham số.
2. <strong>Suy luận và ra quyết định:</strong> Giả sử API trả về kết quả là "trời quang, gió nhẹ". Bộ não LLM của Agent sẽ suy luận dựa trên thông tin này: "trời nắng phù hợp với hoạt động ngoài trời". Tiếp theo, nó sẽ dựa trên phán đoán này, trong kho tri thức của nó hoặc thông qua công cụ công cụ tìm kiếm, lọc ra các địa điểm ngoài trời ở Bắc Kinh, như Cố Cung, Di Hòa Viên, Công viên Thiên Đàn, v.v.
3. <strong>Sinh kết quả:</strong> Cuối cùng, Agent sẽ tổng hợp thông tin, đưa ra một câu trả lời hoàn chỉnh, mang tính nhân văn: "Hôm nay thời tiết Bắc Kinh trời quang, gió nhẹ, rất phù hợp để vui chơi ngoài trời. Xin gợi ý bạn đến [Di Hòa Viên], bạn có thể chèo thuyền trên hồ Côn Minh, ngắm nhìn cảnh sắc tuyệt đẹp của khu vườn hoàng gia."

Trong quá trình này, không hề có bất kỳ quy tắc cứng nhắc nào kiểu `nếu thời_tiết=trời_nắng thì gợi_ý Di_Hòa_Viên`. Nếu thời tiết là "trời mưa", Agent sẽ tự chủ suy luận và gợi ý các địa điểm trong nhà như Bảo tàng Quốc gia, Bảo tàng Thủ đô. <strong>Khả năng suy luận và ra quyết định động dựa trên thông tin thời gian thực này chính là giá trị cốt lõi của Agent.</strong>



## 1.5 Tổng kết chương

Trong chương này, chúng ta đã cùng nhau bắt đầu hành trình làm quen với Agent. Hành trình của chúng ta bắt đầu từ những câu hỏi cơ bản nhất:

- <strong>Agent được dẫn dắt bởi mô hình ngôn ngữ lớn là gì?</strong> Chúng ta trước tiên đã làm rõ định nghĩa của nó, hiểu rằng Agent hiện đại là một thực thể sở hữu năng lực. Nó không còn chỉ là một kịch bản thực thi chương trình đã định sẵn, mà là một người ra quyết định có thể tự chủ suy luận và sử dụng công cụ.
- <strong>Agent hoạt động như thế nào?</strong> Chúng ta đã đi sâu tìm hiểu cơ chế vận hành của việc Agent tương tác với môi trường. Chúng ta đã biết được rằng, vòng khép kín liên tục này là nền tảng để Agent xử lý thông tin, ra quyết định, tác động lên môi trường và điều chỉnh hành vi của bản thân dựa trên phản hồi.
- <strong>Làm thế nào để xây dựng một Agent?</strong> Đây là trọng tâm thực hành của chương này. Chúng ta đã lấy "trợ lý du lịch thông minh" làm ví dụ, tự tay xây dựng một Agent hoàn chỉnh, được dẫn dắt bởi LLM thực sự.
- <strong>Agent có những mô hình ứng dụng chủ đạo nào?</strong> Cuối cùng, chúng ta đã hướng tầm nhìn ra lĩnh vực ứng dụng rộng lớn hơn. Chúng ta đã thảo luận hai mô hình tương tác chủ đạo của Agent: một là "công cụ dành cho nhà phát triển" đại diện bởi GitHub Copilot và Cursor, nhằm tăng cường quy trình làm việc của con người; hai là "cộng tác viên tự chủ" đại diện bởi các framework như CrewAI, MetaGPT và AgentScope, có thể độc lập hoàn thành các mục tiêu cấp cao. Đồng thời cũng giải thích sự khác biệt giữa Workflow và Agent.

Thông qua việc học chương này, chúng ta đã xây dựng được một khung nhận thức cơ bản về Agent. Vậy nó đã từng bước phát triển như thế nào từ ý tưởng ban đầu cho đến ngày nay? Trong chương tiếp theo, chúng ta sẽ khám phá lịch sử phát triển của Agent, một hành trình truy tìm nguồn gốc sắp bắt đầu!



## Bài tập

> <strong>Gợi ý</strong>: Một số bài tập dưới đây không có đáp án chuẩn, trọng tâm nằm ở việc rèn luyện tư duy phản biện sâu sắc và khả năng thực hành của người học đối với hệ thống Agent.

1. Hãy phân tích xem <strong>chủ thể</strong> trong bốn `case` sau đây có phải là Agent hay không, nếu có, thì thuộc loại Agent nào (có thể phân tích từ nhiều khía cạnh phân loại), và giải thích lý do:

   `case A`: <strong>Một siêu máy tính tuân theo kiến trúc Von Neumann</strong>, có sức mạnh tính toán đỉnh lên đến 2EFlop mỗi giây

   `case B`: <strong>Hệ thống tự lái của Tesla</strong> khi đang di chuyển trên đường cao tốc, đột nhiên phát hiện có chướng ngại vật phía trước, cần đưa ra quyết định phanh hoặc chuyển làn trong mili giây

   `case C`: <strong>AlphaGo</strong> khi đối đầu với kỳ thủ con người, cần đánh giá thế cờ hiện tại và lập kế hoạch cho chiến lược tối ưu trong hàng chục nước đi tiếp theo

   `case D`: <strong>Trợ lý khách hàng thông minh do ChatGPT đóng vai</strong> khi xử lý khiếu nại của người dùng, cần tra cứu thông tin đơn hàng, phân tích nguyên nhân vấn đề, đưa ra giải pháp và xoa dịu cảm xúc người dùng

2. Giả sử bạn cần thiết kế môi trường nhiệm vụ cho một "huấn luyện viên thể hình thông minh". Agent này có khả năng:
   - Theo dõi nhịp tim, cường độ vận động và các dữ liệu sinh lý khác của người dùng thông qua thiết bị đeo được
   - Điều chỉnh động kế hoạch tập luyện dựa trên mục tiêu thể hình của người dùng (giảm mỡ/tăng cơ/tăng sức bền)
   - Cung cấp hướng dẫn bằng giọng nói và sửa động tác theo thời gian thực trong quá trình người dùng vận động
   - Đánh giá hiệu quả tập luyện và đưa ra gợi ý về chế độ ăn uống

   Hãy sử dụng mô hình PEAS để mô tả đầy đủ môi trường nhiệm vụ của Agent này, và phân tích xem môi trường đó có những đặc tính nào (như có thể quan sát một phần, tính ngẫu nhiên, tính động, v.v.).

3. Một công ty thương mại điện tử đang cân nhắc hai phương án để xử lý yêu cầu hoàn tiền sau bán hàng:
   
   Phương án A (`Workflow`): Thiết kế một quy trình cố định, ví dụ:

   A.1 Đối với hàng hóa thông thường và trong vòng 7 ngày, số tiền `< 100 nghìn VNĐ` tự động thông qua; `100-500 nghìn VNĐ` do nhân viên chăm sóc khách hàng xét duyệt; `>500 nghìn VNĐ` cần quản lý phê duyệt; còn hàng hóa đặc biệt (như hàng đặt theo yêu cầu) đều từ chối hoàn tiền

   A.2 Đối với hàng hóa quá 7 ngày, bất kể số tiền bao nhiêu, chỉ có thể do nhân viên chăm sóc khách hàng xét duyệt hoặc quản lý phê duyệt;
   
   Phương án B (`Agent`): Xây dựng một hệ thống Agent, để nó hiểu chính sách hoàn tiền, phân tích hành vi lịch sử của người dùng, đánh giá tình trạng hàng hóa, và tự chủ quyết định có phê duyệt hoàn tiền hay không
   
   Hãy phân tích:
   - Ưu nhược điểm của từng phương án là gì?
   - Trong trường hợp nào thì `Workflow` phù hợp hơn? Trường hợp nào thì `Agent` có ưu thế hơn? Nếu bạn là người phụ trách của công ty thương mại điện tử đó, bạn sẽ thiên về áp dụng phương án nào?
   - Liệu có tồn tại một phương án C, có thể kết hợp hai phương án, phát huy ưu điểm và khắc phục nhược điểm không?
   
4. Dựa trên trợ lý du lịch thông minh ở mục 1.3, hãy suy nghĩ cách thêm vào các chức năng sau (có thể chỉ mô tả ý tưởng thiết kế, cũng có thể thử hiện thực hóa thêm bằng code):

   > <strong>Gợi ý</strong>: Hãy suy nghĩ cách sửa đổi vòng lặp `Thought-Action-Observation` để hiện thực hóa các chức năng này.

   - Thêm một chức năng "trí nhớ", để Agent ghi nhớ sở thích của người dùng (như thích các địa điểm lịch sử văn hóa, phạm vi ngân sách, v.v.)
   - Khi vé vào cửa của địa điểm được gợi ý đã hết, Agent có thể tự động gợi ý phương án thay thế
   - Nếu người dùng từ chối liên tiếp 3 gợi ý, Agent có thể phản tư và điều chỉnh chiến lược gợi ý

5. Lý thuyết "Hệ thống 1" (trực giác nhanh) và "Hệ thống 2" (suy luận chậm) của Kahneman<sup>[2]</sup> đã cung cấp một phép ví von tốt cho AI thần kinh-biểu tượng. Trước tiên hãy hình dung một kịch bản ứng dụng Agent cụ thể trong thực tế, sau đó giải thích trong kịch bản đó:

   > <strong>Gợi ý</strong>: Trợ lý chẩn đoán y tế, chatbot tư vấn pháp lý, hệ thống quản trị rủi ro tài chính, v.v. đều là các kịch bản ứng dụng thường gặp

   - Những nhiệm vụ nào nên do "Hệ thống 1" xử lý?
   - Những nhiệm vụ nào nên do "Hệ thống 2" xử lý?
   - Hai hệ thống này phối hợp với nhau như thế nào để đạt được mục tiêu cuối cùng?

6. Mặc dù hệ thống Agent được dẫn dắt bởi mô hình ngôn ngữ lớn đã thể hiện năng lực mạnh mẽ, nhưng chúng vẫn tồn tại nhiều hạn chế. Hãy phân tích các vấn đề sau:
   - Tại sao Agent hoặc hệ thống Agent đôi khi lại sinh ra "ảo giác" (tạo ra thông tin nghe có vẻ hợp lý nhưng thực tế lại sai)?
   - Trong ví dụ ở mục 1.3, chúng ta đã thiết lập số vòng lặp tối đa là 5 lần. Nếu không có giới hạn này, Agent có thể rơi vào vấn đề gì?
   - Làm thế nào để đánh giá mức độ "thông minh" của một Agent? Chỉ sử dụng chỉ số độ chính xác (accuracy) liệu có đủ không?




## Tài liệu tham khảo

[1] RUSSELL S, NORVIG P. Artificial Intelligence: A Modern Approach[M]. 4th ed. London: Pearson, 2020.

[2] KAHNEMAN D. Thinking, Fast and Slow[M]. New York: Farrar, Straus and Giroux, 2011.

---

## 💬 Thảo luận & Trao đổi

Gặp vấn đề trong quá trình học chương này? Muốn trao đổi kinh nghiệm với các học viên khác?

**📝 Đến khu vực thảo luận GitHub Discussions:**
- [💬 Thảo luận & Hỏi đáp bài tập](https://github.com/datawhalechina/Hello-Agents/discussions)
- Tại đây bạn có thể:
  - ✅ Đặt câu hỏi liên quan đến bài tập
  - ✅ Chia sẻ ý tưởng giải bài của bạn
  - ✅ Trao đổi kinh nghiệm với các học viên khác
  - ✅ Nhận được sự giúp đỡ và phản hồi từ cộng đồng

**💡 Gợi ý:** Cuối mỗi trang cũng có khu vực bình luận, bạn có thể thảo luận trực tiếp trong trang!

---
