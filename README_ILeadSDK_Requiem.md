# ILeadSDK客户端接入文档
## 1	简介
### 1.1	SDK包功能简介
SDK包封装了用户中心系统，该系统拥有以下功能接口

* 登录
* 注册
* 密保
* 交叉广告

### 1.2	软硬件支持
	
## 2	安装
### 2.1	下载SDK

到指定地址下载sdk

### 2.2	解压SDK

将下载的SDK解压，可以得到，一个工具包IleadSDK.framework和一个IleadSDK.bundle。
## 3	使用SDK中的常用功能
### 3.1	引用资源

在您的工程目录右键选择Add Files to "XXX"，在弹出的窗口中选择IleadSDK.bundle，建议勾选上Copy items into destination group's folder，选择Create groups for any added folder，然后设置好target即可。

### 3.2	引入framework

在您的工程目录右键选择Add Files to "XXX"，在弹出的窗口中选择`IleadSDK.framework`，建议勾选上Copy items into destination group's folder，选择Create groups for any added folder，然后设置好target。您还需要在Build Phases的Link Binary With Libraries添加这些库`libc++.dylib`。

**注意：** 

最后，您需要在您的工程target的Build Settings标签下找到Other Linker Flags这个选项，设置为-ObjC

此时准备工作完毕，可以开始初始化SDK并使用SDK的登录接口了

### 3.3	初始化SDK

在使用SDK之前需要先进行初始化，初始化SDK主要是为了设置基本的应用信息，比如APPID，APPKEY等信息，这些信息都是创建应用时填写或者返回的。接口信息如下Demo中写在AppDelegate.m，`不要调换SDK方法的顺序`，并且注意`[[IleadSDK sharedInstance] Login]`必须写在`[self.window makeKeyAndVisible]`之后。
如下图所示：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // Override point for customization after application launch.
    self.window.backgroundColor = [UIColor whiteColor];
    self.viewController = [[ViewController alloc] init];
    self.window.rootViewController = self.viewController ;
    [self.window makeKeyAndVisible];
    
    [IleadSDK sharedInstance].serviceArea = ChinaService;
    [IleadSDK sharedInstance].clientLanguage = zhCN_Language;
    [[IleadSDK sharedInstance] setIsShowLog:YES];
    [[IleadSDK sharedInstance] setAppID:71 appKey:@"ei2hfs9e83qmc92jdz01a"];
    [[IleadSDK sharedInstance] setIsPushLoginViewWhenLogout:YES];
    [[IleadSDK sharedInstance] Login];
    return YES;
}
```


 
其中，

1. `[IleadSDK sharedInstance].serviceArea = ChinaService;`;
设置SDK的服务区【注：默认是根据SDK语言来选择服务区】，如果要设置必须在SDK初始化最前面设置才能生效【即：调用setAppID之前进行设置】。
2. `[IleadSDK sharedInstance].clientLanguage = zhCN_Language;`;
设置SDK客户端语言【注：默认设备语言】，如果要设置必须在SDK初始化。
3. `[[IleadSDK sharedInstance] setIsShowLog:YES];`;
设置是否打印日志。
4. `[[IleadSDK sharedInstance] setAppID:71 appKey:@"ei2hfs9e83qmc92jdz01a"]`;
设置您的appID和appKey。
5. `[[IleadSDK sharedInstance] setIsPushLoginViewWhenLogout:YES]`;
设置注销后是否弹出登录界面。


### 3.4	使用SDK中的授权功能
#### 3.4.1	登录


点击登录按钮接口如下：

```
/**
 *  打开登录界面,若已经保存账号密码则自动登录
 */
- (void)Login;
```
 
只需要在初始化时候注册通知接受回调信息。
**注意：只有登陆成功才会回调此方法，回调函数`SwitchToGameMainView`函数名可自行定义）**

示例如下：

```
[[NSNotificationCenter defaultCenter] addObserver:self
									     selector:@selector(SwitchToGameMainView)
										     name:@"LoginSuccessToGame"
										   object:nil];
```
 
 
#### 3.4.2	注销

```
/**
*  注销用户或游客，相当于点击用户中心（游客中心中）的注销按钮
*/
- (void)Logout;
```

只需要在初始化时候注册通知接受回调信息。
**注意：只有注销成功才会回调此方法，回调函数`Logout`函数名可自行定义）**

示例如下：

```
[[NSNotificationCenter defaultCenter] addObserver:self
									     selector:@selector(Logout)
										     name:@"LogoutSuccess"
										   object:nil];
