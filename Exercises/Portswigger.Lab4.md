# Lab: SQL injection UNION attack, finding a column containing text

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

# Overview:

Ta có form lab

![image](https://user-images.githubusercontent.com/115911041/231672062-87cc6ab4-069c-4c70-96bd-406e0160450a.png)
 
 Vẫn như lab trước sử dụng UNION attack, nhưng lần này có khác biệt, ở web này có ghi là `Make the database retrieve the string: '8yIXlG'` nghĩa là làm cho database truy xuất ra chuỗi `'8yIXIG'`. Vậy thì khi sử dụng UNION attack ta phải thêm giá trị đó vào ở vị trí thích hợp.
 
 
 # Solution
 
 Sử dụng UNION attack 
 
 `https://0aa0008b0492fe18804f170300e80077.web-security-academy.net/filter?category=Corporate+gifts%27UNION+SELECT+NULL,%278yIXlG%27,NULL--`
 
 ![image](https://user-images.githubusercontent.com/115911041/231673006-4206f53f-4d54-4bae-b214-7fe9382c577a.png)

