---
title: Android中的LowMemoryKiller机制
date: 2021-01-21 21:43:14
tags: [Android]
categories: [Android]
---
# Android中的LowMemoryKiller机制

> 本文所有源码来自于原生Android 11源代码。在线查看网站https://android-opengrok.bangnimang.net/android-11.0.0_r8/

进程的启动分为冷启动和热启动，当用户退出进程后，Android系统不会立即将此进程回收，而是将其放到后台运行，下次再启动这个程序的时候，直接将这个放在后台的进程拉起来使用，加快启动速度，这种启动方式称为热启动。而冷启动则是重新为这个程序分配进程。

那么问题来了，当启动的程序较多，然后又退出了，后台就会留下很多这种空的进程，占据了大量的内存空间。Android当内存剩余的空间满足一定的条件时，会对后台的进程进行查杀，以保证内存是可用的，这就是Android中LMK（LowMemoryKiller机制）

在查杀的过程中，LMK是有一定的侧重点的，按照一定的等级进行查杀，先查杀等级最低的，以此类推。 

**1.前台进程**

+ 和用户正在进行交互的`activity`所属的应用的进程
+ 拥有某个`Service`，后台绑定到用户正在交互的`Activity`
+ 拥有正在"前台"运行的`Service`（服务已经调用`startForeground()`）
+ 正在执行一个生命周期回调的`Service`（`onCreate()`、`onStart()`、`onDestroy()`）
+ 正在执行`onReceive()`方法的`BroadCastReceiver`

Android系统只有在万不得已的情况下才会回收此进程（一般回收此种进程的概率并不大）

**2.可见进程**

+ 绑定到可见前台的Service
+ 托管不在前台，但仍对用户可见的Activity（已调用其`onPause()`方法）

**3.服务进程**

+ 正在执行`startService()`方法开启服务的进程

**4.后台进程**

后台会存在很多这种进程，当内存不足时，系统会终止他们，这些后台进程会被系统保存在LRU列表中，以保证用户最先访问的进程最后一个被终止。

+ activity已经调用`onStop()`方法，表示已经不可见的进程

**5.空进程**

+ 此种进程就是某一个进程退出后，缓存在后台的进程，只是当做缓存进程，加速下一次的启动，可能随时被回收

## LMK（LowMemoryKiller）原理

Android会为每个进程维护一个adj(在Android 6之前包括6称为`oom_adj`，在android 7开始为`oom_score_adj`)的值，在手机文件中存在着两个文件（不同类型的手机可能以下两个文件中的数值是不一样的）

`/sys/module/lowmemorykiller/parameters/minfree`，这个文件里的内容是一串以逗号分隔开的数组，每一项代表一个内存级别

`/sys/module/lowmemorykiller/parameters/adj` 也是一个数组，每一项代表着oom_adj的值

> 我到目前查遍了整个文件系统也没在手机里找到这两个文件，不知道是什么原因，下面的案例数据我是在参考链接里截取的

这两个文件相互配合，就能确定杀掉哪些进程

```bash
# /sys/module/lowmemorykiller/parameters/minfree
18432,23040,27648,32256,55296,80640
# /sys/module/lowmemorykiller/parameters/adj
0,100,200,300,900,906
```

两个文件相互对应着看，当可用内存低于80640个内存页的时候，会杀掉adj的值大于906的进程，当可用内存小于55296个内存页的时候，会杀掉adj大于900的进程，以此类推。

adj是在动态计算的，当app的状态以及四大组件的生命周期在改变时，都会改变adj的值，每个进程的adj的值都存储在`/proc/pid/oom_adj`下，高版本的内核已经不再使用`oom_adj`这个文件，而是使用`oom_adj_score`这个文件。

其中进程的状态定义在`ActivityManager.java`中，这些状态的作用就是帮助AMS计算adj（其实四大组件的变化就会对应到这里的进程状态）

