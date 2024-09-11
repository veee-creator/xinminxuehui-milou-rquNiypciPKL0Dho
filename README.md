
# Activity启动模式


## 1\. Activity启动模式介绍


### 1\.1 任务栈


在Android开发中，任务栈（Task Stack）是一个非常重要的概念，主要用于管理应用程序中的Activity及其启动模式。它帮助开发者了解当用户在不同应用之间切换，或者应用内部不同Activity之间跳转时，系统如何管理这些Activity的生命周期与显示行为。任务栈是一个**后进先出（LIFO, Last In First Out）**的堆栈结构，用来存放应用程序启动的Activity。当用户启动一个新的Activity时，系统会将其压入任务栈顶部，用户可以通过返回按钮（back）将栈顶的Activity弹出（销毁），显示当前栈顶的Activity。系统中可以存在多个任务栈，当前所在任务栈的栈顶Activity实例会被显示。


### 1\.2 启动模式分类


在Android开发中，启动模式（Launch Mode）是指系统在管理Activity时如何处理该Activity的启动和任务栈的行为。它决定了一个Activity在被启动时是否创建新的实例、如何与已有的任务栈互动，以及在应用导航中如何管理Activity的状态。当你启动一个Activity时，系统需要决定是否创建一个新的Activity实例，或者复用已经存在的实例，这些行为是由启动模式控制的。其中standard模式、singleTop模式和singleInstancePerTask模式都是针对当前任务栈的；singleTask和singleInstance模式是针对系统全局的。启动模式的设置分为**静态设置**和**动态设置**两种。


#### 1\.2\.1 standard


