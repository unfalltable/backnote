# 点赞模块

## 一人点一次

- 给Blog类中添加一个isLike字段，这个字段在数据库中不存在，标识当前用户是否点赞
  - 需要添加@TableField(exist = false)

- 利用redis的set集合实现
- 在首页的博客分页查询和根据id查询的时候判断当前用户是否点过赞，赋值给isLike字段

```java
public void likeBlog(Long id){
    //获取登录用户
    Long userId = UserHolder.getUser().getId();
    String key = "blog:liked:" + id;
    //判断当前登录用户是否点赞
    Double score 
        = stringRedisTemplate.opsForZSet.score(key, userId.toString());
    //如果未点赞，可以点赞
    if(score == null){
        //数据库点赞数+1
        boolean success = update().setSql("like = like + 1").eq("id", id).update();
    	//保存用户到redis集合中
        if(success){
            stringRedisTemplate.opsForZSet.add(key, userId.toString(), System.currentTimeMillis());
        }
    }else{
        //如果已点赞，取消点赞
        //数据库点赞数-1
        boolean success = update().setSql("like = like - 1").eq("id", id).update();
        //把用户从redis集合中删除
        if(success){
            stringRedisTemplate.opsForZSet.add(key, userId.toString());
        }
    } 	
}
```

## 点赞排行榜

- 先点赞的排在前面，需要修改之前校验一人一赞的数据类型，改为SortedSet
- 实现blogService.queryBlogLikes(Long id) 方法

```java
public List<UserDTO> queryBlogLikes(Long id){
    String key = "blog:liked:" + id;
    //查询点赞前5名的用户
    Set<String> top5 = stringRedisTemplate.opsForZSet.range(key, 0, 4);
    //如果为空返回空集合
    if(top5 == null || top5.isEmpty()) return Collections.emptyList();
    
    //解析出用户id
    List<Long> ids =  top5.stream().map(Long::valueOf).collect(Collections.toList());
    String idStr = StrUtil.join(",", ids);
    
    //根据id查询用户
    List<UserDTO> userDTO5 = userService.query()
        //为了使查询的数据按照我们想要的顺序
        .in("id", id).last("ORDER BY FIELD(id,"+idStr+")").list()
        .stream().map(user -> BeanUtil.copyProperties(user, UserDTO.calss))
        .collect(Collections.toList());
    return userDTO5;
}
```

# 关注模块

## 关注和取关

```java
public void follow(Long followUserId, Boolean isFollow){
    //获取登录用户
    Long userId = UserHolder.getUser().getId();
    
    //判断是关注还是取关
    if(isFollow){
        //设置关注信息，保存到数据库
    	Follow follow = new Follow();
    	follow.setUserId(userId);
        follow.setFollowUserId(followUserId);
        boolean isSuccess = save(follow);
        //为了后面的共同关注功能，需要将关注信息保存到redis中
        if(isSuccess){
            //存入userId 和 followUserId
            String key = "follows:" + userId;
            stringRedisTemplate.opsForSet().add(key, followUserId.toString());
        }
    }else{
        //取关
        boolean isSuccess = remove(new QueryWrapper<>()
               .eq("user_id", userId).eq("follow_user_id", followUserId));
        //删除redis中的关注信息
        if(isSuccess){
        	stringRedisTemplate.opsForSet().remove(key, followUserId.toString());
        }
    }
}
```

## 查看是否关注

```java
public boolean isFollow(Long followUserId){
    //获取登录用户
    Long userId = UserHolder.getUser().getId();
    //c数据库是否存在关联数据
    Integer count = query().eq("user_id", userId)
        .eq("follow_user_id", followUserId)).count();
    return count > 0;
}
```

## 共同关注

- 需要在用户关注的时候将用户的关注信息保存到redis中，用Set结构保存，可以查交集

```java
public List<UserDTO> followCommons(LOng id){
    //获取登录用户id
	Long id = UserHolder.getUser().getId();  
    //当前登录用户在redis中的key值
    String key1 = "follows:" + userId;
    //查看的用户在redis中的key值
    String key2 = "follows:" + id;
    //求共同关注即交集
    Set<String> intersect = stringRedisTemplate.opsForSet().intersect(key1,key2);
    //判断是否有交集,无交集返回空list
    if(intersect == null || intersect.isEmpty()) return Collections.emptyList();
    
    //解析id集合
    List<Long> ids =  intersect.stream()
        .map(Long::valueOf).collect(Collections.toList());
    
    //根据id查询用户
    List<UserDTO> users = userService.listByIds(ids)
        .stream().map(user -> BeanUtil.copyProperties(user, UserDTO.calss))
        .collect(Collections.toList());
    return users;
}
```

