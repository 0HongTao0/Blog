---
title: Android 进程
date: 2018-07-06 12:44:05
tags:
---

- 进程的理解

  - 通常情况下，Android 的四大组件都在同一进程中运行，但是也可以指定组件在特定的进程中进行，通过配置 Mainfest.xml 文件下，四大组件标签内配置 android：process 属性，则可以指定该组件在哪个进程中运行。

- 如何达到进程保活？

  - 避免被杀死，先提高进程的优先级（priority），进程优先级越低越易被杀死。查看进程优先级：
		```
		adb shell ps | grep  packageName
		```
  - 在进程被杀死情况下，通过某些方法重新启动进程

<!--more-->

- 为什么 Android 系统会杀死进程？

  ![杀死进程原因](https://segmentfault.com/img/remote/1460000006252378)

- 进程等级（优先级由低到高）

  - 前台进程

    - 正在交互的 Activity 

    - 绑定正在交互 Activity 的 Service 

    - 前台 Service （Service.startForeground( )）

    - 正执行 onCreat(),  onStart(), onDestroy() 的 Service 

    - 正执行 onReceive() 的 BoradcastReceiver 

  - 可见进程

    - 不在前台，但可见的 Activity（onPause()） ，如开启一个 D

    - 绑定到可见（或前台）Activity 的 Service 

  - 后台进程

    - 主要是不可见的 Activity 的进程。这些进程的关闭对用户没有多大影响，Android 系统内存不足可以直接关闭。

  - 空进程

    - 不包含任何应用组件的进程，唯一作用：缓存，缩短下次运行组件所需的启动时间。

- 杀进程回收内存的机制（Low Memory Killer）

|adj级别|值|解释|
| :----: | :----: | :----: |
|UNKNOWN_ADJ|16|预留的最低级别，一般对于缓存的进程才有可能设置成这个级别|
|CACHED_APP_MAX_ADJ|15|缓存进程，空进程，在内存不足的情况下就会优先被kill|
|CACHED_APP_MIN_ADJ|9|缓存进程，也就是空进程|
|SERVICE_B_ADJ|8|不活跃的进程|
|PREVIOUS_APP_ADJ|7|切换进程|
|HOME_APP_ADJ|6|与Home交互的进程|
|SERVICE_ADJ|5|有Service的进程|
|HEAVY_WEIGHT_APP_ADJ|4|高权重进程|
|BACKUP_APP_ADJ|3|正在备份的进程|
|PERCEPTIBLE_APP_ADJ|2|可感知的进程，比如那种播放音乐|
|VISIBLE_APP_ADJ|1|可见进程，如当前的Activity|
|FOREGROUND_APP_ADJ|0|前台进程|
|PERSISTENT_SERVICE_ADJ|-11|重要进程|
|PERSISTENT_PROC_ADJ|-12|核心进程|
|SYSTEM_ADJ|-16|系统进程|
|NATIVE_ADJ|-17|系统起的Native进程|
	
oom_adj 取值越高越易被杀死，oom_adj 与进程优先级成反比，oom_adj 越高进程优先级越低。
- 如何达到进程保活？（让 APP 不被 Android 系统杀死）

	##### 提高进程优先级（在进程被杀死之前）

	- 方法一：若用户使用电源键息屏，监听息屏和解锁的广播，息屏时候启动一个只有一个像素的 Activity（息屏时候应用优先级很高），不易被杀死也不易被用户发现。（提高进程优先级）

	```java
	public class KeepLiveActivity extends Activity {
	  
		//在监听息屏的广播中启动这个 Activity
		public static void startAvtivity(Context context){
			Intent intent = new Intent(context, KeepLiveActivity.class);
			context.startActivity(intent);
	  }
	  
	  private static final String TAG = "KeepLiveActivity";
	  @Override
	  protected void onCreate(Bundle savedInstanceState) {
		   super.onCreate(savedInstanceState);
		   Log.e(TAG,"start Keep app activity");
		   Window window = getWindow();
		   //设置 Activity 窗口位置在 START TOP 位置（也就是左上角）
		   window.setGravity(Gravity.START | Gravity.TOP);
		   //设置宽高都为 1
		   WindowManager.LayoutParams attributes = window.getAttributes();
		   attributes.width = 1;
		   attributes.height = 1;
		   attributes.x = 0;
		   attributes.y = 0;
		   window.setAttributes(attributes);
	   }

	   @Override
	   protected void onDestroy() {
		   super.onDestroy();
		   Log.e(TAG,"stop keep app activity");
	   }
	}
	```

	此外，还要设置 Activity 无背景且透明。KeepLiveActivity 的 taskAffinity 应与应用默认taskAffinity 不同。

	```xml
	<activity
	   android:taskAffinity="com.awqingnian.demo.keep.live"
	   android:theme="@style/KeepLiveTheme" />

	<style name="KeepLiveTheme">
	   <item name="android:windowBackground">@null</item>
	   <item name="android:windowIsTranslucent">true</item>
	</style>
	```

	剩下就是添加息屏广播监听，在广播监听 onReceive() 中启动 KeepLiveActivity.

	- 方法二：开启一个通过调用 startForeground() 方法来绑定一个前台通知的 Service。（提高进程优先级）

	  此方法用户会看到通知栏中存在一个通知。

	  ```java
	  @Override
	  public int onStartCommand(Intent intent, int flags, int startId) {
		 Intent intent = new Intent(this, MainActivity.class);
		 PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
		 NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
		   .setSmallIcon(R.mipmap.ic_launcher);
		   .setContentTitle("Foreground");
		   .setContentText("I am a foreground service");
		   .setContentInfo("Content Info");
		   .setWhen(System.currentTimeMillis());
		 builder.setContentIntent(pi);
		 Notification notification = builder.build();
		 startForeground(FOREGROUND_ID, notification);
		 return super.onStartCommand(intent, flags, startId);
	  }
	  ```

	- 方法三：在方法二基础上隐藏通知。能否实现隐藏与版本有关，SDK >= 19 可以实现。（提高进程优先级）

	  先修改绑定前台通知 Service 的 onStartCommand() 里面部分部分逻辑

	  ```java
	  @Override
	  public int onStartCommand(Intent intent, int flags, int startId) {
		 try {
			 Notification notification = new Notification();
			 if (Build.VERSION.SDK_INT < 18) {
				 startForeground(NOTIFICATION_ID, notification);
			 } else {
				 startForeground(NOTIFICATION_ID, notification);
				 // start InnerService
				 startService(new Intent(this, InnerService.class));
			 }
		 } catch (Throwable e) {
			 e.printStackTrace();
		 }
	  
		 return super.onStartCommand(intent, flags, startId);
	  }
	  ```

	  然后再 InnerService 中关闭之前打开的 Notification （通知）

	  ```
	  @Override
	  public void onCreate() {
		 super.onCreate();
		 try {
			 startForeground(NOTIFICATION_ID, new Notification());
		 } catch (Throwable throwable) {
			 throwable.printStackTrace();
		 }
		 stopSelf();
	  ```

	##### 拉活进程（在进程被杀死之后）

	- 方法一：广播拉活

	- 方法二：系统Service机制拉活

	- 方法三：使用账户同步拉活

	- 方法四：使用JobSchedule拉活

	- 方法五：双进程守护