Android 端 SDK 接入指南
***********************

概述
======================
AWSDK 是一个适用于 Android 的虚拟人解决方案。

阅读对象
======================

- 具备基本的 Android 开发能力
- 准备接入虚拟人 SDK

开发准备
======================

设备以及系统
~~~~~~~~

- Android 4.4 (API 19)及以上操作系统

混淆
~~~~~~~~

为了保证正常使用 SDK ，请在 ``proguard-rules.pro`` 文件中添加以下代码：

.. code-block:: guess
  :linenos:
  
  -keep class com.avatarworks.** { *; }
  -keep class com.libsdl.** { *; }

前置条件
~~~~~~~~

- 取得有效的 SDK license 文件

快速开始
======================

开发环境配置
~~~~~~~~

- Android Studio 开发工具，官方 `下载地址`_

.. _下载地址: http://developer.android.com/intl/zh-cn/sdk/index.html

- Android 官方开发 SDK，官方 `下载地址`_

.. _下载地址: http://developer.android.com/intl/zh-cn/sdk/index.html

SDK 集成
~~~~~~~~

导入 SDK
^^^^^^^

在工程配置中，将 ``awsdk.aar`` 导入工程中的 ``libs`` 目录下

.. image:: /_static/img/awsdk_android_studio_aar.png

修改 build.gradle
^^^^^^^^^^

双击打开您的工程目录下的 ``build.gradle``，确保已经添加了如下依赖，如下所示：

.. code-block:: guess
  :linenos:
  
  dependencies {
    implementation files('libs/awsdk.aar')
  }


使用 license
~~~~~~~~

SDK 需要取得有效的 license 文件才可以使用。为此，我们可以在合适的地方（在 SDK 使用其他 API 之前）调用 ``setLicense`` 接口，导入 license 内容。例如，我们可以在 ``MainActivity.java`` 中这样使用 license 文件：

.. code-block:: java
    :linenos:
   
    public class MainActivity extends AppCompatActivity {

        ...
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            ...
            setupLicense();
            ...
        }
    
       private void setupLicense() {
            String license = "";
            try {
                InputStream stream = getAssets().open("license.hj");

                int size = stream.available();
                byte[] buffer = new byte[size];
                stream.read(buffer);
                stream.close();
                license = new String(buffer);
            } catch (IOException e) {
                // Handle exceptions here
            }
            long expired = AWSDK.getInstance().setLicense(license);
            Log.i(TAG, "license expired in " + expired);
        }
        
        ...
    }

这个例子中，我们把 ``license.hj`` 文件放在了 ``assets`` 目录里面了，如下

.. image:: /_static/img/awsdk_license_assets.png

当然， ``license.hj`` 放在任何目录都可以，只要程序能读取出内容，并将内容传给 ``AWSDK`` 的 ``setLicense`` 接口即可。



初始化虚拟人逻辑
~~~~~~~~~~~

创建虚拟人用的 Activity
^^^^^^^^
- 创建 Activity，选择 subclass 为 ``UIViewController`` ，如图所示

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
   
在这个方法中，我们首先指定好 ``AWSDK`` 的 ``delegate``，然后判断引擎是否准备好。如果没准备好，就启动引擎，否则就将 SDK 提供的 ``renderView`` 插入到 ``CharacterViewController`` 的 ``view`` 中。``renderView`` 是一个将引擎内容渲染出来的视图，当引擎未启动的时候，``renderView`` 是个空指针，只有当引擎准备好的时候，``renderView`` 才有值。那么，我们该如何知道 ``renderView`` 什么时候不是空指针，从而可以将其添加到某个 ``View`` 呢？引擎准备好的时候，会调用 ``AWSDKDelegate`` 的 ``engineEndLoading`` 方法，这个地方就是将 ``renderView`` 添加进来的绝佳位置。例如：

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

对于需要将内置基础资源从 AWSDK.framework 中分离出来的情况，我们需要指定基础资源的路径，如下

.. code-block:: objc
   :linenos:
   
   [[AWResourceManager sharedManager] setBaseDirectory:baseDir];
   
其中，``baseDir`` 是分离出来的基础资源目录。

定义好资源和缓存目录，就可以在 ``engineEndLoading`` 调用 ``setupDirs`` 了。如下

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

