---
date: 2020-11-02T21:31:05.000Z
layout: post
title: Sumo仿真软件的使用
description: >-
  写一篇初步如何跑起Sumo软件的笔记.
image: >-
  https://PurpleT-shirt.github.io/images/traffic.jpg
optimized_image: >-
  https://PurpleT-shirt.github.io/images/traffic.jpg
category: tutorial
tags:
  - traffic
  - tutorial
author: wangwei
---

# Sumo仿真软件的使用

## 零：提前了解

​	Sumo即simulation of urban mobility，是由德国航空航天中心交通运输研究所研发的，具有微观、连续的道路交通仿真架构和模型基础，是一个开源的仿真软件，非常适合于道路交通仿真的研究人员使用。

​	Sumo是一个微观的，空间上连续，时间上离散的交通仿真软件，采用c++语言开发，其宏观特征包括带变道的**多车道**道路，基于道路交叉口的**靠右侧行驶**规则，支持动态路由，可以管理超过10000条街道的网络。其微观特征包括允许碰撞自由的车辆移动模式，支持单车路由。该软件特点是具有快速的OpenGL图形界面，支持多种网络格式输入，缺点是sumo本身不能提供网络仿真器所需要的轨迹文件。

### 0.1工程结构

> ../Sumo
>
> ../map/net.xml（路网文件）
>
> ../map/rou.xml（车流文件）

## 一、在openstreetmap下载相关地图文件

​	openstreetmap，开源的地图服务。

​	获取地图的方式，openstreetmap.org左边栏中选择“导出”、“手动选择不同的区域”、“导出”，获得map.osm文件。存放地址为sumo安装目录的同级map目录。

## 二、对导出的osm地图做处理

​	导出的osm地图不仅包含路网信息也包含大量的别的模块例如建筑和河流，需要做简单的处理。

​	打开..\Sumo\doc\userdoc\Networks\Import\OpenStreetMap.html，截取Importing additional Polygons (Buildings, Water, etc.)中的xml的代码，删除id=“power”的那一行（用不到），粘贴到名为typemap.xml的文本中，保存到map目录中去，和osm地图文件放一起。

​	打开../Sumo/bin/start-command-line.bat，然后用命令行模式对osm文件做处理。到map文件夹下进行操作，输入以下指令

```sh
netconvert --osm-files map.osm -o map.net.xml
```

```shell
polyconvert --net-file map.net.xml --osm-files map.osm --type-file typemap.xml -o map.poly.xml
```

​	完成后就产生了map.net.xml和map.poly.xml文件（路网以外的信息，住宅、水域等）。

## 三、产生map.rou.xml文件

​	输入命令(-n map.net.xml表示输入，-n表述输入的类型是net类型。后面的-e 100 -l 是随机工具的配置)：

```shell
python D:/SoftWare/Sumo/tools/randomTrips.py -n map.net.xml -e 1000 -l
```

​	生成一个旅程随机过程文件trips.trips.xml，把随机的旅程和道路信息结合起来就获得了车流文件（rou.xml）了。要用到的工具是bin文件夹下的duarouter.exe。输入命令：

```shell
python D:/SoftWare/Sumo/tools/randomTrips.py -n map.net.xml -r map.rou.xml -e 100 -l
```

​	执行成功后可以在map目录下查看到map.rou.xml。

> ​	随机数100，指的是车辆数目，可以适当取更大的数取代。

## 四、编辑配置文件

​	在Sumo目录中搜索test.sumocfg，随机拷贝一份，更改input标签，命名为map.sumo.cfg并保存到map目录中。

```xml
<configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://sumo.dlr.de/xsd/sumoConfiguration.xsd">
    <input>
        <net-file value="map.net.xml"/>
        <route-files value="map.rou.xml"/>
        <additional-files value="map.poly.xml"/>
    </input>
	<time>
		<begin value = "0"/>
		<end value = "1000"/>
	</time>
</configuration>
```

## 五、运行map仿真例子

​	输入指令：

```shell
sumo-gui map.sumo.cfg
```

​	设置延时，可以设置在100到200之间，点后点击开始按钮，可以看到随机运动的车辆。