---
title: diffusers初探
date: 2023-12-25
---
# 安装python3

https://www.python.org/downloads/
下载安装，本机环境：
Macbook Pro M3 Max  16 + 40 64GB
Python 3.11.7

# 虚拟环境创建
```
python3 -m venv ./env  
```

# 进入虚拟环境
```
source .env/bin/activate
```

# 安装diffusers
```shell
pip3 install torch torchvision torchaudio  			#torch       2.1.2
																					 			#torchaudio     2.1.2
																					 			#torchvision    0.16.2
																					 			#如果用cuda，要保证版本兼容性，例如：
																					 			#pip install torch==2.1.0+cu121 torchvision==0.16.0 -f 
git clone https://github.com/huggingface/diffusers.git  #diffusers 0.25.0.devs0
cd diffusers
pip3 install -e ".[torch]"
```

# SDXL

```shell
pip3 install transformers accelerate omegaconf invisible-watermark #PyWavelets-1.5.0 
																																	 #antlr4-python3-runtime-4.9.3 
																																	 #invisible-watermark-0.2.0 
																																	 #omegaconf-2.3.0 
																																	 #opencv-python-4.8.1.78 
																																	 #tokenizers-0.15.0 
																																	 #transformers-4.36.2

```

```python
from diffusers import DiffusionPipeline
import torch

pipe = DiffusionPipeline.from_pretrained("stabilityai/stable-diffusion-xl-base-1.0", torch_dtype=torch.float16, use_safetensors=True, variant="fp16")
pipe.to("mps")

# if using torch < 2.0
# pipe.enable_xformers_memory_efficient_attention()

prompt = "An astronaut riding a green horse"

images = pipe(prompt=prompt).images[0]

```

# Anything

###### 安装omegaconf

```shell
pip install omegaconf
```

###### 引入本地模型

```python
from diffusers import StableDiffusionPipeline
import torch

pipeline = StableDiffusionPipeline.from_single_file(
    "/Users/weisun/Documents/ML/Anything-ink-vivid.fp16.ckpt",
    torch_dtype=torch.float16,
    
)
pipeline.to("mps")

# 设置提示
prompt = "masterpiece, earring, 1 girl, royal cloth"

# 生成图像
images = pipeline(prompt=prompt,num_inference_steps=120).images[0]
images
```

