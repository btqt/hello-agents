# Agent Skills và MCP: Hai mô hình mở rộng năng lực cho Agent

## Mở đầu: Sau MCP, chúng ta còn cần gì nữa?

Ở Chương 10, chúng ta đã đi sâu tìm hiểu cách MCP (Model Context Protocol) giải quyết vấn đề kết nối giữa Agent và các công cụ bên ngoài thông qua một giao thức được chuẩn hóa. Bạn đã học được cách để Agent truy cập cơ sở dữ liệu, hệ thống file, dịch vụ API và nhiều tài nguyên khác thông qua MCP. Hãy cùng xem lại một kịch bản sử dụng MCP điển hình:

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="Trợ lý phân tích dữ liệu", llm=llm)

# Kết nối tới MCP server của cơ sở dữ liệu
db_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])
agent.add_tool(db_mcp)

# Giờ đây Agent đã có thể truy cập cơ sở dữ liệu
response = agent.run("Truy vấn 10 nhân viên có lương cao nhất trong bảng nhân viên")
```

Đoạn code này hoạt động rất tốt, Agent đã kết nối thành công tới cơ sở dữ liệu. Nhưng khi bạn thử xử lý những tác vụ phức tạp hơn, bạn sẽ phát hiện một số vấn đề tinh vi:

```python
# Một yêu cầu phức tạp hơn
response = agent.run("""
Phân tích xem ai trong công ty có tiếng nói (quyền lực) lớn nhất?
Cần xem xét tổng hợp:
1. Cấp bậc quản lý và số lượng cấp dưới
2. Mức lương và mức tăng lương
3. Thời gian làm việc và tính ổn định
4. Mức độ ảnh hưởng liên phòng ban
""")
```

Tác vụ này cần thực hiện nhiều lần truy vấn cơ sở dữ liệu, mỗi lần truy vấn kết quả sẽ ảnh hưởng tới chiến lược của lần truy vấn tiếp theo. Quan trọng hơn, nó đòi hỏi Agent phải có <strong>kiến thức chuyên môn (domain knowledge)</strong>: biết cách đo lường "tiếng nói", biết nên phân tích dữ liệu theo những chiều nào, biết cách kết hợp nhiều kết quả truy vấn để đưa ra kết luận.

Lúc này, bạn sẽ gặp phải hai vấn đề căn bản:

<strong>Vấn đề thứ nhất là bùng nổ ngữ cảnh (context explosion)</strong>. Để Agent có thể truy vấn cơ sở dữ liệu một cách linh hoạt, MCP server thường phơi bày (expose) hàng chục thậm chí hàng trăm công cụ (các bảng khác nhau, các phương thức truy vấn khác nhau). Toàn bộ JSON Schema đầy đủ của các công cụ này sẽ được nạp vào system prompt ngay khi kết nối được thiết lập, có thể chiếm tới hàng chục nghìn token. Theo phản hồi từ các nhà phát triển trong cộng đồng, chỉ riêng việc nạp một MCP server Playwright đã chiếm tới 8% cửa sổ ngữ cảnh 200k, và con số này sẽ nhanh chóng tích lũy qua nhiều lượt hội thoại, dẫn tới chi phí tăng vọt và khả năng suy luận suy giảm.

<strong>Vấn đề thứ hai là khoảng cách năng lực (capability gap)</strong>. MCP giải quyết vấn đề "có thể kết nối", nhưng không giải quyết vấn đề "biết cách sử dụng". Có khả năng kết nối cơ sở dữ liệu không đồng nghĩa với việc Agent biết cách viết câu lệnh SQL hiệu quả và an toàn; có thể truy cập hệ thống file không có nghĩa là nó hiểu được cấu trúc code và quy ước phát triển của một dự án cụ thể. Điều này giống như cấp quyền truy cập toàn bộ hệ thống cho một lập trình viên mới vào nghề, nhưng không cung cấp sổ tay vận hành và các thực hành tốt nhất (best practices).

Đây chính là vấn đề cốt lõi mà <strong>Agent Skills</strong> muốn giải quyết. Đầu năm 2025, sau khi ra mắt MCP, Anthropic tiếp tục đề xuất khái niệm Agent Skills, thu hút sự chú ý rộng rãi của giới chuyên môn. Có nhà phát triển bình luận rằng: "Skills và MCP là hai thứ khác nhau, Skills là kiến thức chuyên môn, cho mô hình biết nên làm như thế nào, về bản chất là một dạng Prompt nâng cao; còn MCP kết nối với các công cụ và dữ liệu bên ngoài." Cũng có người cho rằng: "Từ Function Call đến Tool Call đến MCP rồi đến Skills, bản chất cốt lõi không khác nhau nhiều, chỉ là sự tiến hóa tối ưu hóa về mặt kỹ thuật và hình thức thể hiện."

Vậy rốt cuộc Agent Skills là gì? Nó khác biệt về bản chất với MCP ở điểm nào? Hai bên là quan hệ cạnh tranh hay bổ trợ cho nhau? Chương này sẽ đi sâu khám phá những vấn đề này.

## Agent Skills là gì?

### Triết lý thiết kế cốt lõi

<strong>Agent Skills là một định dạng đóng gói tri thức quy trình (procedural knowledge) được chuẩn hóa</strong>. Nếu MCP cung cấp cho Agent "đôi tay" để thao tác với công cụ, thì Skills cung cấp "sổ tay vận hành" hay "SOP (quy trình thao tác chuẩn)", hướng dẫn Agent cách sử dụng các công cụ đó một cách đúng đắn.

Triết lý thiết kế này bắt nguồn từ một nhận định đơn giản nhưng sâu sắc: <strong>Khả năng kết nối (Connectivity) và năng lực (Capability) nên được tách biệt</strong>. MCP tập trung vào vế trước, Skills tập trung vào vế sau. Sự phân tách trách nhiệm này mang lại lợi thế kiến trúc rõ ràng:

- <strong>Trách nhiệm của MCP</strong>: Cung cấp giao diện truy cập được chuẩn hóa, giúp Agent có thể "với tới" dữ liệu và công cụ của thế giới bên ngoài
- <strong>Trách nhiệm của Skills</strong>: Cung cấp kiến thức chuyên môn, cho Agent biết trong tình huống cụ thể "nên phối hợp sử dụng các công cụ này như thế nào"

Dùng một phép ẩn dụ để hiểu rõ hơn: MCP giống như cổng USB hay driver thiết bị, nó định nghĩa cách thiết bị được kết nối; còn Skills giống như một ứng dụng phần mềm, nó định nghĩa cách sử dụng các thiết bị đã kết nối đó để hoàn thành một tác vụ cụ thể. Bạn có thể sở hữu một driver máy in đầy đủ chức năng (MCP), nhưng nếu không ai chỉ cho bạn cách thiết lập lề trang và in hai mặt trong Word (Skill), bạn vẫn không thể hoàn thành việc in ấn một cách hiệu quả.

### Tiết lộ dần dần (Progressive Disclosure): Hóa giải thế khó về ngữ cảnh

Đổi mới cốt lõi nhất của Agent Skills là <strong>cơ chế tiết lộ dần dần (Progressive Disclosure)</strong>. Cơ chế này chia thông tin về kỹ năng thành ba tầng, Agent nạp dần theo nhu cầu, vừa đảm bảo không bỏ sót chi tiết khi cần thiết, vừa tránh nhồi nhét quá nhiều nội dung vào cửa sổ ngữ cảnh cùng một lúc.

<div align="center">
  <img src="./images/Extra05-figures/image1.png" alt="渐进式披露三层架构" width="90%"/>
  <p>Hình 1 Kiến trúc ba tầng tiết lộ dần dần của Agent Skills</p>
</div>

#### Tầng thứ nhất: Metadata (Siêu dữ liệu)

Trong thiết kế của Skills, mỗi kỹ năng được lưu trong một thư mục độc lập, cốt lõi là một file Markdown tên `SKILL.md`. File này phải bắt đầu bằng một Frontmatter định dạng YAML, định nghĩa các thông tin cơ bản của kỹ năng.

Khi Agent khởi động, nó sẽ quét tất cả các thư mục kỹ năng đã cài đặt, <strong>chỉ đọc phần Frontmatter của mỗi `SKILL.md`</strong>, nạp các metadata này vào system prompt. Theo số liệu thực nghiệm, metadata của mỗi kỹ năng chỉ tiêu tốn khoảng <strong>100 token</strong>. Ngay cả khi bạn cài đặt 50 kỹ năng, mức tiêu tốn ngữ cảnh ban đầu cũng chỉ khoảng 5.000 token.

Điều này tương phản rõ rệt với cách thức hoạt động của MCP. Trong cách triển khai MCP điển hình, khi client kết nối tới một server, thông thường sẽ gửi yêu cầu `tools/list` để lấy toàn bộ JSON Schema đầy đủ của tất cả công cụ khả dụng, có thể ngay lập tức tiêu tốn hàng chục nghìn token.

#### Tầng thứ hai: Nội dung chính của kỹ năng (Instructions)

Khi Agent, thông qua phân tích yêu cầu của người dùng, phán đoán rằng một kỹ năng nào đó có liên quan cao tới tác vụ hiện tại, nó sẽ chuyển sang nạp tầng thứ hai. Lúc này, Agent sẽ đọc toàn bộ nội dung file `SKILL.md` của kỹ năng đó, nạp các chỉ dẫn chi tiết, lưu ý, ví dụ... vào ngữ cảnh.

Lúc này, Agent đã có đầy đủ ngữ cảnh cần thiết để hoàn thành tác vụ: cấu trúc cơ sở dữ liệu, mẫu truy vấn, các lưu ý... Mức tiêu tốn token của phần nội dung này tùy thuộc vào độ phức tạp của các chỉ dẫn, thông thường trong khoảng 1.000 đến 5.000 token.

#### Tầng thứ ba: Tài nguyên bổ sung (Scripts & References)

Đối với những kỹ năng phức tạp hơn, `SKILL.md` có thể tham chiếu tới các file khác trong cùng thư mục: script, file cấu hình, tài liệu tham khảo... Agent <strong>chỉ nạp các tài nguyên này khi cần thiết</strong>.

Ví dụ, cấu trúc file của một kỹ năng xử lý PDF có thể như sau:

```
skills/pdf-processing/
├── SKILL.md              # File kỹ năng chính
├── parse_pdf.py          # Script phân tích PDF
├── forms.md              # Hướng dẫn điền biểu mẫu (chỉ nạp khi có tác vụ điền form)
└── templates/            # File mẫu PDF
    ├── invoice.pdf
    └── report.pdf
