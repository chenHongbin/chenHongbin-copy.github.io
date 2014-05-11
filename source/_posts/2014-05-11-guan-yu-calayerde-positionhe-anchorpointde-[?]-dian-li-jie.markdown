---
layout: post
title: "关于CALayer的position和anchorPoint的一点理解"
date: 2014-05-11 12:13:48 +0800
comments: true
categories: 
---

引言
-----------------------------------------

翻看记录，发现自从3月底搭建完这个blog写了第一篇文章后，就再也没有写过任何东西了，只能在心底强烈的鄙视自己一番。鉴于最近在新项目中学到不少东西，同时，发现很多东西没有掌握，所以计划在接下来的一段时间内，好好学习，偶尔记录，希望能多写点东西。

关于CALayer
-------------------------------------------
在Core Animation中最基本也是最重要的内容就是CALayer，每个UIView都有一个CALayer对象。CALayer在很多方面与UIView非常相似。它拥有位置、大小、变形和内容。但是CALayer并不是用来替代UIView对象的，相反，它是和UIView一起协作的。UIView是一个很重量级的对象，它管理绘制与事件处理（尤其是触摸事件），CALayer则完全关乎绘制，UIView依靠CALayer来管理绘制，通过这样的分工两者就能协作的更好。

CALayer的几何属性
-------------------------------------------
就像前面说的，CALayer拥有自己的位置（position）、大小(bounds,frame)、变形(transform)和内容(content)，同时，它还拥有一个UIView所没有的属性anchorPoint。其中position、bounds、frame是基于点坐标系的（point-based coordinate systems），而anchorPoint是基于单元坐标系的（unit coordinate systems），即X坐标和Y坐标的取值在0-1之间。bounds和frame的意思比较好理解，farme为CALayer在其父层（superLayer）上的位置和大小，而bounds则是可以简单理解为CALayer的大小，其X坐标和Y坐标为0。剩下的position和anchorPoint则是最容易让人困惑的。

position
-------------------------------------------
在CALayer.h文件中我们可以看到:

	/* The position in the superlayer that the anchor point of the layer's
     * bounds rect is aligned to. Defaults to the zero point. Animatable. */
     
    @property CGPoint position;

即position是CALayer的anchorPoint在其父层（superLayer）的关联位置。

anchorPoint
-------------------------------------------
同样，在CALayer.h文件中我们可以看到:

    /* Defines the anchor point of the layer's bounds rect, as a point in
     * normalized layer coordinates - '(0, 0)' is the bottom left corner of
     * the bounds rect, '(1, 1)' is the top right corner. Defaults to
     * '(0.5, 0.5)', i.e. the center of the bounds rect. Animatable. */

     @property CGPoint anchorPoint;
     
即anchorPoint的值是相对bounds的比例值来确定的，在坐标系的左上角和右下角anchorPoint的值分别为（0，0）和（1，1），需要注意点是，文档中说的是左下角和右上角，这是在OS X上，iOS与OS X相反。anchorPoint的默认值为（0.5，0.5）即CALayer的中心位置。

接下来看下面两张图，只看iOS部分即可，

![Alt ahchor1](/images/2014-05-10-anchor/layer_coords_anchorpoint_position_2x.png "ahchor1")

图1

![Alt ahchor2](/images/2014-05-10-anchor/anchorpoint2.jpg "ahchor2")

图2

在图1中，我们可以看到，position的值是anchorPoint点在superLayer中的坐标位置。所以position点是相对于superLayer的，而anchorPoint点是相对于CALayer本身的。两者是相对不同的坐标空间的一个重合点。

在图2中，我们可以看到，anchorPoint的作用，其实是在CALayer做动画时的一个锚点，所有的变化是以anchorPoint为锚点进行的，相同的动画，不同的锚点，效果是全然不同的。具体的描述可以看[彻底理解position与anchorPoint](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)这篇博文。

anchorPoint、position和frame之间的关系
--------------------------------------------
在官方的[Core Animation文档中](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)有这样的一句话：
>
 The bounds defines the coordinate system of the layer itself and encompasses the layer’s size on the screen. The position property defines the location of the layer relative to its parent’s coordinate system. Although layers have a frame property, that property is actually derived from the values in the bounds and position properties and is used less frequently.

也就是说CALayer的frame是由CALayer的bounds属性和position属性确定的，bounds决定大小，position决定位置
     
     frame.origin.x = position.x - anchorPoint.x * bounds.size.width;
     frame.origin.y = position.y - anchorPoint.y * bounds.size.height;
     frame.size.width  = bounds.size.width;
     frame.size.height = bounds.size.height;
	
