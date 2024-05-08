---
title:大模型LLM构建导论
data:2024-04-24
---

# 前言

2024年，AI可能不是一个程序员专业方向，但需要是具备的一项基本技能。本篇将从工程的角度，浅显的介绍一下如何搭建一个数据自己的AI助手。

- 模型获取、训练、微调、量化
- 服务搭建  langchain 前端 客户端
- 部署 JetBrain Docker
- 测试

# 模型

从宏观上看，AI的基本原理是：AI系统接收输入数据，经过模型处理后生成输出结果。大型语言模型（LLM）是一种用于处理文本的模型。模型是整个AI应用的核心部分。

模型的获取方式：

- 拿来主义：适合编程小白与懒狗
- 模型微调：所有人都可以尝试的大众方案
- 从头开始训练：超高校级有钱人的玩具

## 模型获取

我推荐两种模型获取方式：

1. 抱抱脸社区

   推荐获取网站：https://huggingface.co/models，人称抱抱脸。抱抱脸（Hugging Face）是一个致力于自然语言处理（NLP）和机器学习的开源社区平台，属于模型界的GitHub。在这里，我们可以轻松发现、分享和使用海量由个人或组织训练好的模型。

2. Ollama

   Ollama是Github上的一个开源项目，我们可以轻松用一行命令行得到Ollama可提供的模型：

   ```
   Ollama pull llama3
   ```

   不过要注意的是，简单的代价是限制。使用Ollama的模型需要通过特定的接口：

   - REST API

     **Generate a response**

     ```
     curl http://localhost:11434/api/generate -d '{
       "model": "llama3",
       "prompt":"Why is the sky blue?"
     }'
     ```

     **Chat with a model**

     ```
     curl http://localhost:11434/api/chat -d '{
       "model": "llama3",
       "messages": [
         { "role": "user", "content": "why is the sky blue?" }
       ]
     }'
     ```

   - LangChain

     ```python
     from langchain_community.llms import Ollama
     llm = Ollama(model="llama3")
     llm.invoke("Why is the sky blue?")
     ```

   - LlamaIndex

     ```python
     from llama_index.llms.ollama import Ollama
     llm = Ollama(model="llama3")
     llm.complete("Why is the sky blue?")
     ```

   也可以直接在目录` ~/.*ollama*/models`里获得模型文件，但笔者未尝试过是否可以直接调用。

## 模型训练

模型的完整训练过程入门参考之前的博客：[机器学习的完整流程——线性回归训练实例](https://harrysunv9x.github.io/机器学习/理论/机器学习的完整流程——线性回归训练实例/)，简单来说模型训练分为：

1、数据准备

2、模型构建

3、定义损失函数

4、定义优化算法

5、执行训练

6、估量误差

几个步骤。对于不同的模型，

# 前后端搭建

本节推荐搭建方式：ollama + langchain.

**安装依赖**

```txt
# pip install -r requirements.txt
fastapi==0.110.2
langchain==0.1.16
langserve==0.1.0
sse-starlette==2.1.0
pydantic==1.10.13
```

**基础搭建**

```python
from langchain_community.llms import Ollama					# llm
from langchain_core.prompts import ChatPromptTemplate		# prompt
from langchain_core.output_parsers import StrOutputParser	# output_parser

llm = Ollama(model="llama3")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a world class technical documentation writer."),
    ("user", "{input}")
])
output_parser = StrOutputParser()

chain = prompt | llm | output_parser
chain.invoke({"input": "how can langsmith help with testing?"})
```

**基于历史的服务端搭建**

```python
from typing import Tuple, List

from langchain_community.llms import Ollama										# llm

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder		# prompts
from langchain.prompts.prompt import PromptTemplate

from langchain_core.output_parsers import StrOutputParser						# output_parser

from fastapi import FastAPI														# fastapi
from fastapi.middleware.cors import CORSMiddleware

from langserve import add_routes												# langserve
from langserve.pydantic_v1 import BaseModel, Field

from langchain.schema.runnable import RunnableMap, RunnablePassthrough			# input

llm = Ollama(model="llama3")
output_parser = StrOutputParser()

_TEMPLATE = """鉴于以下对话和后续问题，请将后续问题重新表述为一个独立的问题

历史对话:
{chat_history}
Follow Up Input: {question}
Standalone question:"""
CONDENSE_QUESTION_PROMPT = PromptTemplate.from_template(_TEMPLATE)


def _format_chat_history(chat_history: List[Tuple]) -> str:
    """Format chat history into a string."""
    buffer = ""
    for dialogue_turn in chat_history:
        human = "Human: " + dialogue_turn[0]
        ai = "Assistant: " + dialogue_turn[1]
        buffer += "\n" + "\n".join([human, ai])
    return buffer


ANSWER_TEMPLATE = """
无论什么情况下，都使用中文回答：

Question: {question}
"""
ANSWER_PROMPT = ChatPromptTemplate.from_template(ANSWER_TEMPLATE)

_question = {
    "question": lambda x: x["standalone_question"],
}

_inputs = RunnableMap(
    standalone_question=RunnablePassthrough.assign(
        chat_history=lambda x: _format_chat_history(x["chat_history"])
    )
                        | CONDENSE_QUESTION_PROMPT
                        | llm
                        | StrOutputParser(),
)


# User input
class ChatHistory(BaseModel):
    """Chat history with the bot."""

    chat_history: List[Tuple[str, str]] = Field(
        ...,
        extra={"widget": {"type": "chat", "input": "question"}},
    )
    question: str


conversational_qa_chain = (
        _inputs | _question | ANSWER_PROMPT | llm | StrOutputParser()
)
chain = conversational_qa_chain.with_types(input_type=ChatHistory)

app = FastAPI(
    title="LangChain Server",
    version="1.0",
    description="A simple api server using Langchain's Runnable interfaces",
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["*"],
)

add_routes(
    app,
    chain,
    path="/llama3",
)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

**前端调用**

```tsx
// {"input": {
//     "chat_history": [
//         ["user", "嘻嘻嘻"],
//         ["Assistant","了解"]
//     ],
//         "question": "你好，今天感觉怎么样"
// }}

// setChatHistory(JSON.stringify([...JSON.parse(chatHistory), {Human: inputText}]))
// setChatHistory(JSON.stringify([...JSON.parse(chatHistory), {Assistant: data.output}]))

fetch(LLM_ADDRESS, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            "input": {
                "chat_history": chat_history,
                "question": inputText
            }
        })
    }).then((res) => res.json())
        .then((data) => {
            setChatHistory((prevChatHistory) =>
                JSON.stringify([
                    ...JSON.parse(prevChatHistory),
                    {Assistant: data.output},
                ])
            );
            setLoadingState(false);
        });
    setInputText('');
};
```

