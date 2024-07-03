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



