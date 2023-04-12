# Overview:

- Đây là dạng tấn công phổ biến nhất và cũng dễ để khai thác lỗ hổng SQL Injection nhất

- Xảy ra khi hacker có thể tổ chức tấn công và thu thập kết quả trực tiếp trên cùng một kênh liên lạc

- In-Band SQLi chia làm 2 loại chính:

  - Error-based SQLi

  - Union-based SQLi

# Error-based SQLi:

- Là một kỹ thuật tấn công SQL Injection dựa vào thông báo lỗi được trả về từ Database Server có chứa thông tin về cấu trúc của cơ sở dữ liệu.

- Trong một vài trường hợp, chỉ một mình Error-based là đủ cho hacker có thể liệt kê được các thuộc tính của cơ sở dữ liệu

![image](https://user-images.githubusercontent.com/115911041/231446106-5cff8a0b-112f-4acc-84d1-7c14c35e261c.png)

# Union-based SQLi:

- Là một kỹ thuật tấn công SQL Injection dựa vào sức mạnh của toán tử UNION trong ngôn ngữ SQL cho phép tổng hợp kết quả của 2 hay nhiều câu truy vấn SELECTION trong cùng 1 kết quả và được trả về như một phần của HTTP response

![image](https://user-images.githubusercontent.com/115911041/231447500-ff09141d-aedc-4662-ae5d-904815b1ee46.png)

Read more:

[Retrieving hidden data](https://portswigger.net/web-security/sql-injection#retrieving-hidden-data)

[SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection)

[Subverting application logic](https://portswigger.net/web-security/sql-injection#subverting-application-logic)
