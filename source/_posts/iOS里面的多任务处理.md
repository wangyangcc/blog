title: iOS里面的多任务处理
date: 2014-11-10 09:27:58
tags:
---
> 官方对于后台模式的说明大意是，当用户不在激活你的应用时候，系统就会把应用切换到后台状态。对于一些应用，后台状态只是暂停状态。这是为了增加续航，也是为了把宝贵的系统资源(如内存)供给前台运行的应用使用。

如果你需要让你的应用保持在后台运行，iOS能给你提供高效率的不过多消耗系统资源和用户电量的方案。有以下几种方式：

- 应用在前台运行时，切换到后台运行的时候，可以请求一段时间来完成一个简短的任务。主要是iOS4以后提供的方法，有时间限制。
- 应用在前台运行时开始`下载任务`，应用可以把这些`下载任务`的管理权交给系统，这样当应用暂停或者终止的时候，还可以继续下载。主要是使用iOS7以后提供的`NSURLSession`。
- 应用需要运行在后台来完成一些特殊的任务(比如Ip电话，持续定位等)，应用可以申请一个或者多个后台运行模式来支持完成这些任务。主要是通过在`info`配置文件里面加入相关的key来实现。(这里要注意一点是，如果你加入了相关的key，却没有实现相关的功能，在提交appstore审核的时候很可能会被拒绝。)

官方强调:`尽量避免做任何后台工作，除非这样做提高了整体的用户体验`。应用可能在以下情况下进入后台，当用户打开另外一个应用，当用户锁屏，这些情况都表示用户不在使用应用了。在这两种情况下也是在告诉应用，用户不需要做任何富有意义的事情了。继续在这种条件保持后台运行只会耗尽设备的电池，可能会让用户感到反感并完全推出你的应用(iOS8以后，用户可以在`设置>通用>用量>电池用量`里面查看应用的耗电量了)。所以要十分小心的处理应用在后台做的工作，尽量避免使用。

## 执行有限时间的任务(iOS4以后提供)

应用在被用户切换到后台的时候就会被系统暂停掉，如果这时你的应用正在进行一个任务中并且需要额外的时间来完成这个任务。你可以使用`UIApplication`的[beginBackgroundTaskWithName:expirationHandler:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/instm/UIApplication/beginBackgroundTaskWithName:expirationHandler:) 或者[beginBackgroundTaskWithExpirationHandler:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/instm/UIApplication/beginBackgroundTaskWithExpirationHandler:)方法来申请一些额外的时间。调用两个方法里面的任何一个方法来延迟系统暂停你的应用，并申请额外的时间来完成未完成的任务。`注意`，当你完成任务后，必须调用[endBackgroundTask: ](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/instm/UIApplication/endBackgroundTask:)方法来告诉系统任务完成了，可以暂停应用了。

每次调用`beginBackgroundTaskWithName:expirationHandler:` 或者`beginBackgroundTaskWithExpirationHandler:` 方法都会生成一个唯一的标记关联的相应任务。当你的应用完成了任务，必须调用` endBackgroundTask:`方法，传入相应标示的参数，让系统知道你完成任务了。在进入后台执行完任务后不调用` endBackgroundTask:`方法，会导致你的应用被终止掉，如果你在开始任务提供了一个`expiration handler`，系统会调用这个方法并给你最后一个机会结束任务(调用`expiration handler`方法)避免被系统终止掉。

你不需要在应用切换到后台的适合才指派后台任务，更优美的设计是在开始任务的时候调用`eginBackgroundTaskWithName:expirationHandler:` 或者`beginBackgroundTaskWithExpirationHandler:`方法，在任务完成的时候调用`endBackgroundTask: `方法，即便应用在前台的时候也可以按照这样的模式写。

示例如下：
       
     - (void)applicationDidEnterBackground:(UIApplication *)application
    {
        bgTask = [application beginBackgroundTaskWithName:@"MyTask"     expirationHandler:^{
            // Clean up any unfinished task business by marking where you
            // stopped or ending the task outright.
            [application endBackgroundTask:bgTask];
            bgTask = UIBackgroundTaskInvalid;
        }];
 
        // Start the long-running task and return immediately.
                dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
 
        // Do the work associated with the task, preferably in chunks.
 
            [application endBackgroundTask:bgTask];
            bgTask = UIBackgroundTaskInvalid;
        });
    }

bgTask 是示例变量
> 通常在开始任务的时候提供一个`expiration handler`，但是如果你想知道你还有多长时间可以运行，可以查看`UIApplication`的[backgroundTimeRemaining](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/instp/UIApplication/backgroundTimeRemaining)属性值。

在你的`expiration handlers`方法里，你可以添加需要的额外的代码来停止任务。但是任何你在这个方法里面添加的代码都不能花费太多的时间。因为系统在调用这个方法的时候，你的应用已经十分接近后台任务的能运行的时间极限。通常只在这个方法里执行最少的代码来清理你的状态信息然后终止任务。