配置完资源和缓存目录，接下来就是载入一个角色。为了加载一个角色，我们需要角色的人脸贴图文件和人脸 target 文件。这两个文件一般可通过重建服务获得，详见：:ref:`人脸服务`

假设 ``media`` 目录下已经存在着人脸贴图文件 ``face/face1.jpg`` 和人脸 target 文件 ``face/face1.target``，则可以通过如下方法载入一个女性（``female``）角色

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
   
至此，不出意外的话，角色就可以加载出来了。

注意事项 Q&A
^^^^^^^^

**Q**：为何 ``AWCharacter`` 创建的对象在被释放后，角色依然显示在 ``renderView`` 中？

**A**：``AWCharacter`` 是一个角色的配置类，不是角色本身。如果想要移除角色，需要调用 ``AWCharacter`` 的 ``remove`` 方法。

**Q**：我已经按照上面的方式进行配置了，但为什么 ``engineEndLoading`` 依然没有回调？

**A**：有可能哪里出错了，可以实现 ``AWSDKDelegate`` 的 ``engineError:`` 协议方法，查看错误提示。



SDK 设计理念
======================

基于状态变化的更新机制
~~~~~~~~~~~

整个 SDK 的设计理念是维护一个全局的状态（State）。这个全局的状态又由若干个子状态组成，如一个角色就构成了一个子状态，一个镜头也构成了一个子状态。每个子状态分别包含了若干个键值对（key-value pair），SDK 会响应键（key）对应的值（value）是否发生变化来更新画面。例如，对于一个角色，当性别 ``AWCharacterConfigKeyGender`` 的值从 ``female`` 变成了 ``male``，画面中的角色就会从女性变成了男性。这些键值对的更新，一般可通过对应类的 ``setConfigs`` 方法来实现。例如，

.. code-block:: objc
   :linenos:
   
   [character setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget,
      AWCharacterConfigKeyFaceTexture: faceTexture,
      AWCharacterConfigKeyGender: gender,
   }];

表示需要对角色的脸部target、脸部贴图和性别做出改变。对于没在这一次 ``setConfigs`` 中指定的键值对，SDK 会认为那些键值对没有做出更改，从而不响应相应的变化。我们把这种方式叫做 ``setConfigs`` 的增量更新。

若想让某一键值对恢复到默认值，可以将这个键值对的值置为 ``[AWValue null]``，例如

.. code-block:: objc
   :linenos:
   
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: [AWValue null]
   }];

表示将角色的位置恢复到默认值。

注意：**和 setConfigs 的增量更新有所不同，单个键值对里的值，在更新的时候总是被替换更新，而不是增量更新。** 例如，假设 ``AWCharacterConfigKeyDressArray`` 的前值是 ``@[@"dress1", @"dress2"]``，当再给它赋值 ``@[@"dress2", @"dress3"]`` 时，最终的结果应该就是 ``@[@"dress2", @"dress3"]``，而不是 ``@[@"dress1", @"dress2", @"dress3"]``。

线程
~~~~~~~~~~~

SDK 跑在一个完全独立的线程上，从而使得 SDK 的内部操作，在一般情况下不影响主线程（或UI线程）的性能。但正如所有异步操作可能带来的同步问题一样，开发者在主线程更新SDK的时候，也不可避免的要注意线程同步问题。为了方便开发者使用，对于 **同类型** 的操作，例如两个更新角色的操作，SDK 会将每一步操作丢入一个 FIFO 队列中，使开发者不需要等待上一个操作的完成，就可以去处理下一个操作。同时，SDK 还提供了解决队列拥堵的机制：即当前一个操作因为耗时而堵塞队列时，后面的操作会自动合并成一个大的操作，使得在前一个操作结束以后，队列后面遗留的操作可以直接同步到最终想要的状态。例如，

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

从上面的例子可以看出，开发者期待的角色最终“位置”和“旋转”应该是 ``position3`` 和 ``rotation``，而这正是自动合并后的结果。

不过，对于非同类型的操作，例如更新角色和截屏这两个操作，由于它们是互相独立的，我们并不能保障谁先进行，所以最好的办法只能是通过一个操作的完成回调去调用另一个操作。

功能使用
=======================

人脸重建授权码
~~~~~~~~~~~~~~~~~~~

开发者可通过 :ref:`人脸服务` 获得用于角色显示所需的脸部贴图和脸部 target。:ref:`人脸服务` 需要的 **签名认证串** 可通过如下方式获得：


