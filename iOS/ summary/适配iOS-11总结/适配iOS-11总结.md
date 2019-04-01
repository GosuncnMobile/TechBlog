---
title: 适配iOS 11总结
date: 2018-01-04 13:59:01
categories:
 - 教程
tags:
 - iOS
 - 适配
---
### Human Interface Guideline的适配建议
#### 遵循Safe Area(安全区域)的界定
您的布局应在填满全屏超视网膜显示屏的同时，保证内容和控件正确显示，且便于点按。遵守安全区域的界定，以确保您的 app 与 iPhone X 新的屏幕比例合作无间。
![safeArea.jpeg](/img/适配iOS-11相关解决方法/safeArea.jpeg)
iOS 11 废弃了 iOS 7 之后出现的 topLayoutGuide/bottomLayoutGuide，取而代之的是safeLayoutGuide 概念。我们的UI元素都应该布局在这些区域之内，避免被各种 bar（NavgationBar、ToolBar、TabBar、StatusBar）遮挡。
<!--more-->
![safeArea1.jpeg](/img/适配iOS-11相关解决方法/safeArea1.jpeg)
![safeArea2.jpeg](/img/适配iOS-11相关解决方法/safeArea2.jpeg)
如果我们用了 AutoLayout，并且开启了 safeAreaLayoutGuide，布局会自动加上这些 safeLayoutGuide，你的视图不会超出这部分 SafeArea。如图所示，如果你需要增加 Guide 的区域，那幺可以设置 UIViewController.additionalSafeAreaInsets 来增加区域。
默认的safeAreaLayoutGuide:
![safeAreaLayoutGuideDefault.jpeg](/img/适配iOS-11相关解决方法/safeAreaLayoutGuideDefault.jpeg)
UIViewController.additionalSafeAreaInsets = UIEdgeInsetsMake(64, 0, 0, 0);的情况下:

![safeAreaLayoutGuideAddition.jpeg](/img/适配iOS-11相关解决方法/safeAreaLayoutGuideAddition.jpeg)

#### 状态栏
遵守安全区域的界定，在状态栏下面留出适当的空间。避免为状态栏高度预设值，这可能会导致您的内容被状态栏遮挡或形成错位。不可写死StatusBar的高度,iPhone X是44pt,其余为20pt."如果你的 App 是隐藏 StatusBar 的，建议重新考虑。iPhone X 为用户在垂直空间上提供了更多展示余地，且状态栏中也包含了用户需要知道的信息，除非能通过隐藏状态栏带给用户额外的价值，否则苹果建议大家将状态栏还给用户。"
另外还有一点，用户在使用 iPhone X 打电话的时候，StatusBar 的高度也不会发生变化了。
![statusBar.png](/img/适配iOS-11相关解决方法/statusBar.png)

#### 圆弧展示角和传感器槽
您的 app 的内容元素和控制按键应避开屏幕角落和传感器槽，让其在填满屏幕的同时不被角落切割。
![BarConrner.png](/img/适配iOS-11相关解决方法/BarConrner.png)

#### 主屏幕指示器
为使 app 的内容和控件始终保持清晰可见且便于点按，请确保您的 app 不会干扰主屏幕指示器。这部分的高度是34pt。
![homeIdi.png](/img/适配iOS-11相关解决方法/homeIdi.png)
![homeIndicator.jpeg](/img/适配iOS-11相关解决方法/homeIndicator.jpeg)

#### 调整视频的缩放度
Phone X 上的视频内容应填满屏幕。但是，如果这导致顶部或底部被切割，或侧面裁剪太多，则应将视频拉伸或缩小以配合屏幕。当 AVPlayerViewController 自动管理时，基于 AVPlayerLayer 的自定义视频播放器需要选择适当的初始视频重力设置，并允许用户根据自己的喜好在 aspect (固定宽高比) 和 aspectFill (固定宽高比且全屏) 观看模式之间进行切换。
>更多详细信息:[Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/overview/iphone-x/)

![videoScale.png](/img/适配iOS-11相关解决方法/videoScale.png)

