## 阿里云

### 简介

- 采用服务端签名后直传
  - 客户端向服务器发送请求，服务端将阿里云的用户和密码生成一个签名
  - 并且携带要上传的位置
  - 然后前端直接将签名和文件交给阿里云，阿里云进行验证

### 使用

1. 依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/com.aliyun.oss/aliyun-sdk-oss -->
   <dependency>
       <groupId>com.aliyun.oss</groupId>
       <artifactId>aliyun-sdk-oss</artifactId>
       <version>3.10.2</version>
   </dependency>
   ```

   - 如果使用spring-cloud-alicloud-oss 的话会和nacos注册中心依赖有冲突，所以使用aliyun-sdk-oss好一些

2. 配置 aliyun.properties

   ```properties
   endpoingt="";
   accessKeyId="";
   accessKeySecret="";
   bucketName="";
   ```

3. 将 OSSClient 加入容器

   ```java
   @Getter
   @Setter
   @Configuration
   @ConfigurationProperties(prefix = "aliyun")
   public class AliyunConfig {
   
       private String endpoint;
       private String accessKeyId;
       private String accessKeySecret;
       private String bucketName;
   
       @Bean
       public OSSClient ossClient(){
           return new OSSClient(endpoint, accessKeyId, accessKeySecret);
       }
   }
   ```

4. 创建一个图片类用于上传，根据AntDesign定义的格式

   ```java
   @Data
   public class PicUploadResult{
       private String uid;
       private String name;
       private String status;
       private String response;
   }
   ```

5. 实现

   ```java
   @Service
   public class PicUploadService {
       private static final String[] TYPE = new String[]{".bmp",".jpg",".jpeg",".gif",".png",}
       public PicUploadResult upload(MultpartFile uploadFile){
           @Autowired
           private OssClient ossClient;
           @Autowired
           private AliyunConfig aliyunConfig;
           
           PicUploadResult file = new PicUploadResult();
           boolean isLegal = false;
           //判断文件后缀名是否正确
           for(String type : TYPE){
               if(StringUtils.endWithIgnoreCase(uploadFile.getOriginalFilename(), type)){
                   isLegal = rue;
                   break;
               }
           }
           if(!isLegal){
               file.setStatus("error");
               return file;
           }
           //上传到阿里云
           //文件新路径
           String fileName = uploadFile.getOriginalFilename();
           String filePath = getFilePath(fileName);
           try{
               ossClient.putObject(aliyunConfig.getBucketName(), filePath, new ByteArrayInputStream(uploadFile.getBytes()));
           }catch(Exection e){
               e.printStackTrace();
               //上传失败
               file.setStatus("error");
               return file;
           }
           //上传成功
           file.setStatus("done");
           file.setName(this.aliyunConfig.getUrlPrefix() + filePath);
           file.setUid(Stirng.valueOf(System.currentTimeMillis()));
           return file;
       }
       //取得文件路径
       private String getFilePath(String sourceFileName){
           DateTime dateTime = new DateTime();
           return "images/" + dateTime.toString("yyyy") + "/" + dateTime.toString("MM") + "/" + dateTime.toString("dd") + "/"
               + System.currentTimeMilllis() + RandomUtils.nextInt(100, 9999) + "."
               + StringUtiles.substringAfterLast(sourceFileName, ".");
       }
   }
   ```

   

## MinIo

### 简介

- 轻量、去中心化（节点对等）、分片存储
- 使用==纠删码技术==来保护数据

### 使用

1. 依赖

   ```xml
   <!--方式一-->
   <dependency>
       <groupId>com.pig4cloud.plugin</groupId>
       <artifactId>oss-spring-boot-starter</artifactId>
       <version>1.0.5</version>
   </dependency>
   <!--方式二-->
   <dependency>
       <groupId>io.minio</groupId>
       <artifactId>minio</artifactId>
       <version>8.4.3</version>
   </dependency>
   <dependency>
       <groupId>com.squareup.okhttp3</groupId>
       <artifactId>okhttp</artifactId>
       <version>4.8.1</version>
   </dependency>
   ```

2. 文件上传

   ```java
   public R uploadFile(MultipartFile file) {
       //检查是否是可执行文件，如果是则不通过
       
   }
   ```

   

## NFS

## GFS（GoogleFs）

### 简介

- 可扩展、大型、分布式，比较冗余，但是可靠性高
- 有容错性、高性能
- 采用主从结构
- 分块存储

## HDFS

### 简介

- 类似GFS