### 熟悉Objective-C

Objective-C是在c语言基础上添加了面相对象的特性。该语言使用了消息结构而非函数调用。

```Object-C
// messaging 

Object *obj = [Object new];
[obj performWith: parameter1 and: parameter2];

//function calling

Object *obj = new Object;
obj -> perform(parameter1, parameter2);
```
特点：

    1. 使用消息结构的语言，其运行时所执行的代码是由运行环境来决定。函数调用的语言由编译器决定。

    2. 消息结构的语言由于在运行时才知道要执行的东西，这就造成了一定的安全隐患，但是这也造就了这门语言的灵活性。

#### Objective-C的内存模型

我们声明了一个变量，变量会存在何处？对象所占用的内存是分配在堆空间中。

```Object-C
    NSString *string = @"ppp";
    NSString *anotherString = string;
    
    NSLog(@"%p", string);
    NSLog(@"%p", anotherString);

    //0x1002d5068
    //0x1002d5068
```

string和anotherString都是指向同一块内存地址的指针，它们都指向0x1002d5068内存的对象，该对象存储在堆上，而这两个变量则存储在栈上它们都保存着0x1002d5068对象的地址.

我们声明了一个名为string的变量，类型是 NSString*，对象所占用的内存分配在堆空间中，而绝不会在栈上。

```Object-C
NSString somestring;
//Interface type cannot be statically allocated
```

我们有时候会遇到一种叫做结构体的数据类型比如CGReact,它们会存储在栈空间里。存储在栈空间里有个好处就是程序会自动管理栈空间的内存，如果都使用对象的话，创建对象要额外的开销，还有分配堆内存。Swift语言中大量使用了结构这种数据类型，比如数组，字典等。官方鼓励我们在应在恰到的时候使用结构体代替类。恰当的时候是值如果该数据模型变动小，结构简单，应该优先考虑使用结构体。

### 尽量少的引用其他头文件

使用@class 类名 向前声明该类， 具体实现中才会在.m中引用。这样做的好处就是将头文件的时机尽量延后，只有在需要时才引入，这样就可以减少类的使用者所引入的头文件数量,减少编译时间。

如果两个文件在头文件里相互import的话，就会导致其中一个无法被正确的编译。如果使用include导入的话还会造成死循环。

必须引入其他的头文件： 如果一个类实现某个协议的时候。可以在头文件中导入,我们可以把协议放在一个单独的头文件中，但是这就会和以前一样造成依赖，我们可以实现文件中声明此类实现协议，并把事项代码写到分类里，这样的话可以减少编译时间，还可以减少依赖。

### 多用字面量语法，少用与之等价的方法

```Object-C
NSNumber *number = @1;
NSNumber *floatNum = @2.5;
NSArray *animals = @[@"cat", @"dog"];
NSDictionary *personData = @{
    @"name" : @"Bill"
};
NSMutableArray *mutableArray = [@[@1, @2] mutableCopy];
```

### 多用类型常量，少用#define预处理指令

```Object-C
#define ANIMATION_DURATION 0.3
//这样做会把多有ANIMATION_DURATION替换成0.3，而且还没有指定类型

static const NSTimeInterval kAnimationDuriation = 3;

//这种方法很好的描述了常量的含义，变量一定使用static const来声明，如果试图修改这个变量，编译器就会报错


//全局找那个使用，我们可以创建一个文件在

//头文件中使用

extern NSString *const kAnimationTime;
//extern是告诉编译器我要声明一个全局符号。命名一定要注意

//实现文件中

NSString *const kAnimationTime = @"3";
```

### 枚举

```Object-C
    enum LogState {
        LogStateInfo,
        LogStateSuccess,
        LogStateError
    };
```

上面那种写法的话定义枚举变量不太容易

```Object-C
enum LogState logState = LogStateInfo;
```

我们可以使用typedef重新进行定义

```Object-C
typedef enum LogState = LogState;

LogState logState = LogStateSuccess;
```

上面的枚举类型还可以直接使用下面这种方法来写

```Object-C
    typedef enum : NSUInteger {
        LogStateError,
        LogStateSuccess,
        LogStateInfo,
    } LogState;

    LogState logState = LogStateInfo;
```

枚举如果不写后面的值默认是从0开始的比如上面的LogStateInfo = 0,在ios中有如下枚举类型，表示某个视图在水平或者垂直方向上调整大小。

```Object-C
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

动画效果

```Object-C
typedef NS_ENUM(NSInteger, UIViewAnimationCurve) {
    UIViewAnimationCurveEaseInOut,         // slow at beginning and end
    UIViewAnimationCurveEaseIn,            // slow at beginning
    UIViewAnimationCurveEaseOut,           // slow at end
    UIViewAnimationCurveLinear,
};
```

我们看到了 NS\_ENUM 和 NS\_OPTIONS这些事Foundation框架中定义的一些辅助函数。


### 理解属性这一概念

对象是用来存储数据和进行操作的，在对象之间传递数据并且执行任务的过程叫做消息的传递。当程序运行起来后，Objective-C运行时为我们提供了一套可以使对象能够传递消息的重要函数。属性是Objective-C 的一项特性，用于封装数据。getter和setter方法分别用来读取数据和存储数据。使用@property进行定义，使用它进行标记的属性编译器会自动生成一套存取方法，此过程叫做自动合成。使用@dynamic关键字，他会告诉编译器不要自动创建实现属性所用的实例变量，也不要为为其创建存取方法。它相信在运行时期内找到，比如CoreData框架中的NSMangedObject类里创建一个自类，那么就需要在运行时期动态的创建存取方法。

```Object-C

@interface Person : NSManagedObjectModel
@property NSString *name;
@property NSInteger age;
@end

@implementation Person
@dynamic name, age;
@end

```

编译器不会为上面的这个类自动合成存取方法或者实例，访问属性也不会报错。

#### 属性特质

属性还有一些特质会影响存取方法的生成。

```Object-C
@property (nonatomic, readonly, copy) NSString *firstName;
```

属相可以拥有的特质分为四类：

1. 原子性
    如果你是nonatomic的特质，就不会使用同步锁。

2. assign 
    设置方法只针对值类型的。
3. strong
    拥有关系，为这种属性设置的时候是先保留新值并释放旧值，让后将新值释放上去
4. weak 表示非拥有关系设置方法是既不保留新值，也不释放旧值和assign差不多，属性所指对象摧毁时属性值也会清空

5. unsafe_unretained 
    非拥有关系，适用于对象类型，对象摧毁了属性也不会自动清空。不安全的

6. copy
    不保留新值而是将其copy一份，当属性是NSString*时经常用此特质来保护其封装性。只要实现属性是可变的，就应该设置新属性值是拷贝一份。

    ```Object-C
    @interface Person : NSObject
    @property(nonatomic, copy) NSString *name;
    @end


    Person *p = [Person new];
    NSMutableString *string = [NSMutableString stringWithFormat:@"Bill"];
    
    p.name = string;
    
    [string appendString:@"12"]; //修改了string
    
    NSLog(@"%@", p.name); //Bill


    //如果将上面的copy改成strong的话，打印的就是Bill12;
    ```