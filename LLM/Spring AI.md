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