```
 

 
注：如果注销后不需要自动弹出登录界面请在AppDelegate中执行

`[[IleadSDK sharedInstance] setIsPushLoginViewWhenLogout:NO]`

 
#### 3.4.3	手动关闭登录窗口
手动关闭登录窗口时，将发送通知给第三方，只需要在初始化时候注册通知接受回调信息。
**注意：只有手动关闭成功后才会回调此方法，回调函数`ExitSDKView`函数名可自行定义）**

示例如下：

```
[[NSNotificationCenter defaultCenter] addObserver:self
									     selector:@selector(ExitSDKView)
										     name:@"CloseSDKView"
										   object:nil];
```

 
#### 3.4.4	修改用户名回调
在SDK初始化时候注册通知接受回调信息，用户修改用户名成功后，回调此方法
**（注：回调函数`ChangeUsernameSuccess`由第三方实现，函数名可自行定义）**。

```
[[NSNotificationCenter defaultCenter] addObserver:self
									     selector:@selector(ChangeUsernameSuccess)
										     name:@"ChangeUsernameSuccess"
										   object:nil];
```
 
#### 3.4.5	进入登录界面
调用`[[IleadSDK sharedInstance] Login]`方法进入登录页面（登录页面包含了注册页面的接口和忘记密码／帐号页面的接口）

示例如下：

```
-(void) UserLogin
{
	[[IleadSDK sharedInstance] Login];
}
```
 
#### 3.4.6	获取信息字段
SDK提供的信息字段有四个，分别是openID，proof，username，UUID，如下所示：

```
/**
 *  openID与proof配套使用
 */
@property (nonatomic, readonly) NSString *openID;
@property (nonatomic, readonly) NSString *proof;

/**
 *  uuid与uuidProof配套使用
 */
@property (nonatomic, readonly) NSString *UUID;
@property (nonatomic, readonly) NSString *UUIDProof;

@property (nonatomic, readonly) NSString *username;
```

 
openID字段存放的是用户登陆成功之后，服务器返回的一个唯一标识符
**注：不同appID下返回的openID是有可能重复的，在appID有重用的情况下建议使用uuid**；
UUID字段作用与openID一致，但保证任何情况下的唯一性；
userName字段存放的是用户的用户名；
proof字段用于登录成功后NCLoginModule模块创建的用户角色，该字段与openID配套使用；
UUIDProof字段作用与proof一样，该字段与UUID配套使用；

登录成功之后，我们可以通过引入头文件`<IleadSDK/IleadSDK.h>`去调用这四个字段。
**注意：只有登陆成功后才可以获取信息字段**


通过接口`[[IleadSDK sharedInstance] openID]`获取openID字段；

通过接口`[[IleadSDK sharedInstance] proof]`可以获取proof字段；

通过接口`[[IleadSDK sharedInstance] UUID]`可以获取UUID字段；

通过接口`[[IleadSDK sharedInstance] UUIDProof]`可以获取UUIDProof字段；

通过接口`[[IleadSDK sharedInstance] username]`可以获取username字段。

#### 3.4.7	设计一个接口按钮
登录成功后，我们需要在游戏主界面内设计一个接口按钮“ilead社区”，按钮的回调事件中我们调用`[[IleadSDK sharedInstance] ILeadCommunity]`接口进入社区，社区主页面包含了修改密码、修改用户名，以及密保管理三个接口；

示例如下：

```
-(void) UserCenter
{
	[[IleadSDK sharedInstance] ILeadCommunity];
}
```

#### 3.4.8	绑定账号
在游戏中，若用户以游客身份登录，在打开用户中心时，将弹出游客中心的界面。游客绑定成功后，将会进入用户中心，并发送`BindAccountSuccess`通知。

```
[[NSNotificationCenter defaultCenter] addObserver:self 
										 selector:@selector(BindBack) 
										     name:@"BindAccountSuccess" 
										   object:nil];
