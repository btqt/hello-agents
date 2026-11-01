# GUI Agent: Phổ cập kiến thức và thực chiến — Hành trình khám phá thế hệ tương tác người-máy tiếp theo

## Mở đầu: Khi AI học cách "nhìn" màn hình

Hãy tưởng tượng một tình huống như sau: bạn nói với điện thoại "giúp tôi đặt một vé tàu cao tốc đi Thượng Hải vào ngày mai, ghế hạng hai, khởi hành khoảng 10 giờ sáng", sau đó AI tự động mở ứng dụng đường sắt 12306, điền điểm xuất phát, điểm đến và ngày tháng, lọc các chuyến tàu phù hợp điều kiện, hoàn tất đặt vé và thanh toán — toàn bộ quá trình không cần bạn thao tác thủ công, AI giống như một trợ lý thực thụ, "nhìn" màn hình, "hiểu" giao diện, "nhấn" nút.

Đây không phải là khoa học viễn tưởng, mà là hiện thực đang được **GUI Agent (Agent giao diện người dùng đồ họa)** hiện thực hóa.

Trong hai mươi năm qua, giải pháp chủ đạo cho tự động hóa doanh nghiệp là **RPA (Robotic Process Automation - Tự động hóa quy trình bằng robot)**. Tuy nhiên, RPA có một điểm yếu chí tử: nó phụ thuộc vào các bộ chọn phần tử UI cố định (Selectors), chỉ cần giao diện thay đổi một chút là script sẽ bị hỏng. Tính mong manh này dẫn đến chi phí bảo trì khổng lồ.

Sự xuất hiện của GUI Agent đã thay đổi hoàn toàn cục diện này. Nó không đơn giản là "phát lại" một script được định sẵn, mà giống như con người, thông qua **nhận thức thị giác** để hiểu nội dung màn hình, thông qua **khả năng suy luận của mô hình ngôn ngữ lớn** để lập kế hoạch đường đi thao tác, tự chủ hoàn thành tác vụ trong môi trường phần mềm động và chưa biết trước.

Chương này sẽ dẫn dắt bạn tìm hiểu sâu về nguyên lý kỹ thuật của GUI Agent, và thông qua ba trường hợp thực chiến, giúp bạn thực sự nắm vững cách sử dụng và triển khai các hệ thống Agent tiên tiến này.

## Phần 1: Phổ cập kiến thức kỹ thuật GUI Agent

### 1.1 GUI Agent là gì?

**GUI Agent (Agent giao diện người dùng đồ họa)** là một loại hệ thống AI có khả năng tự chủ hiểu và thao tác giao diện đồ họa. Khác với việc gọi API truyền thống hay công cụ dòng lệnh, GUI Agent tương tác trực tiếp với giao diện đồ họa mà con người sử dụng — bất kể đó là ứng dụng điện thoại, phần mềm desktop hay ứng dụng web.

#### 1.1.1 Sự chuyển đổi mô hình từ RPA sang AI Agent

Hãy cùng hiểu rõ sự chuyển đổi này thông qua một bảng so sánh:

| Khía cạnh           | RPA truyền thống                                 | GUI Agent (AI Agent)                |
| -------------- | ---------------------------------------- | ------------------------------------ |
| **Nguyên lý hoạt động**   | Phát lại script dựa trên bộ chọn cố định (như XPath, ID) | Thao tác tự chủ dựa trên hiểu thị giác và suy luận mô hình ngôn ngữ |
| **Khả năng thích ứng**     | Giao diện thay đổi là hỏng ngay                          | Thích ứng được với thay đổi giao diện, có tính đàn hồi ngữ nghĩa         |
| **Lập kế hoạch tác vụ**   | Cần đặt sẵn thủ công từng bước thao tác                   | Tự chủ phân rã tác vụ theo chỉ dẫn ngôn ngữ tự nhiên         |
| **Khả năng đa nền tảng** | Cần viết script riêng cho mỗi nền tảng              | Giải pháp thị giác tổng quát, tự nhiên đa nền tảng         |
| **Chi phí bảo trì**   | Cực cao (UI thay đổi phải viết lại script)              | Thấp (mô hình tự động thích ứng)                   |

**Khác biệt cốt lõi**: RPA là "tự động hóa mong manh", còn GUI Agent là "tự chủ hóa thông minh".

#### 1.1.2 Vì sao GUI Agent đột nhiên trở nên nổi bật?

Sự bùng nổ của GUI Agent không phải ngẫu nhiên, mà là kết quả của việc nhiều lĩnh vực kỹ thuật cùng chín muồi đồng thời. Đầu tiên là bước tiến đột phá của mô hình lớn đa phương thức (multimodal). Bắt đầu từ các mô hình như GPT-4o, Claude 3.5 Sonnet, Qwen-VL, mô hình lớn không chỉ hiểu văn bản mà còn "nhìn hiểu" hình ảnh, điều này mang lại cho GUI Agent một "đôi mắt" mạnh mẽ. Khi bạn đưa một ảnh chụp màn hình cho các mô hình này, chúng có thể nhận diện chính xác "đây là nút đăng nhập", "ở đây có một ô tìm kiếm", thậm chí hiểu được bố cục giao diện phức tạp.

