# 聊天模块

## WebSocket + MongoDB + RocketMQ

1. 处理消息（创建消息对象，实现消息的基本功能）

   - 创建消息类并创建MongoDB中的集合（表）

     - @Indexed设置索引
     - 类中需要有User From和User to属性

   - 实现消息的增删改查

     - 查询点对点的聊天记录

       - ```java
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
             //设置分页，按发送时间降序
             PageRequest pageRequest = PageRequest.of(page - 1, pageSize, Sort.by(Sort.Direction.ASC, "send_data"));
             //设置查询条件，分页
             Query query = Query.query(criteria).with(pageRequest);
             //需要查询语句和对象
             return this.mongoTemplate.find(query, Message.class);
         }
         ```

2. 编写WebSocket

   - ```java
     @Component
     public class MessageHandler extends TextWebSocketHandler implements RocketMQListener<String>{
         @Override
         public void afterConnectionEstablished(WebSocketSession session) throws Exection{
             //建立连接后要做的事情
         }
         //接收消息
         @Override
         protected void handleTextMessage(WebSocketSession session, TextMessage textMessage) throws Exception{
             //将消息保存到mongodb
             message = this.messageDAO.saveMessage(message);
             //发送消息
             webSocketSession.sendMessage(new TextMessage(MAPPER.writeValueAsString(message)));
             //用户可能在其他节点中，先将消息发送到MQ中
             //需要添加tag便于消费者筛选
             this.rocketMQTemplate.convertAndSend(topic:tag, 序列化后的消息);
         }
     }
     ```

3. 设置WebSocket拦截器

   - ```java
     @Component
     public class MessageHandshakeInterceptor implements HandshakeInterceptor, WebSocketConfigurer{
         @Override
         public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception{
             //在客户端与服务端建立连接之前执行，握手之前进行一些处理
         }
         
         @Override
         public void registerWebSocketHandlers(WebSocketHandlerRegistry registry){
             registry.addHandler(this.messageHandler, "/ws/{uid}")
                 .setAllowedOrigins("*") //设置跨域请求
                 .addInterceptors(this);
         }
     }
     ```

4. 分布式WebSocket

   - ```java
     @Component
     @RocketMQMessageListener(
         topic = "",				  //topic
         selectorExpression = "",   //tag
         messageModel =  ,		  //消息模式：广播 / 集群
         consumerGroup = "" 		  //消费者组
     )
     public class MessageHandler implements RocketMQListener<String>{
         //接收到MQ消息
         @Override
         public void onMessage(String msg) throws Exception{
     		//找到该机器下的用户并给他发送消息
         }
     }
     ```

# 