# 🦄基础使用

<br/>

## 🐥执行目标

```makefile
# 定义一个名为hello的目标，它需要依赖hello.o和main.o
# 目标: 依赖（多个依赖用空格分开）
hello: hello.o main.o    # 同时它具备检测是否有更新的能力（通过比较时间戳）
	gcc hello.o main.o -o hello

hello.o:
	gcc -c hello.c

main.o:
	gcc -c main.c

```

执行`make`命令得到hello可执行程序

***

## 🐟伪目标

* `.PHONY` ： 用来指定没有依赖的目标。不判断当前目录是否存在指定的依赖

有时候想不管依赖是否有更新，每次都执行目标的方法，可以把它放到`.PHONY`后面

```makefile
# 构建时使用'make clean'命令参数来运行该伪目标
# 当前目录中不能存在名称为'clean'的文件，否则clean被当成该文件来运行；
# 此时需要用到'.PHONY'来告诉make外部文件是假的
.PHONY: clean
clean:                   # 伪目标：make自带的语法标签，常用来清理临时文件或可执行文件；还有一个常用伪目标all
	rm -f *.o hello
```

使用`make clean`命令来运行这个伪目标

***

* `all` : 一般用来编译生成整个项目的完整程序

```makefile
# all伪目标依赖hello 和 world 目标, 在执行该伪目标后这里还有一条输出语句
# 在命令前加一个@符号，可以不输出这条命令本身
# 这里all伪目标依赖了两个目标hello和world
all: hello world
	@echo "build hello word."
```

使用`make all`来运行这个伪目标，会自动运行依赖的所有指令

***

## 🕰自定义最终目标

```makefile
.DEFAULT_GOAL = obj

all: test.elf
	@echo "build all"

obj: obj.elf
	gcc -Wall -c obj.c
```

> 在通常情况下执行`make`命令会默认去make第一个目标，当不想默认执行第一个目标时，可以使用这个标签指定

***


## 🌀依赖类型

* 普通依赖 : 前面写的目标的依赖都是普通依赖

```makefile
main.o: main.c
	gcc -Wall -g -c main.c
```
* 有序依赖 ： order-only，依赖被修改也不重新生成目标，`|`符号右边的依赖

```makefile
# hello.c被修改后，main.o也不会重新生成
main.o: main.c | hello.c
	gcc -Wall -g -c main.c
```

***

## 📻终端相关

* `.ONESHELL` : 如果使用了这个指令，那么每个目标里的方法都在一个shell进程中执行，可以共享环境变量

默认情况下，方法里的每条命令都是单独的子进程执行

```makefile
.ONESHELL:

main.o: main.c
	gcc -Wall -g -c main.c
```

* `.SILENT` : 这个特殊目标可以让指定的依赖方法执行时不输出信息

```makefile
.SILENT: main.o

main.o: main.c
	gcc -Wall -g -c main.c
```

* `-` ： 忽略错误，继续执行其他命令

当方法里的命令执行出错时，会继续执行方法内下一条命令，默认情况下前一条指令执行出错了后面的不会被执行

```makefile
clean:
	-rm main.elf
	-rm main.o
```

***

## 🥤使用变量替换


* `=` ： 常见的变量定义方式，进行字符串替换

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


在使用一下两个变量定义方式之前需要先了解makefile的读取过程，分成两个阶段：

1.读取阶段

2.目标更新阶段

> ⚠️注意：上面一个定义变量的方式是延迟展开，下面的变量定义是立即展开

延迟展开：在make读取到该变量被使用的位置时，它还是变量符号，读取完整个makefile之后执行到这个变量符号时才替换为变量的值

立即展开：在make读取到该变量位置时就将变量符号替换成变量值

* `:=` ： 变量赋值，普通文本替换

* `+=` ： 追加赋值

* `!=` : shell命令结果赋值，将shell命令执行的结果赋值给变量

> ⚠️注意：一些特殊文本在使用`echo`输出到终端时可能会报错，可以使用`$(info 文本或变量)`来输出

* `define` : 多行变量

```makefile
define var_name
@echo hello
@echo make
@echo file
endef

all:
$(info $(var_name))
```

常用来定义多行shell命令变量，也可以组合前面的赋值符号来使用

```makefile
define var_name +=
@echo append
endef

all:
$(info $(var_name))
```

* `undefine` : 可以用来取消之前定义的变量，变成空值

```makefile
objs = main.c

undefine objs

all:
	@echo $(objs)
```

* 引用环境变量 : 可以在makefile中直接使用环境变量的值

```makefile
# 直接使用环境变量名来引用
all:
	@echo $(SHELL)
```

* 变量替换引用 : 将目标变量里指定的字符替换

```makefile
objs != ls *.c
# 使用 : 和 = 实现替换
files = $(objs:%.c=%.o)

all:
	@echo $(files)
```

* 变量覆盖 : 在执行make命令阶段也可以覆盖某个存在的变量值

```bash
$ make objs=test.c
```

> ⚠️注意：=符号两边不能有空格，如果替换成多个文本就使用""符号引起来；如果不想变量被替换，就在变量前加`override`修饰

* 限定变量范围 : 默认定义一个变量是全局的，也可以将限定在某个目标范围内(绑定目标的变量)

