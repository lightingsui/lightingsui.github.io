---
title: Android-Service详解
date: 2021-01-27 19:32:50
tags: Android
categories: Android
---
## Android-Service详解

Service是安卓的四大组件之一，在我们的日常开发中经常会使用到，例如执行一些后台操作，播放音乐、下载等等。

接下来我们就从一个简单的Service的使用开始，一步一步的深入介绍Service

## Service简单使用

创建一个Activity，在Activity中设置两个按钮，用于启动、关闭Service，而在Service中只是打印一些生命周期的Log，并不去做什么事。

**MainActivity**

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private final static String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Button startService = findViewById(R.id.btn_start_service);
        Button stopService = findViewById(R.id.btn_stop_service);
        
        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        Intent intent = new Intent(MainActivity.this, MyService.class);
        switch (v.getId()) {
            case R.id.btn_start_service:
                startService(intent);
                break;
            case R.id.btn_stop_service:
                stopService(intent);
                break;
            default:
                // fall through
        }
    }
}
```

**activity_main**

```xml
<?xml version="2.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn_start_service"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="开启服务"/>

    <Button
        android:id="@+id/btn_stop_service"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="关闭服务"/>

</LinearLayout>
```

**MyService**

```java
public class MyService extends Service {

    private final static String TAG = MyService.class.getSimpleName();

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @RequiresApi(api = Build.VERSION_CODES.O)
    @Override
    public void onCreate() {
        Log.e(TAG, "service onCreate");
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e(TAG, "service onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        Log.e(TAG, "service onDestroy");
        super.onDestroy();
    }
}
```

最后别忘了在AndroidManifest中注册Service

使用一个Service的步骤

+ 创建一个类，继承Service
+ 在AndroidMainfest中注册Service
+ 在Activity中使用`startService`启动Service

注意我这里使用的是显示启动Service，如果要隐式启动Service，则需要在Intent中指定包名和action，步骤如下

```java
intent = new Intent();
intent.setAction("com.sui.SERVICE");
intent.setPackage("com.sui.testapplication");
startService(intent);
```

不允许直接使用action去启动Service是从Android 5开始，即Android Lollipop

这时我们通过点击启动服务的按钮即可打开Service，看到打印的日志。

如果想要停止服务，则可以使用`stopService()`传入Intent即可，注意，开启和结束Service的Intent必须为同一个Intent

## Service绑定

上面的开启服务方式和调用`startService`的Activity一点关系都没有，使用`startService`只是告诉Service一声你可以启动了。如果想要将Activity和Service进行关联，则需要使用`Context.bindService(Intent intent, ServiceConnection conn, int flags)`

+ 第一个参数的为传入的Intent
+ 第二个参数为connect的状态回调
+ 第三个参数为绑定关系

**使用bindService**

```java
 bindService(intent, serviceConnection, BIND_AUTO_CREATE);
```

intent和上面的一样，只不过这里我们需要去定义一下第二个参数

```java
ServiceConnection serviceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {

    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
};
```
实现了一个匿名内部类，该内部类中有两个必须实现的函数，也是非常重要的函数，即`onServiceConnected`与`onServiceDisconnected`

+ `onServiceConnected`表示当Activity与Service建立绑定成功时候的回调函数
+ `onServiceDisconnected`表示当Activity与Service断开连接时候的回调函数

`onServiceConnected`方法有一个非常重要的参数`IBinder service`，这个参数是绑定成功后Service返回的数据，还记不记得我们在Service中必须重写的方法

```java
public IBinder onBind(Intent intent) {
        return null;
}
```

`IBinder service`参数就是接受来自`onBind`的返回信息

如果想要返回消息，则需要创建一个类，继承自Binder

```java
// Service文件中
public static class MyBinder {
    int addPlus(int x, int y) {
        return x + y;
    }
}

public IBinder onBind(Intent intent) {
        return new MyBinder();
}

// Activity文件中
ServiceConnection serviceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
		MyService.MyBinder binder = (MyService.MuBinder)service;
        
