
# 4\. Spring Cloud Ribbon 实现“负载均衡”的详细配置说明


@

目录* [4\. Spring Cloud Ribbon 实现“负载均衡”的详细配置说明](https://github.com)
* [前言](https://github.com)
* [1\. Ribbon 介绍](https://github.com)
	+ [1\.1 LB(Load Balance 负载均衡)](https://github.com)
* [2\. Ribbon 原理](https://github.com)
	+ [2\.2 Ribbon 机制](https://github.com)
* [3\. Spring Cloud Ribbon 实现负载均衡算法\-应用实例](https://github.com):[MeoMiao 萌喵加速](https://biqumo.org)
* [4\. 总结：](https://github.com)
* [5\. 最后：](https://github.com)



---


# 前言



> * 对应上一篇学习内容：🌟🌟🌟[ChinaRainbowSea\-CSDN博客](https://github.com)
> * 对应下一篇学习内容：🌟🌟🌟


# 1\. Ribbon 介绍


**Ribbon 是什么 ？**


1\.Spring Cloud Ribbon 是基于Netflix Ribbon 实现的一套客户端，负载均衡的工具
2\.Ribbon 主要功能是提供客户端负载均衡算法和服务调用
3\.Ribbon 客户端组件提供一系列完善的配置项如“连接超时，重试”
4\.Ribbon 会基于某种规则（如简单轮询，随机连接等）去连接指定服务
5\.程序员很容易是由 Ribbon 的负载均衡算法实现负载均衡
6\.一句话: Ribbon: 负载均衡 \+ RestTemplate 调用
Ribbon 的官网地址: [https://github.com/Netflix/ribbon](https://github.com)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252710-1606620906.png)


**Ribbon 进入维护状态？**


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252699-652835576.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252663-1016783865.png)



> **Ribbon 目前进入维护模式, 未来替换方案 是 Spring Cloud LoadBalancer**


## 1\.1 LB(Load Balance 负载均衡)


**LB 分类**


1. **集中式: LB**


即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5，也可以是软件，如 Nginx)
由该设施负责把访问请求通过某种策略转发至服务的提供方。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252658-2052867128.png)


2. **进程内 LB**


将 LB逻辑集成到消费方，消费方服务注册中心获知有哪些地址可用，然后再从这些地址中选择
一个合适的服务地址


Ribbon 就属于进程内 L B，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252735-1356360908.png)



> 特别说明：在上一篇的学习内容上：实例\-前面 member\-consumer 轮询负载访问 10000/10002 底层就是 Ribbon 默认的轮询负载算法。


# 2\. Ribbon 原理


**Ribbon 架构图\&机制**


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252711-1681691481.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252717-661712657.png)


## 2\.2 Ribbon 机制


**Ribbon 机制：**



> 1. 先选择 EurekaServer，它**优先** 选择在同一个区域内负载较少的 **Server** 服务。
> 2. 再根据用户指定的策略，在从 Server 取到的服务注册列表中选择一个地址
> 3. Ribbon 提供了多种策略：比如：**轮询，随机和根据响应时间加权等等** 。


**Ribbon 常见负载算法**


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252728-1229789912.png)


# 3\. Spring Cloud Ribbon 实现负载均衡算法\-应用实例


需求分析/图解


1. 需求: 将默认的轮询算法改成随机算法 RandomRule
2. 浏览器输入 : [http://localhost/member/consumer/get/1](https://github.com)
3. 要求 访问的 10000/10002 端口的服务是随机的，也就是设置为：**RandomRule** 随机负载均衡


其实：所谓的随机是在少量数据上是随机的，但是在大量数据的基础上，随机的结果基本上和**平均** 差不多。


1. 首先：创建 member\-service\-consumer\-80 com/rainbowsea/springcloud/config/RibbonRule.java，随机负载均衡的算法的配置类，进行配置**随机负载均衡** 。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252726-30036948.png)



```
package com.rainbowsea.springcloud.config;


import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RibbonRule {

    @Bean // 配置注入自己的负载均衡算法
    public IRule myRibbonRule() {
        // 这里老师返回的是 RandomRule，大家也可以指定其他的内容。
        //return new WeightedResponseTimeRule();
        return new RandomRule();  // 随机算法

    }
}


```


> 注意：同时还需要在对应的**Eureka Client 客户端**当中的 `RestTemplate` Http模板 的位置开启， 加入 `@LoadBalanced` 开启负载均衡
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252723-1921595720.png)



> 以及同时还需要在对应的**Eureka Client 客户端** 的场景启动器的位置开启，当中加入`@RibbonClient` 注解标明：使用的是 Ribbon 下的负载均衡上的哪个算法。
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252774-1371571710.png)



> ```
> @RibbonClient(name = "MEMBER_SERVICE_PROVIDER_URL",configuration = RibbonRule.class) // 指定 Ribbon 的负载均衡算法
> 
> ```
> 
> 
> ```
> package com.rainbowsea.springcloud;
> 
> 
> import com.rainbowsea.springcloud.config.RibbonRule;
> import org.springframework.boot.SpringApplication;
> import org.springframework.boot.autoconfigure.SpringBootApplication;
> import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
> import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
> import org.springframework.cloud.netflix.ribbon.RibbonClient;
> 
> // 如果错误: 加入排除 DataSourceAutoConfiguration 自动配置
> //@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
> //@EnableEurekaClient 表示将项目/模块/程序 标注作为 Eureka Client
> @EnableEurekaClient
> @SpringBootApplication
> @EnableDiscoveryClient // 启用服务发现
> @RibbonClient(name = "MEMBER_SERVICE_PROVIDER_URL",configuration = RibbonRule.class) // 指定 Ribbon 的负载均衡算法
> public class MemberConsumerApplication80 {
>     public static void main(String[] args) {
>         SpringApplication.run(MemberConsumerApplication80.class, args);
>     }
> }
> 
> 
> ```


**测试**


1. 浏览器输入 : [http://localhost/member/consumer/get/1](https://github.com)
2. 观察访问的 10000/10002 端口的服务是随机的。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252737-1339445386.png)


# 4\. 总结：


1. Spring Cloud Ribbon 是基于Netflix Ribbon 实现的一套客户端，负载均衡的工具。
2. LB(Load Balance 负载均衡) 两种分类：
	1. **集中式: LB**
	2. **进程内 LB**
3. Ribbon 原理和机制



> 1. 先选择 EurekaServer，它**优先** 选择在同一个区域内负载较少的 **Server** 服务。
> 2. 再根据用户指定的策略，在从 Server 取到的服务注册列表中选择一个地址
> 3. Ribbon 提供了多种策略：比如：**轮询，随机和根据响应时间加权等等** 。


4. Ribbon 常见的负载均衡算法。
5. 配置 Spring Cloud Ribbon 的三点：



> 1. 创建配置对应 Ribbon 下的`负载均衡的算法的配置类(注意标注配置类注解@Configuration 和对应的方法处加入'@Bean加入 ioc 容器管理' )`，进行配置**负载均衡** ，直接 new 对应的 Ribbon 下的 负载均衡算法即可，比如： RandomRule，RoundRobinRule
> 2. 还需要在对应的**Eureka Client 客户端**当中的 `RestTemplate` Http模板 的位置开启， 加入 `@LoadBalanced` 开启负载均衡`@LoadBalanced 注解标注负载均衡`
> 3. 同时还需要在对应的**Eureka Client 客户端** 的场景启动器的`类上` 位置开启，当中加入`@RibbonClient` 注解标明：使用的是 Ribbon 下的负载均衡上的哪个算法。
> 
> 
> 
> ```
> @RibbonClient(name = "MEMBER_SERVICE_PROVIDER_URL",configuration = RibbonRule.class) // 指定 Ribbon 的负载均衡算法
> 
> ```


# 5\. 最后：



> “在这个最后的篇章中，我要表达我对每一位读者的感激之情。你们的关注和回复是我创作的动力源泉，我从你们身上吸取了无尽的灵感与勇气。我会将你们的鼓励留在心底，继续在其他的领域奋斗。感谢你们，我们总会在某个时刻再次相遇。”
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202411/3084824-20241115142252831-2143904551.gif)


