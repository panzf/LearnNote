# 《Android开发艺术第二章》 #
## IPC机制 ##
### ICP简介 ###

ICP全称 Inter-Process Communication 含义为跨进程通信/进程间通信，是指两个进程间的的数据交换过程。说道进程一般会和线程联系在一起几笔比较，我们要先了解什么事进程和什么是线程他们有什么关系 。

### 进程与线程 ###

##### 进程 #####
进程是程序执行时的一个实例应用，即它是程序已经执行到何种程度的数据结构的汇集。从内核的观点看，进程的目的就是担当分配系统资源（CPU时间、内存等）的基本单位。

使用多进程的理由
- 提高应用程序的健壮性，很好的把模块划分把独立的模块进程崩溃不影响主模块的运行。
- 在Adnroid中一个进程最大的内存有限制，当系统资源不足内存暂用越多的进程越容易被系统杀死。
- 在主进程推出的情况下，子进程仍需工作。比如：推送服务

##### 线程 #####
线程是进程的一个执行流，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。一个进程由几个线程组成（拥有很多相对独立的执行流的用户程序共享应用程序的大部分数据结构），线程与同属一个进程的其他的线程共享进程所拥有的全部资源

使用多线程的理由

- 它和进程相比，它是一种非常"节俭"的多任务操作方式    
> 在Linux系统下，启动一个新的进程必须分配给它独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段，这是一种"昂贵"的多任务工作方式。而运行于一个进程中的多个线程，它们彼此之间使用相同的地址空间，共享大部分数据，启动一个线程所花费的空间远远小于启动一个进程所花费的空间，而且，线程间彼此切换所需的时间也远远小于进程间切换所需要的时间。据统计，总的说来，一个进程的开销大约是一个线程开销的30倍左右，当然，在具体的系统上，这个数据可能会有较大的区别。 

    
- 二是线程间方便的通信机制。
> 对不同进程来说，它们具有独立的数据空间，要进行数据的传递只能通过通信的方式进行，这种方式不仅费时，而且很不方便。线程则不然，由于同一进程下的线程之间共享数据空间，所以一个线程的数据可以直接为其它线程所用，这不仅快捷，而且方便。当然，数据的共享也带来其他一些问题，有的变量不能同时被两个线程所修改，涉及到线程安全问题。 


##### 总结 #####
> 进程——资源分配的最小单位，线程——程序执行的最小单位


进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。

### IPC 通信方式 ###

本地的进程间通信（IPC）有很多种方式，但可以总结为下面4类
- 消息传递（管道、FIFO、消息队列）
- 同步（互斥量、条件变量、读写锁、文件和写记录锁、信号量）
- 共享内存（匿名的和具名的）
- 远程过程调用（Solaris门和Sun RPC）

Android是基于Linux内核的移动操作系统它的进程通信不完全集成Linux系统。相反它有自己的进程通信方式也就Binder通信。Android还支持Socket通信，通过Socket也可以实现多进程通信，另外还有Linux里的共享内存等。

### Android 多进程模式 ###

##### 开启多进程 #####

- 唯一的正常方式：通过给四大组件指定`Android：Process=".xxx/xxx.xxx.xxx` 或 `Android：Process=":xxx"`方式。
- 另外特殊的方式：那就通过JNI方式在native层Linux方式fork一个进程。

默认情况下所有组件都是运行在同一个进程中也就是以包名命名的这个进程中。

######.xxx/xxx.xxx.xxx** 与 :xxx的区别 ######
- **:xxx**

意思是指定当前进程名为 包名:xxx 进程的一种简写方式并且当前进程是属于私有进程其他应用组件不能和它跑在同一的进程中。

- **.xxx/xxx.xxx.xx**

意思是当前的进程名就是 .xxx/xxx.xxx.xx 该进程为全局进程其他应用可以通过shareUID方式和它跑在同一进程中。

###### 什么是ShareUID？ ######
Android系统会为每个应用分配一个唯一的UID(可通过配置清单文件 manifest节点中指定`    android:sharedUserId="xxx.xxxx"`)具有相同UID的应用能够共享数据。两个应用通过ShareUID跑在同一进程的要求要有相同UID和相同签名。在这种情况下他们可以相互访问对方的数据，比如data目录，组件信息等，不管它们是否是跑在同一进程。当然如果跑在同一进程中那么除了共享data目录，组件信息，还可以共享内存数据，或者它本就是同一个应用的两部分。

###### 多线程会造成的问题 ###### 

- 静态成员和单例失效，很好理解都是不同的应用程序了当然失效了。
- 线程同步机制完全失效，也很好理解不同的应用程序不是同一块内存线程之间没有仍何关联。
- SharePrefenerces可靠性下降，SharePrefenerce不支持两个进程同时执行写的操作否则会可能导致一定丢失数据的几率，因为SharePrefenerces底层是通过读写XML实现的，并发写显然是完全可能的，甚至并发读写都有可能出问题。


