# 🦄基础使用

<br/>

## 🐥执行目标

***

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


## 🐟伪目标

***

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


* `all` : 一般用来编译生成整个项目的完整程序

```makefile
# all伪目标依赖hello 和 world 目标, 在执行该伪目标后这里还有一条输出语句
# 在命令前加一个@符号，可以不输出这条命令本身
# 这里all伪目标依赖了两个目标hello和world
all: hello world
	@echo "build hello word."
```

使用`make all`来运行这个伪目标，会自动运行依赖的所有指令


## 🕰自定义最终目标

***

```makefile
.DEFAULT_GOAL = obj

all: test.elf
	@echo "build all"

obj: obj.elf
	gcc -Wall -c obj.c
```

> 在通常情况下执行`make`命令会默认去make第一个目标，当不想默认执行第一个目标时，可以使用这个标签指定


## 🌀依赖类型

***

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

## 📻终端相关

***

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

## 🥤使用变量替换

***

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

* `!=` : 将内容当作shell命令执行，将shell命令执行的结果赋值给变量，将每行结果用空格隔开

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

## 📦自动变量

***

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


## 🧬多目标

***

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

## 🔰多规则

***

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

***

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

## 🧚‍♀️指定依赖搜索路径

***

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

## 🧗‍♀️条件判断

***

* `ifdef` : 判断变量是否被定义

```makefile
flag = Linux

ifdef Win
	flag = Windows
else ifdef Mac
	flag = MacOS
endif
```

* `ifndef` : 判断变量未被定义

```makefile
flag = 0

ifndef os
	flag = 1
else
	flag = 0
endif
```

* `ifeq` : 判断两个值是否相等

```makefile
var = abc
var1 = adc

# 注意空格和外括号
ifeq ($(var), $(var1))
	$(info var == var1)
else
	$(info var != var1)
endif
```
或者可以用引号

```makefile
var = abc
var1 = adc

# 单引号或双引号都可以，也可以混用
ifeq "$(var)" "$(var1)"
	$(info var == var1)
else
	$(info var != var1)
endif
```

* `ifneq` : 判断两个值不相等

```makefile
var = abc
var1 = adc

# 单引号或双引号都可以，也可以混用
ifneq "$(var)" "$(var1)"
	$(info var != var1)
else
	$(info var == var1)
endif
```

## 🧑‍🎤字符串处理

***

* `subst` : 该关键字可以用来替换变量里的指定字符串

```makefile
files = main.c src/test.c src/hello.c

# subst有3个参数(被替换的文本、替换为的文本、要替换的目标文本变量)
# 之前那个用=替换的表达式只能替换结尾的内容，这个函数可以替换任意字符
result = $(subst .c,.o,$(files))

all:
	$(info $(result))
```

> ⚠️注意：函数的参数之间不要有空格，其他函数都适用这个规则

* `patsubst` : 支持模式匹配的字符串替换函数

```makefile
files = main.c src/test.c src/hello.c

result = $(patsubst %.c,%.o,$(files))

all:
	$(info $(files))
	$(info $(result))
```

> ⚠️注意：被替换的目标内容每项用空格分隔，没有空格分隔就会被当成一项来进行处理

* `strip` : 去除多余空格，将头部和尾部多余的空格删掉

```makefile
files =   main.c     src/test.c  src/hello.c

result = $(strip $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `findstring` : 查找字符串，返回查找的内容，没查到就返回空。内容不区分大小写

```makefile
files = main.c src/test.c src/hello.c

