## 无名管道：PIPE



### 简介

* 管道的工作原理基于==生产者-消费者模型。==生产者进程将数据写入管道，消费者进程从管道读取数据。
* ==在Linux或Unix系统中，数据是顺序进入和出来的，就像一个队列一样，所以管道的工作机制也被称为FIFO（First-In-First-Out，先进先出）。==
* ==在Linux内核中，PIPE的实现主要基于VFS（Virtual File System）模型。==VFS提供了一个抽象层，使得==各种不同的文件系统可以提供一致的接口给用户空间。==
  * VFS定义了许多操作（例如读、写、打开、关闭等），这些操作会被映射到具体的文件系统操作。
  * 对于管道来说，内核会维护一个数据缓冲区，同时为读和写管道提供相应的系统调用。



### 实例

#### 单向通信

* ```c++
  #include <stdio.h>
  #include <unistd.h>
  
  #define BUFFER_SIZE 25
  #define READ_END	0	//管道的读入端
  #define WRITE_END	1	//管道的写入端
  
  //把管道看成一个特殊的文件就好理解了
  int main(void)
  {
  	char write_msg[BUFFER_SIZE] = "Greetings";
  	char read_msg[BUFFER_SIZE];
  	int fd[2];
  	
  	if (pipe(fd) == -1) {
  		fprintf(stderr,"Pipe failed");
  		return 1;
  	}
  
  	pid_t pid = fork();
  
  	if (pid < 0) {
  		fprintf(stderr, "Fork failed");
  		return 1;
  	}
  
  	if (pid > 0) {  /* parent process */
  		close(fd[READ_END]);
  
  		write(fd[WRITE_END], write_msg, strlen(write_msg) + 1); 
  
  		close(fd[WRITE_END]);
  	}
  	else { /* child process */
  		close(fd[WRITE_END]);
  
  		read(fd[READ_END], read_msg, BUFFER_SIZE);
  		printf("read %s\n",read_msg);
  
  		close(fd[READ_END]);
  	}
  
  	return 0;
  }
  ```

* 创建了一个管道和一个子进程。

* 父进程将 "Greetings" 写入管道，子进程从管道读取数据，并打印出来。

* 基本的管道通信示例，使用父子进程进行交互。

* **==这个代码没有循环，如何保证父进程和子进程都可以走一遍这个代码?==**

  * 没有显示的循环，但其逻辑确实允许父进程和子进程分别走一遍这段代码，==这是由 `fork()` 系统调用实现的。==
  * ==在 `fork()` 调用之后，父进程和子进程会各自独立地执行剩余的代码。==
  * `fork()` 的返回值用于区分父进程和子进程。在父进程中，`fork()` 返回子进程的进程 ID；而在子进程中，`fork()` 返回 0。
  * 在这个程序中，`fork()` 之后的代码会被父进程和子进程各自执行一次。
    * 父进程和子进程通过检查 `fork()` 的返回值来确定它们应该执行的代码部分：父进程会执行 `if (pid > 0)` 中的代码，而子进程会执行 `else` 中的代码。

#### 双向通信

* 使用两个管道，一个用于父进程向子进程发送消息，另一个用于子进程向父进程发送消息。

* ```c++
  #include <stdio.h>
  #include <unistd.h>
  #include <string.h>
  
  #define BUFFER_SIZE 25
  #define READ_END	0
  #define WRITE_END	1
  
  int main(void)
  {
  	char write_msg[BUFFER_SIZE] = "Hello, Child!";
  	char read_msg[BUFFER_SIZE];
  	int fd1[2], fd2[2]; // two pipes
  	
  	if (pipe(fd1) == -1 || pipe(fd2) == -1) {
  		fprintf(stderr,"Pipe failed");
  		return 1;
  	}
  
  	pid_t pid = fork();
  
  	if (pid < 0) {
  		fprintf(stderr, "Fork failed");
  		return 1;
  	}
  
  	if (pid > 0) {  /* parent process */
  		close(fd1[READ_END]);
  		close(fd2[WRITE_END]);
  
  		write(fd1[WRITE_END], write_msg, strlen(write_msg) + 1); 
  		close(fd1[WRITE_END]);
  
  		read(fd2[READ_END], read_msg, BUFFER_SIZE);
  		printf("Parent read %s\n", read_msg);
  		close(fd2[READ_END]);
  	}
  	else { /* child process */
  		close(fd1[WRITE_END]);
  		close(fd2[READ_END]);
  
  		read(fd1[READ_END], read_msg, BUFFER_SIZE);
  		printf("Child read %s\n", read_msg);
  		close(fd1[READ_END]);
  
  		char write_msg_child[BUFFER_SIZE] = "Hello, Parent!";
  		write(fd2[WRITE_END], write_msg_child, strlen(write_msg_child) + 1); 
  		close(fd2[WRITE_END]);
  	}
  
  	return 0;
  }
  
  ```

* 



### 原理

<img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230719220518173.png" alt="image-20230719220518173" style="zoom: 80%;" />

* 在内核空间里开辟了一个缓冲区
* 相互隔离的进程可以通过管道实现通信
* 内核空间把管道看成一个文件对待
  * 管道文件没有名字
  * 通过文件描述符对其进行读写

<img src="C:\Users\yi'k\AppData\Roaming\Typora\typora-user-images\image-20230719220856673.png" alt="image-20230719220856673" style="zoom:80%;" />

#### 通信原理

* 像一个管道连接两个进程
* 一个进程的输出作为另一个进程的输入
* 用于亲缘进程之间的通信：共享资源