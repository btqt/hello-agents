# Hướng dẫn cài đặt Node.js và npx

## 📋 Mục lục

- [Vì sao cần cài đặt Node.js?](#为什么需要安装-nodejs)
- [Hướng dẫn cài đặt trên Windows](#windows-安装教程)
- [Hướng dẫn cài đặt trên macOS](#macos-安装教程)
- [Hướng dẫn cài đặt trên Linux](#linux-安装教程)
- [Xác minh cài đặt](#验证安装)
- [Câu hỏi thường gặp](#常见问题)

---

## Vì sao cần cài đặt Node.js?

Trong phần học về giao thức MCP ở Chương 10, chúng ta cần sử dụng các MCP server do cộng đồng cung cấp, phần lớn các server này được viết bằng JavaScript/TypeScript, nên cần môi trường runtime Node.js.

**Sau khi cài đặt Node.js bạn sẽ có được**:
- ✅ **node**: Runtime JavaScript
- ✅ **npm**: Trình quản lý package của Node (Node Package Manager)
- ✅ **npx**: Công cụ thực thi package npm (tự động tải xuống và chạy package npm)

**Tác dụng của npx**:
```bash
# Cách truyền thống: cần cài đặt trước rồi mới chạy
npm install -g @modelcontextprotocol/server-filesystem
server-filesystem

# Sử dụng npx: tự động tải xuống và chạy (khuyến nghị)
npx @modelcontextprotocol/server-filesystem
```

---

## Hướng dẫn cài đặt trên Windows

### Phương thức 1: Gói cài đặt chính thức (khuyến nghị)

#### Bước 1: Tải gói cài đặt

Truy cập trang chủ Node.js: https://nodejs.org/

Bạn sẽ thấy hai phiên bản:
- **LTS (phiên bản hỗ trợ dài hạn)**: Khuyến nghị cho đa số người dùng ✅
- **Current (phiên bản mới nhất)**: Bao gồm các tính năng mới nhất

**Khuyến nghị tải phiên bản LTS** (ví dụ: 20.x.x LTS)

#### Bước 2: Chạy chương trình cài đặt

1. Nhấp đúp vào file `.msi` đã tải về
2. Nhấp "Next" để bắt đầu cài đặt
3. Chấp nhận thỏa thuận cấp phép (license agreement)
4. Chọn đường dẫn cài đặt (mặc định là được)
5. **Quan trọng**: Đảm bảo đã tích chọn các mục sau:
   - ✅ Node.js runtime
   - ✅ npm package manager
   - ✅ Add to PATH (tự động thêm vào biến môi trường)
6. Nhấp "Install" để bắt đầu cài đặt
7. Chờ cài đặt hoàn tất, nhấp "Finish"

#### Bước 3: Xác minh cài đặt

Mở **PowerShell** hoặc **Command Prompt** (CMD), nhập:

```powershell
# Kiểm tra phiên bản Node.js
node -v
# Sẽ hiển thị: v20.x.x

# Kiểm tra phiên bản npm
npm -v
# Sẽ hiển thị: 10.x.x

# Kiểm tra phiên bản npx
npx -v
# Sẽ hiển thị: 10.x.x
```

Nếu tất cả đều hiển thị số phiên bản bình thường, nghĩa là cài đặt thành công! ✅

---

## Hướng dẫn cài đặt trên macOS

### Phương thức 1: Gói cài đặt chính thức

#### Bước 1: Tải gói cài đặt

Truy cập: https://nodejs.org/

Tải file `.pkg` của **phiên bản LTS**

#### Bước 2: Cài đặt

1. Nhấp đúp vào file `.pkg`
2. Làm theo hướng dẫn của trình cài đặt
3. Nhập mật khẩu quản trị viên (administrator)
4. Hoàn tất cài đặt

#### Bước 3: Xác minh cài đặt

Mở **Terminal**, nhập:

```bash
node -v
npm -v
npx -v
```

---

## Hướng dẫn cài đặt trên Linux

### Ubuntu/Debian

#### Phương thức 1: Sử dụng repository NodeSource (khuyến nghị)

```bash
# Cập nhật danh sách package
sudo apt update

# Cài đặt curl (nếu chưa có)
sudo apt install -y curl

# Thêm repository NodeSource (Node.js 20.x LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Cài đặt Node.js và npm
sudo apt install -y nodejs

# Xác minh cài đặt
node -v
npm -v
npx -v
```

#### Phương thức 2: Sử dụng apt (phiên bản có thể cũ hơn)

```bash
sudo apt update
sudo apt install -y nodejs npm
```

---

### CentOS/RHEL/Fedora

```bash
# Thêm repository NodeSource
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -

# Cài đặt Node.js
sudo yum install -y nodejs

# Xác minh cài đặt
node -v
npm -v
npx -v
```

---

### Arch Linux

```bash
# Sử dụng pacman để cài đặt
sudo pacman -S nodejs npm

# Xác minh cài đặt
node -v
npm -v
npx -v
```

---

## Xác minh cài đặt

### Các bước xác minh đầy đủ

Sau khi cài đặt xong, chạy các lệnh sau để xác minh đầy đủ:

```bash
# 1. Kiểm tra phiên bản
node -v
npm -v
npx -v

# 2. Kiểm thử Node.js
node -e "console.log('Node.js hoạt động bình thường!')"

# 3. Kiểm thử npm
npm --version

# 4. Kiểm thử npx (chạy một package đơn giản)
npx cowsay "Hello MCP!"
```

### Kết quả mong đợi

```
v20.11.0
10.2.4
10.2.4
Node.js hoạt động bình thường!
10.2.4
 _____________
< Hello MCP! >
 -------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

---

## Kiểm thử kết nối MCP server

Sau khi cài đặt xong, hãy kiểm thử kết nối tới MCP server của cộng đồng:

### Kiểm thử filesystem server

```bash
# Sử dụng npx để chạy MCP server hệ thống file (filesystem)
npx -y @modelcontextprotocol/server-filesystem .
```

Nếu thấy thông tin khởi động server, nghĩa là mọi thứ đều ổn!

### Kiểm thử trong Python

Tạo script kiểm thử `test_mcp.py`:

```python
import asyncio
from hello_agents.protocols import MCPClient

async def test():
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."
    ])
    
    async with client:
        tools = await client.list_tools()
        print(f"✅ Kết nối thành công! Các tool khả dụng: {[t['name'] for t in tools]}")

asyncio.run(test())
```

Chạy:

```bash
python test_mcp.py
```

---

## Câu hỏi thường gặp

### Q1: Sau khi cài đặt không tìm thấy lệnh

**Windows**:
```powershell
# Kiểm tra biến môi trường
echo $env:PATH

# Thêm thủ công Node.js vào PATH
# 1. Nhấp chuột phải vào "This PC" -> "Properties"
# 2. "Advanced system settings" -> "Environment Variables"
# 3. Tìm "Path" trong "System variables"
# 4. Thêm: C:\Program Files\nodejs\
```

**macOS/Linux**:
```bash
# Kiểm tra biến môi trường
echo $PATH

# Thêm vào ~/.bashrc hoặc ~/.zshrc
export PATH="/usr/local/bin:$PATH"
source ~/.bashrc  # hoặc source ~/.zshrc
```

---

### Q2: npm chạy rất chậm

Sử dụng mirror trong nước (Trung Quốc, mirror Taobao):

```bash
# Sử dụng tạm thời
npm install --registry=https://registry.npmmirror.com

# Thiết lập vĩnh viễn
npm config set registry https://registry.npmmirror.com

# Xác minh
npm config get registry
```

---

### Q3: Lỗi quyền truy cập (permission) với npx

**Windows**:
```powershell
# Chạy PowerShell với quyền administrator
```

**macOS/Linux**:
```bash
# Không sử dụng sudo để chạy npx
# Nếu gặp vấn đề về quyền, hãy sửa quyền thư mục global của npm
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

### Q4: Xung đột phiên bản

Nếu cần quản lý nhiều phiên bản Node.js, khuyến nghị sử dụng công cụ quản lý phiên bản:

**Windows**: [nvm-windows](https://github.com/coreybutler/nvm-windows)

```powershell
# Sau khi cài đặt nvm-windows
nvm install 20.11.0
nvm use 20.11.0
```

**macOS/Linux**: [nvm](https://github.com/nvm-sh/nvm)

```bash
# Cài đặt nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Cài đặt Node.js
nvm install 20
nvm use 20
```

---

### Q5: Tải package bằng npx rất chậm

```bash
# Cách 1: Sử dụng mirror trong nước
npx --registry=https://registry.npmmirror.com @modelcontextprotocol/server-filesystem

# Cách 2: Cài đặt global trước, rồi sử dụng
npm install -g @modelcontextprotocol/server-filesystem
server-filesystem
```

---

## Bước tiếp theo

Sau khi cài đặt xong, bạn có thể:

1. ✅ Chạy `code/02_Connect2MCP.py` để kiểm thử kết nối MCP client
2. ✅ Khám phá các MCP server của cộng đồng: https://github.com/modelcontextprotocol/servers
3. ✅ Tiếp tục học các nội dung khác của Chương 10

---

## Tài nguyên tham khảo

- **Trang chủ Node.js**: https://nodejs.org/
- **Tài liệu npm**: https://docs.npmjs.com/
- **Tài liệu npx**: https://docs.npmjs.com/cli/v10/commands/npx
- **Danh sách MCP server**: https://github.com/modelcontextprotocol/servers
- **Mirror npm Taobao**: https://npmmirror.com/

---

**Chúc bạn học tập vui vẻ!** 🎉

Nếu có vấn đề, vui lòng tham khảo phần câu hỏi thường gặp hoặc tra cứu tài liệu chính thức.