## 在后台下载内容

当下载文件时，应用应该使用`NSURLSession`来下载，这样系统就能得到下载进程的控制权，防止应用暂停或者终止。当你为后台传输配置一个`NSURLSession`对象，系统在一个特殊的进程里管理这些传输，并通过通用(in the usual way)的方法来回调应用现在的状态。如果你的应用在传输的时候终止了，系统会在后台继续传输，当传输完成或者一个或多个任务完成需要应用关注的时候，系统会加载你的应用(适当的)。

为了支持后台传输，你必须适当的配置`NSURLSession`。你需要先创建一个`NSURLSessionConfiguration`对象然后设置几个属性为恰当的值来配置`session`，当创建`session`的时候通过`NSURLSession`提供的几个初始化方法把配置对象(`NSURLSessionConfiguration`对象)传递过去。

下面列出来创建一个支持后台传输的配置对象的过程：

1. 通过`NSURLSessionConfiguration`的[backgroundSessionConfigurationWithIdentifier:](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/clm/NSURLSessionConfiguration/backgroundSessionConfigurationWithIdentifier:)方法创建一个配置对象.
2. 设置这个配置对象的[sessionSendsLaunchEvents ](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/instp/NSURLSessionConfiguration/sessionSendsLaunchEvents)属性为`YES`
3. 如果你的应用在处于前台的时候就开始传输，推荐设置这个配置对象的[discretionary](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/instp/NSURLSessionConfiguration/discretionary)属性为`YES`
4. 恰当的设置这个配置对象的其他属性。
5. 通过这个配置对象来创建你的`NSURLSession`对象。

一旦配置过，你的`NSURLSession`对象就能在恰当的时机通过系统来启动上传和下载。如果任务完成后你的应用还在运行中(不管是在前台还是后天)，`session`对象就会通过`delegate`通知代理对象。如果你的任务还没有完成的时候系统终止了你的应用，系统会在后台自动继续这个任务并管理它。`如果用户手动终止了你的应用，系统会取消任何没有完成的后台传输任务.`

当一个后台`session`完成相关联的所有任务的时候，系统会重新加载终止的应用(假设`sessionSendsLaunchEvents`设置为`YES`，并且用户没有手动终止应用程序)并调用应用代理的`application:handleEventsForBackgroundURLSession:completionHandler: `方法。()在你实现相关代理的方法里，通过相同的标示符创建和之前一样的具有相同配置的`NSURLSessionConfiguration`对象和`NSURLSession`对象。系统会重新在新的`session `对象和之前完成的任务之间建立联系，并通过`session`对象的代理来报告状态。

## 实现长时间执行的任务
如果你请求了特殊的后台执行任务，那任务就能更多的执行时间而非暂停状态。下面的这些特殊的类型可以运行在后台：

- 应用在后台播放`听得见`的声音，比如音乐播放器
- 应用在后台录音
- 应用在后台始终获得用户的位置更新，比如导航应用
- 应用支持互联网电话(Voice over Internet Protocol，简称`VoIP`)
- 应用需要定期的下载和处理更新的内容，比如报刊
- 应用需要定期的获取从外部附件中获取更新

应用必须申请支持这些服务来实现相关的服务(在info里面指定)并且使用系统提供的相关功能的框架来实现这些功能。让系统知道应用申请了那些服务，一些情况下是系统的相关功能框架阻止了你的应用进入暂停状态。
这部分关于每个部分的详细说明，请看原文，底部有链接。

## 了解进入后台的应用何时会被重新加载
支持后台运行的应用可能由系统传入的事件来重新启动，如果应用除了用户手动终止外由于一些原因被终止了，当下面的事件发生时系统会重新加载应用：

- 对于定位类的应用:1、系统会接受到符合应用配置的位置更新，2、设置进入或者离开注册过的地区(地区可以地理上的地区或者`iBeacon`地区)
- 对于声音播放的应用(包括播放声音的应用或者使用麦克风的)，播放声音的框架需要应用处理一些数据。
- 对于使用蓝牙的应用:1、作为核心角色的应用从周边的设备接收到数据,2、应用从连接中心接收到一个命令
- 对于使用后台下载的应用:1、应用接收到推送的通知，并且通知里面包含key为`content-available`，值为`1`的字段；2、系统会在何时的实际唤醒应用当下载新的内容时候；3、对于使用[NSURLSession](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/cl/NSURLSession)在后台下载内容的应用，所有任务中的任何一个执行完或者收到错误的时候
- 报刊中一个任务下载完成后。

大部分情况下，当用户手动终止应用后系统都不会重新加载应用。但有一个例外，那就是基于位置的应用，在iOS8和iOS8之后就算用户手动终止应用，系统也会重新加载应用。在其他情况下，在系统自动加载处于后台运行的应用前用户必须手动加载应用。

