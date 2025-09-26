Minicom 是 Linux 下一款功能强大的串口调试工具，很多朋友用它来连接和管理嵌入式设备。由于它基于文本界面，刚开始接触可能会觉得有点复杂，但熟悉后你会发现它非常高效。

# 🖥️ Minicom 串口调试工具使用指南

## 1. 安装 Minicom

在 Debian/Ubuntu 系统上，你可以使用 apt 包管理器来安装 Minicom：
```bash
sudo pacman -S minicom
```
安装完成后，可以通过命令 `minicom --version` 来验证是否安装成功。

## 2. 配置 Minicom

首次使用 Minicom 前，**务必先进行配置**。使用以下命令进入配置模式：
```bash
sudo minicom -s
```
这会启动一个文本界面配置菜单。

### 2.1 串口参数设置
在配置菜单中选择 **“Serial port setup”**，你会看到类似这样的界面：
```makefile
    +-----------------------------------------------------------------------+
    | A -    Serial Device      : /dev/ttyS0                                |
    | B - Lockfile Location     : /var/lock                                 |
    | C -   Callin Program      :                                           |
    | D -  Callout Program      :                                           |
    | E -    Bps/Par/Bits       : 9600 8N1                                  |
    | F - Hardware Flow Control : Yes                                       |
    | G - Software Flow Control : No                                        |
    |                                                                       |
    |    Change which setting?                                              |
    +-----------------------------------------------------------------------+
```

你需要根据你的实际硬件连接情况调整以下选项：

*   **Serial Device (A)**：设置串口设备文件。
    *   如果是 **USB 转串口线**，通常是 `/dev/ttyUSB0` (可以使用 `dmesg | grep ttyUSB` 命令确认)
    *   如果是 **主板原生串口**，COM1 通常是 `/dev/ttyS0`, COM2 是 `/dev/ttyS1`。
*   **Bps/Par/Bits (E)**：设置**波特率（Bps）、数据位、奇偶校验和停止位**。常用的配置如 `115200 8N1` (115200 波特率，8 数据位，无校验，1 停止位) 或 `9600 8N1`。请务必确保此设置与你要连接的设备设置**完全一致**，否则会出现乱码。
*   **Hardware Flow Control (F)**：将其设置为 **No**。除非你明确知道需要硬件流控制。
*   **Software Flow Control (G)**：通常保持为 **No**。

> 💡 **操作方法**：要修改某一项，只需按下对应的字母键（如按 `A` 修改 Serial Device），然后输入新值，输入完成后按回车确认。所有串口参数修改完毕后，按回车键返回主配置菜单。

### 2.2 调制解调器设置
返回主配置菜单后，选择 **“Modem and dialing”** 选项。在这个菜单中，**清空**以下字段的内容（因为它们是为拨号调制解调器设计的，对于直接串口连接不需要）：
*   Init string (A)
*   Reset string (B)
*   Hang-up string (K)
清空后，按回车返回主配置菜单。

### 2.3 保存配置
返回主配置菜单后，选择 **“Save setup as dfl”** 选项。这将把你的配置保存为默认配置，以后直接运行 `minicom` 命令就会使用这些设置。
最后，选择 **“Exit”** 或 **“Exit from Minicom”** 退出配置菜单。

## 3. 基本使用

### 3.1 启动与连接
配置保存后，你可以使用以下命令启动 Minicom 并连接到串口设备：
```bash
sudo minicom
```
如果一切正常，你现在应该能看到从串口设备输出的信息了。如果设备已启动且配置正确，按回车键可能会看到命令行提示符（如 `#` 或 `$`）。

### 3.2 Minicom 快捷键
Minicom 的所有功能都通过快捷键访问。**默认的快捷键前缀是 `Ctrl-A`**（即先按住 `Ctrl` 键，再按 `A` 键，然后松开两者，再按功能键）。

以下是一些最常用的快捷键命令：

