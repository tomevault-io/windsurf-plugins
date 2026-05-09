---
trigger: always_on
description: 这是一个适用于本项目的 `markdown` 格式模板，适用于所有以 `.md` 结尾的文档。文档的主要语言以中文为主，也存在部分英文作为辅助语言。所有的标题和内容均为示例，需要在实际的文档中替换成实际内容。项目整体采用模块化目录结构，每章拥有独立文件夹，文件夹内存放本章相关的所有文件。`.md` 文件推荐在编辑器，如 `Typora` 中以 `GitHub` 主题进行预览，保证最终部署网页版后在线阅读的兼容性和正确显示。
---

# 第零章 格式模板和规范指南

这是一个适用于本项目的 `markdown` 格式模板，适用于所有以 `.md` 结尾的文档。文档的主要语言以中文为主，也存在部分英文作为辅助语言。所有的标题和内容均为示例，需要在实际的文档中替换成实际内容。项目整体采用模块化目录结构，每章拥有独立文件夹，文件夹内存放本章相关的所有文件。`.md` 文件推荐在编辑器，如 `Typora` 中以 `GitHub` 主题进行预览，保证最终部署网页版后在线阅读的兼容性和正确显示。

`markdown` 的基本语法和进阶使用请参考 [Markdown 教程](https://markdown.com.cn/)。




## 1 结构

如[表0-1](#tab0-1)所示，文档的结构由符号 `#` 定义，符合通用的 `markdown` 标准。建议采用三级标题体系，禁止出现孤立的三级标题。

章节标题之后需添加一段无标题的序言，概括本章的核心内容。

每个标题下的段落的长度区间需要合理，不宜过长或过短，适合学习者阅读并保持注意力集中。如果检查时发现过长或过短，可以考虑拆分段落或合并段落。每个一级标题段落前后需要空行。<span id="tab0-1"> </span>

<div align="center">
	<p>表0-1 文档结构层级</p>
</div>

<div align="center">

| 符号 | 结构层级 | 示例标题 |
| :--- | :--- | :--- |
| # | 章节标题 | 第一章 引言 |
| ## | 一级标题 | 1 示例一级标题 |
| ### | 二级标题 | 1.2 示例二级标题 |
| #### | 三级标题 | 1.3.3 示例三级标题 |

</div>



## 2 内容

文字内容的部分是文档的核心，文档的主要语言以中文为主，也存在部分英文作为辅助语言。所有英文的部分需要使用反引号进行限定，例如 `transformer`, `this is an English sentence`。当存在中英文交替的时候，位于句中的英文需要和两侧的中文存在一个空格的距离；若存在英文两侧存在标点或符号，则存在标点或符号的一侧不需要空格。此外，段落中如果需要首行缩进，使用符号 `&emsp;` 实现。

内容的语言表达中，统一使用“我们”作为叙述主体，并且注意表达的逻辑清晰和书面化，避免长难句的使用。

对于需要强调的正文中的文字部分，统一使用 `<strong> </strong>` 包裹；正文中出现的引用标记，统一使用 `<sup> </sup>` 进行包裹。例如，<strong>被强调的文字部分</strong>请参考<sup>[[*]](#7-参考文献)</sup>



## 3 图像

图像统一使用 `png` 格式或 `gif` 格式，保证图像的分辨率足够清晰。图中若存在文字，中文使用宋体，英文使用 `Times New Roman`。

如[图0.1](#fig0.1)所示，每个图像必须包含 `caption`，位于图像的下方居中，`caption`  的内容准确概括图像的核心信息。如果存在图像，则正文中必须包含对应的图像引用。每个章节的图像编号独立，形式为 `x.y`，`x` 为章节的编号，`y` 从 $1$ 开始，逐次递增。<span id="fig0.1"> </span>

<div align="center">
  <img src="./assets/figure.png" width="90%"/>
  <span id="Figure 1"><p>图0.1 示例图像</p></span>
</div>



## 4 表格

如[表0-2](#tab0-2)所示，每个表格必须包含 `caption`，位于表格的上方居中，`caption`  的内容描述概括表格的核心信息。如果存在表格，则正文中必须包含对应的表格引用。每个章节的表格编号独立，形式为 `x.y`，`x` 为章节的编号，`y` 从 $1$ 开始，逐次递增。<span id="tab0-2"> </span>

<div align="center">
	<p>表0-2 示例表格标题</p>
</div>

<div align="center">

| 列1 | 列2 | 列3 |
| :--- | :--- | :--- |
| 数据1 | 数据2 | 数据3 |
| 数据4 | 数据5 | 数据6 |

</div>


## 5 代码

代码块是文档中的重要组成部分，用于展示具体的实现细节和示例。代码块需要使用三重反引号包裹，并在开头标注代码语言类型，以便实现语法高亮。

### 5.1 代码块格式

代码块的基本格式如下，需要在开头的三重反引号后紧跟语言标识符（如 `python`、`bash`、`json` 等）：

```python
def hello_world():
    print("Hello, World!")
```

### 5.2 代码风格

代码风格统一遵循 [`Google Python Style Guide`](https://google.github.io/styleguide/pyguide.html)，包括函数文档字符串、类型注解和返回类型说明。

#### 5.2.1 函数文档字符串

函数必须包含文档字符串（`docstring`），使用三引号 `"""` 包裹，包含函数描述、参数说明（`Args`）、返回值说明（`Returns`）和异常说明（`Raises`，如有）：

```python
def train_model(model: torch.nn.Module, 
                dataloader: torch.utils.data.DataLoader,
                epochs: int = 10,
                learning_rate: float = 1e-4) -> dict:
    """训练深度学习模型。
    
    Args:
        model: 需要训练的神经网络模型。
        dataloader: 训练数据加载器，包含批次化的训练数据。
        epochs: 训练的轮数，默认为 10。
        learning_rate: 学习率，默认为 1e-4。
    
    Returns:
        包含训练历史的字典，包括每个 epoch 的损失值和准确率。
        示例：{'loss': [0.5, 0.3, 0.2], 'accuracy': [0.8, 0.85, 0.9]}
    
    Raises:
        ValueError: 如果 epochs 小于 1。
        RuntimeError: 如果模型训练过程中出现错误。
    """
    if epochs < 1:
        raise ValueError("epochs 必须大于等于 1")
    
    optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)
    history = {'loss': [], 'accuracy': []}
    
    for epoch in range(epochs):
        loss, acc = train_one_epoch(model, dataloader, optimizer)
        history['loss'].append(loss)
        history['accuracy'].append(acc)
    
    return history
```

#### 5.2.2 类型注解

所有函数参数和返回值都应该包含类型注解，使用 `Python` 的类型提示（`Type Hints`）：

```python
from typing import List, Tuple, Optional, Dict, Any

def process_batch(inputs: torch.Tensor, 
                  labels: torch.Tensor,
                  mask: Optional[torch.Tensor] = None) -> Tuple[torch.Tensor, float]:
    """处理单个批次的数据。
    
    Args:
        inputs: 输入张量，形状为 (batch_size, seq_len, hidden_dim)。
        labels: 标签张量，形状为 (batch_size, seq_len)。
        mask: 可选的掩码张量，形状为 (batch_size, seq_len)。
    
    Returns:
        包含两个元素的元组：
        - 输出张量，形状为 (batch_size, seq_len, vocab_size)
        - 损失值（标量）
    """
    output = model(inputs, attention_mask=mask)
    loss = criterion(output, labels)
    return output, loss.item()
```

#### 5.2.3 类的文档字符串

类也需要包含文档字符串，说明类的用途和属性：

```python
class TransformerModel(nn.Module):
    """基于 Transformer 架构的语言模型。
    
    该模型实现了标准的 Transformer 编码器-解码器结构，
    适用于序列到序列的任务。
    
    Attributes:
        vocab_size: 词汇表大小。
        d_model: 模型的隐藏层维度。
        nhead: 多头注意力的头数。
        num_layers: Transformer 层数。
        embedding: 词嵌入层。
        transformer: Transformer 主体结构。
    """
    
    def __init__(self, 
                 vocab_size: int,
                 d_model: int = 512,
                 nhead: int = 8,
                 num_layers: int = 6) -> None:
        """初始化 Transformer 模型。
        
        Args:
            vocab_size: 词汇表大小。
            d_model: 模型的隐藏层维度，默认为 512。
            nhead: 多头注意力的头数，默认为 8。
            num_layers: Transformer 层数，默认为 6。
        """
        super().__init__()
        self.vocab_size = vocab_size
        self.d_model = d_model
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.transformer = nn.Transformer(
            d_model=d_model,
            nhead=nhead,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datawhalechina/code-your-own-llm](https://github.com/datawhalechina/code-your-own-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
