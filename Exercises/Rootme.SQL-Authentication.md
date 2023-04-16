# Rootme.SQL-Authentication

Point: 30

Authentication v 0.01 

Statement: Retrieve the administrator password

# Overview:

Form của challenge là 1 trnag web có chức năng login

![image](https://user-images.githubusercontent.com/115911041/232283783-99a878a7-880e-474e-b3df-5af2b3112e4e.png)

Mục dích là ta cần đăng nhập vào để lấy flag

# Solution:

Vì đây là challenge có liên quan đến SQL injection nên mình thử một vài bypass đơn giản

![image](https://user-images.githubusercontent.com/115911041/232283886-99fa1aa9-1d2f-4f79-9203-3655b7be6632.png)

Sau đó ta đã vào được một cách dễ dàng

![image](https://user-images.githubusercontent.com/115911041/232283920-7fe5b04b-5ef3-4bbb-a804-0e006aab0052.png)

Cuối cùng, view page source ta đã có flag chính là password

![image](https://user-images.githubusercontent.com/115911041/232283988-c4544d3f-9c31-40d1-ab53-15e7db765bd4.png)

`Flag: t0_W34k!$`





