## 1、常见的Linux命令

在Linux系统中，有许多常用的命令可以用于管理和操作系统。以下是一些**常见的Linux命令：**

1. ls：列出目录中的文件和子目录。
   示例：ls /home

2. cd：切换当前工作目录。
   示例：cd /var/www

3. pwd：显示当前工作目录的路径。

4. mkdir：创建新目录。
   示例：mkdir mydir

5. rm：删除文件或目录。
   示例：rm myfile.txt（删除文件）；rm -r mydir（删除目录及其内容）

6. cp：复制文件或目录。
   示例：cp myfile.txt /path/to/destination（复制文件）；cp -r sourcedir/ destdir/（复制目录）

7. mv：移动文件或目录，或者重命名文件和目录。
   示例：mv myfile.txt /path/to/destination（移动文件）；mv oldname.txt newname.txt（重命名文件）

8. cat：显示文件内容。
   示例：cat myfile.txt

9. grep：在文件中搜索指定的模式。
   示例：grep "pattern" myfile.txt

10. chmod：修改文件或目录的权限。
    示例：chmod 755 myfile.txt（设置文件权限）；chmod -R 755 mydir/（递归设置目录权限）

11. chown：修改文件或目录的所有者。
    示例：chown user1 myfile.txt（更改文件所有者）；chown -R user1 mydir/（递归更改目录所有者）

12. ps：显示当前运行进程的状态。
    示例：ps aux

13. top：实时显示系统资源的使用情况和进程信息。

14. ifconfig：显示和配置网络接口信息。

15. ping：测试与特定主机的网络连通性。
    示例：ping www.example.com

16. wget：从网络上下载文件。
    示例：wget http://www.example.com/file.txt

17. tar：用于创建和提取归档文件。
    示例：tar -cvf archive.tar files（创建归档文件）；tar -xvf archive.tar（提取归档文件）

18. ssh：通过安全的Shell连接远程主机。
    示例：ssh username@hostname

19. sudo：以超级用户权限执行命令。
    示例：sudo apt install package（使用sudo安装软件包）

20. apt-get：用于管理和更新软件包。
    示例：apt-get update（更新软件包列表）；apt-get install package（安装软件包）

这只是一小部分常见的Linux命令，Linux系统提供了丰富的命令行工具和实用程序来满足各种管理和操作需求。可以使用man命令或者在终端中键入命令名加上"--help"来获取更详细的命令使用说明。

## 2、epoll

Epoll是Linux进行IO多路复用的一种方式，用于在一个线程里监听多个IO源，在IO源可用的时候返回并进行操作。它的特点是基于事件驱动，性能很高。

**epoll将文件描述符拷贝到内核空间后使用红黑树进行维护，同时向内核注册每个文件描述符的回调函数，当某个文件描述符可读可写的时候，将这个文件描述符加入到就绪链表里，并唤起进程，返回就绪链表到用户空间，由用户程序进行处理**。

Epoll有三个系统调用：epoll_create(),epoll_ctl()和epoll_wait()。

- eoll_create()函数在内核中初始化一个eventpoll对象，同时初始化红黑树和就绪链表。

- epoll_ctl()用来对监听的文件描述符进行管理。将文件描述符插入红黑树，或者从红黑树中删除，这个过程的时间复杂度是log(N)。同时向内核注册文件描述符的回调函数。

- epoll_wait()会将进程放到eventpoll的等待队列中，将进程阻塞，当某个文件描述符IO可用时，内核通过回调函数将该文件描述符放到就绪链表里，epoll_wait()会将就绪链表里的文件描述符返回到用户空间。



epoll 的一个重要特性是边缘触发（edge-triggered），它只会通知一次事件，直到下一次事件发生才会再次通知。

- ET是边缘触发模式，在这种模式下，只有当描述符从未就绪变成就绪时，内核才会通过epoll进行通知。然后直到下一次变成就绪之前，不会再次重复通知。也就是说，如果一次就绪通知之后不对这个描述符进行IO操作导致它变成未就绪，内核也不会再次发送就绪通知。优点就是只通知一次，减少内核资源浪费，效率高。缺点就是不能保证数据的完整，有些数据来不及读可能就会无法取出。
- LT是水平触发模式，在这个模式下，如果文件描述符IO就绪，内核就会进行通知，如果不对它进行IO操作，只要还有未操作的数据，内核都会一直进行通知。优点就是可以确保数据可以完整输出。**缺点就是由于内核会一直通知**，**会不停从内核空间切换到用户空间，资源浪费严重。**
  

