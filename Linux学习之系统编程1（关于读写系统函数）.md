> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。
> 
> ---
> 
> 好久没更新了，虽然这几天我并没有停止学习Linux，但是时间确实有点不够，写一篇blog还是比较耗时间的。~~（好吧，其实就是我有点懒，明明可以挤出时间，但是还是去划水了）~~ 

---

## open函数

> **当遇到我们不熟悉的函数，我们可以去`manpage`查看，这也是一个非常重要的自主学习手段和方法**

`manpage` 第二卷，直接`man 2 open`就可以查看`open`函数的的详细信息。
![open1](https://img-blog.csdnimg.cn/direct/3dd8d162004147a195ad1259d32b698f.png#pic_center)

### 根据上面的描述，我们可以得知`open`的基本信息

- 要使用`open`需要引头文件`sys/types.h`和`sys/stat.h>`和`fcntl.h`(其中`sys/types.h`和`sys/stat.h>`这两个头文件可以用`unistd.h`来替代。
- 传入的第一个参数类型是`const char*`，看名称就可以猜到是要打开文件的路径，这里用绝对路径和相对路径都可以。
- 传入的第二个参数类型是`int`，这里没有详细写，我就简单介绍，这个`flag`是权限控制，有`O_RDONLY` `O_WRONLY` `O_RDWR`,分别表示只读，只写，读写。（其实和C语言里的库函数`fopen`差不多）
- 对于第二个`open`就是多了个参数`mode_t mode`，就是当文件不存在时，我们创建文件（前面那个`flag`参数还要指定`O_CREAT`才可以创建文件)，指定文件的权限。（文件权限=`mode`&`~umask`)

### 关于返回值

