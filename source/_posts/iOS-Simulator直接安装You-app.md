title: iOS Simulator直接安装You.app
date: 2014-12-31 15:06:17
tags:
---
> 在提交Facebook应用审核的，奇葩的需要提交`.app`，让审核团队直接在模拟器中安装。分享下找到的方法

### 使用第三方软件 ios-sim

`ios-sim`是一个命令行工具，能直接在iOS模拟器里面安装`.app`文件。也能用作自动化测试工具，在不打开Xcode的情况下打包

下载地址:[https://github.com/phonegap/ios-sim](https://github.com/phonegap/ios-sim)

安装方式：(如果你已经安装过 node.js)

      $ npm install ios-sim -g