# SQL-Blind

Point: 50

Authentication v 0.02

Statement: Retrieve the administrator password.

# Overview:

![image](https://user-images.githubusercontent.com/115911041/234175477-7497d7e8-27d7-44b5-8711-6c8bcbdcc3cd.png)

Form của challenge cho ta trang web có chức năng login.Dự đoán query của bài này

`SELECT ... from ... where username=... and password = ...`

Vậy ta có thê khai thác lỗ hổng ở phần này. Mình sẽ thử fuzz thử ở phần login

`' admin or 1=1--`

![image](https://user-images.githubusercontent.com/115911041/234175844-4bd9a90b-f777-4b84-879f-65b3941f3cb3.png)

Có báo lỗi và ta thấy database bài này đang chạy SQLite3.Mình sẽ thử fuzz thêm nữa

`' or 1=1--`

![image](https://user-images.githubusercontent.com/115911041/234176024-6e5da0f2-79b1-47d9-a831-c735fd7d6983.png)

Adu mình vào được user1 nhưng mà hâu như trong đây không có gì đáng chú ý.

Vậy bài này ta có thể khai thác ở phần login. Challenge lần này là `Blind`. Ý tưởng là ta sẽ cắt các kí tự đầu của password rồi đem đi so sánh, nếu password đó trả về là "Welcome back Admin" thì nó sẽ là password đúng. Vậy ta có thể dùng toán tử [LIKE](https://laptrinhtudau.com/toan-tu-like-trong-sql/) hoặc dùng [SQLmap](https://github.com/sqlmapproject/sqlmap)

# Solution:

Bước đầu cần xác định số column

`admin' order by 2 --` hoặc ` 'order by 3--`

![image](https://user-images.githubusercontent.com/115911041/234177032-44fc97f6-2a26-417e-a452-8c9f3a922b7b.png)

Cái này mình rùa @@ sao vào được admin lunnnn!!!. Nhưng mà không có gì có ích mấy useless vcl. Nhưng kết quả cho ra, thì ta biết là có 2 column.

Bài này sử dụng phương thức POST để truyền truy vấn nên ta sẽ bắt bằng burpsuite.Tại request ta thấy ta truyền vào 2 payload là username và password. Ta sẽ sử dụng sqlmap để thực hiện khai thác bài này.

![image](https://user-images.githubusercontent.com/115911041/234178081-235ce51f-1136-41e7-83c6-e7e1a19e8361.png)

Trước hết copy phần request lưu vào 1 file, ở đây là file bind.txt. Sau đó mở sqlmap lên, tại đây là sử dụng options -r để đọc file. Gõ lệnh:
`python sqlmap.py -r (địa chỉ file blind.txt) --dbs`



