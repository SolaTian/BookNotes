# 模块的编译和链接

自动化编译工具`make`编译项目，`make`自动编译工具依赖项目的`Makefile`文件。
`Makefile`文件主要用来描述各个模块文件的依赖关系，要生成的可执行文件，需要编译哪些源文件，如何编译，先编译哪个，后编译哪个。

`make`在编译项目的时候，会首先解析`Makefile`，分析需要编译哪些源文件，构建一个完整的依赖关系树，然后调用具体的命令一步步去生成各个目标文件和最终的可执行文件。

Windows环境下的`nmake`就相当于`make`，`xx.mak`脚本相当于`Makefile`。

# 系统模块划分

## 规划合理的目录结构

建议项目的文件组织关系和实际源文件的存储目录关系要一致。

# 模块的封装

C语言的一个模块一般对应一个C文件和一个头文件，模块的实现在C源文件中，头文件主要用来存放函数声明，留出模块的`API`供其他模块调用。如MP3的`LCD`显示模块在源文件中实现`lcd_init()`，在头文件中声明，在主程序中想要调用，直接`#include`模块的头文件即可直接使用。

# 头文件深度剖析

## 基本概念

在一个C语言项目中，除了`main`和跳转不需要声明，任何标识符在使用之前都需要进行声明，可以在函数内声明，函数外声明，也可以在头文件中声明。一般为了方便，都是将函数的声明直接放到头文件中，作为本模块封装的`API`，供其他模块使用。如果想要引用这些`API`函数，直接`#include`这个头文件，就可以直接调用。

> 是否分配内存是区分定义和声明的唯一标准。

|变量的声明|变量的定义|
|-|-|
|<li>告诉编译器，该变量可能会在其他文件中定义，编译时先不要报错，等链接的时候可以到指定的文件里看看有没有。如果有直接链接，如果没有再报错<li>变量或函数可以声明多次|变量的定义最终会生成与具体平台相关的内存分配编译指令<li>变量只能定义一次|

一般来说，变量的定义要放到C文件中，不要放到头文件中，因为这个头文件可能会被多人使用，被多个文件包含，头文件经过预处理器多次展开之后就变成了多次定义。


变量和函数可以有多次声明，这个是编译器允许的。如果在头文件中定义了宏或一种新的数据类型，头文件再多次包含展开，可能会报重定义错误。为了防止这种错误产生，可以在头文件中使用条件编译来预防头文件的多次包含。

    //lcd.h
    #ifndef _LCD_H_
    #define _LCD_H_
    ...
    #endif 

通过这个预处理命令，无论包含几次，预处理过程就展开一次。

头文件的多次包含并不会增加可执行文件的体积，因为头文件中只有各种函数的声明，结构体定义，宏定义，只是为了编译检查，不会编译进可执行文件。

## 隐式声明

在C语言中，如果在程序中调用了其他文件中定义的函数，但是没有在本文件中声明，编译器在编译时并不会报错，而是会给一个警告并自动添加一个默认的函数声明。这个函数声明称为隐式声明。

    int f();

编译器隐式声明的函数类型和想要使用的函数类型不一致就会发生错误。

在`ANSI C`中，只是警告，在`C11`中直接报错。

## 变量的声明和定义 

使用`extern`关键字可以将其他文件中的变量或者函数在本文件中使用

    //i.c
    int i = 10;
    int a[10] = {1,2,3,4,5,6,7,8,9,0};
    struct student
    {
        int age;
        int num;
    };
    struct student stu = {20, 100};
    int k;

    //main.c
    extern int i;
    extern int a[10];
    struct student 
    {
        int age;
        int num;
    };                              //要用到student这个类型，需要先定义一遍
    extern struct student stu;
    extern int k;

## 区分定义和声明

上面例子中的`int k`是定义还是声明，使用规则来进行判断
- 省略了`extern`且有初始化语句，则为定义语句。如`int i = 10`
- 使用了`extern`无初始化语句，则为声明。如`extern int i`
- 省略了`extern`且无初始化语句，则为试探性定义，如`int i`

