# Lab: Blind SQL injection with time delays

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay.

# Overview:

Theo tiêu đề của lab, chúng ta sẽ thực hiện tấn công SQL mà kết quả trở về có độ trễ. Từ đó ta có thêm thời gian thực thi câu lệnh SQL để thực hiện các cuộc tấn công khác.

Goal của chúng ta là cần inject query đối với time delays thích hợp.

Cheat sheet, time-delay mà lab đưa ví dụ khá là nhiều cho các version database. Vì ko biết sử dụng cái nào mình phải mò từng cái

```
Oracle: dbms_pipe.receive_message(('a'),10)
Microsoft: WAITFOR DELAY '0:0:10'
PostgreSQL: SELECT pg_sleep(10)
MySQL: SELECT sleep(10) 
``` 

# Solution

```
SELECT trackingId FROM someTable WHERE trackingId = 'vvv' || (<CODE HERE>) || ''
```

**vvv là giá trị của cookie**

Kí tự đầu tiên được inject vào là `'` sau `vvv`, sau đó mình nối với output của code. Tiếp theo là nối với `'`. Đối với Oracle và PostgreSQL, `||` được sử dụng để ghép nối, với Microsoft `+` và  MySQL là `khoảng trắng`.

Mình sử dụng Burp Repeater để thử các payload khác nhau. Đồi với PostgreSQL, query cần thời gian giải để hoàn thành: 10.241s(bạn có thể thấy ở góc phải màn hình). Thường thì query mất khoảng 240millis.

![image](https://user-images.githubusercontent.com/115911041/234788992-81be2926-549d-4b40-86a8-8605b42b80cb.png)

Reload lại page thì ta đã complete được lab

![image](https://user-images.githubusercontent.com/115911041/234790150-3b64c1df-ffb7-4610-9148-fe00e8159198.png)


