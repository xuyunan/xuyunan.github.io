---
title: iOS中的锁(一)--pthread_mutex_lock
tags: [iOS]
date: 2019-06-10T13:00:24+08:00
---

底层pthread级别加锁, 使用c语言的api, 可用作互斥锁、递归锁, 使用api前，需要导入头文件

```objectivec
#import <pthread.h>
```

<!--truncate-->

互斥锁

```cpp
static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_lock(&mutex);
// TODO:
pthread_mutex_unlock(&mutex);
```

递归锁，允许同一个线程多次递归访问被锁资源，加锁和解锁必须成对出现，使得最终锁能够被解开，其它线程能够访问资源.

```cpp
pthread_mutexattr_t    attr;
pthread_mutex_t mutex;
    
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
pthread_mutex_init(&mutex, &attr);

// 下面一段有可能被通一个线程，在一个或多个方法中多次调用
pthread_mutex_lock(&mutex);
// TODO:
pthread_mutex_unlock(&mutex);
```

类似PTHREAD_MUTEX_RECURSIVE影响锁行为的总共有三种类型

```cpp
// 如果相同的线程不尝试先解锁，而再次加锁互斥对象，会造成死锁
#define PTHREAD_MUTEX_NORMAL		0     
// 如果相同的线程试图不先解锁互斥对象而再一次锁定同一个互斥对象，会返回一个非零值的错误，避免死锁
#define PTHREAD_MUTEX_ERRORCHECK	1      
// 递归地使用pthread_mutex_lock加锁互斥对象，不会造成死锁
#define PTHREAD_MUTEX_RECURSIVE		2
#define PTHREAD_MUTEX_DEFAULT		PTHREAD_MUTEX_NORMAL
```
