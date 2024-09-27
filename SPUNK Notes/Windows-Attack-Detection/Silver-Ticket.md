##  Silver Ticket
https://nored0x.github.io/red-teaming/Kerberos-Attacks-Silver-Ticket/

Mỗi Silver Ticket là một vé TGS giả
Mỗi SIlver Ticket chỉ cho phép attacker giả mạo các vé TGS cho các dịch vụ cụ thể 

Adversaries sở hữu password hash của tài khoản dịch vụ mục tiêu (ví dụ: SharePoint, MSSQL) có thể giả mạo Kerberos Ticket Granting Service (TGS) tickets, còn được gọi là Silver Tickets. Silver Tickets có thể được sử dụng để mạo danh bất kỳ người dùng nào, nhưng phạm vi hạn chế hơn so với Golden Tickets, vì chúng chỉ cho phép attacker truy cập một tài nguyên cụ thể (ví dụ: MSSQL) và hệ thống lưu trữ tài nguyên đó.

**Kerberos Authentication Flow**

1. Mật khẩu được chuyển đổi thành băm **NTLM**, một dấu thời gian được mã hóa bằng băm và gửi đến **KDC** như một bộ xác thực trong yêu cầu vé xác thực (**TGT**).
2. **KDC** kiểm tra thông tin người dùng và tạo **TGT**.
3. **TGT** được mã hóa và gửi đến người dùng. Chỉ có **KRBTGT** trong miền mới có thể mở và đọc dữ liệu **TGT**.
4. Người dùng trình bày **TGT** cho **DC** khi yêu cầu vé **TGS**, **DC** mở **TGT** và xác thực **PAC** checksum → Nếu **DC** có thể mở vé và kiểm tra checksum hợp lệ, **TGT** = hợp lệ. Dữ liệu trong **TGT** được sao chép để tạo ra vé **TGS**.
5. **TGS** được mã hóa bằng mật khẩu băm **NTLM** của tài khoản dịch vụ mục tiêu và gửi đến người dùng.
6. Người dùng kết nối với máy chủ lưu trữ dịch vụ và trình bày **TGS**, dịch vụ mở vé **TGS** bằng mật khẩu băm **NTLM** của nó.

### Attack Steps
**Trích xuất NTLM hash** của tài khoản dịch vụ mục tiêu (hoặc tài khoản máy tính cho quyền truy cập CIFS) bằng các công cụ như Mimikatz hoặc các kỹ thuật trích xuất thông tin đăng nhập khác.
![Pasted image 20240924185856](https://github.com/user-attachments/assets/7794a031-79ed-462d-852c-dd7bb8e68072)

**Tạo Silver Ticket**: Sử dụng NTLM hash đã trích xuất, attacker dùng các công cụ như Mimikatz để tạo một TGS ticket giả mạo cho dịch vụ được chỉ định.
![Pasted image 20240924185907](https://github.com/user-attachments/assets/46cdc0e3-29b5-4637-a514-0d17c28836ee)

**Tiêm TGT giả mạo** theo cách tương tự như cuộc tấn công Pass-the-Ticket.

### Detection

-> Kiểm tra các vé TGS để tìm chẳng hạn:
tên người dùng không tồn tại, các thành viên nhóm đã sửa đổi (thêm hoặc xóa)
+ Trường Account Domain để trống trong khi phải là DOMAIN
+ trường Account Domain có giá trị DOMAIN FQDN khi phải là DOMAIN 
+ 4624 **Account Logon**
- 4634 **Account Logoff**
- 4672 **Admin Logon**

Event ID 4720 (**A user account was created**) có thể giúp xác định những người dùng mới được tạo. Sau đó, chúng ta có thể so sánh danh sách người dùng này với những người dùng đã đăng nhập.

Do không có xác thực về quyền của người dùng, attacker có thể cấp quyền administrator cho các tài khoản. Event ID 4672 (**Special Logon**) có thể được sử dụng để phát hiện những quyền bất thường được gán, từ đó giúp xác định các hoạt động đáng ngờ liên quan đến việc phân quyền quản trị.

### Splunk

Tạo danh sách các users -> Event ID 4720 (A user account was created)

```shell-session
index=main latest=1690448444 EventCode=4720
| stats min(_time) as _time, values(EventCode) as EventCode by user
| outputlookup users.csv
```

**outputlookup users.csv -> xuất truy vấn vào một tệp CSV, chứa các thông tin thời gian sớm nhất, event code**
````shell-session
index=main latest=1690545656 EventCode=4624
| stats min(_time) as firstTime, values(ComputerName) as ComputerName, values(EventCode) as EventCode by user
| eval last24h = 1690451977
| where firstTime > last24h
```| eval last24h=relative_time(now(),"-24h@h")```
| convert ctime(firstTime)
| convert ctime(last24h)
| lookup users.csv user as user OUTPUT EventCode as Events
| where isnull(Events)
````
**Lọc event 4624 (Logon) -> lọc ra những người đăng nhập đầu tiên trong 24h, kiểm tra xem có trong tệp user.csv hay không, -> lọc ra các user ko có sự kiện được ghi trong CSV -> phát hiện tài khoản mới** 
![Pasted image 20240924190322](https://github.com/user-attachments/assets/b0777784-f8ac-4833-834a-9ae6856695f7)

#### Special Privileges Assigned To New Logon
````shell-session
index=main latest=1690545656 EventCode=4672
| stats min(_time) as firstTime, values(ComputerName) as ComputerName by Account_Name
| eval last24h = 1690451977 
```| eval last24h=relative_time(now(),"-24h@h") ```
| where firstTime > last24h 
| table firstTime, ComputerName, Account_Name 
| convert ctime(firstTime)
````

=> Tìm kiếm các sự kiện với mã 4672, sự kiện này liên quan đến việc **cấp quyền nâng cao** (Special Privileges Assigned) cho tài khoản người dùng trong Active Directory -> ghi lại thời gian sớm nhất mà quyền được cấp trong từng tài khoảng trong 24h -> tên, time, user

![Pasted image 20240927151948](https://github.com/user-attachments/assets/292fb019-dc5a-4e7f-a000-909d5990efa2)
