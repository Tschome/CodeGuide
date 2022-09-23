#### const *float const *如何理解

1. typedef float *const Ptr;
2. const Ptr *p;

p就是const float *const *类型。

从2来看，首先p是一个指向Ptr类型（而且是常量）的指针

再从1来看，Ptr是一个常量指针，指向一个float类型的变量

因此，const float *const *就是修饰一个指向常量的指针，该常量是一个指向float类型的常量指针。现在尝试使用这种类型：

float f = 0.1f;

float *const p = &f;

const float *const *ptr = &p;

能理解了吗？

###### 疑问？可以理解成常量二级指针吗

