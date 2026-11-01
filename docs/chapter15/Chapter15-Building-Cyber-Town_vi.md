# Chương 15 Xây dựng Thị trấn Cyber

Trong chương này, chúng ta sẽ khám phá một hướng đi hoàn toàn mới: <strong>kết hợp công nghệ Agent với game engine để xây dựng một thị trấn AI tràn đầy sức sống</strong>.

Bạn còn nhớ những NPC sống động trong "The Sims" hay "Animal Crossing" không? Họ có tính cách, ký ức và các mối quan hệ xã hội riêng. Thị trấn Cyber trong chương này sẽ là một dự án tương tự, nhưng khác với game truyền thống, các NPC của chúng ta sở hữu "trí tuệ" thực sự — họ có thể hiểu được cuộc trò chuyện của người chơi, ghi nhớ các tương tác trong quá khứ, và phản ứng khác nhau dựa trên độ thiện cảm (affinity). Thị trấn Cyber trong chương này bao gồm các tính năng cốt lõi sau:

<strong>（1）Hệ thống hội thoại NPC thông minh</strong>: Người chơi có thể trò chuyện bằng ngôn ngữ tự nhiên với NPC, NPC sẽ phản hồi dựa trên thiết lập nhân vật và ký ức của chính mình.

<strong>（2）Hệ thống ký ức (Memory)</strong>: NPC sở hữu ký ức ngắn hạn và ký ức dài hạn, có thể ghi nhớ lịch sử tương tác với người chơi.

<strong>（3）Hệ thống độ thiện cảm (Affinity)</strong>: Thái độ của NPC đối với người chơi sẽ thay đổi theo tương tác, từ xa lạ đến quen thuộc, từ thân thiện đến thân mật.

<strong>（4）Tương tác hóa game (Game hóa tương tác)</strong>: Người chơi có thể tự do di chuyển trong một bối cảnh văn phòng theo phong cách pixel 2D và tương tác với các NPC khác nhau.

<strong>（5）Hệ thống log thời gian thực</strong>: Mọi cuộc hội thoại và tương tác đều được ghi lại, thuận tiện cho việc debug và phân tích.

## 15.1 Tổng quan dự án và thiết kế kiến trúc

### 15.1.1 Tại sao cần xây dựng thị trấn AI

Trong các game truyền thống, NPC thường chỉ có thể nói những câu thoại cố định, hoặc tương tác hạn chế thông qua cây hội thoại được thiết lập sẵn. Ngay cả trong các game RPG phức tạp nhất, hội thoại của NPC cũng do biên kịch viết sẵn từ trước. Cách làm này tuy dễ kiểm soát nhưng thiếu đi "trí tuệ" và "sức sống" thực sự.

Hãy tưởng tượng, nếu NPC trong game có thể hiểu bất cứ điều gì bạn nói, không còn giới hạn ở các lựa chọn được định sẵn, bạn có thể dùng ngôn ngữ tự nhiên để giao tiếp với NPC. NPC sẽ nhớ lần trước bạn đã nói gì, mối quan hệ giữa hai bên ra sao, thậm chí cả sở thích của bạn. Mỗi NPC đều có nghề nghiệp, tính cách và phong cách nói chuyện riêng. Thái độ của NPC đối với bạn sẽ thay đổi theo tương tác, từ người lạ thành bạn bè, thậm chí thành tri kỷ.

Đây chính là khả năng mới mà công nghệ AI mang lại cho game. Bằng cách kết hợp mô hình ngôn ngữ lớn (LLM) với game engine, chúng ta có thể tạo ra những NPC thực sự "sống". Đây không chỉ là một bản demo công nghệ, mà còn là sự khám phá về hình thái game trong tương lai. Trong các game giáo dục, NPC có thể đóng vai nhân vật lịch sử, nhà khoa học, tương tác giảng dạy với học sinh. Trong văn phòng ảo, NPC có thể đóng vai đồng nghiệp, người cố vấn, cung cấp sự giúp đỡ và lời khuyên. NPC cũng có thể đóng vai người bạn đồng hành, giao tiếp cảm xúc với người dùng, ứng dụng trong lĩnh vực sức khỏe tâm lý. Tất nhiên, ứng dụng trực tiếp nhất chính là bổ sung AI NPC cho các game truyền thống, nâng cao trải nghiệm người chơi.

### 15.1.2 Tổng quan kiến trúc kỹ thuật

Thị trấn Cyber áp dụng kiến trúc phân tách <strong>game engine + dịch vụ backend</strong>, chia thành bốn tầng, như Hình 15.1.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-1.png" alt="" width="85%"/>
  <p>Hình 15.1 Kiến trúc kỹ thuật Thị trấn Cyber</p>
</div>

Tầng frontend sử dụng game engine Godot 4.5, phụ trách render game, điều khiển người chơi, hiển thị NPC và UI hội thoại. Godot là một game engine 2D/3D mã nguồn mở, rất phù hợp để phát triển nhanh các game theo phong cách pixel. Tầng backend sử dụng framework FastAPI, phụ trách định tuyến API, quản lý trạng thái NPC, xử lý hội thoại và ghi log. FastAPI là một web framework Python hiện đại, hiệu năng tốt và dễ phát triển. Tầng Agent sử dụng framework HelloAgents mà chúng ta tự xây dựng, phụ trách trí tuệ NPC, quản lý ký ức và tính toán độ thiện cảm. Mỗi NPC đều là một instance SimpleAgent, sở hữu ký ức và trạng thái độc lập. Tầng dịch vụ bên ngoài cung cấp khả năng LLM, lưu trữ vector và lưu trữ dữ liệu bền vững, bao gồm LLM API, cơ sở dữ liệu vector Qdrant và cơ sở dữ liệu quan hệ SQLite.

Quá trình luân chuyển dữ liệu được minh họa ở Hình 15.2:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-2.png" alt="" width="85%"/>
  <p>Hình 15.2 Quá trình luân chuyển dữ liệu</p>
</div>


Người chơi nhấn phím E trong Godot để tương tác với NPC, Godot gửi yêu cầu hội thoại qua HTTP API tới backend FastAPI. Backend gọi SimpleAgent của HelloAgents để xử lý hội thoại, Agent truy xuất lịch sử liên quan từ hệ thống ký ức, sau đó gọi LLM để sinh phản hồi. Backend cập nhật trạng thái NPC và độ thiện cảm, ghi log ra console và file, cuối cùng trả phản hồi về cho frontend Godot. Godot hiển thị phản hồi của NPC và cập nhật UI, hoàn thành một vòng tương tác đầy đủ.

Cấu trúc dự án như sau, giúp bạn dễ dàng định vị source code:

```
Helloagents-AI-Town/
├── helloagents-ai-town/           # Dự án game Godot
│   ├── project.godot              # Cấu hình dự án Godot
│   ├── scenes/                    # Các cảnh (scene) trong game
│   │   ├── main.tscn              # Cảnh chính (văn phòng)
│   │   ├── player.tscn            # Nhân vật người chơi
│   │   ├── npc.tscn               # Nhân vật NPC
│   │   └── dialogue_ui.tscn       # UI hội thoại
│   ├── scripts/                   # Script GDScript
│   │   ├── main.gd                # Logic cảnh chính
│   │   ├── player.gd               # Điều khiển người chơi
│   │   ├── npc.gd                 # Hành vi NPC
│   │   ├── dialogue_ui.gd         # Logic UI hội thoại
│   │   ├── api_client.gd          # API client
│   │   └── config.gd              # Quản lý cấu hình
│   └── assets/                    # Tài nguyên game
│       ├── characters/            # Sprite nhân vật
│       ├── interiors/             # Cảnh nội thất
│       ├── ui/                    # Tài nguyên UI
│       └── audio/                 # Âm thanh, nhạc nền
│
└── backend/                       # Backend Python
    ├── main.py                    # Chương trình chính FastAPI
    ├── agents.py                  # Hệ thống Agent NPC
    ├── relationship_manager.py    # Quản lý độ thiện cảm
    ├── state_manager.py           # Quản lý trạng thái
    ├── logger.py                  # Hệ thống log
    ├── config.py                  # Quản lý cấu hình
    ├── models.py                  # Model dữ liệu
    ├── requirements.txt           # Dependency Python
    └── .env.example               # Ví dụ biến môi trường
```

Thiết kế kiến trúc chi tiết và quá trình luân chuyển dữ liệu sẽ được giới thiệu trong các phần tiếp theo.

### 15.1.3 Trải nghiệm nhanh: chạy dự án trong 5 phút

Trước khi đi sâu vào chi tiết triển khai, hãy để chúng ta chạy thử dự án trước, xem kết quả cuối cùng như thế nào. Điều này sẽ giúp bạn có một hình dung trực quan về toàn bộ hệ thống.

<strong>Yêu cầu môi trường:</strong>

- Godot 4.2 trở lên
- Python 3.10 trở lên
- LLM API key (OpenAI, DeepSeek, Zhipu, v.v.)

<strong>Lấy dự án:</strong>

Bạn có thể xem tại `code/chapter15/Helloagents-AI-Town`, hoặc clone toàn bộ repo hello-agents từ GitHub.

<strong>Khởi động backend:</strong>

```bash
# 1. Vào thư mục backend
cd Helloagents-AI-Town/backend

# 2. Cài đặt dependency
pip install -r requirements.txt

# 3. Cấu hình biến môi trường
cp .env.example .env
# Sửa file .env, điền API key của bạn

# 4. Khởi động dịch vụ backend
python main.py
```

Sau khi khởi động thành công, bạn sẽ thấy output như sau:

```
============================================================
🎮 Dịch vụ backend Thị trấn Cyber đang khởi động...
============================================================
✅ Tất cả dịch vụ đã khởi động!
📡 Địa chỉ API: http://0.0.0.0:8000
📚 Tài liệu API: http://0.0.0.0:8000/docs
============================================================
```

<strong>Khởi động Godot:</strong>

