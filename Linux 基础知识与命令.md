# Linux 基础知识与命令
## 1. Linux 的文件目录结构组成
### 1.1 Linux 下的文件
Linux 的哲学——**万物皆文件**。
Linux 下文件可分成三类：普通文件、目录及特殊文件(如链接文件、设备文件等)。
Linux 系统严格区分大小写，文件名中可包含空格和特殊字符等(但不建议，使用时比较麻烦)，一般使用字母、数字、下划线、减号与小数点组成。在 Linux 系统下，文件 redflag，Redflag，REDflag是三个不同的文件。同时，由于万物皆文件的特性，**同一路径下，不能创建同名的文件与目录**：
```
[cc@redflag ~]$ mkdir file && cd file	# 创建新目录file 并进入
[cc@redflag file]$ touch redflag		# 创建一个空白文件 redflag
[cc@redflag file]$ mkdir redflag		# 创建目录 redflag
mkdir: cannot create directory ‘redflag’: File exists
```
**No news is good news.**
在 Linux 系统中，没有消息就是好消息：这里我们在执行前两条命令时，并无输出，说明成功执行，而在创建目录 redflag 时，则出现了报错，提示 " 'redflag' 文件已存在"，因为目录也是文件的一种。
### 1.2 目录结构 
在 Windows 下，有多个根目录，每块逻辑盘分别有一个根目录。如 C 盘和 D 盘的根目录分别为C:\ 和 D:\，而 Linux 系统中只有一个根目录，用斜线“/”表示。需注意 Linux 中的是斜线，Windows 里则是反斜线。可以通过“cd /” 和“ls”两条命令进入根目录并列出根目录下的子目录：
```
[cc@redflag ~]$ cd /
[cc@redflag /]$ ls
bin  boot  dev  etc  home  lib  lib64  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
| 目录 | 描述 |
| - | - |
| /bin | 主要可执行文件(命令)存放的目录，大多为 Linux 系统里常用命令 |
| /boot | 包括内核在内的系统启动时使用的文件 |
| /dev | 存放设备文件，Linux 系统将所有外设看成文件，操作设备时只需对其代表文件操作即可 |
| /etc | 存放系统及软件的各种配置文件 |
| /home | 系统默认创建的普通用户家目录为/home/\<user>/，用于存放用户个人配置及文件 |
| /lib | 系统的32位库文件存放目录 |
| /lib64 | 系统的64位共享库文件存放目录，仅64位系统上有此目录 |
| /mnt | 空目录，系统通用默认文件系统挂载点目录 |
| /opt | 主要存放第三方的应用和文件，如 Oracle，chrome 安装目录 |
| /proc | 内存映象文件系统，其内文件映射自内存，可通过此中文件查看系统运行情况 |
| /root | 超级管理员 root 的家目录 |
| /run | 包含系统运行时所需的文件，不可随意删除，然每次重启会抛弃并重新生成 |
| /sbin | 存放可执行文件，包含大量需超级用户权限才可使用的系统管理命令 |
| /srv | 一些网络服务启动之后，这些服务所需要取用的数据目录，如http服务 |
| /sys | 与/proc类似，也是一个虚拟文件系统，主要也是记录与内核相关的信息 |
| /tmp | 所有用户均可使用，用于存放临时文件 |
| /usr | 所有命令，程序库、文档及其它安装文件存放的目录，这些一般不会改变 |
| /var | 与/usr 相对应，存放经常改变的东西，如日志，锁文件，数据库等 |
这个介绍仅作为了解，在日常使用中，有些是使用频繁的如用户主目录，配置文件目录（/etc） 和常变数据目录（/var），其它的需要进知道从哪去找即可。

## 2. Linux 常用基础命令
下面介绍一些 Linux 中常用，或者说是必用的基础命令。
### 2.1 whoami / id / groups
这是用来查看当前登录账户信息的命令，我是谁？
查看用户名可以用`whoami`和`id -un`，此命令多用在脚本中，用来确定运行脚本的用户身份，平时好像不大用得上：
```
[cc@redflag ~]$ whoami
cc
[cc@redflag ~]$ su -
Password:
[root@redflag ~]# id -nu
root
```
平时不常用因为在登录 Shell 操作时，一般会有提示"[当前用户@主机名 位置] 权限身份"：
由上可看出，**普通用户（$）** cc 在主机名为 redflag 的 Linux 系统上操作，通过`su -`命令切换到 root，提升为**超级用户权限（#）**，同时工作目录也切换到了 root 用户的**主目录（~）**。
每个用户都有其所属组(至少一个)，查看所属组信息，可使用`id -g`和`groups`分别查看组的 Gid 与组名，若查询其它账户，只需加上用户名参数即可：
```
[cc@redflag ~]$ id -g root
0
[cc@redflag ~]$ groups root
root
```
### 2.2 查看文件清单 ls
`ls` 命令用来列出(list)目录下包含的文件，不加位置参数时，列出当前目录下的内容，如：
```
[cc@redflag ~]$ ls
directory  file  link-file
```
上面的例子可以看出，当前目录下有3 个文件，但不清楚哪些是普通文件，哪些是子目录，此时可以通过加上`-l`参数，意为 long，列出详细信息：
```
[cc@redflag ~]$ ls -l
total 4
drwxr-xr-x 2 cc cc 4096 Jun 25 00:58 directory
-rw-r--r-- 1 cc cc    0 Jun 25 00:59 file
lrwxrwxrwx 1 cc cc    4 Jun 25 00:59 link-file -> file
```
使用`ls -l`命令列出的信息有七个部分。
第一部分为前10个字符，它可以分为两个小部分，首字符表示类型，如"-"表示普通文件，“d”表示目录，“l”表示链接文件等；其余9个字符则分别是所属用户、用户组和其它用户对文件拥有的权限(rwx 对应读写执行)，如从上面可看出，file 是一个普通文件，只有其所有者有读和写的权限，其它用户只有读的权限（-rw-r--r--）。
第二部分，对于文件，表示其硬链接数量（参见`ln`命令）；对于目录，则表示其所含的子目录数，包括隐藏目录。
第三、四部分，则表示文件的所有者与其所属组；第五部分为文件的大小，即字节数；第六部分为文件的修改时间。
第七部分为文件名，若是链接文件，还会显示其链接位置。

`ls`命令默认不列出隐藏文件，可以通过加上`-a`参数，查看目录下所有文件，常和`-l`一起使用：

```
[cc@redflag ~]$ ls -al
total 44
drwx------ 5 cc   cc   4096 Jun 25 01:35 .
drwxr-xr-x 3 root root 4096 Jun 23 23:51 ..
-rw------- 1 cc   cc   2373 Jun 24 10:05 .bash_history
-rw-r--r-- 1 cc   cc     21 Jun  4 16:54 .bash_logout
-rw-r--r-- 1 cc   cc     57 Jun  4 16:54 .bash_profile
-rw-r--r-- 1 cc   cc    156 Jun 24 07:18 .bashrc
drwx------ 3 cc   cc   4096 Jun 24 07:18 .cache
drwxr-xr-x 2 cc   cc   4096 Jun 25 01:35 directory
-rw-r--r-- 1 cc   cc      0 Jun 25 00:59 file
lrwxrwxrwx 1 cc   cc      4 Jun 25 00:59 link-file -> file
drwx------ 2 cc   cc   4096 Jun 24 05:24 .ssh
-rw------- 1 cc   cc   6512 Jun 25 01:35 .viminfo
```

这里我们注意到，**所有的隐藏文件都是以“.”开头的**。如此一来在修改文件的隐藏属性时，只需加上或去掉开头的“.”即可。
在这个例子中，最上方有两个特殊的目录，一个是“ . ”，一个是“..”。“.”表示当前目录，“..”则表示当前目录的父目录，在 Linux 系统中，所有的目录都包含这两个特殊的目录，后续在使用`cp`、`cd`等命令或运行脚本时都会用到。
还有一个小例子，介绍下`-d`参数：
```
[cc@redflag ~]$ ls -l file
-rw-r--r-- 1 cc cc 0 Jun 25 00:59 file
[cc@redflag ~]$ ls -l directory
total 0
[cc@redflag ~]$ ls -ld directory
drwxr-xr-x 2 cc cc 4096 Jun 25 01:35 directory
```
查看文件本身信息，只需在`ls`后加上文件名即可，若要查看目录的本身信息，则需使用`-d`参数，否则会列出目录里的内容。
#### 小结：
列出文件用`ls`，查看所有加`-a`,若要详细需加长`-l`，不进目录要指定`-d`。

### 2.3 生成空文件或更新时间戳命令 touch
`touch`命令，通常用来创建一个新的空文件，`touch file`当 file 不存在时，会生成一个空文件，若文件已存在，则会将文件的访问、修改、状态改动时间都设定为当前时间。
使用`-a`参数则只会设定访问时间，`-m`设定修改时间，通常情况下我们更关心的是文件的修改时间，在使用`ls -l`命令时，显示的便是文件的修改时间。
```
[cc@redflag ~]$ ls
[cc@redflag ~]$ touch empty				# 创建空白文件 empty
[cc@redflag ~]$ ls -l
total 0
-rw-r--r-- 1 cc cc 0 Jun 25 23:09 empty
[cc@redflag ~]$ touch -a empty			# 仅修改访问时间
[cc@redflag ~]$ ls -l
total 0
-rw-r--r-- 1 cc cc 0 Jun 25 23:09 empty
[cc@redflag ~]$ ls -lu					# ls 命令 -u 参数，查看访问时间
total 0
-rw-r--r-- 1 cc cc 0 Jun 25 23:13 empty
```

### 2.4 浏览文件命令 cat、more、less、head 和 more
`cat`命令可以查看文件的内容，使用`cat file`会将 file 文件所有内容打印输出
```
[cc@redflag ~]$ cat /etc/passwd
root:x:0:0::/root:/bin/bash
bin:x:1:1::/:/sbin/nologin
.
.
systemd-coredump:x:979:979:systemd Core Dumper:/:/sbin/nologin
uuidd:x:68:68::/:/sbin/nologin
cc:x:1000:1000::/home/cc:/bin/bash
```
`cat`命令是“一股脑”的显示文件的**所有内容**，查看十几行的小文件还成， 若查看一些稍大的文件，在显示出所有内容后，还需向上翻页查看，并不方便。
此时可以使用 `more`命令，格式为`more file`，此时会从文件开头开始显示第一页内容，按回车键每次下翻一行，空格每次下翻一页，b 键则会向上翻一页，q 会退出命令。
或者使用`less`命令，比较类似，不过更像是 VIM 的命令，按两下`g`跳转文件开头，一下`G`跳转末尾，`d`和`u`每次向下、上翻动半页。

除此之外 ，若确定需查看的内容在文件的开头或结尾位置，只需用`head`与`tail`命令即可，就如其字面意思，不加参数时默认显示前(后)10行，或直接指定查看的行数：
```
[cc@redflag ~]$ tail -3 /etc/passwd
systemd-coredump:x:979:979:systemd Core Dumper:/:/sbin/nologin
uuidd:x:68:68::/:/sbin/nologin
cc:x:1000:1000::/home/cc:/bin/bash
```
另，`tail`命令还有一个**常用的参数 `-f`**，使用它时，当一个文件的内容不断变化(如日志文件，脚本调试时的打印信息等)，它会不断刷新，将文件里的最新内容显示出来。

### 2.5 查看当前位置 pwd 与 切换工作目录 cd
`pwd`意为“print work directory”，即可以使用此命令来查看“当前工作目录”的完整路径，告诉用户：你在哪。其后一般情况下不加参数，若目录为链接路径时，可加`-P`显示 实际路径。
```
[cc@redflag ~]$ pwd
/home/cc
[cc@redflag ~]$ ln -sf directory/ link-dir
[cc@redflag ~]$ cd link-dir
[cc@redflag link-dir]$ pwd
/home/cc/link-dir
[cc@redflag link-dir]$ pwd -P
/home/cc/directory
```
`cd`命令用于切换目录(change directory)，或者说进入某个目录，如上例中，使用 cd 命令进入创建的链接目录 link-dir 中。
首先讲几个特殊的使用方法：
```
[cc@redflag tmp]$ cd
[cc@redflag ~]$ cd -
/tmp
[cc@redflag tmp]$ cd ~
[cc@redflag ~]$pwd
/home/cc
[cc@redflag ~]$ cd ../..
[cc@redflag /]$ 
```
1. 由上面示例可看到，`cd`和`cd ~`都可以切换到用户的家目录，但相对来说第二种方法更直观一些，可读性强，在脚本中多被使用。
2. 使用参数`-`可以回到切换之前的工作路径。
3. 在之前学习`ls`命令时，提到的两个特殊目录中，".."为父目录，“../..”是父目录的父目录，类推。

下面了解一下使用`cd`切换工作目录时，使用绝对路径或相对路径。
  **绝对路径**，指的是由根目录起始的，目标位置的绝对位置，可理解为由根到目标位置的直线路径，`pwd`命令便是用来查看当前目录的绝对路径。
  **相对路径**，则是起始位置为当前目录，到目标位置的路径。

假设当前位置为 /home/cc/dir01/tmp，要进入到 /home/cc/dir02/test 目录。

```
[cc@redflag tmp]$ pwd
/home/cc/dir01/tmp
[cc@redflag tmp]$ cd /home/cc/dir02/test/		# 使用绝对路径
[cc@redflag test]$ pwd
/home/cc/dir02/test
[cc@redflag test]$ cd -
/home/cc/dir01/tmp
[cc@redflag tmp]$ cd ../../dir02/test/			# 使用相对路径
[cc@redflag test]$ pwd
/home/cc/dir02/test
```
在平时的 Shell 操作中，可以以是否简洁方便选择使用绝对或相对路径，而在写 Shell 脚本时，则要更多的考虑文件的位置变动对路径搜索的影响，选取合适的路径表达方式。

### 2.6 创建目录命令 mkdir
`mkdir`命令是用来创建目录(make directories)的，其后跟的参数为所需创建的目录。
```
[cc@redflag ~]$ mkdir redflag /home/cc/linux	# 目录可为绝对路径或相对路径
[cc@redflag ~]$ ls -l
total 8
drwxr-xr-x 2 cc cc 4096 Jun 25 06:08 linux
drwxr-xr-x 2 cc cc 4096 Jun 25 06:08 redflag
```
`-p`是此命令中最常用、实用的参数，在帮助信息中可看到
  `-p, --parents     no error if existing, make parent directories as needed`
即，若目录已存在，也不会报错；且可用来创建多层次目录。
```
[cc@redflag ~]$ mkdir aa/bb/cc/dd
mkdir: cannot create directory ‘aa/bb/cc/dd’: No such file or directory
[cc@redflag ~]$ mkdir aa/bb/cc/dd -p
[cc@redflag ~]$ mkdir -p aa/bb/cc
[cc@redflag ~]$ 
```
#### 小结
在使用`mkdir`命令时，加上`-p`参数是个好习惯，有益无害。

### 2.7 删除命令 rm 和 rmdir
`rm`命令可用于删除文件和目录，`rmdir`仅可用于删除空目录。
常用参数为`-i`删除前问询确认，`-r`将目录下所有文件一同删除，`-f`直接执行删除，不做任何确认。
```
[cc@redflag ~]$ mkdir -p a/b/c aa
[cc@redflag ~]$ rmdir a aa 
rmdir: failed to remove 'a': Directory not empty
[cc@redflag ~]$ rm -rf a
[cc@redflag ~]$ ls
[cc@redflag ~]$
```
#### 小结
`rmdir`仅在确保不删除非空目录时有用，一般情况下，删除文件或目录都会使用`rm`命令，同时`-rf`参数也是一对好搭档。

### 2.8 移动、重命名命令 mv
`mv`即为move，移动。命令的基本格式为：`mv file1 file2`，其中 file1 为源文件，file2 为目标文件，可以是文件到文件，文件到目录，目录到目录，但**不能目录到文件**。`mv`命令可以做到移动、且重命名文件：
```
[cc@redflag ~]$ touch file1
[cc@redflag ~]$ mkdir dir1 dir2 dir3
[cc@redflag ~]$ ls
dir1  dir2  dir3  file1
[cc@redflag ~]$ mv file1 file2		# 将 file1 重命名为 file2
[cc@redflag ~]$ mv file2 dir2		# 将 file2 移动到 dir2 目录下
[cc@redflag ~]$ mv dir3 dir1/bak	# 将 dir3 移动到 dir1目录下并重命名为 bak
[cc@redflag ~]$ mv dir2 dir1/bak	# 将 dir2 移动到 dir1/bak 目录下
[cc@redflag ~]$ ls dir1/bak/dir2/file2
dir1/bak/dir2/file2
```
**注意：**目录到目录的方式移动时，若目标文件不存在，则为移动并重命名，若目标文件已存在且为目录，则意为将源目录移动到目标目录中成为其子目录。

**实用参数**，`-i`和`-b`：适用于对**普通文件**的操作，若目标路径下有同名文件，`-i`会在覆盖旧文件前进行确认，而`-b`则会直接将旧文件加波浪符重命名。
```
[cc@redflag ~]$ touch aa bb
[cc@redflag ~]$ mv -b aa bb
[cc@redflag ~]$ ls
bb  bb~  dir1
```
`-u`意为 update，使用此参数时，只有目标文件不存在，或其时间戳早于源文件时，才会移动成功，即使没有覆盖目标文件，也不会报错。

### 2.9 拷贝命令 cp
`cp`即 copy，用于文件和目录的复制。其命令格式与`mv`命令相同，`cp 源文件 目标文件`。

| 参数 | 功能 |
| - | - |
| -b | backup，可参考`mv`中用法 |
| -f | force，若目标文件无法打开则将其移除并重试 |
| -i | 覆盖前询问、确认 |
| -l | 链接文件而不复制(硬链接) |
| -p | 默认保留原文件的权限、所有、时间属性 |
| -r / -R | 复制目录及目录内所有项目 |
| -u | update，参考`mv`中用法 |
上表只是简单的列出了有关`cp`命令的常用参数，更多详细参数及用法，可自行查看帮助文件。
从中我们不难发现，`cp`与`mv`命令中许多**参数的意义是相通的**。如`-b`、`-i`、`-f`、`-u`，可自行总结下规律，例：`-f`参数多数为--force，强制执行的意思？
下面使用`cp`命令做一些操作：
```
[cc@redflag ~]$ ls -l
total 8
drwxr-xr-x 2 cc cc 4096 Jun 25 19:18 dir1
drwxr-xr-x 2 cc cc 4096 Jun 25 19:18 dir2
-rw-r--r-- 1 cc cc    0 Jun 25 19:17 file1
[cc@redflag ~]$ sudo cp file1 file2
[cc@redflag ~]$ sudo cp -p file1 file3
[cc@redflag ~]$ ls -l				# 注意 file2 file3 的区别
total 8
drwxr-xr-x 2 cc   cc   4096 Jun 25 19:18 dir1
drwxr-xr-x 2 cc   cc   4096 Jun 25 19:18 dir2
-rw-r--r-- 1 cc   cc      0 Jun 25 19:17 file1
-rw-r--r-- 1 root root    0 Jun 25 19:19 file2
-rw-r--r-- 1 cc   cc      0 Jun 25 19:17 file3
[cc@redflag ~]$ cp dir1 dir2		# 对目录的操作必须加 -r 参数
cp: -r not specified; omitting directory 'dir1'
[cc@redflag ~]$ cp -r dir1 dir2
```
#### 小结
在日常对 Linux 系统的操作中，尤其是使用 root 权限时，无论是 `cp`还是`mv`命令，加上`-i`参数都是一个好习惯。同时要善于使用`-b`进行备份操作。

### 2.10 创建链接 ln
`ln`即 link ，链接。其格式与`cp`命令类似，为`ln 源文件 目标文件`。
前面我们曾多次提到链接文件，分为**软链接**和**硬链接**两种：
  **软链接**：文件以路径链接的形式存在，指向源文件，、类似快捷方式，可跨文件系统（a 磁盘到 b 磁盘）；可应用于目录。
  **硬链接**：可理解为存储上同一空间分配了不同的名称，所有的硬链接文件都指向同一节点，故而硬链接**不可跨文件系统**生效；且不可作用于目录。
| 参数 | 说明 |
| - | - |
|   | 创建硬链接文件 |
| -b | backup，同`cp`命令用法 |
| -f | force，强制创建，无论目标文件之前是否存在 |
| -i | 覆盖前问询 |
| -s | 创建软链接文件，可作用于目录 |

记得这几个参数`-b`、`-f`、`-i`！

下面小试一下`ln`命令：
```
[cc@redflag ~]$ ls
[cc@redflag ~]$ touch file1
[cc@redflag ~]$ mkdir dir1
[cc@redflag ~]$ ln file1 file2
[cc@redflag ~]$ ln -s file1 file3
[cc@redflag ~]$ echo test > file1		# 在 file1 里输入 test
[cc@redflag ~]$ rm file1
[cc@redflag ~]$ cat file2
test
[cc@redflag ~]$ cat file3
cat: file3: No such file or directory
[cc@redflag ~]$ ln -s file1 file4		# 为不存在的文件 file1 创建软链接
[cc@redflag ~]$ ln file1 file5			# 为 file1 创建硬链接，失败
ln: failed to access 'file1': No such file or directory
```
链接文件可保证文件的同步性，在删除源文件后，硬链接将不受影响，而软链接则失效；同时，可为一不存在的源文件创建软链接，但不能创建硬链接。

### 2.11 查看文件类型命令 file
不同于 Windows，在 Linux 系统下，不以后缀名“论英雄”，但一般情况下，我们还尽量在给文件命令时加上正确的后缀名，以方便阅读。但若有不确定的文件，则可以用`file`命令。
```
[cc@redflag ~]$ ls										# 确认当前目录为空
[cc@redflag ~]$ touch empty								# 空文件
[cc@redflag ~]$ echo text > word.txt					# 文本文件
[cc@redflag ~]$ tar zcf all.tar.gz word.txt empty		# 压缩文件
[cc@redflag ~]$ cp word.txt fault.tar.gz				# 文本？压缩？
[cc@redflag ~]$ file *
all.tar.gz:   gzip compressed data, last modified: Wed Jun 27 09:52:32 2018, from Unix, original size 10240
empty:        empty
fault.tar.gz: ASCII text
word.txt:     ASCII text
```
上面的示例很简单，也能准确的说明问题，可自行创建或找一些其它文件进行测试，如 /bin 目录下的二进制文件，图片文件、镜像文件等。

### 2.12 
































