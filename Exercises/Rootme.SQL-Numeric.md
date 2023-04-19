# SQL injection - Numeric

Point: 35

CMS v 0.0.1

Statement: Retrieve the administrator password.

# Overview:

Form challenge

![image](https://user-images.githubusercontent.com/115911041/233044599-db2bf729-52f7-4672-83a5-454c273f20a4.png)

Nhìn sơ thì chẳng có gì đặc biệt. Goal lần này ta phải lấy được content trong table, và password của admin là flag.

# Solution

Ban đầu thì mình có thử 1 số payload đơn giản ở mục login nhưng không có nhiều hi vọng mình chuyển qua link `de news` thì để ý URL

![image](https://user-images.githubusercontent.com/115911041/233045425-53241713-63ba-4685-853f-36c4b5c58db1.png)

Thấy tham số `news_id=1`, mình sẽ thử vài payload đơn giản để xem có lỗi SQL hay không

![image](https://user-images.githubusercontent.com/115911041/233045796-fe66f0e7-f380-46aa-8d1b-d458172c80ec.png)

Là có và xuất hiện báo lỗi `SQLite3`.Kế đó mình sẽ xác định số column

`union select 1,2,3--`

![image](https://user-images.githubusercontent.com/115911041/233046459-8f7c7002-e926-46cc-9ef8-7aecacfb08a3.png)

Sau khi mình thử tới 4, thì nhận được kết quả sau với range là 1-3.Vậy ta đã biết database này đang sài SQLite3 và có 3 columns.

Sau khi tìm hiểu về SQLite3 thì SQLite3 có một table chứa thông tin của tất cả database là sqlite_master. Vậy ta sẽ sử làm 1 query để xác định tên các cột trong database.

`union select type,sql,name from sqlite_master--`

![image](https://user-images.githubusercontent.com/115911041/233047204-8e3a1753-6099-45c2-bd30-8cee372caf51.png)

Lúc này ta xác định được trong database có 2 table là `news` và `users`.Nhưng chúng ta cần table `users` hơn. Table `users` có 2 column `username` và `password`

`union select 1,username,password from users--`

![image](https://user-images.githubusercontent.com/115911041/233047781-7a1b4494-242e-4d21-a27d-fdc5de7c2bd6.png)

`Flag: aTlkJYLjcbLmue3`
