---
title: iOS10 Notification Service Extension
date: 2016-12-21 22:32:49
tags: [iOS10, Notification]
---

在iOS10之前，iOS的推送逻辑是服务器想苹果的APNS服务器发送一条消息，然后由APNS服务器推送到手机，然后由操作系统处理后直接展示给用户，这个过程如下：

服务器 → APNS → 操作系统 → 用户

可以看出，这个过程跟我们的App没有任何关系（除了注册推送，获取Token），推送来的任何信息我们都无法对其展示做处理，iOS10苹果推出了Notification Service Extension，使得推送来的的信息可以通过Service Notification进行二次处理，那么现在我们推送发送到展示的过程就变成了：

服务器 → APNS → 操作系统 → Service Extension → 用户

通过Notification Service Extension，我们能在收到推送一条新的推送之后的30s（据说是30s，未测试）之内对推送信息进行二次处理，然后再展示，当然如果我们在规定时间之内未能成功进行二次处理，系统还是会按照当前的推送信息进行展示。

首先我对我们的项目创建一个Notification Service Extension，Notification Service Extension跟以前的Today Extension一样都属于一个应用扩展，那么就需要创建一个Target：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/9.jpg)

然后如图选择：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/10.jpg)

点击Next，输入名称，我们在项目中就多出一个新的target：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/11.jpg)

这里Bundle Identifier就是项目的bundle id加上.扩展名称，所以我这里的Bundle Identifier为mingle.chang.joke.NotificationService，这里我们就创建一个Notification Service Extension。

接下来我来看下Notification Service Extension中的处理逻辑，Notification Service Extension为我们创建三个文件：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/12.jpg)

Info.plist就是Notification Service Extension的配置文件，而NotificationService.h和NotificationService.m则是我们对通知进行二次处理的地方，打开NotificationService.m可以看到，已经系统已经默认帮我们写好了两个方法：

```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];

    self.contentHandler(self.bestAttemptContent);
}

- (void)serviceExtensionTimeWillExpire {
    // Called just before the extension will be terminated by the system.
    // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
    self.contentHandler(self.bestAttemptContent);
}
```

而我们需要做的也就是在这两个方法中进行处理，第一个方法- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler就是当Service Extension收到推送时首先执行的方法，第二个方法- (void)serviceExtensionTimeWillExpire就是如果我们对推送的二次处理超时或者处理出现异常情况将会默认执行这个方法，所以对于我们来说主要对第一个方法进行修改，在之前如果我们收到这样一条推送：

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

那么我们收到的推送将会是如下：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/13.jpg)

我们说过Notification Service Extension可以对推送进行二次处理之后在进行展示，那么需要我们做哪些处理呢？  
首先我们需要后台在推送的JSON中增加一个mutable-content字段，且该字段的值为1，那么我们服务器发出的推送就会是下面这个JSON：

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
    "mutable-content": 1
  }
}
```

只有推送中包含该字段，系统才会将推送发送给Service Extension进行二次处理，然后我们修改- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler中的代码如下：

```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    // 根据收到的推送request修改推送显示的信息
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.title];
    self.bestAttemptContent.subtitle = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.subtitle];
    self.bestAttemptContent.body = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.body];
    
    self.contentHandler(self.bestAttemptContent);
}
```

首先我们介绍这个方法中的两个参数：  
request：就是我们收到的推送请求  
contentHandler：是我们对推送进行二次处理完成后的推送信息回调给系统或者通知中心的block  
接下介绍该方法中的代码：  
首先我们通过：  

```
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
```

将contentHandler和request.content赋值给属性contentHandler和bestAttemptContent；
然后我们通过：

```
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.title];
    self.bestAttemptContent.subtitle = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.subtitle];
    self.bestAttemptContent.body = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.body];
```

修改bestAttemptContent的title，subtitle和body；
最后我们通过：

```
self.contentHandler(self.bestAttemptContent);
```

将修改后的bestAttemptContent回调给系统或者通知中心，这样当我们收到推送信息：

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
    "mutable-content": 1
  }
}
```

之后，系统展示的就会是：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/14.jpg)

