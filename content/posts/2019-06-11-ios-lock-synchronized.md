---
title: iOS中的锁(二)--synchronized
tags: [iOS]
date: 2019-06-11T13:00:24+08:00
---

11年刚入道的时候，只会用这么一种锁，简单方便

```objectivec
@synchronized (obj) {  }
```

<!--truncate-->

我们在main函数里写段简单的代码，然后用clang命令看看它的具体实现

```cpp
id _rethrow = 0; id _sync_obj = (id)obj; objc_sync_enter(_sync_obj);
try {
	struct _SYNC_EXIT {
		_SYNC_EXIT(id arg) : sync_exit(arg) {}
		~_SYNC_EXIT() {objc_sync_exit(sync_exit);}
		id sync_exit;
	} _sync_exit(_sync_obj);
	// TODO:
} catch (id e) {
    _rethrow = e;
}
```

从上面我们可以看出

1、使用的是objc_sync_enter和objc_sync_exit来加锁和解锁  
2、创建了一个结构体，充当了一个defer的角色, 当前代码块执行完之后会执行析构函数, 以释放锁资源

关于锁的两个函数声明我们可以在usr/include/objc/objc-sync.h中找到(cmd+shift+p搜索objc-sync.h)

```cpp
/** 
 * Begin synchronizing on 'obj'.  
 * Allocates recursive pthread_mutex associated with 'obj' if needed.
 * 
 * @param obj The object to begin synchronizing on.
 * 
 * @return OBJC_SYNC_SUCCESS once lock is acquired.  
 */
OBJC_EXPORT int
objc_sync_enter(id _Nonnull obj)
    OBJC_AVAILABLE(10.3, 2.0, 9.0, 1.0, 2.0);

/** 
 * End synchronizing on 'obj'. 
 * 
 * @param obj The object to end synchronizing on.
 * 
 * @return OBJC_SYNC_SUCCESS or OBJC_SYNC_NOT_OWNING_THREAD_ERROR
 */
OBJC_EXPORT int
objc_sync_exit(id _Nonnull obj)
    OBJC_AVAILABLE(10.3, 2.0, 9.0, 1.0, 2.0);
```

注释中有提到使用的是pthread_mutex的递归锁, 我尝试着找了一下.m的实现, 但是网上版本太多了, 最后通过注释搜索, 找到了与该版本匹配的实现文件， 下面也贴出了重要的函数实现

[objc4-371 objc-sync.h](https://opensource.apple.com/source/objc4/objc4-371/runtime/objc-sync.h)  
[objc4-371 objc-sync.m](https://opensource.apple.com/source/objc4/objc4-371/runtime/objc-sync.m)  

```cpp
static pthread_mutexattr_t* recursiveAttributes()
{
    // 略略略...
    err = pthread_mutexattr_settype(&sRecursiveLockAttr, PTHREAD_MUTEX_RECURSIVE);
    // 略略略...
    return &sRecursiveLockAttr;
}
// Begin synchronizing on 'obj'.  
// Allocates recursive pthread_mutex associated with 'obj' if needed.
// Returns OBJC_SYNC_SUCCESS once lock is acquired.  
int objc_sync_enter(id obj)
{
    // 略略略...
    result = pthread_mutex_lock(&data->mutex);
    // 略略略...
    return result;
}

// End synchronizing on 'obj'. 
// Returns OBJC_SYNC_SUCCESS or OBJC_SYNC_NOT_OWNING_THREAD_ERROR
int objc_sync_exit(id obj)
{
    // 略略略...
    result = pthread_mutex_unlock(&data->mutex);
    // 略略略...
    return result;
}
```

这下就很明了了, synchronized其实就是通过pthread_mutex_lock创建的递归锁(PTHREAD_MUTEX_RECURSIVE), 值得注意的是swift已经没有这个语法糖了, 不过可以通过尾随闭包的简写形式实现

```cpp
// 实现
func synchronized(lock: AnyObject, closure: () -> ()) {
    objc_sync_enter(lock)
    closure()
    objc_sync_exit(lock)
}
// 使用
synchronized(anObj) {
    // TODO:
}
```

参考链接 [https://swifter.tips/lock/](https://swifter.tips/lock/)  