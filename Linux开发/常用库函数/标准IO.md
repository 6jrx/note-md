# 一、文件IO

系统标准函数库：glibc.a

> man 3 printf

## print函数

常用于打印字符到标准输出设备，如控制台串口tty1


### 缓冲buffer

####  ① 行缓冲

标准输出的内容会暂存在buffer中，不会立即刷新到设备，需要满足特定条件，大小为1k或8k，常见场景如终端输出

- 缓冲区满刷新

  系统默认buffer满了就会刷新输出到设备
  
- 遇到换行刷新
  
  `\n`

- 强制刷新缓冲区

  `fflush(stdout);`

- 程序正常退出

#### ② 无缓冲

标准错误输出，直接输出，无buffer

- perror

  `perror("puts error msg.");`

#### ③ 全缓冲

常见场景如重定向到文件。缓冲区满或调用fflush()方法刷新，大小通常为4k或8k


## 文件操作

FILE 结构体，所有IO操作的具体对象

`FILE *fp`

### 常用标准函数调用

```text
# 非格式化
fopen 
fclose
fputc
fgetc
fputs
fgets
fwirte
fread
fseek
ftell
rewind
```

文件描述符：int fd;

### 系统调用

直接操作文件，无buffer中间层，进行用户态跟内核态切换

```text
open
close
read
write
lseek
```


