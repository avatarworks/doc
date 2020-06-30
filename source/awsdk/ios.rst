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

- iPhone4s 或 iPad mini3 及以上设备
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

- iPhone 或 iPad 等调试设备，不支持模拟器调试

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

添加生命周期方法
^^^^^^^^^^^^
将如下生命周期方法添加到 ``AppDelegate.m`` 中

.. code-block:: objc
   :linenos:
   
   - (void)applicationWillTerminate:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationWillTerminate];
   }

   - (void)applicationDidBecomeActive:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationDidBecomeActive];
   }

   - (void)applicationWillResignActive:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationWillResignActive];
   }
   - (void)applicationWillEnterForeground:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationWillEnterForeground];
   }

   - (void)applicationDidEnterBackground:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationDidEnterBackground];
   }

   - (void)applicationDidReceiveMemoryWarning:(UIApplication *)application
   {
       [[AWSDK sharedSDK] applicationDidReceiveMemoryWarning];
   }

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
   
在这个方法中，我们首先指定好 ``AWSDK`` 的 ``delegate``，然后判断引擎是否准备好。如果没准备好，就启动引擎，否则就将SDK提供的 ``renderView`` 插入到 ``CharacterViewController`` 的 ``view`` 中。``renderView`` 是一个将引擎内容渲染出来的视图，当引擎未启动的时候，``renderView`` 是个 ``nullptr``，只有当引擎准备好的时候，``renderView`` 才有值。那么，我们该如何知道 ``renderView`` 什么时候从 ``nullptr`` 变成有值呢，从而将 ``renderView`` 添加进来呢？这就需要从引擎结束加载的回调，即 ``AWSDKDelegate`` 的 ``engineEndLoading`` 方法，去处理，如下：

.. code-block:: objc
   :linenos:
   
   - (void)engineEndLoading
   {
       UIView* renderView = [AWSDK sharedSDK].renderView;
       [self.view insertSubview:renderView atIndex:0];
   }

**【特别注意！！！引擎是一个单例，一旦启动就无法关闭。】**

配置资源和缓存目录
^^^^^^^^^
引擎启动后，我们需要配置资源和缓存目录。

