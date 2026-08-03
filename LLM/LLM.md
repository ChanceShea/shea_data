# Agent
## Agent和LLM的区别
**LLM（Large Language Model 即大语言模型）**
LLM的工作原理就是预测下一个字。你给它一段话，它会根据学到的语言规律，一个字一个字地往后接。虽然LLM可以回答各种问题，但是它不能帮助用户实际的解决问题
例如，用户让LLM帮忙修改代码的bug，LLM只会给你改好的代码，但是无法自己打开编辑器去修改文件。LLM只能在对话框中回答问题，却没办法跟外部世界互动，没办法进行任何操作
LLM没有记忆能力，LLM的记忆仅限于当前这轮对话的上下文窗口，对话一结束，一切都会归零。前面聊的各种东西都会失效
同时，LLM无法调用工具，例如用户询问今天天气怎么样，LLM只能根据训练数据里的就知识猜测，而无法上网查询实时的天气情况。同时LLM也无法知道当前的实时日期
当用户给LLM一个复杂任务时，它只能一次性生成一大段文字，而无法分析和拆解问题，然后一步步去执行。LLM只能你问一句它答一句，没办法自主拆解任务，指定计划，分步执行
**Agent**
Agent就是LLM在循环中自主使用工具的系统
Agent通过LLM理解意图，推理判断，决定下一步行动，通过规划模块来把复杂的任务拆分成可执行的步骤。通过记忆模块，负责存储和检索信息，让Agent能在长时间任务中保持连贯。通过工具模块来跟外部世界互动
当用户需要查询天气时，Agent就会调用内置的查询天气的工具，调用api查询实时的天气情况
## Agent和Workflow
Workflow就是开发者提前写好一个工作流程，LLM根据这个固定的工作流程来执行，每一步都是写死在代码中的
Agent是LLM动态主导自生流程与工具调用的，每一步都由LLM推理，然后执行。由LLM来决定如何完成任务
由于Workflow的流程是固定的，因此token的消耗比较节省，而Agent需要反复推理决策，因此token的消耗就会高很多
当任务较为简单时，就可以通过Workflow来固定LLM的执行流程，如果任务是开放式的，无法预测执行过程的，就可以通过Agent来灵活应对各种情况
## Agent的基本架构由哪些核心组成
Agent的基本架构有四个核心组件，LLM、工具、记忆、规划模块
LLM是整个系统的大脑，负责理解任务和做决策；工具让Agent能跟外部世界交互，搜索、执行代码、调API；记忆让Agent在任务执行过程中保持状态；规划模块负责把复杂的目标拆解成可执行的步骤
**LLM**
LLM是整个Agent的大脑，所有的输入，不管是用户的指令、工具返回的结果还是记忆力调出来的内容，都需要经过LLM来理解和做决策。LLM负责判断下一步该做什么，统一了其他三个部件
**工具系统**
工具是Agent和外部世界交互的渠道。LLM本质上只是个语言处理器，它无法对外界环境做出任何感知。而工具就是LLM和外界交互的部件，工具可以是任何能用函数封装的能力。工具的定义需要有名称，描述，参数说明。LLM读取了这个定义之后，就能知道这个工具是怎么调用的，需要哪些参数。之后任务的执行过程中，如果有需要调用工具的地方，LLM就能够知道该如何调用工具
当Agent的功能变得强大之后，Agent需要调用的工具也会变多，因此工具的管理和标准化就是一个问题
所以就有了MCP，MCP底层是一套基于JSON-RPC的通信协议，它把工具调用变得标准化。有了MCP之后，工具的调用就能够统一。工具提供方只需要提供一个MCP Server，任何支持MCP的Agent都能够发现并调用这个Server。不需要写额外的代码
**记忆系统**
最基础的就是**短期记忆**，就是当前这轮对话的上下文，Agent在一次任务执行过程中靠它记住中间状态。对话关闭，记忆就结束
然后是**长期记忆**，通过向量数据库来实现，把重要信息embedding之后保存起来，下次用的时候就可以通过向量数据库做语义检索获得到。向量数据库里存的数据虽然很多，但是不能什么都往数据库中存，这样会导致检索出来的噪音很多，会干扰模型的决策
**规划模块**
Agent应对复杂任务靠的就是规划模块，当遇到复杂任务时，Agent会想将其拆分，变成一个个简单的任务，然后将这些任务分别执行，最终就能完成复杂任务
规划模块底层依赖的是LLM的推理能力，而提升推理能力有几种主要的技术
- **CoT（思维链）**，核心思想是让Agent把思考过程写出来，而不是直接输出最终答案。实现方法也较简单，可以直接在prompt中加一句`任务执行时要一步步思考，分步执行`，模型就能把推理的中间步骤一步步展开
- **ToT（思维树）**，它不是走一条线性的推理链，而是在每个推理节点上展开多个可能的分支，然后评估每个分支的质量，选出最优的路径继续往下走
规划模块在实际运作中有两种主流模式
- **ReAct(Reasoning and Acting)**，边执行边规划，每走一步就根据当前的结果重新思考下一步该做什么，不提前制定完整的计划。好处是灵活性极高，能够根据实际情况随时调整；缺点是容易走偏，因为每一步都是局部最优决策，有时候会忽略整体目标
- **Plan-and-Execute**，先规划后执行，先让LLM输出一个完整的步骤列表，然后按顺序逐步执行。好处是整体结构清晰，能在执行前就看到完整计划，方便人工审核；坏处是如果中间某一部的执行结果与预期不同，原来的计划可能就不合适了，需要重新调整

