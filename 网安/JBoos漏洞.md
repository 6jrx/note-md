#### 一、简介

JBoss是一套开源的企业级Java中间件系统，用于实现基于SOA的企业应用和服务，基于J2EE的开放源代码的应用服务器。JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。JBoss是一个管理EJB的容器和服务器，支持EJB 1.1、EJB 2.0和EJB3的规范。但JBoss核心服务不包括支持servlet/JSP的WEB容器，一般与Tomcat或Jetty绑定使用。

<br/>

#### 二、历史漏洞

- 访问控制不严导致的漏洞
  - JMX Console未授权访问Getshell
  - Administration Console 弱口令 Getshell
  - CVE-2007-1036 -- JMX Console HtmlAdaptor Getshell
  - CVE-2010-0738 -- JMX控制台安全验证绕过漏洞
- 反序列化漏洞
  - CVE-2013-4810 -- JBoss EJBInvokerServlet 反序列化漏洞
  - CVE-2015-7501 -- JBoss JMXInvokerServlet 反序列化漏洞
  - CVE-2017-7504 -- JBoss 4.x JBossMQ JMS 反序列化漏洞
  - CVE-2017-12149 -- JBosS AS 6.X 反序列化漏洞

<br/>

#### 三、历史漏洞发现

漏洞检测工具：

`https://github.com/GGyao/jbossScan`

`https://github.com/joaomatosf/jexboss`

<br/>

#### 四、历史漏洞利用

`https://github.com/vulhub/vulhub/tree/master/jboss/CVE-2017-12149`
