### ThreadLocalMap

ThreadLocalMap内部是利用Entry来进行key-value的存储的。

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
  /** The value associated with this ThreadLocal. */
  Object value;
// key就是ThreadLocal实例，value就是值，Entry继承WeakReference，所以Entry对应key的引用（ThreadLocal实例）是一个弱引用
  Entry(ThreadLocal<?> k, Object v) {
    super(k);
    value = v;
  }
}
```





### ThreadLocal为什么会内存泄漏

内存泄露问题：

   ThreadLocal即使使用了WeakReference（弱引用）也可能会存在内存泄露问题，因为 entry对象中只把key(即threadLocal对象)设置成了弱引用，但是value值没有。还是会存在下面的强依赖：

```
Thread -> ThreaLocalMap -> Entry -> value
```





ThreadLocalMap的key为ThreadLocal实例，他是一个`弱引用`，我们知道弱引用有利于GC的回收，当key == null时，GC就会回收这部分空间，

但value不一定能被回收，因为他和Current Thread之间还存在一个强引用的关系。由于这个强引用的关系，会导致value无法回收，如果线程对象不消除这个强引用的关系，就可能会出现OOM。有些时候，我们调用ThreadLocalMap的remove()方法进行显式处理。

![](https://youpaiyun.zongqilive.cn/image/20201214094343.png)



**正常情况**

当Thread运行结束后，ThreadLocal中的value会被回收，因为没有任何强引用了

**非正常情况**

当Thread一直在运行始终不结束(例如线程池)，强引用就不会被回收，存在以下调用链 `Thread-->ThreadLocalMap-->Entry(key为null)-->value`因为调用链中的 value 和 Thread 存在强引用，所以value无法被回收，就有可能出现OOM。



当ThreadLocal Ref出栈后，由于ThreadLocalMap中Entry对ThreadLocal只是弱引用，所以ThreadLocal对象会被回收，Entry的key会变成null，然后在每次get/set/remove ThreadLocalMap中的值的时候，会自动清理key为null的value，这样value也能被回收了。***注意：\***如果ThreadLocal Ref一直没有出栈（例如上面的connectionHolder，通常我们需要保证ThreadLocal为单例且全局可访问，所以设为static），具有跟Thread相同的生命周期，那么这里的虚引用便形同虚设了，所以使用完后记得调用ThreadLocal.remove将其对应的value清除。



JDK的设计已经考虑到了这个问题，所以在set()、remove()、resize()方法中会扫描到key为null的Entry，并且把对应的value设置为null，这样value对象就可以被回收。

```java
private void resize() {
  Entry[] oldTab = table;
  int oldLen = oldTab.length;
  int newLen = oldLen * 2;
  Entry[] newTab = new Entry[newLen];
  int count = 0;

  for (int j = 0; j < oldLen; ++j) {
    Entry e = oldTab[j];
    if (e != null) {
      ThreadLocal<?> k = e.get();
      //当ThreadLocal为空时，将ThreadLocal对应的value也设置为null
      if (k == null) {
        e.value = null; // Help the GC
      } else {
        int h = k.threadLocalHashCode & (newLen - 1);
        while (newTab[h] != null)
          h = nextIndex(h, newLen);
        newTab[h] = e;
        count++;
      }
    }
  }

  setThreshold(newLen);
  size = count;
  table = newTab;
}

```



###   为什么要做这样的清除

   我们知道entry对象里面包含了threadLocal和value，threadLocal是WeakReference（弱引用）的referent。每次垃圾回收期触发GC的时候，都会回收WeakReference的referent，会将referent设置为null。那么table数组中就会存在很多threadLocal = null 但是 value不为空的entry，这种entry的存在是没有任何实际价值的。这种数据通过getEntry是获取不到值，因为它里面有if (e != null && e.get() == key)这句判断。

### 为什么要使用WeakReference（弱引用）

   如果使用强引用，ThreadLocal在用户进程不再被引用，但是只要线程不结束，在ThreadLocalMap中就还存在引用，无法被GC回收，会导致内存泄漏。如果用户线程耗时非常长，这个问题尤为明显。

   另外在使用线程池技术的时候，由于线程不会被销毁，回收之后，下一次又会被重复利用，会导致ThreadLocal无法被释放，最终也会导致内存泄露问题。