| 常量定义                               | 常量取值 | 含义                                                         |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| PROCESS_STATE_UNKNOWN                  | -1       | 非真实的进程状态                                             |
| PROCESS_STATE_PERSISTENT               | 0        | persistent 系统进程                                          |
| PROCESS_STATE_PERSISTENT_UI            | 1        | persistent 系统进程，并正在执行UI操作                        |
| PROCESS_STATE_TOP                      | 2        | 拥有当前用户可见的 top Activity                              |
| PROCESS_STATE_FOREGROUND_SERVICE       | 3        | 托管一个前台 Service 的进程                                  |
| PROCESS_STATE_BOUND_FOREGROUND_SERVICE | 4        | 托管一个由系统绑定的前台 Service 的进程                      |
| PROCESS_STATE_IMPORTANT_FOREGROUND     | 5        | 对用户很重要的进程，用户可感知其存在                         |
| PROCESS_STATE_IMPORTANT_BACKGROUND     | 6        | 对用户很重要的进程，用户不可感知其存在                       |
| PROCESS_STATE_TRANSIENT_BACKGROUND     | 7        | Process is in the background transient so we will try to keep running. |
| PROCESS_STATE_BACKUP                   | 8        | 后台进程，正在运行backup/restore操作                         |
| PROCESS_STATE_SERVICE                  | 9        | 后台进程，且正在运行service                                  |
| PROCESS_STATE_RECEIVER                 | 10       | 后台进程，且正在运行receiver                                 |
| PROCESS_STATE_TOP_SLEEPING             | 11       | 与 PROCESS_STATE_TOP 一样，但此时设备正处于休眠状态          |
| PROCESS_STATE_HEAVY_WEIGHT             | 12       | 后台进程，但无法执行restore，因此尽量避免kill该进程          |
| PROCESS_STATE_HOME                     | 13       | 后台进程，且拥有 home Activity                               |
| PROCESS_STATE_LAST_ACTIVITY            | 14       | 后台进程，且拥有上一次显示的 Activity                        |
| PROCESS_STATE_CACHED_ACTIVITY          | 15       | 进程处于 cached 状态，且内含 Activity                        |
| PROCESS_STATE_CACHED_ACTIVITY_CLIENT   | 16       | 进程处于 cached 状态，且为另一个 cached 进程(内含 Activity)的 client 进程 |
| PROCESS_STATE_CACHED_RECENT            | 17       | 进程处于 cached 状态，且内含与当前最近任务相对应的 Activity  |
| PROCESS_STATE_CACHED_EMPTY             | 18       | 进程处于 cached 状态，且为空进程                             |
| PROCESS_STATE_NONEXISTENT              | 19       | 不存在的进程                                                 |

在Android 6以及之前，oom_adj的值取值范围为[-17,16]，Android 7开始，adj的取值变为了[-1000, 1000]。

> 将adj的范围变大，可以更加细化当前进程的状态

| 常量定义               | 常量取值 | 含义                                               |
| ---------------------- | -------- | -------------------------------------------------- |
| NATIVE_ADJ             | -1000    | native进程（不被系统管理）                         |
| SYSTEM_ADJ             | -900     | 系统进程                                           |
| PERSISTENT_PROC_ADJ    | -800     | 系统persistent进程，比如telephony                  |
| PERSISTENT_SERVICE_ADJ | -700     | 关联着系统或persistent进程                         |
| FOREGROUND_APP_ADJ     | 0        | 前台进程                                           |
| VISIBLE_APP_ADJ        | 100      | 可见进程                                           |
| PERCEPTIBLE_APP_ADJ    | 200      | 可感知进程，比如后台音乐播放                       |
| BACKUP_APP_ADJ         | 300      | 备份进程                                           |
| HEAVY_WEIGHT_APP_ADJ   | 400      | 后台的重量级进程，system/rootdir/init.rc文件中设置 |
| SERVICE_ADJ            | 500      | 服务进程                                           |
| HOME_APP_ADJ           | 600      | Home进程                                           |
| PREVIOUS_APP_ADJ       | 700      | 上一个App的进程                                    |
| SERVICE_B_ADJ          | 800      | B List中的Service（较老的、使用可能性更小）        |
| CACHED_APP_MIN_ADJ     | 900      | 不可见进程的adj最小值                              |
| CACHED_APP_MAX_ADJ     | 906      | 不可见进程的adj最大值                              |
| UNKNOWN_ADJ            | 1001     | 一般指将要会缓存进程，无法获取确定值               |

看一下`adj`这个值是如何随着四大组件进行变化的：

以今日头条app进行举例，目前一个app可能对应着多个进程，当我们启动头条app后，使用头条app的包名即可查找到所有头条启动的进程

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210113160813.png)

第一个进程的`pid`为7585，此时处于`activity`可见的状态，查看一下`adj`的值

```bash
picasso:/ # cat /proc/7585/oom_score_adj                                 
0
```