```

Trong `SKILL.md`, có thể tham chiếu các tài nguyên bổ sung như sau:

- Khi cần thực hiện phân tích PDF, Agent sẽ chạy script `parse_pdf.py`
- Khi gặp tác vụ điền biểu mẫu, mới nạp `forms.md` để tìm hiểu các bước chi tiết
- File mẫu chỉ được truy cập khi cần tạo một tài liệu theo định dạng cụ thể

Thiết kế này có hai ưu điểm then chốt:

1. <strong>Dung lượng tri thức không giới hạn</strong>: Thông qua script và các file bên ngoài, kỹ năng có thể "mang theo" lượng kiến thức vượt xa giới hạn ngữ cảnh. Ví dụ, một kỹ năng phân tích dữ liệu có thể đính kèm một file dữ liệu 1GB và một script truy vấn, Agent truy cập dữ liệu bằng cách thực thi script, mà không cần nạp toàn bộ tập dữ liệu vào ngữ cảnh.

2. <strong>Thực thi xác định (deterministic)</strong>: Các tác vụ tính toán phức tạp, chuyển đổi dữ liệu, phân tích định dạng... được giao cho code thực thi, tránh được tính không chắc chắn và vấn đề ảo giác (hallucination) trong quá trình LLM sinh ra kết quả.

### Hiệu quả của tiết lộ dần dần: Từ 16k xuống 500 Token

Các trường hợp thực tiễn được chia sẻ bởi cộng đồng nhà phát triển đã chứng minh đầy đủ sức mạnh của tiết lộ dần dần. Trong một kịch bản thực tế:

- <strong>Cách tiếp cận MCP truyền thống</strong>: Kết nối trực tiếp tới một MCP server chứa số lượng lớn định nghĩa công cụ, mức nạp ban đầu tiêu tốn <strong>16.000 token</strong>
- <strong>Sau khi đóng gói bằng Skills</strong>: Tạo một Skill đơn giản đóng vai trò "cổng vào (gateway)", chỉ mô tả chức năng trong Frontmatter, mức tiêu tốn ban đầu chỉ <strong>500 token</strong>

Chỉ khi Agent xác định cần sử dụng kỹ năng đó, nó mới nạp các chỉ dẫn chi tiết và gọi công cụ MCP bên dưới theo nhu cầu. Kiến trúc này không chỉ giảm đáng kể chi phí ban đầu, mà còn giúp việc quản lý ngữ cảnh trong quá trình hội thoại trở nên chính xác và hiệu quả hơn.

## Agent Skills vs MCP: Khác biệt bản chất và mối quan hệ hợp tác

Bây giờ, chúng ta có thể so sánh một cách có hệ thống sự khác biệt bản chất giữa hai công nghệ này.

<div align="center">
  <img src="./images/Extra05-figures/image2.png" alt="MCP与Agent Skills设计哲学对比" width="90%"/>
  <p>Hình 2 So sánh triết lý thiết kế giữa MCP và Agent Skills</p>
</div>

### Hiểu sự khác biệt từ góc nhìn kỹ thuật

Hãy cùng tìm hiểu sự khác biệt này thông qua một ví dụ cụ thể. Giả sử bạn muốn xây dựng một Agent hỗ trợ nhóm thực hiện review code:

<strong>Trách nhiệm của MCP</strong>:

```python
# MCP cung cấp quyền truy cập chuẩn hóa tới GitHub
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])

