---
title: 框架设计
toc: content
keywords: [frame]
---

# 总体

## 字典

## 实体类设计

```java
@Getter
@Setter
public class DbEntity<T> {
    //创建时间
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;

    //修改时间
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
    
    //创建者ID
    @TableField(fill = FieldFill.INSERT)
    private T createBy;
    
    //修改着ID
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private T updateBy;
}
```

## 全局常量

```java
public interface CommonConstants {
    //菜单树根节点
    Long MENU_TREE_ROOT_ID = 0L;

    //正常
    String STATUS_NORMAL = "0";

    //请求开始时间
    String REQUEST_START_TIME = "REQUEST-START-TIME";

    //编码
    String UTF8 = "UTF-8";

    //SON 资源
    String CONTENT_TYPE = "application/json; charset=utf-8";

    //成功标记
    Integer SUCCESS = 1;

    //失败标记
    Integer FAIL = 0;

    //当前页
    String PAGE_CURRENT = "p";

    //size
    String PAGE_SIZE = "ps";

    int PAGE_CURRENT_DEFAULT = 1;
    int PAGE_SIZE_DEFAULT = 20;
}
```

## 异常处理

### 方案一

```java
public enum ExceptionEnum implements ExceptionResponse, ExceptionAssert {
    //内部异常（打印日志）
    INNER_MSG("{0}"),
    //异常信息（返回给客户端异常信息）
    CLIENT_MSG("{0}"),
    //异常编码，配置异常信息，返回给客户端
    CLIENT_CODE("{0}"),
    ;
    private String message;
    ExceptionEnum(String message) {
        this.message = message;
    }
    @Override
    public String getMessage() {
        return message;
    }
    
    @Override
    public DefRunException newException(Object... args) {
        String msg = MessageFormat.format(this.getMessage(), args);
        return new DefRunException(this, msg);
    }

    @Override
    public DefRunException newException(Throwable t, Object... args) {
        String msg = MessageFormat.format(this.getMessage(), args);
        return new DefRunException(this, msg, t);
    }
    //直接抛配置好的异常信息给客户端
    public void th(Object... args) {
        throw newException(args);
    }
    //直接抛内部异常，打印日志
    public static void thro(Object... args) {
        throw INNER_MSG.newException(args);
    }
    
    //格式化输出 ("aa {}", "aa")
    public static String format(String msg, Object... objects) {
        FormattingTuple ft = MessageFormatter.arrayFormat(msg, objects);
        return ft.getMessage();
    }
}
```

```java
public class DefRunException extends RuntimeException implements ExceptionResponse {

    private String message;
    private ExceptionEnum ex;

    public DefRunException(final ExceptionEnum ex, final String message) {
        super(message);
        init(ex, message);
    }

    public DefRunException(final ExceptionEnum ex, final String message, final Throwable cause) {
        super(message, cause);
        init(ex, message);
    }

    @Override
    public String getMessage() {
        return message;
    }

    public ExceptionEnum getExceptionEnum() {
        return this.ex;
    }

    private void init(ExceptionEnum ex, String message) {
        this.message = message;
        this.ex = ex;
    }
}
```

```java
public interface ExceptionResponse {
    //取得错误信息
    String getMessage();
}
```

```java
public interface ExceptionAssert {
    //创建异常
    DefRunException newException(Object... args);
    DefRunException newException(Throwable t, Object... args);
    
    //断言该对象不能为空对象
    default void assertNotNull(Object obj, Object... args) {
        if (obj == null) {
            throw newException(args);
        }
        if (obj instanceof String) {
            if (((String) obj).length() == 0) {
                throw newException(args);
            }
        }
    }
    
    //断言是否True，是True则测试用例通过。
    default void assertTrue(boolean isTrue, Object... args) {
        if (!isTrue) {
            throw newException(args);
        }
    }

    //断言是否False，是False则测试用例通过。
    default void assertFalse(boolean isTrue, Object... args) {
        if (isTrue) {
            throw newException(args);
        }
    }
    
    //有一个为 null
    default void hasEmpty(String msg, Object... object) {
        if (ObjectUtil.hasEmpty(object)) {
            throw newException(ObjectUtil.isNull(msg) ? "参数不能为null!" : msg);
        }
    }
}
```

### 方案二

#### @ControllerAdvice

## 返回值处理

