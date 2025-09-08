#### 1、安装

apt源安装

```sh
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list && sudo apt update && sudo apt install ngrok
```

<br/>

#### 2、添加认证key

```sh
ngrok config add-authtoken 2aOX79DGhkCWw0guDLeWpghZfX5_7kphJzxizuodTHC9iYFvd
```

<br/>

#### 3、使用端口监听

```sh
ngrok tcp 8080
```
