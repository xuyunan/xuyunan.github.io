---
title: iOS中的锁(四)-NSLock, NSRecursiveLock, NSConditionLock
tags: [iOS]
date: 2019-06-22T13:00:24+08:00
---

这几种锁都是对pthread_mutex的封装

NSLock 遵循 NSLocking 协议. lock 加锁，unlock 解锁，tryLock 尝试加锁，如果失败的话返回 NO，lockBeforeDate: 是在指定Date之前尝试加锁，如果在指定时间之前都不能加锁，则返回NO

<!--truncate-->

```objectivec
[self.lock lock];
// todo
[self.lock unlock];
```

NSRecursiveLock 是通过pthread_mutex封装的递归锁, 同一个线程可以多次获取同一个递归锁，不会产生死锁，加锁锁解锁次数也必须相同.

```objectivec
- (void)recursiveFunc:(NSInteger)value
{ 
　　[theLock lock]; 
　　if (value != 0) { 
　　　　--value; 
　　　　[self recursiveFunc:value]; 
　　}
　　[theLock unlock];
}
```

NSConditionLock 即条件锁，可以设置自定义的条件来获得锁和释放锁. 比如

```objectivec
- (void)produce
{
　　while (1) {
       [theLock lockWhenCondition:NO_DATA];
　　　　// create data
　　　　[theLock unlockWithCondition:HAS_DATA];
　　}
}

- (void)consume
{
　　while (1) {
　　　　if ([theLock tryLockWhenCondition:HAS_DATA]) {
　　　　　　// display data
　　　　　　[theLock unlockWithCondition:NO_DATA];
　　　　}
　　　　sleep(1.0); // sleep a while
　　}
}
```