试探性定义，该变量可能在别的文件中定义，先暂时定性为声明，若在别的文件中没有定义，则按照语法规则初始化该变量，并定性为定义。

## 前向引用和前向声明

声明不是给人看的，是给编译器看的，是为了应付编译器语法检查的。

> 前向引用：一个标识符在未声明完成之前，就对其进行引用，一般称为前向引用。

前向引用的特例：
- 隐式声明
- 语句标号：跳转向后的标号时，不需要声明，可以直接使用，即`goto`
- 不完全类型：在被定义完整之前用于某些特定用途。

常见的不完全类型：
- `void`
- 未指定长度的数组：`int a[]`
- 未知内容的结构体或者联合体

    int array(int a[], int len);

    struct LIST_NODE
    {
        struct LIST_NODE *next;
        int data;
    };

对于一个未指定长度的数组，不需要声明就可以直接使用。在链表节点`LIST_NODE`的结构类型中，在`LIST_NODE`声明完成之前，就在结构体内使用其类型定义了一个指针成员`next`，这也算是前向引用，属于C语言允许前向引用的3个特例。

当对一个标识符前向引用时，一般只关注标识符类型，而不关注该标识符的大小，值或者具体实现。也就是说，当对一个不完全类型进行前向引用时，只能使用该标识符的部分属性：类型。其他的一些属性，如变量值、结构成员、大小等，是不可以使用的，否则就会编译报错。

    strcut LIST_NODE
    {
        struct LIST_NODE *next;
        struct LIST_NODE node;          //编译报错
        int data;
    }

编译报错的原因在于，当编译器遇到`struct LIST_NODE node;`这条语句时，需要考虑`node`的大小，但是结构体类型`LIST_NODE`此时还没有完成定义，属于不完全类型，编译器无法知晓其大小，所以会报错。对于结构体指针`struct LIST_NODE *next;`定义的成员是一个指针，使用的只是不完全类型`LIST_NODE`的一个属性：类型，来指定指针的类型。无论什么指针，其大小是不变的。所以可以在结构体内定义任意类型的指针，都不需要实现声明，而且编译不会报错

    strcut LIST_NODE
    {
        struct LIST_NODE *next;
        struct queue *q;
        struct hello *p;
        struct world *r;
        int data;
    }

> 前向声明：如下面的`struct person;`语句

    struct person;                  //对结构类型person进行声明

    struct student
    {
        struct person *p;
        int score;
        int no;
    }

使用前向声明的好处是，当这个声明被多个文件包含时，不会报数据类型的重定义错误。前向声明在Linux内核中被大量使用，尤其是在头文件中，到处可见。从声明这个类型之后到定义这个类型之前的这段区间，这个结构类型就是一个不完全类型。

    struct usb_device;
    struct usb_driver;
    struct wusb_dev;
    struct ep_device;
    ...

    struct usb_bus
    {
        struct device *controller;
        ...
        struct usb_device *root_hub;
        struct usb_bus    *hs_companion;
    };


## 定义与声明的一致性

在模块里包含自己的头文件，除了可以使用头文件中定义的宏或者数据类型。还可以让编译器去检查定义和声明的一致性。

## 头文件的路径

标准库的头文件，使用`<>`包含，自定义的头文件，使用`""`包含，头文件路径一般分为绝对路径和相对路径。绝对路径以根目录`/`或者Windows下的每个盘符为路径起点，相对路径则以程序的当前目录为起点。

    #include "/home/wit/code/xx.h"
    #include "F:/litao/code/xxx.h"
    #include "../lcd/lcd.h"
    #include "./lcd.h"

使用`<>`包含的头文件，头文件的搜索顺序为：
1. `GCC`参数`gcc -I`指定的目录
2. 通过环境变量`CINCLUDEPATH`指定的目录
3. `GCC`的内定目录
4. 搜索规则：当不同目录下存在相同的头文件时，先搜到哪个就使用哪个，搜索到头文件后不再往下搜索。

