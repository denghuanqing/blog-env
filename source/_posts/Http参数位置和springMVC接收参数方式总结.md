---
title: Http参数位置和springMVC接收参数方式总结
date: 2018-06-10 15:15:24
categories: spring
---
最近几个需求，用ajax请求后台数据的时候，contentType的含义总是模糊不清。今天系统的看下http请求参数携带方式已经springMVC如何来接受这些参数。
<!-- more -->
### HTTP的参数
##### URL里放参数
```
http://localhost:8080/http/demo1?param0=1
```
在URL里放参数最简单，就是问号+键值对.
##### Body里放参数
Body参数方式有几种不同的类型，即Content-type
Text	  text/plain
Form	application/x-www-form-urlencoded   （默认）
JSON	application/json
File	不确定
Multipart	multipart/form-data; boundary=X_PAW_BOUNDARY
- text/plain 文本传输，http的报文中是纯文本。
- application/x-www-form-urlencoded 
![image.png](https://upload-images.jianshu.io/upload_images/7027953-46ed5bcae63a461d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	Form表单形式传输，其本身采用Key-Value的方式传递参数.
- application/json
![image.png](https://upload-images.jianshu.io/upload_images/7027953-f88677862f27e33e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	他的HTTP的Body是一串符合JSON格式的字符串,可以用来传输Object
- File 
	这个也很少用，一般只在返回报文中出现，用于传输 单个文件 ，
- multipart/form-data; boundary=X_PAW_BOUNDARY
	Multipart的中文直译是 “多个部分” 就是指的不仅可以传输参数(Value)还可以传输文件(File)，但是参数和文件之间怎么区分呢？其中前者代表 多个部分/表单数据 而后者代表boundary(边界)，是一个字符串 “X_PAW_BOUNDARY“
	
##### 小结
通常情况下x-www-form-urlencoded是最常用的传参方法，如果想传递Object可以使用JSON，使用SpringMVC的话需要特殊配置，上传文件用的最多的就是Multipart，Java有专门的Jar包来处理文件上传


### SpringMVC接受参数的方式
##### spring接受处理参数主要分为4类
- 处理requet uri 部分（这里指uri template中variable，不含queryString部分）的注解：   @PathVariable;
- 处理request header部分的注解：   @RequestHeader, @CookieValue;
- 处理request body部分的注解：@RequestParam,  @RequestBody;
- 处理attribute类型是注解： @SessionAttributes, @ModelAttribute;

#####  @PathVariable
用来取出url path中的参数，使用于rest风格的接口。/post/{postId}  这个时候帖子ID就可以用这个注解来取出来

##### @RequestHeader、@CookieValue
取出http请求投 或者cookie中的参数（比如jsessionid），绑定到形参上。一般不太常用

##### @RequestParam, @RequestBody

###### @RequestParam
- 通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；
- 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；（不能获取 application/json 提交的json数据）
- 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定。若是某个参数不存在，一定要设置为false 否则会报错

###### @RequestBody
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；
它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

##### @SessionAttributes, @ModelAttribute

###### @SessionAttributes
注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。

###### @ModelAttribute
该注解有两个用法，一个是用于方法上，一个是用于参数上；
用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；(不太理解？？)
用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：
- @SessionAttributes 启用的attribute 对象上；
- @ModelAttribute 用于方法上时指定的model对象；
- 上述两种情况都没有时，new一个需要绑定的bean对象，然后*把request中按名称对应的方式把值绑定到bean中

##### 不指定注解怎么绑定呢？
通过分析AnnotationMethodHandlerAdapter和RequestMappingHandlerAdapter的源代码发现，方法的参数在不给定参数的情况下：
若要绑定的对象时简单类型：  调用@RequestParam来处理的。  
若要绑定的对象时复杂类型：  调用@ModelAttribute来处理的。
这里的简单类型指java的原始类型(boolean, int 等)、原始类型对象（Boolean, Int等）、String、Date等ConversionService里可以直接String转换成目标对象的类型；

##### 小结
一些很基础的东西，有时候没有常用就忘记了。有空的时候需要把springMVC解析参数的源码看一看，理解一下别人的设计思想，不能只停留在会用的基础上。



