# Web Agent: Phổ cập kiến thức và Thực hành — Dạy AI biết "lên mạng"

## Lời mở đầu: Khi AI học được cách duyệt web

Hãy tưởng tượng một kịch bản như thế này: bạn nói với AI "Giúp tôi tìm trên các trang web du lịch phổ biến ba chuyến bay thẳng từ Bắc Kinh đi Singapore vào thứ Ba tuần sau, giá dưới 3000 tệ, so sánh xong thì dùng thẻ tín dụng tôi đã lưu để đặt chuyến có lợi nhất" — và rồi AI thực sự mở một trình duyệt, tự chủ điều hướng quy trình đặt vé, xử lý đủ loại popup và trang xác minh trên đường đi, điền thông tin hành khách, và cuối cùng trao vào tay bạn mã xác nhận đơn hàng.

Đây không phải là khoa học viễn tưởng, mà là **Web Agent (tác tử web)** đang dần trở thành hiện thực. Nếu nói GUI Agent được giới thiệu ở Extra06 đã dạy AI cách thao tác điện thoại và ứng dụng desktop, thì thứ mà chương này giới thiệu chính là "họ hàng gần" của nó trong thế giới web — tác tử lấy trình duyệt làm bề mặt hành động chính.

Web xứng đáng có một chương riêng vì hai lý do. Thứ nhất, **phần lớn dữ liệu và quy trình làm việc (workflow) có giá trị nhất trên thế giới đều chạy trên web** — nhiều hơn rất nhiều so với App điện thoại hay phần mềm desktop. Thứ hai, **Web là một bài toán kỹ thuật độc lập**: những thách thức về cảm nhận (perception), hành động và độ tin cậy của nó khác biệt về bản chất so với GUI Agent trên di động/desktop. Bạn không thể cắm thẳng Mobile-Agent-v3 vào Chrome rồi coi đó là Web Agent, con đường này không đi được.

Cam kết của chương này là, sau khi đọc xong bạn sẽ:

- Hiểu được sự khác biệt bản chất giữa Web Agent với RPA truyền thống và GUI Agent đa dụng
- Nắm vững ba chiến lược cảm nhận chủ đạo (DOM, thị giác, lai), và hiểu vì sao các hệ thống cấp production nhìn chung đều đi theo hướng lai
- Thông qua việc gọi SDK, cũng như tích hợp dưới dạng công cụ vào framework **HelloAgents** đã xây dựng ở chương 7, tự tay sử dụng một dịch vụ Web Agent cấp production — **TinyFish**
- Học cách chẩn đoán và vượt qua các cơ chế chống thu thập dữ liệu (anti-scraping / anti-bot) thường gặp
- Biết khi nào **không nên** sử dụng Web Agent dạng dịch vụ được quản lý (managed / hosted)

---

## Phần một: Phổ cập kiến thức kỹ thuật về Web Agent

### 1.1 Web Agent là gì?

**Web Agent** là một loại tác tử tự chủ lấy trình duyệt web làm bề mặt hành động chính. Nó cảm nhận trang web thông qua một tổ hợp nào đó giữa DOM, cây trợ năng (accessibility tree) và ảnh chụp màn hình, dùng mô hình ngôn ngữ lớn để suy luận bước tiếp theo nên làm gì, rồi thực thi hành động trong một instance trình duyệt thật — nhấp chuột, nhập liệu, cuộn, chuyển trang.

Từ khóa là **tự chủ**. Một script crawler được viết cứng bằng Playwright không phải là Web Agent, nó chỉ là một đoạn chương trình mong manh, trang web mục tiêu chỉ cần đổi vị trí nút một chút là nó sập. Còn Web Agent thì quyết định động hành động tiếp theo tại thời điểm chạy (runtime) dựa trên nội dung mà nó "nhìn thấy".

#### 1.1.1 Web Agent vs RPA vs GUI Agent

Dòng dõi kỹ thuật này rất quan trọng. Chúng ta hãy đặt ba thứ cạnh nhau để so sánh:

| Chiều so sánh | RPA truyền thống (Selenium, UiPath) | GUI Agent (Mobile-Agent, AutoGLM) | Web Agent (Browser-Use, TinyFish) |
| --- | --- | --- | --- |
| **Bề mặt chính** | Web hoặc desktop | Di động + desktop, chủ yếu dựa vào ảnh chụp màn hình | Trình duyệt web |
| **Cách cảm nhận** | DOM selector (XPath, CSS) | Thị giác (VLM phân tích ảnh chụp màn hình) | DOM + cây trợ năng + thị giác (lai) |
| **Cơ chế hành động** | Script cố định | Nhấp chuột dựa trên tọa độ | Kết hợp selector, tọa độ và mục tiêu ngữ nghĩa |
| **Có thích ứng với thay đổi UI không** | Không — mất hiệu lực ngay lập tức | Có — thị giác có tính đàn hồi về ngữ nghĩa | Có — nhiều tín hiệu neo (anchor) |
| **Khả năng đa nền tảng** | Hạn chế | Tự nhiên đa nền tảng | Bản thân Web đã là đa nền tảng |
| **Xử lý xác thực / phiên (session)** | Thủ công | Hạn chế | Công dân hạng nhất (first-class citizen) |
| **Ý thức về chống bot** | Gần như không có | Không liên quan | Cực kỳ quan trọng |

Hai điểm then chốt nhất:

1. **Web Agent không phải là "GUI Agent giới hạn trên Chrome"**. Nó sẽ tận dụng thông tin có cấu trúc từ DOM, cây trợ năng và tầng mạng — những thông tin này hoặc là không tồn tại trong môi trường di động/desktop, hoặc là GUI Agent chọn không dùng. Web Agent cấp production nhất định phải là lai: thị giác phụ trách hiểu bố cục, DOM/AX-tree phụ trách định vị chính xác, và chuyển đổi tùy theo tình huống cụ thể của từng hành động.
2. **Web có những đối thủ riêng của nó**. Cloudflare, DataDome, PerimeterX, nhận dạng dấu vân tay (fingerprinting), phân tích đặc trưng sinh trắc học hành vi (behavioral biometrics) — những thứ này đều không tồn tại đối với GUI Agent trên điện thoại/desktop. Một Web Agent không tính đến chống bot thì trong môi trường production chỉ là một Web Agent trả về kết quả rỗng.

#### 1.1.2 Vì sao Web Agent đột nhiên bùng nổ trong giai đoạn 2024–2026?

Ba nguồn lực hội tụ cùng một lúc:

