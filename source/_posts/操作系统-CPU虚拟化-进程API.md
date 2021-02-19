---
title: 操作系统-CPU虚拟化-进程API
date: 2021-02-03 15:47:48
tags: 虚拟化CPU
categories: 操作系统
---
# 操作系统-CPU虚拟化-进程API

上一节的操作系统中我们对进程进行了一个概述，这一篇中我们就围绕着进程的API进行一个总结

## fork系统调用

系统调用`fork()`用于创建新的进程，它创建一次，返回两次，在父进程中，返回大于0的数为子进程的pid，返回小于0的数代表着创建失败（此时系统中的进程数已达规定的最大进程数），在子进程中返回0代表着创建成功。下面通过一个例子认识一下

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    int rc = 0;
    printf("my pid = %d \n", getpid());

    rc = fork();

    if(rc < 0) {
        printf("fork fialed");
        exit(1);
    } else if(rc == 0) {
        printf("I am child process, my pid: %d, my parent id: %d \n", getpid(), getppid());
    } else {
        printf("I am parent process, my pid: %d, my child id: %d \n", getpid(), rc);
    }

    return 0;
}
```

```
# 输出结果
my pid = 10959
I am parent process, my pid: 10959, my child id: 10960
I am child process, my pid: 10960, my parent id: 10959
```

我们使用`fork()`创建了子进程之后，子进程并不是从main开始处执行，而是从fork处开始执行，在父进程中，`fork()`的返回值只能为大于0或者小于0（具体的含义上面也介绍过），而在子进程中，返回0代表着创建成功。

子进程并不是完全的拷贝了父进程，子进程拥有他自己的地址空间、寄存器、程序计数器等。

另外，创建完进程后，是子进程先执行还是父进程先执行是不确定的，这取决于进程的调度算法，往复执行几次，打印的顺序可能是不一样的

> `fork()`系统调用需要使用头文件`unistd.h`

## wait系统调用

上面我们知道，通过fork可以创建出来子进程，但是子进程与父进程的执行顺序是不确定的，我们可以通过`wait()`系统调用让父进程等待子进程执行完毕，然后自己再执行

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    int rc = 0;

    printf("current pid: %d\n", getpid());

    rc = fork();

    if(rc < 0) {
        printf("fork failed!\n");
        exit(1);
    } else if (rc == 0) {
        printf("I am child, pid: %d, ppid: %d\n", getpid(), getppid());
    } else {
        rc = wait(NULL);

        if(rc < 0) {
            printf("wait child failed\n");
        } else {
            printf("I am parent, pid: %d\n", getpid());
        }
    }

    return 0;
}
```

```
# 执行结果
current pid: 5886
I am child, pid: 5887, ppid: 5886
I am parent, pid: 5886
```

可以看到，子进程先被打印出来，然后才打印出父进程

> wait系统调用存在头文件`sys/wait.h`

除了可以使用wait外，还有一个系统调用`waitpid()`，也是用于父进程等待子进程执行完毕

## exec系统调用

这个系统调用可以让子进程执行与父进程不同的程序，在上面的示例中，都是子进程与父进程执行相同的程序，这种执行是没有意义的，例如在网络中，都是父进程负责监听连接，当有连接事件到来的时候，fork出一个子进程去处理，这两个是并行的。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {

    int rc = fork();

    if(rc < 0) {
        printf("fork failed!");
    } else if (rc == 0) {
        char* params[3];

        params[0] = "wc";
        params[1] = "fork_wait_exec.c";
        params[2] = NULL;

        execvp(params[0],  params);
    } else {
        rc = wait(NULL);
    }

    return 0;
}
```

```
# 输出结果
 25  51 349 fork_wait_exec.c
