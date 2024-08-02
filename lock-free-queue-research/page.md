---
description: 记录关键点和疑问点
---

# 😤 CMessageQueue源代码阅读

## CMessageQueue

`CMessageQueue`可被继承，仅析构为虚函数。

使用条件变量来从空队列的pop中恢复，用锁来维护Process、Pop之间的竞争（Process即pop完）

m\_bStop仅在初始化时设为false，Stop调用后设为True.也就是说，Stop一次后将无法重新Run。

thread\_function为静态函数。

由于Process 中cvwait后直接判定m\_bStop标志，当多线程调用时可能导致新Post的任务未调用就终止，如Post完成notify\_all后切换到另一个线程的Stop调用，最后切到wait中唤醒的Process过程。应该按这个逻辑完成吗？或者在单Post和Stop不会多线程交叉的情况下不会引发这种情况。

由于unique\_ptr\<thread>的reset会销毁之前的线程，（而之前线程若在销毁时是joinable会使程序异常终止），若销毁成功则意味着同时只存在一个thread在执行Run代码，可以认为是单消费者？若为单消费者则与notify\_all矛盾？

Process在wait前切换到Stop,然后Stop完Process进行wait，永不唤醒了？需要给Stop加锁？（反映过，确实有问题，但是影响忽略不计）

### 单个运行顺序

先初始化，然后Run持续用另一个线程处理tasks，其他线程用Post和Stop控制Run的过程。



### 优化空间：

1. Process中while循环中的tasks变量的位置，提前至while前？
2. notify\_all??
3. deque?
4. tasks执行可用多线程/协程加速处理？

### 应用源代码阅读

#### OSTTradeApi

值变量公开成员，初始化时调用Run,析构借调用Release间接Stop,多个函数调用PostMsg间接调用Post入列。

#### CHsT2TradeSpi

值变量公开成员，PostSpiMsg调用Post方法，初始化调用Run,析构调用Stop,某方法调用PostSpiMsg间接Post。

### 测试方向

* 各函数耗时
* Post到Process的延时
* 占用率？内存？吞吐量？

