# 基本TCP套接字编程


![](https://img-blog.csdnimg.cn/20190822215013320.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGFvMjE=,size_16,color_FFFFFF,t_70)


## 1、socket函数

为了执行网络I/O，一个进程必须做的第一件事就是调用`socket()`，指定期望的协议类型

    #include <sys/socket.h>
    int socket(int family, int type, int protocol)      //成功则返回非负描述符，出错返回-1。

`family`指明协议族，`type`指明套接字类型，`protocol`指定协议类型，或者设置为0，根据`family`和`type`组合的系统默认值。
|family|说明|
|-|-|
|AF_INET|IPv4协议|
|AF_INET6|IPv6协议|
|AF_LOCAL|UNIX域协议|
|AF_ROUTE|路由套接字|
|AF_KEY|密钥套接字|

|type|说明|
|-|-|
|SOCK_STREAM|字节流套接字|
|SOCK_DGRAM|数据报套接字|
|SOCK_SEQPACKET|有序分组套接字|
|SOCK_RAW|原始套接字|

|protocol|说明|
|-|-|
|IPPROTO_TCP|TCP传输协议|
|IPPROTO_UDP|UDP传输协议|
|IPPROTO_STCP|STCP传输协议|

![基本TCP客户/服务器程序的套接字函数](https://img-blog.csdnimg.cn/20190822215013320.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGFvMjE=,size_16,color_FFFFFF,t_70)

`socket()`函数在成功时返回一个小的非负整数值，与文件描述符类似，称为套接字描述符`sockfd`


## 2、connect函数

TCP客户端用connect函数来建立与TCP服务器的连接

    #include <sys/socket.h>
    int connect(int sockfd, const struct *servaddr, socklen_t addrlen); 

    //成功则返回0，出错返回-1

`sockfd`为由`socket函数`返回的套接字描述符，第2、3个参数分别为一个指向套接字地址结构的指针和该结构的大小。套接字结构中必须有服务器的IP地址和端口号。

客户端在调用`connect`前不必非得调用`bind函数`，因为如果需要的话，内核会确定源IP地址，并选择一个临时的端口作为源端口。

如果是TCP套接字，调用connect函数将会激发TCP的三次握手过程。

若`connect`失败则该套接字不可再用，必须关闭，不能对这样的套接字再次调用`connect函数`。当循环调用`connect`为给定的主机尝试各个IP地址直到有一个成功时，每次`connect`失败后，都必须`close`当前的套接字描述符，并重新调用socket。

## 3、bind函数

`bind函数`把一个本地协议地址赋予一个套接字。协议地址是32位的IPv4地址或者128位的IPv6地址与16位TCP或者UDP的端口号的组合。

    #include <sys/socket.h>
    int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);
    //若成功则返回0，出错返回-1

第二个参数是指向特定协议的地址结构的指针，第三个参数是该地址结构的长度。对于TCP，`bind函数`可以指定一个端口号，或者IP地址，也可以两者都指定，也可以两者都不指定。

- 如果一个TCP客户端或服务器未曾调用`bind`绑定一个端口，当调用`connect`或者`listen`时内核就要为相应的套接字选择一个临时端口。内核选择对于TCP客户端来说是正常的，但是对于TCP服务器是罕见的，因为TCP服务器的端口是众所周知的。
- TCP客户通常不把IP地址绑定到它的套接字上，当连接套接字时，内核将根据所用外出网络接口（网卡）来选择源IP地址。如果TCP服务器没有把IP地址捆绑到它的套接字上，内核就会把客户发送的`SYN`的目的地址作为服务器的源IP地址。


|IP地址|端口|结果|
|-|-|-|
|通配地址|0|内核选择IP地址和端口|
|通配地址|非0|内核选择IP地址，进程指定端口|
|本地IP地址|0|进程指定IP地址，内核选择端口|
|本地IP地址|非0|进程指定IP地址和端口|

对于IPv4来说，通配地址由常值`INADDR_ANY`来指定，其值一般为0。它告知内核去选择IP地址。

    struct sockadddr_in servaddr;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

对于IPv6，系统预先分配`in6addr_any`变量并将其初始化为常值`IN6ADDR_ANY_INIT`。头文件`<netinet/in.h>`中含有`in6addr_any`的`extern`声明。

    struct sockaddr_in6 serv;
    serv.sin6_addr = in6addr_any;

如果指定端口号为0，那么内核在`bind`被调用的时候选择一个临时端口。然而如果指定IP地址为通配地址，那么内核将会等到套接字已连接（TCP）或已在套接字上发出数据报（UDP）时才会选择一个本地IP地址。

如果内核为套接字选择一个临时端口号，`bind`并不返回所选择的值。`bind`的第二个参数有`const`限定词，无法返回所选值。为了得到内核选择的临时的端口值，必须调用函数`getsockname`
来返回协议地址。

从`bind`函数返回的一个常见错误是`EADDRINUSE`，地址已经被使用。


## 4、listen函数

`listen函数`仅供TCP服务器调用。完成以下两件事：
1. 当`socket函数`创建一个套接字时，它被假设为一个主动套接字，也就是它是一个将会调用`connect`发起连接的客户套接字，`listen`函数把一个未连接的套接字转换成一个被动套接字，指示内核应接受指向该套接字的连接请求。
2. `listen`的第2个参数规定了内核应该为相应套接字排队的最大连接个数。

        #include <sys/socket.h>
        int listen(int sockfd, int backlog);
        //若成功返回0，出错返回-1

本函数通常在调用`socket`和`bind`两个函数之后，并在调用`accept`函数之前调用。

内核会为任何一个给定的监听套接字维护两个队列：
1. 未完成连接队列：已经由某个客户端发出并到达服务器，而服务器正在等待完成相应的TCP三次握手过程，这些套接字处于`SYN_RCVD`状态。
2. 已完成连接队列：每个已经完成TCP3次握手过程的客户对应其中一项，这些套接字处于`ESTABLISHED`状态。

两个队列之和不能超过`listen`函数的第2个参数`backlog`。

每当在未完成连接队列中创建一项时，来自监听套接字的参数就复制到即将建立的连接中。连接的创建机制是完全自动的，无需服务器进程插手。

![TCP3路握手和监听套接字的两个队列](https://img2018.cnblogs.com/blog/1169746/201810/1169746-20181001235626296-833035849.png)

当来自客户的`SYN`到达时，TCP在未完成连接队列中创建一个新项，然后响应以三路握手的第2个分节：服务器的`SYN`响应，其中包括对于客户`SYN`的`ACK`。这一项一直保留在未完成连接队列中。直到三路握手的第3个分节（客户端对服务器`SYN`的`ACK`）到达或者该项超时为止。如果三路握手正常完成，该项就从未完成连接队列移到已完成连接队列的队尾。当进程调用`accept`时，已完成连接队列的队头项将返回给进程，或者如果队列为空，那么进程将被置于休眠状态，直到TCP在该队列中放入一项才唤醒它。

## 5、accept函数

`accept`函数由TCP服务器调用，用于从已完成链接队列队头返回下一个已完成连接。如果已完成连接队列为空，那么进程就会被置于休眠状态。

    #include <sys/socket.h>

    int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
    //成功则返回非负描述符，失败则返回-1

参数`cliaddr`和`addrlen`用来返回对已连接的对端进程（客户）的协议地址。`addrlen`是值-结果参数：调用前，将由`*addrlen`所引用的整数值置为由`cliaddr`所指的套接字地址结构的长度，返回时，该整数值即为由内核存放在该套接字地址结构内的确切字节数。

如果`accept`成功，那么其返回值是由内核自动生成的一个全新描述符，代表与所返回客户的TCP连接。在讨论`accept`函数时，第一个参数为监听套接字描述符，称它的返回值为已连接套接字描述符。一个服务器通常只会创建一个监听套接字，它在该服务器的声明周期之内一直存在。内核为每个由服务器进程接受的客户端连接创建一个已连接套接字（也就是说对于它的TCP三路握手过程已经完成）。当服务器完成对某个给定客户的服务时，相应的已连接套接字就会被关闭。

`accept`最多返回3个值：一个即可能是新套接字描述符，也可能是出错指示的整数、客户进程的协议地址（`cliaddr`指针所指向）以及该地址的大小（由`addrlen`指针所指）。如果对于返回客户协议地址不感兴趣，那么可以把`cliaddr`和`addrlen`置为`NULL`。


    //显示客户IP地址和端口号的时间获取服务器程序
    //daytimetcpsrvl.c
    #include "unp.h"
    #include <time.h>

    int main()
    {
        int listenfd, connfd;
        socklen_t len;                          //len将成为一个值-结果变量
        struct sockaddr_in servaddr, cliaddr;   //cliaddr 存放客户的协议地址
        char buff[MAXLINE];
        time_t ticks;

        listenfd = Socket(AF_INET, SOCK_STREAM, 0);

        bzero(&servaddr, sizeof(servadr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_addr.s_addr = htol(INADDR_ANY);
        servaddr.sin_port = htons(13);

        Bind(listenfd, (SA *)&servaddr, sizeof(servaddr));

        Listen(listenfd, LISTENQ);

        for(;;)
        {
            len = sizeof(cliaddr);                                  //len初始化为套接字地址结构的大小
            connfd = Accept(listenfd, (SA *)&cliaddr, &len);        //将指向cliaddr的指针和len的指针作为第2,3个参数
            printf("connection from %s, port %d\n",
                    inet_ntop(AF_INET, &cliaddr.sin_addr, buff, sizeof(buff)),
                    ntohs(cliaddr.sin_port));                       //将套接字地址地址结构中的32位IP地址转换为一个点分十进制的ASCII
                                                                    //调用ntohs将16位端口号从网络字节序转换为主机字节序
            
            tick = time(NULL);
            snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
            Write(connfd, buff, strlen(buff));

            Close(connfd);
        }
    }

    运行结果
    #solaris # daytimetcpsrvl 
    connection from 127.0.0.1, port 43388
    connection from 192.168.1.30, port 43888

其中，这些套接字函数的首字母大写的函数为包裹函数，包裹函数完成实际的函数调用，检查返回值，并在发生错误时终止进程。

## 6、fork和exec函数

`fork`函数是unix中派生新进程的唯一方法

    #include <unistd.h>

    pid_t fork(void);
    //在子进程中为0，在父进程中为子进程ID，若出错则为-1

`fork`函数调用一次，返回两次，在父进程中返回一次，返回值为派生进程（子进程）的进程ID，在子进程中又返回一次，返回值为0。返回值本身就可以告知当前进程是子进程还是父进程。

`fork`在子进程中返回0而不是父进程的进程ID的原因在于：任何子进程只有一个父进程，子进程可以通过调用`getppid`来获取父进程的进程ID。相反，父进程可以有很多的子进程，无法获得各个子进程的进程ID。如果父进程要想跟踪所有子进程的进程ID，必须记录每次调用`fork`的返回值。

父进程中调用`fork`之前打开的所有描述符在`fork`返回之后由子进程共享。网络服务器利用这个特性：父进程调用`accept`之后调用`fork`。所接受的已连接套接字随后就在父进程和子进程之间共享。通常情况下，子进程接着读写这个已连接套接字，父进程则关闭这个已连接套接字。

`fork`有两个典型用法：
1. 一个进程创建一个自身的副本，每个副本在另一个副本执行其他任务的同时，处理各自的某个操作。这是网络服务器的典型用法。
2. 一个进程想要执行另一个程序。进程首先调用`fork`创建一个自身的副本，然后其中一个副本（通常为子进程）调用`exec`把自身替换成新的程序。这是`shell`之类程序的典型用法。

存放在硬盘上的可执行程序文件能够被unix执行的唯一方法：由一个现有进程调用6个`exec`函数中的某一个。`exec`把当前进程映像替换成新的程序文件，而且该新程序通常从`main`函数开始执行。进程ID并不改变。称调用`exec`的进程称为调用进程，称新执行的程序为新程序。


    #include <unistd.h>
    
    int execl(const char *pathname, const char* arg0, .../*(char *) 0*/);
    
    int execv(const char *pathname, char *const *argv[]);
    
    int execle(const char *pathname, const char *arg0, .../*(char *) 0, char * const envp[]*/);
    
    int execve(const char *pathname, char *const argv[], char *const envp[]);

    int execlp(const char *filename, const char *arg0, ... /*(char *) 0*/);

    int execvp(const char *filename, char *const argv[]);

    //上述函数若成功则不返回（传给新程序的起始点），若出错返回-1

上述6个函数的区别在于：
1. 待执行的程序文件是由文件名（`filename`）还是由路径名(`pathname`)指定
2. 新程序的参数是一一列出还是由一个指针数组来引用
3. 把调用进程的环境传递给新程序还是给新程序指定新的环境。

![](https://pic2.zhimg.com/80/v2-14fd9241c4671aa8c057c7c3e33da765_1440w.webp)

- 上图中上面那行的3个函数把新程序的每个参数字符串指定成`exec`的一个独立参数，并以一个空指针结束可变数量的这些参数。下面那行的3个函数都有一个作为`exec`参数的`argv`的数组。其中含有指向新程序各个参数字符串的所有指针。既然没有指定参数字符串的数目，这个`argv`数组必须含有一个用于指定其末尾的空指针。
- 左列2个函数指定一个`filename`参数。`exec`将使用当前的`PATH`环境变量把该文件名参数转换为一个路径名。然而一旦这2个参数的`filename`参数中含有一个`\`斜杠，就不再使用`PATH`环境变量。右两列4个函数指定一个全限定的`pathname`参数。
- 左两列4个函数不显示指定一个环境指针。相反，它们使用外部环境变量`environ`的当前值来构造一个传递给新程序的环境列表。右边2个函数显示指定一个环境列表，其`envp`指针数组必须以一个空指针结束。

## 7、并发服务器

当服务一个客户请求可能花费较长时间时，并不希望整个服务器被单个客户长期占用，而是希望同时服务多个客户。unix中编写并发服务器程序最简单的办法就是`fork`一个子进程来服务每个客户。下面给出一个典型的并发服务器程序轮廓。

    #include "unp.h"
    #include <time.h>

    int main()
    {
        pid_t pid;
        int listenfd, connfd;

        listenfd = Socket(...);

        /*绑定端口*/
        Bind(listenfd, ...);
        Listen(listenfd, LISTENQ);

        for(;;)
        {
            connfd = Accept(listenfd, ...);        
            if((pid = Fork()) == 0)
            {
                Close(listenfd);            //子进程关闭监听套接字
                doit(connfd);               //处理请求
                Close(connfd);              //处理完成客户端请求
                exit(0);                    //子进程结束
            }

            Close(connfd);                  //父进程关闭已连接套接字
        }
    }

当一个连接建立时，`accept`返回，服务器调用`fork`，然后由子进程服务客户（通过已连接套接字`connfd`），父进程则是等待另一个连接（通过监听套接字`listenfd`）。既然新的客户由子进程提供服务，父进程就关闭已连接套接字。


## 8、close函数

unix的`close`函数可以用来关闭套接字，并终止TCP连接。

    #include <unistd.h>
    
    int close(int socked);
    //成功返回0，若出错返回-1

`close`一个TCP套接字的默认行为是把该套接字标记为已关闭，然后立即返回到调用进程。该套接字描述符不能够被再由调用进程使用，不能作为`read`或者`write`的第一个参数。TCP将尝试发送已排队等待发送到对端的任何数据，发送完毕后发生的是正常的TCP连接终止序列。


## 9、getsockname和getpeername函数

    #include <sys/socket.h>

    int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);

    int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);
    //若成功则为0，若出错则为-1

需要这两个函数的理由如下：
1. 在一个没有调用`bind`的TCP客户上，`connect`成功返回之后，`getsockname`用于返回由内核赋予该连接的本地IP和本地端口号；
2. 在以端口号0调用`bind`后，`getsockname`用于返回由内核赋予的本地端口号
3. `getsockname`可用于获取某个套接字的地址族

        #include "unp.h"
        
        int sockfd_to_family(int sockfd)
        {
            struct sockaddr_storage ss;
            socklen_t len;

            len = sizeof(ss);
            if(getsockname(sockfd,  (SA*)&&ss, &len) < 0)
                return -1;
            return(ss.ss_family);
        }
4. 在一个以通配IP地址调用`bind`的TCP服务器，与某个客户的连接一旦建立（`accept`成功返回），`getsockname`就可以用于返回由内核赋予该连接的本地IP地址。在这样的调用中，套接字描述符参数必须是已连接套接字的描述符，而不是监听套接字的描述符。
5. 当一个服务器是由调用过`accept`的某个进程通过调用`exec`执行程序时，它能够获得客户身份的唯一途径就是调用`getpeername`。

