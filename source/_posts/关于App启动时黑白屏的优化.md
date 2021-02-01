---
title: 关于App启动时黑白屏的优化
date: 2021-01-27 10:35:02
tags: Android
categories: Android
---
# 关于App启动时黑白屏的优化

Android中黑白屏是由于系统启动时，Activity正在由WindowManager进行加载，还没有绘制完成，而系统会先将Theme中的windowBackground加载出来，就会导致我们看到的颜色为windowBackground设置的颜色。

可以通过设置windowBackground为一张图片或者其他的颜色，来提升用户体验，或者将windowBackground设置为透明的。

新建立一个主题

```xml
<style name="Theme.Customer" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
	<item name="android:windowFullscreen">true</item>
    <!-- 将背景换为一张图片 -->
    <item name="android:windowBackground">@drawable/unnamed</item>
</style>
```

在AndroidManifest中将activity的主题指定为新创建的这个主题

```xml
...
<activity android:name=".MainActivity" android:theme="@style/Theme.Customer"></activity>
...
```

当系统执行到onCreate方法时，证明Activity已经加载完毕，此时再将主题换回来

```java
protected void onCreate(Bundle ssavedInstanceState) {
    setTheme(R.style.Theme_AppCompat);
    super.onCreate(savedInstance);
    setContentView(R.layout.activity_main);
}
```




