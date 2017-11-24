#Chapter 5.Touches
-
###Introduction

手指点击在屏幕上，就会产生一个touch的实例。UIResponder是touches的潜在接受者，UIView就是UIResponder（还有其他的UIResponder子类但是其他的子类都是不可见的，UIView可见是由于它的Layer）。在实际中，系统会把UITouch对象包装成UIEvent传递到app中，然后app中找到合适的UIView来响应。

###Touch Events and Views
+ multitouch sequence
+ UIEvent就是对用户触摸操作的封装，一个UIEvent包含一个或多个UITouch对象。每个UITouch对象跟一个手指相关。一旦UITouch创建，它代表着相关的手指离开屏幕前所作的操作
+ UITouch的4种状态：
	+ .began
	+ .moved
	+ .stationary
	+ .ended

	这四种状态足以描述出手指能做的所有操作，其实还有另外一种状态
	
	+ cancelled

	系统会向app发出这几种状态（封装成UIEvent）
+  	当UITouch第一次出现（.began），你的app会找到响应的UIView，然后这个view就会成为UITouch对象的view属性，从此以后，UITouch就会在手指离开屏幕之前一直与这个view绑定。
+   组成UIEvent的UITouches有可能与不同的view绑定，因此，UIEvent会传递到与组成这个UIEvent的UITouches相关的所有view。
+   反过来说，若是一个UIEvent传递到某个view上，是因为UIEvent中的至少一个UITouch的view是当前view
+   如果UIEvent里面的每一个UITouch绑定同一个UIView，并且状态是.stationary的话，这个UIEvent是不会传递到UIView的，因为没有任何的事情会发生。


###Receiving Touches
+ 一个UIResponder，或者说是一个UIView,有四个跟UITouch对应的方法。通过调用这四个方法之中的一个来把UIEvent传递到UIView。

	+ touchesBegan(_:with:)
	+ touchesMoved(_:with:)
	+ touchesEnded(_:with:)
	+ touchesCacelled(_:with:)

	这四个方法的参数是：
	
	+ The relevant touches

	UIEvent里面包含的一个或多个UITouch，以Set的形式传递过来。
	
	+ The event
	
	UIEvent的对象。包含一个touches的集合，你可以通过allTouches来获取。这个集合包含第一个参数里面的集合，但不局限于此。

+ UITouch有一些常用的方法和属性

	+ location(in:),previousLocation(in:)
	+ timestamp
	+ tapCount
	+ view
	+ majorRadius,majorRadiusTolerance
	
###Restricting Touches
+ 通过系统级别的beginIgnoringInteractionEvents属性，可以完全忽略所有的触摸事件。

+ 与之对应的是endIgnoringInteractionEvents。这对属性在动画或者其他长时间的操作中经常用到。

+ 一些UIView的属性也能限制特定views的触摸事件的传递。

	+ isUserInteractionEnabled
	+ alpha
	+ isHidden
	+ isMultipleTouchEnabled
	+ isExclusiveTouch

###Interpreting Touches

+ not every touch sequence can be codified through a gesture recognizer; sometimes, directly interpreting touches is the best approach.


###Gesture Recognizers

+ Gesture Recognizer Classes
+ Gesture Recognizer Conflicts
+ Subclassing Gesture Recognizers
+ Gesture Recognizer Delegate
+ Gesture Recognizers in the Nib

###3D Touch Press Gesture
+ 在支持3dtouch的设备，按压可以看做一种手势
+ 最简单的方式是使用UIPreviewInteraction这个类（iOS 10之后）
+ 有几个代理方法（UIPreviewInteractionDelegate）

	+  	previewInteractionShouldBegin(_:)
	+   previewInteraction(_:didUpdatePreviewTransition:ended)
	+   previewInteraction(_:didUpdateCommitTransition:ended:)
	+   previewInteractionDidCancel(_:)

