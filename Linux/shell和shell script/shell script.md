# 1、`shell script`（程序化脚本）

`shell`是一个命令行界面下让用户和系统进行沟通的一个工具接口，`shell script`是针对`shell`写的程序化脚本。

> `shell script`是利用`shell`功能写的一个程序，这个程序使用纯文本文件，将一些`shell`的语法和命令（含外部命令）写在里面，搭配正则表达式，管道命令与数据流重定向等功能，达到想要处理的目的。

`shell script`最简单的功能就是将多个命令写在一起，处理复杂的操作。


## 1.1 `shell script`的编写与执行

- `shell`如果读取到一个`Enter`符号，则会尝试执行该行命令
- `shell`如果一行内容太多，就使用`\Enter`来扩展至下一行
- `#`作为批注，其后文字被视为批注而被忽略

直接命令执行方式(`shell`具备可读可执行权限):

- 绝对路径:使用`/home/dmtsai/shell.sh`执行命令
- 相对路径:在`/home/dmtsai/`目录下，使用`./shell.sh`执行
- 变量`PATH`功能:将`shell.sh`放在`PATH`指定的目录内，例如`~/bin/`，直接`shell.sh`执行

以`bash`进程来执行:通过`bash shell.sh`或者`sh shell.sh`来执行。

    [root@www ~]# mkdir scripts; cd scripts
    [root@www scripts]# vi sh01.sh
    #!/bin/bash
    # Program
    #       This program shows "Hello World!"  in your screen
    # History:
    # 2005/08/23  Vbird First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH
    echo -e "Hello World! \a \n"
    exit 0


- `#!bin/bash`声明这个`script`使用的`shell`名称，声明这个文件内的语法使用`bash`的语法，当这个程序被执行的时候，就能够加载`bash`的相关环境配置文件，并且执行`bash`来使下面的命令可以执行。
- 除了第一行的`#!`是用来声明`shell`之外，其他的`#`都是批注。
- 主要环境变量`PATH`与`LANG`的声明，就可以让程序在进行的时候可以直接执行外部命令而不用写绝对路径。
- `echo`那一行就是主要程序部分。
- `exit`使程序中断，传回一个值给系统，执行完这个脚本之后，接着执行`echo $?`即可以得到0值，因此可以利用`exit n`来自定义错误信息

执行结果

    [root@www script]# sh sh01.sh
    Hello World !

在每个`script`的文件头处记录好：

- `script`的功能
- `script`的版本信息
- `script`的作者和联络方式
- `script`的版权声明方式
- `script`的`History`(历史记录)
- `script`内较特殊的命令，使用绝对路径的方式来执行
- `script`执行时需要的环境变量预先声明和设置。
- 最好以`tab`进行缩排
- 使用`vim`而不是`vi`，`vim`会有额外的语法检验机制


# 2、简单的`shell script`练习

**<font size = 3 color =red>交互式脚本:变量内容由用户决定</font>**

    [root@www script]# vi sh02.sh
    #!bin/bash
    # Program:
    #   User inputs his first name and last name. Program shows his full name.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    read -p "Please input your first name:" firstname       #提示用户输入
    read -p "Please input your last name:" lastname         #提示用户输入
    echo -e "\nYour full name is: $firstname $lastname"     #结果由屏幕输出

**<font size = 3 color =red>随日期变化：利用日期进行文件的创建</font>**

文件名类似`backup.20231018.data`文件

    [root@www script]# vi sh03.sh
    #!bin/bash
    # Program:
    #   User inputs his first name and last name. Program shows his full name.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    #1.让用户输入文件名，并取得fileuser这个变量
    echo -e "I will use 'touch' command to create 3 files"
    read -p "Please input your filename: "fileuser      #提示用户输入

    #2.为了避免用户随意按Enter，利用变量功能分析文件名是否有设置
    filename=${fileuser:-"filename"}        #开始判断是否有配置文件名

    #3.开始利用date命令来取得所需要的文件名
    date1=$(date --date='2 days ago' +%Y%m%d)   #前两天的日期
    date2=$(date --date='2 days ago' +%Y%m%d)   #前一天的日期
    date3=$(date +%Y%m%d)                       #今天的日期
    file1=${filename}${date1}
    file2=${filename}${date2}
    file3=${filename}${date3}

    #4.创建文件名
    touch "$file1"
    touch "$file2"
    touch "$file3"

