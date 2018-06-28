# Instagram集成指南(Android)

## 1. 文档目的
*该文档旨在帮助缺乏Instagram Android SDK集成经验的开发者，顺利集成SDK，并实现第三方登录和分享。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉Instagram的操作，具备一定的Android开发技能*



## 4. 先决条件
* *ShareSDK开发者账号*
* *Instagram开发者帐号*
* *国内开发者请自备科学上网工具*



## 5. 指南

### 5.1 系统分享设置与集成

#### Instagram开发者平台
*点击进入[https://www.instagram.com/developer/]，注册账号并登陆后，点击添加应用，如下图：*

[![Instagram Developer Web](/images/instagram_register1.png)](https://www.instagram.com/developer)

*点击Register Your Application，如下图：*

![register](/images/instagram_register5.png)

点击Register a New Client，如下图：*

![register](/images/instagram_register2.png)

填写相关资料后,需要注意的是Valid redirect URls那里填的网址需要跟Instagram初始化里redirectUri的参数一样的
填写完成后点击Register.然后我们可以看到这个应用

![register](/images/instagram_register3.png)

点击Manage

![register](/images/instagram_register4.png)


*记录下Client ID和Client Secret,应用注册就完成了。


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

在ShareSDK->devInfo下添加需要配置的Instagram的下Client ID和Client Secret及callbackUri(登录或分享后的回调网址，需要与Instagram中注册的回调地址一致)

Onekeyshare是ShareSDK的GUI界面，如果不需要，则需要添加”gui false”，因为默认是使用gui，version字段为SDK的版本号，不设置则使用最新的版本；
```
MobSDK {
    appKey "2663a8be0c7c2"
    appSecret "56a8c28a0f80872c3044b10a9c0caea5"

    ShareSDK {
        devInfo {
            Instagram {
                appKey "f22aa48b50c6419997807484df5394b1"
                appSecret "5b0e3f7467b942d482711887a97fb743 "
                callbackUri "https://mob.com" // Instagram中配置的回调地址
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
public void onLoginInstagram(View view) {
        Platform plat = ShareSDK.getPlatform(Instagram.NAME);
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
           OnekeyShare oks = new OnekeyShare();
//关闭sso授权
        oks.disableSSOWhenAuthorize();
        oks.setImageUrl("http://www.zc-zm.com:8080/my/ic_launcher.png");

// title标题，印象笔记、邮箱、信息、微信、人人网和QQ空间等使用
        oks.setTitle("标题");
// titleUrl是标题的网络链接，QQ和QQ空间等使用
        oks.setTitleUrl("http://sharesdk.cn");
// text是分享文本，所有平台都需要这个字段
        oks.setText("我是分享文本");
// imagePath是图片的本地路径，Linked-In以外的平台都支持此参数
        oks.setImagePath("/sdcard/test.jpg");//确保SDcard下面存在此张图片
// url仅在微信（包括好友和朋友圈）中使用
        oks.setUrl("http://sharesdk.cn");
// comment是我对这条分享的评论，仅在人人网和QQ空间使用
        oks.setComment("我是测试评论文本");
// site是分享此内容的网站名称，仅在QQ空间使用
        oks.setSite(getString(R.string.app_name));
// siteUrl是分享此内容的网站地址，仅在QQ空间使用
        oks.setSiteUrl("http://sharesdk.cn");

// 启动分享GUI
        oks.show(this);
}
```
然后我们点击分享中的Instagram
![share](/images/instagram_share1.jpg)
点击右边的箭头
![share](/images/instagram_share2.jpg)

注意事项:

    1、Instagram不支持图文分享,只支持图片跟视频的分享,demo中把图片push到sdcard根目录