Quan trọng hơn là đột phá về khả năng định vị. Các mô hình thị giác thời kỳ đầu giống như một người cận thị — nó biết trên màn hình có một nút, nhưng không thể nói rõ nút ở đâu. Còn các mô hình mới nhất (như GUI-Owl, Qwen-VL) được huấn luyện chuyên biệt, có thể xuất ra chính xác tọa độ màn hình $(x, y)$ của phần tử UI, điều này giúp Agent không chỉ "nhìn thấy" mà còn "nhấn trúng".

Cuối cùng là sự thay đổi về chất của khả năng suy luận. Khả năng tư duy chuỗi (Chain of Thought) của mô hình ngôn ngữ lớn mang lại cho Agent một "bộ não". Nó có thể phân rã một chỉ dẫn mơ hồ như "đặt một vé tàu cao tốc ngày mai" thành các bước cụ thể như "mở APP → chọn ngày → nhập địa điểm → lọc chuyến tàu → xác nhận thanh toán", và liên tục tự phản tư, sửa lỗi trong quá trình thực thi.

### 1.2 Kiến trúc kỹ thuật cốt lõi của GUI Agent

Một hệ thống GUI Agent hoàn chỉnh có thể được phân rã thành ba mô-đun cốt lõi: **Nhận thức (Perception)** → **Suy luận (Reasoning)** → **Thực thi (Action)**. Đây là một hệ thống ra quyết định tự chủ theo vòng lặp khép kín.

<div align="center">
  <img src="./images/Extra06-figures/image1.png" alt="GUI Agent 三层架构" width="90%"/>
  <p>Hình 1 Vòng lặp khép kín Nhận thức - Suy luận - Thực thi của GUI Agent</p>
</div>

#### 1.2.1 Tầng nhận thức: Máy móc "nhìn thấy" màn hình như thế nào

Tầng nhận thức chịu trách nhiệm chuyển đổi thông tin màn hình thành dữ liệu mà máy móc có thể hiểu được. Hiện tại chủ yếu có hai hướng kỹ thuật, đại diện cho hai triết lý thiết kế hoàn toàn khác nhau.

Hướng thứ nhất là nhận thức có cấu trúc dựa trên DOM hoặc cây khả năng truy cập (accessibility tree). Phương pháp này lấy cấu trúc nội bộ của ứng dụng thông qua API hệ thống — ví dụ như cây DOM HTML của trang web, hoặc View Hierarchy của ứng dụng Android. Giống như cung cấp cho Agent một "bản vẽ kiến trúc", nó có thể biết chính xác loại và vị trí của từng nút, ô văn bản. Ưu điểm của phương pháp này là chính xác và hiệu quả, nhưng vấn đề cũng rất rõ ràng: nhiều ứng dụng hiện đại hoàn toàn không phơi bày những thông tin cấu trúc này. Giao diện vẽ bằng Canvas, game, phần mềm desktop từ xa, đối với giải pháp dựa trên DOM đều là "hộp đen". Hơn nữa phương pháp này mất đi thông tin bố cục thị giác, rất khó hiểu được quan hệ không gian giữa các phần tử, khả năng tương thích đa nền tảng cũng kém.

Hướng thứ hai là nhận thức dựa thuần túy trên thị giác, đây cũng là hướng tiên phong nhất hiện nay. Agent chụp trực tiếp hình ảnh màn hình, dùng mô hình lớn thị giác (VLM) để "nhìn" màn hình giống như con người. Tính tổng quát của phương pháp này cực kỳ mạnh — bất kể giao diện của bạn được xây dựng bằng công nghệ gì, chỉ cần hiển thị được trên màn hình, Agent đều có thể hiểu. Quan trọng hơn, nó có "tính đàn hồi ngữ nghĩa". Ngay cả khi một nút nào đó chuyển từ màu xanh dương sang màu xanh lá, hoặc vị trí dịch chuyển một chút, Agent dựa trên thị giác vẫn có thể nhận diện bằng ngữ nghĩa rằng "đây là nút đăng nhập". RPA truyền thống gặp tình huống này sẽ hỏng, nhưng GUI Agent có thể dễ dàng ứng phó. Tất nhiên, giải pháp thuần thị giác cũng có thách thức, khó khăn lớn nhất là độ chính xác định vị — mô hình không chỉ phải nhận diện nút là gì, mà còn phải xuất ra tọa độ màn hình chính xác của nó.

#### 1.2.2 Tầng suy luận: Quá trình ra quyết định của bộ não

Tầng suy luận là "bộ não" của GUI Agent, chịu trách nhiệm chuyển đổi chỉ dẫn trừu tượng của người dùng thành chuỗi thao tác cụ thể. Ở đây liên quan đến vài năng lực then chốt.

