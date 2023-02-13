## 	短信验证

### 获取验证码和校验验证码

- 服务器将生成的验证码作为value存入redis，手机号作为key来区分每一位用户
  - 因为redis属于共享的内存空间，如果都用一样的key会对应不上

- 用户输入验证码，服务器通过用户的phone查询验证码是否和redis中的一致
- 验证通过
  - 然后拿着这个phone去数据库找是否有这个用户
    - 有就登录
    - 没有就注册，然后登录
  - 登录成功后生成token作为key，value为user对象存入redis，设置token的有效期
    - 将token返回给客户端保存
    - 将user对象转化为map存入，需要考虑类型问题

```java
//发送验证码
public Result sendCode(String phone, HttpSession session) {
    // 1.校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2.如果不符合，返回错误信息
        return Result.fail("手机号格式错误！");
    }
    // 3.符合，生成验证码
    String code = RandomUtil.randomNumbers(6);

    // 4.保存验证码到 session
    stringRedisTemplate.opsForValue().set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);

    // 5.发送验证码
    log.debug("发送短信验证码成功，验证码：{}", code);
    // 返回ok
    return Result.ok();
}

//校验验证码
public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 1.校验手机号
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2.如果不符合，返回错误信息
        return Result.fail("手机号格式错误！");
    }
    // 3.从redis获取验证码并校验
    String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
    String code = loginForm.getCode();
    if (cacheCode == null || !cacheCode.equals(code)) {
        // 不一致，报错
        return Result.fail("验证码错误");
    }

    // 4.一致，根据手机号查询用户 select * from tb_user where phone = ?
    User user = query().eq("phone", phone).one();

    // 5.判断用户是否存在
    if (user == null) {
        // 6.不存在，创建新用户并保存
        user = createUserWithPhone(phone);
    }
    // 7.保存用户信息到 redis中
    // 7.1.使用JWT生成token，作为登录令牌
    Map<String,String> map = new HashMap<>();
    map.put("userId",user.getId().toString());
    map.put("phone",phone);
    String token = JWTUtils.getToken(map);
    // 7.2.将User对象转为HashMap存储
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
    Map<String, Object> userMap 
        = BeanUtil.beanToMap(userDTO, new HashMap<>(),
                             CopyOptions.create()                                                 .setIgnoreNullValue(true)
						  .setFieldValueEditor(
                   (fieldName, fieldValue) -> fieldValue.toString()));
    // 7.3.存储
    String tokenKey = LOGIN_USER_KEY + token;
    stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
    // 7.4.设置token有效期
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);

    /*// 7.保存用户信息到 redis中
        // 7.1.随机生成token，作为登录令牌
        String token = UUID.randomUUID().toString(true);
        // 7.2.将User对象转为HashMap存储
        UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
        Map<String, Object> userMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),
                CopyOptions.create()
                        .setIgnoreNullValue(true)
                        .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString()));
        // 7.3.存储
        String tokenKey = LOGIN_USER_KEY + token;
        stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
        // 7.4.设置token有效期
        stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);*/
    
    // 8.返回token
    return Result.ok(token);
}
```

### 设置拦截器防止用户过期重新登录

- 通过request获取请求头中的token
  - 不存在就拦截
- 通过token去数据库查询对应的user，返回的是一个map集合
- 判断user是否存在即判断map是否为空
  - 为空则拦截
