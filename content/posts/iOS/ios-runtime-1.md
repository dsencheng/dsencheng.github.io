---
title: "ios Runtime 拾遗一"
date: 2025-01-06T14:22:16+08:00
draft: false
tags: [ios, runtime]
categories: [ios]
---

作为iOS开发，一直都知道消息机制是Runtime实现的。
今天从以下几个场景去探索一下，调用一个方法都经历了什么。
文章基于源码 objc4-906(https://github.com/MrOwlSage/objc4-906)。

首先创建一个类`Cat`，在`main.m`中，实例化一个对象，然后按照下面的几个场景，调用相关方法，来梳理下代码逻辑。

1. `Cat.h` 和 `Cat.m`中，添加`-(void)eat`方法;
2. 仅在`Cat.h`中，添加`-(void)eat`方法;
3. 仅在`Cat.m`中，添加`-(void)eat`方法;
4. 去掉所有的`-(void)eat`方法，对`Cat`类添加一个Category `meow`方法;
5. 把`-(void)eat`方法，替换为`+(void)eat`方法,重新探索上面的几个场景;


#### 场景一: `Cat.h` 和 `Cat.m`中，添加`-(void)eat`方法

代码如下

```objc
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Cat *cat = [[Cat alloc] init];
        [cat eat];
    return 0;
}
```

在 `[cat eat]` 这行加上断点，然后找到Xcode的菜单，Debug > Debug Workflow > Always Show Disassembly，勾选好。

然后运行程序，可以看到类似的汇编代码：

```objc
0x100003ec4 <+52>:  add    x8, x8, #0x98             ; (void *)0x00000001000080c8: Cat
0x100003ec8 <+56>:  ldr    x0, [x8]
0x100003ecc <+60>:  bl     0x100003f10               ; symbol stub for: objc_alloc_init
0x100003ed0 <+64>:  str    x0, [sp, #0x8]
0x100003ed4 <+68>:  ldr    x0, [sp, #0x8]
0x100003ed8 <+72>:  bl     0x100003f40               ; objc_msgSend$eat
```

可以看到，`[cat eat]` 被编译成了 `objc_msgSend$eat`。
`objc_msgSend`这个函数在哪呢？
在main函数上导入`#import <objc/message.h>`，然后点进这个文件，搜索一下就可以看到这个函数的声明，但看不到函数的实现。
```
objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```
第一个参数是对象，第二个参数是方法名，后面是可变参数。
可以理解成 `sendMsg(cat, @selector(eat))`， 对实例化对象发送一个方法消息。
`objc_msgSend`因为看不到函数的实现代码，找了一份伪代码，帮助理解这个函数。**伪代码并不全，只是大概的流程**。

```
void objc_msgSend(id self, SEL _cmd, ...) {
    // 获取类的 isa 指针
    // 在isa的缓存中查找方法
    // 缓存没有命中，调用C代码
    // 命中跳转到方法实现
}
```

简单来说，就是去cat类缓存中找eat方法，找到了就执行，找不到就执行C代码。
第一次运行代码，缓存中肯定是没有的，那么是执行到了哪里呢？
继续断点调试，这是需要按住`control`键，然后点击`step into`，直到看到汇编代码： 

`0x100612c3c <+60>:  cbz    x9, 0x100612ea0           ; _objc_msgSend_uncached`

从字母意思上可以猜到这是执行没有缓存的函数。
继续`step into`，会看到汇编代码

`0x100612edc <+60>:  bl     0x1005aef54               ; lookUpImpOrForward at objc-runtime-new.mm:7324`

这行代码很明显了，在`objc-runtime-new.mm`文件中，有个`lookUpImpOrForward`函数，找到这个方法，在方法入口处加上断点。

继续执行下去，发现断点没有触发。不用担心，打开Xcode的菜单，Debug > Debug Workflow > Always Show Disassembly，去掉勾选。

再次重新运行，会发现马上触发`lookUpImpOrForward`上的断点。由于`lookUpImpOrForward`调用太多，暂时关闭断点。当在main函数中的`[cat eat];`触发断点时，再打开`lookUpImpOrForward`的断点。

应该可以看到如下图所示
![截图](/images/iShot_2025-01-06_19.58.21.png "图一")

那么，`lookUpImpOrForward`函数都做了什么呢？
伪代码如下, 补充了部分关键代码
```
for循环 {
    1. 检查当前类的方法缓存，命中执行goto
      cache_getImp()
    2.去当前类的method_list中查找方法，命中执行goto
      findMethodInUnsortedMethodList() -- 比对方法名
      log_and_fill_cache() -- 将方法缓存到当前类的缓存中
    3.如果当前类没有方法，开始去父类找，一直找不到，进入消息转发流程
      imp = forward_imp; break;
      resolveMethod_locked() --消息转发流程
}

```

在当前场景一，代码会执行到第二步，基本上就结束了。如果多次调用`[cat eat]`，因为当前类的cache_t已经缓存了`eat`方法，所以不会在执行`lookUpImpOrForward`函数。

#### 场景二: 仅在`Cat.h`中，添加`-(void)eat`方法 

在该场景下，只是声明了方法，没有实现。通过断点调试，发现`lookUpImpOrForward`函数会执行到第三步，进入消息转发流程。也就是会执行`resolveMethod_locked`函数。

`resolveMethod_locked`的主要作用就是消息转发流程的分发。
区分cls是不是元类，不是就进入实例化的方法转发流程`resolveInstanceMethod()`，是进入类的方法转发流程`resolveClassMethod()`。
上面2个方法的过程中把方法放入到缓存中，最后在调用`lookUpImpOrForwardTryCache`，尝试执行缓存中的sel。

当前是对实例化的对象，所以会执行`resolveInstanceMethod()`，简单写下执行逻辑。可以说是消息转发流程的全逻辑。

1. 检查当前类是否实现了`resolveInstanceMethod:`方法，没有实现，返回NO。 
  这里面执行断点会发现，代码又去执行了`lookUpImpOrForward`函数，Cat类没有实现`resolveInstanceMethod:`方法，会去父类NSObject找，NSObject的默认实现方法返回值为NO。没有任何处理。
1. 找到`resolveInstanceMethod:`响应方法后，向Cat类发 objc_msgSend，因Cat类没有实现方法，返回值为false。
2. 代码继续执行后发现，又到了`lookUpImpOrForward`，向Cat类找eat方法。找不到又去向父类找，直到NSObject也找不到。
   此时在`log_and_fill_cache()`插入缓存，**注意eat的imp**是`_objc_msgForward_impcache`，也就是说，此时eat方法的imp已经指向了消息转发流程。
3. 代码现在执行到了`lookUpImpOrNilTryCache`，方法已经放到了缓存里，查找出来的imp就是`_objc_msgForward_impcache`,断点单步走会执行到汇编文件.图二：
    可以看到，`_objc_msgForward_impcache`里执行了`objc_msgForward`，也就是消息转发流程。
    ![截图](/images/iShot_2025-01-07_11.54.19.png "图二")
4. 代码继续执行后，再次进入`lookUpImpOrForward`，但这次的参数sel已经是`forwardingTargetForSelector:`，图三；
   Cat类当然找不到`forwardingTargetForSelector:`方法，所以会向父类找，直到NSObject找到默认实现，返回为nil。
    ![截图](/images/iShot_2025-01-07_12.04.14.png "图三")
5. 代码继续执行，再次进入`lookUpImpOrForward`，参数sel是`"methodSignatureForSelector:`，最终直到NSObject的方法返回nil，图四。
   ![截图](/images/iShot_2025-01-07_12.10.28.png "图四")
6. 代码继续执行，此时又来到`lookUpImpOrForward`，cls是Cat，sel是eat，再次向Cat找eat方法。
   注意behavior参数是4，也就是`LOOKUP_NIL`，说明找不到了，imp还是 `_objc_msgForward_impcache`，不需要在循环继续找了，返回nil
7. 代码又又又又到了`lookUpImpOrForward`，但这次的参数sel是`doesNotRecognizeSelector:`，图五。
   Cat类当然找不到`doesNotRecognizeSelector:`方法，所以会向父类找，直到NSObject找到默认实现，抛出异常,经典的`unrecognized selector sent to instance `。
   ![截图](/images/iShot_2025-01-07_12.49.17.png "图五")

    NSObject的`doesNotRecognizeSelector:`方法实现：
   ```
   - (void)doesNotRecognizeSelector:(SEL)sel {
    _objc_fatal("-[%s %s]: unrecognized selector sent to instance %p", 
                object_getClassName(self), sel_getName(sel), self);
    }
   ```

#### 场景三：仅在`Cat.m`中，添加`-(void)eat`方法

h文件中没有声明eat方法，代码`[cat eat]`会编译报错。所以直接使用`objc_msgSend`的方式去调用。记得导入头文件`#import <objc/message.h>`.
代码如下

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Cat *cat = [[Cat alloc] init];
//        [cat eat];
        
        SEL sel = @selector(eat);
        void (*sendMsg)(id, SEL) = (void (*)(id, SEL))objc_msgSend;
        sendMsg(cat, sel);
    }
    return 0;
}
```

这个场景没有复杂的逻辑，因为Cat类有eat的方法实现，所以在`lookUpImpOrForward`查找方法过程中，直接找到了eat的imp，就不在赘述了。

#### 场景四：去掉所有的`-(void)eat`方法，对`Cat`类添加一个Category `meow`方法

在该场景下，方法调用的流程和场景一、场景二，基本是一样的。

分类中的方法，广泛的结论是在Runtime期间添加进去的，调用顺序是 `load_images` - `loadAllCategoriesIfNeeded()` - `load_categories_nolock`。
这是一个遍历header_info的过程，我在`load_categories_nolock`方法中加入打印，如图六。

![截图](/images/iShot_2025-01-07_20.01.25.png "图六")

但是，运行查看控制台的输出，并没有看到Cat的分类。那就奇怪了，分类的方法是在什么时候加载的呢？(如果分类重写了+load方法，这里就会打印出来)
用了一个笨办法，在文件中搜锁category，查看哪个地方感觉调用了加载分类的方法。
无意间发现了`attachCategories`，查看方法代码，看到了`if (slowpath(PrintConnecting))`，这好像是个控制台输出的开关。
点进`PrintConnecting`，看到
`OPTION( PrintConnecting,                           Off, OBJC_PRINT_CLASS_SETUP,          "log progress of class and category setup")`
这个是不是Scheme中的环境变量呢？而且这个文件名是`objc-env.h`，然后在Scheme中，添加了`OBJC_PRINT_CLASS_SETUP=1`，重新运行，控制台输出如下：

```
......
objc[99635]: METHOD -[__NSCFConstantString retainCount]
objc[99635]: METHOD -[__NSCFConstantString isNSCFConstantString__]
objc[99635]: METHOD -[__NSCFConstantString redactedDescription]
Hello, World!
objc[99635]: CLASS: realizing class 'Cat' (meta) 0x1000080a8 0x100008000 #0 
objc[99635]: CLASS: methodizing class 'Cat' (meta)
objc[99635]: CLASS: realizing class 'Cat' 0x1000080d0 0x100008048 #0 
objc[99635]: CLASS: methodizing class 'Cat' 
objc[99635]: METHOD -[Cat meow]
Program ended with exit code: 0
```

`meow`就是在分类中的方法。经过测试，如果不实例化一个`Cat`对象，分类中的方法是不会打印。
那么，`Class cls = objc_getClass("Cat");` 这样的方式会打印分类中的方法吗？答案是会的。
追踪`objc_getClass`，一路执行到`realizeClassMaybeSwiftMaybeRelock` - `realizeClassWithoutSwift`, 
这里面大概是把Cat类组装了起来，控制台输出如下：

```
objc[2368]: CLASS: realizing class 'Cat' 0x1000080c8 0x100008048 #0 
objc[2368]: CLASS: realizing class 'Cat' (meta) 0x1000080a0 0x100008000 #0 
objc[2368]: CLASS: methodizing class 'Cat' (meta)
```

代码执行到最后，`methodizeClass`代码注释是 Attach categories， 那这里面是不是添加了分类的方法呢？
继续单步执行，经过几次尝试，感觉meow方法已经被加载了，代码最后，控制台输出的方法里就有meow。
在代码开始，有段代码，ro也就是class_ro_t结构体，这里面存了类的方法，属性，协议等。
```
auto rw = cls->data();
auto ro = rw->ro();
```
然后加了些代码，看看ro里存了什么方法。如图七。

![图七](/images/iShot_2025-01-08_12.50.38.png "图七")

可以看到，控制台输出了`meow`方法，说明分类的方法已经加载了。
class_ro_t是类的只读信息，特点是在编译阶段生成的。
所以，当前场景下，分类的方法是编译阶段已经加载了。
更多可以参考文章 [类与分类加载](https://juejin.cn/post/7207769757571596346#heading-18)


#### 场景五：把`-(void)eat`方法，替换为`+(void)eat`方法

类方法与实例化方法有些不同，会判断该类是否初始化过。
调用顺序，在`lookUpImpOrForward` > `realizeAndInitializeIfNeeded_locked` > `realizeClassMaybeSwiftAndLeaveLocked` > `realizeClassWithoutSwift`
在类方法`+(void)eat`调用前，调用过`[[Cat alloc] init]`，就不会再执行上面的流程了。
经过初始化流程，重写`class_rw_t`，分类也加载了。
然后进入`initializeAndLeaveLocked`方法，调用类的`initialize`方法。
现在Cat类初始化完成，此时`+(void)eat`方法已经加载。
调用`getMethodNoSuper_nolock`时可以直接查到`+(void)eat`方法的IMP。
该场景下，与之前的最大区别就是，执行了类的初始化过程。

#### 结尾
本地探索就此结束，或许是对源码了解不够，没有探索到更多内容。
欢迎大家交流，共同进步。