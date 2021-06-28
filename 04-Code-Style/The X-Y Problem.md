> 此文章来源：https://coolshell.cn/articles/10804.html

## X-Y Problem

对于X-Y Problem的意思如下：

1）有人想解决问题X
2）他觉得Y可能是解决X问题的方法
3）但是他不知道Y应该怎么做
4）于是他去问别人Y应该怎么做？

简而言之，**没有去问怎么解决问题X，而是去问解决方案Y应该怎么去实现和操作**。于是乎：

1）热心的人们帮助并告诉这个人Y应该怎么搞，但是大家都觉得Y这个方案有点怪异。
2）在经过大量地讨论和浪费了大量的时间后，热心的人终于明白了原始的问题X是怎么一回事。
3）于是大家都发现，Y根本就不是用来解决X的合适的方案。

X-Y Problem最大的严重的问题就是：**在一个根本错误的方向上浪费他人大量的时间和精力**！

### 示例

举个两个例子：

> Q) 我怎么用Shell取得一个字符串的后3位字符？
> A1) 如果这个字符的变量是$foo，你可以这样来 echo ${foo:-3}
> A2) 为什么你要取后3位？你想干什么？
> Q) 其实我就想取文件的扩展名
> A1) 我靠，原来你要干这事，那我的方法不对，文件的扩展名并不保证一定有3位啊。
> A1) 如果你的文件必然有扩展名的话，你可以这来样来：echo ${foo##*.}



再来一个示例：

> Q）问一下大家，我如何得到一个文件的大小
> A1)  size = `ls -l $file  | awk ‘{print $5}’`
> Q) 哦，要是这个文件名是个目录呢？
> A2) 用du吧
> A3) 不好意思，你到底是要文件的大小还是目录的大小？你到底要干什么？
> Q)  我想把一个目录下的每个文件的每个块（第一个块有512个字节）拿出来做md5，并且计算他们的大小 ……
> A1) 哦，你可以使用dd吧。
> A2) dd不行吧。
> A3) 你用md5来计算这些块的目的是什么？你究竟想干什么啊？
> Q) 其实，我想写一个网盘，对于小文件就直接传输了，对于大文件我想分块做增量同步。
> A2) 用rsync啊，你妹！

[这里有篇文章](http://www.perlmonks.org/index.pl?node_id=542341)说明了X-Y Problem的各种案例说明，我从其中摘出三个来让大家看看：

> 你试图做X，并想到了用Y方案。所以你去问别人Y，但根本不提X。于是，你可以会错过本来可能有更好更适合的方案，除非你告诉大家X是什么。
>
> — *from [Re: How do I keep the command line from eating the backslashes?](http://www.perlmonks.org/index.pl?node_id=430320) by [revdiablo](http://www.perlmonks.org/index.pl?node_id=163683)*

> 有些人问怎么做Y，但其它他想做的是X。他问怎么做Y是因为他觉得Y是最好搞定X的方法。 于是大家不断地回答“试试这个，试试那个”来帮助他，而他总是在说“这个有问题，那个有问题，因为……”。基本不同的情况，其它的方案可能会更好。
>
> — *from [Re: Re: Re: Re: regex to validate e-mail addresses and phone numbers](http://www.perlmonks.org/index.pl?node_id=327963) by [Limbic~Region](http://www.perlmonks.org/index.pl?node_id=180961)*

> X-Y Problem又叫“过早下结论”：提问者其实并不非常清楚想要解决的X问题，他猜测用Y可以搞定，于是他问大家如何实现Y。
>
> — *from [](http://groups.google.com/groups?hl=en&selm=Pine.GHP.4.21.0009061210570.8800-100000@hpplus03.cern.ch) by Alan J. Flavell*

其实这个问题在我之前的《[你会问问题吗](https://coolshell.cn/articles/3713.html)》里提到的那篇How To Ask Questions the Smart Way中的提到过，你可以[移步去看一下](http://www.beiww.com/doc/oss/smart-questions.html#id265951)。

所以，我们在寻求别人帮助的时候，最好把我们想解决的问题和整个事情的来龙去脉说清楚。

### 一些变种

我们不要以为X-Y Problem就像上面那样的简单，我们不会出现，其实我们生活的这个世界有有各种X-Y Problem的变种。下面我个人觉得非常像XY Problem的总是：

其一、大多数人有时候，非常容易把手段当目的，他们会用自己所喜欢的技术和方法来反推用户的需求，于是很有可能就会出现X-Y Problem – 也许解决用户需求最适合的技术方案是PC，但是我们要让他们用手机。

其二、产品经理有时候并不清楚他想解决的用户需求是什么，于是他觉得可能开发Y的功能能够满足用户，于是他提出了Y的需求让技术人员去做，但那根本不是解决X问题的最佳方案。

其三、因为公司或部门的一些战略安排，业务部门设计了相关的业务规划，然后这些业务规划更多的是公司想要的Y，而不是解决用户的X问题。

其四、对于个人的职业发展，X是成长为有更强的技能和能力，这个可以拥有比别人更强的竞争力，从而可以有更好的报酬，但确走向了Y：全身心地追逐KPI。

其五、本来我们想达成的X是做出更好和更有价值的产品，但最终走到了Y：通过各种手段提升安装量，点击量，在线量，用户量来衡量。

其六、很多团队Leader都喜欢制造信息不平等，并不告诉团队某个事情的来由，掩盖X，而直接把要做的Y告诉团队，导致团队并不真正地理解，而产生了很多时间和经历的浪费。

所有的这些，在我心中都是X-Y Problem的变种，这是不是一种刻舟求剑的表现？

### 参考

- [StackOverflow: What is XY Problem?](http://meta.stackoverflow.com/questions/66377/what-is-the-xy-problem)
- [PerlMonks: XY Problem](http://www.perlmonks.org/?node_id=542341)
- [Greg’s Wiki](http://mywiki.wooledge.org/XyProblem)