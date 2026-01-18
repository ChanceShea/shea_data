# AI大模型
AI大模型是指通过海量数据和超大规模参数构建的深度神经网络系统
## 本地部署AI大模型
### Ollama
Ollama是一个开源框架，专为在本地机器上便捷部署和运行大语言模型（LLM）而设计
#### 部署Ollama
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
            model: qwen-max
```