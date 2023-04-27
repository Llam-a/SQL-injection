# Lab: Blind SQL injection with conditional errors

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.

# Overview:
 
`SQL injection with conditional errors` là kỹ thuật này khác với các cuộc tấn công SQL Injection thông thường ở chỗ nó không trực tiếp trả về kết quả của câu lệnh SQL độc hại, mà sử dụng các lỗi điều kiện để xác định kết quả của truy vấn SQL bằng cách đưa ra các câu hỏi `True` hay `False`

Query được sử dụng trong lab

```
SELECT trackingID FROM someTable WHERE trackingID = '<COOKIE-VALUE'>
```

Mình sẽ bỏ qua hoàn toàn content của cookie từ bây giờ và chỉ cung cấp chuỗi được thêm vào giá trị TrackingId.

Bài này mình làm 2 cách `sqlmap` và `Portswigger`

# Solution:

# PortSwigger:

Kiểm tra `TrackingID` cookie bằng cookie editor. Thêm kí tự `'` thì gây ra lỗi

![image](https://user-images.githubusercontent.com/115911041/234747603-d319fdfc-f2f3-4847-b3fb-01ee3c4b0d6f.png)

Bởi vì loại lỗ hổng này, chúng ta không thể thấy bấy kì kết quả nào của query cũng như sự khác biệt ở output. Tuy nhiên, nếu ta có thể tạo ra các requests gây ra lỗi database, ta có thể suy ra giả trị thật dựa trên việc lỗi có hiển thị ra hay không.

Vậy bước đầu ta cần xác định tham số nào là lỗ hổng bằng cách tạo ra các request mà đưa ra câu trả lời `normal` hoặc `error`.

Inject 1 kí tự `'` tạo ra `server error`, inject 2 kí tự `'` thì không xảy ra. Điều này cho thấy khả năng SQLi, vì thề mình thử vài payload:

`'||(SELECT+null||'` cũng cho ra lỗi, tuy nhiến nếu là  `'||(SELECT+null+FROM_dual)||'` thì không. Vì thế database đang sử dụng là Oracle DB.

***Toán tử `||` cho phép nối chuỗi dữ liệu và kết quả truy vấn. `()` được sử dụng để nhóm các điều kiện hoặc phép tính lại với nhau.***

## TÌm lệnh 'normal' và 'error'

Dựa vào `cheat sheet` mà Lab cung cấp, một ví dụ về conditional error 

`SELECT CASE WHEN (<CONDITION>) THEN to_char(1/0) ELSE NULL END FORM DUAL`

Với `normal` query, chúng ta lấy `input` là `FLASE`, dó đó nó không xảy ra lỗi `divison by zero`

```
SELECT trackingId FROM someTable WHERE trackingId = X'||(SELECT CASE WHEN (1=2) THEN to_char(1/0) ELSE NULL END FROM dual)||'
```

Content của cookie cho `normal` case được cho ra một trang web `normal`:

`'||(SELECT CASE WHEN (1=2) THEN to_char(1/0) ELSE NULL END FROM dual)||'`

Trường hợp `error` có condition được cho là true, do đó khiến cho databse thực hiện `division by zero` dẫn đến `internal server error`:

`'||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE NULL END FROM dual)||'`

Giờ chúng ta đã có cách phân biệt kết quả của boolean condition.

## Xác định database table và columns

Bước tiếp theo là xác định table `users` có thật sự tồn tại ở trong database. Với cái này, mình sử dụng database version oracle giới hạn lượng row output:

`'||(SELECT username||password FROM users WHERE rownum=1)||'`

Nó cho thấy trang web, xác định đó là query hợp lệ và cũng như table name và cả column names đều hợp lệ.

## Xác định user tồn tại trong database

Để gây ra lỗi, chúng ta mở rộng ví dụ bằng cách thực hiện `division by zero` nếu tên người dùng không tồn tại. Câu lệnh này gây ra `internal server error`, chĩ ra rằng query này dẫn đến `division by zero`

`'||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE NULL END FROM users WHERE username='administrator')||'`

Thay đổi username thành một giá trị tùy ý, page được hiển thị. Điều này xác định rằng điều kiện `1=1` và `division by zero` phụ thuộc vào việc `FROM users WHERE...`  một phần trả về thứ gì đó.

Điều này xác định `administrator` tồn tại trong database.

## Xác định chiều dài của password

Xác định chiều dài rất dễ dàng bằng cách sử dụng hàm `LENGTH` ở cuối câu điều kiện SQL.

`'||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE NULL END FROM users WHERE username='administrator' and LENGTH(password)=1)||'`

Sẽ hiển thị ra page, chỉ ra rằng kcheck `1=1`  chưa bào giờ được thực hiện. Sử dụng Burp Intruder, mình kiểm tra các số từ 1 đến 50 dưới dạng độ dài