当app处于不可见（activity不可见）下，adj的值为

```bash
picasso:/ # cat /proc/7585/oom_score_adj                                            
700
```

有了上面的概念，这下我们来总结一下`LowMemoryKiller`的工作流程

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210121142757.jpg)

当操作app的时候，例如从前台切换到后台（导致四大组件的生命周期发生变化），此时AMS会重新计算adj的值，在`native`层的LMKD（LowMemoryKiller Daemo，LMK的守护线程）内部维护着一张表（数组），这个数组内部每一个元素又是一个双向链表，每个链表中的内容为相同adj的进程信息，当AMS重新计算完adj的值后，会通过socket通信的方式将更新的oom_adj的值传递给LMKD，LMKD更新维护的这张表。当内存达到上面文件中配置的要求时，会从链表中选出一些去干掉进程，回收内存空间。下面还有一张图，同样是描述的LowMemoryKiller工作机制

<img src="https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210113155902.webp" style="zoom:80%;" />

上面提到了LMKD守护线程，这个线程是由init进程（pid = 1）的进程通过分析`/etc/init/lmkd.rc`生成的，并且在启动后，还会在`/dev/socket`下创建一个名为lmkd的套接字，提供给framework层和native层通信。

```bash
picasso:/proc/885 # ps -ef | grep lmkd
lmkd           885     1 0 14:56:33 ?     00:00:17 lmkd
root         10808  5253 4 20:16:59 pts/0 00:00:00 grep lmkd
picasso:/proc/885 # cd /proc/885
picasso:/proc/885 # cat status
Name:   lmkd
Umask:  0077
State:  S (sleeping)
Tgid:   885
Ngid:   0
Pid:    885
# 父进程ID init(1)
PPid:   1
TracerPid:      0
Uid:    1069    1069    1069    1069
Gid:    1069    1069    1069    1069
FDSize: 64
```

## 源码分析

先梳理一下流程：当四大组件的状态发生改变时，AMS（ActivitymanagerService）直接或间接调用`ProcessList.java`中的方法与native层的LMKD进行通信，LMKD维护自身的进程数组（数组中的每个节点又是一组相同adj的进程），最后kernel层负责对进程查杀

在`ProcessList.java`中，定义了七种命令

```java
static final byte LMK_TARGET = 0;
static final byte LMK_PROCPRIO = 1;
static final byte LMK_PROCREMOVE = 2;
static final byte LMK_PROCPURGE = 3;
static final byte LMK_GETKILLCNT = 4;
static final byte LMK_SUBSCRIBE = 5;
static final byte LMK_PROCKILL = 6; // Note: this is an unsolicated command
```

并且在LMKD守护进程中，(lmkd.h)也有着与之相同的命令

```c++
/*
* Supported LMKD commands
*/
enum lmk_cmd {
    LMK_TARGET = 0, /* Associate minfree with oom_adj_score */
    LMK_PROCPRIO,   /* Register a process and set its oom_adj_score */
    LMK_PROCREMOVE, /* Unregister a process */
    LMK_PROCPURGE,  /* Purge all registered processes */
    LMK_GETKILLCNT, /* Get number of kills */
    LMK_SUBSCRIBE,  /* Subscribe for asynchronous events */
    LMK_PROCKILL,   /* Unsolicited msg to subscribed clients on proc kills */
    LMK_UPDATE_PROPS, /* Reinit properties */
};
```

native层的lmkd与framework层的AMS就是通过这些命令进行交互的，常用的命令其实只有前三个

+ LMK_TARGET 
+ LMK_PROCPRIO
+ LMK_PROCREMOVE

### LMK_PROCPRIO

这个命令就是负责更新某个进程的adj值，在AMS中，如果感知到某个进程的四大组件状态发生改变，会调用`OomAdjuster.updateOomAdjLocked`方法，而`OomAdjuster.updateOomAdjLocked`又会调用`OomAdjuster.applyOomAdjLocked`，在`applyOomAdjLocked`中最终调用了`ProcessList.setOomAdj`方法。

在AMS中可以找到很多地方调用了`OomAdjuster.updateOomAdjLocked`方法，这些调用者基本上都是四大组件改变时的函数。

主要就是看`ProcessList.setOomAdj`这个方法，此方法是负责与native层的socket通信

