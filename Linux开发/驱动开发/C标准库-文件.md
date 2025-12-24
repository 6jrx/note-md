C标准库的文件操作函数主要定义在 `<stdio.h>` 头文件中。以下是分类整理：

## 1. 文件打开和关闭

### `fopen()` - 打开文件
```c
FILE *fopen(const char *filename, const char *mode);
```

### `fclose()` - 关闭文件
```c
int fclose(FILE *stream);
```

**示例：**
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("example.txt", "w");
    if (file == NULL) {
        printf("文件打开失败\n");
        return 1;
    }
    
    // 文件操作...
    
    fclose(file);
    return 0;
}
```

## 2. 字符输入输出

### `fgetc()` - 读取一个字符
```c
int fgetc(FILE *stream);
```

### `fputc()` - 写入一个字符
```c
int fputc(int char, FILE *stream);
```

### `getc()`, `putc()` - 宏版本的字符IO
```c
int getc(FILE *stream);
int putc(int char, FILE *stream);
```

**示例：**
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("test.txt", "w+");
    
    // 写入字符
    fputc('A', file);
    fputc('B', file);
    
    // 回到文件开头
    rewind(file);
    
    // 读取字符
    int ch;
    while ((ch = fgetc(file)) != EOF) {
        printf("%c", ch);
    }
    
    fclose(file);
    return 0;
}
```

## 3. 字符串输入输出

### `fgets()` - 读取字符串
```c
char *fgets(char *str, int n, FILE *stream);
```

### `fputs()` - 写入字符串
```c
int fputs(const char *str, FILE *stream);
```

**示例：**
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("data.txt", "w+");
    char buffer[100];
    
    // 写入字符串
    fputs("Hello World\n", file);
    fputs("C Programming\n", file);
    
    rewind(file);
    
    // 读取字符串
    while (fgets(buffer, sizeof(buffer), file) != NULL) {
        printf("%s", buffer);
    }
    
    fclose(file);
    return 0;
}
```

## 4. 格式化输入输出

### `fprintf()` - 格式化输出到文件
```c
int fprintf(FILE *stream, const char *format, ...);
```

### `fscanf()` - 从文件格式化读取
```c
int fscanf(FILE *stream, const char *format, ...);
```

**示例：**
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("student.txt", "w+");
    char name[50];
    int age;
    float score;
    
    // 写入格式化数据
    fprintf(file, "%s %d %.2f\n", "张三", 20, 85.5);
    fprintf(file, "%s %d %.2f\n", "李四", 22, 92.0);
    
    rewind(file);
    
    // 读取格式化数据
    while (fscanf(file, "%s %d %f", name, &age, &score) != EOF) {
        printf("姓名: %s, 年龄: %d, 分数: %.2f\n", name, age, score);
    }
    
    fclose(file);
    return 0;
}
```

## 5. 二进制文件操作

### `fread()` - 二进制读取
```c
size_t fread(void *ptr, size_t size, size_t count, FILE *stream);
```

### `fwrite()` - 二进制写入
```c
size_t fwrite(const void *ptr, size_t size, size_t count, FILE *stream);
```

**示例：**

```c
#include <stdio.h>

struct Student {
    char name[50];
    int age;
    float score;
};

int main() {
    FILE *file = fopen("students.dat", "wb+");
    struct Student students[3] = {
        {"Alice", 20, 88.5},
        {"Bob", 21, 92.0},
        {"Charlie", 19, 76.5}
    };
    struct Student read_students[3];
    
    // 写入二进制数据
    fwrite(students, sizeof(struct Student), 3, file);
    
    rewind(file);
    
    // 读取二进制数据
    fread(read_students, sizeof(struct Student), 3, file);
    
    for (int i = 0; i < 3; i++) {
        printf("学生%d: %s, %d, %.1f\n", 
               i+1, read_students[i].name, 
               read_students[i].age, read_students[i].score);
    }
    
    fclose(file);
    return 0;
}
```

## 6. 文件定位

### `fseek()` - 设置文件位置
```c
int fseek(FILE *stream, long offset, int whence);
```

### `ftell()` - 获取当前位置
```c
long ftell(FILE *stream);
```

### `rewind()` - 回到文件开头
```c
void rewind(FILE *stream);
```

**示例：**

```c
#include <stdio.h>

int main() {
    FILE *file = fopen("largefile.txt", "rb");
    
    // 移动到文件末尾
    fseek(file, 0, SEEK_END);
    
    // 获取文件大小
    long size = ftell(file);
    printf("文件大小: %ld 字节\n", size);
    
    // 回到文件开头
    rewind(file);
    
    // 移动到第100字节处
    fseek(file, 100, SEEK_SET);
    
    fclose(file);
    return 0;
}
```

## 7. 错误处理

### `feof()` - 检测文件结束
```c
int feof(FILE *stream);
```

### `ferror()` - 检测文件错误
```c
int ferror(FILE *stream);
```

### `clearerr()` - 清除错误标志
```c
void clearerr(FILE *stream);
```

**示例：**
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("test.txt", "r");
    
    if (file == NULL) {
        perror("文件打开失败");
        return 1;
    }
    
    int ch;
    while ((ch = fgetc(file)) != EOF) {
        putchar(ch);
    }
    
    if (feof(file)) {
        printf("\n已到达文件末尾\n");
    }
    
    if (ferror(file)) {
        printf("读取文件时发生错误\n");
        clearerr(file);
    }
    
    fclose(file);
    return 0;
}
```

## 8. 其他实用函数

### `remove()` - 删除文件
```c
int remove(const char *filename);
```

### `rename()` - 重命名文件
```c
int rename(const char *old_filename, const char *new_filename);
```

### `tmpfile()` - 创建临时文件
```c
FILE *tmpfile(void);
```

**示例：**
```c
#include <stdio.h>

int main() {
    // 创建临时文件
    FILE *temp = tmpfile();
    if (temp == NULL) {
        printf("临时文件创建失败\n");
        return 1;
    }
    
    fputs("这是临时文件内容\n", temp);
    rewind(temp);
    
    char buffer[100];
    while (fgets(buffer, sizeof(buffer), temp)) {
        printf("%s", buffer);
    }
    
    // 临时文件会自动关闭和删除
    
    // 重命名文件示例
    if (rename("old.txt", "new.txt") != 0) {
        perror("重命名失败");
    }
    
    // 删除文件示例
    if (remove("delete.txt") != 0) {
        perror("删除失败");
    }
    
    return 0;
}
```

这些函数提供了完整的文件操作能力，涵盖了从简单的文本文件到复杂的二进制文件的各种操作需求。