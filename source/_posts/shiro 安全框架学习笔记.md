---
title: vim 使用教程
date: 2019-04-11 12:14:04
tags: 测试
---

内容
Apache Shiro 是 Java 的一个安全框架。对比 Spring Security ，功能相对简单和小。是一个轻量级的安全框架，满足日常开发应用毫无问题。

shiro 非常容易开发出足够好的应用，主要在认证，授权，加密，会话管理，与web集成，缓存等方面有很好的实现。

主要功能模块
图

shiro 主要有四个主要功能：

- Authentication：身份认证/登录，验证用户是不是拥有相应的身份；

- Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

- Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

- Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；加密方式有多种，MD5，sha-256，base64。。。。

还有其他的扩展模块功能：

- Web Support：Web支持，可以非常容易的集成到Web环境；

- Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

- Concurrency：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

- Testing：提供测试支持；

- Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

- Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了

> 对 subject 进行认证和授权都需要调用 realm，所以realm 不仅仅相当于数据源，也包含了认证和授权的逻辑。shiro 不会去维护用户，维护权限，这些需要我们自己去设计和提供；然后通过相应的接口注入给shiro。

shiro 处理一个subject的流程：
![kkG674.png](https://s2.ax1x.com/2019/01/22/kkG674.png)

> 应用代码直接交互的对象是Subject

- Subject：主体，代表了当前用户，与当前应用交互的任何东西（包括用户，爬虫，机器人。。。）都是subject；所有subject都绑定到SecurityManager,与subject的所有交互都会委托给SecurityManager；可以把subject认为是一个门面；SecurityManager才是实际的执行者

- SecurityManager：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理这所有的subject，可以看出它是shiro的核心，负责这与后边的其他组件的监护。可以理解为一个前端控制器。

- realm：域，shiro 从 realm 获取安全数据（如用户，角色，权限），就是说 securityManager 要验证用户身份，那么它需要从 realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 realm 得到用户相应的角色、权限进行验证用户是否能进行操作；可以把 realm 看成 datasource ，即安全数据源。

简而言之：应用程序通过 subject 来认证和授权，然后 subject 将验证和授权委托给 securityManager；然后 securityManager 需要从 realm 里的合法用户和权限进行判断，所有 realm 需要注入给 securityManager。

## 拦截器机制
Shiro 使用了与 Servlet 一样的 Filter 接口进行扩展 
![kVcQhR.png](https://s2.ax1x.com/2019/01/23/kVcQhR.png)

1. NameableFilter: 给Filter起名，如果没有设置默认就是FileterName；之前组装拦截器链时会根据这个名字找到相应的拦截器实例。
2. OncePerRequestFilter：用于防止多次执行Filter的；一次请求只会走一次拦截器链；另外提供enabled属性，表示是否开启该拦截器实例。默认表示开启，不想让某拦截器工作，可以设置false
3. shirofilter:是整个shiro的入口点，用于拦截需要安全控制的请求进行处理。
4. adviceFilter：提供aop风格的支持，类似于springmvc中的intercept
 - preHandler：类似于aop中的前置增强；在拦截器链执行之前执行；如果返回true则继续拦截器链，否则中断后续的拦截器链；进行预处理（如基于表单的身份验证，授权）
 - postHandler：类似于aop中的后置增强；在拦截器链执行完成后执行，进行后处理
 - afterCompletion：类似于aop中的后置最终增强；即不管没有异常都会执行；可以进行清理资源（如基础subject 与线程的绑定之类）
5. PathMatchingFiler：提供了基于Ant风格的请求路径匹配功能及拦截器参数解析的功能
 - pathMatch:用于path于请求路径进行匹配的方法；如果匹配返回true
 - onPreHandle：在preHandle中，当pathsMatch匹配一个路径后，会调用onPreHandler方法并将路径绑定参数配置传给mappedValue；然后可以在这个方法中进行一些验证（如角色授权）验证失败则返回false中断流程，默认返回true；也就是说子类可以实现onPreHandle即可。如果没有path 与请求路径匹配，则默认为true
6. AccessControllerFilter：提供了访问控制的基础功能；比如是否允许访问，当访问拒绝时如何处理

 - isAccessAllowed：表示是否允许访问；mappedValue 就是[urls]配置中拦截器参数部分，如果允许访问返回 true，否则 false；

 - onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回 true 表示需要继续处理；如果返回 false 表示该拦截器实例已经处理了，将直接返回即可。

 - onPreHandle 会自动调用这两个方法决定是否继续处理：

## 拦截器链
shiro 对servlet 容器的 filterchain 进行了代理，即 shirofilter 在继续servlet容器的 filter 链的执行之前，通过 proxiedfilterchain 对 servlet 容器的filterchain 进行了代理；即先走 shiro 自己的 filter 体系，然后才会委托给servlet 容器的 filterchain 进行 servlet 容器级别的 filter 链执行。

而代理链是通过 filterchainresolver 根据配置文件中【urls】部分是否与请求的 url 模式（默认ant风格）
拦截器链和请求的 url 是否匹配来解析得到配置的拦截器链的；而 PathMatchingFilterChainResolver 内部通过 FilterChainManager 维护着拦截器链，比如 DefaultFilterChainManager 实现维护着 url 模式与拦截器链的关系。因此我们可以通过 FilterChainManager 进行动态动态增加 url 模式与拦截器链的关系。

> 说了这么多，估计大家也没耐心看下去，总结一下，就是 shiro 有自己的拦截器机制，通过代理filter链作用，是shiro的filter作用顺序在servlet之前
。

| 参数 | 含义 |
|:---:|:---|
| anon |例子/admins/**=anon 没有参数，表示可以匿名使用。
|authc | 例如/admins/user/**=authc表示需要认证(登录)才能使用，没有参数
| roles| 例子/admins/user/=roles[admin],参数可以写多个，多个时必须加上引号并且参数之间用逗号分割，当有多个参数时，例如admins/user/=roles[“admin,guest”]，每个参数通过才算通过，相当于hasAllRoles()方法。
| perms |例子/admins/user/=perms[user:add:*],参数可以写多个，多个时必须加上引号，并且参数之间用逗号分割， 例如/admins/user/=perms[“user:add:,user:modify:“]，当有多个参数时必须每个参数通过才通过， 想当于isPermitedAll()方法。
| rest |例子/admins/user/=rest[user],根据请求的方法，相当于/admins/user/=perms[user:method] , 其中method为post，get，delete等。
| port |例子/admins/user/**=port[8081],当请求的url的端口不是8081是跳转到schemal://serverName:8081?queryString, 其中schmal是协议http或https等，serverName是你访问的host,8081是url配置里port的端口，queryString是你访问的url里的？后面的参数。
| authcBasic |例如/admins/user/**=authcBasic没有参数表示httpBasic认证
| ssl |例子/admins/user/**=ssl没有参数，表示安全的url请求，协议为https
| user |例如/admins/user/**=user没有参数表示必须存在用户，当登入操作时不做检查