# Các công cụ MCP phơi bày (ví dụ rút gọn):
# - list_pull_requests(repo, state)
# - get_pull_request_details(pr_number)
# - list_pr_comments(pr_number)
# - create_pr_comment(pr_number, body)
# - get_file_content(repo, path, ref)
# - list_pr_files(pr_number)
```

MCP giúp Agent "có thể" truy cập GitHub, có thể gọi các API này. Nhưng nó không biết "nên" làm gì.

<strong>Trách nhiệm của Skills</strong>:

```markdown
---
name: code-review-workflow
description: Thực hiện quy trình review code chuẩn, bao gồm kiểm tra phong cách code, vấn đề bảo mật, độ phủ test...
---

# Quy trình review code

## Danh sách kiểm tra

Khi thực hiện review code, hãy tiến hành theo các bước sau:

1. **Lấy thông tin PR**: Gọi `get_pull_request_details` để hiểu bối cảnh thay đổi
2. **Phân tích các file thay đổi**: Gọi `list_pr_files` để lấy danh sách file
3. **Review từng file**:
   - Với file `.py`: kiểm tra có tuân thủ PEP 8 không, có vấn đề hiệu năng rõ ràng không
   - Với file `.js/.ts`: kiểm tra có Promise chưa xử lý không, có dùng API đã lỗi thời (deprecated) không
   - Với file test: xác minh đã bao phủ (cover) các đường code mới thêm chưa
4. **Kiểm tra bảo mật**:
   - Có hardcode thông tin nhạy cảm không (khóa bí mật, mật khẩu)
   - Có rủi ro SQL injection hay XSS không
5. **Đưa ra phản hồi**:
   - Vấn đề nghiêm trọng: dùng `create_pr_comment` để bình luận trực tiếp
   - Đề xuất cải thiện: nêu trong phần tổng kết

## Quy định riêng của công ty

- Tất cả truy vấn cơ sở dữ liệu phải sử dụng tham số hóa (parameterized query)
- Các API endpoint phải có decorator xác thực quyền
- Tính năng mới phải kèm theo unit test (độ phủ > 80%)

## Mẫu bình luận ví dụ

**Vấn đề nghiêm trọng**:

⚠️ Rủi ro bảo mật: Dòng 45 nối chuỗi SQL trực tiếp, tồn tại rủi ro injection.
Đề xuất đổi sang truy vấn tham số hóa: `cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))`

```

Skills cho Agent biết "nên" làm gì, cách tổ chức quy trình review, cần chú ý đến những quy định riêng nào của công ty. Nó là nơi chứa đựng kiến thức chuyên môn và các best practices.

### Sự khác biệt bản chất trong chiến lược quản lý ngữ cảnh

<div align="center">
  <img src="./images/Extra05-figures/image3.png" alt="MCP急切加载 vs Skills惰性加载" width="90%"/>
  <p>Hình 3 So sánh nạp háo hức (eager loading) của MCP và nạp lười (lazy loading) của Skills</p>
</div>

### Bổ trợ chứ không cạnh tranh: Kiến trúc lai Skills + MCP

Sau khi hiểu rõ sự khác biệt giữa hai bên, chúng ta sẽ nhận ra: <strong>Skills và MCP không phải quan hệ cạnh tranh, mà là quan hệ bổ trợ lẫn nhau</strong>. Thực hành tốt nhất là kết hợp cả hai, tạo thành kiến trúc phân tầng:

<div align="center">
  <img src="./images/Extra05-figures/image4.png" alt="Skills + MCP 混合架构" width="90%"/>
  <p>Hình 4 Thiết kế kiến trúc lai Skills + MCP</p>
</div>
<strong>Quy trình làm việc điển hình</strong>:

1. Người dùng hỏi: "Phân tích xem ai trong công ty có tiếng nói lớn nhất"
2. <strong>Tầng Skills</strong> nhận diện đây là một tác vụ phân tích dữ liệu, nạp kỹ năng `mysql-employees-analysis`
3. <strong>Tầng Skills</strong> dựa vào chỉ dẫn của kỹ năng, chia tác vụ thành các bước con: truy vấn quan hệ quản lý, so sánh lương, thời gian làm việc...
4. <strong>Tầng MCP</strong> thực thi các truy vấn SQL cụ thể, trả về kết quả
5. <strong>Tầng Skills</strong> dựa vào kiến thức chuyên môn trong kỹ năng, diễn giải dữ liệu và tạo ra phân tích tổng hợp
6. Trả về câu trả lời có cấu trúc cho người dùng

Ưu điểm của kiến trúc này là:

- <strong>Tách biệt mối quan tâm (separation of concerns)</strong>: MCP tập trung vào "năng lực", Skills tập trung vào "trí tuệ"
- <strong>Tối ưu chi phí</strong>: Nạp dần dần giảm đáng kể mức tiêu tốn token
- <strong>Khả năng bảo trì</strong>: Logic nghiệp vụ (Skills) và hạ tầng (MCP) được tách rời (decoupled)
- <strong>Khả năng tái sử dụng</strong>: Cùng một MCP server có thể được nhiều Skills sử dụng

## Triển khai kỹ thuật: Cách tạo và sử dụng Skills

### Giải thích chi tiết quy cách SKILL.md

Hãy cùng tìm hiểu sâu cấu trúc chuẩn của file `SKILL.md`:

```markdown
---
# === Các trường bắt buộc ===
name: skill-name
  # Định danh duy nhất của kỹ năng, đặt tên theo kebab-case