Đầu tiên là khả năng phân rã tác vụ. Khi bạn nói với Agent "giúp tôi đặt một vé tàu cao tốc đi Thượng Hải vào ngày mai, ghế hạng hai, khởi hành khoảng 10 giờ sáng", nó cần hiểu logic phức tạp đằng sau câu nói này. Agent sẽ tự động tách nhu cầu mơ hồ này thành một chuỗi các bước cụ thể: mở ứng dụng 12306 → nhấn "đặt vé tàu" → nhập điểm xuất phát "Bắc Kinh" → nhập điểm đến "Thượng Hải" → chọn ngày "ngày mai" → nhấn tìm kiếm → lọc chuyến tàu (ghế hạng hai + khoảng 10 giờ sáng) → chọn chuyến phù hợp điều kiện → nhấn đặt vé → điền thông tin hành khách → xác nhận thanh toán. Quá trình phân rã này dựa vào sự hiểu biết của mô hình ngôn ngữ lớn về kiến thức thông thường và quy trình nghiệp vụ.

Tinh vi hơn là cơ chế chuỗi tư duy. Để tăng tỷ lệ thành công của các tác vụ phức tạp, GUI Agent hiện đại sẽ sinh ra "lời độc thoại nội tâm" trước mỗi bước thao tác. Ví dụ màn hình hiện tại là trang chủ 12306, mục tiêu của người dùng là đặt vé tàu cao tốc, Agent sẽ phân tích trước: "Tôi thấy trên màn hình có các lựa chọn như 'đặt vé tàu', 'tra cứu đơn hàng', cần nhấn vào 'đặt vé tàu' mới có thể vào quy trình mua vé." Sau đó ra quyết định: "Nhấn vào nút 'đặt vé tàu' tại tọa độ (540, 320)." Quá trình tư duy tường minh này không chỉ làm cho hành vi của Agent dễ giải thích hơn, mà còn giảm đáng kể sự tích lũy sai số trong các thao tác nhiều bước.

Cuối cùng là khả năng tự phản tư và sửa lỗi. Nếu sau khi Agent nhấn nút "tìm kiếm", phát hiện không xuất hiện danh sách chuyến tàu như mong đợi, mà lại hiện ra thông báo "vui lòng chọn ngày khởi hành", nó sẽ lập tức nhận ra: "Mình đã bỏ sót bước chọn ngày." Sau đó điều chỉnh chiến lược: "Nhấn vào bộ chọn ngày trước, chọn ngày mai, rồi tìm kiếm lại." Khả năng tự sửa lỗi này giúp Agent có thể ứng phó với các tình huống bất ngờ khác nhau trong thế giới thực.

#### 1.2.3 Tầng thực thi: Từ quyết định đến hành động

Tầng thực thi là "đôi tay" của GUI Agent, chịu trách nhiệm chuyển đổi quyết định của mô hình thành thao tác hệ thống thực tế.

Khác với không gian mở của việc sinh văn bản, không gian hành động của thao tác GUI là hữu hạn và rõ ràng. Nhấn, nhấn đúp, nhấn giữ, vuốt, nhập liệu, cuộn, kéo thả — những hành động cơ bản này tạo thành nền tảng cho mọi thao tác phức tạp. Mỗi loại hành động có tham số riêng của nó, ví dụ nhấn cần tọa độ (x, y), vuốt cần điểm bắt đầu và điểm kết thúc (x1, y1, x2, y2), nhập liệu cần nội dung văn bản.

Có một chi tiết kỹ thuật then chốt ở đây: sự chuyển đổi hệ tọa độ. Mô hình thị giác (như Qwen-VL) thường xuất ra tọa độ chuẩn hóa (0-1000), trong khi độ phân giải màn hình thực tế của điện thoại hay máy tính có thể là 1920x1080. Tầng thực thi phải thực hiện ánh xạ tọa độ chính xác, chuyển đổi đầu ra của mô hình thành tọa độ vật lý. Hơn nữa các thiết bị khác nhau còn có DPI và tỷ lệ scale hệ thống khác nhau, tất cả đều cần được tính đến. Một hàm ánh xạ đơn giản có thể như sau: đầu tiên chia tọa độ chuẩn hóa cho 1000, rồi nhân với chiều rộng/cao thực tế của màn hình, cuối cùng làm tròn để được tọa độ vật lý.

Phức tạp hơn là việc thích ứng đa nền tảng. Trên Android, mọi thao tác đều được thực hiện bằng cách gửi lệnh qua ADB (Android Debug Bridge), ví dụ `adb shell input tap 500 1000` để thực hiện nhấn, `adb shell input swipe 500 1000 500 500` để thực hiện vuốt. Trên iOS, cần thông qua libimobiledevice hoặc WDA (WebDriverAgent) để thực hiện chức năng tương tự. Còn trên môi trường desktop Windows, Mac, Linux, thường dùng các thư viện Python như pyautogui, pynput để điều khiển trực tiếp chuột và bàn phím. Cùng một hành động "nhấn", cách triển khai trên các nền tảng khác nhau hoàn toàn khác nhau, tầng thực thi cần cung cấp một giao diện trừu tượng thống nhất cho mỗi nền tảng.

### 1.3 So sánh toàn cảnh các framework mã nguồn mở chủ đạo

Giai đoạn 2024-2025 là thời kỳ bùng nổ của GUI Agent, các công ty công nghệ lớn và các tổ chức nghiên cứu lần lượt mã nguồn mở hóa framework của riêng mình. Hãy cùng so sánh có hệ thống vài dự án tiêu biểu nhất:

