# 修改

## 修改

1. Notification.h删除锁、条件变量、tmpList,改变data类型为适配后的无锁队列类型。
2. Notification.cpp中有关tmpList操作的Merge方法被注释掉
3. LibATPStrategy.cpp中有关插入data加锁、Notify\_all的代码被注释掉。
4. LibATPStrategy.cpp中DoStrategyData将tmpList的获取及遍历改为获取单个并逐个处理，break;改为continue,对应外层循环。
5. CStrategyMDSpi中也有对插入data加锁相关代码被注释掉
6. CStrategyTradeSpi中也有...
7.
