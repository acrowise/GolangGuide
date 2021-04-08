# 5.1.1Redis 基础

## Redis常见的数据结构？

String、Hash、List、Set、SortedSet。

### 1.String 字符串类型

是redis中最基本的数据类型，一个key对应一个value。
String类型是二进制安全的，意思是 redis 的 string 可以包含任何数据。如数字，字符串，jpg图片或者序列化的对象。

**实战场景：**

1. 缓存： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
2. 计数器：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
3. session：常见方案spring session + redis实现session共享

### 2.Hash （哈希）

是一个Mapmap，指值本身又是一种键值对结构，如 value={{field1,value1},......fieldN,valueN}}

![image-20210408152952343](https://raw.githubusercontent.com/zmk-c/GolangGuide/master/img/20210408152952.png)

**实战场景：**

1.缓存： 能直观，相比string更节省空间，的维护缓存信息，如用户信息，视频信息等。

###  3.链表

List 说白了就是链表（redis 使用双端链表实现的 List），是有序的，value可以重复，可以通过下标取出对应的value值，左右两边都能进行插入和删除数据。

![image-20210408153043570](https://raw.githubusercontent.com/zmk-c/GolangGuide/master/img/20210408153043.png)

使用列表的技巧

- lpush+lpop=Stack(栈)
- lpush+rpop=Queue（队列）
- lpush+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）

**实战场景：**

1.timeline：例如微博的时间轴，有人发布微博，用lpush加入时间轴，展示新的列表信息。

### 4.Set   集合

集合类型也是用来保存多个字符串的元素，但和列表不同的是集合中  1. 不允许有重复的元素，2.集合中的元素是无序的，不能通过索引下标获取元素，3.支持集合间的操作，可以取多个集合取交集、并集、差集。


![image-20210408153059921](https://raw.githubusercontent.com/zmk-c/GolangGuide/master/img/20210408153059.png)

**实战场景;**

1. 标签（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。
2. 点赞，或点踩，收藏等，可以放到set中实现

### 5.zset  有序集合

有序集合和集合有着必然的联系，保留了集合不能有重复成员的特性，区别是，有序集合中的元素是可以排序的，它给每个元素设置一个分数，作为排序的依据。

（有序集合中的元素不可以重复，但是score 分数 可以重复，就和一个班里的同学学号不能重复，但考试成绩可以相同）。


![image-20210408153130251](https://raw.githubusercontent.com/zmk-c/GolangGuide/master/img/20210408153130.png)
