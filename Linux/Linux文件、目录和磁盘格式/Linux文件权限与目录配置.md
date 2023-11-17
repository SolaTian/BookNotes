# 1. Linux的文件权限和目录配置

> Linux是一个多用户、多任务的系统。有用户和用户组的概念。

- 默认情况下，Linux系统上所有的账号和一般身份用户，还有root的相关信息，都是记录在```/etc/password```文件内，个人的密码是记录在```/etc/shadow```文件下，Linux的所有组名记录在```/etc/group```中。

上面这三个文件不能随意的删除。


## 1.1 Linux的文件属性

>```ls -al```参数```-al```表示列出所有文件(包括文件名第一个字符为"."的文件)的详细的权限和属性|

- -a:表示显示所有文件及目录 (. 开头的隐藏文件也会列出)
- -l:以长格式显示文件和目录信息，包括权限、所有者、大小、创建时间等。

实例：
    -rw-r--r-- 1 user group 4096 Feb 21 12:00 file.txt


**<font size=4 color=red>1. 第1列代表这个文件的类型和权限</font>**

第1个字符的含义

- 若是[d]则是目录
- 若是[-]则是文件
- 若是[l]则是连接文件
- 若是[b]则是设备文件中可供存储的接口设备
- 若是[c]则是设备文件中的串行端口设备，如键盘，鼠标等。

第2-4个字符
- 文件所有者的权限，rwx的组合，[r]表示可读，[w]表示可写，[x]表示可执行，如果没有权限就用[-]表示。

第5-7个字符

- 同用户组的权限。

第8-10个字符

- 其他非本用户组的权限。

**<font size=4 color=red>2. 第2列代表有多少文件名连接到此节点</font>**

**<font size=4 color=red>3. 第3列代表这个文件或者目录的“所有者账号”</font>**

**<font size=4 color=red>4. 第4列代表这个文件或者目录的所属用户组</font>**

**<font size=4 color=red>5. 第5列代表这个文件或者目录的容量大小，单位为B</font>**

**<font size=4 color=red>6. 第6列代表这个文件或者目录的最近修改日期</font>**

**<font size=4 color=red>7. 第7列代表这个文件名或者目录名</font>**

## 1.2 修改文件属性和权限

> ```chgrp```修改文件所属的用户组

要改变的用户组必须在/etc/group中，

使用方法

        [root@www~]#chgrp [-R] dirname/filename ...
        [root@www~]#chgrp users install.log
        [root@www~]#ls -l
        [root@www~]#root users 68495 Jun 25 08:53 install.log
        [root@www~]#chgrp testing install.log
        chgrp: invalid group name 'testing'

```[-R]```表示递归，连同子目录下的所有文件、目录都更新为新的用户组,常用在更新某一目录内的所有的文件情况。


> ```chown```修改文件所有者

要修改的用户必须是已经存在与系统中的账号，也就是在/etc/password这个文件中有记录的用户名称才可以改变

使用方法

    [root@www~]#chown [-R] 账号名称 文件或目录
    [root@www~]#chown bin install.log
    [root@www~]#ls- l
    [root@www~]#bin users 68495 Jun 25 08:53 install.log
    [root@www~]#chown root:root install.log
    [root@www~]#ls -l
    [root@www~]#root root 68495 Jun 25 08:53 install.log

```[-R]```：进行递归，连同子目录下的所有文件都要更改.

用冒号```:```表示隔开所有者和用户组


在复制文件的时候，会用到上面的```chgrp```,```chown```

```cp```命令复制执行者的属性和权限，如果要给bin这个用户，就需要使用到上面的命令。

    [root@www~]#cp 源文件 目标文件
    [root@www~]#cp .bash .bash_test


> ```chmod```修改文件的权限

可以使用数字或者符号来修改权限

**<font size = 3 color = red>1.使用数字类型改变权限</font>**

Linux文件基本权限有9个，```owner```，```group```和```others```三种身份的read，write和execute

各权限对应的分值

    r:4
    w:2
    x:1
每种身份对应的分值最高为7

对应```chmod```的使用方法为

    [root@www~]#chmod [-R] xyz 文件或者目录
    [root@www~]#ls -al .bash
    -rw-r--r-- root root 68495 Jun 25 08:53 .bash
    [root@www~]#chmod 777 .bash
    [root@www~]#ls -al .bash
    -rwxrwxrwx root root 68495 Jun 25 08:53 .bash


**<font size = 3 color = red>2.使用符号类型改变权限</font>**

用每种身份的首字母```u```,```g```,```o```来代替三种身份的权限，```a```代表all

使用方法

    [root@www~]#chmod u=rwx,go=rx .bash
    [root@www~]#ls -al .bash
    -rwxr-xr-x root root 68495 Jun 25 08:53 .bash
    [root@www~]#chmod u=rwx,g=rx,o=r .bash
    -rwxr-xr-- root root 68495 Jun 25 08:53 .bash