<div align="center">
  <img src="./images/Extra06-figures/image2.png" alt="主流GUI Agent框架对比" width="90%"/>
  <p>Hình 2 Biểu đồ radar so sánh toàn cảnh các framework GUI Agent chủ đạo</p>
</div>

### 1.4 Kịch bản ứng dụng và giới hạn kỹ thuật

#### 1.4.1 Năm kịch bản ứng dụng điển hình

Tiềm năng ứng dụng của GUI Agent vượt xa trí tưởng tượng của chúng ta. Trong lĩnh vực cockpit thông minh (smart cockpit), nhu cầu tương tác giọng nói trong quá trình lái xe đang bùng nổ. Hãy tưởng tượng bạn nói trong khi lái xe "dẫn đường đến quán cà phê gần nhất, và giúp tôi đặt một ly latte 10 phút trước khi đến", GUI Agent có thể phối hợp xuyên ứng dụng giữa app dẫn đường và app đặt đồ ăn, hiểu logic thời gian phức tạp, còn có thể thích ứng với sự khác biệt về UI của các hãng xe khác nhau. Đây chính là điều mà hệ thống xe hơi truyền thống khó làm được.

Trong lĩnh vực kiểm thử phần mềm, GUI Agent mang lại sự thay đổi mang tính cách mạng. Kiểm thử tự động truyền thống dựa vào các công cụ như Selenium, mỗi lần UI thay đổi đều cần cập nhật script kiểm thử, chi phí bảo trì cực cao. Còn GUI Agent có thể tự thích ứng với thay đổi UI — ngay cả khi vị trí nút được điều chỉnh, màu sắc thay đổi, Agent vẫn có thể tìm thấy đúng phần tử thông qua nhận diện ngữ nghĩa. Nó còn có thể thực hiện kiểm thử hồi quy trực quan (visual regression testing), tự động phát hiện bất thường UI, thậm chí chủ động thực hiện kiểm thử khám phá (exploratory testing), phát hiện những trường hợp biên mà kỹ sư kiểm thử con người có thể bỏ sót.

Kịch bản RPA cấp doanh nghiệp là một thị trường khổng lồ khác. RPA truyền thống không thể xử lý những hệ thống cũ không có API, nhưng GUI Agent thì có thể. Trích xuất dữ liệu từ Excel, điền vào hệ thống ERP, gửi email thông báo — toàn bộ workflow xuyên hệ thống có thể hoàn toàn tự động hóa. Đối với những hệ thống legacy đã vận hành hai ba chục năm, không có bất kỳ giao diện hiện đại nào, GUI Agent cuối cùng đã mang lại khả năng tự động hóa.

Trong cuộc sống cá nhân, GUI Agent có thể trở thành trợ lý thông minh thực thụ. Đăng nội dung theo lịch lên nhiều nền tảng mạng xã hội, tự động tổng hợp tin tức, thời tiết, lịch trình mỗi sáng, ghi lại dữ liệu vận động và thói quen ăn uống — những lao động số lặp đi lặp lại này đều có thể giao cho Agent thực hiện. Còn đối với người dùng khiếm thị, khuyết tật vận động, GUI Agent càng mở ra cánh cửa tới một thế giới mới. Hoàn toàn điều khiển điện thoại bằng giọng nói, đọc thông minh nội dung màn hình, chuyển đổi thao tác phức tạp thành chỉ dẫn đơn giản, những tính năng này đang giúp công nghệ thực sự mang lại lợi ích cho mọi người.

#### 1.4.2 Ba giới hạn lớn của công nghệ hiện tại

Nhưng chúng ta cũng phải nhận thức rõ ràng rằng, công nghệ GUI Agent vẫn đang ở giai đoạn phát triển sớm, đối mặt với một số thách thức thực chất.

Đáng lo ngại nhất là rủi ro về bảo mật và ảo giác (hallucination). Vấn đề ảo giác của mô hình ngôn ngữ lớn trên GUI Agent có thể dẫn đến hậu quả nghiêm trọng. Người dùng yêu cầu "dọn dẹp desktop", Agent có thể hiểu nhầm thành xóa toàn bộ file; một con số sai trong thao tác chuyển khoản có thể gây thiệt hại kinh tế. Các giải pháp giảm thiểu hiện nay bao gồm: bắt buộc xác nhận thủ công đối với các thao tác rủi ro cao, ghi nhật ký thao tác chi tiết và hỗ trợ hoàn tác (rollback), cũng như kiểm thử đầy đủ trong môi trường sandbox. Nhưng đây đều chỉ là giải pháp tạm thời, việc giải quyết triệt để vấn đề ảo giác của mô hình vẫn cần thời gian.

Vấn đề chi phí và hiệu quả cũng không thể xem nhẹ. Mỗi bước thao tác đều cần gọi mô hình lớn để suy luận, nếu dùng API đám mây, chi phí sẽ tăng tuyến tính theo số lần gọi. Một tác vụ phức tạp có thể cần hàng chục lần lặp, tổng thời gian tương đối dài. Triển khai mô hình nhỏ cục bộ có thể giảm chi phí, nhưng độ chính xác sẽ giảm xuống. Cache thao tác, nhận diện mẫu, kiến trúc lai (tác vụ đơn giản dùng RPA, tác vụ phức tạp dùng AI) là các hướng đang được khám phá hiện nay, nhưng vẫn chưa hình thành thực hành tốt nhất chín muồi.

