
# Windows Fundamentals

## Operating System Structure

Thư mục root của Windows
```
<drive_letter:\(Thông thường là ổ C)>
```
-> Thư mục gốc hay thư mục khởi động (boot partiton) - nơi mà hệ điều hành được cài đặt 

Cấu trúc phân vùng khơi động:

| **Thư mục**                    | **Chức năng**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Perflogs**                   | Chứa các log hiệu suất của Windows nhưng mặc định thì trống.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **Program Files**              | Trên hệ thống 32-bit, tất cả các chương trình 16-bit và 32-bit được cài đặt ở đây. Trên hệ thống 64-bit, chỉ có các chương trình 64-bit được cài đặt ở đây.                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Program Files (x86)**        | Các chương trình 32-bit và 16-bit được cài đặt ở đây trên các phiên bản Windows 64-bit.                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **ProgramData**                | Đây là thư mục ẩn chứa dữ liệu cần thiết để một số chương trình đã cài đặt có thể chạy. Dữ liệu này có thể được truy cập bởi chương trình bất kể người dùng nào đang sử dụng hệ thống.                                                                                                                                                                                                                                                                                                                                                                        |
| **Users**                      | Thư mục này chứa các hồ sơ người dùng cho mỗi người dùng đăng nhập vào hệ thống và chứa hai thư mục Public và Default.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **Default**                    | Đây là mẫu hồ sơ người dùng mặc định cho tất cả các người dùng mới được tạo. Khi thêm người dùng mới vào hệ thống, hồ sơ của họ sẽ dựa trên hồ sơ Default.                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Public**                     | Thư mục này dành cho việc chia sẻ tệp giữa các người dùng và mặc định được truy cập bởi tất cả người dùng. Thư mục này cũng được chia sẻ qua mạng mặc định nhưng cần tài khoản mạng hợp lệ để truy cập.                                                                                                                                                                                                                                                                                                                                                       |
| **AppData**                    | Dữ liệu ứng dụng và cài đặt riêng cho mỗi người dùng được lưu trữ trong một thư mục con ẩn của người dùng (ví dụ: cliff.moore\AppData). Mỗi thư mục này chứa ba thư mục con: thư mục **Roaming** chứa dữ liệu không phụ thuộc vào máy và nên theo hồ sơ của người dùng, chẳng hạn như từ điển tùy chỉnh. Thư mục **Local** là riêng cho máy tính và không bao giờ được đồng bộ hóa qua mạng. **LocalLow** tương tự với Local nhưng có mức độ toàn vẹn dữ liệu thấp hơn, ví dụ, có thể được sử dụng bởi trình duyệt web khi chạy ở chế độ bảo vệ hoặc an toàn. |
| **Windows**                    | Phần lớn các tệp cần thiết cho hệ điều hành Windows được lưu trữ tại đây.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **System, System32, SysWOW64** | Chứa tất cả các tệp DLL cần thiết cho các tính năng cốt lõi của Windows và Windows API. Hệ điều hành sẽ tìm kiếm trong các thư mục này mỗi khi một chương trình yêu cầu tải một DLL mà không chỉ định đường dẫn tuyệt đối.                                                                                                                                                                                                                                                                                                                                    |
| **WinSxS**                     | Chứa một bản sao của tất cả các thành phần, cập nhật và gói dịch vụ của Windows.                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
## Exploring Directories Using Command Line

dir ->https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir

