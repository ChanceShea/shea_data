# AI大模型
AI大模型是指通过海量数据和超大规模参数构建的深度神经网络系统
## 基本概念
### Prompt
Prompt最初是NLP（自然语言处理）研究者为下游任务设计出来的一种任务专属的输入模版，类似于一种ren'wu
## 本地部署AI大模型
### 部署Ollama
Ollama是一个开源框架，专为在本地机器上便捷部署和运行大语言模型（LLM）而设计

虚拟机内存>4G，CPU数量>=4个
```
mkdir -p /usr/local/ollama
docker run -d -p 11434:11434  --cpus=4  --memory=4g --memory=4g -v /usr/local/ollama:/root/.ollama --name ollama ollama/ollama
```
下载并运行模型
```
docker exec -it ollama ollama run deepseek-r1:1.5b
```
按照上述方法即可在本地部署deepseek-r1模型
## Spring AI
父项目中导入以下项目
```xml
<properties>    
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
    <!-- Spring AI -->   
    <spring-ai.version>1.0.0 </spring-ai.version>  
    <!-- Spring AI Alibaba -->   
    <spring-ai-alibaba.version>1.0.0.2</spring-ai-alibaba.version>  
    <!-- Spring Boot -->    
    <spring-boot.version>3.4.5</spring-boot.version>  
 </properties>
 <!--_ _依赖Jar__包版本管理 -->  
 <dependencyManagement>    
	 <dependencies>  
		 <dependency>  
	        <groupId>org.springframework.boot</groupId>  
	        <artifactId>spring-boot-dependencies</artifactId>  
	        <version>${spring-boot.version}</version>  
	        <type>pom</type>  
	        <scope>import</scope>  
	     </dependency>  
		<dependency>  
	        <groupId>org.springframework.ai</groupId>  
	        <artifactId>spring-ai-bom</artifactId>  
	        <version>${spring-ai.version}</version>  
	        <type>pom</type>  
	        <scope>import</scope>  
	    </dependency>  
        <dependency>  
	        <groupId>com.alibaba.cloud.ai</groupId>  
	        <artifactId>spring-ai-alibaba-bom</artifactId>  
	        <version>${spring-ai-alibaba.version}</version>
			<type>pom</type>  
	        <scope>import</scope>  
        </dependency>  
    </dependencies>  
</dependencyManagement>
```
1. 在子项目导入依赖（本地模型接入）
```xml
<dependency>  
	<groupId>org.springframework.ai</groupId>  
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
	<version>1.0.0-M6</version>  
</dependency>
```
核心配置 application.yml
```yml
server:
  port: 9091
spring:
  application:
    name: ollama-ai
    ai:
      ollama:
      base-url: http://192.168.100.104:11434
      chat:
        model: deepseek-r1:1.5b
```
2. 百炼平台AI模型接入
```xml
 <dependency>  
    <groupId>com.alibaba.cloud.ai</groupId>  
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>  
  </dependency>
```
核心配置application.yml
```yml
server:
  port: 9092
spring:
  application:
    name: alibaba-ai
    ai:
      dashscope:
        api-key: ${ALIBABA_API_KEY}
        #api-base-url: https://api.dashscope.com/api/v1 #可以省略
        chat:
          options:
            model: deepseek-r1 #模型可自行选择
```
### 对话客户端
Spring AI Alibaba的ChatClient是用于与AI模型交互的核心抽象接口，支持同步和响应式编程模型
**创建模型**
ChatClient是使用ChatClient.Builder对象。通过ChatClient.Builder对象可以获得其实例
1. 使用自动配置的ChatClient.Builder
```java
@RestController
@RequestMapping("/ai/chat")
public class ChatController{
	private final ChatClient chatClient;
	// 构造方法注入
	public ChatController(ChatClient.Builder builder){
		this.chatClient = builder.build();
	}
}
```
2. 编程式创建ChatClient
```java
@Configuration
public class ChatClientConfig{
	// 通过此方式可以创建多个ChatClient实例
	@Bean
	public ChatClient chatClient(ChatModel model){
		return ChatClient.builder(model).build();
	}
}
```
