
# CAS 4.1.10

* ### [自定义登录页](#customloginpage) 
* ### [自定义密码验证规则](#custompassword) 
* ### [AttributePrincipal获取额外信息](#moremessage) 
* ### [Server端添加无验证页面](#servernocheck)
* ### [Client端添加URL白名单](#clientnocheck) 
* ### [Ajax导致的跨域解决方案](#aboutajax) 


<hr>
<br>
<h3 id="customloginpage">自定义登录页</h2>
网上常见的方式就是直接修改CAS默认的登录页，位置在WEB-INF/view/jsp/default/ui/casLoginView.jsp<br>
需要注意的是&lt;form:form>&lt;/form:form>标签以及里面的内容需要保留，其它的可以根据情况修改页面样式，对账号密码框修改样式的话，外部样式表如果不起作用，可以直接在标签上使用cssStyle。
<br><br>
<h3 id="custompassword">自定义密码验证规则</h2>
Server端，创建自定义认证类，并继承AbstractUsernamePasswordAuthenticationHandler，重新它的方法，在方法中你可以从UsernamePasswordCredential获取前端传来的账号和密码，并且做你自己的验证规则，最后只需调用createHandlerResult()来返回一个HandlerResult即可。<br>
差点忘记，你还需要把这个类声明在WEB-INF/deployerConfigContext.xml里，<br>&lt;bean id="proxyAuthenticationHandler" class="your class path"/><br>
数据库的配置也是在这个xml里配置，你应该需要它。
<br><br>
<h3 id="moremessage">AttributePrincipal获取额外信息</h2>
Server端，创建一个类，继承自StubPersonAttributeDao，并重新他的方法，最后只需要return new AttributeNamedPersonImpl(username, attributes)即可。attributes为一个Map&lt;String, List<Object>>,至于你想要额外获取的value，你可以想办法set到这个map里面，然后在Client端，你可以使用request来获取它，就像这样:<br>
AttributePrincipal userPrincipal = (AttributePrincipal) request.getUserPrincipal();<br>String email = userPrincipal.getAttributes().get("email").toString();
<br><br>
<h3 id="servernocheck">Server端添加无验证页面</h2>
你可能想要把一些不需要登录的页面放在Server这里，不想被CAS拦截到，就像注册页。你需要做三件事<br>一、创建一个Controller(RegController)继承自AbstractController，重写它的方法并返回一个ModelAndView，这里假设mv.setViewName("reg");<br>二、打开WEB-INF/cas-servlet.xml，找到&lt;util:properties>&lt;/util:properties>，把&lt;prop key="/reg">regController&lt;/prop>添加进里面。<br>三、打开WEB-INF/web.xml，你会看到很多的&lt;servlet-mapping>，在最后加上<br>
&lt;servlet-mapping><br>
        &lt;servlet-name>cas&lt;/servlet-name><br>
        &lt;url-pattern>/reg&lt;/url-pattern><br>
    &lt;/servlet-mapping>
<br><br>
<h3 id="clientnocheck">Client端添加URL白名单</h2>
这里做法是在Client重写CAS的拦截器，它继承自AbstractCasFilter，在web.xml中默认配置的是拦截器是AuthenticationFilter，可以直接把它的代码拷出来，新建一个拦截器，在这份拦截器中做一些修改。
<br><br>
<h3 id="aboutajax">Ajax导致的跨域解决方案</h2>
在上面拦截器的代码里面，doFilter()方法最后,判断如果没有认证身份会以response.sendRedirect(urlToRedirectTo);重定向到Server，这时候，如果是Ajax发起了一个请求，被CAS拦截器拦截的无认证权限的话，再执行这些代码就会出现跨域问题。解决方案之一，在拦截器做重定向之前做一个判断是否为Ajax请求，如果是，转发到自身的一个Controller并且返回一个页面，这个页面只是为了跳转而存在，加载页面后使用js的window.location跳转。第二种解决办法是使用PrintWriter来推，具体代码自行百度。