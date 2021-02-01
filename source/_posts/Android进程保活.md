---
title: Android进程保活
date: 2021-01-28 14:48:39
tags: Android
categories: Android
---
# Android进程保活

目前的app都想自己一直被运行，不想被杀死，从而来获取用户的信息，了解用户的互联网习性。

国内在这一点就做的很到位，每个app几乎都做了保活处理，尤其是类似qq、微信IM类的软件，他们要保证及时的提醒用户有新消息到来，所以不能被杀死。目前国内的手机也针对这样进程保活做了一些反应对处理，他们加入了禁止自启动的机制等，来控制app的权限。同时也设有白名单，白名单中的app会被放松权限，做到自启动。所以要想做到app完全的保活，就需要和国内的手机厂商进行交涉，将我们的app放入白名单中，但是这可不是那么容易的，和大厂谈判是需要资本的，一些小的app厂商是没有资格谈判的。

下面列出了我所知道的进程保活的方法，如果大家有其他的方法，可以指教我一下

## 通过广播保活

当进行拍照、录像、开机等一些列的动作会产生广播，app就是通过接收到这些广播唤醒进程。

假如我们在拍照的时候，点击拍照按钮，这时一个app监听着拍照的广播，结果收到了广播后，例如唤醒了Service的进程，在后台开始进行下载动作，这时多么流氓的行为

从Android N开始，google可能已经意识到了这种问题的存在，所以取消了ACTION_NEW_PICTURE（拍照），ACTION_NEW_VIDEO（拍视频），CONNECTIVITY_ACTION（网络切换）广播。

## 同系拉起

同一个公司的app会相互拉起，例如阿里系的app，打开淘宝的时候会唤醒支付宝的服务进程，另外，使用其他公司的SDK也可能会唤醒其公司的产品，例如使用微信的SDK，可能会唤醒微信进程

> 上面所说的阿里系app会相互唤醒完全是一个例子，并不是真实的，包括举例的微信SDK会唤醒微信也是假的

## 1像素Activity

