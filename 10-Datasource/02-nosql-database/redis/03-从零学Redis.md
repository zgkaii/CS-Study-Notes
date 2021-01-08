# 前言

> 只有光头才能变强

看过相关Redis基础的同学可以知道**Redis是单线程的**，很多面试题也很可能会问到“为什么Redis是单线程的还那么快”。

这篇文章来讲讲单线程的**内部的原理**。

文本**力求简单讲清每个知识点**，希望大家看完能有所收获

# 一、基础铺垫

在讲解Redis之前，我们先来一些基础的铺垫，有更好的阅读体验。

## 1.1网路编程

我们在初学Java的时候肯定会学过网络编程这一章节的，当时学完写的应用可能就是“网络聊天室”。

写出来的效果可能就是在console噼里啪啦的输入数据，然后噼里啪啦的返回数据，就完事了..(扎心了)

网络编程可简单分为TCP和UPD两种，一般我们更多关注的是TCP。TCP网络编程在Java中封装成Socket和SocketServer，我们来回顾一下最简单的TCP网络编程吧：

TCP客户端

```java
public class ClientDemo {
    public static void main(String[] args) throws IOException {
        //创建发送端的Socket对象
        Socket s = new Socket("192.168.1.106",8888);

        //Socket对象可以获取输出流
        OutputStream os = s.getOutputStream();
        os.write("hello,tcp,我来了".getBytes());

        s.close();
    }
}
```

TCP服务端：

```java
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        //创建接收端的Socket对象
        ServerSocket ss = new ServerSocket(8888);

        //监听客户端连接，返回一个对应的Socket对象
        //侦听并接受到此套接字的连接，此方法会阻塞
        Socket s = ss.accept();

        //获取输入流，读取数据
        InputStream is = s.getInputStream();

        byte[] bys = new byte[1024];
        int len = is.read(bys);
        String str = new String (bys,0,len);

        String ip = s.getInetAddress().getHostAddress();
        System.out.println(ip + "    ---" +str);

        //释放资源
        s.close();
        //ss.close();  

    }
}
```

上面的代码就可以实现：**客户端向服务器发送数据，服务端能够接收客户端发送过来的数据**。

## 1.2IO多路复用

之前我已经写过Java NIO的文章了，Java的NIO也是基于IO多路复用模型的，建议先去看一下再回来，文章写得挺详细和通俗的了：[JDK10都发布了，nio你了解多少？](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&token=1834317504&lang=zh_CN&scene=21#wechat_redirect)

这里就简单回顾一下吧：

- I/O多路复用的特点是通过一种机制**一个进程能同时等待多个文件描述符**，而这些文件描述符其中的任意一个进入**读就绪状态、等等**，`select()`函数就可以返回。
- select/epoll的优势并不是对于单个连接能处理得更快，而是**在于能处理更多的连接**。

说白了，使用IO多路复用机制的，一般自己会有一套**事件机制**，使用**一个**线程或者进程监听这些事件，如果这些事件被触发了，则调用对应的函数来处理。

# 二、Redis事件

Redis服务器是一个**事件驱动程序**，主要处理以下两类事件：

- 文件事件：文件事件其实就是**对Socket操作的抽象**，Redis服务器与Redis客户端的通信会产生文件事件，服务器通过监听并处理这些事件来完成一系列的网络操作
- 时间事件：时间事件其实就是对**定时操作的抽象**，前面我们已经讲了RDB、AOF、定时删除键这些操作都可以由服务端去定时或者周期去完成，底层就是通过触发时间事件来实现的！

## 2.1文件事件

Redis开发了自己的网络事件处理器，这个处理器被称为**文件事件处理器**。

文件事件处理器由四部分组成：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCneyf5NgvvFboFxeCPd9mmb58QC9tKQeMZKibsPllxlfGSp0tm1u1OEyCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)文件事件处理器组成

