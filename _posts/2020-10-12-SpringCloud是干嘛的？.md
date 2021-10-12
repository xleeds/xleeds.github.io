---
layout:     post
title:      SpringCloud是干嘛的
subtitle:   学习笔记-SpringCloud框架
date:       2020-10-12
author:     Leeds
header-img: img/post-bg-miui6.jpg
catalog: 	  true
tags:
    - 学习资料
    - SpringCloud
---

##  SpringCloud是干嘛的

**以A服务想要调用B服务为例**

1. Eureka组件：微服务架构的注册中心，专门负责服务的注册与发现。

   ​	没有之前：A服务想要调用B服务，但不知道B服务在哪台机器上，端口号是多少；或者只能写死IP和端口号。

   ​	有了：

   - A和B服务的Eureka Client都向Eureka Server服务注册自己的所在机器的IP和端口号。
   - A服务的Eureka Client向Eureka Server请求B服务的IP和端口并缓存在本地。
   - A服务调用B服务直接问本地的Eureka Client，B服务的IP和端口。

   ![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CvHjEPqBW2iclGBwTzB4sD6VR2NU0xmMsoYicmWCXfPCQTn1LticB61OWA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2. Feign组件：使用动态代理，根据你的注解完成建立连接、构造请求、发起请求、获取响应、解析响应等一系列工作。

   ​	没有之前：写一大堆重复的请求，解析响应等HTTP调用代码

   ​	有了：

   - 如果你对某个接口定义了@FeignClient注解，Feign会对这个接口创建一个动态代理
   - 接着你调用那个接口，**本质上就是调用Feign创建的动态代理**		
   - Feign的动态代理会根据你在接口上的@RequestMapping等注解，来**动态的构造出你请求的服务的地址**
   - 针对这个请求，发起请求、解析响应

   ![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CT5y3EDh8DaEmDM4ibRNW0aTiaD8opMUFUCKX5yYxnGaqc7Kz7dicWm2kw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   

3. Ribbon组件：对请求做负载均衡，帮你在每次请求时选择一台机器，均匀的把请求分发到各个机器上

   <!--微服务架构中，很有可能发生服务挂掉的情况，所以为了系统的高可用，一个服务通常都会部署多个实例-->

   没有之前：无法做负载均衡，不知道应该请求哪台服务器

   有了：

   - Ribbon会从Eureka Client获取到对应服务的注册表，即所有服务的IP和端口号
   - Ribbon使用默认的Round Robin算法（轮询），从中选择一台机器
   - Feign针对这台机器，构造发起请求	

   ![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CtD7kdmviciaXUubWd69erjZn8Hgw5ZZrDWYDBYCAiaBchGDxxwQj7Lic0w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4. Hystrix组件：隔离、熔断、降级。Hystrix会搞很多小线程池，A调用B是一个线程池，A调用C又是一个线程池。

   B服务挂掉了，每次A服务调用时，都会卡住几秒，然后抛出异常

   没有之前：

   - 如果系统处于高并发的场景下，大量请求涌过来，A服务的100个线程都会卡在调用B服务这里，导致A服务没有一个线程可以处理请求
   - 当其他服务调用A服务的时候，发现A服务也挂掉了，不会响应任何请求，从而引发雪崩

   有了：

   - 隔离：由于每个调用（A调B，A调C）是一个独立的线程池，所以B服务挂了，也不会影响A调C服务。
   - 熔断：如果B服务挂了，每次还要去请求并且卡住几秒钟没有意义，所以对A调用B服务在一定时间内，不走网络，直接返回。
   - 降级：B服务挂了，熔断时，可以走降级服务。就是熔断不能正常处理业务逻辑时的默认处理，比如记录日志，返回一个默认值。

   ![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CZsj0neCTAwgfBVClSwTarrZ7gMRKAQlNviabrsOJ0DA9GcSmuZ2BTcQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   熔断的机制：

   ![img](https://img-blog.csdn.net/20171030152111814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFveWVxaXU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

   服务的健康状况 = 请求失败数 / 请求总数. 
   熔断器开关由关闭到打开的状态转换是通过当前服务健康状况和设定阈值比较决定的.

   1. 当熔断器开关关闭时, 请求被允许通过熔断器. 如果当前健康状况高于设定阈值, 开关继续保持关闭. 如果当前健康状况低于设定阈值, 开关则切换为打开状态.
   2. 当熔断器开关打开时, 请求被禁止通过.
   3. 当熔断器开关处于打开状态, 经过一段时间后, 熔断器会自动进入半开状态, 这时熔断器只允许一个请求通过. 当该请求调用成功时, 熔断器恢复到关闭状态. 若该请求失败, 熔断器继续保持打开状态, 接下来的请求被禁止通过.

   熔断器的开关能保证服务调用者在调用异常服务时, 快速返回结果, 避免大量的同步等待. 并且熔断器能在一段时间后继续侦测请求执行结果, 提供恢复服务调用的可能.

   

5. Zuul组件：微服务网关，负责网络路由。所有请求都往网关走，网关会根据请求中的一些特征，将请求转发给后端的各个服务。

   没有之前：前端请求服务，需要记住所有后台服务的名字，并且写死某台服务的地址。

   有了：

   - 做统一的降级、限流、认证授权、安全等。

   

![image-20211012135332156](../img-post/image-20211012135332156.png)

[参考自石杉的架构笔记]:https://mp.weixin.qq.com/s/mOk0KuEWQUiugyRA3-FXwg



