# Chương 13 Trợ lý du lịch thông minh

Trong các chương trước, chúng ta đã xây dựng framework HelloAgents từ đầu, hiện thực nhiều paradigm Agent, hệ thống công cụ (tool), cơ chế ghi nhớ (memory), giao tiếp protocol và đánh giá hiệu năng. Bắt đầu từ chương này, chúng ta sẽ bước vào một giai đoạn hoàn toàn mới: <strong>vận dụng tổng hợp kiến thức đã học để xây dựng một ứng dụng thực tế hoàn chỉnh.</strong>

Bạn còn nhớ Agent đầu tiên mà chúng ta đã xây dựng ở chương 1 không? Đó là một trợ lý du lịch thông minh đơn giản, minh họa nguyên lý cơ bản của vòng lặp `Thought-Action-Observation`. Trợ lý du lịch thông minh trong chương này sẽ là một dự án hoàn chỉnh, bao gồm các chức năng cốt lõi sau:

<strong>(1) Lập kế hoạch hành trình thông minh</strong>: người dùng nhập điểm đến, ngày đi, sở thích và các thông tin khác, hệ thống tự động tạo ra kế hoạch hành trình hoàn chỉnh bao gồm địa điểm tham quan, ẩm thực, khách sạn.

<strong>(2) Trực quan hóa bản đồ</strong>: đánh dấu vị trí địa điểm tham quan trên bản đồ, vẽ tuyến đường tham quan, giúp hành trình rõ ràng dễ nhìn.

<strong>(3) Tính toán ngân sách</strong>: tự động tính toán chi phí vé tham quan, khách sạn, ẩm thực, giao thông, hiển thị chi tiết ngân sách.

<strong>(4) Chỉnh sửa hành trình</strong>: hỗ trợ thêm, xóa, điều chỉnh địa điểm tham quan, cập nhật bản đồ theo thời gian thực.

<strong>(5) Chức năng xuất file</strong>: hỗ trợ xuất ra PDF hoặc hình ảnh, tiện lưu trữ và chia sẻ.



## 13.1 Tổng quan dự án và thiết kế kiến trúc

### 13.1.1 Vì sao cần trợ lý du lịch thông minh

Lập kế hoạch cho một chuyến du lịch là việc vừa thú vị vừa đau đầu. Bạn cần tìm kiếm thông tin địa điểm tham quan trên mạng, so sánh các cẩm nang du lịch khác nhau, xem dự báo thời tiết, đặt khách sạn, tính toán ngân sách, lập tuyến đường. Quá trình này có thể mất vài giờ thậm chí vài ngày. Và dù đã bỏ ra nhiều thời gian như vậy, bạn vẫn không chắc chắn hành trình đã lập có hợp lý không, có bỏ sót địa điểm quan trọng nào không, ngân sách có chính xác không.

Cách lập kế hoạch du lịch truyền thống có một số điểm bất cập. Đầu tiên là <strong>thông tin phân tán</strong>. Thông tin địa điểm tham quan nằm trên các trang web du lịch, thông tin thời tiết nằm trên trang web thời tiết, thông tin khách sạn nằm trên trang web đặt phòng, bạn phải chuyển qua lại giữa nhiều trang web, tự tay tổng hợp những thông tin này. Thứ hai là <strong>thiếu cá nhân hóa</strong>. Phần lớn cẩm nang đều mang tính phổ thông, không tính đến sở thích cá nhân, giới hạn ngân sách, thời gian đi lại của bạn. Cuối cùng là <strong>khó điều chỉnh</strong>. Khi bạn muốn sửa lịch trình, có thể phải lập lại toàn bộ hành trình, vì thứ tự địa điểm, sắp xếp thời gian, ngân sách đều liên quan chặt chẽ với nhau.

Công nghệ AI mang đến khả năng mới để giải quyết những vấn đề này. Hãy tưởng tượng, bạn chỉ cần nói với hệ thống "tôi muốn đi Bắc Kinh chơi 3 ngày, thích lịch sử văn hóa, ngân sách trung bình", hệ thống sẽ tự động tạo ra một kế hoạch hành trình hoàn chỉnh cho bạn, bao gồm mỗi ngày đi những địa điểm nào, ăn ở đâu, ở khách sạn nào, cần bao nhiêu ngân sách. Và kế hoạch này có thể điều chỉnh được, bạn có thể xóa địa điểm không thích, điều chỉnh thứ tự tham quan, hệ thống sẽ tự động cập nhật bản đồ và ngân sách.

Đây chính là trợ lý du lịch thông minh mà chúng ta sẽ xây dựng. Nó không chỉ là một bản demo kỹ thuật, mà là một ứng dụng thực sự hữu ích. Thông qua dự án này, bạn sẽ học được cách ứng dụng công nghệ AI vào các vấn đề thực tế, cách thiết kế hệ thống đa Agent (multi-agent system), cách xây dựng một ứng dụng Web hoàn chỉnh.

### 13.1.2 Tổng quan kiến trúc kỹ thuật

Hệ thống sử dụng <strong>kiến trúc tách biệt front-end và back-end</strong> kinh điển, chia thành bốn tầng, như Hình 13.1 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-1.png" alt="" width="85%"/>
  <p>Hình 13.1 Kiến trúc kỹ thuật trợ lý du lịch thông minh</p>
</div>

<strong>(1) Tầng Front-end (Vue3+TypeScript)</strong>: phụ trách tương tác người dùng và hiển thị dữ liệu, bao gồm nhập form, hiển thị kết quả, trực quan hóa bản đồ.

<strong>(2) Tầng Back-end (FastAPI)</strong>: phụ trách định tuyến API, xác thực dữ liệu, logic nghiệp vụ.

<strong>(3) Tầng Agent (HelloAgents)</strong>: phụ trách phân rã tác vụ, gọi công cụ, tổng hợp kết quả. Bao gồm 4 Agent chuyên trách.

<strong>(4) Tầng dịch vụ bên ngoài</strong>: cung cấp dữ liệu và năng lực, bao gồm API Bản đồ Cao Đức (高德地图), Unsplash API, LLM API.

Quy trình luân chuyển dữ liệu như sau: người dùng điền form ở front-end → back-end xác thực dữ liệu → gọi hệ thống Agent → Agent lần lượt gọi các Agent tìm kiếm địa điểm, truy vấn thời tiết, gợi ý khách sạn, lập kế hoạch hành trình → mỗi Agent thông qua giao thức MCP gọi API bên ngoài → tổng hợp kết quả trả về front-end → front-end render hiển thị.

Cấu trúc tham khảo của dự án như sau, giúp dễ dàng định vị mã nguồn:
```
helloagents-trip-planner/
├── backend/                    # Mã nguồn back-end
│   ├── app/
│   │   ├── agents/            # Hiện thực Agent
│   │   ├── api/               # Định tuyến API
│   │   ├── models/            # Mô hình dữ liệu
│   │   ├── services/          # Tầng dịch vụ
│   │   └── config.py          # File cấu hình
│   └── requirements.txt       # Dependency Python
│
└── frontend/                   # Mã nguồn front-end
    ├── src/
    │   ├── views/             # Component trang
    │   ├── services/          # Dịch vụ API
    │   ├── types/             # Định nghĩa kiểu
    │   └── router/            # Cấu hình router
    └── package.json           # Dependency npm
```

Thiết kế kiến trúc và luân chuyển dữ liệu chi tiết sẽ được giới thiệu ở các phần sau.

### 13.1.3 Trải nghiệm nhanh: chạy dự án trong 5 phút

Trước khi đi sâu vào chi tiết hiện thực, hãy chạy thử dự án để xem hiệu quả cuối cùng. Như vậy bạn sẽ có một cái nhìn trực quan về toàn bộ hệ thống.

<strong>Yêu cầu môi trường:</strong>

- Python 3.10 trở lên
- Node.js 16.0 trở lên
- npm 8.0 trở lên

<strong>Lấy API Key:</strong>

Bạn cần chuẩn bị các API Key sau:

- API của LLM (OpenAI, DeepSeek, v.v.)
- Amap (Bản đồ Cao Đức) Web Service Key: truy cập https://console.amap.com/ để đăng ký và tạo ứng dụng
- Unsplash Access Key: truy cập https://unsplash.com/developers để đăng ký và tạo ứng dụng

Đặt tất cả API Key vào file `.env`.

Khởi động back-end:

```bash
# 1. Vào thư mục back-end
cd helloagents-trip-planner/backend

# 2. Cài đặt dependency
pip install -r requirements.txt

# 3. Cấu hình biến môi trường
cp .env.example .env
# Chỉnh sửa file .env, điền API Key của bạn vào

# 4. Khởi động dịch vụ back-end
uvicorn app.api.main:app --reload
# Hoặc
python run.py
```

Sau khi khởi động thành công, truy cập http://localhost:8000/docs để xem tài liệu API.

Mở một cửa sổ terminal mới:

```bash
# 1. Vào thư mục front-end
cd helloagents-trip-planner/frontend

# 2. Cài đặt dependency
npm install

# 3. Khởi động dịch vụ front-end
npm run dev
```

Sau khi khởi động thành công, truy cập http://localhost:5173 là có thể sử dụng ứng dụng.

Trải nghiệm chức năng cốt lõi:

Trước tiên cần điền vào form ở trang chủ các thông tin như thành phố điểm đến, ngày đi du lịch, sở thích, ngân sách, loại hình giao thông và lưu trú. Sau khi bấm nút "Bắt đầu lập kế hoạch", hệ thống sẽ hiển thị thanh tiến trình tải, và nhanh chóng tạo ra trang kết quả, như Hình 13.2 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-2.png" alt="" width="85%"/>
  <p>Hình 13.2 Trang đang lập kế hoạch của trợ lý du lịch</p>
</div>

Sau khi tải xong thành công, trang này sẽ hiển thị rõ ràng tổng quan hành trình, chi tiết ngân sách, bản đồ địa điểm tham quan, chi tiết hành trình từng ngày và thông tin thời tiết, như Hình 13.3, 13.4 thể hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-3.png" alt="" width="85%"/>
  <p>Hình 13.3 Trang hoàn thành lập kế hoạch của trợ lý du lịch</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-4.png" alt="" width="85%"/>
  <p>Hình 13.4 Trang hoàn thành lập kế hoạch của trợ lý du lịch</p>
</div>

Nếu người dùng cần điều chỉnh cá nhân hóa, có thể bấm nút "Chỉnh sửa hành trình", tự do điều chỉnh thứ tự địa điểm tham quan hoặc xóa một địa điểm nào đó, như Hình 13.5 thể hiện. Sau khi hoàn thành kế hoạch, thông qua menu thả xuống "Xuất hành trình", có thể dễ dàng lưu phương án cuối cùng thành hình ảnh hoặc file PDF, tiện tra cứu bất cứ lúc nào.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-5.png" alt="" width="85%"/>
  <p>Hình 13.5 Trang hoàn thành lập kế hoạch của trợ lý du lịch</p>
</div>

## 13.2 Thiết kế mô hình dữ liệu

### 13.2.1 Luân chuyển dữ liệu trong ứng dụng Web

Khi xây dựng trợ lý du lịch thông minh, chúng ta cần giải quyết một vấn đề cốt lõi: <strong>làm thế nào để biểu diễn và truyền tải dữ liệu kế hoạch du lịch?</strong>