1像素Activity的保活思想是利用的[LowMemoryKiller](https://lightingsui.github.io/2021/01/21/Android%E4%B8%AD%E7%9A%84LowMemoryKiller%E6%9C%BA%E5%88%B6/)机制，基本思路是在手机锁屏后，app打开一个1像素透明的activity，将进程提升为可见进程，降低adj的值，避免被杀。当解锁的时候，将这个1像素的activity再关掉。这样用户完全感知不到，并且还提高了进程的优先级

```java
// 定义一个接收手机开屏锁屏的广播
public class ScreenBroadCastReceiver extends BroadcastReceiver {

    private final static String TAG = ScreenBroadCastReceiver.class.getSimpleName();

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        // 获取到自定义的工具类
        OneActivityManager instance = OneActivityManager.getInstance();

        if (Intent.ACTION_SCREEN_OFF.equals(action)) {
            Log.e(TAG, "receive screen_off broadcast");
            // 当接收到锁屏广播时打开1像素的Activity
            instance.startOnPixelActivity(context);

        } else if (Intent.ACTION_SCREEN_ON.equals(action)) {
            Log.e(TAG, "receive screen_on broadcast");
            // 当接收到开屏广播时关闭1像素的Activity
            instance.closeOnePixelActivity();
        }
    }
}
```

主要就是定义接收开屏以及锁屏时候的广播事件

```java
// 工具类
public class OneActivityManager {

    private OneActivityManager() {
    }

    private AppCompatActivity onePixelActivity;

    private final BroadcastReceiver broadcastReceiver = new ScreenBroadCastReceiver();

    public static OneActivityManager oneActivityManager = new OneActivityManager();

    public static OneActivityManager getInstance() {
        if (oneActivityManager == null) {
            oneActivityManager = new OneActivityManager();
        }

        return oneActivityManager;
    }

    public void setOnePixelActivity(AppCompatActivity activity) {
        onePixelActivity = activity;
    }

    // 打开1像素的Activity方法
    public void startOnPixelActivity(Context context) {
        Intent intent = new Intent(context, OnePixelActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
    }
    
	// 关闭1像素Activity的方法
    public void closeOnePixelActivity() {
        if (onePixelActivity != null) {
            onePixelActivity.finish();
        }
    }

    // 注册广播
    public void registerBroadCastReceiver(Context context) {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Intent.ACTION_SCREEN_ON);
        intentFilter.addAction(Intent.ACTION_SCREEN_OFF);

        context.registerReceiver(broadcastReceiver, intentFilter);
    }
	
    // 解绑广播
    public void unregisterBoardCastReceiver(Context context) {
        if(broadcastReceiver != null) {
            context.unregisterReceiver(broadcastReceiver);
        }
    }
}
```

```java
// 主activity
public class MainActivity extends AppCompatActivity {

    private final static String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		
        // 注册广播
        OneActivityManager.getInstance().registerBroadCastReceiver(this);
    }


    @Override
    protected void onDestroy() {
        // 在MainActivy销毁时解绑广播，防止Memory Leak
        OneActivityManager.getInstance().unregisterBoardCastReceiver(this);
        super.onDestroy();
    }
}
```

```java
// 自定义的1像素的Activity
public class OnePixelActivity extends AppCompatActivity {

    private final static String TAG = OnePixelActivity.class.getSimpleName();

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_one_pixel);

        Window window = getWindow();
        // 放在左上角
        window.setGravity(Gravity.TOP|Gravity.START);
        WindowManager.LayoutParams attributes = window.getAttributes();

        // 宽高为1px
        attributes.width = 1;
        attributes.height = 1;

        // 起始坐标
        attributes.x = 200;
        attributes.y = 0;
        window.setAttributes(attributes);


        Log.e(TAG, "create OnePixelActivity");
        OneActivityManager.getInstance().setOnePixelActivity(this);
    }


    @Override
    protected void onDestroy() {
        Log.e(TAG, "close OnePixelActivity");
        super.onDestroy();
    }
}
```

我用的是小米手机进行测试，测试的时候需要注意，在应用的权限里，必须打开自启动权限以及后台弹出界面权限以及锁屏显示权限，否则自己无法自动启动自己的Activity

测试一下，我们的应用在前台时，adj的值为

```bash
picasso:/ # cat /proc/29075/oom_score_adj
0
```

当锁屏时，adj的值也为0，如果在锁屏时不开启1像素的Activity，adj的值为

```bash
picasso:/ # cat /proc/31415/oom_score_adj
701
```

这样得adj的值就极容易被LMKD杀死。

> 另外注意的一点就是开启新的Activity只能在一个旧的Activity显示的时候开启，如果没有旧的Activity显示，那么1像素的Activity是无法开启的，adj的值同样很高

## 双进程拉活

首先就要注意一点，想要双进程拉活，需要开启应用的自启动权限，目前国内的手机都针对保活机制进行了一定的阻断，所以要自己开启这个权限才能测试

主要思路就是创建两个app，分别在两个app中创建一个Service，每个Service相互绑定，当一个Service被杀的时候，另一个进程的Service拉起这个被杀的Service。

第一个app中的Service

```java
public class MyService extends Service {

    private final static String TAG = MyService.class.getSimpleName();

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d(TAG, "remote service connected");
            IMyService2 myService2 = IMyService2.Stub.asInterface(service);

            try {
                int res = myService2.addPlus2(3, 4);
                System.out.println(res);
            } catch (RemoteException e) {
                e.printStackTrace();
            }

        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d(TAG, "remote service disconnected");
            Intent remoteIntent = new Intent();
            remoteIntent.setAction("com.sui.REMOTE");
            remoteIntent.setPackage("com.sui.mutiprocessapplication2");

            bindService(remoteIntent, connection, BIND_AUTO_CREATE);
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return myBinder;
    }

   private IMyService.Stub myBinder = new IMyService.Stub() {

       @Override
       public int addPlus1(int x, int y) {
           return x + y;
       }
   };


    @Override
    public void onCreate() {
        Log.e(TAG, "local service onCreate");
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e(TAG, "local service onStartCommand");
        Intent remoteIntent = new Intent();
        remoteIntent.setAction("com.sui.REMOTE");
        remoteIntent.setPackage("com.sui.mutiprocessapplication2");
        bindService(remoteIntent, connection, BIND_AUTO_CREATE);

        return START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
```

AndroidMainfest主要文件

```xml
<service android:exported="true" android:name=".MyService">
    <intent-filter>
        <action android:name="com.sui.LOCAL" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</service>
```
注意：命名第一个Service的进程名称为local（只是形式上的命名，并没有真实的指定Service进程的名称，Service还是在主进程中），并且设置 `android:exported`为true，否则其他绑定不了这个Service

第二个app中的Service

```java
public class MyService extends Service {

    private final static String TAG = MyService.class.getSimpleName();

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d(TAG, "local service connected");
            IMyService binder = IMyService.Stub.asInterface(service);

            try {
                int res = binder.addPlus1(1, 2);
                System.out.println(res);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d(TAG, "local service disconnected");
            Intent remoteIntent = new Intent();
            remoteIntent.setAction("com.sui.LOCAL");
            remoteIntent.setPackage("com.sui.mutiprocessapplication1");
            bindService(remoteIntent, connection, BIND_AUTO_CREATE);
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new IMyService2.Stub() {
            @Override
            public int addPlus2(int x, int y) throws RemoteException {
                return x + y;
            }
        };
    }

    @Override
    public void onCreate() {
        Log.e(TAG, "remote service onCreate");
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e(TAG, "remote service onStartCommand");
        Intent remoteIntent = new Intent();
        remoteIntent.setAction("com.sui.LOCAL");
        remoteIntent.setPackage("com.sui.mutiprocessapplication1");
        bindService(remoteIntent, connection, BIND_AUTO_CREATE);
        return START_NOT_STICKY;
    }
}
```

AndroidMainfest文件

```xml
<service android:exported="true" android:name=".MyService">
    <intent-filter>
        <action android:name="com.sui.REMOTE"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</service>
```
命名第二个进程的Service为remote（只是形式上的命名，并没有真实的指定Service进程的名称，Service还是在主进程中）

接下来观察一下现象，我们将两个app启动

```xml
# 第一个Service启动
01-28 11:59:11.165 E/MyService( 2780): local service onCreate
01-28 11:59:11.165 E/MyService( 2780): local service onStartCommand

# 第二个Service启动
01-28 11:59:18.491 E/MyService( 2838): remote service onCreate
01-28 11:59:18.491 E/MyService( 2838): remote service onStartCommand
```

当我们杀死第二个Service的时候

```bash
picasso:/ # ps -ef | grep com.sui
u0_a251       4728 28194 0 12:00:58 ?     00:00:00 com.sui.mutiprocessapplication1
u0_a252       4927 28194 0 12:01:15 ?     00:00:00 com.sui.mutiprocessapplication2
root          5892  5881 3 12:39:11 pts/2 00:00:00 grep com.sui
picasso:/ # kill -9 4927
picasso:/ # ps -ef | grep com.sui
u0_a251       4728 28194 0 12:00:58 ?     00:00:00 com.sui.mutiprocessapplication1
u0_a252       5894 28194 6 12:39:21 ?     00:00:00 com.sui.mutiprocessapplication2
root          5937  5881 0 12:39:23 pts/2 00:00:00 grep com.sui
picasso:/ #
```

可以看到mutiprocessapplication2又重启了，再查看一下日志

```xml
# Service2断开连接
01-28 12:42:32.914 D/MyService( 7030): remote service disconnected
01-28 12:42:32.921 I//vendor/bin/hw/vendor.qti.hardware.servicetracker@1.1-service(28202): size of service connections for service: com.sui.mutiprocessapplication1/.MyServiceafter removal is 0
01-28 12:42:32.922 W/ActivityManager(28320): Scheduling restart of crashed service com.sui.mutiprocessapplication2/.MyService in 1000ms
01-28 12:42:32.929 I//vendor/bin/hw/vendor.qti.hardware.servicetracker@1.1-service(28202): bindService is called for service : com.sui.mutiprocessapplication2/.MyService and for client com.sui.mutiprocessapplication1
01-28 12:42:32.929 I//vendor/bin/hw/vendor.qti.hardware.servicetracker@1.1-service(28202): total connections for service : com.sui.mutiprocessapplication2/
MyServiceare :1
01-28 12:42:32.953 D/Boost   (28320): hostingType=service, hostingName={com.sui.mutiprocessapplication2/com.sui.mutiprocessapplication2.MyService}, callerPackage=com.sui.mutiprocessapplication1, isSystem=false, isBoostNeeded=false.
01-28 12:42:32.953 I/ActivityManager(28320): Start proc 7115:com.sui.mutiprocessapplication2/u0a252 for service {com.sui.mutiprocessapplication2/com.sui.mutiprocessapplication2.MyService} caller=com.sui.mutiprocessapplication1

# Service2被Service1拉起
01-28 12:42:32.993 I//vendor/bin/hw/vendor.qti.hardware.servicetracker@1.1-service(28202): startService() is called for servicecom.sui.mutiprocessapplication2/.MyService
# Service2重新连接
01-28 12:42:33.084 E/MyService( 7115): remote service onCreate
01-28 12:42:33.087 D/MyService( 7030): remote service connected
```

被kill掉的进程再次启动，不会再执行onStartCommand方法，并且，如果一个进程被kill，另一个进程想要拉起，那么这个进程应该在前台，在后台是无法被拉起来的。

> 再次提示，一定要打开两个app的自启动权限，否则app一旦被杀掉则无法被拉起了

## 提升服务进程的等级

当我们开启有个服务的时候，可以将该服务通过setForeground提升为前台服务，从而降低进程的adj值，避免被杀，只不过将服务提升为前台服务后，会在通知栏显示通知，这样还是挺不友好的。但是像天气类的、音乐播放器类的软件可以这么做，在服务中实时的获取信息，在通知栏留下通知或者操作信息，便于用户操作。

```java
public class MyService extends Service {

    private final static String TAG = MyService.class.getSimpleName();

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    private IMyAidlInterface.Stub mBinder = new IMyAidlInterface.Stub() {
        @Override
        public int addPlus(int a, int b) throws RemoteException {
            return a + b;
        }

        @Override
        public void printString(String msg) throws RemoteException {
            Log.e(TAG, "print: " + msg);
        }
    };


    private String createNotificationChannel(String channelID, String channelNAME, int level) {
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            NotificationChannel channel = new NotificationChannel(channelID, channelNAME, level);
            manager.createNotificationChannel(channel);
            return channelID;
        } else {
            return null;
        }
    }



    @RequiresApi(api = Build.VERSION_CODES.O)
    private void setNotification() {

        Intent intent = new Intent(this, MyService.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);

        String channelId = createNotificationChannel("my_channel_ID", "my_channel_NAME", NotificationManager.IMPORTANCE_HIGH);
        NotificationCompat.Builder notification = new NotificationCompat.Builder(this, channelId)
                .setContentTitle("通知")
                .setContentText("收到一条消息")
                .setContentIntent(pendingIntent)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setPriority(NotificationCompat.PRIORITY_HIGH)
                .setAutoCancel(true);
        NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
        notificationManager.notify(100, notification.build());
        // 提升为前台服务
        startForeground(100, notification.build());
    }


    @RequiresApi(api = Build.VERSION_CODES.O)
    @Override
    public void onCreate() {
        Log.e(TAG, "service onCreate");
        super.onCreate();

        setNotification();

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

打开服务的时候，查看adj的值

```bash
picasso:/ # cat /proc/11793/oom_score_adj
200
```

关闭服务后，进程adj的值

```bash
picasso:/ # cat /proc/11793/oom_score_adj
925
```

在Android Api版本小于18的时候，Android系统存在一个bug调用startForeground传入空的Notification（即null），则不会显示通知栏，同样服务进程的adj的值也会被降低

对于 API level >= 18：在需要提优先级的service A启动一个InnerService，两个服务同时startForeground，且绑定同样的 ID。Stop 掉InnerService ，这样通知栏图标即被移除。(但是我在实验的时候并没有什么卵用，可能这个在Android R中是无效的了)。

## 参考链接

+ [https://blog.csdn.net/qq_40881680/article/details/85197044](https://blog.csdn.net/qq_40881680/article/details/85197044)
+ [https://juejin.cn/post/6844904130708766727](https://juejin.cn/post/6844904130708766727)
+ [https://www.jianshu.com/p/1da4541b70ad](https://www.jianshu.com/p/1da4541b70ad)
+ [https://www.jianshu.com/p/06a1a434e057](https://www.jianshu.com/p/06a1a434e057)
