---
publish: true
created: 2024-10-31T00:31:17.000+08:00
cssclasses: ""
---


## Python

### 大版本主要功能性更新

- 3.7
  - dataclasses 加入标准库
  - dict 能够保持 key 创建时的顺序进行迭代，在这一点上与 OrderedDict 一致
- 3.8
  - 海象运算符 `:=`
  - 仅位置形参 `def func(a, /, b, *, c)`：`a`无法作为关键字参数，`c`只能作为关键字参数
  - `f"{var=}"`
- 3.9
  - `|` 字典合并运算符，`|=` 字典更新运算符
  - `str.removeprefix()`, `str.removesuffix()`
  - 可以直接使用 `list[str]` 和 `dict[str, int]` 等作为类型注释
- 3.10
  - `with ... as` 支持一次创建多个上下文管理器
  - 更准确的错误提示：括号未关闭、符号缺失、缩进错误等
  - `match ... case`
  - type hint 中可以直接使用 `|` 表示或
- 3.11
  - 性能大幅提升

### 字符串前缀 stringprefix

- `r"C:\Python\Python310"`：raw 原始字符串，取消 `\` 的转义效果，如 Windows 下的路径
- `b"\xe4\xb8\xad\xe6\x96\x87"`：bytes 字节串
- `f"he is {name}"`：format 格式字符串（Python 3.6+），相当于 `"he is {}".format(name)`
- `u"中文"` ：unicode 字面值，在 Python 3 中已无实际作用

以上前缀字母均**不区分大小写**

参考链接：

- [Python 字符串前缀](https://mp.weixin.qq.com/s/STCDiP7gIwvL5JTlZ8GIpA)
- [词法分析](https://docs.python.org/zh-cn/3/reference/lexical_analysis.html#string-and-bytes-literals)

### 格式字符串

```python
"{2}, {1}, {0}, {2}".format("a", "b", "c")
# "c, b, a, c"
```

以 `{}` 标注字符串中的可替换字段

该语法在大多数情况下与旧式的 `%` 格式化类似，只是增加了 `{}` 和 `:` 来取代 `%`。 例如，`"%03.2f"` 可以被改写为 `"{:03.2f}"`。

使用字符串前缀 `f` 来构造格式字符串（f-string）时，`{}` 内部可以直接写可求值的字符串表达式（类似 `eval(str)`）。

Python 3.8 后 f-string 新增了一个功能：`f"{var=}"` 等价于 `f"var={var}"`。

参考链接：[string --- 常见的字符串操作](https://docs.python.org/zh-cn/3/library/string.html#format-string-syntax)

### 连续赋值

Python 中使用多个等号在一行代码内完成的连续赋值过程，实际上是将**最右侧**的常量或变量，对其左侧 `=` 前的各变量**从左至右**依次赋值。

### Fancy indexing

python 原生的 slice 只支持给定首尾的等间隔 index

原生 python 下，可以用列表推导式、`map` + `__getitem__`、`operator.itemgetter` 等方法实现 fancy indexing（用数组索引数组）

此外，最简单的方法是使用 `np.take(a, indices)`

但上述方法都只能获得一个新的子序列，若想要对原序列进行部分编辑（赋值）并保留其他元素结构，最佳方法是转为 `np.array` 后直接 fancy indexing 完成赋值，这种修改是数组原位修改，最后 `.tolist()` 转回 list 即可。

## PyTorch

### 广播（broadcasting）机制

参与运算的两个 Tensor 维度不一致时：从**尾部**对齐维度，对应同一维度的两个 size 只能有**相等的**、**其中之一为 1**、**其中之一不存在**这三种情况，否则报错。

广播时，首先将维数较少者的缺失维度 size 置 1，然后通过复制元素，将不同 size 的同一维度中的 1 扩展另一个较大的 size，最终得到 size 完全相同的两个 Tensor。

```python
x=torch.empty(5,1,4,1)
y=torch.empty(  3,1,1)
(x+y).size()
# torch.Size([5, 3, 4, 1])