- 将map集合转化为user对象（BeanUtils.fill）
- 通过ThreadLocal将user对象和当前用户线程绑定
- 刷新token的有效期

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    // 1.获取请求头中的token
    String token = request.getHeader("authorization");
    if (StrUtil.isBlank(token)) {
        return true;
    }
    // 2.基于TOKEN获取redis中的用户
    String key  = LOGIN_USER_KEY + token;
    Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(key);
    // 3.判断用户是否存在
    if (userMap.isEmpty()) {
        return true;
    }
    // 5.将查询到的hash数据转为UserDTO
    UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
    // 6.存在，保存用户信息到 ThreadLocal
    UserHolder.saveUser(userDTO);
    // 7.刷新token有效期
    stringRedisTemplate.expire(key, LOGIN_USER_TTL, TimeUnit.MINUTES);
    // 8.放行
    return true;
}
```

### 使用JWT生成token

- 封装jwt工具类
- 使用JWT验证就可以不用写入redis中了，可以直接进行验证，但就需要用别的方式保留用户信息了

### 阿里云短信服务

- ![image-20220711233804141](黑马点评项目/image-20220711233804141.png)
- ![image-20220711235022243](黑马点评项目/image-20220711235022243.png)

## 用户缓存

### 添加用户缓存

- 查询数据时先去Redis缓存中找，有就返回，没有再到数据库找，减少数据库压力
- 到数据库找后将结果写入缓存，然后返回给客户端

```java
public Shop queryById(Long id) {
    String key = CACHE_SHOP_KEY + id;
    //从redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    //判断是否存在
	if(shopJson...)
    	//存在直接返回
        return Shop shop = JSONUtils.toBean(shopJson,Shop.class);
    //不存在，根据id查询数据库
    Shop shop = getById(id);
    //判断shop是否存在
    if(shop...)
    	//不存在返回错误
    	return false;
    //存在写入redis
    stringRedisTemplate.
       opsForValue().set(key,JSONUtils.toJsonStr(shop),过期时间，时间单位);
    //返回
    return shop;
}
```

#### 主动更新策略（即在更新数据库时更新缓存）

- 选择删除缓存，等待有查询时在更新缓存，延迟加载
  - 如果每次更新数据库都更新缓存，无效的写操作太多
- 选择先更新数据库再更新缓存
  - 如果先删除缓存再更新数据库，当线程1删除缓存后正在更新数据库时，线程2去查询缓存未命中，然后线程2去查数据库，会将数据库旧数据写入缓存中

```java
public void update(Shop shop){
    //更新数据库
    updateById(shop);
    //删除缓存
    stringRedisTemplate.delete(CACHE_SHOP_KEY + id);
}
```

### 缓存预热

- 主从之间数据吞吐量较大、数据同步操作频度高、请求数量较高，而且redis中没有数据，导致redis启动时快速宕机

#### 解决方法

- 日常需要对数据的访问进行记录，统计热点数据
- 利用LRU（）数据删除策略，构建数据留存队列
- redis启动时加载热点数据
  - 使用脚本固定触发数据预热


### 缓存穿透

- 缓存穿透是指客户端请求的数据在缓存和数据库中的不存在，这些请求都会打到数据库

#### 解决办法

- 使用缓存空对象解决，可能存在短期不一致性
  - 实现简单，维护方便，但会带来额外的内存消耗，可能造成短期的不一致

- 使用布隆过滤器解决，在客户和缓存之间添加
  - 内存占用少，实现复杂，存在误判
- 增强id的复杂度
- 加强用户权限校验
- 做好热点参数的限流

```java
/*
*   使用缓存空对象解决缓存穿透
* */
public <R,ID> R queryWithPassThrough
       (String keyPrefix, 
        ID id, 
        Class<R> type,
        Function<ID, R> dbFallback,
        Long time, 
        TimeUnit unit ){
    
    String key = keyPrefix + id;
    // 1.从redis查询商铺缓存
    String json = stringRedisTemplate.opsForValue().get(key);
    // 2.判断是否存在
    if (StrUtil.isNotBlank(json)) {
        // 3.存在，直接返回
        return JSONUtil.toBean(json, type);
    }
    // 判断命中的是否是空值 “”
    if (json != null) {
        // 返回一个错误信息
        return null;
    }
    // 4.不存在，根据id查询数据库
    R r = dbFallback.apply(id);
    // 5.不存在，返回错误
    if (r == null) {
        // 将空值写入redis
        stringRedisTemplate
            .opsForValue()
            .set(key, "", 过期时间, 时间单位);
        // 返回错误信息
        return null;
    }
    // 6.存在，写入redis
    this.set(key, r, time, unit);
    return r;
}
```

### 缓存雪崩

- 在较短时段内大量的缓存失效或者redis宕机，导致大量的请求打到数据库

#### 解决方法

- 给不同的缓存key的有效期加上随机的时间
  - 不同业务过期时间不同
  - 对超级热点数据使用永久key

- 利用redis集群
- 给缓存业务添加降级限流策略
- 给业务添加**多级缓存** 
- 优化数据库中严重耗时的业务
- redis监控服务器性能指标
- 把缓存key到期删除策略改成按命中次数删除策略

### 缓存击穿

- 一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求会打到数据库

#### 解决方法

##### 互斥锁

- 没有额外内存消耗，保证一致性，实现简单，但是性能受影响，可能出现死锁
- 利用Redis的 SETNX 实现
  - 只有第一个执行SETNX的成功	

```java
/*
*   互斥锁解决缓存击穿
* */
public <R, ID> R queryWithMutex(
    String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
    
    String key = keyPrefix + id;
    // 1.从redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    // 2.判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {
        // 3.存在，直接返回
        return JSONUtil.toBean(shopJson, type);
    }
    // 判断命中的是否是空值
    if (shopJson != null) {
        // 返回一个错误信息
        return null;
    }
    // 4.实现缓存重建
    String lockKey = LOCK_SHOP_KEY + id;
    R r = null;
    try {
        // 4.1.获取互斥锁
        boolean isLock = tryLock(lockKey);
        // 4.2.判断是否获取成功
        if (!isLock) {
            // 4.3.获取锁失败，休眠并重试
            Thread.sleep(50);
            //重新尝试获取锁
            return queryWithMutex(keyPrefix, id, type, dbFallback, 										time, unit);
        }
        // 4.4.获取锁成功，根据id查询数据库
        r = dbFallback.apply(id);
        // 5.不存在，返回错误
        if (r == null) {
            // 将空值写入redis
            stringRedisTemplate
                .opsForValue()
                .set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            // 返回错误信息
            return null;
        }
        // 6.存在，写入redis
        this.stringRedisTemplate.opsForValue.set(key, r, time, unit);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }finally {
        // 7.释放锁
        unlock(lockKey);
    }
    // 8.返回
    return r;
}
```

##### 逻辑过期

- 性能较好，但不保证一致性，有额外内存消耗，实现复杂

```java
//开启线程池
private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

