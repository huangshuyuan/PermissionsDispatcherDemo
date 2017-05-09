# PermissionsDispatcher2.3.2使用

Android6.0权限官网
https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html?hl=zh-cn

系统权限：
https://developer.android.com/training/permissions/index.html?hl=zh-cn

权限的最佳做法：
https://developer.android.com/training/permissions/best-practices.html?hl=zh-cn#testing

该库的github地址
https://github.com/hotchemi/PermissionsDispatcher

可使用另外的权限库（严振杰）AndPermission
https://github.com/yanzhenjie/AndPermission


![权限](http://upload-images.jianshu.io/upload_images/3805053-e0fc4e86864bec50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Gradle配置
使用PermissionsDispatcher，需要在project的 build.gradle中添加

***

### （1）当Studio的版本在2.2之上
在app module中的build.gradle中添加：

```
dependencies {
  compile 'com.github.hotchemi:permissionsdispatcher:${latest.version}'
  annotationProcessor 'com.github.hotchemi:permissionsdispatcher-processor:${latest.version}'
}
```


目前${latest.version}
 最新的是2.3.2。

***

### （2）当Studio的版低于2.2

在工程目录下build.gradle 文件中添加：
```
buildscript {  
  dependencies {  
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'  
  }  
}  
```
然后在app module中的build.gradle中添加：（必须在app module中添加）

```

apply plugin: 'android-apt'  
  
dependencies {  
  compile 'com.github.hotchemi:permissionsdispatcher:${latest.version}'  
  apt 'com.github.hotchemi:permissionsdispatcher-processor:${latest.version}'  
}  
```

## 用法：
#### 1.注解

PermissionsDispatcher只介绍几个注解，保持其通用API简洁：


注：带注释的方法一定不能private。

|注解 |	需要	  |   描述|
| :-------- | ---------------:| :--: |
|@RuntimePermissions	|  ✓	|在Activity或者Fragment中需要添加，来处理权限的问题
|@NeedsPermission	|✓   	|注释其执行需要一个或多个许可的作用的方法(当用户授予了权限之后，会调用使用此注解的方法)|
|@OnShowRationale	||	注释这解释了为什么需要许可/秒/方法。它通过在一个PermissionRequest可用于继续或中止在用户输入的当前的许可请求对象|
|@OnPermissionDenied	||	注释这是调用的方法，如果用户不授予的权限|
|@OnNeverAskAgain	||	注释如果用户选择让设备“不再询问”有关许可被调用的方法|



具体使用如下：
```

@RuntimePermissions
public class MainActivity extends AppCompatActivity {

    @NeedsPermission(Manifest.permission.CAMERA)
    void showCamera() {
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
                .addToBackStack("camera")
                .commitAllowingStateLoss();
    }

    @OnShowRationale(Manifest.permission.CAMERA)
    void showRationaleForCamera(final PermissionRequest request) {
        new AlertDialog.Builder(this)
            .setMessage(R.string.permission_camera_rationale)
            .setPositiveButton(R.string.button_allow, (dialog, button) -> request.proceed())
            .setNegativeButton(R.string.button_deny, (dialog, button) -> request.cancel())
            .show();
    }

    @OnPermissionDenied(Manifest.permission.CAMERA)
    void showDeniedForCamera() {
        Toast.makeText(this, R.string.permission_camera_denied, Toast.LENGTH_SHORT).show();
    }

    @OnNeverAskAgain(Manifest.permission.CAMERA)
    void showNeverAskForCamera() {
        Toast.makeText(this, R.string.permission_camera_neverask, Toast.LENGTH_SHORT).show();
    }
}
```
#### 2.自动生成的类
Activity继承了AppCompatActivity，是的，如果使用PermissionsDispatcher进行权限管理，那么Activity就要继承AppCompatActivity。这就要使用到了兼容包里的类了。同样此时相应Activity中使用的主题，也需要进行修改，修改成相应兼容包里的主题。
在编译时，PermissionsDispatcher产生的一类MainActivityPermissionsDispatcher（[活动名称] + PermissionsDispatcher），您可以使用安全地访问这些许可保护的方法。

MainActivityPermissionsDispatcher需要自己编译才会有：

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    findViewById(R.id.button_camera).setOnClickListener(v -> {
      // NOTE: delegate the permission handling to generated method
      MainActivityPermissionsDispatcher.showCameraWithCheck(this);
    });
    findViewById(R.id.button_contacts).setOnClickListener(v -> {
      // NOTE: delegate the permission handling to generated method
      MainActivityPermissionsDispatcher.showContactsWithCheck(this);
    });
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    // NOTE: delegate the permission handling to generated method
    MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
}

```
#### 添加SDK支持版本的两种方式：
* 1、AndroidManifest
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="18" />
```
* 2、在注解的时候添加sdk版本控制

```

@RuntimePermissions
public class MainActivity extends AppCompatActivity {

    @NeedsPermission(value = Manifest.permission.WRITE_EXTERNAL_STORAGE, maxSdkVersion = 18)
    void getStorage() {
        // ...
    }
    
}
```