```java
public static void setOomAdj(int pid, int uid, int amt) {
    // This indicates that the process is not started yet and so no need to proceed further.
	if (pid <= 0) {
		return;
	}
	if (amt == UNKNOWN_ADJ)
		return;
  
	long start = SystemClock.elapsedRealtime();
    // 将进程一些信息放进ByteBuffer
	ByteBuffer buf = ByteBuffer.allocate(4 * 4);
	buf.putInt(LMK_PROCPRIO);
	buf.putInt(pid);
	buf.putInt(uid);
	buf.putInt(amt);
    // 写入socket缓冲区，准备发送
	writeLmkd(buf, null);
	long now = SystemClock.elapsedRealtime();
	if ((now-start) > 250) {
		Slog.w("ActivityManager", "SLOW OOM ADJ: " + (now-start) + "ms for pid " + pid
			+ " = " + amt);
	}
}
```

**writeLmkd**

```java
private static boolean writeLmkd(ByteBuffer buf, ByteBuffer repl) {
    // isConnected，此方法是主要，监测socket是否创建连接
	if (!sLmkdConnection.isConnected()) {
		// try to connect immediately and then keep retrying
		sKillHandler.sendMessage(
		sKillHandler.obtainMessage(KillHandler.LMKD_RECONNECT_MSG));
 
		// wait for connection retrying 3 times (up to 3 seconds)
		if (!sLmkdConnection.waitForConnection(3 * LMKD_RECONNECT_DELAY_MS)) {
			return false;
		}
	}
    // 发送消息
    return sLmkdConnection.exchange(buf, repl);
}
```

**LmkdConnection.isConnected**

```java
// 此方法判断mLmkdSocket是否连接，主要通过一个标志mLmkdSocket
public boolean isConnected() {
	synchronized (mLmkdSocketLock) {
		return (mLmkdSocket != null);
	}
}
```

**LmkdConnection.connect**

```java
public boolean connect() {
	synchronized (mLmkdSocketLock) {
		if (mLmkdSocket != null) {
			return true;
		}
		// temporary sockets and I/O streams
        // 打开连接
		final LocalSocket socket = openSocket();
 
		if (socket == null) {
			Slog.w(TAG, "Failed to connect to lowmemorykiller, retry later");
			return false;
		}
		
        // 获得输入输出
		final OutputStream ostream;
		final InputStream istream;
		try {
			ostream = socket.getOutputStream();
			istream = socket.getInputStream();
		} catch (IOException ex) {
			IoUtils.closeQuietly(socket);
			return false;
		}
		// execute onConnect callback
		if (mListener != null && !mListener.onConnect(ostream)) {
			Slog.w(TAG, "Failed to communicate with lowmemorykiller, retry later");
			IoUtils.closeQuietly(socket);
			return false;
		}
		// connection established
		mLmkdSocket = socket;
		mLmkdOutputStream = ostream;
		mLmkdInputStream = istream;
        mMsgQueue.addOnFileDescriptorEventListener(mLmkdSocket.getFileDescriptor(),
			EVENT_INPUT | EVENT_ERROR,
			new MessageQueue.OnFileDescriptorEventListener() {
				public int onFileDescriptorEvents(FileDescriptor fd, int events) {
					return fileDescriptorEventHandler(fd, events);
				}
			}
		);
		mLmkdSocketLock.notifyAll();
	}
	return true;
}
```

会在`ProcessList`中调用connect打开连接，然后再`setOomAdj`向native层中的LMKD发送更新adj的消息。

**转入到LMKD中（lmkd.cpp）**

在main中会调用`mainloop`，`mainloop`中有着一大串的逻辑，简化完就是调用`epoll_wait`等待`framework`层发送的消息，接收到消息后会调用在`init`中定义好的回调方法`ctrl_connect_handler`

```c++
/* Second pass to handle all other events */
for (i = 0, evt = &events[0]; i < nevents; ++i, evt++) {
	if (evt->events & EPOLLERR) {
		ALOGD("EPOLLERR on event #%d", i);
	}
	if (evt->events & EPOLLHUP) {
		/* This case was handled in the first pass */
		continue;
	}
	if (evt->data.ptr) {
        // 接收到消息，调用回调方法
		handler_info = (struct event_handler_info*)evt->data.ptr;
		call_handler(handler_info, &poll_params, evt->events);
	}
}
```

在`ctrl_connect_handler`中调用`ctrl_data_handler`，接下来`ctrl_command_handler`。

`ctrl_command_handler`里面就包含了在`ProcessList`中定义的6个命令，这六个命令会进行不同的逻辑处理

