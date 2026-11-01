# Thực hành hậu huấn luyện (Post-training) Trợ lý Du lịch: Từ giao thức sản phẩm đến hoàn thiện với SFT, DPO và Rerank

Dự án này xuất phát từ repository `helloagents-trip-planner` do tôi duy trì. Nó không phải là một dự án nghiên cứu học thuật viết paper, cũng không nhằm mục đích leo bảng xếp hạng (benchmark). Tôi xem đây là một bài tập thực hành kỹ thuật: làm thế nào để một trợ lý du lịch thoạt nhìn có vẻ biết trò chuyện, từng bước cải tiến đến mức có thể kết nối với frontend/backend, có thể bị các luật quy tắc bắt lỗi, và vẫn có thể tiếp tục sửa chữa hoàn thiện.

Lập kế hoạch du lịch nhìn bề ngoài tưởng đơn giản, nhưng thực chất rất dễ gây hiểu lầm. Bản Demo đầu tiên thường trông khá đẹp mắt: người dùng nói "Tôi muốn đi Hàng Châu chơi 4 ngày, ngân sách 3500 tệ", mô hình nhanh chóng liệt kê ra các điểm tham quan, khách sạn, nhà hàng và lưu ý. Nhưng khi thực sự tích hợp vào frontend và backend, rắc rối mới ập đến: ngân sách là cho cả chuyến đi hay tính theo đầu người, khách sạn tính theo mấy đêm, vé vào cửa có cần nhân với số người không, nhà hàng có thực sự lấy từ danh sách ứng viên của công cụ (tool candidates) hay không, và ngày cuối cùng rốt cuộc có cần sắp xếp bữa tối hay không.

Chương Extra này chỉ viết phiên bản rút gọn: tuyến đường này đã được đi qua như thế nào, những thử nghiệm nào có hiệu quả, những thử nghiệm nào về sau chứng minh là không cần thiết. Các câu lệnh, cấu hình và hồ sơ lưu trữ đầy đủ được đặt trong repository của dự án.

Dự án và tài liệu đi kèm:

- Repository dự án trợ lý du lịch: [helloagents-trip-planner](https://github.com/nameless0120/helloagents-trip-planner)
- Hướng dẫn thực hành hậu huấn luyện đầy đủ: [Hướng dẫn thực hành hậu huấn luyện Trợ lý du lịch](https://github.com/nameless0120/helloagents-trip-planner/blob/main/training/docs/%E6%95%99%E7%A8%8B/%E6%97%85%E8%A1%8C%E5%8A%A9%E6%89%8B%E5%90%8E%E8%AE%AD%E7%BB%83%E5%AE%9E%E6%88%98%E6%95%99%E7%A8%8B.md)
- Dữ liệu đi kèm: `helloagents-后训练数据`, link Baidu Netdisk: <https://pan.baidu.com/s/5oNsK7pwQnqzQEUg5ykb09Q>

Chạy đến cuối cùng, tôi nhận thấy lộ trình thực ra rất mộc mạc: **Prompt cố định giao thức, SFT học cấu trúc, DPO học sở thích (preference), Rerank chọn câu trả lời ổn định hơn trong các ứng viên.**

![Sơ đồ tổng quan tuyến chính hậu huấn luyện](./images/Extra12-figures/01-后训练主线总览图.png)

---

## Mục lục

- [Chương 1: Xem trước một trường hợp so sánh trước và sau](#chương-1-xem-trước-một-trường-hợp-so-sánh-trước-và-sau)
- [Chương 2: Tại sao không thể vừa vào là huấn luyện ngay](#chương-2-tại-sao-không-thể-vừa-vào-là-huấn-luyện-ngay)
- [Chương 3: Sửa đổi giao thức sản phẩm trước](#chương-3-sửa-đổi-giao-thức-sản-phẩm-trước)
- [Chương 4: Đóng băng bộ đánh giá](#chương-4-đóng-băng-bộ-đánh-giá)
- [Chương 5: Tinh chỉnh Prompt là tìm kiếm ranh giới](#chương-5-tinh-chỉnh-prompt-là-tìm-kiếm-ranh-giới)
- [Chương 6: Tạo dữ liệu SFT và kiểm toán](#chương-6-tạo-dữ-liệu-sft-và-kiểm-toán)
- [Chương 7: Huấn luyện LoRA SFT nhiều giai đoạn](#chương-7-huấn-luyện-lora-sft-nhiều-giai-đoạn)
- [Bổ sung chương 7: Tại sao SFT toàn bộ tham số không trở thành tuyến chính](#bổ-sung-chương-7-tại-sao-sft-toàn-bộ-tham-số-không-trở-thành-tuyến-chính)
- [Chương 8: Best-of-N Replay và SFT Rerank](#chương-8-best-of-n-replay-và-sft-rerank)
- [Chương 9: DPO học sở thích, đổi chỉ số cốt lõi thành PlannerSoft](#chương-9-dpo-học-sở-thích-đổi-chỉ-số-cốt-lõi-thành-plannersoft)
- [Chương 10: Rerank đa ứng viên cuối cùng](#chương-10-rerank-đa-ứng-viên-cuối-cùng)
- [Chương 11: So sánh với tham chiếu bên ngoài MiMo như thế nào](#chương-11-so-sánh-với-tham-chiếu-bên-ngoài-mimo-như-thế-nào)
- [Bad Case Gallery: Ba lỗi giá trị nhất](#bad-case-gallery-ba-lỗi-giá-trị-nhất)
- [Chương 12: Kinh nghiệm đúc kết từ thử nghiệm lần này](#chương-12-kinh-nghiệm-đúc-kết-từ-thử-nghiệm-lần-này)
- [Tài nguyên tái hiện](#tài-nguyên-tái-hiện)

---

## Chương 1: Xem trước một trường hợp so sánh trước và sau

Tạm thời chưa bàn đến LoRA, chưa bàn đến DPO, cũng chưa bàn đến rerank. Hãy xem một yêu cầu thông thường:

> Một người đi Hàng Châu chơi 4 ngày, đi taxi, ở khách sạn bình dân (tiết kiệm), thích ẩm thực và các địa danh biểu tượng của thành phố, tổng ngân sách khoảng 3500 tệ, và không được vượt quá.

Mô hình nền không phải là hoàn toàn không biết viết. Nó có thể liệt kê Tây Hồ, chùa Linh Ẩn, Ban công Thành phố (City Balcony), cũng đưa ra khách sạn và nhà hàng. Nhưng nhìn kỹ sẽ thấy thiếu vững chắc: lịch trình khá rỗng, số ngày khách sạn không ổn định, nhà hàng bị lặp lại, và ngân sách cũng cách quá xa mục tiêu của người dùng. Trông nó giống một kế hoạch du lịch, nhưng thực sự giao cho người dùng thì khá bấp bênh.

Phiên bản sau khi hậu huấn luyện cũng không phải đạt điểm tuyệt đối, chẳng hạn ngân sách ăn uống vẫn còn sai số nhỏ 60 tệ. Nhưng ít nhất nó bắt đầu giống một lịch trình được sắp xếp nghiêm túc: số đêm khách sạn ổn định, mật độ điểm tham quan bình thường, nhà hàng không còn lặp lại suốt cả chuyến đi, và ngân sách cũng sát hơn với ràng buộc cứng (hard constraint) mà người dùng đưa ra.

| Điểm quan sát | Mô hình nền | Sau hậu huấn luyện |
| --- | --- | --- |
| Mật độ lịch trình | 4 ngày cơ bản mỗi ngày 1 điểm tham quan, khá trống trải | Mỗi ngày 2 đến 3 điểm tham quan, bao phủ Tây Hồ, Đoạn Kiều, tháp Lôi Phong, chùa Linh Ẩn, Tây Khê, phố Thanh Hà Phường, v.v. |
| Khách sạn | 3 ngày đầu có khách sạn, ngày thứ 4 ghi là "không có lưu trú", nhưng trong ngân sách lại tính thành 4 đêm | 3 đêm đầu sử dụng ổn định cùng một khách sạn bình dân, ngân sách tính theo 3 đêm |
| Ăn uống | KFC lặp lại khá nhiều, luân chuyển bữa trưa và bữa tối kém | Nhà hàng có sự luân chuyển, bao gồm món Hàng Châu, hải sản, đồ nướng, quán mì và một lượng nhỏ đồ ăn nhanh |
| Ngân sách | Báo 1840 tệ, quá xa so với hard budget 3500 tệ | Báo 2500 tệ, rơi vào cận dưới của khoảng chấp nhận được |
| Đánh giá theo quy tắc | Bắt ra 9 loại lỗi | Hiện tại chủ yếu còn sai số nhỏ trong tài khoản ăn uống |

![Hình so sánh Case mở đầu trước và sau](./images/Extra12-figures/02-开篇Case前后对比图.png)

Tôi muốn nói rõ ngay từ đầu: hậu huấn luyện không phải là để mô hình viết văn hay hơn, mà là để nó giống một Planner có thể đưa vào quy trình sản phẩm hơn. Số đêm lưu trú, grounding ăn uống, quan hệ ngân sách, ngày tháng thời tiết, JSON đầu ra — những thứ này trông vụn vặt, nhưng trong sản phẩm thực tế, chúng lại là những thứ dễ lật thuyền nhất.

---

## Chương 2: Tại sao không thể vừa vào là huấn luyện ngay

Ban đầu tôi cũng rất muốn nhảy vào huấn luyện ngay. Chạy LoRA mang lại cảm giác tiến độ rõ rệt nhất: chuẩn bị dữ liệu xong, kích hoạt script, loss bắt đầu giảm xuống, trông như dự án đang tiến về phía trước.

Sau này tôi mới nhận ra thứ tự này đã bị ngược.

Nếu các sự thật nghiệp vụ (business facts) chưa được cố định, việc huấn luyện chỉ làm cho sự hỗn loạn được học một cách ổn định hơn mà thôi. Người dùng nói "ngân sách 3000", mô hình cần biết đây là ngân sách cho cả chuyến hay ngân sách bình quân đầu người; giá khách sạn là cho mỗi phòng mỗi đêm, chứ không phải tổng giá cả chuyến; vé tham quan phải nhân với số người đi cùng; nhà hàng không thể bịa ra bừa bãi, tốt nhất là đến từ danh sách ứng viên của công cụ. Chỉ dựa vào prompt nhắc đi nhắc lại liên tục thì có thể cứu được một phần, nhưng không thể cứu được toàn bộ chuỗi mắt xích.

Vì vậy, sau này tôi không đi theo con đường này:

```text
Viết prompt -> Tạo dữ liệu -> Huấn luyện -> Xem chỉ số
```

Mà là:

```text
Cải tạo giao thức frontend/backend
  -> Đóng băng bộ đánh giá standard / hard
  -> Tinh chỉnh prompt và phác họa chân dung lỗi (failure profiling)
  -> Dùng mô hình mạnh tạo dữ liệu SFT
  -> Kiểm toán dữ liệu và export qua LLaMA-Factory
  -> Huấn luyện LoRA SFT nhiều giai đoạn
  -> Best-of-N Replay
  -> Huấn luyện sở thích DPO
  -> Đánh giá theo quy tắc, so sánh lát cắt (slice comparison) và chọn checkpoint
  -> Thu hoạch hoàn thiện với Rerank đa ứng viên
```

Tuy chậm nhưng lợi ích rất thực tế: mỗi vòng đều biết rõ mình đang thay đổi cái gì, và biết chỗ nào lại vừa bị sửa hỏng.

---

## Chương 3: Sửa đổi giao thức sản phẩm trước

Khi mới bắt đầu làm trợ lý du lịch, tôi cũng rất dễ ném mọi vấn đề cho mô hình: để nó tự đoán số người từ ngôn ngữ tự nhiên, tự đoán cách tính ngân sách, tự đoán số đêm lưu trú, rồi đoán tiếp giá vé và giá nhà hàng. Phiên bản đầu tiên có thể chạy được, nhưng việc huấn luyện phía sau sẽ vô cùng đau đớn, bởi vì "sự thật" trong dữ liệu huấn luyện vốn dĩ đã trôi nổi bấp bênh rồi.

Sau đó, quyết định đầu tiên tôi đưa ra rất mộc mạc: **Đừng để mô hình đoán sự thật nghiệp vụ.**

Frontend không còn chỉ gửi một đoạn văn bản tự do nữa, mà gửi một cách tường minh:

- `party`: người lớn, trẻ em, người già, tổng số người, loại hình chuyến đi;
- `budget_constraint`: số tiền, loại tiền tệ, phạm vi ngân sách, bậc ngân sách (tier), mức độ ràng buộc;
- Các trường có cấu trúc như `travel_days`, phương thức di chuyển, sở thích lưu trú, sở thích hứng thú, v.v.

Backend cũng không còn nhồi nhét tất cả kết quả công cụ vào prompt một cách vô tội vạ nữa, mà trước tiên biên dịch thành `PlannerContext`. Ngữ cảnh này sẽ nói rõ cho mô hình biết: lần này là mấy người, ngân sách là cho cả chuyến hay bình quân đầu người, khách sạn tính theo mấy đêm, mỗi ứng viên điểm tham quan, nhà hàng, khách sạn đến từ đâu, gợi ý giá (price hint) và chiến lược ngân sách là gì, và đầu ra cuối cùng bắt buộc phải đáp ứng JSON shape nào.

![Sơ đồ cải tạo giao thức sản phẩm](./images/Extra12-figures/03-产品协议改造图.png)

Nếu muốn xem theo mã nguồn, bạn có thể lần theo manh mối này: kiểu form frontend, request schema của backend, biên dịch PlannerContext, parse và validation đầu ra. Đường dẫn tệp cụ thể được đặt trong hướng dẫn đầy đủ của `helloagents-trip-planner`, ở đây không đi sâu chi tiết.

Có tầng giao thức này rồi, nhiệm vụ của hậu huấn luyện mới được thu hẹp lại: mô hình không còn viết kế hoạch du lịch theo cảm tính nữa, mà đưa ra lựa chọn trong số các ứng viên có cấu trúc, và xuất ra JSON hợp lệ.

---

## Chương 4: Đóng băng bộ đánh giá

Kết quả huấn luyện xem không hiểu, nhiều khi không phải là vấn đề của mô hình, mà là do đề thi liên tục thay đổi.

Trợ lý du lịch đặc biệt dễ gặp phải tình trạng này: hôm nay ứng viên bản đồ thay đổi, ngày mai thời tiết thay đổi, ngày kia logic tạo ngân sách lại thay đổi. Cuối cùng bạn không thể phân biệt được là do mô hình mạnh lên, hay là do đề thi trở nên dễ hơn.

Vì vậy, trước tiên tôi cố định bộ đánh giá lại.

![Sơ đồ đóng băng bộ đánh giá](./images/Extra12-figures/04-评测集冻结图.png)

Tôi chia đánh giá thành hai loại:

| Bộ đánh giá | Tác dụng |
| --- | --- |
| `standard eval` | Xem độ ổn định dưới các yêu cầu thông thường, chẳng hạn như thành phố phổ thông, ngân sách phổ thông, sở thích thông thường |
| `hard eval` | Chủ động phóng đại các điểm khó, chẳng hạn như nhiều người, có người già trẻ nhỏ, ngân sách khắt khe, sở thích tiêu cực (negative preference), chế độ ăn uống đặc biệt |

Ở đây còn có một chi tiết: về sau chiến lược truy xuất và việc sửa chữa ngữ cảnh thay đổi, quả thực cần xây dựng lại ngữ cảnh đánh giá, nhưng không nên lấy mẫu lại yêu cầu của người dùng. Cách làm của tôi là giữ nguyên request signature, chỉ xây dựng lại các ứng viên công cụ và ngữ cảnh. Như vậy vừa giữ được tính có thể so sánh, vừa sửa được dữ liệu rác trong ngữ cảnh cũ.

Bộ frozen eval này về sau luôn được dùng để lựa chọn mô hình. Dưới góc nhìn nghiêm ngặt của các bài báo học thuật, nó giống validation set hơn là blind test. Cho nên tôi chỉ gọi nó là "đánh giá từng giai đoạn trên bộ đánh giá cố định", chứ không tô vẽ thành một bài test mù độc lập.

Cuối cùng, khi làm dữ liệu kết thúc DPO, tôi đã kiểm tra riêng độ trùng lặp chữ ký: `selected_eval_signature_overlap = 0`. Nghĩa là prompt đánh giá không hề lọt vào dữ liệu huấn luyện.

---

## Chương 5: Tinh chỉnh Prompt là tìm kiếm ranh giới

Sau khi giao thức frontend/backend và bộ đánh giá đã ổn định, mới bước vào giai đoạn tinh chỉnh prompt.

Ở đây tôi không phải đang tìm kiếm một "prompt thần thánh". Điều tôi quan tâm hơn là: những vấn đề nào prompt có thể cứu vãn được, những vấn đề nào bắt buộc phải giao cho dữ liệu, quy tắc và kỹ thuật công trình (engineering).

Vài vòng prompt đầu tiên đại khái giải quyết ba loại vấn đề:

| Vòng | Mục tiêu chính | Kết luận |
| --- | --- | --- |
| Giao thức đầu ra | Ngày tháng không được lộn xộn, bữa ăn không được thiếu, trường khách sạn không được trôi dạt, JSON không được đứt đoạn giữa chừng | Prompt có thể nâng cao rõ rệt độ ổn định của shape, nhưng vẫn cần phối hợp với parser và validation |
| Grounding ăn uống | Nhà hàng bắt buộc phải đến từ danh sách ứng viên, không viết "đồ ăn vặt gần đó", "nhà hàng đặc sản địa phương" | Chỉ viết trong prompt là chưa đủ, bộ đánh giá cũng bắt buộc phải bắt được đầu ra không được grounded |
| Lộ trình chính xác giả tạo | Không viết những thứ công cụ chưa từng cung cấp như "đi bộ 10 phút", "đi taxi 15 phút" | Loại ảo giác (hallucination) này phù hợp hơn để chặn lại trong các quy tắc đầu ra |

Bước này điều thực sự hữu ích không phải là bản thân prompt, mà là phác họa chân dung lỗi (failure profiling). Xem bad case nhiều rồi, các vấn đề sẽ tự động phân tầng: schema, ngày tháng, thiếu bữa ăn phù hợp cho shape validation; nhà hàng và điểm tham quan không grounded cần dựa vào cả prompt và quy tắc để cùng siết lại; quan hệ ngân sách quá phức tạp, tốt nhất tách thành hai phần: tính toán lại bằng mã nguồn và mô hình lựa chọn; mức độ đáp ứng sở thích chưa đủ thì đi bổ sung dữ liệu.

Prompt chỉnh đến cuối cùng, điều quan trọng nhất không phải là viết dài thêm một chút nữa, mà là biết nên dừng lại ở đâu.

---

## Chương 6: Tạo dữ liệu SFT và kiểm toán

Khâu tạo dữ liệu SFT là khâu dễ làm người ta lơ là cảnh giác nhất.

Mô hình mạnh quả thực có thể tạo ra kế hoạch du lịch trông rất ra ngô ra khoai, nhưng "trông được" không đồng nghĩa với "có thể huấn luyện". Nếu trong đầu ra của mô hình giáo viên (teacher), cách tính ngân sách bị sai, nhà hàng không grounded, khách sạn mỗi ngày đổi lung tung, thì mô hình học sinh sẽ học những lỗi đó một cách ổn định hơn, và cũng sẽ sai một cách ổn định hơn.

Vì vậy tôi chia nhỏ quy trình tạo dữ liệu ra làm:

1. Chạy dry-run phân bố yêu cầu trước, xem thành phố, số ngày, ngân sách, loại bạn đồng hành có hợp lý không.
2. Chạy dry-run `PlannerContext` tiếp theo, xác nhận ứng viên công cụ, gợi ý giá, thời tiết và chiến lược ngân sách đều có thể biên dịch ra được.
3. Tạo theo lô nhỏ, chạy trước 20 đến 100 mẫu, đừng vừa vào đã tạo ngay hàng nghìn mẫu.
4. Mỗi lần gọi mô hình mạnh đều ghi lại usage, manifest và cấu hình run.
5. Tạo xong phải kiểm toán trước rồi mới đưa vào tập huấn luyện.

![Phễu tạo dữ liệu và kiểm toán SFT](./images/Extra12-figures/05-SFT数据生成与审计漏斗图.png)

Khi kiểm toán, trước tiên xem xét các bộ lọc cứng (hard filter):

| Mục lọc | Tại sao quan trọng |
| --- | --- |
| JSON / schema hợp lệ | Backend bắt buộc phải parse được |
| Ngày tháng và số ngày nhất quán | Kế hoạch du lịch không được thiếu ngày, thừa ngày, sai ngày |
| Khách sạn và nhà hàng grounded | Không thể tự tiện bịa ra ứng viên |
| Ăn uống không lặp lại | Không thể ăn liên tiếp mấy bữa cùng một quán ăn nhanh |
| Ràng buộc cứng về ngân sách (hard constraint) | Ngân sách cứng không được vượt quá |
| Mối quan hệ ngân sách hợp lý | Số đêm khách sạn, số người tính vé, mức chi tiêu ăn uống phải đúng |

Còn một điểm rất dễ bị bỏ qua: đừng tiếc dữ liệu cũ. Đầu dự án có một lô dữ liệu SFT cũ, nhìn cục bộ thì khá sạch, nhưng lại bắt nguồn từ cách tính ngân sách cũ. Về sau tôi chọn lưu trữ (archive) toàn bộ, không chắp vá để tiếp tục dùng nữa. Quyết định này lúc đó hơi xót, nhưng nhìn lại thì thấy hoàn toàn đúng đắn. Hậu huấn luyện sợ nhất là "giao thức mới + dữ liệu theo cách tính cũ" trộn lẫn vào nhau, bề ngoài mô hình học thêm được nhiều mẫu, nhưng thực tế lại học phải các quy tắc mâu thuẫn lẫn nhau.

---

## Chương 7: Huấn luyện LoRA SFT nhiều giai đoạn

Sau khi có dữ liệu, mới thực sự bước vào LoRA SFT.

Tuyến này sử dụng Qwen2.5-7B-Instruct để làm LoRA. Việc huấn luyện không hoàn thành trong một vòng duy nhất, mà được đẩy mạnh qua nhiều giai đoạn. Nguyên tắc của tôi là: **Cố gắng hạn chế tối đa việc thay đổi nhiều biến số cùng một lúc.**

Các thiết lập nền tảng cơ bản giữ ổn định: LoRA r32, ngữ cảnh dài (long context), bf16, tích lũy gradient (gradient accumulation). Điều quan trọng nhất ở đây không phải là học thuộc lòng bảng tham số, mà là hiểu tại sao mỗi vòng về sau đều chỉ đổi một biến số chính. Trong `PlannerContext` có điểm tham quan, khách sạn, nhà hàng, thời tiết và chiến lược ngân sách, nếu nén ngắn ngữ cảnh lại sẽ trực tiếp cắt mất tín hiệu; rank quá nhỏ thì giao thức JSON dài và việc lựa chọn ứng viên cũng học không vững.

Thứ thực sự được điều chỉnh lặp đi lặp lại là ba loại: dữ liệu, tốc độ học (learning rate), và số epoch huấn luyện.

| Giai đoạn | Điểm bắt đầu | Dữ liệu | Tham số chính | Muốn giải quyết điều gì |
| --- | --- | --- | --- | --- |
| SFT Clean trục chính (main clean) | Qwen2.5-7B-Instruct | `main_clean` | `lr=8e-5 / 6e-5`, `epoch=4` | Học vững giao thức TripPlan trước |
| Bổ sung huấn luyện trộn ngân sách thực tế (usage700) | Huấn luyện tiếp từ adapter `lr6e-5` | main clean + realbudget usage700 | `lr=2e-5`, `epoch=1` | Bổ sung việc sử dụng ngân sách và cách tính ngân sách thực tế |
| Bổ sung huấn luyện chẩn đoán tận dụng ngân sách (patch700) | Huấn luyện tiếp từ adapter `lr6e-5` | budget utilization patch 700 | `lr=1e-5`, `epoch=2` | Xem việc bổ sung dữ liệu kiểu tận dụng ngân sách có thể đẩy xa đến đâu |
| SFT Rerank phát lại 600 (Best-of-N 600) | Huấn luyện tiếp từ adapter usage700 | old replay + Best-of-N winner | `lr=1e-5`, lưu mỗi nửa epoch | Bơm các ứng viên tốt hơn được sàng lọc bằng quy tắc |
| SFT Rerank phát lại 1200 (Best-of-N 1200) | Huấn luyện tiếp từ Best-of-N 600 final | old replay + nhiều Best-of-N winner hơn | `lr=1e-5`, lưu mỗi nửa epoch | Tăng tỷ lệ winner, xem có tiếp tục nâng cao được không |

![Sơ đồ dòng thời gian huấn luyện LoRA nhiều giai đoạn](./images/Extra12-figures/06-LoRA多阶段训练时间线图.png)

Ở đây có một cái hố tôi từng dẫm phải: `adapter_name_or_path` không phải là `resume_from_checkpoint`. Nó chỉ lấy LoRA adapter đã export từ vòng trước để làm warm-start, trạng thái bộ tối ưu (optimizer state) sẽ không chạy tiếp từ vòng trước. Nghĩa là mỗi giai đoạn đều sẽ dùng lại learning rate và scheduler mới trong cấu hình hiện tại.

Điều này ngược lại lại rất phù hợp cho các thử nghiệm theo giai đoạn. Năng lực học được ở vòng trước được giữ lại trong adapter, vòng tiếp theo dùng learning rate nhỏ hơn để tiếp tục sửa chữa các vấn đề cục bộ.

Learning rate giảm dần xuyên suốt lộ trình cũng là vì lý do này:

```text
main clean:       6e-5 / 8e-5
usage700 mixed:   2e-5
patch700 only:    1e-5
Best-of-N replay: 1e-5
DPO closing:      cấp 1e-6 đến 1.5e-6
```

Càng về sau, dữ liệu càng giống như đang vá lỗi cục bộ. Nếu learning rate quá cao, chỉ số ngân sách có thể tăng lên, nhưng grounding ăn uống, tính liên tục của lưu trú hoặc ngày tháng thời tiết lại tụt xuống.

### Bổ sung chương 7: Tại sao SFT toàn bộ tham số không trở thành tuyến chính

Sau khi viết xong tuyến chính LoRA, tôi lại chạy bổ sung một lần SFT toàn bộ tham số (Full-parameter SFT). Không phải muốn lật đổ lộ trình phía trước, chỉ là muốn xem giới hạn trên: nếu không chỉ huấn luyện adapter mà tác động vào toàn bộ mô hình, thì Planner du lịch liệu có tăng thêm một bậc nữa không?

Lần này sử dụng `Qwen2.5-7B-Instruct` và dữ liệu clean SFT phiên bản đầu tiên, duy trì ngữ cảnh dài, chạy 6 epoch. Huấn luyện thì chạy thông suốt được, nhưng chi phí lập tức tăng vọt: một mẻ trên 6 card 40GB mất khoảng 7 tiếng, mô hình lưu ra nặng 28GB. So với một LoRA adapter, đây không còn là cảm giác lặp lại (iteration) trên cùng một cấp độ nữa.

Kết quả khá thú vị. Full-param không phải là vô dụng, nó thực sự đã đẩy planner soft lên cao hơn:

| Chỉ số | Full Instruct toàn tham số | LoRA cùng phiên bản | Độ chênh lệch |
| --- | ---: | ---: | ---: |
| Vượt qua ràng buộc cứng (Hardpass) | 94.4% | 95.8% | -1.4pp |
| Planner Soft | 48.0% | 44.7% | +3.3pp |
| Tính lại ngân sách Soft | 33.8% | 30.7% | +3.1pp |
| Đa dạng ăn uống | 82.6% | 76.4% | +6.2pp |
| Khớp sở thích ngân sách | 67.8% | 66.9% | +0.9pp |
| Nhất quán tổng ngân sách | 68.0% | 72.3% | -4.3pp |
| Ràng buộc ngân sách người dùng | 88.8% | 91.6% | -2.8pp |

![Sơ đồ so sánh giữa SFT toàn bộ tham số và LoRA](./images/Extra12-figures/18-全参SFT与LoRA对比图.png)

Trên tập standard, Planner Soft của bản toàn tham số tăng từ 48.0% lên 57.5%, mức cải thiện này rất rõ rệt. Nó sẵn sàng viết lịch trình phong phú hơn, ăn uống cũng không dễ bị lặp lại như trước. Nói cách khác, cập nhật toàn tham số thực sự đã tác động đến "thói quen lập kế hoạch" của mô hình, chứ không chỉ học định dạng trên bề mặt.

Nhưng cuối cùng tôi vẫn không đưa full-param vào tuyến chính. Lý do rất đơn giản: điều khó nhất của dự án này không phải là để mô hình viết nhiều thêm một chút, mà là để nó giữa một đống ràng buộc cứng đừng tính toán sai. Tổng ngân sách, ngân sách người dùng, số đêm khách sạn, vé tham quan nhân theo số người — những thứ này giống bài toán về quy tắc kỹ thuật và phân bố dữ liệu hơn. Toàn tham số sẽ khiến mô hình linh hoạt hơn, nhưng linh hoạt không đồng nghĩa với ổn định hơn.

Đối với dự án này, cuối cùng tôi vẫn sẵn sàng dùng LoRA hơn:

- Lượng dữ liệu ở mức hàng nghìn mẫu, không phải hàng trăm nghìn dữ liệu chỉ thị. Dung lượng của toàn tham số quá lớn, dễ dàng học luôn cả phong cách cục bộ vào.
- Trọng tâm nhiệm vụ là sao chép ngữ cảnh dài, xuất schema, grounding ứng viên và quy tắc ngân sách. LoRA đã có thể ép những thứ này lên mức rất cao.
- Chi phí lặp lại chênh lệch quá xa. LoRA huấn luyện, lưu trữ, rollback, đánh giá đều nhẹ nhàng; toàn tham số mỗi lần thử đều phải nghiêm túc xếp lịch card đồ họa và dọn dẹp dung lượng đĩa.
- Những điểm thực sự có thể tăng điểm về sau phần lớn nằm ở việc làm sạch dữ liệu, đào bới bad case, cặp DPO pair và rerank. Toàn tham số sẽ kéo chậm nhịp độ này lại.

Vì vậy, vị trí của thử nghiệm toàn tham số lần này trong hướng dẫn rất rõ ràng: **Nó chứng minh Planner Soft vẫn còn dư địa đi lên; đồng thời cũng chứng minh dự án này không thể dùng sức mạnh cơ bắp của toàn tham số để đẩy tới.**

Toàn tham số có thể thử nghiệm cho vui, đặc biệt là khi muốn làm một kết quả offline thật đẹp mắt. Nhưng dự án này đòi hỏi phải liên tục bổ sung dữ liệu, chạy đánh giá, rollback so sánh, tôi vẫn sẽ chọn LoRA + DPO + Rerank.

---

## Chương 8: Best-of-N Replay và SFT Rerank

Sau khi SFT đã học vững giao thức, tôi làm Best-of-N replay trước, rồi mới làm rerank ở giai đoạn hiển thị cuối cùng. Tên gọi nghe hơi giống nhau, nhưng chúng không phải là một thứ.

![Sơ đồ Best-of-N Replay và Rerank](./images/Extra12-figures/07-Best-of-N-Replay与Rerank图.png)

Best-of-N Replay là quy trình kiến tạo dữ liệu huấn luyện: với cùng một `PlannerContext`, cho mô hình hiện tại lấy mẫu (sample) nhiều câu trả lời, dùng bộ đánh giá theo quy tắc để chọn ra một câu tốt hơn, sau đó export câu trả lời chiến thắng (winner) thành dữ liệu SFT cho vòng tiếp theo.

```text
PlannerContext
  -> Lấy mẫu đa nhiệt độ (multi-temperature) t=0.2 / 0.5 / 0.8
  -> Mỗi ứng viên chạy rule metrics
  -> Ưu tiên chọn ứng viên đạt hardpass
  -> Xem tiếp các phần thưởng mềm (soft rewards) như ngân sách, mức chi tiêu ăn uống, tính đa dạng
  -> Winner đi vào vòng SFT tiếp theo
```

Rerank cuối cùng là quy trình ở thời điểm suy luận (inference time): cùng một prompt tạo ra nhiều ứng viên, không còn ghi winner trở lại tập huấn luyện nữa, mà chọn một câu trả lời ổn định hơn từ nhóm ứng viên trên môi trường online để trả về cho người dùng.

Cả hai quy trình này cuối cùng đều phải quay về frozen eval để quan sát. Việc quy tắc chọn winner chắc chắn có thiên kiến (bias), nếu reward quá thiên về một chỉ số nào đó, mô hình có thể trở nên bảo thủ, hoặc có thể hy sinh trải nghiệm. Nếu chỉ nhìn vào điểm số của một winner đơn lẻ thì rất dễ mừng hụt quá sớm.

Sau khi giai đoạn SFT kết nối các ứng viên đa nhiệt độ + quy tắc rerank, vài phiên bản nhìn chung đã bước lên một nấc thang mới:

| Phiên bản | hardpass | softpass | softpass tính lại ngân sách | số học ngân sách | sở thích ngân sách | quan hệ ngân sách | mức chi ăn uống |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Checkpoint giữa kỳ SFT + rerank (ckpt104) | 98.0 | 65.6 | 54.6 | 81.2 | 77.0 | 86.4 | 88.8 |
| Best-of-N 1200 phát lại + rerank (final1200) | 98.2 | 66.8 | 54.6 | 78.0 | 78.4 | 85.0 | 88.0 |
| Best-of-N 600 phát lại bản cuối + rerank (old600final) | 98.2 | 66.2 | 59.2 | 78.4 | 75.4 | 87.0 | 89.4 |

![Sơ đồ so sánh SFT Rerank](./images/Extra12-figures/09-sft-rerank-comparison.png)

Đến đây, giai đoạn SFT có thể khép lại. Việc tiếp tục bổ sung SFT thì lợi ích mang lại đã trở nên bão hòa, các mức tăng chính về sau nên đến từ dữ liệu sở thích (preference data) và việc lựa chọn ứng viên.

---

## Chương 9: DPO học sở thích, đổi chỉ số cốt lõi thành PlannerSoft

SFT đã có thể viết vững cái khung của TripPlan, nhưng giữa các câu trả lời hợp lệ cũng có câu hay câu dở. Hai kế hoạch đều qua được schema, đều tìm được khách sạn và nhà hàng, nhưng một cái có thể rất tiết kiệm nhưng lại không giống chuyến đi người dùng mong muốn, cái kia ngân sách sát hơn, ăn uống ít lặp lại hơn, các điểm tham quan cũng thuận đường hơn.

Sự đánh đổi này rất khó học vững chỉ dựa vào từng mẫu giáo viên đơn lẻ, DPO thì thuận tay hơn nhiều.

Tôi không coi DPO là liều thuốc tăng lực vạn năng. Ở đây nó chỉ làm duy nhất một việc: **Trong số các ứng viên đã vượt qua hardpass, học xem cái nào giống một lịch trình tốt hơn.**

### Cặp DPO pair trước tiên phải vượt qua ngưỡng cứng

`chosen` / `rejected` của DPO pair không thể làm bừa. Một JSON hỏng đối đầu với một JSON tốt, loại pair này dĩ nhiên có tín hiệu cho mô hình, nhưng thứ nó học được là định dạng chứ không phải sở thích. Trong dự án này, loại pair hữu ích hơn là như sau:

```text
Cùng một PlannerContext
  -> chosen: schema qua, hardpass qua, planner soft qua
  -> rejected: schema qua, hardpass qua, nhưng ngân sách/lặp lại/sở thích không qua
```

Mô hình được huấn luyện theo cách này mới thực sự học cách lựa chọn giữa các kế hoạch hợp lệ, chứ không phải học lại cách viết JSON từ đầu.

Còn một ranh giới đỏ: không được đào pair huấn luyện từ bộ đánh giá đã đóng băng. Trong dữ liệu kết thúc ngân sách, đã thực hiện lọc chữ ký chuyên biệt:

```text
frozen eval signature count = 497
selected eval signature overlap = 0
```

Bước này rất phiền phức, nhưng không thể bỏ qua. Nếu không, điểm số nhìn thì đẹp, nhưng thực chất là đang học vẹt thuộc lòng đề bài.

![Sơ đồ sàng lọc mẫu DPO và chống rò rỉ](./images/Extra12-figures/12-DPO样本筛选与防泄漏图.png)

### Chuyển chỉ số chính sang PlannerSoft

Về sau tôi càng lúc càng cảm thấy softpass thông thường vẫn chưa đủ. Đầu ra của trợ lý du lịch không phải là một câu hỏi trắc nghiệm, mà là một bản kế hoạch người dùng có thể thực sự cầm đi sử dụng. Do đó chỉ số chính dần dần chuyển sang `planner soft`: độ bám sát ngân sách, lặp lại ăn uống, lặp lại điểm tham quan, quan hệ ngân sách — tất cả đều phải xem xét.

![Sơ đồ phân rã chỉ số PlannerSoft](./images/Extra12-figures/16-PlannerSoft指标分解图.png)

Lộ trình qua các vòng DPO đại khái là:

| Giai đoạn | Mục đích | Kết luận |
| --- | --- | --- |
| Chạy thử DPO sở thích độ tin cậy cao | Xác minh trước xem DPO ngữ cảnh dài có chạy thông suốt không | Quy trình chạy thông suốt, sau đó bắt đầu đổi chỉ số |
| DPO quy tắc PlannerSoft | Chuyển mục tiêu tối ưu từ hardpass sang planner soft | Chọn ra điểm bắt đầu DPO đầu tiên có thể tiếp tục mở rộng dữ liệu (ckpt25) |
| Mở rộng dữ liệu PlannerSoft + Neo giữ Direct (Direct anchor) | Mở rộng dữ liệu planner soft, đồng thời giữ lại direct preference | Thu được đường cơ sở (baseline) mở rộng dữ liệu ổn định hơn (ckpt126) |
| PlannerSoft Clean cải thiện đơn sinh (single generation) | Dùng dữ liệu clean quy mô lớn hơn để tiếp tục huấn luyện | Phiên bản đơn sinh tốt nhất (260519 ckpt138) |
| DPO kết thúc ngân sách (Budget closing DPO) | Xây dựng clean pair nhắm vào việc ngân sách quá bảo thủ, vượt chi, lặp lại | Đơn sinh không tiếp tục tăng, nhưng nhóm ứng viên phù hợp hơn cho rerank (260520 / 260521) |

DPO loss cũng đừng so sánh cứng nhắc giữa các batch khác nhau. Các vòng pair đầu tiên rất dễ phân biệt, loss thấp, accuracy cao; các pair kết thúc ngân sách thì gần nhau hơn, cả chosen và rejected đều là kế hoạch đạt hardpass, chỉ khác biệt ở mức độ sử dụng ngân sách, lặp lại và sở thích, loss cao hơn một chút ngược lại là bình thường.

![Sơ đồ DPO loss không thể so sánh chéo batch](./images/Extra12-figures/15-DPO-loss跨批次不可比图.png)

Về sau tôi coi trọng hai tín hiệu hơn: reward accuracy có ổn định đi lên hay không, và trên frozen eval thì planner soft cùng các chỉ số liên quan đến ngân sách có thực sự nhúc nhích hay không. Nhật ký huấn luyện cho bạn biết mẻ này có bị hỏng không, còn đánh giá mới nói cho bạn biết mẻ này có tác dụng hay không.

---

## Chương 10: Rerank đa ứng viên cuối cùng

Nửa sau của DPO là giai đoạn dễ bị đọc hiểu sai nhất. Bản đơn sinh ổn định nhất là bản PlannerSoft Clean, tức 260519 ckpt138; hai vòng kết thúc ngân sách sau đó (260520 ckpt66, 260521 ckpt64) không tiếp tục đẩy điểm đơn sinh lên cao hơn.

Nhưng hiển thị cuối cùng không phải là đơn sinh, mà là rerank đa ứng viên. Giá trị của việc kết thúc ngân sách chủ yếu thể hiện ở nhóm ứng viên (candidate pool): nó không nhất thiết làm cho câu trả lời phát đầu tiên có điểm cao hơn, nhưng lại dễ lấy mẫu ra được câu trả lời tốt có thể được quy tắc lựa chọn hơn.

![Sơ đồ so sánh đơn sinh và Rerank đa ứng viên](./images/Extra12-figures/17-单生成与多候选Rerank对比图.png)

Vài nhóm chỉ số cuối cùng đại khái là:

| Phiên bản | hardpass | planner soft | soft tính lại ngân sách |
| --- | ---: | ---: | ---: |
| Baseline mở rộng dữ liệu (ckpt126) | 98.4% | 66.9% | 48.5% |
| PlannerSoft Clean đơn sinh (260519 ckpt138) | 98.4% | 71.5% | 50.9% |
| Vòng 1 kết thúc ngân sách đơn sinh (260520 ckpt66) | 99.0% | 70.1% | 48.3% |
| Vòng 2 kết thúc ngân sách đơn sinh (260521 ckpt64) | 98.2% | 69.7% | 47.6% |
| Vòng 2 kết thúc ngân sách + 4 ứng viên rerank (260521 ckpt64) | **99.4%** | **80.6%** | **68.2%** |

![Sơ đồ hiệu quả Rerank kết thúc DPO](./images/Extra12-figures/10-dpo-rerank-closing.png)

Cho nên không thể nói đơn giản là "mẻ cuối cùng đơn sinh là tốt nhất". Tôi sẽ ghi nhớ như sau:

- Đơn sinh tốt nhất: Bản PlannerSoft Clean đơn sinh (260519 ckpt138).
- Rerank đa sinh tốt nhất: Vòng 2 kết thúc ngân sách + 4 ứng viên rerank (260521 ckpt64).
- Bản chủ lực hiển thị: 260521 ckpt64 rerank n4, 500 mẫu planner soft đạt `80.6%`, hard split planner soft đạt `77.0%`.

Đơn sinh nhìn vào chất lượng trung bình của một lần lấy mẫu; rerank nhìn vào việc trong nhóm ứng viên có câu trả lời tốt hơn hay không, và quy tắc có thể chọn nó ra được hay không. Cả hai có thể không cùng là một checkpoint.

---

## Chương 11: So sánh với tham chiếu bên ngoài MiMo như thế nào

Cuối cùng có thể thêm một mô hình mạnh bên ngoài làm tham chiếu, nhưng phần này nhất định phải nêu rõ quy chuẩn (khẩu kính). MiMo không phải là một checkpoint trong tuyến huấn luyện LoRA của chúng ta, cũng không phải là bảng xếp hạng (leaderboard) dưới cùng một bộ script, cùng một phiên bản quy tắc nghiêm ngặt. Cách sử dụng phù hợp hơn là: xem nó mách nước cho chúng ta biết mô hình mạnh đại khái sẽ mạnh ở đâu, và chỗ nào chưa hoàn toàn khớp với quy tắc cục bộ của chúng ta.

Tôi cũng từng lấy API bên ngoài MiMo v2.5 Pro để làm đánh giá tham chiếu. Để tránh giới hạn độ dài đầu ra ảnh hưởng đến kết luận, lúc đó đã mở rộng max token lên một chút. Khi đặt cạnh bản cục bộ cuối cùng, đại khái trông như thế này:

| Mô hình | hardpass | planner soft | soft tính lại ngân sách | sở thích ngân sách | bám sát ngân sách tính lại |
| --- | ---: | ---: | ---: | ---: | ---: |
| MiMo v2.5 Pro (mở rộng max token) | 98.8% | 78.7% | **76.6%** | 85.5% | **82.4%** |
| Bản cục bộ cuối cùng (260521 ckpt64 rerank n4) | **99.4%** | **80.6%** | 68.2% | **86.0%** | 73.4% |

![Sơ đồ so sánh tham chiếu bên ngoài MiMo](./images/Extra12-figures/11-mimo-reference-comparison.png)

Bảng này tôi sẽ nhìn nhận như sau:

- Bản cục bộ cuối cùng dưới quy chuẩn quy tắc của dự án này, `hardpass` và `planner soft` đã đuổi kịp và nhỉnh hơn một chút so với tham chiếu MiMo.
- Soft tính lại ngân sách và mức bám sát ngân sách tính lại của MiMo vẫn mạnh hơn, cho thấy việc kiểm soát tổng mức ngân sách nó làm vững chắc hơn.
- Quan hệ ngân sách và mức chi tiêu ăn uống của MiMo trong các báo cáo ban đầu không quá cao, chủ yếu là do nó đưa ra tiền ăn bình quân đầu người thực tế hơn, nhưng những khoản tiền ăn này đôi khi lại thấp hơn cận dưới của bậc quy tắc hiện tại của chúng ta.

Cho nên tôi sẽ không viết thành "vượt trội toàn diện so với MiMo". Cách nói thận trọng hơn là: dưới bộ đánh giá đóng băng và quy chuẩn quy tắc của dự án này, planner soft của mô hình cục bộ cuối cùng đã bắt kịp tham chiếu mô hình mạnh; việc bám sát ngân sách vẫn còn khoảng cách, sau này nếu thực sự muốn làm tiếp, sẽ bổ sung việc kiểm soát tổng mức ngân sách và điều phối giữa các bậc ngân sách.

---

## Bad Case Gallery: Ba lỗi giá trị nhất

Thứ hữu ích nhất trong hậu huấn luyện thường không phải là những ví dụ thành công đẹp đẽ nhất, mà là những bad case liên tục "vả vào mặt" người làm. Cuối cùng tôi giữ lại ba loại: tổng ngân sách sai, ăn uống lặp lại hoặc không grounded, và số đêm khách sạn cùng số phòng tính không đúng.

### 1. Tổng ngân sách sai: Trông như tài khoản nhỏ, nhưng thực chất sẽ đánh thủng trực tiếp hard budget

Ví dụ đầu tiên là Tế Nam 3 ngày, 1 người đi công tác kết hợp du lịch, hard budget 3200 tệ. Trường ngân sách trong đầu ra của mô hình trông như thế này:

```json
{
  "total_attractions": 240,
  "total_hotels": 2600,
  "total_meals": 1748,
  "total_transportation": 400,
  "total": 5088
}
```

Vấn đề có hai tầng.

Tầng thứ nhất là phép tính số học đã sai: `240 + 2600 + 1748 + 400 = 4988`, nhưng mô hình lại viết thành `5088`. Tầng thứ hai phiền phức hơn, sau khi rule tính toán lại theo tiền ăn mỗi ngày thì phát hiện ra, tổng chi phí ăn uống thực tế phải là `1928`, chứ không phải `1748`. Nghĩa là nó vừa viết sai từng khoản lẻ, vừa viết sai tổng số, lại còn làm vượt quá mức hard budget 3200.

Rule bắt được là:

```text
budget_arithmetic_inconsistent: part_sum=4988, total=5088, diff=-100
meal_budget_inconsistent: expected_total_meals=1928, reported_total_meals=1748
budget_hard_constraint_exceeded: requested_budget=3200, total=5088
```

Sau này sửa thế nào? Tôi không tiếp tục trông chờ mô hình tự mình tính toán chuẩn xác tất cả các khoản chi tiêu nữa, mà tách việc đánh giá ngân sách thành hai bộ: một bộ nhìn vào `budget.total` do mô hình tự khai báo, một bộ dùng điểm tham quan, khách sạn, ăn uống, di chuyển để tính toán lại thành `recomputed_budget`. Trong huấn luyện và rerank cũng tách riêng số học ngân sách, quan hệ ngân sách, độ bám sát ngân sách ra xem xét. Bằng cách này, mô hình có thể tiếp tục chịu trách nhiệm lựa chọn phương án và lịch trình, còn sổ sách kế toán thì do quy tắc bảo đảm đáy (đỡ đầu).

### 2. Ăn uống lặp lại: Không phải không được ăn cùng một quán, mà là không được lười biếng suốt cả chặng đường

Ví dụ thứ hai là Trường Sa 4 ngày, thân thiện với gia đình có trẻ nhỏ và người già. Mô hình quả thực đã tìm được nhà hàng, nhưng lựa chọn quá lười biếng:

```text
2026-03-08 lunch  Tân Trường Phúc (chi nhánh Thế Gia)
2026-03-08 dinner Tân Trường Phúc (chi nhánh Thế Gia)
2026-03-10 dinner Tân Trường Phúc (chi nhánh Thế Gia)
2026-03-11 dinner Tân Trường Phúc (chi nhánh Thế Gia)
```

Cùng một ngày bữa trưa và bữa tối cùng một quán, những ngày sau lại tiếp tục lặp lại. Đối với mô hình, đây có thể là "nhà hàng địa phương an toàn và ổn định", nhưng đối với người dùng, trong 4 ngày mà cứ ăn đi ăn lại một quán thì trông rất không giống một lịch trình được lên kế hoạch nghiêm túc.

Rule bắt được là:

```text
meal_same_day_lunch_dinner_repeat: 2026-03-08 lunch/dinner đều là Tân Trường Phúc (chi nhánh Thế Gia)
meal_repeat_too_many: name_key=Tân Trường Phúc, count=4, max_allowed=3
meal_diversity_ok=False
```

Sau này sửa thế nào? Phần ăn uống này không thể chỉ viết mỗi "gợi ý nhà hàng đặc sản". Tôi đã thêm ba loại ràng buộc: bữa trưa và bữa tối trong cùng một ngày không được trùng nhau, cùng một chuỗi nhà hàng không được vượt quá giới hạn trên, nhà hàng tốt nhất nên đến từ danh sách ứng viên của công cụ chứ không được viết chung chung "đồ ăn vặt gần đó". Trong Best-of-N Replay và rerank cuối cùng, cũng đưa việc lặp lại ăn uống, tính đa dạng và grounding vào làm tín hiệu xếp hạng. Thay đổi này rất thực tế, vì nó không yêu cầu mô hình phải tạo ra câu trả lời hoàn hảo ngay trong một lần duy nhất, chỉ cần trong nhóm ứng viên có một phiên bản ít lặp lại hơn, rerank có thể vớt nó lên được.

### 3. Sai số đêm khách sạn: Số phòng dễ bị mô hình bỏ sót hơn số ngày

Ví dụ thứ ba là Bắc Kinh 3 ngày, 3 người bạn, ở khách sạn cao cấp. Lịch trình là từ 2026-01-07 đến 2026-01-09, do đó cần 2 đêm; 3 người lớn thông thường cần 2 phòng. Khách sạn mỗi ngày trong đầu ra của mô hình như sau:

```text
2026-01-07 Khách sạn Hoa Bắc estimated_cost=1500
2026-01-08 Khách sạn Hoa Bắc estimated_cost=1500
2026-01-09 Không có lưu trú
budget.total_hotels=3000
```

Thoạt nhìn không có vấn đề gì: 2 đêm, mỗi đêm 1500, tổng cộng 3000. Nhưng nó đã bỏ sót số lượng phòng. 3 người bạn không phải là ở 1 phòng 2 đêm, mà theo chiến lược nên tính là 2 phòng ở 2 đêm, ngân sách khách sạn ít nhất phải bao quát `1500 * 2 phòng * 2 đêm = 6000`.

Rule bắt được là:

```text
hotel_budget_underestimated:
  lodging_nights=2
  party_total=3
  room_count=2
  expected_min_total_hotels=6000
  reported_total_hotels=3000
```

Sau này sửa thế nào? Loại vấn đề này không thể trông chờ mô hình tự "hiểu một chút về số người" trong ngôn ngữ tự nhiên. Frontend và backend phải truyền `party.total` một cách tường minh, tầng chiến lược backend phải biên dịch ra `room_count` và `lodging_nights`, trong bộ đánh giá dùng tiếp `hotel_budget_covers_nights` để tóm ra. Dữ liệu huấn luyện cũng không thể chỉ viết các mẫu đơn giản một người một phòng, nếu không mô hình sẽ mặc định "giá khách sạn = một phòng một đêm * số đêm", hễ gặp bạn bè, gia đình, người già đi cùng là lập tức tính thiếu tiền.

Ba bad case này về sau cơ bản đã trở thành trình tự rà soát lỗi của tôi: trước tiên xem các khoản tính toán có cộng đúng không, tiếp theo xem ăn uống có lười biếng không, cuối cùng xem khách sạn có bao quát đủ theo số đêm và số phòng hay không. Rất nhiều vấn đề của mô hình trông có vẻ rất huyền bí, nhưng khi bóc tách đến đây thì thực ra đều rất mộc mạc.

---

## Chương 12: Kinh nghiệm đúc kết từ thử nghiệm lần này

Làm xong đợt này, điều tôi chắc chắn nhất là: hậu huấn luyện Agent không đơn giản là "tạo thêm ít dữ liệu rồi huấn luyện lại một chút". Điều thực sự phiền phức là khớp nối giữa giao thức sản phẩm, dữ liệu, huấn luyện, đánh giá và chiến lược suy luận.

Những điều cuối cùng tôi ghi nhớ lại không hề phức tạp:

1. **Sửa giao thức sản phẩm trước.** Những trường có thể gửi có cấu trúc thì đừng bắt mô hình đoán.
2. **Biên dịch ngữ cảnh cho thật tốt.** Mô hình nên đưa ra lựa chọn dựa trên các ứng viên, chứ không phải bịa ra sự thật từ hư vô.
3. **Đóng băng bộ đánh giá trước.** Huấn luyện có thể đổi dữ liệu, nhưng đánh giá đừng mỗi vòng lại đổi đề bài.
4. **Tinh chỉnh Prompt dùng để tìm ranh giới.** Những gì prompt giải quyết được thì viết vào prompt, không giải quyết được thì giao cho dữ liệu, quy tắc hoặc kỹ thuật công trình.
5. **Mô hình mạnh tạo dữ liệu, nhưng teacher bắt buộc phải được kiểm toán.** Trông ra ngô ra khoai không đồng nghĩa với có thể dùng để huấn luyện.
6. **LoRA SFT đẩy mạnh theo từng giai đoạn.** Mỗi vòng cố gắng chỉ giải quyết một loại vấn đề chính.
7. **Phải phân biệt rõ Best-of-N Replay và Rerank cuối cùng.** Cái trước để tạo dữ liệu huấn luyện, cái sau để chọn câu trả lời lúc suy luận.
8. **DPO chỉ học sở thích giữa các ứng viên hợp lệ.** Đừng lấy pair sửa định dạng làm pair sở thích.
9. **Dữ liệu huấn luyện phải tránh bộ đánh giá đóng băng.** Dự án thử nghiệm cũng không cần học thuộc lòng đề, việc kiểm tra trùng lặp rất rẻ.
10. **Tách các chỉ số ra để xem.** hardpass, planner soft, quan hệ ngân sách, tính lại ngân sách đừng nhào nặn chung thành một điểm tổng.

Cuối cùng tôi sẽ kết lại như thế này:

> Những gì có thể cấu trúc hóa hãy giao cho kỹ thuật công trình, những gì có thể quy tắc hóa hãy làm thành bộ đánh giá, những phần còn lại thực sự cần mô hình học thì mới đưa vào SFT hoặc huấn luyện sở thích.

Mô hình được huấn luyện theo cách này dĩ nhiên sẽ biết ăn nói hơn, nhưng trọng tâm không nằm ở đó. Quan trọng hơn là nó có thể kết nối vào frontend và backend; khi gặp sự cố, quy tắc có thể định vị được lỗi; và ở vòng tiếp theo vẫn có thể tiếp tục sửa chữa nâng cấp.

## Tài nguyên tái hiện

Nếu muốn tái hiện (reproduce), chỉ cần bắt đầu từ những tài liệu này:

- Repository dự án trợ lý du lịch: [https://github.com/nameless0120/helloagents-trip-planner](https://github.com/nameless0120/helloagents-trip-planner)
- Hướng dẫn đầy đủ: [Hướng dẫn thực hành hậu huấn luyện Trợ lý du lịch](https://github.com/nameless0120/helloagents-trip-planner/blob/main/training/docs/%E6%95%99%E7%A8%8B/%E6%97%85%E8%A1%8C%E5%8A%A9%E6%89%8B%E5%90%8E%E8%AE%AD%E7%BB%83%E5%AE%9E%E6%88%98%E6%95%99%E7%A8%8B.md)
- Dữ liệu đi kèm: `helloagents-后训练数据`, <https://pan.baidu.com/s/5oNsK7pwQnqzQEUg5ykb09Q>

Chi tiết phần cứng và script bài viết này không đi sâu. Nói một cách đơn giản, tuyến chính này dựa vào LoRA ngữ cảnh dài; nếu cắt ngắn ngữ cảnh, đổi thành QLoRA hoặc giảm rank nhỏ lại, cũng có thể chạy được, nhưng sẽ không còn là lộ trình như trong bài viết nữa. Khi tái hiện thử nghiệm, hãy lấy cấu hình và tài liệu lưu trữ trong repository `helloagents-trip-planner` làm chuẩn.
