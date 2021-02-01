---
title: Android的启动方式
date: 2021-01-25 19:33:56
tags: Android
categories: Android
---
# Android的启动方式

Android进程的启动方式有三种，分别为冷启动，热启动与温启动

**冷启动**主要指在缓存进程中没有当前启动进程的信息，例如第一次启动进程或者将进程kill掉了重新启动，冷启动的时间会长一点

**温启动**为应用已经启动了，但是通过返回键退出了，此时再次启动这个进程就为温启动

**热启动**代表的就是应用已经启动，但是应用通过Home键返回，再次启动进程就为热启动，热启动花费的时间最少，但是系统开销最大

我们当然希望手机上的进程都为热启动，这样能有很好的用户体验，但是热启动也是要付出内存的代价的，大量缓存的进程在后台会造成严重的内存积压，当内存不可用的时候，就会触发`LowMemoryKiller`机制进行内存回收，关于[LowMemoryKiller](https://lightingsui.github.io/2021/01/21/Android%E4%B8%AD%E7%9A%84LowMemoryKiller%E6%9C%BA%E5%88%B6/)机制可以查看之前总结的一篇博客。

可以通过`adb`指令查看冷启动、温启动与热启动耗费的时间

```bash
adb shell am start -W xxxxxxx/xxxxxxx.activity
```

以启动相机为例子，可以通过日志来抓取相机主Activity

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210125184202.png)

```bash
adb shell am start -W com.android.camera/.Camera
```

**冷启动**

```bash
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.camera/.Camera }
Status: ok
LaunchState: COLD
Activity: com.android.camera/.Camera
TotalTime: 1120
WaitTime: 1125
Complete
```

**热启动**

```bash
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.camera/.Camera }
Warning: Activity not started, its current task has been brought to the front
Status: ok
LaunchState: HOT
Activity: com.android.camera/.Camera
TotalTime: 146
WaitTime: 173
Complete
```

**温启动**

```bash
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.camera/.Camera }
Status: ok
LaunchState: WARM
Activity: com.android.camera/.Camera
TotalTime: 725
WaitTime: 732
Complete
```

TotalTime：自己的所有Activity的启动耗时

WaitTime：AMS启动的Activity总耗时