```

> exec系统调用存在于头文件`unistd.h`

上面代码中创建了一个子进程，子进程被`wc`命令替换，用来统计`fork_wait_exec.c`文件的行数等，最后父进程等待子进程执行完毕。

当通过`exec`在子进程中执行其他功能时，`exec`会从可执行程序中加载代码和静态数据，并用它覆盖自己的代码段、静态数据、堆、栈，其他内存空间也会被重新的初始化。总之就是替换子进程的数据，不会在再开启一个进程去执行`wc`命令

其实`exec`是一个家族，我们可以通过`man`手册查看

它存在于头文件`unistd.h`中

```c
int execl(const char *path, const char *arg, .../* (char  *) NULL */);
int execlp(const char *file, const char *arg, .../* (char  *) NULL */);
int execle(const char *path, const char *arg, .../*, (char *) NULL, char * const envp[] */);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
```

其中共有6个，其中5个是库函数，只有execve是系统调用。

execl、execv的参数含义：

+ 第一个参数代表了可执行文件的**路径**
+ 第二个参数为可执行文件的名称
+ 从第三个参数其为可执行文件的参数（数组最后一个元素应该为NULL，其他的单个参数最后一个参数也必须为NULL）

execvp、execlp参数的含义：

+ 第一个参数为可执行文件的名字，不需要路径，执行文件时会从环境变量中搜索
+ 第二个参数及以后为可执行文件的参数（不管是数组还是单个的，都需要以NULL结尾）

execle、execvpe参数的含义：

+ 第一个参数为可执行文件的名字，不需要路径，执行文件时会从环境变量中搜索
+ 第二个参数及以后为可执行文件的参数（不管是数组还是单个的，都需要以NULL结尾）
+ 最后一个参数是为这个可执行文件添加临时的环境变量（环境变量的结尾必须用NULL结尾）

这6个函数的返回值含义是一样的，当执行失败时候，返回-1

```c
// 执行失败的例子
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    int execRet = 0;
    int rc = fork();

    if(rc < 0) {
        printf("fork failed!");
    } else if (rc == 0) {
        char* params[4];
		
        // 此处多加了一个a，并非chmod
        params[0] = "chmoad";
        params[1] = "+x";
        params[2] = "/etc/hosts";
        params[3] = NULL;

        execRet = execvp(params[0],  params);

        if(execRet == -1) {
            printf("exec failed");
        }
    } else {
        rc = wait(NULL);
    }

    return 0;
}
```

```
# 执行结果
exec failed
```

以下情况exec可能会执行失败

+ 待执行的文件没有权限
+ 找不到文件或者路径（上面示例的情况）
+ exec函数中的参数忘记以NULL结尾

## 这样设计API的好处

fork、wait、exec单独拿出来一个作用可能都不是很大，但是当他们组合在一起的，却可以创造出无限的价值，在Unix、Linux中，与用户交互的shell就是采用三者的结合创造出来的，有点类似于我们上面的示例，利用fork与exec之间的时间间隔，可以做出很多事，例如shell在这个期间做出一些改变环境的操作。

>shell是怎么执行的呢？shell其实也是一个进程，在与用户交互的时候，fork出一个子进程负责处理用户的输入，最后在shell进程中通过wait等待用户输入的子进程执行完毕

一个典型的用处就是Linux中的重定向

```bash
cat fork_wait_exec.c > tmp.c
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <fcntl.h>

int main() {
    int rc = fork();

    if(rc < 0) {
        printf("fork failed");
        exit(1);
    } else if(rc == 0) {
        close(STDOUT_FILENO);
        open("./tmp.c", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);

        char* params[3];
        params[0] = "cat";
        params[1] = "redirect.c";
        params[2] = NULL;

        execvp(params[0], params);
    } else {
        rc = wait(NULL);
    }

    return 0;
}
```

重定向的实现就是在fork到exec之间，关闭标准的输入输出文件描述符，然后将新打开的文件的描述符赋值给`STDOUT_FILENO`，这样在执行cat的时候，就把输出的结果写入了`tmp.c`文件

另外，Linux中的管道实现机制也类似于重定向，但是用的是`pipe()`系统调用，一个进程的输出被链接到管道的一端，另一个进程的输入被链接到管道的另一端，这样就形成了管道机制。

其他的进程API例如`kill()`，主要用于杀死进程、让进程睡眠等。

关于进程的API很多很多，难的不是如何使用这些API，而是理解这些API在特定的场景下的作用。
