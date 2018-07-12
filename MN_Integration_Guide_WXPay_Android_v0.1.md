微信支付App集成指南(Android)

## 1. 文档目的
*该文档旨在帮助缺乏微信App支付 SDK集成经验的开发者，顺利集成SDK，并实现支付功能。*


## 2. 目标读者
* *Android开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉微信App支付的操作，具备一定的Android开发技能*



## 4. 先决条件
* *微信开放平台开发者账号*
* *微信开放平台申请企业认证*
* *微信开放平台申请App支付*



## 5. 指南

### 5.1 帐号申请

#### 支付宝企业帐号申请及app支付申请
*具体请参考下面两个链接：
1、https://jingyan.baidu.com/article/ff411625d3d12412e48237e4.html
该链接大概介绍了微信开放平台申请
2、http://kf.qq.com/faq/161220Brem2Q161220uUjERB.html
该链接详细介绍了微信开放平台申请企业认证的流程.

### 5.2 支付集成

对于Android客户端,集成微信支付需要微信appid和支付订单信息,支付订单信息一般通过服务器返回.

#### 配置AndroidManifest.xml
1、打开项目根目录的AndroidManifest.xml,首先需要添加如下权限：

```
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
2、添加如下activity,需要注意的是该Activity必须在包名目录下的wxapi包中，在application标签中添加如下配置：

```
 <activity
            android:name=".wxapi.WXPayEntryActivity"
            android:exported="true"
            android:launchMode="singleTop"
            android:theme="@android:style/Theme.NoDisplay"/>
```

#### 配置lib
将支付宝sdk添加到依赖,打开项目根目录的build.gradle，在buildscrip–>dependencies 模块下面添加
```
dependencies {
    ......
    compile files('libs/libammsdk.jar')
    ......
}
```

#### 添加混淆规则
```
-libraryjars libs/libammsdk.jar
-keep class com.tencent.** { *;}

```

#### 代码集成

新建一个WxPay.java用于处理支付宝支付相关的功能
```
package com.vulog.carshare.whed.test.wxapi.pay;


import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;

import com.tencent.mm.sdk.modelpay.PayReq;
import com.tencent.mm.sdk.openapi.IWXAPI;
import com.tencent.mm.sdk.openapi.WXAPIFactory;
import com.vulog.carshare.whed.test.model.WXPayModel;


public class WxPay {
    private IWXAPI mApi;
    private Context mContext;
    private WxPaySuccessListener mWxPaySuccessListener;
    public static final String ACTION_WXPAY_RESP = "ACTION_WXPAY_RESP";
    public final static String WX_ID = "wx21189743c4ef6ebc";

    public WxPay(Context context, WxPaySuccessListener listener) {
        mContext = context;
        mWxPaySuccessListener = listener;
        mApi = WXAPIFactory.createWXAPI(context, WX_ID);
        mApi.registerApp(WX_ID);
        registerBoradcastReceiver();
    }

    private void registerBoradcastReceiver() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(ACTION_WXPAY_RESP);
        // register broadcast
        mContext.registerReceiver(mBroadcastReceiver, intentFilter);
    }

    private BroadcastReceiver mBroadcastReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (action.equals(ACTION_WXPAY_RESP)) {
                boolean isSuccess = intent.getBooleanExtra("isSuccess", false);
                if (isSuccess) {
                    paySuccess();
                } else {
                    payFail();
                }
                destory();
            }
        }
    };

    private void paySuccess() {
        if (mWxPaySuccessListener != null) {
            mWxPaySuccessListener.onWxPaySuccess();
        }
    }

    private void payFail() {
        if (mWxPaySuccessListener != null) {
            mWxPaySuccessListener.onWxPayFail();
        }
    }

    /**
     * call wx pay
     *
     * @param wxpay
     */
    public void sendPayReq(WXPayModel wxpay) {
        PayReq req = new PayReq();
        req.appId = wxpay.getAppid();
        req.partnerId = wxpay.getMch_id();
        req.prepayId = wxpay.getPrepay_id();
        req.nonceStr = wxpay.getNonce_str();
        req.timeStamp = wxpay.getTimestamp();
        req.packageValue = "Sign=WXPay";
        req.sign = wxpay.getSign();
        mApi.sendReq(req);
    }

    public boolean isWXAppInstalled() {
        return mApi.isWXAppInstalled();
    }

    public static String getTempTime() {
        long tempTime = System.currentTimeMillis();
        String str = tempTime + "";
        return str.substring(0, 10);
    }

    public void destory() {
        try {
            mContext.unregisterReceiver(mBroadcastReceiver);
        } catch (Exception e) {

        }
        mApi.detach();
        mApi = null;
        mContext = null;
    }

    public interface WxPaySuccessListener {
         void onWxPaySuccess();

         void onWxPayFail();
    }
}