###Touch Delivery
+ 当一个touch出现，系统会通过hit-testing来找到对此响应的view。然后这个view会与这个touch绑定，因此叫做hit-test view。UIView的一些属性如：isUserInteractionEnable，isHidden，alpha就是通过影响系统的hit-test过程来实现让系统忽略这个view的目的。
+ 每次触摸状态改变的时候，
	+ 系统都会调用自己的sentEvent(_:)方法，
	+ 传递到window，window再调用它的sentEvent(_:)方法
	+ 触摸事件第一次出现的时候，系统会考虑```isMultipleTouchEnabled```以及```isExclusiceTouch```
	+ 不违反相应规则的情况下，以下两种情况都有可能发生：
		+ 触摸事件传递到hit-test view的gesture recognizers
		+ 触摸事件传递到hit-test view本身
	+ 当触摸事件被一个gesture recognizer识别，对于所有与这个手势相关的触摸：
		+ touchsCancelled(_:for:)会传递到touch绑定的view，之后touch不再会传递给这个view
		+ 如果touch还与其他的gesture recognizer相关，那些gesture recognizer会识别失败


###Hit-Testing
Hit-testing就是系统用来决定哪一个view来响应触摸事件的过程，是通过UIView的实例方法hitTest(_:with:)来实现的，这个方法放回一个view或者nil。目的就是找出在最上层的，包含触摸点的view。逻辑如下：

1. view的hitTest(_:with:)方法首先会在view的子view上调用hitTest方法。如果是两个相同层级的view，在上面的那个view会先调用hitTest方法。
1. 一旦这些subviews的其中一个的hitTest返回view而不是nil，这个寻找过程就会停止，那一个view就会成为hit-test view。
2. 如果这个view没有subview，或者说他的所有subview都返回nil，那么这个view会自己调用point(inside:with:)这个方法，如果触摸点是在这个view里面，这个view会返回自己，这代表着自己就是hit-test view,否则会返回nil

如果一个view的isUserInteractionEnabled被设成false,或者说isHidden被设成true，或者alpha被设成0，hitTest方法直接返回nil，子view不会调用hitTest方法，自身也不会调用point(inside:with:)方法。

+ Performing Hit-Testing


+ Hit-Testing Munging

	你可以在子类中重写hitTest(_:with:)方法来影响系统的hit-test过程，从而实现某种目的。
	
	其中一种用途：当父view的clipsToBounds被设成false的时候，子view超出父view的范围你还是能看到，但是这种情况下子view超出父view的范围是不能正常接收点击事件的。这个时候重写hit-test方法就能解决这个问题。
	
	```Swift
	override func hitTest(_ point: CGPoint, with e: UIEvent?) -> UIView? {
    if let result = super.hitTest(point, with:e) {
        return result
    }
    for sub in self.subviews.reversed() {
        let pt = self.convert(point, to:sub)
        if let result = sub.hitTest(pt, with:e) {
            return result
        }
    }
    return nil
}
	```

+ Hit-Testing for layers

	正常情况下，layer是不会接收到点击事件的。然而，我们是有可能通过重写hitTest来达到这个目的。

+ Hit-Testing for drawings
+ Hit-Testing During Animation

	+ 要在动画过程中接收触摸事件是困难的，因为动画过程中view并不是在看到的位置

###Initial Touch Event Delivery
当触摸事件的状态改变时，一个包含所有touches的event就会通过调用sendEvent(_:)传到UIApplication，然后UIAppication会通过调用UIWindow的sendEvent(_:)传到UIWindow中。UIWindow会通过一系列复杂的逻辑，综合考虑每一个touch的hit-test view，父views,还有gesture recognizers,来决定其中的哪一个来相应。



