---
title: 关于cors实现之简单请求与预检请求（发送请求头带令牌token）
date: 2017-6-29 21:23:14
categories: '基础知识'
tags:
    - cors
    - token
    - content-Type
    - 预检请求
---
写于：2017-6-29  
## 引子 ##
自从从JAVA伪全栈转前端以来，学习的路上就充满了荆棘（奇葩问题），而涉及前后端分离这个问题，对cors的应用不断增多，暴露出的问题也接踵而至。
这两天动手实践基于Token的WEB后台认证机制，看过诸多理论（[较好一篇][1]推荐），正所谓虑一千次，不如去做一次。 犹豫一万次，不如实践一次，所以就有了下文，关于token的生成，另外一篇文章会细讲，本篇主要讨论在发送ajax请求，头部带上自定义token验证验证，暴露出的跨域问题。
## 问题描述 ##
话不多说，先上代码：
```javascript
// 前端（ajax库：vue-resource）
userLogin:function(){
this.$http({
    method:'post',
    url:'http://localhost:8089/StockAnalyse/LoginServlet',
    params:{
        flag:'ajaxlogin',
        loginName:this.userInfo.id,loginPwd:this.userInfo.psd
        },
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    credientials:false,
    emulateJSON: true
}).then(function(response){
    sessionStorage.setItem('token',response.data);
    this.isActive =false;
    document.querySelector('#showInfo').classList.toggle('isLogin');
})
}
后端相关配置：
    response.setHeader('Access-Control-Allow-Origin', 'http://localhost'); //允许来之域名为http://localhost的请求
response.setHeader('Access-Control-Allow-Headers', 'Origin,No-Cache, X-Requested-With, If-Modified-Since, Pragma, Last-Modified, Cache-Control, Expires, Content-Type, X-E4M-With, userId, token');
response.setHeader('Access-Control-Allow-Methods', 'POST, GET, OPTIONS, DELETE'); //请求允许的方法
response.setHeader('Access-Control-Max-Age', '3600');	//身份认证(预检)后，xxS以内发送请求不在需要预检，既可以直接跳过预检，进行请求(前面只是照猫画虎，后面才理解)
```
关于上面一段代码，是我的用户首次登录认证，生成token令牌，保存在sessionStorage中，供后面调用；需要说明的是，前端服务器地址是：localhost:80,后端服务器地址：localhost:8089，所以前后端涉及到跨域，自己在后端做了相应的跨域设置：response.setHeader('Access-Control-Allow-Origin', 'http://localhost'); 所以登录认证,安全的实现了跨域信息认证，后端相应发送回来了相应的token信息。
但获取到token后，想在需要的时候，在请求的头部携带上这个令牌，来做相应的身份认证，所以自己在请求中做了这些改动（有标注），后端没改动，源码：

            checkIdentity:function(){
                let token =sessionStorage.getItem('token');
                this.$http({
                    method:'post',
                    url:'http://localhost:8089/StockAnalyse/LoginServlet',
                    params:{'flag':'checklogin','isLogin':true,'token':token},
                    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                    headers:{'token':token},        //header中携带令牌信息
                    credientials:false,
                    emulateJSON: true
                }).then(function(response){
                    console.log(response.data);
                })
            }
但实际上在devtools打印了如下错误信息：Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost' is therefore not allowed access.仔细想一想，好像，似乎这个问题遇到过，还提过问，确实提过，[链接在这里][2]。但这次的设置和上次一样，就在header里多加了一个自定义token，但却报了和上次没有设置headers: {'Content-Type': 'application/x-www-form-urlencoded'}一样的错误信息，于是，不知所措，算了，重头再来，好好百度，研究一下cors跨域。
## 理论学习 ##
运气不错，找到了[一篇好文][3]，文章讲的很细，也找到自己问题的所在：触发 CORS 预检请求。引用原文的话加以自己总结：跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request：似曾相识有没有？诶，对，上面那个错误信息中，就有一个这样陌生的词汇），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。所以跨域请求分两种：简单请求和预检请求。一次完整的请求不需要服务端预检，直接响应的，归为简单请求；而响应前需要预检的，称为预检请求，只有预检请求通过，才有接下来的简单请求。对于那些是简单请求，那些会触发预检请求，文章做了详细的总结，这里列出触发预检请求的条件（不知道脑子为啥会想到那些会触发BFC的条件），不要跑题，原文是这样总结的：

    当请求满足下述任一条件时，即应首先发送预检请求：
    使用了下面任一 HTTP 方法：
    PUT
    DELETE
    CONNECT
    OPTIONS
    TRACE
    PATCH
    人为设置了对 CORS 安全的首部字段集合之外的其他首部字段。该集合为：
    Accept
    Accept-Language
    Content-Language
    Content-Type (but note the additional requirements below)
    DPR
    Downlink
    Save-Data
    Viewport-Width
    Width
     Content-Type 的值不属于下列之一:
    application/x-www-form-urlencoded
    multipart/form-data
    text/plain
