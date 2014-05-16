---
layout: post
title: "View-Laye协作的学习笔记"
date: 2014-05-16 07:31:38 +0800
comments: true
categories: 

---


闲言
----------

又到了周五，一周就这样灰溜溜的过去了，想着即将到来的周末还是挺激动的，虽然周日被公司活动无情的占用了，虽然得到晚上9点半后才能回来，虽然。。。。。。

这是我的第三篇博客文章，到目前为止感觉还是挺不错的，挺有冲动写文章的，但是，总是苦于不知道写什么。建这个博客的初衷是为了记录自己的生活、工作和学习。曾经志向远大，1、每读完一本书，就在这上面写一篇读书笔记（今年到现在为了读了十多本，还没写过一篇）2、每看到一篇好的英语博客，都把它翻译到这里来（medium.com已经收藏了一堆的文章、RSS里也藏了一堆、Pocket里也有一堆） 3、每学习新的技术知识就这里写几篇总结文章（最近用Mantle和AF用的巨爽，但是也是没有什么行动）。所以，问题其实也并不是不知道写什么，而是自己略懒，同时，时间管理上还是存在很多问题。但是无论如何，现在已经开始了也为时不晚，就像那天姐夫发给我的正能量说的
    
    人生永远都可以随时开始，关键在于你敢不敢果断的转身；任何事都可能有好结果，重要的是你有没有承受光明到来之前的黑暗
    
还有一件事就是，决定以后无论什么文章，都要在最前面加上这个“闲言”栏目，先扯扯人生、谈谈理想、抒发抒发感慨，再板砖（感觉有点像那些饱读成功学书籍的人，在每次出门前都要对着镜子里的自己高喊“我一定可以成功的！我是最棒的！加油！加油！”）。


正文
-----------

