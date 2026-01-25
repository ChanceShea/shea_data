# LLM API
从硅基流动官网注册账号并获取API key，创建.env文件后保存API key到.env文件中
```
OPENAI_API_KEY="sk-..."
```
## 调用API
```python
import os  
from dotenv import load_dotenv  
from openai import OpenAI  
  
load_dotenv()  
  
client = OpenAI(  
    api_key=os.environ.get("OPENAI_API_KEY"),  
    base_url="https://api.siliconflow.cn"  
)  
completion = client.chat.completions.create(  
    model="deepseek-ai/DeepSeek-V3.1",  
    messages=[  
        {"role": "system", "content": "You are a helpful assistant."},  
        {"role": "user", "content": "Hello!"}  
    ]  
)  
  
print(completion)
```
调用API会返回一个ChatCompletion对象，其中包括了回答文本、创建时间、id等属性
```
ChatCompletion(id='019bf33d0b1a7eafefdc165adcb41345', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='Hello! How can I assist you today? 😊', refusal=None, role='assistant', annotations=None, audio=None, function_call=None, tool_calls=None))], created=1769312422, model='deepseek-ai/DeepSeek-V3.1', object='chat.completion', service_tier=None, system_fingerprint='', usage=CompletionUsage(completion_tokens=11, prompt_tokens=15, total_tokens=26, completion_tokens_details=CompletionTokensDetails(accepted_prediction_tokens=None, audio_tokens=None, reasoning_tokens=0, rejected_prediction_tokens=None), prompt_tokens_details=None))
```
completion.choices[0].message.content 就是我们需要的AI回复
**参数**
model：调用的模型
messages：即prompt，ChatCompletion的messages需要传入一个列表，列表中包括了多个不同角色的prompt(system,user)
temperature：即控制模型回复随机性的系数
max_tokens：模型输出的最大token数

---
OenAI提供了充分的自定义空间，支持我们自定义prompt来提升模型回答效果
```python
import os  
  
from dotenv import load_dotenv  
from openai import OpenAI  
  
load_dotenv()  
  
client = OpenAI(  
    api_key=os.environ.get("OPENAI_API_KEY"),  
    base_url="https://api.siliconflow.cn"  
)  
  
def gen_messages(prompt):  
    """  
    :param prompt:    :return:  
    """    messages = [  
        {"role":"user","content":prompt}  
    ]  
    return messages  
def get_completion(prompt,temperature,model="deepseek-ai/DeepSeek-V3.1"):  
    response = client.chat.completions.create(  
        model=model,  
        messages=gen_messages(prompt),  
        temperature=temperature,  
    )  
    if len(response.choices)>0:  
        return response.choices[0].message.content  
    return "generate answer error"
```
# 向量数据库
## 向量
在NLP中，词向量是一种以单词为单位将每个单词转化为实数向量的技术。这些实数向量可以更好被理解和处理。相似相关的对象在向量空间中的距离应该更近
词向量实际上是将单词转化为固定的静态向量，虽然可以在一定程度上捕捉并表达文本中的语义信息，但忽略了单词在不同语境中的意思会受到影响这一现实，因此在RAG应用中使用的向量技术一般为通用文本向量，该技术可以对一定范围内任意长度的文本进行向量化，与词向量不同的是，向量化的单位不是单词而是输入的文本，输出的向量也会捕捉更多的语义信息
## Embedding API
## 调用API
