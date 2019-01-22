# 友盟统计App集成指南(iOS)
## 1. 文档目的
*该文档旨在帮助缺乏友盟统计SDK集成经验的开发者，顺利集成SDK。*


## 2. 目标读者
* *iOS开发工程师*
* *项目经理*



## 3. 预备知识
*具备一定的iOS开发技能*



## 4. 先决条件
* *友盟账号*
* *申请友盟统计appid*

## 5. 友盟统计集成

适用范围

该文档适用于海棠版SDK6.0.0及以上版本。

UMeng Analytics iOS SDK适用于iOS 7.0及以上操作系统。

集成准备

创建应用并获取AppKey

AppKey是友盟+后台用来标示App的唯一标识符，集成SDK前需要在创建应用并获取相应的AppKey。

请开发者到友盟+官网注册自己的账号，创建运用程序并获取对应的AppKey.


快速集成

手动集成

依赖库

CoreTelephony.framework    获取运营商标识

libz.tbd  数据压缩

libsqlite.tbd  数据缓存

SystemConfiguration.framework  判断网络状态
工程配置

集成步骤如下：

选择SDK功能组件并下载，解压.zip文件得到相应组件包(例如：UMCommon.framework，UMAnalytics.framework， UMPush.framework等)。

XcodeFile —> Add Files to "Your Project"，在弹出Panel选中所下载组件包－>Add。（注：选中“Copy items if needed”）



添加依赖库，在项目设置target -> 选项卡General ->Linked Frameworks and Libraries如下：
CoreTelephony.framework    获取运营商标识
libz.tbd  数据压缩
libsqlite.tbd  数据缓存
SystemConfiguration.framework  判断网络状态

U-APP基础功能

初始化

1.说明和用途

组件化SDK用来聚合(移动统计 SDK，消息推送 SDK，社会化分享 SDK等)业务的初始化接口，方便对各个业务的初始化统一调用。

2.接口函数

接口:

/** 初始化友盟所有组件产品
@param appKey 开发者在友盟官网申请的AppKey.
@param channel 渠道标识，可设置nil表示"App Store".
*/
+ (void)initWithAppkey:(NSString *)appKey channel:(NSString *)channel;

3.示例代码

#import <UMCommon/UMCommon.h>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [UMConfigure initWithAppkey:@"Your Appkey" channel:@"App Store"];
}

场景设置

1.说明和用途

组件化统计SDK用户不同业务场景设置（目前支持普通统计场景和游戏场景）

游戏场景需设置，否则调用相对应的统计api无法使用

2.接口函数

接口:

代码:
/** 设置 统计场景类型，默认为普通应用统计：E_UM_NORMAL
@param 游戏统计设置为：E_UM_GAME
*/
+ (void)setScenarioType:(eScenarioType)eSType;
参数：
eSType    eScenarioType    设置适用场景    默认为普通应用场景,目前还支持游戏统计场景
注意：如需要支持多个场景，则以 | 分隔。
代码:
[MobClick setScenarioType:E_UM_NORMAL];//支持普通场景

3.示例代码
代码:
[MobClick setScenarioType:E_UM_GAME];//支持游戏场景


设置日志

1.说明和用途

设置是否在console输出sdk的log信息。

2.接口函数
代码:
/** 设置是否在console输出SDK的log信息.
@param bFlag 默认NO(不输出log); 设置为YES, 输出可供调试参考的log信息. 发布产品时必须设置为NO.
*/
+ (void)setLogEnabled:(BOOL)bFlag;

用户传入的AppKey不合法，请到官网申请AppKey，以免影响自己App的统计数据。： 指提示开发者的错误信息，帮助开发者找到错误原因。

3.示例代码
代码:
#import <UMCommonLog/UMCommonLogHeaders.h>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// Override point for customization after application launch.
//开发者需要显式的调用此函数，日志系统才能工作
[UMCommonLogManager setUpUMCommonLogManager];
}