R

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class R<T> implements Serializable {
    private static final long serialVersionUID = 1L;

    private int code;
    private String msg;
    private T data;
    
    public static R ok() {
        return new R(CommonConstants.SUCCESS, null, null);
    }
    public static <T> R<T> ok(T data) {
        return new R(CommonConstants.SUCCESS, null, data);
    }
    public static R failed() {
        return new R(CommonConstants.FAIL, null, null);
    }
    public static R failed(String msg) {
        return new R(CommonConstants.FAIL, msg, null);
    }
    public static R bol(boolean result) {
        return bol(result, null);
    }
    public static R bol(boolean result, String msg) {
        return result ? R.ok() : R.failed(msg);
    }
    public static <T> R<T> bol(T t, String msg) {
        return Objects.nonNull(t) ? R.ok(t) : R.failed(msg);
    }
}
```

## 分页处理

### 分页入参处理

```java
public class PageFactory {

    public static <T> Page<T> getPage() {
        HttpServletRequest request = HttpServletUtil.getRequest();
        switch (HttpMethod.valueOf(request.getMethod())) {
            case GET:
                return getPageOfGetReq(request);
            case POST:
                // TODO
            default:break;
        }
        return new Page<T>();
    }

    private static <T> Page<T> getPageOfGetReq(HttpServletRequest request) {
        String ps = null;
        String p = null;
        if (ContentType.JSON.equals(request.getContentType())) {
            // TODO
        } else {
            ps = request.getParameter(CommonConstants.PAGE_SIZE);
            p = request.getParameter(CommonConstants.PAGE_CURRENT);
        }
        return new Page<T>(Convert.toInt(p, CommonConstants.PAGE_CURRENT_DEFAULT), Convert.toInt(ps, CommonConstants.PAGE_SIZE_DEFAULT));
    }
}
```

### 分页返回处理

```java
public class PageUtils {
    public static <S, T> IPage<T> covert(IPage<S> s, Class<T> t, Function<S, T> convert) {
        Objects.requireNonNull(s);
        Page<T> page = new Page<>(s.getCurrent(), s.getSize(), s.getTotal());
        page.setPages(s.getPages());
        List<S> data = s.getRecords();
        page.setRecords(data.stream().map(convert).collect(Collectors.toList()));
        return page;
    }
    public static <S, T> IPage<T> covert(IPage<S> s, List<S> data, Class<T> t, Function<S, T> convert) {
        Objects.requireNonNull(s);
        Objects.requireNonNull(data);
        Page<T> page = new Page<>(s.getCurrent(), s.getSize(), data.size());
        page.setRecords(data.stream().map(convert).collect(Collectors.toList()));
        return page;
    }
}
```



## 数据校验

### 使用Hibernate-Validator

- 先在controller上加 @Validated

#### 字段校验

- 入参是字段的话在前面加上 Valid相关注解即可
- 如果是对象的话在其前加 @Valid，然后在对象内部的各个字段上加 Valid相关注解

#### 分组校验

- 在不同的场景下使用不同的校验规则，有些字段可能新增的时候需要检验，但是修改删除不用校验等等
- 需要继承 BaseParam ，使用 Valid相关注解时添加 groups 属性，然后对象上添加 @Validated(BaseParam.对应的组.class)
- 如果使用了分组校验的话所有需要校验的字段都需要加入对应的组，否则不起作用

##### BaseParam

```java
@Getter
@Setter
public class ValidParam implements Serializable {
    private static final long serialVersionUID = 1L;
    //参数校验分组：增加
    public @interface add {}
    //参数校验分组：编辑
    public @interface edit {}
    //参数校验分组：删除
    public @interface delete {}
}
```

#### 自定义校验注解



#### 注解表

| 注解                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| @Null                       | 被注释的元素必须为 null                                      |
| @NotNull                    | 被注释的元素必须不为 null                                    |
| @AssertTrue                 | 被注释的元素必须为 true                                      |
| @AssertFalse                | 被注释的元素必须为 false                                     |
| @Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值     |
| @Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值     |
| @DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值     |
| @DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值     |
| @Size(max, min)             | 被注释的元素的大小必须在指定的范围内，元素必须为集合，代表集合个数 |
| @Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内         |
| @Past                       | 被注释的元素必须是一个过去的日期                             |
| @Future                     | 被注释的元素必须是一个将来的日期                             |
| @Email                      | 被注释的元素必须是电子邮箱地址                               |
| @Length(min=, max=)         | 被注释的字符串的大小必须在指定的范围内，必须为数组或者字符串，若微数组则表示为数组长度，字符串则表示为字符串长度 |
| @NotEmpty                   | 被注释的字符串的必须非空                                     |
| @Range(min=, max=)          | 被注释的元素必须在合适的范围内                               |
| @NotBlank                   | 被注释的字符串的必须非空                                     |
| @Pattern(regexp = )         | 正则表达式校验                                               |

## 配置处理

1. 继承 GsSettingEntity，实现其抽象方法进行值初始化

```java
public abstract class GsSettingEntity implements Serializable {
    private static final long serialVersionUID = -4015423388607164978L;

