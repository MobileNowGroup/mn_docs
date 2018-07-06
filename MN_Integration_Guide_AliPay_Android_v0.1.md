# 支付宝App集成指南(Android)

## 1. 文档目的
*该文档旨在帮助缺乏支付宝App支付 SDK集成经验的开发者，顺利集成SDK，并实现支付功能。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉支付宝的操作，具备一定的Android开发技能*



## 4. 先决条件
* *支付宝企业开发者账号*
* *申请支付宝手机app支付*



## 5. 指南

### 5.1 帐号申请

#### 支付宝企业帐号申请及app支付申请
*具体请参考下面两个链接：
1、https://jingyan.baidu.com/article/ff411625d3d12412e48237e4.html
该链接大概介绍了集成支付宝app支付的步骤
2、https://jingyan.baidu.com/article/c74d6000bbcf110f6a595d34.html
该链接详细介绍了企业帐号的申请,app支付的申请，并带有图文说明.

### 5.2 支付集成

对于Android客户端,集成支付宝不需要任何支付宝帐号相关的资料.

#### 配置AndroidManifest.xml
1、打开项目根目录的AndroidManifest.xml,首先需要添加如下权限：

```
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
2、如果需要网页支付,也就是如果手机没有安装支付宝客户端的时候,通过网页完成支付功能，在application标签中添加如下配置：

```
<activity
    android:name="com.alipay.sdk.app.H5PayActivity"
    android:configChanges="orientation|keyboardHidden|navigation|screenSize"
    android:exported="false"
    android:screenOrientation="behind"
    android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
 <activity
    android:name="com.alipay.sdk.app.H5AuthActivity"
    android:configChanges="orientation|keyboardHidden|navigation"
    android:exported="false"
    android:screenOrientation="behind"
    android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
```

#### 配置lib
将支付宝sdk添加到依赖,打开项目根目录的build.gradle，在buildscrip–>dependencies 模块下面添加
```
dependencies {
    ......
    compile files('libs/alipaySdk-2.0.jar')
    ......
}
```

#### 添加混淆规则
```
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
-keep class com.alipay.sdk.app.H5PayCallback {
    <fields>;
    <methods>;
}
-keep class com.alipay.android.phone.mrpc.core.** { *; }
-keep class com.alipay.apmobilesecuritysdk.** { *; }
-keep class com.alipay.mobile.framework.service.annotation.** { *; }
-keep class com.alipay.mobilesecuritysdk.face.** { *; }
-keep class com.alipay.tscenter.biz.rpc.** { *; }
-keep class org.json.alipay.** { *; }
-keep class com.alipay.tscenter.** { *; }
-keep class com.ta.utdid2.** { *;}
-keep class com.ut.device.** { *;}
```

#### 代码集成
新建一个AliPay.java用于处理支付宝支付相关的功能
```
package com.mobile.vlog.alipay;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Context;
import android.os.Handler;
import android.os.Message;
import android.text.TextUtils;
import android.widget.Toast;

import com.alipay.sdk.app.PayTask;

import java.util.Map;

public class AliPay {
    private static final int SDK_PAY_FLAG = 1;
    private static final String RESULT_STATUS = "resultStatus";
    private static final String RESULT_STATUS_SUCCESS_CODE = "9000";
    private IAliPayCallBack mCallback;
    private Context mContext;

    public AliPay(Context context, IAliPayCallBack callback) {
        mContext = context;
        mCallback = callback;
    }

    public void sendPayReq(String payment) {
        Runnable payRunnable = () -> {
            PayTask alipay = new PayTask((Activity) mContext);
            Map<String, String> result = alipay.payV2(payment, true);

            Message msg = new Message();
            msg.what = SDK_PAY_FLAG;
            msg.obj = result;
            mHandler.sendMessage(msg);
        };
        // 必须异步调用
        Thread payThread = new Thread(payRunnable);
        payThread.start();
    }

