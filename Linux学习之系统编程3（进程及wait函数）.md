> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## wait&waitpid函数

![1](https://img-blog.csdnimg.cn/direct/d8318df92e7d419db6233796daf82bf4.png#pic_center)

### 我们先来看wait函数

#### 传入参数：

一个`int`类型的指针，各位童鞋看名字应该就可以猜出来是表示状态信息的。没错，`wstate`就是表示回收的子进程 信息，我们可以调用一些宏函数去判断子进程的信息（详细的后面我在说）。当然，你也可以传`NULL`，也不会报错。

#### 返回值：

- 成功，回收进程的`pid`
- 失败，-1，设置`errno`

#### 函数的作用

1. 阻塞等待子进程退出          
2. 清理子进程残留在内核的 `pcb` 资源
3. 通过传出参数，得到子进程结束状态

#### 利用宏函数获取子进程信息

- `WIFEXITED(status)`——>为真，子进程正常终止。再次调用`WSTATUS(status)`——>得到子进程退出值
- `WIFSIGNALED(status)`——>为真，子进程被信号终止。调用`WTERMSIG(status)`——>得到子进程异常终止的的信号编号。

一个进程终止时会关闭所有文件描述符，释放在用户空间分配的内存，但它的 PCB 还保留着，内
核在其中保存了一些信息：如果是正常终止则保存着退出状态，如果是异常终止则保存着导致该进程
终止的信号是哪个。这个进程的父进程可以调用` wait `或者` waitpid `获取这些信息，然后彻底清除掉
这个进程。我们知道一个进程的退出状态可以在 shell 中用特殊变量$？查看，因为 shell 是它的父
进程，当它终止时，`shell `调用 `wait `或者` waitpid `得到它的退出状态，同时彻底清除掉这个进程。

#### 举个栗子

源代码：

```c
#include<stdio.h>
#include<sys/wait.h>
#include<unistd.h>
#include<string.h>

int main()
{
        pid_t pid,wpid;
        int state;
        pid=fork();
        if(pid==0)
        {
                printf("I'm child,my pid is %d\n",getpid());
                sleep(3);
                printf("child go to die\n");
        }
        else if(pid>0)
        {
                wpid=wait(&state);
                if(wpid>0)
                        printf("i am parent,wait child id is %d,wait  successfully\n",wpid);
                else
                        perror("wait error");
        }
        return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/2bf57bc0540449879cdee9153349cd93.png#pic_center)

---

### 接着看waitpid函数

![3](https://img-blog.csdnimg.cn/direct/d8314c8fc052436284146af0660b7e3a.png#pic_center)

#### 传入参数

- `pid`，有四种可能的值：
  -  `<-1`:等待回收任何组id（`gid`)等于该值的绝对值的子进程。（`manpage`原文：`meaning  wait for any child process whose process group ID is equal to the absolute value of pid.`)
  - `-1`:等待回收任意一个子进程。（`manage`原文：`meaning wait for any child process.`)
  - `0`:等待回收任意一个组id和该进程组id一致的子进程。（`manpage`原文：`meaning wait for any child process whose process group ID is equal to that  of  the  calling process at the time of the call to waitpid()`)
  - `<0`:等待回收指定pid的子进程（这一种是用的最多的）。(`manpage`原文：`meaning wait for the child whose process ID is equal to the value of pid.`)
- `status`：（传出） 回收进程的状态。
- `options`：一般默认是阻塞，即一直等待直到回收一个子进程为止。也可以指定为`WNOHANG` 指定回收方式为，非阻塞。

#### 返回值：

- `>0`: 表成功回收的子进程 `pid`
- `0 `: 函数调用时， 参 3 指定了 `WNOHANG`， 并且，没有子进程结束。
- `-1`: 失败。`errno`

#### 举个栗子

我们演示一个小`demo`,之前我们循环创建过5个子进程，我们这次就来回收指定子进程（下面以回收第三个子进程为例）。

##### 错误演示

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main()
{
    pid_t pid,wpid;
    int i;
    for(i=0;i<5;i++)
    {
        if(fork()==0)
        {
            if(2==i)
                pid=getpid();
            break;
        }
    }
    if(5==i)
    {
        sleep(5);
        printf("--------in parent , before waitpid, pid= %d\n", pid);
        wpid(pid,NULL,0);
        printf("I'm parent, wait a child finish : %d \n", wpid);
    }
    else
    {
        sleep(i);
        printf("I'm %dth child, pid= %d\n", i+1, getpid());
    }
    return 0;
}                
```

效果：
![3](https://img-blog.csdnimg.cn/direct/93f01caa46fe443891266a9a1c03b0f9.png#pic_center)

> **错误分析：**
> 上面的代码看似非常对，我们用`pid`来获取第三个子进程的`id`，然后让父进程使用`waitpid`来回收。但是，再仔细看，我们是在子进程把第三个子进程的`id`赋值给变量`pid`,但是由于父子进程间遵循的是"读时共享，写时复制"，所以对于父进程中变量`pid`还是没有变，还是0.

##### 正确演示

源代码：

```c
#include<unistd.h>
#include<string.h>
#include<stdlib.h>
#include<sys/wait.h>

int main()
{
        pid_t pid,wpid,tpid;
        int i;
        for(i=0;i<5;i++)
        {
                pid=fork();
                if(0==pid)
                        break;
                if(2==i)
                        tpid=pid;
        }

        if(5==i)
        {
                sleep(5);
                wpid=waitpid(tpid,NULL,0);
                printf("tpid=%d\n",tpid);
                if(wpid==-1)
                {
                        perror("waitpid error");
                        exit(1);
                }
                printf("i am parent,wait child id is %d\n",wpid);
        }
        else
        {
                sleep(i);
                printf("I am %dth child,my pid is %d\n",i+1,getpid());
        }
        return 0;
}
```

效果：
![4](https://img-blog.csdnimg.cn/direct/c0c04804a5504747bb587c6a9bdc706b.png#pic_center)

#### waitpid回收多个子进程

**首先我们要无论是`wait`还是`waitpid`每一次调用都只能回收一个子进程**
源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main()
{
        int i;
        pid_t pid,wpid;
        for(i=0;i<5;i++)
        {
                pid=fork();
                if(!pid)
                        break;
        }

        if(5==i)
        {
                while((wpid=wait(NULL))!=-1)
                {
                        printf("wait child pid is %d\n",wpid);
                }
        }
        else
        {
                sleep(i);
                printf("i am %dth child,my pid is %d\n",i+1,getpid());
        }
        return 0;
}
```

效果：
![5](https://img-blog.csdnimg.cn/direct/1f411c780e6047b885767e86dd38de3b.png#pic_center)

#### 利用宏函数判断子进程返回状态

 源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main()
{
        pid_t pid,wpid;
        int state;
        pid=fork();
        if(pid==0)
        {
                printf("i am child,my pid is %d\n",getpid());
                sleep(15);
                printf("i am going to die\n");
        }
        else
        {
                wpid=wait(&state);
                if(WIFEXITED(state))
                        printf("i am parent,my child was terminal normally, child's pid is %d\n",wpid);
                else if(WIFSIGNALED(state))
                        printf("my child was terminaled with signal %d,child's pid is %d\n",WTERMSIG(state),wpid);
        }
        return 0;
}
```

效果：
自然终止
![6](https://img-blog.csdnimg.cn/direct/98c8f909ac9846e498c97bc8439a0c65.png#pic_center)
被信号终止
![7](https://img-blog.csdnimg.cn/direct/2cc87aef3b4f40faacc474d9732b27c2.png#pic_center)

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

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
