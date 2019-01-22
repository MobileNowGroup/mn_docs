# Google Analytics（分析）App集成指南(iOS)
## 1. 文档目的
*该文档旨在帮助缺乏Google Analytics SDK集成经验的开发者，顺利集成SDK。*


## 2. 目标读者
* *iOS开发工程师*
* *项目经理*



## 3. 预备知识
*具备一定的iOS开发技能*



## 4. 先决条件
* *google账号*
* *申请Google Analytics（分析）跟踪 ID*

## 5. Google Analytics集成
本指南介绍了如何将 Google Analytics（分析）添加到您的 iOS 应用以衡量用户在已命名屏幕上的活动。如果您目前没有应用，而是仅仅想了解一下 Google Analytics（分析）的工作原理，请参阅我们的示例应用。

注意：开始使用 iOS 版 Google Analytics（分析）SDK 3.16 版之前，要求安装 Xcode 7.3 或更高版本。

Google Analytics（分析）使用 CocoaPods 来安装和管理依赖关系。打开终端窗口，然后导航至您应用的 Xcode 项目所在的位置。如果您还没有为应用创建 Podfile，请立即创建一个。

pod init
打开为应用创建的 Podfile，然后添加以下内容：

pod 'GoogleAnalytics'
保存该文件并运行：

pod install
这将为您的应用创建一个 .xcworkspace 文件。在以后开发您的应用时都要使用此文件。

针对应用初始化 Google Analytics（分析）

现在，为您的项目获取配置文件后，您就可以开始实现自己的方案了。首先，在 AppDelegate 中配置共享的 Google Analytics（分析）对象。这样，您的应用就可以将数据发送到 Google Analytics（分析）了。您需要执行以下操作：

添加必要的标头。
在 didFinishLaunchingWithOptions 内设置 Google Analytics（分析）跟踪器。
将 YOUR_TRACKING_ID 替换为您自己的 Google Analytics（分析）跟踪 ID，例如 UA-47605289-8。
发送异常和日志信息（可选）。
要进行上述更改，先在 AppDelegate 内添加 Google Analytics（分析）：

#import <GoogleAnalytics/GAI.h>
#import <GoogleAnalytics/GAIDictionaryBuilder.h>
然后，替换 didFinishLaunchingWithOptions 方法以配置 Google Analytics（分析）：

GAI *gai = [GAI sharedInstance];
[gai trackerWithTrackingId:@"YOUR_TRACKING_ID"];

// Optional: automatically report uncaught exceptions.
gai.trackUncaughtExceptions = YES;

// Optional: set Logger to VERBOSE for debug information.
// Remove before app release.
gai.logger.logLevel = kGAILogLevelVerbose;
添加屏幕跟踪

此时，每当用户打开或切换您应用上的屏幕时，您都会向 Google Analytics（分析）发送一次已命名的屏幕浏览。打开您要跟踪的“数据视图控制器”，或者如果该应用是一个新应用，请打开默认的数据视图控制器。您的代码应能够：

添加必要的标头：
#import <GoogleAnalytics/GAI.h>
#import <GoogleAnalytics/GAIDictionaryBuilder.h>
#import <GoogleAnalytics/GAIFields.h>
ViewController.m
使用 viewWillAppear 方法或函数替换值来插入屏幕跟踪。
为屏幕提供名称并执行跟踪。
id<GAITracker> tracker = [GAI sharedInstance].defaultTracker;
[tracker set:kGAIScreenName value:name];
[tracker send:[[GAIDictionaryBuilder createScreenView] build]];
ViewController.m
注意：无论是强制性地（通过代码）向您的用户展示还是通过分镜脚本向您的用户展示，您都可以向代表屏幕的每个 UIViewController 添加跟踪代码。如果您想在 Google Analytics（分析）中区分您应用的不同屏幕浏览数据，请在每个 UIViewController 内设置一个名称。记录在共享跟踪器上的所有活动会发送最新的屏幕名称，直到这些名称被替换或清除（设置为 null）。


广告系列衡量 - iOS SDK

本文档将大略介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量广告系列和流量来源。

概览

在 Google Analytics（分析）中衡量广告系列可让您将应用中的用户活动归因于特定的广告系列和流量来源。iOS 版 Google Analytics（分析）SDK 提供了以下这些用于广告系列和流量来源归因的选项：

iOS 应用安装广告系列归因 - 了解是其他应用的哪些广告系列将用户引荐到 iTunes 下载您的 iOS 应用。
常规广告系列和流量来源归因 - 了解是哪些广告系列或引荐来源网址启动了您的应用（在应用安装之后）。
下文介绍了何时及如何在您的应用中实现每种广告系列衡量功能。

广告系列参数

广告系列参数用于传递将用户带到您的应用中的流量来源和广告系列的相关信息。

下表列出了可用于常规广告系列衡量的广告系列参数：

utm_campaign    广告系列名称，用于关键字分析，以标识具体的产品推广活动或战略广告系列。    utm_campaign=spring_sale
utm_source    广告系列来源，用于确定具体的搜索引擎、简报或其他来源。    utm_source=google
utm_medium    广告系列媒介，用于确定电子邮件或采用每次点击费用 (CPC) 的广告等媒介。    utm_medium=cpc
utm_term    广告系列字词，用于付费搜索，为广告提供关键字    utm_term=running+shoes
utm_content    广告系列内容，用于 A/B 测试和内容定位广告，以区分指向相同网址的不同广告或链接。    utm_content=logolink 
utm_content=textlink
gclid    AdWords 自动标记参数，用于衡量 Google AdWords 广告。此值会动态生成，请勿修改。    
常规广告系列和流量来源归因

应用在安装之后，即可被来自广告系列、网站或其他应用的引荐所启动。在这种情况下，您可以直接在跟踪器中设置广告系列字段，以便将一系列会话中的用户活动归因到特定的引荐流量来源或营销广告系列。

发送广告系列数据的最简单方式是使用 [GAIDictionaryBuilder setCampaignParametersFromUrl:urlString]，其中 urlString 是一个字符串，代表一个可能包含 Google Analytics（分析）广告系列参数的网址。请注意在下例中，广告系列数据未直接在跟踪器中设置，因为该数据只需要发送一次：

/*
* MyAppDelegate.m
*
* An example of how to implement campaign and referral attribution.
* If no Google Analytics campaign parameters are set in the referring URL,
* use the hostname as a referrer instead.
*/

// For iOS 9.0 and later
- (BOOL)application:(UIApplication *)app openURL:(nonnull NSURL *)url
options:(nonnull NSDictionary<NSString *,id> *)options {

// For iOS versions prior to 9.0
//- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
//  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {

NSString *urlString = [url absoluteString];

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithName:@"tracker"
trackingId:@"UA-XXXX-Y"];

// setCampaignParametersFromUrl: parses Google Analytics campaign ("UTM")
// parameters from a string url into a Map that can be set on a Tracker.
GAIDictionaryBuilder *hitParams = [[GAIDictionaryBuilder alloc] init];

// Set campaign data on the map, not the tracker directly because it only
// needs to be sent once.
[hitParams setCampaignParametersFromUrl:urlString];

// Campaign source is the only required campaign field. If previous call
// did not set a campaign source, use the hostname as a referrer instead.
if(![hitParams get:kGAICampaignSource] && [url host].length !=0) {
// Set campaign data on the map, not the tracker.
[hitParams set:@"referrer" forKey:kGAICampaignMedium];
[hitParams set:[url host] forKey:kGAICampaignSource];
}

NSDictionary *hitParamsDict = [hitParams build];

// A screen name is required for a screen view.
[tracker set:kGAIScreenName value:@"screen name"];

// Previous V3 SDK versions.
// [tracker send:[[[GAIDictionaryBuilder createAppView] setAll:hitParamsDict] build]];

// SDK Version 3.08 and up.
[tracker send:[[[GAIDictionaryBuilder createScreenView] setAll:hitParamsDict] build]];
或者，如果您掌握的广告系列信息并不是 Google Analytics（分析）广告系列参数的形式，则可以在 NSDictionary 中设置并手动发送：

// Assumes at least one tracker has already been initialized.
id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];

// Note that it's not necessary to set kGAICampaignKeyword for this email campaign.
NSDictionary *campaignData = [NSDictionary dictionaryWithObjectsAndKeys:
@"email", kGAICampaignSource,
@"email_marketing", kGAICampaignMedium,
@"summer_campaign", kGAICampaignName,
@"email_variation1", kGAICampaignContent, nil];

