---
title: 创建私有POD
tags: [iOS]
date: 2017-04-04T13:00:24+08:00
---

## 为什么搭建私有库

私有库可以保证代码的安全性，亦可以将复用性强的代码、控件封装成模块，提供给部门或公司的人员用。

<!--truncate-->

## 搭建过程

首先需要创建一个私有git仓库，用来存放描述文件，可以使用bitbucket，假设创建了一个名为REPO_NAME的私有仓库。然后创建一个可复用的项目叫HexColors, 是一个swift中使用十六进制颜色的扩展，放到了bitbucket上。并且打了tag，HexColors.podspec文件也已经创建好了，整个过程和创建开源库没有什么区别。

关联私有库地址

```bash
pod repo add REPO_NAME SOURCE_URL
```

验证是否配置正确

```bash
cd ~/.cocoapods/repos/REPO_NAME
pod repo lint .
```

将刚刚创建好的pod描述文件，推至私有仓库

```bash
pod repo push REPO_NAME SPEC_NAME.podspec
```

此时，你的私有仓库应该是这个结构

```bash
.
├── Specs
    └── [SPEC_NAME]
        └── [VERSION]
            └── [SPEC_NAME].podspec
```

## 使用

切到要使用的项目，这里我还用我的YNPodDemo，在Podfile里加上这样一行，来指定私有和共开仓库

```bash
# Private Specs
source 'URL_TO_REPOSITORY'
# Public Specs
source 'https://github.com/CocoaPods/Specs.git'
```

然后添加HexColors

```bash
pod 'HexColors', '~>0.0.1'
```

之后就跟使用开源库一样了

```bash
pod install
```

## YNPodDemo

这个Demo打了三个tag:

0.1 测试使用Cocoapods   
0.2 测试使用自己创建的开源库   
0.3 测试使用自己创建的私有库  

使用下面这条命令可以检出特定Tag对应的代码

```bash
git clone --branch <tag_name> <repo_url>
```
