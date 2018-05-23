---
title: jfinal源码04：web之Interceptor
date: 2018-05-23 14:40:54
categories: JFinal
---
还记得我们第03节的Controller的初始化过程吗？
<!-- more -->
Jfinal类
```
private void initActionMapping() {
// 从配置文件获取路由  初始化actionMapping
        this.actionMapping = new ActionMapping(Config.getRoutes());
//  把路由分别拆分成action映射（利用反射）。 这就完成了路径到controller方法的映射
// 这一步同时完成了action的初始化
        this.actionMapping.buildActionMapping();
        Config.getRoutes().clear();
    }
```
```
protected void buildActionMapping() {
        this.mapping.clear();
        Set<String> excludedMethodName = this.buildExcludedMethodName();
        InterceptorManager interMan = InterceptorManager.me();
        Iterator var3 = this.getRoutesList().iterator();

        while(var3.hasNext()) {
            Routes routes = (Routes)var3.next();
            Iterator var5 = routes.getRouteItemList().iterator();

            while(var5.hasNext()) {
                Route route = (Route)var5.next();
                Class<? extends Controller> controllerClass = route.getControllerClass();
                Interceptor[] controllerInters = interMan.createControllerInterceptor(controllerClass);
                boolean sonOfController = controllerClass.getSuperclass() == Controller.class;
                Method[] methods = sonOfController ? controllerClass.getDeclaredMethods() : controllerClass.getMethods();
                Method[] var11 = methods;
                int var12 = methods.length;

                for(int var13 = 0; var13 < var12; ++var13) {
                    Method method = var11[var13];
                    String methodName = method.getName();
                    if (!excludedMethodName.contains(methodName) && method.getParameterTypes().length == 0 && (!sonOfController || Modifier.isPublic(method.getModifiers()))) {
                  // 获取每一个action（方法的拦截器list）
                        Interceptor[] actionInters = interMan.buildControllerActionInterceptor(routes.getInterceptors(), controllerInters, controllerClass, method);
                        String controllerKey = route.getControllerKey();
                        ActionKey ak = (ActionKey)method.getAnnotation(ActionKey.class);
                        String actionKey;
                        if (ak != null) {
                            actionKey = ak.value().trim();
                            if ("".equals(actionKey)) {
                                throw new IllegalArgumentException(controllerClass.getName() + "." + methodName + "(): The argument of ActionKey can not be blank.");
                            }

                            if (!actionKey.startsWith("/")) {
                                actionKey = "/" + actionKey;
                            }
                        } else if (methodName.equals("index")) {
                            actionKey = controllerKey;
                        } else {
                            actionKey = controllerKey.equals("/") ? "/" + methodName : controllerKey + "/" + methodName;
                        }
      // 初始化化了action，**这里就初始化了action上的interceptors数组**
                        Action action = new Action(controllerKey, actionKey, controllerClass, method, methodName, actionInters, route.getFinalViewPath(routes.getBaseViewPath()));
                        if (this.mapping.put(actionKey, action) != null) {
                            throw new RuntimeException(this.buildMsg(actionKey, controllerClass, method));
                        }
                    }
                }
            }
        }

        this.routes.clear();
        Action action = (Action)this.mapping.get("/");
        if (action != null) {
            this.mapping.put("", action);
        }

    }
```

具体看下Inteceptor init 流程
- config file： me.add(new AuthInterceptor());默认添加全局拦截器
- InterceptorManager 类里面维护globalActionInters数组
- 初始化Action时会调用InterceptorManager类的doBuild（），此时全局的拦截器就绑定在action的interceptors数组里
- ActionHandler类：(new Invocation(action, controller)).invoke();调用目标方法时就会按顺序调用 this.inters[this.index++].intercept(this);

总结一下：
Handler和Inteceptor区别？
Handler会拦截所有的web请求，是一个处理链的结构，链的结尾就是ActionHandler
全局Inteceptor绑定在每一个Controller中的每一个方法（action）上,每个action都会维护一个拦截器数据。会不会比较耗内存呢。
启示：
所以全局的操作最好定义handler，局部的操作定义inteceptor。减少对内存的消耗。
补充：(这里是否能这么做还是一个疑问，因为Handler会拦截所有的web请求，包括静态资源)。