Việc cài đặt Godot rất đơn giản, Windows cung cấp file `.exe` có thể mở trực tiếp, Mac cũng cung cấp file `.dmg`. Bạn có thể tải trực tiếp trên trang chủ ([Windows](https://godotengine.org/download/windows/) / [Mac](https://godotengine.org/download/macos/))

Mở Godot engine, nhấn nút "Import" (Nhập), duyệt tới `Helloagents-AI-Town/helloagents-ai-town/scenes/main.tscn`, nhấn "Import & Edit" (Nhập và chỉnh sửa). Đợi Godot nhập xong tài nguyên, nhấn `F5` hoặc nút "Run" (Chạy) để khởi động game.

<strong>Trải nghiệm các tính năng cốt lõi:</strong>

Sau khi game khởi động, bạn sẽ thấy một bối cảnh văn phòng Datawhale theo phong cách pixel, như Hình 15.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-3.png" alt="" width="85%"/>
  <p>Hình 15.3 Cảnh game Thị trấn Cyber</p>
</div>

Dùng phím WASD để di chuyển nhân vật người chơi, khi đến gần NPC, màn hình sẽ hiển thị gợi ý "Nhấn E để tương tác". Nhấn phím E sẽ bật ra hộp thoại, bạn có thể nhập bất cứ điều gì muốn nói, như Hình 15.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-4.png" alt="" width="85%"/>
  <p>Hình 15.4 Giao diện hội thoại với NPC</p>
</div>

NPC sẽ phản hồi dựa trên thiết lập vai trò (kỹ sư Python, product manager, UI designer) và lịch sử tương tác của bạn. Khi cuộc trò chuyện tiếp diễn, độ thiện cảm của NPC đối với bạn sẽ dần tăng lên, từ "xa lạ" đến "quen thuộc", rồi đến "thân thiện", "thân mật" thậm chí "tri kỷ".

<strong>Hệ thống độ thiện cảm được triển khai ở backend</strong>, mỗi lần hội thoại đều điều chỉnh giá trị thiện cảm dựa trên nội dung tin nhắn của người chơi và phân tích cảm xúc. Mặc dù giao diện game ở frontend không trực tiếp hiển thị số liệu thiện cảm, nhưng tất cả các thay đổi về thiện cảm đều được ghi chi tiết trong log của backend. Bạn có thể xem sự thay đổi thiện cảm của mỗi lần hội thoại trong file `backend/logs/dialogue_YYYY-MM-DD.log`. File log sẽ ghi lại thông tin chi tiết của mỗi lần hội thoại, bao gồm: giá trị thiện cảm hiện tại, các ký ức liên quan được truy xuất, phản hồi của NPC, lượng thay đổi thiện cảm (+2.0, +3.0, v.v.), nguyên nhân thay đổi (chào hỏi thân thiện, giao tiếp bình thường, v.v.) cũng như kết quả phân tích cảm xúc (positive, neutral, v.v.). Thiết kế này giúp nhà phát triển có thể theo dõi rõ ràng sự phát triển mối quan hệ giữa NPC và người chơi, đồng thời cung cấp nền tảng dữ liệu cho việc bổ sung UI thiện cảm ở frontend sau này.

Tất cả các cuộc hội thoại đều được ghi lại trong file log của backend, bạn có thể xem theo thời gian thực bằng lệnh sau:

```bash
# Trong thư mục backend
python view_logs.py
```

Trải nghiệm đơn giản này đã thể hiện các tính năng cốt lõi của thị trấn AI. Tiếp theo, chúng ta sẽ đi sâu vào học cách triển khai những tính năng này.




## 15.2 Hệ thống Agent NPC

### 15.2.1 Dựa trên SimpleAgent của HelloAgents

Trong Thị trấn Cyber, mỗi NPC đều là một Agent độc lập. Chúng ta sử dụng SimpleAgent trong framework HelloAgents để triển khai trí tuệ của NPC. SimpleAgent là một triển khai Agent nhẹ, nó đóng gói các chức năng cốt lõi như gọi LLM, quản lý tin nhắn và gọi tool.

Hãy ôn lại SimpleAgent mà chúng ta đã học ở Chương 7, cốt lõi của nó là một vòng lặp hội thoại đơn giản: nhận tin nhắn người dùng, gọi LLM sinh phản hồi, trả về kết quả. Trong Thị trấn Cyber, chúng ta cần tạo một instance SimpleAgent cho mỗi NPC, và cấu hình system prompt riêng biệt cho nó, để mỗi NPC có tính cách và thiết lập vai trò khác nhau.

Hãy cùng xem cách tạo một NPC Agent. Đầu tiên, chúng ta cần định nghĩa thông tin cơ bản của NPC, bao gồm ID, tên, nghề nghiệp và tính cách. Sau đó, chúng ta dựa vào các thông tin này để xây dựng system prompt, để LLM đóng vai nhân vật này. Cuối cùng, chúng ta tạo instance SimpleAgent, và cấu hình hệ thống ký ức.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.memory import MemoryManager, WorkingMemory, EpisodicMemory

def create_npc_agent(npc_id: str, name: str, role: str, personality: str):
    """Tạo NPC Agent"""
    # Xây dựng system prompt
    system_prompt = f"""Bạn là {name}, một {role}.
Đặc điểm tính cách của bạn: {personality}

Bạn làm việc tại văn phòng Datawhale, cùng đồng nghiệp thúc đẩy sự phát triển của cộng đồng mã nguồn mở.
Hãy đối thoại một cách tự nhiên với người chơi dựa trên vai trò và tính cách của bạn.
Hãy nhớ nội dung các cuộc trò chuyện trước đó, giữ cho hội thoại liền mạch.
"""

    # Tạo instance LLM
    llm = HelloAgentsLLM()

    # Tạo memory manager
    memory_manager = MemoryManager(
        working_memory=WorkingMemory(capacity=10, ttl_minutes=120),
        episodic_memory=EpisodicMemory(
            db_path=f"memory_data/{npc_id}_episodic.db",
            collection_name=f"{npc_id}_memories"
        )
    )

    # Tạo Agent
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt,
        memory_manager=memory_manager
    )

    return agent
```

Đoạn code này minh họa cách tạo một NPC Agent. System prompt định nghĩa danh tính và tính cách của NPC, memory manager giúp NPC có thể ghi nhớ lịch sử hội thoại với người chơi. WorkingMemory là ký ức ngắn hạn, dung lượng 10 tin nhắn, thời gian lưu giữ là 120 phút. EpisodicMemory là ký ức dài hạn, sử dụng cơ sở dữ liệu SQLite và cơ sở dữ liệu vector Qdrant để lưu trữ, có thể truy xuất các cuộc hội thoại lịch sử liên quan.

Luồng xử lý của NPC Agent được minh họa ở Hình 15.5:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-5.png" alt="" width="85%"/>
  <p>Hình 15.5 Luồng xử lý của NPC Agent</p>
</div>


### 15.2.2 Thiết lập vai trò NPC và thiết kế Prompt

Một NPC tốt cần có tính cách và thiết lập vai trò rõ nét. Trong Thị trấn Cyber, chúng ta thiết kế ba NPC, đại diện cho các nghề nghiệp và tính cách khác nhau.

<strong>Trương Tam (张三) - Kỹ sư Python</strong>

Trương Tam là một kỹ sư Python kỳ cựu, phụ trách phát triển phần lõi của framework HelloAgents. Anh có tính cách nghiêm túc, nói chuyện trực tiếp, thích dùng thuật ngữ kỹ thuật. Anh có yêu cầu rất cao về chất lượng code, thường chia sẻ một số kỹ thuật lập trình và best practice.

```python
npc_zhang = {
    "npc_id": "zhang_san",
    "name": "张三",
    "role": "Python工程师",
    "personality": "严谨、专业、喜欢分享技术知识。说话直接,注重代码质量。"
}
```

<strong>Lý Tứ (李四) - Product Manager</strong>

Lý Tứ là một product manager giàu kinh nghiệm, phụ trách hoạch định sản phẩm và thiết kế trải nghiệm người dùng cho framework HelloAgents. Anh có tính cách hướng ngoại, giỏi giao tiếp, luôn suy nghĩ vấn đề từ góc độ người dùng. Anh thích thảo luận về thiết kế sản phẩm và nhu cầu người dùng, thường hay hỏi "tại sao".

```python
npc_li = {
    "npc_id": "li_si",
    "name": "李四",
    "role": "产品经理",
    "personality": "外向、善于沟通、注重用户体验。喜欢从用户角度思考问题。"
}
```

<strong>Vương Ngũ (王五) - UI Designer</strong>

Vương Ngũ là một UI designer giàu tính sáng tạo, phụ trách thiết kế giao diện và thể hiện thị giác cho framework HelloAgents. Anh có tính cách ôn hòa, thẩm mỹ độc đáo, có cảm nhận nhạy bén về màu sắc và bố cục. Anh thích thảo luận về triết lý thiết kế và thẩm mỹ, thường chia sẻ một số cảm hứng thiết kế.

```python
npc_wang = {
    "npc_id": "wang_wu",
    "name": "王五",
    "role": "UI设计师",
    "personality": "温和、富有创意、审美独特。注重视觉呈现和用户体验。"
}
```

Thiết lập của ba NPC này mỗi người một vẻ, người chơi có thể chọn tương tác với NPC khác nhau tùy theo sở thích của mình. Trương Tam có thể dạy bạn kỹ thuật lập trình, Lý Tứ có thể cùng bạn thảo luận thiết kế sản phẩm, Vương Ngũ có thể chia sẻ cảm hứng thiết kế.

### 15.2.3 Tích hợp hệ thống ký ức

Hệ thống ký ức là chìa khóa của trí tuệ NPC. Một NPC có thể ghi nhớ các cuộc hội thoại trong quá khứ sẽ khiến người chơi cảm thấy chân thực và thú vị hơn. Chúng ta sử dụng `WorkingMemory` và `EpisodicMemory` của helloagents để xây dựng ký ức ngắn hạn và dài hạn.

Ký ức ngắn hạn lưu trữ nội dung hội thoại gần nhất, dung lượng có hạn, sẽ tự động dọn dẹp theo thời gian. Vai trò của nó là giữ cho hội thoại liền mạch, giúp NPC hiểu được ngữ cảnh. Ví dụ, khi người chơi nói "nó màu gì vậy?", NPC cần tìm trong ký ức ngắn hạn xem "nó" là chỉ cái gì.

Ký ức dài hạn lưu trữ toàn bộ lịch sử hội thoại, sử dụng cơ sở dữ liệu vector để truy xuất ngữ nghĩa. Khi người chơi nhắc đến một chủ đề nào đó, NPC có thể truy xuất các hội thoại lịch sử liên quan từ ký ức dài hạn, nhớ lại nội dung đã từng thảo luận trước đó. Ví dụ, khi người chơi nói "bạn còn nhớ dự án mà chúng ta thảo luận lần trước không?", NPC có thể tìm thấy bản ghi hội thoại liên quan từ ký ức dài hạn.

Kiến trúc hệ thống ký ức được minh họa ở Hình 15.6:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-6.png" alt="" width="85%"/>
  <p>Hình 15.6 Kiến trúc hệ thống ký ức</p>
</div>


Trong thực tế sử dụng, Agent sẽ lấy các hội thoại gần nhất từ ký ức ngắn hạn trước, sau đó truy xuất các hội thoại lịch sử liên quan từ ký ức dài hạn, gửi các thông tin này cùng nhau đến LLM, để sinh ra phản hồi chính xác và cá nhân hóa hơn.

```python
# Luồng xử lý hội thoại của Agent
def process_dialogue(agent, player_message):
    # 1. Lấy hội thoại gần nhất từ ký ức ngắn hạn
    recent_messages = agent.memory_manager.working_memory.get_recent_messages(5)

    # 2. Truy xuất lịch sử liên quan từ ký ức dài hạn
    relevant_memories = agent.memory_manager.episodic_memory.search(
        query=player_message,
        top_k=3
    )

    # 3. Xây dựng ngữ cảnh
    context = {
        "recent": recent_messages,
        "relevant": relevant_memories
    }

    # 4. Gọi Agent sinh phản hồi
    reply = agent.run(player_message, context=context)

    # 5. Lưu vào hệ thống ký ức
    agent.memory_manager.add_interaction(player_message, reply)

    return reply
```

Luồng xử lý này đảm bảo NPC có thể ghi nhớ lịch sử tương tác với người chơi, và thể hiện điều đó trong hội thoại.

### 15.2.4 Sinh hội thoại hàng loạt: chế độ tải nhẹ

Trong quá trình chạy thực tế, chúng ta sẽ sớm phát hiện một vấn đề: khi nhiều người chơi đồng thời trò chuyện với các NPC khác nhau, backend cần xử lý đồng thời nhiều request LLM. Mỗi request đều cần gọi API, điều này không chỉ làm tăng chi phí, mà còn có thể khiến request thất bại hoặc bị trễ do giới hạn concurrency.

Để giải quyết vấn đề này, chúng ta thiết kế một <strong>hệ thống sinh hội thoại hàng loạt (batch)</strong>. Ý tưởng cốt lõi là: gộp các yêu cầu hội thoại của nhiều NPC thành một lần gọi LLM, để LLM sinh ra phản hồi của tất cả NPC trong một lần. Điều này giống như "món ăn chế biến sẵn" của nhà hàng vậy, chuẩn bị hàng loạt trước, dùng ngay khi cần, giảm đáng kể chi phí và độ trễ.

Luồng xử lý của sinh hàng loạt được minh họa ở Hình 15.7:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-7.png" alt="" width="85%"/>
  <p>Hình 15.7 Sinh hàng loạt so với chế độ truyền thống</p>
</div>


Triển khai của bộ sinh hàng loạt rất khéo léo. Chúng ta xây dựng một prompt đặc biệt, yêu cầu LLM sinh hội thoại của tất cả NPC trong một lần, và trả về theo định dạng JSON. Như vậy, một lần gọi API đã có thể lấy được phản hồi của tất cả NPC, chi phí giảm xuống còn 1/3 so với trước, độ trễ cũng giảm đáng kể.

```python
class NPCBatchGenerator：
    """批量生成NPC对话的生成器"""

    def __init__(self):
        self.llm = HelloAgentsLLM()
        self.npc_configs = NPC_ROLES  # 所有NPC的配置

    def generate_batch_dialogues(self, context: Optional[str] = None) -> Dict[str, str]:
        """批量生成所有NPC的对话

        Args:
            context: 场景上下文(如"上午工作时间"、"午餐时间"等)

        Returns:
            Dict[str, str]: NPC名称到对话内容的映射
        """
        # 构建批量生成提示词
        prompt = self._build_batch_prompt(context)

        # 一次LLM调用生成所有对话
        response = self.llm.invoke([
            {"role": "system", "content": "你是一个游戏NPC对话生成器,擅长创作自然真实的办公室对话。"},
            {"role": "user", "content": prompt}
        ])

        # 解析JSON响应
        dialogues = json.loads(response)
        # 返回格式: {"张三": "...", "李四": "...", "王五": "..."}

        return dialogues

    def _build_batch_prompt(self, context: Optional[str] = None) -> str:
        """构建批量生成提示词"""
        # 根据时间自动推断场景
        if context is None:
            context = self._get_current_context()

        # 构建NPC描述
        npc_descriptions = []
        for name, cfg in self.npc_configs.items():
            desc = f"- {name}({cfg['title']}): 在{cfg['location']}{cfg['activity']},性格{cfg['personality']}"
            npc_descriptions.append(desc)

        npc_desc_text = "\n".join(npc_descriptions)

        prompt = f"""请为Datawhale办公室的3个NPC生成当前的对话或行为描述。

【场景】{context}

【NPC信息】
{npc_desc_text}

【生成要求】
1. 每个NPC生成1句话(20-40字)
2. 内容要符合角色设定、当前活动和场景氛围
3. 可以是自言自语、工作状态描述、或简单的思考
4. 要自然真实,像真实的办公室同事
5. **必须严格按照JSON格式返回**

【输出格式】(严格遵守)
{{"张三": "...", "李四": "...", "王五": "..."}}

【示例输出】
{{"张三": "这个bug真是见鬼了,已经调试两小时了...", "李四": "嗯,这个功能的优先级需要重新评估一下。", "王五": "这杯咖啡的拉花真不错,灵感来了!"}}

请生成(只返回JSON,不要其他内容):
"""
        return prompt
```

Điểm then chốt của thiết kế này nằm ở việc xây dựng prompt. Chúng ta yêu cầu rõ ràng LLM trả về định dạng JSON, và cung cấp ví dụ output. LLM sẽ sinh phản hồi nghiêm ngặt theo định dạng này, chúng ta chỉ cần parse JSON là có thể lấy được hội thoại của tất cả NPC.

Sinh hàng loạt còn có một lợi ích phụ: hội thoại của tất cả NPC được sinh ra trong cùng một ngữ cảnh, nên giữa chúng sẽ có một mối liên hệ nhất định. Ví dụ, nếu Trương Tam đang debug bug, Lý Tứ có thể nhắc đến việc sẽ giúp xem thử; nếu Vương Ngũ đang thiết kế giao diện, Trương Tam có thể nói lát nữa sẽ đi xem bản thiết kế. Điều này làm cho bầu không khí của cả văn phòng trở nên chân thực và liền mạch hơn.

Tất nhiên, sinh hàng loạt cũng có một số hạn chế. Nó phù hợp hơn với việc sinh "hội thoại nền" hoặc "tự nói một mình" của NPC, chứ không phải tương tác trực tiếp với người chơi. Đối với hội thoại do người chơi khởi xướng, chúng ta vẫn sử dụng Agent riêng để xử lý, nhằm đảm bảo phản hồi được cá nhân hóa và chính xác. Sinh hàng loạt chủ yếu được dùng trong các trường hợp sau:

1. <strong>Hội thoại nền của NPC</strong>: khi người chơi bước vào bối cảnh, NPC đang làm gì, nói gì
2. <strong>Cập nhật định kỳ</strong>: cứ sau một khoảng thời gian lại cập nhật trạng thái và hội thoại của NPC
3. <strong>Không khí bối cảnh</strong>: sinh các hội thoại khác nhau tùy theo thời gian (sáng, trưa, tối)
4. <strong>Giảm chi phí</strong>: trong các bối cảnh concurrency cao, dùng sinh hàng loạt để giảm số lần gọi API

<strong>Chế độ lai: sinh hàng loạt + phản hồi tức thời</strong>

Trong triển khai thực tế, chúng ta áp dụng một chế độ lai (hybrid mode), kết hợp sinh hàng loạt với phản hồi tức thời. Thiết kế này rất khéo léo, vừa đảm bảo hiệu suất, vừa đảm bảo chất lượng tương tác.

Cụ thể, hệ thống sẽ chạy sinh hàng loạt định kỳ ở background, để sinh "hội thoại nền" của bối cảnh hiện tại cho tất cả NPC. Các hội thoại này sẽ được cache lại, khi người chơi đến gần NPC nhưng chưa khởi xướng tương tác, NPC sẽ hiển thị các hội thoại nền này, ví dụ như "đang debug code...", "đang xem tài liệu sản phẩm...", v.v. Điều này khiến NPC trông như "đang sống", chứ không phải một mô hình đứng yên.

Nhưng, khi người chơi nhấn phím E để khởi xướng tương tác, hệ thống sẽ ngay lập tức chuyển sang chế độ phản hồi tức thời. Lúc này, backend sẽ gọi Agent chuyên biệt của NPC đó, dựa trên tin nhắn cụ thể của người chơi, lịch sử ký ức và độ thiện cảm, để sinh ra phản hồi cá nhân hóa. Quá trình này diễn ra theo thời gian thực, đảm bảo phản hồi của NPC có liên quan cao với input của người chơi.

```python
# Triển khai chế độ lai trong main.py
@app.post("/dialogue")
async def dialogue(request: DialogueRequest):
    """处理玩家与NPC的对话(即时响应模式)"""
    npc_id = request.npc_id
    player_message = request.player_message
    player_name = request.player_name

    # 获取NPC Agent(每个NPC有独立的Agent)
    agent = npc_agents.get(npc_id)
    if not agent:
        raise HTTPException(status_code=404, detail="NPC not found")

    # 即时生成个性化回复
    # 这里不使用批量生成,而是调用Agent的run方法
    reply = agent.run(player_message)

    # 更新好感度
    affinity_change = relationship_manager.update_affinity(
        npc_id, player_name, player_message, reply
    )

    return {
        "npc_reply": reply,
        "affinity_score": affinity_change["score"],
        "affinity_level": affinity_change["level"]
    }

# 后台任务:定期批量生成背景对话
async def background_dialogue_update():
    """后台任务:每5分钟更新一次NPC背景对话"""
    while True:
        try:
            # 使用批量生成器生成所有NPC的背景对话
            batch_generator = get_batch_generator()
            dialogues = batch_generator.generate_batch_dialogues()

            # 更新到状态管理器
            for npc_name, dialogue in dialogues.items():
                state_manager.update_npc_background_dialogue(npc_name, dialogue)

            print(f"✅ 背景对话更新完成: {len(dialogues)}个NPC")
        except Exception as e:
            print(f"❌ 背景对话更新失败: {e}")

        # 等待5分钟
        await asyncio.sleep(300)
```

Lợi thế của chế độ lai này rất rõ ràng:

1. <strong>Giảm chi phí</strong>: hội thoại nền dùng sinh hàng loạt, một lần gọi sinh ra hội thoại của tất cả NPC, chi phí thấp
2. <strong>Đảm bảo chất lượng</strong>: tương tác người chơi dùng phản hồi tức thời, mỗi phản hồi đều được cá nhân hóa, chất lượng cao
3. <strong>Nâng cao trải nghiệm</strong>: NPC luôn có "hội thoại nền", trông rất sống động; khi người chơi tương tác thì phản hồi chính xác, trải nghiệm tốt
4. <strong>Điều chỉnh linh hoạt</strong>: có thể điều chỉnh động tần suất sinh hàng loạt tùy theo tải của server

Thông qua sự kết hợp giữa sinh hàng loạt và phản hồi tức thời, chúng ta đã triển khai một hệ thống NPC vừa hiệu quả vừa thông minh. Trong điều kiện bình thường, người chơi không cảm nhận được bất kỳ sự khác biệt nào, nhưng chi phí và hiệu năng của backend đã được tối ưu đáng kể. Ý tưởng thiết kế này cũng có thể áp dụng cho các bối cảnh khác cần nhiều lượt gọi AI.


## 15.3 Thiết kế hệ thống độ thiện cảm

### 15.3.1 Phân chia cấp độ thiện cảm

Trong Thị trấn Cyber, thái độ của NPC đối với người chơi sẽ thay đổi theo tương tác. Chúng ta thiết kế một hệ thống thiện cảm 5 cấp, từ xa lạ đến tri kỷ, mỗi cấp đều có khoảng điểm và biểu hiện hành vi tương ứng khác nhau.

Ý tưởng cốt lõi của hệ thống thiện cảm là: thông qua việc lượng hóa mối quan hệ giữa NPC và người chơi, giúp phản hồi của NPC chân thực và có chiều sâu hơn. Khi người chơi mới vào game, tất cả NPC đều có thái độ xa lạ với người chơi, phản hồi khá lịch sự nhưng xa cách. Khi hội thoại tiếp diễn, nếu người chơi thể hiện sự thân thiện, độ thiện cảm của NPC sẽ dần tăng lên, phản hồi cũng trở nên gần gũi và chi tiết hơn.

Chúng ta chia độ thiện cảm thành năm cấp, mỗi cấp tương ứng với một khoảng điểm, như Hình 15.8:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-8.png" alt="" width="85%"/>
  <p>Hình 15.8 Phân chia cấp độ thiện cảm</p>
</div>

- <strong>Xa lạ (0-20 điểm)</strong>: NPC vừa mới quen người chơi, thái độ lịch sự nhưng giữ khoảng cách. Phản hồi ngắn gọn, sẽ không chủ động chia sẻ thông tin cá nhân.

- <strong>Quen thuộc (21-40 điểm)</strong>: NPC bắt đầu nhớ người chơi, sẵn lòng giao tiếp đơn giản. Phản hồi trở nên tự nhiên hơn, thỉnh thoảng sẽ chia sẻ một số thông tin liên quan đến công việc.

- <strong>Thân thiện (41-60 điểm)</strong>: NPC coi người chơi như bạn bè, sẵn lòng chia sẻ nhiều thông tin hơn. Phản hồi chi tiết hơn, sẽ chủ động hỏi han tình hình của người chơi.

- <strong>Thân mật (61-80 điểm)</strong>: NPC rất tin tưởng người chơi, sẵn lòng chia sẻ chuyện riêng tư. Phản hồi đầy nhiệt tình, sẽ cung cấp sự giúp đỡ và lời khuyên cho người chơi.

- <strong>Tri kỷ (81-100 điểm)</strong>: NPC coi người chơi là bạn thân nhất, không gì không nói. Phản hồi vô cùng gần gũi, sẽ chia sẻ suy nghĩ và cảm xúc nội tâm.

Thiết kế này giúp người chơi cảm nhận rõ ràng sự thay đổi trong mối quan hệ với NPC, đồng thời cũng tạo nền tảng cho lối chơi sau này. Ví dụ, chỉ khi đạt đến một mức thiện cảm nhất định, NPC mới chia sẻ một số thông tin đặc biệt hoặc cung cấp nhiệm vụ đặc biệt.

### 15.3.2 Logic tính toán độ thiện cảm

Việc tính toán độ thiện cảm cần xem xét nhiều yếu tố. Chúng ta không thể đơn giản để mỗi lần hội thoại đều tăng một số điểm cố định, điều đó sẽ khiến hệ thống trông máy móc và thiếu chân thực. Một hệ thống thiện cảm tốt cần có khả năng nhận diện thái độ của người chơi, và điều chỉnh điểm số động theo nội dung hội thoại.

Trong Thị trấn Cyber, chúng ta dùng LLM để phân tích nội dung hội thoại, phán đoán thái độ của người chơi là thân thiện, trung lập hay không thân thiện. Sau đó dựa vào kết quả phán đoán để điều chỉnh điểm thiện cảm. Quá trình này là tự động, không cần người chơi cố ý chọn lựa chọn nào, giúp tương tác tự nhiên hơn.

Luồng tính toán độ thiện cảm được minh họa ở Hình 15.9:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-9.png" alt="" width="85%"/>
  <p>Hình 15.9 Luồng tính toán độ thiện cảm</p>
</div>


```python
class RelationshipManager:
    """好感度管理器"""

    def __init__(self):
        self.affinity_data = {}  # 存储好感度数据
        self.llm = HelloAgentsLLM()  # 用于分析对话

    def analyze_sentiment(self, player_message: str, npc_reply: str) -> int:
        """分析对话情感,返回好感度变化值"""
        prompt = f"""分析以下对话中玩家的态度:
玩家: {player_message}
NPC: {npc_reply}

请判断玩家的态度是:
1. 友好(+5分): 礼貌、热情、表示感谢或赞同
2. 中立(+2分): 普通的询问或陈述
3. 不友好(-3分): 粗鲁、冷漠、批评或否定

只返回数字,不要其他内容。"""

        response = self.llm.think([{"role": "user", "content": prompt}])
        try:
            score_change = int(response.strip())
            return max(-3, min(5, score_change))  # 限制在-3到5之间
        except:
            return 2  # 默认中立

    def update_affinity(self, npc_id: str, player_name: str,
                       player_message: str, npc_reply: str) -> dict:
        """更新好感度"""
        key = f"{npc_id}_{player_name}"

        # 获取当前好感度
        if key not in self.affinity_data:
            self.affinity_data[key] = {
                "score": 0,
                "level": "陌生",
                "interaction_count": 0
            }

        # 分析对话情感
        score_change = self.analyze_sentiment(player_message, npc_reply)

        # 更新分数
        current_score = self.affinity_data[key]["score"]
        new_score = max(0, min(100, current_score + score_change))

        # 更新等级
        level = self.get_affinity_level(new_score)

        # 更新数据
        self.affinity_data[key].update({
            "score": new_score,
            "level": level,
            "interaction_count": self.affinity_data[key]["interaction_count"] + 1
        })

        return self.affinity_data[key]

    def get_affinity_level(self, score: int) -> str:
        """根据分数获取好感度等级"""
        if score <= 20:
            return "陌生"
        elif score <= 40:
            return "熟悉"
        elif score <= 60:
            return "友好"
        elif score <= 80:
            return "亲密"
        else:
            return "挚友"
```

Triển khai này sử dụng LLM để phân tích nội dung hội thoại, tự động phán đoán thái độ của người chơi và điều chỉnh độ thiện cảm. Thiết kế như vậy giúp hệ thống thiện cảm thông minh và tự nhiên hơn, người chơi không cần cố ý lấy lòng NPC, chỉ cần giao tiếp bình thường là được.

### 15.3.3 Độ thiện cảm ảnh hưởng đến hội thoại

Độ thiện cảm không chỉ đơn thuần là một con số, nó cần thực sự ảnh hưởng đến hành vi của NPC. Trong Thị trấn Cyber, chúng ta thông qua việc sửa đổi system prompt của NPC, để NPC điều chỉnh phong cách phản hồi dựa trên cấp độ thiện cảm hiện tại.

Khi độ thiện cảm thấp, NPC sẽ giữ thái độ lịch sự nhưng xa cách. Khi độ thiện cảm tăng lên, NPC sẽ trở nên nhiệt tình và cởi mở hơn. Sự thay đổi này được thực hiện thông qua việc điều chỉnh động system prompt.

```python
def create_npc_agent_with_affinity(npc_id: str, name: str, role: str,
                                   personality: str, affinity_level: str):
    """创建带好感度的NPC Agent"""

    # 根据好感度等级调整提示词
    affinity_prompts = {
        "陌生": "你刚认识这位玩家,保持礼貌但不要过于热情。回复简短专业。",
        "熟悉": "你已经认识这位玩家,可以进行正常的交流。回复自然友好。",
        "友好": "你把这位玩家当作朋友,愿意分享更多信息。回复详细热情。",
        "亲密": "你非常信任这位玩家,可以分享私人话题。回复充满关心。",
        "挚友": "你把这位玩家当作最好的朋友,无话不谈。回复亲切真诚。"
    }

    system_prompt = f"""你是{name},一位{role}。
你的性格特点:{personality}

当前与玩家的关系:{affinity_level}
{affinity_prompts.get(affinity_level, affinity_prompts["陌生"])}

你在Datawhale办公室工作,与同事们一起推动开源社区的发展。
请根据你的角色、性格和与玩家的关系,自然地回复。
"""

    # 创建Agent
    llm = HelloAgentsLLM()
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt
    )

    return agent
