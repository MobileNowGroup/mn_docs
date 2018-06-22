# Facebook集成指南(Android)

|版本|更新日期|修改人|更新说明|
|----|-----------|--------------- |--------------- |
|0.1|14 June, 2018|Gary Yao|大纲起草|
|0.2|19 June, 2018|Jerry Xu|文档内容起草|
|0.2|21 June, 2018|Bruce Lee|社会化分享集成与开发内容补充|


## 1. 文档目的
*该文档旨在帮助缺乏Facebook Android SDK集成经验的开发者，顺利集成SDK，并实现第三方登录和分享。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉Facebook的操作，具备一定的Android开发技能*



## 4. 先决条件
* *ShareSDK开发者账号*
* *Facebook开发者帐号*
* *国内开发者请自备科学上网工具*



## 5. 指南

### 5.1 系统分享设置与集成

#### Facebook开发者平台
*点击进入[https://developers.facebook.com/](https://developers.facebook.com/)，注册账号并登陆后，点击添加应用，如下图：*

[![Facebook Developer Web](/images/1404CEDD-58D8-4FE0-A081-D6EB0F71920C.png)](https://developers.facebook.com/)

*填写好名称后进入应用配置界面，如下图：*

![App Key](/images/7468706D-F507-4696-9F13-AAB221E0D1A1.png)

*记录下应用编号和应用密钥（SDK集成会用到），再点击该界面下方的添加平台，填写好包名、类名及密钥散列，如下图：*

![Android平台配置](/images/24966215-18D9-474C-A91B-3FC77A48DA59.png)

##### *注：生成开发密钥散列*
*每个 Android 开发环境都将会有一个唯一的开发密钥散列。*
##### *Mac 操作系统*
要生成开发密钥散列，请打开一个终端窗口，运行以下命令：
```      
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```
##### *Windows*
您需要以下各项：
* Java 开发包中的密钥和证书管理工具 (keytool)
* [Google Code Archive](https://code.google.com/archive/p/openssl-for-windows/downloads) 的 Windows 版 openssl-for-windows openssl 函数库
要生成开发密钥散列，请在 Java SDK 文件夹的命令提示符中运行以下命令：
```
keytool -exportcert -alias androiddebugkey -keystore "C:\Users\USERNAME\.android\debug.keystore" | "PATH_TO_OPENSSL_LIBRARY\bin\openssl" sha1 -binary | "PATH_TO_OPENSSL_LIBRARY\bin\openssl" base64
```
此命令将针对您的开发环境生成一个包含 28 个字符的唯一密钥散列。将其复制粘贴到下面的字段中。对于参与应用开发的每个人的开发环境，您必须分别提供开发密钥散列。
###### 生成发布密钥散列
Android 应用必须先使用发布密钥进行电子签名，然后才能上传到商店中。要生成发布密钥散列，请在 Mac 或 Windows 内运行以下命令，并替换您的发布密钥别名和 keystore 路径：
```
keytool -exportcert -alias YOUR_RELEASE_KEY_ALIAS -keystore YOUR_RELEASE_KEY_PATH | openssl sha1 -binary | openssl base64
```
这会生成一个包含 28 个字符的字符串，您应将其复制粘贴到下面的字段中。另外，请参阅 [Android 文档](https://developer.android.com/tools/publishing/app-signing.html)，了解有关应用签名的信息。

##### 配置登录授权
点击左侧导航栏中的 产品 -> facebook登录 -> 设置，然后填上有效 OAuth 跳转 URI和取消授权回调网址，如下图所示：

![登录授权配置](/images/72DEC643-F939-46F5-B1B2-72FF3A0F3953.png)

注：URI域名必须一致，且为第三方服务端地址，如果不需要对分享或登录进行回调操作，可默认为[https://mob.com/](https://mob.com/)，SDK集成时需要将此地址配置在gradle中。

#### Mob ShareSDK开发者平台
*点击进入[http://mob.com/](http://mob.com/)，注册账号并登陆进入后台，点击顶部导航栏中的“添加应用”按钮，如下图所示：*

[![添加应用](/images/047A3651-EBF7-418D-96C9-59AFD05B17C6.png)](http://dashboard.mob.com/#!/index)

*进入新添加应用后，点击下图中的“ShareSDK”右侧的“+”*

[![添加ShareSDK](/images/AF346DEA-57C3-40AA-9F41-D40DD93783C9.png)](http://dashboard.mob.com/#!/setup/app)

*然后在弹出的dialog中选择“确定添加”（为新应用开放ShareSDK功能），如下图所示：*

[![确定添加](/images/3B0EDE9E-FA56-4538-93D0-F33D2798AFE1.png)](http://dashboard.mob.com/#!/setup/app)

*最后进入App设置主页，记录下AppKey和App Secret（SDK集成会用到），如下图所示：*

[![App设置](/images/5F5B4845-F9C0-4DFD-9BFB-FE35833F45E2.png)](http://dashboard.mob.com/#!/setup/app)


### 5.2 第三方登录集成与开发

#### 配置gradle
1、打开项目根目录的build.gradle，在buildscrip–>dependencies 模块下面添加  classpath ‘com.mob.sdk:MobSDK:+’，如下所示：

![MobSDK依赖](http://wiki.mob.com/wp-content/uploads/2017/05/123-1.png)

```
buildscript {
    repositories {
        jcenter()
    }
 
    dependencies {
        ...
        classpath 'com.mob.sdk:MobSDK:+'
 
    }
}
```
2、在使用到Mob产品的module下面的build.gradle文件里面添加引用

![MobSDK依赖](http://wiki.mob.com/wp-content/uploads/2017/11/2.jpg)

```
apply plugin: 'com.mob.sdk'
```
3、然后添加MobSDK方法，配置mob的key和秘钥 （与第2步是一个gradle中；注意：MobSDK方法是配置到文件根目录，与android并列，不要配置到android里面）

在ShareSDK->devInfo下添加需要配置的Facebook的appKey、appSecret及callbackUri(登录或分享后的回调网址，需要与Facebook中注册的有效 OAuth 跳转 URI和取消授权回调网址域名一致)

Onekeyshare是ShareSDK的GUI界面，如果不需要，则需要添加”gui false”，因为默认是使用gui，version字段为SDK的版本号，不设置则使用最新的版本；
```
MobSDK {
    appKey "2663a8be0c7c2"
    appSecret "56a8c28a0f80872c3044b10a9c0caea5"

    ShareSDK {
        devInfo {
            Facebook {
                appKey "180293596001540"
                appSecret "2e2f10639084e9d79362cedfbe888cee"
                callbackUri "https://mob.com" // facebook中配置的回调地址
            }
        }
    }
}
```
注：如果您没有在AndroidManifest中设置appliaction的类名，MobSDK会将这个设置为com.mob.MobApplication，但如果您设置了，请在您自己的Application类中调用：
```
MobSDK.init(this);
```

#### 添加代码

如果需要通过第三方登录获取用户的信息，如：头像、用户名等，就需要使用showUser(null)的方法，如果仅仅只需要验证登录的结果，则使用authorize()方法
```
public void onLoginFacebook(View view) {
        Platform plat = ShareSDK.getPlatform(Facebook.NAME);
        plat.removeAccount(true); //移除授权状态和本地缓存，下次授权会重新授权
        plat.SSOSetting(false); //SSO授权，传false默认是客户端授权，没有客户端授权或者不支持客户端授权会跳web授权
        plat.setPlatformActionListener(this);//授权回调监听，监听oncomplete，onerror，oncancel三种状态
        if(plat.isClientValid()){
	        //判断是否存在授权凭条的客户端，true是有客户端，false是无			
        }
        if(plat.isAuthValid()){
            //判断是否已经存在授权状态，可以根据自己的登录逻辑设置
            Toast.makeText(this, "已经授权过了", 0).show();
            return;
        }
        //plat.authorize();	//要功能，不要数据		
        plat.showUser(null);    //要数据不要功能，主要体现在不会重复出现授权界面
    }
```


### 5.3 社会化分享集成与开发

完成上述步骤后，可以使用以下代码来实现内容分享操作：
```
public void shareWithFacebook(View view) {
    final OnekeyShare oks = new OnekeyShare();
    //指定分享的平台，如果为空，还是会调用九宫格的平台列表界面
    oks.setPlatform(Facebook.NAME);
    //关闭sso授权
    oks.disableSSOWhenAuthorize();
    // title标题，印象笔记、邮箱、信息、微信、人人网和QQ空间使用
    oks.setTitle("标题");
    // titleUrl是标题的网络链接，仅在Linked-in,QQ和QQ空间使用
    oks.setTitleUrl("http://sharesdk.cn");
    // text是分享文本，所有平台都需要这个字段
    oks.setText("我是分享文本");
    //分享网络图片，新浪微博分享网络图片需要通过审核后申请高级写入接口，否则请注释掉测试新浪微博
    oks.setImageUrl("http://f1.sharesdk.cn/imgs/2014/02/26/owWpLZo_638x960.jpg");
    // imagePath是图片的本地路径，Linked-In以外的平台都支持此参数
    //oks.setImagePath("/sdcard/test.jpg");//确保SDcard下面存在此张图片
    // url仅在微信（包括好友和朋友圈）中使用
    oks.setUrl("http://sharesdk.cn");
    // comment是我对这条分享的评论，仅在人人网和QQ空间使用
    oks.setComment("我是测试评论文本");
    // site是分享此内容的网站名称，仅在QQ空间使用
    oks.setSite("ShareSDK");
    // siteUrl是分享此内容的网站地址，仅在QQ空间使用
    oks.setSiteUrl("http://sharesdk.cn");

    //启动分享
    oks.show(this);
}
```