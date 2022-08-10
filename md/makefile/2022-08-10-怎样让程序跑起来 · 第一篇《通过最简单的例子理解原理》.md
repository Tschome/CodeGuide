

### Makefile的规则

```makefile
target  : prerequisites 
command
#target 也就是一个目标文件,可以是 Object File,也可以是执行文件。还可以是一个标签(Label),对于标签这种特性,在后续的“伪目标”章节中会有叙述。
#prerequisites 就是,要生成那个 target 所需要的文件或是目标。
#command 也就是 make 需要执行的命令。(任意的 Shell 命令)

```

### Makefile示例编写

示例：

```makefile
edit: main.o kdb.o command.o
cc -o edit main.o kdb.o command.o

main.o : main.c defs.h
cc -c main.c

kdb.o : kdb.c defs.h command.h
cc -c kdb.c

command.o : command.c defs.h command.h
cc -c command.c

clean :
rm -rf *.o 
```

在上述的简单makefile里，目标文件（target）包含：执行文件edit和中间目标文件（*.o），依赖文件就是':'之后的那些.c和.h文件。每一个.o文件都有一组依赖文件,而这些.o文件又是执行文件edit的依赖文件。依赖关系实质上说明了目标文件由哪些文件生成，源头在哪。

make会比较targets文件和prerequisites文件的修改日期，如果prerequisites的文件的日期比targets文件要新，或targets不存在，那么make就会执行后续定义的命令。

### make执行之后，他会干什么？

1.在当前目录下找名字叫"Makefile"或"makefile"文件;

2.如果找到，它会找Makefile文件中第一个目标文件（targets），根据示例，他会找到“edit”这个文件，并把这个问价作为最终的目标文件。

3.如果edit文件不存在，或是edit所依赖的.o文件的修改时间要比edit文件新，那么它就会执行后面所定义的命令来生成edit文件。

4.如果edit所依赖的.o文件也存在，那么make会在当前makefile文件中找目标.o文件的依赖性，如果找到了则再根据上述规则生成.o文件。

5.所依赖的.c和.h文件都存在，于是make会生成.o文件，再用.o文件生成执行文件edit。

如果make在期间遇见错误，会直接退出返回。







