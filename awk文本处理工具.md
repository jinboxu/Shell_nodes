## awk文本处理工具

#### awk简介

awk 是一种编程语言，用于在 linux/unix 下对文本和数据进行处理。 

awk 的处理文本和数据的方式是这样的，它逐行扫描文件，从第一行到最后一行，寻找匹配 的特定 模式的行，并在这些行上进行你想要的操作。如果没有指定处理动作，则把匹配的行显示到 标准输出( 屏幕)，如果没有指定模式，则所有被操作所指定的行都被处理。



#### awk 的两种形式语法格式 

```shell
awk [options] 'commands' filenames 
awk [options] -f awk-script-file filenames
```

 

> ＝＝options： 

-F 定义输入字段分隔符，默认的分隔符是空格或制表符(tab) 



>＝＝command： 

BEGIN{}  {}  END{} 

行处理前 行处理 行处理后 

```shell
[root@localhost scripts]# awk 'BEGIN{print "Hello"} /root/{print $1} END{print "ok"}' passwd 
Hello
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
dockerroot:x:996:993:Docker
ok
[root@localhost ~]# awk -F":" 'BEGIN{print "Hello"} /root/{print $1} END{print "ok"}' /etc/passwd 
Hello
root
operator
dockerroot
ok
```

*BEGIN{} 通常用于定义一些变量，例如 BEGIN{FS=":";OFS="---"}*

```shell
[root@localhost ~]# awk -F":" 'BEGIN{print 2/5}'    //BEGIN发生在读文件之前
0.4
[root@localhost ~]# awk  'BEGIN{FS=":"} {print $1}' /etc/passwd
...略
[root@localhost ~]# awk  'BEGIN{FS=":";OFS="=="} /root/{print $1,$2} END{print "ok"}' /etc/passwd    //OFS输出字段分隔符
root==x
operator==x
dockerroot==x
ok
```



#### awk工作原理

\#awk -F:  '{print \$1,\$3}'  /etc/passwd

- (1)awk 使用一行作为输入，并将这一行赋给内部变量$0，每一行也可称为一个记录，以换行 符结束 
- (2)然后，行被:（默认为空格或制表符）分解成字段（或域），每个字段存储在已编号的变量 中，从$1 开始， 最多达 100 个字段 
- (3)awk 如何知道用空格来分隔字段的呢？ 因为有一个内部变量 FS 来确定字段分隔符。初始 时，FS 赋为空格 
- (4)awk 打印字段时，将以设置的方法使用 print 函数打印，awk 在打印的字段间加上空格， 因为\$1,\$3 之间 有一个逗号。逗号比较特殊，它映射为另一个内部变量，称为输出字段分隔符 OFS，OFS 默 认为空格 
- (5)awk 输出之后，将从文件中获取另一行，并将其存储在$0 中，覆盖原来的内容，然后将 新的字符串分隔 成字段并进行处理。该过程将持续到所有行处理完毕 



> awk内部变量

> \$0： awk 变量$0 保存当前记录的内容 # awk -F: '{print $0}' /etc/passwd 
>
> NR: The total number of input records seen so far.  总记录数
>
> FNR： The input record number in the current input file  当前文件记录数
>
> 
>
> NF： 当前记录字段数 
>
> FS： 输入字段分隔符
>
> OFS： 输出字段分隔符 
>
> RS
>
> ORS

**字段分隔符：  FS   OFS    默认为空格或制表符**

**记录分隔符：  RS  ORS   默认为换行符**

```shell
[root@localhost ~]# awk -F: '{print NR,$0}' /etc/hosts ./scripts/dd.txt 
1 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
2 ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
3     		   	 	
4 		  	 
5 fegg  fff
6 		fff   	
[root@localhost ~]# awk -F: '{print FNR,$0}' /etc/hosts ./scripts/dd.txt 
1 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
2 ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
1     		   	 	
2 		  	 
3 fegg  fff
4 		fff

[root@localhost ~]# awk -F: '/bash$/{print $0,NF}' ./scripts/passwd  //当前记录字段数
root:x:0:0:root:/root:/bin/bash 7
aa:x:1000:1000::/home/aa:/bin/bash 7
bb:x:1001:1001::/home/bb:/bin/bash 7
cc:x:1002:1002::/home/cc:/bin/bash 7
ee:x:1003:1003::/home/ee:/bin/bash 7
nn:x:1004:1004::/home/nn:/bin/bash 7
ff:x:1005:1005::/home/ff:/bin/bash 7

[root@localhost ~]#  awk 'BEGIN{FS=":"; OFS="+++"} /^root/{print $1,$2,$3,$4}' /etc/passwd
root+++x+++0+++0
```



