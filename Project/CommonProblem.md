## redis集群ip问题

- docker生成的主机ip地址，无法搭建集群
- 容器的ip地址，可以搭建集群，但是集群在客户端不可用，服务端可用
- 主机的ip地址，无法搭建集群
- 解决方案：

  - 创建容器的时候指定网络类型为host

  - 弊端：不安全，容器ip地址暴露

## Request流只能读取一次

- 包装Request解决
- 通过过滤器进行包装request对象
  - 用instanceof 判断是否包装过

- 创建`MyServletRequestWrapper` 继承 `HttpServletRequestWrapper` 
  - ![image-20220722155426368](../../../../../pic/image-20220722155426368.png)
  - ![image-20220722155549300](../项目/功能模块/模块/image-20220722155549300.png)
  - ![image-20220722180045329](../项目/功能模块/模块/image-20220722180045329.png)
- 通过过滤器包装request
  - ![image-20220722180138480](../项目/功能模块/模块/image-20220722180138480.png)

## Session共享问题

- 分布式WebSocket解决

## Netty冲突

- Redis中使用到了netty，elasticSearch中也引入了netty，发生冲突
- 需要在启动类中添加
  - `System.setProperty("es.set.netty.runtime.available.processors", "false")`

## 解决表名大小写敏感问题

https://www.cnblogs.com/ljincheng/p/13581326.html

只有在初始化的时候才可以设置

## 终端不能输入中文

`vim /etc/profile` 

输入 `export LC_CTYPE='zh_CN.UTF-8' ` 

## 跨域问题

- @CrossOrigin

- ```java
  //支持跨域
  response.setHeader("Access-Control-Allow-Origin", "*");
  response.setHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS");
  response.setHeader("Access-Control-Allow-Credentials", "true");
  response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Token");
  response.setHeader("Access-Control-Allow-Credentials", "true");
  ```

## 非法访问异常

- Jdk > 1.8   虚拟机加参数
  - `--add-opens java.base/java.lang=ALL-UNNAMED`

## spring-security获取不到Authentication

- 异常信息
  -  **For input string: "ANONYMOUS"** 的错误

- 可能原因
  - 没有获取到登录的用户导致的获取不到用户信息
- 解决方法
  - 修改鉴权配置
  - 排查代码问题

## There is no PasswordEncoder mapped for the id "null" 

- 这个错主要发生在Spring-Sercurity5.X版本上，例如SpringBoot2.x。导致这个错误发生主要原因就是在之前版本中的**NoOpPasswordEncoder**被**DelegatingPasswordEncoder**取代了，而你保存在数据库中的密码没有没有指定加密方式。
- 使用DelegatingPasswordEncoder解决

## 数据库字段采用一个字母加下划线开头

- 数据库字段例如：c_type
- 然后pojo类如果是：cType
- 会导致前端传参的时候取不到值
- 解决方法：
  - pojo改为 ctype 然后用@TableField("c_type")，前端传 ctype

## 写XML的SQL如果有del_flag要记得写上

## 注入Service失败

- 方法一：使用静态变量 加 @PostConstruct 解决。

  ```java
  @Component //关键1
  public class ArticlesReceiver {
  
      @Resource
      private  WechatArticlesTempService wechatArticlesTempService;
       
      public static ArticlesReceiver articlesReceiver; //关键2
       
      @PostConstruct //关键3
      public void init(){
      　　　　articlesReceiver = this;
      }
           
          public WechatArticlesTemp getResposeArticlesBoby(String mediaId) {
  　　　　　　　　　　WechatArticlesTemp articlesTemp = articlesReceiver.wechatArticlesTempService.getById(mediaId); //关键4
  　　　　　　　　　　return articlesTemp ;
      }
  }   
  ```

- 方法二：使用静态变量，加set注入　

  ```java
  @Component //关键1
  public class ArticlesReceiver {
   
      private static WechatArticlesTempService wechatArticlesTempService; //关键2
   
      @Autowired  //关键3
          public void setWechatArticlesTempService (WechatArticlesTempService wechatArticlesTempService){
                ArticlesReceiver.wechatArticlesTempService = wechatArticlesTempService;
          }
           
          public WechatArticlesTemp getResposeArticlesBoby(String mediaId) {
  　　　　　　　　　　WechatArticlesTemp articlesTemp = wechatArticlesTempService.getById(mediaId); //关键4
  　　　　　　　　　　return articlesTemp ;
      }
  }
  ```

- 方法三：代码注入 ， SpringContectHolder类将用到的类的class读入让后再调用类中方法

  ```java
  @Component //关键1
  public class ArticlesReceiver {
   
      private static WechatArticlesTempService wechatArticlesTempService =  SpringContextHolder.getBean(WechatArticlesTempService.class); //关键2
   
          public WechatArticlesTemp getResposeArticlesBoby(String mediaId) {
  　　　　　　　　　　WechatArticlesTemp articlesTemp = wechatArticlesTempService.getById(mediaId); //关键3
  　　　　　　　　　　return articlesTemp ;
      }
  }
  ```

##  导入的接口如果添加了日志注解可能会报错，但是接口能正常通过

- 解决方法
  - 删除日志注解