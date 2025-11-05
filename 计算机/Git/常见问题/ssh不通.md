## 一、代理软件导致的协议不通


开启代理软件后经常碰到ssh流量被TUN代理导致服务端无法识别，通常需要在本地ssh目录加入一下配置


```bash
# vim ~/.ssh/config
# 添加如下配置
# You can also manually open the file and copy/paste the section below
# Add section below to it
Host github.com
  Hostname ssh.github.com
  Port 443
```
