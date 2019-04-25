---
title: linux-signal
date: 2019-04-25 19:30:56
updated: 2019-04-25 19:30:56
tags: linux
---
## 信号概述
1. 信号的名字和编号，以“SIG”开头。  
信号定义在`signal.h`中，信号名都为正整数。可以用`kill -l`查看信号名以及序号(从1开始编号)。  
2. 信号的处理：
+ 忽略：大多数信号都可以使用这个方式处理，但是`SIGKILL`和`SIGSTOP`不能被忽略。
+ 捕获：当该信号产生时，由内核来调用用户自定义函数。
+ 默认动作：当信号产生时，系统执行默认的处理动作。具体的处理如下：
<!-- more -->
```
       信号         值      动作   说明
       ─────────────────────────────────────────────────────────────────────
       SIGHUP        1       A     在控制终端上是挂起信号, 或者控制进程结束
       SIGINT        2       A     从键盘输入的中断
       SIGQUIT       3       C     从键盘输入的退出
       SIGILL        4       C     无效硬件指令
       SIGABRT       6       C     非正常终止, 可能来自 abort(3)
       SIGFPE        8       C     浮点运算例外
       SIGKILL       9      AEF    杀死进程信号
       SIGSEGV      11       C     无效的内存引用
       SIGPIPE      13       A     管道中止: 写入无人读取的管道
       SIGALRM      14       A     来自 alarm(2) 的超时信号
       SIGTERM      15       A     终止信号
       SIGUSR1   30,10,16    A     用户定义的信号 1
       SIGUSR2   31,12,17    A     用户定义的信号 2
       SIGCHLD   20,17,18    B     子进程结束或停止
       SIGCONT   19,18,25          继续停止的进程
       SIGSTOP   17,19,23   DEF    停止进程
       SIGTSTP   18,20,24    D     终端上发出的停止信号
       SIGTTIN   21,21,26    D     后台进程试图从控制终端(tty)输入
       SIGTTOU   22,22,27    D     后台进程试图在控制终端(tty)输出
       
	   "动作(Action)"栏 的 字母 有 下列 含义:

       A      缺省动作是结束进程.

       B      缺省动作是忽略这个信号.

       C      缺省动作是结束进程, 并且核心转储.

       D      缺省动作是停止进程.

       E      信号不能被捕获.

       F      信号不能被忽略.

       (译注: 这里 "结束" 指 进程 终止 并 释放资源, "停止" 指 进程 停止  运行,
       但是 资源 没有 释放, 有可能 继续 运行.)
```
## 信号处理函数的注册
信号处理函数的注册有下面两种：  
1. signal
2. sigaction（复杂）
信号处理发送函数有下面两种：
1. kill
2. sigqueue（复杂）

signal的函数原型  
```c
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```
signal函数若成功执行，返回上一次的注册函数（即注册之前的信号捕获行为）。否则返回SIG\_ERR同时置位errno。  
一个简单的信号捕获函数如下：  
```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
void handler(int signum){
	printf("receive signum is %d\n", signum);
}

int main()
{
	signal(SIGINT, handler);
	//sleep(10);
	while(1);
	return 0;
}
```

> 这里遇到另一个坑，本意是想调用sleep，使得在给定的时间内，每键入一次ctrl+c就打印一行数据，结果是一次ctrl+c之后进程直接退出。查阅了sleep的信息定义，原来sleep是推迟一个进程的执行直到设定时间到或**有信号传递给进程**。  

```
     The sleep() function suspends execution of the calling thread until ei‐
     ther seconds seconds have elapsed or a signal is delivered to the thread
     and its action is to invoke a signal-catching function or to terminate
     the thread or process.  System activity may lengthen the sleep by an in‐
     determinate amount.
```
信号的处理还有两种状态，分别是默认处理和忽略，分别需要将handler设置为SIG\_INT或SIG\_DFL即可。  

> 需要说明的两个问题：
1. 当执行一个程序时，所有信号的状态都是系统默认或者忽略状态的，除非是调用exec进程忽略了某些信号。exec函数将原先设置为捕捉的信号都更改为默认动作，其他信号的状态则不会改变。2. 当一个进程调用了fork函数，那么子进程会继承父进程的信号处理方式。

### 信号发送函数
kill的函数原型如下:  
```c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```
关于pid的取值有下面几种情况：  
1. pid > 0: 信号发送给对应pid的进程；
2. pid == 0：信号发送给所有与该进程有着相同组ID的其他进程（需要有权限）；
3. pid == -1：如果该进程具有root权限，则将信号发送给所有进程，除了系统进程和该进程本身。如果该进程没有root权限，则将信号发送给所有与该进程有着相同uid的进程，除了该进程本身。只要任何一个进程响应了信号，就不会返回错误。
4. pid < 0 && pid != -1：信号发送给所有与pid的绝对值有着相同组id的进程。
如果成功执行，返回0。否则返回-1并且置位error。  

## 可靠信号和不可靠信号



reference:  
[Linux 信号（signal）](https://www.jianshu.com/p/f445bfeea40a)  
[中文 man 手册页计划](https://github.com/man-pages-zh/manpages-zh)  

