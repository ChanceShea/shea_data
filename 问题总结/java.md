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

## MCP服务启动超时
```
[org/springframework/ai/autoconfigure/mcp/client/McpClientAutoConfiguration.class]: Failed to instantiate [java.util.List]: Factory method 'mcpSyncClients' threw exception with message: java.util.concurrent.TimeoutException: Did not observe any item or terminal signal within 20000ms in 'Mono.create ⇢ at io.modelcontextprotocol.spec.DefaultMcpSession.sendRequest(DefaultMcpSession.java:228)
```
```json
{  
  "mcpServers": {  
    "amap-maps": {  
      "command": "npx.cmd",  
      "args": [  
        "-y",  
        "@amap/amap-maps-mcp-server",  
        "-Dspring.profiles.active=stdio"  // 新增参数就能解决超时问题
      ],  
      "env": {  
        "AMAP_MAPS_API_KEY": "f8d6f9268d52ef45a406f5121cd944f2"  
      }  
    }  
  }  
}
```
## sse流式输出md格式的数据，换行符会被吞掉
SSE 是基于文本行的协议，它用 \n（或 \r\n）作为字段的分隔符。一条消息的格式是：
```text
data:这是内容\n
\n
```
data: 后面跟内容，然后一个 \n 表示这个字段结束，再一个空行 \n 表示整条消息结束。
问题就在这儿：如果你要传的内容本身就包含 \n，直接塞进 data: 后面，这个 \n 会被 SSE 协议当成字段分隔符处理掉，而不是当成内容的一部分。
换句话说，后端如果这么干：
```text
data:第一行\n第二行\n
```
SSE 在解析的时候，会把它理解成"data 字段是『第一行』，然后字段结束了"，后面的第二行要么被忽略，要么被错误处理。内容里的换行，就这么被协议吃掉了。
我们后端当时就是直接把大模型输出的文本（带 Markdown 换行的）塞进 data: 转发出来，换行符在传输过程中全部丢失。前端收到的，自然是一堆没有换行的碎片。
**解决**
可以在后端输出data之前使用json包装一下，包装之后，换行符就会被解析成'\n'就不会被sse输出吞掉
```text
event:message
data:{"content":"## 核心功能\n\n这是正文"}
```
AI流式输出时，可以按照以下方法包装
```java
private final ObjectMapper objectMapper = new ObjectMapper();
public Flux<ServerSentEvent<String>> interviewChatRag(String userMessage,String chatId) {  
    return codeApp.doChatWithRag(userMessage, chatId)  
            .map(chunk -> ServerSentEvent.<String>builder().data(wrapContent(chunk)).build());  
}  
  
/**  
 * 将内容包装为 JSON 字符串，避免换行符破坏 SSE 解析  
 */  
private String wrapContent(String content) {  
    try {  
        return objectMapper.writeValueAsString(Map.of("content", content));  
    } catch (Exception e) {  
        return "{\"content\":\"\"}";  
    }  
}
```
对于前端，在解析时，只需要通过json.parse将数据进行解析即可
# 