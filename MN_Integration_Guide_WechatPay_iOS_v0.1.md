# 微信支付集成指南(iOS)
##  1.文档目的
**该文档旨在帮助缺乏支付宝App支付 SDK集成经验的开发者，顺利集成SDK，并实现支付功能。**
## 2. 目标读者
* **iOS开发工程师**
* **项目经理**

## 3. 预备知识
**熟悉微信的操作，具备一定的iOS开发技能**

## 4.下载微信SDK
前往[微信开放平台](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417694084&token=&lang=zh_CN)下载 SDK，PS:如果集成了友盟分享里的微信，那就不用下载,也不用配置环境，因为配置友盟分享的时候已经把微信支付的环境都配置好了
## 5.导入库集成SDK
### *5.1 导入SDK库*
导入上面那个iOS头文件和库下载中的SDK包，然后需要链接上依赖库，在Target —> BuildPhases —> Link Binary With Libraries— 点击+号 -> 搜索你需要的系统库。

* SystemConfiguration.framework
* libz.tbd
* libsqlite3.0.tbd
* CoreTelephony.framework
* QuartzCore.framework
* libc++.tbd
* Security.framework
* CFNetwork.framework
![weixin-iOS2.png](/images/weixin-iOS2.png)
### 5.3 配置Build Setting
1.在你的工程文件中选择Build Setting，在”Other Linker Flags”中加入”-Objc -all_load”（如下图）
![weixin-iOS4.png](/images/weixin-iOS4.png)

2.在Search Paths中添加 libWeChatSDK.a ，WXApi.h，WXApiObject.h，文件所在位置（如下图）
![weixin-iOS5.png](/images/weixin-iOS5.png)

### 5.2 *设置URL Scheme*
商户在微信开放平台申请开发APP应用后，微信开放平台会生成APP的唯一标识APPID，在 [APP端开发步骤](https://link.jianshu.com/?t=https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_5) 里面说得很清楚了，需要填在URL Schemes这个地方。
![weixin-iOS3.png](/images/weixin-iOS3.png)

### 5.3 配置微信白名单
在Xcode中，选择你的工程设置项，选中“TARGETS”一栏，在“info”标签栏的“LSApplicationQueriesSchemes“添加“weixin“、 “wechat”（如下图所示）。
![weixin-iOS6.png](/images/weixin-iOS6.png)

## 6.代码集成
1. 在你需要使 用微信终端API的文件中import WXApi.h 头文件，并增加 WXApiDelegate 协议。
```
#import "AppDelegate.h"
#import "WXApi.h"

@interface AppDelegate ()<WXApiDelegate>

@end

@implementation AppDelegate
```
2. 在 AppDelegate 的 didFinishLaunchingWithOptions 函数中向微信注册id
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    [WXApi registerApp:@"your appId"];
    return YES;
}
```
3. 重写AppDelegate的handleOpenURL和openURL方法：
```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    return [WXApi handleOpenURL:url delegate:self];
}
```
4. 实现WXApiDelegate协议，获取微信支付SDK的返回结果 
```
-(void) onResp:(BaseResp*)resp
{
    //启动微信支付的response
    NSString *payResoult = [NSString stringWithFormat:@"errcode:%d", resp.errCode];
    if([resp isKindOfClass:[PayResp class]]){
        //支付返回结果，实际支付结果需要去微信服务器端查询
        switch (resp.errCode) {
            case 0:
                payResoult = @"支付结果：成功！";
                break;
            case -1:
                payResoult = @"支付结果：失败！";
                break;
            case -2:
                payResoult = @"用户已经退出支付！";
                break;
            default:
                payResoult = [NSString stringWithFormat:@"支付结果：失败！retcode = %d, retstr = %@", resp.errCode,resp.errStr];
                break;
        }
    }
}
```
6. 在调用微信支付类里面，首先增加头文件引用。
```
#import "WXApi.h"
```
5. 在微信支付前，需要先判断手机是否安装微信客户端和当前微信版本是否支持微信支付
```
/// 判断手机有没有微信
    if (WXApi.isWXAppInstalled == NO) {
        NSLog(@"您的手机上没有安装微信");
    }else if (WXApi.isWXAppSupportApi == NO){
        NSLog(@"当前微信版本不支持微信支付");
    }
```
6. 在调起支付的方法中，需要上传的参数包括：appid、partid（商户号）、prepayid（预支付订单ID）、noncestr（参与签名的随机字符串）、timestamp（参与签名的时间戳）、sign（签名字符串）这六个。代码如下：
```
#pragma mark 微信支付方法
- (void)wechatPay{
    
    //需要创建这个支付对象
    PayReq *req   = [[PayReq alloc] init];
    //由用户微信号和AppID组成的唯一标识，用于校验微信用户
    req.openID = appid;
    // 商家id，在注册的时候给的
    req.partnerId = partnerid;
    // 预支付订单这个是后台跟微信服务器交互后，微信服务器传给你们服务器的，你们服务器再传给你
    req.prepayId  = prepayid;
    // 根据财付通文档填写的数据和签名
    req.package  = package;
    // 随机编码，为了防止重复的，在后台生成
    req.nonceStr  = noncestr;
    // 这个是时间戳，也是在后台生成的，为了验证支付的
    NSString * stamp = timestamp;
    req.timeStamp = stamp.intValue;
    // 这个签名也是后台做的
    req.sign = sign;
    //发送请求到微信，等待微信返回onResp
    [WXApi sendReq:req];
    
}
```
到这里微信支付的集成工作就基本完成了。
