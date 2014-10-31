title: 为iOS 8 开发应用扩展
date: 2014-10-31 15:25:38
tags:
---
# 通过应用扩展提升你的应用(`lncrease Your Impact`)
从iOS 8.0 和 OS X v10.10 开始，应用扩展允许你在应用外部提供自定义的功能或内容，以便用户可以在他们使用其他应用或iOS系统的时候使用到该项功能。你可以开发一个应用扩展来执行特定的任务；当用户使用了你的应用扩展后就能在多种环境下(`a variety of contexts.`)执行该任务。例如，你提供一个可以把内容分享到社交网络的应用扩展，用户可以在上网的时候通过应用扩展发表评论；你提供一个扩展来显示用户当前的运动成绩，用户可以“通知中心”里面显示，这样在他们打开“Today view”就可以看到最新的运动成绩；你创建一个自定义键盘的扩展，用户就可以用来替代iOS系统键盘。
例如下面这些 扩展：

## APP扩展的类型
iOS 和 OS X 定义了几种类型的应用扩展，每一种扩展都对应系统的一个区域。比如分享、通知中心、iOS 键盘扩展等。把支持扩展的区域称为称为扩展点(`extension point`)，当你创建扩展时每一个扩展点都定义了使用策略(`usage policies`)并提供了API。已你想提供的功能点为基础选择扩展点(`extension point`)。需要注意的是，当你针对某个扩展点开发应用扩展时，该应用扩展的功能必须要符合该扩展点的功能特性。

下面的表格罗列了iOS 和 OS X 里面的扩展点，并针对每个扩展点举例说明了你可以实现的任务

Extension point | Typical app extension functionality
----|------
Today (iOS and OS X) | Get a quick update or perform a quick task in the Today view of Notification Center(A Today extension is called a `widget`)
Share (iOS and OS X) | Post to a sharing website or share content with others 
Action (iOS and OS X; UI and non-UI variants) | Manipulate or view content originating in a host app
Photo Editing (iOS) | Edit a photo or video within the Photos app
Finder Sync (OS X) | Present information about file sync state directly in Finder.
Document Provider (iOS; UI and non-UI variants) | Provide access to and manage a repository of files.
Custom Keyboard (iOS) | Replace the iOS system keyboard with a custom keyboard for use in all apps 

因为系统为应用扩展提供了特定的区域，选择一个最符合你想提供功能的一个扩展区域是很重要的。例如，你想创建一个分享阅历的扩展，使用分享这个扩展点，在Xcode 扩展模板里选择 "Share Extension"
> 你开发的app扩展要精确的匹配上面表格列出的扩展点，你不能开发一个通用的app扩展来匹配一个以上的扩展点。

## Xcode 和 App Store 帮助你创建并实现(`deliver `) 应用扩展
应用扩展和应用是不一样的，虽然你需要基于应用去包含并实现应用扩展，但是当通过应用来实现应用扩展的时候，每个应用扩展都是独立运行的二进制文件。

在一个app里面添加一个新的Target来新建应用扩展。和其他Target一样，扩展 Target 将设置信息和相关文件打包在Products文件下生成一个扩展名为.appex的包。你可以在一个app项目里面添加多个扩展Target。(一个app可以包含一个或多个应用扩展，该应用程序称为主体app(`a containing app`))

开发一个应用扩展最好的切入点是使用Xcode在iOS和OS X上为各个扩展点提供的模板。每个模板包含扩展点的具体实现文件和相关设置，并生成独立的二进制文件添加到应用程序的包中。

> 在iOS中，包含扩展的应用必须提供一个扩展之外的功能。而在OS X中没有这个硬性要求，一个包含扩展的应用不要求必须提供一个额外功能。        

为了让用户使用到应用扩展，你需要提交一个包含扩展的app到App Store。当用户安装你的包含扩展的app时，app 包含的应用扩展同时也会安装。

当安装了应用扩展后，用户必须要手动开启它。通常，用户可以在他们当前任务的环境中启动扩展。如果你的扩展是Today widget，用户可以在通知中心里面的Today view编辑并添加你的扩展。在其他情况中，用户可以使用 iOS 中的“Settings”或者 OS X 中的“System Preferences”来启用和管理扩展。
## 用户在不同的使用环境下体验不同的应用扩展
虽然不用的应用扩展类型能完成不同的任务，但对大部分扩展来说，它们在用户体验上还是有一些共同点的。如果你准备开发一个应用扩展，有一点很重要，那就是要理解在你选择的扩展点中，用户体验是什么样的。从一个更高角度看，对于所有扩展来说，最佳的用户体验是快速、流畅以及只关注单一任务。

通常用户通过与系统提供的用户界面进行交互来开启应用扩展。比如说，用户在app中通过激活系统提供的分享按钮来访问 Share 扩展，并从展示的列表中选择扩展。

虽然大多数的应用程序扩展都提供了一些自定义的UI元素，但一般用户不会看到你的自定义用户界面，除非他们进入到应用程序扩展中。当用户进入应用扩展，你的自定义UI可以让用户知晓他们正进入一个新的上下文环境。由于用户可以把你的扩展和当前应用区别开来，所以他们会欣赏你提供的独特功能。当用户意识到扩展其实是独立运行的实体时，他们也可以确认并移除体验不好或功能不好的扩展。

为了让用户平滑过渡到你的应用程序扩展，你要斟酌自定义界面与扩展点界面的风格，做一个权衡。比如说，一个很好的方法就是让你的插件看起来像是通知中心中原生的Widget，再比如说照片编辑扩展，你应该创建一个和 iOS 中 Photos 应用风格相协调的用户界面。

> 即使你的应用程序扩展没有展示自定义UI（不包括图标），但用户仍然知道该扩展不同于当前的应用，因为它们需要采用特定的操作来激活。

以上内容是本人参考[cocoachina.com](http://www.cocoachina.com/ios/20140808/9340.html)的翻译结果，转载请注明出处

其他翻译查看  http://www.cocoachina.com/ios/20141023/10027.html