```

Thiết kế này giúp hành vi của NPC thay đổi động theo độ thiện cảm. Người chơi có thể cảm nhận rõ ràng rằng, khi tương tác tăng lên, thái độ của NPC đối với mình đang dần thay đổi, điều này tăng cường đáng kể cảm giác đắm chìm và sự thú vị của game.


## 15.4 Triển khai dịch vụ backend

### 15.4.1 Cấu trúc ứng dụng FastAPI

Backend của Thị trấn Cyber được xây dựng bằng framework FastAPI, phụ trách xử lý request từ frontend Godot, gọi NPC Agent của HelloAgents, quản lý trạng thái NPC và độ thiện cảm, cũng như ghi log. Một cấu trúc ứng dụng rõ ràng giúp code dễ bảo trì và mở rộng hơn.

Ứng dụng FastAPI của chúng ta áp dụng thiết kế module hóa, tách các chức năng khác nhau vào các file khác nhau, như Hình 15.10:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-10.png" alt="" width="85%"/>
  <p>Hình 15.10 Cấu trúc ứng dụng backend</p>
</div>


Hãy bắt đầu từ `main.py`, đây là file đầu vào của ứng dụng FastAPI:

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional
import uvicorn

from agents import NPCAgentManager
from relationship_manager import RelationshipManager
from state_manager import StateManager
from logger import DialogueLogger
from config import settings

# 创建FastAPI应用
app = FastAPI(
    title="赛博小镇后端服务",
    description="基于HelloAgents的AI NPC对话系统",
    version="1.0.0"
)

# 配置CORS,允许Godot前端访问
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 生产环境应该限制具体域名
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 初始化各个管理器
agent_manager = NPCAgentManager()
relationship_manager = RelationshipManager()
state_manager = StateManager()
dialogue_logger = DialogueLogger()

@app.on_event("startup")
async def startup_event():
    """应用启动时的初始化"""
    print("=" * 60)
    print("🎮 赛博小镇后端服务启动中...")
    print("=" * 60)

    # 初始化NPC Agents
    agent_manager.initialize_npcs()
    print("✅ NPC Agents已初始化")

    # 初始化状态管理器
    state_manager.initialize_npcs()
    print("✅ 状态管理器已初始化")

@app.get("/")
async def root():
    """健康检查"""
    return {
        "status": "running",
        "message": "赛博小镇后端服务正在运行",
        "version": "1.0.0",
        "npcs": state_manager.get_npc_count()
    }

if __name__ == "__main__":
    uvicorn.run(
        app,
        host=settings.HOST,
        port=settings.PORT,
        log_level="info"
    )
```

