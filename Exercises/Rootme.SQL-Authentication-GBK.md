# SQL injection - Authentication - GBK

Point: 30

Statement: Get an administrator access.

# Overview:

![image](https://user-images.githubusercontent.com/115911041/232288993-86d49422-ebb6-46e6-8338-966a37534099.png)

Challenge lần này cho ta một form login.Vậy ta cần login vào để lấy được flag

# Solution:

Đầu tiên ta thử bypass đơn giản ` or '1'='1'--`, nhưng không xảy ra gì cả. Vậy thì ta đổi luôn query thành `login[]=admin&password[]=admin`

![image](https://user-images.githubusercontent.com/115911041/232289478-663571cf-fbde-442e-940c-9b0396ebf16b.png)

Và ta thầy lúc này web báo lỗi là hàm `addslashes()` cho username mà `md5` cho password

Hàm `addslashes` sẽ add thêm dấu `/` trước kí tự `'` nên câu lệnh fuzz của ta không thực hiện được.Ví dụ:

![image](https://user-images.githubusercontent.com/115911041/232289706-5d1b1d86-ee38-4e73-a04c-b5f86e0991d4.png)

Tiếp theo ta cần hiểu [BGK](https://en.wikipedia.org/wiki/GBK_(character_encoding). Vậy GBK là môt bảng kí tự của Trung Quốc, khá đúng với những gì đề bài cho

Và bây giờ ta cần tìm cách bypass hàm `addslashes` này.Mình có tham khảo một vài bài viết [ở đây](https://www.securityidiots.com/Web-Pentest/SQL-Injection/addslashes-bypass-sql-injection.html).Theo bài viết đó ta chỉ cần thêm `%af` hay `%bf` vì khi ta thực thi thì nó sẽ thêm `%5c` vào và `%af%5c`, `%bf%5c` sẽ ra kí tự hợp lệ trong bảng BGK.

Vậy ta có payload như sau:

`login=%af’ or 1=1 -- -&password=1234 hoặc

login=%bf’ or 1=1 -- -&password=1234`




