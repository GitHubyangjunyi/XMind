```objective-c
@interface BNRPerson: NSObject
{
  float _heightInMeters;
  int _weightInKilos;
}

- (float)heightInMeters;
- (void)setHeightInMeters:(float)h;
- (int)weightinKilos;
- (void)setWeightInKilos:(int)w;
- (float)bodyMassIndex;

@end

@implementation BNRPerson
- (float)heightInMeters
{
	return _heightInMeters;
}
- (void)setHeightInMeters:(float)h
{
  _heightInMeters = h;
}
- (int)weightinKilos
{
  return _weightInKilos;
}
- (void)setWeightInKilos:(int)w
{
  _weightInKilos = w;
}
- (float)bodyMassIndex
{
  return _weightInKilos / (_heightInMeters * _heightInMeters);
}
@end
```

###使用@property

```objective-c
@interface BNRPerson: NSObject

@property (nonatomic) float heightInMeters;
@property (nonatomic) int weightInKilos;

@end

@implementation BNRPerson
- (float)bodyMassIndex
{
  return [self weightInKilos] / ([self heightInMeters] * [self heightInMeters]);
}
@end
  
```

###另一种风格

```objective-c
```



类扩展或者说匿名范畴

```objective-c
@interface BNREmployee : BNRPerson
//{
//    NSMutableArray *_assets;//移动到类扩展以隐藏可变属性
//}
@property (nonatomic) unsigned int employeeID;
//@property (nonatomic) unsigned int officeAlarmCode;       //移动到实现文件中成为类扩展
@property (nonatomic) NSDate *hireDate;
@property (nonatomic, copy) NSArray *assets;      //分别声明一个属性和一个实例变量实例变量的类型和属性的类型就可以不一样
                                            //NSArray类型的含义是告诉其他类,如果询问我的assets就会得到一些不可变的对象,然而assets数组实际上是NSMutableArray实例,所以可以在BNREmployee.m文件中增加或者移除元素
-(double)yearsOfEmployment;
-(void)addAssets:(BNRAsset *)objects;
-(unsigned int)valueOfAssets;

@end
  
@interface BNREmployee()
{
    NSMutableArray *_assets;//移动到类扩展以隐藏可变属性
}

@property (nonatomic) unsigned int officeAlarmCode;

@end

@implementation BNREmployee

-(double)yearsOfEmployment
{
    if (self.hireDate) {
        NSDate *now = [NSDate date];
        NSTimeInterval secs = [now timeIntervalSinceDate: self.hireDate];
        return  secs / 31557600.0;
    } else {
        return 0;
    }
}

-(float)bodyMassIndex
{
    return 0.9 * [super bodyMassIndex];
}

-(void)setAssets:(NSArray *)assets
{
    _assets = [assets mutableCopy];
}

-(NSArray *)assets
{
    return [_assets copy];
}

-(void)addAssets:(BNRAsset *)objects
{
    if (!_assets) {
        _assets = [[NSMutableArray alloc] init];
    }
    [_assets addObject:objects];
}

-(unsigned int)valueOfAssets
{
    unsigned int sum = 0;
    for (BNRAsset *a in _assets) {
        sum += [a resaleValue];
    }
    return sum;
}

-(NSString *)description
{
    return [NSString stringWithFormat:@"<Employee %u: $%u in assets>", self.employeeID, self.valueOfAssets];
}

-(void)dealloc
{
    NSLog(@"deallocating %@", self);
}

@end
```

子类无法获取父类的类扩展
在类的头文件声明属性时其他对象只能看到属性的存取方法而不能直接获取到属性声明生成的实例变量，如果BNRPerson.h中声明了@property (nonatomic) NSMutableArray *friends;
那么在BNREmployee.m中即使BNREmployee是BNRPerson的子类也不能获取到_friends实例变量但是可以使用self.friends



字典的字面量初始化