1. **Mô hình lớn đa phương thức có thể đọc hiểu đồng thời cả ảnh chụp màn hình và DOM**. GPT-4o, Claude Sonnet 4, Gemini 2.5, Qwen-VL — năng lực định vị trực quan (visual grounding) các phần tử UI của chúng đều đã đủ mạnh, có thể điều khiển trình duyệt mà không cần huấn luyện thị giác chuyên biệt.
2. **Hạ tầng trình duyệt không giao diện (headless browser) đã trưởng thành**. Playwright và Chrome DevTools Protocol (CDP) cho phép chúng ta khởi chạy một instance Chromium thật ở bất kỳ đâu trên thế giới với chi phí rẻ, kèm theo các bản vá ẩn danh (stealth patch), định tuyến proxy, điều khiển từ xa.
3. **Nỗi đau thực tế đủ lớn**. Một nửa dữ liệu trên thế giới nằm sau những trang web không có API — giá cả, tồn kho, hồ sơ công khai của chính phủ, kho lưu trữ tin tức, đủ loại dashboard nội bộ mà các nhà cung cấp triển khai cho doanh nghiệp. Giá trị kinh tế của việc tự động hóa những workflow này là cực lớn, và chỉ có Web Agent mới lấp được khoảng trống này.

Chúng ta cũng vừa hay bắt kịp thời điểm các phòng thí nghiệm AI lớn bắt đầu trực tiếp sản phẩm hóa năng lực này — Computer Use của Anthropic, Operator của OpenAI, Project Mariner của Google. "Web Agent" đang nhanh chóng trở thành năng lực nền tảng mà mọi sản phẩm AI-native (thuần AI) đều sẽ tích hợp.

### 1.2 Kiến trúc kỹ thuật cốt lõi

Giống như mọi GUI Agent, Web Agent cũng là một vòng lặp khép kín **cảm nhận → suy luận → hành động**. Nhưng mỗi tầng đều có những điểm tinh tế đặc thù của Web.

#### 1.2.1 Tầng cảm nhận: Ba chiến lược, mỗi chiến lược đều có điểm yếu chí mạng

Trong ngành có ba chiến lược cảm nhận chủ đạo, mỗi chiến lược đều có điểm yếu mà hai chiến lược còn lại vừa hay có thể bù đắp:

**Chiến lược A — Dựa trên DOM**: Phân tích HTML và cây trợ năng của trang. Vừa nhanh vừa chính xác, có thể trực tiếp cho bạn biết loại phần tử, nội dung văn bản, selector. **Kịch bản thất bại**: trang là SPA được render bằng Canvas (Google Docs, Figma, v.v.), hoặc trang web cố tình làm rối (obfuscate) DOM, hoặc nội dung có ý nghĩa được render vào Shadow DOM mà crawler không đọc được.

**Chiến lược B — Dựa trên thị giác**: Chụp màn hình → đưa vào mô hình lớn thị giác → mô hình nhận diện phần tử cần nhấp và xuất ra tọa độ. **Kịch bản thất bại**: mô hình đọc sai chữ nhỏ, ảo giác (hallucination) về vị trí phần tử, trên trang 4K có những thứ nó đơn giản là không nhìn thấy. Hơn nữa chi phí cao — mỗi bước đều phải gọi suy luận VLM một lần.

**Chiến lược C — Lai (người chiến thắng trong môi trường production)**: Dùng thị giác để hiểu bố cục và suy luận ngữ cảnh ("trong ba ô tìm kiếm này, ô nào mới là cái tôi cần?"), rồi dùng DOM hoặc cây trợ năng để định vị chính xác ("đây là phần tử cụ thể cần nhấp"). Tất cả các Web Agent cấp production nghiêm túc — Operator, Computer Use, Browser-Use, TinyFish — cuối cùng đều đi theo hướng lai.

Vì sao ngành lại hội tụ về điểm này? Hãy để dữ liệu thực nghiệm lên tiếng: trên các workflow production phổ biến, những hệ thống trưởng thành đã qua phương án lai + trình duyệt ẩn danh + gia cố chống bot có tỷ lệ thành công ổn định ở mức khoảng 90%; trong khi các phương án thuần thị giác, chưa gia cố trên các benchmark học thuật mang tính đối kháng như WebArena thường chỉ đạt 30–40%. Lưu ý rằng hai con số này đo lường **những thứ khác nhau** — benchmark học thuật cố tình xây dựng các kịch bản chuỗi dài, hiểm hóc; còn hệ thống production thì đã được tinh chỉnh có mục tiêu rất nhiều trên các luồng nghiệp vụ thực tế. Nhưng cả hai đường cong đều cho thấy cùng một điều: phương án lai rõ ràng chiếm ưu thế về mặt kỹ thuật.

#### 1.2.2 Tầng suy luận: Bộ nhớ, phản tư và vấn đề trạng thái

Thách thức lớn nhất của Web ở tầng suy luận là **đột biến trạng thái**. Trang web sẽ thay đổi ngay trước mắt bạn:

- Cuộn vô hạn (infinite scroll) sẽ liên tục tải nội dung mới khi bạn cuộn xuống cuối trang
- Hai giây sau khi trang tải xong thì bật ra một hộp thoại modal, che mất nút bạn muốn nhấp
- Sau khi nhấp "Thêm vào giỏ hàng", trang render lại, bản chụp DOM (DOM snapshot) trước đó của bạn lập tức mất hiệu lực
- Quy trình đăng nhập sẽ đi qua ba trang trung gian rồi mới đưa bạn đến dashboard

Một Web Agent đạt chuẩn cần có:

- **Phân rã nhiệm vụ** — chia "giúp tôi tìm chuyến bay thứ Bảy giá dưới 3000" thành một chuỗi các bước cụ thể
- **Phản tư (reflection)** — nhận ra một hành động không tạo ra hiệu quả như mong đợi, và có thể phục hồi
- **Bộ nhớ** — khi lật sang trang thứ hai vẫn nhớ nội dung kết quả tìm kiếm của trang thứ nhất
- **Theo dõi trạng thái** — phân biệt "nút biến mất vì tôi đã nhấp vào nó" và "nút biến mất vì trang đã sập"

Đây chính là sân khấu để **mô hình ReAct** ở chương 4 tỏa sáng. Web Agent về bản chất chính là một vòng lặp ReAct, trong đó không gian hành động là `{click, type, scroll, navigate, wait, extract}`, và quan sát là trạng thái trang tiếp theo.

#### 1.2.3 Tầng hành động: Những khó khăn đặc thù trên Web

Không gian hành động thoạt nhìn tương tự với GUI Agent trên di động — nhấp, nhập liệu, cuộn. Nhưng tầng Web có thêm:

- **Trạng thái phiên**: cookies, localStorage, sessionStorage cần được duy trì xuyên suốt các hành động (hoặc cố tình xóa sạch)
- **Lịch sử điều hướng**: quay lại, tiến tới, tab mới, chuyển tab
- **Tải xuống và tải lên tệp**: không đi qua DOM
- **Duyệt iframe và Shadow DOM**: một trang web không phải là một tài liệu, mà là một cây các tài liệu
- **Các mối quan tâm ở tầng mạng**: sau khi bạn nhấp "Gửi", có thể phải đợi một XHR hoàn tất mới được coi là "xong"

