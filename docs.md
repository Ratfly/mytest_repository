## H5 容器外部插件
    为了让各个业务更加灵活的接入H5容器，H5容器提供一种可以调用外部插件实现的API的机制。
    
    容器启动的时候会读取钱包里的API配置文件。当有配置表里的API调用触发时，动态加载配置的bundle，并实例化相应plugin，分发事件到plugin。
    
    业务方使用这种方式加载自定义的外部插件，代码可以放在自己的bundle里，全过程不需要H5容器介入。
    
    Android的配置文件在 nebulacore/assets/config/h5_plugin.json

### 配置格式
```
[  
   {  
      "name":"h5Toast|h5Alert|h5Dialog",
      "bundle":"android-phone-businesscommon-bizsdk",
      "class":"com.alipay.mobile.bizsdk.h5plugin.ExtUIPlugin",
      "scope":"service"
   },
   {  
      "name":"h5ProImpl",
      "bundle":"android-phone-businesscommon-bizsdk",
      "class":"com.alipay.mobile.bizsdk.h5plugin.ExtImplPlugin",
      "scope":"page",
      "restrictApp":"20000067|20000088",
      "restrictDomain":"tmall.com|alipay.com"
   }
]
```

### 参数说明
| 参数 | 类型 | 描述 | 可选 | 默认值
| ------------ | ------------- | ------------ | ------------ | ------------ |
| name | string | API名称(同一个实现类的多个api以'\|'隔开) | N | |
| bundle | string | API实现所在bundle | N | |
| class | string | API实现所在类，必须实现H5Plugin接口 | N | |
| scope | string | API的作用域，仅限service、session、page | Y | session |
| restrictApp | array | 限制API调用的appId（`不支持正则`）,多个条件以'\|'隔开| Y | *（全部匹配） |
| restrictDomain | array | 限制API调用的域（`不支持正则`）,多个条件以'\|'隔开 | Y | *（全部匹配）|

### 注意事项
* file协议访问的本地离线包可以访问任何匹配的JSAPI，不受restrictDomain字段的限制。
* scope 默认设置为'session',如果设置为service,则bundle一旦载入后就不会释放。
* Android plugin的实现类请`不要`混淆，否则运行时会找不到对应的类。

### 参考示例
[配置源码](http://gitlab.alibaba-inc.com/shenxin.dsx/h5container/blob/master/h5_plugins.json)

[插件源码](http://gitlab.alibaba-inc.com/shenxin.dsx/h5container/blob/master/BizPlugin.java)