**<font size = 3 color =red>数值运算:简单的加减乘除</font>**

    [root@www script]# vi sh04.sh
    #!bin/bash
    # Program:
    #   User inputs his first name and last name. Program shows his full name.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    echo -e "You SHOULD input 2 numbers, I will cross them! \n"
    read -p "first number: " firstnu
    read -p "second number: " secnu
    total=$(($firstnu*$secnu))
    echo -e "\nThe result of $firstnu x $secnu is ==> $total"

`var=$((运算内容))`

    [root@www script]# echo $((13%3))
    1

## 2.2、`script`的执行方式的区别(`source`,`shscrpit`,`./script`)

**<font size = 3 color =red>利用直接执行的方式来执行`script`</font>**

不管是通过直接命令执行（绝对路径/相对路径/$PATH）或者利用`bash`(或`sh`)来执行脚本，该`script`都会使用一个新的`bash`环境来执行脚本内的命令。即该`script`是在子进程的`bash`内执行的。子进程完成后，子进程内的各项变量或操作会结束而不会传回到父进程中。

以`sh02.sh`为例

    [root@www script]# echo $firstname $lastname
        <== 这两个变量并不存在
    [root@www script]# sh sh02.sh
    Please input your first name: Xbid
    Please input your last name:Tsai

    Your full name is Xbid Tsai
    [root@www script]# echo $firstname $lastname
        <== 这两个变量不存在

**<font size = 3 color =red>利用`source`来执行脚本:在父进程中执行</font>**

    [root@www script]# source sh02.sh
    Please input your first name: Xbid
    Please input your last name:Tsai

    Your full name is Xbid Tsai
    [root@www script]# echo $firstname $lastname
    Xbid Tsai


# 3、判断式

## 3.1、利用`test`命令的测试功能

`test`可以用来检验某些文件或者是相关的属性

如用来检查`/dmtsai`是否存在时

    [root@www ~]# test -e /dmtsai
        <== 不显示任何值
    [root@www ~]# test -e /dmtsai && echo "exist" || echo "Not exist"
    Not exist       <== 结果显示不存在

关于某个文件名的“文件类型”判断，如`test -e filename`
|测试标志|代表意义|
|-|-|
|`-e`|该文件名是否存在(常用)|
|`-f`|该文件名是否存在且为文件(常用)|
|`-d`|该文件名是否存在且为目录(常用)|
|`-b`|该文件名是否存在且为一个`block device`设备|
|`-c`|该文件名是否存在且为一个`character device`设备|
|`-S`|该文件名是否存在且为一个`Socket`文件|
|`-p`|该文件名是否存在且为一个`FIFO(pipe)`设备|
|`-L`|该文件名是否存在且为一个连接文件|


关于文件的权限检测，如`test -r filename`表示可读否（但`root`权限常有例外）

|测试标志|代表意义|
|-|-|
|`-r`|检测该文件名是否存在且具有“可读”权限|
|`-w`|检测该文件名是否存在且具有“可写”权限|
|`-x`|检测该文件名是否存在且具有“可执行”权限|
|`-u`|检测该文件名是否存在且具有“SUID”权限|
|`-g`|检测该文件名是否存在且具有“SGID”权限|
|`-k`|检测该文件名是否存在且具有“Sticky bit”权限|
|`-s`|检测该文件名是否存在且为非空白文件|


两个文件之间的比较，如`test file1 -nt file2`

|测试标志|代表意义|
|-|-|
|`-nt`|(newer than)判断`file1`是否比`file2`新|
|`-ot`|(older than)判断`file1`是否比`file2`旧|
|`-ef`|判断`file1`和`file2`是否为同一文件，可用在判断`hard link`的判定上，主要用于判断两个文件是否均指向同一个`node`|

关于两个整数之间的判定，例如`test n1 -eq n2`

|测试标志|代表意义|
|-|-|
|`-eq`|两数值相等|
|`-ne`|两数值不等|
|`-gt`|`n1`大于`n2`|
|`-lt`|`n1`小于`n2`|
|`-ge`|`n1`大于等于`n2`|
|`-le`|`n1`小于等于`n2`|


判定字符串的数据

