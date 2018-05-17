# windows iis 自动更新

![iis-mime](./_static/img/iis-mime.jpg )

* 再配置IIS 的http头选项中添加 MIME 类型

```
扩展名 .ini
类型   plain/te


扩展名 .aus
类型  plain/text


扩展名 .dll
类型 application/x-msdownload
```

## 升级脚本示例子

* 流程
1. 客户端启动后读取Config.ini文件中[Server]区块下UpgradeUrl的Url地址
2. 下载UpgradeUrl的文件数据
3. 启动LiveUpdate.exe，分析下载的文件数据，然后判断是否需要升级，如果需要升级则显示升级界面。
因此，需要配置自动更新系统主要配置的就是Config.ini文件和UpgradeUrl指向的升级规则文件。

#### [MACRO]
区块中主要定义远程文件下载目录和下载到本地的文件存储目录。BASEURL用于定义远程文件下载的目录，必须为HTTP协议地址；UPGRADE定义下载文件后存储文件的目录，$(APP)Upgrade\表示"程序根目录\Upgrade\”目录下，当LiveUpdate.exe程序从网络中下载的文件都将缓存在此目录下。

#### [RUN]
区块中主要定义升级完成后自动运行的程序，一般此处无需改动，默认就是$(APP)NetHookClient.exe。

#### [VERSION]
区块负责定义当前最新版本和升级程序支持的版本。VERSION定义的是当前最新版本，SUPPORT定义客户端哪些版本支持升级，其余版本则不能自动更新，需要下载完整的客户端程序。这两个配置项的版本都是M.N形式，M为主版本，N为次版本，如果程序的版本为3.1.0.9, 则版本为3.1，后边的0.9将忽略。客户端判断程序当前的版本是通过VersionDLL.dll的文件版本来判断，升级时此文件必须修改版本，且每次更新都必须更新此文件。更改文件的版本可以使用exescope等资源编辑软件来进行，无需程序的源代码。

#### [DOWNLOAD]
区块定义本次更新需要下载哪些文件，序号从1开始，标准格式为“1 GET=下载路径,存储路径”，参考例子即可

#### [ACTION]
区块定义下载到缓存目录文件执行怎样操作，REPLACE表示替换，一般情况下仅用到REPLACE, 标准格式为“1 REPLACE=源文件路径,目标文件路径”


```
;=====================
; 自动升级脚本 (示例)
;=====================

;所有以分号为第一个字符开始的行都被脚本执行器忽略；
;请勿在脚本中使用TAB键，否则在Win98下会出问题；

;定义脚本中要用到的宏
[MACRO]
BASEURL=http://www.net963.com/Download/
UPGRADE=$(APP)Upgrade\

;升级成功后要启动的程序
[RUN]
RUN=$(APP)NetHookClient.exe

;VERSION 升级后可到达的版本
;SUPPORT 支持升级的版本(用逗号隔开)
[VERSION]
VERSION=3.1
SUPPORT=3.0

;首先下载文件，如果有任何文件下载错误则放弃更新
[DOWNLOAD]
1 GET=$(BASEURL)NetHookServer2.3.rar,          $(UPGRADE)NetHookServer2.3.rar
2 GET=$(BASEURL)NetHookServer2.4.rar,             $(UPGRADE)NetHookServer2.4.rar

;升级文件下载完整后，开始执行升级动作
;EXECUTE 执行指定的文件
;REPLACE 替换指定的文件，如果没有就新增
;ADD     增加一个文件，如果有则放弃
;DELETE  删除指定的文件

[ACTION]
1 REPLACE=$(UPGRADE)NetHookServer2.3.rar,      $(APP)NetHookServer2.3.rar
2 REPLACE=$(UPGRADE)NetHookServer2.4.rar,   $(APP)NetHookServer2.4.rar
```