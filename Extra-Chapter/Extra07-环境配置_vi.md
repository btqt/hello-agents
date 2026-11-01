# Extra07 - Cấu hình môi trường

> Mục này sẽ hướng dẫn bạn cấu hình môi trường đầy đủ cần thiết để chạy FirstAgentTest.py. Đoạn code này triển khai một trợ lý du lịch thông minh, minh họa mô hình triển khai Agent dựa trên gọi công cụ (tool calling).

## I. Yêu cầu môi trường

### 1.1 Yêu cầu phiên bản Python

- **Python 3.10+** (khuyến nghị sử dụng Python 3.10 hoặc phiên bản cao hơn)
- Hệ điều hành được hỗ trợ: Windows, macOS, Linux

### 1.2 Giải thích code mục tiêu

Mục tiêu của chúng ta là chạy thành công dự án `code\chapter1\FirstAgentTest.py`, đoạn code này triển khai:

- Chức năng trợ lý du lịch thông minh
- Công cụ tra cứu thời tiết (dựa trên API wttr.in)
- Công cụ gợi ý địa điểm tham quan (dựa trên API Tavily Search)
- Lời gọi LLM tương thích OpenAI
- Luồng thực thi Agent theo mô hình ReAct

## II. Cấu hình API

### 2.1 Cấu hình API mô hình ngôn ngữ lớn

#### Lựa chọn 1: API AIHubmix (khuyến nghị)

AIHubmix là một nền tảng tổng hợp mô hình AI đặt tại bang Delaware, Mỹ, tích hợp các mô hình ngôn ngữ lớn chủ đạo trên thị trường, các mô hình mới phát hành thường có thể sử dụng được trong vòng một tuần. Nền tảng này kết nối trực tiếp với API gốc của các nhà cung cấp dịch vụ đám mây lớn (như OpenAI qua Azure, Anthropic qua AWS, Google qua giao diện chính thức...), triển khai theo kiến trúc cluster của Google Cloud tại Mỹ, có khả năng cân bằng tải đa node, thể hiện xuất sắc về độ ổn định và tốc độ phản hồi.

> Hạn mức miễn phí mà nền tảng cung cấp đủ để đáp ứng nhu cầu học tập của chúng ta.