# 使用步骤：
#####  一、在Manifest中添加权限

```
  <uses-permission android:name="android.permission.CAMERA" />
```
##### 二、在Activity中添加注解

```

@RuntimePermissions
public class MainActivity extends AppCompatActivity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }

    /**
     * 显示相机权限
     */

    @NeedsPermission(Manifest.permission.CAMERA)
    void showCamera() {//处理当用户允许该权限时需要处理的方法
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
                .addToBackStack("camera")
                .commitAllowingStateLoss();


    }


    @OnShowRationale(Manifest.permission.CAMERA)
    void showRationaleForCamera(final PermissionRequest request) {// 提示用户权限使用的对话框
        new AlertDialog
                .Builder(this)
                .setMessage("是否打开权限")
                .setPositiveButton("允许", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        request.proceed();
                    }
                })
                .setNegativeButton("拒绝", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        request.cancel();
                    }
                })
                .show();
    }

    /**
     * 如果用户拒绝该权限执行的方法
     */
    @OnPermissionDenied(Manifest.permission.CAMERA)
    void showDeniedForCamera() {
        Toast.makeText(this, "获取权限失败", Toast.LENGTH_SHORT).show();
    }

    @OnNeverAskAgain(Manifest.permission.CAMERA)
    void showNeverAskForCamera() {
        Toast.makeText(this, "再次获取权限", Toast.LENGTH_SHORT).show();
    }
```
#####  三、重写回调方法，并且使用MainActivityPermissionsDispatcher（此方法编译生成【Activity】+PermissionsDispatcher）

```
  /**
     * 权限请求回调，提示用户之后，用户点击“允许”或者“拒绝”之后调用此方法
     *
     * @param requestCode  定义的权限编码
     * @param permissions  权限名称
     * @param grantResults 允许/拒绝
     */
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
    }


    @OnClick({R.id.button_camera, R.id.button_contacts})
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.button_camera:
//                show(this, "相机");
//                默认是没有此类的，需要编译下才会有此类
                MainActivityPermissionsDispatcher.showCameraWithCheck(this);

                break;
       
        }
    }
```
***

## 注意
* 使用到的权限需要在Mnifest里面注册
* PermissionsDispatcher依赖于support-v4由默认库，以便能够使用一些权限compat的类。
* 需要添加support-v13库一起PermissionsDispatcher在您的项目，它将使原生片段支持

***
***
***
***
***
# 原生6.0权限使用
### Android 6.0 变更


另请参阅
[Android 6.0 API 概览](https://developer.android.com/about/versions/marshmallow/android-6.0.html?hl=zh-cn)

Android 6.0（API 级别 23）除了提供诸多新特性和功能外，还对系统和 API 行为做出了各种变更。
如果您之前发布过 Android 应用，请注意您的应用可能受到这些平台变更的影响。
#####运行时权限
此版本引入了一种新的权限模式，如今，用户可直接在运行时管理应用权限。这种模式让用户能够更好地了解和控制权限，同时为应用开发者精简了安装和自动更新过程。用户可为所安装的各个应用分别授予或撤销权限。
对于以 Android 6.0（API 级别 23）或更高版本为目标平台的应用，请务必在运行时检查和请求权限。要确定您的应用是否已被授予权限，请调用新增的 [checkSelfPermission()](https://developer.android.com/reference/android/content/Context.html?hl=zh-cn#checkSelfPermission(java.lang.String))
 方法。要请求权限，请调用新增的 [requestPermissions()](https://developer.android.com/reference/android/app/Activity.html?hl=zh-cn#requestPermissions(java.lang.String[], int))
 方法。即使您的应用并不以 Android 6.0（API 级别 23）为目标平台，您也应该在新权限模式下测试您的应用。
#### 使用步骤

###### 1、在AndroidManifest文件中添加需要的权限。

这个步骤和我们之前的开发并没有什么变化，试图去申请一个没有声明的权限可能会导致程序崩溃。

###### 2、检查权限
```
if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {
}else{
    //
}
```
这里涉及到一个API，ContextCompat.checkSelfPermission，主要用于检测某个权限是否已经被授予，方法返回值为PackageManager.PERMISSION_DENIED或者PackageManager.PERMISSION_GRANTED。当返回DENIED就需要进行申请授权了。

###### 3、申请授权
```
 ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);
```
该方法是异步的，第一个参数是Context；第二个参数是需要申请的权限的字符串数组；第三个参数为requestCode，主要用于回调的时候检测。可以从方法名requestPermissions以及第二个参数看出，是支持一次性申请多个权限的，系统会通过对话框逐一询问用户是否授权。

###### 4、处理权限申请回调
```
@Override
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted, yay! Do the
                // contacts-related task you need to do.

            } else {

                // permission denied, boo! Disable the
                // functionality that depends on this permission.
            }
            return;
        }
    }
}
```
首先验证requestCode定位到你的申请，然后验证grantResults对应于申请的结果，这里的数组对应于申请时的第二个权限字符串数组。如果你同时申请两个权限，那么grantResults的length就为2，分别记录你两个权限的申请结果。如果申请成功，就可以做你的事情了~
