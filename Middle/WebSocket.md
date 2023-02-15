---
title: WebSocket
toc: content
keywords: [middle]
---

## 是什么

- 基于TCP的全双工通信，双向传输

## SpringBoot整合WebSocket

- 发送消息
  - `session.sendMessage(TextMessage对象)`

## WebSocket拦截器

- 可以在建立连接之前写一些业务逻辑

  - ```java
    public class MyHandshakeInterceptor implements HandshakeInterceptor{
        
        @Override
        public boolean beforeHandshake(ServerHttpRequest request, ServerHttpReponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception{
            //将用户id放入socket处理器的会话中
            attributes.put("uid", 1001);
            return true;
        }
        @Override
        public void afterHandshake(ServerHttpRequest request, ServerHttpReponse response, WebSocketHandler wsHandler, Exception exception){
            //握手成功后的逻辑
        }
    }
    ```
    


- 编写WebSocketConfig，添加拦截器后才能生效

  - ```java
    public class WebSocketConfig implements WebSocketConfigurer{
        @Autowired
        private MyHandshakeInterceptor myHandshakeInterceptor;
        
        @Override
        public void registerWebSocketHandlers(WebSocketHandlerRegistry registry){
            registry.addHandler()
                .addInterceptors(myHandshakeInterceptor);
        }
    }
    ```

## 分布式WebSocket

- 生产者
  - 当发现发送信息的对象toid为空或者不在线时，可能在其他节点中
    - 通过rocketMQ去查询其他socket服务端是否存在该客户端的session
  - 注入Rocket'MQTemplate，使用convertAndSend(topic:tags, 序列化的message)发送到MQ消息系统
- 消费者
  - 需要实现RocketMQListener<String>接口
  - 添加@RocketMqMessageListener(topic = “topic”, selectorExpression = “tag”, messageModel =  , consumerGroup = “”)
  - 重写 onMessage(String msg)
