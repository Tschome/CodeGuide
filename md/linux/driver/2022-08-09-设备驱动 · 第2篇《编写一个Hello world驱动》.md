# 编写一个Hello world驱动



> 哈，这里你可能会有些疑惑，为什么不先写预备知识再写helloworld驱动呢。其中有一个原因，就是有些读者是一个很心急的孩子，和我一样想一口吃个胖子，不过怎么可能呢，但是为了爽一下，所以就先写啦。
>
> 再者，抱着疑问去学习，难道不也是一个很好的习惯吗？（话说驱动型学习真的是一个很好的方法）

------

### 1.helloworld

> 在看代码前，要回想驱动和应用程序的区别哦

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

//谨记，在驱动编写时的入口已经是module_init了哦
module_init(hello_init);
module_exit(hello_exit);
```





### 2. Hello world驱动的Makefile

```makefile
#交叉编译器
CROSS_COMPILE ?= arm-linux-gnu-
#内核源码路径
KERNEL_DIR ?= $(PWD)/../../kernel/
KDIR := ${KERNEL_DIR}

obj-m:= hello.o

all :
	@$(MAKE) -C $(KDIR) M=$(shell pwd) $@

clean :
	rm -rf *.o *.mod.c

```

这里只对Makefile做一个简单的描述，后续将完整描述。

> 有关makefile，你是不是有很多疑惑，
>
> 1. 为什么驱动编写要链接追踪到内核代码呢？:thinking:
> 2. 驱动链接到内核都干了什么事呀？:thinking:
>
> 当然，我也对这很疑惑，但是有这个问题是很好的，是一颗未来内核大佬的苗子，不过对于现在刚入门的你我来说，这些暂时就不要作为重点啦，后续会对这一点进行说明，并在后文附上链接。