#### 各个机型尺寸变化
![手机尺寸区别.png](/img/适配iOS-11相关解决方法/手机尺寸区别.png)
下图是 iPhone X 对比其他机型的变化部分。iPhone X 和 iPhone 8 的宽度一致，在垂直方向上多了145pt，这就意味着首页可以展示更多的内容.
![iponeX.jpeg](/img/适配iOS-11相关解决方法/iponeX.jpeg)
iPhone X 的坐标系统以及能显示内容的区域如下图所示：
![显示区域.jpeg](/img/适配iOS-11相关解决方法/显示区域.jpeg)

### 消除iPhone X展示界面的上下黑边
设置LaunchImage中对iPhone X的启动页图,或者设置LaunchScreen.storyboard
### SearchBar适配
在使用SearchBar时,当hidesNavigationBarDuringPresentation为Yes时,在iPhone X以下机型会出现SearchBar上移6pt的问题.
出现该问题的原因是在iOS11中,SearchBar的高度改变了,从44pt变成了50pt.当默认present一个SearchResultController的时候,会隐藏NavigationBar.但是,此时SearchBar和StatusBar加起来的高度被限制成64pt,所以SearchBar的Frame会向上偏移6pt,可以通过KVO去监听SearchBar的Frame属性,当其searchBar.frame.origin.y == 14时,来更改其Frame.
`使用KVO进行更改Frame不会使用户看到SearchBar视图位置的更改或者跳.`
```objectivec
//在该函数中处理Frame的变化
#pragma mark - View Life Cycle
//在退出该controller后移除观察者
- (void)viewDidDisappear:(BOOL)animated{
    [super viewDidDisappear:animated];
    //移除观察者
    if (@available(iOS 11, *)) {
        [self.sourceController.searchController.searchBar removeObserver:self forKeyPath:@"frame"];
    }
}
#pragma mark - UISearchControllerDelegate
//在展示该controller之前监听searchBar的"frame"属性
- (void)willPresentSearchController:(UISearchController *)searchController
{
    _strSearchKey = @"test";
    //修复iOS 11 search bar向上偏移
    if (@available(iOS 11, *)) {
        [searchController.searchBar addObserver:self forKeyPath:@"frame" options:NSKeyValueObservingOptionNew context:nil];
    }
}
#pragma mark - KVO
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    CGRect rect = [change[@"new"] CGRectValue];
    if (rect.origin.y == 14) {
        CGRect temp = rect;
        temp.origin.y = 20;
        [self.sourceController.searchController.searchBar setValue:@(temp) forKey:@"frame"];
    }else if (rect.size.height == 56){
        CGRect temp = rect;
        temp.size.height = 50;
        [self.sourceController.searchController.searchBar setValue:@(temp) forKey:@"frame"];
    }
}
```
* Navigation 集成 UISearchController
把你的UISearchController赋值给navigationItem，就可以实现将UISearchController集成到Navigation。
```objectivec
navigationItem.searchController //iOS 11新增属性
navigationItem.hideSearchBarWhenScrolling//决定滑动的时候是否隐藏搜索框
```

### ScrollView界面偏移20pt或者64pt.
#### 原因
原因是iOS 11中Controller的automaticallyAdjustsScrollViewInsets属性被废弃了，所以当tableView超出安全区域时系统自动调整了SafeAreaInsets值，进而影响adjustedContentInset值，在iOS 11中决定tableView的内容与边缘距离的是adjustedContentInset属性，而不是contentInset。因为系统对adjustedContentInset值进行了调整，所以导致tableView的内容到边缘的距离发生了变化，导致tableView下移了20pt（statusbar高度）或64pt（navigationbar高度)。
#### adjustContentInset属性的计算方式
首先看scrollView在iOS11新增的两个属性：adjustContentInset 和 contentInsetAdjustmentBehavior。
```objectivec
    /* Configure the behavior of adjustedContentInset.
Default is UIScrollViewContentInsetAdjustmentAutomatic.
*/@property(nonatomic) UIScrollViewContentInsetAdjustmentBehavior contentInsetAdjustmentBehavior
```
>adjustContentInset表示contentView.frame.origin偏移了scrollview.frame.origin多少；是系统计算得来的，计算方式由contentInsetAdjustmentBehavior决定。有以下几种计算方式：

1.UIScrollViewContentInsetAdjustmentAutomatic：如果scrollview在一个automaticallyAdjustsScrollViewContentInset = YES的controller上，并且这个Controller包含在一个navigation controller中，这种情况下会设置在top & bottom上 adjustedContentInset = safeAreaInset + contentInset不管是否滚动。其他情况下与UIScrollViewContentInsetAdjustmentScrollableAxes相同