## 3、 IO复用的三种方法（select,poll,epoll）区别和内部原理实现？

### 3.1 select的方法

select把所有监听的文件描述符拷贝到内核中，挂起进程。当某个文件描述符可读或可写的时候，中断程序唤起进程，select将监听的文件描述符再次拷贝到用户空间，然select后遍历这些文件描述符找到IO可用的文件。下次监控的时候需要再次拷贝这些文件描述符到内核空间。select支持监听的描述符最大数量是1024.

### 3.2 poll

poll是对select的改进，**它解决了select中文件描述符集合大小限制问题**。与select类似，poll也可以监视多个文件描述符的可读、可写和异常等事件。但是，相比于select，**poll使用一个pollfd结构数组来传递文件描述符集合，并且没有文件描述符数量的限制**。使用poll的方式与select类似，可以通过遍历pollfd` 结构数组来确定哪些文件描述符已经就绪。



### 3.3 epoll(最高效的方法)

**epoll将文件描述符拷贝到内核空间后使用红黑树进行维护，同时向内核注册每个文件描述符的回调函数，当某个文件描述符可读可写的时候，将这个文件描述符加入到就绪链表里，并唤起进程，返回就绪链表到用户空间**



## 4、查询进程占用CPU的命令

1. top命令查看linux负载：
2. uptime查看linux负载
3. w查看linux负载：
4. vmstat查看linux负载
5. 文件权限怎么看（rwx） ls -l 1.txt    输出： -rw-r--r--  1 user group     0 Sep  1 10:00 example.txt

#### kill，find，cp



## 5、文件的三种时间（mtime, atime，ctime），分别在什么时候会改变

在Linux和其他类Unix系统中，文件具有三个时间戳：修改时间（mtime）、访问时间（atime）和更改时间（ctime）。这些时间戳记录了文件在系统上的不同操作和状态更改。



## 6、写三个线程交替打印ABC

```c++
#include<iostream>
#include<thread>
#include<mutex>
#include<condition_variable>
using namespace std;

mutex mymutex;

//在需要等待条件的线程中，首先获取互斥锁，然后调用 wait() 方法等待条件满足。
//wait() 方法会释放互斥锁，并使当前线程进入等待状态，
//直到其他线程调用 notify_one() 或 notify_all() 方法通知条件满足。
condition_variable cv;//条件变量

int flag=0; //临界资源

void printa(){
    unique_lock<mutex> lk(mymutex); //自动化地获取和释放互斥锁,也可以手动释放，如lk.unlock;
    int count=0;
    while(count<10){
        while(flag!=0) cv.wait(lk); //如果资源不满足，释放锁
        cout<<"thread 1: a"<<endl;
        flag=1;
        cv.notify_all();//通知别人释放了锁
        count++;
    }
    cout<<"my thread 1 finish"<<endl;
}
void printb(){
    unique_lock<mutex> lk(mymutex);
    for(int i=0;i<10;i++){
        while(flag!=1) cv.wait(lk);
        cout<<"thread 2: b"<<endl;
        flag=2;
        cv.notify_all();
    }
    cout<<"my thread 2 finish"<<endl;
}
void printc(){
    unique_lock<mutex> lk(mymutex);
    for(int i=0;i<10;i++){
        while(flag!=2) cv.wait(lk);
        cout<<"thread 3: c"<<endl;
        flag=0;
        cv.notify_all();
    }
    cout<<"my thread 3 finish"<<endl;
}
int main(){
    thread th2(printa);//创建三个线程
    thread th1(printb);
    thread th3(printc);

    th1.join();//阻塞主线程，直到子线程完成； //detach() 用于将线程与线程对象分离，使其在后台运行
    th2.join();
    th3.join();
    cout<<" main thread "<<endl;
}

```

