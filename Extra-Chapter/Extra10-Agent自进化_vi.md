# Agent Self-Evolution: Bốn loại vòng lặp khép kín (closed loop) cho sự tự tiến hóa của tác tử

## Lời mở đầu: Vì sao Agent cần tự tiến hóa?

Trong các chương trước, chúng ta đã học về ReAct, Reflection, MCP, Agent Skills, hệ thống bộ nhớ (memory system), kỹ thuật ngữ cảnh (context engineering) và Agentic RL. Khi đặt tất cả những năng lực này cạnh nhau, một câu hỏi rất tự nhiên sẽ xuất hiện: nếu mỗi lần tác tử (agent) đều phải mò mẫm lại từ những lỗi tương tự, mỗi lần đều tra lại cùng một tài liệu, mỗi lần đều viết lại cùng một bộ các bước thao tác, thì thực ra nó vẫn chưa thực sự "trưởng thành".

**Agent Self-Evolution** bàn chính về vấn đề này: liệu tác tử có thể tích lũy (lắng đọng) các tín hiệu như quỹ đạo tương tác (interaction trajectory), phản hồi nhiệm vụ, sự sửa lỗi của người dùng, kết quả thực thi công cụ, kinh nghiệm tập thể, v.v., và để những gì đã tích lũy đó tiếp tục ảnh hưởng đến hành vi về sau hay không?

"Tự tiến hóa" ở đây không đồng nghĩa với bộ nhớ dài hạn (long-term memory) đơn thuần, cũng không đồng nghĩa với việc cài đặt thủ công một **plugin**. Một định nghĩa thực tế hơn là:

> Agent tự tiến hóa là một hệ thống tác tử có khả năng dựa trên quỹ đạo tương tác của chính nó, phản hồi nhiệm vụ hoặc tín hiệu môi trường để cập nhật liên tục ngữ cảnh, bộ nhớ, kỹ năng, công cụ, quy trình làm việc (workflow), mã nguồn hoặc tham số mô hình, và để những cập nhật này ảnh hưởng đến hiệu suất thực hiện nhiệm vụ trong tương lai.

Định nghĩa này có ba điểm then chốt.

1. **Được thúc đẩy bởi kinh nghiệm**: Các cập nhật đến từ nhiệm vụ thực tế, phản hồi thực thi, sự sửa lỗi của người dùng, kết quả đánh giá hoặc tín hiệu môi trường, chứ không phải từ cấu hình thủ công một lần.
2. **Có hiệu lực liên tục**: Các cập nhật sẽ đi vào bộ nhớ, thư viện kỹ năng, workflow, mã nguồn hoặc tham số, và tiếp tục phát huy tác dụng trong các nhiệm vụ tương lai.
3. **Có thể đánh giá, có thể khôi phục (rollback)**: Sự tự tiến hóa càng mạnh thì càng cần bộ đánh giá (evaluator), ghi nhận phiên bản, **sandbox**, kiểm soát quyền hạn và cơ chế **rollback**.

Chương này tổ chức nội dung theo tiêu chí "vòng lặp tiến hóa được đặt ở đâu, đối tượng tiến hóa là gì", chia thành bốn loại: **vòng lặp ngữ cảnh tích hợp sẵn (built-in context loop)**, **vòng lặp tài sản hóa kỹ năng (skill assetization loop)**, **vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể (external supervision / collective intelligence loop)**, **vòng lặp tự sửa đổi tham số, mã nguồn hoặc workflow**. Mỗi loại chọn ra một số phương pháp hoặc kỹ thuật tiêu biểu, **tổng cộng 10 dự án**.

<div align="center">
  <img src="./images/Extra10-figures/agent-evolution.png" alt="Agent Self-Evolution: four closed loops and ten representative projects" width="88%"/>
  <p><strong>Hình 1</strong> Agent Self-Evolution: Bốn loại vòng lặp khép kín và 10 dự án tiêu biểu. Ràng buộc xuyên suốt: mọi con đường tiến hóa đều phải đặt trên nền tảng đánh giá, phiên bản, rollback, quyền hạn và quản trị chuỗi cung ứng (supply chain governance).</p>
</div>

## Tổng quan bốn loại vòng lặp khép kín

Trước tiên, hãy dùng một bảng để thiết lập góc nhìn tổng thể. Bảng này giúp mọi người nhanh chóng đánh giá: một hệ thống tự tiến hóa rốt cuộc đang thay đổi cái gì, dựa vào phản hồi nào để thay đổi, và sau khi thay đổi thì ảnh hưởng đến nhiệm vụ lần sau như thế nào.