Chúng ta cần hiểu rõ dữ liệu luân chuyển như thế nào trong một ứng dụng Web hoàn chỉnh. Hãy tưởng tượng, khi người dùng bấm nút "Bắt đầu lập kế hoạch" trên trình duyệt, điều gì sẽ xảy ra?

Dữ liệu form mà người dùng điền ở front-end (điểm đến, ngày đi, ngân sách, v.v.) cần được gửi tới máy chủ back-end thông qua HTTP request. Sau khi back-end nhận được dữ liệu, sẽ gọi hệ thống Agent để xử lý. Agent lại gọi các dịch vụ bên ngoài như API Bản đồ Cao Đức, Unsplash API để lấy dữ liệu. Định dạng dữ liệu trả về từ các API bên ngoài này khác nhau, có cái dùng `lng`, có cái dùng `lon`, có cái dùng `longitude`. Cuối cùng, back-end cần trả dữ liệu đã xử lý xong về cho front-end, front-end lại render thành trang mà người dùng nhìn thấy.

Trong quá trình này, dữ liệu trải qua nhiều lần chuyển đổi: form front-end → HTTP request → đối tượng Python ở back-end → response từ API bên ngoài → đối tượng Python ở back-end → HTTP response → đối tượng TypeScript ở front-end → hiển thị trang. Nếu không có định dạng dữ liệu thống nhất, mỗi bước chuyển đổi đều có thể xảy ra lỗi. Đây chính là lý do vì sao chúng ta cần <strong>mô hình dữ liệu</strong>.

### 13.2.2 Từ dictionary đến mô hình Pydantic

Hãy bắt đầu từ bản nguyên mẫu đơn giản ở chương 1. Trong nguyên mẫu đó, chúng ta sử dụng dictionary của Python để biểu diễn dữ liệu địa điểm tham quan:

```python
# Cách làm ở chương 1: dùng dictionary
attraction = {
    "name": "故宫",
    "location": {"lng": 116.397128,"lat": 39.916527},
    "price": 60
}

# Truy cập dữ liệu
lng = attraction["location"]["lng"]
```

Cách này rất tiện lợi ở giai đoạn làm nguyên mẫu, nhưng trong dự án thực tế sẽ gặp nhiều vấn đề. Đầu tiên là vấn đề <strong>tên trường (field) không thống nhất</strong>. Dữ liệu vị trí trả về từ API Bản đồ Cao Đức là một chuỗi kiểu `"116.397128，39.916527"`, cần tách thủ công thành kinh độ vĩ độ. Còn Unsplash API có thể dùng `longitude` và `latitude`. Nếu trong code chúng ta dùng dictionary khắp nơi, thì ở mỗi nơi đều phải xử lý những khác biệt này.

Thứ hai là vấn đề <strong>an toàn kiểu (type safety)</strong>. Giả sử chúng ta vô tình viết `price` thành chuỗi `"60"`, trong Python điều này sẽ không báo lỗi ngay lập tức, nhưng khi tính tổng ngân sách sẽ gặp vấn đề. Tệ hơn nữa, loại lỗi này chỉ có thể phát hiện khi chạy (runtime), và thông báo lỗi có thể rất khó định vị.

Cuối cùng là vấn đề <strong>khả năng bảo trì</strong>. Khi chúng ta cần thêm trường mới cho địa điểm tham quan (ví dụ `rating` điểm đánh giá), cần sửa ở nhiều nơi trong code. Nếu bỏ sót một chỗ nào đó, sẽ dẫn đến dữ liệu không nhất quán.

Pydantic cung cấp một giải pháp. Đây là thư viện xác thực dữ liệu của Python, cho phép chúng ta dùng class để định nghĩa cấu trúc dữ liệu, và tự động xử lý xác thực, chuyển đổi và tuần tự hóa (serialization). Hãy xem một ví dụ đơn giản:

```python
from pydantic import BaseModel,Field

class Location(BaseModel):
    longitude: float = Field(...,description="经度")
    latitude: float = Field(...,description="纬度")

class Attraction(BaseModel):
    name: str
    location: Location
    ticket_price: int = 0

# Tạo đối tượng
attraction = Attraction(
    name="故宫",
    location=Location(longitude=116.397128,latitude=39.916527),
    ticket_price=60
)

# Truy cập an toàn về kiểu
lng = attraction.location.longitude  # IDE sẽ cung cấp code completion
```

Làm như vậy có một số lợi ích. Trước hết, nếu chúng ta truyền vào kiểu dữ liệu sai (ví dụ đặt `ticket_price` là chuỗi), Pydantic sẽ ngay lập tức ném ra exception, cho biết chỗ nào bị sai. Thứ hai, IDE có thể dựa trên định nghĩa kiểu để cung cấp code completion và kiểm tra kiểu, giảm đáng kể lỗi gõ nhầm. Cuối cùng, khi cần sửa cấu trúc dữ liệu, chỉ cần sửa định nghĩa class, tất cả những nơi sử dụng class này sẽ tự động cập nhật.

### 13.2.3 Các khái niệm cốt lõi của Pydantic

Trước khi đi sâu thiết kế mô hình dữ liệu của chúng ta, hãy tìm hiểu một vài khái niệm cốt lõi của Pydantic. Nền tảng của Pydantic là class `BaseModel`, tất cả mô hình dữ liệu đều cần kế thừa class này. Mỗi trường (field) đều có thể chỉ định kiểu, Pydantic sẽ tự động kiểm tra và chuyển đổi kiểu.

Định nghĩa trường sử dụng hàm `Field`, có thể chỉ định giá trị mặc định, mô tả, quy tắc xác thực, v.v. `...` biểu thị trường này là bắt buộc, nếu khi tạo đối tượng không cung cấp trường này, Pydantic sẽ ném ra exception. Chúng ta cũng có thể dùng `Optional` để biểu thị trường tùy chọn, hoặc trực tiếp cung cấp giá trị mặc định.

```python
from pydantic import BaseModel,Field
from typing import Optional,List

class Attraction(BaseModel):
    name: str = Field(...,description="景点名称")  # Bắt buộc
    rating: float = Field(default=0.0,ge=0,le=5)  # Giá trị mặc định, xác thực phạm vi
    visit_duration: int = Field(default=60,gt=0)  # Lớn hơn 0
    description: Optional[str] = None  # Trường tùy chọn
```

Pydantic còn hỗ trợ mô hình lồng nhau (nested model) và danh sách (list). Chúng ta có thể dùng một mô hình khác làm kiểu của trường trong một mô hình, như vậy có thể xây dựng cấu trúc dữ liệu phức tạp. Ví dụ, một địa điểm tham quan chứa thông tin vị trí, một hành trình chứa nhiều địa điểm tham quan.

```python
class DayPlan(BaseModel):
    date: str
    attractions: List[Attraction]  # Danh sách địa điểm tham quan
    hotel: Optional[Hotel] = None  # Thông tin khách sạn tùy chọn
```

Một trong những chức năng mạnh mẽ nhất là <strong>validator tùy chỉnh</strong>. Đôi khi định dạng dữ liệu trả về từ API bên ngoài không phù hợp với yêu cầu của chúng ta, chúng ta có thể dùng decorator `field_validator` để tùy chỉnh logic xác thực và chuyển đổi. Ví dụ, nhiệt độ trả về từ Bản đồ Cao Đức là một chuỗi kiểu `"16°C"`, chúng ta cần chuyển nó thành số:

```python
from pydantic import field_validator

class WeatherInfo(BaseModel):
    temperature: int
    
    @field_validator('temperature',mode='before')
    def parse_temperature(cls,v):
        """解析温度字符串："16°C" -> 16"""
        if isinstance(v,str):
            v = v.replace('°C','').replace('℃','').strip()
            return int(v)
        return v
```

Validator này sẽ tự động thực thi trước khi tạo đối tượng, chuyển chuỗi thành số nguyên. Như vậy chúng ta không cần phải xử lý thủ công định dạng nhiệt độ này ở mọi nơi trong code.

### 13.2.4 Thiết kế mô hình từ dưới lên trên

Bây giờ hãy bắt đầu thiết kế mô hình dữ liệu cho trợ lý du lịch thông minh. Một nguyên tắc thiết kế tốt là <strong>từ dưới lên trên (bottom-up)</strong>: trước tiên định nghĩa mô hình cơ bản nhất, sau đó dần dần kết hợp thành cấu trúc phức tạp. Lợi ích của cách làm này là mỗi mô hình đều rất đơn giản, dễ hiểu và dễ bảo trì.

Mô hình cơ bản nhất là <strong>thông tin vị trí</strong>. Dù là địa điểm tham quan, khách sạn hay nhà hàng, đều cần thông tin vị trí. Chúng ta định nghĩa một class `Location` để biểu diễn tọa độ kinh độ vĩ độ:

```python
class Location(BaseModel):
    """位置信息(经纬度坐标)"""
    longitude: float = Field(...,description="经度",ge=-180,le=180)
    latitude: float = Field(...,description="纬度",ge=-90,le=90)
```

Ở đây chúng ta sử dụng xác thực phạm vi (`ge` biểu thị lớn hơn hoặc bằng, `le` biểu thị nhỏ hơn hoặc bằng), đảm bảo giá trị kinh độ vĩ độ nằm trong phạm vi hợp lý.

Tiếp theo là <strong>thông tin địa điểm tham quan</strong>. Một địa điểm tham quan bao gồm tên, địa chỉ, vị trí, thời gian tham quan, mô tả, đánh giá, hình ảnh và giá vé, v.v. Lưu ý chúng ta đã dùng `Location` làm kiểu trường, đây chính là mô hình lồng nhau:

```python
class Attraction(BaseModel):
    """景点信息"""
    name: str = Field(...,description="景点名称")
    address: str = Field(...,description="地址")
    location: Location = Field(...,description="经纬度坐标")
    visit_duration: int = Field(...,description="建议游览时间(分钟)",gt=0)
    description: str = Field(...,description="景点描述")
    category: Optional[str] = Field(default="景点",description="景点类别")
    rating: Optional[float] = Field(default=None,ge=0,le=5,description="评分")
    image_url: Optional[str] = Field(default=None,description="图片URL")
    ticket_price: int = Field(default=0,ge=0,description="门票价格(元)")
```

Tương tự, chúng ta định nghĩa <strong>thông tin ẩm thực</strong> và <strong>thông tin khách sạn</strong>. Cấu trúc của các mô hình này đều khá giống nhau, đều chứa các thông tin cơ bản như tên, địa chỉ, vị trí và chi phí:

```python
class Meal(BaseModel):
    """餐饮信息"""
    type: str = Field(...,description="餐饮类型：breakfast/lunch/dinner/snack")
    name: str = Field(...,description="餐饮名称")
    address: Optional[str] = Field(default=None,description="地址")
    location: Optional[Location] = Field(default=None,description="经纬度坐标")
    description: Optional[str] = Field(default=None,description="描述")
    estimated_cost: int = Field(default=0,description="预估费用(元)")

class Hotel(BaseModel):
    """酒店信息"""
    name: str = Field(...,description="酒店名称")
    address: str = Field(default="",description="酒店地址")
    location: Optional[Location] = Field(default=None,description="酒店位置")
    price_range: str = Field(default="",description="价格范围")
    rating: str = Field(default="",description="评分")
    distance: str = Field(default="",description="距离景点距离")
    type: str = Field(default="",description="酒店类型")
    estimated_cost: int = Field(default=0,description="预估费用(元/晚)")
```

<strong>Thông tin ngân sách</strong> là một mô hình đặc biệt, nó không chứa thông tin vị trí, mà chứa tổng hợp các khoản chi phí:

