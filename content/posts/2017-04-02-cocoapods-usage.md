---
title: 使用COCOAPODS
tags: [iOS]
date: 2017-04-02T13:00:24+08:00
---

## 简介

可以说这是一个平台，因为它保存了无数的开源库。 可以说是一个工具，一个帮你管理第三方库的工具。

<!--truncate-->

## 安装

pod 检查是否已经安装，如果出现pod COMMAND，说明你已经安装过了。
sudo gem install cocoapods 安装或者更新
pod search Alamofire 搜索

## 使用

- 首先我创建了个项目YNPodDemo
- 切到项目主目录，执行pod init，系统就会自动帮你创建一个名为Podfile的文件
- 将你要使用的库写进去 pod 'Alamofire', '~>4.3.0'
-  执行pod install，将使用的库安装到本地
- 之后就会在工程文件夹下多出一个类似YNPodDemo.xcworkspace这样的工程文件，以后用这个打开项目就可以了，不要在使用YNPodDemo.xcodeproj
- 导入&使用

在使用库时，如果有错误提示 Cannot load underlying module for ‘Alamofire’, clean然后build。[issues441](https://github.com/Alamofire/Alamofire/issues/441)

## 卸载

sudo gem uninstall cocoapods 一般也用不到

## 参考

[cocoapods.org](https://www.cocoapods.org/)  
[How to Use CocoaPods with Swift](https://www.raywenderlich.com/97014/use-cocoapods-with-swift) 

## Podfile示例

```ruby
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'YNPodDemo' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for YNPodDemo
  pod 'Alamofire', '~>4.3.0'

  target 'YNPodDemoTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'YNPodDemoUITests' do
    inherit! :search_paths
    # Pods for testing
  end

end
```