.. code-block:: objc
   :linenos:
   
   [[AWSDK sharedSDK] genAuthString];


全局背景色
~~~~~~~~~~~~~~~~~~~

``renderView`` 可通过如下方式设置全局背景色

.. code-block:: objc
   :linenos:
   
   // 将全局背景色设置为白色
   [[AWSDK sharedSDK] setFogColor:[UIColor whiteColor]];


AWCharacter
~~~~~~~~~~~~~~~~~~~~

``AWCharacter`` 用于配置角色的状态，使角色显示在 ``renderView`` 中。

监听角色的状态变化
^^^^^^^^^^^^^^^^^^^
通过实现 ``AWCharacter`` 的 ``AWCharacterDelegate`` 协议，程序可以监听角色的各种状态变化，如：

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
^^^^^^^^^^^^^^^^^^^

若开发者取得了授权的服装、发型等资源（为了方便讨论，以下统称为“服饰”），就可以在 SDK 里使用这些服饰，并穿在角色身上。假设开发者的资源目录有如下结构：

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
^^^^^^^^^^^^^^^^^^^

SDK 提供了丰富的变形参数，具体可查询：

- :ref:`男性角色变形 Target 查询表` 
- :ref:`女性角色变形 Target 查询表`

假设我们需要给女性角色应用如下变形，

- 可爱脸型，id：20005，权重：0.625
- 模特体型，id：23002，权重：1
- 胸部大小，id：23503，权重：0.32

那么，就需要通过如下代码来实现角色的变形：

.. code-block:: objc
   :linenos:
   
   NSArray* targetArr = @[
      @{@"id": @"20005", @"weight": 0.625},
      @{@"id": @"23002", @"weight": 1},
      @{@"id": @"23503", @"weight": 0.32}
   ];
   NSData* targetData = [NSJSONSerialization dataWithJSONObject:targetArr options:NSJSONWritingPrettyPrinted error:NULL];
   AWValue* targets = [AWValue valueOfJson:targetData];
   [character setConfigs:@{
      AWCharacterConfigKeyTargetArray: targets
   }];


让角色播放动画
^^^^^^^^^^^^^^^^^^^

角色的动画分肢体动画和口型动画，现分别介绍两种动画的播放。

肢体动画
"""""""""""""

若开发者取得了授权的肢体动画资源，就可以在 SDK 里使用这些动画，并作用在角色身上。现假设开发者的资源目录有如下结构：

::

   .
   ├── face
   |   ├── face1.jpg
   |   └── face1.target
   ├── dress
   |   ├── hair.zip
   |   ├── shirt.zip
   |   ├── pant.zip
   |   └── shoe.zip
   └── animation
       ├── anim1.zip
       └── anim2.zip

前面已经讨论过 ``face`` 和 ``dress`` 两个目录，这里不再赘述，而 ``animation`` 文件夹包含了两个肢体动画资源文件。

和肢体动画相关的键有：

- ``AWCharacterConfigKeyAnimation`` 动画本身
- ``AWCharacterConfigKeyAnimationLoop`` 动画是否循环，如果不循环，动画播放结束后会停留在最后一帧
- ``AWCharacterConfigKeyAnimationFade`` 在两个动画之间切换的过渡时间

我们的目标是先让角色播放 ``animation/anim1.zip``，动画结束后播放 ``animation/anim2.zip``，然后回到初始状态。

.. code-block:: objc
   :linenos:
   
   - (AWCharacter *)getCharacter
   {
      static AWCharacter* character = NULL;
      if (character == NULL) {
         character = [AWCharacter new];
         character.delegate = self;
      }
      return character;
   }
   
   - (void)playAnimation:(NSString *)anim
   {
      AWCharacter* character = [self getCharacter];
      AWValue* animation;
      if (anim == null) {
         animation = [AWValue null];
      } else {
         animation = [AWValue valueOfString:anim];
      }
      [character setConfigs:@{
         AWCharacterConfigKeyAnimation: animation,
         AWCharacterConfigKeyAnimationLoop: [AWValue valueOfBool:NO],
         AWCharacterConfigKeyAnimationFade: [AWValue valueOfLong:300]
      }];
   }
   
   - (void)characterAnimationEnd:(NSString *_Nonnull)characterId animation:(AWValue *_Nonnull)animation
   {
      if ([[animation stringValue] isEqualToString:@"animation/anim1"]) {
         [self playAnimation:@"animation/anim2"];
      } else {
         [self playAnimation:null];
      }
   }
   
   - (void)start
   {
      [self playAnimation:@"animation/anim1"];
   }

