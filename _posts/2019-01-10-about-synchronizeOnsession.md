---
layout:     post
title:      关于 synchronizeOnSession
subtitle:   
author:     WeYunx
header-style: text
catalog: false
signature: true
tags:
    - Spring
---

最近在维护一个老项目，发现了一个问题。我们新增了一个耗时较久的复杂查询的功能，页面采用了ajax异步请求数据，但是请求未返回之前，点击页面其他功能都只能打开空白页，必须等待之前的数据返回后才能开始加载，整个过程是串行等待，调试过程中发现服务器仅分配了一个线程给该用户。故查看了一下原始代码，发现web.xml中配置了如下参数：

````xml
<init-param>
	<param-name>synchronizeOnSession</param-name>
    <param-value>true</param-value>
</init-param>
````

看了一下spring mvc 的说明文档，仅找到一处说明：

> Enforces the presence of a session. As a consequence, such an argument is never null. Note that session access is not thread-safe. Consider setting the RequestMappingHandlerAdapter instance’s synchronizeOnSession flag to true if multiple requests are allowed to concurrently access a session.

因为 session 是非线程安全的，如果需要保证用户能够在多次请求中正确的访问同一个 session ，就要将 ***synchronizeOnSession*** 设置为 ***TRUE*** 。

所以此处把synchronizeOnSession 改为 false 后，问题随之解决，调试中可以看到服务器为用户分配了多个线程。

同时也可以参考这个[例子](https://qiita.com/siumennel/items/f7d0973a0b6acbfd94b4)

##### 参考资料

- [docs.spring.io](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/web.html#mvc-caching-etag-lastmodified)

