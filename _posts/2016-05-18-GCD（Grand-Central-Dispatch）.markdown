---
layout: post
title:  "GCD（Grand Central Dispatch）"
date:   2016-05-18 17:40:58 +0800
categories: jekyll update
tags: [iOS]
---

GCD（Grand Central Dispatch），是 Apple 开发的一个多核编程的解决方法。该方法在 Mac OS X 10.6 雪豹中首次推出，并随后被引入到了 iOS4.0 中。GCD 是一个替代诸如NSThread,NSOperationQueue, NSInvocationOperation 等技术的很高效和强大的技术。

GCD 和 block 的配合使用，可以方便地进行多线程编程。

# 优势
1）  苹果官方为多核的并行运算提出的解决方案。

2）  会自动利用更多的CPU内核。

3）  会自动管理线程的生命周期（创建线程、调度任务、销毁线程）。


# 核心概念
1）  任务：执行什么操作。block

2）  队列：用来存放任务。

串行队列：顺序，一个一个执行。一个任务执行完毕后才执行下一个任务。

并发队列：同时，同时执行很多个任务。自动开启多个线程同时执行任务。并发功能只有在异步函数下才生效。


# 使用步骤：
1）  定制任务

确定想要做的事情。

2）  将任务添加到队列中

GCD会自动将队列中的任务取出，放到对应的线程中执行。

任务的取出原则遵循队列的原则：先进先出，后进后出。


# 执行任务的函数
1）同步方式

dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);

queue:队列

block:任务


2）异步方式

dispatch_async(dispatch_queue_t queue, dispatch_block_t block);

queue:队列

block:任务


**同步和异步的区别：**

同步：在当前线程中执行。

异步：在另一条线程中执行。
**同步任务的作用：**

1）  用户登录

2）  下载任务1

3）  下载任务2


# 术语
1）  同步和异步决定了是否要开辟新线程。

同步：在当前线程中执行任务，不具备开启新线程的能力。

异步：在新的线程中执行任务，具备开启新线程的能力。

2）  并发和串行决定了任务执行的方式。

并发：多个任务同时执行。

串行：一个任务执行完毕后，再执行下一个任务。



