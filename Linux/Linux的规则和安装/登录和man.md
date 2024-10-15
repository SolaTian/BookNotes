# 1、命令行模式下执行命令

## 1.1、基本命令操作

### 显示日期与时间：date

    [qintian@www ~]$ date
    Mon Aug 17 17:02:52 CST 2009
    [qintian@www ~]$ date +%Y/%m/%d
    2009/08/17
    [qintian@www ~]$ date +%H:%M
    17:04


### 显示日历的命令：cal

    [qintian@www ~]$ cal                    //显示当前日历
    Augst 2009
    Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6  7
    ...

    [qintian@www ~]$ cal 2009               //显示2009年的所有日历
    ...
    [qintian@www ~]$ cal [[month] year]
    [qintian@www ~]$ cal 10 2009            //显示2009年10月的日历

### 计算器：bc

    [qintian@www ~]$ bc
    bc 1.06
    CopyRight ...
    ...
    _                                       <== 光标等待输入
    1+2+3
    6
    2*4
    8
    1/3
    0
    quit                                    //使用quit退出

`bc`默认输出整数，如果要输出全部小数，还需要执行`scale=number`，其中`number`表示的是小数点后的位数。

    [qintian@www ~]$ bc
    bc 1.06
    CopyRight ...
    ...
    _
    scale=3
    1/3
    .333
    quit

## 1.2、重要热键[Tab]、[Ctrl+C]、[Ctrl+D]

|热键|说明|
|-|-|
|[Tab]|<li>接在一串命令的第一个命令后面，表示命令补全；<li>接在一串命令的第二个命令以后时，表示文件补齐。<li>在命令行模式下，直接按下两个[Tab]键可以查看有多少个命令可以用|
|[Ctrl+C]|中断当前程序|
|[Ctrl+D]|代表键盘输入结束，可以用来代替`exit`，如想要离开文字界面，可以直接[Ctrl+D]离开|

## 1.3、man page

### 如何查看man page

如果不知道某一个命令的作用以及如何使用，可以使用`man`命令来查看

    [qintian@www ~]$ man date
    DATE(1)         User Commands           DATE(1)
    NAME
        ...
    SYNOPSIS
        ...
    DESCRIPTION
        ...
    OPTIONS
        ...
    COMMANDS
        ...
    FILES
        ...
    SEE ALSO
        ...
    EXAMPLE
        ...
    BUGS
        ...


`man page`中`DATE`是命令的名字，`(1)`代表的是一般用户可以使用的命令。常见的几个数字的意义如下所示：
|代号|代表内容|
|-|-|
|1|用户在`shell`环境下可以操作的命令或者可执行文件|
|2|系统内核可调用的函数与工具等|
|3|一些常用的函数与函数库，大部分为C的函数库(`libc`)|
|4|设备文件的说明，通常是在`/dev`下的文件|
|5|配置文件或者是某些文件的格式|
|6|游戏|
|7|惯例与协议，如Linux文件系统，网络协议，ASCII说明|
|8|系统管理员可用的管理命令|
|9|与kernel有关的文件|

其中1，5，8比较重要。

|`man page`组成|说明|
|-|-|
|`NAME`|简短的命令，数据名称说明|
|`SYNOPSIS`|简短的命令执行语法|
|`DESCRIPTION`|较为完整的说明，需要仔细看|
|`OPTIONS`|针对`SYNOPSIS`部分中，有列举的所有可用的选项说明|
|`COMMANDS`|当这个程序执行的时候，可以在此程序中执行的命令|
|`FILES`|这个程序或数据所使用或参考或连接到某些文件|
|`SEE ALSO`|这个命令或数据有相关的其他说明|
|`EXAMPLE`|一些可以参考的范例|
|`BUGS`|是否有相关的错误|

通常在查询数据时，可以按照这样的顺序来查
1. 先查看`NAME`项目，粗略看一下这个数据或者命令的意思
2. 再仔细看一下`DESCRIPTION`，这个部分会用到很多相关的资料与用法，从这个地方可以学到一些细节
3. 如果命令很熟悉了，主要查询关于`OPTIONS`的部分
4. 最后再看一下跟这个资料有关的还有哪些东西可以使用的。如看`SEE ALSO`
5. 某些说明内容，还会列举有关的文件`FILES`来提供参考。


### man page操作

|`man page`操作|说明|
|-|-|
|去到首页|[Home]|
|去到最后一页|[End]|
|向下翻页|空格键，[Page Down]|
|向上翻页|[Page Up]|
|向下查找关键字|`/word`|
|向上查找关键字|`?word`|
|`n`，`N`|利用`?`或者`/`查询字符串时，可以使用`n`继续下一个查询，使用`N`进行反向查询|
|退出|`q`|

