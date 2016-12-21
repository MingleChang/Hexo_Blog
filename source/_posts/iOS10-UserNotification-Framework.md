---
title: iOS10 UserNotification.Framework
date: 2016-12-21 21:42:53
tags: [ios,UserNotification]
---

作为一个App推送功能基本是每个App都会有的功能，尤其是国内应用，推送功能基本达到了滥用的地步，但是随着苹果公司对推送功能不断的加强，我们能通过推送实现更多的功能，尤其是这次iOS10的发布，增加了UserNotification.Framework，Notification Content和Notification Service Extension，推送功能变得更加强大。

这里主要是介绍UserNotification.Framework。

在iOS10之前我们注册通知的方式有两种，在iOS8之前我们使用

```
[application registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
```

在iOS8之后我们使用

```
UIUserNotificationSettings *settings = [UIUserNotificationSettings  settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
[application registerUserNotificationSettings:settings];

- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)settings
{
    [application registerForRemoteNotifications];
}
```

然后使用下面的方法获取token

```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"Registration successful, bundle identifier: %@, device token: %@",[NSBundle.mainBundle bundleIdentifier], deviceToken);
    NSString *pushToken = [[[[deviceToken description]
                             stringByReplacingOccurrencesOfString:@"<" withString:@""] stringByReplacingOccurrencesOfString:@">" withString:@""]
                           stringByReplacingOccurrencesOfString:@" " withString:@""];
    NSLog(@"device token: %@",pushToken);
}
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
NSLog(@"Failed to register: %@", error);
}
```

然后只需要后台通过APNS发送一条JSON：

```
{
  "aps": {
    "alert": {
      "body": "你收到一个内容"
    },
    "badge": 1,
    "sound": "default"
  }
}
```

在App端就能收到一条如下的推送：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/1.jpg)

以上就是在iOS10我们实现基本推送功能的方式，至于之前推送的一些其他功能可以自行百度或者谷歌。

 

在iOS10，苹果引入了一个新的Framework：UserNotification.Framework，将之前的RemoteNotification和LocalNotification的进行了统一处理，这里主要是对于RemoteNotification。

首先是注册通知：

```
[[UNUserNotificationCenter currentNotificationCenter] requestAuthorizationWithOptions:(UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert)
completionHandler:^(BOOL granted, NSError * _Nullable error) {
    if (granted==YES) {
        NSLog(@"request authorization succeeded!");
        [application registerForRemoteNotifications];
    }else{
        NSLog(@"request authorization failed!");
        NSLog(@"Error:%@",error);
    }
}];
```

UNUserNotificationCenter是用于专门管理推送通知的类，通过requestAuthorizationWithOptions向系统请求推送权限，请求完成后会有一个block的回调，如果granted为YES则代表获取权限成功，然后通过[application registerForRemoteNotifications]注册通知，获取token方式与之前，然后使用后台发起推送，就能收到跟之前一样的推送了。

在iOS10之前，一条推送上只能显示一句话，但是在iOS10之后如果我们推送下面这条JSON：

```
{
  "aps": {
    "alert": {
      "title": "这是一个标题",
      "subtitle": "这是一个副标题",
      "body": "你收到一个内容"
    },
    "badge": 1,
    "sound": "default"
  }
}
```

那么你收到的推送将会是如下：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/2.jpg)

这里除了我们之前能看到的消息外，还额外增加了title（标题）和subtitle（副标题）的显示，当然如果还想有更复杂的推送显示，在iOS10中也可以通过Notification Service Extension实现，但不在这篇文章介绍。

有时我们需要在点击了推送进入app时根据推送内容进行对应的操作，在iOS10之前在UIApplicationDelegate提供了对应的处理方法，那么如果我们使用UserNotification.Framework又该如何实现这个功能，这里我们需要使用到UNUserNotificationCenterDelegate，UNUserNotificationCenterDelegate提供了两个代理方法，分别为：

```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler
```

第一个代理方法是当应用处于前台时，收到推送时响应的代理方法，第二个代理方法是在点击了推送或者点击了推送的Action会响应的代理方法。

当然我们首先需要在注册推送的时候添加如下代码：

```
[[UNUserNotificationCenter currentNotificationCenter]setDelegate:self];
```

首先介绍第一个方法，当我们的程序正处于前台运行时候，这时候如果服务端向app发送了一个推送，那么我们的app将会响应到- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler，这里我们可以看到该方法有三个参数：

第一个center就是我们注册推送使用的UNUserNotificationCenter。  

第二个参数notification就是我们收到的推送对象UNNotification，这里系统将推送信息整理成一个对象给我们处理，比以前直接传递一个Dictionary要有好很多，UNNotification中的个字段的含义就不一一说明了。

第三个参数completionHandler是一个block，在iOS10以前如果当系统App处于前台时收到推送，系统只会向App发出推送信息，但不会在界面上弹出推送提示语，但是现在我们可以使用completionHandler使我们处于前台时系统也会弹出推送提示信息，我们需要做的就是在这个代理方法中针对想要弹出提示框的推送执行如下代码：