```c++
static void ctrl_command_handler(int dsock_idx) {
	......
 
	switch(cmd) {
        // 第一个指令
		case LMK_TARGET:
			targets = nargs / 2;
			if (nargs & 0x1 || targets > (int)ARRAY_SIZE(lowmem_adj))
				goto wronglen;
            // 处理逻辑
			cmd_target(targets, packet);
			break;
		case LMK_PROCPRIO:
			/* process type field is optional for backward compatibility */
			if (nargs < 3 || nargs > 4)
				goto wronglen;
			cmd_procprio(packet, nargs, &cred);
			break;
		case LMK_PROCREMOVE:
			if (nargs != 1)
				goto wronglen;
			cmd_procremove(packet, &cred);
			break;
		case LMK_PROCPURGE:
			if (nargs != 0)
				goto wronglen;
			cmd_procpurge(&cred);
			break;
		case LMK_GETKILLCNT:
			if (nargs != 2)
				goto wronglen;
			kill_cnt = cmd_getkillcnt(packet);
			len = lmkd_pack_set_getkillcnt_repl(packet, kill_cnt);
			if (ctrl_data_write(dsock_idx, (char *)packet, len) != len)
				return;
			break;
		case LMK_SUBSCRIBE:
			if (nargs != 1)
				goto wronglen;
			cmd_subscribe(dsock_idx, packet);
			break;
		case LMK_PROCKILL:
			/* This command code is NOT expected at all */
			ALOGE("Received unexpected command code %d", cmd);
			break;
		case LMK_UPDATE_PROPS:
			if (nargs != 0)
				goto wronglen;
			update_props();
			if (!use_inkernel_interface) {
				/* Reinitialize monitors to apply new settings */
				destroy_monitors();
				result = init_monitors() ? 0 : -1;
			} else {
				result = 0;
			}
			len = lmkd_pack_set_update_props_repl(packet, result);
			if (ctrl_data_write(dsock_idx, (char *)packet, len) != len) {
				ALOGE("Failed to report operation results");
			}
			if (!result) {
				ALOGI("Properties reinitilized");
			} else {
				/* New settings can't be supported, crash to be restarted */
				ALOGE("New configuration is not supported. Exiting...");
				exit(1);
			}
			break;
		default:
			ALOGE("Received unknown command code %d", cmd);
			return;
	}
 
	return;
 
	wronglen:
		ALOGE("Wrong control socket read length cmd=%d len=%d", cmd, len);
}
```

我们此次分析的是更新进程`adj`逻辑，即`LMK_PROCPRIO`指令，所以进入到`cmd_procprio`

```c++
static void cmd_procprio(LMKD_CTRL_PACKET packet, int field_count, struct ucred *cred) {
	......
	
	// 将adj的值写入到/proc/%d/oom_score_adj文件下
	snprintf(path, sizeof(path), "/proc/%d/oom_score_adj", params.pid);
	snprintf(val, sizeof(val), "%d", params.oomadj);
	if (!writefilestring(path, val, false)) {
		ALOGW("Failed to open %s; errno=%d: process %d might have been killed",
			path, errno, params.pid);
		/* If this file does not exist the process is dead. */
		return;
	}
	

	if (use_inkernel_interface) {
		stats_store_taskname(params.pid, proc_get_name(params.pid, path, sizeof(path)));
		return;
	}
	......

	// 通过循环从数组的链表中找到进程的节点
	procp = pid_lookup(params.pid);
	if (!procp) { 	// 进程节点为null则创建一个节点
		int pidfd = -1;

		if (pidfd_supported) {
			pidfd = TEMP_FAILURE_RETRY(sys_pidfd_open(params.pid, 0));
			if (pidfd < 0) {
				ALOGE("pidfd_open for pid %d failed; errno=%d", params.pid, errno);
				return;
			}
		}

        procp = static_cast<struct proc*>(calloc(1, sizeof(struct proc)));
        if (!procp) {
            // Oh, the irony.  May need to rebuild our state.
            return;
        }
  
        procp->pid = params.pid;
        procp->pidfd = pidfd;
		procp->uid = params.uid;
		procp->reg_pid = cred->pid;
		procp->oomadj = params.oomadj;
		proc_insert(procp);
	} else {	// 进程节点不为null则更新节点adj的值
		if (!claim_record(procp, cred->pid)) {
			char buf[LINE_MAX];
			/* Only registrant of the record can remove it */
			ALOGE("%s (%d, %d) attempts to modify a process registered by another client",
			proc_get_name(cred->pid, buf, sizeof(buf)), cred->uid, cred->pid);
			return;
		}
		proc_unslot(procp);
        
        // 更新当前节点adj
		procp->oomadj = params.oomadj;
		proc_slot(procp);
	}
}
```

