---
title: 'Android--多线程之Looper - 承香墨影 - 博客园'
...

[承香墨影](http://www.cnblogs.com/plokmju/)

[Android--多线程之Looper](http://www.cnblogs.com/plokmju/p/android_Looper.html)
-------------------------------------------------------------------------------

**前言**

　　上一篇博客讲解了Handler实现线程间通信，这篇博客讲解一下Handler运行的原理，其中涉及到MessageQueue、Looper。简要来讲，Handler会把一个线程消息发送给当前线程的消息队列，从最后Handler会处理消息队列中的消息，这里的消息队列就是MessageQueue，而管理消息队列的，就是Looper。

　　Handler、MessageQueue、Looper三者间的关系如图：

![](http://images.cnitblog.com/blog/234895/201308/19093258-aa3efb1164ba4959a55cb9b3369b98e0.x-png)

 　　从上图可以看出，Handler发送线程消息到当前线程的MessageQueue中，而Looper用来管理MessageQueue，它从MessageQueue中取到消息交给Handler处理。

 

**Looper**

　　Looper，通常运行在一个消息的循环队列当中，线程在默认的情况下，不会给我们提供一个消息循环去管理消息队列的。如果想管理消息队列，需要在线程当中调用Looper.prepare()方法使消息循环初始化，并且调用Looper.loop()使消息循环一直处于运行状态，直到停止循环。所以Looper主要就是完成MessageQueue与Handler进行交互的功能。

　　在Activity中的UI主线程中，无需使用显式的方式进行Looper的初始化以及开始循环，是因为Activity内部包含一个Looper对象，它会自动管理Looper，处理子线程中发送过来的消息。而初始化Handler的时候，在Handler的构造函数中，会把当前线程的Looper与Handler关联，所以在Activity中，无需显式使用Looper。下面通过一个例子讲解一下Activity中，工作线程给主线程发送消息的例子。

　　实现代码：

     1 package com.example.handlerlooperdemo;
     2 
     3 import android.app.Activity;
     4 import android.os.Bundle;
     5 import android.os.Handler;
     6 import android.os.Looper;
     7 import android.os.Message;
     8 import android.view.View;
     9 import android.widget.Button;
    10 import android.widget.TextView;
    11 
    12 public class MainThreadActivity extends Activity {
    13     private Button btnSendToUI1, btnSendToUI2;
    14     private TextView tvSendMes1;
    15     private static MyHandler handler;
    16 
    17     @Override
    18     protected void onCreate(Bundle savedInstanceState) {
    19         super.onCreate(savedInstanceState);
    20         setContentView(R.layout.looper_activity);
    21         tvSendMes1 = (TextView) findViewById(R.id.tvSendMes1);
    22         btnSendToUI1 = (Button) findViewById(R.id.btnSendToUI1);
    23         btnSendToUI2 = (Button) findViewById(R.id.btnSendToUI2);
    24 
    25         btnSendToUI1.setOnClickListener(new View.OnClickListener() {
    26             @Override
    27             public void onClick(View v) {
    28                 // 使用Activity内部的Looper对象
    29                 handler=new MyHandler();
    30                 Message msg=Message.obtain();
    31                 msg.what=1;
    32                 msg.obj="默认Looper";
    33                 handler.sendMessage(msg);
    34             }
    35         });
    36         btnSendToUI2.setOnClickListener(new View.OnClickListener() {
    37             @Override
    38             public void onClick(View v) {
    39                 // 获取当前线程的Looper
    40                 Looper looper=Looper.myLooper();
    41                 handler=new MyHandler(looper);
    42                 
    43                 Message msg=Message.obtain();
    44                 msg.what=2;
    45                 msg.obj="使用当前线程的Looper";
    46                 handler.sendMessage(msg);
    47             }
    48         });
    49     }
    50 
    51     public class MyHandler extends Handler {
    52 
    53         public MyHandler() {
    54         }
    55 
    56         public MyHandler(Looper looper) {
    57             super(looper);
    58         }
    59 
    60         @Override
    61         public void handleMessage(Message msg) {
    62             super.handleMessage(msg);
    63             tvSendMes1.setText("what=" + msg.what + "," + msg.obj);
    64         }
    65     }
    66 }

　　实现效果：

![](http://images.cnitblog.com/blog/234895/201308/19102753-5a284ae16f7b4b1fbee294d97a636e16.x-png)![](http://images.cnitblog.com/blog/234895/201308/19102818-97b7beba9e4146bb95f84a1dbfb06d0e.x-png)

　　通过上面的Demo可以看到，虽然Handler有两个构造函数，一个需要传递一个Looper对象，一个不需要，但是对于UI线程而言，使用哪个其实效果是一样的，因为Activity会帮我们维护Looper对象。

　　而对于子线程而言，没有任何对象帮助我们维护Looper对象，所以需要我们自己手动维护。这时候就需要使用prepare()和loop()方法了。下面通过一个Demo来演示一下在UI线程中给子线程发送消息的实现。注意，因为这个Handler是在子线程中声明的，所以在Handler中处理消息的实现，也必须符合子线程的限制，一些无法在子线程中做的事情，也无法这里实现，比如访问网络、操作UI组件。

　　实现代码：

     1 package com.example.handlerlooperdemo;
     2 
     3 import android.app.Activity;
     4 import android.os.Bundle;
     5 import android.os.Handler;
     6 import android.os.Looper;
     7 import android.os.Message;
     8 import android.util.Log;
     9 import android.view.View;
    10 import android.widget.Button;
    11 import android.widget.Toast;
    12 
    13 public class WorkThreadActivity extends Activity {
    14     private Button btnSendToWorkUI;
    15     private Handler handler;
    16     @Override
    17     protected void onCreate(Bundle savedInstanceState) {
    18         super.onCreate(savedInstanceState);
    19         setContentView(R.layout.looper_activity2);
    20         
    21         // 在UI线程中开启一个子线程
    22         new Thread(new Runnable() {            
    23             @Override
    24             public void run() {
    25                 // 在子线程中初始化一个Looper对象
    26                 Looper.prepare();
    27                 handler=new Handler(){
    28                     @Override
    29                     public void handleMessage(Message msg) {
    30                         super.handleMessage(msg);
    31                         // 把UI线程发送来的消息显示到屏幕上。
    32                         Log.i("main", "what="+msg.what+","+msg.obj);
    33                         Toast.makeText(WorkThreadActivity.this, "what="+msg.what+","+msg.obj, Toast.LENGTH_SHORT).show();
    34                     }
    35                 };    
    36                 // 把刚才初始化的Looper对象运行起来，循环消息队列的消息
    37                 Looper.loop();
    38             }
    39         }).start();
    40         
    41         btnSendToWorkUI=(Button)findViewById(R.id.btnSendToWorkUI);
    42         
    43         btnSendToWorkUI.setOnClickListener(new View.OnClickListener() {            
    44             @Override
    45             public void onClick(View v) {
    46                 // onClick方法是运行在UI线程上的 
    47                 Message msg=Message.obtain();
    48                 msg.what=1;
    49                 msg.obj="向子线程中发送消息！";
    50                 // 向子线程中发送消息
    51                 handler.sendMessage(msg);
    52             }
    53         });
    54     }
    55     
    56     
    57 }

　　效果展示：

![](http://images.cnitblog.com/blog/234895/201308/19103827-695a76142b1a4bfdb2105da710520a53.x-png)

 　　[源码下载](http://download.csdn.net/detail/plokmju88/5973243)

 

**总结**

　　本篇博客讲解了Handler在发送消息的原理，并且也说明了如何在UI线程和子线程中互相通信的方式，虽然在实际开发过程中，很少会用到UI线程向子线程发送消息的实现，但是通过这个例子也更好的说明了Looper的原理。

 

posted on 2013-08-22 08:10 [承香墨影](http://www.cnblogs.com/plokmju/)
阅读(...) 评论(...)
[编辑](http://i.cnblogs.com/EditPosts.aspx?postid=3266988) [收藏](#)

[刷新评论](javascript:void(0);)[刷新页面](#)[返回顶部](#top)

[博客园首页](http://www.cnblogs.com/ "开发者的网上家园")[博问](http://q.cnblogs.com/ "程序员问答社区")[新闻](http://news.cnblogs.com/ "IT新闻")[闪存](http://home.cnblogs.com/ing/)[程序员招聘](http://job.cnblogs.com/)[知识库](http://kb.cnblogs.com/)

Powered by: \
 [博客园](http://www.cnblogs.com/) \
 Copyright © 承香墨影

### 导航

-   [博客园](http://www.cnblogs.com/)
-   [首页](http://www.cnblogs.com/plokmju/)
-   [新随笔](http://i.cnblogs.com/EditPosts.aspx?opt=1)
-   [联系](http://space.cnblogs.com/msg/send/%e6%89%bf%e9%a6%99%e5%a2%a8%e5%bd%b1)
-   [订阅](http://www.cnblogs.com/plokmju/rss)[![订阅](http://www.cnblogs.com/images/xml.gif)](http://www.cnblogs.com/plokmju/rss)
-   [管理](http://i.cnblogs.com/)

### 统计

-   随笔 - 80
-   文章 - 2
-   评论 - 260
-   引用 - 0

### 公告