// A screen name is required for a screen view.
[tracker set:kGAIScreenName value:@"screen name"];

// Note that the campaign data is set on the Dictionary, not the tracker.
// Previous V3 SDK versions.
// [tracker send:[[[GAIDictionaryBuilder createAppView] setAll:campaignData] build]];

// SDK Version 3.08 and up.
[tracker send:[[[GAIDictionaryBuilder createScreenView] setAll:campaignData] build]];
iOS 应用安装广告系列衡量

Google Analytics（分析）默认情况下即支持 iOS 应用安装广告系列衡量功能，它既支持常见网络，也支持为其他网络生成自定义网址。

要启用 iOS 应用安装广告系列衡量功能，请使用下面的 iOS 广告系列跟踪网址构建工具来为您将用户发送到 App Store 的广告生成目标网址。为了让 iOS 广告系列跟踪功能正常发挥作用，您必须已在您的 iOS 应用中实现了 Google Analytics（分析）、启用了 IDFA 收集，并且已在跟踪您应用中的一个或多个屏幕浏览或事件。如果您想使用自动的 iAd 应用安装广告系列衡量功能，还需要向您的应用添加额外的框架。

注意： iOS 安装跟踪功能仅适用于通过移动广告网络（例如投放应用内广告的 AdMob）投放的广告。
对 iOS 转化跟踪问题进行自助排查

如果看不到您的 iOS 广告系列的转化，请按以下步骤进行问题排查：

确认您已启用 iOS 广告系列跟踪
确认应用 ID 正确无误
确认 GA SDK 发送了 IDFA
查看受众特征报告以确认发送了 IDFA
确保 iOS 广告系列跟踪网址正确无误

第 1 步：确认您已启用 iOS 广告系列跟踪
要确认针对目标媒体资源的 iOS 广告系列跟踪功能已启用，请执行以下操作：
点击管理标签。
“管理”标签
选择媒体资源，然后点击媒体资源设置。
媒体资源设置
确保 iOS 广告系列跟踪处于开启状态。
iOS 广告系列跟踪
注意：如果未开启“iOS 广告系列跟踪”，Google Analytics（分析）就不会将 iOS 广告系列数据归因到应用数据。

第 2 步：确认应用 ID 正确无误
要使用应用数据正确归因广告系列，iOS 广告系列跟踪网址和您的应用跟踪实现需要使用相同的应用 ID。要确定您当前跟踪的应用的应用 ID，请在 Google Analytics（分析）网页界面中以应用 ID 和会话分别作为维度和指标创建一个自定义报告。
在创建 iOS 广告系列点击跟踪网址时，请使用该自定义报告中显示的应用 ID。

第 3 步：确认 Google Analytics（分析）SDK 发送了 IDFA
Google Analytics（分析）使用广告客户标识符 (IDFA) 作为一种键值 将移动点击加入 Google Analytics（分析）匹配。您需要确保：
您的应用使用的是 iOS 版 Google Analytics（分析）SDK 版本 3.10 或更高版本。
如果您使用独立版 SDK 下载：
作为 iOS 版 Google Analytics（分析）的一部分，您的应用与 libAdIdAccess.a 相关联。
您的应用与 AdSupport.framework 相关联。
如果您使用 CocoaPods 来安装和管理依赖关系，请将 GoogleIDFASupport Cocoapod 添加到 Podfile：
pod 'GoogleIDFASupport'
您已在每个跟踪器上启用 IDFA 收集：
tracker.allowIDFACollection = YES;
可能的话，使用调试代理应用来查看 HTTP 请求并确认其中包含 IDFA。

第 4 步：查看受众特征报告以确认发送了 IDFA
Google Analytics（分析）使用 IDFA 来生成受众特征报告。在 Google Analytics（分析）中，点击报告标签，然后依次点击受众群体 > 受众特征 > 概览，检查是否存在受众特征数据。如果存在，说明 IDFA 发送正确。

第 5 步：确保 iOS 广告系列跟踪网址正确无误
使用 iOS 广告系列跟踪网址构建工具来验证 iOS 广告系列跟踪网址是否正确。
在选择广告网络的自定义选项时，还请务必咨询该广告网络以确认该网络支持使用重定向网址来跟踪单个设备 ID。如果不能得到支持，则 Google Analytics（分析）无法向您报告任何数据。


崩溃和异常 - iOS SDK

本文档将大略介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 对崩溃和异常进行衡量。

概览

崩溃和异常衡量功能可帮助您衡量您的应用中发生的崩溃和异常的次数及类型。异常数据包含以下字段：
Description    kGAIExDescription    NSString    否    异常的相关说明（最多 100 个字符）。nil 是可接受的值。
isFatal    kGAIExFatal    BOOL    是    表示异常是否严重。 YES 表示严重。
崩溃和异常数据主要在“崩溃和异常”报告中提供。

捕获的异常

捕获的异常是指您的应用中您为其定义了异常处理代码的那些错误，例如请求数据时偶尔发生的网络连接超时情况。

为了对捕获的异常进行衡量，您应当在跟踪器上设置异常字段值并发送匹配，如下例所示：

/*
* An app tries to load a list of high scores from the cloud. If the request
* times out, an exception is sent to Google Analytics
*/
@try {

// Request some scores from the network.
NSArray *highScores = [self getHighScoresFromCloud];

}
@catch (NSException *exception) {

// May return nil if a tracker has not already been initialized with a
// property ID.
id tracker = [[GAI sharedInstance] defaultTracker];

[tracker send:[[GAIDictionaryBuilder
createExceptionWithDescription:@"Connection timeout"  // Exception description. May be truncated to 100 chars.
withFatal:@NO] build]];  // isFatal (required). NO indicates non-fatal exception.
}
衡量未捕获的异常

未捕获的异常表示您的应用在运行时遇到的意外错误，此类异常通常是严重的，会导致应用崩溃。通过将 trackUncaughtExceptions 的属性设置为 YES，可以自动向 Google Analytics（分析）发送未捕获的异常。例如：

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
[[GAI sharedInstance] setTrackUncaughtExceptions:YES];
return YES;
}
在使用自动异常衡量功能时，请注意以下事项：
所有使用自动异常衡量功能发送的异常都会在 Google Analytics（分析）中显示为严重异常。
默认情况下，说明字段会自动填充以下信息：异常类型、类名称、方法名称和线程名称。


自定义维度和指标 - iOS SDK

本开发者指南将介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来实现自定义维度和指标。

概览

自定义维度可让您在 Google Analytics（分析）中将元数据关联到匹配、用户及会话，而自定义指标可以让您在 Google Analytics（分析）中创建和增加您自己的指标。

使用 Google Analytics（分析）网络界面来配置自定义维度或指标。 了解如何配置自定义维度或指标（帮助中心）。
在应用中设置并发送自定义维度和指标值。
自定义维度和指标包含两个字段：

NSNumber Index – 自定义维度或指标的索引。此索引值以 1 起始。
NSString Value – 自定义维度或指标的值。对于指标而言，此值会被解析为整数（如果指标被配置为货币类型，则值会解析为固定位数的小数值）。
设置和发送值

要设置和发送自定义维度值，请使用以下代码：

// May return nil if a tracker has not yet been initialized with a property ID.
id tracker = [[GAI sharedInstance] defaultTracker];

// Set the custom dimension value on the tracker using its index.
[tracker set:[GAIFields customDimensionForIndex:1]
value:@"Premium user"];

[tracker set:kGAIScreenName
value:@"Home screen"];

// Send the custom dimension value with a screen view.
// Note that the value only needs to be sent once, so it is set on the Map,
// not the tracker.

// Previous V3 SDK versions.
// [tracker send:[[[GAIDictionaryBuilder createAppView] set:@"premium"
//                                                   forKey:[GAIFields customDimensionForIndex:1]] build]];

// // SDK Version 3.08 and up.
[tracker send:[[[GAIDictionaryBuilder createScreenView] set:@"premium"
forKey:[GAIFields customDimensionForIndex:1]] build]];
自定义维度值可以随任何 Google Analytics（分析）匹配类型发送，包括屏幕浏览、事件、电子商务交易、用户计时和社交互动。在数据处理过程中，自定义维度的范围定义将会决定哪些匹配会与该维度值相关联。

要设置和发送自定义指标值，请使用以下代码：

