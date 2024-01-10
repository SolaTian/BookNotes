# C语言标准和C编译器

> C语言的标准是ANSI C标准，也叫C89或C90标准

`ANSI`是美国国家标准协会简称。

## C语言标准

C标准：
- 定义各种关键字、数据类型；
- 定义各种运算规则、运算符优先级和结合性；
- 数据类型转换；
- 变量的作用域；
- 函数原型、函数嵌套层数、函数参数个数限制；
- 标准库函数接口

程序员按照这种标准编写程序，编译器按照这种标准去解析、翻译。


|C标准经历|说明|
|-|-|
|`K&R C`|由C语言作者编写，是`ANSI C`的雏形|
|`ANSI C`|在`K&R C`基础上，统一了各大编译器厂商的不同标准，对C语言的语法和特性进行了扩展，也叫`C89/C90标准`。<li>增加了`signed`、`volatile`、`const`；<li>增加了`void *`；<li>增加了预处理命令；<li>增加了宽字符串；<li>定义了C标准库|
|`C99`|`ANSI`基于`C89/C99`发布的一个新标准，新增了一些关键字<li>布尔型：`_Bool`；<li>复数：`_Complex`；<li>虚数：`_Imaginary`；<li>内联：`inline`；<li>指针修饰符：restrict；<li>支持`long long`、`long double`；<li>支持变长数组；<li>允许对结构体特定成员赋值；<li>变量声明可以放在代码块的任何地方；<li>支持`//`单行注释；<br>等等|
|`C11`|`ANSI`在2011发布的新标准，增加了一些新特性<li>增加`_Noreturn`，声明函数无返回值；<li>增加`_Generic`，支持泛型编程；<li>修改了标准库函数的一些`bug`；<li>新增文件锁功能；<li>支持多线程<br>等等，新发布的C11标准，大多数编译器还不支持|

有些编译器只支持`ANSI C`标准，目前对`C99标准`支持的最好的是`GNU C`编译器，支持`C99`的几乎所有特性。

不同的编译器可以对C语言做不同的标准。

> `GNU`：一个开源平台，里面有大量的开源项目、开源软件的自由操作系统。

> `GCC`：是`GNU`中的编译器项目，包括`g++`、`gcc`、还包括`mingw`调用`GCC`编译代码，`mingw`是Windows下`GNU`工具链，包括`gcc`等。



# 指定初始化

## 数组元素的指定初始化

在`GNU C`中，可以通过如下方式给数组指定初始化

    int a[100] = {[10] = 1, [30] = 2};
    int b[100] = {[10...30] = 1, [50...60] = 2};

也可以使用在`switch-case`语句

    switch(i)
    {
        case 1:
            printf("1\n");
            break;
        case 2...8:
            printf("2\n");
            break;
        default:
            printf("default\n);
            break;
    }

## 结构体成员指定初始化

    struct student{
        char name[20];
        int age;
    }

    int main(void)
    {
        struct student stu1 = {"wit", 20};          //C标准初始化
        struct student stu2 = 
        {
            .name = "qintian",
            .age = 28
        };                                          //GNU C初始化
    }

## Linux内核驱动注册

在驱动程序中，经常使用`file_operations`这个结构体来注册开发的驱动。在Linux内核驱动中，有大量使用指定初始化的方式初始化结构体。不用按照成员的顺序来进行初始化，使得代码易于维护。

# 宏构造：语句表达式

`GNU C`对C语言标准做了扩展，允许在一个表达式里内嵌语句，允许在表达式内部使用局部变量，`for循环`和`goto语句`，这种类型称为语句表达式

    ({表达式1;表达式2;表达式3;})

语句表达式的值为内嵌语句中最后一个表达式的值。

## 在宏定义中使用语句表达式

语句表达式的主要用途在于定义功能复杂的宏

定义一个宏，求两数的最大值

    #define MAX(x,y) x>y?x:y                //漏洞：x或y为一个表达式，展开后表达式运算顺序发生改变。如1!=1>1!=2?1!=1:1!=2
    #define MAX(x,y) (x)>(y)?(x):(y)        //优化上一个，漏洞：同时含有宏定义和其他高优先级运算符。如3+MAX(1,2)
    #define MAX(x,y) ((x)>(y)?(x):(y))      //优化上一个，避免当一个表达式同时含有宏定义和其他高优先级运算符，破坏了整个表达式的运算顺序。
    #define MAX(x,y)({    \
        int _x = x;       \
        int _y = y;       \
        _x > _y ? _x : _y;\
    })                                      //使用语句表达式，对临时变量进行比较，避免了如MAX(i++, j++)出现错误的结果
    #define MAX(type, x, y)({       \
        type _x = x;                \
        type _y = y;                \
        _x > _y ? _x : _y           \
    })                                      //添加参数type，用来指定临时变量_x和_y的类型
    #define MAX(x,y)({              \
        typeof(x) _x = (x);         \
        typeof(y) _y = (y);         \
        (void)(&_x == &_y);         \        
        _x > _y ? _x : _y;          \
    })                                      //使用GNU C新增关键字typeof获取数据类型


