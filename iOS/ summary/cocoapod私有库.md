# Cocoapod私有库
[TOC]
## 建立私有pod lib
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
- **注意** 如果dependency是另外一个私有库,则应该在podspec写私有库的名字和版本号,在引用项目的Podfile里写私有库的引用地址
- **注意** podspace的 s.source 地址要正确,不然无法获得正确的版本号
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
### 编写库文件
打开pod 里面的Replace.Swift,把库文件放进里面.或者直接在里面编辑.

- 新建文件的位置很可能错乱,如果文件不在Classes,会导致Pod不下来
    - 可以选中Replace.Swift然后add,或者在Finder里新建再拖进去
    - Add Targets要选择正确的Target!!!(他会出现很多个而且默认的是首字母排序第一个)
- 要分清open,public,private的权限控制
- 如果要直接用Example工程调试,需要每次都Clean Build Folder
- Swift可以直接复制出来Playground进行调试

### 静态库
静态库.a文件可以新建一个lib配置,然后在podspec里加入以下配置
```ruby
  s.frameworks = 'CoreGraphics'#用到的系统frameworks
  s.source_files = 'GSStaic/source/*.h'#头文件
  s.static_framework = true
  s.vendored_libraries = 'GSStaic/lib/*.a'#库文件
  s.xcconfig = { 'LD_RUNPATH_SEARCH_PATHS' => '$(PODS_ROOT)/GSStaic/source' }#头文件查找目录
```

## Git
在github上新建一个Repository,然后把整个lib推到远程库上.注意.podspec文件要在根目录地址
删除.gitignore最后一行Pods/前的#,即不上传ods/
```sh
git remote add origin https://127.0.0.1/GosuncnMobile/GSSwiftKit.git
git pull
git add.
git commit -a -m ""
git push origin master
```
此时,就可以通过Pod来集成库了,例如
```ruby
pod 'GSSwiftKit', :git => 'https://127.0.0.1/GosuncnMobile/GSSwiftKit.git'
```
这样的pod是没有版本管理的,类似于快照版本
### 版本管理
先修改**podspec**里的版本号
```sh
git tag 0.1.0
git push --tags
```
这样就可以通过
```ruby
pod 'GSSwiftKit', :git => 'https://127.0.0.1/GosuncnMobile/GSSwiftKit.git', :tag => '0.1.0'
```
另外,branch也可以获取
```ruby
pod 'GSSwiftKit', :git => 'https://127.0.0.1/GosuncnMobile/GSSwiftKit.git', :branch => 'CQ'
```

### 私用spec
在Git服务上新建仓库,例如GSSpec,同时新建Readme.md
```shell
pod repo add GSSpec http://127.0.0.1/GosuncnMobile/GSSpec.git
```
私用库一定要打好和podspec对应的Tag,然后才能提交
```shell 
pod push GSSwiftKit GSSpec --allow-warnings
#--allow-warnings是忽略警告信息
```
使用的时候,podfile用同时配置官方的source 和 私有 source
```ruby
use_frameworks!
source 'https://github.com/CocoaPods/Specs.git' #官方仓库地址
source 'http://127.0.0.1/GosuncnMobile/GSSpec.git'  #私有仓库地址(spec)
target 'Example' do
    pod 'RxSwift'
    pod 'GSSwiftKit'
end 
```
## SVN
先要设置全局的SVN用户名和密码
```shell
svn export --non-interactive --trust-server-cert --force --username USERNAME --password PASAWORD http://svnpath/projectName/ DEST_FOLDER
```
由于SVNtag管理比较麻烦,可以直接在SVN地址后面加入Tag就行了
```ruby
pod 'GSNetWork', :svn =>'http://127.0.0.1/svn/GSSwiftKit/tags/0.1.0 '
```


## 参考连接
[cocoapod搭建私有库超级详细教程](https://www.jianshu.com/p/9992feb8b00b)
[CocoaPods 私有源配置](https://www.jianshu.com/p/71b1f57b0ea1)