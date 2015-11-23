title: Backbone学习笔记
date: 2014-11-12 22:24:23
categories: blog
tags:
---
## Backbone简介
> 补充官网或者其他博客的介绍
[中文文档](http://www.csser.com/tools/backbone/backbone.js.html)
[jQuery源码分析](http://www.cnblogs.com/nuysoft/archive/2011/11/14/2248023.html)
[Backbone0.9.1源码分析分析系列（停止更新）](http://www.cnblogs.com/nuysoft/archive/2012/03/14/2395260.html)
[Backbone源码分析-Backbone架构+流程图](http://www.cnblogs.com/nuysoft/archive/2012/03/19/2404274.html)


```javascript
var test = {
  haha : function (a) {
    if (!a) {
      return this.xixi();
    }
  },
  xixi : function () {
	alert("xixi");
  }
}

test.haha();

// print "xixi"
```
## Backbone适用领域
1. 适合于单页面应用，并且页面上有大量数据模型，模型之间需要进行复杂的信息沟通。Backbone 在这种场景下，能很好的实现模块间松耦合和事件驱动。 例如新版有道笔记网页版([note.youdao.com](http://note.youdao.com))。


## Backbone设计思路

## 实例