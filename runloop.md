### RunLoop

一般来讲， 一个线程只能处理一个任务，处理完任务之后就会退出。但是我们需要一种技术就是想让线程时时去处理任务而不退出。所以在计算机科学里就提出了一种方案[Event Loop](https://en.wikipedia.org/wiki/Event_loop),比如windows上与用户交互的进程必须响应外界消息，所以在这个进程中要有[消息循环](https://en.wikipedia.org/wiki/Message_loop_in_Microsoft_Windows)来完成。

所以在iOS中就是使用RunLoop来进行管理消息和事件的对象，并且提供了一个入口函数来实现上面Event Loop逻辑，当线程执行到这个函数时，就会停留在这个函数内部一直处理事件和消息直到用户终止或者其他异常造成终止。

RunLoop结构

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)

线程和RunLoop是一一对应的关系，线程刚创建时并没有RunLoop你需要去主动获取。在你第一次获取时会创建RunLoop, 在线程销毁之后进行销毁。

#### RunLoop的组成

```swift

<CFRunLoop 0x6040001ef600 [0x10c023960]>{wakeup port = 0x1c03, stopped = false, ignoreWakeUps = false, 
current mode = kCFRunLoopDefaultMode,
common modes = <CFBasicHash 0x60400005ef60 [0x10c023960]>{type = mutable set, count = 2,
entries =>
	0 : <CFString 0x109df3060 [0x10c023960]>{contents = "UITrackingRunLoopMode"}
	2 : <CFString 0x10bff9790 [0x10c023960]>{contents = "kCFRunLoopDefaultMode"}
}
,
common mode items = <CFBasicHash 0x60800005f020 [0x10c023960]>{type = mutable set, count = 13,
entries =>
	0 : <CFRunLoopSource 0x60c00016cf00 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10fc856c6)}}
	1 : <CFRunLoopSource 0x60800016d680 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 1, info = 0x2d03, callout = PurpleEventCallback (0x10fc87bef)}}
	7 : <CFRunLoopSource 0x60c00016d680 [0x10c023960]>{signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>{version = 0, info = 0x60c0000bcf80, callout = FBSSerialQueueRunLoopSourceHandler (0x10f38c9fb)}}
	8 : <CFRunLoopObserver 0x608000125280 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x108c60b93), context = <CFRunLoopObserver context 0x7ff9614018e0>}
	9 : <CFRunLoopObserver 0x6080001251e0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x108c60b18), context = <CFRunLoopObserver context 0x7ff9614018e0>}
	10 : <CFRunLoopSource 0x60800016d380 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x604000153e50, callout = __handleEventQueue (0x10958f7e8)}}
	11 : <CFRunLoopObserver 0x60c000125640 [0x10c023960]>{valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x1092177a1), context = <CFRunLoopObserver context 0x60c0000c6eb0>}
	12 : <CFRunLoopObserver 0x6080001250a0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (
	0 : <0x7ff9620042e8>
)}}
	13 : <CFRunLoopObserver 0x608000125140 [0x10c023960]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (
	0 : <0x7ff9620042e8>
)}}
	14 : <CFRunLoopSource 0x60800016d2c0 [0x10c023960]>{signalled = Yes, valid = Yes, order = -2, context = <CFRunLoopSource context>{version = 0, info = 0x60800005ef60, callout = __handleHIDEventFetcherDrain (0x10958f7f4)}}
	15 : <CFRunLoopObserver 0x608000125320 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x11037b404), context = <CFRunLoopObserver context 0x0>}
	16 : <CFRunLoopSource 0x60400016d800 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x604000153da0 [0x10c023960]>{valid = Yes, port = 1a07, source = 0x60400016d800, callout = _ZL27change_notify_port_callbackP12__CFMachPortPvlS1_ (0x1102ce174), context = <CFMachPort context 0x0>}}
	21 : <CFRunLoopSource 0x60c00016cfc0 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server> {port = 15891, subsystem = 0x109daa9a8, context = 0x0}}
}
,
modes = <CFBasicHash 0x60400005ef90 [0x10c023960]>{type = mutable set, count = 4,
entries =>
	2 : <CFRunLoopMode 0x608000183740 [0x10c023960]>{name = UITrackingRunLoopMode, port set = 0x2003, queue = 0x6080000f8180, source = 0x60800011b630 (not fired), timer port = 0x2203, 
	sources0 = <CFBasicHash 0x60800005f050 [0x10c023960]>{type = mutable set, count = 4,
entries =>
	0 : <CFRunLoopSource 0x60c00016cf00 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10fc856c6)}}
	1 : <CFRunLoopSource 0x60800016d380 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x604000153e50, callout = __handleEventQueue (0x10958f7e8)}}
	2 : <CFRunLoopSource 0x60800016d2c0 [0x10c023960]>{signalled = Yes, valid = Yes, order = -2, context = <CFRunLoopSource context>{version = 0, info = 0x60800005ef60, callout = __handleHIDEventFetcherDrain (0x10958f7f4)}}
	6 : <CFRunLoopSource 0x60c00016d680 [0x10c023960]>{signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>{version = 0, info = 0x60c0000bcf80, callout = FBSSerialQueueRunLoopSourceHandler (0x10f38c9fb)}}
}
,
	sources1 = <CFBasicHash 0x60800005efc0 [0x10c023960]>{type = mutable set, count = 3,
entries =>
	0 : <CFRunLoopSource 0x60400016d800 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x604000153da0 [0x10c023960]>{valid = Yes, port = 1a07, source = 0x60400016d800, callout = _ZL27change_notify_port_callbackP12__CFMachPortPvlS1_ (0x1102ce174), context = <CFMachPort context 0x0>}}
	1 : <CFRunLoopSource 0x60800016d680 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 1, info = 0x2d03, callout = PurpleEventCallback (0x10fc87bef)}}
	2 : <CFRunLoopSource 0x60c00016cfc0 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server> {port = 15891, subsystem = 0x109daa9a8, context = 0x0}}
}
,
	observers = (
    "<CFRunLoopObserver 0x608000125140 [0x10c023960]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (\n\t0 : <0x7ff9620042e8>\n)}}",
    "<CFRunLoopObserver 0x60c000125640 [0x10c023960]>{valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x1092177a1), context = <CFRunLoopObserver context 0x60c0000c6eb0>}",
    "<CFRunLoopObserver 0x6080001251e0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x108c60b18), context = <CFRunLoopObserver context 0x7ff9614018e0>}",
    "<CFRunLoopObserver 0x608000125320 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x11037b404), context = <CFRunLoopObserver context 0x0>}",
    "<CFRunLoopObserver 0x608000125280 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x108c60b93), context = <CFRunLoopObserver context 0x7ff9614018e0>}",
    "<CFRunLoopObserver 0x6080001250a0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (\n\t0 : <0x7ff9620042e8>\n)}}"
),
	timers = (null),
	currently 534240076 (1660662897071797) / soft deadline in: 1.84450834e+10 sec (@ -1) / hard deadline in: 1.84450834e+10 sec (@ -1)
},

	3 : <CFRunLoopMode 0x60c000183740 [0x10c023960]>{name = GSEventReceiveRunLoopMode, port set = 0x2503, queue = 0x60c0000f7b80, source = 0x60c00011b510 (not fired), timer port = 0x2b03, 
	sources0 = <CFBasicHash 0x60c00005e660 [0x10c023960]>{type = mutable set, count = 1,
entries =>
	0 : <CFRunLoopSource 0x60c00016cf00 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10fc856c6)}}
}
,
	sources1 = <CFBasicHash 0x60c00005e5d0 [0x10c023960]>{type = mutable set, count = 1,
entries =>
	0 : <CFRunLoopSource 0x60800016d740 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 1, info = 0x2d03, callout = PurpleEventCallback (0x10fc87bef)}}
}
,
	observers = (null),
	timers = (null),
	currently 534240076 (1660662898181775) / soft deadline in: 1.84450834e+10 sec (@ -1) / hard deadline in: 1.84450834e+10 sec (@ -1)
},

	4 : <CFRunLoopMode 0x6040001835a0 [0x10c023960]>{name = kCFRunLoopDefaultMode, port set = 0x1d03, queue = 0x6040000f8100, source = 0x60400011bc60 (not fired), timer port = 0x1f03, 
	sources0 = <CFBasicHash 0x60800005eff0 [0x10c023960]>{type = mutable set, count = 4,
entries =>
	0 : <CFRunLoopSource 0x60c00016cf00 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10fc856c6)}}
	1 : <CFRunLoopSource 0x60800016d380 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 0, info = 0x604000153e50, callout = __handleEventQueue (0x10958f7e8)}}
	2 : <CFRunLoopSource 0x60800016d2c0 [0x10c023960]>{signalled = Yes, valid = Yes, order = -2, context = <CFRunLoopSource context>{version = 0, info = 0x60800005ef60, callout = __handleHIDEventFetcherDrain (0x10958f7f4)}}
	6 : <CFRunLoopSource 0x60c00016d680 [0x10c023960]>{signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>{version = 0, info = 0x60c0000bcf80, callout = FBSSerialQueueRunLoopSourceHandler (0x10f38c9fb)}}
}
,
	sources1 = <CFBasicHash 0x60800005f080 [0x10c023960]>{type = mutable set, count = 3,
entries =>
	0 : <CFRunLoopSource 0x60400016d800 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x604000153da0 [0x10c023960]>{valid = Yes, port = 1a07, source = 0x60400016d800, callout = _ZL27change_notify_port_callbackP12__CFMachPortPvlS1_ (0x1102ce174), context = <CFMachPort context 0x0>}}
	1 : <CFRunLoopSource 0x60800016d680 [0x10c023960]>{signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>{version = 1, info = 0x2d03, callout = PurpleEventCallback (0x10fc87bef)}}
	2 : <CFRunLoopSource 0x60c00016cfc0 [0x10c023960]>{signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server> {port = 15891, subsystem = 0x109daa9a8, context = 0x0}}
}
,
	observers = (
    "<CFRunLoopObserver 0x608000125140 [0x10c023960]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (\n\t0 : <0x7ff9620042e8>\n)}}",
    "<CFRunLoopObserver 0x60c000125640 [0x10c023960]>{valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x1092177a1), context = <CFRunLoopObserver context 0x60c0000c6eb0>}",
    "<CFRunLoopObserver 0x6080001251e0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x108c60b18), context = <CFRunLoopObserver context 0x7ff9614018e0>}",
    "<CFRunLoopObserver 0x608000125320 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x11037b404), context = <CFRunLoopObserver context 0x0>}",
    "<CFRunLoopObserver 0x608000125280 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x108c60b93), context = <CFRunLoopObserver context 0x7ff9614018e0>}",
    "<CFRunLoopObserver 0x6080001250a0 [0x10c023960]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x108c3028a), context = <CFArray 0x60800005f4d0 [0x10c023960]>{type = mutable-small, count = 1, values = (\n\t0 : <0x7ff9620042e8>\n)}}"
),
	timers = <CFArray 0x6000000bd280 [0x10c023960]>{type = mutable-small, count = 1, values = (
	0 : <CFRunLoopTimer 0x60000016d980 [0x10c023960]>{valid = Yes, firing = No, interval = 0, tolerance = 0, next fire date = 534240077 (1.14089197 @ 1660664041943506), callout = (Delayed Perform) UIApplication _accessibilitySetUpQuickSpeak (0x10ae47470 / 0x109122bf1) (/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/System/Library/Frameworks/UIKit.framework/UIKit), context = <CFRunLoopTimer context 0x6000000714c0>}
)},
	currently 534240076 (1660662898213181) / soft deadline in: 1.14373031 sec (@ 1660664041943506) / hard deadline in: 1.14373027 sec (@ 1660664041943506)
},

	5 : <CFRunLoopMode 0x6000001838e0 [0x10c023960]>{name = kCFRunLoopCommonModes, port set = 0x5407, queue = 0x6000000f9300, source = 0x60000011c440 (not fired), timer port = 0x5507, 
	sources0 = (null),
	sources1 = (null),
	observers = (null),
	timers = (null),
	currently 534240076 (1660662901113154) / soft deadline in: 1.84450834e+10 sec (@ -1) / hard deadline in: 1.84450834e+10 sec (@ -1)
},

}
}

```

