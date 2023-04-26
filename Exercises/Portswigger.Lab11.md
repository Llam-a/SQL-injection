# Lab: Blind SQL injection with conditional errors

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.

# Overview:
 
`SQL injection with conditional errors` là kỹ thuật này khác với các cuộc tấn công SQL Injection thông thường ở chỗ nó không trực tiếp trả về kết quả của câu lệnh SQL độc hại, mà sử dụng các lỗi điều kiện để xác định kết quả của truy vấn SQL bằng cách đưa ra các câu hỏi `True` hay `False`

Bài này mình làm 2 cách `sqlmap` và `BruteForce`

# Solution:

## SQLmap

Đầu tiên mình test `TrackingID` bằng `cookie editor`. Thêm kí tử `'` gây ra lỗi. Mình sử dụng SQLmap để `dump` password administration ở mục này

![image](https://user-images.githubusercontent.com/115911041/234613134-d6a2ab8e-f684-4072-a5a0-63f4de759730.png)

Trước hết copy lưu vào `/`  vào file, ở đây là file `error.txt`. Mình tạo 1 payload để lấy password của administrator. Mình phải sử dụng `--random-agent` để cho sqlmap chạy đúng mục dích. Và sau khi hỏi a Khánh có cách nào sqlmap chạy nhanh hơn, anh Khánh có gợi í là dùng Thread. Để nó nhanh hơn thì mình cần thăng Thread lên.

```
p [test parameter(s)] Testable parameter(s)

--level=2 Level 2 is required to test cookies

--current-db Retrieve DBMS current database

--threads [THREADS] Max number of concurrent HTTP(s) requests (default 1)

--random-agent Use randomly selected HTTP User-Agent header value.

--batch Never ask for user input, use the default behavior.

-D [database] DBMS database to enumerate

-T [table(s)] DBMS database table(s) to enumerate

-C [column(s)] DBMS database table column(s) to enumerate
```

[SQLmap wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

