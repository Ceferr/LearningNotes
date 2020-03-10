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

## 1.@Cacheable

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

## 2.缓存工作原理

1. 自动配置类 CacheAutoConfiguration

2. 缓存的配置类：

   0 = "org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration"
   1 = "org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration"
   2 = "org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration"
   3 = "org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration"
   4 = "org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration"
   5 = "org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration"
   6 = "org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration"
   7 = "org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration"
   8 = "org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration"
   9 = "org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration"

3. 默认是SimpleCacheConfiguration；

   给容器注册了一个CacheManager:ConcurrentMapManager；

   获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中

## 3.运行流程

1. 方法运行之前，先去查询Cache(缓存组件)，按照cacheNames指定的名字获取；

   （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。

2. 去Cache中查找缓存的内容，使用一个key，默认就是方法的参数

   ke是按照某种缓存策略生成的；默认是使用keyGenerator生成的额，默认使用SimpleKeygenerator生成key；

   ​		SimpleKeyGenerator 生成key的默认策略；

   ​										如果没有参数：key = new SimpleKey();

   ​										如果有一个参数：key = 参数的值

   ​										如果有多个参数：key = new SImplekey(params);

3. 没有查到缓存久调用目标方法；

4. 将目标方法返回的结果，放进缓存中



**@Cacheable标注的方法执行之前先检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存**，如果没有就运行方法并将结果放入缓存，以后再来调用就可以直接使用缓存中的数据；



**核心：**

1. 使用CacheManager[ConcurrentMapCacheManager]按照名字得到Cache[ConcurrentMapCache]组件
2. key使用keyGenerator生成的，默认是SimpleKeyGenerator  



## 4.@CachePut

修改了数据库的某个数据，同时更新缓存

 运行时机：

1. 先调用目标方法
2. 将目标方法的结果缓存起来



## 5.@CacheEvict

缓存清除

key:指定要清除的缓存

allEntries = true 指定清除这个缓存中的所有数据

beforeInvocation = false 缓存的清楚是否在方法之前执行

​				默认代表缓存清除操作是在方法之后执行，如果出现异常缓存就不会清除

beforeInvocation = true;

​				代表清除缓存操作是在方法执行之前执行，无论是否出现异常，都会清除缓存的数据



## 6.整合Redis

```java
stringRedisTemplate.opsForValue();
stringRedisTemplate.opsForList();
stringRedisTemplate.opsForSet();
stringRedisTemplate.opsForHash();
stringRedisTemplate.opsForZSet();
```

1. 当引入了redis的starter后，容器中保存的是RedisCacheManager

2. RedisCacheManager帮我们创建RedisCache作为缓存组件；RedisCache通过操作redis缓存数据的

3. 默认保存数据k-v都是object利用序列化保存

   如何保存为json：

   1. 引入redis的starter，cacheManager变为RedisCacheManager；

   2. 默认创建的RedisCacheManager操作redis的时候默认使用的是RedisTemplate<Object, Object>

   3. RedisTemplate<Object, Object>默认使用jdk的序列化机制

   4. 自定义CacheManager

      ```java
       		@Bean
          public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
              //初始化一个RedisCacheWriter
              RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
              //设置CacheManager的值序列化方式为json序列化
              RedisSerializer<Object> jsonSerializer = new GenericJackson2JsonRedisSerializer();
              RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair
                      .fromSerializer(jsonSerializer);
              RedisCacheConfiguration defaultCacheConfig=RedisCacheConfiguration.defaultCacheConfig()
                      .serializeValuesWith(pair);
              //设置默认超过期时间是30秒
              defaultCacheConfig.entryTtl(Duration.ofSeconds(30));
              //初始化RedisCacheManager
              return new RedisCacheManager(redisCacheWriter, defaultCacheConfig);
          }
      ```

      



# 三.SpringBoot与消息

## 1.RabbitMQ核心概念：

- Message：消息，消息是不具名的，由**消息头**和**消息体**组成，消息体是不透明的，而消息头则是由一系列可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其它消息的优先权）、delivery-mode（指出该消息可能需要持久性存储等）。

- Publisher：消息生产者

- Consumer：消费者

- Exchange：交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

  Exchange有4种类型：direct（默认），fanout，topic和headers，不同类型的Exchange转发消息的策略有所区别

- Queue：消息队列

- Binding：绑定，用于消息队列和交换器直接的关联，一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

- Connection：网络连接，例如一个TCP连接

- Channel：信道，多路复用连接中的一条独立的双向数据流通道，信道是建立在TCP连接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息，订阅队列还是接收消息，都是通过信道完成的。

