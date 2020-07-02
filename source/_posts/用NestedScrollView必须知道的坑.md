---
title: 用NestedScrollView必须知道的坑
date: 2020-04-18 15:28:46
tags:
---
【本文待优化】

做企业项目遇到了个坑，

那这个坑是怎么遇到的呢，刚开始是已经做好了商品详情页：


详情页面用的是NestedScrollView组件，轮播图那一块用的是SliverAppBar，

也就是写在NestedScrollView的头部，然后下面的都是在身体部分了，

身体部分是可以滑动的，刚开始是没任何问题，正常滑动运行，

但是来了这个需求：


是在商品详情加个tabbar，然后我就加在SliverAppBar里面的bottom内个，

加上去显示也是没什么问题，但是锚点这个需求实现的时候就来了问题了。

大家都知道，想要锚点(jumpTo到指定位置)，嘚让他的body也加个控制器啊，

然后我就把之前给的滚动组件

new SingleChildScrollView(

  child: new Column(children: widget.widgets),

);

改成了

new ListView(children: widget.widgets);

虽然SingleChildScrollView也是可以加控制器并且jumpTo的，

但是我感觉用ListView比较舒服，代码也比较简洁，所以就用这个，

但是用哪个实现的效果都是差不多的。

然鹅

惊人的一幕就出现了。


NestedScrollView的头部内容完全固定，滑动body部分是不能控制到头部的，

但是滑动头部就是可以控制头部，

也就是头部和身体部分 分开了。

这是为什么呢？

因为NestedScrollView是有内外两个控制器的:

out控制header，inner控制body。只有当out不能滚动了才会滚动inner

body不写控制器就没事,写了就出现这种情况，

而且我去测试了下打印控制器最大滚动位置发现只有300左右，

也就是只能打印出头部的，

print(_C.position.maxScrollExtent);

那我要怎么去实现这个功能啊，只能在轮播图内跳来跳去，

难道是贫穷限制了我的想象吗？

头部固定解决方案：（不是唯一的）

既然都说了是有内外两个控制器那我们一定有办法来获取并使用他的内部控制器，

第一步：（尝试封装body为有状态类来从context中取到内控制器）
```dart
@override

  Widget build(BuildContext context) {

    return new Scaffold(
    
      body: new NestedScrollView(
    
          controller: _ctl,
    
          headerSliverBuilder: _sliverBuilder,
    
          body: new BodyView(widget.widgets, type)),
    
    );

  }
```
BodyView就是我们封装的，

class BodyView extends StatefulWidget {

  final List<Widget> widgets;

  final int type;

  BodyView(this.widgets, this.type);

  @override

  _BodyViewState createState() => _BodyViewState();

}

class _BodyViewState extends State<BodyView> {

  @override

  Widget build(BuildContext context) {

    return new SingleChildScrollView(
    
      child: new Column(children: widget.widgets),
    
    );

  }

}

第二步：（type是干啥的先不用管）

class BodyView extends StatefulWidget {

  ...

}

class _BodyViewState extends State<BodyView> {

  Type typeOf<T>() => T;

  ScrollController _innerC;

  @override

  void initState() {

    super.initState();
    
    PrimaryScrollController primaryScrollController =
    
        context.ancestorWidgetOfExactType(typeOf<PrimaryScrollController>());
    
    _innerC = primaryScrollController.controller;

  }

  @override

  Widget build(BuildContext context) {

    ...

  }

}

我们定义了一个类型和控制器，然后再初始化的时候写了一个主控制器，

主控制器的值是从上下文的父类取的类型，然后typeOf的泛型就是我们写的

主控制器，那么内控制器就是等于我们取到的这个控制器，

头部固定问题就完美解决了

只要能取到，就算不用也是可以的

当然也可以直接使用:

@override

Widget build(BuildContext context) {

  _actions(widget.type);

  return PrimaryScrollController(controller: _innerC, child: new SingleChildScrollView(

    child: new Column(children: widget.widgets),

  ));

}

这个都无所谓的。

但是我们发现两个控制器开始分开的，打印外控制器最大滚动还是300左右，

但是打印内控制器最大滚动位置是body的全部，2k左右，

那么我这个需求还有没有解决方案了？

当然是有的：

点击锚点跳转解决方案

# 第一步（直接使用外部控制器jumpTo）

