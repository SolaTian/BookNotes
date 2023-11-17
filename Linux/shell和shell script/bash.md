# 1、认识`bash`这个`shell`

管理计算机硬件的其实是Linux的内核，内核需要被保护，一般用户只能通过，`shell`来跟内核通信。

## 1.1 硬件、内核与`shell`

只要有操作系统，就离不开`shell`，用户可以通过应用程序来指挥内核，让内核完成我们的硬件任务。应用程序是在最外层，如同鸡蛋外壳，被称为`shell`。

`shell`的功能只是提供用户操作系统的一个接口，`shell`需要可以直接调用其他的软件才可以，之前学过的很多命令如`ls`，`man`，`chmod`等命令，都是独立的应用程序，可以通过`shell`来操作这些应用程序。

也即，只要能操作应用程序的接口都能称为`shell`。狭义的`shell`指的是命令行方面的软件，包括本章要介绍的`bash`，广义的`shell`包括图形界面的软件，图形界面其实也包括各种应用程序来调用内核进行工作。

## 1.2 命令行界面的`shell`


## 1.3 系统的合法`shell`与`/etc/shells`功能

可以检查`/etc/shell`这个文件，查看Linux支持的`shell`,Linux默认的`shell`就是`/bin/bash`。`bash`兼容`sh`

## 1.4 `bash shell`的内置命令:`type`

为了方便`shell`的操作，其实`bash`已经内置了很多命令，如`cd`,`umask`等命令。可以通过`type`命令查看哪些是`bash`内置命令，哪些是外部命令。

    [root@www ~]# type [-tpa] name

- `type`:不加任何参数时，会显示出`name`是外部命令还是内部命令。
- `-t`:`type`将`name`以一下这些字眼显示出它的意义:
  - `file`:表示为外部命令
  - `alias`:表示该命令为命令别名所设置的名称
  - `builtin`:表示该命令为`bash`内置的命令功能
- `-p`:如果后面的`name`是外部命令，则显示完整的文件名
- `-a`:会由`PATH`变量定义的路径中，将所有含有`name`的命令都列出来，包含`alias`


示例

    [root@www ~]# type ls
    ls is aliased to `ls --color=tty`       <== 未加任何参数，列出ls的最主要使用情况

    [root@www ~]# type -t ls
    alias                                   <== 仅列出ls执行时的依据

    [root@www ~]# type -a ls
    ls is aliasd to `ls --color=tty`        <== 最先使用alias
    ls is /bin/ls                           <== 还有找到外部命令在/bin/ls

    [root@www ~]# type cd
    cd is a shell builtin                   <== cd是shell的内置命令

当我们在终端机上登录之后，Linux就会依据`/etc/passwd`文件的设置给我们一个`shell`，然后就可以执行`shell`。

# 2、`shell`的变量功能

## 2.1 变量的显示与设置:`echo`,`uset`

**<font size = 3 color =red>变量的显示`echo`</font>**

变量的显示

    [root@www ~]# echo $variable
    [root@www ~]# echo $PATH
    [root@www ~]# echo ${PATH}
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

设置或者修改变量内容

    [root@www ~]# echo $myname
                                        <== 变量未被设置，为空
    [root@www ~]# myname=qintian
    [root@www ~]# echo $myname
    qintian

变量的设置规则

1. 变量与变量内容以一个等号连接
2. 等号两边不能直接接空格符
3. 变量名只能是英文字母或者数字，开头不可以是数字
4. 变量内容如果有空格，则通过双引号`"`或者单引号`'`将变量内容结合起来。
   1. 双引号内的特殊字符如`$`等，可以保持原有特性，如`myname="lang is $LANG"`，则`echo $myname`可以得到`lang is en_US`;
   2. 单引号内的特殊字符则仅为一般字符(纯文本)，如`myname='lang is $LANG'`则`echo myname`可以得到`lang is $LANG`