同一个应用的多进程可以理解为两个不同的应用采取了ShareUID模式，这样更加直接理解多进程模式的本质。

##### IPC基础 #####

###### Serializable ######
>Serializable 是Java提供的序列化接口，一个空接口，它为对象提供序列化和反序列化的操作， 使用起来也方便只要实现这个接口然后添加一个常量
`private static final long serialVersionUID= xx ` 即可。

###### 工作原理 ######

> 序列化时系统会把当前的类 serialVersionUID 写入序列化文件中，当反序列化时候系统会检测文件的 serialVersionUID 看它是否和当前的类的版本相同。相同则反序列化成功，否则说明当前的类有做新的修改比如成员变量变多，或者类型改变了这时候是无法反序列成功的。 

其次注意 transient 修饰的变量是不参与序列化的过程的，为了添加序列化的灵活性我们应该手动添加 serialVersionUID 可以在添加成员操作不影响之前的数据反序列化，在原来的成员没有修改的情况下还是可以反序列成功。

具体操作查看ObjectOutputStream/ObjectInputStream用法。

###### Parcelable ######

Parcelable 是Android提供的特有的序列化的接口，这比 Serializable 的效率更高一般提倡使用Parcelable这也是系统推荐的方式唯一缺点就是实现这个接口比较复杂 (AS可一键实现)。

总结：Serializable是Java提供的序列化的接口使用起来简单但是开销大需要大量的IO操作。而Parcelable是Android提供的序列化方式，因此使用Android平台，缺点就是使用起来比较麻烦，但是效率很高因此首选。

##### Binder #####
###### 简介 ######

Binder是Android中的一个类，它实现了IBinder的接口。从IPC中来讲Binder是Android的跨进程的通信方式。Binder还可以理解为是一种虚拟的物理设备,它的设备驱动是/dev/binder，该通信方式在Linux中没有。从Android Framework 中来讲Binder是ServiceManager连接各种Manager(ActivityManager,WindowManager,等等)和相应MnangerService的桥梁。从Android应用层来说Binder是客户端与服务端进行的通信媒介，当bindService时候服务端就会返回包含了服务端业务的Binder的对象，通过这个对象客户端就可以获取服务端提供的服务或者数据，这些服务包括了普通服务和AIDL服务。

###### AIDL ######

AIDL是Android Interface definition language 它是Android内部进程通信接口描述语言实现进程通信。

###### 通过AIDL生成的接口方法分析 ######

- asInterface(IBinder obj)

 用于服务端的Binder对象转换成客户端所需要的AIDL接口类型对象这种转换是区分进程的，如果客户端和服务端位于同一进程，那么返回的是服务端对象的本身，否则返回的是系统封装过后的Stub.Proxy对象。

- asBinder
    
    返回当前的Binder对象。
     
- onTransact
    
    此方法运行在服务端中的Binder线程持中，当客户端发起跨进程请求时，远程会通过系统底层封装后交由此方法处理。该方法的原型为 `public boolean onTransact(int code,android.os.Parcel data,android.os.Parcel reply,int flags)`。服务端通过code即可确定客户端的请求目标是什么，接着从data取出目标方法所需要的参数(如果目标有参数)，然后执行目标方法。当方法执行完毕后就向reply写入返回值（如果有返回值）onTransact方法执行完毕。需要注意：如果次方法返回false,那么客户端请求失败因此我们可利用这个特性进行权限验证。
     
    当客户端发起远程请求时候，由于当前线程会处于挂起状态直到服务进程返回数据，所以一个远程方法如果是很耗时的，那么就不能在UI线程中发起请求。其次服务端的Binder运行在Binder的线程池中所以Binder不管是否耗时都得采取同步方式去实现因为Binder线程池最大线程数15完全有可能并发操作导致的线程安全问题。另外服务端是我们主进程也就是反过调用时候记得此时服务端的方法是运行在Binder线程池中步能更新UI操作。

