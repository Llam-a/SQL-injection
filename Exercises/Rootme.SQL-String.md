# SQL-String

Point: 30

CMS v 0.0.2

Statement: Retrieve the administrator password

# Overview:

Form của challenge

![image](https://user-images.githubusercontent.com/115911041/233056520-36ff56d1-3756-4660-a9d0-0614a2b97d06.png)

Goal thì như mọi lần lấy được password của administrator

# Solution

Nhìn sơ qua, ta có thấy vài điểm giống với challenge Numeric, nên mình thử vài payload đơn giản ở các link. Nhưng mà không có nhiều hi vọng lắm. Mình thử qua `Login` có tìm kiếm được gì không, và không. Chỉ còn lại là `Search`.
Và hình như đã có chút kết quả khi mình nhập `'or 1=1--`

![image](https://user-images.githubusercontent.com/115911041/233057421-ea6c9713-b7ea-4c30-9c38-1e3215610900.png)

Bởi vì `1=1` là đúng nên kết quả trả về các link, có khá nhiều hi vọng, cũng gần như gần tới được flag rồi.Mình tiếp tục thử một số payload khác, cho đến khi mình nhập `'admin --` thì có xuất hiện lỗi

![image](https://user-images.githubusercontent.com/115911041/233065104-e3c98112-20b5-431c-a247-3932704624d4.png)

Và ta cũng biết database đang sài SQLite3 khá giống bài SQL numeric.Nên mình thử sử dụng payload bài đó thử xem
`union select username,password from users --`

![image](https://user-images.githubusercontent.com/115911041/233065961-4c45406e-86e1-4b85-a8ef-d5db588ad2fd.png)
 
 `Flag: c4K04dtIaJsuWdi`