```
在包名下的wxapi下新建一个WXPayEntryActivity.java用于接收微信支付结果信息
```
package com.vulog.carshare.whed.test.wxapi;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.widget.Toast;

import com.tencent.mm.sdk.constants.ConstantsAPI;
import com.tencent.mm.sdk.modelbase.BaseReq;
import com.tencent.mm.sdk.modelbase.BaseResp;
import com.tencent.mm.sdk.openapi.IWXAPI;
import com.tencent.mm.sdk.openapi.IWXAPIEventHandler;
import com.tencent.mm.sdk.openapi.WXAPIFactory;
import com.vulog.carshare.whed.test.R;
import com.vulog.carshare.whed.test.wxapi.pay.WxPay;

/**
 * Attention: you must implement wxapi.WXPayEntryActivity.java in you package name path(if package
 * name or class name is not right ,you will not get the pay result from wx sdk)
 * https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_5(Android doc point four)
 */

public class WXPayEntryActivity extends Activity implements IWXAPIEventHandler {

    private static final String TAG = "WXPayEntryActivity";

    private IWXAPI api;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_wxpay_result);
        api = WXAPIFactory.createWXAPI(this, WxPay.WX_ID, false);
        //register appid
        api.registerApp(WxPay.WX_ID);
        api.handleIntent(getIntent(), this);
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
        api.handleIntent(intent, this);
    }

    // wx send req will call-back this method
    @Override
    public void onReq(BaseReq req) {

    }

    /***
     * wx will call this method to tell us the pay result
     * errCode : 0  success
     *         : -1 fail(may be sign is error 、appid is not register、appid is not correct 、 other
     *         exception)
     *         : -2 user cancel
     * @param resp
     */
    @Override
    public void onResp(BaseResp resp) {
        Intent intent = new Intent(WxPay.ACTION_WXPAY_RESP);
        boolean isSuccess = false;
        if (resp.errCode == 0) {
            isSuccess = true;
        }
        intent.putExtra("isSuccess", isSuccess);
        sendBroadcast(intent);
        finish();
    }
}

```

然后我们就可以在其他地方通过调用sendPayReq()方法完成支付功能sendPayReq的参数一般通过服务器返回的微信支付的相关参数
需要的参数如下
```
 package com.vulog.carshare.whed.test.model;


import java.io.Serializable;

public class WXPayModel extends BaseModel implements Serializable {
    private String prepay_id;
    private String timestamp;
    private String nonce_str;
    private String mch_id;
    private String package_name;
    private String appid;
    private String sign;
    private String errcode;
    private String errmsg;

    public String getPrepay_id() {
        return prepay_id;
    }

    public void setPrepay_id(String prepay_id) {
        this.prepay_id = prepay_id;
    }

    public String getErrcode() {
        return errcode;
    }

    public void setErrcode(String errcode) {
        this.errcode = errcode;
    }

    public String getErrmsg() {
        return errmsg;
    }

    public void setErrmsg(String errmsg) {
        this.errmsg = errmsg;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(String timestamp) {
        this.timestamp = timestamp;
    }

    public String getNonce_str() {
        return nonce_str;
    }

    public void setNonce_str(String nonce_str) {
        this.nonce_str = nonce_str;
    }

    public String getMch_id() {
        return mch_id;
    }

    public void setMch_id(String mch_id) {
        this.mch_id = mch_id;
    }

    public String getPackage_name() {
        return package_name;
    }

    public void setPackage_name(String package_name) {
        this.package_name = package_name;
    }

    public String getAppid() {
        return appid;
    }

    public void setAppid(String appid) {
        this.appid = appid;
    }

    public String getSign() {
        return sign;
    }

    public void setSign(String sign) {
        this.sign = sign;
    }

}
```
然后就可以调用支付了

```
     public void payWeix(WXPayModel payment, IPayCallBack iPayCallBack) {
        this.iPayCallBack = iPayCallBack;
        mWxPay = null;
        if (mWxPay == null) {
            mWxPay = new WxPay(mActivity, this);
        }
        mWxPay.sendPayReq(payment);
    }
```

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
