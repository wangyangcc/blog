title: 用Mantle构建Model层
date: 2014-11-04 11:27:43
tags:
---
> 在项目开发过程中，经常要自定义Model，然后在请求服务器得到数据后(一般是Json数据)，用字典取值的方式给自定义的Model赋值，封装成数据对象。这样做有几个问题：

- 服务器更新字段(或者添加字段)后，客户端要在每个Model初始化的地方修改(或者添加)取值字段，过程繁琐。
- 实现这些自定义的Model对象序列化保存到本地，需要自己一个一个实现，在字段添加或者修改的时候也要一个一个更改，过程繁琐。
- 自定义的Model没办法Copy，除非你实现<copying>协议，没办法反序列化成Json。

庆幸的是，伟大的`github`工程师们在`OC`平台上提供了一个设计优化、高度统一的框架`Mantle`来解决这些问题。

## Mantle为我们带来的：

- 实现了`NSCopying protocol`，子类终于可以直接copy了
- 实现了`NSCoding protocol`，可以通过`NSKeyedArchiver`保存到本地了。(`NSUserDefaults` 的替换选择)
- 提供了`-isEqual`:和`-hash`的默认实现，model作NSDictionary的key方便了许多
- 能在Model 和 Json 元数据之间相互转换

## Model的基本用法

#### 自定义的Model

自定义的Model都需要集成自`MTLModel`，并且实现`MTLJSONSerializing`协议，例如下面的:

	typedef enum : NSUInteger {
	    GHIssueStateOpen,
	    GHIssueStateClosed
	} GHIssueState;

	@interface GHIssue : MTLModel <MTLJSONSerializing>

	@property (nonatomic, copy, readonly) NSURL *URL;
	@property (nonatomic, copy, readonly) NSURL *HTMLURL;
	@property (nonatomic, copy, readonly) NSNumber *number;
	@property (nonatomic, assign, readonly) GHIssueState state;
	@property (nonatomic, copy, readonly) NSString *reporterLogin;
	@property (nonatomic, strong, readonly) GHUser *assignee;
	@property (nonatomic, copy, readonly) NSDate *updatedAt;

	@property (nonatomic, copy) NSString *title;
	@property (nonatomic, copy) NSString *body;

	@property (nonatomic, copy, readonly) NSDate *retrievedAt;

	@end

m 文件如下

	@implementation GHIssue
	
	+ (NSDateFormatter *)dateFormatter {
    	NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    	dateFormatter.locale = [[NSLocale alloc] initWithLocaleIdentifier:@"en_US_POSIX"];
    	dateFormatter.dateFormat = @"yyyy-MM-dd'T'HH:mm:ss'Z'";
    	return dateFormatter;
	}

	+ (NSDictionary *)JSONKeyPathsByPropertyKey {
    	return @{
        	@"URL": @"url",
        	@"HTMLURL": @"html_url",
        	@"reporterLogin": @"user.login",
        	@"assignee": @"assignee",
        	@"updatedAt": @"updated_at"
    };
	}

	+ (NSValueTransformer *)URLJSONTransformer {
    	return [NSValueTransformer valueTransformerForName:MTLURLValueTransformerName];
	}

	+ (NSValueTransformer *)HTMLURLJSONTransformer {
    	return [NSValueTransformer valueTransformerForName:MTLURLValueTransformerName];
	}

	+ (NSValueTransformer *)stateJSONTransformer {
    	return [NSValueTransformer mtl_valueMappingTransformerWithDictionary:@{
        	@"open": @(GHIssueStateOpen),
        	@"closed": @(GHIssueStateClosed)
    	}];
	}

	+ (NSValueTransformer *)assigneeJSONTransformer {
    	return [NSValueTransformer mtl_JSONDictionaryTransformerWithModelClass:GHUser.class];
	}

	+ (NSValueTransformer *)updatedAtJSONTransformer {
    	return [MTLValueTransformer reversibleTransformerWithForwardBlock:^(NSString *str) {
        	return [self.dateFormatter dateFromString:str];
    	} reverseBlock:^(NSDate *date) {
        	return [self.dateFormatter stringFromDate:date];
    	}];
	}

	- (instancetype)initWithDictionary:(NSDictionary *)dictionaryValue error:(NSError **)error {
    	self = [super initWithDictionary:dictionaryValue error:error];
    	if (self == nil) return nil;

    	// Store a value that needs to be determined locally upon initialization.
    	_retrievedAt = [NSDate date];

    	return self;
	}

	@end

#### MTLJSONSerializing 

继承自`MTLModel`并实现了`MTLJSONSerializing`协议的对象可以这样转换