5. 可以用转义字符`\`将特殊符号如`Enter`,`$`,`\`等变成一般字符,如`\+[space]`可以实现空格符的转义，实现变量内容中含有空格。
6. 在一串命令中，还需要通过其他的命令提供的信息，可以使用`反单引号`或者`$(命令)`，如想要取得内核版本的设置。`version=$(uname-r)`，再`echo $version`可以得到`2.6.18-128.el5`
7. 若该变量为了增加变量内容，可以用`$变量名称`，`"$变量名称"`或者`${变量}累加内容`，如`PATH="$PATH":/home/bin`
8. 若该变量需要在其他子进程执行，则需要以`export`来使变量成为环境变量，如`export PATH`
9. 通常大写字符为系统默认变量，自行设置变量可以使用小写字符
10. 取消变量的方法为使用`unset 变量名称`，如`unset myname`


> 子进程：在目前的`shell`下，打开另外一个新的`shell`，新的`shell`就是子进程。一般情况下，父进程的自定义变量无法在子进程中使用，但是通过`export`将变量变成环境变量之后，就可以在子进程中使用。

## 2.2 环境变量的功能

**<font size = 3 color =red>使用`env`查看环境变量和常见环境变量说明</font>**

使用`env`即可以列出所有的环境变量，使用`export`也可以，不过`export`还有其他的功能。

|环境变量|功能|
|-|-|
|`HOME`|代表用户的主文件夹|
|`SHELL`|目前环境中使用的`shell`是那个程序的，Linux默认是`/bin/bash`|
|`HISTSIZE`|与历史命令有关，记录曾经执行过的命令的条数|
|`MAIL`|当使用`mail`命令在收信时系统会去读取的邮件信箱文件|
|`PATH`|执行查找文件的路径，目录与目录之间通过`:`分割，由于文件的查找顺序由`PATH`变量的目录来查询，所以目录的顺序也很重要|
|`LANG`|语系数据|
|`RANDOM`|随机数变量，在`/bin/bash`的环境下，`RANDOM`是介于0-32767之间的数|

**<font size = 3 color =red>使用`set`查看所有变量(含环境变量和自定义变量)</font>**

`bash`包含环境变量，一些与`bash`操作接口有关的变量，以及用户自定义的变量。可以使用`set`进行查看，有几个比较重要的变量

|系统内定变量|功能|
|-|-|
|`PS1`(提示符的设置)|命令提示符，每次按下`Enter`执行某个命令，最后要再次出现提示符是，就会主动读取这个变量。`PS1`内显示的是一些特殊的符号：具体参考《鸟哥的Linux私房菜》Page306|
|`$`(关于本`shell`的`ID`)|`$`代表目前这个`shell`的线程代号，即所谓的`PID`，若想要知道`shell`的`PID`，用`echo $$`即可，出现的数字即为`PID`|
|`?`(关于上个执行命令的回传码)|这个变量是上一个执行的命令所回传的值，当我们执行一个命令的时候，这些命令都会回传一个执行后的代码，一般来说，如果成功执行该命令，则会回传一个0，否则会回传一个错误代码|
|`OSTYPE`,`HOSTTYPE`,`MACHTYPE`(主机硬件与内核的等级)|-|

    [root@www ~]# echo $SHELL
    /bin/bash
    [root@www ~]# echo $?
    0
    [root@www ~]# 12name=qintian
    -bash: 12name=qintian: command not found
    [root@www ~]# echo $?
    127


**<font size = 3 color =red>`export`自定义变量转成环境变量</font>**

环境变量和自定义变量的差别就在于该变量是否可以被子进程所引用。

当登录Linux并取得一个`bash`之后，`bash`就是一个独立的进程，接下来在`bash`中执行的任何命令，都是由这个`bash`所衍生出来的，那些被执行的命令就被成为子进程。

子进程会继承父进程的环境变量，但是不会继承父进程的自定义变量。

使用`export`就可以在子进程中使用父进程的自定义变量

    [root@www ~]# export 变量名称

如果`export`没有接变量，会把所有的环境变量显示出来，类似于`env`

## 2.3 影响显示结果的语系变量 

使用`locale`这个命令可以查看Linux支持的语系

    [root@www ~]# locale -a

当`locale`后面不跟参数的时候，就可以以变量的形式显示出这些变量

    [root@www ~]# locale
    LANG=en_US                  <== 主语言的环境
    ...

可以手动设置`LANG`，其他的语系变量就会被这个变量所替代

系统默认的语系定义在`/etc/sysconfig/i18n`这个文件里面

    [root@www ~]# cat /etc/sysconfig/i18n
    LANG="zh_CN.UTF-8"

## 2.4 变量键盘读取、数组与声明

**<font size = 3 color =red>`read`</font>**

    [root@www ~]# read [-pt] 变量

- `-p`:后面可以接提示符
- `-t`:后面可以接等待的秒数

    [root@www ~]# read test
    This is a test                  <== 键盘输入
    [root@www ~]# echo $test
    This is a test

    [root@www ~]# read -p "Please input your name:" -t 30 name
    Please input your name:qintian
    [root@www ~]# echo $name
    qintian

**<font size = 3 color =red>`declare`/`typeset`</font>**

`declare`和`typeset`一样的功能，声明变量的类型

    [root@www ~]# declare [-aixr] 变量名

- `-a`:将后面的变量定义成为数组类型
- `-i`:将后面的变量定义成为整数类型
- `-x`:用法与`export`一样，将后面的变量变成环境变量
- `-r`:将变量设置为`readonly`类型，不可以被更改，也不可以被重设

示例

    [root@www ~]# sum=100+300+50
    [root@www ~]# echo $sum
    100+300+50
    [root@www ~]# declare -i sum=100+300+50
    [root@www ~]# echo $sum
    450

变量类型默认为字符串，若不指定变量类型，则100+300+50是一个字符串，而不是计算式。

`bash`中的数值运算，默认最多仅能达到整数类型。

    #将sum变成环境变量
    [root@www ~]# declare -x sum
    [root@www ~]# export | grep sum
    declare -ix sum="450"


    #让sum变成只读属性，不可以改动
    [root@www ~]# declare -r sum
    [root@www ~]# sum=tesgting
    -bash: sum: readonly variable

    #让sum变成自定义变量
    [root@www ~]# declare +x sum        <== 将-变成+，可以进行取消操作
    [root@www ~]# declare -p sum        <== -p可以单独列出变量的类型
    declare -ir sum="450"               <== 不具有x

**<font size = 3 color =red>数组变量类型</font>**

在`bash`中，数组的设置方式为`var[index]=content`

    [root@www ~]# var[1]="small min"
    [root@www ~]# var[2]="big min"
    [root@www ~]# var[3]="nice min"
    [root@www ~]# echo "${var[1]},{var[2]},{var[3]}"
    small min, big min, nice min

## 2.5 与文件系统及程序的限制关系:`ulimit`

`bash`可以限制用户的某些系统资源，包括打开的文件数量，可以使用的CPU时间，可以使用的内存总量等。使用`ulimit`命令

    [root@www ~]# ulimit [-SHacdfltu] [配额]

- `-H`:hard limit,必定不能超过这个设置的数值
- `-S`:soft limit,超过这个设置的值，将会出现警告
- `-a`:后面不接任何参数，可列出所有的限制额度
- `-c`:当某些进程发生错误时，系统可能将该进程在内存中的信息写成文件，这种文件被称为内核文件(core file)。此为限制每个内核文件的最大容量。
- `-f`:此`shell`能够穿件的最大文件容量，单位为KB

示例

    #列出root的所有限制数据数值
    [root@www ~]# ulimit -a
    ......                      <== 参考《鸟哥的Linux私房菜》Page[312]
    
    #限制用户仅能创建10MB以下的容量的文件
    [root@www ~]# ulimit -f 10240
    [root@www ~]# ulimit -a
    file size  (block, -f) 10240    <== 最大为10240KB
    [root@www ~]# dd if=/dev/zero of=123 bs=1M count=20
    File size limit exceeded        <== 尝试创建20MB的文件，结果失败

想要复原`ulimit`设置的最简单的方法为注销再登录，否则就要重新设置才行，需要注意的是，一般身份用户用`ulimit -f`设置了文件大小，只能继续减小文件容量，不能增加文件容量。

## 2.6 变量内容的删除、替代与替换

**<font size = 3 color =red>变量内容的删除与替换</font>**

    [root@www ~]# path=${PATH}          <== 先用path设置与PATH内容相同
    [root@www ~]# echo $path
    /usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin  
    [root@www ~]# echo ${path#/*kerberos/bin:}      <== 将前两个目录删除
    /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin  


    #删除前面所有的目录，进保留最后一个目录
    [root@www ~]# echo ${path#/*:}
    /usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin  
    [root@www ~]# echo ${path##/*:}
    /root/bin 


`#`表示从变量内容的最前面开始向右删除，且仅删除最短的那个。`*`代表0-无穷多个字符。`##`表示删除最长的那个数据。