.. code-block:: objc
   :linenos:
   
   - (void)setupDirs
   {
       NSURL* documentUrl = [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
       NSString * cacheDir = [documentUrl.path stringByAppendingString:@"/cache"];
       NSString *resDir = [[[NSBundle mainBundle] bundlePath] stringByAppendingString:@"/media"];

       [[AWResourceManager sharedManager] setCacheDirectory:cacheDir];
       [[AWResourceManager sharedManager] addResourceDirectory:resDir];
   }

在这个例子里，我们分别调用了两个 ``AWResourceManager`` 提供的接口来配置资源和缓存路径。其中，

- ``setCacheDirectory`` 用于设置缓存路径。缓存路径要求必须具备可让程序读写的权限，一般像 ``NSDocumentDirectory`` 就是一个理想的路径。
- ``addResourceDirectory`` 用于添加资源路径。**程序可以添加任意多个资源路径**。为了方便，我们把 ``mainBundle`` 下的 ``media`` 目录添加进了资源路径列表中。为此，请确保 ``media`` 目录能被正确拷贝到 ``mainBundle`` 中，如下

.. image:: /_static/img/awsdk_media_bundle.png

对于需要将内置资源从 AWSDK.framework 中分离出来的情况下，可通过如下方式实现

.. code-block:: objc
   :linenos:
   
   [[AWResourceManager sharedManager] setBaseDirectory:baseDir];
   
其中，``baseDir`` 是分离出来的资源目录。

定义好资源和缓存目录，我们就可以在 ``engineEndLoading`` 调用 ``setupDirs`` 了。如下

.. code-block:: objc
   :linenos:
   
   - (void)engineEndLoading
   {
       UIView* renderView = [AWSDK sharedSDK].renderView;
       [self.view insertSubview:renderView atIndex:0];
       [self setupDirs];
   }


加载角色
^^^^^^^^^

配置完资源和缓存目录，接下来就是载入一个角色。为了加载一个角色，我们需要角色的人脸贴图文件和人脸target文件。这两个文件一般可通过重建服务获得，详见：:ref:`人脸服务`

假设 ``media`` 目录下已经存在着人脸贴图文件 ``face/face1.jpg`` 和人脸target文件 ``face/face1.target``，则可以通过如下方法载入一个女性（``female``）角色

.. code-block:: objc
   :linenos:
   
   - (void)loadCharacter
   {
       AWCharacter* character = [AWCharacter new];

       AWValue* faceTarget = [AWValue valueOfString:@"face/face1.target"];
       AWValue* faceTexture = [AWValue valueOfString:@"face/face1.jpg"];
       AWValue* gender = [AWValue valueOfString:@"female"];

       [character setConfigs:@{
           AWCharacterConfigKeyFaceTarget: faceTarget,
           AWCharacterConfigKeyFaceTexture: faceTexture,
           AWCharacterConfigKeyGender: gender,
       }];
   }
   

这个方法可以在 ``setupDirs`` 之后调用，例如

.. code-block:: objc
   :linenos:
   
   - (void)engineEndLoading
   {
       UIView* renderView = [AWSDK sharedSDK].renderView;
       [self.view insertSubview:renderView atIndex:0];
       [self setupDirs];
       [self loadCharacter];
   }
   
注意事项 Q&A
^^^^^^^^

**Q**：为何 ``AWCharacter`` 创建的对象在被释放后，角色依然显示在 ``renderView`` 中？

**A**：``AWCharacter`` 是一个角色的配置类，不是角色本身。如果想要移除角色，需要调用 ``AWCharacter`` 的 ``remove`` 方法。

**Q**：我按照上面的配置，但 ``engineEndLoading`` 并没有回调

**A**：有可能哪里出错了，可以实现 ``AWSDKDelegate`` 的 ``engineError:`` 方法，查看错误提示。



功能使用
--------------------

SDK 设计理念
~~~~~~~~~~~~~

基于状态变化的更新机制
^^^^^^^^^^^

整个 SDK 的设计理念是维护一个全局的状态（State）。这个全局的状态又由若干个子状态组成，如一个角色就构成了一个子状态，一个镜头也构成了一个子状态。每个子状态分别包含了若干个键值对（key-value pair），SDK 会响应键（key）对应的值（value）是否发生变化来更新画面。例如，对于一个角色，当性别 ``AWCharacterConfigKeyGender`` 的值从 ``female`` 变成了 ``male``，画面中的角色就会从女性变成了男性。这些键值对的更新，一般可通过对应类的 ``setConfigs`` 方法来实现。例如，

.. code-block:: objc
   :linenos:
   
   [character setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget,
      AWCharacterConfigKeyFaceTexture: faceTexture,
      AWCharacterConfigKeyGender: gender,
   }];

表示需要对角色的脸部target、脸部贴图和性别做出改变。对于没在这一次 ``setConfigs`` 中指定的键值对，SDK 会认为那些键值对没有做出更改，从而不响应相应的变化。

若想让某一键值对恢复到默认值，可以将这个键值对的值置为 ``[AWValue null]``，例如

.. code-block:: objc
   :linenos:
   
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: [AWValue null]
   }];

表示将角色的位置恢复到默认值。
    

线程
^^^^^^^^^

SDK 完全跑在一个独立的线程上，从而使得 SDK 的内部操作，在一般情况下不影响主线程（或UI线程）的性能。但正如所有异步操作可能带来的同步问题一样，开发者在主线程更新SDK的时候，也不可避免的要注意线程同步问题。为了方便开发者使用，对于 **同类型** 的操作，例如更新操作，SDK 会将每一步操作丢入一个 FIFO 队列中，使开发者不需要等待上一个操作的完成，就可以去处理下一个操作。同时，SDK 还提供了解决队列拥堵的机制：即当前一个操作因为耗时而堵塞队列时，后面的操作会自动合并成一个大的操作，从而使得在前一个操作结束以后，队列后面遗留的操作可以直接同步到最终想要的状态。例如，

