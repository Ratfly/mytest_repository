# 容器一期改造 #


----------


### 一 改造背景 ###

现有容器很多业务其实是独立于钱包框架的，可以将现有容器代码去掉依赖于钱包的部分，将容器单独独立出来，开放给更多的开发者使用，不只局限于集团内部。
容器一期改造只涉及全部在容器内部实现的模块，不改造依赖其他模块的部分。


----------


### 二 改造原则 ###

尽量和现有容器的使用方式保持一致。


----------


### 三 改造工程包结构 ###

以道深师兄之前单独的H5容器工程为基础，进行扩展。
Gitlab地址：http://gitlab.alibaba-inc.com/h5/h5container
下载导入即可运行。基本原则就是在这个工程的基础上，进行开发。将没有的功能移植到单独的容器工程中。


----------


### 四 改造方案 ###

#### 4.1 接入方式
钱包容器的实现机制：
url启动，通过url传启动参数，               通过框架提供的schemeService或者startapp启动

独立容器实现机制：
	H5Container.getService().startPage(h5Context, bundle); 

----------

#### 4.2 启动参数（只保留第一期要做的）
H5容器运行时的外观和行为受一组参数控制，可在启动一个新实例或者pushWindow时指定。
示例：__webview_options__=showToolBar%3DNO%26showTitleBar%3DYES // urlencode('showToolBar=YES&showTitleBar=NO') => showToolBar%3DNO%26showTitleBar%3DYES
另外，对于新实例的第一个url, 也可以带一个魔法参数__webview_options__ ， 其内容将被容器取出，传给容器本身



名称          | 缩写    | 类型  | 说明 
------------- | -------------  | ------------- | -------------
url           |	 u     | string|  起始url  
defaultTitle  |  dt    |   string |	默认标题, 在页面第一次加载之前显示在标题栏上
showTitleBar  |  st    |   string|	YES/NO, 是否显示顶部标题栏
showLoading	|sl|	string|	YES/NO, 是否在页面加载前显示全局菊花
closeButtonText|	cb|	string|	工具条上的关闭按钮文案
readTitle|		rt|		string|		YES/NO, 是否读取网页标题显示在titleBar上
bizScenario|		bz|		string|		业务场景。用户的相关配置(如设置的字体大小等)将以此为key保存                              
backBehavior|		bb|		string|		back/pop/auto. 指定后退按钮行为
pullRefresh|		pr|		string|		YES / NO.是否支持下拉刷新
showProgress|		sp|		bool|		YES/NO，是否显示加载的进度条
canPullDown|		pd|		bool|		YES/NO，页面是否支持下拉（显示出黑色背景或者域名）
showDomain|		sd|		bool|		YES/NO，页面下拉时是否显示域名
backgroundColor|		bc|		int|		设置背景颜色（十进制，例如：bc=16775138）
showOptionMenu|		so|		bool|		YES/NO，是否显示右上角点点点按钮
showTitleLoading|		tl|		bool|		YES/NO，是否在TitleBar的标题左边显示小菊花）


----------
             
#### 4.3 服务端配置


通过H5ConfigProvider的机制自由切换钱包和外在的服务器环境。（详细实现机制待定）

----------
#### 4.4 容器功能

#### 反钓鱼 
* 容器接收一个bool型的参数antiPhishing, 指明一次http会话是否为信任模式
- Native App通过startActivity/createWebviewController方式接入容器时，应设置antiPhishing为false
- 通过url scheme方式启动的容器实例不能指定bool型参数，因此antiPhishing一定是true
* 当antiPhishing为true时 
- 1.如果第一个url的protocol不是http/https, 将拒绝打开(无提示) 
- 2.如果第一个url不能匹配h5_EntryWhitelist的任何项，将拒绝打开(无提示)
- 3.每遇到一个不在h5_EntryWhitelist中的新的域名，容器在打开该页面的同时，将在后台请求远程请求反钓鱼服务; 如果反钓鱼服务报异常，容器将退出（提示: "您正在查看的内容存在安全风险，无法使用钱包继续浏览", "关闭", "用浏览器打开") 