```python
class Budget(BaseModel):
    """预算信息"""
    total_attractions: int = Field(default=0,description="景点门票总费用")
    total_hotels: int = Field(default=0,description="酒店总费用")
    total_meals: int = Field(default=0,description="餐饮总费用")
    total_transportation: int = Field(default=0,description="交通总费用")
    total: int = Field(default=0,description="总费用")
```

Bây giờ chúng ta có thể kết hợp các mô hình cơ bản này, xây dựng <strong>hành trình một ngày</strong>. Một hành trình một ngày bao gồm ngày tháng, mô tả, phương thức giao thông, sắp xếp lưu trú, khách sạn, danh sách địa điểm tham quan và danh sách ẩm thực:

```python
class DayPlan(BaseModel):
    """单日行程"""
    date: str = Field(...,description="日期")
    day_index: int = Field(...,description="第几天(从0开始)")
    description: str = Field(...,description="当日行程描述")
    transportation: str = Field(...,description="交通方式")
    accommodation: str = Field(...,description="住宿安排")
    hotel: Optional[Hotel] = Field(default=None,description="酒店信息")
    attractions: List[Attraction] = Field(default_factory=list,description="景点列表")
    meals: List[Meal] = Field(default_factory=list,description="餐饮安排")
```

Lưu ý ở đây đã dùng `List[Attraction]` để biểu diễn danh sách địa điểm tham quan, `default_factory=list` biểu thị giá trị mặc định là một danh sách rỗng.

<strong>Thông tin thời tiết</strong> cần xử lý đặc biệt, vì định dạng nhiệt độ trả về từ Bản đồ Cao Đức không chuẩn. Chúng ta dùng validator tùy chỉnh để xử lý:

```python
class WeatherInfo(BaseModel):
    """天气信息"""
    date: str = Field(...,description="日期")
    day_weather: str = Field(...,description="白天天气")
    night_weather: str = Field(...,description="夜间天气")
    day_temp: int = Field(...,description="白天温度(摄氏度)")
    night_temp: int = Field(...,description="夜间温度(摄氏度)")
    wind_direction: str = Field(...,description="风向")
    wind_power: str = Field(...,description="风力")
    
    @field_validator('day_temp','night_temp',mode='before')
    def parse_temperature(cls,v):
        """解析温度字符串："16°C" -> 16"""
        if isinstance(v,str):
            v = v.replace('°C','').replace('℃','').replace('°','').strip()
            try:
                return int(v)
            except ValueError:
                return 0  # Xử lý khoan dung lỗi
        return v
```

Cuối cùng, chúng ta định nghĩa <strong>kế hoạch du lịch hoàn chỉnh</strong>. Đây là mô hình cấp cao nhất, chứa tất cả thông tin:

```python
class TripPlan(BaseModel):
    """旅行计划"""
    city: str = Field(...,description="目的地城市")
    start_date: str = Field(...,description="开始日期")
    end_date: str = Field(...,description="结束日期")
    days: List[DayPlan] = Field(default_factory=list,description="每日行程")
    weather_info: List[WeatherInfo] = Field(default_factory=list,description="天气信息")
    overall_suggestions: str = Field(...,description="总体建议")
    budget: Optional[Budget] = Field(default=None,description="预算信息")
```

Như vậy, chúng ta đã hoàn thành toàn bộ thiết kế mô hình dữ liệu. Từ `Location` cơ bản nhất, đến `Attraction`, `Meal`, `Hotel`, rồi đến `DayPlan`, cuối cùng đến `TripPlan`, hình thành một cấu trúc phân tầng rõ ràng.

### 13.2.5 Ứng dụng mô hình dữ liệu trong ứng dụng Web

Bây giờ hãy xem những mô hình dữ liệu này được sử dụng như thế nào trong ứng dụng Web thực tế. Trong FastAPI, mô hình Pydantic có thể dùng trực tiếp làm định nghĩa kiểu cho request và response. FastAPI sẽ tự động thực hiện xác thực dữ liệu, tuần tự hóa và tạo tài liệu.

```python
from fastapi import FastAPI
from app.models.schemas import TripPlanRequest,TripPlan

app = FastAPI()

@app.post("/api/trip/plan",response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    """
    创建旅行计划
    
    FastAPI自动：
    1. 验证请求数据(TripPlanRequest)
    2. 验证响应数据(TripPlan)
    3. 生成OpenAPI文档
    """
    trip_plan = await generate_trip_plan(request)
    return trip_plan
```

Khi người dùng gửi POST request tới `/api/trip/plan`, FastAPI sẽ tự động chuyển đổi dữ liệu JSON thành đối tượng `TripPlanRequest`. Nếu định dạng dữ liệu không đúng (ví dụ thiếu trường bắt buộc, hoặc kiểu không khớp), FastAPI sẽ tự động trả về lỗi 400, và cho biết người dùng chỗ nào bị sai.

Ở front-end, chúng ta cũng cần định nghĩa kiểu TypeScript tương ứng. Mặc dù TypeScript và Python là ngôn ngữ khác nhau, nhưng cấu trúc dữ liệu là giống nhau:

```typescript
interface Location {
  longitude: number;
  latitude: number;
}

interface Attraction {
  name: string;
  address: string;
  location: Location;
  visit_duration: number;
  ticket_price: number;
}

interface TripPlan {
  city: string;
  start_date: string;
  end_date: string;
  days: DayPlan[];
}
```

Như vậy, front-end và back-end đã sử dụng định dạng dữ liệu thống nhất. Khi back-end trả về đối tượng `TripPlan`, front-end có thể dùng trực tiếp, không cần bất kỳ chuyển đổi nào. Việc kiểm tra kiểu của TypeScript cũng giúp chúng ta tránh được rất nhiều lỗi.

## 13.3 Thiết kế phối hợp đa Agent

### 13.3.1 Vì sao cần nhiều Agent

Ở chương 7, chúng ta đã học cách dùng SimpleAgent để xây dựng Agent. Triết lý thiết kế của SimpleAgent là đơn giản trực tiếp: mỗi lần gọi phương thức `run()`, Agent sẽ phân tích câu hỏi của người dùng, quyết định có cần gọi công cụ (tool) hay không, sau đó trả về kết quả. Cách thiết kế này rất hiệu quả khi xử lý các tác vụ đơn giản, nhưng khi đối mặt với tác vụ như lập kế hoạch du lịch, sẽ gặp một số vấn đề.

Nếu dùng một Agent duy nhất để hoàn thành việc lập kế hoạch du lịch. Agent này cần làm gì? Trước tiên, nó phải tìm kiếm thông tin địa điểm tham quan, điều này cần gọi công cụ tìm kiếm POI của Bản đồ Cao Đức. Sau đó, nó phải truy vấn thông tin thời tiết, điều này cần gọi công cụ truy vấn thời tiết. Tiếp theo, nó phải tìm kiếm thông tin khách sạn, điều này lại cần gọi công cụ tìm kiếm POI. Cuối cùng, nó phải tổng hợp tất cả những thông tin này lại, tạo ra một kế hoạch du lịch hoàn chỉnh.

Nghe có vẻ đơn giản, nhưng khi thực hiện thực tế sẽ gặp vấn đề đầu tiên: <strong>giới hạn của việc gọi công cụ</strong>. Mỗi lần gọi `run()` của SimpleAgent chỉ có thể thực thi một công cụ. Điều này có nghĩa là chúng ta cần gọi phương thức `run()` nhiều lần, mỗi lần xử lý một tác vụ. Nhưng làm vậy sẽ phát sinh một vấn đề mới: làm thế nào để truyền thông tin giữa các lần gọi? Thông tin địa điểm tham quan thu được ở lần gọi đầu tiên, làm sao truyền cho lần gọi thứ hai? Chúng ta cần tự quản lý những kết quả trung gian này, code sẽ trở nên rất phức tạp.

Tất nhiên, chúng ta có thể dùng ReactAgent để giải quyết vấn đề này. ReactAgent có thể thực thi nhiều công cụ trong một lần gọi, nó sẽ tự động tiến hành nhiều vòng suy nghĩ và hành động. Nhưng điều này lại phát sinh vấn đề mới: <strong>chi phí thời gian</strong>. Mỗi vòng suy nghĩ của ReactAgent đều cần gọi LLM, nếu cần gọi ba công cụ, thì cần ít nhất ba vòng suy nghĩ, nghĩa là ít nhất ba lần gọi LLM. Và những lần gọi này là tuần tự, phải đợi cái trước hoàn thành mới có thể bắt đầu cái sau, tổng thời gian sẽ rất lâu.

Vấn đề thứ hai là <strong>độ phức tạp của prompt</strong>. Nếu chúng ta để một Agent hoàn thành tất cả tác vụ, thì cần mô tả chi tiết logic thực thi của từng tác vụ trong prompt. Ví dụ:

```python
COMPLEX_PROMPT = """你是旅行规划助手。你需要：
1. 使用maps_text_search搜索景点，关键词根据用户偏好确定
2. 使用maps_weather查询天气,获取未来几天的天气预报
3. 使用maps_text_search搜索酒店,类型根据用户需求确定
4. 整合所有信息生成旅行计划,包括每天的景点、餐饮、住宿安排
注意：必须按顺序执行,每个工具只能调用一次,输出必须是JSON格式...
"""
```

Prompt kiểu này có một số vấn đề. Trước hết là <strong>khó bảo trì</strong>. Nếu chúng ta muốn sửa logic tìm kiếm địa điểm tham quan (ví dụ thêm lọc theo đánh giá), thì cần sửa toàn bộ prompt, rất dễ ảnh hưởng đến các phần khác. Thứ hai là <strong>dễ gây lỗi</strong>. LLM cần hiểu đồng thời yêu cầu của nhiều tác vụ, rất dễ nhầm lẫn định dạng và tham số của các tác vụ khác nhau. Cuối cùng là <strong>khó debug</strong>. Khi kế hoạch tạo ra không như mong đợi, chúng ta rất khó biết khâu nào xảy ra vấn đề, là tìm kiếm địa điểm không chính xác, hay truy vấn thời tiết thất bại, hay logic tổng hợp có vấn đề?

Đối mặt với những vấn đề này, một ý tưởng tự nhiên là: có thể phân rã tác vụ phức tạp thành nhiều tác vụ đơn giản, để các Agent khác nhau mỗi người phụ trách một phần không? Đây chính là tư tưởng cốt lõi của việc phối hợp đa Agent (multi-agent collaboration).

Hãy tưởng tượng một công ty du lịch trong thế giới thực. Khi bạn đến công ty du lịch tư vấn kế hoạch du lịch, sẽ không chỉ có một người phục vụ bạn. Thường sẽ có cố vấn địa điểm tham quan chuyên trách, phụ trách gợi ý địa điểm; có cố vấn khách sạn, phụ trách đặt phòng; còn có người lập kế hoạch hành trình, phụ trách tổng hợp tất cả thông tin thành hành trình hoàn chỉnh. Mỗi người đều tập trung vào lĩnh vực mà mình giỏi, cuối cùng do người lập kế hoạch hành trình tổng hợp tất cả thông tin lại. Cách phân công phối hợp này hiệu quả hơn nhiều so với việc để một người làm tất cả mọi việc.

### 13.3.2 Thiết kế vai trò Agent

