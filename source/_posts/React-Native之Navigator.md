title: React-Native之Navigator
categories:
  - ReactNative
date: 2016-11-24 11:26:00
---

#### 结合官方文档，简单使用Navigator

React Native内置了navigation组件，在[官方文档](http://facebook.github.io/react-native/docs/using-navigators.html)中对于如何使用Navigator有了一个简要的说明，下面就让我们跟随文档进行学习。 

1. 页面搭建

   根据文档的描述，我们需要先创建一个文件命名为`MyScene.js`，内容如下：

   ```javascript
   // 导入需要使用的组件
   import React, { Component } from 'react';
   import { View, Text, Navigator } from 'react-native';

   // 声明MyScene类，并设置为外部可以引用
   export default class MyScene extends Component {
     static get defaultProps() {
       return {
         title: 'MyScene'
       };
     }

     // 绘制界面
     render() {
       return (
         <View style={{height: 200, top: 64}}>
           <Text>Hi! My name is {this.props.title}.</Text>
         </View>
       )
     }
   }
   ```

​      创建完`MyScene.js`后我们就需要进入到`index.ios.js`中进行更改。 

```javascript
// 导入需要用到的组件
import React, { Component } from 'react';
import { AppRegistry } from 'react-native';

// 导入需要引用的文件
import MyScene from './MyScene';

// 声明入口类
class YoDawgApp extends Component {
  // 绘制界面，通过加载 MyScene
  render() {
    return (
      <MyScene />
    )
  }
}
// 注册入口类
AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
```

​      进行上述操作后运行项目来看一下效果：

![React-Native之Navigator_1](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/ReactNative/React-Native之Navigator_1.png)

显然，跟我们想要的效果差很远。       

2. 使用Navigator

   接下来我们就需要利用`Navigator`组件，修改`index.ios.js`文件

   ```javascript
   // 导入需要用到的组件
   import React, { Component } from 'react';
   import { AppRegistry } from 'react-native';

   // 导入需要引用的文件
   import MyScene from './MyScene';

   // 声明入口类
   class YoDawgApp extends Component {
     // 绘制界面，通过加载 MyScene
     render() {
       return (
         // 创建一个Navigator 并嵌套 MyScene
         <Navigator
            initialRoute={{ title: 'My Initial Scene', index: 0}}
            renderScene={(route, navigator) => {
              return <MyScene title={route.title} />
            }}
         />
       )
     }
   }
   // 注册入口类
   AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
   ```

   进行上述操作后我们再次运行项目：

   ![React-Native之Navigator_2.png](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/ReactNative/React-Native之Navigator_2.png)

   比较两次的运行结果，似乎只是文案内容发生了改变，让我们继续下去看到更多的变化

3. 加入`push` `pop` 

   `navigator`给我们提供了`push` `pop` 操作，我们来继续修改`index.ios.js`

   ```javascript
   // 导入需要用到的组件
   import React, { Component } from 'react';
   import { AppRegistry, Navigator, Text, StyleSheet, View, TouchableOpacity } from 'react-native';

   // 导入需要引用的文件
   import MyScene from './MyScene';

   // 声明入口类
   class YoDawgApp extends Component {
     // 绘制界面，通过加载 MyScene
     render() {
       return (
         // 创建一个Navigator 并嵌套 MyScene
         <Navigator
            initialRoute={{ title: 'My Initial Scene', index: 0}}
            renderScene={(route, navigator) => {
              return <MyScene title={route.title} 
                 // push操作调用函数
                  onForward={ () => {
                    const nextIndex = route.index + 1;
                    navigator.push({
                       title: 'Scene ' + nextIndex,
                       index: nextIndex,
                    });
                  }}
     
                 // pop操作调用函数
                  onBack={ () => {
                    if (route.index > 0) {
                      navigator.pop();
                    }
                  }}
              />
            }}
         />
       )
     }
   }
   // 注册入口类
   AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
   ```

   现在我们在`MyScene.js` 里添加上点击事件，并调用`onForward`和`onBack`方法看看会有怎样的效果

   ```javascript
   // 导入需要使用的组件
   import React, { Component } from 'react';
   import { View, Text, Navigator, TouchableHighlight } from 'react-native';

   // 声明MyScene类，并设置为外部可以引用
   export default class MyScene extends Component {
     static get defaultProps() {
       return {
         title: 'MyScene'
       };
     }

     // 绘制界面
     render() {
       return (
         <View style={{height: 200, top: 64}}>
           <Text>Hi! My name is {this.props.title}.</Text>
         // 添加两个可点击区域，分别用于push和pop
           <TouchableHighlight onPress={this.props.onForward}>
              <Text>Tap me to load the next scene</Text>
           </TouchableHighlight>
           <TouchableHighlight onPress={this.props.onBack}>
              <Text>Tap me to go back</Text>
           </TouchableHighlight>
         </View>
       )
     }
   }

   MyScene.propTypes = {
     title: PropTypes.string.isRequired,
     onForward: PropTypes.func.isRequired,
     onBack: PropTypes.func.isRequired,
   };
   ```

   再次运行项目看一下效果：

   ![React-Native之Navigator_3.png](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/ReactNative/React-Native之Navigator_3.png)

   点击`Tap me to load the next scene`能够push下一个页面

   点击`Tap me to go back`能够pop到上一个页面

   看来我们已经顺利集成了`Navigator` ,不过，似乎还缺少什么？

4. 显示Navigator Bar

   通过上述操作我们能够正常实现`Navigator` 的`push` 和`pop` 操作啦，但是，我们一直没有看到导航栏，这是因为在`React Native`中我们需要额外设置才能够展现`Navigator Bar` ，下面就让我们来继续修改`index.ios.js`

   ```javascript
   // 导入需要用到的组件
   import React, { Component } from 'react';
   import { AppRegistry, Navigator, Text, StyleSheet, View, TouchableOpacity } from 'react-native';

   // 导入需要引用的文件
   import MyScene from './MyScene';

   // 声明入口类
   class YoDawgApp extends Component {
     // 绘制界面，通过加载 MyScene
     render() {
       return (
         // 创建一个Navigator 并嵌套 MyScene
         <Navigator
            initialRoute={{ title: 'My Initial Scene', index: 0}}
            renderScene={(route, navigator) => {
              return <MyScene title={route.title} 
                 // push操作调用函数
                  onForward={ () => {
                    const nextIndex = route.index + 1;
                    navigator.push({
                       title: 'Scene ' + nextIndex,
                       index: nextIndex,
                    });
                  }}
     
                 // pop操作调用函数
                  onBack={ () => {
                    if (route.index > 0) {
                      navigator.pop();
                    }
                  }}
              />
            }}
            // 增加navigationBar
             navigationBar={
                     	<Navigator.NavigationBar
                          routeMapper={NavigationBarRouteMapper}
                          style={{backgroundColor:'#fff',
                                  borderColor:'#dddddd',
                                  borderWidth:1,
                                  height: 64}}
                     	/>
            }
         />
       )
     }
   }

   // 导航栏的Mapper
   var NavigationBarRouteMapper = {
     // 左键
     LeftButton(route, navigator, index, navState) {
       if (index > 0) {
         return (
           <View style={styles.navContainer}>
             <TouchableOpacity
               underlayColor='transparent'
               onPress={() => {if (index > 0) {navigator.pop()}}}>
               <Text style={styles.leftNavButtonText}>
                 后退
               </Text>
             </TouchableOpacity>
           </View>
         );
       } else {
         return null;
       }
     },
     // 右键
     RightButton(route, navigator, index, navState) {
       if (route.onPress)
         return (
           <View style={styles.navContainer}>
             <TouchableOpacity
               onPress={() => route.onPress()}>
               <Text style={styles.rightNavButtonText}>
                 {route.rightText || '右键'}
               </Text>
             </TouchableOpacity>
           </View>
         );
     },
     // 标题
     Title(route, navigator, index, navState) {
       return (
         <View style={styles.navContainer}>
           <Text style={styles.title}>
             应用标题
           </Text>
         </View>
       );
     }
   };

   var styles = StyleSheet.create({
     navBar: {
         backgroundColor:'#fff',
         borderColor:'#dddddd',
         borderWidth:1,
         height: 64
     },
     navBarTitleText: {
       fontWeight: '500',
     },
     navBarLeftButton: {
       paddingLeft: 5,
     },
     navBarRightButton: {
         marginRight:5,
     },
     icon: {
         width:30,
         height:30,
         marginTop:6,
         textAlign:'center'
     }
   });

   // 注册入口类
   AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
   ```

   我们来最后看一下效果：

   ![React-Native之Navigator_4](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/ReactNative/React-Native之Navigator_4.png)

   ![React-Native之Navigator_5](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/ReactNative/React-Native之Navigator_5.png)

   ​

