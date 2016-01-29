---
layout: post
title: RESTful Spring Security for SPA? No really
description: "This is a notes for ReactJs"
modified: 2016-01-29
tags: [spring security, SPA, Ajax, RESTful]
---

### Background
现在的Team在维护一个用户数据监测系统，采用的是比较老的 JSP + Spring Security + Spring MVC 这套框架。以前虽然有过一些 Spring 的基本了解，但是对 Spring MVC 和 Spring Security 根本没有系统的看过，正好凑这个机会看了一下，顺便跟据项目需求完成几个 task，总结了一些经验，赶紧写下来，怕以后忘掉。
因为也没有完全系统的都去看一遍，是一边猜一边试的结果，就按照我的需求来一一列出来吧。

<!--more-->

### Implementation

将 Spring Security 的验证结果用像RESTful的方式返回，而不是让 Spring Security 自己 Redirect to 其他的 page
Spring Security 的默认行为是用户可以指定相应的 url 让 Spring Security 去实现自动跳转，但是如果我想自己写一个 login page，我希望通过 Ajax call 去调用 Spring Security 的 authentication，但是我不想让 Spring Security 帮我去跳转，我只想知道 Spring Security authentication 的结果是什么，换句话说就是让 Spring Security 将验证的结果返回给我，而不是返回给我一个 forward 的 web page。例如：Spring Security 默认的验证 url 为 `j_spring_security_check`，那么我可以通过下面的代码实现 Spring Security authentication:

{% highlight javascript %}

$.ajax({
    url: domain + "j_spring_security_check", // http://10.172.16.3:8080/j_spring_security_check
    data: {
        j_username: username,
        j_password: password
    },
    type: "POST",
    crossDomain: true,
    error: function(req, status) {
        if (req.status === 401) {
            console.log("验证未通过，或许是用户名或密码错误！");
        }
    },
    success: function(data, status, req) {
        // Pay attension on this function, 理论上来讲，如果验证正确，Spring Security 应该帮我们自动跳转到其他页面
        // 但是，如果你发现没有跳转，这是因为 Ajax 请求的原因，所以我们要在下面通过前端实现跳转，
        // 如果是普通请求，就会跳转成功，
        // 当然，如果你没有这个需求，在 server 端就不要做类似： response.sendRedirect("success") 这样的事情
        if (req.status === 200) {
            window.location.href = "http://domain:port/success";
        }
    }
});

{% endhighlight %}
  以上就是前端所有要做的事情，剩下的事情都交给后端处理吧

  后端要做的事情，首先就是配置，如果你使用的是 Spring 4.0+，也可以写成 java 代码的方式，而不用非要写在 XML 配置文件中，下面就是配置代码：

{% highlight xml %}
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns="http://www.springframework.org/schema/security" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.2.xsd">

    <http use-expressions="true">
        <intercept-url pattern="/login" access="permitAll" />
        <intercept-url pattern="/other/**" access="hasRole('admin')" />

        <form-login login-page="/login"
            authentication-success-handler-ref="authSuccessHandler"
            authentication-failure-handler-ref="authFailureHandler" />
    </http>
</beans:beans>
{% endhighlight %}
这是 Spring Security 的xml配置，可以看到 `authSuccessHandler` `authFailureHandler`这两个 bean，下面是对应的这两个类的代码

{% highlight java %}
@Service
public class AuthFailureHandler extends SimpleUrlAuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException exception) throws IOException, ServletException {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        PrintWriter writer = response.getWriter();
        writer.write(exception.getMessage());
        writer.flush();
    }
}
{% endhighlight %}

{% highlight java %}
@Component
public class AuthSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {

        // Add your user info handler
<!--         UserDetails userDetails = UserDetails) authentication.getPrincipal();
        User user = userDetails.getUser();
        userDetails.setUser(user); -->
        // user these code to return status code: 200
<!--         response.setStatus(HttpServletResponse.SC_OK);
        PrintWriter writer = response.getWriter();
        mapper.writeValue(writer, user);
        writer.flush();
 -->
        // use these code to direct to corresponding page according to device
        String url = "";
        if (isMobile(req.getHeader("User-Agent"))) {
            url = "mobile/view";
        } else {
            url = "pc/view";
        }
        res.sendRedirect(url);
    }
}
{% endhighlight %}

