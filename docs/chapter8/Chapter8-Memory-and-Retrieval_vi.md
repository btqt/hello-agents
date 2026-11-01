# Chương 8: Memory và Retrieval (Bộ nhớ và truy xuất)

Trong các chương trước, chúng ta đã xây dựng kiến trúc nền tảng của framework HelloAgents, triển khai nhiều paradigm Agent và hệ thống Tool khác nhau. Tuy nhiên, framework của chúng ta vẫn còn thiếu một năng lực then chốt: <strong>Memory (bộ nhớ)</strong>. Nếu Agent không thể ghi nhớ nội dung tương tác trước đó, cũng không thể học hỏi từ kinh nghiệm lịch sử, thì trong các cuộc hội thoại liên tục hoặc các tác vụ phức tạp, hiệu quả hoạt động của nó sẽ bị hạn chế rất nhiều.

Chương này sẽ dựa trên nền tảng framework được xây dựng ở Chương 7, bổ sung cho HelloAgents hai năng lực cốt lõi: <strong>Memory System (Memory System)</strong> và <strong>Retrieval-Augmented Generation (RAG)</strong>. Chúng ta sẽ áp dụng phương pháp "mở rộng framework + phổ cập kiến thức", trong quá trình xây dựng sẽ hiểu sâu nền tảng lý thuyết của Memory và RAG, cuối cùng hiện thực hóa một hệ thống Agent có đầy đủ năng lực Memory và truy xuất tri thức.


## 8.1 Từ khoa học nhận thức đến Memory của Agent

### 8.1.1 Gợi ý từ hệ thống Memory của con người

Trước khi xây dựng hệ thống Memory cho Agent, hãy cùng tìm hiểu từ góc độ khoa học nhận thức (cognitive science) về cách con người xử lý và lưu trữ thông tin. Memory của con người là một hệ thống nhận thức đa tầng, nó không chỉ lưu trữ thông tin mà còn có thể phân loại và sắp xếp thông tin dựa trên mức độ quan trọng, thời gian và ngữ cảnh. Tâm lý học nhận thức (cognitive psychology) đã cung cấp một framework lý thuyết kinh điển để hiểu cấu trúc và quá trình của Memory<sup>[1]</sup>, như minh họa trong Hình 8.1.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-1.png" alt="人类记忆系统结构图" width="85%"/>
  <p>Hình 8.1 Cấu trúc phân tầng của hệ thống Memory con người</p>
</div>

Theo nghiên cứu của tâm lý học nhận thức, Memory của con người có thể được chia thành các tầng sau:

1. <strong>Sensory Memory (Memory cảm giác)</strong>: Thời gian tồn tại cực ngắn (0.5-3 giây), dung lượng khổng lồ, chịu trách nhiệm lưu giữ tạm thời toàn bộ thông tin mà giác quan tiếp nhận
2. <strong>Working Memory (Memory làm việc)</strong>: Thời gian tồn tại ngắn (15-30 giây), dung lượng hạn chế (7±2 mục), chịu trách nhiệm xử lý thông tin của tác vụ hiện tại
3. <strong>Long-term Memory (Memory dài hạn)</strong>: Thời gian tồn tại dài (có thể kéo dài cả đời), dung lượng gần như vô hạn, được chia nhỏ hơn thành:
   - <strong>Procedural Memory (Memory thủ tục)</strong>: Kỹ năng và thói quen (như đi xe đạp)
   - <strong>Declarative Memory (Memory tường thuật)</strong>: Tri thức có thể diễn đạt bằng ngôn ngữ, lại được chia thành:
     - <strong>Semantic Memory (Memory ngữ nghĩa)</strong>: Tri thức và khái niệm chung (như "Paris là thủ đô nước Pháp")
     - <strong>Episodic Memory (Memory sự kiện)</strong>: Trải nghiệm và sự kiện cá nhân (như "nội dung cuộc họp hôm qua")

### 8.1.2 Vì sao Agent cần Memory và RAG

Dựa trên thiết kế của hệ thống Memory con người, chúng ta có thể hiểu vì sao Agent cũng cần năng lực Memory tương tự. Một đặc điểm quan trọng của trí tuệ con người chính là khả năng ghi nhớ những trải nghiệm trong quá khứ, học hỏi từ đó, và áp dụng những kinh nghiệm này vào các tình huống mới. Tương tự, một Agent thực sự thông minh cũng cần phải có năng lực Memory. Đối với Agent dựa trên LLM, thường phải đối mặt với hai hạn chế căn bản: <strong>sự quên lãng trạng thái hội thoại</strong> và <strong>giới hạn của tri thức tích hợp sẵn</strong>.

(1) Hạn chế 1: Sự quên lãng hội thoại do tính stateless

Các mô hình ngôn ngữ lớn (LLM) hiện nay tuy mạnh mẽ, nhưng về mặt thiết kế lại <strong>stateless (không trạng thái)</strong>. Điều này có nghĩa là, mỗi lần yêu cầu của người dùng (hoặc lời gọi API) đều là một lần tính toán độc lập, không liên quan đến nhau. Bản thân mô hình sẽ không tự động "ghi nhớ" nội dung của cuộc hội thoại trước đó. Điều này dẫn đến một số vấn đề:

1. <strong>Mất ngữ cảnh (context)</strong>: Trong các cuộc hội thoại dài, những thông tin quan trọng ở giai đoạn đầu có thể bị mất do giới hạn của context window
2. <strong>Thiếu cá nhân hóa</strong>: Agent không thể ghi nhớ sở thích, thói quen hay nhu cầu đặc thù của người dùng
3. <strong>Năng lực học tập bị hạn chế</strong>: Không thể học hỏi cải thiện từ những kinh nghiệm thành công hay thất bại trong quá khứ
4. <strong>Vấn đề về tính nhất quán</strong>: Có thể xuất hiện các câu trả lời mâu thuẫn trước sau trong hội thoại nhiều lượt

Hãy cùng xem xét một ví dụ cụ thể để hiểu vấn đề này:

```python
# Cách sử dụng Agent của Chương 7
from hello_agents import SimpleAgent, HelloAgentsLLM

agent = SimpleAgent(name="学习助手", llm=HelloAgentsLLM())

# Cuộc hội thoại lần thứ nhất
response1 = agent.run("我叫张三，正在学习Python，目前掌握了基础语法")
print(response1)  # "很好！Python基础语法是编程的重要基础..."
 
# Cuộc hội thoại lần thứ hai (phiên mới, ví dụ sau khi khởi động lại chương trình rồi tạo lại Agent)
agent = SimpleAgent(name="学习助手", llm=HelloAgentsLLM())
response2 = agent.run("你还记得我的学习进度吗？")
print(response2)  # "抱歉，我不知道您的学习进度..."
```

Cần lưu ý rằng, `SimpleAgent` trong Chương 7 sẽ tạm lưu cuộc hội thoại hiện tại trong `_history` của cùng một instance, do đó trong cùng một process, cùng một instance thì các cuộc hội thoại liên tục có thể mang theo ngữ cảnh gần nhất. Nhưng loại lịch sử này chỉ là danh sách message tạm thời, không được lưu trữ bền vững (persist) xuyên suốt các phiên (session), cũng không thể thực hiện truy xuất, quên lãng và tích hợp dài hạn.

Để giải quyết vấn đề này, framework của chúng ta cần đưa vào hệ thống Memory.

(2) Hạn chế 2: Giới hạn của tri thức tích hợp sẵn trong mô hình

Ngoài việc quên lãng lịch sử hội thoại, một hạn chế cốt lõi khác của LLM nằm ở chỗ tri thức của nó là <strong>tĩnh (static) và hữu hạn</strong>. Những tri thức này hoàn toàn đến từ dữ liệu huấn luyện của nó, và vì vậy dẫn đến một loạt vấn đề:

1. <strong>Tính thời sự của tri thức</strong>: Dữ liệu huấn luyện của mô hình lớn có mốc thời gian cắt (cutoff), không thể lấy được thông tin mới nhất
2. <strong>Tri thức chuyên ngành</strong>: Mô hình tổng quát (general model) có thể thiếu chiều sâu tri thức trong các lĩnh vực đặc thù
3. <strong>Độ chính xác của sự kiện</strong>: Thông qua việc xác minh bằng truy xuất (retrieval), giảm bớt vấn đề ảo giác (hallucination) của mô hình
4. <strong>Khả năng giải thích</strong>: Cung cấp nguồn gốc thông tin, tăng cường độ tin cậy của câu trả lời

Để khắc phục hạn chế này, công nghệ RAG đã ra đời. Ý tưởng cốt lõi của nó là trước khi mô hình sinh ra câu trả lời, trước tiên truy xuất ra những thông tin liên quan nhất từ một knowledge base (kho tri thức) bên ngoài (như tài liệu, cơ sở dữ liệu, API), và cung cấp những thông tin này cùng với ngữ cảnh cho mô hình.

### 8.1.3 Thiết kế kiến trúc hệ thống Memory và RAG

Dựa trên nền tảng framework đã xây dựng ở Chương 7 và những gợi ý từ khoa học nhận thức, chúng ta đã thiết kế một kiến trúc hệ thống Memory và RAG theo kiểu phân tầng, như minh họa trong Hình 8.2. Kiến trúc này không chỉ tham khảo cấu trúc phân tầng của hệ thống Memory con người, mà còn cân nhắc đầy đủ đến khả năng mở rộng trong triển khai kỹ thuật. Về mặt triển khai, chúng ta thiết kế Memory và RAG thành hai Tool độc lập: `memory_tool` chịu trách nhiệm lưu trữ và duy trì thông tin tương tác trong quá trình hội thoại, còn `rag_tool` chịu trách nhiệm truy xuất thông tin liên quan từ knowledge base do người dùng cung cấp để làm ngữ cảnh, và có thể tự động lưu trữ các kết quả truy xuất quan trọng vào hệ thống Memory.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-2.png" alt="HelloAgents记忆与RAG系统架构图" width="95%"/>
  <p>Hình 8.2 Kiến trúc tổng thể của hệ thống Memory và RAG trong HelloAgents</p>
</div>

Hệ thống Memory áp dụng thiết kế kiến trúc bốn tầng:

```
HelloAgents记忆系统
├── 基础设施层 (Infrastructure Layer)
│   ├── MemoryManager - 记忆管理器（统一调度和协调）
│   ├── MemoryItem - 记忆数据结构（标准化记忆项）
│   ├── MemoryConfig - 配置管理（系统参数设置）
│   └── BaseMemory - 记忆基类（通用接口定义）
├── 记忆类型层 (Memory Types Layer)
│   ├── WorkingMemory - 工作记忆（临时信息，TTL管理）
│   ├── EpisodicMemory - 情景记忆（具体事件，时间序列）
│   ├── SemanticMemory - 语义记忆（抽象知识，图谱关系）
│   └── PerceptualMemory - 感知记忆（多模态数据）
├── 存储后端层 (Storage Backend Layer)
│   ├── QdrantVectorStore - 向量存储（高性能语义检索）
│   ├── Neo4jGraphStore - 图存储（知识图谱管理）
│   └── SQLiteDocumentStore - 文档存储（结构化持久化）
└── 嵌入服务层 (Embedding Service Layer)
    ├── DashScopeEmbedding - 通义千问嵌入（云端API）
    ├── LocalTransformerEmbedding - 本地嵌入（离线部署）
    └── TFIDFEmbedding - TFIDF嵌入（轻量级兜底）
```

Hệ thống RAG tập trung vào việc thu thập và tận dụng tri thức bên ngoài:

```
HelloAgents RAG系统
├── 文档处理层 (Document Processing Layer)
│   ├── DocumentProcessor - 文档处理器（多格式解析）
│   ├── Document - 文档对象（元数据管理）
│   └── Pipeline - RAG管道（端到端处理）
├── 嵌入表示层 (Embedding Layer)
│   └── 统一嵌入接口 - 复用记忆系统的嵌入服务
├── 向量存储层 (Vector Storage Layer)
│   └── QdrantVectorStore - 向量数据库（命名空间隔离）
└── 智能问答层 (Intelligent Q&A Layer)
    ├── 多策略检索 - 向量检索 + MQE + HyDE
    ├── 上下文构建 - 智能片段合并与截断
    └── LLM增强生成 - 基于上下文的准确问答
```

### 8.1.4 Mục tiêu học tập và trải nghiệm nhanh của chương này

Hãy cùng xem qua nội dung học tập cốt lõi của Chương 8:

```
hello-agents/
├── hello_agents/
│   ├── memory/                   # 记忆系统模块
│   │   ├── base.py               # 基础数据结构（MemoryItem, MemoryConfig, BaseMemory）
│   │   ├── manager.py            # 记忆管理器（统一协调调度）
│   │   ├── embedding.py          # 统一嵌入服务（DashScope/Local/TFIDF）
│   │   ├── types/                # 记忆类型实现
│   │   │   ├── working.py        # 工作记忆（TTL管理，纯内存）
│   │   │   ├── episodic.py       # 情景记忆（事件序列，SQLite+Qdrant）
│   │   │   ├── semantic.py       # 语义记忆（知识图谱，Qdrant+Neo4j）
│   │   │   └── perceptual.py     # 感知记忆（多模态，SQLite+Qdrant）
│   │   ├── storage/              # 存储后端实现
│   │   │   ├── qdrant_store.py   # Qdrant向量存储（高性能向量检索）
│   │   │   ├── neo4j_store.py    # Neo4j图存储（知识图谱管理）
│   │   │   └── document_store.py # SQLite文档存储（结构化持久化）
│   │   └── rag/                  # RAG系统
│   │       ├── pipeline.py       # RAG管道（端到端处理）
│   │       └── document.py       # 文档处理器（多格式解析）
│   └── tools/builtin/            # 扩展内置工具
│       ├── memory_tool.py        # 记忆工具（Agent记忆能力）
│       └── rag_tool.py           # RAG工具（智能问答能力）
└──
```

<strong>Bắt đầu nhanh: Cài đặt framework HelloAgents</strong>

Để người đọc có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp một package Python có thể cài đặt trực tiếp. Bạn có thể cài đặt phiên bản tương ứng với chương này bằng lệnh sau:

```bash
# 0.2.0版本若遇到模型不可用，查看issue#320或切换0.2.9版本进行测试
pip install "hello-agents[all]==0.2.0"
python -m spacy download zh_core_web_sm
python -m spacy download en_core_web_sm
```

Ngoài ra, còn cần cấu hình API của graph database, vector database, LLM và phương án Embedding trong file `.env`. Trong tutorial này, vector database sử dụng Qdrant, graph database sử dụng Neo4J, Embedding ưu tiên nền tảng Bách Luyện (Bailian), nếu không có API có thể chuyển sang phương án mô hình triển khai local.