在Linux内核中，有大量的宏定义使用了语句表达式。


# typeof和container_of

## typeof()关键字

    sizeof()            //用来获取一个变量或者数据类型在内存中所占的字节数——ANSI C 
    typeof()            //用来获取一个变量或者表达式的类型——GNU C

    int i;
    typeof(i) j = 20;       //相当于int j = 20
    typeof(int *)a;         //相当于int *a
    int f();
    typeof(f()) k;          //f()的返回值为int，相当于int k

    typeof(int *)y;         //把y定义为指向int类型的指针，相当于int *y
    typeof(int) *y;         //定义一个指向int类型的指针变量
    typeof(*x) y;           //定义一个指针x所指向类型的变量y
    typeof(int)y[4];        //相当于int y[4]
    typeof(typeof(char *)[4])y;  //相当于字符数组char *y[4];
    typeof(int x[4])y;      //相当于int y[4]

## Linux内核中的container_of宏

Linux内核第一宏

    #define offsetof(TYPE, MEMBER)((size_t)&(TYPE *)0->MEMBER)
    #define container_of(ptr, type, member)({       \
        const typeof(((type *)0)->member)*_mptr = (ptr); \
        (type *)((char *)_mptr - offsetof(type, member)); \
    })

`container_of`的主要作用就是，根据结构体某一成员的地址，获取这个结构体的首地址，`type`为结构体类型，`member`为结构体内的成员，`ptr`为结构体成员`member`的地址。

这个宏在内核中非常重要，Linux内核中有大量的结构体，往往一个结构体中嵌套了多层结构体。内核中不同层次的子系统或模块，使用的就是对应的不同封装程度的结构体。

## container_of宏实现

当我们定义一个结构体变量时，编译器要给这个变量在内存中分配存储空间。根据每个成员的数据类型和字节对齐方式，编译器会按照结构体中各个成员的顺序，在内存中分配一片连续的空间来存储它们。

一个结构体数据类型，在同一个编译环境下，各个成员相对于结构体首地址的偏移是固定不变的。

根据这个特征即可根据成员的地址减去该成员在结构体内的偏移，得到结构体的首地址。

`container_of`是一个语句表达式，其值为最后一个表达式的值`(type *)((char *)_mptr - offsetof(type, member));`，其含义为：取结构体某个成员`member`的地址，减去这个成员在结构体`type`中的偏移，运算结果就是结构体`type`的首地址。

    #define offsetof(TYPE, MEMBER)((size_t)&((TYPE *)0)->MEMBER)

这个宏有两个参数，一个是结构体类型`TYPE`，另外一个是结构体成员`MEMBER`。技巧是将0强制转换成为一个指向`TYPE`类型的结构体常量指针，然后通过这个常量指针访问成员，获取`MEMBER`的地址，其大小在数值上等于`MEMBER`成员在结构体`MEMBER`中的偏移。

    const typeof(((type *)0)->member)*_mptr = (ptr);

结构体成员的数据类型可以是任意的数据类型，所以定义一个临时的指针变量`_mptr`，用来存储结构体成员`MEMBER`的地址，即`ptr`。`typeof(((type *)0)->member)`用来获取结构体成员的数据类型，然后使用该类型定义一个指针变量。

最后需要注意的是，因为返回的是结构体的首地址，所以整个地址还必须要强制转换一下`type *`，返回一个指向`type`结构体类型的指针。

# 零长度数组

`GNU C`编译器和`C99标准`支持变长数组。

    int len;
    int a[len];

数组长度在编译时未确定，在程序运行时才确定。

`GNU C`编译器还支持零长度数组

    int a[0];

零长度数组不占用内存空间，`sizeof(a) == 0`。

## 零长度数组使用示例

零长度数组常以变长结构体的形式出现。

    struct buffer{
        int len;
        int a[0];
    };

    int main(void)
    {
        struct buffer *buf;
        buf = (struct buffer *)malloc(sizeof(struct buffer)+20);
        buf->len = 20;
        strcpy(buf->a, "hello zhaixue.cc!\n");
        puts(buf->a);

        free(buf);
        return 0;
    }

申请了24字节空间，4字节表示内存长度20，剩下的20字节表示可以使用的内存空间，可以通过结构体成员`a`直接访问这片内存。

## 内核中的零长度数组

零长度数组在内核中一般以变长结构体形式出现，如`USB`驱动使用的`URB`（`USB`请求块结构体：用来传输`USB`数据包）

    struct urb{
        struct kref kref;
        ...
        struct usb_iso_packet_descriptor iso_frame_desc[0];
    };

`USB`有多种传输模式，不同的`USB`设备对传输速度、传输数据安全性以及传输模式不同。`USB`驱动可以根据一帧图像数据的大小，灵活地申请内存空间，以满足不同大小的数据传输，而且零长度数组还不占用存储空间。

