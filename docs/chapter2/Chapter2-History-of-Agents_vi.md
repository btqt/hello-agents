# Chương 2 Lịch sử phát triển của Agent

Để hiểu sâu sắc vì sao các Agent hiện đại lại có hình thái như ngày nay, cũng như nguồn gốc của các tư tưởng thiết kế cốt lõi của chúng, chương này sẽ hồi cố lại lịch sử: xuất phát từ thời kỳ cổ điển của lĩnh vực trí tuệ nhân tạo, khám phá xem "trí tuệ" sớm nhất đã được định nghĩa như thế nào trong hệ thống quy tắc logic và ký hiệu; tiếp đó chứng kiến bước ngoặt quan trọng từ mô hình trí tuệ đơn nhất, tập trung sang tư tưởng trí tuệ phân tán, hợp tác; cuối cùng là hiểu rõ cách mà paradigm "học" (learning) đã thay đổi triệt để phương thức Agent có được năng lực, và từ đó tạo ra các Agent hiện đại mà chúng ta thấy ngày nay.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-00.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.1 Bậc thang tiến hóa của AI Agent</p>
</div>

Như Hình 2.1 thể hiện, <strong>mỗi paradigm mới xuất hiện đều nhằm giải quyết "điểm đau" (pain point) cốt lõi hoặc hạn chế căn bản của paradigm thế hệ trước.</strong> Trong khi giải pháp mới mang lại bước nhảy vọt về năng lực, nó cũng đồng thời đưa vào những "hạn chế" mới mà ở thời điểm đó khó có thể khắc phục, và điều này lại đặt nền móng cho sự ra đời của paradigm thế hệ tiếp theo. Việc hiểu rõ quá trình lặp "do vấn đề thúc đẩy" (problem-driven) này sẽ giúp chúng ta nắm bắt sâu sắc hơn các nguyên nhân sâu xa và tính tất yếu lịch sử đằng sau việc lựa chọn công nghệ cho Agent hiện đại.

## 2.1 Agent sơ khai dựa trên ký hiệu và logic

Những khám phá sơ khai trong lĩnh vực trí tuệ nhân tạo chịu ảnh hưởng sâu sắc từ logic toán học và các nguyên lý cơ bản của khoa học máy tính. Trong thời kỳ đó, các nhà nghiên cứu phổ biến tin rằng: trí tuệ của con người, đặc biệt là năng lực suy luận logic, có thể được nắm bắt và tái hiện bằng một hệ thống ký hiệu hình thức hóa. Tư tưởng cốt lõi này đã khai sinh paradigm quan trọng đầu tiên của trí tuệ nhân tạo — Chủ nghĩa ký hiệu (Symbolicism), còn được gọi là "AI logic" hay "AI truyền thống".

Theo chủ nghĩa ký hiệu, cốt lõi của hành vi thông minh là thao tác trên các ký hiệu dựa trên một tập quy tắc rõ ràng. Do đó, một Agent có thể được xem là một hệ thống ký hiệu vật lý (physical symbol system): nó biểu diễn thế giới bên ngoài thông qua các ký hiệu nội tại, và lập kế hoạch hành động thông qua suy luận logic. "Trí tuệ" của Agent trong thời kỳ này hoàn toàn đến từ tri thức (knowledge base) và quy tắc suy luận (reasoning rules) được người thiết kế mã hóa sẵn, chứ không phải thông qua học tập tự chủ mà có được.

### 2.1.1 Giả thuyết hệ thống ký hiệu vật lý

Cơ sở lý thuyết của thời kỳ chủ nghĩa ký hiệu là <strong>Giả thuyết hệ thống ký hiệu vật lý (Physical Symbol System Hypothesis, PSSH)</strong><sup>[1]</sup> do <strong>Allen Newell</strong> và <strong>Herbert A. Simon</strong> cùng đề xuất vào năm 1976. Hai chủ nhân giải thưởng Turing này, thông qua giả thuyết này, đã cung cấp định hướng lý thuyết và tiêu chuẩn đánh giá cho việc hiện thực hóa trí tuệ nhân tạo tổng quát trên máy tính.

Giả thuyết này bao gồm hai luận điểm cốt lõi:

1. <strong>Luận điểm về tính đầy đủ (Sufficiency)</strong>: Bất kỳ hệ thống ký hiệu vật lý nào cũng đều có đủ phương tiện để tạo ra hành vi thông minh tổng quát.
2. <strong>Luận điểm về tính tất yếu (Necessity)</strong>: Bất kỳ hệ thống nào có thể thể hiện hành vi thông minh tổng quát thì bản chất của nó nhất định phải là một hệ thống ký hiệu vật lý.

Hệ thống ký hiệu vật lý ở đây chỉ một hệ thống có thể tồn tại trong thế giới vật lý, được cấu thành từ một tập các ký hiệu có thể phân biệt và một loạt các quá trình thao tác trên các ký hiệu đó, với các thành phần cấu thành được minh họa trong Hình 2.2. Các ký hiệu này có thể kết hợp thành các cấu trúc phức tạp hơn (ví dụ như biểu thức), còn các quá trình thì có thể tạo, sửa đổi, sao chép và hủy bỏ các cấu trúc ký hiệu đó.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-0.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.2 Các thành phần cấu thành của hệ thống ký hiệu vật lý</p>
</div>

Nói ngắn gọn, PSSH tuyên bố một cách táo bạo rằng: <strong>bản chất của trí tuệ chính là việc tính toán và xử lý ký hiệu.</strong>

Giả thuyết này có ảnh hưởng sâu rộng. Nó đã chuyển hóa vấn đề triết học mơ hồ, phức tạp về tâm trí con người thành một vấn đề cụ thể có thể được hiện thực hóa về mặt kỹ thuật trên máy tính. Nó đã tiêm vào các nhà nghiên cứu AI sơ khai một niềm tin mạnh mẽ rằng, chỉ cần tìm được cách biểu diễn tri thức đúng đắn và thiết kế được thuật toán suy luận hiệu quả, chắc chắn có thể tạo ra một cỗ máy trí tuệ sánh ngang con người. Toàn bộ nghiên cứu của thời kỳ chủ nghĩa ký hiệu, từ hệ chuyên gia (expert system) cho đến lập kế hoạch tự động, hầu như đều được triển khai dưới sự dẫn dắt của giả thuyết này.

### 2.1.2 Hệ chuyên gia

Dưới ảnh hưởng trực tiếp của giả thuyết hệ thống ký hiệu vật lý, <strong>Hệ chuyên gia (Expert System)</strong> đã trở thành thành tựu ứng dụng quan trọng nhất, thành công nhất của thời kỳ chủ nghĩa ký hiệu. Mục tiêu cốt lõi của hệ chuyên gia là mô phỏng năng lực giải quyết vấn đề của chuyên gia con người trong một lĩnh vực cụ thể. Nó mã hóa tri thức và kinh nghiệm của chuyên gia thành chương trình máy tính, giúp hệ thống có thể đưa ra kết luận hoặc đề xuất sánh ngang, thậm chí vượt trội chuyên gia con người khi đối mặt với các vấn đề tương tự.

Một hệ chuyên gia điển hình thường được cấu thành từ các thành phần cốt lõi như cơ sở tri thức (knowledge base), máy suy luận (inference engine), giao diện người dùng, với kiến trúc tổng quát được minh họa trong Hình 2.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-1.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.3 Kiến trúc tổng quát của hệ chuyên gia</p>
</div>


Kiến trúc này thể hiện rõ ràng tư tưởng thiết kế tách biệt giữa tri thức và suy luận, là đặc trưng quan trọng của AI chủ nghĩa ký hiệu.

<strong>Cơ sở tri thức và máy suy luận</strong>

"Trí tuệ" của hệ chuyên gia chủ yếu bắt nguồn từ hai thành phần cốt lõi: cơ sở tri thức và máy suy luận.

- <strong>Cơ sở tri thức (Knowledge Base)</strong>: Đây là trung tâm lưu trữ tri thức của hệ chuyên gia, dùng để lưu trữ tri thức và kinh nghiệm của chuyên gia trong lĩnh vực. <strong>Biểu diễn tri thức (Knowledge Representation)</strong> là yếu tố then chốt để xây dựng cơ sở tri thức. Trong hệ chuyên gia, phương pháp biểu diễn tri thức thường dùng nhất là <strong>quy tắc sản xuất (Production Rules)</strong>, tức một loạt các câu điều kiện dạng "IF-THEN". Ví dụ: IF bệnh nhân có triệu chứng sốt AND ho THEN có thể bị nhiễm trùng đường hô hấp. Các quy tắc này liên kết một tình huống cụ thể (phần IF, điều kiện) với kết luận hoặc hành động tương ứng (phần THEN, kết luận). Một hệ chuyên gia phức tạp có thể bao gồm hàng trăm, hàng nghìn quy tắc như vậy, cùng nhau tạo thành một mạng lưới tri thức đồ sộ.
- <strong>Máy suy luận (Inference Engine)</strong>: Máy suy luận là động cơ tính toán cốt lõi của hệ chuyên gia. Đây là một chương trình tổng quát, có nhiệm vụ dựa trên các sự kiện do người dùng cung cấp, tìm kiếm và áp dụng các quy tắc liên quan trong cơ sở tri thức, từ đó suy ra kết luận mới. Máy suy luận có hai phương thức hoạt động chủ yếu:
  - <strong>Suy luận tiến (Forward Chaining)</strong>: Xuất phát từ các sự kiện đã biết, liên tục đối sánh phần IF của các quy tắc, kích hoạt kết luận ở phần THEN, và thêm kết luận mới vào cơ sở sự kiện, cho đến khi cuối cùng suy ra được mục tiêu hoặc không còn quy tắc mới nào khớp. Đây là phương thức suy luận "hướng theo dữ liệu" (data-driven).
  - <strong>Suy luận lùi (Backward Chaining)</strong>: Xuất phát từ một mục tiêu giả định (ví dụ "bệnh nhân có bị viêm phổi hay không"), tìm kiếm các quy tắc có thể suy ra mục tiêu đó, sau đó lấy phần IF của quy tắc đó làm mục tiêu con mới, đệ quy như vậy cho đến khi tất cả các mục tiêu con đều được chứng minh bởi các sự kiện đã biết. Đây là phương thức suy luận "hướng theo mục tiêu" (goal-driven).

<strong>Nghiên cứu điển hình và phân tích: Hệ thống MYCIN</strong>

MYCIN là một trong những hệ chuyên gia nổi tiếng và có ảnh hưởng nhất trong lịch sử, được Đại học Stanford phát triển vào những năm 1970<sup>[2]</sup>. Nó được thiết kế để hỗ trợ bác sĩ chẩn đoán nhiễm trùng máu do vi khuẩn và đề xuất phác đồ điều trị kháng sinh phù hợp.

