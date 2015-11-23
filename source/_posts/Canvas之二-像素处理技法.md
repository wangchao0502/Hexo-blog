title: Canvas之二 像素处理技法
date: 2014-11-25 22:39:03
categories: blog
tags:
- 前端
- canvas
- 图像处理
---

继续上一篇文章，这篇文章介绍通过Canvas对图片（Image对象）的像素处理方法，同样文章参考阮一峰前辈的[Canvas API文档](http://javascript.ruanyifeng.com/htmlapi/canvas.html)。

作者也曾经玩过一年多的Ps，但是一直没有深入了解过图片处理的计算方法，这里也顺便复习一下，一些概念知识片面的带过，有时间再写文章专门讨论。

## 正题

其实，HTML5中通过Canvas处理图像只需要了解两个函数就够了，`getImageData`和`putImageData`，前者是读取一个Canvas的内容，返回一个对象，我们取名imageDate，
```javascript
var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
```
其中的data属性，其实是一个一维数组，每四个值为一组[data[4k], data[4k+3]]，分别表示r，g，b光的三原色和Alpha通道（透明度），取值都是[0, 255]。

后者是一个写方法，可以将数组内容重新绘制到Canvas中。
```javascript
context.putImageData(imageData, 0, 0);
```

我们以下面的一张图片为例：

![Canvas Pic 1](/image/canvas-1.png)

首先，我们要把图片插入到Canvas中。
```javascript
var img = new Image();
img.src = "/image/canvas-1.png";
context.drawImage(img, 0, 0); // 设置对应的图像对象，以及它在画布上的位置
```

由于图像的载入需要时间，drawImage方法只能在图像完全载入后才能调用，因此上面的代码需要改写。
```javascript
var image = new Image(); 

image.onload = function() { 

    if (image.width != canvas.width)
        canvas.width = image.width;
    if (image.height != canvas.height)
        canvas.height = image.height;

    context.clearRect(0, 0, canvas.width, canvas.height);
    context.drawImage(image, 0, 0);

} 

image.src = "/image/canvas-1.png";
```
drawImage()方法接受三个参数，第一个参数是图像文件的DOM元素（即img标签），第二个和第三个参数是图像左上角在Canvas元素中的坐标，上例中的（0, 0）就表示将图像左上角放置在Canvas元素的左上角。

当我们把图片用画笔（context）绘制到画布上（Canvas），就可以用上边提到的getImageData方法获得图像的数据了。假设我们图像处理函数叫filter，那么整个处理过程如下：
```javascript
if (canvas.width > 0 && canvas.height > 0) {
    var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
    filter(imageData);
    context.putImageData(imageData, 0, 0);
}
```

## 1. 灰度效果

### 数学原理

灰度图（grayscale）就是取红、绿、蓝三个像素值的算术平均值，这实际上将图像转成了黑白形式。假定d[i]是像素数组中一个象素的红色值，则d[i+1]为绿色值，d[i+2]为蓝色值，d[i+3]就是alpha通道值。转成灰度的算法，就是将红、绿、蓝三个值相加后除以3，再将结果写回数组。

### 代码

```javascript
grayscale = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i] = d[i + 1] = d[i + 2] = (r+g+b)/3;
    }
    return pixels;
};
```

## 2. 复古效果

### 数学原理

复古效果（sepia）则是将红、绿、蓝三个像素，分别取这三个值的某种加权平均值，使得图像有一种古旧的效果。

### 代码

```javascript
sepia = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i]     = (r * 0.393)+(g * 0.769)+(b * 0.189); // red
      d[i + 1] = (r * 0.349)+(g * 0.686)+(b * 0.168); // green
      d[i + 2] = (r * 0.272)+(g * 0.534)+(b * 0.131); // blue
    }
    return pixels;
};
```


## 3. 红色蒙版效果
### 数学原理
红色蒙版指的是，让图像呈现一种偏红的效果。算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为0。
### 代码
```javascript
red = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i] = (r+g+b)/3;        // 红色通道取平均值
      d[i + 1] = d[i + 2] = 0; // 绿色通道和蓝色通道都设为0
    }
    return pixels;
};
```

## 4. 亮度效果
### 数学原理
亮度效果（brightness）是指让图像变得更亮或更暗。算法将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。
### 代码
```javascript
brightness = function (pixels, delta) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
          d[i] += delta;     // red
          d[i + 1] += delta; // green
          d[i + 2] += delta; // blue   
    }
    return pixels;
};
```

## 5. 反转效果
### 数学原理
反转效果（invert）是指图片呈现一种色彩颠倒的效果。算法为红、绿、蓝通道都取各自的相反值（255-原值）。
### 代码
```javascript
invert = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
        d[i] = 255 - d[i];
        d[i+1] = 255 - d[i + 1];
        d[i+2] = 255 - d[i + 2];
    }
    return pixels;
};
```

实例：
<div style="display: inline-block;"><button onclick="doBack();">复原</button><button onclick="doGrayscale();">灰度效果</button><button onclick="doSepia();">复古效果</button><button onclick="doRed();">红色蒙版效果</button><input text="text" placeholder="亮度参数 -255 ~ 255" id="delta"><button onclick="doBrightness();">亮度效果</button><button onclick="doInvert();">反转效果</button></div><canvas id="myCanvas" width="1300" height="400">您的浏览器不支持canvas！</canvas>
<script type="text/javascript">
var canvas = document.getElementById('myCanvas');
var context = canvas.getContext('2d');
var image = new Image();
var originImageData;
var imageData;

image.onload = function() { 

    if (image.width != canvas.width)
        canvas.width = image.width;
    if (image.height != canvas.height)
        canvas.height = image.height;

    context.clearRect(0, 0, canvas.width, canvas.height);
    context.drawImage(image, 0, 0);
    originImageData = context.getImageData(0, 0, canvas.width, canvas.height);

} 
image.src = "/image/canvas-1.png";

function doWhat(filter, delta) {
	if (canvas.width > 0 && canvas.height > 0) {
    	imageData = context.getImageData(0, 0, canvas.width, canvas.height);
    	delta ? filter(imageData, delta) : filter(imageData);
    	context.putImageData(imageData, 0, 0);
	}
}

function doBack() {
	if (canvas.width > 0 && canvas.height > 0) {
    	context.putImageData(originImageData, 0, 0);
	}
}

var grayscale = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i] = d[i + 1] = d[i + 2] = (r+g+b)/3;
    }
    return pixels;
};

var sepia = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i]     = (r * 0.393)+(g * 0.769)+(b * 0.189); // red
      d[i + 1] = (r * 0.349)+(g * 0.686)+(b * 0.168); // green
      d[i + 2] = (r * 0.272)+(g * 0.534)+(b * 0.131); // blue
    }
    return pixels;
};

var red = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i] = (r+g+b)/3;        // 红色通道取平均值
      d[i + 1] = d[i + 2] = 0; // 绿色通道和蓝色通道都设为0
    }
    return pixels;
};

var brightness = function (pixels, delta) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
          d[i] += delta;     // red
          d[i + 1] += delta; // green
          d[i + 2] += delta; // blue   
    }
    return pixels;
};

var invert = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
        d[i] = 255 - d[i];
        d[i+1] = 255 - d[i + 1];
        d[i+2] = 255 - d[i + 2];
    }
    return pixels;
};

function doGrayscale() {
	doWhat(grayscale);
}

function doSepia() {
	doWhat(sepia);
}

function doRed() {
	doWhat(red);
}

function doBrightness() {
	var delta = parseInt(document.getElementById("delta").value, 10);
	doWhat(brightness, delta);
}

function doInvert() {
	doWhat(invert);
}

</script>

