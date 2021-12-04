# Linux操作命令

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
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '{print toupper($0)}' password 
ROOT:X:0:0:ROOT:/ROOT:/BIN/BASH  
BIN:X:1:1:BIN:/BIN:/SBIN/NOLOGIN  
DAEMON:X:2:2:DAEMON:/SBIN:/SBIN/NOLOGIN  
ADM:X:3:4:ADM:/VAR/ADM:/SBIN/NOLOGIN  
LP:X:4:7:LP:/VAR/SPOOL/LPD:/SBIN/NOLOGIN
```

* 显示/etc/passwd中有var的行

```shell
[root@iZbp1jd5ee7h52j00jed2wZ linux-study]# awk -F ':' '$0 ~ /var/' password
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
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

