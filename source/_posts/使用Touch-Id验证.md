title: 使用Touch Id验证
date: 2014-12-26 14:06:57
tags:
---

在新版支付宝和新版QQ中都加入了指纹识别功能，这么高大上的功能自然要学学怎么使用了。

像这种调用系统功能的，自然要用到系统提供的库。`LocalAutoentiaction.framework`，这个包在iOS8里面是自动加入的。

### 先判断是否支持指纹识别

iPhone 4s是能安装iOS8的，但直到iPhone 5s才支持指纹识别，所以要先判断下

Object-c 代码：

    LAContext *context = [[LAContext alloc] init];
    __block  NSString *msg;
    NSError *error;
    BOOL success;
    success = [context canEvaluatePolicy: LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error];
    if (success) {
        NSLog(@"指纹识别 可用");
    } else {
        NSLog(@"指纹识别 不可用");
    }
Swift 代码：
        
        //初始化一个 不可变 LAContext对象
        let context: LAContext! = LAContext()
        var errora: NSError?
        var msg: String?
        
        //查看设备是否支持指纹识别，只支持iOS8以上
        if context.canEvaluatePolicy(LAPolicy.DeviceOwnerAuthenticationWithBiometrics, error: &errora)
        {
            msg = "touch id 可以使用"
        }
        else
        {
            msg = "touch id 不可以使用"
        }
> 注意，上面的代码必须在iOS8下才能运行

### 启动指纹识别验证

Object-c 代码：

    LAContext *context = [[LAContext alloc] init];
    __block  NSString *msg;
    // show the authentication UI with our reason string
    [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"验证理由" reply:
     ^(BOOL success, NSError *authenticationError) {
         if (success) {
             msg = @"验证成功";
         } else {
             msg = @"验证失败";
         }
     }];
     
Swift 代码：

        let context: LAContext! = LAContext()
        var magT: String?
        
        context.evaluatePolicy(LAPolicy.DeviceOwnerAuthenticationWithBiometrics, localizedReason: "验证理由") { (success, authenticationError) -> Void in
            if success
            {
                magT = "验证成功"
            }
            else
            {
                magT = "验证失败"
            }
            //打印提示信息
            self.printResult(magT!)
        }
        
> 注意一点，当用户点击“输入密码”时，是调用自己程序的输入密码界面，因为在使用touch Id验证的时候，一般是程序内部需要使用密码验证的地方。

指纹识别很简单吧，下面提供两个示例demo：

官方的：[https://developer.apple.com/library/ios/samplecode/KeychainTouchID/Introduction/Intro.html](https://developer.apple.com/library/ios/samplecode/KeychainTouchID/Introduction/Intro.html)
本人用Swift写的：[https://github.com/wangyangcc/TouchIdTest_Swift](https://github.com/wangyangcc/TouchIdTest_Swift)