可以看到，这里我们将收到的推送的title，subtitle和body都增加了[NotificationService]进行展示。  
之前说过系统留给我们处理推送的时间是30s，而我们上面的处理估计连1s都不到，那么我们在这30s还能干点其他什么吗？  
当然，这里留给我们处理足够长，我们能够处理很多东西，比如可以让服务器推送一段加密的信息，我们将信息解密之后在进行展示；又比如可以让服务器推送一条信息的唯一标识，然后我们通过唯一标识向服务器获取需要展示的信息；我们也可以在收到推送后向服务器下载图片，视频，语音进行展示，当然这些文件也有一些要求规定，如图：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/15.png)

这里我们以下载图片为例：  
首先我们修改推送JSON，在推送JSON增加一个自定义的字段image，这个字段就是对应的图片的地址，这里我们收到推送JSON则如下：

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
    "mutable-content": 1,
    "image": "xxxxx"
  }
}
```

然后修改- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler中的代码如下：

```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.title];
    self.bestAttemptContent.subtitle = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.subtitle];
    self.bestAttemptContent.body = [NSString stringWithFormat:@"%@ [NotificationService]", self.bestAttemptContent.body];
    
    NSDictionary *lApsDic = self.bestAttemptContent.userInfo[@"aps"];
    NSString *lImageUrl=lApsDic[@"image"];
    if (lImageUrl.length>0) {
        [self loadAttachmentForUrlString:lImageUrl withType:@"png" completionHandle:^(UNNotificationAttachment *attach) {
            if (attach) {
                self.bestAttemptContent.attachments = [NSArray arrayWithObject:attach];
            }
            self.contentHandler(self.bestAttemptContent);
        }];
    }else{
        self.contentHandler(self.bestAttemptContent);
    }
}
```

这里我们在之前修改body的后增加了图片下载和加入通知信息的代码，首先我们从推送信息中获取image字段，得到图片的链接地址，然后使用loadAttachmentForUrlString:withType:completionHandle:下载我们的图片，该方法的代码如下：

```
- (void)loadAttachmentForUrlString:(NSString *)urlStr withType:(NSString *)type completionHandle:(void(^)(UNNotificationAttachment *attach))completionHandler{
    __block UNNotificationAttachment *attachment = nil;
    NSURL *attachmentURL = [NSURL URLWithString:urlStr];
    
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSURLSessionDownloadTask *lTask=[session downloadTaskWithURL:attachmentURL completionHandler:^(NSURL *temporaryFileLocation, NSURLResponse *response, NSError *error) {
        if (error != nil) {
            NSLog(@"Error:%@", error.localizedDescription);
        } else {
            NSFileManager *fileManager = [NSFileManager defaultManager];
            NSURL *localURL = [NSURL fileURLWithPath:[temporaryFileLocation.path stringByAppendingPathExtension:type]];
            [fileManager moveItemAtURL:temporaryFileLocation toURL:localURL error:&error];
            
            NSError *attachmentError = nil;
            attachment = [UNNotificationAttachment attachmentWithIdentifier:@"" URL:localURL options:nil error:&attachmentError];
            if (attachmentError) {
                NSLog(@"Error:%@", attachmentError.localizedDescription);
            }
        }
        completionHandler(attachment);
    }];
    [lTask resume];
}
```

这个方法就是用于图片下载，并将下载的图片生成一个UNNotificationAttachment对象，然后通过block回调给- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler中，然后使用self.bestAttemptContent.attachments = [NSArray arrayWithObject:attach];对bestAttemptContent.attachments进行赋值，最后执行self.contentHandler(self.bestAttemptContent);通过这样，当我们收到上面的推送之后展示给用户的就会是如图所示：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/16.jpg)

下拉推送信息将会展示为：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/17.jpg)

这样我们就成功的推送了一条图片信息。

需要注意的地方：
一、是生成UNNotificationAttachment的时候，对应的图片文件名必须有正确的文件后缀名，否则在生成UNNotificationAttachment时将会抛出如下错误：
2016-10-30 23:08:17.135250 NotificationService[1581:190750] Error:Unrecognized attachment file type
二、如果我们需要对Notification Service Extension进行调试，需要选中Notification Service Extension的Target进行调试，如图：

![](https://github.com/MingleChang/img/raw/master/2016/12/21/18.jpg)

三、如果在Notification Service Extension中的网络请求不是HTTPS，那么必须该Target的Info.plist中添加App Transport Security Settings说明。

代码下载：[UserNotificationsTest](https://github.com/MingleChang/UserNotificationsTest.git)