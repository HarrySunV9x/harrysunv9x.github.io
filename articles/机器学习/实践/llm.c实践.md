---
title: llm.c实践
data: 2024-04-15
---

**数据准备**

运行脚本`python prepro_tinyshakespeare.py`，需要依赖：

1. `tqdm`：`tqdm`是一个用于在命令行界面显示进度条的Python库。它可以很方便地显示循环迭代过程中的进度，并提供了多种样式和选项来自定义进度条的外观和行为。通过使用`from tqdm import tqdm`语句，您可以在代码中使用`tqdm`库来显示进度条。
2. `tiktoken`：`tiktoken`是一个用于将文本分割成单词或子词的Python库。它可以将文本转换为标记序列，以便进行自然语言处理（NLP）任务，如机器翻译、文本分类等。通过使用`import tiktoken`语句，您可以在代码中使用`tiktoken`库来进行文本分割和标记化。
3. `numpy`：`numpy`是一个用于进行科学计算和数值操作的Python库。它提供了高性能的多维数组对象（`ndarray`）和各种用于处理这些数组的函数。`numpy`是许多其他科学计算库的基础，如`pandas`、`scikit-learn`等。通过使用`import numpy as np`语句，您可以在代码中使用`numpy`库，并使用`np`作为别名来引用它。

```
pip install tqdm tiktoken numpy
```

**下载并保存预训练的 GPT-2 权重**

使用脚本`python train_gpt2.py `进行模型训练，需要依赖：

PyTorch 是一个用于构建深度学习模型的开源机器学习库。它提供了丰富的工具和函数，用于定义、训练和评估各种类型的神经网络模型。通过导入 `torch`，您可以使用 PyTorch 提供的功能来创建和操作张量（Tensors），执行计算图和进行模型训练等。

Transformers 是 Hugging Face 公司开发的一个开源库，专注于自然语言处理（NLP）中的预训练模型和文本生成任务。它提供了各种预训练模型（如BERT、GPT、RoBERTa等），这些模型在大规模文本数据上进行了训练，并具有出色的语言理解和生成能力。通过导入 `transformers`，您可以使用该库提供的功能来加载、使用和微调这些预训练模型。

`pip install torch transformers`

它将保存两个文件：

- gpt2_124M.bin 文件，包含在 C 语言中加载模型所需的权重；
- gpt2_124M_debug_state.bin 文件，包含更多调试状态：输入、目标、logits 和损失。这对于调试 C 语言代码、单元测试以及确保 llm.c 与 PyTorch 参考实现完全可媲美非常重要。

**编译并运行**

```
make train_gpt2					# make

OMP_NUM_THREADS=8 ./train_gpt2  # Linux

$env:OMP_NUM_THREADS = "8"		# PowerShell
./train_gpt2
```

**测试**

```
make test_gpt2
./test_gpt2
```

对比torch实现的效果

