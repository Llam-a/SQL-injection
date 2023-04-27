#  SQL injection with filter bypass via XML encoding

This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

The database contains a users table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.

# Overview

Ta sẽ thực hiện tấn công UNION, để lấy data từ các table khác. Theo hint đề bài ta sử dụng `hackvector`- là một method mà attacker sử dụng để truy cập bất hợp pháp vào network hay máy tính.

# Solution

Bước đầu quan sát `stock` gửi `productID` và `storeID` gửi đến application dưới dạng XML

![image](https://user-images.githubusercontent.com/115911041/234872241-9d7d31a7-558a-4e8c-b3d4-fb9a6ad471ac.png)

Bây giờ mình sẽ inject lệnh SQL đơn giản ở `productID` và `storeID`

![image](https://user-images.githubusercontent.com/115911041/234872850-03a34213-55fe-4d11-bb8b-3a337c11ca1d.png)

## Theory

WAF-Web Application Firewall.Một application như vậy giám sát các yêu cầu và cố gắng tìm lưu lượng truy cập độc hại. Các quy tắc có thể đơn giản như một blacklist với các ký tự hoặc yêu cầu bị cấm. Nó cũng có thể là whitelist chỉ cho phép một số structor tốt đã biết thông qua. Hoặc bất kỳ sự kết hợp nào của chúng.

Đây là một số tài liệu mình tham khảo về XML [XML - Character Entities](https://www.tutorialspoint.com/xml/xml_character_entities.htm) và [XML Character Entities and XAML](https://learn.microsoft.com/en-us/dotnet/desktop/xaml-services/xml-character-entities)

