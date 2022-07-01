# Makefile从入门到~~入门~~精通
[toc]

本篇文章参考了网络上多篇关于makefile的文章，在文末会一一列出。

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
make -f test.mk
# makefile的后缀可以为mk
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

有一些目标文件不需要依赖文件，这样的目标文件我们叫做伪目标，我们可以用.PHONY点明这样的伪目标，例如：

```makefile
helloworld : helloworld.o
	@g++ helloworld.o -o helloworld

helloworld.o : helloworld.cpp
	@g++ -c helloworld.cpp -o helloworld.o
.PHONY : clean
# 其实也可以不加上面那行，不过一般还是加上
clean : 
	-rm *.o helloworld
# 这里的*是通配符，后面会讲
# 这里rm前面也可以不加-，加-是表示如果删除遇见错误忽视继续删除后面的
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

我们在看网上的一些例子的时候，会发现有的变量定义用的是=，有的是:=，这两者有什么区别呢？

##### makefile中“=”和“:=”的区别

```makefile
  make会将整个makefile展开后，再决定变量的值。也就是说，变量的值将会是整个makefile中最后被指定的值。看例子：

        x = foo
        y = $(x) bar
        x = xyz

  在上例中，y的值将会是 xyz bar ，而不是 foo bar 。

  2、“:=”

  “:=”表示变量的值决定于它在makefile中的位置，而不是整个makefile展开后的最终值。

        x := foo
        y := $(x) bar
        x := xyz

  在上例中，y的值将会是 foo bar ，而不是 xyz bar 了。

```

参考：[makefile中“=”和“:=”的区别](https://blog.csdn.net/PVZHTB/article/details/78229282/)

#### make是如何工作的

1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 `.o` 文件的文件修改时间要比 `edit` 这个文件新，那么，他就会执行后面所定义的命令来生成 `edit` 这个文件。
4. 如果 `edit` 所依赖的 `.o` 文件也不存在，那么make会在当前文件中找目标为 `.o` 文件的依赖性，如果找到则再根据那一个规则生成 `.o` 文件。（这有点像一个堆栈的过程）
5. 当然，你的C文件和H文件是存在的啦，于是make会生成 `.o` 文件，然后再用 `.o` 文件生成make的终极任务，也就是执行文件 `edit` 了。

#### make的自动推导

GNU的make是一个很强大的工具，对于.o文件，如果我们没有显式的把它作为目标文件，那么makefile的隐晦规则会加上一个基本单元，该基本单元把相应.o文件作为目标文件，对应.c或.cpp文件作为依赖文件并且加上编译命令，这个时候我们需要把其他依赖文件写上即可（例如头文件）。例如：

我们有如下的主函数：

```cpp
#include <iostream>
#include "num_max.h"

using namespace std;

int main() {
    int a, b;
    cin>>a>>b;
    NumMax(a, b);
}
```

其中num_max.h为：

```cpp
#include <iostream>

void NumMax(int a, int b) {
    if (a > b)
        std::cout<<a<<std::endl;
    else
        std::cout<<b<<std::endl;
}
```

我们可编写makefile文件如下：

```makefile
main : main.o
	@g++ main.o -o helloworld -o main

main.o : num_max.h
# 这里不用再写命令和main.cpp了，因为makefile帮我们自动添上了

clean : 
	rm *.o main
```

不过也没必要这么写，反正都写了一个基本单元了，何不如直接写完呢。

#### 换行

本来换行不用单独拿出来说的，但是有时候还是就是容易在换行上遇到坑，makefile用 \ 表示换行，特别注意：**在使用换行符的时候要注意在“\”后面不能再加上其他字符，包括注释和空格，**否则make检测到“\”不在一行的最后，就不会把它当成换行符解释，就会出现错误。

下面三部分都是直接引用陈皓的[跟我一起写Makefile](https://seisman.github.io/how-to-write-makefile/index.html)，写的已经很完整了，就不再添加我自己的理解和例子了。

#### 在makefile引用其他的makefile

在Makefile使用 `include` 关键字可以把别的Makefile包含进来，这很像C语言的 `#include` ，被包含的文件会原模原样的放在当前文件的包含位置。 `include` 的语法是：

```
include <filename>
```

`filename` 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）。

在 `include` 前面可以有一些空字符，但是绝不能是 `Tab` 键开始。 `include` 和 `<filename>` 可以用一个或多个空格隔开。举个例子，你有这样几个Makefile： `a.mk` 、 `b.mk` 、 `c.mk` ，还有一个文件叫 `foo.make` ，以及一个变量 `$(bar)` ，其包含了 `e.mk` 和 `f.mk` ，那么，下面的语句：