- Binder 工作机制流程图
![](http://i.imgur.com/EjlJM3h.jpg)

###### 服务进程异常断开问题 ######

- Binder 中 linkToDeath 与 unlinkToDeath 方法

 Binder的linkToDeath与unlinkToDeath这两个方法很重要，因为Binder实体运行在服务端进程，如果服务端因为某种原因导致进程异常终止这时候Binder连接断裂(称之为Binder死亡),会导致我们客户端的远程调用会失败。关键是如果我们客户端不知道Binder已经死亡那么客户端的功能就会受到影响。为了解决这一问题解可以通过这两个方法来给binder设置死亡代理,当Binder死亡时收到通知然后可以重新启动服务。
     
```
	binder.linkToDeath(deathRecipient, 0);//在返回一个binder时候调用
	binder.unlinkToDeath(deathRecipient, 0);//在死亡时或者需要停止服务调用
	
    //binder死亡代理回调通知
    private DeathRecipient deathRecipient = new DeathRecipient() {

		@Override
		public void binderDied() {
            //binder.unlinkToDeath(deathRecipient, 0)
            //binder = null;
            //可以在次新启动服务service.startService
		}
	};
```

##### Android IPC通信方式 #####
Android 应用进程通信细分为：Bundle,文件共享，AIDL,Messenger,ContentProvider和Socker。

- Bundle

    在四大组件中的三大组件(Activity，Service,Receiver)Bundle是支持通过Intent传递数据,由于Bunde实现了Parcelable接口可以方便在不同的进程中传输。Bundle支持传递基本数据类型和基本数据类型的数组 String类型和实现序列化类的对象。
    
- 共享文件
    
    共享文件也是一种不错的进程通信方式，这两个进程读写同一个文件来交换数据A进程写文件，B进程读文件来获取数据。在windows中，一个文件加了排斥锁会导致其他线程无法进行读写的访问，而Android基于Linux系统，使其能够并发读写文件没有限制的进行，甚至两个线程一起写的可能，尽管这可能会出问题。共享文件适合在对数据同步要求不高的进程通信，并且要妥善处理并发读/写问题 。

    Android SharePreferences是一个特例它通过键值对的方式来存储数据，在底层是通过XML来存储键值对，但是系统对于它的读写有一定的缓存策略，即在内存中有一份SharePreferences的文件缓存。在多进程下系统对它的读写就变得不可靠，在面对高并发情况下有很大的几率会丢失数据，因此不建议在多进程通信使用SharePreferences。

- Messenger
 
     Messenger 翻译为信使，通过它也可以在不同进程中传递Message对象，Message中放入我们需要传递的数据即可轻松的实现数据进程间的传递，Messenger是一种轻量级的 IPC方案，它底层是通过AIDL来实现的。  
    ```
    public Messenger(IBinder target) {
        mTarget = IMessenger.Stub.asInterface(target);
    }
    
    public Messenger(Handler target) {
        mTarget = target.getIMessenger();
    }
    
    ```
    **注意:** 通过Message传递对象也就是 message 中的object来当载体传递对象数据的在那2.2版本前是不支持的2.2后添加支持也要实现了序列化的对象才能传递也就是实现了Parcelable接口,导致object实用性大减不过我们可以借助Bundle来传递`message.setData(bundle)`。
  - client   

    ```    
        public class MainActivity extends Activity {
    
    	private ServiceConnection conn = new ServiceConnection() {
    
    		@Override
    		public void onServiceDisconnected(ComponentName name) {
    		
            }
    		@Override
    		public void onServiceConnected(ComponentName name, IBinder service) {
    			Messenger mService = new Messenger(service);
    			Message msg = Message.obtain();
    			Bundle data = new Bundle();
    			msg.what = 100;
    			data.putString("KEY", "this msg from client");
    			msg.setData(data);
    			msg.replyTo = mClientMessenger;
    			try {
    				mService.send(msg);
    			} catch (RemoteException e) {
    				e.printStackTrace();
    			}
    		}
    	};
    
    	private static class ClientHandler extends Handler {
    		@Override
    		public void handleMessage(Message msg) {
    			switch (msg.what) {
    			case 200:
    				Log.i("TAG", "msg:" + msg.getData().getString("KEY"));
    				break;
    
    			default:
    				super.handleMessage(msg);
    				break;
    			}
    			super.handleMessage(msg);
    		}
    	}
    
    	private Messenger mClientMessenger = new Messenger(new ClientHandler());
    
    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
    		super.onCreate(savedInstanceState);
    		setContentView(R.layout.activity_main);
    		Intent service = new Intent(this, MessengerService.class);
    		bindService(service, conn, BIND_AUTO_CREATE);
    	}
     }
    ```

  - service
    
    ```   
        public class MessengerService extends Service {
    
    	private static class ServiceHandler extends Handler {
    		@Override
    		public void handleMessage(Message msg) {
    			switch (msg.what) {
    			case 100:
    				Log.i("TAG", "msg:" + msg.getData().getString("KEY"));
    				Message message = Message.obtain();
    				Bundle clientData = new Bundle();
    				message.what = 200;
    				clientData.putString("KEY", "this msg from Service");
    				message.setData(clientData);
    				try {
    					msg.replyTo.send(message);
    				} catch (RemoteException e) {
    					e.printStackTrace();
    				}
    				break;
    			default:
    				super.handleMessage(msg);
    				break;
    			}
    		}
    	}
    
        	private final Messenger messenger = new Messenger(new ServiceHandler());
        
        	@Override
        	public IBinder onBind(Intent intent) {
        		return messenger.getBinder();
        	}
    }
    ```

 - 工作流程 
![](http://i.imgur.com/OVxtSpL.jpg) 


    