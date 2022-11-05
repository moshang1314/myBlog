# 1 系统调用简介和实现

## 1.1 什么是系统调用

系统调用，顾名思义，说的是操作系统提供给用户程序调用的一组“特殊”接口。用户程序可以通过这组“特殊”接口来获得操作系统内核提供的服务，比如用户可以通过文件系统相关的调用请求系统打开文件、关闭文件或读写文件，可以通过时钟相关的系统调用获得系统时间或设置定时器等。

从逻辑上来说，系统调用可被看成一个内核与用户空间程序交互的接口——它好比一个中间人，把用户进程的请求传达给内核，待内核把请求处理完毕后再将处理结果送回到用户空间。

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527649958892.png)

系统服务之所以需要通过系统调用来提供给用户空间的根本原因是为了对系统进行“保护”，因为我们知道Linux的运行空间分为内核空间与用户空间，它们各自运行在不同的级别中，逻辑上相互隔离。

所以用户进程在通常情况下不允许访问内核数据，也无法使用内核函数，它们只能在用户空间操作用户数据，调用用户空间函数。

比如我们熟悉的“hello world”程序（执行时）就是标准的用户空间进程，它使用的是打印函数printf就书友用户空间函数，打印的字符“hello word”字符串也属于用户空间数据。

但是很多情况下，用户进程需要获得系统服务（调用系统程序），这时就必须利用系统提供给用户的“特殊接口”——系统调用了，它的特殊性主要在于规定了用户进程进入内核的具体位置。

换句话说，用户访问内核的路径是事先规定好的，只能从规定位置进入内核，而不准许肆意跳入内核。有了这样的进入内核的统一访问路径限制才能保证内核安全无误。我们可以形象地描述这种机制：：作为一个游客，你可以买票要求进入野生动物园，但你必须老老实实地坐在观光车上，按照规定的路线观光游览。当然，不准下车，因为那样太危险，不是让你丢掉小命，就是让你吓坏了野生动物。

## 1.2 系统调用的实现

系统调用属于操作系统内核的一部分，必须以某种方式提供给进程让它们去调用。CPU可以在不同的特权级别下运行，而相应的的操作系统也有不同的运行级别，用户态和内核态。运行在内核态的进程可以毫无限制的访问各种资源，而在用户态的用户进程的各种操作都有限制，比如不能随意的访问内存，不能开闭中断以及切换运行的特权级别。显然，属于内核的系统调用一定是运行在内核态下，但是如何切换到内核态呢？

答案是软中断。软中断和我们常说的中断（硬件中断）不同之处在于，它是通过软件指令触发而并非外设引发的中断，也就是说，又是编程人员开发出的一种异常（该异常为正常的异常）。**操作系统一般是通过软件中断从用户态切换到内核态。**

## 1.3 中断

中断有两种，一种是CPU本身在执行程序的过程中产生的，一种是由CPU外部产生的。
外部中断，就是通常所讲的“中断”（interrupt）。对于执行程序来说，这种“中
断”的发生完全是异步的，因为不知道什么时候会发生。CPU对其的响应也完全是被动的，
可以通过“关中断”指令关闭对其的响应。
由软件产生的中断一般是由专设的指令，如X86中的“INT n”在程序中有意产生的，
是主动的，同步的。只要CPU执行一条INT指令，在开始执行下一条指令之前一定会进入中
断服务程序。这种主动的中断称为“陷阱”（trap）。 

 

 

