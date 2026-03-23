# nanoGPT-Chinese-Poetry
基于 nanoGPT 实现的轻量化中文唐诗生成模型，支持诗词自动生成、开头续写、本地 CPU/GPU 快速训练与推理。

## 项目背景
nanoGPT 是 Andrej Karpathy 开源的轻量级 GPT 实现，本项目基于该框架做了以下适配优化：
- 适配中文唐诗语料的字符级编码
- 简化训练配置，降低硬件门槛
- 优化生成逻辑，提升诗词通顺度
- 提供完整的「数据预处理→训练→生成」全流程

 ## 项⽬依赖：
(1) pytorch < 3.0
(2) numpy < 3.0
(3) transformers < 3.0
(4) datasets < 3.0 ：加载训练数据集
(5) tiktoken < 3.0 ：可以使⽤BPE编码
(6) tqdm < 3.0 ：⽀持代码进度条

### 1. 环境准备
#### 1.1 克隆仓库
git clone https://github.com/zhzh520/DMX_nanoGPT.git
cd nanoGPT-master

### 2. 数据预处理
#### 2.1 准备语料
项目已内置 tang_poet.txt（唐诗语料）
文本格式要求：每行一首诗，编码为 UTF-8

#### 2.2 执行预处理
将文本转为模型可读取的二进制文件：
python prepare.py
执行后生成：
train.bin：训练数据（占比 90%）
val.bin：验证数据（占比 10%）
同时生成 meta.pkl：字符映射表（记录所有汉字与编码的对应关系）

### 3. 训练模型
使用诗词专用配置训练：
train_poemtext_char.py 
代码如下：

```
out_dir = 'out-poemtext-char'
eval_interval = 250 # keep frequent because we'll overfit
eval_iters = 200
log_interval = 10 # don't print too too often
always_save_checkpoint = False
dataset = 'poemtext'
gradient_accumulation_steps = 1
batch_size = 64
block_size = 256 # context of up to 256 previous characters
n_layer = 6
n_head = 6
n_embd = 384
dropout = 0.2
learning_rate = 1e-3 # with baby networks can afford to go a bit higher
max_iters = 5000
lr_decay_iters = 5000 # make equal to max_iters usually
min_lr = 1e-4 # learning_rate / 10 usually
beta2 = 0.99 # make a bit bigger because number of tokens per iter is small
warmup_iters = 100 # not super necessary potentially
```

<img width="434" height="196" alt="image" src="https://github.com/user-attachments/assets/3349ecff-98b9-4820-9cb3-427699e3a909" />

### 4.模型训练
python train.py config/train_poemtext_char.py
<img width="947" height="715" alt="屏幕截图 2026-03-19 132046" src="https://github.com/user-attachments/assets/45ab1f14-13c6-422a-8aa5-ffafee21d0e0" />
模型训练结束后，会在项⽬⽬录下⽣成⼀个⽂件权重out-poemtext-chat/ckpt.pt，该模型权重就是由58000⾸诗词tang_poet.txt数据得到的GPT

### 5.模型的推理及采样
python sample.py --out_dir=out-poemtext-char
<img width="1460" height="507" alt="屏幕截图 2026-03-19 134345" src="https://github.com/user-attachments/assets/8fd39e35-a1c5-466e-b2be-1aeec36f70de" />