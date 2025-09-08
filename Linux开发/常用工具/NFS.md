**网络文件系统**

***

<br/>

### 挂载网络目录

将主机的`/home/book/nfs_rootfs`目录通过网络IP挂载到本地`/mnt`目录

以下命令在开发机执行

```text
mount -t nfs -o nolock,vers=3 192.168.1.31:/home/book/nfs_rootfs /mnt
```