​    Linux 内核需要对连接到计算机上的所有硬件设备进行管理，毫无疑问这是它的份内事。如果要管理这些设备，首先得和它们互相通信才行，一般有两种方案可实现这种功能：
1.轮询（polling） 让内核定期对设备的状态进行查询，然后做出相应的处理；
2.[中断](https://so.csdn.net/so/search?q=中断&spm=1001.2101.3001.7020)（interrupt） 让硬件在需要的时候向内核发出信号（变内核主动为硬件主动）。
​    第一种方案会让内核做不少的无用功，因为轮询总会周期性的重复执行，大量地耗用 CPU 时间，因此效率及其低下，所以一般都是采用第二种方案。

**什么是中断?**

​    从物理学的角度看，中断是一种电信号，由硬件设备产生，并直接送入中断控制器（如 8259A）的输入引脚上，然后再由中断控制器向处理器发送相应的信号。处理器一经检测到该信号，便中断自己当前正在处理的工作，转而去处理中断。此后，处理器会通知 OS 已经产生中断。这样，OS 就可以对这个中断进行适当的处理。
​    不同的设备对应的中断不同，而每个中断都通过一个唯一的数字标识，这些值通常被称为中断请求线(IRQ)。

中断可分为同步（synchronous）中断和异步（asynchronous）中断：
\1. 同步中断是当指令执行时由 CPU 控制单元产生，之所以称为同步，是因为只有在一条指令执行完毕后 CPU 才会发出中断，而不是发生在代码指令执行期间，比如系统调用。
\2. 异步中断是指由其他硬件设备依照 CPU 时钟信号随机产生，即意味着中断能够在指令之间发生，例如键盘中断。

**什么是异常?**

同步中断又称为异常（exception），异步中断则被称为中断（interrupt）。我们通常讲的中断指的都是异步中断。
1.中断可分为可屏蔽中断（Maskable interrupt）和非屏蔽中断（Nomaskable interrupt）。
2.异常可分为故障（fault）、陷阱（trap）、终止（abort）三类。

这些类别之间的异同点请参看 表 1。

表 1：中断类别及其行为

| 类别 |       原因        | 异步/同步 |       返回行为       |
| :--: | :---------------: | :-------: | :------------------: |
| 中断 | 来自I/O设备的信号 |   异步    | 总是返回到下一条指令 |
| 陷阱 |    有意的异常     |   同步    | 总是返回到下一条指令 |
| 故障 | 潜在可恢复的错误  |   同步    |    返回到当前指令    |
| 终止 |  不可恢复的错误   |   同步    |       不会返回       |

**什么是中断处理程序?**

在响应一个特定中断的时候，内核会执行一个函数，该函数叫做中断处理程序或中断服务例程。产生中断的每个设备都有一个相应的中断处理程序，如果一个设备可以产生多种不同的中断，那么该设备就可以对应多个中断处理程序。一个设备的中断处理程序是它设备驱动程序的一部分。

# 2 系统调用和库函数的区别

Linux下对文件操作有两种方式：系统调用（system call）和库函数调用（Library functions）。

库函数由两类函数组成：

1）不需要调用系统调用

不需要切换到内核空间即可完成函数全部功能，并且将结果反馈给应用程序，如strcpy、bzero等字符串操作函数。

2）需要调用系统调用

需要切换到内核空间，这类函数通过封装系统调用去实现相应的功能，如printf、fread等。

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527650321383.png)

系统调用时需要时间的，程序中频繁的使用系统调用会降低程序的运行效率。当运行内核代码时，CPU工作在内核态，在系统调用发生前需要保存用户态的栈和内存环境，然后转入内核态工作。系统调用结束后，又要切换回用户态。这种环境的切换回消耗掉许多时间。

# 3 C库中IO函数工作流程

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527650554264.png)

库函数访问文件的时候根据需要，设置不同类型的缓冲区，从而减少了直接调用IO系统调用的次数，提高了访问效率。

这个过程类似于快递员给某个区域（内核空间）送快递一样，快递员有两种方式送：

1）来一件快递就马上送到目的地，来一件送一件，这样导致来回走比较频繁（系统调用）

2）等快递攒着差不多后（缓冲区），才一次性送到目的地（库函数调用）

# 4 错误处理函数

errno是记录系统的最后一次错误代码。代码是一个int型的值，在errno.h中定义。查看错误代码errno是调试程序的一个重要方法。

当Linux C api函数发生异常时，一般会将errno全局变量赋一个整数值，不同的值表示不同的含义，可以通过查看该值推测出错的原因。

测试程序：

