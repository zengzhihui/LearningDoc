# Objectice C 动态性

## 看个例子

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

OC里经典的继承链，注意右上角实线箭头：Root Class的meta class是Root class。也就是说上面代码中TestObject调用testRuntime方法，先在本类的类方法查到IMP，没找到转向父类NSObject类方法里查找。由于NSObject的类实例就是NSObject，因此找到分类里的testRuntime。



