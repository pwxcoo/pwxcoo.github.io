---
title: makefile和常见gcc参数
date: 2018-12-07 19:40:59
categories: memo
tags:
- linux
- gcc
- makefile
---

## makefile

makefile 的大概规则模板: 

```makefile
target ... : prerequisites ...
	command
	...
	...
```

好，就是上面这种样子。讲完了。

----

**makefile 的原理就是根据 target 文件和 prerequisites 文件修改时间的值来确定 target 文件是否需要重新编译。**

而且 prerequisites 也有 prerequisites's prerequisites 的。makefile 文件首先去寻找 prerequisites，如果没有继续递归寻找，过程有点像堆栈的过程，编译的过程有点像拓扑排序的一个过程。

但是 makefile 有一些 tricks 的，所以经常看到一些复杂工程文件的 makefile 很大。

然后有些可以提一下常用的一些东西。

- `\` 用来换行
- makefile 中使用变量，用 `$(variable)` 就可以用了
- `.PHONY` 指明某个 label 是伪目标文件，防止 makefile 中定义的目标和实际工作目录下的实际文件名出现冲突
    ```makefile
    .PHONY : clean
    clean :
        -rm edit $(objects)
    ```
- `$@` 表示目标文件，`$^` 表示全部依赖，`$<` 表示第一个依赖。

一个最简单的 makefile 例子: 

```makefile
CXX = g++
CFLAGS= --std=c++11
SOURCE = lexer.cpp
DEPS = lexer.h
TARGET = lexer

$(TARGET): $(SOURCE) $(DEPS)
	$(CXX) $(CFLAGS) $(SOURCE) -o $(TARGET)
	strip $(TARGET)

clean:
	rm $(TARGET)
```

复杂的话，github 上有很多 makefile 的模板。

## GCC参数

- `x` 指明使用语言的参数
    - 'c', 'objective-c', 'c-header', 'c++', 'cpp-output', 'assembler', 'assembler-with-cpp'
- `c` 预处理，编译，汇编 => obj 文件
- `S` 预处理和编译 => 汇编代码
- `E` 预处理，不生成文件，需要重定向到一个文件里
    ```sh
    gcc -E hello.c > pianoapan.txt 
    gcc -E hello.c | more 
    ```
- `o` 编译成可执行文件
- `-O0 、-O1 、-O2 、-O3` 编译器优化，`O1` 缺省，`O3`级别最高
- `g` 产生调试信息
- `Wall` 生成所有 warning / `w` 不生成 warning

## Other

阿，后天回学校了。今年也马上要到头了，最近有点烦恼阿。

## References

- [跟我一起写Makefile:MakeFile介绍](http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:MakeFile%E4%BB%8B%E7%BB%8D)
- [GCC参数详解](http://www.runoob.com/w3cnote/gcc-parameter-detail.html)