Dựa trên nguyên tắc phân rã tác vụ, chúng ta thiết kế bốn Agent chuyên trách, như Hình 13.6 thể hiện:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-6.png" alt="" width="85%"/>
  <p>Hình 13.6 Quy trình phối hợp đa Agent</p>
</div>

- <strong>AttractionSearchAgent (chuyên gia tìm kiếm địa điểm tham quan)</strong> chuyên tập trung tìm kiếm thông tin địa điểm tham quan. Nó chỉ cần hiểu sở thích của người dùng (ví dụ "lịch sử văn hóa", "phong cảnh thiên nhiên"), sau đó gọi công cụ tìm kiếm POI của Bản đồ Cao Đức, trả về danh sách địa điểm tham quan liên quan. Prompt của nó rất đơn giản, chỉ cần nói rõ cách chọn từ khóa theo sở thích, cách gọi công cụ.

- <strong>WeatherQueryAgent (chuyên gia truy vấn thời tiết)</strong> chuyên tập trung truy vấn thông tin thời tiết. Nó chỉ cần biết tên thành phố, sau đó gọi công cụ truy vấn thời tiết, trả về dự báo thời tiết vài ngày tới. Nhiệm vụ của nó rất rõ ràng, hầu như không thể sai.

- <strong>HotelAgent (chuyên gia gợi ý khách sạn)</strong> chuyên tập trung tìm kiếm thông tin khách sạn. Nó cần hiểu nhu cầu lưu trú của người dùng (ví dụ "kinh tế", "sang trọng"), sau đó gọi công cụ tìm kiếm POI, trả về danh sách khách sạn phù hợp yêu cầu.

- <strong>PlannerAgent (chuyên gia lập kế hoạch hành trình)</strong> phụ trách tổng hợp tất cả thông tin. Nó nhận đầu ra của ba Agent trước, cộng thêm yêu cầu gốc của người dùng (ngày đi, ngân sách, v.v.), sau đó tạo ra kế hoạch du lịch hoàn chỉnh. Nó không cần gọi bất kỳ công cụ bên ngoài nào, chỉ cần tập trung vào việc tổng hợp thông tin và sắp xếp hành trình.

Bây giờ hãy thiết kế chi tiết vai trò và prompt của từng Agent. Khi thiết kế prompt, chúng ta cần cân nhắc một vài vấn đề then chốt: Agent này cần đầu vào gì? Nó nên tạo ra đầu ra gì? Nó cần gọi công cụ gì? Nó có thể gặp vấn đề gì?

Nhiệm vụ của <strong>AttractionSearchAgent</strong> là tìm kiếm địa điểm tham quan theo sở thích người dùng. Đầu vào của nó là tên thành phố và sở thích người dùng (ví dụ "lịch sử văn hóa", "phong cảnh thiên nhiên"). Nó cần gọi công cụ `amap_maps_text_search`, tham số là từ khóa và thành phố. Đầu ra của nó là danh sách địa điểm tham quan, bao gồm tên, địa chỉ, đánh giá và các thông tin khác.

```python
ATTRACTION_AGENT_PROMPT = """你是景点搜索专家。

**工具调用格式:**
`[TOOL_CALL:amap_maps_text_search:keywords=景点,city=城市名]`

**示例:**
- `[TOOL_CALL:amap_maps_text_search:keywords=景点,city=北京]`
- `[TOOL_CALL:amap_maps_text_search:keywords=博物馆,city=上海]`

**重要:**
- 必须使用工具搜索,不要编造信息
- 根据用户偏好({preferences})搜索{city}的景点
"""
```

Prompt này rất ngắn gọn, nhưng chứa tất cả thông tin cần thiết. Nó nói rõ định dạng gọi công cụ, cung cấp ví dụ cụ thể, còn nhấn mạnh hai nguyên tắc quan trọng: phải sử dụng công cụ (không được bịa đặt thông tin), phải tìm kiếm theo sở thích người dùng.

Nhiệm vụ của <strong>WeatherQueryAgent</strong> đơn giản hơn, chỉ cần truy vấn thời tiết. Đầu vào của nó là tên thành phố, đầu ra là thông tin thời tiết.

```python
WEATHER_AGENT_PROMPT = """你是天气查询专家。

**工具调用格式:**
`[TOOL_CALL:amap_maps_weather:city=城市名]`

请查询{city}的天气信息。
"""
```

Nhiệm vụ của <strong>HotelAgent</strong> là tìm kiếm khách sạn. Đầu vào của nó là tên thành phố và loại hình lưu trú, đầu ra là danh sách khách sạn.

```python
HOTEL_AGENT_PROMPT = """你是酒店推荐专家。

**工具调用格式:**
`[TOOL_CALL:amap_maps_text_search:keywords=酒店,city=城市名]`

请搜索{city}的{accommodation}酒店。
"""
```

<strong>PlannerAgent</strong> phức tạp nhất, vì nó cần tổng hợp tất cả thông tin. Đầu vào của nó là yêu cầu người dùng và đầu ra của ba Agent trước, đầu ra là kế hoạch du lịch hoàn chỉnh (định dạng JSON).

```python
PLANNER_AGENT_PROMPT = """你是行程规划专家。

**输出格式:**
严格按照以下JSON格式返回:
{
  "city": "城市名称",
  "start_date": "YYYY-MM-DD",
  "end_date": "YYYY-MM-DD",
  "days": [...],
  "weather_info": [...],
  "overall_suggestions": "总体建议",
  "budget": {...}
}

**规划要求:**
1. weather_info必须包含每天的天气
2. 温度为纯数字(不带°C)
3. 每天安排2-3个景点
4. 考虑景点距离和游览时间
5. 包含早中晚三餐
6. 提供实用建议
7. 包含预算信息
"""
```

### 13.3.3 Quy trình phối hợp Agent

Bây giờ hãy xem bốn Agent này phối hợp như thế nào để hoàn thành tác vụ lập kế hoạch du lịch. Toàn bộ quy trình có thể chia thành năm bước:

```python
class TripPlannerAgent:
    def __init__(self):
        self.attraction_agent = SimpleAgent(name="景点搜索"prompt=ATTRACTION_PROMPT)
        self.weather_agent = SimpleAgent(name="天气查询", prompt=WEATHER_PROMPT)
        self.hotel_agent = SimpleAgent(name="酒店推荐", prompt=HOTEL_PROMPT)
        self.planner_agent = SimpleAgent(name="行程规划", prompt=PLANNER_PROMPT)

    def plan_trip(self, request: TripPlanRequest) -> TripPlan:
        # 步骤1: 景点搜索
        attraction_response = self.attraction_agent.run(
            f"请搜索{request.city}的{request.preferences}景点"
        )

        # 步骤2: 天气查询
        weather_response = self.weather_agent.run(
            f"请查询{request.city}的天气"
        )

        # 步骤3: 酒店推荐
        hotel_response = self.hotel_agent.run(
            f"请搜索{request.city}的{request.accommodation}酒店"
        )

        # 步骤4: 整合生成计划
        planner_query = self._build_planner_query(
            request, attraction_response, weather_response, hotel_response
        )
        planner_response = self.planner_agent.run(planner_query)

        # 步骤5: 解析JSON
        trip_plan = self._parse_trip_plan(planner_response)
        return trip_plan
```

Quy trình này thực thi tuần tự bốn bước, đầu ra của mỗi bước là đầu vào của bước tiếp theo. Lưu ý chúng ta đã sử dụng hai mô hình Pydantic `TripPlanRequest` và `TripPlan`, đây là những mô hình được định nghĩa ở mục 13.2.

### 13.3.4 Xây dựng truy vấn (query)

PlannerAgent cần tổng hợp tất cả thông tin, truy vấn này cần chứa tất cả thông tin cần thiết, và phải được tổ chức rõ ràng, mạch lạc, để LLM có thể hiểu chính xác.

```python
def _build_planner_query(
    self,
    request: TripPlanRequest,
    attraction_response: str,
    weather_response: str,
    hotel_response: str
) -> str:
    """构建规划Agent的查询"""
    return f"""
请根据以下信息生成{request.city}的{request.days}日旅行计划:

**用户需求:**
- 目的地: {request.city}
- 日期: {request.start_date} 至 {request.end_date}
- 天数: {request.days}天
- 偏好: {request.preferences}
- 预算: {request.budget}
- 交通方式: {request.transportation}
- 住宿类型: {request.accommodation}

**景点信息:**
{attraction_response}

**天气信息:**
{weather_response}

**酒店信息:**
{hotel_response}

请生成详细的旅行计划,包括每天的景点安排、餐饮推荐、住宿信息和预算明细。
"""
```

Thông qua thiết kế phối hợp đa Agent này, chúng ta đã phân rã một tác vụ lập kế hoạch du lịch phức tạp thành bốn tác vụ con đơn giản. Mỗi Agent đều tập trung vào lĩnh vực mình giỏi, cũng đặt nền tảng tốt cho việc mở rộng chức năng trong tương lai (ví dụ thêm Agent gợi ý nhà hàng, Agent lập kế hoạch giao thông).

## 13.4 Giải thích chi tiết tích hợp công cụ MCP

### 13.4.1 Vì sao không gọi trực tiếp API

Ở mục 13.3, chúng ta đã thiết kế bốn Agent để phối hợp hoàn thành tác vụ lập kế hoạch du lịch. Trong đó AttractionSearchAgent, WeatherQueryAgent và HotelAgent đều cần gọi API của Bản đồ Cao Đức để lấy dữ liệu. Một câu hỏi tự nhiên là: tại sao không gọi trực tiếp HTTP API của Bản đồ Cao Đức trong Agent?

Hãy xem trước việc gọi trực tiếp API sẽ như thế nào. Bản đồ Cao Đức cung cấp API tìm kiếm POI, chúng ta cần tạo HTTP request, truyền tham số, phân tích response:

```python
import requests

def search_poi(keywords: str,city: str,api_key: str):
    """直接调用高德地图POI搜索API"""
    url = "https://restapi.amap.com/v3/place/text"
    params = {
        "keywords": keywords,
        "city": city,
        "key": api_key,
        "output": "json"
    }
    response = requests.get(url,params=params)
    data = response.json()
    return data
```

Cách này trông có vẻ đơn giản, nhưng khi sử dụng thực tế sẽ gặp một số vấn đề. Trước hết là <strong>Agent không thể tự chủ gọi</strong>. Trong framework HelloAgents của chúng ta, Agent thông qua nhận diện ký hiệu gọi công cụ trong prompt (ví dụ `[TOOL_CALL:tool_name:arg1=value1]`) để gọi công cụ. Nếu chúng ta gọi trực tiếp API trong code, Agent sẽ mất đi khả năng tự chủ quyết định, trở thành một lời gọi hàm đơn giản.

Thứ hai là <strong>truyền tham số phức tạp</strong>. API của Bản đồ Cao Đức có rất nhiều tham số, ví dụ tìm kiếm POI có `keywords`, `city`, `types`, `offset`, `page` và cả chục tham số khác. Nếu muốn Agent có thể sử dụng linh hoạt những tham số này, thì cần mô tả chi tiết ý nghĩa và định dạng của từng tham số trong prompt, điều này sẽ khiến prompt trở nên rất phức tạp.

Thứ ba là <strong>phân tích response khó khăn</strong>. API của Bản đồ Cao Đức trả về dữ liệu định dạng JSON, cấu trúc khá phức tạp. Chúng ta cần viết code để phân tích dữ liệu này, trích xuất các trường cần thiết. Nếu định dạng response của API thay đổi, chúng ta cần sửa code phân tích.