- <strong>Nguyên lý hoạt động</strong>: MYCIN thu thập triệu chứng, bệnh sử và kết quả xét nghiệm của bệnh nhân thông qua tương tác hỏi đáp với bác sĩ. Cơ sở tri thức của nó bao gồm khoảng 600 quy tắc "IF-THEN" do các chuyên gia y tế cung cấp. Máy suy luận chủ yếu hoạt động theo phương thức suy luận lùi: xuất phát từ mục tiêu cao nhất là "xác định vi khuẩn gây bệnh", suy luận ngược lại xem cần những bằng chứng và điều kiện gì, sau đó đặt câu hỏi cho bác sĩ để thu thập các thông tin đó. Quy trình hoạt động đơn giản hóa của nó được minh họa trong Hình 2.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-2.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.4 Sơ đồ minh họa quy trình suy luận lùi của MYCIN</p>
</div>

- <strong>Xử lý tính bất định</strong>: Chẩn đoán y khoa chứa đầy tính bất định. Một cải tiến quan trọng của MYCIN là việc đưa vào khái niệm <strong>hệ số tin cậy (Certainty Factor, CF)</strong>, dùng một giá trị số nằm giữa -1 và 1 để biểu diễn mức độ tin cậy của một kết luận. Điều này giúp hệ thống có thể xử lý tri thức y khoa bất định, mơ hồ, và đưa ra kết quả chẩn đoán kèm theo đánh giá độ tin cậy, gần với thế giới thực hơn so với logic Boolean đơn giản.
- <strong>Thành tựu và ý nghĩa</strong>: Trong một đánh giá, hiệu quả chẩn đoán nhiễm trùng máu của MYCIN đã vượt qua các bác sĩ không chuyên khoa, thậm chí đạt đến trình độ chuyên gia con người. Thành công của nó đã chứng minh một cách thuyết phục tính hiệu quả của giả thuyết hệ thống ký hiệu vật lý: thông qua kỹ thuật tri thức (knowledge engineering) tỉ mỉ và suy luận ký hiệu, máy móc quả thực có thể thể hiện "trí tuệ" xuất sắc trong lĩnh vực chuyên môn có độ phức tạp cao. MYCIN không chỉ là một cột mốc trong lịch sử phát triển hệ chuyên gia, mà còn mở đường cho các ứng dụng thương mại hóa của AI trong nhiều lĩnh vực dọc (vertical) sau này.

### 2.1.3 SHRDLU

Nếu hệ chuyên gia thể hiện "chiều sâu" của AI ký hiệu trong lĩnh vực chuyên môn, thì dự án SHRDLU<sup>[3]</sup> do <strong>Terry Winograd</strong> phát triển trong giai đoạn 1968-1970 lại đạt được bước đột phá mang tính cách mạng về "chiều rộng". Như minh họa trong Hình 2.5, SHRDLU nhằm mục đích xây dựng một Agent tổng hợp có thể tương tác trôi chảy với con người bằng ngôn ngữ tự nhiên trong môi trường vi mô "thế giới khối gỗ" (Blocks World). "Thế giới khối gỗ" là một không gian ảo ba chiều được mô phỏng, trong đó chứa các khối gỗ có hình dạng, màu sắc và kích thước khác nhau, cùng với một cánh tay máy ảo có thể cầm nắm và di chuyển chúng. Người dùng ra lệnh hoặc đặt câu hỏi cho SHRDLU bằng ngôn ngữ tự nhiên, SHRDLU sẽ thực hiện hành động trong thế giới ảo hoặc đưa ra câu trả lời bằng văn bản.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-3.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.5 Giao diện tương tác "thế giới khối gỗ" của SHRDLU</p>
</div>

SHRDLU thu hút sự chú ý rộng rãi vào thời điểm đó, nguyên nhân chủ yếu nằm ở việc nó lần đầu tiên tích hợp nhiều module AI độc lập (như phân tích ngôn ngữ, lập kế hoạch, bộ nhớ) vào một hệ thống thống nhất, và khiến chúng phối hợp hoạt động:

- <strong>Hiểu ngôn ngữ tự nhiên</strong>: SHRDLU có thể phân tích các câu tiếng Anh có cấu trúc phức tạp và chứa tính nhập nhằng (ambiguity). Nó không chỉ hiểu được các mệnh lệnh trực tiếp (như `Pick up a big red block.`), mà còn có thể xử lý các mệnh lệnh phức tạp hơn, ví dụ:
  - Giải quyết tham chiếu (reference resolution): `Find a block which is taller than the one you are holding and put it into the box.` Trong mệnh lệnh này, hệ thống cần hiểu `the one you are holding` ám chỉ vật thể mà cánh tay máy đang cầm nắm hiện tại.
  - Ghi nhớ ngữ cảnh: Người dùng có thể nói `Grasp the pyramid.`, sau đó tiếp tục hỏi `What does the box contain?`, hệ thống có thể liên hệ ngữ cảnh để trả lời.
- <strong>Lập kế hoạch và hành động</strong>: Sau khi hiểu mệnh lệnh, SHRDLU có thể tự chủ lập kế hoạch một loạt hành động cần thiết để hoàn thành nhiệm vụ. Ví dụ, nếu mệnh lệnh là "đặt khối gỗ màu xanh lên trên khối gỗ màu đỏ", mà trên khối gỗ màu đỏ đã có sẵn một khối gỗ màu xanh lá khác, hệ thống sẽ lập kế hoạch chuỗi hành động "di chuyển khối gỗ xanh lá ra trước, sau đó mới đặt khối gỗ xanh dương lên".
- <strong>Bộ nhớ và hỏi đáp</strong>: SHRDLU sở hữu bộ nhớ về môi trường nó đang ở và hành vi của chính nó. Người dùng có thể đặt câu hỏi về điều này, ví dụ:
  - Hỏi về trạng thái thế giới: `Is there a large block behind a pyramid?`
  - Hỏi về lịch sử hành vi: `Did you touch any pyramid before you put the green one on the little cube?`
  - Hỏi về động cơ hành vi: `Why did you pick up the red block?` SHRDLU có thể trả lời: `BECAUSE YOU ASKED ME TO.`

Vị thế lịch sử và ảnh hưởng của SHRDLU chủ yếu thể hiện ở ba khía cạnh:

- <strong>Hình mẫu của trí tuệ tổng hợp</strong>: Trước SHRDLU, nghiên cứu AI phần lớn tập trung vào một chức năng đơn lẻ. Đây là lần đầu tiên nhiều module AI như hiểu ngôn ngữ, suy luận lập kế hoạch và ghi nhớ hành động được tích hợp vào một hệ thống thống nhất, thiết kế vòng lặp khép kín "nhận thức - tư duy - hành động" của nó đã đặt nền móng cho nghiên cứu Agent hiện đại.
- <strong>Phổ biến phương pháp nghiên cứu thế giới vi mô (Micro-world)</strong>: Thành công của nó chứng minh tính khả thi của việc khám phá và kiểm chứng các nguyên lý cơ bản của Agent phức tạp trong một môi trường đơn giản hóa với quy tắc rõ ràng, phương pháp này đã ảnh hưởng sâu sắc đến nghiên cứu robot học và lập kế hoạch AI sau này.
- <strong>Sự lạc quan và phản tư mà nó khơi dậy</strong>: Thành công của SHRDLU đã khơi dậy kỳ vọng lạc quan sớm về AGI, nhưng năng lực của nó lại bị giới hạn nghiêm ngặt trong thế giới khối gỗ. Hạn chế này đã khơi dậy cuộc tranh luận lâu dài trong lĩnh vực AI về sự khác biệt giữa "xử lý ký hiệu" và "hiểu thực sự", từ đó hé lộ những thách thức sâu xa trên con đường hướng đến trí tuệ tổng quát.

### 2.1.4 Những thách thức căn bản mà chủ nghĩa ký hiệu phải đối mặt

Mặc dù các dự án sơ khai đã đạt được thành tựu đáng kể, nhưng từ những năm 1980, khi AI chủ nghĩa ký hiệu đi từ "thế giới vi mô" bước ra thế giới thực mở và phức tạp, nó đã gặp phải những khó khăn căn bản vốn có trong phương pháp luận của mình. Những khó khăn này có thể quy về hai nhóm lớn:

<strong>(1) Tri thức thông thường và nút thắt cổ chai trong thu thập tri thức</strong>

"Trí tuệ" của Agent chủ nghĩa ký hiệu hoàn toàn phụ thuộc vào chất lượng và tính đầy đủ của cơ sở tri thức. Tuy nhiên, việc xây dựng một cơ sở tri thức có thể hỗ trợ tương tác trong thế giới thực đã được chứng minh là một nhiệm vụ vô cùng gian nan, thể hiện chủ yếu ở hai khía cạnh:

- <strong>Nút thắt cổ chai trong thu thập tri thức (Knowledge Acquisition Bottleneck)</strong>: Tri thức của hệ chuyên gia cần được xây dựng bởi các chuyên gia con người và kỹ sư tri thức (knowledge engineer) thông qua quá trình phỏng vấn, chắt lọc và mã hóa rườm rà. Quá trình này tốn kém, kéo dài, và khó mở rộng quy mô. Quan trọng hơn, rất nhiều tri thức của chuyên gia con người là tri thức ẩn (tacit), mang tính trực giác, rất khó diễn đạt rõ ràng thành quy tắc "IF-THEN". Việc cố gắng ký hiệu hóa thủ công toàn bộ tri thức của thế giới được xem là một nhiệm vụ gần như bất khả thi.
- <strong>Vấn đề tri thức thông thường (Common-sense Problem)</strong>: Hành vi của con người phụ thuộc vào một nền tảng tri thức thông thường đồ sộ (ví dụ, "nước thì ướt", "dây thừng có thể kéo nhưng không thể đẩy"), nhưng hệ thống ký hiệu lại hoàn toàn không biết gì về điều này trừ khi được mã hóa rõ ràng. Việc xây dựng một cơ sở tri thức đầy đủ cho tri thức thông thường rộng lớn, mơ hồ cho đến nay vẫn là một thách thức lớn, dự án Cyc<sup>[4]</sup> đã nỗ lực trong hàng chục năm, nhưng thành quả và ứng dụng của nó vẫn còn rất hạn chế.

<strong>(2) Vấn đề khung (Frame Problem) và tính dễ vỡ của hệ thống</strong>

Bên cạnh thách thức về mặt tri thức, chủ nghĩa ký hiệu cũng gặp phải bế tắc về mặt logic khi xử lý một thế giới thay đổi động.

