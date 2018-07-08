# soft-and-hard
《软硬结合——写给硬件开发工程师的全栈入门实战》，软硬件结合可以说是所有硬件开发人员心中的一大追求，当一个人技能树上点亮了软硬件，所有创意想法基本都能靠自己去实现。整个教程与源码都是免费公开的，但不允许别人通过这教程赚钱，我就赚个人气~ __如果你想支持这个免费教程，请在github上给我一个关注的星星以示支持__

## 暂缓更新公告
由于此免费教程关注的星星数量太少，感觉没什么人看所以决定暂缓更新，如果关注的星星数上来了我再继续更新。  
暂时只在这里把整个思路给出来，供动手能力比较强的读者参考。  
demo1已经把基本流程打通，软硬件正常通信。  
demo1的后续：
### 设备id问题
为了搞清成千上万个设备接入来后，谁是谁的问题，每一个设备都应该有一个独一无二的id，以保证能监控到要监控的设备。
设备首次建立连接时，主动上报id，服务器就利用这个id对应唯一的设备。
什么时候确定id生成：
一种是硬件出厂前在硬件里烧录写死的id，这相对于后者要麻烦些，需要做更多的事情。
另一种是首次建立连接时才通过软件动态生成的id，利用uuid算法能生成独一无二的id，同时硬件与服务器之间建立连接时会传输自己的id给服务器。
关于id，在前期只需要考虑唯一对应用起来就行，到了后期扩展时还有很多需要考虑：如何区分出设备类型、如何防止其它设备重复id以冒充的问题。

id问题存在两个场景：一种是面向管理员的系统（几个管理员管理所有设备），另一种是面向消费者的系统（每个消费者管理自己的设备，如小米物联网设备）。
为了方便书写与阅读，在后面的讨论里，我暂且把前一种当作2B，后者当作2C。
对于2B：使用建立连接时才动态生成id，这省事。
对于2C，还要额外考虑绑定用户的问题，下讨论。

### 数据库
数据肯定要保存到磁盘里去的，对于物联网项目来说，因为事务型比较少，所以使用Mongodb优于MySQL。

### 用户问题
对于2B的系统来说，不存在用户问题，接入的都是一个系统的，整个系统的用户只有超级管理员、管理员1、管理员2等等，超级管理员给其它管理员分配权限。
对于2C，就要考虑如何确定这个设备是谁的。
首先要建立用户体系，允许用户注册登陆等等，这些就不再讨论。讨论主要问题在于，如何让设备与用户绑定起来。
目前最佳的方案，贴一个二维码，用户登陆后再用手机扫这个二维码，这个二维码内容包含设备id，加上用户已经登陆，一扫，两者关系就确定下来了。
所以对于2C，设备id要在出厂前就要分配好设备id。
这问题就来了，2C与2B方案竟然不一样？这得多麻烦呀，所以不管2B还是2C，直接按2C的方案来做，出厂前就给每个设备有一个独一无二的id，并写入到二维码里贴在设备上。
而对于2B来说，这个二维码并不需要扫也能正常使用。

### 保持心跳
怎么让服务器知道这个设备还在正常运行，还处于在线状态？ 每隔一段时间向服务器发一简单的数据（这数据称为心跳），告诉服务器我还活着即可。
当服务器长时间没收到心跳，就可以判断这设备出问题了，向使用者发出提醒警告（邮件、短信、微信）。

### 重新连接
当电脑出现无法解决的异常时，最后的大招就是重启电脑，硬件设备也是。另外一点，网络是不稳定的，断开连接是不可抗拒的，所以对于设备来说拥有重新建立连接能力是必须要有的。

### 日志记录
任何开发都需要日志记录，否则出现问题很难调试。
注意服务器的日志要每隔一段时间就要处理以防把服务器磁盘撑爆。

### 服务器监控
设备挂了，服务器可以根据心跳知道这设备挂了。但服务器挂了，怎么办？这就需要监控服务器。
可以对外提供一个API，第三方服务用这个API每隔一段时间使用一次，若失败证明服务器出现问题了，就向管理员发出提醒警告。

### 客户端与服务器之间的通信协议
http用起来最成熟最方便。由于物联网项目不像访问网页，网页加载完就没数据了，数据是不断地更新，http轮询太浪费资源，所以再辅以websocket协议。（socketio库）
为了安全起见，微信小程序是强制要求走https，websocke也要加密，所以也是要挂证书的。

### 设备与服务器之间的通信协议
一开始为了方便，使用http。由于存在控制问题，所以使用http同样会被迫http轮询，存在性能不好，浪费流量的问题。
所以到后期，会自定义设计一个TCP协议通信格式，为了有良好的扩展性，设计时使用TLV格式。
对于不在乎那点流量性能，使用MQTT协议能获得更好的扩展性，与更快的开发速度。