@override

void initState() {

  super.initState();

  tabs = ['商品', '评价', '详情'];

  _tabC = new TabController(length: tabs.length, vsync: this);

  _tabC.addListener(() => _onTabChanged());

}

_onTabChanged() {

  setState(() {

    switch (_tabC.index) {
    
      case 0:
    
        _ctl.jumpTo(0.1);
    
        type = 0;
    
        break;
    
      case 1:
    
        type = 1;
    
        break;
    
      case 2:
    
        type = 2;
    
        break;
    
    }

  });

}

_tabC就是外部控制器，在初始化的时候监听tabbar是否被点击，

如果被点击的话直接写个tab改变的方法，tabbar的三个Bar分别是0，1，2，

所以我们也接收一个0，1，2，来处理，

然后直接给它jumpTo跳转，然后那个type就是我们的BodyView接收的

具体有什么用呢？

class BodyView extends StatefulWidget {

...

}

class _BodyViewState extends State<BodyView> {

  ...

  _actions(int type) {

    setState(() {
    
      _binding.addPostFrameCallback((callback) {
    
        switch (type) {
    
          case 1:
    
            _innerC.jumpTo(1000);
    
            print(_innerC.position.maxScrollExtent);
    
            break;
    
          case 2:
    
            _innerC.jumpTo(2000);
    
            break;
    
        }
    
      });
    
    });

  }

  @override

  void initState() {

    ...

  }

  @override

  Widget build(BuildContext context) {

    _actions(widget.type);
    
    ...

  }

}

我们可以看到，这边也是监听接收的int类型，

如果监听到传过来的是0的话就调到我们的顶部，（heard控制器控制）

如果监听到传过来的是1的话就调到我们想要到的评论的位置。

如果监听到传过来的是2的话就跳到我们想要的商品详情的位置。

Position为null的解决方案

当我以为这样就没问题的时候发现又出现了一个错误，


真的是坑一个接着一个啊，

解决方案为：

调用第一帧绘制完毕之后再执行jumpTo

具体：

class BodyView extends StatefulWidget {

    ...

}

class _BodyViewState extends State<BodyView> {

  WidgetsBinding _binding = WidgetsBinding.instance;