Cuối cùng là <strong>quản lý công cụ lộn xộn</strong>. Bản đồ Cao Đức cung cấp cả chục API khác nhau (tìm kiếm POI, truy vấn thời tiết, lập tuyến đường, v.v.), nếu chúng ta viết một hàm cho mỗi API, rồi thủ công đăng ký vào danh sách công cụ của Agent, code sẽ trở nên rất dài dòng. Và khi muốn thêm API mới, cần sửa nhiều nơi.

### 13.4.2 Tích hợp MCP Bản đồ Cao Đức

MCP (Model Context Protocol) là giao thức chuẩn hóa do Anthropic đề xuất, dùng để kết nối LLM và công cụ bên ngoài. Mục này sẽ giới thiệu cách tích hợp máy chủ MCP Bản đồ Cao Đức vào dự án. Dự án của chúng ta dùng `amap-mcp-server`, đây là một máy chủ MCP được hiện thực bằng Node.js:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-7.png" alt="" width="85%"/>
  <p>Hình 13.7 Công cụ amap-mcp-server</p>
</div>

Máy chủ MCP Bản đồ Cao Đức cung cấp nhiều loại công cụ, chủ yếu được chia thành các nhóm sau, như Bảng 13.1 thể hiện:

<div align="center">
  <p>Bảng 13.1 Phân loại công cụ MCP Bản đồ Cao Đức</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-1.png" alt="" width="85%"/>
</div>

Thông qua giao thức MCP, chúng ta có thể tích hợp rất thuận tiện vào HelloAgents:

```python
from hello_agents.tools import MCPTool
from app.config import get_settings

settings = get_settings()

# 创建MCP工具
mcp_tool = MCPTool(
    name="amap_mcp",
    command="npx",
    args=["-y", "@sugarforever/amap-mcp-server"],
    env={"AMAP_API_KEY": settings.amap_api_key},
    auto_expand=True
)
```

Đoạn code này đã làm gì? Trước hết, `command` và `args` chỉ định cách khởi động máy chủ MCP. `npx -y @sugarforever/amap-mcp-server` sẽ tải xuống và chạy package `amap-mcp-server` từ kho npm. Tham số `env` truyền biến môi trường, ở đây chúng ta truyền API Key của Bản đồ Cao Đức.

**Lưu ý:** một số ví dụ trong tài liệu này sử dụng `npx` để khởi động dịch vụ MCP (Model Context Protocol). Còn trong kho mã của mục này, chúng ta thực tế sử dụng cách `uvx`. Cần nói rõ rằng, `npx` và `uvx` về mặt triết lý thiết kế có mức độ nhất quán cao, khác biệt chỉ nằm ở hệ sinh thái mà chúng hướng tới, `npx` hướng tới JavaScript/Node.js (package đến từ npm), còn `uvx` hướng tới Python (package đến từ PyPI). Hai cách này không có sự hơn kém, mọi người có thể lựa chọn tùy theo nhu cầu khi sử dụng.

Khi chúng ta tạo đối tượng `MCPTool`, nó sẽ khởi động tiến trình máy chủ MCP ở phía sau, và giao tiếp với máy chủ thông qua đầu vào/đầu ra chuẩn (stdin/stdout). Đây là một đặc điểm của giao thức MCP: sử dụng giao tiếp liên tiến trình (IPC) thay vì HTTP, như vậy hiệu quả hơn, cũng dễ quản lý hơn.

Tham số quan trọng nhất là `auto_expand=True`. Khi đặt là True, `MCPTool` sẽ tự động truy vấn máy chủ MCP cung cấp những công cụ nào, sau đó tạo một đối tượng Tool độc lập cho mỗi công cụ. Đây chính là lý do tại sao chúng ta chỉ tạo một `MCPTool`, nhưng Agent lại có được 16 công cụ. Hãy xem quá trình này:

```python
# 创建一个MCPTool
mcp_tool = MCPTool(..., auto_expand=True)
agent.add_tool(mcp_tool)

# Agent实际上获得了16个工具！
print(list(agent.tools.keys()))
# ['amap_maps_text_search', 'amap_maps_weather', ...]
```

Như Hình 13.8 thể hiện, giả sử người dùng muốn tìm kiếm địa điểm tham quan ở Bắc Kinh, AttractionSearchAgent nhận được truy vấn "请搜索北京的历史文化景点" (Vui lòng tìm kiếm các địa điểm lịch sử văn hóa ở Bắc Kinh). Agent phân tích truy vấn này, quyết định gọi công cụ `amap_maps_text_search`, tham số là `keywords=景点，city=北京`.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-8.png" alt="" width="85%"/>
  <p>Hình 13.8 Quy trình gọi công cụ MCP</p>
</div>

Agent sinh ra ký hiệu gọi công cụ: `[TOOL_CALL:amap_maps_text_search:keywords=景点，city=北京]`. Framework HelloAgents phân tích ký hiệu này, trích xuất tên công cụ và tham số, sau đó gọi đối tượng Tool tương ứng.

Đối tượng Tool là do `MCPTool` tự động tạo ra, nó sẽ gửi yêu cầu gọi tới máy chủ MCP. Cụ thể, nó sẽ tạo một thông điệp định dạng JSON-RPC, gửi tới tiến trình máy chủ thông qua stdin:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "amap_maps_text_search",
    "arguments": {
      "keywords": "景点",
      "city": "北京"
    }
  }
}
```

Máy chủ MCP nhận được thông điệp này, phân tích tham số, sau đó gọi HTTP API của Bản đồ Cao Đức. Nó sẽ tạo HTTP request, thêm API Key, gửi request, nhận response.

API Bản đồ Cao Đức trả về dữ liệu định dạng JSON, chứa danh sách địa điểm tham quan, địa chỉ, tọa độ và các thông tin khác. Máy chủ MCP phân tích dữ liệu này, trích xuất các trường then chốt, sau đó tạo thông điệp response, trả về cho `MCPTool` thông qua stdout:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "找到以下景点：\n1. 故宫博物院 - 地址：东城区景山前街4号\n2. 天坛公园 - 地址：东城区天坛路\n..."
      }
    ]
  }
}
```

`MCPTool` nhận được response, trích xuất nội dung văn bản, trả về cho Agent. Agent lấy kết quả này làm đầu ra của lần gọi công cụ, tiếp tục sinh ra phản hồi cuối cùng.

Quy trình này trông có vẻ phức tạp, nhưng đối với Agent, nó chỉ cần biết có một công cụ tên `amap_maps_text_search`, có thể tìm kiếm địa điểm tham quan. Tất cả chi tiết bên dưới đều được giao thức MCP và `MCPTool` đóng gói lại.

### 13.4.3 Chia sẻ instance MCP

Trong hệ thống đa Agent của chúng ta, có ba Agent đều cần dùng công cụ Bản đồ Cao Đức. Vậy mỗi Agent nên tạo instance `MCPTool` riêng của mình, hay chia sẻ cùng một instance?

Nếu mỗi Agent đều tạo một instance `MCPTool`, điều này có nghĩa là sẽ có ba tiến trình máy chủ chạy đồng thời. Mỗi tiến trình đều độc lập gọi API Bản đồ Cao Đức, điều này có thể vượt quá giới hạn tốc độ (rate limit) của API. Và nhiều tiến trình sẽ chiếm dụng nhiều bộ nhớ và CPU hơn.

Cách làm tốt hơn là để tất cả Agent chia sẻ cùng một instance `MCPTool`. Như vậy chỉ cần khởi động một tiến trình máy chủ MCP, tất cả các lần gọi API đều thông qua tiến trình này. Điều này không chỉ tiết kiệm tài nguyên, mà còn kiểm soát tần suất gọi API tốt hơn.

Trong code, chúng ta tạo một instance `MCPTool` trong constructor của `TripPlannerAgent`, sau đó thêm nó vào danh sách công cụ của mỗi Agent con:

```python
class TripPlannerAgent:
    def __init__(self):
        settings = get_settings()
        self.llm = HelloAgentsLLM()

        # 创建共享的MCP工具实例(只创建一次)
        self.mcp_tool = MCPTool(
            name="amap_mcp",
            command="npx",
            args=["-y", "@sugarforever/amap-mcp-server"],
            env={"AMAP_API_KEY": settings.amap_api_key},
            auto_expand=True
        )

        # 创建多个Agent,共享同一个MCP工具
        self.attraction_agent = SimpleAgent(
            name="AttractionSearchAgent",
            llm=self.llm,
            system_prompt=ATTRACTION_AGENT_PROMPT
        )
        self.attraction_agent.add_tool(self.mcp_tool)  # 共享

        self.weather_agent = SimpleAgent(
            name="WeatherQueryAgent",
            llm=self.llm,
            system_prompt=WEATHER_AGENT_PROMPT
        )
        self.weather_agent.add_tool(self.mcp_tool)  # 共享

        self.hotel_agent = SimpleAgent(
            name="HotelAgent",
            llm=self.llm,
            system_prompt=HOTEL_AGENT_PROMPT
        )
        self.hotel_agent.add_tool(self.mcp_tool)  # 共享
```

Như vậy, cả ba Agent đều có thể sử dụng 16 công cụ của Bản đồ Cao Đức, nhưng bên dưới chỉ có một tiến trình máy chủ MCP đang chạy. Khi chúng ta gọi phương thức `plan_trip` của `TripPlannerAgent`, ba Agent sẽ lần lượt gọi công cụ, tất cả các request đều thông qua cùng một máy chủ MCP gửi tới API Bản đồ Cao Đức.

### 13.4.4 Tích hợp Unsplash Image API

Ngoài Bản đồ Cao Đức, chúng ta còn cần lấy hình ảnh cho địa điểm tham quan, giúp kế hoạch du lịch sinh động trực quan hơn. Chúng ta dùng Unsplash API để tìm kiếm hình ảnh địa điểm tham quan. Cần lưu ý, Unsplash là dịch vụ nước ngoài, và là một trong số ít API hình ảnh có thể sử dụng miễn phí, vì vậy kết quả tìm kiếm có thể không đủ chính xác. Trong dự án thực tế, có thể cân nhắc dùng API hình ảnh POI của Bing, Baidu hoặc Bản đồ Cao Đức, nhưng những dịch vụ này thường cần trả phí.

Việc tích hợp Unsplash API khá đơn giản, chúng ta tạo một class `UnsplashService` để đóng gói lời gọi API:

```python
# backend/app/services/unsplash_service.py
import requests
from typing import Optional, List, Dict
import logging

logger = logging.getLogger(__name__)

class UnsplashService:
    """Unsplash图片服务"""

    def __init__(self, access_key: str):
        self.access_key = access_key
        self.base_url = "https://api.unsplash.com"

    def search_photos(self, query: str, per_page: int = 10) -> List[Dict]:
        """搜索图片"""
        try:
            url = f"{self.base_url}/search/photos"
            params = {
                "query": query,
                "per_page": per_page,
                "client_id": self.access_key
            }

            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()

            data = response.json()
            results = data.get("results", [])

            # 提取图片URL
            photos = []
            for result in results:
                photos.append({
                    "url": result["urls"]["regular"],
                    "description": result.get("description", ""),
                    "photographer": result["user"]["name"]
                })

            return photos

        except Exception as e:
            logger.error(f"搜索图片失败: {e}")
            return []

    def get_photo_url(self, query: str) -> Optional[str]:
        """获取单张图片URL"""
        photos = self.search_photos(query, per_page=1)
        return photos[0].get("url") if photos else None
```

