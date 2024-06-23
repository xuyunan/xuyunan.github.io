---
title: "大前端领域下, 都是有哪些常用单位"
tags: [大前端, 前端, Web, iOS, Android]
date: 2022-05-18T23:40:06+08:00
---

## WEB端常用单位

web端常用的单位比较多，有px、em、rem、%、vw、wh、vmin、vmax等，可以将其分为两类，一类是绝对单位，一类是相对单位

**绝对单位**, px: (pixels), 即像素单位，如果元素是固定大小的，那么px无疑是最好的选择，常用于margin、padding、shadow  

**相对单位**, 相对单位用来处理响应式网页很有帮助，em、rem常用于font size、width、height，%、vw、vh、vmin、vmax常用于布局. 

- em: (emeters), 相对于父级元素，默认font size = 16px
- rem: (root emeters) 相对于根元素(HTML tag)，默认font size = 16px
- %: 相对于父级元素
- vw: 相对于浏览器窗口宽度(viewport's width)
- vh: 相对于浏览器窗口高度(viewport's height)
- vmin: 取vw和vh的最小值
- vmax: 取vw和vh的最大值

举个例子

在大多数的浏览器中，默认字体大小是16px，那么em、rem就是相对这个值来做改变

```
1em = 16px (1 * 16)
2em = 32px (2 * 16)
.5em = 8px (.5 * 16)

1rem = 16px
2rem = 32px
.5rem = 8px

100% = 16px
200% = 32px
50% = 8px
```

如果我们把默认的大小改为14px，那么实际的值也会相应变化

```
1em = 14px (1 * 14)
2em = 28px (2 * 14)
.5em = 7px (.5 * 14)

1rem = 14px
2rem = 28px
.5rem = 7px

100% = 14px
200% = 28px
50% = 7px
```

然后来看看vw和vh，假如窗口大小是480px x 800px，那么

```
1vw = 1% 的窗口宽度 (4.8px)
50vw = 50% 的窗口宽度 (240px)

1vh = 1% 的窗口高度 (8px)
50vh = 50% 的窗口高度 (400px)
```

为什么在看别人代码的时候，会有人将body的默认设置改为 font-size：62.5% ? 这是因为设置过后，方便计算，此时的1em=16px * 62.5% = 10px, 1.2em=12px, 1.8em=18px

## 小程序端常用单位 rpx

小程序中的 WXSS 对 CSS 进行了扩充以及修改, 除了有一些上面提到的单位外，还增加了 rpx (responsive pixel)，用作响应式单位，需要特殊说明的是，小程序把屏幕宽度设定为750rpx，永久的把屏幕宽度平分成了750份，假设把组件宽度设定为750rpx，则无论在什么设备上，这个组件的宽度都是跟屏幕一样宽.

iPhone6的物理像素宽度恰好是750px，也就是说在iPhone6上，1rpx就是一个物理像素.

## iOS端常用单位 pt

在iOS开发中，使用的单位是pt，在 iPhone4 以前，1pt = 1px，但是苹果为了让屏幕内容看起来更清晰，引入了多倍屏的概念，也就出现了更清晰的retina屏手机，这是在有限的空间内显示更多像素的一种技术；iPhone4以后开始使用2倍屏，如果想要图片显示看起来正常，就需要使用2倍图，也就是image@2x.png，这张图具体的像素需要多大呢，如果原来使用的是100x100px的图，那么image@2x.png就需要是200x200px；iPhone6 6s 7 8还是2倍屏，到了6+,6s+,7+,8+及以后开始使用更为清晰的3倍屏，也就是说在同样一块屏幕的大小上，我们可以展示300x300px的图片.

网络上很多文章写在2倍屏下1pt = 2px，其实是不准确的，1pt应当是4px，也就是一个点里面有2x2个物理像素.
    
## Android端常用单位 dp

由于Android的源代码是开源的，所以就有很多厂商用Android系统研发了各种尺寸的手机，也就没有标准的1倍图，2倍图，3倍图，而是通过ldpi, mdpi, hdpi, xhdpi, xxhdpi划分了几个密度等级，越往后用在越高清屏幕的手机上。

```
ldpi：对应的dpi范围为0 ~ 120，也就是说每英寸有0到120个像素点的屏幕的屏幕密度都属于低密度
mdpi：dpi范围为120 ~ 160
hdpi：dpi范围为160 ~ 240
xhdpi：dpi范围为240 ~ 320
xxhdpi：dpi范围为320 ~ 480
```

在Android开发中，常用的单位是dp和sp，dp常用于组件，sp常用于字体. 其他还有一些不常用的单位，比如px, in, mm.  
dp跟pt类似, Android的基本屏幕密度160dpi(英寸160点)，就是说在密度为160的设备上1dp = 1px，在320dpi的设备上，1dp = 2px.  
sp跟dp类似，但是sp还受系统字体大小缩放比例的影响，更为适合用在字体单位.  

参考

[Pixels vs points](https://www.kickassux.com/ux-library/pixels-vs-points)  
[What are pixels and points in iOS?](https://stackoverflow.com/questions/12019170/what-are-pixels-and-points-in-ios)  
[The Ultimate Guide To iPhone Resolutions](https://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions)  
[List of iOS and iPadOS devices](https://en.wikipedia.org/wiki/List_of_iOS_and_iPadOS_devices)  
[What is the difference between "px", "dip", "dp" and "sp"?](https://stackoverflow.com/questions/2025282/what-is-the-difference-between-px-dip-dp-and-sp)  