下面我们整理一下代码，可以更清楚的了解RunLoop的构成

``` swift
<CFRunLoop>{
current mode: kCFRunLoopDefaultMode, //当前模式
common modes: set //一个集合元素有 UITrackingRunLoopMode,kCFRunLoopDefaultMode
common mode items: set //一个集合元素有source,observer,
modes : set //里面放的都是CFRunLoopMode
}

```

结合上两段代码我们可以大概画出RunLoop的结构图，图片来源于一位[博客](https://blog.ibireme.com/2015/05/18/runloop/) 

![](https://blog.ibireme.com/wp-content/uploads/2015/05/RunLoop_0.png)

我们通过上图和上面的代码可以清楚的知道每个RunLoop里都会有多个Mode，而每个Mode又是输入源，定时器集合，观察员集合组成的。每次运行循环时，都可以（显式或隐式）使用特定模式运行。每次调用RunLoop只能指定一种模式，就是current mode,每次切换完mode后run loop都要退出，在重新指定一个新的mode进入。

#### 经典例子

ScrollView中的timer不生效？

``` swift
RunLoop.current.add(timer, forMode: RunLoopMode.defaultRunLoopMode)
```

当你这样添加一个timer到runloop中去时，在滚动ScrollView时timer不会生效。


主线程(main thread) 的run loop 有两个默认的Mode,分别是 default mode 和 tracking mode. default mode是app平时所处的模式， tracking mode 是与用户交互时的模式，比如滚动scrollview.所以当你滚动scrollview时run loop 已经切换了模式，所以这个模式下的timer，sources, observers都不会在运行。我们该如何解决这个问题？

1. 我们可以重新创建一个线程，把它加入到一个新的run loop里，这样就不会和主线程里的模式有冲突了。

```swift
    //生成一个timer
    let timer = Timer(fire: Date(), interval: 1.0, repeats: true, block: { (_) in
    })

    let timerThread = Thread {
      RunLoop.current.add(timer, forMode:RunLoopMode.defaultRunLoopMode)
    }
    
    //这里就是我们一开始所说的，一个线程对应一个run loop，线程销毁， run loop也被销毁。 
```

2. 修改timer的模式

```swift
//让timer运行在多个模式下，这样在模式切换后也能继续工作

RunLoop.current.add(timer, forMode: RunLoopMode.commonModes)

// 我们在RunLoop的构成中会发现common mode，主线程下common modes包括 default mode 和 tracing mode.
```

#### AutoreleasePool



这里有一个很好的实例，去理解进程和线程。
[进程和线程](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)