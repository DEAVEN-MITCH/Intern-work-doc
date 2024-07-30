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

#### 第二部分

即绝大部分为initUrlPath部分。这一部分分为3个部分，其中第三部分占比最大

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

再细分为4个部分，其中1、2没有触发，3占2.4μs,4占1.3μs

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

3中self是if-else+字符串拼接调用了NormalizePriceString才造就的。

将if中复杂变量访问保存到临时变量中后提升mean400ns，median200ns。进一步改为switch，与if else+临时变量相比没有明显提升，放弃switch。

urlPathsize大概是150少一些

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

reserve 160后initurlPath提升了400多ns,总体没提升，可能是影响小于误差。reserve 512后提升了将近1微秒，总体也没提升，看Self only时间发现HMAC平均多了3微秒，应该是HMAC不稳定导致整体结果变差。

4中主要还是字符串拼接和NormalizeVolume开销，暂时无法进一步优化。

## onFlowControl

55+μs