若要重复查询某个字符串时，只需要使用`n`或者`N`来操作即可。

### 查询特定命令/文件的man page

有时候忘记了需要查询的命令的完整名称，或者只是记得部分关键字。

    #查出系统中和man这个命令有关的说明文件
    [qintian@www ~]$ man -f man
    man                     (1)     - format and display the on-line manual pages
    man                     (7)     - marcos to format man pages
    man.config(man)         (5)     - configuration data for man
    
使用`-f`可以获取更多与`man`相关的信息。可以通过指定不同的数字将上面的三个和`man`相关的文件找出来。

    [qintian@www ~]$ man 1 man              //取出在man page中和1相关的文件
    [qintian@www ~]$ man 7 man              //取出在man page中和7相关的文件

如果忘记输入数字，输入了`man man`则取出的数据和查询的顺序有关，这个顺序记录在`/etc/man.conf`这个配置文件中，先查询到的那个说明文件就会被先显示出来。一般来说，因为排序的关系通常会先找到数字较小的那个。

使用`man -f man`输出的结果中，输出的数据：
- 左边部分：命令或文件以及该命令所代表的含义（就是那个数字）
- 右边部分：这个命令的简易说明
- 使用`man -f`命令时，`man`只会找数据中的左边的那个命令或者文件的完整名称，有一点不同都不行，如果想要找关键字，不需要完全相同的命令或文件就能够找到时。使用`man -k`可以根据关键字找出

    [qintian@www ~]$ man -k man
    zshall          (1)         - the Z shell meta-man page
    zshbuiltins     (1)         - zsh built-in commands
    zshzle          (1)         - zsh command line editor
    ...

说明文件中主要含有`man`这个字相关的就将它取出。

    [qintian@www ~]$ whatis  [命令或者数据]         <== 相当于man -f [命令或者数据]
    [qintian@www ~]$ apropos [命令或者数据]         <== 相当于man -k [命令或者数据]

需要注意的是，这两个特殊命令能够使用，必须要创建`whatis`数据库才可以。这个数据库的创建需要使用`root`身份执行。

真正在使用命令的时候
- 先用疑似命令`xx [Tab] [Tab]`将可能的命令找出来
- 再使用`man`去查询命令的用法

## 1.4、info page

`info page`将文件数据拆成一个个段落，每个段落用自己的页面来撰写，并在各个页面中还有类似网页的超链接来跳到各不同的页面中，每个独立的页面也被称为一个节点（node）。可以将`info page`想成是命令行模式的网页显示数据。

不过要查询的目标数据的说明文件必须要以`info`的格式来写成才能够使用`info`的特殊功能（如超链接）。支持`info`命令的文件默认是放在`/usr/share/info/`这个目录下。

    [qintian@www ~]$ info info 
    File: info.info   Node: Top   Next: Getting Started, Up:(dir)
    ...
    ...
    * Getting Started::     Getting started using an Info reader
    * Expert Info::         Info Commands for experts
    * Creating an Info Files:: How to make your own Info file
    * Index::               An index of topics, commands,and variables.

- `File`：代表这个`info page`的数据是来自于`info.info`文件所提供的
- `Node`：代表目前的这个节点是属于`Top`节点
- `Next`：下一个节点的名称为`Getting Started`，使用`N`到下个节点
- `Up`：上一层的节点总览界面，按下`U`到上一层。
- `Prev`：前一个节点，由于`Top`是`info.info`的第一个节点，所以上面没有前一个节点的信息。

使用`N/P/U`到下一个，上一个与上一层的节点。`Menu`分为四小节，分别是`Getting Started`等。使用上下左右按键将光标移到该文件或者`*`上按下`[Enter]`就可以前往该小节，可以按下`[Tab]`键，在上面的界面中的节点快速移动。


### info page的操作

|`info page`操作|说明|
|-|-|
|空格键|向下翻一页|
|[Page Down]|向下翻一页|
|[Page Up]|向上翻一页|
|[Tab]|在节点之间移动，有节点的地方，通常会以`*`显示|
|[Enter]|当光标在节点上面时，按下[Enter]可以进入该节点|
|`B`|移动光标到该`info`界面中的第一个节点处|
|`E`|移动光标到该`info`界面中的最后一个节点处|
|`N`|前往下一个节点处|
|`P`|前往上一个节点处|
|`U`|向上移动一层|
|`S(/)`|在`info page`中进行查询|
|`H`|显示求助菜单|
|`?`|命令一览表|
|`Q`|退出`info page`|