错误收集

1.说明和用途
代码:
收集app适用过程中产生的Crash信息,统计SDK默认是开启Crash收集机制的
2.接口函数
代码:
** 开启CrashReport收集, 默认YES(开启状态).
@param value 设置为NO,可关闭友盟CrashReport收集功能.
@return void.
*/
+ (void)setCrashReportEnabled:(BOOL)value;
3.示例代码
代码:
```
[MobClick setCrashReportEnabled:NO];   // 关闭Crash收集
```
**注意：**
此函数需在common sdk初始化前适用，默认是开启状态


设置加密

1.说明和用途
设置是否对要发送的统计信息进行加密
2.接口函数
代码:
/** 设置是否对统计信息进行加密传输, 默认NO(不加密).
@param value 设置为YES, umeng SDK 会将日志信息做加密处理
*/
+ (void)setEncryptEnabled:(BOOL)value;
3.示例代码
代码:
#import <UMCommon/UMCommon.h>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
[UMConfigure setEncryptEnabled:YES];//打开加密传输
[UMConfigure setLogEnabled:YES];//设置打开日志
[UMConfigure initWith
:@"Your AppKey" channel:@"App Store"];
}

自定义事件可以实现在应用程序中埋点来统计用户的点击行为。自定义事件目前包括“计数事件”和“计算事件”。

使用自定义事件功能请先登陆友盟+官网，在“统计分析->设置->事件”（子账户由于权限限制可能无法看到“设置”选项，请联系主帐号开通权限。）页面中添加相应的事件id然后服务器才会对相应的事件请求进行处理。

注意事项：

若您使用计数事件，请在页面注册的时候选择“多参数类型事件”类型，若使用计算事件，选择“计算事件”类型即可；
event id或者key请使用（英文、数字、下划线、中划线、小数点及加号）进行定义，使用其中一种或者几种都可以，为保证数据计算的准确性，非这些“合法”以外的字符无法添加
event id长度不能超过128个字节，key不能超过128个字节，value不能超过256个字节
Id、ts、du是保留字段，不能作为event id 及key的名称
每个应用至多添加500个event，一个event下支持100个key同时计算，若超过100个，需要开发者手动指定哪些参数需要计算
一个key下支持可传1000个value，value不建议使用特殊字符定义（建议使用标准ASCII码中可见字符部分进行定义）
如需要统计支付金额、使用时长等数值型的连续变量，请使用计算事件（不允许通过key-value结构来统计类似搜索关键词，网页链接等随机生成的字符串信息）
为方便使用者理解及使用，可通过显示名称进行重命名（支持中文），进入【应用设置-事件-编辑】进行操作；
埋码完成后，建议使用集成测试进行验证
注：event为事件，key为参数，value为参数值

接口函数

计数事件

使用计数事件需要在后台添加事件时选择“计数事件”。

代码:
+ (void)event:(NSString *)eventId; 
+ (void)event:(NSString *)eventId label:(NSString *)label; 
+ (void)event:(NSString *)eventId attributes:(NSDictionary *)attributes;
+ (void)event:(NSString *)eventId attributes:(NSDictionary *)attributes counter:(int)number;
参数：
eventId    NSString    网站上注册的事件Id    
label    NSString    分类标签    不同的标签会分别进行统计，方便同一事件的不同标签的对比,为nil或空字符串时后台会生成和eventId同名的标签。
attributes    NSDictionary    自定义属性    属性中的key－value必须为String类型, 每个应用至多添加500个自定义事件，key不能超过100个。
counter    int    自定义数值

示例：统计微博应用中”转发”事件发生的次数，那么在转发的函数里调用:
代码:
[MobClick event:@"Forward"];

示例：统计电商应用中“购买”事件发生的次数，以及购买的商品类型及数量，那么在购买的函数里调用：
代码:
NSDictionary *dict = @{@"type" : @"book", @"quantity" : @"3"};
[MobClick event:@"purchase" attributes:dict];
计算事件

