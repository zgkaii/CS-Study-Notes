# 前言

> 只有光头才能变强

这篇文章主要讲的是**Redis主从复制**。因为Redis集群的知识点有点多，所以铂金上分得要好几篇~

文本**力求简单讲清每个知识点**，希望大家看完能有所收获

# 一、主从架构

## 1.1为什么要主从架构

Redis也跟关系型数据(MySQL)一样，如果有过多请求还是撑不住的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYZVWvXXh3ukvQibzSEhsVuNrnCn6jCL5YcS9mBXt3gCDNBEs2CUgBRxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)一台Redis撑

因为Redis如果只有一台服务器的话，那随着请求越来越多：

- **Redis的内存是有限的**，可能放不下那么多的数据
- 单台Redis**支持的并发量也是有限的**。
- **万一这台Redis挂了**，所有的请求全走关系数据库了，那就更炸了。

显然，出现的上述问题是因为一台Redis服务器不够，所以多搞几台Redis服务器就可以了

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzY4B9rmybBDy74twGvHu0I1nt5ibu6yGa4KOFt8rTiarpLgak7sxOAsrrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)多搞几台Redis服务器

为了实现我们服务的**高可用性**，可以将这几台Redis服务器做成是**主从**来进行管理

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYOwG8zdc4bStcnyVZswgKEMt4qpHUZyiblYibgWut87TGMutWUJm9ZbLA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)主从架构

> tip:Redis作者已将Master/Slave架构改名为Master/Replica

## 1.2主从架构的特点

下面我们来看看Redis的主从架构特点：

- **主**服务器负责接收**写**请求
- **从**服务器负责接收**读**请求
- 从服务器的数据由主服务器**复制**过去。主从服务器的数据是**一致**的

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzY9Kqt3bnM4PNALGwgj56NKOE3iaRoaWujgPAgkDFpKEwTxSOCjibrlD5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)主从架构特点

主从架构的**好处**：

- 读写分离(主服务器负责写，从服务器负责读)
- 高可用(某一台从服务器挂了，其他从服务器还能继续接收请求，不影响服务)
- 处理更多的并发量(每台从服务器**都可以接收读请求**，读QPS就上去了)

主从架构除了上面的形式，也有下面这种的(只不过用得比较少)：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYn17yEBcp9tSagay2Gj2wbhqsaibWByuicPtWo7pviaGbfvHuRGSbic9Qgg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)从服务器又挂着从服务器

# 二、复制功能

主从架构的特点之一：主服务器和从服务器的数据是**一致**的。

因为主服务器是能接收写请求的，主服务器处理完写请求，会做什么来保证主从数据的一致性呢？如果主从服务器断开了，过一阵子才重连，又会怎么处理呢？下面将会了解到这些细节~

> 在Redis中，用户可以通过执行SALVEOF命令或者设置salveof选项，让一个服务器去复制(replicate)另一个服务器，我们称呼被复制的服务器为主服务器(master)，而对主服务器进行复制的服务器则被称为从服务器(salve)

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYbRzTTaq01WK2cuA7AYyFGT8qrzibib8MibMIQ7I24yNEZEqv9qf4DCHBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)复制

## 2.1复制功能的具体实现

复制功能分为两个操作：

- 同步(sync)

- - 将从服务器的数据库状态**更新至**主服务器的数据库状态

- 命令传播(command propagate)

- - 主服务器的数据库状态**被修改**，导致主从服务器的数据库状态**不一致**，让主从服务器的数据库状态**重新回到一致状态**。

![主从数据一致性](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)主从数据一致性

从服务器对主服务器的**同步又可以分为两种情况**：

- 初次同步：从服务器**没有复制过任何**的主服务器，或者从服务器要复制的主服务器跟上次复制的主服务器**不一样**。
- 断线后同步：处于**命令传播阶段**的主从服务器因为**网络原因**中断了复制，从服务器通过**自动重连**重新连接主服务器，并继续复制主服务器

在Redis2.8以前，断线后复制这部分其实缺少的只是**部分的数据**，但是要让主从服务器**重新执行SYNC命令**，这样的做法是非常低效的。(因为执行SYNC命令是把**所有的数据**再次同步，而不是只同步丢失的数据)

接下来我们来详细看看Redis2.8以后复制功能是怎么实现的：

### 2.1.1复制的前置工作

首先我们来看一下**前置的工作**：