    {
        defaultValue();
    }

    /**
     * 必须初始化值
     */
    abstract void defaultValue();
}
```

```java
@Getter
@Setter
@ToString
public class Gs extends GsSettingEntity {
    private static final long serialVersionUID = -1629258515233154346L;

    //对象内部要对值进行初始化，也需要继承 GsSettingEntity
    @Valid
    private Gs1 gs1;

    @Valid
    private String str;
    @Override
    void defaultValue() {
        gs1 = new Gs1();
        this.str = "默认值";
    }
}
```



## 导入导出

### EasyExcel

导出

```java
@ResponseExcel
@GetMapping("/export")
public List<ProductLineVO> export(@Validated Param param) {
    //返回数据
}
```

导入

```java
@PostMapping("/import")
public R<?> importUser(@RequestExcel(ignoreEmptyRow = true) List<ProducePlanVO> excelVOList, 
                       BindingResult bindingResult) {
    //数据存入数据库
}
```

## 日志处理

### LogBack

### 二

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface OperationLog {
    /** 业务的名称,例如:"修改菜单" */
    String title() default "";

    /** 业务操作类型枚举 */
    LogOpTypeEnum opType() default LogOpTypeEnum.OTHER;
}
```

```java
@Getter
public enum LogOpTypeEnum {
    //其它 增加   删除    编辑   更新    查询 
    OTHER, ADD, DELETE, EDIT, UPDATE, QUERY,  
    //详情   树   导入   导出   授权   强退  清空  
    DETAIL,TREE,IMPORT,EXPORT,GRANT,FORCE,CLEAN,
    //修改状态 
    CHANGE_STATUS
}
```

## 文件上传

## 权限处理

- 使用 @PreAuthorize("@pms.hasPermission("权限字段")") 进行权限配置

- 主要分为四种使用场景来设计

  - 外部访问内部有可能造成危险的接口
    - 对用户进行登录校验
    - 对用户进行登录校验和权限校验

  - 外部访问内部安全的接口
    - 通过配置文件配置添加忽略路径
    - 通过 @Inner注解
    - 通过配置类
  - 内部访问内部接口
    - 通过OpenFeign进行远程调用
  - 访问第三方接口
    - 通过OpenFeign进行远程调用

## 部署

### Docker-compose

1. 需要将各个模块分别部署，所有模块需要在同一级目录下
   - mysql需要打包配置和数据
2. 编写对应的 DockerFile 文件

```dockerfile
# 第一阶段
FROM openjdk:8 as builder
# 工作目录为/build 相对目录
WORKDIR /build
# 设置常量
ARG JAR_FILE=./模块名.jar
# 拷贝target/模块名.jar  到/build/app.jar
COPY ${JAR_FILE} app.jar
# 执行shell命令 使用分层 这里就是解压jar包分层成各个目录  目的是为了更新的时候局部更新 加快镜像pull速度
# /build/dependencies
# /build/snapshot-dependencies
# /build/spring-boot-loader
# /build/application
RUN java -Djarmode=layertools -jar app.jar extract && rm app.jar

# 第二阶段 引用第一阶段的镜像builder
FROM openjdk:8
MAINTAINER 569421432@qq.com
ENV JAVA_OPTS="-Xms1024m -Xmx1024m -Duser.timezone=GMT+8 -DNACOS_HOST=10.1.3.160 -Djava.security.egd=file:/dev/./urandom"
ENV JAVA_EX="-DMYSQL_HOST=192.168.8.91 -DMYSQL_PORT=3306 -DMYSQL_PWD=hj@123456 -DMYSQL_DB=bling -DREDIS_HOST=192.168.8.91 -DREDIS_PWD=hj@123456 -Dspring.profiles.active=test"
WORKDIR /opt

# 将builder（第一阶段）的文件copy到/opt
COPY --from=builder /build/dependencies/ ./
COPY --from=builder /build/snapshot-dependencies/ ./
COPY --from=builder /build/spring-boot-loader/ ./
COPY --from=builder /build/application/ ./

EXPOSE 8080

# 容器启动时执行
CMD java $JAVA_OPTS $JAVA_EX org.springframework.boot.loader.JarLauncher
```

