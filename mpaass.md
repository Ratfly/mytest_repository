
# &#160;&#160;一  UC内核说明  #
     
     

>  &#160; &#160; &#160; &#160; mPaas 1.2.0容器已经支持UC内核。用户可以自由选择使用UC内核或者系统webiView.

# 二  使用方法 #
### &#160; &#160;   1. 添加依赖: ###
          <dependency>
                <groupId>com.alipay.android.phone.businesscommon</groupId>
                <artifactId>h5container-api</artifactId>
                <version>1.2.0</version>
				<scope>provided</scope>
            </dependency>


### &#160; &#160;  2. UC内核切换： ###

#### &#160; &#160;  ① debug环境下： ####

  
>  &#160; &#160; &#160; &#160; 在debug情况下，即portal下的AndroidManifest的application标签下配置了 android:debuggable="true"，容器会优先去读取吱吱客户端的配置，在吱吱客户端的配置列表最后一栏，可以配置是否使用uc内核。


#### &#160; &#160;  ② 线上环境下： ####

>  &#160; &#160;  如果不是debug情况下，外部模块需要实现H5EnvProvider接口，
prvider的实现信息。


>  &#160; &#160;  配置demo:


``` 

 private H5Service mH5Service;

 mH5Service = H5Utils.getExtServiceByInterface(H5Service.class.getName());

 // 设置UC开关
        mH5Service.getProviderManager().setProvider(H5ConfigProvider.class.getName(), new H5ConfigProvider() {
            @Override
            public String getConfig(String key) {

                if (key.equals("h5_webViewConfig")) {
                    return "{\"h5_enableExternalWebView\":\"YES\",\"h5_externalWebViewUsageRule\":{},\"h5_externalWebViewSdkVersion\":{\"min\":11,\"max\":22}}";
                } else {
                    return "";
                }
            }

            @Override
            public boolean permitLocation(String s) {
                return false;
            }

            @Override
            public boolean isAliDomains(String s) {
                return false;
            }
        });


``` 
 

>  &#160; &#160;  实现H5ConfigProvider下的 String getConfig(String key)，即自定义配置开关，其中


``` 
key="h5_webViewConfig"
```   

>  &#160; &#160;  value组成如下：

``` 
value="{\"h5_enableExternalWebView\":\"YES\",\"h5_externalWebViewUsageRule\":{},\"h5_externalWebViewSdkVersion\":{\"min\":11,\"max\":22}}")
``` 

>  &#160; &#160;  其中各节点意义如下：

``` 
h5_enableExternalWebView: 是否容许使用uc内核  
h5_externalWebViewUsageRule：用于设置对应的appid使用uc内核还是系统webview，一般默认为空，即所有业务都使用					
h5_externalWebViewSdkVersion：uc内核支持sdk版本范围，如果在当前应用此范围内则可以使用，默认为11到22
日志下可以监听tag=H5WebViewFactory，来判断是否启用uc内核
``` 

### &#160; &#160;  3. 添加uses-permission： ###

>  &#160; &#160; 由于cp上现在不容许单独的bundle添加uses-permission，需要统一通过portal配置。所以需要添加容器的uses-permission如下：


``` 
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.GET_TASKS" />
    <uses-permission android:name="android.permission.READ_CALENDAR" />
    <uses-permission android:name="android.permission.WRITE_CALENDAR" />
``` 

### &#160; &#160;  4. 打包： ###

>  &#160; &#160; 容器用到3个bundle，分别为nebula（对外的接口），nebulacore(容器核心，包括事件机制和插件等），nebulauc(uc内核），需要在portal下的pom添加打包配置如下：