## 指针和零长度数组

|数组名和指针的区别|说明|
|-|-|
|数组名|用来表征一块连续内存空间的地址，编译器不会再给它分配一个单独的存储空间|
|指针|是一个变量，编译器需要单独分配一个内存空间，用来存放它指向的变量的地址|

如果使用指针，指针本身占用存储空间，零长度数组不会对结构体造成冗余。

# 属性声明：section

## GNU C编译器扩展关键字：\_attribute\_

`GNU C`增加了`_attribute_`关键字用来声明一个函数、变量或者类型的特殊属性。

    _attribute_ ((ATTRIBUTE))

`ATTRIBUTE`表示要声明的属性。
- `section`
- `aligned`
- `packed`
- `format`
- `weak`
- `alias`
- `noinline`


其中，`aligned`和`packed`用来显示指定一个变量的存储对齐方式。正常情况下，定义一个变量时，编译器会根据变量类型给这个变量分配合适大小的存储空间，按照默认的边界对齐方式分配一个地址。

    char c2_attribute_((aligned(8))) = 4;           //按照8字节地址对齐
    int global_val_attribute_((section(".data")));  //属性参数是字符串，需要加""
    char c2_attribute_((packed, aligned(4)));       //属性之间通过逗号,隔开
    _attribute_((packed, aligned(4))) char c2 = 4;  //属性声明需要紧挨着变量

## 属性声明：section

> `section`：在程序编译的时候，将一个函数或者变量放到指定的段，即放到指定的`section`中。

## U-boot镜像自复制分析

> U-boot的用途是加载Linux内核镜像到内存，给内核传递启动参数，然后引导Linux操作系统启动。

`U-boot`一般存储在`NOR Flash`或者`NAND Flash`上。`U-boot`其本身在启动过程中，都会从`Flash`存储介质上加载自身代码到内存，然后进行重定位，跳到内存`RAM`中执行。


`U-boot`是如何识别自身代码的？如何知道从哪里开始复制代码？如何知道复制到哪里停止？

`U-boot`源码中有一个零长度数组。

    char _image_copy_start[0] _attribute_((section("._image_copy_start")));
    char _image_copy_end[0] _attribute_((section("._image_copy_end")));

在`U-boot`的链接脚本中，`_image_copy_start`和`_image_copy_end`两个`section`在链接的时候分别链接在了代码段`.text`的前面和`.data`段的后面，作为`U-boot`复制自身代码的起始地址和结束地址。这两个`section`中只放了两个零长度数组，没有放其他变量。就代表了`U-boot`镜像要复制自身镜像的起始地址和结束地址。只要知道了这两个地址，就可以直接调用相关代码进行复制。

真正的复制是由一段汇编代码完成复制的。

# 属性声明：aligned

## 地址对齐：aligned

> `aligned`：指定一个变量或者类型的对齐方式。注意，地址对齐的方式为2的幂次方，否则编译会报错。

地址对齐会造成一定的内存空洞，但是有一个主要的原因在于，对齐设置可以简化`CPU`和内存`RAM`之间的接口和硬件设计，一个32位的计算机系统，在`CPU`读取内存时，硬件设计上可能只支持4字节或4字节倍数对齐的地址访问，`CPU`每次向内存读写数据时，一个周期可以读取4字节，如果将`int`放在4字节对齐的地址上，那么`CPU`一次就可以把数据读写完，如果`int`放在非4字节对齐的地址上，`CPU`可能要分两次才能把4字节大小的数据读写完毕。

虽然边界字节对齐会造成一些内存空洞，但是在硬件设计上却大大的简化了。

## 结构体对齐

编译器在给一个结构体分配存储空间的时候，不仅要考虑结构体内部成员的地址对齐，还要考虑结构体整体的对齐。可能会在结构体内填充一些空间，也可能在结构体的末尾填充一些空间。

> 结构体整体对齐规则：按照结构体所有成员中最大对齐字节数或者其整数倍对齐。或者说结构体整体长度为其最大成员字节数的整数倍。如果不是则要进行补齐。

结构体成员按照不同的顺序摆放，肯能会导致结构体的整体长度不一样

    struct data{
        char a;         //1字节
        int b;          //4字节
        short c;        //2字节
    };                   //总长度1+3+4+2=10字节

    struct data{
        char a;
        short b;
        int c;
    };                   //总长度1+2+1+4=8字节

    struct data{
        char a;
        short b _attribute_((aligned(4)));
        int c;
    };                   //总长度1+3+4+4=12字节

    struct data{
        char a;
        short b;
        int c;
    }_attribute_((aligned((16))));  //8字节之后按照16字节对齐，总长度为16字节

## 编译器一定会按照aligned的方式对齐吗

一个编译器，对每个基本数据类型都有默认的最大边界对齐字节数。如果超过了，编译器只能按照它规定的最大对齐字节数来给变量分配地址。

## 属性声明：packed

