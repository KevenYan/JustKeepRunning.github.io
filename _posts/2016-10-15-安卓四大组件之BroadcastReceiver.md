---
layout: post
title: 安卓四大组件之BroadcastReceiver
tags:
- Android
categories: Android四大组件

---
# *BroadcastReceiver*  
##1 为什么需要广播接收者

	
	(1)老年人 
	(2)出租车司机  
	(3)寂寞的大学生  情感夜话  卖药的 音乐台 
	(4)想听广播必须的有电台   你在买一个收音机 安装一块电池  调到一个合适的频道 
	(5)Android系统内部相当于已经有一个电台 定义了好多的广播事件  比如外拨电话 短信到来 sd卡状态  电池电量变化....
	(6)谷歌工程师给我们定义了一个组件专门用来接收这些事件的 
	(7)谷歌工程师为什么要设计这样一个组件  目的就是为了方便开发者进行开发 
	(8)javase javaee javame 
	(9)android应用程序里面的电台：系统内置的一个服务，会把事件（电量不足、电量充满、开机启动完成）作为一个广播消息发送其他的接收者；
	   
##2 如何注册广播接收者
1 定义一个类，继承 BroadcastReceiver，实现onReceive()方法  
  当我们配置的action 的事件发生了  onReceive方法就会执行 
	
	 //方法
	public void onReceive(Context context, Intent intent) {
       
    }
2 在清单文件中配置  

	//配置广播接收者
    <receiver
        android:name=".MyReveiver"
		enabled="true"
        android:exported="true">
        <intent-filter>
          <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
        </intent-filter>
    </receiver>
##3_广播接受者案例_  ip拨号器
* 开发广播接收者的步骤：
  	
    1、买个收音机：

		public class OutCallBroadCastReceiver extends BroadcastReceiver {
			public void onReceive(Context context, Intent intent) {
		}

    2、插上电池：

		<receiver android:name="com.itheima.ipcall.OutCallBroadCastReceiver" />

    3、调整到一个频道：

		<intent-filter >
           <action android:name="android.intent.action.NEW_OUTGOING_CALL" />
		</intent-filter>
 
##4_广播接受者案例_ sd卡状态监听(重点)

	测试的时使用2.3的模拟器，4.0之后版本没有卸载、挂载、移除SD卡的功能。

步骤：

	 1、买个收音机
	 2、插上电池
	 3、调整到一个频道	


   配置文件：

		 <receiver android:name="com.itheima.sdlistener.SDBroadCastReceiver">
	            <intent-filter >
	                <action android:name="android.intent.action.MEDIA_MOUNTED" />
	                <action android:name="android.intent.action.MEDIA_UNMOUNTED" />
	                <action android:name="android.intent.action.MEDIA_REMOVED" />
	             <!-- 必须加上data这个属性 -->
	                <data android:scheme="file"/>
	            </intent-filter>
            
        </receiver>

 代码：

	package com.itheima.sdlistener;

	import android.content.BroadcastReceiver;
	import android.content.Context;
	import android.content.Intent;
	import android.widget.Toast;
	
	public class SDBroadCastReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		 //获取当前广播的事件类型
		String action = intent.getAction();
		
		if("android.intent.action.MEDIA_MOUNTED".equals(action)){
			Toast.makeText(context, "已经插上了SD卡.................", 0).show();
		}
		if("android.intent.action.MEDIA_UNMOUNTED".equals(action)){
			Toast.makeText(context, "拔掉了SD卡.................", 0).show();
		}
		
		if("android.intent.action.MEDIA_REMOVED".equals(action)){
			Toast.makeText(context, " 移除了SD卡.................", 0).show();
		}


	
		}
	}
##5_广播接受者案例_开机启动(重点)

步骤：

	 1、买个收音机
	 2、插上电池
	 3、调整到一个频道	

	
要做的事情：让软件开启后关闭不了：

    禁用返回键和最小化键（小房子键）；

* 示例代码
   
 配置文件：

	 <receiver android:name="com.itheima.lesuo.BootCompletedBroadCastReceiver">
            
            <intent-filter >
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
    </application>

 代码：

	package com.itheima.lesuo;

	import android.content.BroadcastReceiver;
	import android.content.Context;
	import android.content.Intent;
	
	public class BootCompletedBroadCastReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		
		//开启mainactivity
		Intent i = new Intent(context,MainActivity.class);
		//告诉activity自己来维护任务栈,如果任务栈没有当前任务，就会重新创建一个任务放入任务栈
		i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
		
		context.startActivity(i);
		System.out.println("***********88888888888启动完成*********************************");
		}

	}
##6 发送自定义广播

 创建广播电台的步骤：

		//创建一个传递消息的意图对象
		Intent intent = new Intent();
		
		//设置要广播的事件类型
		intent.setAction("com.itheima.broadcast.HMSSDT");
		
		//设置广播的消息数据
		intent.putExtra("news", "黑马49期，晚上12点半准时开播.........");
		
		//发送一个广播消息
		sendBroadcast(intent);
##7 有序广播和无序广播

* 无序广播：
 			
		类似新闻联播:广播不可以被终止（abortBroadcast）  数据不可以被修改 
		广播接收者只要注册接收相应的事件类型，就能接收到的广播；
		
	  	//发送一个广播消息（无序广播）
		sendBroadcast(intent);


* 有序广播：
		
		类似中央发送的红头文件  按照优先级一级一级的接收 有序广播可以被终止 数据可以被修改
		当广播把消息发送出去后，消息会根据广播接收者的优先级从高到低一级一级地下发消息。
		可以拦截消息，也可以修改消息。

   发送有序广播：

	Intent intent = new Intent();
		
        intent.setAction("com.itheima.orderedbroadcast.ZYFFNTBT");
        //发送一个有序的广播
        //intent 意图
        //permission 指定接收者需要添加了权限
        //resultReceiver 指定哪个广播接收者最后接到消息
        //scheduler 消息处理器
        //initialCode 给消息指定初始代码
        //initialData 指定消息的数据
        //initialExtras 指定额外的参数
        sendOrderedBroadcast(intent, null, null, null, 1, "国务院开始发放2014年农田补贴:900元", null);

   


  广播接收者的配置文件：

	 <receiver android:name="com.itheima.zf.ProvinceBroadCastReceiver">
            <intent-filter android:priority="1000" >
                <action  android:name="com.itheima.orderedbroadcast.ZYFFNTBT"/>
            </intent-filter>
        </receiver>


  广播接收者的代码：
	
		String info = getResultData();
		System.out.println("---------我是省级人民政府,已经接收到了中央发的消息:"+info);
		
	   //Toast.makeText(context, "我是省级人民政府,已经接收到了中央发的消息:"+info, 0).show();
        setResultData("国务院开始发放2014年农田补贴:400元");
##8 动态注册广播
  
	    //[1]动态的去注册屏幕解锁和锁屏的广播
		screenReceiver = new ScreenReceiver();
		//[2]创建intent-filter对象
		IntentFilter filter = new IntentFilter();
		//[3]添加要注册的action
		filter.addAction("android.intent.action.SCREEN_OFF");
		filter.addAction("android.intent.action.SCREEN_ON");
		//[4]注册广播接收者
		this.registerReceiver(screenReceiver, filter);
		
		//当activity销毁的时候  取消注册广播接收者
		protected void onDestroy() {
		unregisterReceiver(screenReceiver);
		super.onDestroy();
	}
	
##9 特殊广播接收者
- 比如操作特别频繁的广播事件 屏幕的锁屏和解锁 电池电量的变化 这样 的广播接收者在清单文件里面注册无效	