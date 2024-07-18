---
description: >-
  investigate into the existing realization of lock-free queue,compare them and
  pick up some common ideas
---

# 🙃 已有无锁队列调研

## 参考代码库

* moodycamel::ReaderWriterQueue(spsc)
* moodycamel::ConcurrentQueue(mpmc)
* atomic\_queue::AtomicQueue(ε|2|B|B2)
* boost::lockfree::spsc\_queue
* boost::lockfree::queue

## 简单无锁队列

* 无队列长度限制、无缓冲区的：用链表实现，对链表头、尾以及next指针进行atomic包装
* 有队列长度限制、有缓冲区的：用T\[]的buffer搭配头尾的index实现，其中头尾用atomic包装
* spsc因可能指令重排下面理论不可行，必须借助atomic

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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
* fence(memory\_order\_sync)用来让所有相关线程同步。
* ReentrantGuard用来在debug模式下指示是否多个消费者/生产者进入队列。
* front完全由消费者更新，所以在消费者端不需要同步，生产者端需要同步。tail完全由生产者更新，同理。消费者拥有localTail,生产者拥有localFront.
*

### members

* largestBlockSize 由初始化的size参数决定，为size的二次幂顶或MAX\_BLOCK\_SIZE。若二次幂顶<=2\*MBS，则先分配一个MBS的块；否则将largestBlockSize更新为MAX\_BLOCK\_SIZE（此时size>MBS），并分配initialBlockCount个块,保证有一个空闲块外剩下的块slots大于等于size。构造结束时，最后一个block的next指向firstblock
* 移动构造函数没有互斥处理，需要用户自己同步，构造后other只剩下一个新的块
* 移动赋值函数交换各参数，间接交换块，需手动同步
* 析构时 首先一个最强的内存同步来获取最新参数（即Frontblock,tailblock），随后依次对block中元素进行析构。这里对于sizeMask(低位全一数）用&比%快。
* try\_xxx与xxx只是调用inner\_xxx这个模板函数，根据enum模板变量决定是否要分配空间。减少代码重复。
*

### static member

* make\_block是分配大小为capacity的block块(实际可用大小为capacity-1，即sizeMask)，分配格局为：|可能的block对齐填充|block|可能的block对齐填充|可能的T元素对齐填充|capacity个T元素块（相邻两个之间有默认对齐填充）|可能的T元素对齐填充。    注意到由于需要手动placement new将Block和elements分配到邻近的内存地址，所以需要手动计算合适对齐后的内存分配地址。            手动分配地址的好处：仅调用一次堆分配（malloc），且空间局部性更好。
  * 最终返回构造好的Block的地址。
* ceilToPow2是找到大于等于x的最小二次幂，利用位运算内联加速。
* align\_for是一个找到U类型对象在ptr开始的地址中第一个对齐的地址，运用内联优化。

### atomicops.h

* memory\_order这个枚举类用来提供统一的下层接口
* 只考虑编译环境下用atomic库实现的atomicops.h内容，因为VS显示会这么编译
* weak\_atomic类封装了atomic的relaxed存取与加载，fetch\_add\_acquire是acquire语义，fetch\_add\_release是release语义。



## moodycamel::BlockingReaderWriterQueue

* 用了一个LightweightSemaphore来进行自旋等待、通知、定时阻塞。内部用ReaderWriterQueue作为子数据结构。提供阻塞和非阻塞接口。
* SPSC

## moodycamel::BlockingReaderWriterCircularBuffer

* SPSC
* 循环缓冲区，缓冲区大小一定。与ReaderWriterQueue不同，没有block。用LightweightSemaphore实现了自选等待、通知、定时阻塞的子功能。有阻塞和非阻塞接口。

## moodycamel::ConcurrentQueue

### Features

* Knock-your-socks-off [blazing fast performance](http://moodycamel.com/blog/2014/a-fast-general-purpose-lock-free-queue-for-c++#benchmarks).
* Single-header implementation. Just drop it in your project.
* Fully thread-safe lock-free queue. Use concurrently from any number of threads.
* C++11 implementation -- elements are moved (instead of copied) where possible.
* Templated, obviating the need to deal exclusively with pointers -- memory is managed for you.
* No artificial limitations on element types or maximum count.
* Memory can be allocated once up-front, or dynamically as needed.
* Fully portable (no assembly; all is done through standard C++11 primitives).
* Supports super-fast bulk operations.
* Includes a low-overhead blocking version (BlockingConcurrentQueue).
* Exception safe.

### Constraints

* **not linearizable** (see the next section on high-level design). The foundations of its design assume that producers are independent; if this is not the case, and your producers co-ordinate amongst themselves in some fashion, be aware that the elements won't necessarily come out of the queue in the same order they were put in _relative to the ordering formed by that co-ordination_ (but they will still come out in the order they were put in by any _individual_ producer). If this affects your use case, you may be better off with another implementation; either way, it's an important limitation to be aware of.
* **not NUMA aware**, and does a lot of memory re-use internally, meaning it probably doesn't scale particularly well on NUMA architectures; however, I don't know of any other lock-free queue that _is_ NUMA aware (except for [SALSA](http://webee.technion.ac.il/\~idish/ftp/spaa049-gidron.pdf), which is very cool, but has no publicly available implementation that I know of).
* &#x20;**not sequentially consistent**; there _is_ a happens-before relationship between when an element is put in the queue and when it comes out, but other things (such as pumping the queue until it's empty) require more thought to get right in all eventualities, because explicit memory ordering may have to be done to get the desired effect. In other words, it can sometimes be difficult to use the queue correctly. This is why it's a good idea to follow the [samples](https://github.com/cameron314/concurrentqueue/blob/master/samples.md) where possible. On the other hand, the upside of this lack of sequential consistency is better performance.

### Efforts

*

    <figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

### Reference links

{% embed url="https://moodycamel.com/blog/2014/a-fast-general-purpose-lock-free-queue-for-c++" %}

{% embed url="https://moodycamel.com/blog/2014/detailed-design-of-a-lock-free-queue" %}

### Findings

* a set of sub-queues is used,one for each producder thread.This means that different threads ca enqueue items completely in parallel, independently of each other.
* All elements from a given producer thread will necessarily still be seen in that same order relative to each other when dequeued (since the sub-queue preserves that order), albeit with elements from other sub-queues possibly interleaved
* 用cusumerToken来提供就近匹配的信息，而如果没有这个token就是平常的遍历
* 每个子队列用的是数组而非链表，依然是blocks和innerindices，由于与Producer一一对应，tail无写竞争，head有写竞争，而head有回退机制防止出错。

> So, that's the high-level design. What about the core algorithm used within each sub-queue? Well, instead of being based on a linked-list of nodes (which implies constantly allocating and freeing or re-using elements, and typically relies on a compare-and-swap loop which can be slow under heavy contention), I based my queue on an array model. Instead of linking individual elements, I have a "block" of several elements. The logical head and tail indices of the queue are represented using atomically-incremented integers. Between these logical indices and the blocks lies a scheme for mapping each index to its block and sub-index within that block. An enqueue operation simply increments the tail (remember that there's only one producer thread for each sub-queue). A dequeue operation increments the head if it sees that the head is less than the tail, and then it checks to see if it accidentally incremented the head past the tail (this can happen under contention -- there's multiple consumer threads per sub-queue). If it did over-increment the head, a correction counter is incremented (making the queue eventually consistent), and if not, it goes ahead and increments another integer which gives it the actual final logical index. The increment of this final index always yields a valid index in the actual queue, regardless of what other threads are doing or have done; this works because the final index is only ever incremented when there's guaranteed to be at least one element to dequeue (which was checked when the first index was incremented)

* 批处理性能对于block而言是优越的
* o simplify the interface, if no token is provided by the user for a producer, [a lock-free hash table](#user-content-fn-1)[^1] is used (keyed to the current thread ID) to look up a thread-local producer queue
* 批处理时的活锁可能性？

> Since tokens contain what amounts to thread-specific data, they should never be used from multiple threads simultaneously (although it's OK to transfer ownership of a token to another thread; in particular, this allows tokens to be used inside thread-pool tasks even if the thread running the task changes part-way through)

* All the producer queues link themselves together into a lock-free linked list.
*

## boost::lockfree::queue

* mpmc
* CAS循环入列，底层用链表
* detail::select\_freelist\<node, node\_allocator, compile\_time\_sized, fixed\_sized, capacity>::type pool\_t;似乎是一个内存池来管理新节点的内存分配,具体是一个可扩容的空闲链表
* 节点的next是原子变量，队列的head和tail为原子变量

## boost::lockfree::spsp\_queue

* 满满的模板，层次交错，用模板元来降低耦合度，扩展性较高。
* 基于ringbuffer,其中定长与不定长均基于ringbuffer\_base，而ringbuffer\_base中也用了padding让write\_index\_和read\_index\_不在同一缓存行。
* 基于同一块内存上的write\_index和read\_index\_管理，index均指的是在指定buffer上的字节偏移量，均为原子变量。
*

    ```cpp
    Requirements:
     *  - T must have a default constructor
     *  - T must be copyable or movable
    ```

## DNedic/lockfree::spsc::Queue

* 仅支持push和pop,利用循环缓冲区和read、write的atomic变量实现。

## DNedic/lockfree::mpmc::Queue

* 仅支持push和pop，利用循环缓冲区实现，为避免竞争，每个元素都被slot封装，每个slot包含push\_count和pop\_countatomic变量用于防止竞争，对r、w\_counts队列的原子变量进行cas循环来获得竞争权。
* 显然O(size)个atomic变量降低了性能预期

## max0x7ba/atomic\_queue::AtomicQueue

### Design Principles

* Bare minimum of atomic instructions. Inlinable by default push and pop functions can hardly be any cheaper in terms of CPU instruction number / L1i cache pressure.
* Explicit contention/false-sharing avoidance for queue and its elements.
* Linear fixed size ring-buffer array. No heap memory allocations after a queue object has constructed. It doesn't get any more CPU L1d or TLB cache friendly than that.
* Value semantics. Meaning that the queues make a copy/move upon `push`/`pop`, no reference/pointer to elements in the queue can be obtained.

***

**任务改变，暂停调研。**

[^1]: what?