我们对adj更新的分析就到这啦，在深入一层的kernel就不分析了，主要就是对满足条件的进程进行kill

### LMK_TARGET

当系统启动时，需要更新`minfree`和adj范围数组，这两个数组在`ProcessList.java`中有两个范围值

```java
private final int[] mOomMinFreeLow = new int[] {
        12288, 18432, 24576,
        36864, 43008, 49152
};
// These are the high-end OOM level limits.  This is appropriate for a
// 1280x800 or larger screen with around 1GB RAM.  Values are in KB.
private final int[] mOomMinFreeHigh = new int[] {
        73728, 92160, 110592,
        129024, 147456, 184320
};
```

更新就是通过`ProcessList.updateOomLevels()`更新adj范围值和`minfree`的，主要就是在系统启动的时候更新，调用是在`ProcessList`的构造函数中

```java
ProcessList() {
	MemInfoReader minfo = new MemInfoReader();
	minfo.readMemInfo();
	mTotalMemMb = minfo.getTotalSize()/(1024*1024);
	updateOomLevels(0, 0, false);
}
```

而`updateOomLevels`则是根据当前屏幕的分辨率宽高等进行更新的

```java
private void updateOomLevels(int displayWidth, int displayHeight, boolean write) {
    // 根据屏幕分辨率计算minfree
    // Scale buckets from avail memory: at 300MB we use the lowest values to
    // 700MB or more for the top values.
    float scaleMem = ((float) (mTotalMemMb - 350)) / (700 - 350);
    831  
    // Scale buckets from screen size.
    int minSize = 480 * 800;  //  384000
    int maxSize = 1280 * 800; // 1024000  230400 870400  .264
    float scaleDisp = ((float)(displayWidth * displayHeight) - minSize) / (maxSize - minSize);
    if (false) {
        Slog.i("XXXXXX", "scaleMem=" + scaleMem);
        Slog.i("XXXXXX", "scaleDisp=" + scaleDisp + " dw=" + displayWidth
			+ " dh=" + displayHeight);
	}

    float scale = scaleMem > scaleDisp ? scaleMem : scaleDisp;
    if (scale < 0) scale = 0;
    else if (scale > 1) scale = 1;
    int minfree_adj = Resources.getSystem().getInteger(
		com.android.internal.R.integer.config_lowMemoryKillerMinFreeKbytesAdjust);
	int minfree_abs = Resources.getSystem().getInteger(
		com.android.internal.R.integer.config_lowMemoryKillerMinFreeKbytesAbsolute);
	if (false) {
		Slog.i("XXXXXX", "minfree_adj=" + minfree_adj + " minfree_abs=" + minfree_abs);
	}
 
	final boolean is64bit = Build.SUPPORTED_64_BIT_ABIS.length > 0;
	
    // 以下逻辑主要是更新minfree
	for (int i = 0; i < mOomAdj.length; i++) {
		int low = mOomMinFreeLow[i];
		int high = mOomMinFreeHigh[i];
		if (is64bit) {
			// Increase the high min-free levels for cached processes for 64-bit
			if (i == 4) high = (high * 3) / 2;
			else if (i == 5) high = (high * 7) / 4;
		}
		mOomMinFree[i] = (int)(low + ((high - low) * scale));
	}
 
	if (minfree_abs >= 0) {
		for (int i = 0; i < mOomAdj.length; i++) {
			mOomMinFree[i] = (int)((float)minfree_abs * mOomMinFree[i]
				/ mOomMinFree[mOomAdj.length - 1]);
		}
	}

	if (minfree_adj != 0) {
		for (int i = 0; i < mOomAdj.length; i++) {
			mOomMinFree[i] += (int)((float) minfree_adj * mOomMinFree[i]
				/ mOomMinFree[mOomAdj.length - 1]);
            if (mOomMinFree[i] < 0) {
                mOomMinFree[i] = 0;
            }
		}
	}

    // The maximum size we will restore a process from cached to background, when under
    // memory duress, is 1/3 the size we have reserved for kernel caches and other overhead
    // before killing background processes.
    mCachedRestoreLevel = (getMemLevel(ProcessList.CACHED_APP_MAX_ADJ) / 1024) / 3;

    // Ask the kernel to try to keep enough memory free to allocate 3 full
    // screen 32bpp buffers without entering direct reclaim.
    int reserve = displayWidth * displayHeight * 4 * 3 / 1024;
    int reserve_adj = Resources.getSystem().getInteger(
        com.android.internal.R.integer.config_extraFreeKbytesAdjust);
    int reserve_abs = Resources.getSystem().getInteger(
        com.android.internal.R.integer.config_extraFreeKbytesAbsolute);

    if (reserve_abs >= 0) {
		reserve = reserve_abs;
	}
  
	if (reserve_adj != 0) {
		reserve += reserve_adj;
		if (reserve < 0) {
			reserve = 0;
		}
	}
	
    // 通过socket将信息发送到native层LMKD
	if (write) {
		ByteBuffer buf = ByteBuffer.allocate(4 * (2 * mOomAdj.length + 1));
        // 推送LMK_TARGET指令
		buf.putInt(LMK_TARGET);
		for (int i = 0; i < mOomAdj.length; i++) {
			buf.putInt((mOomMinFree[i] * 1024)/PAGE_SIZE);
			buf.putInt(mOomAdj[i]);
		}

		writeLmkd(buf, null);
		SystemProperties.set("sys.sysctl.extra_free_kbytes", Integer.toString(reserve));
		mOomLevelsSet = true;
	}
	// GB: 2048,3072,4096,6144,7168,8192
	// HC: 8192,10240,12288,14336,16384,20480
}
```

