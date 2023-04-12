# Overview:

- SQL Injection là một kỹ thuật cho phép những kẻ tấn công lợi dụng lỗ hổng của việc kiểm tra dữ liệu đầu vào trong các ứng dụng web và các thông báo lỗi của hệ quản trị cơ sở dữ liệu trả về để inject và thi hành các câu lệnh SQL bất hợp pháp.
- SQL injection có thể cho phép những atacker thực hiện các thao tác trên cơ sở dữ liệu của ứng dụng, thậm chí là server mà ứng dụng đó đang chạy.

# Cause:

- Dữ liệu đầu vào từ người dùng hoặc từ các nguồn khác không được kiểm tra hoặc kiểm tra không kỹ lưỡng.

- Ứng dụng sử dụng các câu lệnh SQL động, trong đó dữ liệu được kết nối với mã SQL gốc để tạo câu lệnh SQL hoàn chỉnh.

# What is the impact of a successful SQL injection attack?

- Tùy vào mức độ tinh vi, SQL Injection có thể cho phép kẻ tấn công :

   - Vượt qua các khâu xác thực người dùng

   - Chèn, xóa hoặc sửa đổi dữ liệu

   - Đánh cắp các thông tin trong CSDL

   - Chiếm quyền điều khiển hệ thống
# Types of SQL Injection :

 SQL Injection có thể chia nhỏ thành các dạng sau:

  - In-band SQLi:

    - Error-based SQLi
   
    - Union-based SQLi
  
  - Inferential SQLi (Blind SQLi)
  
  - Blind-boolean-based SQLi:

    - Time-based-blind SQLi
  
  - Out-of-band SQLi
  
   # How to detect SQL injection vulnerabilities:
  
  - Nhập ký tự `'` và tìm ra lỗi hoặc những cái bất thường khác.
  
  - Nhập các SQL syntax cụ thể để nhận xét những giá trị ban đầu của entry point và các giá trị khác, và tìm các diểm khác biệt của hệ thống trong các phản hồi kết quả của ứng dụng.
  
  - Nhập các toán tử Boolean như là `OR 1=1` hoặc `OR 1=1` , và tìm điểm khác biệt trong phản hồi của ứng dụng.
  
  - Nhập những payload để trigger time delays khi được thực thi trong SQL query và tìm kiếm sự khác biệt trong thời gian phản hồi.

  - Nhập [OAST payload](https://portswigger.net/blog/oast-out-of-band-application-security-testing#:~:text=OAST%20combines%20the%20delivery%20mechanism%20of%20conventional%20DAST,through%20the%20application%E2%80%99s%20processing%20in%20the%20normal%20way.) để trigger tương tác out-of-band network khi thực thi SQL query và giám sát mọi kết quả tương tác.
  
  
 
