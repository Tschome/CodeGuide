### grep

1. 作用

grep 命令可以指定文件中搜索特定的内容,并将含有这些内容的行标准输出。grep 全称是 Global Regular Expression Print,表示全局正则表达式版本,它的使用权限是所有用户。

2. 格式
   grep [options]

3. 主要参数

   ```shell
   [options] 主要参数:
   - c:只输出匹配行的计数。
   - I:不区分大小写(只适用于单字符)。
   - h :查询多文件时不显示文件名。
   - l:查询多文件时只输出包含匹配字符的文件名。
   - n :显示匹配行及行号。
   - s:不显示不存在或无匹配文本的错误信息。
   - v:显示不包含匹配文本的所有行。
   
   pattern 正则表达式主要参数:
   :忽略正则表达式中特殊字符的原有含义。
   ^:匹配正则表达式的开始行。
   $: 匹配正则表达式的结束行。
   \<:从匹配正则表达式的行开始。
   \>:到匹配正则表达式的行结束。
   [ ]:单个字符,如 [A] 即 A 符合要求
   [ - ] :范围,如 [A-Z] ,即 A、B 、C 一直到 Z 都符合要求
   。:所有的单个字符。
   * :有字符,长度可以为0 。
   ```

正则表达式是Linux/Unix 系统中非常重要的概念。 正则表达式(也称为 “regex 或
” “regexp )”是一个可以描述一类字符串的模式(Pattern )。如果一个字符串可以用某个正则表达式
来描述,我们就说这个字符和该正则表达式匹配(Match )。这和 DOS 中用户可以使用通配符 “*”代表任意字符类似。在 Linux 系统上,正则表达式通常被用来查找文本的模式,
以及对文本执行 “搜索-替换 ”操作和其它功能。

#### grep查找文件内容

获取哪些文件包含搜索的内容，并列出文件名

命令格式：grep -H –r "被查找的字符串" 文件目录 | cut -d: -f1 [| uniq]

```shell
#在当前路径下查找"find_String"zifuchuan
grep -r -H "find_String" ./

```



### find查找文件

```
find ./ -name "*.hevc"
```





### mkdir

1. 作用

mkdir 命令的作用是建立名称为dirname 的子目录,与MS DOS 下的 md 命令类似,它
的使用权限是所有用户。

2. 格式

mkdir [options] 目录名

3. [options] 主要参数

- m, -- mode= 模式:设定权限 <模式 >,与 chmod 类似。
- p, -- parents :需要时创建上层目录;如果目录早已存在,则不当作错误。
- v, -- verbose :每次创建新目录都显示信息。
  -- version :显示版本信息后离开。

3. 应用实例

在进行目录创建时可以设置目录的权限,此时使用的参数是"-m ”。假设要创建的目录名
是 “tsk ",让所有用户都有"rwx"( 即读、写、执行的权限),那么可以使用以下命令:

```shell
$ mkdir - m 777 tsk
```