## Agent有哪些工作模式
### ReAct（Reasoning + Acting）
ReAct是最经典，最基础的Agent工作模式，ReAct即Reasoning + Acting，也就是推理+行动。几乎所有主流的Agent框架底层都在用它
其核心思想是：Agent在思考和行动之间不断交替，是一个三步循环，先思考，分析当前情况并决定下一步做什么；然后是行动，调用工具来执行任务；之后是观察，查看工具的返回结果。然后继续思考，判断任务是否已经完成。如此循环，直到任务完成
**优点**
ReAct的优点是透明可审计，每一步思考过程都能看得见、灵活适应，遇到意外能随时调整、通用性强
**缺点**
token消耗大，每一步都需要完整推理一次；有时候可能会陷入循环，反复执行相同动作导致任务无法结束
### Plan-and-Execute
如果说ReAct是边想边做，Plan-and-Execute就是先想好再做
它把Agent的工作分为两个阶段，第一阶段是规划，Agent先把完整的执行流程规划好，想清楚；第二步是执行，按计划逐步完成，不用每步都重新思考全局
Plan-and-Execute最大的优势就是token消耗少，因为规划只做一次，执行阶段不需要反复进行推理，缺点就是不够灵活，如果执行到第3步发现情况变了，原来的计划可能不适用了。所以也有个做法是在执行过程中加入重新规划检查点，每隔几步检查一下计划是否还靠谱
### Reflection
Reflection就是做完再检查，核心思想是Agent完成任务后不着急交付，而是先自我检查一遍
实现方式有两种
- 自我反思，同一个Agent完成任务后切换到审查者的角色来审视自己的输出，发现问题就修改，然后再审查，直到满意
- 双Agent对话，一个Agent负责生成，另一个Agent负责评审，两者来回迭代直到评审方满意，就像Code Review。这种模式非常适合对质量要求较高的场景
### Multi-Agent
当任务太过复杂，一个Agent搞不定的时候，就可以通过Multi-Agent模式，让多个专业化的Agent各司其职，每个Agent负责一个小人物，通过多个Agent协作来完成任务
不过通常情况下，一个强大的单Agent就足够了。只有任务确实需要拆分成多个并行子任务时，多Agent才值得引入
同时，这几种模式并不是互斥的，实际情况下往往是组合使用的。一个多Agent系统中，每个Agent内部可能用的是ReAct模式，整体协作用的是Multi-Agent模式，最后还有一个Reflection环节来检查质量
## Function Call
Function Call 即让LLM调API
Function Call解决了两个核心问题
- 什么时候调用：LLM能根据用户的自然语言意图，自动判断需不需要调用工具、调用哪个工具。不需要写复杂的条件判断，LLM可以自己推理
- 传什么参数：LLM能从用户的自然语言中提取出结构化的参数。不需要用户传递方法所需的参数
Function Call是一次性的单步调用，LLM判断需要调用一个函数，调用完就结束了。而Agent是循环调用，Agent在一个循环中反复使用Function Call，每次调用后观察结果，再决定下一步要不要继续调用其他的函数
Function Call是Agent的原子操作，Agent是Function Call的高级编排。一个Agent完成一个复杂的任务，可能需要调用十几次的Function Call
## MCP
当一个Agent越做越大，需要连接的工具和服务越来越多，就会导致集成这些工具过于麻烦了。如果一个Agent要集成几十个工具，开发者就需要为每一个服务单独写一套适配代码
而为了解决上述问题，就有了MCP（模型上下文协议）
MCP提供了一个统一的标准，让任何AI应用都能用同一种方式链接任何外部工具和数据源
类似于HTTP协议，不同的浏览器，不同的服务器，都需要遵循HTTP协议，客户端和服务端之间进行数据传输，如果没有HTTP协议，可能就会有很多种传输形式，而有了HTTP协议将数据传输进行了统一，就会极大地方便了开发，而MCP协议就是这样的一种协议，MCP协议让LLM调用外部工具的方法进行统一
MCP协议中有三种角色
- MCP Host，就是AI应用，例如Claude Code，Codex，或者自己开发的Agent应用。是整个交互的发起方
- MCP Client，它住在Host里面，负责和MCP Server通信。Host需要什么能力，就可以通过Client去跟对应的Server进行沟通
- MCP Server，负责对外暴露具体的工具能力和数据资源。比如有一个Github MCP Server，它能提供”搜索代码“，”创建Issue“等工具
MCP整个流程就是：用户在AI应用中提问，AI应用通过MCP Client发现有哪些可用工具，AI决定调用某个工具，MCP Client向对应的MCP Server发送请求，MCP Server执行操作返回结果，AI基于结果生成回答
MCP Server暴露的工具是可发现的，AI应用启动时能自动查询有哪些MCP Server可用、每个Server提供哪些工具、每个工具的参数是什么，这意味着Agent可以在运行时动态发现新的能力，而不是只能用开发者写死的函数
## Skills
skills是一种自然语言指令文件，通常是Markdown格式，用来教Agent在什么场景下、按照什么方法、遵循什么规范来完成特定人物
在Claude Code、Cursor等AI工具中，Skills通常以`SKILL.md`文件的形式存在
Skills的结构很简单，顶部有一段yml格式的元数据，声明这个skill什么时候应该被激活；下面是具体的行为指令，用自然语言写成
Skills的核心是质量和知识，它在执行的过程中可以调用工具
当Agent启动时，它会扫描可用的Skills列表。当用户提出请求时，Agent会自动判断有没有匹配的Skill。如果有，Agent就会把这个Skill的内容加载到上下文中，然后按照Skill中的指令来思考和行动
Skill不只是告诉Agent怎么想，还能告诉Agent怎么做。一个Skill可以在SKILL.md中通过`allowed-tools`字段声明它需要使用哪些工具，也可以打包可执行的脚本文件，甚至可以直到Agent去调用MCP或发起Function Call
Skills的核心价值在于将专业知识和最佳实践编码成可复用的模块
## A2A协议
A2A协议是用于多个Agent之间进行协作的协议
假设有多个Agent，这些Agent由不同的团队开发，使用不同的框架，它们之间需要协作，就需要A2A协议
A2A协议能够让不同的AI Agent互相发现、互相通信、互相委派任务，不管这些Agent是用什么框架开发的，运行在什么平台
A2A协议有以下几个关键概念
- Agent Card（智能体名片），每个支持A2A协议的Agent都会发布一个JSON格式的名片，描述自己的身份、能力，擅长的领域、支持的交互方式，认证要求等信息。其他Agent通过读取这个名片来了解“这个Agent会什么”
- Task（任务），A2A中所有的协作都围绕Task展开。一个Client Agent创建一个任务，发送给一个Remote Agent。任务有完整的生命周期，从创建、处理中、到完成或失败，每个状态变化都有明确的定义。这让双方能清楚地跟踪任务进展
- Message&Artifact（消息和制品），Agent之间通过Message来沟通过程中的信息，通过Artifact来传递最终结果。比如，一个研究Agent完成分析后，会把分析报告作为制品返回给请求方
### Agent和MCP
MCP是竖向的，处理Agent到工具的连接；A2A是横向的，处理Agent到Agent的协作。两者是互补的关系，一个编排Agent通过A2A协议把任务委派给不同的专业Agent，而每个专业Agent内部通过MCP来连接它自己需要的工具。Agent之间的通信走A2A，Agent调用工具走MCP
A2A协议完全基于现有的Web标准，HTTP用于传输、JSON用于消息格式、SSE用于实时流式通信。这意味着不需要引入任何新的基础设施，现有的Web技术栈就可以支持
其次，它支持异步和长时间运行的任务。Agent之间的协作往往不是一瞬间能完成的，一个研究Agent可能需要几个小时才能完成深度调研。A2A对此有完整的支持，包括任务状态查询、进度更新、断线重连等
Agent之间不仅能传递文本，还能交换音频、视频、图像、结构化数据等各种内容
## AI Engineering
### Prompt Engineering
**Prompt Engineer**是最基础的层面，核心在于把指令说清楚，通过精心设计的输入给大模型语言的提示词，引导模型生成更符合预期的单词回答。技巧包括角色设定、提供示例(Few-shot)、要求模型一步步思考(CoT)等。但是它难以应对复杂，多步骤的任务
### Context Engineering
当AI应用从聊天问答升级为执行任务时，仅靠输入指令就不够了，必须要把上下文信息给充分。**Context Engineering**负责在有限的上下文中，为模型提供完成当前任务所需的外部知识、历史记录和业务规则，让模型基于更充分的信息做决策
### Harness Engineering
当AI Agent需要自主完成长周期、多步骤的复杂任务时。**Harness Engineering**会为这个系统构建一个“外骨骼”，确保AI在真实、复杂的环境中稳定、可控的工作
# RAG
## RAG
当用户需要询问的问题是某个公司内部的问题时，大模型在训练时，没有这些数据，就可能会导致要么大模型说无法获取到公司的内部信息，要么就是开始编答案，但是全都是错误的
大模型有以下问题
- 知识有截止日期，大模型的知识来源于训练数据，而训练数据是有时间截止的。比如某个模型的训练数据截止于2025年，那么2025年之后发生的事情，大模型就无法知道了
- 大模型不知道私有的数据，一些公司的内部文档，内部信息，规章制度等信息，这些数据没有出现在大模型的训练集中，大模型也无法知道
- 大模型有幻觉，对于用户问的一些问题，大模型会给出一些完全不存在的答案，会虚构一些不存在的东西回答给用户
RAG就是来解决上述这些问题的
RAG，全称Retrieval-Augmented Generation，即检索增强生成，其核心思想非常简单
- 先从外部知识库中**检索**出和用户问题相关的资料
- 把检索到的资料作为**增强**上下文，和用户的问题一起给到大模型
- 大模型基于这些真实的资料来**生成**回答
如此，大模型的回答就有依据，不会凭空捏造，而是基于真实文档生成的。知识可以随时更新，只需要更新知识库即可；私有数据也可以使用，只需要把公司文档放进知识库；幻觉问题也会大幅度减少，可以参考真实的资料
RAG是一种让大模型基于真实、可更新的知识来回答问题的技术方案。它和微调、Prompt Engineering等技术是互补的
## RAG的流程
RAG系统分为两大阶段：索引阶段（离线）和查询阶段（在线）
### 索引阶段
索引阶段的目的是把各种非结构化的文档，变成可以被高效检索的形式。这个阶段通常包括以下几个步骤
- 文档加载：这一步是把各种格式的文档加载进来。解析各种格式的文档并将其统一转换成纯文本格式
- 文档切割：将上一步转换出来的纯文本格式的文档进行切割。因为转换出来的文档可能会很长，如果一次性全部扔给大模型的话，就会导致大模型检索效率很低。很难检索出有效答案。文档切割策略是RAG中非常关键的一环，文档切割并不能一味地按照固定长度切，这样很可能会把一句话切成两半，或者把一个段落切割成两半，导致大模型无法检索到相关内容
- 向量化：文本块切好之后，下一步是把每个文本块转换成一个数学向量。这个步骤叫做Embedding。Embedding模型把一段文本翻译成了一个高维空间中的一个坐标点，语义相近的文本在这个空间的距离也会很近
- 存入向量数据库：将所有文本块向量存入向量数据库。向量数据库是专门用来存储和检索向量的数据库，它能高效地进行相似度搜索。即给一个查询向量，它能快速找到和它最相近的Top-K个向量
### 查询阶段
知识库建立好之后，当用户提出问题，就进入查询阶段
- 用户提问向量化：用户提出一个问题，系统先用同样的Embedding模型，把这个问题也转换成一个向量
- 相似度检索：拿着问题的向量，去向量数据库中进行相似度搜索，找到和这个问题最相近的Top-K个文本块
- 构造增强Prompt：把检索出来的文本和用户的原始问题拼接在一起，构造一个增强Prompt
- 大模型生成回答：把增强的Prompt发给大模型，大模型基于这些真实的参考资料来生成回答。因为有了具体的参考资料，大模型就不会凭空编造了
## 微调和RAG的区别
微调，简单来说就是拿一个已经预训练好的大模型，用自己的数据对它进行进一步训练，让它在某个特定的领域或特定任务上表现得更好
微调的方式也有不同的级别
- 最重的方式是**全量微调**，也就是更新模型的所有参数，效果最好但是成功极高，需要大量GPU资源
- 比较轻量的方式是参数**高效微调**，比如现在最流行的LoRA技术，它只训练一个很小的低秩适配器，不动模型主体，所以硬件需求就会大幅度降低
- 还有一种叫做**指令微调**，是用指令-回答格式的数据来微调，目的是让模型学会听懂指令、按指令来回答，而不是自由发挥
**微调**相当于让模型上了一门课，将知识全都记忆到脑子里了，考试的时候是闭卷考试，模型靠记忆来回答。但记忆可能会出错，而且新知识需要重新训练
**RAG**则相当于给模型配了一个参考书架，考试的时候是开卷考试，模型可以翻书找答案。参考书可以随时更新，新增知识不需要重新训练模型
RAG和微调的区别可以从以下几个方面来进行对比
- **知识更新方式**：这是两者最核心的差异，微调需要更新知识，就得重新训练模型，成本高、周期长。对于一些经常更新的知识，如果用微调，就需要经常重新训练模型。但RAG就比较简单，只需要把新的知识替换旧的知识，重新做一下索引即可，不需要动模型，成本极低，而且几乎可以实时更新
- **准确性**：微调的模型是靠记忆来回答的，就像闭卷考试，有可能记错或者记混。微调更多的是让模型学到了数据的模式和风格，对具体事情的准确性并不一定有保障。RAG的回答是基于检索到的真实文档的，可以追溯、验证。只要检索不出大问题，准确性通常比微调更高
- **语言风格控制**：这是微调的强项。如果你希望模型用文言文回答，用特定的客服话术回答、模仿某个名人的说话方式，微调能做到。因为它在模型训练的过程中就学会了这种风格。RAG对输出风格的控制能力就比较有限，检索到的内容只是给模型的参考，但模型最终怎么组织语言，还是取决于模型本身
- **成本**：微调的初始成本高，需要准备大量的高质量训练数据、租GPU资源，但推理阶段的成本相对较低，因为不需要走检索流程。RAG的初始成本低，不需要训练模型，只需要建个知识库，但是每次查询都有检索开销，token消耗更多，因为你得把检索到的上下文都放到Prompt里
## RAG的文档切割策略
由于大模型上下文窗口的限制，我们不可能把整篇文档都丢进大模型上下文中。因此必须先检索出最相关的部分，再把这部分内容喂给大模型。要进行检索，就必须先把文档切分成小块，这样才能精确匹配到和用户问题最相关的那个片段。如果你不切割，整篇文章作为一个检索单元，那么当用户问一个问题时，就需要把整篇文档都检索出来，大模型的上下文根本放不下
但是文档切割有两个问题，文档切的太小，文本块的信息太少，可能会丢失上下文；文档切得太大，文本块信息太大，检索精度就会下降，而且会浪费大模型上下文窗口
### 固定大小切割
最简单的一种方式，按照固定字符数或token数来切割，比如每500个字符切一块
这个方法的好处就是简单、可预测、容易实现。但是缺点也很明显，它完全不考虑文本的语义和结构，很可能把一句话或一个段落从中间切开
而为了缓解这个问题，通常会引入重叠(Overlap)，也就是相邻两个文本块之间有一段重复的内容，比如每500个字符，重叠50个字符，这样即使句子被切开了，相邻块之间还有一部分重叠信息可以保持连贯
### 递归字符切割
这是最常用的一种策略，就是LangChain框架中`RecursiveCharacterTextSplitter`的实现方式
核心思想是按照文本的自然边界来切，而不是死板地按字符数切
具体来说，它会维护一个分隔符的优先级列表，比如`["\n\n","\n","。","！","？","；","，",","," "]`。它会先尝试双换行符来切割，如果切出来的块太大，就用单换行符来切割，如果还是太大，就用句号来切割，依次类推，直到块的大小符合要求
这样做的好处是，它会尽量保证段落、句子、短语的自然完整性，不会把一句话从中间切开
### 基于文档结构的切割
这种方法是利用文档本身的结构格式来切割
比如Markdown文档，可以按照标题层级来切分：一级标题下面是一块，二级标题下面一块。这样切出来的内容天然具有语义完整性。比如代码文件，可以按照函数、类来切割。HTML文件可以按照标签来切。PDF文件可以按照页面来切
这种方法的好处是充分尊重文档的结构，切出来的语义完整。但缺点是只适用于格式规范的文档，如果文档本身结构混乱，或者没有明显的格式标记，就不太好用了
### 语义切割
前三种方法都是基于规则的，不管内容是什么，都按照预设的规则来切割。语义切割则不一样，它是基于内容的语义相似度来决定在哪切
核心思想是把语义相似的句子放在一起，语义发生转折的地方就是切割点
先将文档按句子切分，然后计算相邻句子之间的语义相似度（用Embedding模型将句子转换成向量，然后计算余弦相似度）。如果相邻句子的相似度很高，说明它们在讨论同一个话题，应该放在同一个块里；如果相似度突然下降，说明话题发生了转变，这里就是一个切割点
这种方法的切割质量最高，但是计算成本也最大，因为需要对每个句子都做Embedding计算
### Agent驱动的智能切割
这是最近出现的前沿的方法，思路是用LLM本身来判断该怎么切分
具体做法是，先用上述方法生成初始的文本块，然后用LLM来审视每个块，判断这个块的内容是否完整，是否需要和相邻的块合并或拆分。LLM充当一个智能编辑的角色，根据对内容的理解来做出切割决策
理论上这种方法的效果最好，因为LLM能理解文本的深层语义，但成本也是最高的，因为需要大量调用LLM
## Re-rank
向量检索返回的结果中，顺序不一定是最优的。即检索到的最相关的答案不一定是我们最想要的那个答案，这时候就需要Re-rank
RAG系统中，检索阶段用的是`Bi-Encoder`（双编码器），也就是Embedding模型。它的工作方式是把查询和文档分别编码成向量，然后通过计算向量之间的余弦相似度来判断相关性。这种方式有一个先天的不足：它是把查询和文档分别处理的。模型在编码一篇文档的时候，根本不知道用户会问什么问题，所以它只能生成一个泛化的、平均化的语义向量，无法针对具体的查询做优化
具体来说，Bi-Encoder有两个关键缺陷
- **信息压缩缺失**：好比用一句话来总结一篇论文，再精彩的总结也不可能保留每个细节。Embedding也是类似，一段几百字的文本被压缩成一串数字，那些细微但关键的信息可能就丢了
- **缺乏查询意图感知**：文档的向量是在索引阶段就预先计算好的，那时候大模型根本不知道用户会问什么。所以同一篇文档的向量，不管用户是在问事实、做对比分析还是查步骤说明，都是完全一样的，没法根据不同的问题做出针对性的判断
### Cross-Encoder
Re-rank使用的是`Cross-Encoder`（交叉编码器）。它和Bi-Encoder的根本区别在于，它把查询和文档拼在一起，让模型同时看到两者，进行深度的交互分析
Cross-Encoder把用户查询和文档拼接成一个输入，然后把整个输入送进Transformer模型，通过自注意力机制，让查询中的每个词都能和文档中的每个词进行交互。模型能精确地分析查询的每个关键词在文档中是否有对应的回答，文档的内容是否真正满足了查询的意图
但是Cross-Encoder的检索速度太慢了，Bi-Encoder的检索之所以快，是因为文档的向量是预先计算好的，检索的时候只需要计算查询的向量，然后做向量相似度搜索就行，几毫秒就能搞定。而Cross-Encoder需要把查询和每一个候选文档都拼在一起，送进模型做一次完整的推理。如果有1000万篇文档，就需要做1000万次推理，这个速度就会很慢
所以就会使用**两阶段检索**
- 第一阶段使用Bi-Encoder快速从全部文档中筛选出少部分最相关的文档，这一步追求的是速度和召回率，宁可多捞一些，不能漏掉相关的
- 第二阶段使用Cross-Encoder对这少部分候选文档做精细化重排序，因为数量已经很少了，所以即使是速度比较慢的Cross-Encoder也能在可接受的时间内完成
对于一些需要串联多个文档的问题，或者用户问题表述的不够精确的场景下，Re-rank的准确率会大幅度提升
## 向量数据库
假设你有100万个文本块，每个文本块都被转换成了一个1536维的向量。当用户提问时，你需要快速找到和问题向量最相近的Top10个文本块
如果是暴力搜索，那每次查询都需要和100万个向量逐一计算相似度，速度就会非常的慢
向量数据库就是为了解决上述问题的。它使用特殊的索引结构（HNSW，IVF，PQ等），能够在海量向量中实现毫秒级的近似最近邻搜索(ANN)
**核心原理**
向量数据库能做到毫秒级检索的核心思路是用空间换时间，通过提前构建索引结构，让检索的时候不需要便利所有向量。目前主流的索引思路有三种
- 最常用的就是**HNSW**，HNSW给所有向量建立了一个多层导航图，从粗到细，层层缩小范围，几跳就能直接定位到目标。这种查询方式速度极快，召回率也非常高，是目前大多数向量数据库的默认选择。类似于在一栋大楼里找人，这栋楼有一个智能导航系统，能告诉你要找的人在第几层，然后再告诉你在第几个房间，相比于一层一层找人，这样的速度就会快很多
- 第二种是**IVF**，核心思想是先分区域再搜索。IVF先把所有向量按相似度聚合成若干个桶，查询的时候先判断目标最可能在哪几个桶里，然后只在这几个桶内进行搜索，不用遍历全部向量。好处是内存效率高，适合超大规模数据，但是因为只搜索部分桶，所以会损失一些召回率
- 第三种是**PQ**，核心是压缩存储。PQ讲一个完整的向量切成几段，每段做一个简化版的表示，就想把一张高清图片压缩成缩略图，虽然细节丢了，但是关键特征还在。这样每个向量只需要很少的字节就能够存储，在内存有限的场景下特别实用。实际场景中通常会PQ和IVF结合使用，先分桶再压缩，兼顾速度和内存
## 多路召回
向量检索擅长捕捉语义相似性，但是可能会出现以下情况：用户搜索手机的精确型号，却会出现一些不同型号的手机。这是因为通常情况下两个手机的手机型号的语义很像，向量检索无法分清，但是用户想要的是某一个具体型号的手机，向量检索却会检索出另一个型号，这很明显不太行
向量检索虽然在语义理解方面非常强大，但是在精确匹配方面却有短板。因此光靠向量检索这一条路，召回的结果可能会有遗漏
**多路召回**的核心思想就是：既然一条路不保险，那就多走几条路，每条路各有侧重，最后把各路结果合并
常见的召回通道有
- **向量检索**：用Embedding模型把文本转成稠密向量，通过相似度搜索来召回。擅长语义匹配，是RAG系统的主力召回通道
- **关键词检索**：传统的全文检索方式，基于词频和文档频率来计算相关性。BM25是最经典的算法。擅长精确关键词匹配，对于专有名词、产品编号、人名等有天然优势
- **知识图谱检索**：如果知识库有结构化的知识图谱，可以用图查询来做第三路召回。擅长实体关系推理
多路召回的关键在于融合。各路召回的结果需要合并去重，并按综合相关性排序。融合的方法有很多，最常用的是RRF（倒数排名融合，Reciprocal Rank Fusion）。RRF的核心思想是，不看每个文档的绝对分数，只看它在各路召回中的排名，排名越靠前，得分越高；被越多路召回命中的文档，综合得分也更高
除了向量检索和关键词检索，还有一些进阶的召回策略
- **查询改写**：在检索之前，先用LLM把用户的原始问题改写成更适合检索的形式，然后再进行检索
- **HyDE**：先让LLM根据用户问题生成一个假设性的答案文档，然后用这个假设文档的向量去检索。因为假设文档在表述上更接近真实的答案文档，所以检索效果更好
- **多查询扩展**：把用户的一个问题，用LLM改写成多个不同角度的子问题，分别检索后合并结果
## 大模型幻觉
幻觉，就是大模型自己编造一些听起来很真实但实际上根本不存在的内容
幻觉通常分为两大类
**事实性幻觉**
生成的内容与客观事实不符，而这种幻觉又分为两类
- **内在幻觉**：生成内容与输入的上下文信息矛盾，比如文档中写的是A，但模型说是B
- **外在幻觉**：生成内容无法从输入上下文或任何已知知识中验证
**忠实性幻觉**
在RAG场景中更为常见。模型虽然拿到了正确的检索文档，但在生成回答时偏离了文档内容，加入了文档中没有的信息，或者曲解了文档的含义
### 如何降低幻觉
RAG通过以下机制来降低幻觉
- **注入真实知识**：检索到的文档是真实存在的资料，模型基于这些资料来生成回答，大幅减少了凭空编造的可能性。研究表明RAG能将事实性幻觉降低20%-40%
- **注意力机制偏向**：当Prompt中包含了检索文档时，大模型的注意力权重会显著偏向这些文档，这确保了模型优先参考检索到的真实信息
- **可追溯可验证**：RAG生成的回答可以追溯到具体的文档来源，如果发现回答有误，可以定位是那个文档出现了问题
但是RAG不能完全消除幻觉。如果检索到了错误的文档、模型曲解了文档内容、或者在检索结果之外做了越界推断，幻觉依然会发生。在实际项目中，通常需要多种策略组合使用，形成一套多层次的防幻觉体系。以下是几种主要的防幻觉策略
**Prompt工程约束**
Prompt工程是成本最低、见效最快的防幻觉手段。它的核心思路是通过精心设计的提示词，来约束模型的行为边界。具体有以下几种策略
- **限制知识范围**：在Prompt中明确告诉模型，只能基于给定的资料来回答，不能随意编造答案
- **要求标注来源**：让模型在回答时标注每个观点是来自哪个文档、哪个段落的
- **鼓励承认不确定**：可以在Prompt指令中加入以下内容`如果你对某个问题的答案不确定，请坦诚说我不确定或者根据现有信息无法确认，而不是猜测`
- **结构化推理**：让模型给出推理步骤，再给出结论。分步推理的方式，能让模型在每一步都停下来思考，减少”脱口而出“的错误
**输出验证**
Prompt约束是事前预防，输出验证是事后检查。核心思路很简单：**不让模型的回答直接到达用户，中间加一层验证环节**。有以下几种策略
- **交叉验证**：用另一个模型来验证回答是否正确。比如用GPT-4生成回答后，再用Claude或另一个GPT-4模型来检查这个回答是否存在事实错误。验证模型的Prompt可以这样设计：`以下是一段AI生成的回答和它引用的参考资料，请逐一检查回答中的每个事实陈述是否都能在参考资料中找到支撑。标注出所有无法被支撑的陈述`
- **用规则引擎验证关键数据**：对于数字、日期、人名、金额等关键信息，可以建立规则引擎来校验。系统可以自动去查询一下回答是否正确。如果不正确，就把这部分回答标红或直接修正
- **基于一致性的多次采样验证**：对同一个问题，让模型生成回答，然后比较这些回答是否一致。如果多个回答对某个关键事实的说法不一致，说明模型对这个事实没有把握，这个事实很可能就是幻觉
**领域微调**
如果模型在某个特定领域的幻觉率非常高，可以考虑用高质量的领域数据对模型进行微调。微调降低幻觉的方式有以下几种
- **用领域数据做有监督微调**：收集高质量的`问题-正确答案`对，对模型进行有监督微调。这些数据中的答案都是经过人工审核的准确内容，模型通过学习这些数据，能在特定领域建立起更可靠的知识体系
- **用RLHF引导诚实**：在人类反馈强化学习（RLHF）阶段，不是奖励模型给出看起来好听的回答，而是奖励它说真话或者坦率承认不知道。
不过微调的成本较高，需要大量的高质量标注数据和计算资源
**GraphRAG和LightRAG**
传统的RAG只能检索文档片段，对于需要全局理解的问题效果并不好
**GraphRAG**的核心思路是：先让LLM从文档中提取实体和关系，构建一个知识图谱，然后基于知识图谱来做检索和回答。知识图谱能捕捉实体之间的关联关系，比单纯的文档片段检索更适合回答需要推理的问题
**LightRAG**是GraphRAG的轻量版，解决了GraphRAG的token开销大，动态更新成本高的问题，通过融合图增强文本索引和双层检索范式，在保持效果的同时大幅度降低了成本
## GraphRAG
传统RAG有以下这些缺点
- 当用户一个问题需要跨多个文档查询时，RAG检索时可能会检索出多个文档的内容，然后并不会根据用户的问题把这些文档做关联，而是一股脑全部扔给大模型。这种需要A文档关联B文档，B文档关联C文档，然后A文档推到C文档的问题，叫做**多跳推理**，传统RAG在这种场景下，给出的答案要么不完整，要么就是直接瞎编
- 传统RAG无法回答全局性问题，即答案不在某一个文本块内，而是需要通读所有文档，然后进行归纳总结才能得出。传统RAG的本质是检索，只能找出语义最相似的TopK个文本块。如果用户询问文章的主题思想，可能就根本查不到正确的东西，而是检索出一些零散的东西。这种问题叫做`Query-Focused Summarization`（查询聚焦的摘要）问题，本质上不是一个检索任务，而是全局归纳任务
- 文档切块会导致语义断裂。原本文档中一段完整的论述可能横跨三个段落，但是在文档切块时，可能会把一个完整的观点拦腰斩开，检索时只召回了上半段，下半段没召回，答案就会不完整；同时，文档切块可能会导致实体之间的联系被切断，导致大模型只能拿到一堆碎片化的东西，而不能关联出原来的逻辑关系
对于传统的RAG，其只能通过检索获取出最相似的TopK个文本块，但是如果涉及到推理、归纳等，传统RAG就无法做到
**GraphRAG**，就是用LLM把文档读成一张知识图谱，然后基于这张知识图谱来做检索和回答
GraphRAG可以将知识以图的形式存储。文档里的人、地、物、概念（实体节点），以及它们之间的关系（边），都会被显式提取出来，形成一张结构化的关系网络。GraphRAG里的知识不是一段话，而是一张关系网，LLM可以在这张网里明确查询出每个节点，每条边
GraphRAG就可以解决上述文本中提到的传统RAG无法解决的那几个痛点
- GraphRAG能让多跳推理变得可行，因为多跳推理的任务就是需要了解多个文档之间的关系，然后顺着文档得出最终的答案，而对于GraphRAG，因为其利用的知识是一张知识图谱而不是一段话，知识图谱天生就会将这些文档之间关联起来，所以GraphRAG很适合做这种任务
- GraphRAG可以将整张知识图谱通过社区检测算法划分成若干个社区，再用LLM为每个社区生成一份摘要（社区，即图里那些关系密切的一群节点，社区内的节点关系非常紧密，而社区之间的联系就会比较稀疏）。而社区有了摘要，对于一些全局性的问题，GraphRAG就可以把每个社区的摘要都拿出来归纳一下，给出一个完整的答案
- 因为GraphRAG会提前将文档中实体的关系抽取出来，所以对于文本切块语义断裂的问题也被解决了，几个实体之间能被联系起来，检索时就不会丢掉因果关系，LLM也不容易混淆了
### GraphRAG的工作流程
GraphRAG的工作流程也分为两个阶段：索引阶段和查询阶段
**索引阶段**
索引阶段就是将原始文档变成一张带有社区的知识图谱，这个阶段很重，需要LLM大量调用，通常只需要离线跑一次即可
索引阶段分为五步
- **文档切块**：这一步和传统的RAG一样。把一整个文档按照一定的字符数切分成一个个文本块
- **实体与关系提取**：这是GraphRAG的一个创新点。每个文本块都会单独发送给LLM，让它把里面的实体和关系抽出来。GrapgRAG会使用一个专门设计的prompt
```text
你是一个实体关系抽取助手。请从下面的文本中：
1. 识别所有的实体，每个实体给出：
   - entity_name：实体名（大写）
   - entity_type：类型（比如 [人物, 组织, 地点, 事件]）
   - entity_description：实体的综合描述

2. 识别所有实体之间的关系，每对关系给出：
   - source_entity：源实体
   - target_entity：目标实体
   - relationship_description：关系的描述
   - relationship_strength：关系强度（1-10 的数字）

文本内容：
{input_text}

请按以下格式输出：
("entity"|实体名|类型|描述)
("relationship"|源|目标|描述|强度)
```
但是这一步可能导致重复和歧义，比如一个人物有多种称呼，LLM就可能把他们抽取成多个实体
- **生成实体/关系摘要**，GraphRAG会把同一个实体在所有文本块里的描述汇总起来，让LLM融合成一段更完整的实体描述。关系也是同样的处理。这一步做完之后，每个实体节点就有了一段综合描述，这段描述会被向量化后存入向量数据库，供后面的Local Serach使用
- **社区检测**：GraphRAG采用Leiden算法，这是一种层次化社区检测算法。GraphRAG会通过这个Leiden算法将知识图谱中所有的节点组织起来，并且划分成多个层次的社区。每个社区中的节点之间的联系都会更紧密，而社区之下还会分为一个个小社区。以此划分，直到社区无法再拆分。这种层次化的划分，就是GraphRAG后面能够处理不同粒度的全局问题的基础
- **生成社区摘要**：GraphRAG会给每个社区单独生成一份摘要报告。这一步也是靠LLM来做。GraphRAG会把一个社区内的所有实体、关系、还有可选的claim都一股脑喂给LLM，让它写一份总结报告。这份报告里会写清楚社区讨论的主题是什么，里面最重要的实体和关系有哪些。这份社区报告就是GraphRAG回答全局性问题的核心资产
**查询阶段**
索引跑完之后，图谱就建好了，而真正使用图谱的时候，GraphRAG提供了两种查询方式：Local Search和Global Search
- **Local Search**：适用于用户问的问题是关于某个具体实体的问题。LLM会先用问题的向量，在所有的实体节点的向量索引里做相似度搜索，找到最相关的几个尸体；然后从入口实体出发，沿着图的边扩展，找到相关的领居实体、关系、原始文本块、所属社区报告；接着把找到的这些信息整合成一段上下文，全部交给LLM去生成答案
- **Global Search**：适用于用户问的问题是全局性、宏观性的问题。而GraphRAG在这里采用的是**Map-reduce**的策略：Map阶段就是把某个层级的所有社区报告都拿出来，分成好多批，对每一批都让LLM生成一个中间答案，并给每个观点打一个重要性评分。Reduce阶段把所有中间答案里的观点汇总，按评分排序，保留最重要的那些，最后让LLM合并成一个完整的答案
### GraphRAG的缺点
- **索引成本高**：传统RAG的索引过程很便宜，就只有切片和Embedding时需要消耗token，而Embedding模型的定价很低，整个过程几乎可以忽略成本。但GraphRAG的索引过程，每一步都需要调用LLM。每个文本块要调LLM来抽取实体和关系，抽完之后还要再调用LLM合成一段综合描述，然后每个社区都需要调用LLM生成一份社区报告。就等于把所有的原文通过LLM反复读了好多遍
- **实体消歧**：这是GraphRAG的第二个难点，生成实体时，由于多篇文档对同一个实体有不同的称呼，多篇文档里的同一个名字，在不同的语境内也有不同的含义，代表着不同的东西。LLM做实体抽取，可能就会把同一个实体在不同文档中的不同名字，抽取成好几个独立的实体。这时候可能就会导致实体堆积，原本可能只有一万个节点，现在以下变成好几万个节点，其中大部分都是同一个实体的不同名字。关系会碎片化，一个实体被抽取成多个独立的节点，每个节点下都有不同的描述和内容，就会导致查询时根本查不出来
- **查询延迟**：传统的RAG查询，向量库的ANN搜索通常能毫秒级返回，整个端到端的问答可能也就一两秒。而GraphRAG就不一样了，Global Search中的Map阶段，可能会涉及成百上千个社区，每个社区都要进行一次LLM调用，意味着同时需要发起上千次的LLM调用；然后是Reduce阶段，让LLM做一次汇总。这样就会导致生成答案的速度变得很慢
- **增量更新**：GraphRAG中，一份新的文档进来，里面可能会有一个实体，和已有图里面的某个节点是同一个东西，但是名字不一样，这时又有实体消歧的问题了。再比如新文档中抽取出一堆关系，这些关系加进图之后，原来的社区划分可能就过时了；社区一变，之前花费大价钱生成的知识图谱全部失效，还需要重新生成一遍
# 流式输出
类似于ChatGPT、deepseek这类上线的大模型，这些大模型都有一个类似于打字机效果的输出，而不是和传统的http请求一样，一次请求，然后一次性将所有的数据全都响应回来
对于流式输出，最常被讨论的有四种方式，分别是**HTTP长轮询、SSE、WebSockte、gRPC流**
## 流式输出协议
### HTTP长轮询
这是最基础的互联网协议，本质是一问一答。客户端发送一次请求，服务端返回一次数据，然后连接关闭
传统的HTTP并不能做流式输出，但是可以利用长轮询来实现服务器主动推送的效果。即**客户端发送一次请求，服务器不着急回复，而是将连接挂起，等到有了新数据之后再回复。客户端收到回复之后，立刻再发起一个新请求，继续挂着等待。如此循环往复**
长轮询有几个很明显的问题
- 每次轮询都有HTTP头开销，大量无效请求浪费宽带
- 服务器维护大量挂起的连接，内存压力大
- 数据有延迟，新数据来了，但是客户端还没发新的请求接收
- 实现复杂，容易出错
在AI流式输出的场景中，几乎不会考虑使用长轮询，因为大模型每个token之间的间隔可能只有几十毫秒，用长轮询意味着几十毫秒内就得重连一次，根本不现实
### SSE
SSE全称Server-Sent Events，这是专门为服务器发送事件给客户端设计的协议
SSE底层还是HTTP，它是HTTP的一种特殊用法。客户端发送一个普通的HTTP GET请求，带上了`Accept: text/event-stream`这个请求头，告诉服务器需要的是一个事件流，让服务端不要一次性返回。服务器收到后，保持连接不断开，把响应的`Content-Type`设置为`text/event-stream`，然后就开始一段一段地往响应体中写入数据，每写一段就是一个时间
SSE的数据格式十分简单，就是纯文本，每个事件用空行分割
```text
data: a

data: b

data: c
```
没有复杂的二进制编码，就是`data:`后面跟着文本内容，然后空行结束一个事件
并且SSE有几个非常好的特性
- **自动重连**：浏览器内置了断线重连机制，连接断了会自动重新连上
- **事件ID**：每个事件可以带上一个id字段，断线重连时浏览器会通过`Last-Event-ID`请求头告诉服务器上次收到了第几条事件，从下一条开始给
- **自定义事件类型**：可以给事件分类，客户端只监听感兴趣的事件类型
- **纯文本**：调试方便，用浏览器开发者工具就能看到原始数据流
### WebSocket
WebSocket是一个完全独立的协议，特点是全双工通信，**服务器和客户端之间可以随时互发消息**
SSE是单向的，只有服务端到客户端，WebSocket是双向的，客户端和服务端都能发送消息
WebSocket的建立过程如下：客户端先发送一个HTTP请求，带上`Upgrade: websocket`头，服务器如果同意升级，就返回`101 Switching Protocols`响应，之后这条TCP连接就从HTTP协议切换到了WebSocket协议，双方可以随时互发消息帧
WebSocket的优势是全双工、低延迟、支持二进制数据。劣势是：
- 协议比SSE复杂得多
- 需要单独的心跳保活机制（SSE的HTTP连接天然有心跳）
- 没有内置断线重连
- 需要专门的消息格式设计
- 在某些企业网络环境/代理服务器下可能被拦截
### gRPC流
gRPC是Google推出的高性能RPC框架，底层使用HTTP/2和Protocol Buffers序列化。gRPC支持三种流模式，服务端流、客户端流，双向流
对于AI场景，最常用的是服务端流。即客户端发送一次请求，服务器持续返回多条响应消息
gRPC流的优势在于二进制序列化的极高效率和HTTP/2的多路复用。劣势在于浏览器原生不支持，需要引入gRPC-Web代理层，对前端开发门槛较高；调试不如纯文本直观
## SSE
为什么WebSocket功能强，但是SSE还是成为了AI流式输出的主流
### 优势
AI聊天的通信模式非常简单，用户发送一句话，AI一个字一个字地回复，回复完了，等待用户发送下一条消息
这是典型的SSE模式，用户不需要再AI回复的过程中打断它、给它发送消息。WebSocket的全双工能力在这种场景下是完全用不上的。WebSocket的全双工通信在这个场景下完全是多余的
SSE的`服务端→客户端`单向推送模型与AI对话场景的匹配度是100%，不需要双向通道
**部署友好**
SSE的底层是标准的HTTP，任何的反向代理都天然支持，不需要任何特殊配置。WebSocket的握手过程需要代理支持`Upgrade`头，但是不少企业网络的代理会拦截或篡改这些非标准HTTP头，gRPC则需要HTTP/2支持，很多老旧设施还停留在HTTP/1.1
**调试友好**
SSE的数据是纯文本，打开浏览器的开发者工具的Network面包，选中`text/event-stream`类型的请求，就能看到原始数据一行行流过来。不需要任何专用工具
**内置API**
浏览器原生提供了`EventSource`对象来消费SSE流，几行代码就能跑
**内置断线重连**
AI生成可能需要30s甚至更长时间。网络一抖动，连接断了。WebSocket断了就得自己写重连逻辑、自己记录上下文、自己恢复状态。而使用SSE，浏览器会自动实现断线重连，并且会带上`Last-Event-ID`告诉服务器从哪里继续
**基础设施兼容**
SSE可以无缝穿越各种CDN、负载均衡器、API网关，不需要特殊配置
### 建立过程
SSE的建立过程就是一个普通的HTTP GET请求，但是有两个特殊的地方
- **请求端**：请求头会带上`Accept: text/event-stream`告诉服务器客户端需要的是事件流。`Cache-Control: no-cache`告诉服务器别给缓存数据，发送实时数据
- **响应端**：响应头会带上`Accept: text/event-stream`确认返回的是事件流。`Cache-Control: no-cache`不缓存。`Connection: keep-alive`保持TCP连接。`Transfer-Encoding: chunked`分块传输，而不是一次性返回完
HTTP传统模式需要服务器在响应头中声明`Content-Length`，用于标注响应体多大，浏览器收到这么多字节就认为响应结束了。但是SSE的特点就是不知道总共有多大，什么时候结束，所以用chunked编码，让数据一块一块地发，每块前面标注多大，最后一块用`0\r\n\r\n`表示结束
### 消息格式
SSE的消息格式规范定义在HTML5标准中，一条SSE消息由若干个字段行组成，以一个空行结束
完整的事件如下
```text
id: 42
event: token
retry: 3000
data: {"text": "你好"，"index": 0}
```
- **id**：事件ID，客户端会记住最后收到的ID，断线重连时通过`Last-Event-ID`请求头发给服务器，让服务器知道从哪里续传。这个在AI流式输出中特别有用，如果AI回复到一半连接断了，重连后可以从断点继续
- **event**：事件类型，默认是message，你可以自定义，比如`token`、`thinking`、`done`。客户端可以只监听特定类型的事件
- **retry**：重连等待时间，告诉浏览器如果连接断了，等待这么多毫秒再重连，默认值通常是3s
- **data**：实际数据。这是最重要的字段，可以是多行（多个`data:`行会被`\n`拼接），但最终是一个字符串
**一个事件必须以空行结束**。浏览器看到空行才会把之前攒的数据当做一个完整事件处理，如果忘了空行，浏览器会一直攒着不触发
还有几个特殊规则
- **冒号开头的行是注释**：以`:`开头的行被忽略，通常用来发送心跳。比如`:keep-alive`
- **`data:`后面有一个空格**：规范写法是`data: a`，冒号后面跟一个空格。但实际上浏览器对空格很宽容，有没有都能解析
- **多行`data:`自动拼接**：如下
```text
data: 1
data: 2
```
客户端收到的`event.data`是`"1\n2"`，中间用换行符拼接
### 场景设计
AI流式输出中，通常需要区分不同类型的事件。比如大模型可能同时输出正文文本、思考过程、工具调用信息。这些不应该混在一起，SSE的`event:`字段就是为此设计的
```text
event: thinking
data: {"content": "我需要先分析用户的问题..."}

event: thinking
data: {"content": "这个问题涉及到..."}

event: token
data: {"content": "根据"}

event: token
data: {"content": "您的描述"}

event: tool_call
data: {"name": "search", "arguments": "{\"query\": \"相关资料\"}"}

event: tool_result
data: {"name": "search", "result": "找到了3条相关结果"}

event: token
data: {"content": "，我找到了以下信息"}

event: done
data: {"totalTokens": 142, "finishReason": "stop"}
```
前端可以根据event类型分别渲染，所有信息在同一条SSE连接上有序传输，互不干扰
### 生命周期
SSE连接的完整生命周期如下
- 首先是客户端发起请求，然后收到200响应，表示连接成功
- 之后就是开始接收数据，每次收到事件，根据事件类型判断是否完成传输
- 传输过程中如果发生了网络异常，就会发起自动重连，retry毫秒之后自动发起重连
- 重连成功，则重新开始接收数据
- 重连失败超过一定的次数限制，则直接失败，SSE连接结束
- 收到`done`类型的事件，表示完成传输，服务器关闭连接，SSE连接结束
**连接保持**：SSE是长连接，服务器端需要确保这个连接不被中间的代理服务器超时关闭。通常的做法是定期发送注释行`:heartbeat\n\n`作为心跳。Nginx默认的`proxy_read_timeout`是60s，如果超过60s内没有任何数据流过，Nginx会断开连接，所以如果AI生成可能超过60s，一定要在服务端加心跳
**连接关闭**：当AI生成完毕后，服务器应该主动关闭连接。在HTTP chunked编码中，关闭就是发送最后的`0\r\n\r\n`标记，在SpringBoot中，当`Flux`发出`onComplete`信号时，框架会自动处理连接关闭
**连接数限制**：HTTP/1.1规定浏览器对同一域名的并发连接数限制是6个。如果你在一个页面里打开了6个SSE连接，第7个就会被阻塞。HTTP/2没有这个限制，所以如果有多个SSE连接，确保使用HTTP/2
## SpringBoot中的SSE流式输出
SpringBoot中实现SSE流式输出，绕不开Flux。它是Project Reactor框架的核心类，也是Spring WebFlux的基础
举例
假设有一根水管，水管一头连接着水源（生产者），另一头有一个人正在使用水（消费者）
**同步模式**就是有一个水桶，水管将水全部灌入到这个水桶中，直到水桶满，然后消费者一次性将这个水桶中的水全部倒走。消费者必须等到水桶满了之后才能倒水，这就是传统的`String`返回模式，方法执行完毕，全部数据准备好，才返回
**Flux模式**就像是一根水管，水源每生产一滴水就会顺着管子流出，消费者只要一打开水龙头就可以直接用水，不需要等待装满。这就是流式输出，数据产生和消费是通信进行的
`Flux`和`Mono`是Reactor的两个核心模型
- **`Mono<T>`** 表示0或1个数据的异步序列，相当于`CompletableFuture<T>`的增强版
- **`Flux<T>`** 0到N个数据的异步序列，相当于异步的 **`List<T>`** ，但是元素是逐个产生的
SSE和Flux是天生一对。它们在概念上是完全对应的。SSE是服务器持续推送事件的协议，Flux是持续产生元素的数据结构。Spring WebFlux内置了对这两者配合的支持，当Controller方法返回`Flux<String>`且指定`produces = MediaType.TEXT_EVENT_STREAM_VALUE`时，Spring会自动把Flux的每个元素包装成一个SSE事件发送给客户端，不需要手写任何代码
**tip**
传统的`spring-boot-starter-web`是基于Servlet的，每个请求都要占用一个线程，而`spring-boot-starter-webflux`是基于Netty的，利用Netty的多路复用特性，可以实现少量线程处理大量连接。SSE是长连接，如果使用传统的MVC，100并发连接就需要100个线程，很容易把线程池耗尽，WebFlux的非阻塞模型可以用极少数的线程支撑大量长连接，非常适合SSE场景
### 示例
最简单的Flux例子，纯粹用Flux模拟流式输出
```java
@RestController
public class StreamController {

	@GetMapping(value = "/hello",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<String> streamHello() {
		return Flux.interval(Duration.ofMillis(500))
					.map(i -> "第" + (i + 1) + "条消息")
					.take(10)
					.doOnNext(s -> System.out.println("发送：" + s))
					.doOnComplete(() -> System.out.println("结束连接"));
	}
}
```
- `Flux.interval(Duration.ofMillis(500))`：每500ms产生一个数字
- `.map(...)`：把数字转换成字符串
- `.take(10)`：只取前10条数据，取完则结束
- `produces = MediaType.TEXT_EVENT_STREAM_VALUE`：告诉Spring这是SSE响应
Spring会自动把每个Flux元素编码成SSE格式发送给浏览器，访问时就能看到文字一条一条出现
### 大模型流式输出
大模型使用的是`SpringAI Alibaba`的依赖
```xml
<!--SpringAI Alibaba DashScope-->  
<dependency>  
    <groupId>com.alibaba.cloud.ai</groupId>  
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>  
    <version>1.1.2.0</version>
</dependency>
```
```java
@RestController
@RequestMapping("/ai/chat")
public class ChatController {
	private final ChatClient chatClient;

	public AiAgent(ChatClient.Builder builder) {  
		this.chatClient = builder  
	        .defaultAdvisors(new MyLoggerAdvisor())  
	        .build();  
	}
	
	@GetMapping(value = "/stream",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<String> stream(String query) {
		return chatClient.prompt()
					.user(query)
					.stream()
					.content();
	}
}
```
上述代码将用户消息发送给了大模型，然后调用模型的流式API，将模型返回的token转换成`Flux<String>`返回，Spring自动把Flux编码成SSE返回给浏览器
上述代码中全程流式输出，不会积攒数据
**结构化SSE事件**
在大模型的流式输出中不仅会有正文文本，还可能会有思考过程、工具调用、错误信息等。直接返回`Flux<String>`只能传输纯文本，无法区分事件类型。所以需要对事件进行一些封装。
可以通过自定义一个SSE事件对象，然后使用`ServerSentEvent`包装
```java
public record ChatStreamEvent(String event,String content,LocalDateTime timestamp) {

	public static ChatStreamEvent token(String content) {
		return new ChatStreamEvent("token",content,LocalDateTime.now());
	}
	
	public static ChatStreamEvent thinking(String content) {
		return new ChatStreamEvent("thinking",content,LocalDateTime.now());
	}
	
	public static ChatStreamEvent done(int totalTokens) {
		return new ChatStreamEvent("done","{\"totalTokens\":" + totalTokens + "}",LocalDateTime.now());
	}
	
	public static ChatStreamEvent error(String message) {
		return new ChatStreamEvent("error",message,LocalDateTime.now());
	}
}
```
```java
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.http.MediaType;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/api/chat")
public class ChatStreamController {

    private final ChatClient chatClient;

    public ChatStreamController(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultSystem("你是一个友好的AI助手。")
                .build();
    }

    /**
     * 结构化 SSE 流式接口
     * 返回 Flux<ServerSentEvent<ChatStreamEvent>>
     * Spring 会把每个 ServerSentEvent 编码成带 event: 和 data: 的 SSE 消息
     */
    @GetMapping(value = "/v2/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<ChatStreamEvent>> streamChatV2(@RequestParam String q) {

        // 前置事件：告诉客户端"开始生成"
        Flux<ServerSentEvent<ChatStreamEvent>> startEvent = Flux.just(
                ServerSentEvent.<ChatStreamEvent>builder()
                        .event("start")
                        .data(ChatStreamEvent.thinking("正在思考你的问题..."))
                        .build()
        );

        // AI token 流：每个 token 包装成一个 SSE 事件
        Flux<ServerSentEvent<ChatStreamEvent>> tokenStream = chatClient.prompt()
                .user(q)
                .stream()
                .content()
                .map(token -> ServerSentEvent.<ChatStreamEvent>builder()
                        .event("token")
                        .data(ChatStreamEvent.token(token))
                        .build()
                );

        // 结束事件：告诉客户端"生成完毕"
        Flux<ServerSentEvent<ChatStreamEvent>> endEvent = Flux.just(
                ServerSentEvent.<ChatStreamEvent>builder()
                        .event("done")
                        .data(ChatStreamEvent.done(0))
                        .build()
        );

        // 拼接：start + tokens + done
        // concat 按顺序执行：先发 start，再流式发 tokens，最后发 done
        // onErrorResume：如果 AI 调用失败，发送 error 事件而不是让连接异常断开
        return Flux.concat(startEvent, tokenStream, endEvent)
                .onErrorResume(e -> Flux.just(
                        ServerSentEvent.<ChatStreamEvent>builder()
                                .event("error")
                                .data(ChatStreamEvent.error(e.getMessage()))
                                .build()
                ))
                // 心跳：每 15 秒发一个注释行，防止代理超时断开
                .mergeWith(
                        Flux.interval(Duration.ofSeconds(15))
                                .map(i -> ServerSentEvent.<ChatStreamEvent>builder()
                                        .comment("keep-alive")
                                        .build()
                                )
                                .takeUntilOther(tokenStream.then())
                );
    }
}
```
上述代码的关键在于
- **结构化事件**：用`ServerSentEvent`包装每个事件，可以指定`event`事件类型和`data`数据内容。Spring会把这些编码成带`event:`前缀的SSE格式
- **三段式拼接**：`Flux.concat(startEvent,tokenStream,endEvent)`，先发一个开始事件，然后流式发出AI的每个token，最后发送一个完成事件，这样前端可以清晰知道生成状态
- **错误处理**：`onErrorResume`确保即使AI调用失败，也会通过SSE发送一个error事件，而不是让连接突然断开，前端可以收到错误信息并展示给用户
- **心跳保活**：`.mergeWith(heartbeat)`确保每15s发送一个注释行，防止Nginx等代理因超时关闭连接。`takeUntilOther`确保在token流结束后心跳也自动结束
对于LangChain4j，实现方式类似，LangChain4j可以通过`@AiService`直接声明返回`Flux<String>`
```java
@AiService
public interface Assistant {
	@SystemMessage("你是一个友好的AI助手")
	Flux<String> stream(String userMessage);
}
```
```java
@RestController
@RequestMapping("/ai/chat")
@RequiredArgsConstructor
public class ChatController {

	private final Assistant assistant;

	@GetMapping(value = "/stream",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<String> stream(String query) {
		return assistant.streamChat(query);
	}
}
```
LangChain4j的`langchain4j-core`模块在底层做了Flux适配，它把`TokenStream`的`onPartialResponse`回调转成了`Flux<String>`的元素发射
同理，LangChain4j也可通过`ServerSentEvent`精细控制发射的元素
```java
@GetMapping(value = "/stream",produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<String>> stream(String query) {
	return Flux.create(sink -> {
		assistant.chat(query)
				.onPartialThinking(thinking -> {
					sink.next(ServerSentEvent.<String>builder()
								.event("thinking")
								.data(thinking.text)
								.build());
				})
				.onPartialResponse(token -> {
					sink.next(ServerSentEvent.<String>builder()
								.event("token")
								.data(token.text)
								.build());
				})
				.onCompleteResponse(response -> {
					sink.next(ServerSentEvent.<String>builder()
								.event("done")
								.data("{\"token\":" + response.tokenUsage().totalTokenCount() + "}")
								.build());
				})
				.onError(err -> {
					sink.next(ServerSentEvent.<String>builder()
								.event("error")
								.data(err.getMessage())
								.build());
				})
				.start();
	});
}
```
通过`Flux.create()`手动桥接，TokenStream的每个回调都对应发射一个`ServerSentEvent`。这种方式可以完全控制每个事件的内容和类型
### 前端消费SSE数据
前端有两种渲染方式，`EventSource` API 和 `fetch + ReadableStream`
**EventSource**
EventSource是浏览器原生API，专门用于消费SSE流
```javascript
const eventSource = new EventSource('/api/chat/stream?query=你好')

eventSource.onmessage = function(event) {
	document.getElementById('output').textContent += event.data;
}

eventSource.addEventListener('thinking',functino(event) {
	const thinkingDiv = docuemtn.getElemetById('thinking');
	thinkingDiv.textContent += event.data; 
	thinkingDiv.style.color = 'gray'; 
	thinkingDiv.style.fontStyle = 'italic';
})

eventSource.addEventListener('token',function(event) {
	const data = JSON.parse(event.data);
	document.getElementById('output').textContent += data.content;
})

eventSource.addEventListener('done',function(event) {
	console.log('生成完成');
	eventSource.close();
})

eventSource.addEventListener('error',function(event) {
	console.error('SSE错误');
	eventSource.close();
})

// 错误处理，可以让浏览器自动重连
eventSource.onerror = functino(event) {
	console.error('连接错误，浏览器自动重连...');
}
```
EventSource的优点是简单、自带重连。缺点是只支持GET请求，如果想利用POST传一个很大的prompt，或者携带复杂的请求体，EventSource做不到
**fetch + ReadableStream**
现代AI更常使用fetch + ReadableStream来消费SSE，因为它支持POST请求和自定义请求体
```javascript
async function streamChat(messages) {
	const resp = await fetch('/api/chat/stream',{
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
			'Accept': 'text/event-stream'
		},
		body: JSON.stringify({messages: messages})
	});
	
	if (!resp.ok) {
		throw new Error(`HTTP ${resp.status}`);
	}
	
	const reader = resp.body.getReader();
	const decoder = new TextDecoder('utf-8');
	let buffer = '';
	
	try {
		while(true) {
			const {done,value} = await reader.read();
			if (done) {
				console.log('流结束');
				break;
			}
			buffer += decoder.decode(value,{stream:true});
			// SSE事件以 \n\n 分割，从buffer中切出完整的事件
			const lines = buffer.split('\n');
			buffer = lines.pop() || '';
			let currentEvent = 'message';
			let curretnData = '';
			
			for (const line of lines) {
				if (line.startsWith('event: ')) {
					currentEvent = line.slice(6).trim();
				} else if (line.startsWith('data: ')) {
					currentData += (currentData ? '\n' : '') + line.slice(5).trim();
				} else if (line === '' && currentData) {
				// 空行 = 事件结束，处理这个事件
					handleSSEEvent(currentEvent,currentData);
					currentEvent = 'message';
					currentData = '';
				}
			}
		}
	} finally {
		reader.releaseLock();
	}
}

function handleSSEEvent(eventType, data) { 
	switch (eventType) { 
		case 'token': 
			const token = JSON.parse(data); 
			appendToChat(token.content); 
			break; 
		case 'thinking': 
			appendToThinking(JSON.parse(data).content); 
			break; 
		case 'done': 
			console.log('生成完成', data); 
			break; 
		case 'error': 
			console.error('错误:', data);
			 break; 
		} 
}
// 使用 
streamChat([ { role: 'user', content: '你好，介绍一下你自己' } ]);
```
通过`fetch`发送一个POST请求，拿到响应后，通过`response.body.getReader()`获取一个字节流reader。然后循环调用`reader.read()`读取每一块数据，解码成字符串，手动解析SSE格式（按`\n\n`分割事件，按`event:`/`data:`提取字段）
这种方式更灵活，但是代码也更加复杂。实际项目中，通常可以封装成一个工具函数或使用第三方库`@microsoft/fetch-event-source`来简化
# 杂项
## Function Calling 和 MCP
**Function Calling**即工具调用，让LLM可以通过工具调用去感知外部环境，执行一些API。给定一个函数定义和参数，调用工具时会根据用户问题填充参数。调用完后返回的结果在交给大模型，大模型根据工具调用的结果进行输出回复
**MCP**是一个JSON-RPC协议，定义了Server和Client之间该如何通信。当有大量工具需要调用时，如果对每个工具都编写一套代码，就会显得特别繁琐。而MCP定义了一套统一标准的规范，提供服务的Server都遵循这个规范来开发，而Client需要使用时，就可以根据Server提供的工具直接调用