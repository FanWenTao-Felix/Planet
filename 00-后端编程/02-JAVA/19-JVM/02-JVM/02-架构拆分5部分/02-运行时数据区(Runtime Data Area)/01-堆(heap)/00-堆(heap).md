## java堆

Java 堆用来存放实例化对象，它被所有线程共享，在虚拟机启动时创建，用来存放对象实例，其占用了 Java 内存的大部分空间，是 GC 的主要管理区域

### JDK7及之前
**逻辑上** 可分为`年轻代`、`老年代`、`永久代`。


### JDK8及之后
**逻辑上** 可分为`年轻代`、`老年代`、`元空间`。

### 总结
![](https://youpaiyun.zongqilive.cn/image/20200521142129.png)

![](https://youpaiyun.zongqilive.cn/image/20200318143732.png)

![](https://youpaiyun.zongqilive.cn/image/20200318143641.png)

说明:

- **Java 8**把**永久区**换成了**`元空间`**
- **堆逻辑上由”新生+养老+元空间“三个部分组成，物理上由”新生+养老“两个部分组成**




























































































