- <strong>Vấn đề khung (Frame Problem)</strong>: Trong một thế giới động, sau khi Agent thực hiện một hành động, việc phán đoán hiệu quả xem những thứ nào không bị thay đổi là một bài toán logic nan giải<sup>[5]</sup>. Việc khai báo tường minh tất cả các trạng thái không đổi cho mỗi hành động là bất khả thi về mặt tính toán, trong khi con người lại có thể dễ dàng bỏ qua những thay đổi không liên quan mà không tốn công sức.
- <strong>Tính dễ vỡ của hệ thống (Brittleness)</strong>: Hệ thống ký hiệu hoàn toàn phụ thuộc vào các quy tắc được định sẵn, khiến hành vi của nó rất "dễ vỡ". Một khi gặp phải bất kỳ thay đổi nhỏ hoặc tình huống mới nào nằm ngoài quy tắc, hệ thống có thể hoàn toàn bị tê liệt, không thể linh hoạt ứng biến như con người. Thành công của SHRDLU cũng chính là vì nó vận hành trong một thế giới khép kín với quy tắc đầy đủ, trong khi thế giới thực lại đầy rẫy những ngoại lệ.

## 2.2 Xây dựng chatbot dựa trên quy tắc

Sau khi thảo luận về các thách thức lý thuyết của chủ nghĩa ký hiệu, trong phần này chúng ta sẽ thông qua một thực hành lập trình cụ thể để cảm nhận trực quan cách thức hoạt động của một hệ thống dựa trên quy tắc (rule-based). Chúng ta sẽ thử tái hiện một trong những chatbot sơ khai có ảnh hưởng lớn nhất trong lịch sử trí tuệ nhân tạo — ELIZA.

### 2.2.1 Tư tưởng thiết kế của ELIZA

ELIZA là một chương trình máy tính được nhà khoa học máy tính <strong>Joseph Weizenbaum</strong> của Học viện Công nghệ Massachusetts (MIT) công bố vào năm 1966<sup>[6]</sup>, là một trong những nỗ lực nổi tiếng sơ khai trong lĩnh vực xử lý ngôn ngữ tự nhiên. ELIZA không phải là một chương trình đơn nhất, mà là một framework có thể thực thi các "kịch bản" (script) khác nhau. Trong đó, kịch bản được biết đến rộng rãi nhất và thành công nhất là "DOCTOR", mô phỏng một nhà trị liệu tâm lý theo trường phái Rogers (non-directive).

Cách thức hoạt động của ELIZA cực kỳ khéo léo: nó không bao giờ trả lời trực tiếp câu hỏi hay cung cấp thông tin, mà thông qua việc nhận diện từ khóa trong đầu vào của người dùng, sau đó áp dụng một tập quy tắc chuyển đổi được định sẵn, chuyển hóa câu nói của người dùng thành một câu hỏi mở. Ví dụ, khi người dùng nói "Tôi cảm thấy buồn vì bạn trai của mình", ELIZA có thể nhận diện từ khóa "Tôi... cảm thấy buồn", và áp dụng quy tắc để sinh ra phản hồi: "Tại sao bạn lại cảm thấy buồn vì bạn trai của mình?"

Tư tưởng thiết kế của Weizenbaum không phải là tạo ra một Agent thực sự "hiểu" được cảm xúc con người, ngược lại, điều ông muốn chứng minh là, thông qua một vài kỹ thuật chuyển đổi câu đơn giản, máy móc có thể tạo ra một ảo giác về "trí tuệ" và "sự đồng cảm" mà hoàn toàn không hiểu nội dung cuộc trò chuyện. Tuy nhiên, ngoài dự kiến của ông, rất nhiều người đã tương tác với ELIZA (bao gồm cả thư ký của ông) đều nảy sinh sự phụ thuộc về mặt cảm xúc, tin chắc rằng nó có thể hiểu được mình.

Mục tiêu thực hành của phần này là tái hiện cơ chế cốt lõi của ELIZA, để hiểu sâu sắc ưu điểm và hạn chế căn bản của phương pháp dựa trên quy tắc này.

### 2.2.2 Khớp mẫu và thay thế văn bản

Quy trình thuật toán của ELIZA dựa trên <strong>khớp mẫu (Pattern Matching) và thay thế văn bản (Text Substitution)</strong>, có thể được phân tách rõ ràng thành bốn bước sau:

1. <strong>Nhận diện và sắp xếp thứ tự ưu tiên từ khóa:</strong>Cơ sở quy tắc đặt một mức độ ưu tiên cho mỗi từ khóa (như `mother`, `dreamed`, `depressed`). Khi đầu vào chứa nhiều từ khóa, chương trình sẽ chọn quy tắc tương ứng với từ khóa có mức độ ưu tiên cao nhất để xử lý.
2. <strong>Quy tắc phân rã:</strong>Sau khi tìm thấy từ khóa, chương trình sử dụng quy tắc phân rã với ký tự đại diện (`*`, wildcard) để thu nạp phần còn lại của câu.
   1. <strong>Ví dụ quy tắc</strong>: `* my *`
   2. <strong>Đầu vào của người dùng</strong>: `"My mother is afraid of me"`
   3. <strong>Kết quả thu nạp</strong>: `["", "mother is afraid of me"]`
3. <strong>Quy tắc tái tổ hợp:</strong>Chương trình chọn một quy tắc từ một tập quy tắc tái tổ hợp liên quan đến quy tắc phân rã để sinh ra phản hồi (thường được chọn ngẫu nhiên để tăng tính đa dạng), và có thể tùy chọn sử dụng nội dung đã thu nạp ở bước trên.
   1. <strong>Ví dụ quy tắc</strong>: `"Tell me more about your family."`
   2. <strong>Đầu ra được sinh ra</strong>: `"Tell me more about your family."`
4. <strong>Chuyển đổi đại từ:</strong>Trước khi tái tổ hợp, chương trình sẽ thực hiện chuyển đổi đại từ đơn giản (như `I` → `you`, `my` → `your`), để duy trì tính liên tục của cuộc hội thoại.

Toàn bộ quy trình làm việc có thể được biểu diễn bằng một đoạn mã giả (pseudo-code) đơn giản như sau:

```Python
FUNCTION generate_response(user_input):
    // 1. Tách đầu vào của người dùng thành các từ
    words = SPLIT(user_input)

    // 2. Tìm quy tắc từ khóa có mức độ ưu tiên cao nhất
    best_rule = FIND_BEST_RULE(words)
    IF best_rule is NULL:
        RETURN a_generic_response() // Ví dụ: "Please go on."

    // 3. Dùng quy tắc để phân rã đầu vào của người dùng
    decomposed_parts = DECOMPOSE(user_input, best_rule.decomposition_pattern)
    IF decomposition_failed:
        RETURN a_generic_response()

    // 4. Chuyển đổi đại từ cho phần đã phân rã
    transformed_parts = TRANSFORM_PRONOUNS(decomposed_parts)

    // 5. Dùng quy tắc tái tổ hợp để sinh ra phản hồi
    response = REASSEMBLE(transformed_parts, best_rule.reassembly_patterns)

    RETURN response
```

Thông qua cơ chế này, ELIZA đã thành công đơn giản hóa bài toán hiểu ngôn ngữ tự nhiên phức tạp thành một trò chơi khớp mẫu dựa trên quy tắc có thể thao tác được.

### 2.2.3 Hiện thực hóa logic cốt lõi

Bây giờ, chúng ta sẽ chuyển hóa nguyên lý kỹ thuật được mô tả ở phần trên thành một hàm Python đơn giản, có thể chạy được. Đoạn code dưới đây hiện thực hóa một phiên bản ELIZA thu nhỏ, chứa một tập hợp nhỏ các quy tắc, nhưng đủ để thể hiện cơ chế hoạt động cốt lõi của nó.

```Python
import re
import random

# Định nghĩa cơ sở quy tắc: mẫu (regex) -> danh sách mẫu phản hồi
rules = {
    r'I need (.*)': [
        "Why do you need {0}?",
        "Would it really help you to get {0}?",
        "Are you sure you need {0}?"
    ],
    r'Why don\'t you (.*)\?': [
        "Do you really think I don't {0}?",
        "Perhaps eventually I will {0}.",
        "Do you really want me to {0}?"
    ],
    r'Why can\'t I (.*)\?': [
        "Do you think you should be able to {0}?",
        "If you could {0}, what would you do?",
        "I don't know -- why can't you {0}?"
    ],
    r'I am (.*)': [
        "Did you come to me because you are {0}?",
        "How long have you been {0}?",
        "How do you feel about being {0}?"
    ],
    r'.* mother .*': [
        "Tell me more about your mother.",
        "What was your relationship with your mother like?",
        "How do you feel about your mother?"
    ],
    r'.* father .*': [
        "Tell me more about your father.",
        "How did your father make you feel?",
        "What has your father taught you?"
    ],
    r'.*': [
        "Please tell me more.",
        "Let's change focus a bit... Tell me about your family.",
        "Can you elaborate on that?"
    ]
}

# Định nghĩa quy tắc chuyển đổi đại từ
pronoun_swap = {
    "i": "you", "you": "i", "me": "you", "my": "your",
    "am": "are", "are": "am", "was": "were", "i'd": "you would",
    "i've": "you have", "i'll": "you will", "yours": "mine",
    "mine": "yours"
}

def swap_pronouns(phrase):
    """
    Chuyển đổi ngôi thứ nhất/thứ hai cho đại từ trong cụm từ đầu vào
    """
    words = phrase.lower().split()
    swapped_words = [pronoun_swap.get(word, word) for word in words]
    return " ".join(swapped_words)

def respond(user_input):
    """
    Sinh ra phản hồi dựa trên cơ sở quy tắc
    """
    for pattern, responses in rules.items():
        match = re.search(pattern, user_input, re.IGNORECASE)
        if match:
            # Thu nạp phần đã khớp
            captured_group = match.group(1) if match.groups() else ''
            # Thực hiện chuyển đổi đại từ
            swapped_group = swap_pronouns(captured_group)
            # Chọn ngẫu nhiên một mẫu và định dạng
            response = random.choice(responses).format(swapped_group)
            return response
    # Nếu không khớp với bất kỳ quy tắc cụ thể nào, dùng quy tắc ký tự đại diện cuối cùng
    return random.choice(rules[r'.*'])

# Vòng lặp chat chính
if __name__ == '__main__':
    print("Therapist: Hello! How can I help you today?")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["quit", "exit", "bye"]:
            print("Therapist: Goodbye. It was nice talking to you.")
            break
        response = respond(user_input)
        print(f"Therapist: {response}")
        
>>>
Therapist: Hello! How can I help you today?
You: I am feeling sad today.
Therapist: How long have you been feeling sad today?
You: I need some help with my project.
Therapist: Are you sure you need some help with your project?
You: My mother is not happy with my work.
Therapist: Tell me more about your mother.
You: quit
Therapist: Goodbye. It was nice talking to you.
```

Thông qua thực hành lập trình trên, chúng ta có thể trực quan tổng kết những hạn chế căn bản của hệ thống dựa trên quy tắc, những hạn chế này chính là minh chứng trực tiếp cho các thách thức lý thuyết của chủ nghĩa ký hiệu đã nêu ở mục `2.1.4`:

- <strong>Thiếu hiểu biết về ngữ nghĩa</strong>: Hệ thống không hiểu nghĩa của từ. Ví dụ, khi đối mặt với đầu vào "I am <strong>not</strong> happy", nó vẫn sẽ khớp một cách máy móc với quy tắc `I am (.*)` và sinh ra phản hồi không thông về mặt ngữ nghĩa, vì nó không thể hiểu được tác dụng của từ phủ định "not".
- <strong>Không có bộ nhớ ngữ cảnh</strong>: Hệ thống là <strong>không có trạng thái (Stateless)</strong>, mỗi phản hồi chỉ dựa trên câu đầu vào hiện tại, không thể thực hiện hội thoại nhiều lượt một cách mạch lạc.
- <strong>Vấn đề về khả năng mở rộng của quy tắc</strong>: Việc cố gắng thêm nhiều quy tắc hơn sẽ dẫn đến quy mô cơ sở quy tắc tăng trưởng bùng nổ, việc quản lý xung đột và mức độ ưu tiên giữa các quy tắc sẽ trở nên cực kỳ phức tạp, cuối cùng khiến hệ thống khó bảo trì.

Tuy nhiên, dù tồn tại những khiếm khuyết rõ ràng này, ELIZA ở thời điểm đó đã tạo ra hiệu ứng nổi tiếng gọi là "<strong>hiệu ứng ELIZA</strong>" (ELIZA effect), rất nhiều người dùng tin rằng nó có thể hiểu được mình. Ảo giác về trí tuệ này chủ yếu bắt nguồn từ chiến lược đối thoại khéo léo của nó (như đóng vai người đặt câu hỏi thụ động, sử dụng mẫu câu mở) cũng như tâm lý phóng chiếu cảm xúc bẩm sinh của con người.

Thực hành với ELIZA đã hé lộ rõ ràng mâu thuẫn cốt lõi của phương pháp chủ nghĩa ký hiệu: biểu hiện có vẻ thông minh của hệ thống hoàn toàn phụ thuộc vào các quy tắc được người thiết kế mã hóa sẵn. Tuy nhiên, khi đối mặt với tính vô hạn khả năng của ngôn ngữ trong thế giới thực, phương pháp liệt kê (exhaustive) này chắc chắn không thể mở rộng. Hệ thống không có sự hiểu biết thực sự, chỉ đang thực hiện thao tác ký hiệu, đây chính là nguồn gốc của tính dễ vỡ của nó.

## 2.3 "Xã hội của tâm trí" của Marvin Minsky

Những khám phá về chủ nghĩa ký hiệu và thực hành với ELIZA cùng chỉ ra một vấn đề: một động cơ suy luận đơn nhất, tập trung, được xây dựng thông qua các quy tắc định sẵn, dường như khó có thể dẫn đến trí tuệ thực sự. Dù cơ sở quy tắc có đồ sộ đến đâu, hệ thống vẫn luôn tỏ ra cứng nhắc và dễ vỡ khi đối mặt với tính mơ hồ, phức tạp và biến đổi vô hạn của thế giới thực. Bế tắc này đã thúc đẩy một số nhà tư tưởng hàng đầu bắt đầu phản tư về triết lý thiết kế tầng nền tảng nhất của trí tuệ nhân tạo. Trong đó, <strong>Marvin Minsky</strong> không tiếp tục cố gắng thêm nhiều quy tắc hơn cho một lõi suy luận đơn nhất, mà trong cuốn sách <strong>《Xã hội của tâm trí》 (The Society of Mind)</strong><sup>[7]</sup> của mình, ông đã đặt ra một câu hỏi mang tính cách mạng: "What magical trick makes us intelligent? The trick is that there is no trick. The power of intelligence stems from our vast diversity, not from any single, perfect principle."

### 2.3.1 Phản tư về mô hình trí tuệ đơn nhất, toàn thể

Vào những năm 1970-1980, những hạn chế của chủ nghĩa ký hiệu ngày càng trở nên rõ ràng. Hệ chuyên gia tuy đạt được thành công trong các lĩnh vực theo chiều dọc có tính chuyên môn cao, nhưng chúng không thể có được tri thức thông thường như trẻ con; SHRDLU tuy có thể thể hiện xuất sắc trong một thế giới khối gỗ khép kín, nhưng nó không thể hiểu bất cứ điều gì bên ngoài thế giới đó; ELIZA tuy có thể mô phỏng hội thoại, nhưng nó hoàn toàn không biết gì về nội dung của cuộc hội thoại. Những hệ thống này đều tuân theo một tư duy thiết kế từ trên xuống (Top-down): một bộ xử lý trung tâm toàn năng, xử lý thông tin và đưa ra quyết định dựa trên một tập quy tắc logic thống nhất.

Đối mặt với sự thất bại phổ biến này, Minsky bắt đầu đặt ra một loạt câu hỏi mang tính căn bản:

- <strong>"Hiểu" là gì?</strong> Khi chúng ta nói rằng chúng ta hiểu một câu chuyện, đó là một năng lực đơn nhất hay sao? Hay thực ra nó là kết quả của sự phối hợp hoạt động giữa hàng chục quá trình tâm trí khác nhau như năng lực hình dung, năng lực suy luận logic, năng lực đồng cảm, tri thức thông thường về quan hệ xã hội?
- <strong>"Tri thức thông thường" là gì?</strong> Tri thức thông thường có phải là một cơ sở tri thức đồ sộ bao gồm hàng triệu quy tắc logic hay không (như nỗ lực của dự án Cyc)? Hay nó là một mạng lưới phân tán, được đan xen bởi vô số kinh nghiệm cụ thể và các mảnh quy tắc đơn giản?
- <strong>Agent nên được xây dựng như thế nào?</strong> Chúng ta có nên tiếp tục theo đuổi một hệ thống logic hoàn hảo, thống nhất, hay nên thừa nhận rằng bản thân trí tuệ là "không hoàn hảo", là một mớ hỗn tạp được cấu thành từ nhiều bộ phận đơn giản có chức năng khác nhau, thậm chí mâu thuẫn lẫn nhau?

Những câu hỏi này nhắm thẳng vào nhược điểm cốt lõi của mô hình trí tuệ đơn nhất, toàn thể. Loại mô hình này cố gắng dùng một cơ chế biểu diễn và suy luận thống nhất để giải quyết mọi vấn đề, nhưng điều này lại cách rất xa so với cách thức vận hành của trí tuệ tự nhiên (đặc biệt là trí tuệ con người) mà chúng ta quan sát được. Minsky cho rằng, việc ép buộc nhồi nhét các hoạt động tâm trí đa dạng vào một khung logic cứng nhắc, chính là căn nguyên khiến nghiên cứu trí tuệ nhân tạo sơ khai bị đình trệ.

Chính dựa trên sự phản tư này, Minsky đã đề xuất một ý tưởng mang tính lật đổ, ông không còn xem tâm trí là một cấu trúc phân cấp hình kim tự tháp, mà xem nó như một "xã hội" phẳng, đầy ắp tương tác và hợp tác.

### 2.3.2 Trí tuệ như một thể hợp tác

Trong khung lý thuyết của Minsky, định nghĩa về Agent có sự khác biệt so với Agent hiện đại mà chúng ta thảo luận ở Chương 1. Agent ở đây chỉ một quá trình tâm trí cực kỳ đơn giản, chuyên biệt hóa, bản thân nó là "vô tâm" (mindless). Ví dụ, một Agent `LINE-FINDER` phụ trách nhận diện đường nét, hay một Agent `GRASP` phụ trách cầm nắm.

Những Agent đơn giản này được tổ chức lại để tạo thành <strong>cơ quan (Agency)</strong> mạnh mẽ hơn về chức năng. Một cơ quan là một nhóm các Agent phối hợp làm việc, nhằm hoàn thành một nhiệm vụ phức tạp hơn. Ví dụ, một cơ quan `BUILD` phụ trách xếp khối gỗ, có thể được cấu thành từ nhiều Agent hoặc cơ quan tầng dưới như `SEE`, `FIND`, `GET`, `PUT`. Chúng ảnh hưởng lẫn nhau thông qua tín hiệu kích hoạt (activation) và ức chế (inhibition) phi tập trung, hình thành luồng điều khiển động.

<strong>Tính nổi sinh (Emergence)</strong> là chìa khóa để hiểu lý thuyết xã hội của tâm trí. Hành vi thông minh phức tạp, có mục đích, không phải được một Agent cấp cao nào lập kế hoạch trước, mà tự phát sinh ra từ sự tương tác cục bộ giữa vô số Agent tầng dưới đơn giản.

Hãy lấy nhiệm vụ kinh điển "xếp tháp bằng khối gỗ" làm ví dụ để minh họa quá trình này, như Hình 2.6. Khi một mục tiêu cấp cao (như "tôi muốn xếp một cái tháp") xuất hiện, nó sẽ kích hoạt một cơ quan cấp cao có tên `BUILD-TOWER`.

1. Cơ quan `BUILD-TOWER` không biết cách thực hiện hành động vật lý cụ thể, tác dụng duy nhất của nó là kích hoạt cơ quan cấp dưới của nó, ví dụ `BUILDER`.
2. Cơ quan `BUILDER` cũng đơn giản tương tự, nó có thể chỉ chứa một logic vòng lặp: miễn là tháp chưa xếp xong, thì kích hoạt cơ quan `ADD-BLOCK`.
3. Cơ quan `ADD-BLOCK` phụ trách điều phối các nhiệm vụ con cụ thể hơn, nó sẽ lần lượt kích hoạt ba cơ quan con `FIND-BLOCK`, `GET-BLOCK` và `PUT-ON-TOP`.
4. Mỗi cơ quan con lại được cấu thành từ các Agent ở tầng thấp hơn. Ví dụ, cơ quan `GET-BLOCK` sẽ kích hoạt Agent `SEE-SHAPE` trong hệ thống thị giác, cùng các Agent `REACH` và `GRASP` trong hệ thống vận động.

Trong quá trình này, không có bất kỳ Agent hay cơ quan nào sở hữu kế hoạch toàn cục của toàn bộ nhiệm vụ. `GRASP` chỉ phụ trách cầm nắm, nó không biết tháp là gì; `BUILDER` chỉ phụ trách vòng lặp, nó không biết cách điều khiển cánh tay. Tuy nhiên, khi xã hội được cấu thành từ vô số Agent "vô tâm" này tương tác với nhau thông qua các quy tắc kích hoạt và ức chế đơn giản, một hành vi có vẻ cực kỳ thông minh — xếp tháp bằng khối gỗ — đã tự nhiên nổi sinh ra.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-4.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.6 Sơ đồ minh họa cơ chế nổi sinh của hành vi xếp tháp bằng khối gỗ trong "xã hội của tâm trí"</p>
</div>

