---
title: jfinal源码02：处理web请求
date: 2018-05-23 14:38:06
categories: JFinal
---
通过01节，我们知道handler处理链最后都是来到ActionHandler处理具体的业务逻辑，这节重点关注ActionHandler类：
<!-- more -->
#####1.1 ActionHandler之成员变量ActionMapping
![image.png](http://upload-images.jianshu.io/upload_images/7027953-b8aadc0aff6600a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从调试过程中可以看出 ActionMapping维护了3个集合。每一个web请求的uri当作key，Controller里面的每一个方法为value（Action类的实体），组成了mapping集合。
接下来看看Action的具体实现：
![image.png](http://upload-images.jianshu.io/upload_images/7027953-c280e7006d3a0f7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一个action实例对应一个controller中的一个方法，是处理每一个web请求的最小单位！
ActionHandler类的handle方法中主要的一行代码
````
(new ActionInvocation(action, controller)).invoke();

看下实现：
public void invoke() {
        if (this.index < this.inters.length) {
            // 链式调用作用于该方法上的拦截器
            this.inters[this.index++].intercept(this);
        } else if (this.index++ == this.inters.length) {
            try {
                // 调用目标方法
                this.action.getMethod().invoke(this.controller, NULL_ARGS);
            } catch (InvocationTargetException var3) {
                Throwable cause = var3.getTargetException();
                if (cause instanceof RuntimeException) {
                    throw (RuntimeException)cause;
                }

                throw new RuntimeException(var3);
            } catch (RuntimeException var4) {
                throw var4;
            } catch (Exception var5) {
                throw new RuntimeException(var5);
            }
        }

    }
````

#####1.2 ActionHandler之渲染-业务逻辑执行完，结果渲染回浏览器
````
// 此处可以发现render是和Controller绑定在一起的  是通过controller类里面的render绑定上去的
Render render = controller.getRender(); 
render.setContext(request, response, action.getViewPath()).render(); // 渲染方法
````
这里拓展一下试图渲染的流程
RenderFactory类的init方法
````
 public void init(Constants constants, ServletContext servletContext) {
        this.constants = constants;
        servletContext = servletContext;
        Render.init(constants.getEncoding(), constants.getDevMode());
        this.initFreeMarkerRender(servletContext);
        this.initVelocityRender(servletContext);
        this.initJspRender(servletContext);
        this.initFileRender(servletContext);
// mainRenderFactory  渲染工厂  支持自己扩展 可以通过两种方式来扩展
//1.自定义render工厂me.setMainRenderFactory(new SqqRenderFactory());
//2.通过设置viewqType 来分别实例化不用的RenderFactory
        if (mainRenderFactory == null) {
            ViewType defaultViewType = constants.getViewType();
            if (defaultViewType == ViewType.FREE_MARKER) {
                mainRenderFactory = new RenderFactory.FreeMarkerRenderFactory();
            } else if (defaultViewType == ViewType.JSP) {
                mainRenderFactory = new RenderFactory.JspRenderFactory();
            } else {
                if (defaultViewType != ViewType.VELOCITY) {
                    throw new RuntimeException("View Type can not be null.");
                }

                mainRenderFactory = new RenderFactory.VelocityRenderFactory();
            }
        }

        if (errorRenderFactory == null) {
            errorRenderFactory = new RenderFactory.ErrorRenderFactory();
        }

    }

````

看一下freemarker 支持的集中底层渲染工厂FreeMarkerRenderFactory，JspRenderFactory，VelocityRenderFactory  也可以自定义
````
private static final class FreeMarkerRenderFactory implements IMainRenderFactory {
        private FreeMarkerRenderFactory() {
        }
      
      // 返回具体的渲染类
        public Render getRender(String view) {
            return new FreeMarkerRender(view);
        }

        public String getViewExtension() {
            return ".html";
        }
    }
````
继续跟新具体的渲染类FreeMarkerRender类的render方法，通过输出流像浏览器输出
````
this.response.setContentType(contentType);
        Map root = new HashMap();
        Enumeration attrs = this.request.getAttributeNames();

        while(attrs.hasMoreElements()) {
            String attrName = (String)attrs.nextElement();
            root.put(attrName, this.request.getAttribute(attrName));
        }

        PrintWriter writer = null;

        try {
            Template template = config.getTemplate(this.view);
            writer = this.response.getWriter();
            template.process(root, writer);
        } catch (Exception var7) {
            throw new RenderException(var7);
        } finally {
            if (writer != null) {
                writer.close();
            }

        }
````
看得有点绕，我们总结一下渲染的流程：
1.controller里面调用render方法  实际上调用的是IMainRenderFactory类的getRender，IMainRenderFactory类自带有三个实现，可以拓展。  获取一个Render类的实例。最后会调用这个实例的render方法（FreeMarkerRender） 通过response输出刘渲染页面

至此，整一个web请求的接受-处理-渲染的大体流程走完了。