```bash
# ================================
# Qdrant 向量数据库配置 - 获取API密钥：https://cloud.qdrant.io/
# ================================
# 使用Qdrant云服务 (推荐)
QDRANT_URL=https://your-cluster.qdrant.tech:6333
QDRANT_API_KEY=your_qdrant_api_key_here

# 或使用本地Qdrant (需要Docker)
# QDRANT_URL=http://localhost:6333
# QDRANT_API_KEY=

# Qdrant集合配置
QDRANT_COLLECTION=hello_agents_vectors
QDRANT_VECTOR_SIZE=384
QDRANT_DISTANCE=cosine
QDRANT_TIMEOUT=30

# ================================
# Neo4j 图数据库配置 - 获取API密钥：https://neo4j.com/cloud/aura/
# ================================
# 使用Neo4j Aura云服务 (推荐)
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password_here

# 或使用本地Neo4j (需要Docker)
# NEO4J_URI=bolt://localhost:7687
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=hello-agents-password

# Neo4j连接配置
NEO4J_DATABASE=neo4j
NEO4J_MAX_CONNECTION_LIFETIME=3600
NEO4J_MAX_CONNECTION_POOL_SIZE=50
NEO4J_CONNECTION_TIMEOUT=60

# ==========================
# 嵌入（Embedding）配置示例 - 可从阿里云控制台获取：https://dashscope.aliyun.com/
# ==========================
# - 若为空，dashscope 默认 text-embedding-v3；local 默认 sentence-transformers/all-MiniLM-L6-v2
EMBED_MODEL_TYPE=dashscope
EMBED_MODEL_NAME=
EMBED_API_KEY=
EMBED_BASE_URL=
```

Việc học tập chương này có thể áp dụng hai cách:

1. <strong>Học theo kiểu trải nghiệm</strong>: Trực tiếp dùng `pip` để cài đặt framework, chạy code ví dụ, nhanh chóng trải nghiệm các chức năng
2. <strong>Học sâu</strong>: Theo sát nội dung của chương, tự xây dựng từ đầu từng component, hiểu sâu tư duy thiết kế và chi tiết triển khai của framework

Chúng tôi khuyến nghị áp dụng lộ trình học tập "trải nghiệm trước, triển khai sau". Trong chương này, chúng tôi cung cấp đầy đủ các file test, bạn có thể viết lại các hàm cốt lõi và chạy test để kiểm tra xem cách triển khai của mình có đúng hay không.

Tuân theo nguyên tắc thiết kế đã được xác lập ở Chương 7, chúng ta sẽ đóng gói năng lực Memory và RAG thành các Tool tiêu chuẩn, thay vì tạo ra một class Agent mới. Trước khi bắt đầu, hãy dùng 30 giây để trải nghiệm việc sử dụng Hello-agents xây dựng một Agent có năng lực Memory và RAG!

```python
# 配置好同级文件夹下.env中的大模型API
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool, RAGTool

# 创建LLM实例
llm = HelloAgentsLLM()

# 创建工具注册表
tool_registry = ToolRegistry()

# 添加记忆工具
memory_tool = MemoryTool(user_id="user123")
tool_registry.register_tool(memory_tool)

# 添加RAG工具
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")
tool_registry.register_tool(rag_tool)

# 创建Agent并配置工具
agent = SimpleAgent(
    name="智能助手",
    llm=llm,
    system_prompt="你是一个有记忆和知识检索能力的AI助手",
    tool_registry=tool_registry
)

# 开始对话
response = agent.run("你好！请记住我叫张三，我是一名Python开发者")
print(response)
```

Nếu mọi cấu hình đã hoàn tất, bạn có thể thấy nội dung như sau.

```bash
[OK] SQLite 数据库表和索引创建完成
[OK] SQLite 文档存储初始化完成: ./memory_data\memory.db
INFO:hello_agents.memory.storage.qdrant_store:✅ 成功连接到Qdrant云服务: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ 使用现有Qdrant集合: hello_agents_vectors
INFO:hello_agents.memory.types.semantic:✅ 嵌入模型就绪，维度: 1024
INFO:hello_agents.memory.types.semantic:✅ Qdrant向量数据库初始化完成
INFO:hello_agents.memory.storage.neo4j_store:✅ 成功连接到Neo4j云服务: neo4j+s://851b3a28.databases.neo4j.io      NFO:hello_agents.memory.types.semantic:✅ Neo4j图数据库初始化完成
INFO:hello_agents.memory.storage.neo4j_store:✅ Neo4j索引创建完成
INFO:hello_agents.memory.types.semantic:✅ Neo4j图数据库初始化完成
INFO:hello_agents.memory.types.semantic:🏥 数据库健康状态: Qdrant=✅, Neo4j=✅
INFO:hello_agents.memory.types.semantic:✅ 加载中文spaCy模型: zh_core_web_sm
INFO:hello_agents.memory.types.semantic:✅ 加载英文spaCy模型: en_core_web_sm
INFO:hello_agents.memory.types.semantic:📚 可用语言模型: 中文, 英文
INFO:hello_agents.memory.types.semantic:增强语义记忆初始化完成（使用Qdrant+Neo4j专业数据库）
INFO:hello_agents.memory.manager:MemoryManager初始化完成，启用记忆类型: ['working', 'episodic', 'semantic']      
✅ 工具 'memory' 已注册。
INFO:hello_agents.memory.storage.qdrant_store:✅ 成功连接到Qdrant云服务: https://0c517275-2ad0-4442-8309-11c36dc7eNFO:hello_agents.memory.storage.qdrant_store:✅ 使用现有Qdrant集合: rag_knowledge_base
811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ 使用现有Qdrant集合: rag_knowledge_base
✅ RAG工具初始化成功: namespace=default, collection=rag_knowledge_base
✅ 工具 'rag' 已注册。
你好，张三！很高兴认识你。作为一名Python开发者，你一定对编程很有热情。如果你有任何技术问题或者需要讨论Python相关 
的话题，随时可以找我。我会尽力帮助你。有什么我现在就能帮到你的吗？
```

## 8.2 Hệ thống Memory: Giúp Agent có khả năng ghi nhớ

### 8.2.1 Luồng hoạt động của hệ thống Memory

Trước khi đi vào giai đoạn triển khai code, chúng ta cần xác định trước luồng hoạt động của hệ thống Memory. Luồng này tham khảo mô hình Memory trong khoa học nhận thức, và ánh xạ mỗi giai đoạn nhận thức thành các component và thao tác kỹ thuật cụ thể. Hiểu rõ mối quan hệ ánh xạ này sẽ giúp ích cho việc triển khai code sau này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-3.png" alt="记忆形成过程" width="90%"/>
  <p>Hình 8.3 Quá trình nhận thức của việc hình thành Memory</p>
</div>

Như minh họa trong Hình 8.3, theo nghiên cứu của khoa học nhận thức, quá trình hình thành Memory của con người trải qua các giai đoạn sau:


1. <strong>Encoding (Mã hóa)</strong>: Chuyển đổi thông tin cảm nhận được thành dạng có thể lưu trữ
2. <strong>Storage (Lưu trữ)</strong>: Lưu giữ thông tin đã mã hóa vào hệ thống Memory
3. <strong>Retrieval (Truy xuất)</strong>: Trích xuất thông tin liên quan từ Memory khi cần
4. <strong>Consolidation (Tích hợp)</strong>: Chuyển hóa Memory ngắn hạn thành Memory dài hạn
5. <strong>Forgetting (Quên lãng)</strong>: Xóa bỏ những thông tin không quan trọng hoặc đã lỗi thời

Dựa trên những gợi ý này, chúng ta đã thiết kế một bộ hệ thống Memory hoàn chỉnh cho HelloAgents. Ý tưởng cốt lõi của nó là mô phỏng cách bộ não con người xử lý các loại thông tin khác nhau, phân chia Memory thành nhiều module chuyên biệt, và xây dựng một cơ chế quản lý thông minh. Hình 8.4 mô tả chi tiết luồng hoạt động của hệ thống này, bao gồm các khâu then chốt như thêm, truy xuất, tích hợp và quên lãng Memory.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-4.png" alt="记忆系统工作流程" width="95%"/>
  <p>Hình 8.4 Luồng hoạt động hoàn chỉnh của hệ thống Memory trong HelloAgents</p>
</div>

Hệ thống Memory của chúng ta được cấu thành từ bốn loại module Memory khác nhau, mỗi module đều được tối ưu hóa cho một kịch bản ứng dụng và vòng đời cụ thể:

Đầu tiên là <strong>Working Memory (Memory làm việc)</strong>, nó đóng vai trò là "Memory ngắn hạn" của Agent, chủ yếu dùng để lưu trữ thông tin ngữ cảnh của cuộc hội thoại hiện tại. Để đảm bảo truy cập và phản hồi tốc độ cao, dung lượng của nó bị giới hạn một cách có chủ đích (ví dụ, mặc định 50 mục), và vòng đời gắn liền với một phiên (session) duy nhất, sẽ tự động dọn dẹp sau khi phiên kết thúc.

Thứ hai là <strong>Episodic Memory (Memory sự kiện)</strong>, nó chịu trách nhiệm lưu trữ dài hạn các sự kiện tương tác cụ thể và kinh nghiệm học tập của Agent. Khác với Working Memory, Episodic Memory chứa thông tin ngữ cảnh phong phú, và hỗ trợ truy xuất hồi tưởng theo trình tự thời gian hoặc theo chủ đề, là nền tảng để Agent "nhìn lại" và học hỏi từ những kinh nghiệm đã qua.

Tương ứng với các sự kiện cụ thể là <strong>Semantic Memory (Memory ngữ nghĩa)</strong>, nó lưu trữ những tri thức, khái niệm và quy tắc trừu tượng hơn. Ví dụ, sở thích của người dùng được biết qua hội thoại, các chỉ thị cần tuân thủ lâu dài hoặc các điểm tri thức chuyên ngành, đều phù hợp để lưu ở đây. Phần Memory này có tính bền vững và quan trọng cao, là cốt lõi để Agent hình thành "hệ thống tri thức" và thực hiện suy luận liên kết (associative reasoning).

Cuối cùng, để đối ứng với tương tác đa phương tiện (multimedia) ngày càng phong phú, chúng ta đưa vào <strong>Perceptual Memory (Memory tri giác)</strong>. Module này chuyên xử lý thông tin đa phương thức (multi-modal) như hình ảnh, âm thanh, và hỗ trợ truy xuất xuyên phương thức (cross-modal). Vòng đời của nó sẽ được quản lý động dựa trên mức độ quan trọng của thông tin và không gian lưu trữ khả dụng.

### 8.2.2 Trải nghiệm nhanh: 30 giây làm quen với chức năng Memory

Trước khi đi sâu vào chi tiết triển khai, hãy cùng trải nghiệm nhanh các chức năng cơ bản của hệ thống Memory:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool

# 创建LLM实例
llm = HelloAgentsLLM()

# 创建记忆工具
memory_tool = MemoryTool(user_id="user123")
tool_registry = ToolRegistry()
tool_registry.register_tool(memory_tool)

# 创建具有记忆能力的Agent
agent = SimpleAgent(name="记忆助手", llm=llm, tool_registry=tool_registry)
 
# 体验记忆功能
print("=== 添加多个记忆 ===")

# 添加第一个记忆
result1 = memory_tool.run("add", content="用户张三是一名Python开发者，专注于机器学习和数据分析", memory_type="semantic", importance=0.8)
print(f"记忆1: {result1}")

# 添加第二个记忆
result2 = memory_tool.run("add", content="李四是前端工程师，擅长React和Vue.js开发", memory_type="semantic", importance=0.7)
print(f"记忆2: {result2}")

# 添加第三个记忆
result3 = memory_tool.run("add", content="王五是产品经理，负责用户体验设计和需求分析", memory_type="semantic", importance=0.6)
print(f"记忆3: {result3}")

print("\n=== 搜索特定记忆 ===")
# 搜索前端相关的记忆
print("🔍 搜索 '前端工程师':")
result = memory_tool.run("search", query="前端工程师", limit=3)
print(result)

print("\n=== 记忆摘要 ===")
result = memory_tool.run("summary")
print(result)
```

### 8.2.3 Giải thích chi tiết MemoryTool

Bây giờ hãy áp dụng cách tiếp cận từ trên xuống (top-down), bắt đầu từ các thao tác cụ thể mà MemoryTool hỗ trợ, rồi dần dần đi sâu vào triển khai bên dưới. MemoryTool với vai trò là giao diện thống nhất của hệ thống Memory, thiết kế của nó tuân theo mô hình kiến trúc "điểm vào thống nhất, phân phối xử lý":

````python
def execute(self, action: str, **kwargs) -> str:
    """执行记忆操作

    支持的操作：
    - add: 添加记忆（支持4种类型: working/episodic/semantic/perceptual）
    - search: 搜索记忆
    - summary: 获取记忆摘要
    - stats: 获取统计信息
    - update: 更新记忆
    - remove: 删除记忆
    - forget: 遗忘记忆（多种策略）
    - consolidate: 整合记忆（短期→长期）
    - clear_all: 清空所有记忆
    """

    if action == "add":
        return self._add_memory(**kwargs)
    elif action == "search":
        return self._search_memory(**kwargs)
    elif action == "summary":
        return self._get_summary(**kwargs)
    # ... 其他操作
````

Thiết kế giao diện `execute` thống nhất này đơn giản hóa cách Agent gọi Tool, thông qua tham số `action` để chỉ định thao tác cụ thể, sử dụng `**kwargs` để cho phép mỗi thao tác có yêu cầu tham số khác nhau. Ở đây chúng ta sẽ liệt kê những thao tác tương đối quan trọng:

(1) Thao tác 1: add

Thao tác `add` là nền tảng của hệ thống Memory, nó mô phỏng quá trình bộ não con người mã hóa thông tin cảm nhận được thành Memory. Trong triển khai, chúng ta không chỉ lưu trữ nội dung Memory, mà còn bổ sung thông tin ngữ cảnh phong phú cho mỗi Memory, những thông tin này sẽ phát huy vai trò quan trọng trong việc truy xuất và quản lý sau này.

````python
def _add_memory(
    self,
    content: str = "",
    memory_type: str = "working",
    importance: float = 0.5,
    file_path: str = None,
    modality: str = None,
    **metadata
) -> str:
    """添加记忆"""
    try:
        # 确保会话ID存在
        if self.current_session_id is None:
            self.current_session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # 感知记忆文件支持
        if memory_type == "perceptual" and file_path:
            inferred = modality or self._infer_modality(file_path)
            metadata.setdefault("modality", inferred)
            metadata.setdefault("raw_data", file_path)

        # 添加会话信息到元数据
        metadata.update({
            "session_id": self.current_session_id,
            "timestamp": datetime.now().isoformat()
        })

        memory_id = self.memory_manager.add_memory(
            content=content,
            memory_type=memory_type,
            importance=importance,
            metadata=metadata,
            auto_classify=False
        )

        return f"✅ 记忆已添加 (ID: {memory_id[:8]}...)"

    except Exception as e:
        return f"❌ 添加记忆失败: {str(e)}"
````

Ở đây chủ yếu triển khai ba nhiệm vụ then chốt: quản lý tự động session ID (đảm bảo mỗi Memory đều có归属 session rõ ràng), xử lý thông minh dữ liệu đa phương thức (tự động suy luận loại file và lưu metadata liên quan), cũng như tự động bổ sung thông tin ngữ cảnh (thêm timestamp và thông tin session cho mỗi Memory). Trong đó, tham số `importance` (mặc định 0.5) dùng để đánh dấu mức độ quan trọng của Memory, phạm vi giá trị 0.0-1.0, cơ chế này mô phỏng việc bộ não con người đánh giá tầm quan trọng của các thông tin khác nhau. Thiết kế này giúp Agent tự động phân biệt các cuộc hội thoại ở các khoảng thời gian khác nhau, và cung cấp ngữ cảnh phong phú cho việc truy xuất và quản lý sau này.

Trong đó, đối với mỗi loại Memory, chúng tôi cung cấp các ví dụ sử dụng khác nhau:

```python
# 1. 工作记忆 - 临时信息，容量有限
memory_tool.run("add",
    content="用户刚才问了关于Python函数的问题",
    memory_type="working",
    importance=0.6
)