```objective-c
NSDictionary *numberOfMoons = @{@"Mercury" : @0,
                                @"Venus"	 : @0}
```



###关于预编译文件Constants-Prefix.pch

在开发项目时经常需要为每一个代码文件导入一组特定的头文件PCH为了保持代码文件顶部的简洁

凡是出现在预编译文件中的头文件Xcode都会事前编译好并自动导入每个文件









头文件NSLocale.h中

extern NSString const *NSLocaleCurrencyCode；指针的值不变

NSLocale.m中

NSString const *NSLocaleCurrencyCode = @"currency";



```objective-c
enum Blend {
  Blend1 = 1,
  Blend2 = 2
};

typedef enum {
  Blend1 = 1,
  Blend2 = 2
}Blend;

typedef NS_ENUM(int, Blend) {
  Blend1,
  Blend2
}
```



init高级主题

```objective-c
@interface BNRAppliance : NSObject

@property (nonatomic, copy) NSString *productName;
@property (nonatomic) int voltage;

-(instancetype) initWithProductName:(NSString *)name;

@end

@implementation BNRAppliance

@synthesize productName = pName;

- (instancetype)init {
    return [self initWithProductName:@"UnknownName"];
}

// MARK: 指定初始化方法
-(instancetype) initWithProductName:(NSString *)name {
    if (self = [super init]) {
        pName = [name copy];
        _voltage = 120;
    }
    return self;
}

@end


@interface BNROwnedAppliance : BNRAppliance

@property (readonly) NSSet *ownerNames;

-(instancetype)initWithProductName:(NSString *)name firstOwnerName:(NSString *)fname;
-(void)addOwnerName:(NSString *)n;
-(void)removeOwnerName:(NSString *)n;

@end

@implementation BNROwnedAppliance
{
    NSMutableSet *_ownerNames;
}

// MARK: 指定初始化方法
-(instancetype)initWithProductName:(NSString *)name firstOwnerName:(NSString *)fname{
    if (self = [super initWithProductName:name]) {
        _ownerNames = [[NSMutableSet alloc] init];
    }
    if (fname) {
        [_ownerNames addObject:name];
    }
    return self;
}

-(instancetype)initWithProductName:(NSString *)name{
    return [self initWithProductName:name firstOwnerName: nil];
}
// 无需重写init因为init会调用到上面的方法
-(void)addOwnerName:(NSString *)n{
    [_ownerNames addObject:n];
}

-(void)removeOwnerName:(NSString *)n{
    [_ownerNames removeObject:n];
}

-(NSSet *)ownerNames{
    return [_ownerNames copy];
}

@end
```

其他初始化方法应该直接或者间接调用指定初始化

指定初始化方法应该先调用父类指定初始化然后再对实例变量进行初始化

如果某个类的指定初始化方法与父类的不同就需要重写父类指定初始化方法以调用新的指定初始化方法





如果声明一个属性并手动实现了存方法和取方法那么编译器不会合成实例变量

如果需要实例变量就在实现文件中添加@synthesize cxz = _cxz;

如果声明一个只读属性并且手动实现了取方法那么也不会合成实例变量



ARC不管理Core Foundation对象的生命周期所以需要用到

```objective-c
__bridge																		// 只做类型转换但不修改内存管理权
__bridge_retained/CFBridgingRetain 					// 将Objective-C对象转成Core Foundation对象并交由ARC管理相当于编译器做了Retained
__bridge_transfer/CFBridgingRelease					// 将Core Foundation对象转换成Objective-C对象并交由ARC管理相当于编译器做了Release
```





```objective-c
__unused	// 避免编译器报未使用参数的警告
_cmd			// 表示当前方法
```





通知中心不拥有观察者,如果将某个对象注册为观察者那么需要在对象的dealloc中移除观察者

```objective-c
- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver: self];
}
```



对象不拥有委托对象或者数据源对象

```objective-c
- (void)dealloc
{
  [setDelegate: nil];
}
```



