## 适用范围
大部分情况下均可用React Native一套代码跑两个平台，比如信息展示和交互等等。  
如果涉及到手机设备如摄像头、定位、地图等，则要么封装原生代码给React Native调用，要么直接跳转到用原生代码开发的页面。

## 搭建开发环境
**硬件条件**：推荐使用Mac电脑开发，可以同时开发iOS和Android两个平台

### 必装工具
[安装指南](http://reactnative.cn/docs/0.31/getting-started.html)  
1. [Homebrew](http://brew.sh/) (Mac下的包管理工具)   
安装方法：
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. [Node](https://nodejs.org/en/)(Javascript后端，主要用他的npm包管理工具)  
安装方法：

```
brew install node
```
注意：若安装速度太慢，可考虑换[国内的brew源](http://zhihu.com/question/31360766/answer/74155248)
3. [react-native-cli](https://www.npmjs.com/package/react-native-cli)(React-Native命令行工具)  
安装方法：

```
npm install -g react-native-cli
```
注意：若安装速度太慢，可考虑换[国内的淘宝npm源]http://npm.taobao.org/)
4. [Xcode](https://developer.apple.com/xcode/)  
安装方法:
App Store，版本Xcode7以上
5. [Android Studio](https://developer.android.com/studio/index.html)  
安装方法：
主要是下载正确的SDK，请参照此[目标平台为Android的文章](http://reactnative.cn/docs/0.31/getting-started.html)
6. [Watchman](https://facebook.github.io/watchman/)(用于检测文件变化)  
安装方法：

```
brew install watchman
```

### 测试安装

```
react-native init AwesomeProject //下载react native的项目模版并命名为AwesomeProject
cd AwesomeProject
react-native run-ios //启动该项目的iOS模拟器并运行
```
若能成功启动，则你的环境已配置好，可以开始开发啦！
