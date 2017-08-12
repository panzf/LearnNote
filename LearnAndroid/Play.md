## 第三方支付流程 ##

### 第一步准备阶段 ###

- 下载SDK（第三方支付平台官网）
- 导入Jar包、修改Manifest（注册组件、添加权限）
- 添加混淆规则

### 第二步支付流程 ###

##### 第一阶段WebView交互 #####
- 通过服务器下发支付链接然后客户端通过`webview.load(url)`加载支付页面选择支付类型点击支付客户端通过`webview.setWebViewClient`中的`shouldOverrideUrlLoading`方法拦截指定链接
- 借助js调用js指定函数比如`javascript:webobj.getParams();`让web端处理封装好需要的支付参数后通过`WebView.addJavascriptInterface(mBridge, "obj")`调用协商好的javaBridge函数比如`public void callClient(String data)`函数`window:obj.callClient(json)`传递json结果到javabridge对象中。

##### 第二阶段调用SDK支付 #####
- 通过json封装的参数得到调用的平台参数与支付参数比如`platform=data.getString("platform")`和` payParams = data.getString("payParams")`商品支付参数然后根据platform调用指定的平台支付传入支付参数调起支付得到支付结果。

##### 第三阶段上传支付结果 #####
- 把支付结果和支付平台信息等需要的参数上传到服务器中返回最终结果

### 最后一步 ###
- 最后关闭页面返回得到最终结果