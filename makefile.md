# 介绍

```makefile
target objfile/prog/label : prerequesites 依赖
	command(shell)
--------------------------------------------------------------------
objects=xx.o xx2.o xx3.o
edit : $(objects)
	cc -o edit $(objects)
xx1.o : xx.c xxx.h
	cc -c xx.c
xx2.o : ...
clean : 
	rm edit $(objects)
-------------------------------------------------------------------
# 自动推导
objects=xx.o xx2.o xx3.o
edit : $(objects)
	cc -o edit $(objects)
xx1.o : xxx.h
xx2.o : xxx.h xxx2.h
...
.PHONY : clean # 表示clean是个伪目标文件
clean :
	rm edit $(objects)
------------------------------------------------------------------
# 另一种方法
objects = main.o kbd.o command.o display.o \
	insert.o search.o files.o utils.o
edit : $(objects)
	cc -o edit $(objects)
	
$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
	rm edit $(objects)
---------------------------------------------------------------------
.PHONY : clean
clean :
	-rm edit $(objects)
---------------------------------------------------------------------
# 引用其他Makefile
include <filename>
include xx.make *.mk $(bar)
# 有-I/--include-dir的话会在这个参数指定的文件夹搜索
# 如果目录<prefix>/include存在的话make也会去找
# 如果你想让 make 不理那些无法读取的文件，而继续执行，你可以在include 前加一个减号“-
-include <filename>

```

# 规则

```makefile
# 依赖关系+生成目标的方法
# 通配符：
	~/test :$HOME目录下的test
	~hchen/test :用户hchen的test
-----------------------------------------------------------------------
print: *.c
    lpr -p $? # 用来将一个或多个文件放入打印队列等待打印。将料资送给本地或远端的主机处理。
    touch print
--------------------------------------------------------------------------
# 让 objects的值是所有 .o 的文件名的集合
objects := $(wildcard *.o)
# 列出所有文件对应的 .o 文件，它是由 make 自动编译出的
$(patsubst %.c,%.o,$(wildcard *.c))
# 编译并链接所有 .c 和 .o 文件
objects := $(patsubst %.c,%.o,$(wildcard *.c))
foo : $(objects)
	cc -o foo $(objects)
---------------------------------------------------------------------------
# 文件搜寻
VPATH = src:../path
# vpath以指定不同的文件在不同的搜索目录中
# 1.vpath <pattern> <directories> 为符合<pattern>的文件指定搜索目录<directories>。
vpath %.h ../headers
# 2.vpath <pattern> 清除符合模式 <pattern> 的文件的搜索目录。
# 3.vpath 清除所有已被设置好了的文件搜索目录。
---------------------------------------------------------------------------
# 伪目标
# “伪目标”并不是一个文件，只是一个标签，make 无法生成它的依赖关系和决定它是否要执行。
# “.PHONY”来显式地指明一个目标是“伪目标"
.PHONY : clean
clean :
	rm *.o temp
# 伪目标同样可以作为“默认目标”，只要将其放在第一个。如果你的 Makefile 需要一口气生成若干个可执行文件，但你只想简单地敲一个 make 完事，并且，所有的目标文件都写在一个 Makefile 中，那么你可以使用“伪目标”这个特性
all : prog1 prog2 prog3
.PHONY : all
prog1 : prog1.o utils.o
	cc -o prog1 prog1.o utils.o
prog2 : prog2.o
	cc -o prog2 prog2.o
prog3 : prog3.o sort.o utils.o
	cc -o prog3 prog3.o sort.o utils.o
# 伪目标同样也可成为依赖
.PHONY : cleanall cleanobj cleandiff
cleanall : cleanobj cleandiff
	rm program
cleanobj :
	rm *.o
cleandiff :
	rm *.diff
---------------------------------------------------------------------------
# 多目标
# 自动化变量 $@:目标的集合,依次取出目标并执行命令
bigoutput littleoutput : text.g
	generate text.g -$(subst output,,$@) > $@ # 执行一个Makefile函数subst
# 等价于
bigoutput : text.g
	generate text.g -big > bigoutput
littleoutput : text.g
	generate text.g -little > littleoutput
--------------------------------------------------------------------------
# 静态模式
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
# 例子1
objects = foo.o bar.o
all: $(objects)
$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@ # $<表示第一个依赖文件 $@表示目标集
# 例子2
files = foo.elc bar.o lose.o
$(filter %.o,$(files)): %.o: %.c # 函数，过滤$files集
    $(CC) -c $(CFLAGS) $< -o $@
$(filter %.elc,$(files)): %.elc: %.el
	emacs -f batch-byte-compile $<
------------------------------------------------------------------------
# 自动生成依赖性
cc -M main.c # 自动寻找源文件中包含的头文件->main.o : main.c xx.h
gcc -MM main.c # gnu编译器用-M会把标准库的头文件也包含进来
# 为每一个源文件name.c的自动生成的依赖关系放到一个Makefile文件name.d中，让make自动更新或生成.d并包含在主Makefile中
%.d: %.c
	@set -e; rm -f $@; \ # 删除所有目标.d
	$(CC) -M $(CPPFLAGS) $< > $@.$$$$; \ # 为每个依赖文件$<生成依赖文件。$$$$随机编号
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \ # sed:替换
	rm -f $@.$$$$ # 删除临时文件
# 引入别的Makefile
sources = foo.c bar.c
include $(source:.c=.d) # 把source中所有的.c替换成.d
```

# 书写命令

```makefile
# 显示命令
@echo xxxxxx
# 调试：
make -n 或 --just-print # 只显示不执行
-------------------------------------------------------------------------
# 命令执行：应该用分号分隔两个命令
exec:
	command1; command2
--------------------------------------------------------------------------
# 命令出错：在命令前加上-，忽略错误。全局方法：make -i,Makefile中所有命令都忽略错误
# make -k 如果某规则出错，终止该规则，继续执行其他规则
-----------------------------------------------------------------------
# 嵌套执行make
subsystem:
	cd subdir && $(MAKE) # =>$(MAKE) -C subdir
# 把变量传递到下级Makefile中：export <variable ...>;
export variable = value
# 不传递：unexport <variable ...>;

```