Web Agent chỉ dùng tọa độ sẽ đụng phải từng giới hạn ở trên. Đây chính là lý do phương án lai trở thành tiêu chuẩn production.

### 1.3 Những bài toán khó riêng có của Web

Những vấn đề dưới đây Extra06 không có và cũng không thể bao quát, bởi vì chúng hoàn toàn không tồn tại trên di động và desktop:

**Cơ chế chống bot**. Cloudflare, DataDome, PerimeterX, Akamai Bot Manager. Chúng theo dõi dấu vân tay TLS của bạn, chữ ký đối tượng JavaScript của trình duyệt, mẫu di chuyển chuột, nhịp thời gian thao tác. Một script Playwright đơn giản sẽ bị chặn lại chỉ trong vài giây. Web Agent cấp production cần **trình duyệt ẩn danh (stealth browser)** (Chromium đã được vá, trông không thể phân biệt với trình duyệt của người dùng thật), thường còn phải đi kèm proxy IP dân cư (residential proxy).

**Xác thực và duy trì phiên**. Luồng OAuth, xác thực hai lớp, cookie phiên, token "ghi nhớ đăng nhập", mã captcha khi đăng nhập. Một Web Agent không đăng nhập được chỉ có thể nhìn thấy các trang web công khai. Phương án đi kèm kho thông tin xác thực được mã hóa (vault) là một mô hình mới nổi.

**Nội dung động và điều kiện tranh chấp (race condition)**. Cuộn vô hạn, ảnh tải lười (lazy loading), hộp thoại modal 800ms sau mới xuất hiện, animation chặn cú nhấp. Tác tử phải biết khi nào trang được coi là "sẵn sàng", mà "sẵn sàng" thì không có định nghĩa rõ ràng.

**SPA nặng JS**. Khi bạn truy cập một trang React hoặc Vue hiện đại, HTML ban đầu về cơ bản là rỗng — DOM thực sự là do JavaScript render ra sau ba giây. Crawler đơn giản sẽ không nhìn thấy bất kỳ nội dung nào.

**Mạng, địa lý và giới hạn tốc độ (rate limiting)**. Một số nội dung bị giới hạn theo địa lý, một số API giới hạn tốc độ theo IP, một số trang web sẽ âm thầm hạ cấp phản hồi khi phát hiện "quá nhiều" yêu cầu đến từ cùng một nguồn.

**Chi phí và độ trễ**. Mỗi bước hành động là một lần gọi LLM + một lượt khứ hồi với trình duyệt. Một nhiệm vụ tự động hóa 10 bước có thể mất 30–60 giây, riêng chi phí token LLM đã là 0,10–1,00 đô la Mỹ. Đây chính là lý do ngành ra sức theo đuổi các mô hình nhỏ hơn, nhanh hơn, được tinh chỉnh chuyên biệt cho Web.

### 1.4 Toàn cảnh năm 2026: So sánh bốn loại "người chơi"

Hệ sinh thái Web Agent năm 2026 đại khái có thể chia thành bốn loại, sự tồn tại của mỗi loại đều tương ứng với một sự đánh đổi (trade-off) thực tế:

| Loại | Dự án tiêu biểu | Ưu điểm | Điểm yếu |
| --- | --- | --- | --- |
| **Tự động hóa trình duyệt nguyên thủy** | Playwright, Puppeteer, Selenium | Nhanh, có thể lặp lại, miễn phí, hoàn toàn kiểm soát được | Không có AI, mong manh, mỗi lần UI thay đổi đều phải sửa script |
| **Web Agent AI mã nguồn mở** | Browser-Use, Skyvern, WebVoyager, AgentE | Miễn phí, có thể tùy biến sâu, hoàn toàn minh bạch | Tự triển khai, tự xử lý chống bot, proxy, hạ tầng, khả năng quan sát (observability) |
| **Computer-use API** | Anthropic Computer Use, OpenAI Operator, Google Project Mariner | Năng lực suy luận đa dụng mạnh, mô hình tiên tiến | Đắt, UX mang tính áp đặt (opinionated), không có sẵn chống bot hay định tuyến địa lý, thường lấy desktop làm chính chứ không phải thuần Web |
| **API tự động hóa Web được quản lý** | **TinyFish**, Browserbase, Apify, Bright Data | Trình duyệt ẩn danh + proxy + vòng lặp tác tử được đóng gói trọn bộ, vài phút là chạy được; tích hợp sẵn khả năng quan sát | Phụ thuộc vào nhà cung cấp, ít kiểm soát hơn đối với vòng lặp tác tử bên trong, tính phí theo nhiệm vụ |

Đánh giá trung thực: mỗi loại đều thắng trên một trục khác nhau. Bạn là nhà nghiên cứu, muốn nghiên cứu hành vi của Agent? Dùng mã nguồn mở. Bạn là nhà phát triển cá nhân, crawl một trang nhỏ không có chống bot? Playwright thuần là đủ. Bạn đang làm tính năng cho sản phẩm, cần "dùng được ngay" trên các trang web của thế giới thực (bao gồm cả chống bot)? API được quản lý có thể tiết kiệm cho bạn vài tuần.

Trong phần hai, chúng ta sẽ dùng TinyFish để luyện tập thực hành — thiết kế API của nó là gọn gàng nhất trong loại này, và nó phơi bày vòng lặp tác tử một cách rõ ràng, thuận tiện cho việc học.

### 1.5 Các kịch bản ứng dụng trong thế giới thực

Dưới đây là danh sách (chưa đầy đủ) các trường hợp Web Agent đã được triển khai trong môi trường production:

- **Giám sát thương mại điện tử**: giá cả, tồn kho, sản phẩm của đối thủ, tổng hợp đánh giá
- **Làm giàu dữ liệu B2B và tạo khách hàng tiềm năng (lead generation)**: thu thập thông tin từ các trang kiểu LinkedIn, xây dựng cơ sở dữ liệu công ty
- **RPA nội bộ cho các hệ thống Web cũ (legacy)**: hàng nghìn hàng vạn công ty có nghiệp vụ cốt lõi vẫn chạy trên các hệ thống thuần UI không có API, Web Agent cuối cùng đã giúp chúng có thể được tự động hóa
- **Đảm bảo chất lượng và kiểm thử tự động**: kiểm thử hồi quy (regression test), kiểm toán khả năng truy cập (accessibility audit), tuân thủ nội dung
- **Tác tử nghiên cứu (chủ đề của chương 14)**: một tác tử nghiên cứu không thể duyệt web thì coi như mù một nửa, Web Agent là backend tự nhiên của các hệ thống kiểu DeepResearch
- **Workflow du lịch và đặt chỗ (chủ đề của chương 13)**: giá trị sử dụng của trợ lý du lịch sẽ tăng gấp 10 lần khi nó "có thể thực sự hoàn tất việc đặt chỗ" thay vì "chỉ có thể đưa ra gợi ý"
- **Tự động hóa cá nhân**: theo dõi một tin tuyển dụng, giám sát hàng có lại trong kho, theo dõi thông báo công khai của chính phủ