```
completionHandler(UNNotificationPresentationOptionBadge | UNNotificationPresentationOptionSound | UNNotificationPresentationOptionAlert);
```
那么当我们App处于前台时系统也会弹出提示信息，如图：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/3.jpg)

这样我们App处于前台时也能让系统弹出推送的提示信息。
UNNotificationPresentationOptionBadge，UNNotificationPresentationOptionSound，UNNotificationPresentationOptionAlert分别代表红点，声音和提示语，大家可以自行测试其功能。

第二个代理方法- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler，当我们点击了推送的提示信息后，App将启动（如果App不处于前台）并执行该方法，这个代理方法也有三个参数：

第一个center就是我们注册推送使用的UNUserNotificationCenter。

第二个response我们点击推送后收到的UNNotificationResponse对象，UNNotificationResponse有两个变量，一个notification就是我们对应的UNNotification，另一个actionIdentifier是一个字符串，功能在后面说明。

第三个completionHandler是一个block，具体功能还在研究。
在该代理中我们添加如下代码：

```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler{
    NSString *lString=@"点击了通知";
    UIAlertController *lAlertController=[UIAlertController alertControllerWithTitle:lString message:nil preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *lOKAction=[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        
    }];
    [lAlertController addAction:lOKAction];
    [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:lAlertController animated:YES completion:nil];
    completionHandler();
}
```

当我们点击了推送之后，App将启动，并弹出一个Alert，效果如下：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/4.jpg)

在iOS8的时候苹果提供了UIUserNotificationCategory，UIUserNotificationAction等相关类，可以在当程序在后台收到推送，对应的按钮或者输入文字后进入App进行不同的操作，效果如下：
当我们后台发送如下的推送信息：

```
{
  "aps": {
    "alert": {
      "title": "这是一个标题",
      "subtitle": "这是一个副标题",
      "body": "你收到一个内容"
    },
    "badge": 1,
    "sound": "default",
    "category":"category.test"
  }
}
```

在收到推送后下拉这个推送：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/5.jpg)

点击confirm按钮：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/6.jpg)

如果点击text按钮：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/7.jpg)

点击send：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/8.jpg)

那么我们使用UserNotification.Framework，要如何实现该功能呢？  
首先我们需要在注册通知的时候同时注册NotificationCategoriy:

```
[self registerNotificationCategory];
```

registerNotificationCategory方法如下：

```
static NSString *kCategoryTestKey=@"category.test";
static NSString *kCategoryTestInputKey=@"category.test.input";
static NSString *kCategoryTestConfirmKey=@"category.test.confirm";
-(void)registerNotificationCategory{
    UNTextInputNotificationAction *lTextAction=[UNTextInputNotificationAction actionWithIdentifier:kCategoryTestInputKey title:@"text" options:UNNotificationActionOptionForeground textInputButtonTitle:@"send" textInputPlaceholder:@"please"];
    
    UNNotificationAction *lConfirmAction=[UNNotificationAction actionWithIdentifier:kCategoryTestConfirmKey title:@"Confirm" options:UNNotificationActionOptionForeground];
    
    UNNotificationCategory *lCategory=[UNNotificationCategory categoryWithIdentifier:kCategoryTestKey actions:@[lTextAction,lConfirmAction] intentIdentifiers:@[] options:UNNotificationCategoryOptionNone];
    [[UNUserNotificationCenter  currentNotificationCenter]setNotificationCategories:[NSSet setWithObjects:lCategory, nil]];
}
```

这里我们为UNUserNotificationCenter设置了一个identifier为category.test的category，这个category包含一个标题为text的文字输入的action（identifier为category.test.input），一个标题为confirm的按钮action（identifier为category.test.confirm）。

那么我们该如何处理各种action操作后需要执行的行为，这里我就要用到之前我们提到的UIApplicationDelegate中的第二个代理方法- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler，之前我们提到第二个参数response还有一个actionIdentifier属性，这个属性就代表我们注册的category中各自action的identifier，那么我修改代理方法如下：

```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler{
    NSString *lString=@"点击了通知";
    if ([response.actionIdentifier isEqualToString:kCategoryTestInputKey]) {
        UNTextInputNotificationResponse *inputResponse=(UNTextInputNotificationResponse *)response;
        lString=[NSString stringWithFormat:@"点击了input,输入内容为:%@",inputResponse.userText];
    }else if([response.actionIdentifier isEqualToString:kCategoryTestConfirmKey]){
        lString=@"点击了confirm";
    }
    UIAlertController *lAlertController=[UIAlertController alertControllerWithTitle:lString message:nil preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *lOKAction=[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        
    }];
    [lAlertController addAction:lOKAction];
    [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:lAlertController animated:YES completion:nil];
    completionHandler();
}
```

就可以实现前面同样的功能。

代码下载：[UserNotificationsTest](https://github.com/MingleChang/UserNotificationsTest.git) 