`aligned`一般用来增大变量的地址对齐，`packed`一般用来减少地址对齐，指定变量或类型使用最可能小的地址对齐方式。

    struct data{
        char a;
        short b _attribute_((packed));
        int c _attribute_((packed));
    };                                  //总长度1+2+4=7字节

    struct data{
        char a;
        short b;
        int c;
    } _attribute_((packed));        //对整个结构体添加packed属性，和分别对每个成员添加packed属性的效果是一样的，总长度也是7字节


## 内核中的aligned、packed声明

在Linux内核源码中，可以看到`aligned`和`packed`一起使用，这样的好处在于，即避免了结构体内各成员因地址对齐不产生内存空洞，又指定了整个结构体的对齐方式。

    struct data{
        char a;
        short b;
        int c;
    } _attribute_((packed, aligned(8)));        //总长度为8字节



# 属性声明：format

## 变参函数的格式检查

`GNU`通过`_attribute_`扩展的`format`属性，指定变参函数的参数格式检查。

    _attribute_((format(archetype, string-index, first-to-check)))
    void LOG(const char *fmt, ...) _attribute_((format(printf,1,2)));

在企业项目中，常常会实现一些自定义的打印调试函数，甚至实现一个独立的日志打印模块。这种一般都是变参函数。编译器在编译程序的时候，就是通过`_attribute_`的`format`属性来进行检查。

`LOG()`函数的属性，`format(printf,1,2)`，告诉编译器，按照`printf()`函数的标准来检查。第2个参数表示`LOG()`函数所有的参数列表中格式字符串的位置索引，第3个参数是告诉编译器要检查的起始位置。

即告诉编译器，`LOG()`函数的参数，其格式字符串的位置在所有的参数列表中的索引是1，即第一个参数，要编译器帮忙检查的参数，在所有的参数列表里的索引是2。知道了`LOG()`参数列表中格式字符串和要检查的参数位置，编译器就会按照检查`printf()`的格式打印一样，对`LOG()`进行参数检查。

也可以将`LOG()`定义为下面的形式

    void LOG(int num, char *fmt, ...) _attribute_((format(printf,2,3)));

## 变参函数的设计和实现

    //变参函数1.0
    void print_num(int count, ...)
    {
        int *args;
        args = &count + 1;
        for(int i = 0; i<count; i++)
        {
            printf("*args :%d\n", *args);
            args++;
        }
    }
    //调用
    print_num1(5, 1, 2, 3, 4, 5);

    //变参函数2.0
    void print_num2(int count, ...)
    {
        char *args;
        args = (char *)&count + 4;
        for(int i = 0; i<count; i++)
        {
            printf("*args :%d\n", *(int *)args);
            args += 4;
        }
    }
    //调用
    print_num2(5, 1, 2, 3, 4, 5);


    //变参函数3.0 
    #include <stdarg.h>
    void print_num3(int count, ...)
    {
        va_list args;
        va_start(args, count);
        for(int i = 0; i<count; i++)
        {
            printf("*args :%d\n", *(int *)args);
            args += 4;
        }
        va_end(args);
    }
    //调用
    print_num3(5, 1, 2, 3, 4, 5);

    //变参函数4.0
    #include <stdio.h>
    #include <stdarg.h>
    void my_printf(char *fmt, ...)
    {
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
    }
    //调用函数
    my_printf("I am qintian, I have 0 car\n", num);

    //变参函数5.0
    void _attribute_ ((format(printf,1,2)))
    my_printf(char *fmt, ...)
    {
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
    }

    //调用
    my_printf("I am qintian, I have 0 car\n", num);

变参函数的参数存储其实和`main()`函数的参数存储有点像，由一个连续的参数列表组成，列表里面存放的是每个参数的地址。

- 变参函数1.0中，首先获取固定参数`count`的地址，`&count+1`就是下一个参数的地址，使用指针变量`args`保存这个地址，并依次访问下一个地址，就可以直接打印传进来的各个实参值。

- 变参函数2.0中，使用`char *`类型指针，因为每一个参数的地址都是4字节大小，所以是`(char *)&count + 4`。对于一个指向`int`类型的指针变量`p`，`p+1`表示`p+1*sizeof(int)`，对于一个指向`char`类型的指针变量`p`，`p+1`表示的是`p+1*sizeof(char)`。
- 变参函数3.0中，编译器提供了一些宏：
  - `va_list`：定义在编译器头文件`stdarg.h`中，如`typedef char* va_list`
  - `va_start(fmt, args)`：根据参数`args`的地址，获取`args`后面参数的地址，并保存在`fmt`指针变量中。
  - `va_end(args)`：释放`args`指针，将其赋值为`NULL`
- 变参函数4.0中，使用`vprintf()`函数完成打印，定义在头文件`stdio.h`中
        
        CRTIMP int _cdecl _MINGW_NOTHROW \
            vprintf(const char*, _VALIST);
        
    `vprintf()`有两个参数，一个是格式字符串指针，一个是变参列表
