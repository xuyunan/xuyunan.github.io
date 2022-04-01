---
title: Objective-C Runtime Programming Guide
tags: [iOS]
date: 2019-06-07T13:00:24+08:00
---

以下内容根据官方文档和其他博客整理而得, 由于文档版本较旧, 有些api已经过时, 比如标记OBJC2_UNAVAILABLE的api, 此文档仅作了解, 在使用过程中, 还需多看代码注释和文档, 多动手测试.

<!--truncate-->

## 运行时版本与平台

老版本对应Objective-C 1.0, 新版本随Objective-C 2.0一起发布. 两个版本有什么区别呢?

- 在老版本中，如果你改变了一个类的实例变量，就需要重新编译所有继承自它的类
- 在新版本中，则不需要重新编译继承自它的类

什么应用程序是运行在新版本中呢?

iPhone应用程序和Mac OS X 10.5以后的64位应用程序使用的是新版本. 其它(Mac OS X的32位应用程序)使用的是旧版本.

## 与运行时的交互

oc中我们时时刻刻在跟运行时打交道，可以通过以下三种形式和运行时接触.

1. Objective-C源代码

oc是个动态语言，要依托于运行时才能执行，当我们写源代码时，其实已经在和运行时打交道了，比如对象方法调用，在运行时会被解释成objc_msgSend.

2. NSObject的方法

```objectivec
- (BOOL)isKindOfClass:(Class)aClass;        // 检测是否是某类的子类
- (BOOL)isMemberOfClass:(Class)aClass;      // 检测是否是某类的成员 
- (BOOL)conformsToProtocol:(Protocol *)aProtocol;   // 检测是否实现了协议
- (BOOL)respondsToSelector:(SEL)aSelector;  // 检测是否含有方法
- (IMP)methodForSelector:(SEL)aSelector;    // 返回一个指向方法实现的指针
```

这些方法恰恰是运用了运行时的能力.

3. 运行时方法

运行时系统是一个动态共享库，在头文件中，有很多的方法和数据结构，头文件位于/usr/include/objc

## 发送消息

objc_msgSend

方法调用[receiver message]在运行时会被解释成

```cpp
objc_msgSend(receiver, selector)                    // 无参数情况下  
objc_msgSend(receiver, selector, arg1, arg2, ...)   // 有参数情况下  
```

receiver是调用函数的对象，selector是message方法的选择器，通过它运行时可以找到对应的函数实现.

对象执行方法的具体过程

对象在运行时会被解释成结构体objc_object, 类会被解释成objc_class, objc_object里面只有一个isa指针, isa指针指向objc_class, objc_class里面保存着类的变量和方法等信息, 以下代码都能在/usr/include/objc/runtime.h中找到, 头文件中还有两个比较重要的是SEL和IMP, SEL也是一个结构体指针, 是方法的选择器，IMP是方法的实现, objc_class中有一个dispatch table, 在table里SEL和IMP一一对应.

```cpp
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

```cpp
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */
```

```cpp
/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;

