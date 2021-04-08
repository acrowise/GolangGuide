# GolangGuide	制作人——无__忧
总结了golang常见的面试题，汇总了一些资料提供查看，后续会继续补充完善，欢迎大家star~:smiley:

![go_monkey](https://raw.githubusercontent.com/zmk-c/GolangGuide/master/img/20210403192938.jpeg)

# [1.基础知识归纳](golang/base.md)

# [2.常见类型源码分析](golang/advance.md)

- 2.1深度解密Go语言之slice
- 2.2深度解密Go语言之map
- 2.3深度解密语言之channel
- 2.4深度解密Go语言之context
- 2.5深度解密Go语言之unsafe
- 2.6深度解密Go语言之反射
- 2.7深度解密Go语言之关于interface 的10个问题
- 2.8图解Go语言内存分配
- 2.9Go程序是怎样跑起来的
- 2.10Go语言的GPM调度器是什么？

# [3.常见面试题](golang/FAQ.md)

- 3.1Q：字符串转换成byte数组，会发生内存拷贝吗?有没有什么办法可以在转换时不发生拷贝呢?
- 3.2Q：能说说uintptr和unsafe.Pointer的区 别吗?
- 3.3Q：拷贝大切片-定比小切片代价大吗?
- 3.4Q：知道Golang的内存逃逸吗?什么情况下回发生内存逃逸?
- 3.5Q：怎么避免逃逸分析?
- 3.6Q：reflect (反射包)如何获取字段tag? 为什么json包不能导出私有变量的tag?
- 3.7Q：对已经关闭的chan进行读写会怎么样?为什么?
- 3.8Q：对未初始化的chan进行读写，会怎么样?为什么?
- 3.9Q：for循环select时， 如果通道关闭会怎么样?如果select中的case只有一 个，又会怎么样?
- 3.10Q：Go语言并发题目测试

# 4.常见算法题

- [4.1《剑指offer》解析](https://github.com/zmk-c/go-offer)
- [4.2leetcode刷题顺序解析](https://github.com/zmk-c/leetcode)
- 4.3经典排序算法

# 5.其他

- 5.1Redis相关
  - [5.1.1Redis基础结构](redis/base.md)
  - [5.1.2Redis的底层数据结构](redis/under.md)
  - [5.1.3Redis持久化原理及优化](redis/persistence.md)
  - [5.1.4Redis中内存淘汰算法实现](redis/algorithm.md)
  - [5.1.5Redis中主从复制原理](redis/policy.md)
- 5.2MySQL相关
  - [5.2.1MySQL数据库经典面试题解析](mysql/mysql100.md)
  - [5.2.2MySQL InnoDB MVCC 机制的原理及实现](mysql/mysql-mvcc.md)
  - [5.2.3为什么MySQL使用B+树做索引？](mysql/mysql-B+.md)
- 5.3操作系统相关
- 5.4计算机网络相关
  - [5.4.1计算机网络基础](network/network.md)
  - [5.4.2在B站看猫片被老板发现？不如按下F12学学HTTP](https://mp.weixin.qq.com/s/T41YBEmG4lkxokDLzRxVgA)
  - [5.4.3TCP粘包 数据包：我只是犯了每个数据包都会犯的错 |硬核图解](https://mp.weixin.qq.com/s/0H8WL6QeZ2VbO1hHPkn8Ug)
- [5.5Git相关](git/git.md)
- [5.6Docker相关](docker/docker.md)
- 5.7Kubernetes相关
- 5.8常见共识算法
  - [5.8.1raft协议](consensus/raft.md)
  - [5.8.2PBFT (Practical Byzantine Fault Tolerance,实用拜占庭容错)](consensus/pbft.md)
  - [5.8.3Gossip协议](consensus/gossip.md)

# 参考

- 公众号:
  - Go语言中文网
  - golang小白成长记
  - 码农桃花源

- 网址：

  - [共识算法：Raft](https://www.jianshu.com/p/8e4bbe7e276c)
  - [Raft原理动画](http://thesecretlivesofdata.com/raft/)
  - [100道MySQL数据库经典面试题解析（收藏版）](https://juejin.im/post/5ec15ab9f265da7bc60e1910 )
  - [再有人问你为什么MySQL用B+树做索引，就把这篇文章发给她](https://mp.weixin.qq.com/s?__biz=MzIwOTE2MzU4NA==&mid=2247484085&idx=1&sn=92639430ac7ef3e412b550a09bde0115&chksm=9779469aa00ecf8c157e899fe0d5c5b060b282a4e5f2f2f63c187eb3c04d04ef6fad7a1e09e3&token=472896045&lang=zh_CN#rd)
  - [你真的懂MVCC吗？来手动实践一下？](https://juejin.im/post/5da8493ae51d4524b25add55)
  - [一文看懂Redis的持久化原理](https://juejin.im/post/5b70dfcf518825610f1f5c16)
  -  [Redis中的数据结构](https://www.cnblogs.com/neooelric/p/9621736.html)
  -  [Redis中内存淘汰算法实现](http://fivezh.github.io/2019/01/10/Redis-LRU-algorithm/)
  -  [写给大忙人的Redis主从复制，花费五分钟让你面试不尴尬](https://juejin.im/post/5ed5ccb66fb9a047df7ca9a4)
  
- 论文：

  - Ongaro D, Ousterhout J. In search of an understandable consensus algorithm［C］// USENIX Annual Technical Conference. [s.l.]: USENIX. 2014: 305-319．