- 变参函数5.0在变参函数4.0实现自己的打印函数的基础上，添加了`format`属性声明，让编译器在编译的时候检查参数格式。

## 实现自己的日志打印函数

自己实现的打印函数，除了可以实现自己需要的格式，还可以控制打印开关和优先级。

    #define DEBUG   //打印开关

    void _attribute_ ((format(printf,1,2)))
    LOG(char *fmt, ...)
    {
        #ifdef DEBUG
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
        #endif 
    }

    int main(void)
    {
        int num = 0;
        LOG("I am qintian, I have %d car\n", num);
        return 0;
    }

当关闭`DEBUG`宏时，`LOG()`就是一个空函数，开启之后就可以实现正常的打印功能。此外，还可以根据宏来设置打印等级，如可以分为`ERROR`，`WARNING`，`INFO`等打印等级。

    #include <stdio.h>
    #include <stdarg.h>

    #define ERR_LEVEL 1
    #define WARN_LEVEL 2
    #define INFO_LEVEL 3 

    #define DEBUG_LEVEL 3

    void _attribute_ ((format(printf,1,2)))
    INFO(char *fmt, ...)
    {
        #if (DEBUG_LEVEL >= INFO_LEVEL) 
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
        #endif 
    }

    void _attribute_ ((format(printf,1,2)))
    WARN(char *fmt, ...)
    {
        #if (DEBUG_LEVEL >= WARN_LEVEL) 
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
        #endif 
    }

    void _attribute_ ((format(printf,1,2)))
    ERR(char *fmt, ...)
    {
        #if (DEBUG_LEVEL >= ERR_LEVEL) 
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
        #endif 
    }

    int main(void)
    {
        ERR("ERR log level: %d\n", 1);
        WARN("WARN log level: %d\n", 2);
        INFO("INFO log level: %d\n", 3);
        return 0;
    }

在实际的调试中，可以根据自己需要的打印信息，设置合适的打印等级。

# 属性声明：weak

## 强符号和弱符号

`GNU C`通过`weak`属性声明，可以将一个强符号转化成为弱符号。

    void _attribute_((weak)) func(void);
    int num _attribute_((weak));

不管是函数名还是变量名，在编译器的眼里都是一个符号。
- 强符号：函数名，初始化的全局变量名
- 弱符号：未初始化的全局变量名

在一个项目中，不能同时存在两个强符号。如在一个多文件工程中定义两个同名的全局变量或者函数，链接器链接的时候就会报重定义错误。但是在一个工程中，允许强符号和弱符号同时存在，如定义一个初始化的全局变量和未初始化的全局变量，编译器在做符号决议时，一般会选择强符号而丢弃弱符号。

当同名的符号都是弱符号时，编译器会选择内存中存储空间大的那个弱符号。

## 函数的强符号和弱符号

函数名本身就是强符号，同名函数编译时会报错，也可以通过`weak`属性声明将其中一个函数名转化成弱符号

## 弱符号的用途

在一个源文件中引用一个变量或者函数，当编译器只看到声明而没有看到定义时，编译器一般不会报错：编译器会认为这个符号可能在其他的文件中定义。在链接阶段，会去其他文件中找这些符号的定义，若未找到则报未定义错误。

当函数被声明为一个弱符号时，当链接器找不到这个函数的定义时，也不会报错。编译器会将这个函数名，即弱符号，设置为0或者一个特殊的值。只有当程序运行时，调用到这个函数，跳转到零地址或者一个特殊的地址才会报错，产生一个内存错误。

    //fun.c
    int a = 4;

    //main.c
    int a _attribute_((weak)) = 1;
    void _attribute_((weak))func(void);

    int main(void)
    {
        printf("main: a = %d\n", a);
        func();
        return 0;
    }

    #gcc -o a.out main.c fun.c
    #./a.out
    main: a = 4
    Segmentation fault(core dumped)

为了防止错误，可以在运行这个函数之前，先进行判断，看这个函数的地址是不是0，然后决定是否调用和运行，避免出现段错误。

    //fun.c
    int a = 4;

    //main.c
    int a _attribute_((weak)) = 1;
    void _attribute_((weak))func(void);

    int main(void)
    {
        printf("main: a = %d\n", a);
        if(func)                        //函数名的本质就是一个地址
        {
            func();
        }
        return 0;
    }

这一特性在开发库时用的很广泛，如果库中有些高级功能没有实现，可以通过`weak`属性声明将这些函数转化成一个弱符号。通过这样的设置，即使还没有定义函数，只要在调用之前做一个非0的判断就可以，不会影响程序的正常运行。等以后开发出了新的库，实现了这些高级功能，不用修改应用程序，直接运行就可以调用这些功能。

## 属性声明：alias

`alias`属性可以用来给函数定义一个别名。

    void _f(void)
    {
        printf("_f\n");
    }

    void f() _attribute_((alias("_f")));    //给_f()函数定义一个别名f()

    int main(void)
    {
        f();
        return 0; 
    }

