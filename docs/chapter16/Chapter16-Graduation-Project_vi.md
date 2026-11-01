# Chương 16 Đồ án tốt nghiệp: Xây dựng ứng dụng Multi-Agent của riêng bạn

Chúc mừng bạn đã đến với chương cuối cùng của giáo trình Hello-Agents! Trong 15 chương trước, chúng ta đã xây dựng framework HelloAgents từ con số không, học về các khái niệm cốt lõi của Agent, nhiều paradigm khác nhau, hệ thống công cụ (tool), cơ chế bộ nhớ (memory), giao thức giao tiếp (communication protocol), huấn luyện reinforcement learning và đánh giá hiệu năng. Trong các Chương 13-15, chúng ta còn trình bày ba dự án thực chiến hoàn chỉnh (Trợ lý du lịch thông minh, Agent nghiên cứu sâu tự động hóa, Thị trấn điều khiển học) để minh họa cách vận dụng tổng hợp những kiến thức đã học.

Bây giờ, đã đến lúc để bạn trở thành một người xây dựng hệ thống Agent thực thụ! Chương này sẽ hướng dẫn bạn <strong>xây dựng ứng dụng multi-agent của riêng mình</strong> và chia sẻ thành quả với cộng đồng thông qua hình thức hợp tác mã nguồn mở (open source).

## 16.1 Ý nghĩa của đồ án tốt nghiệp

### 16.1.1 Vì sao nên làm đồ án tốt nghiệp

Cách học công nghệ tốt nhất không phải là đọc tài liệu hướng dẫn, mà là <strong>tự tay thực hành</strong>. Qua việc học các chương trước, bạn đã nắm được kiến thức lý thuyết và công cụ kỹ thuật để xây dựng hệ thống Agent. Nhưng thách thức thực sự nằm ở: <strong>làm thế nào để áp dụng những kiến thức này vào vấn đề thực tế? Làm thế nào để thiết kế một hệ thống hoàn chỉnh? Làm thế nào để xử lý các trường hợp biên và ngoại lệ?</strong>

Giá trị cốt lõi của đồ án tốt nghiệp là rèn luyện khả năng ứng dụng tổng hợp của bạn, tích hợp có chọn lọc tất cả kiến thức đã học trước đó (paradigm Agent, hệ thống tool, cơ chế memory, communication protocol, v.v.) vào một dự án hoàn chỉnh.

Thông qua việc học và thực hành ở chương này, hy vọng bạn có thể độc lập thiết kế và triển khai một ứng dụng Agent hoàn chỉnh, sử dụng thành thạo các chức năng của framework HelloAgents, nắm vững các thao tác cơ bản của Git và GitHub, học cách viết tài liệu dự án rõ ràng, tham gia hợp tác phát triển trong cộng đồng mã nguồn mở, và cuối cùng có được một sản phẩm kỹ thuật có thể trình diễn.

### 16.1.2 Hình thức của đồ án tốt nghiệp

Đồ án tốt nghiệp của bạn sẽ được nộp dưới dạng <strong>dự án mã nguồn mở</strong> vào kho lưu trữ dự án đồng sáng tạo của Hello-Agents (thư mục `Co-creation-projects`). Yêu cầu cụ thể như sau:

1. <strong>Đặt tên dự án</strong>: Sử dụng định dạng `{tên người dùng GitHub của bạn}-{tên dự án}`, ví dụ `jjyaoao-CodeReviewAgent`

2. <strong>Nội dung dự án</strong>:
   - Một Jupyter Notebook (file `.ipynb`) hoặc script Python có thể chạy được
   - Danh sách dependency đầy đủ (`requirements.txt`)
   - Tài liệu README rõ ràng (`README.md`)
   - Tùy chọn: video demo, ảnh chụp màn hình, dataset, v.v.

3. <strong>Cách nộp bài</strong>: Nộp qua Pull Request (PR) của GitHub

4. <strong>Quy trình đánh giá</strong>: Thành viên cộng đồng sẽ review code của bạn, đưa ra đề xuất cải thiện, sau khi thông qua sẽ merge vào kho lưu trữ chính

## 16.2 Hướng dẫn chọn đề tài dự án

### 16.2.1 Nguyên tắc chọn đề tài

Một dự án đồ án tốt nghiệp tốt nên có tính thực tiễn, giải quyết vấn đề thực sự thay vì công nghệ vì công nghệ, chúng ta cần hướng tới việc có thể hoàn thành trong thời gian và nguồn lực hạn chế, đồng thời có thể thể hiện rõ ràng năng lực kỹ thuật của bạn.

### 16.2.2 Các hướng đề tài được đề xuất

Dưới đây là một số hướng dự án được đề xuất, bạn có thể chọn một trong số đó, hoặc tự đề xuất ý tưởng mới:

<strong>(1) Nhóm công cụ năng suất</strong>

- <strong>Trợ lý review code thông minh</strong>: Tự động phân tích chất lượng code, phát hiện bug tiềm ẩn, đề xuất tối ưu
- <strong>Bộ sinh tài liệu thông minh</strong>: Tự động sinh tài liệu API, hướng dẫn sử dụng dựa trên code
- <strong>Trợ lý họp thông minh</strong>: Ghi lại nội dung cuộc họp, sinh biên bản họp, trích xuất action item
- <strong>Trợ lý email thông minh</strong>: Tự động phân loại email, sinh bản nháp trả lời, nhắc nhở việc quan trọng