# 2. 情景记忆 - 具体事件和经历
memory_tool.run("add",
    content="2024年3月15日，用户张三完成了第一个Python项目",
    memory_type="episodic",
    importance=0.8,
    event_type="milestone",
    location="在线学习平台"
)

# 3. 语义记忆 - 抽象知识和概念
memory_tool.run("add",
    content="Python是一种解释型、面向对象的编程语言",
    memory_type="semantic",
    importance=0.9,
    knowledge_type="factual"
)

# 4. 感知记忆 - 多模态信息
memory_tool.run("add",
    content="用户上传了一张Python代码截图，包含函数定义",
    memory_type="perceptual",
    importance=0.7,
    modality="image",
    file_path="./uploads/code_screenshot.png"
)
```

(2) Thao tác 2: search

Thao tác `search` là chức năng cốt lõi của hệ thống Memory, nó cần tìm nhanh nội dung liên quan nhất với câu truy vấn (query) trong một lượng lớn Memory. Nó liên quan đến nhiều khâu như hiểu ngữ nghĩa (semantic understanding), tính toán độ liên quan (relevance) và sắp xếp kết quả.

````python
def _search_memory(
    self,
    query: str,
    limit: int = 5,
    memory_types: List[str] = None,
    memory_type: str = None,
    min_importance: float = 0.1
) -> str:
    """搜索记忆"""
    try:
        # 参数标准化处理
        if memory_type and not memory_types:
            memory_types = [memory_type]

        results = self.memory_manager.retrieve_memories(
            query=query,
            limit=limit,
            memory_types=memory_types,
            min_importance=min_importance
        )

        if not results:
            return f"🔍 未找到与 '{query}' 相关的记忆"

        # 格式化结果
        formatted_results = []
        formatted_results.append(f"🔍 找到 {len(results)} 条相关记忆:")

        for i, memory in enumerate(results, 1):
            memory_type_label = {
                "working": "工作记忆",
                "episodic": "情景记忆", 
                "semantic": "语义记忆",
                "perceptual": "感知记忆"
            }.get(memory.memory_type, memory.memory_type)

            content_preview = memory.content[:80] + "..." if len(memory.content) > 80 else memory.content
            formatted_results.append(
                f"{i}. [{memory_type_label}] {content_preview} (重要性: {memory.importance:.2f})"
            )

        return "\n".join(formatted_results)

    except Exception as e:
        return f"❌ 搜索记忆失败: {str(e)}"
````

Thao tác search về mặt thiết kế hỗ trợ cả hai dạng tham số số ít và số nhiều (`memory_type` và `memory_types`), giúp người dùng diễn đạt nhu cầu theo cách tự nhiên nhất. Trong đó, tham số `min_importance` (mặc định 0.1) dùng để lọc bỏ những Memory chất lượng thấp. Đối với cách sử dụng chức năng search, có thể tham khảo ví dụ sau.

```python
# 基础搜索
result = memory_tool.execute("search", query="Python编程", limit=5)

# 指定记忆类型搜索
result = memory_tool.execute("search",
    query="学习进度",
    memory_type="episodic",
    limit=3
)

# 多类型搜索
result = memory_tool.execute("search",
    query="函数定义",
    memory_types=["semantic", "episodic"],
    min_importance=0.5
)
```

(3) Thao tác 3: forget

Cơ chế quên lãng (forgetting) là chức năng mang đậm màu sắc khoa học nhận thức nhất, nó mô phỏng quá trình quên lãng chọn lọc của bộ não con người, hỗ trợ ba chiến lược: dựa trên mức độ quan trọng (xóa Memory không quan trọng), dựa trên thời gian (xóa Memory đã lỗi thời) và dựa trên dung lượng (khi dung lượng lưu trữ gần đạt giới hạn, xóa những Memory kém quan trọng nhất).

````python
def _forget(self, strategy: str = "importance_based", threshold: float = 0.1, max_age_days: int = 30) -> str:
    """遗忘记忆（支持多种策略）"""
    try:
        count = self.memory_manager.forget_memories(
            strategy=strategy,
            threshold=threshold,
            max_age_days=max_age_days
        )
        return f"🧹 已遗忘 {count} 条记忆（策略: {strategy}）"
    except Exception as e:
        return f"❌ 遗忘记忆失败: {str(e)}"
````

<strong>Cách sử dụng ba chiến lược quên lãng:</strong>

```python
# 1. 基于重要性的遗忘 - 删除重要性低于阈值的记忆
memory_tool.execute("forget",
    strategy="importance_based",
    threshold=0.2
)

# 2. 基于时间的遗忘 - 删除超过指定天数的记忆
memory_tool.execute("forget",
    strategy="time_based",
    max_age_days=30
)

# 3. 基于容量的遗忘 - 当记忆数量超限时删除最不重要的
memory_tool.execute("forget",
    strategy="capacity_based",
    threshold=0.3
)
```

(4) Thao tác 4: consolidate

````python
def _consolidate(self, from_type: str = "working", to_type: str = "episodic", importance_threshold: float = 0.7) -> str:
    """整合记忆（将重要的短期记忆提升为长期记忆）"""
    try:
        count = self.memory_manager.consolidate_memories(
            from_type=from_type,
            to_type=to_type,
            importance_threshold=importance_threshold,
        )
        return f"🔄 已整合 {count} 条记忆为长期记忆（{from_type} → {to_type}，阈值={importance_threshold}）"
    except Exception as e:
        return f"❌ 整合记忆失败: {str(e)}"
````

Thao tác consolidate tham khảo khái niệm memory consolidation (tích hợp Memory) trong khoa học thần kinh, mô phỏng quá trình bộ não con người chuyển hóa Memory ngắn hạn thành Memory dài hạn. Cài đặt mặc định là chuyển đổi Working Memory có mức độ quan trọng vượt quá 0.7 thành Episodic Memory, ngưỡng (threshold) này đảm bảo chỉ những thông tin thực sự quan trọng mới được lưu trữ lâu dài. Toàn bộ quá trình được tự động hóa, người dùng không cần tự tay chọn Memory cụ thể, hệ thống sẽ thông minh nhận diện những Memory phù hợp điều kiện và thực hiện chuyển đổi loại.

<strong>Ví dụ sử dụng tính năng tích hợp Memory:</strong>

```python
# 将重要的工作记忆转为情景记忆
memory_tool.execute("consolidate",
    from_type="working",
    to_type="episodic",
    importance_threshold=0.7
)

# 将重要的情景记忆转为语义记忆
memory_tool.execute("consolidate",
    from_type="episodic",
    to_type="semantic",
    importance_threshold=0.8
)
```

Thông qua sự phối hợp của các thao tác cốt lõi trên, MemoryTool đã xây dựng nên một hệ thống quản lý vòng đời Memory hoàn chỉnh. Từ việc tạo, truy xuất, tóm tắt cho đến quên lãng, tích hợp và quản lý Memory, hình thành nên một hệ thống quản lý Memory thông minh khép kín, giúp Agent thực sự sở hữu năng lực Memory giống con người.

### 8.2.4 Giải thích chi tiết MemoryManager

Sau khi hiểu thiết kế giao diện của MemoryTool, hãy cùng đi sâu vào triển khai bên dưới, xem MemoryTool phối hợp với MemoryManager như thế nào. Thiết kế phân tầng này thể hiện nguyên tắc tách biệt mối quan tâm (separation of concerns) trong software engineering, MemoryTool tập trung vào giao diện người dùng và xử lý tham số, còn MemoryManager chịu trách nhiệm logic quản lý Memory cốt lõi.

Khi khởi tạo, MemoryTool sẽ tạo một instance MemoryManager, và kích hoạt các loại module Memory khác nhau dựa trên cấu hình. Thiết kế này cho phép người dùng lựa chọn kích hoạt loại Memory nào dựa trên nhu cầu cụ thể, vừa đảm bảo tính đầy đủ của chức năng, vừa tránh tiêu tốn tài nguyên không cần thiết.

````python
class MemoryTool(Tool):
    """记忆工具 - 为Agent提供记忆功能"""
    
    def __init__(
        self,
        user_id: str = "default_user",
        memory_config: MemoryConfig = None,
        memory_types: List[str] = None
    ):
        super().__init__(
            name="memory",
            description="记忆工具 - 可以存储和检索对话历史、知识和经验"
        )
        
        # 初始化记忆管理器
        self.memory_config = memory_config or MemoryConfig()
        self.memory_types = memory_types or ["working", "episodic", "semantic"]
        
        self.memory_manager = MemoryManager(
            config=self.memory_config,
            user_id=user_id,
            enable_working="working" in self.memory_types,
            enable_episodic="episodic" in self.memory_types,
            enable_semantic="semantic" in self.memory_types,
            enable_perceptual="perceptual" in self.memory_types
        )
````
MemoryManager với vai trò điều phối viên cốt lõi của hệ thống Memory, chịu trách nhiệm quản lý các module Memory thuộc nhiều loại khác nhau, và cung cấp một giao diện thao tác thống nhất. Năng lực lưu trữ và truy xuất cụ thể được từng loại Memory tự triển khai bên trong.

````python
class MemoryManager:
    """记忆管理器 - 统一的记忆操作接口"""

    def __init__(
        self,
        config: Optional[MemoryConfig] = None,
        user_id: str = "default_user",
        enable_working: bool = True,
        enable_episodic: bool = True,
        enable_semantic: bool = True,
        enable_perceptual: bool = False
    ):
        self.config = config or MemoryConfig()
        self.user_id = user_id

        # 存储和检索功能由各记忆类型内部实现

        # 初始化各类型记忆
        self.memory_types = {}

        if enable_working:
            self.memory_types['working'] = WorkingMemory(self.config)

        if enable_episodic:
            self.memory_types['episodic'] = EpisodicMemory(self.config)

        if enable_semantic:
            self.memory_types['semantic'] = SemanticMemory(self.config)

        if enable_perceptual:
            self.memory_types['perceptual'] = PerceptualMemory(self.config)
````
### 8.2.5 Bốn loại Memory

Bây giờ hãy cùng tìm hiểu sâu về cách triển khai cụ thể của bốn loại Memory, mỗi loại Memory đều có đặc điểm và kịch bản ứng dụng riêng:

(1) Working Memory (WorkingMemory)

Working Memory là phần hoạt động tích cực nhất trong hệ thống Memory, nó chịu trách nhiệm lưu trữ thông tin tạm thời trong phiên hội thoại hiện tại. Trọng tâm thiết kế của Working Memory nằm ở việc truy cập nhanh và tự động dọn dẹp, thiết kế này đảm bảo tốc độ phản hồi và hiệu quả tài nguyên của hệ thống.

Working Memory áp dụng phương án lưu trữ hoàn toàn trong bộ nhớ (in-memory), kết hợp với cơ chế TTL (Time To Live) để tự động dọn dẹp. Ưu điểm của thiết kế này là tốc độ truy cập cực nhanh, nhưng cũng đồng nghĩa với việc nội dung của Working Memory sẽ bị mất sau khi hệ thống khởi động lại. Đặc tính này phù hợp đúng với định vị của Working Memory, lưu trữ thông tin tạm thời, dễ thay đổi.


````python
class WorkingMemory:
    """工作记忆实现
    特点：
    - 容量有限（默认50条）+ TTL自动清理
    - 纯内存存储，访问速度极快
    - 组合词法检索：TF-IDF 词项权重相似度 + 关键词匹配
    """
    
    def __init__(self, config: MemoryConfig):
        self.max_capacity = config.working_memory_capacity or 50
        self.max_age_minutes = config.working_memory_ttl or 60
        self.memories = []
    
    def add(self, memory_item: MemoryItem) -> str:
        """添加工作记忆"""
        self._expire_old_memories()  # 过期清理
        
        if len(self.memories) >= self.max_capacity:
            self._remove_lowest_priority_memory()  # 容量管理
        
        self.memories.append(memory_item)
        return memory_item.id
    
    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """组合词法检索：TF-IDF 词项权重相似度 + 关键词匹配"""
        self._expire_old_memories()
        
        # 计算 TF-IDF 词项权重相似度
        vector_scores = self._try_tfidf_search(query)
        
        # 计算综合分数
        scored_memories = []
        for memory in self.memories:
            vector_score = vector_scores.get(memory.id, 0.0)
            keyword_score = self._calculate_keyword_score(query, memory.content)
            
            # 混合评分
            base_relevance = vector_score * 0.7 + keyword_score * 0.3 if vector_score > 0 else keyword_score
            time_decay = self._calculate_time_decay(memory.timestamp)
            importance_weight = 0.8 + (memory.importance * 0.4)
            
            final_score = base_relevance * time_decay * importance_weight
            if final_score > 0:
                scored_memories.append((final_score, memory))
        
        scored_memories.sort(key=lambda x: x[0], reverse=True)
        return [memory for _, memory in scored_memories[:limit]]
````
Working Memory áp dụng chiến lược truy xuất từ vựng (lexical retrieval) kết hợp: sử dụng TF-IDF để biểu diễn văn bản thành vector từ vựng thưa (sparse term vector) và tính độ tương đồng từ vựng, đồng thời kết hợp với khớp từ khóa (keyword matching); khi TF-IDF không khả dụng hoặc không cho ra điểm số hợp lệ, thì sử dụng điểm khớp từ khóa. Biểu diễn vector hóa của TF-IDF đến từ tần suất từ (term frequency) và tần suất tài liệu nghịch đảo (inverse document frequency), không tương đương với truy xuất ngữ nghĩa (semantic retrieval) dựa trên dense embedding. Thuật toán tính điểm kết hợp độ liên quan từ vựng, sự suy giảm theo thời gian (time decay) và trọng số quan trọng, công thức điểm cuối cùng là: `(độ liên quan × hệ số suy giảm thời gian) × (0.8 + mức độ quan trọng × 0.4)`.

(2) Episodic Memory (EpisodicMemory)

Episodic Memory chịu trách nhiệm lưu trữ các sự kiện và trải nghiệm cụ thể, trọng tâm thiết kế của nó nằm ở việc duy trì tính toàn vẹn của sự kiện và mối quan hệ trình tự thời gian. Episodic Memory áp dụng phương án lưu trữ lai (hybrid) SQLite+Qdrant, SQLite chịu trách nhiệm lưu trữ dữ liệu có cấu trúc (structured) và truy vấn phức tạp, Qdrant chịu trách nhiệm truy xuất vector hiệu quả.

````python
class EpisodicMemory:
    """情景记忆实现
    特点：
    - SQLite+Qdrant混合存储架构
    - 支持时间序列和会话级检索
    - 结构化过滤 + 语义向量检索
    """
    
    def __init__(self, config: MemoryConfig):
        self.doc_store = SQLiteDocumentStore(config.database_path)
        self.vector_store = QdrantVectorStore(config.qdrant_url, config.qdrant_api_key)
        self.embedder = create_embedding_model_with_fallback()
        self.sessions = {}  # 会话索引
    
    def add(self, memory_item: MemoryItem) -> str:
        """添加情景记忆"""
        # 创建情景对象
        episode = Episode(
            episode_id=memory_item.id,
            session_id=memory_item.metadata.get("session_id", "default"),
            timestamp=memory_item.timestamp,
            content=memory_item.content,
            context=memory_item.metadata
        )
        
        # 更新会话索引
        session_id = episode.session_id
        if session_id not in self.sessions:
            self.sessions[session_id] = []
        self.sessions[session_id].append(episode.episode_id)
        
        # 持久化存储（SQLite + Qdrant）
        self._persist_episode(episode)
        return memory_item.id
    
    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """混合检索：结构化过滤 + 语义向量检索"""
        # 1. 结构化预过滤（时间范围、重要性等）
        candidate_ids = self._structured_filter(**kwargs)
        
        # 2. 向量语义检索
        hits = self._vector_search(query, limit * 5, kwargs.get("user_id"))
        
        # 3. 综合评分与排序
        results = []
        for hit in hits:
            if self._should_include(hit, candidate_ids, kwargs):
                score = self._calculate_episode_score(hit)
                memory_item = self._create_memory_item(hit)
                results.append((score, memory_item))
        
        results.sort(key=lambda x: x[0], reverse=True)
        return [item for _, item in results[:limit]]
    
    def _calculate_episode_score(self, hit) -> float:
        """情景记忆评分算法"""
        vec_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)
        
        # 评分公式：(向量相似度 × 0.8 + 时间近因性 × 0.2) × 重要性权重
        base_relevance = vec_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)
        
        return base_relevance * importance_weight
````
Cách triển khai truy xuất của Episodic Memory thể hiện một cơ chế tính điểm đa yếu tố phức tạp. Nó không chỉ xem xét độ tương đồng ngữ nghĩa (semantic similarity), mà còn đưa vào yếu tố tính gần đây theo thời gian (recency), cuối cùng được điều chỉnh thông qua trọng số quan trọng. Công thức tính điểm là: `(độ tương đồng vector × 0.8 + tính gần đây về thời gian × 0.2) × (0.8 + mức độ quan trọng × 0.4)`, đảm bảo kết quả truy xuất vừa có liên quan về ngữ nghĩa vừa có liên quan về thời gian.

(3) Semantic Memory (SemanticMemory)

Semantic Memory là phần phức tạp nhất trong hệ thống Memory, nó chịu trách nhiệm lưu trữ các khái niệm, quy tắc và tri thức trừu tượng. Trọng tâm thiết kế của Semantic Memory nằm ở việc biểu diễn tri thức có cấu trúc và năng lực suy luận thông minh. Semantic Memory áp dụng kiến trúc lai giữa graph database Neo4j và vector database Qdrant, thiết kế này giúp hệ thống vừa có thể thực hiện truy xuất ngữ nghĩa nhanh chóng, vừa có thể tận dụng knowledge graph (đồ thị tri thức) để thực hiện suy luận quan hệ phức tạp.

````python
class SemanticMemory(BaseMemory):
    """语义记忆实现
    
    特点：
    - 使用HuggingFace中文预训练模型进行文本嵌入
    - 向量检索进行快速相似度匹配
    - 知识图谱存储实体和关系
    - 混合检索策略：向量+图+语义推理
    """
    
    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)
        
        # 嵌入模型（统一提供）
        self.embedding_model = get_text_embedder()
        
        # 专业数据库存储
        self.vector_store = QdrantConnectionManager.get_instance(**qdrant_config)
        self.graph_store = Neo4jGraphStore(**neo4j_config)
        
        # 实体和关系缓存
        self.entities: Dict[str, Entity] = {}
        self.relations: List[Relation] = []
        
        # NLP处理器（支持中英文）
        self.nlp = self._init_nlp()
````
Quá trình thêm (add) của Semantic Memory thể hiện đầy đủ quy trình xây dựng knowledge graph. Hệ thống không chỉ lưu trữ nội dung Memory, mà còn tự động trích xuất thực thể (entity) và quan hệ (relation), xây dựng nên biểu diễn tri thức có cấu trúc:

```python
def add(self, memory_item: MemoryItem) -> str:
    """添加语义记忆"""
    # 1. 生成文本嵌入
    embedding = self.embedding_model.encode(memory_item.content)
    
    # 2. 提取实体和关系
    entities = self._extract_entities(memory_item.content)
    relations = self._extract_relations(memory_item.content, entities)
    
    # 3. 存储到Neo4j图数据库
    for entity in entities:
        self._add_entity_to_graph(entity, memory_item)
    
    for relation in relations:
        self._add_relation_to_graph(relation, memory_item)
    
    # 4. 存储到Qdrant向量数据库
    metadata = {
        "memory_id": memory_item.id,
        "entities": [e.entity_id for e in entities],
        "entity_count": len(entities),
        "relation_count": len(relations)
    }
    
    self.vector_store.add_vectors(
        vectors=[embedding.tolist()],
        metadata=[metadata],
        ids=[memory_item.id]
    )
