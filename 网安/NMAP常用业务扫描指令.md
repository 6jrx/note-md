> -sS         TCP SYN
> 
> -sT          Connect()
> 
> -sA          ACK
> 
> -sW          Window
> 
> -sM          Maimon扫描

#### 0x01 NetBios协议探测

```sh
nmap -sU -T4 --script nbstat.nse -p137 10.10.10.0/24
```

<br/>

#### 0x02 扫描C段存活

```sh
nmap -sn -PE -T4 192.168.0.0/24
```

<br/>

#### 0x03 UDP协议探测

```sh
nmap -sU –T4 -sV --max-retries 1 192.168.1.100 -p 500
```

<br/>

#### 0x04 ARP协议探测

```sh
nmap -sn -PR 192.168.1.1/24
```

<br/>

#### 0x05 SMB协议探测

```sh
nmap -sU -sS --script smb-enum-shares.nse -p 445 192.168.1.119
```

<br/>

#### 0x06 内网存活探测

```sh
proxychains nmap -sT -Pn -p- -n -T4 192.168.22.22
```