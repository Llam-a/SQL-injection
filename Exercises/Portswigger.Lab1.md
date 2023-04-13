# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

To solve the lab, perform a SQL injection attack that causes the application to display details of all products in any category, both released and unreleased.

# Overview:

Ta có form challenge như sau

![image](https://user-images.githubusercontent.com/115911041/231650323-d6335e6a-e23c-498d-a5d3-9c33aa2a7e3a.png)

Nhìn sơ qua đây là một trang web mua sắm. Mặc dù theo đề của lab lần nay ta đã biết là có lỗ hổng ở filter các loại sản sản phẩm, mình vẫn sẽ thực hành cách phát hiện lỗi sql bằng cách thêm kí tự `'` vào cuối phần category.

`https://insecure-website.com/products?category=Gifts'--`

![image](https://user-images.githubusercontent.com/115911041/231651630-2edbe342-2f75-49be-b062-a5861e6efdea.png)

Web sẽ hiện ra `Internal Server Error` như là có cảnh báo lỗi sql.

# Solution:

Và giờ ta đã biết trang web có lỗ hổng ở phần filter các loại sản phẩm, ta bắt đầu chèn toán tử `'OR+1=1--`.

`https://0aac00d604fa0f49853c5d5800780031.web-security-academy.net/filter?category=Gifts%27OR+1=1--`

![image](https://user-images.githubusercontent.com/115911041/231652112-331af09b-adb8-4fb2-b8b7-1f75a671cb51.png)

Và ta đã hoàn thành challenge.

# Explain:

Đây là URL khi bạn chọn vào Gifts category:

`https://insecure-website.com/products?category=Gifts`

Điều này khiến ứng dụng thực hiện truy vấn SQL để truy xuất chi tiết của các sản phẩm có liên quan đến database:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Truy vấn này yêu cầu database trả về:

  - all details (*)

  - from the products table
     
  - where the category is Gifts

  -  and released is 1.

Giới hạn `released = 1` dùng để ẩn các sản phẩm không được release. Cho những sản phẩm unrelease, thì thường là `release = 0 `.

Vậy khi mà mình thêm toán tử Boolean `'OR+1=1--` ở category thì có nghĩ là:

`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

Điều quan trọng ở đây là chuỗi dấu gạch ngang kép `--` là một chỉ số nói rằng đó là comment trong SQL, và nó có nghĩ là tất cả phẩn còn lại của cuộc truy vấn đều là thuộc comment. Điều này có hiệu quả loại bỏ phần còn lại của truy vấn, vậy nó không còn bao gồm `AND released = 1`. Tất cả các sản phẩm sẽ được hiện lên, bao gồm cả sản phẩm `unreleased`.

Tận dụng điều đó ta chèn thêm toán tử vào. Vậy sẽ trả về tất cả sản phẩm trong đó category là gifts hoặc là 1=1. Bởi vì 1=1 là luôn đúng, cuộc truy vấn sẽ trả về tất cả sản phẩm.

