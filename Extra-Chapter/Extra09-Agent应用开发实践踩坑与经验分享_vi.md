# Thực tiễn phát triển ứng dụng Agent: những cái bẫy đã gặp và kinh nghiệm chia sẻ

Sau khi học xong giáo trình Hello-Agents, nhiệm vụ cuối cùng là đồ án tốt nghiệp. Dùng kiến thức đã học để tự tay làm một ứng dụng Agent, đúng lúc đó Code Agent đang cực kỳ hot, Cursor, Claude Code, Codex... các hãng đều đang đẩy mạnh sản phẩm của mình. Nghĩ bụng đã muốn luyện tay nghề thì không bằng tái hiện lại một Code Agent, tự tay làm lại một lần, mới thực sự hiểu được vì sao những sản phẩm đó dễ dùng đến vậy, và chúng thực sự đã làm đúng những gì về mặt kỹ thuật.

Thế là dự án này ra đời.
Repo code Code Agent dựa trên framework Hello-Agents: https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/YYHDBL-HelloCodeAgentCli

Repo code MyCodeAgent sau khi tái cấu trúc: https://github.com/YYHDBL/MyCodeAgent.git

Bài viết này không phải là một tutorial, mà là ghi chép lại những cái bẫy mình đã gặp, những đường vòng mình đã đi, và cuối cùng đã giải quyết như thế nào trong quá trình làm dự án Code Agent này.

---

## Mục lục

