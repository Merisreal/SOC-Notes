## Password Spraying

https://cookiearena.org/hoc-pentester/tan-cong-password-spraying-attack/
### Password Spraying

Không giống như BruteForce với phương pháp Dictionary Attack 
+ Dictionary Attack thử password với 2 danh sách, username.txt và password.txt có liên quan tới mục tiêu -> sau đó thử tất cả các trường hợp khả thi 
+ Tuy nghiên với Account Lockout Policy -> sẽ tự động khóa tài khoản người dùng sau khi nhập sai N lần 

Password Spraying thử một vài mật khẩu với nhiều user, để bypass policy block bruteforce 
Ví dụ: thử 1 lần  vơi usernameA, 1 lần với usernameB
https://github.com/insidetrust/statistically-likely-usernames
https://github.com/Greenwolf/Spray
![Pasted image 20240917120337](https://github.com/user-attachments/assets/ba3c8a6e-3355-4166-8f2b-a649a93e9e39)

#### Detection
Một vài Event có thể phát hiện Spraying password

+ `4625 Failed Logon`
- `4768 and ErrorCode 0x6 - Kerberos Invalid Users`
- `4768 and ErrorCode 0x12 - Kerberos Disabled Users`
- `4776 and ErrorCode 0xC000006A - NTLM Invalid Users`
- `4776 and ErrorCode 0xC0000064 - NTLM Wrong Password`
- `4648 - Authenticate Using Explicit Credentials`
- `4771 - Kerberos Pre-Authentication Failed`



#### Splunk:
```shell-session
index=main earliest=1690280680 latest=1690289489 source="WinEventLog:Security" EventCode=4625
| bin span=15m _time
| stats values(user) as Users, dc(user) as dc_user by src, Source_Network_Address, dest, EventCode, Failure_Reason
```



| bin span=15m _time: giới hạn time trong 15p

stats values(user) as Users, dc(user) as dc_user by src: lọc ra các user bị trùng


**=> Tìm các event 4625 (failed logon), với giới hạn time =15 phút -> hiện các user ko trùng, hiện địa chỉ mạng nguồn, đích, event code, lỗi**

![Pasted image 20240917120608](https://github.com/user-attachments/assets/24ec8c7c-1498-4517-9db4-7ac9ff3a55e0)