```cmd-session
C:\htb> dir c:\ /a
 Volume in drive C has no label.
 Volume Serial Number is F416-77BE

 Directory of c:\

08/16/2020  10:33 AM    <DIR>          $Recycle.Bin
06/25/2020  06:25 PM    <DIR>          $WinREAgent
07/02/2020  12:55 PM             1,024 AMTAG.BIN
06/25/2020  03:38 PM    <JUNCTION>     Documents and Settings [C:\Users]
08/13/2020  06:03 PM             8,192 DumpStack.log
08/17/2020  12:11 PM             8,192 DumpStack.log.tmp
08/27/2020  10:42 AM    37,752,373,248 hiberfil.sys
08/17/2020  12:11 PM    13,421,772,800 pagefile.sys
12/07/2019  05:14 AM    <DIR>          PerfLogs
08/24/2020  10:38 AM    <DIR>          Program Files
07/09/2020  06:08 PM    <DIR>          Program Files (x86)
08/24/2020  10:41 AM    <DIR>          ProgramData
06/25/2020  03:38 PM    <DIR>          Recovery
06/25/2020  03:57 PM             2,918 RHDSetup.log
08/17/2020  12:11 PM        16,777,216 swapfile.sys
08/26/2020  02:51 PM    <DIR>          System Volume Information
08/16/2020  10:33 AM    <DIR>          Users
08/17/2020  11:38 PM    <DIR>          Windows
               7 File(s) 51,190,943,590 bytes
              13 Dir(s)  261,310,697,472 bytes free
```

tree ->https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tree
```cmd-session
C:\htb> tree "c:\Program Files (x86)\VMware"
Folder PATH listing
Volume serial number is F416-77BE
C:\PROGRAM FILES (X86)\VMWARE
├───VMware VIX
│   ├───doc
│   │   ├───errors
│   │   ├───features
│   │   ├───lang
│   │   │   └───c
│   │   │       └───functions
│   │   └───types
│   ├───samples
│   └───Workstation-15.0.0
│       ├───32bit
│       └───64bit
└───VMware Workstation
    ├───env
    ├───hostd
    │   ├───coreLocale
    │   │   └───en
    │   ├───docroot
    │   │   ├───client
    │   │   └───sdk
    │   ├───extensions
    │   │   └───hostdiag
    │   │       └───locale
    │   │           └───en
    │   └───vimLocale
    │       └───en
    ├───ico
    ├───messages
    │   ├───ja
    │   └───zh_CN
    ├───OVFTool
    │   ├───env
    │   │   └───en
    │   └───schemas
    │       ├───DMTF
    │       └───vmware
    ├───Resources
    ├───tools-upgraders
    └───x64
```


### **Standard Directory**
#### Examples:

1. **In Linux/Unix:**
    
    - `/bin`: Contains essential binaries (commands) for users.
    - `/etc`: Configuration files for the system.
    - `/var/log`: Log files generated by the system and applications.
    - `/home`: User home directories.
2. **In Windows:**
    
    - `C:\Windows\System32`: Contains core system files.
    - `C:\Program Files`: Default location for installed applications.
    - `C:\Users`: User profiles and data.
    - `C:\Windows\Temp`: Temporary files.
### Non Standard Directory

#### Examples:
- **Custom Installation Directories:** If a user installs software in `D:\MySoftware` instead of `C:\Program Files` on Windows.
- **User-defined Directories:** Directories that users create to store data, such as `D:\WorkFiles\Projects` on a system.
- **Non-conventional Placement:** Placing configuration files in a directory like `/opt/myapp/config` instead of the usual `/etc` in Linux.

### File System
#### Integrity Control Access Control List (icacls)

Xem Quyền, thêm quyền, xóa quyền

```cmd-session
C:\htb> icacls c:\windows
c:\windows NT SERVICE\TrustedInstaller:(F)
           NT SERVICE\TrustedInstaller:(CI)(IO)(F)
           NT AUTHORITY\SYSTEM:(M)
           NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
           BUILTIN\Administrators:(M)
           BUILTIN\Administrators:(OI)(CI)(IO)(F)
           BUILTIN\Users:(RX)
           BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
           CREATOR OWNER:(OI)(CI)(IO)(F)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
```

- **(CI)**: **container inherit** - thừa kế cho các thư mục con (container)
- **(OI)**: **object inherit** - thừa kế cho các đối tượng con (object)
- **(IO)**: **inherit only** - chỉ thừa kế
- **(NP)**: **do not propagate inherit** - không truyền thừa kế
- **(I)**: **permission inherited from parent container** - quyền thừa kế từ thư mục cha