|测试标志|代表意义|
|-|-|
|`test -z string`|判定字符串是否为0，若`string`为空，则为`true`|
|`test -n string`|判定字符串是否非0，若`string`为空，则为`false`|
|`test str1=str2`|判定`str1`是否等于`str2`,若相等，则回传`true`|
|`test str1!=str2`|判定`str1`是否不等于`str2`,若相等，则回传`false`|

多重条件判断，例如`test -r filename -a -x filename`
|测试标志|代表意义|
|-|-|
|`-a`|两个条件同时成立,例如`test -r file -a -x file`则只有当file同时具有`r`和`x`权限的时候，才会回传`true`|
|`-o`|任何一个条件成立，例如`test -r file -o -x file`则file具有`r`或者`x`权限的时候，就会回传`true`|
|`!`|反向状态，例如`test !x file`,当`file`不具有`x`时，回传`true`|

    [root@www scripts]# vi sh05.sh
    #!bin/bash
    # Program:
    #   User inputs a filename, Program will check the following: 1)exist? 2)file/directory? 3)file permessions
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    #1.让用户输入文件名，并且判断用户是否真的有输入字符串
    echo -e "Please input a filename, I will check the filename's type and permession \n\n"
    read -p "Input a filename : " filename
    test -z $filename && echo "You MUST input a filename." && exit 0

    #2.判断文件是否存在，若不存在则显示信息并结束脚本
    test ! -e $filename && echo "The filename '$filename' DO NOT exist" && exit 0

    #3.开始判断文件类型和属性
    test -f $filename && filetype="regular file"
    test -d $filename && filetype="directory"
    test -r $filename && perm="readable"
    test -w $filename && perm="$perm writeable"
    test -x $filename && perm="$perm executable"

    #4.开始输出信息
    echo "The filename: $filename is a $filetype"
    echo "And the permissions are: $perm"
 
## 3.2、利用判断符号`[]`

判断变量`HOME`是否为空

    [root@www ~]# [ -z "$HOME"]; echo $?

中括号可以用在很多地方，包括通配符和正则表达式，在`bash`中使用中括号作为判断符号时，中括号的两端需要有空格符来进行分隔。

如判断`HOME`和`MAIL`变量是否相同

    [root@www ~]# [ "$HOME" == "$MAIL" ]

- 在中括号的每个组件都需要有空格键来分隔
- 在中括号的变量，最好都以双引号括起来
- 在中括号的敞亮，最好都以单或双引号括起来

判断符号示例

    [root@www script]# vi sh06.sh
    #!bin/bash
    # Program:
    #    This program shows the user's choice
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    read -p "Please input (Y/N): " yn
    [ "$yn" == "Y" -o "$yn" == "y" ] && echo "OK, continue" && exit 0
    [ "$yn" == "N" -o "$yn" == "n" ] && echo "Oh interrupt!" && exit 0
    echo "I don't know what your choice is" && exit 0

`-o`表示或连接判断
    

## 3.3、`shell script`的默认变量(`$0`,`$1`...)

使用`read`命令需要从键盘中手动输入，通过命令后接参数，则不需要再次手动输入。在`shell script`中，执行脚本的名称为`$0`这个变量，第一个接的参数为`$1`，第二个接的参数为`$2`……除此之外，还有一些比较特殊的变量

- `$#`代表后面接的参数的个数
- `$@`代表"$1"、"$2"、"$3"、"$4"，每个变量是独立的
- `$*`代表`"$1c$2c$3c$4"`，其中`c`为分隔字符，默认为空格键，所以本例中代表`"$1 $2 $3 $4"`


示例

    [root@www script]# vi sh07.sh
    #!bin/bash
    # Program:
    #    This program shows the script name, parameters...
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    echo "The script name is            ==> $0"
    echo "Total parameters number is    ==> $#"
    [ "$#" -lt 2 ] && echo "The number of parameter is less than 2. Stop here." \
    && exit 0
    echo "Your whole parameter is ==> '$@'"
    echo "The 1st parameter       ==> $1"
    echo "The 2nd parameter       ==> $2"

执行结果如下

    [root@www script]# sh sh07.sh theone haha quot
    The script name is      ==> sh07.sh
    Total parameter number is ==> 3
    Your whole parameter is   ==> 'theone haha quot'
    The 1st parameter         ==> theone
    The 2nd parameter         ==> haha


