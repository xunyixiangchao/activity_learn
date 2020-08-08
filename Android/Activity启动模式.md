# 从四个视角理解Android Activity启动模式



系统视角：

## 1. Android的软件体系结构

![系统结构图](.\系统体系结构图.jpg)



## 1.2 Task

![Task](.\task.png)

Activity代码属于Application，但是Task属于Android操作系统

Task是可以跨应用的

### 手机查看Task：（用户角度）

手机中按home键旁边那个方形键（recent-apps）时，屏幕上展示的就是一个个task。

![查看手机Task](.\查看Task.png)

### 代码中查看Task：（程序角度）

adb shell dumpsys activity activities | sed -En -e '/Stack #/p' -e '/Running activities/,/Run #0/p'

sed工具不用单独下载，`D:\soft\Git\usr\bin\sed.exe` Git安装目录下包含，配置下环境变量就可以。 



用户视角：



## 2.1 Task启动方式(launcher启动)

Launcher启动

1、Task不存在

2、Task存在

## 2.2 Task启动方式（新建）

```java
Intent intent = new Intent(this, SecondActivity.class);
intent.putExtra("message", "message");
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
startActivity(intent);
```

通知：

1.系统通知
2.自己

其他第三方应用：

1、Scheme协议
2、第三方应用start



**launcher,新建  都是通过startActivity来创建的。**





## 2.3 Task启动方式（恢复）

**恢复**   这属于Activity生命周期由不可见到获得焦点的范畴

![Task恢复](.\Task恢复.png)

程序视角：



## 3.1 Activity和Fragment

Fragment是Android3.0后引入的一个新的API，他出现的初衷是为了适应大屏幕的平板电脑， 当然现在他仍然是平板APP UI设计的宠儿，而且我们普通手机开发也会加入这个Fragment， 我们可以把他看成一个小型的Activity，又称Activity片段！

![fragment静态加载](.\fragment静态加载.png)

![fragment动态载](.\fragment动态加载.png)

## 3.2 Activity的生命周期

![Activity生命周期](.\Activity生命周期.png)



Activity是否可见：

![Activity是否可见](.\Activity是否可见.png)

PS:Fragment生命周期

![fragment生命周期](.\fragment生命周期.webp)

Activity与Fragment生命周期

![Activity与Fragment生命周期](.\activity与fragment.png)

## 3.3 相邻状态之间的区别

![A启动B和B返回A](.\相邻状态区别.png)

A启动B    和    B返回A



**1.onCreate**和**onStart**之间有什么区别？

（1）可见与不可见的区别。前者不可见，后者可见。
（2）执行次数的区别。onCreate方法只在Activity创建时执行一次，而onStart方法在Activity的切换以及按Home键返回桌面再切回应用的过程中被多次调用。因此Bundle数据的恢复在onStart中进行比onCreate中执行更合适。
 （3）onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如前文所说的，setContentView和资源初始化在两者都能做，然而想动画的初始化在onStart中做比较好。

**2.onStart**方法和**onResume**方法有什么区别？

（1）是否在前台。onStart方法中Activity可见但不在前台，不可交互，而在onResume中在前台。
（2）职责不同，onStart方法中主要还是进行初始化工作，而onResume方法，根据官方的建议，可以做开启动画和独占设备的操作。

**3.onPause**方法和**onStop**方法有什么区别？

（1）是否可见。onPause时Activity可见，onStop时Activity不可见，但Activity对象还在内存中。
（2）在系统内存不足的时候可能不会执行onStop方法，因此程序状态的保存、独占设备和动画的关闭、以及一些数据的保存最好在onPause中进行，但要注意不能太耗时。

**4.onStop**方法和**onDestroy**方法有什么区别？

onStop阶段Activity还没有被销毁，对象还在内存中，此时可以通过切换Activity再次回到该Activity，而onDestroy阶段Acivity被销毁



**PS:**闪屏页：在onStop()方法中进行finish();



## 3.4 onNewIntent的生命周期

![onNewIntent](.\onNewIntent.png)

1、只对**singleTop，singleTask，singleInstance**有效，因为standard每次都是新建(不是绝对，使用了Intent.FLAG_ACTIVITY_NEW_TASK,要启动的Activity已经有Task在运行了，新的activity不会再创建，而是把当前堆栈的activity带到前台)，所以不存在onNewIntent；

2、只对startActivity有效，对于从Navigation切换回来的恢复无效；



## 4.1 Activity启动模式

![四种启动模式](.\四种启动模式.png)



## 4.2 standard启动模式

**1、standard**  **默认模式**

系统在启动 Activity 的任务中创建 Activity 的新实例并向其传送 Intent。Activity 可以多次实例化，不管这个实例是否已经存在，而每个实例均可属于不同的任务，并且一个任务可以拥有多个实例。这种模式的 Activity 被创建时它的 onCreate、onStart 都会被调用。这是一种典型的多实例实现，一个任务栈中可以有多个实例，每个实例也可以属于不同的任务栈。在这种模式下，谁启动了这个 Activity，那么这个 Activity 就运行在启动它的那个 Activity 所在的栈中。



a、当从非Activity的context启动activity时，需要带new_task的flag；

b、当启动一个带有affinity的activity，如果这个activity已经有实例存在该task，则不会重新创建；

c、如果从应用内启动的standard activity的Affinity就是App默认的Affinity，则会每次新建一个实例；



## 4.3 singleTop启动模式

一个singleTop Activity 的实例可以无限多，唯一的区别是如果在栈顶已经有一个相同类型的Activity实例，Intent不会再创建一个Activity，而是通过onNewIntent()被发送到现有的Activity。

![singleTop](.\singetop.png)



## 4.4 singleTask模式

这是一种单实例模式，在这种模式下，只要 Activity 在一个栈中存在，那么多次启动此 Activity 都不会重新创建实例，和 singleTop一样，系统也会回调其 onNewIntent。当一个具有 singleTask 模式的Activity请求启动后，比如 Activity A，系统首先会寻找是否存在 A 想要的任务栈，如果不存在，就重新创建一个任务栈，然后创建 A 的实例后把 A 放到栈中。如果存在 A 所需的任务栈，这时要看 A 是否在栈中有实例存在，如果有实例存在，那么系统就会把 A 调到栈顶并调用它的 onNewIntent 方法，如果实例不存在，就创建 A 的实例并把 A 压入栈中 。

![singleTask](.\singleTask.png)



不需要关注NEW_TASK



## 4.5 singleInstance模式

与 singleTask 相同，只是系统不会将任何其他 Activity 启动到包含实例的任务中。该 Activity 始终是其任务唯一仅有的成员；由此 Activity 启动的任何 Activity 均在单独的任务中打开。也就是有此种模式的 Activity 只能单独地位于一个任务栈中



PS：4种模式只能在AndroidManifest.xml中定义（定义层定义的）



## 4.6 Intent Activity Flag

启动层定义

![IntentFlag](.\IntentFlag.png)



## 5.1 启动模式的应用场景

| **launchMode** | **使用场景**                                                 |
| -------------- | ------------------------------------------------------------ |
| singleTop      | 适合启动同类型的   Activity，例如：   •接收通知启动的内容显示页面   •耗时操作返回页面   •登录页面 |
| singleTask     | 适合作为程序入口，例如：   •WebView页面   •扫一扫页面   •确认订单界面   •付款界面 |
| singleInstance | 适合需要与程序分离开的页面，例如：   •闹铃的响铃界面   •来电页面   •锁屏页 |





