使用`""`包含的头文件，头文件的搜索顺序为：
1. 项目当前目录
2. 通过`GCC`参数`gcc -I`指定的目录
3. 通过环境变量`CINCLUDEPATH`指定的目录
4. `GCC`的内定目录
5. 搜索规则：当不同目录下存在相同的头文件时，先搜到哪个就使用哪个

Linux下经常使用的环境变量：
- `PATH`：可执行文件的搜索路径
- `C_INCLUDE_PATH`：C语言头文件搜索路径
- `CPULS_INCLUDE_PATH`：C++头文件搜索路径
- `LIBRARY_PATH`：库搜索路径

## Linux内核中的头文件

    #include <linuc/xx.h>
    #include <asm/xx.h>
    #include <mach/xx.h>
    #include <plat/xx.h>

这些使用`<>`包含的头文件使用的是相对路径：
- 与CPU架构有关：`arch/$(ARCH)/include`
- 与板级平台有关：`arch/$(ARCH)/mach-xx(plat-xx)/include`
- 主目录：`include`
- 内核头文件专用目录：`include/linux`

内核源码中使用的头文件路径一般都是相对路径，在内核编译的过程中通过`gcc -I`参数来指定头文件的起始目录，打开Linux内核源码顶层的`Makefile`，有一个`LINUXINCLUDE`变量，用来指定内核编译时的头文件路径。

## 头文件中的内联函数

内联函数一般定义在C文件中，但是在Linux内核源码的头文件中，经常会看到一些内联函数。内联函数可以在头文件中定义的原因：当多个模块引用该文件时，内联函数在编译时已经在多个调用处展开，不复存在了，因此不存在重定义问题。即使编译器没有对内联函数展开，也可以在内联函数前通过添加一个`static`关键字将该函数的作用域限制在本文件中，避免出现重定义错误。

    static inline void func(int a, int b);


# 模块设计原则

高内聚低耦合是模块设计的基本原则。

> 模块的内聚度：指模块内各个元素的关联、交互程度。从功能角度看，就是各个模块在实现各自功能的时候，自己的事情自己做，尽量不麻烦其他模块。

> 模块的耦合：指模块之间的关联和依赖，包括调用关系、控制关系、数据传递等。模块之间的关联越强，耦合度越高，模块的独立性越差，内聚度越低。

模块之间耦合方式：
- 非直接耦合：两个模块之间没有直接联系
- 数据耦合：通过参数来交换数据
- 标记耦合：通过参数传递记录信息
- 控制耦合：通过标志、开关、名字等，控制另一个模块
- 外部耦合：所有模块访问同一个全局变量


低耦合可以让系统的层次更加清晰，升级维护更加方便。可以通过以下方法降低模块的耦合度：
- 接口设计：隐藏不必要的接口和内部数据类型，模块引出的`API`封装在头文件中，其余函数使用`static`修饰
- 全局变量：尽量少使用，可以改为通过`API`访问以减少外部耦合
- 模块设计：尽可能独立存在，功能单一明确，接口少而简单
- 模块依赖：模块之间最好是单向调用，上下依赖，禁止相互调用。

# goto 

`goto`的无条件跳转的特性会大大简化程序的设计，如有多个出错出口的函数们可以使用`goto`将函数内的出错指定一个同一的出口，统一处理，反而会使函数的结构更加清晰。

通过模块化设计，将函数侏罗纪代码和出错处理部分隔离，使函数的内部结构更加清晰。通过代码复用，将一个函数多个出口归并为一个总出口，在总出口处对出错统一处理，释放`malloc()`申请的动态内存，释放锁，文件句柄等资源。

在一个多重循环的程序中，想从最内层循环直接跳出，可以直接使用`goto`，避免使用多个`break`。在Linux内核中，`goto`也被广泛使用。

使用`goto`的注意点：
- 只能在同一个函数内跳转。