Class dịch vụ này cung cấp hai phương thức: `search_photos` tìm kiếm nhiều hình ảnh, `get_photo_url` lấy URL của một hình ảnh. Chúng ta sử dụng dịch vụ này trong route API, để lấy hình ảnh cho mỗi địa điểm tham quan:
```python
# backend/app/api/routes/trip.py
from app.services.unsplash_service import UnsplashService

unsplash_service = UnsplashService(settings.unsplash_access_key)

@router.post("/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    # 生成旅行计划
    trip_plan = trip_planner_agent.plan_trip(request)

    # 为每个景点获取图片
    for day in trip_plan.days:
        for attraction in day.attractions:
            if not attraction.image_url:
                image_url = unsplash_service.get_photo_url(
                    f"{attraction.name} {trip_plan.city}"
                )
                attraction.image_url = image_url

    return trip_plan
```

Lưu ý chúng ta không đóng gói Unsplash thành Tool hay công cụ MCP, mà gọi trực tiếp trong route API. Đó là vì việc tìm kiếm hình ảnh không cần quyết định thông minh của Agent, chỉ là một bước tăng cường dữ liệu đơn giản. Nếu bạn muốn Agent có thể tự chủ quyết định có cần hình ảnh hay không, hoặc chọn nguồn hình ảnh khác nhau, có thể cân nhắc đóng gói nó thành Tool.

## 13.5 Giải thích chi tiết phát triển Front-end

### 13.5.1 Kiến trúc Web tách biệt Front-end và Back-end

Trước khi bắt đầu phát triển front-end, chúng ta cần hiểu rõ mô hình kiến trúc của ứng dụng Web hiện đại. Trong giai đoạn đầu của phát triển Web, front-end và back-end được trộn lẫn với nhau, ví dụ các công nghệ như PHP, JSP, template HTML và code logic nghiệp vụ được viết trong cùng một file. Cách này rất tiện lợi trong các dự án nhỏ, nhưng trong dự án lớn sẽ gặp nhiều vấn đề: các nhà phát triển front-end và back-end cần phối hợp thường xuyên, code khó tái sử dụng, kiểm thử khó khăn.

Ứng dụng Web hiện đại phổ biến áp dụng kiến trúc <strong>tách biệt front-end và back-end</strong>. Back-end chỉ phụ trách cung cấp giao diện API, trả về dữ liệu định dạng JSON. Front-end là một ứng dụng độc lập, thông qua HTTP request gọi API back-end, nhận dữ liệu xong thì render trang. Kiến trúc này có một số ưu điểm rõ ràng: front-end và back-end có thể phát triển độc lập, triển khai độc lập, kiểm thử độc lập; front-end có thể là ứng dụng Web, ứng dụng di động hoặc ứng dụng desktop, đều dùng chung một bộ API back-end; front-end có thể dùng framework và công cụ hiện đại, mang lại trải nghiệm người dùng tốt hơn.

Trong dự án trợ lý du lịch thông minh của chúng ta, back-end được hiện thực bằng Python và FastAPI, cung cấp một giao diện API cốt lõi `POST /api/trip/plan`, nhận yêu cầu du lịch, trả về kế hoạch du lịch. Front-end được hiện thực bằng Vue 3 và TypeScript, là một ứng dụng trang đơn (SPA), người dùng điền form trên trình duyệt, bấm nút "Bắt đầu lập kế hoạch", front-end gửi HTTP request tới back-end, chờ response, rồi render trang kết quả. Trong toàn bộ quá trình, trang không bị làm mới (refresh), trải nghiệm người dùng rất mượt mà.

Việc lựa chọn công nghệ front-end cần cân nhắc một số yếu tố: hiệu quả phát triển, hiệu năng, hệ sinh thái, đường cong học tập. Như Bảng 13.2 thể hiện, dự án này đã chọn bộ công nghệ sau:

<div align="center">
  <p>Bảng 13.2 Bộ công nghệ Front-end</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-2.png" alt="" width="85%"/>
</div>

Cấu trúc thư mục của dự án như sau:
```
frontend/
├── src/
│   ├── views/              # 页面组件
│   │   ├── Home.vue        # 首页(表单)
│   │   └── Result.vue      # 结果页
│   ├── services/           # API服务
│   │   └── api.ts
│   ├── types/              # 类型定义
│   │   └── index.ts
│   ├── router/             # 路由配置
│   │   └── index.ts
│   ├── App.vue
│   └── main.ts
├── package.json
├── vite.config.ts
└── tsconfig.json
```

Trong đó thư mục `views` chứa component trang, thư mục `services` chứa logic gọi API, thư mục `types` chứa định nghĩa kiểu TypeScript, thư mục `router` chứa cấu hình router.

### 13.5.2 Định nghĩa kiểu

Ở mục 13.2, chúng ta đã dùng Pydantic để định nghĩa mô hình dữ liệu ở back-end, ví dụ `Location`, `Attraction`, `DayPlan`, `TripPlan`, v.v. Ở front-end, chúng ta cần định nghĩa kiểu TypeScript tương ứng.

Hãy xem cách định nghĩa những kiểu này. Đầu tiên là kiểu `Location` cơ bản nhất, biểu diễn tọa độ kinh độ vĩ độ:

```typescript
// frontend/src/types/index.ts
export interface Location {
  longitude: number
  latitude: number
}
```

Kiểu này tương ứng hoàn toàn với mô hình Pydantic ở back-end. Lưu ý TypeScript dùng từ khóa `interface` để định nghĩa kiểu, kiểu trường được phân cách bằng dấu hai chấm, không cần giá trị mặc định.

Tiếp theo là kiểu `Attraction`, biểu diễn thông tin địa điểm tham quan:

```typescript
export interface Attraction {
  name: string
  address: string
  location: Location
  visit_duration: number
  description: string
  category?: string
  rating?: number
  image_url?: string
  ticket_price?: number
}
```

Lưu ý ở đây đã dùng kiểu `Location` làm kiểu trường, đây chính là kiểu lồng nhau. Dấu hỏi `?` biểu thị trường tùy chọn, tương ứng với `Optional` trong mô hình Pydantic ở back-end.

Tương tự, chúng ta định nghĩa các kiểu `Meal`, `Hotel`, `Budget`, `WeatherInfo`, v.v. Cuối cùng là kiểu `TripPlan` cấp cao nhất:

```typescript
export interface TripPlan {
  city: string
  start_date: string
  end_date: string
  days: DayPlan[]
  weather_info: WeatherInfo[]
  overall_suggestions: string
  budget?: Budget
}
```

Còn có kiểu request `TripPlanRequest`, tương ứng với mô hình request ở back-end:

```typescript
export interface TripPlanRequest {
  city: string
  start_date: string
  end_date: string
  days: number
  preferences: string
  budget: string
  transportation: string
  accommodation: string
}
```

Những định nghĩa kiểu này có ích gì? Trước hết, khi chúng ta gọi API, TypeScript sẽ kiểm tra dữ liệu chúng ta truyền có phù hợp với kiểu `TripPlanRequest` không. Nếu chúng ta vô tình viết `days` thành chuỗi, TypeScript sẽ báo lỗi ngay lập tức. Thứ hai, khi chúng ta nhận response từ API, TypeScript sẽ kiểm tra dữ liệu response có phù hợp với kiểu `TripPlan` không. Nếu cấu trúc dữ liệu back-end trả về thay đổi, front-end sẽ phát hiện ngay lập tức. Cuối cùng, IDE có thể dựa trên định nghĩa kiểu để cung cấp code completion, khi chúng ta gõ `tripPlan.`, IDE sẽ tự động liệt kê tất cả các trường có thể dùng.

### 13.5.3 Đóng gói dịch vụ API

Có định nghĩa kiểu rồi, chúng ta có thể đóng gói lời gọi API. Chúng ta tạo một file `api.ts`, dùng Axios để gửi HTTP request:

```typescript
import axios from 'axios'
import type { TripPlanRequest,TripPlan } from '../types'

const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  timeout: 120000, // 2分钟超时
  headers: {
    'Content-Type': 'application/json'
  }
})
```

Ở đây chúng ta đã tạo một instance Axios, cấu hình URL gốc, thời gian timeout và request header. Tại sao thời gian timeout đặt là 2 phút? Vì việc tạo kế hoạch du lịch cần gọi nhiều Agent, mỗi Agent đều phải gọi LLM và API bên ngoài, toàn bộ quá trình có thể mất 10-30 giây. Nếu thời gian timeout quá ngắn, request sẽ bị gián đoạn.

Tiếp theo chúng ta thêm interceptor. Interceptor có thể thực thi một số logic chung trước khi gửi request và sau khi nhận response, ví dụ ghi log, xử lý lỗi, xác thực, v.v.:

```typescript
// 请求拦截器
api.interceptors.request.use(
  config => {
    console.log('发送请求：',config)
    return config
  },
  error => Promise.reject(error)
)

// 响应拦截器
api.interceptors.response.use(
  response => {
    console.log('收到响应：',response)
    return response
  },
  error => {
    console.error('请求失败：',error)
    return Promise.reject(error)
  }
)
```

Cuối cùng chúng ta định nghĩa hàm API, đây là điểm vào (entry point) duy nhất của front-end gọi back-end:

```typescript
// 生成旅行计划
export const generateTripPlan = async (request: TripPlanRequest): Promise<TripPlan> => {
  const response = await api.post<TripPlan>('/trip/plan',request)
  return response.data
}
```

Lưu ý chữ ký kiểu (type signature) của hàm này: tham số là kiểu `TripPlanRequest`, giá trị trả về là kiểu `Promise<TripPlan>`. Điều này có nghĩa là TypeScript sẽ kiểm tra tham số mà người gọi truyền vào có phù hợp yêu cầu không, cũng sẽ kiểm tra việc sử dụng giá trị trả về có đúng không.

### 13.5.4 Thiết kế form trang Home

Trang Home là điểm vào của người dùng, chứa một form, để người dùng điền yêu cầu du lịch. Chúng ta dùng Composition API của Vue 3 để tổ chức code:

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { generateTripPlan } from '@/services/api'
import type { TripPlanRequest } from '@/types'