- `F` : full access
- `D` :  delete access
- `N` :  no access
- `M` :  modify access
- `RX` :  read and execute access
- `R` :  read-only access
- `W` :  write-only access
- 
Vs user

```cmd-session
C:\htb> icacls c:\Users
c:\Users NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Administrators:(OI)(CI)(F)
         BUILTIN\Users:(RX)
         BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
         Everyone:(RX)
         Everyone:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
```




## Windows Services

Các **services** là một thành phần quan trọng của hệ điều hành Windows. Chúng cho phép tạo và quản lý các tiến trình chạy dài hạn. Các Windows services có thể được khởi động tự động khi hệ thống khởi động mà không cần sự can thiệp của người dùng. Những services này có thể tiếp tục chạy nền ngay cả khi người dùng đã đăng xuất khỏi hệ thống.

Các ứng dụng cũng có thể được tạo để cài đặt dưới dạng một service, chẳng hạn như một ứng dụng giám sát mạng được cài đặt trên máy chủ. Các Windows services chịu trách nhiệm cho nhiều chức năng trong hệ điều hành Windows, chẳng hạn như các chức năng mạng, chẩn đoán hệ thống, quản lý thông tin xác thực người dùng, kiểm soát cập nhật Windows, và nhiều hơn nữa.

Windows services được quản lý thông qua hệ thống **Service Control Manager (SCM)**, có thể truy cập qua tiện ích **services.msc** trong **MMC add-in**.
![[Pasted image 20240911161942.png]]

Tiện ích này cung cấp một giao diện đồ họa (GUI) để tương tác và quản lý các services và hiển thị thông tin về mỗi service đã cài đặt. Thông tin này bao gồm Tên service, Mô tả, Trạng thái, Kiểu khởi động, và tài khoản mà service đang chạy dưới quyền.

Ngoài ra, còn có thể truy vấn và quản lý các services qua dòng lệnh sử dụng `sc.exe` hoặc sử dụng các PowerShell cmdlets như **Get-Service**.

```powershell-session
S C:\htb> Get-Service | ? {$_.Status -eq "Running"} | select -First 2 |fl


Name                : AdobeARMservice
DisplayName         : Adobe Acrobat Update Service
Status              : Running
DependentServices   : {}
ServicesDependedOn  : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : True
ServiceType         : Win32OwnProcess

Name                : Appinfo
DisplayName         : Application Information
Status              : Running
DependentServices   : {}
ServicesDependedOn  : {RpcSs, ProfSvc}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : True
ServiceType         : Win32OwnProcess, Win32ShareProcess
```
 Windows có ba loại services chính: **Local Services**, **Network Services**, và **System Services**. Thông thường, chỉ người dùng có quyền quản trị mới có thể tạo, chỉnh sửa, và xóa các services. Các sai cấu hình về quyền của service thường là một điểm yếu dẫn đến leo thang đặc quyền trên các hệ thống Windows.

Trong Windows, có một số dịch vụ hệ thống quan trọng không thể dừng và khởi động lại mà không cần khởi động lại hệ thống. Nếu chúng ta cập nhật bất kỳ tệp hoặc tài nguyên nào được sử dụng bởi một trong những dịch vụ này, chúng ta phải khởi động lại hệ thống.
### Các services quan trọng:

| **Service**                  | **Mô tả**                                                                                                                                                                                                                   |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **smss.exe**                 | **Session Manager SubSystem**. Chịu trách nhiệm quản lý các phiên làm việc trên hệ thống.                                                                                                                                   |
| **csrss.exe**                | **Client Server Runtime Process**. Phần chạy trong chế độ người dùng của hệ thống con Windows.                                                                                                                              |
| **wininit.exe**              | Khởi động tệp Wininit .ini liệt kê tất cả các thay đổi cần thực hiện cho Windows khi máy tính khởi động lại sau khi cài đặt một chương trình.                                                                               |
| **logonui.exe**              | Hỗ trợ đăng nhập người dùng vào máy tính.                                                                                                                                                                                   |
| **lsass.exe**                | **Local Security Authentication Server** xác minh tính hợp lệ của đăng nhập người dùng vào máy tính hoặc máy chủ. Tạo tiến trình chịu trách nhiệm xác thực người dùng cho dịch vụ **Winlogon**.                             |
| **services.exe**             | Quản lý hoạt động khởi động và dừng các services.                                                                                                                                                                           |
| **winlogon.exe**             | Chịu trách nhiệm quản lý chuỗi chú ý bảo mật, tải hồ sơ người dùng khi đăng nhập và khóa máy tính khi trình bảo vệ màn hình đang chạy.                                                                                      |
| **System**                   | Một tiến trình hệ thống nền chạy kernel của Windows.                                                                                                                                                                        |
| **svchost.exe với RPCSS**    | Quản lý các dịch vụ hệ thống chạy từ các dynamic-link libraries (tệp có đuôi .dll) như "Automatic Updates", "Windows Firewall", và "Plug and Play". Sử dụng **Remote Procedure Call (RPC) Service**.                        |
| **svchost.exe với Dcom/PnP** | Quản lý các dịch vụ hệ thống chạy từ các dynamic-link libraries như "Automatic Updates", "Windows Firewall", và "Plug and Play". Sử dụng **Distributed Component Object Model (DCOM)** và **Plug and Play (PnP)** services. |
https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_components#Services




## Windows Sercurity 

### Security Identifier (SID)


Mỗi nguyên tắc bảo mật trên hệ thống đều có một mã định danh bảo mật (SID) duy nhất. Hệ thống tự động tạo SID.
Ví dụ có 2 user giống hệt nhau trên hệ thóng, Windows vẫn nhận ra được sự khác nhau nhờ vào SID của họ
-> Xác định tất cả các hành động mà người dùng được phép thực hiện

SID bao gồm Cơ quan định danh và ID tương đối (dentifier Authority and the Relative ID - RID), Active Directory (AD), SID cũng bao gồm SID miền

```powershell-session
PS C:\htb> whoami /user

USER INFORMATION
----------------

User Name           SID
=================== =============================================
ws01\bob S-1-5-21-674899381-4069889467-2080702030-1002
```

S-1-5-21-674899381-4069889467-2080702030-1002
+ S: Xác định chuỗi là SID
+ 1:  Revision Level - cấp độ sửa đổi, ko bao giờ thay đổi luôn luôn là 1
+ 5: Identifier-authority - Chuỗi 48 bit xác định authority (máy tính hoặc mạng) đã tạo SID.
+ 21: Subauthority1
+ 674899381-4069889467-2080702030: Subauthority2 - máy tính (hoặc tên miền) nào đã tạo số đó
+ 1002: Subauthority3 - RID để phân biệt tài khoản này với tài khoản khác. Người dùng bình thường, khách, quản trị viên hay thành viên của một số nhóm khác

### Security Accounts Manager (SAM) and Access Control Entries (ACE)

 SAM cấp quyền cho mạng để thực hiện các quy trình cụ thể.

Access Control Entries (ACE): quản lý quyền truy cập
Access Control Lists (ACL): chứa ACEs xác định người dùng, nhóm hoặc quy trình nào có quyền truy cập vào tệp hoặc thực thi một quy trình.

discretionary access control list (DACL) : Xác định những người được ủy thác được phép hoặc bị từ chối truy cập vào một đối tượng có thể bảo mật 
+ Khi một process truy cập vào đối tượng đó, hệ thống sẽ kiểm tra các ACE trong DACL của đối tượng để xác định quyền truy cập -> Nếu ko có DACL hệ thống sẽ cấp quyền truy cập đầy đủ cho mọi người -> Nếu DACL của đối tượng ko có ACE hệ thống sẽ từ chối mọi nỗ lực truy cập 

system access control list (SACL): Cho phép các administrator ghi lại các lần thử truy cập vào đối tượng bảo mật, mỗi ACE thì chỉ các loại nỗ lực truy cập 


