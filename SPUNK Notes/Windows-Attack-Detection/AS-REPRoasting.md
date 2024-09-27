## AS-REPRoasting

https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/asreproast
https://sec.vnpt.vn/2023/01/ky-thuat-attacking-kerberos-as-rep-roasting/

Tấn công vào các tài khoản user ko sử dụng pre-authentication, Trong Kerberos, pre-authentication đuọc sử dụng để nhận diện trước khi TGT được cấp -> Cho phép kẻ tấn công yêu cầu xác thực cho một người dùng từ Domain Controller mà ko cần mật khẩu.

- DONT_REQUIRE_PREAUTH -> Tài khoản không yêu cầu Kerberos pre-authentication, bất kì ai cũng có thể gửi AS-REQ của người dùng trên và nhận được phản hồi AS-REP với phần mã hóa có thể bruteforce offline để lấy mật khẩu -> tuy nhiên vẫn cần biết username 
![Pasted image 20240926181720](https://github.com/user-attachments/assets/36508262-2adc-4143-8174-7b6988300811)

![Pasted image 20240919165822](https://github.com/user-attachments/assets/fb675248-0421-49b0-9a85-f777c441c564)

### Attack Steps:
Nhận dạng các user không bật pre-authen
![Pasted image 20240919165901](https://github.com/user-attachments/assets/73e18b22-9585-4217-b797-e38acb0ad956)

Yêu cầu vé dịch vụ AS-REQ: attacker bắt đầu yêu cầu vé dịch vụ AS-REQ cho từng tài khoản người dùng mục tiêu được xác định.
![Pasted image 20240919165921](https://github.com/user-attachments/assets/9b4b3446-0231-45b9-ae0e-c3f5b1286a81)

Offline Bruteforce Attack -> done

### DETECTION

`Event ID 4768 (TGT Request)` contains a `PreAuthType`

### SPLUNK
#### Detecting AS-REPRoasting - Querying Accounts With Pre-Auth Disabled
```shell-session
index=main earliest=1690392745 latest=1690393283 source="WinEventLog:SilkService-Log" 
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter 
| search SearchFilter="*(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304)*"
```

 => trích xuất các dữ liệu từ JSON hoặc XML trong trường message ,Đổi tên các trường có tiền tố "XmlEventData." thành các trường không có tiền tố (để dễ thao tác hơn), hiện bảng theo thời gian,**** 
**ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter -> nội dung Search có trong danh sách liên quan tới sử dụng LDAP -> **userAccountControl:1.2.840.113556.1.4.803:=4194304**: Đây là một điều kiện LDAP đặc biệt, tìm các tài khoản có thuộc tính `userAccountControl` với giá trị cờ cụ thể:
- **4194304** là giá trị của cờ `PASSWORD_EXPIRED`, điều này có nghĩa là tài khoản đã hết hạn mật khẩu
**
![Pasted image 20240919170246](https://github.com/user-attachments/assets/4aa7056b-5b3e-477f-9aad-fa5488df5d3c)

#### Detecting AS-REPRoasting - TGT Requests For Accounts With Pre-Auth Disabled
```shell-session
index=main earliest=1690392745 latest=1690393283 source="WinEventLog:Security" EventCode=4768 Pre_Authentication_Type=0
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip>[0-9\.]+)"
| table _time, src_ip, user, Pre_Authentication_Type, Ticket_Options, Ticket_Encryption_Type
```


**=>Lọc Event Code 4768 (Kerberos Authentication) ->Lọc các sự kiện mà Pre-Authentication (xác thực trước) không yêu cầu**
**`| rex field=src_ip "(\:\:ffff\:)?(?<src_ip>[0-9\.]+)"`:trích xuất ip**

![Pasted image 20240919170314](https://github.com/user-attachments/assets/dec8e5e2-f020-4139-9d34-8c4df37769c6)