- 从服务器设置主服务器的IP和端口
- 建立与主服务器的Socket连接
- 发送PING命令(检测Socket读写是否正常与主服务器的通信状况)
- 身份验证(看有没有设置对应的验证配置)
- 从服务器给主服务器发送端口的信息，主服务器记录监听的端口

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYBqRh72uVGvQktTCorTYWwKEDGIibCAGjsEleZnujDibT2BwTaadUz7yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Redis复制的前置工作

前面也提到了，Redis2.8之前，断线后同步会重新执行SYNC命令，这是非常**低效**的。下面我们来看一下Redis2.8之后是怎么进行同步的。

> Redis从2.8版本开始，使用PSYNC命令来**替代**SYNC命令执行复制时同步的操作。

PSYNC命令具有**完整**重同步和**部分**重同步两种模式(其实就跟上面所说的初次复制和断线后复制差不多个意思)。

### 2.1.2完整重同步

下面先来看看**完整**重同步是怎么实现的：

- 从服务器向主服务器发送PSYNC命令
- 收到PSYNC命令的主服务器执行BGSAVE命令，在后台**生成一个RDB文件**。并用一个**缓冲区**来记录从现在开始执行的所有**写命令**。
- 当主服务器的BGSAVE命令执行完后，将生成的RDB文件发送给从服务器，**从服务器接收和载入RBD文件**。将自己的数据库状态更新至与主服务器执行BGSAVE命令时的状态。
- 主服务器将所有缓冲区的**写命令发送给从服务器**，从服务器执行这些写命令，达到数据最终一致性。

![完整重同步](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)完整重同步

### 2.1.2部分重同步

接下来我们来看看**部分**重同步，部分重同步可以让我们断线后重连**只需要同步缺失的数据**(而不是Redis2.8之前的同步全部数据)，这是符合逻辑的！

部分重同步功能由以下部分组成：

- 主从服务器的**复制偏移量**
- 主服务器的**复制积压缓冲区**
- 服务器运行的ID(**run ID**)

首先我们来解释一下上面的名词：

复制偏移量：执行复制的双方都会**分别维护**一个复制偏移量

- 主服务器每次传播N个字节，就将自己的复制偏移量加上N
- 从服务器每次收到主服务器的N个字节，就将自己的复制偏移量加上N

通过**对比主从复制的偏移量**，就很容易知道主从服务器的数据是否处于一致性的状态！

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYpILEleTn1tctRfXcianicheNcbk43o1bWMZh0qDjjHGEIbMyyAlb9xNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)复制偏移量

那断线重连以后，从服务器向主服务器发送PSYNC命令，报告现在的偏移量是36，那么主服务器该对从服务器执行完整重同步还是部分重同步呢？？这就交由**复制积压缓冲区**来决定。

当主服务器进行命令传播时，不仅仅会将写命令发送给所有的从服务器，还会将写命令**入队到复制积压缓冲区**里面(这个大小可以调的)。如果复制积压缓冲区**存在**丢失的偏移量的数据，那就执行部分重同步，否则执行完整重同步。

服务器运行的ID(**run ID**)实际上就是用来比对ID是否相同。如果不相同，则说明从服务器断线之前复制的主服务器和当前连接的主服务器是两台服务器，这就会进行完整重同步。

所以流程大概如此：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3ZO9Sp7eLYY6pHbicfkMlzYpD7lobZpNv0oPdGKA5Fcw6dJp3SGLl8ibFXicLJk34YogtJ4EwemK0tA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)同步的流程

### 2.1.3命令传播

> 当完成了同步之后，主从服务器就会进入命令传播阶段。这时主服务器只要将自己的写命令发送给从服务器，而从服务器接收并执行主服务器发送过来的写命令，就可以保证主从服务器一直保持数据一致了！

在命令传播阶段，从服务器默认会以每秒一次的频率，向服务器发送命令`REPLCONF ACK <replication_offset>` 其中replication_offset是从服务器当前的复制偏移量

发送这个命令主要有三个作用：

- **检测主从服务器的网络状态**
- 辅助实现min-slaves选项
- 检测命令丢失

# 五、最后

画了好久好久的图，终于写完啦。

> 抛个问题：如果从服务器挂了，没关系，我们一般会有多个从服务器，其他的请求可以交由没有挂的从服务器继续处理。如果主服务器挂了，怎么办？因为我们的写请求由主服务器处理，只有一台主服务器，那就无法处理写请求了？

问题留到下篇解决~~

参考资料：

- 《Redis设计与实现》
- 《Redis实战》