3. 编写一个方便重新启动的文件 `restart.sh` 

```shell
#!/bin/bash
if [ "$1" = '' ]; then
    echo "You need to enter a project name!"
else
    docker-compose stop $1
    docker-compose rm -f $1
    docker-compose build $1
    docker-compose up -d $1
    echo "ok !!!"
fi
```

4. 启动：`./restart.sh 模块名`



# 工具封装

## Redis工具类

## Date工具类

## Security工具类

## Url工具类

```java
public class UrlUtils {
    //判断一个字符串是否为url
    public static boolean isURL(String str){
        //转换为小写
        str = str.toLowerCase();
        String regex = "^((https|http|ftp|rtsp|mms)?://)"  //https、http、ftp、rtsp、mms
                + "?(([0-9a-z_!~*'().&=+$%-]+: )?[0-9a-z_!~*'().&=+$%-]+@)?" //ftp的user@
                + "(([0-9]{1,3}\\.){3}[0-9]{1,3}" // IP形式的URL- 例如：199.194.52.184
                + "|" // 允许IP和DOMAIN（域名）
                + "([0-9a-z_!~*'()-]+\\.)*" // 域名- www.
                + "([0-9a-z][0-9a-z-]{0,61})?[0-9a-z]\\." // 二级域名
                + "[a-z]{2,6})" // first level domain- .com or .museum
                + "(:[0-9]{1,5})?" // 端口号最大为65535,5位数
                + "((/?)|" // a slash isn't required if there is no file name
                + "(/[0-9a-z_!~*'().;?:@&=+$,%#-]+)+/?)$";
        return  str.matches(regex);
    }

    //解析出url参数中的键值对
    //如 "index.jsp?Action=del&id=123"，解析出Action:del,id:123存入map中 
    public static Map<String, String> URLRequest(String URL) {
        Map<String, String> mapRequest = new HashMap<String, String>();
        String[] arrSplit = null;
        String strUrlParam = TruncateUrlPage(URL);
        if (strUrlParam == null) {
            return mapRequest;
        }
        //每一个键值为一组 www.2cto.com
        arrSplit = strUrlParam.split("[&]");
        for (String strSplit : arrSplit) {
            String[] arrSplitEqual = null;
            arrSplitEqual = strSplit.split("[=]");
            //解析出键值
            if (arrSplitEqual.length > 1) {
                //正确解析
                mapRequest.put(arrSplitEqual[0], arrSplitEqual[1]);

            } else {
                if (arrSplitEqual[0] != "") {
                    //只有参数没有值，不加入
                    mapRequest.put(arrSplitEqual[0], "");
                }
            }
        }
        return mapRequest;
    }

    //去掉url中的路径，留下请求参数部分 
    private static String TruncateUrlPage(String strURL) {
        String strAllParam = null;
        String[] arrSplit = null;
        strURL = strURL.trim();
        arrSplit = strURL.split("[?]");
        if (strURL.length() > 1) {
            if (arrSplit.length > 1) {
                if (arrSplit[1] != null) {
                    strAllParam = arrSplit[1];
                }
            }
        }

        return strAllParam;
    }
}
```

## OkHttp3工具类