result = $(findstring src,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `filter` : 筛选出符合模式的内容并返回

```makefile
files = main.c src/test.c src/hello.c include/hello.h

# 也可以同时筛选多个目标，用空格隔开
result = $(filter %.h,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `filter-out` : 与filter相反，筛选出不符合模式的内容

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(filter-out %.h,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `sort` : 按首字母排序，默认去除了重复内容

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(sort $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `word` : 返回指定位置的目标，位置小于1就会报错，大于总长度就会返回空

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(word 2,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `wordlist` : 返回指定位置范围的多个目标，起始大于总长度就会返回空

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(wordlist 1，3,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `words` : 返回目标里的总项数

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(words $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `firstword` : 返回目标里第一项

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(firstword $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `lastword` : 返回目标里最后一项

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(lastword $(files))

all:
	$(info $(files))
	$(info $(result))
```

## 👨‍🏫文件处理函数

***

* `dir` : 返回目标里的每一项的目录

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(dir $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `notdir` : 返回目标里每一项去除目录后的文件名

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(notdir $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `suffix` : 返回目标里每一项的后缀名

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(suffix $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `basename` : 返回目标里每一项去除后缀名之后的内容

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(basename $(files))

all:
	$(info $(files))
	$(info $(result))
```

* `addsuffix` : 给目标添加后缀名

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(addsuffix .elf,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `addprefix` : 给目标添加前缀

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(addprefix ok_,$(files))

all:
	$(info $(files))
	$(info $(result))
```

* `join` : 将两个目标里的每一项一对一连接，如果两个目标内容项数不对等，多出来的原样返回

```makefile
files = main.c src/test.c src/hello.c include/hello.h
files1 = .a .b .o
result = $(join $(files),$(files1))

all:
	$(info $(files))
	$(info $(result))
```

* `wildcard` : 通配符匹配，返回使用通配符匹配到的文件

```makefile
result = $(wildcard *.c) $(wildcard *.h) $(wildcard include/*.h) $(wildcard src/*.c)

all:
	$(info $(result))
```

* `realpath` : 返回目标里每项文件的绝对路径，如果文件不存在就会被剔除

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(realpath $(files))
all:
	$(info $(files))
	$(info $(result))
```

* `abspath` : 与上面的realpath不同的是，就算文件不存在也会返回当前目录

```makefile
files = main.c src/test.c src/hello.c include/hello.h

result = $(abspath $(files))
all:
	$(info $(files))
	$(info $(result))
```

## 🤓条件函数

***

* `if` : 判断目标是否不为空，不为空就返回这个目标，为空返回指定内容

```makefile
files = main.c src/test.c src/hello.c include/hello.h
# 三个参数，效果就是普通的判断语句
result = $(if $(files),$(files),Not exists)

all:
	$(info $(result))
```

* `or` : 返回条件中第一个不为空的部分

```makefile
f1 =
f2 =
f3 = hello.o
# 这里返回了第三个
result = $(or $(f1),$(f2),$(f3))

all:
	$(info $(result))
```

* `and` : 只要有一个为空就返回空，都不为空就返回最后一个

```makefile
f1 = main.o
f2 = test.o
f3 = hello.o
# 这里返回了最后一个
result = $(and $(f1),$(f2),$(f3))

all:
	$(info $(result))
```

* `intcmp` : 比较两个整数的大小，返回对应操作结果（gcc4.4以上版本）

```makefile
# 比较第1个和第2个参数，小于就返回第3个参数，等于就返回第4个，大于就返回第5个
result = $(intcmp 1,2,le,eq,gt)
# 相等就返回1，不相等就返回空
result1 = $(intcmp 1,2)
all:
	$(info $(result))
	$(info $(result1))
```

## 📄文件函数

***

* `file` : 读写文件

```makefile
# 三个参数，第一个是操作符(>覆盖、>>追加、<读取)，第二个是文件名，第三个是写入的文本(如果是读取就不需要这个参数)
# 执行成功返回greater提示
text = This is write text
result = $(file >>,test.txt,$(text))

all:
	$(info $(result))
```

## 🏂循环函数

***

* `foreach` : 对目标里每一项进行处理，并返回处理后的列表

```makefile
files = main.c src/test.c src/hello.c include/hello.h

# 循环处理，将每一项的目录去掉并将.c替换为.o,将处理好的结果放到result
# 三个参数，1 临时变量，2 目标列表，3 处理动作
result = $(foreach each,$(files),$(patsubst %.c,%.o,$(notdir $(each))))

all:
	$(info $(result))
```

## 🏌️‍♂️自定义函数调用

***

* `call` : 可以用来调用自定义函数(复杂表达式)，也可以调用普通函数

```makefile
func = var1 $(1), var2 $(2), var3 $(3)

# 参数不定长，第1个参数固定为自定义函数名(变量名)
result = $(call func,a,b,c)

all:
	$(info $(result))
```

或者组合其他函数使用

```makefile
# 使用realpath获得绝对路径，然后使用dir获得目录(剔除了文件名)
dirof = $(dir $(realpath $(1)))
# 调用dirof，传入文件名
result = $(call dirof,main.c)

all:
	$(info $(result))
```

## 🕺变量相关函数

* `value` : 返回变量的定义，延迟展开变量如果未被展开就返回表达式本身，立即展开变量会直接返回值

```makefile
file = $(var)

result = $(value file)

var = main.c

all:
	$(info $(result))
```

* `origin` : 返回变量的定义来源，如果变量未被定义就返回undefined，make软件自身的变量就返回default，如果是系统环境变量就返回environment，当前makefile中定义的变量就返回file，自动变量就返回automatic

```makefile
file = main.c

result = $(origin SHELL)

all:
	$(info $(result))
```

* `flavor` : 返回变量的赋值方式，变量未定义就返回undefined，如果不是立即展开变量就返回recursive，是立即展开变量就返回simple

```makefile
file = main.c

result = $(flavor file)

all:
	$(info $(result))
```

## 🐡其他函数

* `eval` : 将文本当成makefile语句执行

```makefile
# 定义一个多行变量
define var =
ext_run:
	@echo Target ext test
endef

# 相当于把上面定义的变量内容插入到了这里，使用make命令执行时就会默认执行第一个目标，也就是ext_run目标，而all目标不会被执行到
$(eval $(var))

all:
	$(info eval is run)
```

* `shell` : 执行shell命令，返回执行结果

```makefile
result = $(shell ls)

all:
	$(info $(result))
```

* `let` : 将目标列表内容逐个放入指定变量中，如果目标超过了自定义变量个数那么会将剩余的都放入最后一个变量中

一般配合call函数可以实现一些比较灵活的操作

```makefile
files = main.c src/test.c src/hello.c include/hello.h

# 前n个参数是自定义个数，后两个参数一个是源变量，一个是指定要返回的自定义变量
result = $(let v1 v2 v3,$(files),v3)

all:
	$(info $(result))
```

## 🪴日志信息

* `info` : 输出提示信息

```makefile
all:
	$(info printf info)
``

* `warning` : 输出警告信息，并且附带当前行号

```makefile
all:
	$(warning printf warning)
```

* `error` : 输出错误信息，并附带当前行号和停止make运行

```makefile
all:
	$(error printf error)
```

<br>

# 🦅自动推导与隐式规则

* 自动推导 : 默认情况下只写目标和依赖，gcc会自动使用命令工具生成目标，如果目标是最终程序会自动链接

```makefile
files = main.c src/test.c src/hello.c include/hello.h

main: $(files)

clean:
	-rm main *.o *.i *.s
```

> C代码默认链接时使用的是make变量里CC里的工具(命令工具:cc)，如果是需要用其他工具就需要手动覆盖一下CC变量

* 隐式规则 : 在指定.o目标和依赖时，make默认使用CC进行编译，并且添加了常用的参数

如果想破坏这种默认规则，需要如下操作

```makefile
# 这样写目标和依赖并且没有方法，就会破坏默认将.c编译成.o的默认规则
%.o : %.c

# 写上方法就可以手动指定规则
%.o : %.c
	$(CC) -c $<
```

C编译规则：`$(CC) $(CPPFLAGS) $(CFLAGS) -c`

C++编译规则：`$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c`

链接规则：`$(CC) $(LDFLAGS) *.o $(LOADLIBES) $(LDLIBS)`

## 🦉常用make变量

1.程序名变量 (Program Names)

这些变量定义了在隐式规则中要使用的程序路径和名称。

| 变量名 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `CC` | `cc` | **C 编译器**。这是最常用的变量，通常在你的 Makefile 中会覆盖它，例如 `CC = gcc` 或 `CC = clang`。 |
| `CXX` | `g++` | **C++ 编译器**。用于编译 `.cc`, `.cpp`, `.C` 等 C++ 源文件。 |
| `CPP` | `$(CC) -E` | **C 预处理器**。 |
| `AR` | `ar` | **静态库打包程序**。用于创建 `.a` 归档文件。 |
| `AS` | `as` | **汇编器**。用于编译 `.s` 文件。 |
| `LD` | `ld` | **链接器**。用于直接链接目标文件。 |
| `FC`/`F77` | `f77` | **Fortran 编译器**。 |
| `RM` | `rm -f` | **删除命令**。在清理规则（如 `clean`）中非常有用。 |
| `MAKE` | `make` | make命令，在指定目录执行make, 或者在后面附带其他参数 |

如何使用：

你通常会在 Makefile 的开头覆盖这些变量，以指定你偏好的工具链。
```makefile
CC = gcc
CXX = g++
AR = ar
```

2.标志变量 (Flags)

这些变量定义了传递给相应程序的命令行参数。

| 变量名 | 说明 |
| :--- | :--- |
| `CFLAGS` | **传递给 C 编译器的标志**。例如优化等级 `-O2`、调试信息 `-g`、警告选项 `-Wall`。 |
| `CXXFLAGS` | **传递给 C++ 编译器的标志**。用法同 `CFLAGS`。 |
| `CPPFLAGS` | **传递给 C/C++ 预处理器的标志**。通常用于定义宏（`-DNAME`）和包含路径（`-Iinclude_dir`）。它同时用于编译（`*.c` -> `*.o`）和链接（`*.o` -> 可执行文件）阶段。 |
| `LDFLAGS` | **传递给链接器的标志**（当编译器充当链接器驱动时）。例如 `-L` 指定库搜索路径。 |
| `LDLIBS` | **传递给链接器的库标志**。通常放在命令末尾，指定要链接的库，例如 `-lm`（数学库）、`-lpthread`（线程库）。 |
| `ARFLAGS` | **传递给 `ar` 程序的标志**。默认值是 `rv`（r: 替换成员，v: 显示详细操作）。 |

如何使用：

这些变量是你定制构建过程的核心。你可以根据需要添加各种编译和链接选项。
```makefile
CFLAGS = -O2 -g -Wall -pedantic
CPPFLAGS = -I./include -DDEBUG
LDFLAGS = -L./lib
LDLIBS = -lmylib -lm
```

3.自动变量 (Automatic Variables)

这些变量在规则的命令部分被自动定义，它们的值取决于规则的目标和依赖关系。它们不是“隐式规则变量”，但编写任何规则（无论是显式还是隐式）都极其重要。

| 变量名 | 说明 |
| :--- | :--- |
| `$@` | **当前规则的目标文件名**。 |
| `$<` | **第一个依赖文件的文件名**。 |
| `$^` | **所有依赖文件的列表**，以空格分隔。 |
| `$?` | **所有比目标更新的依赖文件列表**。 |
| `$*` | **匹配隐式规则后缀模式的主干（stem）**。例如，如果目标是 `file.o`，依赖是 `file.c`，则 `$*` 的值就是 `file`。 |

示例：

假设你有一条规则：`app: main.o utils.o`
```makefile
app: main.o utils.o
    $(CC) -o $@ $^ $(LDLIBS) # 等价于：gcc -o app main.o utils.o -lm
```
在这个命令中：
`$@` 被展开为 `app`
`$^` 被展开为 `main.o utils.o`

4.参数控制变量 (Parameter Control)

这些变量可以修改命令本身的行为。

| 变量名 | 说明 |
| :--- | :--- |
| `MAKEFLAGS` | 传递给子 make 进程的标志。 |
| `SHELL` | 指定 Make 执行命令时使用的 shell。默认是 `/bin/sh`。**注意：** 这个变量不会从环境中继承，你必须在 Makefile 中显式设置它。 |
| `MAKECMDGOALS` | 用户在执行 make 时指定的目标列表。例如，`make clean all`，则 `$(MAKECMDGOALS)` 包含 `clean all`。可用于条件判断。 |

### 一个综合示例

让我们看一个完整的 Makefile，它利用了这些变量：

```makefile
# 覆盖程序名和标志变量
CC = gcc
CFLAGS = -O2 -g -Wall
CPPFLAGS = -I./include
LDFLAGS = -L./lib
LDLIBS = -lm

# 最终目标
myapp: main.o utils.o helper.o
    $(CC) $(LDFLAGS) -o $@ $^ $(LDLIBS)

# 为 helper.o 指定额外的编译标志
helper.o: helper.c
    $(CC) $(CFLAGS) $(CPPFLAGS) -DSPECIAL_OPTION -c -o $@ $<

# 告诉 make ‘clean’ 不是一个文件，而是一个动作
.PHONY: clean

# 使用 RM 变量进行清理
clean:
    $(RM) myapp *.o
```

**在这个例子中：**

1.  `myapp` 的链接规则使用了自动变量 `$@` (目标名) 和 `$^` (所有依赖)。
2.  隐式规则被用于构建 `main.o` 和 `utils.o`。Make 会自动调用：
    ```bash
    gcc -O2 -g -Wall -I./include -c -o main.o main.c
    gcc -O2 -g -Wall -I./include -c -o utils.o utils.c
    ```
3.  我们为 `helper.o` 写了一个**显式规则**，因为它需要一个特殊的编译选项 `-DSPECIAL_OPTION`。这个规则覆盖了隐式规则。
4.  `clean` 目标使用了 `$(RM)` 变量，它默认展开为 `rm -f`。


<br>

# 🦦多Makefile

* 包含 : 包含其他makefile文件到当前文件中

```makefile
objs := main.o test.o hello.o hello.h

# 直接使用include关键字，会将后面指定的文件内容展开到当前位置
# 如果里面定义了一个目标，执行make就会默认执行这个展开的目标，因为这里展开就成了第一个目标
# 这里也可以指定多个makefile文件，如果文件不存在就会报错，如果要忽略错误就在include前面加个-符号
# 还可以使用通配符来匹配其他makefile文件，makefile*
include submk

main: $(objs)
	$(CXX) -o $@ $^
clean:
	-$(RM) main *.o
```

* 嵌套 : 通常在一些大型项目中会将业务拆分成多模块，每个模块单独有一个makefile文件，在项目根目录定义一个总makefile文件嵌套这些子模块

文件结构
```bash
$ tree --dirsfirst
.
├── bin
├── include
│   ├── font.h
│   ├── hello.h
│   └── main.h
├── lib
│   ├── hello.c
│   ├── font.c
│   └── Makefile
├── src
│   ├── main.c
│   └── Makefile
└── Makefile
```

lib/Makefile
```makefile
cfile := $(wildcard *.c)

objs := $(cfile:%.c=%.o)

CFLAGS += -I../include -Wall -O2 -g

libmain.a: $(objs)
	$(AR) rcs $@ $^
```

src/Makefile
```makefile
vpath %.a ../lib

CFLAGS += -I../include

LDFLAGS += -L../lib

LDLIBS += -lmain

../bin/main: main.o libmain.a
	$(CC) -o $@ $< $(LDFLAGS) $(LDLIBS)

.PHONY: clean
clean:
	$(RM) *.o ../bin/*
```

./Makefile
```makefile
.PHONY: subsrc sublib

# 方法：进入src目录执行make命令
subsrc: sublib
	$(MAKE) -C src

# 同上
sublib:
	$(MAKE) -C lib

# 进入指定目录执行make clean
clean:
	$(MAKE) clean -C src
	$(MAKE) clean -C lib
```

* 传递变量 : 向子模块Makefile中传递指定变量

```makefile
# 传递var
export var
# 传递所有变量
export
# 取消传递
unexport
```

<br>

# 🦈GCC常见选项

***

* `-f` : 当makefile文件内容中找不到指定的文件时，可以使用`-f`来指定依赖的文件

比如文件中写的是`hello.c`，但是真实文件的名称是`hello.123`

执行命令: `make -f hello.123`


* 显示要执行的命令

在执行`make`命令是想看看`makefile`中需要执行哪些指令

可以使用`make -n `选项来查看指令，并不会真的执行里面的指令，常用于调试


* 指定目录

当项目中包含有子模块的makefile时，可以使用`make -C`来指定子目录

在makefile汇总根目录执行

`make -C dir01`

***

> ⭐提示：后续进阶可以学Cmake或者Meson，自动化操作跟高

内容来源: Bilibili(@无限十三年)
