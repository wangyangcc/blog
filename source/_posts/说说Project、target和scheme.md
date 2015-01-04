title: 说说Project、target和scheme
date: 2014-12-05 11:32:19
tags:
---
> 工作中PTC(Project、target和scheme简称)经常编辑，但苦于完成项目，对PTC的用途，差异了解的不多，下面就简单谈谈

## Project 和 target
查看官方解释猛戳：[Project](https://developer.apple.com/library/ios/featuredarticles/XcodeConcepts/Concept-Projects.html#//apple_ref/doc/uid/TP40009328-CH5-SW1) 和 [Target](https://developer.apple.com/library/ios/featuredarticles/XcodeConcepts/Concept-Targets.html#//apple_ref/doc/uid/TP40009328-CH4-SW1) 和 [Build_Setting](https://developer.apple.com/library/mac/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html#//apple_ref/doc/uid/TP40003931-CH3-SW105)

主要谈谈两者区别：

- Project 是一组文件和常规配置参数集合
- Target 是包含一部分文件(会成为Project文件夹的一部分)和特殊配置参数(在Target配置的参数会优先于Project配置的相同参数执行)
- 每个Target编译运行后都会得到product(产物，比如app,扩展,静态库等)
- 因为Target编译后能得到一些产物，所以target里面还有`Build Phases` 和 `Build Rules`。这些是Project里面没有的，在这里你可以添加依赖关系，修改复制内容的位置和方式

这部分参考自:http://stackoverflow.com/questions/5881048/what-is-the-difference-between-target-and-project

## Scheme

Scheme 是一系列参数的集合，用来指定那个target被build，指定使用什么build配置，指定target对应的特殊项目启动的时候什么样的执行环境被使用。当你打开存在的项目或者创建一个新的项目时,Xcode会自动为每个target创建一个scheme.

更多查看:[https://developer.apple.com/library/mac/recipes/xcode_help-scheme_editor/Articles/SchemeDialog.html#//apple_ref/doc/uid/TP40010402-CH1-SW1](https://developer.apple.com/library/mac/recipes/xcode_help-scheme_editor/Articles/SchemeDialog.html#//apple_ref/doc/uid/TP40010402-CH1-SW1)

## target里面的一些参数用法

#### `Write Link Map File`用法
此参数默认为NO设置为YES，可以查看整个target 编译后的可执行文件的构成

编译后，到编译目录里找到该txt文件，文件名和路径就是target里面的`Path to Link Map File`对应的值，路径如下：
~/Library/Developer/Xcode/DerivedData/XXX-eumsvrzbvgfofvbfsoqokmjprvuh/Build/Intermediates/XXX.build/Debug-iphoneos(或者Release-iphoneos)/XXX.build/
XXX表示项目名，编译是针对cpu类型编译的，如果你的项目支持多种cpu，会有多个tex文件
关于这个参数更多的信息可查看:[http://blog.cnbang.net/tech/2296/](http://blog.cnbang.net/tech/2296/)

会陆续更新一些使用心得。