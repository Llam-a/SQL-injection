# SQL-Truncation

Point : 35

SQL Limits

Related resources: [ Blackhat US 2006 : SQL Injections by truncation (Exploitation - Web)](https://repository.root-me.org/Exploitation%20-%20Web/EN%20-%20Blackhat%20US%202006%20:%20SQL%20Injections%20by%20truncation.pdf?_gl=1*139i4zh*_ga*MTE0NDg5MTA5LjE2Nzk2MzIzMDY.*_ga_SRYSKX09J7*MTY4MTYyMzY0My4zOC4xLjE2ODE2MjM2ODkuMC4wLjA.)

# Overview:

![image](https://user-images.githubusercontent.com/115911041/232274744-08a3fcc6-bd18-408f-ade2-14b0abea8c05.png)

Trong challenge lần này, xuất hiện lỗi Truncate ở SQL. Truncate ở đây không phải là func Truncate trong sql mà đó là cắt ngắn chuỗi query bất hợp pháp tạo thành lỗi.

Lần này form của challenge là trang web có chức năng đăng kí, nên mình nghĩ là ta sẽ khai thác lỗi truncate ở phần này.

# Solution:

Bắt đầu viewsource

![image](https://user-images.githubusercontent.com/115911041/232275068-f0e0a858-af68-4c98-869a-285f2c328cd0.png)

Bingo!! Thật sự là có lỗi ở phần đăng kí.

![image](https://user-images.githubusercontent.com/115911041/232275126-62c3887a-88ca-4f3e-b7d4-95b2a8931c72.png)

Nghĩa là, Column login sẽ nhận giá trị username được tạo với độ dài max là 12. Nếu vượt quá 12 thì chuỗi sau vị trí 12 đó sẽ bị cắt bỏ đi mà nhận giá trị trước vị trí 12.

Vậy ta có thể tiến hành ghi đè như sau:

`adminxxxxxxxxxxxxxhackhack`

Bởi vì, độ dài của login là 12, nên từ kí tự 12 trở đi sẽ bị cắt.Là chỉ còn lại là `admin`

![image](https://user-images.githubusercontent.com/115911041/232276597-434244d4-ecf0-4179-a4e2-d94d66306263.png)

`Flag:J41m3Qu4nD54Tr0nc`

[Read more](https://resources.infosecinstitute.com/topic/sql-truncation-attack/#gref)