# 模块间通信

## 全局变量

各个模块共享全局变量是各个模块之间进行数据通信最简单直接的方式。可以使用`extern`关键字将全局变量的作用域扩展到不同的文件中，然后各个模块之间就可以通过全局变量进行通信。

把对于全局变量的直接访问修改为通过函数接口间接访问。就像类的私有成员变量。该全局变量只能在一个模块中创建或直接修改，如果其他模块想要访问这个全局变量，则只能通过引出的函数读写接口进行访问。


    //module.h
    void val_set(int value);
    int val_get(void);

    //module.c
    int global_val = 10;
    void val_set(int value)
    {
        global_val = value;
    }

    int val_get(void)
    {
        return global_val;
    }

其他模块想要对全局变量`global_val`访问，不再通过变量名直接访问，而是通过`module`封装的`val_set()`和`val_get()`函数进行访问。在多任务的环境下，有时候还需要注意全局变量的互斥访问。通过函数接口访问共享的全局变量在一定的程度上减少了模块之间的外部耦合。

Linux内核中的全局变量在定义的时候要先通过`EXPORT_SYMBOL`导出，然后其他的模块才可以引用。这样做的原因：
1. Linux内核代码量巨大，全局变量数巨大，如果所有的全局变量都可以访问，都导出到符号表中，生成的可执行文件很大；
2. 如果有人在自己的源文件中定义了同名的全局变量，多个文件在链接时还会发生符号冲突，产生重定义错误；
3. 有些全局变量可能只是一个内核模块的几个文件之间共享的”区域性全局变量“，使用`EXPORT_SYMBOL`可以区分出哪些全局变量是真正的全局变量，是所有内核模块都可以访问的。如果定义了一个全局变量，而且只是在自己的模块中使用，不想被其他人使用，为了避免重定义错误，建议使用`static`关键字来修饰这一个全局变量，将它的作用域限定在本文件中，可以有效地避免名字冲突。


## 回调函数

一个系统的不同模块之间还可以通过数据耦合、标记耦合的方式进行通信，即通过函数调用过程中的参数传递，返回值来实现模块间通信。

最普遍的函数调用就可以使用模块间通信：

    //module.c
    int send_data(char *buf, int len)
    {
        char data[10];
        int i;
        for(i = 0; i<len; i++)
            data[i] = buf[i];
        for(i = 0; i<len; i++)
            printf("received data[%d] = %d\n", i, data[i]);
        return len;
    }

    //main.c
    #include <stdio.h>

    int send_data(char *buf, int len);

    int mian(void)
    {
        char buffer[10] = {1,2,3,4,5,6,7,8,9,0};
        int return_data;
        return_data = send_data(buffer, 10);
        printf("send data len: %d\n", return_data);
        return 0;
    }

这种通信方式的缺点就是单向调用，无法实现双向通信。当底层模块想要主动与上一层的模块进行通信时，就需要回调函数来实现。

> 回调函数：在编写程序实现一个函数时，通常会直接调用底层模块的`API`函数或者库函数，如果反过来，写一个函数，让系统直接调用该函数，那么这个过程被称为回调。这个函数也被称为回调函数。

回调函数最显著的特点就是：`Do not call me,I will call you`。底层模块也可以调用上层模块的函数，实现双向通信。

    //module.h
    #ifndef _RUNCALLBACK_H
    #define _RUNCALLBACK_H
    void runcallback(void (*fp)(void));
    #endif

    //module.c

    void runcallback(void (*fp)(void))
    {
        fp();
    }

    //app.c
    #include <stdio.h>
    #include "module.h"

    void func1(void)
    {
        printf("func1...\n");
    }

    void func2(void)
    {
        printf("func2...\n");
    }

    int main(void)
    {
        runcallback(func1);
        runcallback(func2);
        return 0;
    }

程序运行结果

    func1...
    func2...

