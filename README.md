---
description: introduce the definition of Lock-Free and Lock-Free Queue
---

# 😘 Definition

## 无锁（lock-free)

无锁从字面上理解是没有使用锁的编程方式，然而无锁编程可以有另一种定义（也有可能是唯一正确定义）如下图，这个定义是指线程之间不互相阻塞，即一个线程不依赖于其他线程总能在有限过程中执行完毕。这样一来，没有锁是不充分的，还需要执行操作不依赖。任意死锁、活锁都不能存在于lock-free编程中。

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>引用自网站</p></figcaption></figure>

这里有一个例子，没有用到互斥锁，却非lock-free的：

{% code fullWidth="false" %}
```clike
while (X == 0)
{
    X = 1 - X;
}
```
{% endcode %}

这里考虑一个其他的邪恶的线程一直将X=1-X=1后的X改为0，那么这个循环就死循环了，无法在有限步完成，受其它线程阻塞，不是lock-free了。



> Herlihy & Shavit, authors of [The Art of Multiprocessor Programming](http://www.amazon.com/gp/product/0123973376/ref=as\_li\_ss\_tl?ie=UTF8\&tag=preshonprogr-20\&linkCode=as2\&camp=1789\&creative=390957\&creativeASIN=0123973376)![](http://www.assoc-amazon.com/e/ir?t=preshonprogr-20\&l=as2\&o=1\&a=0123973376), tend to express such operations as class methods, and offer the following succinct definition of lock-free (see [slide 150](https://docs.google.com/viewer?a=v\&q=cache:HaWgz4g5e7QJ:www.elsevierdirect.com/companions/9780123705914/Lecture%2520Slides/05\~Chapter\_05.ppt+\&hl=en\&gl=ca\&pid=bl\&srcid=ADGEESghbD6JBTSkCnlPP8ZjPwxS2kM6bbvEGUJaHozCN1CGYW0hnR0WkwmG7LvVj5BUOYZTfTXUClM7uXmr-nXPYlOvZulPJMgYXHaXqqo\_m9qkn38gw8qMn01tFoxTmTkvjalHzQOB\&sig=AHIEtbRChU00kpYARLAr5Cv5Z5aB2NLo5w)): “In an infinite execution, infinitely often some method call finishes.” In other words, as long as the program is able to keep _calling_ those lock-free operations, the number of _completed_ calls keeps increasing, no matter what. It is algorithmically impossible for the system to lock up during those operations.
>
> One important consequence of lock-free programming is that if you suspend a single thread, it will never prevent other threads from making progress, as a group, through their own lock-free operations. This hints at the value of lock-free programming when writing interrupt handlers and real-time systems, where certain tasks must complete within a certain time limit, no matter what state the rest of the program is in.

这里的定义指的是一个操作在无限次调用的情况下会无限次完成，不会受其他线程影响而被阻塞。

> A final precision: Operations that are _designed_ to block do not disqualify the algorithm. For example, a queue’s pop operation may intentionally block when the queue is empty. The remaining codepaths can still be considered lock-free.

这篇文章中将队列的pop对push的逻辑依赖（非编程上锁）排除在外的话，依然能算作lock-free。也就是说除了本质需求上有阻塞的部分外，其他的部分可以单独拎出来满足无锁编程。



{% embed url="https://preshing.com/20120612/an-introduction-to-lock-free-programming/" %}
原文链接
{% endembed %}

## 无锁队列

由于队列的push和pop操作可能逻辑上相互依赖，所以这里按照不用“锁”来认定无锁队列的定义。或者说，只要没有队列满后的push/空队列的pop的无限尝试，这个队列操作都应该互不阻塞。



### 种类

* 以生产者和消费者的单一/多个为区分，共有2×2种搭配
* 以队列是否限制最大长度未区分，共有2种搭配