- Virtual Host：虚拟主机，每个vhost本质上是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。默认的vhost是/。

- Broker：消息队列服务实体

- ![rabbitmq](/Users/S/Desktop/LearningNotes/rabbitmq.png)



## 2.路由键：

- direct：点对点
- fanout：广播
- topic：路由键做模糊匹配

## 3.自动配置

1. RabbitAutoConfiguration
2. 有自动配置了连接工厂ConnectionFactory
3. RabbitProperties封装了RabbitMQ的配置
4. RabbitTemplate：给rabbitmq发送和接收消息
5. AmqpAdmin：rabbimq系统管理功能组件



```java
//Message需要自己构造一个，定义消息体内容和消息头
//rabbitTemplate.send(exchange,routeKey,message);

//object默认当成消息体，只需传入要发送的对象，自动序列化发送给rabbitmq
//rabbitTemplate.convertAndSend(exchange,routKey,object);
```

# 四.SpringBoot与检索

# 五.SpringBoot与任务

## 1.异步任务

启动类开启@EnableAsync

异步方法加上@Async

## 2.定时任务

@EnableScheduling、@Scheduled(corn = "")



| 序号 | 说明 | 必填 | 允许填写的值 | 允许的通配符 |
| ---- | ---- | ---- | ------------ | ------------ |
| 1    | 秒   | 是   | 0-59         | , - * /      |
| 2    | 分   | 是   | 0-59         | , - * /      |
| 3    | 时   | 是   | 0-23         | , - * /      |
| 4    | 日   | 是   | 1-31         | , - * ?/ L W |
| 5    | 月   | 是   | 1-12/JAN-DEC | , - * /      |
| 6    | 周   | 是   | 1-7/SUN-SAT  | , - * ?/ L#  |
| 7    | 年   | 否   | 1970-2099    | , - * /      |

## 3.邮件任务

```yml
mail:
  username: 932622760@qq.com
  password: ftgcttsazfugbdjb
  host: smtp.qq.com
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

```java
    @Autowired
    JavaMailSenderImpl mailSender;

@Test
void test03(){
    SimpleMailMessage mailMessage = new SimpleMailMessage();
    mailMessage.setSubject("通知-今晚上课");
    mailMessage.setText("今晚7:30上课");

    mailMessage.setTo("774378428@qq.com","547745464@qq.com","3112785317@qq.com");
    mailMessage.setFrom("932622760@qq.com");
    mailSender.send(mailMessage);
}

@Test
void test04()throws  Exception{
    MimeMessage mimeMessage = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);

    helper.setSubject("通知-今晚上课");
    helper.setText("<b style='color:red'>今晚7:30上课</b>",true);

    helper.setTo("3112785317@qq.com");
    helper.setFrom("932622760@qq.com");

    helper.addAttachment("1.jpeg",new File("/Users/S/Desktop/1.jpeg"));
    mailSender.send(mimeMessage);
}
```

# 六.SpringBoot与安全

# 七.SpringBoot与分布式

1. 引入依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.boot</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>0.1.0</version>
   </dependency>
   <dependency>
       <groupId>com.github.sgroschupf</groupId>
       <artifactId>zkclient</artifactId>
       <version>0.1</version>
   </dependency>
   ```

2. application.yml

   ​	

   生产者：

   ```yml
   dubbo:
     application:
       name: provider-ticket
   
     registry:
       address: zookeeper://47.102.149.102:2181
     scan:
       base-packages: com.cefer.ticket.service
   ```

   消费者：

   ```yml
   dubbo:
     application:
       name: consumer-user
   
     registry:
       address: zookeeper://47.102.149.102:2181
   ```

3. 生产者：

   ```java
   package com.cefer.ticket.service;
   
   import com.alibaba.dubbo.config.annotation.Service;
   import org.springframework.stereotype.Component;
   
   @Component
   @Service//dubbo的service将服务发不出去
   public class TicketServiceImpl implements TickerService{
   
       @Override
       public String getTicket() {
           return "《功夫》";
       }
   }
   ```

   消费者：

   ```java
   package com.cefer.user.service;
   
   import com.alibaba.dubbo.config.annotation.Reference;
   import com.cefer.ticket.service.TickerService;
   import org.springframework.stereotype.Service;
   
   @Service
   public class UserService {
   
       @Reference
       TickerService tickerService;
   
       public void hello(){
           String ticket = tickerService.getTicket();
           System.out.println("买到票了"+ticket);
       }
   }
   
   ```

   

   





# 八.SpringBoot与开发热部署

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