```

Việc truy xuất của Semantic Memory triển khai chiến lược tìm kiếm lai (hybrid search), kết hợp năng lực hiểu ngữ nghĩa của truy xuất vector và năng lực suy luận quan hệ của truy xuất đồ thị (graph retrieval):

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """检索语义记忆"""
    # 1. 向量检索
    vector_results = self._vector_search(query, limit * 2, user_id)
    
    # 2. 图检索
    graph_results = self._graph_search(query, limit * 2, user_id)
    
    # 3. 混合排序
    combined_results = self._combine_and_rank_results(
        vector_results, graph_results, query, limit
    )
    
    return combined_results[:limit]
```

Thuật toán sắp xếp lai áp dụng cơ chế tính điểm đa yếu tố:

```python
def _combine_and_rank_results(self, vector_results, graph_results, query, limit):
    """混合排序结果"""
    combined = {}
    
    # 合并向量和图检索结果
    for result in vector_results:
        combined[result["memory_id"]] = {
            **result,
            "vector_score": result.get("score", 0.0),
            "graph_score": 0.0
        }
    
    for result in graph_results:
        memory_id = result["memory_id"]
        if memory_id in combined:
            combined[memory_id]["graph_score"] = result.get("similarity", 0.0)
        else:
            combined[memory_id] = {
                **result,
                "vector_score": 0.0,
                "graph_score": result.get("similarity", 0.0)
            }
    
    # 计算混合分数
    for memory_id, result in combined.items():
        vector_score = result["vector_score"]
        graph_score = result["graph_score"]
        importance = result.get("importance", 0.5)
        
        # 基础相似度得分
        base_relevance = vector_score * 0.7 + graph_score * 0.3
        
        # 重要性权重 [0.8, 1.2]
        importance_weight = 0.8 + (importance * 0.4)
        
        # 最终得分：相似度 * 重要性权重
        combined_score = base_relevance * importance_weight
        result["combined_score"] = combined_score
    
    # 排序并返回
    sorted_results = sorted(
        combined.values(),
        key=lambda x: x["combined_score"],
        reverse=True
    )
    
    return sorted_results[:limit]
```

Công thức tính điểm của Semantic Memory là: `(độ tương đồng vector × 0.7 + độ tương đồng đồ thị × 0.3) × (0.8 + mức độ quan trọng × 0.4)`. Ý tưởng cốt lõi của thiết kế này là:

- <strong>Trọng số truy xuất vector (0.7)</strong>: Độ tương đồng ngữ nghĩa là yếu tố chính, đảm bảo kết quả truy xuất liên quan về mặt ngữ nghĩa với truy vấn
- <strong>Trọng số truy xuất đồ thị (0.3)</strong>: Suy luận quan hệ đóng vai trò bổ sung, phát hiện mối liên kết ngầm giữa các khái niệm
- <strong>Phạm vi trọng số quan trọng [0.8, 1.2]</strong>: Tránh mức độ quan trọng ảnh hưởng quá mức đến thứ tự sắp xếp độ tương đồng, duy trì độ chính xác của truy xuất

(4) Perceptual Memory (PerceptualMemory)

Perceptual Memory hỗ trợ lưu trữ và truy xuất dữ liệu đa phương thức như văn bản, hình ảnh, âm thanh. Nó áp dụng chiến lược lưu trữ tách biệt theo phương thức (modality separation), tạo ra các vector collection độc lập cho dữ liệu của từng phương thức khác nhau, thiết kế này tránh được vấn đề không khớp chiều (dimension mismatch), đồng thời đảm bảo độ chính xác của việc truy xuất:

````python
class PerceptualMemory(BaseMemory):
    """感知记忆实现
    
    特点：
    - 支持多模态数据（文本、图像、音频等）
    - 跨模态相似性搜索
    - 感知数据的语义理解
    - 支持内容生成和检索
    """
    
    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)
        
        # 多模态编码器
        self.text_embedder = get_text_embedder()
        self._clip_model = self._init_clip_model()  # 图像编码
        self._clap_model = self._init_clap_model()  # 音频编码
        
        # 按模态分离的向量存储
        self.vector_stores = {
            "text": QdrantConnectionManager.get_instance(
                collection_name="perceptual_text",
                vector_size=self.vector_dim
            ),
            "image": QdrantConnectionManager.get_instance(
                collection_name="perceptual_image", 
                vector_size=self._image_dim
            ),
            "audio": QdrantConnectionManager.get_instance(
                collection_name="perceptual_audio",
                vector_size=self._audio_dim
            )
        }
````
Việc truy xuất của Perceptual Memory hỗ trợ cả hai chế độ cùng phương thức (same-modality) và xuyên phương thức (cross-modal). Truy xuất cùng phương thức tận dụng bộ mã hóa (encoder) chuyên biệt để khớp chính xác, trong khi truy xuất xuyên phương thức cần đến cơ chế căn chỉnh ngữ nghĩa (semantic alignment) phức tạp hơn:

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """检索感知记忆（可筛模态；同模态向量检索+时间/重要性融合）"""
    user_id = kwargs.get("user_id")
    target_modality = kwargs.get("target_modality")
    query_modality = kwargs.get("query_modality", target_modality or "text")
    
    # 同模态向量检索
    try:
        query_vector = self._encode_data(query, query_modality)
        store = self._get_vector_store_for_modality(target_modality or query_modality)
        
        where = {"memory_type": "perceptual"}
        if user_id:
            where["user_id"] = user_id
        if target_modality:
            where["modality"] = target_modality
        
        hits = store.search_similar(
            query_vector=query_vector,
            limit=max(limit * 5, 20),
            where=where
        )
    except Exception:
        hits = []
    
    # 融合排序（向量相似度 + 时间近因性 + 重要性权重）
    results = []
    for hit in hits:
        vector_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency_score(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)
        
        # 评分算法
        base_relevance = vector_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)
        combined_score = base_relevance * importance_weight
        
        results.append((combined_score, self._create_memory_item(hit)))
    
    results.sort(key=lambda x: x[0], reverse=True)
    return [item for _, item in results[:limit]]
```

Công thức tính điểm của Perceptual Memory là: `(độ tương đồng vector × 0.8 + tính gần đây về thời gian × 0.2) × (0.8 + mức độ quan trọng × 0.4)`. Cơ chế tính điểm của Perceptual Memory còn hỗ trợ truy xuất xuyên phương thức, thông qua không gian vector thống nhất để thực hiện căn chỉnh ngữ nghĩa giữa dữ liệu các phương thức khác nhau như văn bản, hình ảnh, âm thanh. Khi thực hiện truy xuất xuyên phương thức, hệ thống sẽ tự động điều chỉnh trọng số tính điểm, đảm bảo tính đa dạng và độ chính xác của kết quả truy xuất. Ngoài ra, việc tính toán tính gần đây theo thời gian trong Perceptual Memory áp dụng mô hình suy giảm theo hàm mũ (exponential decay):

```python
def _calculate_recency_score(self, timestamp: str) -> float:
    """计算时间近因性得分"""
    try:
        memory_time = datetime.fromisoformat(timestamp)
        current_time = datetime.now()
        age_hours = (current_time - memory_time).total_seconds() / 3600
        
        # 指数衰减：24小时内保持高分，之后逐渐衰减
        decay_factor = 0.1  # 衰减系数
        recency_score = math.exp(-decay_factor * age_hours / 24)
        
        return max(0.1, recency_score)  # 最低保持0.1的基础分数
    except Exception:
        return 0.5  # 默认中等分数