> 文件事件处理器使用I/O多路复用程序来**同时监听多个Socket**。当被监听的Socket准备好执行连接应答(accept)、读取(read)等等操作时，与操作相对应的文件事件就会产生，根据文件事件来为Socket关联对应的事件处理器，从而实现功能。

要值得注意的是：Redis中的I/O多路复用程序会将所有**产生事件的Socket放到一个队列**里边，然后通过这个队列以有序、同步、每次一个Socket的方式向文件事件分派器传送套接字。也就是说：当上一个Socket处理完毕后，I/O多路复用程序才会向文件事件分派器传送下一个Socket。

首先，IO多路复用程序首先会监听着Socket的`AE_READABLE`事件，该事件对应着连接应答处理器

- 可以理解简单成`SocketServet.accpet()`

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCneq1LrKtkoxxRUFLbqXCFMLNknW2RuoLS5S8Xuv8pDQvic38DcMWAu7Ow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)监听着Socket的AE_READABLE事件

此时，一个名字叫做3y的Socket要连接服务器啦。服务器会用**连接应答处理器**处理。创建出客户端的Socket，并将客户端的Socket与命令请求处理器进行关联，使得客户端可以向服务器发送命令请求。

- 相当于`Socket s = ss.accept();`，创建出客户端的Socket，然后将该Socket关联**命令请求处理器**
- 此时客户端就可以向主服务器发送命令请求了

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCneVCdvjBuPIhLLOTBkwy99ibrib3ZXQp1ljdMDj2y0oA0C7FwQQCapw4Cw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)客户端请求连接，服务器创建出客户端Scoket，关联命令请求处理器

假设现在客户端发送一个命令请求`set Java3y "关注、点赞、评论"`，客户端Socket将产生`AE_READABLE`事件，**引发命令请求处理器执行**。处理器读取客户端的命令内容，然后传给对应的程序去执行。

客户端发送完命令请求后，**服务端总得给客户端回应的**。此时服务端会将客户端的Scoket的`AE_WRITABLE`事件与命令回复处理器关联。

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCnetxNrVcsQ8QqfFgHkF2tZic83fy6nPEuib75NZIqLl0UhBYKhvcme3v6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)客户端的Scoket的AE_WRITABLE事件与命令回复处理器关联

最后客户端尝试读取命令回复时，客户端Socket**产生**AE_WRITABLE事件，触发命令回复处理器执行。当把所有的回复数据写入到Socket之后，服务器就会**解除**客户端Socket的AE_WRITABLE事件与命令回复处理器的关联。

最后以《Redis设计与实现》的一张图来概括：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCneh0AD017J6kHzSkNBicSInjyo75QER2hruqQyc7zYHANVh4mmYVHVKOw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Redis事件交互过程

## 2.2时间事件

持续运行的Redis服务器会**定期**对自身的资源和状态进行检查和调整，这些定期的操作由**serverCron**函数负责执行，它的主要工作包括：

- 更新服务器的统计信息(时间、内存占用、数据库占用)
- 清理数据库的过期键值对
- AOF、RDB持久化
- 如果是主从服务器，对从服务器进行定期同步
- 如果是集群模式，对进群进行定期同步和连接
- …

Redis服务器将时间事件放在一个**链表**中，当时间事件执行器运行时，会遍历整个链表。时间事件包括：

- **周期性事件**(Redis一般只执行serverCron时间事件，serverCron时间事件是周期性的)
- 定时事件

## 2.3时间事件和文件事件

- 文件事件和时间事件之间是**合作**关系，服务器会轮流**处理**这两种事件，并且处理事件的过程中不会发生抢占。
- 时间事件的实际处理事件通常会比设定的到达时间**晚**一些

# 三、Redis单线程为什么快？

- 1）纯内存操作
- 2）核心是基于非阻塞的IO多路复用机制
- 3）单线程避免了多线程的频繁上下文切换问题

# 四、客户端与服务器

