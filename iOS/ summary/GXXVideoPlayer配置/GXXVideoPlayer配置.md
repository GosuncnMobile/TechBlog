---
title: GXXVideoPlayer配置
date: 2017-12-04 15:22:27
categories:
  - 教程
tags:
  - iOS
  - video
---
#### 在Link Binary With Libraries中添加libGXXVideoPlayer.a和libvideostack.a两个静态库。
#### 在User Header Search Paths中拖入GXXVideoPlayer和videostack两个库的地址。

![1.jpeg](1.jpeg)

#### 在Link Binary With Libraries中加入7个动态库：libstdc++.6.0.9.tbd,libz.1.2.8.tbd,libz.tbd,libbz2.tbd,libiconv.tbd, VideoToolbox.framework,CoreMedia.framework
<!--more-->
![2.jpeg](2.jpeg)


#### 在Other Linker Flags添加-ObjC,-lbz2,-liconv（注意大小写）

![3.jpeg](3.jpeg)

#### 在Preprocessor Marcos中，Debug添加：DEBUG=1,_LINUX,_IOS,SD_WEBP,Release添加：_IOS,_LINUX

![4.jpeg](4.jpeg)

![5.jpeg](5.jpeg)

![6.jpeg](6.jpeg)

#### 把项目中至少一个文件的.m后缀改成.mm支持c++.（例如修改AppDelegate.m为AppDelegate.mm）

![7.jpeg](7.jpeg)