| 快捷键       | 功能描述                                     | 备注                                                         |
| :----------- | :------------------------------------------- | :----------------------------------------------------------- |
| **Ctrl-A Z** | **显示帮助菜单**                             | 查看所有可用的命令                         |
| Ctrl-A O     | 进入配置菜单                                 | 相当于启动时的 `minicom -s`                            |
| Ctrl-A Q     | **退出 Minicom（不复位 modem）**             | 会询问是否保存宏                              |
| **Ctrl-A X** | **退出 Minicom（并复位 modem）**             | 最常用的退出方式，会询问是否保存宏     |
| Ctrl-A C     | **清屏**                                     | 清除当前屏幕内容                             |
| Ctrl-A S     | **发送文件**                                 | 会弹出协议选择菜单，支持 Zmodem, Ymodem, Xmodem 等 |
| Ctrl-A R     | **接收文件**                                 | 需要设备端配合发送文件                         |
| Ctrl-A E     | 切换本地回显 (on/off)                        | 用于检查你输入的内容是否正确发送                 |
| Ctrl-A W     | 切换自动换行 (on/off)                        |                                           |

> 🚨 **注意权限问题**：在大多数 Linux 系统上，访问串口设备（如 `/dev/ttyUSB0`, `/dev/ttyS0`）需要 **root 权限**。这就是为什么上述命令前面通常要加 `sudo`。
> 为了避免每次都要使用 `sudo`，你可以将你的用户添加到 `dialout` 用户组（该组通常拥有串口设备的访问权限）：
> ```bash
> sudo usermod -a -G dialout $USER
> ```
> **执行此命令后，你需要注销并重新登录，或重启计算机，更改才会生效。**

## 4. 文件传输

Minicom 支持通过多种协议（如 **Zmodem**）进行文件传输，这对于嵌入式开发非常方便。

1.  **发送文件 (PC -> 设备)**：
    *   在 Minicom 中，按 `Ctrl-A S`。
    *   选择传输协议（例如 **Zmodem**）。
    *   选择你要发送的文件（通常使用空格键标记文件）。
    *   按回车开始传输。
2.  **接收文件 (设备 -> PC)**：
    *   在 Minicom 中，按 `Ctrl-A R`。
    *   选择传输协议（例如 Zmodem）。
    *   接收过程会自动开始，文件通常会保存在你配置的默认下载目录中。

> ⚡ 使用 **Zmodem** 协议通常需要在设备端也支持相应的命令（如在嵌入式 Linux 中，设备端可能需要安装 `lrzsz` 包）。

## 5. 常见问题与解决

*   **无法打开串口 / 权限不够**：
    *   确保使用了 `sudo` 或将用户加入了 `dialout` 组并已重新登录。
    *   检查设备文件路径是否正确（如 `/dev/ttyUSB0` 还是 `/dev/ttyS0`）。
*   **接收数据为乱码**：
    *   **这是最常见的问题**。**99% 的情况是波特率等串口参数设置错误**。请**仔细检查**设备端的波特率、数据位、停止位、校验位是否与 Minicom 中的设置 (**Bps/Par/Bits, 选项 E**) **完全一致**。
*   **Minicom 无法启动，提示设备被锁定**：
    *   有时非正常退出 Minicom 会在 `/var/lock` 目录下留下锁文件（如 `LCK..ttyUSB0`）。删除对应的锁文件即可解决：
        ```bash
        sudo rm /var/lock/LCK..ttyUSB0
        ```
*   **无法发送文件或接收文件**：
    *   确认设备端是否支持并准备好了相应的文件传输协议（如 Zmodem）。

## 6. 怎么找到你的串口

在 Linux 下，串口设备不像在 Windows 里叫 `COM1`、`COM2`，而是以文件的形式存在于 `/dev/` 目录下。确定你的开发板具体对应哪个设备文件，有以下几种可靠的方法。

### 方法一：最可靠的方法——插拔对比法 (推荐)

这是最直接、最不会出错的方法，强烈推荐使用。

1.  **首先，在不连接开发板的情况下**，打开终端，输入以下命令，查看当前系统已识别到的串口设备：
    ```bash
    ls /dev/ttyUSB* /dev/ttyS* /dev/ttyACM* 2>/dev/null
    ```
    *   `ttyUSB*`： 最常见的USB转串口设备（如FT232、CH340、CP2102等芯片）。
    *   `ttyACM*`： 某些USB设备（如Arduino Uno）会使用CDC ACM协议，出现在这里。
    *   `ttyS*`： 主板自带的原生串口（也称COM口）。

    此时，这个列表可能是空的，或者显示一些系统原有的设备（如 `ttyS0` 是主板上的串口）。**记下当前已有的设备。**