```

Mô hình suy giảm theo thời gian này mô phỏng đường cong quên lãng (forgetting curve) trong Memory con người, đảm bảo rằng hệ thống Perceptual Memory có thể ưu tiên truy xuất những nội dung Memory có liên quan hơn về mặt thời gian.

## 8.3 Hệ thống RAG: Tăng cường truy xuất tri thức

### 8.3.1 Kiến thức nền tảng về RAG

Trước khi đi sâu vào việc triển khai hệ thống RAG của HelloAgents, hãy cùng tìm hiểu các khái niệm nền tảng, lịch sử phát triển và nguyên lý cốt lõi của công nghệ RAG. Do nội dung bài viết này không được tạo ra dựa trên nền tảng RAG, vì vậy ở đây chỉ giúp người đọc nhanh chóng hệ thống hóa các khái niệm liên quan, để hiểu rõ hơn về những lựa chọn kỹ thuật và điểm sáng tạo trong thiết kế hệ thống.

(1) RAG là gì?

Retrieval-Augmented Generation (RAG) là một công nghệ kết hợp giữa information retrieval (truy xuất thông tin) và text generation (sinh văn bản). Ý tưởng cốt lõi của nó là: trước khi sinh ra câu trả lời, trước tiên truy xuất thông tin liên quan từ một knowledge base bên ngoài, sau đó cung cấp thông tin truy xuất được làm ngữ cảnh cho LLM, từ đó sinh ra câu trả lời chính xác và đáng tin cậy hơn.

Do đó, Retrieval-Augmented Generation có thể được tách thành ba từ. <strong>Retrieval (Truy xuất)</strong> nghĩa là truy vấn nội dung liên quan từ knowledge base; <strong>Augmented (Tăng cường)</strong> là đưa kết quả truy xuất vào Prompt, hỗ trợ mô hình sinh ra câu trả lời; <strong>Generation (Sinh)</strong> thì xuất ra câu trả lời vừa chính xác vừa minh bạch.

(2) Luồng hoạt động cơ bản

Một luồng ứng dụng RAG hoàn chỉnh chủ yếu được chia thành hai khâu cốt lõi. Ở <strong>giai đoạn chuẩn bị dữ liệu</strong>, hệ thống thông qua <strong>trích xuất dữ liệu (data extraction)</strong>, <strong>phân tách văn bản (text splitting)</strong> và <strong>vector hóa (vectorization)</strong>, xây dựng tri thức bên ngoài thành một database có thể truy xuất được. Sau đó ở <strong>giai đoạn ứng dụng</strong>, hệ thống sẽ phản hồi lại <strong>câu hỏi</strong> của người dùng, <strong>truy xuất (retrieval)</strong> thông tin liên quan từ database, <strong>đưa (inject)</strong> vào Prompt, và cuối cùng thúc đẩy LLM <strong>sinh ra câu trả lời (generation)</strong>.

(3) Lịch sử phát triển

Giai đoạn một: Naive RAG (RAG sơ khai, 2020-2021). Đây là giai đoạn chớm nở của công nghệ RAG, luồng xử lý của nó trực tiếp và đơn giản, thường được gọi là mô hình "Retrieve-Read" (Truy xuất - Đọc). <strong>Cách truy xuất</strong>: Chủ yếu dựa vào các thuật toán khớp từ khóa (keyword matching) truyền thống, như `TF-IDF` hoặc `BM25`. Những phương pháp này tính toán tần suất từ (term frequency) và tần suất tài liệu (document frequency) để đánh giá độ liên quan, hiệu quả tốt với khớp chữ (literal matching), nhưng khó hiểu được độ tương đồng về ngữ nghĩa. <strong>Mô hình sinh</strong>: Nội dung tài liệu truy xuất được sẽ được nối trực tiếp vào ngữ cảnh của Prompt mà không qua xử lý gì, sau đó đưa cho mô hình sinh.

Giai đoạn hai: Advanced RAG (RAG nâng cao, 2022-2023). Cùng với sự trưởng thành của vector database và công nghệ text embedding, RAG bước vào giai đoạn phát triển nhanh. Các nhà nghiên cứu và nhà phát triển đã đưa vào một lượng lớn kỹ thuật tối ưu ở từng khâu "truy xuất" và "sinh". <strong>Cách truy xuất</strong>: Chuyển sang truy xuất ngữ nghĩa dựa trên <strong>Dense Embedding</strong>. Thông qua việc chuyển đổi văn bản thành vector đa chiều, mô hình có thể hiểu và khớp được độ tương đồng về ngữ nghĩa, chứ không chỉ là từ khóa. <strong>Mô hình sinh</strong>: Đưa vào nhiều kỹ thuật tối ưu, ví dụ như query rewriting (viết lại truy vấn), document chunking (phân khối tài liệu), re-ranking (xếp hạng lại) v.v.

Giai đoạn ba: Modular RAG (RAG dạng module, 2023-đến nay). Dựa trên nền tảng Advanced RAG, hệ thống RAG hiện đại tiếp tục phát triển theo hướng module hóa, tự động hóa và thông minh hóa. Từng bộ phận của hệ thống được thiết kế thành các module độc lập, có thể cắm vào/tháo ra (pluggable), có thể tổ hợp (composable), nhằm thích ứng với các kịch bản ứng dụng đa dạng và phức tạp hơn. <strong>Cách truy xuất</strong>: Như hybrid retrieval (truy xuất lai), multi-query expansion (mở rộng đa truy vấn), hypothetical document embedding v.v. <strong>Mô hình sinh</strong>: Chain-of-Thought reasoning (suy luận chuỗi tư duy), tự phản tư (self-reflection) và tự sửa lỗi v.v.



### 8.3.2 Nguyên lý hoạt động của hệ thống RAG

Trước khi đi sâu vào chi tiết triển khai, có thể thông qua sơ đồ luồng để hệ thống hóa toàn bộ luồng hoạt động của hệ thống RAG trong Helloagents:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-5.png" alt="RAG系统核心原理" width="85%"/>
  <p>Hình 8.5 Nguyên lý hoạt động cốt lõi của hệ thống RAG</p>
</div>

Như minh họa trong Hình 8.5, thể hiện hai chế độ hoạt động chính của hệ thống RAG:
1. <strong>Luồng xử lý dữ liệu</strong>: Xử lý và lưu trữ tài liệu tri thức, ở đây chúng ta sử dụng công cụ `Markitdown`, ý tưởng thiết kế là chuyển đổi thống nhất mọi nguồn tri thức bên ngoài được truyền vào thành định dạng Markdown để xử lý.
2. <strong>Luồng truy vấn và sinh</strong>: Truy xuất thông tin liên quan dựa trên truy vấn và sinh ra câu trả lời.

### 8.3.3 Trải nghiệm nhanh: 30 giây làm quen với chức năng RAG

Hãy cùng trải nghiệm nhanh các chức năng cơ bản của hệ thống RAG:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import RAGTool

# 创建LLM实例
llm = HelloAgentsLLM()

# 创建RAG工具
rag_tool = RAGTool(
    knowledge_base_path="./knowledge_base",
    collection_name="test_collection",
    rag_namespace="test"
)

tool_registry = ToolRegistry()
tool_registry.register_tool(rag_tool)

# 创建具有RAG能力的Agent
agent = SimpleAgent(name="知识助手", llm=llm, tool_registry=tool_registry)

# 体验RAG功能
# 添加第一个知识
result1 = rag_tool.execute("add_text", 
    text="Python是一种高级编程语言，由Guido van Rossum于1991年首次发布。Python的设计哲学强调代码的可读性和简洁的语法。",
    document_id="python_intro")
print(f"知识1: {result1}")

# 添加第二个知识  
result2 = rag_tool.execute("add_text",
    text="机器学习是人工智能的一个分支，通过算法让计算机从数据中学习模式。主要包括监督学习、无监督学习和强化学习三种类型。",
    document_id="ml_basics")
print(f"知识2: {result2}")

# 添加第三个知识
result3 = rag_tool.execute("add_text",
    text="RAG（检索增强生成）是一种结合信息检索和文本生成的AI技术。它通过检索相关知识来增强大语言模型的生成能力。",
    document_id="rag_concept")
print(f"知识3: {result3}")


print("\n=== 搜索知识 ===")
result = rag_tool.execute("search",
    query="Python编程语言的历史",
    limit=3,
    min_score=0.1
)
print(result)

print("\n=== 知识库统计 ===")
result = rag_tool.execute("stats")
print(result)
```

Tiếp theo, chúng ta sẽ đi sâu khám phá cách triển khai cụ thể của hệ thống RAG trong HelloAgents.

### 8.3.4 Thiết kế kiến trúc hệ thống RAG

Trong phần này, chúng ta áp dụng cách giải thích khác với hệ thống Memory. Bởi vì `Memory_tool` là một triển khai mang tính hệ thống, còn RAG trong thiết kế của chúng ta được định nghĩa là một Tool, có thể được hệ thống hóa thành một pipeline. Kiến trúc cốt lõi của hệ thống RAG có thể được khái quát thành mô hình thiết kế "năm tầng bảy bước":

```
用户层：RAGTool统一接口
  ↓
应用层：智能问答、搜索、管理
  ↓  
处理层：文档解析、分块、向量化
  ↓
存储层：向量数据库、文档存储
  ↓
基础层：嵌入模型、LLM、数据库
```

Ưu điểm của thiết kế phân tầng này nằm ở chỗ mỗi tầng đều có thể được tối ưu và thay thế một cách độc lập, đồng thời vẫn duy trì được sự ổn định của toàn bộ hệ thống. Ví dụ, có thể dễ dàng chuyển embedding model từ sentence-transformers sang Bách Luyện (Bailian) API, mà không ảnh hưởng đến logic nghiệp vụ ở tầng trên. Tương tự, code của những quy trình xử lý này có thể tái sử dụng hoàn toàn, cũng có thể chọn lấy phần mình cần để đưa vào dự án của riêng mình. RAGTool với vai trò điểm vào thống nhất của hệ thống RAG, cung cấp một giao diện API đơn giản.

````python
class RAGTool(Tool):
    """RAG工具
    
    提供完整的 RAG 能力：
    - 添加多格式文档（PDF、Office、图片、音频等）
    - 智能检索与召回
    - LLM 增强问答
    - 知识库管理
    """
    
    def __init__(
        self,
        knowledge_base_path: str = "./knowledge_base",
        qdrant_url: str = None,
        qdrant_api_key: str = None,
        collection_name: str = "rag_knowledge_base",
        rag_namespace: str = "default"
    ):
        # 初始化RAG管道
        self._pipelines: Dict[str, Dict[str, Any]] = {}
        self.llm = HelloAgentsLLM()
        
        # 创建默认管道
        default_pipeline = create_rag_pipeline(
            qdrant_url=self.qdrant_url,
            qdrant_api_key=self.qdrant_api_key,
            collection_name=self.collection_name,
            rag_namespace=self.rag_namespace
        )
        self._pipelines[self.rag_namespace] = default_pipeline
````
Toàn bộ luồng xử lý được thể hiện như sau:
```
任意格式文档 → MarkItDown转换 → Markdown文本 → 智能分块 → 向量化 → 存储检索
```

(1) Nạp tài liệu đa phương thức

Một trong những ưu thế cốt lõi của hệ thống RAG là năng lực xử lý tài liệu đa phương thức mạnh mẽ của nó. Hệ thống sử dụng MarkItDown làm công cụ chuyển đổi tài liệu thống nhất, hỗ trợ hầu hết các định dạng tài liệu phổ biến. MarkItDown là công cụ chuyển đổi tài liệu tổng quát mã nguồn mở của Microsoft, nó là component cốt lõi của hệ thống RAG trong HelloAgents, chịu trách nhiệm chuyển đổi tài liệu ở bất kỳ định dạng nào thành văn bản Markdown có cấu trúc. Bất kể đầu vào là PDF, Word, Excel, hình ảnh hay âm thanh, cuối cùng đều sẽ được chuyển đổi thành định dạng Markdown chuẩn, sau đó đi vào luồng phân khối (chunking), vector hóa và lưu trữ thống nhất.

```python
def _convert_to_markdown(path: str) -> str:
    """
    Universal document reader using MarkItDown with enhanced PDF processing.
    核心功能：将任意格式文档转换为Markdown文本
    
    支持格式：
    - 文档：PDF、Word、Excel、PowerPoint
    - 图像：JPG、PNG、GIF（通过OCR）
    - 音频：MP3、WAV、M4A（通过转录）
    - 文本：TXT、CSV、JSON、XML、HTML
    - 代码：Python、JavaScript、Java等
    """
    if not os.path.exists(path):
        return ""
    
    # 对PDF文件使用增强处理
    ext = (os.path.splitext(path)[1] or '').lower()
    if ext == '.pdf':
        return _enhanced_pdf_processing(path)
    
    # 其他格式使用MarkItDown统一转换
    md_instance = _get_markitdown_instance()
    if md_instance is None:
        return _fallback_text_reader(path)
    
    try:
        result = md_instance.convert(path)
        markdown_text = getattr(result, "text_content", None)
        if isinstance(markdown_text, str) and markdown_text.strip():
            print(f"[RAG] MarkItDown转换成功: {path} -> {len(markdown_text)} chars Markdown")
            return markdown_text
        return ""
    except Exception as e:
        print(f"[WARNING] MarkItDown转换失败 {path}: {e}")
        return _fallback_text_reader(path)
```

(2) Chiến lược phân khối thông minh

Sau khi chuyển đổi qua MarkItDown, tất cả tài liệu đều được thống nhất thành định dạng Markdown chuẩn. Điều này cung cấp nền tảng có cấu trúc cho việc phân khối (chunking) thông minh sau đó. HelloAgents đã triển khai một chiến lược phân khối thông minh chuyên biệt cho định dạng Markdown, tận dụng tối đa đặc tính có cấu trúc của Markdown để phân tách chính xác.

Luồng phân khối nhận biết cấu trúc Markdown:

```
标准Markdown文本 → 标题层次解析 → 段落语义分割 → Token计算分块 → 重叠策略优化 → 向量化准备
       ↓                ↓              ↓            ↓           ↓            ↓
   统一格式          #/##/###        语义边界      大小控制     信息连续性    嵌入向量
   结构清晰          层次识别        完整性保证    检索优化     上下文保持    相似度匹配
```

