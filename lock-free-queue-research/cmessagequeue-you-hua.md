# CMessageQueue优化

要求：开销不要明显增大，优化延时

## 测时

### 初始化700+ns

### Run即开新线程耗时约100μs

### Post即入队耗时约11μs

90ns，1μs，8.5μs

其中Post2再细分为3块，每块150ns,20ns,20ns,其它是Zone开销

Post 1 为empty的初始化

Post2 为获取锁并push\_back，释放锁

Post3为notify\_all

将notify\_all改为notify\_one没有明显变化，峰值均在20ns、100ns与10μs处，10μs处的峰值数量与Process数量一致，表明在wait状态的Process的线程上下文切换？会导致notify的10μs延迟。

Post2第一部分为获取锁，第二部分为设置empty值，第三部分为push\_back

### Process while loop once即一次成功出队并消耗操作用时几百μs

分别370ns,160ns,175μs,50ns,160ns,14μs

Process 1为deque初始化，

Process2为锁的获取，

Process3为wait，

Process4为empty判断

Process5 为task的move

Process6为task的执行，其中遍历460ns，执行任务14μs

Process总耗时52s，Processtasks耗时3s，近16/17的时间都是Process3在wait cv，不占用cpu。



## 优化方向

用futex代替conditional\_variable。

由于线程唤醒开销是10μs级，可能优化用途不大。