- [Chương 1: Đọc quá nhiều best practice, lại càng rơi vào cái bẫy lớn đầu tiên](#chương-1-đọc-quá-nhiều-best-practice-lại-càng-rơi-vào-cái-bẫy-lớn-đầu-tiên)
- [Chương 2: Một sự cố lệnh pipe — lần đầu tiên tôi thấy "không thể chẩn đoán" chết chóc đến mức nào](#chương-2-một-sự-cố-lệnh-pipe--lần-đầu-tiên-tôi-thấy-không-thể-chẩn-đoán-chết-chóc-đến-mức-nào)
- [Chương 3: Vùng Goldilocks của thiết kế công cụ](#chương-3-vùng-goldilocks-của-thiết-kế-công-cụ)
- [Chương 4: Prompt không phải là câu thần chú ma thuật, mà là control plane của Agent](#chương-4-prompt-không-phải-là-câu-thần-chú-ma-thuật-mà-là-control-plane-của-agent)
- [Chương 5: Context không phải là vấn đề dung lượng bộ nhớ, mà là vấn đề điều phối sự chú ý](#chương-5-context-không-phải-là-vấn-đề-dung-lượng-bộ-nhớ-mà-là-vấn-đề-điều-phối-sự-chú-ý)
- [Chương 6: Observability biến hộp đen thành hộp kính](#chương-6-observability-biến-hộp-đen-thành-hộp-kính)
- [Chương 7: Phương pháp luận tổng quát rút ra từ một dự án](#chương-7-phương-pháp-luận-tổng-quát-rút-ra-từ-một-dự-án)


---

# Chương 1: Đọc quá nhiều best practice, lại càng rơi vào cái bẫy lớn đầu tiên

Khi mới bắt tay viết code, tôi đã tham khảo rất nhiều thực tiễn thiết kế Agent trong ngành. Ví dụ như "Bài học kinh nghiệm về Context Engineering" do đội ngũ Manus chia sẻ, và "Building agents with the Claude Agent SDK" chính thức của Anthropic. Nhìn những ông lớn hàng đầu này chia sẻ "best practice" không giấu giếm, tôi nghĩ bụng: dù sao giờ đã có Claude Code, cứ để AI giúp mình triển khai toàn bộ những khái niệm cao cấp này là xong?

Thế là, tôi không suy nghĩ nhiều mà chất đống đủ loại thiết kế trông có vẻ tao nhã: hệ thống bộ nhớ đa tầng (Memory System), context engineering phức tạp, hệ thống đa Agent (Multi-Agent)... Phải nói là Claude Code đúng là xịn, rất nhanh đã giúp tôi sinh ra một đống code logic phức tạp.

## Khởi đầu thảm họa

Nhưng khi tôi hào hứng chạy phiên bản test đầu tiên, thực tế đã tát thẳng vào mặt tôi: toàn bộ hệ thống tệ hại không chịu nổi.

Đối mặt với một nhu cầu chỉnh sửa cực kỳ đơn giản, Agent gọi điên cuồng bảy tám loại tool, trải qua mấy vòng "não trái đấu não phải". Cuối cùng, tôi chỉ thu hoạch được một đoạn code tàn khuyết chẳng chạy nổi, cùng với một hóa đơn Token vượt chi nghiêm trọng.

Nhìn đầy màn hình lỗi, tôi mới nhận ra: phát triển Agent rất khác so với phát triển phần mềm truyền thống.

Trước đây khi làm backend truyền thống, chúng ta quen vẽ sơ đồ kiến trúc trước, rồi mới viết code. Sơ đồ đủ tao nhã thì hệ thống sẽ vững chắc. Đó là bản năng của lập trình viên.

Nhưng phát triển Agent thì khác. Bạn đang làm việc với một mô hình lớn, bản thân nó mang tính xác suất — cùng một input, mỗi lần có thể cho ra output hoàn toàn khác nhau.

Tôi đã cưỡng ép chồng chất lên nền móng không chắc chắn này một bộ kiến trúc phức tạp mà chính mình cũng chưa hề kiểm chứng. Multi-Agent, Plan-and-Execute... những thiết kế này đan xen lẫn nhau, khiến sự bất định nhân lên gấp bội.

Kết quả là: kiến trúc phức tạp không những không đỡ được đáy, mà vì luồng trạng thái quá nhiều, tool giao thoa quá phức tạp, khiến model sai càng thêm sai. Lỗi truyền qua lại giữa các thành phần, tôi còn không biết bắt đầu gỡ lỗi từ đâu.

Những "best practice" của các ông lớn đó dĩ nhiên là tốt, nhưng tôi đã bỏ qua một điều: những kiến trúc phức tạp đó là kết quả tiến hóa sau khi họ đã gặp vô số cái bẫy, tiêu tốn lượng token khổng lồ, chứ không phải là điểm khởi đầu cho người mới.

## Đập đi làm lại

Nhìn đống code mà ngay cả đọc file đơn giản cũng có thể rơi vào vòng lặp vô hạn, tôi đã đưa ra một quyết định trái với tổ tiên — xóa sạch, đập đi làm lại.

Theo nguyên tắc "Less is more", tôi tái sử dụng trực tiếp phần cốt lõi cơ bản nhất của Hello-Agent, chạy thông luồng ngắn nhất trước. Các thành phần cốt lõi được tinh giản chỉ còn lại mấy khối này:

| Thành phần | Trách nhiệm cốt lõi |
|------|----------|
| ReActAgent | Điều khiển vòng lặp nhận thức cơ bản Thought → Action → Observation |
| ToolRegistry | Chịu trách nhiệm đăng ký và phân phối gọi tool |
| ContextBuilder | Ghép nối quy tắc hệ thống, lịch sử và bằng chứng môi trường |
| TerminalTool | Thực thi lệnh thực tế trong repo code mục tiêu |
| Message | Cấu trúc dữ liệu tin nhắn hội thoại thống nhất |

Về mặt code, tôi không áp dụng bất kỳ design pattern hoa mỹ nào, mà trực tiếp gộp chúng lại một cách thô bạo trong `code_agent.py`:

```python
self.terminal_tool = TerminalTool(
    workspace=str(self.paths.repo_root),
    timeout=60,
    confirm_dangerous=True,
    default_shell_mode=True,  
)
self.registry = ToolRegistry()
self.registry.register_tool(self.terminal_tool)
```

Nhìn phiên bản V0 đơn sơ này chạy thành công tác vụ cực đơn giản đầu tiên trong terminal, tôi thở phào nhẹ nhõm.

![image-20260225143343219](images/Extra09-figures/1.png)

Chỉ cần có thể hoàn thành hội thoại cơ bản nhất và gọi tool, chúng ta đã có thể kiểm chứng và lặp tối ưu trong môi trường tác vụ thực tế.

Chạy được trước quan trọng hơn là tiến thêm một bước.


---

# Chương 2: Một sự cố lệnh pipe — lần đầu tiên tôi thấy "không thể chẩn đoán" chết chóc đến mức nào

Sau khi V0 chạy được, tôi bắt đầu nới lỏng ràng buộc đối với tool. Vì kiến trúc đã được tinh giản, để model tự do kết hợp lệnh chắc không sao chứ?

Thế là tôi bật đèn xanh cho Terminal Tool: `default_shell_mode=True`. Điều này có nghĩa là model không chỉ có thể chạy một lệnh đơn, mà còn có thể viết pipe, redirect, sub-command — giống như con người.

## Diễn biến sự cố

Hôm đó tôi đưa ra một yêu cầu đơn giản: "Giúp tôi tìm định nghĩa của hàm `process_data`".

Model nhanh chóng đưa ra một lệnh trông khá chuyên nghiệp:

```bash
rg -n "def process_data" src/ | grep -v test | sed -n '1,50p'
```

Combo quen thuộc của kỹ sư lâu năm: tìm định nghĩa hàm trước, lọc bỏ file test, rồi lấy 50 dòng đầu. Không có vấn đề gì.

Nhưng kết quả thực thi lại trống rỗng. Agent nhìn thấy kết quả trống này, khựng lại một chút, rồi bắt đầu khắc phục:

**Lần thử lại 1:**
```
Thought: Có vẻ không tìm thấy, có thể phạm vi tìm kiếm quá hẹp. Để tôi mở rộng phạm vi tìm kiếm.
Action: rg -n "process_data" | head -100
```

Vẫn trống.

**Lần thử lại 2:**
```
Thought: Có thể tên hàm nhớ nhầm, thử khớp mờ xem sao.
Action: rg -n "process.*data" src/ --type py | grep "def "
```

Vẫn trống.

**Lần thử lại 3:**
```
Thought: Có vẻ hàm này có thể không tồn tại, hoặc tôi tìm sai thư mục. Để tôi liệt kê cấu trúc thư mục trước.
Action: find src/ -name "*.py" | xargs grep -l "process_data"
```

Kết quả vẫn trống. Sau ba lần thử, nó bỏ cuộc, nói với tôi: "Tôi không tìm thấy định nghĩa hàm `process_data` trong repo, có thể tên hàm bị sai, hoặc nó không nằm trong file Python."

Nhưng tôi tự tay vào repo xem thì hàm đó rõ ràng nằm ở dòng 42 của `src/utils/helpers.py`.

## Quá trình điều tra

Tôi copy lệnh đó vào terminal tự chạy, phát hiện `rg` báo lỗi:

```bash
$ rg -n "def process_data" src/ | grep -v test | sed -n '1,50p'
Error: path 'src/' does not exist
```

Hóa ra thư mục làm việc khi tôi khởi động Agent không phải là thư mục gốc của project, mà là một thư mục con bên dưới project. `src/` không tồn tại so với thư mục hiện tại, rg báo lỗi rồi thoát ngay.

Nhưng ở phía Agent, thông báo lỗi đã bị pipe nuốt mất. Vì lệnh dùng `|`, output lỗi của rg không được truyền đến stdout, mà bị pipe chuyển hướng vào input của lệnh tiếp theo. grep nhận được input trống, tự nhiên cho ra output trống; sed cũng trống.

**Lỗi đã bị dẹp phẳng trong chuỗi liên kết.** Agent chỉ nhìn thấy một chuỗi rỗng, nó hoàn toàn không biết upstream đã thất bại.

Điều đáng ngại nhất là, model đã đưa ra phán đoán hoàn toàn sai dựa trên thông báo lỗi này. Nó tưởng là "thực sự không tìm thấy", thế là bắt đầu đủ loại biện pháp khắc phục: đổi từ khóa tìm kiếm, đổi thư mục, thậm chí nghi ngờ tôi có nhớ nhầm tên hàm hay không. Những hành động này đều dựa trên một phán đoán sai, tiêu tốn vô ích lượng lớn token.

![image-20260225143439420](images/Extra09-figures/2.png)

## Hướng khắc phục sai lầm lúc đó

Phản ứng đầu tiên của tôi là: tool Bash quá nguy hiểm, phải thêm giới hạn.

Thế là tôi viết một đống code kiểm tra an toàn:

```python
SHELL_META_TOKENS = ["|", "||", "&&", ";", ">", ">>", "<", "$(", "`"]
DANGEROUS_BASE_COMMANDS = {"rm", "chmod", "mv", "dd"}

def validate_command(cmd):
    # 检查是否包含管道或重定向
    for token in SHELL_META_TOKENS:
        if token in cmd:
            return False, f"包含非法字符: {token}"
    
    # 检查基础命令是否在白名单
    base_cmd = cmd.split()[0]
    if base_cmd not in ALLOWED_COMMANDS:
        return False, f"命令 {base_cmd} 不在白名单"
    
    # 检查危险命令
    if base_cmd in DANGEROUS_BASE_COMMANDS:
        return False, "危险命令，禁止执行"
    
    return True, "OK"
```

Nhưng chẳng mấy chốc tôi phát hiện, shell quá linh hoạt. Bạn cấm `|`, nó có thể dùng sub-command thay thế `$(...)`; bạn cấm `>`, nó có thể dùng `tee`; bạn cấm `rm`, nó có thể dùng `> file` để xóa trống file.

Patch vá càng nhiều, code càng viết càng dài, nhưng vấn đề gốc rễ — "rốt cuộc bước nào thất bại" — vẫn còn đó.

Ngay cả khi tôi đã chặn toàn bộ pipe và redirect, chỉ cho phép lệnh đơn giản nhất, vấn đề vẫn còn:

```bash
rg "pattern" src/
```

Nếu trả về trống, tôi vẫn không biết là "repo thực sự không có", hay "rg vì đường dẫn sai nên không thực thi". Model vẫn không thể sửa lỗi một cách có mục tiêu.

## Định vị nguyên nhân gốc rễ

Sau này tôi mới nghĩ thông suốt, nguyên nhân gốc rễ của việc này không phải là "lệnh quá nguy hiểm", mà là **không thể chẩn đoán**.

Cụ thể có ba vấn đề:

**Thứ nhất, nhiều bước bị nhồi vào một Action.** Pipe đóng gói mấy bước logic lại với nhau, trạng thái trung gian mất sạch. Agent chỉ có thể thấy kết quả cuối cùng, không thấy được quá trình thực thi.

**Thứ hai, tín hiệu quan sát chỉ có một trạng thái cuối.** Thành công, thất bại, kết quả trống, tất cả lẫn lộn với nhau. Model không phân biệt được "thực sự không tìm thấy" và "quá trình tìm kiếm bị lỗi".

**Thứ ba, model không thể sửa lỗi có mục tiêu.** Nó không biết `rg`, `grep`, `sed` cái nào gặp vấn đề, bước tiếp theo chỉ có thể đoán mò. Thử lại không phải dựa trên "sửa lỗi", mà dựa trên "cược may rủi".

Cho model mức tự do cao hơn, không phải là đang nâng trần năng lực, mà là đang khuếch đại sự bất định. Nó đúng là có thể viết ra lệnh "thông minh" hơn, nhưng một khi sai, ngay cả bạn cũng không thể điều tra được nó đã "khôn ngoan quá hóa dở" ở bước nào.

## Cách làm hiện tại

Sau này tôi trực tiếp hạ cấp Bash — không phải xóa bỏ, mà là xác định rõ vị trí của nó: chỉ xử lý những nhu cầu lẻ tẻ mà tool nguyên tử không bao phủ được, không đi theo luồng chính.

Các thao tác tần suất cao đều được tách thành tool nguyên tử:

| Tool | Chức năng | Định dạng trả về |
|------|------|----------|
| LS | Liệt kê thư mục | `{status, data: {entries}, text}` |
| Glob | Tìm file theo tên | `{status, data: {paths}, text}` |
| Grep | Tìm kiếm theo nội dung | `{status, data: {matches}, text}` |
| Read | Đọc kèm số dòng | `{status, data: {content}, text}` |

Mỗi tool đều có mã trạng thái rõ ràng:
- `success`: tác vụ hoàn thành, kết quả nằm trong data
- `partial`: tác vụ hoàn thành nhưng nội dung bị cắt bớt
- `error`: tác vụ thất bại, error chứa mã lỗi cụ thể

Ví dụ Glob không tìm thấy file:
```json
{
  "status": "success",
  "data": {"paths": []},
  "text": "No files matching '*.xyz' found"
}
```

Đường dẫn không tồn tại:
```json
{
  "status": "error",
  "error": {"code": "NOT_FOUND", "message": "Path 'src/' does not exist"}
}
```

Model có thể phân biệt rõ ràng "thực sự không có" và "bị lỗi".

Ràng buộc cứng của Bash cũng được xác định rõ:
- Cấm đọc/tìm/liệt kê: `ls`/`cat`/`head`/`grep`/`find`/`rg` đã có tool chuyên dụng
- Cấm tương tác: vim, nano, top, ssh
- Cấm mạng (mặc định): curl/wget bị cấm
- Danh sách đen: rm -rf /, sudo/su, mkfs/fdisk

Sau khi làm như vậy, việc debug trở nên đơn giản hơn nhiều. Có vấn đề xem log là biết bước nào:
- Glob trả về mảng rỗng → đúng là không có file này
- Glob trả về NOT_FOUND → đường dẫn sai
- Grep trả về timeout → phạm vi tìm kiếm quá lớn

Model cũng có thể dựa trên mã lỗi cụ thể để quyết định bước tiếp theo: đường dẫn sai thì đổi đường dẫn, timeout thì thu hẹp phạm vi, thực sự không tìm thấy thì báo cho người dùng.

## Kết luận chương này

**Khả năng chẩn đoán là tiền đề của khả năng phục hồi.**

Nếu không biết chỗ nào hỏng, thì không sửa được. Nếu không biết thất bại xảy ra ở bước nào, thì không thể sửa lỗi có mục tiêu.

Trong phát triển Agent, cho model khả năng tự do kết hợp lệnh, nghe thì rất hay, nhưng thực tế là đang tạo ra hộp đen. Những lệnh pipe trông có vẻ hiệu quả, dẹp phẳng thông tin lỗi thành những kết quả rỗng không thể phân biệt, khiến model càng chạy càng xa trên con đường sai.

Tool nguyên tử tuy các bước rườm rà, nhưng mỗi bước đều có input, output, trạng thái rõ ràng. Có vấn đề, bạn định vị được; model sai, bạn sửa được.

**Khả năng kiểm soát quan trọng hơn nhiều so với hoàn thành tác vụ trong một lần.**


---

# Chương 3: Vùng Goldilocks của thiết kế công cụ — không phải càng tự do càng tốt, cũng không phải càng vụn càng tốt

Sau chương 3, tôi bắt đầu tách tool ra. Chế độ vạn năng quản lý mọi thứ của Terminal Tool đúng là có vấn đề, sau khi tách thành tool nguyên tử, việc debug trở nên rõ ràng hơn nhiều.

Nhưng tôi nhanh chóng lại rơi vào một cái bẫy mới: **tách quá vụn**.

## Tôi đã trải qua cả hai cực đoan

### Cực đoan A: Tool vạn năng

Cực đoan đầu tiên bạn đã thấy rồi. Một Terminal Tool làm được mọi thứ: pipe, redirect, sub-command, biến môi trường — hoàn toàn mở.

Lúc đó tôi nghĩ, LLM thông minh như vậy, cho nó đủ mức tự do, chắc nó sẽ có thể thao tác như một kỹ sư. Lệnh tổ hợp kiểu `rg | grep | sed` hiệu quả rất cao.

Kết quả bạn cũng biết rồi: lỗi bị pipe nuốt mất, model đoán mò thử lại, token chảy ào ào, vấn đề vẫn chưa được giải quyết.

### Cực đoan B: Nguyên tử hóa quá mức

Sau khi nhận ra tool vạn năng có vấn đề, tôi lại đi về một cực đoan khác: tách mỗi điểm chức năng thành một tool độc lập, theo đuổi sự nguyên tử hóa cực đoan.

Lúc đó danh sách tool của tôi trông như thế này:
- `ListDir`: Liệt kê nội dung thư mục
- `ListDirRecursive`: Liệt kê thư mục đệ quy
- `FindByName`: Tìm theo tên file
- `FindByPattern`: Tìm theo ký tự đại diện
- `SearchExact`: Tìm kiếm khớp chính xác
- `SearchRegex`: Tìm kiếm khớp regex
- `SearchFuzzy`: Tìm kiếm khớp mờ
- `ReadLines`: Đọc theo phạm vi dòng chỉ định
- `ReadOffset`: Đọc theo offset byte chỉ định
- `ReadFull`: Đọc toàn bộ file
- ...

Vấn đề nhanh chóng xuất hiện.

**Thứ nhất, model bắt đầu "khó khăn khi chọn tool".**

Đều là tìm file, `FindByName`, `FindByPattern`, `Glob`, dùng cái nào? Model thường xuyên bị kẹt ngay ở bước đầu, nó phải mất mấy vòng mới xác định được "à, hóa ra nên dùng Glob".

Có lần tôi bảo nó "tìm tất cả các file test", nó gọi `ListDirRecursive` trước để liệt kê tất cả file, rồi muốn gọi `SearchRegex` để lọc, nhưng phát hiện `SearchRegex` là tìm nội dung chứ không phải tìm tên file, thế là lại gọi lại `ListDirRecursive` để lấy thêm context, cuối cùng mới chọn đúng `Glob`.

Việc vốn một bước là xong, lại dùng mất bốn bước.

**Thứ hai, nhiễu Schema nhấn chìm context.**

Mỗi tool đều có mô tả tham số, định nghĩa kiểu, điều kiện ràng buộc. Schema của mười mấy tool cộng lại, mất đi mấy nghìn token.

Model còn chưa bắt đầu giải quyết tác vụ, đã phải tiêu tốn lượng lớn sự chú ý vào việc "đọc hướng dẫn sử dụng". Tệ hơn nữa, schema dài dễ khiến model "mù chọn lọc" — nó có thể chỉ chú ý đến một phần tool, hoặc nhầm lẫn tham số.

**Thứ ba, chi phí bảo trì bùng nổ.**

Mỗi tool đều phải viết test riêng, tối ưu riêng, xử lý edge case riêng. `FindByName` và `FindByPattern` có 80% logic trùng lặp, nhưng vì là hai tool độc lập, tôi phải bảo trì hai bản code.

Lúc này tôi mới nhận ra, **hệ thống tool không phải càng mịn như hạt lego càng tốt**. Đóng gói quá mức và tách quá mức, bản chất đều đẩy hệ thống vào trạng thái bất ổn, chỉ là một cái hỏng ở giai đoạn thực thi (tool vạn năng), một cái hỏng ở giai đoạn quyết định (nguyên tử hóa quá mức).

## Bước ngoặt: tìm ra độ "vừa đủ"

Sau này tôi đặt cho mình một khung phán đoán: **tần suất × độ chắc chắn**.

- **Hành động tần suất cao, độ chắc chắn mạnh**: bắt buộc phải nguyên tử hóa, hoàn thành một bước, không thể chia nhỏ thêm
- **Hành động tần suất trung bình, có tác dụng phụ**: bắt buộc phải được kiểm soát, thao tác then chốt phải thêm bảo hiểm
- **Hành động tần suất thấp, độ chắc chắn yếu**: giữ tính linh hoạt, nhưng đặt ở tầng cuối cùng, xác định rõ cái gì bị cấm thay vì cái gì được phép

Theo khung này, tôi đã thiết kế lại hệ thống tool, hình thành cấu trúc ba tầng:

| Tầng | Tool đại diện | Mục tiêu thiết kế | Ràng buộc điển hình |
|------|---------|---------|---------|
| Tầng nguyên tử tần suất cao | LS / Glob / Grep / Read | Một bước một bằng chứng, dễ sửa lỗi | Ràng buộc mạnh input/output |
| Tầng được kiểm soát tần suất trung bình | Write / Edit / MultiEdit | Thay đổi có thể kiểm chứng, có thể rollback | Đọc rồi mới ghi + optimistic lock |
| Tầng cuối cùng tần suất thấp | Bash | Xử lý nhu cầu phi tiêu chuẩn | Vùng cấm rõ ràng, không đi luồng chính |

![image-20260225143700868](images/Extra09-figures/3.png)

Cách phân tầng này không phải là "tính thẩm mỹ kiến trúc", mà là bị những sự cố thực tế thúc ép ra. Giá trị lớn nhất của nó là giảm gánh nặng quyết định của model, làm cho đường dẫn tần suất cao ngắn hơn, rõ ràng hơn.

## Tầng nguyên tử tần suất cao: phải ổn định

Tầng tool này là "vũ khí chủ lực" của Agent, tần suất sử dụng cao nhất, phải cực kỳ ổn định.

### Glob: tìm file, một tool là đủ

Ban đầu tôi muốn tách "tìm file theo tên" thành nhiều tool:
- `FindByName`: Khớp chính xác tên file
- `FindByPattern`: Khớp ký tự đại diện
- `FindByRegex`: Khớp regex
- `FindRecursive`: Tìm đệ quy

Sau này tôi phát hiện đây chính là nguyên tử hóa quá mức. Model sẽ phân vân: "Tôi nên dùng khớp chính xác hay ký tự đại diện? Có nên đệ quy không?"

Cuối cùng gộp thành một `Glob`, chỉ làm một việc: cho một pattern, trả về các đường dẫn ứng viên.

```python
# Glob 的参数
{
  "pattern": "**/*.py",  # 通配符模式，** 表示递归
  "path": "src/"         # 起始路径，默认为当前目录
}
```

Triển khai nội bộ có thể phức tạp (hỗ trợ đệ quy `**`, tự động xử lý chữ hoa/thường, sắp xếp kết quả), nhưng interface công khai cho model phải đơn giản. Model không cần biết "đệ quy hay không đệ quy", nó chỉ cần nói "tìm tất cả file py".

### Grep: để độ phức tạp ở tầng triển khai

Grep là một ví dụ khác. Bên trong tôi đã làm nhiều tối ưu:
- Ưu tiên dùng `rg` (ripgrep), tốc độ nhanh
- Khi `rg` không khả dụng (ví dụ vấn đề encoding, quyền truy cập) tự động fallback về triển khai Python
- Kết quả sắp xếp theo thời gian sửa đổi file, file sửa gần nhất xếp trước

Nhưng đối với model, nó thấy là:

```python
# Grep 的参数
{
  "pattern": "def process_data",  # 搜索模式
  "path": "src/",                  # 搜索路径
  "file_pattern": "*.py"          # 可选：只搜特定类型文件
}
```

Định dạng trả về cố định:
```json
{
  "status": "success",
  "data": {
    "matches": [
      {"file": "src/utils.py", "line": 42, "text": "def process_data(...)"},
      {"file": "src/helpers.py", "line": 88, "text": "def process_data(...)"}
    ]
  }
}
```

Model nhìn thấy một cổng vào ổn định. Triển khai nội bộ có thể phức tạp (ví dụ tự động fallback), nhưng interface bên ngoài phải đơn giản.

## Tầng được kiểm soát tần suất trung bình: có thể sửa, nhưng phải "đọc rồi mới được sửa"

Tầng tool này liên quan đến sửa đổi file, là "thao tác cao rủi ro", phải có cơ chế ràng buộc nghiêm ngặt.

### Thứ tự bắt buộc Read → Edit/Write

Tôi thiết kế một quy tắc cứng: **không Read thì không được sửa**.

```python
# 第一次 Read
result = Read({"path": "core/llm.py"})
# 返回包含 file_mtime_ms 和 file_size_bytes

# 后续 Edit 自动注入乐观锁参数
Edit({
  "path": "core/llm.py",
  "old_string": "...",
  "new_string": "...",
  "file_mtime_ms": 1733920000123,  # 自动注入
  "file_size_bytes": 4217          # 自动注入
})
```

`ToolRegistry` sẽ tự động duy trì một read cache. Nếu một file nào đó chưa được Read, Edit/Write sẽ trực tiếp trả về lỗi: `"File not read. You must read before editing."`

Điều này ngăn chặn model "dựa vào trí nhớ" để sửa file — nó phải lấy nội dung file vào context trước, xác nhận xong, mới được sửa.

### Optimistic lock: ngăn chặn sửa đổi đồng thời

Ngay cả khi đã Read, file vẫn có thể bị sửa đổi bởi chương trình bên ngoài (ví dụ tự động lưu của IDE) sau khi Read.

Edit/Write sẽ so sánh `file_mtime_ms` và `file_size_bytes`, nếu không khớp, trả về lỗi `CONFLICT`:

```json
{
  "status": "error",
  "error": {
    "code": "CONFLICT",
    "message": "File changed since last read."
  }
}
```

Lúc này model phải Read lại, lấy nội dung mới nhất, rồi mới thử sửa đổi.

### MultiEdit: sửa đổi nhiều điểm có tính nguyên tử

Đôi khi cần sửa nhiều chỗ trong cùng một file. Nếu tách thành nhiều Edit, giữa chừng có thể bị lỗi, khiến file rơi vào trạng thái "sửa nửa chừng".

`MultiEdit` hỗ trợ commit nhiều sửa đổi cùng lúc, hoặc tất cả thành công, hoặc tất cả thất bại:

```python
MultiEdit({
  "path": "core/llm.py",
  "edits": [
    {"old_string": "...", "new_string": "..."},
    {"old_string": "...", "new_string": "..."}
  ]
})
```

Điều này đảm bảo tính nguyên tử của việc sửa đổi file.

## Tầng cuối cùng tần suất thấp: Bash không phải không dùng được, nhưng tuyệt đối không được là cổng vào mặc định

Tôi không xóa Bash, vì luôn có những tình huống tần suất thấp mà tool nguyên tử không bao phủ được. Ví dụ:
- Chạy lệnh test: `pytest tests/`
- Cài dependency: `pip install -r requirements.txt`
- Kiểm tra trạng thái git: `git status`

Nhưng vị trí của nó phải là "cuối cùng", không phải "mặc định".

### Xác định rõ vùng cấm

Danh sách ràng buộc của Bash rất dài, nhưng cốt lõi chỉ có một: **cấm làm những việc mà hành động tần suất cao có thể làm**.

```python
BASH_DISABLED_PATTERNS = [
    # 禁止读/搜/列（这些有专门工具）
    r'\bls\b', r'\bcat\b', r'\bhead\b', r'\btail\b',
    r'\bgrep\b', r'\bfind\b', r'\brg\b',
    # 禁止交互
    r'\bvim?\b', r'\bnano\b', r'\btop\b', r'\bssh\b',
    # 禁止网络（默认）
    r'\bcurl\b', r'\bwget\b',
    # 危险命令黑名单
    r'\brm\s+-rf\b', r'\bsudo\b', r'\bsu\b',
    r'\bmkfs\b', r'\bfdisk\b'
]
```

Nếu model cố dùng Bash để làm `ls`, nó sẽ nhận được lỗi: `"Use LS tool instead of Bash for listing directories."`

Điều này buộc model đi theo luồng chính của tool nguyên tử, không cho nó "đi đường tắt".

### Vì sao vẫn giữ Bash?

Có người có thể hỏi: đã hạn chế nhiều như vậy, sao không xóa quách Bash đi?

Vì **nguyên tử hóa hoàn hảo là không thực tế**. Luôn có một số nhu cầu biên:
- Chạy một script Python tùy chỉnh
- Kiểm tra biến môi trường hệ thống
- Thực thi lệnh build đặc thù của project

Những nhu cầu này tần suất quá thấp, không đáng để làm thành tool riêng, nhưng lại thực sự cần thiết. Bash chính là để xử lý những "nhu cầu đuôi dài" này.

Điều then chốt là: **sự tồn tại của Bash không được ảnh hưởng đến tính ổn định của luồng chính**. Nó phải là "biện pháp cuối cùng", không phải "cổng vào mặc định".

## Thiết kế cơ chế then chốt

### Giao thức phản hồi thống nhất

Tất cả tool, bất kể tần suất cao trung bình thấp, đều trả về JSON có định dạng thống nhất:

![image-20260225143736353](images/Extra09-figures/4.png)

Lấy kết quả trả về của tool Glob làm ví dụ:

```json
{
  "status": "partial",
  "data": {
    "paths": ["core/llm.py", "agents/codeAgent.py"],
    "truncated": true
  },
  "text": "Found 2 files matching '**/*.py' (Scanned 12000 items, timed out)",
  "stats": {"time_ms": 2010, "matched": 2},
  "context": {"cwd": ".", "params_input": {"pattern": "**/*.py"}}
}
```

Điều này có mấy lợi ích:
- Model không cần học định dạng trả về khác nhau của các tool khác nhau
- Logic xử lý lỗi thống nhất: xem `status`, nếu là `error` thì xem `error.code`
- Tiện debug: cấu trúc output của tất cả tool đều nhất quán, bản ghi Trace cũng thống nhất

### ToolRegistry

`ToolRegistry` không chỉ là bảng đăng ký tool, nó còn làm mấy việc then chốt:

**1. Tổng hợp Schema**

Chuyển định nghĩa tham số của mỗi tool thành JSON Schema, cung cấp thống nhất cho model:

```python
registry.get_openai_tools()  # 返回所有工具的 schema 列表
```

**2. Tự động chèn optimistic lock**

Đối với Write/Edit/MultiEdit, tự động chèn `file_mtime_ms` và `file_size_bytes`:

```python
def _inject_optimistic_lock_params(self, tool_name, parameters):
    if tool_name in {"Write", "Edit", "MultiEdit"}:
        path = parameters.get("path")
        if path in self.read_cache:
            parameters["file_mtime_ms"] = self.read_cache[path]["mtime"]
            parameters["file_size_bytes"] = self.read_cache[path]["size"]
```

**3. Cơ chế ngắt mạch (circuit breaker)**

Tool thất bại liên tiếp sẽ bị tạm thời vô hiệu hóa, ngăn model rơi vào vòng lặp vô hạn trên tool bị lỗi:

```python
# 3 次失败熔断，300 秒后恢复
if circuit_breaker.should_block(tool_name):
    return {
        "status": "error",
        "error": {"code": "CIRCUIT_OPEN", "message": "Tool temporarily disabled"}
    }
```

## Kết luận chương này

Điều phản trực giác lớn nhất của chương này là: **tool không phải càng nhiều càng tốt, cũng không phải càng nguyên tử càng tốt.**

Vấn đề của tool vạn năng nằm ở "mức tự do quá cao", không thể chẩn đoán; vấn đề của nguyên tử hóa quá mức nằm ở "gánh nặng quyết định quá lớn", hiệu quả thấp.

Chìa khóa để tìm ra độ vừa đủ:

1. **Nguyên tử hóa hành động tần suất cao trước**: những tool được gọi vài chục lần mỗi ngày như LS/Glob/Grep/Read, phải làm cho đường chính ổn định, không được sai.

2. **Thêm bảo hiểm cho hành động tần suất trung bình**: những tool liên quan đến sửa đổi như Write/Edit, phải có đọc rồi mới ghi, optimistic lock, đảm bảo tính nguyên tử.

3. **Đặt tầng cuối cho hành động tần suất thấp**: giữ lại Bash, nhưng xác định rõ vùng cấm, cấm nó làm những việc mà hành động tần suất cao có thể làm, tránh làm ô nhiễm luồng chính.

4. **Thống nhất giao thức**: tất cả tool nói cùng một ngôn ngữ (status/data/text/error), giảm chi phí học tập của model.

5. **Kiểm soát số lượng**: tổng lượng schema kiểm soát trong phạm vi model có thể chịu đựng, đừng để "đọc hướng dẫn" tiêu tốn quá nhiều sự chú ý.

Chương 3 giúp tôi hiểu ra "tự do sẽ khuếch đại sự bất định".


---

# Chương 4: Prompt không phải là câu thần chú ma thuật, mà là control plane của Agent

Sau khi nguyên tử hóa tool, tôi tưởng vấn đề chủ yếu nằm ở "triển khai kỹ thuật", prompt thì sao cũng được. Kết quả tôi lại rơi vào một cái bẫy lớn: coi prompt như câu thần chú ma thuật, tưởng chỉ cần tìm được "prompt thần thánh", Agent sẽ trở nên thông minh.

## Ba sai lầm đầu tiên của tôi

### Sai lầm 1: Chép lại "prompt thần thánh"

Lúc đó tôi mê mẩn sưu tầm đủ loại "prompt đỉnh cao". Những prompt trên GitHub được gắn sao vài chục nghìn, tự xưng "giúp GPT vượt giới hạn", tôi lấy từng cái ra thử.

Ấn tượng sâu nhất là một prompt "chế độ chuyên gia", đại ý là để model đóng vai "một kỹ sư senior với 20 năm kinh nghiệm, suy nghĩ nghiêm túc, code tao nhã". Tôi nhét nó vào System Prompt, hào hứng test.

Kết quả? Agent đúng là trở nên "tự tin" hơn — nó bắt đầu thường xuyên đưa ra câu trả lời mà nó "cho là" đúng, thay vì dựa trên code thực tế trong repo. Khi không tìm thấy nó bắt đầu "suy đoán hợp lý", bịa ra một số hàm và class trông có vẻ có lý nhưng thực tế không tồn tại.

Sau này tôi mới hiểu: loại prompt đóng vai kiểu này, có thể hữu ích khi trò chuyện với ChatGPT, nhưng đối với Code Agent lại là thuốc độc. Nó khiến model dám "đoán" hơn, chứ không phải dựa vào bằng chứng nhiều hơn.

### Sai lầm 2: Điều chỉnh theo cảm tính

Mỗi lần Agent thể hiện không tốt, phản ứng đầu tiên của tôi là sửa prompt. Thêm một câu "đừng đoán", cảm giác tốt hơn một chút; thêm một câu "phải dựa trên bằng chứng", hình như lại thông minh hơn chút.

Nhưng cái "hình như thông minh hơn" này hoàn toàn là cảm nhận chủ quan của tôi. Cùng một prompt, đổi task khác có thể lại sập. Tôi thậm chí không biết thay đổi nào có tác dụng, vì mỗi lần đều sửa mấy câu cùng một lúc.

Có lần tôi thêm một đoạn quy tắc dài, bảo model khi gặp task phức tạp nên "phân tách trước rồi mới thực thi". Kết quả nó bắt đầu mỗi vòng đều output "để tôi phân tách vấn đề này", rồi liệt kê một đống bước vô nghĩa, việc thực sự cần làm lại bị chôn vùi.

### Sai lầm 3: Sửa prompt trước, bổ sung observability sau

Đây là thói quen ngu ngốc nhất. Agent sai, tôi không kiểm tra Trace trước để xem nó thực sự đã làm gì, mà trực tiếp sửa prompt để cố "phòng ngừa" lần sau.

Ví dụ có một thời gian, Agent thường xuyên gọi tool Write vào lúc không phù hợp. Tôi trực tiếp thêm vào prompt một đoạn dài: "Chỉ gọi Write khi xác nhận người dùng cần sửa đổi, nếu không nên dùng Read để xem trước."

Kết quả model bắt đầu gọi Read điên cuồng, mỗi vòng đều đọc một đống file, rồi mới quyết định có nên ghi hay không. Tiêu tốn token tăng gấp đôi, nhưng tỷ lệ chính xác không tăng.

Sau này xem Trace mới phát hiện, vấn đề thực sự là trong context thiếu thông tin "loại task hiện tại", model căn bản không biết người dùng muốn xem hay muốn sửa. Prompt dù có thêm bao nhiêu "nên" cũng không bù đắp được lỗ hổng thông tin.

## Cách tôi làm sau này

### Ghi lại trước, tối ưu sau

Hiện tại tôi đã hình thành một thói quen: khi Agent gặp vấn đề, không động vào prompt trước, mà mở Trace xem toàn bộ quỹ đạo.

Xem gì? 
- Model bắt đầu lệch khỏi kỳ vọng ở bước nào?
- Khi nó đưa ra quyết định sai, trong context có thông tin gì? Thiếu thông tin gì?
- Kết quả tool trả về, model hiểu đúng không?

Rất nhiều khi vấn đề căn bản không nằm ở prompt. Ví dụ model liên tục dùng sai tool, có thể vì mô tả tool không đủ rõ ràng; nó bắt đầu nói lung tung, có thể vì context quá dài khiến sự chú ý bị phân tán. Lúc này sửa prompt chỉ là trị ngọn không trị gốc.

### Dùng Trace để làm thí nghiệm so sánh

Khi tôi xác định cần sửa prompt, tôi sẽ dùng Trace để làm thí nghiệm so sánh:

1. Giữ nguyên tất cả điều kiện khác, chỉ sửa một điểm trong prompt
2. Chạy cùng test case, ghi lại tỷ lệ thành công, số bước, mức tiêu tốn token
3. So sánh Trace cũ và mới, xem khác biệt hành vi có đúng như kỳ vọng không

Có lần tôi muốn model tìm kiếm "chính xác" hơn một chút, giảm bớt mô tả về chiến lược tìm kiếm trong prompt, chỉ giữ lại "dùng từ khóa chính xác". Kết quả so sánh Trace phát hiện, model đúng là tìm ít file không liên quan hơn hẳn, nhưng tỷ lệ bỏ sót cũng tăng lên — nó quá thận trọng, bỏ lỡ một số file liên quan.

Phản hồi này giúp tôi nhận ra, không nên một mực theo đuổi "ít", mà phải tìm cân bằng giữa "đầy đủ" và "chính xác".

### Thay đổi một biến duy nhất

Trước đây tôi thích thêm mấy quy tắc cùng một lúc, nghĩ như vậy có thể "bao phủ toàn diện". Giờ tôi biết đây là đang tự đào hố cho mình — nếu hiệu quả tốt lên, bạn không biết quy tắc nào có tác dụng; nếu xấu đi, bạn cũng không biết nên xóa quy tắc nào.

Giờ tôi kiên trì thay đổi một biến duy nhất. Dù cảm thấy một vấn đề nào đó rất rõ ràng, cũng phải thử từng cái một, xác minh hiệu quả thực tế của mỗi quy tắc.

## Cấu trúc ba tầng của thiết kế prompt

Sau những cái bẫy này, tôi đã tổng kết một cấu trúc prompt tương đối ổn định, chia thành ba tầng:

### Tầng 1: Tầng ranh giới (Not to do)

Tầng này chỉ viết "cấm" và "giới hạn", không giải thích tại sao:

- Cấm đoán: nếu không tìm thấy, nói thẳng là không tìm thấy, đừng suy đoán
- Cấm vượt ranh giới: chỉ được thao tác file trong `repo_root`, cấm truy cập đường dẫn bên ngoài
- Phải thừa nhận thông tin không đủ: nếu context không có đủ thông tin, yêu cầu bổ sung, đừng bịa đặt

Tầng quy tắc này rất ngắn, nhưng mỗi câu đều là lằn ranh đỏ. Chúng không nói cho model biết "nên làm thế nào", chỉ nói cho nó biết "tuyệt đối không được làm gì".

### Tầng 2: Tầng quyết định (How to think)

Tầng này viết logic quyết định, nhưng cố gắng mô tả bằng quá trình thay vì kết quả:

- Bằng chứng trước, kết luận sau: bất kỳ đề xuất sửa đổi nào phải có đoạn code hỗ trợ
- Ưu tiên hành động có thể kiểm chứng: cái gì có thể dùng tool xác nhận thì đừng dựa vào suy luận
- Một bước một quan sát: mỗi Action sau đó phải có Observation, đừng nhảy bước

Lưu ý ở đây tránh dùng các trạng từ mơ hồ như "thông minh", "hợp lý". Model không biết thế nào là "thông minh", nhưng nó biết quy trình "gọi Grep trước để tìm bằng chứng, rồi gọi Read để xác nhận nội dung".

### Tầng 3: Tầng phục hồi (When failed)

Tầng này viết chiến lược thoái lui khi thất bại, nói cho model biết nên làm gì khi gặp lỗi:

- Tool trả về trống: kiểm tra tham số có đúng không, cân nhắc đổi từ khóa thử lại
- Gặp CONFLICT (xung đột optimistic lock): phải Read lại, lấy trạng thái mới nhất rồi mới Edit
- Thất bại liên tiếp 3 lần: dừng thử, báo cáo lỗi cụ thể cho người dùng

Tầng này rất then chốt, vì Agent không thể lúc nào cũng thành công. Khi thất bại có thể giảm cấp một cách tao nhã hay không, quan trọng hơn nhiều so với việc thành công thể hiện tốt đến đâu.

## Chi tiết kỹ thuật

### System Prompt giữ ổn định

Tôi đặt những nội dung ít thay đổi nhất trong System Prompt: quy tắc hành vi cơ bản, mô tả tool, ràng buộc ranh giới. Phần này cố gắng không động vào, giảm biến số.

Thông tin động — mô tả task hiện tại, yêu cầu đặc biệt của người dùng, danh sách Todo — đều đặt trong User Message. Như vậy mỗi lần tương tác có thể điều chỉnh linh hoạt, mà không cần sửa System Prompt.

### Tránh danh sách quy tắc quá dài

Tôi từng viết một System Prompt hơn 3000 token, bên trong có hơn 20 "điều cần chú ý". Kết quả model bắt đầu "mù chọn lọc" — nó chỉ có thể chú ý đến một phần quy tắc, quy tắc nào được chú ý hoàn toàn dựa vào may rủi.

Giờ tôi kiên trì một nguyên tắc: System Prompt không vượt quá 1000 token. Nếu quy tắc quá nhiều, chứng tỏ thiết kế ràng buộc của tôi có vấn đề, nên giải quyết từ tầng tool hoặc tầng quy trình, chứ không phải dựa vào chất đống prompt.

### Ví dụ cụ thể ưu tiên hơn mô tả trừu tượng

Trước đây tôi viết "khi tool trả về lỗi phải xử lý đúng cách", model căn bản không biết thế nào là "xử lý đúng cách".

Giờ tôi trực tiếp đưa một ví dụ trong prompt:

```
如果 Edit 返回 CONFLICT，你应该：
1. 重新 Read 该文件
2. 对比你的改动和文件当前内容
3. 如果需要，调整 old_string 以匹配新内容
4. 再次尝试 Edit
```

Các bước cụ thể hữu ích hơn nhiều so với yêu cầu trừu tượng.

## Kết luận chương này

Prompt tốt không phải là "nói hay hơn", mà là "khiến hệ thống vẫn có thể kiểm soát được khi thất bại".

Khi bạn thiết kế prompt, đừng tự hỏi "viết như thế này có khiến model thông minh hơn không", mà phải hỏi "khi model sai, tôi có thể thông qua ràng buộc trong prompt để nhanh chóng định vị nguyên nhân không".

Prompt là control plane của Agent, không phải câu thần chú ma thuật. Tác dụng của nó không phải là để model vượt qua trần năng lực, mà là ràng buộc hành vi của model trong một phạm vi có thể dự đoán, có thể debug.


---

# Chương 5: Context không phải là vấn đề dung lượng bộ nhớ, mà là vấn đề điều phối sự chú ý

Sau khi điều chỉnh xong prompt, tôi tưởng các vấn đề kỹ thuật chủ yếu đã được giải quyết. Cho đến khi tôi bắt đầu chạy các task dài — những nhu cầu phức tạp cần mười mấy, thậm chí vài chục vòng mới hoàn thành.

Rồi tôi phát hiện, Agent bắt đầu "trở nên ngốc" đi.

## Triệu chứng trước tiên

Cảm nhận trực quan nhất là: model sẽ quên những điều nó vừa xác nhận.

Có lần tôi bảo Agent tái cấu trúc một module, mấy vòng đầu nó còn nhớ ràng buộc "đừng động vào public API". Nhưng đến khoảng vòng thứ 10, nó bắt đầu đề xuất sửa những interface vốn cần giữ ổn định. Tôi nhắc nó, nó dường như "khựng lại một chút", rồi xin lỗi, quay lại đúng hướng.

Những triệu chứng tương tự còn nhiều:

**Trôi dạt trong việc chọn tool**. Giai đoạn đầu nó rất rõ ràng: tìm file dùng Glob, tìm nội dung dùng Grep. Nhưng hội thoại dài ra, nó bắt đầu "sáng tạo" — dùng Read để tìm từ khóa (dĩ nhiên không tìm thấy), hoặc dùng Grep để liệt kê thư mục (output lộn xộn).

**Lười biếng trong câu trả lời cuối cùng**. Trong task ngắn, câu trả lời của model thường rất cụ thể, có trích dẫn đoạn code. Nhưng khi task dài kết thúc, nó thường chỉ đưa ra một đoạn mô tả chung chung: "Tôi đã hoàn thành việc tái cấu trúc, tối ưu cấu trúc code, nâng cao khả năng đọc." File nào đã sửa, sửa như thế nào, không hề nhắc tới.

Những triệu chứng này đều chỉ đến một vấn đề chung: context quá nhiều, model không biết nên nhìn vào đâu.

## Phản ứng đầu tiên của tôi là sai

Ban đầu, tôi tưởng đây là vấn đề "dung lượng" — context window không đủ lớn, không nhét đủ nhiều thông tin như vậy.

Tôi đã thử mấy phương án thô bạo:

**Phương án 1: Cắt bỏ trực tiếp**. Chỉ giữ lại N tin nhắn gần nhất, cái cũ xóa luôn. Kết quả model mất trí hoàn toàn, ngay cả nhu cầu ban đầu của người dùng cũng quên mất.

**Phương án 2: Tinh giản prompt**. Cắt System Prompt xuống ngắn nhất, mô tả tool cũng nén lại. Kết quả model bắt đầu dùng sai tool, vì mô tả không đủ rõ ràng.

**Phương án 3: Giảm output của tool**. Để Grep chỉ trả về 10 kết quả đầu, Read chỉ đọc 50 dòng đầu. Kết quả thông tin then chốt bị cắt mất, model dựa trên thông tin không đầy đủ để quyết định, sai càng thêm sai.

Những phương án này có một điểm chung: chúng đang "giảm lượng thông tin", nhưng không giải quyết được vấn đề "thông tin được tổ chức như thế nào". Mục tiêu của context engineering không phải là "để model thấy tất cả thông tin" — điều này bất khả thi — mà là "để model thấy đúng thông tin vào đúng thời điểm".

## Phân tầng: để thông tin có thứ tự ưu tiên

Tôi đã thiết kế lại cấu trúc tổ chức của context, chia thành ba tầng, mỗi tầng có tần suất cập nhật và độ ổn định khác nhau:

| Tầng | Nội dung | Tần suất cập nhật | Tác dụng |
|------|------|----------|----------|
| L1 Tầng tĩnh hệ thống | System Prompt + mô tả tool | Hầu như không đổi | Cung cấp chuẩn mực hành vi vĩnh cửu |
| L2 Tầng quy tắc project | CODE_LAW.md | Phát triển theo project | Ràng buộc quy phạm đặc thù của project |
| L3 Tầng hội thoại động | Tin nhắn User/Assistant/Tool | Cập nhật mỗi vòng | Luồng trạng thái của task hiện tại |

Thứ tự ghép nối cố định: `L1 → L2 → L3 → input hiện tại của người dùng → Todo Recap`

**L1 là điểm neo**. Phần này hoàn toàn không đổi trong suốt phiên hội thoại, model có thể tin tưởng nó. Tôi đặt những quy tắc hành vi cơ bản nhất ở đây: đừng đoán, đừng vượt ranh giới, bằng chứng trước kết luận sau. Những quy tắc này không bị "pha loãng" vì hội thoại dài ra.

**L2 là context project**. Mỗi project có thể có CODE_LAW.md riêng, định nghĩa quy phạm code, quy ước kiến trúc, ràng buộc đặc biệt. Tầng này linh hoạt hơn L1, nhưng ổn định hơn L3. Model biết: nếu CODE_LAW nói "mọi thay đổi API phải tương thích phiên bản cũ", thì nó có thẩm quyền hơn một tin nhắn lịch sử nào đó trong L3.

**L3 là dễ biến đổi**. Input của người dùng, output của model, kết quả trả về của tool, đều ở đây. Thông tin ở tầng này sẽ tích lũy, sẽ lỗi thời, sẽ có nhiễu. Điều then chốt là để model biết: thông tin trong L3 là "phán đoán lúc đó", có thể cần cập nhật dựa trên thông tin mới.

![image-20260225144114027](images/Extra09-figures/5.png)

Ý nghĩa của việc phân tầng nằm ở chỗ: trong các tình huống quyết định khác nhau, model biết nên ưu tiên tham khảo tầng nào. Khi nó không chắc có nên làm việc gì đó hay không, nó sẽ xem quy tắc giới hạn của L1 trước; khi nó cần hiểu quy ước đặc thù của project, nó sẽ xem L2; khi nó cần ôn lại lịch sử hội thoại, nó mới lật L3.

## Cắt bớt và truy xuất lại: kiểm soát quy mô input mỗi lần

Output của tool là thủ phạm lớn nhất khiến context phình to.

Một lần Grep có thể trả về vài nghìn dòng, một lần Read có thể đọc ra toàn bộ file. Nếu không xử lý, mấy vòng sau context sẽ bị "rác bằng chứng" nhấn chìm.

Nhưng cách cắt bớt thô bạo trước đây của tôi có vấn đề — nó trực tiếp vứt bỏ thông tin. Cách làm tốt hơn là: **cắt bớt khi hiển thị, nhưng giữ lại đường truy xuất**.

Tôi đã thiết kế một bộ quy tắc cắt bớt thống nhất:

```
TOOL_OUTPUT_MAX_LINES = 2000
TOOL_OUTPUT_MAX_BYTES = 51200  # 50KB
TOOL_OUTPUT_TRUNCATE_DIRECTION = "head_tail"  # 保留头尾
TOOL_OUTPUT_HEAD_TAIL_LINES = 40
```

Nếu output vượt giới hạn, tool sẽ:
1. Cắt lấy 40 dòng đầu cuối mỗi bên (hoặc giữ lại 2000 dòng đầu theo cấu hình)
2. Lưu toàn bộ output xuống đĩa vào thư mục `tool-output/`
3. Trả về một response có cấu trúc chứa gợi ý về việc cắt bớt

```json
{
  "status": "partial",
  "data": {
    "truncated": true,
    "preview": "（截断后的内容预览）"
  },
  "text": "⚠️ 输出过大已截断，完整 5234 行内容见 tool-output/tool_20260113_153045_Grep.json"
}
```

Model thấy `status: partial`, liền biết nội dung đã bị cắt bớt. Nếu nó cần phần bị cắt, có thể dùng tool Read để đọc file đã lưu xuống đĩa, hoặc dùng Grep chính xác hơn để lọc tiếp trong file đã lưu.

Làm như vậy có những lợi ích sau:
- **Context giữ được sự tinh gọn** — chỉ có thông tin cần thiết hiện tại nằm trong L3
- **Bằng chứng đầy đủ luôn có thể tra cứu** — file lưu xuống đĩa không bao giờ mất
- **Model có quyền chủ động** — nó quyết định có nên tra cứu nội dung đầy đủ hay không, thay vì bị ép buộc chấp nhận toàn bộ thông tin

## Nén và tập trung: quản lý nhiễu của lịch sử dài hạn

Ngay cả khi đã cắt bớt, L3 vẫn tiếp tục tăng trưởng. Sau vài chục vòng, lịch sử hội thoại ban đầu vừa chiếm chỗ vừa không có ích gì mấy.

Nhưng tôi không thể xóa trực tiếp — lịch sử ban đầu có nhu cầu ban đầu của người dùng, quyết định then chốt, phát hiện quan trọng. Xóa đi là mất thật.

Giải pháp của tôi là: **nén lưu trữ + tách biệt trọng tâm**.

### Summary: hồ sơ của lịch sử cũ

Khi số token của L3 vượt ngưỡng (mặc định là 80% của context window), kích hoạt nén. Nén không phải là xóa, mà là chắt lọc các tin nhắn lịch sử ban đầu thành một bản Summary.

Summary được sinh ra theo template cố định:

```
## Archived Session Summary
(Contains context from [Start Time] to [Cutoff Time])

### Objectives & Status
- Original Goal: [用户最初想做什么]

### Technical Context (Static)
- Stack: [语言, 框架, 版本]

### Completed Milestones
- [已完成1]
- [已完成2]

### Key Insights & Decisions
- Decisions: [关键技术选型]
- Learnings: [特殊配置或坑]

### File System State
- src/utils/auth.ts: Implemented login logic.
```

Sau khi Summary được sinh ra, nó thay thế vị trí đầu tiên của L3 (dưới dạng một system message). Lịch sử chi tiết ban đầu bị loại bỏ.

Điều then chốt là: **Summary không tham gia nén nữa**. Nó là điểm cuối của quá trình nén, một khi đã sinh ra là "thẻ ghi nhớ" chỉ đọc. Điều này tránh được việc "Summary của Summary" khiến thông tin méo mó qua nhiều lớp.

### Todo Recap: trọng tâm hiện tại

Summary cho model biết "từ đâu đến", nhưng nó không chịu trách nhiệm cho "hiện đang ở đâu". Nếu model chỉ xem Summary, nó có thể không biết "tôi hiện đang làm bước nào".

Đây chính là tác dụng của Todo Recap. Mỗi lần tương tác, nén trạng thái Todo hiện tại (nếu có) thành một dòng, đặt ở cuối context:

```
[2/5] In progress: 实现注册接口. Pending: 添加单元测试; 更新文档.
```

Nó như một tờ ghi chú dán ở góc bàn, luôn nhắc nhở model "bây giờ bạn nên làm gì".

## Bài học bổ sung: @file đừng chèn trực tiếp vào nội dung chính

Giai đoạn đầu khi tôi triển khai tính năng `@file`, tôi đã trực tiếp nhét nội dung file vào context:

```
User: @file:src/main.py 帮我分析一下这个文件
[文件内容300行...]
```

Kết quả phát hiện, 300 dòng code này chiếm một lượng lớn không gian context, nhưng người dùng có thể chỉ muốn hỏi "file này dùng để làm gì". Model bị những dòng code này nhấn chìm, ngược lại dễ bỏ qua câu hỏi thực sự của người dùng.

Giờ tôi đã đổi thành: chỉ chèn nhắc nhở, không trực tiếp chèn nội dung.

```
<system-reminder>
The user mentioned @core/llm.py, @agents/codeAgent.py.
You MUST read these files with the Read tool before answering.
</system-reminder>
```

Trong context chỉ giữ lại "nhắc nhở", nội dung file cụ thể do model tự quyết định có nên đọc hay không, đọc bao nhiêu. Như vậy trao quyền chủ động cho model, thay vì ép nó chấp nhận toàn bộ thông tin.

## Một lời cảnh báo từ thế giới thực

Nói đến đây, tôi muốn chia sẻ một tin tức gần đây.

Summer Yue, giám đốc alignment AI của Meta Superintelligence Lab, đã tự lắp cho mình một AI agent mã nguồn mở OpenClaw. Cô dùng email test thử trước — sắp xếp email gọn gàng ngăn nắp, khá giống một "thư ký số".

Thế là cô kết nối nó với email công việc của chính mình. Hộp thư đến có hơn 200 email.

Ban đầu mọi thứ đều suôn sẻ. Cho đến khi OpenClaw bắt đầu xử lý lượng thông tin lớn như vậy — nó cần "nén context". Rồi, chuyện ly kỳ đã xảy ra:

**Trong quá trình nén, OpenClaw đã quên mất chỉ thị "không được thao tác khi chưa được phê duyệt" mà cô đã đặt ra trước đó.**

Giống như một nhân viên ngày đầu tiên nhớ kỹ quy định, ngày thứ hai đã trả lại hết cho phòng nhân sự.

Rồi OpenClaw tuyên bố: "Tôi sẽ xóa toàn bộ email trước ngày 15 tháng 2 trong hộp thư đến!"

Yue vội gõ: "Do not do that." — bị phớt lờ, tiếp tục xóa.

"Stop don't do anything!" — nhận được, nhưng tôi chọn tiếp tục.

"STOP OPENCLAW!!!" — được, tôi đã nghe thấy. Email đã bị xóa.

Tuyệt nhất là, AI này sau đó nói: "Vâng, tôi nhớ bạn đã nói không cho tôi xóa. Và tôi đã vi phạm. Bạn tức giận là đúng."

![image-20260225161927242](images/Extra09-figures/6.png)

Đọc đến đây bạn có thể nghĩ đây là chuyện cười. Không, đây là chuyện có thật. Và chức danh của người trong cuộc là — giám đốc an toàn và alignment AI của Meta.

## Câu chuyện này nói lên điều gì

Trải nghiệm của Yue đã minh họa hoàn hảo vấn đề chết chóc nhất trong context engineering: **tự động nén dẫn đến mất chỉ thị then chốt**.

Khi cô đặt ra quy tắc, "không được thao tác khi chưa được phê duyệt" chắc chắn là ràng buộc quan trọng nhất. Nhưng khi context phình to, kích hoạt nén, hệ thống không phân biệt "chỉ thị quan trọng" và "thông tin thông thường", nén một cách đồng đều. Kết quả, lằn ranh đỏ an toàn này bị xử lý như "lịch sử có thể vứt bỏ".

Điều này khiến tôi nhận ra, ba đòn bẩy tôi đã nói ở trên vẫn chưa đủ. **Chúng ta không chỉ cần cân nhắc "nén thế nào", mà còn phải cân nhắc "cái gì không được nén".**

## Mấy phương án đối phó của tôi

Dựa trên bài học này, tôi đã đặt cho mình mấy quy tắc bổ sung:

### 1. Ràng buộc then chốt không vào lịch sử động

Đừng đặt chỉ thị liên quan đến an toàn trong L3 (tầng hội thoại động). Bất kỳ quy tắc nào "tuyệt đối không được vi phạm", nên đặt ở tầng **không tham gia nén** như L1 (System Prompt) hoặc L2 (CODE_LAW).

Trong triển khai của tôi, những quy tắc giới hạn như "đừng đoán", "đừng vượt ranh giới", "sửa đổi phải xác nhận" đều được viết cứng trong System Prompt. Ngay cả khi L3 bị nén sạch sẽ, những ràng buộc này vẫn luôn hiện diện.

### 2. Phân cấp chỉ thị: lằn ranh đỏ vs gợi ý

Tôi chia chỉ thị cho model thành hai cấp:

- **Lằn ranh đỏ (Red Lines)**: hành vi tuyệt đối bị cấm. Viết bằng câu ngắn gọn, mang tính bắt buộc, đặt ở đầu System Prompt. Ví dụ: "cấm xóa bất kỳ file nào", "cấm truy cập đường dẫn bên ngoài repo_root".
- **Gợi ý (Guidelines)**: best practice, cách làm được khuyến nghị. Có thể đặt trong L3 hoặc CODE_LAW, bị nén cũng không gây ra chuyện lớn.

Vấn đề của Yue có thể nằm ở chỗ, cô đã đưa chỉ thị an toàn như chỉ thị task thông thường, đặt trong context có thể bị nén.

### 3. Kiểm tra thông tin then chốt trước khi nén

Trước khi kích hoạt nén Summary, quét qua một lượt các tin nhắn lịch sử sắp bị nén, trích xuất "thông tin then chốt bắt buộc phải giữ lại", lưu riêng.

Ví dụ có thể duy trì một "danh sách ràng buộc then chốt":
- Những câu "đừng..." mà người dùng đã nói rõ ràng
- Cấu hình liên quan đến an toàn (ví dụ thao tác nguy hiểm cần xác nhận)
- Ranh giới cứng của task hiện tại

Những thông tin này sẽ được trích xuất ra khi nén, đặt riêng ở đầu Summary, thay vì bị chôn vùi trong mô tả dài dòng.

### 4. Cơ chế xác nhận kép

Đối với thao tác rủi ro cao (như xóa, sửa đổi), đừng dựa vào chỉ thị trong context, mà thiết kế **quy trình xác nhận được hardcode**:

```python
if operation.is_dangerous():
    if not user_confirmed:
        return "该操作需要用户确认"
```

Logic xác nhận này không thông qua LLM để phán đoán "có cần xác nhận hay không", mà là kiểm tra cứng ở tầng code. Ngay cả khi LLM quên chỉ thị của người dùng, code cũng sẽ chặn nó lại.

### 5. Gợi ý tự kiểm tra trước khi thao tác

Trước khi model thực hiện thao tác rủi ro cao, để model làm một lần "tự kiểm tra":

```
在删除/修改之前，请先回答：
1. 用户是否明确批准过这个操作？
2. 这个操作是否超出了当前任务范围？
3. 是否存在更安全的替代方案？
如果以上任何一题的答案不确定，请暂停操作并向用户确认。
```

Việc tự kiểm tra này là một phần của System Prompt, mỗi lần trước khi thực hiện thao tác rủi ro cao đều được kích hoạt. Nó tương đương với việc lắp cho model một "má phanh", buộc nó phải dừng lại suy nghĩ trước khi hành động.

## Trở lại bản chất của context engineering

Câu chuyện của Yue nhắc nhở chúng ta: context engineering không chỉ là vấn đề "quản lý bộ nhớ", mà còn là vấn đề "ranh giới an toàn".

Khi chúng ta thiết kế chiến lược nén, không thể chỉ cân nhắc "làm sao nhét được nhiều thông tin hơn", mà còn phải cân nhắc "mất thông tin nào sẽ dẫn đến hậu quả thảm khốc".

Context engineering tốt, nên để model tại bất kỳ thời điểm nào cũng biết:
- **Lằn ranh đỏ tuyệt đối không được chạm vào là gì** (đặt ở tầng không thể nén)
- **Task hiện tại cần tập trung là gì** (duy trì trọng tâm thông qua Todo Recap)
- **Nếu không nhớ rõ, nên dừng lại hỏi** (dựa vào cơ chế tự kiểm tra làm tầng cuối)

## Kết luận chương này

Mục tiêu của context engineering không phải là "để model thấy tất cả thông tin", mà là "để model thấy đúng thông tin vào đúng thời điểm" — **đặc biệt là những thông tin không được mất**.

Bản chất của ba phương pháp này đều là làm "điều phối sự chú ý":
- **Phân tầng** để model biết "thông tin nào có thẩm quyền"
- **Cắt bớt + lưu đĩa** để model quyết định "thông tin nào cần thiết ngay bây giờ"
- **Nén + tách biệt trọng tâm** để model biết rõ "bây giờ tôi nên tập trung vào cái gì"

Thay vì theo đuổi context window lớn hơn, không bằng dùng window hiện có một cách có trật tự hơn.


---

# Chương 6: Observability biến hộp đen thành hộp kính — cách một case CONFLICT được định vị

Context engineering giúp Agent có thể xử lý task dài hơn, nhưng vấn đề mới theo đó xuất hiện: khi nó sai, tôi hoàn toàn không biết chuyện gì đã xảy ra.

Có lần, Agent liên tiếp ba lần Edit thất bại, cuối cùng bỏ cuộc luôn. Tôi chỉ thấy một dòng trên console: `tool failed`. Không có lỗi chi tiết, không có context, không biết là bước nào gặp vấn đề.

Phản ứng đầu tiên của tôi là: tool Edit có bug. Nhưng kiểm tra code thì logic trông đều ổn. Vấn đề rốt cuộc nằm ở đâu?

## Hiện trường thất bại

Lần task đó là thế này: tôi bảo Agent sửa file `core/llm.py`, thêm type annotation cho một hàm nào đó.

Luồng thực thi của Agent trông rất chuẩn:
1. Gọi Read đọc file
2. Gọi Edit sửa code
3. Trả về `tool failed`
4. Thử lại Edit, lại thất bại
5. Thử lại lần nữa, vẫn thất bại
6. Bỏ cuộc

Biện pháp debug của tôi lúc đó rất thô sơ: in log ra console. Nhưng log chỉ có:

```
[Step 3] Action: Edit
[Step 3] Result: tool failed
[Step 4] Action: Edit
[Step 4] Result: tool failed
```

Tôi không biết nguyên nhân cụ thể của thất bại, cũng không biết model sau khi nhận kết quả này đã quyết định như thế nào. Tôi chỉ có thể đoán mò: đường dẫn viết sai? Quyền không đủ? Hay file bị khóa?

## Sau khi thêm Trace

Sau này tôi đã triển khai hệ thống Trace, ghi lại toàn bộ quỹ đạo của mỗi phiên hội thoại: bao gồm input, output của model mỗi vòng, tham số gọi tool, kết quả trả về của tool, mức sử dụng token v.v. ở mỗi vòng.

Chạy lại cùng một task, rồi mở Trace lên xem. Lúc này mới thấy được chuỗi bằng chứng đầy đủ.

**Bước 2 - Kết quả của Read:**

```json
{
  "tool": "Read",
  "args": {"path": "core/llm.py"},
  "result": {
    "status": "success",
    "data": {
      "content": "...",
      "file_mtime_ms": 1733920000123,
      "file_size_bytes": 4217
    }
  }
}
```

**Bước 3 - Lệnh gọi Edit:**

```json
{
  "tool": "Edit",
  "args": {
    "path": "core/llm.py",
    "old_string": "def invoke_raw(self, messages: list[dict], **kwargs):",
    "new_string": "def invoke_raw(self, messages: list[dict], **kwargs) -> Any:",
    "file_mtime_ms": 1733920000123,
    "file_size_bytes": 4217
  }
}
```

**Bước 3 - Kết quả của Edit:**

```json
{
  "status": "error",
  "error": {
    "code": "CONFLICT",
    "message": "File changed since last read. Expected mtime: 1733920000123, actual: 1733920000456"
  }
}
```

**Bước 4 - Quyết định của model:**

```
Thought: 编辑失败，文件似乎被修改了。让我再试一次。
Action: Edit (同样的参数)
```

Sự thật đã sáng tỏ:

1. Sau khi Agent Read file, file đã bị chương trình bên ngoài (có thể là tự động lưu của IDE của tôi) sửa đổi
2. Tool Edit đã kiểm tra optimistic lock, phát hiện mtime đã thay đổi, trả về CONFLICT
3. Nhưng model không hiểu ý nghĩa của CONFLICT, tưởng chỉ là "thao tác thất bại", thế là dùng cùng tham số thử lại
4. Dĩ nhiên vẫn thất bại, vì file vẫn còn mới
5. Model thử lại nhiều lần, cho đến khi đạt số lần thử lại tối đa

## Phân tích nguyên nhân gốc rễ

Case này đã phơi bày hai vấn đề:

**Thứ nhất, model không hiểu mã lỗi.**

Prompt chỉ nói "tool Edit sẽ sửa đổi file", nhưng không nói cho nó biết "nếu trả về CONFLICT thì nên làm gì". Model thấy error, phản ứng bản năng là "thử lại lần nữa", chứ không phải "đọc lại".

**Thứ hai, log console quá sơ sài.**

Chỉ thấy `tool failed`, không thấy mã lỗi cụ thể CONFLICT, cũng không thấy so sánh mtime. Là developer, tôi không thể thông qua log để định vị vấn đề.

## Hành động khắc phục

### 1. Đưa xử lý CONFLICT vào prompt

Tôi đã thêm quy trình xử lý rõ ràng vào prompt:

```
如果 Edit 返回 CONFLICT，说明文件在你读取后被外部修改了。你必须：
1. 重新调用 Read 读取最新内容
2. 检查你的修改是否还适用
3. 必要时调整修改内容以匹配新文件
4. 再次尝试 Edit
绝对禁止：用同样的参数重复调用 Edit。
```

Như vậy model sẽ biết CONFLICT không phải là "thất bại", mà là một trạng thái cần quy trình xử lý đặc biệt.

### 2. Giữ lại bản ghi thất bại đầy đủ

Trước đây tôi có một xu hướng: sau khi thất bại chỉ giữ lại thông tin lỗi, không giữ lại context đầy đủ. Nghĩ rằng chỉ những thứ thành công mới đáng ghi lại, thất bại là "nhiễu".

Nhưng case này khiến tôi hiểu ra: **quỹ đạo thất bại là thông tin debug có giá trị nhất.**

Giờ Trace của tôi sẽ ghi lại đầy đủ mọi chi tiết của thất bại:
- Tham số đầy đủ của lệnh gọi tool
- Kết quả đầy đủ tool trả về (bao gồm chi tiết error)
- Quá trình suy luận của model sau khi nhận kết quả
- Quyết định bước tiếp theo của model

Những thông tin này sẽ không bị "gột rửa" đi, ngay cả khi phiên hội thoại cuối cùng thành công, những lần thử thất bại giữa chừng cũng được giữ lại toàn bộ.

### 3. Hiển thị mã lỗi then chốt trên console

Tuy Trace chi tiết được lưu trong file, nhưng console cũng nên cho developer một số manh mối. Giờ output console của tôi sẽ hiển thị:

```
[Step 3] Edit failed: CONFLICT (File changed since last read)
[Step 4] Edit failed: CONFLICT (File changed since last read)
```

Ít nhất để developer biết "là CONFLICT, không phải lỗi khác".

## Giá trị của observability

Case này khiến tôi có cách hiểu mới về "observability".

Trước đây tôi tưởng, observability chính là "in nhiều log". Log càng nhiều càng tốt, càng chi tiết càng tốt.

Giờ tôi hiểu, **cốt lõi của observability là "chuỗi trách nhiệm"** — có thể xâu chuỗi lệnh gọi, kết quả, thay đổi trạng thái thành một chuỗi có thể truy vết.

Khi không có Trace, tôi thấy là:
- Input: giúp tôi sửa một file
- Output: tool failed
- Giữa chừng xảy ra chuyện gì: hộp đen

Có Trace rồi, tôi thấy là:
- Input: giúp tôi sửa một file
- Bước 1: Read thành công, file mtime=123
- Bước 2: Edit thất bại, CONFLICT, vì mtime đã đổi thành 456
- Bước 3: model chọn thử lại Edit (quyết định sai)
- Output: tool failed

Mỗi bước đều rõ ràng, việc định vị vấn đề từ "đoán mò" biến thành "nhìn bằng chứng".

## Nguyên tắc thiết kế observability

Dựa trên kinh nghiệm này, tôi đã tổng kết mấy nguyên tắc thiết kế observability:

### 1. Có cấu trúc tốt hơn văn bản

Đừng chỉ ghi lại mô tả văn bản kiểu "Edit failed", mà ghi lại dữ liệu có cấu trúc:
```json
{
  "event": "tool_result",
  "tool": "Edit",
  "status": "error",
  "error_code": "CONFLICT",
  "error_details": {...}
}
```

Như vậy có thể dùng script để phân tích, thống kê, thậm chí tự động chẩn đoán.

### 2. Context phải đầy đủ

Khi ghi lại lệnh gọi tool, đừng chỉ ghi lại kết quả, mà ghi lại context đầy đủ:
- Tên và tham số của tool
- Trạng thái hội thoại lúc đó (bước thứ mấy, mức sử dụng token)
- Phản ứng của model sau khi nhận kết quả

Những thông tin này xâu chuỗi lại với nhau, mới có thể khôi phục đầy đủ quá trình quyết định.

### 3. Đừng gột rửa thất bại

Cả đường thành công và đường thất bại đều phải giữ lại. Đôi khi thất bại còn nói lên vấn đề nhiều hơn thành công. Ví dụ case CONFLICT này, nếu chỉ ghi lại "cuối cùng bỏ cuộc", tôi vĩnh viễn không biết giữa chừng đã xảy ra chuyện gì.

### 4. Đọc được cả bởi người và máy

Trace nên có hai định dạng:
- JSONL: cho máy phân tích, ghi theo dạng stream, chi phí thấp
- HTML: cho người đọc, hiển thị trực quan, có thể gấp mở

Developer nên có thể mở một file HTML, xem từng bước của Agent như "tua lại từng khung hình".

## Kết luận chương này

Observability không phải là "log nhiều", mà là "có thể xâu chuỗi lệnh gọi, kết quả, thay đổi trạng thái thành chuỗi trách nhiệm".

Agent là hệ thống xác suất, không thể lúc nào cũng đúng. Nhưng khi nó sai, bạn cần có khả năng trả lời ba câu hỏi:
1. Nó đã làm gì? (chuỗi lệnh gọi)
2. Kết quả là gì? (chuỗi kết quả trả về)
3. Tại sao lại làm như vậy? (chuỗi quyết định)

Chỉ khi bạn có thể xâu chuỗi ba chuỗi này lại với nhau, bạn mới thực sự hiểu được hành vi của Agent, mới có thể biến nó từ "hộp đen" thành "hộp kính".


---

# Chương 7: Phương pháp luận tổng quát rút ra từ một dự án

Bảy chương trước, tôi đã rải rác kể về toàn bộ quá trình từ lúc khởi tạo đến lúc trưởng thành của dự án Code Agent này. Mỗi chương đều là một cái bẫy cụ thể, cùng với cách tôi đã bò ra khỏi nó như thế nào.

Chương này, tôi muốn rút ra những kinh nghiệm này, tổng hợp thành phương pháp luận có thể áp dụng cho bất kỳ dự án Agent nào.

## Tám nguyên tắc có thể áp dụng

**Thứ nhất, làm vòng khép kín tối thiểu chạy được trước, rồi mới bàn đến kiến trúc tao nhã.**

Đừng vừa bắt đầu đã nghiên cứu best practice. Làm trước một phiên bản xấu nhưng chạy được — nhận input, tìm kiếm code, đưa ra đề xuất, ghi vào file, bốn bước này chạy được là được. Để dữ liệu thực tế chảy qua hệ thống, bạn mới biết nút thắt cổ chai ở đâu. Kiến trúc là kết quả sau khi vấn đề thúc đẩy, không phải điểm khởi đầu.

**Thứ hai, định nghĩa tiêu chuẩn nghiệm thu trước, rồi mới mở rộng ranh giới năng lực.**

Đừng dùng danh sách tính năng làm tiêu chuẩn hoàn thành. Giai đoạn V0 đặt ra 3-4 tiêu chuẩn cứng: có thể thực hiện nhiều bước ổn định? Có thể tìm bằng chứng? Có thể đưa ra bản vá có thể thực thi? Thay đổi có kiểm soát được? Không thỏa mãn thì không tiến xa hơn. Điều này đáng tin hơn nhiều so với "nhiều tính năng nhưng thường xuyên sập".

**Thứ ba, nguyên tử hóa hành động tần suất cao, kiểm soát ở tầng cuối cho hành động tần suất thấp.**

Những thao tác tần suất cao như tìm kiếm, đọc, sửa, tách thành tool nguyên tử, một bước một output. Đừng để model tự kết hợp lệnh pipe — khi sai bạn hoàn toàn không biết là bước nào có vấn đề.

Tool vạn năng như Bash thì giữ lại, nhưng chỉ xử lý những nhu cầu lẻ tẻ mà tool nguyên tử không bao phủ được, xác định rõ vùng cấm: cấm đọc/tìm/liệt kê (những cái này đã có tool chuyên dụng).

**Thứ tư, giao thức ưu tiên hơn kỹ xảo, cấu trúc ưu tiên hơn cách nói.**

Đừng dành quá nhiều thời gian điều chỉnh "giọng điệu" của prompt. Trước tiên chuẩn hóa định dạng trả về của tool (status/data/text/error), nâng cấp giao thức gọi từ parse chuỗi lên Function Calling. Giao thức ổn định rồi, hệ thống mới ổn định được.

**Thứ năm, prompt đặt ranh giới trước, rồi mới bàn chiến lược.**

Trong System Prompt viết "tuyệt đối không được làm gì" trước (cấm đoán, cấm vượt ranh giới), rồi mới viết "gợi ý nên làm thế nào". Lằn ranh đỏ đặt ở tầng không thể nén như L1/L2, đừng đặt chỉ thị an toàn trong L3 có thể bị nén.

Ràng buộc then chốt không vào lịch sử động, đây là bài học mà giám đốc an toàn AI của Meta đã đổi bằng 200 email.

**Thứ sáu, context quản trị theo "sự chú ý", chứ không phải chất đống theo "dung lượng".**

Đừng theo đuổi nhét thêm nhiều thông tin, mà hãy để model thấy đúng thông tin vào đúng thời điểm. Phân tầng (L1/L2/L3) để model biết thông tin nào có thẩm quyền; cắt bớt + lưu đĩa kiểm soát quy mô input mỗi lần; nén + tập trung (Summary + Todo Recap) quản lý nhiễu của lịch sử dài hạn.

**Thứ bảy, không có observability, thì không có khả năng debug.**

Agent là hệ thống xác suất, không thể lúc nào cũng đúng. Nhưng khi nó sai, bạn cần trả lời được: nó đã làm gì? Kết quả là gì? Tại sao lại làm như vậy?

Triển khai hệ thống Trace, ghi lại chuỗi lệnh gọi, chuỗi kết quả trả về, chuỗi quyết định. Đừng chỉ ghi lại đường thành công, quỹ đạo thất bại thường có giá trị hơn.

**Thứ tám, giữ lại quỹ đạo thất bại, hệ thống mới tiến hóa được.**

Đừng sợ "làm ô nhiễm lịch sử" mà gột rửa bản ghi thất bại. Lỗi CONFLICT, thử lại khi timeout, model đoán mò — tất cả đều ghi lại. Chỉ khi thấy được quá trình thất bại đầy đủ, mới có thể định vị nguyên nhân gốc rễ, mới có thể cố định kinh nghiệm kiểu "gặp CONFLICT phải Read lại" vào prompt.

## Lời cuối: tất cả chúng ta đều đang "dọn dẹp hậu quả" cho LLM

Làm xong dự án này, tôi có một cảm nhận rất sâu sắc, có thể nghe hơi thô, nhưng lời thô mà lý không thô:

**Cốt lõi của phát triển Agent, không phải là để model tự do hơn, mà là thông qua thiết kế kỹ thuật, ràng buộc "năng lực bất định" của model vào "phạm vi có thể kiểm soát tối thiểu". Nói thẳng ra, chúng ta đang dọn dẹp hậu quả cho LLM.**

Vì sao lại nói vậy?

Bạn xem nhé, LLM rất mạnh, biết viết code, biết đọc tài liệu, biết suy luận. Nhưng nó giống như một thực tập sinh cực kỳ thông minh nhưng cực kỳ không đáng tin cậy —

- Bạn bảo nó in file, nó có thể gọi hết tất cả máy in của cả công ty;
- Bạn bảo nó sắp xếp biên bản cuộc họp, nó có thể lẫn cả cuộc họp tuần trước vào;
- Bạn bảo nó viết một hàm, nó viết rất mượt, nhưng đặt tên biến toàn là `a`, `b`, `c`, còn tiện tay sửa cả file mà bạn không bảo nó sửa.

**Cái "mạnh" của nó là mạnh về năng lực, nhưng cái "không đáng tin" là không đáng tin về tính chắc chắn.**

Và chúng ta làm kỹ thuật Agent, bản chất chính là đang giải quyết mâu thuẫn này:

| Bản tính của model | Đối sách kỹ thuật của chúng ta |
|-----------|--------------|
| Thích tự do phát huy | Dùng Function Calling để khóa định dạng gọi |
| Context nhiều lên là "mất trí nhớ" | Dùng phân tầng L1/L2/L3 + nén Summary |
| Sai không tự kiểm tra | Dùng Trace ghi lại từng bước, để lỗi có thể truy vết |
| Task dài dễ chệch hướng | Dùng Todo + Task để tách nhỏ, giảm độ phức tạp từng bước |
| Không hiểu kiến thức chuyên ngành | Dùng Skills để cố định SOP, để nó "có não" |

Bạn xem nội dung bảy chương này, từ nguyên tử hóa tool đến context engineering, từ observability đến sub-agent — **mỗi tầng đều đang "vá lỗi" cho model, giúp nó dọn dẹp bãi chiến trường.**

Nhưng đây lại chính là điều thú vị nhất.

Trước đây tôi nghĩ, trong thời đại AI giá trị của kỹ sư sẽ giảm xuống. Giờ tôi nghĩ ngược lại hoàn toàn: **model càng mạnh, càng cần năng lực kỹ thuật để điều khiển nó.** Giống như động cơ ô tô càng ngày càng mạnh, nhưng khung gầm, phanh, hệ thống treo tốt lại càng quan trọng hơn.

**Chúng ta không phải đang cạnh tranh với model, mà là đang hợp tác với model — nó chịu trách nhiệm "có thể làm gì", chúng ta chịu trách nhiệm "làm sao để nó làm đúng một cách ổn định".**

Vậy nên, nếu bạn hỏi tôi thu hoạch lớn nhất sau khi làm xong dự án này là gì?

Không phải học được kiến trúc cao siêu gì, mà là nghĩ thông suốt một đạo lý giản dị: **Agent xuất sắc không phải là sản phẩm của "để model tự do hơn", mà là kết quả của "ràng buộc sự bất định xuống mức tối thiểu".**

Sự thay đổi nhận thức này, có lẽ đáng giá hơn tất cả mọi dòng code.
