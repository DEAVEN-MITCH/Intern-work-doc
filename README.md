---
description: introduce the definition of Lock-Free and Lock-Free Queue
---

# 😘 Definition

## 无锁（lock-free)

无锁从字面上理解是没有使用锁的编程方式，然而无锁编程可以有另一种定义（也有可能是唯一正确定义）如下图，这个定义是指线程之间不互相阻塞，即一个线程不依赖于其他线程总能在有限过程中执行完毕。这样一来，没有锁是不充分的，还需要执行操作不依赖。任意死锁、活锁都不能存在于lock-free编程中。

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>引用自网站</p></figcaption></figure>

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



{% embed url="https://preshing.com/20120612/an-introduction-to-lock-free-programming/" %}
