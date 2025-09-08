### 一、使用官方压缩包安装

<br/>

1. 将目标文件拷贝到任意目录
   ```sh
   cp jdk-8u333-linux-x64.tar.gz ~
   ```
2. 解压缩目标文件
   ```sh
   tar -zxf jdk-8u333-linux-x64.tar.gz
   ```
3. 将目标文件夹拷贝到`/usr/local/java`
   ```
   cp jdk-8u333 /usr/local/java
   ```
4. 使用`update-alternatives`命令建立连接到命令行目录
   ```sh
   update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_202/bin/java" 1112
   update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_202/bin/javac" 1112
   ```

<br/>

### 二、多个版本切换

<br/>

1. 列出所有版本
   ```sh
   update-alternatives --list java
   ```
2. 选择默认的运行时版本
   ```sh
   # 使用该命令是，终端会提示选择需要作为默认的版本
   update-alternatives --config java
   ```

<br/>

### 三、添加环境变量

<br/>

1. 在当前shell环境配置文件中添加
   
   我这里使用的是`zsh`，所以直接修改`.zshrc`追加如下代码
   ```sh
   export JAVA_HOME=/java/jdk1.8.0_202(改为jdk解压后文件所在目录)
   
   export PATH=$JAVA_HOME/bin:$PATH
   
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   
   export JAVA_HOME PATH CLASSPATH
   
   ```
