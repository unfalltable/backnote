# @Scheduled实现定时任务

### 参数详解

- cron表达式

  ```text
  ┌───────────── second (0-59)
  │ ┌───────────── minute (0 - 59)
  │ │ ┌───────────── hour (0 - 23)
  │ │ │ ┌───────────── day of the month (1 - 31)
  │ │ │ │ ┌───────────── month (1 - 12) (or JAN-DEC)
  │ │ │ │ │ ┌───────────── day of the week (0 - 7) (0 or 7 is Sunday, or MON-SUN)
  │ │ │ │ │ │
  │ │ │ │ │ │
  * * * * * *
  ```

  - \* 和 ? 都代表通配符，? 可以用在第四（天--月）和第六（天--周）
  - L 表示最后一个时间单位，例如一个月最后的一个星期天
  - W 表示工作日
  - \# 表示每月中的第几个星期几，5#2 表示每月第二个星期五，MON#1 表示每月第一个星期一
  - 也可以使用@yearly、@monthly、@weekly、@daily、@midnight、@hourly
    - 表示整年，整月，整周，整天或者整夜，整小时

  - 例子

    ```
    0 0 10,14,16 * * ?          每天上午10点，下午2点，4点
    0 0/30 9-17 * * ?    		朝九晚五工作时间内每半小时
    0 0 12 ? * WED 				表示每个星期三中午12点
    0 0 12 * * ? 				每天中午12点触发 
    0 15 10 ? * * 				每天上午10:15触发 
    0 15 10 * * ? 				每天上午10:15触发 
    0 15 10 * * ? * 			每天上午10:15触发 
    0 15 10 * * ? 2005 			2005年的每天上午10:15触发 
    0 * 14 * * ? 				在每天下午2点到下午2:59期间的每1分钟触发 
    0 0/5 14 * * ? 				在每天下午2点到下午2:55期间的每5分钟触发 
    0 0/5 14,18 * * ? 			在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 
    0 0-5 14 * * ? 				在每天下午2点到下午2:05期间的每1分钟触发 
    0 10,44 14 ? 3 WED 			每年三月的星期三的下午2:10和2:44触发 
    0 15 10 ? * MON-FRI 		周一至周五的上午10:15触发 
    0 15 10 15 * ? 				每月15日上午10:15触发 
    0 15 10 L * ? 				每月最后一日的上午10:15触发 
    0 15 10 ? * 6L 				每月的最后一个星期五上午10:15触发 
    0 15 10 ? * 6L 2002-2005 	2002年至2005年的每月的最后一个星期五上午10:15触发 
    0 15 10 ? * 6#3 			每月的第三个星期五上午10:15触发
    ```

- zone：时区

- timeUnit：时间单位，从 5.3.10开始，spring boot 2.5.5开始

- fixedDelay：固定间隔，参数类型为long，以任务结束时间算起

  - fixedDelayString：同fixedDelay，参数类型为String，用于接收配置文件的值

- fixedRate：固定速率，参数类型为long，以任务开始时间算起

  - fixedRateString：同fixedRate，参数类型为String，用于接收配置文件的值

- initialDelay：第一次延时时间，参数类型为long。

  - initialDelayString：第一次延时时间，参数类型为String。

### 配置文件

- 一般写执行时间，cron表达式等等

  ```yaml
  erwin:
  	crom: 
  	fixed-delay: 
  	fixed
  ```

  

### 步骤

- 启用调度注解

  ```java
  @Configuration
  @EnableScheduling
  public class ScheduledConfiguration {
  
  }
  ```

- 在需要执行定时任务的方法上

  ```java
  @Scheduled(initialDelay = 5, fixedRate = 5, timeUnit = TimeUnit.SECONDS)
  public void method() {
      log.info("开始执行数据同步任务");
  }
  ```

# 字段校验

## @Validated

## @Valid

- 在实体类上添加对应的注解
  - @NotBlank、@Length、@NotNull、@Range（范围）
- 在接口入参处使用BingingResult bindingResult 检验返回结果
  - 使用hasErrors处理