1. **Truy cập trang chủ AIHubmix**

   Dùng trình duyệt truy cập [trang web chính thức AIHubmix](https://aihubmix.com/?aff=Igcn/)

   ![image1](images/Extra07-figures/image1.png)

2. **Hoàn tất đăng ký tài khoản**

   Lần đầu sử dụng cần đăng ký tài khoản. Nhấn vào nút đăng ký ở góc trên bên phải, hỗ trợ đăng ký bằng email hoặc số điện thoại.

3. **Duyệt các mô hình khả dụng**

   Sau khi đăng ký thành công, truy cập [Trung tâm mô hình](https://aihubmix.com/models) để xem tất cả các mô hình khả dụng. Trong điều kiện lọc, chọn thẻ `Miễn phí` để xem danh sách các mô hình miễn phí mà nền tảng cung cấp. Khuyến nghị chọn `coding-glm-4.7-free` hoặc các mô hình miễn phí tương thích định dạng OpenAI khác.

   ![image2](images/Extra07-figures/image2.png)

4. **Lấy thông tin xác thực API**

   Truy cập trang [Quản lý khóa API](https://console.aihubmix.com/token), hệ thống mặc định sẽ sinh ra một khóa khả dụng. Bạn cũng có thể nhấn nút `Create Key` để tự đặt tên khóa và sinh khóa mới.

   ![image3](images/Extra07-figures/image3.png)
   
   Hãy lưu lại cẩn thận thông tin cấu hình sau:
   - API Key: `your_api_key`
   - Base URL: `https://aihubmix.com/v1`
   - Mô hình khuyến nghị: `coding-glm-4.7-free`

#### Lựa chọn 2: ModelScope

ModelScope là nhà cung cấp dịch vụ mô hình lớn hàng đầu trong nước (Trung Quốc), cung cấp dịch vụ API với tỷ lệ giá/hiệu năng cao. Ở đây chúng ta lấy Qwen làm ví dụ, bạn có thể lấy từ [ModelScope](https://modelscope.cn/docs/model-service/API-Inference/intro), nền tảng cung cấp API định dạng tương thích (OpenAI) miễn phí cho dòng Qwen, mỗi ngày được gọi miễn phí 2000 lần.

Hãy đảm bảo bạn có một tài khoản ModelScope đã đăng ký hợp lệ và có thể sử dụng được. Để sinh API KEY riêng của bạn, có thể tham khảo hình minh họa của chúng tôi.

![image4](images/Extra07-figures/image4.png)

![image5](images/Extra07-figures/image5.png)

SDK token trong hình chính là API KEY của chúng ta.

> Xin lưu ý, cần liên kết [tài khoản Alibaba Cloud](https://modelscope.cn/docs/accounts/aliyun-binding-and-authorization) trong **Model Service** trước, nếu không API sẽ hiển thị là không thể sử dụng

**Phạm vi mô hình có thể chọn**

Trong [kho mô hình](https://modelscope.cn/models?filter=inference_type&page=1) của ModelScope, chọn mục suy luận API-Inference, tất cả các mô hình trong đó đều có thể chọn, chúng ta có thể trải nghiệm mô hình Llama-70B mới nhất được chưng cất dữ liệu từ DeepSeek-R1.

![image6](images/Extra07-figures/image6.png)

Định dạng cuối cùng cần thiết giống với thông tin cấu hình của AIHubmix (Key, URL, tên mô hình)

### 2.2 Cấu hình API Tavily Search

Tavily là một API tìm kiếm được thiết kế chuyên biệt cho các ứng dụng AI, dùng cho chức năng gợi ý địa điểm tham quan.

1. **Truy cập nền tảng Tavily**

   Mở trình duyệt, truy cập [Tavily](https://tavily.com/)

   ![image7](images/Extra07-figures/image7.png)

2. **Đăng ký và lấy khóa API**

   ![image8](images/Extra07-figures/image8.png)

   1. Đăng ký tài khoản
   2. Lấy API Key trong bảng điều khiển
   3. Ghi lại API Key: `your_tavily_key`

## III. Cấu hình môi trường Python

### 3.1 Cài đặt Python (nếu chưa cài)

**Người dùng Windows:**
1. Truy cập [trang chủ Python](https://www.python.org/downloads/)
2. Tải xuống phiên bản Python 3.10+
3. Khi cài đặt, tích chọn "Add Python to PATH"

**Người dùng macOS:**
```bash
# Cài đặt bằng Homebrew
brew install python@3.10
```

**Người dùng Linux:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.10 python3.10-pip python3.10-venv

# CentOS/RHEL
sudo yum install python3.10 python3.10-pip
```

### 3.2 Xác minh cài đặt Python

```bash
python --version
# hoặc
python3 --version
```

Đảm bảo hiển thị Python 3.10 hoặc phiên bản cao hơn.

## IV. Cấu hình môi trường dự án

### 4.1 Tạo môi trường ảo (khuyến nghị)

```bash
# Vào thư mục dự án
cd "hello-agents"

# Tạo môi trường ảo
python -m venv venv

# Kích hoạt môi trường ảo
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### 4.2 Cài đặt các gói dependency

```bash
# Cài đặt dependency cốt lõi
pip install requests>=2.31.0
pip install tavily-python>=0.3.0
pip install openai>=1.0.0

# Tùy chọn: cài đặt các gói thường dùng khác
pip install python-dotenv>=1.0.0
```

### 4.3 Cấu hình biến môi trường

#### Cách 1: Sử dụng file .env (khuyến nghị)

Tạo file `.env` ở thư mục gốc của dự án:

```bash
# Tạo file .env ở thư mục gốc dự án
touch .env  # Linux/macOS
# hoặc tạo thủ công trên Windows
```

Chỉnh sửa file `.env`, thêm nội dung sau:

```env
# Cấu hình API Tavily
TAVILY_API_KEY=your_tavily_api_key

# Cấu hình API mô hình ngôn ngữ lớn (chọn một trong các cách)
# Lựa chọn 1: AIHubmix
OPENAI_API_KEY=your_aihubmix_api_key
OPENAI_BASE_URL=https://aihubmix.com/v1
MODEL_NAME=xxxx

# Lựa chọn 2: Modelscope
# OPENAI_API_KEY=your_modelscope_api_key
# OPENAI_BASE_URL=https://api-inference.modelscope.cn/v1/
# MODEL_NAME=xxxx
```

#### Cách 2: Biến môi trường hệ thống

Dưới đây là phương án biến môi trường lâu dài, cũng có thể nạp tạm thời trong terminal.

**Windows:**
1. Chuột phải vào "This PC" → "Properties" → "Advanced system settings"
2. Nhấn "Environment Variables"
3. Thêm trong "User variables":
   - `TAVILY_API_KEY`: `your_tavily_api_key`

**macOS/Linux:**
```bash
# Chỉnh sửa ~/.bashrc hoặc ~/.zshrc
export TAVILY_API_KEY="your_tavily_api_key"

# Làm cho cấu hình có hiệu lực
source ~/.bashrc
```

## V. Cấu hình code

### 5.1 Chỉnh sửa cấu hình FirstAgentTest.py

Mở file `code/chapter1/FirstAgentTest.py`, tìm phần cấu hình ở dòng 143-148:

```python
# --- 1. Cấu hình LLM client ---
# Tùy theo dịch vụ bạn sử dụng, hãy thay thế bằng thông tin xác thực và địa chỉ tương ứng
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"
```

**Thay thế bằng cấu hình thực tế của bạn:**

#### Ví dụ cấu hình khi dùng AIHubmix:
```python
API_KEY = "your_aihubmix_api_key"
BASE_URL = "https://aihubmix.com/v1"
MODEL_ID = "coding-glm-4.7-free"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"
```

## VI. Chạy và xác minh

### 6.1 Kiểm tra kết nối mạng

Trước tiên kiểm tra khả năng kết nối của từng API:

```python
# Kiểm tra API thời tiết
import requests
response = requests.get("https://wttr.in/Beijing?format=j1")
print("Trạng thái API thời tiết:", response.status_code)

# Kiểm tra API Tavily
from tavily import TavilyClient
tavily = TavilyClient(api_key="your_tavily_key")
try:
    result = tavily.search("test", search_depth="basic")
    print("Kết nối API Tavily thành công")
except Exception as e:
    print("Lỗi API Tavily:", e)

# Kiểm tra API LLM - AIHubmix
from openai import OpenAI
client = OpenAI(
    api_key="your_aihubmix_api_key",
    base_url="https://aihubmix.com/v1"
)
try:
    response = client.chat.completions.create(
        model="coding-glm-4.7-free",
        messages=[{"role": "user", "content": "Hello"}],
        max_tokens=10
    )
    print("Kết nối API LLM thành công:", response.choices[0].message.content)
except Exception as e:
    print("Lỗi API LLM:", e)

# Kiểm tra API LLM - ModelScope (nếu bạn dùng ModelScope, hãy bỏ comment và thay cấu hình)
# from openai import OpenAI
# client = OpenAI(
#     api_key="your_modelscope_api_key",
#     base_url="https://api-inference.modelscope.cn/v1/"
# )
# try:
#     response = client.chat.completions.create(
#         model="Qwen/Qwen2.5-72B-Instruct",
#         messages=[{"role": "user", "content": "Hello"}],
#         max_tokens=10
#     )
#     print("Kết nối API LLM thành công:", response.choices[0].message.content)
# except Exception as e:
#     print("Lỗi API LLM:", e)
```

### 6.2 Chạy chương trình hoàn chỉnh

```bash
# Đảm bảo đang ở đúng thư mục
cd "hello-agents\code\chapter1"

# Chạy chương trình
python FirstAgentTest.py
```

### 6.3 Đầu ra mong đợi

Khi chương trình chạy thành công, bạn sẽ thấy đầu ra tương tự như sau:

```
用户输入: 你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。
========================================
--- 循环 1 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 用户想要查询北京的天气，然后根据天气情况推荐合适的旅游景点。我需要先调用get_weather工具查询北京的天气情况。
Action: get_weather(city="北京")

Observation: 北京当前天气：Clear，气温15摄氏度
========================================
--- 循环 2 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 现在我知道了北京的天气是晴朗的，气温15摄氏度，这是一个很适合户外活动的天气。接下来我需要根据这个天气情况推荐合适的旅游景点。
Action: get_attraction(city="北京", weather="Clear，气温15摄氏度")

Observation: 根据搜索，为您找到以下信息：...
========================================
任务完成，最终答案: 根据查询，北京今天天气晴朗，气温15摄氏度，非常适合户外游览。推荐您去...
```


## VII. Xử lý sự cố thường gặp

### 7.1 Vấn đề cài đặt dependency

**Vấn đề: cài pip chậm**

Giải pháp: sử dụng mirror trong nước (Trung Quốc)
```bash
# Dùng mirror Thanh Hoa tạm thời
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests tavily-python openai

# Cấu hình mirror vĩnh viễn
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**Vấn đề: ModuleNotFoundError**

Giải pháp:
```bash
# Xác nhận môi trường ảo đã được kích hoạt
# Cài lại các gói còn thiếu
pip install requests tavily-python openai python-dotenv
```

### 7.2 Vấn đề gọi API

**Vấn đề: API Tavily trả về lỗi**

Nguyên nhân có thể:
- API Key chưa được thiết lập đúng
- Hạn mức API đã dùng hết
- Vấn đề kết nối mạng

Giải pháp:
```python
# Kiểm tra biến môi trường
import os
print("TAVILY_API_KEY:", os.environ.get('TAVILY_API_KEY'))

# Kiểm tra kết nối API
from tavily import TavilyClient
client = TavilyClient(api_key="your_key")
result = client.search("test")
```



## VIII. Tổng kết

Sau khi hoàn tất cấu hình môi trường, khuyến nghị bạn:

1. Hiểu cấu trúc code của FirstAgentTest.py
2. Thử chỉnh sửa System Prompt và quan sát hiệu ứng
3. Thêm hàm công cụ mới
4. Triển khai logic Agent phức tạp hơn

Làm theo các bước trong tài liệu này, bạn sẽ có thể chạy thành công code trợ lý du lịch thông minh, và hiểu được nguyên lý triển khai Agent dựa trên gọi công cụ.
</content>
