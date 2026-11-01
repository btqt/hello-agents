# 05. Cách viết một Skill tốt

> Skill là gì? Làm thế nào để viết skill tốt?
> Chúng ta sẽ đi theo hướng thiết kế của skill-creator để tìm ra câu trả lời.
> Mục tiêu của bài viết này là: đọc xong, bạn sẽ hiểu được thực hành tốt nhất (best practices) để viết skill.

---

![Mục lục](./images/Extra08-figures/toc.png)

## I. Skill là gì?

### 1.1 Định nghĩa

Skill là một thư mục, bên trong chứa tài liệu chỉ dẫn, tài liệu tham khảo, script có thể thực thi và các tài nguyên khác. AI khi có được nó, sẽ có thể đảm nhận một công việc cụ thể mà trước đây nó chưa biết làm.

Ví dụ, trong một thư mục kỹ năng `pdf-editor`, có thể có một tài liệu chỉ dẫn "cách xử lý PDF", một script Python để xoay PDF, một tài liệu tham khảo API — AI không cần tìm kiếm thêm bất cứ thứ gì từ bên ngoài, mọi thứ đều có trong thư mục này.

Khái niệm này không giới hạn ở một sản phẩm cụ thể nào. Dù là Codex, Claude hay các AI Agent khác, bản chất của skill đều giống nhau. Bạn có thể hiểu nó như một **plugin năng lực** của AI — cắm vào, AI có thêm một sở trường; rút ra, AI vẫn là trợ lý tổng quát như ban đầu.

### 1.2 Hình thái tối giản

Một skill tối thiểu chỉ cần một file duy nhất:

```
my-skill/
└── SKILL.md
```

Cấu trúc của `SKILL.md` rất đơn giản — nửa trên cho AI biết "khi nào dùng tôi", nửa dưới cho AI biết "làm cụ thể như thế nào":

```yaml
---
name: my-skill                    # ← Nửa trên: metadata
description: >-                   #    AI dựa vào đây để quyết định có kích hoạt kỹ năng này không
  Khi người dùng cần làm một việc gì đó, sử dụng kỹ năng này.
---

Nửa dưới: chỉ dẫn thao tác                   # ← AI chỉ đọc đến đây sau khi kỹ năng được kích hoạt
Thực hiện theo các bước sau...
```

Nửa trên gọi là **frontmatter** (YAML nằm giữa `---`), chứa hai trường `name` và `description`. AI sẽ quét frontmatter của tất cả các kỹ năng đã cài đặt ở đầu mỗi cuộc hội thoại, dựa vào description để phán đoán "kỹ năng này có liên quan tới yêu cầu hiện tại không" — đây là **căn cứ duy nhất** để kỹ năng được kích hoạt.

Nửa dưới gọi là **body** (nội dung chính Markdown), là chỉ dẫn thao tác chỉ được nạp sau khi kỹ năng được kích hoạt. Nếu kỹ năng không được kích hoạt, AI sẽ không bao giờ đọc đến phần này.

### 1.3 Cấu trúc hoàn chỉnh

Khi một kỹ năng trở nên phức tạp, chỉ dựa vào một file SKILL.md thôi là không đủ.

Ví dụ bạn muốn làm một kỹ năng "xử lý PDF": SKILL.md viết quy trình xử lý, nhưng code xoay PDF thì lần nào cũng giống nhau, mỗi lần để AI viết lại vừa tốn thời gian vừa có thể sai sót — chi bằng đặt trực tiếp một script Python đã viết sẵn. Lại ví dụ kỹ năng "trình tạo dự án frontend": mỗi lần đều cần một bộ file mẫu HTML/React, chi bằng đặt sẵn một thư mục template để AI copy ra rồi sửa.

Vì vậy, thư mục skill hoàn chỉnh có thể chứa các thành phần sau:

```
skill-name/
├── SKILL.md                  # [Bắt buộc] File vào: frontmatter + body
├── agents/
│   └── openai.yaml           # [Khuyến nghị] "Danh thiếp" của kỹ năng
├── scripts/                  # [Tùy chọn] Script có thể thực thi
├── references/               # [Tùy chọn] Tài liệu tham khảo
└── assets/                   # [Tùy chọn] Mẫu sản phẩm đầu ra
```

Giải thích từng mục:

- **SKILL.md** — File duy nhất bắt buộc, đã giới thiệu ở trên

