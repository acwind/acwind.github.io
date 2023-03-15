---

title: 使用OpenAI API构建自定义知识库AI的指南
categories:
- 编程笔记
key: f49c35ea-1ac7-11ed-861d-0242ac120017

---

原文：Irina Nik
原文连接：[uxdesign.cc](https://uxdesign.cc/i-built-an-ai-that-answers-questions-based-on-my-user-research-data-7207b052e21c)

现代产品通常拥有来自不同来源的大量用户研究数据：产品的数据、用户研究访谈、各种对话文档、客户电子邮件、调查、各种平台上的客户评论等等。

理解所有这些数据是一项具有挑战性的任务。传统的做法是维护一个有着各种相应标签的整洁有序的数据库。

但是，如果我们能够拥有一个个人的AI聊天机器人，可以回答任何关于我们的用户研究数据的问题呢？

通过查询大量历史用户研究数据，聊天机器人可以为新项目、产品或营销活动提供洞察和建议。

现在，只需几行代码就可以实现这一目标。即使没有技术背景，您也可以做到这一点。在本文中，我将逐步解释如何做到这一点。

这项技术最早由丹·希珀（Dan Shipper）描述。

### OpenAI API 

您可能已经熟悉ChatGPT，并希望已经在您的工作流程中使用它了。如果没有，我建议您先阅读我的有关AI对设计影响的文章。

OpenAI还提供了API来发送请求。我们需要这个API来能够向模型发送相关上下文，并保持信息的私密性。

在开始API之前，您可以尝试通过[点击这里 GPT-3 Playground](https://platform.openai.com/playground?model=text-davinci-003)的用户界面与GPT-3模型进行交互。

### 隐私问题 

处理用户数据时存在许多隐私问题。默认情况下，OpenAI不会使用客户通过我们的API提交的数据来训练OpenAI模型或改进OpenAI的服务提供。当然，可能会有更多的安全限制。请查阅OpenAI文档以获取更多信息，并与您的法律团队咨询。

### 自定义知识库 

我们希望使用我们的研究数据，而不仅仅是来自互联网的通用知识。我们该如何做到？

当我首次接触这个问题时，我认为可以使用我们的数据集微调模型。结果证明，微调是通过提供提示-响应示例来训练模型以特定方式回答问题的。

微调可帮助训练模型识别情感，例如。为此，您需要在训练数据中提供句子-情感值对，如下例所示：

```json
{"prompt":"非常高兴拥有新iPhone！ ->", "completion":" 正面情绪"}  
{"prompt":"湖人 连续第三个晚上让人失望。 ->", "completion":" 负面情绪"}
```

但在我们的情况下，我们没有提示-响应示例。我们只有想要使用的数据来找到答案。因此，在这种情况下，微调无法奏效。

### 将上下文发送到提示中 

相反，我们需要让模型了解上下文。我们可以通过在提示本身中提供上下文来实现。

像这样发送内容给ChatGPT：
```text
已知有如下信息： 
---------------------  
{我们准备喂给ChatGPT的数据}  
---------------------  
根据我刚刚提供的内容，请回答下面的问题: {用户的提问}
```

这个做法不错!

不过，有一个问题。我们不能仅在一个提示中发送我们所有的研究数据。这是计算上不合理的，而且GPT-3模型的请求/响应有一个硬限制，最多只能处理2049个“标记”。这大约相当于请求和响应组合的8k字符。

我们需要找到一种方法，以便只发送相关信息，帮助我们的聊天机器人回答问题，而不是在请求中发送所有数据。

### 有一个库可以实现这个功能 

好消息是，我们可以通过一个名为[ GPT Index ](https://gpt-index.readthedocs.io/en/latest/)的开源库很容易地实现。这个库由 Jerry Liu 创建。

#### GPT Index 的工作原理

1.它将你的知识库创建文本块的索引; 
2.根据你的问题找到最相关的文本; 
3.使用最相关的文本块向 GPT-3 提问。

这个库可以为我们完成所有繁重的工作，我们只需要编写几行代码即可。让我们开始吧！

#### 开始动手

动手之前你需要具有一定的Python编程知识。首先你需要安装GPT Index，推荐使用Python3.10或者更高的版本，命令行如下:

```bash
	pip install llama-index
```

开始写两个函数。第一个函数从我们的数据构建一个索引，第二个函数将请求发送到 GPT-3。以下是伪代码：

```python
def construct_index: 
	1. 设置将要使用的LLM参数 
	2. 构建索引 
	3. 将索引保存到文件中

def ask_ai: 
	1. 获取用户的查询
	2. 发送带有相关上下文的查询给ChatGPT 
	3. 显示回复
```

#### 构建索引

首先，我们需要构建一个索引。索引就像一个数据库，将我们要投喂的资料以一种易于查找的方式存储文本片段。

为此，我们必须将所有的数据收集到一个文件夹中。然后我们要求 GPT Index 获取文件夹中的所有文件，并将每个文件分成小的连续片段。然后将这些片段以可搜索的格式存储。


```python
import os
os.environ["OPENAI_API_KEY"] = '你的OpenAI的 api key'
from llama_index import GPTSimpleVectorIndex, SimpleDirectoryReader

def construct_index(directory_path):  
	documents = SimpleDirectoryReader(directory_path).load_data()
	index = GPTSimpleVectorIndex(documents)
	index.save_to_disk('index.json') 
```
给这个函数你资料库的目录作为参数即可,它在当前目录下生成索引文件<b>index.json</b>

#### 开始问问题

现在，让我们问问题。为了搜索我们创建的索引，我们只需要在GPT Index中输入一个问题。

GPT Index会找到与问题最相关的索引部分 它将把它们与问题结合起来，打包然后发送到GPT-3。 然后，它将打印出响应。

```python
import os
os.environ["OPENAI_API_KEY"] = '你的OpenAI的 api key'
from llama_index import GPTSimpleVectorIndex, SimpleDirectoryReader

def ask_ai():  
	index = GPTSimpleVectorIndex.load_from_disk('index.json')
	response = index.query("{用户针对我司产品的提问内容}")
	print(response.response)
```
GPT对问题的回复就在response.response变量中,你的程序对输出结果进行处理即可。

<small>(译者注:GPT Index 会在函数执行时返回所消耗的token数,我试了一段常见的网站隐私协议文本,前后大概一共消耗了10,000的token)</small>

### 测试一下  

由于用户研究数据是保密的，我不能与您分享。因此，为了测试代码，我将使用自动生成的采访作为示例的知识库。

我请求ChatGPT生成有关在家烹饪和使用家用电器的采访问题。然后，我请求根据这些问题生成采访脚本。这些采访结果相当空洞，没有什么见地，但这已足以测试我们的人工智能。

在上传文件并构建索引之后，我可以尝试提出关于这些采访的问题。

例如，我可以要求我的聊天机器人“为一个空气炸锅进行营销活动创意，以吸引那些在家烹饪的人”。它将根据我提供的采访生成创意，而不是基于互联网上的一般知识。

[代码示例，请点击此处](https://colab.research.google.com/drive/1PQXcM_jhN6QJ7uTkxvNbxoI54r03uSr3?usp=sharing)


### 可能的应用 

我们只用几行代码就创建了一个具有自定义知识库的人工智能。想一想，自定义人工智能变得多么容易。

在短短几分钟内，我们成功建立了一个用于搜索研究数据库见解的自定义解决方案。相同的技术可以用于数十种不同的用例中。

想象一下一个医疗保健聊天机器人，它可以根据用户的健康历史和症状提供医疗建议。或者一位人工法律顾问，能够挖掘法律文件以提供有意义的信息。或者一位具有金融法规和最佳实践知识库的聊天机器人，能够协助财务规划并帮助您做出明智的决策。

而这只是今天我们探讨的一个小技巧。随着大型语言模型的进步，现在比以往任何时候都更容易创建能够满足您特定需求的自定义人工智能。

接下来，您想要创建什么？

<small style="color:gray">由ChatGTP翻译后,进行了二次代码的整理</small>