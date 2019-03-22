# Android代码混淆
### 1.什么是代码混淆：
```
Android SDK 自带了混淆工具Proguard。它位于SDK根目录\tools\proguard下面。如果开启了混淆，Proguard默认情况下会对所有代码，包括第三方包都进行混淆，可是有些代码或者第三方包是不能混淆的，这就需要我们手动编写混淆规则来保持不能被混淆的部分。
作者：半岛铁盒1979
链接：https://www.jianshu.com/p/e747b538d0f0
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

```
### 2.为什么要混淆：
```
优化java的字节码
减小apk文件的大小，在混淆过程中会删除未使用过的类和成员
代码安全，使类、函数、变量名随机变成无意义的代号形如：a,b,c...之类。防止app被反编译之后能够很容易的看懂代码
```
### 3.怎样使用混淆：
```
在app下面的build.gradle添加使用混淆
```
```
 signingConfigs {
        config {
            storeFile file("./****.jks") //签名文件路径
            storePassword "*******"
            keyAlias "*****"
            keyPassword "*****" //签名密码
        }
    }
    buildTypes {
        debug {
            //在debug环境下不用开启混淆，方便测试
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.config
        }
        release {
            //开启混淆,删除无用代码
            minifyEnabled true
            //开启删除无用资源
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.config
        }
    }
```
### 4.混淆文件 proguard-rules.pro：
```
通用混淆配置（APP通用）
```
```
##################################通过混淆配置#################################
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5

# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses

# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名->混淆后类名的映射关系
-verbose

# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers

# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify
-dontoptimize

# 混淆时是否记录日志
-verbose
-ignorewarnings

# 保留Annotation不混淆
-keepattributes *Annotation*

# 避免混淆泛型
-keepattributes Signature
-keepattributes Exceptions,InnerClasses

-dontnote com.google.vending.licensing.ILicensingService
-dontnote com.android.vending.licensing.ILicensingService

# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable
-keepattributes Deprecated,Synthetic,EnclosingMethod

# 重命名抛出异常时的文件名称
-renamesourcefileattribute SourceFile

# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*
```
### 5.APP需要保留的公共部分（通用）
```
1)  四大组件以及子类；
2)  自定义Application；
3)  support下面的继承子类
4)  R下面的资源
5)  native方法
6)  Activity中参数是view的方法
7)  枚举
8)  自定义View
9)  序列化（Parcelable,Serializable）
10) 带有回调函数（On*Listener,OnEvent）
11) WebView

```
```
#############################################
#
# Android开发中一些需要保留的公共部分
#
#############################################

# 保留我们使用的四大组件，自定义的Application等等这些类不被混淆
# 因为这些子类都有可能被外部调用
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
#-keep public class com.android.vending.licensing.ILicensingService

# 保留support下的所有类及其内部类
-keep class android.support.** {*;}

# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保留R下面的资源
-keep class **.R$* {*;}

# 保留本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}

# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留我们自定义控件（继承自View）不被混淆
-keep public class * extends android.view.View{
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留Parcelable序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留Serializable序列化的类不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 对于带有回调函数的onXXEvent、**On*Listener的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
}

# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, jav.lang.String);
```
### 6.保留自己的项目部分代码不能被混淆（需要根据自己项目）
```
1) 网络请求（如果混淆,就会发生字段的错乱，无法正常解析）
2) 加密类
3) 数据库实体类
4) 工具类
5) 项目中应用到的第三方工具类（如okhttp，eventbus，rxjava等），需要根据具体的工具介绍进行操作
6) 保留lib和compile引用的第三方jar包不被混淆的方法：
-keep class 包名.** { *; } 。
如：保留引用的科大讯飞的第三方jar包不被混淆
-keep class com.iflytek.** { *; }

```
```
#网络请求等与外界通信不能混淆
-keep class com.xxxxx.function.**.net.** { *; }
-keep class com.xxxxx.function.**.bean.** { *; }
-keep class com.xxxxx.common.net.** { *; }
-keep class com.xxxxx.common.bean.** { *; }
#加密不能混淆
-keep class com.xxxxx.crypt.** {*;}

#数据库实体类不能混淆
-keep class com.xxxxxx.function.**.dao.** { *; }
#工具类不混淆
-keep class com.xxxxx.common.utils.** { *; }
#greenDAO 3
#-keepclassmembers class * extends org.greenrobot.greendao.AbstractDao {
#public static java.lang.String TABLENAME;
#}
#-keep class **$Properties
#-dontwarn org.greenrobot.greendao.database.**
#-dontwarn rx.**
#greenDAO 3

# Fresco
#-keep class com.facebook.** {*;}
#-keep interface com.facebook.** {*;}
#-keep enum com.facebook.** {*;}

# OkHttp3
-dontwarn com.squareup.okhttp3.**
-keep class com.squareup.okhttp3.** { *;}
-dontwarn okio.**

#Picasso
#-dontwarn com.squareup.okhttp.**

#zxing
-dontwarn com.google.zxing.**
-keep class com.google.zxing.** { *; }

#webview
-dontwarn  com.tencent.**

#eventbus不能混淆
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }

# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
#如果项目中倒入了de.greenrobot.even的beta1版本，需要添加此行代码
-keepclassmembers class ** {
    public void onEvent*(**);
}

#高德
-dontwarn  com.amap.api.**
-keep class com.amap.api.** {*;}

#bugout
-dontwarn com.qamaster.android.**
-dontwarn com.testin.agent.**
-keepattributes InnerClasses
-keep class com.testin.agent.** { *; }
-keepattributes SourceFile, LineNumberTable

#butterknife
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }
-keepclasseswithmembernames class * {
   @butterknife.* <fields>;
}
-keepclasseswithmembernames class * {
   @butterknife.* <methods>;
}

#sharesdk
-keep class cn.sharesdk.**{*;}
-keep class com.sina.**{*;}
-keep class **.R$* {*;}
-keep class **.R{*;}
-dontwarn cn.sharesdk.**
-dontwarn **.R$*
-dontwarn com.tencent.**
-keep class com.tencent.** {*;}

# rx
-dontwarn rx.**
-keepclassmembers class rx.** { *; }
# retrolambda
-dontwarn java.lang.invoke.*

# Glide specific rules #
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
    **[] $VALUES;
    public *;
}

#友盟统计
-keep class com.umeng.message.protobuffer.* {
     public <fields>;
         public <methods>;
}
-keep class com.umeng.message.* {
     public <fields>;
         public <methods>;
}

#ormLite
-keep public class * extends com.j256.ormlite.android.apptools.OrmLiteSqliteOpenHelper
-keep public class * extends com.j256.ormlite.android.apptools.OpenHelperManager
-keepclassmembers class * {@com.j256.ormlite.field.DatabaseField *;}
-keep class com.j256.ormlite.** {*;}

# Gson specific classes
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.stream.** { *; }

#zbar
-keep class net.sourceforge.zbar.**{*;}

-keep class com.nineoldandroids.** {*;}

#身份证ocr
-keep class com.yd.ocr.idcard.** { *; }
-keep class com.googlecode.** { *; }
-keep class org.opencv.** { *; }

#讯飞语音
-keep class com.iflytek.** { *; }

# FastJson 混淆代码
-dontwarn com.alibaba.fastjson.**
-keep class com.alibaba.fastjson.** { *; }
-keepattributes Signature
-keepattributes *Annotation*
```