在内核中，随着版本的升高，函数的接口发生了变化，可以通过`alias`属性对这个旧的接口名字进行封装，重新起一个接口名字。

    //f.c
    void _f(void)
    {
        printf("_f\n");
    }
    void f() _attribute_((weak,alias("_f")));

    //main.c
    void _attribute_((weak)) f(void);

    void f(void)
    {
        printf("f()\n");
    }

    int main(void)
    {
        f();
        return 0;
    }

如果`main.c`中定义了`f()`函数，则使用`main.c`中的`f()`，如果没有被定义时，则使用`_f()`。

# 内联函数

## 属性声明：noinline

    static inline _attribute_((noinline)) int func();
    static inline _attribute_((always_inline)) int func();

一个使用`inline`声明的函数称为内联函数，内联函数一般前面会有`static`和`extern`修饰。使用`inline`声明一个内联函数，和使用关键字`register`声明一个寄存器变量一样，只是建议编译器在编译时内联展开。


使用`register`修饰一个变量，只是建议编译器在为变量分配存储空间时，将这个变量放到寄存器中，使得程序的运行效率更高，但是编译器具体会不会放，需要视具体情况而定，根据资源是否紧张，这个变量的类型以及是否频繁使用来权衡。

同理，当一个函数使用`inline`关键字修饰时，编译器也不一定会内联展开。但是当使用`noinline`和`always_inline`对一个内联函数做显示声明后，编译器就确定了。`noinline`告诉编译器不内联展开；`always_inline`告诉编译器要内联展开。


## 什么是内联函数

函数的调用过程一般会执行下面的过程：

- 保存当前函数的现场，将当前函数的返回地址、寄存器等现场信息保存在堆栈中；
- 跳到调用函数执行；
- 恢复当前函数现场，将保存到堆栈中的返回地址赋值给PC指针；
- 继续执行当前函数。

假设一个函数很短，被大量调用，按照上面的过程就会花费很大开销。这个时候就可以把这个函数声明为内联函数。编译器在编译过程中遇到内联函数，像宏一样，将内联函数在调用处直接展开，这样就会减少了函数调用的开销：直接执行内联函数展开的代码，不用再保存现场和恢复现场。


## 内联函数和宏

内联函数和宏的功能差不多，但是有以下优点：
1. 参数类型检查：内联函数虽然具有宏的展开特性，但其本质仍是函数，在编译过程中，编译器仍可以对其进行参数检查，而宏不具备这个功能。
2. 便于调试，函数支持的调试功能有断点、单步等，内联函数同样支持。
3. 返回值：内联函数有返回值，返回一个结果给调用者。这个优势是相对于`ANSI C`，因为现在的宏也可以有一个返回值和类型了，如前面的语句表达式定义的宏。
4. 接口封装：有些内联函数可以用来封装一个接口，而宏不具备这个特性。

## 编译器对内联函数的处理

函数的作用之一就是提高代码的复用性。但是内联函数往往又降低了函数的复用性。

一般来说，判断一个内联函数是否展开，从程序员的角度出发，需要考虑：
1. 函数体积小
2. 函数体内无指针赋值、递归、循环语句等
3. 调用频繁


当编译器对内联函数展开时，会直接在调用处展开内联函数的代码，不再给函数本身生成单独的汇编代码。


## 内联函数为什么定义在头文件中

内联函数为什么定义在头文件中？

因为内联函数可以像宏一样使用，任何想要使用这个内联函数的源文件，都不必再定义一遍，直接包含这个头文件，即可像宏一样使用。

使用`static`修饰，是因为使用`inline`定义的内联函数，编译器不一定会内联展开，当一个工程中多个文件都包含这个内联函数的定义时，编译就有可能会报重定义错误，而使用`static`关键字修饰，则可以将这个函数的作用域限制在各自的文件内，避免重定义错误发生。


# 内建函数

## 什么是内建函数

> 内建函数：编译器内部实现的函数，这些函数和关键字一样，可以直接调用，无需向标准库函数那样，要先声明后使用。

内建函数的命名，通常以`_builtin`开头。这些函数主要在编译器内部使用，主要是为了编译器服务的。内建函数的主要用途如下：

- 用来处理变长参数列表；
- 用来处理程序运行异常、编译优化、性能优化
- 查看函数运行时的底层信息、堆栈信息
- 实现C标准库的常用函数

这些内建函数主要供与编译器相关的工具和程序调用，变动较频繁，对于应用程序开发者来说，不建议使用这些函数。

但是了解有些函数，对于了解程序底层运行机制，编译优化有帮助，在Linux内核中也经常使用这些函数，所以有必要了解Linux内核中常用的一些内建函数。

## 常用的内建函数

常用的内建函数：`_builtin_return_address()`和`_builtin_frame_address()`

    _builtin_return_address(LEVEL);

这个函数用来返回当前函数或调用者的返回地址，参数`LEVEL`表示函数调用链中不同层级的函数。