x=torch.empty(5,2,4,1)
y=torch.empty(3,1,1)
(x+y).size()
# RuntimeError: The size of tensor a (2) must match the size of tensor b (3) at non-singleton dimension 1
```

<https://pytorch.org/docs/stable/index.html>

### Tensor 的维度顺序

torch.Tensor: `[batch, channel, height, width]`
（与 np.array 一致）

而 OpenCV 中, `cv2.imread` 读取到的图片维度为 `[height, width, channel]`

### concat v.s. stack

- `torch.cat` (`torch.concat`, `torch.concatenate`; `np.concatenate`)：沿着现有的轴合并，所有数据在*除待合并轴以外的其他维度*上 shape 均应相同，e.g. `[[3, 4], [4, 4]] --> [7, 4]`
- `torch.stack` (`np.stack`)：沿着新的轴合并，所有数据 shape 应完全相同，\*e.g.\_ `[[3, 4], [3, 4]] --> [2, 3, 4]`

### repeat v.s. tile v.s. expand

- `np.repeat` = `torch.repeat_interleave`
  - 输入希望复制的次数、希望复制的 axis/dim（不指定则默认先 `flatten` 后再复制）
  - 数组的 `shape` 仅在指定的 dim 上有变化
  - 以**单个元素**为单位复制： (1, 2, 3) --> (1, 1, 2, 2, 3, 3)
- `np.tile` = `torch.repeat` =`torch.tile`
  - 输入为各维度希望复制的次数
  - 数组 `shape` 的每一 dim 会乘上相应的复制次数
  - 以同一维度**所有元素整体**为单位复制： (1, 2, 3) --> (1, 2, 3, 1, 2, 3)
  - 三者都支持输入的 repea.ndims >= array.ndims，此时 array 将在左边 `unsqueeze` 出相应数量的新维度
  - `np.tile` 与 `torch.tile` 还支持 repeat.ndims < array.ndims，此时 repeat 数将在左边新增相应数量的 `1`，即 repeat 是右对齐的
- `torch.expand` = `torch.broadcast_to` = `np.broadcast_to`
  - 输入是扩展后的**目标尺寸**，`-1` 代表无变化
  - 只支持扩展原本为 1 的维度，或是增加新的维度（右对齐，即维度新增在左侧）
  - 共享内存，因此不应该在 `expand` 之后进行 in-place 操作，而应当先 `clone`

### einsum

- [numpy.einsum 官方文档](https://numpy.org/doc/stable/reference/generated/numpy.einsum.html)
- [torch.einsum 官方文档](https://pytorch.org/docs/stable/generated/torch.einsum.html)
- [EINSUM IS ALL YOU NEED - EINSTEIN SUMMATION IN DEEP LEARNING](https://rockt.github.io/2018/04/30/einsum)
- [einsum 初探 - 知乎](https://zhuanlan.zhihu.com/p/101157166)

```python
# from numpy import einsum
from torch import einsum
```

遵循「爱因斯坦求和约定/爱因斯坦标记法」（Einstein notation）

用字符串（以 `,` 相隔的字母）来标记张量的各个轴，只关注输入和输出（以 `->` 相隔）的维度。

在输入数组的标记之间，**重复**字母表示沿这些轴的值将**相乘**，这些乘积构成输出数组的值。

从输出标记中**省略**的字母表示沿该轴的值将被**求和**。

\*e.g.\_ 矩阵 $A$、$B$ 的乘积：

```python
einsum('ij,jk->ik', A, B)
```

> The subscripts string is a comma-separated list of subscript labels, where each label refers to a dimension of the corresponding operand. Whenever a label is repeated it is summed, so `np.einsum('i,i', a, b)` is equivalent to `np.inner(a,b)`. If a label appears only once, it is not summed, so `np.einsum('i', a)` produces a view of a with no changes. A further example `np.einsum('ij,jk', a, b)` describes traditional matrix multiplication and is equivalent to `np.matmul(a,b)`. Repeated subscript labels in one operand take the diagonal. For example, `np.einsum('ii', a)` is equivalent to `np.trace(a)`.

### einops

- [官网](http://einops.rocks/pytorch-examples.html)
- [GitHub: arogozhnikov/einops](https://github.com/arogozhnikov/einops)
- [【详解】einops 优美的处理张量维度](https://blog.csdn.net/ViatorSun/article/details/116010049)

```python
# pip install einops
import einops
```

> einops 主要包含 rearrange、reduce、repeat 等方法。采用爱因斯坦标记法，关注的是接口，即输入和输出是什么，而不是如何计算输出。

### nn.Linear 与 nn.Conv1d

参考链接：[Are fully connected and convolution layers equivalent? If so, how? – Weights & Biases](https://wandb.ai/wandb_fc/pytorch-image-models/reports/Are-fully-connected-and-convolution-layers-equivalent-If-so-how---Vmlldzo4NDgwNjY)

结论：在 Conv1d 的 `kernel_size=stride=1`, `padding=0`的情况下，其与 Linear 模块是等价的（MLP / Fully Connected），但二者的初始化和具体实现过程可能有所不同，造成深层堆叠后的计算和梯度传导结果略有一些数值差异。

注意，两个模块的**输入输出维度顺序**是不同的：

`nn.Linear` ([doc](https://pytorch.org/docs/stable/generated/torch.nn.Linear.html))：接受输入维度 [\*, $C_{in}$]，输出 [\*, $C_{out}$]，其中 \* 表示前面可以任意增加维度，均作为 batch 并行处理。

`nn.Conv1d` ([doc](https://pytorch.org/docs/stable/generated/torch.nn.Conv1d.html))：接受输入维度 [batch\_size, $C_{in}$, $L_{in}$]，输出 [batch\_size, $C_{out}$, $L_{out}$]，其中 $L$ 为数据长度，在等价 FC 时输入输出长度相同。

即：`nn.Linear`的维度变换作用于**最后一维**，`nn.Conv1d`的维度变换作用于**第二维**。

一个带 batch 的逐点三维坐标处理 MLP 示例：

```python
batch_size = 50
point_num = 1024
x = torch.randn(batch_size, point_num, 3)