  _actions(int type) {

    setState(() {
    
      _binding.addPostFrameCallback((callback) {
    
        ...
    
    });

  }

}

我们写了一个小部件绑定的东西，让他能监听第一帧是否绘制完毕，

绘制完毕之后再执行jumpTo

这样就只差获取评论和商品详情的组件位置然后传入具体的Offset就完美执行了，

因为时间关系就到这了，任何问题可以加我微信：zonggeyl_com来问我。

接下来我把我这个文件的整体代码发出来，能看的懂的可以看一下，

直接运行肯定是不能运行的，因为里面调用的资源文件和封装你们都没有，

要懂查看和使用，

import 'package:flutter/material.dart';

class SliverAppBarPage extends StatefulWidget {

  SliverAppBarPage({

    this.widgets,
    
    this.headerView,
    
    this.height = 200,
    
    this.background,

  });

  final List<Widget> widgets;

  final Widget headerView;

  final Widget background;

  final double height;

  @override

  State<StatefulWidget> createState() => new SliverAppBarPageState();

}

class SliverAppBarPageState extends State<SliverAppBarPage>

    with TickerProviderStateMixin {

  TabController _tabC;

  ScrollController _ctl = new ScrollController();

  int type;

  List tabs;

  WidgetsBinding _binding = WidgetsBinding.instance;

  @override

  void initState() {

    super.initState();
    
    tabs = ['商品', '评价', '详情'];
    
    _tabC = new TabController(length: tabs.length, vsync: this);
    
    _tabC.addListener(() => _onTabChanged());

  }

  _onTabChanged() {

    setState(() {
    
      switch (_tabC.index) {
    
        case 0:
    
          _binding.addPostFrameCallback((callback) => _ctl.jumpTo(0.1));
    
          type = 0;
    
          break;
    
        case 1:
    
          type = 1;
    
          break;
    
        case 2:
    
          type = 2;
    
          break;
    
      }
    
    });

  }

  List<Widget> _sliverBuilder(BuildContext context, bool innerBoxIsScrolled) {

    return <Widget>[
    
      new SliverAppBar(
    
        centerTitle: true,
    
        expandedHeight: widget.height,
    
        floating: false,
    
        pinned: true,
    
        backgroundColor: Colors.white,
    
        elevation: 0,
    
        brightness: Brightness.light,
    
        leading: new InkWell(
    
          child: innerBoxIsScrolled
    
              ? new Container(
    
                  width: 15,
    
                  height: 20.0,
    
                  child: new Image.asset('assets/images/nav_ic_back.webp',
    
                      color: innerBoxIsScrolled ? mainFontColor : Colors.white),
    
                )
    
              : new Container(
    
                  padding: EdgeInsets.only(left: 10.0),
    
                  alignment: Alignment.center,
    
                  child: new Container(
    
                    height: 35,
    
                    width: 35,
    
                    decoration: BoxDecoration(
    
                        color: Color.fromRGBO(0, 0, 0, 0.2),
    
                        borderRadius: BorderRadius.circular(17.5)),
    
                    child: new Image.asset('assets/images/nav_ic_back.webp',
    
                        color:
    
                            innerBoxIsScrolled ? mainFontColor : Colors.white),
    
                  ),
    
                ),
    
          onTap: () => Navigator.pop(context),
    
        ),
    
        title: new Text(
    
          innerBoxIsScrolled ? '商品详情' : '',
    
          style: TextStyle(color: Color(0xff000000), fontSize: 19.0),
    
        ),
    
        bottom: innerBoxIsScrolled
    
            ? new PreferredSize(
    
                child: new Container(
    
                  padding: EdgeInsets.symmetric(horizontal: 80.0),
    
                  child: new TabBar(
    
                      controller: _tabC,
    
                      indicatorSize: TabBarIndicatorSize.label,
    
                      labelColor: Color(0xffFF4F73),
    
                      indicatorColor: Color(0xffFF4F73),
    
                      unselectedLabelColor: Color(0xff000000),
    
                      labelStyle: new TextStyle(fontSize: 14.0),
    
                      labelPadding: EdgeInsets.only(bottom: 20),
    
                      indicatorPadding: EdgeInsets.only(
    
                          bottom: 15, top: 10, left: 5, right: 5.0),
    
                      tabs: tabs.map((item) => new Text('$item')).toList()),
    
                ),
    
                preferredSize: Size(30, 50))
    
            : null,
    
        actions: <Widget>[],
    
        flexibleSpace: new FlexibleSpaceBar(
    
            centerTitle: true,
    
            title: widget.headerView,
    
            background: widget.background),
    
      ),
    
    ];

  }

  @override

  Widget build(BuildContext context) {

    return new Scaffold(
    
      body: new NestedScrollView(
    
          controller: _ctl,
    
          headerSliverBuilder: _sliverBuilder,

//        body: new SingleChildScrollView(

//          controller: _ctl,

//            child: new Column(children: widget.widgets)),

//      ),

          body: new BodyView(widget.widgets, type)),
    
    );

  }

}

class BodyView extends StatefulWidget {

  final List<Widget> widgets;

  final int type;

  BodyView(this.widgets, this.type);

  @override

  _BodyViewState createState() => _BodyViewState();

}

class _BodyViewState extends State<BodyView> {

  Type typeOf<T>() => T;

  ScrollController _innerC;

  WidgetsBinding _binding = WidgetsBinding.instance;

  _actions(int type) {

    setState(() {
    
      _binding.addPostFrameCallback((callback) {
    
        switch (type) {
    
          case 1:
    
            _innerC.jumpTo(1000);
    
            print(_innerC.position.maxScrollExtent);
    
            break;
    
          case 2:
    
            _innerC.jumpTo(2000);
    
            break;
    
        }
    
      });
    
    });

  }

  @override

  void initState() {

    super.initState();
    
    PrimaryScrollController primaryScrollController =
    
        context.ancestorWidgetOfExactType(typeOf<PrimaryScrollController>());
    
    _innerC = primaryScrollController.controller;

  }

  @override

  Widget build(BuildContext context) {

    _actions(widget.type);
    
    return new SingleChildScrollView(
    
      child: new Column(children: widget.widgets),
    
    );

  }

}



关注公众号“Flutter前线”，各种Flutter项目实战经验技巧，干活知识，Flutter面试题答案，等你来领取。