// May return nil if a tracker has not yet been initialized with a property ID.
id tracker = [[GAI sharedInstance] defaultTracker];

// Set the custom metric to be incremented by 5 using its index.
[tracker set:[GAIFields customMetricForIndex:1]
value:[[NSNumber numberWithInt:5] stringValue]];

[tracker set:kGAIScreenName
value:@"Home screen"];

// Custom metric value is sent with this screen view.
// [tracker send:[[GAIDictionaryBuilder createAppView] build]];     // Previous V3 SDK versions.
[tracker send:[[GAIDictionaryBuilder createScreenView] build]];     // SDK Version 3.08 and up.
实现方面的注意事项

本节将简略介绍在实现自定义维度或指标的过程中需要注意的一些额外事项。

自定义维度的注意事项

范围为用户级或会话级的值将会应用于过去的匹配

范围为用户级或会话级的自定义维度值将会应用于当前会话的所有匹配，包括之前的匹配。如果您不希望范围为用户级或会话级的自定义维度值在当前会话中应用到过去的匹配，请在向匹配应用该值之前开始一个新会话。
例如，如果您使用“会员类型”作为用户级自定义维度，而某位用户在会话过程中升级了其会员类型，您就可能想要在设置新的自定义维度值之前开始一个新会话。这可确保升级之前的匹配与旧的会员类型值相关联，而新的匹配则与新的值相关联。
自定义维度和数据视图（配置文件）过滤器

即使一起发送的匹配从数据视图（配置文件）中被滤除，用户或会话级自定义维度值仍然将与当前和/或将来会话中的所有匹配相关联。
如果根据自定义维度值来进行过滤，则对匹配的过滤将会以相应自定义维度值的范围为依据。 详细了解过滤器和自定义维度值如何在处理您数据的过程中相互影响。
自定义指标的注意事项

自定义指标值会在报告中汇总

与 Google Analytics（分析）中其他预定义的指标一样，自定义指标值会在报告中汇总。因此，如果要将您报告中的指标汇总值增加 1，您就要设置一个自定义指标值 1。
自定义指标和数据视图（配置文件）过滤器

虽然自定义指标值一般可以视情况随时设置，但还是请避免对可能从数据视图（配置文件）中被滤除的匹配设置自定义指标值。如果某次匹配被数据视图（配置文件）过滤器所滤除，则相关联的自定义指标值也将被滤除。 详细了解自定义维度和指标以及数据视图（配置文件）过滤器。
在使用自动屏幕衡量的情况下设置值

要在使用自动屏幕衡量的情况下向发送的屏幕浏览匹配应用自定义维度值，请在视图控制器的 viewDidAppear: 方法执行过程中设置该值。例如，您的视图控制器的 .m 文件可能如下所示：
#import "myViewController.h"
#import "GAI.h"

@implementation myViewController

-(void)viewDidAppear
{
id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];  // Get the tracker object.
[tracker set:[GAIFields customDimensionForIndex:1]
value:@"premium"];
[super viewDidAppear:animated];   // Custom dimension value will be sent with the screen view.

}

// The remainder of the implementation is omitted.


调度 - iOS SDK

本文档将介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 管理将数据调度到 Google Analytics（分析）的工作。

概览

使用 iOS 版 Google Analytics（分析）SDK 收集的数据会先存储在本地，然后才会在单独的线程中调度到 Google Analytics（分析）。

数据必须在各数据视图本地时区的次日凌晨 4 点之前进行调度和接收。送达时间晚于此时限的数据将不会出现在报告中。例如，如果某次匹配在本地时间晚上 11:59 加入队列，则必须在 4 个小时内（凌晨 3:59 之前）调度到 Google Analytics（分析），否则该匹配无法出现在报告中。但是，如果某次匹配在凌晨 00:00 加入队列，则需要在 28 小时内（即次日凌晨 3:59 之前）调度到 Google Analytics（分析）才能出现在报告中。

定期调度

默认情况下，iOS 版 Google Analytics（分析）SDK 会每隔 2 分钟调度一次数据。

// Set the dispatch interval in seconds.
// 2 minutes (120 seconds) is the default value.
[GAI sharedInstance].dispatchInterval = 120;
将此值设为负数会停用定期调度，在这种情况下，如果您希望向 Google Analytics（分析）调度数据，则需要使用手动调度。

// Disable periodic dispatch by setting dispatch interval to a value less than 1.
[GAI sharedInstance].dispatchInterval = 0;
如果用户的网络连接断开，或是用户在匹配数据尚待调度时就退出了您的应用，那么您的匹配数据会继续存储在本地。这些数据会在您的应用下次运行并发出调度请求时进行调度。

手动调度

要手动调度匹配数据（比如当您知道应用正在使用设备的网络连接调度其他数据时），请使用：

[[GAI sharedInstance] dispatch];
后台调度

要在 iOS 应用上启用后台调度，请执行以下操作：

为 dispatchHandler 添加属性。
实现 sendHitsInBackground 方法。
替换 applicationDidEnterBackground 方法。
替换 applicationWillEnterForeground 方法。
为 dispatchHandler 添加属性

在 AppDelegate 类的实现文件 (AppDelegate.m) 中，请将以下属性添加到 @implementation AppDelegate 之前：

@interface AppDelegate ()
@property(nonatomic, copy) void (^dispatchHandler)(GAIDispatchResult result);
@end

@implementation AppDelegate
// ...
实现 sendHitsInBackground 方法

在 AppDelegate 类中，实现 sendHitsInBackground 方法，以便在应用转入后台时发送匹配：

// This method sends any queued hits when the app enters the background.
- (void)sendHitsInBackground {
__block BOOL taskExpired = NO;

__block UIBackgroundTaskIdentifier taskId =
[[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
taskExpired = YES;
}];

if (taskId == UIBackgroundTaskInvalid) {
return;
}

__weak AppDelegate *weakSelf = self;
self.dispatchHandler = ^(GAIDispatchResult result) {
// Send hits until no hits are left, a dispatch error occurs, or
// the background task expires.
if (result == kGAIDispatchGood && !taskExpired) {
[[GAI sharedInstance] dispatchWithCompletionHandler:weakSelf.dispatchHandler];
} else {
[[UIApplication sharedApplication] endBackgroundTask:taskId];
}
};

[[GAI sharedInstance] dispatchWithCompletionHandler:self.dispatchHandler];
}
dispatchWithCompletionHandler 方法以一个完成代码块为参数，每当包含一个或多个 Google Analytics（分析）信标的请求完成时就调用该代码块。 其返回结果指示是否还有更多数据需要发送以及/或者上次调用是否成功。使用此方法，您可以在应用转入后台时有效管理调度信标。

覆盖 applicationDidEnterBackground 方法

覆盖 AppDelegate 类的 applicationDidEnterBackground 方法以调用 sendHitsInBackground 方法，此方法在应用转入后台时发送匹配：

- (void)applicationDidEnterBackground:(UIApplication *)application {
[self sendHitsInBackground];
}
覆盖 applicationWillEnterForeground 方法

覆盖 AppDelegate 类的 applicationWillEnterForeground 方法以恢复调度间隔。例如：

- (void)applicationWillEnterForeground:(UIApplication *)application {
// Restores the dispatch interval because dispatchWithCompletionHandler
// has disabled automatic dispatching.
[GAI sharedInstance].dispatchInterval = 120;
}



展示广告功能 - iOS SDK

本指南将介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 启用广告功能。

概览

启用 Google Analytics（分析）广告功能后，即可利用再营销、“受众特征和兴趣”报告等诸多服务。

详细了解 Google Analytics（分析）广告功能。

实现

要使用 Google Analytics（分析）广告功能，您需要收集 IDFA（广告客户标识符）。要启用 IDFA 收集功能，请将 libAdIdAccess.a 和 AdSupport.framework 库与您的应用相关联，然后将每个要为其启用广告功能的跟踪器上的 allowIDFACollection 属性设置为 YES。例如：

// Assumes a tracker has already been initialized with a property ID, otherwise
// getDefaultTracker returns nil.
id tracker = [[GAI sharedInstance] defaultTracker];

// Enable Advertising Features.
tracker.allowIDFACollection = YES;
请注意：根据您的构建设置，可能需要使用链接器标记 -force_load /path/to/libAdIdAccess.a。
该功能会收集广告标识符。在使用该功能时，请务必仔细阅读并严格遵守所有适用的 SDK 政策。


