---
title: 深入研究微信小程序的wepy框架
date: 2017-12-30 13:24:28
tags:
---

## 小程序现状
微信小程序自发布到如今已经有半年多的时间了,凭借微信平台的强大影响力，越来越多企业加入小程序开发。 小程序于传统web页和APP比相比，有以下优势： 

1. 小程序拥有更多的能力，包括定位、录音、文件、媒体、各种硬件能力等，想象空间更大 
2. 运行在微信内部，体验更接近APP
3. 在过度竞争的互联网行业中，获取一个有效APP用户的成本已经非常高了，小程序相比APP更加轻量、即用即走， 更容易获取用户

## 小程序问题

从**开发角度**来讲，小程序官方封装了很多常用组件给开发带来很多便利性，但同时也带来很多不便： 

1、小程序重新定义了DOM结构，没有`window`、`document`、`div`、`span`等HTML标签，小程序只有`view`、`text`、`imag`e等封装好的组件，页面布局只能通过这些基础组件来实现，对开发人员来讲需要一定的习惯转换成本 

2、小程序不推荐直接操作DOM（仅仅从2017年7月开始才可以获取DOM和部分属性），如果不熟悉`MVVM`模式的开发者， 需要很高的学习成本

3、小程序没有`cookie`，只能通过`storage`来模拟各项`cookie`操作（包括`htt`p中的`setCookie`也需要自行处理）

## Wepy框架

基于小程序存在的问题,腾讯的官方团队推出了`wepy`框架，该框架是腾讯内部基于小程序的开发框架，设计思路基本参考**VUE**，开发模式和编码风格上80%以上接近VUE，开发者可以以很小的成本从VUE开发切换成小程序开发。
WePY 是一款让小程序真正支持组件化开发的框架，通过预编译的手段让开发者可以选择自己喜欢的开发风格去开发小程序。框架的细节优化，`Promise`，`Async`、`await`的引入都是为了能让开发小程序项目变得更加简单，高效。

### Wepy框架的优势

**1.开发模式容易转换** 
`wepy`在原有的小程序的开发模式下进行再次封装，更贴近于现有`MVVM`框架开发模式。框架在开发过程中参考了 一些现在框架的一些特性，并且融入其中，以下是使用wepy前后的代码对比图。

官方DEMO代码：


	/index.js

	//获取应用实例
	
	var app = getApp()
	
	Page({

	    data: {
	
	        motto: 'Hello World',
	
	        userInfo: {}
	
	    },
	
	    //事件处理函数
	
	    bindViewTap: function() {
	
	        console.log('button clicked')
	
	    },
	
	    onLoad: function () {
	
	        console.log('onLoad')
	
	    }
	
	})




基于wepy的实现：



	import wepy from 'wepy';
	
	
	export default class Index extends wepy.page {


	    data = {
	
	        motto: 'Hello World',
	
	        userInfo: {}
	
	    };
	
	    methods = {
	
	        bindViewTap () {
	
	            console.log('button clicked');

	        }
	
	    };
	
	    onLoad() {
	
	        console.log('onLoad');
	
	    };
	
	}

2.**真正的组件化开发**
 小程序虽然有 标签可以实现组件复用，但仅限于模板片段层面的复用，业务代码与交互事件 仍需在页面处理。无法实现组件化的松耦合与复用的效果。

	/ index.wpy

	<template>
	    <view>
	        <panel>
	            <h1 slot="title"></h1>
	        </panel>
	        <counter1 :num="myNum"></counter1>
	        <counter2 :num.sync="syncNum"></counter2>
	        <list :item="items"></list>
	    </view>
	</template>
	<script>
	
	import wepy from 'wepy';
	
	import List from '../components/list';
	
	import Panel from '../components/panel';
	
	import Counter from '../components/counter';
	
	
	export default class Index extends wepy.page {
	    config = {
	        "navigationBarTitleText": "test"
	    };
	
	    components = {
	        panel: Panel,
	        counter1: Counter,
	        counter2: Counter,
	        list: List
	    };
	
	    data = {
	        myNum: 50,
	        syncNum: 100,
	        items: [1, 2, 3, 4]
	    }
	}
	
	</script>


3.**支持加载外部NPM包** 
小程序较大的缺陷是不支持`NPM`包，导致无法直接使用大量优秀的开源内容，`wepy`在编译过程当中，会递归 遍历代码中的`require`然后将对应依赖文件从`node_modules`当中拷贝出来，并且修改`require`为相对路径， 从而实现对外部NPM包的支持。

![](https://user-gold-cdn.xitu.io/2017/9/25/5086b45c47bd07ac1419b8d1170309ad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


4.**单文件模式**
使得目录结构更加清晰 小程序官方目录结构要求app必须有三个文件`app.json`，`app.js`，`app.wxss`，页面有4个文件 `index.json`，`index.js`，`index.wxml`，`index.wxss`。而且文件必须同名。 所以使用wepy开发前后开发目录对比如下：

官方DEMO：


	project
	
	├── pages
	
	|   ├── index
	
	|   |   ├── index.json  index 页面配置
	
	|   |   ├── index.js    index 页面逻辑
	
	|   |   ├── index.wxml  index 页面结构
	
	|   |   └── index.wxss  index 页面样式表
	
	|   └── log
	
	|       ├── log.json    log 页面配置
	
	|       ├── log.wxml    log 页面逻辑
	
	|       ├── log.js      log 页面结构
	
	|       └── log.wxss    log 页面样式表
	
	├── app.js              小程序逻辑
	
	├── app.json            小程序公共设置
	
	└── app.wxss            小程序公共样式表


使用wepy框架后目录结构：

	project
	
	└── src
	
	    ├── pages
	
	    |   ├── index.wpy    index 页面配置、结构、样式、逻辑
	
	    |   └── log.wpy      log 页面配置、结构、样式、逻辑
	
	    └──app.wpy           小程序配置项（全局样式配置、声明钩子等）


5.默认使用`babel`编译，支持ES6/7的一些新特性，如promise，async/await等等

6.wepy支持使用`less`、`sass`、`Styus`；



### wepy编译原理

![](https://cloud.githubusercontent.com/assets/2182004/22774706/422375b0-eee3-11e6-9046-04d9cd3aa429.png)




### weby快速起步
weby配备了专门的构建工具脚手架`wepy-cli`协助开发,帮助快速起步


1.安装



	npm install wepy-cli -g



2.脚手架



	wepy new myproject



3.切换至项目目录



	cd myproject

4.实时编译


	wepy build --watch


`wepy`作为一款优秀的微信小程序框架，可以帮我们大幅提高开发效率，在为数不多的小程序框架中一枝独秀，希望有更多的团队选择wepy。