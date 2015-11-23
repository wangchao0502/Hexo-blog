title: Canvas之三 康威生命游戏1号
date: 2014-11-27 01:36:14
categories: blog
tags: 
- 前端
- canvas
- 算法
---

本文实现了一个随机小世界的繁衍过程，可以控制迭代速度，纯原生js实现

<div style="display:inline-block;">第<span id="generation" style="width: 100px;"></span>代<button onclick="reset();">重置</button><button onclick="start();">开始</button><input id="speed" placeholder="100 ms/g" style="width: 80px"><button onclick="stop();">暂停</button><button onclick="nextGeneration();">下一代</button></div><hr><div><canvas id="myCanvas" width="400" height="400">您的浏览器不支持HTML5</canvas></div>
<script>
    var canvas = document.getElementById("myCanvas");
    var height = canvas.height,
        width = canvas.width,
        context = canvas.getContext("2d");
    var lastData = [];  // 记录上一代的数据
    var data = [];     // (height+2) * (width + 2)
    var imageData;
    var generation;
    var born = 3,   // 当周围有三个活着的时候，这个位置会活一个
        keep = 2;   // 当周围有两个的时候，这个位置不变，其他情况都死
    var running; // 是否运行
    var intervalTime = 100;   // 每次迭代的时间间隔


    // 初始化数据和canvas
    function init() {
        generation = 0;
        running = false;
        intervalTime = parseInt(document.getElementById("speed").value);
        lastData = buildTwoDArray(height + 2, width + 2);
        data = buildTwoDArray(height + 2, width + 2);
        context.clearRect(0, 0, width, height);
        imageData = context.getImageData(0, 0, width, height);
        document.getElementById("generation").innerHTML = generation.toString();
        random();
    }

    // 随机生成数据
    function random() {
        // 生成画面的内容，即data的数据
        for (var i = 0; i < height; i++) {
            for (var j = 0; j < width; j++) {
                if (Math.random() > 0.8) {
                    data[i + 1][j + 1] = 1;
                }
            }
        }
        // 随机生成后要画出来
        paint();
    }

    // 将数据花到canvas上
    function paint() {
        var d = imageData.data;
        // 生成绘制到canvas上的，下面代码的操作对象是图像流（数组）
        for (var i = 0; i < height; i++) {
            for (var j = 0; j < width; j++) {
                if (data[i + 1][j + 1] && !lastData[i + 1][j + 1]) {
                    var pos = (i * width + j) * 4;
                    // 有生命，变成红色
                    d[pos] = 255; // red
                    d[pos + 1] = 0; // green
                    d[pos + 2] = 0; // blue
                    d[pos + 3] = 255;   // 阿尔法通道，这个不设置会变成全透明
                } else if (!data[i + 1][j + 1] && lastData[i + 1][j + 1]) {
                    var pos = (i * width + j) * 4;
                    // 有生命，变成红色
                    d[pos] = 255; // red
                    d[pos + 1] = 255; // green
                    d[pos + 2] = 255; // blue
                }
            }
        }
        context.putImageData(imageData, 0, 0);
        document.getElementById("generation").innerHTML = generation.toString();
    }

    // 生成二维数组，所有初始化为0
    function buildTwoDArray(x, y) {
        var arr = [];
        for (var i = 0; i < x; i++) {
            arr[i] = [];
            for (var j = 0; j < y; j++) {
                arr[i][j] = 0;
            }
        }
        return arr;
    }

    // 进行这一代的计算，并将上一代的数据保存到lastData中
    function generate() {
        lastData = data;
        data = buildTwoDArray(height + 2, width + 2);
        // 重新生成新一代的数据
        for (var i = 1; i <= height; i++) {
            for (var j = 1; j <= width; j++) {
                // 周围有几个或者的
                var state = lastData[i - 1][j - 1] + lastData[i - 1][j] + lastData[i - 1][j + 1] +
                        lastData[i][j - 1] + lastData[i][j + 1] +
                        lastData[i + 1][j - 1] + lastData[i + 1][j] + lastData[i + 1][j + 1];
                if (state == born) {
                    data[i][j] = 1;
                } else if (state == keep) {
                    data[i][j] = lastData[i][j];
                } else {
                    data[i][j] = 0;
                }
            }
        }
        generation++;
    }

    function start() {
        intervalTime = parseInt(document.getElementById("speed").value);
        function run() {
            if (running) {
                setTimeout(function () {
                    paint();
                    generate();
                    run();
                }, intervalTime);
            }
        }
        running = true;
        run();
    }

    function stop() {
        running = false;
    }

    function reset() {
        init();
    }

    function nextGeneration() {
        paint();
        generate();
    }
    window.addEventListener("load", init, true);
</script>
