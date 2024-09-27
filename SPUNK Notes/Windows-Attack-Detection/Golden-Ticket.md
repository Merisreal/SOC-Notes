Golden Ticket
https://nored0x.github.io/red-teaming/Kerberos-golden-Ticket/
Cuộc tấn công Golden Ticket là một kỹ thuật tấn công trong đó một kẻ tấn công thao túng giao thức xác thực Kerberos được sử dụng trong các mạng Windows để có quyền truy cập không giới hạn vào toàn bộ domain của một tổ chức — bao gồm thiết bị, tệp tin, và các domain controller.

Khi thành công, phương pháp này cho phép các cá nhân trái phép khai thác tài khoản krbtgt, một thành phần quan trọng của hệ thống Kerberos chịu trách nhiệm mã hóa và ký tất cả các ticket trong domain.

Attacker trước tiên xâm nhập vào hệ thống, sau đó leo thang quyền hạn để trở thành domain administrator, từ đó có thể trích xuất NTHash của tài khoản krbtgt và xác định Security Identifier (SID) của domain. Với thông tin này, attacker có thể tạo ra một "Golden Ticket", một Kerberos ticket giả mạo hoạt động như một Ticket Granting Ticket (TGT) hợp lệ, thường sao chép quyền của domain administrator.

Sức mạnh của cuộc tấn công này nằm ở khả năng cung cấp cho attacker quyền truy cập rộng rãi và lâu dài vào mạng, bỏ qua các quy trình xác thực thông thường. Tính bền bỉ của cuộc tấn công xuất phát từ việc nó gây biến dạng cơ bản cho hệ thống Kerberos, và vẫn có hiệu lực ngay cả sau khi thay đổi mật khẩu tài khoản krbtgt. Để vô hiệu hóa hoàn toàn một cuộc tấn công Golden Ticket đang hoạt động, mật khẩu của tài khoản krbtgt cần phải được thay đổi hai lần.

Do đó, kỹ thuật tấn công Golden Ticket nhấn mạnh sự cần thiết của các biện pháp bảo mật toàn diện, chẳng hạn như chính sách mật khẩu nghiêm ngặt và giám sát hoạt động mạng liên tục.


https://www.picussecurity.com/resource/blog/golden-ticket-attack-mitre-t1558.001


### Attack Steps:
+ Trích xuaasat NTlm hast của tài khoản KRBTGT sử dụng -> DCSync (hay NTDS.dit, LSASS dump trên domain controller)
![[Pasted image 202409241856![Pasted image 20240924185608](https://github.com/user-attachments/assets/30c9b333-7124-4281-9e30-8ee1169753c2)
08.png]]

+ Được trang bị hàm băm KRBTGT, kẻ tấn công giả mạo TGT cho một tài khoản người dùng tùy ý, gán cho nó đặc quyền quản trị viên miền

![Pasted image 20240924185629](https://github.com/user-attachments/assets/38524360-76d7-44f7-90cb-0e0b508f2d97)

+ Kẻ tấn công tiêm TGT giả mạo theo cách tương tự như tấn công Pass-the-Ticket.

### Detection
Việc phát hiện các cuộc tấn công Golden Ticket có thể gặp nhiều khó khăn, vì TGT có thể được giả mạo offline bởi attacker mà gần như không để lại dấu vết của việc thực thi Mimikatz. Một phương án là giám sát các phương pháp phổ biến để trích xuất hash của tài khoản KRBTGT:
- **DCSync attack**
- **Truy cập tệp NTDS.dit**
- **Đọc bộ nhớ LSASS trên domain controller** (Sysmon Event ID 10)

Event ID 4769 - A Kerberos Service Ticket was requested.
- Key Description Fields: Account Name, Service Name, Client Address
Event ID 4624 - An account was successfully logged on.
- Key Description Fields: Account Name, Account Domain, Logon ID
Event ID 4627 **-** Identifies the account that requested the logon.
- Key Description Fields: Security ID, Account Name, Account Domain, Logon ID
### SPLUNK

```shell-session
index=main earliest=1690451977 latest=1690452262 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

**=> Lọc 3 event 4768 (A Kerberos authentication ticket (TGT) was requested), 4769(A Kerberos Service Ticket was requested.), 4770: A Kerberos service ticket was renewed, Lọc user và ip, chỉ lọc các sự kiện bắt đầu bằng 4768 mà chưa end, ko lấy các tài khoản của hệ thống**
