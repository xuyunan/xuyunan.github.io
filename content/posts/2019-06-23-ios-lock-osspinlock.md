---
title: iOS中的锁(五)--OSSpinLock, os_unfair_lock
tags: [iOS]
date: 2019-06-23T13:00:24+08:00
---

OSSpinLock 是一种自旋锁, 和互斥锁类似，都是为了保证线程安全的锁。但二者的区别是不一样的，对于互斥锁，当一个线程获得这个锁之后，其他想要获得此锁的线程将会被阻塞，直到该锁被释放。但自旋锁不一样，当一个线程获得锁之后，其他线程将会一直循环在哪里查看是否该锁被释放。所以，此锁比较适用于锁的持有者保存时间较短的情况下。

<!--truncate-->

```objectivec
spinLock = OS_SPINLOCK_INIT;
OSSpinLockLock(&spinLock);
sleep(4);
OSSpinLockUnlock(&spinLock);
```

为什么说这种自旋锁不再安全了?

优先级翻转的问题

新版 iOS 中，系统维护了 5 个不同的线程优先级/QoS: background，utility，default，user-initiated，user-interactive。高优先级线程始终会在低优先级线程前执行，一个线程不会受到比它更低优先级线程的干扰。这种线程调度算法会产生潜在的优先级反转问题，从而破坏了 spin lock。

具体来说，如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock。这并不只是理论上的问题，libobjc 已经遇到了很多次这个问题了，于是苹果的工程师停用了 OSSpinLock。

iOS10以后，苹果给出了新的api, 当然也可以通过前面几章提到的各种锁

```objectivec
os_unfair_lock_t unfairLock = &(OS_UNFAIR_LOCK_INIT);
os_unfair_lock_lock(unfairLock);
sleep(4);
os_unfair_lock_unlock(unfairLock);
```

参考 

[不再安全的 OSSpinLock](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)  