- **AwesomeProject项目的目录结构**
```
├── AwesomeProject
   ├── .buckconfig //默认
   ├── .flowconfig //默认
   ├── .gitignore //默认，可自行配置不纳入Git版本管理的文件
   ├── .watchmanconfig //默认
   ├── android //Android的Android Studio项目
   ├── index.android.js //Android App的入口js
   ├── index.ios.js //iOS App的js入口
   ├── ios //iOS的Xcode项目
   ├── node_modules //下载的第三方库的存放位置
   └── package.json //项目的第三方库的描述文件，用语NPM安装第三方库
```
插句话：这个目录结构树是用命令行tree -al 1生成的，tree命令只需要是用brew install tree安装。

- **入门先看package.json文件，整体的描述这个项目**

```
{
  "name": "AwesomeProject", //项目名
  "version": "0.0.1", //自定版本号
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": { //以后引入的第三方库会增加在这里
    "react": "15.3.1", //必备
    "react-native": "0.33.0" //必备
  }
}

```


- **再看项目入口文件index.ios.js的代码**

```
import React, { Component } from 'react'; //标准写法
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native'; //标准写法，基本上提供的React Native的API都在这个'react-native'包里

class AwesomeProject extends Component { //ES6(ES2015)的类的标准写法，类的概念ES6才引入
  render() {  //这个类的的成员函数render的返回值是此类的UI界面,注意：一定要叫'render'
    return (
      <View style={styles.container}> //style即是该UI组件的样式
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.ios.js
        </Text>
        <Text style={styles.instructions}>
          Press Cmd+R to reload,{'\n'}
          Cmd+D or shake for dev menu
        </Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({  //样式和UI分离开，类似HTML和CSS分离
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);  //标准写法，只用调用一次，相当于说明该index.ios.js是iOS的代码入口

```

- **index.android.js同理**

- **最后来个更常用更标准的文件结构**

```
import React, {Component} from "react";
import {
  Platform,
  StyleSheet,
  Text,
  View,
  Image,
  TextInput,
  TouchableOpacity,
  Alert,
  Navigator
} from "react-native"; //常用的一些类
import {COLOR_BG_NORMAL, APP_VERSION} from "../GlobalConst";  //引入相对于该文件的路径的文件中用export关键词暴露的变量
import Header from "../component/Header"; //引入相对于该文件的路径的类
import Spinner from "react-native-loading-spinner-overlay"; //引入package.json安装后或自行NPM安装的第三方库

export default class AboutPage extends Component {
  constructor(props) {
    super(props);
    this.state = {
      loadingVisible: false, //该类本身的状态值,用于判断是否更新UI
    };
  }

  static propTypes = {
    data: React.PropTypes.object, //验证调用类的属性传的是否是预期的值类型
  };

  static defaultProps = {
    data: {},  //若调用该类时没提供该属性的值,则初始化为这个值
  };

  //该React组件的生命周期函数

  //第一次加载阶段
  componentWillMount() { //render 调用前
  }

  componentDidMount() { //render 绘制结束后
  }

  //更新阶段
  shouldComponentUpdate() { //受到this.setState信号，是否触发render函数,根据render里面的state值是否改变
  }

  componentDidUpdate() {
  }

  //卸载阶段
  componentWillUnMount() {  //该页面销毁前
  }

  componentDidUnMount() { //该页面销毁后
  }

  render() {
    return (<View style={styles.container}>
        <Spinner visible={this.state.loadingVisible}/> //第三方库的组件
        <Header navigator={this.props.navigator} title="关于"/> //自己封装的一个组件
        <Text>我是关于页面</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLOR_BG_NORMAL,
  },
});
```

- **附：学习资源：**
1. [React基础知识](http://www.ruanyifeng.com/blog/2015/03/react.html)
2. [React Native布局指南](http://www.jianshu.com/p/bfe15685c9e0)
3. [React Native ES6新式写法](http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8)