### 1.6 Những hạn chế thực tế của công nghệ hiện tại

Hãy trung thực với độc giả của bạn. Web Agent vừa thú vị, vừa có những giới hạn của nó:

- **Tỷ lệ thành công: benchmark và production là hai đường cong khác nhau**. Web Agent cấp production (phương án lai + ẩn danh + gia cố chống bot) trên các luồng nghiệp vụ phổ biến đã có thể ổn định ở mức **khoảng 90%**; nhưng trên các benchmark học thuật đối kháng được xây dựng có chủ đích như WebArena, VisualWebArena, ngay cả những hệ thống tốt nhất vẫn đang tiếp tục đuổi theo. Nói cách khác: những gì có thể làm được trong vòng lặp kỹ thuật nhiều hơn rất nhiều so với vẻ bề ngoài của các bài đánh giá học thuật — nhưng nhiệm vụ càng ngách, chuỗi càng dài thì khoảng cách càng lớn.
- **Chống bot là một trò chơi mèo vờn chuột luôn dịch chuyển**. Phương án dùng tốt tháng trước có thể tháng này đã bị phát hiện. Cuộc đối đầu với các nhà cung cấp không bao giờ kết thúc.
- **Chi phí là có thật**. API được quản lý trung bình 0,10–1,00 đô la Mỹ cho mỗi nhiệm vụ. Tự triển khai thì tiết kiệm tiền, nhưng đốt thời gian kỹ thuật.
- **Captcha là giới hạn cứng**. Dù nhà cung cấp quảng cáo thế nào, Web Agent hiện đại **vẫn không thể giải reCAPTCHA hoặc hCaptcha một cách đáng tin cậy**. Cách làm thực tế là thiết kế nhiệm vụ để tránh nó, chứ không phải cố đối đầu.
- **Cái giá của ảo giác là tiền thật**. Một trường biểu mẫu điền sai có thể khiến bưu kiện bị gửi đến sai địa chỉ. Các hành động rủi ro cao bắt buộc phải có điểm kiểm tra có con người trong vòng lặp (human-in-the-loop).

---

## Phần hai: Hướng dẫn thực hành

Chúng ta sẽ đi ba bước từ dễ đến khó:

1. **Bắt đầu siêu tốc** — chạy thành công lần gọi TinyFish đầu tiên trong vòng 5 phút, không framework, không cần môi trường
2. **Quan sát tác tử làm việc** — dùng URL phát trực tiếp (live URL) để gỡ lỗi trực quan từng lần chạy
3. **Tích hợp vào HelloAgents** — đóng gói TinyFish thành một `Tool`, kết nối vào tác tử ReAct trong framework của chương 7

### 2.1 Bắt đầu siêu tốc: Web Agent đầu tiên của bạn trong vòng 5 phút

TinyFish là một API Web Agent dạng dịch vụ được quản lý. Bạn đưa cho nó một URL và một đoạn **mục tiêu (goal)** bằng ngôn ngữ tự nhiên, nó trả về JSON có cấu trúc.

#### Bước 1: Lấy API key

1. Đăng ký tài khoản tại `agent.tinyfish.ai`
2. Vào trang API Keys, nhấp **Create API Key**
3. Sao chép key — nó chỉ hiển thị một lần
4. Ghi vào shell của bạn:

```bash
export TINYFISH_API_KEY="sk-tinyfish-..."
```

#### Bước 2: Cài đặt SDK

```bash
# Python
pip install tinyfish

# Hoặc TypeScript
npm install @tiny-fish/sdk
```

#### Bước 3: Nhiệm vụ tự động hóa đầu tiên của bạn

**Python:**

```python
from tinyfish import TinyFish, CompleteEvent

client = TinyFish()  # Tự động đọc TINYFISH_API_KEY từ biến môi trường

with client.agent.stream(
    url="https://scrapeme.live/shop",
    goal="Trích xuất tên và giá của 2 sản phẩm đầu tiên, trả về dưới dạng JSON.",
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            print(event.result_json)
```

**TypeScript:**

```typescript
import { TinyFish, EventType } from "@tiny-fish/sdk";

const client = new TinyFish();

const stream = await client.agent.stream({
  url: "https://scrapeme.live/shop",
  goal: "Trích xuất tên và giá của 2 sản phẩm đầu tiên, trả về dưới dạng JSON.",
});

for await (const event of stream) {
  if (event.type === EventType.COMPLETE) {
    console.log(event.result);
  }
}
```

**HTTP nguyên thủy (không dùng SDK):**

```bash
curl -N -X POST https://agent.tinyfish.ai/v1/automation/run-sse \
  -H "X-API-Key: $TINYFISH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://scrapeme.live/shop",
    "goal": "Trích xuất tên và giá của 2 sản phẩm đầu tiên"
  }'
```

Khi chạy, bạn sẽ thấy các Server-Sent Events chảy vào terminal theo thời gian thực:

```
{"type": "STARTED", "run_id": "abc123"}
{"type": "STREAMING_URL", "run_id": "abc123", "streaming_url": "https://tf-abc123.fra0-tinyfish.unikraft.app/stream/0"}
{"type": "PROGRESS", "run_id": "abc123", "purpose": "Visit the page to extract product information"}
{"type": "PROGRESS", "run_id": "abc123", "purpose": "Check for product information on the page"}
{"type": "COMPLETE", "run_id": "abc123", "status": "COMPLETED", "result": {
  "products": [
    {"name": "Bulbasaur", "price": "$63.00"},
    {"name": "Ivysaur", "price": "$87.00"}
  ]
}}
```

Vậy là xong — bạn vừa chạy xong một Web Agent. Những gì đã xảy ra đằng sau lần gọi này: một trình duyệt Chromium thật được khởi chạy trong trung tâm dữ liệu, điều hướng đến trang mục tiêu, phân tích bố cục, nhận diện các thẻ sản phẩm, trích xuất dữ liệu, và stream tiến trình trở lại cho bạn.

#### Giải phẫu một request

Các trường JSON mà endpoint `/run-sse` của TinyFish tiếp nhận (**các trường quan trọng đã được in đậm**):

| Trường | Kiểu | Mô tả |
| --- | --- | --- |
| **`url`** | string | Trang khởi đầu |
| **`goal`** | string | Chỉ thị bằng ngôn ngữ tự nhiên |
| `output_schema` | object | Tập con JSON Schema tùy chọn, ràng buộc cấu trúc trả về |
| `browser_profile` | `"lite"` \| `"stealth"` | Mặc định là `lite` |
| `proxy_config` | object | Tùy chọn `{enabled, type, country_code}` |
| `use_vault` | boolean | Có sử dụng thông tin xác thực đã lưu để hoàn tất đăng nhập hay không |
| `credential_item_ids` | string[] | Giới hạn các thông tin xác thực cụ thể trong vault |

