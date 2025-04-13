### 5.1.3 记忆
在某些应用程序中，记住之前的交互是至关重要的，无论是短期记忆还是长期记忆。使用LangChain，你可以轻松地为链和智能体添加状态以管理记忆。构建聊天机器人是这种能力最常见的用例。在LangChain中，你可以使用`ConversationChain`很快地完成这个过程，只需几行代码即可将语言模型转换为聊天工具。


以下代码使用`text-ada-001`模型创建一个聊天机器人。这是一个只能执行基本任务的小模型。然而，它是GPT - 3系列中速度最快、成本最低的模型。该模型从未针对聊天任务做过微调，但我们可以看到，只需几行LangChain代码，即可使用这个简单的文本补全模型开始聊天：

```python
from langchain import OpenAI, ConversationChain
chatbot_llm = OpenAI(model_name='text-ada-001')
chatbot = ConversationChain(llm=chatbot_llm, verbose=True)
chatbot.predict(input='Hello')
```

在以上代码的最后一行，我们执行了`predict(input='Hello')`。这要求聊天机器人回复我们的`'Hello'`消息。模型的回答如下所示：

```
> Entering new ConversationChain chain...
Prompt after formatting:
The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.
Current conversation:
Human: Hello
AI:
> Finished chain.
'Hello! How can I help you?'
```

由于将`ConversationChain`中的`verbose`设置为`True`，因此我们可以查看LangChain使用的完整提示词。当我们执行`predict(input='Hello')`时，`text-ada-001`模型收到的不仅仅是`'Hello'`消息，而是完整的提示词。该提示词位于标签`> Entering new ConversationChain chain...`和`> Finished chain`之间。

如果继续对话，我们会发现该函数在提示词中保留了对话的历史记录。如果我们接着问模型是不是AI，那么这个问题也将被包含在提示词中：

```
> Entering new ConversationChain chain...
Prompt after formatting:
The following [...] does not know.
Current conversation:
Human: Hello
AI: Hello! How can I help you?
Human: Can I ask you a question? Are you an AI?
AI:
> Finished chain.
'\n\nYes, I am an AI.'
```

`ConversationChain`对象使用提示工程技巧和记忆技巧，将进行文本补全的LLM转换为聊天工具。

尽管LangChain让所有语言模型拥有了聊天能力，但这个解决方案并不像GPT - 3.5 Turbo和GPT - 4这样强大，后两者已经专门针对聊天任务进行了优化。此外，OpenAI已经宣布弃用`text-ada-001`模型。

### 5.1.4 嵌入

将语言模型与你自己的文本数据相结合，这样做有助于将应用程序所用的模型知识个性化。原理与第3章讨论的相同：首先检索信息，即获取用户的查询并返回最相关的文档；然后将这些文档发送到模型的输入上下文中，以便它响应查询。本节展示使用LangChain和嵌入技术实现这一点是多么简单。

`document_loaders`是LangChain中的一个重要模块。通过这个模块，你可以快速地将文本数据从不同的来源加载到应用程序中。

比如，应用程序可以加载CSV文件、电子邮件、PowerPoint文档、Evernote笔记、Facebook聊天记录、HTML页面、PDF文件和许多其他格式。要查看完整的加载器列表，请查阅LangChain文档。每个加载器设置起来都非常简单。本示例复用了第3章中的《塞尔达传说：旷野之息》PDF文件。

如果PDF文件位于当前工作目录下，则以下代码将加载文件内容并按页进行划分：

```python
from langchain.document_loaders import PyPDFLoader
loader = PyPDFLoader("ExplorersGuide.pdf")
pages = loader.load_and_split()
```

在使用PDF加载器之前，需要安装`pypdf`包。这可以通过`pip install pypdf`来完成。