```c
#include <stdio.h>	//fopen perror
#include <errno.h>	//errno
#include <string.h>	//strerror(errno)
//errno是一个全局变量，在errno.h头文件中有定义
int main(void)
{
	FILE *fp = fopen("xxxx", "r");
	if(NULL == fp)
	{
		printf("%d\d", errno);	//打印错误码
		printf("%s\n", strerror(errno)); //把errno的数字转换成相应的文字
        //根据errno的值输出错误信息
        //提示字符串：出错原因
		perror("fopen err");	//打印错误原因的字符串
	}
	return 0;
}
```

查看错误号：

> /usr/include/asm-generic/errno-base.h
>
> /usr/include/asm-generic/errno.h

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527755557333.png)

# 5 虚拟地址空间

每个进程都会分配虚拟地址空间，在32位机器上，该地址空间为4G。

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527650975663.png)

在进程里平时所说的指针变量，保存的就是虚拟地址。当应用程序使用虚拟地址访问内存时，处理器（CPU）会将其转化为物理地址（MMU）。

MMU：将虚拟的地址转化为物理地址。

这样做的好处在于：

* 进程隔离，更好的保护系统安全运行
* 屏蔽物理差异带来的麻烦，方便操作系统和编译器安排进程地址

# 6 文件描述符

## 6.1 文件描述符详解

Linux系统内维护了三个系统文件表：文件描述符表（file descriptor table），打开文件表（open file table），inode table。这三个表的结构见Table-1。

在like-unix系统中的所有的IO操作（包括socket等）都是基于文件描述符的。程序刚刚启动的时候已经默认帮我们分配了三个文件描述符，就是我们常用的0标准输入，1标准输出，2标准错误。如果此时进程再打开新的文件，它的文件描述符会加1也就是3。POSIX标准要求每次打开文件时（包含socket）必须使用当前进程中最小可用的文件描述符号。

<h3> 文件描述符表(file descriptor table)</h3>

每一个进程维护了一份自己的文件描述符表，该表中的每条记录维护了该条文件描述符的相关信息，包括：

* 控制标志（fd flags)，截至目前，内核仅定义了一个flag，close-on-exec
* open file ptr：指向open file table的指针

<h3>打开文件表（open file table）</h3>

内核内维护的一个系统范围的打开文件表。该表中包含每一个打开的文件条目，文件表项跟踪所有对文件的读或写操作，当前偏移量和文件的打开模式（O_RDONLY，O_WRONLY，or O_RDWR）。

该表中的每条记录维护了与打开的文件相关的全部信息，包括：

* 文件偏移量（file offset）：调用read()或者write()操作更新，调用fseek()直接修改，流类型的文件（比如pipes和sockets）不能使用偏移量，因为文件中的数据不是随机访问的。
* 访问模式（status flags）：只读，只写或读写等。
* inode指针：指向inode表元素的指针，关联物理文件
* 引用计数：比如一条记录被2个文件描述符引用，计数就为2

<h3>inode表</h3>

在linux系统中使用inode号描述文件，就像进程使用pid描述符进程一样。inode存储了文件的元信息。系统上的所有文件都分配了一个inode记录。

每个inode记录包含以下信息（可以使用stat命令查看）：

```shell
$ stat a
  File: a
  Size: 4         	Blocks: 8          IO Block: 4096   regular file
Device: 805h/2053d	Inode: 1446030     Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/zhangboqi)   Gid: ( 1000/zhangboqi)
Access: 2021-09-29 14:35:32.344894535 +0800
Modify: 2021-09-29 14:35:32.344894535 +0800
Change: 2021-09-29 14:35:32.344894535 +0800
 Birth: -
```

* 文件的字节数（Size）
* Blocks：文件占用块大小，单位512字节，就是文件实际占用大小
* IO Blocks：文件系统每次IO操作的最小大小，extr默认是4096字节
* 文件类型：如常规文件（regular file），目录，套接字或FIFO等
* 设备号（Device）：设备号
* inode号：文件对应的inode号
* 硬链接数（Links）：即由多少个文件指向这个inode
* 文件权限（Access）
* 文件uid和gid
* 文件的时间戳：Access为文件上一次打开的时间，Modify为文件上一次修改的时间，Change为inode上一次变动的时间