description: >
  Mô tả ngắn gọn nhưng chính xác, nêu rõ:
  1. Kỹ năng này làm gì
  2. Khi nào nên sử dụng nó
  3. Giá trị cốt lõi của nó là gì
  # Lưu ý: description là căn cứ duy nhất để Agent lựa chọn kỹ năng, phải viết thật rõ ràng!

# === Các trường tùy chọn ===
version: 1.0.0
  # Số phiên bản theo ngữ nghĩa (semantic version)

allowed_tools: [tool1, tool2]
  # Danh sách công cụ mà kỹ năng này được phép gọi (whitelist)

required_context: [context_item1]
  # Thông tin ngữ cảnh mà kỹ năng này cần

license: MIT
  # Giấy phép

author: Your Name <email@example.com>
  # Thông tin tác giả

tags: [database, analysis, sql]
  # Thẻ (tag) để dễ phân loại và tìm kiếm
---

# Tiêu đề kỹ năng

## Tổng quan
(Giới thiệu chi tiết về kỹ năng, bao gồm kịch bản sử dụng, bối cảnh kỹ thuật...)

## Điều kiện tiên quyết
(Cấu hình môi trường, dependency... cần thiết để sử dụng kỹ năng này)

## Quy trình làm việc
(Hướng dẫn các bước chi tiết, cho Agent biết cách thực hiện tác vụ)

## Thực hành tốt nhất
(Tổng kết kinh nghiệm, lưu ý, các cạm bẫy thường gặp...)

## Ví dụ
(Các trường hợp sử dụng cụ thể, giúp Agent hiểu rõ hơn)

## Xử lý sự cố
(Các vấn đề thường gặp và giải pháp)
```

### Nguyên tắc viết Skills chất lượng cao

Dựa theo tài liệu chính thức của Anthropic và các thực hành tốt nhất từ cộng đồng, viết Skills hiệu quả cần tuân theo các nguyên tắc sau:

#### 1. Description chính xác

`description` là yếu tố then chốt cho việc ra quyết định của Agent. Nó cần:

- <strong>Định nghĩa chính xác phạm vi áp dụng</strong>: tránh các mô tả mơ hồ như "hỗ trợ xử lý dữ liệu"
- <strong>Chứa từ khóa kích hoạt</strong>: giúp Agent có thể khớp với ý định của người dùng
- <strong>Nêu rõ giá trị độc đáo</strong>: để phân biệt với các kỹ năng khác

❌ <strong>Description không tốt</strong>:
```yaml
description: Xử lý truy vấn cơ sở dữ liệu
```

✅ <strong>Description tốt</strong>:
```yaml
description: >
  Chuyển đổi câu hỏi nghiệp vụ tiếng Trung thành truy vấn SQL và phân tích cơ sở dữ liệu mẫu MySQL employees.
  Áp dụng cho các kịch bản truy vấn thông tin nhân viên, thống kê lương, phân tích phòng ban, lịch sử thay đổi chức vụ...
  Sử dụng kỹ năng này khi người dùng hỏi về dữ liệu liên quan tới nhân viên, lương, phòng ban.
```

#### 2. Mô-đun hóa và trách nhiệm đơn nhất

Một Skill nên tập trung vào một lĩnh vực hoặc loại tác vụ rõ ràng duy nhất. Nếu một Skill cố gắng làm quá nhiều việc, sẽ dẫn đến:

- Description quá rộng, độ chính xác khi khớp giảm
- Nội dung chỉ dẫn quá dài, lãng phí ngữ cảnh
- Khó bảo trì và cập nhật

<strong>Khuyến nghị</strong>: thay vì tạo một kỹ năng "phân tích dữ liệu tổng quát", hãy tạo nhiều kỹ năng chuyên biệt:
- `mysql-employees-analysis`: chuyên phân tích cơ sở dữ liệu employees
- `sales-data-analysis`: chuyên phân tích dữ liệu bán hàng
- `user-behavior-analysis`: chuyên phân tích hành vi người dùng

#### 3. Nguyên tắc ưu tiên tính xác định (deterministic)

Đối với các tác vụ phức tạp, cần được thực thi chính xác, ưu tiên sử dụng script thay vì dựa vào LLM để sinh ra. Ví dụ, trong kịch bản xuất dữ liệu Excel, thay vì để LLM sinh ra nội dung nhị phân của file Excel (dễ sai sót), tốt hơn nên viết một script chuyên dụng để xử lý tác vụ này, SKILL.md chỉ cần hướng dẫn Agent khi nào gọi script này là đủ.

#### 4. Chiến lược tiết lộ dần dần

Tận dụng hợp lý cấu trúc ba tầng, phân tầng thông tin theo mức độ quan trọng và tần suất sử dụng:

- <strong>Nội dung chính SKILL.md</strong>: đặt quy trình làm việc cốt lõi, các mẫu thường dùng
- <strong>Tài liệu bổ sung</strong> (như `advanced.md`): đặt cách dùng nâng cao, các trường hợp đặc biệt
- <strong>File dữ liệu</strong>: đặt dữ liệu tham khảo lớn, truy vấn theo nhu cầu thông qua script

### Trường hợp thực tiễn: Giải thích chi tiết Skill phân tích nhân viên MySQL

Hãy cùng tìm hiểu một ứng dụng cụ thể của Agent Skills thông qua một trường hợp thực tế từ cộng đồng Anthropic. Kỹ năng này dùng để phân tích cơ sở dữ liệu mẫu `employees` chính thức của MySQL.

#### Cấu trúc file kỹ năng

```
skills/mysql-employees-analysis/
├── SKILL.md          # File kỹ năng chính (chứa metadata và chỉ dẫn chi tiết)
└── db_schema.sql     # Tham khảo cấu trúc cơ sở dữ liệu (tùy chọn, nạp theo nhu cầu)
```

#### Ví dụ nội dung cốt lõi của SKILL.md

Frontmatter (tầng metadata) của kỹ năng này:

```markdown
---
name: mysql-employees-analysis
description: >
  Chuyển đổi câu hỏi nghiệp vụ tiếng Trung thành truy vấn SQL và phân tích cơ sở dữ liệu mẫu MySQL employees.
  Áp dụng cho truy vấn thông tin nhân viên (như "thông tin nhân viên mã số 12345"),
  thống kê lương (như "phòng ban có lương trung bình cao nhất"),
  phân tích phòng ban (như "phân bố số lượng nhân viên theo từng phòng ban"),
  lịch sử thay đổi chức vụ (như "lộ trình thăng tiến của một nhân viên nào đó")...
