⼀个数据⻚中存储的记录的状态信 息，⽐如本⻚中已经存储了多少条记录，第⼀条记录的地址是什么， ⻚⽬录中存储了多少个槽等等，特意在⻚中定义了⼀个叫Page Header的部分，它是⻚结构的第⼆部分，这个部分占⽤固定的56个 字节，专⻔存储各种状态信息，具体各个字节都是⼲嘛的看下表：



![](https://youpaiyun.zongqilive.cn/image/20200901140424.png)

![](https://youpaiyun.zongqilive.cn/image/20200901140456.png)

- PAGE_DIRECTION

  假如新插⼊的⼀条记录的主键值⽐上⼀条记录的主键值⼤，我 们说这条记录的插⼊⽅向是右边，反之则是左边。⽤来表示最 后⼀条记录插⼊⽅向的状态就是PAGE_DIRECTION。

- PAGE_N_DIRECTION

  假设连续⼏次插⼊新记录的⽅向都是⼀致的，InnoDB会把沿 着同⼀个⽅向插⼊记录的条数记下来，这个条数就 ⽤PAGE_N_DIRECTION这个状态表示。当然，如果最后⼀条 记录的插⼊⽅向改变了的话，这个状态的值会被清零重新统 计。

































































