## 基础

### 1. Activity

#### 主要的生命周期有哪些？

* onCreate()：表示Activity正在被创建，常用来初始化工作，比如调用setContentView加载界面布局资源，初始化Activity所需数据等；
* onRestart()：表示Activity正在重新启动，一般情况下，当前Acitivty从不可见重新变为可见时，OnRestart就会被调用；
* onStart()：表示Activity正在被启动，此时Activity可见但不在前台，还处于后台，无法与用户交互；
* onResume()：表示Activity获得焦点，此时Activity可见且在前台并开始活动，这是与onStart的区别所在；
* onPause()：表示Activity正在停止，此时可做一些存储数据、停止动画等工作，但是不能太耗时，因为这会影响到新Activity的显示，onPause必须先执行完，新Activity的onResume才会执行；
* onStop()：表示Activity即将停止，可以做一些稍微重量级的回收工作，比如注销广播接收器、关闭网络连接等，同样不能太耗时；
* onDestroy()：表示Activity即将被销毁，这是Activity生命周期中的最后一个回调，常做回收工作、资源释放；

从整个生命周期来看，onCreate和onDestroy是配对的，分别标识着Activity的创建和销毁，并且只可能有一次调用；  
从Activity是否可见来说，onStart和onStop是配对的，这两个方法可能被调用多次；  
从Activity是否在前台来说，onResume和onPause是配对的，这两个方法可能被调用多次；  
除了这种区别，在实际使用中没有其他明显区别；

![](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)

#### Activity A 启动另一个Activity B 会调用哪些方法？如果B是透明主题的又或则是个DialogActivity呢？

* Activity A 的onPause() → Activity B的onCreate() → onStart() → onResume() → Activity A的onStop()；
* 如果B是透明主题又或则是个DialogActivity，则不会回调A的onStop；

#### 横竖屏切换时 Activity 的生命周期

* 不设置Activity的android:configChanges时，切屏会销毁当前Activity，然后重新加载调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次；
onPause() →onStop()→onDestory()→onCreate()→onStart()→onResume()
* 设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次；
* 设置Activity的android:configChanges="orientation|keyboardHidden|screenSize"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法；

#### 前台切换到后台，然后再回到前台时 Activity 的生命周期
#### 弹出 Dialog 的时候按 Home 键时 Activity 的生命周期
#### 两个 Activity 之间跳转时的生命周期
#### 下拉状态栏时 Activity 的生命周期

#### Activity 状态保存与恢复

发生条件：异常情况下（系统配置发生改变时导致Activity被杀死并重新创建、资源内存不足导致低优先级的Activity被杀死）

* 系统会调用onSaveInstanceState来保存当前Activity的状态，此方法调用在onStop之前，与onPause没有既定的时序关系；
* 当Activity被重建后，系统会调用onRestoreInstanceState，并且把onSave(简称)方法所保存的Bundle对象同时传参给onRestore(简称)和onCreate()，因此可以通过这两个方法判断Activity是否被重建；

#### Activity 的四种 LaunchMode（启动模式）的区别

使用android:launchMode="standard|singleInstance|singleTask|singleTop"来控制Acivity任务栈。

任务栈是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其onDestory()方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名。

* standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(onCreate()->onStart()->onResume())都会执行。
* singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的onNewIntent()方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与standard模式一样.
* singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,onNewIntent(),并且singleTask会清理在当前Activity上面的所有Activity.(clear top)
* singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

#### Acitivty的启动流程

调用startActivity()后经过重重方法会转移到ActivityManagerService的startActivity()，并通过一个IPC回到ActivityThread的内部类ApplicationThread中，并调用其scheduleLaunchActivity()将启动Activity的消息发送并交由Handler H处理。Handler H对消息的处理会调用handleLaunchActivity()→performLaunchActivity()得以完成Activity对象的创建和启动；