version: 1.0.0
allowed_tools: [execute_sql]
tags: [database, mysql, sql, employees, analysis]
---

# Kỹ năng phân tích cơ sở dữ liệu nhân viên MySQL

## Tổng quan

Kỹ năng này chuyên dùng để phân tích cơ sở dữ liệu mẫu `employees` do MySQL chính thức cung cấp.
Cơ sở dữ liệu này chứa khoảng 300.000 bản ghi nhân viên ảo, bao phủ dữ liệu từ năm 1985-2000.

**Năng lực cốt lõi**:
- Hiểu các câu hỏi nghiệp vụ bằng ngôn ngữ tự nhiên tiếng Trung
- Chuyển đổi thành truy vấn SQL hiệu quả
- Thực thi truy vấn và diễn giải kết quả
- Cung cấp góc nhìn và diễn giải nghiệp vụ về dữ liệu

## Cấu trúc cơ sở dữ liệu

### Cấu trúc bảng cốt lõi

| Tên bảng       | Mô tả         | Trường khóa                                                     |
| -------------- | ------------ | ------------------------------------------------------------ |
| `employees`    | Thông tin cơ bản nhân viên | emp_no, birth_date, first_name, last_name, gender, hire_date |
| `salaries`     | Lịch sử lương     | emp_no, salary, from_date, to_date                           |
| `titles`       | Lịch sử chức vụ     | emp_no, title, from_date, to_date                            |
| `dept_emp`     | Quan hệ nhân viên - phòng ban | emp_no, dept_no, from_date, to_date                          |
| `dept_manager` | Quản lý phòng ban     | emp_no, dept_no, from_date, to_date                          |
| `departments`  | Thông tin phòng ban  | dept_no, dept_name                                           |

### Quy ước then chốt

⚠️ **Quan trọng**: `to_date = '9999-01-01'` biểu thị bản ghi "đang có hiệu lực hiện tại".
Khi truy vấn trạng thái "hiện tại" (như nhân viên đương nhiệm, lương hiện tại), bắt buộc phải thêm điều kiện lọc này.

Tham khảo cấu trúc bảng đầy đủ tại: `db_schema.sql`

## Quy trình làm việc

### Bước 1: Hiểu yêu cầu

Phân tích kỹ mô tả tiếng Trung của người dùng, xác định:
- **Mục tiêu truy vấn**: cần truy vấn dữ liệu gì? (nhân viên, lương, phòng ban...)
- **Điều kiện lọc**: có hạn chế gì? (phòng ban cụ thể, khoảng thời gian, khoảng lương...)
- **Chiều thống kê**: có cần thống kê không? (trung bình, tổng số, xếp hạng...)
- **Phạm vi thời gian**: là dữ liệu lịch sử hay trạng thái hiện tại?

### Bước 2: Xây dựng câu lệnh SQL

Dựa theo yêu cầu, chọn mẫu truy vấn phù hợp (xem mục "Các mẫu truy vấn thường gặp" bên dưới).

**Nguyên tắc viết**:
1. Sử dụng alias bảng rõ ràng (như `e` cho employees)
2. Khi JOIN ưu tiên sử dụng khóa chính/khóa ngoại
3. Chú ý lọc theo ngày (đặc biệt là `to_date`)
4. Sử dụng hợp lý các trường có index
5. Tập kết quả lớn cần thêm LIMIT

### Bước 3: Thực thi truy vấn

Gọi công cụ `execute_sql` để thực thi câu lệnh SQL đã xây dựng.

```python
# Ví dụ gọi (Agent sẽ tự động chuyển đổi thành lời gọi công cụ)
result = execute_sql(query="SELECT ...")
```

### Bước 4: Diễn giải kết quả

Chuyển kết quả truy vấn thành câu trả lời bằng ngôn ngữ tự nhiên:
- Trình bày dữ liệu có cấu trúc bằng bảng
- Làm nổi bật các điểm dữ liệu then chốt
- Cung cấp góc nhìn nghiệp vụ (như xu hướng, bất thường)
- Nếu kết quả rỗng, giải thích nguyên nhân có thể

## Các mẫu truy vấn thường gặp

### Mẫu 1: Truy vấn thông tin cơ bản

```sql
-- Truy vấn thông tin cơ bản của nhân viên cụ thể
SELECT emp_no, CONCAT(first_name, ' ', last_name) AS full_name,
       gender, birth_date, hire_date
FROM employees
WHERE emp_no = <mã nhân viên>;
```

### Mẫu 2: Truy vấn trạng thái hiện tại

```sql
-- Truy vấn 10 nhân viên có lương hiện tại cao nhất
SELECT e.emp_no,
       CONCAT(e.first_name, ' ', e.last_name) AS name,
       s.salary
FROM employees e
JOIN salaries s ON e.emp_no = s.emp_no
WHERE s.to_date = '9999-01-01'  -- Lương hiện tại
ORDER BY s.salary DESC
LIMIT 10;
```

### Mẫu 3: Phân tích xu hướng lịch sử

```sql
-- Truy vấn lịch sử thay đổi lương của một nhân viên
SELECT emp_no, salary, from_date, to_date,
       salary - LAG(salary) OVER (ORDER BY from_date) AS increase
FROM salaries
WHERE emp_no = <mã nhân viên>
ORDER BY from_date;
```

### Mẫu 4: Truy vấn liên kết nhiều bảng

```sql
-- Truy vấn lương trung bình của từng phòng ban (hiện tại)
SELECT d.dept_name,
       COUNT(DISTINCT de.emp_no) AS emp_count,
       ROUND(AVG(s.salary), 2) AS avg_salary
FROM departments d
JOIN dept_emp de ON d.dept_no = de.dept_no
JOIN salaries s ON de.emp_no = s.emp_no
WHERE de.to_date = '9999-01-01'  -- Đang tại chức hiện tại
  AND s.to_date = '9999-01-01'   -- Lương hiện tại
GROUP BY d.dept_name
ORDER BY avg_salary DESC;
```

