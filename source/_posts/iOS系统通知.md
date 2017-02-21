---
title: iOS系统通知
date: 2017-02-21 20:19:22
tags: [iOS, Notification]
---
## 一、键盘

1.UIKeyboardWillShowNotification //将要弹出键盘  
2.UIKeyboardDidShowNotification //显示键盘  
3.UIKeyboardWillHideNotification //将要隐藏键盘  
4.UIKeyboardDidHideNotification //键盘已经隐藏  
5.UIKeyboardWillChangeFrameNotification //键盘将要改变frame  
6.UIKeyboardDidChangeFrameNotification //键盘已经改变frame  

## 二、窗口

1.UIWindowDidBecomeVisibleNotification //当window激活时并展示在界面的时候触发，返回空  
2.UIWindowDidBecomeHiddenNotification //当window隐藏的时候触发，返回空  
3.UIWindowDidBecomeKeyNotification //当window被设置为keyWindow时触发，返回空  
4.UIWindowDidResignKeyNotification //当window的key位置被取代时触发，返回空  

## 三、程序消息

1.UIApplicationDidEnterBackgroundNotification //程序进入后台  
2.UIApplicationDidEnterBackgroundNotification //程序进入前台  
3.UIApplicationDidFinishLaunchingNotification //程序加载完成  
4.UIApplicationDidBecomeActiveNotification //程序变成活动状态  
5.UIApplicationWillResignActiveNotification //程序变为非活动状态  
6.UIApplicationDidReceiveMemoryWarningNotification //内存警告  
7.UIApplicationWillTerminateNotification //程序进程停止  
8.UIApplicationSignificantTimeChangeNotification //重要的时间变化(新的一天开始或时区变化)   
9.UIApplicationWillChangeStatusBarOrientationNotification //将要改变状态栏方向  
10.UIApplicationDidChangeStatusBarOrientationNotification //状态栏的方向改变  
11.UIApplicationWillChangeStatusBarFrameNotification //将要改变状态栏的frame  
12.UIApplicationDidChangeStatusBarFrameNotification //状态栏的frame改变  
13.UIApplicationBackgroundRefreshStatusDidChangeNotification  
14.UIApplicationProtectedDataWillBecomeUnavailable  
15.UIApplicationProtectedDataDidBecomeAvailable  

## 四、设备

1.UIDeviceOrientationDidChangeNotification //设备方向改变  
2.UIDeviceBatteryStateDidChangeNotification //设备电池状态改变  
3.UIDeviceBatteryLevelDidChangeNotification //设备电池电量改变  
4.UIDeviceProximityStateDidChangeNotification //设备近距离传感器  

## 五、屏幕

1.UIScreenDidConnectNotification //屏幕设备连接  