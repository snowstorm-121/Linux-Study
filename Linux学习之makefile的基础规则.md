> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

---

## makefile的基础规则

> makefile：项目管理
> 命令：makefile or Makefile          ---make命令

- **一个规则**：
    目标：依赖条件
  
        （一个tab缩进）命令
  1. 目标的时间必须晚于依赖条件，否则，更新目标。
  2. 依赖条件如果不存在，寻找新的规则去生成依赖条件。
- **两个函数**：
  1. `src=$(wildcard ./*.c)`：寻找当前目录下的所有`.c`文件，将文件名组成列表，赋值给变量`src`
  2. `obj=$(patsubst %.c %.o $(src))`: 将参数3中所有包含参数1的部分替换成参数2
- **三个自动变量**：
  1. `$@`:在规则的命令中，表示规则中的目标。
  2. `$^`:在规则的命令中，表示所有的依赖条件。
  3. `$<`:在规则的命令中，表示第一个依赖条件。如果将该变量应用于模式规则中，它可将依赖条件列表中的依赖一次取出，套用模式规则。
- **模式规则**：

```bash
%.o : %.c
    gcc -c $< -o %@
```

- **静态模式规则**：

```bash
$(obj) : %.o : %.c
    gcc -c $< -o %@
```

- 终极目标：
  `ALL : 最终的目标`
- 伪目标：
  `.PHONY: clean ALL`    
- 清除：

```bash
clean : (没有依赖)
    -rm -rf $(obj) a.out
```

`-`的作用的删除不存在的文件不报错，顺序执行结束。

- 一些参数：
  - `-n`:模拟执行`make` `make clean`命令
  - `-f`: 指定文件执行 `make`命令

---

## 上机操作

### 第一版