### Mẫu 5: Phân tích nghiệp vụ phức tạp

```sql
-- Phân tích "tiếng nói": tổng hợp cấp bậc quản lý, lương, thời gian làm việc
WITH manager_hierarchy AS (
    -- Thống kê số cấp dưới mà mỗi quản lý quản lý
    SELECT dm.emp_no, COUNT(de.emp_no) AS subordinate_count
    FROM dept_manager dm
    JOIN dept_emp de ON dm.dept_no = de.dept_no
    WHERE dm.to_date = '9999-01-01'
      AND de.to_date = '9999-01-01'
      AND de.emp_no != dm.emp_no
    GROUP BY dm.emp_no
),
current_salary AS (
    -- Lương hiện tại
    SELECT emp_no, salary
    FROM salaries
    WHERE to_date = '9999-01-01'
),
tenure AS (
    -- Thời gian làm việc (năm)
    SELECT emp_no,
           TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_employed
    FROM employees
)
SELECT e.emp_no,
       CONCAT(e.first_name, ' ', e.last_name) AS name,
       COALESCE(mh.subordinate_count, 0) AS team_size,
       cs.salary,
       t.years_employed,
       -- Điểm "tiếng nói" đơn giản (có thể điều chỉnh trọng số theo nghiệp vụ)
       (COALESCE(mh.subordinate_count, 0) * 10 +
        cs.salary / 1000 +
        t.years_employed * 5) AS influence_score
FROM employees e
JOIN current_salary cs ON e.emp_no = cs.emp_no
JOIN tenure t ON e.emp_no = t.emp_no
LEFT JOIN manager_hierarchy mh ON e.emp_no = mh.emp_no
WHERE cs.salary > 60000  -- Lọc nhân viên lương thấp
ORDER BY influence_score DESC
LIMIT 20;
```

## Lưu ý

### ⚠️ Xử lý đúng trường thời gian

- <strong>Trạng thái hiện tại</strong>: bắt buộc dùng điều kiện lọc `to_date = '9999-01-01'`
- <strong>Truy vấn lịch sử</strong>: chú ý phạm vi của `from_date` và `to_date`
- <strong>Tính toán thời gian</strong>: sử dụng các hàm `TIMESTAMPDIFF`, `DATEDIFF`...

### ⚠️ Tối ưu hiệu năng

- <strong>JOIN bảng lớn</strong>: ưu tiên dùng các trường có index (emp_no, dept_no)
- <strong>Truy vấn tổng hợp</strong>: sử dụng hợp lý GROUP BY và HAVING
- <strong>Giới hạn kết quả</strong>: với truy vấn phục vụ hiển thị, thêm giới hạn LIMIT
- <strong>Tối ưu truy vấn con</strong>: với truy vấn phức tạp dùng WITH (CTE) để tăng khả năng đọc và hiệu năng

### ⚠️ Chất lượng dữ liệu

- <strong>Xử lý giá trị NULL</strong>: sử dụng COALESCE hoặc IFNULL để xử lý giá trị rỗng
- <strong>Bản ghi trùng lặp</strong>: chú ý nhân viên có thể đổi vị trí nhiều lần, cần cân nhắc loại bỏ trùng lặp khi truy vấn
- <strong>Phạm vi dữ liệu</strong>: cơ sở dữ liệu chỉ chứa dữ liệu từ năm 1985-2000, cần chú ý giới hạn thời gian khi truy vấn

## Xử lý sự cố

<strong>Vấn đề 1: Kết quả truy vấn rỗng</strong>
- Kiểm tra đã sử dụng đúng `to_date = '9999-01-01'` chưa
- Xác minh mã nhân viên hoặc mã phòng ban có tồn tại không
- Kiểm tra phạm vi ngày tháng có hợp lý không

<strong>Vấn đề 2: Tốc độ truy vấn chậm</strong>
- Kiểm tra có thiếu điều kiện WHERE trên trường có index không
- Cân nhắc tách truy vấn phức tạp thành nhiều bước
- Sử dụng EXPLAIN để phân tích kế hoạch thực thi truy vấn

<strong>Vấn đề 3: Dữ liệu thống kê không chính xác</strong>
- Chú ý phân biệt trạng thái "lịch sử" và "hiện tại"
- Kiểm tra có bỏ sót điều kiện JOIN không
- Xác minh việc sử dụng hàm tổng hợp có đúng không
```

File SKILL.md này thể hiện cấu trúc của một kỹ năng hoàn chỉnh:
- Metadata rõ ràng (dùng để Agent khám phá và khớp)
- Mô tả đầy đủ cấu trúc cơ sở dữ liệu
- Hướng dẫn quy trình làm việc chi tiết
- Các ví dụ mẫu truy vấn phong phú (template SQL có thể tái sử dụng trực tiếp)
- Lưu ý và hướng dẫn xử lý sự cố thiết thực

#### Hiệu quả sử dụng kỹ năng

Khi người dùng hỏi một Agent hỗ trợ Agent Skills (như Claude Desktop, Claude Code):

**Câu hỏi của người dùng**:
> "Phân tích xem ai trong công ty có tiếng nói lớn nhất? Cần xem xét tổng hợp cấp bậc quản lý, mức lương và thời gian làm việc."

<div align="center">
  <img src="./images/Extra05-figures/image5.png" alt="Agent Skills工作流程" width="90%"/>
  <p>Hình 5 Minh họa quy trình làm việc đầy đủ của Agent Skills</p>
</div>

**Ví dụ kết quả đầu ra**:

| Xếp hạng | Mã NV | Tên                 | Quy mô đội | Lương    | Số năm làm việc | Điểm ảnh hưởng |
| ---- | ------ | -------------------- | -------- | ------- | -------- | ---------- |
| 1    | 110022 | Margareta Markovitch | 45       | 152.710 | 18       | 692,71     |
| 2    | 110039 | Vishwani Minakawa    | 38       | 138.273 | 16       | 598,27     |
| 3    | 110085 | Ebru Alpin           | 32       | 124.054 | 15       | 519,05     |

**Góc nhìn then chốt**:
1. Nhân viên có tiếng nói lớn nhất thường quản lý đội nhóm lớn (30+ người), lương thuộc top 1% (>120 nghìn), thời gian làm việc trên 15 năm
2. Mức độ ảnh hưởng của trưởng phòng ban vượt xa nhân viên thường, quy mô quản lý là yếu tố then chốt
3. Nhân viên làm việc lâu năm với lương cao ngay cả khi không giữ chức vụ quản lý vẫn có tiếng nói khá mạnh

Trong toàn bộ quá trình, kỹ năng đã cung cấp:
- **Kiến thức chuyên môn**: cách đo lường "tiếng nói" (quy mô quản lý + lương + thời gian làm việc)
- **Hướng dẫn kỹ thuật**: cách viết SQL hiệu quả (sử dụng CTE, hàm cửa sổ, JOIN nhiều bảng)
- **Hiểu biết nghiệp vụ**: cách diễn giải dữ liệu và tạo ra góc nhìn

### Chia sẻ và tái sử dụng Skills

Một đặc tính quan trọng khác của Agent Skills là **tính cộng đồng hóa**. Anthropic đã xây dựng kho lưu trữ Skills chính thức:

**Kho kỹ năng chính thức**: https://github.com/anthropics/skills

Tính đến năm 2025, đã có hàng trăm kỹ năng do cộng đồng đóng góp, bao phủ:

- **Công cụ phát triển**: thiết kế frontend, test API, review code, quy trình Git
- **Phân tích dữ liệu**: truy vấn SQL, trực quan hóa dữ liệu, phân tích thống kê
- **Xử lý tài liệu**: phân tích PDF, sinh Markdown, viết tài liệu kỹ thuật
- **Quy trình nghiệp vụ**: quản lý dự án, hỗ trợ khách hàng, kiểm tra tuân thủ

Sử dụng kỹ năng từ cộng đồng rất đơn giản:

```bash
# Clone kho kỹ năng chính thức
git clone https://github.com/anthropics/skills.git

