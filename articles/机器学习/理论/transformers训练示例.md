---
title: transformers训练示例
date: 2024-03-18
---

```python
!pip install transformers[torch] -U
!pip install torch torchvision torchaudio -U
!pip install datasets
#%%
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForSequenceClassification, Trainer, TrainingArguments
import torch
from accelerate import Accelerator
#%%
# 检查CUDA或MPS可用性并设置相应的设备
if torch.cuda.is_available():
    device = torch.device("cuda")
elif torch.backends.mps.is_available():
    device = torch.device("mps")
else:
    device = torch.device("cpu")

print(f"Using device: {device}")
#%%
# 加载IMDb数据集
dataset = load_dataset("imdb")

# 加载预训练的分词器
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# 定义预处理函数
def preprocess_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True, max_length=512)

# 应用预处理
tokenized_dataset = dataset.map(preprocess_function, batched=True)

# 加载预训练模型并发送到设备
model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2).to(device)

# 定义训练参数
training_args = TrainingArguments(
    output_dir="./results",
    learning_rate=2e-5,
    per_device_train_batch_size=16,
    num_train_epochs=3,
    weight_decay=0.01,
)

# 初始化 Accelerator
accelerator = Accelerator()

# 创建Trainer实例
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset["train"],
    eval_dataset=tokenized_dataset["test"],
    tokenizer=tokenizer
)
#%%
# 使用 Accelerator 进行训练
trainer.model = accelerator.prepare(model)

# 训练模型
trainer.train()
#%%
# 评估模型
trainer.evaluate()
```

