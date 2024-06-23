---
title: 创建开源POD
tags: [iOS]
date: 2017-04-03T13:00:24+08:00
---

## 准备内容

这里我使用YNWebViewController，一开始目录结构是这样的

```
├── Demo
├── LICENSE
├── README.md
└── YNWebViewController
```

<!--truncate-->

要分享的代码和资源文件都在YNWebViewController文件夹里了, 项目源代码在[Github](https://github.com/xuyunan/YNWebViewController)上。

先给项目打个tag，后面要用到

```bash
git tag 0.1.5
git push --tags
```

## Pod描述文件

- pod spec create YNWebViewController create后面跟的是项目名，在项目主目录下执行该命令后，系统会自动在该目录下创建一个名为YNWebViewController.podspec的描述文件，用于记录项目信息。
- 在podspec文件里，写入相关的项目信息，如：作者，项目地址，要包含的源文件和资源文件路径，开源协议等，具体还是要看里面的注释。[Podspec Syntax Reference](https://guides.cocoapods.org/syntax/podspec.html)
- pod spec lint YNWebViewController.podspec 然后我们该命令来验证这个描述文件有没有错误， 没问题的话，就把这个文件也push到项目git。

## 上传描述文件

- pod trunk register xu_yunan@163.com 'Tommy Xu' --description='macbook pro' 首先在你电脑上注册信息, 注册好之后可以用 pod trunk me 查看个人信息。
- pod trunk push YNWebViewController.podspec 接下来使用这个命令，将pod描述文件推至cocoapods库。
- 成功之后就可以在cocoapods网站上看到你的库了YNWebViewController

> 经过以上几个步骤，一般情况下，别人就已经可以使用你的开源库了。如果在执行命令的时候总提示timeout，那就挂个梯子吧。

## 遇到的问题

以下是我在创建的过程遇到的几个问题及解决方法，希望对你有用:

1. 明明已经提示上传好了，却还搜不到，这时就需要把本地缓存删掉
rm ~/Library/Caches/CocoaPods/search_index.json

2. 搜到之后，在使用的过程中报错 Cannot call value of non-function type ‘module‘
将class和需要开放出来的方法和属性声明为public

3. 方法，属性都能正常使用了，但是图片资源加载不出来，此时需要更换图片加载方式 Pod images not loading

```swift
let bundle = Bundle(for: YNWebViewController.self)
let backImage = UIImage(named: "YNWebViewControllerBack.png", in: bundle, compatibleWith: nil)
```

[项目地址](https://cocoapods.org/pods/YNWebViewController)  
[测试项目地址](https://github.com/xuyunan/YNPodDemo)  

## 参考

[CREATE A POD](https://cocoapods.org/)  
[Getting setup with Trunk](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)  
[Delete a tag](http://stackoverflow.com/questions/5480258/how-to-delete-a-git-remote-tag)   

