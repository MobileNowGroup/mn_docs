# Facebook集成指南(Android)

|版本|更新日期|修改人|更新说明|
|----|-----------|--------------- |--------------- |
|0.1|14 June, 2018|Gary Yao|大纲起草|
|0.1|14 June, 2018|Jerry Xu|文档内容起草|


## 1. 文档目的
*该文档旨在帮助缺乏Facebook Android SDK集成经验的开发者，顺利集成SDK，并实现第三方登录和分享。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉Facebook的操作，具备一定的Android开发技能*



## 4. 先决条件
* *1、ShareSDK开发者账号*
* *2、Facebook开发者帐号*
* *3、国内开发者请自备科学上网工具*



## 5. 指南

### 5.1 系统分享设置与集成

#### Facebook开发者平台
* *点击进入[https://developers.facebook.com/](https://developers.facebook.com/)，注册账号并登陆后，点击添加应用，如下图：*
[![Facebook Developer Web](/images/1404CEDD-58D8-4FE0-A081-D6EB0F71920C.png)](https://developers.facebook.com/)

* *填写好名称后进入应用配置界面，如下图：*
![App Key](/images/7468706D-F507-4696-9F13-AAB221E0D1A1.png)

*记录下应用编号和应用密钥，稍后会将其配置在项目应用中*

* *记录下应用编号和应用密钥（SDK集成会用到），再点击该界面下方的添加平台，填写好包名、类名及密钥散列，如下图：*
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
*利用文字，截图，代码详细说明如何 设置、集成和开发*




### 5.3 社会化分享集成与开发
*利用文字，截图，代码详细说明如何 设置、集成和开发*



