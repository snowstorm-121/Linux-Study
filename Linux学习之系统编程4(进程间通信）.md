> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## 进程间通信

 ![1](https://img-blog.csdnimg.cn/direct/604d8ca8d34041a78a4ca08e4511a30a.png#pic_center) IPC（Interprocess Communication)进程间通信
进程间通信的常用方式，特征：

- 管道：简单
- 信号：开销小
- mmap映射：非血缘关系进程通信
- socket（本地套接字）：稳定

---

## 管道(pipe)通信

### 实现原理：内核借助环形队列机制，使用内核缓冲区实现。

### 特质：

- 伪文件
- 管道中的数据只能一次读取
- 数据在管道中只能单向流动

### 局限性：

- 自己写，不能自己读。
- 数据不能反复读取。
- 半双工通信
- 仅限于血缘关系进程使用

### 基本用法

`pipe()`函数，创建并打开管道。
`int pipe(int fd[2]);`
参数：

- `fd[0]`:读端
- `fd[1]`: 写端

返回值：

- 成功，0
- 失败，-1，`errno`被设置

管道通信原理（图)：
![1](https://img-blog.csdnimg.cn/direct/5fe98fe878994c89acd018dfefb44953.png#pic_center)

### 举个栗子

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<string.h>

int main()
{
    int fd[2];
    char p[30]="This a test about pipe\n";
    char buf[30];
    pid_t pid;
    pipe(fd);
    pid=fork();
    if(pid==0)
    {
        close(fd[1]);
        read(fd[0],buf,sizeof buf);
        printf("%s",buf);
    }
    else 
    {
        close(fd[0]);
        printf("I am parent,i will write something to mychild\n");
        write(fd[1],p,strlen(p));
    }
    return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/903d2e9f5e3640c9a66479e756af85a6.png#pic_center)

### 管道的读写行为

- 读管道
  - 管道有数据，`read`读取数据，返回实际读到的字节数。
  - 管道没有数据：
    - 没有写端，`read`返回0（类似读到文件的末尾）
    - 有写端，`read`阻塞等待。
- 写管道
  - 没有读端，异常终止。
  - 有读端：
    - 管道未满，往管道里写数据，返回实际写入的字节数。
    - 管道已满，阻塞等待。

### 综合练习

我们使用管道通信实现父子进程`ls | wc -l`功能
思路分析：

1. 我们先让父进程写，子进程读（父进程读，子进程写也可以）
2. 对于父进程，我们可以用之前学的函数`execlp`，但是`ls`命令输出到屏幕 ，我们又想到之前学的函数`dup2`重定向，我们可以把`STDOUT_FILENO`重定向到管道的写端。
3. 对于子进程，我们也可以用之前学的函数`execlp`,但是`wc`接受的命令是来自屏幕，我们又想到之前的学的函数`duo2`重定向，把`STDIN_FILENO`重定向到管道的读端。

源代码：

```c
#include<stdio.h>
#include<unistd.h>

int main()
{
        int fd[2];
        pipe(fd);
        pid_t pid;
        pid= fork();
        if(pid==0)
        {
                close(fd[0]);
                dup2(fd[1],STDOUT_FILENO);
                execlp("ls","ls",NULL);
                perror("child error");
        }
        else
        {
                close(fd[1]);
                dup2(fd[0],STDIN_FILENO);
                execlp("wc","wc","-l",NULL);
                perror("parent error");
        }
        return 0;
}
```

效果：
![3](https://img-blog.csdnimg.cn/direct/556cc2c2101944a3839596d9e788e780.png#pic_center)

### 兄弟间通信

 我们用一个父进程创建两个子进程，用这两个子进程来实现上面的功能`ls | wc -l`
 源代码：

```c
#include<stdio.h>
#include<unistd.h>

int main()
{
    int fd[2],i;
    pipe(fd);
    for(i=0;i<2;i++)
    {
        if(fork())
            break;
    }

    if(2==i)
    {
        close(fd[1]);    
        close(fd[0]);
        wait(NULL);
        wait(NULL);
        printf("I am parent,i wait two children successfully\n");
    }
    else if(0==i)
    {
        close(fd[0]);
        dup2(fd[1],STDOUT_FILENO);
        execlp("ls","ls",NULL);
        perror("1th child error");
    }
    else if(1==i)
    {
        close(fd[1]);
        dup2(fd[0],STDIN_FILENO);
        execlp("wc","wc","-l",NULL);
        perror("2th child error");
    }
    return 0;
}
```

效果：
![4](https://img-blog.csdnimg.cn/direct/385705b599b84890afa64e9c0e1a069b.png#pic_center)

> **这里唯一要注意的是我们用兄弟通信时，要把父进程的读端和写端都关闭**

> 一个pipe可以有一个写端多个读端
> 一个pipe可以有多个写端一个读端

> 管道的默认大小是4096（4k)
> ![5](https://img-blog.csdnimg.cn/direct/de62b99f9ff94d5badcd6a7ed8e86b66.png#pic_center)

---

## 命令管道（fifo)通信

### 优管道的优缺点

 优点：

- 简单，相比信号，套接字实现进程通信，简单很多
  缺点：
  - 只能单向通信，双向通信需建立两个管道
  - 只能用于有血缘关系的进程间通信。该问题后来使用fifo命名管道解决。

fifo管道：可以用于无血缘关系的进程间通信。fifo操作起来像文件

### mkfifo函数

![1](https://img-blog.csdnimg.cn/direct/a0d555d3d3f64db893be4b0581958964.png#pic_center)
这个函数和`open`差不多，只不过创建的文件类型不同罢了。
返回值：

- 成功，0
- 失败，-1，`errno`被设置

### fifo实现非血缘关系进程间通信

#### 思路：

1. 我们打开两个没有血缘关系的进程，一个负责写（我们命令为`fifo_w`，一个负责读（我们命令为`fifo,r`)。同时使用`mkfifio`创建一个命名管道（我们命名为`fifo_test`)。
2. 对于`fifo_w`，我们打开管道的写端`fd=open("fifo_test",O_WRONLY)`，接下来操作和文件一样，使用`write`向`fifo_test`里写数据。
3. 对于`fifo_r`，我们打开管道的读端`fd=open("fifo_test",O_RDONLY)`，接下来操作和文件一样，使用`read`向`fifo_test`里读数据。

#### 实际演示

##### 源代码：

`fifo_w`部分：

```c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<fcntl.h>

int main()
{
    char p[100]="This is a test about fifo";
    char buf[100];
    int cnt=0;
    int fd=open("fifo_test",O_WRONLY);
    if(fd==-1)
        perror("open file error");
    while(1)
    {
        sprintf(buf,"%s---%d",p,cnt++);
        write(fd,buf,strlen(buf));
        sleep(1);
    }
    return 0;
}
```

`fifo_r`部分：

```c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>

int main()
{
    int fd=open("fifo_test",O_RDONLY);
    if(fd==-1)
        perror("open file error");
    while(1)
    {
        int res;
        char buf[100];
        res=read(fd,buf,sizeof buf);
        if(res<0)
            perror("read error");
        printf("%s\n",buf);
        sleep(1);
    }
    return 0;
}
```

##### 效果：

![2](https://img-blog.csdnimg.cn/direct/8e45472c790f4503b496c8eba59cce5f.png#pic_center)

---

## 文件用于进程间通信

### 原理：

打开的文件是内核中的一块缓冲区。多个无血缘关系的进程，可以同时访问该文件。
![1](https://img-blog.csdnimg.cn/direct/adba51c8bf0342c6b9289d471eeb7db1.png#pic_center)

### 总结：

- 只是有血缘关系的进程对于同一个文件，使用的同一个文件描述符。
- 没有血缘关系的进程，对同一个文件使用的文件描述符可能不同。
- 这些都不是问题，打开的是同一个文件就行。

---

## mmap函数

### 存储映射I/O(Memory-mapped I/O)

- 使一个磁盘文件与存储空间中的一个缓冲区相映射。于是从缓冲区中取数据，就相当于读文件中的相应字节。
- 与此类似，将数据存入缓冲区，则相应的字节就自动写入文件。这样，就可在不使用read和write函数的情况下，使地址指针完成I/O操作。
- 使用这种方法，首先应该通知内核，将一个指定文件映射到存储区域中。这个映射工作可以通过mmap函数来实现。

### 函数解读

![1](https://img-blog.csdnimg.cn/direct/52f592edbc04480c84d020c39864cad8.png#pic_center)
`void *mmap(void *addr,size_t length,int prot,int flags,int fd,off_t offset`

#### 参数

- `addr`:指定映射区的首地址。通常传`NULL`，表示让系统自动分配
- `length`:共享内存映射区的大小。（`<= `文件的实际大小）
- `prot`:共享内存映射区的读写属性。
  - `PROT_READ`：读
  - `PROT_WRITE`：写
  - `PROT_READ|PROT_WRITE`：读/写
- `flags`:标注共享内存的共享属性。
  - `MAP_SHARED` 修改会反映到磁盘上。
  - `MAP_PRIVATE` 修改不反映到磁盘上。   
- `fd`:用于创建共享内存映射区的那个文件的 文件描述符。
- `offset`:默认0，表示映射文件全部。偏移位置。需是 `4k 的整数倍`。

#### 返回值：

- 成功，映射区的首地址
- 失败，返回宏`MAP_FAILED`,其实就是`void*`类型的`0`.

### munmap函数

`int munmap(void *addr, size_t length);`
作用：释放映射区
参数传的一般和`mmap`一样即可（这样肯定不会错）。

### 举个栗子

我们手动传入一个参数（表示映射的文件名），然后建立映射区，通过映射区的首地址来读写。
源代码：

```c
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/mman.h>
#include<fcntl.h>

int main(int argc,char* argv[])
{
    if(argc==1)
    {
        printf("argument error\n");
        return -1;
    }
    int fd=open(argv[1],O_RDWR| O_CREAT | O_TRUNC,0644);
    ftruncate(fd,100);
    int len=lseek(fd,0,SEEK_END);
    char* ret=mmap(NULL,len,PROT_READ | PROT_WRITE,MAP_SHARED,fd,0);
    if(ret==MAP_FAILED)
        perror("mmap error");
    char p[100]="This is a test about mmap\n";
    memcpy(ret,p,strlen(p));
    printf("%s",ret);
    close(fd);
    munmap(ret,len);
    return 0;
}    
```

效果：
![2](https://img-blog.csdnimg.cn/direct/089fc266c3d34ad7b1aec110616a987c.png#pic_center)

### 注意事项

1. 用于创建映射区的文件大小为 0，实际指定非0大小创建映射区，出 “总线错误”。
2. 用于创建映射区的文件大小为 0，实际制定0大小创建映射区， 出 “无效参数”。
3. 用于创建映射区的文件读写属性为，只读。映射区属性为 读、写。 出 “无效参数”。
4. 创建映射区，需要read权限。当访问权限指定为 “共享”MAP_SHARED时， mmap的读写权限，应该 <=文件的open权限。    只写不行
5. 文件描述符fd，在mmap创建映射区完成即可关闭。后续访问文件，用 地址访问。
6. offset 必须是 4096的整数倍。（MMU 映射的最小单位 4k ）
7. 对申请的映射区内存，不能越界访问。 
8. munmap用于释放的 地址，必须是mmap申请返回的地址。
9. 映射区访问权限为 “私有”MAP_PRIVATE, 对内存所做的所有修改，只在内存有效，不会反应到物理磁盘上。
10. 映射区访问权限为 “私有”MAP_PRIVATE, 只需要open文件时，有读权限，用于创建映射区即可。 

### 保险用法

1. `fd = open（"文件名"， O_RDWR）;`
2. `mmap(NULL, 有效文件大小， PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);`

### 进阶练习——无血缘关系进程间mmap通信

`mmap_w`部分：

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/mman.h>
#include<fcntl.h>
#include<string.h>

int main()
{
    int fd=open("mmap_test",O_RDWR|O_TRUNC);
    ftruncate(fd,100);
    int len =lseek(fd,0,SEEK_END);
    char* p=mmap(NULL,len,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if(p==MAP_FAILED)
        perror("mmap error");
    scanf("%s",p);
    close(fd);
    munmap(p,len);
    return 0;
}
```

`mmap_r`部分：

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/mman.h>
#include<fcntl.h>
#include<string.h>

int main()
{
    sleep(10);
    int fd=open("mmap_test",O_RDWR);
    int len =lseek(fd,0,SEEK_END);
    char* p=mmap(NULL,len,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if(p==MAP_FAILED)
        perror("mmap error");
    printf("%s",p);
    close(fd);
    munmap(p,len);
    return 0;
}
```

效果：
![3](https://img-blog.csdnimg.cn/direct/2a279e185e3a4321913dc55313520640.png#pic_center)

### mmap匿名映射区

匿名映射：只能用于 血缘关系进程间通信。
    `p = (int *)mmap(NULL, 40, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0);`

### 总结

1. 创建映射区的过程中，隐含着一次对映射文件的读操作，所以要求文件必须有读的权限。
2. 当`MAP_SHARED`时，要求：映射区的权限应该`<=`文件打开的权限（出于对映射区的保护）。而`MAP_PRIVATE`则无所谓，因为`mmap`中的权限是对内存的限制.
3. 映射区的释放与文件关闭无关。只要映射建立成功，文件可以立即关闭
4. 特别注意，当映射文件大小为0时，不能创建映射区。所以：用于映射的文件必须要有实际大小！！`mmap`使用时常常会出现总线错误，通常是由于共享文件存储空间大小引起的。如，400字节大小的文件，在建立映射区时，`offset`4096字节，则会报出总线错误.
5. `munmap`传入的地址一定是mmap返回的地址。坚决杜绝指针++操作,即`adrr++`.想要操作，先拷贝一份。
6. 文件偏移量必须为4K的整数倍。没有特殊要求就传`0`.
7. `mmap`创建映射区出错概率非常高，一定要检查返回值，确保映射区建立成功再进行后续操作。

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。正可谓“**键盘敲烂，月薪过万**”

---

> 如果你觉得我写的题解还不错的，请各位**王子公主**移步到我的其他题解看看
> 
> 1. **数据结构与算法部分（还在更新中）：**
>    - [***C++ STL总结 - 基于算法竞赛（强力推荐***）](https://blog.csdn.net/yourgrandfather_/article/details/135051716?spm=1001.2014.3001.5501)
>    - [动态规划——完全背包问题]([动态规划——完全背包问题-CSDN博客](https://blog.csdn.net/yourgrandfather_/article/details/135111459)[动态规划——完全背包问题]([动态规划——完全背包问题-CSDN博客](https://blog.csdn.net/yourgrandfather_/article/details/135111459)
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
>    - [Linux学习之系统编程3（进程及wait函数）](https://blog.csdn.net/yourgrandfather_/article/details/135354402)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
