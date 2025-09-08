在 Linux 中压缩打包文件夹，掌握几个常用命令就能轻松搞定。下面我用一个表格汇总这些命令和它们的常见用法，方便快速了解：

| 压缩格式 | 典型命令示例 (打包压缩)                                  | 特点                                                                 | 解压命令示例                           |
| :------- | :------------------------------------------------------- | :------------------------------------------------------------------- | :------------------------------------- |
| **.tar** | `tar -cvf archive.tar your_folder/`                     | 仅打包，不压缩，保留权限和属性                      | `tar -xvf archive.tar`                 |
| **.tar.gz** | `tar -czvf archive.tar.gz your_folder/`                 | **最常用**，较好平衡压缩率和速度                                | `tar -xzvf archive.tar.gz`             |
| **.tar.bz2** | `tar -cjvf archive.tar.bz2 your_folder/`                | 压缩率更高，适合大文件，但速度较慢                              | `tar -xjvf archive.tar.bz2`            |
| **.tar.xz** | `tar -cJvf archive.tar.xz your_folder/`                 | 压缩率很高，但速度较慢，耗时更长                                     | `tar -xJvf archive.tar.xz`             |
| **.zip** | `zip -r archive.zip your_folder/`                       | **跨平台友好**，Windows 和 macOS 都能方便打开                       | `unzip archive.zip`                    |
| **.7z**  | `7z a archive.7z your_folder/`                          | 压缩率通常很高 ，需先安装 `p7zip` 包                               | `7z x archive.7z`                      |

💡 **参数解析**：
表格命令中的参数含义如下：
*   `-c`：创建新的归档文件。
*   `-z`：使用 gzip 过滤进行压缩（用于 `.tar.gz`）。
*   `-j`：使用 bzip2 进行压缩（用于 `.tar.bz2`）。
*   `-J`：使用 xz 进行压缩（用于 `.tar.xz`）。
*   `-v`：详细输出处理的文件信息。
*   `-f`：指定归档文件名。
*   `-r`：递归操作（用于 `zip` 命令处理目录）。
*   `-x`：解压模式。

🧭 **操作技巧**

*   **只想打包，暂不压缩**：使用 `tar -cvf archive.tar your_folder/`。这在需要归档文件但不在意磁盘空间时有用。
*   **排除特定文件或目录**：`tar` 命令支持 `--exclude` 选项。例如，要排除 `your_folder` 下的 `temp` 目录和所有 `.log` 文件：
    ```bash
    tar -czvf archive.tar.gz your_folder/ --exclude='temp' --exclude='*.log'
    ```
*   **分卷压缩大文件**（例如每卷 100M）：这在进行网络传输或有大小限制时很实用。对于 `tar` 归档，可以结合 `split` 命令：
    ```bash
    tar -czvf - your_folder/ | split -b 100M - archive.tar.gz.part
    ```
    这将生成类似 `archive.tar.gz.partaa`, `archive.tar.gz.partab` 的文件。解压时使用 `cat archive.tar.gz.part* | tar -xzvf -`。
*   **确保压缩包内容路径正确**：在压缩时，**相对路径** 和 **绝对路径** 的行为不同。通常建议在目标文件夹的**父目录**中执行压缩命令，这样解压后不会包含多余的顶层目录。例如，如果文件夹是 `/home/user/data`，在 `/home/user` 下执行 `tar -czvf data.tar.gz data/`。

🔍 **选择建议**

*   **通用与分享**：如果需要与 **Windows 或 macOS 用户分享**，优先选择 **`.zip`** 格式 。
*   **Linux 环境高效压缩**：在 Linux 系统间备份或传输数据，**`.tar.gz`** 是**最常见和均衡**的选择 。
*   **追求更高压缩率**：若不介意更长的压缩时间（如用于归档很少访问的大文件），可考虑 **`.tar.xz`** 或 **`.7z`** 。

