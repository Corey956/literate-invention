---
layout: mypost
title: CompletableFuture 问题
categories: [Java]
extMath: true


---

# CompletableFuture 与 Openfeign 一起使用的问题

## 问题描述



### 直接错误信息

```java
Could not find class 
[org.springframework.boot.autoconfigure.condition.OnPropertyCondition]， 
[org.springframework.util.ClassUtils.resolveClassName(ClassUtils.java:334) ,
org.springframework.context.annotation.ConditionEvaluator.getCondition(ConditionEvaluator.java:124) ,
org.springframework.context.annotation.ConditionEvaluator.shouldSkip(ConditionEvaluator.java:96) ,
org.springframework.context.annotation.ConditionEvaluator.shouldSkip(ConditionEvaluator.java:88) ,
org.springframework.context.annotation.ConditionEvaluator.shouldSkip(ConditionEvaluator.java:71) ,
org.springframework.context.annotation.AnnotatedBeanDefinitionReader.doRegisterBean(AnnotatedBeanDefinitionReader.java:254) ,
org.springframework.context.annotation.AnnotatedBeanDefinitionReader.registerBean(AnnotatedBeanDefinitionReader.java:147) ,
org.springframework.context.annotation.AnnotatedBeanDefinitionReader.register(AnnotatedBeanDefinitionReader.java:137) ,
org.springframework.context.annotation.AnnotationConfigApplicationContext.register(AnnotationConfigApplicationContext.java:162) ,
org.springframework.cloud.context.named.NamedContextFactory.createContext(NamedContextFactory.java:120) ,
org.springframework.cloud.context.named.NamedContextFactory.getContext(NamedContextFactory.java:102) ,
org.springframework.cloud.netflix.ribbon.SpringClientFactory.getContext(SpringClientFactory.java:131) ,
org.springframework.cloud.context.named.NamedContextFactory.getInstance(NamedContextFactory.java:146) ,
org.springframework.cloud.netflix.ribbon.SpringClientFactory.getInstance(SpringClientFactory.java:121) ,
org.springframework.cloud.netflix.ribbon.SpringClientFactory.getClientConfig(SpringClientFactory.java:75) ,
org.springframework.cloud.openfeign.ribbon.CachingSpringLoadBalancerFactory.create(CachingSpringLoadBalancerFactory.java:60) ,
org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.lbClient(LoadBalancerFeignClient.java:121) ,
org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.execute(LoadBalancerFeignClient.java:83) ,
feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:119) ,
feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:89) ,
com.alibaba.cloud.sentinel.feign.SentinelInvocationHandler.invoke(SentinelInvocationHandler.java:107) ,
com.sun.proxy.$Proxy257.serviceCall(Unknown Source) ,
***.application.impl.HandlingFirmwareBusinessImpl.serviceCallUnbind(HandlingFirmwareBusinessImpl.java:152) ,
***.application.impl.HandlingFirmwareBusinessImpl.execute(HandlingFirmwareBusinessImpl.java:117) ,
***.user.infrastructure.mq.kafka.MessageConsumer.lambda$handleDeviceNotifyExecuteResult$0(MessageConsumer.java:54) ,
java.base/java.util.concurrent.CompletableFuture$AsyncRun.run(Unknown Source) ,
java.base/java.util.concurrent.CompletableFuture$AsyncRun.exec(Unknown Source) ,
java.base/java.util.concurrent.ForkJoinTask.doExec(Unknown Source) ,
java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(Unknown Source) ,
java.base/java.util.concurrent.ForkJoinPool.scan(Unknown Source) ,
java.base/java.util.concurrent.ForkJoinPool.runWorker(Unknown Source) ,
java.base/java.util.concurrent.ForkJoinWorkerThread.run(Unknown Source)]
```

### 使用地方

```java
List<CompletableFuture<Void>> futureList = Lists.newArrayListWithCapacity(records.size());
for (ConsumerRecord<String, String> record : records) {
    futureList.add(CompletableFuture.runAsync(() -> {
       // feign 调用
    }));
}
for (CompletableFuture<Void> future : futureList) {
    future.get();
}

```

## 问题分析解决

这个问题存在很久，Spring Cloud 官方已经对该问题有说明

[openfeign 说明 ]:https://github.com/spring-cloud/spring-cloud-openfeign/issues/475

在这个链接问题下有开发者集体出了一些解决方案，可以参考尝试

现在官方已经有了解决版本

[解决方案]:hthttps://github.com/spring-cloud/spring-cloud-commons/pull/1256

### 分析该问题

在使用CompletableFuture.runAsync()没有指定线程会默认调用原本的默认全局共享的线程池ForkJoinPool.commonPool()，并行流（多线程）中调用 FeignBlockingLoadBalancerClient时会出现 ClassNotFoundException
	也就是说如果我们在ThreadPoolExecutor自定义线程池或者使用jdk提供的线程池（例如FixedThreadPool）中调用Fegin接口是不会报错的，而恰巧CompletableFuture中的默认线程池也是通过ForkJoinPool线程池来实现的
	此类问题只有在openJDK11版本下才会出现这个问题，并且如果再ForkJoinPool中调用Fegin调用就会出现这个问题。
	所以环境使用的是openJDK11版本，并且在使用异步调用的时候里面有fegin操作的时候，建议定义一个自定义的线程池，不可使用默认线程池(ForkJoinPool)