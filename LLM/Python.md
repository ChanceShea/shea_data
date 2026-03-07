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
### 调用API
```python
from dotenv import load_dotenv  
import os  
from openai import OpenAI  
load_dotenv()  
  
def openai_embedding(text:str,model:str=None):  
  
    api_key = os.getenv("OPENAI_API_KEY")  
    if not api_key:  
        print("Please set an OpenAI API key")  
        return  
    client = OpenAI(api_key=api_key,base_url="https://api.siliconflow.cn/v1")  
  
    if model is None:  
        model = "Qwen/Qwen3-Embedding-4B"  
  
    resp = client.embeddings.create(  
        input=text,  
        model=model,  
    )  
    return resp  
  
response = openai_embedding(text="今天天气晴朗")  
  
print(response)
```
输出结果
```
CreateEmbeddingResponse(data=[Embedding(embedding=[-9.541741019347683e-05, -0.0012793000787496567, 0.04636579379439354,... 0.006643879227340221, -0.0159453097730875], index=0, object='embedding')], model='Qwen/Qwen3-Embedding-4B', object='list', usage=Usage(prompt_tokens=5, total_tokens=5, completion_tokens=0))
```
## 数据处理
### 数据读取
对于PDF文档，我们可以使用LangChain的PyMuPDFLoader来读取知识库的PDF文件。PyMuPDFLoader是PDF解析器中速度最快的一种，结果会包含PDF及其页面的详细元数据，并且每页返回一个文档
```python
from langchain_community.document_loaders import PyMuPDFLoader  
  
# 创建一个 PyMuPDFLoader Class 实例，输入为待加载的 pdf 文档路径  
loader = PyMuPDFLoader("pumpkin_book.pdf")  
# 调用 PyMuPDFLoader Class 的函数 load 对 pdf 文件进行加载  
pdf_pages = loader.load()  
  
print(f"再入后的变量类型为:{type(pdf_pages)}，",f"该PDF一共包含{len(pdf_pages)}页")
```
page中每一个元素为一个文档，文档变量类型包含两个属性
- page_content，包含该文档的内容
- meta_data，文档相关的描述性信息
```python
pdf_page = pdf_pages[1]  
print(f"每一个元素的类型：{type(pdf_page)}.",  
    f"该文档的描述性数据：{pdf_page.metadata}",  
    f"查看该文档的内容:\n{pdf_page.page_content}",  
    sep="\n------\n")
```
```
每一个元素的类型：<class 'langchain_core.documents.base.Document'>.
------
该文档的描述性数据：{'producer': 'xdvipdfmx (20200315)', 'creator': 'LaTeX with hyperref', 'creationdate': '2023-11-17T15:20:45+00:00', 'source': 'pumpkin_book.pdf', 'file_path': 'pumpkin_book.pdf', 'total_pages': 196, 'format': 'PDF 1.5', 'title': '', 'author': '', 'subject': '', 'keywords': '', 'moddate': '', 'trapped': '', 'modDate': '', 'creationDate': "D:20231117152045-00'00'", 'page': 1}
------
查看该文档的内容:
前言
“周志华老师的《机器学习》（西瓜书）是机器学习领域的经典入门教材之一，周老师为了使尽可能多的读
者通过西瓜书对机器学习有所了解, 所以在书中对部分公式的推导细节没有详述，但是这对那些想深究公式推
导细节的读者来说可能“不太友好”，本书旨在对西瓜书里比较难理解的公式加以解析，以及对部分公式补充
具体的推导细节。”
读到这里，大家可能会疑问为啥前面这段话加了引号，因为这只是我们最初的遐想，后来我们了解到，周
老师之所以省去这些推导细节的真实原因是，他本尊认为“理工科数学基础扎实点的大二下学生应该对西瓜书
中的推导细节无困难吧，要点在书里都有了，略去的细节应能脑补或做练习”。所以...... 本南瓜书只能算是我
等数学渣渣在自学的时候记下来的笔记，希望能够帮助大家都成为一名合格的“理工科数学基础扎实点的大二
下学生”。
使用说明
• 南瓜书的所有内容都是以西瓜书的内容为前置知识进行表述的，所以南瓜书的最佳使用方法是以西瓜书
为主线，遇到自己推导不出来或者看不懂的公式时再来查阅南瓜书；
• 对于初学机器学习的小白，西瓜书第1 章和第2 章的公式强烈不建议深究，简单过一下即可，等你学得
有点飘的时候再回来啃都来得及；
• 每个公式的解析和推导我们都力(zhi) 争(neng) 以本科数学基础的视角进行讲解，所以超纲的数学知识
我们通常都会以附录和参考文献的形式给出，感兴趣的同学可以继续沿着我们给的资料进行深入学习；
• 若南瓜书里没有你想要查阅的公式，或者你发现南瓜书哪个地方有错误，请毫不犹豫地去我们GitHub 的
Issues（地址：https://github.com/datawhalechina/pumpkin-book/issues）进行反馈，在对应版块
提交你希望补充的公式编号或者勘误信息，我们通常会在24 小时以内给您回复，超过24 小时未回复的
话可以微信联系我们（微信号：at-Sm1les）；
配套视频教程：https://www.bilibili.com/video/BV1Mh411e7VU
在线阅读地址：https://datawhalechina.github.io/pumpkin-book（仅供第1 版）
最新版PDF 获取地址：https://github.com/datawhalechina/pumpkin-book/releases
编委会
主编：Sm1les、archwalker、jbb0523
编委：juxiao、Majingmin、MrBigFan、shanry、Ye980226
封面设计：构思-Sm1les、创作-林王茂盛
致谢
特别感谢awyd234、feijuan、Ggmatch、Heitao5200、huaqing89、LongJH、LilRachel、LeoLRH、Nono17、
spareribs、sunchaothu、StevenLzq 在最早期的时候对南瓜书所做的贡献。
扫描下方二维码，然后回复关键词“南瓜书”，即可加入“南瓜书读者交流群”
版权声明
本作品采用知识共享署名-非商业性使用-相同方式共享4.0 国际许可协议进行许可。
```
对于md文档，可以使用几乎完全一致的方式读取
```python
from langchain_community.document_loaders.markdown import UnstructuredMarkdownLoader

loader = UnstructuredMarkdownLoader("../data_base/knowledge_db/prompt_engineering/1. 简介 Introduction.md")
md_pages = loader.load()
```
读取的对象和PDF文档读取出来的是完全一致的
### 数据清洗
我们希望知识库的数据精良是有序的、优质的、精简的，因此我们要删除低质量的、影响理解的文本数据
上文中读取的pdf文件不仅将一句话按照原文的分行添加了换行符`\n`，也在原本两个符号中间插入了`\n`，我们可以使用正则表达式匹配并删除掉`\n`
同时，数据中还有不少的`·`和空格，同样可以使用正则表达式去掉
```python
import re  
pattern = re.compile(r'[^\u4e00-\u9fff](\n)[^\u4e00-\u9fff]', re.DOTALL)  
pdf_page.page_content = re.sub(pattern,  
                               lambda match:match.group(0).replace('\n',''),  
                               pdf_page.page_content)  
pdf_page.page_content = pdf_page.page_content.replace('•', '')  
pdf_page.page_content = pdf_page.page_content.replace(' ', '')  
print(pdf_page.page_content)
```
```
“周志华老师的《机器学习》（西瓜书）是机器学习领域的经典入门教材之一，周老师为了使尽可能多的读
者通过西瓜书对机器学习有所了解,所以在书中对部分公式的推导细节没有详述，但是这对那些想深究公式推
导细节的读者来说可能“不太友好”，本书旨在对西瓜书里比较难理解的公式加以解析，以及对部分公式补充
具体的推导细节。”
读到这里，大家可能会疑问为啥前面这段话加了引号，因为这只是我们最初的遐想，后来我们了解到，周
老师之所以省去这些推导细节的真实原因是，他本尊认为“理工科数学基础扎实点的大二下学生应该对西瓜书
中的推导细节无困难吧，要点在书里都有了，略去的细节应能脑补或做练习”。所以......本南瓜书只能算是我
等数学渣渣在自学的时候记下来的笔记，希望能够帮助大家都成为一名合格的“理工科数学基础扎实点的大二
下学生”。
使用说明
南瓜书的所有内容都是以西瓜书的内容为前置知识进行表述的，所以南瓜书的最佳使用方法是以西瓜书
为主线，遇到自己推导不出来或者看不懂的公式时再来查阅南瓜书；对于初学机器学习的小白，西瓜书第1章和第2章的公式强烈不建议深究，简单过一下即可，等你学得
有点飘的时候再回来啃都来得及；每个公式的解析和推导我们都力(zhi)争(neng)以本科数学基础的视角进行讲解，所以超纲的数学知识
我们通常都会以附录和参考文献的形式给出，感兴趣的同学可以继续沿着我们给的资料进行深入学习；若南瓜书里没有你想要查阅的公式，或者你发现南瓜书哪个地方有错误，请毫不犹豫地去我们GitHub的
Issues（地址：https://github.com/datawhalechina/pumpkin-book/issues）进行反馈，在对应版块
提交你希望补充的公式编号或者勘误信息，我们通常会在24小时以内给您回复，超过24小时未回复的
话可以微信联系我们（微信号：at-Sm1les）；
配套视频教程：https://www.bilibili.com/video/BV1Mh411e7VU
在线阅读地址：https://datawhalechina.github.io/pumpkin-book（仅供第1版）
最新版PDF获取地址：https://github.com/datawhalechina/pumpkin-book/releases
编委会
主编：Sm1les、archwalker、jbb0523
编委：juxiao、Majingmin、MrBigFan、shanry、Ye980226
封面设计：构思-Sm1les、创作-林王茂盛
致谢
特别感谢awyd234、feijuan、Ggmatch、Heitao5200、huaqing89、LongJH、LilRachel、LeoLRH、Nono17、spareribs、sunchaothu、StevenLzq在最早期的时候对南瓜书所做的贡献。
扫描下方二维码，然后回复关键词“南瓜书”，即可加入“南瓜书读者交流群”
版权声明
本作品采用知识共享署名-非商业性使用-相同方式共享4.0国际许可协议进行许可。
```
### 文档分割
由于单个文档的长度往往会超出模型支持的上下文，导致检索得到的知识太长超出模型的处理能力，因此，在构建向量知识库的过程中，我们往往需要对文档进行分割，将单个文档按长度或固定的规则分割成若干个chunk，然后将每个chunk转化为词向量，存储到向量数据库中
在检索时，会以chunk作为检索的元单位，也就是每次检索得到k个chunk作为模型可以参考来回答用户问题的知识，这个k是我们可以自定义的
Langchain中文本分割器都根据chunk_size（块大小）和chunk_overlap（块与块之间的重叠大小）进行分割
- chunk_size指每个块包含的字符或token数量
- chunk_overlap指两个块之间共享的字符数量，用于保持上下文的连贯性，避免分割丢失上下文信息
Langchain提供了多种文档分割方式，区别在于怎么确定块与块之间的边界、块有哪些字符/token组成、以及如何测量块大小
- RecursiveCharacterTextSplitter(): 按字符串分割文本，递归地尝试按不同的分隔符进行分割文本。
- CharacterTextSplitter(): 按字符来分割文本。
- MarkdownHeaderTextSplitter(): 基于指定的标题来分割markdown 文件。
- TokenTextSplitter(): 按token来分割文本。
- SentenceTransformersTokenTextSplitter(): 按token来分割文本
- Language(): 用于 CPP、Python、Ruby、Markdown 等。
- NLTKTextSplitter(): 使用 NLTK（自然语言工具包）按句子分割文本。
- SpacyTextSplitter(): 使用 Spacy按句子的切割文本。
```python
"""  
* RecursiveCharacterTextSplitter 递归字符文本分割  
RecursiveCharacterTextSplitter 将按不同的字符递归地分割(按照这个优先级["\n\n", "\n", " ", ""])，  
    这样就能尽量把所有和语义相关的内容尽可能长时间地保留在同一位置  
RecursiveCharacterTextSplitter需要关注的是4个参数：  
  
* separators - 分隔符字符串数组  
* chunk_size - 每个文档的字符数量限制  
* chunk_overlap - 两份文档重叠区域的长度  
* length_function - 长度计算函数  
"""  
from langchain_text_splitters import RecursiveCharacterTextSplitter  
# 知识库中单段文本长度  
CHUNK_SIZE = 500  
# 知识库中相邻文本重合长度  
OVERLAP_SIZE = 50  
text_splitter = RecursiveCharacterTextSplitter(  
    chunk_size=CHUNK_SIZE,  
    chunk_overlap=OVERLAP_SIZE,  
)  
split = text_splitter.split_text(pdf_page.page_content[0:1000])  
print(split)
```
```
['前言\n“周志华老师的《机器学习》（西瓜书）是机器学习领域的经典入门教材之一，周老师为了使尽可能多的读\n者通过西瓜书对机器学习有所了解,所以在书中对部分公式的推导细节没有详述，但是这对那些想深究公式推\n导细节的读者来说可能“不太友好”，本书旨在对西瓜书里比较难理解的公式加以解析，以及对部分公式补充\n具体的推导细节。”\n读到这里，大家可能会疑问为啥前面这段话加了引号，因为这只是我们最初的遐想，后来我们了解到，周\n老师之所以省去这些推导细节的真实原因是，他本尊认为“理工科数学基础扎实点的大二下学生应该对西瓜书\n中的推导细节无困难吧，要点在书里都有了，略去的细节应能脑补或做练习”。所以......本南瓜书只能算是我\n等数学渣渣在自学的时候记下来的笔记，希望能够帮助大家都成为一名合格的“理工科数学基础扎实点的大二\n下学生”。\n使用说明\n南瓜书的所有内容都是以西瓜书的内容为前置知识进行表述的，所以南瓜书的最佳使用方法是以西瓜书\n为主线，遇到自己推导不出来或者看不懂的公式时再来查阅南瓜书；对于初学机器学习的小白，西瓜书第1章和第2章的公式强烈不建议深究，简单过一下即可，等你学得', '有点飘的时候再回来啃都来得及；每个公式的解析和推导我们都力(zhi)争(neng)以本科数学基础的视角进行讲解，所以超纲的数学知识\n我们通常都会以附录和参考文献的形式给出，感兴趣的同学可以继续沿着我们给的资料进行深入学习；若南瓜书里没有你想要查阅的公式，或者你发现南瓜书哪个地方有错误，请毫不犹豫地去我们GitHub的\nIssues（地址：https://github.com/datawhalechina/pumpkin-book/issues）进行反馈，在对应版块\n提交你希望补充的公式编号或者勘误信息，我们通常会在24小时以内给您回复，超过24小时未回复的\n话可以微信联系我们（微信号：at-Sm1les）；\n配套视频教程：https://www.bilibili.com/video/BV1Mh411e7VU\n在线阅读地址：https://datawhalechina.github.io/pumpkin-book（仅供第1版）\n最新版PDF获取地址：https://github.com/datawhalechina/pumpkin-book/releases\n编委会', '编委会\n主编：Sm1les、archwalker']
```
```python
split_docs = text_splitter.split_documents(pdf_pages)  
print(f"切分后的文件数量{len(split_docs)}")  
print(f"切分后的字符数：{sum([len(doc.page_content) for doc in split_docs])}")
```
```
切分后的文件数量712
切分后的字符数：305676
```