这是一篇学习笔记，来自于这两天学习[View-Layer 协作](http://objccn.io/issue-12-4/)这篇文章，这篇是微博上的大神[@onevcat](http://weibo.com/onevcat)译自[objc.io](http://www.objc.io/)的[View-Layer Synergy](http://www.objc.io/issue-12/view-layer-synergy.html)。看完这篇文章后，发现自己的两点问题，
     
     1、在学习时，动手实践的精神和思考问题的深度还远远不够，往往总是浅尝则止
     2、在学习官方文档的时候粗心大意，细节都被我忽略了，当然，这个跟本人的英语水平也有一定必然的   
        联系，所以，还是多背单词、少睡觉、多查字典、少偷懒。

View和Layer的关系
----------

1、所有的View都是由一个底层的Layer来驱动的，View其实直接从Layer对象中获取了绝大多数他所需要的数据，因此，对于Layer的修改将会自动映射到所对应的View上，所以，你可以通过修改Layer或者View的属性来达到相同的效果；

2、存在单据的Layer，如[AVCaptureVideoPreviewLayer](https://developer.apple.com/library/mac/documentation/AVFoundation/Reference/AVCaptureVideoPreviewLayer_Class/Reference/Reference.html) 和 [CAShapeLayer](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/Reference/Reference.html)，不需要附加到View上就可以在屏幕上显示内容；

3、以上两种情况，都是Layer在起决定作用，但是附加到View上的Layer和单独的Layer在行为上还是稍有不同的。改变一个单独的Layer的任何animatable属性，都会触发一个从旧值过度到新值的默认简单动画，然后如果改变一个View中的Layer的同一个属性，它只会从这一帧直接跳到下一帧，因为当Layer附件到View上时，默认的隐式动画的Layer行为就不起作用了；

>
If you want to use Core Animation classes to initiate animations, you must issue all of your Core Animation calls from inside a view-based animation block. The UIView class disables layer animations by default but reenables them inside animation blocks. 

Layer是如何执行动画的？CAAction
-------------------------

4、无论何时一个可动画的Layer属性改变时，Layer都会寻找并运行合适的‘action’来实行这个改变，在Core Animation的专业术语中把这样的动画统称为动作（[CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html)）,[CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html),从技术上来说，这是一个接口，并可以用来做各种事情。但是实际中，某种程度上你可以只把它理解为用来处理动画。

5、在改变一个Layer的属性，并执行一个动作([CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html))之前，Layer需要找到相应的动作对象([CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html))。当一个恰当的动作事件发生在Layer时，Layer对象调用 `actionForKey:` 方法来寻找相应的动作对象([CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html))。我们可以在某些节点提供我们需要的特定动作([CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html))。

6、Layer将像文档中所写的那样去寻找([CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html))，整个过程分为五部，第一步为Layer通过向他的delegate发送`actionForLayer:forKey:` 消息来询问提供一个对象属性变化的[CAAction](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CAAction_protocol/Introduction/Introduction.html), delegate可以通过返回以下三者之一来进行响应：
    
    (1)、返回一个动作对象，这种情况下Layer将使用这个动作。
    (2)、返回一个nil, 这样layer会到其它地方继续寻找
    (3)、返回一个NSNull对象，告诉layer这里不需要执行一个动作，搜索也会就此结束
    
[Core Animation文档中如此描述:](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/ReactingtoLayerChanges/ReactingtoLayerChanges.html#//apple_ref/doc/uid/TP40004514-CH7-SW1)

>Core Animation looks for action objects in the following order:

>1、If the layer has a delegate and that delegate implements the actionForLayer:forKey: method, the layer calls that method. The delegate must do one of the following:
>
>·Return the action object for the given key.

>·Return nil if it does not handle the action, in which case the search continues.

>·Return the NSNull object, in which case the search ends immediately.

>2、The layer looks for the given key in the layer’s actions dictionary.

>3、The layer looks in the style dictionary for an actions dictionary that contains the key. (In other word, the style dictionary contains an actions key whose value is also a dictionary. The layer looks for the given key in this second dictionary.)

>4、The layer calls its defaultActionForKey: class method.

>5、The layer performs the implicit action (if any) defined by Core Animation.


UIView的动画Block解析
---------------------

7、当Layer在背后支持一个View的时候，View就是它的delegate。在iOS中，如果Layer与一个UIView对象关联时，view就是它的delegate。
	
	“在 iOS 中，如果 layer 与一个 UIView 对象关联时，这个属性必须被设置为持有这个 layer 的那个 view”

8、属性改变时 layer 会向 view 请求一个动作，而一般情况下 view 将返回一个 NSNull，只有当属性改变发生在动画 block 中时，view 才会返回实际的动作。

9、对于 view 中的 layer 来说，对动作的搜索只会到第一步为止，即返回一个默认动画或者NSNull。

10、插播：值得注意的是打印出的 NSNull 是带着一对尖括号的 ("<null>")，这和其他对象一样，而打印 nil 的时候我们得到的是普通括号((null))。

11、当属性在动画block中改变时，view将向layer返回一个基本的动画，然后动画通过通常的addAnimation:forkey:方法被添加到layer中，就像显示地添加动画那样。

输出值：
	
	<CABasicAnimation:0x8c73680; 
      delegate = <UIViewAnimationState: 0x8e91fa0>;
      fillMode = both; 
	  timingFunction = easeInEaseOut; 
      duration = 0.3; 
	  fromValue = NSPoint: {5, 5}; 
	  keyPath = position>

12、当动画刚被添加到layer时，属性的新值还没有被改变。在构建动画时，只有fromValue被显示地指定了。在[CABasicAnimation的文档](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CABasicAnimation_class/Introduction/Introduction.html)中可以看到具体的关于`fromeValue`、`toValue`、`byValue`的值的问题：

>The fromValue, byValue and toValue properties define the values being interpolated between. All are optional, and no  more than two should be non-nil. The object type should match the type of the property being animated.

>The interpolation values are used as follows:

>·Both fromValue and toValue are non-nil. Interpolates between fromValue and toValue.

>·fromValue and byValue are non-nil. Interpolates between fromValue and (fromValue + byValue).

>·byValue and toValue are non-nil. Interpolates between (toValue - byValue) and toValue.
		
>·fromValue is non-nil. Interpolates between fromValue and the current presentation value of the property.

>·toValue is non-nil. Interpolates between the current value of keyPath in the target layer’s presentation layer and toValue.

>·byValue is non-nil. Interpolates between the current value of keyPath in the target layer’s presentation layer and that value plus byValue.

>·All properties are nil. Interpolates between the previous value of keyPath in the target layer’s presentation layer and the current value of keyPath in the target layer’s presentation layer.

13、关于上面输出的delegate，是一个UIViewAnimationState的私有类对象，主要用来维护动画的一些状态，如持续时间、延时、重复次数等等。还负责对一个栈做push和pop,这是为了在多个动画block嵌套时，能够获取正确的动画状态 可以看看[dump出来的头文件](https://github.com/rpetrich/iphoneheaders/blob/master/UIKit/UIViewAnimationState.h)

14、UIViewAnimationState对象作为UIView返回给Layer的动画的delegate实现了`animationDidStart:` 和 `animationDidStop:finished:`方法，并将信息传给了自己的delegate, 这个delegate是一个私有类：UIViewAnimationBlockDelegate，[dump出来的头文件](https://github.com/EthanArbuckle/IOS-7-Headers/blob/master/Frameworks/UIKit.framework/UIViewAnimationBlockDelegate.h)，这是一个很小的类，只负责一件事情：响应动画的delegate回调并且执行相应的block。