注：inode存储在磁盘设备上，但内核会在内存中维护一份。这里的inode table就是指内存中的副本。**inode table中的记录同样维护了一个引用计数，该记录每被open file记录引用一次就加1。**

<h3>
  文件描述符数量限制  
</h3>
进程可用的文件描述符的数量由sys/limits.h文件中的/OPEN_MAX控制。文件描述符的数量也可由ulimit -n控制。可以分配给进程的文件描述符的数量由资源限制控制。默认值是在/etc/security/limits文件中设置的，通常设置为2000。可以通过ulimit命令或setrlimit子例程更改该限制。最大大小由常量OPEN_MAX定义。

<h3>文件描述符的创建</h3>
open, pipe, create和fcntl子程序都可以创建文件描述符。文件描述符在每个进程里都是唯一的，但是他们可以有fork子程序创建的子进程共享。也可以由fcntl，dup和dup2子程序复制。

文件描述符是一个存储在内核为每个进程维护的u_block区域中的文件描述符表的索引。进程获取文件描述符的最常见方法是通过open或create操作，或者从父进程继承。当一个fork操作发生时。系统为子进程复制父进程的文件描述符表，它允许子进程平等地访问父进程使用的文件。

<h3>管理文件描述符</h3>
因为文件是可以同时被多个用户共享的，所以有必要允许相关进程共享一个公共偏移指针，并且访问同一文件的独立进程拥有一个单独的当前偏移指针，open file table 条目维护了一个引用计数，用来跟踪分配给文件的文件描述符的数量。

对单个文件的多个引用可能是由下列任何一种情况引起的:

* 打开文件的单独进程
* 子进程保留分配给父进程的文件描述符
* fcntl或dup子例程创建一个文件描述符的副本

<h3>共享打开文件</h3>

**每个打开操作创建一个系统打开文件表项。**单独的表项确保每个进程有单独的当前I/O偏移量。独立偏移量可以保护数据的完整性。当一个文件描述符被复制时，两个进程将共享相同的偏移量，并可能发生交错，在这种情况下，字节不是按顺序读或写的。

<h3>复制文件描述符</h3>

文件描述符可以通过以下方式在进程之间复制:**dup**或**dup2**子例程、**fork**子例程和**fcntl**(文件描述符控制)子例程。

**dup和dup2子程序**
dup子例程创建一个文件描述符的副本。副本是在包含原始描述符的用户文件描述符表中的空白位置创建的。dup进程将open file table项中的引用计数加1，并返回副本所在的文件描述符的索引号。

**fork子程序**
fork子例程创建一个子进程，子进程继承分配给父进程的文件描述符。然后子进程执行一个新进程。当fcntl子程序关闭时继承的文件描述符会设置close-on-exec标志

**FCNTL(文件描述符控制)子程序**
fcntl子例程操作文件结构并控制打开的文件描述符。它可以用来对描述符进行以下更改:

* 复制一个文件描述符(与dup子例程相同)。
* 获得或设置“close-on-exec ”标志。
* 为描述符设置非阻塞模式。
* Append future writes to the end of the file(O_APPEND).
* 当可以进行I/O时，允许向进程生成信号。
* Set or get the process ID or the group process ID for SIGIO handling.
* 关闭所有文件描述符。

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5byg5Y2a5aWH,size_20,color_FFFFFF,t_70,g_se,x_16)

在Linux的世界里，一切设备皆文件。我们可以使用系统调用中的I/O函数（I：Input，输入；O：output，输出），对文件进行相应的操作（open()、close()、write()、read()等）。

打开现存文件或新建文件时，系统（内核）会返回一个文件描述符，文件描述符用来指定已打开的文件。这个文件描述符相当于这个已打开的文件的标号，文件描述符是非负整数，是文件的标识，操作这个文件描述符相当于操作这个描述符所指的文件。

