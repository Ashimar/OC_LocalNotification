#iOS 10 通知之本地通知

> 本地通知顾名思义就是本地推送到本地，类似于闹钟，事件提醒等。在iOS 10 后，本地通知可以通过UserNotifications/UserNotifications.h 框架 进行操作。


实现步骤：

1、基本操作

- 导入UserNotifications/UserNotifications.h 库
	
	最好这样写：
	
	```
	#ifdef NSFoundationVersionNumber_iOS_9_x_Max
	#import <UserNotifications/UserNotifications.h>
	#endif 	
	```
- 遵守<UNUserNotificationCenterDelegate>的协议。
- 在Appdelegate.m 中 注册通知中心
在 `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` 方法中注册通知：

```
	
if ([[UIDevice currentDevice].systemVersion floatValue] >= 10.0) {
    // iOS 10 特有
    // 1、创建一个 UNUserNotificationCenter
    UNUserNotificationCenter *requestCenter = [UNUserNotificationCenter currentNotificationCenter];
    // 必须写代理，不然无法监听通知的接收与点击
    requestCenter.delegate = self;
    [requestCenter requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionBadge | UNAuthorizationOptionSound) completionHandler:^(BOOL granted, NSError * _Nullable error) {
       
        if (granted) {
            
            // 点击允许
            NSLog(@"注册成功");
            [requestCenter getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
                NSLog(@"%@",settings);
            }];
            
        }else {
            // 点击不允许
            NSLog(@"注册失败");
        }
        
    }];
} else if ([[UIDevice currentDevice].systemVersion floatValue] > 8.0) {
    // iOS 8 ~iOS 10
    [application registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil]];
} else if ([[UIDevice currentDevice].systemVersion floatValue] < 8.0) {
    
    
    [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound];
}
// 注册获得device Token
[[UIApplication sharedApplication] registerForRemoteNotifications];

```

2、两个回调方法

```
// iOS 10收到通知
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
    
    
    NSDictionary *userInfo = notification.request.content.userInfo;
    UNNotificationRequest *request = notification.request;  // 收到推送的请求
    UNNotificationContent *content = request.content;       // 收到推送的消息内容
    NSNumber *badge = content.badge;                        // 推送消息的角标
    NSString *body = content.body;                          // 推送消息体
    UNNotificationSound *sound = content.sound;             // 推送消息的声音
    NSString *subString = content.subtitle;                 // 推送消息的副标题
    NSString *title = content.title;                        // 推送消息的标题
    
    if ([notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
        
        NSLog(@"iOS10 前台收到本地通知：");
        
    } else {
        // 判断为本地通知
        NSLog(@"iOS 10 收到本地通知：{\nbody:%@,\ntitle:%@,\nsubtitle:%@,\nbadge:%@,\nsound:%@,\nuserInfo:%@\n}",body,title,subString,badge,sound,userInfo);
    }
    // Warning: UNUserNotificationCenter delegate received call to -userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler: but the completion handler was never called.
    completionHandler(UNNotificationPresentationOptionBadge|UNNotificationPresentationOptionSound|UNNotificationPresentationOptionAlert); // 需要执行这个方法，选择是否提醒用户，有Badge、Sound、Alert三种类型可以设置
}

// 通知的点击事件
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler{
    
    // 通知图标减少一
    [UIApplication sharedApplication].applicationIconBadgeNumber --;
    
    NSDictionary * userInfo = response.notification.request.content.userInfo;
    UNNotificationRequest *request = response.notification.request; // 收到推送的请求
    UNNotificationContent *content = request.content; // 收到推送的消息内容
    NSNumber *badge = content.badge;  // 推送消息的角标
    NSString *body = content.body;    // 推送消息体
    UNNotificationSound *sound = content.sound;  // 推送消息的声音
    NSString *subtitle = content.subtitle;  // 推送消息的副标题
    NSString *title = content.title;  // 推送消息的标题
    if([response.notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
        NSLog(@"iOS10 收到远程通知:");
        
    }
    else {
        // 判断为本地通知
        NSLog(@"iOS10 收到本地通知:{\\\\nbody:%@，\\\\ntitle:%@,\\\\nsubtitle:%@,\\\\nbadge：%@，\\\\nsound：%@，\\\\nuserInfo：%@\\\\n}",body,title,subtitle,badge,sound,userInfo);
    }
    
    // Warning: UNUserNotificationCenter delegate received call to -userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler: but the completion handler was never called.
    completionHandler();  // 系统要求执行这个方法
    
}
```

其他版本的接收通知的回调方法

```
- (void)application:(UIApplication *)application
didReceiveRemoteNotification:(NSDictionary *)userInfo {
//        NSLog(@"iOS6及以下系统，收到通知:%@", [self logDic:userInfo]);
    
}

- (void)application:(UIApplication *)application
didReceiveRemoteNotification:(NSDictionary *)userInfo
fetchCompletionHandler:
(void (^)(UIBackgroundFetchResult))completionHandler {
    
    //    NSLog(@"iOS7及以上系统，收到通知:%@", [self logDic:userInfo]);
    completionHandler(UIBackgroundFetchResultNewData);
}


-(void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification{
    //通知的图标减少方法一：  单例  减少
    [UIApplication sharedApplication].applicationIconBadgeNumber --;
    //减少方法二：
    //    application.applicationIconBadgeNumber--;
    
    // 视图推送到

}
```

3、在需要设置本地通知的地方设置

```
  // 1、创建通知内容
    UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
    content.title = @"本地通知测试";
    content.subtitle = @"测试通知";
    content.body = @"来自ZHZ的CSDN";
    // 设置消息提醒的数目
    NSInteger count = [UIApplication sharedApplication].applicationIconBadgeNumber+1;
    content.badge = [NSNumber numberWithInteger:count];
    
    NSError *error = nil;
    NSString *path = [[NSBundle mainBundle] pathForResource:@"1" ofType:@"jpg"];
    
    // 2、设置通知附件内容
    UNNotificationAttachment *att = [UNNotificationAttachment attachmentWithIdentifier:@"att1" URL:[NSURL fileURLWithPath:path] options:nil error:&error];
    
    if (error) {
        NSLog(@"attachment error %@",error);
    }
    content.attachments = @[att];
    content.launchImageName = @"1";
    
    // 2、设置声音
    UNNotificationSound *sound = [UNNotificationSound defaultSound];
    content.sound = sound;
    
    // 3、触发模式
    UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    
    // 4、设置UNNotificationRequest
    NSString *requestIdentifier = @"TestRequest";
    UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:requestIdentifier content:content trigger:trigger];
    
    // 5、把通知加到UNUserNotificationCenter，到指定触发点会被触发
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        NSLog(@"通知");
    }];


```

参考：[简书徐不同](http://www.jianshu.com/p/3d602a60ca4f)