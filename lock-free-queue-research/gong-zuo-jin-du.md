# 😘 工作进度

## 修改

1. Notification.h删除锁、条件变量、tmpList,改变data类型为适配后的无锁队列类型。
2. Notification.cpp中有关tmpList操作的Merge方法被注释掉
3. LibATPStrategy.cpp中有关插入data加锁、Notify\_all的代码被注释掉。
4. LibATPStrategy.cpp中DoStrategyData将tmpList的获取及遍历改为获取单个并逐个处理，break;改为continue,对应外层循环。
5. CStrategyMDSpi中也有对插入data加锁相关代码被注释掉
6. CStrategyTradeSpi中也有...
7. logQueue使用模板变量提供对Queue的push\_back、pop的封装

## 相关考虑

* 得到一个元素进队到出队之间的延时。
* Release版本测试
* git保存代码
* OnTickUpdate打出行情

结果：

* 从尝试到成功push\_back在Release版本下约为600-1400ns,估计均值为1μs
* 从尝试到成功pop在Release版本下延迟非常高，意味着队列几乎时时为空，空转尝试pop；
* 用人眼识别push\_back成功时间点之后成功的pop操作（已空转很久），口算得从尝试Push\_back到成功pop延迟约为2μs
* 在消息从SPI传入到Strategy端输出，测量平均延迟为22μs（1000次取平均）
* 由于22μs的测量中包含了一次clog输出，其误差可能较大，尝试测量clog在程序运行时耗时（即用两个clog测量一次clog时间），能有8-16μs，而用传入-输出延时-2\*clog延时平均下来为负数，这就不好说明没有clog时延时为多少。
* 尝试通过全局变量来克服即时clog的需要，然而两端文件共有的头文件均在libCommon中，且libCommon编译为静态库，两端编译为动态库，导致两个so文件中有两份所谓的“全局变量”，不能成功进行start到end的记录。失败。