```
include foo.make *.mk $(bar)
```

等价于：

```
include foo.make a.mk b.mk c.mk e.mk f.mk
```

make命令开始时，会找寻 `include` 所指出的其它Makefile，并把其内容安置在当前的位置。就好像C/C++的 `#include` 指令一样。如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找：

1. 如果make执行时，有 `-I` 或 `--include-dir` 参数，那么make就会在这个参数所指定的目录下去寻找。
2. 如果目录 `<prefix>/include` （一般是： `/usr/local/bin` 或 `/usr/include` ）存在的话，make也会去找。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”。如：

```
-include <filename>
```

其表示，无论include过程中出现什么错误，都不要报错继续执行。和其它版本make兼容的相关命令是sinclude，其作用和这一个是一样的。

#### Makefile里有什么？

Makefile里主要包含了五个东西：显式规则、隐晦规则、变量定义、文件指示和注释。

1. 显式规则。显式规则说明了如何生成一个或多个目标文件。这是由Makefile的书写者明显指出要生成的文件、文件的依赖文件和生成的命令。
2. 隐晦规则。由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较简略地书写 Makefile，这是由make所支持的。
3. 变量的定义。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
4. 文件指示。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。
5. 注释。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用 `#` 字符，这个就像C/C++中的 `//` 一样。如果你要在你的Makefile中使用 `#` 字符，可以用反斜杠进行转义，如： `\#` 。

最后，还值得一提的是，在Makefile中的命令，必须要以 `Tab` 键开始。

#### make的工作方式

GNU的make工作时的执行步骤如下：（想来其它的make也是类似）

1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐晦规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

1-5步为第一个阶段，6-7为第二个阶段。第一个阶段中，如果定义的变量被使用了，那么，make会把其展开在使用的位置。但make并不会完全马上展开，make使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开。

参考文章：

## 三.makefile的书写规则

### 通配符

makefile中是可以使用通配符来进行匹配的，前面的clean中的*就是一个通配符，一般来说有如下几个通配符：

| 通配符 | 使用说明                                              |
| ------ | ----------------------------------------------------- |
| %      | 匹配0个或任意个字符                                   |
| *      | 匹配0个或者是任意个字符                               |
| ?      | 匹配任意一个字符                                      |
| []     | 我们可以指定匹配的字符放在 “[]” 中                    |
| ~      | `~/test` ，这就表示当前用户的 `$HOME`目录下的test目录 |

例如：

```makefile
main : main.o
	@g++ main.o -o main

%.o : %.cpp
	@g++ -c $^ -o $@

.PHONY : clean
clean :
	-rm main *.o
```

上面makefile的make执行过程如下：

第一个（最终的）目标文件为main，依赖文件为main.o，发现没有，于是往下找，找到%.o，匹配为main.o，于是找相应的依赖文件main.cpp，进而执行命令完成编译。

这里其实我还有一个疑问，即%.cpp是如何与main.cpp匹配上的，我猜想是前面的%与后面的%表示的应该是一样的，因为后面的换成*.cpp就会报错。

通配符使用在依赖中的话一定要注意这个问题：不能通过引用变量的方式来使用，例如：

```makefile
OBJ = *.cpp
test : $(OBJ)
    g++ -o $@ $^
```

这时OBJ作为一个变量，其值就为*.cpp，而且它在后面的使用中是不会展开的。所以最好不要这么写。

// 未完待续

