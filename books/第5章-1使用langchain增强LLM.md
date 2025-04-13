### 第5章 使用LangChain框架和插件增强LLM的功能

本章探讨LangChain框架和GPT - 4的插件。我们将研究LangChain框架如何实现与不同语言模型的交互，以及插件在扩展GPT - 4功能方面的重要性。这些高阶知识对于开发复杂、尖端的LLM驱动型应用程序至关重要。

### 5.1 LangChain框架

LangChain是专用于开发LLM驱动型应用程序的框架。你会发现，集成LangChain的代码比第3章提供的示例代码更优雅。该框架还提供了许多额外的功能。

使用`pip install langchain`可以快速、简便地安装LangChain。

在我们撰写本书之时，LangChain仍处于beta版本0.0.2XX，并且几乎每天都有新版本发布。LangChain的功能可能会有所变化，因此我们建议谨慎使用该框架。

图5 - 1显示了LangChain框架的关键模块。

![image](https://github.com/user-attachments/assets/739a8740-dc1c-4409-b1d8-a8d62ce3dbad)


![LangChain框架的关键模块](此处无实际图片，原书对应图5 - 1)


以下概述这些关键模块。

- **Models（模型）**：该模块是由LangChain提供的标准接口，你可以通过它与各种LLM进行交互。LangChain支持集成OpenAI、Hugging Face、Cohere、GPT4All等提供商提供的不同类型的模型。

- **Prompts（提示词）**：提示词已成为LLM编程的新标准。该模块包含许多用于管理提示词的工具。

- **Indexes（索引）**：该模块让你能够将LLM与你的数据结合使用 ¹。截至2023年12月2日，LangChain已将Indexes模块改名为Retrieval模块。——译者注

- **Chains（链）**：通过该模块，LangChain提供了Chain接口。你可以使用该接口创建一个调用序列，将多个模型或提示词组合在一起。

- **Agents（智能体）**：该模块引入了Agent接口。所谓智能体，就是一个可以处理用户输入、做出决策并选择适当工具来完成任务的组件。它以迭代方式工作，采取一系列行动，直到解决问题。

- **Memory（记忆）**：该模块让你能够在链调用或智能体调用之间维持状态。默认情况下，链和智能体是无状态的。这意味着它们独立地处理每个传入的请求，就像LLM一样。


LangChain是用于不同LLM的通用接口，你可以查阅其文档以了解更多信息。

LangChain的文档包含一份集成列表，其中涉及OpenAI和其他许多LLM提供商。大多数集成需要API密钥才能建立连接。对于OpenAI模型，你可以按照第2章介绍的方式，在环境变量`OPENAI_API_KEY`中设置API密钥。

#### 5.1.1 动态提示词

要解释LangChain的工作原理，最简单的方法就是展示一个简单的脚本。在这个例子中，我们用OpenAI模型和LangChain来完成一个简单的文本补全任务：

```python
from langchain.chat_models import ChatOpenAI
from langchain import PromptTemplate, LLMChain

template = """Question: {question}
Let's think step by step.
Answer: """
prompt = PromptTemplate(template=template, input_variables=["question"])
llm = ChatOpenAI(model_name="gpt-4")
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = """ What is the capital of the country where the Olympic Games were held in 2016? """
llm_chain.run(question)
```

输出如下：

```
Step 1: Identify the country where the Olympic Games were held in 2016.
Answer: The 2016 Olympic Games were held in Brazil.
Step 2: Identify the capital of Brazil.
Answer: The capital of Brazil is Brasilia.
```

`PromptTemplate`负责构建模型的输入。也就是说，它能以可复制的方式生成提示词。它包含一个名为`template`的输入文本字符串，其中的值可以通过`input_variables`进行指定。在本例中，我们定义的提示词会自动将“Let's think step by step”部分添加到问题中。

本例使用的LLM是gpt - 4。目前，默认模型是gpt - 3.5 - turbo。`ChatOpenAI`函数将模型的名称赋给变量`llm`。这个函数假定用户在环境变量`OPENAI_API_KEY`中设置了API密钥，就像前几章的示例所示的那样。

提示词和模型由`LLMChain`函数组合在一起，形成包含这两个元素的一条链。最后，我们需要调用`run`函数来请求补全输入问题。当运行`run`函数时，LLMChain使用提供的输入键值（以及可用的记忆键值）格式化提示词模板，随后将经过格式化的字符串传递给LLM，并返回LLM输出。我们可以看到，模型运用“逐步思考”的技巧自动回答问题。

正如你所见，对于复杂的应用程序来说，动态提示词是简单而又颇具价值的功能。

#### 5.1.2 智能体及工具

智能体及工具是LangChain框架提供的关键功能：它们可以使应用程序变得非常强大，让LLM能够执行各种操作并与各种功能集成，从而解决复杂的问题。

这里所指的“工具”是围绕函数的特定抽象，使语言模型更容易与之交互。智能体可以使用工具与世界进行交互。具体来说，工具的接口有一个文本输入和一个文本输出。LangChain中有许多预定义的工具，包括谷歌搜索、维基百科搜索、Python REPL、计算器、世界天气预报API等。要获取完整的工具列表，请查看LangChain文档中的工具页面。除了使用预定义的工具，你还可以构建自定义工具并将其加载到智能体中，这使得智能体非常灵活和强大。

正如第4章所述，通过运用“逐步思考”的技巧，你可以在一定程度上提高模型的推理能力。在提示词末尾添加“Let’s think step by step”，相当于要求模型花更多时间来回答问题。

本节介绍一种适用于应用程序的智能体，它需要一系列中间步骤。该智能体安排执行这些步骤，并可以高效地使用各种工具响应用户的查询。从某种意义上说，因为“逐步思考”，所以智能体有更多的时间来规划行动，从而完成更复杂的任务。

智能体安排的步骤如下所述。

1. 智能体收到来自用户的输入。

2. 智能体决定要使用的工具（如果有的话）和要输入的文本。

3. 使用该输入文本调用相应的工具，并从工具中接收输出文本。

4. 将输出文本输入到智能体的上下文中。

5. 重复执行步骤2 - 步骤4，直到智能体决定不再需要使用工具。此时，它将直接回应用户。

![image](https://github.com/user-attachments/assets/fb1dc3e0-dd1a-4026-9be7-7fb72f67e1f0)


图5 - 2展示了LangChain智能体如何使用工具。

![LangChain智能体如何使用工具](此处无实际图片，原书对应图5 - 2)

就本节而言，我们希望模型能够回答以下问题：2016年奥运会举办国首都的人口的平方根是多少？这个问题并没有特殊的含义，但它很好地展示了LangChain智能体及工具如何提高LLM的推理能力。

如果将问题原封不动地抛给GPT - 3.5 Turbo，那么我们会得到以下回答：
```
The capital of the country where the Olympic Games were held in 2016 is Rio de Janeiro, Brazil. The population of Rio de Janeiro is approximately 6.32 million people as of 2021. Taking the square root of this population, we get approximately 2,513.29. Therefore, the square root of the population of the capital of the country where the Olympic Games were held in 2016 is approximately 2,513.29.
```
这个回答至少有两处错误：巴西的首都是巴西利亚，而不是里约热内卢；6320000的平方根约等于2513.96，而不是2513.29。我们可以通过运用“逐步思考”或其他提示工程技巧来获得更好的结果，但由于模型在推理和数学运算方面存在困难，因此我们很难相信结果是准确的。使用LangChain可以给我们更好的准确性保证。


如以下代码所示，LangChain智能体可以使用两个工具：维基百科搜索和计算器。在通过`load_tools`函数创建工具之后，我们使用`initialize_agent`函数创建智能体。智能体的推理功能需要用到一个LLM，本例使用的是gpt - 3.5 - turbo。参数`ZERO_SHOT_REACT_DESCRIPTION`定义了智能体如何在每一步中选择工具。通过将`verbose`的值设置为`True`，我们可以查看智能体的推理过程，并理解它是如何做出最终决策的。

```python
from langchain.chat_models import ChatOpenAI
from langchain.agents import load_tools, initialize_agent, AgentType

llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)
tools = load_tools(["wikipedia", "llm-math"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)
question = """What is the square root of the population of the capital of the country where the Olympic Games were held in 2016?"""
agent.run(question)
```

在使用维基百科搜索工具之前，需要安装相应的Python包`wikipedia`。可以使用`pip install wikipedia`来安装这个包。


正如你看到的，智能体决定查询维基百科以获取有关2016年奥运会的信息：

```
> Entering new chain...
I need to find the country where the Olympic Games were held in 2016 and then find the population of its capital city. Then I can take the square root of that population.
Action: Wikipedia
Action Input: "2016 Summer Olympics"
Observation: Page: 2016 Summer Olympics
[...]
```

输出的下一行包含维基百科关于奥运会的摘录。接下来，智能体使用维基百科搜索工具又进行了两次额外的操作：

```
Thought:I need to search for the capital city of Brazil.
Action: Wikipedia
Action Input: "Capital of Brazil"
Observation: Page: Capitals of Brazil
Summary: The current capital of Brazil, since its construction in 1960, is Brasilia. [...]
Thought: I have found the capital city of Brazil, which is Brasilia. Now I need to find the population of Brasilia.
Action: Wikipedia
Action Input: "Population of Brasilia"
Observation: Page: Brasilia
[...]
```

下一步，智能体使用计算器工具：

```
Thought: I have found the population of Brasilia, but I need to calculate the square root of that population.
Action: Calculator
Action Input: Square root of the population of Brasilia (population: found in previous observation)
Observation: Answer: 1587.051038876822
```

得出最终答案：

```
Thought: I now know the final answer
Final Answer: The square root of the population of the capital of the country where the Olympic Games were held in 2016 is approximately 1587.
> Finished chain.
```

正如你所见，该智能体展示了较强的推理能力：在得出最终答案之前，它完成了4个步骤。LangChain框架使开发人员能够仅用几行代码就实现这种推理能力。 