File chương trình chính này định nghĩa cấu trúc cơ bản của ứng dụng FastAPI, cấu hình middleware CORS để cho phép request cross-origin, và khởi tạo các manager khi khởi động. Tiếp theo chúng ta sẽ triển khai các route API cụ thể.

### 15.4.2 Thiết kế route API

Backend của Thị trấn Cyber cần cung cấp một vài endpoint API cốt lõi, để xử lý request từ frontend Godot. Chúng ta sẽ thêm các route này vào `main.py`.

<strong>Lấy trạng thái NPC</strong>

API này trả về trạng thái hiện tại của tất cả NPC, bao gồm vị trí, có đang bận hay không, v.v.:

```python
from models import NPCStatusResponse

@app.get("/npcs/status", response_model=NPCStatusResponse)
async def get_npc_status():
    """获取所有NPC的状态"""
    npcs = state_manager.get_all_npc_states()
    return {"npcs": npcs}

@app.get("/npcs/{npc_id}/status")
async def get_single_npc_status(npc_id: str):
    """获取单个NPC的状态"""
    npc = state_manager.get_npc_state(npc_id)
    if not npc:
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} 不存在")
    return npc
```

<strong>Endpoint hội thoại</strong>

Đây là API cốt lõi nhất, xử lý hội thoại giữa người chơi và NPC:

```python
from models import DialogueRequest, DialogueResponse

@app.post("/dialogue", response_model=DialogueResponse)
async def dialogue(request: DialogueRequest):
    """处理玩家与NPC的对话"""
    # 1. 验证NPC是否存在
    if not agent_manager.has_npc(request.npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {request.npc_id} 不存在")

    # 2. 检查NPC是否忙碌
    if state_manager.is_npc_busy(request.npc_id):
        raise HTTPException(status_code=409, detail=f"NPC {request.npc_id} 正在与其他玩家对话")

    # 3. 标记NPC为忙碌状态
    state_manager.set_npc_busy(request.npc_id, True)

    try:
        # 4. 获取当前好感度
        affinity_info = relationship_manager.get_affinity(
            request.npc_id,
            request.player_name
        )

        # 5. 调用Agent生成回复
        agent = agent_manager.get_agent(request.npc_id, affinity_info["level"])
        reply = agent.run(request.player_message)

        # 6. 更新好感度
        new_affinity = relationship_manager.update_affinity(
            request.npc_id,
            request.player_name,
            request.player_message,
            reply
        )

        # 7. 记录日志
        dialogue_logger.log_dialogue(
            npc_id=request.npc_id,
            player_name=request.player_name,
            player_message=request.player_message,
            npc_reply=reply,
            affinity_info=new_affinity
        )

        # 8. 返回回复
        return DialogueResponse(
            npc_reply=reply,
            affinity_level=new_affinity["level"],
            affinity_score=new_affinity["score"]
        )

    except Exception as e:
        dialogue_logger.log_error(f"对话处理失败: {str(e)}")
        raise HTTPException(status_code=500, detail=f"对话处理失败: {str(e)}")

    finally:
        # 9. 释放NPC状态
        state_manager.set_npc_busy(request.npc_id, False)
```

<strong>Truy vấn độ thiện cảm</strong>

API này cho phép truy vấn độ thiện cảm giữa người chơi và NPC:

```python
from models import AffinityInfo

@app.get("/affinity/{npc_id}/{player_name}", response_model=AffinityInfo)
async def get_affinity(npc_id: str, player_name: str):
    """获取玩家与NPC的好感度"""
    if not agent_manager.has_npc(npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} 不存在")

    affinity = relationship_manager.get_affinity(npc_id, player_name)
    return affinity
```

Luồng gọi route API được minh họa ở Hình 15.11:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-11.png" alt="" width="85%"/>
  <p>Hình 15.11 Luồng gọi API</p>
</div>


### 15.4.3 Quản lý trạng thái và hệ thống log

<strong>Trình quản lý trạng thái (State Manager)</strong>

Trình quản lý trạng thái phụ trách theo dõi trạng thái hiện tại của mỗi NPC, bao gồm vị trí, có đang bận hay không, hành động hiện tại, v.v. Điều này rất quan trọng để ngăn chặn các vấn đề concurrency, ví dụ như tránh việc một NPC đồng thời hội thoại với nhiều người chơi.

```python
# state_manager.py
from typing import Dict, List, Optional
from datetime import datetime

class StateManager:
    """NPC状态管理器"""

    def __init__(self):
        self.npc_states: Dict[str, dict] = {}

    def initialize_npcs(self):
        """初始化NPC状态"""
        npcs = [
            {
                "npc_id": "zhang_san",
                "name": "张三",
                "role": "Python工程师",
                "position": {"x": 300, "y": 200}
            },
            {
                "npc_id": "li_si",
                "name": "李四",
                "role": "产品经理",
                "position": {"x": 500, "y": 200}
            },
            {
                "npc_id": "wang_wu",
                "name": "王五",
                "role": "UI设计师",
                "position": {"x": 700, "y": 200}
            }
        ]

        for npc in npcs:
            self.npc_states[npc["npc_id"]] = {
                **npc,
                "is_busy": False,
                "current_action": "idle",
                "last_interaction": None
            }

    def get_npc_state(self, npc_id: str) -> Optional[dict]:
        """获取NPC状态"""
        return self.npc_states.get(npc_id)

    def get_all_npc_states(self) -> List[dict]:
        """获取所有NPC状态"""
        return list(self.npc_states.values())

    def is_npc_busy(self, npc_id: str) -> bool:
        """检查NPC是否忙碌"""
        npc = self.npc_states.get(npc_id)
        return npc["is_busy"] if npc else False

    def set_npc_busy(self, npc_id: str, busy: bool):
        """设置NPC忙碌状态"""
        if npc_id in self.npc_states:
            self.npc_states[npc_id]["is_busy"] = busy
            if busy:
                self.npc_states[npc_id]["last_interaction"] = datetime.now().isoformat()

    def get_npc_count(self) -> int:
        """获取NPC数量"""
        return len(self.npc_states)
```

<strong>Hệ thống log</strong>

Hệ thống log triển khai xuất đồng thời hai nơi: console và file. Điều này vừa thuận tiện xem theo thời gian thực, vừa có thể lưu lại lịch sử.

