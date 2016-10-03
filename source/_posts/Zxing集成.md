---
title: Zxing集成
date: 2016/10/3 17:27:37 
categories: Android
tags: 
    - Android
    - 第三方
---

快速集成二维码扫描库，使用了[一片枫叶的专栏](http://blog.csdn.net/qq_23547831/article/details/52037710)的集成库

本文的项目地址是在：android-zxingLibrary。

使用说明

	可打开默认二维码扫描页面
	
	支持对图片Bitmap的扫描功能
	
	支持对UI的定制化操作
	
	支持对条形码的扫描功能
	
	支持生成二维码操作
	
	支持控制闪光灯开关

使用方式：

集成默认的二维码扫描页面
在具体介绍该扫描库之前我们先看一下其具体的使用方式，看看是不是几行代码就可以集成二维码扫描的功能。

在module的build.gradle中执行compile操作
```java
compile 'cn.yipianfengye.android:zxing-library:1.9'
```
在Application中执行初始化操作
```java
@Override
    public void onCreate() {
        super.onCreate();

        ZXingLibrary.initDisplayOpinion(this);
    }
```

在代码中执行打开扫描二维码界面操作

```java
/**
 * 打开默认二维码扫描界面
 */
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
        startActivityForResult(intent, REQUEST_CODE);
    }
});
```

这里的REQUEST_CODE是我们定义的int型常量。

在Activity的onActivityResult方法中接收扫描结果
```java
/**
         * 处理二维码扫描结果
         */
        if (requestCode == REQUEST_CODE) {
            //处理扫描结果（在界面上显示）
            if (null != data) {
                Bundle bundle = data.getExtras();
                if (bundle == null) {
                    return;
                }
                if (bundle.getInt(CodeUtils.RESULT_TYPE) == CodeUtils.RESULT_SUCCESS) {
                    String result = bundle.getString(CodeUtils.RESULT_STRING);
                    Toast.makeText(this, "解析结果:" + result, Toast.LENGTH_LONG).show();
                } else if (bundle.getInt(CodeUtils.RESULT_TYPE) == CodeUtils.RESULT_FAILED) {
                    Toast.makeText(MainActivity.this, "解析二维码失败", Toast.LENGTH_LONG).show();
                }
            }
        }
```

怎么样是不是很简单？下面我们可以来看一下具体的执行效果：

执行效果：

![](http://oegt3xt9o.bkt.clouddn.com/20160727170831543-1)

但是这样的话是不是太简单了，如果我想选择图片解析呢？别急，对二维码图片的解析也是支持的

集成对二维码图片的解析功能

调用系统API打开图库
```java
Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
intent.addCategory(Intent.CATEGORY_OPENABLE);
intent.setType("image/*");
startActivityForResult(intent, REQUEST_IMAGE);
```
在Activity的onActivityResult方法中获取用户选中的图片并调用二维码图片解析API
```java
if (requestCode == REQUEST_IMAGE) {
            if (data != null) {
                Uri uri = data.getData();
                ContentResolver cr = getContentResolver();
                try {
                    Bitmap mBitmap = MediaStore.Images.Media.getBitmap(cr, uri);//显得到bitmap图片

                    CodeUtils.analyzeBitmap(mBitmap, new CodeUtils.AnalyzeCallback() {
                        @Override
                        public void onAnalyzeSuccess(Bitmap mBitmap, String result) {
                            Toast.makeText(MainActivity.this, "解析结果:" + result, Toast.LENGTH_LONG).show();
                        }

                        @Override
                        public void onAnalyzeFailed() {
                            Toast.makeText(MainActivity.this, "解析二维码失败", Toast.LENGTH_LONG).show();
                        }
                    });

                    if (mBitmap != null) {
                        mBitmap.recycle();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
```
执行效果
![](http://oegt3xt9o.bkt.clouddn.com/20160802150716190-2)

有了默认的二维码扫描界面，也有了对二维码图片的解析，可能有的同学会说如果我想定制化显示UI怎么办呢？没关系也支持滴。

定制化显示扫描UI
由于我们的扫描组件是通过Fragment实现的，所以能够很轻松的实现扫描UI的定制化。

在新的Activity中定义Layout布局文件
```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_second"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/second_button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="取消"
        android:layout_marginTop="20dp"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginBottom="10dp"
        android:layout_gravity="bottom|center_horizontal"
        />

    <FrameLayout
        android:id="@+id/fl_my_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        ></FrameLayout>

</FrameLayout>
```
启动id为fl_my_container的FrameLayout就是我们需要替换的扫描组件，也就是说我们会将我们定义的扫描Fragment替换到id为fl_my_container的FrameLayout的位置。而上面的button是我们添加的一个额外的控件，在这里你可以添加任意的控件，各种UI效果等。具体可以看下面在Activity的初始化过程。

在Activity中执行Fragment的初始化操作
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        /**
         * 执行扫面Fragment的初始化操作
         */
        CaptureFragment captureFragment = new CaptureFragment();
        // 为二维码扫描界面设置定制化界面
        CodeUtils.setFragmentArgs(captureFragment, R.layout.my_camera);

        captureFragment.setAnalyzeCallback(analyzeCallback);
        /**
         * 替换我们的扫描控件
         */ getSupportFragmentManager().beginTransaction().replace(R.id.fl_my_container, captureFragment).commit();
    }
```

其中analyzeCallback是我们定义的扫描回调函数，其具体的定义：
```java
/**
     * 二维码解析回调函数
     */
    CodeUtils.AnalyzeCallback analyzeCallback = new CodeUtils.AnalyzeCallback() {
        @Override
        public void onAnalyzeSuccess(Bitmap mBitmap, String result) {
            Intent resultIntent = new Intent();
            Bundle bundle = new Bundle();
            bundle.putInt(CodeUtils.RESULT_TYPE, CodeUtils.RESULT_SUCCESS);
            bundle.putString(CodeUtils.RESULT_STRING, result);
            resultIntent.putExtras(bundle);
            SecondActivity.this.setResult(RESULT_OK, resultIntent);
            SecondActivity.this.finish();
        }

        @Override
        public void onAnalyzeFailed() {
            Intent resultIntent = new Intent();
            Bundle bundle = new Bundle();
            bundle.putInt(CodeUtils.RESULT_TYPE, CodeUtils.RESULT_FAILED);
            bundle.putString(CodeUtils.RESULT_STRING, "");
            resultIntent.putExtras(bundle);
            SecondActivity.this.setResult(RESULT_OK, resultIntent);
            SecondActivity.this.finish();
        }
    };
```

仔细看的话，你会发现我们调用了CondeUtils.setFragmentArgs方法，该方法主要用于修改扫描界面扫描框与透明框相对位置的，与若不调用的话，其会显示默认的组件效果，而如果调用该方法的话，可以修改扫描框与透明框的相对位置等UI效果，我们可以看一下my_camera布局文件的实现。
```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent" >

    <SurfaceView
        android:id="@+id/preview_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

    <com.uuzuche.lib_zxing.view.ViewfinderView
        android:id="@+id/viewfinder_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:inner_width="200dp"
        app:inner_height="200dp"
        app:inner_margintop="150dp"
        app:inner_corner_color="@color/scan_corner_color"
        app:inner_corner_length="30dp"
        app:inner_corner_width="5dp"
        app:inner_scan_bitmap="@drawable/scan_image"
        app:inner_scan_speed="10"
        />

</FrameLayout>
```

上面我们自定义的扫描控件的布局文件，下面我们看一下默认的扫描控件的布局文件：
```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent" >

    <SurfaceView
        android:id="@+id/preview_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

    <com.uuzuche.lib_zxing.view.ViewfinderView
        android:id="@+id/viewfinder_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

</FrameLayout>
```

可以发现其主要的区别就是在自定义的扫描控件中多了几个自定义的扫描框属性：
```java
<declare-styleable name="innerrect">
        <attr name="inner_width" format="dimension"/><!-- 控制扫描框的宽度 -->
        <attr name="inner_height" format="dimension"/><!-- 控制扫描框的高度 -->
        <attr name="inner_margintop" format="dimension" /><!-- 控制扫描框距离顶部的距离 -->
        <attr name="inner_corner_color" format="color" /><!-- 控制扫描框四角的颜色 -->
        <attr name="inner_corner_length" format="dimension" /><!-- 控制扫描框四角的长度 -->
        <attr name="inner_corner_width" format="dimension" /><!-- 控制扫描框四角的宽度 -->
        <attr name="inner_scan_bitmap" format="reference" /><!-- 控制扫描图 -->
        <attr name="inner_scan_speed" format="integer" /><!-- 控制扫描速度 -->
    </declare-styleable>
```

通过以上几个属性我们就可以定制化的显示我们的扫描UI了，比如定制化微信扫描UI：

执行效果
![](http://oegt3xt9o.bkt.clouddn.com/20160802151453053-3)

当然了如果以上的以上，你还是对定制化UI方面不太满意，可以直接下载我的项目，然后引入lib-zxing module作为你的module，直接修改其代码。

生成二维码图片

生成带Logo的二维码图片：
```java

/**
         * 生成二维码图片
         */
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String textContent = editText.getText().toString();
                if (TextUtils.isEmpty(textContent)) {
                    Toast.makeText(ThreeActivity.this, "您的输入为空!", Toast.LENGTH_SHORT).show();
                    return;
                }
                editText.setText("");
                mBitmap = CodeUtils.createImage(textContent, 400, 400, BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
                imageView.setImageBitmap(mBitmap);
            }
        });