<strong>(2) Nhóm hỗ trợ học tập</strong>

- <strong>Bạn đồng hành học tập thông minh</strong>: Đề xuất tài nguyên học tập theo tiến độ, sinh bài tập, giải đáp thắc mắc
- <strong>Trợ lý luận văn thông minh</strong>: Giúp tìm tài liệu, tóm tắt bài báo, sinh trích dẫn
- <strong>Gia sư lập trình thông minh</strong>: Cung cấp bài tập lập trình, review code, lập lộ trình học tập
- <strong>Trợ lý học ngôn ngữ thông minh</strong>: Cung cấp luyện hội thoại, sửa lỗi ngữ pháp, mở rộng từ vựng

<strong>(3) Nhóm sáng tạo giải trí</strong>

- <strong>Bộ sinh truyện thông minh</strong>: Sinh tiểu thuyết, kịch bản, thơ dựa trên input của người dùng
- <strong>NPC game thông minh</strong>: Tạo nhân vật game có cá tính, có thể trò chuyện tự nhiên với người chơi
- <strong>Gợi ý nhạc thông minh</strong>: Gợi ý nhạc theo tâm trạng, bối cảnh, tạo playlist
- <strong>Trợ lý công thức nấu ăn thông minh</strong>: Gợi ý công thức theo nguyên liệu, khẩu vị, sinh danh sách mua sắm

<strong>(4) Nhóm phân tích dữ liệu</strong>

- <strong>Chuyên gia phân tích dữ liệu thông minh</strong>: Tự động phân tích dữ liệu, sinh biểu đồ trực quan, viết báo cáo phân tích
- <strong>Phân tích cổ phiếu thông minh</strong>: Phân tích dữ liệu cổ phiếu, dư luận tin tức, đưa ra gợi ý đầu tư
- <strong>Giám sát dư luận thông minh</strong>: Giám sát mạng xã hội, trang tin tức, phân tích xu hướng dư luận
- <strong>Phân tích đối thủ cạnh tranh thông minh</strong>: Thu thập thông tin đối thủ, so sánh phân tích, sinh báo cáo

<strong>(5) Nhóm dịch vụ đời sống</strong>

- <strong>Trợ lý sức khỏe thông minh</strong>: Ghi lại dữ liệu sức khỏe, đưa ra lời khuyên sức khỏe, lập kế hoạch tập luyện
- <strong>Trợ lý quản lý tài chính thông minh</strong>: Ghi chi tiêu, phân tích thói quen tiêu dùng, đưa ra lời khuyên tài chính
- <strong>Trợ lý mua sắm thông minh</strong>: So sánh giá, gợi ý sản phẩm, sinh danh sách mua sắm
- <strong>Điều khiển nhà thông minh</strong>: Điều khiển thiết bị nhà thông minh bằng ngôn ngữ tự nhiên

### 16.2.3 Ví dụ chọn đề tài

Hãy cùng xem một ví dụ cụ thể để minh họa cách chọn đề tài và thiết kế dự án.

<strong>Tên dự án</strong>: Trợ lý review code thông minh (CodeReviewAgent)

<strong>Phân tích vấn đề</strong>: Review code là khâu quan trọng trong phát triển phần mềm, nhưng review thủ công tốn thời gian và dễ bỏ sót vấn đề. Các công cụ phân tích tĩnh hiện có chỉ phát hiện được lỗi cú pháp, không hiểu được logic code, do đó cần một trợ lý thông minh có thể hiểu ngữ nghĩa code và cung cấp phân tích chuyên sâu.

<strong>Chức năng cốt lõi</strong>: Dự án này sẽ triển khai phân tích chất lượng code (kiểm tra code style, quy ước đặt tên, tính đầy đủ của comment), phát hiện bug tiềm ẩn (phát hiện lỗi logic, vấn đề điều kiện biên, rò rỉ tài nguyên), đề xuất tối ưu hiệu năng (xác định bottleneck hiệu năng, đề xuất phương án tối ưu), quét lỗ hổng bảo mật (phát hiện SQL injection, XSS và các vấn đề bảo mật khác) cũng như đề xuất best practice (đề xuất cải tiến dựa trên đặc tính ngôn ngữ và design pattern).

<strong>Kết quả kỳ vọng</strong>: Cuối cùng sẽ bàn giao một Jupyter Notebook có thể chạy được, thể hiện toàn bộ quy trình review, hỗ trợ các ngôn ngữ chủ đạo như Python, JavaScript, có thể sinh báo cáo review theo định dạng Markdown có cấu trúc, và cung cấp ví dụ code cụ thể cùng đề xuất cải thiện.

## 16.3 Chuẩn bị môi trường phát triển

### 16.3.1 Cài đặt công cụ cần thiết

Trước khi bắt đầu phát triển, hãy đảm bảo môi trường phát triển của bạn đã cài đặt các công cụ sau:

<strong>(1) Môi trường Python</strong>

```bash
# Cài đặt HelloAgents
pip install "hello-agents[all]"
```