```python
# logger.py
import logging
from datetime import datetime
from pathlib import Path

class DialogueLogger:
    """对话日志记录器"""

    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)

        # 创建日志文件名(按日期)
        today = datetime.now().strftime("%Y-%m-%d")
        log_file = self.log_dir / f"dialogue_{today}.log"

        # 配置日志
        self.logger = logging.getLogger("DialogueLogger")
        self.logger.setLevel(logging.INFO)

        # 控制台处理器
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%H:%M:%S'
        )
        console_handler.setFormatter(console_formatter)

        # 文件处理器
        file_handler = logging.FileHandler(log_file, encoding='utf-8')
        file_handler.setLevel(logging.INFO)
        file_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        file_handler.setFormatter(file_formatter)

        # 添加处理器
        self.logger.addHandler(console_handler)
        self.logger.addHandler(file_handler)

    def log_dialogue(self, npc_id: str, player_name: str,
                    player_message: str, npc_reply: str,
                    affinity_info: dict):
        """记录对话"""
        log_message = f"""
{'='*60}
NPC: {npc_id}
玩家: {player_name}
玩家消息: {player_message}
NPC回复: {npc_reply}
好感度: {affinity_info['level']} ({affinity_info['score']}/100)
互动次数: {affinity_info['interaction_count']}
{'='*60}
"""
        self.logger.info(log_message)

    def log_error(self, error_message: str):
        """记录错误"""
        self.logger.error(error_message)
```

Hệ thống log này sẽ hiển thị nội dung hội thoại theo thời gian thực trên console, đồng thời lưu vào file. Log của mỗi ngày sẽ được lưu vào file riêng, thuận tiện cho việc phân tích sau này.

### 15.4.4 Tìm hiểu hệ thống Scene của Godot

Trước khi bắt đầu xây dựng cảnh (scene) trong game, chúng ta cần hiểu khái niệm cốt lõi của Godot — scene (cảnh) và node (nút). Đây là điểm khác biệt lớn nhất của Godot so với các game engine khác, và cũng là một trong những đặc tính mạnh mẽ nhất của nó.

<strong>Node là gì?</strong>

Node là khối xây dựng cơ bản nhất trong Godot. Bạn có thể tưởng tượng node như những viên gạch Lego, mỗi node đều có chức năng riêng. Ví dụ, node Sprite2D dùng để hiển thị hình ảnh, node AudioStreamPlayer dùng để phát âm thanh, node CharacterBody2D dùng để xử lý chuyển động vật lý của nhân vật. Godot cung cấp hàng trăm loại node khác nhau, mỗi loại tập trung làm tốt một việc.

Các node có thể tạo thành quan hệ cha-con, hình thành cấu trúc dạng cây. Node cha có thể ảnh hưởng đến node con, ví dụ di chuyển node cha sẽ đồng thời di chuyển tất cả node con, ẩn node cha sẽ đồng thời ẩn tất cả node con. Quan hệ phân cấp này giúp chúng ta dễ dàng tổ chức và quản lý các đối tượng game phức tạp.

<strong>Scene là gì?</strong>

Scene là một tập hợp các node, được lưu trong một file .tscn. Bạn có thể hiểu scene như một "bản mẫu dựng sẵn" (prefab). Ví dụ, chúng ta có thể tạo một scene "Player", bao gồm sprite nhân vật, collision body, âm thanh và tất cả các node liên quan khác. Sau đó dùng scene này nhiều lần trong game, mỗi lần dùng sẽ tạo ra một instance độc lập.

Sức mạnh của scene nằm ở khả năng tái sử dụng và module hóa. Chúng ta có thể instantiate (khởi tạo instance) một scene khác bên trong một scene, tạo thành cấu trúc lồng nhau. Ví dụ, scene chính có thể chứa scene người chơi, nhiều scene NPC và scene UI. Sửa đổi scene NPC sẽ tự động ảnh hưởng đến tất cả instance NPC, điều này giúp đơn giản hóa đáng kể việc phát triển và bảo trì.

<strong>Một ví dụ đơn giản</strong>

Hãy dùng một ví dụ đơn giản để hiểu scene và node. Giả sử chúng ta muốn tạo một scene "Player":

```
Player (CharacterBody2D)  ← Node gốc, phụ trách chuyển động vật lý
├─ AnimatedSprite2D       ← Node con, hiển thị animation nhân vật
├─ CollisionShape2D       ← Node con, định nghĩa hình dạng va chạm
└─ Camera2D               ← Node con, camera theo dõi người chơi
```

Scene này bao gồm 4 node, hình thành cấu trúc dạng cây. CharacterBody2D là node gốc, ba node còn lại là node con của nó. Chúng ta có thể gắn script cho mỗi node để điều khiển hành vi của nó, cũng có thể gắn script cho node gốc để điều phối tất cả node con.

Khi chúng ta instantiate scene Player này trong scene chính, Godot sẽ tạo ra một bản sao của toàn bộ cây node này. Chúng ta có thể tạo nhiều instance người chơi, mỗi instance đều độc lập, có vị trí, trạng thái và hành vi riêng.

<strong>Lợi thế của việc instantiate scene</strong>

Trong Thị trấn Cyber, chúng ta có ba NPC: Trương Tam, Lý Tứ và Vương Ngũ. Nếu không sử dụng hệ thống scene, chúng ta cần tạo node, thiết lập thuộc tính, viết script riêng cho mỗi NPC, điều này sẽ dẫn đến rất nhiều công việc lặp lại. Còn khi sử dụng hệ thống scene, chúng ta chỉ cần tạo một scene NPC chung, sau đó instantiate ba lần, thông qua tham số script để thiết lập tên và thông tin vai trò khác nhau.

Lợi ích của thiết kế này là: nếu chúng ta muốn thêm một tính năng mới cho tất cả NPC (ví dụ hiển thị bong bóng hội thoại phía trên đầu), chỉ cần sửa đổi scene NPC, tất cả instance sẽ tự động có được tính năng này.

## 15.5 Xây dựng scene game Godot

<strong>Tại sao chọn Godot làm game engine?</strong>

Trong số nhiều game engine, chúng ta chọn Godot 4.5 làm engine frontend, chủ yếu dựa trên các cân nhắc sau:

（1）Godot có lợi thế tự nhiên trong phát triển game 2D</strong>. Thị trấn Cyber là một game góc nhìn từ trên xuống theo phong cách pixel 2D, engine 2D của Godot rất trưởng thành, cung cấp các loại node được thiết kế chuyên biệt cho game 2D như TileMap, AnimatedSprite2D, CharacterBody2D, hiệu suất phát triển cao hơn nhiều so với các engine như Unity. Hệ thống scene của Godot cho phép chúng ta đóng gói các yếu tố như người chơi, NPC, UI thành các scene độc lập, sau đó instantiate trong scene chính, kiểu thiết kế component hóa này rất phù hợp với nhu cầu của chúng ta.

（2）<strong>Godot hoàn toàn mã nguồn mở và miễn phí</strong>. Godot sử dụng giấy phép MIT, không có bất kỳ phí bản quyền hay chia sẻ doanh thu nào, điều này rất thân thiện với các dự án giảng dạy và dự án mã nguồn mở. Bạn có thể tự do sửa đổi mã nguồn engine, cũng có thể thương mại hóa game mà không cần lo lắng về vấn đề bản quyền. Ngược lại, Unity tuy mạnh mẽ nhưng đã đưa ra chính sách phí runtime vào năm 2024, gây ra tranh cãi rộng rãi trong cộng đồng nhà phát triển.

（3）<strong>Chi phí học tập của Godot cực kỳ thấp</strong>. Godot sử dụng GDScript làm ngôn ngữ script chính, đây là một ngôn ngữ động kiểu như Python, cú pháp ngắn gọn dễ hiểu, đường cong học tập rất thoải mái. Đối với độc giả đã quen thuộc với Python, học GDScript hầu như không có rào cản — khai báo biến, định nghĩa hàm, luồng điều khiển đều có cú pháp rất giống Python, bạn thậm chí có thể bắt đầu viết script game chỉ trong vài giờ. Cấu trúc cây node của Godot cũng rất trực quan, bạn có thể trực quan thấy quan hệ phân cấp của scene trong editor, điều này rất thân thiện với người mới bắt đầu.

（4）<strong>Việc tích hợp Godot với backend Python rất đơn giản</strong>. Godot tích hợp sẵn node HTTPRequest, có thể dễ dàng giao tiếp HTTP với backend FastAPI. Chúng ta chỉ cần tạo một script API client, đóng gói tất cả các lệnh gọi API, là có thể gọi khả năng AI của backend trong game. Kiến trúc tách frontend-backend này cho phép chúng ta phát triển và kiểm thử độc lập logic game và logic AI, nâng cao đáng kể hiệu suất phát triển.

Tất nhiên, Godot cũng có một số hạn chế. Ví dụ, khả năng 3D của Godot vẫn còn khoảng cách so với Unreal Engine và Unity, nếu bạn muốn phát triển game 3D lớn, có thể cần cân nhắc engine khác. Nhưng đối với game 2D, game indie và dự án giảng dạy, Godot là một lựa chọn rất tốt.

### 15.5.1 Thiết kế scene và tổ chức tài nguyên

Sau khi hiểu hệ thống scene của Godot, hãy cùng xem thiết kế scene của Thị trấn Cyber. Toàn bộ game bao gồm bốn scene cốt lõi: Main (cảnh chính), Player (người chơi), NPC (nhân vật không phải người chơi) và DialogueUI (giao diện hội thoại). Mỗi scene đều là một module độc lập, có thể chỉnh sửa và kiểm thử riêng, sau đó kết hợp lại với nhau để tạo thành game hoàn chỉnh.

Việc tổ chức scene của Thị trấn Cyber áp dụng thiết kế module hóa. Đầu tiên chúng ta tạo ba scene cơ bản: Player (người chơi), NPC (nhân vật không phải người chơi) và DialogueUI (giao diện hội thoại). Sau đó trong Main (cảnh chính) chúng ta instantiate các scene này và kết hợp chúng lại. Đặc biệt đáng chú ý, ba NPC (Trương Tam, Lý Tứ, Vương Ngũ) đều là instance của cùng một scene NPC, chỉ là thông qua tham số script thiết lập thông tin vai trò khác nhau.

Hãy xem cấu trúc của bốn scene cốt lõi, như Hình 15.12:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-12.png" alt="" width="85%"/>
  <p>Hình 15.12 Bốn scene cốt lõi của Thị trấn Cyber</p>
</div>


Hình này minh họa bốn scene độc lập cùng cấu trúc bên trong của chúng. <strong>Scene 1 (Main)</strong> là scene chính, nó bao gồm hình nền (Sprite2D), instance người chơi, node tổ chức NPCs (bên dưới có ba instance NPC), instance giao diện hội thoại, node tổ chức tường và nhạc nền. Lưu ý, Player, NPC_Zhang, NPC_Li, NPC_Wang và DialogueUI ở đây đều là instance scene, không phải node thông thường. <strong>Scene 2 (Player)</strong> định nghĩa cấu trúc của nhân vật người chơi, bao gồm animation, va chạm, camera và hai node âm thanh. <strong>Scene 3 (NPC)</strong> là một mẫu chung, Trương Tam, Lý Tứ, Vương Ngũ đều là instance của scene này, bao gồm va chạm, animation, vùng tương tác và hai label. <strong>Scene 4 (DialogueUI)</strong> là một node CanvasLayer, bao gồm Panel và các yếu tố UI khác nhau.

Quá trình instantiate scene có thể được hiểu như sau: chúng ta tạo file scene NPC.tscn trong editor Godot, định nghĩa cấu trúc node của NPC. Sau đó trong scene Main, chúng ta "instantiate" scene NPC này ba lần, tạo ra ba bản sao độc lập, lần lượt đặt tên là NPC_Zhang, NPC_Li và NPC_Wang. Mỗi bản sao đều có vị trí và trạng thái riêng, nhưng chúng chia sẻ cùng một cấu trúc node. Nếu chúng ta sửa đổi NPC.tscn, ví dụ thêm một node âm thanh mới cho NPC, thì cả ba instance đều sẽ tự động có được âm thanh đó.

Các bước tạo những scene này trong Godot như sau:

1. <strong>Tạo scene Player</strong>: tạo scene mới, chọn CharacterBody2D làm node gốc, thêm các node con AnimatedSprite2D, CollisionShape2D, Camera2D, InteractSound và RunningSound, lưu thành Player.tscn.

