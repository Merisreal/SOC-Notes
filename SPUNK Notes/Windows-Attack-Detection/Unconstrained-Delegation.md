## Unconstrained Delegation
https://sec.vnpt.vn/2023/02/kerberos-unconstrained-delegation-attack/

Unconstrained Delegation là một quyền có thể được cấp cho User Accounts hoặc Computer Accounts trong môi trường Active Directory, cho phép một dịch vụ xác thực tới một tài nguyên khác thay mặt cho bất kỳ user nào. Điều này có thể cần thiết khi, chẳng hạn, một web server yêu cầu quyền truy cập vào một database server để thực hiện các thay đổi thay mặt cho một user
![Pasted image 20240924221152](https://github.com/user-attachments/assets/392d3b5f-0ac3-4c2d-81da-6e011e3bcefe)

![Pasted image 20240927152200](https://github.com/user-attachments/assets/d4d3acda-454d-4563-8d60-e2e1962c66e2)

Nếu Delegation được tùy chọn, webserver có thể mạo danh người dùng xác thực và chuyển thông tin xác thực của họ để yêu cầu dữ liệu từ database
-> Nếu attacker có thể dump được TGT ticket của tất cả các tài khoản xác thực từ Unconstrained Delegation Server là một trong những tài khoản Domain Admin, attacker có thể mạo danh để truy cập bất kỳ dịch vụ nào trong Domain trong Domain Admin 
### Attack Steps:
+ Attacker xác định các hệ thống  mà Unconstrained Delegation được bật cho các tài khoản service
![Pasted image 20240924221322](https://github.com/user-attachments/assets/b3c962cf-981d-45a7-b893-e781a7809216)

+ Attacker giành quyền truy cập vào hệ thống mà nconstrained Delegation được bật 
+ Lấy TGT ticket từ bộ nhớ -> Mimikatz
![Pasted image 20240924221439](https://github.com/user-attachments/assets/c2745b78-f2b0-4d4b-8d31-d93f4dce6e69)


### Kerberos Authentication With Unconstrained Delegation

Khi Unconstrained Delegation được bật
+ Điểm khác biệt trong Xác thực Kerberos khi người dùng yêu cầu vé TGS cho service remote, 
+ Domain controller nhúng TGT của user vào service ticket
+ Khi kết nối dịch vụ từ xa, user ko chỉ đưa vé TGS mà còn cả TGT 
+ Khi dịch vụ cần xác thực với dịch vụ khác thay mặt user, nó sẽ đưa vé TGT của user mà dịch vụ đã nhận được cùng với TGS
![Pasted image 20240924221836](https://github.com/user-attachments/assets/b005f303-dfa8-437a-a03b-84478b4d6d51)

![Pasted image 20240927152758](https://github.com/user-attachments/assets/b0e0e374-9bbe-4097-9899-9930821d2099)

1. user xác thực với KDC -> gửi request được mã hóa với thông tin xác thưc, KDC xác thực và gửi lại vé TGT
2. User có TGT -> gửi lại KDC để yêu cầu Server Ticket cho một dịch vụ cụ thể ví dụ web server, KDC kiểm tra TGT và cấp lại TGS (Service Ticket)
3. User dùng TGS để truy cập web dc yêu cầu, tuy nhiên nếu dịch vụ được yêu cầu cần truy cập một dịch vụ khác như SQL thì user phải gửi TGT tới Web service cùng với TGS
4. Web server sẽ lưu trữ cục bộ  TGT vừa được gửi từ user và sử dụng nó để yêu cầu TGS từ KDC để truy cập SQL thay cho người dùng
5. Sau đó KDC sẽ xác minh TGT được gửi và cấp cho Web Server một TGS-SQL để truy cập SQL với tư cách user


### Detection

Powershell command & LDAP search filter dùng cho Unconstrained Delegation có thể được phát hiện
+ Event ID 4104, và LDAP request loggin
+ Mục tiêu chính của cuộc tấn công Ủy quyền không giới hạn là lấy và sử dụng lại các vé TGT, do đó, tính năng phát hiện Pass-the-Ticket cũng có thể được sử dụng.
```
TrustedForDelegation -eq $true
```
### SPlunk

```shell-session
index=main earliest=1690544538 latest=1690544540 source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*TrustedForDelegation*" OR Message="*userAccountControl:1.2.840.113556.1.4.803:=524288*" 
| table _time, ComputerName, EventCode, Message
```
=> Lọc  **EventCode=4104**: Tìm kiếm các sự kiện với mã 4104, sự kiện này liên quan đến việc **chạy lệnh PowerShell** từ xa hoặc việc thực thi một tập lệnh PowerShell. -> tìm  TrustedForDelegation liên quan tới việc tài khoản ủy quyền  -> Lọc các sự kiện mà thông điệp chứa từ khóa liên quan đến thuộc tính `userAccountControl`, cụ thể là mã số 524288, cho biết rằng tài khoản được đánh dấu là **trusted for delegation**.

![Pasted image 20240924222750](https://github.com/user-attachments/assets/2aa6ea0b-5c3e-49f6-a3d0-6d0b6b833882)


