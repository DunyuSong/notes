# SpringMVC 拦截器
## 执行时机
- 客户端发送请求-->过滤器-->达到 DispatcherServlet-->执行拦截器
- 过滤器是在客户端发送请求达到Servlet之前执行的，而拦截器是属于Servlet中一部分。拦截器执行时在 DispatcherServlet 里面执行的
### 拦截器实现
- 实现 HandlerInterceptor 接口 或者可以继承 HandlerInterceptorAdapter 适配器类。
1. 编写自定义拦截器MyInterceptor.java
```java
public class MyFirstInterceptor implements HandlerInterceptor {

    // 请求处理方法之前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println(this.getClass().getSimpleName() + " preHandle method invoked!!!");
        return true;
    }

    // 请求处理方法之后，视图处理之前执行
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println(this.getClass().getSimpleName() + " postHandle method invoked!!!");
    }

    // 视图处理之后执行。转发/重定向之后
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println(this.getClass().getSimpleName() + " afterCompletion method invoked!!!");

    }
}

```
2. 编写配置文件 springDispatcherServlet-servlet.xml
```xml
    <!-- 配置拦截器。 多个拦截器按照配置顺序执行，但是拦截器方法执行顺序是按照迭代的顺序来执行的。preHandle()是按照从小到大，而postHandle()和afterCompletion()是按照从大到小-->
    <mvc:interceptors>
        <!-- 拦截所有请求 -->
        <bean class="com.github.miller.shan.interceptor.MyFirstInterceptor"/>
        <bean class="com.github.miller.shan.interceptor.MySecondInterceptor"/>

        <!-- 指定拦截请求 -->
        <!--        <mvc:interceptor>-->
        <!--            <mvc:mapping path="/test/testJson"/>    &lt;!&ndash; 指定拦截请求 &ndash;&gt;-->
        <!--            &lt;!&ndash;            <mvc:exclude-mapping path="/test/testUpload"/>      &lt;!&ndash; 指定不拦截请求&ndash;&gt;&ndash;&gt;-->
        <!--            <bean class="com.github.miller.shan.interceptor.MyFirstInterceptor"/>-->
        <!--        </mvc:interceptor>-->
    </mvc:interceptors>
```
### 分析拦截器的执行顺序
- 通过Debug查看DispacherServlet.java代码的1033行可以看得出来。
- preHandle()   方法是在请求处理方法之前执行。
- postHandle()  方法是在请求处理方法之后，视图处理之前执行
- afterCompletion   方法是在视图处理之后执行。转发/重定向之后
```java

        // 执行preHandle
		if (!mappedHandler.applyPreHandle(processedRequest, response)) {
			return;
		}

		// 请求处理方法
		mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

		if (asyncManager.isConcurrentHandlingStarted()) {
			return;
		}

		applyDefaultViewName(processedRequest, mv);
		
		// 执行postHandle
		mappedHandler.applyPostHandle(processedRequest, response, mv);
	}
	catch (Exception ex) {
		dispatchException = ex;
	}
	catch (Throwable err) {
		// As of 4.3, we're processing Errors thrown from handler methods as well,
		// making them available for @ExceptionHandler methods and other scenarios.
		dispatchException = new NestedServletException("Handler dispatch failed", err);
	}
	// 视图处理，处理结果
	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
catch (Exception ex) {

	triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
}
    // 视图处理方法里面执行
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
		    // render方法里面处理视图、模型数据，整合输出数据，将请求进行转发/重定向操作
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		if (mappedHandler != null) {
		    // 执行afterCompletion
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}
```
## 多个拦截器执行顺序
1. 多个拦截器执行顺序是按照配置文件 springDispatcherServlet-servlet.xml 编写顺序来执行的。
2. 多个拦截器里面的方法执行顺序是按照迭代类执行的，其中preHandle()是按照从小到大，而postHandle()和afterCompletion()是按照从大到小。
3. 执行多个拦截器输出结果如下
```java
MyFirstInterceptor preHandle method invoked!!!
MySecondInterceptor preHandle method invoked!!!
MySecondInterceptor postHandle method invoked!!!
MyFirstInterceptor postHandle method invoked!!!
MySecondInterceptor afterCompletion method invoked!!!
MyFirstInterceptor afterCompletion method invoked!!!
```
4. 查看源码"HandlerExecutionChain.java"。还是在DispatcherServlet.java中查看1033行中的applyPreHandle方法，if (!mappedHandler.applyPreHandle(processedRequest, response)) {
```java
    // PreHandle 迭代执行
	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
	    // 获取当前所有拦截器
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
		// 然后从小到大开始执行，所以执行顺序 MyFirstInterceptor.preHandle --> MySecondInterceptor preHandle
			for (int i = 0; i < interceptors.length; i++) {
				HandlerInterceptor interceptor = interceptors[i];
				if (!interceptor.preHandle(request, response, this.handler)) {
					triggerAfterCompletion(request, response, null);
					return false;
				}
				// interceptorIndex 默认值是-1，每次执行后将i的值赋值给i
				this.interceptorIndex = i;
			}
		}
		return true;
	}
	// PostHandle 迭代执行
	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
			throws Exception {
        // 获取当前所有拦截器
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
		// 然后从大到小开始执行，所以执行顺序 MySecondInterceptor postHandle --> MyFirstInterceptor postHandle 
			for (int i = interceptors.length - 1; i >= 0; i--) {
				HandlerInterceptor interceptor = interceptors[i];
				interceptor.postHandle(request, response, this.handler, mv);
			}
		}
	}
	// AfterCompletion 迭代执行
	void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
		throws Exception {
        // 获取当前所有拦截器
    	HandlerInterceptor[] interceptors = getInterceptors();
    	if (!ObjectUtils.isEmpty(interceptors)) {
    	    // 然后从大到小开始执行，所以执行顺序 MySecondInterceptor afterCompletion --> MyFirstInterceptor afterCompletion 
    		for (int i = this.interceptorIndex; i >= 0; i--) {
    			HandlerInterceptor interceptor = interceptors[i];
    			try {
    				interceptor.afterCompletion(request, response, this.handler, ex);
    			}
    			catch (Throwable ex2) {
    				logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
    			}
    		}
    	}
    }
```
