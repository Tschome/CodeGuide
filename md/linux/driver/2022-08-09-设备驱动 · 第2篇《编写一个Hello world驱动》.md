# 编写一个Hello world驱动（暂留）

### 预备知识

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/version.h>

//module.h包含了大量加载模块的函数
//init.h包含了初始化和清理函数
//version.h包含在简历的内核版本信息

MODULE_LICENSE("GPL");//内核人事的特定许可
//GPL===>适用GNU通用公共许可的任何版本
//GPL v2===>只适用GPL的版本2
MODULE_AUTHOR("");//声明是谁编写的该模块
MODULE_DESCRIPION("");//该模块是干嘛的描述
MODULE_VERSION("");//模块版本
MODULE_ALIAS("");//模块的另一个为人所知的名称,即别称
MODULE_DEVICE_TABLE();//模块支持哪些设备===》？？？？？

LINUX_VERSION_CODE;	//整型宏定义, 对 #ifdef 版本依赖有用.

EXPORT_SYMBOL (symbol);
EXPORT_SYMBOL_GPL (symbol);
//宏定义, 用来输出一个符号给内核. 第 2 种形式输出没有版本信息, 第 3 种限制输出给 GPL 许可的模块.===》？？？？？
    
#include <linux/moduleparam.h>
module_param(variable, type, perm);
//宏定义, 创建模块参数, 可以被用户在模块加载时调整( 或者在启动时间, 对于内嵌代码). 类型可以是 bool, charp, int, invbool, short, ushort, uint, ulong,或者 intarray.

#include <linux/kernel.h>
int printk(const char * fmt, ...);
//内核代码的 printf 类似物.
```

再加上，上一节所说，驱动程序的入口是module_init()，出口是module_exit()。

> 上述信息如何在终端打印出来呢？
>
> modinfo audio -a
>
> ```shell
> Usage: modinfo [-adlp0] [-F keyword] MODULE               
>      -a              Shortcut for '-F author'           
>      -d              Shortcut for '-F description'     
>      -l              Shortcut for '-F license'         
>      -p              Shortcut for '-F parm'             
>      -F keyword      Keyword to look for               
>      -0              Separate output with NULs  
> ```
>
> 

#### helloworld

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");

static int hello_init(void)
{
	printk(KERN_ALERT "Hello, world\n");
	return 0;
}
static void hello_exit(void)
{
	printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

#### moudle_init（）

使用 moudle_init 是强制的. 这个宏定义增加了特别的段到模块目标代码中, 表明在哪里找到模块的初始化函数. 没有这个定义, 你的初始化函数不会被调用.

#### module_exit()

如果你的模块没有定义一个清理函数, 内核不会允许它被卸载.

一个标识 __exit 的函数只在模块卸载或者系统停止时调用; 任何别的使用是错的. 其次, moudle_exit 声明对于使得内核能够找到你的清理函数是必要的.

#### [printk()]()

### helloworld驱动的Makefile(待验证)

```makefile
CROSS_COMPILE ?= mips-v720s229-linux-gnu-
KERNEL_DIR ?= $(PWD)/../../kernel/
KDIR := ${KERNEL_DIR}
obj-m:= hello.o

all :
	@$(MAKE) -C $(KDIR) M=$(shell pwd) $@

clean :
	rm -rf *.o *.mod.c
#应该如何写这个makefile？？？？？？？？？？？？
```

这里只对Makefile做一个简单的描述，后续将完整描述。

——暂留