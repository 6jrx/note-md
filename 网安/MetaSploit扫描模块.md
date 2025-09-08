#### 主机发现

***

模块路径

> modules/auxiliary/scanner/discovery/

搜索模块

```sh
msf6 > search aux /scanner/discovery

Matching Modules
================

   #  Name                                                            Disclosure Date  Rank    Check  Description
   -  ----                                                            ---------------  ----    -----  -----------
   0  auxiliary/scanner/discovery/arp_sweep                                            normal  No     ARP Sweep Local Network Discovery
   1  auxiliary/scanner/discovery/ipv6_multicast_ping                                  normal  No     IPv6 Link Local/Node Local Ping Discovery
   2  auxiliary/scanner/discovery/ipv6_neighbor                                        normal  No     IPv6 Local Neighbor Discovery
   3  auxiliary/scanner/discovery/ipv6_neighbor_router_advertisement                   normal  No     IPv6 Local Neighbor Discovery Using Router Advertisement
   4  auxiliary/scanner/discovery/empty_udp                                            normal  No     UDP Empty Prober
   5  auxiliary/scanner/discovery/udp_probe                                            normal  No     UDP Service Prober
   6  auxiliary/scanner/discovery/udp_sweep                                            normal  No     UDP Service Sweeper


Interact with a module by name or index. For example info 6, use 6 or use auxiliary/scanner/discovery/udp_sweep
```

使用模块

```sh
msf6 > use auxiliary/scanner/discovery/arp_sweep
msf6 auxiliary(scanner/discovery/arp_sweep) > set RHOSTS 192.168.64.0/24
msf6 auxiliary(scanner/discovery/arp_sweep) > set threads 10
msf6 auxiliary(scanner/discovery/arp_sweep) > run
```

<br/>

<br/>

#### 服务扫描

***

确定主机开发端口后，对相应端口上所运行的服务信息进行挖掘

在Metasploit 的Scanner辅助模块中，用于服务扫描和查点的工具常以`[service_name]_version`和`service_name]_login`命名

`[service_name]_version`可用于便利网络中包含某种服务的主机，进一步去定服务的版本

`service_name]_login`对某种服务进行口令探测攻击

模块搜索

```sh
msf6 > search _version

Matching Modules
================

   #   Name                                                     Disclosure Date  Rank    Check  Description
   -   ----                                                     ---------------  ----    -----  -----------
   0   auxiliary/scanner/snmp/aix_version                                        normal  No     AIX SNMP Scanner Auxiliary Module
   1   auxiliary/scanner/amqp/amqp_version                                       normal  No     AMQP 0-9-1 Version Scanner
   2   auxiliary/scanner/http/apache_nifi_version                                normal  No     Apache NiFi Version Scanner
   3   auxiliary/scanner/misc/rocketmq_version                                   normal  No     Apache RocketMQ Version Scanner
   4   auxiliary/scanner/http/coldfusion_version                                 normal  No     ColdFusion Version Scanner
   5   auxiliary/scanner/db2/db2_version                                         normal  No     DB2 Probe Utility
   6   auxiliary/scanner/scada/digi_addp_version                                 normal  No     Digi ADDP Information Discovery
   7   auxiliary/scanner/scada/digi_realport_version                             normal  No     Digi RealPort Serial Server Version
   8   auxiliary/scanner/http/docker_version                                     normal  No     Docker Server Version Scanner
   9   auxiliary/scanner/http/emby_version_ssrf                                  normal  No     Emby Version Scanner
   10  auxiliary/scanner/ftp/ftp_version                                         normal  No     FTP Version Scanner
```

模块使用

```sh
msf6 >use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS 121.41.50.152
msf6 auxiliary(scanner/ssh/ssh_version) > set THREADS 5
msf6 auxiliary(scanner/ssh/ssh_version) > run
```

<br/>

其他服务扫描

- Telnet
  ```sh
  search scanner/telnet
  ```

- SSH
  ```sh
  search scanner/ssh
  ```

- Oracle
  ```sh
  search scanner/oracle
  ```

- SMB
  ```sh
  search scanner/smb
  ```

- MSSQL
  ```sh
  search scanner/mssql
  ```

- FTP
  ```sh
  search scanner/ftp
  ```

- SMTP
  ```sh
  search scanner/snmp
  ```

<br/>

<br/>

#### WEB扫描-WMAP

***

web应用漏洞查找等模块基本都在`modules/auxiliary/`下，Metasploit内置了WMAP web扫描器

a.初始化

```sh
msf6 > load wmap

.-.-.-..-.-.-..---..---.
| | | || | | || | || |-'
`-----'`-'-'-'`-^-'`-'
[WMAP 1.5.1] ===  et [  ] metasploit.com 2012
[*] Successfully loaded plugin: wmap
```

b.查看选项

```sh
msf6 > help wmap

wmap Commands
=============

    Command       Description
    -------       -----------
    wmap_modules  Manage wmap modules
    wmap_nodes    Manage nodes
    wmap_run      Test targets
    wmap_sites    Manage sites
    wmap_targets  Manage targets
    wmap_vulns    Display web vulns
```

c.使用扫描

```sh
msf6 > wmap_sites -a http://121.41.50.152           #添加要扫描的网站
msf6 > wmap_sites -l                                #查看添加的项目
msf6 > wmap_targets -t http://121.41.50.152         #把添加的网站作为扫描目标
msf6 > wmap_run -t                                  #查看哪些模块将在扫描中使用
msf6 > wmap_run -e                                  #开始扫描
msf6 > vulns                                        #查看漏洞信息
```

<br/>

#### 端口扫描

***