# 推送模块

## 推模式

- 给用户创建一个收件箱，用户发送笔记时推送到所有粉丝的收件箱，收件箱要满足可以根据时间戳排序，使用redis实现

- 用户查询收件箱时实现分页查询 （SortedSet）

- 在用户发布笔记时将博客id存入redis

- 推送笔记到粉丝邮箱

  - ```java
    //查询笔记作者的所有粉丝
        List<Follow> follows = followService.query().eq("follow_user_id",user.getId()).list();
    //推送笔记id给所有粉丝
        for(Follow follow : follows){
            //获取粉丝id
            Long userId = follow.getUserId();
            //推送
            String key = "feed:" + userId; 
            stringRedisTemplate.opsForZSet().add(key, blog.getId().toString(), System.currentTimeMillis());
        }
    ```

- 粉丝邮箱读取笔记

  - 需要先创建滚动分页的dto类

    - ```JAVA
      @Data
      public class ScoreResult{
          //查询的博客
          private List<?> list;
          //上一次查询的时间，用于下一次查询作为最小值
          private Long minTime;
          private Integer offset;
      }
      ```

  - 读取收件箱封装到ScoreResult中

    - ```java
      public ScoreResult queryBlogOfFollow(Long max, Integer offset){
          //获取当前用户
          Long userId = UserHolder.getUser().getId(); 
          //查询收件箱
          String key = "feed:" + userId;
          Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate
              //offset：跳过最大值的个数，0则包含
              .opsForZset().reverseRangeByScoreWithScores(key, 0, max, offset, 2);
          if(typedTuples == null || typedTuples.isEmpty()) return;
          
          //解析数据
          List<long> ids = new ArrayList<>(typedTuples.size());
          long minTime = 0;
          int os = 1;
          for(ZSetOperations.TypedTuple<String> tuple : typedTuples){
              //获取id
              ids.add(Long.valueOf(tuple.getValue()));
              //获取分数（时间戳）
              long time = tuple.getScore().longValue();
              if(time == minTime){
                  os++;
              }else{
                  minTime = time;
                  os = 1;
              }
          }
          //根据id查询blog
          String idStr = StrUtil.join(",", ids);
          List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id,"+idStr+")").list();
          
          for(Blog blog : blogs){
              //查询blog有关的用户
              queryBlogUser(blog);
              //查询Blog是否被点赞
              isBlogLiked(blog);
          }
          //返回
          ScrollResult r = new ScrollResult();
          r.setList(blogs);
          r.setOffset(os);
          r.setMinTime(minTime);
          return r;
      }
      ```

## 拉模式

## 推拉结合

# 签到模块

## 使用Reids的BitMap实现

- 签到
  - `stringRedisTemplate.opsForValue.setBit(key, dayOfMonth - 1, true)` 
- 统计
  - `List<Long> result = stringRedisTemplate.opsForValue().bitField(
        	key, BitFieldSubCommands.create()
            .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
            .valueAt(0)
        );`【怕
  - \

# 聊天模块

## Websocket + RocketMQ+MongoDB

处理消息（增删改查，创建对象）- 编写WebSocket（发送 / 接收消息）- 

### 处理消息

- `Message`类

  - ```java
    @Data
    @AllArgsConstruceor
    @NoArgsConstructor
    @Document(collection = "message") //指定
    @Builder //转变为建造者模式，链式构建对象
    public class Message{
        @Id 		//主键
        private ObjectId id;
        private String msg;
        @Indexed 	//需要建立索引
        private Integer status; 	// 1-未读  2-已读
        @Field("send_date")
        @Indexe		//需要建立索引
        private Date sendDate;
        @Field("read_date")
        private Date readDate;
        @Indexed 	//需要建立索引
        private User from;
        @Indexed 	//需要建立索引
        private User to;  
    }
    ```

- 创建`User`类

  - ```java
    @Data
    @AllArgsConstruceor
    @NoArgsConstructor
    @Builder //转变为建造者模式，链式调用
    public class User{
        private Long id;
        private String username;
    }
    ```