在`PATH`这个变量内容中，每个目录都是以`:`隔开的，所以要从头删除目录就是介于斜线`/`到冒号`:`之间的数据，但是`PATH`中不只有一个冒号，所以`#`和`##`就代表最短的和最长的那个。

如果想要从后面删除变量内容，使用`%`符号

    [root@www ~]# echo ${path%:*bin}
    /usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:      <== 删除掉最后一个目录
    [root@www ~]# echo ${path%%:*bin}
    /usr/kerberos/sbin              <== %代表最长的符合字符串


变量的替换

    [root@www ~]# echo ${path/sbin/SBIN}
    /usr/kerberos/SBIN:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

    #替换掉所有内容
    [root@www ~]# echo ${path//sbin/SBIN}
    /usr/kerberos/SBIN:/usr/kerberos/bin:/usr/local/SBIN:/usr/local/bin:/SBIN:/bin:/usr/SBIN:/usr/bin:/root/bin

两个斜线中间的是旧字符串，后面的是新字符串，两条斜线`//`表示所有符合的内容都会被替代


**<font size = 3 color =red>变量的测试与内容替换</font>**

某些时候判断某个变量是否存在，若变量存在则使用即有的设置，若不存在则给予一个常用的设置

    [root@www ~]# echo $username
                            <== 空白，可能不存在，可能空字符串
    [root@www ~]# username=${username-root}
    [root@www ~]# echo $username
    root            <== 因为username没有设置，所以主动给予root
    [root@www ~]# username="qintian"
    [root@www ~]# username=${username-root}
    [root@www ~]# echo $username
    qintian         <== 因为username已经被设置，所以使用旧的设置而不以root代替        


在上面的示例中可以这样理解

    new_var=${old_var-content}

- `new_var`代表新的变量，用来替换旧的变量
- `old_var`代表旧的变量，被测试的选项
- `content`变量的内容，这个部分是在给与未设置变量的内容

还可以在大括号里面加上`:`，表示被测试的变量未被设置或者已经被设置为空字符串是，都能够用后面的`content`去替换与设置。如` new_var=${old_var:-content}`

其他设置方式可以查看《鸟哥的Linux私房菜》Page[316]


# 3、命令别名与历史命令

## 3.1命令别名的设置:`alias`，`unalias`

    #设置别名
    [root@www ~]# alias lm='ls -l | more'
    #取消别名设置
    [root@www ~]# unlias lm

使用方法为，`alias {'别名'='命令+参数'}`,`unlias 别名`

> 清屏命令:clear

## 3.2历史命令`history`

    [root@www ~]# history [n]
    [root@www ~]# history [-c]
    [root@www ~]# history [-raw] histfiles

- `n`:数字，列出最近的n条命令行
- `-c`:将目前`shell`中的所有的`history`都消除
- `-a`:将目前新增的`history`命令新增到`histfiles`中，若没有则加`histfiles`,则默认写入`~/.bash_history`
- `-r`:将`histfiles`的内容读到目前这个`shell`的`history`中
- `-w`:将目前的`history`记忆写入`histfiles`中

示例

    #列出目前内存中的所有history
    [root@www ~]# history
    ...
    1017 man bash
    1018 ll
    1019 history                <== 数字为在shell中的代码，后面为命令本身的内容

    #列出最近3条数据
    [root@www ~]# history 3
    1019 history
    1020 history
    1021 history3

    #立即将目前的数据写入histfiles
    [root@www ~]# history -w        <== 默认情况下，会将历史记录写入~/.bash_history中
    [root@www ~]# echo $HISTSIZE
    1000