###Gesture Recognizer and View
+ 触摸事件第一次出现并传递到一个gesture recognizer时也会被传递到这个触摸事件的hit-test view，gesure recognizer和hit-test view都会调用同样的触摸事件响应方法。如果加到view上的所有gesture recognizer都不能识别当前触摸事件的话，这个view就会处理这个触摸事件，跟完全没有gesture recognizer一样。
+ 触摸(touches)跟手势(gesture)是两个完全不一样的东西，有时候你会想同时响应这两者。
+ 如果加在view上的某个手势识别了当前的触摸事件，view的touchesCancelled方法会被调用，所有跟这个手势有关并且以这个view为hit-test view的触摸事件都不会被传递到这个view，也就是说这个view不再处理这些触摸事件。
+ 通过设置gesture recognizer的```cancelsTouchesInView```为false，可以更改上述的这个机制。如果加在view上的所有手势的```cancelsTouchesInView```都为false，则这个view会像没有gesture recognizer那样接收事件。
+ 假如一个手势碰巧忽略了一个手势（比如说调用了ignore(_:for:)这个方法），则view的touchesCancelled(_:with)不会被调用。所以说如果一个手势忽略了触摸事件，则它的hit-test view会像没有添加任何手势那样去处理这个触摸事件。
+ 默认情况下，UIGestureRecognizer的```delaysTouchesEnded```属性为true，因此，手势在默认情况下会使事件延迟传递到UIView。
+ 这意味着除非手势被识别出来，或者识别失败，事件才会被传递到UIView

	+ 若手势被识别出来，UIView的touchesCancelled方法会被调用。
	+ 若手势识别失败，UIView的touchesEnded方法会被调用。
+ 为什么要有这种机制？

	+ 假如有一个需要多次点击的手势，在这种机制下，第一次点击之后，因为这个手势没法判断是否成功识别，所以这一系列的触摸事件暂时不会传递到view，直到这个手势成功识别或者识别失败为止。
	+ 如果没有这种机制，第一次点击之后，UIView的touchesEnded方法就会被调用，那么这种手势就永远不可能识别成功。
	+ 所以说，gesutre recognizer的优先级比UIView要高，而这样做是很合理的。
+ 此外，你也可以把Gesture recognizer的```delaysTouchesBegan```属性设为true，这样的话

	+ 当触摸事件被Gesture recognizer处理完之后

		+ 识别成功

			UIView的touchesCancelled方法会被调用
		+ 识别失败
		
			UIView的touchesBegan方法会被调用
		
+ 但是并不推荐这么做，因为如果这样做了延时太多，在用户看来这会很奇怪。

###Touch Exclusion Logic
+ ```isMultipleTouchEnabled```以及```isExclusiveTouch```的逻辑是在UIWindow的sendEvent(_:)中定义的。
+ 如果一个新touch的hit-test view的```isMultipleTouchEnabled```为false，并且这个view已经是别的触摸手势的hit-test view。新的touch是不会再传递到这个view了，但是仍然会传递到这个view的gesture recognizers
+ 相似的，如果```xclusiveTouch```为false，则新的touch既不会传递到这个view，也不会传递到这个view的gesture recognizers。

###Gesture Recognition Logic
+ 当一个gesture recognizer识别出了手势之后，会改变很多东西。首先touches的hit-test view的touchesCancelled(_:with:)方法会被调用。然后其他的gesture recognizers也会变成fail状态
+ 如果一个event可能会被多个gesture recognizer识别，会有一个算法来判断哪个手势识别：后面加上去的手势先识别、靠近hit-test view的先识别。
+ 有很多方式可以改变这种机制

	+ Dependency order

		+ require(toFail:)
		+ shouldRequireFailure(of:)
		+ shouldBeRequiredToFail(by:)
		+ gestureRecognizer(_:shouldRequireFail)
		+ gestureRecognizer(_:shouldBeRequiredTo)
	
	+ Success into failure

		+ UIView的gestureRecognizerShouldBegin(_:)
		+ delegate的gestureRecognizerShouleBegin(_:)
	
	+ Simultaneous recognition

		+ delegate的gestureRecognizer(_:shouldRecognizeSimultaneous)
		+ 子类中的canPrevent(_:)
		+ 子类中的canBePrevented(by:)


###Touches and the Responder Chain
+ UIView是一个responder，参与响应链
+ 如果一个touch被传递到一个UIView（因为这个view是hit-test view），并且这个view没有定义好相关的触摸响应方法，就会有一个流程来找出哪一个responder定义好了相关的触摸响应方法。
+ 
