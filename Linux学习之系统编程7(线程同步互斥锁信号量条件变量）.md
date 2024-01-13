> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## 线程同步

### 概念：

协同步调，对公共区数据按序访问，防止数据混乱，产生与时间有关的错误。

### 数据混乱的原因：

- 资源共享（独享资源则不会）
- 调度随机（意味着数据访问会出现竞争）
- 线程间缺乏必要同步机制

### 解决方法

使用锁。建议锁！对公共数据进行保护。所有线程应该在访问公共数据前先拿锁在访问，但锁本身不具备强制性。

> 这段话可能有点绕，我在这里稍微解释一下。正确使用锁可以保证**线程同步**，但你也可以**不使用锁直接去访问公共数据**，也可以访问到，但这样就不能保证线程同步，就是说**程序本身并不能强制你使用锁。**

### 数据混乱的演示

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

pthread_mutex_t mutex;

void sys_err(char* str,int errno)
{
    fprintf(stderr,"%s:%s",str,strerror(errno));
    exit(-1);
}

void* fun(void* arg)
{
    while(1)
    {
        printf("hellow ");
        sleep(rand()%3);
        printf("world\n");
        sleep(rand()%3);
    }
    return NULL;
}


int main()
{
    pthread_t tid;
    srand(time(NULL));
    int res=pthread_mutex_init(&mutex,NULL);
    if(res!=0)
        sys_err("init error",res);
    res=pthread_create(&tid,NULL,fun,NULL);
    if(res!=0)
        sys_err("pthread create error",res);
    while(1)
    {
        printf("HELLOW ");
        sleep(rand()%3);
        printf("WORLD\n");
        sleep(rand()%3);
    }
    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/ca563c86c8e446319cb798d1c6efe441.png#pic_center)

> 我们可以看到，我们的本意是让大写的“HELLOW WORLD”和小写的“hellow world”在一行输出，这里的公共资源是屏幕`STDOUT_FILENO`，但是由于我们让每一个线程打印完上一句话就睡一下，这样就会导致线程混乱。

## 借助互斥锁实现线程同步

### 相关函数的介绍

- `pthread_mutex_t mutex`这个不是函数，是一个类型。是后面函数都会用到的参数。
- `int pthread_mutex_init(pthread_mutex_t* restrict mutex,const pthread_mutexattr_t* restrict attr);`创建
- `int pthread_mutex_destory(pthread_mutex* mutex);`销毁
- `int pthread_mutex_lock(pthread_mutex_t *mutex);`上锁
- `int pthread_mutex_trylock(pthread_mutex_t *mutex);`尝试上锁
- `int pthread_mutex_unlock(pthread_mutex_t *mutex);`解锁

> **restrict（关键字）**: 用来限定指针变量。被该关键字限定的指针变量所指向的内存操作，必须由本指针完成。

> `pthread_mutex_t `类型，其**本质是一个结构体**。为简化理解，应用时可忽略其实现细节，**简单当成整数看待**`pthread_mutex_t mutex`；变量`mutex`只有两种取值：0,1

### 使用锁（互斥量，互斥锁）的一般步骤

1. `pthread_mutex_t lock;`  创建锁
2. `pthread_mutex_init;` 初始化
3. `pthread_mutex_lock;`加锁    
4. 访问共享数据
5. `pthrad_mutext_unlock();`解锁
6. `pthead_mutex_destroy；`销毁锁

### 初始化互斥量

1. 动态初始化：`pthread_mutex_init(&mutex,NULL);`
2. 静态初始化：`pthread_mutex_t mutex=PTHREAD_MUTEX_INITIALIZER`    

### 举个栗子

我们还是实现上面的功能，只不过这次我们加锁，实现大写的在一行，小写的在一行。
源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

pthread_mutex_t mutex;

void sys_err(char* str,int errno)
{
    fprintf(stderr,"%s:%s\n",str,strerror(errno));
    exit(-1);
}

void* fun(void* arg)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);
        printf("hellow ");
        sleep(rand()%3);
        printf("world\n");
        pthread_mutex_unlock(&mutex);
        sleep(rand()%3);
    }
    return NULL;
}


