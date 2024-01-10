# 代码复用与分层思想

- 函数级的代码复用：定义一个函数实现某个功能，所有的程序都可以调用这个函数；
- 库级的代码复用：将一些通用的函数打包封装成库，并引出`API`供程序调用，实现库级的代码复用；
- 框架级的代码复用：将一些类似的应用程序抽象成应用骨架，然后进一步慢慢迭代成框架，如`MVC`、`Django`等；
- 操作系统级的代码复用：操作系统也是对任务调度、任务间通信的功能实现，并引出`API`供应用程序调用。

随着系统越来越复杂，集成的模块越来越多，模块之间也会产生依赖关系。为了便于系统的管理和维护，出现了分层思想。计算机系统可以分为应用层、系统层、硬件层。在操作系统内部也会存在各种分层。每一层都是对其下一层的封装，并留出`API`，为上一层提供服务，实现代码复用。

# 面向对象编程基础

## OOP和POP

> `OOP`：面向对象编程是和`POP`（面向过程编程）相对应的一种编程思想。

在面向过程编程中，函数是程序的基本单元，可以把一个问题分解成多个步骤来执行，每一步都可以使用函数来实现。在面向对象的编程中，对象是程序的基本单元，对象是类的实例化，类则是对客观事物抽象而成的一种数据类型，内部包括属性和方法。

面向过程编程侧重于解决问题的步骤过程。如把大象放进冰箱里，分为3步走……

C语言可以通过结构体、函数指针来实现面向对象编程思想。

## 类的封装和实例化

在一个类中，主要包括两种基本成员，属性和方法。如下
    
    //Animal.cpp
    #include <iostream>
    using namespace std;

    class Animal
    {
        public:
            int age;                //属性
            int weight;             //属性
            Animal();               //方法，构造函数
            ~Animal()               //方法，析构函数 
            {
                cout<<"~Animal()..."<<endl;
            }
            void speak(void)        //方法
            {
                cout<<"Animal speaking..."<<endl;
            }
    };

    Animal::Animal(void)
    {
        cout<<"Animal()..."<<endl;
    }

    int main(void)
    {
        Animal animal;
        animal.age = 1;
        cout<<"animal.age = "<<animal.age<<endl;
        animal.speak();
        return 0;
    }

编译运行：

    #g++ Animal.cpp -o a.out
    #./a.out 
    Animal()...
    animal.age = 1
    Animal speaking...
    ~Animal()...

类的成员函数可以在类内部进行定义，也可以先在类内部声明，然后在类的外部定义。在外部定义时，需要使用类成员运算符`::`指定该成员属于哪一个类。当使用一个类去实例化一个对象或者销毁一个对象时，会分别调用类的构造函数和析构函数。

类的本质就是一种数据类型，与C语言的结构体类似，唯一不同的地方在于类的内部可以包含类的方法，即成员函数；而结构体内部只能是数据成员，不能包含函数。一个类定义好之后，就可以使用这个类去实例化一个对象（类似于C语言中使用某种数据类型定义一个变量），然后就可以直接操作该对象，为该对象的属性赋值，或者调用该对象中的方法。

## 继承和多态

对类的封装，目的就是为了继承，通过继承来实现代码复用。动物有很多种，可以再进行细分：针对某种具体的动物，如猫，不需要在`Cat`类中重复定义`Animal`类中的各种属性，通过继承机制，使`Cat`类去继承原先`Animal`类的属性和方法。

    //Cat.cpp

    #include <iostream>
    using namespace std;

    class Animal
    {
        public:
            int age;                //属性
            int weight;             //属性
            Animal();               //方法，构造函数
            ~Animal()               //方法，析构函数 
            {
                cout<<"~Animal()..."<<endl;
            }
            void speak(void)        //方法
            {
                cout<<"Animal speaking..."<<endl;
            }
    };

    Animal::Animal(void)
    {
        cout<<"Animal()..."<<endl;
    }

    Class Cat::public Animal
    {
        public:
            char sex;
            Cat(void){cout<<"Cat()..."<<endl;}
            ~Cat(void){cout<<"~Cat()..."<<endl;}
            void speak()
            {
                cout<<"cat speaking...miaomiao"<<endl;
            }
            void eat(void)
            {
                cout<<"cat eating..."<<endl;
            }
    }

    int main(void)
    {
        Cat cat;
        cat.age = 2;
        cat.sex = 'F';
        cout<<"cat.age:"<<cat.age<<endl;
        cout<<"cat.sex:"<<cat.sex<<endl;
        cat.speak();
        cat.eat();
    }