Cuối cùng là nút thắt về độ chính xác. Ngay cả hệ thống tốt nhất, tỷ lệ thành công trong kịch bản thực tế cũng chỉ khoảng 40-50%. Định vị phần tử trong giao diện phức tạp, xử lý nội dung động (quảng cáo, popup), tích lũy lỗi trong tác vụ chuỗi dài, đây đều là những khó khăn kỹ thuật thực sự. Hướng đột phá bao gồm mô hình lớn thị giác mạnh hơn, tối ưu chiến lược thao tác thông qua học tăng cường (reinforcement learning), cũng như thiết kế hợp tác "con người trong vòng lặp" (Human-in-the-loop). Nhưng để nâng từ 50% lên mức 90% khả dụng để thương mại hóa, có lẽ vẫn cần thêm thời gian nữa.

---

## Phần 2: Hướng dẫn thực chiến GUI Agent

Sau khi học lý thuyết, hãy cùng thông qua hai trường hợp thực chiến có độ khó tăng dần, thực sự nắm vững cách sử dụng và triển khai GUI Agent.

### Thực chiến 1: Trải nghiệm trực tuyến Mobile-Agent (không cần cài đặt)

#### 2.1.1 Truy cập Demo trực tuyến

Mobile-Agent-v3 không chỉ hỗ trợ điện thoại, mà còn có thể thao tác máy tính. Như Hình 3 cho thấy, trong trang Demo của ModelScope, chúng ta chuyển lựa chọn thiết bị ở góc trên bên trái thành "Máy tính", là có thể vào môi trường trải nghiệm PC Agent.

**Lựa chọn 1: ModelScope Demo** (khuyến nghị)
Liên kết: https://modelscope.cn/studios/wangjunyang/Mobile-Agent-v3

**Lựa chọn 2: Alibaba Cloud Bailian**
Liên kết: https://bailian.console.aliyun.com/next?tab=demohouse#/experience/adk-computer-use/pc

Cả hai nền tảng này đều cung cấp **môi trường điện thoại đám mây/máy tính đám mây**, không cần triển khai cục bộ vẫn có thể trải nghiệm đầy đủ tính năng.

### 2.1.2 Hướng dẫn tính năng giao diện

Sau khi vào trang, bạn sẽ thấy giao diện thao tác như Hình 3. Để đảm bảo trải nghiệm nhất quán, hãy nhớ thực hiện các **thiết lập then chốt** sau:

1. **Chọn thiết bị**: trong menu thả xuống ở góc trên bên trái, xác nhận chọn **"Máy tính"** (không phải điện thoại).
2. **Xem trước desktop**: cửa sổ bên phải hiển thị desktop Windows 10 được cấp phát trên đám mây cho bạn, đã cài sẵn các phần mềm cơ bản như Office, trình duyệt.
3. **Vùng tương tác**: góc dưới bên trái là vùng nhập chỉ dẫn, quá trình tư duy (Thinking Process) và các bước thao tác của Agent sẽ hiển thị trong hộp thoại phía trên.

<div align="center">
  <img src="./images/Extra06-figures/image3.png" alt="ModelScope Demo界面" width="90%"/>
  <p>Hình 3 Giải thích giao diện Demo trực tuyến Mobile-Agent-v3</p>
</div>

Trong giao diện này, bạn có thể trực tiếp chỉ huy Agent thực hiện các thao tác văn phòng, tuy nhiên hiện tại thời gian sử dụng có giới hạn.

### 2.1.3 Thực hành tác vụ điển hình

Dựa theo năng lực có sẵn mà giao diện cung cấp, khuyến nghị người mới bắt đầu thử với hai loại tác vụ sau:

- **Điều khiển cấp hệ thống**: thử để Agent thay đổi thiết lập hệ thống.
  - *Ví dụ chỉ dẫn*: "Đặt màu hệ thống thành **chế độ sáng (light mode)**."
  - *Điểm quan sát*: Agent có thể mở "Start Menu -> Settings -> Personalization" giống như con người không.
- **Làm việc văn phòng xuyên ứng dụng**: thử để Agent kết hợp trình duyệt và phần mềm văn phòng.
  - *Ví dụ chỉ dẫn*: "Tìm giá cổ phiếu của Alibaba trong trình duyệt Edge, sau đó tạo một bảng tính mới trong WPS, điền tên công ty và giá cổ phiếu hiện tại."
  - *Điểm quan sát*: Agent có thể xử lý chính xác việc chuyển đổi ngữ cảnh xuyên phần mềm từ "tìm kiếm thông tin" sang "nhập thông tin" không.

### 2.1.4 Kỹ thuật prompt: Cách chỉ huy PC Agent

Trong kịch bản GUI, Prompt chất lượng cao là chìa khóa thành công. Kết hợp với kịch bản văn phòng nêu trên, chúng tôi tổng kết ba kỹ thuật cốt lõi:

