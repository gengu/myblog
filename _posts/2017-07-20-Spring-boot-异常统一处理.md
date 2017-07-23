---
layout: post
title: Spring-boot-统一异常统一处理实践
description: "统一异常处理实践"
modified: 2017-07-20
tags: [spring-boot系列 , exception]
---

## 原则
在我们使用Spring-boot REST API开的过程中，经常会遇到形形色色的异常，受检异常||非受检异常，如何选择异常处理的逻辑，常常会让我们如履薄冰，稍有不慎，就会给系统带来线上故障。

处理异常也让我很头疼，各种错误出现以后不知道该如何处理，系统做的多了，才领悟了一个原则：

> 假如你知道如何处理当前的异常，你就处理；假如你不知道如何处理当前的异常，那就往上抛。

## 问题
我们都知道，异常分两种，受检异常与非受检异常  
常见的例如NPE就是非受检异常，也就是在代码编写阶段不用显式处理的   
另外一种，例如FileNotFoundException是受检异常，也就是在程序编译前需要显式处理   
这里的处理又有两种，一种是try catch住，另外一种在方法定义时抛出  
trycatch是我们使用最多的方式，但是也有个很不好解决的问题，就是大部分情况下，其实我们是不知道该如何处理当前异常的，有些异常在调试开发阶段根本遇不到，更谈不上处理  

基于抛异常原则，如果异常一直得不到处理，对于容器（例如tomcat）来说，将会销毁当前线程，并向Response返回错误堆栈信息，这种处理逻辑对客户端来说特别不友好，另外由于使用了Mybatis等技术，异常堆栈信息中可能会存在相当多的敏感信息，例如sql模板、各种jar包的版本信息，假如你刚好使用了安全漏洞的包，黑客就能攻击你的系统了。

> 所以从整个系统框架层面来考虑一种统一的、安全的异常处理机制是非常有必要的。

有几点需要考虑
1、拦截异常堆栈信息的透出
2、保留现场，即你需要在错误出现的时候把异常堆栈信息
3、不能改变系统原有的逻辑，例如Spring的回滚机制是利用异常来判断的，这个也是需要考虑的

## 异常处理的方案

### 常规的异常处理
Spring提供了AOP，借用AOP，其实我们能够很优雅的处理各类异常
{% highlight java %}  
{% raw %}
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Aspect
@Component
public class ExceptionHandler {

	@Autowired
	UserDOMapper userDOMapper ;

	private Logger logger = LogManager.getLogger(POExceptionHandler.class.getName()) ;

	/**
	 * 用来存储当前登录用户
  */
	public static ThreadLocal<UserDO> appThreadLocal = new ThreadLocal<>() ;

	/**
	 * 定义切面
	*/
	@Pointcut("execution(* com.xxx.controller.*.* (..))")
	public void web(){}

	/**
	 * 向httpRequest里面塞一个rid
	 * 向httpResponse里面塞一个rid
	 * 然后做一个异常的处理
	 * @param joinPoint
	 * @throws Throwable
	 */
	@Around("web()")
	public Object doAround(ProceedingJoinPoint joinPoint) {
		try {
			ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
			String app = attributes.getRequest().getHeader("app") ;
			if (Util.isNull(app)) {
				return ResultBOBean.ofError("app为空");
			}
			UserDO userDO = userDOMapper.selectByName(app ) ;
			if (userDO == null) {
				return ResultBOBean.ofError("用户不存在或已经过期");
			}
			if(null != attributes) {
				String rid = Util.getUuid();
				HttpServletRequest request = attributes.getRequest();
				HttpServletResponse response = attributes.getResponse() ;
				response.setHeader("rid", rid) ;
				request.setAttribute("rid",rid);
				appThreadLocal.set(userDO);
			}
			return joinPoint.proceed(joinPoint.getArgs()) ;
		} catch (Throwable throwable) {
			logger.error("POExceptionHandler : " , throwable);
			if (throwable instanceof POException) {
				return ResultBOBean.ofError(throwable.getMessage());
			} else if (throwable instanceof IllegalArgumentException) {
				return ResultBOBean.ofError(throwable.getMessage());
			} else if (throwable instanceof NullPointerException) {
				return ResultBOBean.ofError(throwable.getMessage());
			} else if (throwable instanceof BadSqlGrammarException) {
				return ResultBOBean.ofError("内部错误！请联系管理员！");
			} else {
				String errorMsg = throwable.toString() == null ? throwable.getMessage() : throwable.toString() ;
				return ResultBOBean.ofError(errorMsg == null || errorMsg.equals("") ? "未知错误" : throwable.toString());
			}
		}
	}
}
{% endraw %}   
{% endhighlight %}

在统一处理异常的时候，我们还可以加入特别多的元素，这就像是一个插件一样可以做很多有意义的事情，就像代码中一样，我们可以为每种异常定义一种错误处理方式    
在某些情况下，如果我们必须得处理trycatch（例如Lambda表达式的处理中），也可以将非受检异常转化为受检异常然后抛出   
在一般的代码开发中，我们有许多事情都可以变得简单，例如参数校验
{% highlight java %}  
{% raw %}
@RequestMapping("/listHeatTrendDay")
public ResultBOBean<List<BO>> listHeatTrendDay(@RequestParam("start") String start, @RequestParam("end") String end) {
    ...xxx...
    Preconditions.checkArgument(DateUtil.checkDay(start), "参数start非法");
    Preconditions.checkArgument(DateUtil.checkDay(end), "参数end非法");
    ...xxx...
}
{% endraw %}   
{% endhighlight %}
以上代码中，每个参数的校验只需要一行代码，Preconditions是Guava的一个类，它的原理就是遇到错误抛出一个异常，然后POExceptionHandler会自动来处理这种异常。

## 总结
通过这种方式，可以大大减少系统维护的成本，并且整个实现方式非常优雅。







