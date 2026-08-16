
# C++项目推荐：基于异步日志系统的云存储 | 代码随想录

> **本项目目前只在[知识星球](https://programmercarl.com/other/kstar.html)答疑并维护**。

这次带大家做一个全新的C++项目：基于异步日志系统的云存储。

这个项目主要有两个部分：

日志部分：
1. 支持异步写日志，防止写日志阻塞外部业务逻辑
2. 支持备份重要日志，防止crush后无法debug
3. 支持多线程程序并发写日志
4. 支持输出日志到控制台、文件以及按照文件大小滚动文件中，文件大小可配置

存储部分：
1. 是一个类似于网盘的项目
2. 支持浅度存储和深度存储
3. web端上传下载

其实这个项目可以拆成两个项目，一个是异步日志系统，一个是云存储。

为什么要结合一些？

主要是亮点：

* 带大家感受一下，异步日志系统 如何嫁接在另一个系统上
* 这个项目更丰满，符合星球里 四星的标准！

为什么要做这个项目呢。 首先来聊一聊 日志系统的重要性。

日志系统在软件开发中作用主要在代码编写和调试以及项目启动后的系统运行状况记录。

能够详细记录程序的执行流程、变量的值以及函数的调用情况等，所需要的任何信息都可以通过日志来获取。

由于日志系统在项目的整个生命周期中都有着不可替代的作用，**因此可以说任何项目都可以并且应该集成日志系统以便debug，性能分析等操作**。

也就是说**设计好日志系统 可以嫁接到 所有C++项目里**。

就拿星球里目前的C++项目来举例，例如：

* 基于异步日志系统的 [HTTP服务框架（C++）](https://programmercarl.com/other/project_http.html)
* 基于异步日志系统的[手撕RPC框架（新项目）（C++）](https://programmercarl.com/other/project_C++RPC.html)
* 基于异步日志系统的[分布式存储项目（第二版）（C++）](https://programmercarl.com/other/project_fenbushi.html)
* 基于异步日志系统的[轻量级网络库muduo（第二版）（C++）](https://programmercarl.com/other/project_muduo.html)
* 基于异步日志系统的[高性能服务器项目（C++）](https://programmercarl.com/other/project_webserver.html)

等等等， 写好异步日志系统，可以嫁接到所有的项目里，为项目添加亮点毕竟所有的项目都需要打日志！

**如果感觉你自己的项目没有亮点可说，那么就可以在项目里添加个异步日志系统，增加亮点**！

## 基于异步日志系统的云存储项目精讲


该项目的专栏是[知识星球](https://programmercarl.com/other/kstar.html)录友专享的。

项目专栏依然是将 「简历写法」给大家列出来了，大家学完就可以参考这个来写简历：

给出一般写法，适用于 基础不太好的录友写：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325110714.png' width=500 alt=''></img></div>

给出高阶写法，适用于 想冲刺大厂的录友写：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325110743.png' width=500 alt=''></img></div>

做完该项目，面试中大概率会有哪些面试问题，以及如何回答，也列出好了：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325110819.png' width=500 alt=''></img></div>

专栏中的项目面试题都掌握的话，这个项目在面试中基本没问题。

很多录友在做项目的时候，把项目运行起来 就是第一大难点！

不少录友对日志系统还不了解，所以我们先从最基本的日志系统开始讲解，然后再讲异步日志：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325110957.png' width=500 alt=''></img></div>

日志系统主要有四大点：日志记录器、日志级别、日志格式化器、日志输出器 ：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325111033.png' width=500 alt=''></img></div>

接下来再讲 基于异步日志系统的云存储整体框架：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325111140.png' width=500 alt=''></img></div>

图文并茂：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325111217.png' width=500 alt=''></img></div>

我们这个项目是有页面的： （C++项目很少有页面，主要是本项目是云存储，我们从web端上传和下载文件）

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325111254.png' width=500 alt=''></img></div>

最后也给出项目的拓展方向，大家如果学有余力，可以自行去拓展，不断深挖：

<div align="center"><img src='https://file1.kamacoder.com/i/algo/20250325111409.png' width=500 alt=''></img></div>

## 答疑

本项目在[知识星球](https://programmercarl.com/other/kstar.html)里为 文字专栏形式，大家不用担心，看不懂，星球里每个项目有专属答疑群，任何问题都可以在群里问，都会得到解答：

![](https://file1.kamacoder.com/i/web/2025-09-26_11-30-13.jpg)


## 获取本项目专栏

**本文档仅为星球内部专享，大家可以加入[知识星球](https://programmercarl.com/other/kstar.html)里获取，在星球置顶一**



