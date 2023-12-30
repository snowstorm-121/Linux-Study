> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

## 动态库和静态库的理论对比

- **静态库**在文件中静态展开，所以有但多少文件就展开多少次，**非常吃内存**，100M的文件展开一百次就是1G。但是这样的好处是静态**加载的速度快**。
- **动态库**使用时，会讲动态库加载到内存，10个文件也只需要加载一次，然后将这些文件用到库时候临时去加载，**速度慢一些**，**但是很省内存**。
  
  ![总述](https://img-blog.csdnimg.cn/direct/adb0340d31bd410c8b113c613d527a8e.png#pic_center)
  ---
  
  ## 静态库的制作
  1. **写好源代码**。下面我以写一个可以四则运算的静态库的为例）
     ![源码](https://img-blog.csdnimg.cn/direct/49b5ce1db6ef4834872fc609146a7f6d.png#pic_center)

---

2. **编译源代码生成 `.o`文件。**
   ![编译生成](https://img-blog.csdnimg.cn/direct/4b477c0bfbfc4ee69a182739d449de7c.png#pic_center)
   
   > ### 可能有些童鞋没学过编译的四个过程，下面我来介绍一下
   > 1. **预处理：** 展开宏，头文件，替换编译条件，删除注释，空行，空白。
   > 2. **编译：** 检查语法规范。
   > 3. **汇编：** 将汇编指令翻译成机器指令。
   > 4. **链接：** 数据段合并，地址回填。
   > 
   > ---
   > 
   > ![gcc](https://img-blog.csdnimg.cn/direct/799db8878ec44ddbbfdd4a0bd5f3e64a.png#pic_center)
   > 
   > ---
   > 
   > ### 关于gcc常用的编译参数
   > 
   > - `-I`：指定头文件所在目录位置。
   > - `-c`：只做预处理，编译，汇编。得到二进制文件。
   > - `-g`：编译时添加调试文件，用于 gdb 调试。
   > - `-Wall`：显示所有警告信息。
   > - `-D`：向程序中“动态”注册宏定义
   > - `-L`：指定动态库路径。
   > - `-l`：指定动态库库名。

---

3. **制作静态库**
   命令：`ar rcs lib库名.a file1.o file2.o ...`
   ![3](https://img-blog.csdnimg.cn/direct/543ed84653144ebda1c7873e9a4b8cc5.png#pic_center)

---

4. **制作头文件** 
   如果我们直接使用静态库`gcc test.c lib库名.a -o a.out`,就会有以下警告。
   ![4](https://img-blog.csdnimg.cn/direct/90c049305e8f4550a8d8c3975019aea2.png#pic_center)
   为了防止出现以上警告，我们就要做一个头文件来包含函数声明（当然你也可以直接在`test`中声明，但这体现我们的水平😁）
   ![5](https://img-blog.csdnimg.cn/direct/cbb6f5e3301d4a91a64cc3d40745609f.png#pic_center)
   
   > **左边的 define 为头文件守卫，防止在代码中多次 include 头文件，多次展开静态库，带来的额外开销**

接下来我们再来编译，这样就不会就有警告报错了。
![6](https://img-blog.csdnimg.cn/direct/53c0a72664b14c95ad28e3a5b3fb56aa.png#pic_center)

> 最后`-I  ...`是写头文件的地址

---

## 动态库的制作

1. **编写源代码**
   ![1](https://img-blog.csdnimg.cn/direct/b0abe82f3e96461e946734b8fd490cb1.png#pic_center)

---

2. **生成位置无关的`.o `文件**
   命令：`gcc -c add.c -o add.o -fPIC`
   
   > **使用这个参数`-fPIC`过后，生成的函数就和位置无关，挂上@plt 标识，等待动态绑定**
   > ![2](https://img-blog.csdnimg.cn/direct/6249d04f2fb141a190eb812026e477db.png#pic_center)

---

3. **使用制作动态库**
   命令:`gcc -shared -o lib库名.so file1.o file2.o ....`
   ![1](https://img-blog.csdnimg.cn/direct/66f2325699e54819ba7092eec810ac3d.png#pic_center)

---

4. 编译程序
   命令：`gcc test,c  -o test -l 库名 -L 路径 -I 头文件的位置`

---

5. 执行文件，然后出错😂。
   ![2](https://img-blog.csdnimg.cn/direct/f30301bcbaa54a7cb59d24f375e5df6c.png#pic_center)

---

6. 出错原因及解决方法
- **出错原因分析：**
  - 连接器：工作于链接阶段，工作时需要 -l 和 -L
  - 工作于程序运行阶段，工作时需要提供动态库所在目录位置
- **解决方法：**
  1. 通过环境变量指定动态库所在位置：`export LD_LIBRARY_PATH=动态库路径`。但是这个方法有个弊端，当关闭终端，再次执行`a.out` 时，又报错。这是因为，环境变量是进程的概念，关闭终端之后再打开，是两个进程，环境变量发生了变化。
     ![3](https://img-blog.csdnimg.cn/direct/07b35aba663541a3892d9b2b146d2b59.png#pic_center)
  2. 写入终端配置文件。 步骤如下：
     - ` vi ~/.bashrc` 
     - 写入 `export LD_LIBRARY_PATH=动态库路径`
     - `. .bashrc/ source` 或者`.bashrc / ` 或者重启 终端 ---> 让修改后的.bashrc 生效 
     - `./a.out` 成功！！！
  3. 配置文件法。步骤如下：
     - `sudo vi /etc/ld.so.conf`
     - 写入 动态库绝对路径 保存
     - `sudo ldconfig -v` 使配置文件生效。
     - `./a.out` 成功！！！--- 使用` ldd a.out `查看

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。正可谓“**键盘敲烂，月薪过万**”

---

> 如果你觉得我写题解还不错的，请各位**王子公主**移步到我的其他题解看看
> 
> 1. **数据结构与算法部分（还在更新中）：**
>    - [***C++ STL总结 - 基于算法竞赛（强力推荐***）](https://blog.csdn.net/yourgrandfather_/article/details/135051716?spm=1001.2014.3001.5501)
>    - [动态规划——多重背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135125267)
>    - [动态规划——完全背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135111459)
>    - [动态规划——01背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135103012?spm=1001.2014.3001.5501)
>    - [动态规划——分组背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135134277)
>    - [动态规划——最长上升子序列（LIS)](https://blog.csdn.net/yourgrandfather_/article/details/135150351)
>    - [二叉树的中序遍历（三种方法）](https://blog.csdn.net/yourgrandfather_/article/details/135167817)
>    - [最长回文子串](https://blog.csdn.net/yourgrandfather_/article/details/135183977)
>    - [最短路算法——Dijkstra（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134869064?spm=1001.2014.3001.5501)
>    - [最短路算法———Bellman_Ford算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134935786?spm=1001.2014.3001.5501)
>    - [最短路算法———SPFA算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135004393?spm=1001.2014.3001.5501)
>    - [最小生成树算法———prim算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135026901?spm=1001.2014.3001.5501)
>    - [最小生成树算法———Kruskal算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135039904?spm=1001.2014.3001.5501)
>    - [染色法判断二分图（C++实现）]([染色法判断二分图（C++实现）-CSDN博客](https://blog.csdn.net/yourgrandfather_/article/details/135094296?spm=1001.2014.3001.5501)
> 2. **Linux部分（还在更新中）：**
>    - [Linux学习之初识Linux](https://blog.csdn.net/yourgrandfather_/article/details/134953315?spm=1001.2014.3001.5501)
>    - [Linux学习之基础命令（适合小白）](https://blog.csdn.net/yourgrandfather_/article/details/135189166)
>    - [Linux学习之命令行基础操作](https://blog.csdn.net/yourgrandfather_/article/details/134956923?spm=1001.2014.3001.5501)
>    - [Linux学习之权限管理和用户管理](https://blog.csdn.net/yourgrandfather_/article/details/135222868)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
