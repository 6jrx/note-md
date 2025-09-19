## 搭建Ubuntu 18.4容器

在使用linux开发机中需要用到低版本的编译环境，比如gcc7.5，之前一直使用的ubuntu_18.4虚拟机作为开发机，所以这里直接docker来搭建环境，减少虚拟机开销。

### 📦 使用 Ubuntu 18.04 官方镜像

#### 操作步骤

1.  **创建 `Dockerfile`**
    创建一个名为 `Dockerfile` 的文件，内容如下。这个文件会指导 Docker 构建出你需要的环境。

    ```dockerfile
    # 使用 Ubuntu 18.04 作为基础镜像
    FROM ubuntu:18.04

    # 设置环境变量，避免 apt-get 安装过程中交互式提示
    ENV DEBIAN_FRONTEND=noninteractive

    # 更新软件包列表并安装 GCC 7.5、Python 环境及常用构建工具
    RUN apt-get update && \
        apt-get install -y --no-install-recommends \
        software-properties-common \
        build-essential \
        gcc-7 \
        g++-7 \
        python \
        python3 \
        python3-pip \
        && rm -rf /var/lib/apt/lists/*

    # 设置 GCC-7 和 G++-7 为默认的编译器（可选，但建议）
    RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70 \
        && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 70

    # 设置工作目录
    WORKDIR /app

    # 设置默认的启动命令
    CMD ["/bin/bash"]
    ```

2.  **构建 Docker 镜像**
    在终端中，切换到包含 `Dockerfile` 的目录，执行以下命令来构建镜像。`-t` 参数用于给镜像命名和打标签，这里我们将其命名为 `ubuntu18-gcc7.5-env`。

    ```bash
    docker build -t ubuntu18-gcc7.5-env .
    ```

3.  **运行容器并验证环境**
    镜像构建成功后，你可以运行一个交互式容器来使用这个环境。

    ```bash
    docker run -it --name my-dev-env ubuntu18-gcc7.5-env
    ```
    在容器内部，你可以执行以下命令来验证环境：
    *   **验证 GCC 版本**：
        ```bash
        gcc --version
        ```
        （应显示 `gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0` 或类似信息）
    *   **验证 Python 版本**：
        ```bash
        python --version   # 检查 Python 2.x
        python3 --version  # 检查 Python 3.x
        ```
        （应分别显示 `Python 2.7.17` 和 `Python 3.6.9` 或类似版本信息，这与 Ubuntu 18.04 官方源保持一致）

#### 管理你的开发环境

为了在容器和宿主机之间方便地共享代码文件，建议在运行容器时使用 `-v` 参数将宿主机的目录挂载到容器内：

```bash
docker run -it --name my-dev-env -v /path/to/your/code:/app ubuntu18-gcc7.5-env
```

这样，你在宿主机 `/path/to/your/code` 目录下的文件改动，会立刻反映在容器的 `/app` 目录中，方便进行编译和测试。

### 🔄 其他方式：使用预构建的 GCC 镜像

你也可以考虑直接使用 Docker Hub 上官方提供的 `gcc` 镜像，并通过标签指定版本。例如，`gcc:7.5` 镜像通常基于较新的 Debian 或 Ubuntu 版本构建。**但需要注意**，这类镜像的基础系统可能不是 Ubuntu 18.04，其内置的 Python 版本可能与你期望的（Ubuntu 18.04 自带的 Python 3.6）不一致。

如果你的项目对系统底层库或 Python 版本有严格要求，那么**第一种方法（基于 `ubuntu:18.04` 自定义构建）是更可靠的选择**。

### 💡 两种主要安装方法对比

| 特性 | 基于 `ubuntu:18.04` 自定义构建 | 使用 `gcc:7.5` 官方镜像 |
| :--- | :--- | :--- |
| **基础系统** | **Ubuntu 18.04** | Debian / Ubuntu (较新版本) |
| **Python 环境** | **原生 Python 2.7 和 3.6** | 取决于基础系统版本 |
| **GCC 版本** | 7.5.0 (来自 Ubuntu 官方源) | 7.5.0 |
| **灵活性** | **高**，可随意安装其他软件包 | 中等，取决于基础镜像 |
| **可靠性** | **高**，环境与 Ubuntu 18.04 完全一致 | 可能因基础镜像更新而变化 |

### 💎 总结

要进入你已经搭建好的 Ubuntu 18.04 且包含 GCC 7.5 的开发环境 Docker 容器，可以通过几个简单的命令来实现。

