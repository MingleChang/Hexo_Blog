---
title: iOS10 Notification Content Extension
date: 2016-12-24 11:13:56
tags: [iOS10, Notification]
---
iOS10对推送通知的增加了两个扩展框架，之前介绍了Notification Service Extension，允许在收到推送之后，通知展示之前对推送信息进行二次处理；而另一个就是Notification Content Extension，允许开发者对推送信息自定义一个展示界面，在这个界面里你可以自定义任何视图，但是有一个限制，这个界面不能有用户交互，也就是这个界面用户不能点击它。但是对于整个通知我还是可以继续使用actions进行交互。
首先我们介绍下这个推送界面的组成，当我收到一条新的推送，我们下拉这条推送就会出现下面如图界面：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/1.jpg)

Header是属于系统默认的部分，所有推送都有这个Header且不可修改

Custom Content就是我们要介绍的Notification Content Extension的内容，在这里你可以自定义为任何样式。

Default Content系统默认的推送展示样式，不可修改，但可以选择不显示（这个我们后面会说到）。

Actions就是我们之前介绍UserNotification.Framework是介绍过的，这里用户可以进行一些操作并体现到Custom Content中。

现在我们来创建一个Notification Content Extension的Target，Xcode会自动创建如下图的几个文件：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/2.png)

NotificationViewController就是我们Custom Conten的代码部分，MainInterface.storyboard就是布局部分，Info.plist就是配置文件。

打开Info.plist文件，我们可以看到如下内容：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/3.jpeg)

这里的UNNotificationExtensionCategory就是响应这个Content Extension的通知的categoryId，那如果有多个categoryId都对应的是这个Content Extension怎么办呢？那么我们可以将UNNotificationExtensionCategory改为一个数组，数组中包含多个categoryId，如图：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/4.jpg)  

UNNotificationExtensionInitialContentSizeRatio这个是Content Extension初始化时候的高宽比。

除了这两个我之前说过Default Content可以选择是否显示，那么如果我们希望不显示Default Content，我们可以在NSExtensionAttributes增加UNNotificationExtensionDefaultContentHidden并设置为YES，如图：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/5.jpg) 

然后我们在MainInterface.storyboard中的ViewController中增加一些子视图：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/6.jpg)  

一个imageView和两个Label，并将其关联为NotificationViewController的属性：  

```
@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;
@property (weak, nonatomic) IBOutlet UILabel *contentLabel;
```

现在我们开始处理NotificationViewController，NotificationViewController实际上就是一个继承于UIViewController的一个视图控制器，但是他实现了UNNotificationContentExtension协议。
UNNotificationContentExtension协议主要有两个方法：

```
- (void)didReceiveNotification:(UNNotification *)notification;
@optional
- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption option))completion;
```

第一个方法是当NotificationContentExtension收到指定categoryId的推送时，那么就将会响应这个方法，然后我们可以根据通知内容设置我们的界面。  
第二个方法是一个可选方法，这个就是当用户进行Actions的操作是，NotificationContentExtension会响应的方法，我们可以根据对应的Action修改NotificationContentExtension界面或者进行网络请求等操作。  
我先根据我们收到推送修改对应的Content Extension界面，那么我们在didReceiveNotification：中添加如下代码：

```
- (void)didReceiveNotification:(UNNotification *)notification {
    self.imageView.image=[UIImage imageNamed:@"push_image"];
    self.titleLabel.text=notification.request.content.title;
    self.contentLabel.text=notification.request.content.body;
}
```

那么当我们收到推送之后，我们的Content Extension界面界面将会展示如下：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/7.jpg)  

之前在介绍Service Extension时，我们可以根据推送请求一些图片等并设置到attachments，那么当我们在Content Extension中收到一个包含attachments的推送时，我们要如何展示，这里我们将代码如下修改：  

```
- (void)didReceiveNotification:(UNNotification *)notification {
    if (notification.request.content.attachments.count==0) {
        self.imageView.image=[UIImage imageNamed:@"push_image"];
    }else{
        UNNotificationAttachment *lAttachment=notification.request.content.attachments.firstObject;
        if (lAttachment) {
            if ([lAttachment.URL startAccessingSecurityScopedResource]) {
                self.imageView.image = [UIImage imageWithContentsOfFile:lAttachment.URL.path];
                dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                    [lAttachment.URL stopAccessingSecurityScopedResource];
                });
            }
        }
    }
    self.titleLabel.text=notification.request.content.title;
    self.contentLabel.text=notification.request.content.body;
}
```

这里如果我们收到的推送包含attachments内容，那么我们就将图片赋值给imageView，但是attachment是由系统管理的，系统会把它们单独的管理，这意味着它们存储在我们sandbox之外。所以这里我们要使用attachment之前，我们需要告诉iOS系统，我们需要使用它，并且在使用完毕之后告诉系统我们使用完毕了。对应上述代码就是-startAccessingSecurityScopedResource和-stopAccessingSecurityScopedResource的操作。
这是如果我们收到一个带图片的推送，Content Extension展示如下：  

![](https://github.com/MingleChang/img/raw/master/2016/12/24/8.jpg)   

这里的imageView展示的就是在Service Extension下载的图片。

然后我们来介绍UNNotificationContentExtension协议的第二个方法，- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption option))completion
该方法有两个参数，第一个response就是点击Actions传递过来的参数，我们可以根据response来判断是点击了哪个Actions，并对其做对应的处理，第二个completion是一个block，当我处理完response后需要回调的block，系统根据该block做后续处理，这个block有一个参数UNNotificationContentExtensionResponseOption，这个是UNNotificationContentExtensionResponseOption是一个枚举，定义如下：  

```
typedef NS_ENUM(NSUInteger, UNNotificationContentExtensionResponseOption) {
    UNNotificationContentExtensionResponseOptionDoNotDismiss,
    UNNotificationContentExtensionResponseOptionDismiss,
    UNNotificationContentExtensionResponseOptionDismissAndForwardAction,
} __IOS_AVAILABLE(10_0) __TVOS_UNAVAILABLE __WATCHOS_UNAVAILABLE __OSX_UNAVAILABLE;
```

UNNotificationContentExtensionResponseOptionDoNotDismiss代表不关闭Content Extension
UNNotificationContentExtensionResponseOptionDismiss代表关闭Content Extension，这里需要注意如果点击的action类型为UNNotificationActionOptionForeground，Content Extension仍然不会关闭
UNNotificationContentExtensionResponseOptionDismissAndForwardAction代表关闭Content Extension，且打开App。  
这里我们修改代码如下：

```
- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption option))completion{
    if ([response.actionIdentifier isEqualToString:kCategoryTestInputKey]) {
        UNTextInputNotificationResponse *inputResponse=(UNTextInputNotificationResponse *)response;
        NSString *lString=inputResponse.userText;
        self.contentLabel.text=lString;
    }else if ([response.actionIdentifier isEqualToString:kCategoryTestConfirmKey]) {
        self.contentLabel.text=@"Confirm";
    }
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        completion(UNNotificationContentExtensionResponseOptionDismiss);
    });
    
}
```

这里我们根据对应的Action修改contentLabel的显示，并在1.5s后关闭Content Extension，当然你也可以在这里进行一些网络请求后在执行block。  
那么我们在收到推送点击了Confirm后，Content Extension将修改为如下：  
![](https://github.com/MingleChang/img/raw/master/2016/12/24/9.jpg)  

对于Content Extension的介绍就如下了。

代码下载：[UserNotificationsTest](https://github.com/MingleChang/UserNotificationsTest.git)
