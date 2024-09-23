# CATS接口开发

## 记录

* catsapi\_linux.zip用windows资源管理器打开显示无法打开，无效，用bandzip打开显示该文件已损坏。用bandzip解压显示libACE.so.7.0.8-无法读取压缩包数据，出现错误。bandzip尽管显示错误，仍然在解压路径下看到了libACE.so.7.0.8，但banzip中显示大小为16122272字节，windows资源管理器显示解压后为16572198字节。发到linux下用unzip显示End-of-central-directory signature not found，解压失败。用bsdtar -xvf解压,在解压so.7.0.8时显示Too much data:Truncating file at 16122272 bytes:No such file or directory后Error exit delayed from previous errors，然后linux下解压文件夹中so大小为16122272字节。原因是文件损坏，换好的就行。
* VS以包括文件的最近公共根目录为传输主目录传输到远程项目目录，所以vcxproj应该放在公共根目录下以防目录结构传输中改变。
* 由于CATSApi支持异步回调，直接同步调用各种己方接口回调即可。
*
