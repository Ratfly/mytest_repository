## Android Nebula H5容器SDK接入指南

### 一 简介

   Nebula（以下称“Nebula”或者“内核”）是移动端Hybrid解决方案SDK，提供了良好的外部扩展功能，拥有功能插件化、事件机制、JSApi定制和H5App推送更新管理能力。

### 二 功能

（1）加载H5页面，并按照Session的概念进行管理各个页面；

（2）提供丰富的内置JSApi，实现譬如页面push、pop，标题设置等功能；

（3）支持用户自定义JSApi和插件功能，扩展业务需求；

（4）接入H5App后台，方便管理离线或者在线H5App，实现类似钱包首页应用的推送、更新；

（5）接入H5监控平台，能够实时看到页面加载的性能、错误报警和监控；

（6）使用UCWebView,拥有解决系统级Webview Crash的能力，内存管理更合理, 网络加载提升更快，兼容性更好


### 三 如何接入

#### 3.1  引入Nebula Android容器的aar包。

#### 3.2  Nebula Android调用。


    
启动标准的H5页面(单独的Activity)

```
 Bundle param = new Bundle();
 param.putString(H5Param.LONG_URL, h5PageUrl);
 param.putBoolean(H5Param.LONG_SHOW_TITLEBAR, true);
 param.putBoolean(H5Param.LONG_SHOW_TOOLBAR, false);
 param.putBoolean(H5Param.LONG_SHOW_LOADING, false);
 param.putBoolean(H5Param.LONG_PULL_REFRESH, false);
 param.putBoolean(H5Param.LONG_SHOW_OPTION_MENU, true);
 param.putBoolean(H5Param.LONG_CAN_PULL_DOWN, true);
 param.putBoolean(H5Param.LONG_READ_TITLE, true);
 param.putString(H5Param.LONG_BACK_BEHAVIOR, "back");
 param.putString(H5Param.LONG_BIZ_SCENARIO, "dawson_scenario");    
 param.putString(H5Param.DEFAULT_TITLE, "Default Title");
 H5Bundle bundle = new H5Bundle();
 bundle.setParams(param);
 H5Context h5Context = new H5Context(this);
 // 获取容器实例
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

#### 3.3  JS API调用：

  JSApi是定义Native和JS交互的抽象，提供了H5页面调用Native功能的能力。


JS 引入： http://antui.alipay.net/antbridge/nav.html#include

JS API参数说明：(http://ux.alipay-inc.com/index.php/H5%E5%AE%B9%E5%99%A8%E6%96%87%E6%A1%A3) 



其他相关连接： (http://gitlab.alipay-inc.com/nebula/android/wikis/home)

#### 3.4  自定义插件：

   自定义插件必须继承H5SimplePlugin：

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
   
   统一注册自定义插件到插件管理器：
```   
   Set<H5Plugin> plugins = new HashSet<H5Plugin>()
   plugins.add(new DemoPlugin ());
   Nebula.getService().getPluginManager().register(plugins);
```    
 
#### 3.5  自定义view：

对外通过Provider的形式提供自定义Titlebar,webview等。

自定义Nebula View使用教程： (http://gitlab.alipay-inc.com/nebula/android/wikis/H5DEMO)


#### 3.6  其他自定义：

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

## 技术支持

旺旺：道深 

邮箱：shenxin.dsx@alibaba-inc.com