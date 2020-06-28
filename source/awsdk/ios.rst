iOS 端 SDK 接入指南
=======================

概述
--------------------
AWSDK 是一个适用于 iOS 的虚拟人解决方案SDK。

功能以及版本
~~~~~~~~



阅读对象
--------------------

- 具备基本的 iOS 开发能力
- 准备接入虚拟人SDK

开发准备
--------------------

设备以及系统
~~~~~~~~

- iPhone4s 及以上设备
- iOS 9 及以上操作系统

前置条件
~~~~~~~~

- 取得有效的SDK license文件

快速开始
--------------------
开发环境配置
~~~~~~~~
- Xcode 开发工具。App Store `下载地址`_

.. _下载地址: https://apps.apple.com/us/app/xcode/id497799835?ls=1&mt=12

- 使用真机调试

导入 SDK
~~~~~~~~
- 在工程配置中，将 AWSDK.framework 导入工程中，勾选Embed方式为 ``Embed & Sign`` ，如图所示
.. image:: /_static/img/awsdk_xcode_config.png

- 找到 Product -> Schemes -> Edit Scheme... -> Run -> Options， 将 ``Metal API Validation`` 选项勾选为 ``Disabled``
.. image:: /_static/img/awsdk_xcode_scheme.png

使用license
^^^^^^^^
SDK需要取得有效的license文件才可以使用，为此，我们可以在合适的地方（SDK其他Api使用之前）调用 ``setLicense`` 接口，导入license。例如，我们可以在 ``AppDelegate.m`` 中这样使用license文件：

.. code-block:: objc
   :linenos:
   
   #import "AppDelegate.h"
   #import <AWSDK/AWSDK.h>

   @interface AppDelegate ()
   
   @end

   @implementation AppDelegate
   
   ...
   
   - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
       // Override point for customization after application launch.
       [self setupLicense];
       return YES;
   }
   
   - (void)setupLicense
   {
      NSError *error;
      NSString *filepath = [[NSBundle mainBundle] pathForResource:@"license" ofType:@"hj"];
      NSString *license = [NSString stringWithContentsOfFile:filepath encoding:NSUTF8StringEncoding error:&error];
      if (error)
         NSLog(@"Error reading file: %@", error.localizedDescription);
      NSTimeInterval expired = [[AWSDK sharedSDK] setLicense:license];
      NSDate *date = [NSDate dateWithTimeIntervalSince1970:expired];
      NSLog(@"License过期于：%@", date);
   }
   
   ...
   
   @end

这个例子中，我们把 ``license.hj`` 文件放在了 ``mainBundle`` 里面了，因此需要确保license文件 ``license.hj`` 被正确拷贝到指定的目录中，如下

.. image:: /_static/img/awsdk_license_bundle.png

当然， ``license.hj`` 放在任何目录都可以，只要程序能读取出内容，并将内容传给 ``AWSDK`` 的 ``setLicense`` 接口即可。

初始化虚拟人逻辑
~~~~~~~~~~~

创建虚拟人用的 ViewController
^^^^^^^^
- 创建 View Controller，选择 subclass 为 ``UIViewController`` ，如图所示

.. image:: /_static/img/xcode_create_viewcontroller.png

添加引用
^^^^^^^^
在 ``CharacterViewController.h`` 头文件中添加引用

.. code-block:: objc
   :linenos:

   #import <AWSDK/AWSDK.h>
   
   
添加声明
^^^^^^^^
在 ``CharacterViewController.h`` 头文件中声明支持 ``AWSDKDelegate``，如下

.. code-block:: objc
   :linenos:
   
   #import <UIKit/UIKit.h>
   #import <AWSDK/AWSDK.h>
   @interface CharacterViewController : UIViewController <AWSDKDelegate>
   @end

在 ``CharacterViewController.m`` 源文件中，找到 ``- (void)viewDidLoad`` 方法，我们需要在这个方法中启动引擎。
.. code-block:: objc
   :linenos:
   
   - (void)viewDidLoad {
       [super viewDidLoad];
       // Do any additional setup after loading the view.
       [AWSDK sharedSDK].delegate = self;
       if (![AWSDK sharedSDK].engineReady) {
           [[AWSDK sharedSDK] startEngine];
       } else {
           UIView* renderView = [AWSDK sharedSDK].renderView;
           [self.view insertSubview:renderView atIndex:0];
       }
   }
   
在这个方法中，我们首先指定好 ``AWSDK`` 的 ``delegate``，然后判断引擎是否准备好。如果没准备好，就启动引擎，否则就将SDK提供的 ``renderView`` 插入到 ``CharacterViewController`` 的 ``view`` 中。``renderView`` 是一个将引擎内容渲染出来的视图，当引擎未启动的时候，``renderView`` 是个 ``nullptr``，只有当引擎准备好的时候，``renderView`` 才有值。那么，我们该如何知道 ``renderView`` 什么时候从 ``nullptr`` 变成有值呢？也就说，引擎什么时候准备好呢？这就需要监听

【需要注意的是，**引擎是一个单例，一旦启动就无法关闭**。】

功能使用
--------------------

