> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## 信号的相关的概念

### 信号的共性

- 简单
- 不能携带大量信息
- 满足条件才能 发送
  
  ### 信号的特质
  
  信号是软件层面的“中断”。一旦信号产生，无论程序执行到什么位置，必须立即停止，处理信号，处理结束后，再继续执行后续指令。
  
  > **所有信号的产生及处理全部都是由内核完成的**

### 信号的产生

1. 按键产生。如`CTRL+c`,`CTRL+z`,`CTRL+\`.
2. 系统调用产生。如`kill`,`raise`,`abort`
3. 软件条件产生。如定时器`alarm`,`setitimer`,`sleep`
4. 硬件异常产生。如非法访问内存（段错误），内存对齐错误（总线错误）
5. 命令产生。如`kill`命令

### 信号的两种状态

- 未决：产生与递达之间的状态。
- 递决：产生并且送达到进程，直接被内核处理掉。

### 信号的处理方式

- 执行默认动作
- 忽略
- 捕捉（自定义）

### 信号屏蔽字和未决信号集

![1](https://img-blog.csdnimg.cn/direct/4361e91137ad4214a48085d2a86c8915.png#pic_center)

- 阻塞信号集（信号屏蔽字）：本质是位图。用来记录信号的屏蔽状态。一旦被屏蔽的信号，在解除屏蔽前，一直处于未决态。
- 未决信号集：本质是位图。用来记录信号的处理状态。该信号集中的信号，表示，已经产生，但尚未被处理。

### 信号四要素

- 编号
- 名称
- 对应事件
- 默认处理动作
  
  > **信号使用之前，应先确定其4要素，而后再用！！！**

### 常规信号

| **编号** | **名称**    | **对应事件**                                                                   | **默认处理动作**                                           |
|:------:|:---------:|:--------------------------------------------------------------------------:|:----------------------------------------------------:|
| 1      | SIGHUP    | 本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联。 | 挂断                                                   |
| 2      | SIGINT    | 在用户键入INTR字符(通常是Ctrl+C)时发出                                                  | 通知前台进程组终止进程                                          |
| 3      | SIGQUIT   | 由QUIT字符(通常是Ctrl+)来控制                                                       | 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号。        |
| 4      | SIGILL    | 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出也有可能产生这个信号。                               | 终止程序                                                 |
| 5      | SIGTRAP   | 由断点指令或其它陷阱（trap）指令产生                                                       |                                                      |
| 6      | SIGABRT   | 调用abort函数生成的信号。                                                            |                                                      |
| 7      | SIGBUS    | 访问非法地址                                                                     | 终止程序                                                 |
| 8      | SIGFPE    | 在发生致命的算术运算错误时发出                                                            | 终止程序                                                 |
| 9      | SIGKILL   | 发现某个进程终止不了                                                                 | 立即结束程序的运行                                            |
| 10     | SIGUSR1   | 留给用户自定义                                                                    | 留给用户自定义                                              |
| 11     | SIGSEGV   | 试图访问未分配给自己的内存,或试图往没有写权限的内存地址写数据                                            | 终止程序                                                 |
| 12     | SIGUSR2   | 留给用户自定义                                                                    | 留给用户自定义                                              |
| 13     | SIGPIPE   | 通常在进程间通信产生                                                                 | 终止程序                                                 |
| 14     | SIGALRM   | 时钟定时信号                                                                     | 终止程序                                                 |
| 15     | SIGTERM   | 程序结束(terminate)信号                                                          | 终止程序                                                 |
| 16     | SIGCHLD   | 子进程（child）状态变化时, 父进程会收到这个信号                                                | 忽略                                                   |
| 17     | SIGCONT   |                                                                            | 让一个停止(stopped)的进程继续执行                                |
| 18     | SIGSTOP   | 需要暂停某一程序                                                                   | 暂停(stopped)进程的执行，注意它和terminate以及interrupt的区别:该进程还未结束 |
| 19     | SIGTSTP   | 用户键入SUSP字符时(通常是Ctrl+Z)发出这个信号                                               | 终止程序                                                 |
| 20     | SIGTTIN   | 当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行.                      | 终止程序                                                 |
| 21     | SIGTTOU   | 类似于SIGTTIN, 但在写终端(或修改终端模式)时收到                                              | 终止程序                                                 |
| 22     | SIGURG    | 有”紧急”数据或out-of-band数据到达socket时产生.                                          | 终止程序                                                 |
| 23     | SIGXCPU   | 超过CPU时间资源限制                                                                | 终止程序                                                 |
| 24     | SIGXFSZ   | 当进程企图扩大文件以至于超过文件大小资源限制。                                                    | 终止程序                                                 |
| 25     | SIGVTALRM | 虚拟时钟信号                                                                     | 终止程序                                                 |
| 26     | SIGPROF   | 类似于SIGALRM/SIGVTALRM, 但包括该进程用的CPU时间以及系统调用的时间.                              | 终止程序                                                 |
| 27     | SIGWINCH  | 窗口大小改变时发出.                                                                 | 终止程序                                                 |
| 28     | SIGIO     | 文件描述符准备就绪, 可以开始进行输入/输出操作.                                                  | 终止程序                                                 |
| 29     | SIGPWR    | Power failure                                                              | 终止程序                                                 |
| 30     | SIGSYS    | 非法的系统调用。                                                                   | 终止程序                                                 |

> 由于**初学**，后面的一些信号我**不敢保证正确性**（后面的都是我**查资料cv过来**的，等我学明白之后我会来**更新这个表格**）
> 上面列出的信号中，程序**不可捕捉，阻塞，忽略**的是`SIGKILL`,`SIGSTOP`

---

## kill函数

`int kill(pid_t  pid, int signum`

### 参数

- `pid`:
  - `>0`，发送信号给指定的进程
  - `=0`,发送信号给跟调用kill函数的那个进程处于同一进程组的进程。
  - `<-1`，取绝对值，发送信号给该绝对值所对应的进程组的所有组员。
  - `=-1`,发送信号给，有权限发送的所有进程。
- `signum`,待发送的信号

### 返回值

- 成功，0
- 失败，-1，`errno`被设置。

### 举个栗子

我们写一个程序，用子进程调用`kill`杀死父进程
源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

int main()
{
    pid_t pid;
    pid=fork();
    if(0==pid)
{
        printf("I am child,i will kill my parent\n");
        sleep(1);
        int res=kill(getppid(),SIGKILL);
        if(res==-1)
            perror("kill error");
    }
    else 
    {
        for(int i=0;;i++)    
            printf("%d-------I am parent, i will be killed by my child\n",i);
    }
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/0f61cc2b5e5c47309221a427be8bcfaa.png#pic_center)

---

## alarm函数

每个进程都有唯一的闹钟
`unsigned int alarm(unsigned int seconds)`

### 参数

- second，定时的秒数，单位是`s`

### 返回值

- 上次定时剩余的时间

### 取消闹钟

`alarm(0);`

### 举个栗子

我们使用`alarm`来计时，统计计算机一秒钟能打印多少。
源代码

```c
#include<stdio.h>
#include<unistd.h>

int main()
{
    alarm(1);
    int i;
    for(i=0;;i++)
        printf("%d\n",i);
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/a36de318942a4b58963c533ae9ca9623.png#pic_center)

---

## setitimer函数

设置闹钟，可以替代alarm函数，精度微秒us，可以实现周期定时
`int setitimer(int which,const struct itimerval * new_value,struct itimerval * old_val)`

### 参数

- `which`:选择计时方式
  - `ITIMER_REAL`:采用自然计时。（一般用这个）——>`SIGALRM`
  - `ITIMER_VIRTUAL`:采用用户空间计时——>`SIGVTALRM`
  - `ITIMER_PROF`:采用内核+用户空间计时——>`SIGPROF`
- `new_value`:定时秒数
- `old_value`:传出参数，上次定时剩余的时间.（如果不关心的话，可以直接传`NULL`)

### 返回值

- 成功，0
- 失败，-1，`errno`被设置

### `struct itimerval`类型

```c
struct itimerval
{
    struct timeval 
    {
        time_t tv_sec;            /*second*/
        suseconds_t tv_usec;    /*microsecond*/
    }it_interval;                /*用于设定两个任务之间的间隔时间*/
    struct timerval
    {
        time_t tv_sec;            /*second*/
        susecond_t tv_usec ;    /*microsecond*/
    }it_value;                    /*第一次定时秒数*/
}
```

可以理解有两个定时器

- 一个用于第一个闹钟什么时候触发
- 第一个闹钟触发后间隔多少时间再次触发闹钟。

### 举个栗子

#### 计数

功能和上面的`alarm`一样
源代码：

```c
#include<unistd.h>
#include<stdio.h>
#include<sys/time.h>

int main()
{
    struct itimerval t;
    t.it_interval.tv_sec=0;
    t.it_interval.tv_usec=0;

    t.it_value.tv_sec=1;
    t.it_value.tv_usec=0;
    int res=setitimer(ITIMER_REAL,&t,NULL);
    for(int i=0;;i++)
        printf("%d\n",i);
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/3db06b75e79a4edb98502cbe86f397dc.png#pic_center)

### 向屏幕打印信息

源代码：

```c
#include<unistd.h>
#include<stdio.h>
#include<sys/time.h>
#include<signal.h>

void fun(int signal)
{
    printf("hellow linux\n");
}

int main()
{
    signal(SIGALRM,fun);
    struct itimerval t;
    t.it_interval.tv_sec=2;
    t.it_interval.tv_usec=0;

    t.it_value.tv_sec=1;
    t.it_value.tv_usec=0;
    int res=setitimer(ITIMER_REAL,&t,NULL);
    while(1);
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/dffeefb97b21414cb5fc1c85226aa007.png#pic_center)

---

## 信号集操作函数

- 信号集`set`函数
  - `sigset_t set;` ：自定义信号集（这是一种类型）
  - `sigemptyset(sigset_t* set);`：清空信号集，全部置为0
  - `sigfillset(sigset_t * set);`：填满信号集，全部置为1
  - `sigaddset(sigset_t* set);`：将指定信号添加到信号集中，即将指定信号置为1
  - `sigdelset(sigset_t* set);`：将指定信号移除信号集，即将指定信号置为0
  - `sigismember(const sigset_t* set,int signum);`：判断指定信号是否在集合中。在—>1,不在—>0.
- `sigprocmask`函数
  `int sigprocmask(int how,const sigset_t * set,sigset* oldset);`
  - `how`:
    - `SIG_BLOCK`:设置阻塞（位与）
    - `SIG_UNBLOCK`:取消阻塞（取反位与）
    - `SIG_SETMASK`:用自定义`set`填充`mask` （不推荐）
  - `set`:自定义的`set`\
  - `oldset`：旧有的`mask`（不用的话可以传`NULL`）
- `sigpending`函数
  `int sigpending(sigset* set)`
  读取未决信号集，参数是传出的未决信号集。

![1](https://img-blog.csdnimg.cn/direct/44875c1387584901adaab5b22b8b6e52.png#pic_center)

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

void print_set(sigset_t* pset)
{
    for(int i=1;i<33;i++)
        if(sigismember(pset,i))
            printf("1");
        else 
            printf("0");
    printf("\n");
}

int main()
{
    sigset_t set,pset;
    sigemptyset(&set);
    sigaddset(&set,SIGINT);
    sigaddset(&set,SIGQUIT);
    sigaddset(&set,SIGKILL);
    sigaddset(&set,SIGHUP);
    sigprocmask(SIG_BLOCK,&set,NULL);
    while(1)
    {
        sigpending(&pset);
        print_set(&pset);
        sleep(1);
    }
    return 0;
}
```

效果
![2](https://img-blog.csdnimg.cn/direct/af4b8351cfa348538ed5ffc9dfa7f52b.png#pic_center)

---

## signal实现信号捕捉

**注册**一个信号捕捉函数，ANS设置，不同操作系统存在差异**建议使用sigaction函数**
`sighandler_t signal(int signum,sighandler_t handler)`

### 参数：

- `signum`     :捕捉信号
- `handler`:捕捉信号后的操作函数

### 返回值

- 成功，返回操作函数的返回值
- 失败，返回`SIG_ERR`,`errno`被设置。

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

void fun(int signal)
{
    printf("hellow linux,SIGQUIT has been not work\n");
}

int main()
{
    signal(SIGQUIT,fun);
    while(1);
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/c607a7aee7e14981a79f9495c0b5ff25.png#pic_center)

---

## sigaction实现信号捕捉

注册一个信号捕捉函数
`int sigaction(int signum,const struct sigaction * act,struct sigaction * oldact)`

### `struct sigaction`类型

```c
struct sigaction {
    void     (*sa_handler)(int);        //操作函数名
    void     (*sa_sigaction)(int, siginfo_t *, void *);        //一般不用管
    sigset_t   sa_mask;        //只作用于函数捕捉期间
    int        sa_flags;    //一般置为0，表示用默认的
    void     (*sa_restorer)(void);
};
```

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

void fun(int signal)
{
    printf("catch you SIGQUIT\n");
}

int main()
{
    struct sigaction act;
    act.sa_handler=fun;
    sigemptyset(&act.sa_mask);
    act.sa_flags=0;
    sigaction(SIGQUIT,&act,NULL);
    while(1);
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/72f9f4a7073d4c00bf512fa156250357.png#pic_center)

---

## 信号捕捉

### 信号捕捉的特性

- 捕捉函数执行期间，信号屏蔽字 由 `mask` --> `sa_mask` , 捕捉函数执行结束。 恢复回`mask`
- 捕捉函数执行期间，本信号自动被屏蔽(`sa_flgs = 0`).其他信号不屏蔽，如需屏蔽则调用`sigsetadd`函数修改
- 捕捉函数执行期间，被屏蔽信号多次发送，解除屏蔽后只处理一次！

### 内核实现信号捕捉简析

![3](https://img-blog.csdnimg.cn/direct/5f8317d02e8342cc85fca53fca464068.png#pic_center)

---

## 综合练习

我们借助信号捕捉回收子进程
源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<signal.h>
#include<sys/wait.h>

void fun(int signal)
{
    while(waitpid(-1,NULL,0)!=-1)
        printf("wait successfully\n");

}

int main()
{
    int i;
    pid_t pid;
     sigset_t set;
    sigemptyset(&set);
    sigaddset(&set,SIGCHLD);
    sigprocmask(SIG_BLOCK,&set,NULL);
    for(i=0;i<15;i++)
    {
        pid=fork();
        if(0==pid)
            break;
    }
    if(15==i)
    {
        struct sigaction act;
        act.sa_handler=fun;
        sigemptyset(&act.sa_mask);
//        sigdelset(&set,SIGCHLD);
        sigprocmask(SIG_UNBLOCK,&set,NULL);
        act.sa_flags=0;
        sigaction(SIGCHLD,&act,NULL);
        printf("i am parent,i wait my all child\n");
        while(1);
    }
    else
    {
        printf("i am %dth child,i will be waited\n",i+1);
    }
    return 0;
}
```

效果：
![4](https://img-blog.csdnimg.cn/direct/07d233ade8f748909028ad71bac61cd9.png#pic_center)

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
>    - [最小生成树算法———Kruskal算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135039904?spm=1001.2014.3001.5501)
>    - [染色法判断二分图（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135094296?spm=1001.2014.3001.5501)
> 2. **Linux部分（还在更新中）：**
>    - [Linux学习之初识Linux](https://blog.csdn.net/yourgrandfather_/article/details/134953315?spm=1001.2014.3001.5501)
>    - [Linux学习之命令行基础操作](https://blog.csdn.net/yourgrandfather_/article/details/134956923?spm=1001.2014.3001.5501)
>    - [Linux学习之基础命令（适合小白）](https://blog.csdn.net/yourgrandfather_/article/details/135189166)
>    - [Linux学习之权限管理和用户管理](https://blog.csdn.net/yourgrandfather_/article/details/135222868)
>    - [Linux学习之制作静态库和动态库](https://blog.csdn.net/yourgrandfather_/article/details/135275850)
>    - [Linux学习之makefile](https://blog.csdn.net/yourgrandfather_/article/details/135288190)
>    - [Linux学习之系统编程1（关于读写系统函数）](https://blog.csdn.net/yourgrandfather_/article/details/135324720)
>    - [Linux学习之系统编程2（关于进程及其相关的函数）](https://blog.csdn.net/yourgrandfather_/article/details/135338115)
>    - [Linux学习之系统编程3（进程及wait函数）](https://blog.csdn.net/yourgrandfather_/article/details/135354402)
>    - [Linux学习之系统编程4(进程间通信）](https://blog.csdn.net/yourgrandfather_/article/details/135383973)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
