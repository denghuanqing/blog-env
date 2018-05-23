---
title: jfinal源码01：接管web请求
date: 2018-05-23 14:26:57
categories: JFinal
---
最近年前不是很忙，同时感觉自己有点迷失。少了刚出学校的那份激情。这种思想挺危险的，所以就决定沉寂一下自己的心灵。好好看看大神的代码。然后坚持写自己技术文章的第一个系列-jfinal源码之路。2017.12.10
<!-- more -->
开始正题
1.1 web.xml文件入口
````
<filter>
        <filter-name>jfinal</filter-name>
        <filter-class>com.jfinal.core.JFinalFilter</filter-class>
        <init-param>
            <param-name>configClass</param-name>
            <param-value>**********.JfinalTemplateConfig</param-value>
        </init-param>
    </filter>
````
web.xml文件配置了拦截器JFinalFilter实现了Filter接口  可以拦截所有的强求。
重点关注JFinalFilter类的doFilter方法
````
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)req;
        HttpServletResponse response = (HttpServletResponse)res;
        request.setCharacterEncoding(this.encoding);
        String target = request.getRequestURI();
        if (this.contextPathLength != 0) {
            target = target.substring(this.contextPathLength);
        }

        boolean[] isHandled = new boolean[]{false};

        try {
//这一句是重点，jfinal自定义了Handler抽象类，用来约束所有处理web请求的规范。也是所有web请求的默默处理者
            this.handler.handle(target, request, response, isHandled);
        } catch (Exception var10) {
            if (log.isErrorEnabled()) {
                String qs = request.getQueryString();
                log.error(qs == null ? target : target + "?" + qs, var10);
            }
        }
    // 直接放行web请求
        if (!isHandled[0]) {
            chain.doFilter(request, response);
        }

    }
````
1.2好了  成功找到入了  我们来看Handler到底是怎么处理这些web请求的
我们知道Handler有多个实现类 并且jfinal支持自定义handler 那么框架怎么知道最先执行那个handler呢，我们看JFinal类的initHandler()方法
````  
//new 了一个actionHandler 从名字上看  是处理路由的  具体什么功能  我们后面再来看
      Handler actionHandler = new ActionHandler(this.actionMapping, this.constants);
// 这个方法就返回了handler处理链的第一个handler
        this.handler = HandlerFactory.getHandler(Config.getHandlers().getHandlerList(), actionHandler);
````
继续看看HandlerFactory.getHandler()的实现
````
 public static Handler getHandler(List<Handler> handlerList, Handler actionHandler) {
//  这是框架new出来的路由handler
        Handler result = actionHandler;
// handlerList就是 用户自己配置的handler  List集合
  Handler 类的成员变量nextHandler实现了类似于链表的结构，用于实现请求的下一个处理节点
所以这个for循环的意义就是  用户配置了handler0 handler1 handler2
请求的执行顺序就是 0->1->2->actionHandler  也就是自定的handler是有处理顺序的，并且优先于框架内部的handler
        for(int i = handlerList.size() - 1; i >= 0; --i) {
            Handler temp = (Handler)handlerList.get(i);
            temp.nextHandler = result;
            result = temp;
        }

        return result;
    }
````
1.3 好了 找到罪魁祸首的ActionHandler 来看看具体的实现:
````
public final void handle(String target, HttpServletRequest request, HttpServletResponse response, boolean[] isHandled) {
// 这一句暂时没明白为什么要判断 uri中没有.才能进行处理
        if (target.indexOf(46) == -1) {
// 修改handler的钩子 表示此请求已经被处理
            isHandled[0] = true;
            String[] urlPara = new String[]{null};
// 根据路由获取对应的action= this.actionMapping.getAction(target, urlPara);
            if (action == null) {
                if (log.isWarnEnabled()) {
                    String qs = request.getQueryString();
                    log.warn("404 Action Not Found: " + (qs == null ? target : target + "?" + qs));
                }

                renderFactory.getErrorRender(404).setContext(request, response).render();
            } else {
                String qs;
                String qs;
                try {
                // 根据action 示例化controller
                    Controller controller = (Controller)action.getControllerClass().newInstance();
                    controller.init(request, response, urlPara[0]);
                    if (this.devMode) {
                        boolean isMultipartRequest = ActionReporter.reportCommonRequest(controller, action);
                        (new ActionInvocation(action, controller)).invoke();
                        if (isMultipartRequest) {
                            ActionReporter.reportMultipartRequest(controller, action);
                        }
                    } else {
                      // 先执行action上的拦截器 ，在执行action的方法  从这里我们可以看出 Action类就是jfinal处理请求的最小粒度。  后面重点理解Action类的生命周期
                        (new ActionInvocation(action, controller)).invoke();
                    }

                    Render render = controller.getRender();
                    if (render instanceof ActionRender) {
                        qs = ((ActionRender)render).getActionUrl();
                        if (target.equals(qs)) {
                            throw new RuntimeException("The forward action url is the same as before.");
                        }

                        this.handle(qs, request, response, isHandled);
                        return;
                    }

                    if (render == null) {
                        render = renderFactory.getDefaultRender(action.getViewPath() + action.getMethodName());
                    }
                      // render 单元暂时也没明白具体怎么渲染 下一节通过bebug分别理解疑惑
                    render.setContext(request, response, action.getViewPath()).render();
                } catch (RenderException var10) {
                    if (log.isErrorEnabled()) {
                        qs = request.getQueryString();
                        log.error(qs == null ? target : target + "?" + qs, var10);
                    }
                } catch (ActionException var11) {
                    int errorCode = var11.getErrorCode();
                    if (errorCode == 404 && log.isWarnEnabled()) {
                        qs = request.getQueryString();
                        log.warn("404 Not Found: " + (qs == null ? target : target + "?" + qs));
                    } else if (errorCode == 401 && log.isWarnEnabled()) {
                        qs = request.getQueryString();
                        log.warn("401 Unauthorized: " + (qs == null ? target : target + "?" + qs));
                    } else if (errorCode == 403 && log.isWarnEnabled()) {
                        qs = request.getQueryString();
                        log.warn("403 Forbidden: " + (qs == null ? target : target + "?" + qs));
                    } else if (log.isErrorEnabled()) {
                        qs = request.getQueryString();
                        log.error(qs == null ? target : target + "?" + qs, var11);
                    }

                    var11.getErrorRender().setContext(request, response).render();
                } catch (Throwable var12) {
                    if (log.isErrorEnabled()) {
                        qs = request.getQueryString();
                        log.error(qs == null ? target : target + "?" + qs, var12);
                    }

                    renderFactory.getErrorRender(500).setContext(request, response).render();
                }

            }
        }
    }
````
小节：jfinal对web请求始于servlet的Filter  中间链式调用自定义Handler的实现类   最后通过ActionHandler找到路由映射，具体处理业务。（ActionHandler里面包含了路由的映射，参数的获取，拦截器的执行，结果试图的渲染。是一个重点）
