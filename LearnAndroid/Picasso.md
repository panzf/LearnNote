## Picasso图片加载框架总结 ##
### 流程图 ###
![](http://i.imgur.com/Q87lq7q.jpg)
### 总体设计流程 ###

> 整个库分为 Dispatcher，RequestHandler 及 Downloader，PicassoDrawable 等模,创建 Request 并将它交给 Dispatcher，Dispatcher 分发任务到具体 RequestHandler，任务通过 MemoryCache 及 Handler(数据获取接口) 获取图片，图片获取成功后通过 PicassoDrawable 显示到 Target 中。

#### 具体流程 ####
- Picasso构建有内部Builder.build()中初始化Picasso中所需参数 Downloader，LruCache ExecutorService(为PicassoExecutorService)，RequestTransformer Dispatcher。
- Downloader 根据当前Project 中是否使用了Okthttp来初始化如果有这使用Okhttp并且为其配置磁盘缓存下载否则使用HttpURLConnection下载无磁盘缓存
- LruCache 根据手机配置内存缓存
- Dispatcher 负责分发和处理 Action，包括提交、暂停、继续、取消、网络状态变化、重试等等。
- PicassoExecutorService 配置了一个3条工作线程的线程池用于提交请求

#### 优点 ####
- **自带统计监控功能:** 支持图片缓存使用的监控，包括缓存命中率、已使用内存大小、节省的流量等
- **支持优先级处理:** 每次任务调度前会选择优先级高的任务，比如 App 页面中 Banner 的优先级高于 Icon 时就很适用
- **支持延迟到图片尺寸计算完成加载**
- **"无"本地缓存:**无本地缓存，不是说没有本地缓存，而是 Picasso 自己没有实现，交给了 Square 的另外一个网络库 okhttp 去实现，这样的好处是可以通过请求 Response Header 中的 Cache-Control 及 Expired 控制图片的过期时间
