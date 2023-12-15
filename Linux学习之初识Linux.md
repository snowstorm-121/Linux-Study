> ### 写在前面：
> 
> 我的Linux的学习之路非常坎坷。第一次学习Linux是在大一下的开学没多久，结果因为不会安装VMware就无疾而终了，可以说是没开始就失败了。第二次学习Linux是在大一下快放暑假（那个时候刚刚过完考试周），我没什么事做就又重拾Linux，不服输的我选择再战Linux，这一次学习还算顺利，虽然中间有些小插曲但是不影响整体学习进度， 我看着B站上的视频一点点学习Linux，基本上把Linux的基础指令学完了。学完之后我又遇到问题了，视频基本上到这就结束了，而我却不知道下一步该学什么，于是就没怎么碰Linux，结果没过多长时间我就把学的Linux指令忘的一干二净。现在是我第三次学习Linux，我决定重新开始学Linux，同时为了让自己学习的效果更好，我选择以写blog的形式逼迫自己每天把学习到的Linux知识整理下来。这也就是我写这个系列blog的原因。

## Linux的历史渊源

### unix

- 1965年之前，电脑只有军事或者学院的研究机构碰的起，当时大型主机至多能提供30台终端（30个键盘、显示器)的连接。
- 1965年左后由贝尔实验室、麻省理工学院 以及 通用电气共同发起了Multics项目，想让大型主机支持300台终端
- 1969年前后这个项目进度缓慢，资金短缺，贝尔实验室退出了研究
-  1969年从这个项目中退出的Ken Thompson当时在实验室无聊时，为了让一台空闲的电脑上能够运行“星际旅行”游行，在8月份左右趁着其妻子探亲的时间，用了1个月的时间编写出了 Unix操作系统的原型
- 1970年，美国贝尔实验室的 Ken Thompson，以 BCPL语言 为基础，设计出很简单且很接近硬件的 B语言（取BCPL的首字母），并且他用B语言写了第一个UNIX操作系统。
  因为B语言的跨平台性较差，为了能够在其他的电脑上也能够运行这个非常棒的Unix操作系统，Dennis Ritchie和Ken Thompson 从B语言的基础上准备研究一个更好的语言
  - 1972年，美国贝尔实验室的 Dennis Ritchie在B语言的基础上最终设计出了一种新的语言，他取了BCPL的第二个字母作为这种语言的名字，这就是C语言
  - 1973年初，C语言的主体完成。Thompson和Ritchie迫不及待地开始用它完全重写了现在大名鼎鼎的Unix操作系统
    
    ### Minix
    
    因为AT&T(通用电气)的政策改变，在Version 7 Unix推出之后，发布新的使用条款，将UNIX源代码私有化，在大学中不再能使用UNIX源代码。Andrew S. Tanenbaum(塔能鲍姆)教授为了能在课堂上教授学生操作系统运作的实务细节，决定在不使用任何AT&T的源代码前提下，自行开发与UNIX兼容的操作系统，以避免版权上的争议。他以小型UNIX（mini-UNIX）之意，将它称为MINIX。
    
    ### Linux
    
    因为Minix只是教学使用，因此功能并不强，因此Torvalds利用GNU的bash当做开发环境，gcc当做编译工具，编写了Linux内核-v0.02，但是一开始Linux并不能兼容Unix，即Unix上跑的应用程序不能在Linux上跑，即应用程序与内核之间的接口不一致，因为Unix是遵循POSIX规范的，因此Torvalds修改了Linux，并遵循POSIX（Portable Operating System Interface，他规范了应用程序与内核的接口规范）； 一开始Linux只适用于386，后来经过全世界的网友的帮助，最终能够兼容多种硬件。
    ![操作系统的发展](https://img-blog.csdnimg.cn/img_convert/df96e289eb08cd5e6e4170b17f5cf742.gif#pic_center)
    
    ## Linux的介绍
    
    ### Linux内核和发行版本
    
    #### Linux内核版本
    
    内核(kernel)是系统的心脏，是运行程序和管理像磁盘和打印机等硬件设备的核心程序，它提供了一个在裸设备与应用程序间的抽象层。
    Linux内核版本又分为稳定版和开发版，两种版本是相互关联，相互循环：
  - 稳定版：具有工业级强度，可以广泛地应用和部署。新的稳定版相对于较旧的只是修正一些bug或加入一些新的驱动程序。
  - 开发版：由于要试验各种解决方案，所以变化很快。

内核源码网址：http://www.kernel.org 所有来自全世界的对Linux源码的修改最终都会汇总到这个网站，由Linus领导的开源社区对其进行甄别和修改最终决定是否进入到Linux主线内核源码中。

#### Linux发行版本

Linux发行版 (也被叫做 GNU/Linux 发行版) 通常包含了包括桌面环境、办公套件、媒体播放器、数据库等应用软件。

目前市面上较知名的发行版有：Ubuntu、RedHat、CentOS、Debian、Fedora、SuSE、OpenSUSE、Arch Linux、SolusOS 等。
![Linux发行版本](https://img-blog.csdnimg.cn/img_convert/94fda88f37da710407059bf585dfccf4.png#pic_center)

### Linux目录

![Linux目录](https://img-blog.csdnimg.cn/img_convert/cee83d5e4c0ec6e1e6b1158c8ec56dba.png#pic_center)

- /：根目录，一般根目录下只存放目录，在Linux下有且只有一个根目录。所有的东西都是从这里开始。当你在终端里输入“/home”，你其实是在告诉电脑，先从/（根目录）开始，再进入到home目录。
- /bin: /usr/bin: 可执行二进制文件的目录，如常用的命令ls、tar、mv、cat等。
- /boot：放置linux系统启动时用到的一些文件，如Linux的内核文件：/boot/vmlinuz，系统引导管理器：/boot/grub。
- /dev：存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱 mount - /dev/cdrom /mnt。
- /etc：系统配置文件存放的目录，不建议在此目录下存放可执行文件，重要的配置文件有/etc/inittab、/etc/fstab、/etc/init.d、/etc/X11、/etc/sysconfig、/etc/xinetd.d。  
- /home：系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下，表示当前用户的家目录，edu 表示用户 edu 的家目录。
- /lib: /usr/lib: /usr/local/lib：系统使用的函数库的目录，程序在执行过程中，需要调用一些额外的参数时需要函数库的协助。
- /lost+fount：系统异常产生错误时，会将一些遗失的片段放置于此目录下。
- /mnt: /media：光盘默认挂载点，通常光盘挂载于 /mnt/cdrom 下，也不一定，可以选择任意位置进行挂载。
- /opt：给主机额外安装软件所摆放的目录。
- /proc：此目录的数据都在内存中，如系统核心，外部设备，网络状态，由于数据都存放于内存中，所以不占用磁盘空间，比较重要的目录有 /proc/cpuinfo、/proc/interrupts、/proc/dma、/proc/ioports、/proc/net/* 等。
- /root：系统管理员root的家目录。
- /sbin: /usr/sbin: /usr/local/sbin：放置系统管理员使用的可执行命令，如fdisk、shutdown、mount 等。与 /bin 不同的是，这几个目录是给系统管理员 root使用的命令，一般用户只能"查看"而不能设置和使用。
- /tmp：一般用户或正在执行的程序临时存放文件的目录，任何人都可以访问，重要数据不可放置在此目录下。
- /srv：服务启动之后需要访问的数据目录，如 www 服务需要访问的网页数据存放在 /srv/www 内。
- /usr：应用程序存放目录，/usr/bin 存放应用程序，/usr/share 存放共享数据，/usr/lib 存放不能直接运行的，却是许多程序运行所必需的一些函数库文件。/usr/local: 存放软件升级包。/usr/share/doc: 系统说明文件存放目录。/usr/share/man: 程序说明文件存放目录。
- /var：放置系统执行过程中经常变化的文件，如随时更改的日志文件 /var/log，/var/log/message：所有的登录文件存放目录，/var/spool/mail：邮件存放的目录，/var/run:程序或服务启动后，其PID存放在该目录下。

```
├── bin -> usr/bin # 用于存放二进制命令
├── boot # 内核及引导系统程序所在的目录
├── dev # 所有设备文件的目录（如磁盘、光驱等）
├── etc # 配置文件默认路径、服务启动命令存放目录
├── home # 用户家目录，root用户为/root
├── lib -> usr/lib # 32位库文件存放目录
├── lib64 -> usr/lib64 # 64位库文件存放目录
├── media # 媒体文件存放目录
├── mnt # 临时挂载设备目录
├── opt # 自定义软件安装存放目录
├── proc # 进程及内核信息存放目录
├── root # Root用户家目录
├── run # 系统运行时产生临时文件，存放目录
├── sbin -> usr/sbin # 系统管理命令存放目录
├── srv # 服务启动之后需要访问的数据目录
├── sys # 系统使用目录
├── tmp # 临时文件目录
├── usr # 系统命令和帮助文件目录
└── var # 存放内容易变的文件的目录
```

> 这一章中有很多都是借鉴[小小明-代码实体](https://blog.csdn.net/as604049322/article/details/120446586?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170223743616800192243115%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170223743616800192243115&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-120446586-null-null.142%5Ev96%5Epc_search_result_base3&utm_term=linux&spm=1018.2226.3001.4187)这位大佬的，大家可以去看他的原文。
> 本人实力尚浅，有什么错误，还请大家斧正。
> 后面我会持续更新，记录我学习Linux的进度，也希望大家点点赞，一起进步
