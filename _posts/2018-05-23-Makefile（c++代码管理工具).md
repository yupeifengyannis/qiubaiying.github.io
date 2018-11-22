---
layout:     post
title:      Makefile（c++代码管理工具）
subtitle:   
date:       2018-05-23
author:     Yannis
header-img: 
catalog: true
tags:	
   - Makefile
   - c++
---

## Makefile(代码的管理工具)

1、Makefile的命名 ：makefile和Makefile皆可。

2、Makefile的规则：

​	规则的三要素：目标、依赖、命令

​	目标：依赖条件

​	(制表符缩进)命令

makefile1第一个版本，这样每次都要重新编译一下会导致编译耗时很严重

```makefile
myapp:main.cc ./src/add.cc ./src/sub.cc ./src/mul.cc ./src/div.cc
	g++ main.cc ./src/*.cc -o myapp -I ./include
```



makefile2第二个版本，先将所有的依赖全部变为.o文件而不是.cc文件，这样在编译的时候如果我们只是修改了某一个.cc的源文件，这样编译的时候只会编译被修改的文件，而不会去编译没有被修改过的文件，可以节省大量编译的时间，当然系统时通过修改时间这一个参数来判别那些.cc文件需要进行重新编译的。但是像下面写的makefile写的比较杂乱。

```makefile
myapp:main.o ./src/add.o ./src/sub.o ./src/mul.o ./src/div.o
	g++ main.o ./src/*.o -o myapp
	
main.o:main.cc
	g++ -c main.cc -o main.o -I ./include

./src/add.o:./src/add.o
	g++ -c ./src/add.cc -o ./src/add.o -I ./include

./src/sub.o:./src/sub.cc
	g++ -c ./src/sub.cc -o ./src/sub.o -I ./include
        
./src/mul.o:./src/mul.cc
	g++ -c ./src/mul.cc -o ./src/mul.o -I ./include

./src/div.o:./src/div.cc
	g++ -c ./src/div.cc -o ./src/div.o -I ./include

```



第三个版本的makefile，利用自定义的变量和自动变量来使得makefile变得简洁。

```makefile
obj=main.o ./src/add.o ./src/sub.o ./src/mul.o ./src/div.o
target=myapp
include=./include
CC= g++
CPPFLAGS= -I
$(target):$(obj)
	$(CC) $^ -o $@ $(CPPFLAGS) $(include) 
#下面的时模式匹配规则

%.o:%.cc
	$(CC) -c $< -o $@ $(CPPFLAGS) $(include)
```

模式匹配：

%.o:%.cc

​	g++ -c $<  -o  \$ @

makefile中的自动变量

$<：规则中的第一个依赖

$@：规则中的目标

$^：规则中的所有依赖

只能在规则的命令中进行使用

makefile中自己维护的变量：首先他们都是大写，比如CC等等，我们在用的时候可以直接给它赋值，它会将之前的值给覆盖掉。



第4个makefile，这里我们添加了一个clean的功能，可以用于实际的用途

```makefile
obj=main.o ./src/add.o ./src/sub.o ./src/mul.o ./src/div.o
target=myapp
include=./include
CC=g++
CPPFLAGS = -I
$(target):$(obj)
	$(CC) $^ -o $@ $(CPPFLAGS) $(include)
%.o:%.cc
	$(CC) -c $< -o $@ $(CPPFLAGS) $(include)

#伪目标

.PHONY:clean
clean:
	rm $(obj) $(target) -rf
```

[代码链接](https://github.com/yupeifengyannis/misc/tree/master/makefile)

