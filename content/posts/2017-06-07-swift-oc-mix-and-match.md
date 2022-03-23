---
title: Swift和OC混用
tags: [iOS]
date: 2017-06-07T13:00:24+08:00
---

## 前言

你有一个使用Objective-c语言写的项目，想要混用部分Swift代码，于是你创建了一个Swift文件，就在此时，Xcode提示会生成一个跟你项目名称相关联的桥接文件ProductModuleName-Bridging-Header.h, 我们暂且叫它OC桥接文件, 其实Xcode还创建了另外一个桥接文件，叫ProductModuleName-Swift.h, 这个文件在列表里看不到而已, 我们暂且把它叫做Swift桥接文件。

## Swift使用OC代码

想要在Swift中使用OC代码，我们只需要做两件事情

1、将要使用的OC文件, 在OC桥接文件导入

```objectivec
#import "XYZCustomCell.h"
#import "XYZCustomView.h"
#import "XYZCustomViewController.h"
```

2、确保Build Settings里, Objective-C Bridging Header这个选项里有正确的桥接文件相对地址, 一般系统都帮你填好了, 类似ProductModuleName/ProductModuleName-Bridging-Header.h

同一个target下不需要导入任何代码即可使用, 不用target, 要导入模块名.

## OC使用Swift代码

1、Swift桥接文件是Xcode自动处理的，我们只需要在写Swift代码的时候稍微主要下就可以了，那么具体要注意什么?

使用public或者open标记的代码，将会无条件的引入到桥接文件中; 关于被internal标记的代码，只有在项目中有OC桥接文件的情况下，才会被引入到Swift桥接文件中，之后暴露给OC代码; private和fileprivate标记的代码是不会被OC代码所看到的。

:::tip 
如果被private和fileprivate标记的代码块里含有@IBAction, @IBOutlet, or @objc, 那么这些代码也将会暴露给OC。
:::

2、如果不再一个模块里，需要确保Build Settings里, Defines Module这个选项的值为YES, 才能混用

使用Swift代码时, 需要导入Swift桥接文件

```objectivec
#import "ProductModuleName-Swift.h"
```

如果不在一个模块，怎么办?

如果混用时不再一个模块, 需要导入模块名

1、将模块引入Swift代码中

```swift
import FrameworkName
```

2、将模块引入OC代码中

```swift
@import FrameworkName;
```