`shift`：造成参数变量号码后移,后面接数字表示拿掉最前面几个参数的意思

    [root@www script]# vi sh08.sh
    #!bin/bash
    # Program:
    #    This program shows the effect of shift function
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    echo "Total parameters number is    ==> $#"
    echo "Your whole parameter is ==> '$@'"
    shift 
    echo "Total parameters number is    ==> $#"
    echo "Your whole parameter is ==> '$@'"
    shift
    echo "Total parameters number is    ==> $#"
    echo "Your whole parameter is ==> '$@'"

执行结果如下

    [root@www script]# sh sh08.sh one two three four five six               <== 给予六个参数
    Total parameter number is ==> 6
    Your parameter is         ==> 'one two three four five six'
    Total parameter number is ==> 5
    Your parameter is         ==> 'two three four five six'
    Total parameter number is ==> 2
    Your parameter is         ==> 'five six'


# 4、条件判断式

## 4.1、`if...then`

基础语法如下

    if [条件判断式]; then
        当判断条件成立时，可以进行的命令工作内容
    fi      <== if反过来写，表示结束if

当有多个条件需要判断的时候

- `&&`代表AND
- `||`代表or(`sh06.sh`脚本中的`-o`可以改写为[ "$yn" == "Y" ]||[ "$yn" == "y" ])
  

将`sh06.sh`改成`if...then`的形式

    [root@www scripts]# cp sh06.sh sh06-2.sh
    [root@www scripts]# vi sh06-02.sh
    #!bin/bash
    # Program:
    #    This program shows the user's choice
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    read -p "Please input (Y/N):" yn

    if [ "$yn" == "Y" ]||[ "$yn" == "y" ];then
        echo "OK, continue"
        exit 0
    fi

    if [ "$yn" == "N" ]||[ "$yn" == "n" ];then  
        echo "Oh interrupt"
        exit 0
    fi
    echo "I don't know what your choice is " && exit 0


**<font size = 3 color =red>多重、复杂条件判断式</font>**

    #一个条件判断
    if [条件判断式]; then
        条件成立时，执行命令
    else 
        条件不成立时，执行的命令
    fi

    #多个条件判断
    if [条件判断式1]; then
        条件判断式1成立时，执行命令
    elif [条件判断式2]; then
        条件判断式2成立时，执行命令
    else 
        条件判断式1和2均不成立时，执行命令
    fi

将`sh06.sh`再进行改写

    [root@www scripts]# cp sh06.sh sh06-3.sh
    [root@www scripts]# vi sh06-02.sh
    #!bin/bash
    # Program:
    #    This program shows the user's choice
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    read -p "Please input (Y/N):" yn

    if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
        echo "OK,continue" && exit 0
    elif [ "$yn" == "N" ] || [ "$yn" == "n" ]; then
        echo "Oh, interrupt" && exit 0 
    else
        echo "I don't know what your choice is" && exit 0
    fi

再结合之前提到的不从键盘输入，使用参数功能

    [root@www scripts]# vi sh09.sh
    #!bin/bash
    # Program:
    #    Check $1 is equal to "hello"
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH
    
    if [ "$1" == "hello" ]; then
        echo "Hello, how are you?"
    elif [ "$1" == "" ]; then
        echo "You must input parameters, ex>{$0 someworld}"
    else
        echo "The only parameter is 'hello', ex>{$0 hello}"
    fi

使用`netstat -tuln`可以获取目前主机启动的服务

写一个脚本判断主机有没有启动主要的网络服务端端口

    [root@www scripts]# vi sh10.sh
    #!bin/bash
    # Program:
    #    Using netstat and grep to detect www,ssh,ftp and mail service
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    1.先做一些告知的操作
    echo "Now, I will detect your linux server's service!"
    echo -e "The www,ftp,ssh,and mail will be detected!\n "

    2.开始进行测试工作并输出信息
    testing=$(netstat -tuln | grep ":80")   #检测port 80在否
    if [ "$testing" != "" ]; then
        echo "WWW is running in your system."
    fi
    testing=$(netstat -tuln | grep ":22")   #检测port 22在否
    if [ "$testing" != "" ]; then
        echo "SSH is running in your system."
    fi
    testing=$(netstat -tuln | grep ":21")   #检测port 21在否
    if [ "$testing" != "" ]; then
        echo "FTP is running in your system."
    fi
    testing=$(netstat -tuln | grep ":25")   #检测port 25在否
    if [ "$testing" != "" ]; then
        echo "Mail is running in your system."
    fi



