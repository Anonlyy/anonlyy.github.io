---
title: HTML5.1和HTML5.2的新特性
date: 2018-03-08 16:42:05
tags:
---

HTML5.1和HTML5.2提供很多新特性,但大部分都比较鸡肋,以下只列举部分有实际用处的特性.

### 1.为响应设计定义多个图像资源

#### srcset 图像属性

srcset 属性让你可以指定一个多个可选的图像来源，对应于不同的像素分辨率。它将允许浏览器根据用户设备的不同选择合适质量的实现来进行显示。例如，对于使用网络比较慢的移动设备的用户，显示一张低分辨率的图片会比较好。

你可以使用 srcset 属性并且带上它自有的 x 修饰符来描述每一个图片的像素比例, 如果用户的像素比例等于 3，就会显示 high-res 这张图片。

	<img src="clicks/low-res.jpg" srcset="clicks/low-res.jpg 500w, clicks/medium-res.jpg 1000w,clicks/high-res.jpg 1600w">


#### picture标签
picture 元素让你可以针对不同的屏幕尺寸声明图片。这个可以通过用 <picture> 元素封装 <img> ，并且描述多个 <source> 子元素来实现。

`<picture>` 标记单独使用并不会显示任何东西。你已经被假定会声明默认的图像来源作为 src 属性的取值，而可选的图像来源则会放在 scrset 属性之中，如下所示：

	<picture> 
		<source srcset="mobile.jpg, mobile-hd.jpg 2x" media="(max-width: 360px)"> 
		<source srcset="large.jpg, large-hd.jpg 2x" media="(min-width: 1920px)"> 
		<img src="default.jpg" srcset="default-hd.jpg 2x" alt="your image"> 
	</picture>



### 2.iframe支持更多的权限

#### iframe可以使用支付请求接口 allowpaymentrequest
在HTML5.2之前.对于支付请求的API是不能在iframe中来完成的.所以每次我们在进行移动支付时都需要跳转到另外一个支付页面才能完成付款.而现在,使用allowpaymentrequest属性应用在iframe上

	<iframe allowpaymentrequest>

这样,就可以让iframe使用`Payment Request API`从而让

>嵌入了第三方内容的页面能够控制该第三方内容是否可向用户请求获取支付凭证，进而让可嵌入的购物车工具可以利用Payment Request API。

>来自Forrester的Brendan Miller阐述了支付请求API所带来的好处，他说：该新标准让开发者可以创建一个简化的结帐页面，用户可以重复使用保存的付款和地址信息来加快结账速度，并减少错误输入。

Payment Request API的兼容性:
![](https://i.imgur.com/gm6lz5c.png)




### 解决"_blank"的钓鱼攻击

