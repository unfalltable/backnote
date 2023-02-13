# 短信登录模块

## Token实现

### 获取验证码

- 发送验证码发一个请求，登录再发一个请求
  - 俩次都需要校验手机号，防止获取验证码后手机号修改了

### 校验验证码

- 服务器将生成的验证码作为value存入redis
  - 因为redis属于共享的内存空间，如果都用一样的key会对应不上
  - 验证码用String存储，手机号为key
- 用户输入验证码，服务器通过用户的phone查询验证码是否和redis中的一致
- 验证通过
  - 然后拿着这个phone去数据库找是否有这个用户
    - 有就登录
    - 没有就注册，然后登录
  - 登录成功后服务器生成token作为key，value为user对象存入redis，设置token的有效期
    - 将token返回给客户端保存
    - 将user对象转化为map存入，需要考虑类型问题

### 设置拦截器拦截用户访问需要用户信息的页面

- 查询ThreadLocal中的user对象
  - 不存在则拦截
  - 存在则放行

### 设置拦截器防止用户过期重新登录

- 拦截一切路径
- 通过request获取请求头中的token
- 通过token去Redis查询对应的user，返回的是一个map集合
  - 判断user是否存在即判断map是否为空
    - 为空则放行
    - 不为空
      - 则将map集合转化为user对象（BeanUtils.fill）
      - 保存到ThreadLocal中，刷新有效期 expire()，放行

## 常见问题

### 集群Session共享问题

- 使用session保存收集好user对象，每一个session都有一个唯一的sessionId，在访问tomcat时sessionId会自动写入cookie中，session的原理是cookie
- 集群下不同的服务器并不共享session，导致请求切换到不同服务器时需要重新登录

# 二维码登录

# 第三方登录