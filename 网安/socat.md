#### 使用命令

<br/>

> -lh 将主机名添加到日志消息
-v 详细数据流量，文本
-x 详细数据流量，十六进制
-d 增加详细程度（最多使用4次；建议使用2次）
-lf <logfile> 记录到文件

<br/>

```sh
socat -d -d -d -d -lh -v -lf /var/log/socat.log TCP4-LISTEN:2087,fork TCP4:121.41.50.152:6623
```
