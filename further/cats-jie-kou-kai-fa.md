# CATS接口开发

## 记录

* catsapi\_linux.zip用windows资源管理器打开显示无法打开，无效，用bandzip打开显示该文件已损坏。用bandzip解压显示libACE.so.7.0.8-无法读取压缩包数据，出现错误。bandzip尽管显示错误，仍然在解压路径下看到了libACE.so.7.0.8，但banzip中显示大小为16122272字节，windows资源管理器显示解压后为16572198字节。发到linux下用unzip显示End-of-central-directory signature not found，解压失败。用bsdtar -xvf解压,在解压so.7.0.8时显示Too much data:Truncating file at 16122272 bytes:No such file or directory后Error exit delayed from previous errors，然后linux下解压文件夹中so大小为16122272字节。原因是文件损坏，换好的就行。
* VS以包括文件的最近公共根目录为传输主目录传输到远程项目目录，所以vcxproj应该放在公共根目录下以防目录结构传输中改变。
* 由于CATSApi支持异步回调，直接同步调用各种己方接口回调即可。
* CATS 的conf需要在./conf下，文件结构？lib64?
* CATS 错误信息为中文，GB2312格式，需要vim -c "e ++enc=GB2312 xx"打开LOG或iconv -f gb2312 -t utf-8 xxx|vi 来读
* 连接服务器失败，请确认服务器ip及端口配置是否正确！尝试将行情端口设为示例中的11000失败，仍然该错误；尝试将行情地址、端口处理删除，失败报错hq服务器参数错误，将行情地址、端口用“”初始化，依然hq服务器参数错误。
* Trade能订阅但无文档。Instruction能订阅但无submitId可以传。Order能订阅。三者均没有Query接口，需要自己维护。
* Query有且仅有Position,account,fund，其它要么没有要么没用。得自己维护。
* 算法参数没有文档
* 订阅三个
* Connect将API包更新并把demo下的xml复制到项目下后不报服务器IP、端口错误了，而是在连接中报Segmentation fault
* 尝试排除std库版本问题，于是尝试编译运行Release版，但CreateTradeApi失败，Segmentation fault。重新编译AnyTradexxx项目后运行相同地方segmentation fault。demo也是
* 加上推荐的libudev.so.1后仍然报错，经建议登录时传hdSerialNo免得自主查询仍报错，得到最新so版本后成功。Login报禁止非API用户登录，联系对方开权限即可。
* 开平标志不是简单对应买卖。。。。HPI有待更新。
* 主线程pthread\_setname\_np后调试显示改名不成功。
* CATS中cout不见了，经检查CATSInit后cout被重定向到./CONOUT$,而cin被重定向到某个socket?Init时传0就好。
* position、fund查询有头文件，找不到文档。可以订阅，自己维护？
* 全部都订阅，自己维护Qry结果
* submitId通过source\_id来传，在算法实例订阅中得到
* AlgoInstance状态由stopped维护，C、A自己维护
* 多个AlgoExec回报是多个统计积压推送，按序回调即可
* 算法实例、Trade订阅、
* position没有期初持仓，第一次订阅Trade的回调结束后计算出期初持仓，不需要控制订阅顺序，假定重启时单全撤了，第一次Position回调和Trade回调期间不会有变化。第一次回调Position和Trade时初始化期初持仓。第一次回调完成后startpolling
* Algo实例的订阅，必须在初始化前完成submitId和instanceId的对应，然而如何确定每个InstanceId都与submitId绑定了呢？sleep(5s)
* Order、AlgoExec、Trade需要依赖submitId和Instruction的绑定关系，若当前SysId找不到submitId,放到队列末尾等待处理。如果多次找不到再填000000
*