进行信息检索时，需要嵌入每个加载的页面。正如我们在第2章中讨论的那样，在信息检索中，嵌入是用于将非数值概念（如单词、标记和句子）转换为数值向量的一种技术。这些嵌入使得模型能够高效地处理这些概念之间的关系。借助OpenAI的嵌入端点，开发人员可以获取输入文本的数值向量表示。此外，LangChain提供了一个包装器来调用这些嵌入，如下所示：

```python
from langchain.embeddings import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```
要使用`OpenAIEmbeddings`，请先使用`pip install tiktoken`安装`tiktoken`包。

索引保存页面嵌入并使搜索变得容易。LangChain以向量数据库为中心。有许多向量数据库可供选择，详见LangChain文档。以下代码片段使用Faiss向量数据库，这是一个主要由Facebook AI团队开发的相似性搜索库：

```python
from langchain.vectorstores import FAISS
db = FAISS.from_documents(pages, embeddings)
```
在使用Faiss向量数据库之前，需要使用`pip install faiss-cpu`命令安装`faiss-cpu`包。

![image](https://github.com/user-attachments/assets/e5644388-3822-47b6-8b50-86f837c66b45)


图5 - 3展示了PDF文件的内容如何被转换为嵌入向量并存储在Faiss向量数据库中。

现在很容易搜索相似内容：

```python
q = "What is Link's traditional outfit color?"
db.similarity_search(q)[0]
```

我们得到以下内容：

```
Document(page_content='while Link's traditional green tunic is certainly an iconic look, his wardrobe has expanded [...] Dress for Success',
metadata={'source': 'ExplorersGuide.pdf', 'page': 35})
```
这个问题的答案是，Link的服装颜色是绿色。我们可以看到，答案就在选定的内容中。输出显示，答案在`ExplorersGuide.pdf`的第35页。请记住，Python从0开始计数。因此，如果查看原始PDF文件，你会发现答案在第36页，而非第35页。

![image](https://github.com/user-attachments/assets/8d54e946-f797-4398-bdf6-3368b8567866)


图5 - 4显示了信息检索过程如何使用查询的嵌入和向量数据库来识别与查询最相似的页面。

你可能希望将嵌入整合到聊天机器人中，以便在回答问题时使用它检索到的信息。再次强调，使用LangChain，只需几行代码即可轻松实现。我们使用`RetrievalQA`，它接受LLM和向量数据库作为输入。然后，我们像往常一样向所获得的对象提问：

```python
from langchain.chains import RetrievalQA
from langchain import OpenAI
llm = OpenAI()
chain = RetrievalQA.from_llm(llm=llm, retriever=db.as_retriever())
q = "What is Link's traditional outfit color?"
chain(q, return_only_outputs=True)
```

这一次，我们得到以下答案：

```
{'result': "Link's traditional outfit color is green."}
```
![image](https://github.com/user-attachments/assets/a62f6d49-3295-4e9e-81f4-18674dc208c0)




图5 - 5显示了`RetrievalQA`如何使用信息检索来回答用户的问题。正如我们从图中看到的，“提供上下文”将信息检索系统找到的页面和用户最初的查询进行分组。然后，上下文被发送给LLM。LLM可以利用上下文中的附加信息正确回答用户的问题。

你可能会问：为什么在将信息添加到LLM的上下文中之前需要进行信息检索？


事实上，目前已有的语言模型无法处理包含数百页的大型文件。因此，如果输入文件过大，那么我们会对其进行预过滤。这是信息检索过程的任务。在不久的将来，随着输入上下文的不断增加，可能就不再需要使用信息检索技术了。

### 5.2 GPT - 4插件


尽管包括GPT - 4在内的LLM在各种任务上都表现出色，但它们仍然存在固有的局限性。比如，这些模型只能从训练数据中学习，这些数据往往过时或不适用于特定的应用。此外，它们的能力仅限于文本生成。我们还发现，LLM不适用于某些任务，比如复杂的计算任务。

本节关注GPT - 4的一个开创性特点：插件。在AI的发展过程中，插件已经成为一种新型的革命性工具，它重新定义了我们与LLM的互动方式。插件的目标是为LLM提供更广泛的功能，使LLM能够访问实时信息，进行复杂的数学运算，并利用第三方服务。

我们在第1章中看到，模型无法执行复杂的计算，如计算`3695 × 123548`。如图5 - 6所示，我们激活了计算器插件。可以看到，当需要进行计算时，模型会自动调用计算器，从而得到正确的结果。


![image](https://github.com/user-attachments/assets/dcc52e0e-5c04-43dd-9c0d-91b04df3b3dd)


通过迭代部署方法，OpenAI逐步向GPT - 4添加插件，这使得OpenAI能够考虑插件的实际用途及可能引入的安全问题和定制化挑战。虽然自2023年5月以来，插件功能已对所有付费用户开放，但在我们撰写本书之时，OpenAI尚未面向所有开发人员提供创建新插件的功能。

OpenAI的目标是创建一个生态系统，插件可以帮助塑造AI与人类互动的未来。如今，我们很难想象一家企业没有自己的网站会是什么样，但也许很快，每家企业都需要有自己的插件。事实上，Expedia、FiscalNote、Instacart、KAYAK、Klarna、Milo、OpenTable、Shopify和Zapier等公司已经率先推出了几款插件。

除了主要功能，插件还在几个方面扩展了GPT - 4的功能。在某种意义上，插件与5.1.2节讨论的智能体及工具存在一些相似之处。比如，插件可以使LLM检索体育比分和股票价格等实时信息，从企业文档等知识库中提取数据，并根据用户的需求执行任务，如预订航班或订餐。两者都旨在帮助AI获取最新信息并进行计算。然而，GPT - 4中的插件更专注于第三方服务，而不是LangChain工具。

本节从概念上介绍通过探索OpenAI网站所提供的示例来创建插件。我们将以待办事项列表定义插件为例进行说明。由于在我们撰写本书之时，插件仍处于有限的测试阶段，因此我们建议你访问OpenAI网站以了解最新信息。同时请注意，在测试阶段，用户必须在ChatGPT的用户界面中手动启用插件，并且作为开发人员，你最多只能与100个用户共享你的插件。

#### 5.2.1 概述

在开发插件前，你必须创建一个API并将其与两个描述性文件关联起来：一个插件清单和一个OpenAPI规范。当你开始与GPT - 4进行交互时，OpenAI会向GPT - 4发送一条隐藏消息，以检查你的插件是否已安装。这条消息会简要介绍你的插件，包括其描述信息、端点和示例。

这样一来，模型就成了智能的API调用者。当用户询问关于插件的问题时，模型可以调用你的插件API。是否调用插件是基于OpenAPI规范和关于应该使用API的情况的自然语言描述所做出的决策。一旦模型决定调用你的插件，它就会将API的结果合并到上下文中，以向用户提供响应。因此，插件的API响应必须返回原始数据而不是自然语言响应。这使得GPT - 4可以根据返回的数据生成自己的自然语言响应。

如果用户问模型“我在纽约可以住哪里”，那么模型可以使用酒店预订插件，然后将插件的API响应与其文本生成能力结合起来，提供既含有丰富信息又对用户友好的回答。

#### 5.2.2 API

以下是OpenAI在GitHub上提供的待办事项列表定义插件的简化版本：

```python
import json
import quart
import quart_cors
from quart import request

app = quart_cors.cors(
    quart.Quart(__name__), allow_origin="https://chat.openai.com"
)

# 跟踪待办事项。如果Python会话重新启动，则不会持久保存
_TODOS = {}
@app.post("/todos/<string:username>")
async def add_todo(username):
    request = await quart.request.get_json(force=True)
    if username not in _TODOS:
        _TODOS[username] = []
    _TODOS[username].append(request["todo"])
    return quart.Response(response="OK", status=200)

@app.get("/todos/<string:username>")
async def get_todos(username):
    return quart.Response(
        response=json.dumps(_TODOS.get(username, [])), status=200
    )

@app.get("/.well-known/ai-plugin.json")
async def plugin_manifest():
    host = request.headers["Host"]
    with open(".well-known/ai-plugin.json") as f:
        text = f.read()
        return quart.Response(text, mimetype="text/json")

@app.get("/openapi.yaml")
async def openapi_spec():
    host = request.headers["Host"]
    with open("openapi.yaml") as f:
        text = f.read()
        return quart.Response(text, mimetype="text/yaml")


def main():
    app.run(debug=True, host="0.0.0.0", port=5003)
if __name__ == "__main__":
    main()
```
这段Python代码是一个简单的插件示例，用于管理待办事项列表。首先，变量`app`被初始化为`quart_cors.cors()`。这行代码创建了一个新的Quart应用程序，并配置它允许来自`https://chat.openai.com`的跨源资源共享（cross - origin resource sharing, CORS）。Quart是一个Python Web微框架，Quart - CORS则是一个扩展，可以控制CORS。这个设置允许插件与指定URL上托管的ChatGPT应用程序进行交互。

然后，代码定义了几个HTTP路由，分别对应待办事项列表定义插件的不同功能：`add_todo`函数与POST请求相关联，`get_todos`函数与GET请求相关联。

接着，代码定义了两个额外的端点：`plugin_manifest`和`openapi_spec`。这两个端点分别用于提供插件清单文件和OpenAPI规范，这对于GPT - 4和插件之间的交互至关重要。这些文件包含有关插件及其API的详细信息，GPT - 4使用这些信息来了解何时及如何使用插件。

#### 5.2.3 插件清单

每个插件都需要在API域上有一个`ai-plugin.json`文件。举例来说，如果你的公司在`iTuring.cn`上提供服务，那么必须将此文件放在`iTuring.cn/.well-known/`下。在安装插件时，OpenAI将按照路径`/.well-known/ai-plugin.json`查找此文件。如果没有该文件，则无法安装插件。

以下是`ai-plugin.json`文件的极简定义：

```json
{
    "schema_version": "v1",
    "name_for_human": "TODO Plugin",
    "name_for_model": "todo",
    "description_for_human": "Plugin for managing a TODO list. You can add, remove and view your TODOs.",
    "description_for_model": "Plugin for managing a TODO list. You can add, remove and view your TODOs.",
    "auth": {
        "type": "none"
    },
    "api": {
        "type": "openapi",
        "url": "http://localhost:3333/openapi.yaml",
        "is_user_authenticated": false
    },
    "logo_url": "http://localhost:3333/logo.png",
    "contact_email": "support@thecompany.com",
    "legal_info_url": "http://thecompany-url/legal"
}
```
表5 - 1详细列出了该文件中的一些字段。

|字段名称|类型|描述|
| ---- | ---- | ---- |
|`name_for_human`|字符串|人们看到的名称。这可以是公司的全名，但必须少于20个字符|
|`name_for_model`|字符串|模型用于识别插件的简短名称。它只能包含字母和数字，并且不能超过50个字符|
|`description_for_human`|字符串|对插件功能的简单说明，供人们阅读，应少于100个字符|
|`description_for_model`|字符串|详细说明，帮助AI理解插件。因此，向模型解释插件的目的至关重要。该字段最多可以有8000个字符|
|`logo_url`|字符串|插件标识的URL。标识的尺寸最好为512像素×512像素|
|`contact_email`|字符串|人们在需要帮助时可以联系的电子邮件地址|
|`legal_info_url`|字符串|通过该URL，人们可以查找有关插件的更多详细信息|

#### 5.2.4 OpenAPI规范

创建插件的下一步是创建`openapi.yaml`文件。该文件必须遵循OpenAPI标准（详见后文）。GPT模型只通过此文件和插件清单文件中的详细信息来了解你的API。

以下是一个示例，包含待办事项列表定义插件的`openapi.yaml`文件的第一行：（此处未完整展示相关内容 ） 