/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```

当一个对象调用方法时, objc_object会通过isa指针找到当前的类objc_class, 检查cache中是否已经缓存了方法，没缓存就再在方法列表中找，找不到就会顺着super_class往父类查找, 然后在按照同样的方法，直到NSObject, 一旦定位到选择器之后，从dispatch table中找出方法的实现IMP, 然后执行.

查找过程

![messaging framework](/images/ios-messaging-framework.png)

#### 隐藏的参数

在方法调用过程中, 有两个隐藏的参数

self 这个就比较常见了,
_cmd, 就是当前方法的选择器, 类型为SEL, 结构体指针, 我们可以打印出来看看

```objectivec
NSLog(@"_cmd: %@", NSStringFromSelector(_cmd));
```

#### 获取方法地址

```objectivec
void (*setter)(id, SEL, BOOL);
int i;
setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];
for ( i = 0; i < 1000, i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```

通过methodForSelector可以获取一个指向函数实现的指针IMP, 然后执行它, 这样可以节省不少消息传递开销, 一般情况下我们都不需要这么做, 像上面这样, 同一个方法频繁执行时, 我们才考虑这么优化.

## 动态方法解析

有时候我们需要动态提供方法的实现, 像使用了@dynamic的属性, 该关键字告诉编译器, 属性相关的方法会在运行时动态提供(setter, getter).

```swift
@dynamic propertyName;
```

可以通过实现resolveInstanceMethod:来给类添加实例方法实现, 通过实现resolveClassMethod:来给类添加类方法实现. OC方法就是有至少两个参数(self, _cmd)的简单c函数, 我们使用class_addMethod函数来添加一个方法, 函数如下:

```cpp
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
```

通过resolveInstanceMethod:将方法resolveThisMethodDynamically动态的添加到类中

```objectivec
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```

动态加载

OC可以再运行时动态加载和链接新的类和类别, 这些代码被合并进程序，和启动时加载的类和类别同等对待. 原文中还有关于objc/objc-load.h, objc_loadModules的描述, 但是在iOS中，并不支持, macOS才支持.

## 消息转发

当我们给一个对象发送消息时, 若该对象不能处理该消息, 比如, 方法该对象没有该方法, 此时我们有两种方式将此消息转发到其它有能力处理的对象上.

forwardingTargetForSelector:

```objectivec
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if ([self.cat respondsToSelector:aSelector]) {
        return self.cat;
    } else {
        return nil;
    }
}
```

当forwardingTargetForSelector:没有实现, 或者返回nil时, 使用forwardInvocation:, 在使用该方法转发消息是, 需要同时实现methodSignatureForSelector:对选择器进行签名, 消息转发机制使用从这个方法中返回的信息来创建NSInvocation对象.

```objectivec
// [Cat respondsToSelector:anInvocation.selector]  处理Cat类方法
// [Cat instancesRespondToSelector:anInvocation.selector]   处理Cat实例方法
// [Cat instancesRespondToSelector:anInvocation.selector] 等价于 [self.cat respondsToSelector:aSelector]
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([Cat instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:self.cat];
    } else {
        [super forwardInvocation:anInvocation];
    }
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    NSMethodSignature *sign = [super methodSignatureForSelector:aSelector];
    if (!sign) {
        if ([Cat instancesRespondToSelector:aSelector]) {
            sign = [Cat instanceMethodSignatureForSelector:aSelector];
        }
    }
    return sign;
}
```

> forwardInvocation:在swift中是不支持的, 有关的api都已经被标示为UNAVAILABLE

思考多重继承

虽然OC不支持多重继承, 但是通过以上的功能, 我们可以模拟出多重继承的效果, 用一个类, 接收处理多个类的事件, 然后将事件通过消息转发分散到其它类中处理.

## 类型编码

为协助运行时系统，编译器会将每个方法中的返回值和参数类型编码成字符串并且和方法选择器联系起来, 给一个类型, @encode()会返回对应的编码字符串, 使用方法就像c语言的sizeof一样.

```objectivec
char *buf1 = @encode(int **);       // ^^i
char *buf2 = @encode(long);         // q
char *buf3 = @encode(void);         // v
char *buf4 = @encode(NSObject);     // {NSObject=#}
char *buf5 = @encode(NSObject *);   // @
char *buf6 = @encode(Class);        // #
// NSLog(@"%s", buf1);
```

函数的类型编码，我们也可以通过运行时函数获取

```cpp
const char * _Nullable method_getTypeEncoding(Method _Nonnull m) 
```

例如函数 void function(id self, SEL _cmd)的类型编码为”v@:”

想要查看其它类型编码, 可以到官方编码表查找 Table: Objective-C type encodings

参考

[Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)  
[【翻译】Objective-C Runtime Programming Guide](https://176zane.github.io/2017/11/03/【翻译】Objective-C%20Runtime%20Programming%20Guide/)  
[Objective-C Runtime Programming Guide中文版](https://danisfabric.gitbooks.io/objective-c-runtime-programming-guide/content/)  