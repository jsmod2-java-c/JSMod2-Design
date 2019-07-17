# JSMOD2 架构论 

Jsmod2的整个架构极其简单，该文章为了方便其他开发者贡献代码和维护项目

> 总论

JSMod2(JSM)是一款基于sMod2的java插件开发框架,JSM作为服务的形式与sMod2对接,sMod2的链接JSM的客户端是ProxyHandler，在JSM-PH-SMOD2组成的跨java-c#语言开发框架建立起来，同时，内部实行了挂接注册机制来定位api对象.
> JSMod2协议
JSM底层采用了JSMod2协议与ProxyHandler进行对接

参见[Protocol](https://github.com/jsmod2-java-c/Jsmod2_protocol)

，实质上是JSON数据串,最终会被实例化为一个具体的数据对象,关于JSM协议,参见GitHub.
> JSMod2的工作原理图
内部最主要的特征是挂接注册机制

![main](main.png)
##### 图中少了个部分，ApiIdMapping的对象来源是当发生事件时，会将事件对象和其中的其他对象加入到这个表里(同时随机生成id，然后一起把id发出去)
##### 然后下一次发出去的对象重新发个修改的请求，也会顺带着之前发给它的ID，proxyHandler那边就通过这个ID找到那个对象，然后进行设置，这也是对于sendObject里的IdMapper作用的解释

如果可以通过springboot

说明是有效的http协议，会将jsmod2协议串传输到协议翻译器

协议翻译器负责协议的解码，并获取id号和json对象串，分发到每个PacketManager中

协议翻译器首先将byte翻译为base64，再解码为注册的jsmod2协议串，然后拆分为id，json，附加请求

Packet协议管理器负责处理Event和Command请求

注册机注册了包id和事件id等
注册机相当于一个字典，记录一些定义信息

PacketManager从Register通过id拉取event类型对象
将json转化为event。
如果不是类型对象，会转化为CommandVO对象

转化为event或者vo后，将对象加入到PluginManager进行插件调度

指令调度器负责将CommandVO转化为CommandSender，args，指令名称，将调度Command对象

事件调度器将转化的event对象放入事件调度器，就会调用事件方法

当调用指令或者事件监听器内部调用api，会通过Server的发包器，将数据base64并转化为byte数组

ProxyHandler解析器会将数据再解析为jsmod2协议

会将解码后并拆分的json mapper(json转化的),id,end传输到NetworkHandler，NetworkHandler会根据id找到对应的子Handler，并将响应返回,如果是普通请求，则当Set和Get包处理，不是则当指令注册处理

ApiIdMapping是维护的对象映射表，会通过发包的uuid从里面查到对象并在子处理器操作,当jsmod2触发事件时候，会通过ProxyHandler代理解析器解析，然后发送到jsmod2的协议翻译器

注册时将CommandHandler注册进去，如果Smod2调用指令，则触发CommandHandler，然后发包

> jsmod2-server的组成

java端称为jsmod2-server，同时也可以叫做jsmod2，由core,api,protocol,child组成

* core包包含了jsmod2的核心部分，如一些模板，基本类，工具框架，协议处理器，http处理器等,为了方便移植和扩展功能

* api包含了基本的数据类(供插件开发调用的api),可以发送数据和封装等

* protocol 包含了定义的数据包，基于jsmod2-protocol的定义，适配于proxyHandler

* child是Server的子类，RegisterTemplate的子类等，负责jsmod2的启动

> jsmod2的设计优势
  
* 方便兼容机制

可以设置MultiAdmin的log文件，服务端可以监听log更新并输出在控制台，因此只需要管理Jsmod2的后台即可
并且支持迭代log文件处理和高效断枝算法，当产生最新log文件，jsmod2会自动替换到最新的配置文件去监听

* 自动注册机制

jsmod2支持自动注册监听器，加入@EnableRegister即可实现自动注册，但是插件必须是打包成jar包的文件

* 无配置文件模式

废除plugin.yml,实现@Main注解注册主类，类似于smod2的插件主类注册方式

* 配置文件框架

支持3种主流配置文件和一个服务端自带的配置文件的系统

* 高速处理算法

每秒处理20000条以上请求，并不丢包(在没有强烈网络波动下)，设计的缓冲字节算法，保证了不会出现丢包错误

* Simple Web API

可以通过在web发送jsmod2请求串(带着http协议),来获取数据json，方便设计网站联通服务端，以网站形式反映服务器数据

* 不同的监听器，注册事件更简易

使用@EventManager注解可以直接标记监听方法，加入自动注册，开发效率提升一半，无需关心监听器是否在主类使用过

```java
public class ExampleListener implements Listener{
  
  @EventManager
  public void onJoin(PlayerJoinEvent e){
    //监听内容
  }
}

```

* 指令权限控制

实现了指令通过玩家权限来控制的方案(此功能在测试阶段)

* emerald脚本引擎

jsmod2开发团队的老卢手撸的一个半吊子脚本引擎，可以实现命令行脚本，未来将实现游戏可以写脚本
目前还在测试，大部分功能没有完成，源代码:cn.jsmod2.core.script

* Application机制和扩展机制

你可以自己创建项目，引入core包，搭建一个diy的游戏服务端

* 测试数据测试

使用测试数据测试后，可以无需开启游戏实现测试功能

目前该功能在策划中

* 挂接注册机制

当事件发生或者请求发生时，将可设置对象存到hash映射表中，并赋予一个唯一的id号，当可设置对象在jsmod2端发送请求时

会根据请求指定的id找到对象，并对其进行设置，这个叫做挂接注册机制

> emerald语法

目前emerald处于初始开发阶段，语法不多(作者手撸了6个小时)

声明变量:a=value

  字符串是'value' bool类型是true|false int类型和double类型不多说
  
  使用变量${a}
  
  字符串可以拼接'a'+'b'='ab',拼接变量 'a'+'${a}'+'b'
  
  指针类型，初始化指针 a_ptr:*a 将取出a地址
  
声明函数:

func name(a1,*a2);start:

  code1;
  
  code2;
  
  return(0);
  
:end

a1是普通参数，值会复制，*a2是指针参数，值不会复制，但是前提是指针类型，并且这里可以允许多重指针的使用
如以下代码
func name(a1,*a2);start:
  a=1;
  *a2=12;
  return(${a});
:end

a=12
a_ptr:*a
a_ptr_ptr:*a_ptr
name(a,a_ptr)

> 控制台

服务端采用命令行模式操作服务器，其中支持emerald语言语法，服务器的自带命令机制，还有插件的注册命令(除了noConsole权限外),在架构图中没有提及到是因为它是简易的用户交互架构，使用help命令会调用Console类对象，Console类对象会找到指令映射表，把指令映射表打印出来，当发送一个命令，控制台会判断是否存在，不存在就返回指令不存在，要求输入help,如果存在，就判断权限是否符合控制台的权限，不符合则反应权限不足，符合执行命令，从命令映射表找到Command对象即可(PluginManager的指令调用方法)

> 注册机

注册机是jsmod2中的本地数据注册器，为了直观的反应服务端的字典数据，降低数据和代码的耦合性，以便扩展开发，它的子类是RegisterTemplate，通过注册注解使服务器查询到注册方法，注册机在服务器开始加载时就会被调用，注册机的数据是静态的，一旦确定，不会在服务器运行时改变，和api注册表不同，api注册表是一定变化的