程序运行结果：

    #g++ cat.cpp -o a.out
    #./a.out
    Animal()...
    Cat()...
    cat.age:2
    cat.sex:F
    cat.speaking...miaomiao
    cat eating...
    ~Cat()...
    ~Animal()...

子类可以通过继承机制，复用父类的属性和方法，还可以在父类的基础上，扩展自己的属性和方法，如`sex`和`eat()`方法。

在继承的过程中，子类重新定义父类的方法一般被称为多态（如上面`Cat`类重新定义的`speak()`方法），一个接口的多种实现，在不同的子类中有不同的实现，通过函数的重载和覆盖，既实现了代码复用，又保持了实现的多样性。

## 虚函数和纯虚函数

因为不同的子类会有不同的叫声，因此`speak()`可以实现也可以不实现。可以使用关键字`virtual`关键字修饰，使用`virtual`关键字修饰的成员函数被称为虚函数。虚函数一般可以用来实现多态，允许使用父类指针来调用子类的继承函数。

    //Virtual.cpp

    #include <iostream>
    using namespace std;

    class Animal
    {
        public:
            int age;                //属性
            int weight;             //属性
            Animal();               //方法，构造函数
            ~Animal()               //方法，析构函数 
            {
                cout<<"~Animal()..."<<endl;
            }
            virtual void speak(void)        //方法
            {
                cout<<"Animal speaking..."<<endl;
            }
    };

    Animal::Animal(void)
    {
        cout<<"Animal()..."<<endl;
    }

    Class Cat::public Animal
    {
        public:
            char sex;
            Cat(void){cout<<"Cat()..."<<endl;}
            ~Cat(void){cout<<"~Cat()..."<<endl;}
            void speak()
            {
                cout<<"cat speaking...miaomiao"<<endl;
            }
            void eat(void)
            {
                cout<<"cat eating..."<<endl;
            }
    }

    int main(void)
    {
        Cat cat;
        Animal *p = &cat;
        p->speak();             //父类指针调用子类中实现的函数实现多态
        cat.speak();
    }

程序运行结果

    Animal()...
    Cat()...
    cat speaking...miaomiao
    cat speaking...miaomiao
    ~Cat()...
    ~Animal()...

纯虚函数的要求比虚函数更加严格一些，它在基类中不实现，但是子类继承后必须实现。含有纯虚函数的类被称为抽象类，如`Animal`类。如果在类中删除`speak()`方法的实现，就可以将其视为一个抽象类，不能使用`Animal`类去实现一种叫做`animal`的实例对象。


# Linux内核中的OOP思想：封装

## 类的C语言模拟实现

使用结构体来模拟一个类，结构体内部不能像类一样可以直接定义函数，但是可以在结构体内部内嵌指针函数来模拟类中的方法。

    struct animal
    {
        int age;
        int weight;
        int (*fp)(void);
    };

如果一个结构体中需要内嵌许多函数指针，则可以将这些函数指针进一步封装到一个结构体中

    struct func_operation
    {
        void (*fp1)(void);
        void (*fp2)(void);
        void (*fp3)(void);
        void (*fp4)(void);
    };

    struct animal
    {
        int age;
        int weight;
        struct func_operation fp;
    };

C语言可以通过在结构体中内嵌另一个结构体或者结构体指针来模拟类的继承

    struct cat
    {
        struct animal *p;
        struct animal ani;
        char sex;
        void (*eat)(void);
    };

测试程序

    //cat.c
    #include <stdio.h>
    
    void speak(void)
    {
        printf("animal speaking...\n");
    }

    struct func_operation
    {
        void (*fp1)(void);
        void (*fp2)(void);
        void (*fp3)(void);
        void (*fp4)(void);
    };

    struct animal
    {
        int age;
        int weight;
        struct func_operation fp;
    };

    struct cat
    {
        struct animal *p;
        struct animal ani;
        char sex;
        void (*eat)(void);
    };

    int main(void)
    {
        struct animal ani;
        ani.age = 1;
        ani.weight = 2;
        ani.fp.fp1 = speak;
        printf("%d %d\n", ani.age, ani.weight);
        ani.fp.fp1();

        struct cat c;
        c.p = &ani;
        c.p->fp.fp1();
        printf("%d %d\n", c.p->age, c.p->weight);
        return 0;
    }

程序运行结果：

    1 2
    animal speaking...
    animal speaking...
    1 2


