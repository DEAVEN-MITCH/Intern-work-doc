# 技术考虑点

## 种类

1. MPMC
2. SPSC
3. MPSC

* 长度满了阻塞/自动扩容（调研）

## atomic库相关

### 内存顺序

默认的atomic内存序是顺序执行，如果需要加速，得充分了解内存障的类型确保正确下选择更松散的内存序

### 平台上是否真正无锁（atomic\<T>::is\_lock\_free()->bool)

因为atomic库只支持平台支持的数据类型的无锁原子操作，复杂类型的atomic对象往往基于锁实现。如某平台上支持128位字长的数据结构的无锁原子值操作。

需要根据平台选择合适的原子对象的泛型类型

支持泛型，Linux平台

### CAS问题

考虑ABA问题的避免

考虑其他原子操作？

考虑平台相关性，自定义RMW指令？

## 队列元素相关

考虑对象入队列时是否需要复制构造、移动构造，以及emplace\_back

考虑元素竞争？

考虑构建、无空间异常？

考虑队列元素类型泛型支持

## 测试相关

google benchmark，有待了解

测试用例

与已有代码实际用途适配

正确性测试

## 功能方面

SPSC、MPMP、MPSC……可能被多次应用

## 已有无锁队列参考

已有代码中include的messagequeue，spsc，长度固定

liblfds.org中无锁队列

{% embed url="https://liblfds.org/" %}

boost::lockfree::queue

{% embed url="https://max0x7ba.github.io/atomic_queue/html/benchmarks.html" %}
很多无锁、有锁队列的性能测试比较
{% endembed %}



## 性能考虑点

* 延迟
* ~~吞吐率~~
* 其他

## 其他考虑点

1. 与其他优化的有锁队列进行对比，比如concurrent\_queue等
2. 无等待编程、无锁编程、无障碍编程思想的应用
3. ~~RAII智能指针的使用~~
4. ~~协程替代线程使用？协程适合IO密集型应用，适合吗？（这些是应用层考虑，组件级不考虑）~~

## 参考资料链接

{% embed url="https://dl.acm.org/doi/epdf/10.1145/114005.102808" %}

{% embed url="https://stackoverflow.com/questions/45907210/lock-free-progress-guarantees-in-a-circular-buffer-queue" %}

{% embed url="https://github.com/max0x7ba/atomic_queue?tab=readme-ov-file" %}