### 🔍 进入容步骤

1.  **查看运行中的容器**：首先，你需要知道要进入的容器的名称或 ID。打开终端，使用以下命令：
    ```bash
    docker ps
    ```
    这会列出所有正在运行的容器，记下你想进入的那个容器的 **NAME** 或 **CONTAINER ID**。

2.  **执行进入容器的命令**：使用 `docker exec` 命令来进入一个正在运行的容器。这是最常用的方式。
    ```bash
    docker exec -it <容器名称或ID> /bin/bash
    ```
    *   **`-it`**：这是 `-i`（保持标准输入打开）和 `-t`（分配一个伪终端）选项的组合，它们能让你以**交互模式**进入容器，就像在使用一个普通的 shell 一样。
    *   **`<容器名称或ID>`**：替换成你第一步中查到的容器的实际名称或 ID。
    *   **`/bin/bash`**：这是指定在容器内启动 Bash shell。如果你的容器镜像基于其他系统（如 Alpine），可能需要使用 `/bin/sh`。

    例如，如果你的容器名叫 `my_ubuntu_gcc_container`，命令就是：
    ```bash
    docker exec -it my_ubuntu_gcc_container /bin/bash
    ```

3.  **开始工作**：执行上述命令后，你就会进入容器的命令行环境，可以在这个 Ubuntu 18.04 的环境中使用 GCC 7.5 和 Python 进行开发了。

### ⚠️ 如果容器未运行

如果你之前关闭了容器或者系统重启了，你需要先启动它：

1.  **查看所有容器（包括已停止的）**：
    ```bash
    docker ps -a
    ```
    找到你的那个容器，记下它的 **NAME** 或 **CONTAINER ID**。

2.  **启动容器**：
    ```bash
    docker start <容器名称或ID>
    ```
    例如：
    ```bash
    docker start my_ubuntu_gcc_container
    ```

3.  **进入已启动的容器**：容器启动后，再使用 `docker exec -it ...` 命令进入即可。

### 🧭 其他实用技巧

*   **以特定用户身份进入容器**：如果你在 Dockerfile 中创建了特定用户，并希望以其身份进入容器，可以在 `exec` 命令中加上 `-u` 参数：
    ```bash
    docker exec -it -u <用户名> <容器名称或ID> /bin/bash
    ```

*   **退出容器而不停止它**：当你完成工作，需要退出容器时，有两种方式：
    *   输入 `exit`：这会退出容器，并且**通常会导致容器停止运行**，因为 Bash 是容器的主进程。
    *   按 **Ctrl + P + Q**：这个组合键可以**退出容器但让容器继续保持后台运行**，特别有用。

### 💡 总结与提醒

| 操作场景 | 命令 | 说明 |
| :--- | :--- | :--- |
| **查看运行中容器** | `docker ps` | 获取容器的名称或 ID。 |
| **进入运行中容器** | `docker exec -it <名称/ID> /bin/bash` | 核心命令，交互式进入容器。 |
| **查看所有容器** | `docker ps -a` | 包括已停止的容器。 |
| **启动已停止容器** | `docker start <名称/ID>` | 先启动，才能进入。 |
| **退出并停止容器** | 在容器内输入 `exit` | |
| **退出但保持容器运行** | 在容器内按 **Ctrl + P + Q** | 非常实用。 |

**最重要的提醒**：确保你每次进入的都是**正确的容器**。特别是在同时运行多个容器时，仔细核对 `docker ps` 输出的容器名称或 ID。

## 转发图形软件到主机

### 步骤1：在宿主机上安装X11服务器

确保Arch Linux主机已安装X11服务器：
```bash
sudo pacman -S xorg-xhost
```

### 步骤2：修改Docker容器运行命令

使用以下命令运行您的Ubuntu 18.04容器，添加X11转发所需的参数：

```bash
docker run -it \
  --name qt-dev-container \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v /dev/dri:/dev/dri \
  -v /path/to/your/project:/home/project \
  --device /dev/snd \
  --privileged \
  your-ubuntu-qt-image
```

参数说明：
- `-e DISPLAY=$DISPLAY`：传递显示环境变量
- `-v /tmp/.X11-unix:/tmp/.X11-unix`：共享X11套接字
- `-v /dev/dri:/dev/dri`：允许使用硬件加速（可选）
- `-v /path/to/your/project:/home/project`：映射项目目录
- `--device /dev/snd`：允许音频（可选）

### 步骤3：在容器内配置Qt Creator