## 链表的抽象与封装

链表`list`由不同的节点`node`组成，一个链表节点包含两部分：数据域和指针域。

    struct list_node
    {
        int data;
        struct list_node *next;
        struct list_node *prev;
    }

Linux内核为了实现对链表的操作的代码复用，定义了一个通用的链表及相关操作

    struct list_head
    {
        struct list_head *next, *prev;
    }
    void INIT_LIST_HEAD(struct list_head *list);
    int list_empty(const struct list_head *head);
    void list_add(struct liat_head *new, struct list_head *head);
    void list_del(struct list_head *entry);
    void list_replace(struct list_head *old, struct list_head *new);
    void list_move(struct list_head *list, struct list_head *head);

可以将结构体类型`list_head`以及相关的操作看成一个基类，其他的子类如果想要继承父类的属性和方法，直接将`list_head`内嵌到自己的结构体内即可。

    struct my_list_node
    {
        int data;
        struct list_head list;
    };

## 设备管理模型

在Windows下有一个设备管理器，以一个树的结构，将计算机中的所有硬件设备信息进行分类，并显示出来。Linux使用`sysfs`文件系统来显示设备的信息，在`/sys`目录下，会有一个`device`目录，在`device`目录下还有很多分类，在各个分类目录下就是Linux系统下各个具体硬件设备的信息。

Linux内核中定义了一个重要的结构体类型`kobject`用来表示Linux系统中的一个设备。

    struct kobject
    {
        const char *name;
        struct list_head entry;
        struct kobject  *parent;
        struct kset   *kset;
        struct kobj_type *ktype;
        struct kernfs_node *sd
        struct kerf  kerf;
        unsigned int state_initalized = 1;
        unsigned int state_in_sysfs = 1;
        unsigned int state_add_uevent_sent = 1;
        unsigned int state_remove_uevent_sent = 1;
        unsigned int state_suppress = 1;
    }

相同类型的`kobject`通过其内嵌的`list_head`链成一个链表，然后使用另外一个结构体`kset`来指向和管理这个列表。

    struct kset
    {
        struct list_head list;
        spinlock_t       list_lock;
        struct kobject   kobj;
        struct kset_uevent_ops *uevent_ops;
    }

`kset`结构体就是Linux的`/sys`目录下看到的不同设备的分类目录。在这个目录下的每一个子目录，其实都是相同类型的`kobject`集合。然后不同的`kset`组织成树状层次的结构，就构成了`sysfs`子系统。在`kobject`结构体中内嵌了一个结构体`kobj_type`，该结构体封装了很多关于设备插拔、添加、删除的方法。

将`kobject`看做一个基类，其他的字符设备、块设备、USB设备都是它的子类，这些子类通过继承`kobject`基类的`kobject_add()`和`kobject_del()`方法来完成各自设备的注册和注销。以字符设备为例

    strcut cdev
    {
        struct kobject kobj;        //内嵌kobject结构体
        struct module *owner;
        const struct file_operation *ops;       //实现自己的接口，包括read/write/open/close等接口以函数指针的形式封装在结构体中
        struct list_head list;
        dev_t dev;
        unsigned int count;
    }

## 总线设备模型

在Linux中，每个设备都要有一个对应的驱动程序，否则无法对设备进行读写。每一个字符设备，都有与其对应的字符设备驱动程序。每一个块设备，都有与其对应的块设备驱动程序。对于一些总线型的设备，如鼠标，键盘、U盘等设备，都是按照USB标准协议进行的。Linux系统为了实现最大化的驱动代码复用，设计了`设备-总线-驱动`模型。用总线提供的一些方法来管理设备的插拔信息，所有的设备都挂到总线上，总线会根据设备的类型选择合适的驱动匹配。这样相同类型的设备可以共享一个总线驱动。

与总线设备模型相关的结构体有`device`，可以看成是基类`kobject`的子类

    struct device 
    {
        struct device *parent;
        struct device_private   *p;
        struct kobject      kobj;           //内嵌kobject结构体
        const struct device_type   *type;   
        struct bus_type  *bus;
        struct device_driver    *driver;
        void    *plateform_data;
        void    *driver_data;
        dev_t   devt;
        u32     id;
        struct  klist_node    knode_class;
        struct  class     *class;
        void (*release)(struct device *dev);
    }

`device`通过内嵌结构体`kobject`完成对于类`kobject`的继承，同时还内嵌了`bus_type`和`device_driver`，用来表示其挂载的总线和与其匹配的设备驱动。

