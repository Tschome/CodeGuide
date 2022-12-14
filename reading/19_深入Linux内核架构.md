#### C编译器的工作过程

##### 1.从源代码到机器码程序

- 预处理：所有的预处理器操作都在这一阶段进行。根据编译器的版本，该阶段由一个外部实用程序(cpp)或专门的库函数支持，二者都是由编译器自动启动。在完成预处理之后，从源文件和使用#include指令所包含的所有头文件，会生成一个（大的）输入文件。编译器本身接下来就无须考虑C语言程序分布在几个源文件中的问题了。

- 扫描和解析：程序设计语言的语法可以通过语法规则描述,这些语法规则类似于自然语言(如英语),但这种语法的限制要强得多。尽管用多种方式来表示同一事实可以增加语言的吸引力和微妙性,但在编程语言中必须不惜任何代价避免二义性。该阶段通常由两个密切相关的任务组成。扫描器(又称之为词法分析器)逐字符分析源文件文本,查找程序设计语言的关键字。解析器(又称之为语法分析器)获取由扫描器提供的输入流,输入流的表示已经从源文件的文本表示抽象出来,解析器将根据语言的语法规则来检查所检测到的结构是否正确。它还在计算机内存中创建数据结构,以便作为源代码的更抽象表示,用于计算机的处理(这与程序的实际源代码相反,源代码应该易于读写) 

- 中间代码生成：在生成最终机器代码的路径上,下一步是将扫描器和解析器建立的语法分析树(即,在内存中创建的数据结构)转换为另一种语言,称之为寄存器传输语言(register transferlanguage,简称RTL)。这是一种用于理想机器的汇编语言。这种语言是可以优化的,在很大程度上独立于目标处理器。但这并不意味着,对所有目标处理器来说,编译过程的这一阶段会生成相同的RTL代码。体系结构不同,可能提供的汇编语句也有所不同,在RTL生成期间必须考虑这点。RTL的各个语句已经是非常底层的,它处于高层的C语言到汇编语言之间的过渡路径中。其主要任务是操作寄存器值,以支持被编译程序的执行。当然,其中也有条件语句和其他控制程序流程的机制。但这种中间代码仍然包含了各种高级程序设计语言所共有的各种数据元素和结构(这些并非特定于某种语言如C、Pascal,等等),这些不会出现在纯粹的汇编语言中。

- 优化:程序编译中,计算最密集的阶段就是对以RTL语言编写的中间代码进行优化。优化程序的原因很显然。但编译器如何进行优化?因为其中使用的机制精巧复杂且颇多曲折(总是需要考虑一些微妙的细节),仅编译优化技术就需要一大本书才能讲清楚,GCC就采用了很多优化技术。但本附录至少会说明所采用的一些优化技术。所有优化技术所基于的思想,最初看起来都很简单。但实际上(以及理论上)很难实现。此类选项最重要的包含,对算术表达式的简化(对表达式进行代数重写,使之能够更高效地计算,和/或使用较少的内存)、消除死代码(程序控制流无法到达的部分代码)、合并同一个程序中重复的表达式和代码项、重写程序控制流使之效率更高,等等。本附录将一一讲解这些内容。

- 代码生成:最后一个阶段只关注针对目标处理器生成实际的汇编代码。但这里并不生成一个可执行二进制文件,而是产生一个汇编指令组成的文本文件,由其他外部程序(汇编器和可能的链接器)转换为二进制机器代码。原则上,汇编代码与程序最终的机器代码是等同的,虽然各个指令的语义都已经达到机器层次,但仍然可以被人阅读(而不是机器) 

##### 2.汇编和链接

 在实际编译过程末尾,原来的C语言程序已经被转换为汇编代码,而最后一步向二进制代码的转换基本上不需要编译器的工作,因为剩余的工作由汇编器和链接器(通常也称之为binder,联接器)完成。 

 与编译器的任务相比,汇编器的工作非常简单。各个汇编语句(及其参数)被转换为依处理器类型而不同的专用二进制格式(各个汇编指令都有自身的二进制码表示法。在一些系统上,如IA-32,根据使用的参数类型不同,一个指令可能转换为不同的二进制形式)。汇编器的另一个任务是将常数(如固定的字符串或数值常数)放置到二进制代码中。Linux下通常使用ELF格式(将在附录E中详细讲述)在二进制文件中保存程序代码和数据。除了其他事务之外,链接器必须调整汇编代码中的分支地址。尽管汇编语言源代码中仍然可以引用符号名称(例如,前述的汇编代码调用了标准库中定义的printf函数),但二进制机器码则必须指定相对或绝对的分支地址。例如,“跳转到后面第5个字节”或“转移到位置x” 

