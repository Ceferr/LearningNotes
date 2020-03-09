# 一.SpringBoot与数据访问

## 1.JDBC

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

```yml
spring:
  datasource:
#   数据源基本配置
    username: root
    password: xxxxxx
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://47.102.149.102:3306/tmall?serverTimezone=GMT%2B8&characterEncoding=utf8
```

效果：

​	默认使用class com.zaxxer.hikari.HikariDataSource数据源；

​	数据源的相关配置都在DataSourceProperties里面；

自动配置原理：

org.springframework.boot.autoconfigure.jdbc

1.参考DataSourceConfiguration，根据配置常见数据源，默认使用Hikari连接池；可以使用spring.datasource.type指定自定义的数据源类型；

2.SpringBoot默认支持：

```
org.apache.commons.dbcp2.BasicDataSource、com.zaxxer.hikari.HikariDataSource、
org.apache.tomcat.jdbc.pool.DataSource
```

3.自定义数据源类型

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(name = {"spring.datasource.type"})
static class Generic {
    Generic() {}

    @Bean
    DataSource dataSource(DataSourceProperties properties) {
      //使用DataSourceBuilder创建数据源，利用反射创建相应type的数据源，并且绑定相关属性
        return properties.initializeDataSourceBuilder().build();
    }
}
```

## 2.整合Druid数据源

配置过滤取和拦截器

```java
导入druid数据源
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }

    //配置Druid的监控
    //1.配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(),"/druid/*");

        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456a");
        initParams.put("allow","");//默认允许所有

        bean.setInitParameters(initParams);
        return bean;
    }

    //2.配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}

```



# 二.SpringBoot与缓存

缓存注解

| Cache          | 缓存接口，定义缓存等操作，实现有RedisCache、EhCacheCache、ConcurrentMapCache等 |
| -------------- | :----------------------------------------------------------- |
| CacheManager   | 缓存管理器，管理各种缓存(Cache)组件                          |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又希望结果被缓存                             |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

- @Cacheable

  CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每个缓存组件有自己唯一一个名字。

  几个属性：

  - cacheNames/value：指定缓存组件的名字
  - key：缓存数据使用的key；可以用它来指定，默认是使用方法参数的值
  - keyGenerator：key的生成器；可以自己指定key的生成器的组件id
  - cacheManager：指定缓存管理器，或者cacheResolver指定缓存解析器
  - condition：指定符合条件下才缓存
  - unless：否定缓存，当unless条件为true时，方法的返回值不会被缓存；可以获取到结果进行判断 unless = "#result == null"
  - sync：是否使用异步模式