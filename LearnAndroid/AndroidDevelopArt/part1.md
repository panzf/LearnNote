# 《Android开发艺术第一章》 #
## Activity 生命周期与启动模式 ##
### Activity 生命周期 ###
- **onCreate**：
    > 表示Activity正在被创建，这是生命周期的第一个方法在这个方法里我们可以做一些初始化化的工作，比如setContentView加载布局控件findview以及Activity所需要的数据,注意：不要做一些耗时太长的操作，影响界面加载显示，此时View还没开始绘制。
- **onStart**：
    >表示Activity正在被启动，这时候Activity已经变为可见了，但是还没出现在前台，还无法与用户交互，可以理解为以显示展示看不见。也是不要做太过多耗时间的操作，影响界面显示速度。 
- **onResume**：
    > 表示Activity已经可见，并且显示在前台开始活动可以做交互了和onstar对比就是当前是可见可交互状态onStart不可以交互。
- **onPause**：
    > 表示当前Activity正在停止，正常情况下onStop会马上被调用，特殊情况就死迅速返回当前Activity那么onresume会被调用，注意不能做一些耗时长的操作，因为会影响到新的Activity的显示。只有当onPause执行完才会执行新的Activity的onCreate到onResume 

- **onStop**：
    > 表示Activity即将被停止可以做一些稍微重量级的一些操作，也不要做耗时性长的操作。 

- **onRestart**：
     >表示Activity重新启动，一般在当前需要启动Activity由不可见状态变为可见状态时候被调用 。这种情况一般是用户行为导致的，比如Home键切换回桌面或者启动一个被打开的一个新的Activity并且这个Activity执行了onPause与onStop，返回情况。

- **onDestroy**：
    > 表示Activity即将被停止，这是最后的一个回调，我们可以在这里做回收工作和资源释放。
 