以上就是我为了从 Spring Security 得到 RESTful 结果所做的主要的改动，其中还是有一部分是关于
AuthenticationEntryPoint，但是我发现我们这边不用加这个也是可以 work 的，我就没有加这部分，但是这部分在我这 work 或许正好跟我的需求吻合，具体原因将在下面阐述。要看全部的实现，请参考下面这两个链接得到更多相关的资料.

- [Secure REST services using Spring Security](https://crazygui.wordpress.com/2014/08/29/secure-rest-services-using-spring-security/)
- [Spring Security for a REST API](http://www.baeldung.com/2011/10/31/securing-a-restful-web-service-with-spring-security-3-1-part-3/)


### Thinking in Design
ok，以上就是代码部分，现在聊聊设计部分。
一开始，我以为要将 Spring Security authentication 包装成一个 RESTful API，前端直接通过 Ajax Call 就ok了，这个想法现在想起来真是 Too Young Too Simple, Sometimes Naive. 身为一个前端娃儿，我也是够封闭的。那问题是什么呢，如果 Spring Security 仅仅是一个 API，那用户登陆后再发请求就都是合法的了，谁去验证？用户登录过期时间谁来维护，难道前端每次发请求都把用户名/密码带上？前端如果存储用户名/密码？带着这些问题，我终于明白了 Spring Security 配置中 `<intercept-url pattern="/other/**" access="hasRole('admin')" />` 的意思。这段 code 的意思就是凡是符合这个规则的请求，都必须有 `admin` 权限才可以有正确的返回结果，而谁去验证这个请求呢，当然是 Spring Security 啦，也就是说 Spring Security 就是一个门神，什么请求到 server 端都得它的验证才能通过，终于对 Spring Security 有了基本的概念了。

再来说说上面提到的 AuthenticationEntryPoint，我为什么说我们当前的工程部需要这部分的改动呢，是因为我们当前的改动并没有把 Spring Security 改成是一个完完全全的 RESTful API，如果用户发了 request，但是并没有登陆过，我们就通过 Spring Security 默认的规则，让页面定位到登陆页面。不过 AuthenticationEntryPoint 是不是完成我说的这个功能，这仅仅是我的猜想，有待验证。所以从上面我给出的代码可以看出，我们当前的项目实际上不是一个 SPA，而是一个 Login 页面 + SPA, 为什么要这么设计，首先，登陆界面负责验证，通过后会跳转到相应的页面，从上面代码看到，我们项目要跟据当前的 device 定位到不同的 page，为什么要这么做，是因为以前的东西现在还不能移掉，新的 SPA 是为移动端开发的，开发完成才会替换掉 pc 端的 jsp，就为实现这个东西，我才研究的 Spring Security。同时也是因为不想改变当前 request url 规则，前端人应该知道，不管是 Angular.js 还是 React.js，它们 route 的实现都是基于 window.location.hash，而 hash 的值都是在 `#/`，而我们当前的 url 完全不包含 `#/`,所以才没办法将 mobile 端整个合并在一起，不过貌似可以把网站的 root 直接设为 `#`，只是看到有人说，未曾试过。

再说新的发现
参考链接 [Spring Security and Angular JS](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page) 这篇文章讲了一下 SPA 如何跟 Spring Security 集成到一起，只看了个开头，注意到下面的code

{% highlight java %}
if (csrf != null) {
    Cookie cookie = WebUtils.getCookie(request, "XSRF-TOKEN");
    String token = csrf.getToken();
    if (cookie==null || token!=null && !token.equals(cookie.getValue())) {
        cookie = new Cookie("XSRF-TOKEN", token);
        cookie.setPath("/");
        response.addCookie(cookie);
    }
}
{% endhighlight %}

文章中还提到每个验证过的 request header 都带了 cookie，不清楚原理是什么，这篇文章还得多读几遍，等 get the key point 以后再来写。
