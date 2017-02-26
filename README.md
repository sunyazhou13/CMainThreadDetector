# CMainThreadDetector
##起
软件发布后，偶尔会有这样的反馈信息：“打开某个页面时卡了一会儿”、“在某种情况下，做某种操作时软件很卡”。  
但是，开发者拿起手机打开某个页面，试呀试呀试呀试，难以重现。。。  
这些问题，  
可能是在特定用户的特定环境下才会发生，  
可能需要极特殊的时机下才会出现，  
难以重现，难以发觉。大多数情况下只能去沿着代码逻辑细细推敲。  
于是，有了这个MainThreadDetector的需求。  
##承  
（以下节选自：[微信iOS卡顿监控系统](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207890859&idx=1&sn=e98dd604cdb854e7a5808d2072c29162&scene=21#wechat_redirect)）
发生卡顿，原因大致可分为一下几种：  
* 抢锁：主线程需要访问 DB，而此时某个子线程往 DB 插入大量数据。通常抢锁的体验是偶尔卡一阵子，过会就恢复了。
* 主线程大量 IO：主线程为了方便直接写入大量数据，会导致界面卡顿。
* 主线程大量计算：算法不合理，导致主线程某个函数占用大量 CPU。
* 大量的 UI 绘制：复杂的 UI、图文混排等，带来大量的 UI 绘制。  
针对这些问题，如何解决呢？  
* 抢锁不好办，将锁等待时间打出来用处不大，我们还需要知道是谁占了锁。
* 大量 IO 可以在函数开始结束打点，将占用时间打到日志中。
* 大量计算同理可以将耗时打到日志中。
* 大量 UI 绘制一般是必现，还好办；如果是偶现的话，想加日志点都没地方，因为是慢在系统函数里面。  
如果可以将当时的线程堆栈捕捉下来，那么上述难题都迎刃而解。主线程在什么函数哪一行卡住，在等什么锁，而这个锁又是被哪个子线程的哪个函数占用，有了堆栈，我们都可以知道。自然也能知道是慢在UI绘制，还是慢在我们的代码。
所以，思路就是监控主线虫，如果发现有卡顿，就将堆栈 dump 下来。