- 0：获取当前函数的返回地址；
- 1：获取上一级函数的返回地址；
- 2：获取上二级函数的返回地址；
- ...

每一层函数调用，都会将当前函数的下一条指令地址，即返回地址压入堆栈保存，各级函数调用就构成了一个函数调用链。

    _builtin_frame_address(LEVEL);

函数每调用一次，都会将当前函数的现场保存在栈中，每一层函数调用都会将各自的现场信息保存在各自的栈中。这个栈就是当前函数的栈帧，每一个栈帧都有起始地址和结束地址，多层函数调用就会有多个栈帧，每个栈帧都会保存上一层栈帧的起始地址，这样各个栈帧就形成了一个调用链。

很多调试器其实都是通过回溯函数的栈帧调用链来获取函数底层的各种信息的，如返回地址、调用关系等。在`ARM`处理器平台下，一般使用`FP`和`SP`两个寄存器，分别指向当前函数栈帧的起始地址和结束地址。当函数继续调用其他函数，或者运行结束返回上一级函数时，这两个寄存器的值也会发生变化，总是指向当前函数栈帧的起始地址和结束地址。


内建函数`_builtin_frame_address(LEVEL)`查看函数的栈帧地址。
- 0：查看当前函数的栈帧地址；
- 1：查看上一级函数的栈帧地址；
- ...


## C标准库的内建函数

在`GNU C`编译器内部，C标准库的内建函数实现了一些与C标准库函数类型的内建函数。这些函数与C标准库功能类似，函数名也相同，只是在前面增加了一个前缀`_builtin`。如果不想使用C标准库函数，也可以加一个前缀，直接使用对应的内建函数。

    int main(void)
    {
        char a[100];
        _builtin_memcpy(a, "hello world!", 20);
        _builtin_puts(a);

        return 0;
    }

## 内建函数：_builtin_constant_p(n)

`_builtin_constant_p(n)`函数主要用来判断参数`n`在编译时是否为常量。如果是常量，则返回1，否则返回0。该函数常用于宏定义中，用来编译优化。一个宏定义，根据宏的参数是常量还是变量，可能实现的方法不一样。在内核源码中，有下面这样的宏

    #define _dma_cache_sync(addr, sz, dir)              \
    do{
        if(_builtin_constant_p(dir))                    \
            _inline_dma_cache_sync(addr, sz, dir);      \
                            \
        else
            _arc_dma_cache_sync(addr, sz, dir);         \
                            \
    }
    while(0);


## 内建函数：_builtin_expect(exp, c)

内建函数`_builtin_expect(exp, c)`有两个参数，返回值就是其中一个参数，仍是`exp`。这个函数的意义在于告诉编译器：参数`exp`的值为`c`的可能性比较大，然后编译器根据这个提示信息，做一些分支预测上的代码优化。无论`c`为何值，函数的返回值都是`exp`。

    int main(void)
    {
        int a;
        a = _builtin_expect(3, 1);
        printf("a = %d\n", a);

        a = _builtin_expect(3, 10);
        printf("a = %d\n", a);

        a = _builtin_expect(3, 100);
        printf("a = %d\n", a);

        return 0;
    }

程序运行结果为

    a = 3
    a = 3
    a = 3

这个函数的主要作用就是编译器的分支预测优化。

之前介绍过`Cache`，如程序在执行过程中遇到函数调用，`if分支`，`goto`等程序结构，会跳到其他地方执行，原先缓存到`Cache`中的指令不是`CPU`要执行的指令。一般建议将大概率发生的分支写在前面。

## Linux内核中的likely和unlikely

在Linux内核中，使用了`_builtin_expect()`内建函数，定义了两个宏。

    #define likely(x) _builtin_expect(!!(x), 1)
    #define unlikely(x) _builtin_expect(!!(x), 0)

这两个宏的主要作用就是告诉编译器，某一个分支发生的概率很高，或者很低。这个宏定义中，对宏的参数`x`做两次取非操作，将参数`x`转换成布尔类型，然后直接和1和0直接作比较，告诉编译器`x`为真或者假的可能性很高。

    int main(void)
    {
        int a;
        scanf("%d", &a);
        if(unlikely(a == 0))
        {
            printf("%d\n", 1);
            printf("\n");
        }
        else
        {
            printf("%d\n", 2);
            printf("\n");
        }
        return 0;
    }

反汇编这段代码，编译器就会将小概率发生的`if`分支的汇编代码放在后面，将大概率的`else`分支的汇编代码放在前面。

# 可变参数宏

之前介绍了可变参数函数，`GNU C`将宏定义也支持了可变参数。

## 什么是可变参数宏

可变参数宏在`C99`中已经支持，但是只有`CNU C`编译器很好的支持了这个功能。

先看使用可变参数函数实现的`LOG()`函数

    void _attribute_((format(printf, 2, 3)))
    LOG(int k, char *fmt, ...)
    {
        va_list args;
        va_start(args, fmt);
        vprintf(fmt, args);
        va_end(args);
    }