fc_linear = nn.Linear(3, 256)
y_linear = fc_linear(x) # [50, 1024, 256]

fc_conv = nn.Conv1d(3, 256, kernel_size=1)
y_conv = fc_conv(x.permute(0, 2, 1)) # [50, 256, 1024]
```

### nn.BCELoss 与 nn.CrossEntropyLoss

本质上都是对交叉熵损失的实现，但内部计算方式和使用场景有所不同：

[BCELoss](https://pytorch.org/docs/stable/generated/torch.nn.BCELoss.html) $=-y\log(\hat{y}) - (1-y)\log(1-\hat{y})$， 用于二分类或者**多标签**分类（每类独立不互斥，计算总 loss 时对所有类的交叉熵取平均即可）。虽然是两项之和，但对于确定的 label $y \in \{0, 1\}$，总是只有一项不为 0。输入的预测值应当是 $[0, 1]$ 之内的概率值，即应当经过 sigmoid 函数。另有 [BCEWithLogitsLoss](https://pytorch.org/docs/stable/generated/torch.nn.BCEWithLogitsLoss.html)，相当于 Sigmoid + BCELoss。

[CrossEntropyLoss](https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html) $=-\sum y\log(\operatorname{softmax}(\hat{y}))$ 用于多分类（各类互斥）。Ground truth label 应是 ~~one-hot 向量~~ 指示真值 label 所在 index 的索引序号。相当于 LogSoftmax + [NLLLoss](https://pytorch.org/docs/stable/generated/torch.nn.NLLLoss.html)。

### Attention Mask

`nn.Transformer` 及其相关的高层 API 中，输入的 bool `mask`都是 `True` 表示不参与 attention；而 `F.scaled_dot_product_attention()` 函数中的 `attn_mask` 参数（包括一些自实现的 Attention 代码）则恰恰相反。

> If a boolean tensor is provided for any of the [src/tgt/memory]\_mask arguments, positions with a `True` value are not allowed to participate in the attention, which is the opposite of the definition for `attn_mask` in [`torch.nn.functional.scaled_dot_product_attention()`](https://pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html#torch.nn.functional.scaled_dot_product_attention "torch.nn.functional.scaled_dot_product_attention").

参考：<https://pytorch.org/docs/stable/generated/torch.nn.Transformer.html>

如果是 causal 的 mask，`nn.Transformer` 所需的 mask 形式可以这样生成：

```python
mask = torch.triu(torch.ones((N, N), dtype=torch.bool), diagonal=1)
```

即 矩阵中 `(i, j)` 位置为 `False` 就表示 token `i` 可以看到 token `j`。

### model.train() 与 eval()

训练完样本后，如果模型中包含 BN 层 或 Dropout 层，在测试集上进行测试前需要调用`model.eval()`，否则一旦有输入数据，即使并不在训练，模型也会改变权值。

`model.train()`：将模型设为 training 模式。开启 BN 和 Dropout 层的功能。

`model.eval()`：将模型设为 evaluation 模式。1. 使 BN 层不再根据输入的 batch 数据计算均值和方差，而是固定使用全部训练集的均值和方差。2. 使 Dropout 层不再随机舍弃神经元。

> `with torch.no_grad()`:
> eval 模式不会影响各层的 gradient 计算行为，即 gradient 的计算和存储与 training 模式一样，只是不进行反向传播、更新参数。
> 而 `with torch.no_grad()` 则主要是用于停止 autograd 模块的工作，以起到加速和节省显存的作用。它的作用是将该 `with` 语句包裹起来的部分停止梯度的更新，从而节省了 GPU 算力和显存，但是并不会影响 Dropout 和 BN 层的行为。
> 如果不在意显存大小和计算时间的话，仅仅使用 `model.eval()` 已足够得到正确的 validation/test 的结果；而 `with torch.no_grad()` 则是更进一步加速和节省 GPU 空间（因为不用计算和存储梯度），从而可以更快计算，也可以跑更大的 batch 来测试。

### 数据加载

`Dataset` 定义了整个数据集，`Sampler` 提供了取数据的机制（返回采样的 index），最后由 `Dataloader` 根据 `Sampler` 的返回值从 `Dataset` 中真正取出数据。

参考链接：

- [一文弄懂 Pytorch 的 DataLoader, DataSet, Sampler 之间的关系 - marsggbo - 博客园](https://www.cnblogs.com/marsggbo/p/11308889.html)
- [PyTorch 教程-5：详解 PyTorch 中加载数据的方法--Dataset、Dataloader、Sampler、collate_fn 等 - CSDN 博客](https://blog.csdn.net/qq_38962621/article/details/111146427)
- [PyTorch 源码解读之 torch.utils.data：解析数据处理全流程 - 知乎](https://zhuanlan.zhihu.com/p/337850513)

### 典型的模型训练流程

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import datasets, transforms
import numpy as np

class NeuralNetwork(nn.Module):
    def __init__(self):
        super(NeuralNetwork, self).__init__()
        self.flatten = nn.Flatten()
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10),
        )
    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits


def criterion(output, target):
    return F.binary_cross_entropy_with_logits(output, target)


device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
training_data = datasets.FashionMNIST(
    root="data",
    train=True,
    download=True,
    transform=transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,))
    ])
)
train_loader = torch.utils.data.DataLoader(training_data, batch_size=8, shuffle=True)
model = NeuralNetwork().to(device)
optimizer = optim.Adam(model.parameters())

epoches = 10
for epoch in range(epoches):
    for batch, (inputs, target) in enumerate(train_loader):
        inputs = inputs.cuda(device, non_blocking=True)
        target = torch.from_numpy(np.array(target)).float().cuda(device, non_blocking=True)

        outputs = model(inputs)
        loss = criterion(outputs, target)

        optimizer.zero_grad()   # reset gradient
        loss.backward()
        optimizer.step()

torch.save(model.state_dict(), "model_weights.pth")
# model.load_state_dict(torch.load("model_weights.pth"))
# model.eval()
```

