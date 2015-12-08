## H5接入FAQ

1.  H5容器和Android WebView有什么区别？

    其实是没什么区别，H5容器就是对WebView进行了一系列的封装。这些包括WebView的参数设置、安全控制、url控制、JSBridge实现等。
    使用H5容器的WebView和Android WebView是一样的。

2.  H5容器的WebView可不可以在xml定义？

    暂时不能,H5容器也不推荐这么做。

3.  H5容器只能在单独的一个activity里运行么？
    
    不是的，业务可以根据需求把H5 WebView嵌入到自己的布局里去，并且支持业务嵌入一个或者多个H5WebView到自定义布局，使用方式和Android里代码添加View是一样的。

4.  H5容器我需要再去设置那一堆乱七八糟的参数么？

    正常来讲，大多数业务不需要了，当然要自己去设置也是可以。

5.  H5容器里我怎么去注册plugin？
    
    在H5容器里，Service、Session、Page都是核心节点，他们都可以注册plugin上去：

    ```
	H5BasePlugin plugin = new H5BasePlugin();
	h5Page.getPluginManager().register(plugin);
	```

6.  如何去获取H5相关的状态？
    
    请不要自己去设置WebView的WebViewClient和WebChromeClient， 容器已经做好了封装，WebView相关的状态改变都以intent的方式发送到plugin上去。你只需要在page里注册相应的filter就可以收到这些intent啦。
    
    通过注册插件来监听，具体的intent类型都定义在H5Plugin文件里，比如获取WebView的加载进度就可以这样：
    
    ```
    class ProgressPlugin implements H5Plugin {
      public void onInitialize() {  
      }

		public void onRelease() {
		}
		
		public void onPrepare(H5EventFilter filter) {
		}
		
		public boolean interceptEvent(H5Event event, H5BridgeContext bridgeContext) {
			JSONObject param = event.getParam();
			String action = event.getAction();
			if (H5Plugin.H5_PAGE_PROGRESS.equals(action)) {
				int progress = param.getIntValue("progress");
				// update your progress here
			}
			return false;
		}
		
		public boolean handleEvent(H5Event event, H5BridgeContext bridgeContext) {
			return false;
		}
	}
    ```
7.  H5Event都是从哪里产生的？

    H5Event是对容器内部产生的事件的封装，它可能来自于javascript通过bridge的调用也可能是webview内部的状态改变，也可能是别的模块通过核心节点发过来的啦。也就是说H5Event可以是javascript调用java，也可以是java调用javascript，也可以是java和java之间互相调用。

8.  interceptIntent和hanldeIntent方法我应该在什么时候返回false，什么时候返回true？

    一般来讲，如果你需要在某些场景（比如安全、校验、白名单控制）下拦截一个intent，你应当在intercept里处理并在必要时返回true以结束intent的分发。如果你只是需要`知道`这个intent，并获得某些状态的情况，应该在intercept里去实现自己的逻辑，并返回false。如果你的plugin是这个intent的`明确`处理者，你应当
    在handleIntent里去实现并返回true。
    
9. 怎么用H5容器打开一个url？
    
    H5容器只需要你传进来一个url参数，就可以帮你打开这个页面啦：
    
    ```
    String url = "http://ux.alipay-inc.com/ftp/h5/dawson/entry.html";
    Bundle param = new Bundle();
    param.putString(H5Param.LONG_URL, url);
    H5Bundle bundle = new H5Bundle();
    bundle.setParams(param);
    H5Service h5Service = AlipayApplication.getInstance()
        .getMicroApplicationContext()
        .getExtServiceByInterface(H5Service.class.getName());
    h5Service.startPage(activity, bundle);
    ```
    是的，只需要上面几句代码，你就可以打开一个页面了，不需要设置各种Webview相关的参数设置，不需要担心安全漏洞，也不需要去实现jsbridge。


10. 在钱包里怎么创建一个H5容器的page？

    ```
    Bundle param = new Bundle();
    param.putString(H5Param.LONG_URL,
    		"http://ux.alipay-inc.com/ftp/h5/dawson/entry.html");
    param.putString(H5Param.SESSION_ID, "multiple_h5");
    H5Bundle bundle = new H5Bundle();
    bundle.setParams(param);
    H5Service h5Service = AlipayApplication.getInstance()
    		.getMicroApplicationContext()
    		.getExtServiceByInterface(H5Service.class.getName());
    H5Page h5Page = h5Service.createPage(this, bundle);
    ```

11. 怎么获得WebView？
    
    非H5容器是不能直接获得WebView的，但是可以通过H5Page的getContentView方法获得包含WebView的View。
    
12. 页面里有多个H5 WebView时的注意事项？

    当你的页面有多个H5Page时，你应当设置H5Page的handler,当单个page需要退出的时候，你需要在handler里返回false，否则默认会退出page所在的activity：
    
    ```
    h5Page.setHandler(new H5Page.H5PageHandler() {
        @Override
        public boolean shouldExit() {
            // the activity will finish if return true
            return false;
        }
    });
    ```
    
13.  启动一系列的H5页面怎么监听页面的生命周期？

    在启动参数H5Bundle里添加listener，以便掌握H5容器里的Session和Page的生命周期，在合适的时候注入plugin。

    ```
    H5BaseListener listener = new H5BaseListener();
    bundle.addListener(listener);
    ```

14. 如果页面主动退出，我需要干啥？

    如果页面主动退出，请记得在onDestroy里调用h5Page.exitPage()方法，否则会导致H5容器内存泄漏。。。

15. H5都支持那些启动参数以及怎么设置？

    参考H5文档: http://ux.alipay-inc.com/index.php/H5容器文档
  
16. 和页面无关的插件怎么注册？
    
    和页面无关的业务逻辑插件可以直接注册到H5Service上：
    
    ```
    H5BasePlugin plugin = new H5BasePlugin();
    h5Service.getPluginManager().register(plugin);
    ```
17. 容器是否支持缓存webview页面
    
    不支持。

18. 如果还有问题？

    请联系:
    
    道深(shenxin.dsx@alibaba-inc.com)
    
    希德(xide.wf@alibaba-inc.com)