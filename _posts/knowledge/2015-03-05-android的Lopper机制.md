---
layout: post
title: 'Android--多线程之Looper - 承香墨影 - 博客园'
category: opinion
description: 随便写点什么东西 
...


该博文转载自[Android--多线程之Looper](http://www.cnblogs.com/plokmju/p/android_Looper.html)
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

    package com.example.handlerlooperdemo;
    
    import android.app.Activity;
    import android.os.Bundle;
    import android.os.Handler;
    import android.os.Looper;
    import android.os.Message;
    import android.view.View;
    import android.widget.Button;
    import android.widget.TextView;
    
    public class MainThreadActivity extends Activity {
        private Button btnSendToUI1, btnSendToUI2;
        private TextView tvSendMes1;
        private static MyHandler handler;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.looper_activity);
            tvSendMes1 = (TextView) findViewById(R.id.tvSendMes1);
            btnSendToUI1 = (Button) findViewById(R.id.btnSendToUI1);
            btnSendToUI2 = (Button) findViewById(R.id.btnSendToUI2);
    
            btnSendToUI1.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    // 使用Activity内部的Looper对象
                    handler=new MyHandler();
                    Message msg=Message.obtain();
                    msg.what=1;
                    msg.obj="默认Looper";
                    handler.sendMessage(msg);
                }
            });
            btnSendToUI2.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    // 获取当前线程的Looper
                    Looper looper=Looper.myLooper();
                    handler=new MyHandler(looper);
                    
                    Message msg=Message.obtain();
                    msg.what=2;
                    msg.obj="使用当前线程的Looper";
                    handler.sendMessage(msg);
                }
            });
        }
    
        public class MyHandler extends Handler {
    
            public MyHandler() {
            }
    
            public MyHandler(Looper looper) {
                super(looper);
            }
    
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                tvSendMes1.setText("what=" + msg.what + "," + msg.obj);
            }
        }
    }

　　实现效果：

![](http://images.cnitblog.com/blog/234895/201308/19102753-5a284ae16f7b4b1fbee294d97a636e16.x-png)![](http://images.cnitblog.com/blog/234895/201308/19102818-97b7beba9e4146bb95f84a1dbfb06d0e.x-png)

　　通过上面的Demo可以看到，虽然Handler有两个构造函数，一个需要传递一个Looper对象，一个不需要，但是对于UI线程而言，使用哪个其实效果是一样的，因为Activity会帮我们维护Looper对象。

　　而对于子线程而言，没有任何对象帮助我们维护Looper对象，所以需要我们自己手动维护。这时候就需要使用prepare()和loop()方法了。下面通过一个Demo来演示一下在UI线程中给子线程发送消息的实现。注意，因为这个Handler是在子线程中声明的，所以在Handler中处理消息的实现，也必须符合子线程的限制，一些无法在子线程中做的事情，也无法这里实现，比如访问网络、操作UI组件。

　　实现代码：

    package com.example.handlerlooperdemo;
    
    import android.app.Activity;
    import android.os.Bundle;
    import android.os.Handler;
    import android.os.Looper;
    import android.os.Message;
    import android.util.Log;
    import android.view.View;
    import android.widget.Button;
    import android.widget.Toast;
    
    public class WorkThreadActivity extends Activity {
        private Button btnSendToWorkUI;
        private Handler handler;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.looper_activity2);
            
            // 在UI线程中开启一个子线程
            new Thread(new Runnable() {            
                @Override
                public void run() {
                    // 在子线程中初始化一个Looper对象
                    Looper.prepare();
                    handler=new Handler(){
                        @Override
                        public void handleMessage(Message msg) {
                            super.handleMessage(msg);
                            // 把UI线程发送来的消息显示到屏幕上。
                            Log.i("main", "what="+msg.what+","+msg.obj);
                            Toast.makeText(WorkThreadActivity.this, "what="+msg.what+","+msg.obj, Toast.LENGTH_SHORT).show();
                        }
                    };    
                    // 把刚才初始化的Looper对象运行起来，循环消息队列的消息
                    Looper.loop();
                }
            }).start();
            
            btnSendToWorkUI=(Button)findViewById(R.id.btnSendToWorkUI);
            
            btnSendToWorkUI.setOnClickListener(new View.OnClickListener() {            
                @Override
                public void onClick(View v) {
                    // onClick方法是运行在UI线程上的 
                    Message msg=Message.obtain();
                    msg.what=1;
                    msg.obj="向子线程中发送消息！";
                    // 向子线程中发送消息
                    handler.sendMessage(msg);
                }
            });
        }
        
        
    }

　　效果展示：

![](http://images.cnitblog.com/blog/234895/201308/19103827-695a76142b1a4bfdb2105da710520a53.x-png)

 　　[源码下载](http://download.csdn.net/detail/plokmju88/5973243)

 

**总结**

　　本篇博客讲解了Handler在发送消息的原理，并且也说明了如何在UI线程和子线程中互相通信的方式，虽然在实际开发过程中，很少会用到UI线程向子线程发送消息的实现，但是通过这个例子也更好的说明了Looper的原理。

