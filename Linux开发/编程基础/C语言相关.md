### main方法的参数

```c_cpp
int main(int argc, char *args[])
{
  
  
}
```

- C99标准之前通常无参的写法

```c_cpp
int main()
{
  
}
```

- C99之后必须告诉编译器无参就是不接收参数

```c_cpp
int main(void)
{
  
}
```

- 有参数的写法

```c_cpp
int main(int argc, char *args[])
{
  // argc：表示传入参数的个数
  // args：数组里是具体参数
  //      args[0]: 一般是当前程序名称
  //      args[1]: 是程序执行时跟在后面的参数字符
}
```
