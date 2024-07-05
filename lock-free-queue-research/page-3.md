---
description: >-
  investigate into the existing realization of lock-free queue,compare them and
  pick up some common ideas
---

# 🙃 已有无锁队列调研

## 参考代码库

* moodycamel::readerwriterqueue(spsc)
* moodycamel::concurrentqueue(mpmc)
* atomic\_queue::AtomicQueue(ε|2|B|B2)
* boost::spsc

## 简单无锁队列

* 无队列长度限制、无缓冲区的：用链表实现，对链表头、尾以及next指针进行atomic包装
* 有队列长度限制、有缓冲区的：用T\[]的buffer搭配头尾的index实现，其中头尾用atomic包装
* spsc因可能指令重排下面理论不可行，必须借助atomic

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* 用位域来存储引用计数并能支持无锁atomic操作，实现对node结点的正确释放？

## moodycamel::readerwriterqueue

### Features

* Blazing fast
* Compatible with C++11 (supports moving objects instead of making copies)
* Fully generic (templated container of any type) -- just like `std::queue`, you never need to allocate memory for elements yourself (which saves you the hassle of writing a lock-free memory manager to hold the elements you're queueing)
* Allocates memory up front, in contiguous blocks
* Provides a `try_enqueue` method which is guaranteed never to allocate memory (the queue starts with an initial capacity)
* Also provides an `enqueue` method which can dynamically grow the size of the queue as needed
* Also provides `try_emplace`/`emplace` convenience methods
* Has a blocking version with `wait_dequeue`
* Completely "wait-free" (no compare-and-swap loop). Enqueue and dequeue are always O(1) (not counting memory allocation)
* On x86, the memory barriers compile down to no-ops, meaning enqueue and dequeue are just a simple series of loads and stores (and branches)

### Findings

* 仅在x86\64上测试过，但只要atomicops.h的实现正确，并且对齐的整数/指针访问具有自然的原子性，对于任何架构这个实现就应该是对的。
* 仅对固定一个生产者线程+固定一个消费者线程线程安全，否则不能保证
* 受某论文启发，对缓存友好性进行了优化。
* 低层次的队列是循环数组（缓冲区为block)，front,tail are indices,when tail == front the queue is empty and can't be full,分别对应出队元素索引和插入位置索引，恰好有一个元素被浪费；高层次是block组成的链表队列，front是出队块（非空的话），back是入队块（非满的话）。p拥有tail，c拥有front（读共享，写只写所拥有的）
* 如果入队块空间不够，则分配新的块。块永不删除。
* 利用内部结构体block来分配块，避免假共享？这里的block就是低层次队列。
  * 利用localTail来加速tail对consumer的访问，localFront同理
  * front和tail都是weak\_atomic\<size\_t>索引，而next为weak\_atmoic\<Block\*>指针用于指向下一个block。
  * **用cachelineFiller字符数组填充Block结构体，来让原子变量不在同一cacheline，避免伪共享。**
  * char\*指针指向实际元素的地址，保证了Block的size对任一模板参数都是固定的（控制变量）。
  * sizeMask是实际能容纳元素个数，比data分配大小少1
  * rawThis指向make\_block中为对齐而分配到的原始块的大小。

### members

* largestBlockSize 由初始化的size参数决定，为size的二次幂顶或MAX\_BLOCK\_SIZE。若二次幂顶<=2\*MBS，则先分配一个MBS的块；否则将largestBlockSize更新为MAX\_BLOCK\_SIZE（此时size>MBS），并分配initialBlockCount个块,保证有一个空闲块外剩下的块slots大于等于size。

### static member

* make\_block是分配大小为capacity的block块(实际可用大小为capacity-1，即sizeMask)，分配格局为：|可能的block对齐填充|block|可能的block对齐填充|可能的T元素对齐填充|capacity个T元素块（相邻两个之间有默认对齐填充）|可能的T元素对齐填充。    注意到由于需要手动placement new将Block和elements分配到邻近的内存地址，所以需要手动计算合适对齐后的内存分配地址。            手动分配地址的好处：仅调用一次堆分配（malloc），且空间局部性更好。
  * 最终返回构造好的Block的地址。
* ceilToPow2是找到大于等于x的最小二次幂，利用位运算内联加速。
* align\_for是一个找到U类型对象在ptr开始的地址中第一个对齐的地址，运用内联优化。

### doubt

1. 117行应该是大于等于号，否则不能保证largestBlockSize>=size+1?例如size=600,ceilToPow2(size+1)=1024,MSB=512满足largestBlockSize<=MSB\*2,而MSB\<size+1????

### atomicops.h

* memory\_order这个枚举类用来提供统一的下层接口
* 只考虑编译环境下用atomic库实现的atomicops.h内容，因为VS显示会这么编译
*

