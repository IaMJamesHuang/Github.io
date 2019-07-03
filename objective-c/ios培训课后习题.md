# class 1 : Objective-c课后习题

## 为什么NSString 、 NSArray、 NSDictionary的属性要用copy，集合的深浅拷贝是怎样的

Q1:

1、首先copy修饰的属性在赋值的时候会对入参进行拷贝后赋值，大致生成如下代码：
```
- (void) setObject(NSObject *obj) {
    [obj retain];
    self.mObj = [obj copy];
    [obj release];
}
```

2、对于NSString、NSArray等类型，调用方可能传入NSMutableString等可变子类，如果不采用copy的方式赋值而是只保存其引用的话，就可能会出现属性中途被外部篡改的情况

Q2：

1、对于不可变集合类，其copy方法拷贝的是其引用，而mutableCopy方法拷贝的则是进行了内容的拷贝

2、对于可变集合类，其copy和mutableCopy方法都是进行内容的拷贝

3、集合的内容拷贝只是生成新的集合对象，但是里面存放的仍是相同的引用，大致生成代码如下：
```
- (id) mutableCopy() {
    NSMutableArray copyArr = [[[self class] alloc] init];
    for(int i = 0; i < [self count]; i++) {
        [copyArr addObject: [self objectInIndex: i]];
    }
    return copyArr;
}
```


## 用Runtime新增一个类Person, person有name属性，有sayHi方法

```
void sayHi(id self, SEL _cmd) {
    NSLog(@"hi");
}

- (void) sayHi {
    
}

int creatPersonDemo() {
    
    Class person = objc_allocateClassPair([NSObject class], "Person", 0);
    
    class_addIvar(person, "name", sizeof(NSString *), 0, "@");
    
    //插入方法
    class_addMethod(person, @selector(sayHi), (IMP)sayHi, "v@:");
    
    return 0;
}
```

## 如何做selector not found Crash 防护？参考消息转发流程

1、方法1：每次进行消息发送时手动判断对象是否可相应该消息：
```
if ([obj respondsToSelector:aSelect]) {
[obj performSelector:aSelect withObject:nil];
//[obj update];
}  
```

2、方法2: 通过resolveInstanceMethod+runtime手动创建方法并输出日志：
```
void dynamicMethodIMP(id self, SEL _cmd)
{
    NSString *name = NSStringFromSelector(_cmd);
    NSLog(@"method %@ can't be found", name);
    //日志
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(missMethod)) {
        class_addMethod([self class], sel, (IMP)dynamicMethodIMP, "@v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

* 在上个例子中，当实现了-(void)setName:(NSString *)name方法，则在运行的时候，调用完我们实现的-(void)setName:(NSString *)name方法后，运行时系统仍然会调+(BOOL) resolveInstanceMethod:(SEL) sel方法，只不过这里的sel会变成_doZombieMe，从而我们实现重定向的if分支就进不去了，即我们实现的方法不会被覆盖。

* "v@:"属于Objective-C类型编码的内容。


3、方法3: 对方法二进行改进，直接通过category集成到NSObject类上：
```
#import "NSObject+MethodProtect.h"
#import "objc/runtime.h"


@implementation NSObject (MethodProtect)


void dynamicMethodIMP(id self, SEL _cmd)
{
    NSString *name = NSStringFromSelector(_cmd);
    NSLog(@"method %@ can't be found", name);
    //日志
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(missMethod)) {
        class_addMethod([self class], sel, (IMP)dynamicMethodIMP, "@v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
@end
```

方法4：在forwardInvocation方法做转发，当resolveInstanceMethod方法返回NO的时候就会调用forwardInvocation方法，可以在forwardInvocation中将消息转发给其他对象

```
-(void)forwardInvocation:(NSInvocation *)invocation
{
    SEL invSEL = invocation.selector;
    if ([someOtherObject respondsToSelector:invSEL])
        [anInvocation invokeWithTarget:someOtherObject];
    } else {
        [self doesNotRecognizeSelector:invSEL]; 
    }                                                                          
}
```

方法5:forwardingTargetForSelector方法允许我们替换消息当接受者，理论上可以统一将消息转发到一个实例，在那个实例中通过runtime动态生成方法

## class 1 附录

![image.png](img/img22.jpg)
