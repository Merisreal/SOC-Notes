## PASS THE HASH
https://sec.vnpt.vn/2023/01/pth/


Windows lưu trữ mật khẩu người dùng ở dạng NT hash (NTLM Hash) -> Windows thiết lập hệ thống SSO, lưu trữ thông tin xác thực, mà ko cần nhập lại mật khẩu

![Pasted image 20240927162747](https://github.com/user-attachments/assets/313a29f0-562d-42af-aa32-e8b6285355ab)

Sử dụng giao thức NTLMv2  và Hash NTLMv2 sử dụng NTLM hash trong phương thức trao đổi khóa Challenge/Response -> chỉ cần biết NTLM hash mà ko cần biết mật khẩu để lấy NTLMv2 Hash -> Pass The Hash


### Các Nơi để lấy NTLM Hash

1.  Security Account Manager (SAM)
Nơi lưu trữ các Hash NTLM của các tài khoản cục bộ trên máy tính (local Account/Microsoft Account) -> ko bao gồm các tài khoản trong domain 
```
%SystemRoot%/system32/config/SAM  
```
khóa registry : HKLM/SAM
Các thông tin giải mã SAM đều có thể tìm thấy trên máy tính -> do đó khi chiếm được quyền administrator có thể trích xuất các hash NTLM được lưu trữ trong SAM

2. LSASS Memory
Local Sercurity Authority Subsystem Service (LSASS) có thể chứa tệp NTLM hash
```
%SystemRoot%\System32\Lsass.exe
```
Mở bởi wininit.exe khi hệ thống khởi động 
Mỗi khi người dùng đăn nhập vào hệ thống, cấu trúc dữ liệu gồm tên và NTLM Hash được tạo và lưu vào vùng nhớ của tiến trình LSASS (tài khoản local, microsoft, domain) -> và chỉ bị xóa khi logoff

3. DCSyncs và NTDS
Directory Replication Service Remote (MS-DRSR) -> mô phỏng lại hành vi của domain và trích xuất các thông tin của tài khoản mục tiêu, NTLM Hash
Ngoài ra, khi truy cập được domain controller -> có thể đọc được nội dung trên cơ sở dữ liệu của Active Directory
```
%SystemRoot%\NTDS\Ntds.dit
```
-> trích xuất NTLM hash từ đây
### Attack Steps:
tool : MimiKatz
+ Sử dụng Mimikatz để lấy NTLM hash của user hiện tại -> máy chiếm quyền phải có quyền administrator
![Pasted image 20240919155603](https://github.com/user-attachments/assets/38e1acf4-0d0d-46f8-bdc2-1cf20278bb7a)

+ Có NTLM hash -> có thể xác thực là user trên hệ thống mà ko cần biết mật khẩu thực tế
![Pasted image 20240919155607](https://github.com/user-attachments/assets/b72dfe44-2f25-4bae-adb6-954d73c718fb)

+ Có thể di chuyển ngang trong mạng
![Pasted image 20240919155629](https://github.com/user-attachments/assets/da3dba20-621a-44d9-a357-24f1e84f9b08)

### Windows Access Tokens & Alternate Credentials
https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens

Access Token: định nghĩa ngữ cảnh bảo mật của một process hay thread. Chứa thông tin về quyền hạn của tài khoản người dùng liên quan
+ Khi một user đang nhập, hệ thống so sánh với thông tin trong hệ thống để xác thực, nếu đúng sẽ cấp một access token, bất kì process nào thực thi thay mặt cho user đó nó đều có một bản sao access token này
Alternate Credentials: Cho phép người dùng truy cập tài nguyên, thực thi lệnh dưới tư cách người dùng khác mà không cần chuyển đổi tài khoản

=> runas command: thực thi dòng lệnh dưới tư cách user khác -> khi thực thi một access token được tạo 
![Pasted image 20240919160149](https://github.com/user-attachments/assets/0893b28a-1fd8-4a37-b41e-62c264511983)

Trong runas có /netonly: cho biết thông tin người dùng được chỉ định truy cập từ xa, Mặc dù lệnh whoami trả về username ban đầu, nhưng cmd.exe được tạo ra vẫn có thể truy cập thư mục gốc của Domain Controller.
![Pasted image 20240919160254](https://github.com/user-attachments/assets/92bab215-9acb-41b9-97d3-9f2c240b4a9d)

Mỗi access token tham chiếu đến một LogonSession được tạo ra khi người dùng đăng nhập. Cấu trúc bảo mật LogonSession này chứa các thông tin như Username, Domain và AuthenticationID (NTHash/LMHash), và được sử dụng khi process cố gắng truy cập tài nguyên từ xa. Khi cờ netonly được sử dụng, process vẫn có cùng một access token nhưng có một LogonSession khác.

![Pasted image 20240919160326](https://github.com/user-attachments/assets/ef32f29c-07fb-458b-aad3-22ddee892fcb)

### Detection 
https://blog.netwrix.com/2021/11/30/how-to-detect-pass-the-hash-attacks/
https://sec.vnpt.vn/2023/01/pth/

1. Dấu hiệu khi trích xuất NTLM Hash từ Lsass.exe
Sysmon event ID 10
![Pasted image 20240926183908](https://github.com/user-attachments/assets/a4c0ba1e-5142-4b7d-943a-0e6e7184f528)

Mỗi khi dump bộ nhớ của Isass.exe sẽ tạo ra các file lsass.dmp trong thư mục %temp% hay %windows%\temp 
Sysmon event ID 11
![Pasted image 20240926184010](https://github.com/user-attachments/assets/6a6ca872-d10c-498d-8489-926b56201863)

3. Khi triết xuất NTLM hash từ SAM
Event ID 4656 khi thiết lập cấu hình audit cho khóa registry HKLM/SAM
![Pasted image 20240926184058](https://github.com/user-attachments/assets/df0aa27b-3fa0-4437-91b8-abf56c086e17)

Có một lỗ hổng cho phép kẻ tấn công truy cập dữ liệu từ SAM thông qua VoulumeShadowCopy. Với lỗ hổng này, kẻ tấn công có thể thực hiện trích xuất NTLM hash từ tài khoản thường sau đó dùng Pth để leo thang lên quyền Admin hoặc System. Sự kiện này có thể phát hiện bằng việc cấu hình audit cho tập tin %Systemroot%\system32\config\SAM.

![Pasted image 20240926184110](https://github.com/user-attachments/assets/4fb5b0bd-6891-44c2-b744-786629556efa)


3. Access ntds.dit
Khi attacker sử dụng ntdsutil để truy cập dữ liệu tại ntds.dit sự kiện ESENT sẽ lưu lại
 325, 326, 327,
![Pasted image 20240926184302](https://github.com/user-attachments/assets/5440bd2d-7688-4459-b79f-54a91855a1ce)

 Ngoài ra, khi bật cấu hình audit cho tập tin tại 
 ```
 %SystemRoot%\NTDS\Ntds.dit
 ```
cũng có thể được các hành vi truy cập nó, từ đó xác định các hành vi từ các tiến trình bất thường để cảnh báo
 ![Pasted image 20240926184328](https://github.com/user-attachments/assets/c3510567-28be-4a4b-8547-50a8e38aadf6)

 4. DCSync

Đối với hành vi tấn công bằng kỹ thuật DCSync, các sự kiện 4662 sẽ được ghi lại và các quyền truy cập được cấp sẽ bao gồm:

- DS-Replication-Get-Changes (1131f6aa-9c07-11d1-f79f-00c04fc2dcd2)
- DS-Replication-Get-Changes-All (1131f6ad-9c07-11d1-f79f-00c04fc2dcd2)
- DS-Replication-Get-Changes-In-Filtered-Set (89e95b76-444d-4c62-991a-0facbeda640c)
Sau khi lọc ra những sự kiện trên, có thể kiểm tra trường “Account Name”. So sánh với một hành vi bình thường trên domain thì trong khi tấn công bằng DCSync, trường này sẽ có giá trị là user thông thường thay vì tên của domain controller.




![Pasted image 20240926184359](https://github.com/user-attachments/assets/8877585c-9544-44a4-9b05-c6c913d406c5)




5. PASS THE HASH LOG

Khi sử dụng kerberos ticket để tấn công pth, kẻ tấn công cần tạo ra một phiên đăng nhập giả và chèn ntlm hash vào phiên đó. Việc này có thể phát hiện bằng event 4624 với kiểu đăng nhập Logon Type = 9 và tiến trình thực hiện Logon Process là seclogo.

+ Khi runas ko có cờ /netonly 
EventID 4624 (logon) -> logontype 2 (interactive)
![Pasted image 20240919160650](https://github.com/user-attachments/assets/9b2bab46-59fa-40dc-99ac-55980716f7af)

+ Khi runas có cờ /netonly:
EventID 4624 (logon) -> Logontype 9 (NewCredentials) 
![Pasted image 20240919160652](https://github.com/user-attachments/assets/04c3e6d6-e48a-4d10-9029-116eeebd664a)

Với MimiKAT  sẽ truy cập bộ nhớ quy trình LSASS để thay đổi tài liệu thông tin xác thực LogonSession
-> Sysmon Event Code 10

### Splunk
![Pasted image 20240919162022](https://github.com/user-attachments/assets/66c9dd1b-afcc-4f51-b7f2-d97a0800211b)



```shell-session
index=main earliest=1690450708 latest=1690451116 source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo
| table _time, ComputerName, EventCode, user, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
```
![Pasted image 20240919162717](https://github.com/user-attachments/assets/e5909a3d-2018-4f79-beb1-96b4da962b5b)


```shell-session
index=main earliest=1690450689 latest=1690451116 (source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe" SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe") OR (source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo)
| sort _time, RecordNumber
| transaction host maxspan=1m endswith=(EventCode=4624) startswith=(EventCode=10)
| stats count by _time, Computer, SourceImage, SourceProcessId, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
| fields - count
```

**=> Tìm các sự kiện liên quan đến việc sysmon 10 triết xuất hash từ lsass  nhưng lọc ra các sự kiện ko phải của Windows, tiếp theo lọc thêm các sự kiên Khi sử dụng kerberos ticket để tấn công pth, kẻ tấn công cần tạo ra một phiên đăng nhập giả và chèn ntlm hash vào phiên đó. Việc này có thể phát hiện bằng event 4624 với kiểu đăng nhập Logon Type = 9 và tiến trình thực hiện Logon Process là seclogo -> transacsion trong 1p,  bắt đầu bằng triết xuất lsass.exe và kết thúc khi đăng nhập 4624**

**| fields - count: trừ đi trường count**

![Pasted image 20240919163731](https://github.com/user-attachments/assets/4d1559bd-8091-4b18-bed5-310f6434ca90)