    @SuppressLint("HandlerLeak")
    private Handler mHandler = new Handler() {
        @SuppressWarnings("unused")
        public void handleMessage(Message msg) {
            if (msg.what == SDK_PAY_FLAG) {
                @SuppressWarnings("unchecked") Map<String, String> map = (Map<String, String>) msg.obj;
                /**
                 对于支付结果，请商户依赖服务端的异步通知结果。同步通知结果，仅作为支付结束的通知。
                 */
                String resultStatus = map.get(RESULT_STATUS);
                // 判断resultStatus 为9000则代表支付成功
                if (TextUtils.equals(resultStatus, RESULT_STATUS_SUCCESS_CODE)) {
                    // 该笔订单是否真实支付成功，需要依赖服务端的异步通知。
                    if (mCallback != null) {
                        mCallback.onAliPaySuccess();
                    }
                } else {
                    // 该笔订单真实的支付结果，需要依赖服务端的异步通知。
                    Toast.makeText(mContext, "支付失败.", Toast.LENGTH_SHORT).show();
                    if (mCallback != null) {
                        mCallback.onAliPayFailure();
                    }
                }
            }
        }
    };

    public interface IAliPayCallBack {
        void onAliPaySuccess();

        void onAliPayFailure();
    }
}

```

然后我们就可以在其他地方通过调用sendPayReq()方法完成支付功能sendPayReq的参数一般通过服务器返回,是已经用私钥签名好的订单信息
```
 NetService netService = retrofit.create(NetService.class);
            Call<AliPayModel> call = netService.prePay();
            //进行网络请求
            call.enqueue(new Callback<AliPayModel>() {
                @Override
                public void onResponse(Call<AliPayModel> call,
                                       Response<AliPayModel> response) {
                    if (mAliPay == null) {
                        mAliPay = new AliPay(PayActivity.this, new AliPay
                                .IAliPayCallBack() {
                            @Override
                            public void onAliPaySuccess() {

                            }

                            @Override
                            public void onAliPayFailure() {

                            }
                        });
                    }
                    if (response != null && response.body() != null) {
                        mAliPay.sendPayReq(response.body().getData());
                    }
                }

                @Override
                public void onFailure(Call<AliPayModel> call, Throwable t) {
                    t.printStackTrace();
                }
            });
```
这里通过NetService调用里面的prePay方法从服务器获取支付订单相关的信息,获取到支付订单后调用AliPay类的sendPayReq方法，并把获取到的支付订单信息作为参数传进去，里面会调用支付宝sdk，并把该订单信息传过去.

#### demo效果
首先首页demo是这样的：

![demo](/images/zhifubao1.jpg)

当我们点击支付时,首先通过后台接口获取订单信息,这里测试时获取到的是：
```
charset=utf-8&biz_content=%7B%22timeout_express%22%3A%2230m%22%2C%22product_code%22%3A%22QUICK_MSECURITY_PAY%22%2C%22total_amount%22%3A%220.01%22%2C%22subject%22%3A%22%E6%B5%8B%E8%AF%951%22%2C%22body%22%3A%22%E6%B5%8B%E8%AF%951%22%2C%22out_trade_no%22%3A%22test20180705142512601%22%7D&method=alipay.trade.app.pay&notify_url=http%3A%2F%2F123.57.220.164%3A8080%2Fmy%2Fpay_test%2Fcallback&app_id=2017110909824511&sign_type=RSA2&version=1.0&timestamp=2016-07-29+16%3A55%3A53&sign=6agXX8qhi39rm0vqmyXnkstGIWNS4wznkH9ODEvpTK2IGCgKQviSC%2FkUzcHj41iCbeJAjJw834voRMvTDOErEKuqIZNyOHDSNQ%2FmGKCcC61yeqFpdg2QYSOoc2MOsBQv0ojnefbg6lI%2FmaKiT46fQMUF5%2FUZHnc5hOJXDqRerialFwp60q%2B%2FXmfTFX88JPegKS5B7fcfFLuirp5kkuHr7Qsgi04nVXTVmbCNB0bgmrFQVyV6Ix6ZmHMMY6UFsM3C2hGtfaX4pbIzZDRdM2V1vQzrfovPQFAWozh4SMxeCKAOdLYKeXtJ8El%2B9rFK%2BZljvkHdFpQbJjcet38CjoMGLQ%3D%3D
```
点击支付后如果手机有安装支付宝,则会跳转到支付宝客服端进行支付：

![demo](/images/zhifubao2.jpg)

如果没有安装支付宝客服端，则跳转到支付宝网页进行支付：

![demo](/images/zhifubao3.png)

#### 沙箱环境测试

在调用支付之前调用
```
EnvUtils.setEnv(EnvUtils.EnvEnum.SANDBOX);
```
设置沙箱环境

需要注意的是测试帐号也是需要用沙箱环境的

![demo](/images/zhifubao4.png)

到这里支付宝的集成工作就基本完成了。