```shell
[root@localhost scripts]# cat test.txt 
111  222   333
444
555  666
777  888  999
[root@localhost scripts]# awk 'BEGIN{ORS=" "} {print $0}' test.txt  //将文件的每一行合并为一行
111  222   333 444 555  666 777  888  999 [root@localhost scripts]# 
[root@localhost scripts]# awk 'BEGIN{RS="[ ]+"} {print $0}' test.txt 
111
222
333
444
555
666
777
888
999

[root@localhost scripts]# 
[root@localhost scripts]# head -1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
[root@localhost scripts]# head -1 /etc/passwd > passwd1
[root@localhost scripts]# cat passwd1
root:x:0:0:root:/root:/bin/bash
[root@localhost scripts]# awk 'BEGIN{RS=":"} {print $0}' passwd1
root
x
0
0
root
/root
/bin/bash

[root@localhost scripts]# 
```



> 格式化输出

- print函数

```shell
[root@localhost ~]# date |awk '{print "Month: " $2 "\nYear: " $NF}'
Month: Jan
Year: 2019
[root@localhost ~]#awk -F: '{print "username is: " $1 "\t uid is: " $3}' /etc/passwd
```

- printf函数

```shell
# awk -F: '{printf "%-15s %-10s %-15s\n", $1,$2,$3}' /etc/passwd
# awk -F: '{printf "|%-15s| %-10s| %-15s|\n", $1,$2,$3}' /etc/passwd
```

%s 字符类型 

%d 数值类型 

占 15 字符       - 表示左对齐，默认是右对齐      printf 默认不会在行尾自动换行，加\n 



#### awk模式和动作

任何 awk 语句都由模式和动作组成。模式部分决定动作语句何时触发及触发事件。模式可以是任何条件语句或复合语句或正则表达式。  

> 正则表达式

*匹配记录(整行)*

```shell
# awk '/^alice/' /etc/passwd         //awk '$0 ~ /^alice/' /etc/passwd
# awk '!/alice/' passwd              //awk '$0 !~ /^alice/' /etc/passwd
```

*匹配字段：匹配操作符（~   !~）*

```shell
# awk -F: '$1 ~ /^alice/' /etc/passwd
# awk -F: '$NF !~ /bash$/' /etc/passwd
```



>条件表达式

```shell
#awk -F: '$3>300 {print $0}' /etc/passwd   //或  awk -F: '$3>300' passwd，模式匹配
#awk -F: '{ if($3>300) {print $0} }' /etc/passwd   //跟上面的效果是一致的，条件判断
#
#awk -F: '{ if($3>300) {print $3} else{print $1} }' /etc/passwd
```



>逻辑操作符和复合模式 

```shell
# awk -F: '$1~/root/ && $3<=15' /etc/passwd
# awk -F: '$1~/root/ || $3<=15' /etc/passwd
# awk -F: '!($1~/root/ || $3<=15)' /etc/passwd
```



> 范围模式

\# awk '/Tom/,/Suzanne/' filename  

```shell
[root@localhost scripts]# cat test.txt 
111  222   333
444 
555  666
777  888  999
[root@localhost scripts]# awk '/^444/,/^555/' test.txt 
444 
555  666
```



#### awk脚本编程

> = = 条件判断

**if语句**

{if(表达式)｛语句;语句;...｝} 

```shell
[root@localhost scripts]# awk -F: '{if($3==0) {print $1 " is administrator."}}' /etc/passwd
root is administrator.
[root@localhost scripts]# awk -F: '{if($3>0 && $3<1000){count++;}} END{print count}' /etc/passwd  //统计系统用户的个数
22
```



**if...else语句**

{if(表达式)｛语句;语句;...｝else{语句;语句;...}} 

```shell
[root@localhost scripts]# awk -F: '{if($3>1000) {print $0} else {print $7}}' /etc/passwd
...
[root@localhost scripts]# awk -F: '{if($3==0){count++} else{i++}} END{print "管理员个数: "count ; print "系统用户数: "i}' /etc/passwd
管理员个数: 1
系统用户数: 28
```