- **scripts/** — Chương trình đã viết sẵn, AI không cần hiểu nó, chỉ cần gọi shell để thực thi là được. Ví dụ `scripts/rotate_pdf.py`, AI chỉ cần chạy `python rotate_pdf.py input.pdf 90` là có thể xoay PDF, không cần mỗi lần viết lại logic xoay. Phù hợp với những thao tác **mà kết quả phải chính xác, không thể để AI tự do phát huy**

- **references/** — Tài liệu tham khảo mà AI cần tra cứu trong quá trình làm việc. Ví dụ một kỹ năng "truy vấn BigQuery", AI cần biết công ty có những bảng nào, mỗi bảng có trường gì, những thông tin này đặt trong `references/schema.md`, AI sẽ đọc khi cần. Điểm khác biệt với scripts là: references là để AI **đọc**, scripts là để AI **thực thi**

- **assets/** — Không phải để AI xem, mà là file được dùng trực tiếp trong sản phẩm đầu ra cuối cùng. Ví dụ một kỹ năng "trình tạo dự án frontend", `assets/frontend-template/` chứa một bộ code mẫu HTML/React, AI copy trực tiếp bộ template này ra rồi sửa trên đó. Lại ví dụ `assets/logo.png` là logo công ty, AI khi sinh trang web sẽ tham chiếu trực tiếp nó. AI không cần "hiểu" một bức ảnh logo, chỉ cần biết nó ở đâu, khi nào thì đặt vào

- **agents/openai.yaml** — "Danh thiếp" của kỹ năng. Nhiều sản phẩm AI sẽ hiển thị một danh sách kỹ năng trên giao diện, để người dùng chọn hoặc tìm kiếm. File này lưu tên, giới thiệu, icon... hiển thị trong danh sách đó. Nó không ảnh hưởng tới hành vi của AI, hoàn toàn chỉ phục vụ giao diện sản phẩm

---

## II. Bạn đang viết chỉ dẫn cho người, hay cho AI?

Biết được skill là gì, bước tiếp theo là viết một cái. Nhưng hầu hết mọi người khi viết skill lần đầu đều mắc phải cùng một vấn đề.

Hãy xem một ví dụ. Giả sử bạn muốn làm một kỹ năng "review code", bạn có thể viết như sau:

```markdown
---
name: code-review
description: Kỹ năng review code
---

# Code Review Skill

## Bối cảnh
Kỹ năng này được đúc kết từ nhiều năm kinh nghiệm review code của đội ngũ, nhằm nâng cao chất lượng code và hiệu quả hợp tác của đội nhóm.

## Nguyên tắc review
- Giữ giọng điệu chuyên nghiệp, mang tính xây dựng
- Quan tâm đến chất lượng code hơn là phong cách cá nhân
- Cân bằng giữa sự nghiêm khắc và linh hoạt

## Cách sử dụng
Khi người dùng nộp code, thực hiện review toàn diện cho code, đưa ra đề xuất cải thiện. Chú ý giữ thái độ thân thiện và khích lệ.

## Lịch sử phiên bản
- v1.0: Phiên bản ban đầu
- v1.1: Thêm hỗ trợ cho Python
```

Nếu đây là một tài liệu đội nhóm viết cho người đọc, nó được viết khá tốt — có bối cảnh, có nguyên tắc, có cách sử dụng, thậm chí còn có lịch sử phiên bản.

Nhưng người đọc skill là AI. Dùng góc nhìn này để xem xét lại:

- **"Đúc kết từ nhiều năm kinh nghiệm đội ngũ"** — AI không quan tâm kỹ năng này đến từ đâu, nó chỉ cần biết **bây giờ nên làm gì**
- **"Giữ giọng điệu chuyên nghiệp, mang tính xây dựng"** — con người đọc có thể cảm nhận được một cảm giác đại khái, nhưng AI sẽ triển khai "chuyên nghiệp" và "mang tính xây dựng" thành vô số tổ hợp, mỗi lần đầu ra một kiểu khác nhau
- **"Cân bằng giữa sự nghiêm khắc và linh hoạt"** — người review giàu kinh nghiệm biết khi nào nên nghiêm khắc khi nào nên linh hoạt, nhưng AI không có trực giác này, câu này coi như không nói gì
- **"Review toàn diện, đưa ra đề xuất cải thiện"** — đây là kỳ vọng dành cho người review là con người, nhưng AI cần cái gì: kiểm tra cái gì trước? kiểm tra cái gì sau? vấn đề nào bắt buộc phải chỉ ra? vấn đề nào có thể bỏ qua?
- **"Lịch sử phiên bản"** — mỗi lần AI được đánh thức đều là hoàn toàn mới, v1.0 hay v1.1 không có ý nghĩa gì với nó
- **description chỉ viết "Kỹ năng review code"** — AI dựa vào description để phán đoán có kích hoạt hay không, năm chữ "kỹ năng review code" quá mơ hồ: người dùng nói "giúp tôi xem đoạn code này" có kích hoạt không? "hàm này hiệu năng thế nào" có kích hoạt không?

Mỗi điều nhìn riêng lẻ đều không phải là "sai", nhưng chúng đều là viết cho người đọc. **Vấn đề không nằm ở việc viết chưa đủ nhiều, mà nằm ở việc viết sai đối tượng.**

Vậy cách viết đúng là như thế nào? Hãy xem một câu trả lời có sẵn — skill-creator của Codex. Nó là một "skill để tạo ra skill", bản thân SKILL.md của nó chính là một thực hành tốt nhất về "cách viết chỉ dẫn cho AI".

---

## III. Khung tổng thể của skill-creator

Mở SKILL.md của skill-creator (khoảng 370 dòng), trước khi đi sâu vào bất kỳ chi tiết nào, hãy xây dựng nhận thức tổng thể về nó trước.

skill-creator chỉ giải quyết một vấn đề duy nhất: **làm thế nào để đưa ra chỉ dẫn hiệu quả nhất cho AI trong một cửa sổ ngữ cảnh có hạn?**

Xoay quanh vấn đề này, nó đưa ra một hệ thống thiết kế hoàn chỉnh, có thể hiểu qua ba tầng.

### Tầng thứ nhất: Ràng buộc căn bản — Sự ngắn gọn

Cửa sổ ngữ cảnh của AI là có hạn, và được chia sẻ (system prompt, lịch sử hội thoại, metadata của tất cả các kỹ năng đã cài đặt đều nằm trong đó). Skill của bạn chiếm càng nhiều, phần dành cho các mục đích khác càng ít. Vì vậy nguyên tắc đầu tiên của skill-creator là: **mỗi câu chữ đều phải xứng đáng với lượng token nó chiếm dụng**.

### Tầng thứ hai: Hai chiều thiết kế

Trong ràng buộc "ngắn gọn" này, khi viết skill sẽ đối mặt với hai quyết định cốt lõi:

**Chiều thứ nhất: Thông tin nên đặt ở đâu?**

Không phải thông tin nào cũng cần nạp ngay từ đầu. skill-creator thiết kế một kiến trúc phân tầng ba cấp, giúp các thông tin khác nhau đi vào ngữ cảnh ở thời điểm khác nhau:

![Cấu trúc chuẩn của Skill và ba cấp nạp](./images/Extra08-figures/skill-structure.png)

- **L1 (Metadata)**: luôn ở trong ngữ cảnh, khoảng 100 từ — AI dựa vào đây để phán đoán có kích hoạt kỹ năng này không
- **L2 (SKILL.md body)**: nạp sau khi kích hoạt, giới hạn trong 5k từ — chỉ dẫn thao tác
- **L3 (scripts/references/assets)**: dùng khi cần, không giới hạn — trong đó scripts thực thi mà không cần đọc vào ngữ cảnh, chi phí token bằng không

Điều này giải quyết vấn đề "làm thế nào để chứa nhiều thông tin nhất với ít token nhất".

**Chiều thứ hai: Cho AI bao nhiêu mức độ tự do?**

Không phải tác vụ nào cũng phù hợp để AI tự do phát huy.

Ví dụ: để AI viết một bài blog kỹ thuật, mười người viết ra mười phong cách khác nhau đều được — bạn chỉ cần đưa ra hướng, cụ thể viết thế nào để AI tự quyết định. Đây là **mức tự do cao**.

Nhưng để AI sinh ra một file cấu hình YAML thì lại khác. Ví dụ `openai.yaml` mà skill-creator cần sinh ra, bên trong có trường `short_description`, yêu cầu 25-64 ký tự, chữ cái đầu viết hoa, không được có dấu ngoặc kép. AI viết thành 65 ký tự? Không được, giao diện sản phẩm sẽ cắt bớt. Viết thành 24 ký tự? Không được, không qua kiểm tra. Thiếu viết hoa chữ cái đầu? Giao diện hiển thị không đồng nhất. Loại tác vụ này chỉ cần sai một ký tự là có vấn đề, bạn không thể để AI tự do phát huy, phải dùng script để khóa chặt định dạng — đây là **mức tự do thấp**. Loại tác vụ này gọi là "thao tác dễ vỡ" (fragile operations): không phải nói nó phức tạp, mà nói **làm đúng chỉ có một cách, làm sai có một trăm cách**.

![Phổ mức độ tự do](./images/Extra08-figures/freedom-spectrum.png)

Điều này giải quyết vấn đề "làm thế nào để cân bằng giữa tính linh hoạt của AI và độ tin cậy của đầu ra".

### Tầng thứ ba: Quy trình triển khai thực tế

Có nguyên tắc và kiến trúc rồi, cuối cùng skill-creator đưa ra một quy trình tạo kỹ năng gồm sáu bước, biến tư tưởng thiết kế thành các bước thao tác có thể thực thi:

![Quy trình tạo sáu bước](./images/Extra08-figures/creation-flow.png)

Hiểu → Lập kế hoạch → Khởi tạo → Chỉnh sửa → Kiểm tra → Lặp lại. Trong đó script xuyên suốt toàn bộ quy trình, tạo thành chuỗi đảm bảo chất lượng mang tính xác định (deterministic):

![Quan hệ tương tác giữa các file](./images/Extra08-figures/file-interaction.png)

### Tổng quan khung

Quan hệ giữa ba tầng:

```
Ngắn gọn (ràng buộc căn bản)                         → Chương 4
 ├── Thông tin nên đặt ở đâu? → Kiến trúc phân tầng ba cấp        → Chương 5
 ├── Cho AI bao nhiêu tự do? → Phổ mức độ tự do và script  → Chương 6
 └── Triển khai thực tế như thế nào? → Quy trình tạo sáu bước            → Chương 7
```

Mỗi chương tiếp theo sẽ triển khai trong khung này.

---

## IV. Ràng buộc căn bản: Sự ngắn gọn
> Vị trí trong khung: Tầng thứ nhất

### 4.1 Ràng buộc cốt lõi

Cửa sổ ngữ cảnh của AI giống như một bàn làm việc — lượng tài liệu nó có thể trải ra cùng một lúc là có hạn. Mà trên chiếc bàn làm việc này đã có sẵn không ít thứ: quy tắc của hệ thống, những gì người dùng đã nói trước đó, tóm tắt của tất cả các kỹ năng đã cài đặt. Một khi skill của bạn được kích hoạt, nội dung của nó cũng phải trải ra trên đó. Bàn làm việc chỉ có bấy nhiêu đó chỗ, bạn chiếm càng nhiều, không gian dành cho những thứ khác càng ít.

Vì vậy skill-creator đã viết điều này thành nguyên tắc đầu tiên:

> The context window is a public good. Skills share the context window with everything else Codex needs: system prompt, conversation history, other Skills' metadata, and the actual user request.

Đã là không gian bàn làm việc có hạn, vậy khi viết skill, làm sao để phán đoán một đoạn nội dung có nên đưa vào hay không? skill-creator đưa ra một giả định tiền đề: **bản thân AI đã rất thông minh rồi, bạn chỉ cần bổ sung những gì nó chưa biết.**

> Default assumption: Codex is already very smart. Only add context Codex doesn't already have.

Dựa trên giả định này, trước khi viết mỗi đoạn nội dung, hãy tự hỏi hai câu hỏi:

- "AI có biết điều này rồi không?" — ví dụ "vòng lặp for của Python viết thế nào", AI tất nhiên biết, không cần dạy
- "Đoạn nội dung này có đáng để chiếm không gian trên bàn làm việc không?" — một đoạn giải thích 200 chữ, có thể thay bằng một ví dụ code 10 dòng không?

**Hệ quả thực tiễn**: dùng ví dụ ngắn gọn thay cho giải thích dài dòng. Một ví dụ code tốt hơn ba đoạn mô tả bằng văn bản.

### 4.2 Những gì không nên đưa vào Skill?

Skill-creator liệt kê rõ ràng một **danh sách cấm**:

> A skill should only contain essential files that directly support its functionality. Do NOT create extraneous documentation or auxiliary files.

Các file không nên có:
- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md

> The skill should only contain the information needed for an AI agent to do the job at hand. It should not contain auxiliary context about the process that went into creating it, setup and testing procedures, user-facing documentation, etc. Creating additional documentation files just adds clutter and confusion.

Lý do rất đơn giản: người đọc skill là AI, không phải nhà phát triển con người. AI không cần những "tài liệu hỗ trợ con người" như hướng dẫn cài đặt, changelog, tham khảo nhanh. Mỗi file thừa đều là nhiễu (noise).

### 4.3 Khi viết ràng buộc, "không làm gì" chính xác hơn "làm gì"

Ngắn gọn không chỉ là "viết ít đi", mà còn bao gồm "viết đúng". Hãy xem một ví dụ.

Khi skill-creator tạo `laotou-thought-style` (một kỹ năng về phong cách viết), nó **không** viết:

```
Hãy viết với giọng điệu ấm áp, kiềm chế, sâu sắc.
```

Mô tả tích cực kiểu này nhìn có vẻ rõ ràng, nhưng đối với AI, mức độ "ấm áp", sự cân bằng giữa "kiềm chế" và "sâu sắc" — toàn là không gian mơ hồ.

Điều nó làm là viết một **danh sách phản mẫu (anti-patterns)** (`references/anti-patterns.md`):

| Không nên làm | Triệu chứng | Cách sửa |
|-----------|------|--------|
| Chồng chất nhân vật | Xuất hiện liên tiếp nhiều tên và lời thoại | Giữ lại một tình huống xung đột, bổ sung đúc kết trừu tượng |
| Chỉ có lời sáo rỗng không có hành động | Cả bài toàn "phải kiên trì, phải nỗ lực" | Đổi thành một bước nhỏ có thể làm ngay hôm nay |
| Đạo lý trực tiếp | Mở đầu đã nói quy luật | Trải bối cảnh cuộc sống trước |
| Kết thúc quá gắt | Kết "phải thay đổi!" | Đổi thành "từ từ thôi" "là được" |
| Tuyệt đối hóa quá mức | "mãi mãi" "chắc chắn" | Thêm từ hạn định "đa phần" "thường thì" |

**Mỗi điều đều cụ thể, có thể kiểm tra, có phương án sửa rõ ràng.**

Nguyên lý đằng sau:

```
"Làm gì" → Mô tả một miền khả thi vô hạn → AI đi lang thang ngẫu nhiên trong đó
"Không làm gì" → Vẽ biên giới trên miền khả thi → Không gian hành vi của AI được thu hẹp lại đúng phạm vi bạn muốn
```

Bản thân skill-creator cũng tuân theo nguyên tắc này — SKILL.md của nó dùng rất nhiều dung lượng để nói "không nên viết gì" (What to Not Include in a Skill), thay vì nói chung chung "viết nội dung tốt".

Khi bạn viết xong SKILL.md, hãy làm một "phép thử đảo ngược": mỗi chỉ dẫn tích cực, có thể viết lại thành dạng "không nên làm X" không? Nếu có thể, sau khi viết lại thường chính xác hơn.

### 4.4 Thống nhất dùng ngữ khí mệnh lệnh

skill-creator yêu cầu phần nội dung chính của SKILL.md thống nhất dùng **ngữ khí mệnh lệnh/dạng nguyên mẫu** (Always use imperative/infinitive form). Đây không phải sở thích về mặt thẩm mỹ, mà là để giảm sự mơ hồ — câu mệnh lệnh tự nhiên chính là chỉ dẫn.

---

## V. Chiều thiết kế thứ nhất: Thông tin nên đặt ở đâu?
> Vị trí trong khung: Tầng thứ hai — Chiều thứ nhất

Trong phần tổng quan khung ở Chương 3, chúng ta đã thấy toàn cảnh của kiến trúc phân tầng ba cấp. Chương này sẽ đi sâu vào chi tiết của nó.

### 5.1 Nạp dần theo ba cấp

Định nghĩa gốc của skill-creator về ba tầng:

> 1. **Metadata (name + description)** - Always in context (~100 words)
> 2. **SKILL.md body** - When skill triggers (<5k words)
> 3. **Bundled resources** - As needed by Codex (Unlimited because scripts can be executed without reading into context window)

| Tầng | Nội dung | Khi nào trong ngữ cảnh | Chi phí token |
|------|------|--------------|-----------|
| **L1** | frontmatter (name + description) | **Luôn luôn** | ~100 từ |
| **L2** | SKILL.md body | Nạp sau khi kích hoạt | <5k từ |
| **L3** | scripts/ references/ assets/ | Nạp theo nhu cầu | Không giới hạn |

**Về bản chất đây là một hệ thống quản lý entropy thông tin**:

- **L1 là bộ lọc** — chọn ra kỹ năng cần dùng hiện tại từ hàng chục kỹ năng đã cài đặt. description không chính xác → kích hoạt nhầm hoặc bỏ sót kích hoạt
- **L2 là sổ tay vận hành** — cho AI biết nên làm gì sau khi kích hoạt. Quá dài → sự chú ý bị pha loãng. body giới hạn trong 500 dòng
- **L3 là hộp công cụ** — chỉ mở khi cần. Trong đó scripts/ hiệu quả nhất — **thực thi mà không đọc vào**, chi phí token bằng không

### 5.2 Frontmatter: Toàn bộ nguồn gốc của cơ chế kích hoạt

Frontmatter chỉ có hai trường bắt buộc: `name` và `description`. Nhưng cách viết description cực kỳ quan trọng:

> This is the primary triggering mechanism for your skill, and helps Codex understand when to use the skill.

description của chính skill-creator được viết như sau:

```yaml
description: Guide for creating effective skills. This skill should be used when
  users want to create a new skill (or update an existing skill) that extends
  Codex's capabilities with specialized knowledge, workflows, or tool integrations.
```

Nó không chỉ nói "làm gì" (creating effective skills), mà còn nói "khi nào dùng" (when users want to create a new skill or update an existing skill).

**Quy tắc then chốt**:
- Đặt tất cả thông tin "khi nào dùng" vào description, **không đặt trong body**. body chỉ được nạp sau khi kích hoạt, lúc đó Codex đã quyết định dùng rồi, thông tin "khi nào dùng" đã muộn
- Không đặt các trường khác ngoài `name` và `description` trong frontmatter (trừ `license`, `allowed-tools`, `metadata`)

Một ví dụ description tốt (kỹ năng docx):

> "Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Codex needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

### 5.3 Sự khác biệt bản chất giữa bốn loại tài nguyên đóng gói

Hiểu được sự khác biệt giữa bốn loại tài nguyên này là chìa khóa để hiểu toàn bộ hệ thống skill:

#### Scripts (`scripts/`)

Code có thể thực thi (Python/Bash...), dùng cho các tác vụ cần **độ tin cậy xác định (deterministic)** hoặc phải viết lại nhiều lần.

- **Khi nào cần**: cùng một đoạn code phải viết lại mỗi lần, hoặc cần đầu ra đáng tin cậy có tính xác định
- **Ví dụ**: `scripts/rotate_pdf.py` dùng cho tác vụ xoay PDF
- **Ưu điểm cốt lõi**: hiệu quả về token, có tính xác định, có thể thực thi mà không cần đọc vào cửa sổ ngữ cảnh
- **Lưu ý**: đôi khi script vẫn cần được Codex đọc, dùng để vá lỗi hoặc thích ứng môi trường

#### References (`references/`)

Tài liệu và vật liệu tham khảo, được nạp vào ngữ cảnh khi cần, hỗ trợ quá trình tư duy của Codex.

- **Khi nào cần**: tài liệu chi tiết mà Codex cần tham khảo khi làm việc
- **Ví dụ**: `references/finance.md` (schema tài chính), `references/api_docs.md` (quy cách API), `references/policies.md` (chính sách công ty)
- **Công dụng**: schema cơ sở dữ liệu, tài liệu API, kiến thức chuyên môn, chính sách công ty, hướng dẫn quy trình chi tiết
- **Ưu điểm cốt lõi**: giữ cho SKILL.md tinh gọn, chỉ nạp khi Codex phán đoán là cần thiết
- **Thực hành tốt nhất**: nếu file rất lớn (>10k từ), hãy đưa mẫu tìm kiếm grep vào trong SKILL.md
- **Tránh trùng lặp**: thông tin chỉ nên tồn tại ở SKILL.md **hoặc** file references, không được có ở cả hai nơi. Thông tin chi tiết ưu tiên đặt ở references, SKILL.md chỉ giữ lại chỉ dẫn quy trình cốt lõi và hướng dẫn workflow

#### Assets (`assets/`)

Không phải file dùng để nạp vào ngữ cảnh, mà là tài nguyên được dùng trực tiếp trong sản phẩm đầu ra của Codex.

- **Khi nào cần**: file mà kỹ năng cần dùng trong đầu ra cuối cùng
- **Ví dụ**: `assets/logo.png` (vật liệu thương hiệu), `assets/slides.pptx` (template PPT), `assets/frontend-template/` (mẫu HTML/React), `assets/font.ttf` (font chữ)
- **Công dụng**: template, hình ảnh, icon, code mẫu, font chữ, tài liệu ví dụ — những thứ này sẽ được copy hoặc chỉnh sửa
- **Ưu điểm cốt lõi**: tách tài nguyên đầu ra khỏi tài liệu, Codex có thể dùng chúng mà không cần đọc vào ngữ cảnh

#### Metadata Agents (`agents/openai.yaml`) (khuyến nghị)

Metadata hướng tới UI, không phải để AI đọc, mà để frontend sản phẩm đọc:

- Chứa các trường như `display_name`, `short_description`, `default_prompt`
- Được sinh ra bằng script `generate_openai_yaml.py` một cách xác định, thay vì viết tay
- Sau khi cập nhật SKILL.md, cần kiểm tra `agents/openai.yaml` còn khớp không, nếu lỗi thời thì sinh lại
- Định nghĩa trường chi tiết xem tại `references/openai_yaml.md`

### 5.4 Ba mẫu thực chiến của tiết lộ dần dần

Skill-creator đưa ra ba mẫu cụ thể để tách nội dung vào references:

**Mẫu 1: Hướng dẫn cấp cao + file tham khảo**

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber:
[code example]

## Advanced features
- **Form filling**: See [FORMS.md](FORMS.md) for complete guide
- **API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
- **Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
```

Codex chỉ nạp FORMS.md, REFERENCE.md hoặc EXAMPLES.md khi cần.

**Mẫu 2: Tổ chức theo lĩnh vực**

Kỹ năng đa lĩnh vực/đa biến thể, tách theo lĩnh vực để tránh nạp nội dung không liên quan:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

Khi người dùng hỏi về chỉ số bán hàng, Codex chỉ đọc `sales.md`.

Tương tự áp dụng cho kịch bản đa framework/đa biến thể:

```
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md (AWS deployment patterns)
    ├── gcp.md (GCP deployment patterns)
    └── azure.md (Azure deployment patterns)
```

**Mẫu 3: Chi tiết có điều kiện**

Hiển thị trực tiếp chức năng cơ bản, liên kết theo nhu cầu cho chức năng nâng cao:

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents
For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

### 5.5 Hai hướng dẫn tránh bẫy quan trọng

1. **Tránh tham chiếu lồng nhau quá sâu** — tất cả file reference nên được liên kết trực tiếp từ SKILL.md, không nên theo kiểu lồng A → B → C
2. **File dài cần thêm mục lục** — file reference vượt quá 100 dòng cần thêm TOC ở đầu, giúp Codex xem trước toàn cảnh

### 5.6 Các lỗi sai tầng thường gặp

| Lỗi | Hậu quả | Cách sửa |
|------|------|------|
| Điều kiện kích hoạt đặt trong body | body chỉ nạp sau khi kích hoạt, đã muộn | Đặt trong frontmatter description |
| "When to Use This Skill" viết trong body | Tương tự, Codex đã quyết định dùng mới thấy | Chuyển sang description |
| Nhồi chi tiết tham khảo vào SKILL.md | body phình to, mật độ thông tin giảm | Tách ra references/, body chỉ giữ liên kết tham chiếu |
| Thao tác có tính xác định viết thành chỉ dẫn văn bản | AI phải hiểu lại mỗi lần, có thể sai sót | Đóng gói thành scripts/, thực thi không đọc vào |
| references tham chiếu lẫn nhau | AI cần nhiều bước nhảy để lấy thông tin | Tất cả references liên kết trực tiếp từ SKILL.md |
| Nội dung SKILL.md và references trùng lặp | Lãng phí token, cập nhật có thể không đồng nhất | Thông tin chỉ tồn tại ở một nơi |

---

## VI. Chiều thiết kế thứ hai: Cho AI bao nhiêu mức độ tự do?
> Vị trí trong khung: Tầng thứ hai — Chiều thứ hai

Biết được thông tin nên đặt ở đâu, nên ràng buộc thế nào, câu hỏi tiếp theo là: **AI làm gì, script làm gì?**

AI rất giỏi hiểu ngữ nghĩa, sinh văn bản, làm công việc sáng tạo. Nhưng nó không giỏi kiểm soát định dạng chính xác, ràng buộc độ dài, quy tắc đặt tên — những "thao tác dễ vỡ" này.

### 6.1 Ba mức độ tự do

Skill-creator dùng một **phổ mức độ tự do** để xử lý sự không đồng đều này (xem sơ đồ khung ở Chương 3):

> Think of Codex as exploring a path: a narrow bridge with cliffs needs specific guardrails (low freedom), while an open field allows many routes (high freedom).

**Tự do cao** (chỉ dẫn văn bản): khi nhiều phương pháp đều khả thi, quyết định phụ thuộc vào ngữ cảnh, dùng heuristic để dẫn dắt.

**Tự do trung bình** (giả mã/script có tham số): có best practice nhưng cho phép biến đổi, cấu hình ảnh hưởng tới hành vi.

**Tự do thấp** (script cụ thể, ít tham số): thao tác dễ vỡ dễ sai sót, tính nhất quán là tối quan trọng, phải tuân theo trình tự cụ thể.

Logic cốt lõi:

```
Tác vụ càng dễ vỡ (dễ sai) → Tự do càng thấp → Dùng script khóa chặt
Tác vụ càng linh hoạt (nhiều phương án đều đúng) → Tự do càng cao → Dùng văn bản dẫn dắt
```

### 6.2 Phân bổ mức độ tự do của chính skill-creator

| Tác vụ | Mức tự do | Cách triển khai |
|------|--------|---------|
| Hiểu nhu cầu người dùng và đặt câu hỏi | Cao | Chỉ dẫn văn bản trong SKILL.md |
| Lập kế hoạch cấu trúc nội dung kỹ năng | Trung bình | Template + đề xuất mẫu dạng trắc nghiệm |
| Khởi tạo cấu trúc thư mục | **Thấp** | Script `init_skill.py` |
| Sinh openai.yaml | **Thấp** | Script `generate_openai_yaml.py` |
| Viết nội dung SKILL.md | Cao | Hướng dẫn nguyên tắc + gợi ý viết |
| Kiểm tra kết quả cuối cùng | **Thấp** | Script `quick_validate.py` |

### 6.3 Hai hướng sai lầm

**Sai lầm 1: Cho tác vụ dễ vỡ quá nhiều tự do**

```markdown
# Sai
Hãy sinh một file openai.yaml, bao gồm display_name và short_description.

# Hậu quả: short_description có thể vượt quá giới hạn 64 ký tự, viết hoa/thường có thể không nhất quán
```

Cách làm của Skill-creator: dùng script `generate_openai_yaml.py` để khóa chặt định dạng. AI chỉ cung cấp giá trị tham số, script đảm bảo đầu ra đúng quy cách.

**Sai lầm 2: Cho tác vụ sáng tạo quá nhiều ràng buộc**

```markdown
# Sai
Đoạn đầu tiên phải bắt đầu bằng "hôm qua", đoạn thứ hai phải chứa "về bản chất", đoạn cuối cùng phải kết bằng "từ từ thôi".

# Hậu quả: văn bản sinh ra giống như trò chơi điền từ
```

Cách làm của Skill-creator: đưa ra tỷ lệ cấu trúc (tầng bối cảnh ≤30%, tầng nguyên lý 30-40%), nhưng không khóa chặt cách dùng từ cụ thể.

### 6.4 Tiêu chí phán đoán

Hai câu hỏi:
1. **Làm sai thì hậu quả nghiêm trọng đến mức nào?** — càng nghiêm trọng → mức tự do càng thấp
2. **Có bao nhiêu cách làm "đúng"?** — càng nhiều → mức tự do càng cao

### 6.5 Triển khai mức tự do thấp: Ba script của skill-creator

Hiểu được phổ mức độ tự do, sẽ hiểu được vì sao skill-creator có ba script — chúng chính là sự triển khai cụ thể của "mức tự do thấp" (quan hệ tương tác giữa các script xem sơ đồ khung Chương 3).

**`init_skill.py` (Đảm bảo đầu vào, 398 dòng)**

Công cụ scaffold khởi tạo thư mục kỹ năng mới, tương tự `create-react-app` đối với dự án React:

```bash
scripts/init_skill.py <skill-name> --path <output-directory> \
  [--resources scripts,references,assets] [--examples] \
  [--interface key=value]
```

Chức năng cốt lõi:
- Tạo thư mục kỹ năng
- Sinh template SKILL.md có placeholder TODO (TODO là "bài tập điền chỗ trống" dành cho Codex)
- Gọi `generate_openai_yaml.py` để sinh `agents/openai.yaml` (truyền display_name, short_description, default_prompt do AI sinh ra qua `--interface key=value`)
- Có thể tùy chọn tạo các thư mục con `scripts/`, `references/`, `assets/`
- Có thể tùy chọn thêm file ví dụ (`--examples`)
- Tích hợp sẵn `normalize_skill_name()` tự động chuẩn hóa bất kỳ đầu vào nào của người dùng thành hyphen-case

Ví dụ sử dụng:
```bash
scripts/init_skill.py my-skill --path skills/public
scripts/init_skill.py my-skill --path skills/public --resources scripts,references
scripts/init_skill.py my-skill --path skills/public --resources scripts --examples
```

**`generate_openai_yaml.py` (Đảm bảo định dạng, 226 dòng)**

Chuyên trách sinh và cập nhật `agents/openai.yaml`:

- Đọc tên kỹ năng từ frontmatter của SKILL.md
- Tự động chuyển hyphen-case thành Title Case (`my-cool-skill` → `My Cool Skill`)
- Tích hợp sẵn từ điển viết tắt (GH, MCP, API... giữ nguyên viết hoa) và từ điển thương hiệu (openai → OpenAI)
- Tự động sinh `short_description` dài 25-64 ký tự
- Hỗ trợ `--interface key=value` để ghi đè bất kỳ trường nào

```bash
scripts/generate_openai_yaml.py <path/to/skill-folder> --interface key=value
```

**`quick_validate.py` (Đảm bảo đầu ra, 102 dòng)**

"Nhân viên kiểm định chất lượng" sau khi tạo kỹ năng:

```bash
scripts/quick_validate.py <path/to/skill-folder>
```

Nội dung kiểm tra:
- SKILL.md có tồn tại không
- Định dạng YAML frontmatter có hợp lệ không
- `name`: có phải hyphen-case không, ≤ 64 ký tự, không có dấu gạch nối liên tiếp/ở đầu-cuối
- `description`: có tồn tại không, không có dấu ngoặc nhọn, ≤ 1024 ký tự
- Chỉ cho phép 5 khóa frontmatter: `name`, `description`, `license`, `allowed-tools`, `metadata`

### 6.6 Chuỗi đảm bảo chất lượng

Ba script tạo thành một **chuỗi đảm bảo tính xác định**, kẹp lấy bước sáng tạo ở giữa:

```
init_skill.py (Đảm bảo đầu vào)
  Chuẩn hóa tên + tạo cấu trúc thư mục + sinh template
  → Đảm bảo điểm bắt đầu đúng
       ↓
  AI viết sáng tạo (mức tự do cao)
  → Nội dung SKILL.md, references, scripts tùy chỉnh
       ↓
quick_validate.py (Đảm bảo đầu ra)
  Kiểm tra định dạng frontmatter + quy tắc đặt tên + ràng buộc độ dài
  → Đảm bảo điểm kết thúc đúng quy cách
```

Góc nhìn then chốt: script là "thực thi mà không đọc vào" — **chi phí token bằng không**. Bạn có thể đóng gói bất kỳ logic xác định phức tạp nào vào script, mà không lo nó chiếm dụng ngữ cảnh. Đây là lý do vì sao skill-creator giao toàn bộ những thao tác nhỏ nhặt nhưng dễ vỡ như chuyển đổi tên (từ điển viết tắt, từ điển thương hiệu), ràng buộc độ dài (25-64 ký tự), kiểm tra định dạng cho script.

### 6.7 Cái gì nên đóng gói thành script?

```
Kết quả thực thi mỗi lần phải giống nhau      → Script
Liên quan đến ràng buộc định dạng/độ dài chính xác     → Script
Liên quan đến chuyển đổi quy tắc đặt tên          → Script
Cần khớp quy tắc kiểm tra          → Script
Cùng một đoạn code phải viết lại mỗi lần   → Script

Cần hiểu ngữ cảnh            → Chỉ dẫn văn bản
Có nhiều cách làm hợp lý            → Chỉ dẫn văn bản
Cần phán đoán mang tính sáng tạo            → Chỉ dẫn văn bản
```

Đôi khi script vẫn cần được Codex đọc (dùng để vá lỗi hoặc thích ứng môi trường), nhưng phần lớn thời gian chúng là "thực thi mà không đọc vào".

---

## VII. Triển khai thực tế: Quy trình tạo sáu bước
> Vị trí trong khung: Tầng thứ ba

Có các nguyên tắc và kiến trúc ở trên rồi, cuối cùng skill-creator đưa ra một quy trình tạo kỹ năng gồm sáu bước, biến tư tưởng thiết kế thành các bước thao tác có thể thực thi (xem sơ đồ khung Chương 3).

### 7.0 Quy tắc đặt tên

Trước khi bắt đầu, hãy xác định tên trước:

- Chỉ dùng chữ thường, số và dấu gạch nối; chuẩn hóa tên do người dùng cung cấp thành hyphen-case (ví dụ "Plan Mode" → `plan-mode`)
- Tên ≤ 64 ký tự
- Ưu tiên dùng cụm từ ngắn gọn, bắt đầu bằng động từ để mô tả hành động
- Khi cần, dùng tên công cụ làm namespace (như `gh-address-comments`, `linear-address-issue`)
- Tên thư mục kỹ năng hoàn toàn giống với tên kỹ năng

### 7.1 Bước 1: Hiểu kỹ năng — Xây dựng sự đồng thuận bằng ví dụ cụ thể

> Skip this step only when the skill's usage patterns are already clearly understood.

Để tạo ra một skill hiệu quả, trước tiên phải hiểu rõ **các ví dụ sử dụng cụ thể**. Những hiểu biết này có thể đến từ ví dụ do người dùng cung cấp, cũng có thể đến từ ví dụ được sinh ra và đã qua xác nhận của người dùng.

Lấy ví dụ xây dựng kỹ năng image-editor, có thể hỏi người dùng:

- "Kỹ năng image-editor nên hỗ trợ chức năng gì? Chỉnh sửa, xoay ảnh, còn gì khác nữa?"
- "Có thể cho vài ví dụ sử dụng kỹ năng này không?"
- "Tôi có thể nghĩ ra người dùng sẽ nói 'xóa mắt đỏ trong ảnh này' hoặc 'xoay ảnh này'. Còn cách sử dụng nào khác không?"
- "Người dùng sẽ nói gì để kích hoạt kỹ năng này?"

**Lưu ý**: đừng hỏi quá nhiều câu hỏi cùng lúc. Hỏi câu quan trọng nhất trước, sau đó hỏi thêm theo nhu cầu.

**Dấu hiệu hoàn thành**: đã có nhận thức rõ ràng về những chức năng mà kỹ năng nên hỗ trợ.

### 7.2 Bước 2: Lập kế hoạch nội dung kỹ năng có thể tái sử dụng

Với mỗi ví dụ cụ thể, làm hai phân tích:
1. Nếu làm việc này từ đầu, cần những gì?
2. Trong đó phần nào sẽ được dùng lặp lại?

Những thứ dùng lặp lại → đóng gói thành scripts/references/assets.

skill-creator đưa ra ba trường hợp phân tích điển hình:

**Trường hợp 1: Kỹ năng `pdf-editor`** (người dùng hỏi "giúp tôi xoay PDF này")
- Xoay PDF mỗi lần đều phải viết lại cùng một đoạn code
- → Đóng gói thành `scripts/rotate_pdf.py`

**Trường hợp 2: Kỹ năng `frontend-webapp-builder`** (người dùng hỏi "giúp tôi làm một todo app" hoặc "làm một dashboard theo dõi số bước chân")
- Viết webapp frontend mỗi lần đều cần cùng một bộ code mẫu HTML/React
- → Đóng gói thành thư mục template `assets/hello-world/`

**Trường hợp 3: Kỹ năng `big-query`** (người dùng hỏi "hôm nay có bao nhiêu người dùng đăng nhập?")
- Truy vấn BigQuery mỗi lần đều phải khám phá lại schema và quan hệ của các bảng
- → Đóng gói thành `references/schema.md`

**Dấu hiệu hoàn thành**: đã liệt kê danh sách tất cả các tài nguyên có thể tái sử dụng cần đưa vào (scripts, references, assets).

### 7.3 Bước 3: Khởi tạo kỹ năng

> When creating a new skill from scratch, always run the `init_skill.py` script.

Ở đây dùng từ "always" — không phải "khuyến nghị", mà là "luôn luôn". Lý do:
- Cấu trúc thư mục do script sinh ra đảm bảo đúng quy cách
- TODO trong template đảm bảo không bỏ sót trường bắt buộc
- Ràng buộc định dạng của `agents/openai.yaml` (độ dài trường, quy tắc dấu ngoặc kép) nếu viết tay rất dễ sai

Đây là **ứng dụng trực tiếp của nguyên tắc mức tự do thấp**: khởi tạo là một thao tác dễ vỡ, dùng script để loại bỏ khả năng sai sót.

Sau khi khởi tạo:
- Tùy chỉnh SKILL.md và thêm tài nguyên theo nhu cầu
- Nếu đã dùng `--examples`, thay thế hoặc xóa các file placeholder

### 7.4 Bước 4: Chỉnh sửa kỹ năng

Đây là bước cốt lõi nhất, chia thành hai giai đoạn:

#### Giai đoạn 1: Triển khai tài nguyên có thể tái sử dụng trước

Bắt đầu từ các tài nguyên đã lập kế hoạch ở Bước 2: triển khai các file `scripts/`, `references/`, `assets/`.

Lưu ý:
- Bước này có thể cần đầu vào từ người dùng (ví dụ kỹ năng `brand-guidelines` cần người dùng cung cấp vật liệu thương hiệu)
- Script mới thêm vào **phải được kiểm thử bằng cách chạy thực tế**, đảm bảo không có bug và đầu ra đúng như mong đợi
- Nếu có nhiều script tương tự nhau, chỉ cần kiểm thử một mẫu đại diện để tạo niềm tin
- Nếu đã dùng `--examples`, xóa các file placeholder không cần thiết. Chỉ tạo các thư mục tài nguyên thực sự cần thiết

#### Giai đoạn 2: Cập nhật SKILL.md

**Cách viết Frontmatter**:

```yaml
---
name: skill-name
description: >-
  Mô tả kỹ năng làm gì + cụ thể khi nào dùng.
  Đặt tất cả thông tin "khi nào dùng" ở đây, không đặt trong body.
---
```

**Cách viết Body**:

Viết chỉ dẫn thao tác cho một thực thể Codex khác. Bao gồm những thông tin hữu ích cho Codex nhưng không hiển nhiên: kiến thức quy trình, chi tiết chuyên môn, cách dùng tài nguyên có thể tái sử dụng.

Thống nhất dùng **ngữ khí mệnh lệnh/dạng nguyên mẫu**.

### 7.5 Bước 5: Kiểm tra kỹ năng

```bash
scripts/quick_validate.py <path/to/skill-folder>
```

Kiểm tra định dạng YAML frontmatter, trường bắt buộc, quy tắc đặt tên. Không qua thì sửa rồi chạy lại.

### 7.6 Bước 6: Lặp lại

> After testing the skill, users may request improvements. Often this happens right after using the skill, with fresh context of how the skill performed.

Quy trình lặp:
1. Sử dụng kỹ năng trên tác vụ thực tế
2. Phát hiện chỗ khó khăn hoặc kém hiệu quả
3. Tìm ra SKILL.md hoặc tài nguyên đóng gói nên cập nhật như thế nào
4. Thực hiện thay đổi và kiểm thử lại

Skill tốt không phải viết một lần là xong. Kỹ năng laotou-thought-style mà skill-creator tạo ra, sau lần sinh đầu tiên đã lặp lại `short_description` và `default_prompt` của `openai.yaml` — từ mô tả chung chung chuyển thành chỉ dẫn thao tác chính xác hơn.

---

## VIII. Tổng kết

Quay lại câu hỏi ban đầu: làm thế nào để viết một skill tốt?

Nhìn lại toàn bộ khung:

```
Ràng buộc căn bản: Sự ngắn gọn (Chương 4)
 ├── Thông tin nên đặt ở đâu? → Phân tầng ba cấp, nạp theo nhu cầu (Chương 5)
 ├── Cho AI bao nhiêu tự do? → Thao tác dễ vỡ dùng script khóa chặt, công việc sáng tạo dùng văn bản dẫn dắt (Chương 6)
 └── Triển khai thực tế như thế nào? → Quy trình sáu bước: Hiểu→Lập kế hoạch→Khởi tạo→Chỉnh sửa→Kiểm tra→Lặp lại (Chương 7)
```

**Skill là viết chỉ dẫn cho AI, không phải cho người. Dùng ít token nhất, ở đúng tầng, đưa ra ràng buộc chính xác nhất cho AI, để nó tự do phát huy trong giới hạn.**
</content>