```java
public class OkHttp3Utils {

    private static OkHttpClient httpClient = null;
    private static OkHttpClient httpsClient = null;
    private static OkHttpClient httpsClientW = null;
    
    private static Type type = Type.HTTP;
    public enum Type {
        /**
         * http
         */
        HTTP,
        /**
         * https
         */
        HTTPS,
        /**
         * https 绕过
         */
        HTTPSW;
    }

    public static String get(final String url) throws IOException {
        return commonRequest(url, Request.Builder::get);
    }

    // application/x-www-form-urlencoded
    public static String post8from(final String url, final Map<String, String> parameters) throws IOException {
        return post8from(url, formBody -> {
                if (parameters == null) {
                    throw new IllegalAccessError("parameters is null!");
                }
                for(Map.Entry<String, String> entry : parameters.entrySet()) {
                    formBody.add(entry.getKey(), entry.getValue());
                }
            });
    }

    // application/x-www-form-urlencoded
    public static String post8from(final String url, final String... values) throws IOException {
        return post8from(url, formBody -> {
            if (values != null && values.length > 2 && values.length%2 == 0) {
                for(int index = 0; index < values.length;) {
                    formBody.add(values[index++], values[index++]);
                }
            }
        });
    }

    // application/x-www-form-urlencoded
    public static String post8from(final String url, final Consumer<FormBody.Builder> consumer) throws IOException {
        FormBody.Builder formBody = new FormBody.Builder();
        consumer.accept(formBody);
        RequestBody requestBody = formBody.build();
        return commonRequest(url, builder -> builder.post(requestBody));
    }

    // application/json
    public static String post8json(final String url, final Object jsonObject) throws IOException {
        if (jsonObject == null) {
            throw new IllegalAccessError("jsonObject is null!");
        }
        return post8json(url, JSONObject.toJSONString(jsonObject));
    }

    // application/json
    public static String post8json(final String url, final String json) throws IOException {
        if (json == null) {
            throw new IllegalAccessError("json is null!");
        }
        RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8"), json);
        return commonRequest(url, builder -> builder.post(body));
    }

    // application/json
    public static String post8json(final Consumer<Request.Builder> header, final String url, final String json) throws IOException {
        if (json == null) {
            throw new IllegalAccessError("json is null!");
        }
        RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8"), json);
        return commonRequest(url, builder -> {
                header.accept(builder);
                builder.post(body);
        });
    }

    public static String checkHttpUrl(final String url) {
        // TODO 汉子处理
        /*if (url == null || !UrlUtils.isURL(url)) {
            throw new IllegalArgumentException("Http 请求 Url 错误: " + url);
        }*/
        return url;
    }

    private static String commonRequest(final String url, final Consumer<Request.Builder> consumer) throws IOException {
        Request.Builder builder = new Request.Builder().url(checkHttpUrl(url));

        // 请求做成
        consumer.accept(builder);
        Request request = builder.build();
        final Call call = getHttpClient().newCall(request);
        Response response = call.execute();
        return Objects.requireNonNull(response.body()).string();
    }

    public static synchronized String executeHttpRequest(Supplier<String> http, Type type) {
        String result;
        OkHttp3Utils.type = type;
        result = http.get();
        OkHttp3Utils.type = Type.HTTP;
        return result;
    }

    private static synchronized OkHttpClient getHttpClient() {
        OkHttpClient client = null;
        switch (type) {
            case HTTP:
                if (httpClient == null) {
                    httpClient = new OkHttpClient.Builder()
                            // 设置连接超时时间
                            .connectTimeout(60, TimeUnit.SECONDS)
                            //设置读取超时时间
                            .readTimeout(60, TimeUnit.SECONDS)
                            .build();
                }
                client = httpClient;
                break;
            case HTTPS:
                // 暂时不用
            case HTTPSW:
                if (httpsClientW == null) {
                    X509TrustManager manager = SSLSocketClient.getX509TrustManager();
                    OkHttpClient.Builder builder = new OkHttpClient.Builder();
                    builder.sslSocketFactory(SSLSocketClient.getSocketFactory(manager), manager);
                    builder.hostnameVerifier(SSLSocketClient.getHostnameVerifier());
                    httpsClientW = builder
                            // 设置连接超时时间
                            .connectTimeout(60, TimeUnit.SECONDS)
                            //设置读取超时时间
                            .readTimeout(60, TimeUnit.SECONDS)
                            .build();
                }
                client = httpsClientW;
                break;
            default:
                // nothing do to.
                break;
        }
        // 重置
        type = Type.HTTP;
        return client;
    }

    
}
```



# 实用技巧

### 接口访问时长统计注解 @ApiTime

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ApiTime {
}
```

### 数据托敏 @DataMasking(maskFunc = ）

### Echarts统一数据集

### Excel导出动态列宽Excel导出下拉框