使用可变参数宏，定义`LOG()`函数

    #define LOG(fmt, ...) printf(fmt, _VA_ARGS_)
    #define DEBUG(fmt, ...) printf(_VA_ARGS_)

    int main(void)
    {
        LOG("Hello! I'm %s\n", "qintian");
        DEBUG("Hello! I'm %s\n", "qintian");
        return 0;
    }

可变参数宏的实现形式和可变参数函数差不多：用`...`表示变参列表，变参列表由不确定的参数构成，各个参数之间用逗号隔开。可变参数宏使用`C99`标准新增加的一个`_VA_ARGS_`预定义标识符来表示前面的变参列表，而不是像变参函数，使用`va_list`，`va_start`，`va_end`这些宏去解析变参列表。预处理器在将宏展开时，会用变参列表替换掉宏定义中的所有`_VA_ARGS_`标识符。


    #define LOG(fmt, ...) printf(fmt, _VA_ARGS_)

这个宏有一个漏洞，就是当变参为空时，宏展开就会产生一个语法错误。

    int main(void)
    {
        LOG("hello\n");
        return 0;
    }

这个宏展开时，就会变成`printf("hello\n", );`产生语法错误。

## 改进宏

使用宏连接符`##`来改进上面的宏，宏连接符`##`的主要作用就是连接两个字符串。预处理器在将宏展开时，会将`##`两边的字符合并，并删除`##`这个连接符。

    #define A(x) a##x

    int main(void)
    {
        int A(1) = 2;       //int a1 = 2;
        int A() = 3;        //int a = 3;
        printf("%d %d\n", a1, a);
        return 0;
    }

对宏`LOG`进行修改

    #define LOG(fmt, ...) printf(fmt, ##_VA_ARGS_)

    int main(void)
    {
        LOG("hello\n");
        return 0;
    }

当变参列表非空时，`##`的作用是连接`fmt`和变参列表，各个参数之间用逗号隔开，宏可以正常使用；
当变参列表为空时，`##`还有一个特殊作用，将固定参数`fmt`后面的逗号删除掉，这样宏就可以正常使用了。

## 可变参数宏的另一种写法

除了使用`_VA_ARGS_`表示变参列表，还可以使用下面的写法

    #define LOG(fmt, args...) printf(fmt, ##args)

`_VA_ARGS_`定义一个变参宏，是`C99`的写法，`args...`是`GNU C`扩展的一个新写法。

## 内核中的可变参数宏

可变参数宏在内核中主要用于日志打印。一些驱动模块或子系统会定义自己的打印宏，支持打印开关，打印格式，优先级控制等功能。如在`printk.h`头文件中

    #if defined(CONFIG_DYNAMIC_DEBUG)
    #define pr_debug(fmt, ...)          \
        dynamic_pr_debug(fmt, ##_VA_ARGS_)
    #elif defined(DEBUG)
    #define pr_debug(fmt, ...)          \
        printk(KERN_DEBUG pr_fmt(fmt), ##_VA_ARGS_)
    #else
    #define pr_debug(fmt, ...)          \
        no_printk(KERN_DEBUG pr_fmt(fmt), ##_VA_ARGS_)
    #endif

    #define dynamic_pr_debug(fmt, ##_VA_ARGS_)          \
    do{
        DEFINE_DYNAMIC_DEBUG_METADATA(descriptor, fmt); \
        if(unlikely(descriptor.flags        \
                    & _DPRINTK_FLAGS_PRINT))    \
            _dynamic_pr_debug(&descriptor, pr_fmt(fmt),\
                            ##_VA_ARGS_);       \
    }while(0)

    static inline _printf(1, 2)
    int no_printk(const char *fmt, ...)
    {
        return 0;
    }

    #define _printf(a, b)  \
    _attribute_((format(printf, a, b)))

这个宏定义了3个版本，在编译内核时有动态调试选项，这个宏就定义为`dynamic_pr_debug`。如果没有配置动态调试选项，则可以通过`DEBUG`宏，来控制宏的打开和关闭。

`no_printk()`是一个内联函数，定义在`printk.h`头文件，而且通过`format`属性声明，指示编译器按照`printf()`的格式做参数检查。

宏`dynamic_pr_debug`采用`do{...}while`结构，这样定义是为了防止宏在条件、选择等分支结构图中展开后，产生歧义。如下

    #define DEBUG()     \
    printf("hello");printf("else\n")

    int main(void)
    {
        if(1)
            printf("hello if\n");
        else
            DEBUG();
        return 0;
    }
程序执行结果为

    hello if
    else

是因为展开后，程序如下

    int main(void)
    {
        if(1)
            printf("hello if\n");
        else
            printf("hello\n");
            printf("else\n");
        return 0;
    }

多条语句在宏调用处直接展开，就破坏了程序原本的`if-else`的分支结构，导致程序逻辑发生了变化，采用`do{...}while(0)`这种结构，可以将宏定义的复合语句包起来。宏展开后，是一个代码块，避免了逻辑错误。