.. code-block:: objc
   :linenos:
   
   // 操作1 -> 更新脸部Target、脸部贴图和性别
   [character setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget,
      AWCharacterConfigKeyFaceTexture: faceTexture,
      AWCharacterConfigKeyGender: gender,
   }];
   
   // 操作2 -> 更新到位置1
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: position1
   }];
   
   // 操作3 -> 更新到位置2
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: position2
   }];
   
   // 操作4 -> 更新到位置3
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: position3
   }];
   
   // 操作5 -> 更新旋转角
   [character setConfigs:@{
      AWCharacterConfigKeyRotation: rotation
   }];
   
操作1是一个耗时的操作，这会造成操作2到操作5滞留在队列中。但是，当操作1执行结束后，操作2到操作5会自动合并成如下一个 *等价* 的操作，

.. code-block:: objc
   :linenos:
   
   // 等价的操作: 更新到位置3 + 更新旋转角
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: position3,
      AWCharacterConfigKeyRotation: rotation
   }];

从上面的例子可以看出，开发者期待的角色最终“位置”和“旋转”应该是 ``position3`` 和 ``rotation``，而这正是最终自动合并后的结果。

不过，对于非同类型的操作，例如更新角色和截屏这两个操作，由于它们是互相独立的，我们并不能保障谁先进行，所以最好的办法只能是通过一个操作的完成回调去调用另一个操作。

监听角色的状态变化
~~~~~~~~~~~~~~~~
``AWCharacter`` 支持 ``AWCharacterDelegate`` 协议，后者可以监听角色的各种状态变化，如：

- 即将加载 ``characterWillLoad:``
- 成功加载 ``characterDidLoad:``
- 加载失败 ``characterLoadFailed:withError:``
- 即将更新 ``characterWillUpdate:``
- 成功更新 ``characterDidUpdate:``
- 更新失败 ``characterUpdateFailed:withError:``
- 即将释放 ``characterWillRelease:``
- 成功释放 ``characterDidRelease:``

等等。

给角色更换服饰
~~~~~~~~~~~~~~~~

若开发者取得了授权的服装、发型等服饰资源，就可以在 SDK 里使用这些服饰，并穿在角色身上。假设开发者的资源目录有如下结构：

::

   .
   ├── face
   |   ├── face1.jpg
   |   └── face1.target
   └── dress
       ├── hair.zip
       ├── shirt.zip
       ├── pant.zip
       └── shoe.zip
   
``face`` 文件夹我们已经在前文介绍了，这里不再赘述。``dress`` 文件夹存放的资源是用于给角色穿戴的服装、发型、鞋子等。我们可以使用如下方式给角色穿上这些服饰：

.. code-block:: objc
   :linenos:
   
   NSArray* dressArr = @[
      @"dress/hair",
      @"dress/shirt",
      @"dress/pant",
      @"dress/shoe",
   ];
   NSData* dressData = [NSJSONSerialization dataWithJSONObject:dressArr options:NSJSONWritingPrettyPrinted error:NULL];
   AWValue* dress = [AWValue valueOfJson:dressData];
   [character setConfigs:@{
      AWCharacterConfigKeyDressArray: dress
   }];
   
需要注意的是，``dressArr`` 指定的服饰资源列表中，我们需要把 ``.zip`` 后缀去掉。


给角色变形
~~~~~~~~~~~~~~~~

SDK 提供了丰富的变形参数，具体可查询：

- :ref:`男性角色变形 Target 查询表` 
- :ref:`女性角色变形 Target 查询表`



让角色播放动画
~~~~~~~~~~~~~~~~

肢体动画
^^^^^^^^^^^

口型动画
^^^^^^^^^^^

调整角色的位置
~~~~~~~~~~~~~~~~

调整镜头的位置
~~~~~~~~~~~~~~~~

载入更多角色
~~~~~~~~~~~~~~~~

开启多镜头
~~~~~~~~~~~~~~~~