电子商务跟踪 - iOS SDK

本文档将大略介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量应用内付款和收入。

请注意：电子商务跟踪功能需要您发送 transaction 和 item 匹配。您不应将增强型电子商务相关方法用于标准电子商务跟踪。
概览

借助电子商务衡量功能，您可以向 Google Analytics（分析）发送应用内购买和销售数据。Google Analytics（分析）中的电子商务数据由交易匹配和商品匹配组成，两者又由共同的交易 ID 相关联。

交易数据包含以下字段：

字段名称    跟踪器字段    类型    是否必须提供    说明
交易 ID    kGAITransactionId    NSString    是    代表某次交易的唯一 ID。此 ID 不应与其他交易 ID 重复。
关联公司    kGAITransactionAffiliation    NSString    是    此次交易关联的实体（例如某家商店）。
收入    kGAITransactionRevenue    NSNumber    是    交易的总收入，含税费和运费。
税款    kGAITransactionTax    NSNumber    是    交易的总税费。
运费    kGAITransactionShipping    NSNumber    是    交易的总运费。
货币代码    kGAICurrencyCode    NSString    否    交易的本地货币。默认值为此次交易对应的数据视图（配置文件）的货币。
商品数据包含以下字段：

字段名称    跟踪器字段    类型    是否必须提供    说明
交易 ID    kGAITransactionId    NSString    是    该商品关联的交易 ID。
名称    kGAIItemName    NSString    是    产品名称。
库存单位 (SKU)    kGAIItemSku    NSString    是    产品 SKU。
类别    kGAIItemCategory    NSString    否    产品所属的类别。
价格    kGAIItemPrice    NSNumber    是    产品的价格。
数量    kGAIItemQuantity    NSNumber    是    产品的数量。
货币代码    kGAICurrencyCode    NSString    否    交易的本地货币。默认值为此次交易对应的数据视图（配置文件）的货币。
电子商务数据主要用于以下标准报告中：

电子商务概览
产品业绩
销售业绩
交易次数
购买前所耗时间
实现

为了向 Google Analytics（分析）发送交易和商品数据，您需要在跟踪器中逐一设置交易和商品字段值并进行发送。例如：

/*
* Called when a purchase is processed and verified.
*/
- (void)onPurchaseCompleted {

// Assumes a tracker has already been initialized with a property ID, otherwise
// this call returns null.
id tracker = [[GAI sharedInstance] defaultTracker];

[tracker send:[[GAIDictionaryBuilder createTransactionWithId:@"0_123456"             // (NSString) Transaction ID
affiliation:@"In-app Store"         // (NSString) Affiliation
revenue:@2.16F                  // (NSNumber) Order revenue (including tax and shipping)
tax:@0.17F                  // (NSNumber) Tax
shipping:@0                      // (NSNumber) Shipping
currencyCode:@"USD"] build]];        // (NSString) Currency code

[tracker send:[[GAIDictionaryBuilder createItemWithTransactionId:@"0_123456"         // (NSString) Transaction ID
name:@"Space Expansion"  // (NSString) Product Name
sku:@"L_789"            // (NSString) Product SKU
category:@"Game expansions"  // (NSString) Product category
price:@1.9F               // (NSNumber) Product price
quantity:@1                  // (NSInteger) Product quantity
currencyCode:@"USD"] build]];    // (NSString) Currency code

}
电子商务货币字段支持负数货币值，以用于退款或退货的情况。

注意：请勿在货币字段中加入货币符号或逗号。
指定货币

默认情况下，交易值会被视为采用相应数据视图（配置文件）的货币。

要覆盖某次交易或相关产品的局部货币值，请将交易和商品匹配的货币代码字段设置为新的货币代码。如需所支持货币和货币代码的完整列表，请参阅“支持的货币”参考。

/*
In this example, the currency of the transaction is set to Euros. The
currency values will appear in reports using the global currency
type of the view (profile).
*/
- (void)onPurchaseCompleted {

// Assumes a tracker has already been initialized with a property ID, otherwise
// this call returns null.
id tracker = [[GAI sharedInstance] defaultTracker];

[tracker send:[[GAIDictionaryBuilder createTransactionWithId:@"0_123456",         // (NSString) Transaction ID, should be unique among transactions.
affiliation:@"In-app Store",     // (NSString) Affiliation
revenue:(int64_t) 2.16,      // (int64_t) Order revenue (including tax and shipping)
tax:(int64_t) 0.17,      // (int64_t) Tax
shipping:(int64_t) 0,         // (int64_t) Shipping
currencyCode:@"EUR"] build]];     // (NSString) Currency code
}


增强型电子商务跟踪 - iOS SDK

本文档将大略介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量应用内电子商务相关操作和展示。

概览

增强型电子商务功能可让您衡量用户在其购物历程中与产品的互动，包括产品展示、产品点击、查看产品详情、将产品添加到购物车、开始结帐流程、交易以及退款。

重要提示：增强型电子商务功能要求您使用 iOS SDK 中包含的与增强型电子商务相关的库和字段。增强型电子商务值可随现有匹配（例如 screenviews 和 events）一起发送。您不应发送 transaction 和 item 匹配，这两者适用于标准电子商务跟踪。
实现

在您的应用中实现增强型电子商务跟踪功能之前，您必须将增强型电子商务库添加到您的应用。

在将您的应用配置为使用增强型电子商务后，您就可以：

衡量电子商务活动
衡量交易
衡量退款
衡量结帐流程
衡量内部促销
衡量电子商务活动

典型的增强型电子商务实现将会衡量产品展示次数以及以下任一操作：

选择产品。
查看产品详情。
内部促销信息的展示和选择。
向购物车中添加产品或从中移除产品。
开始产品结帐流程。
购买和退款。
衡量展示

要衡量产品展示，请设置 product 值和 impression 值，并将相应信息随匹配一起发送：

id tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"Black"];
[product setCustomDimension:1 value:@"Member"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createScreenView];

// Sets the product impression for the next available slot, starting with 1.
[builder addProductImpression:product
impressionList:@"Search Results"
impressionSource:@"From Search"];
[tracker set:kGAIScreenName value:@"My Impression Screen"];
[tracker send:[builder build]];
Product 必须有 name 或 id 值。其他所有值都非必需，可以不用设置。

衡量操作

操作的衡量方法如下：先设置 product 值，然后设置 product action 值来指定对产品执行的操作。

例如，以下代码衡量对搜索结果列表中展示的某个产品的选择：

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"Black"];
[product setCustomDimension:1 value:@"Member"];

GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPAClick];
[action setProductActionList:@"Search Results"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createScreenView];
[builder setProductAction:action];

// Sets the product for the next available slot, starting with 1
[builder addProduct:product];
[tracker set:kGAIScreenName value:@"My Impression Screen"];
[tracker send:[builder build]];
Product 必须有 name 或 id 值。其他所有值都非必需，可以不用设置。

合并展示和操作数据

既有产品展示又有操作时，可以将两者合并到同一次匹配中进行衡量。

下例显示了如何衡量一次在相关产品部分中的展示以及一次产品详情查看：

// The product from the related products section.
id tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12346"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"White"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createScreenView];

// Sets the product impression for the next available slot, starting with 1.
[builder addProductImpression:product
impressionList:@"Related Products"
impressionSource:@"From Related"];

// The product being viewed.
product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"Black"];

GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPADetail];
[builder setProductAction:action];
// Sets the product for the next available slot, starting with 1.
[builder addProduct:product];
[tracker set:kGAIScreenName value:@"Related Products Screen"];
[tracker send:[builder build]];
衡量交易

交易的衡量方法如下：先设置 product 值，然后设置 product action 值来指定购买操作。总收入、税费、运费等交易级详情在 product action 值中设置。

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"Black"];
[product setPrice:@29.20];
[product setCouponCode:@"APPARELSALE"];
[product setQuantity:@1];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Purchase"
label:nil
value:nil];
GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPAPurchase];
[action setTransactionId:@"T12345"];
[action setAffiliation:@"Google Store - Online"];
[action setRevenue:@37.39];
[action setTax:@2.85];
[action setShipping:@5.34];
[action setCouponCode:@"SUMMER2013"];
[builder setProductAction:action];

// Sets the product for the next available slot, starting with 1
[builder addProduct:product];
[tracker send:[builder build]];
指定货币

默认情况下，您可以通过 Google Analytics（分析）的管理网页界面为所有交易和商品配置一种通用的全局货币。

