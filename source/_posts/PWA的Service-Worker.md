---
title: PWA的Service Worker
date: 2018-01-09 09:48:25
tags:
---


**PWA**(Progressive Web Apps,渐进式增强 WEB 应用)由谷歌提出，用前沿的技术开发，让网页使用如同App般的体验的一系列方案。

用来自谷歌工程师的解答`Progressive Web Apps`:

- 渐进增强 – 所有特性都是以渐进的方式增强的，在比传统网页应用更好的同时也保证了降级兼容。。
- 响应式用户界面 – 适应任何环境：桌面电脑，智能手机，笔记本电脑，或者其他设备。
- 不依赖网络连接 – 通过 Service Workers 可以在离线或者网速极差的环境下工作。
- 类原生应用 – 有像原生应用般的交互和导航给用户原生应用般的体验，因为它是建立在 app shell model 上的。
- 持续更新 – 受益于 `Service Worker` 的更新进程，应用能够始终保持更新。
- 再次访问 – 通过消息推送等特性让用户再次访问变得容易。
- 可安装 – 允许用户保留对他们有用的应用在主屏幕上，不需要通过应用商店。

其中,PWA的主要核心功能实现就是依赖`Service Worker`这个API.

----------

## Service Worker

`Service Worker` 是 Chrome 团队提出和力推的一个 `WEB API`，用于给 `web` 应用提供高级的可持续的后台处理能力。该 `WEB API `标准起草于 2013 年，于 2014 年纳入 W3C WEB 标准草案，当前还在草案阶段。

`Service Worker` 最主要的特点是：在页面中注册并安装成功后，运行于浏览器后台，不受页面刷新的影响，可以监听和截拦作用域范围内所有页面的 HTTP 请求。

基于 `Service Worker API` 的特性，结合 `Fetch API`、`Cache API`、`Push API`、`postMessage API` 和 `Notification API`，可以在基于浏览器的 web 应用中实现如离线缓存、消息推送、静默更新等 native 应用常见的功能，以给 web 应用提供更好更丰富的使用体验。


### Service Worker 特点

1. 网站必须使用 `HTTPS`。除了使用本地开发环境调试时(如域名使用 localhost)
2. 运行于浏览器后台，可以控制打开的作用域范围下所有的页面请求
3. 单独的作用域范围，单独的运行环境和执行线程
4. 不能操作页面 DOM。但可以通过事件机制来处理

----------

### Service Worker 浏览器支持情况

除了 Safari 和 IE/Edge，大部分现代浏览器都已经得到了支持。
![](https://lzw.me/wp-content/uploads/2017/03/sw-caniuse.png)

----------

### Service Worker生命周期

`Service Worker` 脚本通过 `navigator.serviceWorker.register` 方法注册到页面, 之后就可以脱离页面在浏览器后台运行:
![](https://pic1.zhimg.com/50/v2-2cf08481cdf4e1bd8448c8daef88dd76_hd.jpg)

1.**parsed**: 注册完成, 脚本解析成功, 尚未安装

2.**installing**: 对应 Service Worker 脚本 install 事件执行, 如果事件里有`event.waitUntil()`则会等待传入的 Promise 完成才会成功

3.**installed(waiting)**: 页面被旧的`Service Worker`脚本控制, 所以当前的脚本尚未激活。可以通过`self.skipWaiting()`激活新的 Service Worker

4.**activating**: 对应`Service Worker`脚本`activate`事件执行, 如果事件里有`event.waitUntil()`则会等待这个`Promise`完成才会成功。这时可以调用 Clients.claim() 接管页面

5.**activated**: 激活成功, 可以处理`fetch`, `message`等事件

6.**redundant**: 安装失败, 或者激活失败, 或者被新的 `Service Worker`替代掉


----------

## 关于事件
**install** 事件:当前`Service Worker`脚本被安装时，会触发 install 事件。

**push**事件:
push 事件是为推送通知而准备的。不过首先你需要了解一下 `Notification API` 和 `PUSH API`。

通过 `PUSH API`，当订阅了推送服务后，可以使用推送方式唤醒 `Service Worker` 以响应来自系统消息传递服务的消息，即使用户已经关闭了页面。


**online/offline**事件:
当网络状态发生变化时，会触发 `online` 或 `offline` 事件。结合这两个事件，可以与 `Service Worker` 结合实现更好的离线使用体验，例如当网络发生改变时，替换/隐藏需要在线状态才能使用的链接导航等。


**fetch** 事件：
当我们安装完`Service Worker`成功并进入激活状态后即运行于浏览器后台,我们的这个线程就会一直监控我们的页面应用,如果出现`HTTP`请求,那么就会触发`fetch`事件，并且给出自己的响应。
这个功能是十分强大的,借助 `Fetch API` 和 `Cache API` 可以编写出复杂的策略用来区分不同类型或者页面的资源的处理方式。它能够提供更加好的用户体验:
例如可以实现**缓存优先、降级处理**的策略逻辑：监控所有 http 请求，当请求资源已经在缓存里了，直接返回缓存里的内容；否则使用 fetch API 继续请求，如果是 图片或 css、js 资源，请求成功后将他们加入缓存中；如果是离线状态或请求出错，则降级返回预缓存的离线内容。

还有一些常用的资源缓存的策略:

- 网络优先: 从网络获取, 失败或者超时再尝试从缓存读取
- 缓存优先: 从缓存获取, 缓存插叙不到再尝试从网络抓取
- 速度优先: 同时查询缓存和网络, 返回最先拿到的
- 仅限网络: 仅从网络获取
- 仅限缓存: 仅从缓存获取

#### 关于Fetch API
w3c提供了一个新的`fetch API`，用于取代`XMLHttpRequest`，与`XMLHttpRequest`最大不同有两点：

1.fetch()方法返回的是Promise对象，通过then方法进行连续调用，减少嵌套。ES6的Promise在成为标准之后，会越来越方便开发人员。

2.fetch api与XMLHttpRequest相比，更加简洁，并且提供的功能更全面，资源获取方式比ajax更优雅。


		/* 由于是get请求，直接把参数作为query string传递了 */
		var URL = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key=your_api_key&format=json&nojsoncallback=1&tags=penguins';
		 
		function fetchDemo() {
		  // fetch(url, option)支持两个参数，option中可以设置header、body、method信息
		  fetch(URL).then(function(response) {
		    // 通过promise 对象获得相应内容，并且将响应内容按照json格式转成对象，json()方法调用之后返回的依然是promise对象
		    // 也可以把内容转化成arraybuffer、blob对象
		    return response.json();
		  }).then(function(json) {
		    // 渲染页面
		    dosomething(json)；
		  });
		}
		 
		fetchDemo();

关于Fetch API,可参考这篇文章[传统 Ajax 已死，Fetch 永生](https://github.com/camsong/blog/issues/2 "传统 Ajax 已死，Fetch 永生")

#### Cache API