### 2.3.3 Gợi mở lý thuyết cho hệ thống đa Agent

Ảnh hưởng sâu rộng nhất của lý thuyết xã hội của tâm trí nằm ở chỗ nó đã cung cấp nền tảng khái niệm quan trọng cho <strong>trí tuệ nhân tạo phân tán (Distributed Artificial Intelligence, DAI)</strong> và <strong>hệ thống đa Agent (Multi-Agent System, MAS)</strong> sau này. Nó dẫn dắt các nhà nghiên cứu suy nghĩ:

<strong>Nếu trí tuệ bên trong một tâm trí nổi sinh ra thông qua sự hợp tác của vô số Agent đơn giản, vậy thì, giữa nhiều thực thể tính toán độc lập, tách biệt về mặt vật lý (máy tính, robot), liệu có thể cũng nổi sinh ra một "trí tuệ bầy đàn" mạnh mẽ hơn thông qua hợp tác hay không?</strong>

Việc đặt ra câu hỏi này đã trực tiếp chuyển trọng tâm nghiên cứu từ "làm thế nào để xây dựng một Agent đơn nhất toàn năng" sang "làm thế nào để thiết kế một bầy Agent hợp tác hiệu quả". Cụ thể, xã hội của tâm trí đã trực tiếp gợi mở cho nghiên cứu hệ thống đa Agent ở một số khía cạnh sau:

- <strong>Điều khiển phi tập trung (Decentralized Control)</strong>: Cốt lõi của lý thuyết nằm ở việc không tồn tại bộ điều khiển trung tâm. Tư tưởng này được lĩnh vực MAS kế thừa hoàn toàn, việc làm thế nào để thiết kế cơ chế điều phối và chiến lược phân công nhiệm vụ không có nút trung tâm đã trở thành một trong những chủ đề nghiên cứu cốt lõi của MAS.
- <strong>Tính toán nổi sinh (Emergent Computation)</strong>: Lời giải cho vấn đề phức tạp có thể tự phát sinh ra từ các quy tắc tương tác cục bộ đơn giản. Điều này đã gợi mở nhiều thuật toán dựa trên tư tưởng nổi sinh trong MAS, như thuật toán đàn kiến (ant colony algorithm), tối ưu hóa bầy đàn (particle swarm optimization), dùng để giải quyết các bài toán tối ưu hóa và tìm kiếm phức tạp.
- <strong>Tính xã hội của Agent (Agent Sociality)</strong>: Lý thuyết của Minsky nhấn mạnh sự tương tác (kích hoạt, ức chế) giữa các Agent. Lĩnh vực MAS đã mở rộng hơn nữa điều này, nghiên cứu một cách có hệ thống ngôn ngữ giao tiếp giữa các Agent (như ACL), giao thức tương tác (như mạng hợp đồng - contract net), chiến lược đàm phán, mô hình tin cậy cho đến cấu trúc tổ chức, từ đó xây dựng nên một xã hội tính toán thực sự.

Có thể nói, lý thuyết "xã hội của tâm trí" của Minsky đã cung cấp một khung phân tích quan trọng để các nhà nghiên cứu AI hiểu được cấu trúc nội tại của "trí tuệ bầy đàn". Nó đã cung cấp cho các nhà nghiên cứu sau này một góc nhìn hoàn toàn mới để khám phá các hệ thống phức tạp được cấu thành từ các Agent tính toán độc lập, tự chủ, có năng lực xã hội, từ đó chính thức mở màn cho nghiên cứu hệ thống đa Agent.

## 2.4 Sự tiến hóa của paradigm học tập và Agent hiện đại

Lý thuyết "xã hội của tâm trí" được thảo luận ở phần trên, về mặt triết học đã chỉ ra phương hướng cho trí tuệ bầy đàn và hợp tác phi tập trung, nhưng con đường hiện thực hóa vẫn chưa rõ ràng. Đồng thời, những thách thức căn bản mà chủ nghĩa ký hiệu bộc lộ khi đối mặt với tính phức tạp của thế giới thực cũng cho thấy chỉ dựa vào các quy tắc được mã hóa sẵn thì không thể xây dựng được trí tuệ thực sự vững chắc (robust).

Hai manh mối này cùng chỉ ra một vấn đề: nếu trí tuệ không thể được thiết kế hoàn toàn, vậy liệu nó có thể được học ra hay không?

Câu hỏi này đã mở ra thời đại "học tập" (learning) của trí tuệ nhân tạo. Mục tiêu cốt lõi của nó không còn là mã hóa tri thức thủ công, mà là xây dựng các hệ thống có thể tự động thu nạp tri thức và năng lực từ kinh nghiệm và dữ liệu. Phần này sẽ hồi cố quá trình tiến hóa của paradigm này: từ nền tảng học tập do chủ nghĩa liên kết đặt ra, đến học tập tương tác được hiện thực hóa bởi học tăng cường, cho đến các Agent hiện đại ngày nay được thúc đẩy bởi mô hình ngôn ngữ lớn.

### 2.4.1 Từ ký hiệu đến liên kết

Như một phản hồi trực tiếp đối với hạn chế của chủ nghĩa ký hiệu, <strong>chủ nghĩa liên kết (Connectionism)</strong> đã nổi lên trở lại vào những năm 1980. Khác với triết lý thiết kế từ trên xuống, phụ thuộc vào quy tắc logic rõ ràng của chủ nghĩa ký hiệu, chủ nghĩa liên kết là một phương pháp từ dưới lên (bottom-up), lấy cảm hứng từ việc mô phỏng cấu trúc mạng lưới thần kinh của bộ não sinh học<sup>[8]</sup>. Tư tưởng cốt lõi của nó có thể được khái quát thành các điểm sau:

1. <strong>Biểu diễn phân tán của tri thức</strong>: Tri thức không được lưu trữ dưới dạng ký hiệu hay quy tắc rõ ràng trong một cơ sở tri thức nào đó, mà được lưu trữ một cách phân tán dưới dạng trọng số liên kết (connection weight), giữa các kết nối của một số lượng lớn đơn vị xử lý đơn giản (tức nơ-ron nhân tạo). Bản thân mô hình kết nối của toàn bộ mạng lưới cấu thành nên tri thức.
2. <strong>Đơn vị xử lý đơn giản</strong>: Mỗi nơ-ron chỉ thực hiện phép tính cực kỳ đơn giản, như nhận đầu vào có trọng số từ các nơ-ron khác, xử lý thông qua một hàm kích hoạt (activation function), sau đó xuất kết quả cho nơ-ron tiếp theo.
3. <strong>Điều chỉnh trọng số thông qua học tập</strong>: Trí tuệ của hệ thống không đến từ chương trình phức tạp được người thiết kế viết sẵn, mà đến từ quá trình "học tập". Hệ thống tiếp xúc với một lượng lớn mẫu dữ liệu, dựa trên một thuật toán học tập nào đó (như thuật toán lan truyền ngược - backpropagation) để tự động, lặp đi lặp lại điều chỉnh trọng số kết nối giữa các nơ-ron, từ đó khiến đầu ra của toàn bộ mạng lưới dần tiệm cận với mục tiêu mong muốn.

Dưới paradigm này, Agent không còn là một máy suy luận logic thụ động thực thi quy tắc, mà là một hệ thống thích nghi (adaptive) có thể tự tối ưu hóa thông qua kinh nghiệm. Như Hình 2.7 minh họa, đây đại diện cho một sự chuyển biến căn bản trong tư tưởng cốt lõi của việc xây dựng Agent. Chủ nghĩa ký hiệu cố gắng mã hóa tường minh tri thức của con người cho máy móc, còn chủ nghĩa liên kết lại cố gắng tạo ra một cỗ máy có thể học tri thức giống như con người.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-5.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.7 So sánh paradigm chủ nghĩa ký hiệu và chủ nghĩa liên kết</p>
</div>

Sự trỗi dậy của chủ nghĩa liên kết, đặc biệt là thành công của học sâu (deep learning) trong thế kỷ 21, đã trao cho Agent năng lực nhận thức (perception) và nhận diện mẫu (pattern recognition) mạnh mẽ, giúp nó có thể hiểu thế giới trực tiếp từ dữ liệu thô (như hình ảnh, âm thanh, văn bản), điều mà thời kỳ chủ nghĩa ký hiệu khó có thể tưởng tượng được. Tuy nhiên, việc làm thế nào để Agent học được cách đưa ra quyết định tuần tự (sequential decision) tối ưu trong tương tác động với môi trường, thì cần sự bổ sung của một paradigm học tập khác.

### 2.4.2 Agent dựa trên học tăng cường

Chủ nghĩa liên kết chủ yếu giải quyết vấn đề nhận thức (ví dụ, "bức ảnh này có gì?"), nhưng nhiệm vụ cốt lõi hơn của Agent là ra quyết định (ví dụ, "trong tình huống này, tôi nên làm gì?"). <strong>Học tăng cường (Reinforcement Learning, RL)</strong> chính là paradigm học tập tập trung giải quyết vấn đề ra quyết định tuần tự. Nó không học trực tiếp từ một tập dữ liệu tĩnh đã được gán nhãn, mà thông qua tương tác trực tiếp giữa Agent và môi trường, học cách tối đa hóa lợi ích dài hạn của mình trong quá trình "thử và sai" (trial and error).

Lấy AlphaGo làm ví dụ, quá trình học tập tự đối đầu (self-play) cốt lõi của nó chính là biểu hiện kinh điển của học tăng cường<sup>[9]</sup>. Trong quá trình này, AlphaGo (Agent) thông qua việc quan sát cách bố trí hiện tại của bàn cờ (trạng thái môi trường), quyết định vị trí đặt quân cờ ở bước tiếp theo (hành động). Sau khi một ván cờ kết thúc, dựa trên kết quả thắng thua, nó sẽ nhận được một tín hiệu rõ ràng: thắng thì được thưởng dương (positive reward), thua thì bị phạt âm (negative reward). Thông qua hàng triệu lần tự đối đầu như vậy, AlphaGo không ngừng điều chỉnh chiến lược nội tại của mình, dần dần học được nên chọn hành động nào trong tình huống cờ nào thì có khả năng dẫn đến chiến thắng cuối cùng cao nhất. Quá trình này hoàn toàn tự chủ, không phụ thuộc vào sự hướng dẫn trực tiếp từ các ván cờ của con người.

Cơ chế học tập thông qua tương tác với môi trường, tối ưu hóa hành vi của bản thân dựa trên tín hiệu phản hồi này, chính là khung cốt lõi của học tăng cường. Dưới đây chúng ta sẽ phân tích chi tiết các yếu tố cấu thành cơ bản và phương thức hoạt động của nó.

Khung của học tăng cường có thể được mô tả bằng một vài yếu tố cốt lõi sau:

