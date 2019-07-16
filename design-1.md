> 总论

JSMod2(JSM)是一款基于sMod2的java插件开发框架,JSM作为服务的形式与sMod2对接,sMod2的链接JSM的客户端是ProxyHandler，在JSM-PH-SMOD2组成的跨java-c#语言开发框架建立起来，同时，内部实行了挂接注册机制来定位api对象.
> JSMod2协议
JSM底层采用了JSMod2协议与ProxyHandler进行对接
(参见[Protocol]( [https://github.com/jsmod2-java-c/Jsmod2_protocol](https://github.com/jsmod2-java-c/Jsmod2_protocol) )，实质上是JSON数据串,最终会被实例化为一个具体的数据对象,关于JSM协议,参见GitHub.
> JSMod2的工作原理图sh
> 内部最主要的特征是挂接注册机制
