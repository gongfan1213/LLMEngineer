#### 3.4.2 项目2：YouTube视频摘要

LLM已被证明在总结文本方面表现出色。在大多数情况下，LLM能够提取文本的核心思想并重新表达，使生成的摘要流畅且清晰。文本摘要在许多情况下很有用，举例如下：

- **媒体监测**：

快速了解重要信息，避免信息过载。

- **趋势观察**：


生成技术新闻的摘要或对学术论文进行分组并生成有用的摘要。 

- **客户支持**：


生成文档概述，避免客户被大量的信息所淹没。 

- **电子邮件浏览**：


突出显示最重要的信息，并防止电子邮件过载。

在本项目中，我们将为YouTube视频生成摘要。你可能会感到惊讶：如何将视频提供给GPT - 4或ChatGPT呢？

诀窍在于，将这个任务分为以下两个步骤：

1. 从视频中提取文字记录。

2. 根据文字记录生成摘要。

要提取YouTube视频的文字记录，方法很简单。在你选择观看的视频下方有一些可选操作项。单击“...”选项，然后选择“Show transcript”，如图3 - 2所示。

![image](https://github.com/user-attachments/assets/15399da6-b22a-4b2e-bb2f-3b38e169c89a)



图3 - 2：获取YouTube视频的文字记录

你将看到一个文本框，其中包含视频的文字记录，如图3 - 3所示。你可以在该文本框中切换时间戳。

![image](https://github.com/user-attachments/assets/796a7c74-ee09-4d92-b67e-d6e6826c0235)



图3 - 3：YouTube视频文字记录示例

如果只为一个视频执行此操作一次，那么你只需复制并粘贴出现在YouTube页面上的文本记录即可。否则，你需要使用自动化解决方案，比如由YouTube提供的API，该API让你能够以编程方式与视频进行交互。你可以直接使用此API和字幕资源，或者使用第三方库，如youtube - transcript - api，又或者使用Captions Grabber等Web实用工具。

获得文字记录之后，你需要调用OpenAI模型以生成摘要。对于本项目，我们使用GPT - 3.5 Turbo。该模型非常适合这个简单的任务，并且是目前为止最便宜的选择。

以下代码片段要求模型生成一份视频文字记录的摘要：

```python
import openai
# 从文件中读取文字记录
with open("transcript.txt", "r") as f:
    transcript = f.read()
# 调用ChatCompletion端点，并使用gpt-3.5-turbo模型
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Summarize the following text"},
        {"role": "assistant", "content": "Yes."},
        {"role": "user", "content": transcript},
    ],
)
print(response["choices"][0]["message"]["content"])
```

请注意，如果视频很长，那么文字记录会超过该模型的上限，即4096个标记。在这种情况下，你需要通过执行图3 - 4所示的步骤来重设标记上限。

![image](https://github.com/user-attachments/assets/b781bd2e-7c00-4937-8623-c4651c9ff65d)


图3 - 4：重设标记上限的步骤

图3 - 4所示的方法称为映射 - 归约。第5章介绍的LangChain框架提供了一种使用映射 - 归约链自动完成此操作的方法。

本项目证明，将简单的摘要功能集成到应用程序中可以带来价值，而只需少量代码即可实现。直接在你自己的用例中使用它，你将获得一个非常实用的应用程序。此外，你还可以基于相同的原理构建其他功能：关键词提取、标题生成、情感分析等。 