![open2](https://img-blog.csdnimg.cn/direct/270a2d5ce6774a3598ea4e0a6d507403.png#pic_center)
简单来说就是一个文件描述符，用它可以读写文件。这里实在理解不了的，可以把它理解成一个非负整数，用它来操作文件，后面我会详细讲解这个文件描述符。

### 使用`open`常见的错误

- 打开文件不存在
- 以写方式打开只读文件（权限问题）
- 以只写方式打开目录

### 下面我举例演示

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<errno.h>
#include<stdlib.h>

int main()
{
    int fd;
    fd=open("./test.c",O_RDONLY);
    if(fd<0)
    {
        printf("fd=%d,errno=%d:%s",fd,errno,strerror(errno));
        exit(0);
    }
    printf("fd=%d\n",fd);
    close(fd);
    return 0;
}
```

执行结果：
![open3](https://img-blog.csdnimg.cn/direct/bb10f09c980045f7aacd6a28e35bfce8.png#pic_center)

---

## close函数

学会了`open`函数，这个函数就没什么好说的。很简单。直接上图。
![close](https://img-blog.csdnimg.cn/direct/d0aa0a0b690540eeb2d9a65e11bfc96d.png#pic_center)

---

## read函数

还是先查`man 2 read`，直接上图。
![read](https://img-blog.csdnimg.cn/direct/5314c7c372044a8e913dd25ab2c966ed.png#pic_center)
其实这个函数也没啥好说的，和C语言的标库函数差不多。从一个文件中读取。

### 参数解读

- 第一个`fd`就是上面我们在`open`中学的返回值，一个文件描述符，用来操作文件。
- 第二个`buf`，就是缓冲区，用来接收从`fd`文件中读取的内容。
- 第三个`count`，就是从文件中读取的字节数。

### 返回值

- 如果读取成功，就返回读取到的字节数。
- 如果返回0，不代表读取失败，而是表示读取到文件的末尾了。
- 如果读取失败，返回`-1`,并且`errno`被设置。
  
  > **特别提醒**：每次读取文件的**偏移量都会移动相应的读取的字节数**。（原文是`the
  > 
  >      file position is advanced by this number`)

---

## write函数

还是先查`man 2 write`，直接上图。
![write](https://img-blog.csdnimg.cn/direct/07cf57e4b2f046e4b9c3206c548a4922.png#pic_center)
和`read`差不多

### 参数解读

- 第一个`fd`就是上面我们在`open`中学的返回值，一个文件描述符，用来操作文件。
- 第二个`buf`，就是缓冲区，里面是要写入`fd`文件中的内容。
- 第三个`count`，就是从文件中写入的字节数。

### 返回值

- 如果写入成功，就返回写入到的字节数。
- 如果写入失败，返回`-1`,并且`errno`被设置。

> **特别提醒**：要写入文件时，**一定打开相应的权限**，`O_WRONLY`,同时要确保当**前用户有权限向文件写入**（即该用户对该文件有`w`权限）

---

## 利用`read`和`write`来实现`cp`功能

先来说一说思路，我们先打开被复制的文件，用`fd1`来表示它，再创建复制后的文件。用`fd2`来表示。每次都用`read`从`fd1`读取，再直接用`write`写入`fd2`即可。
废话少说，直接上源代码：

```c
#include<unistd.h>
#include<errno.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>

int main(int argc,char* argv[])
{
        int fd1,fd2,n;
        char buf[4096];
        fd1=open(argv[1],O_RDONLY);
        fd2=open(argv[2],O_RDWR | O_CREAT,0664);
        if(fd1<0 || fd2<0)
        {
                printf("errno:%d:%s",errno,strerror(errno));
                exit(1);
        }
        while((n=read(fd1,buf,1024))!=0)
                write(fd2,buf,n);
        close(fd1);
        close(fd2);
        return 0;
}
```

效果如图
![cp](https://img-blog.csdnimg.cn/direct/4c0878c800b34347b514bf7aec2901d0.png#pic_center)

---

## 预读入和缓输出

废话少说，先上图。
![1](https://img-blog.csdnimg.cn/direct/dca2fb5d579848f88b1d120eb2429473.png#pic_center)

---

![2](https://img-blog.csdnimg.cn/direct/5dc85c57558a482d94f787f5e02386d2.png#pic_center)
标准IO函数自带用户缓冲区，系统调用无用户级缓冲。可能上面的图各位童鞋看不懂，没关系，记住“**系统函数并不是一定比库函数牛逼，能使用库函数的地方就使用库函数。**”

---

## 文件描述符

![1](https://img-blog.csdnimg.cn/direct/303238a221a84b21bab475e1251eae24.png#pic_center)
文件描述符的本质是指向文件结构体的指针。
PCB进程控制块：本质 结构体
成员： 文件描述符表。
文件描述符：0/1/2/3/4......./1023 表中可用的最小的。

> **0 - STDIN_FILENO**
> **1 - STDOUT_FILENO**
> **2 - STDERR_FILENO**

---

## lseek函数

还是先查看`man 2 lseek`
![1](https://img-blog.csdnimg.cn/direct/185a60a930264099b6c4d4e7b2fb800b.png#pic_center)

### 参数解读

- `fd`应该很熟悉了吧，传要操作的文件。
- `offset`是偏移量，就是你要文件移动的偏移量。
- `whence`总共有三个值可以设定。`SEEK_SET`表示文件的开头，`SEEK_CUR`表示文件的当前的偏移量，`SEEK_END`表示文件的末尾。

### 返回值

返回值是个整数，表示当前文件的位置和开头的偏移量。

### 注意点

1. **文件的“读”和“写”使用同一偏移量。**
2. 可以使用`lseek`获取大小
3. 使用`lseek`拓展文件大小，但想真正引起文件大小变化，必须IO操作    。（这不是正规军的打法，正规军是应该是`tuncate`来拓展文件大小）

### 利用`lseek`获取文件大小

源代码：

```c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>

int main(int argc,char* argv[])
{
        int fd,res;
        fd=open(argv[1],O_RDONLY);
        if(fd<0)
        {
                perror("open file");
                exit(1);
        }
        res=lseek(fd,0,SEEK_END);
        printf("%s size is %d\n",argv[1],res);
        return 0;
}
```

效果：
![2](https://img-blog.csdnimg.cn/direct/8b3a3c0906fe48928ad02c32af4b1b59.png#pic_center)

---

## 目录项和    inode

![1](https://img-blog.csdnimg.cn/direct/240f58a17e5f4825a6d7ff24cb374740.png#pic_center)

> 一个文件主要由两部分组成，dentry(目录项)和 `inode`
> `inode` 本质是结构体，存储文件的属性信息，如：权限、类型、大小、时间、用户、盘快位置…也叫做文件属性管理结构，大多数的` inode` 都存储在磁盘上。少量常用、近期使用的`inode` 会被缓存到内存中。
> 所谓的删除文件，就是删除`inode`，但是数据其实还是在硬盘上，以后会覆盖掉。

---

## stat 函数

还是先查看`man 2 stat`
![1](https://img-blog.csdnimg.cn/direct/112cb03de1354d8aa85d628b0c5f152e.png#pic_center)

### 参数解读

- `pathname`看名字就知道是文件的路径。
- `statbuf`这个参数类型是`struct stat`，虽然是第一次见，我这里简单介绍一下，包含文件各种信息的结构。

### 返回值

- 成功返回0
- 失败返回-1，`errno`被设置

### 关于`struct stat`

![2](https://img-blog.csdnimg.cn/direct/ae97c9ab0e774cc0ba142180438a26ae.png#pic_center)

### 举例

源代码：

```c
#include<unistd.h>
#include<stdio.h>
#include<fcntl.h>
#include<sys/stat.h>
#include<stdlib.h>

int main(int argc,char* argv[])
{
        struct stat sbf;
        stat(argv[1],&sbf);
        printf("The file size is %ld\n",sbf.st_size);
        if(S_ISREG(sbf.st_mode))
                printf("It is a regular file\n");
        else if (S_ISDIR(sbf.st_mode))
                printf("It is a directory\n");
        else
                printf("unkown file\n");
        return 0;
}
```

结果：
![3](https://img-blog.csdnimg.cn/direct/eb407ee3214b41518be2517f2d23c5a0.png#pic_center)

---

## link函数和unlink函数

![1](https://img-blog.csdnimg.cn/direct/3beea4b274634694ad2d93612a2f988e.png#pic_center)

> 硬链接数就是 `dentry` 数目
> `link` 就是用来创建硬链接的
> `link` 可以用来实现 `mv `命令

删除一个链接` int unlink(const char *pathname)`

> `unlink` 是删除一个文件的目录项 `dentry`，使硬链接数-1
> `unlink` 函数的特征：清除文件时，如果文件的硬链接数到 0 了，没有 `dentry` 对应，但该文件仍不会
> 马上被释放，要等到所有打开文件的进程关闭该文件，系统才会挑时间将该文件释放掉。

**由于这个函数比较简单，我这里就不赘述了。**

---

## 目录操作函数

我这里就介绍三个最常用的函数:`opendir` `readdir` `closedir`
![1](https://img-blog.csdnimg.cn/direct/660bd79fd30f4e9380f335b0ab807abf.png#pic_center)

---

![2](https://img-blog.csdnimg.cn/direct/d3076807500646ac8dc7a529b30147ef.png#pic_center)
接下来我来用这几个函数来模拟实现Linux中的`ls`的功能来让具体体现这几个函数的用法

---

## 实现Linux的`ls`功能

源代码

```c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<sys/stat.h>
#include<string.h>
#include<dirent.h>

void is_file(char*);

void read_dir(char* dir)
{
        char path[256];
        DIR* dp;
        struct dirent* sdp;
        dp=opendir(dir);
        if(dp==NULL)
        {
                perror("opendir error");
                return;
        }
        while((sdp=readdir(dp))!=NULL)
        {
                if(strcmp(".",sdp->d_name)==0 || strcmp("..",sdp->d_name)==0)
                        continue;
                sprintf(path,"%s/%s",dir,sdp->d_name);
                is_file(path);
        }
        closedir(dp);
        return;
}


void is_file(char* name)
{
        int ret;
        struct stat sb;
        ret=stat(name,&sb);
        if(ret==-1)
        {
                perror("stat error");
                return;
        }
}

int main(int argc,char* argv[])
{
        if(argc==1)
                is_file(".");
        else
                is_file(argv[1]);
        return 0;
}
```

效果展示
![3](https://img-blog.csdnimg.cn/direct/b5e52d6c8b3f436595a077158386006e.png#pic_center)

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

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