```makefile
global_var = this is global var

all: obj.o obj1.o obj2.c
# 限定变量范围
obj.o: var = this is var
obj1.o: var1 = this is var2
# 也可以使用模式匹配目标，根据实际场景进行选择
%.c = var1 = this is .o

obj.o:
	@echo $(global_var)
	@echo $(var)
	@echo $(var1)
obj1.o:
	@echo $(global_var)
	@echo $(var)
	@echo $(var1)
obj2.c:
	@echo $(global_var)
	@echo $(var)
	@echo $(var1)
```

***

## 📦自动变量

* `$@`: 表示目标名

* `$<`: 第一个依赖的名称

* `$^`: 所有依赖文件，不包含order-only依赖和重复的依赖

* `$+` : 类似上一个，但是重复的也会加进来

* `$|` : 所有order-only依赖

* `$?` : 依赖中有改动过的文件

* `$*` : 目标文件名，但是去掉了后缀名

* `$%` : 库文件成员中被更新的依赖文件名，不是库文件中的成员就不会被输出。通常用在打包静态库的目标方法中


```makefile
# 打包库文件的目标
libfun.a(hello.o): hello.o
	@echo $%
	ar rcs libfun.a hello.o
```

```makefile
# 此处用目标的第一个依赖替换变量和目标文件变量符号替换一样的效果
# 如果是库文件目标的方法中使用，输出的就是库文件名
hello.o: hello.c
	gcc -c $< -o $@
```

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

> ⚠️注意：在使用自动变量时碰到依赖是延迟展开的变量时，需要用到`.SECONDEXPANSION:`来二次展开

除了核心变量，还有一些用于处理特定情况的自动变量：

| 变量名             | 含义                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------- |
| **`$(@D)`**        | 目标文件的目录部分（例如，目标为 `build/main.o`，则 `$(@D)` 就是 `build`）。               |
| **`$(@F)`**        | 目标文件的文件名部分（例如，目标为 `build/main.o`，则 `$(@F)` 就是 `main.o`）。            |
| **`$(<D)`**        | 第一个依赖文件的目录部分。                                                                 |
| **`$(<F)`**        | 第一个依赖文件的文件名部分。                                                                 |
| **`$(^D)`**        | 所有依赖文件的目录部分列表。                                                               |
| **`$(^F)`**        | 所有依赖文件的文件名部分列表。                                                               |

***

## 🧬多目标

### 🪒独立多目标

当多个目标的依赖相同并且方法也相同时可以使用独立多目标，这些目标的方法会分别执行一次

```makefile
.PHONY: all main

all main: t1 t2 t3

t1 t2 t3:
	@echo $@ exec...
```

多个目标简写示例

```makefile
objs = main.o hello.o test.o

app.elf: $(objs)
	gcc -o $@ $^

# gcc -c main.c
# gcc -c hello.c
# gcc -c test.c
$(objs):
gcc -c $(@:%.o=%.c)

clean:
	-rm app.elf
	-rm *.o
```

> ⚠️注意：这样写就没办法单独检测依赖更新了，只能每次都全量编译


### 🧼组合多目标

目标只会被执行一次，而不是分别被执行

```makefile
objs = main.o hello.o test.o

app.elf: $(objs)
	gcc -o $@ $^

# 多了一个&符号，并且目标的方法要都写出来
$(objs)&:
	gcc -c main.c
	gcc -c hello.c
	gcc -c test.c

clean:
	-rm app.elf
	-rm *.o
```

通常情况下不会使用到这个

***

## 🔰多规则

1.当一个目标重复定义时，只会运行最后一个目标的方法，但是它们的依赖会被合并后全部执行

2.如果最后一个目标不写方法，那么就会执行前面那个相同目标的方法

```makefile
all: t1 t2
	@echo a1
all: t3 t4
	@echo a2

t1:
	@echo exec t1...
t2:
	@echo exec t1...
t3:
	@echo exec t1...
t4:
	@echo exec t1...
```

> 通常用来追加单独的依赖，比如只有一个单独的`.h`头文件需要被检测依赖更新时

## 📝静态模式

通常用来组合独立多目标情况下的依赖更新检测问题

```makefile
objs = main.o hello.o test.o

app.elf: $(objs)
	gcc -o $@ $^

# 将目标依赖单独拆分出来，以达到依赖更新检测的目的，避免全量编译
$(objs): %.o : %.c %.h
	gcc -c $<

# 这里再例举有一个单独的头文件需要检测依赖
# 使用多规则追加一个依赖，但是会在每个目标中都如这个依赖，还是存在一些问题
$(objs): font.h

clean:
	-rm app.elf
	-rm *.o
```

## 指定依赖搜索路径

* `VPATH` : 指定依赖路径，可以是相对路径也可以是绝对路径。Makefile默认从所在目录找依赖文件

```makefile
# 同时指定多个目录，用:分隔，
VPATH = src:include

# 命令中也要指定
# gcc在编译时也是从自身所在目录找依赖文件，所以也需要手动包含头文件目录
main.o: main.c
	gcc -Wall -g -Iinlude -c $<
```

* `vpath` : 小写的vpath更灵活，可以指定某个类型的文件去哪里搜索

```makefile
vpath %.h include
vpath %.c src

main.o: main.c
	gcc -Wall -g -Iinclude -c $<
```

也可以单独指定某个文件，或所以文件

```makefile
vpath test.h include
vpath % src:include
```


***

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