# Copy kỹ năng cần dùng vào dự án của bạn
cp -r skills/frontend-design ./my-project/skills/

# Agent sẽ tự động phát hiện và nạp
```

Bạn cũng có thể chia sẻ kỹ năng của riêng mình:

```bash
# Đăng tải lên GitHub
cd my-custom-skill
git init
git add SKILL.md
git commit -m "Add custom SQL analysis skill"
git remote add origin https://github.com/yourname/my-skill.git
git push -u origin main

# Nhà phát triển khác có thể sử dụng trực tiếp
# git clone https://github.com/yourname/my-skill.git
```

## Động thái ngành và sự tiến hóa hệ sinh thái

### Tiến trình chuẩn hóa và sự hỗ trợ của các nhà cung cấp

Agent Skills dù do Anthropic đề xuất, nhưng triết lý thiết kế của nó đang ảnh hưởng tới toàn ngành.

**Anthropic Claude**:
- Claude Desktop và Claude Code hỗ trợ Skills ngay từ gốc (native)
- Cung cấp SDK và công cụ phát triển chính thức
- Duy trì kho kỹ năng chính thức

**Phản ứng của OpenAI**:
Mặc dù OpenAI chưa chính thức áp dụng thuật ngữ "Skills", nhưng trong bản cập nhật tháng 3 năm 2025, ChatGPT đã giới thiệu các khái niệm tương tự:

- **Tăng cường Custom Instructions**: hỗ trợ các chỉ dẫn nhiều bước phức tạp hơn
- **Memory và Context Profiles**: cho phép người dùng lưu và tái sử dụng kiến thức chuyên môn trong lĩnh vực cụ thể
- **Tính năng "kho kiến thức" của GPTs**: có thể đính kèm tài liệu và script, nạp theo nhu cầu

Về bản chất, các tính năng này là những hình thức triển khai khác nhau của triết lý Skills.

**Google Vertex AI**:
Google đã giới thiệu **"Grounding with Functions"** trong mô hình Gemini, cho phép nhà phát triển định nghĩa "gói hàm" (Function Packages), mỗi gói chứa:

- Định nghĩa hàm (tương tự tools của MCP)
- Hướng dẫn sử dụng (tương tự instructions của Skills)
- Ví dụ (examples)

Thiết kế này có độ tương đồng cao với kiến trúc lai Skills + MCP.

### Tính tất yếu của kiến trúc phân tầng

Tổng hợp các quan điểm, chúng tôi cho rằng: **Skills và MCP đại diện cho hai tầng tất yếu phải tách biệt trong kiến trúc Agent**. Khi độ phức tạp của hệ thống Agent tăng lên, việc phân tầng này là không thể tránh khỏi:

```
Tầng ứng dụng (Application Layer)
  ↓ Agent Skills
  ↓ Kiến thức chuyên môn, quy trình làm việc, thực hành tốt nhất

Tầng truyền tải (Transport Layer)
  ↓ MCP
  ↓ Giao diện chuẩn hóa, gọi công cụ, truy cập tài nguyên

Tầng hạ tầng (Infrastructure Layer)
  ↓ Cơ sở dữ liệu, API, hệ thống file, dịch vụ bên ngoài
```

Điều này hoàn toàn nhất quán với con đường tiến hóa của kiến trúc phần mềm truyền thống (từ monolithic đến phân tầng đến microservices), chỉ là được diễn giải lại một lần nữa trong lĩnh vực AI.

### Xu hướng chuẩn hóa

Khi ngành công nghiệp ngày càng coi trọng công nghệ Agent, chúng tôi dự đoán các xu hướng sau:

**1. Hợp nhất giao thức**

Trong tương lai có thể xuất hiện một giao thức mô tả năng lực Agent thống nhất, hợp nhất khả năng kết nối của MCP và cách biểu đạt tri thức của Skills:

```yaml
# Ví dụ giao thức thống nhất trong tương lai (giả định)
apiVersion: agent.io/v1
kind: Capability
metadata:
  name: enterprise-data-analysis
spec:
  transport:
    protocol: mcp
    server: database-mcp-server
    tools: [query, schema]
  knowledge:
    type: skill
    workflow: data-analysis-workflow.md
    examples: examples/
