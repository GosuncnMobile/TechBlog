# Cocoapod私有库
## 建立本地pod 工程
```sh
pod lib create 库名
```
接下来会问几个问题,按需求选择
```sh
What platform do you want to use?? [ iOS / macOS ]
 > iOS

What language do you want to use?? [ Swift / ObjC ]
 > Swift

Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Quick / None ]
 > Quick

Would you like to do view based testing? [ Yes / No ]
 > No

Running pod install on your new library.
```

### 配置.podspec
打开默认生产的.podspec里面直白明了按着填就是了.就写几个要注意的点
- dependency Lib的依赖库,多个另起一行.
- **注意** 如果你的库不需要依赖这个库,但是你的Example需要依赖它(例:需要SnapKit来编写Example界面,库本身不涉及SnapKit).那这个依赖写在Example/Podfile里
```ruby
Pod::Spec.new do |s|
    s.dependency 'Moya', '~> 12.0'
    s.dependency 'RxSwift',    '~> 4.0'
    s.dependency 'RxCocoa',    '~> 4.0'
    s.dependency 'Moya/RxSwift', '~> 12.0'
end
```
- swift_version
    默认文件不生成这个选项,需要手动加入.    
```ruby
Pod::Spec.new do |s|
   s.swift_version = '4.2'
end
```

完成后使用下面命令验证库是否正常
```sh
pod lib lint
```
## 编写库文件
打开pod 里面的Replace.Swift,把库文件放进里面.或者直接在里面编辑.

- 新建文件的位置很可能错乱,如果文件不在Classes,会导致Pod不下来
    - 可以选中Replace.Swift然后add,或者在Finder里新建再拖进去
    - Add Targets要选择正确的Target!!!(他会出现很多个而且默认的是首字母排序第一个)
- 要分清open,public,private的权限控制
- 如果要直接用Example工程调试,需要每次都Clean Build Folder
- Swift可以直接复制出来Playground进行调试

## Git
在github上新建一个Repository,然后把整个lib推到远程库上.注意.podspec文件要在根目录地址
```sh
git remote https://github.com/GosuncnMobile/GSSwiftKit.git
git pull
git add.
git commit -a -m ""
git push origin master
```
此时,就可以通过Pod来集成库了,例如
```ruby
pod 'GSSwiftKit', :git => 'https://github.com/GosuncnMobile/GSSwiftKit.git'
```
这样的pod是没有版本管理的,类似于快照版本
#### 版本管理

## SVN