<strong>(2) Git và GitHub</strong>

```bash
# Kiểm tra phiên bản Git
git --version

# Cấu hình thông tin người dùng Git
git config --global user.name "Tên của bạn"
git config --global user.email "Email của bạn"

# Cấu hình SSH key cho GitHub (khuyến nghị)
# 1. Tạo SSH key
ssh-keygen -t ed25519 -C "Email của bạn"

# 2. Thêm public key vào GitHub
# Copy nội dung của ~/.ssh/id_ed25519.pub
# Thêm vào GitHub Settings > SSH and GPG keys

# 3. Kiểm tra kết nối
ssh -T git@github.com
```

<strong>(3) Jupyter Notebook</strong>

```bash
# Cài đặt Jupyter
pip install jupyter notebook

# Hoặc sử dụng JupyterLab (khuyến nghị)
pip install jupyterlab

# Khởi động Jupyter
jupyter lab
```

### 16.3.2 Fork kho lưu trữ dự án

<strong>Bước 1: Fork kho lưu trữ</strong>

1. Truy cập kho lưu trữ Hello-Agents: https://github.com/datawhalechina/hello-agents
2. Nhấn nút "Fork" ở góc trên bên phải, như vị trí khung đỏ trong Hình 16.1
3. Chọn tài khoản GitHub của bạn, tạo Fork

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-1.png" alt="" width="85%"/>
  <p>Hình 16.1 Các bước Fork kho lưu trữ</p>
</div>

<strong>Bước 2: Clone về máy cục bộ</strong>

```bash
# Như trong Hình 16.2, clone kho lưu trữ bạn đã Fork
git clone git@github.com:tên_người_dùng_của_bạn/hello-agents.git

# Vào thư mục dự án
cd Hello-Agents

# Thêm upstream repository (dùng để đồng bộ cập nhật)
git remote add upstream https://github.com/datawhalechina/hello-agents.git

# Xem các remote repository
git remote -v
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-2.png" alt="" width="85%"/>
  <p>Hình 16.2 Clone kho lưu trữ về máy cục bộ</p>
</div>

<strong>Bước 3: Tạo nhánh phát triển</strong>

```bash
# Tạo và chuyển sang nhánh mới
git checkout -b feature/tên_dự_án_của_bạn

# Ví dụ:
git checkout -b feature/code-review-agent
```


### 16.3.3 Cấu trúc thư mục dự án

Tạo thư mục dự án của bạn trong thư mục `Co-creation-projects`:

```bash
# Vào thư mục dự án đồng sáng tạo
cd Co-creation-projects

# Tạo thư mục dự án (định dạng: tên_người_dùng_GitHub-tên_dự_án)
mkdir tên_người_dùng_của_bạn-tên_dự_án

# Ví dụ:
mkdir jjyaoao-CodeReviewAgent

# Vào thư mục dự án
cd jjyaoao-CodeReviewAgent
```

Cấu trúc dự án được đề xuất:

```
jjyaoao-CodeReviewAgent/
├── README.md              # Tài liệu mô tả dự án
├── requirements.txt       # Danh sách dependency Python
├── main.ipynb            # Jupyter Notebook chính
├── data/                 # File dữ liệu (tùy chọn)
│   ├── sample_code.py
│   └── test_cases.json
├── outputs/              # Kết quả đầu ra (tùy chọn)
│   ├── review_report.md
│   └── screenshots/
├── src/                  # Mã nguồn (tùy chọn, nếu code nhiều)
│   ├── agents/
│   ├── tools/
│   └── utils/
└──
```

## 16.4 Hướng dẫn phát triển dự án

### 16.4.1 Viết tài liệu README

README là bộ mặt của dự án, một README tốt nên bao gồm các nội dung sau:

```markdown
# Tên dự án

> Mô tả dự án của bạn trong một câu

## 📝 Giới thiệu dự án

Giới thiệu chi tiết về dự án của bạn:
- Giải quyết vấn đề gì?
- Có những tính năng đặc sắc nào?
- Phù hợp với tình huống sử dụng nào?

## ✨ Chức năng cốt lõi

- [ ] Chức năng 1: mô tả
- [ ] Chức năng 2: mô tả
- [ ] Chức năng 3: mô tả

## 🛠️ Tech stack

- Framework HelloAgents
- Paradigm Agent được sử dụng (như ReAct, Plan-and-Solve, v.v.)
- Tool và API được sử dụng
- Các thư viện dependency khác

## 🚀 Bắt đầu nhanh

### Yêu cầu môi trường

- Python 3.10+
- Yêu cầu khác

### Cài đặt dependency


pip install -r requirements.txt


### Cấu hình API key


# Tạo file .env
cp .env.example .env

# Chỉnh sửa file .env, điền API key của bạn


### Chạy dự án


# Khởi động Jupyter Notebook
jupyter lab

# Mở main.ipynb và chạy


## 📖 Ví dụ sử dụng

Minh họa cách sử dụng dự án của bạn, tốt nhất nên kèm ví dụ code và kết quả chạy.

## 🎯 Điểm nổi bật của dự án

- Điểm nổi bật 1: giải thích
- Điểm nổi bật 2: giải thích
- Điểm nổi bật 3: giải thích