### 调试工具
物联网存在很多环节，所以必须要有一个调试工具快速定位到是哪个环节出问题，到底是设备出故障了，还是通信出异常了，还是服务器程序里的bug？
快速定位问题的速度代表着这个系统的成熟程度。


### 向我咨询
如果你正在开发物联网项目，卡住了很迷茫，不知道怎么继续搞下去，需要向有经验的人咨询一下，可以来值乎付费咨询。大家都是脚踏实地的工程师，我回答不了的问题不收你钱，我能回答的保证把你的迷茫扫空。
![值乎提问](http://ww1.sinaimg.cn/large/005BIQVbgy1ft1c0p6d4uj30e50idn0d.jpg)


## 起源
2017年毕业时做的毕业设计是一个物联网项目，硬件上是STM32+ESP8266，自己搭服务器（nodejs+mongodb），客户端做网站做微信小程序（我觉得我是第一个用微信小程序做毕设的人），打算将这个项目重构并写出教程，针对硬件开发人员写的全栈应用开发入门实战。2018年开始写这教程时，我的水平是不足一年工作经难的初级全栈工程师+ 略懂硬件开发。
由于整个教程在缓慢更新中，遇到任何疑惑欢迎进Q群交流提出：__638456092__。
## 如何开始
1. 在github上点击下载，并解压
![在github上下载](http://ww1.sinaimg.cn/large/005BIQVbgy1fsr38x82u2j31hc0t4adw.jpg)
2. 打开文件夹，点击`第一次阅读.html`，先从整体阅读整个教程的特点，先从整体了解思路。
3. Part1内容：点击`Part1/index.html`正式开始阅读，其它部分同理。

学习过git：
因为整个教程在不断更新中，进入文件并运行`git pull`即可更新。
![git clone示意](http://ww1.sinaimg.cn/large/005BIQVbgy1fqtnqg91l9g31h30rmu0z.gif)

## 教程目录 与 安排
章节 | 情况 | 备注
------------ | ------------- | -------------
Part1 | 基本完成 |
Part2 | 基本完成 |
Part3 | 基本完成 |有待优化
Part4 | 暂缓更新 | 
Part5 | 暂缓更新 | 
Part6 | 暂缓更新 | 

虽然整个教程没有完成，要点已经列出来，并不影响你们提前学习~
### Part1
整个项目介绍并让大家先简单地运行起来~ 包含ESP8622烧录固件，各环节自调与联调。
- 前提：有一定硬件调试经验（USART串口调试，AT指令）
- 目标：把Part1 的demo运行起来，实现本地WIFI下手机监控硬件。
- 关键词：STM32、串口调试、AT指令、ESP8266、git、网络调试助手
### Part2
讲解Part1 demo，包含静态网页制作，express框架。
- 目标：简单的网页开发与Nodejs应该能入门了，有能力修改出自己想要的页面效果。
- 关键词：HTML、Javascript、CSS、Jquery、bootstrap、w3cschool、菜鸟教程、《深入浅出nodejs》、《七天学会NodeJS》、TCP服务器、express
### Part3
将Part1 demo运行在云服务器上，主要是学习linux（ubuntu），云服务器各种折腾。
- 目标：Linux初步入门、部署到云服务器上，此时真正实现远程监控硬件。最后讨论一下demo1的不足，向demo2进化。
- 关键词：ubuntu、云服务器、vi、《鳥哥的 Linux 私房菜》、bash、Xshell、winSCP
### Part4
引入mongoDB数据库，将数据保存到数据库里，并将历史数据可视化。  
（可选学）并讲一些协议，包含TCP协议(讲一下基于TCP自定义自己的协议规则)，HTTP协议，websocket协议。然后会进行优化，如把HTTP轮询换成websocket协议。
- 目标：完成Part4 demo，数据可视化会让整个效果更炫，增加实时性，这个物联网项目基本成型。
- 关键词：《计算机网络》、tcpdump、wireshark、《TCP/IP详解》、socketio库、echart库、TLV格式
### Part5
Part5及往后看阅读情况写吧。
各种性能测试与优化，ESP8266的AT固件改成自己编译的固件（这个蛮难搞的），说一下各物联网平台。
- 关键词：redis、nginx、CDN
### Part6
使用electron开发PC桌面软件，使用cordova开发手机APP。开发微信小程序。
- 关键词：electron、cordova
### 这难吗？
![教程不难](http://ww1.sinaimg.cn/large/005BIQVbgy1fss6qz59w6j30jj0a2t9h.jpg)

## 声明
保留一切权利，禁止商业转载，非商业转载时必须保留此声明与网址：https://github.com/alwxkxk/soft-and-hard。


