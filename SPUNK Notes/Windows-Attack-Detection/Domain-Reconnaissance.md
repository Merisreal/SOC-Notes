## Domain Reconnaissance

Một vài Tool sẽ sử dụng các câu lệnh sau để reconnaissance:
+ `whoami /all`
- `wmic computersystem get domain`
- `net user /domain`
- `net group "Domain Admins" /domain`
- `arp -a`
- `nltest /domain_trusts`
### User/Domain Reconnaissance -> BloodHound/SharpHound

https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/bloodhound

BloodHound trực quan hóa môi trường trong AD, MÔ PHỎNG CÁC ĐƯỜNG DẪN ĐẾN DOMAIN ADMIN

LDAP: DUMP THÊM CÁC THÔNG TIN TRONG DOMAIN
![Pasted image 20240916164942](https://github.com/user-attachments/assets/4f2b6815-2333-41fd-a7e5-8b986dc07173)


Sharphound là trình thu thập dữ liệu C# cho BloodHound
![Pasted image 20240916164958](https://github.com/user-attachments/assets/98f0e570-f8e7-4a66-a114-73f6c2ae1f7d)

####  Detection 

 BloodHound thực hiện nhiều truy vấn LDAP(_ldapsearch.exe_, _dsget.exe_, and _dsquery.exe_))hướng tới Bộ điều khiển miền, nhằm mục đích tích lũy thông tin về miền.

![[Pasted image 20240925160950.png]]
![Pasted image 20240925160950](https://github.com/user-attachments/assets/f12e6de5-6598-4bf8-a645-1765039c7be2)


Windows Event Logs không thể record được LDAP -> Sử dụng Windows ETW provider Microsoft-Windows-LDAP-Client -> https://github.com/mandiant/SilkETW





 [list of LDAP filters frequently used by reconnaissance tools](https://techcommunity.microsoft.com/t5/microsoft-defender-for-endpoint/hunting-for-reconnaissance-activities-using-ldap-search-filters/ba-p/824726).
![Pasted image 20240925161107](https://github.com/user-attachments/assets/2aa05b45-7ea2-4a1f-ba6f-b31dd2d154af)

### Splunk

####  User/Domain Recon By Targeting Native Windows Executables
```shell-session
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=1 earliest=1690447949 latest=1690450687
| search process_name IN (arp.exe,chcp.com,ipconfig.exe,net.exe,net1.exe,nltest.exe,ping.exe,systeminfo.exe,whoami.exe) OR (process_name IN (cmd.exe,powershell.exe) AND process IN (*arp*,*chcp*,*ipconfig*,*net*,*net1*,*nltest*,*ping*,*systeminfo*,*whoami*))
| stats values(process) as process, min(_time) as _time by parent_process, parent_process_id, dest, user
| where mvcount(process) > 3
```

**=> Tìm các sự kiện liên quan tới việc tạo tiến trình -> các tên Process Name có trong số các keyword( arp.exe,....) hay trong CMD và Powershell có chứa các câu lệnh đặc thù để recon, tạo bảng thống kê danh sách các tiến trình gán cho process, lấy thời gian sớm nhất được tạo nhóm dữ liệu theo các trường process cha, id process cha, dest, user, chỉ dữ lại nhóm mà tiến trình >3**

+ Source: XmlWinEventLog:Microsoft-Windows-Sysmon/Operationan: 
+ earliest=1690447949 latest=1690450687: thời gian

+ search process_name IN (arp.exe,chcp.com,ipconfig.exe,net.exe,net1.exe,nltest.exe,ping.exe,systeminfo.exe,whoami.exe) OR (process_name IN (cmd.exe,powershell.exe) AND process IN (*arp*,*chcp*,*ipconfig*,*net*,*net1*,*nltest*,*ping*,*systeminfo*,*whoami*)): tìm các tên process đặc thù. và tìm trong cmd và powershell có chứa các câu lệnh đặc thù để recon
+ stats values(process) as process, min(_time) as _time by parent_process, parent_process_id, dest, user: min(_time) để lấy thời gian bắt đầu sự kiện
+ where mvcount(process) > 3: nếu có hơn 3 process thì mới hiện
![Pasted image 20240925161331](https://github.com/user-attachments/assets/55d50c69-3357-49ec-af94-d3b8e2900507)


####  Recon By Targeting BloodHound

```shell-session
index=main earliest=1690195896 latest=1690285475 source="WinEventLog:SilkService-Log"
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter
| sort 0 _time
| search SearchFilter="*(samAccountType=805306368)*"
| stats min(_time) as _time, max(_time) as maxTime, count, values(SearchFilter) as SearchFilter by ComputerName, ProcessName, ProcessId
| where count > 10
| convert ctime(maxTime)
```



+ index=main earliest=1690195896 latest=1690285475 source="WinEventLog:SilkService-Log": 
+ spath input=Message : dùng spatch lấy thông tin từ cấu trúc XML và Json vì nó.. khó nhìn
![Pasted image 20240916174334](https://github.com/user-attachments/assets/c8a747de-7825-4374-9ff2-250850bbd349)

+ rename XmlEventData.* as * 
Giả sử bạn có dữ liệu trong trường `XmlEventData` như sau:
`<XmlEventData>     <Field1>Value1</Field1>     <Field2>Value2</Field2> </XmlEventData>`
Khi sử dụng `| rename XmlEventData.* as *`, trường `Field1` và `Field2` sẽ được đổi tên thành các trường chính như sau:
`Field1: Value1 Field2: Value2`
Thay vì phải truy cập các trường thông qua `XmlEventData.Field1` và `XmlEventData.Field2`,  có thể truy cập trực tiếp bằng `Field1` và `Field2`

+ search SearchFilter="*(samAccountType=805306368)*":Lọc các sự kiện có trường `SearchFilter` chứa giá trị `samAccountType=805306368` ->https://techcommunity.microsoft.com/t5/microsoft-defender-for-endpoint/hunting-for-reconnaissance-activities-using-ldap-search-filters/ba-p/824726
+ | convert ctime(maxTime): chuyển unix thành giờ human read



**=> trích xuất các dữ liệu từ JSON hoặc XML trong trường message ,Đổi tên các trường có tiền tố "XmlEventData." thành các trường không có tiền tố (để dễ thao tác hơn), hiện bảng theo thời gian,**** 
**ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter ->sắp xếp theo thời gian, nội dung Search có trong danh sách liên quan tới sử dụng LDAP, tạo thống kê, thời gian sớm nhất mỗi nhóm, muộn nhất, đếm các kết quả của searchfilter lưu ở searchfiter,computer,processnname,processID -> chỉ hiện khi count > 10 -> chuyển time sang dạng human read**
![Pasted image 20240925161338](https://github.com/user-attachments/assets/8606e16b-1673-423a-a9e0-e11624a5b36d)