转到native层LMKD，由于逻辑与处理函数同上面的更新adj方法，所以只拿出来`ctrl_command_handler`进行分析即可，通过命令选择，最终执行了`cmd_target`方法，这个方法就只看两部分就可以了，重要逻辑就是更新minfree和adj范围值

```c++
snprintf(val, sizeof(val), "%d", use_inkernel_interface ? lowmem_minfree[i] : 0);
strlcat(minfreestr, val, sizeof(minfreestr));
snprintf(val, sizeof(val), "%d", use_inkernel_interface ? lowmem_adj[i] : 0);
strlcat(killpriostr, val, sizeof(killpriostr));

writefilestring(INKERNEL_MINFREE_PATH, minfreestr, true);
writefilestring(INKERNEL_ADJ_PATH, killpriostr, true);
```

### LMK_PROCREMOVE

该命令的主要作用就是将当前进程从LMKD策略中移除，也就是将进程节点信息从`lmkd.cpp`中的数组中的链表移除，不受`lmkd`策略的控制。

当AMS调用`handleAppDiedLocked`时，会调用`cleanUpApplicationRecordLocked`，其中`AMS.cleanUpApplicationRecordLocked`调用`ProcessList.java`中的

```java
if (!app.isPersistent() || app.isolated) {
	if (DEBUG_PROCESSES || DEBUG_CLEANUP) Slog.v(TAG_CLEANUP,
		"Removing non-persistent process during cleanup: " + app);
	if (!replacingPid) {
        // 此处为重要点，调用ProcessList中的移除方法
		mProcessList.removeProcessNameLocked(app.processName, app.uid, app);
	}
	mAtmInternal.clearHeavyWeightProcessIfEquals(app.getWindowProcessController());
} else if (!app.removed) {
    // This app is persistent, so we need to keep its record around.
    // If it is not already on the pending app list, add it there
    // and start a new process for it.
	if (mPersistentStartingProcesses.indexOf(app) < 0) {
		mPersistentStartingProcesses.add(app);
		restart = true;
	}
}
```

**ProcessList.removeProcessNameLocked**

```
@GuardedBy("mService")
final ProcessRecord removeProcessNameLocked(final String name, final int uid,
	final ProcessRecord expecting) {
	
	......
	
    if ((expecting == null) || (old == expecting)) {
    	// 通信native层LMKD
    	mProcessNames.remove(name, uid);
    }
    
    ......
}
```

