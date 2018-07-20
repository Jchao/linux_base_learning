# Linux Shell 编程

## 基本格式
在编写 shell 程序时，请依照以下格式

```
#!/usr/bin/env bash
# =====================================================
# 这是一个 shell 脚本
# CC <cc@redflag-linux.com>
# test.sh
# =====================================================
.
.
echo "hello world!"
command...
.
.
```
作为传统，第一个 shell 程序，hello world!

## Shell 中的变量与数组
### 1. 变量的定义与清除
Shell 中的变量是弱类型，无须声明可直接赋值，引用时需在前面加美元符 $：
```
[cc@redflag ~]$ r_name=redflag
[cc@redflag ~]$ echo $r_name
redflag
[cc@redflag ~]$ set | grep r_name
r_name=redflag
[cc@redflag ~]$ unset r_name
```
如有兴趣，可以自己试一下，首先变量名=值，等号两边是不能有空格的，否则赋值失败，同时运行内置命令 set ，可以查看当前 shell 的所有变量和函数定义，使用 unset 可以清除变量定义。
### 2. 字符串的定义及单双引号与大括号的应用
在定义字符串变量时，若变量为非连续的(空格分隔)，则需使用引号将内容包含起来或在空格前使用反斜线进行转义（\）：
```
# No news is good news.
[cc@redflag ~]$ a='I '
[cc@redflag ~]$ b="love "
[cc@redflag ~]$ c=linux\ shell
[cc@redflag ~]$ echo $a$b$c
I love linux shell
```
此外，在单(双)引号中可直接使用双(单)引号，或使用反斜线转义内容中的单(双)引号。
在变量中引用变量，必须使用双引号(“)：
```
[cc@redflag ~]$ d='$c is good'
[cc@redflag ~]$ echo $d
c is good
[cc@redflag ~]$ e="$c is good"
[cc@redflag ~]$ echo $e
linux shell is good
```
大括号在 shell 变量中的应用也颇为重要，若变量被内嵌在一个字符串中，变量名两边有大括号，则括中部分则被认为是一独立变量：
```
[cc@redflag ~]$ f="$a$blinux"
[cc@redflag ~]$ echo $f			# 只识别到 $a,没有变量 $blinux
I
[cc@redflag ~]$ g="$a${b}linux"
[cc@redflag ~]$ echo $g
I love linux
```
大括号除了作为变量的定界符外，也有扩展功能，其内以逗号作为分隔，加前导与后续，组成新的字符串：
```
[cc@redflag ~]$ mkdir -p tmp/x{aa,bb,cc} && ls tmp
xaa xbb xcc
[cc@redflag ~]$ echo a{b,c,d{x,y,z}}e		# 思考一下？
???
```
### 3. 将命令执行结果赋予变量
在命令中将其它命令的输出作为参数，或直接将命令执行结果赋予变量，可以使用反引号或加美元符的括号将命令包起来：
```
$ 变量=`命令`
$ 变量=$(命令)
```
一般情况下，两种方法都可使用且第一种更为便捷，但在多重嵌套中，则必须使用第二种方法：
```
[cc@redflag ~]$ hw=`echo `uname -i``
bash: -i: 未找到命令...
[cc@redflag ~]$ hw=$(echo $(uname -i))
[cc@redflag ~]$ echo $hw
x86_64
```
### 4. 读取键盘键入值
使用内置函数 **read 命令** ，可通过键盘输入对变量赋值。
| 参数 | 说明 |
| - | - |
| -a | 赋值给一数组 |
| -p | read -p 提示信息 变量 |
| -t num | timeout 时间，单位为秒，超时自动退出 |
| -n num | 读取字符数，超出后自动退出并将值赋予变量 |
| -s | slience mode 类似 Linux 密码输入界面，静默读取并赋值 |
### 5. 定义只读变量
和诸多语言一样，shell 中也支持只读变量的定义，只读变量不可再次赋值，同时也不 允许被取消定义。声明可使用 readonly 或 declare -r：
```
[cc@redflag ~]$ readonly r
[cc@redflag ~]$ r="readonly variable"
bash: r: readonly variable
[cc@redflag ~]$ unset r
bash: unset: r cannot unset: readonly variable
[cc@redflag ~]$ declare -r t=10
[cc@redflag ~]$echo $t
10
```
### 6. 变量的导出与应用
我们在将一个变量赋值后，该变量只能在当前的 shell 中生效，进入子 shell 中变量便不再起作用。下面一个小例子：
```
[cc@redflag ~]$ name="Jerry J"
[cc@redflag ~]$ echo $name
Jerry J
[cc@redflag ~]$ bash		# 执行 bash 进入子 shell 中
[cc@redflag ~]$ echo $name
			# 显示空行，即变量为空
[cc@redflag ~]$ exit		# 退回到父 shell
[cc@redflag ~]$ echo $name
Jerry J
```
由上可见，在父 shell 中定义的变量，在切入新的子 shell 中，变量没有定义，而退回后变量依旧生效，我们可以使用两种方式导出变量，使子 shell 可继承父 shell 中的变量定义：
```
[cc@redflag ~]$ export EXP1="export variable1"
[cc@redflag ~]$ declare -x EXP2="export variable2"
[cc@redflag ~]$ bash
[cc@redflag ~]$ echo -e "$EXP1 \n$EXP2"
export variable1
export variable2
```
在 shell 终端运行脚本时，其命令是在一个全新的子 shell 中运行的，同时偶尔也在进入二级甚至三级子 shell 中运行命令，故需要定义一个有穿透力的变量，以简化脚本，方便使用。同时也可以通过 export -p 或 declare -xp 列出所有已导出变量：
```
[cc@redflag ~]$ export -p		# 部分输出如下
...
declare -x SSH_TTY="/dev/pts/1"
declare -x TERM="xterm-256color"
declare -x USER="cc"
declare -x XDG_RUNTIME_DIR="/run/user/1000"
declare -x XDG_SESSION_ID="497"
```
### 7. declare 命令介绍
在上面实例中曾再次使用到 declare 命令，即定义只读变量与导出变量，以下仅列出它的常用参数 ，详细请查看 man 手册：
| 参数 | 说明 |
| - | - |
| -a | 定义索引数组 |
| -A | 定义关联数组 |
| -i | 定义整形数(使变量属性为整型) |
| -l | 赋值时将所有大写字母转换为小写 |
| -r | 赋予只读属性 |
| -u | 赋值时将所有小写字母转换为大写 |
| -p | 打印出属性及值 |
| -x | 导出变量或函数 |
此命令非常简单，不作示例，可自行尝试、熟悉。
### 8. 环境变量与特殊变量
#### 1. 环境变量
在 Linux Shell 中的操作离不开环境变量，同时上面也接触到了一些，环境变量一般使用大写字母，同时会被导出使其有穿透力(export)，如 PATH，查看当前环境变量可使用 env 命令：
```
[cc@redflag ~]$ env		# 部分输出如下
...
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1000
EXP2=2
EXP1=1
_=/usr/bin/env
```
在 Linux 中定义变量的习惯是：普通变量用小写，环境变量用大写。而相较于普通变量，环境变量需要使用 **export 命令**导出，向上翻查学习过的内容，会发现一直都在遵循这个规律。
#### 2. 特殊变量
除环境变量外，Bash 还自带一些**特殊**内置变量，其属于内置变量的一部分，但在平时工作中会经常用到，故单拿出做一简单介绍：
| 变量 | 含义 |
| - | - |
| $0 | 脚本自身的命令(脚本中)，或 shell 的名字(命令行，如：bash) |
| $N | 脚本或函数的位置参数 $1, $2... ${10}...，N 大于9时需用{}括起来 |
| $# | 位置参数的个数 |
| $* | 所有的位置参数(一起作为单个字符串) |
| $@ | 所有的位置参数(每个作为独立的字符串) |
| $_ | 上条命令的最后一个参数 |
| $? | 上条命令的退出状态(0为正确执行) |
| $$ | 当前 shell 的进程 ID |
简单的小例子：
```
[cc@redflag ~]$ cat sp_variable.sh
#!/bin/bash
echo "\$0 is $0"
echo "Script $0 have $# parameters."
echo "\$1 \$2 \$3 parameters are "$1,$2,$3""
echo "All parameters \$* are "$*""
echo "All parameters \$@ are "$@""

[cc@redflag ~]$ ./sp_variable.sh 1 2 3 4 5
$0 is ./sp_variable.sh
Script ./sp_variable.sh have 5 parameters.
$1 $2 $3 parameters are 1,2,3
All parameters $* are 1 2 3 4 5
All parameters $@ are 1 2 3 4 5
[cc@redflag ~]$ echo $_
5
[cc@redflag ~]$ echo $?
0
```
通过这个例子，应该已经很好的理解了位置参数及其应用，就目前来看，$* 与 $@ 似乎作用相同，其实不然，在后续讲  for 循环时，会说明其区别。
**set 命令**可以丢弃、重置位置参数，在脚本中：
```
set par1 par2 par3		# 清除所有位置参数并重新赋值
...
set --					# 清除所有位置参数( $1,$2...)
```

