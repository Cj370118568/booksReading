# booksReading
平时看书看到的一些小知识
-----------------

* View && UIWindow
	+ UIWindow不一定只有一个，但是KeyWindow只有一个
	+ UIWindow有3个优先级别，分别为Normal，StatusBar，Alert
	+ UIWindow有4个相关的通知，分别为：
		UIWindowDidBecomeVisibleNotification
		UIWindowDidBecomeHiddenNotification
		UIWindowDidBecomeKeyNotification
		UIWindowDidResignKeyNotification
	+ windows之间可以通过makeKeyAndVisible和resignKeyWindow来进行切换

####参考：
+ [理解UIWindow及其常见使用场景] (http://www.jianshu.com/p/1e402922ee32/)

+ [iOS 关于UIWindow 的认识] (http://www.mamicode.com/info-detail-462727.html)