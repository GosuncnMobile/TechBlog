# iOS 12

## 新特性
- SiriKit API 快速从Siri里启动某些功能
- 推送消息分组 [参考链接](http://www.cocoachina.com/ios/20180627/23944.html) 
- 安全码自动填充,例如验证码
```C
if (@available(iOS 12.0, *)) {
    //Xcode 10 适配
    self.codeField.textContentType = UITextContentTypeOneTimeCode; 
}
```
[众多新功能，随 iOS 12 而来。](https://www.apple.com/cn/ios/ios-12/features/)
## 问题
### libstdc++
苹果移除了libstdc++库,而用libc++代替,该操作会导致Undefined symbols,编译失败  
以后所以的都应该使用libc++库进行开发
[参考链接](https://www.jianshu.com/p/ecced2f49e59)

### iPhone XR不支持3D-Touch
```C
if (self.traitCollection.forceTouchCapability == UIForceTouchCapabilityAvailable) {

}
```
```Swift
self.traitCollection.forceTouchCapability == .availible
```

### 获取Wi-Fi的Mac（BSSID）地址失败
需要在xcode打开权限
[适配iOS12](https://www.jianshu.com/p/08047c084b13)
