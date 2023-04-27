# Lab: Blind SQL injection with time delays and information retrieval

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.

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

# Solution:

```
SELECT trackingId FROM someTable WHERE trackingId = 'vvv' || (<CODE HERE>) || ''
```

**vvv là giá trị của cookie**

Kí tự đầu tiên được inject vào là `'` sau `vvv`, sau đó mình nối với output của code. Tiếp theo là nối với `'`. Đối với Oracle và PostgreSQL, `||` được sử dụng để ghép nối, với Microsoft `+` và  MySQL là `khoảng trắng`.