在《Redis设计与实现》中各用了一章节来写客户端与服务器，我看完觉得比较底层的东西，也很难记得住，所以我决定**总结**一下比较重要的知识。如果以后真的遇到了，再来补坑~

服务器使用clints**链表**连接多个客户端状态，新添加的客户端状态会被放到链表的末尾

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2qAQiartrqRhynB9qVVPCnejR51wFLibPrGicdibBAPo6KSJLBZeDfFC8BGWJkXJISXf5NW87vqIJh3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)客户端--链表

- 一个服务器可以与多个客户端建立网络连接，每个**客户端可以向服务器发送命令请求**，而服务器则接收并处理客户端发送的命令请求，并向客户端返回命令回复。
- Redis服务器使用**单线程单进程**的方式处理命令请求。在数据库中保存**客户端执行命令所产生的数据**，并通过**资源管理来维持**服务器自身的运转。

## 4.1客户端

客户端章节中主要讲解了**Redis客户端的属性**(客户端状态、输入/输出缓冲区、命令参数、命令函数等等)

```
typedef struct redisClient{

    //客户端状态的输入缓冲区用于保存客户端发送的命令请求,最大1GB,否则服务器将关闭这个客户端
    sds querybuf;  


    //负责记录argv数组的长度。
    int argc;   

    // 命令的参数
    robj **argv;  

    // 客户端要执行命令的实现函数
    struct redisCommand *cmd, *lastcmd;  


    //记录了客户端的角色(role),以及客户端所处的状态。 (REDIS_SLAVE | REDIS_MONITOR | REDIS_MULTI) 
    int flags;             

    //记录客户端是否通过了身份验证
    int authenticated;     

    //时间相关的属性
    time_t ctime;           /* Client creation time */       
    time_t lastinteraction; /* time of the last interaction, used for timeout */
    time_t obuf_soft_limit_reached_time;


    //固定大小的缓冲区用于保存那些长度比较小的回复
    /* Response buffer */
    int bufpos;
    char buf[REDIS_REPLY_CHUNK_BYTES];

    //可变大小的缓冲区用于保存那些长度比较大的回复
    list *reply; //可变大小缓冲区由reply 链表和一个或多个字符串对象组成
    //...
}
```

## 4.2服务端

**服务器章节**中主要讲解了Redis服务器读取客户端发送过来的命令是如何解析，以及初始化的过程。

服务器从启动到能够处理客户端的命令请求需要执行以下的步骤：

- 初始化服务器状态
- 载入服务器配置
- 初始化服务器的数据结构
- 还原数据库状态
- 执行事件循环

总的来说是这样子的：

```
def main():

    init_server();

    while server_is_not_shutdown();
        aeProcessEvents()

    clean_server();
```

从客户端发送命令道完成主要包括的步骤：

- 客户端将命令请求发送给服务器
- 服务器读取命令请求，分析出**命令参数**
- 命令执行器根据参数**查找命令的实现函数**，执行实现函数并得出命令回复
- 服务器将命令回复返回给客户端

# 五、最后

现在临近双十一买阿里云服务器就**特别省钱**！之前我买**学生机也要9.8**块钱一个月，现在最低价只需要**8.3**一个月！

无论是Nginx/Elasticsearch/Redis这些技术都是在Linux下完美运行的，如果还是程序员新手，买一个学习Linux基础命令，学习搭建环境也是不错的选择。

如果有要买服务器的同学可通过我的链接**直接享受最低价**：https://m.aliyun.com/act/team1111/#/share?params=N.FF7yxCciiM.pfn5xpli

------

本来也想把“复制”(主从)在这边一起写的，但写完可能就很长了，所以留到下一篇吧。

如果大家有更好的理解方式或者文章有错误的地方还请大家不吝在评论区留言，大家互相学习交流~~~

参考资料：

- 《Redis设计与实现》
- 《Redis实战》

> 一个**坚持原创的Java技术公众号：Java3y**，欢迎大家关注