- <strong>Agent</strong>: Người học và người ra quyết định. Trong ví dụ AlphaGo, đó chính là chương trình ra quyết định của nó.
- <strong>Môi trường (Environment)</strong>: Toàn bộ những gì bên ngoài Agent, là đối tượng mà Agent tương tác cùng. Đối với AlphaGo, đó chính là luật chơi cờ vây và đối thủ.
- <strong>Trạng thái (State, S)</strong>: Mô tả cụ thể về môi trường tại một thời điểm nào đó, là căn cứ để Agent đưa ra quyết định. Ví dụ, vị trí hiện tại của tất cả các quân cờ trên bàn cờ.
- <strong>Hành động (Action, A)</strong>: Thao tác mà Agent có thể thực hiện dựa trên trạng thái hiện tại. Ví dụ, đặt một quân cờ vào một vị trí hợp lệ nào đó trên bàn cờ.
- <strong>Phần thưởng (Reward, R)</strong>: Một tín hiệu vô hướng (scalar) mà môi trường phản hồi cho Agent sau khi Agent thực hiện một hành động, dùng để đánh giá mức độ tốt xấu của hành động đó trong trạng thái cụ thể. Ví dụ, sau khi một ván cờ kết thúc, thắng thì được +1 phần thưởng, thua thì được -1 phần thưởng.

Dựa trên các yếu tố cốt lõi nêu trên, Agent học tăng cường liên tục lặp trong một vòng lặp khép kín "nhận thức - hành động - học tập", phương thức hoạt động của nó được minh họa trong Hình 2.8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-6.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.8 Vòng lặp tương tác cốt lõi của học tăng cường</p>
</div>

Các bước cụ thể của vòng lặp này như sau:

1. Tại bước thời gian t, Agent quan sát được trạng thái hiện tại $S_{t}$ của môi trường.
2. Dựa trên trạng thái $S_{t}$, Agent chọn một hành động $A_{t}$ dựa trên <strong>chiến lược (Policy, π)</strong> nội tại của nó và thực hiện hành động đó. Về bản chất, chiến lược là một ánh xạ từ trạng thái đến hành động, định nghĩa phương thức hành vi của Agent.
3. Sau khi môi trường nhận được hành động $A_{t}$, nó sẽ chuyển sang một trạng thái mới $S_{t+1}$.
4. Đồng thời, môi trường sẽ phản hồi cho Agent một phần thưởng tức thời $R_{t+1}$.
5. Agent sử dụng phản hồi này (trạng thái mới $S_{t+1}$ và phần thưởng $R_{t+1}$) để cập nhật và tối ưu hóa chiến lược nội tại của mình, nhằm đưa ra quyết định tốt hơn trong tương lai. Quá trình cập nhật này chính là học tập.

Mục tiêu học tập của Agent không phải là tối đa hóa phần thưởng tức thời tại một bước thời gian nào đó, mà là tối đa hóa <strong>phần thưởng tích lũy (Cumulative Reward)</strong> từ thời điểm hiện tại đến tương lai, còn được gọi là <strong>lợi nhuận (Return)</strong>. Điều này có nghĩa là Agent cần có "tầm nhìn xa", đôi khi để đạt được phần thưởng lớn hơn trong tương lai, cần hy sinh phần thưởng tức thời hiện tại (ví dụ, chiến lược "thí quân" trong cờ vây). Thông qua việc không ngừng khám phá, thu thập phản hồi và tối ưu hóa chiến lược trong vòng lặp nêu trên, Agent cuối cùng có thể học được cách ra quyết định tự chủ và lập kế hoạch dài hạn trong môi trường động phức tạp.

### 2.4.3 Tiền huấn luyện dựa trên dữ liệu quy mô lớn

Học tăng cường đã trao cho Agent năng lực học chiến lược ra quyết định từ tương tác, nhưng điều này thường đòi hỏi một lượng lớn dữ liệu tương tác dành riêng cho một nhiệm vụ cụ thể, dẫn đến việc Agent thiếu tri thức tiên nghiệm (prior knowledge) khi bắt đầu học, cần xây dựng sự hiểu biết về nhiệm vụ từ con số không. Dù là tri thức thông thường mà chủ nghĩa ký hiệu cố gắng mã hóa thủ công, hay tri thức nền tảng mà con người dựa vào khi ra quyết định, đều bị thiếu hụt trong Agent RL. Làm thế nào để Agent có được sự hiểu biết rộng rãi về thế giới trước khi bắt đầu học một nhiệm vụ cụ thể? Lời giải cho vấn đề này cuối cùng xuất hiện trong lĩnh vực <strong>xử lý ngôn ngữ tự nhiên (Natural Language Processing, NLP)</strong>, cốt lõi của nó chính là <strong>tiền huấn luyện (Pre-training)</strong> dựa trên dữ liệu quy mô lớn.

<strong>Từ mô hình cho nhiệm vụ cụ thể đến mô hình tổng quát</strong>

Trước khi paradigm tiền huấn luyện xuất hiện, các mô hình xử lý ngôn ngữ tự nhiên truyền thống thường được huấn luyện độc lập từ con số không cho một nhiệm vụ cụ thể, đơn lẻ (như phân tích cảm xúc, dịch máy) trên các tập dữ liệu quy mô vừa và nhỏ được gán nhãn chuyên biệt. Mô hình này dẫn đến một số vấn đề: phạm vi tri thức của mô hình hẹp, khó tổng quát hóa tri thức học được từ một nhiệm vụ sang nhiệm vụ khác, và mỗi nhiệm vụ mới đều cần tiêu tốn một lượng lớn nhân lực để gán nhãn dữ liệu. Việc đề xuất paradigm tiền huấn luyện và tinh chỉnh (Pre-training, Fine-tuning) đã thay đổi hoàn toàn hiện trạng này. Tư tưởng cốt lõi của nó chia thành hai bước:

1. <strong>Giai đoạn tiền huấn luyện</strong>: Đầu tiên, huấn luyện một mô hình mạng nơ-ron cực kỳ lớn trên một kho ngữ liệu (corpus) tổng quát chứa lượng dữ liệu văn bản khổng lồ ở quy mô Internet, thông qua phương thức <strong>học tự giám sát (Self-supervised Learning)</strong>. Mục tiêu của giai đoạn này không phải là hoàn thành bất kỳ nhiệm vụ cụ thể nào, mà là học các quy luật nội tại của bản thân ngôn ngữ, cấu trúc ngữ pháp, tri thức thực tế cũng như logic ngữ cảnh. Mục tiêu phổ biến nhất là "dự đoán từ tiếp theo".
2. <strong>Giai đoạn tinh chỉnh</strong>: Sau khi hoàn thành tiền huấn luyện, mô hình này đã học được một lượng tri thức phong phú liên quan đến tập dữ liệu. Sau đó, đối với nhiệm vụ downstream cụ thể, chỉ cần sử dụng một lượng nhỏ dữ liệu gán nhãn của nhiệm vụ đó để tinh chỉnh mô hình, là có thể giúp mô hình thích ứng với nhiệm vụ tương ứng.

Như Hình 2.9 minh họa, thể hiện trực quan toàn bộ quy trình tiền huấn luyện và tinh chỉnh này: dữ liệu văn bản tổng quát trải qua học tự giám sát để hình thành mô hình nền tảng (foundation model), sau đó được tinh chỉnh thông qua dữ liệu nhiệm vụ cụ thể, cuối cùng thích ứng với các nhiệm vụ downstream khác nhau.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-7.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.9 Sơ đồ minh họa paradigm "tiền huấn luyện - tinh chỉnh"</p>
</div>

<strong>Sự ra đời của mô hình ngôn ngữ lớn và năng lực nổi sinh</strong>

Thông qua việc tiền huấn luyện trên lượng văn bản ở quy mô hàng nghìn tỷ, trọng số mạng nơ-ron của mô hình ngôn ngữ lớn trên thực tế đã xây dựng nên một mô hình ẩn (implicit model), được nén cao độ, về tri thức thế giới. Nó đã giải quyết theo một phương thức hoàn toàn mới vấn đề "nút thắt cổ chai trong thu thập tri thức" nan giải nhất của thời kỳ chủ nghĩa ký hiệu. Điều đáng ngạc nhiên hơn là, khi quy mô của mô hình (số lượng tham số, lượng dữ liệu, lượng tính toán) vượt qua một ngưỡng nào đó, chúng bắt đầu thể hiện <strong>năng lực nổi sinh (Emergent Abilities)</strong> chưa được huấn luyện trực tiếp, nằm ngoài dự kiến, ví dụ:

- <strong>Học trong ngữ cảnh (In-context Learning)</strong>: Không cần điều chỉnh trọng số mô hình, chỉ cần cung cấp <strong>một vài ví dụ (Few-shot)</strong> thậm chí <strong>không có ví dụ nào (Zero-shot)</strong> trong đầu vào, mô hình đã có thể hiểu và hoàn thành nhiệm vụ mới.
- <strong>Suy luận chuỗi tư duy (Chain-of-Thought)</strong>: Thông qua việc hướng dẫn mô hình xuất ra quá trình suy luận từng bước trước khi trả lời câu hỏi phức tạp, có thể nâng cao đáng kể độ chính xác của nó trong các nhiệm vụ logic, số học và suy luận tri thức thông thường.

Sự xuất hiện của những năng lực này đánh dấu việc LLM không còn chỉ là một mô hình ngôn ngữ nữa, nó đã tiến hóa thành một thành phần mang kép vai trò vừa là cơ sở tri thức khổng lồ vừa là động cơ suy luận tổng quát.

Đến đây, trong dòng chảy lịch sử phát triển của Agent, một vài mảnh ghép công nghệ then chốt đã lần lượt xuất hiện đầy đủ: chủ nghĩa ký hiệu cung cấp khung suy luận logic, chủ nghĩa liên kết và học tăng cường cung cấp năng lực học tập và ra quyết định, còn mô hình ngôn ngữ lớn cung cấp tri thức thế giới và năng lực suy luận tổng quát chưa từng có, có được thông qua tiền huấn luyện. Phần tiếp theo, chúng ta sẽ thấy các công nghệ này đã được hòa quyện với nhau như thế nào trong thiết kế của Agent hiện đại.

### 2.4.4 Agent dựa trên mô hình ngôn ngữ lớn

Cùng với sự phát triển vũ bão của công nghệ mô hình ngôn ngữ lớn, Agent lấy LLM làm cốt lõi đã trở thành paradigm mới của lĩnh vực trí tuệ nhân tạo. Nó không chỉ có thể hiểu và sinh ra ngôn ngữ con người, mà quan trọng hơn, có thể tự chủ nhận thức, lập kế hoạch, ra quyết định và thực thi nhiệm vụ thông qua tương tác với môi trường.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-8.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.10 Kiến trúc các thành phần cốt lõi của Agent được thúc đẩy bởi LLM</p>
</div>