##### 3.调用过程

 C语言中一个有趣的方面是过程和函数调用的实现,这不是特定于GNU编译器的。因为在某些时候,内核负责确保汇编语言和C语言代码的互操作性(换言之,从汇编代码中调用C函数),因此重要的是知道函数调用背后的机制。本节根据IA-32体系结构描述了这些机制,当然其他体系结构上的方法通常是类似的。我们根据图C-1,来讨论过程调用所涉及的基本术语。系统栈(system stack)是一个内存区,位于进程地址空间的末端。在将数据压栈时,栈自顶向下增长,这与“增长”这个词所预期的方向刚好相反。该内存区用于为函数的局部变量提供内存。它也支持在调用函数时传递参数。如果调用了嵌套的过程,找会自上而下增长,并接受新的活动记录(activation record)来保存一个过程所需的所有数据。当前执行过程的活动记录,由标记顶部位置的帧指针(frame pointer)和标记底部位置的栈指针(stack pointer)定义。在过程执行时,虽然其顶部的限制是固定的,但底部的限制是可以扩展的(在需要更多空间时)。 

![1656936990485](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1656936990485.png)

 图C-1也给出了第2个栈帧的详述,指出了其组成部分,如下所示。

- 在栈顿顶部是返回地址,以及保存的顿指针值。返回地址指定了在当前过程结束时代码的控制流转向的内存地址,而保存的帧指针则是前一个活动记录的帧指针。在当前过程执行结束后,该帧指针值可用于重建调用过程的栈帧,在试图调试调用栈回溯时,这一点很重要。

- 活动记录的主要部分是为过程局部变量分配的内存空间。在C语言中,这种变量也称之为自动变量(automatic variable) 

- 在函数调用时以参数形式传递到函数的值,存储在栈的底部。

  

  所有常见的体系结构都提供了以下两个栈操作指令。

- push将一个值放置在栈上,并将栈指针减去该值所占用的内存字节数。栈的末端下移到更低的地址。

- pop从栈删除一个值,并相应增加栈指针的值。也就是说,栈的末端上移。

  

  其还提供了以下两个指令,用于调用和退出函数(自动返回到调用过程),它们也会自动操作栈。

- call将指令指针的当前值压栈,跳转到被调用函数的起始地址。

-  return从栈上弹出返回地址,并跳转到该地址。过程的实现必须将return作为最后一条指令,由cal1放置在栈上的返回地址位于栈的底部。

  过程调用因而由以下两个步骤组成。

  (1) 在栈中建立参数列表。传递到被调用函数的第一个参数最后入栈。这使得可以传递可变数目的参数,然后将其从栈上逐一弹出(pop).

  (2) 调用call,这将指令指针的当前值(call之后的下一条指令)压栈,代码的控制流转向被调用的函数。

  被调用的过程负责管理顿指针,需要执行下列步骤。

  (1)前一个顿指针压栈,因而栈指针下移。

  (2)将栈指针的当前值赋值给帧指针,标记当前执行函数的栈区的起始位置。

  (3)执行当前函数的代码。

  (4)在函数结束时,存储的原帧指针位于栈的底部。其值从栈弹出到帧指针寄存器,使之指向前一个函数的栈区起始位置。现在,对当前函数执行call指令时压栈的返回地址位于栈底。

  (5)调用return,将返回地址从栈弹出。处理器转移到返回地址,代码的控制流也返回到调用函数。初看起来,这种方法似乎有些混乱。

  ##### 4.优化

  ​		优化是编译器的一个重要功能,它可以在不修改程序的情况下,生成快速的代码。这使得程序员从微优化(micro-optimization)工作中解放出来。他们可以专注于编写有内容、易于理解的C语言代码,编译器可以将C代码自动转换为尽可能最优的汇编代码。不过,优化是一个很复杂的主题,不仅需要C语言和汇编语言方面大量的编程技巧,而且需要对数学和形式逻辑有深刻的了解。为此,以下各节将只简要概述GCC的优化特性。 

  - 常数优化
  - 循环优化
  - 公共子表达式消除
  - 死代码消除

##### 5.内联函数

##### 6.属性

##### 7.内联汇编

##### 8.__builtin函数

##### 9.指针运算