## C: Consistency(强一致性) 

 所有节点同一时间看到是相同的数据；

分布式系统中，一份数据一般会存在多个副本，要求多副本数据对于数据的更新与单份数据相同，即强一致性。

> 任何读请求都必须返回最新数据

## A: Availability(可用性) 

 不管是否成功，确保每一个请求都能接收到响应；

在任意的时刻，对分布式系统来说要保证在限定延时内正常响应客户端的读写请求，并且不会报错。（提前纠正一个误区，这里的可用性并不是咱们通常理解的服务的SLA可用性，即3个9、4个9之类的定义。实际上指的是是数据访问的可达性。）

> 所有读写请求都必须正常响应，可终止、不会一直等待

## P: Partition tolerance(分区容错性) 

系统任意分区后，在网络故障时，仍能操作

在分布式服务中，节点之间网络通信异常导致(丢包、延迟、中断等)的网络分区现象是必然存在的，要保证出现网络分区现象时，分布式系统不受影响。

> 在网络分区的情况下，被分隔的节点仍能正常对外服务

##  CAP理论的核心

一个分布式系统不可能同时很好的满足一致性, 可用性和分区容错性 , 

==最多只能同时满足2个==

为什么说 CAP 只能三选二?

由于网络肯定会出现延迟丢包等问题, 所以 ==分区容错性 是必须要实现的==



CA -- 单点集群, 满足一致性, 可用性, 通常在可扩展性上不太强大

CP -- 满足一致性, 分区容错性的系统, 通常性能不是特别高

AP -- 满足可用性, 分区容错性的系统, 通常可能对一致性要求低一些









![](https://youpaiyun.zongqilive.cn/image/20201116110345.png)

![](https://youpaiyun.zongqilive.cn/image/20200610170605.png)



## CAP的3进2





![](https://youpaiyun.zongqilive.cn/image/20200610170732.png)

![](https://youpaiyun.zongqilive.cn/image/20200610170749.png)







































