Do tất cả tài liệu đã được chuyển đổi thành định dạng Markdown, hệ thống có thể tận dụng cấu trúc tiêu đề (heading) của Markdown (#, ##, ### v.v.) để thực hiện phân tách ngữ nghĩa chính xác:

```python
def _split_paragraphs_with_headings(text: str) -> List[Dict]:
    """根据标题层次分割段落，保持语义完整性"""
    lines = text.splitlines()
    heading_stack: List[str] = []
    paragraphs: List[Dict] = []
    buf: List[str] = []
    char_pos = 0
    
    def flush_buf(end_pos: int):
        if not buf:
            return
        content = "\n".join(buf).strip()
        if not content:
            return
        paragraphs.append({
            "content": content,
            "heading_path": " > ".join(heading_stack) if heading_stack else None,
            "start": max(0, end_pos - len(content)),
            "end": end_pos,
        })
    
    for ln in lines:
        raw = ln
        if raw.strip().startswith("#"):
            # 处理标题行
            flush_buf(char_pos)
            level = len(raw) - len(raw.lstrip('#'))
            title = raw.lstrip('#').strip()
            
            if level <= 0:
                level = 1
            if level <= len(heading_stack):
                heading_stack = heading_stack[:level-1]
            heading_stack.append(title)
            
            char_pos += len(raw) + 1
            continue
        
        # 段落内容累积
        if raw.strip() == "":
            flush_buf(char_pos)
            buf = []
        else:
            buf.append(raw)
        char_pos += len(raw) + 1
    
    flush_buf(char_pos)
    
    if not paragraphs:
        paragraphs = [{"content": text, "heading_path": None, "start": 0, "end": len(text)}]
    
    return paragraphs
```

Trên nền tảng phân tách đoạn văn (paragraph) theo Markdown, hệ thống tiếp tục thực hiện phân khối thông minh dựa trên số lượng Token. Do đầu vào đã là văn bản Markdown có cấu trúc, hệ thống có thể kiểm soát biên giới phân khối chính xác hơn, đảm bảo mỗi khối (chunk) vừa phù hợp để xử lý vector hóa, vừa giữ được tính toàn vẹn của cấu trúc Markdown:

```python
def _chunk_paragraphs(paragraphs: List[Dict], chunk_tokens: int, overlap_tokens: int) -> List[Dict]:
    """基于Token数量的智能分块"""
    chunks: List[Dict] = []
    cur: List[Dict] = []
    cur_tokens = 0
    i = 0
    
    while i < len(paragraphs):
        p = paragraphs[i]
        p_tokens = _approx_token_len(p["content"]) or 1
        
        if cur_tokens + p_tokens <= chunk_tokens or not cur:
            cur.append(p)
            cur_tokens += p_tokens
            i += 1
        else:
            # 生成当前分块
            content = "\n\n".join(x["content"] for x in cur)
            start = cur[0]["start"]
            end = cur[-1]["end"]
            heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)
            
            chunks.append({
                "content": content,
                "start": start,
                "end": end,
                "heading_path": heading_path,
            })
            
            # 构建重叠部分
            if overlap_tokens > 0 and cur:
                kept: List[Dict] = []
                kept_tokens = 0
                for x in reversed(cur):
                    t = _approx_token_len(x["content"]) or 1
                    if kept_tokens + t > overlap_tokens:
                        break
                    kept.append(x)
                    kept_tokens += t
                cur = list(reversed(kept))
                cur_tokens = kept_tokens
            else:
                cur = []
                cur_tokens = 0
    
    # 处理最后一个分块
    if cur:
        content = "\n\n".join(x["content"] for x in cur)
        start = cur[0]["start"]
        end = cur[-1]["end"]
        heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)
        
        chunks.append({
            "content": content,
            "start": start,
            "end": end,
            "heading_path": heading_path,
        })
    
    return chunks
```

Đồng thời để tương thích với nhiều ngôn ngữ khác nhau, hệ thống đã triển khai thuật toán ước lượng Token dành riêng cho văn bản hỗn hợp Trung-Anh, điều này rất quan trọng đối với việc kiểm soát chính xác kích thước phân khối:

```python
def _approx_token_len(text: str) -> int:
    """近似估计Token长度，支持中英文混合"""
    # CJK字符按1 token计算
    cjk = sum(1 for ch in text if _is_cjk(ch))
    # 其他字符按空白分词计算
    non_cjk_tokens = len([t for t in text.split() if t])
    return cjk + non_cjk_tokens

def _is_cjk(ch: str) -> bool:
    """判断是否为CJK字符"""
    code = ord(ch)
    return (
        0x4E00 <= code <= 0x9FFF or  # CJK统一汉字
        0x3400 <= code <= 0x4DBF or  # CJK扩展A
        0x20000 <= code <= 0x2A6DF or # CJK扩展B
        0x2A700 <= code <= 0x2B73F or # CJK扩展C
        0x2B740 <= code <= 0x2B81F or # CJK扩展D
        0x2B820 <= code <= 0x2CEAF or # CJK扩展E
        0xF900 <= code <= 0xFAFF      # CJK兼容汉字
    )
```

(3) Embedding thống nhất và lưu trữ vector

Embedding model là cốt lõi của hệ thống RAG, nó chịu trách nhiệm chuyển đổi văn bản thành vector đa chiều, giúp máy tính có thể hiểu và so sánh được độ tương đồng về ngữ nghĩa (semantic similarity) của văn bản. Năng lực truy xuất của hệ thống RAG phần lớn phụ thuộc vào chất lượng của embedding model và hiệu quả của việc lưu trữ vector. HelloAgents đã triển khai một giao diện embedding thống nhất. Ở đây để minh họa, sử dụng Bách Luyện (Bailian) API, nếu chưa cấu hình có thể chuyển sang mô hình local `all-MiniLM-L6-v2`, nếu cả hai phương án đều không khả dụng, cũng đã cấu hình sẵn thuật toán TF-IDF để làm phương án dự phòng (fallback). Khi sử dụng thực tế có thể thay thế bằng mô hình hoặc API mà bạn muốn, cũng có thể thử mở rộng nội dung framework~

```python
def index_chunks(
    store = None, 
    chunks: List[Dict] = None, 
    cache_db: Optional[str] = None, 
    batch_size: int = 64,
    rag_namespace: str = "default"
) -> None:
    """
    Index markdown chunks with unified embedding and Qdrant storage.
    Uses百炼 API with fallback to sentence-transformers.
    """
    if not chunks:
        print("[RAG] No chunks to index")
        return
    
    # 使用统一嵌入模型
    embedder = get_text_embedder()
    dimension = get_dimension(384)
    
    # 创建默认Qdrant存储
    if store is None:
        store = _create_default_vector_store(dimension)
        print(f"[RAG] Created default Qdrant store with dimension {dimension}")
    
    # 预处理Markdown文本以获得更好的嵌入质量
    processed_texts = []
    for c in chunks:
        raw_content = c["content"]
        processed_content = _preprocess_markdown_for_embedding(raw_content)
        processed_texts.append(processed_content)
    
    print(f"[RAG] Embedding start: total_texts={len(processed_texts)} batch_size={batch_size}")
    
    # 批量编码
    vecs: List[List[float]] = []
    for i in range(0, len(processed_texts), batch_size):
        part = processed_texts[i:i+batch_size]
        try:
            # 使用统一嵌入器（内部处理缓存）
            part_vecs = embedder.encode(part)
            
            # 标准化为List[List[float]]格式
            if not isinstance(part_vecs, list):
                if hasattr(part_vecs, "tolist"):
                    part_vecs = [part_vecs.tolist()]
                else:
                    part_vecs = [list(part_vecs)]
            
            # 处理向量格式和维度
            for v in part_vecs:
                try:
                    if hasattr(v, "tolist"):
                        v = v.tolist()
                    v_norm = [float(x) for x in v]
                    
                    # 维度检查和调整
                    if len(v_norm) != dimension:
                        print(f"[WARNING] 向量维度异常: 期望{dimension}, 实际{len(v_norm)}")
                        if len(v_norm) < dimension:
                            v_norm.extend([0.0] * (dimension - len(v_norm)))
                        else:
                            v_norm = v_norm[:dimension]
                    
                    vecs.append(v_norm)
                except Exception as e:
                    print(f"[WARNING] 向量转换失败: {e}, 使用零向量")
                    vecs.append([0.0] * dimension)
                    
        except Exception as e:
            print(f"[WARNING] Batch {i} encoding failed: {e}")
            # 实现重试机制
            # ... 重试逻辑 ...
        
        print(f"[RAG] Embedding progress: {min(i+batch_size, len(processed_texts))}/{len(processed_texts)}")
```

### 8.3.5 Chiến lược truy xuất nâng cao

Năng lực truy xuất của hệ thống RAG là sức cạnh tranh cốt lõi của nó. Trong ứng dụng thực tế, cách diễn đạt truy vấn của người dùng và nội dung thực tế trong tài liệu có thể tồn tại sự khác biệt về dùng từ, dẫn đến các tài liệu liên quan không thể được truy xuất. Để giải quyết vấn đề này, HelloAgents đã triển khai ba chiến lược truy xuất nâng cao bổ trợ lẫn nhau: Multi-Query Expansion (MQE), Hypothetical Document Embeddings (HyDE) và framework truy xuất mở rộng thống nhất.

(1) Multi-Query Expansion (MQE)

Multi-Query Expansion (mở rộng đa truy vấn) là một kỹ thuật nhằm tăng recall rate (tỷ lệ gọi lại) của việc truy xuất, thông qua việc sinh ra nhiều truy vấn đa dạng nhưng tương đương về ngữ nghĩa. Nhận định cốt lõi của phương pháp này là: cùng một câu hỏi có thể có nhiều cách diễn đạt khác nhau, và những cách diễn đạt khác nhau có thể khớp với những tài liệu liên quan khác nhau. Ví dụ, "cách học Python" có thể được mở rộng thành "hướng dẫn nhập môn Python", "phương pháp học Python", "cẩm nang lập trình Python" và nhiều truy vấn khác. Thông qua việc thực thi song song các truy vấn mở rộng này rồi hợp nhất kết quả, hệ thống có thể bao phủ phạm vi tài liệu liên quan rộng hơn, tránh bỏ sót thông tin quan trọng do khác biệt về dùng từ.

Ưu điểm của MQE nằm ở chỗ nó có thể tự động hiểu nhiều ý nghĩa khả dĩ của truy vấn người dùng, đặc biệt hiệu quả rõ rệt đối với các truy vấn mơ hồ hoặc truy vấn thuật ngữ chuyên ngành. Hệ thống sử dụng LLM để sinh ra các truy vấn mở rộng, đảm bảo tính đa dạng và liên quan về ngữ nghĩa của việc mở rộng:

```python
def _prompt_mqe(query: str, n: int) -> List[str]:
    """使用LLM生成多样化的查询扩展"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "你是检索查询扩展助手。生成语义等价或互补的多样化查询。使用中文，简短，避免标点。"},
            {"role": "user", "content": f"原始查询：{query}\n请给出{n}个不同表述的查询，每行一个。"}
        ]
        text = llm.invoke(prompt)
        lines = [ln.strip("- \t") for ln in (text or "").splitlines()]
        outs = [ln for ln in lines if ln]
        return outs[:n] or [query]
    except Exception:
        return [query]
```

(2) Hypothetical Document Embeddings (HyDE)

Hypothetical Document Embeddings (nhúng tài liệu giả định, HyDE) là một kỹ thuật truy xuất sáng tạo, ý tưởng cốt lõi của nó là "dùng câu trả lời để tìm câu trả lời". Phương pháp truy xuất truyền thống là dùng câu hỏi để khớp với tài liệu, nhưng câu hỏi và câu trả lời trong không gian ngữ nghĩa thường tồn tại sự khác biệt về phân bố — câu hỏi thường là câu nghi vấn, còn nội dung tài liệu là câu trần thuật. HyDE thông qua việc để LLM trước tiên sinh ra một đoạn văn trả lời giả định, sau đó dùng đoạn văn trả lời này để truy xuất tài liệu thực sự, từ đó thu hẹp khoảng cách ngữ nghĩa (semantic gap) giữa truy vấn và tài liệu.

Ưu điểm của phương pháp này nằm ở chỗ, câu trả lời giả định gần với câu trả lời thực sự hơn trong không gian ngữ nghĩa, do đó có thể khớp chính xác hơn với các tài liệu liên quan. Ngay cả khi nội dung của câu trả lời giả định không hoàn toàn chính xác, các thuật ngữ then chốt, khái niệm và phong cách diễn đạt mà nó chứa đựng cũng có thể hiệu quả dẫn dắt hệ thống truy xuất tìm đến đúng tài liệu. Đặc biệt đối với các truy vấn trong lĩnh vực chuyên ngành, HyDE có thể sinh ra tài liệu giả định chứa thuật ngữ chuyên ngành, nâng cao đáng kể độ chính xác của truy xuất:

```python
def _prompt_hyde(query: str) -> Optional[str]:
    """生成假设性文档用于改善检索"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "根据用户问题，先写一段可能的答案性段落，用于向量检索的查询文档（不要分析过程）。"},
            {"role": "user", "content": f"问题：{query}\n请直接写一段中等长度、客观、包含关键术语的段落。"}
        ]
        return llm.invoke(prompt)
    except Exception:
        return None
```

(3) Framework truy xuất mở rộng

HelloAgents tích hợp hai chiến lược MQE và HyDE vào một framework truy xuất mở rộng thống nhất. Hệ thống thông qua các tham số `enable_mqe` và `enable_hyde` để người dùng có thể chọn kích hoạt chiến lược nào dựa trên kịch bản cụ thể: đối với kịch bản cần recall rate cao có thể kích hoạt đồng thời cả hai chiến lược, đối với kịch bản nhạy cảm về hiệu năng có thể chỉ sử dụng truy xuất cơ bản.

Cơ chế cốt lõi của truy xuất mở rộng là quy trình ba bước "mở rộng - truy xuất - hợp nhất". Đầu tiên, hệ thống sinh ra nhiều truy vấn mở rộng dựa trên truy vấn gốc (bao gồm các truy vấn đa dạng do MQE sinh ra và tài liệu giả định do HyDE sinh ra); sau đó, thực hiện truy xuất vector song song cho mỗi truy vấn mở rộng, thu được candidate pool (tập ứng viên); cuối cùng, thông qua việc loại bỏ trùng lặp và sắp xếp theo điểm số để hợp nhất tất cả kết quả, trả về top-k tài liệu liên quan nhất. Điểm khéo léo của thiết kế này nằm ở chỗ, nó thông qua tham số `candidate_pool_multiplier` (mặc định là 4) để mở rộng candidate pool, đảm bảo có đủ tài liệu ứng viên để sàng lọc, đồng thời thông qua việc loại bỏ trùng lặp thông minh để tránh trả về nội dung lặp lại.

```python
def search_vectors_expanded(
    store = None,
    query: str = "",
    top_k: int = 8,
    rag_namespace: Optional[str] = None,
    only_rag_data: bool = True,
    score_threshold: Optional[float] = None,
    enable_mqe: bool = False,
    mqe_expansions: int = 2,
    enable_hyde: bool = False,
    candidate_pool_multiplier: int = 4,
) -> List[Dict]:
    """
    Search with query expansion using unified embedding and Qdrant.
    """
    if not query:
        return []
    
    # 创建默认存储
    if store is None:
        store = _create_default_vector_store()
    
    # 查询扩展
    expansions: List[str] = [query]
    
    if enable_mqe and mqe_expansions > 0:
        expansions.extend(_prompt_mqe(query, mqe_expansions))
    if enable_hyde:
        hyde_text = _prompt_hyde(query)
        if hyde_text:
            expansions.append(hyde_text)

    # 去重和修剪
    uniq: List[str] = []
    for e in expansions:
        if e and e not in uniq:
            uniq.append(e)
    expansions = uniq[: max(1, len(uniq))]

    # 分配候选池
    pool = max(top_k * candidate_pool_multiplier, 20)
    per = max(1, pool // max(1, len(expansions)))

    # 构建RAG数据过滤器
    where = {"memory_type": "rag_chunk"}
    if only_rag_data:
        where["is_rag_data"] = True
        where["data_source"] = "rag_pipeline"
    if rag_namespace:
        where["rag_namespace"] = rag_namespace

    # 收集所有扩展查询的结果
    agg: Dict[str, Dict] = {}
    for q in expansions:
        qv = embed_query(q)
        hits = store.search_similar(
            query_vector=qv, 
            limit=per, 
            score_threshold=score_threshold, 
            where=where
        )
        for h in hits:
            mid = h.get("metadata", {}).get("memory_id", h.get("id"))
            s = float(h.get("score", 0.0))
            if mid not in agg or s > float(agg[mid].get("score", 0.0)):
                agg[mid] = h
    
    # 按分数排序返回
    merged = list(agg.values())
    merged.sort(key=lambda x: float(x.get("score", 0.0)), reverse=True)
    return merged[:top_k]
```

Trong ứng dụng thực tế, việc kết hợp sử dụng cả ba chiến lược này mang lại hiệu quả tốt nhất. MQE giỏi xử lý vấn đề đa dạng về dùng từ, HyDE giỏi xử lý vấn đề khoảng cách ngữ nghĩa, còn framework thống nhất thì đảm bảo chất lượng và tính đa dạng của kết quả. Đối với truy vấn thông thường, khuyến nghị kích hoạt MQE; đối với truy vấn lĩnh vực chuyên ngành, khuyến nghị kích hoạt đồng thời MQE và HyDE; đối với kịch bản nhạy cảm về hiệu năng, có thể chỉ sử dụng truy xuất cơ bản hoặc chỉ kích hoạt MQE.

Tất nhiên còn rất nhiều phương pháp thú vị khác, ở đây chỉ giới thiệu mở rộng vừa phải cho người đọc, trong các kịch bản sử dụng thực tế cũng cần phải thử nghiệm để tìm ra giải pháp phù hợp với vấn đề.



## 8.4 Xây dựng Trợ lý hỏi đáp tài liệu thông minh

Trong các phần trước, chúng ta đã giới thiệu chi tiết về thiết kế và triển khai của hệ thống Memory và hệ thống RAG trong HelloAgents. Bây giờ, hãy cùng thông qua một case study thực chiến hoàn chỉnh, để minh họa cách kết hợp hữu cơ hai hệ thống này, xây dựng nên một trợ lý hỏi đáp tài liệu thông minh.

### 8.4.1 Bối cảnh và mục tiêu của case study

Trong công việc thực tế, chúng ta thường xuyên cần xử lý một lượng lớn tài liệu kỹ thuật, bài báo nghiên cứu, sổ tay sản phẩm dưới dạng PDF. Cách đọc tài liệu truyền thống có hiệu quả thấp, khó định vị nhanh thông tin then chốt, càng không thể thiết lập mối liên kết giữa các tri thức.

Case study này sẽ lấy file PDF bản thử nghiệm công khai (beta) `Happy-LLM-0727.pdf` của một tutorial khác thuộc Datawhale là Happy-LLM (dạy thực hành mô hình lớn) làm ví dụ, xây dựng một <strong>ứng dụng Web dựa trên Gradio</strong>, minh họa cách sử dụng RAGTool và MemoryTool để xây dựng một trợ lý học tập tương tác hoàn chỉnh. PDF có thể lấy tại [liên kết này](https://github.com/datawhalechina/happy-llm/releases/download/v1.0.1/Happy-LLM-0727.pdf).

Chúng ta mong muốn triển khai các chức năng sau:

1. <strong>Xử lý tài liệu thông minh</strong>: Sử dụng MarkItDown để chuyển đổi thống nhất PDF sang Markdown, chiến lược phân khối thông minh dựa trên cấu trúc Markdown, vector hóa và xây dựng index hiệu quả

2. <strong>Hỏi đáp truy xuất nâng cao</strong>: Multi-Query Expansion (MQE) nâng cao recall rate, Hypothetical Document Embedding (HyDE) cải thiện độ chính xác truy xuất, hỏi đáp thông minh nhận biết ngữ cảnh

3. <strong>Quản lý Memory đa tầng</strong>: Working Memory quản lý tác vụ học tập và ngữ cảnh hiện tại, Episodic Memory ghi lại sự kiện học tập và lịch sử truy vấn, Semantic Memory lưu trữ tri thức khái niệm và sự hiểu biết, Perceptual Memory xử lý đặc trưng tài liệu và thông tin đa phương thức

4. <strong>Hỗ trợ học tập cá nhân hóa</strong>: Đề xuất cá nhân hóa dựa trên lịch sử học tập, tích hợp Memory và quên lãng chọn lọc, sinh báo cáo học tập và theo dõi tiến độ

Để minh họa rõ ràng hơn luồng hoạt động của toàn bộ hệ thống, Hình 8.6 thể hiện mối quan hệ và luồng dữ liệu giữa năm bước. Năm bước hình thành một vòng khép kín hoàn chỉnh: bước 1 ghi lại thông tin đã xử lý của tài liệu PDF vào hệ thống Memory, bước 2 kết quả truy xuất cũng sẽ được ghi vào hệ thống Memory, bước 3 thể hiện đầy đủ chức năng của hệ thống Memory (thêm, truy xuất, tích hợp, quên lãng), bước 4 tích hợp RAG và Memory để cung cấp định tuyến thông minh (intelligent routing), bước 5 thu thập toàn bộ thông tin thống kê để sinh báo cáo học tập.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-6.png" alt="" width="85%"/>
  <p>Hình 8.6 Luồng thực thi năm bước của trợ lý hỏi đáp thông minh</p>
</div>

Tiếp theo, chúng ta sẽ minh họa cách triển khai ứng dụng Web này. Toàn bộ ứng dụng được chia thành ba phần cốt lõi:

1. <strong>Class trợ lý cốt lõi (PDFLearningAssistant)</strong>: Đóng gói logic gọi của RAGTool và MemoryTool
2. <strong>Giao diện Web Gradio</strong>: Cung cấp giao diện tương tác thân thiện với người dùng, phần này có thể tham khảo code ví dụ để học
3. <strong>Các chức năng cốt lõi khác</strong>: Ghi chú học tập, ôn lại học tập, xem thống kê và sinh báo cáo

### 8.4.2 Triển khai Class trợ lý cốt lõi

Đầu tiên, chúng ta triển khai class trợ lý cốt lõi `PDFLearningAssistant`, nó đóng gói logic gọi của RAGTool và MemoryTool.

(1) Khởi tạo class

```python
class PDFLearningAssistant:
    """智能文档问答助手"""

    def __init__(self, user_id: str = "default_user"):
        """初始化学习助手

        Args:
            user_id: 用户ID，用于隔离不同用户的数据
        """
        self.user_id = user_id
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # 初始化工具
        self.memory_tool = MemoryTool(user_id=user_id)
        self.rag_tool = RAGTool(rag_namespace=f"pdf_{user_id}")

        # 学习统计
        self.stats = {
            "session_start": datetime.now(),
            "documents_loaded": 0,
            "questions_asked": 0,
            "concepts_learned": 0
        }

        # 当前加载的文档
        self.current_document = None
```

Trong quá trình khởi tạo này, chúng ta đã đưa ra một số quyết định thiết kế then chốt:

<strong>Khởi tạo MemoryTool</strong>: Thông qua tham số `user_id` để thực hiện cách ly Memory ở cấp độ người dùng. Memory học tập của các người dùng khác nhau hoàn toàn độc lập, mỗi người dùng đều có không gian Working Memory, Episodic Memory, Semantic Memory và Perceptual Memory riêng của mình.

<strong>Khởi tạo RAGTool</strong>: Thông qua tham số `rag_namespace` để thực hiện cách ly namespace của knowledge base. Sử dụng `f"pdf_{user_id}"` làm namespace, mỗi người dùng đều có knowledge base PDF độc lập của riêng mình.

<strong>Quản lý phiên (session)</strong>: `session_id` dùng để theo dõi toàn bộ quá trình của một phiên học tập, tiện cho việc ôn lại và phân tích lịch trình học tập sau này.

<strong>Thông tin thống kê</strong>: Dictionary `stats` ghi lại các chỉ số học tập then chốt, dùng để sinh báo cáo học tập.

(2) Nạp tài liệu PDF

```python
def load_document(self, pdf_path: str) -> Dict[str, Any]:
    """加载PDF文档到知识库

    Args:
        pdf_path: PDF文件路径

    Returns:
        Dict: 包含success和message的结果
    """
    if not os.path.exists(pdf_path):
        return {"success": False, "message": f"文件不存在: {pdf_path}"}

    start_time = time.time()

    # 【RAGTool】处理PDF: MarkItDown转换 → 智能分块 → 向量化
    result = self.rag_tool.execute(
        "add_document",
        file_path=pdf_path,
        chunk_size=1000,
        chunk_overlap=200
    )

    process_time = time.time() - start_time

    if result.get("success", False):
        self.current_document = os.path.basename(pdf_path)
        self.stats["documents_loaded"] += 1

        # 【MemoryTool】记录到学习记忆
        self.memory_tool.execute(
            "add",
            content=f"加载了文档《{self.current_document}》",
            memory_type="episodic",
            importance=0.9,
            event_type="document_loaded",
            session_id=self.session_id
        )

        return {
            "success": True,
            "message": f"加载成功！(耗时: {process_time:.1f}秒)",
            "document": self.current_document
        }
    else:
        return {
            "success": False,
            "message": f"加载失败: {result.get('error', '未知错误')}"
        }
```

Chúng ta chỉ cần một dòng code để hoàn thành việc xử lý PDF:

```python
result = self.rag_tool.execute(
    "add_document",
    file_path=pdf_path,
    chunk_size=1000,
    chunk_overlap=200
)
```

Lời gọi này sẽ kích hoạt toàn bộ luồng xử lý của RAGTool (chuyển đổi MarkItDown, xử lý tăng cường, phân khối thông minh, lưu trữ vector hóa), những chi tiết bên trong này đã được giới thiệu chi tiết ở mục 8.3. Chúng ta chỉ cần quan tâm đến:

- <strong>Loại thao tác</strong>: `"add_document"` - Thêm tài liệu vào knowledge base
- <strong>Đường dẫn file</strong>: `file_path` - Đường dẫn của file PDF
- <strong>Tham số phân khối</strong>: `chunk_size=1000, chunk_overlap=200` - Kiểm soát việc phân khối văn bản
- <strong>Kết quả trả về</strong>: Dictionary chứa trạng thái xử lý và thông tin thống kê

Sau khi nạp tài liệu thành công, chúng ta sử dụng MemoryTool để ghi vào Episodic Memory:

```python
self.memory_tool.execute(
    "add",
    content=f"加载了文档《{self.current_document}》",
    memory_type="episodic",
    importance=0.9,
    event_type="document_loaded",
    session_id=self.session_id
)
```

<strong>Vì sao dùng Episodic Memory?</strong> Bởi vì đây là một sự kiện cụ thể, có timestamp, phù hợp để ghi lại bằng Episodic Memory. Tham số `session_id` liên kết sự kiện này với phiên học tập hiện tại, tiện cho việc ôn lại lịch trình học tập sau này.

Việc ghi lại Memory này đặt nền tảng cho các dịch vụ cá nhân hóa sau này:

- Người dùng hỏi "trước đây tôi đã nạp những tài liệu nào?" → Truy xuất từ Episodic Memory
- Hệ thống có thể theo dõi lịch trình học tập và tình hình sử dụng tài liệu của người dùng

### 8.4.3 Chức năng hỏi đáp thông minh

Sau khi hoàn tất việc nạp tài liệu, người dùng có thể đặt câu hỏi với tài liệu. Chúng ta triển khai một phương thức `ask` để xử lý câu hỏi của người dùng:

```python
def ask(self, question: str, use_advanced_search: bool = True) -> str:
    """向文档提问

    Args:
        question: 用户问题
        use_advanced_search: 是否使用高级检索（MQE + HyDE）

    Returns:
        str: 答案
    """
    if not self.current_document:
        return "⚠️ 请先加载文档！"

    # 【MemoryTool】记录问题到工作记忆
    self.memory_tool.execute(
        "add",
        content=f"提问: {question}",
        memory_type="working",
        importance=0.6,
        session_id=self.session_id
    )

    # 【RAGTool】使用高级检索获取答案
    answer = self.rag_tool.execute(
        "ask",
        question=question,
        limit=5,
        enable_advanced_search=use_advanced_search,
        enable_mqe=use_advanced_search,
        enable_hyde=use_advanced_search
    )

    # 【MemoryTool】记录到情景记忆
    self.memory_tool.execute(
        "add",
        content=f"关于'{question}'的学习",
        memory_type="episodic",
        importance=0.7,
        event_type="qa_interaction",
        session_id=self.session_id
    )

    self.stats["questions_asked"] += 1

    return answer
```

Khi chúng ta gọi `self.rag_tool.execute("ask", ...)`, bên trong RAGTool đã thực thi luồng truy xuất nâng cao sau:

1. <strong>Multi-Query Expansion (MQE)</strong>:

   ```python
   # 生成多样化查询
   expanded_queries = self._generate_multi_queries(question)
   # 例如，对于"什么是大语言模型？"，可能生成：
   # - "大语言模型的定义是什么？"
   # - "请解释一下大语言模型"
   # - "LLM是什么意思？"
   ```

   MQE thông qua LLM sinh ra các truy vấn tương đương về ngữ nghĩa nhưng diễn đạt khác nhau, hiểu ý định người dùng từ nhiều góc độ, nâng cao recall rate 30%-50%.

2. <strong>Hypothetical Document Embedding (HyDE)</strong>:

   - Sinh ra tài liệu trả lời giả định, bắc cầu khoảng cách ngữ nghĩa giữa truy vấn và tài liệu
   - Sử dụng vector của câu trả lời giả định để truy xuất

Cách triển khai bên trong của các kỹ thuật truy xuất nâng cao này đã được giới thiệu chi tiết ở mục 8.3.5.

### 8.4.4 Các chức năng cốt lõi khác

Ngoài việc nạp tài liệu và hỏi đáp thông minh, chúng ta còn cần triển khai các chức năng như ghi chú học tập, ôn lại học tập, xem thống kê và sinh báo cáo:

```python
def add_note(self, content: str, concept: Optional[str] = None):
    """添加学习笔记"""
    self.memory_tool.execute(
        "add",
        content=content,
        memory_type="semantic",
        importance=0.8,
        concept=concept or "general",
        session_id=self.session_id
    )
    self.stats["concepts_learned"] += 1

def recall(self, query: str, limit: int = 5) -> str:
    """回顾学习历程"""
    result = self.memory_tool.execute(
        "search",
        query=query,
        limit=limit
    )
    return result

def get_stats(self) -> Dict[str, Any]:
    """获取学习统计"""
    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    return {
        "会话时长": f"{duration:.0f}秒",
        "加载文档": self.stats["documents_loaded"],
        "提问次数": self.stats["questions_asked"],
        "学习笔记": self.stats["concepts_learned"],
        "当前文档": self.current_document or "未加载"
    }

def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
    """生成学习报告"""
    memory_summary = self.memory_tool.execute("summary", limit=10)
    rag_stats = self.rag_tool.execute("stats")

    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    report = {
        "session_info": {
            "session_id": self.session_id,
            "user_id": self.user_id,
            "start_time": self.stats["session_start"].isoformat(),
            "duration_seconds": duration
        },
        "learning_metrics": {
            "documents_loaded": self.stats["documents_loaded"],
            "questions_asked": self.stats["questions_asked"],
            "concepts_learned": self.stats["concepts_learned"]
        },
        "memory_summary": memory_summary,
        "rag_status": rag_stats
    }

    if save_to_file:
        report_file = f"learning_report_{self.session_id}.json"
        with open(report_file, 'w', encoding='utf-8') as f:
            json.dump(report, f, ensure_ascii=False, indent=2, default=str)
        report["report_file"] = report_file

    return report
```

Các phương thức này lần lượt triển khai:

- <strong>add_note</strong>: Lưu ghi chú học tập vào Semantic Memory
- <strong>recall</strong>: Truy xuất lịch trình học tập từ hệ thống Memory
- <strong>get_stats</strong>: Lấy thông tin thống kê của phiên hiện tại
- <strong>generate_report</strong>: Sinh báo cáo học tập chi tiết và lưu thành file JSON

### 8.4.5 Minh họa kết quả chạy thử

Tiếp theo là minh họa kết quả chạy thử, như Hình 8.7, sau khi vào trang chủ cần khởi tạo trợ lý trước, tức là nạp database, mô hình, các thao tác tải API của chúng ta. Sau đó truyền vào tài liệu PDF, và nhấn nạp tài liệu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-7.png" alt="" width="85%"/>
  <p>Hình 8.7 Trang chủ của trợ lý hỏi đáp</p>
</div>

Chức năng đầu tiên là hỏi đáp thông minh, sẽ có thể thực hiện truy xuất dựa trên tài liệu đã tải lên, và trả về nguồn tham khảo cùng tính toán độ tương đồng của tư liệu liên quan, đây là sự thể hiện năng lực của RAG tool, như Hình 8.8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-8.png" alt="" width="85%"/>
  <p>Hình 8.8 Trang chủ của trợ lý hỏi đáp</p>
</div>

Chức năng thứ hai là ghi chú học tập, như Hình 8.9, có thể tích chọn các khái niệm liên quan, cũng như viết nội dung ghi chú, phần này sử dụng đến Memory tool, sẽ lưu ghi chú cá nhân của bạn vào database, tiện cho việc thống kê và trả về báo cáo học tập tổng thể sau này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-9.png" alt="" width="85%"/>
  <p>Hình 8.9 Trang chủ của trợ lý hỏi đáp</p>
</div>

Cuối cùng là thống kê tiến độ học tập và sinh báo cáo, như Hình 8.10, chúng ta sẽ thấy được số lượng tài liệu đã nạp, số lần đặt câu hỏi, và số lượng ghi chú trong thời gian sử dụng trợ lý, cuối cùng sắp xếp kết quả hỏi đáp và ghi chú của chúng ta thành một file JSON để trả về.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-10.png" alt="" width="85%"/>
  <p>Hình 8.10 Trang chủ của trợ lý hỏi đáp</p>
</div>

Thông qua case study trợ lý hỏi đáp này, chúng ta đã minh họa cách sử dụng RAGTool và MemoryTool để xây dựng một <strong>hệ thống hỏi đáp tài liệu thông minh dựa trên Web</strong> hoàn chỉnh. Toàn bộ code có thể tìm thấy tại `code/chapter8/11_Q&A_Assistant.py`. Sau khi khởi động, truy cập `http://localhost:7860` là có thể sử dụng trợ lý học tập thông minh này.

Khuyến nghị người đọc tự mình chạy case study này, trải nghiệm năng lực của RAG và Memory, và trên nền tảng đó thực hiện mở rộng và tùy chỉnh, xây dựng nên ứng dụng thông minh phù hợp với nhu cầu của riêng mình!

## 8.5 Tổng kết và triển vọng của chương

Trong chương này, chúng ta đã thành công bổ sung cho framework HelloAgents hai năng lực cốt lõi: hệ thống Memory và hệ thống RAG.

Đối với những độc giả mong muốn học sâu và ứng dụng nội dung của chương này, chúng tôi đưa ra một số đề xuất sau:

1. Từ con số không, tự tay thiết kế một module Memory cơ bản, và dần dần lặp lại (iterate), bổ sung cho nó những đặc tính phức tạp hơn.

2. Thử nghiệm và đánh giá các embedding model và chiến lược truy xuất khác nhau trong dự án, tìm ra giải pháp tối ưu cho tác vụ cụ thể.

3. Áp dụng hệ thống Memory và RAG đã học vào một dự án cá nhân thực tế, kiểm nghiệm và nâng cao năng lực trong thực chiến.

Khám phá nâng cao

1. Theo dõi và nghiên cứu các repository memory, RAG tiên phong, học hỏi những cách triển khai xuất sắc.
2. Khám phá khả năng áp dụng kiến trúc RAG vào các kịch bản đa phương thức (văn bản + hình ảnh) hoặc xuyên phương thức.
3. Tham gia dự án mã nguồn mở HelloAgents, đóng góp ý tưởng và code của riêng bạn

Thông qua việc học tập chương này, bạn không chỉ nắm vững kỹ thuật triển khai của hệ thống Memory và RAG, mà quan trọng hơn là hiểu được cách chuyển hóa lý thuyết khoa học nhận thức thành giải pháp kỹ thuật thực tế. Lối tư duy liên ngành này sẽ đặt nền tảng vững chắc cho sự phát triển sâu hơn của bạn trong lĩnh vực AI.

Cuối cùng, hãy cùng thông qua một sơ đồ tư duy để tổng kết toàn bộ hệ thống tri thức của chương này, như Hình 8.11:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-11.png" alt="" width="85%"/>
  <p>Hình 8.11 Tổng kết tri thức Chương 8 của Hello-agents</p>
</div>

Chương này đã minh họa năng lực của hệ thống Memory và công nghệ RAG trong framework HelloAgents, chúng ta đã thành công xây dựng nên một trợ lý học tập thực sự có "trí tuệ". Kiến trúc này có thể dễ dàng mở rộng sang các kịch bản ứng dụng khác, như dịch vụ khách hàng, hỗ trợ kỹ thuật, trợ lý cá nhân và các lĩnh vực khác.

Trong chương tiếp theo, chúng ta sẽ tiếp tục khám phá cách thông qua context engineering (kỹ thuật ngữ cảnh) để nâng cao hơn nữa chất lượng hội thoại và trải nghiệm người dùng của Agent, hãy cùng chờ đón!

## Bài tập

> <strong>Gợi ý</strong>: Một số bài tập không có đáp án chuẩn, trọng tâm nằm ở việc rèn luyện cho người học sự hiểu biết toàn diện và năng lực thực hành đối với hệ thống Memory và công nghệ RAG.

1. Chương này đã giới thiệu bốn loại Memory: Working Memory, Episodic Memory, Semantic Memory và Perceptual Memory. Hãy phân tích:

   - Trong mục 8.2.5, mỗi loại Memory đều có công thức tính điểm riêng biệt. Hãy so sánh cơ chế tính điểm của Episodic Memory và Semantic Memory, giải thích vì sao Episodic Memory nhấn mạnh hơn vào "tính gần đây về thời gian" (trọng số 0.2), còn Semantic Memory nhấn mạnh hơn vào "truy xuất đồ thị" (trọng số 0.3)?
   - Nếu phải thiết kế một "trợ lý quản lý sức khỏe cá nhân" (cần ghi lại dữ liệu ăn uống, vận động, giấc ngủ của người dùng, và đưa ra lời khuyên sức khỏe), bạn sẽ kết hợp sử dụng bốn loại Memory này như thế nào? Hãy thiết kế kịch bản ứng dụng cụ thể cho từng loại Memory.
   - Working Memory áp dụng cơ chế TTL (Time To Live) để tự động dọn dẹp dữ liệu quá hạn. Hãy suy nghĩ: trong trường hợp nào thì Working Memory quan trọng nên được "tích hợp" (consolidate) thành Memory dài hạn? Làm thế nào để thiết kế một điều kiện kích hoạt tích hợp tự động?

2. Trong hệ thống RAG ở mục 8.3, chúng ta sử dụng MarkItDown để chuyển đổi thống nhất các tài liệu ở nhiều định dạng khác nhau thành Markdown. Hãy suy nghĩ sâu hơn:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thao tác thực tế

   - Chiến lược phân khối thông minh hiện tại dựa trên cấu trúc tiêu đề (#, ##, ###) của Markdown để phân tách. Nếu xử lý những tài liệu không có cấu trúc tiêu đề rõ ràng (như tiểu thuyết, điều khoản pháp luật), nên tối ưu chiến lược phân khối như thế nào? Hãy thử triển khai một thuật toán phân khối dựa trên "ranh giới ngữ nghĩa" (semantic boundary).
   - Trong mục 8.3.5 đã giới thiệu hai chiến lược truy xuất nâng cao MQE (mở rộng đa truy vấn) và HyDE (nhúng tài liệu giả định). Hãy chọn một kịch bản thực tế (như hỏi đáp tài liệu kỹ thuật, truy xuất tri thức y tế), so sánh sự khác biệt về hiệu quả của ba phương pháp truy xuất cơ bản, MQE và HyDE, và phân tích kịch bản phù hợp của từng phương pháp.
   - Chất lượng truy xuất của hệ thống RAG phần lớn phụ thuộc vào việc lựa chọn embedding model. Hãy so sánh ba phương án embedding được đề cập trong chương này (Bách Luyện API, Transformer local, TF-IDF), đánh giá theo các chiều độ chính xác, tốc độ, chi phí, khả năng triển khai offline, và đưa ra khuyến nghị lựa chọn.

3. Cơ chế "quên lãng" của hệ thống Memory là một thiết kế quan trọng mô phỏng nhận thức con người. Dựa trên MemoryTool ở mục 8.2.3, hãy hoàn thành các thực hành mở rộng sau:

   > <strong>Gợi ý</strong>: Đây là bài tập thực hành, khuyến nghị thao tác thực tế

   - Hiện tại cung cấp ba chiến lược quên lãng: dựa trên mức độ quan trọng, dựa trên thời gian, dựa trên dung lượng. Hãy thiết kế và triển khai một chiến lược "quên lãng thông minh", xem xét tổng hợp nhiều yếu tố như mức độ quan trọng, tần suất truy cập, suy giảm theo thời gian, sử dụng tính điểm có trọng số để quyết định những Memory nào nên bị quên lãng.
   - Trong hệ thống Agent chạy dài hạn, database Memory có thể tích lũy một lượng lớn dữ liệu. Hãy thiết kế một cơ chế "lưu trữ Memory" (archive): chuyển những Memory lâu không dùng nhưng có thể có giá trị sang cold storage, và khôi phục lại khi cần. Cơ chế này nên được tích hợp với bốn loại Memory hiện có như thế nào?
   - Suy nghĩ: nếu Agent cần "quên" một số thông tin nhạy cảm (như dữ liệu riêng tư của người dùng), chỉ đơn giản xóa khỏi database liệu có đủ không? Trong trường hợp sử dụng vector database và graph database, làm thế nào để đảm bảo dữ liệu được xóa bỏ triệt để?

4. Trong case study "trợ lý học tập thông minh" ở mục 8.4, chúng ta đã kết hợp MemoryTool và RAGTool. Hãy phân tích sâu hơn:

   - Phương thức `ask_question()` trong case study sử dụng đồng thời truy xuất RAG và truy xuất Memory. Hãy phân tích: trong trường hợp nào nên ưu tiên sử dụng RAG? Trong trường hợp nào nên ưu tiên sử dụng Memory? Làm thế nào để thiết kế một cơ chế "định tuyến thông minh" (intelligent routing) để tự động chọn phương thức truy xuất phù hợp nhất?
   - Báo cáo học tập hiện tại (`generate_report()`) chỉ chứa thông tin thống kê. Hãy mở rộng chức năng này, thiết kế một bộ sinh báo cáo học tập thông minh hơn: có thể phân tích quỹ đạo học tập của người dùng, nhận diện điểm mù tri thức (knowledge blind spot), đề xuất nội dung học tập tiếp theo. Việc này cần sử dụng đến những loại Memory và chiến lược truy xuất nào?
   - Giả sử bạn cần triển khai trợ lý học tập này thành dịch vụ Web đa người dùng, mỗi người dùng đều có Memory và knowledge base độc lập. Hãy thiết kế phương án cách ly dữ liệu: làm thế nào để thực hiện cách ly dữ liệu ở cấp độ người dùng trong Qdrant và Neo4j? Làm thế nào để tối ưu hiệu năng truy xuất trong kịch bản đa người dùng?

5. Semantic Memory sử dụng graph database Neo4j để lưu trữ knowledge graph. Hãy suy nghĩ:

   - Trong cách triển khai Semantic Memory ở mục 8.2.5, hệ thống sẽ tự động trích xuất thực thể và quan hệ để xây dựng knowledge graph. Hãy phân tích: độ chính xác của việc trích xuất tự động này như thế nào? Trong trường hợp nào có thể trích xuất ra thực thể hoặc quan hệ sai? Làm thế nào để thiết kế một cơ chế "đánh giá chất lượng knowledge graph"?
   - Một ưu thế quan trọng của knowledge graph là hỗ trợ suy luận quan hệ phức tạp. Hãy thiết kế một kịch bản truy vấn, tận dụng tối đa năng lực truy vấn đồ thị của Neo4j (như quan hệ đa bước nhảy - multi-hop, tìm đường đi - path finding), thực hiện một tác vụ mà truy xuất vector thuần túy không thể hoàn thành.
   - So sánh chiến lược lai "truy xuất vector + truy xuất đồ thị" của Semantic Memory với truy xuất vector thuần túy: trong loại truy vấn nào thì truy xuất đồ thị có thể mang lại sự cải thiện hiệu năng đáng kể? Hãy minh họa bằng ví dụ cụ thể.

## Tài liệu tham khảo

[1] Atkinson, R. C., & Shiffrin, R. M. (1968). Human memory: A proposed system and its control processes. In *Psychology of learning and motivation* (Vol. 2, pp. 89-195). Academic press.