程序运行起来后（每个进程）都有一张文件描述符的表，标准输入、标准输出、标准错误输出设备文件被打开，对应的文件描述符0、1、2记录在表中。程序运行起来后这三个文件描述符是默认打开的。

```C
#define STDIN_FILENO  0 //标准输入的文件描述符
#define STDOUT_FILENO 1 //标准输出的文件描述符
#define STDERR_FILENO 2 //标准错误的文件描述符
```

在程序运行起来后打开其他文件时，系统会返回文件描述符表中最小可用的文件描述符，并将此文件描述符记录在表中。

![](https://cdn.jsdelivr.net/gh/moshang1314/myBlog@main/image/1527651181126.png)

<h3>最大打开的文件个数</h3>

Linux中一共进程最多只能打开NR_OPEN_DEFAULT（即1024）个文件，故当文件不再使用时应及时调用close()函数关闭文件。

* 查看当前系统允许打开最大文件个数：

  * > cat /proc/sys/fs/file-max

* 当前默认设置最大打开文件个数1024

  * > ulimit -a

* 修改默认设置最大打开文件个数为4096

  * > ulimit -n 4096

# 7 常用文件IO函数

## 7.1 open函数

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags);
int open(const char *pathame, int flags, mode_t mode);

/*
功能：
	打开文件，如果文件不存在则可以选择创建
参数：
	pathname：文件的路径及文件名
	flags：打开文件的行为标志，必选项O_RDONLY，O_WRONLY，O_RDWR
	mode：这个参数，只有在文件不存在时有效，指新建文件时指定文件的权限
返回值：
	成功：成功返回打开的文件描述符
	失败：-1
*/
```

**flags详细说明**

必选项：

|   取值   |          含义          |
| :------: | :--------------------: |
| O_RDONLY |    以只读的方式打开    |
| O_WRONLY |    以只写的方式打开    |
|  O_RDWR  | 以可读、可写的方式打开 |

可选项，和必选项按位或起来

|    取值    |                             含义                             |
| :--------: | :----------------------------------------------------------: |
|  O_CREAT   | 文件不存在则创建文件，使用此选项时需要使用mode说明文件的权限 |
|   O_EXCL   |        如果同时指定了O_CREAT，且文件已经存在，则出错         |
|  O_TRUNC   |                 如果文件存在，则清空文件内容                 |
|  O_APPEND  |                 写文件时，数据添加到文件末尾                 |
| O_NONBLOCK |      对于设备文件，以O_NONBLOCK方式打开可以做非阻塞I/O       |

**mode补充说明**

1）文件最终权限：mode & ~umask

2）shell进程的umask掩码可以使用umask命令查看

* umask：查看掩码（补码）
* umask mode：设置掩码，mode为八进制
* umask -S：查看各组用户的默认操作权限

| **取值** | **八进制** | 含义                                   |
| :------- | :--------- | :------------------------------------- |
| S_IRWXU  | 00700      | 文件所有者的读、写、可执行权限         |
| S_IRUSR  | 00400      | 文件所有者的读权限                     |
| S_IWUSR  | 00200      | 文件所有者的写权限                     |
| S_IXUSR  | 00100      | 文件所有者的可执行权限                 |
| S_IRWXG  | 00070      | 文件所有者同组用户的读、写、可执行权限 |
| S_IRGRP  | 00040      | 文件所有者同组用户的读权限             |
| S_IWGRP  | 00020      | 文件所有者同组用户的写权限             |
| S_IXGRP  | 00010      | 文件所有者同组用户的可执行权限         |
| S_IRWXO  | 00007      | 其他组用户的读、写、可执行权限         |
| S_IROTH  | 00004      | 其他组用户的读权限                     |
| S_IWOTH  | 00002      | 其他组用户的写权限                     |
| S_IXOTH  | 00001      | 其他组用户的可执行权限                 |

```C
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <fcntl.h>
#include <unistd.h>