2. <strong>Tạo scene NPC</strong>: tạo scene mới, chọn CharacterBody2D làm node gốc, thêm các node con CollisionShape2D, AnimatedSprite2D, InteractionArea (Area2D, bên dưới có CollisionShape2D), NameLabel và DialogueLabel, lưu thành NPC.tscn.

3. <strong>Tạo scene DialogueUI</strong>: tạo scene mới, chọn CanvasLayer làm node gốc, thêm node con Panel, dưới Panel thêm NPCName, NPCTitle, DialogueText (RichTextLabel), PlayerInput (LineEdit), SendButton và CloseButton, lưu thành DialogueUI.tscn.

4. <strong>Tạo scene Main</strong>: tạo scene mới, chọn Node2D làm node gốc, thêm Background (Sprite2D) làm hình nền, dưới Background thêm trang trí cá voi nhỏ, sau đó instantiate scene Player, tạo node NPCs và bên dưới nó instantiate scene NPC ba lần, instantiate scene DialogueUI, tạo node Walls để tổ chức va chạm tường, cuối cùng thêm AudioStreamPlayer để phát nhạc nền.

Lợi thế của cách tổ chức scene này là: mỗi scene đều độc lập, có thể kiểm thử riêng; các NPC dùng instance của cùng một scene, sửa một lần là ảnh hưởng đến tất cả NPC; các scene giao tiếp qua signal, độ coupling thấp, dễ bảo trì và mở rộng.

### 15.5.2 Triển khai điều khiển người chơi

Nhân vật người chơi là một trong những yếu tố quan trọng nhất của game. Chúng ta cần triển khai điều khiển di chuyển WASD, chuyển đổi animation, phát hiện va chạm, tương tác với NPC, cũng như hệ thống âm thanh.

Cấu trúc scene người chơi bao gồm: một CharacterBody2D làm node gốc, phụ trách chuyển động vật lý và va chạm; một AnimatedSprite2D hiển thị animation nhân vật; một CollisionShape2D định nghĩa hình dạng va chạm; một Camera2D theo dõi người chơi; hai AudioStreamPlayer lần lượt phát âm thanh tương tác và âm thanh đi bộ.

Script điều khiển người chơi `player.gd` triển khai logic di chuyển, tương tác và âm thanh:

```python
extends CharacterBody2D

# 移动速度
@export var speed: float = 200.0

# 当前可交互的NPC
var nearby_npc: Node = null

# 交互状态(交互时禁用移动)
var is_interacting: bool = false

# 节点引用
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var camera: Camera2D = $Camera2D

# 音效引用
@onready var interact_sound: AudioStreamPlayer = null
@onready var running_sound: AudioStreamPlayer = null

# 走路音效状态
var is_playing_running_sound: bool = false

func _ready():
    # 添加到player组(重要!NPC需要通过这个组来识别玩家)
    add_to_group("player")

    # 获取音效节点(可选,如果不存在也不会报错)
    interact_sound = get_node_or_null("InteractSound")
    running_sound = get_node_or_null("RunningSound")

    # 启用相机
    camera.enabled = true

    # 播放默认动画
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

func _physics_process(_delta: float):
    # 如果正在交互,禁用移动
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # 播放idle动画
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        # 停止走路音效
        stop_running_sound()
        return

    # 获取输入方向
    var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")

    # 设置速度
    velocity = input_direction * speed

    # 移动
    move_and_slide()

    # 更新动画和朝向
    update_animation(input_direction)

    # 更新走路音效
    update_running_sound(input_direction)

func update_animation(direction: Vector2):
    """更新角色动画(支持4方向)"""
    if animated_sprite.sprite_frames == null:
        return

    # 根据移动方向播放动画
    if direction.length() > 0:
        # 移动中 - 判断主要方向
        if abs(direction.x) > abs(direction.y):
            # 左右移动
            if direction.x > 0:
                # 向右
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                # 向左
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # 上下移动
            if direction.y > 0:
                # 向下
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                # 向上
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # 静止
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func _input(event: InputEvent):
    # 按E键与NPC交互
    if event is InputEventKey:
        if event.pressed and not event.echo:
            if event.keycode == KEY_E or event.keycode == KEY_ENTER:
                if nearby_npc != null:
                    interact_with_npc()

func interact_with_npc():
    """与附近的NPC交互"""
    if nearby_npc != null:
        # 播放交互音效
        if interact_sound:
            interact_sound.play()

        # 发送信号给对话系统
        get_tree().call_group("dialogue_system", "start_dialogue", nearby_npc.npc_name)

func set_nearby_npc(npc: Node):
    """设置附近的NPC"""
    nearby_npc = npc

func set_interacting(interacting: bool):
    """设置交互状态"""
    is_interacting = interacting
    if interacting:
        # 停止走路音效
        stop_running_sound()

func update_running_sound(direction: Vector2):
    """更新走路音效"""
    if running_sound == null:
        return

    # 如果正在移动
    if direction.length() > 0:
        # 如果音效还没播放,开始播放
        if not is_playing_running_sound:
            running_sound.play()
            is_playing_running_sound = true
    else:
        # 如果停止移动,停止音效
        stop_running_sound()

func stop_running_sound():
    """停止走路音效"""
    if running_sound and is_playing_running_sound:
        running_sound.stop()
        is_playing_running_sound = false
```

Script này triển khai điều khiển người chơi đầy đủ. Người chơi dùng phím WASD (hoặc phím mũi tên) để di chuyển, nhân vật sẽ phát animation 4 hướng tương ứng (walk_up/down/left/right) dựa trên hướng di chuyển. Khi người chơi đến gần NPC, NPC sẽ gọi `set_nearby_npc()` để tự đặt mình là đối tượng có thể tương tác, người chơi nhấn phím E là có thể kích hoạt tương tác. Khi tương tác sẽ phát âm thanh, và thông qua `call_group()` thông báo cho hệ thống hội thoại bắt đầu đối thoại. Trong lúc hội thoại, `set_interacting(true)` sẽ vô hiệu hóa di chuyển của người chơi, sau khi hội thoại kết thúc sẽ khôi phục di chuyển. Âm thanh đi bộ sẽ tự động phát khi người chơi di chuyển, và tự động dừng khi dừng di chuyển.

### 15.5.3 Hành vi và tương tác của NPC

NPC cần triển khai ba chức năng cốt lõi: tuần tra ngẫu nhiên trong scene, phản hồi tương tác của người chơi, hiển thị bong bóng hội thoại. Chúng ta dùng Area2D để phát hiện xem người chơi có đến gần NPC hay không, khi người chơi vào vùng tương tác sẽ thông báo cho người chơi, người chơi nhấn phím E là có thể bắt đầu hội thoại.

Cấu trúc scene NPC bao gồm: CharacterBody2D làm node gốc; CollisionShape2D định nghĩa hình dạng va chạm của NPC; AnimatedSprite2D hiển thị animation NPC; InteractionArea (Area2D) phát hiện người chơi vào vùng tương tác, bên dưới có CollisionShape2D định nghĩa phạm vi tương tác; NameLabel hiển thị tên NPC; DialogueLabel hiển thị bong bóng hội thoại.

Script NPC `npc.gd` triển khai logic tuần tra, tương tác và bong bóng hội thoại:

```python
extends CharacterBody2D

# NPC信息
@export var npc_name: String = "张三"
@export var npc_title: String = "Python工程师"

# NPC外观配置
@export var sprite_frames: SpriteFrames = null  # 自定义精灵帧资源

# NPC移动配置
@export var move_speed: float = 50.0  # 移动速度
@export var wander_enabled: bool = true  # 是否启用巡逻
@export var wander_range: float = 200.0  # 巡逻范围
@export var wander_interval_min: float = 3.0  # 最小巡逻间隔(秒)
@export var wander_interval_max: float = 8.0  # 最大巡逻间隔(秒)

# 当前对话内容(从后端获取)
var current_dialogue: String = ""

# 节点引用
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var interaction_area: Area2D = $InteractionArea
@onready var name_label: Label = $NameLabel
@onready var dialogue_label: Label = $DialogueLabel

# 玩家引用
var player: Node = null

# 巡逻相关变量
var wander_target: Vector2 = Vector2.ZERO  # 巡逻目标位置
var wander_timer: float = 0.0  # 巡逻计时器
var is_wandering: bool = false  # 是否正在巡逻
var is_interacting: bool = false  # 是否正在与玩家交互
var spawn_position: Vector2 = Vector2.ZERO  # 出生位置

func _ready():
    # 添加到npcs组
    add_to_group("npcs")

    # 设置NPC名字
    name_label.text = npc_name

    # 连接交互区域信号
    interaction_area.body_entered.connect(_on_body_entered)
    interaction_area.body_exited.connect(_on_body_exited)

    # 初始化对话标签
    dialogue_label.text = ""
    dialogue_label.visible = false

    # 设置自定义精灵帧(如果有)
    if sprite_frames != null:
        animated_sprite.sprite_frames = sprite_frames

    # 播放默认动画
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

    # 记录出生位置
    spawn_position = global_position

    # 初始化巡逻计时器
    if wander_enabled:
        wander_timer = randf_range(wander_interval_min, wander_interval_max)
        choose_new_wander_target()

func _on_body_entered(body: Node2D):
    """玩家进入交互范围"""
    if body.is_in_group("player"):
        player = body

        if player.has_method("set_nearby_npc"):
            player.set_nearby_npc(self)

func _on_body_exited(body: Node2D):
    """玩家离开交互范围"""
    if body.is_in_group("player"):
        if player != null and player.has_method("set_nearby_npc"):
            player.set_nearby_npc(null)
        player = null

func update_dialogue(dialogue: String):
    """更新NPC对话内容"""
    current_dialogue = dialogue
    dialogue_label.text = dialogue
    dialogue_label.visible = true

    # 10秒后隐藏对话
    await get_tree().create_timer(10.0).timeout
    dialogue_label.visible = false

func _physics_process(delta: float):
    """物理更新 - 处理移动"""
    # 如果正在与玩家交互,停止移动
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # 播放idle动画
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        return

    # 如果未启用巡逻,不移动
    if not wander_enabled:
        return

    # 更新巡逻计时器
    wander_timer -= delta

    # 如果计时器结束,选择新目标并开始移动
    if wander_timer <= 0:
        choose_new_wander_target()
        wander_timer = randf_range(wander_interval_min, wander_interval_max)

    # 如果正在巡逻,移动到目标
    if is_wandering:
        # 检查是否到达目标
        if global_position.distance_to(wander_target) < 10:
            # 到达目标,停止移动
            is_wandering = false
            velocity = Vector2.ZERO
            move_and_slide()
            # 播放idle动画
            if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
                animated_sprite.play("idle")
        else:
            # 继续移动到目标
            var direction = (wander_target - global_position).normalized()
            velocity = direction * move_speed
            move_and_slide()
            # 更新动画
            update_animation(direction)
    else:
        # 停止移动
        velocity = Vector2.ZERO
        move_and_slide()
        # 播放idle动画
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func choose_new_wander_target():
    """选择新的巡逻目标"""
    # 在出生位置附近随机选择一个点
    var offset = Vector2(
        randf_range(-wander_range, wander_range),
        randf_range(-wander_range, wander_range)
    )
    wander_target = spawn_position + offset
    is_wandering = true

func update_animation(direction: Vector2):
    """更新动画"""
    if animated_sprite.sprite_frames == null:
        return

    if direction.length() > 0:
        # 移动动画
        if abs(direction.x) > abs(direction.y):
            # 左右移动
            if direction.x > 0:
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # 上下移动
            if direction.y > 0:
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # 静止动画
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func set_interacting(interacting: bool):
    """设置交互状态"""
    is_interacting = interacting
```

Script này triển khai hành vi đầy đủ của NPC. NPC sẽ tuần tra ngẫu nhiên trong phạm vi `wander_range` gần vị trí xuất hiện, cứ sau mỗi khoảng `wander_interval_min` đến `wander_interval_max` giây sẽ chọn một điểm mục tiêu mới và di chuyển đến đó. Khi di chuyển sẽ phát animation 4 hướng (walk_up/down/left/right), sau khi đến mục tiêu sẽ dừng lại và phát animation idle. Khi người chơi vào InteractionArea, NPC sẽ gọi phương thức `set_nearby_npc(self)` của người chơi, đặt mình làm đối tượng có thể tương tác. Sau khi người chơi nhấn phím E, hệ thống hội thoại sẽ gọi phương thức `set_interacting(true)` của NPC, NPC dừng di chuyển. Sau khi hội thoại kết thúc sẽ gọi `set_interacting(false)`, NPC khôi phục tuần tra. Scene chính sẽ định kỳ gọi phương thức `update_dialogue()` để cập nhật bong bóng hội thoại của NPC, hiển thị nội dung các cuộc hội thoại tự chủ giữa các NPC.