如果不知道原先文件的属性，要实现对于文件的权限的增加和减少，使用方法如下

    [root@www~]#ls -al .bash
    -rwxr-xr-x root root 68495 Jun 25 08:53 .bash
    [root@www~]#chmod a+w .bash
    [root@www~]#ls -al .bash
    -rwxrwxrwx root root 68495 Jun 25 08:53 .bash
    [root@www~]#chmod a-x .bash
    [root@www~]#ls -al .bash
    -rw-rw-rw- root root 68495 Jun 25 08:53 .bash

## 1.3 目录与文件的权限的意义
在Linux中，文件的```r```,```w```容易理解，文件是否可以被执行时根据是否具有权限```x```来决定的，不像Windows是根据文件后缀名来判断的。而目录的权限```r```表示可以查询该目录下的文件名数据，即可以使用```ls```命令来显示目录下的内容，```w```表示可以更改目录，包括
-  新建新的文件和目录
-  删除已经存在的文件和目录
-  将已经存在的文件或者目录进行重命名
-  转移该目录的文件、目录的位置

```x```表示用户是否可以进入该目录，要开放目录给任何人浏览时，至少给出```r```和```x```的权限

下面给出一个例子，对权限含义进行讲解

首先在```/tmp```目录下新建一个目录和文件，并赋予权限

    [root@www~]#cd /tmp 
    [root@www tmp]#mkdir testing
    [root@www tmp]#chmod 744 testing
    [root@www tmp]#touch testing/testing      <==新建空的文件
    [root@www tmp]#chmod 600 testing/testing
    [root@www tmp]#ls -ald testing testing/testing
    drwxr--r-- 2 root root 4096 Jun 25 08:53 testing
    -rw------- 1 root root 0 Jun 25 08:53 testing/testing

一般身份的用户对于上面的这个目录/文件的权限

    [root@www tmp]# su - qintian     <==切换身份为qintian
    [qintian@www ~]$ cd /tmp
    [qintian@www tmp]$ ls -l testing/
    ?--------- ? ? ? ?      ? testing      <==因为具有r的权限可以查询文件名，但是权限不足，所以会有一堆问号。
    [qintian@www tmp]$ cd testing/
    -bash: cd testing/: Permission denied


本用户对于这个目录/文件的权限

    [qintian@www tmp]$exit      <==从qintian切换回原来的root
    [root@www tmp]#chown qintian testing  <==修改权限，让qintian拥有此目录
    [root@www tmp]#su - qintian
    [qintian@www ~]$cd /tmp/testing
    [qintian@www testing]$ls -l
    -rw------- 1 root root 0 Jun 25 08:53 testing
    [qintian@www testing]$rm testing
    rm:remove write-protected regular empty file 'testing'?

## 1.4 Linux文件种类与扩展名

|文件种类|描述|
|:-----:|------|
|普通文件|```ls -al```中，第一列的第一个字符为```[-]```|
|纯文本文件(ASCII)|内容为可以直接读到的数据，例如，使用```cat ~/.bashrc```即可以看到文件内容|
|二进制文件(binary)|如命令```cat```就是一个二进制文件|
|数据格式文件(data)|如在程序运行过程中会读取的某些特定格式的文件，称为数据文件，使用```cat```会读出乱码|
|连接文件(link)|第一个属性为```[l]```,类似于Windows下的快捷方式|
|目录(directory)|第一个属性为```[d]```|
|设备与设备文件(device)|与系统外设及存储相关的文件，通常集中在```/dev```目录下，分为两类:<li>块设备文件(block):一些存储数据，以提供系统随机访问的接口设备，如硬盘、软盘等，如在```/dev/sda```中的文件第一个属性为```[b]```<li>字符设备文件(character)：一些串行端口的设备，例如键盘鼠标，第一个属性为```[c]```|
|套接字(socket)|第一个属性为```[s]```，通常在```/var/run```目录中可以看到这种文件类型，被用来在网络上的数据连接|
|管道(FIFO)|用来解决多个程序同时访问一个文件所造成的错误问题，第一个属性为```[p]```|

Linux文件是否可以被执行与Windows不同，仅看第一列的的10的属性有关，只要具有x即可被执行，Windows下则看文件的后缀名，如```.com```、```.exe```、```.bat```。但是Linux下虽然可以被执行，但是不一定执行成功，如执行一个文本文件install.log。下面给出一些文件的后缀名

|文件后缀名|文件描述|
|----|------|
|```*.sh```|脚本或者批处理文件，使用shell编写|
|```*Z```、```*.tar```、```*.tar.gz```、```*.zip```、```*.tgz```|经过打包的压缩文件，压缩软件分别为```gunzip```和```tar```|
|```*.html```、```*.php```|网页相关文件|


## 1.5 Linux目录配置

Linux目录配置标准:FHS

**FHS仅定义了三层目录下应该放置什么软件**，分别是下面三个，其中```/usr```和```/var```也属于```/```的子目录

<font size=4 color =red>根目录```/```</font>
> ```/```(root,根目录):与开机系统有关

FHS定义，根目录```/```所在的分区要有以下的子目录存在

