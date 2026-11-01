Ở đây chúng tôi giới thiệu Docker — một trong các phương thức cài đặt local được sử dụng trong dự án, vì cách này ổn định nhất và thuận lợi nhất cho việc tiếp tục khám phá cách sử dụng n8n.

Trước tiên chúng ta vào trang chủ của docker: [Docker: Accelerated Container Application Development](https://www.docker.com/)

Chọn thiết bị terminal của bạn để tải xuống, ở đây minh họa bằng Windows.

![image-20250912025341155](./N8N_INSTALL_GUIDE/image-20250912025341272.png)

Sau khi tải xong, bạn có thể chuyển đổi đường dẫn lưu trữ ổ đĩa, vì image thường rất lớn, nên cố gắng không lưu ở ổ C.

![image-20250912032540657](./N8N_INSTALL_GUIDE/image-20250912032540657.png)

Sau đó mở command line của bạn, nhập lệnh sau để pull n8n

```
docker volume create n8n_data
docker run -d --restart unless-stopped --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```

Bây giờ chúng ta có thể thấy n8n đang chạy trong docker rồi

![image-20250912033251997](./N8N_INSTALL_GUIDE/image-20250912033251997.png)

Nhấp vào 5678:5678 để vào giao diện khởi động của n8n.

![image-20250912033341666](./N8N_INSTALL_GUIDE/image-20250912033341666.png)

Sau khi vào trang, bạn có thể thấy nút mở project mới

![image-20250912034040656](./N8N_INSTALL_GUIDE/image-20250912034040656.png)

Có ba chức năng chính thường được sử dụng
![image-20250912234709064](./N8N_INSTALL_GUIDE/image-20250912234709064.png)

Sau khi mở nút thêm node mới, bạn có thể tìm kiếm node hoặc chọn node mà bạn cần để thêm vào~

![image-20250912234748845](./N8N_INSTALL_GUIDE/image-20250912234748845.png)