正常情况下，系统会主动由主文件夹的`~/.bash_history`读取历史命令，记录的历史命令条数就与`HISTSIZE`有关。


    [root@www ~]# !number
    [root@www ~]# !command
    [root@www ~]# !!

- `number`:执行第几条命令的意思
- `command`:由最近的命令向前搜索命令串开头为`command`的那个命令，并执行
- `!`:执行上一个命令


示例

    [root@www ~]# history
    66 man am
    67 alias
    68 man history
    69 history
    [root@www ~]# !66               <== 执行第66条命令
    [root@www ~]# !!                <== 执行上一个命令
    [root@www ~]# !al               <== 执行最近以al开头的命令


# 4 Bash Shell的操作环境

## 4.1 路径与命令查找顺序

当我们执行一个命令的时候，命令的运行顺序如下

1. 以相对/绝对路径执行命令，例如`/bin/ls`或者`./ls`
2. 由`alias`找到该命令来执行
3. 由`bash`内置的命令来执行
4. 通过`$PATH`这个变量的顺序找到第一个命令来执行

## 4.2 `bash`的登录与欢迎信息，`/etc/issue`,`/etc/motd`

终端登录时会有提示字符串，字符串写在`/etc/issue`里面

    [root@www ~]# cat /etc/issue
    CentOS release 5.3 Final
    Kernal \r on an \m

在`issue`内的各代码的意义

|代码|含义|
|-|-|
|`\d`|本地端时间的日期|
|`\I`|显示第几个终端接口|
|`\m`|显示硬件的等级|
|`\n`|显示主机的网络名称|
|`\o`|显示`domain name`|
|`\r`|操作系统的版本|
|`\t`|显示本地端时间的时间|
|`\s`|操作系统的名称|
|`\v`|操作系统的版本|


除了`/etc/issue`这个文件还有`/etc/issue.net`这个文件，这个文件是提供给`telnet`这个远程登录程序使用的，`telnet`登录主机时显示的是`/etc/issue.net`而不是`/etc/issue`。

如果想要用户登录之后获取一些信息，例如想要大家都知道的信息，那么可以将信息加入/etc/motd里面去，例如告知登录者系统在某个固定的时间进行维护工作。

    [root@www ~]# vi /etc/motd
    Hello everyone,
    Our server will be maintained at 2009/02/28 0:00~24:00
    Please don't login server at that time.

## 4.3 `bash`的环境配置文件

系统存在一些环境配置文件的存在，让`bash`可以在启动的时候直接读取这些配置文件，以规划好`bash`的操作环境，这些配置文件分成全体系统的配置文件以及用户个人偏好配置文件。之前提到的命令的别名，自定义变量在注销`bash`之后就会失效，如果想要保留配置，就需要将这些配置写入配置文件才可以。


<font size = 3 color =red>`login`与`non-login shell`</font>

- `login shell`取得`bash`时需要完整登录流程的，就称为`login shell`
- 取得`bash`接口的方法不需要重复登录的举动，如原本在`bash`环境下，再次执行`bash`无需输入账号和密码，第二个`bash`也是`non-login shell`


<font size = 3 color =red>`/etc/profile`(login shell才会读)</font>

每个用户登录取得`bash`时一定会读取的配置文件,要帮助所有用户设置整体环境，就在这里修改，不过没有事不要随便修改这个文件。

这个文件设置的变量如下

- `PATH`:会依据用户的标识符(UID)决定`PATH`变量要不要含有`sbin`的系统命令目录
- `MAIL`:依据账号设置好用户的`mailbox`到`/var/spool/mail/账号名`
- `USER`:根据用户的账号设置此变量内容
- `HOSTNAME`:依据主机的`hostname`命令决定此变量的内容
- `HISTSIZE`:历史命令记录条数

`/etc/profile`除了上面的变量设置，还会调用外部的设置数据。以下的数据会依序被调用进来

<font size = 3 color =red>`/etc/inputrc`</font>

`/etc/profile`会主动判断用户有没有自定义输入的按键功能，如果没有，则`/etc/profile`就会决定设置`INPUTRC=/etc/inputrc`这个变量，此文件内容为热键，`tab`键是否有声音等数据。一般这个文件也不建议修改。


<font size = 3 color =red>`/etc/profile.d/*.sh`</font>

该目录下的扩展名为`.sh`的文件，且用户具有`r`的权限,就会被`/etc/profile`调用

<font size = 3 color =red>`~/.bash_profile`(login shell才会调用)</font>

`bash`在读完整体环境设置的`/etc/profile`并借此调用其他配置文件后，接下来会读取用户的个人配置文件，在`login shell`的`bash`环境中，所读取的个人偏好配置文件其实有三个，`~/.bash_profile`，`~/.bash_login`，`~/.profile`

`bash`的`login shell`只会读取其中一个，读取的顺序是按照上面的顺序去读。

<font size = 3 color =red>`source`读入环境配置文件的命令</font>

`/etc/profile`与`~/.bash_profile`都是在取得`login shell`之后才会读取的配置文件，所以将自己的偏好设置写入上述文件之后，通常需要注销再登录才会生效。使用`source 配置文件名`即可将刚才最新设置的内容读进目前的环境中。

<font size = 3 color =red>`~/.bashrc`（non-login shell会读）</font>

当取得`non login shell`时，该`bash`配置文件仅会读取`~/.bashrc`

<font size = 3 color =red>其他相关配置文件</font>

