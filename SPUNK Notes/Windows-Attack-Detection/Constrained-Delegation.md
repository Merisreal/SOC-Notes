## Constrained Delegation

Constrained Delegation là một tính năng trong Active Directory cho phép các dịch vụ ủy quyền thông tin xác thực của user chỉ đến các tài nguyên được chỉ định, giảm thiểu rủi ro liên quan đến Unconstrained Delegation. Bất kỳ user hoặc computer accounts nào có service principal names (SPNs) được thiết lập trong thuộc tính msDS-AllowedToDelegateTo của chúng có thể giả mạo bất kỳ user nào trong miền đối với các SPNs cụ thể đó.

Cách hoạt động: user A cần truy cập tài nguyên lưu trữ trên Server B, nhưng xác thực của user A lại quản lý bới Serivce C -> Servcer C có trách nhiệm xác thực cho user A và lấy tài nguyên được yêu cầu thay mặt cho user A-> mất thời gian
Tuy nhiên với Constrained Delegation, Service C có thể ủy quyền thông tin xác thực của user A cho Server B, Server B với thông tin đó, có thể giả mạo người A truy cập vào cà tài nguyên -> cách này giúp loại bỏ hoạt động của Service C -> cải thiện hiệu suất
![[Pasted image 20240927154015.png]]


### Attack Steps
+ Xác định các nơi mà Constrained Delegation được bật -> xác định các tài nguyên cho phép được ủy quyền
![[Pasted image 20240924222830.png]]

+ truy cập TGT của user hay compuuter -> lấy TGT từ memory (Rubeus dump)  hay yêu cầu bằng hàm băm của user hay computer
![[Pasted image 20240924223015.png]]
+ Kẻ tấn công sử dụng kỹ thuật S4U để mạo danh một tài khoản có đặc quyền cao cho dịch vụ được nhắm mục tiêu (yêu cầu vé TGS).
![[Pasted image 20240924223046.png]]
+ Kẻ tấn công tiêm vé được yêu cầu và truy cập các dịch vụ được nhắm mục tiêu với tư cách là người dùng mạo danh
![[Pasted image 20240924223100.png]]
###  Kerberos Protocol Extensions - Service For User

Service for User to Self (S4U2self) và Service for User to Proxy (S4U2proxy) cho phép một dịch vụ yêu cầu một ticket từ Key Distribution Center (KDC) thay mặt cho một user. S4U2self cho phép một dịch vụ nhận một TGS cho chính nó thay mặt cho một user, trong khi S4U2proxy cho phép dịch vụ nhận một TGS thay mặt cho một user cho một dịch vụ thứ hai.

S4U2self được thiết kế để cho phép một user yêu cầu một TGS ticket khi một phương pháp xác thực khác được sử dụng thay vì Kerberos. Quan trọng là, TGS ticket này có thể được yêu cầu thay mặt cho bất kỳ user nào, chẳng hạn như một Administrator.

S4U2proxy được thiết kế để lấy một ticket có thể chuyển nhượng và sử dụng nó để yêu cầu một TGS ticket cho bất kỳ SPN nào được chỉ định trong các tùy chọn msds-allowedtodelegateto cho user được chỉ định trong phần S4U2self.

Với sự kết hợp của S4U2self và S4U2proxy, một kẻ tấn công có thể giả mạo bất kỳ user nào tới các service principal names (SPNs) được thiết lập trong các thuộc tính msDS-AllowedToDelegateTo.
![[Pasted image 20240924223115.png]]


### Detection
Giống với Unconstrained Delegation, có thể phát hiện bằng command Powershell hay LDAP request nhằm mục đích phát hiện Constrained Delegation users and computers.

Để yêu cầu vé TGT users and computers. cũng như vé TGS bằng kỹ thuật S4U, Rubeus tạo kết nối với Domain Controller. Hoạt động này có thể được phát hiện dưới dạng kết nối mạng quy trình bất thường tới cổng TCP/UDP 88 (Kerberos).

```shell-session
msDS-AllowedToDelegateTo
```



### Splunk
```shell-session
index=main earliest=1690544553 latest=1690562556 source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*msDS-AllowedToDelegateTo*" 
| table _time, ComputerName, EventCode, Message
```
**Lọc Event 4104 liên quan tới thực thi câu lệnh powershell tìm message liên quan tới Constrained Delegation "*msDS-AllowedToDelegateTo*"**
![Pasted image 20240924224848](https://github.com/user-attachments/assets/ab92a158-2379-47d4-ad94-c9c2ad9de90a)

##### Sysmon Log
```shell-session
index=main earliest=1690562367 latest=1690562556 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" 
| eventstats values(process) as process by process_id
| where EventCode=3 AND dest_port=88
| table _time, Computer, dest_ip, dest_port, Image, process

```

**EventCode =3 -> kết nối mạng, và dest-port 88 liên quan tới việc xác thực Kerberos**


![Pasted image 20240924224846](https://github.com/user-attachments/assets/7c97b1e8-b7c3-475e-82ec-83742381ae5d)