         int res = binder.addPlus(1, 1);
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
};
```

连接成功后即可在`serviceConnection`中的`onServiceConnected`中执行`addPlus`方法

> 这里需要注意一下：如果在Service中的onBind方法中没有返回任何信息，即null，那么回调方法`onServiceConnected`与`onServiceDisconnected`就不会执行，即使Service与Activity绑定成功了也不会执行

第三个参数flags，他有一个特殊的值，`Context.BIND_AUTO_CREATE`。这个值特殊的地方就在于当我们绑定Service时，如果Service还未开启，`bindService`就会开启Service，注意，**这时会执行Service的全部与新建相关的生命周期函数**

解绑服务只需使用`unbindService`即可，参数传入与`bindService`同一定义的`ServiceConnectiong`

## Service的生命周期

Service的生命周期主要有`onCreate`、`onStartCommand`、`onDestroy`

当我们第一次启动Service的时候，会先执行`onCreate` -> `onStartCommand`

当我们一直在执行`startService`时，`onCreate`只会执行一次，而`onStartCommand`会每次都执行。

当我们使用`bindService`的时候，`onCreate`和`onStartCommand`都不会执行（如果指定了BIND_AUTO_CREATE时且没有创建Service则会都执行）

最后我们使用`stopService`的时候，会执行`onDestroy`方法

> 如果我们在开启服务后，又使用`bindService`绑定Service，这时我们再关闭服务（`stopService`），可以看到`onDestroy`是不会执行的（即Service不会关闭），只有当前服务的所有绑定都解绑了（所有的绑定执行了`unbindService`），然后使用`stopService`才会销毁Service

## 跨进程Service

Service可以单独放在一个进程之中，只需要在AndroidMainfest文件中指定`android:process`即可

```xml
<application>
	......
    <service android:process=":remote" android:name=".MyService" />
    ......
</application>
```

可以在另一个进程的Activity中开启其他进程的Service，还是使用`startService`即可，关闭也可以使用`stopService`。

如果使用`bindService`绑定其他进程的Service，这就涉及到唤醒Service了，需要使用隐式启动Service，显示启动无法获取到类的字节码信息（也就无法传入Intent）。

```java
intent = new Intent();
intent.setAction("com.sui.SERVICE");
intent.setPackage("com.sui.testapplication");
bindService(intent, connect, BIND_AUTO_CREATE);
```

此时如果想要使用binder进行通信，可以使用Android的跨进程通信aidl，创建一个aidl文件

```java
package com.sui.mutiprocessapplication;
interface IMyService {
    int addPlus(int x, int y);
}
```

有以下几个注意点：

+ interface 前不要加public
+ 函数的返回值前也不要加访问限制修饰符

Android Studio中将项目重新rebuild即可（Build -> Rebuild Project），即可生成对应的接口文件

一切准备就绪了，下面我们可以在Service的`onBind`方法中返回**实例化自定义aidl接口的类**了

```java
public iBinder onBind(Intent intent) {
    return myBinder;
}

private IMyService.Stub.myBinder = new IMyService.Stub() {
    
    // 实现aidl接口中的自定义函数
    @Override
    public int addPlus(int x, int y) {
        return x + y
    }
}
```

在Activity中的`ServiceConnect`中调用`addPlus`

```java
ServiceConnect connect = new ServiceConnection() {
    @Override
    public void onServiceConnection(ComponentName name, IBinder service) {
        IMyService binder = IMyService.Stub.asInterface(service);
        
        int res = binder.addPlus(1, 2);
    }
    
    @Override
    public void onServiceDisconnected(ComponentName name) {
        
        
    }
}
```

以上我们就完成了一次跨进程的Activity与Service之间的通信。

## 关于Service的一些问题

**1.Service与Thread有什么区别？**

在Service中不要进行一些耗时的操作，Service如果没有单独的开启一个进程，那么它是运行在主线程中的，即UI线程，执行耗时的操作可能会引发ANR，引入Service的目的是为了执行一些后台的操作，例如下载播放音乐等。如果在Service中执行耗时的操作，应该在其中开启一个线程。

关于Service的使用会在以后的操作过程中去慢慢更新，当前时间2021年1月27日

## 参考链接

+ [https://blog.csdn.net/rongwenbin/article/details/38274541](https://blog.csdn.net/rongwenbin/article/details/38274541)
+ [https://blog.csdn.net/qq_30993595/article/details/78452064](https://blog.csdn.net/qq_30993595/article/details/78452064)
+ [https://blog.csdn.net/yuzhiqiang_1993/article/details/78211385](https://blog.csdn.net/yuzhiqiang_1993/article/details/78211385)
+ [https://blog.csdn.net/pvpheroszw/article/details/77981553](https://blog.csdn.net/pvpheroszw/article/details/77981553)
+ [https://blog.csdn.net/guolin_blog/article/details/9797169](https://blog.csdn.net/guolin_blog/article/details/9797169)