为此我写了一段测试代码
    
    CALayer *testLayer = [[CALayer alloc] init];
    testLayer.frame = (CGRect){
        .origin.x = 10,
        .origin.y = 10,
        .size.width =  100,
        .size.height = 100
    };
    testLayer.backgroundColor = [UIColor redColor].CGColor;
    [self.view.layer addSublayer:testLayer];
    
    NSLog(@"testLayer frame = %@",NSStringFromCGRect(testLayer.frame));
    NSLog(@"testLayer bounds = %@",NSStringFromCGRect(testLayer.bounds));
    NSLog(@"testLayer position = %@",NSStringFromCGPoint(testLayer.position));
    NSLog(@"testLayer anchorPoint = %@",NSStringFromCGPoint(testLayer.anchorPoint));
    
    NSLog(@"\n");
    NSLog(@"Change Position(20, 20)");
    testLayer.position = CGPointMake(20, 20);

    NSLog(@"testLayer frame = %@",NSStringFromCGRect(testLayer.frame));
    NSLog(@"testLayer bounds = %@",NSStringFromCGRect(testLayer.bounds));
    NSLog(@"testLayer position = %@",NSStringFromCGPoint(testLayer.position));
    NSLog(@"testLayer anchorPoint = %@",NSStringFromCGPoint(testLayer.anchorPoint));
    

    NSLog(@"\n");
    NSLog(@"Change anchorPoint(0.4, 0.4)");
    testLayer.anchorPoint = CGPointMake(0.4, 0.4);
    
    NSLog(@"testLayer frame = %@",NSStringFromCGRect(testLayer.frame));
    NSLog(@"testLayer bounds = %@",NSStringFromCGRect(testLayer.bounds));
    NSLog(@"testLayer position = %@",NSStringFromCGPoint(testLayer.position));
    NSLog(@"testLayer anchorPoint = %@",NSStringFromCGPoint(testLayer.anchorPoint))
    
其输出为：

	testLayer frame = ((10, 10), (100, 100))
	testLayer bounds = ((0, 0), (100, 100))
    testLayer position = (60, 60)
    testLayer anchorPoint = (0.5, 0.5)
 
    Change Position(20, 20)
    testLayer frame = ((-30, -30), (100, 100))
    testLayer bounds = ((0, 0), (100, 100))
    testLayer position = (20, 20}
    testLayer anchorPoint = (0.5, 0.5)

    Change anchorPoint(0.4, 0.4)
    testLayer frame = ((-20, -20), (100, 100))
    testLayer bounds = ((0, 0), (100, 100))
    testLayer position = (20, 20)
    testLayer anchorPoint = (0.40000001, 0.40000001)
		 
从中我们可以验证我们之前的公式是正确的，即frame的origin是由position和anchorPoints共同决定的，而frame的size则是由bounds决定的。

另外一个问题就是，单方面修改layer的position位置或者是anchorPoint，其两者互不影响，受影响的只会是frame.origin，也就是layer坐标原点相对superLayer会有所改变。

最后，在[彻底理解position与anchorPoint](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)这篇博文的有说到一点，
>
Apple doc中还有一句描述是这样的：

>> When you specify the frame of a layer, position is set relative to the anchor point. When you specify the position of the layer, bounds is set relative to the anchor point.

>大意是：当你设置图层的frame属性的时候，position根据锚点（anchorPoint）的值来确定，而当你设置图层的>position属性的时候，bounds会根据锚点(anchorPoint)来确定。
>这段翻译的上半句根据前面的公式容易理解，后半句可能就有点令人迷惑了，当修改position时，bounds的width与>height会随之修改吗？其实,position是点，bounds是矩形，根据锚点(anchorPoint)来确定的只是它们的位置，而不>是内部属性。所以，上面这段英文这么翻译就容易理解了：

>>当你设置图层的frame属性的时候，position点的位置（也就是position坐标）根据锚点（anchorPoint）的值来确 定，而当你设置图层的position属性的时候，bounds的位置（也就是frame的orgin坐标）会根据锚点(anchorPoint)来确定

最后
--------------------------------------------
终于完成第一篇技术博客，发现确实写文章和写代码很像，对于如何提高自己的思路和条理有大帮助，应该好好把握，多写写，哈哈！

### 参考

- [彻底理解position与anchorPoint](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)

- [Core Animation Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)


















