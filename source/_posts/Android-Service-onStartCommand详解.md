---
title: Android-Service-onStartCommand详解
date: 2021-02-01 11:34:55
tags: Service
categories: Android
---
# Android-Service(onStartCommand详解)

在上一篇我们总结了Android中的Service，接下来这篇就围绕着其中的一个生命周期方法`onStartCommand()`进行总结。

之前Service中的`onStart()`方法已经被废弃了，取而代之的是`onStartCommand()`方法，该方法有三个参数

```java
public int onStartCommand(Intent intent, int flags, int startId)
```

+ 第一个参数是启动过来的Intent信息，也就是调用者的Intent信息
+ 第二个参数flags代表着启动请求的附加参数，由系统传入
  + 通过`startService()`启动，flags传入0
  + `onStartCommand()`返回值为START_STICKY_COMPATIBILITY或者START_STICKY并且服务被强制杀死时重启后，flags传入START_FLAG_REDELIVERY（1）
  + `onStartCommand()`返回值为START_REDELIVER_INTENT并且服务被强制杀死重启后，flags传入START_FLAG_REDELIVERY（2）
+ 第三个参数为启动的ID，每次启动都对应不同的ID，与`stopSelf()`连同停止Service

**执行时机**

当通过`startService()`启动时，多次执行`startService()`，但是生命周期函数`onCreate()`只会执行一次，而`onStartCommand()`会执行多次，当通过`bindService()`启动Service时，不会执行`onStartCommand()`方法。

**返回值与自启动机制**

当Service被强行停止的时候，例如使用kill命令强行关闭，通过配置参数可以让Service重新启动

`onStartCommand()`不同的返回值又对Service重启有不同的影响（上述的参数配置）

+ START_STICKY
  + onStartCommand默认的返回值，表示Service被杀掉后会重新启动，但是不会携带之前的Intent信息
  + 例如我们的系统可以接受在任意时刻被杀死，并且重启的时候不需要Intent信息，就可以用此返回值，例如播放音乐

+ START_NOT_STICKY
  + Service被kill后，不会进行重启
  + 适用于在Service中执行一些无关紧要的工作，被杀掉了也无需关心

+ START_REDELIVER_INTENT
  + Service被杀掉后，会进行重新启动，并且还会携带之前的Intent信息
  + 适用于对Intent有强依赖性的Service，重启后必须获得之前的Intent信息

> 在小米手机上，必须为app手动开始自启动权限，否则被杀掉的Service不管在onStartCommand中设置什么返回值都不会重启

## 参考链接

+ [https://blog.csdn.net/qq_20328181/article/details/79353616](https://blog.csdn.net/qq_20328181/article/details/79353616)
+ [https://blog.csdn.net/daisyjingjinglj/article/details/77837933](https://blog.csdn.net/daisyjingjinglj/article/details/77837933)
+ [http://jerey.cn/2017/01/10/Service%E8%A2%AB%E6%84%8F%E5%A4%96%E6%9D%80%E6%AD%BB%E4%BC%9A%E8%87%AA%E5%8A%A8%E9%87%8D%E5%90%AF%E4%B9%88/](http://jerey.cn/2017/01/10/Service%E8%A2%AB%E6%84%8F%E5%A4%96%E6%9D%80%E6%AD%BB%E4%BC%9A%E8%87%AA%E5%8A%A8%E9%87%8D%E5%90%AF%E4%B9%88/)
