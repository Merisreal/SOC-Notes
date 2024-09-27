![Pasted image 20240917130043](https://github.com/user-attachments/assets/16f89b5e-6921-4595-b7a3-de9f50daf799)DNS Resolution Example:

Khi một PC muốn truy cập tới www.cisco.com vào thanh địa chỉ 
+ Tại thời điểm này máy tính không biết địa chỉ IP của www.cisco.com, nên nó sẽ gửi DNS query tới ISP's DNS Server 
+ ISP's DNS Server cũng không biết địa chỉ IP của cissco.com nên nó sẽ hỏi ROOT DNS Servers
+ ROOT DNS Server kiểm tra trong database  Primary DNS của cisco.com là 192.133.221.214 -> gửi lại tới ISP severs
+ ISP DNS severs biết địa chỉ của Cisco's DNS Sever, nên nó gửi **recursive query** (truy vấn đệ quy )tới cisco.com's DNS servers và yêu cầu phân giải full tên miền của www.cisco.com
+ Cisco.com's DNS servers  và tìm thấy một bản ghi cho www.cisco.com, bản ghi chứa địa chỉ 192.133.221.214 -> địa chỉ DNS Server và webserver (www) gioosgn nhau điều này có nghĩa là chúng có khả năng ở trên cùng một server vật lý. Cơ chế load-balancing cũng có thể gây ra hiệu ứng tương tự, khiến nhiều dịch vụ và máy chủ vật lý có cùng một địa chỉ IP
+ **DNS server của ISP** của bạn bây giờ đã biết địa chỉ IP cho [www.cisco.com](http://www.cisco.com) và gửi kết quả đến máy tính của bạn.
- Máy tính của bạn bây giờ đã biết địa chỉ IP của trang web Cisco và có thể liên hệ trực tiếp với nó. Bước tiếp theo tự nhiên là gửi một **http request** trực tiếp đến **Cisco's webserver** và tải trang web xuống.

![Pasted image 20240917123639](https://github.com/user-attachments/assets/1980afe7-0feb-454d-90bf-2a453c7dfd7c)

https://www.firewall.cx/networking/network-protocols/dns-protocol/protocols-dns-resolution.html


Tuy nhiên Khi **DNS resolution** fail hay DNS Server không khả dụng
LLMNR và NetBios sẽ được bật mặt định trên hệ thoosgn windows

Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS)
tool: [Responder](https://github.com/lgandx/Responder)

![Pasted image 20240917124949](https://github.com/user-attachments/assets/eab964a0-0daa-46d8-827a-e01ad6870d5e)




Link-Local Multicast Name Resolution (LLMNR) và NetBIOS Name Service (NBT-NS) là các thành phần của Microsoft Windows, hoạt động như những phương pháp thay thế để xác định máy chủ. LLMNR dựa trên định dạng Domain Name System (DNS) và cho phép các máy chủ trên cùng một liên kết cục bộ thực hiện việc phân giải tên cho các máy chủ khác. NBT-NS xác định các hệ thống trên mạng cục bộ thông qua tên NetBIOS của chúng.


+ Kiểm tra xem tên có phân giải thành máy chủ cục bộ (localhost **127.0.0.1** ) hay không
+ Kiểm tra xem tên có nằm trong **cache** hoặc được chỉ định thủ công trong file **hosts** của hệ thống (C:\Windows\System32\drivers\etc\hosts).
- Gửi một yêu cầu tra cứu đến **DNS server** đã được cấu hình 
- Nếu DNS FAILS Phát một truy vấn tên **LLMNR** đến tất cả các máy trên mạng cục bộ.
- Phát một yêu cầu truy vấn tên **NBT-NS** đến tất cả các máy trên mạng cục bộ

Các truy vấn **LLMNR** và **NBT-NS** sẽ được gửi đến tất cả các máy khác trên mạng cục bộ, yêu cầu chúng phản hồi nếu biết địa chỉ IP của hostname đang được truy vấn.
![Pasted image 20240925222918](https://github.com/user-attachments/assets/4107e96e-d1b5-4ee0-b72d-675ac42fe142)


https://medium.com/@aniswersighni/active-directory-attacks-llmnr-nbt-ns-poisoning-and-relay-5767f81639a6

### Detection

-> Có thể tận dụng nếu  tiêm các yêu cầu LLMNR và NBT-NS cho những máy chủ không tồn tại ở các subnet khác nhau -> nếu có phản hồi -> có một attacker hiện diện và giả mạo các phản hồi phân giải tên -> có được địa chỉ attacker


Resolve: https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/
Script:
![Pasted image 20240917125947](https://github.com/user-attachments/assets/b8b3c0f7-12da-4318-924a-f83751ad2489)

Add New-EventLog và Event Log
```powershell-session
PS C:\Users\Administrator> New-EventLog -LogName Application -Source LLMNRDetection
```
-> 
```powershell-session
PS C:\Users\Administrator> Write-EventLog -LogName Application -Source LLMNRDetection -EventId 19001 -Message $msg -EntryType Warning
```

### SPLUNK:
```shell-session
index=main earliest=1690290078 latest=1690291207 SourceName=LLMNRDetection
| table _time, ComputerName, SourceName, Message
```
![Pasted image 20240917130043](https://github.com/user-attachments/assets/5e82c5c1-4e92-4486-9be9-53b28c8aac45)

Hoặc sử dụng Sysmon Event ID 22:

```shell-session
index=main earliest=1690290078 latest=1690291207 EventCode=22 
| table _time, Computer, user, Image, QueryName, QueryResults
```
![Pasted image 20240917130110](https://github.com/user-attachments/assets/0a85a063-98b6-459b-a7cc-bb6404c62d45)

Hoặc  [Event 4648](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4648) có thể được sử dụng để phát hiện các thông tin đăng nhập rõ ràng để chia sẻ tệp giả mạo mà kẻ tấn công có thể sử dụng để thu thập thông tin xác thực của người dùng hợp pháp.
```shell-session
index=main earliest=1690290814 latest=1690291207 EventCode IN (4648) 
| table _time, EventCode, source, name, user, Target_Server_Name, Message
| sort 0 _time
```
![Pasted image 20240917130214](https://github.com/user-attachments/assets/ef14dc40-d168-4bff-bf0a-c2dfed0520ce)

