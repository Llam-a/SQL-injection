# Lab: SQL injection UNION attack, retrieving multiple values in a single column

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

# Overview

Lab này ta có database bao gồm các table khác nhau gọi là users, với các cột là `username` và `password`. Ta vẫn sẽ tiếp tục thực hiện UNION Attack để hoàn thành.

Form của Lab

![image](https://user-images.githubusercontent.com/115911041/232038241-313ab141-953c-48da-aa93-e5d343ad201c.png)

# Solution:

Ta cần xác định số cột được trả về bởi cuộc truy vấn và cột nào có chứa dữ liệu dạng text.Sử dụng payload

`+UNION+SELECT+NULL,'a'--`

![image](https://user-images.githubusercontent.com/115911041/232038713-d1622cfe-538c-4212-9941-7e755951b3fc.png)

Vậy là gồm 2 cột và chỉ có 1 cột gồm dữ liệu dạng text.Và giờ ta chỉ cần sử dụng payload này để lấy contents từ table `users`.Bởi vì theo đề là `retrieving multiple values in a single column` nghĩa là ta lấy các dữ liệu ở mỗi cột, nên ta phải dùng payload khác.

Đây là kiểu payload mình dùng `PostgreSQL`

![image](https://user-images.githubusercontent.com/115911041/232040378-c2330682-ee04-413f-bfaf-248d42d0bb19.png)


`'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`

![image](https://user-images.githubusercontent.com/115911041/232039923-2238a3ac-f546-4643-97bb-c47b618b860b.png)

Theo đề là log in vào `administrator`

![image](https://user-images.githubusercontent.com/115911041/232040118-afc8a410-397d-466e-9a1a-f4aaafad35d1.png)

[SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) sẽ cung cấp cho payload khác nhau.



