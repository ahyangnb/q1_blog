---
title: flutter实现文件读写篇
date: 2020-04-18 15:25:05
tags:
toc: true
---
flutter文件读写可以对磁盘文件进行操作，实现某些业务场景，那么我们开始来讲下这个文件读写操作。

使用的库插件(package)

[dart:io](https://api.flutter.dev/flutter/dart-io/dart-io-library.html)（用于数据处理）
[path_provider](https://pub.flutter-io.cn/packages/path_provider) （用于获取路劲）

# 操作步骤
1.获取正确的本地路径
2.创建指向文件位置的引用
3.写入数据到文件内
4.从文件读取数据

# 步骤
## 1.获取正确的本地路径
我们获取路劲用的是这个插件
[path_provider](https://pub.flutter-io.cn/packages/path_provider) 
可以看到里面提供了两个获取路劲的方式

### Example

```
Directory tempDir = await getTemporaryDirectory();
String tempPath = tempDir.path;

Directory appDocDir = await getApplicationDocumentsDirectory();
String appDocPath = appDocDir.path;
```

getTemporaryDirectory：【临时文件夹】
也就是系统可以随时清空的临时缓存文件夹，在IOS中对应[NSTemporaryDirectory](https://developer.apple.com/documentation/foundation/1409211-nstemporarydirectory)在安卓中对应[getCacheDir() ](https://developer.android.google.cn/reference/android/content/Context#getCacheDir())

我们来将信息储存在临时文件夹中，首先我们创建一个Storage类里面开始写

```
class Storage {
  Future<String> get _localPath async {
    final _path = await getTemporaryDirectory();
    return _path.path;
  }
}
```

## 2.创建指向文件位置的引用
确定文件储存位置之后，导入我们的io库，使用包里面的File类做泛型，然后获取路劲并且指向我们的文件名

```
Future<File> get _localFile async {
  final path = await _localPath;
  return File('$path/counter.txt');
}
```


## 3.写入数据到文件内
现在有了可以使用的File，直接就可以来读写数据了，因为我们使用了计数器，所以只需将证书储存为字符串格式，
使用“$counter”即可（解析成整数方法在下一步）

```
Future<File> writeCounter(counter) async {
    final file = await _localFile;

    return file.writeAsString('$counter');
  }
```

## 4.从文件读取数据
现在可以直接用file类来读取文件数据，然后用int的自带解析方法来解析我们读取的String

```
Future<int> readCounter() async {
    try {
      final file = await _localFile;

      var contents = await file.readAsString();

      return int.parse(contents);
    } catch (e) {
      return 0;
    }
  }
```

# 代码

```
import 'dart:io';
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';

class Storage {
  Future<String> get _localPath async {
    final _path = await getTemporaryDirectory();
    return _path.path;
  }

  Future<File> get _localFile async {
    final path = await _localPath;

    return File('$path/counter.txt');
  }

  Future<int> readCounter() async {
    try {
      final file = await _localFile;

      var contents = await file.readAsString();

      return int.parse(contents);
    } catch (e) {
      return 0;
    }
  }

  Future<File> writeCounter(counter) async {
    final file = await _localFile;

    return file.writeAsString('$counter');
  }
}

class OnePage extends StatefulWidget {
  final Storage storage;

  OnePage({this.storage});

  @override
  _OnePageState createState() => _OnePageState();
}

class _OnePageState extends State<OnePage> {
  int _counter;

  @override
  void initState() {
    super.initState();
    widget.storage.readCounter().then((value) {
      setState(() => _counter = value);
    });
  }

  Future<File> _incrementCounter() async {
    setState(() => _counter++);
    return widget.storage.writeCounter(_counter);
  }

  Future<File> _incrementCounterj() async {
    setState(() => _counter--);
    return widget.storage.writeCounter(_counter);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text(
          '$_counter',
          style: Theme.of(context).textTheme.display1,
        ),
      ),
      floatingActionButton: Row(
        children: <Widget>[
          FloatingActionButton(
            onPressed: () => _incrementCounter(),
            child: new Icon(Icons.add),
          ),
          FloatingActionButton(
            onPressed: () => _incrementCounterj(),
            child: new Icon(Icons.title),
          )
        ],
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
    );
  }
}

```