Ba endpoint, ba chế độ:

- `POST /v1/automation/run` — đồng bộ, chặn cho đến khi hoàn tất, trả về kết quả cuối cùng
- `POST /v1/automation/run-async` — gửi xong là quên (fire-and-forget), trả về `run_id` ngay lập tức
- `POST /v1/automation/run-sse` — streaming, đẩy các sự kiện tiến trình về cho bạn theo thời gian thực

Nhiệm vụ ngắn (< 30 giây) dùng đồng bộ; tác vụ hàng loạt (batch job) dùng bất đồng bộ; khi cần hiển thị tiến trình theo thời gian thực trên UI thì dùng SSE.

### 2.2 Quan sát tác tử làm việc — URL phát trực tiếp

Một trong những tính năng hữu ích nhất của TinyFish về mặt giảng dạy là **`streaming_url`**: mỗi lần chạy đều tạo ra một URL, cho phép bạn **xem trình duyệt theo thời gian thực**. Bạn có thể nhúng nó vào iframe trong sản phẩm của mình, hoặc mở thẳng một Tab trong trình duyệt để vừa gỡ lỗi vừa xem.

```python
from tinyfish import TinyFish, CompleteEvent

client = TinyFish()

with client.agent.stream(
    url="https://scrapeme.live/shop",
    goal="Trích xuất tên và giá của 3 sản phẩm đầu tiên",
    on_streaming_url=lambda e: print(f"\nXem trực tiếp: {e.streaming_url}\n"),
    on_progress=lambda e: print(f"  → {e.purpose}"),
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            print("\nKết quả cuối cùng:", event.result_json)
```

Khi chạy, gần như ngay lập tức bạn sẽ nhận được một URL có thể nhấp vào. Mở nó trong trình duyệt — thứ bạn nhìn thấy chính là **phiên trình duyệt thật mà tác tử đang điều khiển**. Trang đang tải, chuột đang di chuyển, các trường đang được điền. Đây là khoảnh khắc mà phần lớn độc giả sẽ thốt lên "Trời đất, nó thực sự đang làm kìa".

Và đây không chỉ là một chiêu trò demo. Luồng phát trực tiếp là công cụ **hữu dụng nhất** để gỡ lỗi Web Agent:

- Một lần chạy trả về kết quả rỗng? Mở luồng trực tiếp — rất có thể bạn sẽ thấy một trang chặn "Checking your browser" của Cloudflare, hoặc một popup ngoài dự kiến
- Tác tử nhấp nhầm nút? Xem luồng trực tiếp, sửa goal
- Đang làm sản phẩm hướng đến người dùng? Nhúng URL trực tiếp vào iframe, để người dùng nhìn thấy công việc đang diễn ra

### 2.3 Làm thế nào để viết goal cho tốt

Riêng trường `goal` này gần như là toàn bộ bề mặt API của TinyFish — viết tốt nó chính là ranh giới giữa tỷ lệ thành công 90% và tỷ lệ thành công 30%.

Mô hình tư duy (mental model) mà TinyFish chính thức khuyến nghị: **hãy coi tác tử như một trợ lý "thực thi theo nghĩa đen", đang ngồi trước trình duyệt**. Nó có thể nhìn thấy mọi thứ trên màn hình, có thể ra tay — nhưng nó không thể đoán ý bạn.

Một goal xuất sắc bao gồm tối đa bảy thành phần:

| Thành phần | Ví dụ |
| --- | --- |
| **Mục tiêu (Objective)** | "Trích xuất thông tin giá" |
| **Phạm vi (Target)** | "Từ bảng giá" |
| **Trường (Fields)** | "Tên gói, phí hàng tháng, tính năng bao gồm" |
| **Cấu trúc (Schema)** | "Trả về JSON: `[{plan: string, price_monthly: number}]`" |
| **Các bước (Steps)** | "Đóng banner cookie trước" |
| **Rào chắn (Guardrails)** | "Không nhấp vào bất kỳ nút 'Mua ngay' nào" |
| **Trường hợp biên (Edge cases)** | "Nếu giá hiển thị là 'Liên hệ với chúng tôi', đặt thành null" |

Sự tiến hóa từ tệ nhất đến tốt nhất:

**Mơ hồ (chắc chắn thất bại):**
```
"Lấy giá của trang này"
```

**Tốt hơn một chút (có thể chạy được):**
```
"Trích xuất tên sản phẩm, giá và trạng thái tồn kho"
```

**Chất lượng cấp production:**
```
1. Đợi trang tải hoàn toàn.
2. Nếu có banner đồng ý cookie, nhấp "Chấp nhận tất cả".
3. Định vị khu vực bảng giá.
4. Với mỗi gói, trích xuất: tên gói, phí hàng tháng (số), tính năng bao gồm (mảng chuỗi).

Không nhấp vào bất kỳ nút mua hàng hoặc thanh toán nào.
Nếu gói hiển thị "Liên hệ với chúng tôi", đặt monthly_price thành null.

Trả về JSON: [{"plan": string, "monthly_price": number | null, "features": string[]}]
```

Trong các thử nghiệm của chính TinyFish, với cùng một nhiệm vụ, goal cụ thể **hoàn thành nhanh hơn 4,9 lần**, **trả về dữ liệu thừa ít hơn 16 lần**. Viết goal chính là Prompt Engineering kiểu mới, và giống như mọi Prompt Engineering khác, hiệu quả cộng dồn (lãi kép) rất rõ ràng.

### 2.4 Món chính: Tích hợp TinyFish vào HelloAgents

Phần này là cốt lõi của chương. Gọi API trực tiếp tất nhiên là được — nhưng thông điệp cốt lõi mà cuốn sách này, đặc biệt là chương 4 và chương 7, luôn muốn truyền tải là: **khi dịch vụ bên ngoài trở thành công cụ trong framework của chính bạn, sức mạnh thực sự mới xuất hiện**. Đây chính là ranh giới giữa "sử dụng AI" và "xây dựng bằng AI".

Chúng ta sẽ đóng gói TinyFish thành một `Tool` của HelloAgents, kết nối vào `ReActAgent`.

> **Gợi ý**: HelloAgents là framework đi kèm mà bạn đã xây dựng ở chương 7. Nếu chưa cài đặt: `pip install hello-agents`. Nếu muốn phiên bản tương ứng hoàn toàn với nội dung chính, có thể chuyển sang nhánh `learn_version` trên GitHub.

#### Bước 1: Định nghĩa `TinyFishWebTool`

Chúng ta viết một công cụ, phơi bày tự động hóa Web thành một năng lực duy nhất. Tác tử dùng ngôn ngữ tự nhiên để mô tả việc nó muốn làm, công cụ phụ trách gọi API và trả về JSON có cấu trúc.