## 15.6 Triển khai giao tiếp giữa frontend và backend

### 15.6.1 Đóng gói API Client

Frontend Godot cần giao tiếp HTTP với backend FastAPI. Chúng ta tạo một script API client `api_client.gd`, đóng gói tất cả các lệnh gọi API, và thiết lập nó làm AutoLoad (tự động tải) singleton, để các script khác có thể dễ dàng sử dụng.

API client dùng node HTTPRequest của Godot để gửi HTTP request. HTTPRequest là một node bất đồng bộ, sau khi gửi request sẽ không chặn game, mà thông qua signal để thông báo khi request hoàn thành. Điều này đảm bảo sự mượt mà của game, ngay cả khi độ trễ mạng cao cũng không bị giật lag. Chúng ta dùng cơ chế signal để thông báo cho các script khác về phản hồi API, thay vì dùng await, để nhiều script có thể đồng thời lắng nghe cùng một phản hồi API.

```python
# api_client.gd
extends Node

# 信号定义
signal chat_response_received(npc_name: String, message: String)
signal chat_error(error_message: String)
signal npc_status_received(dialogues: Dictionary)
signal npc_list_received(npcs: Array)

# HTTP请求节点
var http_chat: HTTPRequest
var http_status: HTTPRequest
var http_npcs: HTTPRequest

func _ready():
    # 创建HTTP请求节点
    http_chat = HTTPRequest.new()
    http_status = HTTPRequest.new()
    http_npcs = HTTPRequest.new()

    add_child(http_chat)
    add_child(http_status)
    add_child(http_npcs)

    # 连接信号
    http_chat.request_completed.connect(_on_chat_request_completed)
    http_status.request_completed.connect(_on_status_request_completed)
    http_npcs.request_completed.connect(_on_npcs_request_completed)

# ==================== 对话API ====================
func send_chat(npc_name: String, message: String) -> void:
    """发送对话请求"""
    var data = {
        "npc_name": npc_name,
        "message": message
    }

    var json_string = JSON.stringify(data)
    var headers = ["Content-Type: application/json"]

    var error = http_chat.request(
        Config.API_CHAT,
        headers,
        HTTPClient.METHOD_POST,
        json_string
    )

    if error != OK:
        print("[ERROR] 发送对话请求失败: ", error)
        chat_error.emit("网络请求失败")

func _on_chat_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """处理对话响应"""
    if response_code != 200:
        print("[ERROR] 对话请求失败: HTTP ", response_code)
        chat_error.emit("服务器错误: " + str(response_code))
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] 解析响应失败")
        chat_error.emit("响应解析失败")
        return

    var response = json.data

    if response.has("success") and response["success"]:
        var npc_name = response["npc_name"]
        var msg = response["message"]
        print("[INFO] 收到NPC回复: ", npc_name, " -> ", msg)
        chat_response_received.emit(npc_name, msg)
    else:
        chat_error.emit("对话失败")

# ==================== NPC状态API ====================
func get_npc_status() -> void:
    """获取NPC状态"""
    # 检查是否正在处理请求
    if http_status.get_http_client_status() != HTTPClient.STATUS_DISCONNECTED:
        print("[WARN] NPC状态请求正在处理中,跳过本次请求")
        return

    var error = http_status.request(Config.API_NPC_STATUS)

    if error != OK:
        print("[ERROR] 获取NPC状态失败: ", error)

func _on_status_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """处理NPC状态响应"""
    if response_code != 200:
        print("[ERROR] NPC状态请求失败: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] 解析NPC状态失败")
        return

    var response = json.data

    if response.has("dialogues"):
        var dialogues = response["dialogues"]
        print("[INFO] 收到NPC状态更新: ", dialogues.size(), "个NPC")
        npc_status_received.emit(dialogues)

# ==================== NPC列表API ====================
func get_npc_list() -> void:
    """获取NPC列表"""
    var error = http_npcs.request(Config.API_NPCS)

    if error != OK:
        print("[ERROR] 获取NPC列表失败: ", error)

func _on_npcs_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """处理NPC列表响应"""
    if response_code != 200:
        print("[ERROR] NPC列表请求失败: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] 解析NPC列表失败")
        return

    var response = json.data

    if response.has("npcs"):
        var npcs = response["npcs"]
        print("[INFO] 收到NPC列表: ", npcs.size(), "个NPC")
        npc_list_received.emit(npcs)
```

API client này đóng gói ba chức năng cốt lõi: gửi yêu cầu hội thoại (`send_chat`), lấy trạng thái NPC (`get_npc_status`) và lấy danh sách NPC (`get_npc_list`). Tất cả các HTTP request đều bất đồng bộ, thông qua signal để thông báo kết quả phản hồi. Chúng ta tạo một node HTTPRequest độc lập cho mỗi API, như vậy có thể gửi nhiều request đồng thời mà không ảnh hưởng lẫn nhau. URL của API được lấy từ singleton Config, thuận tiện cho việc quản lý tập trung. Hệ thống hội thoại lắng nghe signal `chat_response_received` để nhận phản hồi của NPC, scene chính lắng nghe signal `npc_status_received` để cập nhật bong bóng hội thoại của NPC.

### 15.6.2 Triển khai UI hội thoại

UI hội thoại là giao diện tương tác giữa người chơi và NPC. Chúng ta cần thiết kế một hộp thoại gọn gàng và đẹp mắt, bao gồm tên NPC, chức vụ, hiển thị nội dung hội thoại, ô nhập liệu và các nút bấm.

Cấu trúc UI hội thoại được minh họa ở Hình 15.13:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-13.png" alt="" width="85%"/>
  <p>Hình 15.13 Cấu trúc UI hội thoại</p>
</div>


Thiết kế của UI hội thoại rất gọn gàng. DialogueUI là một node CanvasLayer, điều này có nghĩa là nó sẽ luôn hiển thị ở lớp trên cùng của màn hình game, không bị các đối tượng game khác che khuất. Panel là nền của hộp thoại, được neo ở đáy màn hình. Dưới Panel đặt trực tiếp 6 yếu tố UI: NPCName hiển thị tên NPC, NPCTitle hiển thị chức vụ, DialogueText dùng RichTextLabel để hiển thị nội dung hội thoại (hỗ trợ định dạng rich text), PlayerInput là một LineEdit để người chơi nhập liệu, SendButton và CloseButton lần lượt dùng để gửi tin nhắn và đóng hộp thoại.

Script UI hội thoại `dialogue_ui.gd` triển khai logic giao diện hội thoại:

```python
# dialogue_ui.gd
extends CanvasLayer

# UI节点引用
@onready var panel = $Panel
@onready var npc_name_label = $Panel/NPCName
@onready var npc_title_label = $Panel/NPCTitle
@onready var dialogue_text = $Panel/DialogueText
@onready var input_field = $Panel/PlayerInput
@onready var send_button = $Panel/SendButton
@onready var close_button = $Panel/CloseButton

# API客户端
var api_client: Node = null

# 当前对话的NPC
var current_npc_name: String = ""

func _ready():
    # 初始化时隐藏对话框
    visible = false

    # 连接按钮信号
    send_button.pressed.connect(_on_send_button_pressed)
    close_button.pressed.connect(_on_close_button_pressed)
    input_field.text_submitted.connect(_on_text_submitted)

    # 获取API客户端
    api_client = get_node_or_null("/root/APIClient")

func start_dialogue(npc_name: String):
    """开始与NPC对话"""
    current_npc_name = npc_name

    # 设置NPC信息
    npc_name_label.text = npc_name
    npc_title_label.text = get_npc_title(npc_name)

    # 清空对话内容
    dialogue_text.clear()
    dialogue_text.append_text("[color=gray]与 " + npc_name + " 的对话开始...[/color]\n")

    # 清空输入框
    input_field.text = ""

    # 显示对话框
    show_dialogue()

    # 聚焦输入框
    input_field.grab_focus()

func show_dialogue():
    """显示对话框"""
    visible = true

    # 通知玩家进入交互状态(禁用移动)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(true)

func hide_dialogue():
    """隐藏对话框"""
    visible = false
    current_npc_name = ""

    # 通知玩家退出交互状态(启用移动)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(false)

func _on_send_button_pressed():
    """发送按钮点击"""
    send_message()

func _on_close_button_pressed():
    """关闭按钮点击"""
    hide_dialogue()

func _on_text_submitted(_text: String):
    """输入框回车"""
    send_message()

func send_message():
    """发送消息"""
    var message = input_field.text.strip_edges()

    if message.is_empty():
        return

    if current_npc_name.is_empty():
        return

    # 显示玩家消息
    dialogue_text.append_text("\n[color=cyan]玩家:[/color] " + message + "\n")

    # 清空输入框
    input_field.text = ""

    # 禁用输入
    input_field.editable = false
    send_button.disabled = true

    # 发送API请求
    if api_client:
        api_client.send_chat_request(current_npc_name, message)

func on_chat_response_received(npc_name: String, response: String):
    """收到NPC回复"""
    if npc_name == current_npc_name:
        # 显示NPC回复
        dialogue_text.append_text("[color=yellow]" + npc_name + ":[/color] " + response + "\n")

        # 启用输入
        input_field.editable = true
        send_button.disabled = false
        input_field.grab_focus()

func get_npc_title(npc_name: String) -> String:
    """获取NPC职位"""
    var titles = {
        "张三": "Python工程师",
        "李四": "产品经理",
        "王五": "UI设计师"
    }
    return titles.get(npc_name, "")
```

UI hội thoại này triển khai đầy đủ chức năng đối thoại. Người chơi có thể nhập tin nhắn và gửi đi, UI dùng phương thức append_text của RichTextLabel để hiển thị nội dung hội thoại, hỗ trợ định dạng rich text (màu sắc, in đậm, v.v.). Tất cả các lệnh gọi API đều bất đồng bộ, trong khi chờ phản hồi sẽ vô hiệu hóa ô nhập liệu, ngăn gửi trùng lặp. Khi hộp thoại hiển thị sẽ thông báo người chơi vào trạng thái tương tác, vô hiệu hóa di chuyển, khi đóng lại sẽ khôi phục di chuyển.

### 15.6.3 Tích hợp scene chính

Cuối cùng, chúng ta cần tích hợp tất cả các chức năng trong scene chính: điều khiển người chơi, tương tác NPC, UI hội thoại và cập nhật trạng thái NPC. Script scene chính `main.gd` phụ trách điều phối các thành phần này, và định kỳ lấy trạng thái NPC từ backend, cập nhật bong bóng hội thoại của NPC.

```python
# main.gd
extends Node2D

# NPC节点引用
@onready var npc_zhang: Node2D = $NPCs/NPC_Zhang
@onready var npc_li: Node2D = $NPCs/NPC_Li
@onready var npc_wang: Node2D = $NPCs/NPC_Wang

# API客户端
var api_client: Node = null

# NPC状态更新计时器
var status_update_timer: float = 0.0

func _ready():
    print("[INFO] 主场景初始化")

    # 获取API客户端
    api_client = get_node_or_null("/root/APIClient")
    if api_client:
        api_client.npc_status_received.connect(_on_npc_status_received)

        # 立即获取一次NPC状态
        api_client.get_npc_status()
    else:
        print("[ERROR] API客户端未找到")

func _process(delta: float):
    # 定时更新NPC状态
    status_update_timer += delta
    if status_update_timer >= Config.NPC_STATUS_UPDATE_INTERVAL:
        status_update_timer = 0.0
        if api_client:
            api_client.get_npc_status()

func _on_npc_status_received(dialogues: Dictionary):
    """收到NPC状态更新"""
    print("[INFO] 更新NPC状态: ", dialogues)

    # 更新各个NPC的对话
    for npc_name in dialogues:
        var dialogue = dialogues[npc_name]
        update_npc_dialogue(npc_name, dialogue)

func update_npc_dialogue(npc_name: String, dialogue: String):
    """更新指定NPC的对话"""
    var npc_node = get_npc_node(npc_name)
    if npc_node and npc_node.has_method("update_dialogue"):
        npc_node.update_dialogue(dialogue)

func get_npc_node(npc_name: String) -> Node2D:
    """根据名字获取NPC节点"""
    match npc_name:
        "张三":
            return npc_zhang
        "李四":
            return npc_li
        "王五":
            return npc_wang
        _:
            return null
```