1. 编写`nakefile`文件
   ![1](https://img-blog.csdnimg.cn/direct/f431511decea4bc184e883078672402e.png#pic_center)
   
   > `makefile` 的依赖的从上至下的，换句话说就是目标文件是第一句里的目标，如果不满足执行依赖，就会继续向下执行。如果满足了生成目标的依赖，就不会再继续向下执行了。`make` 会自动寻找规则里需要的材料文件，执行规则下面的行为生成规则中的目标。

2. 执行`make`命令
   ![2](https://img-blog.csdnimg.cn/direct/60e92f1b94c14e2dba8c672ffa1fc5f5.png#pic_center)

3. 执行生成的`test`
   ![3](https://img-blog.csdnimg.cn/direct/298eb7971dd646fe8069012cdfcedcbe.png#pic_center)

---

### 第二版

1. 我们上点强度，修改下`test`,修改后需要多文件联合编译。
   ![1](https://img-blog.csdnimg.cn/direct/4bc9c26a07ff42708307dd4f1d35491b.png#pic_center)
2. 此时就要联合多文件编译，我们先从简单的来，先想想我们在之前用`gcc`是怎么写的，我们就怎么写到`makefile`
   ![2](https://img-blog.csdnimg.cn/direct/b47515e6655b484daea9a3403808737c.png#pic_center)
3. 执行`make `
   
   ![3](https://img-blog.csdnimg.cn/direct/008ac0cff5e04b64b18bfa2c6a41348c.png#pic_center)
   ---
   
   ### 第三版
   
   在实际开发中，有可能`add.c`会有些许改动。但是如果是第二版的话，我们就要把所有的文件都在编译一遍，这非常浪费时间，非常蠢。之前我们学到编译的四个步骤中第二步是最好时间的，所以我们可以把步骤拆分。
   1. 修改`makefile` ，将步骤拆分
      ![1](https://img-blog.csdnimg.cn/direct/cfa981d808a24a519e08e85ef18b47a3.png#pic_center)
4. 修改`add.c`，看实际效果
   ![2](https://img-blog.csdnimg.cn/direct/0ec3e82ec56c4ec3a551dcb0d9c32f20.png#pic_center)
   我们可以看到修改`add,c`之后，我们再`make`就只是把`add.c`和`test.o add.o sub.o mul.o div1.o`再重新编译了一下，这就节省不少时间。
   
   > **原因**
   > `makefile` 检测原理：修改文件后，文件的修改时间发生变化，会出现目标文件的时间早于依赖文件的时间，出现这种情况的文件会重新执行规则，重新编译。例如上面，我修改`add.c`后，`add.c`  的文件时间就晚于`add.o`，这种情况下就重新编译生成`add.o`，而其他文件时间符合，就不变。

---

### 第四版

有童鞋可能会问我们之前介绍的函数还没有发挥作用，别急，接下来我就为你介绍怎么把函数用到`makefile`里面。

1. 修改`makefile`，使用函数
   ![1](https://img-blog.csdnimg.cn/direct/ee04e5f2ab6f4002a53588281c4a0b91.png#pic_center)
   
   > `src = add.c sub.c mul.c div1.c`
   > `obj = add.o sub.o mul.o div1.o`
2. 执行`make`
   ![2](https://img-blog.csdnimg.cn/direct/dc754b76f76244cc957cd27066fe3805.png#pic_center)
3. 执行`./test`,效果和之前和一样，我这里就不演示了。
   
   > **pay attention:** 在使用`wildcard`和`patsubst` 函数参数之间不要随便加空格，严格按照我的格式来。~~（因为我一开始学的时候，我多加了个空格，结果debug了半天，心态差点炸了）~~ 

---

### 第五版

> 先来重温一下前面讲的三个**自动变量**：
> 
> - `$@`:在规则命令中，表示规则中的目标
> - `$^`在规则命令中，表示规则中的所有条件，组成一个列表，以空格隔开，如果这个列表中有重复项，则去重
> - `$<`::在规则命令中，表示规则中的第一个条件，如果将该变量用在模式规则中，它可以将依赖条件列表中的依赖依次取出，套用模式规则

1. 修改`makefile`，使用三个自动变量
   ![1](https://img-blog.csdnimg.cn/direct/e7704f6960af468899a3d6ac96448c52.png#pic_center)
2. 执行`make`
   ![2](https://img-blog.csdnimg.cn/direct/1d0b22ea6d9a49369254f6446115a70f.png#pic_center)
3. 执行`./test`，效果和之前是一样的，我这里就不展示了。
   
   > **`sub，add` 这些指令中使用`$<`和`$^`都能达到效果，但是为了模式规则，所以使用的`$<`**

---

### 第六版

很多人以为第五版是最终版，但其实不是，因为第五版的可拓展性太差了，比如在加一个功能（如取模运算）的函数就需要再增加取模函数部分，这很蠢，所以，模式规则来了。

1. 修改`makefile`，套用模式规则。

```bash
%.o : %.c
    gcc $< -o $@
```

![1](https://img-blog.csdnimg.cn/direct/8729a5714b344c03a9a90f6b0549362b.png#pic_center)

2. 执行`make`，执行生成的文件。
   ![2](https://img-blog.csdnimg.cn/direct/ad08c764b3a44d739eb13a2264d5363c.png#pic_center)
   3. 添加一个运算，取模。我们只需要写好源代码，修改`test.c`，然后直接`make`即可，不用改`makefie`，这就很`nice`
      ![3](https://img-blog.csdnimg.cn/direct/b71c50f84b644870ba35964959ffbf0a.png#pic_center)

---

### 第七版

第七版是对第六版的优化，以后文件集合会有很多，我们就需要指定哪个文件集合使用哪个规则，这就要用到静态模式规则。

1. 修改`makefile`，指定模式规则给`obj`用。
   ![1](https://img-blog.csdnimg.cn/direct/cd3eee7269504f8b8dd70e2795cd9be9.png#pic_center)

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
>      - [染色法判断二分图（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135094296?spm=1001.2014.3001.5501)
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