局部货币必须按 ISO 4217 标准指定。如需支持的完整转换货币列表，请参阅货币代码参考文档。

局部货币可通过为跟踪器设置 currency code 值来指定。例如，此跟踪器将以欧元发送货币金额值：

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTracker"];
[tracker set:kGAIScreenName value:@"transaction"];
[tracker set:kGAICurrencyCode value:@"EUR"]; // Set tracker currency to Euros.
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createScreenView];
[tracker send:[builder build]];
要详细了解 Google Analytics（分析）中的货币转换机制，请参阅电子商务功能参考中的多种货币部分。
衡量退款

要为整个交易退款，请设置 product action 值来指定交易 ID 和退款操作类型：

// Refund an entire transaction.
id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Refund"
label:nil
value:nil];
GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPARefund];
[action setTransactionId:@"T12345"];
[builder setProductAction:action];
[tracker send:[builder build]];
如果未找到相符的交易，则退款将不会得到处理。

要衡量部分退款，请设置 product action 值来指定要退款的交易 ID、产品 ID 和产品数量：

// Refund a single product.
id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Refund"
label:nil
value:nil];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"]; // Product ID is required for partial refund.
[product setQuantity:@1]; // Quanity is required for partial refund.
[builder addProduct:product];

GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPARefund];
[action setTransactionId:@"T12345"]; // Transaction ID is required for partial refund.
[builder setProductAction:action];
[tracker send:[builder build]];
为退款使用非互动事件

如果您需要使用事件来发送退款数据，但该事件不属于通常衡量的行为（即并非由用户发起），则建议您发送非互动事件。这可让特定指标免受该事件的影响。例如：

// Refund an entire transaction.
id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Refund"
label:nil
value:nil];

[builder set:@"1" forKey:kGAINonInteraction];

GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPARefund];
[action setTransactionId:@"T12345"];

[builder setProductAction:action];
[tracker send:[builder build]];
衡量结帐流程

为衡量结帐流程中的每个步骤，您需要：

添加跟踪代码，以衡量结帐流程中的每一步。
如果适用，添加跟踪代码以衡量结帐选项。
（可选）设置直观易懂的步骤名称以用于结帐渠道报告，方法是在网页界面的“管理”部分中配置电子商务设置。
1. 衡量结帐步骤

对于结帐流程中的每一步，您都需要实现相应的跟踪代码，以便向 Google Analytics（分析）发送数据。

Step 字段

对于要衡量的每一个结帐步骤，您都应加入 step 值。此值用于将结帐操作映射到您在电子商务设置中为每个步骤配置的标签。

请注意：如果您的结帐流程只有一步，或是您没有在电子商务设置中配置结帐渠道，则可以不设置 step 字段。
Option 字段

在衡量某个结帐步骤时，如果您有关于此步骤的更多信息，则可以为 checkout 操作设置 option 字段来捕获此信息，例如用户的默认付款方式（如“Visa”）。

衡量某个结帐步骤

要衡量某个结帐步骤，请先设置 product 值，然后设置 product action 值来指示结帐操作。如果适用，还可以设置该结帐步骤的 step 和 option 值。

下例显示了如何衡量结帐流程的第一步（一个产品，拥有关于付款方式的额外信息）：

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommerceProduct *product = [[GAIEcommerceProduct alloc] init];
[product setId:@"P12345"];
[product setName:@"Android Warhol T-Shirt"];
[product setCategory:@"Apparel/T-Shirts"];
[product setBrand:@"Google"];
[product setVariant:@"Black"];
[product setPrice:@29.20];
[product setCouponCode:@"APPARELSALE"];
[product setQuantity:@1];

GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Checkout"
label:nil
value:nil];

// Add the step number and additional info about the checkout to the action.
GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPACheckout];
[action setCheckoutStep:@1];
[action setCheckoutOption:@"Visa"];

[builder addProduct:product];
[builder setProductAction:action];
[tracker send:[builder build]];
2. 衡量结帐选项

结帐选项可让您衡量关于结帐状态的额外信息。有时您已经衡量了某个结帐步骤，但在用户设置了选项之后，关于此步骤有了新的额外信息，在这种情况下，结帐选项就可以派上用场。例如，用户选择了送货方式。

要衡量结帐选项，请设置 product action 值来指示结帐选项，并加入步骤序号和选项说明信息。

注意：您不应设置任何 product 值或 impression 值
您很可能希望在用户执行特定操作进入结帐流程中的下一步时衡量此操作。例如：

// (On "Next" button click.)
id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"CheckoutOption"
label:nil
value:nil];

GAIEcommerceProductAction *action = [[GAIEcommerceProductAction alloc] init];
[action setAction:kGAIPACheckoutOption];
[action setCheckoutStep:@1];
[action setCheckoutOption:@"Fedex"];

[builder setProductAction:action];
[tracker send:[builder build]];
// Advance to next page.
3. 结帐渠道配置

您可以为结帐流程中的每一步指定一个描述性的名称，以在报告中使用。要配置此类名称，请转到 Google Analytics（分析）网络界面的管理部分，选择相应数据视图（配置文件），然后点击电子商务设置。请按照相应电子商务设置说明，为要跟踪的每个结帐步骤设置标签。

注意：如果您不配置结帐步骤名称，它们将会显示为“第 1 步”、“第 2 步”、“第 3 步”等等。
Google Analytics（分析）网络界面“管理”部分中的“电子商务设置”。开启了电子商务功能，为 4 个结帐渠道步骤指定了标签：1. 检查购物车，2. 填写付款信息，3. 确认订单详情，4. 收据

增强型电子商务功能支持对内部促销信息的展示次数和选择次数进行衡量，例如对促销活动进行宣传的横幅。

促销信息展示

内部促销信息的展示一般在初始屏幕浏览或事件发生时衡量，方法为设置 promotion 值。例如：

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommercePromotion *promotion = [[GAIEcommercePromotion alloc] init];
[promotion setId:@"PROMO_1234"];
[promotion setName:@"Summer Sale"];
[promotion setCreative:@"summer_banner2"];
[promotion setPosition:@"banner_slot1"];

GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Ecommerce"
action:@"Promotion"
label:nil
value:nil];

[builder addPromotion:promotion];
[tracker send:[builder build]];
重要提示：虽然可以为促销信息展示设置操作，但该操作不能是促销信息点击操作。如果您要衡量促销信息点击操作，应在促销信息展示之后，在单独的匹配中发送该操作。
促销信息点击

内部促销信息点击的衡量方法如下：先设置 promotion 值，然后设置 product action 值来指示促销信息点击操作。例如：

id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"MyTrackingId"];
GAIEcommercePromotion *promotion = [[GAIEcommercePromotion alloc] init];
[promotion setId:@"PROMO_1234"];
[promotion setName:@"Summer Sale"];
[promotion setCreative:@"summer_banner2"];
[promotion setPosition:@"banner_slot1"];

GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createEventWithCategory:@"Internal Promotions"
action:@"click"
label:@"Summer Sale"
value:nil];

[builder set:kGAIPromotionClick forKey:kGAIPromotionAction];
[builder addPromotion:promotion];
[tracker send:[builder build]];


事件跟踪 - iOS SDK

本开发者指南将介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量您的应用中的事件。

概览

事件是一种实用的工具，可帮助您衡量用户与您的应用中的互动式组件的互动情况，例如按钮点击或游戏中某个道具的使用情况。

每个事件由 4 个字段组成，您可以使用这些字段来描述用户与您的应用内容的互动：

字段名称    跟踪器字段    类型    是否必须提供    说明
Category    kGAIEventCategory    NSString    是    事件类别
Action    kGAIEventAction    NSString    是    事件操作
Label    kGAIEventLabel    NSString    否    事件标签
Value    kGAIEventValue    NSNumber    否    事件值
实现

要向 Google Analytics（分析）发送事件，请使用 GAIDictionaryBuilder.createEventWithCategory:action:label:value: 并发送匹配，如下例所示：

// May return nil if a tracker has not already been initialized with a property
// ID.
id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];

[tracker send:[[GAIDictionaryBuilder createEventWithCategory:@"ui_action"
action:@"button_press"
label:@"play"
value:nil] build]];



屏幕 - iOS SDK

本文档将简要介绍屏幕以及如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量屏幕浏览。

概览

