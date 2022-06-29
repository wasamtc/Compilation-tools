# Makefile从入门到~~入门~~精通
[toc]

## 一.为什么需要Makefile

在Linux中使用gcc编译c或cpp文件需要在命令行中输入相应的命令（当然也可以下编译器编译），对于一个或几个文件手输命令还比较容易，但是对于一个大型的项目来说往往有许多文件，这个时候再靠自己输入命令不容易，不方便，易出错，而且如果更改了文件又要重新输入命令了，所以我们需要使用Makefile用来自动编译c或cpp文件，并且当文件改变时可以只重新编译被改变影响的文件。这一切都是通过我们在Makefile文件中写入的命令实现的。即我们希望通过Makefile完成如下目标：

1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
2. 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

## 二.Makefile编写基本规则

### 基本单元

Makefile编写最核心的其实就只有一个格式：

```cpp
目标 : 依赖
TAB键命令
```

**为了之后叙述方便，我们姑且把上面这两行代码叫做基本单元吧。**目标是我们要生成的目标文件，依赖是要生成目标文件所依赖的文件，而命令就是利用依赖文件生成目标文件所需要的命令。例如我们有一个helloworld.cpp程序如下：

```cpp
#include <iostream>

using namespace std;

int main() {
    cout<<"hello world!"<<endl;
    return 0;
}
```

编写Makefile（也可以叫makefile）文件如下：

```makefile
helloworld : helloworld.cpp
	g++ helloworld.cpp -o helloworld
```

然后在命令行输入make回车，就可以看到生成了名字为helloworld的可执行文件，输入./helloworld命令便可以打印出字符串了。

依赖文件也可以是其他命令生成的目标文件，即一个目标文件的依赖文件也可以是其他的目标文件，Makefile执行时若发现目标文件对应的依赖文件还没有生成，则会先执行下面的基本单元来生成上面没有的依赖文件。**即Makefile最终的目的是生成第一个基本单元的目标文件，之后的基本单元也是为这个目标服务的。**

例如对于上面的helloworld编译我们也可以写出这样的Makefile文件：

```makefile
helloworld : helloworld.o
	g++ helloworld.o -o helloworld

helloworld.o : helloworld.cpp
	g++ -c helloworld.cpp -o helloworld.o
```

如果对于c或cpp文件编译过程及相应的命令不熟悉的可以看看下面这篇文章

[使用GCC编译过程及编译程序常用命令](https://blog.csdn.net/qq_42570601/article/details/121261526)

### 其他基本操作

#### 指定文件名

当项目中存在多个Makefile时，可以使用make -f 指定相应的文件进行make，例如：

```bash
make -f test.makefile
```

#### 注释文本

可以在行首使用#对Makefile文本进行注释，例如：

```makefile
helloworld : helloworld.cpp
	g++ helloworld.cpp -o helloworld

#helloworld.o : helloworld.cpp
#	g++ -c helloworld.cpp -o helloworld.o
```

#### 取消回显文本

我们发现当我们make时编译过程会显示出来，即：

```bash
wasam@wasam-laptop:~/alltest/maketest$ make
g++ -c helloworld.cpp -o helloworld.o
g++ helloworld.o -o helloworld
```

我们可以在Makefile文件中的相应基本单元的命令行行首加上@即可取消显示的文本

```makefile
helloworld : helloworld.o
	@g++ helloworld.o -o helloworld

helloworld.o : helloworld.cpp
	@g++ -c helloworld.cpp -o helloworld.o
```

#### 伪目标的使用

有一些目标文件不需要依赖文件，这样的目标文件我们叫做伪目标，例如：

```makefile
helloworld : helloworld.o
	@g++ helloworld.o -o helloworld

helloworld.o : helloworld.cpp
	@g++ -c helloworld.cpp \
 -o helloworld.o
# \是换行符，可以让命令分成多行写。注意换行符的下一行前面不能再是tab键了，需是一个空格。
clean : 
	rm *.o helloworld
# 注意这里伪代码的命令也一定要按照格式来写，即写在目标的下一行并且前面是tab键
```

这里的clean就是一个伪目标，如何执行它呢，因为它不是前面任何目标文件的依赖文件，makefile在执行时不会根据依赖关系执行clean的命令，所以我们要执行clean必须显式的指出，即使用make clean显式的执行clean。

#### 变量的使用

这里先非常简要地说一下变量。为了减少重复代码（如某些文件名要重复在makefile中使用）我们需要用到变量，变量的定义很简单：

```
变量名 = 文件名 文件名 文件名...
```

使用变量即$(变量名)表示变量。

例如：

```makefile
obj = helloworld.o

helloworld : $(obj)
	@g++ helloworld.o -o helloworld

$(obj) : helloworld.cpp
	@g++ -c helloworld.cpp  -o $(obj)

clean : 
	rm *.o helloworld
```

同时系统还存在其他默认的自动化变量，这样可以大大简化makefile文件，便于设计和后期维护，如:

- $^  表示所有的依赖文件
- $@   表示生成的目标文件
- $<  代表第一个依赖文件
- 等等   

例如：

```makefile
obj = helloworld.o

helloworld : $(obj)
	@g++ $^ -o helloworld

$(obj) : helloworld.cpp
	@g++ -c $<  -o $@

clean : 
	rm *.o helloworld

# 这里只是举个例子，实际当然没有人会这么写
```