**if...else if...else语句**

 {if(表达式 1)｛语句;语句；...｝else if(表达式 2)｛语句;语句；...｝else if(表达式 3)｛语句;语 句；...｝else｛语句;语句；...｝} 

```shell
[root@localhost scripts]# awk -F: '{if($3==0){i++} else if($3>999){k++} else{j++}} END{print "管理员个数: "i; print "普通用个数: "k; print "系统用户: "j}' /etc/passwd 
管理员个数: 1
普通用个数: 6
系统用户: 22
```



> = = 循环

**while**

```shell
[root@localhost scripts]#  awk 'BEGIN{ i=1; while(i<=10){print i; i++} }'
1
2
3
4
5
6
7
8
9
10
[root@localhost scripts]# cat test.txt 
111  222   333
444 
555  666
777  888  999
[root@localhost scripts]# awk '{i=1;while(i<=NF) {print $i;i++}}' test.txt //依次打印每行每列 1
111
222
333
444
555
666
777
888
999
[root@localhost scripts]# 
```



**for**

```shell
[root@localhost scripts]# awk 'BEGIN{for(i=1;i<=10;i++) {print i}}'   //C 风格 for，单括号
1
2
3
4
5
6
7
8
9
10
[root@localhost scripts]# awk '{for(i=1;i<=NF;i++) {print $i}}' test.txt  //依次打印每行每列 2
111
222
333
444
555
666
777
888
999
[root@localhost scripts]# awk -F: '/^root/{for(i=1;i<=NF;i++) {print $i}}' /etc/passwd
root
x
0
0
root
/root
/bin/bash
```



#### 数组

```shell
[root@localhost scripts]# awk -F: '{username[i++]=$1} END{print username[1]}' /etc/passwd
bin
[root@localhost scripts]# awk -F: '{username[i++]=$1} END{print username[0]}' /etc/passwd
root
[root@localhost scripts]# awk -F: '{username[++i]=$1} END{print username[1]}' /etc/passwd
root
```

*i++先赋值再运算，++i先运算再赋值*



> 数组的遍历

**1. 按索引遍历**

```shell
# awk -F: '{username[x++]=$1} END{for(i in username) {print i,username[i]} }' /etc/passwd
# awk -F: '{username[++x]=$1} END{for(i in username) {print i,username[i]} }' /etc/passwd

[root@localhost scripts]# awk -F: '{shells[$7]++} END{for(i in shells) {print i": "shells[i]}}' /etc/passwd     //统计/etc/passwd 中各种类型 shell 的数量
/bin/sync: 1
/bin/bash: 7
/sbin/nologin: 19
/sbin/halt: 1
/sbin/shutdown: 1
```



**2. 按元素个数遍历(不推荐)**

```shell
# awk -F: '{username[x++]=$1} END{for(i=0;i<x;i++) print i,username[i]}' /etc/passwd  //不推荐
# awk -F: '{username[++x]=$1} END{for(i=1;i<=x;i++) print i,username[i]}' /etc/passwd //不推荐
```



#### awk自定义变量和外部变量

> 自定义变量

```shell
[root@localhost ~]# awk -v user="root" -F:  '$1==user' /etc/passwd
root:x:0:0:root:/root:/bin/bash
```



> 外部变量

**1.awk命令使用双引号的情况下** 

此时在awk命令里面使用\"$var\"就可以引用外部环境变量的var的值 

```shell
[root@localhost ~]# var="BASH";echo "unix script"| awk "gsub(/unix/,\"$var\")"
BASH script
```

**2.awk命令使用单引号的情况下** 

此时在awk命令里面使用"'"$var"'"就可以应用外部变量var的值，注意五个点表示两个双引号之间有一个单引号 

```shell
[root@localhost ~]# var="BASH";echo "unix script"| awk 'gsub(/unix/, "'"$var"'")'
BASH script
```



**3. awk正则使用外部变量**

此时在正则里面使用'"$var"'

```shell
[root@localhost jianshu]# var="cc"
[root@localhost jianshu]# awk -F: '/'"$var"'[0-9]+/{print $1,"'"$var"'"}' /etc/passwd 
cc1 cc
cc2 cc
```

