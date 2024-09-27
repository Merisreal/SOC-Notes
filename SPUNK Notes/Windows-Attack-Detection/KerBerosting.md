Thu nhập password Hash của một tài Khoản ACTIVE DIRECTORY có Servicee Principal Name (SPN)
+ Một domain user đã được xác thực yêu cầu một Kerberos Ticket cho một SPN -> tools GhostPack's, Ruberus, SecureAuth Corporation’s GetUserSPNs.py.
+ Kerberos Ticket được trả về được mã hóa bằng password hast của tài khoản service liên kết với SPN đó(SPN kết nối một service với một user account trong AD)
+ Bruteforce Offline  -> pass
=> Kerberosting không yêu cầu tài khoản Domin Admin, hay có quyền cao -> vì bất cứ domain user nào cũng yêu cầu service ticket từ TGS

https://www.crowdstrike.com/cybersecurity-101/kerberoasting/

https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast
Tool : [Rubeus](https://github.com/GhostPack/Rubeus) `kerberoast` module.
![Pasted image 20240918174215](https://github.com/user-attachments/assets/e4abf298-4ed3-44fb-9e96-f9ce8bc6e440)


Kerberoasting nhắm vào các dịch vụ account trong AD để trích xuất và bẻ khóa mật khẩu -> khai thác các mật khẩu và mã hóa yếu

### ATTACK STEPS:

**Identify Target Service Accounts**: Attacker liệt kê Active Directory để xác định các service accounts có Service Principal Names (SPNs) được đặt. Service accounts thường liên quan đến các dịch vụ chạy trên mạng, chẳng hạn như SQL Server, Exchange, hoặc các ứng dụng khác. Đoạn mã sau đây từ Rubeus liên quan đến bước này. 
![Pasted image 20240925230150](https://github.com/user-attachments/assets/08891010-17f3-4ef1-995d-c2b1a09fc168)


**Request TGS Tickets**: Attacker sử dụng các service accounts đã được xác định để yêu cầu Ticket Granting Service (TGS) tickets từ Key Distribution Center (KDC). Các TGS tickets này chứa các hash mật khẩu được mã hóa của service account. 
![Pasted image 20240925230158](https://github.com/user-attachments/assets/9084f9f7-abfc-4fd3-a5e6-ec06f9244139)


**Offline Brute-Force Attack**: Attacker sử dụng các kỹ thuật brute-force offline, sử dụng các công cụ bẻ khóa mật khẩu như Hashcat hoặc John the Ripper, để cố gắng crack các hash mật khẩu đã được mã hóa.

### Benign Service Access Process 
Các sự kiện liên quan khi user connect database với SPN
+ TGT Request: client bắt đầu quy trình xác thực bằng cách yêu cầu **Ticket Granting Ticket (TGT)** từ **Key Distribution Center (KDC)** 
+ TGT Issue: KDC xác định danh tính (pass hash) -> cấp mã hóa bằng khóa bí mật của client, có hiệu lực trong thời gian nhất định 
+ Service Ticket Request: Client gửi yêu cầu vé dịch vụ (TGS-REQ) -> KDC cho SPN Của máy chủ MSSQL bằng TGT đã nhận được ở bước trên
+ Service Ticket Issue: KDC xác thực TGT client -> thành công -> cáp TGS được mã hóa bằng khóa bí mật của dịch vụ -> chứa danh tính và một session key -> CLient có TGS
+ Client Connection: Client kết nối MSSSQL servers và gửi TGS đến máy chủ
+ MSSQL Server Validates the TGS: MSSQL giải mã TGS bằng khóa bí mật của nó dể lấy session key và danh tính client, -> hợp lệ -> key đúng -> kết nối được chấp nhận 
![Pasted image 20240918183912](https://github.com/user-attachments/assets/a59aeefd-9303-489e-a482-ebd25816978d)

Wireshark:
![Pasted image 20240918183921](https://github.com/user-attachments/assets/a76cc4fd-2c87-466b-8e7b-2ec4b086f653)

### DETECTION
-> ví dụ với MSSSQL server
- `Event ID 4768 (Kerberos TGT Request)`:. Khi User request TGT từ KDC, 
- `Event ID 4769 (Kerberos Service Ticket Request)`: Tạo sau khi user nhận TGT và request TGS cho MSSQL 
- `Event ID 4624 (Logon)`:  Log vào MSSQL server bằng sử dụng service account với SPN được thiết lập để kết nối
![Pasted image 20240918184939](https://github.com/user-attachments/assets/a49b8f27-3369-4ea1-812c-6d58dd19036f)

1- Event ID 4769: Event log ID number 4769 is detected.

2- Service Name not equal to 'krbtgt': Brings services whose service name is not the same as 'krbtgt'.

3- Service Name does not end with $ : Lists services whose Service Name does not end with '$'.

4- Account Name does not match ' $@ ': Account name ' $@ Returns accounts that do not match '.

5- Failure Code is '0x0': Error Code '0x0' lists the codes.

6- Ticket Encryption Type is '0x17': Returns Ticket Encryption Type 0x17.

### Splunk

Example with service iis
#### Benign TGS Requests

```
index=main earliest=1690388417 latest=1690388630 EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc) 
| dedup RecordNumber 
| rex field=user "(?<username>[^@]+)"
| table _time, ComputerName, EventCode, name, username, Account_Name, Account_Domain, src_ip, service_name, Ticket_Options, Ticket_Encryption_Type, Target_Server_Name, Additional_Information
```

| dedup RecordNumber : loại các record trùng lặp
``| rex field=user "(?<username>[^@]+)": john.doe@example.com -> john.doe

**=> lọc EventCode 4648 ( explicit credentials đăng nhập ko phải người dùng hiện tại) HOẶC Event ID 4769 (Kerberos Service Ticket Request) và log tới servicename -> loại các Record trùng -> lọc lấy user trước**
**@ và lưu vào username,**``
![Pasted image 20240919160412](https://github.com/user-attachments/assets/cac477a4-f826-44af-97cf-3c4866a061d6)

#### Detecting Kerberoasting - SPN Querying
```shell-session
index=main earliest=1690448444 latest=1690454437 source="WinEventLog:SilkService-Log" 
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter 
| search SearchFilter="*(&(samAccountType=805306368)(servicePrincipalName=*)*"
```


+ spath input=Message : dùng spatch lấy thông tin từ cấu trúc XML và Json vì nó.. khó nhìn
![Pasted image 20240916174334](https://github.com/user-attachments/assets/11c1cc85-dda0-46c9-b3aa-90a9b9c6958e)

+ rename XmlEventData.* as * 
Giả sử bạn có dữ liệu trong trường `XmlEventData` như sau:
`<XmlEventData>     <Field1>Value1</Field1>     <Field2>Value2</Field2> </XmlEventData>`
Khi sử dụng `| rename XmlEventData.* as *`, trường `Field1` và `Field2` sẽ được đổi tên thành các trường chính như sau:
`Field1: Value1 Field2: Value2`
Thay vì phải truy cập các trường thông qua `XmlEventData.Field1` và `XmlEventData.Field2`,  có thể truy cập trực tiếp bằng `Field1` và `Field2`



**=> trích xuất các dữ liệu từ JSON hoặc XML trong trường message ,Đổi tên các trường có tiền tố "XmlEventData." thành các trường không có tiền tố (để dễ thao tác hơn), hiện bảng theo thời gian,**** 

![Pasted image 20240925231420](https://github.com/user-attachments/assets/63037f07-5a11-4983-8442-a93f72c99a8a)

#### Detecting Kerberoasting - TGS Requests

```shell-session
index=main earliest=1690450374 latest=1690450483 EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| bin span=2m _time 
| search username!=*$ 
| stats values(EventCode) as Events, values(service_name) as service_name, values(Additional_Information) as Additional_Information, values(Target_Server_Name) as Target_Server_Name by _time, username
| where !match(Events,"4648")
```

**=> lọc EventCode 4648 ( explicit credentials đăng nhập ko phải người dùng hiện tại) HOẶC Event ID 4769 (Kerberos Service Ticket Request) và log tới servicename -> loại các Record trùng -> lọc lấy user trước****
**@ và lưu vào username -> giới hạn time trong 2 phút -> loại các recordnumber trùng  -> loại các tài khoản hệ thống ví dụ MACHINE$ ->Tạo thống kê theo username và time lọc các nhóm ko có mã 4648**
![Pasted image 20240925231628](https://github.com/user-attachments/assets/8b3337b7-f201-488d-9aef-b2b6f66619f7)


#### Detecting Kerberoasting Using Transactions - TGS Requests
```shell-session
index=main earliest=1690450374 latest=1690450483 EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| search username!=*$ 
| transaction username keepevicted=true maxspan=5s endswith=(EventCode=4648) startswith=(EventCode=4769) 
| where closed_txn=0 AND EventCode = 4769
| table _time, EventCode, service_name, username
```

| search username!=*$ :`COMPUTERNAME$`.  loại bỏ tất cả các tài khoản máy tính và chỉ hiển thị tài khoản người dùng thông thường (những tài khoản không kết thúc bằng `$`)

| transaction username keepevicted=true maxspan=5s endswith=(EventCode=4648) startswith=(EventCode=4769): : Nhóm các sự kiện vào các transaction dựa trên trường username. Tùy chọn keepevicted=true bao gồm cả những sự kiện không đáp ứng tiêu chí của transaction. Tùy chọn maxspan=5s đặt thời lượng tối đa của một transaction là 5 giây. Các tùy chọn endswith=(EventCode=4648) và startswith=(EventCode=4769) xác định rằng các transaction nên bắt đầu bằng một sự kiện có EventCode 4769 và kết thúc bằng một sự kiện có EventCode 4648.

| where closed_txn=0 AND EventCode = 4769:  
+ **where closed_txn=0**: Lọc các bản ghi trong đó trường `closed_txn` có giá trị bằng `0`, nghĩa là chỉ lấy những giao dịch chưa được đóng.
- **AND EventCode=4769**: Kết hợp điều kiện thứ hai, chỉ lấy những bản ghi có mã sự kiện (`EventCode`) bằng 4769.



**=>endswith=(EventCode=4648): Giao dịch sẽ kết thúc khi gặp sự kiện có mã EventCode là 4648 (đăng nhập tường minh).Startswith=(EventCode=4769): Giao dịch sẽ bắt đầu khi gặp sự kiện có mã EventCode là 4769 (liên quan đến Kerberos).**

**where closed_txn=0: Lọc ra các giao dịch chưa hoàn thành (closed_txn=0), tức là những giao dịch bắt đầu nhưng chưa kết thúc (chưa gặp EventCode 4648).**
**AND EventCode=4769: Chỉ giữ lại các sự kiện có mã sự kiện 4769.**

**tìm kiếm các sự kiện với mã EventCode 4769 (Kerberos Service Ticket Operations) liên quan đến dịch vụ IIS và theo dõi các sự kiện đăng nhập tiếp theo (EventCode 4648). Nó tìm những giao dịch (transaction) liên quan đến cùng một người dùng, bắt đầu với sự kiện 4769 và giới hạn trong 5 giây, nhưng không có sự kiện kết thúc (EventCode 4648). Điều này có thể giúp phát hiện các tình huống trong đó một người dùng yêu cầu vé dịch vụ Kerberos nhưng không thực hiện đăng nhập tường minh, có thể gợi ý về sự bất thường hoặc vấn đề bảo mật.**
