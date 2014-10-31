title: iOS 8 适配的几点总结
date: 2014-10-31 15:18:45
tags:
---
> 自从苹果公司在9月17号开放升级以来，官方显示iOS8已经有46%的份额，微信，支付宝，新浪微博等也已经兼容iOS8，对于开发者来说，兼容iOS8 也是迟早的事情。下面说几点我在兼容iOS8时，发现的几点问题。

## 1、SDK 里面的某些API不能在iOS8下使用
如果，你的老项目在iOS8下运行，打开就闪退(iOS8之前没问题)，那么“恭喜你”，你中招了，比如下面我遇到的，是因为旧版本的高德地图引用了 iOS8 里面不能用的api，如果你也需要类似的问题，那么是时候升级需要升级的第三方库了。
<pre><code>2014-09-28 14:32:25.576 WoZaiXianChang[4505:140022] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIDevice asUniqueDeviceIdentifier]: unrecognized selector sent to instance 0x7c020080' </code></pre>

## 2、iOS8 下面定位功能使用改变了
之前版本的SDk是这样启动系统定位的
   <pre><code> // 判断定位操作是否被允许  
if([CLLocationManager locationServicesEnabled]) {  
    locationManager = [[CLLocationManager alloc] init];  
    locationManager.delegate = self; 
    [locationManager startUpdatingLocation];  
}else {  
    //提示用户无法进行定位操作  
}  </code></pre>如果在iOS8下用这样的方式，你会发现无法定位，那是因为iOS8下添加了新的方法
<pre><code> /表示使用应用程序期间  开启定位  
- (void)requestWhenInUseAuthorization  
//表示始终 开启定位  
- (void)requestAlwaysAuthorization </code></pre>

两者区别在于，iOS7 开始，有更强大的后台运行功能，如果 用  requestAlwaysAuthorization 方法，则表示后台运行时也会用到定位
iOS8 下使用系统定位如下：
<pre><code> // 判断定位操作是否被允许  
    if([CLLocationManager locationServicesEnabled]) {  
        locationManager = [[CLLocationManager alloc] init];  
        locationManager.delegate = self;  
        //兼容iOS8定位  
        SEL requestSelector = NSSelectorFromString(@"requestWhenInUseAuthorization");  
        if ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusNotDetermined &&  
            [locationManager respondsToSelector:requestSelector]) {  
            [locationManager requestWhenInUseAuthorization];  
        } else {  
            [locationManager startUpdatingLocation];  
        }  
        return YES;  
    }else {  
        //提示用户无法进行定位操作  
    }  
    return NO; </code></pre>
同时还需要添加新的方法，其他的都一样
<pre><code> - (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status {  
    if (status == kCLAuthorizationStatusAuthorizedWhenInUse) {  
        [locationManager startUpdatingLocation];  
    } else if (status == kCLAuthorizationStatusAuthorized) {  
        // iOS 7 will redundantly call this line.  
        [locationManager startUpdatingLocation];  
    } else if (status > kCLAuthorizationStatusNotDetermined) {  
        //...  
        [locationManager startUpdatingLocation];  
    }  
}  </code></pre>
除了这些，你还需要在 info.plist 里面添加新的键值，否则 也是无法定位的
<pre><code>//表示使用应用程序期间  开启定位  
- (void)requestWhenInUseAuthorization   对应 NSLocationWhenInUseUsageDescription key 
//表示始终 开启定位  
- (void)requestAlwaysAuthorization      对应 NSLocationAlwaysUsageDescription key</code></pre>

其中,NSLocationWhenInUseUsageDescription(或者NSLocationAlwaysUsageDescription) 对应的文字会在第一次请求用户同意定位的时候出现，还有 设置 > 隐私 > 定位 > your app 里面也会看到，比如下面就是开启app时出现的
![](http://yspe2371e4aa7697989.yunshipei.cn/dHlwZT1mdyZzaXplPTY0MCZzcmM9YUhSMGNDVXpRU1V5UmlVeVJtbHRaeTVpYkc5bkxtTnpaRzR1Ym1WMEpUSkdNakF4TkRBNU1qZ3hOREU0TVRRM09UY2xNMFozWVhSbGNtMWhjbXNsTWtZeUpUSkdkR1Y0ZENVeVJtRklVakJqUkc5MlRESktjMkl5WTNWWk0wNXJZbWsxZFZwWVVYWmtNa1oxV2pOc2FHSnRaRFZaVnpWdVdUSk5KVE5FSlRKR1ptOXVkQ1V5UmpWaE5rdzFUREpVSlRKR1ptOXVkSE5wZW1VbE1rWTBNREFsTWtabWFXeHNKVEpHU1RCS1FsRnJSa05OUVNVelJDVXpSQ1V5Um1ScGMzTnZiSFpsSlRKR056QWxNa1puY21GMmFYUjVKVEpHUTJWdWRHVnk=)
## 3、iOS8 下注册通知的改变
 这个不用多说，直接看代码就明白了，有一点需要注意的是，蓝色部分必须要加，不然即便能取的token值，app 接受到的推送也是无声的。
<pre><code>//注册消息通知  
if (IOS8After) {  
    [[UIApplication sharedApplication] registerForRemoteNotifications];  
    <span style="color:#3333ff;">[[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert categories:nil]];</span>  
}  
else {  
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeBadge)];  
}  </code></pre>
## 4、iOS8 cell 层级的改变
如果你像这样取cell 的row 的话，那你又要加个判断方法了，在iOS8下cell的层级又改了，基本上每升级一个版本，苹果都会对cell的结构进行调整，在此建议不要用这样的方式取cell 的row，而是用属性的方式保存 indexPath
<pre><code>NSUInteger curRow = 0;  
if ([[MetaData getOSVersion] integerValue] == 7)  
{  
    curRow = [(UITableView *)[[self superview] superview] indexPathForCell:self].row;  
}  
else  
{  
    curRow = [(UITableView *)[self superview] indexPathForCell:self].row;  
}  </code></pre>
## 5、UIActionSheet and UIAlertView 的升级
在iOS8里面，官方提供了新的类`UIAlertController `来替换UIActionSheet and UIAlertView。
示例代码如下：
<pre><code>
UIAlertController* alert = [UIAlertController alertControllerWithTitle:@"My Alert"
                               message:@"This is an alert."
                               preferredStyle:UIAlertControllerStyleAlert];
 
UIAlertAction* defaultAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault
   handler:^(UIAlertAction * action) {}];
 
[self presentViewController:alert animated:YES completion:nil];
</pre></code>
至于为什么为加这个类，本人猜测是和iOS8新加的size classes有关，目的是统一屏幕在各个尺寸各个方向上的显示。如果你在iOS 8 里面使用`UIActionSheet and UIAlertView` 可能会出现一些很奇怪的问题，建议在iOS 8 里面使用`UIAlertController`,iOS 8 之前使用`UIActionSheet and UIAlertView`
这是目前为止发现的 iOS8 兼容性问题，大家发现别的兼容性问题，一起讨论哦
本文在`简书`上面也有发布：http://www.jianshu.com/p/22ec8d179d2e