//打开和关闭文件
int main()
{
	int fd = -1;

	//1、以只读的方式打开一个文件,如果文件不存在就报错
	//fd = open("txt", O_RDONLY);
    
	//2、以只写的方式打开一个文件
	//fd = open("txt", O_WRONLY | O_CREAT, 0644);
	
	//3、以只写的方式打开一个文件，如果文件操作就报错，如果文件不存在就新建一个文件
	//fd = open("txt", O_WRONLY | O_CREAT | O_EXCL, 0644);
	
	//4、以读写的方式打开一个文件，如果文件操作就打开，如果文件不存在就新建一个文件
	//fd = open("txt", ORDWR | O_CREAT, 0644);
	
	// 5、如果文件不存在，则创建，存在，则打开并清空内容
	//fd = open("txt2", O_WRONLY | O_CREAT | O_TRUNC);
	
	//以只写的方式和追加的方式打开一个文件，如果文件不存在会报错
	fd = open("txt", O_WRONLY | OAPPEND);
	if(-1 == fd)
	{
		perror("open");
		return 1;
	}

	//关闭文件
	close(fd);

	return 0;
}
```



## 7.2 close函数

```C
#include <unistd.h>

int close(int fd);
/*
功能：
	关闭已打开的文件
参数：
	fd：文件描述符，open()的返回值
返回值：
	成功：0
	失败：-1，并设置errno
*/
```

需要说明的是，当一个进程终止时，内核对该进程所有尚未关闭的文件描述符调用close关闭，所以即使用户程序不调用close，在终止时内核也会自动关闭它打开的所有文件。

但是对于一个长年累月运行的程序(比如网络服务器)，打开的文件描述符一定要记得关闭,否则随着打开的文件越来越多，会占用大量文件描述符和系统资源。

## 7.3 write函数

```C
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_f count);
/*
功能：
	把指定数目的数据写到文件(fd)
参数：
	fd：文件描述符
	buf：数据首地址
	count：写入数据的长度（字节）
返回值：成功：实际写入数据的字节个数
	   失败：-1
*/
```

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

//写文件
int main(void)
{
	int fd = -1;
	int ret = -1;

	char *str = "hello itcast";
	//1. 以只写的方式打开一个文件
    fd = open("txt", O_WRONLY | O_CREAT, 0644);
	if(-1 == fd)
	{
		perror("open");
		return 1;
	}

	printf("fd = %d\n", fd);
	//2. 写文件
	ret = write(fd, str, strlen(str));
	if(-1 == ret)
	{
		perror("write");
		return 1;
	}

	printf("write len: %d\n", ret);
	//3. 关闭文件
	close(fd);

	return 0;
}
```

## 7.4 read函数

```C
#include <unistd.h>

ssize_t read(int fd, void *buf, size_f count);

/*
功能：
	把指定数目的数据读到指定缓冲区
参数：
	fd：文件描述符
	buf：内存首地址
	count：读取的字节个数
返回值：
	成功：实际读取到的字节个数
	失败：-1
*/
```

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define SIZE 128

// 读文件
int main(void)
{
	int fd = -1;
	int ret = -1;
	
	char buf[SIZE];
	
	//1.以只读的方式打开一个文件
	fd = open("txt", O_RDONLY);
	if(-1 == fd)
	{
		perror("open");
		return 1;
	}

	printf("fd = %d\n", fd);

	// 2.读文件
	memset(buf, 0, SIZE);
	ret = read(fd, buf, SIZE);
	if(-1 == ret)
	{
		perror("read");
		return 1;
	}

	printf("read len: %d %s\n", ret, buf);

	//3.关闭文件
	close(fd);

	return 0;
}
```

### 7.4.1 阻塞和非阻塞的概念

读常规文件是不会阻塞的，不管读多少字节，read一定会在有限的时间内返回。

从终端设备或网络读则不一定，**如果从终端输入的数据没有换行符，**调用read读终端设备就会阻塞，如果网络上没有接收到数据包，调用read从网络读就会阻塞，至于阻塞多长时间也是不确定的，如果一直没有数据到达就一直阻塞在那里。比如使用getchar读取终端字符，就会阻塞。

同样，写常规文件是不会阻塞的，**而向终端设备或网络写则不一定**

【注意】阻塞与非阻塞是对于文件而言的，而不是read、write等的属性。

以非阻塞方式打开文件程序示例：

```C

