[TOC]

# RequestContextListener

web.xml:

```xml
<listener-class>
	org.springframework.web.context.request.RequestContextListener
</listener-class>
```

当bean的scope设置成request、session时候，需要配置此监听器，使得Spring容器与Web容器的HTTP请求事件建立联系，辅助完成bean的创建销毁。



# DispatcherServlet

web.xml:

```xml
<servlet>
  <servlet-name>dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring-mvc.xml</param-value>
	</init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
```

配置Spring mvc需要在web.xml添加上述内容，如果没有设置init-param里的contextConfigLocation，默认会在/WEB-INF下找文件dispatcher-servlet.xml。

# Spring事务

全局事务: JTA事务[Java Transaction API，分布式事务]
局部事务: JDBC事务、Hibernate事务

局部事务:

```xml
<!-- 配置事务管理 -->
<bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">     
	<property name="dataSource" ref="dataSource"></property>
</bean> 


<!-- 1 非注解配置-->
<!-- 事务相关控制配置：例如配置事务的传播机制 -->
<tx:advice id="iccardTxAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="delete*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" no-rollback-for="java.lang.RuntimeException"/>
		<tx:method name="insert*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="add*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="create*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="update*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" /> 
		<tx:method name="find*" propagation="SUPPORTS" />
		<tx:method name="get*" propagation="SUPPORTS" />
    <tx:method name="select*" propagation="SUPPORTS" />
    <tx:method name="query*" propagation="SUPPORTS" />
	</tx:attributes>
</tx:advice>
    	
<!-- 把事务控制在service层 -->
<aop:config>    
	<aop:pointcut id="iccardTerm" expression="execution(public * com.shfft.iccardterm.service.*.*(..))" />
  <aop:advisor pointcut-ref="iccardTerm" advice-ref="iccardTxAdvice" />
</aop:config>
<!-- 1 -->

<!-- 2 注解配置 @Transactional-->
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
<!-- 2 注解配置-->


```



# @ResponseBody
Spring mvc返回json

**dispatcher-servlet.xml**:

```xml
	<context:component-scan base-package="com.ll.controller" />
	<mvc:annotation-driven />
```



# Spring mvc 接受请求参数

只要form里字段的name与User对象里字段对应，User对象就会被注入。

User.java [要具有getter setter]

```java
public class User{
  private String username;
  private String password;
  
  getter/setter...
}
```

Controller:

```java
...
  @RequestMapping(value="login")
	public String login(User user) {
  	...
  	return "index";
	}  
...
```

html:

```html
<form action="..." method="...">
  <input name="username" />
  <input name="password" />
</form>
```



# Spring MVC post请求编码

在web.xml配置filter

```xml
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```



# Spring MVC重定向

直接return想要定向的地址