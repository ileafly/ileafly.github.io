
title: Flutter--状态管理
date: 2019-01-07 14:24:28 +0800
tags: []
categories: Flutter
---

> 
> 重点知识
> - 有些widgets是有状态的，有些是无状态的
> - 如果用户与widget交互，widget会发生变化，那么它就是有状态的
> - widget的状态保存在一个State对象中，它和widget的布局显示分离
> - 当widget状态改变时，State对象调用setState()，告诉框架去重绘widget


<a name="37c07438"></a>
#### 创建一个有状态的widget
> - 要创建一个自定义有状态的widget，需要创建两个类：StatefullWidget和State
> - 状态对象包含widget的状态和build()方法
> - 当widget的状态改变时，状态对象调用setState(), 告诉框架重绘widget


```
class MyApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _MyAppState();
  }
}

class _MyAppState extends State<MyApp> {

  int counter;

  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    counter = counter ?? 0;
  }

  void _decrementCounter(_) {
    setState(() {
      counter--;
      print('decrement: $counter');
    });
  }

  void _incrementCounter(_) {
    setState(() {
      counter++;
      print('increment: $counter');
    });
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: "flutter_app",
      home: new Home(
        title: 'My Home Page',
        counter: counter,
        decrementCounter: _decrementCounter,
        incrementCounter: _incrementCounter,
      ),
    );
  }
}
```

<a name="6f17667e"></a>
#### 管理状态
> - 有多种方法可以管理状态
> - 选择使用何种管理方法
> - 如果不是很清楚时，那就在父widget中管理状态吧

<a name="161fbf07"></a>
##### 常见的状态管理方法有：

1. widget管理自己的state
1. 父widget管理widget状态
1. 混搭管理
<a name="37812ab7"></a>
##### 如何决定使用哪种管理方法？

1. 如果状态是用户数据，如复选框的选中状态、滑块的位置等，该状态最好由父widget管理
1. 如果状态是有关界面外观效果的，如动画，该状态最好由widget本身管理

