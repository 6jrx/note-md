SQLite的“命令”分为两类，理解这一点很重要：
*   **点命令 (Dot-Commands)**：以点 `.` 开头，是 `sqlite3` 命令行工具特有的管理指令，**不是SQL**。例如管理数据库文件、格式化输出等。
*   **SQL语句**：标准的数据库操作语言，用于 `增(Create)、删(Delete)、改(Update)、查(Read)` 数据，**所有SQL语句都必须以分号 `;` 结尾**。

下面我将这两类命令结合数据库操作流程进行说明。

### **第一部分：启动与退出**
这是操作的前提。
*   **启动并打开数据库文件**：`sqlite3 数据库文件名.db`
    *   如果文件存在则打开，不存在则创建。
    *   示例：`sqlite3 mytest.db`
*   **退出命令行**：
    *   `.quit` 或 `.exit`

### **第二部分：点命令（常用管理命令）**
在 `sqlite3` 命令行环境中使用。

| 命令                  | 用途说明                                                     |
| :-------------------- | :----------------------------------------------------------- |
| `.help`               | 查看所有点命令的帮助信息。                                   |
| `.databases`          | 显示当前打开的数据库文件及其路径。                           |
| `.tables`             | 列出当前数据库中的所有表名。可加模式匹配，如 `.tables %user%`。 |
| `.schema [表名]`      | 查看建表语句。不指定表名则查看所有表的结构。                 |
| `.mode`               | 设置输出格式。常用模式有：`column` (分列显示，默认)、`list` (列表)、`csv`、`table` (表格)。 |
| `.headers on/off`     | 查询时，打开或关闭列名标题显示。                             |
| `.output [文件名]`    | 将后续所有命令的输出重定向到指定文件，而非屏幕。用 `.output stdout` 改回屏幕。 |
| `.import [文件] [表]` | 将外部文件（如CSV）的数据导入到指定表中。                    |
| `.dump`               | 以SQL文本格式导出整个数据库。常用于备份。                    |

### **第三部分：SQL语句（核心：增删改查）**
这些是标准的SQL，需要在 `sqlite3` 命令行中输入，**并以分号 `;` 结束**。

**1. 创建表 (Create)**
```sql
CREATE TABLE 表名 (
    列1名 数据类型,
    列2名 数据类型,
    ...
);
-- 示例：创建一个用户表
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    age INTEGER
);
```

**2. 插入数据 (Insert)**
```sql
INSERT INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...);
-- 示例
INSERT INTO users (name, age) VALUES ('张三', 25);
INSERT INTO users (name, age) VALUES ('李四', 30);
```

**3. 查询数据 (Select)**
```sql
SELECT 列名 FROM 表名 WHERE 条件;
-- 示例
SELECT * FROM users; -- 查询所有列的所有行
SELECT name, age FROM users WHERE age > 25; -- 条件查询
SELECT * FROM users ORDER BY age DESC; -- 按年龄降序排序
```

**4. 更新数据 (Update)**
```sql
UPDATE 表名 SET 列1 = 新值1, 列2 = 新值2 WHERE 条件;
-- 示例：将张三的年龄改为26
UPDATE users SET age = 26 WHERE name = '张三';
-- **警告：如果不加WHERE条件，会更新表中所有行！**
```

**5. 删除数据 (Delete)**
```sql
DELETE FROM 表名 WHERE 条件;
-- 示例：删除年龄大于30的用户
DELETE FROM users WHERE age > 30;
-- **警告：如果不加WHERE条件，会删除表中所有数据！**
```

**6. 删除表 (Drop)**
```sql
DROP TABLE 表名;
```

### **第四部分：实战流程示例**
以下命令模拟一个完整操作流程：
```bash
# 1. 启动并创建数据库
sqlite3 company.db

# 2. 创建表
sqlite> CREATE TABLE employee (id INTEGER PRIMARY KEY, name TEXT, dept TEXT, salary REAL);

# 3. 插入几条数据
sqlite> INSERT INTO employee (name, dept, salary) VALUES ('小王', '技术部', 8000);
sqlite> INSERT INTO employee (name, dept, salary) VALUES ('小李', '市场部', 7500);

# 4. 设置美观的查询格式并查看
sqlite> .mode table
sqlite> .headers on
sqlite> SELECT * FROM employee;

# 5. 给技术部全员加薪
sqlite> UPDATE employee SET salary = salary * 1.1 WHERE dept = '技术部';

# 6. 再次查看结果
sqlite> SELECT * FROM employee;

# 7. 退出
sqlite> .quit
```

### **重要补充：事务**
对于连续的写操作（INSERT/UPDATE/DELETE），使用事务可以大幅提升效率并保证原子性。
```sql
BEGIN TRANSACTION; -- 开始事务
-- 执行多条修改语句...
INSERT INTO ...;
UPDATE ...;
COMMIT; -- 提交事务（如果出错可以用 ROLLBACK; 回滚）
```

### **总结与建议**
- **记住分号**：SQL语句必须有 `;`，点命令不能有 `;`。
- **善用 `.help`**：随时在命令行输入 `.help` 查看完整命令列表。
- **先备份**：在对重要数据库进行批量修改前，可以用 `.dump > backup.sql` 命令备份。

根据你的使用场景，如果是在程序（如C、Python）中操作SQLite，那么主要使用的是上述的 **SQL语句部分**。如果想进一步了解特定操作（例如复杂的多表查询、索引创建或数据导入导出），可以随时提出。