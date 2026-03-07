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