进入容器并配置Qt Creator：

```bash
# 进入容器
docker exec -it qt-dev-container bash

# 启动Qt Creator（在容器内）
qtcreator
```

## 步骤4：创建便捷启动脚本

在宿主机上创建一个启动脚本`start-qtcreator.sh`：

```bash
#!/bin/bash
xhost +local:docker
docker start qt-dev-container 2>/dev/null || docker run -it \
  --name qt-dev-container \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v /dev/dri:/dev/dri \
  -v $HOME/qt-projects:/home/project \
  --device /dev/snd \
  your-ubuntu-qt-image \
  qtcreator
```

给脚本添加执行权限：
```bash
chmod +x start-qtcreator.sh
```

## 使用已存在的Docker容器运行Qt Creator

是的，您完全可以使用已经运行过的容器来运行Qt Creator。以下是具体操作步骤：

### 步骤1：确保容器已安装Qt Creator

首先，进入您已存在的容器并安装Qt Creator（如果尚未安装）：

```bash
# 进入已存在的容器
docker exec -it your-container-name bash

# 在容器内安装Qt Creator和相关依赖
apt-get update
apt-get install qtcreator qt5-default -y
```

### 步骤2：配置X11转发

对于已存在的容器，您需要确保X11转发已正确配置：

1. 在宿主机上允许Docker连接X11服务器：
   ```bash
   xhost +local:docker
   ```

2. 检查容器是否已有X11相关配置：
   ```bash
   docker inspect your-container-name | grep -i x11
   ```

3. 如果容器没有X11配置，您需要重新创建容器（保留数据）：
   ```bash
   # 首先提交当前容器为镜像
   docker commit your-container-name my-qt-image

   # 然后停止并删除旧容器
   docker stop your-container-name
   docker rm your-container-name

   # 使用新镜像创建容器，并添加X11配置
   docker run -it \
     --name your-container-name \
     -e DISPLAY=$DISPLAY \
     -v /tmp/.X11-unix:/tmp/.X11-unix \
     -v /dev/dri:/dev/dri \
     -v /path/to/your/project:/home/project \
     my-qt-image
   ```

### 步骤3：在现有容器中启动Qt Creator

如果您的容器已经运行且配置了X11转发，可以直接执行：

```bash
# 在运行的容器中启动Qt Creator
docker exec -it your-container-name qtcreator
```

### 步骤4：创建便捷启动脚本

为了让使用更方便，创建一个启动脚本：

```bash
#!/bin/bash
# 启动Qt Creator容器脚本
CONTAINER_NAME="your-container-name"

# 检查容器是否正在运行
if [ ! "$(docker ps -q -f name=$CONTAINER_NAME)" ]; then
    # 容器未运行，启动它
    if [ "$(docker ps -aq -f status=exited -f name=$CONTAINER_NAME)" ]; then
        # 容器存在但已停止，启动它
        docker start $CONTAINER_NAME
    else
        # 容器不存在，需要重新创建（使用之前保存的镜像）
        docker run -it -d \
          --name $CONTAINER_NAME \
          -e DISPLAY=$DISPLAY \
          -v /tmp/.X11-unix:/tmp/.X11-unix \
          -v /dev/dri:/dev/dri \
          -v /path/to/your/project:/home/project \
          my-qt-image \
          /bin/bash
    fi
    # 等待容器完全启动
    sleep 2
fi

# 允许X11连接
xhost +local:docker

# 在容器中启动Qt Creator
docker exec -it $CONTAINER_NAME qtcreator
```

### 步骤5：处理可能的权限问题

如果遇到权限问题，可以尝试以下解决方案：

1. 在容器内创建一个与宿主机用户相同UID的用户：
   ```bash
   # 在容器内执行
   useradd -u $(id -u) -m dockeruser
   ```

2. 使用特定用户运行Qt Creator：
   ```bash
   docker exec -it --user $(id -u) your-container-name qtcreator
   ```

### 注意事项

1. 确保宿主机和容器内的Qt版本兼容
2. 如果使用硬件加速，确保正确挂载了GPU设备（如NVIDIA GPU需要额外配置）
3. 考虑使用Docker volume来持久化Qt Creator的配置
4. 如果遇到显示问题，尝试使用软件渲染：
   ```bash
   docker exec -it your-container-name qtcreator -software-gl
   ```

通过以上步骤，您可以在已存在的Docker容器中使用Qt Creator进行开发，同时保持原有的编译环境和项目文件。
