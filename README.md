
# 2 mPaaS IDEA Plugin
[TOC]

 ## 2.1 概述
mPaaS IDEA Plugin 是专门为mPaaS的Android开发者提供的一键部署开发插件，包括打包，修改依赖bundle版本号和依赖等，可用于IDEA与Android Studio开发工具

## 2.2 开发工具准备
1、首先要有一个java环境（需要jdk版本为1.7，1.6和1.8编译可能会有问题，已安装可忽略）

2、Android Studio开发工具，若未安装请去android开发者官网下载，建议选择zip包的免安装版（已安装可忽略）

http://developer.android.com/sdk/index.html

3、Android SDK，确保有API Level 16的版本（已安装可忽略）

若没有请在SDK Manager中下载，下载步骤：

a)启动Android Studio， 打开设置主界面（Mac系统为Preferences,windows系统为Settings）

b)搜索框中输入sdk，找到Android SDK，勾选Android 4.1.2 API为16的版本进行安装
![image](https://os.alipayobjects.com/rmsportal/aRJqvinCarBuMpc.jpg)

4、下载mvn工具（请勿跳过），下载完后解压即可，后面需要将此目录配置为Maven Home

下载地址：

https://os.alipayobjects.com/rmsportal/FwAeSNRdIJDrNIK.tar.gz



## 2.3 导入一个mPaaS工程（bundle）
mPaaS工程可以去主站勾选您想要的组建生成，也可以下载下面提供的默认demo

下载地址：

http://code.taobao.org/p/idea_plugin/src/trunk/mpaasdemo.git.zip?orig


打开Android Studio，打开File主菜单，选择import Project
![image](https://os.alipayobjects.com/rmsportal/KunzQgFHvKiUKku.jpg)

点击后，打开选择工程目录的窗口，找到mPaaS demo中的launcher（一个demo的bunlde），选择launcher下的pom文件，然后点击OK按钮。

![image](https://os.alipayobjects.com/rmsportal/lbEvsxBjFVhBiAb.jpg)

随后会打开一个创建project页面，选择右下角的"Environment Settings.."，配置上一步mvn的路径(2.2开发工具准备中第4步下载的mvn工具) ，并选择本地的mvn仓库，默认是windows的D:/m2目录，mac请自行修改自己想保存的目录。

![image](https://os.alipayobjects.com/rmsportal/vpBjluNERfXrvNe.jpg)

配置完成后点击ok然后next，后面的三个页面可以一直点击"next"，最后finish就可以了

导入完成后，不出意外会右侧看到一个"Frameworks dectected"的小气泡提示，点击Configure,将此项目配置成一个Android工程，然后就可以继续安装插件了。

![image](https://os.alipayobjects.com/rmsportal/fPeRnjeAVbiWOuu.jpg)

## 2.4 插件的安装与配置

mPaaS IDEA Plugin的安装十分简单，与普通的IDEA插件安装过程一样，可以通过本地zip包安装，也可通过网络下载安装插件。插件下载地址：http://code.taobao.org/p/idea_plugin/src/trunk/mpaas_plugin/mPaaSPlugin.zip?orig


第一步

打开Android Studio，点击Android Studio主菜单，选择Preferences（windows系统为Settings）选项，打开设置主界面

![image](https://t.alipayobjects.com/images/rmsweb/T1WFtiXbJXXXXXXXXX.jpg )


第二步

在配置主界面选择Plugins，然后在右下方选择Install plugs from disk...

![image](https://t.alipayobjects.com/images/rmsweb/T1C.xhXXlgXXXXXXXX.jpg )

然后选择本地的mPaaS-plugin.zip插件包，选择后点击OK或Apply，随后会提示重启（restart）Android Studio 生效

![image](https://t.alipayobjects.com/images/rmsweb/T1qFtiXfBXXXXXXXXX.jpg )

第三步


重启后，再次打开设置主界面（Mac系统为Preferences,windows系统为Settings），即可看到mPaaS插件已经安装完毕

![image](https://os.alipayobjects.com/rmsportal/oDllWpRzmyFpKtV.png)

此处配置Android Sdk Home路径，portal工程路径，Maven Home路径（若导入工程时候已选则不需要配置），并输入mPaaS公网的mvn仓库用户名和密码，点击apply即可

![image](https://os.alipayobjects.com/rmsportal/FNLpOcuUrLWxiOP.jpg)

## 2.5 插件的使用
### 2.5.1 编译Bundle与Portal

配置完成后，可以看到Android Studio多了一个mPaaS的下拉菜单，分别有三个选项

![image](https://os.alipayobjects.com/rmsportal/uEmrJgGgEdorxuA.jpg)


1、Build bundle : 编译当前Bundle

2、Build Portal and Run : 编译Portal并且将最终的apk部署在手机上

3、Build Setting 配置当前Bunlde的依赖Api接口管理，与编译Bundle的设置，如下图：

![image](https://os.alipayobjects.com/rmsportal/UKVViKkshFJTFig.jpg)

这个可以修改依赖api的版本号，查看每个版本的changeLog等，下面是编译Bundle的编译选项，若要进行debug，需要将“跳过混淆”勾选。前期开发建议几个都勾选，这样可以加快编译速度。

### 2.5.2 Portal的管理

再次打开设置主界面（Mac系统为Preferences,windows系统为Settings），可以看到面板下方多了一个选项卡，这里可以管理portal依赖的mPaaS中Bundle的版本号和依赖关系，右侧的Edit小icon支持输入定制的小版本号。

![image](https://os.alipayobjects.com/rmsportal/bSAYvKtjODwyNtb.png)


### 2.5.3 使用插件创建工程

一 、创建portal:

安装完成idea plugin之后便可以在开发工具中开始创建项目了，首先需要创建整个工程的入口项目：portal
![image](https://t.alipayobjects.com/images/rmsweb/T1nptiXXNcXXXXXXXX.jpg)
![image](https://t.alipayobjects.com/images/rmsweb/T1co0hXeXfXXXXXXXX.png)
![image](https://t.alipayobjects.com/images/rmsweb/T1TUJhXXBhXXXXXXXX.png)
导入成功之后右上角如果出现notification对话框，就点击配置一下
![image](https://t.alipayobjects.com/images/rmsweb/T1zohhXbXlXXXXXXXX.png)
成功导入项目后，打开terminal 输入 gradle clean install 编译一下项目
![image](https://t.alipayobjects.com/images/rmsweb/T1RpliXolcXXXXXXXX.png)
出现BUILD SUCCESSFUL说明项目创建成功
![image](https://t.alipayobjects.com/images/rmsweb/T1fFNiXdFaXXXXXXXX.png)

二 、创建bundle

请选中New -> mPass Projects -> Bundle Project
![image](https://t.alipayobjects.com/images/rmsweb/T1nptiXXNcXXXXXXXX.jpg)
填写创建bundle需要的参数：参数说明在基础术语中和文档输入框中有说明
![image](https://t.alipayobjects.com/images/rmsweb/T1ME0hXkleXXXXXXXX.png)

可以选择编译类型：Gradle 或者 Maven，点击创建，插件会自动创建编译类型相符的工程并导入到工程中。
导入成功之后在terminal中输入gradle clean install 或者 mvn clean install(如果build type是maven的话)。出现Build SUCCESSFUL标识创建成功.







