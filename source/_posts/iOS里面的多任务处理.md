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

主要翻译自:[https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8)

持续更新中...