https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists

### User Account Control (UAC)

Tính năng của Windows để giảm thiểu malware thực thi, Admin Approval Mode trong UAC, được thiết kế để ngăn chặn việc cài đặt phần mềm không mong muốn mà quản trị viên không biết hoặc để ngăn chặn việc thực hiện các thay đổi trên toàn hệ thống

https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works



![[Pasted image 20240911182003.png]]



## Registry

là cơ sở dữ liệu phân cấp lưu trữ các cài đặt cấp thấp cho hệ điều hành Microsoft Windows và cho các ứng dụng chọn sử dụng registry. The [kernel](https://en.wikipedia.org/wiki/Kernel_(operating_system) "Kernel (operating system)"), [device drivers](https://en.wikipedia.org/wiki/Device_driver "Device driver"), [services](https://en.wikipedia.org/wiki/Windows_service "Windows service"), [Security Accounts Manager](https://en.wikipedia.org/wiki/Security_Accounts_Manager "Security Accounts Manager"), and [user interfaces](https://en.wikipedia.org/wiki/Graphical_user_interface "Graphical user interface") Đều sử dụng registry
https://en.wikipedia.org/wiki/Windows_Registry

### Registry value types: 
https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-value-types

### Location Registry
HKEY_LOCAL_MACHINE\\SYSTEM : \\system32\\config\\system

HKEY_LOCAL_MACHINE\\SAM : \\system32\\config\\sam

HKEY_LOCAL_MACHINE\\SECURITY : \\system32\\config\\security

HKEY_LOCAL_MACHINE\\SOFTWARE : \\system32\\config\\software

HKEY_USERS\\UserProfile : \\winnt\\profiles\\username

HKEY_USERS.DEFAULT : \\system32\\config\\default


![[Pasted image 20240911182900.png]]
https://www.thewindowsclub.com/where-are-the-windows-registry-files-located-in-windows-7

### Run and RunOnce Registry Keys
https://learn.microsoft.com/en-us/windows/win32/setupapi/run-and-runonce-registry-keys


```powershell-session
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```


Run & RunOnce registry key làm chương trình chạy khi user log on 
+ Run làm chương trình chạy mỗi lần user log
+ RunOnce chỉ chạy một lần, và sau đó xóa Key
=> 2 key này đều có thể được set bởi USER



# CHEETSHEET COMMAND WINDOWS PRI

- `Get-WmiObject -Class win32_OperatingSystem`: Lấy thông tin về hệ điều hành
- `dir c:\ /a`: Xem tất cả các tệp và thư mục trong thư mục gốc `C:\`
- `tree <directory>`: Hiển thị cấu trúc thư mục dưới dạng đồ họa của một đường dẫn
- `tree c:\ /f | more`: Xem từng trang kết quả của lệnh `tree`
- `icacls <directory>`: Xem quyền được thiết lập trên một thư mục
- `icacls c:\users /grant joe:f`: Cấp cho người dùng toàn quyền trên một thư mục
- `icacls c:\users /remove joe`: Xóa quyền của người dùng trên một thư mục
- `Get-Service`: PowerShell cmdlet để xem các dịch vụ đang chạy
- `help <command>`: Hiển thị menu trợ giúp cho một lệnh cụ thể
- `get-alias`: Liệt kê các alias trong PowerShell
- `New-Alias -Name "Show-Files" Get-ChildItem`: Tạo một alias PowerShell mới
- `Get-Module | select Name,ExportedCommands | fl`: Xem các module PowerShell đã được nhập và các lệnh liên quan của chúng
- `Get-ExecutionPolicy -List`: Xem chính sách thực thi của PowerShell
- `Set-ExecutionPolicy Bypass -Scope Process`: Thiết lập chính sách thực thi PowerShell để bỏ qua cho phiên hiện tại
- `wmic os list brief`: Lấy thông tin về hệ điều hành bằng `wmic`
- `Invoke-WmiMethod`: Gọi các phương thức của đối tượng WMI
- `whoami /user`: Xem SID của người dùng hiện tại
- `reg query <key>`: Xem thông tin về một khóa registry
- `Get-MpComputerStatus`: Kiểm tra các cài đặt bảo vệ của Windows Defender
- `sconfig`: Tải menu cấu hình máy chủ trong Windows Server Core




# Windows Logs
https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx?i=j
### 1. Event Viewer

Userfull Win
dows Event logs:

5 loại 
+ Application: Infomation, error
+ System: Information
+ Sercurity:  Event
+ Setup: Activity setup
+ Forwarded Event (Log từ máy khác)

Đuôi "**.evtx**"

![[Pasted image 20240808142730.png]]



![[Pasted image 20240808143400.png]]

Một vài thành phần chính:
1. `Log Name`: Tên của loại log đó (Application, sercurity,...)
2. `Source`: Phần mềm để ghi lại event đó
3. `Event ID`: Mã của event
4. `Task Category`: chứa giá trị, hay tên cho biết nội dung 
5. `Level`: Mức độ bảo mật (Information, Warning, Error, Critical, and Verbose).
6. `Keywords`:  "Audit Success" or "Audit Failure" ở sercurity log -  success khi 
7. `User`: user account đang logged lúc sự kiện xảy ra
8. `OpCode`: Hoạt động cụ thể mà sự kiện đang báo cáo
9. `Logged`: time & date khi event được log
10. `Computer`: Name PC
11. `XML Data`: Thông tin dưới dạng XML




1. System log:
[Event ID 1074](https://serverfault.com/questions/885601/windows-event-codes-for-startup-shutdown-lock-unlock) `(System Shutdown/Restart)`: Tại sao hệ thống tắt hoặc khởi động, -> giám sát biết lí do vì sao tắt, có phải do malware ko

[Event ID 6005](https://superuser.com/questions/1137371/how-to-find-out-if-windows-was-running-at-a-given-time) `(The Event log service was started)`:  thời gian mà Event log khởi động -> Xác định thời gian nó mở, cũng như phát hiện có khởi động lại trái phép

[Event ID 6006](https://learn.microsoft.com/en-us/answers/questions/235563/server-issue) `(The Event log service was stopped)`: Thông thường xảy ra khi hệ thống shutdown -> Các trường hợp bất thường cần được xem xét

[Event ID 6013](https://serverfault.com/questions/885601/windows-event-codes-for-startup-shutdown-lock-unlock) `(Windows uptime)`: Xảy ra 1 lần mỗi ngày -> hiển thị thời gian hoạt động của hệ thống tính bằng giây ->Thời gian ngắn hơn dự kiến -> hệ thống đã khởi động lại -> cần xem xét

[Event ID 7040](https://www.slideshare.net/Hackerhurricane/finding-attacks-with-these-6-events) `(Service status change)`: Thay đổi trạng thái của service, thủ công -> tự động or ngược lại, -> nếu cái nào quan trọng bị thay đổi -> cần xem xét


2. Sercurity Logs
[Event ID 1102](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=1102) `(The audit log was cleared)`: Các thay đổi liên quan tới vấn đề nào đó log, -> nếu bị xóa cần xem xét

[Event ID 1116](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) `(Antivirus malware detection)`: Windef phát hiện virus

[Event ID 1118](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) `(Antivirus remediation activity has started)` : Windef bắt đầu và loại bỏ hay cách ly nguồn virus

[Event ID 1119](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) `(Antivirus remediation activity has succeeded)`: Windef thành công loại bỏ hay cách ly

[Event ID 1120](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) `(Antivirus remediation activity has failed)`: Windef thất bại trong việc loại bỏ hay cách ly

[Event ID 4624](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624) `(Successful Logon)`): đăng nhập thành công 

[Event ID 4625](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625) `(Failed Logon)` Đăng nhập thất bại -> nhiều lần thất bại -> brute force

[Event ID 4648](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4648) `(A logon was attempted using explicit credentials)`: Administrator was logged on to the local computer.
Tạo ra một phiên bản của nó với 2 4624

[Event ID 4656](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4656) `(A handle to an object was requested)`: Một handel đến một đối tượng (như tệp, khóa registry, hoặc quá trình) được yêu cầu. 

[Event ID 4672](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4672) `(Special Privileges Assigned to a New Logon)`: Sự kiện này được ghi lại mỗi khi một tài khoản đăng nhập với quyền siêu người dùng. Việc theo dõi các sự kiện này giúp đảm bảo rằng quyền siêu người dùng không bị lạm dụng 

[Event ID 4698](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4698) `(A scheduled task was created)`:  Lập lịch sự kiện -> Theo dõi xem có backdoor hay gì 

[Event ID 4700](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4700) & [Event ID 4701](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4701) `(A scheduled task was enabled/disabled)` : Điều này ghi lại việc kích hoạt hoặc vô hiệu hóa một nhiệm vụ đã được lập lịch. Các nhiệm vụ đã được lập lịch thường bị kẻ tấn công thao túng để duy trì hoặc chạy mã độc, do đó các nhật ký này có thể cung cấp thông tin quan trọng về các hoạt động đáng ngờ.

[Event ID 4702](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4702) `(A scheduled task was updated)`: giống 4698

[Event ID 4719](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4719) `(System audit policy was changed)`:  Các thay đổi liên quan tới vấn đề nào đó log, -> nếu bị THAY ĐÔI cần xem xét

[Event ID 4738](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4738) `(A user account was changed)`: Bất kì thay đổi với user account, quyền, nhóm, cài đặt -> chiếm quyền

[Event ID 4771](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4771) `(Kerberos pre-authentication failed)` : giống 4625 đăng nhập thất bại cho xác thự Kerberos -> Brute Force

[Event ID 4776](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4776) `(The domain controller attempted to validate the credentials for an account)` :Các lần xác thực thành công hay thất bại -> brute force

 [Event ID 4907](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4907), which signifies an audit policy change.:

[Event ID 5001](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) `(Antivirus real-time protection configuration has changed)`: thiết lập cấu hình của windef -> làm yếu windef

[Event ID 5140](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5140) `(A network share object was accessed)`: mỗi khi một chia sẻ mạng được truy cập

[Event ID 5142](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5142) `(A network share object was added)` tạo ra một chia sẻ mạng mới  -> phát tán malware

[Event ID 5145](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5145) `(A network share object was checked to see whether client can be granted desired access)` : ai đó đã cố gắng truy cập vào một chia sẻ mạng

[Event ID 5157](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5157) `(The Windows Filtering Platform has blocked a connection)`sự kiện này được ghi lại khi Nền tảng Lọc Windows chặn một lần cố gắng kết nối.

[Event ID 7045](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=7045) `(A service was installed in the system)`: service được cài đặt  đột ngột -> malware'
\

### 2. Sysmon

https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx?i=j
 **Note**: If we observe Sysmon event IDs 1 and 3 (related to "dangerous" or uncommon binaries) occurring within a short time frame, it could potentially indicate the presence of a process communicating with a Command and Control (C2) server.


- [Sysmon Event ID 1 - Process Creation](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90001):hunt parent-child process
- [Sysmon Event ID 2 - A process changed a file creation time](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90002)Giúp phát hiện các cuộc tấn công "time stomp",  thay đổi thời gian tạo tệp. Tuy nhiên, không phải tất cả các hành động này đều nghi vấn
- [Sysmon Event ID 3 - Network connection](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90003): Một nguồn gây nhiễu nhiều vì các máy tính luôn thiết lập kết nối mạng. Chúng ta có thể phát hiện ra các bất thường, nhưng hãy cân nhắc tìm kiếm sysmon 1 và 12 trước
- [Sysmon Event ID 4 - Sysmon service state changed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90004): Có thể hữu ích trong việc săn tìm nếu kẻ tấn công cố gắng dừng Sysmon, mặc dù đa số các sự kiện này có thể là vô hại và mang tính thông tin, vì Sysmon thường xuyên được khởi động và dừng hợp lệ.
- [Sysmon Event ID 5 - Process terminated](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90005): phát hiện kill các tiến trình quan trọng hoặc sử dụng các tiến trình tạm thời. Ví dụ, Cobalt Strike thường sinh ra các tiến trình tạm thời như werfault, việc kết thúc chúng sẽ được ghi nhận ở đây, cũng như việc tạo ra chúng trong ID 1.
- [Sysmon Event ID 6 - Driver loaded](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90006): Có thể là dấu hiệu của các cuộc tấn công BYOD (bring your own driver), mặc dù điều này ít phổ biến hơn. 
- [Sysmon Event ID 7 - Image loaded](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90007): Cho phép chúng ta theo dõi việc nạp các dll, hữu ích trong việc phát hiện các cuộc tấn công DLL hijack.
- [Sysmon Event ID 8 - CreateRemoteThread](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90008):Có thể giúp xác định các luồng bị tiêm nhiễm. Mặc dù các luồng từ xa có thể được tạo ra hợp lệ, nhưng nếu kẻ tấn công lạm dụng API này -> có thể truy vết tiến trình rogue của chúng và những gì chúng đã tiêm vào.
- [Sysmon Event ID 10 - ProcessAccess](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90010): Hữu ích trong việc phát hiện tiêm mã từ xa và dump bộ nhớ, vì nó ghi lại khi các handle trên tiến trình được thực hiện.
- [Sysmon Event ID 11 - FileCreate](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90011): Với nhiều tệp được tạo ra thường xuyên do cập nhật, tải xuống, v.v., có thể sẽ khó nhắm mục tiêu trực tiếp vào đây. Tuy nhiên, các sự kiện này có thể hữu ích trong việc tương quan hoặc xác định nguồn gốc của một tệp sau này
- [Sysmon Event ID 12 - RegistryEvent (Object create and delete)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90012) & [Sysmon Event ID 13 - RegistryEvent (Value Set)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90013):Mặc dù có nhiều sự kiện diễn ra ở đây, nhiều sự kiện registry có thể là ác ý, và nếu biết rõ những gì cần tìm -> có thể hữu ích
- [Sysmon Event ID 15 - FileCreateStreamHash](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90015): Liên quan đến các luồng tệp và "Mark of the Web" liên quan đến tải xuống từ bên ngoài
- [Sysmon Event ID 16 - Sysmon config state changed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90016): Ghi lại các thay đổi trong cấu hình Sysmon, hữu ích trong việc phát hiện sự can thiệp.
- [Sysmon Event ID 17 - Pipe created](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90017) & [Sysmon Event ID 18 - Pipe connected](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90018):  việc tạo và kết nối các pipe, liên lạc giữa các tiến trình của phần mềm độc hại, sử dụng PsExec hay SMB.
- [Sysmon Event ID 22 - DNSEvent](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90022): theo dõi các truy vấn DNS, có thể hữu ích cho việc giám sát các beacon resolutions và DNS beacons.
- [Sysmon Event ID 23 - FileDelete](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90023): Giám sát việc xóa tệp, có thể cung cấp thông tin về việc liệu tác nhân đe dọa có xóa sạch phần mềm độc hại của họ, xóa các tệp quan trọng hay có thể là cố gắng thực hiện một cuộc tấn công ransomware.
- [Sysmon Event ID 25 - ProcessTampering (Process image change)](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon): Cảnh báo về các hành vi như process herpadering, hoạt động như một bộ lọc cảnh báo mini của AV.

# 3. Event Tracing For Windows (ETW)

Sử dụng cơ chế đệm và ghi nhật ký được triển khai trong kernel, ETW cung cấp cơ chế theo dõi các sự kiện được tạo ra bởi cả ứng dụng ở chế độ người dùng và trình điều khiển thiết bị ở chế độ kernel.

### 3.  Get-WinEvent

https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.4&viewFallbackFrom=powershell-7.3
