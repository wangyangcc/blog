title: iPhone 6/6plus 适配时要注意的一个点
date: 2014-11-20 14:24:33
tags:
---
xCode 6 之前新建的项目默认是不适配6/6plus的，整个屏幕界面都会变大，可以通过下面两种方式来标记项目适配了6/6plus
### 1: 添加 Launch Screen.xib 文件
新建一个Launch Screen.xib，步骤 File > New > File >User Interface >Launch Screen，并在 TARGETS > General >App Iconsand Launch Images 中 指定 Launch Screen File 为刚才新建的文件

通过这种方式建立的，系统会认为应用在iPhone 6Plus 下支持横屏模式，如果你没添加横屏模式功能，并且`Device Orientation`设置为支持Landscape Left 和Landscape Right就会有问题，会看到试图错乱
解决方法是Device Orientation设置如下
![Device Orientation](http://upload-images.jianshu.io/upload_images/68070-c6eaa09154a8c028.png)

并在App delegate里面，添加下面的方法

     - (NSUInteger)application:(UIApplication *)application  supportedInterfaceOrientationsForWindow:(UIWindow *)window {
         return UIInterfaceOrientationMaskAllButUpsideDown;
     }

再次运行会看到一切如初了，这样就Ok啦

### 2: Images.xcassets 里面添加  New Launch Image
如果你没有做iPad 版本，那`LaunchImage`看起来应该是这样的，注意不要勾选图中圈住的那个框
![Launch Image](http://upload-images.jianshu.io/upload_images/68070-f9f5fe6aeded9ab1.png)

1 和 2 两种方式选择一种即可，只是2里面如果6plus 横屏模式下启动应用会是黑色没有启动图的，1则没有这个问题
