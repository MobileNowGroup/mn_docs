# WhatsApp集成指南(Android)



## 1. 文档目的
*该文档旨在帮助缺乏WhatsApp Android SDK集成经验的开发者，顺利集成SDK，并实现分享。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉WhatsApp 的操作，具备一定的Android开发技能*



## 4. 先决条件
* *ShareSDK开发者账号*
* *WhatsApp 帐号(只需要一个普通的WhatsApp帐号,不需要开发者帐号)*
* *国内开发者请自备科学上网工具*



## 5. 指南

### 5.1 系统分享设置与集成

#### WhatsApp测试帐号
下载WhatsApp app 或者在电脑端注册一个普通用户帐号即可。

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

WhatsApp没有第三方登录(不支持第三方登录).


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
在弹出来的对话框中选择WhatsApp即可跳转到WhatsApp进行分享操作。

[![App设置](/images/whatsapp1.jpg)]

点击WhatsApp

[![App设置](/images/whatsapp2.jpg)]

点击我的动态

[![App设置](/images/whatsapp3.jpg)]

输入分享内容，点击下面的三角形发送按钮，然后登录WhatsApp,即可看到分享的内容

[![App设置](/images/whatsapp4.jpg)]