```python
# tools/tinyfish_tool.py
import json
import os
from typing import Any, Dict, List

from tinyfish import (
    TinyFish,
    BrowserProfile,
    ProxyConfig,
    ProxyCountryCode,
)

from hello_agents.tools import Tool, ToolParameter


class TinyFishWebTool(Tool):
    """Công cụ cho phép tác tử ReAct điều khiển trình duyệt thật thông qua ngôn ngữ tự nhiên."""

    def __init__(self, api_key: str | None = None):
        super().__init__(
            name="web_automation",
            description=(
                "Tự động hóa bất kỳ trang web nào bằng ngôn ngữ tự nhiên. Đầu vào là một chuỗi JSON, gồm hai trường bắt buộc: "
                "`url` (trang khởi đầu) và `goal` (mô tả nhiệm vụ rõ ràng, cụ thể). "
                "Trường tùy chọn: `stealth` (boolean, dành cho các trang có bảo vệ chống bot), "
                "`country` (US/GB/CA/DE/FR/JP/AU, dùng cho định tuyến địa lý). "
                "Trả về JSON có cấu trúc mà tác tử trích xuất được, hoặc mô tả lỗi."
            ),
        )
        self.client = TinyFish(
            api_key=api_key or os.environ["TINYFISH_API_KEY"],
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        # ToolRegistry sẽ gói văn bản đầu vào của ReAct thành {"input": "..."}
        raw = parameters.get("input", "")
        try:
            params = json.loads(raw)
        except json.JSONDecodeError:
            return json.dumps(
                {"error": "Đầu vào phải là chuỗi JSON hợp lệ"},
                ensure_ascii=False,
            )

        url = params.get("url")
        goal = params.get("goal")
        if not url or not goal:
            return json.dumps(
                {"error": "Thiếu trường bắt buộc url hoặc goal"},
                ensure_ascii=False,
            )

        kwargs: Dict[str, Any] = {"url": url, "goal": goal}
        if params.get("stealth"):
            kwargs["browser_profile"] = BrowserProfile.STEALTH
        if (country := params.get("country")):
            kwargs["proxy_config"] = ProxyConfig(
                enabled=True,
                country_code=ProxyCountryCode(country),
            )

        # Dùng run đồng bộ — vòng lặp ReAct cần lấy được kết quả rồi mới tiếp tục.
        # Với nhiệm vụ dài có thể chuyển sang dùng queue + polling.
        run = self.client.agent.run(**kwargs)

        if run.status.value != "COMPLETED" or run.result is None:
            err = run.error.message if run.error else "Thất bại không rõ nguyên nhân"
            return json.dumps(
                {"error": err, "status": run.status.value},
                ensure_ascii=False,
            )

        return json.dumps(
            {"data": run.result, "run_id": run.run_id},
            ensure_ascii=False,
        )

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="input",
                type="string",
                description=(
                    "Chuỗi JSON, các trường: url (bắt buộc), goal (bắt buộc), "
                    "stealth (tùy chọn), country (tùy chọn)"
                ),
                required=True,
            )
        ]
```

Có vài chỗ trong code đáng đặc biệt lưu ý:

1. **`description` của công cụ là thông tin duy nhất mà LLM nhìn thấy khi quyết định có gọi nó hay không**. Hãy viết mô tả như thể đang dặn dò một lập trình viên junior chưa từng đọc code của bạn — nói rõ đầu vào, đầu ra, khi nào sử dụng.
2. **Công cụ luôn trả về chuỗi**. ReAct là văn bản vào, văn bản ra. Chúng ta tuần tự hóa (serialize) kết quả thành JSON, để tác tử có thể tiếp tục suy luận trong "Thought" ở bước tiếp theo.
3. **`stealth` và `country` không được bật mặc định mà là tham số tùy chọn**. Hãy để LLM tự quyết định — thông qua mô tả công cụ để cho nó biết khi nào nên bật.

#### Bước 2: Kết nối công cụ vào tác tử ReAct

```python
# main.py
from hello_agents import ReActAgent, HelloAgentsLLM, ToolRegistry
from tools.tinyfish_tool import TinyFishWebTool

llm = HelloAgentsLLM()  # Đọc cấu hình provider từ .env
registry = ToolRegistry()
registry.register_tool(TinyFishWebTool())

agent = ReActAgent(
    agent_name="research_assistant",
    llm=llm,
    tool_registry=registry,
)

result = agent.run(
    "Tra giá hiện tại của iPhone 17 Pro trên cửa hàng chính thức của Apple và trên JD.com, "
    "sau khi tính đến phí vận chuyển được ghi trên trang sản phẩm thì cho tôi biết bên nào rẻ hơn."
)
print(result)
```

Khi chạy, vòng lặp ReAct đại khái sẽ trải qua quá trình như sau:

1. **Thought (Suy nghĩ)**: "Tôi cần lấy giá từ hai trang web khác nhau. Tôi nên gọi web_automation hai lần."
2. **Action (Hành động)**: `web_automation({"url": "https://www.apple.com/.../iphone-17-pro", "goal": "Trích xuất giá khởi điểm của iPhone 17 Pro. Trả về JSON: {price_cny: number, free_shipping: boolean}"})`
3. **Observation (Quan sát)**: `{"data": {"price_cny": 9999, "free_shipping": true}}`
4. **Thought**: "Bây giờ lấy giá JD. JD có chống bot — tôi nên bật stealth."
5. **Action**: `web_automation({"url": "https://item.jd.com/...", "goal": "...", "stealth": true})`
6. **Observation**: `{"data": {"price_cny": 9799, "free_shipping": true}}`
7. **Thought**: "Cả hai bên đều miễn phí vận chuyển. JD rẻ hơn 200 tệ."
8. **Final Answer**: "Hiện tại JD rẻ hơn cửa hàng chính thức của Apple 200 tệ: JD ¥9.799, Apple chính thức ¥9.999, cả hai đều miễn phí vận chuyển."

Những gì vừa xảy ra chính là cốt lõi mà chương 7 muốn truyền tải: **dịch vụ bên ngoài bước vào framework của chính bạn với tư cách công dân hạng nhất**. Vòng lặp ReAct không thay đổi, LLM cũng không thay đổi. Chúng ta chỉ thêm một công cụ, và tác tử của bạn đã có năng lực hành động trên mạng công khai. Đây chính là **khả năng kết hợp (composability)**.

#### Bước 3: Nâng nó lên cấp production

Phiên bản ở trên là đủ để demo. Môi trường production thực tế còn cần bổ sung thêm vài thứ:

**1. Xác thực nội dung, đừng chỉ nhìn trạng thái**

Một lần chạy `COMPLETED` cũng có thể trả về rác, nếu tác tử đụng phải chặn mềm (soft block) (trang thử thách Cloudflare, captcha, render "Truy cập bị từ chối" thành nội dung chính). Luôn luôn phải kiểm tra **nội dung kết quả**:

```python
def is_real_success(result: dict | None) -> bool:
    if not result:
        return False
    s = json.dumps(result, ensure_ascii=False).lower()
    failure_signals = ["captcha", "blocked", "access denied", "could not", "unable to"]
    return not any(signal in s for signal in failure_signals)
```

**2. Cache được thì cache**

Một lần tự động hóa web 30 giây là tốn kém. Nếu trong cùng một phiên, tác tử ReAct yêu cầu cùng một URL hai lần, thì nên trả về kết quả đã cache. (Hệ thống bộ nhớ ở chương 8 là một điểm phù hợp để bắt tay vào.)

**3. Thiết lập timeout**

Bên trong TinyFish, mỗi lần chạy có timeout 10 phút, nhưng công cụ của bạn nên thất bại sớm hơn — phần lớn các nhiệm vụ có ý nghĩa hoàn thành trong vòng 10–60 giây. Vượt quá khoảng thời gian này, nhiều khả năng là đang bị kẹt ở trang thử thách.

**4. Dùng luồng phát trực tiếp để quan sát**

Ghi lại `streaming_url` của mỗi lần chạy, ghi vào log. Khi môi trường production gặp sự cố, bản ghi hình lần chạy là công cụ nhanh nhất để định vị lỗi.

### 2.5 Đối phó với chống bot

Sớm hay muộn tác tử của bạn cũng sẽ đụng phải một trang web trả về kết quả rỗng hoặc 403. Đây là quy trình chẩn đoán:

**Bước 1 — Xác nhận vấn đề đúng là chống bot**. Mở `streaming_url` của lần chạy thất bại, tìm các đặc điểm sau:

| Bạn nhìn thấy gì | Nguyên nhân có khả năng cao |
| --- | --- |
| Trang "Checking your browser" của Cloudflare | Phát hiện bot của Cloudflare |
| Popup hoặc chuyển hướng của DataDome | DataDome |
| Trang trắng hoặc xoay vòng mãi mãi | Chặn dựa trên IP hoặc dấu vân tay |
| Captcha (reCAPTCHA, hCaptcha) | Captcha — giới hạn cứng |
| "Access Denied" / 403 | Chặn IP hoặc User-Agent |
| Đáng lẽ thấy nội dung lại xuất hiện tường đăng nhập | Chống bot dựa trên phiên |

**Bước 2 — Bật cả ẩn danh và proxy cùng lúc**. Ẩn danh thay đổi dấu vân tay trình duyệt, proxy thay đổi IP. Các nhà cung cấp chống bot sẽ liên kết hai tín hiệu này — chỉ đổi một cái thường là không đủ:

```python
run = client.agent.run(
    url="https://protected-site.com",
    goal="Trích xuất giá sản phẩm",
    browser_profile=BrowserProfile.STEALTH,
    proxy_config=ProxyConfig(enabled=True, country_code=ProxyCountryCode.US),
)
```

**Bước 3 — Để tác tử hành xử giống con người hơn**. Một số trang web để ý đến hành vi chứ không chỉ dấu vân tay. Hãy thêm vào goal:

- Đóng banner cookie một cách tường minh
- Đợi trang tải xong rồi mới trích xuất
- Mô tả phần tử bằng thị giác ("nút 'Thêm vào giỏ hàng' màu xanh"), đừng dùng selector
- Với quy trình nhiều bước, dùng các bước được đánh số, để tác tử tự chậm lại

**Bước 4 — Thực sự không được thì đổi cách đánh**. Một số trang web chặn crawler đến cùng, nhưng lại rộng rãi cung cấp RSS, sitemap, API công khai. Dành năm phút lục lọi một chút, tiết kiệm được vài ngày thử-sai. Nếu dữ liệu thực sự chỉ nằm trong các trang được định dạng sau tường phí (paywall), hãy tự hỏi: liệu nguồn thông tin gốc (thông báo, thông cáo báo chí, trang của nhà cung cấp) có xuất hiện ở nơi phòng thủ yếu hơn hay không.

Một ví dụ hoàn chỉnh đã được gia cố:

```python
from tinyfish import (
    TinyFish, BrowserProfile, ProxyConfig, ProxyCountryCode,
    CompleteEvent, RunStatus,
)

client = TinyFish()

with client.agent.stream(
    url="https://protected-site.com/pricing",
    browser_profile=BrowserProfile.STEALTH,
    proxy_config=ProxyConfig(enabled=True, country_code=ProxyCountryCode.US),
    goal="""
        1. Đợi trang tải hoàn toàn.
        2. Đóng mọi banner đồng ý cookie / GDPR.
        3. Đợi 1 giây trước khi tiếp tục.
        4. Định vị khu vực bảng giá (thường là dạng lưới hoặc bảng thẻ).
        5. Với mỗi gói, trích xuất: tên gói, phí hàng tháng, phí hàng năm (nếu có).

        Nếu xuất hiện trang Cloudflare hoặc trang kiểm tra bảo mật, đợi nó tự động vượt qua.
        Nếu thấy trang Access Denied hoặc CAPTCHA, trả về {"error": "blocked"}.
        Không nhấp vào bất kỳ nút mua hàng hoặc thanh toán nào.

        Trả về JSON: [{"plan": "Pro", "monthly_price": 49, "annual_price": 39}]
    """,
    on_streaming_url=lambda e: print(f"Xem trực tiếp: {e.streaming_url}"),
    on_progress=lambda e: print(f"  → {e.purpose}"),
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            if event.status == RunStatus.COMPLETED:
                print("Kết quả:", event.result_json)
            else:
                print("Thất bại:", event.error.message if event.error else "unknown")
```

**Tuyên bố trung thực quan trọng**: bao gồm cả TinyFish, không có bất kỳ Web Agent nào có thể giải các captcha hiện đại (reCAPTCHA v2/v3, hCaptcha) một cách đáng tin cậy. Nếu một trang web bật captcha với bạn, thì hiện tại đó là một bức tường cứng. Cách làm đúng là thiết kế lần chạy của bạn sao cho không kích hoạt nó, chứ không phải cố đối đầu.

### 2.6 Phần thêm: Gọi TinyFish thông qua MCP (tương ứng với chương 10)

Chương 10 đã giới thiệu **MCP (Model Context Protocol)** — giao thức mở do Anthropic đề xuất, cho phép trợ lý AI kết nối với công cụ và dữ liệu bên ngoài. TinyFish cung cấp một MCP server. Điều này có nghĩa là bạn có thể khiến bất kỳ trợ lý nào tương thích MCP (Claude Desktop, Cursor, Windsurf, Claude Code) có năng lực điều khiển trình duyệt — mà bản thân bạn không cần viết một dòng code nào.

Cài đặt bằng một dòng lệnh:

```bash
# Claude Code
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client claude-code

# Claude Desktop
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client claude

# Cursor
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client cursor
```