#include <unistd.h> //read
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <errno.h> //EAGAIN
​
int main()
{
    // /dev/tty --> 当前终端设备
    // 以不阻塞方式(O_NONBLOCK)打开终端设备
    int fd = open("/dev/tty", O_RDONLY | O_NONBLOCK);
​
    char buf[10];
    int n;
    n = read(fd, buf, sizeof(buf));
    if (n < 0)
    {
        // 如果为非阻塞，但是没有数据可读，此时全局变量 errno 被设置为 EAGAIN
        if (errno != EAGAIN)
        {
            perror("read /dev/tty");
            return -1;
        }
        printf("没有数据\n");
    }
​
    return 0;
}
```

## 7.5 lseek函数

```C
#include <sys/types.h>
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
/*
功能：
	改变文件的偏移量
参数：
	fd：文件描述符
	offset：根据whence来移动的位移数（偏移量），可以是正数，也可以是负数，如果是正数，则相对于whence往右移动，如果是负数，则相对于whence往左移动。如果向前移动的字节超过了文件开头则出错返回，如果向后移动的字节数超过了文件末尾，再次写入时将增大文件尺寸。
	
		whence：取值如下：
			SEEK_SET：从文件开头移动offset个字节
			SEEK_CUR：从当前位置移动offset个字节
			SEEK_END：从文件末尾移动offset个字节
返回值：
	若lseek成功执行，则返回新的偏移量
	如果失败，返回-1
*/
```

所有打开的文件都有一个当前文件偏移量（current file offset），以下简称为cfo。cfo通常是一个非负整数，用于表明文件开始处到文件当前位置的字节数。

读写操作通常开始于cfo，并且使cfo增大，增量为读写的字节数。文件别打开时，cfo会被初始化为0，除非使用了O_APPEND。

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define SIZE 128

//lseek使用
int main(void)
{
	int ret = -1;
	int fd = -1;

	char buf[SIZE];

	//1.打开文件
	fd = open("txt", O_RDWR | O_CREAT, 0644);
	if(-1 == fd)
	{
		perror("open");
		return -1;
	}
	printf("fd = %d\n", fd);

	// 2.lseek操作
	write(fd, "ABCDEFG", 7);

	//从文件开头偏移32个字节
	ret = lseek(fd, 32, SEEK_SET);
	if(-1 == ret)
	{
		perror("lseek");
		return 1;
	}

	write(fd, "1234567890", 10);
	
	lseek(fd, 0, SEEK_SET);
	memset(buf, 0, SIZE);
	ret = read(fd, buf, SIZE);
	printf("reas len: %d, read buf: %s\n",ret, buf);
	// 3.关闭文件
	close(fd);

	return 0;
}
```

# 8 扩展-ctags使用

第一步： 安装ctags

> deng@itcast:~/linux-4.16.12$ sudo apt install exuberant-ctags

 

第二步：/usr/include中生成tags文件

> 切换到/usr/include
>
> deng@itcast:/usr/include$ pwd /usr/include
>
> 生成ctags文件
>
> deng@itcast:/usr/include的密码： ls -l tags -rw-r--r-- 1 root root 5271877 5月 31 13:00 tags deng@itcast:/usr/include$

 

第二个tags

> deng@itcast:/usr/src/linux-headers-4.10.0-28 sudo ctags -Rn .

 

第三步: 配置vimrc

在~/.vimrc文件中最后一行添加如下内容：

> set tags+=/usr/include/tags
>
> set tags+=/usr/src/linux-headers-4.10.0-28/tags

 

第四步： 生效vimrc

执行如下命令

> deng@itcast:~/linux-4.16.12$ source ~/.vimrc

 

Ctrl + ] 表示跟踪代码

Ctrl + t 表示回去

> deng@itcast:~$ vim -t STDIN_FILENO
