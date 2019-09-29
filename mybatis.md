[TOC]

# #SqlSessionFactoryBuilder

创建SqlSessionFactory的类，最好使用过后就销毁以便释放资源，因此最好是个局部变量。

# #SqlSessionFactory

SqlSessionFactory被创建后在应用运行期间一直存在，没有理由重新创建或者丢弃它。

## ##通过XML配置文件创建SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

## ##通过代码配置创建SqlSessionFactory

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```



# #SqlSession

通过SqlSessionFactory获得SqlSession

```java
SqlSession session = factory.openSession();
List<House> selectList = session.selectList("com.obj.HouseMapper.selectHouses");
for (House house : selectList) {
	System.out.println(house);
}
session.close();
```





# #调用sql语句

## ##用ibatis方式

```java
SqlSession session = factory.openSession();
List<House> selectList = session.selectList("com.obj.HouseMapper.selectHouses");
```

## ##用Mapper接口方式

```java
SqlSession session = factory.openSession();
HouseMapper mapper = session.getMapper(HouseMapper.class);
List<House> houseList = mapper.getAllHouses();
```



# # ${} 和 #{}的区别

## ${}

直接替换

select id,name,age from student where name=${name}    —> name=cy

select id,name,age from student where name='${name}'    —> name='cy'

## #{}

占位符，根据类型替换

select id,name,age from student where name=#{name}   —> name='cy'



# #mysql查看执行sql语句的记录日志 	

1、使用processlist，但是有个弊端，就是只能查看正在执行的sql语句，对应历史记录，查看不到。好处是不用设置，不会保存。

-- use information_schema; 

-- show processlist;

 或者： 

-- select * from information_schema.`PROCESSLIST` where info is not null;

2、开启日志模式

-- 1、设置 

-- SET GLOBAL log_output = 'TABLE';SET GLOBAL general_log = 'ON';  //日志开启

-- SET GLOBAL log_output = 'TABLE'; SET GLOBAL general_log = 'OFF';  //日志关闭

-- 2、查询 

SELECT * from mysql.general_log ORDER BY event_time DESC;

-- 3、清空表（delete对于这个表，不允许使用，只能用truncate） 

-- truncate table mysql.general_log;

 

# #mybatis源码



## ##SqlSessionFactoryBuilder

**SqlSessionFactoryBuilder build方法** 

1. 解析XML配置文件生成Configuration对象

2. 传入Configuration对象到DefaultSqlSessionFactory的构造函数，生成对象



**Configuration：**

这个类里包含了配置，还有sql语句等信息，
sql语句被存进Map<String, MappedStatement> mappedStatements里。



**SqlSessionFactoryBuilder.class:**

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) 
	{
    try {
      // 1. 解析XML文件，加载配置进内存
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      // 2. 调用XMLConfigBuilder.class的parser.parse()，解析XML配置文件
      return build(parser.parse());
    } catch (Exception e) {
			...
    } finally {
      ...
    }
}

// 3. 解析完XML生成Configuration实例，然后调用此方法生成SqlSessionFactory
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
```

**XMLConfigBuilder.class:**
**parser.parse();**

```java
public Configuration parse() {
    ...
		// 调用parseConfiguration解析XML配置成Configuration对象
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}

private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```



## ##SqlSessionFactory接口

**实现类：**DefaultSqlSessionFactory

**作用：**

1. 获得SqlSession对象，通过调用openSession方法
2. 获得Configuration对象，通过get方法



## ##SqlSession接口

**实现类：**DefaultSqlSession

**作用：**

1. 提供数据访问API (例如selectOne方法)
2. 把所有数据访问操作交给内部的一个private Executor对象处理