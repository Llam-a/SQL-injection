# Lab: SQL injection vulnerability allowing login bypass

This lab contains a SQL injection vulnerability in the login function.

To solve the lab, perform a SQL injection attack that logs in to the application as the `administrator` user

# Overview:

Theo đề bài, ta cần thực hiện  tấn công SQLi mà đăng nhập vào web là `administrator`

Đây là form của challenge như cũ

![image](https://user-images.githubusercontent.com/115911041/231657900-0cf30e3a-f230-4027-8a3c-c9df470e42a9.png)

Vào My account để thực hiện tấn công SQLi

Ở phần username ta chỉ cần nhập `administrator'--` và để trống hoặc điền bất kì thứ gì ở password.

![image](https://user-images.githubusercontent.com/115911041/231658902-e228f981-38d3-4409-9418-371a0ee822b6.png)

# Explain:

Khi ta đăng nhập bình thường với username `abcd` password `efgh`, web kiểm tra thông tin đăng nhập với truy vần như sau:

`SELECT * FROM users WHERE username = 'abcd' AND password = 'efgh'`

Vậy khi ta log in với username `administrator'--` thì SQL comment `--` sẽ loại bỏ kiểm tra password từ câu lệnh `WHERE` của truy vấn.

