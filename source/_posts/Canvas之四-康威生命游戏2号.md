title: Canvas之四 康威生命游戏2号
date: 2014-11-27 01:38:09
categories: blog
tags: 
- 前端
- canvas
- 算法
---
本文可以自己创建一个符合康威游戏规则的小世界，怎么发挥都随便你，看着这个这些虚拟小生命的繁衍，体验做上帝的感觉。
<link rel="stylesheet" type="text/css" href="/css/canvas4.css">
<div style="display:inline-block;">第<span id="generation" style="width: 100px;"></span>代<button onclick="reset();">重置</button><button onclick="start();">开始</button><input id="speed" placeholder="100 ms/g" style="width: 80px"><button onclick="stop();">暂停</button><button onclick="random();">随机</button><button onclick="nextGeneration();">下一代</button></div><hr><div><div class="left"><table id="diyTable"></table></div><canvas id="myCanvas" width="440" height="440">您的浏览器不支持HTML5</canvas></div>
<script>
    var ifInit = false;
    var canvas = document.getElementById("myCanvas");
    var height = canvas.height,
        width = canvas.width,
        context = canvas.getContext("2d");
    var tx = 40,
        ty = 40;    // 记录长宽有多少个方块
    var cellWidth = height / ty,
        cellHeight = width / tx;    // 小细胞的长宽
    var lastData = [];  // 记录上一代的数据
    var data = [];     // (height+2) * (width + 2)
    var generation;
    var born = 3,   // 当周围有三个活着的时候，这个位置会活一个
        keep = 2;   // 当周围有两个的时候，这个位置不变，其他情况都死
    var running; // 是否运行
    var intervalTime = 100;   // 每次迭代的时间间隔
    var generationText = document.getElementById("generation");
    var diyTable = document.getElementById("diyTable");


    // 初始化数据和canvas
    function init() {
        generation = 0;
        running = false;
        intervalTime = parseInt(document.getElementById("speed").value);
        lastData = buildTwoDArray(ty + 2, tx + 2);
        data = buildTwoDArray(ty + 2, tx + 2);
        context.clearRect(0, 0, width, height);
        generationText.innerHTML = generation;
        ifInit ? 1 : initTable();
        clearTable();
    }

    // 随机生成数据
    function random() {
    	init();
        lastData = buildTwoDArray(ty + 2, tx + 2);
        data = buildTwoDArray(ty + 2, tx + 2);
        // 生成画面的内容，即data的数据
        for (var i = 0; i < ty; i++) {
            for (var j = 0; j < tx; j++) {
                if (Math.random() > 0.8) {
                    data[i + 1][j + 1] = 1;
                }
            }
        }
        // 随机生成后要画出来
        paintTable();
        paint();
    }

    // 初始化选择变革
    function initTable() {
        // 绘制表格
        var html = [];
        for (var i = 0; i < ty; i++) {
            html.push("<tr>");
            for (var j = 0; j < tx; j++) {
                html.push("<td onclick='selectCell(this, " + i + "," + j + ");'></td>")
            }
            html.push("</tr>");
        }
        diyTable.innerHTML = html.join("");
        ifInit = true;
    }

    function selectCell(cell, x, y) {
        if (data[x + 1][y + 1]) {
            // 取消选择
            cell.style.backgroundColor = "white";
            data[x + 1][y + 1] = 0;
        } else {
            // 选择
            cell.style.backgroundColor = "blue";
            data[x + 1][y + 1] = 1;
        }
    }

    // 清空选择表格
    function clearTable() {
        // 恢复所有单元格的颜色
        for (var i = 0; i < ty; i++) {
            for (var j = 0; j < tx; j++) {
                diyTable.rows[i].cells[j].style.backgroundColor = "white";
            }
        }
        // 恢复数据
        lastData = buildTwoDArray(ty + 2, tx + 2);
        data = buildTwoDArray(ty + 2, tx + 2);
    }

    // 填充选择表格
    function paintTable() {
        for (var i = 0; i < ty; i++) {
            for (var j = 0; j < tx; j++) {
                if (data[i + 1][j + 1]) {
                    diyTable.rows[i].cells[j].style.backgroundColor = "blue";
                } else {
                    diyTable.rows[i].cells[j].style.backgroundColor = "white";
                }
            }
        }
    }

    function drawCell(y, x) {
        var cx = x * cellWidth;
        var cy = y * cellHeight;
        context.fillStyle = "Gold";
        context.fillRect(cx, cy, cellWidth, cellHeight);
        context.strokeStyle = "DarkGoldenRod";
        context.strokeRect(cx + 1, cy + 1, cellWidth - 2, cellHeight - 2);
    }

    function killCell(y, x) {
        var cx = x * cellWidth;
        var cy = y * cellHeight;
        context.clearRect(cx, cy, cellWidth, cellWidth);
    }

    // 将数据画到canvas上
    function paint() {
        for (var i = 0; i < ty; i++) {
            for (var j = 0; j < tx; j++) {
                if (data[i + 1][j + 1] && !lastData[i + 1][j + 1]) {
                    // 有生命
                    drawCell(i, j);
                } else if (!data[i + 1][j + 1] && lastData[i + 1][j + 1]) {
                    // 没有生命
                    killCell(i, j);
                }
            }
        }
        generationText.innerHTML = generation;
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
        data = buildTwoDArray(ty + 2, tx + 2);
        // 重新生成新一代的数据
        for (var i = 1; i <= ty; i++) {
            for (var j = 1; j <= tx; j++) {
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
        intervalTime = parseInt(document.getElementById("speed").value, 10);
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