```

生成不带logo的二维码图片
```java
/**
         * 生成不带logo的二维码图片
         */
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                String textContent = editText.getText().toString();
                if (TextUtils.isEmpty(textContent)) {
                    Toast.makeText(ThreeActivity.this, "您的输入为空!", Toast.LENGTH_SHORT).show();
                    return;
                }
                editText.setText("");
                mBitmap = CodeUtils.createImage(textContent, 400, 400, null);
                imageView.setImageBitmap(mBitmap);
            }
        });
```

执行效果
![](http://oegt3xt9o.bkt.clouddn.com/20160802152820667-4)

支持控制闪光灯
```java
/**
 * 打开闪光灯
 */
CodeUtils.isLightEnable(true);
/**
 * 关闭闪光灯
 */
 CodeUtils.isLightEnable(false);
```

创建二维码库的流程：

创建一个android studio项目，并创建zxingLibrary库；
![](http://oegt3xt9o.bkt.clouddn.com/20160726180250175-5)

在zxingLibrary库中实现二维码的扫描操作；
在实现过程中有以下几个注意点：

（1）由于二维码扫描界面是一个Activity，所以需要在AndroidManifest.xml中定义，而我为了减少对项目的依赖，选择了在zxingLibrary中的AndroidManifest.xml中定义该Activity。
```java
<application android:allowBackup="true"
        >

        <activity
            android:configChanges="orientation|keyboardHidden"
            android:name="com.uuzuche.lib_zxing.activity.CaptureActivity"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="stateAlwaysHidden"
            android:label="扫描二维码"
           android:theme="@style/Theme.AppCompat.NoActionBar"
            ></activity>
    </application>
