---
title: 最简单轻量便捷路由管理方案NavRouter
date: 2020-04-18 15:31:38
tags:
toc: true
---
【本文待优化】

大家好，我是CrazyQ1，今天给大家推荐一个路由管理方案，用的非常不错的，叫nav_router。

项目地址是：[https://github.com/fluttercandies/nav_router](https://github.com/fluttercandies/nav_router)

这篇文章主要是来介绍这个东西的使用。

nav_router是flutter最简单/轻量/便捷的路由管理方案，支持各种路由动画，跳转/传参起来非常方便，跳转新页面只需：routePush(NewPage());

**开始使用**

**添加依赖**

```
dependencies:
  nav_router: any #具体版本自定义（any表示最新）
```

然后使用 flutter packages upgrade 来更新flutter的插件包

在项目的example里面有示例项目，可以直接去运行并参考代码。

下面来说一下相关配置和使用。

**配置**

1.在`MaterialApp`的页面先导入我们的插件
```
import 'package:nav_router/nav_router.dart';
```

2.在`MaterialApp`的`navigatorKey`属性写上`navGK`值
```
Widget build(BuildContext context) {
    return new MaterialApp(
      title: '',
      navigatorKey: navGK,
    );
  }
```

3.然后，我们就可以开始使用啦，下面是一个跳转页面的例子
```
Widget buildItem(RouteModel item) {
  return new FlatButton(
    onPressed: () => routePush(new NewPage()),
    child: new Text('点击跳转'),
  );
}
```

4.如果我们想用其他路由动画跳转可以在后面添加跳转属性,比如：渐变动画
```
routePush(new NewPage(), RouterType.fade);
```

然后我们来看看这个怎么方便的传递参数和接收并使用。

**传递参数**

这里主要讲两个方式来传输，具体的大家可以自己去研究下。

**方式1**

正常push新页面，但是在后面加上Then，后面的v就是我们跳转之后的页面带回来的数据，然后我们给它打印出来
```
routePush(NewPage()).then((v) {
  print('I received::$v');
});
```

那么我们新页面就要pop返回值了，直接在pop然后括号里加上我们的参数，可以是任何类型的参数值，之后上面写的东西就能接收到我们这次返回并带过去的参数了。
```
FlatButton(
  onPressed: () {
    pop('This is the parameter');
  },
  child: Text('Return with parameters'),
),
```
**方式2**

方式二可以用我们的NavData，先在我们要push到的页面添加NavData类型的参数接收,
```
class NewPage extends StatlessWidget {
  final NavData navData;

  NewPage({this.navData});
}
```

然后下面就判断这个navData是否为空，也就是上级是否有接收这个方法，如果有的话就带参数返回。
```
FlatButton(
  onPressed: () {
    if(navData == null) return;
    widget.navData('NavData mode parameter transmission');
    pop();
  },
  child: Text('Return with parameters'),
),
```

那么我们push的那个地方就可以用navData来接收值并且打印出来了。

```
routePush(NewPage(navData: (v) {
    print('I received::$v');
  }),
);
```

**示例效果图**

[![点击查看原图](https://upload-images.jianshu.io/upload_images/14347887-58a599a65489ca81.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.flutterj.com/content/uploadfile/201912/b5e91576074142.gif) 

具体可以去项目里面查看。

这里我推荐个FLutter学习群，分别有微信群和QQ群

![image](https://upload-images.jianshu.io/upload_images/14347887-4f430b9284039cf6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter教程网：[www.flutterj.com](http://www.flutterj.com/)

Flutter交流QQ群：[874592746](https://jq.qq.com/?_wv=1027&k=5coTYqE)

然后贴上我们的公众号“Flutter前线”

关注公众号“`Flutter前线`”，各种Flutter项目实战经验技巧，干活知识，Flutter面试题答案，等你来领取。

![image](https://upload-images.jianshu.io/upload_images/14347887-09a59c358aceebf1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
