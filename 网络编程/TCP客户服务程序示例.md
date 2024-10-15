# TCP客户/服务器程序示例

## 1、程序概述

1. 客户从标准输入读入一行文本，并写给服务器
2. 服务器从网络输入读入这行文本，并回射给客户
3. 客户从网络输入读入这行回射文本，并显示在标准输出上

![](https://ask.qcloudimg.com/http-save/1205848/b0xhrwqbrg.png)

## 2、TCP回射服务器程序：main函数


    //tcpcliserv/tcpserv01.c
    #include "unp.h"

    int main()
    {
        int listenfd, connfd;
        pid_t childpid;
        socklen_t clilen;
        struct sockaddr_in cliaddr, serveraddr;
        listenfd = Socket(AF_INET, SOCK_STREAM, 0);

        bzero(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_addr.s_addr = htonl(INADDR_ANY);       //在头文件中填入通配地址
        servaddr.sin_port = htons(SERV_PORT);               //在头文件中填入众所周知的端口号

        Bind(listenfd, (SA *)&servaddr, sizeof(servaddr));
        Listen(listenfd, LISTENNQ);

        for(;;)
        {
            clien = sizeof(cliaddr);
            connfd = Accept(listenfd, (SA *)&cliaddr, &clilen);     //服务器阻塞于accept调用，等待客户连接的完成
            if((childpid = Fork()) == 0)            //为每个客户派生一个处理它们的子进程
            {
                Close(listenfd);            //关闭监听套接字
                str_echo(connfd);           //处理客户
                exit(0);
            }
            Close(fd);
        }
    }

## 3、TCP回射服务器程序：str_echo函数

    //lib/str_echo.c
    //在套接字上回射程序
    #include "unp.h"

    void str_echo(int sockfd)
    {
        ssize_t n;
        char buf[MAXLINE];

        again:
            while((n = read(sockfd, buf, MAXLINE)) > 0)     //read函数从套接字读入数据
            {
                Writen(sockfd, buf, n);     //把其中内容回射给客户
                if(n<0 && errno == EINTR)
                {
                    goto again;
                }
                else if(n<0)
                {
                    err_sys("str_echo: read error");
                }
            }
    }

如果客户端关闭连接（正常情况），那么接收到客户的`FIN`将导致服务器子进程的`read`函数返回0，导致`str_echo`的返回，中止子进程。

## 4、TCP回射客户程序：main函数

    //tcpcliserv/tcpcli01.c
    #include "unp.h"

    int main(int argc, char **argv)
    {
        int sockfd;
        struct sockaddr_in servaddr;

        if(argc != 2)
        {
            err_quit("usage: tcpcli <IPaddress>");
        }

        sockfd = Socket(AF_INET, SOCK_STREAM, 0);

        bzero(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;          
        servaddr.sin_port = htons(SERV_PORT);   //从头文件中获取众所周知的端口号
        Inet_pton(AF_INET, argv[1], servaddr.sin_addr);     //从命令行参数获得服务器IP地址
        Connect(sockfd, (SA *)&servaddr, sizeof(servaddr));     //建立与服务器的连接

        str_cli(stdin, sockfd);         //客户处理工作

        exit(0);

    }

## 5、TCP回射客户程序：str_cli函数

`str_cli`函数完成客户处理循环：从标准输入读入一行文本，写到服务器上，读回服务器对该行的回射，并把回射行写到标准输出上。

    //lib/str_cli.c
    #include "unp.h"

    void str_cli(FILE *fp, int sockfd)
    {
        char sendline[MAXLINE], recvline[MAXLINE];

        while(Fgets(sendline, MAXLINE, fp) != NULL)     //读入一行
        {   
            Writen(sockfd, sendline, strlen(sendline));     //发送给服务器
            if(Readline(sockfd, recvline, MAXLINE) == 0)    //读取服务器回射行
            {
                err_quit("str_cli: server terminated permaturely");
            }
            Fputs(recvline, stdout);    //写到标准输出
        }
    }

当遇到文件结束符或者错误时，`fgets`将返回一个空指针，于是客户处理循环终止。包裹函数检查是否发生错误，若发生错误则中止进程，因此`Fgets`只是在遇到文件结束符时才会返回一个空指针。

## 6、正常启动

在主机Linux上启动服务器

    linux % tcpserv01 &
    [1] 17870

启动客户之前，运行`netstat`程序检查服务器监听套接字的状态。

    linuc % netstat -a
    Active Internet connections(servers and established)
    Proto Recv-Q Send-Q  Local Address       Foreign Address         State
    tcp        0      0        *:9877                   *:*          LISTEN

`netstat -a`查看监听套接字，通配的本地IP地址，本地端口号为9877，`netstat`用`*`表示一个为0的地址（`INADDR_ANY`，通配地址）或为0的端口号。

在同一个主机上启动客户，并指定服务器主机的地址为127.0.0.1（环回地址）

    linux % tcpcli01 127.0.0.1

客户调用`socket`和`connect`，后者引起TCP的3路握手过程，当3路握手完成后，客户中的`connect`和服务器的`accept`均返回，连接于是建立。
- 客户调用`str_cli`函数，该函数将阻塞于`fgets`调用，因为还未输入一行文本
- 服务器的`accept`返回时，服务器调用`fork`，再由子进程调用`str_echo`。该函数用`readline`，`readline`调用`read`，`read`在等待客户送入一行文本期间阻塞。
- 服务器父进程再次调用`accept`并阻塞，等待下一个客户连接。

至此，有三个在休眠（已阻塞）的进程：客户进程，服务器父进程，服务器子进程。

    linux % netstat -a
    Active Internet connections(servers and established)
    Proto Recv-Q Send-Q  Local Address       Foreign Address         State
    tcp      0      0 localhost:9877       localhost:42578     ESTABLISHED
    tcp      0      0 localhost:42578       localhost:9877     ESTABLISHED 
    tcp      0      0 *:9877                          *:*      LISTEN

第一个`ESTABLISHED`对应于服务器子进程的套接字，因为它本地的端口是9877；第2个`ESTABLISHED`对应于客户进程的套接字，因为它的本地端口是42578。如果在不同的主机上面运行客户和服务器，那么客户主机就会只显示客户进程的套接字，服务器主机也就只显示两个服务器进程的套接字。

使用`ps`命令查看进程的状态和关系

    linux % ps -t pts/6 -o pid,ppid,tty,stat,args,wchan
    PID     PPID    TT      STAT    COMMAND             WCHAN
    22038   22036   pts/6   S       -bash               wait4
    17870   22038   pts/6   S       ./tcpserv01         wait_for_connect
    19315   17870   pts/6   S       ./tcpserv01         tcp_data_wait
    19314   22038   pts/6   S       ./tcpcli01 127.0.0.1 read_chan

`ps`命令加了特定的命令行参数，只显示客户和服务器相关的信息。
- 客户和服务器运行在同一个窗口`pts/6`（伪终端号为6）
- `PID`和`PPID`列给出进程之间的父子关系，子进程的`PPID`是父进程的`PID`，第一个`tcpserv01`是父进程，第二个`tcpserv01`是子进程，父进程的`PPID`是`shell(bash)`
- 三个网络进程的`STAT`列都是`S`，表明进程在为等待某些资源而休眠
- 进程处于休眠状态时，`WCHAN`列指出相应的条件。Linux在进程阻塞与`accept`或`connect`时，输出`wait_for_connect`；在进程阻塞于套接字的输入或者输出时，输出`tcp_data_wait`；在进程阻塞于终端`I/O`时，输出`read_chan`。


## 7、正常终止

    linux % tcpcli01 127.0.0.1
    hello world 
    hello world 
    good bye
    good bye
    ^D                      //终端的EOF字符

输入两行都得到回射，输入`EOF`字符终止客户，此时再执行`netstat`命令

    linux % netstat -a | grep 9877
    tcp     0       0   *:9877              *:*                 LISTEN
    tcp     0       0   localhost:42578     localhost:9877      TIME_WAIT

当前连接的客户端进入了`TIME_WAIT`状态，而监听服务器在等待另一个客户连接。

正常终止客户和服务器的步骤
1. 键入`EOF`字符，`fgets`返回一个空指针，于是`str_cli`函数返回
2. 当`str_cli`返回到客户的`main`函数时，`main`函数通过调用`exit`终止
3. 进程终止处理的部分工作是关闭所有打开的描述符，因此客户打开的套接字由内核关闭。导致客户TCP发送一个`FIN`给服务器，服务器TCP则以`ACK`响应，这就是TCP连接终止序列的前半部分。至此，服务器套接字处于`CLOSE_WAIT`状态，客户套接字则处于`FIN_WAIT_2`状态
4. 服务器TCP接收`FIN`时，服务器子进程阻塞于`readline`调用，于是`readline`返回0。导致`str_echo`函数返回服务器子进程的`main`函数
5. 服务器子进程通过调用`exit`来终止
6. 服务器子进程中打开的所有描述符随之关闭。由子进程来关闭已连接套接字会引发TCP连接终止序列的最后两个分节：一个从服务器到客户的`FIN`和一个从客户到服务器的`ACK`。至此，连接完全终止，客户套接字进入`TIME_WAIT`状态。
7. 进程终止处理的另一部分内容是：在服务器子进程终止时，给父进程发送一个`SIGNCHLD`信号。本例中父进程未加处理这个信号，子进程进入僵死状态`Z`。

使用`ps`命令验证这一点

    linux % ps -t pts/6 -o pid,ppid,tty,stat,args,wchan
    PID     PPID    TT      STAT    COMMAND             WCHAN
    22038   22036   pts/6   S       -bash               read_chan
    17870   22038   pts/6   S       ./tcpserv01         wait_for_connect
    19315   17870   pts/6   Z       [tcpserv01 <defu    do_exit

子进程的状态现在是`Z`，必须要处理掉僵死进程，涉及到Unix信号的处理。

## 8、POSIX信号处理

信号就是告诉某个进程发生了某个事件的通知，有时候也称为软件中断。信号通常是异步发生的，也就是通常不知道信号的准确发生时刻。

信号可以
1. 由一个进程发给另一个进程（或自身）
2. 由内核发给某个进程

上节中提到的`SIGCHLD`信号就是由内核在任何一个进程终止时发给它的父进程的一个信号。每个信号都有一个与之关联的处置。通过调用`sigaction`函数来设定一个信号的处置，有3种选择：
1. 提供一个函数，只要有特定的信号发生就被调用。这样的函数称为信号处理函数，这种行为称为捕获信号。有两个信号不能被捕获，一个是`SIGKILL`一个是`SIGSTOP`。信号处理函数由信号值单一的整数参数来调用，并且没有返回值，函数原型如下：
   
        void handler(int signo);
2. 可以将某个信号的处置设定为`SIG_IGN`来忽略它。`SIGKILL`和`SIGSTOP`不可以被忽略
3. 可以将某个信号的处置设定为`SIG_DFL`来启用它的默认处置。默认处置通常是在收到信号之后终止进程，其中某些信号还在当前工作目录产生一个进程的核心映像(`core image`)。另有个别信号的处置是忽略，`SIGCHLD`和`SIGURG`就是默认处置为忽略的两个信号。

建立信号处置的`POSIX`方法就是调用`sigaction`函数

    //调用POSIXsigaction函数的signal函数
    #include "unp.h"

    Sigfunc *signal(int signo, Sigfunc *func)
    {
        struct sigaction act, oact;

        act.sa_handler = func;
        sigemptyset(&act.sa_mask);
        act.sa_flags = 0;
        if(signo == SIGALRM)
        {
            #ifdef SA_INTERRUPT
            act.sa_flags |= SA_INTERRUPT;
            #endif 
        }
        else
        {
            #ifdef SA_RESTART
            act.sa_falgs |= SA_RESTART;
            #endif 
        }
        if(sigaction(signo, &act, &oact) < 0)
        {
            return SIG_ERR;
        }
        return (oact.sa_handler);
    }


1. 函数`signal`的正常函数原型因为层次太多而变的很复杂：

    void (*signal(int signo, void (*func)(int)))(int);

在头文件`unp.h`中定义了如下的`Sigfunc`类型：

    typedef void Sigfunc(int);

它说明信号处理函数是仅有一个整数且不返回值的函数，`signal`函数的原型于是变为：

    Sigfunc *signal(int signo, Sigfunc *func);

该函数的第2个参数和返回值都是指向信号处理函数的指针。

2. `sigaction`结构的`sa_handler`成员被置为`func`参数。
3. 设置处理函数的信号掩码：`POSIX`允许在信号处理函数被调用时阻塞。任何阻塞的信号都不能被递交给进程。把`sa_mask`成员设置为空集，意味着在该处信号处理函数运行期间，不阻塞额外的信号。`POSIX`保证被捕获的信号在其信号处理函数运行期间总是阻塞的。
4. 设置`SA_RESTART`标志。`SA_RESTART`标志是可选的。如果设置，由相应信号中断的系统调用将由内核自动重启。如果被捕获的信号不是`SAGALRM`且`SA_RESTART`有定义，就设置该标志
5. 调用`sigaction`函数，并将相应信号的旧行为作为`signal`函数的返回值。

`POSIX`的系统上的信号处理总结为以下几点：
1. 一旦安装了信号处理函数，它便一直安装着；
2. 在一个信号处理函数运行期间，正被递交的信号是阻塞的。而且，安装处理函数时在传递给`sigaction`函数的`sa_mask`信号集中指定的任何额外信号也被阻塞。将`sa_mask`置空，意味着除了被捕获的信号外，没有额外信号被阻塞；
3. 如果一个信号在被阻塞期间产生了一次或者多次，那么该信号在被解阻塞之后通常只递交一次，也就是说Unix信号默认是不排队的。
4. 利用`sigprocmask`函数选择性地阻塞或者解阻塞一组信号是可能的。

## 9、处理SIGCHLD信号

设置僵死进程的目的是维护子进程的信息，以便父进程在以后的某个时间获取，这些信息包括子进程的进程ID、终止状态以及资源利用信息。

### 处理僵死进程

僵死进程会占用内核中的空间，最终可能会导致耗尽进程资源。无论何时`fork`子进程时，都需要`wait`它们，防止变成僵死进程。建立一个`SIGCHLD`信号的信号处理函数，在函数体中调用`wait`函数。在TCP回射服务器程序的`listen`调用之后增加如下函数调用：

    Signal(SIGCHLD, sig_chld);

其中`sig_chld`信号处理函数的定义如下：

    #include "unp.h"

    void sig_chld(int signo)
    {
        pid_t pid;
        int stat;

        pid = wait(&stat);
        printf("child %d terminated\n", pid);
        return;
    }

当键入`EOF`字符来终止客户。客户TCP发送一个`FIN`给服务器，服务器响应一个`ACK`；收到客户的`FIN`导致服务器TCP递送一个`EOF`给子进程阻塞中的`readline`，从而子进程终止；当`SIGCHLD`信号递交时，父进程阻塞于`accept`调用。`sig_chld`函数执行，其`wait`调用取到子进程的PID和终止状态，随后是`printf`调用，最后返回。

既然该信号是在父进程阻塞于慢系统调用`accept`时由父进程获取的，内核就会使`accept`返回一个`EINTR`错误（被中断的系统调用）。而父进程不处理该错误，于是中止。

编写捕获信号的网络程序时，必须认清被中断的系统调用并处理它们。有些操作系统可能会自动重启被中断的系统调用，有些不会，编写自己的`signal`函数，应对不同的操作系统。

### 处理被中断的系统调用

