# Twitter集成指南(Android)

|版本|更新日期|修改人|更新说明|
|----|-----------|--------------- |--------------- |
|0.1|14 June, 2018|Gary Yao|大纲起草|
|0.2|20 June, 2018|Jerry Xu|文档内容起草|
|0.2|20 June, 2018|Bruce Lee|社会化分享集成与开发内容补充|


## 1. 文档目的
*该文档旨在帮助缺乏Twitter Android SDK集成经验的开发者，顺利集成SDK，并实现第三方登录和分享。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉Twitter的操作，具备一定的Android开发技能*



## 4. 先决条件
* *ShareSDK开发者账号*
* *Twitter开发者帐号*
* *国内开发者请自备科学上网工具*



## 5. 指南

### 5.1 系统分享设置与集成

#### Twitter应用管理平台
*点击进入[https://apps.twitter.com/](https://apps.twitter.com/)，注册账号并登陆后，点击Create New App，如下图：*

[![Twitter app management](/images/1CE7A02A-C834-4FF5-8134-DCBC0286F478.png)](https://apps.twitter.com/)

*填写好Name,Description,Website和Callback URLs（选填，但网址必须与SDK集成时在gradle中配置的callbackUri保持一致，可默认为[https://mob.com/](https://mob.com/)），如下图：*

![Create New App](/images/6D4BBCA1-B34F-4590-9721-CCBEE1DA49C8.png)

*点击Create your Twitter application，创建好应用之后会进入新应用详情页面，在新页面中点击顶部导航栏中的Keys and Access Tokens，进入下图：*

![Keys and Access Tokens](/images/A8808475-D65C-4BDC-A9BB-C1D6B23E4E3E.png)

*记录下上图中的Consumer Key (API Key)和Consumer Secret (API Secret)（SDK集成会用到）*

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

在ShareSDK->devInfo下添加需要配置的Twitter的appKey、appSecret及callbackUri(登录或分享后的回调网址，需要与Twitter中注册的Callback URL网址域名一致)

Onekeyshare是ShareSDK的GUI界面，如果不需要，则需要添加”gui false”，因为默认是使用gui，version字段为SDK的版本号，不设置则使用最新的版本；
```
MobSDK {
    appKey "2663a8be0c7c2"
    appSecret "56a8c28a0f80872c3044b10a9c0caea5"

    ShareSDK {
        devInfo {
            Twitter {
                appKey "JYqWP7ARsaEB7CWMW36BsSnEZ" //Twitter Consumer Key (API Key)
                appSecret "FvTq2TtxlVi15XOSVQQ9uFx5jg6drSYl3UuNRJwRcf9I9CxiV2" //Twitter Consumer Secret (API Secret)
                callbackUri "https://mob.com" //Twitter中注册的Callback URLs
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
public void onLoginTwitter(View view) {
        Platform plat = ShareSDK.getPlatform(Twitter.NAME);
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
*利用文字，截图，代码详细说明如何 设置、集成和开发*