使用计算事件需要在后台添加事件时选择“计算事件”（字符串型）。

代码:
+ (void)beginEvent:(NSString *)eventId;
+ (void)endEvent:(NSString *)eventId;
+ (void)beginEvent:(NSString *)eventId label:(NSString *)label;
+ (void)endEvent:(NSString *)eventId label:(NSString *)label;
+ (void)beginEvent:(NSString *)eventId primarykey :(NSString *)keyName attributes:(NSDictionary *)attributes;
+ (void)endEvent:(NSString *)eventId primarykey:(NSString *)keyName;
+ (void)event:(NSString *)eventId durations:(int)millisecond;
+ (void)event:(NSString *)eventId label:(NSString *)label durations:(int)millisecond;
+ (void)event:(NSString *)eventId attributes:(NSDictionary *)attributes durations:(int)millisecond;
使用计算事件需要在后台添加事件时选择“计算事件”（数值型）。

代码:
+ (void)event:(NSString *)eventId attributes:(NSDictionary *)attributes counter:(int)number;
参数：
eventId    NSString    网站上注册的事件Id    
label    NSString    分类标签    不同的标签会分别进行统计，方便同一事件的不同标签的对比,为nil或空字符串时后台会生成和eventId同名的标签。
attributes    NSDictionary    自定义属性    属性中的key－value必须为String类型, 每个应用至多添加500个自定义事件，key不能超过100个 。
primarykey    NSString    唯一标识    这个参数用于和event_id一起标示一个唯一事件，并不会被统计；对于同一个事件在beginEvent和endEvent 中要传递相同的eventId 和 primarykey。
durations    int    时长    传入自己统计的时长（单位：毫秒）。
注意：

beginEvent需要和endEvent配对使用，需要传入相同的eventId。

示例：购买《Swift Fundamentals》这本书，花了110元

代码:
[MobClick event:@"pay" attributes:@{@"book" : @"Swift Fundamentals"} counter:110]


页面统计

1.说明和用途

用户统计app页面信息

2.接口函数

代码:
+ (void)logPageView:(NSString *)pageName seconds:(int)seconds;
+ (void)beginLogPageView:(NSString *)pageName;
+ (void)endLogPageView:(NSString *)pageName;
参数：

pageName    NSString    统计的页面名称    
seconds    int    时长    单位为秒
注意：

必须配对调用beginLogPageView:和endLogPageView:两个函数来完成自动统计，若只调用某一个函数不会生成有效数据；

在该页面展示时调用beginLogPageView:，当退出该页面时调用endLogPageView。

3.示例代码

在ViewController类的viewWillAppear: 和 viewWillDisappear:中配对调用如下方法：

代码:
- (void)viewWillAppear:(BOOL)animated
{
[super viewWillAppear:animated];
[MobClick beginLogPageView:@"Pagename"]; //("Pagename"为页面名称，可自定义)
}
代码:
- (void)viewWillDisappear:(BOOL)animated 
{
[super viewWillDisappear:animated];
[MobClick endLogPageView:@"Pagename"];
}

账号统计

1.说明和用途

用于用户进行账户统计

2.接口函数

代码:
+ (void)profileSignInWithPUID:(NSString *)puid;
+ (void)profileSignInWithPUID:(NSString *)puid provider:(NSString *)provider;
+ (void)profileSignOff;
参数：

puid    NSString    用户ID    
provider    NSString    账号来源    不能以下划线”_”开头，使用大写字母和数字标识; 如果是上市公司，建议使用股票代码。
3.示例代码

友盟+在统计用户时以设备为标准，若需要统计应用自身的账号，下述两种API任选其一接口：

代码:
// PUID：用户账号ID.长度小于64字节
// Provider：账号来源。不能以下划线"_"开头，使用大写字母和数字标识，长度小于32 字节 ; 
[MobClick profileSignInWithPUID:@"UserID"];
[MobClick profileSignInWithPUID:@"UserID" provider:@"WB"];
Signoff调用后，不再发送账号内容。

