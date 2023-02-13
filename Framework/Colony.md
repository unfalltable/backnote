---
title: 集群搭建
keywords: [colony]
---

## Mysql集群

## Redis集群

1. 使用Docker搭建redis集群，使用host网络类型，配置

   - 拉取镜像-创建容器-启动容器指定host网络类型

2. 创建`ClusterConfigurationProperties` 

   - ```java
     @Component
     @ConfigurationProperties(prefix="spring.redis.cluster")
     @Data
     public class ClusterConfigurationProperties{
         //存放所有节点的ip地址
         private List<String> nodes;
         //最大重定向次数
         private Integer maxRedirects;
     }
     ```


3. 建立连接

   - ```java
     @Configuration
     public class RedisClusterConfig{
         @Autowired
         private ClusterConfigurationProperties clusterProperties;
         
         @Bean
         public RedisConnectionFactory connectionFactory(){
             RedisClusterConfiguration configuration = new RedisClusterConfiguration(clusterProperties.getMaxRedirects());
             configuration.setMaxRedirects(clusterProperties.getMaxRedirects());
             return new JedisConnectionFactory(configuration);
         }
         @Bean
         public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionfactory){
             RedisTemplate<String, String> redisTemplate = new redisTemplate<>();
             redisTemplate.setConnectionFactory(redisConnectionfactory);
             redisTemplate.setKeySerializer(new StringRedisSerializer());
             redisTemplate.setValueSerializer(new StringRedisSerializer());
             redisTemplate.afterPropertiesSet();
             return redisTemplate;
         }
     } 
     ```


4. 采用统一控制缓存逻辑，使用拦截器实现

   - ```java
     public class Interceptor(){
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){
             if(StringUtils.equalsIgnoreCase(request.getMethod(), "OPTIONS")){
                 return true;
             }
             //判断请求方式
             if(!StringUtils.equalsIgnoreCase(request.getMethod(), "GET")){
                 //非Get，如果不是graphql，则放行
                 if(!StringUtils.equalsIgnoreCase(request.getRequestURI(), "/graphql")){
                     return true;
                 }
             }
             //通过缓存做命中，查询redis，redisKey ? 组成md5
             String redisKey = createRedisKey(request);
             String data = this.redisTemplate.opsForValue().get(redisKey);
             if(StringUtils.isEmpty(data)) return true;
             //支持跨域
             response.setHeader("Access-Control-Allow-Origin", "*");
             response.setHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS");
             response.setHeader("Access-Control-Allow-Credentials", "true");
             response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Token");
             response.setHeader("Access-Control-Allow-Credentials", "true");
             response.getWriter().write(data);
             return false;
         }
         public static String createRedisKey(HttpServletRequest request) throws Exception{
             String paramStr = request.getRequestURI();
             Map<String, String[]> parameterMap = request.getParameterMap();
             if(parameterMap.isEmpty()){
                 paramStr += IOUtils.toString(request.getInputStream(), "UTF-8");
             }else{
                 paramStr += mapper.writeValueAsString(request.getParameterMap());
             }
          }
     }
     ```

  - ![image-20220722032058524](..//pic//image-20220722032058524.png)

5. 将拦截器注入Spring容器

   - ![image-20220722031848914](..//pic//image-20220722031848914.png)
   - 将数据库查询结果写入Redis缓存中
     - 使用**ResponseBodyAdvice** 在响应结果被处理前拦截，拦截的逻辑自己实现，这样就可以将响应结果写入缓存中了
       - ![image-20220722033053796](..//pic//image-20220722033053796.png)
       - ![image-20220722033132817](..//pic//image-20220722033132817.png)
         - @ControllerAdvice进行拦截
         - supprot方法返回true才会执行beforeBodyWrite方法

## ES集群

### 节点

- master节点
  - 配置文件中node.master设置为true，则有资格被选为master节点
  - 创建/删除索引，管理非master
- data节点
  - 配置文件中node.data设置为true，就有资格
  - CRUD
- 客户端节点
  - node.master和node.data都为false
  - 响应客户的请求，转化请求
- 部落节点
  - 配置tribe.*，则是一个部落节点
  - 可以连接多个集群，在所有集群上进行搜索等操作

### 步骤

- 创建es-cluster文件夹，在下面创建节点目录
- 需要将es下的`elasticsearch.yml`和`jvm.options`拷贝到节点目录下
- 编辑`elasticsearch.yml`，添加节点名字
  - `cluster.name: 集群名字`
  - `node.name: 节点名字`
  - `http.port: 端口号`
  - `discovery.zen.ping.unicast.hosts: ["广播地址"]`
  - `discovery.zen.minimum_master_nodes: 最小master节点数`
  - `node.master: 是否为master节点`
  - `node.data: 是否为data节点`
- 编辑`jvm.options` ，修改jvm最大和最小堆内存
- docker创建容器进行配置文件挂载

### 分片和副本

- 一个分片是一个最小级别的工作单元，他只是保存了索引中所有数据的一部分
- 我们需要知道分片就是一个Lucene实例，并且它本身就是一个完整的搜索引擎，应用程序不会和它直接通信
- 分片可以是主分片或者是复制分片
- 索引中的每个文档属于一个单独的主分片，所以主分片的数量决定了所应最多能存储多少数据
- 复制分片只是主分片的一个副本，它可以防止硬件故障导致的数据丢失，同时可以提供读请求，比如搜索或者从别的分片取回文档
- 当索引创建完成的时候，主分片的数量就固定了，但是复制分片的数量可以随时调整

### 脑裂

- master发生宕机，然后集群重新选举master，宕机的master恢复后，集群中出现了俩个master，会分成俩个集群
- 设置选举master数为：（N/2）+ 1 可解决

## RocketMQ集群

## MongoDB集群

