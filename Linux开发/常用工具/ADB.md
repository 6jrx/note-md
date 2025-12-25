# 1.常用命令

- 在主机拉取开发机文件或目录

```text
adb pull /root/dir
```

<br/>

- 在主机推送文件或目录到开发机

```text
adb push /root/tmp
```





# 2.无线投屏

使用工具：`scrcpy`

项目地址：`https://github.com/Genymobile/scrcpy`

操作步骤

1. 启用开发者选项

2. 开启USB调试

3. 开启WIFI调试（如果有）

4. ADB查看设备连接

   ```cmd
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>adb devices
   List of devices attached
   TTANCN2100003543        device
   ```

5. WIFI连接

   设备连接到同一局域网中

   ```c
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>adb tcpip 9999
   restarting in TCP mode port: 9999
   
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>adb connect 192.168.2.109:9999
   connected to 192.168.2.109:9999
   
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>adb devices
   List of devices attached
   TTANCN2100003543        device
   192.168.2.109:9999      device          // 这时候可以断开USB了
       
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>scrcpy.exe          //加上 -S 选项可以关闭手机屏幕
   scrcpy 3.3.4 <https://github.com/Genymobile/scrcpy>
   INFO: ADB device found:
   INFO:     --> (tcpip)  192.168.2.109:9999              device  Titan_2
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4\scrcpy-server: 1 file pushed, 0 skipped. 24.7 MB/s (90980 bytes in 0.004s)
   [server] INFO: Device: [Unihertz] Unihertz Titan 2 (Android 15)
   INFO: Renderer: direct3d
   INFO: Texture: 1440x1440
   ```

6. 回到USB线连接

   ```cmd
   C:\Users\lj\Desktop\scrcpy-win64-v3.3.4>adb usb
   ```

   