Hoặc thêm thủ công vào `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tinyfish": {
      "url": "https://agent.tinyfish.ai/mcp"
    }
  }
}
```

Sau khi khởi động lại, trợ lý của bạn sẽ có thêm các công cụ như `run_web_automation`, `search`, `fetch_content`, `create_browser_session`, v.v. Sau đó bạn có thể nói trực tiếp với Claude:

> "Thu thập song song trang bảng giá của 5 đối thủ cạnh tranh sau đây, sắp xếp kết quả thành bảng."

Claude sẽ dùng TinyFish để hoàn thành việc này ở phía sau, thực thi hàng loạt, phản hồi tiến trình theo thời gian thực.

Đây chính là phần thưởng **xây dựng một lần, dùng được ở mọi nơi** mà chương 10 đã báo trước. MCP với vai trò là giao thức phân phối, biến mỗi công cụ thành năng lực mà **mọi trợ lý AI đều có thể sử dụng trực tiếp**.

---

## Phần ba: Suy ngẫm tổng hợp

### 3.1 Khi nào **không nên** dùng Web Agent được quản lý

Để trung thực: Web Agent được quản lý không phải lúc nào cũng là lựa chọn đúng.

- **Có API chính thức** — dùng API. API nhanh hơn, rẻ hơn, đáng tin cậy hơn
- **Crawl tần suất cao các trang không có chống bot** — Playwright thuần là đủ, rẻ hơn vài bậc độ lớn
- **Các workflow then chốt có cái giá của lỗi cao** — bất kể bạn dùng công cụ gì, luôn luôn phải thêm điểm kiểm tra có con người trong vòng lặp ở các bước then chốt
- **Công cụ nội bộ mà bạn hoàn toàn kiểm soát** — đơn giản là thêm một endpoint API

Khi nào nên dùng Web Agent được quản lý: trang web mục tiêu ở bên ngoài, không có API, có bảo vệ chống bot, quy trình nhiều bước, **và thời gian kỹ thuật đáng giá hơn đơn giá tính theo nhiệm vụ**.

### 3.2 Hướng đi tiếp theo của Web Agent

Lĩnh vực này phát triển rất nhanh. Vài điều đáng chú ý trong 12–24 tháng tới:

- **Các mô hình nhỏ hơn, nhanh hơn, được tinh chỉnh chuyên biệt cho Web**. Dùng mô hình tiên tiến cho việc "nhấp vào nút màu xanh" là dùng dao mổ trâu giết gà. Thế hệ tiếp theo của các tác tử chuyên gia OS — ví dụ như các mô hình sau Qwen-VL-2.5-VL — sẽ giảm mạnh độ trễ và chi phí của mỗi bước hành động.
- **Phiên đã xác thực trở thành công dân hạng nhất**. Tích hợp Vault (cầu nối với trình quản lý mật khẩu được mã hóa) bắt đầu được triển khai. Điều này sẽ mở khóa làn sóng ứng dụng tác tử tiếp theo — ngân hàng, y tế, công cụ nội bộ, những kịch bản mà đăng nhập là lối vào.
- **Benchmark học thuật tiến gần đến mức của production**. Tỷ lệ thành công production trên các luồng nghiệp vụ phổ biến đã ổn định ở mức khoảng 90%; điểm số trên các benchmark đối kháng như WebArena cũng đang tăng nhanh, khoảng cách đang bị thu hẹp đồng thời từ cả hai phía kỹ thuật và mô hình.
- **Web Agent cục bộ (local)**. Các nhiệm vụ nhạy cảm về quyền riêng tư (hồ sơ y tế, thuế) cần tác tử chạy cục bộ. VLM cục bộ kết hợp với Chromium cục bộ trông đã khả thi.
- **Đánh giá Web Agent trưởng thành hơn**. Phương pháp luận đánh giá được giảng ở chương 12 vừa hay có thể áp dụng — sẽ sớm xuất hiện các bộ đánh giá tiêu chuẩn hóa dành cho Web Agent production.

### 3.3 Tiếp theo nên đi đâu

Nếu chương này khơi gợi hứng thú của bạn, dưới đây là vài con đường có thể tiếp tục:

- **Kết hợp với chương 8 (Bộ nhớ)**: để Web Agent của bạn nhớ hôm qua đã thu thập được gì, hôm nay chỉ lấy phần gia tăng
- **Kết hợp với chương 12 (Đánh giá)**: trang bị cho Web Agent của bạn khả năng theo dõi tỷ lệ thành công. Bạn sẽ nhanh chóng nắm rõ trang nào cần stealth, goal nào cần tinh chỉnh thêm
- **Kết hợp với chương 13 và chương 14**: trợ lý du lịch và tác tử DeepResearch, một khi mở rộng không gian hành động ra "trình duyệt thật", sẽ có sự thay đổi về chất. Hãy thử dùng Web Agent làm backend để viết lại một trong số chúng
- **Các giải pháp thay thế mã nguồn mở có thể nghiên cứu**: Browser-Use (Python), Skyvern (Python), WebVoyager (benchmark nghiên cứu), AgentE (hạ tầng tác tử mã nguồn mở)
- **Benchmark**: WebArena, VisualWebArena, Mind2Web, WebVoyager-2024 — đọc qua các bài báo một lượt, bạn sẽ hiểu rõ Web Agent năm 2026 đang ở mức nào

---

## Lời kết

Suy cho cùng, Web Agent chẳng qua chỉ là một vòng lặp ReAct — chỉ có điều bề mặt hành động của nó là trình duyệt. Mọi thứ bạn đã học ở các chương trước — cảm nhận, suy luận, hành động, bộ nhớ, đánh giá — đều áp dụng trực tiếp được. Điều mới mẻ là **môi trường đối kháng** này: sự thù địch của các trang web thật đối với tự động hóa, thứ mà App di động và phần mềm desktop không có, và chính sự thù địch này là lý do kỹ thuật Web Agent trở thành một ngành học độc lập.

Tin tốt là: bạn không cần phải giải quyết tất cả những điều này từ con số không. Những primitive (thành phần nguyên thủy) cấp production như TinyFish tồn tại chính là vì mục đích này. Là một người xây dựng tác tử AI-native, công việc của bạn là **kết hợp** những primitive này thành thứ gì đó hữu ích — và hoàn thành việc đó trong một framework mà chính bạn **hiểu rõ từ đầu đến cuối**.

Hãy bắt tay vào xây dựng. Web mở là bề mặt hành động lớn nhất thế giới, và chưa bao giờ có thời điểm nào tốt hơn bây giờ để dạy một tác tử cách sử dụng nó.

---

*Lời cảm ơn: Cảm ơn đội ngũ TinyFish đã đóng góp ý kiến cho các chi tiết kỹ thuật của chương này. Tất cả chi tiết API đều được đối chiếu từ [tài liệu chính thức của TinyFish](https://docs.tinyfish.ai).*
