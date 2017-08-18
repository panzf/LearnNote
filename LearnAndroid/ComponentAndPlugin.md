## 组件化与插件化 ##
### 组件化开发 ###

将一个APP分为多个模块,每个模块都是一个组件（Module）开发过程可以将这些组件相互依耐或单独调试最终发布将这些组件合并统一一个APK。

##### 概述 #####

- android工程的组件一般分为两种，lib组件和application组件
- application组件是指该组件本身就可以运行并打包成apk
- lib组件是指该组件属于app的一部分，可以供其它组件使用但是本身不能打包成apk

###### 为什么要有组件化？ ######
假如一个app工程只有一个组件，随着app业务的壮大模块越来越多，代码量超10万是很正常的，这个时候我们会遇到以下问题

1. 稍微改动一个模块的一点代码都要编译整个工程，耗时耗力
2. 公共资源、业务、模块混在一起耦合度太高
3. 不方便测试

###### 划分组件 #######
- .新建一个lib组件，new Module—>Andorid Library，取名BaseUtilLib，我们将所有的公共的工具类、网络分装等类放在其中
- 新建一个lib组件，BaseReslLib，我们将所有的公共资源、drawable、String等类放在其中
- 将app按照自己的规则划分成多个模块，比如按业务划分
- 逐一开发某个模块引用 BaseUtilLib，BaseReslLib模块等

### 插件化开发 ###

将整个APP拆分成多个模块，这些模块分为宿主模块和插件模块，每个模块都是一个APK，最终打包时将数组APK和插件APK分开打包

###### 为什么有插件化？ ######
有了组件化，为什么还要用插件化呢？插件化开发总的来说有以下几点好处（不同插件框架不一样）

1. 宿主和插件分开编译
2. 并发开发
3. 动态更新插件
4. 按需下载模块
5. 方法数或变量数爆棚

###### 关于插件化开发 ######
[动态加载APK1](https://github.com/panzf/DLAPK)
[实现Android App多apk插件化和动态加载，支持资源分包和热修复](https://github.com/CtripMobile/DynamicAPK)
[dynamic-load-apk](https://github.com/singwhatiwanna/dynamic-load-apk)