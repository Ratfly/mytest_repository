## Android H5容器文档

### 概述

H5容器经过钱包8.4版本的重构内核后，终于可以独立于钱包环境单独运行。新的容器整体采用插件（Plugin）的机制构建。内部通过意图（Intent）传递消息和事件。所有的UI和逻辑模块都以插件运行在容器里。可以通过每个核心节点（Core Node）上的插件管理器来注册（register）和注销（unregister）插件。插件在添加了感兴趣的intent类型后，当有容器内部有intent发生后，插件会在拦截（intercept）和处理(handle)两个阶段被调用到。

### 核心节点

在容器里有三种核心节点：Service、Session、Page，他们都继承自H5CoreNode：
  
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

### Event分发

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


### Event处理

Event会分发到注册了对应filter的插件，插件必须实现H5Plugin接口：

```
class DemoPlugin implements H5Plugin{
	@Override
	public void onRelease() {
	    // release resources on plugin destoryed
	}

	@Override
	public boolean interceptEvent(H5Event event, H5BridgeContext context) {
	    // do something here to intercept event
	    // return true if event been intercepted
		return false;
	}

	@Override
	public boolean handleEvent(H5Event event, H5BridgeContext context) {
	    // do something here to handle event
	    // return true if event been handled
		return false;
	}

	@Override
	public void getFilter(H5IntentFilter filter) {
	    // add event filter
	    filter.addAction("intent_type");
	}
}
```

插件全部注册在核心节点的插件管理器里，每个核心节点上都有一个插件管理器：

```
H5PluginManager pm = coreNode.getPluginManager();
pm.register(h5PagePlugin);
pm.unregister(h5SafePlugin);
```

## 启动H5页面

启动标准的H5页面(单独的Activity)

```
Bundle param = new Bundle();
String url = "http://ux.alipay-inc.com/ftp/h5/dawson/entry.html";
param.putString(H5Param.LONG_URL, url);
param.putBoolean(H5Param.LONG_SHOW_TITLEBAR, true);
param.putBoolean(H5Param.LONG_SHOW_TOOLBAR, true);
H5Bundle bundle = new H5Bundle();
bundle.addListener(h5Listener);
param.setParams(param);
h5Service.startPage(activity, param);
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


## 技术支持

旺旺：道深

邮箱：shenxin.dsx@alibaba-inc.com