# 1 EXT2文件系统

## 1.1 硬盘组成和分区简介

磁盘分为硬(磁)盘和软(磁)盘

磁盘的组成主要有：

1. 圆形的盘片（主要记录数据的部分）
2. 机械手臂和机械手臂上的磁头(可读写盘片上的数据)
3. 马达:转动盘片，让机械手臂的磁头在盘片上读写数据。


盘片的组成：

1. 扇区(Sector):最小的物理存储单位，每个扇区为512bytes
2. 柱面:扇区组成一个圆，柱面是分区(partition)的最小单位
3. 第一个扇区最重要，里面有硬盘主引导记录(`MBR`)以及分区表，其中(`MBR`)占有446bytes，而分区表则占有64bytes

> 磁盘分区:分区的起始和结束柱面。

这个分区的范围记录在第一个扇区的分区表中，有64bytes，最多只能记录4条磁盘分区记录，这4条记录被称为`主分区`或者`扩展分区`，`扩展分区`还可以再分出`逻辑分区`，能被格式化的只有`主分区`与`逻辑分区`，扩展分区无法被格式化。

`扩展分区`最多只能有1个

## 1.2 文件系统特性

磁盘分区之后还需要格式化，操作系统才可以使用这个分区。因为不同的操作系统所设置的文件属性/权限不相同，为了存放这些文件所需要的数据，就需要将分区进行格式化。

例如：`Windows 98`以前的操作系统使用的文件系统为`FAT`;`Windows 2000`以后的版本使用的文件系统为`NTFS`;`Linux`正规文件系统使用的是`Ext2`。

文件数据除了文件实际内容以外，还包括非常多的属性。Linux会将这两部分数据放在不同的块，权限与属性放置在`inode`里面，实际数据放到`data block`中，此外，还有一个超级块`super block`会记录整个文件系统的整体信息。包括`inode`、`block`的总量，使用量和剩余量。

- `super block`:记录此文件系统的整体信息，包括`inode`和`block`的总量，剩余量，使用量，以及文件系统的格式和相关信息
- `inode`:记录文件的属性，一个文件占用一个`inode`，同时记录此文件所在的`block`号码
- `block`:实际记录文件的内容，若文件太大时，则会占用多个`block`

像上面的这种文件系统叫做`索引式文件系统`，通过`inode`去读取`block`号，一下子读取所有`block`的内容,如`Ext2`文件系统。

U盘使用的文件系统一般为`FAT`格式，`FAT`这种文件系统没有`inode`存在，每一个`block`号码都记录在前一个`block`中，这样如果一个文件的`block`写入过离散，文件读取性能就会很差，因此`FAT`文件系统尝尝需要碎片整理一下，把`block`汇合一下,这样文件读取会变容易。


## 1.3 Linux的`Ext2`文件系统

文件系统一开始就将`inode`和`block`规划好了，除非重新格式化(或者利用`resize2fs`等命令更改文件系统的大小)，否则`inode`和`block`固定后就不再变动。如果文件系统高达数百GB时，将所有的`inode`和`block`放在一起，数量很大，不容易管理。

`Ext2`文件系统在格式化的时候基本是区分多个块组(`block group`),每个块组都有独立的`inode`,`block`,`super block`。

文件系统最前面会有个启动扇区(`boot sector`)，这个启动扇区可以安装引导装载程序，这样就可以将不同的引导装载程序安装到个别的文件系统最前端，而不用覆盖整块硬盘唯一的`MBR`。

下图为`Ext2`文件系统示意图

