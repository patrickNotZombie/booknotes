---
layout:     post
title:      "ZUUL组件改造"
subtitle:   "修改转发请求参数"
date:       2019-01-01 16：19：00
author:     "Patrick"
header-img: "img/spring-logo.jpg"
catalog:    true
tags:
    - Java
    - SpringBoot
    - Zuul
---

>增加请求参数

分为请求路径/zuul和不带/zuul两种情况，也就是DispatcherServlet和ZuulServlet处理的所有请求。网上可以查阅的例子都是普通的由DispatcherServlet处理请求的情况，不适用于上传文件等请求路径包含/zuul,直接交由ZuulServlet处理的请求，故写下此篇文档，以供后来者参考。

zuul作为spring cloud中的网关组件，是前端请求与众多微服务进行联系的门户，相当于传统服务器中的nginx(或者apache)，可以对所有的请求进行鉴权，过滤和转发，这些功能在其他的博客中都有详细的介绍。原有单体应用是通过session来判断是否是同一用户，新的微服务是通过OAuth2.0的jwt进行判断，每次发送的请求头中都包含Authorization信息，则可以有效通过验证，调用相应的接口。

详细的zuul的使用和源码分析可以参考以下博客或者文档
- [zuul学习四：zuul 过滤器详解](https://www.jianshu.com/p/ff863d532767)
- [springcloud zuul 修改转发传递的参数](https://blog.csdn.net/u012930316/article/details/80975563)
- [深入理解Zuul之源码解析](https://blog.csdn.net/forezp/article/details/76211680)

现在我要做的是将之前的老系统与新的微服务架构的系统进行融合，新搭建的为服务系统采用OAuth+JWT进行认证，所以新的请求无法通过seesion保存当前操作者的信息，需要在请求参数中添加额外的用户相关参数，例如currentUserId等，为了保证后台系统的升级对前端无感，无需客户端对url进行改造，所以在zuul进行参数添加。

Zuul默认注入的过滤器，它们的执行顺序在FilterConstants类，我们可以先定位在这个类，然后再看这个类的过滤器的执行顺序以及相关的注释，可以很轻松定位到相关的过滤器，也可以直接打开spring-cloud-netflix-core.jar的 zuul.filters包，可以看到一些列的filter，现在我以表格的形式，列出默认注入的filter.

|过滤器	|order	|描述	|类型|
| :-----: | :-----: |:-----: |:-----: |
|ServletDetectionFilter	|-3	|检测请求是用 DispatcherServlet还是 ZuulServlet|	pre|
|Servlet30WrapperFilter	|-2	|在Servlet 3.0 下，包装 requests|	pre|
|FormBodyWrapperFilter	|-1	|解析表单数据	|pre|
|SendErrorFilter|	0	|如果中途出现错误	|error|
|DebugFilter|	1|	设置请求过程是否开启debug	|pre|
|PreDecorationFilter|	5	|根据uri决定调用哪一个route过滤器|	pre|
|RibbonRoutingFilter|	10	|如果写配置的时候用ServiceId则用这个route过滤器，该过滤器可以用Ribbon 做负载均衡，用hystrix做熔断|	route|
|SimpleHostRoutingFilter|	100|	如果写配置的时候用url则用这个route过滤	|route|
|SendForwardFilter	|500|	用RequestDispatcher请求转发	|route|
|SendResponseFilter	|1000	|用RequestDispatcher请求转发	|post|

过滤器的order值越小，就越先执行，并且在执行过滤器的过程中，它们共享了一个RequestContext对象，该对象的生命周期贯穿于请求，可以看出优先执行了pre类型的过滤器，并将执行后的结果放在RequestContext中，供后续的filter使用，比如在执行PreDecorationFilter的时候，决定使用哪一个route，它的结果的是放在RequestContext对象中，后续会执行所有的route的过滤器，如果不满足条件就不执行该过滤器的run方法。最终达到了就执行一个route过滤器的run()方法。而error类型的过滤器，是在程序发生异常的时候执行的。post类型的过滤，在默认的情况下，只注入了SendResponseFilter，该类型的过滤器是将最终的请求结果以流的形式输出给客户单。


通过前面的文档介绍，我们知道zuul的处理请求分为两种不同的情况，```DispatcherServlet```和```ZuulServlet```,
使用```ZuulController```(```ServletWrappingController```的子类)封装```ZuulServlet```实例, 处理从```DispatcherServlet```进来的请求；```ZuulHandlerMapping```负责注册handler mapping, 将Route的fullPath的请求交由ZuulController处理；使用```ServletRegistrationBean```注册```ZuulServlet```, 默认使用/zuul作为urlMapping. 所有来自以/zuul开头的path的请求都会直接进入```ZuulServlet```, 不会进入```DispatcherServlet```.

下面上代码,网上的普通适用于进入DispatcherServlet请求添加请求参数的方法：

	public static void  setReqParams()  {
    	RequestContext ctx = RequestContext.getCurrentContext();
    	HttpServletRequest request = ctx.getRequest();
    	// 一定要get一下,下面这行代码才能取到值
    	request.getParameterMap();
    	Map<String, List<String>> requestQueryParams = ctx.getRequestQueryParams();
    	
    	if (requestQueryParams==null) {
    		requestQueryParams=new HashMap<>();
    	}
    	
    	//将要新增的参数添加进去,被调用的微服务可以直接 去取,就想普通的一样,框架会直接注入进去
    	ArrayList<String> arrayList = new ArrayList<>();
    	arrayList.add("1");
    	requestQueryParams.put("test", arrayList);
    	
    	ctx.setRequestQueryParams(requestQueryParams);
	}

**注释**：简而言之，就是取到上下文中的参数Map，然后添加自己想要添加的参数进去，但是以上获取参数的方法无法获取由ZuulServlet处理的请求中的参数，本人的做法如下：


    public Object run() {
        //获取request
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        if (RequestUtils.isDispatcherServletRequest()) {
            System.out.println("isDispatcherServletRequest:" + "YES");
        } else if(RequestUtils.isZuulServletRequest()){
            System.out.println("isZuulServletRequest:" + "YES");

        }

        log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

        String access_token = request.getHeader("Authorization");
        if (access_token != null) {
           //其他具体业务代码

            //查看请求参数
            System.out.println("URL:" + request.getRequestURL().toString());
			//获取ZuulServlet处理的请求中的参数
            Map<String, String[]> map = request.getParameterMap();

			//获取DispatcherServlet处理的请求中的参数
            Map<String, List<String>> requestQueryParams = ctx.getRequestQueryParams();
            if (requestQueryParams==null) {
                requestQueryParams=new HashMap<>();
                if (map != null && !map.isEmpty()&&RequestUtils.isZuulServletRequest()) {
                    for(Map.Entry<String , String[]> entry : map.entrySet()){
                        List<String> arrayList_tmp = Arrays.asList(entry.getValue());

                        System.out.print(entry.getKey()+":");//entry.getKey() 参数名;
                        //entry.getValue();参数值，类型为数组
                        System.out.println(StringUtils.join((Object[]) entry.getValue(), ","));
                        requestQueryParams.put(entry.getKey(), arrayList_tmp);
                    }
                }
            } 

            //将要新增的参数添加进去,被调用的微服务可以直接 去取,就想普通的一样,框架会直接注入进去
            ArrayList<String> arrayList = new ArrayList<>();
            ArrayList<String> arrayList2 = new ArrayList<>();
            arrayList.add(var1+"");
            arrayList2.add(var2+"");
            requestQueryParams.put("var1", arrayList);
            requestQueryParams.put("var2", arrayList2);

            ctx.setRequestQueryParams(requestQueryParams);
        } else {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            return null;
        }

        return null;
    }

主要的不同就是注意通过RequestUtils.isZuulServletRequest()判断是由啥Servlet处理的请求，再对请求参数进行处理，否则会导致有的参数重复或者需要加的参数没加进去。