在 Google Analytics（分析）中，屏幕表示用户在您的应用内查看的内容。在网站分析领域，与此对应的概念是“网页浏览”。通过衡量屏幕浏览，您可以了解用户浏览最多的是哪些内容，以及他们如何在不同的内容之间跳转。

一次屏幕浏览的数据由一个字符串字段构成，此字段在您的 Google Analytics（分析）报告中将会用作屏幕的名称。

字段名称    跟踪器字段    类型    是否必须提供    说明
Screen Name    kGAIScreenName    NSString    是    应用屏幕的名称。
屏幕浏览数据主要用于以下标准 Google Analytics（分析）报告中：

“屏幕”报告
互动流
手动屏幕衡量

要手动发送屏幕浏览数据，请在跟踪器上设置屏幕字段值，然后发送匹配：

// May return nil if a tracker has not already been initialized with a
// property ID.
id tracker = [[GAI sharedInstance] defaultTracker];

// This screen name value will remain set on the tracker and sent with
// hits until it is set to a new value or to nil.
[tracker set:kGAIScreenName
value:@"Home Screen"];

// Previous V3 SDK versions
// [tracker send:[[GAIDictionaryBuilder createAppView] build]];

// New SDK versions
[tracker send:[[GAIDictionaryBuilder createScreenView] build]];
自动屏幕衡量

使用 GAITrackedViewController 类自动衡量屏幕浏览。请使用您的每个视图控制器对 GAITrackedViewController 进行扩展，并添加名为 screenName 的属性。此属性将用于设置屏幕名称字段。

//
// MyViewController.h
// An example of using automatic screen tracking in a ViewController.
//
#import "GAITrackedViewController.h"

// Extend the provided GAITrackedViewController for automatic screen
// measurement.
@interface AboutViewController : GAITrackedViewController

@end

//
// MyViewController.m
//
#import "MyViewController.h"
#import "AppDelegate.h"

@implementation MyViewController

- (void)viewDidLoad {
[super viewDidLoad];

// Set screen name.
self.screenName = @"Home Screen";
}

// Rest of the ViewController implementation.
@end


会话 - iOS SDK

本文档将大略介绍与 iOS 版 Google Analytics（分析）SDK v3 有关的会话。

概览

会话表示用户与您的应用互动的单段过程。会话是一种很有用的容器，可帮助您衡量屏幕浏览、事件和电子商务交易等活动。

管理会话

默认情况下，Google Analytics（分析）会将相互间隔不到 30 分钟的匹配归到同一个会话中。此间隔可在媒体资源一级进行配置。了解如何配置此会话超时时长。

手动会话管理

要手动开始或结束一个会话，请在您传递到跟踪器 send: 方法的字典中设置会话控制参数。

// May return nil if a tracker has not yet been initialized.
id tracker = [[GAI sharedInstance] defaultTracker];

// Start a new session with a screenView hit.
GAIDictionaryBuilder *builder = [GAIDictionaryBuilder createScreenView];
[builder set:@"start" forKey:kGAISessionControl];
[tracker set:kGAIScreenName value:@"My Screen"];
[tracker send:[builder build]];
// There should be no need to end a session explicitly.  However, if you do
// need to indicate end of session with a hit, simply add the following line
// of code to add the parameter to the builder:
[builder set:@"end" forKey:kGAISessionControl];



社交互动 - iOS SDK

本开发者指南将介绍如何使用 iOS 版 Google Analytics（分析）SDK v3 来衡量您的应用中的社交互动。

概览

社交互动衡量功能可让您衡量用户与您的内容中嵌入的各种社交网络分享和推荐小部件的互动情况。

社交互动数据包含以下字段：

字段名称    跟踪器字段    类型    是否必须提供    说明
Social Network    kGAISocialNetwork    NSString    是    用户与之互动的社交网络（例如 Facebook、Google+、Twitter 等）。
Social Action    kGAISocialAction    NSString    是    进行的社交操作（例如赞、分享、+1 等）。
Social Target    kGAISocialTarget    NSString    否    发生社交操作的内容（例如某篇文章或某段视频）。
iOS 版 Google Analytics（分析）SDK v3.x 收集的社交互动数据可在自定义报告中查看或通过 Core Reporting API 提取。

实现

要向 Google Analytics（分析）发送社交互动数据，请使用 GAIDictionaryBuilder.createSocialWithNetwork:action:target，如下例所示：

// May return nil if a tracker has not already been initialized with a property
// ID.
id tracker = [[GAI sharedInstance] defaultTracker];

NSString *targetUrl = @"https://developers.google.com/analytics";

[tracker send:[[GAIDictionaryBuilder createSocialWithNetwork:@"Twitter"
action:@"Tweet"
target:targetUrl] build]];


User ID - iOS SDK

本开发者指南将演示如何使用 iOS 版 Google Analytics（分析）SDK v3.x 来实现 User ID。

概览

User ID 功能可让您在 Google Analytics（分析）中衡量跨多种设备的用户活动，例如将在某台移动设备上与某个营销广告系列的某次互动归因于在另一台移动设备上或浏览器中发生的某次转化。

如果您使用 userId 字段随 Google Analytics（分析）匹配一起发送 User ID，则不但您的报告中的唯一身份用户数将更加准确，而且您还可以获得新的跨设备报告选项。 详细了解使用 User ID 的好处。

本指南介绍了如何使用 userId 字段和 iOS 版 Google Analytics（分析）SDK 将 User ID 发送到 Google Analytics（分析）。

前提条件

在将 User ID 发送到 Google Analytics（分析）之前：

设置 User ID。
查看 User ID 政策。
查看 User ID 功能参考，了解 User ID 的工作原理。
重要提示：由于您要自行负责提供您自己的 ID 并将其发送到 Google Analytics（分析），一般情况下，这就要求您的应用中已经实现某种身份验证系统，并能为您的用户提供登录使用体验。
实现

当您的 iOS 应用能够识别某位用户的身份时，您应当使用 userId 字段，随您的所有 Google Analytics（分析）匹配（例如网页浏览、事件、电子商务交易等）一起，发送代表该用户的 ID。

要发送 User ID，请使用 Measurement Protocol“&”符号语法和 kGAIUserId 参数名称来设置 userId 字段，如下例所示：

/**
* An example method called when a user signs in to an authentication system.
*
* @param user represents a generic User object returned by an authentication system on sign in.
*/
- void signInWithUser:(User *)user {

id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];

// You only need to set User ID on a tracker once. By setting it on the tracker, the ID will be
// sent with all subsequent hits.
[tracker set:kGAIUserId
value:user.id];

// This hit will be sent with the User ID value and be visible in User-ID-enabled views (profiles).
[tracker send:[[GAIDictionaryBuilder createEventWithCategory:@"UX"            // Event category (required)
action:@"User Sign In"  // Event action (required)
label:nil              // Event label
value:nil] build]];    // Event value
}
本例说明了如何获取 User ID：

NSString *userId = [tracker get:kGAIUserId];


用户计时- iOS SDK

开发者指南说明如何使用 iOS 版 Google Analytics（分析）SDK v3 衡量用户计时。

概览

用户计时在 Google Analytics（分析）中提供了一种原生的时间衡量方式。比如，这可用于衡量资源加载时间。

用户计时数据包含以下字段：

字段名称    跟踪器字段    类型    是否必需    说明
Category    kGAITimingCategory    NSString    是    计时事件的类别。
Value    kGAITimingValue    NSNumber    是    以毫秒表示的计时值。
Name    kGAITimingVar    NSString    是    计时事件的名称。
Label    kGAITimingLabel    NSString    否    计时事件的标签。
用户计时数据一般可在“应用速度用户计时”报告中找到。

实现

要向 Google Analytics（分析）发送用户计时数据，请使用 GAIDictionaryBuilder.createTimingWithCategory:interval:name:label: 构建计时匹配，然后使用 send 进行发送：

/*
* Called after a list of high scores finishes loading.
*
* @param loadTime The time it takes to load a resource.
*/
- (void)onLoad:(NSTimeInterval)loadTime {

// May return nil if a tracker has not already been initialized with a
// property ID.
id tracker = [[GAI sharedInstance] defaultTracker];

[tracker send:[[GAIDictionaryBuilder createTimingWithCategory:@"resources"
interval:@((NSUInteger)(loadTime * 1000))
name:@"high scores"
label:nil] build]];
}




高级配置 - iOS SDK

本文档将大略介绍 iOS 版 Google Analytics（分析）SDK v3 的高级配置功能。

概览