代码:
[MobClick profileSignOff];


预置事件

1.说明和用途

对预置事件增加属性，可以设置持久化的预置事件属性，增加预置事件属性。

预置事件属性会保存在系统存储区，并在用户后续行为的预置事件中，自动包含该属性。这样分析数据时，就可按照“姓名”“年龄”对用户注册及后续行为进行查看和筛选，目前预置事件属性最大支持10个。

2.接口函数

代码:
+(void) registerPreProperties:(NSDictionary *)property;
+(void) unregisterPreProperty:(NSString *)propertyName;
+(NSDictionary *)getPreProperties;
+(void)clearPreProperties;
3.示例代码

代码:
// 设置预置事件属性, 标记预置事件属性后,该用户后续触发的所有预置事件的行为事件都将自动包含这些属性；且这些属性存入系统文件，APP重启后仍然存在。
[MobClick registerPreProperties:@{@"user": @"sanzhang"}];
/*针对同一预置事件属性，设定的新值会改写旧值。*/
// 获取所以预置事件属性
NSDictionary *allSPProMap = [MobClick getPreProperties];
// 删除某一个预置事件属性
[MobClick unregisterPreProperty:@"user"];
// 删除所有预置事件属性
[MobClick clearPreProperties];
自动采集页面设置

1.说明和用途

用于设置是否自动采集页面，默认是不自动采集，此接口只能设置一次，建议在初始化之前设置。

2.接口函数

代码:
+ (void)setAutoPageEnabled:(BOOL)value;
3.示例代码

代码:
//设置为自动采集页面
[MobClick setAutoPageEnabled:YES];
功能调试

集成测试

集成测试是通过收集和展示已注册测试设备发送的日志，来检验SDK集成有效性和完整性的一个服务。 所有由注册设备发送的应用日志将实时地进行展示，您可以方便地查看包括应用版本、渠道名称、自定义事件、页面访问情况等数据，提升集成与调试的工作效率。

注: 使用集成测试的设备，其测试数据不会进入应用正式的统计后台，只在“管理—集成测试—实时日志”里查看

测试过程中在Xcode的console窗口查看日志信息，可以打开日志模式：

代码:
[UMConfigure setLogEnabled:YES];
使用集成测试服务请点击集成测试。


发送策略

设置发送策略说明

发送策略设定了用户产生的数据发送回友盟+服务器的频率，此发送策略的数据都是离线计算。

iOS平台数据发送策略包括BATCH（启动时发送）和SEND_INTERVAL（按间隔发送）两种，友盟+默认使用退出时发送（更省流量）

组件化SDK不同以以前非组件化的SDK，用户现在不需要在SDK端显式的设置发送策略。 组件化SDK默认使用BATCH（启动时发送），减少用户的网络发送请求。 同时在用户做前后台切换的时候，组件化SDK也会触发网络请求，批量的把数据发送出去，以节约网络请求的流量。

启动时发送：新增、活跃、启动次数、使用时长、自定义事件等数据在APP本次启动或退出时即刻发送，错误统计产生的消息数据会在下次启动应用时发送。如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。
按间隔发送：按特定间隔发送数据，间隔时长介于90秒与1天之间。新增、活跃、启动次数等数据在APP本次打开时即刻发送，使用时长、自定义事件、错误统计等在使用过程中产生的所有数据都按间隔发送，如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。
发送策略设置方法

在后台 统计分析->设置->发送策略 页面自定义发送间隔。

注意:

（1）在没有获取到在线配置时，默认使用启动时发送的策略；

（2）在打开debug调试模式或者使用集成测试时，不受发送策略控制。

（3）在iOS应用中，可以通过代码设置发送策略，也可以在后台进行设置，后台配置的优先级高于本地配置（即代码中的配置）



