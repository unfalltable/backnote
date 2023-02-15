---
title: 数据库连接池
toc: content
keywords: [pool, db]
---



# Druid

## 简介

- 是一个JDBC组件，包含三个部分
  - DruidDriver：代理Driver，能够提供基于Filter－Chain模式的插件体系
  - DruidDataSource ：高效可管理的数据库连接池
  - SQLParser ：
- 用途
  - 监控数据库访问性能，Druid内置提供了一个功能强大的StatFilter插件，能够详细统计SQL的执行性能，这对于线上分析数据库访问性能有帮助
  - 高效、功能强大、可扩展性好
  - 数据库密码加密。直接把数据库密码写在配置文件中，这是不好的行为，容易导致安全问题。DruidDruiver和DruidDataSource都支持PasswordCallback
  - SQL执行日志，Druid提供了不同的 LogFilter，能够支持 Common-Logging、Log4j 和 JdkLog
  - 可以通过Druid提供的Filter-Chain机制，很方便编写JDBC层的扩展插件

## 使用步骤

1. 导入依赖
2. 单数据源配置配置

```yaml
spring:
  application:
    name: druidDemo
  datasource:
    url: jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: xxx # 数据库账号
    password: xxx@ # 数据库密码
    type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
    # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
    druid: # 设置 Druid 连接池的自定义配置。然后 DruidDataSourceAutoConfigure 会自动化配置 Druid 连接池。
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
```

3. 多数据源配置

```yaml
spring:
  application:
    name: druidDemo
  datasource:
    mall:
      url: jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: root # 数据库账号
      password: root0319@ # 数据库密码
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
    # 用户数据源配置
    users:
      url: jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: root # 数据库账号
      password: root0319@ # 数据库密码
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
```

4. 配置监控台

```yaml
druid: # 设置 Druid 连接池的自定义配置。然后 DruidDataSourceAutoConfigure 会自动化配置 Druid 连接池。
	filter:
		stat: # 配置 StatFilter 
			log-slow-sql: true # 开启慢查询记录
			slow-sql-millis: 5000 # 慢 SQL 的标准，单位：毫秒
			merge-sql: true # SQL合并配置
	stat-view-servlet: # 配置 StatViewServlet
		enabled: true # 是否开启 StatViewServlet
		login-username: root # 账号
		login-password: root # 密码
```

- stat文档：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter
- stat-view-servlet文档：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
- Druid监控台：http://127.0.0.1:8082/druid/sql.html 

使用

```java
public static Connection getConnection(){
    try {
        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("Druid.Properties");
        Properties prop = new Properties();
        prop.load(is);
        DataSource source = DruidDataSourceFactory.createDataSource(prop);
        return source.getConnection();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

# Hikari