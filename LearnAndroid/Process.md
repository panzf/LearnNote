# Android进程 #
### 分类 ###
1. **前台进程：**即与用户正在交互的Activity或者Activity用到的Service等，如果系统内存不足时前台进程是最后被杀死的

2. **可见进程：**可以是处于暂停状态(onPause)的Activity或者绑定在其上的Service，即被用户可见，但由于失去了焦点而不能与用户交互

3. **服务进程：**其中运行着使用startService方法启动的Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面自己下载的文件等；当系统要空间运行前两者进程时才会被终止

4. **后台进程：**其中运行着执行onStop方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这样的进程系统一旦没了有内存就首先被杀死
5. **空进程：**不包含任何应用程序的程序组件的进程，这样的进程系统是一般不会让他存在的

### 如何避免后台进程被杀死 ###
1. 调用startForegound，让你的Service所在的进程成为前台进程
2. Service的onStartCommond返回START_STICKY或START_REDELIVER_INTENT
3. Service的onDestroy里面重新启动自己