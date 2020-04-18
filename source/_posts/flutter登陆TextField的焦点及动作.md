---
title: flutter登陆TextField的焦点及动作
date: 2020-04-18 15:18:23
tags:
toc: true
---
 视频链接：[**https://www.bilibili.com/video/av44936399/?p=2**](https://www.bilibili.com/video/av44936399/?p=2)

# 实现效果
在多个TextField中获取焦点，并通过键盘跳到下一个焦点的demo，主要讲解获取焦点，

# 第一个文件，main.dart代码
```
import 'package:flutter/material.dart';
import 'textfields_focus_demo.dart'; // 导包

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  final Widget child;

  MyApp({Key key, this.child}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      child: MaterialApp(
        title: 'Flutter demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          primaryColor: Colors.blue,
        ),
        home: TextFieldDemo(),
      ),
    );
  }
}
```
导包之后，定义username和password的控制器和焦点，

然后给他初始化，初始化完之后就开始调用了，


# 实现原理：
使用FocusNode获取当前textField焦点
在TextNode的textInputAction属性中选择键盘action（next/down）
对于最后一个之前的TextField：在onSubmitted属性中解除当前focus状态
再聚焦下一个FocusNode:FocusScope.of(context).requestFocus( nextFocusNode );
对于最后一个TextField,直接解除focus并提交表单

# 代码
```

	textfields_focus_demo.dart 
import 'package:flutter/material.dart';

class TextFieldDemo extends StatefulWidget {
  _TextFieldDemoState createState() => _TextFieldDemoState();
}

class _TextFieldDemoState extends State<TextFieldDemo> {
  FocusNode _namefocusNode, _pwfocusNode;
  TextEditingController _nameController, _pwController;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController();
    _pwController = TextEditingController();
    _namefocusNode = FocusNode();
    _pwfocusNode = FocusNode();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(//安全区，主要兼容屏幕
        child: ListView(
          children: <Widget>[
            SizedBox(height: 80.0),// 空出80高度
            Center(//居中
              child: Text(
                'Login',
                style: TextStyle(fontSize: 30.0),
              ),
            ),
            SizedBox(height: 80.0),
            Padding(
              padding: EdgeInsets.all(16.0),
              child: Material(
                borderRadius: BorderRadius.circular(10.0),
                child: TextField(
                  focusNode: _namefocusNode,
                  controller: _nameController,
                  obscureText: false,
                  textInputAction: TextInputAction.next,
                  onSubmitted: (input) {
                    _namefocusNode.unfocus();
                    FocusScope.of(context).requestFocus(_pwfocusNode);
                  },
                  decoration: InputDecoration(
                    labelText: 'name',
                  ),
                ),
              ),
            ),
            Padding(
              padding: EdgeInsets.all(16.0),
              child: Material(
                borderRadius: BorderRadius.circular(10.0),
                child: TextField(
                  focusNode: _pwfocusNode,
                  controller: _pwController,
                  obscureText: true,
                  textInputAction: TextInputAction.done,
                  onSubmitted: (input) {
                    _pwfocusNode.unfocus();
                  },
                  decoration: InputDecoration(
                    labelText: 'password'
                  ),
                ),
              ),
            ),
            ButtonBar(
              children: <Widget>[
                RaisedButton(
                  onPressed: (){},
                  child: Text('login'),
                ),
              ],
            )
          ],
        ),
      ),
    );
  }
}
```

# 效果图：
![image](http://upload-images.jianshu.io/upload_images/14347887-7e4ca6a91ec952e0?imageMogr2/auto-orient/strip)