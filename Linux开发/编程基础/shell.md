# 😋常用shell程序

* /bin/bash ： 一般linux发行版默认使用的shell程序

* /bin/sh ：这是最古老的、经典的Unix Shell，它是许多现代Shell的“祖先”

* /bin/csh ： 使用C语言风格的语法，曾是交互式使用的热门选择，但现在已基本被更现代的Shell（如 Zsh, Fish）取代，不推荐使用

* /bin/ksh : 在Bash流行之前，它是BourneShell的一个重要增强版，对后来的Bash和Zsh都有很大影响。现在仍有一些老牌Unix系统，如AIX在使用

* /bin/zsh : Zsh被看作是Bash的一个强大的“现代化”替代品，它吸收了Bash,Ksh,Tcsh等其他Shell的优点，功能极其强大

* /bin/fish ： Fish的设计宗旨是“开箱即用”，提供一流的交互体验，无需复杂的配置

核心区别总结表

| 特性 | **Bash (Bourne-Again)** | **Zsh (Z Shell)** | **Fish (Friendly)** | **Sh (Bourne)** |
| :--- | :--- | :--- | :--- | :--- |
| **交互体验** | 良好 | **优秀**（需配置） | **极佳**（开箱即用） | 很差 |
| **脚本兼容性** | **极高**（Linux 标准） | 高（兼容大部分 Bash） | 低（自有语法） | **最高**（POSIX 标准） |
| **可配置性** | 高（需手动配置） | **极高**（Oh My Zsh 等） | 中（开箱即用） | 很低 |
| **自动补全** | 基础 | **强大**（可扩展） | **强大**（自带） | 无 |
| **性能速度** | 良好 | 良好 | 良好 | **极快**（轻量） |
| **学习曲线** | 平缓 | 中等 | **平缓**（对新手友好） | 陡峭（已过时） |
| **默认系统** | 多数 Linux | macOS (since Catalina) | - | 脚本标准 |


查看当前系统有哪些shell程序

```bash
$ cat /etc/shells
# Pathnames of valid login shells.
# See shells(5) for details.

/bin/sh
/bin/bash
/bin/rbash
/usr/bin/sh
/usr/bin/bash
/usr/bin/rbash
/usr/bin/systemd-home-fallback-shell
/usr/bin/git-shell
/bin/zsh
/usr/bin/zsh
```

查看当前用户使用的shell

```bash
$ cat /etc/passwd | grep $USER
lj:x:1000:1000::/home/lj:/usr/bin/zsh
```

## 🤖bash基本特性

* 快捷键

  `Ctrl + l`

  `Ctrl + c`

  `Ctrl + a`

  `Ctrl + e`

  `Ctrl + .`

* tab键补全

  `<tab>`

* 历史命令

  `history`

* 标准输入与输出重定向

  `>` / `2>` / `>>` / `&>` / `<`

* 管道

  `|`


# 😻命令行

## 常用命令

* ls : 查看文件和目录

* cat : 读取指定文件内容

* grep : 过滤输出的内容

* echo : 输出文本到标准输出

* cd : 切换目录

## 👏权限管理命令

* chmod : 修改文件或目录权限

* chown : 修改文件归属


## 💨执行命令的方式

1. 交互式

执行命令程序

2. 非交互式

执行脚本程序

* 编写第一个脚本程序

```bash
#!/bin/bash

echo "Hello shell."
```

> ⚠️注意：第一行的注释为固定写法，指定shell解释器，其他行以#开头的都是普通注释


3. 执行脚本的方式

* 赋予文件执行权限

```bash
$ chmod +x test.sh
$ ./test.sh
```

* 使用shell程序执行脚本

```bash
$ sh test.sh
```

> 默认开启子进程来执行脚本

```bash
$ source test.sh
```

> 不会启动子进程执行

# 🦷语法规则

## 👁变量

* 定义变量

```bash
$ abc=123
```
> 变量定义规则：1.默认不要修改系统环境变量 2.不能以数字开头 3.不能有特殊符号

* 使用或查看变量

```bash
$ echo $abc
# 将变量用{}包起来就可以拼接其他内容
$ echo ${abc}
```

* 取消变量

```bash
$ unset abc
```
