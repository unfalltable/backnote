---
title: Zookeeper
toc: content
keywords: [middle]
---

## 数据模型

- ZK是一个树形结构
- 每一个节点都称为ZNode，每个节点保存数据和节点信息
- 节点可以拥有子节点，也允许少量数据存储
- 节点分为四大类
  - Persistent  持久化节点
  - Ephemeral  临时节点
  - Persistent_Sequential 持久化顺序节点
  - Ephemeral  _Sequential 持久化临时节点

## 命令

- 启动服务
  - `./zkServer.sh start`
- 查看服务状态
  - `./zkServer.sh status`
- 停止服务
  - `./zkServer.sh stop`
- 重启服务
  - `./zkServer.sh restart`
- 连接zk
  - `./zkCLi.sh -server ip:port`
- 创建节点值
  - `create /节点path value`
- 设置节点值
  - `set /节点path value`
- 获取节点值
  - `get /节点path`
- 删除节点值
  - `delete /节点path`
- 删除有子节点的节点值
  - `deleteall /节点path`
- 创建临时节点
  - `create -e /path value`
- 创建顺序节点
  - `create -s /path value`
- 查询节点详细信息
  - `ls -s /path`
    - ![image-20220716152537751](..//pic//image-20220716152537751.png)

### Curator

- 建立连接

  - ```java
    //重试策略
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000, 10);
    //第一种方式
    CuratorFramework client = CuratorFrameworkFactory.newClient("ip地址", session过期时间, 连接过期时间, 重试策略);
    //第二种方式，建造者模式
    CuratorFramework client = CuratorFrameworkFactory.builder().connectString("ip地址")
        	.sessionTimeoutMs()
        	.connectionTimeoutMs()
        	.retryPolicy(retryPolicy).namespace("").build();
    //开启连接
    client.start();
    ```

- 添加节点

  - ```java
    public void Create() throws Exception{
        //创建节点，没传数据的话，则会将当前客户端的ip地址存储
        //.withMode(CreateMode.*)设置节点类型，默认是持久化
        //.creatingParentsIfNeeded()创建多级节点，路径名/节点名
        String path = client.create().forPath("路径名", "数据".getBytes());
    }
    ```

- 查询节点

  - ```java
    public void Get() throws Exception{
        //查询节点
        byte[] data = client.getData().forPath("路径名");
        //查询子节点
        List<String> path = client.getChildren().forPath("路径名");
        //查询节点状态，保存在status中
        Stat status = new Stat();
        client.getData().storingStatIn(status).forPath("路径名");
    }
    ```

- 修改节点

  - ```java
    public void Set(){
        //通过status.getVersion()获取version
        //.withVersion(version)
        client.setData().forPath("路径名", "数据".getBytes());
    }
    ```

- 删除节点

  - ```java
    public void Delete(){
        //删除节点
        client.delete().forPath("路径名");
        //删除带有子节点的节点
        client.delete().deletingChildrenIfNeeded().forPath("路径名");
        //保证删除成功
        client.delete().guaranteed().forPath("路径名");
        //失败回调
        client.delete().guaranteed().inBackground((client, event) -> {
            //回调逻辑
        }).forPath("路径名");
    }
    ```

## 事件监听

- Curator引入了Cache实现服务端的事件监听

  - NodeCache：监听一个节点

    - ```java
      public void NodeCache(){
          //创建NodeCache对象
          final NodeCache nodeCache = new NodeCache(client, "路径名");
          //注册监听
          nodeCache.getListenable().addListener(()->{
              /*节点发生变化后执行的逻辑*/
              //获取节点此时的数据
              byte[] data = nodeCache.getCurrentData().getData();
          });
          //开启监听
          nodeCache.start();
      }
      ```

  - PathChildrenCache：监听一个节点的子节点

    - ```java
      public void PathChildrenCache(){
          //创建PathChildrenCache对象
          PathChildrenCache pathChildrenCache = new PathChildrenCache(client, "路径名", 是否缓存数据);
          //注册监听
          pathChildrenCache.getListenable().addListener((client, event) -> {
              /*节点发生变化后执行的逻辑*/
              //获取节点此时的数据
              PathChildrenCache.Type type = event.getType(); //获取类型
              if(type.equals(PathChildrenCache.Type.CHILD_UPDATED)) //判断类型是否是Update
                  byte[] data = event.getData.getData();
          });
          //开启监听
          pathChildrenCache.start();
      }
      ```

  - TreeCache：监听所有节点

    - ```java
      public void TreeCache(){
          //创建TreeCache对象
          TreeCache treeCache = new TreeCache(client, "路径名");
          //注册监听
          treeCache.getListenable().addListener((client, event) -> {
              /*节点发生变化后执行的逻辑*/
          });
          //开启监听
          treeCache.start();
      }
      ```

## 分布式锁

- 核心思想：当客户端需要获取锁时，则创建节点，使用完锁后，删除该节点
  - 客户端获取锁时，在lock节点下创建临时顺序节点
  - 获取lock下所有子节点
    - 若发现自己创建的节点序号最小，那么获取锁，使用完后释放锁并删除节点
    - 若发现自己创建的节点序号不是最小，那么找到那个最小的节点，对其注册监听器，监听其删除事件
  - 监听到最小节点被删除时，再次判断自己的创建的节点是否是序号最小的
    - 是就获取锁，使用完后释放锁并删除节点
    - 不是就重复上面的步骤

```java
//获取连接
RetryPolicy retryPolicy = new ExponentiaBackoffRetry(3000,10);
CuratorFramework client = CuratorFrameworkFactory.builder()
    .connectString("地址")
    .sessionTimeoutMs("会话超时时间 毫秒")
    .connectionTimeoutMs("连接超时时间 毫秒")
    .retryPolicy(retryPolicy)
    .builder();
//开启连接
client.start();
//创建锁
InterProcessMutex lock = new InterProcessMutex(client, "/lock");
//获取锁
lock.acquire(超时时间，时间单位);

---临界区---

//释放锁
lock.release();
```

## 集群

- Leader选举，获得半数以上选票
  - Serverid：服务器ID，值越大，选中Leader可能性越大
  - Zxid：数据ID，值越大，选中Leader可能性越大

### 配置

- 在/data下创建myid文件，内容是服务器ID

- 编辑/conf下的zoo.cfg文件

  - ```yaml
    server.服务器ID=服务器ip地址:服务器之间通信端口:服务器之间投票选举端口
    server.服务器ID=服务器ip地址:服务器之间通信端口:服务器之间投票选举端口
    server.服务器ID=服务器ip地址:服务器之间通信端口:服务器之间投票选举端口
    ```

### 角色

- Leader 领导者
  - 处理事务请求
  - 集群内部各服务器调度者
- Follower 追随者
  - 处理非事务请求，转发事务请求给Leader
  - 参与Leader选举投票
- Observer 观察者
  - 处理非事务请求，转发事务请求给Leader