## 4.2、`利用case...esac判断`

    case $变量名称 in 
        "第一个变量内容")
            程序段
            ;;              <== 每个类型结尾用两个连续的分号来处理
        "第二个变量内容")
            程序段
            ;;
        *)                  <== 最后一个变量内容都会用*来代表所有其他值，不包含第一个变量内容与第二个变量内容的其他程序执行段
            exit 1
            ;;
    esac                    <== case反过来写代表结束

拿`sh09.sh`的案例来进行修改

    [root@www scripts]# vi sh09-2.sh
    #!bin/bash
    # Program:
    #    Show "Hello" from $1... by using case ... esac
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    case $1 in 
        "hello")
            echo "Hello,how are you?"
            ;;
        "")
            echo "You must input parameters, ex>{$0 someword}"
            ;;
        *)     #其实就相当于通配符，0-无穷多个任意字符的意思
            echo "Usage $0 {hello}"
            ;;
    esac


让用户输入one,two,three,并且将用户的变量显示到屏幕上，如果不是one,two,three时，就告知用户仅有这三种选择

    [root@www scripts]# vi sh12.sh
    #!bin/bash
    # Program:
    #    This script only accepts the following parameters: one,two or three.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    echo "This program will print your selection!"
    #read -p "Input your choice:" choice        #暂时取消，可以替换
    #case $choice in 
    case $1 in 
        "one")
            echo "Your choice is ONE"
            ;;
        "two")
            echo "Your choice is TWO"
            ;;
        "three")
            echo "Your chi=oice is THREE"
            ;;
        *)
            echo "Usage $0 {one|two|three}"
            ;;
    esac

## 4.3、`利用function功能`

语法如下

    function fname(){
        程序段
    }

`shell script`的执行方式是由上而下，由左往右，因此在`shell script`当中的`function`的设置一定要在程序的最前面

将`sh12.sh`改写，自定义一个`printit`的函数来使用

    [root@www scripts]# vi sh12-02.sh
    #!bin/bash
    # Program:
    #    Use function to repeat information.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    function printit(){
        echo -n "Your choice is "           #加上n可以不断行继续在同一行显示
    }

    echo "This program will print your selection !"
    case $1 in 
        "one")
            printit; echo $1 | tr 'a-z' 'A-Z'   #将参数做大小写转换
            ;;
        "two")
            printit; echo $1 | tr 'a-z' 'A-Z'   #将参数做大小写转换
            ;;
        "three")
            printit; echo $1 | tr 'a-z' 'A-Z'   #将参数做大小写转换
            ;;
        *)
            echo "Usage $0 {one|two|three}"
            ;;
    esac

`function`是有内置变量，它的内置变量与`shell script`很类似。函数名称代表是`$0`，后续接的变量是以`$1`、`$2`…来代替的

    [root@www scripts]# vi sh12-03.sh
    #!bin/bash
    # Program:
    #    Use function to repeat information.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    function printit(){
        echo "Your choice is $1"     #这个$1必须参考下面命令的执行
    }

    echo "This program will print your selection !"
    case $1 in 
        "one")
            printit 1  #printit命令后需要接参数
            ;;
        "two")
            printit 2 
            ;;
        "three")
            printit 3
            ;;
        *)
            echo "Usage $0 {one|two|three}"
            ;;
    esac


# 5、循环`loop`

## 5.1、`while do done`,`until do done`(不定循环)

    while [ condition ]         
    do 
        程序段落    
    done                 <== 条件成立时，进行循环，知道condition不成立时停止
    或
    until [ condition ]
    do 
        程序段落
    done                <== 当condition条件成立时，就终止循环，否则持续循环

写一个脚本，需要用户输入`yes`或`YES`才结束程序的执行，否则就一直告知用户输入祖父穿

    [root@www scripts]# vi sh13.sh
    #!bin/bash
    # Program:
    #    Repeat question until input correct answer.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    while [ "$yn" != "yes" -a "$yn" != "YES" ]
    do 
        read -p "Please input yes/YES" to stop this program: " yn
    done
    echo "OK! you input the correct answer."

同理也可以使用`until`来进行替换