### 其他有用的文件（documents）

一般的命令或者软件开发者都会将自己的命令或者软件的说明制作成“在线帮助文件”。Linux下，放在`usr/share/doc`目录下。这个目录下的数据主要是以软件包为主，例如`GCC`


## 1.5、文本编辑器：nano

    [qintian@www ~]$ nano test.txt
    # test.txt不存在则创建新文件打开，存在就打开原先文件
    GNU nano 1.3.12        File:test.txt
    _                   <== 鼠标光标处



    ^G  Get Help  ^C WriteOut   ^R Read File
    ...

组合键含义

- [Ctrl]+G：获取在线帮助
- [Ctrl]+X：离开nano，若有修改则会提示是否需要保存
- [Ctrl]+O：保存文件，若有权限则可以保存文件
- [Ctrl]+R：从其他文件读入数据，可以将某个文件的内容贴在本文件中
- [Ctrl]+W：查询字符串
- [Ctrl]+C：说明光标所在处的行数和列数信息
- [Ctrl]+_：可以直接输入行号，让光标快速移动到该行
- [ALT]+Y：矫正语法功能开启或者关闭
- [ALT]+M：支持鼠标来移动光标的功能

## 1.6、正确的关机方法

1. 查看系统的使用状态
   1. 查看目前谁在线：`who`；
   2. 查看网络连接状态：`netstat -a`
   3. 查看后台执行程序：`ps-aux`
2. 通知在线用户关机的时刻
3. 正确的关机命令
   1. 将数据同步写入硬盘中的命令：`sync`
   2. 惯用的关机命令：`shutdown`
   3. 重启、关机：`reboot`，`halt`，`poweroff`
   
### 数据同步写入磁盘：sync

在系统关机或者重启时，直接在文字界面下输入`sync`，将内存中尚未被更新的数据会被写入硬盘中。虽然`shutdown`，`reboot`，`halt`等命令在关机前进行了`sync`这个工具的调用。

    [qintian@www ~]$ sync

### 惯用的关机命令：shutdown

Linux除非是以图形界面来登录系统时，不论什么身份都可以正常关机，若是使用远程工具，那么关机就只能有root权限才可以。

`shutdown`可以完成一下的工作
- 自由选择关机模式：关机、重启或者进入单用户操作模式均可
- 设置关机时间：设置成立刻关机，也可以设置某个特定的时间才关机
- 自定义关机消息：在关机之前，可以将自己设置的消息传送给在线用户
- 可以发出警告消息：有时候可能只是进行一些测试，不想被其他用户干扰，或者是告诉用户哪一段时间注一下，可以使用`shutdown`来通知用户
- 可以选择是否使用`fsck`检查文件系统。

    [qintian@www ~]$ /sbin/shutdown [-t 秒] [-arkhncfF] 时间 [警告信息]

参数：
- `-t`：过几秒后关机
- `-k`：不要真的关机，只是发送警告消息
- `-r`：在系统的服务停掉之后就重启
- `-h`：将系统的服务停掉之后，立刻关机
- `-n`：不经过`init`程序，直接以`shutdown`的功能来关机
- `-f`：关机并开机之后，强制进行`fsck`的磁盘检查
- `-F`：系统重启之后，强制略过`fsck`的磁盘检查
- `-c`：取消在进行的`shutdown`命令内容
- 时间：一定要加入的参数，指定系统关机的时间


        [qintian@www ~]$ /sbin/shutdown -h 10 "I will shutdown after 10 min"


示例

    [qintian@www ~]$ shutdown -h now        <== 立刻关机
    [qintian@www ~]$ shutdown -h 20:25      <== 今天20:25关机
    [qintian@www ~]$ shutdown -h +10        <== 过10min后关机
    [qintian@www ~]$ shutdown -r now        <== 系统立刻重启
    [qintian@www ~]$ shutdown -r +30 "The system will reboot"   <== 警告，系统39min后重启
    [qintian@www ~]$ shutdown -k now "The system will reboot"   <== 警告，不关机

### 重启、关机：reboot，halt，poweroff

`halt`可以在不理会目前系统状况下，进行硬件关机的特殊功能。基本上在默认的情况下，`reboot`，`halt`，`poweroff`会完成一样的工作。

### 切换执行等级：init

Linux的执行等级：
- run level 0：关机
- run level 3：纯命令行模式
- run level 5：含有图形界面模式
- run level 6：重启
切换工作模式，可以使用`init`命令来执行，例如想要关机

    [qintian@www ~]$ init 0