Như đã trình bày ở Chương 1, tương tác giữa Agent và môi trường có thể được trừu tượng hóa thành một vòng lặp cốt lõi. Agent được thúc đẩy bởi LLM hoàn thành nhiệm vụ thông qua một quy trình khép kín, liên tục lặp lại, được phối hợp bởi nhiều module. Quy trình này tuân theo kiến trúc minh họa trong Hình 2.10, các bước cụ thể như sau:

1. <strong>Nhận thức (Perception)</strong>: Quy trình bắt đầu từ <strong>module nhận thức (Perception Module)</strong>. Nó nhận đầu vào thô từ <strong>môi trường bên ngoài (Environment)</strong> thông qua cảm biến (sensor), hình thành <strong>quan sát (Observation)</strong>. Những thông tin quan sát này (như mệnh lệnh người dùng, dữ liệu trả về từ API hoặc sự thay đổi trạng thái môi trường) là điểm khởi đầu cho việc ra quyết định của Agent, sau khi xử lý sẽ được chuyển đến giai đoạn tư duy.
2. <strong>Tư duy (Thought)</strong>: Đây là cốt lõi nhận thức của Agent, tương ứng với sự phối hợp làm việc giữa <strong>module lập kế hoạch (Planning Module)</strong> và <strong>mô hình ngôn ngữ lớn (LLM)</strong> trong hình.
   - <strong>Lập kế hoạch và phân rã</strong>: Trước tiên, module lập kế hoạch nhận thông tin quan sát, tiến hành hoạch định chiến lược cấp cao. Nó thông qua các cơ chế <strong>phản tư (Reflection)</strong> và <strong>tự phê bình (Self-criticism)</strong> để phân rã mục tiêu vĩ mô thành các bước cụ thể hơn, có thể thực thi được.
   - <strong>Suy luận và ra quyết định</strong>: Tiếp theo, <strong>LLM</strong> đóng vai trò trung tâm nhận chỉ thị từ module lập kế hoạch, và tương tác với <strong>module bộ nhớ (Memory)</strong> để tích hợp thông tin lịch sử. LLM tiến hành suy luận sâu, cuối cùng quyết định thao tác cụ thể cần thực hiện ở bước tiếp theo, thường thể hiện dưới dạng một <strong>lệnh gọi công cụ (Tool Call)</strong>.
3. <strong>Hành động (Action)</strong>: Sau khi ra quyết định xong, sẽ bước vào giai đoạn hành động, do <strong>module thực thi (Execution Module)</strong> đảm nhận. Lệnh gọi công cụ do LLM sinh ra được gửi đến module thực thi. Module này phân tích lệnh, chọn và gọi công cụ phù hợp (như trình thực thi code, công cụ tìm kiếm, API, v.v.) từ <strong>bộ công cụ (Tool Use)</strong> để tương tác với môi trường hoặc thực thi nhiệm vụ. Tương tác thực tế với môi trường này chính là <strong>hành động (Action)</strong> của Agent.
4. <strong>Quan sát (Observation)</strong> và vòng lặp: Hành động sẽ thay đổi trạng thái của môi trường, và tạo ra kết quả.
   - Sau khi công cụ được thực thi sẽ trả về một <strong>kết quả công cụ (Tool Result)</strong> cho LLM, đây là phản hồi trực tiếp về hiệu quả của hành động. Đồng thời, hành động của Agent đã làm thay đổi môi trường, từ đó tạo ra một <strong>trạng thái môi trường</strong> hoàn toàn mới.
   - "Kết quả công cụ" và "trạng thái môi trường mới" này cùng nhau cấu thành một vòng <strong>quan sát (Observation)</strong> hoàn toàn mới. Quan sát mới này sẽ lại được module nhận thức thu nhận, đồng thời LLM sẽ <strong>cập nhật bộ nhớ (Memory Update)</strong> dựa trên kết quả hành động, từ đó khởi động vòng lặp "nhận thức - tư duy - hành động" tiếp theo.

Cơ chế phối hợp theo module và vòng lặp lặp đi lặp lại liên tục này cấu thành nên quy trình làm việc (workflow) cốt lõi của Agent được thúc đẩy bởi LLM trong việc giải quyết các vấn đề phức tạp.



### 2.4.5 Tổng quan các mốc then chốt trong sự phát triển của Agent

Lịch sử phát triển của Agent trí tuệ nhân tạo không phải một con đường một chiều thẳng tắp, mà là quá trình đan xen, cạnh tranh và hòa quyện kéo dài hơn nửa thế kỷ của nhiều trường phái tư tưởng cốt lõi. Hiểu rõ quá trình này sẽ giúp chúng ta nhìn thấu căn nguyên sâu xa của việc hình thành paradigm kiến trúc Agent hiện nay.

Trong đó, chủ yếu có ba trào lưu tư tưởng lớn chi phối paradigm nghiên cứu ở các thời kỳ khác nhau:

1. <strong>Chủ nghĩa ký hiệu (Symbolism)</strong>: Đại diện bởi các bậc tiên phong như <strong>Herbert A. Simon</strong>, <strong>Marvin Minsky</strong>, cho rằng cốt lõi của trí tuệ nằm ở việc thao tác ký hiệu và suy luận logic. Tư tưởng này đã khai sinh ra SHRDLU có thể hiểu được mệnh lệnh ngôn ngữ tự nhiên, hệ chuyên gia lấy tri thức làm động lực, và máy tính "Deep Blue" đã giành được thành công vang dội trong lĩnh vực cờ vua.
2. <strong>Chủ nghĩa liên kết (Connectionism)</strong>: Lấy cảm hứng từ việc mô phỏng mạng lưới thần kinh của bộ não. Mặc dù sự phát triển sơ khai bị hạn chế, nhưng dưới sự thúc đẩy của các nhà nghiên cứu như <strong>Geoffrey Hinton</strong>, thuật toán lan truyền ngược (backpropagation) đã đặt nền móng cho sự phục hưng của mạng nơ-ron. Cuối cùng, cùng với sự đến của thời đại học sâu, tư tưởng này đã trở thành dòng chủ lưu hiện nay thông qua các mô hình như mạng nơ-ron tích chập (CNN), Transformer.
3. <strong>Chủ nghĩa hành vi (Behaviorism)</strong>: Nhấn mạnh việc Agent học chiến lược tối ưu thông qua tương tác và thử-sai với môi trường, hóa thân hiện đại của nó là học tăng cường. Từ TD-Gammon sơ khai cho đến AlphaGo kết hợp với học sâu và đánh bại các kỳ thủ cờ vây hàng đầu thế giới, trường phái này đã trao cho Agent năng lực học được hành vi ra quyết định phức tạp từ kinh nghiệm.

Bước vào thập niên 2020 của thế kỷ 21, các trường phái tư tưởng này đã hòa quyện sâu sắc theo một phương thức chưa từng có. Mô hình ngôn ngữ lớn đại diện bởi dòng GPT, bản thân nó là sản phẩm của chủ nghĩa liên kết, nhưng lại trở thành "bộ não" cốt lõi thực hiện suy luận ký hiệu, gọi công cụ và lập kế hoạch ra quyết định, hình thành nên kiến trúc Agent hiện đại kết hợp thần kinh-ký hiệu (neuro-symbolic). Để tổng kết có hệ thống mạch phát triển này, Hình 2.11 dưới đây đã hệ thống hóa các lý thuyết, dự án và sự kiện then chốt trong lịch sử phát triển của Agent trí tuệ nhân tạo từ những năm 1950 đến nay, cung cấp cho độc giả một cái nhìn tổng quan rõ ràng, như một sự đúc kết tri thức của chương này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-9.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.11 Dòng thời gian tiến hóa phát triển của Agent (phiên bản chưa đầy đủ)</p>
</div>

Nhờ bước đột phá của mô hình ngôn ngữ lớn, technology stack của Agent thể hiện mức độ sôi động và đa dạng chưa từng có. Hình 2.12 thể hiện toàn cảnh một technology stack điển hình của lĩnh vực AI Agent hiện nay, bao trùm từ mô hình tầng nền tảng cho đến các ứng dụng tầng trên.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-10.png" alt="Mô tả hình ảnh" width="90%"/>
  <p>Hình 2.12 Tổng quan technology stack của AI Agent</p>
</div>

Sơ đồ technology stack này do công ty Letta công bố vào tháng 11 năm 2024<sup>[10]</sup>, nó đã phân tầng và phân loại các công cụ, nền tảng và dịch vụ liên quan đến AI Agent, cung cấp cho chúng ta tài liệu tham khảo quý giá để hiểu cục diện thị trường và lựa chọn công nghệ hiện nay.

## 2.5 Tổng kết chương

Chương này đã hồi cố mạch lịch sử phát triển của Agent, khám phá quá trình tư tưởng cốt lõi của nó từ khi ra đời đến khi tiến hóa, nội dung bao trùm một số cuộc cách mạng paradigm then chốt của lĩnh vực trí tuệ nhân tạo:

- <strong>Khám phá và hạn chế của chủ nghĩa ký hiệu</strong>: Xuất phát từ thời kỳ cổ điển của trí tuệ nhân tạo, chương này đã trình bày cách mà các Agent sơ khai, đại diện bởi hệ chuyên gia, đã thử mô phỏng trí tuệ thông qua "tri thức + suy luận". Thông qua việc tự tay xây dựng một chatbot dựa trên quy tắc, chúng ta đã cảm nhận sâu sắc ranh giới năng lực của paradigm này cũng như những thách thức căn bản mà nó phải đối mặt.
- <strong>Mầm mống của tư tưởng trí tuệ phân tán</strong>: Thảo luận về lý thuyết "xã hội của tâm trí" của Marvin Minsky. Tư tưởng mang tính cách mạng này đã hé lộ rằng trí tuệ tổng thể phức tạp có thể nổi sinh từ sự tương tác giữa các đơn vị cục bộ đơn giản, cung cấp sự gợi mở triết học quan trọng cho nghiên cứu hệ thống đa Agent sau này.
- <strong>Sự tiến hóa của paradigm học tập</strong>: Chứng kiến sự biến đổi căn bản trong phương thức Agent thu nạp năng lực. Từ việc chủ nghĩa liên kết trao cho Agent năng lực nhận thức thế giới, đến học tăng cường giúp nó học cách ra quyết định tối ưu trong tương tác với môi trường, rồi đến mô hình ngôn ngữ lớn (LLM) dựa trên tiền huấn luyện với dữ liệu quy mô lớn cung cấp cho nó tri thức thế giới và năng lực suy luận tổng quát chưa từng có.
- <strong>Sự ra đời của Agent hiện đại</strong>: Cuối cùng, chúng ta đã phân tích Agent được thúc đẩy bởi LLM. Thông qua việc phân tích các thành phần cốt lõi (mô hình, bộ nhớ, lập kế hoạch, công cụ, v.v.) và nguyên lý hoạt động của nó, chúng ta đã hiểu được các tư tưởng công nghệ khác nhau trong lịch sử đã được hòa quyện về mặt kỹ thuật như thế nào trong kiến trúc của Agent hiện đại.