``` 
       <dependency>
            <version>1.2.0</version>
            <type>xml</type>
            <groupId>com.alipay.android.phone.businesscommon</groupId>
            <classifier>AndroidManifest</classifier>
            <artifactId>h5container-build</artifactId>
        </dependency>
        <dependency>
            <version>1.2.0</version>
            <groupId>com.alipay.android.phone.businesscommon</groupId>
            <artifactId>h5container-build</artifactId>
        </dependency>
		
		 <dependency>
           <version>1.2.0</version>
            <type>xml</type>
            <groupId>com.alipay.android.phone.wallet</groupId>
            <classifier>AndroidManifest</classifier>
            <artifactId>nebulauc-build</artifactId>
        </dependency>
        <dependency>
           <version>1.2.0</version>
            <groupId>com.alipay.android.phone.wallet</groupId>
            <artifactId>nebulauc-build</artifactId>
        </dependency>
		
		<dependency>
            <version>1.2.0</version>
            <type>xml</type>
            <groupId>com.alipay.android.phone.wallet</groupId>
            <classifier>AndroidManifest</classifier>
            <artifactId>nebulacore-build</artifactId>
        </dependency>
        <dependency>
            <version>1.2.0</version>
            <groupId>com.alipay.android.phone.wallet</groupId>
            <artifactId>nebulacore-build</artifactId>
        </dependency>
      
在<project.static.link> 添加com.alipay.android.phone.businesscommon-h5container,com.alipay.android.phone.wallet-nebulacore,com.alipay.android.phone.wallet-nebulauc

``` 


### &#160; &#160;  5. 添加UC使用授权： ###

>  &#160; &#160; UC为防止内核乱用，会对使用者通过app name+签名证书作为指纹生成授权码检测。检测秘钥生成如下：

``` 
APP授权，通过签名证书+app name作为指纹生成授权码检测。
其中：对于com.UCMobile，com.ucsdk.cts，com.eg.android.AlipayGphone进行特殊放行。
首先，用户使用命令获取证书摘要：keytool -list -v -keystore 【你的.keystore】。注意到：app有debug和release两种签名，如果需要，请同时提供两个摘要。
然后，将app的包名和证书摘要一起发送给UCSDK相关人员。
接着，UCSDK相关人员使用算法生成授权码：uc私钥加密（md5( pkgname + app证书摘要 )）。
最后，用户将授权码放到AndroidMenifest.xml里面的Application设置的meta属性UCSDKAppKey中，如下代码所示。

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="com.UCMobile.ucsdk"
      android:installLocation="auto"
      android:versionCode="24"
      android:versionName="8.7.0">
    <uses-sdk android:minSdkVersion="9" android:targetSdkVersion="14" />
    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name"
        android:name="com.test.tests.SDKApp"
        android:debuggable="false"
        android:theme="@style/MyStyle">
       
        <meta-data android:name="UCSDKAppKey"
            android:value="GitsHu860Mu/tK/akIR7pUNr/Y8kM14JNLnr54646567asaCITljWRnbdg6OHjjB9k6iB8J0lWkya\nBK78tUsHHw==\n">
        </meta-data>
    </application>
</manifest>


使用者需要按照UC的步骤把需要的信息提交给他们以获得key。联系人：蔼霖
``` 

### &#160; &#160;  6. 如何判断UC内核是否生效： ###

>  &#160; &#160; ① 下拉的时候会出现UC浏览器强力驱动的provider；② 查看tag=H5WebViewFactory, 在容器启动的时候，查看相关日志，是否使用uc内核。日志如下：

``` 
09-17 11:16:07.572    7493-7493/? D/H5WebViewFactory﹕ [main]getWebViewType bizType 20000067
09-17 11:16:07.582    7493-7493/? D/H5WebViewFactory﹕ [main]debug config to enable uc webview
09-17 11:16:07.582    7493-7493/? D/H5WebViewFactory﹕ [main]create uc web view
09-17 11:16:07.752    7493-7493/? D/H5WebViewFactory﹕ [main]webViewType: (WebView type:THIRD_PARTY, version: 0.1) on device: samsung-SM-N9108V-armeabi-v7a-api21
``` 


技术支持
旺旺：道深 荣阳
