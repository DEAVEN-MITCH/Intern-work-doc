---
description: 以下耗时可能因时间戳而具有几μs/几百ns误差
---

# ReqOrderInsert优化分析

## LOG

耗时10微左右，注释掉，不管它。等待后人的智慧。

## make Request

15μs,其中构造新NetRequest2-3μs，makexxxATPInputOrder18μs,其他分支不清楚，BuildHeader1-2μs

由于frame数量与makexxinputOrder数量一样而少于makeRequest数量，认为优化目标是xxxorder而不是其他分支。

### makexxxInputOrder

分4个部分，2为8μs，4为6μs，1为2.5μs，3为百纳秒可忽略不计

#### makexxxInputOrder1

总共2.4微秒左右，分为4部分

260ns,120ns,1.8μs,120ns

第一个部分就是一个枚举赋值，应该是内存为缓存访问耗时几百纳秒，第二个部分是一个静态转换， 第四个是一个if判断，成功抛异常。第三个部分是unordered\_map的find操作，换成map慢了150ns左右，不行。优化Hash函数？不直接，风险高，暂不考虑。

order1结束。

#### makexxxInputOrder2

即绝大部分为initUrlPath部分。这一部分分为3个部分，其中第三部分占比最大

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

initurlpath1是if\_else+字符串拼接，没有显著优化空间，initurlpath2是字符串拼接，没有显著优化空间。

initulPath3再细分为4个部分，其中1、2没有触发，3占2.4μs,4占1.3μs

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

initUrlPath3\_3中self是if-else+字符串拼接调用了NormalizePriceString才造就的。

将if中复杂变量访问保存到临时变量中后提升mean400ns，median200ns。进一步改为switch，与if else+临时变量相比没有明显提升，放弃switch。

initUrlPath3\_4中有一个"&"的+=，改成push\_back('&'),结果始终在误差范围内不能说有提升，放弃。

urlPathsize大概是150少一些

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

addrequest方法中，前两个区域中位数只有100+ns，最后一个有近3微秒。

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

其中这个3微秒的区域也有多个函数调用，拆分开来看，一个1.4微秒的时间函数调用（boost库，觉得不能轻易优化），一个1微秒的其他（3\_2)。

再拆。应该是Holder对象变量的赋值占主要矛盾（700ns），make\_pair只占350ns，其他可忽略。不好优化。

如果再有log,log更要占大头，10微秒左右，其他部分5微，不是主要矛盾。

addrequest结束。

order4结束

order结束

### makerequest order前后

前1.7μs,500ns,后110ns,1.6μs

第一个部分为new NetRequest并赋值到shared\_ptr中，由于该结构体包含众多复杂子结构，new起来为主要开销，第二个部分为reserve512字节，500ns，显然利大于弊，第三部分为一个bool赋值，110ns……。

将nrqshared\_ptr的间接访问改为裸指针的间接访问，减少一层嵌套，结果如下：

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>raw ptr</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>shared ptr</p></figcaption></figure>

有近一微秒的性能提升。

为了测试temp变量的效果是否源于避免shared\_ptr的间接访问，控制变量进行测试（raw ptr版+non temp）比temp版慢近1μs。重测两遍，中位数分别慢0.3、0.58μs，说明temp有效。

注释掉NetRequest中不用的部分，优化了new时间1μs。

第四部分为buildHeader，函数很简单，emplace\_back一个string，没什么优化空间。换成inline会破坏头文件风格，内联优化估计也没几个ns，决定不动。

makeRequest结束。



***

makeRequest和onFlowControl间隔了1μs,是setTool的调用，setTool为一个lambda表达式，传入shared\_ptr，shared\_ptr间接内容被赋值。由于被赋值的是两个string成员，占比较大，间接引用非主要矛盾，不需要吹毛求疵地优化。实测setTool600ns左右，string赋值为主。

## onFlowControl

55+μs 8.1测median为53μs，postRequest前5μs，postRequest46μs

### postRequest前

5，5.5μs。分为四个部分，中位数550ns,120ns,3.7μs,110ns，（第一部分前Zone的构建要500ns+……）

第一部分为Unordered\_map的find,显然优化困难，map更差，不考虑。第二部分为一个if判断，真则log，120ns无法更优化。第四部分也是if判断可能LOG，无法更优化。第三部分占比较多。第三部分分为tuple的get和一个check函数。经测试get占几十一百ns，check占3μs+，为主要矛盾。check中是>=3个map的find，不知道是干嘛的，map改不了，因为他会被顺序遍历,不好改。更深入的函数调用由于看不懂数据结构含义，不好改，以后再说。

### postRequest

划分为四个区域后，耗时为47μs

300ns,19μs,7μs,17μs

onRequestReturn不在帧内，约200μs。

第一个部分 考虑raw ptr优化，测试总体结果在1微的误差范围内，算了。

第二个部分就是onPreSend，走的是if\_true,内容是getSign+字符串拼接。字符串拼接可能怕signature不是原子性加入urlPath,加了个括号？但+=本身并不是原子的，如果用两个+=可以提升，从420ns提升到230ns。

getSign……

第三个部分中有stringstream,单独拆分来看。stringstream构造近1.9微，格式化输入2.1微，static\_cast80ns，取str400ns，LOG1.7微。测出来stringstream的str大小介于100-300之间。

用一个512的字符数组替代Stringstream的功能，利用snprintf格式化输出，将整个第三部分优化到3μs左右，优化效果为4-5微，整体有6微。但因为涉及日志库部分，暂时不用优化，不管它。

> 由于可能要换接口，PostRequest的优化可能无用武之地，停止。



