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
​	awk会根据分隔符将行分成若干个字段，$0为整行，$1为第一个字段，$2 为第2个地段，依此类推…为打印一个字段或所有字段，使用print命令。这是一个`awk`动作