生命周期切换过程图 ![](http://i.imgur.com/NAJqSJu.png)
##### 针对分析 #####
1. 一个Activity，第一次启动回调如下：onCreate -> onStart -> onResume。这是一般是连续的过程当启动完成上一个Activity的onStop才会被回调。
2. 当打开新的一个Activity或者切换到桌面时候，当前Activity回调如下：onPause -> onStop。当然有一种特殊情况就是新的Activity采用了透明的主题时候，当前Activity并不会调用onStop。
3. 当用户再次回到原来的Activity中时候的回调如下：onRestart -> onStart -> onResume当然有特殊情况就是那个Activity由于系统资源紧张被回收了那么它的回调如下onCreate -> onStart -> onResume。也就是重新执行启动一个新的Activity一样的过程。
4. 当用户按下回退键时候当前的Activity生命周期回调如下：onPause -> onStop -> onDestroy

##### 异常情况下的生命周期分析 #####

- 情况1：资源相关的系统配置发生改变导致的Activity被杀死并重新创建
    > 在横竖屏切换的过程比如drawable目录的资源分为 drawable-land drawable-sw720等等等情况下由于是要拿不同图片资源默认情况下Activity是会被重新创建的不管有没有区分资源，当然我们也可以阻止系统重新创建 。当配置发生改变时候声明周期的调用onSaveInstance -> onStop -> onDestroy。我们可以通过onSaveInstance来保存数据状态等数据。然后重新启动的Activity执行onCreate -> onStart -> onResume这一过程并且还会调用onRestoreState，然后我们就可以在这个方法通过传入的参数拿出保存的数据，注意这个方法只有在Activity调用了onSaveInstance保存了数据才会回调拿出保存的数据，要是没有保存调用了没有数据。

- 情况2：资源内存不足导致地优先级的Activity被杀死
    > Activity的优先级由高到低，可分为三种情况。  
    
    -  前台可见的Activity-也就是正在和用户交互的Activity，优先级最高。
    -  可见但非前台的Activity-比如在Activity弹出了一个对话框（dialog）导致的Activity可见但是位于后台无法直接交互。
    -  后台Activity-已经被暂停的Activity执行了onStop，优先级最低。

如果一个进程没有四大组件在执行，那么这个进程就很容易被杀死，因此一些后台工作不适合脱离四大组件独自运行。比较好的方法就是将后台工作放在Service中从而保证进程有一定的优先级，这样才不容易被杀死。

- 阻止因为系统配置而导致从新创建
    >我们可以通过配置清单文件中的Activity组件注册中添加android：configChanges="orientation|screenSize"，然后当前Activity就不会因为旋转屏幕而重新创建了相应的它会回调 onConfigurationChanged()方法


### Activity 启动模式 ###
 
Activity启动模式有四种分别为：standard，singleTop，singleTask，和singleInstance。

- standard模式
    >标准模式，也是系统默认的启动模式，不管这个实例是否已存在都会启动一个Activity都会创建一个新的实例置于栈顶中，被那个Activity启动就加入启动它的那个Activity的栈中去。注意ApplicationContext启动一个standard模式的Activity会报错的因为Application中并没有任务栈说法也就入不了栈，当然我们可以制定一个标志位为FLAG_ACTIVITY_NEW_TASK这也就是singleTask的启动模式了。
        
    - 场景有：标准的基本页面模式
    

- singleTop模式
    >栈顶复用模式，在这种情况下呢如果要启动的Activity已经位于栈顶中那么不会创建一个新Activity，同时它的onNewIntent会被调用，通过此方法我们可以拿到请求的参数其他回调方法不会调用。如果栈顶不是当前要启动的Activity实例那么会创建一个实例加入栈顶中不管栈中是否有当前启动的实例。
    
    - 场景有：分享页面，Service或Broadcast启动当前栈顶的Activity并且是singleTop模式的.


- singleTask模式
    > 栈内复用模式，这一种是单例模式的情况在这种情况下只要启动它那个Activity栈内有那么就会复用并且调用 onNewInstance 方法并且把处于它之上的Activity都弹出栈去自己置于栈顶启动过程如下：Activity需找是否有它想要要启动的栈如果没有那么创建栈并且新建一个一个当前要启动的实例置于栈中也就是栈顶了，如果有Activity想要启动的栈则不会创建一个新的栈而是去已存在的栈中找有没有当前实例有则把处于它之上的Activity都弹出栈自己处于栈顶，如果没有这创建一个当前Activity的实例置于栈顶中。
    
    - 场景有：浏览器Activity等一个应用只能存在一个实例对象的Activity。


- singleInstance
    >单实例模式，这是一种加强版singleTask模式，除了有singleTask的特点外并且还加强了一点只能位于单独的一个栈中去不能位于启动它的Activity栈中。
   
    - 场景：暂无   

##### 如何指定 Activity 启动模式 ##### 
- 第一种：通过在配置清单文件中activity节点加入 android：lanuchMode="上述四种中的一种"此方法优先级最高。

- 第二种：就是通过Intent启动Activity时候添加flag也就是 `intent.addFlags
(Intent.FLAG_ACTIVITY_NEW_TASK)`添加这一句代码来启动singleTask模式。


另外还有两点要说：
- 第一点:在启动模式为singleTask中指定`android:taskAffinity="xxx.xxx.xxx.xxx"`属性并且不能和包名相同要相同则不需要添加这一属性就可以指定启动的任务栈。默认所有Activity的任务栈都是包名那个栈默认是有这个包名的栈。任务栈分为前台任务栈与后台任务栈后台的任务栈中的Activity都处于暂停状态。
- 第二点：singleTask加`android:taskAffinity="xxx.xxx.xxx.xxx"`和        `android:allowTaskReparenting=""`有特殊效果比如应用A启动应用B的一个Activity并且含有这三个条件的启动完成，然后返回桌面在启动应用B那么此Activity会直接从A的任务栈中移到应用B中去然后B显示这个Activity


### IntentFilter 的匹配规则 ###

Activity 启动分为2种：显示调用与隐式调用
> 显示调用需要明确指定Activity的对象详细包括报名和全类名，而隐式调用不要指定明确的组件的信息。如果二者并存以显示为主。

隐式调用需要Intent完全匹配目标组件的IntentFilter的信息否则无法启动，而且组件可以有多个IntentFilter规则能完全匹配一个就可以启动完全匹配意思就是如果有intent Action，Categoty，data需要每个完全匹配才能启动另外Service不支持隐式启动。

##### Action 的匹配规则 #####
> action的匹配规则就是intent中的action和IntentFliter中的一个相同，一个IntentFilter可以有多个action。

##### Category 的匹配规则 #####

> Category的匹配规则就是要求Intent中含有的Category那么必须全部不管有几个都要与IntentFilter中的其中一个匹配，可以没有Category。

##### Data 的匹配规则 #####
> Data匹配规则与action类是IntentFilter如果有data，intent要求含有data数据，并且指定了data数据才匹配了IntentFilter中的一个data才能能算匹配。
   
另外data中只有mineType类型没有指定scheme类型则默认scheme为（contnet://或则file：//）中的一种。如果intent要指定data必须通过`intent.setDataAndType(uri, "*/*")`这个方法来设置，因为i`ntent.setData()`或者`intent.setType()`会清除另外一方。