- 创建`MessageDAO` 接口，定义业务方法

  - ```java
    public interface MessageDAO{
        //查询点对点的聊天记录
        List<Message> findListByFromAndTo(Long fromId, Long toId, Integer page, Integer rows);
        //根据id查询数据
        Message findMessageById(String id);
        //更新消息状态
        UpdateResult updateMessageState(ObjectId id, Integer status);
        //新增消息，保存到mongodb
        Message saveMessage(Message message);
        //根据消息id删除数据
        DeleteResult deleteMessage(String id);
    }
    ```

- 创建`MessageDaoImpl` 实现类

  - ```java
    @Component
    public class MessageDaoImpl implements MessageDAO {
        @Autowired
        private MongoTemplate mongoTemplate;
        
        //查询点对点的聊天记录
        public List<Message> findListByFromAndTo(Long fromId, Long toId, Integer page, Integer rows){
            //设置查询条件
            //用户A发送给用户B的条件
            Criteria criteriaFrom = new Criteria().andOperator(
            	Criteria.where("from.id").id(fromId),.
                Criteria.where("to.id").id(toId)
            );
            //用户B发送给用户A的条件
            Criteria criteriaTo = new Criteria().andOperator(
            	Criteria.where("from.id").id(toId),
                Criteria.where("to.id").id(fromId)
            );
            //创建查询条件对象
            Criteria criteria = new Criteria().orOperator(criteriaFrom, criteriaTo);
            
            //设置分页
            PageRequest pageRequest = PageRequest.of(page - 1, pageSize, Sort.by(Sort.Direction.ASC, "send_data"));
            
            //设置查询条件， 分页
            Query query = Query.query(criteria).with(pageRequest);
            
            //需要查询语句和对象
            return this.mongoTemplate.find(query, Message.class);
        }
        //根据id查询数据
        public Message findMessageById(String id){
            return this.mongoTemplate.findById(new ObjectId(id), Message.class);
        }
        //更新消息状态
        public UpdateResult updateMessageState(ObjectId id, Integer status){
            Query query = Query.query(Criteria.where("id").is(id));
            Update update = Update.update("status", status);
            if(status.intValue() == 1) update.set("send_date", new Date());
            if(status.intValue() == 2) update.set("read_date", new Date());
            return this.mongoTemplate.updateFirst(query, update,  Message.class);
        }
        //新增消息，保存到mongodb
        public Message saveMessage(Message message){
            //需要写入发送时间，设置状态为1-未读
            message.setSendDate(new Date());
            message.setStatus(1);
            message.setId(ObjectId.get());
            return this.mongoTemplate.save(message);
        }
        //根据消息id删除数据
        public DeleteResult deleteMessage(String id){
            Query query = Query.query(Criteria.where("id").is(id));
            return this.mongoTemplate.remove(query, Message.class);
        }
    }
    ```

### 编写WebSocket

`MessageHandler`  发送 / 接收消息处理

