---
description: notes
---

# ⚠️ notes

## 宽松内存序

即指令可能重排序

## points

* 在 x86/64 指令级别，每次从内存加载都带有获取语义，并且每次存储到内存都提供释放语义——至少对于非 SSE 指令和非写组合内存

## discrimination&#x20;

* **Wait-free** guarantees progress for every thread, regardless of others.
* **Lock-free** guarantees progress for the system as a whole, but individual threads might be blocked momentarily.
* **Obstruction-free** guarantees progress only when there's no interference from other threads.

## Wait-Free Synchronization reading notes

* In a system of n or more concurrent processes, we show that it is impossible to construct a wait-free implementation of an object with consensus number n from an object with a lower consensus number.
* We give a simple test for universality, showing that an object is universal in a system of n processes if and only if it has a consensus number greater than or equal to n.
* From a set of atomic registers, we show that it is impossible to construct a wait-free implementation of (1) common data types such as sets, queues, stacks, priority queues, or lists, (2) most if not all the classical synchronization primitives, such as test\&set,compare\&swap, and fetchd-add, and (3) such simple memory-to-memory operations as move or memory-to-memory swap.
* The fetch\&add operation is quite flexible: it can be used for semaphores, for highly concurrent queues, and even for database synchronization \[11, 14, 30]. Nevertheless, we show that it is not universal, disproving a conjecture of Gottlieb et al. \[11]. We also show that message-passing architectures such as hypercubes \[28] are not universal either
*

    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption><p>what is well-formed</p></figcaption></figure>
*

    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
*   universal构建方法需要O(n^3)空间，显然不符合性能需要，终止阅读



## 模板相关

* 继承模板类时，模板类方法不可见，得using才能找到（可能是C++11问题）
* using xx::template xxx不会用，错的
* C++11不支持返回值auto
