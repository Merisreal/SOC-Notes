## Overpass The Hash
https://medium.com/r3d-buck3t/play-with-hashes-over-pass-the-hash-attack-2030b900562d

[Mitre category](https://attack.mitre.org/techniques/T1210/) “Exploitation of remote services.”

Sau Khi đã xâm nhập domain user trên mạng, bước tiếp theo phải thu nhập tất cả username, hashes, thông tin nhậy cảm, thiết kế mạng 
-> Có 2 cách để di chuyển ngang qua mạng 
+ Pass The Hash
+ Over Pass The Hash
Pass The Hash sử dụng dump NTLM hash 
Over Pass The Hash Kết hợp của Pass The Hash và Pass The Ticket, sử dụng các Hash đã thu nhập được để yêu cầu Kerberos TGT ticket từ KDC -> các forged tickets được truyền và tái sử dụng nhiều lần để tránh tương tác với KDC

tool: Mimikatz và Rubeus.

### ATTACK Steps:

https://blog.netwrix.com/2022/10/04/overpass-the-hash-attacks/

Attacker sử dụng các công cụ như Mimikatz để trích xuất NTLM hash của một người dùng đang đăng nhập vào hệ thống bị xâm nhập. Attacker phải có ít nhất quyền **local administrator** trên hệ thống để có thể trích xuất được hash của người dùng.
![Pasted image 20240923230314](https://github.com/user-attachments/assets/29830121-7cc8-419f-ab92-d51645a0469d)

Tiếp theo, attacker sử dụng một công cụ như Rubeus để tạo một yêu cầu AS-REQ raw cho một người dùng cụ thể nhằm yêu cầu TGT ticket. Bước này không yêu cầu quyền cao trên host để yêu cầu TGT, điều này làm cho phương pháp này kín đáo hơn so với tấn công Pass-the-Hash của Mimikatz.  
![Pasted image 20240923230328](https://github.com/user-attachments/assets/fd432392-2550-4ab6-804c-3635c347def0)

Tương tự với kỹ thuật Pass-the-Ticket, attacker sẽ gửi ticket được yêu cầu cho phiên đăng nhập hiện tại.
![Pasted image 20240923230334](https://github.com/user-attachments/assets/550915ff-95b8-4a55-954a-93f60c4fc847)

### Detection 
Tấn công Overpass-the-Hash của Mimikatz để lại các dấu vết giống với cuộc tấn công Pass-the-Hash và có thể được phát hiện bằng các chiến lược tương tự. 
-> Phân tích nhật ký mạng và phát hiện các điểm bất thường do người dùng bắt đầu, bắt đầu bằng việc giám sát ID sự kiện như 4768 đối với yêu cầu TGT và 4769 đối với phiếu dịch vụ có **Logon type 9**

Tuy nhiên, Rubeus đưa ra một kịch bản hơi khác. Trừ khi TGT được yêu cầu được sử dụng trên một host khác, các cơ chế phát hiện Pass-the-Ticket có thể không hiệu quả, vì Rubeus gửi yêu cầu AS-REQ trực tiếp đến Domain Controller (DC), tạo ra Event ID 4768 (Kerberos TGT Request). Tuy nhiên, việc giao tiếp với DC (cổng TCP/UDP 88) từ một tiến trình bất thường có thể là một dấu hiệu của một cuộc tấn công Overpass-the-Hash tiềm năng.

### SPLUNK


```shell-session
index=main earliest=1690443407 latest=1690443544 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode=3 dest_port=88 Image!=*lsass.exe) OR EventCode=1
| eventstats values(process) as process by process_id
| where EventCode=3
| stats count by _time, Computer, dest_ip, dest_port, Image, process
| fields - count
```

**=> Lọc Eventcode 3 -> kết nối mạng, port 88, khác lsass HOẶC eventcode =1** 

![Pasted image 20240923230457](https://github.com/user-attachments/assets/6a0e9a8f-03e0-4992-b2cb-97ab41a09e75)