对象不拥有目标

```objective-c
- (void)dealloc
{
  [button setTarget: nil];
}
```





Block相关

```objective-c
void (^devBlock)(id, NSString *, BOOL *)
// 外部变量捕获
// Block对象经常会使用外部创建的变量
// 对于基本类型的变量捕获会拷贝变量的值并保存在Block对象内部局部变量中
// 对于指针类型的变量Block对象会使用强引用
// 如果Employee对象有个Block
  block = ^{
  	NSLog("%@", self);
}
// 那么Block会捕获self导致强引用循环

	__weak Employee *weakSelf = self
	block = ^{
  	NSLog("%@", weakSelf); // 可能会被提前释放掉所以需要改成下面的
}

	__weak Employee *weakSelf = self
	block = ^{
    Employee *innerSelf = weakSelf;// 再次创建强引用循环保持，但是innerSelf只针对Block内部，Block执行完就消失了
  	NSLog("%@", innerSelf);
}
```

无意导致的self捕获

```objective-c
	__weak Employee *weakSelf = self
	block = ^{
    Employee *innerSelf = weakSelf;
  	NSLog("%@", innerSelf);
    NSLog("%@", _id); // 编译器将这里改成NSLog("%@", self->_id);导致强引用循环
}

	__weak Employee *weakSelf = self
	block = ^{
    Employee *innerSelf = weakSelf;
  	NSLog("%@", innerSelf);
    NSLog("%@", innerSelf.id); // 应该使用存取方法
}
```

需要修改外部变量

```objective-c
__block int counter = 0;
void (^block)() = ^{
  counter++;
}
block(); // counter = 1
```

协议

```objective-c
@protocol UITableViewDataSource <NSObject> // 继承自NSObject协议
  @required
  
  @optional
@end
```





Plist文件由以下对象组成

```obje
NSArray
NSDictionary
NSString
NSData
NSDate
NSNumber(整数/浮点/布尔值)
```



KVC

```objective-c
// setValue:forkey:方法会查找对应名称的存方法,如果没有存方法那么将直接为实例变量赋值
// valueForkye:方法会查找对应名称的取方法,如果没有取方法那么将直接返回实例变量
// 只声明注释掉

@interface BNRAppliance : NSObject
{
    NSString *_productName;
}
//@property (nonatomic, copy) NSString *productName;        //为了证明KVC能够在没有存取方法的情况下直接存取实例变量,所以注释掉这个属性声明
@property (nonatomic) int voltage;

-(instancetype) initWithProductName:(NSString *)name;

@end

NS_ASSUME_NONNULL_END

@implementation BNRAppliance

-(instancetype) initWithProductName:(NSString *)name{
    if (self = [super init]) {
        _productName = [name copy];
        _voltage = 120;
    }
    return self;
}

- (instancetype)init{
    return [self initWithProductName:@"UnknownName"];
}

-(NSString *)description{
    return [NSString stringWithFormat:@"<%@: %d volts>", _productName, self.voltage];
}

-(void)setVoltage:(int)value{
    NSLog(@"setting voltage to %d", value);
    _voltage = value;
}

@end  
  
```

非对象类型的KVC需要使用NSNumber

```obje
[a setValue:[NSNumber numberWithInt:: 240] forKey: "vol"];
```



KeyPath

大部分应用最后都有一个复杂的对象表,遍历关系时就不方便,使用KeyPath简化

```objective-c
NSString *str = [oa valueForKeyPath:@"manager.emergencyContact.phoneNumber"];
[sales setValue:@"555-606-0842" forKeyPath:@"manager.emergencyContact.phoneNumber"];
```



动作方法总是有一个实参,它是传入发送动作消息的那个对象

KVO

```obje
// 简单KVO观察
// 在KVO中使用context
// 直接设置实例变量需要显示触发KVO
// 独立的属性关联到KVO


```











