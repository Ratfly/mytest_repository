### 容器二期改造 ###

------------------
#### 1、自定义UI组件

* 目前已经支持TitleBar、ToolBar、ToolMenu、NavMenu、下拉刷新的自定义。
* 自定义的UI必须继承UI的父类,H5TitleBar、H5ToolBar、H5ToolMenu、H5NavMenu、H5PullHeader。
* 这些父类都是容器中默认的View。

----------------------------------
#### 自定义的UI如何生效？
* 实现H5ViewProvider这个接口，接口中提供了所有需要自定义UI的创建接口。实现之后的Provider必须调用 Nebula.getProviderManager().setProvider(H5ViewProvider.class.getName(),new H5ViewProviderImpl());

----------------------------------
#### 2、更改ShowDomain颜色

*接口：H5WebContentView  父类：H5WebContentViewImpl。

接入方式：继承父类，直接操作父类变量即可改变背景颜色。
