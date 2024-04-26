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