![`EXT2`文件系统示意图](https://img2018.cnblogs.com/blog/952033/201907/952033-20190719130705497-2100295137.png)

|块组(`block group`)主要内容|详细说明|
|-|-|
|`data block`数据块|用来放置文件内容的地方。`EXT2`文件系统支持的`block`大小有1KB、2KB以及4KB。由于`block`大小不同，该文件系统能够支持的最大磁盘容量与单一文件容量并不相同。`block`还有以下限制<li>block的大小和数量在格式化的时候就不能再改变，除非重新格式化<li>每个`block`内最多只能放置一个文件的数据<li>如果文件大于`block`的大小，一个文件就会占用多个`block`数量<li>如果文件小于`block`，则该`block`的剩余空间就不能够再被使用，会造成磁盘空间的浪费|
|`inode table`(`inode 表格`)|`inode`记录的文件数据至少包含以下内容<li>该文件的访问模式(`read`,`write`,`execute`)<li>该文件的所有者与组(`owner`,`group`)<li>该文件的大小<li>该文件创建或状态改变的时间(`ctime`)<li>最近一次的读取时间(`atime`)<li>最近修改的时间(`mtime`)<li>定义文件特性的标志，如`SetUID`等<li>该文件真正内容的指向(`pointer`)<br>每个`inode`大小固定为128bytes；<br>每个文件仅会占用一个`inode`而已，因此文件系统能创建的文件数量与`inode`有关；<br>系统在读取文件时需要先找到`inode`，并分析`inode`所记录的权限与用户是否符合，符合才开始读取`block`内容。<br>(具体`inode`里面存放`block`的细节可以查看《鸟哥的Linux私房菜》[Page201-202])|
|`super blocks`超级块|记录整个文件系统的相关信息的地方，没有`super block`，就没有这个文件系统了，记录的主要信息有:<li>`block`和`inode`的总量;<li>未使用的和已使用的`inode`和`block`数量;<li>`block`与`inode`的大小(`block`为1K，2K，4K，`inode`为128bytes);<li>文件系统的挂载时间、最近一次写入数据的时间、最近一次检验磁盘的时间等文件系统的相关信息;<li>一个`validbit`数值，若文件系统已被挂载，则该值为0，否则值为1。<br>一般`super block`的大小为1024bytes。一般来说一个文件系统只有一个`super block`，除了第一个`block group`有`super block`外，其他的`super block`为第一个的备份，用于救援文件系统。|
|`File system Description`文件系统描述说明|描述每个`block group`的开始和结束的`block`号码，以及每个区段(`super block`、`bitdump`、`inodedump`、`data block`)分别介于哪一个`block`号码之间|
|`block bitdump`块对照表|`block bitdump`记录哪些`block`是空的，可以在添加新文件的时候帮助找到可使用的空间。同时`block bit dump`会将文件删除后空的`block`标志修改为`未使用中`|
|`inode bitdump`inode对照表|与`block bitdump`功能类似，记录的是`inode`|

**一般来说，`inode table`和`data block`称为数据存放区域，`super block`、`inode bitdump`、`block bitdump`等区段称为称为`中间数据`，因为每次添加、删除、编辑时可能会影响到这三部分的数据。**

查看文件系统的相关参数和命令可以参阅《鸟哥的Linux私房菜》[Page203-204]

## 1.4 与目录树的关系

每个文件不管是一般文件还是目录文件都会占用一个`inode`

目录和文件在`Ext2`文件系统中如何记录数据

**<font size = 3 color = red>1.目录</font>**

当在Linux的`Ext2`文件系统下新建一个目录时，`Ext2`会分配一个`inode`和至少一个`block`给该目录，其中`inode`记录该目录的相关权限和属性，并可记录分配的那块`block`号码；而`block`记录这个目录下的文件名与该文件名占用的`inode`号码数据。

查看目录内文件所占用的`inode`号码，可以使用`ls -i`

    [root@www ~]# ls -li
    654683 -rw------- 1 root root 1474 Sep 4 18:27 anaconda-ks.cfg
    648322 -rw-r--r-- 1 root root 42304 Sep 4 18:26 install.log
    654683 -rw-r--r-- 1 root root 5661 Sep 4 18:25 install.log.syslog

当目录下的文件数过大，会导致一个`block`无法容纳所有文件名与`inode`对照表，Linux会给该目录多一个`block`去记录相关的数据。

**<font size = 3 color = red>2.文件</font>**

在Linux的`Ext2`目录下新建一般文件是，`Ext2`会分配一个`inode`和相对于该文件大小的`block`数量给该文件。

**<font size = 3 color = red>3.目录树读取</font>**

因为目录下的文件名是记录在目录的`block`中，因此当需要读取某个文件时，就会经过目录的`inode`和`block`，才能够找到待读取的文件的`inode`，最终才可以读取到正确的文件的`block`。所以新增、删除、重命名文件和目录的权限`w`有关。


目录树是从根目录开始读起，因此系统通过挂载的信息可以找到挂载点的`inode`(通常一个文件系统的最顶层的`inode`从2开始)依据根目录的`inode`读取`block`里面的文件名，一层一层往下读。


**<font size = 3 color = red>4.文件系统大小与磁盘读取功能</font>**

虽然`inode`中记录了`block`号码，但是文件系统很大的时候，如果`block`很离散，还是会发生读取效率低的问题。可以将整个文件都复制出来，将文件系统重新格式化，再将数据复制回去可以解决这个问题。



## 1.5 `Ext2/Ext3`文件的访问与日志文件系统的功能

1.4节介绍了读取，新建一个文件，文件系统的行为流程如下:

1. 先确定用户对于欲添加文件的目录是否具有`w`和`x`的权限，有才可以添加;
2. 根据`inode bitdump`找到没有使用的`inode`号码，并将新文件的权限/属性写入;
3. 根据`block bitdump`找到没有使用的`block`号码，将实际的数据写入`block`中，且更新`inode`的`block`指向数据。
4. 将刚才写入的`inode`和`block`数据同步更新`inode bitdump`与`block bitdump`，并更新`super block`的内容。

**<font size = 3 color = red>1.数据不一致状态</font>**

当写入文件的时候，如果遇到突发情况如系统断电，最后只写入了`inode table`和`data block`,最后一个同步`中间数据`的步骤没有完成，就会`中间数据`的内容与与实际数据存放区不一致的情况。

早期`Ext2`文件系统会在重启后，通过`super block`中记录的`valid bit`是否挂载，与文件系统的`state`是否为`clean`等状态来判断是否强制进行数据一致性的检查。但是当文件系统很大时，这样的检查会十分耗时。

**<font size = 3 color = red>2.日志文件系统统</font>**

为了应对上面提到的检查耗时情况，引入了日志文件系统，在文件系统中划分出一个块。

1. 预备：当系统要写入一个文件时，会在日志记录快中记录某个文件准备要写入的信息。
2. 实际写入：开始写入文件的权限与数据；开始更新`中间数据`的数据
3. 结束：完成数据和`中间数据`的更新之后，在日志记录快当中完成该文件的记录。

如果数据的记录过程中发生问题，那么我们的系统只要检查日志记录快就可以知道哪个问题发生了问题。只需针对该问题来做一致性的检查即可，不必针对整块文件系统进行检查。

`Ext3`是`Ext2`而的升级版，可以向下兼容`Ext2`的版本。

## 1.6 Linux文件系统的操作

## 1.7 挂载点的意义

文件系统中包含`inode`，`block`，`super block`等信息，文件系统只有链接到目录树才能被使用，`将文件系统与目录树结合的操作称为挂载`。挂载点一定是目录，该目录为进入该文件系统的入口。只有挂载之后才可以使用文件系统。

可以通过文件或者目录的`inode`号码判断文件是否为同一个文件。

## 1.8 其他Linux支持的文件系统与`VFS`

Linux的标准文件系统是`Ext2`，还有增加了日志功能的`Ext3`。Linux 还支持一些其他的文件系统，如`FAT`,`NFS`等文件系统。


查询Linux系统支持的文件系统有哪些,可以查看下面这个目录

    [root@www ~]# ls -l /lib/modules/$ (uname -r) /kernel/fs

系统目前已经加载到内存中支持的文件系统

    [root@www ~]# cat /proc/filensystems

Linux的内核通过以`VFS`(虚拟文件系统)的内核功能去读取文件系统，真个Linux认识的文件系统都是在`VFS`中进行管理。
# 2 文件系统的简单操作


## 2.1 磁盘与目录的容量`df`，`du`

**<font size = 5 color = red>`df`</font>**
 
列出文件系统的整体磁盘使用量。

    [root@www ~]# df [-参数] [目录或者文件名]

- `-a`:列出所有的文件系统，包括系统持有的`/proc`等文件系统
- `-k`:以KB的容量显示所有的文件系统
- `-h`:以GB、MB、KB等格式自行显示
- `-i`:不用硬盘容量，而以`inode`的数量来显示。

示例

    #将系统的所有的文件系统列出来
    [root@www ~]# df
    FileSystem  1k-blocks  Used    Availble  Use%  Mounted on
    /dev/hdc2    9920642   3823112 5585444   41%   /
    /dev/hdc3    4956316   141376  4559108   4%    /home
    /dev/hdc1    101086    11126   84741     12%   /boot
    tmpfs        371332    0       371332    0%    /dev/shm        <== 与系统内存有关的挂载

    #将容量以易读的结果显示出来
    [root@www ~]# df -h
    FileSystem  size  Used  Availble  Use%  Mounted on
    /dev/hdc2   9.5G  3.7G  5.4G      41%   /
    /dev/hdc3   4.8G  139M  4.4G      4%    /home
    /dev/hdc1   99M   11M   83M       12%   /boot
    tmpfs       363M  0     363M      0%    /dev/shm

    #将系统中所有的特殊文件格式及名称都列出来
    [root@www ~]# df -aT
    FileSystem Type   1k-blocks  Used    Availble  Use%  Mounted on
    /dev/hdc2  ext3   9920642    3823112 5585444   41%   /
    proc       proc   0          0       0         -     /proc          <== 挂载在内存当中
    sysfs      sysfs  0          0       0         -     /sys
    devpts     devpts 0          0       0         -     /dev/pts
    /dev/hdc3     4956316   141376  4559108   4%    /home
    /dev/hdc1     101086    11126   84741     12%   /boot
    tmpfs         371332    0       371332    0%    /dev/shm

下面说一下输出结果的各个表头的含义

- `Filesystem`:代表该文件系统是在哪个分区，所以列出设备名称。
- `1k-blocks`:说明下面的数字单位为1KB,可以利用`-h`或者`-m`来改变容量。
- `Used`:使用掉的硬盘空间
- `Available`:剩下的磁盘空间大小
- `Use%`:磁盘使用率
- `Mountedon`:磁盘挂载的目录所在点。


**<font size = 5 color = red>`du`</font>**

评估文件系统的磁盘使用量(常用于评估目录所占容量)

    [root@www ~]# du [-参数] 文件或者目录系统

- `-a`:列出所有的文件与目录容量
- `-h`:以G/M显示
- `-s`:列出总量，而不列出每个各别的目录占用容量
- `-m`:以MB形式列出容量显示
- `-k`:以KB形式列出容量显示

示例

    #列出目前目录下的所有容量
    [root@www ~]# du
    8     ./test4
    8     ./test2
    ...(中间省略)...
    12    ./.gcond
    220   .                 <== 当前目录(.)所占有的总量
    #du不加参数，分析当前目录，目录中有很多文件没有列出，所以全部#加起来不等于(.)的总量，
    #此外，输出的数据为1K大小的容量单位。

    #同上，且将文件的容量也列出来
    [root@www ~]# du -a
    12    ./install.log.syslog
    12    ./.bash_logout
    8     ./test4
    8     ./test2
    ...(中间省略)...
    12    ./.gcond
    220   .  


    #检查根目录下的每个目录所占的容量
    [root@www ~]# du -sm /*
    7       /bin
    6       /boot
    ...中间省略...
    0       /proc
    ...中间省略...
    1       /tmp
    3859    /usr
    77      /var    
    这是一个常用的命令，如果想要检查某个目录下那个最大的子目录占用最大的容量，可以使用这个方法。

`du`命令的使用，可以查阅《鸟哥的Linux私房菜》[Page212]


## 2.2 连接文件:`ln`

Linux下的连接文件有两种：一种是类似于Windows下的快捷方式功能的文件，可以快速连接到目标文件(或目录),称为符号连接(`symbolic link`)；第二种是通过文件系统的`inode`连接来产生新的文件名，而不是产生新文件，这种成为硬连接(`hard link`)

**<font size = 3 color = red>`hard link硬连接`</font>**

`hard link`是在某个目录下新建一条文件名连接到某`inode`号码的关联记录。可以理解为多个文件名对应到同一个`inode`上。

    [root@www ~]# ln /etc/crontab           <== 创建实际连接的命令
    [root@www ~]#  ll -i /etc/crontab /root/crontab
    1912701  -rw-r--r-- 2 root root 255 Jan 6 2007 /etc/crontab 
    1912701  -rw-r--r-- 2 root root 255 Jan 6 2007 /root/crontab 

两个文件名都连接到了同一个`inode`号码，第二个字段有原来的1变成了2，表示有2个文件名连接到了这个`inode`。

上述例子中可以通过`/etc`或者`/root`目录`inode`指定的`block`找到两个不同的文件名(`/etc/corntab`和`/root/corntab`),不管使用哪个文件名均可以指到1912701这个`inode`去读取最终数据，这样的好处就是安全，将任意一个文件名删除，`inode`和`block`都是存在的。可以通过另外一个文件名来读取和写入正确的数据。

硬连接设置连接文件时，磁盘空间和`inode`数量都不会发生改变，只是在某个目录的`block`多写入一个关联数据，既不会增加`inode`，也不会耗用`block`数量（除非那个`block`满了,不过一般不会）。

硬连接的限制
1. 不能跨文件系统
2. 不能连接到目录

如果使用`hard link`连接到目录时，连接的数据需要连同被连接目录下的所有数据都建立连接。

**<font size = 3 color = red>`symbolic link符号连接`</font>**

`symbolic link`就是创建一个独立的文件，而这个文件会让数据的读取指向它连接的那个文件的文件名。由于只是利用文件来作为指向的操作，所以源文件删除之后，`symbolic link`的文件就会打开不了。

    [root@www ~]# ln -s /etc/crontab crontab2
    [root@www ~]# ll -i /etc/crontab /root/crontab2
    1912701 -rw-r--r-- 2 root root 255 Jan 6  2007   /etc/crontab
    654687  lrwxrwxrwx 1 root root 12  Oct 22 13:58  /root/crontab2 -> /etc/crontab

连接文件大小为12bytes,因为->指向的文件名`/etc/crontab`一共有12个英文字母。

`/root/crontab2`文件名`inode`读取到的文件内容仅有文件名，根据文件名连接到正确的目录`/etc/crontab`获取文件的`inode`，最终读取正确的数据。如果目标源文件被删除，那么就无法读取。

`symbolic link`相当于Windows下的快捷方式，创建的文件为一个新文件，所以会占用`inode`和`block`。

**<font size = 5 color = red>`ln`</font>**

    [root@www ~]# ln [-sf] 源文件 目标文件

- `-s`:如果不加任何参数就进行连接，那就是`hard link`，否则就是`symbolic link`
- `-f`:如果目标文件存在时，就主动将目标文件直接删除之后再创建。

示例

    #将/etc/passwd复制到/tmp下，查看inode和block
    [root@www ~]# cd /tmp
    [root@www tmp]# cp -a /etc/passwd .
    [root@www tmp]# du -sb ; df -i .        <== du -sb 计算真个/tmp下有多少bytes
    18340     .     
    Filesystem    Inodes   IUsed   IFree   IUse%  Mounted on
    /dev/hdc2     2560864  149738  2411126 6%     /


    #将/tmp/passwd制作hard link成为passwd-hd文件并查看文件内与容量
    [root@www tmp]# ln passwd passwd-hd
    [root@www tmp]# du -sb ; df -i .
    18340      .
    Filesystem    Inodes   IUsed   IFree   IUse%  Mounted on
    /dev/hdc2     2560864  149738  2411126 6%     /         <== 即使多了一个文件，整个inode和block的容量没有发生改变
    [root@www tmp]# ls -il passwd*
    586361 -rw-r--r-- 2 root root 1945 Sep 29 02:21  passwd
    586361 -rw-r--r-- 2 root root 1945 Sep 29 02:21  passwd-hd


    #将/tmp/passwd创建一个符号连接
    [root@www tmp]# ln -s passwd passwd-so
    [root@www tmp]# ls -li passwd*
    586361 -rw-r--r-- 2 root root 1945 Sep 29 02:21  passwd
    586361 -rw-r--r-- 2 root root 1945 Sep 29 02:21  passwd-hd
    586401 lrwxrwxrwx 1 root root    6 Oct 22 14:18  passwd-so ->passwd
    [root@www tmp]# du sb ; df -i .
    18346   .
    Filesystem    Inodes   IUsed   IFree   IUse%  Mounted on
    /dev/hdc2     2560864  149739  2411125 6%     /         <== inode 使用数发生了变化


    #删除源文件passwd,其他两个文件是否可以开启？
    [root@www tmp]# rm passwd
    [root@www tmp]# cat passwd-hd
    ...正常显示...
    [root@www tmp]# cat passwd-so
    cat:passwd-so:No such file or directory
    [root@www tmp]# ll passwd*
    -rw-r--r-- 2 root root 1945 Sep 29 02:21  passwd-hd
    lrwxrwxrwx 1 root root    6 Oct 22 14:18  passwd-so ->passwd     <== 符号连接的目标文件不存在，文件名会有特殊颜色显示。

当创建一个目录的时候，新的目录的连接数为2，而上层目录的连接数会增加1

    [root@www ~]# ls -ld /tmp
    drwxrwxrwx  5 root root 4096 Oct 22 14:22 /tmp
    [root@www ~]# mkdir /tmp/testing1
    [root@www ~]# ls -ld /tmp
    drwxrwxrwx  6 root root 4096 Oct 22 14:37 /tmp
    [root@www ~]# ls -ld /tmp/testing1
    drwxrwxrwx  2 root root 4096 Oct 22 14:37 /tmp/testing1

# 3 磁盘的分区、格式化、检验与挂载

当在系统中新增一块硬盘时，需要做以下操作：

1. 对磁盘进行分区，新建可用的分区
2. 对该分区进行格式化，以创建系统可用的文件系统
3. 若想要仔细一点，需要对新建好的文件系统进行检验
4. 在Linux系统上，需要创建挂载点(也即是目录)，并将它挂载上来。

## 3.1 磁盘分区、格式化、检验与挂载

**<font size = 5 color = red>磁盘分区`fdisk`</font>**

    [root@www ~]# fdisk [-l] 设备名称

- `-l`:输出后面的接的设备所有的分区内容，若仅有`fdisk -l`时，则系统将会把整个系统内能够找到的设备的分区均列出来。

示例

    [root@www ~]# df /          <== 找出可用磁盘文件名
    Filesystem      1K-blocks      Used   Available  Use%  Mounted on
    /dev/hdc2         9920624   3823168     5585388   41%   /

    [root@www ~]# fdisk /dev/hdc
    ......

该部分参阅《鸟哥的Linux私房菜》[Page217-222]



**<font size = 5 color = red>磁盘格式化`mkfs`、`mke2fs`</font>**

    [root@www ~]# mkfs [-t 文件系统格式] 设备文件名

    #将之前制作出来的/dev/hdc6格式化为ext3文件系统
    [root@www ~]# mkfs -t ext3 /dev/hdc6
    ....
    [root@www ~]# mkfs [Tab][Tab]
    mkfs     mkfs.cramfs  mkfs.ext2 mkfs.ext3 mkfs.msdos mkfs.vfat
    #连按两个[Tab]mkfs支持的文件格式如上所示，可以格式vfat

如果没有详细指定文件系统的具体选项，系统会使用默认值来格式化。如果需要指定，可以使用`mke2fs`命令

    [root@www ~]# mke2fs [-b block大小] [-i inode 数量][-L 卷标] [-cj] 设备

- `-b`:可以设置每个`block`的大小，目前支持1024,2048,4096bytes三种
- `-i`:给予一个`inode`的大小
- `-c`:检查磁盘错误，`-c`会进行快速读取测试;`-c -c`会测试读写
- `-L`:接卷标`lable`
- `-j`:本来`mke2fs`是ext2，加上`-j`之后，会主动加入journal成为ext3

示例

文件系统的卷标为`"qintian_logical"`,`block`大小指定为2048,每8192bytes分配一个`inode`,创建journal的ext3文件系统。

    [root@www ~]# mke2fs -j -L "qintian_logical" -b 2048 -i 8192 /dev/hdc6
    .....
    .....


**<font size = 5 color = red>磁盘检验`fsck`、`badblocks`</font>**

    [root@www ~]# fsck [-t 文件系统] [-ACay] 设备名称

- `-t`:如`mkfs`，指定文件系统，随着Linux系统的升级会自动分辨文件系统，可以不加这个参数
- `-A`:依据`/etc/fstab`的内容，将需要的设备扫描一次
- `-a`:自动修复检查到的有问题的扇区
- `-y`:与`-a`类似，但是有的文件系统仅支持`-y`
- `-C`:使用一个直方图显示目前的进度。
- `-f`:强制检查，如果`fsck`没有发现任何unclean，就不会进行强制检查
- `-D`:针对文件系统下的目录进行优化配置。


示例

    [root@www ~]# fsck -C -f -t ext3 /dev/hdc6
    .......

    #查看fsck支持的文件系统
    [root@www ~]# fsck[tab][tab]
    fsck    fsck.cramfs fsck.ext2 fsck.ext3 fsck.msdos fsck.vfat

这个命令通常只有root和文件系统出现问题的时候才会使用，正常情况下使用，会对系统造成危害。在检查的时候会造成部分文件系统的损坏，所以执行`fsck`时，被检查的分区不可以挂载到系统上，需要在卸载的状态。


**<font size = 5 color = red>磁盘的挂载与卸载`mount`、`umount`</font>**

挂载前需要先确定：

1. 单一文件系统不应被重复挂载在不同的挂载点(目录)中
2. 单一目录不应重复挂载多个文件系统
3. 作为挂载点的目录理论上应该都是空目录。

如果挂载的目录不是空的，那么挂载了文件系统之后，原先目录下的东西就会被隐藏，暂时消失。

    [root@www ~]# mount -a
    [root@www ~]# mount [-l]
    [root@www ~]# mount [-t 文件系统][-L label名][-o 额外选项][-n] 设备文件名 挂载点

- `-a`:依照配置文件`/etc/fstab`的数据将所有未挂载的磁盘都挂载上来
- `-l`:单纯输入`mount`会显示目前挂载的信息，加上`-l`可增加Label名称
- `-t`:与`mkfs`类似，可以加上文件系统种类来指定欲挂载的类型，常见的Linux支持的文件类型有`ext2`、`ext3`、`vfat`、`reiserfs`、`iso9660`（光盘格式）、`nfs`、`cifs`、`smbfs`(网络文件类型)
- `-n`:在默认的情况，系统会将实际挂载的情况写入`/etc/matab`中，以利其他程序运行，在某些情况下，刻意不写入，需要加`-n`
  

示例

    #挂载ext2/ext3文件系统，用默认的方式将刚才创建的/dev/hdc6挂载到 /mnt/hdc6上
    [root@www ~]# mkdir /mnt/hdc6
    [root@www ~]# mount /dev/hdc6 /mnt/hdec6
    [root@www ~]# df
    Filesystem     1K-blocks   Used   Available Use% Mounted on
    ...(中间省略)...
    /dev/hdc6        1976312   42072   1833836   3%  /mnt/hdc6

不用加`-t`指定文件系统类型，Linux系统支持的文件系统的驱动程序在如下的目录中。`/lib/modules/$(uname -r) /kernel/fs/`,因为几乎所有的文件系统都会有`super block`，我们的Linux会分析`super block`来搭配自己的驱动程序去挂载。

    #查看目前已经挂载的信息，包含各种文件系统的Label
    [root@www ~]# mount -l
    ...(中间省略)...
    /dev/hdc6 on /mnt/hdc6 type ext3(rw)    [qintian_logical]


**<font size =3 color = yellow>挂载光盘</font>**

光盘已经被淘汰，不再赘述

**<font size =3 color = yellow>格式化和挂载软盘</font>**

软盘已经被淘汰，不再赘述，具体可以参阅《鸟哥的Linux私房菜》[Page228]

**<font size =3 color = yellow>挂载U盘</font>**

    #找出U盘的设备文件名，并挂载到/mnt/flash目录中
    [root@www ~]# fdisk -l
    ...(中间省略)...
    Device Boot  Start     End  Blocks  Id System
    /dev/sda1        1    4745  8118260 b   W95  FAT32
    [root@www ~]# mkdir /mnt/flash 
    [root@www ~]# mount -t vfat - o iocharset=cp950 /dev/sda1  /mnt/flash
    [root@www ~]# df
    Filesystem     1K-blocks     Used   Available Use% Mounted on
    ...(中间省略)...
    /dev/sda1      8102416     4986228    3116188  62%   /mnt/flash

在`vfat`文件格式中可以使用`iocharset`来指定语系，中文语系是`cp950`。

**<font size =3 color = yellow>重新挂载根目录与挂载特定的目录</font>**

如果根目录`/`的挂载参数需要改变，或者根目录出现只读的情况，如何重新挂载，最可能得处理方式就是重新启动，也可以手动将根目录`/`重新挂载

    #将/重新挂载，加入参数rw与auto
    [root@www ~]# mount -o remount,rw,auto/

也可以利用`mount`将某个目录挂载到另外一个目录，不是挂载文件系统。

    #将/home挂载到/mnt/home目录下
    [root@www ~]# mkdir /mnt/home
    [root@www ~]# mount --bind /home /mnt/home
    [root@www ~]# ls -lid /home /mnt/home
    2 drwxr-xr-x  6 root root 4096 Sep 29 02:21  /home
    2 drwxr-xr-x  6 root root 4096 Sep 29 02:21  /mnt/home
    [root@www ~]# mount -l 
    /home on /mnt/home type none (rw,bind)

通过`mount --bind`功能，可以将某个目录挂载到其他的目录中去，而不是整块文件系统，进入`/mnt/home`就是进入`/home`的意思。



**<font size =3 color = yellow>将设备文件卸载`umount`</font>**

    [root@www ~]# umount [-fn] 设备文件名或者挂载点

- `-f`:强制卸载，可用在类似网络文件系统`NFS`无法读取到的情况下
- `-n`:不更新`/etc/mtab`的情况下卸载

示例

    #将之前挂载的文件系统全部卸载
    [root@www ~]# mount
    ...(中间省略)...
    /dev/hdc6   ...
    /dev/sda1   ...

    [root@www ~]# umount /dev/hdc6
    [root@www ~]# umount /media/cdrom
    [root@www ~]# umount /dev/fd0
    [root@www ~]# umount /mnt/home

注意，卸载需要离开文件系统的挂载目录。可以回到根目录进行卸载

**<font size =3 color = yellow>使用`Label name`进行挂载的方法</font>**

除了设备文件名，还可以使用`Label name`进行挂载，上面卸载的`/dev/hdc6`的卷标名为`qintian_logical`。可以先使用`dumpe2fs`这个命令来查询一下

    [root@www ~]# dump2fs -h /dev/hdc6
    Filesystem volume name: qintian_logical
    ...(下面省略)...
    [root@www ~]# mount -L "qintian_logcial" /mnt/hdc6

这种方法系统不必知道该文件系统所在的接口与磁盘文件名。


**<font size = 5 color = red>磁盘参数的修改</font>**

**<font size =3 color = yellow>`mknod`</font>**

在Linux中，所有的设备都可以用文件来代表，具体是通过文件的`major`和`minor`数值来代替。

    [root@www ~]# ll /dev/hdc*
    brw-r----- 1 root disk 22, 0 Oct 24 15:55 /dev/hdc
    ......

其中，22为主设备代码`major`，0-6为次设备代码`minor`，Linux内核认识的设备数据就是通过这两个数值来决定。

    [root@www ~]# mknod 设备文件名 [bcp] [major] [minor]

- `b`:将设备名称设置成为一个外部存储设备文件，如硬盘等
- `c`:设置设备名称成为一个外部输入设备文件，如鼠标/键盘等
- `p`:设置设备名称成为一个FIFO文件。

示例

    #将/dev/hdc10设备代码设置为22,10
    [root@www ~]# mknod /dev/hdc10 b 22 10
    [root@www ~]# ll /dev/hdc10
    brw-r----- 1 root disk 22, 10 Oct 24 15:55 /dev/hdc10

其余命令，请参阅《鸟哥的Linux私房菜》[Page232-233]


# 4 设置开机挂载

## 4.1 开机挂载`/etc/fstab`以及`/etc/mtab`

首先说明系统挂载的限制

- 根目录是必须挂载的，而且一定要先于其他的挂载点被挂载起来
- 其他的挂载点必须是已经新建的目录，可以任意指定，但是一定要遵守必须的系统目录架构原则
- 所有挂载点在同一时间之内，只能挂载一次
- 所有分区在同一个时间之内，只能挂载一次
- 如果进行卸载，必须先将工作目录移到挂载点之外。

示例

    [root@www ~]# cat /etc/fstab
    #Device     Mount point  filesystem  parameters  dump  fsck
    LABEL=/1        /        ext3         defaults     1   1
    LABEL=/home     /home    ext3         defaults     1   2
    LABEL=/boot     /boot    ext3         defaluts     1   2
    tmpfs           /dev/shm tmpfs        defaults     0   0
    ......

上述LABEL开头的部分与实际磁盘有关，其他则是虚拟文件系统或者与内存交换空间有关。


`/etc/fstab(file system table)`这个文件，就是将我们利用`mount`命令进行挂载的时候，将所有的参数写入到这个文件就可以了。

下面介绍`/etc/fstab`的每一列字段的含义

1. 磁盘设备文件名或者该设备的`Label`
2. 挂载点
3. 磁盘分区的文件系统
4. 文件系统参数(具体参数参阅《鸟哥的Linux私房菜》[Page235])
5. 能够被`dump`备份命令作用
6. 是否以`fsck`检验扇区

`dump`是一个作为备份的命令，可以通过`fstab`来指定哪个文件系统必须要进行`dump`备份，0表示不要做备份，1表示要每天进行`dump`操作，2代表其他不定日期的`dump`操作，通常不是0就是1

`fsck`0表示要检验，1表示最早检验，一般只有根目录会设置1，2也是要检验，其他的目录一般设置成为2。

`/etc/fstab`是开机时候的配置文件，实际的文件系统的挂载是记录到`/etc/tab`和`/proc/mounts`两个文件中的，每次更改文件系统的挂载时，也会同时更动这两个文件。当`/etc/fstab`出现错误，无法顺利开机成功时，可以输入以下命令

    [root@www ~]# mount -n -o remount,rw /

重新挂载根目录。

## 4.2 特殊设备loop挂载（镜像文件不刻录就挂载使用）

**<font size = 5 color = red>挂载光盘/DVD镜像文件</font>**

当下载了Linux或者其他所需光盘/DVD的镜像文件，不一定要通过刻录成光盘才能够使用该文件里面的数据，可以通过`loop`设备来进行挂载使用。

    #将整个CentOS 5.2的DVD镜像文件挂载到测试机上
    [root@www ~]# ll -h /root/centos 5.2_x86_64.iso
    -rw-r--r-- 1 root root 4.3G Oct 27 17:34 /root/centos5.2_x86_64.iso
    [root@www ~]# mkdir /mnt/centos_dvd
    [root@www ~]# mount -o loop /root/centos 5.2_x86_64.iso  /mnt/centos_dvd
    [root@www ~]# df
    Filesystem                     1K-blocks      Used  Available Use%    Mounted on
    /root/centos 5.2_x86_64.iso      4493125   4493152          0 100%  /mnt/centos_dvd
    [root@www ~]# ll /mnt/centos_dvd
    total 584
    drwxr-xr-x 2 root root 522240 Jun 24 00:57 CentOS    <== DVD的内容
    ...(下面省略)...
    [root@www ~]# umount /mnt/centos_dvd/         <== 卸载

# 5 内存交换空间(`swap`)的构建

在安装Linux时一定需要的两个分区，一个是根目录，还有一个就是`swap`，`swap`的功能就是在物理内存不足的情况下，所造成的内存扩展记录的功能。

当系统内存不足时，为了让后续的程序顺利运行，因此在内存中暂时不使用的程序和数据就会被挪到`swap`中，空出来的内存就会给需要执行的程序加载，`swap`是用硬盘来暂存内存中的信息，当使用到`swap`时，主机上的硬盘灯就会闪个不停。


## 5.1 使用物理分区构建`swap`

1. 分区：使用`fdisk`在硬盘中分出一个分区给系统作为`swap`
2. 格式化：利用新建`swap`格式的`mkswap 设备文件名`将该分区格式化为`swap`格式。
3. 使用：将该`swap`设备启动，方法为`swapon 设备文件名`
4. 查看：最后通过`free`这个命令来查看内存的使用情况

示例

    #分区
    [root@www ~]# fdisk /dev/hdc
    ...(中间省略)...            <== 参考[Page239]
    [root@www ~]# partprobe    <== 让内核更新分区表
    #构建swap格式
    [root@www ~]# mkswap /dev/hdc7
    Setting up swapspace version 1, size = 263172 kB  
    #开始查看和加载
    [root@www ~]# free 
            total        used    free  shared   buffers   cached
    Mem     742664      684592   58072       0     43820   497144
    -/+ buffers/cache:  143628   599036
    Swap:   1020088         96   1019992
    #有742664KB的物理内存，使用684592KB剩余58072KB，
    #使用掉的内存有43820KB/497144KB用在缓冲/快取的用途中，
    #至于swap已经存在了1020088KB
    [root@www ~]# swapon /dev/hdc7
    [root@www ~]# free
            total        used    free  shared   buffers   cached
    Mem     742664      684712   57952       0     43872   497180
    -/+ buffers/cache:  143660   599004
    Swap:   1277088         96   1276992       <== 增加

## 5.2 使用文件构建`swap`

具体内容参阅《鸟哥的Linux私房菜》[Page240]

现在的计算机基本用不到`swap`，内存都很大，但是针对服务器或者工作站这些常年上线的系统来说，无论如何，`swap`还是必须要创建的。


# 6 文件系统的特殊查看与操作

## 6.1 `boot sector`和`super block`的关系

参阅

## 6.2 磁盘空间的浪费问题

参阅
