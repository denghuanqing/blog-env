---
title: jfinal源码03：web之Controller
date: 2018-05-23 14:39:03
categories: JFinal
---
Controller层聚合了相同功能模块的action（contriller里面的方法）。
<!-- more -->
##### 1.类结构
成员方法分为3类
-  从request获取|设置参数
-  操作cookie （set&get）
-  render系列方法：给controller绑定不同的试图渲染实例
总结起来就是操作request、response.也就是servlet规范的薄封装。
##### 2.维护Controller映射之ActionMapping
jfinal框架启动时会初始化ActionMapping：
```

```
具体的流程如下：
- 配置文件 routes.add("/user", UserController.class);
 - JFinalFilter jfinal.init(this.jfinalConfig, filterConfig.getServletContext());
-  Jfinal类
```
private void initActionMapping() {
// 从配置文件获取路由  初始化actionMapping
        this.actionMapping = new ActionMapping(Config.getRoutes());
//  把路由分别拆分成action映射（利用反射）。 这就完成了路径到controller方法的映射
        this.actionMapping.buildActionMapping();
        Config.getRoutes().clear();
    }
```
- ActionHandler 类Action action = this.actionMapping.getAction(target, urlPara);就能取到