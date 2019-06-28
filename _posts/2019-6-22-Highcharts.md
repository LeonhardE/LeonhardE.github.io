---
layout: post
title: 使用Highcharts绘制统计图表
categories: Web开发
description: 使用Highcharts绘制统计图表
keywords: 前端 交互 Highcharts
---

# 使用 Highcharts 绘制统计图表

本博客将介绍如何在基于 Python Flask 框架的 Web 应用中借助 Highcharts 绘制统计图表。绘制图表是 Web 开发中的一项基本需求，许多涉及到大量数据的项目都需要通过绘制图表将数据可视化。在这里我将介绍一个基于 JavaScript 的图表库 Highcharts，以及如何在你的 Flask 应用中使用 Highcharts 绘图。

## Highcharts 简介

Highcharts 是一个用 JavaScript 编写的图表库，可以方便快捷地添加到 Web 应用中并绘制出有交互性的图表。Highcharts 库支持的图表类型包括直线图、曲线图、区域图、柱状图、饼状图、散状点图、仪表图、气泡图、瀑布流图等 20 余种，可谓功能强大。

要想在页面中引入 Highcharts，首先需要添加 highcharts.js 文件，对应的代码如下

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

除了条形图，Highcharts 还可以绘制折线图，对上述代码稍作修改如下

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
                type: 'line'                          
            },
            title: {
                text: '我的第二个图表'                
            },
            xAxis: {
                categories: ['2019-01-01', '2019-01-02', '2019-01-03', '2019-01-04', 
                             '2019-01-05', '2019-01-06', '2019-01-07', '2019-01-08']   // x 轴分类
            },
            yAxis: {
                title: {
                    text: '打代码行数'                // y 轴标题
                }
            },
            series: [{                              // 数据列
                name: '小明',                        // 数据列名
                data: [100, 20, 45, 64, 23, 0, 90, 123]                     // 数据
            }, {
                name: '小红',
                data: [35, 71, 32, 60, 92, 13, 20, 102]
            }]
        };
        // 图表初始化函数
        var chart = Highcharts.chart('container', options);
    </script>
</body>
</html>
```

![屏幕快照 2019-06-22 下午8.20.59](https://LeonhardE.github.io/images/屏幕快照 2019-06-22 下午8.24.55.png)

除了这两种，Highcharts 还可以绘制许多种不同类型的统计图表，在此不一一列举。

## 在 Flask 应用中添加 Highcharts 图表

本节中我将以我们小组开发的后台管理系统作为例子，讲述如何在 Flask 应用中添加Highcharts 图表。

我们的后台管理系统 Python 代码基本结构分为 Model (数据库模型)、Service (服务)与 Web (网页功能)。项目的基本目录如下：

![屏幕快照2019-06-28下午8.10.34](https://LeonhardE.github.io/images/小欣餐饮png/屏幕快照2019-06-28下午8.10.34.png)

我们的目的是，利用 Highcharts 绘出订单量统计图与日营收统计图。首先展示图表添加完成后的效果：

![屏幕快照2019-06-25下午5.05.46](https://LeonhardE.github.io/images/小欣餐饮png/后台ui/屏幕快照2019-06-25下午5.05.46.png)

图表会显示所选日期之间的日常营收情况或订单情况。想要完成这个功能，首先需要在Service 中完成为图表提供数据的函数。该函数的传入参数为餐厅id、开始时间与结束时间，返回值为可供 Highcharts 绘图的列表。该函数位于 Service 文件夹内，函数内容如下所示：

```Python
class StatDailySite:

    # get a list of date str that is between begin_date and end_date
    @staticmethod
    def getEveryDay(begin_date, end_date):
        date_list = []
        begin_date = datetime.datetime.strptime(begin_date, "%Y-%m-%d")
        end_date = datetime.datetime.strptime(end_date, "%Y-%m-%d")
        while begin_date <= end_date:
            date_str = begin_date.strftime("%Y-%m-%d")
            date_list.append([date_str, 0, 0])
            begin_date += datetime.timedelta(days=1)
        return date_list

    # get a list of income info in the restaurant with restaurant_id that is between date_from and date_to
    @staticmethod
    def getdailyincome(restaurant_id, date_from, date_to):
        orderlist = db.session.query(Order).filter(Order.rid == restaurant_id,
                                                    func.date(Order.pay_time) >= datetime.datetime.strptime(date_from, "%Y-%m-%d") + datetime.timedelta(days=-1),
                                                    func.date(Order.pay_time) <= datetime.datetime.strptime(date_to, "%Y-%m-%d")).all()
        list = StatDailySite.getEveryDay(date_from, date_to)
        for order in orderlist:
            for item in list:
                if order.pay_time.strftime("%Y-%m-%d") == item[0]:
                    item[1] = item[1] + 1
                    item[2] += order.pay_price
        return list