2.  **然后，将你的开发板通过USB线连接到电脑**。

3.  **再次执行相同的命令**：
    ```bash
    ls /dev/ttyUSB* /dev/ttyS* /dev/ttyACM* 2>/dev/null
    ```

4.  **对比两次的结果**。**新出现**的那个设备文件就是你的开发板！
    *   **例如**：第一次执行没有任何输出。插上开发板后，第二次执行输出 `/dev/ttyUSB0`。那么你的开发板就是 `/dev/ttyUSB0`。
    *   **又如**：第一次执行输出了 `/dev/ttyS0`。插上开发板后，第二次执行输出了 `/dev/ttyS0 /dev/ttyUSB0`。那么新出现的 `/dev/ttyUSB0` 就是你的开发板。


### 方法二：查看系统内核日志 (`dmesg`)

当你的开发板连接到电脑时，Linux内核会检测到硬件变化并记录日志。`dmesg` 命令可以查看这些日志。

1.  首先，打开一个终端，准备运行命令。
2.  **将你的开发板拔掉**。
3.  运行以下命令，清空之前的内核环形缓冲区，让我们能更容易看到新日志：
    ```bash
    sudo dmesg -C
    ```
4.  **迅速将开发板插入电脑**。
5.  运行 `dmesg` 命令查看最新的日志：
    ```bash
    dmesg
    ```

    你会看到类似下面的输出，这非常清楚地指明了设备名：

    ```
    [ 1234.567890] usb 1-1.2: new full-speed USB device number 7 using ehci-pci
    [ 1234.678901] usb 1-1.2: New USB device found, idVendor=10c4, idProduct=ea60
    [ 1234.678903] usb 1-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [ 1234.678904] usb 1-1.2: Product: CP2102 USB to UART Bridge Controller
    [ 1234.678905] usb 1-1.2: Manufacturer: Silicon Labs
    [ 1234.678906] usb 1-1.2: SerialNumber: 0001
    # 最关键的是下面这一行，它明确告诉你系统将这个设备分配为了 ttyUSB0
    [ 1234.789012] cp210x 1-1.2:1.0: cp210x converter detected
    [ 1234.789123] usb 1-1.2: cp210x converter now attached to ttyUSB0
    ```

    寻找最后几行，关键字是 **`attached to`**。在这个例子中，它非常明确地告诉你设备是 **`ttyUSB0`**。

6.  运行`sudo dmesg | grep tty`命令过滤之前的串口连接信息

    ```bash
    sudo dmesg | grep tty
    ```
    输出以下信息
    ```bash
    [    0.056380] printk: legacy console [tty0] enabled
    [    6.117741] systemd[1]: Created slice Slice /system/getty.
    [    6.341471] cdc_acm 7-1.2:1.0: ttyACM0: USB ACM device
    ```
    可以看到`USB ACM device`等信息。


### 方法三：使用图形化设备管理器（如果系统有桌面环境）

如果你使用的是带有图形界面的Linux发行版（如Ubuntu、Fedora等），可以更方便地查看。

1.  打开你的系统设置或应用程序菜单。
2.  搜索并打开 **“设备和打印机”** 或者类似名称的工具。
3.  插入你的开发板，稍等片刻，列表中通常会出现一个新设备，它的名字通常会包含芯片型号（如 **CP2102**、**FT232R**、**CH340** 等）。
4.  右键点击该设备，选择“属性”，在“硬件”或“详细信息”选项卡里，通常也能找到它对应的端口信息（如 `ttyUSB0`）。

---

### 总结与操作流程建议

1.  **首选【方法一】**：`ls /dev/` 插拔对比，简单粗暴，绝对准确。
2.  **如果不确定，用【方法二】**：`dmesg` 命令会给你最详细和权威的答案，告诉你是什么芯片以及分配到了哪个设备文件。
3.  图形化方法作为辅助验证。

一旦你确定了设备文件（例如 `/dev/ttyUSB0`），就可以在 Minicom、Picocom 或任何其他串口工具的配置中设置它了。

**重要提示**：别忘了解决权限问题！
```bash
# 将当前用户加入 dialout 组，避免每次使用 sudo
sudo usermod -a -G dialout $USER
```
**执行此命令后，必须注销并重新登录，或者重启电脑，更改才会生效。**

现在，你应该可以 confidently (有信心地) 找到你的开发板了！