`/etc/man.config`这个文件规定了使用`man`的时候，`man page`的路径去哪里寻找。即这个文件规定了执行`man`的时候该去哪里查看数据的路径设置。

<font size = 3 color =red>`~/.bash_history`</font>

默认情况下，我们的历史命令就记录在这里，记录的条数与`HISTSIZE`有关。

<font size = 3 color =red>`~/.bash_logout`</font>

当注销`bash`之后系统帮我做完什么操作再离开。


## 4.4 终端的环境设置:`stty`，`set`

`stty`列出终端所有的按键和按键内容

    [root@www ~]# stty -a
    ...
    intr = ^C;...;erase = ^?;...;eof = ^D...
    ...

`-a`表示将目前所有的参数列出来

`^`表示`Ctrl`

`intr`:发出一个中断信号给当前正在运行的程序
`erase`:向后删除字符
`eof`:代表结束输入`End of file`
`quit`:送出一个`quit`信号给目前正在运行的程序
`stop`:停止目前屏幕的输出

也可以设置相关的热键，如下设置`erase`

    [root@www ~]# stty erase ^h

除了使用`stty`外，还可以利用`set`来设置自己的一些终端机设置值

    #显示目前所有的set设置值
    [root@www ~]# echo $-
    himBH
    #$- 就是set的所有设置，bash默认的是himBH

详细参考《鸟哥的Linux私房菜》[Page326]


## 4.5 通配符和特殊符号

|通配符|说明|
|-|-|
|`*`|代表0-无穷多个任意字符|
|`?`|代表一定有一个任意字符|
|`[]`|代表一定有一个在括号里面的字符，例如[abcd]代表一定有一个字符，是a,b,c,d中任意一个|
|`[-]`|代表在编码顺序内的所有字符，如[0-9]代表从0-9的所有数字|
|`[^]`|代表原向选择，如[^abc]代表一定有一个字符，只要是非a,b,c,的其他字符就接受的意思|

    [root@www ~]# ll -d /etc/cron*          <== cron开头的文件名
    [root@www ~]# ll -d /etc/?????          <== 文件名5个字母的文件
    [root@www ~]# ll -d /etc/*[0-9]*        <== 文件名中含有数字的文件
    [root@www ~]# ll -d /etc/[^a-z]*        <== 文件名开头非小写字母的文件