Thông qua việc học chương này, chúng ta không chỉ hiểu được Agent hiện đại được giới thiệu ở Chương 1 đến từ đâu, mà còn có thể xây dựng được một khung nhận thức vĩ mô về sự tiến hóa công nghệ của Agent. Có thể thấy, sự phát triển của Agent không phải là sự lặp lại công nghệ đơn giản, mà là một cuộc biến cách tư tưởng về cách định nghĩa "trí tuệ", thu nạp "tri thức", tiến hành "ra quyết định".

Vì cốt lõi của Agent hiện đại là mô hình ngôn ngữ lớn, nên việc hiểu sâu nguyên lý tầng nền tảng của nó là vô cùng quan trọng. Chương tiếp theo sẽ tập trung vào bản thân mô hình ngôn ngữ lớn, thảo luận về các khái niệm cơ bản của nó, đặt nền móng vững chắc cho các ứng dụng nâng cao trong hệ thống đa Agent ở các chương sau.



## Bài tập

> <strong>Gợi ý</strong>: Một số bài tập dưới đây không có đáp án chuẩn, nhằm giúp người học xây dựng sự hiểu biết có hệ thống về lịch sử phát triển của Agent, và rèn luyện khả năng nhìn nhận công nghệ "lấy lịch sử làm gương".

1. Giả thuyết hệ thống ký hiệu vật lý<sup>[1]</sup> là nền tảng lý thuyết của thời kỳ chủ nghĩa ký hiệu. Hãy phân tích:
   
   - "Luận điểm về tính đầy đủ" và "luận điểm về tính tất yếu" của giả thuyết này lần lượt mang ý nghĩa gì?
   - Kết hợp với nội dung chương này, hãy trình bày những vấn đề mà Agent chủ nghĩa ký hiệu gặp phải trong thực tiễn đã đặt ra thách thức gì đối với "tính đầy đủ" của giả thuyết này?
   - Agent được thúc đẩy bởi mô hình ngôn ngữ lớn có phù hợp với giả thuyết hệ thống ký hiệu vật lý hay không?

2. Hệ chuyên gia MYCIN<sup>[2]</sup> đã đạt được thành công đáng kể trong lĩnh vực chẩn đoán y khoa, nhưng cuối cùng lại không được ứng dụng rộng rãi trong thực hành lâm sàng. Hãy suy nghĩ:
   
   > <strong>Gợi ý</strong>: Có thể phân tích từ nhiều góc độ như công nghệ, đạo đức, pháp lý, mức độ chấp nhận của người dùng
   
   - Ngoài "nút thắt cổ chai trong thu thập tri thức" và "tính dễ vỡ" đã được đề cập trong chương, còn những nhân tố nào khác có thể đã cản trở việc ứng dụng hệ chuyên gia trong các lĩnh vực rủi ro cao như y tế?
   - Nếu để bạn ở hiện tại thiết kế một Agent chẩn đoán y khoa, bạn sẽ thiết kế hệ thống như thế nào để khắc phục hạn chế của MYCIN?
   - Trong những lĩnh vực dọc nào, hệ chuyên gia dựa trên quy tắc cho đến nay vẫn là lựa chọn tốt hơn so với học sâu? Hãy nêu ví dụ minh họa.

3. Trong mục 2.2, chúng ta đã hiện thực hóa một phiên bản đơn giản hóa của chatbot ELIZA. Hãy thực hành mở rộng dựa trên nền tảng đó:
   
   > <strong>Gợi ý</strong>: Đây là một bài tập thực hành thao tác, khuyến khích viết code thực tế
   
   - Thêm 3-5 quy tắc mới cho ELIZA, giúp nó có thể xử lý các tình huống hội thoại đa dạng hơn (như nói về công việc, học tập, sở thích, v.v.)
   - Hiện thực hóa một chức năng "bộ nhớ ngữ cảnh" đơn giản: giúp ELIZA có thể ghi nhớ các thông tin then chốt mà người dùng đề cập trong hội thoại (như tên, tuổi, nghề nghiệp), và tham chiếu đến chúng trong các lượt hội thoại sau
   - So sánh ELIZA đã được mở rộng của bạn với [ChatGPT](https://chatgpt.com/), liệt kê ít nhất 3 khía cạnh khác biệt bản chất tồn tại giữa chúng
   - Tại sao phương pháp dựa trên quy tắc lại gặp phải vấn đề "bùng nổ tổ hợp" (combinatorial explosion) và khó mở rộng bảo trì khi xử lý hội thoại miền mở (open-domain)? Có thể dùng phương pháp toán học để giải thích điều này không?

4. Marvin Minsky trong lý thuyết "xã hội của tâm trí"<sup>[7]</sup> đã đề xuất một quan điểm mang tính cách mạng: trí tuệ bắt nguồn từ sự hợp tác của vô số Agent đơn giản, chứ không phải từ một hệ thống hoàn hảo đơn nhất.
   
   - Trong ví dụ "xếp tháp bằng khối gỗ" ở Hình 2.6, nếu Agent `GRASP` đột nhiên bị lỗi, toàn bộ hệ thống sẽ xảy ra chuyện gì? Ưu điểm và nhược điểm của kiến trúc phi tập trung này là gì?
   - Hãy so sánh lý thuyết "xã hội của tâm trí" với một số hệ thống đa Agent hiện nay (như [CAMEL-Workforce](https://docs.camel-ai.org/key_modules/workforce), [MetaGPT](https://github.com/FoundationAgents/MetaGPT), [CrewAI](https://github.com/crewAIInc/crewAI)), chúng có những liên hệ và khác biệt gì?
   - Marvin Minsky cho rằng Agent có thể là một quá trình đơn giản "vô tâm", tuy nhiên các mô hình ngôn ngữ lớn và Agent hiện nay thường sở hữu năng lực suy luận mạnh mẽ. Điều này có đồng nghĩa với việc lý thuyết "xã hội của tâm trí" không còn phù hợp trong thời đại mô hình ngôn ngữ lớn nữa hay không?
   
5. Học tăng cường và học có giám sát (supervised learning) là hai paradigm học tập khác nhau. Hãy phân tích:
   
   - Dùng ví dụ AlphaGo để trình bày cơ chế "học tập thử-sai" của học tăng cường hoạt động như thế nào
   - Tại sao học tăng cường đặc biệt phù hợp với bài toán ra quyết định tuần tự? Nó khác biệt bản chất gì với học có giám sát về nhu cầu dữ liệu?
   - Giả sử bây giờ chúng ta cần huấn luyện một Agent biết chơi game Super Mario. Nếu lần lượt sử dụng học có giám sát và học tăng cường, mỗi phương pháp cần loại dữ liệu gì? Phương pháp nào phù hợp hơn với nhiệm vụ này?
   - Trong quá trình huấn luyện mô hình ngôn ngữ lớn, học tăng cường đóng vai trò then chốt gì?

6. Paradigm tiền huấn luyện-tinh chỉnh là bước đột phá quan trọng của lĩnh vực trí tuệ nhân tạo hiện đại. Hãy suy nghĩ sâu hơn:
   
   - Tại sao nói tiền huấn luyện đã giải quyết vấn đề "nút thắt cổ chai trong thu thập tri thức" của thời kỳ chủ nghĩa ký hiệu? Chúng khác biệt bản chất gì về phương thức biểu diễn tri thức?
   - Tri thức của mô hình tiền huấn luyện phần lớn đến từ dữ liệu Internet, điều này có thể mang lại những vấn đề gì? Làm thế nào để giảm thiểu những vấn đề trên?
   - Bạn cho rằng paradigm "tiền huấn luyện-tinh chỉnh" có khả năng bị thay thế bởi một paradigm mới nào đó hay không? Hay nó sẽ tồn tại lâu dài?
   
7. Giả sử bạn cần thiết kế một "trợ lý review code thông minh", nó có thể tự động review các commit được nộp lên (Pull Request), tóm tắt logic triển khai của code, kiểm tra chất lượng code, phát hiện BUG tiềm ẩn, đề xuất cải tiến.
   
   - Nếu thiết kế hệ thống này vào thời kỳ chủ nghĩa ký hiệu (những năm 1980), bạn sẽ hiện thực hóa như thế nào? Sẽ gặp phải khó khăn gì?
   - Nếu thiết kế vào thời kỳ học sâu chưa có mô hình ngôn ngữ lớn (khoảng năm 2015), bạn sẽ hiện thực hóa như thế nào?
   - Trong thời đại mô hình ngôn ngữ lớn và Agent hiện nay, bạn sẽ thiết kế kiến trúc của Agent này như thế nào? Nó nên bao gồm những module nào (tham khảo Hình 2.10)?
   - So sánh phương án của ba thời kỳ này, hãy trình bày sự tiến hóa của công nghệ Agent đã khiến nhiệm vụ này từ "gần như bất khả thi" trở thành "khả thi" như thế nào



## Tài liệu tham khảo

[1] NEWELL A, SIMON H A. Computer science as empirical inquiry: symbols and search[J]. Communications of the ACM, 1976, 19(3): 113-126.

[2] BUCHANAN B G, SHORTLIFFE E H, ed. Rule-based expert systems: the MYCIN experiments of the Stanford Heuristic Programming Project[M]. Reading, Mass.: Addison-Wesley, 1984.

[3] WINOGRAD T. Understanding natural language[M]. New York: Academic Press, 1972.

[4] LENAT D B, GUHA R V. Cyc: a midterm report[J]. AI magazine, 1990, 11(3): 32.

[5] MCCARTHY J, HAYES P J. Some philosophical problems from the standpoint of artificial intelligence[C]//MELTZER B, MICHIE D, ed. Machine intelligence 4. Edinburgh: Edinburgh University Press, 1969: 463-502.

[6] WEIZENBAUM J. ELIZA: a computer program for the study of natural language communication between man and machine[J]. Communications of the ACM, 1966, 9(1): 36-45.

[7] MINSKY M. The society of mind[M]. New York: Simon & Schuster, 1986.

[8] RUMELHART D E, MCCLELLAND J L, PDP RESEARCH GROUP. Parallel distributed processing: explorations in the microstructure of cognition[M]. Cambridge, MA: MIT Press, 1986.

[9] SILVER D, HUANG A, MADDISON C J, ed. Mastering the game of Go with deep neural networks and tree search[J]. Nature, 2016, 529(7587): 484-489.

[10] LETTA. The AI agents stack[EB/OL]. (2024-11) [2025-09-07]. https://www.letta.com/blog/ai-agents-stack.
