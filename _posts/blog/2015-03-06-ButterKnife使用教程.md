---
layout: post
title: ButterKnife使用教程
description: 教你如何快速上手Butterknife
category: blog
---

### 一、简介

使用ButterKnife框架，可以使你从大量的findViewById中解脱出来，更专注于业务逻辑的实现。

**Butterknife的特性：**

* 支持 Activity 中的 View 注入

* 支持 View 中的 View 注入

* 支持 View 事件回调函数注入

**目前支持如下事件回调函数：**

- View: @OnLongClick and @OnFocusChanged.

- TextView: @OnEditorAction.

- AdapterView: @OnItemClick and @OnItemLongClick.

- CompoundButton: @OnCheckedChanged.

### 二、安装及配置

1.去[官网](http://jakewharton.github.io/butterknife/)或者[github](https://github.com/JakeWharton/butterknife)下载jar包（目前最新版本为：butterknife-6.1.0.jar），放到工程的libs目录，如果你使用的是Android Studio，在app目录下的build.gradle中添加如下依赖即可：

```
dependencies {  
    compile fileTree(dir: 'libs', include: ['*.jar'])  
    compile 'com.android.support:appcompat-v7:21.0.2'  
    compile 'com.jakewharton:butterknife:6.1.0'  
}  

```

2.IDE中的配置

2.1 Eclipse中的配置

右键单击工程，选择properties--->java compile---> Annotation Processing，勾选 "Enable project specific settings"。如下图所示：

![eclipse配置1](/images/ButterKnife/eclipse1.png)

然后，点开Annotation Processing，选中Factory Path，勾选 "Enable project specific settings" ，选择 "Add JARs…"添加你下载的butterknife jar包，如下所示：

![eclipse配置2](/images/ButterKnife/eclipse2.png)

2.2 IntelliJ IDEA中的配置

需要开启AnnotationProcessor配置。如下图：

![idea配置1](/images/ButterKnife/idea.jpg)

### 三、使用

没使用butterknife之前，你的代码如下：

```
public class Main extends Activity {
    EditText editInfo;
    Button btnOK;
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        editInfo = (EditText) findViewById(R.id.edit_info);
        btnOK = (Button) findViewById(R.id.btn_ok);
        editInfo.setText("Hello World");
        btnOK.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String str = editInfo.getText().toString();
                if("Hello World".equals(str)) {
                    editInfo.setText("Hello Java");
                    return;
                }
                if("Hello Java".equals(str)) {
                    editInfo.setText("Hello World");
                    return;
                }
            }
        });
    }    
}

```
使用butterknife之后，你的代码可简写如下所示：

```
public class Main extends Activity {
    @InjectView(R.id.edit_info)
    EditText editInfo;
    @InjectView(R.id.btn_ok)
    Button btnOK;
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);       
        ButterKnife.inject(this);
        editInfo.setText("Hello World")      
    }
   @OnClick(R.id.btn_ok)
    public void changeText() {
        String str = editInfo.getText().toString();
        if("Hello World".equals(str)) {
            editInfo.setText("Hello Java");
            return;
        }
        if("Hello Java".equals(str)) {
            editInfo.setText("Hello World");
            return;
        }
    }
}

```

一些示例代码如下所示：

**在Activity 中注入**

```
class ExampleActivity extends Activity {
   @InjectView(R.id.title) TextView title;
   @InjectView(R.id.subtitle) TextView subtitle;
   @InjectView(R.id.footer) TextView footer;
   @Override 
   public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.simple_activity);
      ButterKnife.inject(this);
      // TODO Use "injected" views...
  }
}

```

**在Fragment中注入**

```
public class FancyFragment extends Fragment {
    @InjectView(R.id.button1) Button button1;
    @InjectView(R.id.button2) Button button2;
    @Override 
    View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fancy_fragment, container, false);
        ButterKnife.inject(this, view);
        // TODO Use "injected" views...
        return view;
    }
}

```
**在ViewHolder中注入**

```
public class MyAdapter extends BaseAdapter {
    @Override 
    public View getView(int position, View view, ViewGroup parent) {
        ViewHolder holder;
        if (view != null) {
            holder = (ViewHolder) view.getTag();
        } else {
            view = inflater.inflate(R.layout.whatever, parent, false);
            holder = new ViewHolder(view);
            view.setTag(holder);
       }
 
        holder.name.setText("John Doe");
        // etc...
        return convertView;
    }
 
    static class ViewHolder {
        @InjectView(R.id.title) TextView name;
        @InjectView(R.id.job_title) TextView jobTitle;
        public ViewHolder(View view) {
            ButterKnife.inject(this, view);
        }
    }
}

```
**注入回调函数**

下面是几种注入回调函数的方法示例：

```
// 带有 Button 参数
@OnClick(R.id.submit)
public void sayHi(Button button) {
    button.setText("Hello!");
} 
// 不带参数
@OnClick(R.id.submit)
public void submit() {
    // TODO submit data to server...
}
// 同时注入多个 View 事件
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
    if (door.hasPrizeBehind()) {
        Toast.makeText(this, "You win!", LENGTH_SHORT).show();
    } else {
        Toast.makeText(this, "Try again", LENGTH_SHORT).show();
    }
}
```
**Reset函数**

如果需要在**界面**销毁的时候，把注入的View设置为 Null，则可以用 reset 函数：

```
public class FancyFragment extends Fragment {
    @InjectView(R.id.button1) Button button1;
    @InjectView(R.id.button2) Button button2;
    @Override 
    View onCreateView(LayoutInflater inflater,ViewGroup group, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fancy_fragment, container, false);
        ButterKnife.inject(this, view);
        // TODO Use "injected" views...
        return view;
    }
    @Override void onDestroyView() {
        super.onDestroyView();
        Views.reset(this);
    }
}

```

另外，还支持**可选**的 View注入，如果该 View 没有，也没关系：

```
@Optional @InjectView(R.id.might_not_be_there) TextView mightNotBeThere;
@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
    // TODO ...
}
```

### 四、参考文献

* [eclipse中的详细配置](http://jakewharton.github.io/butterknife/ide-eclipse.html)
* [Butterknife----View注入框架](http://stormzhang.com/openandroid/android/2014/01/12/android-butterknife/)
* [使用InjectView和findViewById说拜拜](http://www.mzule.com/%E4%BD%BF%E7%94%A8injectview%E5%92%8Cfindviewbyid%E8%AF%B4%E6%8B%9C%E6%8B%9C/)
* [简述Butterknife示例和原理](http://blog.csdn.net/guijiaoba/article/details/43020059)