`device`可以看做是一个抽象类，无法使用它去创建一个具体的设备。其他的设备如USB设备，I2C设备可以通过内嵌`device`结构体完成对`device`类属性和方法的继承。如USB设备的结构体`usb_device`结构体内嵌`device`，而不同类型的USB设备可以内嵌`usb_device`结构体，如USB网卡的结构体`usbnet`就是通过内嵌`usb_device`。

# Linux内核中的OOP思想：继承

封装是为了更好的继承。通过内嵌结构体或者结构体指针来模拟继承，这种方法适用于一级继承。父类和子类相差不大的情况

## 继承与私有指针

C语言还可以通过私有指针进行继承，把使用结构体类型定义各个不同的结构体变量，也可以看做继承，各个结构体变量就是子类，然后各个子类通过私有指针扩展各自的属性和方法。

如网卡，不同的网卡的读写操作基本上都是一样的，唯一不同的就是不同网卡之间存在一些差异，如`I/O`寄存器，中断号等硬件资源等。

将各个网卡一些相同的属性抽取出来，构建一个通用的结构体`net_device`，然后通过一个私有指针，指向每个网卡各自不同的属性和方法。如Linux内核中的`net_device`结构体。

    //bfin_can.c

    struct bfin_can_priv  *priv = netdev_priv(dev);

    struct net_device
    {
        char name [IFNAMSIZ];
        const struct net_device_ops     *netdev_ops;
        const struct ethool_ops     *ethool_ops;
        void *ml_priv;        
        struct device dev;
    }

在`net_device`结构体定义中，有一个私有指针变量`ml_priv`，当使用该结构体类型定义不同的变量表示不同型号的网卡设备时，这个私有指针就会指向各个网卡自身扩展的一些属性（使用结构体`bfin_can_priv`定义）。每个使用`net_device`定义的结构体变量，可以被看做是`net_device`的一个子类，各个子类可以通过自定义的结构体类型，如`bfin_can_priv`在父类的基础上扩展自己的属性或者方法。然后使用结构体中的私有指针`ml_priv`指向它们。

## 继承与抽象类

含有纯虚函数的类，被称为抽象类，抽象类不能被实例化。抽象类的作用，主要就是实现分层：实现抽象层。当父类和子类之间相差太大，无法通过继承实现代码复用，就可以在他们之间添加一个抽象类。抽象类用来管理父类和子类的继承关系，通过分层来提高代码的复用性。

## 继承与接口

> 多重继承：B和C作为A的子类，分别继承了A的属性和方法，D又以B和C为父类进行多路继承。

处理多重继承的方法：将多重继承简化成单继承，另外一个继承通过使用接口代替。如USB网卡驱动，既有USB子系统，又有网络驱动模块。将`usb_device`为基类的这条继承分支作为一个接口来处理，USB网卡通过`usb_device`封装的接口可以实现USB网卡设备的插拔检测、底层数据传输等功能。而对于`net_device`为基类的这路继承，当做一个普通的单继承。



# Linux内核中的OOP思想：多态

使用C语言来模拟多态，把使用同一个结构体类型定义的不同结构体变量看成这个结构体类型的各个子类，那么在初始化各个结构体变量时，如果基类是抽象类，类成员中包含纯虚函数，则可以为函数指针赋予不同的具体函数，然后通过指针调用各个结构体变量的具体函数即可以实现多态。

    #include <stdio.h>

    strcut file_operation
    {
        void (*read)(void);
        void (*write)(void);
    };

    struct file_system
    {
        char name[20];
        struct file_operation fops;
    };

    void ext_read(void)
    {
        printf("ext read...\n");
    }

    void ext_write(void)
    {
        printf("ext write...\n");
    }

    void fat_read(void)
    {
        printf("fat read...\n");
    }

    void fat_write(void)
    {
        printf("fat write...\n");
    }

    int main(void)
    {
        struct file_system ext = {"ext3", {ext_read, ext_write}};
        struct file_system fat = {"fat32", {fat_read, fat_write}};

        struct file_system *fp;
        fp = &ext;
        fp->fpos.read();
        fp = &fat;
        fp->fpos.read();
        return 0;
    }
程序运行结果

    ext read...
    fat read...

同理，USB网卡驱动，当一个指向`net_device`结构体类型的基类指针指向不同的结构体变量时，就可以分别去调用不同的子类（具体网卡）的读写函数，从而实现多态。