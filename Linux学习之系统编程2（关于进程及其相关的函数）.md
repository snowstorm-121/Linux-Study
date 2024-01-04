> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## 基础概念介绍

### 关于进程和程序

程序：死的。只占用磁盘空间。
进程：活的。运行起来的程序。占用内存、cpu 等系统资源。 

> 我们可以**把程序比作剧本**，**进程比作戏**。把剧本演绎起来就是戏（**程序运行起来就是进程**），剧本可以在不同的戏台上演绎，也可以让不同的人来演绎（**程序可以不同的环境上运行，也可以用不同的系统资源来运行**），这样可以更方便理解。

### 关于cpu

![cpu](https://img-blog.csdnimg.cn/direct/9e83f245205246c8815a9c4ef6fd90d1.png#pic_center)

### 虚拟内存和物理内存映射关系

![mem](https://img-blog.csdnimg.cn/direct/a9b57572989f48bab741cb3b3343d2bf.png#pic_center)
不同程序在物理内存的0-3G部分经过MMU映射到不同的虚拟内存，而在物理内存的3G-4G经过MMU映射后到相同的虚拟内存。

### PCB 进程控制块

PCB进程控制块：

- 进程 id
- 文件描述符表
- 进程状态： 初始态、就绪态、运行态、挂起态、终止态               
- 进程工作目录位置
- 信号相关信息资源
- 用户 id 和组 id
  ![pcb](https://img-blog.csdnimg.cn/direct/1bbd438f57164443a6be69201da52d71.png#pic_center)

### 环境变量

- echo $PATH 查看环境变量
- echo $TERM 查看终端
- echo $LANG 查看语言
- env 查看所有环境变量
  
  > **path** 环境变量里记录了一系列的值，当运行一个可执行文件时，系统会去环境变量记录的位置里查找这个文件并执行。
  > 这些查看环境变量的还有很多，我这里就只是写了几个最常见的的。

---

## fork函数

### fork介绍

![1](https://img-blog.csdnimg.cn/direct/101678ebc7254651834c72471104590d.png#pic_center)
这个函数使用起来非常简单，不用传入参数，就一个`int`的返回参数。我们先来看看返回参数。
![2](https://img-blog.csdnimg.cn/direct/46645b722fbb4bb9a5f016611f4424a4.png#pic_center)
我们乍一看可能觉得有点古怪，一个函数怎么能返回两个值呢？

- 成功时，在父进程中返回子进程的PID，在子进程中返回0。这其实并没有矛盾，因为`fork`之后，就有两个进程，两个进程返回两种值。
- 失败时，在父进程中返回-1，没有子进程被创建，`errno`被设置。

### 父子进程相同

- data 段、text 段、堆、栈、环境变量、全局变量
- 宿主目录位置
- 进程工作目录位置
- 信号处理方式

### 父子进程不同

-  进程 id
- 返回值
- 各自的父进程
- 进程创建时间
- 闹钟
- 未决信号集

### 父子进程共享

读时共享，写时复制————全局变量

1. 文件描述符
2. mmap映射区

### 实例

我们以循环创建N个子进程为例
源代码（代码里用到`getpid` `getppid`这两个函数，不懂的先不用管，我后面会介绍）：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>

int main()
{
        pid_t pid;
        int i;
        printf("-------start1---------\n");
        printf("-------start2---------\n");
        printf("-------start3---------\n");
        printf("-------start4---------\n");
        for(i=0;i<5;i++)
        {
                pid=fork();
                if(pid==0)
                        break;
        }
        if(i==5)
        {
                sleep(i);
                printf("I'm parent,my pid is %d,my parent id is %d\n",getpid(),getppid());
        }
        else
        {
                sleep(i);
                printf("I'm %dth child,my pid is %d,my parent id is %d\n",i+1,getpid(),getppid());
        }
        return 0;
}
```

效果：
![3](https://img-blog.csdnimg.cn/direct/1b9a1a35420d4bd4aa78d8ac564d0396.png#pic_center)
可能会犯的错误：

- 在`for`循环中没有判断`pid==0`跳出循环。如果没有判断跳出，那就不是创造5个进程。
- 很多人在第一次写的时候发现子进程出来的顺序并没有按照自己创建的顺序，并且每一次执行都不一样。这是因为每个子进程被创建出来后，都要去争抢CPU资源，谁抢到了就可以打印，虽然可能第一个子进程被创建早一些，但是这早一点的时间对于抢CPU并没有什么太大的优势。所以我们就可以人为加一个`sleep`来控制每个子进程打印的顺序。（必须要说的是，不加`sleep`这样的人为控制的，也可以成功创建5个子进程，这样只不过让这个过程更可视化）

---

## getpid和getppid函数

![1](https://img-blog.csdnimg.cn/direct/0546c3fc233c464aba07130ea84c72af.png#pic_center)

这两个函数非常简单，看名字就知道`getpid()`是取得该进程的`pid`，`getppid`是取得该进程的父进程的`pid`.
具体的实例可以参考上面的代码。

---

## exec函数族

### `exec`函数族：使进程执行某一程序。成功无返回值，失败返回 -1。

![1](https://img-blog.csdnimg.cn/direct/1cc5a2b61d4245c2a3cd6853a2d5bb97.png#pic_center)

### 参数解读：

以`int execlp(const char* file,const char * arg,.....)`为例（借助 PATH 环境变量找寻待执行程序）

- 参数1，程序名
- 参数2，argv[0]
- 参数3，argv[1]
- ......
- 最后一个参数，哨兵，`NULL`

### 举例说明

#### `execl`

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>

int main()
{
        pid_t pid=fork();
        if(pid==0)
        {
                execl("./test","./test",NULL);
                perror("execl error");
                exit(1);
        }
        else
        {
                sleep(1);
                printf("i am father\n");
        }
        return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/935367b264204ec587eb95df421d8315.png#pic_center)

> 这里是展示`execl`函数，execl函数可以执行**自定义的程序**，也可以执行**系统命令。**并且要注意**第一个参数是路径，不是文件名。**

#### `execlp`

源代码：

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main()
{
        pid_t pid;
        printf("--------------start----------\n");
        pid=fork();
        if(pid==-1)
        {
                perror("fork error");
                exit(1);
        }
        if(pid==0)
        {
                execlp("ls","ls","-l","-h","-a",NULL);
                perror("execlp error");
                exit(1);
        }
        if(pid>0)
        {
                sleep(1);
                printf("I'm father,my child id is %d,my id is %d\n",pid,getpid());
        }
        return 0;
}
```

效果：
![3](https://img-blog.csdnimg.cn/direct/0f9dfff79812418da4cda23126580cdb.png#pic_center)

> `execlp`用来执行一些**系统命令**

#### 综合实例

我们实现一个功能，把`ps aux`的信息写入一个文件。
源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>

void sys_error(char* s)
{
        perror(s);
        exit(1);
}

int main()
{
        int fd;
        fd=open("exec.txt",O_RDWR | O_CREAT,0644);
        if(fd<0)
                sys_error("open file");
        dup2(fd,STDOUT_FILENO);
        execlp("ps","ps","aux",NULL);
}
```

效果：
![4](https://img-blog.csdnimg.cn/direct/fa2a0f69a3d547988100891d880bcc6a.png#pic_center)

> `exec` 函数一旦调用成功，即执行新的程序，不返回。只有失败才返回，错误值-1，所以通常我们直接在 `exec` 函数调用后直接调用`perror()`，和 `exit()`，无需 `if `判断。
> 
> - l                    (list) 命令行参数列表
> - p                    (path) 搜索 file 时使用 path 变量
> - v                    (vector) 使用命令行参数数组
> - e                    (environment) 使用环境变量数组，不适用进程原有的环境变量，设置新加载程序运行的环境变量
> 
> 事实上，只有 `execve` 是真正的系统调用，其他 5 个函数最终都调用 `execve`，是库函数，所以`execve` 在` man` 手册第二节，其它函数在` man` 手册第 3 节。

---

## 孤儿进程和僵尸进程

### 孤儿进程：

父进程先于子进终止，子进程沦为“孤儿进程”，会被 init 进程领养。

### 僵尸进程：

子进程终止，父进程尚未对子进程进行回收，在此期间，子进程为“僵尸进程”。 kill 对其无效。这里要注意，每个进程结束后都必然会经历僵尸态，时间长短的差别而已。子进程终止时，子进程残留资源 PCB 存放于内核中，PCB 记录了进程结束原因，进程回收就是回收 PCB。回收僵尸进程，得 kill 它的父进程，让孤儿院去回收它。

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。正可谓“**键盘敲烂，月薪过万**”

---

> 如果你觉得我写的题解还不错的，请各位**王子公主**移步到我的其他题解看看
> 
> 1. **数据结构与算法部分（还在更新中）：**
>    - [***C++ STL总结 - 基于算法竞赛（强力推荐***）](https://blog.csdn.net/yourgrandfather_/article/details/135051716?spm=1001.2014.3001.5501)
>    - [动态规划——完全背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135111459)
>    - [动态规划——01背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135103012?spm=1001.2014.3001.5501)
>    - [动态规划——多重背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135125267)
>    - [动态规划——分组背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135134277)
>    - [动态规划——最长上升子序列（LIS)](https://blog.csdn.net/yourgrandfather_/article/details/135150351)
>    - [二叉树的中序遍历（三种方法）](https://blog.csdn.net/yourgrandfather_/article/details/135167817)
>    - [最长回文子串](https://blog.csdn.net/yourgrandfather_/article/details/135183977)
>    - [最短路算法——Dijkstra（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134869064?spm=1001.2014.3001.5501)
>    - [最短路算法———Bellman_Ford算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134935786?spm=1001.2014.3001.5501)
>    - [最短路算法———SPFA算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135004393?spm=1001.2014.3001.5501)
>    - [最小生成树算法———prim算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135026901?spm=1001.2014.3001.5501)
>    - [最小生成树算法———Kruskal算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135039904?spm=1001.2014.3001.5501
>    - [染色法判断二分图（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135094296?spm=1001.2014.3001.5501)
> 2. **Linux部分（还在更新中）：**
>    - [Linux学习之初识Linux](https://blog.csdn.net/yourgrandfather_/article/details/134953315?spm=1001.2014.3001.5501)
>    - [Linux学习之命令行基础操作](https://blog.csdn.net/yourgrandfather_/article/details/134956923?spm=1001.2014.3001.5501)
>    - [Linux学习之基础命令（适合小白）](https://blog.csdn.net/yourgrandfather_/article/details/135189166)
>    - [Linux学习之权限管理和用户管理](https://blog.csdn.net/yourgrandfather_/article/details/135222868)
>    - [Linux学习之制作静态库和动态库](https://blog.csdn.net/yourgrandfather_/article/details/135275850)
>    - [Linux学习之makefile](https://blog.csdn.net/yourgrandfather_/article/details/135288190)
>    - [Linux学习之系统编程1（关于读写系统函数）](https://blog.csdn.net/yourgrandfather_/article/details/135324720)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