通过回调函数的设计，两个模块之间实现了双向通信，模块之间通过函数调用或者变量引用产生了耦合，也有了依赖关系。

上层模块不应该依赖底层模块，它们共同依赖某一个抽象。抽象不能依赖具象，具象依赖抽象。为了减少模块的耦合性，在两个模块之间定义一个抽象接口。

    //device_manager.h
    #ifndef _STORAGE_DEVICE_H
    #define _STORAGE_DEVICE_H
    typedef int (*read_fp)(void);

    struct storage_device
    {
        char name[20];
        read_fp read;
    };

    extern int register_device(struct storage_device dev);
    extern int read_device(char *device_name);
    #endif

    //device_manager.c
    #include <stdio.h>
    #include <string.h>
    #include "device_manager.h"

    struct storage_device device_list[100] = {0};
    unsigned char num;

    int register_device(struct storage_device dev)
    {
        device_list[num++] = dev;
        return 0;
    }

    int read_device(char *device_name)
    {
        int i;
        for(i=0; i<100; i++)
        {
            if(!strcmp(device_name, device_list[i].name))
                break;
        }
        if(i == 100)
        {
            printf("Error! can't find device:%s\n", device_name);
            return -1;
        }
        return device_list[i].read();
    }

    //app.c
    #include <stdio.h>
    #include "device_manager.h"

    int sd_read(void)
    {
        printf("sd read data...\n");
        return 10;
    }

    int udisk_read(void)
    {
        printf("udisk read data...\n");
        return 20;
    }

    struct storage_device sd = {"sdcard", sd_read};
    struct storage_device udisk = {"udisk", udisk_read};

    int main(void)
    {
        register_device(sd);        //高层模块函数注册，使用回调
        register_device(udisk);     

        read_device("udisk");       //实现回调，控制反转
        read_device("udisk");
        read_device("uk");
        read_device("sdcard");
        read_device("sdcard");

        return 0;
    }

程序运行的结果

    udisk read data...
    udisk read data...
    Error! can't find device: uk
    sd read data...
    sd read data...


让系统回调我们注册到设备管理模块中的自定义函数，高层模块和底层模块通过`device_manager`模块实现的抽象接口，解除了模块之间的耦合关系

## 异步通信

模块间通信无论是通过模块接口，还是通过回调函数，其实都是属于阻塞式同步调用，会占用CPU资源。

常用的异步通信：
- 消息机制：具体实现和平台有关
- 事件驱动机制：状态机、`GUI`、前端编程等
- 中断
- 异步回调

Linux内核模块之间可以使用`notify`进行通信；内核和用户之间可以通过`AIO`，`netlink`进行通信；用户模块之间通信包括操作系统支持的管道、信号、信号量、消息队列，还可以使用`socket`等方式进行异步通信。

# 模块设计进阶

## 跨平台设计

为了使编写的程序能够在不同的环境下运行，此时应该考虑尽量使用C标准库函数，而不是使用操作系统的系统调用接口。

还需要注意大端模式和小端模式、内存对齐、不同数据类型的字长等。

- 将与操作系统有关的系统调用封装成统一的接口，隐藏不同的操作系统之间接口的差异；
- 头文件路径分隔符使用通用的`/`
- 禁止使用编译器的扩展语法或特性，使用C语言的标准语法
- 尽量不要使用内嵌汇编
- 打开所有的警告，重视`warning`
- 使用条件编译，使代码兼容适配每个平台

## 框架

框架就是一个可扩展的应用程序骨架。

将一个行业领域内众多应用软件的相同功能进行分离和抽象，将应用中的一些通用的功能模块化，把通用的模块下沉，沉淀为底层，将专用的模块上浮，提供可配置和扩展的接口，经过优化和完善，迭代成一个软件框架。

# AloT时代的模块化编程

`AloT`指的是`AI`+`loT`人工智能加物联网，嵌入式设备开始具备联网功能，接入云端，将感知的数据传入云服务器，越来越多的协议栈，组件，服务开始集成到嵌入式系统。