---
title: Flutter单屏启动动画介绍页面制作【附视频】
date: 2020-04-18 15:00:38
tags:
toc: true
---
Flutter单屏启动动画介绍页面制作【附视频】
本文为本人原创，

效果：![image](http://upload-images.jianshu.io/upload_images/14347887-cb96fed106fb127a.gif?imageMogr2/auto-orient/strip)
视频链接：https://www.bilibili.com/video/av46276578/?p=2
 这节课主要讲的是一个单屏的启动动画，其实很简单的，之前以为大家都会就没讲，然后有位小伙伴私聊我，说让我讲一下，因为很多软件用的都是单屏或者单屏下面还有跳过按钮倒计时数字啥的，这个大家随机应变应该会感觉很简单的，看完我的这些教程的朋友，

        那我就不说那么多了直接开始文字教程。

main等东西就不说了，home里面写了个SingleScreen()然后我们就创建文件之后导入了，SingleScreen是一个动态的widget类，我们在里面就写个充满屏幕的图片就行了，用的图片获取方式是network，

然后我们写了个初始化，里面有个倒计时，
```
void initState() { 
    super.initState();
    conutDown();
  }
```
然后倒计时里面我们写了个延时的东西，里面的参数是转到新页面的方法
```
void conutDown() {
    var _duration = Duration(seconds: 3);
    Future.delayed(_duration, newPage);
  }
```
之后新页面的方法写的就是给他替换路由名字为/newPage
```
void newPage() {
    Navigator.of(context).pushReplacementNamed('/NewPage');
  }
```
之后我们的main.dart的materialApp就接收一个新的路由并写东西，就写了给他跳转到新页面
```
routes: <String, WidgetBuilder> {
        '/NewPage' : (context) => NewPage()
      },
```
然后新页面就很简单了，就是我们的想跳转到的页面了，
```
Scaffold(
      appBar: AppBar(
        title: Text('单屏介绍'),
        centerTitle: true,
      ),
      body: Center(
        child: Text(
          '新页面',
          style: Theme.of(context).textTheme.display2,
        ),
      ),
    )
```
大概就是介个样子啦，那我们就来把源码呈上来了：
main.dart
```
import 'package:flutter/material.dart';
import 'single_screen.dart';
import 'new_page.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SingleScreen',
      theme: ThemeData(
        primaryColor: Colors.blue,
      ),
      home: SingleScreen(),
      routes: <String, WidgetBuilder> {
        '/NewPage' : (context) => NewPage()
      },
    );
  }
}
```
single_screen.dart
```
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';
import 'dart:async';

class SingleScreen extends StatefulWidget {
  @override
  _SingleScreenState createState() => _SingleScreenState();
}

class _SingleScreenState extends State<SingleScreen> {
  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.white,
      child: Image.network(
        'http://img.wxcha.com/file/201606/30/1978c43117.jpg',
        fit: BoxFit.cover,
      ),
    );
  }

  void initState() { 
    super.initState();
    conutDown();
  }

  void conutDown() {
    var _duration = Duration(seconds: 3);
    Future.delayed(_duration, newPage);
  }

  void newPage() {
    Navigator.of(context).pushReplacementNamed('/NewPage');
  }
}
```
new_page.dart
```
import 'package:flutter/material.dart';

class NewPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('单屏介绍'),
        centerTitle: true,
      ),
      body: Center(
        child: Text(
          '新页面',
          style: Theme.of(context).textTheme.display2,
        ),
      ),
    );
  }
}
```