1. **Xác định rõ ranh giới ứng dụng (Explicit Context)**
   - Tránh các chỉ dẫn chung chung như "viết một bài giới thiệu".
   - **Cách viết khuyến nghị**: "Viết một đoạn giới thiệu trong **tài liệu WPS Office**..."
   - *Giải thích*: chỉ định rõ tên phần mềm (App Name) giúp giảm thời gian Agent tìm công cụ.
2. **Phân rã theo chuỗi bước (Chain of Steps)**
   - Đừng cố gắng gói gọn toàn bộ logic phức tạp trong một câu.
   - **Cách viết khuyến nghị**: "Bước 1, mở Edge tìm kiếm...; Bước 2, sau khi xác nhận trang web tải xong, chụp lại dữ liệu...; Bước 3, mở Excel dán vào."
   - *Giải thích*: thao tác GUI có tính thứ tự thời gian nghiêm ngặt, chỉ dẫn từng bước giúp giảm đáng kể tỷ lệ lỗi thực thi.
3. **Mô tả thuộc tính thị giác (Visual Attributes)**
   - Agent thao tác bằng cách "nhìn" màn hình, nên mô tả bằng đặc điểm thị giác sẽ hiệu quả hơn.
   - **Cách viết khuyến nghị**: "Nhấn vào **nút lưu màu xanh dương** ở góc trên bên phải" hoặc "đổi màu chữ thành **màu đỏ**".

#### 2.1.5 Giá trị và giới hạn của trải nghiệm trực tuyến

Giá trị lớn nhất của Demo trực tuyến mà ModelScope cung cấp là **trải nghiệm không cần cài đặt gì**. Bạn không cần cấu hình bất kỳ môi trường nào, không cần chuẩn bị điện thoại, thậm chí không cần tải xuống bất kỳ phần mềm nào, vẫn có thể trực tiếp cảm nhận sức mạnh của GUI Agent. Điều này rất hữu ích để nhanh chóng xác thực ý tưởng, tìm hiểu giới hạn của công nghệ.

Nhưng môi trường trực tuyến cũng có giới hạn riêng. Đầu tiên là **vấn đề quyền riêng tư**, mọi thao tác đều diễn ra trên máy ảo đám mây, bạn không thể truy cập dữ liệu cá nhân thực tế. Thứ hai là **giới hạn tính năng**, môi trường ảo chỉ cài sẵn một số ứng dụng thường dùng, không thể kiểm thử các kịch bản ứng dụng cụ thể. Cuối cùng là **khác biệt về hiệu năng**, độ trễ của suy luận trên đám mây sẽ cao hơn một chút so với triển khai cục bộ.

