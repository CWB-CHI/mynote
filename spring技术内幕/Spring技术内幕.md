[TOC]



# IOC 容器的实现

## IOC的设计与实现

<img src="img/ioc设计.png" alt="image-2019101720154519" style="zoom:40%;" />

### 两条重要的设计路线

BeanFactory和ApplicationContext是两个重要的接口，也是IOC的具体体现。

* 红线: 
  1. BeanFactory 
  2. HierarchicalBeanFactory 实现父子容器的相关接口
  3. ConfigurableBeanFacotry 定义配置BeanFactory的方法

* 蓝线: 
  1. BeanFactory 
  2. ListableBeanFactory 细化BeanFactory的接口功能
  3. ApplicationContext 
  4. WebApplicationContext / ConfigurableApplicationContext