iOS 版 Google Analytics（分析）SDK 提供了一个 GAITracker 类用于设置并向 Google Analytics（分析）发送数据，并提供了一个 GAI 单例来作为您的实现方案的全局配置值的接口。

初始化

您需要先通过 GoogleAnalytics 单例至少初始化一个跟踪器，然后才能开始跟踪数据。在初始化时，您需要提供跟踪器的名称和一个 Google Analytics（分析）媒体资源 ID：

// Initialize a tracker using a Google Analytics property ID.
[[GAI sharedInstance] trackerWithTrackingId:@"UA-XXXX-Y"];
现在此跟踪器就可用于设置并向 Google Analytics（分析）发送数据了。

设置和发送数据

为了向 Google Analytics（分析）发送数据，您需要在跟踪器中设置“参数-值”对的映射关系，并通过 set 和 send 方法来发送这些数据：

/*
* Send a screen view to Google Analytics by setting a map of parameter
* values on the tracker and calling send.
*/

// Retrieve tracker with placeholder property ID.
id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"UA-XXXX-Y"];

NSDictionary *params = [NSDictionary dictionaryWithObjectsAndKeys:
@"appview", kGAIHitType, @"Home Screen", kGAIScreenName, nil];
[tracker send:params];
GAIDictionaryBuilder 类可以简化构建匹配的过程，因此在大多数情况下建议您使用此类。这样，我们便可以使用更少的代码量来发送同样的屏幕浏览匹配：

// Previous V3 SDK versions.
// Sending the same screen view hit using [GAIDictionaryBuilder createAppView]
// [tracker send:[[[GAIDictionaryBuilder createAppView] set:@"Home Screen"
//                                                   forKey:kGAIScreenName] build]];

// SDK Version 3.08 and up.
// Sending the same screen view hit using [GAIDictionaryBuilder createScreenView]
[tracker send:[[[GAIDictionaryBuilder createScreenView] set:@"Home Screen"
forKey:kGAIScreenName] build]];
Measurement Protocol“&”符号语法

您可以选择在 Builder 中设置值，以便为一次匹配设置该值；也可以使用 Measurement Protocol“&”符号语法在跟踪器对象本身中设置值，以便为所有后续匹配设置该值。

// Sending the same screen view hit using ampersand syntax.
// Previous V3 SDK versions.
// [tracker send:[[[GAIDictionaryBuilder createAppView] set:@"Home Screen"
//                                                   forKey:kGAIScreenName] build]];

// SDK Version 3.08 and up.
[tracker send:[[[GAIDictionaryBuilder createScreenView] set:@"Home Screen"
forKey:kGAIScreenName] build]];
如要了解可用 Measurement Protocol 参数的完整列表，请参阅 Measurement Protocol 参数参考。

将值应用于多次匹配

直接在跟踪器中设置的值将是持久值，并会应用于多次匹配，如下例所示：

// May return nil if a tracker has not yet been initialized with
// a property ID.
id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];

// Set screen name on the tracker to be sent with all hits.
[tracker set:kGAIScreenName
value:@"Home Screen"];

// Send a screen view for "Home Screen".
// [tracker send:[[GAIDictionaryBuilder createAppView] build]];   // Previous V3 SDK versions.
[tracker send:[[GAIDictionaryBuilder createScreenView] build]];   // SDK Version 3.08 and up.

// This event will also be sent with &cd=Home%20Screen.
[tracker send:[[GAIDictionaryBuilder createEventWithCategory:@"UX"
action:@"touch"
label:@"menuButton"
value:nil] build]];

// Clear the screen name field when we're done.
[tracker set:kGAIScreenName
value:nil];
只有在想要使值成为持久值并应用于多次匹配时，您才应当直接在跟踪器中设置值。在跟踪器中设置屏幕名称是一种可取的做法，因为同样的值可以应用于一系列的屏幕浏览和事件匹配。不过，不建议在跟踪器中设置匹配类型等字段，因为此类字段很可能随每次匹配而改变。

使用多个跟踪器

您可以在一个实现实例中使用多个跟踪器，以便向多个媒体资源发送数据：

id<GAITracker> t1 = [[GAI sharedInstance] trackerWithTrackingId:@"UA-XXXX-1"];

// Trackers may be named. By default, name is set to the property ID.
id<GAITracker> t2 = [[GAI sharedInstance] trackerWithName:@"altTracker"
trackingId:@"UA-XXXX-2"];

[t1 set:kGAIScreenName
value:@"Home Screen"];

[t2 set:kGAIScreenName
value:NSStringFromClass([self class])];

// Send a screenview to UA-XXXX-1.
// [t1 send:[[GAIDictionaryBuilder createAppView] build]];   // Previous V3 SDK versions.
[t1 send:[[GAIDictionaryBuilder createScreenView] build]];   // SDK Version 3.08 and up.

// Send a screenview to UA-XXXX-2.
// [t2 send:[[GAIDictionaryBuilder createAppView] build]];   // Previous V3 SDK versions.
[t2 send:[[GAIDictionaryBuilder createScreenView] build]];   // SDK Version 3.08 and up.
自动屏幕衡量和未捕获异常衡量等自动衡量功能都只会使用一个跟踪器来向 Google Analytics（分析）发送数据。如果您在使用这些功能，但又想使用其他跟踪器来发送数据，则需要采取手动方式。

作为参考，自动屏幕衡量会使用 GAITrackedViewController 的 tracker 属性中指定的跟踪器。未捕获异常衡量会使用 GAI 实例中指定的默认跟踪器。

使用默认跟踪器

Google Analytics（分析）会维持一个默认跟踪器。第一个初始化的跟踪器将成为默认跟踪器，但您也可以覆盖此设置：

// t1 becomes the default tracker because it is the first tracker initialized.
id<GAITracker> t1 = [[GAI sharedInstance] trackerWithTrackingId:@"UA-XXXX-1"];

id<GAITracker> t2 = [[GAI sharedInstance] trackerWithTrackingId:@"UA-XXXX-2"];

// Returns t1.
id<GAITracker> defaultTracker = [[GAI sharedInstance] defaultTracker];

// Hit sent to UA-XXXX-1.
// Previous V3 SDK versions.
// [defaultTracker send:[[[GAIDictionaryBuilder createAppView]
//                 set:@"Home Screen" forKey:kGAIScreenName] build]];

// SDK Version 3.08 and up.
[defaultTracker send:[[[GAIDictionaryBuilder createScreenView]
set:@"Home Screen" forKey:kGAIScreenName] build]];

// Override the default tracker.
[[GAI sharedInstance] setDefaultTracker:t2];

// Returns t2.
defaultTracker = [[GAI sharedInstance] defaultTracker];

// Hit sent to UA-XXXX-2.
// Previous V3 SDK versions.
// [defaultTracker send:[[[GAIDictionaryBuilder createAppView]
//                        set:NSStringFromClass([self class])
//                     forKey:kGAIScreenName] build]];

// SDK Version 3.08 and up.
[defaultTracker send:[[[GAIDictionaryBuilder createScreenView]
set:NSStringFromClass([self class])
forKey:kGAIScreenName] build]];
抽样

您可以启用客户端抽样，以限制向 Google Analytics（分析）发送的匹配数量。如果您的应用有大量用户，或因其他原因而向 Google Analytics（分析）发送大量数据，则启用抽样有助于确保报告不发生中断。

例如，要将客户端抽样率设为 50%，请使用以下代码：

// Assumes a tracker has already been initialized with a property ID, otherwise
// getDefaultTracker returns nil.
id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];

// Set a sample rate of 50%.
[tracker set:kGAISampleRate value:@"50.0"];
为避免报告数据不一致，您的每个应用数据视图（配置文件）收集的数据所采用的抽样率都应当相同。如果应用的不同版本使用不同的抽样率，您将需要根据应用版本维度来配置数据视图（配置文件）过滤器，以便将来自应用不同版本的数据分隔开来。
应用级选择停用

您可以启用“应用级选择停用”标记，以便在整个应用内停用 Google Analytics（分析）。请注意，此标记必须在每次应用启动时设置（默认值为 NO）。

要获取应用级选择停用设置，请使用以下代码：

// Get the app-level opt out preference.
if ([GAI sharedInstance].optOut) {
... // Alert the user they have opted out.
}
要设置应用级选择停用，请使用：

// Set the app-level opt out preference.
[[GAI sharedInstance] setOptOut:YES];
删除客户端用户数据

