# 支付宝App集成指南(iOS)
## 1. 文档目的
*该文档旨在帮助缺乏支付宝App支付 SDK集成经验的开发者，顺利集成SDK，并实现支付功能。*


## 2. 目标读者
* *iOS开发工程师*
* *项目经理*



## 3. 预备知识
*熟悉支付宝的操作，具备一定的iOS开发技能*



## 4. 先决条件
* *支付宝企业开发者账号*
* *申请支付宝手机app支付*


### 5.1 帐号申请

#### 支付宝企业帐号申请及app支付申请
*具体请参考下面两个链接：
1、https://jingyan.baidu.com/article/ff411625d3d12412e48237e4.html
该链接大概介绍了集成支付宝app支付的步骤
2、https://jingyan.baidu.com/article/c74d6000bbcf110f6a595d34.html
该链接详细介绍了企业帐号的申请,app支付的申请，并带有图文说明.

### 5.2 SDK下载
前往下载地址[开放平台文档中心](https://docs.open.alipay.com/54/104509) 下载App支付客户端DEMO&SDK  
### 5.3 前期准备
1. 把下载的的压缩文件中  AlipaySDK.bundle、AlipaySDK.framework 文件拷贝到项目文件夹下，并导入到项目工程中。
2.  在Build Phases选项卡的Link Binary With Libraries中，增加以下依赖：
![Alipay-iOS1](/images/Alipay-iOS1.png)
3. 点击项目名称，点击“Build Settings”选项卡，在搜索框中，以关键字“search”搜索，对“Header Search Paths”增加头文件路径：$(SRCROOT)/项目名称。如果头文件信息已增加，可不必再增加。
![Alipay-iOS2](/images/Alipay-iOS2.png)
4. 配置白名单，在info.plist增加key：LSApplicationQueriesSchemes，类型为NSArray，添加支持的白名单alipayauth、alipay、safepay,类型为String。如下图
![Alipay-iOS3](/images/Alipay-iOS3.png)
5. 点击项目名称，点击“Info”选项卡，在“URL Types”选项中，点击“+”，在“URL Schemes”中输入唯一标识（如: “alisdkdemo”）
![Alipay-iOS4](/images/Alipay-iOS4.png)
### 5.4 代码集成
1. 在需要使用支付宝的开发包类库中，引入头文件
 `# import <AlipaySDK/AlipaySDK.h>`
2. 从服务器获取加签后的订单信息，调用(AlipaySDK *)defaultService类下面的支付接口函数，唤起支付宝支付页面。
```
-(void)payOrder:(NSString *)orderStr
fromScheme:(NSString *)schemeStr
callback:(CompletionBlock)completionBlock
```
appScheme为app在info.plist注册的scheme。`
3. 当这笔交易被买家支付成功后支付宝收银台上显示该笔交易成功，并提示用户“返回”。此时在APAppDelegate.m的 - (BOOL)application:(UIApplication )application openURL:(NSURL )url sourceApplication:(NSString )sourceApplication annotation:(id)annotation 中调用获取返回数据的代码【iOS9.0以上（包括iOS9.0）需要在 - (BOOL)application:(UIApplication )app openURL:(NSURL )url options:(NSDictionary<NSString, id> *)options 中执行 】：
```
 if ([url.host isEqualToString:@"safepay"]) {
        // 支付跳转支付宝钱包进行支付，处理支付结果
        [[AlipaySDK defaultService]processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
           
            NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
            if (resultDic.count > 0) {
                if ([resultDic[@"resultStatus"] integerValue] == 9000) {
                    //支付成功
                } else {
                //支付失败
                }
            } else {
                //支付失败
            }
            
        }];
        
        // 授权跳转支付宝钱包进行支付，处理支付结果
        [[AlipaySDK defaultService] processAuth_V2Result:url standbyCallback:^(NSDictionary *resultDic) {
            NSLog(@"result = %@",resultDic);
            // 解析 auth code
            NSString *result = resultDic[@"result"];
            NSString *authCode = nil;
            if (result.length>0) {
                NSArray *resultArr = [result componentsSeparatedByString:@"&"];
                for (NSString *subResult in resultArr) {
                    if (subResult.length > 10 && [subResult hasPrefix:@"auth_code="]) {
                        authCode = [subResult substringFromIndex:10];
                        break;
                    }
                }
            }
            NSLog(@"授权结果 authCode = %@", authCode?:@"");
        }];
    }
```
到这里支付宝的集成工作就基本完成了。