```
（2）由于扫描操作需要使用摄像头等权限，也在zxingLibrary中的AndroidManifest.xml中申请了部分权限：
```java
<uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.FLASHLIGHT" />

    <uses-feature android:name="android.hardware.camera" />
    <uses-feature android:name="android.hardware.camera.autofocus" />

    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />
```

（3）原来的项目中二维码扫面的部分是作为主项目实现的，部分代码使用了switch（id）的操作，而这样的代码在library中有问题，具体可参考：在Android library中不能使用switch-case语句访问资源ID的原因分析及解决方案，所以我做了修改：

修改之前：
```java
@Override
  public void handleMessage(Message message) {
    switch (message.what) {
      case R.id.decode:
        decode((byte[]) message.obj, message.arg1, message.arg2);
        break;
      case R.id.quit:
        Looper.myLooper().quit();
        break;
    }
  }
```

修改之后：
```java
@Override
  public void handleMessage(Message message) {
    if (message.what == R.id.decode) {
      decode((byte[]) message.obj, message.arg1, message.arg2);
    } else if (message.what == R.id.quit) {
      Looper.myLooper().quit();
    }
  }
```
将zxingLibrary库上传至Jcenter中
为了使用compile，所以讲zxingLibrary库上传至了jCenter中，具体的上传过程可参考我的：[Github项目解析（二）–>将Android项目发布至JCenter代码库](http://blog.csdn.net/qq_23547831/article/details/50017151)

有同学反应扫描过程中，二维码图片有拉伸的现象，最新的compile已经改正，具体原因可参考：[Zxing图片拉伸解决 Android 二维码扫描](http://blog.csdn.net/aaawqqq/article/details/24852915)

总结：

以上就是我实现的这个快速继承二维码扫描库的过程。主要优点就是可以通过android studio快速的继承到我们的项目中，有兴趣的同学可以到github上看一下具体实现。项目地址：android-zxingLibrary