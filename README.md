# TouchIDDemo
简单的TouchID Demo
![Touch ID](http://upload-images.jianshu.io/upload_images/2487620-2e8b1d031a72da4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
>最近一直比较有空,所以就在这段空闲时间就打算梳理一下以前做过的一些项目，并且把一些简单常用的API回顾回顾，今天看了一下指纹解锁的官方Demo，就想学习一下。其实功能实现起来很简单，毕竟苹果官方都帮我们封装好了，我们只需要学习几个方法，实现一下即可让自己的App有手势解锁的功能了。


![实现效果图](http://upload-images.jianshu.io/upload_images/2487620-30bf8856a39ef3d7.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

###一、API介绍
>iOS的指纹解锁官方提供了```LocalAuthentication.framework```这个系统库，打开这个库我们会发现里面只有以下4个.h文件
>- LAContext.h
- LAError.h
- LAPublicDefines.h
- LocalAuthentication.h

######①LAContext.h（最核心部分）
一开始我们就看到```LAPolicy```这个枚举
```
typedef NS_ENUM(NSInteger, LAPolicy)
```
>```LAPolicyDeviceOwnerAuthenticationWithBiometrics```（iOS8以上可用）：这种代表的是只用指纹去验证。第一次指纹失败，会出现“输入密码”按钮，输入密码的标题及功能可以自定义；第三次指纹失败，弹窗消失；再次启动验证，还有两次机会，如果都失败了，指纹验证锁定，不再弹出验证窗。直至输入密码来解锁指纹（可以锁屏重新进来使用输入密码的方式解锁）。
```LAPolicyDeviceOwnerAuthentication```（iOS9以上可用）：这种代表的是可以用指纹或密码两种方式去验证，优先用指纹。第一次指纹失败，会出现“输入密码”按钮，输入密码的标题可以自定义，但是功能不能自定义了，而是必须输入系统密码（锁屏密码）；第三次验证失败，弹窗消失，弹出输入系统密码的界面；如果连续五次指纹失败，则指纹锁定，此时只会弹出输入密码界面，直至输入密码成功解锁。

两种验证方式的比较：
相同点：都是连续五次验证失败就会锁定
不同点：前者的输入密码功能可以自定义，后者输入密码功能是固定输入系统密码一般经常使用```LAPolicyDeviceOwnerAuthenticationWithBiometrics```

然后我们可以看到以下面几个方法
######①这个方法是判断当前设备是否支持TouchID的
```
/*
  这个方法用来检查当前设备是否可用touchID，返回一个BOOL值
  policy: 这个就是上面的枚举的两个验证方式，一般用前者
  error:  错误的类型可参考LAError.h里的类型
*/
- (BOOL)canEvaluatePolicy:(LAPolicy)policy error:(NSError * __autoreleasing *)error __attribute__((swift_error(none)));
```
>@discussion Policies can have certain requirements which, when not satisfied, would always cause
///             the policy evaluation to fail. Examples can be a passcode set or a fingerprint
///             enrolled with Touch ID. This method allows easy checking for such conditions.
///
///             Applications should consume the returned value immediately and avoid relying on it
///             for an extensive period of time. At least, it is guaranteed to stay valid until the
///             application enters background.

>@return YES if the policy can be evaluated, NO otherwise.


######②这个方法是用来验证TouchID的，会有弹出框出来
```
/*
  这个方法是开始验证指纹的方法
  policy: 这个就是上面的枚举的两个验证方式，一般用前者
  localizedReason: 指纹验证框上面的提示信息，一般为“通过Home键验证已有手机指纹”（不能为空否则崩溃）
  reply: 一个block，返回指纹验证结果，成功:success为YES，失败:success为NO，同时返回错误类型的error，同样参考LAError.h里的类型
*/
- (void)evaluatePolicy:(LAPolicy)policy
       localizedReason:(NSString *)localizedReason
                 reply:(void(^)(BOOL success, NSError * __nullable error))reply;
```
######③这个方法用来废止该实例对象context
```
- (void)invalidate NS_AVAILABLE(10_11, 9_0);
```
然后下面还有几个方法不是很常用，我们可以忽略
接下来介绍一下比较重要的属性
```
// 可以设置指纹弹框“输入密码”按钮的标题，如果不设置或设置为nil，则显示默认的“输入密码”；如果设置为@""，则弹框不再显示这个按钮
@property (nonatomic, nullable, copy) NSString *localizedFallbackTitle;
```
```
// 可以设置指纹弹框“取消”按钮的标题（iOS10.0以上可用），如果不设置或设置为nil或设置为@""，都显示默认的“取消”
@property (nonatomic, nullable, copy) NSString *localizedCancelTitle NS_AVAILABLE(10_12, 10_0);
```
```
// 最大指纹尝试错误次数（iOS8.3 - iOS9.0可用）
@property (nonatomic, nullable) NSNumber *maxBiometryFailures NS_DEPRECATED_IOS(8_3, 9_0) __WATCHOS_UNAVAILABLE __TVOS_UNAVAILABLE;
```
```
// 这个可以检测你的指纹数据库的变化,增加或者删除指纹这个属性会做出相应的反应（iOS9.0以上可用）
@property (nonatomic, nullable, readonly) NSData *evaluatedPolicyDomainState NS_AVAILABLE(10_11, 9_0) __WATCHOS_UNAVAILABLE __TVOS_UNAVAILABLE;
```
```
// 两次开启指纹之间的时间间隔，决定第二次是否需要指纹解锁
@property (nonatomic) NSTimeInterval touchIDAuthenticationAllowableReuseDuration NS_AVAILABLE(NA, 9_0) __WATCHOS_UNAVAILABLE __TVOS_UNAVAILABLE;。
```
***
######②LAError.h
这个类主要枚举了可能出现的错误类型，具体中文解释如下
```
 LAErrorAuthenticationFailed //连续三次指纹验证失败，可能指纹模糊或用错手指
 LAErrorUserCancel           //用户取消验证，点击了取消按钮
 LAErrorUserFallback         //用户取消验证，点击了输入密码按钮
 LAErrorSystemCancel         //系统取消授权，如其他APP切入
 LAErrorPasscodeNotSet       //指纹验证无法启动/失败，因为设备没有设置密码
 LAErrorTouchIDNotAvailable  //设备TouchID不可用，例如未打开
 LAErrorTouchIDNotEnrolled   //指纹验证无法启动，因为没有录入指纹(设置密码了)
 LAErrorTouchIDLockout       //设备TouchID被锁定，因为失败的次数太多了
 LAErrorAppCancel            //应用程序取消了身份验证，APP调用了-(void)invalidate方法使LAContext失效
 LAErrorInvalidContext       //实例化的LAContext对象失效，再次调用evaluation...方法则会弹出此错误信息
```
***
######③LAPublicDefines.h
只是这个库的一些宏定义
![LAPublicDefines.h](http://upload-images.jianshu.io/upload_images/2487620-bdb94ae6fab05b7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)
***
######④LocalAuthentication.h
这个里面就两行引入头文件，很显然这是我们引入指纹库要调用的类，在文件中引入```#import <LocalAuthentication/LocalAuthentication.h>```即可。
***
###二、实现指纹解锁

#*第1步：引入指纹解锁头文件
```
#import <LocalAuthentication/LocalAuthentication.h>
```
#*第2步：实现指纹解锁的主要方法
```
- (void)evaluateAuthenticate
{
    //iOS 8以上才支持指纹识别接口
    if ([[UIDevice currentDevice].systemVersion floatValue] < 8) {
        NSLog(@"不支持TouchID (版本必须高于iOS 8.0才能使用)");
        return;
    }

    //创建LAContext
    LAContext *context = [[LAContext alloc] init];
    context.localizedFallbackTitle = @"输入密码吧";

    NSError *Error = nil;

    //判断设备支持状态
    if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&Error]) {
        //支持指纹验证
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"请验证已有手机指纹" reply:^(BOOL success, NSError *error) {
            if (success) {
                //验证成功，主线程处理UI
                [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                    NSLog(@"指纹验证成功");
                }];
            } else {
                NSLog(@"验证失败 == %@", error.localizedDescription);
                switch (error.code) {
                    case LAErrorSystemCancel:{
                        NSLog(@"系统取消授权，如其他APP切入");
                    }
                        break;
                    case LAErrorUserCancel:{
                        NSLog(@"用户取消验证，点击了取消按钮");
                    }
                        break;
                    case LAErrorUserFallback:{
                        NSLog(@"用户取消验证，点击了输入密码按钮");
                        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                            //用户选择输入密码，切换主线程处理

                        }];
                    }
                        break;
                    case LAErrorAuthenticationFailed:{
                        NSLog(@"连续三次指纹验证失败，可能指纹模糊或用错手指");
                    }
                        break;
                    case LAErrorTouchIDLockout:{
                        NSLog(@"设备TouchID被锁定，因为失败的次数太多了");
                    }
                        break;
                    default:{
                        NSLog(@"设备TouchID不可用。。。");
                        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                            //其他情况，切换主线程处理
                        }];
                    }
                        break;
                }
            }
        }];
    } else {
        //该设备不支持TouchID
        NSLog(@"不支持TouchID == %@", Error.localizedDescription);
        switch (Error.code) {
            case LAErrorTouchIDNotEnrolled:{
                NSLog(@"指纹验证无法启动，因为没有录入指纹");
            }
                break;
            case LAErrorPasscodeNotSet:{
                NSLog(@"指纹验证无法启动，因为设备没有设置密码");
            }
                break;
            case LAErrorTouchIDLockout:{
                NSLog(@"设备TouchID被锁定，因为失败的次数太多了");
            }
                break;
            default:{
                NSLog(@"设备TouchID不可用。。。");
            }
                break;
        }
    }
}
```
最后就附上简单的[Demo原码](https://github.com/mylovetmcg/TouchIDDemo),如果文章对你有帮助记得点赞哦!
