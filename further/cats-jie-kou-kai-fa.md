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
