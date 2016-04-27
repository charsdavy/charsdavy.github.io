# Objective-C中的alloc和init问题

从开始学的`NSString *name=[[NSString alloc] init] `起，仅知道这句话是分配内存空间，一直在用，从来没考虑过它的内部是怎么实现的。今天无意中看到了这一句代码：
```
NSString *name = [NSString alloc];
NSLog(@"%p",name);
name = [name init];
NSLog(@"%p",name);
```
试着打印了一下，发现两个的内存地址不一样：
![](http://upload-images.jianshu.io/upload_images/1492739-3b68beae6d8cc9fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

alloc是开辟一个内存空间，init是初始化，为什么初始化不在原有的内存空间上初始化，而是重新开辟一个内存空间。于是开始查资料，这时又发现了一个新的迷惑：
```
NSObject *obj = [NSObject alloc];
NSLog(@"%p",obj);
obj = [obj init];
NSLog(@"%p",obj);
```
打印结果：
![](http://upload-images.jianshu.io/upload_images/1492739-799b0c9493a36b0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
怎么地址又变一样了？再打印NSArray的试一试：
```
NSArray *person = [NSArray alloc];
NSLog(@"%p",person);
person = [person init];
NSLog(@"%p",person);
```
再次打印结果：
![](http://upload-images.jianshu.io/upload_images/1492739-7eacb95d56d56b8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
仍然是不一样的。原因是什么呢？首先看看NSStrng的init方法吧：
```
-(id)init{ 
  if(self = [super init]) {
    // 重新赋值 
    //… 
  }
}
```
从代码中可以分析，`self=[super init]`如果不为nil，就重新分配内存空间，这就解释了为什么 NSString，NSArray的调用alloc]init]方法后，内存地址会不一样，但是NSObject为什么会一样呢，我们知道NSObject是一切类的基类，当`[[NSString alloc]init]`执行时，调用的`[super init]`就是 NSObject中的init方法，既然NSObject身为基类，它也就无法调用super init，所以当NSObject执行`[[NSObject alloc]init]`时，也就没有了init重新分配空间这一环节。
至于苹果公司为什么初始化一个实例要分两步，个人认为是方便构造后初始化不同的方法，如果用 new关键字，只能调用一个init，而不能调用initWithName等方法。

**知识拓展：**
NSString alloc之后，没有init，那么这部分alloc后的内存空间可不可以用？答案是显而易见的，如果可以用，苹果公司也就没必要提供一个init方法，那么alloc后的指针称为什么呢？ ——悬挂指针。
如果一个地方指针既不为空，也没有被设置为指向一个已知的对象，则这样的指针称为**悬挂指针**。在程序里面是很危险的事。当程序运行使用该指针时，程序不能判断指针的合法性，将会产生很严重的错误。