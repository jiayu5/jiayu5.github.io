# 构建首个应用

> 创建“Hello World!”项目并运行。

1. Android 应用都是将各种可单独调用的组件加以组合构建而成。
1. Android 允许为不同的设备提供不同的资源。



## 1. 创建Android项目



### 1.1 Android Studio安装

1. 安装网址：https://www.androiddevtools.cn/#android-studio

2. maven国内源：https://help.aliyun.com/document_detail



### 1.2  关键文件位置

**app > java > com.example.xxx > MainActivity**

这是主 activity。它是应用的入口点。当您构建和运行应用时，系统会启动此 `Activity`的实例并加载其布局。

**app > res > layout > activity_main.xml**

此 XML 文件定义了 activity 界面 (UI) 的布局。它包含一个 `TextView` 元素，其中具有“Hello, World!”文本

**app > manifests > AndroidManifest.xml**

[清单文件](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=zh-cn)描述了应用的基本特性并定义了每个应用组件。

**Gradle Scripts > build.gradle**

有两个使用此名称的文件：一个针对项目“Project: My First App”，另一个针对应用模块“Module: My_First_App.app”。每个模块均有自己的 `build.gradle` 文件，但此项目当前仅有一个模块。使用每个模块的 `build.gradle` 文件控制 [Gradle 插件](https://developer.android.com/studio/releases/gradle-plugin?hl=zh-cn)构建应用的方式。如需详细了解此文件，请参阅[配置 build](https://developer.android.com/studio/build?hl=zh-cn#module-level)。



## 2. 在模拟器上运行

1. 创建一个Android虚拟设备，设备管理器 -> Create Device；
2. 点击 **Run** 图标；



## 3. 构建简单的界面

Android 应用的界面 (UI) 以布局和微件的层次结构形式构建而成。布局是 [`ViewGroup`](https://developer.android.com/reference/android/view/ViewGroup?hl=zh-cn) 对象，即控制其子视图在屏幕上的放置方式的容器。微件是 [`View`](https://developer.android.com/reference/android/view/View?hl=zh-cn)对象，即按钮和文本框等界面组件。

![viewgroup_2x](picture/viewgroup_2x.png)



### 3.1 布局编辑

> `ConstraintLayout` 是一种布局，它根据同级视图和父布局的约束条件定义每个视图的位置。这样一来，使用扁平视图层次结构既可以创建简单布局，又可以创建复杂布局。

1. 添加布局约束条件；
2. 添加按钮；



### 3.2 更改界面字符串

**app > res > values > strings.xml**

这是一个[字符串资源](https://developer.android.com/guide/topics/resources/string-resource?hl=zh-cn)文件，可在此文件中指定所有界面字符串。可以利用该文件在一个位置管理所有界面字符串，使字符串的查找、更新和本地化变得更加容易。



## 4. 启动另一个activity

1. 在MainActivity文件中，添加方法桩；
2. `Intent` 是在相互独立的组件（如两个 activity）之间提供运行时绑定功能的对象。`Intent` 表示应用执行某项操作的意图；

3. 创建新activity并添加视图；



# 应用基础知识

> Android 系统设计的独特之处在于，任何应用都可启动其他应用的组件。
>
> 当系统启动某个组件时，它会启动该应用的进程（如果尚未运行），并实例化该组件所需的类。
>
> 由于系统在单独的进程中运行每个应用，且其文件权限会限制对其他应用的访问，因此您的应用无法直接启动其他应用中的组件，但 Android 系统可以。



## 1. Android应用运行机制

- Android 操作系统是一种多用户 Linux 系统，其中的每个应用都是一个不同的用户；
- 默认情况下，系统会为每个应用分配一个唯一的 Linux 用户 ID（该 ID 仅由系统使用，应用并不知晓）。系统会为应用中的所有文件设置权限，使得只有分配给该应用的用户 ID 才能访问这些文件； 
- 每个进程都拥有自己的虚拟机 (VM)，因此应用代码独立于其他应用而运行。
- 默认情况下，每个应用都在其自己的 Linux 进程内运行。Android 系统会在需要执行任何应用组件时启动该进程，然后当不再需要该进程或系统必须为其他应用恢复内存时，其便会关闭该进程。



## 2. 应用组件

> 应用组件是Android应用的基本构建块。
>
> - Activity
> - 服务
> - 广播接收器
> - 内容提供程序



### 2.1 Activity

> *Activity* 是与用户交互的入口点。它表示拥有界面的单个屏幕。

Activity可完成系统和应用程序之间的以下交互：

- 追踪用户当前关心的内容（屏幕上显示的内容），以确保系统继续运行托管 Activity 的进程。
- 了解先前使用的进程包含用户可能返回的内容（已停止的 Activity），从而更优先保留这些进程。
- 帮助应用处理终止其进程的情况，以便用户可以返回已恢复其先前状态的 Activity。
- 提供一种途径，让应用实现彼此之间的用户流，并让系统协调这些用户流。（此处最经典的示例是共享。）



### 2.2 服务

> *服务*是一个通用入口点，用于因各种原因使应用在后台保持运行状态。它是一种在后台运行的组件，用于执行长时间运行的操作或为远程进程执行作业。服务不提供界面。

- 音乐播放是用户可直接感知的服务，因此，应用会向用户发送通知，表明其希望成为前台，从而告诉系统此消息；在此情况下，系统明白它应尽全力维持该服务进程运行，因为进程消失会令用户感到不快。
- 通常，用户不会意识到常规后台服务正处于运行状态，因此系统可以更自由地管理其进程。如果系统需要使用 RAM 来处理用户更迫切关注的内容，则其可能允许终止服务（然后在稍后的某个时刻重启服务）。



### 2.3 广播接收器

> 借助*广播接收器*组件，系统能够在常规用户流之外向应用传递事件，从而允许应用响应系统范围内的广播通知



### 2.4 内容提供程序

> *内容提供程序*管理一组共享的应用数据，您可以将这些数据存储在文件系统、SQLite 数据库、网络中或者您的应用可访问的任何其他持久化存储位置。其他应用可通过内容提供程序查询或修改数据（如果内容提供程序允许）。

- 分配 URI 无需应用保持运行状态，因此 URI 可在其所属的应用退出后继续保留。当系统必须从相应的 URI 检索应用数据时，系统只需确保所属应用仍处于运行状态。
- 这些 URI 还会提供重要的细粒度安全模型。例如，应用可将其所拥有图像的 URI 放到剪贴板上，但将其内容提供程序锁定，以便其他应用程序无法随意访问它。当第二个应用尝试访问剪贴板上的 URI 时，系统可允许该应用通过临时的 *URI 授权*来访问数据，这样便只能访问 URI 后面的数据，而非第二个应用中的其他任何内容。



### 2.5 启动组件

在四种组件类型中，有三种（Activity、服务和广播接收器）均通过异步消息 *Intent* 进行启动。Intent 会在运行时对各个组件进行互相绑定。

1. 对于 Activity 和服务，Intent 会定义要执行的操作（例如，*查看*或*发送*某内容），并且可指定待操作数据的 URI，以及正在启动的组件可能需要了解的信息。
2. 对于广播接收器，Intent 只会定义待广播的通知。
3. 与 Activity、服务和广播接收器不同，内容提供程序并非由 Intent 启动。



**不同组件的启动方法：**

- 如要启动 Activity，可以向 `startActivity()` 或 `startActivityForResult()` 传递 `Intent`（当想让 Activity 返回结果时），或者为其安排新任务。
- 在 Android 5.0（API 级别 21）及更高版本中，可以使用 `JobScheduler` 类来调度操作。对于早期 Android 版本，可以通过向 `startService()` 传递 `Intent` 来启动服务（或对执行中的服务下达新指令）。也可通过向将 `bindService()` 传递 `Intent` 来绑定到该服务。 
- 可以通过向 `sendBroadcast()`、`sendOrderedBroadcast()` 或 `sendStickyBroadcast()` 等方法传递 `Intent` 来发起广播。
- 可以通过在 `ContentResolver` 上调用 `query()`，对内容提供程序执行查询。