从字典数据(JSONDictionary表示字典元数据)到 Model:

	NSError *error = nil;
	XYUser *user = [MTLJSONAdapter modelOfClass:XYUser.class 	fromJSONDictionary:JSONDictionary error:&error]; 


从Model到JSONDictionary数据
	
	NSDictionary *JSONDictionary = [MTLJSONAdapter JSONDictionaryFromModel:user];

##### `+ (NSDictionary *)JSONKeyPathsByPropertyKey;`用法如下：

	@interface XYUser : MTLModel

	@property (readonly, nonatomic, copy) NSString *name;
	@property (readonly, nonatomic, strong) NSDate *createdAt;

	@property (readonly, nonatomic, assign, getter = isMeUser) BOOL meUser;
	@property (readonly, nonatomic, strong) XYHelper *helper;

	@end

	@implementation XYUser

	+ (NSDictionary *)JSONKeyPathsByPropertyKey {
    	return @{
        	@"createdAt": @"created_at",
        	@"meUser": NSNull.null
    	};
	}

	- (instancetype)initWithDictionary:(NSDictionary *)dictionaryValue error:(NSError **)error {
    	self = [super initWithDictionary:dictionaryValue error:error];
    	if (self == nil) return nil;

    	_helper = [XYHelper helperWithName:self.name createdAt:self.createdAt];

    	return self;
	}

@end

返回的字典用来指定该Model的属性从字典里面怎么取值，比如`createdAt`是从字典里面取`created_at`字段，指定`@"meUser": NSNull.null`表示不从字典里面取值，没有在上面列出的Model属性取和属性同名的字典字段(比如`name`就从字典里面取`name`字段)。

##### +JSONTransformerForKey: 用法


	+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key {
    	if ([key isEqualToString:@"createdAt"]) {
        	return [NSValueTransformer valueTransformerForName:XYDateValueTransformerName];
    	}

    	return nil;
	}

实现上面的方法，用来指定属性从字典数据里面取出来的是什么类型的数据。比如上面`createdAt`属性从字典里面取值后会自动转换为`Date`类型。

如果有很多类型需要指定取值类型，那岂不有很多if，这太不优雅了！`Mantle`提供了更优雅的方法：实现类似`+<key>JSONTransformer`的方法，来指定某个属性从字典里面取值后的类型(或怎么取值)：

	+ (NSValueTransformer *)createdAtJSONTransformer {
    	return [MTLValueTransformer reversibleTransformerWithForwardBlock:^(NSString *str) {
        	return [self.dateFormatter dateFromString:str];
    	} reverseBlock:^(NSDate *date) {
        	return [self.dateFormatter stringFromDate:date];
    	}];
	}

##### +classForParsingJSONDictionary: 用法

当你自定义了一组Model，实现`+classForParsingJSONDictionary:`方法可以指定在转换`deserializing`字典的时候，用那个`Model class`

	@interface XYMessage : MTLModel

	@end

	@interface XYTextMessage: XYMessage

	@property (readonly, nonatomic, copy) NSString *body;

	@end

	@interface XYPictureMessage : XYMessage

	@property (readonly, nonatomic, strong) NSURL *imageURL;

	@end

	@implementation XYMessage

	+ (Class)classForParsingJSONDictionary:(NSDictionary *)JSONDictionary {
    	if (JSONDictionary[@"image_url"] != nil) {
        	return XYPictureMessage.class;
    	}

    	if (JSONDictionary[@"body"] != nil) {
        	return XYTextMessage.class;
    	}

    	NSAssert(NO, @"No matching class for the JSON dictionary '%@'.", 	   JSONDictionary);
    	return self;
	}

@end

`MTLJSONAdapter` 会根据你传入的字典数据实例化合适的类。

	NSDictionary *textMessage = @{
    	@"id": @1,
    	@"body": @"Hello World!"
	};

	NSDictionary *pictureMessage = @{
    	@"id": @2,
    	@"image_url": @"http://example.com/lolcat.gif"
	};

	XYTextMessage *messageA = [MTLJSONAdapter modelOfClass:XYMessage.class 	fromJSONDictionary:textMessage error:NULL];

	XYPictureMessage *messageB = [MTLJSONAdapter modelOfClass:XYMessage.class 	fromJSONDictionary:pictureMessage error:NULL];

`Mantle`代码托管在:[https://github.com/Mantle/Mantle](https://github.com/Mantle/Mantle)

唱吧6.0版本使用`Mantle`后，据说：`crash率比之前的版本有显示的降低，并且Mantle相关的crash占总crash的比率不到3%`，点[这里](http://www.iwangke.me/2014/10/13/Why-Changba-iOS-choose-Mantle/)查看更多关于唱吧使用`Mantle`后的总结