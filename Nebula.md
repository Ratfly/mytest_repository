## Android Nebula H5容器简介

### 一 概述

随着接入容器的业务方越来越多，接入的方式也越来越多样化。容器不仅要支持钱包，支持mPaas分支，同时也要支持独立的SDK调用。容器的重构迫在眉睫。经过钱包9.2版本的重构内核后，现在已经完全支持支付宝钱包，mPaas分支，独立SDK等各种接入环境。目前接入的客户端包括支付宝钱包，蚂蚁聚宝，商家客户端，阿里内外，知数据等。

同时nebula全面支持ucwebview,使用uc内核，使容器拥有了修复native crash的能力，页面加载时间下降了20%左右，同时webview的内存使用也得到较好的控制。

### 二 模块

容器现在分为4个bundle。

![Alt text](http://ux.alipay-inc.com/ftp/h5/rongyang/nebula/bundle.jpg)



nebula:容器对外的接口，主要用于在支付宝钱包中作为对外的依赖。

nebulacore：容器的核心部分，包括事件管理，插件管理，离线包管理，Provider管理等核心模块。容器独立SDK的主要部分。

nebulauc：用于加载ucwebview。容器现在已经全面支持ucwebview,支持线上开关自定义是否使用uc内核。

nebulabiz:主要为依赖钱包的各种plugin和provider，主要用于钱包环境。


### 三 设计


 容器架构设计如下图所示：

![Alt text](http://ux.alipay-inc.com/ftp/h5/rongyang/nebula/design.jpg)



#### nebulacore各模块说明如下：

#### 3.1 core:
核心包，主要负责定义H5Page,H5Data,H5Session，H5CoreNode等基本对象。主要对象说明如下：


H5CoreNode：在容器里有三种核心节点：Service、Session、Page，他们都继承自H5CoreNode：
  
```
public void setParent(H5CoreNode parent);
public H5CoreNode getParent();
public boolean addChild(H5CoreNode child);
public boolean removeChild(H5CoreNode child);
public H5PluginManager getPluginManager();
public void sendIntent(String action, JSONObject param);
```
  
Service是全局的服务类，提供容器的的入口功能（注意此Service并非钱包内的H5Service）

```
public H5Page createPage(Activity activity, Bundle params);
public boolean startPage(Activity activity, H5Bundle params);
public boolean exitService();
public boolean addSession(H5Session session);
public H5Session getSession(String sessionId);
public boolean removeSession(String sessionId);
public H5Session getTopSession();
```

Session由一系列Page组成，这些Page拥有共同的上下文，session提供了操作
这些page的方法。

```
public boolean addPage(H5Page page);
public boolean removePage(H5Page page);
public H5Page getTopPage();
public Stack<H5Page> getPages();
```

H5Page是容器的核心类，提供了一系列容器WebView相关的封装。可以通过实例化
H5Page然后动态布局Webview到页面，实现嵌套一个或者多个Webview到自定义
页面。也可以直接使用H5Activity对Webview的封装打开一个页面。

```
public H5Session getSession();
public Activity getActivity();
public View getContentView();
public String getUrl();
public String getTitle();
public H5Bridge getBridge();
public Bundle getParams();
public boolean exitPage();
public void setHandler(H5PageHandler handler);
```












终于可以独立于钱包环境单独运行。新的容器整体采用插件（Plugin）的机制构建。内部通过意图（Intent）传递消息和事件。所有的UI和逻辑模块都以插件运行在容器里。可以通过每个核心节点（Core Node）上的插件管理器来注册（register）和注销（unregister）插件。插件在添加了感兴趣的intent类型后，当有容器内部有intent发生后，插件会在拦截（intercept）和处理(handle)两个阶段被调用到。


#### 3.2 事件管理:

在容器里，所有的通信都通过意图（H5Event）进行分发。

```
public String getAction();
public String getType();
public H5CallBack getCallBack();
public H5Bridge getBridge();
public boolean sendBack(JSONObject data);
public boolean sendBack(String k, Object v);
public boolean keepSend(JSONObject data);
public boolean keepSend(String k, Object v);
public Error getError();
public boolean sendError(Error code);
```
H5Event分发分两个阶段：

1. intercept阶段，event从service->session->page方向分发，
在其中任何一个阶段如果interceptEvent返回true，即表示event
被plugin成功截获，不再继续分发；

2. handle阶段，event由page->session->service方向分发，
在其中任何一个阶段如果handleEvent返回true，即表示event被
plugin成功处理，不再继续分发；

如果分发流程走完，没有任何plugin处理该event，则event会报告
NOT_FOUND 错误。


Event处理：

Event会分发到注册了对应filter的插件，插件必须继承H5SimplePlugin：

```
class DemoPlugin extends H5SimplePlugin{
	

	@Override
	public boolean interceptEvent(H5Event event, H5BridgeContext context) {
	       // 拦截事件
		return false;
	}

	@Override
	public boolean handleEvent(H5Event event, H5BridgeContext context) {
	    // 处理事件
	    // 如果是改事件的最终处理者则返回true
		return true;
	}

	
	 @Override
    public void onPrepare(H5EventFilter filter) {
         // 添加监听的事件
        filter.addAction(START_APP);
    }
}
```

#### 3.3 插件管理:

插件全部注册在核心节点的插件管理器里，每个核心节点上都有一个插件管理器：

```
H5PluginManager pm = coreNode.getPluginManager();
pm.register(h5PagePlugin);
pm.unregister(h5SafePlugin);
```

#### 3.4 离线包管理:

离线包管理，对离线包进行验证签名，验证成功以后对资源进行解压，存放到本地。访问页面实际访问的是本地的页面资源，
通过离线包管理可以加快页面访问速度，提升用户体验。


#### 3.5 Provider管理:


容器为了将实现和依赖更好的解耦开来，实现不同开发框架下的快速切换，实现了Provider机制。
主要涉及Provider如下;
```
H5AppProvider：设置app对象的属性，包括安装路径，设置版本号等方法。
H5CacheProvider：设置容器依赖的缓存对象，包括获取缓存对象和缓存对象移除
H5ConfigProvider;设置容器依赖的配置对象,包括各种开关
H5EnvProvider:设置容器的环境信息，包括获取当前语言设置，设置RpcUrl等
H5LogProvider:设置容器依赖的日志对象
H5UaProvider:设置容器的userAgent
.....
```











### 四 加载页面

启动标准的H5页面(单独的Activity)

```
Bundle param = new Bundle();
String url = "http://ux.alipay-inc.com/ftp/h5/dawson/entry.html";
param.putString(H5Param.LONG_URL, url);
param.putBoolean(H5Param.LONG_SHOW_TITLEBAR, true);
param.putBoolean(H5Param.LONG_SHOW_TOOLBAR, true);
H5Bundle bundle = new H5Bundle();
bundle.addListener(new H5Listener(){
......
});
bundle.setParams(param);
H5Context h5Context = new H5Context(this);
Nebula.getService().startPage(h5Context, bundle);
```

嵌入一个或者多个H5Page到自定义页面

```
Bundle param = new Bundle();
param.putString(H5Param.LONG_URL, "http://ux.alipay-inc.com/ftp/h5/dawson/entry.html");
param.putString(H5Consts.SESSION_ID, "multiple_h5");
H5Bundle bundle = new H5Bundle();
bundle.setParams(param);
H5Page h5Page = h5Service.createPage(this, bundle);
h5Page.setHandler(handler);
LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(
LinearLayout.LayoutParams.MATCH_PARENT, 400);
lp.setMargins(20, 32, 20, 32);
viewGroup.addView(h5Page.getContentView(), lp);
```


### 五 开关配置
 
为了保证业务的灵活性，容器配置了很多开关来实现快速调整。主要包括

```
h5_SafePayUrls：设置调用支付的url白名单

h5_ssoLogin:设置免登的域名单

h5_AliWhiteList：设置阿里域白名单

h5_ApkWhiteList：设置apk下载名单

h5_webViewConfig：设置ucwebview使用开关

h5_preRenderMax：设置预渲染默认最大时间

h5_ucproxy_can_enable：设置是否使用uc代码开关

......

```

## 技术支持

旺旺：道深

邮箱：shenxin.dsx@alibaba-inc.com