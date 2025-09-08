### 0x01. 基础使用

<br/>

#### 执行目标

```makefile
# 定义一个名为hello的目标，它需要依赖hello.o和main.o
hello: hello.o main.o    # 同时它具备检测是否有更新的能力
	gcc hello.o main.o -o hello

hello.o:
	gcc -c hello.c

main.o:
	gcc -c main.c

```

执行`make`命令得到hello可执行程序

<br/>

#### 伪目标clean

```makefile
# 构建时使用'make clean'命令参数来运行该伪目标
# 当前目录中不能存在名称为'clean'的文件，否则clean被当成该文件来运行；
# 此时需要用到'.PHONY'来告诉make外部文件是假的
clean:                   # 伪目标：make自带的语法标签，常用来清理临时文件或可执行文件；还有一个常用伪目标all
	rm -f *.o hello
```

使用`make clean`命令来运行这个伪目标

<br/>

#### 伪目标all

```makefile
# all伪目标依赖hello 和 world 目标, 在执行该伪目标后这里还有一条输出语句
# 在命令前加一个@符号，可以不输出这条命令本身
# 这里all伪目标依赖了两个目标hello和world
all: hello world
	@echo "build hello word."
```

使用`make all`来运行这个伪目标，会自动运行依赖的所有指令

<br/>

#### 目标合并

```makefile
# 将两个目标合并到一行，命令中使用占位符号会自动替换成目标名
# 这里会生成两个可执行文件
hello world: hello.o main.o 
	gcc hello.o main.o -o $@
```

<br/>

#### 使用变量替换

```makefile
.PHONY: clean all

# 定义常规变量
OPTIONS = -Wall -g -02
TARGETS = hello world
SOURCES = main.c hello.c
OBJECTS = main.o hello.o

all: $(TARGETS)
	@echo "build hello world."

$(TARGETS): $(OBJECTS)
	gcc $(OPTIONS) $(OBJECTS) -o $@  		

hello.o:
	gcc $(OPTIONS) -c hello.c

main.o:
	gcc $(OPTIONS) -c main.c

clean:
	rm -f *.o hello world

```

还有几种自动变量如下：

`$@`: 表示目标文件

`$<`: 第一个依赖文件

```makefile
hello.o: hello.c
	gcc -c $< -o $@   #此处用目标的第一个依赖替换变量和目标文件变量符号替换一样的效果
```

`$^`: 所有依赖文件

<br/>

- 通配符

```makefile
hello.o: hello.c
	gcc -c $< -o $@

main.o: main.c
	gcc -c $^ -o $@

# 可以使用通配符%来简化目标文件
%.o: %.c
	gcc $(OPTIONS) -c $^ -o $@
# %.c  %.c 通配所有.c和.o文件
```

<br/>

### 0x02. make的常用参数

<br/>

#### 指定文件

当makefile文件内容中找不到指定的文件时，可以使用`-f`来指定依赖的文件

比如文件中写的是`hello.c`，但是真实文件的名称是`hello.123`

执行命令: `make -f hello.123` 

<br/>

#### 显示要执行的命令

在执行`make`命令是想看看`makefile`中需要执行哪些指令

可以使用`make -n `选项来查看指令，并不会真的执行里面的指令，常用于调试

<br/>

#### 指定目录

当项目中包含有子模块的makefile时，可以使用`make -C`来指定子目录

在makefile汇总根目录执行

`make -C dir01`