## 问题分析 ##
所以，再来看自己两次犯错（第一次是没有设置：headers: {'Content-Type': 'application/x-www-form-urlencoded'}, 第二次是设置自定义header，headers:{'token':token}。很巧，有没有，一次少，一次多，都点燃了导火索），其实都是触发了预检请求。对于第一次的错误，很好解决，增加headers: {'Content-Type': 'application/x-www-form-urlencoded'}，就解决了，[关于Conten-Type的几种取值，你需要知道的][4]。但对于第二个错误，好像没法向第一种那样，将预检请求转变为简单请求，所以，只有寻找方法怎么在后端实现相应的预检请求，来返回一个状态码2xx，告诉浏览器此次跨域请求可以继续。所以注意力转向后端。
关于JAVA实现预检请求，基本都是采用过滤器，不要问我为什么不是监听器或者拦截器（我就是个伪全栈，就不要相互为难了，自己百度之），自定义（copy）了一个filter,并在web.xml中进行了设置。源码：

    Filter接口实现部分：
    package stock.model;
    import java.io.IOException;
    import javax.servlet.Filter;
    import javax.servlet.FilterChain;
    import javax.servlet.FilterConfig;
    import javax.servlet.ServletException;
    import javax.servlet.ServletRequest;
    import javax.servlet.ServletResponse;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import org.apache.commons.httpclient.HttpStatus;   //这里需要添加commons-httpclient-3.1.jar
    public class CorsFilter implements Filter { 	//filter 接口的自定义实现
    	public void init(FilterConfig filterConfig) throws ServletException {
    	}
    	@Override
    	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    		HttpServletResponse response = (HttpServletResponse) servletResponse;
    		HttpServletRequest request = (HttpServletRequest) servletRequest;
    		response.setHeader('Access-Control-Allow-Origin', '*');
    		response.setHeader('Access-Control-Allow-Methods', 'POST, GET, DELETE, OPTIONS, DELETE');
    		response.setHeader('Access-Control-Allow-Headers', 'Content-Type, x-requested-with, Token');
    		String token = request.getHeader('token');
    		System.out.println('filter origin:' token);//通过打印，可以看到一次非简单请求，会被过滤两次，即请求两次，第一次请求确认是否符合跨域要求（预检），这一次是不带headers的自定义信息，第二次请求会携带自定义信息。
    		if ('OPTIONS'.equals(request.getMethod())){//这里通过判断请求的方法，判断此次是否是预检请求，如果是，立即返回一个204状态吗，标示，允许跨域；预检后，正式请求，这个方法参数就是我们设置的post了
    		  response.setStatus(HttpStatus.SC_NO_CONTENT); //HttpStatus.SC_NO_CONTENT = 204
    		}
    		filterChain.doFilter(servletRequest, servletResponse);
    	}
    	@Override
    	public void destroy() {
    	}
    }
    web.xml配置部分
    <filter>
    <filter-name>cors</filter-name>
    <filter-class>stock.model.CorsFilter</filter-class>
    </filter>
    <filter-mapping>
    <filter-name>cors</filter-name>
    <url-pattern>/*</url-pattern>
    </filter-mapping>
## 结论 ##
当在后端实现添加上面的源码后，皆大欢喜，问题得以解决，补上失败和成功,自己截下的两张请求响应图。![图片描述][5]
仔细看请求响应失败发起响应那张图，在General的数据集中，可以看到方法是options，而非代码指定的post请求，所以这是一次浏览器发出的一次预检请求，让服务器确认此IP是否有访问的权限，如果有，服务器需要返回一个2xx的状态码给浏览器。紧接着再发起一次简单请求。如下面在devtools中的截取图片（为了对比清除，我把两次分别截取，做了拼接，因为不会做动态图）。可以看到同一个post请求，实际上产生了两次网络连接。![图片描述][6]

但关于cors,要去探索的，还有很多很多，所以遵循革命语录：实践（有时也可以是时间）是检验真理的唯一标准，是没有错的。后续有新的收获，再补充
## 知识补充 ##
Cors定义：跨来源资源共享（CORS）是一份浏览器技术的规范，提供了 Web 服务从不同网域传来沙盒脚本的方法，以避开浏览器的同源策略，是 JSONP 模式的现代版。与 JSONP 不同，CORS 除了 GET 要求方法以外也支持其他的 HTTP 要求。用 CORS 可以让网页设计师用一般的 XMLHttpRequest，这种方式的错误处理比JSONP要来的好，JSONP对于 RESTful 的 API 来说，发送 POST/PUT/DELET 请求将成为问题，不利于接口的统一。但另一方面，JSONP 可以在不支持 CORS 的老旧浏览器上运作。不过现代的浏览器（IE10以上）基本都支持 CORS。


  [1]: http://www.cnblogs.com/xiekeli/p/5607107.html
  [2]: https://segmentfault.com/q/1010000009255088
  [3]: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
  [4]: http://zccst.iteye.com/blog/2180127
  [5]: https://sfault-image.b0.upaiyun.com/256/380/2563800622-5953c240d7699_articlex
  [6]: https://sfault-image.b0.upaiyun.com/114/216/114216967-595610350e335_articlex