2.UIScrollViewContentInsetAdjustmentScrollableAxes: 在可滚动方向上adjustedContentInset = safeAreaInset + contentInset，在不可滚动方向上adjustedContentInset = contentInset；依赖于scrollEnabled和alwaysBounceHorizontal / vertical = YES，scrollEnabled默认为yes，所以大多数情况下，计算方式还是adjustedContentInset = safeAreaInset + contentInset

3.UIScrollViewContentInsetAdjustmentNever: adjustedContentInset = contentInset

4.UIScrollViewContentInsetAdjustmentAlways: adjustedContentInset = safeAreaInset + contentInset

当contentInsetAdjustmentBehavior设置为UIScrollViewContentInsetAdjustmentNever的时候，adjustContentInset值不受SafeAreaInset值的影响。
#### 解决方法
设置tableView的contentInsetAdjustmentBehavior属性.
如果不需要系统为你设置边缘距离，可以做以下设置：
```objectivec
 //如果iOS的系统是11.0，会有这样一个宏定义“#define __IPHONE_11_0  110000”；如果系统版本低于11.0则没有这个宏定义
#ifdef __IPHONE_11_0
if ([tableView respondsToSelector:@selector(setContentInsetAdjustmentBehavior:)]) {
    tableView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
}
#endif
```
contentInsetAdjustmentBehavior属性也是用来取代automaticallyAdjustsScrollViewInsets属性的，推荐使用这种方式。
### UITableViewStyleGrouped造成的偏移
来着[Bugly](https://mp.weixin.qq.com/s/W1_0VrchCO50owhJNmJnuQ)的解决方法.
>tableView的style:UITableViewStyleGrouped类型，默认tableView开头和结尾是有间距的，不需要这个间距的话，可以通过实现heightForHeaderInSection方法（返回一个较小值：0.1）和viewForHeaderInSection（返回一个view）来去除头部的留白，底部同理。
>iOS 11上发生tableView顶部有留白，原因是代码中只实现了heightForHeaderInSection方法，而没有实现viewForHeaderInSection方法。那样写是不规范的，只实现高度，而没有实现view，但代码这样写在iOS 11之前是没有问题的，iOS 11之后应该是由于开启了估算行高机制引起了bug。添加上viewForHeaderInSection方法后，问题就解决了。或者添加以下代码关闭估算行高，问题也得到解决。

```objectivec
self.tableView.estimatedRowHeight = 0;
self.tableView.estimatedSectionHeaderHeight = 0;
self.tableView.estimatedSectionFooterHeight = 0;
```

### BackButton的图片设置
iOS 11对整一个NavigationBar视图的层级进行了修改,原本设置backButton的方式不再起效.
```objectivec
[[UIBarButtonItem appearance] setBackButtonBackgroundImage:[[UIImage imageNamed:@"navi_back"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 30, 0, 0)] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
```
会出现两个返回按钮,如图:
![iOS11BackButtonOlder.jpeg](/img/适配iOS-11相关解决方法/iOS11BackButtonOlder.jpeg)
相对应的视图层级:
![iOS11BackButtonOlderDetail.jpeg](/img/适配iOS-11相关解决方法/iOS11BackButtonOlderDetail.jpeg)
因此采用另外一种方式去设置backButton的图片:
```objectivec
UIImage *backImage = [UIImage imageNamed:@"navi_back"];
    [[UINavigationBar appearance] setBackIndicatorImage:backImage];
    [[UINavigationBar appearance] setBackIndicatorTransitionMaskImage:backImage];
```
iOS 10下的视图层级:
![iOS10BackButtonCurrent.jpeg](/img/适配iOS-11相关解决方法/iOS10BackButtonCurrent.jpeg)
iOS 11下的视图层级:
![iOS11BackButtonCurrent.jpeg](/img/适配iOS-11相关解决方法/iOS11BackButtonCurrent.jpeg)
### 去除BackTitle的方法
设置BackTitle的偏移量,根据是否iOS 11分别给一个偏移量.
```objectivec
//系统返回按钮处的title偏移到可视范围之外
    //iOS11 和 iOS11以下分别处理
    UIOffset offset = SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"11.0") ? UIOffsetMake(-200, 0) : UIOffsetMake(0, -80);
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:offset forBarMetrics:UIBarMetricsDefault];
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:offset forBarMetrics:UIBarMetricsCompact];
```
### rightBarButtonItem设置
因为在iOS11中leftBarButtonItems以及rightBarButtonItems的视图层级进行了更改,所有的BarItem都会放在UIButtonBarStackView里面.UIButtonBarStackView是一个相对布局.会根据item的个数自动设置各个Item间的距离以及约束.用`initWithCustomView:`生成的UIBarButtonItem在没加约束的情况下会在屏幕上错位,或者第一次能出现,当第二次视图出现时就消失,在iOS 11以下为正常.分析下视图层级发现,当不加约束的情况下,系统会给contentView外层添加一个UITAMICAdaptorView作为适配器.而加了约束之后,contentView外层便只是UIButtonBarStackView.如图.
`iOS11 BarButton的contentView不添加约束的情况下的视图层级:`
![iOS11ItemNoConstant.jpeg](/img/适配iOS-11相关解决方法/iOS11ItemNoConstant.jpeg)

`iOS11 BarButton的contentView添加约束的情况下的视图层级:`
![iOS11ItemConstant.jpeg](/img/适配iOS-11相关解决方法/iOS11ItemConstant.jpeg)

`iOS11 之前BarButton的contentView不添加约束的情况下的视图层级:`
![iOS10ItemNoConstant.jpeg](/img/适配iOS-11相关解决方法/iOS10ItemNoConstant.jpeg)
因此,解决方法在iOS 11下为给contentView添加约束,若一个items里面有多个item用contentView生成的话,也是同理分别设置约束.代码如下:
```objectivec
	_dropdownMenu = [[MKDropdownMenu alloc] initWithFrame:CGRectMake(0, 0, 100, 44)];
    self.dropMenuItem = [[UIBarButtonItem alloc]initWithCustomView:_dropdownMenu];
    if(@available(iOS 11, *)){
        [self.dropMenuItem.customView.heightAnchor constraintEqualToConstant:44].active = YES;
        [self.dropMenuItem.customView.widthAnchor constraintEqualToConstant:100].active = YES;
    }
    self.navigationItem.leftBarButtonItem = self.dropMenuItem;
```
### 流式布局的iPhone X适配
流式布局中的cell无需避开home indicator区域,直接展示即可.虽然cell的边角可能会被home indicator区域的圆角遮挡,减少某些信息.但给予用户的可视空间更大,而且与home indicator有冲突的交互操作可以上拉至安全区域进行点击.
### 适配到iOS8的布局设置
SafeArea最低适配到iOS9,但项目要求最低适配到iOS8.因此可以先使用topLayoutGuide和BottomLayoutGuide代替.
### 横屏时tableview的适配
* 先看几个在iPhone X上适配错误的例子
![tableViewEroor1.gif](/img/适配iOS-11相关解决方法/tableViewEroor1.gif)

![tableViewError2.gif](/img/适配iOS-11相关解决方法/tableViewError2.gif)

* 头部导航栏不予许进行用户交互的，意味着上面这两种情况 Apple 官方是不允许的
* 使用官方推荐的safe Area在大多数情况下可以解决问题,但在横屏情况下还需要额外进行适配.

![tableViewLandError.png](/img/适配iOS-11相关解决方法/tableViewLandError.png)
* 产生这个原因代码是：`[headerView.contentView setBackgroundColor:[UIColor headerFooterColor]]，`这个写法看起来没错，但是只有在 iPhone X上有问题
* 原因:

![tableViewLandDetail.png](/img/适配iOS-11相关解决方法/tableViewLandDetail.png)
* 解决方法：设置backgroundView颜色 `[headerView.backgroundView setBackgroundColor:[UIColor headerFooterColor]]`

![ios11tableViewLandRight.jpeg](/img/适配iOS-11相关解决方法/ios11tableViewLandRight.jpeg)

### 隐藏底部Indicator
如果业务需求需要隐藏底部Indicator(apple官方不推荐隐藏)
```objectivec
// 在VC里面重写下面这个方法即可
- (BOOL)prefersHomeIndicatorAutoHidden{
    return YES;
}
```
### 发送原图功能
对于IM的发送原图功能，iOS11启动全新的**HEIC**格式的图片，iPhone7以上设备+iOS11拍出的live照片是`.heic`格式图片，同一张live格式的图片，iOS10发送就没问题（转成了jpg），iOS11就不行.
* 微信的处理方式是一比一转化成**jpg**格式
* QQ和钉钉的处理方式是直接压缩,即使是原图也压缩为非原图
* 可采取微信的方案,使用以下代码转化成jpg格式
```objectivec
// 0.83能保证压缩前后图片大小是一致的
// 造成不一致的原因是图片的bitmap一个是8位的，一个是16位的
imageData = UIImageJPEGRepresentation([UIImage imageWithData:imageData], 0.83);
```

### 权限处理
iOS 11中,隐私权限配置发生了改变,将原来的相册访问权限放开了,现在有读写两种权限.
>iOS 11访问权限列表

|隐私数据|对应key值|提示语|
|:------:|:------:|:------:|
|NFC(使用)|NFCReaderUsageDescription|"XXX"需要您的同意，才能使用NFC功能|
|相册(读)|NSPhotoLibraryUsageDescription|"XXX"需要您的同意，才能访问相册|
|相册(写)|NSPhotoLibraryAddUsageDescription|"XXX"需要您的同意，才能添加照片|
|相机|NSCameraUsageDescription|"XXX"需要您的同意，才能访问相机|
|麦克风|NSMicrophoneUsageDescription|"XXX"需要您的同意，才能访问麦克风|
|位置|NSLocationUsageDescription|"XXX"需要您的同意，才能访问位置|
|在使用期间访问位置(前台)|NSLocationWhenInUseUsageDescription|"XXX"需要您的同意，才能在使用期间访问位置|
|始终访问位置|NSLocationAlwaysUsageDescription|"XXX"需要您的同意，才能始终访问位置|
|日历|NSCalendarsUsageDescription|"XXX"需要您的同意，才能访问日历|
|提醒事项|NSRemindersUsageDescription|"XXX"需要您的同意，才能访问提醒事项|
|运动与健身|NSMotionUsageDescription|"XXX"需要您的同意，才能访问运动与健身|
|健康更新|NSHealthUpdateUsageDescription|"XXX"需要您的同意，才能访问健康更新|
|健康分享|NSHealthShareUsageDescription|"XXX"需要您的同意，才能访问健康分享|
|蓝牙|NSBluetoothPeripheralUsageDescription|"XXX"需要您的同意，才能访问蓝牙|
|媒体资料库|NSAppleMusicUsageDescription|"XXX"需要您的同意，才能访问媒体资料库|
#### 近场通讯NFC权限案例
>[iOS 11 Core NFC - any sample code?](https://stackoverflow.com/questions/44380305/ios-11-core-nfc-any-sample-code)

#### ios11 地图定位权限设置
iOS 11在定位权限设置上有更新，可以按以下方式进行设置：
在项目的 Info.plist 添加定位权限申请，根据业务需求，选择下列方式设置。
其中：
`iOS 8 - iOS 10 版本：` 
NSLocationWhenInUseUsageDescription 表示应用在前台的时候可以搜到更新的位置信息。
NSLocationAlwaysUsageDescription 申请Always权限，以便应用在前台和后台（suspend 或 terminated）都可以获取到更新的位置数据。

`iOS 11 版本：`
NSLocationAlwaysAndWhenInUseUsageDescription 申请Always权限，以便应用在前台和后台（suspend 或 terminated）都可以获取到更新的位置数据（NSLocationWhenInUseUsageDescription 也必须有）。

`iOS 8-iOS 11`
如果需要同时支持在iOS8-iOS10和iOS11系统上后台定位，建议在plist文件中同时添加NSLocationWhenInUseUsageDescription、NSLocationAlwaysUsageDescription和NSLocationAlwaysAndWhenInUseUsageDescription权限申请。

---
参考:
[1.关于刘海打理这种事儿，美团点评的iOS工程师早就有经验了，不信你看！](https://www.imuo.com/a/943178cc983067710b47289d983da868881bea05e142bc6014324f0cb9a91d82)
[2.iOS11、iPhone X、Xcode9 适配指南](https://www.jianshu.com/p/f5ee206c7df0)
[3.为 iPhone X 更新您的 app。](https://developer.apple.com/cn/ios/update-apps-for-iphone-x/)
[4.iOS 11 安全区域适配总结](https://mp.weixin.qq.com/s/W1_0VrchCO50owhJNmJnuQ)