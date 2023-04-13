# Lab: SQL injection UNION attack, determining the number of columns returned by the query

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection [UNION attack](https://portswigger.net/web-security/sql-injection/union-attacks) that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

# Overview:

Theo mô tả của đề, ta biết challenge này ta sẽ dùng kĩ thuật tấn công UNION để lấy dữ liệu tử table khác. Dựa theo những kĩ năng từ các lab trước cùng với UNION attack để hoàn thành lab này.

Form của lab này như sau

![image](https://user-images.githubusercontent.com/115911041/231664102-4adcee19-ace4-437f-b545-fe51262fc83e.png)

Và giờ ta sẽ lợi dụng lỗ hổng ở phần filter category bằng UNION attack.

# Soulution:
 
Vào phần category bất kì 

![image](https://user-images.githubusercontent.com/115911041/231665909-af75e449-f7a6-4f22-955e-aa9b64e9f0cd.png)

Sau đó bắt đầu sử dụng UNION Attack.Ở đây ta sẽ dò từng giá trị NULL, nhưng để tiết kiệm thời gian mình cho đáp án.

`https://0af700b504a995d4892bab7c00220083.web-security-academy.net/filter?category=Corporate+gifts%27UNION+SELECT+NULL,NULL,NULL--`

![image](https://user-images.githubusercontent.com/115911041/231668022-f6387699-7e7b-4835-b76b-2a50df2a31f4.png)
