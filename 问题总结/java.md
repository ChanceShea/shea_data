# Lombok的注解无效

删除原有的lombok依赖

添加下列依赖，并在plugins中添加lombok的版本信息

```xml
 <dependency>
   <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
   <version>1.18.32</version>   ------添加版本信息
</dependency>


<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version> -----指定version信息
  <configuration>
     <annotationProcessorPaths>
        <path>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>1.18.32</version> ------指定version信息
        </path>
     </annotationProcessorPaths>
  </configuration>
</plugin>
```

# MybatisPlus兼容性问题

springboot 3.x版本以上需使用下列依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>  
    <version>3.5.7</version>   ------artifactId中有个3
</dependency>
```

# 后端请求uri变成/error

检验前后端数据格式是否一致，前端是否有多出来的字段

# 发送邮件报错

**异常信息为Failed messages: jakarta.mail.MessagingException: IOException while sending message;
  nested exception is:
    java.io.IOException: No MimeMessage content**

发送的文本格式为html格式，没有使用适当的Mime处理，从而导致编码错误

**在EmailUtil中添加以下代码，并通过EmailUtil发送请求**

```java
public static MimeMessage sendHtmlEmail(String recipient, String subject, String htmlContent) throws MessagingException {
        MimeMessage message = new MimeMessage(getSession());
        message.setFrom(new InternetAddress(USER));
        message.setRecipient(Message.RecipientType.TO, new InternetAddress(recipient));
        message.setSubject(subject);
        MimeBodyPart mimeBodyPart = new MimeBodyPart();
        mimeBodyPart.setContent(htmlContent, "text/html;charset=utf-8"); // 设置为"application/octet-stream"或"text/plain;charset=utf-8"
        Multipart multipart = new MimeMultipart();
        multipart.addBodyPart(mimeBodyPart);
        message.setContent(multipart);
        return message;
    }

    private static Session getSession() throws MessagingException {
        return Session.getInstance(
                new Properties()
        );
    }
    }
```

# 后端无法获取到token

```java
@Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 所有接口
                .allowCredentials(true) // 是否发送 Cookie
                .allowedOriginPatterns("*") // 支持域
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH") // 支持方法，添加 OPTIONS
                .allowedHeaders("*") // 允许所有请求头
                .exposedHeaders("*") // 暴露所有响应头
                .maxAge(3600); // 预检请求缓存时间(秒)
    }
```

检查跨域配置中是否有**allowedHeaders("*")** 函数，如果没有则会把Authorization请求头拦截，导致后端无法获取到token

# 跨域问题

配置了CorsConfig，但是前后端联调时还是会有跨域问题

```java
// 放行OPTIONS预检请求
if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {
    return true;
}
```

浏览器在发送跨域请求时，会先发送一个OPTIONS预检请求
LoginInterceptor 拦截了所有请求（包括OPTIONS），要求携带token
但预检请求不会携带自定义header（token），导致拦截器抛出异常