```
final MyProcessMap mProcessNames = new MyProcessMap();

final class MyProcessMap extends ProcessMap<ProcessRecord> {
@Override
public ProcessRecord put(String name, int uid, ProcessRecord value) {
	final ProcessRecord r = super.put(name, uid, value);
	mService.mAtmInternal.onProcessAdded(r.getWindowProcessController());
	return r;
}
 
@Override
public ProcessRecord remove(String name, int uid) {
	// 调用remove方法
    final ProcessRecord r = super.remove(name, uid);
    mService.mAtmInternal.onProcessRemoved(name, uid);
    return r;
}
```

**remove**

```
public static final void remove(int pid) {
    // This indicates that the process is not started yet and so no need to proceed further.
    if (pid <= 0) {
    	return;
    }
    ByteBuffer buf = ByteBuffer.allocate(4 * 2);
    
    // 发送指令，进行通信
    buf.putInt(LMK_PROCREMOVE);
    buf.putInt(pid);
    writeLmkd(buf, null);
}
```

在native层LMKD中，进行分发任务，最终到`cmd_procremove`中

```
static void cmd_procremove(LMKD_CTRL_PACKET packet, struct ucred *cred) {
    struct lmk_procremove params;
    struct proc *procp;

    lmkd_pack_get_procremove(packet, &params);

	// 当use_inkernel_interface为1的时候，就使用内核去杀进程，
	// 当use_inkernel_interface为0的时候，就用lmkd去杀进程
	if (use_inkernel_interface) {

		poll_kernel(kpoll_fd);

		stats_remove_taskname(params.pid);
		return;
	}
	
	// 循环找到当前节点
	procp = pid_lookup(params.pid);
	
	......
	
	// 移除节点
	pid_remove(params.pid);
}
```

我想大家应该明白这个流程了，不用介绍太多大家也会懂得

### 关于杀进程的逻辑

上一步中有一个标志`use_inkernel_interface`，当其值为1的时候，是调用内核的逻辑去杀进程的，大概的流程就是系统内部存在一个进程kswapd，内部维护了一张shrinker表，当内存不足的时候或者达到了一定的界限的时候，就会触发绑定在shrinker表的回调函数，而回调函数基本上就是杀进程的逻辑，得到当前内存值，然后获取当前内存对应的adj的值，遍历进程表，将遍历得到的进程adj，把所有大于剩余内存对应的adj的进程全部干掉（发送SIGKILL信号）。

### 关于LMK策略的一些思考

**1.有没有杀不死的进程**

从native进程的角度来说，是存在杀不死的进程的，native进程的最小值为-1000，如果将进程的adj值修改为-1000，从一定的角度来说是杀不死的

**2.LMKD进程会不会被LMK策略杀死**

如果native进程不会被杀死，那么LMKD进程也是不会被杀死的，看一下LMKD的adj就明白怎么回事了

```bash
picasso:/ # ps -ef | grep lmkd
lmkd           885     1 0 14:56:32 ?     00:00:19 lmkd
root         17365  5253 0 21:25:09 pts/0 00:00:00 grep lmkd
picasso:/ # cat /proc/885/oom_score_adj
-1000
```

**3.市面上的app可不可以将自己的adj值提升到-1000,以保活**

当然不可以，市面上的app无法root手机，也就是获取不到手机的root权限，也就无法修改adj，修改`/proc/pid/oom_score_adj`是需要root权限的，如果，说的是如果，如果被修改了adj值，那么下次重新启动也是会恢复的，除非不关机

系统软件如桌面都会进程一些保活的机制，就是降低自己的adj值，查看一下桌面的adj值

```bash
picasso:/ # ps -ef | grep com.miui.home
u0_a90        3224   667 0 14:56:46 ?     00:02:05 com.miui.home
root         18033  5253 0 21:32:18 pts/0 00:00:00 grep com.miui.home

# 桌面在前台时（前台进程）
picasso:/ # cat /proc/3224/oom_score_adj
0
# 桌面在后台时（即我们打开某个app）
picasso:/ # cat /proc/3224/oom_score_adj
100
```

可见，桌面在后台时adj值都这么低，被杀的几率当然很小

## 参考链接

+ https://juejin.cn/post/6844903552037421069
+ https://www.jianshu.com/p/7bd3d0ee8a56
+ https://www.jianshu.com/p/221f4a246b45（特别推荐）
+ https://www.jianshu.com/p/980b6ce4e051（特别推荐）
+ https://www.jianshu.com/p/7bd3d0ee8a56（特别推荐）
+ https://juejin.cn/post/6844903750474137613（特别推荐）
+ http://gityuan.com/2016/09/17/android-lowmemorykiller/（特别推荐）