```
回调方法`BindBack`由游戏方自行定义，在该方法中开发人员可以通过上面3.4.4所介绍的接口`[[IleadSDK sharedInstance] UUID]`获取所需的唯一标示符。

#### 3.4.9	第三方登录
要实现第三方登录的话，需要引入头文件`<IleadSDK/IleadSDK.h>`,并且调用接口`[[IleadSDK sharedInstance] TPLogin: ID Platform: platform]`，这里的ID是第三方认证成功后返回的openID。

如下所示：

```
/**
 *  第三方平台登陆接口
 *
 *  @param ID       openID
 *  @param platform 平台名称
 */
- (void)TPLogin:(NSString*)ID Platform:(NSString*)platform;

```

#### 3.4.10	获取用户其他信息
游戏方可以调用`[[IleadSDK sharedInstance] getUserInfo]`接口获取当前用户的详细信息。该接口返回一个NSDictionary。
详细的key如下：

key | 描述 
:-----------  | :-----------
isGuest         | 用户是否是游客，1代表是，0代表否。       
isBindEmail     | 用户是否已经绑定邮箱，1代表是，0代表否。      

### 4 使用SDK交叉推广广告功能

#### 4.1 初始化广告功能

1. 在您使用广告功能前，您应该先初始化SDK，具体详见`3.3.初始化SDK`。
2. 使用广告功能，要先调用`[IleadSDK sharedInstance] initAd]`初始化广告功能。建议您在`-(BOOL)application:didFinishLaunchingWithOptions:`中调用此方法，并且要在`[[IleadSDK sharedInstance] setAppID:71 appKey:@"ei2hfs9e83qmc92jdz01a"];`之后才能调用。

#### 4.2 横幅广告

##### 4.2.1 初始化方法

方法 | 描述 
:-----------  | :-----------
-(instancetype)initWithRootViewController:(UIViewController *)rootViewController  | 返回一个320x50的横幅广告，广告将会显示在您传入的rootViewController的顶部居中。     
-(instancetype)initWithFrame:(CGRect)frame     | 返回一个自定义大小和显示位置的横幅广告，在调用此方法之后您还必须调用setRootViewController设置该横幅广告要放入的ViewController。

##### 4.2.2 主要方法

方法 | 描述 
:-----------  | :-----------
@property (nonatomic) UIViewController *rootViewController | 广告所在的ViewController。
@property (nonatomic) BannerPosition position | 设置广告显示位置，BannerPosition可选值为Top顶部居中，Bottom底部居中，Custom自定义。
@property (nonatomic) NSTimeInterval switchInterval | 设置广告切换时间，默认为60秒。如果自定义，则必须在调用-(void)loadAd方法前设置。 
@property (nonatomic) BOOL isReady | 查看横幅广告是否已经准备就绪，可以显示。
-(void)loadAd | 加载广告。

##### 4.2.3 示例

简单用法

	IleadBannerAd *bannerAd = [[IleadBannerAd alloc] initWithRootViewController:self];
    [self.view addSubview:bannerAd];
    [bannerAd loadAd];
    
自定义

	IleadBannerAd *bannerAd = [[IleadBannerAd alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
    bannerAd.rootViewController = self;
    bannerAd.switchInterval = 30.0;
    [self.view addSubview:bannerAd];
    [bannerAd loadAd];
    
#### 4.3 插页广告

##### 4.3.1 初始化方法

方法 | 描述 
:-----------  | :-----------
-(instancetype)init | 返回插页广告实例。

##### 4.3.2 主要方法

方法 | 描述 
:-----------  | :-----------
@property (nonatomic) BOOL isReady | 查看插页广告是否已经准备就绪，可以显示。
-(void)loadAd | 加载广告。
-(void)presentFromRootViewController:(UIViewController *)rootViewController | 展示插页广告。
@property (nonatomic,assign) id`<IleadInterstitialAdDelegate>` delegate | 委托对象，详见4.3.3

##### 4.3.3 IleadInterstitialAdDelegate

方法 | 描述 
:-----------  | :-----------
-(void)onAdReady | （可选）广告加载完毕后调用
-(void)onAdClosed | （可选）用户点击广告关闭按钮后调用


##### 4.3.4 示例

由于插页广告图片比较大且全屏显示，建议您在空闲的时候先调用`loadAd`加载广告，判断加载完后再在需要时展示。

	IleadInterstitialAd *interstitialAd = [[IleadInterstitialAd alloc] init];
    [interstitialAd loadAd];
    
    if([interstitialAd isReady]){
    	[interstitialAd presentFromRootViewController:self];
    }

    
