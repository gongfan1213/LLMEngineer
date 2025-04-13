### 5.2.4 OpenAPI规范（续）
```yaml
openapi: 3.0.1
info:
  title: TODO Plugin
  description: A plugin that allows the user to create and manage a TODO list using ChatGPT. If you do not know the user's username, ask them first before making queries to the plugin. Otherwise, use the username "global".
  version: 'v1'
servers:
  - url: http://localhost:5003
paths:
  /todos/{username}:
    get:
      operationId: getTodos
      summary: Get the list of todos
      parameters:
        - in: path
          name: username
          schema:
            type: string
          required: true
          description: The name of the user.
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
  [...]
```

可以将OpenAPI规范视为描述性文档，它足以理解和使用API。在GPT - 4中进行搜索时，使用`info`部分中的描述来确定插件与搜索内容的相关性。OpenAPI规范的其余内容遵循标准的OpenAPI格式。许多工具可以根据现有的API代码自动生成OpenAPI规范。

#### 理解OpenAPI规范

OpenAPI规范（以前被称为Swagger规范）是描述HTTP API的标准。OpenAPI定义允许消费者与远程服务进行交互，而无须提供额外的文档或访问源代码。OpenAPI文档可为各种有价值的用例打下基础，比如生成API文档、通过代码生成工具以多种编程语言创建服务器和客户端、促进测试过程等。


OpenAPI文档以JSON格式或YAML格式定义或描述API及其元素。OpenAPI文档的内容一般从标题、描述和版本号开始。若想深入了解这个主题，请查看OpenAPI GitHub repo，其中包含文档和各种示例。

#### 5.2.5 描述

当插件有助于响应用户请求时，模型会扫描OpenAPI规范中的端点描述及插件清单文件中的`description_for_model`字段。你的目标是创建最合适的响应，这通常涉及测试不同的请求和描述。

OpenAPI文档应该提供关于API的各种细节，比如可用的函数及其参数。它还应包含特定于属性的描述字段，提供有价值的解释，说明每个函数的作用及查询字段期望的信息类型。这些描述指导模型以最恰当的方式使用API。

在这个过程中的关键要素是`description_for_model`字段。通过它，你可以通知模型如何使用插件。我们强烈建议为该字段创建简明、清晰和描述性强的说明。


在撰写描述时，必须遵循以下最佳实践：

- 不要试图影响GPT的“情绪”、个性或确切回应。

- 避免指示GPT使用特定的插件，除非用户明确要求使用该类别的服务。

- 不要为GPT指定特定的触发器来使用插件，因为它旨在自主确定何时使用插件。

回顾一下，开发GPT - 4插件涉及创建一个API，使用OpenAPI规范来指定其行为，并在插件清单文件中描述插件及其用法。通过这种设置，GPT - 4可以有效地充当智能的API调用者，从而获得超越文本生成的能力。

### 5.3 小结
LangChain框架和GPT - 4插件有助于大幅提升LLM的潜力。

凭借其强大的工具和模块套件，LangChain已成为LLM领域的核心框架。它在集成不同模型、管理提示词、组合数据、为链排序、处理智能体和管理记忆等方面的多功能性为开发人员和AI爱好者开辟了新的道路。第3章中的示例证明，使用GPT - 4和ChatGPT从头开始编写复杂指令存在局限性。请记住，LangChain的真正潜力在于创造性地利用各种功能来解决复杂问题，并将通用语言模型转化为功能强大且具体的应用程序。

GPT - 4插件是语言模型和实时可用的上下文信息之间的桥梁。如本章所述，开发插件需要结构良好的API和描述性文件。因此，开发人员必须在这些文件中提供详细和自然的描述。这将有助于GPT - 4充分利用API。

LangChain框架和GPT - 4插件证明，LLM等AI领域正在迅猛发展，本章仅展示了其颠覆性潜力的冰山一角。

### 5.4 总结
本书为你提供了必要的基础知识和进阶知识，以帮助你在真实的应用程序中利用LLM的潜力。我们一起学习了基本原理和API集成知识，了解了提示工程和微调，并研究了GPT - 4和ChatGPT的使用案例。在本书的末章中，我们详细了解了如何利用LangChain框架和GPT - 4插件 ² 充分发挥LLM的潜力，并真正构建创新的应用程序。

有了这些工具，我们就可以在AI领域中进一步开拓，开发利用这些先进语言模型的应用程序。但请记住，AI领域的发展是动态的，要时刻关注进展并相应地进行调整。进入LLM世界只是开始，你的探索不应止步于此。我们鼓励你利用新知识探索AI技术的未来。

注2：2023年11月7日，OpenAI在首届开发者大会上发布了可定制版本的ChatGPT，称为GPTs。与此同时，OpenAI宣布将在2024年初上线GPTs应用商店，允许用户发布自己创建的GPT并获得收益。GPTs应用商店发布后将替代原插件市场。 