----------
#### 安全提示

- 用户在网页上填写表单，弹出键盘时，toast提示“请勿输入支付宝登录或支付密码”。参照微信
- 对于h5_InputWarningWhitelist中的url，不弹出此提示  

----------
#### 环境变量

- navigator.userAgent

包含钱包版本信息，如
>  Mozilla/5.0 (Linux; U; Android 4.2.1; ... Safari/534.30 AlipayClient/8.1.0.032000 

----------
- AlipayJSBridge.startupParams (8.2)

包含应用启动参数，如
>  {"st": "YES", "sb": "NO", "url": "http://m.baidu.com/"}

----------
- sharedData (8.2)

可通过H5Service读取或者写入数据，使用jsapi读取（参见setSharedData/getSharedData）


----------

4.5 扩展事件

- AlipayJSBridgeReady

window.onload以后，容器会初始化，产生一个全局变量AlipayJSBridge, 然后触发此事件

``` java
document.addEventListener('AlipayJSBridgeReady', function () {
  console.log(typeof AlipayJSBridge);
}, false);
```

- optionMenu

导航条右上角按钮被点击时触发

``` java
document.addEventListener('optionMenu', function () {
  // ... do something
}, false);

```


- resume

当一个webview界面重新回到栈顶时，会触发此事件. 如果这个界面是通过popWindow/popTo到达，且传递了data参数，此页可以收到
如果在界面的resume之前先发生了app的resume, 则event还会有一个resumeParams, 包含app resume时接收到的参数

``` java
document.addEventListener('resume', function (e) {
  console.log(e.data);
  console.log(e.resumeParams);
}, false);

```

- resume with data

导航条右上角按钮被点击时触发

``` 
/ popWindow示例
AlipayJSBridge.call('popWindow', {
  data: {hello: 1}
});
// popTo示例
AlipayJSBridge.call('popTo', {
  index: -1,
  data: {hello: 1}
});
// 目标页代码
document.addEventListener('resume', function (event) {
  console.log(event.data);
  console.log(event.resumeParams);
});

```


- back

用户点击导航栏左上角返回按钮或者Android的物理返回键，页面将会收到此事件
如果在事件的处理函数中调用了event.preventDefault()，容器将忽略backBehaviour，js需要负责回退或做其他操作

``` 
document.addEventListener('back', function (e) {
  e.preventDefault();
  AlipayJSBridge.call('popTo', {index: 0});
}, false);

```


- titleClick

事件描述：点击标题触发回调

``` 
document.addEventListener('titleClick', function () {
	
}, false);

```


- subtitleClick

subtitleClick事件回调会触发titleClick回调，建议titleClick 和 subtitleClick只使用其中一个.
事件描述：点击子标题触发回调

``` 
document.addEventListener('subtitleClick', function () {
	
}, false);

```

- downloadEvent



``` 
document.addEventListener('downloadEvent', function (event) {
	console.log(event.url);
	console.log(event.status);
	console.log(event.progress);
}, false);

```
----------

4.6 扩展接口（详细的接口参数见http://ux.alipay-inc.com/index.php/H5容器文档）

``` 

getNetworkType (获取网络状态)

titlebar (控制标题栏)

setOptionMenu (设置右按钮属性)

pushWindow(窗口控制）

loading

title loading 

toast (弱提示)

sessionData (会话数据)

showAlert (对话框)

Alert (对话框)

Confirm (选择对话框)

checkJSAPI (JSApi可用性判断)

isInstalledApp (外部应用存在性判断)

popTo (退回指定界面)

openInBrowser (浏览器中打开)

getConfig (获取配置数据)})  接口保存，格式待议

rsa (RSA加解密)

vibrate (调用震动)

watchShake (摇一摇)

getClientInfo (获取客户端信息)

startPackage (通过包名启动app)

SharedData (共享数据)

clipboard (剪贴板)

exitSession (退出应用内所有H5页面)

httpRequest (网络请求)

ping

snapshot（截屏）

addNotifyListener （添加native通知的监听）


```