## 📊 Đánh giá hiệu năng

Nếu có kết quả đánh giá, hãy trình bày ở đây:
- Độ chính xác: XX%
- Thời gian phản hồi: XX giây
- Các chỉ số khác

## 🔮 Kế hoạch tương lai

- [ ] Chức năng 1 chưa triển khai
- [ ] Chức năng 2 chưa triển khai
- [ ] Phần cần tối ưu

## 🤝 Hướng dẫn đóng góp

Hoan nghênh đóng góp Issue và Pull Request!

## 📄 Giấy phép

MIT License

## 👤 Tác giả

- GitHub: [@tên_người_dùng_của_bạn](https://github.com/tên_người_dùng_của_bạn)
- Email: Email của bạn (tùy chọn)

## 🙏 Lời cảm ơn

Cảm ơn cộng đồng Datawhale và dự án Hello-Agents!
```

### 16.4.2 Viết requirements.txt

Liệt kê tất cả các dependency Python mà dự án cần:

```txt
# Dependency cốt lõi
hello-agents[all]>=0.2.7

# Trực quan hóa (nếu cần)
matplotlib>=3.7.0
plotly>=5.14.0

# Web framework (nếu cần)
fastapi>=0.109.0
uvicorn>=0.27.0
```

### 16.4.3 Phát triển Jupyter Notebook

<strong>(1) Đề xuất cấu trúc Notebook</strong>

Một Jupyter Notebook tốt nên bao gồm các phần sau:

```python
# ========================================
# Phần 1: Giới thiệu dự án
# ========================================

"""
# Tên dự án

## Giới thiệu dự án
Giới thiệu ngắn gọn về mục tiêu và chức năng của dự án

## Thông tin tác giả
- Tên: XXX
- GitHub: @XXX
- Ngày: 2025-XX-XX
"""

# ========================================
# Phần 2: Cấu hình môi trường
# ========================================

# Cài đặt dependency
!pip install -q hello-agents[all]

# Import các thư viện cần thiết
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BaseTool
import os
from dotenv import load_dotenv

# Load biến môi trường
load_dotenv()

# ========================================
# Phần 3: Định nghĩa tool
# ========================================

class CustomTool(BaseTool):
    """Lớp tool tùy chỉnh"""

    name = "tool_name"
    description = "Mô tả tool"

    def run(self, query: str) -> str:
        """Logic thực thi tool"""
        # Triển khai logic tool của bạn
        return "Kết quả"

# ========================================
# Phần 4: Xây dựng Agent
# ========================================

# Tạo LLM
llm = HelloAgentsLLM()

# Tạo Agent
agent = SimpleAgent(
    name="Tên Agent",
    llm=llm,
    system_prompt="Prompt hệ thống"
)

# Thêm tool
agent.add_tool(CustomTool())

# ========================================
# Phần 5: Demo chức năng
# ========================================

# Ví dụ 1: Chức năng cơ bản
print("=== Ví dụ 1: Chức năng cơ bản ===")
result = agent.run("Input của người dùng")
print(result)

# Ví dụ 2: Tình huống phức tạp
print("\n=== Ví dụ 2: Tình huống phức tạp ===")
result = agent.run("Input phức tạp của người dùng")
print(result)

# ========================================
# Phần 6: Đánh giá hiệu năng (tùy chọn)
# ========================================

# Code đánh giá
# ...

# ========================================
# Phần 7: Tổng kết và triển vọng
# ========================================

"""
## Tổng kết dự án

### Chức năng đã triển khai
- Chức năng 1
- Chức năng 2

### Thách thức gặp phải
- Thách thức 1 và giải pháp
- Thách thức 2 và giải pháp

### Hướng cải thiện trong tương lai
- Cải thiện 1
- Cải thiện 2
"""
```

### 16.4.4 Kiểm thử dự án của bạn

Trước khi nộp bài, bạn có thể dùng checklist kiểm thử sau để tự đánh giá xem dự án của mình đã đáp ứng yêu cầu nộp bài hay chưa:

```markdown
- [ ] Code chạy bình thường, không báo lỗi
- [ ] Tài liệu README đầy đủ, mô tả rõ ràng
- [ ] requirements.txt bao gồm đầy đủ dependency
- [ ] Có ví dụ sử dụng rõ ràng
- [ ] Code có comment phù hợp
- [ ] Kết quả đầu ra đúng như kỳ vọng
- [ ] Đã xử lý các tình huống ngoại lệ thường gặp
- [ ] Cấu trúc dự án rõ ràng, đặt tên file theo quy chuẩn
- [ ] File lớn đã được xử lý đúng cách (xem mục bên dưới)
```

### 16.4.5 Hướng dẫn xử lý file lớn

<strong>⚠️ Quan trọng: Tránh làm kho lưu trữ chính quá nặng</strong>

Để giữ cho kho lưu trữ chính của Hello-Agents nhẹ nhàng, vui lòng tuân theo quy chuẩn xử lý file lớn sau:

<strong>(1) Giới hạn dung lượng file</strong>

- **Tổng dung lượng dự án**: Không vượt quá 5MB
- **Cấm nộp trực tiếp**: File video, dataset lớn, file model

<strong>(2) Phương án xử lý file lớn</strong>

Nếu dự án của bạn chứa file lớn (dataset, video, model, v.v.), hãy sử dụng các phương án sau:

**Phương án 1: Sử dụng link bên ngoài (khuyến nghị)**

Upload file lớn lên nền tảng bên ngoài, cung cấp link tải trong README:

```markdown
## Dataset

Dataset sử dụng trong dự án này khá lớn, vui lòng tải từ các link sau:

- Dataset 1: [Baidu Netdisk](link) Mã trích xuất: xxxx
- Dataset 2: [Google Drive](link)
- Video demo: [Bilibili](link) / [YouTube](link)
```

Các nền tảng bên ngoài được đề xuất:
- **Dataset**: Baidu Netdisk, Google Drive, Kaggle, HuggingFace Datasets
- **Video**: Bilibili, YouTube, Tencent Video
- **Model**: HuggingFace Models, ModelScope
- **Hình ảnh**: GitHub Issues, dịch vụ lưu trữ ảnh

**Phương án 2: Tạo kho lưu trữ độc lập**

Nếu dự án có nhiều tài nguyên, nên tạo một kho lưu trữ dữ liệu độc lập:

```markdown
## Tài nguyên dự án

Do dự án chứa nhiều dữ liệu và tài nguyên demo, đã tạo riêng một kho lưu trữ tài nguyên:

- Kho lưu trữ tài nguyên: https://github.com/tên_người_dùng_của_bạn/tên_dự_án-resources
- Nội dung bao gồm: dataset, video demo, file model, dữ liệu kiểm thử, v.v.

### Cách sử dụng

\`\`\`bash
# Clone kho lưu trữ tài nguyên
git clone https://github.com/tên_người_dùng_của_bạn/tên_dự_án-resources.git

# Đưa dữ liệu vào thư mục dự án
cp -r tên_dự_án-resources/data ./data
\`\`\`
```

**Phương án 3: Sử dụng dữ liệu mẫu**

Trong kho lưu trữ chính chỉ cung cấp dữ liệu mẫu quy mô nhỏ:

```python
# Giải thích trong README
## Giải thích dữ liệu

- `data/sample.csv`: Dữ liệu mẫu (100 bản ghi)
- Dataset đầy đủ (100.000 bản ghi) vui lòng tải từ [đây](link)
```

<strong>(3) Ví dụ best practice</strong>

```
tên_người_dùng_của_bạn-tên_dự_án/
├── README.md              # Bao gồm link tài nguyên bên ngoài
├── requirements.txt
├── main.ipynb
├── .gitignore            # Bỏ qua file lớn
├── data/
│   └── sample.csv        # Chỉ dữ liệu mẫu (<1MB)
└── outputs/
    └── demo_result.png   # Chỉ kết quả demo (<1MB)
```

Giải thích trong README:

```markdown
## Dữ liệu và tài nguyên

### Dữ liệu mẫu
Dự án chứa dữ liệu mẫu quy mô nhỏ để kiểm thử nhanh (nằm tại `data/sample.csv`)

### Dataset đầy đủ
Dataset đầy đủ (500MB) vui lòng tải từ link sau:
- Baidu Netdisk: [link] Mã trích xuất: xxxx
- Sau khi tải xuống, giải nén vào thư mục `data/`

### Video demo
- Bilibili: [Video demo dự án](link)
- YouTube: [Demo Video](link)
```

## 16.5 Nộp Pull Request

### 16.5.1 Đẩy code lên GitHub

<strong>Bước 1: Kiểm tra thay đổi</strong>

```bash
# Xem các file đã thay đổi
git status
```

<strong>Bước 2: Thêm file</strong>

```bash
# Thêm tất cả các file đã thay đổi
git add .

# Hoặc thêm file cụ thể
git add Co-creation-projects/tên_người_dùng_của_bạn-tên_dự_án/
```

<strong>Bước 3: Commit thay đổi</strong>

Thông điệp commit nên tuân theo định dạng sau:

```bash
# Định dạng: loại: mô tả ngắn gọn
git commit -m "feat: thêm dự án đồ án tốt nghiệp XXX"
```

<strong>Quy chuẩn loại commit:</strong>

- `feat`: Thêm chức năng hoặc dự án mới (dự án đồ án tốt nghiệp sử dụng loại này)
- `fix`: Sửa bug
- `docs`: Cập nhật tài liệu
- `style`: Điều chỉnh định dạng code (không ảnh hưởng chức năng)
- `refactor`: Tái cấu trúc code
- `test`: Liên quan đến kiểm thử
- `chore`: Thay đổi khác (như cập nhật dependency)

<strong>Bước 4: Push lên GitHub</strong>

```bash
# Push lên kho lưu trữ Fork của bạn
git push origin feature/tên_dự_án_của_bạn
```

### 16.5.2 Tạo Pull Request

<strong>Bước 1: Truy cập GitHub</strong>

1. Truy cập kho lưu trữ Fork của bạn: `https://github.com/tên_người_dùng_của_bạn/hello-agents`
2. Nhấn tab "Pull requests", như trong Hình 16.3
3. Nhấn nút "New pull request"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-3.png" alt="" width="85%"/>
  <p>Hình 16.3 Tạo Pull Request</p>
</div>


<strong>Bước 2: Chọn nhánh</strong>

- Base repository: `datawhalechina/hello-agents`
- Base branch: `main`
- Head repository: `tên_người_dùng_của_bạn/hello-agents`
- Compare branch: `feature/tên_dự_án_của_bạn`

<strong>Bước 3: Điền thông tin PR</strong>

<strong>⚠️ Quan trọng: Định dạng tiêu đề PR thống nhất</strong>

Để thuận tiện cho việc quản lý và tra cứu, tiêu đề PR của tất cả các dự án đồ án tốt nghiệp phải tuân theo định dạng sau:

```
[Đồ án tốt nghiệp] Tên dự án - Mô tả ngắn gọn
```

Ví dụ:
- `[Đồ án tốt nghiệp] CodeReviewAgent - Trợ lý review code thông minh`
- `[Đồ án tốt nghiệp] StudyBuddy - Bạn đồng hành học tập AI`
- `[Đồ án tốt nghiệp] DataAnalyst - Chuyên gia phân tích dữ liệu thông minh`

<strong>Template mô tả PR:</strong>

```markdown
## Thông tin dự án

- **Tên dự án**: XXX
- **Tác giả**: @tên_người_dùng_của_bạn
- **Loại dự án**: Công cụ năng suất/Hỗ trợ học tập/Sáng tạo giải trí/Phân tích dữ liệu/Dịch vụ đời sống

## Giới thiệu dự án

Mô tả ngắn gọn về dự án của bạn (2-3 câu)

## Chức năng cốt lõi

- [ ] Chức năng 1
- [ ] Chức năng 2
- [ ] Chức năng 3

## Điểm nổi bật kỹ thuật

- Đã sử dụng paradigm XXX
- Đã triển khai chức năng XXX
- Đã tối ưu hiệu năng XXX

## Hiệu ứng demo

(Tùy chọn) Thêm ảnh chụp màn hình hoặc GIF để minh họa hiệu ứng dự án

## Checklist tự kiểm tra

- [ ] Code chạy bình thường
- [ ] Tài liệu README đầy đủ
- [ ] requirements.txt đầy đủ
- [ ] Có ví dụ sử dụng rõ ràng
- [ ] Code có comment phù hợp

## Giải thích khác

(Tùy chọn) Nội dung khác cần giải thích
```

<strong>Bước 4: Nộp PR</strong>

Như trong Hình 16.4, nhấn nút "Create pull request" để nộp.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-4.png" alt="" width="85%"/>
  <p>Hình 16.4 Nộp Pull Request</p>
</div>



### 16.5.3 Phản hồi ý kiến Review

Sau khi nộp PR, thành viên cộng đồng sẽ review code của bạn và đưa ra đề xuất. Vui lòng phản hồi kịp thời:

1. <strong>Xem bình luận</strong>: Xem bình luận của reviewer trên trang PR
2. <strong>Sửa code</strong>: Sửa code theo đề xuất
3. <strong>Nộp cập nhật</strong>:
   ```bash
   git add .
   git commit -m "fix: sửa XXX theo ý kiến review"
   git push origin feature/tên_dự_án_của_bạn
   ```
4. <strong>Phản hồi bình luận</strong>: Phản hồi reviewer trên GitHub, giải thích những thay đổi của bạn

## 16.6 Trình diễn dự án mẫu

Để giúp bạn hiểu rõ hơn yêu cầu của đồ án tốt nghiệp, dưới đây là một dự án mẫu hoàn chỉnh, xin đừng lo lắng, những ý tưởng nhỏ cũng hoàn toàn có thể được chấp nhận, miễn là tác phẩm do chính bạn tự tay làm ra thì đều đáng trân trọng.

<strong>Thông tin dự án</strong>

- **Tên dự án**: CodeReviewAgent
- **Tác giả**: @jjyaoao
- **Đường dẫn dự án**: `Co-creation-projects/jjyaoao-CodeReviewAgent/`

<strong>Cấu trúc dự án</strong>

```
jjyaoao-CodeReviewAgent/
├── README.md              # Tài liệu dự án
├── requirements.txt       # Danh sách dependency
├── main.ipynb            # Chương trình chính (bao gồm demo nhanh và chức năng đầy đủ)
├── .env.example          # Ví dụ biến môi trường
├── .gitignore            # Quy tắc bỏ qua của Git
├── data/
│   └── sample_code.py    # Code mẫu
└── outputs/
    └── review_report.md  # Báo cáo mẫu
```

<strong>Đoạn code cốt lõi (main.ipynb)</strong>

```python
# ========================================
# Trợ lý review code thông minh
# ========================================

from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import Tool, ToolParameter
from typing import Dict, Any, List
import ast
import os

# ========================================
# 0. Cấu hình tham số LLM
# ========================================

os.environ["LLM_MODEL_ID"] = "Qwen/Qwen2.5-72B-Instruct"
os.environ["LLM_API_KEY"] = "your_api_key_here"
os.environ["LLM_BASE_URL"] = "https://api-inference.modelscope.cn/v1/"
os.environ["LLM_TIMEOUT"] = "60"

# ========================================
# 1. Định nghĩa tool phân tích code
# ========================================

class CodeAnalysisTool(Tool):
    """Tool phân tích tĩnh code"""

    def __init__(self):
        super().__init__(
            name="code_analysis",
            description="Phân tích cấu trúc, độ phức tạp và vấn đề tiềm ẩn của code Python"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Phân tích code và trả về kết quả"""
        code = parameters.get("code", "")
        if not code:
            return "Lỗi: code không được để trống"

        try:
            tree = ast.parse(code)
            functions = [node for node in ast.walk(tree)
                        if isinstance(node, ast.FunctionDef)]
            classes = [node for node in ast.walk(tree)
                      if isinstance(node, ast.ClassDef)]

            result = {
                "Số lượng hàm": len(functions),
                "Số lượng class": len(classes),
                "Số dòng code": len(code.split('\n')),
                "Danh sách hàm": [f.name for f in functions],
                "Danh sách class": [c.name for c in classes]
            }
            return str(result)
        except SyntaxError as e:
            return f"Lỗi cú pháp: {str(e)}"

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Code Python cần phân tích",
                required=True
            )
        ]

class StyleCheckTool(Tool):
    """Tool kiểm tra code style"""

    def __init__(self):
        super().__init__(
            name="style_check",
            description="Kiểm tra code có tuân theo quy chuẩn PEP 8 hay không"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Kiểm tra style của code"""
        code = parameters.get("code", "")
        if not code:
            return "Lỗi: code không được để trống"

        issues = []
        lines = code.split('\n')
        for i, line in enumerate(lines, 1):
            if len(line) > 79:
                issues.append(f"Dòng {i}: vượt quá 79 ký tự")
            if line.startswith(' ') and not line.startswith('    '):
                if len(line) - len(line.lstrip()) not in [0, 4, 8, 12]:
                    issues.append(f"Dòng {i}: thụt lề không chuẩn")

        if not issues:
            return "Code style tốt, tuân theo quy chuẩn PEP 8"
        return "Phát hiện các vấn đề sau:\n" + "\n".join(issues)

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Code Python cần kiểm tra",
                required=True
            )
        ]

# ========================================
# 2. Tạo tool registry và Agent
# ========================================

# Tạo tool registry
tool_registry = ToolRegistry()
tool_registry.register_tool(CodeAnalysisTool())
tool_registry.register_tool(StyleCheckTool())

# Khởi tạo LLM
llm = HelloAgentsLLM()

# Định nghĩa system prompt
system_prompt = """Bạn là một chuyên gia review code giàu kinh nghiệm. Nhiệm vụ của bạn là:

1. Sử dụng tool code_analysis để phân tích cấu trúc code
2. Sử dụng tool style_check để kiểm tra code style
3. Dựa trên kết quả phân tích, cung cấp báo cáo review chi tiết

Báo cáo review nên bao gồm:
- Phân tích cấu trúc code
- Vấn đề về style
- Bug tiềm ẩn
- Đề xuất tối ưu hiệu năng
- Đề xuất best practice

Vui lòng xuất báo cáo theo định dạng Markdown."""

# Tạo Agent
agent = SimpleAgent(
    name="Trợ lý review code",
    llm=llm,
    system_prompt=system_prompt,
    tool_registry=tool_registry
)

# ========================================
# 3. Chạy ví dụ
# ========================================

# Đọc code mẫu
with open("data/sample_code.py", "r", encoding="utf-8") as f:
    sample_code = f.read()

print("=== Code cần review ===")
print(sample_code)
print("\n" + "="*50 + "\n")

# Thực hiện review code
print("=== Bắt đầu review code ===")
review_result = agent.run(f"Vui lòng review đoạn code Python sau:\n\n```python\n{sample_code}\n```")

print(review_result)

# Lưu báo cáo review
with open("outputs/review_report.md", "w", encoding="utf-8") as f:
    f.write(review_result)

print("\nBáo cáo review đã được lưu vào outputs/review_report.md")
```

<strong>Ví dụ README.md</strong>

```markdown
# CodeReviewAgent - Trợ lý review code thông minh

> Công cụ review code thông minh dựa trên framework HelloAgents

## 📝 Giới thiệu dự án

CodeReviewAgent là một trợ lý review code thông minh, có thể tự động phân tích chất lượng code Python, phát hiện vấn đề tiềm ẩn và đưa ra đề xuất tối ưu.

### Chức năng cốt lõi

- ✅ Phân tích cấu trúc code: thống kê số hàm, số class, số dòng code, v.v.
- ✅ Kiểm tra style: kiểm tra có tuân theo quy chuẩn PEP 8 hay không
- ✅ Đề xuất thông minh: cung cấp phân tích sâu và đề xuất tối ưu dựa trên LLM
- ✅ Sinh báo cáo: sinh báo cáo review theo định dạng Markdown

## 🛠️ Tech stack

- Framework HelloAgents (SimpleAgent + ToolRegistry)
- Module Python AST (parse code)
- API ModelScope (model Qwen2.5-72B)

## 🚀 Bắt đầu nhanh

### Cài đặt dependency

\`\`\`bash
pip install -r requirements.txt
\`\`\`

### Cấu hình tham số LLM

**Cách 1: Sử dụng file .env**

\`\`\`bash
cp .env.example .env
# Chỉnh sửa file .env, điền API key của bạn
\`\`\`

**Cách 2: Thiết lập trực tiếp trong Notebook**

Dự án đã cấu hình sẵn API ModelScope, có thể chạy trực tiếp. Nếu cần sửa đổi, hãy chỉnh sửa code cấu hình ở Phần 1 của main.ipynb.

### Chạy dự án

\`\`\`bash
jupyter lab
# Mở main.ipynb và chạy tất cả các cell
\`\`\`

## 📖 Ví dụ sử dụng

1. Đặt code cần review vào `data/sample_code.py`
2. Chạy `main.ipynb`
3. Xem báo cáo review được sinh ra tại `outputs/review_report.md`

## 🎯 Điểm nổi bật của dự án

- **Tự động hóa**: Không cần kiểm tra thủ công từng dòng, tự động phát hiện vấn đề
- **Thông minh hóa**: Tận dụng LLM để hiểu ngữ nghĩa code, cung cấp đề xuất chuyên sâu
- **Có thể mở rộng**: Dễ dàng thêm quy tắc kiểm tra và tool mới

## 👤 Tác giả

- GitHub: [@jjyaoao](https://github.com/jjyaoao)
- Link dự án: [CodeReviewAgent](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/jjyaoao-CodeReviewAgent)

## 🙏 Lời cảm ơn

Cảm ơn cộng đồng Datawhale và dự án Hello-Agents!
```



## 16.7 Tổng kết và triển vọng

Qua việc hoàn thành đồ án tốt nghiệp, bạn hẳn đã nắm được toàn bộ quy trình thiết kế hệ thống Agent. Từ việc xuất phát từ yêu cầu để thiết kế kiến trúc hệ thống, sử dụng thành thạo các chức năng và thành phần của framework HelloAgents, phát triển tool tùy chỉnh để mở rộng năng lực của Agent, hoàn thành toàn bộ quá trình phát triển dự án từ phân tích yêu cầu đến triển khai code, học cách sử dụng Git và GitHub để hợp tác mã nguồn mở, cũng như viết tài liệu kỹ thuật rõ ràng.

Trong dự án này, chúng ta đã xây dựng framework HelloAgents từ con số không, và dùng nó để triển khai nhiều ứng dụng thực tiễn. Hoàn thành đồ án tốt nghiệp chỉ là khởi đầu, bạn có thể tiếp tục học sâu hơn về nhiều paradigm và thuật toán Agent, prompt engineering và context engineering, cơ chế hợp tác multi-agent cùng các kiến thức lý thuyết khác; cũng có thể mở rộng tech stack, học phát triển Web để xây dựng ứng dụng hoàn chỉnh, học database để triển khai lưu trữ dữ liệu bền vững, học triển khai (deploy) để đưa ứng dụng lên production; còn có thể tiếp tục tối ưu dự án của mình, thêm nhiều chức năng hơn, tối ưu hiệu năng và trải nghiệm người dùng, hoàn thiện kiểm thử và tài liệu; quan trọng hơn, hãy tích cực tham gia đóng góp cho cộng đồng, giúp đỡ những người học khác, tham gia phát triển framework Hello-Agents, chia sẻ kinh nghiệm và tâm đắc của bạn.

Từ Agent đơn giản ở chương đầu tiên, đến nay có thể độc lập xây dựng một ứng dụng multi-agent hoàn chỉnh, bạn đã đi qua một hành trình học tập tuyệt vời. Nhưng đây không phải là điểm kết thúc, mà là một khởi đầu mới.

Công nghệ AI thay đổi từng ngày, lĩnh vực Agent càng chứa đựng vô vàn khả năng. Hy vọng bạn có thể giữ sự tò mò để tiếp tục học hỏi công nghệ mới, dũng cảm dùng công nghệ AI để giải quyết vấn đề thực tế và tạo ra giá trị, sẵn lòng chia sẻ kinh nghiệm và thành quả của mình với cộng đồng, không ngừng trau chuốt tác phẩm của mình để theo đuổi sự xuất sắc.

Cuối cùng, cảm ơn bạn đã đọc trọn vẹn dự án này. Hy vọng bạn đã thu hoạch được điều gì đó trong quá trình học tập, cũng hy vọng bạn có thể áp dụng những gì đã học vào dự án thực tế, tạo ra những ứng dụng Agent đáng kinh ngạc. Tương lai của AI chứa đựng vô vàn khả năng, hãy cùng nhau khám phá và sáng tạo!

<strong>Hãy nhớ: cách học tốt nhất chính là tự tay thực hành!</strong>

Bây giờ, hãy bắt đầu xây dựng ứng dụng Agent của riêng bạn! Chúng tôi mong chờ được thấy tác phẩm tuyệt vời của bạn trong thư mục Co-creation-projects!

Nếu bạn thấy dự án Hello-Agents hữu ích, hãy tặng cho chúng tôi một ⭐Star!

---
<div align="center">
  <strong>🎓 Chúc mừng bạn đã hoàn thành khóa học Hello-Agents! 🎉</strong>
</div>
