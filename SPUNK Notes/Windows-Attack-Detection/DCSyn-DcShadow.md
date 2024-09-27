https://blog.netwrix.com/2021/11/30/what-is-dcsync-an-introduction/


## DCSync

DCSync là một kỹ thuật được kẻ tấn công khai thác để trích xuất password hashes từ Active Directory Domain Controllers (DCs). Phương pháp này tận dụng quyền Replication Directory Changes thường được cấp cho các domain controller, cho phép chúng đọc tất cả các thuộc tính của đối tượng, bao gồm cả password hashes. Các thành viên trong nhóm Administrators, Domain Admins, và Enterprise Admin, hoặc các computer accounts trên domain controller, có khả năng thực hiện DCSync để trích xuất dữ liệu password từ Active Directory. 
Dữ liệu này có thể bao gồm cả các hash hiện tại và lịch sử của các tài khoản có giá trị, chẳng hạn như KRBTGT và Administrators.

![[Pasted image 20240924225738.png]]

### Attack Steps

+ Kẻ tấn công đảm bảo quyền  quản trị vào một hệ thống đã tham gia miền hoặc nâng cao quyền để có được các quyền cần thiết để yêu cầu dữ liệu replication.
+ Sử dụng tool như Mimikatz, kẻ tấn công yêu cầu dữ liệu replication miền bằng cách sử dụng giao diện DRSGetNCChanges giống domain controller hợp pháp
+ ![[Pasted image 20240924225943.png]]
+ Tạo Golder Tickets, Silver Tickets, hay Pass The Hash/ Overpass-The-Hash Attacks
### Detection

DS-Replication-Get-Changes có thể bị phát hiện với:
+ Event ID 4662, Tuy nhiên, cần phải cấu hình **Audit Policy** bổ sung vì nó không được bật theo mặc định (Computer Configuration/Windows Settings/Security Settings/Advanced Audit Policy Configuration/DS Access).
![[Pasted image 20240924230141.png]]

Tìm kiếm các sự kiện chứa thuộc tính `{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}`, tương ứng với **DS-Replication-Get-Changes**, vì Event 4662 chỉ bao gồm các GUID.

### Splunk
```shell-session
index=main earliest=1690544278 latest=1690544280 EventCode=4662 Message="*Replicating Directory Changes*"
| rex field=Message "(?P<property>Replicating Directory Changes.*)"
| table _time, user, object_file_name, Object_Server, property
```

**Tìm Event 4662 (An operation was performed on an object) với Replicating Directory Changes, lưu tất cả log và property**
## DCShadow


https://www.dcshadow.com/

**DCShadow** là một kỹ thuật tiên tiến được kẻ tấn công sử dụng để thực hiện các thay đổi trái phép đối với các đối tượng trong Active Directory, bao gồm tạo hoặc chỉnh sửa các đối tượng mà không tạo ra các log bảo mật thông thường. Cuộc tấn công này khai thác quyền **Directory Replicator (Replicating Directory Changes)**, thường được cấp cho các domain controller để thực hiện các tác vụ replication. DCShadow là một kỹ thuật ngầm cho phép kẻ tấn công thao túng dữ liệu Active Directory và thiết lập sự tồn tại bền vững trong mạng. Việc đăng ký một domain controller giả mạo đòi hỏi phải tạo ra các đối tượng **server** và **nTDSDSA** mới trong partition Configuration của AD schema, yêu cầu quyền Administrator (Domain hoặc local trên DC) hoặc hash của **KRBTGT**
DCShadow là một lệnh trong module lsadump của công cụ hack mã nguồn mở Mimikatz

Các cuộc tấn công DCShadow rất khó ngăn chặn. Giống như DCSync, nó không khai thác một lỗ hổng có thể được vá; nó lợi dụng các chức năng hợp lệ và cần thiết của quá trình sao chép, mà không thể tắt hoặc vô hiệu hóa. Các cuộc tấn công DCShadow cũng khó phát hiện, vì các thay đổi mà kẻ thù yêu cầu được ghi lại, xử lý và cam kết như một sao chép miền hợp lệ.


### Attack Steps
+ Kẻ tấn công đảm bảo quyền truy cập quản trị vào một hệ thống đã tham gia domain hoặc nâng cao quyền để có được các quyền cần thiết để yêu cầu dữ liệu replication.
+ Kẻ tấn công đăng ký một domain controller giả mạo trong domain, sử dụng quyền **Directory Replicator**, và thực hiện các thay đổi đối với các đối tượng AD, chẳng hạn như chỉnh sửa các nhóm user thành nhóm **Domain Administrator**.
![[Pasted image 20240924230345.png]]

+ Domain controller giả mạo khởi tạo quá trình replication với các domain controller hợp pháp, phát tán các thay đổi trong toàn bộ domain.
![[Pasted image 20240924230404.png]]

### Detection

Để mô phỏng Domain Controller, DCShadow phải thay đổi các thay đổi sau trong Activev Directory:
+ Thêm nTDSDSA object
+ Nối một danh mục chung ServicePrincipalName vào đối tượng máy tính
=> Event ID 4742 (Computer account was changed) => ServicePrincipalName

### Splunk

```shell-session
index=main earliest=1690623888 latest=1690623890 EventCode=4742 
| rex field=Message "(?P<gcspn>XX\/[a-zA-Z0-9\.\-\/]+)" 
| table _time, ComputerName, Security_ID, Account_Name, user, gcspn 
| search gcspn=*
```
Tìm Message liên quan tới Event 4742 (##  Windows Security Log Event ID 4742) chỉ lọc khi có gcspn 