### 9. 常用内置变量
此表将列出一些常用的 Bash 内置变量
| 变量 | 含义 |
| - | - |
| BASH | bash 的完整路径，默认 /bin/bash |
| HOME | 用户的主目录，普通用户一般为 /home/用户名 |
| HOSTNAME | 主机名 |
| LOGNAME | 当前用户的登录名 |
| PWD | 当前工作目录，echo $PWD 等同于 pwd 命令 |
| PATH | 外部命令搜索路径，以冒号(:)分隔 |
| RANDOM | 取一个随机数，范围(0-32767) |
| REPLY | 若 read 命令后没加变量名，则变量存储于此变量中 |
| SECONDS | 当前 shell 的启动时间 |
| SHELL | 当前默认 shell，当前为 /bin/bash |
| TIMEOUT | 等待时间，单位 S ，若此时间内无操作自动退出登录 |
| UID | 当前用户的 id 号|
### 10. 变量的测试与赋值
在编写脚本时，定义一个变量之前，有时需要测试变量名是否已被使用(有无赋值)，给它一个默认值，Bash 提供了这项功能：
| 表达式 | 说明 |
| - | - |
| ${var-Default} | 若未定义 var，则以 Default 作为表达式的值，var 不变 |
| ${var:-Default} | 若未定义 var或 var 的值为空，则以 Default 作为表达式的值，var 不变|
| ${var=Default} | 若未定义 var，则 Default 为 var 和表达式的值 |
| ${var:=Default} | 若未定义 var 或其值为空，则 Default 为 var 和表达式的值 |
| ${var+Other} | 若定义了 var，则表达式的值为 Other，var 不变|
| ${var:+Other} | 若定义了 var（非空），则表达式值为Other，var 不变 |
| ${var?Message} | 若未定义 var，则打印 Message 信息，var 不变 |
| ${var:?Message} | 若未定义 var 或其值为空，则打印 Message 信息，var 不变 |
| ${!varprefix*} | 匹配之前所有声明的以 varprefix 开头的变量 |
| ${!varprefix@} | 匹配之前所有声明的以 varprefix 开头的变量 |
从上面的表格中，很容易找到几个共同点。
其一，如若判断中有冒号(:)，则会考虑变量已被定义，值却为空的情况(等同于未定义)；
其二，+ 与 - 是相反运算，如 ${var-Default} 与 ${var+Other} 作用刚好相反；
其三，只有表达式中有等号(=)，var 的值才有可能改变。
一个简单的小例子：
```
[cc@redflag ~]$ cat drink.sh
#!/bin/bash
echo "Would you like something to drink?"
read drink
echo "I'd like a cup of ${drink:-coffee}."
[cc@redflag ~]$ ./drink.sh
Would you like something to drink?
			# 直接回车，drink 已定义，但为空
I'd like a cup of coffee.
```
### 11. 字符串操作
在 Linux 下，对字符串的操作常用 awk 和 sed 命令，功能强大使用却并非特别便捷，Bash 里也内置了一些字符串操作功能，速度更快也更为简单，如下：
| 表达式 | 说明 |
| - | - |
| ${#string} | 字符串 string的长度 |
| ${string:position} | 从字符串 string 中位置 position 开始提取字符串 |
| ${string:position:length} | 从字符串 string 中位置 position 开始提取长度为 length 的字符串 |
| ${string#regexp} | 从字符串 string 开头始，删除匹配的最短 regexp 的字符串 |
| ${string##regexp} | 从字符串 string 开头始，删除匹配的最长 regexp 的字符串 |
| ${string%regexp} | 从字符串 string 结尾始，删除匹配的最短 regexp 的字符串 |
| ${string%%regexp} | 从字符串 string 结尾始，删除匹配的最长 regexp 的字符串 |
| ${string/regexp/replacement} | 使用 replacement 代替匹配到的第一个 regexp |
| ${string//regexp/replacement} | 使用 replacement 代替匹配到的所有 regexp |
| ${string/#regexp/replacement} | 若字符串 string 的前缀为 regexp，则使用 replacement 替代 |
| ${string/%regexp/replacement} | 若字符串 string 的后缀为 regexp，则使用 replacement 替代 |
注：上表中的**regexp**为一**正则表达式**
老规矩，先观察，总结规则，找出共同点：
一，凡涉及数字(位置，长度)，则使用冒号连接参数；
二，若使用匹配功能，涉及前缀/后缀或由始起/由结尾起，必定用到 #/%;
三，替换命令为斜线(/)，参数紧随其后；
四，匹配时，同一命令使用两次表示最长或全部(##，%%，//)
几个小例子：
```
# 查看字符串长度
[cc@redflag ~]$ str=morning
[cc@redflag ~]$ echo ${#str}
7
# 提取第1-5个字符
[cc@redflag ~]$ echo ${str:1:5}
orni
# 从字符串 str 结尾，删除最长的 n*g
[cc@redflag ~]$ echo ${str%%n*g}
mor
# 未完
```
### 12. 数组
Bash 中，只可定义一维数组，其下标起始值为0，只需用小括号括起来，其内元素以空格分隔，即可定义一数组：
```
[cc@redflag ~]$ arr=(this is array)
# 指定数组下标时，必须用{}括起来，不然 $arr 等同于其首个元素(第0个)
[cc@redflag ~]$ echo ${arr[1]}, $arr[2]
is, this[2]	
```
通过${arr[*]}可以列出数组中所有元素，${#arr[*]}可以得到数组的元素个数，${!arr[*]}可以取到数组的下标。
```
[cc@redflag ~]$ echo ${arr[*]} ${#arr[*]}
this is array 3
```
对数组变量的赋值，还可以用 read 和 declare 命令：
```
# -a 参数指定数组
[cc@redflag ~]$ read -a rarr
aa bb cc
# 给数组增加或修改元素元素，可直接通过下标定义
[cc@redflag ~]$ rarr[3]=gg
[cc@redflag ~]$ rarr[2]=ff
[cc@redflag ~]$ echo ${rarr[*]}
aa bb ff gg
# declare 定义数组，-a 为索引数组，-A 为关联数组
[cc@redflag ~]$ declare -A stu1=([name]='Jerry' [sex]='male' [age]=18)
# ${array[*]} 与 ${array[@]} 都可列出数组所有元素，但有所不同
[cc@redflag ~]$ echo -e "${!stu1[@]} \n${stu1[@]}"
name age sex
Jerry 18 male
[cc@redflag ~]$ unset arr rarr stu1
```
上面用到了关联数组，打印数组后可以发现，显示出来的元素顺序和定义时顺序是不一样的，因为它采用了散列法存储，而非普通数组的顺序法。
同样的，定义 stu1的用法可以用来定义下标不连续的索引数组。
先简单介绍下 ${array[\*]} 与 ${array[@]} 的不同，* 是将数组中所有元素作为一个字符串表示出来，而 @ 则是分别列出所有元素，并以空格分隔，即遍历一次，后续在讲 **for** 循环时，会看出其区别。
### 13. expr 计算表达式的值与处理字符串
#### 1. 计算表达式值
外部命令 expr 用于计算表达式的值，格式为：
	expr 表达式
同时需注意，运算符两边都需要有空格，否则运算无法进行
```
[cc@redflag ~]$ expr 1+2
1+2
[cc@redflag ~]$ expr 1 + 2
3
[cc@redflag ~]$ expr \( 1 + 2 \) \* 3
9
```
进行基本四则运算，需注意：所有**元素**均需使用**空格分隔开**，因乘号(*)在 Linux 中的特殊含义，故和括号一样，需使用反斜线进行转义。
用 expr 命令亦可以进行变量参与的运算，且支持自增(减)运算
```
[cc@redflag ~]$ i=1
[cc@redflag ~]$ i=$(expr $i + 1)
[cc@redflag ~]$ echo $i
2
[cc@redflag ~]$ i=`expr $i + 1`
[cc@redflag ~]$ echo $i
3
```
使用 expr 命令进行计算时，变量必须是整数，否则会出错，可以依此特性**判断某变量是否为整数**
```
[cc@redflag ~]$ n=a
[cc@redflag ~]$ expr $n + 1
expr: no-integer argument
```
同样的，expr 也可以用在关系运算中(整数)，如 <，<=，!=，=，e.g.，与 C 语言一样，0为假，1为真，这里需留神，因为 < 与 >在 Shell 中有输入(出)重定向的含义，故也须加反斜线处理：
```
[cc@redflag ~]$ i=3
[cc@redflag ~]$ expr $i \>= 3
1
[cc@redflag ~]$ unset i n
```
#### 2. 处理字符串
如表
| 命令 | 含义 |
| - | - |
|expr length string | 字符串 string 的长度 |
| expr index string char | 查找字符 char 在 string 中首次出现的位置(起始为1)，没有则返回0 |
| expr substr string pos length | 从字符串 string 的位置 pos 开始，截取一长度为 length 的子串 |
| expr match string regexp | 从字符串 string **开头**匹配 regexp 的长度 |
| expr string : regexp | 同上，注意空格分隔 |
| expr match string \(regexp\) | 从字符串 string **开头**位置提取 regexp |
| expr string : \\(regexp\\) | 同上，注意括号前的反斜线 |
计算字符串长度，既可以用于变量(等同 ${#string})，也可以用于普通字符串：
```
[cc@redflag ~]$ expr length "hello shell"
11
```
及查找字符出现位置，截取字符串的小例子：
```
[cc@redflag ~]$ expr index 'hello shell' s
7
[cc@redflag ~]$ expr substr 'hello shell' 7 7
shell
```
在截取字符串时，若最终长度超过剩余长度，则到最后最后一个字符停止
在上表中，regexp 表示正则表达式，这里先简单介绍一下，“.”代表任意字符，“[a-z]”表示任意一个小写字母，“*”则表示匹配0次或多次：
```
# 计算字符串以小写字母开头的长度
[cc@redflag ~]$ expr 'abcde!@abc' : '[a-z]*'
5
# 若字符串以小写字母开头，则将之提取出来
[cc@redflag ~]$ expr 'abcde!@abc' : '\([a-z]*\)'
abcde
```
提取字符串的进阶用法，尝试提取一个文件的文件名或后缀名，这里需**注意**的是，匹配的起始位置必定是字符串的开头，但提取的部分则为“\(regexp\)”:
```
[cc@redflag ~]$ file=test12#4.md
# 此匹配分为两部分，“.*”匹配任意多个字符，“\.md”则是以“.md”为结尾
[cc@redflag ~]$ expr $file : '\(.*\)\.md'
test12#4
# 其一，由始起任意多个字符且以最后一个“.”结尾，其二，所需提取的后缀
[cc@redflag ~]$ expr $file : '.*\.\(.*\)'
md
```
使用 expr 提取字符串时，遵循最大化原则，即匹配至最后符合条件的位置结束，有兴趣可以自己试下，看看提取 test.doc.md 得到什么。



## Shell 程序中常用的命令

### echo
**echo 命令**与 PHP 中的 echo 命令一样，用于变量或字符串输出
##### 1. 显示普通字符串
```
[cc@redflag ~]$ echo "Test echo" 
Test echo
```
##### 2. 显示转义字符(仅限于特殊字符)
如 1 中，若想打印出双引号：
```
[cc@redflag ~]$ echo ""Test echo""
Test echo
[cc@redflag ~]$ echo "\"Test echo\""		# 这里需用反斜线，与其它语言一样
"Test echo"
```
##### 3. 进行换行或插入一个制表格
```
[cc@redflag ~]$ echo "\"\n\tTest echo\""
"\n\tTest echo"
```
在 shell 中执行可看到换行与 tab 键未成功转义，这里我们需要添加参数 -e 开启：
```
[cc@redflag ~]$ echo -e "\"\n\tTest echo\""
"
	Test echo"
```
故现需养成一个良好的习惯，使用转义符，便定要加上 -e 参数
##### 4. 使用变量
read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量：
```
#!/usr/bin/env bash
echo "Please input your name:"
read name
echo "Hello, $name!"
```
将之保存为 test.sh，执行：
```
[cc@redflag ~]$ sh test.sh
Please input your name:
CC
Hello, CC!
```
##### 5. 原样输出，不进行转义
应已注意到，上述所有操作均采用双引号包含引用内容，若不执行转义操作，可采用单引号：
```
[cc@redflag ~]$ echo '$PWD\"'
$PWD\"
```
### test
** test 命令**用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。
##### 1. 数值测试
| 参数 | 说明 |
| - | - |
| -eq | 等于则为真 |
| -ne | 不等于则为真 |
| -gt | 大于则为真 |
| -ge | 大于等于则为真 |
| -lt | 小于则为真 |
| -le | 小于等于则为真 |

实例测试：
```
num1=100
num2=99
if test $[num1] -gt $[num2]
then
	echo "num1 greater than num2"
else
	echo "num1 equals or less than num2"
fi
```
测试结果：
```
num1 greater than num2

```
##### 2. 字符串测试
| 参数 | 说明 |
| - | - |
| = | 等于则为真 |
| != | 不等于则为真 |
| -z | 字符串长度为零则为真 |
| -n | 字符串长度不为零则为真 |
实例测试：
```
str1=tata
str2=tato
if test $str1 = $str2
then
	echo "=="
else
	echo "!="
fi
```
测试结果：
```
!=
```
##### 3. 文件测试
| 参数 | 说明 |
| - | - |
| -e FILE | 若文件存在则为真 |
| -r FILE | 若文件可读则为真 |
| -w FILE | 若文件可写则为真 |
| -x FILE | 若文件可执行则为真 |
| -d FILE | 若文件存在且为目录则为真 |
| -f FILE | 若文件存在且为文件则为真 |
| -s FILE | 若文件存在且至少有一个字符则为真 |
| -c FILE | 若文件存在且为字符型特殊文件则为真 |
| -b FILE | 若文件存在且为块特殊设备则为真 |
实例测试：
```
if test -r /etc/passwd
then
	echo right
else
	echo wrong
fi
```
测试结果：
```
right
```
### find
**find命令**用来在指定目录下查找文件，任何位于参数之前的字符串都将被视为欲查找的目录名。若使用此命令时不加任何参数，则默认查找并列出当前目录下所有子目录与文件：
```
find [选项参数] [查找目录] [匹配表达式]	# 默认从当前目录起始
```
| 参数 | 说明 |
| - | - |
| -name FILE | 按文件名查找，可使用通配符（*为多个字符，? 为一个字符） |
| -perm mode | 按权限查找 (e.g., 660) |
| -group/user G/U | 查找某个所属组/用户的文件 |
| -mtime -n/+n | 查找最后修改时间在 n 天以内(外)的文件 |
| -type -b/d/c/p/l/f | 查找某类型文件 (e.g.，-f 普通，-d 目录) |
| -size {+/-}n{b,c,w,k,M,G} | 查找大于(+)或小于(-)某值的文件 |
| -maxdepth/mindepth | 设置向下查找的目录层级 |
| -exec command {} \; | 对匹配的文件执行该参数所给出的shell命令  |
实例测试：
```
[cc@redflag ~]$ find . -type f -size +1k -mtime -3 -exec ls -alh {} \;
-rw------- 1 cc cc 392K 6月   9 19:20 ./.zsh_history
-rw-r--r-- 1 cc cc 2.3K 6月   7 22:19 ./.ssh/known_hosts
-rw------- 1 cc cc 9.5K 6月   8 11:22 ./.viminfo
-rw------- 1 cc cc 2.5K 6月   8 13:26 ./.bash_history
```
上述命令是查找当前目录下，3天内有过更改且大于1k 的文件(非目录)，且使用 ls -alh 命令列出，使用时定要注意：{} 与 反斜线 \ 间的空格，且以 ；结束。
### grep
**grep 命令**是一种常用、强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。
以下仅列出常用参数供参考，详细请查看 man 文件。
| 参数 | 说明 |
| - | - |
| -a | 不要忽略二进制数据文件 |
| -A n | 打印出匹配行及其后续 n 行 |
| -c | 统计出符合的行数(不打印) |
| -n | 打印的同时输入所在行数 |
| -l/L | 只输出匹配/不匹配内容所在文件的文件名 |
| -r | 目标是目录且期望递归查找 |
| -i | 查找时忽略字母大小写 |
| -e <匹配格式> | 指定匹配样式，可多次指定，关系为或 |
| -v | 反转查找 |
| -o | 只输出文件上匹配到的部分 |
| -E | 等同于 egrep，即可使用扩展正则表达式匹配 |
实例测试：
在多个文件中查找相关内容
```
grep "matches" file1 file2 file3 ...
```
搜索多个文件并查找匹配文本在哪些文件中
```
grep -l "matches" file1 file2 file3 ...
```
查找指定目录中的所有文件，不区分大小写，且显示匹配的行数
```
grep -iarn "matches" <DIR>
```
在grep搜索结果中包括或者排除指定文件：
```
# 仅在 html 和 php 文件中查找
grep -r "main()" . --include *.{html,php}

# 在搜索结果中排队所有 js 文件
grep -r "main()" . --exclude *.js

# 从文件列表中排除某些文件（注意，没有 --include-from 参数）
grep -r "main()" . --exclude-from filelists
```
### sort
**sort 命令**是 Linux 中非常好用的排序命令，简单实用
```
sort [选项参数] [文件]
```
| 参数 | 说明 |
| - | - |
| -b | 忽略每行前的空格字符 |
| -c | 检查文件是否已按顺序排序 |
| -n | 依照数值大小排序 |
| -r | 进行反向排序 |
| -m | 后跟多个文件，对已排序的文件进行合并 |
| -u | 忽略相同重复行，启用 uniq 命令|
| -R | 将文件内容进行随机排序 |
实例测试
sort 使用方法很简单，既可以从读取特定文件，也可从 stdin 中获取输入：
```
# 获取系统已安装所有软件包
[cc@redflag ~]$ rpm -qa > rpms && rpm -qa >> rpms

# 将 rpms 去重排序并输出到原文件 rpms
[cc@redflag ~]$ sort rpms -o newlist
```
### uniq
uniq命令用于统计、报告或忽略文件中的重复行，一般与sort命令结合使用。下面仅说明常用规则。
| 参数 | 说明 |
| - | - |
| -c | 去重后，在第行前显示出现次数 |
| -d | 仅显示重复出现的行列 |
| -s | 仅显示出现一次的行列 |




