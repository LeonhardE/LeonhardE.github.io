---
layout: post
title: Highcharts
categories: Web开发
description: 使用Highcharts绘制统计图表
keywords: 前端 交互 Highcharts
---

# 使用Highcharts绘制统计图表

​	本博客将介绍如何在基于 Python Flask 框架的 Web 应用中借助 Highcharts 绘制统计图表。绘制图表是 Web 开发中的一项基本需求，许多涉及到大量数据的项目都需要通过绘制图表将数据可视化。在这里我将介绍一个基于 JavaScript 的图表库 Highcharts，以及如何在你的 Flask 应用中使用 Highcharts 绘图。

## Highcharts

​	Highcharts 是一个用 JavaScript 编写的图表库，可以方便快捷地添加到 Web 应用中并绘制出有交互性的图表。Highcharts 库支持的图表类型包括直线图、曲线图、区域图、柱状图、饼状图、散状点图、仪表图、气泡图、瀑布流图等 20 余种，可谓功能强大。

​	要想在页面中引入 Highcharts，首先需要添加 highcharts.js 文件，对应的代码如下

```html
<script src="http://cdn.highcharts.com.cn/highcharts/highcharts.js"></script>
```

创建图表可以通过 Highcharts 的初始化函数 ```Highcharts.chart``` 来实现，下面给出一段最简单的绘图示例 (来源 [Highcharts中文教程](https://www.highcharts.com.cn/docs/start-helloworld))

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>第一个 Highcharts 图表</title>
</head>
<body>
    <!-- 图表容器 DOM -->
    <div id="container" style="width: 600px;height:400px;"></div>
    <!-- 引入 highcharts.js -->
    <script src="http://cdn.highcharts.com.cn/highcharts/highcharts.js"></script>
    <script>
        // 图表配置
        var options = {
            chart: {
                type: 'bar'                          //指定图表的类型，默认是折线图（line）
            },
            title: {
                text: '我的第一个图表'                 // 标题
            },
            xAxis: {
                categories: ['苹果', '香蕉', '橙子']   // x 轴分类
            },
            yAxis: {
                title: {
                    text: '吃水果个数'                // y 轴标题
                }
            },
            series: [{                              // 数据列
                name: '小明',                        // 数据列名
                data: [1, 0, 4]                     // 数据
            }, {
                name: '小红',
                data: [5, 7, 3]
            }]
        };
        // 图表初始化函数
        var chart = Highcharts.chart('container', options);
    </script>
</body>
</html>
```

![屏幕快照 2019-06-22 下午7.58.37](https://LeonhardE.github.io/images/屏幕快照 2019-06-22 下午7.58.37.png)

除了条形图，Highcharts 还可以绘制曲线图