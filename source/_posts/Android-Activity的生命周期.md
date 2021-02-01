---
title: Android-Activity的生命周期
date: 2021-01-25 18:15:12
tags: Android
categories: Android
---
# Android-Activity的生命周期

Activity的生命周期主要有如下几个

+ onCreate
+ onRestart
+ onStart
+ onResume
+ onPause
+ onStop
+ onDestroy

onCreate主要在Activity创建的时候执行

onRestart在返回Activity前台时候执行，例如使用home键挂在后台，然后再次打开前台，就会执行onRestart，或者在一个Activity中打开了另一个Activity，再次返回这个Activity时，也会执行onRestart

onStart执行在onCreate之后或者onRestart之后，此时前台可见但是还不可以与用户进行交互

onResume执行在onStart之后，前台可见也可以与用户进行交互

onPause在退出Activity或者关闭Activity时执行，此时Activity可见但是不可以进行交互，应该在此处保存一些信息，因为此时进程的优先级可能较低，会随时被系统回收

onStop执行时Activity已经不可见，被下一个Activity覆盖了或者退回到桌面了

onDestroy执行时Activity已经被彻底的销毁了

## 关于生命周期的例子

```java
public class MainActivity extends Activity {

    private final static String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Log.e(TAG, "onCreate");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
                                                                                                                  
    @Override
    protected void onResume() {
        Log.e(TAG, "onResume");
        super.onResume();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        Log.e(TAG, "window hasFocus: " + hasFocus);
        super.onWindowFocusChanged(hasFocus);
    }

    @Override
    protected void onPause() {
        Log.e(TAG, "onPause");
        super.onPause();
    }

    @Override
    protected void onStop() {
        Log.e(TAG, "onStop");
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        Log.e(TAG, "onDestroy");
        super.onDestroy();
    }
}
```

如上单个Activity，当我们打开app时候，执行的顺序为

```bash
01-25 17:45:12.330  4998  4998 E MainActivity: onCreate
01-25 17:45:12.366  4998  4998 E MainActivity: onStart
01-25 17:45:12.367  4998  4998 E MainActivity: onResume
```

当按返回键时执行

```bash
01-25 17:45:54.847  4998  4998 E MainActivity: onPause
01-25 17:45:55.445  4998  4998 E MainActivity: onStop
01-25 17:45:55.448  4998  4998 E MainActivity: onDestroy
```

当按home键时

```bash
01-25 17:46:46.097  4998  4998 E MainActivity: onPause
01-25 17:46:46.119  4998  4998 E MainActivity: onStop
```

按home键，并且再次返回时

```bash
01-25 17:47:20.141  4998  4998 E MainActivity: onRestart
01-25 17:47:20.144  4998  4998 E MainActivity: onStart
01-25 17:47:20.145  4998  4998 E MainActivity: onResume
```

我们用两个Activity跳转来观察一下生命周期（MainActivity -> MainActivity1）

```java
public class MainActivity extends Activity {

    private final static String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Log.e(TAG, "onCreate");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button toBtn = findViewById(R.id.to_btn);
        toBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction("com.lightingsui.TEST");
                intent.addCategory("android.intent.category.DEFAULT");
            }
        }
    }
    @Override
    protected void onResume() {
        Log.e(TAG, "onResume");
        super.onResume();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        Log.e(TAG, "window hasFocus: " + hasFocus);
        super.onWindowFocusChanged(hasFocus);
    }

    @Override
    protected void onPause() {
        Log.e(TAG, "onPause");
        super.onPause();
    }

    @Override
    protected void onStop() {
        Log.e(TAG, "onStop");
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        Log.e(TAG, "onDestroy");
        super.onDestroy();
    }
}
```

```java
public class MainActivity1 extends Activity {

    private final static String TAG = MainActivity1.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Log.e(TAG, "onCreate1");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main1);
    }

    @Override
    protected void onStart() {
        Log.e(TAG, "onStart1");
        super.onStart();
    }

    @Override
    protected void onRestart() {
        Log.e(TAG, "onRestart1");
        super.onRestart();
    }

    @Override
    protected void onResume() {
        Log.e(TAG, "onResume1");
        super.onResume();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        Log.e(TAG, "window hasFocus1: " + hasFocus);
        super.onWindowFocusChanged(hasFocus);
    }

    @Override
    protected void onPause() {
        Log.e(TAG, "onPause1");
        super.onPause();
    }

    @Override
    protected void onStop() {
        Log.e(TAG, "onStop1");
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        Log.e(TAG, "onDestroy1");
        super.onDestroy();
    }
}
```

从MainActivity跳转到MainActivity1

```bash
01-25 17:53:56.161  5972  5972 E MainActivity: onPause
01-25 17:53:56.174  5972  5972 E MainActivity1: onCreate1
01-25 17:53:56.203  5972  5972 E MainActivity1: onStart1
01-25 17:53:56.204  5972  5972 E MainActivity1: onResume1
01-25 17:53:56.761  5972  5972 E MainActivity: onStop
```

再从MainActivity1返回到MainActivity

```bash
01-25 17:54:55.766  5972  5972 E MainActivity1: onPause1
01-25 17:54:55.773  5972  5972 E MainActivity: onRestart
01-25 17:54:55.773  5972  5972 E MainActivity: onStart
01-25 17:54:55.774  5972  5972 E MainActivity: onResume
01-25 17:54:56.302  5972  5972 E MainActivity1: onStop1
01-25 17:54:56.303  5972  5972 E MainActivity1: onDestroy1
```

## onWindowFocusChanged

在Android的Activity生命周期中，我们上面见到的都不是真正的visible点，而onWindowFocusChanged才是真正的visible点，这个函数当Activity获取到或者失去焦点的时候执行。

当执行这个函数的时候，Activity已经对用户可见并且可以交互了，在这之前执行的是onResume，之前对用户的交互是会受到一点限制的。

**用处**

+ 检测Acticity是否加载完毕
+ 获取手机屏幕的宽高
+ 获取View组件的宽高

我们对上面MainActivity代码稍作修改，分别在onCreate、onResume、onWindowFocusChanged中获取TextView的宽高并且通过日志打印

```java
public class MainActivity extends Activity {

    private final static String TAG = MainActivity.class.getSimpleName();
    TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = findViewById(R.id.tv_hello_world);
    }
    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        
        int width = textView.getWidth();
        int height = textView.getHeight();
        
        Log.e(TAG, "TextView width: " + width);
        Log.e(TAG, "TextView height: " + height);
    }

    @Override
    protected void onPause() {
        super.onPause();
    }

    @Override
    protected void onStop() {
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
}
```

通过上述的例子发现，当TextView的宽高为wrap_content时，只有在onWindowFocusChanged中才能获取到其宽高，其他获取到的都为0。

**失去焦点的情况**

+ 软键盘弹出
+ 状态栏下拉
+ 关闭Activity
+ ......

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    // 形参 hasFocus指的是当失去焦点时，hasFocus为false，得到焦点时，hashFocus为true
    super.onWindowFocusChanged(hasFocus);
}
```
## 参考链接

+ [https://www.jianshu.com/p/d4d677727a0a](https://www.jianshu.com/p/d4d677727a0a)
+ [https://www.jianshu.com/p/ecca892ea72e](https://www.jianshu.com/p/ecca892ea72e)
+ [https://kidsea.github.io/2017/07/12/%E6%B5%85%E6%9E%90onWindowsFocusChanged()%E6%96%B9%E6%B3%95/](https://kidsea.github.io/2017/07/12/%E6%B5%85%E6%9E%90onWindowsFocusChanged()%E6%96%B9%E6%B3%95/)
