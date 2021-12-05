# 三剑客（awk、sed、grep）

## awk

​	awk是一个强大的文本分析工具，简单来说awk就是把文件逐行读入，（空格，制表符）为默认分隔符将每行切片，切开的部分再进行各种分析处理。

```shell
awk [-F field-separator] 'commands' input-file(s)
```

​	[-F 分隔符]是可选的，因为awk使用空格，制表符作为缺省的字段分隔符，因此如果要浏览字段间有空格，制表符的文本，不必指定这个选项，但如果要浏览诸如/etc/passwd文件，此文件各字段以冒号作为分隔符，则必须指明-F选项，例如：

```shell
[root@iZbp1jd5ee7h52j00jed2wZ ~]# echo "this, is, a test"|awk -F , '{print $2}'
 is
[root@iZbp1jd5ee7h52j00jed2wZ ~]# echo "this, is, a test"|awk -F , '{print $3}'
 a test
```

​	shell读取用户输入的字符串发现|，代表有管道。|左右被理解为简单[命令](https://www.linuxcool.com/)，即前一个（左边）简单命令的标准输出指向后一个（右边）标准命令的标准输入
​	awk会根据分隔符将行分成若干个字段，$0为整行，$1为第一个字段，$2 为第2个地段，依此类推…为打印一个字段或所有字段，使用print命令。这是一个`awk`动作：

```shell
echo "this is a test" | awk '{ print $1 }'  
## 输出为  
this  
echo "this is a test" | awk '{ print $1, $2 }'  
## 输出为  
this is
```

### 案例

创建一个passwd文件，用来我们学习awk案例，文件如下：

```shell
root:x:0:0:root:/root:/bin/bash  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* **只显示/etc/passwd的账户：**

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F : ' {print $1}' password 
root
bin
daemon
adm
lp
```

* **显示/etc/passwd的第1列和第7列，用逗号分隔显示，所有行开始前添加列名start1，start7，最后一行添加，end1，end7（学到了BEGIN和END的用法）**
  * 注意点1：$1和$7在" "号外
  * 注意点2：BEGIN和 END都是大写

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' 'BEGIN {print"start1,start7"} \
{print $1","$7} END {print "End1,End7"}' password 
start1,start7
root,/bin/bash  
bin,/sbin/nologin  
daemon,/sbin/nologin  
adm,/sbin/nologin  
lp,/sbin/nologin
End1,End7
```

* **统计/etc/passwd文件中，每行的行号，每行的列数，对应的完整行内容(NR是行号，NF是列数)**

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F : '{print NR " " NF " "$0 }' password 
1 7 root:x:0:0:root:/root:/bin/bash  
2 7 bin:x:1:1:bin:/bin:/sbin/nologin  
3 7 daemon:x:2:2:daemon:/sbin:/sbin/nologin  
4 7 adm:x:3:4:adm:/var/adm:/sbin/nologin  
5 7 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* **将`/etc/passwd`的用户名变成大写输出(toupper)**

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '{print toupper($1)}' password 
ROOT
BIN
DAEMON
ADM
LP
```

* 显示/etc/passwd中有var的行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '$0 ~ /var/' password
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 每一行的长度

```shell
* [root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' ' {print length($0)}' password 
  33
  34
  41
  38
  40
```

可以用的语句：if while do/while for break continue

* 输出第一个字段的第一个字符大于d的行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '{if($1>"d"){print $1}else{print "--"}}' password 
root
--
daemon
--
lp
```

* 把上面的语句写入脚本中，可以用awk使用
  * 脚本语句

```shell
{   
    if ($1 > "d") {  
        print $1   
    } else {  
        print "-"   
    }   
}
```

使用语句：

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' -f password.sh password
root
-
daemon
-
lp
```



### 支持的内置变量

​	上面示例中`NR`，和`NF`其实就是`awk`的内置变量，一些内置变量如下：

```
变量名 解释
FILENAMEawk浏览的文件名
FS设置输入字段分隔符，等价于命令行-F选项
NF 浏览记录的字段个数
NR 已读的记录数
```

### 支持函数

* **输出字符串的长度(length)**

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk 'BEGIN{print length("this is a test ")}'
15
```

* **将`/etc/passwd`的全部语句变成大写输出(toupper)**

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '{print toupper($0)}' password 
ROOT:X:0:0:ROOT:/ROOT:/BIN/BASH  
BIN:X:1:1:BIN:/BIN:/SBIN/NOLOGIN  
DAEMON:X:2:2:DAEMON:/SBIN:/SBIN/NOLOGIN  
ADM:X:3:4:ADM:/VAR/ADM:/SBIN/NOLOGIN  
LP:X:4:7:LP:/VAR/SPOOL/LPD:/SBIN/NOLOGIN
```

常用函数：

```shell
函数名 作用
toupper(s)返回s的大写
tolower(s) 返回s的小写
length(s) 返回s长度
substr(s,p) 返回字符串s中从p开始的后缀部分
```

### 正则表达式

​	常用正则表达式：

```shell
操作符	描述
<  小于 < = 小于等于 == 等于 != 不等于 ~ 匹配正则表达式 !~ 不匹配正则表达式 
```

* 显示/etc/passwd中有var的行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '$0 ~ /var/' password
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

### 流程控制语句

可以用的语句：if while do/while for break continue

* 输出第一个字段的第一个字符大于d的行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '{if($1>"d"){print $1}else{print "--"}}' password 
root
--
daemon
--
lp
```

#### 执行脚本

* 把上面的语句写入脚本中，可以用awk使用
  * 脚本语句

```shell
{   
    if ($1 > "d") {  
        print $1   
    } else {  
        print "-"   
    }   
}
```

使用语句：

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' -f password.sh password
root
-
daemon
-
lp
```

## sed

​	**sed**是一种流编辑器，它是文本处理中非常好的工具，能够完美的配合正则表达式使用，功能不同凡响。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed[命令](https://www.linuxcool.com/)处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件，可以将数据行进行替换、删除、新增、选取等特定工作，简化对文件的反复操作，编写转换程序等。

```shell
sed的命令格式：sed [options] 'command' file(s);
sed的脚本格式：sed [options] -f scriptfile file(s);
```

### 选项

ps：（sed的删除默认带-e，不对具体文件进行操作）

```shell
 -e ：直接在命令行模式上进行sed动作编辑，此为默认选项;
 -f ：将sed的动作写在一个文件内，用–f filename 执行filename内的sed动作;
 -i ：直接修改文件内容;
 -n ：只打印模式匹配的行；
 -r ：支持扩展表达式;
 -h或--help：显示帮助；
 -V或--version：显示版本信息。
```

### 具体参数

```shell
 a\ 在当前行下面插入文本;
 i\ 在当前行上面插入文本;
 c\ 把选定的行改为新的文本;
 d 删除，删除选择的行;
 D 删除模板块的第一行;
 s 替换指定字符;
 h 拷贝模板块的内容到内存中的缓冲区;
 H 追加模板块的内容到内存中的缓冲区;
 g 获得内存缓冲区的内容，并替代当前模板块中的文本;
 G 获得内存缓冲区的内容，并追加到当前模板块文本的后面;
 l 列表不能打印字符的清单;
 n 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令;
 N 追加下一个输入行到模板块后面并在二者间嵌入一个新行，改变当前行号码;
 p 打印模板块的行。 P(大写) 打印模板块的第一行;
 q 退出Sed;
 b lable 分支到脚本中带有标记的地方，如果分支不存在则分支到脚本的末尾;
 r file 从file中读行;
 t label if分支，从最后一行开始，条件一旦满足或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾;
 T label 错误分支，从最后一行开始，一旦发生错误或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾;
 w file 写并追加模板块到file末尾;
 W file 写并追加模板块的第一行到file末尾;
 ! 表示后面的命令对所有没有被选定的行发生作用;
 = 打印当前行号;
 # 把注释扩展到下一个换行符以前;
```

### 案例

先把password复制一份到password.sed上：

```shell
cp password password.sed
```

#### 删除

* 删除第二行数据（sed默认使用-e，不改变文件，所以文件没发生改变）

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# cat password.sed 
root:x:0:0:root:/root:/bin/bash  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# sed '2d' password.sed 
root:x:0:0:root:/root:/bin/bash  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# cat  password.sed 
root:x:0:0:root:/root:/bin/bash  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 对文件进行真正的删除（-i）
  * 这边删除完，我用cp又复制回去会

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# sed -i '2d' password.sed 
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# cat password.sed 
root:x:0:0:root:/root:/bin/bash  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 删除2~4行数据，删除第二行往后的所有数据

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# sed '2,4d' password.sed 
root:x:0:0:root:/root:/bin/bash  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# sed '2,$d' password.sed 
root:x:0:0:root:/root:/bin/bash  
```

#### 增加

* 在第二行后(亦即是加在第三行)加上『drink tea?』字样）add（a是加在后面）

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed |sed '2a drink tea '
     1	root:x:0:0:root:/root:/bin/bash  
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
drink tea 
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin  
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 那如果是要在第二行前加上drink tea，则使用insert（i是加在后面）

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed | sed '2i drink tea'
     1	root:x:0:0:root:/root:/bin/bash  
drink tea
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin  
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 如果是要增加两行以上，在第二行后面加入两行字，例如『Drink tea or .....』与『drink beer?』（使用换行符\）

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed | sed '2a drink tea ......\
> dirnk beer'
     1	root:x:0:0:root:/root:/bin/bash  
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
drink tea ......
dirnk beer
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin  
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

#### 替换与显示

* 将第2-4行的内容取代成为『No 2-4 number』

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed |sed '2,4c No 2-4 number'
     1	root:x:0:0:root:/root:/bin/bash  
No 2-4 number
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 只显示2-4行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed |sed -n '2,4p'
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin  
```

#### 搜索

* 搜索带root的行（如果只要匹配行，则可以加上-n）

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed |sed   '/root/p'
     1	root:x:0:0:root:/root:/bin/bash  
     1	root:x:0:0:root:/root:/bin/bash  
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin  
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* 搜索带var的行，并且删除

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# nl password.sed |sed   '/var/d'
     1	root:x:0:0:root:/root:/bin/bash  
     2	bin:x:1:1:bin:/bin:/sbin/nologin  
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin  
```



# Linux基本命令

## nl

​	nl命令在[linux](http://codex.wordpress.org.cn/Linux)系统中用来计算文件中行号。nl 可以将输出的文件内容自动的加上行号！其默认的结果与 cat -n 有点不太一样， nl 可以将行号做比较多的显示设计，包括位数与是否自动补齐 0 等等的功能。 

```
nl [选项]... [文件]...
```

### 命令参数

```shell
-b  ：指定行号指定的方式，主要有两种：
-b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
-b t ：如果有空行，空的那一行不要列出行号(默认值)；
-n  ：列出行号表示的方法，主要有三种：
-n ln ：行号在萤幕的最左方显示；
-n rn ：行号在自己栏位的最右方显示，且不加 0 ；
-n rz ：行号在自己栏位的最右方显示，且加 0 ；
-w  ：行号栏位的占用的位数。
```

## scp

### 简单的传输文件

```shell
Administrator@XTZJ-20210428JL MINGW64 ~/Desktop
$ scp Linux操作命令.md root@121.196.202.119:/home/linux-study
The authenticity of host '121.196.202.119 (121.196.202.119)' can't be established.
ED25519 key fingerprint is SHA256:4kPpb4CLNd1IVRzy+lZSZPPMNVHiEmMVvolZxxsKcOY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '121.196.202.119' (ED25519) to the list of known hosts.
root@121.196.202.119's password:
Linux操作命令.md                          100% 1262    44.8KB/s   00:00
```

