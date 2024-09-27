## PASS THE TICKET

Các tickets trong Windows được quản lý và lưu trữ bởi process lsass (Local Security Authority Subsystem Service), chịu trách nhiệm xử lý các chính sách bảo mật. Để trích xuất các tickets này, cần phải tương tác với process lsass. Người dùng không có quyền quản trị chỉ có thể truy cập các tickets của riêng mình, trong khi người quản trị có quyền trích xuất tất cả các tickets trên hệ thống

Pass-the-Ticket (PtT) là một kỹ thuật lateral movement được sử dụng bởi attacker để di chuyển ngang trong một mạng bằng cách lợi dụng Kerberos TGT (Ticket Granting Ticket) và TGS (Ticket Granting Service) tickets. Thay vì sử dụng NTLM hashes, PtT tận dụng các Kerberos tickets để xác thực với các hệ thống khác và truy cập vào tài nguyên mạng mà không cần biết mật khẩu của người dùng. Kỹ thuật này cho phép attacker di chuyển ngang và truy cập trái phép vào nhiều hệ thống.

### ATTACK STEPS:
+ Attacker dành được quyền administrator vào hệ thống
+ Sử dụng Mimikatz hay rubeus để lấy TGT hay TGS ticket 
![Pasted image 20240926185321](https://github.com/user-attachments/assets/b223d5aa-2825-4f02-8cdd-863f5451070f)

+ Kẻ tấn công gửi vé đã trích xuất cho phiên đăng nhập hiện tạI -> có thể xác thực với các hệ thống và tài nguyên mạng khác mà không cần mật khẩu văn bản gốc
![Pasted image 20240926185329](https://github.com/user-attachments/assets/07d1edeb-0aa1-4f7b-b39a-d3591eac50db)

### Kerberos Authentication Process

+ Client yêu cầu TGT từ KDC
+ KDC xác minh danh tính người dungfg thông qua password
+ KDC cấp một TGT được mã hóa bằng secret key của user
+ TGT này có hiệu lực trong một khoảng thời gian (thường là 8 tiếng) -> cho phép người dùng yêu cầu service ticket mà ko cần phải xác thực
+ Client gửi yêu cầu service ticker (TGS-REQ)  -> KDC bằng TGT đã nhận được
+ KDC xác thực TGT của client -> thành công -> cấp TGS được mã hóa bằng sercret key của tài khoản dịch vụ chứa thông tin của client cùng với session key
+ Client có TGS -> kết nối với server và gửi TGS cho server
![Pasted image 20240920170012](https://github.com/user-attachments/assets/e06b9237-20c3-419c-8bd5-38031a62b1e0)

### DETECTION 

+ EventID 4648: (Explicit Credential logon attemps): logon với user password
+ EventID 4624 (logon): đăng nhập thành công
+ EventID 4768 (Kerberos TGT request): Client yêu cầu TGT trong quá trình xác thực Kerberos
+ EventID 4769 (Kerberos Service Ticket Request): Client yêu cầu vé TGS ticket
![Pasted image 20240926185422](https://github.com/user-attachments/assets/ada6b811-b902-4c96-9458-b7a371a32993)

Tuy nhiên, tùy thuộc vào ngữ cảnh:
1. Attacker có thể nhập một TGT họ đã có sẵn vào logon session thay vì yêu cầu từ KDC -> ko có log EventID 4768
->  Cho nên xem xét khi có Event ID 4769 hoặc EventID 4770(TGS Request hoặc TGS Renewal), nhưng lại không có EventID 4768 trong một time nhất định 
2.  Xem xét EventID 4769 (TGS Request) 
ví dụ
**Sự kiện bình thường**:
- **Source IP**: 192.168.1.80 (máy tính của người dùng hợp lệ).
- **Service**: HR-Service@domain.com (dịch vụ nhân sự).
- **Host ID**: HR-Service@domain.com.
- **Destination IP**: 192.168.1.50 (máy chủ dịch vụ nhân sự).
**Sự kiện bất thường** (Pass-the-Ticket):
- **Source IP**: 192.168.1.100 (máy tính của attacker).
- **Service**: HR-Service@domain.com (dịch vụ nhân sự).
- **Host ID**: HR-Service@domain.com.
- **Destination IP**: 192.168.1.50 (máy chủ dịch vụ nhân sự).
Trong trường hợp này, attacker đang yêu cầu dịch vụ từ **192.168.1.100**, nhưng Kerberos ticket của họ lại liên quan đến **HR-Service@domain.com**, dịch vụ trên **192.168.1.50**. Điều này có thể được phát hiện thông qua việc tìm sự không khớp giữa **Source IP** và **Host ID** trong **Event ID 4769**.

3. **Event ID 4771 (Pre-Authentication Failed)** : Trong trường hợp này, **attacker** có thể đã gửi một yêu cầu **AS-REQ** giả mạo với **timestamp** mã hóa không hợp lệ. KDC không thể giải mã và phát hiện lỗi trong quá trình **Pre-Authentication**, dẫn đến việc ghi lại **Failure Code 0x18** trong **Event ID 4771**.

### SPLUNK

```shell-session
index=main earliest=1690392405 latest=1690451745 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

**=> Lọc Các event ID 4768, 4766, 4770, user ko phải của tài khoản hệ thoosgn user!='*'$ -> lấy user, ip** 
**gộp transaction username và ip trong time 10h các giao dịch đang mở mà ko end sẽ được vào kết quả** 


![Pasted image 20240920172424](https://github.com/user-attachments/assets/fa726a53-3d91-4a27-a644-cade0b225d48)