## 成为一个可靠的后台运行程序
在使用系统资源和硬件上，前台运行程序当对于后台运行程序有优先权。应用在后台运行的时候要适应这种差异并且调整自身的行为。具体点说，切换到后台的应用应该符合下面列出的：

- `不在代码里调用任何OpenGL ES相关api.`在后台运行的时候不能创建[EAGLContext](https://developer.apple.com/library/ios/documentation/OpenGLES/Reference/EAGLContext_ClassRef/index.html#//apple_ref/occ/cl/EAGLContext)对象或任何`OpenGL ES`相关的绘画命令，使用这些会导致你的应用被系统立刻终止掉。应用必须保证在切换到后台的时候，任何之前注册的相关命令已经完成了。更多的关于处理OpenGL ES和进入推出后台运行的资料查看[Implementing a Multitasking-aware OpenGL ES Application](https://developer.apple.com/library/ios/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/ImplementingaMultitasking-awareOpenGLESApplication/ImplementingaMultitasking-awareOpenGLESApplication.html#//apple_ref/doc/uid/TP40008793-CH5)。
- `在进入暂停状态前取消所有的Bonjour-related服务。`当你的应用切换到后台，在暂停前，应用应该注销并关闭相关的`sockets`网络服务。不管什么情况下一个暂停的应用都不能响应新回复的网络请求。如果你自己没有关闭`Bonjour `，系统会自动关闭处于暂停状态的`Bonjour`服务。
- `准备好处理你基于网络套接字的连接失败`，系统会因一些原因取消掉`sockets`请求。
- `在切换到后台的时候保存你的应用状态。`但系统可用内存很低的时候，后台运行的应用可能被清理掉以便节省内存空间。首先会清理暂停的应用，并且在清理前没有任何通知。更多的请查看[Preserving Your App’s Visual Appearance Across Launches](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html#//apple_ref/doc/uid/TP40007072-CH5-SW2)
- `切换到后台的时候移除不需要的强引用对象`,更多查看[Reduce Your Memory Footprint.](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW28)
- `进入暂停状态后停止使用共享的系统资源`，应用会和共享的系统资源(例如通讯录，日历等)发生相互作用，在进入暂停状态后应该停止使用这些资源。通常处于前台的应用有使用这些资源的优先权。当你的应用处于暂停状态，被系统发现使用共享的资源时，应用会被终止掉。
- `避免更新你的windows和views`，因为你的windows和views都不再显示了，有个例外是更新你的屏幕截图。
- `对外部配件的连接和取消连接请求作出相应`，对于连接外部配件的应用，系统会在应用切换到后天的时候自动发送断开连接的通知，当应用切换到前台的时候，系统会发送自动连接通知。更多查看[External Accessory Programming Topics.](https://developer.apple.com/library/ios/featuredarticles/ExternalAccessoryPT/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009502)
- `切换到后台的时候清楚现实的alerts资源`，由于系统不会自动取消显示，所以需要手动取消`UIAlertView`和`UIActionSheet`的显示。
- `切换到后台的时候清楚敏感信息`，当应用切换到后台，系统会保存一个当前屏幕的快照，当下次用户再切换到应用的时候，会短暂的显示出来。在你的[applicationDidEnterBackground: ](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/index.html#//apple_ref/occ/intfm/UIApplicationDelegate/applicationDidEnterBackground:)方法返回前，你应该隐藏或者模糊掉可能显示在屏幕快照上的密码或者个人敏感信息。
- `在后台运行的时候做最少的工作`，给后台运行应用的执行时间比起前台运行程序有更多的限制，花太多时间在后台执行的应用可能被系统终止掉。

如果你实现了后台播放的应用，或者别的在后台运行的应用类型，你的应用以平常的方式响应信息。换句话说，系统会在可用内存很低的时候发送内存警告，在这种那个情况下系统会终止应用以便获得更多可用内存，系统会调用应用的[applicationWillTerminate: ](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/index.html#//apple_ref/occ/intfm/UIApplicationDelegate/applicationWillTerminate:)以便在终止前做最后的处理。

## 退出后台运行
如果你不想你的应用总是保持后台运行，你可以手动退出后台运行模式，在应用的`Info.plist`里面添加`UIApplicationExitsOnSuspend`key设置为`YES`,当应用程序退出时，它在不运行、 不活动，和积极的状态之间循环，永远不会进入后台运行或暂停的状态。当用户按home键终止应用时，[applicationWillTerminate: ](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/index.html#//apple_ref/occ/intfm/UIApplicationDelegate/applicationWillTerminate:)会被调用，大约在被系统终止掉前有五秒的时间来处理一些任务。

翻译自:[https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8)

水平所限，翻译的不对的地方还望指出哦~
