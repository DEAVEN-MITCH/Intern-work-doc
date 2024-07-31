---
description: 以下耗时可能因时间戳而具有几μs/几百ns误差
---

# ReqOrderInsert优化分析

## LOG

耗时10微左右，注释掉，不管它。等待后人的智慧。

## make Request

15μs,其中构造新NetRequest2-3μs，makexxxATPInputOrder18μs,其他分支不清楚，BuildHeader1-2μs

### makexxxInputOrder

分4个部分，2为8μs，4为6μs，1为2.5μs，3为百纳秒可忽略不计

#### makexxxInputOrder1

#### makexxxInputOrder2

即绝大部分为initUrlPath部分。这一部分分为3个部分，其中第三部分占比最大

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

initurlpath1是if\_else+字符串拼接，没有显著优化空间，initurlpath2是字符串拼接，没有显著优化空间。

initulPath3再细分为4个部分，其中1、2没有触发，3占2.4μs,4占1.3μs

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

initUrlPath3\_3中self是if-else+字符串拼接调用了NormalizePriceString才造就的。

将if中复杂变量访问保存到临时变量中后提升mean400ns，median200ns。进一步改为switch，与if else+临时变量相比没有明显提升，放弃switch。

initUrlPath3\_4中有一个"&"的+=，改成push\_back('&'),结果始终在误差范围内不能说有提升，放弃。

urlPathsize大概是150少一些

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

reserve 160后initurlPath提升了400多ns,总体没提升，可能是影响小于误差。reserve 512后提升了将近1微秒，总体也没提升，看Self only时间发现HMAC平均多了3微秒，应该是HMAC不稳定导致整体结果变差。

4中主要还是字符串拼接和NormalizeVolume开销，暂时无法进一步优化。

以上initurl部分结束，也就是makexxxInputOrder2结束，删除其中时间戳点。

优化前后对比，可以看出Order2部分优化了1.5微秒左右（删除所有url相关时间戳后）

前：

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

后：

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

***

#### makexxxInputOrder4

尝试优化lock\_guard，单独测试性能结果。1e5lock快可能是L1缓存命中率高的原因。反正相差无几，lock、unlock仅几十ns，不是性能瓶颈，性能瓶颈应该是锁的竞争。

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

mutex方法非inline，经查询，inline优化仅几个ns，忽略不计。

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>Debug版防止编译器优化，进行测试</p></figcaption></figure>



## onFlowControl

55+μs
