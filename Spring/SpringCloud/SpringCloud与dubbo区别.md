# SpringCloud 与 Dubbo 的区别

> 来源：`SpringCloud/SpringCloud与dubbo区别.md`

Dubbo是一个RPC框架，而SpringCloud的定位是微服务下的一站式解决方案

Dubbo基于RPC，而SpringCloud基于HTTP的Rest风格

Dubbo的注册中心一般是用Zookeeper、SpringCloud使用eureka（都可以更换成其他的注册中心）

## 二、RPC 与 REST 的区别


1. 所属类别不同
RPC一般基于TCP/UDP也可以基于HTTP，Rest基于HTTP
2. 使用方式不同
从使用上来看，HTTP 接口只关注服务提供方，对于客户端怎么调用并不关心。接口只要保证有客户端调用时，返回对应的数据就行了。而RPC则要求客户端接口保持和服务端的一致。REST 是服务端把方法写好，客户端并不知道具体方法。客户端只想获取资源，所以发起HTTP请求，而服务端接收到请求后根据URI经过一系列的路由才定位到方法上面去RPC是服务端提供好方法给客户端调用，客户端需要知道服务端的具体类，具体方法，然后像调用本地方法一样直接调用它。
3. 面向对象不同
从设计上来看，RPC，所谓的远程过程调用 ，是面向方法的 ，REST：所谓的 Representational state transfer ，是面向资源的
4. 序列化协议不同
REST 通常使用的是 JSON或者XML，而 RPC 使用的是 JSON-RPC，或者 XML-RPC

![图片](../../assets/SpringCloud与dubbo区别/img-01.webp)

RPC 和 REST 两者的定位不同，REST 面向资源，更注重接口的规范，因为要保证通用性更强，所以对外最好通过 REST。而 RPC 面向方法，主要用于函数方法的调用，可以适合更复杂通信需求的场景。RESTful API客户端与服务端之间采用的是同步机制，当发送HTTP请求时，客户端需要等待服务端的响应。

REST 接口更加规范，通用适配性要求高，建议对外的接口都统一成 REST。而组件内部的各个模块，可以选择 RPC，一个是不用耗费太多精力去开发和维护多套的HTTP接口，一个RPC的调用性能更高（见下条）从性能角度看，由于HTTP本身提供了丰富的状态功能与扩展功能，但也正由于HTTP提供的功能过多，导致在网络传输时，需要携带的信息更多，从性能角度上讲，较为低效。而RPC服务网络传输上仅传输与业务内容相关的数据，传输数据更小，性能更高

Dubbo：

支持多传输协议(Dubbo、Rmi、http、redis等等)，可以根据业务场景选择最佳的方式。非常灵活。

默认的Dubbo协议：利用Netty，TCP传输，单一、异步、长连接，适合数据量小、高并发和服务提供者远远少于消费者的场景。

Feign：基于Http传输协议，短连接，不适合高并发的访问。

负载均衡

Dubbo：

支持4种算法（随机、轮询、活跃度、Hash一致性），而且算法里面引入权重的概念。

配置的形式不仅支持代码配置，还支持Dubbo控制台灵活动态配置。

负载均衡的算法可以精准到某个服务接口的某个方法。

Feign：

只支持N种策略：轮询、随机、ResponseTime加权。

负载均衡算法是Client级别的。

容错策略

Dubbo：支持多种容错策略：failover、failfast、brodecast、forking等，也引入了retry次数、timeout等配置参数。

Feign：利用熔断机制来实现容错的，处理的方式不一样。
