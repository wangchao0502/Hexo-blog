title: Canvas之一 旋转的太极
date: 2014-11-25 21:48:01
categories: blog
tags: 
- 前端
- canvas
---

今天看到了[康威生命游戏](http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)，有点小激动，想自己用javascript实现一个。本来想用表格的方式着色去做，但是又看到了已经有人通过HTML5的canvas完成了，代码都公布出来了[HTML5版本的生命游戏](http://blog.csdn.net/lifegame/article/details/6385081)，而且下下来就能运行。。然后就想着我可以做个更好的，但是不会canvas怎么办？学呗！经常知识栈溢出的我就是这么任性。

在寻找好的学习资料的道路上又遇到了阮一峰大神，他还是我的校友你敢信？真是学习的榜样啊！[一个好的Canvas API学习文档](http://javascript.ruanyifeng.com/htmlapi/canvas.html)。

然后就这么折腾了一小会儿，写了个简单的旋转的太极图。

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<canvas id="myCanvas" width="400" height="400">
    您的浏览器不支持canvas！
</canvas>
</body>
<script>
    var canvas = document.getElementById('myCanvas');
    var ctx = canvas.getContext('2d');
    var height = 400;
    var width = 400;
    /*
     ctx.beginPath(); // 开始路径绘制
     ctx.moveTo(20, 20); // 设置路径起点，坐标为(20,20)
     ctx.lineTo(200, 20); // 绘制一条到(200,20)的直线
     ctx.lineTo(200, 100);
     ctx.lineTo(20, 100);
     ctx.closePath();
     ctx.lineWidth = 1.0; // 设置线宽
     ctx.strokeStyle = "#CC0000"; // 设置线的颜色
     ctx.stroke(); // 进行线的着色，这时整条线才变得可见
     ctx.fillStyle = 'yellow';   // 设置矩形的填充色
     ctx.fillRect(50, 50, 200, 100); // 绘制一个有填充色的矩形
     ctx.strokeRect(10, 10, 200, 100);  // 透明矩形
     ctx.clearRect(100, 50, 50, 50);    // 清楚对应位置矩形区域
     // 设置字体
     ctx.font = "Bold 20px Arial";
     // 设置对齐方式
     ctx.textAlign = "left";
     // 设置填充颜色
     ctx.fillStyle = "#008600";
     // 设置字体内容，以及在画布上的位置， 不支持文本断行
     ctx.fillText("Hello!", 10, 50);
     // 绘制空心字， 不支持文本断行
     ctx.strokeText("Hello!", 10, 100);
     // 清空画布
     ctx.clearRect(0, 0, 400, 200);
     // 绘制圆和扇形
     ctx.beginPath();
     ctx.arc(60, 60, 50, 0, Math.PI, true);
     ctx.fillStyle = "#000000";
     ctx.fill();
     // 绘制空心圆
     ctx.beginPath();
     ctx.arc(60, 60, 50, 0, Math.PI * 2, true);
     ctx.lineWidth = 1.0;
     ctx.strokeStyle = "#000";
     ctx.stroke();
     // 根据上边画出来的，绘制一个太极图
     ctx.beginPath();
     ctx.arc(35, 60, 25, 0, Math.PI * 2, true);
     ctx.fillStyle = "#ffffff";
     ctx.fill();
     ctx.beginPath();
     ctx.arc(85, 60, 25, 0, Math.PI * 2, true);
     ctx.fillStyle = "#000000";
     ctx.fill();
     ctx.beginPath();
     ctx.arc(35, 60, 8, 0, Math.PI * 2, true);
     ctx.fillStyle = "#000000";
     ctx.fill();
     ctx.beginPath();
     ctx.arc(85, 60, 8, 0, Math.PI * 2, true);
     ctx.fillStyle = "#ffffff";
     ctx.fill();
     // 设置渐变色
     var myGradient = ctx.createLinearGradient(0, 0, 0, 160);
     myGradient.addColorStop(0, "#BABABA");
     myGradient.addColorStop(1, "#636363");
     ctx.fillStyle = myGradient;
     ctx.fillRect(10,10,200,100);
     // 设置阴影
     ctx.shadowOffsetX = 10; // 设置水平位移
     ctx.shadowOffsetY = 10; // 设置垂直位移
     ctx.shadowBlur = 5; // 设置模糊度
     ctx.shadowColor = "rgba(0,0,0,0.5)"; // 设置阴影颜色
     ctx.fillStyle = "#CC0000";
     ctx.fillRect(10,10,200,100);
     */

    // 设置坐标原点
    ctx.translate(width / 2, height / 2);

    // 做一个旋转的太极图
    setInterval(function () {
        // 清空画布
        ctx.clearRect(-width / 2, height / 2, width / 2, -height / 2);
        // 旋转画布
        ctx.rotate(Math.PI / 20);
        // 绘制圆的轮廓
        ctx.beginPath();
        ctx.arc(0, 0, 50, 0, Math.PI * 2, true);
        ctx.lineWidth = 2.0;
        ctx.strokeStyle = "#000";
        ctx.stroke();
        // 画个黑圆
        ctx.beginPath();
        ctx.arc(0, 0, 50, 0, Math.PI, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        // 画个白圆
        ctx.beginPath();
        ctx.arc(0, 0, 50, Math.PI, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        // 根据上边画出来的，绘制一个太极图
        ctx.beginPath();
        ctx.arc(-25, 0, 25, 0, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(25, 0, 25, 0, Math.PI * 2, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(-25, 0, 8, 0, Math.PI * 2, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(25, 0, 8, 0, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
    }, 50);
</script>
</html>
```

如下为效果图：
<canvas id="myCanvas" width="400" height="400">
    您的浏览器不支持canvas！
</canvas>
<script type="text/javascript">
    var canvas = document.getElementById('myCanvas');
    var ctx = canvas.getContext('2d');
    var height = 400;
    var width = 400;

    // 设置坐标原点
    ctx.translate(width / 2, height / 2);

    // 做一个旋转的太极图
    setInterval(function () {
        // 清空画布
        ctx.clearRect(-width / 2, height / 2, width / 2, -height / 2);
        // 旋转画布
        ctx.rotate(Math.PI / 20);
        // 绘制圆的轮廓
        ctx.beginPath();
        ctx.arc(0, 0, 50, 0, Math.PI * 2, true);
        ctx.lineWidth = 2.0;
        ctx.strokeStyle = "#000";
        ctx.stroke();
        // 画个黑圆
        ctx.beginPath();
        ctx.arc(0, 0, 50, 0, Math.PI, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        // 画个白圆
        ctx.beginPath();
        ctx.arc(0, 0, 50, Math.PI, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        // 根据上边画出来的，绘制一个太极图
        ctx.beginPath();
        ctx.arc(-25, 0, 25, 0, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(25, 0, 25, 0, Math.PI * 2, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(-25, 0, 8, 0, Math.PI * 2, true);
        ctx.fillStyle = "#000000";
        ctx.fill();
        ctx.beginPath();
        ctx.arc(25, 0, 8, 0, Math.PI * 2, true);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
    }, 50);
</script>