const router = useRouter()
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const formData = ref<TripPlanRequest>({
  city: '',
  start_date: '',
  end_date: '',
  days: 3,
  preferences: '历史文化',
  budget: '中等',
  transportation: '公共交通',
  accommodation: '经济型酒店'
})
</script>
```

Ở đây chúng ta dùng `ref` để tạo biến phản ứng (reactive variable). `formData` là dữ liệu form, kiểu là `TripPlanRequest`. `loading` biểu thị có đang tải hay không, `loadingProgress` biểu thị tiến trình tải, `loadingStatus` biểu thị văn bản trạng thái tải.

Logic submit form như sau:

```typescript
const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0
  
  // 模拟进度更新
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 正在搜索景点...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ 正在查询天气...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 正在推荐酒店...'
      else loadingStatus.value = '📋 正在生成行程计划...'
    }
  },500)
  
  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    router.push({ name: 'result',state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('生成计划失败,请重试')
  } finally {
    loading.value = false
  }
}
```

Đoạn code này làm một vài việc. Trước tiên, đặt `loading` là true, hiển thị trạng thái tải. Sau đó, khởi động một timer, mỗi 500 mili giây cập nhật một lần thanh tiến trình và văn bản trạng thái. Đây là một tiến trình mô phỏng, vì chúng ta không thể biết chính xác tiến trình xử lý ở back-end. Nhưng làm vậy có thể để người dùng biết hệ thống đang hoạt động, chứ không phải bị treo.

Tiếp theo, gọi hàm `generateTripPlan` để gửi request API. Đây là một thao tác bất đồng bộ (asynchronous), chúng ta dùng `await` để chờ response. Nếu request thành công, xóa timer, đặt tiến trình là 100%, sau đó chuyển hướng tới trang kết quả, và truyền dữ liệu kế hoạch du lịch qua. Nếu request thất bại, hiển thị thông báo lỗi. Cuối cùng, dù thành công hay thất bại, đều đặt `loading` là false, ẩn trạng thái tải.

Phần template dùng component của Ant Design Vue:

```vue
<template>
  <div class="home-container">
    <div class="page-header">
      <h1 class="page-title">✈️ 智能旅行助手</h1>
      <p class="page-subtitle">基于AI的个性化旅行规划</p>
    </div>
    
    <a-card class="form-card">
      <a-form :model="formData" @finish="handleSubmit">
        <a-form-item label="目的地城市" name="city" :rules="[{ required: true }]">
          <a-input v-model:value="formData.city" placeholder="如：北京" />
        </a-form-item>
        
        <!-- 更多表单项... -->
        
        <a-form-item>
          <a-button type="primary" html-type="submit" size="large" :loading="loading">
            开始规划
          </a-button>
        </a-form-item>
        
        <!-- 加载进度条 -->
        <a-form-item v-if="loading">
          <a-progress :percent="loadingProgress" status="active" />
          <p>{{ loadingStatus }}</p>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>
```

Lưu ý directive `v-model:value`, nó hiện thực ràng buộc dữ liệu hai chiều (two-way data binding). Khi người dùng nhập nội dung vào ô input, `formData.city` sẽ tự động cập nhật. Khi giá trị của `formData.city` thay đổi, nội dung của ô input cũng sẽ tự động cập nhật.

### 13.5.5 Hiển thị trang Result

Trang Result là cốt lõi của toàn bộ ứng dụng, hiển thị kế hoạch du lịch đã tạo ra. Trang này chứa một vài phần: tổng quan hành trình, chi tiết ngân sách, trực quan hóa bản đồ, chi tiết hành trình từng ngày, thông tin thời tiết.

Trước tiên là trực quan hóa bản đồ. Chúng ta dùng JS API Bản đồ Cao Đức để đánh dấu vị trí địa điểm tham quan trên bản đồ:

```typescript
import AMapLoader from '@amap/amap-jsapi-loader'

const initMap = async () => {
  const AMap = await AMapLoader.load({
    key: 'your_amap_web_key',
    version: '2.0'
  })
  
  map = new AMap.Map('amap-container',{
    zoom: 12,
    center: [116.397128,39.916527]
  })
  
  // 添加景点标记
  tripPlan.value.days.forEach((day) => {
    day.attractions.forEach((attraction,index) => {
      const marker = new AMap.Marker({
        position: [attraction.location.longitude,attraction.location.latitude],
        title: attraction.name,
        label: { content: `${index + 1}`,direction: 'top' }
      })
      map.add(marker)
    })
  })
}
```

Đoạn code này trước tiên tải SDK Bản đồ Cao Đức, sau đó tạo instance bản đồ, cuối cùng duyệt qua tất cả địa điểm tham quan, tạo một marker cho mỗi địa điểm. Vị trí của marker là tọa độ kinh độ vĩ độ của địa điểm tham quan, những tọa độ này được lấy từ đối tượng `Attraction` ở back-end.

Chức năng xuất file dùng thư viện `html2canvas` và `jsPDF`. `html2canvas` có thể chuyển đổi phần tử DOM thành Canvas, sau đó chúng ta có thể xuất Canvas thành hình ảnh hoặc PDF:

```typescript
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'

// 导出为图片
const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element,{ scale: 2 })
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city}旅行计划.png`
  link.href = canvas.toDataURL()
  link.click()
}

// 导出为PDF
const exportAsPDF = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element,{ scale: 2 })
  const imgData = canvas.toDataURL('image/png')
  const pdf = new jsPDF('p','mm','a4')
  const imgWidth = 210
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  pdf.addImage(imgData,'PNG',0,0,imgWidth,imgHeight)
  pdf.save(`${tripPlan.value.city}旅行计划.pdf`)
}
```

Thông qua những công nghệ front-end này, chúng ta đã hiện thực một ứng dụng Web hoàn chỉnh. Người dùng có thể điền form trên trình duyệt, gửi yêu cầu, chờ AI tạo kế hoạch du lịch, sau đó xem chi tiết sắp xếp hành trình, nhìn thấy vị trí địa điểm tham quan trên bản đồ, còn có thể xuất ra hình ảnh hoặc PDF. Toàn bộ quá trình mượt mà tự nhiên, đây chính là sức hút của ứng dụng Web hiện đại.

## 13.6 Giải thích chi tiết hiện thực chức năng

Mục này giới thiệu hiện thực các chức năng cốt lõi của trợ lý du lịch thông minh, bao gồm tính toán ngân sách, thanh tiến trình tải, chỉnh sửa hành trình, chức năng xuất file và điều hướng thanh bên.

### 13.6.1 Chức năng tính toán ngân sách

Khi lập kế hoạch du lịch, ngân sách là một yếu tố cân nhắc rất quan trọng. Người dùng cần biết chuyến du lịch này ước chừng tốn bao nhiêu tiền, tiền đều tiêu vào đâu. Trợ lý du lịch thông minh của chúng ta cung cấp chức năng tính toán ngân sách tự động, chia chi phí thành bốn nhóm lớn: vé tham quan, lưu trú khách sạn, ẩm thực và giao thông.

Logic tính toán ngân sách được hiện thực ở đâu? Chúng ta chọn hiện thực trong PlannerAgent ở back-end. Tại sao không tính toán ở front-end? Vì việc ước tính ngân sách cần dựa trên thông tin giá vé địa điểm tham quan, khoảng giá khách sạn, tiêu chuẩn ẩm thực, v.v., những thông tin này đều đã được PlannerAgent lấy được khi tạo hành trình. Nếu tính toán ở front-end, cần lặp lại những logic này, và có thể không chính xác.

Trong prompt của PlannerAgent, chúng ta yêu cầu rõ ràng LLM tạo ra thông tin ngân sách:

```python
PLANNER_AGENT_PROMPT = """
你是行程规划专家。

**输出格式：**
严格按照以下JSON格式返回：
{
  ...
  "budget": {
    "total_attractions": 180,
    "total_hotels": 1200,
    "total_meals": 480,
    "total_transportation": 200,
    "total": 2060
  }
}

**规划要求：**
...
7. 包含预算信息,根据景点门票、酒店价格、餐饮标准和交通方式估算
"""
```

LLM sẽ dựa trên sắp xếp địa điểm tham quan, khách sạn, ẩm thực trong hành trình, ước tính chi phí của từng mục. Ví dụ, nếu hành trình bao gồm Cố Cung (vé 60 tệ), Thiên Đàn (vé 15 tệ), Di Hòa Viên (vé 30 tệ), thì tổng chi phí vé tham quan là 105 tệ. Nếu là hành trình 3 ngày 2 đêm, khách sạn là loại kinh tế (300 tệ/đêm), thì tổng chi phí khách sạn là 600 tệ.

Ở front-end, chúng ta dùng component Statistic của Ant Design Vue để hiển thị thông tin ngân sách. Component này chuyên dùng để hiển thị dữ liệu thống kê, hỗ trợ hiệu ứng động cho số, tiền tố hậu tố, kiểu tùy chỉnh, v.v.:

```vue
<a-card v-if="tripPlan.budget" title="💰 预算明细">
  <a-row :gutter="16">
    <a-col :span="6">
      <a-statistic title="景点门票" :value="tripPlan.budget.total_attractions" suffix="元" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="酒店住宿" :value="tripPlan.budget.total_hotels" suffix="元" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="餐饮费用" :value="tripPlan.budget.total_meals" suffix="元" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="交通费用" :value="tripPlan.budget.total_transportation" suffix="元" />
    </a-col>
  </a-row>
  <a-divider />
  <a-row>
    <a-col :span="24" style="text-align: center;">
      <a-statistic
        title="预估总费用"
        :value="tripPlan.budget.total"
        suffix="元"
        :value-style="{ color: '#cf1322',fontSize: '32px',fontWeight: 'bold' }"
      />
    </a-col>
  </a-row>
</a-card>
```

Đoạn code này sử dụng bố cục lưới (grid layout) (`a-row` và `a-col`), hiển thị song song bốn khoản chi phí. Mỗi khoản chi phí dùng một component `a-statistic`, hiển thị tiêu đề và giá trị. Cuối cùng dùng một đường phân cách (`a-divider`) ngăn ra, bên dưới hiển thị tổng chi phí, dùng font chữ lớn màu đỏ để làm nổi bật.

Lưu ý điều kiện render `v-if="tripPlan.budget"`. Vì thông tin ngân sách là tùy chọn (được định nghĩa là `Optional[Budget]` trong mô hình Pydantic), nếu LLM không tạo ra thông tin ngân sách, card này sẽ không hiển thị. Điều này thể hiện khả năng xử lý khoan dung lỗi của front-end đối với dữ liệu.

### 13.6.2 Thanh tiến trình tải

Tạo kế hoạch du lịch là một thao tác tốn thời gian. Back-end cần lần lượt gọi AttractionSearchAgent, WeatherQueryAgent, HotelAgent và PlannerAgent, mỗi Agent đều phải gọi LLM và API bên ngoài. Toàn bộ quá trình có thể mất 10-30 giây. Nếu sau khi người dùng bấm nút "Bắt đầu lập kế hoạch", trang không có bất kỳ phản hồi nào, người dùng sẽ tưởng hệ thống bị treo, có thể sẽ làm mới trang hoặc bấm lặp lại.

Để nâng cao trải nghiệm người dùng, chúng ta đã thêm thanh tiến trình tải và gợi ý trạng thái. Hiện tại chỉ là mô phỏng tiến trình, có thể để người dùng biết hệ thống đang hoạt động.

```typescript
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // 模拟进度更新
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 正在搜索景点...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ 正在查询天气...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 正在推荐酒店...'
      else loadingStatus.value = '📋 正在生成行程计划...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    loadingStatus.value = '✅ 完成！'
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('生成计划失败')
  } finally {
    loading.value = false
  }
}
```

### 13.6.3 Chức năng chỉnh sửa hành trình

Kế hoạch du lịch do AI tạo ra tuy rất thông minh, nhưng có thể không hoàn toàn phù hợp với nhu cầu cá nhân của người dùng. Ví dụ, người dùng có thể không thích một địa điểm tham quan nào đó, muốn xóa nó đi; hoặc muốn điều chỉnh thứ tự tham quan địa điểm. Chúng ta cung cấp chức năng chỉnh sửa hành trình, để người dùng có thể tùy chỉnh hành trình.

Cốt lõi của chức năng chỉnh sửa là <strong>quản lý trạng thái (state management)</strong>. Chúng ta cần duy trì hai trạng thái: kế hoạch hành trình hiện tại và kế hoạch hành trình gốc. Khi người dùng vào chế độ chỉnh sửa, chúng ta lưu một bản sao của kế hoạch gốc. Nếu người dùng hủy chỉnh sửa, thì khôi phục kế hoạch gốc. Nếu người dùng lưu chỉnh sửa, thì cập nhật kế hoạch hiện tại:

```typescript
const editMode = ref(false)
const originalPlan = ref<TripPlan | null>(null)

// 进入编辑模式
const toggleEditMode = () => {
  editMode.value = true
  originalPlan.value = JSON.parse(JSON.stringify(tripPlan.value))
}
```

Lưu ý ở đây đã dùng `JSON.parse(JSON.stringify(...))` để sao chép sâu (deep copy) đối tượng. Tại sao không gán trực tiếp? Vì trong JavaScript, đối tượng là kiểu tham chiếu (reference type), nếu gán trực tiếp, `originalPlan` và `tripPlan` sẽ trỏ tới cùng một đối tượng, sửa một cái sẽ ảnh hưởng tới cái còn lại. Sao chép sâu có thể tạo ra một bản sao hoàn toàn độc lập.

Logic di chuyển địa điểm tham quan là hoán đổi vị trí hai phần tử trong mảng:

```typescript
// 移动景点
const moveAttraction = (dayIndex: number,attractionIndex: number,direction: 'up' | 'down') => {
  const attractions = tripPlan.value.days[dayIndex].attractions
  const newIndex = direction === 'up' ? attractionIndex - 1 : attractionIndex + 1
  
  if (newIndex >= 0 && newIndex < attractions.length) {
    [attractions[attractionIndex],attractions[newIndex]] = 
    [attractions[newIndex],attractions[attractionIndex]]
  }
}
```

Ở đây đã dùng cú pháp destructuring assignment của ES6 để hoán đổi hai phần tử. `[a,b] = [b,a]` là một cách hoán đổi rất tinh tế, không cần biến tạm.

Xóa địa điểm tham quan dùng phương thức `splice` của mảng:

```typescript
// 删除景点
const deleteAttraction = (dayIndex: number,attractionIndex: number) => {
  tripPlan.value.days[dayIndex].attractions.splice(attractionIndex,1)
}
```

Khi lưu chỉnh sửa, chúng ta cần khởi tạo lại bản đồ, vì vị trí địa điểm tham quan có thể đã thay đổi:

```typescript
// 保存修改
const saveChanges = () => {
  editMode.value = false
  message.success('修改已保存')
  initMap()  // 重新初始化地图
}

// 取消编辑
const cancelEdit = () => {
  if (originalPlan.value) {
    tripPlan.value = originalPlan.value
  }
  editMode.value = false
}
```

Trong template, chúng ta hiển thị UI khác nhau tùy theo giá trị của `editMode`. Trong chế độ chỉnh sửa, bên cạnh mỗi địa điểm tham quan sẽ hiển thị nút di chuyển lên, di chuyển xuống, xóa:

```vue
<div v-if="editMode" class="edit-buttons">
  <a-button size="small" @click="moveAttraction(dayIndex,index,'up')">上移</a-button>
  <a-button size="small" @click="moveAttraction(dayIndex,index,'down')">下移</a-button>
  <a-button size="small" danger @click="deleteAttraction(dayIndex,index)">删除</a-button>
</div>
```

### 13.6.4 Chức năng xuất file

Sau khi người dùng tạo ra được kế hoạch du lịch ưng ý, có thể muốn lưu lại hoặc chia sẻ cho bạn bè. Chúng ta cung cấp hai cách xuất file: xuất ra hình ảnh và xuất ra PDF.

Cốt lõi của chức năng xuất file là thư viện `html2canvas`. Thư viện này có thể chuyển đổi phần tử DOM thành Canvas, sau đó chúng ta có thể xuất Canvas thành hình ảnh. Nhưng ở đây có một khó khăn kỹ thuật: bản đồ được render bằng Canvas, mà `html2canvas` khi xử lý Canvas lồng nhau tồn tại vấn đề tương thích.

Chúng ta đã thử nhiều giải pháp, bao gồm chuyển đổi Canvas bản đồ thành hình ảnh rồi mới xuất file, nhưng do cơ chế render Canvas của Bản đồ Cao Đức và giới hạn cross-origin, phương án này không giải quyết hoàn toàn được vấn đề. Trong dự án thực tế, có thể cần cân nhắc các phương án thay thế sau:

1. <strong>Dùng API bản đồ tĩnh của Bản đồ Cao Đức</strong>: gọi công cụ `maps_staticmap` để tạo hình ảnh bản đồ tĩnh, thay thế bản đồ động
2. <strong>Xuất file riêng biệt</strong>: xuất riêng bản đồ và nội dung hành trình, cuối cùng hợp nhất ở back-end
3. <strong>Dùng dịch vụ chụp màn hình</strong>: dùng trình duyệt không giao diện (headless browser) như Puppeteer để chụp màn hình phía server
4. <strong>Đơn giản hóa nội dung xuất file</strong>: khi xuất file thì ẩn bản đồ, chỉ xuất nội dung văn bản

Trong hiện thực hiện tại, chúng ta đã áp dụng phương án đơn giản hóa, khi xuất file thì tạm thời ẩn phần bản đồ, chỉ xuất nội dung văn bản hành trình và thông tin địa điểm tham quan. Mặc dù đây không phải phương án lý tưởng nhất, nhưng có thể đảm bảo khả năng sử dụng của chức năng xuất file.

Logic xuất ra hình ảnh rất đơn giản:

```typescript
import html2canvas from 'html2canvas'

const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  if (!element) return
  
  const canvas = await html2canvas(element,{
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true
  })
  
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city}旅行计划.png`
  link.href = canvas.toDataURL('image/png')
  link.click()
  message.success('导出成功！')
}
```

`scale: 2` biểu thị dùng độ phân giải gấp 2 lần, như vậy hình ảnh xuất ra sẽ rõ nét hơn. `useCORS: true` cho phép tải hình ảnh cross-origin, điều này rất quan trọng đối với hình ảnh địa điểm tham quan (đến từ Unsplash).

Xuất ra PDF cần thêm bước: chuyển đổi thành Canvas trước, rồi chuyển đổi thành hình ảnh, cuối cùng thêm vào PDF:

```typescript
import jsPDF from 'jspdf'

const exportAsPDF = async () => {
  // 先截取地图
  await captureMapImage()
  
  const element = document.getElementById('trip-plan-content')
  if (!element) return
  
  const canvas = await html2canvas(element,{
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true,
    allowTaint: true
  })
  
  // 恢复地图
  restoreMap()
  
  const pdf = new jsPDF('p','mm','a4')
  const imgData = canvas.toDataURL('image/png')
  const imgWidth = 210  // A4宽度
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  
  pdf.addImage(imgData,'PNG',0,0,imgWidth,imgHeight)
  pdf.save(`${tripPlan.value.city}旅行计划.pdf`)
  message.success('导出成功！')
}
```

Ở đây cần tính toán chiều cao của hình ảnh, giữ nguyên tỷ lệ khung hình. Chiều rộng giấy A4 là 210mm, chúng ta dựa trên tỷ lệ khung hình của Canvas để tính ra chiều cao tương ứng.

### 13.6.5 Điều hướng thanh bên và nhảy đến anchor

Trang Result có rất nhiều nội dung, bao gồm tổng quan hành trình, chi tiết ngân sách, bản đồ, hành trình từng ngày, thông tin thời tiết, v.v. Nếu người dùng muốn nhảy nhanh tới một phần nào đó, cần cuộn một đoạn rất dài. Chúng ta cung cấp chức năng điều hướng thanh bên và nhảy tới anchor, để người dùng có thể định vị nhanh chóng.

Điều hướng thanh bên dùng component Menu của Ant Design Vue:

```vue
<a-menu
  v-model:selectedKeys="[activeSection]"
  mode="inline"
  @click="scrollToSection"
>
  <a-menu-item key="overview">📋 行程概览</a-menu-item>
  <a-menu-item key="budget">💰 预算明细</a-menu-item>
  <a-menu-item key="map">🗺️ 地图</a-menu-item>
  <a-menu-item key="days">📅 每日行程</a-menu-item>
  <a-menu-item key="weather">🌤️ 天气</a-menu-item>
</a-menu>
```

Khi bấm vào mục menu, gọi hàm `scrollToSection`:

```typescript
const activeSection = ref('overview')

// 滚动到指定区域
const scrollToSection = ({ key }: { key: string }) => {
  activeSection.value = key
  const element = document.getElementById(key)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth',block: 'start' })
  }
}
```

`scrollIntoView` là API gốc của trình duyệt, có thể cho phần tử cuộn vào vùng hiển thị. `behavior: 'smooth'` biểu thị cuộn mượt, chứ không phải nhảy tức thời. `block: 'start'` biểu thị phần trên cùng của phần tử được căn chỉnh tới phần trên cùng của vùng hiển thị.

Ở các phần khác nhau của trang, chúng ta cần thêm id tương ứng:

```vue
<div id="overview">
  <!-- 行程概览内容 -->
</div>

<div id="budget">
  <!-- 预算明细内容 -->
</div>

<div id="map">
  <!-- 地图内容 -->
</div>
```

Như vậy, khi người dùng bấm vào một mục menu nào đó ở thanh điều hướng bên, trang sẽ cuộn mượt tới phần tương ứng.

Thông qua việc hiện thực những chức năng này, trợ lý du lịch thông minh của chúng ta không chỉ có thể tạo kế hoạch du lịch, mà còn cung cấp các chức năng tương tác phong phú: tính toán ngân sách giúp người dùng nắm rõ chi phí, thanh tiến trình tải giúp việc chờ đợi bớt lo lắng, chỉnh sửa hành trình giúp kế hoạch phù hợp hơn với nhu cầu cá nhân, chức năng xuất file giúp kế hoạch có thể chia sẻ và lưu trữ, điều hướng thanh bên giúp trang dài dễ duyệt xem. Sự kết hợp của những chức năng này tạo nên một ứng dụng Web hoàn chỉnh, dễ dùng, thực dụng.

## 13.7 Kết luận

Chúc mừng bạn đã hoàn thành việc học Chương 13!

Thông qua chương này, bạn không chỉ học được cách xây dựng một ứng dụng trợ lý du lịch thông minh hoàn chỉnh, mà quan trọng hơn là đã nắm vững:

1. <strong>Tư duy thiết kế hệ thống</strong>: Cách phân rã vấn đề phức tạp thành nhiều tác vụ đơn giản
2. <strong>Năng lực thực hành kỹ thuật</strong>: Cách chuyển hóa kiến thức lý thuyết thành code có thể chạy được
3. <strong>Năng lực phát triển full-stack</strong>: Cách tích hợp bộ công nghệ front-end và back-end
4. <strong>Phát triển ứng dụng AI</strong>: Cách tận dụng LLM để xây dựng ứng dụng thực dụng

Dự án này là một điểm khởi đầu, chứ không phải điểm kết thúc. Bạn có thể dựa trên dự án này để:

- Thêm nhiều chức năng hơn
- Tối ưu trải nghiệm người dùng
- Mở rộng sang các lĩnh vực khác (như trợ lý mua sắm thông minh, trợ lý học tập thông minh, v.v.)
- Triển khai lên môi trường production, phục vụ người dùng thực tế

Cách học tốt nhất là thực hành. Đừng chỉ đọc code, mà hãy tự tay sửa đổi, mở rộng, tối ưu. Mỗi lần thực hành đều sẽ giúp bạn hiểu sâu hơn về hệ thống đa Agent.

Chúc bạn ngày càng tiến xa trên con đường phát triển ứng dụng AI!