>[Android四大组件启动机制之Activity启动过程](https://blog.csdn.net/qq_30379689/article/details/79611217)

### 2. Fragment

#### 生命周期

Fragment从创建到销毁整个生命周期中涉及到的方法依次为：onAttach()→onCreate()→ onCreateView()→onActivityCreated()→onStart()→onResume()→onPause()→onStop()→onDestroyView()→onDestroy()→onDetach()，其中和Activity有不少名称相同作用相似的方法，而不同的方法有:

* onAttach()：当Fragment和Activity建立关联时调用；
* onCreateView()：当fragment创建视图调用，在onCreate之后；
* onActivityCreated()：当与Fragment相关联的Activity完成onCreate()之后调用；
* onDestroyView()：在Fragment中的布局被移除时调用；
* onDetach()：当Fragment和Activity解除关联时调用；

![](https://developer.android.google.cn/images/fragment_lifecycle.png)

#### Fragment 各种情况下的生命周期
#### Activity 与 Fragment 之间生命周期比较

![](https://developer.android.google.cn/images/activity_fragment_lifecycle.png)

#### Activity 和 Fragment 之间怎么通信， Fragment 和 Fragment 怎么通信

Activity 传值给 Fragment：通过 Bundle 对象来传递，Activity 中构造 bundle 数据包，调用 Fragment 对象的 setArguments(Bundle b) 方法，Fragment 中使用 getArguments() 方法获取 Activity 传递过来的数据包取值。

Fragment 传值给 Activity：在 Fragment 中定义一个内部回调接口，Activity 实现该回调接口， Fragment 中获取 Activity 的引用，调用 Activity 实现的业务方法。接口回调机制式 Java 不同对象之间数据交互的通用方法。

Fragment 传值给 Fragment：一个 Fragment 通过 Activity 获取到另外一个 Fragment 直接调用方法传值。

#### 谈谈Activity和Fragment的区别？

相似点：都可包含布局、可有自己的生命周期
不同点：
* Fragment相比较于Activity多出4个回调周期，在控制操作上更灵活；
* Fragment可以在XML文件中直接进行写入，也可以在Activity中动态添加；
* Fragment可以使用show()/hide()或者replace()随时对Fragment进行切换，并且切换的时候不会出现明显的效果，用户体验会好；Activity虽然也可以进行切换，但是Activity之间切换会有明显的翻页或者其他的效果，在小部分内容的切换上给用户的感觉不是很好；

#### Fragment中add与replace的区别（Fragment重叠）

* add不会重新初始化fragment，replace每次都会。所以如果在fragment生命周期内获取获取数据,使用replace会重复获取；
* 添加相同的fragment时，replace不会有任何变化，add会报IllegalStateException异常；
* replace先remove掉相同id的所有fragment，然后在add当前的这个fragment，而add是覆盖前一个fragment。所以如果使用add一般会伴随hide()和show()，避免布局重叠；
* 使用add，如果应用放在后台，或以其他方式被系统销毁，再打开时，hide()中引用的fragment会销毁，所以依然会出现布局重叠bug，可以使用replace或使用add时，添加一个tag参数；

#### getFragmentManager、getSupportFragmentManager 、getChildFragmentManager之间的区别？

* getFragmentManager()所得到的是所在fragment 的父容器的管理器，getChildFragmentManager()所得到的是在fragment  里面子容器的管理器，如果是fragment嵌套fragment，那么就需要利用getChildFragmentManager()；
* 因为Fragment是3.0 Android系统API版本才出现的组件，所以3.0以上系统可以直接调用getFragmentManager()来获取FragmentManager()对象，而3.0以下则需要调用getSupportFragmentManager() 来间接获取；

#### FragmentPagerAdapter与FragmentStatePagerAdapter的区别与使用场景?

相同点：二者都继承PagerAdapter  
不同点：
* FragmentPagerAdapter的每个Fragment会持久的保存在FragmentManager中，只要用户可以返回到页面中，它都不会被销毁。因此适用于那些数据相对静态的页，Fragment数量也比较少的那种；
* FragmentStatePagerAdapter只保留当前页面，当页面不可见时，该Fragment就会被消除，释放其资源。因此适用于那些数据动态性较大、占用内存较多，多Fragment的情况；

### 7. Service
    
#### Service 的生命周期

* onCreate()：如果service没被创建过，调用startService()后会执行onCreate()回调；如果service已处于运行中，调用startService()不会执行onCreate()方法。也就是说，onCreate()只会在第一次创建service时候调用，多次执行startService()不会重复调用onCreate()，此方法适合完成一些初始化工作；
* onStartComand()：服务启动时调用，此方法适合完成一些数据加载工作，比如会在此处创建一个线程用于下载数据或播放音乐；
* onBind()：服务被绑定时调用；
* onUnBind()：服务被解绑时调用；
* onDestroy()：服务停止时调用；

![](https://developer.android.google.cn/images/service_lifecycle.png)

#### Service 的启动方式

* startService()：通过这种方式调用startService，onCreate()只会被调用一次，多次调用startSercie会多次执行onStartCommand()和onStart()方法。如果外部没有调用stopService()或stopSelf()方法，service会一直运行。
* bindService()：如果该服务之前还没创建，系统回调顺序为onCreate()→onBind()。如果调用bindService()方法前服务已经被绑定，多次调用bindService()方法不会多次创建服务及绑定。如果调用者希望与正在绑定的服务解除绑定，可以调用unbindService()方法，回调顺序为onUnbind()→onDestroy()；

> [Android Service两种启动方式详解（总结版）](https://www.jianshu.com/p/4c798c91a613)

#### Service保活

* onStartCommand方法中，返回START_STICKY或则START_REDELIVER_INTENT
  * START_STICKY：如果返回START_STICKY，表示Service运行的进程被Android系统强制杀掉之后，Android系统会将该Service依然设置为started状态（即运行状态），但是不再保存onStartCommand方法传入的intent对象
  * START_NOT_STICKY：如果返回START_NOT_STICKY，表示当Service运行的进程被Android系统强制杀掉之后，不会重新创建该Service
  * START_REDELIVER_INTENT：如果返回START_REDELIVER_INTENT，其返回情况与START_STICKY类似，但不同的是系统会保留最后一次传入onStartCommand方法中的Intent再次保留下来并再次传入到重新创建后的Service的onStartCommand方法中
* 提高Service的优先级  
  在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播；
* 在onDestroy方法里重启Service  
  当service走到onDestroy()时，发送一个自定义广播，当收到广播时，重新启动service；
* 提升Service进程的优先级  
  进程优先级由高到低：前台进程 一 可视进程 一 服务进程 一 后台进程 一 空进程 可以使用startForeground将service放到前台状态，这样低内存时，被杀死的概率会低一些；
* 系统广播监听Service状态
* 将APK安装到/system/app，变身为系统级应用

#### Service 与 IntentService 的区别
#### Service 和 Activity 之间的通信方式
  * 通过 Binder 对象
  * 通过 Broadcast（广播）的形式

### 8.  ContentProvider

####  对 ContentProvider 的理解

ContentProvider作为四大组件之一，其主要负责存储和共享数据。与文件存储、SharedPreferences存储、SQLite数据库存储这几种数据存储方法不同的是，后者保存下的数据只能被该应用程序使用，而前者可以让不同应用程序之间进行数据共享，它还可以选择只对哪一部分数据进行共享，从而保证程序中的隐私数据不会有泄漏风险。

#### ContentProvider、ContentResolver、ContentObserver 之间的关系

* ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。
* ContentResolver：ContentResolver可以为不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。
* ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。

#### ContentProvider 是如何实现数据共享的？
#### ContentProvider 的权限管理？

* 读写分离
* 权限控制-精确到表级
* URL控制

#### Android 系统为什么会设计 ContentProvider？

### 9. BroadcastReceiver

>[Android四大组件：BroadcastReceiver史上最全面解析](https://www.jianshu.com/p/ca3d87a4cdf3)

#### 广播的分类？使用方式和场景

* 普通广播：开发者自身定义 intent的广播（最常用），所有的广播接收器几乎会在同一时刻接受到此广播信息，接受的先后顺序随机；
* 有序广播：发送出去的广播被广播接收者按照先后顺序接收，同一时刻只会有一个广播接收器能够收到这条广播消息，当这个广播接收器中的逻辑执行完毕后，广播才会继续传递，且优先级（priority）高的广播接收器会先收到广播消息。有序广播可以被接收器截断使得后面的接收器无法收到它；
* 本地广播：仅在自己的应用内发送接收广播，也就是只有自己的应用能收到，数据更加安全，效率更高，但只能采用动态注册的方式；
* 粘性广播：这种广播会一直滞留，当有匹配该广播的接收器被注册后，该接收器就会收到此条广播；

#### 动态广播和静态广播有什么区别

* 使用方式：静态广播在AndroidManifest.xml中通过<receive>标签声明；动态广播在代码中调用Context.registerReceiver()方法使用；
* 特点：静态广播不受任何组件的生命周期的影响；动态广播跟随组件的生命周期变化；
* 应用场景：需要时刻监听广播就用静态广播；只需要在特定时刻监听广播就用动态广播；

#### 广播发送和接收的原理了解吗 ？（Binder机制、AMS）

> [安卓广播的底层实现原理](https://www.jianshu.com/p/02085150339c)

### 10. AlertDialog、popupWindow、Activity 之间的区别

引申：Activity、Window、Drawable、View的区别

Activity 创建时通过attach()初始化了一个 Window 也就是 PhoneWindow，一个 PhoneWindow 持有一个 DecorView 的实例，DecorView 本身是一个 FrameLayout，继承于View，Activty通过setContentView将xml布局控件不断addView()添加到View中，最终显示到Window与我们交互；

> [Activity、View、Window的理解一篇文章就够了](https://blog.csdn.net/zane402075316/article/details/69822438)

### 11. 数据存储

#### 描述一下Android数据持久存储方式？

* SharedPreferences存储：一种轻型的数据存储方式，本质是基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息（如应用程序的各种配置信息）；
* SQLite数据库存储：一种轻量级嵌入式数据库引擎，它的运算速度非常快，占用资源很少，常用来存储大量复杂的关系数据；
* ContentProvider：四大组件之一，用于数据的存储和共享，不仅可以让不同应用程序之间进行数据共享，还可以选择只对哪一部分数据进行共享，可保证程序中的隐私数据不会有泄漏风险；
* File文件存储：写入和读取文件的方法和 Java中实现I/O的程序一样；
* 网络存储：主要在远程的服务器中存储相关数据，用户操作的相关数据可以同步到服务器上；

#### SharedPreferences的应用场景？注意事项？

* SharedPreferences是一种轻型的数据存储方式，本质是基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息，如int，String，boolean、float和long；
* 注意事项：
  * 勿存储大型复杂数据，这会引起内存GC、阻塞主线程使页面卡顿产生ANR
  * 勿在多进程模式下，操作Sp
  * 不要多次edit和apply，尽量批量修改一次提交
  * 建议apply，少用commit

> [史上最全面，清晰的SharedPreferences解析](https://blog.csdn.net/geekerhw/article/details/79713068)  
> [SharedPreferences在多进程中的使用及注意事项](http://zmywly8866.github.io/2015/09/09/sharedpreferences-in-multiprocess.html)

#### SharedPrefrences的apply和commit有什么区别？

* apply没有返回值而commit返回boolean表明修改是否提交成功。
* apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。
* apply方法不会提示任何失败的提示。 由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。

#### 了解SQLite中的事务操作吗？是如何做的？

SQLite在做CRDU操作时都默认开启了事务，然后把SQL语句翻译成对应的SQLiteStatement并调用其相应的CRUD方法，此时整个操作还是在rollback journal这个临时文件上进行，只有操作顺利完成才会更新db数据库，否则会被回滚。

#### 使用SQLite做批量操作有什么好的方法吗？

### 11. Application 和 Activity 的 Context 之间的区别

[Android Context 上下文 你必须知道的一切](https://blog.csdn.net/lmj623565791/article/details/40481055)

### 12. 动画
### 13. 对 SurfaceView 的了解
### 14. Serializable 和 Parcelable 的区别
### 15. Android 中数据存储方式有哪些
### 16. 屏幕适配的处理技巧都有哪些
### 17. Android 各个版本 API 的区别
### 18. 动态权限适配方案，权限组的概念

* 权限管理系统（底层的权限是如何进行 grant 的）

### 19. 为什么不能在子线程更新 UI
### 20. ListView 图片加载错乱的原理和解决方案
### 21. RecycleView

* 对 RecycleView 的了解
* Recycleview 和 ListView 的区别
* RecycleView 实现原理
  
### 22. Android Manifest 的作用与理解
### 23. 多线程在 Android 中的使用
### 24. 低版本 SDK 如何使用高版本 API
### 25. AsyncTask 机制、原理及不足
### 26. 如果在 onStop() 的时候做了网络请求，onResume() 的时候怎么恢复
### 27. Handler

* Handler 机制和底层实现
* Handler的延迟消息是如何实现的
* Handler、Thread、HandlerThread 区别

### 28. ThreadLocal 原理、实现及如何保证 Local 属性
### 29. 自定义 View

* ViewRoot：连接WindowManager(外界访问Window的入口)和DecorView（顶级View）的纽带，View的三大流程均是通过ViewRoot来完成的。
* DecorView：DecorView是顶级View，本质就是一个FrameLayout包含了两个部分，标题栏和内容栏。内容栏id是content，也就是activity中setContentView所设置的部分，最终将布局添加到id为content的FrameLayout中

#### View的绘制流程

View的绘制流程是从ViewRoot的PerformTraversals方法开始的。

performTraversals会依次调用performMeasure, performLayout, performDraw三个方法，这三个方法分别完成顶层View的measure,layout,draw方法，onMeasure又会调用所有子元素的measure过程，直到完成整个View树的遍历。同理，performLayout, performDraw的传递流程与performMeasure相似。唯一不同在于，performDraw的传递过程在draw方法中通过dispatchDraw实现，但没有本质区别。

Measure过程后可以调用getMeasureWidth和getMeasureHeight方法获取View测量后的宽高，与getWidth和getHeight的区别是：getMeasuredHeight()返回的是原始测量高度，与屏幕无关，getHeight()返回的是在屏幕上显示的高度。实际上在当屏幕可以包裹内容的时候，他们的值是相等的，只有当view超出屏幕后，才能看出他们的区别。当超出屏幕后，getMeasuredHeight()等于getHeight()加上屏幕之外没有显示的高度。

Layout过程确定View四个顶点的位置和实际的宽高。

Draw过程确定View的显示，只有draw方法完成后View的内容才会出现在屏幕上。

* 自定义 View 的流程？如何机型适配？
* 自定义 View 的时怎么获取 View 的大小？
* View 的事件传递分发机制？
####  requestLayout()，onLayout()，onDraw()，drawChild() 区别与联系？
#### invalidate() 和 postInvalidate() 以及 requestLayout() 的区别？

invalidate()和postInvalidate()都会递归调用父View的invalidateChildInParent()方法，直到调用ViewRootImpl的invalidateChildInParent()方法，最终触发ViewRootImpl的performTraversals()方法，此时mLayoutRequestede为false，不会触发onMesaure()与onLayout()方法，有可能会触发onDraw()方法。

共同点：都是调用onDraw()方法，然后去达到重绘view的目的。

区别：invalidate()用于主线程，postInvalidate()用于子线程。

requestLayout()也会递归调用父窗口的requestLayout()方法，直到触发ViewRootImpl的performTraversals()方法，此时mLayoutRequestede为true，会触发onMesaure()与onLayout()方法，不一定会触发onDraw()方法。

当view确定自身已经不再适合现有的区域时，该view本身调用这个方法要求parent view(父类的视图)重新调用他的onMeasure、onLayout来重新设置自己位置。特别是当view的layoutparameter发生改变，并且它的值还没能应用到view上时，这时候适合调用这个方法requestLayout()。requestLayout调用onMeasure和onLayout，不一定调用onDraw()。

#### requestLayout()何时不会触发onDraw()？

如果没有改变控件的leftrighttopbottom就不会触发onDraw()，即没有改变View自身的大小。

#### invalidate()在什么情况下不会触发onDraw()？

在ViewGroup中，invalidate()默认不重新绘制子view。

#### 如何让ViewGroup在invalidate()时会触发onDraw()？

本质需要将ViewGroup的dirtyOpaque设置为false。

* 在构造函数中调用setWillNotDraw(false);
* 给ViewGroup设置背景。调用setBackground。

#### 如何计算一个 View 的嵌套层级？
####  onDraw()的顺序

![绘制过程](https://upload-images.jianshu.io/upload_images/2354038-dec64dea3ad60a0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/477/format/webp)

![绘制顺序](https://upload-images.jianshu.io/upload_images/2354038-df280209213f573c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/479/format/webp)

* 在 ViewGroup 的子类中重写除 dispatchDraw() 以外的绘制方法时，可能需要调用 setWillNotDraw(false)（出于效率的考虑，ViewGroup 默认会绕过 draw() 方法，换而直接执行 dispatchDraw() 以此来简化绘制流程，ScrollView 等已调用过）
* 在重写的方法有多个选择时，优先选择 onDraw()（Andorid优化了无需绘制时自动跳过onDraw的重复执行，以提升开发效率）

#### 双指缩放的实现？



### 30. 进程和 Application 的生命周期的关系？
### 31. SpareArray 的实现原理？Android中其他优化集合的实现？
### 32. SharedPreferences 的实现原理？是否进程同步？如何做到同步？
### 33. Android 线程有没有上限
### 35. 图片

* 对于图片显示：根据需要显示图片控件的大小对图片进行压缩显示。
* 如果图片数量非常多：则会使用LruCache等缓存机制，将所有图片占据的内容维持在一个范围内
* 单个图片非常巨大，并且还不允许压缩

* 对 Bitmap 对象的了解
* 图片加载原理？
* 图片压缩原理？
* 图片框架实现原理？LRUCache 原理？

### 自定义View

[安卓自定义View教程目录](http://www.gcssloop.com/customview/CustomViewIndex/)
[Android应用自定义View绘制方法手册](https://blog.csdn.net/yanbober/article/details/50577855)

### 36. 开源项目

####  EventBus 实现原理

[EventBus 3.0初探: 入门使用及其使用 完全解析](https://www.jianshu.com/p/acfe78296bb5)  
[EventBus 3.0进阶：源码及其设计模式 完全解析](https://www.jianshu.com/p/bda4ed3017ba)

* 三要素
  1. Event 事件。它可以是任意类型。
  2. Subscriber 事件订阅者。在EventBus3.0之前我们必须定义以onEvent开头的那几个方法，分别是onEvent、onEventMainThread、onEventBackgroundThread和onEventAsync，而在3.0之后事件处理的方法名可以随意取，不过需要加上注解@subscribe()，并且指定线程模型，默认是POSTING。
  3. Publisher 事件的发布者。我们可以在任意线程里发布事件，一般情况下，使用EventBus.getDefault()就可以得到一个EventBus对象，然后再调用post(Object)方法即可。
* 四种线程模型
  1. POSTING (默认) 表示事件处理函数的线程跟发布事件的线程在同一个线程。
  2. MAIN 表示事件处理函数的线程在主线程(UI)线程，因此在这里不能进行耗时操作。
  3. BACKGROUND 表示事件处理函数的线程在后台线程，因此不能进行UI操作。如果发布事件的线程是主线程(UI线程)，那么事件处理函数将会开启一个后台线程，如果果发布事件的线程是在后台线程，那么事件处理函数就使用该线程。
  4. ASYNC 表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作。

####  ButterKnife 实现原理
####  Volley 实现原理
####  okhttp 实现原理
   *  参数是如何组装的？

#### RxJava

RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext() （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：onCompleted() 和 onError()。

* onCompleted(): 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。
* onError(): 事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。
* 需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

#### Retrofit

### 37. 服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达
### 38. SQLite 数据库升级，数据迁移问题
### 39. 数据库框架对比和源码分析
### 40. CAS介绍，OAuth 授权机制
### 41. 谈谈你对安卓签名的理解
### 42. App 是如何沙箱化，为什么要这么做？

### 版本适配

#### 6.0

动态权限

#### 7.0

* 应用间共享文件

    > [Android 7.0 行为变更 通过FileProvider在应用间共享文件吧](https://blog.csdn.net/lmj623565791/article/details/72859156)

* APK signature scheme v2

  * 只勾选v1签名就是传统方案签署，但是在7.0上不会使用V2安全的验证方式。 
  * 只勾选V2签名7.0以下会显示未安装，7.0上则会使用了V2安全的验证方式。 
  * 同时勾选V1和V2则所有版本都没问题。

* 后台优化

    在Android 7.0中删除了三项隐式广播，以帮助优化内存使用和电量消耗。

  * 在 Android 7.0上 应用不会收到 CONNECTIVITY_ACTION 广播，即使你在manifest清单文件中设置了请求接受这些事件的通知。 但，在前台运行的应用如果使用BroadcastReceiver 请求接收通知，则仍可以在主线程中侦听 CONNECTIVITY_CHANGE。
  * 在 Android 7.0上应用无法发送或接收 ACTION_NEW_PICTURE 或ACTION_NEW_VIDEO 类型的广播。

* 多语言特性
* 通知栏适配

  * [Android通知栏介绍与适配总结](https://iluhcm.com/2017/03/12/experience-of-adapting-to-android-notifications/)

* WebView问题

  * [Android 7.0 WebView 部分机型打不开](http://blog.csdn.net/u012347067/article/details/70829013)
  * [Android 7.0 WebView 二级跳转后界面空白](http://www.jianshu.com/p/07b781795b78)

* PopupWindow位置不正确

#### 8.0

* 关于权限，对6.0的修缮

> 在 Android 8.0 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。  
对于针对 Android 8.0 的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而，一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准。
例如，假设某个应用在其清单中列出 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`。应用请求 `READ_EXTERNAL_STORAGE`，并且用户授予了该权限。如果该应用针对的是 API 级别 24 或更低级别，系统还会同时授予 `WRITE_EXTERNAL_STORAGE`，因为该权限也属于同一 `STORAGE` 权限组并且也在清单中注册过。如果该应用针对的是 Android 8.0，则系统此时仅会授予 `READ_EXTERNAL_STORAGE`；不过，如果该应用后来又请求 `WRITE_EXTERNAL_STORAGE`，则系统会立即授予该权限，而不会提示用户。

* 通知适配

管理应用是否接收通知

* 悬浮窗适配
* 安装apk
* 集合的处理
* 后台执行限制

#### 9.0

* non-SDK接口的使用

    Android P 引入了针对非 SDK 接口的新使用限制，无论是直接使用还是通过反射或 JNI 间接使用。 无论应用是引用非 SDK 接口还是尝试使用反射或 JNI 获取其句柄，均适用这些限制。

* 挖孔屏适配

## Framework

### 1. 请介绍一下 NDK
### 2. 如何在 jni 中注册 native 函数，有几种注册方式
### 3. Android 进程分类
### 4. 谈谈对进程共享和线程安全的认识
### 5. 谈谈对多进程开发的理解以及多进程应用场景
### 6. 什么是协程
### 7. 逻辑地址与物理地址，为什么使用逻辑地址
### 8. Android 为每个应用程序分配的内存大小是多少
### 9. 进程保活的方式
### 10. 系统启动流程是什么
### 11. 一个应用程序安装到手机上的过程发生了什么
### 12. App 启动流程，从点击桌面开始（Activity 启动流程）
### 13. 什么是 AIDL？解决了什么问题？如何使用？
### 14. Binder 机制及工作原理？
### 15. App 中唤醒其他进程的实现方式？
### 16. ActivityThread，ActivityManagerService，WindowManagerService 的工作原理
### 17. PackageManagerService 的工作原理
### 18. PowerManagerService 的工作原理
### 19. IPC

> [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
> []()

## 性能优化

[Android最佳性能实践(一)——合理管理内存](https://blog.csdn.net/guolin_blog/article/details/42238627)  
[Android最佳性能实践(二)——分析内存的使用情况](https://blog.csdn.net/guolin_blog/article/details/42238633)  
[Android最佳性能实践(三)——高性能编码优化](https://blog.csdn.net/guolin_blog/article/details/42318689)  
[Android最佳性能实践(四)——布局优化技巧](https://blog.csdn.net/guolin_blog/article/details/43376527)

### 内存优化

* 节制地使用Service
* 当界面不可见时释放内存
* 当内存紧张时释放内存（onTrimMemory）
* 避免在Bitmap上浪费内存
* 使用优化过的数据集合
* 知晓内存的开支情况
* 谨慎使用抽象编程
* 尽量避免使用依赖注入框架
* 使用ProGuard简化代码

### WebView优化

### 启动优化

[Android 性能优化(一) —— 启动优化提升60%](https://blog.csdn.net/qian520ao/article/details/81908505)

#### 主题优化

* 默认主题

  有白屏时间

* 使用透明主题

  治标不治本，启动有延迟

* 设置闪屏图片主题

  用户体验比较好

总的来说，这只是视觉上的优化。

#### 代码优化

首先可以通过adb命令和系统日志（displayed）统计启动时长。

##### Application 优化
##### 闪屏页业务优化

### 1. 如何对 Android 应用进行性能分析以及优化?
### 2. ANR 产生的原因是什么？怎么定位？
### 3. OOM 是什么？怎么解决？是否可以 try catch？
### 4. 内存泄露的解决方法？
### 5. ddms 和 traceView 的使用？
### 6. 性能优化如何分析 systrace？
### 7. 用 IDE 如何分析内存泄漏？
### 8. Java 多线程引发的性能问题，怎么解决？
### 9. 启动页白屏、黑屏、太慢怎么解决？
### 10. App 启动崩溃异常怎么捕捉？
### 11. Bitmap 如何处理大图？如何预防 OOM？
### 12. 如何缩小 Apk 的体积?
### 13. 如何统计启动时长？

## 动态化

### 热修复的原理，你都了解过哪几种热修复框架

[Android 主要的热修复方案原理分析](https://www.jianshu.com/p/d10aa991ca76)