|目录|放置文件内容|
|---|---|
|```/bin```|```/bin```放置的是单用户维护模式下还能被操作的命令，可以被root与一般账号所使用，主要有```cat```、```chmod```、```mv```、```cp```等命令|
|```/boot```|放置开机时使用的文件，包括Linux内核文件以及开机菜单与开机所需要的配置文件|
|```/dev```|Linux下任何设备与接口设备都是以这个文件的形式存在于这个目录当中，访问这个目录下的某个文件就等于访问某个设备，如```/dev/tty/```、```/dev/sd*```等|
|```/etc```|系统主要的配置文件都是放置在这个目录下，例如人员的账号密码，各种服务的起始文件等，一般用户都是可以查阅的，但是只有root有权利修改，FHS建议不放置可执行文件在该目录下，有一些重要的目录如下<li>```/etc/ini.d```:所有服务的默认启动脚本都是在这里面<li>```/etc/xinetd.d```:就是所谓的super daemon管理的各项服务的配置文件目录|
|```/home```|系统默认的用户主文件夹，主文件夹有两种代号：<li>```~```代表目前这个用户的主文件夹<li>```dmtsai```代表```dmtsai```的主文件夹|
|```/lib```|放置开机时会用到的函数库，以及在```/bin```和```/sbin```下面的命令会调用的函数。```/lib/modules```下放置内核相关的模块|
|```/media```|放置可删除的设备。包括软盘、光盘、DVD等设备都暂时挂载在此|
|```/mnt```|暂时挂载某些额外的设备，用途和```/media```相同。有了```/media```该目录就用来暂时挂载|
|```/opt```|给第三方软件放置的目录|
|```/root```|系统管理员(root)的主文件夹|
|```/sbin```|放置开机过程中需要的，包括开机、修复、还原系统所需要的命令|
|```/srv```|一些网络服务启动之后，这些服务需要取用的数据目录。图WWW、FTP等服务|
|```/tmp```|这是让一般用户或者是正在执行的程序暂时放置文件的地方，任何人都能访问，需要定时清理。|

除了FHS定义，还有其他一些重要的目录
|目录|放置文件内容|
|-|-|
|```/lost+found```|使用标准的```ext2/ext3```文件系统格式才会产生的一个目录|
|```/proc```|虚拟文件系统，放置的数据都是在内存中，例如系统内核、进程、外部设备的状态及网络状态等，不占用任何的硬盘空间|
|```/sys```|与```/proc```类似，是一个虚拟的文件系统，主要也是记录与内核相关的信息。|


**<font size = 4 color=red>```/usr```</font>**

> ```/usr```(UNIX software resource):与软件安装执行有关
安装时会占用较大硬盘容量的目录

FHS建议所有的软件将数据合理的放置在这个目录下的子目录。因为所有的系统默认的软件都会放到```/usr```，有些类似Windows的```c:\Windows```和```C:\Program files```

|文件目录|放置在目录下的文件|
|-|-|
|```/usr/X11R6```|为X Window系统重要数据放置的目录|
|```/usr/bin```|绝大部分的用户命令都可以放在这里，与```/bin```的区别在与是否与开机有关|
|```/usr/include```|C/C++等程序语言的头文件与包含文件的放置处|
|```/usr/lib```|包含各种应用软件的函数库，目标文件，以及不被一般用户惯用的执行文件或者脚本|
|```/usr/local```|系统管理员在本机自行安装的自己下载的软件，建议安装到此目录|
|```/usr/sbin```|非系统正常运行所需要的系统命令|
|```/usr/share```|放置共享文件的地方|
|```/usr/src```|一般源码放置到这里，内核源码放置到```/usr/src/linux```中|

**<font size = 4 color =red>```/var```</font>**

>```/var```:系统运行后才渐渐占用硬盘容量的目录，包括缓存、登录文件以及某些软件运行所产生的文件。包括程序文件或者例如MySQL数据库文件。


|文件目录|放置在目录下的文件|
|-|-|
|```/var/cache```|应用程序运行过程中产生的一些暂存文件|
|```/var/lib```|程序执行过程中需要使用的数据文件的放置目录，如MySQL的数据库放置到```/var/lib/mysql```中，rqm的数据库放置到```/var/lib/rqm```|
|```/var/log```|登录文件放置的目录|
|```/var/mail```|放置个人电子邮件信箱的目录|
|```/var/run```|某些程序或者服务启用之后，会将它们的PID放置到这个目录下面|
|```/var/spool```|通常放置一些队列数据，排队等待其他程序使用的数据，这些数据使用后通常都会被删掉|



![目录树结构图](/Users/qintian/Pictures/wechat/dirtree.png)

## 1.6绝对路径和相对路径

> 绝对路径:由根目录```/```开始泄气的文件名或者目录名如```/home/dmtsai/.bashrc```
> 相对路径:相对于目前路径的写法,例如```./home/dmtsai```或者```../../home/dmtsai```等开头不是```/```的写法

> ```.```表示的是当前的目录，也可以使用```./```来表示
> ```..```表示的是上一层目录。也可以用```../```来表示

```./run.sh```这个命令就表示执行本目录下的名为```run.sh```的文件