写一个脚本，计算1到100的和

    [root@www scripts]# vi sh14.sh
    #!bin/bash
    # Program:
    #    Use loop to calculate "1+2+...+100" result.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    s=0
    i=0
    while [ "$i" != 100 ]
    do 
        i=$(($i+1))   #每次i都会加1
        s=$(($s+$i))  #每次都会累加1次
    done
    echo "The result of '1+2+...+100' is ==> $s"

## 5.2、`for...do...done`(固定循环)

    for var in con1 con2 con3
    do 
        程序
    done

第一次循环时，`$var`的内容是`con1`;第二次循环时，`$var`的内容是`con2`;...

    [root@www scripts]# vi sh15.sh
    #!bin/bash
    # Program:
    #    Using for... loop to print 2 animals.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    for animal in dog cat elephant
    do 
        echo "There are ${animal}s"
    done


    [root@www scripts]# vi sh16.sh
    #!bin/bash
    # Program:
    #    Use id,finger command to check system account's information.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    users=$(cut -d ':' -f1 /etc/passwd)     #开始获取账号名称
    for username in $users
    do 
        id $username
        finger $username
    done

写一个脚本，显示出`192.168.1.1`~`192.168.1.100`的主机是够能与本机相通。

    [root@www scripts]# vi sh17.sh
    #!bin/bash
    # Program:
    #    Use ping command to check the network's PC state.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    network="192.168.1"                 #定义一个域的前面部分
    for sitenu in $(seq 1 100)          #seq为sequence的缩写
    do 
        #下面的语句取得ping的回传值是正确的还是失败的
        ping -c 1 -w 1 ${network}.${sitenu} &> /dev/null && result=0 || result=1
        #开始显示结果是正确的启动还是错误的没有连同
        if [ "$result" == 0 ]; then
            echo "Server ${network}.${sitenu} is UP"
        else
            echo "Server ${network}.${sitenu} is DOWN"
        fi
    done


写一个脚本，让用户输入某个目录文件名，然后找出某目录内的文件名权限

    [root@www scripts]# vi sh18.sh
    #!bin/bash
    # Program:
    #    Use input dir name, I find the permission of files.
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    #1.先查看这个目录是否存在
    read -p "Please input a directory: " dir
    if [ "$dir" == "" -o ! -d "$dir" ]; then
        echo "The $dir is NOT exist in your system."
        exit 1
    fi

    #2.开始测试文件
    filelist=$(ls $dir)         #列出该目录下的所有的文件名
    for filename in #filelist
    do
        perm=""
        test -r "$dir/$filename" && perm="perm readable"
        test -w "$dir/$filename" && perm="perm writeable"
        test -x "$dir/$filename" && perm="perm executable"
        echo "The file $dir/$filelist's permission is $perm" 

## 5.3、`for...do...done`的数值处理

    for ((初始值;限制值;执行步长))
    do  
        程序段
    done

写一个脚本，将1到用户输入的数累加

    [root@www scripts]# vi sh19.sh
    #!bin/bash
    # Program:
    #    Try do calculate 1+2+...+${your_input} .
    # History:
    # 2005/08/23 Vbird    First release
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    export PATH

    read -p "Please input a number,I will count for 1+2+...+your_input: " nu

    s=0
    for ((i=1; i<=nu; i+1))
    do 
        s=$(($s+$i))
    done
    echo "The result of '1+2+3+...+$nu' is ==> $s"


# 6、`shell scrpit`的追踪和调试

在执行脚本之前，，怕出现语法错误问题，可以通过`bash`的相关参数来进行判断。

    [root@www ~]# sh [-nvx] scripts.sh

- `-n`:不要执行`script`仅查询语法的问题
- `-v`:在执行`script`之前，先将`script`的内容输出到屏幕上
- `-x`:将使用到的`script`内容显示到屏幕上

示例

    #测试sh16.sh有无语法问题
    [root@www ~]# sh -n sh16.sh
    #若语法没有问题，则不会显示任何信息

    #将sh15.sh的执行过程全部列出来
    [root@www ~]# sh -x sh15.sh
    + PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
    + export PATH
    + for animal in dog cat elephant
    + echo 'There are dogs...'
    There are dogs...
    + for animal in dog cat elephant
    + echo 'There are cats...'
    There are cats...
    + for animal in dog cat elephant
    + echo 'There are elephants...'
    There are elephants...

`+`后面的都是命令串

`sh -x`可以用来判断程序代码执行到哪一段时会出现相关的信息。
  