参考：[Makefile文件编写(四)☞通配符](https://seisman.github.io/how-to-write-makefile/rules.html)

### 文件搜寻

如果没有指定文件搜寻路径，那么我们在makefile中用到的所有目录都是在当前文件目录（即makefile文件所在的目录）进行查找，但是显然我们会在当makefile所在目录下创建多个文件夹，这个时候就需要在makefile文件中指定相应文件的目录了，例如我们把头文件num_max.h放在headers文件夹下：

![image-20220701092338645](C:\Users\q1369\AppData\Roaming\Typora\typora-user-images\image-20220701092338645.png)

这个时候再make就会报错。需要指定num_max搜寻的路径。

在makefile中指定搜寻路径有两种方法，

1. 利用特殊变量VPATH，设定VPATH的值设置路径，例如VPATH = src : ../headers，即文件先在当前目录找（当前目录的优先级始终最高），再在src中找，再在../headers（ . 是指当前目录，.. 是指上一级目录）中找，用冒号:分隔。这里的headers是当前目录下的，所以makefile可以这样写：

   ```makefile
   VPATH = ./headers
   
   main : main.o
   	@g++ main.o  -o main
   
   main.o : num_max.h
   	@g++  -c main.cpp -o main.o
   
   .PHONY : clean
   clean : 
   	-rm *.o main
   ```

2. 利用关键字vpath（注意上面那个是特殊变量全大写，这个是关键字全小写），这个关键字更加灵活一点：

   ```
   vpath <pattern> <directories>
   ```

   为符合模式<pattern>的文件指定搜索目录<directories>。

   ```
   vpath <pattern>
   ```

   清除符合模式<pattern>的文件的搜索目录。

   ```
   vpath
   ```

   清除所有已被设置好了的文件搜索目录。

   我们可以给 .h 文件设置目录headers，例如：

   ```makefile
   vpath %.h ./headers
   
   main : main.o
   	@g++ main.o  -o main
   
   main.o : num_max.h
   	@g++  -c main.cpp -o main.o
   
   .PHONY : clean
   clean : 
   	-rm *.o main
   ```

   如果连续指定多个目录，如vpath %.h ./headers     vpath %.h ./src   ，则先在当前目录找（当前目录始终优先级最高），再在headers中找，再在src中找。

### 多目标

makefile可以一次性生成多个目标文件，我觉得算是有两种方法。

1. 将要生成的目标文件作为伪目标的依赖文件，然后将伪目标放在makefile文件首部使其成为最终目标，此时make即可生成多个文件，例如：

   ```makefile
   vpath %.h ./headers
   
   .PHONY : all
   all : helloworld main
   
   helloworld : helloworld.cpp
   	@g++ helloworld.cpp -o helloworld
   
   main : main.o
   	@g++ main.o  -o main
   
   main.o : num_max.h
   	@g++  -c main.cpp -o main.o
   
   .PHONY : clean
   clean : 
   	-rm *.o main
   ```

2. 第二种方法才是真正的多目标方法，把要生成的目标文件放在基本单元的目标区，然后把需要的依赖文件放在右边的依赖区，命令中需要使用特殊变量$@（前面也说过，$@表示目标文件的集合），例如：

   ```
   
   ```

   // 未完待续

### 静态模式

静态模式用来更加方便的指定多目标的规则，其语法为：

```makefile
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
```

接下来的话引自：[静态模式](https://seisman.github.io/how-to-write-makefile/rules.html#id8)

targets定义了一系列的目标文件，可以有通配符。是目标的一个集合。

target-pattern是指明了targets的模式，也就是的目标集模式。

prereq-patterns是目标的依赖模式，它对target-pattern形成的模式再进行一次依赖目标的定义。

这样描述这三个东西，可能还是没有说清楚，还是举个例子来说明一下吧。如果我们的<target-pattern>定义成 `%.o` ，意思是我们的<target>;集合中都是以 `.o` 结尾的，而如果我们的<prereq-patterns>定义成 `%.c` ，意思是对<target-pattern>所形成的目标集进行二次定义，其计算方法是，取<target-pattern>模式中的 `%` （也就是去掉了 `.o` 这个结尾），并为其加上 `.c` 这个结尾，形成了依赖文件集合。

所以，我们的“目标模式”或是“依赖模式”中都应该有 `%` 这个字符，如果你的文件名中有 `%` 那么你可以使用反斜杠 `\` 进行转义，来标明真实的 `%` 字符。

例如：

// 未完待续

### 自动生成依赖性

我们知道一个项目中某些个.cpp文件可能有多个头文件，如果在写的时候手动添加到依赖关系会有点麻烦，而且如果修改的话更不方便，所以大多数编译器都支持一个自动寻找头文件的功能，使用这个功能的语法为：

```
g++ -M main.cpp
# 如果是GNU的编译器，-M为-MM，否则会把标准库的头文件加进来
```

会生成

```
main.o : main.cpp num_max.h
```

至于这个怎么跟makefile联系起来。。。// 未完待续。

## 四.命令执行

命令执行在第二部分中已经说了一部分了，这里再补充一些内容吧。

### make的一些命令参数

如果make执行时，带入make参数 `-n` 或 `--just-print` ，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。

而make参数 `-s` 或 `--silent` 或 `--quiet` 则是全面禁止命令的显示。

给make加上 `-i` 或是 `--ignore-errors` 参数，那么，Makefile中所有命令都会忽略错误。而如果一个规则是以 `.IGNORE` 作为目标的，那么这个规则中的所有命令将会忽略错误。这些是不同级别的防止命令出错的方法，你可以根据你的不同喜欢设置。

make的参数的是 `-k` 或是 `--keep-going` ，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则（也可以在基本单元的命令行前加一个-表示不管该命令出不出错都表示正确）。

### 下条命令使用上条命令的结果

一般来说上条命令的结果不会影响下条命令，如：

```makefile
vpath %.h ./headers


main : main.o
	@g++ main.o  -o main

main.o : num_max.h
	@g++  -c main.cpp -o main.o

.PHONY : clean
clean : 
	-rm *.o main helloworld

.PHONY : exec
exec : 
	@cd ./headers
	@pwd
```

执行命令make exec，这时pwd打印出的路径仍是makefile所在目录，没有受到上条cd命令的影响，如果要让其受到cd结果影响，需要将其写在cd命令的后面（即在同一行）并且用分号分隔。

```makefile
vpath %.h ./headers


main : main.o
	@g++ main.o  -o main

main.o : num_max.h
	@g++  -c main.cpp -o main.o

.PHONY : clean
clean : 
	-rm *.o main helloworld

.PHONY : exec
exec : 
	@cd ./headers ; pwd
# 其实也可以用换行符来写，例如
exec : 
	@cd ./headers ;\
	pwd
```

### 嵌套执行make

此段内容全部引自：[嵌套执行make](https://seisman.github.io/how-to-write-makefile/recipes.html#make)

在一些大的工程中，我们会把我们不同模块或是不同功能的源文件放在不同的目录中，我们可以在每个目录中都书写一个该目录的Makefile，这有利于让我们的Makefile变得更加地简洁，而不至于把所有的东西全部写在一个Makefile中，这样会很难维护我们的Makefile，这个技术对于我们模块编译和分段编译有着非常大的好处。

例如，我们有一个子目录叫subdir，这个目录下有个Makefile文件，来指明了这个目录下文件的编译规则。那么我们总控的Makefile可以这样书写：

```
subsystem:
    cd subdir && $(MAKE)
```

其等价于：

```
subsystem:
    $(MAKE) -C subdir
```

定义$(MAKE)宏变量的意思是，也许我们的make需要一些参数，所以定义成一个变量比较利于维护。这两个例子的意思都是先进入“subdir”目录，然后执行make命令。

我们把这个Makefile叫做“总控Makefile”，总控Makefile的变量可以传递到下级的Makefile中（如果你显示的声明），但是不会覆盖下层的Makefile中所定义的变量，除非指定了 `-e` 参数。

如果你要传递变量到下级Makefile中，那么你可以使用这样的声明:

```makefile
export <variable ...>;
```

如果你不想让某些变量传递到下级Makefile中，那么你可以这样声明:

```makefile
unexport <variable ...>;
```

如：

示例一：

```makefile
export variable = value
```

其等价于：

```makefile
variable = value
export variable
```

其等价于：

```
export variable := value
```

其等价于：

```makefile
variable := value
export variable
```

示例二：

```makefile
export variable += value
```

其等价于：

```makefile
variable += value
export variable
```

如果你要传递所有的变量，那么，只要一个export就行了。后面什么也不用跟，表示传递所有的变量。

需要注意的是，有两个变量，一个是 `SHELL` ，一个是 `MAKEFLAGS` ，这两个变量不管你是否export，其总是要传递到下层 Makefile中，特别是 `MAKEFLAGS` 变量，其中包含了make的参数信息，如果我们执行“总控Makefile”时有make参数或是在上层 Makefile中定义了这个变量，那么 `MAKEFLAGS` 变量将会是这些参数，并会传递到下层Makefile中，这是一个系统级的环境变量。

但是make命令中的有几个参数并不往下传递，它们是 `-C` , `-f` , `-h`, `-o` 和 `-W` （有关Makefile参数的细节将在后面说明），如果你不想往下层传递参数，那么，你可以这样来：

```makefile
subsystem:
    cd subdir && $(MAKE) MAKEFLAGS=
```

如果你定义了环境变量 `MAKEFLAGS` ，那么你得确信其中的选项是大家都会用到的，如果其中有 `-t` , `-n` 和 `-q` 参数，那么将会有让你意想不到的结果，或许会让你异常地恐慌。

还有一个在“嵌套执行”中比较有用的参数， `-w` 或是 `--print-directory` 会在make的过程中输出一些信息，让你看到目前的工作目录。比如，如果我们的下级make目录是“/home/hchen/gnu/make”，如果我们使用 `make -w` 来执行，那么当进入该目录时，我们会看到:

```
make: Entering directory `/home/hchen/gnu/make'.
```

而在完成下层make后离开目录时，我们会看到:

```
make: Leaving directory `/home/hchen/gnu/make'
```

当你使用 `-C` 参数来指定make下层Makefile时， `-w` 会被自动打开的。如果参数中有 `-s` （ `--slient` ）或是 `--no-print-directory` ，那么， `-w` 总是失效的。

### 定义命令包

有些命令可能需要重复执行，我们可以定义变量来代表命令，语法为：

```makefile
define 变量名
命令
命令
...
endef
```

例如：

```makefile
vpath %.h ./headers

define run-main
@g++ main.o -o main
endef


main : main.o
	$(run-main)

main.o : num_max.h
	@g++  -c main.cpp -o main.o

.PHONY : clean
clean : 
	-rm *.o main helloworld

.PHONY : exec
exec : 
	@cd ./headers ; pwd
```

## 五.使用变量

makefile中的变量代表一个文本字符串，在使用时用$(变量名)标识，makefile会将变量代表的字符串原样的代替出现在相应位置。

### 变量的定义

变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有 `:` 、 `#` 、 `=` 或是空字符（空格、回车等）（不应该不代表不能含有，代表可以含有但最好别含有，之后会展示怎么定义值为空格的变量）。变量是大小写敏感的，“foo”、“Foo”和“FOO”是三个不同的变量名。传统的Makefile的变量名是全大写的命名方式，但推荐使用大小写搭配的变量名，如：MakeFlags。这样可以避免和系统的变量冲突，而发生意外的事情。

怎么定义一个空格呢，如下：

```makefile
nullstring :=
space := $(nullstring) # end of the line 
```

nullstring是空字符串，而space是空字符串加上其后的一个空格，为什么其后的一个空格要算呢，因为#注释符表示这一行已经结束，字符串的初始到#前的内容都算是变量的内容，所以一定要警惕#的这种特性。例如：

```makefile
dir := /foo/bar    # directory to put the frobs in
```

（此段内容参考[使用变量](https://seisman.github.io/how-to-write-makefile/variables.html#id3)）dir这个变量的值是“/foo/bar”，后面还跟了4个空格，如果我们这样使用这样变量来指定别的目录——“$(dir)/file”那么就完蛋了。

还有一个比较有用的操作符是 `?=` ，先看示例：

```makefile
FOO ?= bar
```

其含义是，如果FOO没有被定义过，那么变量FOO的值就是“bar”，如果FOO先前被定义过，那么这条语将什么也不做

### 变量值的替换

我们可以替换变量中的共有的部分，其格式是 `$(var:a=b)` 或是 `${var:a=b}` ，其意思是，把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。这里的“结尾”意思是“空格”或是“结束符”。

还是看一个示例吧：

```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

这个示例中，我们先定义了一个 `$(foo)` 变量，而第二行的意思是把 `$(foo)` 中所有以 `.o` 字串“结尾”全部替换成 `.c` ，所以我们的 `$(bar)` 的值就是“a.c b.c c.c”。

另外一种变量替换的技术是以“静态模式”（参见前面章节）定义的，如：

```makefile
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```

这依赖于被替换字串中的有相同的模式，模式中必须包含一个 `%` 字符，这个例子同样让 `$(bar)` 变量的值为“a.c b.c c.c”。

### 追加变量值

引自[追加变量值](https://seisman.github.io/how-to-write-makefile/variables.html#id5)

我们可以使用 `+=` 操作符给变量追加值，如：

```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```

于是，我们的 `$(objects)` 值变成：“main.o foo.o bar.o utils.o another.o”（another.o被追加进去了）

如果变量之前没有定义过，那么， `+=` 会自动变成 `=` ，如果前面有变量定义，那么 `+=` 会继承于前次操作的赋值符。如果前一次的是 `:=` ，那么 `+=` 会以 `:=` 作为其赋值符，如：

```
variable := value
variable += more
```

等价于：

```
variable := value
variable := $(variable) more
```

但如果是这种情况：

```makefile
variable = value
variable += more
```

由于前次的赋值符是 `=` ，所以 `+=` 也会以 `=` 来做为赋值，那么岂不会发生变量的递补归定义，这是很不好的，所以make会自动为我们解决这个问题，我们不必担心这个问题。

### override 指示符

引自[override提示符](https://seisman.github.io/how-to-write-makefile/variables.html#override)

如果有变量是通常make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。如果你想在Makefile中设置这类参数的值，那么，你可以使用“override”指示符。其语法是:

```makefile
override <variable>; = <value>;

override <variable>; := <value>;
```

当然，你还可以追加:

```makefile
override <variable>; += <more text>;
```

对于多行的变量定义，我们用define指示符，在define指示符前，也同样可以使用override指示符，如:

```makefile
override define foo
bar
endef
```

### 使用def定义变量

大致引自[多行变量](https://seisman.github.io/how-to-write-makefile/variables.html#id6)

还有一种设置变量值的方法是使用define关键字。使用define关键字设置变量的值可以有换行，这有利于定义一系列的命令（前面我们讲过“命令包”的技术就是利用这个关键字）。

define指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以endef 关键字结束。其工作方式和“=”操作符一样。变量的值可以包含函数、命令、文字，或是其它变量。因为命令需要以[Tab]键开头，所以如果你用define定义的命令变量中没有以 `Tab` 键开头，那么make 就不会把其认为是命令（即你使用该变量的时候若前面没有tab键则make认为其实变量）。

下面的这个示例展示了define的用法:

```makefile
define two-lines
echo foo
echo $(bar)
endef
```

### 环境变量

引自[环境变量](https://seisman.github.io/how-to-write-makefile/variables.html#id7)

make运行时的系统环境变量可以在make开始运行时被载入到Makefile文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了“-e”参数，那么，系统环境变量将覆盖Makefile中定义的变量）

因此，如果我们在环境变量中设置了 `CFLAGS` 环境变量，那么我们就可以在所有的Makefile中使用这个变量了。这对于我们使用统一的编译参数有比较大的好处。如果Makefile中定义了CFLAGS，那么则会使用Makefile中的这个变量，如果没有定义则使用系统环境变量的值，一个共性和个性的统一，很像“全局变量”和“局部变量”的特性。

当make嵌套调用时（参见前面的“嵌套调用”章节），上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile 中。当然，默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层Makefile传递，则需要使用export关键字来声明。（参见前面章节）

当然，我并不推荐把许多的变量都定义在系统环境中，这样，在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。

### 目标变量

截止到目前我们设置的变量都是全局变量，即在整个makefile中都可以用的变量，其实也可以设置局部变量，这样的变量一般叫目标变量，因为其作用的范围是对应目标及由此目标引发的基本单元，其语法为：

```
<target ...> : <variable-assignment>;

<target ...> : overide <variable-assignment>
```

<variable-assignment>;可以是前面讲过的各种赋值表达式，如 `=` 、 `:=` 、 `+=` 或是 `?=` 。第二个语法是针对于make命令行带入的变量，或是系统环境变量。

上面说的有点抽象，举个例子：

```makefile
vpath %.h ./headers

main : ++ = @g++
# 这里定义了一个目标变量，目标为main，即其作用范围为以main为目标的基本单元及由此引发的基本单元
main : main.o
	$(++) main.o -o main
	
main.o : num_max.h
	$(++) -c main.cpp -o main.o

.PHONY : clean
clean : 
	-rm *.o main helloworld

.PHONY : exec
exec : 
	@cd ./headers ; pwd
```

### 模式变量

引自[模式变量](https://seisman.github.io/how-to-write-makefile/variables.html#id9)

在GNU的make中，还支持模式变量（Pattern-specific Variable），通过上面的目标变量中，我们知道，变量可以定义在某个目标上。模式变量的好处就是，我们可以给定一种“模式”，可以把变量定义在符合这种模式的所有目标上。（其实计算机里面这么干是很平常的事，例如数据库中的角色之类的）

我们知道，make的“模式”一般是至少含有一个 `%` 的，所以，我们可以以如下方式给所有以 `.o` 结尾的目标定义目标变量：

```makefile
%.o : CFLAGS = -O
```

同样，模式变量的语法和“目标变量”一样：

```
<pattern ...>; : <variable-assignment>;

<pattern ...>; : override <variable-assignment>;
```

override同样是针对于系统环境传入的变量，或是make命令行指定的变量。

## 六.条件判断

这个陈皓的文章已经说的很清楚了，这里给一个链接即可：

[使用条件判断](https://seisman.github.io/how-to-write-makefile/conditionals.html#id1)

如果上面那个连接失效的可以用下面这个

[使用条件判断](https://wiki.ubuntu.com.cn/跟我一起写Makefile:使用条件判断)