//逻辑过期解决缓存击穿
public <R, ID> R queryWithLogicalExpire(
    String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
    
    String key = keyPrefix + id;
    // 1.从redis查询商铺缓存
    String json = stringRedisTemplate.opsForValue().get(key);
    // 2.判断是否存在
    if (StrUtil.isBlank(json)) {
        // 3.未命中，直接返回
        return null;
    }
    // 4.命中，需要先把json反序列化为对象
    RedisData redisData = JSONUtil.toBean(json, RedisData.class);
    R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
    //获取缓存过期时间
    LocalDateTime expireTime = redisData.getExpireTime();
    // 5.判断是否过期
    if(expireTime.isAfter(LocalDateTime.now())) {
        // 5.1.未过期，直接返回店铺信息
        return r;
    }
    // 5.2.已过期，需要缓存重建
    // 6.缓存重建
    // 6.1.获取互斥锁
    String lockKey = LOCK_SHOP_KEY + id;
    boolean isLock = tryLock(lockKey);
    // 6.2.判断是否获取锁成功
    if (isLock){
        // 6.3.成功，开启独立线程，实现缓存重建
        CACHE_REBUILD_EXECUTOR.submit(() -> {
            try {
                // 查询数据库
                R newR = dbFallback.apply(id);
                // 重建缓存
                this.setWithLogicalExpire(key, newR);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }finally {
                // 释放锁
                unlock(lockKey);
            }
        });
    }
    // 6.4.返回过期的商铺信息
    return r;
}
//缓存重建
public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit){
    RedisData redisdata = new RedisData();
    redisdata.setData(value);
    //设置逻辑过期时间
    redisdata
        .setExpireTime(LocalDateTime.now()
        .plusSeconds(unit.toSeconds(time)));
    stringRedisTemplate
        .opsForValue()
        .set(key, JSONUtils.toJsonStr(redisdata))
}
//RedisData
class RedisData{
    private LocalDateTime time;
    private Object data;
}
```

## 优惠券秒杀

### 全局ID生成器

- 需要满足唯一性、递增性、安全性、高可用、高性能

#### Redis自增实现

- 避免id规律明显和分布式系统下id冲突问题

![image-20220330182543875](C:\Users\15630\AppData\Roaming\Typora\typora-user-images\image-20220330182543875.png)

```java
//全局ID生成器
class RedisIdWorker{
    //开始时间戳(2022年1月1日0时0分0秒)
	private static final long BEGIN_TIMESTAMP = 1640995200L;
	public static long nextId(String keyPrefix){
        //生成时间戳
        LocalDateTime now = LocalDateTime.now();
        Long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        Long timeStamp = nowSecond - BEGIN_TIMESTAMP;
        //生成序列号
        String date = 
            now.format(DateTimeFormatter.ofPattern("YY:MM:dd"));
        Long count = 
            stringRedisTemplate
                .opsForValue()
                .increment("icr:" + keyPrefix + ":" + date);
        //拼接并返回
    	return timeStamp << 32 | count;
	}
}
```

#### 雪花算法

![image-20220330185106819](C:\Users\15630\AppData\Roaming\Typora\typora-user-images\image-20220330185106819.png)

- 同一毫秒生成的id不超过4095，超过4095则到下一毫秒生成
- 由于每一台机器的工作机器id不同，所以分布式环境下也不会冲突

### 秒杀实现

```java
//秒杀券
private ISeckillVoucherService seckillVoucherService;
//优惠券秒杀，需要返回订单id
private Long seckillVoucher(Long voucherId){
    //查询优惠券
    seckillVoucher voucher = seckillVoucherService.getById(voucherId);
    //判断秒杀是否开始
    if(voucher.getBeginTime().isAfter(LocalDateTime.now())){
        //秒杀未开始
    }
    //判断秒杀是否已经结束
    if(voucher.getEndTime().isBefore(LocalDateTime.now())){
        //秒杀已结束
    }
    //判断库存是否充足
    if(voucher.getStock < 1){
        //库存不足
    }
    //扣减库存（存在超卖问题）
    boolean success = seckillVoucherService.update()
        .setsql("stock = stock - 1")
        .eq("voucher_id", voucherId).update();
    //判断是否扣减成功
    if(!success){
        //扣减失败，库存不足
    }
    //创建订单
    VoucherOrder voucherOrder = new VoucherOrder();
    //设置订单id
    int orderId = RedisIdWorker.nextId("order");
    voucherOrder.setId(orderId);
    //设置用户id
    //获取threadLocal中绑定的用户的id
    Long userId = UserHolder.getUser().getId();
    voucherOrder.setUserId(userId);
    //设置秒杀券id
    voucherOrder.setVoucherId(voucherId);
    //返回订单id
    return orderId;
}
```

### 库存超卖

- 使用CAS解决，不用加锁

```java
//扣减库存
boolean success = seckillVoucherService.update()
    .setsql("stock = stock - 1")
    .eq("voucher_id", voucherId)
    //扣减库存时再查询一次库存
    .gt("stock",0).update();