- ```java
  @Component
  @RocketMQMessageListener(
      topic = "",				  //topic
      selectorExpression = "",   //tag
      messageModel =  ,		  //消息模式：广播 / 集群
      consumerGroup = "" 		  //消费者组
  )
  public class MessageHandler extends TextWebSocketHandler implements RocketMQListener<String>{
      //消息的增删改查
      @Autowired
      private MessageDAO messageDAO;
      @Autowired
      private RocketMQTemplate rocketMQTemplate;
      
      //格式转换
      private static final ObjectMapper MAPPER = new ObjectMapper();
      //收集用户session对象，用于判断用户是否在线
      private static final Map<Long, WebSocketSession> SESSION = new HashMap<>();
      
      //建立连接后要做的事情
      @Override
      public void afterConnectionEstablished(WebSocketSession session) throws Exection{
          //用户建立连接后将用户Session收集
          Long id = (Long) session.getAttributes().get("uid"); //需要设置拦截器获取
          SESSION.put(id, session);
      }
      
      //发送消息
      @Override
      protected void handleTextMessage(WebSocketSession session, TextMessage textMessage) throws Exception{
          Long uid = (Long) session.getAttributes().get("uid");
          JsonNode jsonNode = MAPPER.readTree(textMessage.getPayload());
          Long toId = jsonNode.get("toId").asLong();
          String msg = jsonNode.get("msg").asText();
          Message message = Message.builder()
              .from(UserData.USER_MAP.get(uid))
              .to(UserData.USER_MAP.get(toId))
              .msg(msg)
              .build();
          //将消息保存到mongodb
          message = this.messageDAO.saveMessage(message);
          
          //判断目标用户是否在线
          WebSocketSession toSession = SESSIONS.get(toId);
          if(toId != null && toSession.isOpen()){
              toSession.sendMessage(new TextMessage(MAPPER.writeValueAsString(message)));
              //将消息更新为已读
              this.messageDAO.updateMessageState(message.getId(), 2);
          }
          //分布式WebSocket
          else{
              //用户可能在其他节点中，先将消息发送到MQ中
              //需要添加tag便于消费者筛选
              this.rocketMQTemplate.convertAndSend(topic:tag, 序列化后的消息);
          }
      }
      //消费消息
      @Override
      public void onMessage(String msg) throws Exception{
          JsonNode jsonNode = MAPPER.readTree(msg);
          long toId = jsonNode.get("to").get("id").longValue();
          //判断to用户是否在线
          WebSocketSession toSession = SESSION.get(toId);
          if(toId != null && toSession.isOpen()){
              toSession.sendMessage(new TextMessage(msg));
              this.messageDAO.updateMessageState(new ObjectId(jsonNode.get("id").asText), 2);
          }
      }
  }
  ```

`MessageHandshakeInterceptor` 拦截器，用于获取用户id

- ```java
  @Component
  public class MessageHandshakeInterceptor implements HandshakeInterceptor{
      //在客户端与服务端建立连接之前执行，握手之前
      @Override
      public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception{
          String path = request.getURI().getPath();
          String[] ss = StringUtils.split(path, '/');
          if(ss.length != 2) return false;
          if(!StringUtils.isNumeric(ss[1])) return false;
          attributes.put("uid", Long.valueOf(ss[1]));
          return true;
      }
  }
  ```

`WebSOcketConfig` 添加拦截器

- ```java
  @Configuration
  @EnableWebSocket
  public class WebSocketConfig implements WebSocketConfigurer{
      @Autowired
      private MessageHandler messageHandler;
      @Autowired
      private MessageHandshakeInterceptor messageHandshakeInterceptor;
      
      @Override
      public void registerWebSocketHandlers(WebSocketHandlerRegistry registry){
          registry.addHandler(this.messageHandler, "/ws/{uid}")
              .setAllowedOrigins("*") //设置跨域请求
              .addInterceptors(this.messageHandshakeInterceptor);
      }
  }
  ```

### 查询历史信息，其实就是查询消息列表

- 创建`MessageController`

  - ```java
    @RequestMapping("message")
    @CrossOrigin //跨域
    public class MessageController {
        @autowired
        private MessageService messageService;
        
    	@GetMapping
        public List<Message> queryMessageList(
        	@RequestParam("fromId") Long fromId,
        	@RequestParam("toId") Long toId,
       		@RequestParam(value = "page", default = "1") Integer page,
        	@RequestParam(value = "rows", default = "10" Integer rows){
    	return this.messageService.queryMessageList(fromId, toId, page, rows);
    }
    ```

- 创建`MessageService` 

  - ```java
    @Service
    public class MessageService{
        @Autowired
        private MessageDAO messageDAO;
        
        public List<Message> queryMessageList(Long fromId, Long toId, Integer page, Integer rows){
            List<Message> list = this.messageDAO.findListByFromAndTo(fromId, toId, page, rows);
            //设置消息状态为已读
            for(Message message : list){
                if(message.getStatus().intValue() == 1){
                    this.messageDAO.updateMessageState(message.getId(), 2);
                }
            }
            return list;
        }
    }
    ```

- 创建`UserController`

  - ```java
    @RestController
    @CrossOrigin
    @RequestMapping("user")
    public class UserController{
        @Autowired
        private MessageService messageService;
        
        @GetMapping
        public List<Map<String, Object>> queryUserList(@RequestParam("fromId") Long fromId){
            List<Map<String, Object>> result = new ArrayList<>();
            for(Map.Entry<Long, User> userEntry : UserData.USER_MAP.entrySet()){
                Map<String,Object> map = new HashMap<>()
                //获得对应的值
                result.add(map);
            }
            return result;
        }
    }
    ```

## 
