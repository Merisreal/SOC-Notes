## Active Directory 

Công nghệ nền tảng, giúp các network Administrators quản lý và tạo thông qua mạng 
+ Domains
+ Users
+ objects
Cấu trúc gồm 3 lớp chính
+ Domains: tập hợp các objects, như user hay devices, chia sẻ cùng một cở sở dữ liệu
+ Trees: các nhóm của những domain được liên kết bởi một cấu trúc chung
+ Forest: tập hợp của nhiều trees, được kết nối với nhau thông qua trust relationships, lớp cao nhất trong cấu trúc

Một vài key quan trọng trong Active Directory

1. Directory: chứa tất cả các objects trong AD
2. Object: biểu thị cacsc thực thể trong thư mục, có users, groups, shared folder
3. Domain: chứa các directory object, cho phép nhiều domains cùng tồn tại trong một forest, mỗi domains quản lý tập hợp objects riêng của nó
4. Tree: nhóm một domain có chung một root domain
5. Forest: tập hợp của nhiều trees, được kết nối với nhau thông qua trust relationships, lớp cao nhất trong cấu trúc

### Active Directory Domain Service (AD DS)
Các dịch vụ quan trọng trong việc quản lý và giao tiếp trong mạng
+ Domain Service: Tập trung hóa lưu trữ dử liệu, quản lý tương tác giữa các users và domain, authentication và search
+ Certificate Service: quản lý việc tạo, phân phối và quản lý các chứng chỉ số an toàn
+ lightweight Directory Service: hỗ trợ các ứng dụng có khả năn sử dụng directory thông qua giao thức LDAP
+ Directory Federation Service: cung cấp khả năng single-sign-on để xác thực users qua nhiều ứng dụng web trong một phiên
+ Rights Management: hỗ trợ bảo vệ tài liệu bản quyền bằng cách kiểm soát việc phân phối và sử dụng không được phép
+ DNS Server: phân giải tên miền