```

该部分代码返回的列表长度为开始时间与结束时间之差，列表元素是一个长度为 3 的列表，第一个元素是日期，第二个元素是当天的订单数，第三个元素是当天的营收。

在完成了提供数据服务的函数后，需要构建一个请求数据的 url，该部分代码位于 Web文件夹下，文件名为 "chart.py"。该文件存在的目的是完成一个请求数据的 url，以供 JavaScript 程序请求数据。该部分分为两个部分，一个返回的是绘制订单统计图的数据，一个返回的是绘制营收统计图的数据，两个 url 分别为 /chart/dashboard 与 /chart/finance。该部分需要用到之前完成的 Service 函数，源码如下：

```Python
# -*- coding: utf-8 -*-
import datetime
from flask import Blueprint
from flask import jsonify

from ..libs.web_help import getFormatDate
from ..Service.Statistics import StatDailySite

route_chart = Blueprint('chart_page', __name__, url_prefix="/chart")


@route_chart.route("/dashboard")
# provide data for chart of order count
def dashboard():
    now = datetime.datetime.now()
    date_before_7days = now + datetime.timedelta(days=-7)
    date_from = getFormatDate(date=date_before_7days, format="%Y-%m-%d")
    date_to = getFormatDate(date=now, format="%Y-%m-%d")

    list = StatDailySite.getdailyincome(1, date_from, date_to)

    resp = {'code': 200, 'msg': '操作成功~~', 'data': {}}
    data = {
        "categories": [],
        "series": [
            {
                "name": "订单总数",
                "data": []
            }
        ]
    }

    if list:
        for item in list:
            data['categories'].append(item[0])
            data['series'][0]['data'].append(item[1])

    resp['data'] = data
    return jsonify(resp)


@route_chart.route("/finance")
# provide data for chart of daily income
def finance():
    now = datetime.datetime.now()
    date_before_7days = now + datetime.timedelta(days=-7)
    date_from = getFormatDate(date=date_before_7days, format="%Y-%m-%d")
    date_to = getFormatDate(date=now, format="%Y-%m-%d")

    list = StatDailySite.getdailyincome(1, date_from, date_to)

    resp = {'code': 200, 'msg': '操作成功~~', 'data': {}}
    data = {
        "categories": [],
        "series":[
            {
                "name": "日营收报表",
                "data": []
            }
        ]
    }

    if list:
        for item in list:
            data['categories'].append(item[0])
            data['series'][0]['data'].append(float(item[2]))

    resp['data'] = data
    return jsonify(resp)

```

该部分代码将数据打包完毕，可以直接供 Highcharts 插件使用。下面需要做的就是在 HTML 文件中添加供 Highcharts 图表显示的区域：

```html
<div class="col-lg-12" id="container" style="height: 400px;" data-highcharts-chart="0">
            图标使用Highchart
</div>
```

接下来需要做的就是编写相应 JavaScript 代码，调用之前编写的函数，绘制 Highcharts 统计图，在这里我给出了绘制日常营收统计图的 JavaScript 源码，绘制订单统计图与此类似，更改调用的函数即可，就不再给出了：

```javascript
drawChart:function(){
    charts_ops.setOption();
    $.ajax({
        url:common_ops.buildUrl("/chart/finance"),
        dataType:'json',
        success:function( res ){
            charts_ops.drawLine( $('#container'),res.data )
        }
    });
}
```

可以看到这段代码是向 /chart/finance 请求数据的，然后将请求回来的数据调用 Highcharts 绘图函数即可，简单清晰，绘图效果也很美观。

至此，在 Flask 应用中使用 Highcharts 绘图的技巧就分享完毕了，希望能对大家今后的开发有所帮助。