```

**2. Thị trường hóa và hệ sinh thái**

Tương tự NPM, PyPI, trong tương lai có thể xuất hiện hệ thống quản lý gói năng lực Agent:

```bash
# Lệnh giả định trong tương lai
agent-cli install @anthropic/frontend-design-skill
agent-cli install @google/data-analysis-suite
agent-cli install @openai/code-review-assistant
```

Nhà phát triển có thể phát hành, chia sẻ, bán Skills và MCP server của riêng mình, hình thành một hệ sinh thái sôi động.

**3. Tự động khám phá năng lực**

Agent có thể phát triển cơ chế tự động khám phá và học hỏi năng lực mới:

```python
# Agent trong tương lai có thể có khả năng tự học
agent = SelfEvolvingAgent()

# Agent phát hiện thiếu một năng lực nào đó khi thực hiện tác vụ
response = agent.run("Tạo file mô hình 3D")

# Agent tự động tìm kiếm và cài đặt Skill liên quan
# [Log nội bộ] Phát hiện loại tác vụ chưa biết: mô hình hóa 3D
# [Log nội bộ] Tìm kiếm kho kỹ năng... phát hiện skill "blender-3d-modeling"
# [Log nội bộ] Yêu cầu người dùng ủy quyền cài đặt... đã được ủy quyền
# [Log nội bộ] Cài đặt kỹ năng hoàn tất, thực thi lại tác vụ
```

### Thách thức và rủi ro

Đồng thời, chúng ta cũng cần cảnh giác với những rủi ro tiềm ẩn:

<strong>Thách thức về bảo mật</strong>:
- Skills chứa script có thể thực thi, tồn tại rủi ro code injection
- MCP server có thể phơi bày giao diện dữ liệu nhạy cảm
- Độ tin cậy của kỹ năng bên thứ ba khó kiểm chứng

<strong>Ô nhiễm ngữ cảnh</strong>:
- Khi số lượng Skills tăng lên, ngay cả metadata cũng có thể chiếm dụng lượng lớn ngữ cảnh
- Cần cơ chế đánh chỉ mục (index) và truy xuất kỹ năng thông minh hơn

<strong>Rủi ro phân mảnh</strong>:
- Mặc dù MCP đang được chuẩn hóa, nhưng định dạng Skills vẫn chưa được thống nhất
- Các nhà cung cấp khác nhau có thể đưa ra các quy cách Skills không tương thích với nhau

## Tổng kết

Agent Skills và MCP đại diện cho hai tầng trừu tượng then chốt trong technology stack của Agent:

- <strong>MCP (Model Context Protocol)</strong>: giải quyết vấn đề "khả năng kết nối", là giao diện chuẩn hóa cho tương tác giữa Agent và thế giới bên ngoài, tương đương với "hệ thần kinh" hoặc "đôi tay"
- <strong>Agent Skills</strong>: giải quyết vấn đề "năng lực", là nơi đóng gói kiến thức chuyên môn và quy trình làm việc, tương đương với "vỏ não" hoặc "sổ tay vận hành"

Hai bên không phải quan hệ cạnh tranh, mà là quan hệ bổ trợ lẫn nhau:

<div align="center">
  <img src="./images/Extra05-figures/image6.png" alt="MCP与Agent Skills总结对比" width="90%"/>
  <p>Hình 6 Tổng kết so sánh toàn diện giữa MCP và Agent Skills</p>
</div>

<strong>Góc nhìn then chốt</strong>:

1. <strong>Kiến trúc phân tầng là xu hướng tất yếu</strong>: khi độ phức tạp của hệ thống Agent tăng lên, việc tách biệt "tầng kết nối" và "tầng tri thức" là không thể tránh khỏi

2. <strong>Hiệu quả ngữ cảnh là mâu thuẫn cốt lõi</strong>: cơ chế tiết lộ dần dần của Skills giảm mức tiêu tốn token hơn 90%, đây là lợi thế kỹ thuật lớn nhất của nó

3. <strong>Dân chủ hóa kiến thức chuyên môn</strong>: Skills cho phép cả những người không phải lập trình viên cũng có thể đóng góp năng lực cho Agent, điều này sẽ mở rộng đáng kể biên giới của các ứng dụng AI

4. <strong>Kiến trúc lai là thực hành tốt nhất</strong>: trong các ứng dụng doanh nghiệp, MCP cung cấp kết nối hạ tầng, Skills cung cấp logic nghiệp vụ, kết hợp cả hai mới có thể xây dựng hệ thống Agent hiệu quả và dễ bảo trì

<strong>Đề xuất thực hành</strong>:

- Đối với <strong>kết nối dịch vụ bên ngoài</strong> (cơ sở dữ liệu, API, dịch vụ đám mây), ưu tiên sử dụng MCP
- Đối với <strong>quy trình làm việc phức tạp</strong> (tác vụ nhiều bước, kiến thức chuyên môn), ưu tiên sử dụng Skills
- Trong các kịch bản <strong>hạn chế ngữ cảnh</strong> (hội thoại dài, nhiều công cụ), sử dụng Skills để quản lý theo kiểu tiết lộ dần dần
- Khi xây dựng <strong>Agent cấp doanh nghiệp</strong>, áp dụng kiến trúc phân tầng MCP + Skills

Thông qua việc học chương này, bạn nên có khả năng:
- Hiểu được sự khác biệt bản chất và mối quan hệ hợp tác giữa Agent Skills và MCP
- Nắm vững cơ chế tiết lộ dần dần của Skills và ưu điểm của nó
- Viết được file SKILL.md chất lượng cao
- Lựa chọn và kết hợp hợp lý hai công nghệ này trong dự án thực tế
- Xây dựng hệ thống Agent có kiến trúc phân tầng rõ ràng, hiệu quả và dễ bảo trì

Công nghệ Agent vẫn đang tiến hóa nhanh chóng. MCP đã trở thành tiêu chuẩn thực tế (de facto standard) của tầng kết nối, triết lý Skills cũng đang ảnh hưởng tới toàn ngành. Nắm vững hai công nghệ này sẽ giúp bạn xây dựng được những ứng dụng Agent mạnh mẽ và thực dụng hơn trong làn sóng AI.

---

## Tài liệu tham khảo

1. Tài liệu chính thức Anthropic Agent Skills: https://docs.anthropic.com/en/docs/agent-skills
2. Kho GitHub Anthropic Skills: https://github.com/anthropics/skills
3. Quy cách Model Context Protocol: https://modelcontextprotocol.io/
4. Blog Anthropic: Improving Frontend Design Through Skills: https://www.claude.com/blog/improving-frontend-design-through-skills
5. Chương 10: Giao thức giao tiếp của Agent (hello-agents)
</content>