代码从 ``- (void)start`` 开始执行，先播放 ``animation/anim1``，在动画结束的回调中，判断当前结束的动画为 ``animation/anim1``，于是播放 ``animation/anim2``；在 ``animation/anim2`` 动画结束的回调中，判断结束的动画为 ``animation/anim2``，于是回到初始状态（把值置为 ``[AWValue null]`` 会回到初始状态）。

值得注意的两点：

- 在 ``- (void)playAnimation:(NSString *)anim`` 方法中，我们设置了动画不循环，并且动画之间的切换时间为 300 毫秒。
- 指定动画资源的时候，需要把 ``.zip`` 后缀去掉。


口型动画
"""""""""""""
（待补充）

调整角色的位置和朝向
^^^^^^^^^^^

角色的位置指的是角色在三维空间中所处的坐标位置。角色若要在 ``renderView`` 被渲染出来，除了要配置好正确的加载步骤，还要指定角色的坐标位置，以及镜头的位置和朝向。默认情况下，角色处在 ``(0, 0, 0）``，即处在三维空间绝对坐标系（也称作 **世界坐标系**）下的原点位置上，主镜头在正 `z` 轴方向的位置上，面向角色。这就保证了角色在默认情况下能够被渲染到 ``renderView`` 上。

在镜头不变的情况下，通过调整角色在世界坐标系下的位置，可以使角色渲染在 ``renderView`` 的不同位置上。例如，

.. code-block:: objc
   :linenos:
   
   AWValue* position = [AWValue valueOfVector3:AWVector3Make(20, 0, 0);
   [character setConfigs:@{
      AWCharacterConfigKeyPosition: position
   }];
   
就表示将角色的世界坐标系位置设定为 ``(20, 0, 0)``。

除了可以设定角色的位置，还可以设定角色的朝向。朝向既可以用欧拉角表示，也可以用四元数表示。假设我们需要角色绕着 `y` 轴旋转 30 度，就可以用如下方式实现：

.. code-block:: objc
   :linenos:
   
   AWValue* rotation = [AWValue valueOfVector3:AWVector3Make(0, 30, 0);
   [character setConfigs:@{
      AWCharacterConfigKeyRotation: rotation
   }];


载入更多角色
^^^^^^^^^^^

前面我们通过 ``[AWCharacter new]`` 创建出来的角色配置对象，始终指向同一个默认角色。如果需要创建多个角色，就需要通过如下方法实现

.. code-block:: objc
   :linenos:
   
   // 创建默认角色
   AWCharacter* defaultCharacter = [AWCharacter new];
   [defaultCharacter setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget1,
      AWCharacterConfigKeyFaceTexture: faceTexture1,
      AWCharacterConfigKeyGender: gender1
   }];
   
   // 创建第二个角色，角色id可以任意指定
   AWCharacter* secondCharacter = [[AWCharacter alloc] initWithCharacterId:@"lily"];
   [secondCharacter setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget2,
      AWCharacterConfigKeyFaceTexture: faceTexture2,
      AWCharacterConfigKeyGender: gender2
   }];
   
   // 创建第三个角色，角色id可以任意指定
   AWCharacter* thirdCharacter = [[AWCharacter alloc] initWithCharacterId:@"lucy"];
   [thirdCharacter setConfigs:@{
      AWCharacterConfigKeyFaceTarget: faceTarget3,
      AWCharacterConfigKeyFaceTexture: faceTexture3,
      AWCharacterConfigKeyGender: gender3
   }];


AWCamera
~~~~~~~~~~~~~~~~

调整镜头的位置和朝向
^^^^^^^^^^^

和角色类似，镜头（``AWCamera``）也可以调整位置和朝向，用法和角色类似，例如

.. code-block:: objc
   :linenos:
   
   AWValue* position = [AWValue valueOfVector3:AWVector3Make(20, 0, 0);
   AWValue* rotation = [AWValue valueOfVector3:AWVector3Make(0, 30, 0);
   [camera setConfigs:@{
      AWCameraConfigKeyPosition: position,
      AWCameraConfigKeyRotation: rotation
   }];

为了更方便地处理旋转，镜头还支持始终盯着世界坐标系下的一个位置点，可通过 ``AWCameraConfigKeyLookAt`` 这个键来实现。 



开启多镜头
^^^^^^^^^^^

和创建多角色类似，我们也可以创建多镜头。默认的镜头是主镜头，不可移除。可以通过如下方式新增一个特写镜头

.. code-block:: objc
   :linenos:
   
   // 新增一个特写镜头
   AWCamera* closeupCamera = [[AWCamera alloc] initWithCameraId:@"closeup"];
   [closeupCamera setConfigs:@{
      AWCameraConfigKeyIndex: [AWValue valueOfInt:1],
      AWCameraConfigKeyViewport: [AWValue valueOfRect:AWRectMake(0, 0, 320, 180)],
      AWCameraConfigKeyPosition: [AWValue valueOfVector3:AWVector3Make(0, 100, 180)],
   }];

在这个特写镜头里，我们需要指定特写镜头的 id 号。另外， ``AWCameraConfigKeyIndex`` 表示多个镜头在层叠过程中的排列顺序，值越大，镜头在屏幕中越靠外；``AWCameraConfigKeyViewport`` 表示镜头的视窗区域，即显示在 ``renderView`` 的指定区域中。

镜头的背景图
^^^^^^^^^^

可以通过 ``AWCameraConfigKeyBackImage`` 指定一张背景图显示在镜头所在的视窗中。默认采用“等比例充满”的方式在视窗中平铺背景图片。

AWPuppet
~~~~~~~~~~~~~~~~~

（待补充）

AWRecorder
~~~~~~~~~~~~~~~~~

AWRecorder 提供了截屏和生成 GIF 的功能。

截屏
^^^^^^^^

截屏提供了两个接口，分别是：

.. code-block:: objc
   :linenos:

   /**
    * @brief 截取整个屏幕的内容。
    */
   - (void)takeScreenShot;

   /**
    * @brief 截取屏幕指定区域的内容。
    * @param rect 指定屏幕的渲染区域，单位是像素。
    */
   - (void)takeScreenShot:(AWRect)rect;


截屏是个异步操作，截屏的结果可以通过实现 ``AWRecorderDelegate`` 协议的如下若干方法来获得

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 开始截屏的回调
    */
   - (void)screenShotStart;

   /**
    * @brief 结束截屏的回调
    */
   - (void)screenShotEnd:(UIImage *_Nonnull)screenShot;

   /**
    * @brief 截屏失败的回调
    * @param error 错误信息
    */
   - (void)screenShotFailed:(NSError * _Nonnull)error;


生成 GIF
^^^^^^^^^^

（待补充）


AWQuery
~~~~~~~~~~~~~~~~~

AWQuery 提供了异步查询引擎内部相关信息的机制。每次查询都需要指定本次查询的 ``queryId``，用于标识查询。查询的结果可以通过实现 ``AWQueryDelegate`` 协议方法获得：

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 查询操作的回调
    * @param result 查询的结果
    * @param queryId 查询的标识id
    */
   -(void)onGetQueryResult:(NSDictionary *_Nonnull)result
                   queryId:(NSString *_Nonnull)queryId;


当 ``result`` 的结果是空的时候，说明没查询到任何信息，说明这是一次无效的查询。


查询角色信息
^^^^^^^^^^^

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 查询角色信息
    * @param keys 角色信息的关键字，例如AWCharacterConfigKeyGender, AWCharacterConfigKeyPosition等
    * @param characterId 角色的唯一标识
    * @param queryId 本次查询的标识id
    */
   - (void)queryCharacterInfo:(NSArray<NSString *> *_Nonnull)keys
                  characterId:(NSString *_Nonnull)characterId
                      queryId:(NSString *_Nonnull)queryId;
                   

查询镜头信息
^^^^^^^^^^^

.. code-block:: objc
   :linenos:

   /**
    * @brief 查询主镜头的信息
    * @param keys 角色信息的关键字，例如AWCameraConfigKeyPosition, AWCameraConfigKeyRotation等
    * @param queryId 本次查询的标识id
    */
   - (void)queryCameraInfo:(NSArray<NSString *> *_Nonnull)keys
                   queryId:(NSString *_Nonnull)queryId;

   /**
    * @brief 查询指定镜头的信息
    * @param keys 角色信息的关键字，例如AWCameraConfigKeyPosition, AWCameraConfigKeyRotation等
    * @param cameraId 镜头的唯一标识
    * @param queryId 本次查询的标识id
    */
   - (void)queryCameraInfo:(NSArray<NSString *> *_Nonnull)keys
                  cameraId:(NSString *_Nonnull)cameraId
                   queryId:(NSString *_Nonnull)queryId;
                   

查询屏幕坐标点落在角色部位上的信息
^^^^^^^^^^^

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 查询主镜头下，屏幕坐标点是否落在指定角色身上的某个部位
    * @param screenPoint 屏幕的坐标点，单位是像素
    * @param characterId 角色的唯一标识
    * @param queryId 本次查询的标识id
    */
   - (void)queryCharacterPickUp:(AWVector2)screenPoint
                    characterId:(NSString *_Nonnull)characterId
                        queryId:(NSString *_Nonnull)queryId;

   /**
    * @brief 查询指定镜头下，屏幕坐标点是否落在指定角色身上的某个部位
    * @param screenPoint 屏幕的坐标点，单位是像素
    * @param characterId 角色的唯一标识
    * @param cameraId 镜头的唯一标识
    * @param queryId 本次查询的标识id
    */
   - (void)queryCharacterPickUp:(AWVector2)screenPoint
                    characterId:(NSString *_Nonnull)characterId
                       cameraId:(NSString *_Nonnull)cameraId
                        queryId:(NSString *_Nonnull)queryId;


查询坐标变换
^^^^^^^^^^^

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 查询在主镜头下，三维世界坐标（World）中的点映射到屏幕（Screen）中的坐标值
    * @param worldPoint 三维世界坐标值
    * @param queryId 本次查询的标识id
    */
   - (void)queryW2SPoint:(AWVector3)worldPoint
                 queryId:(NSString *_Nonnull)queryId;

   /**
    * @brief 查询在指定镜头下，三维世界坐标（World）中的点映射到屏幕（Screen）中的坐标值
    * @param worldPoint 三维世界坐标值
    * @param cameraId 镜头的唯一标识
    * @param queryId 本次查询的标识id
    */
   - (void)queryW2SPoint:(AWVector3)worldPoint
                cameraId:(NSString *_Nonnull)cameraId
                 queryId:(NSString *_Nonnull)queryId;

查询角色身体骨骼点信息
^^^^^^^^^^^

.. code-block:: objc
   :linenos:
   
   /**
    * @brief 查询指定角色的身体骨骼点信息
    * @param boneName 骨骼名称，例如head, spine等
    * @param characterId 角色的唯一标识
    * @param queryId 本次查询的标识id
   */
   - (void)queryCharacterBone:(NSString *_Nonnull)boneName
                  characterId:(NSString *_Nonnull)characterId
                      queryId:(NSString *_Nonnull)queryId;

其中 ``boneName`` 可以从这两张图中查询到：

.. image:: /_static/img/身体骨骼名称.jpg

.. image:: /_static/img/手掌骨骼名称.jpg

AWResourceManager
~~~~~~~~~~~~~~~~~
   
AWResourceManager 作为 SDK 的资源管理器，可以设置缓存路径、添加多个资源目录（可设置路径资源被搜索到的优先级）和释放资源等操作。

- 引擎加载成功后的第一件事情就应该通过 ``setCacheDirectory:`` 设置缓存路径。**缓存路径只有一个，里面的内容在SDK执行期间严禁做清除操作，否则可能会出现渲染错误。** 

- 为了让 SDK 使用资源，还必须通过 ``addResourceDirectory:`` 或 ``addResourceDirectory:withPriority`` 添加资源路径。尽管下面这句话看起来像是一句废话，但还是务必请开发者注意：**SDK 在使用某个资源之前，该资源必须存在于某个资源路径下。**

- 一般情况下，开发者可不需要理会 ``setBaseDirectory:`` 这个方法。但对于需要将基础资源包和可执行文件分离的情况，开发者应该调用 ``setBaseDirectory:`` 来指定基础资源包的路径。 

- 为了加快程序的执行，SDK 默认会把曾经加载过的资源缓存到内存中。开发者可以随时通过调用 ``releaseResources`` 释放掉所有当前可释放的资源。




   
   
   
   
   
   
   
   
   
