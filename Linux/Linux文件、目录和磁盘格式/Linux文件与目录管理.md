# 1 目录与路径

## 1.1 目录的相关操作
- ```.```代表此层目录
- ```..```代表上一层目录
- ```-```代表前一个工作目录
- ```~```代表目前用户身份所在的主文件夹
- ```~account```代表account这个用户的主文件夹

|目录的相关操作|含义|
|-|-|
|```cd```|切换目录|
|```pwd```|显示当前目录|
|```mkdir```|新建一个新的目录|
|```rmdir```|删除一个空的目录|

**<font size = 5 color =red>```cd```</font>**

    [root@www ~]#cd [相对路径或者绝对路径]
    [root@www ~]#cd ~qintian    <== 代表去到用户qintian的主文件夹
    [root@www qintian]#cd ~     <== 表示回到自己的主文件夹
    [root@www ~]#cd ..          <== 表示/root的上层目录
    [root@www /]#cd -           <== 表示回到刚才的目录
    [root@www ~]#cd /var/spool/mail
    [root@www mail]#cd ../mqueue

**注意:仅输入```cd```时，代表的就是```cd ~```d的意思**

**<font size = 5 color =red>```pwd```</font>**

    [root@www ~]#pwd 
    /root


**<font size = 5 color =red>```mkdir```</font>**

    [root@www ~]#mkdir [-mp] 目录名称
    [root@www ~]#cd /tmp
    [root@www tmp]#mkdir test
    [root@www ~tmp#mkdir -p test1/test2/test3    <==加上-p可以创建多层目录，即使已经存在该目录也不会报错
    [root@www tmp]#mkdir -m 711 test2            <==创建一个权限为rwx--x--x的目录


**<font size = 5 color =red>```rmdir```</font>**

    [root@www ~]# rmdir [-p] 目录名称  <==连同上层的空的目录一起删除
    [root@www tmp]# rmdir test        <==test为空，可以直接删除
    [root@www tmp]# rmdir test1       <==test1不为空，无法删除
    rmdir: 'test1': Directory not empty
    [root@www tmp]# rmdir -p test1/test2/test3   <==直接删除test1删除不掉，因为test1目录非空，使用-p可以删除上层的空目录

将目录下的所有的我东西都删除可以使用```rm -r 目录名称```

## 1.2 执行文件路径的变量:```$PATH```

> PATH：环境变量，当执行某一个命令如```ls```时，系统会依照PATH的设置去每个PATH定义的目录下查询文件名为ls的可执行文件，如果在PATH定义的目录中含有多个文件名为ls的可执行文件，则先查询到的同名文件先被执行。

    [root@www ~]# echo $PATH
    /usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin/:/usr/bin:/root/bin
    [root@www ~]# su - qintian
    [root@www ~]# echo $PATH
    /usr/kerberos/bin:/usr/local/bin:/bin:/usr/bin:/home/qintian/bin      <==一般用户的PATH中，不包含任何sbin目录

```echo```命令有打印、显示的意思；```PATH```变量由一堆目录组成，每个目录之间用冒号隔开。无论是```root```还是```qintian```都有```/bin```这个目录在```PATH```中，所以在任何地方都可以执行```ls```来找到```/bin/ls```执行文件。当一般用户要执行的命令位于```/sbin```中，直接执行会找不到命令，可以使用绝对路径来执行该命令，如```/sbin/ifconfig eth0```。

    [root@www ~]# mv /bin/ls /root      <== 将/bin/ls移动成为/root/ls
    [root@www ~]# ls
    ls: Commond not found               <== 无法顺利执行，因为/root目录不在PATH指定的目录中
    [root@www ~]# /root/ls              <== 使用绝对路径指定该文件名
    ···
    [root@www ~]# ./ls                  <== 因为在/root目录下，使用./ls来指定
    ···
    [root@www ~]# PATH="$PATH":/root    <== 将/root加入PATH中，可以通过echo $PATH查看
    [root@www ~]# ls
    ···

- 不同身份用户的```PATH```不同，默认能够执行的命令也不同
- ```PATH```是可以修改的
- 使用绝对路径或者相对路径来指定某个命令的文件名来执行，比查询```PATH```来的正确
- 命令应该被放在正确的目录下，执行才会方便
- 本目录```.```为了安全起见最好不要放到```PATH```中



# 2 目录的相关操作

|命令|命令含义|
|-|-|
|```ls```|查看文件与目录|
|```cp```|复制|
|```rm```|删除|
|```mv```|移动|

**<font size = 5 color =red>```ls```</font>**
该命令比较常见，列出几个常用的参数
- ```-a``` :全部的文件，包括隐藏文件
- ```-d``` :仅列出目录本身，而不是列出目录内的文件数据
- ```-l``` :列出长数据串，包含文件的属性与权限等数据
- ```-t``` :按时间排序

```ls -l```最常用，很多distribution将```ll```设置成```ls -l```的意思

**<font size = 5 color =red>```cp```</font>**

```-i```

    [root@www ~]# cp [-参数] 源文件 目标文件
    [root@www ~]# cp [options] source1 source2 ... directory

    [root@www ~]# cp ~/.bashrc /tmp/bashrc
    [root@www ~]# cp -i ~/.bashrc /tmp/bashrc
    cp:overwrite '/tmp/bashrc'? n          <== n,不覆盖，y，覆盖

```-a```

    [root@www ~]# cd /tmp
    [root@www root]#  cp /var/log/wtmp .   <== 复制到当前目录
    [root@www ~]# ls -l /var/log/wtmp wtmp
    -rw-rw-r-- 1 root utmp 4096 Sep 24 11:54 /var/log/wtmp
    -rw-r--r-- 1 root root 4096 Sep 24 14:06 wtmp
    #不加任何参数，文件的某些属性和权限会发生改变，如果想要所有的特性一起复制过来，则使用-a
    [root@www tmp]# cp -a /var/log/wtmp wtmp_2
    [root@www tmp]# ls -l /var/log/wtmp wtmp_2
    -rw-rw-r-- 1 root utmp 4096 Sep 24 11:54 /var/log/wtmp
    -rw-rw-r-- 1 root utmp 4096 Sep 24 11:54 wtmp
    #整个数据特性完全一致就是-a的特性

```-r```

    [root@www tmp]# cp /etc/ /tmp 
    cp: omitting directory '/etc'      <== 如果是目录不能复制，需要加上-r的参数
    [root@www tmp]# cp -r /etc/ /tmp 

将多个文件一次复制到同一个目录下

    [root@www tmp]# cp ~/.bashrc ~/.bash_history /tmp
    

在默认的条件中，```cp```的源文件与目标文件的权限是不同的，目的文件的所有者通常是命令操作者本身。

下面列出常用的参数

- ```-a```:相当于-pdr的意思
- ```-i```:若目标文件已经存在，则在覆盖时会先进行询问操作
- ```-r```:递归进行复制，用于目录的复制行为

**<font size = 5 color =red>```rm```</font>**

    [root@www ~]# rm [-参数] 文件或者目录

- ```-f```:force,忽略不存在的文件，不会出现告警信息
- ```-i```:互动模式，在删除之前会询问用户是否操作
- ```-r```:递归删除，最常用在目录的删除，是一个危险的参数

```-i```

    [root@www ~]# cd /tmp
    [root@www tmp]# rm -i bashrc
    rm: remove regular file 'bashrc'? y     <== 加上-i则会主动询问，避免错删
    [root@www tmp]# rm -i bashrc*           <== 加通配符表示删除/tmp中所有bashrc开头的文件

```-r```

    [root@www tmp]# rmdir /tmp/etc
    rmdir: etc: Directory not empty
    [root@www tmp]# rm -r /tmp/etc
    rm: descend into directory '/tmp/etc'?y    <== root身份，默认加入了-i的参数，会询问
    [root@www tmp]# \rm -r /tmp/etc            <== 不会询问

删除一个带有```-```开头的文件

    [root@www tmp]# rm -aaa-
    Try 'rm --help' for more information
    [root@www tmp]# rm ./-aaa-

**<font size = 5 color =red>```mv```</font>**

    [root@www ~]# mv [-参数] source destination
    [root@www ~]# mv [-参数] source1 source2 source3 ... directory

    #复制一个文件，创建一个目录。将文件移动到目录中
    [root@www ~]# cd /tmp
    [root@www tmp]# cp ~/.bashrc bashrc
    [root@www tmp]# mkdir mvtest
    [root@www tmp]# mv bashrc mvtest

    #将刚才的目录重命名
    [root@www tmp]# mv mvtest mvtest2

    #创建两个文件，再全部移动到/tmp/mvtest2中
    [root@www tmp]# cp ~/.bashrc bashrc1
    [root@www tmp]# cp ~/.bashrc bashrc2
    [root@www tmp]# mv bashrc1 bashrc2 mvtest2


- ```-f```:force，强制，如果目标文件已经存在，不会询问，直接覆盖
- ```-i```:若目标文件已经存在，则会询问是否覆盖
- ```-u```:若目标文件存在，且source比较新，才会更新

```basename```、```dirname```

这两个命令用于获取文件名和目录名

    [root@www ~]# basename /etc/sysconfig/network       <== 获取文件名
    network                             
    [root@www ~]# dirname /etc/sysconfig/network        <== 获取目录名
    /etc/sysconfig                          

# 3 文件内容查阅

|命令|含义|
|-|-|
|```cat```|从第一行开始显示文件内容|
|```tac```|从最后一行开始显示文件内容|
|```nl```|显示的时候，顺便输出行号|
|```more```|一页一页的显示内容|
|```less```|与```more```类似，可以往前翻页|
|```head```|只看头几行|
|```tail```|只看尾几行|
|```od```|以二进制的方式读取文件|

## 3.1 直接查看文件内容```cat```、```tac```、```nl```

**<font size = 5 color =red>```cat```</font>**

    [root@www ~]# cat [-参数] 文件
    
    #查看/etc/issue这个文件的内容
    [root@www ~]# cat /etc/issue
    ......
    ......
    
    #打印行号
    [root@www ~]# cat -n /etc/issue
    1 ......
    2 ......
    
    #将完整内容显示出来(包含特殊字符)
    [root@www ~]# cat -A /etc/xinetd.conf
    #$
    ......
    $

- ```-A```:-vET的整合参数，显示出如```$```、```[Tab]```(以^I显示),以及一些其他的特殊字符
- ```-b```:列出行号，仅针对非空白行
- ```-n```:打印出行号，空白也会有行号

**<font size = 5 color =red>```tac```</font>**

    [root@www ~]# tac /etc/issue            <== 最后一行先显示
    ......                          

**<font size = 5 color =red>```nl```</font>**

    [root@www ~]# nl [-参数] 文件

参数说明
- ```-b```:指定行号指定的方式
  - ```-b a```:表示不论是否为空行，均列出行号，类似于```cat -n```
  - ```-b t```:如果有空行，空的行不列出行号
- ```-n```:列出行号表示的的方法，主要有三种
  - ```-n ln```:行号在屏幕的最左边显示
  - ```-n rn```:行号在自己字段的最右边显示，且不加0
  - ```-n rz```:行号在自己字段的最右边显示，且加0
- ```-w```:行号字段占用的位数.

示例

    [root@www ~]# nl /etc/issue
    1 ......
    2 ......

    #文件有三行，第三行为空白，所以不显示行号


    [root@www ~]# nl -b a /etc/issue
    1 ......
    2 ......
    3

    [root@www ~]# nl -b a -n rz /etc/issue
    000001  ......
    000002  ......
    000003  

    [root@www ~]# nl -b a -n rz -w 3 /etc/issue
    001 ......  
    002 ......
    003

```nl```可以将输出的文件内容自动加上行号，空白行显示行号，这个与```cat -n```不一样

## 3.2 可翻页查看`more`，`less`

**<font size = 5 color =red>```more```</font>**

    [root@www ~]# more /etc/man.config
    # ......
    # ......
    # ......
    ...(中间省略)...
    --More--(%28)|               <== 光标停在这里

最后一行显示的是目前文件内容显示的百分比，
光标处可以执行以下的命令

- 空格键：代表向下翻页
- 回车键：代表向下一行
- /字符串：代表向下查询“字符串”这个关键字
- ```:f```:立刻显示出文件名以及目前显示的行数
- ```q```:代表立刻离开```more```,不在显示该文件内容
- ```b```:代表往回翻页，只对文件有效，对管道无用


**<font size = 5 color =red>```less```</font>**

    [root@www ~]# less /etc/man.config
    # ......
    # ......
    # ......
    ...(中间省略)...
    :|               <== 等待输入命令

- 空格键：向下翻动一页
- ```[PageDown]```:向下翻动一页 
- ```[PageUp]```:向上翻动一页
- ```/字符串```：向下查询字符串
- ```?字符串```：向上查询字符串
- ```n```：重复前一个查询
- ```N```：反向重复前一个查询
- `q`：离开`less`

## 3.3 数据选取 `head`，`tail`

**<font size = 5 color =red>```head```</font>**

    [root@www ~]# head [-参数] 文件

- `-n`:表示显示几行

示例

    #默认显示10行
    [root@www ~]# head /etc/man.config
    
    #显示前20行
    [root@www ~]# head -n 20 /etc/man.config

    #接负数
    [root@www ~]# head -n -100 /etc/man.config      <== 表示列出前面所有的行数，但是不包括后面的100行

**<font size = 5 color =red>```tail```</font>**

    [root@www ~]# tail [-n number] 文件

- `-n`:后面接数字，代表显示几行的意思
- `-f`:表示持续检测后面所接的文件名，要按下`Ctrl+C`才能结束`tail`的检测

示例

    #默认显示后10行
    [root@www ~]# tail /etc/man.config

    #显示最后的20行
    [root@www ~]# tail -n 20 /etc/man.config

    #不知道文件有多少行，只想列出100行以后的数据
    [root@www ~]# tail -n +100 /etc/man.config

    #持续检测 /var/log/message的内容
    [root@www ~]# tail -f /var/log/message

    #显示文件的第11行到第20行
    [root@www ~]# head -n 20 /etc/man.config|tail -n 10     <== 先读前20行，再取后10行

## 3.4 非纯文本文件`od`

上面的文件内容查阅，查阅的都是纯文本文件的内容，对于一些二进制文件，如果使用上面的命令查看，则会产生类似乱码的数据，可以使用`od`这个命令来读取

**<font size = 5 color =red>```od```</font>**

    [root@www ~]# od [-t type] 文件

`-t`:后面可以接type来输出
- `a`:利用默认的字符来输出
- `c`:使用ASCII字符来输出
- `d[size]`:利用十进制来输出数据，每个整数占用size bytes
- `f[size]`:利用浮点数来输出数据，每个数占用size bytes
- `o[size]`:利用八进制来输出数据，每个整数占用size bytes
- `x[size]`:利用十六进制来输出数据，每个整数占用size bytes

示例
    
    #用ASCII码方式来输出
    [root@www ~]# od -t c /usr/bin/password
    0000000 177 E  L   ...
    0000020 002 \0 003 ...                      <== 最左边第一列是以进制来表示bytes，第2列0000020代表开头是第16个bytes8的内容之意

    #将文件内容以八进制列出存储值和ASCII的对照表
    [root@www ~]# od -t oCc /etc/issue
    0000000 103 145 156 164 ...
            C   e   n   t   ...
    0000020 056 062 040 050 ...
            .   2     ( F   ...

## 3.5 修改文件时间或者创建新文件 `touch`

通过`ls`命令加不同的参数可以获取时间，一共有三个时间，分别为

- 修改时间(mtime)：当文件的内容数据被修改时，会更新这个时间，直接`ls -l 文件名`
- 状态时间(ctime)：当权限与属性被更改了，会更新这个时间,`ls -l --time=ctime`
- 读取时间(atime)：当文件的内容被取用时，就会更新这个时间，如使用`cat 文件名`，就会更改这个时间,`ls -l --time=atime`


使用`touch`命令可以修改时间

    [root@www ~]# touch [-参数] 文件
  - `-a`：仅修改访问时间
  - `-c`：仅修改文件的时间，若文件不存在则不创建新文件
  - `-d`：后面可以接欲修改的时间而不用目前的日期
  - `-m`：仅修改mtime
  - `-t`：后面可以接欲修改的时间而不用目前的时间，格式为[YYMMDDhhmm]

示例

    #创建一个空的文件并查看时间
    [root@www ~]# cd /tmp
    [root@www tmp]# touch testtouch
    [root@www tmp]# ls -l testtouch
    -rw-r--r-- 1 root root 0 Sep 25 21:09 testtouch         <== 在默认状态下，如果touch后面接文件，则该文件的三个时间都会更新成目前的时间

    #将~/.bashrc复制成为bashrc，假设复制完全的属性，查看日期
    [root@www tmp]# cp -a ~/.bashrc bashrc
    [root@www tmp]# ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc       <== 使用;表示连续命令的执行
    -rw-r--r-- 1 root root 176 Jan  6 2007   <== mtime，与源文件相同
    -rw-r--r-- 1 root root 176 Sep 25 21:09   <== atime，刚被创建，是现在的时间
    -rw-r--r-- 1 root root 176 Sep 25 21:09   <== ctime， 刚被创建，是现在的时间

    #修改上面的bashrc的文件，时间调整为2天前
    [root@www tmp]# touch -d "2 days ago" bashrc
    [root@www tmp]# ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc
    -rw-r--r-- 1 root root 176 Sep 23 21:23
    -rw-r--r-- 1 root root 176 Sep 23 21:23
    -rw-r--r-- 1 root root 176 Sep 25 21:10

    #修改上面的bashrc文件，时间调整为2007/09/15/ 2:02
    [root@www tmp]# touch -t 0709150202 bashrc
    [root@www tmp]# ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc
    -rw-r--r-- 1 root root 176 Sep 15 2007
    -rw-r--r-- 1 root root 176 Sep 15 2007
    -rw-r--r-- 1 root root 176 Sep 25 21:11

**`touch`命令常被用于创建一个空文件，将某个文件日期修改为当前日期**

# 4 文件与目录的默认权限和隐藏权限

## 4.1

关于文件默认权限、隐藏权限和特殊权限，可以查阅书籍《鸟哥的Linux私房菜》[P180-186]进行了解

## 4.2 查看文件类型`file`
**<font size = 5 color =red>```file```</font>**

    [root@www ~]# file ~/.bashrc
    /root/.bashrc: ASCII text           <== ASCII纯文本文件
    [root@www ~]# file /var/lib/mlocate/mlocate.db
    /var/lib/mlocate/mlocate.db         <== data文件
    
# 5 命令与文件的查询
某些文件的文件名没变，但是不同的distribution放在不同的目录下，需要查找到，才能对这些文件进行修改。

## 5.1 脚本文件名的查询

**<font size = 5 color =red>```which```</font>**

    [root@www ~]# which [-a] command

- `-a`:将所有由PATH目录中可以找到的命令均列出，而不是第一个被找到的命令名称

示例

    #用root和一般账号查询ifconfig这个命令的完整文件名
    [root@www ~]# which ifconfig
    /sbin/ifconfig
    [root@www ~]# su - qintian
    [qintian@www ~]$ which ifconfig
    /usr/bin/which: no ifconfig in (/usr/kerberos/bin:/usr/local:/bin:/usr/bin:/home/qintian/bing)      <== 因为which是根据用户设置的PATH变量内的目录去查找可执行文件的，所以，不同的PATH设置内容找到的命令会不一样，因为/sbin不在qintian的PATH中，所以找不到


## 5.2 文件名的查找

**<font size = 5 color =red>```whereis```</font>**

寻找特定文件

Linux会将所有的系统的文件都记录在一个数据库文件中，使用whereis或者locate时会以数据库文件的内容为准，因此，速度很快，还有可能会找到被删除掉的文件，找不到最新的刚才创建的文件。

 
    [root@www ~]# whereis [-参数] 文件或目录名

- `-b`:只寻找二进制文件
- `-m`:只找在说明文件manual路径下的文件
- `-s`:只找source文件
- `-u`:查找上述三个选项之外的其他特殊文件

示例

    #使用不同的身份找出ifconfig这个文件
    [root@www ~]# whereis ifconfig
    ifconfig: /sbin/ifconfig /usr/share/man/man8/ifconfig.8.gz
    [root@www ~]# su - qintian
    [qintian@www ~]$ whereis ifconfig
    ifconfig: /sbin/ifconfig /usr/share/man/man8/ifconfig.8.gz
    [qintian@www ~]$ exit                   <== 切换身份为root

`which`一般用户找不到的`ifconfig`可以让`whereis`找到，因为系统真的有`ifconfig`，但是用户的`PATH`没有加入`/sbin`

    #只找出与passwd有关的“说明文件”文件名
    [root@www ~]# whereis -m passwd
    passwd: /usr/share/man/man1/passwd.1.gz /usr/share/man/man5/passwd.5.gz


**<font size = 5 color =red>```locate```</font>**

    [root@www ~]# locate [-参数] 关键字

- `-i`:忽略大小写的差异
- `-r`:后面可以接正则表达式的显示方式

示例

    #找出系统中所有的passwd的相关的文件名
    [root@www ~]# locate passwd
    /etc/passwd
    /etc/passwd-
    /etc/news/passwd.nntp
    ...(中间省略)...


`locate`也是通过查找`/var/lib/mlocate`里的数据库文件，在新建文件之后需要更新数据库才可以查到，因为不同的distribution更新数据库的频率是不同的。更新数据库的命令为`updatadb`，这个命令会去读取`/etc/updatedb.conf`这个配置文件的设置，然后再去硬盘里面进行查找文件名的操作，最后就更新位于`/var/lib/mlocate`数据库文件。

**<font size = 5 color =red>```find```</font>**

    [root@www ~]# find [PATH] [option] [action]

- 与时间有关的参数，共有`-atime`，`-ctime`，`-mtime`,以`-mtime`说明
  - `-mtime n` :`n`为数字，代表为在`n`天之前的“一天之内”被更改过的文件
  - `-mtime +n`:列出在`n`天之前(不含`n`天本身)被更改过的文件
  - `-mtime -n`:列出在`n`天之内(含`n`天本身)被更改过的文件
  - `-new file`:`file`为一个存在的文件，列出比`file`还要新的文件名

示例

    #将过去系统上面24小时内有改动(mtime)的文件列出
    [root@www ~]# find / -mtime 0
    #0代表为目前的时间，也即从现在开始到24前，有改动过内容的文件都会被列出来

    #将3天前的24小时有改动的文件列出来
    [root@www ~]# find / -mtime 3

    #寻找/etc下面的文件，如果文件日期比/etc/passwd列出
    [root@www ~]# find /etc -newer /etc/passwd


还有一些拓展的使用方法，可以查阅数据《鸟哥的Linux私房菜》[Page190-192]