Chức năng cốt lõi của script scene chính là định kỳ lấy trạng thái NPC từ backend. Trong `_ready()`, chúng ta lấy reference của singleton APIClient, và kết nối signal `npc_status_received`. Sau đó gọi ngay `get_npc_status()` để lấy trạng thái NPC một lần. Trong `_process()`, chúng ta dùng bộ đếm thời gian để cứ sau mỗi `Config.NPC_STATUS_UPDATE_INTERVAL` giây (mặc định 30 giây) lại gọi `get_npc_status()` một lần. Khi nhận được cập nhật trạng thái NPC, hàm callback `_on_npc_status_received()` sẽ duyệt qua tất cả NPC, gọi phương thức `update_dialogue()` của chúng để cập nhật bong bóng hội thoại. Như vậy, ngay cả khi người chơi không tương tác với NPC, cũng có thể thấy được các cuộc hội thoại tự chủ giữa các NPC.

Toàn bộ luồng giao tiếp frontend-backend được minh họa ở Hình 15.14:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-14.png" alt="" width="85%"/>
  <p>Hình 15.14 Luồng giao tiếp đầy đủ giữa frontend và backend</p>
</div>


Đến đây, tất cả các chức năng giao tiếp giữa frontend và backend đã được triển khai. Người chơi có thể tự do di chuyển trong game, tương tác với NPC, tiến hành đối thoại bằng ngôn ngữ tự nhiên. Đồng thời, scene chính sẽ định kỳ lấy trạng thái NPC từ backend, cập nhật bong bóng hội thoại của NPC, thể hiện các cuộc hội thoại tự chủ giữa các NPC. Toàn bộ hệ thống dùng cơ chế signal để giao tiếp, các thành phần có độ coupling thấp, dễ bảo trì và mở rộng.


## 15.7 Tổng kết và triển vọng

### 15.7.1 Ôn lại chương này

Trong chương này, chúng ta đã hoàn thành một dự án thị trấn AI hoàn chỉnh — Thị trấn Cyber. Dự án này kết hợp framework HelloAgents với game engine Godot, tạo ra một thế giới ảo tràn đầy sức sống. Hãy cùng ôn lại những nội dung cốt lõi đã học được.

<strong>Thiết kế kiến trúc kỹ thuật</strong>

Chúng ta áp dụng kiến trúc phân tách game engine + dịch vụ backend, tách biệt render frontend, logic backend và trí tuệ AI thành các tầng khác nhau. Godot phụ trách hình ảnh game và tương tác người chơi, FastAPI phụ trách dịch vụ API và quản lý trạng thái, HelloAgents phụ trách trí tuệ NPC và hệ thống ký ức. Thiết kế phân tầng này cho phép mỗi phần được phát triển và kiểm thử độc lập, đồng thời tạo nền tảng tốt cho việc mở rộng sau này.

<strong>Hệ thống Agent NPC</strong>

Chúng ta dùng SimpleAgent của HelloAgents để tạo Agent độc lập cho mỗi NPC. Mỗi NPC đều có thiết lập vai trò, đặc điểm tính cách và hệ thống ký ức riêng. Thông qua việc thiết kế cẩn thận system prompt, chúng ta đã biến Trương Tam thành một kỹ sư Python nghiêm túc, Lý Tứ thành một product manager giỏi giao tiếp, Vương Ngũ thành một UI designer giàu tính sáng tạo. Những NPC này không chỉ có thể hiểu hội thoại của người chơi, mà còn có thể phản hồi tương ứng dựa trên đặc điểm vai trò của mình.

<strong>Hệ thống ký ức và độ thiện cảm</strong>

Chúng ta triển khai hệ thống ký ức hai tầng: ký ức ngắn hạn giữ cho hội thoại liền mạch, ký ức dài hạn lưu trữ toàn bộ lịch sử tương tác. Thông qua truy xuất ngữ nghĩa bằng cơ sở dữ liệu vector, NPC có thể nhớ lại các chủ đề đã từng thảo luận trước đó. Hệ thống độ thiện cảm giúp thái độ của NPC đối với người chơi thay đổi theo tương tác, từ xa lạ đến tri kỷ, mỗi cấp độ đều có biểu hiện hành vi khác nhau. Những thiết kế này giúp NPC trông chân thực và thú vị hơn.

<strong>Xây dựng scene game</strong>

Chúng ta dùng Godot để tạo một bối cảnh văn phòng theo phong cách pixel, triển khai điều khiển người chơi, tuần tra NPC, phát hiện tương tác và UI hội thoại. Thông qua thiết kế module hóa của hệ thống scene, chúng ta có thể dễ dàng thêm NPC mới, scene mới và tính năng mới. Cú pháp ngắn gọn của GDScript giúp việc triển khai logic game trở nên trực quan và hiệu quả.

<strong>Giao tiếp frontend-backend</strong>

Chúng ta dùng HTTP REST API để triển khai giao tiếp giữa frontend Godot và backend FastAPI. Thông qua request bất đồng bộ và hệ thống signal, chúng ta đảm bảo sự mượt mà của game, ngay cả khi độ trễ mạng cao cũng không ảnh hưởng đến trải nghiệm người chơi. Việc đóng gói API client giúp các script khác dễ dàng gọi dịch vụ backend, việc triển khai UI hội thoại giúp người chơi có thể giao tiếp tự nhiên với NPC.

Toàn bộ tech stack của dự án được minh họa ở Hình 15.15:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-15.png" alt="" width="85%"/>
  <p>Hình 15.15 Tech stack của Thị trấn Cyber</p>
</div>


### 15.7.2 Hướng mở rộng

Thị trấn Cyber chỉ là một điểm khởi đầu, vẫn còn rất nhiều hướng có thể mở rộng. Những mở rộng này không chỉ tăng cường sự thú vị của game, mà còn khám phá thêm nhiều khả năng của công nghệ AI trong game.

<strong>（1）Hỗ trợ online nhiều người chơi</strong>

Hiện tại Thị trấn Cyber là game đơn người chơi, nhưng chúng ta có thể mở rộng nó thành game online nhiều người chơi. Nhiều người chơi có thể đồng thời vào cùng một văn phòng, tương tác với NPC và những người chơi khác. Điều này cần đưa vào WebSocket để giao tiếp thời gian thực, cũng như cơ sở dữ liệu để lưu trữ bền vững dữ liệu người chơi và trạng thái NPC. NPC có thể nhớ tương tác với các người chơi khác nhau, giữ độ thiện cảm độc lập cho mỗi người chơi.

<strong>（2）Hệ thống nhiệm vụ</strong>

Chúng ta có thể thiết kế hệ thống nhiệm vụ cho NPC. Khi độ thiện cảm giữa người chơi và NPC đạt đến một mức nhất định, NPC sẽ cung cấp nhiệm vụ đặc biệt. Ví dụ Trương Tam có thể nhờ người chơi giúp debug một đoạn code, Lý Tứ có thể nhờ người chơi thu thập phản hồi người dùng, Vương Ngũ có thể nhờ người chơi đánh giá phương án thiết kế. Hoàn thành nhiệm vụ sẽ nhận được phần thưởng, đồng thời cũng tăng thêm độ thiện cảm.

<strong>（3）Tương tác giữa các NPC</strong>

Hiện tại NPC chỉ tương tác với người chơi, nhưng chúng ta có thể để các NPC cũng tương tác với nhau. Trương Tam có thể thảo luận nhu cầu sản phẩm với Lý Tứ, Lý Tứ có thể thảo luận thiết kế giao diện với Vương Ngũ, Vương Ngũ có thể thảo luận triển khai kỹ thuật với Trương Tam. Những tương tác này có thể tự động diễn ra ở background, người chơi có thể quan sát các cuộc hội thoại giữa các NPC, khiến cho cả thế giới trở nên sống động hơn.

<strong>（4）Hệ thống cảm xúc</strong>

Ngoài độ thiện cảm, chúng ta còn có thể bổ sung hệ thống cảm xúc phức tạp hơn cho NPC. NPC có thể có các trạng thái cảm xúc khác nhau như vui, buồn, giận, hưng phấn, những cảm xúc này sẽ ảnh hưởng đến phong cách phản hồi và hành vi của NPC. Ví dụ khi NPC có tâm trạng tốt, sẽ sẵn lòng chia sẻ thông tin hơn; khi NPC có tâm trạng không tốt, có thể sẽ hơi lạnh nhạt.

<strong>（5）Hệ thống sự kiện động</strong>

Chúng ta có thể thiết kế một số sự kiện động, giúp thế giới game phong phú hơn. Ví dụ định kỳ tổ chức họp nhóm, tất cả NPC và người chơi tụ họp lại để thảo luận tiến độ dự án; hoặc tổ chức tiệc sinh nhật, chúc mừng sinh nhật một NPC nào đó; hoặc nhiệm vụ khẩn cấp đột xuất, cần mọi người phối hợp hoàn thành. Những sự kiện này có thể tăng thêm sự biến hóa và thú vị cho game.

<strong>（6）Thế giới rộng lớn hơn</strong>

Hiện tại Thị trấn Cyber chỉ có một bối cảnh văn phòng, nhưng chúng ta có thể mở rộng thành một thế giới lớn hơn. Có thể thêm các bối cảnh khác nhau như quán cà phê, thư viện, công viên, mỗi bối cảnh có NPC và cách tương tác khác nhau. Người chơi có thể di chuyển giữa các bối cảnh khác nhau, khám phá thế giới ảo rộng lớn hơn.

<strong>（7）Học tập cá nhân hóa</strong>

NPC có thể học sở thích và thói quen của từng người chơi. Ví dụ nếu người chơi thường xuyên thảo luận về Python với Trương Tam, NPC sẽ nhớ rằng người chơi quan tâm đến lập trình, sau này sẽ chủ động chia sẻ nội dung liên quan. Nếu người chơi thích chơi game vào buổi tối, NPC sẽ nhớ thói quen thời gian này, hoạt động tích cực hơn vào buổi tối.

### 15.7.3 Suy ngẫm và triển vọng

Thị trấn Cyber đã cho thấy tiềm năng to lớn của công nghệ AI trong game. NPC trong game truyền thống bị giới hạn bởi cây hội thoại và kịch bản được thiết lập sẵn, trong khi AI NPC có thể hiểu và sinh ngôn ngữ tự nhiên, tiến hành đối thoại thực sự với người chơi. Điều này không chỉ nâng cao cảm giác đắm chìm của game, mà còn mang đến những khả năng mới cho thiết kế game.

Nhưng AI NPC cũng đối mặt với một số thách thức. Trước tiên là vấn đề chi phí, mỗi lần hội thoại đều cần gọi LLM API, điều này sẽ phát sinh một khoản phí nhất định. Đối với các game online nhiều người chơi quy mô lớn, chi phí này có thể sẽ rất cao. Tiếp theo là vấn đề độ trễ, việc suy luận của LLM cần thời gian, nếu độ trễ mạng cao, người chơi có thể cần đợi vài giây mới thấy được phản hồi của NPC. Cuối cùng là vấn đề kiểm soát nội dung, nội dung do LLM sinh ra có thể không hoàn toàn kiểm soát được, cần thiết kế prompt tốt và cơ chế lọc nội dung.

Mặc dù có những thách thức này, tương lai của AI NPC vẫn tràn đầy hy vọng. Cùng với sự phát triển của công nghệ LLM, tốc độ suy luận sẽ ngày càng nhanh hơn, chi phí sẽ ngày càng thấp hơn. Các LLM nhỏ chạy cục bộ (local) cũng đang phát triển nhanh chóng, trong tương lai có thể chạy trực tiếp trên thiết bị của người chơi, hoàn toàn không cần request mạng. Sự kết hợp giữa công nghệ AI và game, sẽ mang đến cho người chơi những trải nghiệm chưa từng có.

Trong chương đồ án tốt nghiệp của Phần 5, chúng ta sẽ học cách dùng single-agent và multi-agent để xây dựng một general agent (agent thông dụng), đây sẽ là thời gian sáng tạo của bạn, hãy cùng chờ đón!