int main()
{
    pthread_t tid;
    srand(time(NULL));
    int res=pthread_mutex_init(&mutex,NULL);
    if(res!=0)
        sys_err("mutex init",res);
    res=pthread_mutex_init(&mutex,NULL);
    if(res!=0)
        sys_err("init error",res);
    res=pthread_create(&tid,NULL,fun,NULL);
    if(res!=0)
        sys_err("pthread create error",res);
    while(1)
    {
        pthread_mutex_lock(&mutex);
        printf("HELLOW ");
        sleep(rand()%3);
        printf("WORLD\n");
        pthread_mutex_unlock(&mutex);
        sleep(rand()%3);
    }
    return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/0847bd8034704dbaa6ec1be39964b53a.png#pic_center)

### 使用技巧

注意事项

- 尽量保证锁的粒度， 越小越好。（访问共享数据前，加锁。访问结束**立即解锁**。）
- 互斥锁，本质是结构体。 我们可以看成整数。 初值为 1。（`pthread_mutex_init() `函数调用成功）

技巧

- 加锁： --操作， 阻塞线程。
- 解锁： ++操作， 唤醒阻塞在锁上的线程。
- try锁：尝试加锁，成功--。失败，返回。同时设置错误号 `EBUSY`

### 两种死锁

1. 对一个锁反复加锁
   源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

pthread_mutex_t mutex;

int main()
{
    pthread_t tid;
    pthread_mutex_init(&mutex,NULL);
    pthread_mutex_lock(&mutex);
    pthread_mutex_lock(&mutex);
    printf("hello linux\n");
    return 0;
}
```

效果就是光标一直在闪，程序一直阻塞在那里。

2. 两个线程，各自持有一把锁，请求另一把锁
   源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

pthread_mutex_t mutex1;
pthread_mutex_t mutex2;

void* fun(void* arg)
{
    pthread_mutex_lock(&mutex2);
    sleep(1);
    printf("hello linux\n");
    pthread_mutex_lock(&mutex1);
    return NULL;
}

int main()
{
    pthread_t tid;
    pthread_mutex_init(&mutex1,NULL);
    pthread_mutex_init(&mutex2,NULL);
    pthread_create(&tid,NULL,fun,NULL);
    pthread_mutex_lock(&mutex1);
    sleep(1);
    printf("HELLO LINUX\n");
    pthread_mutex_lock(&mutex2);    
    printf("hello linux\n");
    return 0;
}
```

效果和上面一种死锁一样，光标一直在那闪。

---

## 读写锁

### 原理

- 锁只有一把。以读方式给数据加锁——读锁。以写方式给数据加锁——写锁。
- **读共享，写独占。**
- **写锁优先级高。**
- 相较于互斥量而言，当读线程多的时候，提高访问效率。

### 相关函数

- `pthread_rwlock_t rwlock;`这不是函数，是个类型。
- `pthread_rwlock_init(&rwlock, NULL);`:创建
- `pthread_rwlock_rdlock(&rwlock);`：加读锁
- `pthread_rwlock_wrlock(&rwlock);`：加写锁
- `pthread_rwlock_unlock(&rwlock);`：解锁
- `pthread_rwlock_destroy(&rwlock);`销毁锁

### 举个栗子

由于这个读写不是重点，我就不亲自写了。
源代码：
![1](https://img-blog.csdnimg.cn/img_convert/e5bb3b04533ca5bb3bd048cccc7b838f.png#pic_center)
效果：
![2](https://img-blog.csdnimg.cn/img_convert/85781c5e56ebbe11999a53a88e457894.png#pic_center)

---

## 条件变量

条件变量不是锁，但是通常结合锁来使用。

### 相关函数

- `pthread_cond_t cond;`这不是函数，是一个类型
- `pthread_cond_init();`：创建条件变量
- `pthread_cond_destroy();`：销毁条件变量
- `pthread_cond_wait();`：阻塞等待条件
- `pthread_cond_timewait();`：有时限的阻塞等待条件
- `pthread-cond_signal();`：至少通知一个阻塞等待的条件变量
- `pthread-cond_broadcast();`通知所有阻塞等待的条件变量

### 初始化条件变量

1. 动态初始化：`pthread_cond_init(&cond,NULL);`
2. 静态初始化：`pthread_cond_t cond=PTHREAD_COND_INITIALIZER;`
   
   ### 重点函数讲解
   
   `pthread_cond_wait(&cond,&mutex)`
   作用
3. 阻塞等待条件变量满足
4. 解锁已经加锁成功的信号量 （相当于` pthread_mutex_unlock(&mutex)`），1,2两步为一个原子操作.
5. 当条件满足，函数返回时，解除阻塞并重新申请获取互斥锁。重新加锁信号量 （相当于`pthread_mutex_lock(&mutex);`）

---

## 条件变量的生产者和消费者模型

### 思路：

![1](https://img-blog.csdnimg.cn/direct/5092f4437b144fd5b3697f7355f28aa8.png#pic_center)

### 单个消费者模型

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

struct mesg
{
    int num;
    struct mesg* next;
};

struct mesg* head;

pthread_mutex_t mutex=PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond=PTHREAD_COND_INITIALIZER;

void sys_err(char* str,int errno)
{
    fprintf(stderr,"%s:%s\n",str,strerror(errno));
    exit(-1);
}

void* produser(void* arg)
{
    while(1)
    {
        struct mesg* m=malloc(sizeof (struct mesg));
        m->num=rand()%1000+1;
        printf("---produce:%d\n",m->num);
        pthread_mutex_lock(&mutex);
        m->next=head;
        head=m;
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond);
        sleep(rand()%3);
    }
    return NULL;
}

void* consumer(void* arg)
{
    while(1)
    {
        struct mesg* m;
        pthread_mutex_lock(&mutex);
        if(head==NULL)
            pthread_cond_wait(&cond,&mutex);
        m=head;
        head=m->next;
        pthread_mutex_unlock(&mutex);
        printf("----------consume:%d\n",m->num);
        free(m);
        sleep(rand()%3);
    }
    return NULL;
}



int main()
{
    pthread_t tid_pro,tid_con;
    srand(time(NULL));
    int res=pthread_create(&tid_pro,NULL,produser,NULL);
    if(res!=0)
        sys_err("pthread create error",res);
    res=pthread_create(&tid_con,NULL,consumer,NULL);
    if(res!=0)
        sys_err("pthread create error",res);
    pthread_detach(tid_pro);    //if use this way,must use pthread_exit to exit main pthread.
    pthread_detach(tid_con);
//    pthread_join(tid_pro,NULL);
//    pthread_join(tid_con,NULL);
//    return 0;
    pthread_exit(0);
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/a02ff5edaac64254be027fab6db26f45.png#pic_center)

### 多个消费者模型

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>

struct mesg
{
    int num;
    struct mesg* next;
};

struct mesg* head;

pthread_mutex_t mutex=PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond=PTHREAD_COND_INITIALIZER;

void sys_err(char* str,int errno)
{
    fprintf(stderr,"%s:%s\n",str,strerror(errno));
    exit(-1);
}

void* produser(void* arg)
{
    while(1)
    {
        struct mesg* m=malloc(sizeof (struct mesg));
        m->num=rand()%1000+1;
        printf("---produce:%d\n",m->num);
        pthread_mutex_lock(&mutex);
        m->next=head;
        head=m;
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond);
        sleep(rand()%3);
    }
    return NULL;
}

void* consumer(void* arg)
{
    int i=(int) arg;
    while(1)
    {
        struct mesg* m;
        pthread_mutex_lock(&mutex);
        while(head==NULL)
            pthread_cond_wait(&cond,&mutex);
        m=head;
        head=m->next;
        pthread_mutex_unlock(&mutex);
        printf("%dth consumer----------consume:%d\n",i,m->num);
        free(m);
        sleep(rand()%3);
    }
    return NULL;
}



int main()
{
    pthread_t tid_pro,tid_con1,tid_con2,tid_con3;
    srand(time(NULL));
    int res=pthread_create(&tid_pro,NULL,produser,NULL);
    if(res!=0)
        sys_err("pthread create error",res);
    res=pthread_create(&tid_con1,NULL,consumer,(void*)1);
    if(res!=0)
        sys_err("pthread create error",res);
    res=pthread_create(&tid_con2,NULL,consumer,(void*)2);
    if(res!=0)
        sys_err("pthread create error",res);
    res=pthread_create(&tid_con3,NULL,consumer,(void*)3);
    if(res!=0)
        sys_err("pthread create error",res);
//    pthread_detach(tid_pro);    
//    pthread_detach(tid_con);
    pthread_join(tid_pro,NULL);
    pthread_join(tid_con1,NULL);
    pthread_join(tid_con2,NULL);
    pthread_join(tid_con3,NULL);
    return 0;
}
```

效果：
![3](https://img-blog.csdnimg.cn/direct/288dc919c4464e179f900f729ca11b7f.png#pic_center)

> **注意事项**
> 我们在多个消费者模型一定在消费者线程中把等待阻塞的判断从`if`改成`while`.
> 
> 1. 两个消费者都阻塞在条件变量上，就是说没有数据可以消费。
> 2. 完事儿都把锁还回去了，生产者此时生产了一个数据，会同时唤醒两个因条件变量阻塞的消费者，完事儿两个消费者去抢锁。
> 3. 结果就是A消费者拿到锁，开始消费数据，B消费者阻塞在锁上（如下图）。
> 4. 之后A消费完数据，把锁归还，B被唤醒，然而此时已经没有数据供B消费了。
> 5. 所以这里有个逻辑错误，消费者阻塞在条件变量那里应该使用while循环。这样A消费完数据后，B做的第一件事不是去拿锁，而是判定条件变量。
>    ![4](https://img-blog.csdnimg.cn/img_convert/c4057760a3166dc1f6ab31011bc7a759.png#pic_center)
>    **实测，如果不改的话就会发生段错误。**

---

## 信号量实现生产者和消费者模型

### 信号量的作用

- 应用于线程、进程间同步。
- 相当于 初始化值为 N 的互斥量。  N值，表示可以同时访问共享数据区的线程数。

### 相关的函数

- `sem_t sem`这不是函数，是类型。
- `int sem_init(sem_t *sem, int pshared, unsigned int value);`创建信号量
- `sem_destroy();`:销毁信号量
- `sem_wait();`:一次调用，做一次-- 操作， 当信号量的值为 0 时，再次 -- 就会阻塞。 （对比 `pthread_mutex_lock`）
- `sem_post();`:一次调用，做一次++ 操作. 当信号量的值为 N 时, 再次 ++ 就会阻塞。（对比 `pthread_mutex_unlock`）

### 重要函数

`int sem_init(sem_t *sem, int pshared, unsigned int value);`
参数：

- `sem`:信号量
- `pshared`:0表示线程同步，1表示进程同步
- `value`:N值。（指定同时访问的线程数）

### 模型

源代码：

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<pthread.h>
#include<semaphore.h>

#define MAX 5
sem_t blank,product;
int queue[MAX];

void* producer(void* arg)
{
    int i=0;
    while(1)
    {
        sem_wait(&blank);
        queue[i]=rand()%1000+1;
        sem_post(&product);
        printf("-------------producer:%d\n",queue[i]);
        i=(i+1)%MAX;
        sleep(rand()%2);
    }
    return NULL;
}

void* consumer(void* arg)
{
    int i=0;
    while(1)
    {
        sem_wait(&product);
        printf("---consumer:%d\n",queue[i]);
        sem_post(&blank);
        i=(i+1)%MAX;
        sleep(rand()%2);
    }
    return NULL;
}


int main()
{
    pthread_t ctid,ptid;

    srand(time(NULL));

    sem_init(&blank,0,MAX);
    sem_init(&product,0,0);        //why set 0?This is mean 0 pthread visit in the same time?

    pthread_create(&ctid,NULL,producer,NULL);
    pthread_create(&ptid,NULL,consumer,NULL);

    pthread_join(ctid,NULL);
    pthread_join(ptid,NULL);

    sem_destroy(&blank);
    sem_destroy(&product);

    return 0;
}
```

效果：
![1](https://img-blog.csdnimg.cn/direct/ca536009bfa64647909fa14d1ec3a47c.png#pic_center)

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。毕竟“**实践出真知**”。

---

> 如果你觉得我写的题解还不错的，请各位**王子公主**移步到我的其他题解看看
> 
> 1. **数据结构与算法部分（还在更新中）：**
>    - [***C++ STL总结 - 基于算法竞赛（强力推荐***）](https://blog.csdn.net/yourgrandfather_/article/details/135051716?spm=1001.2014.3001.5501)
>    - [动态规划——01背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135103012?spm=1001.2014.3001.5501)
>    - [动态规划——完全背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135111459)
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
>    - [Linux学习之系统编程6(线程）](https://blog.csdn.net/yourgrandfather_/article/details/135438595)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
