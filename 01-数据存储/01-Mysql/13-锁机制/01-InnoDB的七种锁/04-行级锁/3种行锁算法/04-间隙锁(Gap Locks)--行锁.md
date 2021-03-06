## 什么是间隙锁

gap锁的作⽤ 仅仅是为了防⽌插⼊幻影记录的⽽已。



1. RR级别存在, RC级别不存在

2. 区间锁, 仅仅锁住一个索引区间（开区间，不包括双端端点）。[间隙锁，锁定一个范围，但不包含记录本身]

3. 在索引记录之间的间隙中加锁，或者是在某一条索引记录之前或者之后加锁，并不包括该索引记录本身。比如在 1、2、3中，间隙锁的可能值有 (∞, 1)，(1, 2)，(2, ∞)，

4. 间隙锁可用于`防止幻读`，保证索引间的不会被插入数据 

5. gap lock可以共存(co-exist)。事务T1持有某个间隙上的gap lock 并不能阻止 事务T2同时持有 同一个间隙上的gap lock。shared gap lock和exclusive gap lock并没有任何的不同，它俩并不冲突，它俩执行同样的功能

6. 在索引记录上查找给定记录时，InnoDB会在`第一个不满足查询条件的记录`上加gap lock，防止新的满足条件的记录插入![](https://youpaiyun.zongqilive.cn/image/20200704110957.png)

   上图演示了：InnoDB在索引上扫描时，找到了c2=11的记录，然后，InnoDB接着扫描，它发现下一条记录是c2=18，不满足条件，InnoDB遇到了第一个不满足查询条件的记录18，于是InnoDB在18上设置gap lock，此gap lock锁定了区间(11, 18)。

   

7. 



用范围条件而不是相等条件查询数据,  并请求共享或者排他锁时, Innodb会给符合条件的已有数据记录的索引加锁;

对于键值在条件范围内但是并不存在的记录, 叫做`间隙`(GAP)

![](https://youpaiyun.zongqilive.cn/image/20200226120859.png)

InnoDb 会对这个`间隙`加锁, 这种锁机制就是间隙锁

如: `delete from tableX where id between 2 and 10;`

在Repeatable read, 的隔离级别，id为唯一主键的条件下，将锁住 2到10之间的间隙，如果其他事务需要插入主键是2到10之间的记录，将在队列中等待。

间隙锁的主要目的是为了防止`幻读`的发生，也就是，防止同一事务中，两次读取的记录数不一致。或者说，在对一个范围的记录进行删除或更新时，防止在事务结束前，其他事务插入也再这个范围的记录。

![](https://youpaiyun.zongqilive.cn/image/20200226120921.png)











假设 有表person，字段有id, name。隔离级别为 Repeatable read。

表内容：

| id   | name |
| ---- | ---- |
| 2    | Ray  |
| 7    | Mike |

 

- id 为主键或唯一键：

1. `update person set name = ‘XX’ where id > 5`

   间隙锁加在(5, 7) 和(7,正无穷)

2. `update person set name = ‘XX’ where id < 5`

   间隙锁加在(负无穷, 2) 和 (2, 5)

3. `update person set name = ‘XX’ where id > 5 and id <9`

   间隙锁加在(5, 7)和(7,9)

- Id 为非唯一键，即使是条件为等于，也会产生间隙锁（使用了部分组合索引的情况除外）

  1.`update person set name = ‘XX’ where id = 5`

  间隙锁加在(2, 5),(5,5), (5, 7) , 键值5上加了排他锁，键值5对应的主键也加了排他锁。至于为什么不只把5这个值锁住，还需要深入研究一下。

## 危害

因为query执行的过程中通过范围查找, 他会锁定整个范围内所有的索引键值, 即使这个键值并不存在.

间隙锁有一个比较致命的弱点, 就是当锁定一个范围值之后, 即使某些不存在的键值也会被无辜的锁定.而造成锁定的时候无法插入锁定键值范围内的任何数据, 在某些场景下可能会造成很大危害.