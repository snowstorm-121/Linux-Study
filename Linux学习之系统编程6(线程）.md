> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## 进程组和会话

### 进程组（别名：作业）

- 多个进程的集合，每个进程都属于一个进程组，简化对多个进程的管理，`waitpid`和`kill`函数的参数中用到。
- 父进程创建子进程的时候默认父子进程属于同一进程组。进程组的ID==第一个进程ID（组长进程），组长进程id==进程组id，组长进程可以创建一个进程组，创建该进程组中的进程，然后终止。
- 只要有一个进程存在，进程组就存在，生存期与组长进程是否终止无关
- `kill -SIGKILL -进程组id` 杀掉整个进程组
- 一个进程可以为自己或子进程设置进程组id

### 会话

多个进程组的集合
![1](https://img-blog.csdnimg.cn/direct/861b947df5234baaa95b27202e962841.png#pic_center)
创建会话的注意事项

- 调用进程不能是进程组组长，该进程变成新会话首进程（平民）
- 该进程成为一个新进程组的组长进程
- 需要root权限（ubuntu不需要）
- 新会话丢弃原有的控制终端，该会话没有控制终端
- 该调用进程是组长进程，则出错返回
- 建立新会话时，先调用fork，父进程终止，子进程调用setsid

### getpid函数

`pid_t getpid(pid_t pid)`

- 功能：获取当前进程的会话id
- 返回值：
  - 成功，返回调用进程的会话id
  - 失败，-1，设置`errno`

### setsid函数

`pid_t setsid()`

- 功能：创建一个会话，并以自己的id设置进程id，同时也是新会话id
- 返回值：
  - 成功，返回调用进程的会话id
  - 失败，-1，设置`errno`

---

## 守护进程

### 概念和特点

- `daemon`进程。通常运行与操作系统后台，脱离控制终端。一般不与用户直接交互。周期性的等待某个事件发生或者周期性执行某一动作。
- 不受用户登录注销影响，通常采用以d结尾的命名方式。
- 创建守护进程的关键是调用`setsid()`函数创建一个新的 `session`,并成为`session leader`.

### 步骤

1. `fork`子进程，让父进程终止。
2. 子进程调用 `setsid()` 创建新会话
3. 通常根据需要，改变工作目录位置` chdir()`， 防止目录被卸载。
4. 通常根据需要，重设umask文件权限掩码，影响新文件的创建权限。（一般设置为`022`)
5. 通常根据需要，关闭/重定向 文件描述符.
6. 守护进程 业务逻辑。

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<fcntl.h>
#include<sys/stat.h>

int main()
{
    pid_t pid;
    pid=fork();
    if(pid>0)
    {
        printf("i am parent,my session id is %d\n",getsid(getpid()));
        exit(1);
    }
    printf("i am child,my session id is %d\n",getsid(getpid()));
    int res=setsid();
    if(res<0)
    {
        perror("getsid error");
        exit(1);
    }    
    printf("i am child, my session id has changed to %d\n",res);
    umask(0002);
    close(STDIN_FILENO);
    int fd=open("/dev/null",O_RDWR);
    dup2(fd,STDOUT_FILENO);
    dup2(fd,STDERR_FILENO);
    while(1);
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/f6ea0a987f2e47f38102b75452188585.png#pic_center)
![3](https://img-blog.csdnimg.cn/direct/9f094abf71f24144bc3112a1e94b3342.png#pic_center)

> **这个daemon不会受用户登录注销影响，想要终止必须使用`kill`命令**

---

## 线程的基本概念

进程：有独立的 进程地址空间。有独立的pcb。    分配资源的最小单位。
线程：有独立的pcb。没有独立的进程地址空间。    最小单位的执行。
![1](https://img-blog.csdnimg.cn/direct/2401a627c50f428fbaf592456dfe3696.png#pic_center)
`ps -Lf 线程id`——> 线程号LWP，CPU执行的最小单位
`ps -Lf 进程号`——> 查看进程的线程

线程共享： `./text./data ./rodataa ./bsss heap`  ---> 共享【全局变量】（`errno`）
线程非共享：栈空间（内核栈、用户栈）

> **三级映射**
> 
> 1. 轻量级线程，存在pcb，创建线程使用的底层函数和进程一样，都是clone
> 2. 从内核看进程和线程一样，都有各自不同的pcb，但是pcb中指向内存资源的三级页表是相同的
> 3. 进程可以变成线程
> 4. 线程可以看成寄存器和栈的集合
> 5. 线程是最小的执行单位，进程是最小的资源分配的单位
>    ![2](https://img-blog.csdnimg.cn/direct/0185caca28ae4a859d7a072e6d6dca43.png#pic_center)

---

## 创建线程

### `pthread_self()`函数

`pthread_t pthread_self();`

- 功能：获取线程id。线程id是在【进程】地址空间内部，用来标识线程身份的id号。
- 返回值：本线程的id号。

### `pthread_create()`函数

`int pthread_create(pthread_t* tid,const pthread_attr_t* attr,void* (*start_rountn)(void*),void* arg);`

- 功能：创建子线程
- 参数
  - `tid`:传出参数，表示新创建的子线程id
  - `attr`:线程属性，一般传`NULL`表示默认属性。
  - `start_rountn`:子线程回调函数。创建成功，`pthread`返回时，该函数自动被调用。
  - `arg`:回调函数的参数，没有的话，传`NULL`.
- 返回值：
  - 成功，0
  - 失败，返回错误号`ret`
    
    > 这里要注意，线程里错误检查和其他地方有些不一样。线程出错是只有`errno`。
    > 所以：`fprintf(stderr,"XXXX error:%s\n",strerror(ret)`

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>

void fun(void * arg)
{
    int i=(int) arg;
    sleep(i);
    printf("i am %dth pthread,my pid is %d,my tid is %lu\n",i+1,getpid(),pthread_self());
    return NULL;
}


int main()
{
    pthread_t tid;
    for(int i=0;i<5;i++)
    {
        int res=pthread_create(&tid,NULL,fun,(void*)i);
        if(res!=0)
        {
            fprintf(stderr,"pthread create error:%s\n",strerror(res));
            exit(1);
        }
    }
    sleep(5);
    printf("i am main,pid=%d,tid=%lu\n",getpid(),pthread_self());
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/ea4dbad0620343108a8fabdad5b7fc7c.png#pic_center)

> **编译时会出现类型强转的警告，指针4字节转int的8字节，不过不存在精度损失，忽略就行。**

---

## 退出线程

### `pthread_exit()`

- 功能：退出当前线程
- 参数：退出值，无退出值时传`NULL`
  
  ### 区分
- `exit();`:退出当前进程
- `return `:返回到函数调用者那里
- `pthread_exit()`:退出当前线程

### 举个栗子

为了更好的说明三种方式的区别，我们还是用上面创建五个子线程的例子，只不过我们要退出第三个子线程。我们分别用这三种方式来试试。

#### exit

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>
#include<errno.h>

            //the first way,use exit.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(i==2)
        exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}


/*             the second way,use pthread_exit().
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        pthread_exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

/*            the third way,use return.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        return NULL;
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

int main()
{
    pthread_t tid;
    for(int i=0;i<5;i++)
    {
        int res=pthread_create(&tid,NULL,fun,(void*)i);
        if(res!=0)
        {
            fprintf(stderr,"pthread_create error %s\n",strerror(errno));
            exit(1);
        }
    }
    sleep(5);
    printf("i am main,my pid is %d,tid=%lu\n",getpid(),pthread_self());
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/52a745d8afb94bb5817a8967e57166a8.png#pic_center)

#### pthread_exit()

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>
#include<errno.h>

/*            //the first way,use exit.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(i==2)
        exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

             //the second way,use pthread_exit().
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        pthread_exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}


/*            the third way,use return.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        return NULL;
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

int main()
{
    pthread_t tid;
    for(int i=0;i<5;i++)
    {
        int res=pthread_create(&tid,NULL,fun,(void*)i);
        if(res!=0)
        {
            fprintf(stderr,"pthread_create error %s\n",strerror(errno));
            exit(1);
        }
    }
    sleep(5);
    printf("i am main,my pid is %d,tid=%lu\n",getpid(),pthread_self());
    return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/d23b03f3ebf84db4af061fa1d861b0ea.png#pic_center)

#### return

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>
#include<errno.h>

/*            //the first way,use exit.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(i==2)
        exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

/*             //the second way,use pthread_exit().
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        pthread_exit(1);
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}
*/

            //the third way,use return.
void fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    if(2==i)
        return NULL;
    printf("i am %dth pthread,my pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    return NULL;
}


int main()
{
    pthread_t tid;
    for(int i=0;i<5;i++)
    {
        int res=pthread_create(&tid,NULL,fun,(void*)i);
        if(res!=0)
        {
            fprintf(stderr,"pthread_create error %s\n",strerror(errno));
            exit(1);
        }
    }
    sleep(5);
    printf("i am main,my pid is %d,tid=%lu\n",getpid(),pthread_self());
    return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/36111a97d64645f2992cf2118a85f31f.png#pic_center)

---

## 回收子线程

`int pthread_join(pthread_t tid,void** retval);`

- 功能：阻塞，回收子线程
- 参数：
  - `tid`:待回收的子线程id
  - `retval`:传出参数，回收线程的退出值。
- 返回值：
  - 成功，0
  - 失败，返回错误号

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
struct stu
{
    int age;
    char mesg[100];
};

void* fun(void* arg)
{
    struct stu* t;
    t=malloc(sizeof t);
    t->age=19;
    strcpy(t->mesg,"hellow linux\n");
    return (void*)t;
}

int main()
{
    pthread_t tid;
    pthread_create(&tid,NULL,fun,NULL);
    struct stu *temp;
    pthread_join(tid,(void**)&temp);
    printf("child pthread exit with age=%d,mesg=%s\n",temp->age,temp->mesg);
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/2c85037947cd4e9e99773e05a9ae7cf4.png#pic_center)

### 综合练习

我们利用`pthread_join`循环回收创建的5子线程
源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>
#include<errno.h>

void* fun(void* arg)
{
    int i=(int) arg;
    sleep(i);
    printf("i am %dth pthread\n",i+1);
    return NULL;
}

int main()
{
    pthread_t tid[5];
    for(int i=0;i<5;i++)
    {
        int ret=pthread_create(&tid[i],NULL,fun,(void*) i);
        if(0!=ret)
        {
            fprintf(stderr,"pthread_create error:%s\n",strerror(ret));
            exit(0);
        }
    }
    for(int i=0;i<5;i++)
    {
        pthread_join(tid[i],NULL);
        printf("i am main pthread,i join %dth pthread\n",i+1);
    }
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/1774f850227743dab03ea050b0a05982.png#pic_center)

---

## 杀死线程

### 函数

`int pthread_cancel(pthread_t tid);`

- 功能： 杀死一个线程。（需要达到取消点）
- 参数：`tid`待杀死的线程
- 返回值：
  - 成功，0
  - 失败，返回错误号

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>


void* fun(void* arg)
{
    while(1)
    {
        printf("i am child pthread\n");        //this is a normal case
        sleep(1);

        //pthread_testcancel();            //this is to set cancel point
    }
    return (void*) 666;
}


int main()
{
    pthread_t tid;
    void* res=NULL;

    pthread_create(&tid,NULL,fun,NULL);
    sleep(3);
    pthread_cancel(tid);
    pthread_join(tid,&res);
    printf("child pthread has been killed me,and it exit with %d\n",(int) res);
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/da7265debaf94e63942cbe0bd5d97cab.png#pic_center)

> 这里要注意一点，`pthread_cancel`工作的**必要条件是进入内核**，如果`fun`真的离谱到没有进入内核`pthread_cancel`不能杀死线程，此时**需要手动设置取消点**，就是`pthread_testcancel()`
> 还有一点，如果子线程被`pthread_cancel()`杀死，无论函数里写的是返回是什么，**最后都是返回`-1`.**

---

## 线程分离

### 函数

`int pthread_detach(pthread_t tid);`

- 功能：设置线程分离
- 参数：`tid`,待分离的线程id
- 返回值：
  - 成功，0
  - 失败，返回错误号

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

void* fun(void* arg)
{
     printf("i am child pthread\n");
     return NULL;
}

int main()
{
    pthread_t tid;

    pthread_create(&tid,NULL,fun,NULL);
    pthread_detach(tid);
    sleep(1);
    int res=pthread_join(tid,NULL);
    if(res!=0)
        fprintf(stderr,"pthread join error:%s\n",strerror(res));
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/a65168cf9d0e4ae581ece910985a20f0.png#pic_center)

---

## 线程属性设置分离线程

1. 创建一个线程属性结构变量:`pthread_attr_t attr;`
2. 初始化线程属性:`pthread_attr_init(&attr)`
3. 设置线程属性为分离态:`pthread_attr_setdetachstate(&attr,PTHREAD_CREATE_DETACHED);`
4. 借助修改后的 设置线程属性 创建为分离态的新线程:`pthread_create(&tid,&attr,fun,NULL)`

---

## 线程和进程的比较

| **进程**             | **线程**                  |
|:------------------:|:-----------------------:|
| `fork()`           | `pthread_create()`      |
| `wait()/waitpid()` | `pthread_join()`        |
| `kill`             | `pthread_cancel()`      |
| `getpid`           | `pthread_self()`        |
| `exit`             | `pthread_exit()/return` |
|                    | `pthread_detach()`      |

---

## 线程使用注意事项

1. 主线程退出其他线程不退出，主线程应该调用`pthread_exit`
2. 避免僵尸线程
   - `pthread_join`
   - `thread_detach`
   - `pthread_create`指定分离属性
   - 被`join`线程可能在`join`函数返回前就释放自己的所有内存资源，所以不应当返回被回收线程栈中的值
3. `malloc`和`mmap`申请的内存可以被其他线程释放
4. 应避免在多线程中调用`fork`，除非马`exec`，子线程中只有调用`fork`的线程存在，其他线程在子进程中均`pthread_exit`
5. 信号的复杂语义很难和多线程共存，在多线程中避免使用信号机制

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。毕竟“**实践出真知**”。

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
>    - [Linux学习之系统编程5(信号）](https://blog.csdn.net/yourgrandfather_/article/details/135408188)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