Vì vậy, trải nghiệm trực tuyến phù hợp làm điểm khởi đầu để học tập và khám phá, nhưng nếu muốn ứng dụng GUI Agent vào kịch bản thực tế, bạn cần thử triển khai cục bộ. Mobile-Agent-v3 chính thức cung cấp một [hướng dẫn](https://github.com/X-PLUG/MobileAgent/blob/main/Mobile-Agent-v3/README_zh.md), bạn có thể tự mình thử.

Phần thực chiến 2 tiếp theo sẽ dẫn bạn dùng AutoGLM vừa được **Zhipu** mã nguồn mở để bước vào thế giới sâu hơn này.

---

### Thực chiến 2: Triển khai cục bộ AutoGLM và thực chiến trên điện thoại

Trải nghiệm trực tuyến giúp chúng ta cảm nhận được năng lực của GUI Agent, nhưng sức mạnh thực sự nằm ở việc triển khai trên thiết bị của chính mình, điều khiển các ứng dụng thực tế. AutoGLM là một framework rất phù hợp cho các nhà phát triển cá nhân mới bắt đầu, kiến trúc của nó rõ ràng, tài liệu đầy đủ, quá trình triển khai tương đối đơn giản.

Mục tiêu của thực chiến này là triển khai AutoGLM trên máy tính của bạn, kết nối với điện thoại Android, sau đó để AI giúp bạn hoàn thành một số tác vụ thực tế — ví dụ như tự động trả lời tin nhắn WeChat, hoặc làm mới định kỳ một ứng dụng nào đó để lấy dữ liệu mới nhất.

#### 2.2.1 Chuẩn bị môi trường: Bạn cần gì

Việc triển khai Open-AutoGLM cần hai thiết bị cốt lõi: một máy tính có thể chạy Python, và một chiếc điện thoại Android. Cấu hình máy tính không cần quá cao, vì AutoGLM hỗ trợ gọi API đám mây, không nhất thiết phải chạy mô hình lớn cục bộ. Nếu bạn định dùng API đám mây (như GLM-4V của Zhipu), một chiếc laptop bình thường là đủ. Nhưng nếu bạn muốn trải nghiệm giải pháp hoàn toàn cục bộ, thì một GPU có ít nhất 8GB VRAM sẽ giúp trải nghiệm tốt hơn nhiều.

Về điện thoại, Android 7.0 trở lên đều được, không cần quyền Root. Người dùng iPhone hiện chưa thể sử dụng, vì tính đóng kín của iOS khiến giải pháp debug ADB không thể áp dụng trực tiếp.

Về môi trường phần mềm, bạn cần cài Python 3.10 trở lên, cùng với công cụ ADB (Android Debug Bridge). ADB là cầu nối giữa máy tính và điện thoại, mọi thao tác chụp màn hình, nhấn, vuốt đều phải thực hiện thông qua nó.

**Cài đặt công cụ ADB (macOS / Linux):** Tùy theo hệ thống của bạn, chạy lệnh sau trong terminal:

```bash
# macOS dùng Homebrew
brew install android-platform-tools

# Linux (Ubuntu/Debian)
sudo apt install android-tools-adb
```

Người dùng Windows thường có thể tải trực tiếp gói nén Platform Tools và cấu hình biến môi trường. [Tham khảo](https://blog.csdn.net/x2584179909/article/details/108319973)

#### 2.2.2 Bước 1: Cài đặt Open-AutoGLM

Nếu bạn có **Claude Code**, bạn có thể cấu hình [GLM Coding Plan](https://bigmodel.cn/glm-coding) sau đó nhập prompt sau để triển khai nhanh:

```
Truy cập tài liệu, cài đặt AutoGLM cho tôi
https://raw.githubusercontent.com/zai-org/Open-AutoGLM/refs/heads/main/README.md
```

Nếu không có CLI tương tự, hãy thao tác theo các bước thủ công sau:

Mở terminal dòng lệnh, trước tiên clone kho code của Open-AutoGLM:

```bash
git clone https://github.com/zai-org/Open-AutoGLM.git
cd Open-AutoGLM
```

Tiếp theo cài đặt dependency. Ngoài các gói dependency cơ bản, **nhất định phải chạy lệnh cài đặt của dự án**, để đảm bảo mọi module đều được gọi đúng cách:

```bash
# 1. Cài đặt dependency cơ bản
pip install -r requirements.txt

# 2. Cài đặt chính dự án ở chế độ chỉnh sửa (editable) (bước then chốt)
pip install -e .

# 3. (Tùy chọn) Nếu bạn là nhà phát triển, cần cài thêm dependency phát triển
pip install -e ".[dev]"
```

Quá trình này thường mất vài phút, tùy thuộc vào tốc độ mạng của bạn. Sau khi cài đặt xong, bạn cần cấu hình API key. Nếu dùng API GLM-4V của Zhipu, trước tiên hãy đăng ký tài khoản trên nền tảng mở của Zhipu và lấy API Key, sau đó tạo một file `.env` ở thư mục gốc của dự án:

```bash
# Nội dung file .env
GLM_API_KEY=your_api_key_here
```

[AutoGLM-Phone-9B · Kho mô hình](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B)

#### 2.2.3 Bước 2: Kết nối điện thoại Android của bạn

Bây giờ đến bước then chốt: giúp máy tính có thể "nhìn thấy" và "điều khiển" điện thoại của bạn. Việc này cần ba bước nhỏ: bật chế độ nhà phát triển, bật USB debugging, và **cài đặt ADB Keyboard**.

**1. Bật chế độ nhà phát triển & USB debugging** Trên điện thoại Android, vào "Settings" → "About phone", tìm "Build number", **nhấn liên tục 7 lần** (hoặc đến khi có thông báo), bạn sẽ thấy thông báo "Bạn đã ở chế độ nhà phát triển". Quay lại màn hình chính của Settings, vào "Developer options", tìm "USB debugging" và **bật** nó lên.

**2. Cài đặt ADB Keyboard (bắt buộc)** Để AI có thể nhập chữ trên điện thoại, chúng ta cần cài bàn phím ADB chuyên dụng.

- Link tải: https://github.com/senzhk/ADBKeyBoard/raw/master/ADBKeyboard.apk

Sau khi cài đặt, nhớ vào mục "Bàn phím (Input method)" trong cài đặt điện thoại, bật và chuyển sang **ADB Keyboard**.

**3. Xác minh kết nối** Dùng cáp USB kết nối điện thoại với máy tính (khi điện thoại hiện hộp thoại xin cấp quyền, nhấn "Cho phép"). Trong terminal máy tính, nhập:

Bash

```
adb devices
```

Nếu mọi thứ bình thường, bạn sẽ thấy số serial của thiết bị:

```
List of devices attached
ABC12345    device
```

Nếu hiển thị `device`, chúc mừng bạn, kết nối phần cứng đã thông suốt! Nếu hiển thị `unauthorized`, hãy kiểm tra xem màn hình điện thoại có hiện hộp thoại xác nhận cấp quyền không.

Đối với người dùng Windows, có thể còn cần cài đặt driver cho điện thoại. Phần lớn điện thoại các hãng (như Xiaomi, Huawei, OPPO) sẽ tự động cài driver khi kết nối với máy tính, nhưng nếu gặp vấn đề, có thể tải driver USB tương ứng trên trang web chính thức.

<div align="center">
  <img src="./images/Extra06-figures/image4.png" alt="手机ADB连接配置步骤" width="90%"/>
  <p>Hình 4 Quy trình cấu hình kết nối ADB hoàn chỉnh cho điện thoại Android</p>
</div>

#### 2.2.4 Bước 3: Chạy tác vụ đầu tiên của bạn

Sau khi kết nối thành công, hãy cùng thực hiện một tác vụ đơn giản nhưng thực dụng.

Có hai cách kết nối gọi API trực tiếp:

**1. Zhipu BigModel**

- Tài liệu: https://docs.bigmodel.cn/cn/api/introduction
- `--base-url`: `https://open.bigmodel.cn/api/paas/v4`
- `--model`: `autoglm-phone`
- `--apikey`: xin API Key của bạn trên nền tảng Zhipu

**2. ModelScope (cộng đồng Modao)**

- Tài liệu: https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B
- `--base-url`: `https://api-inference.modelscope.cn/v1`
- `--model`: `ZhipuAI/AutoGLM-Phone-9B`
- `--apikey`: xin API Key của bạn trên nền tảng ModelScope

Trong readme chính thức có cung cấp một giao diện dòng lệnh, bạn có thể nhập trực tiếp:

```bash
# Dùng Zhipu BigModel
python main.py --base-url https://open.bigmodel.cn/api/paas/v4 --model "autoglm-phone" --apikey "your-bigmodel-api-key" "打开美团搜索附近的火锅店"

# Dùng ModelScope
python main.py --base-url https://api-inference.modelscope.cn/v1 --model "ZhipuAI/AutoGLM-Phone-9B" --apikey "your-modelscope-api-key" "打开美团搜索附近的火锅店"
```

Sau khi chạy lệnh này, AutoGLM sẽ khởi động quy trình suy luận. Bạn sẽ thấy log đầu ra theo thời gian thực trong terminal, đồng thời màn hình điện thoại sẽ bắt đầu tự động thao tác. Toàn bộ quá trình đại khái như sau:

Đầu tiên, AutoGLM sẽ chụp ảnh màn hình hiện tại thông qua ADB, gửi hình ảnh cho mô hình phân tích. Mô hình sẽ nhận diện tất cả icon ứng dụng trên màn hình, và định vị chính xác đến từng pixel vị trí của "Meituan". Sau đó AutoGLM gửi lệnh nhấn, thông qua `adb shell input tap x y` để mở ứng dụng.

Sau khi chờ Meituan khởi động, AutoGLM chụp màn hình lần nữa. Lần này mục tiêu của nó là tìm "thanh tìm kiếm" ở phía trên trang chủ. Sau khi nhận diện và nhấn vào ô tìm kiếm, **nó sẽ gọi ADB Keyboard mà chúng ta đã cài đặt ở bước chuẩn bị môi trường**, nhập chuỗi ký tự "lẩu gần đây" vào, cuối cùng tự động nhấn nút tìm kiếm.

Toàn bộ quá trình thường mất khoảng 15-20 giây (tác vụ tìm kiếm có nhiều bước hơn một chút), thời gian cụ thể tùy thuộc vào tốc độ suy luận của mô hình và độ trễ mạng. Nếu bạn dùng API đám mây, thời gian "suy nghĩ" của mỗi bước khoảng 2-3 giây. Nếu là mô hình triển khai cục bộ, GPU cấu hình tốt có thể rút ngắn thời gian mỗi bước xuống khoảng 1 giây.

---

## Tổng kết và triển vọng

Thông qua hai trường hợp thực chiến với mức độ tăng dần này, chúng ta đã trải nghiệm đầy đủ toàn bộ quá trình của GUI Agent từ demo trực tuyến đến triển khai cục bộ. Demo trực tuyến của Mobile-Agent giúp chúng ta nhanh chóng hiểu được tiềm năng của công nghệ, thực chiến điện thoại của AutoGLM giúp chúng ta nắm vững kỹ năng triển khai thực tế, còn giải pháp phía thiết bị đầu cuối (on-device) của GLM-ZERO thì cho thấy tương lai của bảo vệ quyền riêng tư và ứng dụng ngoại tuyến.

Công nghệ GUI Agent vẫn đang tiến hóa nhanh chóng. Hệ thống hiện tại tuy đã có thể xử lý phần lớn tác vụ hàng ngày, nhưng về độ chính xác, tốc độ suy luận và kiểm soát chi phí vẫn còn nhiều không gian để cải thiện. Khi mô hình lớn thị giác tiếp tục tiến bộ, cùng với sự phát triển của chip suy luận phía thiết bị đầu cuối, chúng ta có lý do để tin rằng GUI Agent sẽ trở thành mô hình quan trọng của tương tác người-máy trong tương lai.

Có lẽ trong tương lai không xa, mỗi người sẽ sở hữu một trợ lý số thực sự thông minh, nó không chỉ hiểu được ý định của bạn, mà còn có thể vượt qua các ứng dụng và nền tảng khác nhau, giúp bạn hoàn thành các công việc lặp đi lặp lại. Lúc đó, những script tự động hóa mà chúng ta hôm nay phải vất vả viết ra, đều sẽ trở thành một câu chỉ dẫn ngôn ngữ tự nhiên đơn giản.

Tương lai này, thực ra đã đang trên đường tới rồi.

---

## Tài liệu tham khảo

1. Paper Mobile-Agent-v3: https://arxiv.org/abs/2508.15144
2. Open-AutoGLM GitHub: https://github.com/zai-org/Open-AutoGLM
3. Dự án UI-TARS: https://github.com/bytedance/UI-TARS
</content>