如果您需要为最终用户重置或删除 Google Analytics（分析）客户端数据，则可以删除 Google Analytics（分析）文件或设置新的客户端 ID。

选项 1：删除 Google Analytics（分析）文件

如果您的应用没有为最终用户明确设置客户端 ID，您可以通过删除用于存储客户端 ID 的 Google Analytics（分析）文件，强制生成新的客户端 ID。

以下示例方法可用于删除 Google Analytics（分析）文件。该方法必须在初始化 Google Analytics（分析）之前执行：

/* Execute this before [GAI sharedInstance] is initialized. */
static void DeleteGAFiles(void) {
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES);
NSFileManager *fileManager = [NSFileManager defaultManager];
NSArray *libraryFiles = [fileManager contentsOfDirectoryAtPath:paths.firstObject error:nil];

NSPredicate *predicate =
[NSPredicate predicateWithFormat:@"self BEGINSWITH 'googleanalytics'"];
NSArray *matchingFileNames = [libraryFiles filteredArrayUsingPredicate:predicate];

for (NSString *fileName in matchingFileNames) {
NSError *error;
NSString *filePath = [paths.firstObject stringByAppendingPathComponent:fileName];
if (![fileManager removeItemAtPath:filePath error:&error]) {
// Log error.
}
}
}
必须在 [GAI sharedInstance] 初始化之前删除 Google Analytics（分析）文件。
选项 2：设置新的客户端 ID

您可以生成并设置一个新的唯一客户端 ID。所有后续的匹配将使用新的客户端 ID，且之前的数据不会与新的客户端 ID 相关联。

执行以下代码可生成并设置新的客户端 ID：

[GAI sharedInstance].optOut = YES;

// This Id should be a valid UUID (version 4) string as described in https://goo.gl/0dlrGx.
NSString *newClientID = [NSUUID UUID].UUIDString.lowercaseString;
id dispatcher = [[GAI sharedInstance] performSelector:@selector(dispatcher)];
id dataStore = [dispatcher performSelector:@selector(dataStore)];
if ([dataStore respondsToSelector:@selector(setClientId:)]) {
[dataStore performSelector:@selector(setClientId:) withObject:newClientID];
}

[GAI sharedInstance].optOut = NO;
如果您决定明确设置（而非随机生成）一个客户端 ID，请确保它是一个有效的 UUID（版本 4）字符串（如 https://goo.gl/0dlrGx 中所述）。
如果您启用了 IDFA（广告客户标识符），则设置新的客户端 ID 后，系统会立即用一个随机生成的新客户端 ID 覆盖它。如果您需要设置特定的客户端 ID 且已启用 IDFA（广告客户标识符），我们建议您不要使用此选项。您应该使用“选项 1：删除 Google Analytics（分析）文件”来重置客户端 ID。
对 IP 地址进行匿名处理

启用匿名处理 IP 功能可以告知 Google Analytics（分析）匿名化处理由 SDK 发送的 IP 信息，即在存储前删除 IP 地址的最后一个八位位组。

下例显示了如何启用跟踪器的匿名处理 IP 功能：

[tracker set:kGAIAnonymizeIp value:@"1"];
匿名处理 IP 功能可以随时设置。

测试和调试

iOS 版 Google Analytics（分析）SDK 提供了多种工具，可让您更轻松地进行测试和调试。

Dry Run

SDK 提供了 dryRun 标志，如果设置了此标志，则不会向 Google Analytics（分析）发送任何数据。在您对实现方案进行测试或调试时，如果不想测试数据出现在您的 Google Analytics（分析）报告中，就应当设置 dryRun 标志。

要设置 dryRun 标志，请使用以下代码：

[[GAI sharedInstance] setDryRun:YES];
Logger

GAILogger 协议用于处理来自 SDK 的实用讯息，有以下详细程度可选：error、warning、info 和 verbose。

SDK 会初始化一个标准 Logger 实现，默认情况下只会记录发送到控制台的警告或错误讯息。要设置 Logger 的详细程度，请使用以下代码：

// Set the log level to verbose.
[[GAI sharedInstance].logger setLogLevel:kGAILogLevelVerbose];
您也可以使用自定义的 Logger 实现：

// Provide a custom logger.
[[GAI sharedInstance].logger = [[CustomLogger alloc] init];
完整示例

下例显示了初始化 Google Analytics（分析）实现和发送一次屏幕浏览所需的代码。

接下来，在第一个屏幕向用户显示时，衡量一次屏幕浏览：

一般情况下，实现的初始化可以从应用代理中完成，如下例所示：

//
//  AppDelegate.h
//
#import <UIKit/UIKit.h>
#import "GAI.h"

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;
@property (strong, nonatomic) id<GAITracker> tracker;

@end

//
//  AppDelegate.m
//
#import "AppDelegate.h"

/** Google Analytics configuration constants **/
static NSString *const kGaPropertyId = @"UA-XXXX-Y"; // Placeholder property ID.
static NSString *const kTrackingPreferenceKey = @"allowTracking";
static BOOL const kGaDryRun = NO;
static int const kGaDispatchPeriod = 30;

@interface AppDelegate ()

- (void)initializeGoogleAnalytics;

@end

@implementation AppDelegate
- (void)applicationDidBecomeActive:(UIApplication *)application {
[GAI sharedInstance].optOut =
![[NSUserDefaults standardUserDefaults] boolForKey:kTrackingPreferenceKey];
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// Do other work for customization after application launch
// then initialize Google Analytics.
[self initializeGoogleAnalytics];

return YES;
}

- (void)initializeGoogleAnalytics {

[[GAI sharedInstance] setDispatchInterval:kGaDispatchPeriod];
[[GAI sharedInstance] setDryRun:kGaDryRun];
self.tracker = [[GAI sharedInstance] trackerWithTrackingId:kGaPropertyId];
}

// The rest of the app delegate code omitted.

@end
然后，在某个视图向用户显示时衡量一次屏幕浏览：

//
// MyViewController.m
//
#import "MyViewController.h"
#import "AppDelegate.h"
#import "GAI.h"
#import "GAIFields.h"
#import "GAITracker.h"
#import "GAIDictionaryBuilder.h"

@implementation MyViewController

- (void)viewDidAppear:(BOOL)animated {
[super viewDidAppear:animated];

// This screen name value will remain set on the tracker and sent with
// hits until it is set to a new value or to nil.
[[GAI sharedInstance].defaultTracker set:kGAIScreenName
value:@"Home Screen"];

// Send the screen view.
// Previous V3 SDK versions.
// [[GAI sharedInstance].defaultTracker
//     send:[[GAIDictionaryBuilder createAppView] build]];

// SDK Version 3.08 and up.
[[GAI sharedInstance].defaultTracker
send:[[GAIDictionaryBuilder createScreenView] build]];
}



可选功能

在使用 CocoaPods 配置您的 iOS 应用或使用 Google Analytics（分析）服务 SDK 手动配置 iOS 应用后，您可以选择添加以下文件以启用某些功能（如增强型电子商务、IDFA（广告客户标识符）和 iAd 应用安装广告系列衡量功能）。

增强型电子商务

要使用增强型电子商务功能，请将以下文件添加到您的应用：

GAIEcommerceFields.h
IDFA（广告客户标识符）

要使用 IDFA（广告客户标识符），请将下列文件与您的应用相关联，然后启用 IDFA 收集功能：

libAdIdAccess.a
AdSupport.framework
注意：根据您的构建设置，可能需要使用链接器标记 -force_load /path/to/libAdIdAccess.a。
如果您使用 CocoaPods 来安装和管理依赖关系并希望收集 IDFA，而不通过手动方式关联 libAdIdAccess 和 AdSupport.framework，请将 GoogleIDFASupport Cocoapod 添加到 Podfile：

pod 'GoogleIDFASupport'
这样将关联所需使用的 IDFA 库和系统框架。

启用 IDFA（广告客户标识符）收集功能

要启用 IDFA 收集功能，请将每个收集 IDFA 的跟踪器的 allowIDFACollection 属性设置为 YES。例如：

// Enable IDFA collection.
tracker.allowIDFACollection = YES;
iAd 应用安装广告系列衡量

要使用自动的 iAd 应用安装广告系列衡量功能，请将以下框架添加到您的应用的链接库中：

iAd.framework
请注意：iAd 应用安装广告系列衡量功能需要使用 IDFA 收集功能。



