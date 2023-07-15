## Lab1: 环境的搭建与简单熟悉环境



链接：[Lab: Xv6 and Unix utilities (mit.edu)](https://pdos.csail.mit.edu/6.828/2021/labs/util.html)



### 在VMware上创建一个Ubuntu虚拟机

<img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714101122023.png" alt="image-20230714101122023" style="zoom:50%;" />



### 配置环境

* 参考博客：https://zhuanlan.zhihu.com/p/624091268
* 遇到的问题：各种包没有安装，报了许多error，逐个安装即可

* 简述步骤

  * 安装需要的工具： QEMU 5.1+, GDB 8.3+, GCC, and Binutils.
  * 克隆仓库
    * <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714101911317.png" alt="image-20230714101911317" style="zoom: 50%;" />

  * 编译成功并进入xv6操作系统的shell，使用chatgpt解决问题
    * <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714102751646.png" alt="image-20230714102751646" style="zoom:150%;" />



### 关于xv6

* 是一个简化的类似Unix的操作系统
  * Unix是许多现代操作系统（如Linux和OS X）的基础
  * 麻雀虽小五脏俱全

* 运行在RISC-V微处理器上
* QEMU是一个模拟器，用C语言编写。xv6运行在这个模拟器之上
* 带有少量实用程序
  * ==**ls程序，启动后它会给出一个所有文件的列表**==
  
    * ![image-20230714104019089](C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714104019089.png)
  
    * `README`: 这是一个文件，通常包含有关目录或项目的基本信息，如使用方法，依赖项，联系信息等。
    * `xargstest.sh`: 这看起来像是一个 ==**shell 脚本文件**==，可能用于测试或者示例。==**"xargs" 是一个 Unix 命令行工具，用于从标准输入读取参数并构建并执行命令。**==
    * `cat`: 这是一个标准 Unix 工具，==**用于串联和打印文件的内容。**==
    * `echo`: 这个命令==**用于在命令行中输出文本或变量。**==
    * `forktest`: 这可能是一个==**特定的测试程序，用于测试操作系统的 fork 系统调用或进程创建。**==
    * `grep`: 这是一个==**强大的文本搜索工具，可以使用正则表达式来查找文件或命令输出中的文本。**==
    * `init`: 这==**是 Unix 和 Unix-like 系统的初始进程。它是所有其他进程的父进程，并负责在系统启动时启动其他进程。**==
    * `kill`: 这个命令==**用于发送信号给其他进程。最常见的用途是停止（“kill”）进程。**==
    * `ln`: 这个命令==**用于创建链接。可以创建硬链接或符号链接。**==
    * `ls`: 这个命令==**用于列出目录中的文件和子目录。**==
    * `mkdir`: 这个命令==**用于创建新的目录。**==
    * `rm`: 这个命令==**用于删除文件或目录。**==
    * `sh`: ==**这是一个 shell，或者命令行解释器。用户可以在其中输入命令。**==
    * `stressfs`: 这可能是一个==**用于给文件系统添加压力或进行压力测试的程序。**==
    * `usertests`: 这可能是==**一组用于测试用户空间功能或应用程序的测试。**==
    * `grind`: 这可能是一个特定的工具或程序。
    * `wc`: 这是一个==**计数工具，用于计算给定输入中的字节数，字数，和行数。**==
    * `zombie`: 这可能是一个==**用于生成或处理僵尸进程的工具或测试。**==
    * `console`: 这可能是一个==**特殊的设备文件，代表物理或虚拟控制台。**==
  
    以上这些都是在 Unix 或类 Unix 系统中常见的工具或应用程序，一般都在 `/bin` 或 `/usr/bin` 目录下。



### 热身实验

* 演示系统调用的copy（可能是安装的xv6版本不同或者输入的命令不对，我并没有找到copy.c这个文件）

  * ```c++
    int main()
    {
        char buf[64];
        
        while(1){
            int n=read(0, buf, sizeof(buf));
            if(n<=0)
                break;
            write(1,buf,n);
        }
        
        exit(0);
    }
    ```

  * read调用

    * 三个参数
    * 第一个参数：文件描述符，实际上是对以前打开的文件的引用。shell确保程序启动时，默认情况下它的==**文件描述符0连接到console输入，文件描述符1连接到console输出。**==（由此能对copy进行输入并等待它的输出）==**01文件描述符是unix的惯例。**==
    * 第二个参数：是指向某一段内存的指针，程序可以通过指针对应的地址读取内存中的数据，上面代码的第三行在栈里申请了64字节内存，以便read将数据保存。
    * 第三个参数：代码想读取的最大长度
    * 返回值：可能是读取到的字节数，当返回-1时说明出错了

* ![image-20230714112750392](C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714112750392.png)

重装为2021版本的，看起来与2020没什么区别

* open.c

  * ```c++
    int main()
    {
        int fd = open("output.txt", O_WRONLY | O_CREATE);
        write(fd, "ooo\n", 4);
        
        exit(0);
    }
    ```

  * open返回一个新分配的文件描述符，文件描述符只是一个小数字，可能是2，3或4等。

  * 对于文件描述符索引用的文件，有多种方式写入数据，==**文件描述符实际做的是索引到内核内的一个小表，这个表维护每个进程的状态，运行我们运行的每个程序。内核根据文件描述符记住每个运行进程的索引表，这个表告诉内核每个文件描述符指的是什么。**==*关键：每个进程都有自己的文件描述符空间，当两个不同进程的程序都打开了同一个文件，它们可能会得到相同的文件描述符编号，但是因为内核为每个进程维护单独的文件描述符，相同的文件描述符号，在不同的进程中可能对应不同的文件。*

* shell功能的演示：

  * 输入ls时，意味着要求shell运行名为ls的程序
  * 自己尝试

* ==**解决问题：输入cat看不到源码，原因如下，切换到主机系统模式并在正确的文件目录下打开终端，再次输入cat命令即可看到源码**==

  * 在 Unix-like 的系统中，`cat` 命令通常用来查看文本文件的内容。==**当你试图使用 `cat` 来查看一个二进制文件（比如一个执行文件或者对象文件）时，你通常会看到一些看起来像乱码的字符。这是因为二进制文件包含了一些不是打印字符的字节，这些字节在终端上显示出来时，看起来就像是乱码。**==

    `ls` 是一个可执行文件，因此它是一个二进制文件。当你运行 `cat ls` 时，你在试图查看这个二进制文件的内容，因此你看到的是乱码。

    ==**如果你想查看 `ls` 程序的源代码，你需要找到它的源代码文件。**==在 xv6 中，`ls` 程序的源代码位于 `ls.c` 文件中。然而，这个文件可能并没有包含在你的 xv6 文件系统镜像中。你可能需要在你的主机系统（也就是你编译和运行 xv6 的系统）上查看这个文件，或者将这个文件添加到你的 xv6 文件系统镜像中。



### 调用系统函数sleep来实现用户函数sleep

一系列的实验里, 无法调用传统意义上的C标准库的. 所有能够调用的系统函数都在**user/user.h**里。

<img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230714221044661.png" alt="image-20230714221044661" style="zoom:67%;" />

* 代码实现：

  * ```c++
    #include "kernel/types.h"
    #include "user/user.h"
    
    int main(int argc, char *argv[]) {
      if (argc != 2) {
        fprintf(2, "usage: sleep [ticks num]\n");
        exit(1);
      }//如果用户没有传入参数，输入提示语句并报错
      // atoi sys call guarantees return an integer
      int ticks = atoi(argv[1]);
      //使用系统调用中的sleep
      int ret = sleep(ticks);
      exit(ret);
    }
    ```

  * 将sleep添加到makefile文件中

    <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230715105349072.png" alt="image-20230715105349072" style="zoom:67%;" />

  * 在命令行中输入make qemu重新编译，使用ls即可查看到新增了sleep

    <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230715105240100.png" alt="image-20230715105240100" style="zoom:67%;" />



### pingpong：利用系统调用函数fork和pipe在父进程和子进程前交换一个字节

* 关于fork和pipe

  * `fork`: 当一个进程调用`fork`时，它创建了一个新的进程，这个新的进程被称为子进程。==**子进程是父进程的一个副本，它从父进程那里复制了大部分的属性，例如代码段、堆和栈的内容、环境变量、打开的文件描述符等。**==然而，子进程有其自己的进程ID，且其其他某些属性也可能和父进程不同。例如，子进程的父进程ID就是创建它的那个进程的进程ID。在`fork`调用返回后，父进程和子进程将从`fork`调用的下一条指令开始执行。

  * `pipe`: 用于创建一个新的管道。**==管道是一种通信方式，可以在进程之间传递数据。一个管道有一个输入端和一个输出端。==**一个进程可以向管道的输入端写数据，然后其他进程可以从管道的输出端读取数据。因此，`pipe`可以用于实现进程间的通信。`pipe`系统调用创建一个新的管道，并返回两个文件描述符，其中一个文件描述符连接到管道的输入端，另一个文件描述符连接到管道的输出端。

  * ```c++
    #include <unistd.h>
    
    int fork(void);
    
    int pipe(int pipefd[2]);
    ```

  * ==**`fork`函数返回0表示这是子进程，返回大于0的值表示这是父进程，返回的值是新创建的子进程的进程ID。如果`fork`调用失败，它返回-1。**==

    ==**`pipe`函数返回0表示成功，返回-1表示失败。如果成功，`pipefd[0]`将包含一个新的文件描述符，它连接到管道的输出端，`pipefd[1]`将包含一个新的文件描述符，它连接到管道的输入端。**==

* 代码实现：

  * ```c++
    #include "kernel/types.h"
    #include "user/user.h"
    
    int main(int argc, char *argv[]) {
      int pid;
      int pipes1[2], pipes2[2];
      char buf[] = {'a'};
      pipe(pipes1);
      pipe(pipes2);
    
      int ret = fork();
    
      // parent send in pipes1[1], child receives in pipes1[0]
      // child send in pipes2[1], parent receives in pipes2[0]
      // should have checked close & read & write return value for error, but i am
      // lazy
      if (ret == 0) {
        // i am the child
        pid = getpid();
        close(pipes1[1]);
        close(pipes2[0]);
        read(pipes1[0], buf, 1);
        printf("%d: received ping\n", pid);
        write(pipes2[1], buf, 1);
        exit(0);
      } else {
        // i am the parent
        pid = getpid();
        close(pipes1[0]);
        close(pipes2[1]);
        write(pipes1[1], buf, 1);
        read(pipes2[0], buf, 1);
        printf("%d: received pong\n", pid);
        exit(0);
      }
    }
    ```

  * 加入makefile中



### primes: 利用pipeline来实现Sieve质数算法

* 简述算法思路：

  * 每个进程被赋予一个质数，它会把这个质数打印在屏幕上
  * 它从左手边的进程不断接收数字，如果这个数字是自己质数的倍数，就把它过滤掉。否则就fork出一个新的进程，并把自己的质数传给这个进程
  * 每个进程都需等待它的子进程结束后才可以退出, 形成了一条进程生命周期依赖链

* 代码实现

  * ```c++
    #include "kernel/types.h"
    #include "user/user.h"
    
    /*
     * Run as a prime-number processor
     * the listenfd is from your left neighbor
     */
    void runprocess(int listenfd) {
      int my_num = 0;
      int forked = 0;
      int passed_num = 0;
      int pipes[2];
      while (1) {
        int read_bytes = read(listenfd, &passed_num, 4);
    
        // left neighbor has no more number to provide
        if (read_bytes == 0) {
          close(listenfd);
          if (forked) {
            // tell my children I have no more number to offer
            close(pipes[1]);
            // wait my child termination
            int child_pid;
            wait(&child_pid);
          }
          exit(0);
        }
    
        // if my initial read
        if (my_num == 0) {
          my_num = passed_num;
          printf("prime %d\n", my_num);
        }
    
        // not my prime multiple, pass along
        if (passed_num % my_num != 0) {
          if (!forked) {
            pipe(pipes);
            forked = 1;
            int ret = fork();
            if (ret == 0) {
              // i am the child
              close(pipes[1]);
              close(listenfd);
              runprocess(pipes[0]);
            } else {
              // i am the parent
              close(pipes[0]);
            }
          }
    
          // pass the number to right neighbor
          write(pipes[1], &passed_num, 4);
        }
      }
    }
    
    int main(int argc, char *argv[]) {
      int pipes[2];
      pipe(pipes);
      for (int i = 2; i <= 35; i++) {
        write(pipes[1], &i, 4);
      }
      close(pipes[1]);
      runprocess(pipes[0]);
      exit(0);
    }
    ```

  * 加入makefile中



### find：给定一个初始路径和目标文件名, 要不断递归的扫描找到所有子目录前匹配的文件全路径

* 涉及到的要点

  * 判断一个路径是文件还是目录
  * 递归地扫描一个目录
  * 需要一路上自己不断构造完整目录，需要保存一个buffer，并在合适地时候加上‘/’。
  * 需要及时关闭不再使用的file descriptor

* 代码实现：

  * ```c++
    #include "kernel/types.h"
    #include "kernel/fcntl.h"
    #include "kernel/fs.h"
    #include "kernel/stat.h"
    #include "user/user.h"
    
    /* retrieve the filename from whole path */
    char *basename(char *pathname) {
      char *prev = 0;
      char *curr = strchr(pathname, '/');
      while (curr != 0) {
        prev = curr;
        curr = strchr(curr + 1, '/');
      }
      return prev;
    }
    
    /* recursive */
    void find(char *curr_path, char *target) {
      char buf[512], *p;
      int fd;
      struct dirent de;
      struct stat st;
      if ((fd = open(curr_path, O_RDONLY)) < 0) {
        fprintf(2, "find: cannot open %s\n", curr_path);
        return;
      }
    
      if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", curr_path);
        close(fd);
        return;
      }
    
      switch (st.type) {
    
      case T_FILE:
        char *f_name = basename(curr_path);
        int match = 1;
        if (f_name == 0 || strcmp(f_name + 1, target) != 0) {
          match = 0;
        }
        if (match)
          printf("%s\n", curr_path);
        close(fd);
        break;
    
      case T_DIR:
        // make the next level pathname
        memset(buf, 0, sizeof(buf));
        uint curr_path_len = strlen(curr_path);
        memcpy(buf, curr_path, curr_path_len);
        buf[curr_path_len] = '/';
        p = buf + curr_path_len + 1;
        while (read(fd, &de, sizeof(de)) == sizeof(de)) {
          if (de.inum == 0 || strcmp(de.name, ".") == 0 ||
              strcmp(de.name, "..") == 0)
            continue;
          memcpy(p, de.name, DIRSIZ);
          p[DIRSIZ] = 0;
          find(buf, target); // recurse
        }
        close(fd);
        break;
      }
    }
    
    int main(int argc, char *argv[]) {
      if (argc != 3) {
        fprintf(2, "usage: find [directory] [target filename]\n");
        exit(1);
      }
      find(argv[1], argv[2]);
      exit(0);
    }
    ```

  * 加入makefile



### xargs：将标准输入里每一个以'**\n**'分割的行作为单独1个额外的参数, 传递并执行下一个命令. 

* 代码实现：

  * ```c++
    #include "kernel/param.h"
    #include "kernel/types.h"
    #include "user/user.h"
    
    #define buf_size 512
    
    int main(int argc, char *argv[]) {
      char buf[buf_size + 1] = {0};
      uint occupy = 0;
      char *xargv[MAXARG] = {0};
      int stdin_end = 0;
    
      for (int i = 1; i < argc; i++) {
        xargv[i - 1] = argv[i];
      }
    
      while (!(stdin_end && occupy == 0)) {
        // try read from left-hand program
        if (!stdin_end) {
          int remain_size = buf_size - occupy;
          int read_bytes = read(0, buf + occupy, remain_size);
          if (read_bytes < 0) {
            fprintf(2, "xargs: read returns -1 error\n");
          }
          if (read_bytes == 0) {
            close(0);
            stdin_end = 1;
          }
          occupy += read_bytes;
        }
        // process lines read
        char *line_end = strchr(buf, '\n');
        while (line_end) {
          char xbuf[buf_size + 1] = {0};
          memcpy(xbuf, buf, line_end - buf);
          xargv[argc - 1] = xbuf;
          int ret = fork();
          if (ret == 0) {
            // i am child
            if (!stdin_end) {
              close(0);
            }
            if (exec(argv[1], xargv) < 0) {
              fprintf(2, "xargs: exec fails with -1\n");
              exit(1);
            }
          } else {
            // trim out line already processed
            memmove(buf, line_end + 1, occupy - (line_end - buf) - 1);
            occupy -= line_end - buf + 1;
            memset(buf + occupy, 0, buf_size - occupy);
            // harvest zombie
            int pid;
            wait(&pid);
    
            line_end = strchr(buf, '\n');
          }
        }
      }
      exit(0);
    }
    ```

  * 加入makefile



### 实验结果

* <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230715140959438.png" alt="image-20230715140959438" style="zoom:67%;" />



* <img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230715141139782.png" alt="image-20230715141139782" style="zoom:67%;" />