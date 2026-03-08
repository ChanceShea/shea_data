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
### 搭建向量数据库
首先我们需要获取到所有的文档，并对文档做一些处理
```python
import os  
  
from dotenv import load_dotenv, find_dotenv  
from langchain_community.document_loaders import PyMuPDFLoader, UnstructuredMarkdownLoader  
  
_ = load_dotenv(find_dotenv())  
  
file_paths = []  
folder_path = "../resources"  
for root,dirs,files in os.walk(folder_path):  
    for file in files:  
        file_path = os.path.join(root,file)  
        file_paths.append(file_path)  
print(file_paths)  
# 遍历文件路径并把实例化的loader存到到loaders里  
loaders = []  
for file_path in file_paths:  
    file_type = file_path.split(".")[-1]  
    if file_type == "pdf":  
        loaders.append(PyMuPDFLoader(file_path))  
    elif file_type == "md":  
        loaders.append(UnstructuredMarkdownLoader(file_path))  
# 下载文件并存储到texts列表  
texts = []  
for loader in loaders:  
    texts.extend(loader.load())  
  
text = texts[0]  
print(f"每一个元素的类型：{type(text)}.",  
    f"该文档的描述性数据：{text.metadata}",  
    f"查看该文档的内容:\n{text.page_content[0:]}",  
    sep="\n------\n")
```
```
['../resources\\pumpkin_book.pdf', '../resources\\Redis.md', '../resources\\Java并发编程.md', '../resources\\MySQL.md']
每一个元素的类型：<class 'langchain_core.documents.base.Document'>.
------
该文档的描述性数据：{'producer': 'xdvipdfmx (20200315)', 'creator': 'LaTeX with hyperref', 'creationdate': '2023-11-17T15:20:45+00:00', 'source': '../resources\\pumpkin_book.pdf', 'file_path': '../resources\\pumpkin_book.pdf', 'total_pages': 196, 'format': 'PDF 1.5', 'title': '', 'author': '', 'subject': '', 'keywords': '', 'moddate': '', 'trapped': '', 'modDate': '', 'creationDate': "D:20231117152045-00'00'", 'page': 0}
------
查看该文档的内容:
本:2.0.0
发布日期:2023.11
南  ⽠  书
PUMPKIN
B  O  O  K
谢	睿 州 贾彬彬
```
获取到文档之后，需要对文档进行分割
```python
text_splitter = RecursiveCharacterTextSplitter(  
    chunk_size=500,  
    chunk_overlap=50  
)  
split_docs = text_splitter.split_documents(texts)
```
**构建Chroma向量数据库**
```python
from langchain_openai import OpenAIEmbeddings  
# 定义embeddings  
embedding = OpenAIEmbeddings(  
    api_key=os.environ.get("OPENAI_API_KEY"),  
    base_url="https://api.siliconflow.cn/v1",  
    model="Qwen/Qwen3-Embedding-0.6B"  
)  
# 定义持久化路径  
persist = "../db/vector_db/chroma"  
  
from langchain_community.vectorstores import Chroma  
vectordb = Chroma.from_documents(  
    documents = split_docs,  
    embedding = embedding,  
    persist_directory = persist,  
)  
  
print(f"向量数据库中存储的数量：{vectordb._collection.count()}")
```
```
向量数据库中存储的数量：1073
```
Chroma的相似度搜索使用的是余弦距离
$$
similarity=cos(A,B)=\frac{A*B}{||A||*||B||}=\frac{\sum_{i=1}^na_ib_i}{{\sqrt{\sum_{i=1}^na_i^2}}{\sqrt{\sum_{i=1}^n=b_i^2}}}
$$
其中$a_i$、$b_i$分别是向量A、B的分量
当需要数据库严谨的返回按余弦相似度排序的结果时
```python
question="介绍一下Redis缓存穿透问题"  
sim_docs=vectordb.similarity_search(question,k=3)  
print(f"检索到的内容数：{len(sim_docs)}")  
for i,sim_doc in enumerate(sim_docs):  
    print(f"检索到的第{i}个内容是：\n{sim_doc.page_content[:200]}",end="\n--------------\n")
```
```
检索到的内容数：3
检索到的第0个内容是：
四大特性(ACID)

A(Atomicity)：原子性，事务是原子的，其中的操作要么同时成功，要么同时失败 C(Consistency)：一致性，事务完成时，所有的数据保持一致的状态 I(Isolation)：隔离性，事务与事务之间是隔离的，互不影响 D(Durability)：持久性，事务一旦提交或回滚，数据就会持久化到磁盘中，数据是持久的

并发事务问题
--------------
检索到的第1个内容是：
业务刚上线的时候，最好提前把数据缓存起来，而不是等待用户来访问才开始构建缓存，这就叫缓存预热 针对Redis故障宕机而引发的缓存雪崩问题 有以下几种应对方式 - 服务熔断或请求限流机制：因为Redis故障宕机而导致缓存雪崩问题时，可以启动服务熔断机制，暂停业务应用对缓存服务的访问，直接返回错误，不用再访问数据库，从而降低对数据库的访问压力，等到Redis恢复正常，再允许业务访问缓存服务 但是服务熔
--------------
检索到的第2个内容是：
缓存击穿

缓存中某个热点数据过期了，而此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库就容易被高并发请求冲垮，这就是缓存击穿 解决缓存击穿，有以下两种方案 - 互斥锁：保证同一时间只有一个业务线程去更新缓存，其他请求要么等到锁释放后重新读取缓存，要么就返回空值或默认值 - 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据快要过期时，提前通知后台线程更新缓存并
--------------
```
由于分块策略以及使用的embedding模型参数较小，并且，只是用了相似度检索。因此可能检索出来的结果不是特别的正确
如果只考虑检索出内容的相关性，可能会导致内容过于单一，可能会丢失重要信息
最大边际相关性(MMR)可以帮助我们在保持相关性的同时，增加内容的丰富度
核心思想是在已经选择了一个相关性高的文档之后，再选择一个与已选文档相关性较低但是信息丰富的文档。这样可以在保持相关性的同时，增加内容多样性，避免过于单一的结果
```python
mmr_docs = vectordb.max_marginal_relevance_search(question,k=3)
for i, sim_doc in enumerate(mmr_docs): 
	print(f"MMR 检索到的第{i}个内容: \n{sim_doc.page_content[:200]}", end="\n--------------\n")
```
```
MMR 检索到的第0个内容: 
四大特性(ACID)

A(Atomicity)：原子性，事务是原子的，其中的操作要么同时成功，要么同时失败 C(Consistency)：一致性，事务完成时，所有的数据保持一致的状态 I(Isolation)：隔离性，事务与事务之间是隔离的，互不影响 D(Durability)：持久性，事务一旦提交或回滚，数据就会持久化到磁盘中，数据是持久的

并发事务问题
--------------
MMR 检索到的第1个内容: 
缓存击穿

缓存中某个热点数据过期了，而此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库就容易被高并发请求冲垮，这就是缓存击穿 解决缓存击穿，有以下两种方案 - 互斥锁：保证同一时间只有一个业务线程去更新缓存，其他请求要么等到锁释放后重新读取缓存，要么就返回空值或默认值 - 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据快要过期时，提前通知后台线程更新缓存并
--------------
MMR 检索到的第2个内容: 
synchronized

@Slf4j
public class Test {

    static boolean flag = true;

    public static final Object object = new Object();

    public static void main(String[] args) {

        new Thread(() ->
--------------
```
# 构建RAG应用
## 基于LangChain调用DeepSeek
LangChain提供了对于多种大模型的封装，基于LangChain的接口可以便捷地调用ChatGPT并将其集合在以LangChain为基础框架搭建的个人应用中
```python
import os  
from dotenv import load_dotenv  

load_dotenv()  
openai_api_key = os.environ.get("OPENAI_API_KEY")  
  
from langchain_openai import ChatOpenAI  
llm = ChatOpenAI(  
    temperature=0.0,  
    base_url="https://api.siliconflow.cn",  
    api_key=openai_api_key,  
    model="deepseek-ai/DeepSeek-V3.2"  
)  
print(llm)
```
```
profile={} client=<openai.resources.chat.completions.completions.Completions object at 0x000002774A684980> async_client=<openai.resources.chat.completions.completions.AsyncCompletions object at 0x000002774A685550> root_client=<openai.OpenAI object at 0x0000027749093A10> root_async_client=<openai.AsyncOpenAI object at 0x000002774A6852B0> model_name='deepseek-ai/DeepSeek-V3.2' temperature=0.0 model_kwargs={} openai_api_key=SecretStr('**********') openai_api_base='https://api.siliconflow.cn'
```
我们只看几个重要的结果变量
- model_name：所使用的模型，上述代码中使用的是deepseek-ai/DeepSeek-V3.2
- temperature：温度系数，值越小，大模型给出的结果越保守，值越大，大模型给出的结果越新奇，更富有想象力
- openai_api_key：OpenAI API Key，如果不使用环境变量设置，也可以在实例化时设置
接下来就可以使用我们选择的LLM
```python
output = llm.invoke("请介绍一下MySQL的MVCC")  
print(f"\n{output}")
```
```
content='好的，我来详细介绍一下 MySQL 的 MVCC。\n\n## 什么是 MVCC？\n\n**MVCC** 全称是 **Multi-Version Concurrency Control**，即**多版本并发控制**。它是一种用于提高数据库并发性能的机制，通过保存数据在某个时间点的快照，让读写操作可以并发执行而无需相互阻塞。\n\n## 核心思想\n\nMVCC 的核心思想是：**为每一行数据维护多个版本**。当一个事务读取数据时，它看到的是在其开始时间点已经提交的数据版本，而不是当前最新的数据。这样，读操作不会阻塞写操作，写操作也不会阻塞读操作。\n\n## MySQL 中 MVCC 的实现\n\n在 InnoDB 存储引擎中，MVCC 主要通过以下三个关键组件实现：\n\n### 1. 隐藏字段\nInnoDB 为每行数据添加了三个隐藏字段：...' additional_kwargs={'refusal': None} response_metadata={'token_usage': {'completion_tokens': 1100, 'prompt_tokens': 10, 'total_tokens': 1110, 'completion_tokens_details': {'accepted_prediction_tokens': None, 'audio_tokens': None, 'reasoning_tokens': 0, 'rejected_prediction_tokens': None}, 'prompt_tokens_details': None}, 'model_provider': 'openai', 'model_name': 'deepseek-ai/DeepSeek-V3.2', 'system_fingerprint': '', 'id': '019cc7479a83d72e1a0f1d9b7dbefbe9', 'finish_reason': 'stop', 'logprobs': None} id='lc_run--019cc747-9370-7081-a575-8346a89126c0-0' tool_calls=[] invalid_tool_calls=[] usage_metadata={'input_tokens': 10, 'output_tokens': 1100, 'total_tokens': 1110, 'input_token_details': {}, 'output_token_details': {'reasoning': 0}}
```
在开发大模型应用的时候，通常不会直接将用户的输入传递给LLM。通常，会将用户的输入添加到一个较大的文本中，称为提示模版，该文本提供有关当前特定任务的附加上下文
PromptTemplates是一种预定义的，可结构化的提示框架。可以将静态的提示文本和动态输入的变量结合，例如
```python
template = "你是一个翻译助手，可以帮助我将 {input_language} 翻译成 {output_language}."
human_template = "{text}"  
chat_prompt = ChatPromptTemplate([  
    ("system",template),  
    ("human",human_template),  
])  
text = "我带着比身体重的行李，\  
游入尼罗河底，\  
经过几道闪电 看到一堆光圈，\  
不确定是不是这里。\  
"  
invoke = chat_prompt.invoke({"input_language": "中文", "output_language": "英文", "text": text})  
print(invoke)
```
```
messages=[SystemMessage(content='你是一个翻译助手，可以帮助我将 中文 翻译成 英文.', additional_kwargs={}, response_metadata={}), HumanMessage(content='我带着比身体重的行李，游入尼罗河底，经过几道闪电 看到一堆光圈，不确定是不是这里。', additional_kwargs={}, response_metadata={})]
```
接下来调用llm和message来输出回答
```python
content="With luggage heavier than my body, I dive into the Nile's depths, past several flashes of lightning, and see a cluster of halos—unsure if this is the place." additional_kwargs={'refusal': None} response_metadata={'token_usage': {'completion_tokens': 37, 'prompt_tokens': 46, 'total_tokens': 83, 'completion_tokens_details': {'accepted_prediction_tokens': None, 'audio_tokens': None, 'reasoning_tokens': 0, 'rejected_prediction_tokens': None}, 'prompt_tokens_details': None}, 'model_provider': 'openai', 'model_name': 'deepseek-ai/DeepSeek-V3.2', 'system_fingerprint': '', 'id': '019cc7520d25d85bb259ffef3305b5ca', 'finish_reason': 'stop', 'logprobs': None} id='lc_run--019cc752-0620-72f1-9566-656ddb882efc-0' tool_calls=[] invalid_tool_calls=[] usage_metadata={'input_tokens': 46, 'output_tokens': 37, 'total_tokens': 83, 'input_token_details': {}, 'output_token_details': {'reasoning': 0}}
```
OutputParsers将语言模型的原始输出转换为可以在下游使用的格式。OutputParsers有几种主要类型
- 将LLM文本转换为结构化信息
- 将ChatMessage转换为字符串
- 将除消息之外的调用返回额外的信息转换为字符串
最后，将模型输出传递给output_parser，它是一个BaseOutputParser，这意味着它接受字符串或BaseMessage作为输入。StrOutputParser可以将任意输入转换为字符串
```python
from langchain_core.output_parsers import StrOutputParser  
  
output_parser = StrOutputParser()  
s = output_parser.invoke(output)  
print(s)
```
```
With luggage heavier than my body, I dive into the Nile's depths, past several flashes of lightning, and see a cluster of halos—I'm not sure if this is the place.
```
从上述结果可知，OutputParser成功将ChatMessage类型的输出解析成字符串
现在可以将这些组合成一条链。该链可以获取输入变量，将这些变量传递给提示模版以创建提示，将提示传递给语言模型，然后通过输出解析器传递输出。
```python
chain = chat_prompt | llm | output_parser  
invoke = chain.invoke({"input_language": "中文", "output_language": "英文", "text": text})  
print(invoke)
```
```text
I carry luggage heavier than my body, diving into the depths of the Nile. After several flashes of lightning, I see a cluster of light circles, unsure if this is the place.
```
上述这种链的使用，叫做LCEL语法
**LCEL(LangChain Expression Language)，LCEL是一种新的语法，是LangChain工具包的重要补充，使我们处理LangChain和代理更加简单便捷**
- LCEL提供了异步、批处理和流处理支持，使代码可以快速在不同服务器中移植
- LCEL拥有后备措施，解决LLM格式输出问题
- LCEL增加了LLM的并行性，提高了效率
- LCEL内置了日志记录，即使代理变得复杂，有助于理解复杂链条和代理的运行情况
```python
chain = prompt | model | output_parser
```
上述代码中，我们使用LCEL将不同的组件拼凑成一个链，在此链中，用户输入传递到提示模版，然后提示模版输出传递到模型，然后模型输出传递到输出解析器。|符号类似于Unix管道运算符，它将不同的组件链接在一起，将一个组件的输出作为下一个组件的输入
## 构建检索问答链
首先需要加载向量数据库
```python
import os  
  
from dotenv import load_dotenv  
from langchain_chroma import Chroma  
from langchain_openai import ChatOpenAI, OpenAIEmbeddings  
  
load_dotenv()  
openai_api_key = os.getenv("OPENAI_API_KEY")  
open_ai = ChatOpenAI(api_key=openai_api_key, base_url="https://api.siliconflow.cn", model="deepseek-ai/DeepSeek-V3.2")  
output = open_ai.invoke("你好，请你自我介绍一下")  
print(output)  
# 这里使用的嵌入模型要和构建向量库时使用的嵌入模型一样
embedding = OpenAIEmbeddings(api_key=openai_api_key, model="Qwen/Qwen3-Embedding-0.6B",  
                                 base_url="https://api.siliconflow.cn/v1", )  
persist_path = "../db/vector_db/chroma"  
chroma = Chroma(persist_directory=persist_path, embedding_function=embedding, )  
print(f"向量库中存储的数量：{chroma._collection.count()}")
```
测试一下加载的向量数据库，可以通过`as_retriever`方法把向量数据库构造成检索器。
接下来，可以使用LangChain的LCEL来构建workflow
```python
question = "什么是缓存穿透"  
retriever = chroma.as_retriever(  
    search_kwargs={"k":3}  
)  
docs = retriever.invoke(question)  
for i,doc in enumerate(docs):  
    print(f"第{i+1}个内容是：\n{doc.page_content}")
```
```text
第1个内容是：
缓存击穿

缓存中某个热点数据过期了，而此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库就容易被高并发请求冲垮，这就是缓存击穿 解决缓存击穿，有以下两种方案 - 互斥锁：保证同一时间只有一个业务线程去更新缓存，其他请求要么等到锁释放后重新读取缓存，要么就返回空值或默认值 - 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据快要过期时，提前通知后台线程更新缓存并重新设置过期时间

缓存穿透
第2个内容是：
缓存击穿

缓存中某个热点数据过期了，而此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库就容易被高并发请求冲垮，这就是缓存击穿 解决缓存击穿，有以下两种方案 - 互斥锁：保证同一时间只有一个业务线程去更新缓存，其他请求要么等到锁释放后重新读取缓存，要么就返回空值或默认值 - 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据快要过期时，提前通知后台线程更新缓存并重新设置过期时间

缓存穿透
第3个内容是：
Character 缓存的范围是 0~127

Integer 缓存的范围是 -128~127，最小值不能变，但是最大值可以通过调整虚拟机参数来改变

Boolean 缓存了 TRUE 和 FALSE

String 缓存池

String 缓存池中当String 对象为字面量时，会自动存入缓存池中，String 是新建对象时，则不会加入缓存池中，会放在堆内存里

String 还可以通过intern()方法，手动将字符串添加进缓存池中

public class Test {
```
### 创建检索链
可以使用LangChain的LCEL来构建workflow
```python
def combine_docs(docs):  
    return "\n\n".join(doc.page_content for doc in docs)  
  
combiner = RunnableLambda(combine_docs)  
retrieval_chain = retriever | combiner  
msg = retrieval_chain.invoke("mysql是什么")  
print(msg)
```
```text
1 1 1 2 2 2 3 5 5 4 3 3 5 4 4

可以看到，客户端B的insert语句生成的id不连续 当主库发生了这种情况，binlog面对t表的更新只会记录这两个客户端的insert语句，如果binlog_format=statment(binlog日志格式介绍看[[#binlog]])，记录的语句就是原始语句，记录的顺序要么先记客户端A的insert语句，要么先记客户端B的insert语句，这取决于哪个事务优先提交 但不论是哪种，这个binlog到从库去执行，这时候从库是按顺序执行语句的，只有当执行完一条SQL语句，才会继续执行下一条，因此，在从库上，不会发生两个客户端一起往t表中插入数据的场景，所以在从库上，执行了客户端B的insert语句，生成的结果里面，id都是连续的，这是就发生了主从库数据不一致的问题 要解决这个问题，binlog日志格式要设置为row，当设置binlog的格式为row时，就不再记录SQL语句，而是记录每一行数据变更前后的值，这样在binlog里面记录的是主库分配的自增值，到从库中执行的时候，主库的自增值是什么，从库的自增值就是什么

1 1 1 2 2 2 3 5 5 4 3 3 5 4 4

可以看到，客户端B的insert语句生成的id不连续 当主库发生了这种情况，binlog面对t表的更新只会记录这两个客户端的insert语句，如果binlog_format=statment(binlog日志格式介绍看[[#binlog]])，记录的语句就是原始语句，记录的顺序要么先记客户端A的insert语句，要么先记客户端B的insert语句，这取决于哪个事务优先提交 但不论是哪种，这个binlog到从库去执行，这时候从库是按顺序执行语句的，只有当执行完一条SQL语句，才会继续执行下一条，因此，在从库上，不会发生两个客户端一起往t表中插入数据的场景，所以在从库上，执行了客户端B的insert语句，生成的结果里面，id都是连续的，这是就发生了主从库数据不一致的问题 要解决这个问题，binlog日志格式要设置为row，当设置binlog的格式为row时，就不再记录SQL语句，而是记录每一行数据变更前后的值，这样在binlog里面记录的是主库分配的自增值，到从库中执行的时候，主库的自增值是什么，从库的自增值就是什么

select * from t where id >= (
    select id from t order by id limit 10000000,1
) order by id limit 10;

对于这条查询语句，会优先执行子查询，也就是在B+树上查询10000000+1条数据，并获取最后一条数据的主键id，然后在外面的查询中，直接通过where条件，获取大于等于子查询中最后一条数据主键id的数据，然后从这条数据开始往后查询10条数据 这个sql与第一个sql最大的区别就是，不再需要进行千万次的回表操作，分页查询里面的每一次查询都只在聚簇索引上进行，提高了性能 3. 覆盖索引：如果只需要查询少量的字段，我们可以直接在这些字段上建立一个联合索引，这样，在查询的时候，就可以直接从索引树上返回结果，就不需要进行回表操作了

可重复读隔离级别，为什么只是部分解决幻读
```
RunnableLambda是LangChain中的一个工具类，作用是把普通的Python函数包装成LangChain中的Runnable对象，从而可以加入LCEL的链式调用中
LCEL中要求所有的组成元素都是Runnable类型，前面的ChatModel、PromptTemplate等都是继承自Runnable类。上方的retrieval_chain是由检索器retriever及组合起combiner组成，由|符号串联，数据从左向右传递，即问题先被retriever检索得到检索结果，再被combiner进一步处理得到输出
### 构建检索问答链
```python
template = """  
使用以下上下文来回答最后的问题。如果你不知道答案，就说你不知道，不要试图编造答  
案。最多使用三句话。尽量使答案简明扼要。请你在回答的最后说“谢谢你的提问！”。  
{context}  
问题: {input}  
"""  
prompt = PromptTemplate(template=template)  
qa_chain = (  
    RunnableParallel({"context":retrieval_chain,"input":RunnablePassthrough()})  
    | prompt  
    | open_ai  
    | StrOutputParser()  
)  
question1 = "什么是南瓜书"  
question2 = "什么是Redis缓存雪崩"  
res1 = qa_chain.invoke(question1)  
print(f"question1的结果: \n{res1}\n")  
res2 = qa_chain.invoke(question2)  
print(f"question2的结果: \n{res2}")
```
```text
question1的结果: 
南瓜书是《机器学习公式详解》一书的别称，旨在解析周志华《机器学习》（西瓜书）中省略或难以理解的公式推导细节。它适合在学习西瓜书时，遇到公式推导困难时查阅，尤其适用于希望深入理解公式的读者。谢谢你的提问！

question2的结果: 
Redis缓存雪崩是指在某一时段，大量缓存数据同时过期失效或Redis服务宕机，导致所有请求直接涌向数据库，造成数据库瞬时压力过大甚至崩溃的现象。它可以由缓存层大规模失效或服务不可用引起。谢谢你的提问！
```
上面代码中把检索链当做子链作为prompt的context，再使用`RunnablePassthrough`存储用户的问题作为prompt的input。又因为这两个操作是并行的，所以使用`RunnableParallel`将它们并行运行
利用大模型自己回答
```python
s1 = open_ai.invoke(question1).content  
s2 = open_ai.invoke(question2).content  
print(s1)  
print(s2)
```
LLM 对于一些近几年的知识以及非常识性的专业问题，回答的并不是很好。而加上我们的本地知识，就可以帮助 LLM 做出更好的回答
### 向检索链添加聊天记录
我们实现了通过上传本地的知识文档，然后将他们保存到向量知识库，通过将查询问题与向量知识库的召回结果进行结合输入到LLM中，我们就到的了一个相比于直接让LLM回答要好得多的结果。而在与大模型交互时，上下文记忆存储也是一个很重要的问题，要让大模型记得前面问过什么和回答过什么，使得对话更具有连续性
我们可以通过ChatPromptTemplate将先前的对话嵌入到语言模型中，使其具有连续对话的功能。ChatPromptTemplate可以接收聊天消息历史记录，这些历史记录将在回答问题时和问题一起传给大模型，从而将它们添加到上下文中
```python
system_prompt = (  
    "你是一个问答任务的助手。"  
    "请使用检索到的上下文片段回答这个问题。"  
    "如果你不知道答案就说不知道。"  
    "请使用简洁的话语回答用户。不超过三句话。"  
    "\n\n"  
    "{context}")  
qa_prompt = ChatPromptTemplate(  
    [  
        ("system",system_prompt),  
        ("placeholder","{chat_history}"),  
        ("human","{input}")  
    ]  
)  
messages = qa_prompt.invoke({  
    "input":"南瓜书是什么？",  
    "chat_history":[],  
    "context":""  
})  
for msg in messages.messages:  
    print(msg.content)  
res = open_ai.invoke(messages)  
print(res.content)  
messages2 = qa_prompt.invoke({  
    "input": "你可以介绍一下他吗",  
    "chat_history": [  
        ("human", "西瓜书是什么？"),  
        ("ai","西瓜书是指周志华老师的《机器学习》一书，是机器学习领域的经典入门教材之一。")  
    ],  
    "context": ""}  
)  
messages = messages2  
for msg in messages.messages:  
    print(msg.content)  
res = open_ai.invoke(messages)  
print(res.content)
```
```text
你是一个问答任务的助手。请使用检索到的上下文片段回答这个问题。如果你不知道答案就说不知道。请使用简洁的话语回答用户。不超过三句话。


南瓜书是什么？
南瓜书是周志华教授《机器学习》教材的学习辅助读物，主要聚焦于教材中的重难点公式推导，帮助初学者更顺畅地学习。它由开源学习社区Datawhale开发，可在GitHub上免费获取。
你是一个问答任务的助手。请使用检索到的上下文片段回答这个问题。如果你不知道答案就说不知道。请使用简洁的话语回答用户。不超过三句话。


西瓜书是什么？
西瓜书是指周志华老师的《机器学习》一书，是机器学习领域的经典入门教材之一。
你可以介绍一下他吗
好的，周志华教授是中国计算机领域人工智能方向的著名学者，现任南京大学计算机系主任、人工智能学院院长。他编著的《机器学习》（即“西瓜书”）因其精炼的内容和生动的比喻，已成为中文机器学习领域最具影响力的入门教材之一。
```
### 带有信息压缩的检索链
因为在搭建问答链带有支持多轮对话的功能，所以与单论对话的问答链相比，会多面临向上方输出结果的问题，即用户最新的对话语义不全，在使用用户问题查询向量数据库时很难检索到相关信息。像上方的“你可以介绍一下他吗？”，实际上是“你可以介绍下周志华老师吗？”的意思。为了解决这个问题，将采取信息压缩的方式，让llm根据历史记录完善用户问题
```python
condense_question_system_template = (  
    "请根据聊天记录完善用户最新的问题，"  
    "如果用户最新的问题不需要完善则返回用户的问题"  
)  
condense_question_prompt = ChatPromptTemplate([  
    ("system",condense_question_system_template),  
    ("placeholder","{chat_history}"),  
    ("human","{input}")  
])  
  
retrieve_docs = RunnableBranch(  
    # 分支1：若聊天记录中没有chat_history，则直接使用用户问题查询向量数据库  
    (lambda x: not x.get("chat_history", False), (lambda x: x["input"]) | retriever,),  
    # 分支2：若聊天记录中有chat_history，则先让llm根据聊天记录完善问题再查询向量数据库  
    condense_question_prompt | open_ai | StrOutputParser() | retriever  
)  
  
def combine_docs(docs):  
    return "\n\n".join(doc.page_content for doc in docs["context"])  
qa_chain2 = (  
    RunnablePassthrough.assign(context=combine_docs)  
    | qa_prompt  
    | open_ai  
    | StrOutputParser()  
)  
qa_history_chain = RunnablePassthrough.assign(  
    context = (lambda x:x) | retrieve_docs  
).assign(answer=qa_chain2)  
# 不带聊天记录
res1 = qa_history_chain.invoke({ 
	"input": "西瓜书是什么？", 
	"chat_history": [] 
})
# 带聊天记录  
res2 = qa_history_chain.invoke({
	"input": "南瓜书跟它有什么关系？", 
	"chat_history": [
		("human", "西瓜书是什么？"), 
		(  "ai", "西瓜书是指周志华老师的《机器学习》一书，是机器学习领域的经典入门教材之一。"), 
]})  
print(res2)
```
**tips**
上面这段代码暂时有点问题，检索出来的context与问题几乎无关，但是得到的答案却和问题有关联
## 部署知识库助手
### Streamlit
Streamlit是一个用于快速创建数据应用程序的开源python库。它的设计目标是让数据科学家能够轻松地将数据分析和机器学习模型转化为具有交互性的web应用程序，无需深入了解web开发。和Flask/Django不同之处在于，它不需要去编写任何前端代码，只需要编写普通的Python模块，就可以在短时间内创建美观并具备高度交互性的界面，从而快速生成数据分析或机器学习的结果
Streamlit提供了一组简单而强大的基础模块，用于构建数据应用程序
- st.write()：最基本的模块之一，用于在应用程序中呈现文本、图像、表格等内容
- st.title()、st.header()、st.subheader()：这些模块用于添加标题、子标题和分组标题，以组织应用程序的布局
- st.text()、st.markdown()：用于添加文本内容，支持markdown语法
- st.image()：用于添加图像到应用程序中
- st.dataframe()：用于呈现Pandas数据框
- st.table()：用于呈现简单的数据表格
- st.pyplot()、st.altair_chart()、st.plotly_chart()：用于呈现Matplotlib、Altair或Plotly绘制的图表
- st.button()、st.checkbox()、st.radio()：用于添加按钮，复选框和单选按钮，以触发特定的操作
- st.selectbox()、st.multiselect()、st.slider()、st.text_input()：用于添加交互式小部件，允许用户在应用程序中进行选择、输入或滑动操作
### 构建应用程序
```python
import streamlit as st  
from dotenv import load_dotenv  
from langchain_openai import ChatOpenAI, OpenAIEmbeddings  
import os  
from langchain_core.output_parsers import StrOutputParser  
from langchain_core.prompts import ChatPromptTemplate  
from langchain_core.runnables import RunnableBranch, RunnablePassthrough, RunnableLambda  
from langchain_community.vectorstores import Chroma
load_dotenv()
```
构建一个检索器
```python
def get_retriever():  
    embedding = OpenAIEmbeddings(  
        api_key=os.getenv("OPENAI_API_KEY"),  
        base_url="https://api.siliconflow.cn/v1",  
        model="Qwen/Qwen3-Embedding-8B",  
    )  
    persist_directory = "../db/vector_db/chroma"  
    vectordb = Chroma(  
        persist_directory=persist_directory,  
        embedding_function=embedding,  
    )  
    return vectordb.as_retriever()
```
定义combine_docs函数，处理检索器返回的文本
```python
def combine_docs(docs):  
    return "\n\n".join(doc.page_content for doc in docs["context"])
```
定义get_qa_history_chain函数，返回一个检索问答链
```python
def get_qa_history_chain():  
    retriever = get_retriever()  
    llm = ChatOpenAI(  
        model="deepseek-ai/DeepSeek-V3.2",  
        base_url="https://api.siliconflow.cn",  
        api_key=os.getenv("OPENAI_API_KEY"),  
        temperature=0  
    )  
    condense_question_system_template = (  
        "请根据聊天记录总结用户最近的问题，"  
        "如果没有多余的聊天记录则返回用户的问题"  
    )  
    condense_question_prompt = ChatPromptTemplate([  
        ("system",condense_question_system_template),  
        ("placeholder","{chat_history}"),  
        ("human","{input}")  
    ])  
    retrieve_docs = RunnableBranch(  
        (lambda x: not x.get("chat_history",False),(lambda x: x["input"]) | retriever,),  
        condense_question_prompt | llm | StrOutputParser() | retriever,  
    )  
    system_prompt = (  
        "你是一个问答任务的助手。"  
        "请使用检索到的上下文片段回答这个问题。"  
        "如果你不知道答案就说不知道。"  
        "请使用简洁的话语回答用户。"  
        "\n\n"  
        "{context}"    )  
    qa_prompt = ChatPromptTemplate.from_messages([  
        ("system",system_prompt),  
        ("placeholder","{chat_history}"),  
        ("human","{input}")  
    ])  
    qa_chain = (  
        RunnablePassthrough.assign(context=RunnableLambda(combine_docs))  
        | qa_prompt  
        | llm  
        | StrOutputParser()  
    )  
    qa_history_chain = RunnablePassthrough.assign(  
        context=retrieve_docs,  
    ).assign(answer=qa_chain)  
    return qa_history_chain
```
定义get_resp函数，接收简算问答链，用户输入以及聊天历史，并以流式返回该链输出
```python
def gen_resp(chain,input,chat_history):  
    resp = chain.stream({  
        "input": input,  
        "chat_history": chat_history,  
    })  
    for res in resp:  
        if "answer" in res:  
            yield res["answer"]
```
定义main函数，制定显示效果与逻辑
```python
def main():  
    st.markdown('### 🦜🔗 动手学大模型应用开发')  
    if "messages" not in st.session_state:  
        st.session_state.messages = []  
    if "qa_history_chain" not in st.session_state:  
        st.session_state.qa_history_chain = get_qa_history_chain()  
    messages = st.container(height=550)  
    for msg in st.session_state.messages:  
        with messages.chat_message(msg[0]):  
            st.write(msg[1])  
    if prompt := st.chat_input("Say something!"):  
        st.session_state.messages.append(("human",prompt))  
        with messages.chat_message("human"):  
            st.write(prompt)  
        answer = gen_resp(  
            chain=st.session_state.qa_history_chain,  
            input=prompt,  
            chat_history=st.session_state.messages,  
        )  
        with messages.chat_message("ai"):  
            output = st.write_stream(answer)  
        st.session_state.messages.append(("ai",output))
```
接着使用
```
streamlit run "文件路径" 
```
就可以看到streamlit创建的应用程序了