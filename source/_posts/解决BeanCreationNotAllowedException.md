---
title: 解决BeanCreationNotAllowedException
id: bean-creation-not-allowed-exception
date: 2018-12-05 10:38:10
updated: 2018-12-05 10:38:10
categories:
  - 那些古怪又令人忧心的问题
tags:
  - java
  - spring
  - exception
---
新接手一个springboot项目，本来想先启动一下试试，结果一启动就看控制台打印如下错误：

```
2018-12-05 09:54:09.752  WARN [pilipa-outside,,,] 58218 --- [           main] s.c.a.AnnotationConfigApplicationContext : Exception thrown from ApplicationListener handling ContextClosedEvent

org.springframework.beans.factory.BeanCreationNotAllowedException: Error creating bean with name 'rabbitConnectionFactory': Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:208)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:315)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:204)
	at org.springframework.context.event.AbstractApplicationEventMulticaster.retrieveApplicationListeners(AbstractApplicationEventMulticaster.java:239)
	at org.springframework.context.event.AbstractApplicationEventMulticaster.getApplicationListeners(AbstractApplicationEventMulticaster.java:196)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:133)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:400)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:406)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:354)
	at org.springframework.context.support.AbstractApplicationContext.doClose(AbstractApplicationContext.java:1000)
	at org.springframework.context.support.AbstractApplicationContext.close(AbstractApplicationContext.java:967)
	at org.springframework.cloud.context.named.NamedContextFactory.destroy(NamedContextFactory.java:76)
	at org.springframework.beans.factory.support.DisposableBeanAdapter.destroy(DisposableBeanAdapter.java:256)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroyBean(DefaultSingletonBeanRegistry.java:571)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingleton(DefaultSingletonBeanRegistry.java:543)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingleton(DefaultListableBeanFactory.java:954)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingletons(DefaultSingletonBeanRegistry.java:504)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingletons(DefaultListableBeanFactory.java:961)
	at org.springframework.context.support.AbstractApplicationContext.destroyBeans(AbstractApplicationContext.java:1041)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:563)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:762)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:398)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:330)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1258)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1246)
	at com.pilipa.outside.OutsideApplication.main(OutsideApplication.java:20)
```

<!-- more -->

这是怎么回事呢？首先怀疑了下是不是自己的mq有问题，连接不上之类的。但是自己做的另一个项目，也用到了mq，就能在本地启动起来，两个项目的mq配置是一样。

接下来就怀疑是不是其他地方出了问题，导致mq出了问题。

根据异常栈信息从下往上看，本来是在执行run()，怎么run着run着就执行了一个destroyBeans()呢？

我进入destroyBeans()的前一步，也就是refresh()方法里去看看，代码如下：

```
public void refresh() throws BeansException, IllegalStateException {
    Object var1 = this.startupShutdownMonitor;
    synchronized(this.startupShutdownMonitor) {
      this.prepareRefresh();
      ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
      this.prepareBeanFactory(beanFactory);

      try {
        this.postProcessBeanFactory(beanFactory);
        this.invokeBeanFactoryPostProcessors(beanFactory);
        this.registerBeanPostProcessors(beanFactory);
        this.initMessageSource();
        this.initApplicationEventMulticaster();
        this.onRefresh();
        this.registerListeners();
        this.finishBeanFactoryInitialization(beanFactory);
        this.finishRefresh();
      } catch (BeansException var9) {
        if (this.logger.isWarnEnabled()) {
          this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
        }

        this.destroyBeans();
        this.cancelRefresh(var9);
        throw var9;
      } finally {
        this.resetCommonCaches();
      }

    }
  }
```
可以看出来执行destroyBeans()方法是因为真长refresh()时出现了一个异常，所以catch里做了对之前初始化的回滚动作，然后在回滚动作里销毁singleton bean时出现了另一个异常，即当前打印出的异常，真正的导致原因，应该在上一条warn日志里。

于是往上翻，果然有一句从控制台一样望去还真不会注意到的：

```
2018-12-05 09:54:09.749  WARN [pilipa-outside,,,] 58218 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskBiz': Unsatisfied dependency expressed through field 'taskService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskServiceImpl': Unsatisfied dependency expressed through field 'taskInsertService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskInsertServiceImpl': Unsatisfied dependency expressed through field 'userClient'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'com.pilipa.outside.client.UserClient': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: PathVariable annotation was empty on param 0.

```
进入代码一看：
```
@GetMapping("/company/list/oldCompanyId/{oldCompanyId}")
ResponseEntity<CompanyDTO> findByOldCompanyId(@PathVariable String companyId);
```
因为上面的路径参数叫oldCompanyId，而方法参数中，变量名称为companyId，且没有标注名称，所以上下参数名称不对应，所以就找不到路径参数对应的入参了，把它改成一致就可以了。

其实通过代码看这个异常在destroyBeans()之后也会抛出来，在控制台往下拉也能找到
```
2018-12-05 09:54:10.100 ERROR [pilipa-outside,,,] 58218 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskBiz': Unsatisfied dependency expressed through field 'taskService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskServiceImpl': Unsatisfied dependency expressed through field 'taskInsertService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'taskInsertServiceImpl': Unsatisfied dependency expressed through field 'userClient'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'com.pilipa.outside.client.UserClient': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: PathVariable annotation was empty on param 0.
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:586)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:91)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:372)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1341)
	...

```
但是因为不是第一个抛出的异常，中间隔了好几个长长的异常堆栈打印，所以注意力全被rabbitConnectionFactory吸走了。

所以异常还是要看全，仔细看，大多数问题都能从异常信息里看出来，能节省很多时间。像java这样能把异常信息打印如此完善的，更应该好好研究，不要做灯下黑。

至于说，为什么之前做的人没有发现呢？

第一是因为，项目里有很多外部依赖，什么mq啊，mongo啊，redis啊，之类的，因为测试环境搭建不完善，想要服务跑起来需要本地搭建，有的人本地起个docker就完了，但是有的人电脑是windows，本地没配这些东西，平常提交之前只能跑一下集成测试，不能把程序运行起来。而有些能本地运行的，可能是由于粗心也没有进行本地运行就提交上去了。

第二是因为，这是前一天晚上新提交的bug。

![emmm](http://cdn.standbyside.com/emoticon/animal/emmm.jpg)

