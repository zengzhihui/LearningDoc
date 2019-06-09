# Objectice C 动态性-方法查找

## 例子

```
NSObject+Category.h:

@interface NSObject (category)
- (void)testRuntime;
@end

NSObject+Category.m:

@implementation NSObject (category)
- (void)testRuntime {
    NSLog(@"testRuntime");
}
@end

TestObject.h:

@interface TestClass : NSObject

@end

TestObject.m:

@implementation TestClass

@end

main.m:

int main(int argc, char * argv[]) {
	// 注意这里
	[TestObject testRuntime];
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

```
思考下以上代码能编译通过吗？执行结果是什么？

* 5
* 4
* 3
* 2
* 1

```
2019-06-02 00:21:37.742198+0800 TestSuper[19525:741621] testRuntime
```
编译通过，没有crash，输出"testRuntime" -- 是不是很惊讶？那究竟是什么原因呢？

![继承](https://raw.githubusercontent.com/WiInputMethod/interview/master/img/ios-runtime-class.png)

上图是OC里经典的继承链。左列是类的实例对象，它的isa指针是指向类，即中间列。类里也有isa指针指向类的meta class。注意右上角实线箭头：Root Class的meta class是Root class。也就是说上面代码中TestObject调用testRuntime类方法，先在本类的类方法查到IMP，没找到转向父类NSObject类方法里查找。由于NSObject的类实例（meta class）就是NSObject，因此找到分类里的testRuntime。

>
Objective C方法调用本质发送消息。例子中的[TestObject testRuntime]的调用是通过TestObject里的isa指针发送消息。TestObject的isa指针指向TestObject的meta class。TestObject的meta class收到消息后查找过程如下：在自己的cache method list里查找testRuntime的IMP（函数指针，指向方法实现的首地址）。若找到即可直接调用，若没有找到则在method list里查找。若还没找到通过superclass指针找到父类，在父类的method list查，直到找到为止。若没有找到，则进行消息动态处理和消息转发。

```
查找源代码

/***********************************************************************
* lookUpMethod.
* The standard method lookup. 
* initialize==NO tries to avoid +initialize (but sometimes fails)
* cache==NO skips optimistic unlocked lookup (but uses cache elsewhere)
* Most callers should use initialize==YES and cache==YES.
* May return _objc_msgForward_internal. IMPs destined for external use 
*   must be converted to _objc_msgForward or _objc_msgForward_stret.
**********************************************************************/
__private_extern__ IMP lookUpMethod(Class cls, SEL sel, 
                                    BOOL initialize, BOOL cache)
{
    Class curClass;
    IMP methodPC = NULL;
    Method meth;
    BOOL triedResolver = NO;

    // Optimistic cache lookup
    if (cache) {
        methodPC = _cache_getImp(cls, sel);
        if (methodPC) return methodPC;    
    }

    // realize, +initialize, and any special early exit
    methodPC = prepareForMethodLookup(cls, sel, initialize);
    if (methodPC) return methodPC;


    // The lock is held to make method-lookup + cache-fill atomic 
    // with respect to method addition. Otherwise, a category could 
    // be added but ignored indefinitely because the cache was re-filled 
    // with the old value after the cache flush on behalf of the category.
 retry:
    lockForMethodLookup();

    // Try this class's cache.

    methodPC = _cache_getImp(cls, sel);
    if (methodPC) goto done;

    // Try this class's method lists.

    meth = _class_getMethodNoSuper_nolock(cls, sel);
    if (meth) {
        log_and_fill_cache(cls, cls, meth, sel);
        methodPC = method_getImplementation(meth);
        goto done;
    }

    // Try superclass caches and method lists.

    curClass = cls;
    while ((curClass = _class_getSuperclass(curClass))) {
        // Superclass cache.
        meth = _cache_getMethod(curClass, sel, &_objc_msgForward_internal);
        if (meth) {
            if (meth != (Method)1) {
                // Found the method in a superclass. Cache it in this class.
                log_and_fill_cache(cls, curClass, meth, sel);
                methodPC = method_getImplementation(meth);
                goto done;
            }
            else {
                // Found a forward:: entry in a superclass.
                // Stop searching, but don't cache yet; call method 
                // resolver for this class first.
                break;
            }
        }

        // Superclass method list.
        meth = _class_getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            log_and_fill_cache(cls, curClass, meth, sel);
            methodPC = method_getImplementation(meth);
            goto done;
        }
    }

    // No implementation found. Try method resolver once.

    if (!triedResolver) {
        unlockForMethodLookup();
        _class_resolveMethod(cls, sel);
        // Don't cache the result; we don't hold the lock so it may have 
        // changed already. Re-do the search from scratch instead.
        triedResolver = YES;
        goto retry;
    }

    // No implementation found, and method resolver didn't help. 
    // Use forwarding.

    _cache_addForwardEntry(cls, sel);
    methodPC = &_objc_msgForward_internal;

 done:
    unlockForMethodLookup();

    // paranoia: look for ignored selectors with non-ignored implementations
    assert(!(sel == (SEL)kIgnore  &&  methodPC != (IMP)&_objc_ignored_method));

    return methodPC;
}

```

## 消息动态处理
* resolveInstanceMethod
* forwardingTargetForSelector

## 消息转发
* methodSignatureForSelector
* forwardInvocation