```

- 可以使用Synchronize锁解决

### 一人一单

- 需要在扣减库存前进行判断，并且把判断过程和扣减库存用synchronize锁住
- 以userId作为锁
- 在分布式系统下无法保证，需要使用分布式锁

```java
private Long seckillVoucher(Long voucherId){
    //一系列的判断
    //一人一单判断
    Long userId = UserHolder.getUser().getId();
    synchronized(userId.toString().intern()){
        //查询用户是否下过单
        int count = query()
            .eq("user_id", userId)
            .eq("voucher_id", voucherId).count();
        //判断
        if(count > 0){
            //已经下过单，返回
        }
        //没下过单
        //扣减库存
    }

```

### 分布式锁

- 分布式锁要满足多进程可见、互斥、高可用、高性能、安全等

#### 方式一

- 使用redis的setnx方法
  - stringRedisTemplate.opsForValue.setIfAbsent("业务名:" , 线程Id, 过期时间, 时间单位)
- 当线程因阻塞问题导致锁过期了，其他线程就能趁虚而入了
- 分布式下有可能出现线程id冲突的情况
- 改进：
  - 使用UUID+线程id 作为value值
  - 释放锁前先判断value值是否一致，必须保证判断和释放锁是原子性操作
    - 使用Lua脚本保证原子性
- 缺点：
  - 不可重入、不可重试、超时释放

#### 方式二

- 使用 <a href="./Redis.md#Redission">Redission</a> 是redis的一个分布式工具集合
- 使用Redission的分布式锁或者连锁

### 异步秒杀

- 由于当前大多数业务都是在tomcat中串行执行的，效率低
- 把秒杀券信息也保存到redis中
- 所以我们可以把判断库存、扣减库存和一人一单校验放在redis中进行
  - 使用lua脚本确保原子性
- 用户秒杀成功后将优惠券id和用户id存入**阻塞队列**中，将订单id返回给用户
- 开启线程任务不断读取**阻塞队列**中的订单异步下单

### 消息队列

- 使用Stream的Group（消费组）实现消息队列
  - 消息分流
    - 队列中的消息会分流给组内不同的消费者，从而加快消息处理速度
  - 消息标示
    - 会维护一个标识记录最后一个被处理的消息，即使宕机重启，还会从标识之后读取消息，确保每一个消息都被消费
  - 消息确认
    - 消费者获得消息后，消息会处于pending状态，并存入一个pending-list，当处理完消息后需要通过XACK来确认消息，标记消息已处理，才会从pending-list移除
  - 可以阻塞读取
  - 消息可回溯，即持久化存储
  
  #### 步骤
  
  - 创建一个StreamGroup类型的消息队列，名为stream.orders
    - `XGROUP CREATE stream.orders g1 0 MKSTREAM`
  - 使用lua脚本向消息队列中添加消息
  - 开启一个线程从消息队列中取出消息
    - 判断信息是否为空
      - 空继续下一轮循环
      - 非空解析信息然后创建订单
  - 确认消息队列中的消息
  - 如果这个过程出现异常需要处理pending-list
  
  ```java
  private class VoucherOrderHandler implements Runnable {
      @Override
      public void run() {
          //在线程池中开启一个线程处理消息队列中的消息
          while (true) {
              try {
                  // 1.获取消息队列中的订单信息
                  List<MapRecord<String, Object, Object>> list 
                      = stringRedisTemplate.opsForStream().read(
                                  Consumer.from("g1", "c1"),StreamReadOptions
                                  .empty()
                                  .count(1)
                                  .block(Duration.ofSeconds(2)),
                                  StreamOffset.create("stream.orders", ReadOffset.lastConsumed())
                          );
                  // 2.判断订单信息是否为空
                  if (list == null || list.isEmpty()) {
                      // 如果为null，说明没有消息，继续下一次循环
                      continue;
                  }
                  // 解析数据
                  MapRecord<String, Object, Object> record = list.get(0);
                  Map<Object, Object> value = record.getValue();
                  VoucherOrder voucherOrder 
                      = BeanUtil.fillBeanWithMap(value, new VoucherOrder(), true);
                  // 3.创建订单
                  createVoucherOrder(voucherOrder);
                  // 4.确认消息 XACK
                  stringRedisTemplate
                      .opsForStream().acknowledge("s1", "g1", record.getId());
              } catch (Exception e) {
                  log.error("处理订单异常", e);
                  handlePendingList();
              }
          }
      }
  
      //创建订单发生异常 或 确认x
      private void handlePendingList() {
          while (true) {
              try {
                  // 1.获取pending-list中的订单信息 
                  List<MapRecord<String, Object, Object>> list 
                      = stringRedisTemplate.opsForStream()
                      .read(
                          Consumer.from("g1", "c1"),
                          StreamReadOptions.empty().count(1),
                          StreamOffset.create("stream.orders", ReadOffset.from("0")));
                  // 2.判断订单信息是否为空
                  if (list == null || list.isEmpty()) {
                      // 如果为null，说明没有异常消息，结束循环
                      break;
                  }
                  // 解析数据
                  MapRecord<String, Object, Object> record = list.get(0);
                  Map<Object, Object> value = record.getValue();
                  VoucherOrder voucherOrder 
                      = BeanUtil.fillBeanWithMap(value, new VoucherOrder(), true);
                  // 3.创建订单
                  createVoucherOrder(voucherOrder);
                  // 4.确认消息 XACK
                  stringRedisTemplate
                      .opsForStream()
                      .acknowledge("s1", "g1", record.getId());
              } catch (Exception e) {
                  log.error("处理订单异常", e);
              }
          }
      }
  }
  ```
  

## 点赞

### 一人点一次

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

### 点赞排行榜

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

## 关注

### 关注和取关

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

### 查看是否关注

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

### 共同关注

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

## 关注推送（Feed流）

- 推送模式有三种（推、拉、推拉结合）
- 本项目采用推模式，实现简单
- 给用户创建一个收件箱，用户发送笔记时推送到所有粉丝的收件箱，收件箱要满足可以根据时间戳排序，使用redis实现
- 用户查询收件箱时实现分页查询 （SortedSet）
- 在用户发布笔记时将博客id存入redis

### 用户发布笔记推送到粉丝收件箱

```java
public Long saveBlog(Blog blog){
    //获取登录用户
    UserDTO user = UserHolder.getUser();
    blog.setUserId(user.getId());
    //保存笔记
    boolean success = save(blog);
    if(!success) return;
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
    //返回id
    return blog.getId();
}
```

### 粉丝读取收件箱笔记

- 需要先创建滚动分页的dto类

```java
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

```java
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

## 附近商铺

- 使用red中的GEO数据结构

- 按照商户类型做分组，相同类型的存入同一个GEO集合中

### 将商户存入redis

```java
public void loadShopData(){
    //查询商铺信息
    List<Shop> list = shopService.list();
    //把店铺按照typeId分组
    Map<Long, List<Shop>> map = list.stream().collect(Collectors.groupingBy(Shop::getTypeId))
    //分批写入redis中
    for(Map.Entry<Long, List<Shop>> entry : map.entrySet()){
        //获取typeId
    	Long typeId = entry.getKey();
        String key = "shop:geo:" + typeId;
    	//获取同类型店铺的集合
    	List<Shop> value = entry.getValue();
        List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>(value.size());
    	//写入redis的GEO集合中
        for(Shop shop : value){
            //封装到一个集合中
            locations.add(new RedisGeoCommands.GeoLocation<>(
                shop.getId().toString(), new Point(shop.getX(), shop.getY())));
        }
        //一次性插入Redis的GEO集合
        stringRedisTemplate.opsForGeo().add(key, locations);
    }	
}
```

### 根据类型查询商户

```java
public Page<Shop> queryShopByType(Integer typeId, Integer current, Double x, Double y){
    //判断是否需要根据坐标查询
    if(x == null || y == null){
        //不需要根据坐标查询，则按数据库查询
        Page<Shop> page = query()
            .eq("type_id", typeId)
            .page(new Page<>(current, SystemConstants.DEFAULT_PAGE_SIZE));
        return page;
    }
    //计算分页查询
    int from = (current - 1) * SystemConstants.DEFAULT_PAGE_SIZE;
    int end = current * SystemConstants.DEFAULT_PAGE_SIZE;
    //查询redis，按照距离排序
    String key = "shop:geo:" + typeId;
    GeoResults<RedisGeoCommands.GeoLocation<String>> results 
        = stringRedisTemplate.opsForGeo().search(
    		key,
        	GeoReference.fromCoordinate(x, y),
        	new Distance(5000),
        	RedisGeoCommands.GeoSearchCommandArgs.newGeoSearchArgs().includeDistance().limit(end)
    	);
    //解析id
    if(results == null) return null;
    List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = results.getContent();
    if(list.size() <= from) return null;
    List<Long> ids = new ArrayList<>(list.size());
    Map<String, Distance> distanceMap = new HashMap<>(list.size());
    list.stream().skip(from).forEach(result -> {
        String shopIdStr = result.getContent().getName();
        ids.add(Long.valueOf(shopIdStr));
        Distance distance = result.getDistance();
    });
    //根据id查店铺
    String idStr = StrUtil.join(",", ids);
    List<Shop> shops = query().in("id", ids).last("ORDER BY FIELD(id,"+idStr+")").list();
    for(Shop shop : shops){
        shop.setDistance(distanceMap.get(shop.getId().toString()).getValue());
    }
    return shops;
}
```

## 签到活动

- 使用Redis的BitMap数据结构
  - Redisstring类型	来实现ap的，最大上限512M（2^32^ bit）

- 有签到为1.没签到为0

### 签到

```java
public void sign(){
    //获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    //获取日期
    LocalDateTime now = new LocalDateTime.now();
    //拼接key
    String keySufix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = "sign:" + userId + keySufix;
    //获取今天是本月第几天
    int dayOfMonth = now.getDayOfMonth();
    //写入redis
    stringRedisTemplate.opsForValue.setBit(key, dayOfMonth - 1, true);
}
```

### 签到统计

```java
public int signCount(){
    //获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    //获取日期
    LocalDateTime now = new LocalDateTime.now();
    //拼接key
    String keySufix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = "sign:" + userId + keySufix;
    //获取今天是本月第几天
    int dayOfMonth = now.getDayOfMonth();
    //获取本月截至到今天的所有签到记录
    List<Long> result = stringRedisTemplate.opsForValue().bitField(
    	key,
        BitFieldSubCommands
        .create()
        .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
        .valueAt(0)
    );
    if(result == null || result.isEmpty()) return 0;
    Long num = result.get(0);
    if(num == null || num == 0) return 0;
    //循环遍历
    int count = 0;
    while(true){
        //与1做与运算，取出最后一位 ,判断这一位是否为0
		if((num & 1) == 0){
            //为0，说明未签到，结束
            break;
		}else{
        	//为1，说明已签到，计数器+1
            count++;
        }
        //把记录右移一位
        num >>>= 1;
    }
    return count;
}
```
