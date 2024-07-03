---
description: >-
  investigate into the existing realization of lock-free queue,compare them and
  pick up some common ideas
---

# 已有无锁队列调研

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

*

## moodycamel::readerwriterqueue

### 适用性