standard(标准模式)是Activity默认的启动模式，即当前Activity未显式设置启动模式情况下，其启动模式为standard。在standard模式下，每次启动该Activity系统都会创建一个新的实例（instance）并压入**当前任务栈**顶，不论是否已有相同Activity实例存在。Activity实例数量没有限制，具体取决于任务栈深度。该模式适用于大多数情景，例如在一个应用中打开多个页面。其模式原理如下图。
![](https://img2024.cnblogs.com/blog/3515100/202409/3515100-20240910144413723-1562222809.png)


#### 1\.2\.2 singleTop


当Activity启动模式设置为singleTop时，系统启动该Activity时若发现**当前任务栈栈顶**（并非系统全局）为该Activity实例，则会直接复用该实例（调用onNewIntent()方法）而不会重新创建；若当前任务栈栈顶不为该Activity实例时，则会创建新的实例无论当前或其他任务栈中是否已存在对应实例。该模式适用于在同一任务中频繁跳转回当前页面的场景，例如从通知点击进入消息页面。其模式原理图如下。
![](https://img2024.cnblogs.com/blog/3515100/202409/3515100-20240910150602198-2059816247.png)


#### 1\.2\.3 singleTask


当Activity启动模式设置为singleTask时，系统启动该Activity时若发现**存在一个任务栈**（系统全局）中存在该Activity实例时，则会直接复用该实例（调用onNewIntent()方法）而不会重新创建，同时将该任务栈中位于该实例之上的其它Activity实例都将会被弹出栈，该实例作为当前任务栈栈顶；若所有任务栈中都不存在该Activity实例时，则会在当前任务栈中创建该Activity实例作为栈顶。该实例在**所有任务栈**中唯一存在。该模式适用于“主页”类Activity，或者需要保证该Activity在整个应用中只有一个实例的场景，如应用的主界面、设置界面。其模式原理图如下。
![](https://img2024.cnblogs.com/blog/3515100/202409/3515100-20240910151844931-1551771375.png)


#### 1\.2\.4 singleInstance


当Activity启动模式设置为singleInstance时，系统全局只允许存在该Activity的一个实例，并且该Activity将独占一个任务栈。系统启动该Activity时，若**所有任务栈**中都不存在该Activity实例时，系统会使用一个独立的任务栈，并在该栈中创建该Activity实例，该任务栈只用于管理该Activity实例，唯一存在；当该Activity实例已经存在于独立的任务栈中时，系统会直接复用该实例，该实例在**所有任务栈**中唯一存在。该模式适用于锁屏页面、视频播放等需要独立管理的Activity，防止干扰。其模式原理图如下。
![](https://img2024.cnblogs.com/blog/3515100/202409/3515100-20240910153823460-1613441376.png)


#### 1\.2\.5 singleInstancePerTask


singleInstancePerTask 是 Android API 31（即 Android 12）引入的一种新的启动模式。这种模式可以看作是对 singleInstance 和 singleTask 启动模式的一种补充，目的是增强应用在多任务场景中的灵活性。当Activity启动模式设置为singleInstancePerTask时，系统启动该Activity时若发现**当前任务栈**中存在该Activity实例时，则会直接复用该实例；若不存在，则在**当前任务栈**中创建新的实例作为栈顶。该模式适用于需要跨多个任务栈独立运行，但每个任务栈中只允许一个实例的 Activity，如多窗口操作中，每个窗口都需要独立的 Activity 实例。


## 2\. 静态设置启动模式


### 2\.1 通过AndroidManifest.xml中的launchMode属性


静态设置是指在AndroidManifest.xml文件中，通过在标签中添加android:launchMode属性来指定Activity的启动模式。这种方式在应用安装时就已经固定，不会在运行时发生变化。示例如下：



点击查看代码

```
<activity android:name=".MainActivity"
          android:launchMode="singleTop">
activity>

```


### 2\.2 静态设置特点


* **编译时确定**：在编译时指定，运行时无法更改。
* **全局适用**：对于该Activity的所有启动方式，都会按照指定的启动模式处理。
* **可读性高**：在Manifest文件中一目了然，有助于维护。


## 3\. 动态设置启动模式


### 3\.1 通过Intent的Flags属性


动态设置是指在代码中，通过为Intent添加特定的Flag（标志）来指定Activity的启动行为。这些Flag会影响Activity的启动模式和任务栈的管理。示例如下：



点击查看代码

```
#kotlin
val intent = Intent(this, MainActivity::class.java)
intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TOP
startActivity(intent)


```


### 3\.2 常用Flag介绍


* **FLAG\_ACTIVITY\_NEW\_TASK**：这个标志会告诉系统为新启动的 Activity 创建一个新的任务栈（Task），如果该 Activity 已经存在于某个栈中，它将复用现有的任务栈并将 Activity 添加到栈顶。该标志在启动一个新的 Activity 时是强制性的，特别是在从非 Activity 上下文（如 Service 或 BroadcastReceiver）启动 Activity 时。
* **FLAG\_ACTIVITY\_SINGLE\_TOP**：如果启动的 Activity 已经在栈顶，则复用它的实例，而不是创建新的实例。如果该 Activity 不在栈顶，系统将创建一个新的实例并将其推到栈顶。这个标志类似于在 launchMode 中设置 singleTop。
* **FLAG\_ACTIVITY\_CLEAR\_TOP**：当启动的 Activity 已经存在于任务栈中时，它会清除这个 Activity 之上的所有其他 Activity。然后，系统会将这个 Activity 置于栈顶并复用它的实例。这通常与 FLAG\_ACTIVITY\_NEW\_TASK 结合使用。
* **FLAG\_ACTIVITY\_CLEAR\_TASK**：这个标志会清除目标 Activity 所在的整个任务栈（Task）。所有位于该任务栈中的 Activity 都会被销毁，然后启动目标 Activity 作为新的栈根。
* **FLAG\_ACTIVITY\_REORDER\_TO\_FRONT**：如果目标 Activity 已经在任务栈中，但不是栈顶，系统会将它移动到栈顶，而不销毁栈中的其他 Activity，也不会创建新的实例。
* **FLAG\_ACTIVITY\_NO\_HISTORY**：该标志表示启动的 Activity 不会被添加到任务栈中，也就是说当用户离开这个 Activity 后，系统将不会保存它的状态。这常用于启动短暂的页面，如登录页面、广告页面等。
* **FLAG\_ACTIVITY\_EXCLUDE\_FROM\_RECENTS**：这个标志会防止启动的 Activity 出现在“最近任务”（Recents）中。即使用户通过多任务按钮查看任务列表，该 Activity 也不会出现在列表中。


### 3\.3 onNewIntent() 方法


当使用动态设置的Flags复用已经存在的Activity时，系统不会调用onCreate()方法，因为没有创建新的实例。相反，系统会调用onNewIntent()方法。你可以在onNewIntent()方法中处理传递给Activity的新的Intent。



点击查看代码

```
#Kotlin
override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent)
    // 处理新Intent
}

```


### 3\.4 动态设置特点


* **灵活性高**：动态设置允许根据应用运行时的不同场景灵活控制Activity的启动行为。例如，你可以在接收到不同的Intent时，根据用户操作动态决定是否复用现有的Activity实例或清理任务栈。
* **更精细的控制**：通过组合多个Flags，动态设置可以提供比静态设置更为细致的行为控制，满足复杂的应用需求。


## 4\. 总结


在实际开发中，可以结合静态和动态设置。例如，为某些核心页面设置静态启动模式，同时在代码中根据业务逻辑使用Intent Flags动态调整特定场景下的启动行为。


 本博客参考[楚门加速器p](https://tianchuang88.com)。转载请注明出处！
