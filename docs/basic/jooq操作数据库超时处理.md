



# jooq操作数据库超时处理





## 全局配置超时时间

> https://www.jooq.org/doc/3.13/manual-single-page/#settings-jdbc-flags

```java
@Component
@Configuration
@Slf4j
public class JooqConfig {

    @Autowired
    DSLContext dslContext;

    @PostConstruct
    public void init(){
        dslContext.settings().setQueryTimeout(3);
    }
}
```



## 每个方法自己单独控制

org.jooq.impl.AbstractQuery#execute()



org.jooq.impl.DSL#using(java.lang.String)











https://www.iteye.com/blog/shift-alt-ctrl-2314088
https://blog.csdn.net/aa283818932/article/details/40379211/

=======================================mybatis配置sql超时时间

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD  Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 全局超时配置，300表示sql执行时间超过5分钟时，报错 -->
<configuration>
    <settings>
        <setting name="defaultStatementTimeout" value="300" />
    </settings>
</configuration>



<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations" value="classpath*:config/mybatis/**/mapper_*.xml" />
    <!-- 引入mysql的全局配置，超时，缓存等 -->
    <property name="configLocation" value="classpath:config/mybatis/mysql.xml" />
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
	<constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```





当然对于个别情况，有的sql需要执行很长时间或其他的话，可以对单个sql做个性化超时设置。

在mapper xml文件中对具体一个sql进行设置,方法为在select/update/insert节点中配置timeout属性,
依然是以秒为单位表示超时时间并只作用于这一个sql

```xml
<select id="queryList" parameterType="hashmap" timeout="10000">
```











===================================JPA中设置

JPA全局配置

```xml
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  
    <property name="jpaProperties">  
        <props>  
            <prop key="eclipselink.cache.shared.default">false</prop>  
            <prop key="eclipselink.weaving">false</prop>  
            <prop key="javax.persistence.query.timeout”>20000<prop/>  
        </props>  
    </property>  
</bean>
```



为当前查询设定timeout

```java
String sql = "SELECT ...";  
TypedQuery<User> query = entityManager.createQuery(sql, User.class);  
query.setParameter("id", id);  

// 此处设置timeout，单位：毫秒  
query.setHint("javax.persistence.query.timeout", 20000);  
```







================================在Mysql的默认设置中，如果一个数据库连接超过8小时没有使用
（闲置8小时，即28800s），mysql server将主动断开这条连接，后续在该连接上进行的查询操作都将失败，将出现：
error 2006 (MySQL server has gone away)!。


查看mysql server超时时间：
msyql> show global variables like '%timeout%';


设置mysql server超时时间（以秒为单位）
msyql> set global wait_timeout=10;

msyql> set global interactive_timeout=10;




show global variables like '%timeout%';
show global variables like '%execution_time%';




https://www.jianshu.com/p/99bafb3a466f
如何配置MySQL数据库超时设置


https://blog.csdn.net/sarahcla/article/details/78043501
=============================mysql语句执行超时设置


服务端设置

mysql 5.6 及以后，有语句执行超时时间变量，用于在服务端对 select 语句进行超时时间限制；
mysql 5.6 中，名为： max_statement_time （毫秒）
mysql 5.7 以后，改成： max_execution_time （毫秒）

超过这个时间，mysql 就终止 select 语句的执行，客户端抛异常：
1907: Query execution was interrupted, max_execution_time exceeded.

三种设置粒度：

（1）全局设置
SET GLOBAL MAX_EXECUTION_TIME=1000;

（2）对某个session设置
SET SESSION MAX_EXECUTION_TIME=1000;

（3）对某个语句设置
SELECT max_execution_time=1000 SLEEP(10), a.* from test a;




