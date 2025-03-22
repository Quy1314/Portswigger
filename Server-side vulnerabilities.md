# Server-side vulnerabilities
## Path traversal
### Path traversal là gì?
- Path traversal hay còn gọi là directory traversal. Nó là những lỗi bảo mật cho phếp kẻ tấn công đọc các files bất kì trên sever đang chạy ứng dụng. Trong một vài trường hợp, kẻ tấn công còn có thể chỉnh sửa hay thêm vào các file mới, thậm chí là chiếm quyền điều khiển sever. 
## Reading arbitrary files via path traversal
Lấy ví dụ một application bán hàng sẽ hiển thị các mặt hàng trên trang web. Hình ảnh có thể được tải lênh thông qua đường dẫn HTML sau:
```html=
<img src="/loadImage?filename=218.png">
```
- ```loadImage``` là một URL nhận ```filename``` làm tham số và trả về tệp hình ảnh được chỉ định.
- Các files ảnh thường được lưu trong đường dẫn ```/var/www/images/...``` và chương trình sẽ gọi API để lấy tệp ảnh đó từ thư mục cơ sở (base directory). Do đó kẻ tấn công có thể dựa trên đường dẫn URL đó để lấy thông tin từ ```/etc/passwd file```.
```url=
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```
## Lab1 : File path traversal, simple case
![image](https://github.com/user-attachments/assets/c534308c-084d-4bdf-9faa-b11fb522bafe)  
Đầu tiên mình sẽ dùng Burpsutie để intercept vào trang web sau đó tìm kiếm các URL có ```filename``` để chèn nhé.
![image](https://github.com/user-attachments/assets/65a56864-204e-40e5-bdc4-9c33cf887a55)
Mình đã tìm được URL cần tìm bây giờ sẽ chèn vào để xem có nhận được dữ liệu từ ```/etc/passwd``` không.
![image](https://github.com/user-attachments/assets/0d6cc7bf-791d-4bad-b621-bb591b83a8d5)
Mình đã thành công nhận được dữ liệu từ ```/etc/passwd```