# 代码使用　
{% highlight objectivec %}
/*串行队列*/
/*创建队列
参数：1.队列标签。 2.队列属性。
*/
dispatch_queue_t queue = dispatch_queue_create(”dengw”,DISPATCH_QUEUE_SERIAL);
/*同步执行任务，不会开辟新线程，在当前线程中顺序执行。
一般只要使用“同步”执行，串行队列对添加的同步任务，立马执行*/
dispatch_sync(queue, ^{
    NSLog(@”%@”, [NSThread currentThread]);
});
/*异步执行任务，开辟新线程，在新线程中执行。开辟新线程的数量与队列模式有关。串行队列中异步执行只会开启一个新线程。*/
for(int I = 0; I < 10; I++){
    dispatch_async(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}
　　
/*并发队列：需要程序员释放。*/
/*创建队列
参数：1.队列标签。 2.队列属性。
*/
dispatch_queue_t queue = dispatch_queue_create(”dengw”,DISPATCH_QUEUE_CONCURRENT);
/*异步执行任务，开辟新线程，在新线程中执行。开辟新线程的数量程序员无法控制。*/
for(int I = 0; I < 10; I++){
    dispatch_async(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}
/*同步执行任务，不开辟新线程，顺序执行*/
for(int I = 0; I < 10; I++){
    dispatch_sync(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}

/*主队列，专门负责在主线程上调度任务。程序启动以后至少有一个主线程，则会创建主队列。*/
/*主队列不允许开辟新线程。不会在子线程调度任务。*/
/*获得主队列*/
dispatch_queue_t queue = dispatch_get_main_queue();
/*异步执行任务，在主队列中，只能顺序执行。*/
for(int I = 0; I < 10; I++){
/*异步：把任务放到主队列中，但不需要马上执行。*/
    dispatch_async(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}
/*同步执行任务*/
for(int I = 0; I < 10; I++){
/*同步：把任务放到主队列中，需要马上执行。*/
/*阻塞*/
    dispatch_sync(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}

/*全局队列：本质是并发队列。
与并发队列的区别：
1）全局队列没有名字，而并发队列有名字。
2）全局队列，是供所有的应用程序使用。
3）在MRC中，全局队列不需要释放，并发队列需要释放。*/
/*获得全局队列
参数：
参数1
iOS7中
DISPATCH_QUEUE_PRIORITY_HEGH    2 高优先级
DISPATCH_QUEUE_PRIORITY_DEFAULT  0 默认优先级
DISPATCH_QUEUE_PRIORITY_LOW   (-2) 低优先级
DISPATCH_QUEUE_PRIORITY_BACKGROUND    INT16_MIN 后台优先级（最低）
iOS8中
DISPATCH_QUEUE_PRIORITY_HEGH:QOS_CLASS_USER_INITIATED
DISPATCH_QUEUE_PRIORITY_DEFAULT:QOS_CLASS_USER_DEFAULT
DISPATCH_QUEUE_PRIORITY_LOW:QOS_CLASS_USER_UTILITY
DISPATCH_QUEUE_PRIORITY_BACKGROUND: QOS_CLASS_USER_BACKGROUND
参数2
保留参数。*/
dispatch_queue_t queue = dispatch_get_global_queue(QOS_CLASS_USER_DEFAULT,0);
/*异步执行任务*/
for(int I = 0; I < 10; I++){
    dispatch_async(queue, ^{
        NSLog(@”%@”, [NSThread currentThread]);
    });
}
{% endhighlight %}

# 各队列的执行效果
 
| | 全局并行队列 | 手动创建串行队列 | 主队列 |
| --- | --- | --- |
| 同步（sync） | 没有开启新线程。串行执行任务。| 没有开启新线程。串行执行任务。 | 会死锁 |
| 异步（async）| 有开启新线程。并行执行任务。| 有开启新线程。串行执行任务。| 没有开启新线程。串行执行任务。|


# 队列的选择
1）串行队列异步执行

开一条线程，顺序执行。

效率不高，执行比较慢，资源占用小，省电。

应用场景：一般3G网络，对性能要求不高。

2）并发队列异步执行

开启多条线程，并发执行。

效率高，执行快，资源消耗大，费电。

应用场景：WIFI网络，或需要快速响应，用户体验要求高，对任务执行顺序没有要求。

3）  同步任务

一般只会在并发队列，需要阻塞后续任务，必须等待同步任务执行完毕，再去执行其他任务。“依赖关系”


# 线程间通信
{% highlight objectivec %}
/*从子线程回到主线程*/
dispatch_async(
    dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    //执行耗时的异步操作…
    dispatch_async(dispatch_get_main_queue(), ^{
    //回到主线程，执行UI刷新操作
    });
});
{% endhighlight %}


# 延时操作

1）方式一，调用NSObject的方法
{% highlight objectivec %}
//2秒后再调用run方法
[self performSelector:@selector(run) withObject:nil afterDelay:2.0];
{% endhighlight %}
2）方式二，使用GCD函数
{% highlight objectivec %}
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)),dispatch_get_main_queue(), ^{    
    //2秒后再异步执行这里的代码
});
{% endhighlight %}


# 调度组（分组）

应用场景：开发的时候，有的时候出现多个网络请求（每一个网络请求时间长短不一），都完成以后统一更新UI或通知用户。
{% highlight objectivec %}
/*实例化一个调度组*/
dispatch_group_t group = dispatch_group_create();
//队列
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
//将任务添加到队列
dispatch_group_async(group, queue, ^{
    NSLog(@”A %@”, [NSThread currentThread]);
});
dispatch_group_async(group, queue, ^{
    NSLog(@”B %@”, [NSThread currentThread]);
});
//获得所有调度组里面的异步任务完成的通知
/*在调度组完成通知里，可以跨队列通信*/
dispatch_group_notifity(group, queue, ^{
    //异步的
    NSLog(@”finished”);
});
{% endhighlight %}


# 一次性执行
常见于单例模型中代码使用。
{% highlight objectivec %}
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    //只执行一次
    NSLog(@”hi”);
});
{% endhighlight %}