### 可复现性 Reproducibility

```python
import os
import random
import numpy as np
import torch

def set_seed(seed):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)  # CPU
    # torch.cuda.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)  # GPU
    # torch.backends.cudnn.benchmark = False
    # torch.backends.cudnn.deterministic = True
    os.environ['PYTHONHASHSEED'] = str(seed)
    # os.environ['CUBLAS_WORKSPACE_CONFIG'] = ':4096:8'

def seed_worker(worker_id):
    worker_seed = torch.initial_seed() % 2**32
    np.random.seed(worker_seed)
    random.seed(worker_seed)

set_seed(1)

g = torch.Generator()
g.manual_seed(0)

DataLoader(
    train_dataset,
    batch_size=batch_size,
    num_workers=num_workers,
    worker_init_fn=seed_worker,
    generator=g,
)
```

参考链接：

- [PyTorch 的可重复性问题（如何使实验结果可复现）- CSDN](https://blog.csdn.net/hyk_1996/article/details/84307108)
- [PyTorch 官方文档](https://pytorch.org/docs/stable/notes/randomness.html)
- [NVIDIA CUDA Toolkit 官方文档](https://docs.nvidia.com/cuda/cublas/index.html#cublasApi_reproducibility)

### 分布式训练

[pytorch(分布式)数据并行个人实践总结——DataParallel/DistributedDataParallel - fnangle - 博客园](https://www.cnblogs.com/yh-blog/p/12877922.html)

#### 数据并行 DataParallel (DP)

```python
model = torch.nn.DataParallel(model)
model = model.cuda()
```

单进程多线程，受到 python 全局解释器锁（GIL）的限制

> 将输入的数据均分成多份，分别送到对应的 GPU 进行计算。与 Module 相关的所有数据也都会以浅复制的方式复制多份。每个 GPU 在单独的线程上将针对各自的输入数据独立并行地进行 forward 计算，然后在主 GPU 上**收集网络输出**，并计算损失。接着，主 GPU **分散损失值**给从属 GPU，每个 GPU 独立进行反向传播以计算梯度。最后，主 GPU **汇总各梯度**、进行梯度下降，并更新主 GPU 上的模型参数，再将更新后的**模型参数广播复制**到剩余的从属 GPU 中。

DP 的 batch-size 是指多卡 batch-size，即 $N$ 张卡各自的 batch-size 是原始的 $1/N$。

#### 分布式数据并行 DistributedDataParallel (DDP)

参考

- [pytorch 单机多卡：从 DataParallel 到 distributedDataParallel](https://blog.csdn.net/weixin_39718268/article/details/105021631)
- [Pytorch Distributed Data Parallal | 摸黑干活](https://fazzie-key.cool/2022/01/23/ddp/)

```shell
# python -m torch.distributed.launch --nproc_per_node=4 train.py
torchrun --nproc_per_node=4 train.py
# 指定可见 GPU 可在命令前加:
CUDA_VISIBLE_DEVICES=0,1,2,3
```

多进程（几个 GPU 就产生几个进程），充分利用性能；传输的数据量少于 DP，速度更快，效率更高

> 只需要编写一份代码，torch 就会自动将其分配给 n 个进程，分别在 n 个 GPU 上运行，每个 GPU 执行相同的任务。每个进程都从磁盘加载其自己的数据，由 DistributedSampler 确保加载的数据在各个进程之间不重叠。损失函数的前向传播和计算在每个 GPU 上独立执行，因此不需要收集网络输出。在反向传播期间，梯度下降在所有 GPU 上均被执行，然后由 `local_rank=0` 的进程**广播汇总平均后的梯度**给其他进程。最后，各进程用该梯度来独立地更新参数。由于梯度值相同，各进程的模型参数也就始终保持一致。

调用 `DistributedSampler` 直接为各进程产生数据：`train_sampler = torch.utils.data.distributed.DistributedSampler(train_data)`。若需要 shuffle，实例化时设置 `shuffle=True` 还不够，需要在 for 循环训练时将当前的 epoch 数传入：`train_sampler.set_epoch(epoch)`（后续 Dataloader 设置 `shuffle=False` 即可）。（[参考链接](https://www.zhihu.com/question/67209417/answer/1017851899)）

DDP 在设计时利用 Pytorch 中的 hook 机制实现了非侵入式的 API：在执行 `loss.backward()` 时，各个进程首先各自开始反向计算梯度，当所有进程的梯度都计算完毕后，再执行一个所有进程间的 all-reduce 梯度平均操作，最后才把平均后的梯度值写入`parameter.grad`。因此，一般来说，DDP 的反向传播过程**无需**添加任何额外的代码。

DDP 的 batch-size 是指单卡 batch-size，相当于 $N$ 张卡的总 batch-size 扩大了 $N$ 倍。

为避免冲突，日志记录和模型参数保存应该只在一个进程中执行，可以通过判断 `args.local_rank==0`（由 `torch.distributed.launch` 或 `torchrun` 自动设置）来选择主进程。此外，训练过程中的 loss 变量值和其他 metrics 信息也只是单进程的，若要汇总各进程的信息，需要手动调用 `torch.distributed.all_reduce`（DDP 在 `loss.backward()` 中自动完了这一步，但不会输出具体数值），参考[此文](https://blog.csdn.net/weixin_39718268/article/details/105021631)最后。

> 注意：当一个模型在 `batch_size=N` 的 `M` 个节点上进行训练时，与在 `batch_size=M*N` 的单个节点上训练的同一个模型相比，如果损失是在一个 batch 中的各个实例之间进行加总（而不是像往常一样进行平均），那么梯度将小 `M` 倍（因为不同节点之间的梯度是平均的）。当你想获得一个与本地训练对应的数学上等价的训练过程时，你应该考虑到这一点。但在大多数情况下，你可以只把一个分布式数据并行封装模型、一个数据并行封装模型和一个单 GPU 上的普通模型视为相同的模型（例如在同等批次大小的情况下使用相同的学习率）。

### PyTorch Lightning

`import pytorch_lightning as pl`

- [官方文档](https://pytorch-lightning.readthedocs.io/en/latest/index.html)
- [GitHub: PyTorchLightning/pytorch-lightning](https://github.com/PyTorchLightning/pytorch-lightning)

![|](https://raw.githubusercontent.com/Lightning-AI/pytorch-lightning/master/docs/source-pytorch/_static/images/general/pl_quick_start_full_compressed.gif)

将网络模块所继承的父类从 `nn.Module` 改为 `pl.LightningModule`。模块实现中，`__init__` 与 `forward` 函数与之前相同，新增了 `training_step`、`configure_optimizers`、`val_step`、`test_step` 等 API 方法，将模型的训练、验证、测试、日志记录等过程，与模型的定义封装到了一起。

`training_step` 函数需要调用 `self`（即 `forward()`）进行前向传播，然后计算损失，最终需要 `return loss`，整个过程相当于原生 PyTorch 训练时每个 epoch 的循环体。不再需要显式调用 `for` 循环、`loss.backward()`、`optmizer.zero_grad()` 和 `optimizer.step()` 等。

训练时，初始化一个 `trainer = pl.Trainer()`，调用 `trainer.fit(network, DataLoader(train), DataLoader(val))` 即可完成全部训练过程。

测试时，同样初始化一个 `trainer`，然后调用 `trainer.validate(network, DataLoader(test))` 即可。

PyTorch Lightning 默认采用 TensorBoard 进行 Log 管理。在 `training_step` 中，可以随时调用 `self.log()` 进行日志记录,也可以调用 `self.logger.experiment.add_scalars()` 等方法进行手动日志管理。

默认情况下，日志目录位于 `os.getcwd()`，可以在初始化 `trainer` 时使用 `pl.Trainer(default_root_dir='/your/path/to/save/checkpoints')` 参数进行修改。默认每 50 steps 记录一次 log。

e.g.

```python
import os
import torch
from torch import nn
import torch.nn.functional as F
from torchvision.datasets import MNIST
from torch.utils.data import DataLoader, random_split
from torchvision import transforms
import pytorch_lightning as pl

class LitAutoEncoder(pl.LightningModule):
    def __init__(self):
        super().__init__()
        self.encoder = nn.Sequential(nn.Linear(28 * 28, 128), nn.ReLU(), nn.Linear(128, 3))
        self.decoder = nn.Sequential(nn.Linear(3, 128), nn.ReLU(), nn.Linear(128, 28 * 28))

    def forward(self, x):
        # in lightning, forward defines the prediction/inference actions
        embedding = self.encoder(x)
        return embedding

    def training_step(self, batch, batch_idx):
        # training_step defines the train loop. It is independent of forward
        x, y = batch
        x = x.view(x.size(0), -1)
        z = self.encoder(x)
        x_hat = self.decoder(z)
        loss = F.mse_loss(x_hat, x)
        self.log("train_loss", loss)
        return loss

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(self.parameters(), lr=1e-3)
        return optimizer

dataset = MNIST(os.getcwd(), download=True, transform=transforms.ToTensor())
train, val = random_split(dataset, [55000, 5000])

autoencoder = LitAutoEncoder()
trainer = pl.Trainer()
trainer.fit(autoencoder, DataLoader(train), DataLoader(val))
```

## OpenCV

### 图片通道顺序

`[B,G,R]`

转为 RGB 以便使用 `matplotlib` 显示：

```python
import cv2
import matplotlib.pyplot as plt

img = cv2.imread("test.jpg")
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
plt.imshow(img)
plt.show()
```

## 3D 视觉中的相机坐标系定义

[参考链接](https://zhuanlan.zhihu.com/p/501704425)

在相机坐标系下，以相机面向物体的方向为前方，则不同框架下的三维坐标定义（均为右手系）如下：

数学中的 3D 右手系（matplotlib）一般为：`[right, forwards, up]` / `[x, y, z]`；

[OpenCV](https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/OWENS/LECT9/node2.html) / [Open3D](https://github.com/isl-org/Open3D/issues/1347) / [Matterport3D](https://github.com/niessner/Matterport/blob/master/data_organization.md) / [COLMAP](https://colmap.github.io/format.html) 坐标系相同，均为 `[right, down, forwards]` / `[x, -z, y]`（Neural Body 系列[采用此坐标系](https://github.com/zju3dv/neuralbody/issues/68#issuecomment-1012897688)）;

[OpenGL](https://learnopengl.com/Getting-started/Coordinate-Systems) 定义为 `[right, up, backwards]` / `[x, z, -y]`；

[初代 NeRF 采用的坐标系](https://github.com/Fyusion/LLFF#using-your-own-poses-without-running-colmap)为 `[down, right, backwards]` / `[-z, x, -y]`。