|特殊符号|说明|
|-|-|
|`#`|批注符号，其后面的数据均不执行|
|`\`|转义符号，将特殊字符或者通配符还原成一般字符|
|`\|`|管道，分隔两个管道命令|
|`;`|连续执行命令分隔符，与管道命令有区别|
|`~`|用户的主文件夹|
|`$`|变量前导符，即变量之前需要加的变量代替值|
|`/`|目录符号，路径分隔的符号|
|`>`,`>>`|数据流重定向，输出导向，分别是替换和累加|
|`<`,`<<`|数据流重定向，输入导向|

# 5 数据流重定向

> 数据流重定向:将某个命令执行后应该要出现在屏幕上的数据传输到其他的地方，例如文件或者是设备。

## 5.1 什么是数据流重定向

<font size = 3 color =red>`standard output`,`standard error`</font>

当执行一个命令的时候，命令可能会由文件读入数据，经过处理之后，再将数据输出到屏幕上，`standard output`和`standard error output`代表的是`标准输出`和`标准错误输出`，不管是正确的或者错误的数据都默认输出到屏幕上，屏幕会很混乱。


可以通过数据流重定向将`stdout`和`stderr`分别传送到其他的文件或者设备去。

- 标准输入`stdin`:代码为0，使用`<`或者`<<`
- 标准输出`stdout`:代码为1，使用`>`或者`>>`
- 标准错误输出`stderr`:代码为2，使用`2>`或者`2>>`

示例

    [root@www ~]# ll / > ~/rootfile         <== 屏幕无任何信息
    [root@www ~]# ll ~/rootfile             <== 有个新文件被创建了
    -rw-r--r-- 1 root root 1089 Feb 6 17:00 /root/rootfile

上述示例将`ll /`的输出重定向到`~/rootfile`中，如果文件不存在，则会将它创建起来，如果存在，则就会将该文件的内容进行清空，再将数据写入。如果想要追加，可以使用`>>`。

示例

    #以一般身份账号查找/home下的.bashrc文件
    [root@www ~]# su - qintian
    [qintian@www ~]$ find /home -name .bashrc
    find /home/lost+found: Permission denied
    ...
    [qintian@www ~]$ find /home -name .bashrc > list_right 2> list_error            <== 屏幕上不会出现信息

上述示例中，正确的输出数据会存储到`list_right`中，错误的输出数据会存储到`list_error`中。

<font size = 3 color =red>`/dev/null`垃圾桶黑洞设备与特殊写法</font>

`/dev/null`可以吃掉任何导向这个设备的信息。当需要忽略错误信息或存储时。

示例

    #接上一个示例，将错误的数据丢弃，屏幕上显示正确的数据
    [qintian@www ~]$ find /home -name .bashrc 2> /dev/null
    /home/qintian/.bashrc       <== 屏幕上只有stdout,没有stderr

    #将正确和错误的消息写到同一个文件list
    [qintian@www ~]$ find /home -name .bashrc > list 2>&1                       <== 正确
    [qintian@www ~]$ find /home -name .bashrc &> list           <== 正确

<font size = 3 color =red>`standard input`</font>

将原本需要由键盘输入的数据改由文件内容替代。

示例

    #利用cat命令来创建一个文件的简单流程
    [root@www ~]# cat > catfile
    testing
    cat file test
    <== 按下Ctrl+d

    [root@www ~]# cat catfile
    testing
    cat file test          <== catfile会被主动创建，文件内容就是刚才键盘上输入的两行

    #用stdin替代键盘的输入以创建新文件的简单流程
    [root@www ~]# cat > catfile < ~/.bashrc
    [root@www ~]# ll catfile ~/.bashrc
    -rw-r--r-- 1 root root 194 Sep 26 13:36 /root/.bashrc
    -rw-r--r-- 1 root root 194 Feb 6  18:29 catfile

`<<`代表结束输入。

    [root@www ~]# cat > catfile << "eof"
    > This is a test
    > OK now stop
    > eof                   <== 输入这个关键字就立刻结束，而不用输入Ctrl+d
    [root@www ~]# cat catfile 
    This is a test
    OK now stop             <== 只有这两行，不会有关键字那一行


要用`cat`直接将输入的信息输出到`catfile`中，且当由键盘输入`eof`时，该次输入就结束。



## 5.2 命令执行的判断依据:`;`,`,`,`&&`,`||`

<font size = 3 color =red>`cmd;cmd`(不考虑命令的相关性的连续执行)</font>

    [root@www ~]# sync;sync;shuntdown -h now

<font size = 3 color =red>`$?(命令回传码)`与`&&`或`||`(两个命令之间存在相关性)</font>

如在某个目录下新建一个文件，也即目录存在的话，才新建这个文件，如果不存在，就不进行操作。这个相关性的主要判断的地方就在于前一个命令的执行结果是否正确。

*若前一个命令执行的结果为正确，在Linux下会回传一个`$?=0`的值。*

|命令执行情况|说明|
|-|-|
|`cmd1&&cmd2`|<li>若`cmd1`执行完毕且正确执行，则开始执行`cmd2`<li>若`cmd1`执行完毕且为错误，则`cmd2`不执行|
|`cmd1\|\|cmd2`|<li>若`cmd1`执行完毕且正确执行，则`cmd2`不执行<li>若`cmd1`执行完毕且为错误，则开始执行`cmd2`|

示例

    #使用ls查阅目录/tmp/abc是否存在，若存在则用touch创建/tmp/abc/hehe
    [root@www ~]# ls /tmp/abc && touch /tmp/abc/hehe
    ls: /tmp/abc: No such file or directory

    [root@www ~]# mkdir /tmp/abc
    [root@www ~]# ls /tmp/abc/ && touch /tmp/abc/hehe
    [root@www ~]# touch /tmp/abc
    -rw-r--r-- 1 root root 0 Feb 7 12:43 hehe


    #测试/tmp/abc是否存在，若不存在则予以创建，若存在就不做任何事情
    [root@www ~]# rm -r /tmp/abc
    [root@www ~]# ls /tmp/abc || mkdir /tmp/abc
    ls: /tmp/abc: No such file or directory         <== 不存在
    [root@www ~]# ll /tmp/abc
    total 0                                     <== 执行了mkdir


    #不管/tmp/abc是否存在，就是要创建/tmp/abc/hehe文件
    [root@www ~]# ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe

Linux的命令执行是从左到右的，不存在优先级。对于不执行的命令，命令的返回码是会向后传递的。

    #/tmp/abc存在则显示exist,否则显示not exist
    [root@www ~]# ls /tmp/abc && echo "exist" || echo "not exist"


# 6 管道命令

管道命令与连续执行命令不同，下面一个例子来说明

    [root@www ~]# ls -al /etc | less

使用`ls -al /etc`来查阅，`ls`输出后的内容被`less`获取，并且利用`less`的功能，就能前后翻动相关的信息。


管道命令`|`只能处理由前面一个命令传来的正确信息，也就是`standard output`的输出。每个管道后面接的第一个数据必定是“命令”，且这个命令必须要能够接收`standard input`的数据。这样的命令才是管道命令。

## 6.1 选取命令:`cut`,`grep`

<font size = 4 color =white>`cut`</font>

`cut`命令可以将一段信息的某一段给切出来，处理的信息是以行为单位。

    [root@www ~]# cut -d '分隔字符' -f fields   <== 用于分隔字符
    [root@www ~]# cut -c 字符范围               <== 用于排列整齐的信息

- `-d`:后接分隔符，与`-f`一起使用
- `-f`:依据`-d`的分隔字符将一段信息切割成数段，用`-f`取出第几段的意思
- `-c`:以字符的单位取出固定字符区间。


示例

    #将PATH变量取出，找到第5个路径
    [root@www ~]# echo $PATH
    /bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:usr/X11R6/bin:/usr/games:

    [root@www ~]# echo $PATH | cut -d ':' -f 5
    /usr/local/bin
    [root@www ~]# echo $PATH | cut -d ':' -f 3,5
    /sbin   /usr/local/bin


    #将export输出的信息取得第12字符以后的所有字符串
    [root@www ~]# export
    declare -x HISTSIZE="1000"
    declare -x INPUTRC="/etc/inputrc"
    ......

    [root@www ~]# export | cut -c 12-
    HISTSIZE="1000"
    INPUTRC="/etc/inputrc
    ......                          <== 还可以指定第12-20个字符串，12-就表示12个字符之后的字符串

`cut`的主要用途在于将同一行里面的数据进行分解，不过`cut`在处理多空格相连的数据时会吃力一点。

<font size = 4 color =white>`grep`</font>

`grep`命令是分析多行信息，若当中有需要的信息，就将该行拿出来

    [root@www ~]# grep [-acinv] [--color=auto] '查找字符串' filename

- `-a`:将binary文件以text文件的方式查找数据
- `-c`:计算找到'查找字符串'的次数
- `-i`:忽略大小写的不同，将大小写视为相同
- `-n`:顺便输出行号
- `-v`:反向选择，即显示出没有'查找字符串'内容的那一行
- `--color=auto`:可以将找到的关键字部分加上颜色显示


示例

    #将last当中有出现root的那一行取出来
    [root@www ~]# last | grep 'root'


    #与上面相反，只要没有就root就取出
    [root@www ~]# last | grep -v 'root'

    #在last的输出信息中，只要有root就取出，并且仅取第一列
    [root@www ~]# last | grep 'root' | cut -d ' ' -f 1


    #取出/etc/man.config内含有MANPATH的那几行
    [root@www ~]# grep --color=auto 'MANPATH' /etc/man.config
    ......
    MANPATH_MAP /usr/X11R6/bin    /usr/X11R6/man
    ......


## 6.2 排序命令:`sort`,`wc`,`uniq`

<font size = 4 color =white>`sort`</font>

    [root@www ~]# sort [-参数] [file or stdin]

- `-f`:忽略大小写的差异
- `-b`:忽略最前面的空格符差异
- `-M`:以月份的名字来排序
- `-n`:使用纯数字进行排序（默认是以文字类型来排序的）
- `-r`:反向排序
- `-u`:就是`uniq`,相同的数据中，仅出现一行代表
- `-t`:分隔符，默认以[Tab]来进行分隔
- `-k`:以那个区间`field`来进行排序的意思。


示例

    #个人账号都记录在/etc/passwd下，将账号进行排序
    [root@www ~]# cat /etc/passwd | sort
    adm:x:.....
    apache:....
    bin.....
    daemon......

    #/etc/passwd以:来分割的，以第三列来进行排序
    [root@www ~]# cat /etc/passwd | sort -t ':' -k 3
    root:x:0:0....
    uucp:x:10:14....
    operator:x:11:0...

<font size = 4 color =white>`uniq`</font>

排序完成，将重复的数据仅列出一个显示

    [root@www ~]# uniq [-ic]

- `-i`:忽略大小写字符的不同
- `-c`:进行计数

示例

    #使用last将账号列出，仅取出账号列，进行排序后仅取出1位
    [root@www ~]# last | cut -d ' ' -f 1 | sort | uniq

    #接上，还需知道每个人的登录总次数
    [root@www ~]# last | cut -d ' ' -f 1 | sort | uniq -c


<font size = 4 color =white>`wc`</font>

计算出文件中有多少行文字，多少行，可以使用`wc`命令

    [root@www ~]# wc [-lwm]

- `-l`:仅列出行
- `-w`:仅列出多少字
- `-m`:多少字符

示例

    #计算/etc/man.config中有多少相关字，行，字符数
    [root@www ~]# cat /etc/man.config | wc
    141 722 4617

    #以一行命令串取得当月登录系统的总人次
    [root@www ~]# last | grep [a-zA-Z] | grep -v 'wtmp' | wc -l                 <== 由于last会输出空白行和wtmp字样在最下面两行，因此利用grep取出非空白行，以及去除wtmp那一行，再计算行数

    #目前账号文件中有多少个用户
    [root@www ~]# cat /etc/passwd | wc -l


## 6.3 双向重定向:`tee`

`tee`命令可以让数据流送到屏幕和文件，输出到屏幕的就是`stdout`，可以让下一个命令继续处理。

    [root@www ~]# tee [-a] file

- `-a`:以累加的方式，将数据加入file中

示例

    #将last的输出存一份到last.list文件中
    [root@www ~]# last | tee last.list | cut -d " " -f 1

    #将ls的数据存一份到~/homefile，同时屏幕也有输出信息
    [root@www ~]# ls -l /home | tee ~/homefile | more

    #tee后接的文件会被覆盖，若加上-a则将信息累加
    [root@www ~]# ls -l /home | tee -a ~/homefile | more

## 6.4 字符转换命令:`tr`,`col`,`join`,`paste`,`expand`

<font size = 4 color =white>`tr`</font>

`tr`可以用来删除一段信息当中的文字，或者是进行文字信息的转化

    [root@www ~]# tr [-ds] SET1 ...

- `-d`:删除信息当中的`SET1`这个字符串
- `-s`:替换掉重复的字符串

示例

    #将last输出的信息中所有的小写字符变成大写字符
    [root@www ~]# last | tr [a-z] [A-Z]

    #将/etc/passwd输出的信息中的冒号(:)删除
    [root@www ~]# cat /etc/passwd | tr -d ':'

    #将/etc/passwd转存成dos断行到/root/passwd中，再将^M符号删除
    [root@www ~]# cp /etc/passwd /root/passwd && UNIX2dos /root/passwd
    [root@www ~]# file /etc/passwd /root/passwd
    /etc/passwd: ASCII text
    /etc/passwd: ASCII text,with CRLF line termination    <== 就是DOS断行
    [root@www ~]# cat /root/passwd | tr -d '\r' > /root/passwd.linux            <== \r就是DOS的断行字符
    [root@www ~]# ll /etc/passwd /root/passwd*
    -rw-r--r-- 1 root root 1986 Feb 6 17:55 /etc/passwd
    -rw-r--r-- 1 root root 2030 Feb 7 15:55 /root/passwd
    -rw-r--r-- 1 root root 1986 Feb 7 15:57 /root/passwd.linux

<font size = 4 color =white>`col`</font>

    [root@www ~]# col [-xb]

- `-x`:将tab键转换成对等的空格键
- `-b`:在文字内有反斜杠`/`时，仅保留反斜杠最后接的那个字符

示例

    #利用cat -A 显示出所有的特殊按键，最后以col将[tab]转换成空白
    [root@www ~]# cat -A /etc/man.config      <== 此时hi看到很多^I的符号，那个就是[tab]
    [root@www ~]# cat /etc/man.config | col -x | cat -A | more


    #将col的man page转存成为/root/col.man的纯文本文件
    [root@www ~]# man col > /root/col.man
    [root@www ~]# vi /root/col.man
    ......              <== 含有^的怪异字符
    [root@www ~]# man col | col -b > /root/col.man

`col`命令经常被利用于将man page转存为纯文本文件，以方便查阅的功能，即第二个示例。

<font size = 4 color =white>`join`</font>

将两个文件中有相同数据的那一行加在一起

    [root@www ~]# join [-ti12] file1 file2

- `-t`:`join`默认以空格符分隔数据，并对比第一个字段的数据,如果两个文件相同，则将两条数据连成一行，且第一个字段放在第一个
- `-i`:忽略大小写的差异
- `-1`:数字1，代表第一个文件要用哪个字段来分析的意思
- `-2`:数字2，代表第二个文件要用哪个字段来分析的意思

示例

    #用root的身份，将/etc/passwd和/etc/shadow相关数据整合成一行
    [root@www ~]# head -n 3 /etc/passwd /etc/shadow
    ==> /etc/passwd <== 
    ......

    ==> /etc/shadow <==
    ......                      <== 输出的数据中，两个文件的最左边都是账号，且以:分割

    [root@www ~]# join -t ':' /etc/passwd /etc/shadow
    root:......                 <== 两个文件的行拼接在一起，相同字段不会显示
    bin:......                  
    daemon:......

需要注意的是在，使用`join`之前，需要处理的文件应该事先经过排序处理。否则有些对比过的项目会被忽略。

<font size = 4 color =white>`paste`</font>

`paste`就是直接将两行贴在一起，且中间以[tab]键隔开

    [root@www ~]# paste [-d] file1 file2

- `-d`:后面可以接分隔字符，默认以[tab]来分割
- `-`:如果`file`部分写成`-`，表示来自`standard input`的数据的意思

示例

    #将/etc/passwd与/etc/shadow的同一行粘贴在一起
    [root@www ~]# paste /etc/passwd /etc/shadow
    ......  
    ......                      <== 两个文件的同一行以[Tab]隔开

    #将/etc/group读出，然后与范例1粘贴在一起，且仅取出前3行
    [root@www ~]# cat /etc/group | paste /etc/passwd /etc/shadow -  | head -n 3

<font size = 4 color =white>`expand`</font>

`expand`就是将[Tab]键转成空格键

    [root@www ~]# expand [-t] file

- `-t`:后面接一个数字，可以自定义一个[Tab]用几个空格字符来表示

示例

    #将/etc/man.config内行首为MANPATH的字样取出，仅取出前3行
    [root@www ~]# grep '^MANPATH' /etc/man.config | head -n 3
    MANPATH /usr/man
    MANPATH /usr/share/man
    MANPATH /usr/local

    #接上例，要将所有的符号都列出来
    [root@www ~]# grep '^MANPATH' /etc/man.config | head -n 3 | cat -A
    MANPATH^I/usr/man
    MANPATH^I/usr/share/man
    MANPATH^I/usr/local

    #接上，将[tab]键设置成6个字符
    [root@www ~]# grep '^MANPATH' /etc/man.config | head -n 3 | expand -t 6 - | cat -A
    MANPATH     /usr/man
    MANPATH     /usr/share/man
    MANPATH     /usr/local

## 6.5 切割命令:`sqlit`

如果一个文件太大，可以利用`split`命令将一个大文件依据文件大小或者行数来切割成小文件

    [root@www ~]# split [-bl] file PREFIX

- `-b`:后面可以接欲切割成的文件大小，可以加单位，如b,k,m等
- `-l`:以行数来进行切割


示例

    #将700多kb的/etc/termacp文件分割成300kb一个文件
    [root@www ~]# cd /tmp; split -b 300k /etc/termcap termcap
    [root@www tmp]# ll -k termcap*
    -rw-r--r-- 1 root root 300 Feb 7 16:39 termcapaa
    -rw-r--r-- 1 root root 300 Feb 7 16:39 termcapab
    -rw-r--r-- 1 root root 300 Feb 7 16:39 termcapac

    #将上述的3个文件合成一个文件，文件名为termcapback
    [root@www tmp]# cat termcap* >> termcapback

    #将ls -al /输出的信息中，每10行记录一个文件
    [root@www tmp]# ls -al / | split -l 10 - lsroot
    [root@www tmp]# wc -l lsroot*
    10 lsrootaa
    10 lsrootab
     6 lsrootac
    <== 如果需要stdout/stdin但又偏偏没有文件，只有- 时，那么这个-就会被当成stdin或stdout


## 6.6 参数代换:`xargs`



## 6.7 关于减号`-`的用途

在管道命令中，常常会使用到前一个命令的`stdout`作为这次的`stdin`，某些命令要用到文件名来进行处理时，该`stdin`与`stdout`可以用`-`来代替。
