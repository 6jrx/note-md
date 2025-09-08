- 安装依赖
  ```sh
  # 先安装部分需要用到的依赖包
  
  apt update
  
  apt -y install apt-transport-https ca-certificates curl software-properties-common
  
  ```
- 添加Docker PGP key
  ```sh
  # 一个链接库用的key
  
  curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-ce-archive-keyring.gpg
  
  ```
- 写入软件源信息
  ```sh
  # docker下载源
  
  printf '%s\n' "deb https://download.docker.com/linux/debian bullseye stable" | sudo tee /etc/apt/sources.list.d/docker-ce.list
  
  ```
- 更新APT
  ```sh
  
  apt update
  ```
- 卸载旧版
  ```sh
  
  apt remove docker docker-engine docker.io
  
  ```
- 安装Docker
  ```sh
  
  apt install  docker-ce docker-ce-cli containerd.io
  ```
- 查看docker状态
  ```sh
  
  systemctl status docker
  ```
- 启动docker
  ```sh
  
  systemctl start docker
  ```
- 开机自动启动
  ```sh
  
  systemctl enable docker
  ```