| Loại             | Phương pháp hoặc kỹ thuật tiêu biểu                                                                                                                                                                   | Điểm chính tóm lược                                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Vòng lặp ngữ cảnh tích hợp sẵn        | 1. [Hermes Agent](https://github.com/NousResearch/hermes-agent) <br>2. [Agent Zero](https://github.com/agent0ai/agent-zero)                                                         | 1. Hermes đưa bộ nhớ, truy xuất phiên hội thoại và tạo kỹ năng vào ngay bản thể Agent. <br>2. Agent Zero hình thành ngữ cảnh liên tục thông qua dự án, lịch sử chat, công cụ và tác tử con (sub-agent).                              |
| Vòng lặp tài sản hóa kỹ năng        | 1. [Darwin Skill](https://github.com/alchaincyf/darwin-skill) <br>2. [JiuwenClaw](https://github.com/openJiuwen-ai/jiuwenclaw) <br>3. [EvoSkill](https://github.com/sentient-agi/EvoSkill) | 1. Darwin Skill coi SKILL.md là tài sản có thể đánh giá, có thể rollback. <br>2. JiuwenClaw tối ưu Skill tại thời điểm chạy (runtime) dựa trên phản hồi. <br>3. EvoSkill tạo, kiểm thử và giữ lại các biến thể kỹ năng từ những quỹ đạo thất bại. |
| Vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể    | 1. [Ultron](https://github.com/modelscope/ultron) <br>2. [OpenSpace](https://github.com/HKUDS/OpenSpace) <br>3. [SkillClaw](https://github.com/AMAP-ML/SkillClaw)                          | 1. Ultron chưng cất (distill) kinh nghiệm cá nhân thành bộ nhớ tập thể, kỹ năng và Harness. <br>2. OpenSpace đóng vai trò dịch vụ tiến hóa bên ngoài, duy trì phả hệ phiên bản (version lineage) của kỹ năng. <br>3. SkillClaw hợp nhất kinh nghiệm xuyên phiên, xuyên thiết bị, xuyên người dùng thành kỹ năng dùng chung.   |
| Vòng lặp tự sửa đổi tham số, mã nguồn hoặc workflow | 1. [OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL) <br>2. [Agent Lightning](https://github.com/microsoft/agent-lightning)                                                   | 1. OpenClaw-RL chuyển phản hồi hội thoại thực tế thành tín hiệu RL/OPD bất đồng bộ. <br>2. Agent Lightning tách rời (decouple) việc thực thi Agent và huấn luyện RL.                             |


Bốn loại này, đi từ trên xuống dưới, thường có nghĩa là giới hạn trên về năng lực ngày càng cao, và rủi ro kỹ thuật cũng ngày càng cao. Tiến hóa ở tầng ngữ cảnh và kỹ năng là dễ kiểm toán (audit) và rollback nhất; tầng trí tuệ tập thể bắt đầu liên quan đến lưu trữ dùng chung, quyền hạn và quyền riêng tư; tự sửa đổi ở tầng tham số, mã nguồn hoặc workflow là gần nhất với việc "chiến lược (policy) đã thay đổi", nhưng cũng phụ thuộc nhiều nhất vào bộ đánh giá đáng tin cậy và môi trường thực thi được cô lập.

## I. Vòng lặp ngữ cảnh tích hợp sẵn: Để Agent học trong chính vòng lặp chính của nó

Tư tưởng cốt lõi của vòng lặp ngữ cảnh tích hợp sẵn là: không trực tiếp sửa đổi tham số mô hình, mà để Agent ghi kinh nghiệm vào bộ nhớ, văn bản phản tư (reflection), chỉ mục phiên hội thoại hoặc thư mục kỹ năng, rồi lấy ra dùng lại trong nhiệm vụ tiếp theo.

Ưu điểm của loại hệ thống này là nhẹ, triển khai nhanh, khả năng giải thích cao. Chúng không cần cụm máy huấn luyện, cũng không cần hạ tầng RL trực tuyến phức tạp. Nhược điểm cũng rất rõ ràng: nếu năng lực của mô hình nền không đủ, chỉ dựa vào ngữ cảnh và bộ nhớ thì rất khó vượt qua giới hạn trên; nếu không có cơ chế quản trị, bộ nhớ sai và phản tư sai cũng sẽ liên tục làm ô nhiễm các hành vi về sau.

### 1. Hermes Agent: Đưa vòng lặp học tập vào ngay bản thể Agent

Địa chỉ dự án: [https://github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

Hermes Agent là dự án kỹ thuật đáng được trình bày riêng nhất trong loại này. Nó hướng theo lộ trình [self-improving AI agent](https://hermes-agent.nousresearch.com/docs): điểm mạnh không nằm ở một chức năng đơn lẻ nào, mà ở việc nhồi nhiều cơ chế "trở nên cứng cáp hơn trong quá trình sử dụng" vào cùng một vòng lặp hội thoại chính. Các mảnh ghép điển hình bao gồm: bộ nhớ lâu dài (persistent memory) do Agent chủ động duy trì (`MEMORY.md` / `USER.md`, v.v.), truy xuất xuyên phiên được kích hoạt theo nhu cầu (SQLite + FTS5, kết hợp với tóm tắt), tải kỹ năng dần dần (progressive loading) và việc kỹ năng được tạo mới hoặc viết lại trong quá trình sử dụng, cùng với các khía cạnh bảo mật như phê duyệt lệnh và cô lập terminal đa backend.

<div align="center">
  <img src="./images/Extra10-figures/hermes-banner.png" alt="Hermes Agent project banner" width="88%"/>
  <p><strong>Hình 2</strong> Hermes Agent</p>
</div>

Có thể hình dung sự tự tiến hóa của Hermes như một vòng lặp nhẹ bám sát vòng lặp chính: **Nhiệm vụ của người dùng → Vòng lặp chính của Hermes → Công cụ / Terminal / Cổng tin nhắn (message gateway) → Phản hồi thực thi và sự sửa lỗi của người dùng**, đầu ra rẽ nhánh ghi vào **bộ nhớ được tuyển chọn (curated memory)** và **Skills**; bộ nhớ thông qua **session_search** cùng với kỹ năng hội tụ vào **ngữ cảnh của lượt tiếp theo**, rồi quay trở lại vòng lặp chính.

Ý nghĩa của lộ trình này là "việc học" diễn ra ngay trên con đường gọi thực tế hằng ngày. Ví dụ, sau khi kết thúc một lần gỡ lỗi phức tạp, Agent có thể ghi thói quen dùng lệnh, ràng buộc môi trường, nguyên nhân thất bại và con đường sửa lỗi cuối cùng vào bộ nhớ hoặc kỹ năng; sau này khi gặp lại vấn đề cùng loại, không cần phải thử dò lại toàn bộ không gian tìm kiếm từ đầu.

Tuy nhiên, đối tượng cập nhật chủ yếu của Hermes vẫn là **ngữ cảnh, bộ nhớ và kỹ năng**, chứ không phải cập nhật trực tuyến trọng số mô hình. Do đó, nó phù hợp hơn để được xem là "hình thái kỹ thuật trưởng thành của vòng lặp ngữ cảnh tích hợp sẵn", chứ không phải là hệ thống tự huấn luyện ở tầng trọng số.

### 2. Agent Zero: Đưa bộ nhớ dự án, công cụ động và tác tử con vào vòng lặp chính

Địa chỉ dự án: [https://github.com/agent0ai/agent-zero](https://github.com/agent0ai/agent-zero)  

Agent Zero phù hợp để xếp vào **vòng lặp ngữ cảnh tích hợp sẵn**. Nó là một **[dynamic, organic agentic framework](https://github.com/agent0ai/agent-zero)** hướng tới các nhiệm vụ thực tế: **không biến Agent thành một script đơn mục đích**, mà cung cấp cho nó một môi trường cấp hệ điều hành có thể sử dụng (terminal, thực thi mã, tệp tin, tự động hóa trình duyệt, v.v.), và cho phép **tạo hoặc viết lại công cụ theo nhu cầu** trong quá trình nhiệm vụ tiến triển. Điều này không liên quan đến việc tinh chỉnh (fine-tune) trọng số ngoại tuyến, mà giống như "trạng thái bàn làm việc" không ngừng tích lũy theo dự án và phiên hội thoại hơn.

Về mặt kỹ thuật, có vài khối xây dựng liên quan trực tiếp đến vòng lặp khép kín: **Project** cô lập không gian làm việc, hướng dẫn, bộ nhớ, khóa bí mật (secret), cơ sở tri thức, kho mã (repository) và cấu hình mô hình đặt sẵn (preset), có thể clone repository vào ngữ cảnh dự án độc lập; **Skills** tuân theo quy ước `SKILL.md` mở, có thể kích hoạt ở cấp toàn cục, cấp dự án hoặc cấp phiên hiện tại; **Agent Profiles** dùng để chuyển đổi hành vi, ghi đè prompt, chuỗi công cụ (toolchain) và cấu hình mô hình mà không cần viết lại toàn bộ hệ thống; về mặt cộng tác thì nhấn mạnh việc Agent cấp trên tạo ra các **subordinate agents** (tác tử cấp dưới), mỗi tác tử cấp dưới tự giữ một ngữ cảnh nhỏ hơn và báo cáo kết quả. Nó còn hỗ trợ MCP, plugin và bố cục `prompts/`, `tools/` có thể kiểm tra được; phía Web UI còn có các năng lực trực quan hóa hướng tới sự đồng diễn giữa người và máy như Universal Canvas, chú thích trên trình duyệt (xem chi tiết trong tài liệu chính thức).

<div align="center">
  <img src="./images/Extra10-figures/agent-zero-banner.png" alt="Agent Zero framework banner" width="88%"/>
  <p><strong>Hình 3</strong> Agent Zero</p>
</div>

Dưới cùng góc nhìn đó, luồng hồi lưu chính có thể được nén thành một chuỗi: **Nhiệm vụ và mục tiêu → Project → Chat → Skills / MCP / Plugin → Tác tử cấp dưới → Phản hồi và sản phẩm đầu ra → Project** (các năng lực sản phẩm như canvas, chú thích trình duyệt xin được lược bỏ ở đây).

Agent Zero và Hermes ở cùng một phân khúc: cả hai đều là "host" (máy chủ chứa) Agent phía cá nhân hoặc nhóm, có ngữ cảnh "dày lên theo quá trình sử dụng", chứ không phải là một thuật toán đơn lẻ trong bài báo khoa học. Agent Zero còn mang màu sắc giao thoa với tài sản hóa kỹ năng (`SKILL.md`) và phân công đa tác tử, nhưng chương này vẫn đặt nhãn chính của nó là vòng lặp ngữ cảnh tích hợp sẵn, bởi vì sự tăng trưởng dễ kiểm toán nhất trước hết xảy ra trong ranh giới dự án, trạng thái phiên hội thoại, cấu hình công cụ và dấu vết thực thi thực tế.

## II. Vòng lặp tài sản hóa kỹ năng: Để kinh nghiệm lắng đọng thành Skill có thể tái sử dụng

Tư tưởng cốt lõi của vòng lặp tài sản hóa kỹ năng là: hiện thực hóa kinh nghiệm thành tài sản kỹ năng có thể đọc được, có thể quản lý phiên bản, có thể kiểm thử, có thể di chuyển.

Trong hệ sinh thái Agent Skills, `SKILL.md` không chỉ là một tệp hướng dẫn, nó có thể trở thành bộ nhớ thủ tục (procedural memory) của Agent: khi nào thì sử dụng, thực thi như thế nào, gọi những script nào, tuân thủ những ràng buộc nào, xử lý ngoại lệ ra sao. Mấu chốt của sự tự tiến hóa ở tầng kỹ năng chính là làm cho những kỹ năng này không còn chỉ dựa vào bảo trì thủ công, mà có thể được đánh giá, viết lại, xác thực và rollback.

### 3. Darwin Skill: Dùng đánh giá và cơ chế ratchet để tối ưu Skill

Địa chỉ dự án: [https://github.com/alchaincyf/darwin-skill](https://github.com/alchaincyf/darwin-skill)

Dưới đây chúng ta bắt đầu loại vòng lặp thứ hai từ Darwin Skill: `SKILL.md` được coi một cách rõ ràng là tài sản có thể đo lường và lặp lại cải tiến. Chủ trương tối ưu SKILL của Darwin Skill có thể tóm gọn là "**tối ưu Agent Skills của bạn giống như huấn luyện mô hình**", phương pháp luận trực tiếp bám theo [autoresearch](https://github.com/karpathy/autoresearch) của Karpathy: dùng mục tiêu có thể định lượng để thúc đẩy thay đổi, và dùng **bánh cóc (ratchet)** để chỉ giữ lại những cải thiện có thể xác minh, phần còn lại thì **git revert**, tránh để đường cơ sở (baseline) âm thầm trở nên tệ hơn theo thời gian.

Về cơ chế, nó được chia thành vài nguyên tắc cứng: **tài sản có thể chỉnh sửa duy nhất** (mỗi lần chỉ sửa một `SKILL.md` cần tối ưu), **đánh giá kép** (phía cấu trúc thiên về phân tích tĩnh, phía hiệu quả thì phải chạy trên các prompt kiểm thử để xem đầu ra), **chấm điểm độc lập** (dùng Agent con để chấm điểm, giảm tình trạng "tự sửa tự chấm"), **con người trong vòng lặp (human-in-the-loop)** (tạm dừng giữa các giai đoạn, hiển thị diff và sự thay đổi điểm số rồi mới tiếp tục). Phía xác thực hiệu quả sẽ dùng đến các bộ kiểm thử kiểu như `test-prompts.json`; tổng điểm 100 đến từ **rubric tám chiều**, trong đó tỷ lệ phân bổ điểm giữa các chiều cấu trúc và chiều hiệu quả được ghi rất rõ ràng (trong phần hiệu quả, "hiệu suất thực nghiệm" có trọng số cao nhất).

<div align="center">
  <img src="./images/Extra10-figures/daerwin-banner.png" alt="Darwin Skill project banner" width="88%"/>
  <p><strong>Hình 4</strong> Darwin Skill</p>
</div>

Hình dưới đây nhất quán với **Core Loop** của nó, tương ứng với Evaluate → Improve → Validate → Confirm → Keep or Revert.

<div align="center">
  <img src="./images/Extra10-figures/darwin-loop.png" alt="Darwin Skill core evaluation loop" width="88%"/>
  <p><strong>Hình 5</strong> Darwin Skill: Vòng lặp khép kín Evaluate → Improve → Validate → Confirm → Keep or Revert</p>
</div>

Quy trình này có thể chia thành năm bước:

1. **Evaluate**: Thực hiện phân tích cấu trúc và xác thực hiệu quả đối với `SKILL.md` mục tiêu, tổng hợp thành điểm có trọng số tám chiều.
2. **Improve**: Tìm ra chiều có điểm thấp nhất, tạo ra một lượt viết lại có mục tiêu và commit thay đổi.
3. **Validate**: Kiểm tra lại trên tập prompt kiểm thử (như `test-prompts.json`) hoặc các nhiệm vụ thực tế tương đương.
4. **Confirm**: Hiển thị diff và sự thay đổi điểm số, để con người xác nhận có tiến vào lượt tiếp theo hoặc Skill tiếp theo hay không.
5. **Keep or Revert**: Nếu tổng điểm mới cao hơn mức tốt nhất hiện tại thì giữ lại, ngược lại thì rollback; ratchet đảm bảo đường cơ sở hiệu lực tăng đơn điệu, không giảm.

Rubric tám chiều chia tổng điểm ra nhiều chiều (tỷ lệ phân bổ điểm giữa phía cấu trúc và phía hiệu quả xem trong hình minh họa); "hiệu suất thực nghiệm" có trọng số cao nhất trong các chiều hiệu quả.

<div align="center">
  <img src="./images/Extra10-figures/darwin-score.png" alt="Darwin Skill eight-dimension rubric" width="88%"/>
  <p><strong>Hình 6</strong> Darwin Skill: Hệ thống chấm điểm tám chiều (tổng điểm 100)</p>
</div>

Thiết kế quan trọng nhất của Darwin Skill không phải là "tự động sửa kỹ năng", mà là **chỉ để những cải tiến có thể xác minh được ở lại**. Mọi toolchain tương thích với host mở **Agent Skill / `SKILL.md`** đều có thể tích hợp, bao gồm Claude Code, Codex, OpenClaw, Trae, CodeBuddy, v.v.; Darwin Skill đóng vai trò là một bộ tối ưu kỹ năng tương đối độc lập trong số đó.

### 4. JiuwenClaw: Tự tiến hóa kỹ năng tại runtime

Địa chỉ dự án: [https://github.com/openJiuwen-ai/jiuwenclaw](https://github.com/openJiuwen-ai/jiuwenclaw)  
Tài liệu về tự tiến hóa kỹ năng: [https://github.com/openJiuwen-ai/jiuwenclaw/blob/develop/docs/en/SkillSelfEvolution.md](https://github.com/openJiuwen-ai/jiuwenclaw/blob/develop/docs/en/SkillSelfEvolution.md)

Khẩu hiệu của JiuwenClaw, hướng tới việc sử dụng theo kiểu đồng hành lâu dài, là "Understands You. Evolves With You": **Autonomous Evolution** tức là liên tục cải tiến các kỹ năng liên quan dựa trên phản hồi khi người dùng không hài lòng hoặc khi thực thi gặp lỗi. Về mặt triển khai, nó dựa vào **SkillCallOperator** để điều phối việc đọc, ghi và hợp nhất, cùng với **SignalDetector** (dựa trên quy tắc, **không gọi LLM**) để giám sát các manh mối sửa lỗi trong kết quả công cụ và cách diễn đạt của người dùng; các sự kiện có thể quy về kỹ năng hiện tại được giao cho **SkillEvolutionManager** để điều phối việc quét và sinh, **SkillOptimizer** gọi LLM để viết ra các mục tiến hóa khi cần thay đổi, trước tiên đưa vào **`evolutions.json`**, rồi vào thời điểm thích hợp sẽ **solidify** (cố định hóa) và hợp nhất trở lại **`SKILL.md`**, cũng có thể kích hoạt thủ công thông qua **`/evolve`**. Các tín hiệu loại thất bại có xu hướng được ghi vào các tiểu mục kiểu **Troubleshooting**, còn sự sửa lỗi của người dùng thì thường được sắp xếp thành **Examples**. Mỗi kỹ năng có thư mục riêng dưới workspace (đường dẫn điển hình có dạng `~/.jiuwenclaw/workspace/agent/skills/<skill_name>/`), trong cấu hình có thể bật **`evolution_auto_scan`**, và sau khi kết thúc một lượt công cụ cũng có thể bổ sung bản ghi tiến hóa ở chế độ nền (background).

Xét về phân loại, nó vẫn phù hợp nhất để xếp vào **vòng lặp tài sản hóa kỹ năng**: phần tăng trưởng bền vững nằm ở `SKILL.md` và `evolutions.json`, chứ không phải ở bản tóm tắt hội thoại dùng một lần.

<div align="center">
  <img src="./images/Extra10-figures/jiuwenclaw.png" alt="JiuwenClaw overview and skill evolution" width="88%"/>
  <p><strong>Hình 7</strong> JiuwenClaw: Tổng quan dự án và bối cảnh tự tiến hóa kỹ năng tại runtime</p>
</div>

Một đường ống (pipeline) nhất quán với **Evolution flow** trong tài liệu của repository: **Hội thoại người dùng hoặc thực thi công cụ → SignalDetector → SkillEvolutionManager → SkillOptimizer → `evolutions.json` → solidify → SkillCallOperator** (thành phần sau cùng thống nhất việc đọc ghi và tải **`SKILL.md`** đã hợp nhất trước lần gọi tiếp theo).

So với Darwin Skill: Darwin nhấn mạnh hơn vào bộ đánh giá đầy đủ kiểu "giống như huấn luyện mô hình", chấm điểm tám chiều, ratchet và con người trong vòng lặp mạnh mẽ; JiuwenClaw nhúng sự tiến hóa vào hội thoại hằng ngày và vòng lặp công cụ, được thúc đẩy bởi tín hiệu, các mục trước tiên vào JSON rồi mới cố định hóa vào tài liệu, trọng tâm kỹ thuật nằm ở **tính khả dụng trực tuyến và lặp nhanh**. Nó cũng không tránh khỏi việc giao thoa với **vòng lặp ngữ cảnh tích hợp sẵn** (phải phát hiện phản hồi ngay trong cùng một lượt hội thoại), nhưng chương này vẫn lấy tài sản kỹ năng làm nhãn chính, vì phần tăng trưởng dài hạn có thể kiểm toán chủ yếu nằm trong tệp kỹ năng và bản ghi tiến hóa.

### 5. EvoSkill: Tiến hóa các biến thể kỹ năng từ quỹ đạo thất bại

Địa chỉ dự án: [https://github.com/sentient-agi/EvoSkill](https://github.com/sentient-agi/EvoSkill)  
Địa chỉ bài báo: [https://arxiv.org/abs/2603.02766](https://arxiv.org/abs/2603.02766)

EvoSkill hướng tới việc tự động khám phá và cải tiến kỹ năng cho **coding agent**: trong bối cảnh repository, nó mở rộng tư duy "dựa vào phản hồi để sửa một chỗ trong prompt" kiểu [GEPA](https://github.com/sentient-agi/gepa-plus) thành sự lặp lại trên **toàn bộ agent program** (có thể đồng thời đề xuất nhiều đột biến của **skill** và **system prompt**), chấm điểm trên **tập xác thực được giữ riêng (held-out validation set)**, và các ứng viên xuất sắc mỗi vòng sẽ bước vào vòng tiếp theo dưới dạng **trạng thái chương trình hoàn toàn mới**, chứ không chỉ vá tại chỗ một đoạn văn bản.

Vòng lặp tự hỗ trợ được chia thành năm đoạn trong tài liệu: **Base Agent** dùng cấu hình tốt nhất hiện tại để chạy các mẫu trong benchmark; **Proposer** đối chiếu với các mẫu thất bại để đề xuất thay đổi có mục tiêu; **Generator** viết ra tệp kỹ năng mới hoặc viết lại system prompt; **Evaluator** chấm điểm phiên bản mới trên dữ liệu held-out; **Frontier** duy trì Top-N bộ chương trình có hiệu suất tốt nhất, và quản lý phiên bản dưới dạng các nhánh Git **`program/*`** (ngoài ra còn có tag **`frontier/*`** để đánh dấu các thành viên tiên phong), thuận tiện cho việc kiểm toán và rollback bằng `evoskill diff`, `evoskill skills`. Về mặt kỹ thuật, nó được điều khiển thông qua **`evoskill init`** / **`evoskill run`**, tập dữ liệu phần lớn là CSV có kèm đáp án chuẩn, có thể viết `.evoskill/task.md` theo từng nhiệm vụ, chế độ thực thi hỗ trợ máy cục bộ, Docker hoặc sandbox từ xa Daytona; harness của host bao phủ Claude Code, OpenCode, OpenHands, Goose, Codex CLI, v.v. Trong cấu hình tiến hóa có thể chọn **`skill_only`** hoặc **`prompt_only`**, việc chấm điểm hỗ trợ nhiều loại **scorer** như so khớp quy tắc, LLM-as-judge, script, v.v.

<div align="center">
  <img src="./images/Extra10-figures/evoskill-framework.png" alt="EvoSkill discovery framework" width="88%"/>
  <p><strong>Hình 8</strong> EvoSkill: Framework đồng tiến hóa kỹ năng và prompt dành cho Agent lập trình</p>
</div>

Khác với Darwin Skill tập trung vào "một SKILL.md duy nhất + ratchet tám chiều", và JiuwenClaw tập trung vào tín hiệu từ phiên hội thoại trực tuyến, EvoSkill gần hơn với "sự tiến hóa của toàn bộ gói cấu hình Agent" **được thúc đẩy bởi benchmark ngoại tuyến**: bằng chứng thành bại đến từ các đánh giá có thể lặp lại, sản phẩm đầu ra là các thư mục skills có thể sao chép và các artifact kiểu `program.yaml`, phù hợp với những nhóm muốn thu hẹp trợ lý lập trình từ mô hình đa dụng thành một **pipeline chuyên biệt**.

## III. Vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể: Để kinh nghiệm luân chuyển giữa các Agent

Hai loại vòng lặp đầu tiên chủ yếu phục vụ cho một Agent đơn lẻ. Đến vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể, kinh nghiệm bắt đầu luân chuyển xuyên phiên, xuyên thiết bị, xuyên người dùng, xuyên Agent.

Giá trị của loại hệ thống này rất trực tiếp: cái hố mà một Agent đã từng vấp phải thì không nên để tất cả các Agent khác vấp lại một lần nữa; một workflow mà một nhóm đã xác thực thì không nên chỉ nằm im trong lịch sử cục bộ của một người nào đó. Cái giá của nó cũng rất trực tiếp: chia sẻ kinh nghiệm đòi hỏi quyền hạn, ẩn danh hóa dữ liệu nhạy cảm (desensitization), quản trị phiên bản, kiểm soát chất lượng (quality gate) và kiểm toán.

### 6. Trí tuệ tập thể Ultron: Memory Hub, Skill Hub, Harness Hub

Địa chỉ dự án: [https://github.com/modelscope/ultron](https://github.com/modelscope/ultron)  
Địa chỉ demo: [https://writtingforfun-ultron.ms.show/dashboard](https://writtingforfun-ultron.ms.show/dashboard)

Ultron là **self-evolving collective intelligence** (trí tuệ tập thể tự tiến hóa) dành cho Agent đa dụng: chưng cất kinh nghiệm rải rác trong các phiên hội thoại thành **tri thức tập thể dễ truy xuất và tái sử dụng**. Ba năng lực được làm nổi bật ra bên ngoài là: **bộ nhớ tập thể phân tầng**, **kỹ năng tập thể có thể tự tiến hóa theo bằng chứng**, **bản thiết kế (blueprint) Harness có thể chia sẻ** (nhập một lần toàn bộ tổ hợp nhân vật (persona), bộ nhớ và kỹ năng). **Trajectory Hub** phía máy chủ tiếp nhận các quỹ đạo `.jsonl`: phân đoạn nhiệm vụ, chỉ số `ms_agent.trajectory`, khử trùng lặp bằng dấu vân tay gia tăng (incremental fingerprint) và trích xuất ở chế độ nền; các quỹ đạo chất lượng cao còn có thể dùng cho **SFT / tự huấn luyện** (có thể kết nối với các framework huấn luyện như [Twinkle](https://github.com/modelscope/twinkle) Workbench), từ đó **giảm chi phí gọi mô hình đầu cuối** ở phía định tuyến (routing). Đây là một vòng lặp dài hạn "quỹ đạo nhập kho → bộ nhớ và kỹ năng phát triển → phân phối blueprint → nhiều quỹ đạo hơn hồi lưu", chứ không phải chỉ dựa vào việc chồng chất prompt dài.

Phần dưới đây trình bày tách riêng theo ba khối năng lực trên bảng điều khiển (console) là Memory Hub, Skill Hub, Harness Hub; tổng thể vẫn thuộc loại vòng lặp thứ ba: kinh nghiệm rời khỏi phiên hội thoại đơn máy, tiến vào tầng dùng chung có thể quản trị.

<div align="center">
  <img src="./images/Extra10-figures/ultron-banner.png" alt="Ultron collective intelligence banner" width="88%"/>
  <p><strong>Hình 9</strong> Ultron: Bộ nhớ tập thể, kỹ năng tập thể và Harness dùng chung</p>
</div>

**Memory Hub** đảm nhận vai trò "kho dữ kiện có thể truy hồi" ở phía tập thể. Các điểm năng lực chính bao gồm: phân tầng **HOT / WARM / COLD** và tái cân bằng theo số lần trúng (hit), truy xuất ngữ nghĩa bằng vector kết hợp với trọng số theo tầng, các cấp tóm tắt **L0 / L1 / Full** (truy xuất trả về tóm tắt ngắn trước để tiết kiệm token, kéo toàn văn khi cần), tự động phân loại kiểu khi tải lên, hợp nhất các vector gần trùng lặp và sắp xếp hàng loạt, mở rộng query truy xuất theo ý định (intent), suy giảm độ "nóng" theo hàm mũ theo thời gian, cũng như phát hiện và ẩn danh hóa **PII** tiếng Trung và tiếng Anh dựa trên **Presidio** trước khi nhập kho. Hình dưới đây tương ứng với góc nhìn "duyệt và truy xuất bộ nhớ phân tầng" trong bảng điều khiển.

<div align="center">
  <img src="./images/Extra10-figures/ultron-memory-hub.png" alt="Ultron Memory Hub" width="88%"/>
  <p><strong>Hình 10</strong> Ultron Memory Hub: Duyệt và truy xuất bộ nhớ tập thể phân tầng</p>
</div>

Phía **Skill Hub** vừa có các gói kỹ năng nội bộ được **chưng cất** từ bộ nhớ "nóng", vừa kết nối với các chỉ mục bên ngoài như **ModelScope Skill Hub** để khám phá thống nhất. Con đường **Skill self-evolution** nhấn mạnh: các bộ nhớ liên quan trước tiên hình thành cụm ngữ nghĩa (semantic cluster), sau đó **kết tinh** thành kỹ năng workflow nhiều bước, khi bằng chứng tích lũy thì **tái kết tinh**; kết hợp với **provenance-grounded verification** (xác minh dựa trên nguồn gốc) và **structure-score upgrade gate** (cổng nâng cấp theo điểm cấu trúc), tránh việc kỹ năng sau khi tiến hóa bị **thoái lùi** về điểm cấu trúc. Nó cũng có mối quan hệ kế thừa với các tư tưởng tiến hóa kỹ năng tập thể như [SkillClaw](https://github.com/AMAP-ML/SkillClaw), có thể đọc đối chiếu với tiểu mục SkillClaw ở phần sau của chương này.

<div align="center">
  <img src="./images/Extra10-figures/ultron-skill-hub.png" alt="Ultron Skill Hub" width="88%"/>
  <p><strong>Hình 11</strong> Ultron Skill Hub: Lối vào thống nhất cho kỹ năng chưng cất nội bộ và kỹ năng từ chỉ mục bên ngoài</p>
</div>

**Harness Hub** biến "nhân vật phiên hội thoại dùng xong là bỏ" thành tài sản có thể quản lý phiên bản: có thể phát hành **Agent profile** hoàn chỉnh (nhân vật, bộ nhớ, kỹ năng hợp nhất) dưới dạng blueprint có thể nhập bằng mã ngắn, đồng thời hỗ trợ **đồng bộ hai chiều** giữa workspace và máy chủ để thuận tiện cho việc tiếp tục trên nhiều thiết bị. Nó còn cung cấp rất nhiều **Soul presets** (tổ hợp vai trò, MBTI, cung hoàng đạo, v.v.) để lắp ráp tài nguyên workspace. Hình dưới đây tương ứng với bối cảnh tổ hợp và phát hành Harness trong bảng điều khiển.

<div align="center">
  <img src="./images/Extra10-figures/ultron-harness-hub.png" alt="Ultron Harness Hub" width="88%"/>
  <p><strong>Hình 12</strong> Ultron Harness Hub: Tổ hợp nhân vật với bộ nhớ, kỹ năng và phát hành blueprint có thể nhập</p>
</div>

Về phía tích hợp, chủ yếu hướng tới các nhà phát triển "đã có sẵn instance dịch vụ Ultron, chỉ cần kết nối Agent host vào"; trong Showcase có cung cấp các script nhập một lần dành cho OpenClaw, Hermes, Nanobot, v.v. Các bộ tối ưu kiểu **Darwin Skill**, vốn thiên về kiểm toán tài sản SKILL đơn máy và ratchet, cũng có thể hình thành sự bổ trợ với tầng bộ nhớ / kỹ năng dùng chung của Ultron: cái trước giữ vững khả năng giải thích của thay đổi, cái sau mở rộng bán kính tái sử dụng xuyên phiên, xuyên instance.

Nếu thu hẹp vấn đề thành một câu: Ultron cố gắng giảm nhẹ ba loại "nỗi đau" điển hình gắn với phiên hội thoại (Session-bound) (kinh nghiệm mất đi theo phiên, chi phí vấp lại cùng một lỗi bị khuếch đại theo số lượng instance, nhân vật không thể di chuyển). Nó không thay thế một framework Agent nào, mà chồng thêm lên trên đó một lớp hạ tầng tập thể **có thể truy xuất, có thể tiến hóa, có thể phân phối**.

### 7. OpenSpace: Dịch vụ tiến hóa bên ngoài và phả hệ phiên bản

Địa chỉ dự án: [https://github.com/HKUDS/OpenSpace](https://github.com/HKUDS/OpenSpace)

OpenSpace hướng tới hình thái "Agent host + công cụ tiến hóa kỹ năng gắn ngoài": con đường tích hợp phổ biến là cấu hình **OpenSpace MCP** vào các toolchain như Claude Code, Codex, OpenClaw, nanobot, Cursor, v.v., để phía OpenSpace đảm nhận việc khám phá kỹ năng, giám sát thực thi và đồng bộ lên đám mây, đồng thời lấy **self-evolving engine** làm câu chuyện sản phẩm: **thất bại có thể kích hoạt sửa chữa, mô hình thành công có thể được lắng đọng, chất lượng kỹ năng có thể được đo lường liên tục**.

Về cơ chế tiến hóa, nó coi kỹ năng là đối tượng lặp cải tiến liên tục, phân biệt ba tuyến chính: **FIX** (vá tại chỗ các hướng dẫn đã lỗi thời hoặc mất hiệu lực), **DERIVED** (phái sinh từ kỹ năng cha các biến thể tăng cường hoặc chuyên biệt), **CAPTURED** (trích xuất quy trình tái sử dụng hoàn toàn mới từ một lần thực thi thành công). Nguồn kích hoạt bao gồm **Post-Execution Analysis** (phân tích sau thực thi) sau khi nhiệm vụ kết thúc, kiểm tra **Tool Degradation** (suy giảm công cụ) khi tỷ lệ thành công của công cụ đi xuống, và **Metric Monitor** quét định kỳ các chỉ số sức khỏe, từ đó hình thành vòng lặp "thực thi → bằng chứng → bản vá hoặc phiên bản kỹ năng mới". Chiến lược viết lại thiên về **thay đổi tối thiểu ở cấp diff**, khi thất bại có thể tự động thử lại; phía phiên bản dùng **DAG** để ghi nhận phả hệ và so sánh, **Dashboard** cục bộ có thể duyệt biểu đồ tiến hóa kỹ năng và lịch sử thực thi. Registry (sổ đăng ký) trên đám mây hỗ trợ chế độ hiển thị công khai, nhóm hoặc riêng tư, và kết nối với tìm kiếm cộng đồng, nhập một lần.

Về chiều tập thể, nó nhất quán với câu chuyện của loại vòng lặp thứ ba: **sự cải tiến của một Agent đơn lẻ có thể lan tỏa sang các instance khác thông qua registry dùng chung**, tương đương với việc đặt hạ tầng tiến hóa bên ngoài host, để nhiều đầu cuối tái sử dụng cùng một thư viện kỹ năng và stack giám sát.

<div align="center">
  <img src="./images/Extra10-figures/openspace-framework.png" alt="OpenSpace framework" width="88%"/>
  <p><strong>Hình 13</strong> OpenSpace: Dịch vụ tiến hóa bên ngoài và phả hệ phiên bản</p>
</div>

Cách dùng điển hình là nhiều instance hoặc nhiều người dùng song song kết nối vào cùng một dịch vụ OpenSpace hoặc cùng một nhóm kỹ năng trên đám mây: quỹ đạo và bằng chứng thực thi hội tụ ở phía engine, phiên bản kỹ năng được lặp cải tiến theo tín hiệu chất lượng và giữ lại phả hệ, rồi được kéo về các host thông qua MCP hoặc CLI, tránh việc mỗi Agent phải tự duy trì một thư mục kỹ năng tĩnh mong manh.

### 8. SkillClaw: Để kỹ năng tiến hóa xuyên phiên, xuyên thiết bị, xuyên người dùng

Địa chỉ dự án: [https://github.com/AMAP-ML/SkillClaw](https://github.com/AMAP-ML/SkillClaw)  
Bài báo: [https://arxiv.org/abs/2604.08377](https://arxiv.org/abs/2604.08377)

SkillClaw hướng tới **collective skill evolution** (tiến hóa kỹ năng tập thể): lắng đọng cách sử dụng trong các phiên hội thoại thực tế thành **`SKILL.md`** có thể tái sử dụng, và chia sẻ cùng một vòng lặp tiến hóa giữa nhiều phiên của một người dùng, nhiều thiết bị, nhiều instance Agent cho đến giữa các thành viên trong nhóm, nhấn mạnh việc tự động tiến hóa, khử trùng lặp và kiểm định chất lượng xuyên phiên (có thể cấu hình **PRM** theo nhu cầu). Phía host tương thích với các luồng phổ biến như Hermes, OpenClaw, Codex, Claude Code, QwenPaw và bất kỳ API tương thích OpenAI nào.

Kiến trúc chia thành hai phần: **Client Proxy** là proxy API cục bộ, chặn **`/v1/chat/completions`** và **`/v1/messages`**, ghi lại sản phẩm phiên hội thoại và duy trì thư viện kỹ năng cục bộ mà không làm gián đoạn nhịp hội thoại, chỉ cần bật riêng nó là có thể hoàn thành việc tích hợp. **Evolve Server** (`evolve_server`) là tùy chọn: đọc dữ liệu phiên từ bộ lưu trữ dùng chung, tạo hoặc viết lại kỹ năng rồi ghi trở lại; engine tiến hóa hỗ trợ hai chế độ là **`workflow`** (pipeline LLM ba giai đoạn cố định Summarize → Aggregate → Execute) và **`agent`** (sửa kỹ năng trực tiếp dựa trên workspace OpenClaw). Client và server chỉ gặp nhau thông qua cùng một tầng lưu trữ (**thư mục cục bộ / OSS / S3**), do đó cá nhân có thể chạy proxy trước, rồi gắn dịch vụ tiến hóa trên máy đơn hoặc từ xa; trong kịch bản nhóm thì để nhiều client trỏ đến cùng một bộ lưu trữ và do một dịch vụ tiến hóa thống nhất tiêu hóa các quỹ đạo.

Hermes trên nhiều máy của cùng một người dùng, hoặc Agent host của nhiều thành viên trong cùng một nhóm, có thể ghi phiên hội thoại của mình vào bộ lưu trữ dùng chung, sau đó phía server hợp nhất, khử trùng lặp và phân phối phiên bản kỹ năng mới trở lại các instance — đây chính là hình dạng triển khai điển hình của "xuyên phiên, xuyên thiết bị, xuyên người dùng" trong loại vòng lặp thứ ba.

<div align="center">
  <img src="./images/Extra10-figures/skillclaw-framework.png" alt="SkillClaw architecture" width="88%"/>
  <p><strong>Hình 14</strong> SkillClaw: Framework tổng thể gồm Client Proxy, bộ lưu trữ dùng chung và Evolve Server</p>
</div>

Ví dụ: các đoạn gỡ lỗi React tích lũy ở môi trường gia đình, các đoạn vận hành Kubernetes tích lũy ở môi trường văn phòng, các đoạn phân tích log do OpenClaw chạy trên máy chủ, nếu thiếu tầng kỹ năng dùng chung thì sẽ bị cô lập riêng rẽ; sau khi tích hợp SkillClaw, chúng đi vào cùng một vòng lặp tiến hóa và kiểm định, rồi hồi lưu về các host khác nhau theo nhu cầu, tránh việc toàn bộ đội (fleet) phải trả lặp lại cùng một chi phí thử-sai.

## IV. Tự sửa đổi tham số, mã nguồn hoặc workflow: Để bản thân chiến lược thay đổi

Loại cuối cùng là vòng lặp mạnh nhất và cũng nặng nhất: hệ thống không chỉ thay đổi ngữ cảnh, bộ nhớ hoặc kỹ năng, mà bắt đầu thay đổi tham số mô hình, mã nguồn Agent, cấu trúc liên kết (topology) workflow hoặc thuật toán ứng viên.

Giới hạn trên của loại hệ thống này rất cao, vì nó có thể thực sự thay đổi bản thân chiến lược; nhưng rủi ro cũng lớn nhất, vì một khi bộ đánh giá có lỗ hổng, thiết kế phần thưởng (reward) không hợp lý, sandbox không đầy đủ, hệ thống có thể học cách "chiều theo bài đánh giá" thay vì thực sự trở nên mạnh hơn.

### 9. OpenClaw-RL: Từ phản hồi hội thoại thực tế đến RL trực tuyến

Địa chỉ dự án: [https://github.com/Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL)  
Báo cáo kỹ thuật: [https://arxiv.org/abs/2603.10165](https://arxiv.org/abs/2603.10165)

OpenClaw-RL chồng hai tuyến chính trong cùng một stack: **Track 1 (Personal Agent)** nhúng mô hình chiến lược tự host vào [OpenClaw](https://openclaw.ai), đối ngoại giữ API tương thích OpenAI, chặn các cuộc hội thoại đa lượt trực tuyến và chuyển các tương tác thành tín hiệu huấn luyện, việc phục vụ (Serving) và tối ưu nền không chặn lẫn nhau; **Track 2 (General Agentic RL)** thì mở rộng hạ tầng RL bất đồng bộ sang các môi trường nặng hơn như terminal, GUI, SWE, gọi công cụ, nâng cao quy mô rollout song song.

Vòng lặp khép kín được chia theo README thành bốn vòng lặp bất đồng bộ: **Agent serving**, **rollout collection**, **PRM / Judge evaluation**, **policy training** (bao gồm **LoRA**). Phía rollout chia các tin nhắn hội thoại thành **main-line** (có thể huấn luyện) và **side**, và coi phản hồi từ người dùng, môi trường hoặc công cụ ở nhịp kế tiếp như một **next-state** tự nhiên; Judge / PRM chấm điểm bất đồng bộ, có thể bỏ phiếu đa số theo nhu cầu rồi mới đưa vào hàng đợi. **Binary RL (GRPO)** dùng mô hình phần thưởng quá trình (process reward model) kết hợp với phản hồi trạng thái kế tiếp để tạo ra phần thưởng vô hướng (scalar reward) theo từng bước; **OPD (On-Policy Distillation)** dùng văn bản hindsight (nhìn lại) để xây dựng giáo viên tăng cường, lấy khoảng cách log-probability giữa giáo viên và học sinh để đưa ra tín hiệu định hướng; ngoài ra còn có công thức **Combine / Hybrid** kết hợp cả hai. Việc triển khai có thể chọn nhiều GPU cục bộ, [Tinker](https://thinkingmachines.ai/tinker/), [Fireworks AI](https://fireworks.ai/), v.v.; cũng có thể thông qua phần mở rộng chính thức để gắn đầu RL vào OpenClaw của riêng mình.

<div align="center">
  <img src="./images/Extra10-figures/openclaw-rl-framework.png" alt="OpenClaw-RL asynchronous loops" width="88%"/>
  <p><strong>Hình 15</strong> OpenClaw-RL: Kiến trúc bất đồng bộ Serving · Rollout · PRM/Judge · Training</p>
</div>

Loại hệ thống này gần nhất với "vừa chat vừa huấn luyện", nhưng một khi phản hồi của người dùng thực tế đi vào đường dẫn gradient, thì bắt buộc phải song song thực hiện sự đồng ý có hiểu biết (informed consent), chu kỳ lưu trữ dữ liệu, ẩn danh hóa quyền riêng tư, chống lợi dụng phần thưởng (reward gaming) và rollback an toàn; nếu không, các chỉ số kỹ thuật rất dễ bị triệt tiêu bởi rủi ro tuân thủ và pháp lý.

### 10. Agent Lightning: Tách rời việc thực thi Agent và huấn luyện RL

Địa chỉ dự án: [https://github.com/microsoft/agent-lightning](https://github.com/microsoft/agent-lightning)  
Bài báo: [https://arxiv.org/abs/2508.03680](https://arxiv.org/abs/2508.03680)  
Tài liệu: [https://microsoft.github.io/agent-lightning/](https://microsoft.github.io/agent-lightning/)

Agent Lightning thu gọn chuỗi huấn luyện thành ba khối: phía **Agent / Tracer** chỉ cần can thiệp tối thiểu (gọi tường minh `agl.emit_xxx()` hoặc để tracer tự động thu thập prompt, lời gọi công cụ và phần thưởng), các sự kiện được chuẩn hóa thành **span** có cấu trúc; **LightningStore** thống nhất lưu đệm định nghĩa nhiệm vụ, ảnh chụp tài nguyên (resource snapshot) và quỹ đạo, tương đương với một bề mặt đọc ghi duy nhất ở phía huấn luyện; **Trainer + Algorithm** đọc span, tạo ra tài nguyên đã cập nhật (như prompt đã tinh chỉnh hoặc trọng số chiến lược), rồi hồi lưu về phía suy luận (inference). Vị trí thuật toán (algorithm slot) vừa có thể gắn RL, vừa có thể gắn tối ưu prompt tự động, tinh chỉnh có giám sát (supervised fine-tuning), v.v. theo cùng một hợp đồng span.

Tài liệu chính thức nhấn mạnh rằng nó có thể cùng tồn tại với các công cụ điều phối (orchestration) như LangChain, OpenAI Agent SDK, AutoGen, CrewAI, Microsoft Agent Framework, v.v., cũng có thể dùng trên chuỗi gọi OpenAI thuần không qua framework; trong kịch bản đa tác tử có thể **chọn lọc** chỉ tính gradient cho một số vai trò, mà không cần viết lại toàn bộ topology thành một đồ thị huấn luyện duy nhất.

<div align="center">
  <img src="./images/Extra10-figures/agent-lightning-architecture.svg" alt="Agent Lightning architecture" width="90%"/>
  <p><strong>Hình 16</strong> Agent Lightning: LightningStore tách rời thực thi trực tuyến và thuật toán ngoại tuyến</p>
</div>

So với OpenClaw-RL: cái trước bám sát OpenClaw Serving + stack huấn luyện RL bất đồng bộ; Agent Lightning giống một "xe buýt quỹ đạo" đa dụng hơn, với mục tiêu là **Trainer / Algorithm** có thể cắm-rút (pluggable), phía host chú trọng giữ lại ngữ nghĩa của framework vốn có.

## Các vấn đề kỹ thuật then chốt của hệ thống tự tiến hóa

Bất kể là loại vòng lặp nào, khi thực sự triển khai đều sẽ gặp bốn vấn đề chung.

### 1. Bộ đánh giá quan trọng hơn bộ sinh

Ảo tưởng nguy hiểm nhất của hệ thống tự tiến hóa là: "Chỉ cần biết tự động sửa, thì sẽ tự động trở nên tốt hơn." Thực tế lại hoàn toàn ngược lại. Sự sửa đổi tự động mà không có bộ đánh giá chỉ là tự động tạo ra sự bất định.

Ratchet của Darwin Skill, worker xác thực của SkillClaw, cổng nâng cấp theo điểm cấu trúc của Ultron, PRM/Judge của OpenClaw-RL, phân tích sau thực thi và kiểm tra chỉ số của OpenSpace, tất cả đều cho thấy cùng một đạo lý: **cốt lõi của tự tiến hóa không phải là thay đổi, mà là chứng minh rằng thay đổi không làm hệ thống trở nên tệ hơn.**

### 2. Khả năng rollback là hạ tầng của tự tiến hóa

Ưu điểm của tầng kỹ năng là tự nhiên có thể quản lý phiên bản: `SKILL.md` có thể diff, có thể git commit, có thể rollback. Ngược lại, cập nhật ở tầng tham số tuy có tiềm năng lớn hơn, nhưng việc giải thích và rollback lại nặng nề hơn. Một nguyên tắc thực tế là:

```
能先在记忆层解决，就不要急着改技能；
能先在技能层解决，就不要急着改代码；
能先在代码/工作流层解决，就不要急着在线更新权重。
```

(Dịch nghĩa: Nếu có thể giải quyết ở tầng bộ nhớ trước, thì đừng vội sửa kỹ năng; nếu có thể giải quyết ở tầng kỹ năng trước, thì đừng vội sửa mã nguồn; nếu có thể giải quyết ở tầng mã nguồn/workflow trước, thì đừng vội cập nhật trọng số trực tuyến.)

### 3. Chia sẻ kinh nghiệm cần có quản trị

Ultron, OpenSpace, SkillClaw đều đưa kinh nghiệm cá nhân vào tầng dùng chung. Việc chia sẻ sẽ mang lại hiệu ứng mạng lưới (network effect), nhưng cũng mang lại rủi ro ô nhiễm. Một kỹ năng sai, một bản ghi bộ nhớ chứa thông tin riêng tư, một đoạn hội thoại bị nhiễm **prompt injection**, nếu đi vào thư viện dùng chung, đều có thể ảnh hưởng đến Agent của cả nhóm.

Do đó, vòng lặp trí tuệ tập thể cần ít nhất bốn năng lực mặc định: phân tầng quyền hạn, ẩn danh hóa PII, xác thực ứng viên, kiểm toán phiên bản.

### 4. An ninh chuỗi cung ứng kỹ năng không thể để sau mới vá

Agent Skills đã trở thành định dạng đóng gói năng lực để tái sử dụng xuyên hệ sinh thái. Một kỹ năng có thể bao gồm `SKILL.md`, script, tài liệu tham khảo, template, phụ thuộc từ xa (remote dependency) và chỉ thị thực thi. Nó vừa giống tài liệu, vừa giống gói phần mềm, lại còn có thể ảnh hưởng đến bộ nhớ dài hạn và hành vi gọi công cụ.

Điều này có nghĩa là chợ kỹ năng (skill marketplace) và việc chia sẻ kỹ năng không thể chỉ xem "viết có hay không", mà còn phải xem: có đọc các đường dẫn nhạy cảm hay không, có thực thi lệnh nguy hiểm hay không, có tải script từ xa hay không, có ghi secret vào đầu ra hay không, có cố gắng làm ô nhiễm các kỹ năng hoặc bộ nhớ khác hay không.

## Lộ trình thực hành: Từ vòng lặp nhẹ đến tự tiến hóa mạnh

Nếu bạn muốn đưa self-evolution vào dự án Agent của mình, có thể tiến hành theo bốn giai đoạn.

### Giai đoạn một: Làm vòng lặp ngữ cảnh tích hợp sẵn trước

Mục tiêu là để Agent có được năng lực phát triển cơ bản nhất: biết ghi nhận sở thích của người dùng, biết tổng kết kinh nghiệm thất bại, biết truy xuất phiên hội thoại lịch sử, biết lắng đọng nhiệm vụ phức tạp thành các bước thao tác.

Dự án có thể tham khảo là [Hermes Agent](https://github.com/NousResearch/hermes-agent). Trọng tâm của giai đoạn này không phải là nhồi lịch sử vô hạn trở lại prompt, mà là để ngữ cảnh dự án, lịch sử phiên hội thoại, trạng thái công cụ và thông tin tự tham chiếu dần dần hình thành một cơ chế ổn định.

### Giai đoạn hai: Lắng đọng kinh nghiệm thành Skill

Khi kinh nghiệm bắt đầu xuất hiện lặp lại, thì nên nâng cấp từ tầng bộ nhớ lên tầng kỹ năng. Kỹ năng có cấu trúc hơn bộ nhớ, và cũng dễ di chuyển sang các Agent khác hơn.

Dự án có thể tham khảo là [Darwin Skill](https://github.com/alchaincyf/darwin-skill). Nếu bạn đã đang sử dụng các host hỗ trợ `SKILL.md` như OpenClaw, Nanobot, Hermes, Codex hoặc Claude Code, thì vòng lặp tài sản hóa kỹ năng thường là bước có tỷ suất đầu tư - lợi nhuận cao nhất.

### Giai đoạn ba: Đưa vào tầng dùng chung và trí tuệ tập thể

Khi nhiều người dùng, nhiều thiết bị, nhiều Agent đều đang tích lũy kinh nghiệm, thì cần một tầng dùng chung để khử trùng lặp, xác thực, hợp nhất và phân phối.

Các dự án có thể tham khảo bao gồm [Ultron](https://github.com/modelscope/ultron), [OpenSpace](https://github.com/HKUDS/OpenSpace) và [SkillClaw](https://github.com/AMAP-ML/SkillClaw). Ở giai đoạn này cần thiết kế trước các yếu tố quyền riêng tư, quyền hạn và kiểm toán.

### Giai đoạn bốn: Thận trọng thử nghiệm tự sửa đổi tham số, mã nguồn hoặc workflow

Chỉ khi bộ đánh giá, sandbox, quản trị phiên bản và cơ chế rollback đã đủ trưởng thành, mới phù hợp để thử các vòng lặp nặng hơn như [OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL), [Agent Lightning](https://github.com/microsoft/agent-lightning) — những loại kết nối tín hiệu đánh giá vào việc cập nhật tham số hoặc chiến lược có cấu trúc.

Nếu bạn chỉ muốn trợ lý cá nhân hiểu bạn hơn, có lẽ không cần đến RL trực tuyến; nếu bạn muốn Agent tối ưu việc sử dụng công cụ, thao tác GUI, sửa lỗi SWE hoặc lựa chọn workflow trong một lượng lớn nhiệm vụ thực tế, thì tự sửa đổi ở tầng tham số hoặc workflow mới dần trở nên cần thiết.

## Tổng kết

Agent Self-Evolution không phải là một công nghệ đơn lẻ, mà là một tập hợp các vòng lặp khép kín đi từ nhẹ đến nặng:

```
上下文/记忆自进化
  → 技能资产自进化
    → 群体经验自进化
      → 参数、代码或工作流自进化
```

(Dịch nghĩa: Tự tiến hóa ngữ cảnh/bộ nhớ → Tự tiến hóa tài sản kỹ năng → Tự tiến hóa kinh nghiệm tập thể → Tự tiến hóa tham số, mã nguồn hoặc workflow.)

**Hermes Agent** và **Agent Zero** đại diện cho vòng lặp ngữ cảnh tích hợp sẵn; **Darwin Skill**, **JiuwenClaw**, **EvoSkill** đại diện cho vòng lặp tài sản hóa kỹ năng; **Ultron**, **OpenSpace**, **SkillClaw** đại diện cho việc lắng đọng và phân phối ở phía trí tuệ tập thể; **OpenClaw-RL** kết nối tương tác trực tuyến vào RL hoặc OPD, còn **Agent Lightning** thì dùng LightningStore để tách rời việc quan sát thực thi với Trainer có thể cắm-rút.

Đối với đa số các nhóm kỹ thuật, lộ trình chắc chắn nhất không phải là bắt tay ngay vào cập nhật trọng số trực tuyến, mà là trước tiên làm tốt vòng lặp bộ nhớ và kỹ năng "có thể giải thích, có thể đánh giá, có thể rollback". Điều thực sự hạn chế việc triển khai tự tiến hóa thường không phải là mô hình chưa đủ thông minh, mà là việc đánh giá, sandbox, quyền hạn, quản trị phiên bản và an ninh chuỗi cung ứng vẫn chưa đủ vững chắc.

## Tài liệu tham khảo

[1] Hermes Agent. [https://github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)  
[2] Agent Zero. [https://github.com/agent0ai/agent-zero](https://github.com/agent0ai/agent-zero)  
[3] Darwin Skill. [https://github.com/alchaincyf/darwin-skill](https://github.com/alchaincyf/darwin-skill)  
[4] JiuwenClaw. [https://github.com/openJiuwen-ai/jiuwenclaw](https://github.com/openJiuwen-ai/jiuwenclaw)  
[5] EvoSkill. [https://github.com/sentient-agi/EvoSkill](https://github.com/sentient-agi/EvoSkill)  
[6] Ultron. [https://github.com/modelscope/ultron](https://github.com/modelscope/ultron)  
[7] OpenSpace. [https://github.com/HKUDS/OpenSpace](https://github.com/HKUDS/OpenSpace)  
[8] SkillClaw. [https://github.com/AMAP-ML/SkillClaw](https://github.com/AMAP-ML/SkillClaw)  
[9] OpenClaw-RL. [https://github.com/Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL)  
[10] Agent Lightning. [https://github.com/microsoft/agent-lightning](https://github.com/microsoft/agent-lightning)
