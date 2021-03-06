`simple dynamic string`, 用作默认字符串表示

## 结构

**简单动态字符串SDS的结构**:

```c
struct sdshdr 
{
// 字节数组，用于保存字符串   
char buf [];
//buf数组中已使用字节的数量,等于SDS所保存字符串的长度    
int len;
// buf数组中未使用字节的数量, 是0则表示这个SDS分配的空间都被用完了  
int free;
}
```

## SDS与C语言字符串的比较

 SDS采用结构体的方式存储字符串，而不是采用c语言的开辟一个连续的存储空间的方式。

### 字符串长度-O(1)

C语言如果要获取字符串的长度，需要从第一个字符开始，遍历整个字符串，直到遍历到\0符号，时间复杂度是O(N)，即字符串的长度。

`Redis`获取字符串长度的复杂度为`O(1)`, 直接读取`len`.所以对一个非常长的字符串反复执行`strlen`也不会有性能问题.

看下边的例子:

![](https://youpaiyun.zongqilive.cn/image/006tKfTcly1g0h79gpdkbj30lo0aw74e.jpg)

`len=5`,表示存储了一个5字节长的字符串, 而最后一个字节保存了空字符`\0`.

注意的是最后的空字符不计算在`len`内.

![](https://youpaiyun.zongqilive.cn/image/006tKfTcly1g0h7cdq2grj30sg0b8q38.jpg)

### 杜绝缓冲区溢出

#### 空间预分配

 空间预分配用于优化字符串的增长操作，实现方式为：当需要增长字符串时，sds不仅会分配足够的空间用于增长，还会预分配未使用空间。

分配的规则是，如果增长字符串后，新的字符串比1MB小，则额外申请字符串当前所占空间的大小作为free值；如果增长后，字符串长度超过1MB，则额外申请1MB大小。

例如，字符串增长后，大小是50kb，则额外申请50kb作为未使用空间。如果字符串增长后的大小是20mb，则额外申请1mb作为未使用空间。以上两种情况都为将\0计算在内，因此，实际上还会需要1字节作为\0存放的空间。

上述机制，避免了redis字符串增长情况下频繁申请空间的情况。每次字符串增长之前，sds会先检查空间是否足够，如果足够则直接使用预分配的空间，否则按照上述机制申请使用空间。该机制，使得字符串增长n次，需要申请空间的次数，从必定为n次的情况，降为最多n次的情况。



#### 懒惰空间释放

懒惰空间释放用于优化sds字符串缩短的操作，实现方式为：当需要缩短sds的长度时，并不立即释放空间，而是使用free来保存剩余可用长度，并等待将来使用。当有剩余空间，而有有增长字符串操作时，则又会调用空间预分配机制。

当redis内存空间不足时，会自动释放sds中未使用的空间，因此也不需要担心内存泄漏问题。

## 二进制安全

sds是二进制安全的，写入的是什么内容，返回的也是什么内容，并没有限制。

- C字符串只能保存文本数据，而不能保存像图片、音频、视频、压缩文件这样的二进制数据
- 如果有一种使用空字符来分割多个单词的特殊数据格式，就不能用C字符串来表示，如”Redis\0String”，C字符串的函数会把’\0’当做结束符来处理，而忽略到后面的”String”。而SDS的buf字节数组不是在保存字符，而是一系列二进制数组，SDS API都会以二进制的方式来处理buf数组里的数据，使用len属性的值而不是空字符来判断字符串是否结束。



## 总结——C语言字符串和SDS字符串比较

![](https://ae01.alicdn.com/kf/H2c3facb71b2c45618e95e67a49ec6a093.jpg)

**SDS与C字符数组区别**

**1.避免内存溢出**

c语言在操作字符串是要先判断内存是否有溢出风险，而sds事先可以知道free空间是否允许被覆盖，也防止了溢出到其他空间的风险。

**2.二进制安全**

C语言字符数组是以\0结尾，因此不允许存储带空格的二进制数据。而SDS是根据len判断字符串长度，所以是二进制安全的。

**3.计算字符串长度等时间复杂度更低**

SDS可以根据len属性O(1)时间返回字符串长度，还封装了一些高效的API，效率比C字符串更高。

**4.减少重新分配内存次数**

C语言字符串每次有修改可能都要重新分配更合适的内存，而SDS有len和alloc还有buf可以允许存储更多数据，或者惰性释放必然会减少内存重新分配操作。





## embstr vs raw

Redis 的字符串有两种存储方式，在长度特别短时，使用 `emb` 形式存储 (embeded)，当长度超过 44 时，使用 `raw` 形式存储。

### embstr 和 raw 的本质区别

内存分配上: 
embstr调用1次malloc, 因此redisObject和SDS内存是连续分配的
raw需要调用2次malloc, 因此redisObject和SDS内存不连续分配
使用上: 
embstr 整体 64 byte, 正好和cpu cache line 64byte 一样, 可以更好的使用缓存, 效率更高



### 分析

![](https://ae01.alicdn.com/kf/H14a2b37ab9e84bc1902f072ab470843eg.jpg)

![](https://ae01.alicdn.com/kf/H66623cfd6145409eac0e5331e64d4263N.jpg)

![](https://ae01.alicdn.com/kf/Ha38142fd18cb4aa2a2757048b194088fX.jpg)

![](https://ae01.alicdn.com/kf/H78b054464f5042d4abbb1cb557c64ab4Q.jpg)



## 扩容策略

字符串在长度小于 1M 之前，扩容空间采用加倍策略，也就是保留 100% 的冗余空间。

当长度超过 1M 之后，为了避免加倍后的冗余空间过大而导致浪费，每次扩容只会多分配 1M 大小的冗余空间。









