---
publish: true
created: 2024-06-21T07:00:46.000Z
---

[Deep Learning cheatsheets for Stanford's CS 230](https://github.com/afshinea/stanford-cs-230-deep-learning)

![[Academic/assets/super-cheatsheet-deep-learning.pdf]]

## 神经网络与深度学习

线性回归加上 Sigmoid 函数就是逻辑回归（LR）。多个逻辑回归“串联”、“并联”就是神经网络。实际上，可以将 LR 看做是仅含有一个神经元的单层的神经网络。

### 逻辑回归（Logistic Regression）

输入 $x\in R^{n_x}$（$n_x\times 1$ 列向量），真值 $y$，预测值（概率）$\hat{y}=a=P(y=1\mid x)\in [0,1]$，权系数 $w\in R^{n_x}$，偏置 $b\in R$

$$
\begin{gathered}
z=w^Tx+b\\
\hat{y}=a=\sigma(z)
\end{gathered}
$$

### 激活（Activation）函数：Sigmoid

$$
\sigma(z)=\frac{1}{1+e^{-z}}
$$

![|400](Academic/assets/16053578218393.jpg)

其导数具有便捷的计算性质：

$$
\frac{d\sigma(z)}{dz}=\frac{1}{1+e^{-z}}(1-\frac{1}{1+e^{-z}})=\sigma(z)(1-\sigma(z))
$$

### 损失（Loss/error）函数：交叉熵（Cross Entropy）

衡量参数在单个训练样本上的表现

$$
L(a,y)=-[y\log(a)+(1-y)\log(1-a)]
$$

其导数：

$$
\begin{gathered}
\frac{dL(a,y)}{da}=-\frac{y}{a}+\frac{1-y}{1-a}\\
\frac{dL(a,y)}{dz}=(-\frac{y}{a}+\frac{1-y}{1-a})\times a(1-a)=a-y
\end{gathered}
$$

### 梯度下降法

迭代更新权系数：（$\alpha$ 为学习率）

$$
\begin{gathered}
w_1=w_1-\alpha\frac{dL}{dw_1}\\
w_2=w_2-\alpha\frac{dL}{dw_2}\\
\cdots\\
b=b-\alpha\frac{dL}{db}
\end{gathered}
$$

### 成本/代价函数（Cost）

衡量参数在全体训练样本上的表现

设有 $m$ 个训练样本：$\{(x^{(1)},y^{(1)}),\cdots,(x^{(m)},y^{(m)})\}$
（每个样本 $x^{(i)}=[x_1;x_2;\cdots;x_{n_x}]$）

预测结果为 $\hat{y}^{(i)}=a^{(i)}$

则成本函数：

$$
J(w,b)=\frac{1}{m}\sum_{i=1}^{m}L(a^{(i)},y)
$$

### 向量化逻辑回归

$$
\begin{gathered}
Z=w^T X+b\\
[z^{(1)} z^{(2)} \cdots z^{(m)}] = [w_1 w_2 \cdots w_{n_x}] [x^{(1)} x^{(2)} \cdots x^{(m)}] + [b b \cdots b]
\end{gathered}
$$

其中 $X\in R^{n_x \times m}$

$$
A=[a^{(1)} a^{(2)} \cdots a^{(m)}]=\sigma(Z)
$$

大写的 $X$ 表示多个训练样本构成的矩阵，每列为一个样本。但之前无下标的小写 $x$ 也是矩阵，由单一样本的多个输入构成。

### 神经网络

**单样本：**

![|500](Academic/assets/16057723585386.jpg)

$$
\begin{gathered}
z_i^{[l]}=w_i^{[l]T} x+b_i^{[l]}\\
a_i^{[l]}=\sigma(z_i^{[l]})
\end{gathered}
$$

其中上标 $[l]$ 表示网络的第 $l$ 层，下标 $i$ 表示该层的第 $i$ 个节点。这里带上下标的变量均为矩阵。

**每一层合并表示：**

$$
\begin{gathered}
z^{[l]}=W^{[l]}a^{[l-1]}+b^{[l]}\\
a^{[l]}=\sigma(z^{[l]})
\end{gathered}
$$

其中：$z^{[l]}=[z_1^{[l]};z_2^{[l]};\cdots;z_{n^{[l]}}^{[l]}]$，$W^{[l]}=[w_1^{[l]T};w_2^{[l]T};\cdots;w_{n^{[l]}}^{[l]T}]$（维度 $n^{[l]}\times n^{[l-1]}$），$b^{[l]}=[b_1^{[l]};b_2^{[l]};\cdots;b_{n^{[l]}}^{[l]}]$（维度 $n^{[l]}\times 1$）；

$a^{[l]}=[a_1^{[l]};a_2^{[l]};\cdots;a_{n^{[l]}}^{[l]}]$，其中：$a^{[0]}=x=[x_1;x_2;\cdots;x_{n_x}]$，$n^{[l]}$ 表示第 $l$ 层的节点数，$n^{[0]}=n_x$；用 $L$ 表示总层数（不计输入层），则输出层节点数为 $n^{[L]}$。

**多样本向量化：**

每列为一个训练样本进行堆叠： $Z^{[l]}=[z^{[l](1)} z^{[l](2)} \cdots z^{[l](m)}]$，$A^{[l]}=[a^{[l](1)} a^{[l](2)} \cdots a^{[l](m)}]$

其中 $A^{[0]}=X=[x^{(1)} x^{(2)} \cdots x^{(m)}]$

则：

$$
\begin{gathered}
Z^{[l]}=W^{[l]} A^{[l-1]}+b^{[l]}\\
A^{[l]}=\sigma(Z^{[l]})
\end{gathered}
$$

### 其他激活函数

每层可以选择不同的激活函数 $g^{[l]}(z^{[l]})$

![|600](Academic/assets/16057807184072.jpg)

- Sigmoid 一般仅用于二分类的输出层
- tanh 一般总比 Sigmoid 效果好
- ReLU （线性整流函数/修正线性单元）$=\max(0,z)$ 最常用，斜率为常数，缓解了梯度消失问题（实际上关闭了输入为负的部分神经元，为网络引入了稀疏性，提高了计算效率）
- Leaky（带泄漏的）ReLU 在输入为负时仍存在很小的斜率，因此仍能传播梯度，解决了 ReLu 的神经元死亡问题，以提高模型的表达能力

#### Softmax

常用于多分类任务。将一个向量映射到 $(0,1)$ 区间内，且各元素之和为 $1$。可理解为输出了一组**概率值**（一般称 softmax 之前网络输出的 $x_{i}$ 为 **logits**）。

$$
y_{i}=\frac{e^{x_{i}}}{\sum_{j=1}^{n} e^{x_{j}}}
$$

此外，softmax 函数还可以作为 argmax（输出 one-hot 向量指示最大值的位置）的一种光滑近似，使得取最大值位置的这一过程可导，可以在训练中完成梯度下降。

### 所谓「深度学习」

“多隐层神经网络”

### 梯度消失与梯度爆炸

梯度的反向传播过程中，根据求导的链式法则可以推导出，损失函数对浅层网络参数所求的偏导（梯度值）就是“损失函数的导数、各层激活函数的导数、各层权重值”之间的连乘形式。

当网络较深时，如果连乘因子的绝对值小于 $1$，则反向传播的梯度会以指数形式减小，导致浅隐藏层的权值更新缓慢或更新停滞，于是网络的训练就等价于只有后几层的浅层网络学习，很难优化收敛。反之，若连乘因子的绝对值大于 $1$，则反向传播的梯度会以指数形式增大，导致网络权重大幅更新，变得不稳定。

由于权重的初始化值通常都小于 $1$（0-1 高斯分布），常见的激活函数的导数也普遍小于 $1$（如 Sigmoid 函数的导数的最大值为 $0.25$），因此梯度消失问题更为常见。

解决方法：

- 更好的权重初始化方法（如 He 初始化）
- 更好的激活函数（如 **ReLU**）
- 梯度剪切：对梯度设定阈值，解决梯度爆炸
- 正则化：限制过大的权重值，解决梯度爆炸
- **Batch Normalization**：将输入值拉回标准正态分布，使其落在激活函数对输入比较敏感的区域，缓解梯度消失问题
- **残差结构**（shortcut/skip-connection）：跨接使得梯度的连乘因子中增加了恒定的 $1$，即使权重值很小也能将梯度恒等地传播下去
- LSTM 的门结构：改善 RNN 的梯度消失问题

### 正则化（Regularization）

为损失函数加上惩罚项，以防止过拟合

模型参数值越小，模型就越简单。因此为了阻止模型过分复杂化，加入的正则化项有：$L_1$ 范数（$\Vert w\Vert_1$ 绝对值之和）、$L_2$ 范数（$\Vert w\Vert_2$ 欧氏距离）的平方等。

#### Dropout

Dropout 是一种正则化技术，用于防止神经网络的过拟合。

在训练过程中，以 $p$（超参数）的概率随机失活/丢弃部分神经元，使得模型不会过度依赖某些特定的神经元，从而增强模型的泛化能力。为了使得训练和推理阶段的输出期望一致，我们需要在训练阶段对保留的神经元进行缩放，即乘以 $\frac{1}{1-p}$，这样可以抵消丢弃神经元带来的期望值减少。

在推理阶段，所有神经元都被激活（禁用 Dropout）。

#### 岭回归（Ridge Regression）

加入 $L_2$ 正则项的最小二乘（MOSSE）代价函数

$$
L(w)=\sum_{j=1}^{m} \gamma_{j}\left\|f\left(x_{j} ; w\right)-y_{j}\right\|^{2}+\sum_{k} \lambda_{k}\left\|w_{k}\right\|^{2}
$$

其中 $\gamma$ 为学习率，$w$ 为模型参数，$x$ 为输入样本，$y$ 为真值标签，$f$ 为模型所拟合的函数，$\lambda$ 为正则化系数。

### Batch 训练

batch：计算一次成本函数 cost 所使用的样本个数

无 batch：使用损失函数，每跑一个样本更新一次参数，不稳定；
整个样本作为 batch：内存占用大

mini-batch：将样本划分为一定数量的批次

### Normalization

#### BatchNorm

对第 $l$ 层同一神经元（**同一特征**）处激活前的所有 $z^{[l](i)}$（$i=1,2,\cdots,m$，$m$ 为 batch 内的样本数量）做标准化处理（均值 $0$ 方差 $1$）变为 $z^{(i)}_{norm}$，然后再进行线性的缩放和平移：$\tilde{z}^{(i)}_{norm}=\alpha z^{(i)}_{norm}+\beta$，其中 $\alpha$（缩放）与 $\beta$（平移）是新加入的可学习参数。

> 其后紧接 BN 层的 CONV 层的 bias 参数可以忽略，因为无论偏置为多少，都会被 BN 中的标准化处理去除。

![|500](Academic/assets/16304984767847.jpg)

特别地，当 $\alpha$ 恰为 batch 的标准差、$\beta$ 恰为 batch 的均值时，$\tilde{z}^{(i)}_{norm}$ 实际上就等于处理前的 $z^{(i)}$。因此平移和缩放参数的意义在于，使得网络能够自行学习最合适的 Norm 程度（将数据标准化为标准正态分布会减弱其表达能力，而平移和缩放能还原一部分的表达能力），既能通过 Norm 操作加快收敛速度，又能使数据在一定程度上保留原来学习到的分布特征（甚至可以完全还原到进行 Norm 操作之前）。

此外，在隐层进行的 BN 能够在前层参数不断更新、本层输入值发生较大变化时，保证本层输入值的均值与方差仍基本保持不变，使得深层神经网络更加稳定。另外，BN 还具有轻微的正则化作用（batch 数量有限，其均值和方差含有噪声，这就为隐层引入了一定的噪声，使得后续网络不过分依赖任何一个隐藏元）。

在网络进行测试时，测试样本很可能较少，不足以构成 batch 来直接计算均值和方差（如单个输入样本的推断）。通常的做法是利用 batch 间指数加权平均等方法，在训练集上不断更新维护运行时均值和方差（running mean & var），在测试时就直接使用该参数进行 BN 层的运算。

值得注意的是，在 **CNN** 中，**同一特征**指的是同一个卷积核生成的像素集合，即**同一通道**的所有像素，包括 batch 和宽、高维度（因为 CNN 的实质是部分参数共享的全连接层），而并非是单一像素，因此标准化所取的均值与方差是针对 channel 以外的所有维度进行的；有多少个卷积核，就需要学习多少组缩放和平移参数。

各 Norm 方式比较：([Group Normalization, Kaiming He](https://arxiv.org/abs/1803.08494))

![|600](Academic/assets/16310841917101.png)

#### LayerNorm

对每个样本在某一层的特征维度上进行标准

LN 和 BN 的区别：

1. 归一化的维度：LN 在特征维度上进行归一化，而 BN 在批次维度上进行归一化。
2. 应用场景：LN 更适用于 Recurrent Neural Networks (RNNs) 和 Transformer 等序列模型，而 BN 通常用于 Convolutional Neural Networks (CNNs)。
3. batch size 依赖性：LN 不依赖于 batch size，因此在小 batch 甚至单样本的情况下也能很好地工作。BN 依赖于较大的批量大小以稳定均值和方差的估计。

为什么 Transformer 使用 LN 而不是 BN：

- Transformer 模型通常处理变长序列数据，其批次大小可能会变化或者在推理阶段可能只有一个样本。
- BN 依赖于批次维度的均值和方差估计，因此在这种情况下表现可能不稳定。LN 则对每个样本独立进行归一化，不依赖于批次大小，因此更适合于 Transformer 这种模型。
- 此外，LN 对序列模型的时间步长无关的归一化方式有助于保持输入数据的顺序特性，从而提高模型的性能和稳定性。

#### RMSNorm

研究发现 LayerNorm 的优势在于缩放不变性（即方差的计算，让模型对于输入和权重上的偏移噪声不敏感），而不是重新居中（即均值的计算，让模型在输入和权重都被随机缩放时保持输出表示不变）。

所以，RMSNorm 省略了归一化过程中的均值计算，仅计算特征的均方根，使得算法更加简洁，而效果不减，且运算效率显著提升。此外，归一化后应用可学习的仿射变换时，通常仅保留缩放参数，而省略偏移参数。实验表明，中心化（减均值）对性能影响有限，移除后仍能保持模型表达能力。

### 梯度下降优化算法

即为了不断减小成本函数，采用何种方式更新模型参数

- SGD（Stochastic 随机梯度下降）
  固定学习率

- Momentum SGD（动量梯度下降）
  采用指数加权平均（EWMA, Exponentially Weighted Moving Average），利用梯度的均值更新参数，抑制震荡，加速收敛

- RMSProp（Root Mean Square）
  采用微分平方加权平均数，为每个参数自适应调整学习率（对历史梯度很大的参数减小学习率，对历史梯度较小的参数增大学习率），进一步修正梯度下降过程中的摆动，加快收敛

- Adam（Adaptive Moment Estimation）
  将 Momentum 与 RMSProp 结合

- AdamW（Adam + Weight decay）
  相当于 Adam + 模型参数 L2 正则化

### 次梯度（Subgradient）

参考：[凸优化笔记 16：次梯度 - 知乎](https://zhuanlan.zhihu.com/p/139083837)

在使用反向传播进行优化的深度学习算法中，传统的激活函数需要满足单调、处处可导、有界等条件。然而，许多现代的激活函数并不是处处可导的，如 ReLU，在 $x=0$ 处就不可导。于是，为了实现梯度反向传播，需要引入次梯度。

![|500](Academic/assets/20230313110854.jpg)

> 次梯度实际上是下水平集的一个支撑超平面。

每个点的次梯度是一个集合，可以在一个取值区间内任意取值（如上图左侧不可导点处的两条可取的导数线）。

通常情况下，只需应用 Weak subgradient calculus，计算其中一个次梯度就够了。如 ReLU，$x=0$ 处的次梯度 $\in [0,1]$，在工程上取 $0$ 即可。

### 重参数化技巧（Reparametrization tricks）

参考 [Gumbel-Softmax Trick - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/144140006)

在对连续分布的采样中利用重参数化技巧

VAE 中的隐变量 $z$ 一般取高斯分布，即 $z=\mathcal{N}\left(\mu, \sigma^2\right)$，然后从这个分布中采样。但是这个采样操作是不可导的，进而导致整个模型无法 BP。

应用重参数化技巧，首先从从均值为 $0$，标准差为 $1$ 的高斯分布中采样，再放缩平移得到 $z$：

$$
\mathbf{z}_i=\mu_i+\sigma_i * \epsilon, \epsilon \sim \mathcal{N}(0, \mathbf{I})
$$

这样从 $\epsilon$ 到 $z$ 只涉及了线性操作（平移缩放），采样操作在计算图之外，而 $\epsilon$ 对于网络来说只是一个常数。

## 卷积神经网络 CNN

相比于全连接神经网络，CNN 的特点在于**局部感受野**（滑窗卷积）、**参数/权值共享**（同一输出通道在滑窗过程中所用的卷积核是相同的），从而大大减少了参数量。

对比：

![|600](Academic/assets/16316064387773.png)

### 矩阵卷积

即对应元素相乘（加权）、求和、滑窗重复操作。

e.g. 边缘检测

以灰度图像为例，矩阵的每一个元素就是对应像素的灰度（亮度）值，数值越大表示越亮。

![|600](Academic/assets/16063839915187.jpg)

中间的 $3\times3$ 矩阵称为**滤波器**（filter）或**卷积核**（kernel）

卷积过程所用到的参数就是卷积核的所有元素（相当于权系数）。此外还可以为一个卷积核配置一个 bias 偏置项，因此一个 $n\times n$ 的卷积核所含的参数量为 $n\times n+1$。

图中的滤波器可以检测垂直的正边缘（亮到暗）和负边缘（暗到亮）

卷积维度规律：
$(n\times n) * (f\times f) = (n-f+1\quad\times\quad n-f+1)$

#### 填充（Padding）

为了解决经过滤波后图像缩小，以及边缘像素丢失的问题，可以对原图像进行升维，即在外围填充 $p$ 层像素（习惯上填充的是 $0$），使原图维度变为 $n+2p$。

填充像素数的选择：

- Valid：不填充
- Same/Half：使步长为 $1$ 时，输出维度与 Padding 前的原输入维度一致，即令 $n+2p-f+1=n$，因此 $p=(f-1)/2$；此法中卷积操作从卷积核的中心与输入的边角像素重合时开始（若步长为 $s$，则令输出维度等于输入 $/s$，此时 $p=(f-s)/2$）
- Full：$p=f-1$；此法中卷积操作从卷积核与输入刚相交时开始
- Arbitrary：自定义

一般 CV 中使用的滤波器维数都是奇数。

e.g. Same Padding:

![|200](Academic/assets/16310782721546.gif)

#### 步长（Stride）

即滤波器行、列移动步幅为 $s$

卷积维度规律（padding $p$ & stride $s$）：

$$
(n\times n) * (f\times f) = ([\frac{n+2p-f}{s}+1]\quad\times\quad [\frac{n+2p-f}{s}+1])
$$

其中 $[\cdot]$ 表示 $\operatorname{floor}(\cdot)$ 向下取整

e.g. stride=2:

![|200](Academic/assets/16310792749440.gif)

### 三维卷积

将多个矩阵堆叠为矩阵数组（3 维张量）

步长为 1、无填充时，三维卷积维度规律：

$$
(n \times n \times n_c) * (f \times f \times n_c) = (n-f+1 \quad \times \quad n-f+1 \quad \times \quad n_c')
$$

其中，$n_c$ 为**通道数**或深度。

滤波器与输入图像的通道数必须相同。

$n_c'$ 为输出的通道数，与一次卷积所使用的 $(f \times f \times n_c)$ 维度的**滤波器个数**相同，以此法可以一次检测多个特征。

CNN 例子：

![|600](Academic/assets/16080344387462.jpg)

图片高度 $n_H$ 和宽度 $n_W$ 不断减小（无 padding），而通道数 $n_c$ 不断增加。

最后存在一个平整化（flatten）的过程，即将三维展平成一个列向量（1 维张量）。

CNN 中有三种典型层：

- 卷积层 CONV
- 池化层 POOL
- 全连接层 FC

### 池化层 Pooling

e.g. Max / Average Pooling

相当于一层参数给定（**不需要**学习权重和偏置）、步长等于卷积核大小的卷积层。用于缩减模型大小，提高计算速度，提高提取特征的鲁棒性。

与卷积层的不同之处在于，池化操作是对输入的每个通道单独进行的（**channel-wise**），**不改变通道数**。

常用超参数：$f=2,s=2,p=0$，效果是将层高度和宽度缩减一半

### 全连接层 Fully Connected

将二维或三维 feature map 平整化后（flatten）为一维向量后，使用单层神经网络全连接到下一层向量。

### 特殊卷积核

卷积可视化示意：<https://github.com/vdumoulin/conv_arithmetic/blob/master/README.md>

#### 1x1 卷积

一个 $1\times 1 \times n_c$ 卷积核完成了将三维立体（加权）采样到二维平面的过程，在**不改变宽度和高度**的同时，完成通道数的压缩以减少计算量。

#### 3x3 卷积

在二维卷积中，两个连续 $3\times 3$ 卷积的输出大小与一个 $5\times 5$ 卷积相同（即二者感受野相同），但两次 $3\times 3$ 卷积所用的参数量更少，且在输入维度大于 $10$ 时计算量也更少。此外，主流的深度学习框架均对 $3\times 3$ 卷积做了额外的优化。

VGG 和 ResNet 等经典网络结构都倾向于使用 $3\times 3$的小卷积核。

#### 等价全连接的卷积层

$(n\times n \times n_c)$ 的图像通过 FC 连接到 $N$ 个节点上，等价于该图像通过包含 $N$ 个$(n\times n \times n_c)$（与原图尺寸相同）滤波器的 CONV 层，输出是 $(1\times 1 \times N)$。

#### 转置卷积/反卷积

在输入图的元素中间增加空洞，从而实现上采样（增大特征图尺寸）

`torch.nn.ConvTranspose2d` 实现中的步长（stride）参数指的是输入元素之间的空洞数量

![|200](Academic/assets/padding_strides_transposed.gif)

#### 空洞卷积

在卷积核中增加空洞，从而增大感受野

![|200](Academic/assets/16310785625221.gif)

#### 分组卷积与深度可分离卷积

[参考链接](https://zhuanlan.zhihu.com/p/490685194)

#### Separable Convolution

降低卷积运算参数量的一种典型方法

[参考链接](https://yinguobing.com/separable-convolution/)

核心思想是将一个完整的卷积运算分解为两步进行，分别为 Depthwise Convolution 与 Pointwise Convolution。其中，Depthwise Convolution 完全在二维平面内进行，即各通道完全独立进行 2D 卷积，只改变图像尺寸，不改变通道数；Pointwise Convolution 对每个像素点独立进行，聚合相同空间位置上各个通道的信息，本质上就是核为 $1\times 1 \times n_c$ 的 3D 卷积，不改变图像尺寸，只改变通道数。

![|400](Academic/assets/16367224146714.jpg)

上中下分别为：常规卷积；分解后的 Depthwise Convolution 和 Pointwise Convolution。

### 反卷积/转制卷积

![|200](Academic/assets/16485350920617.gif)

[参考链接](https://blog.csdn.net/disanda/article/details/105762054)

反卷积（deconvolution）是上采样的一种方式，也叫转置卷积（transposed convolution）。其与卷积过程的主要区别在于输出的图片尺寸会大于输入图片的尺寸，这是通过为输入增加 padding 来实现的。（在进行反卷积时设置的 stride 并不是指反卷积在进行卷积时候卷积核的移动步长，而是被卷积矩阵填充的 padding）

反卷积并不能还原被卷积之前的矩阵，只能从大小上进行还原。反卷积的本质还是卷积，只是在进行卷积之前，会进行一个自动的 padding 补 0，从而使得输出的矩阵和指定输出的矩阵的形状相同。

应用场景：在 GAN 中的生成器、自动编码器（Autoencoder）、语义分割等模型中，通常希望进行与正常卷积相反的转换，即执行上采样。如对于语义分割任务，需要首先用编码器提取特征图，然后用解码器恢复原始图像大小，以分类原始图像的每个像素。

### 感受野（Receptive Field）

定义：CNN 每一层的输出特征图上的一个像素点在输入图像（或其他某一层特征图）上映射的区域大小

计算方法：由最深层递推到浅层

$$
RF_i=(RF_{i+1}-1)\times stride_i+K_{size_i}
$$

其中，$i$ 为层数，$stride$ 为卷积步长，$K_{size}$ 为卷积核大小。padding 不影响感受野。
特别地，最后一层（即待分析的输出特征图）的 $RF$ 就等于该层卷积核的大小。

池化（下采样）、空洞卷积（dilated conv）都能够扩大感受野，使得每个输出像素都包含较大范围的信息。其中空洞卷积不会导致空间分辨率的下降（信息损失）：

### 典型 CNN 结构

![|500](Academic/assets/16161414016254.png)

- LeNet
- AlexNet
- VGG
- GoogleNet (Google Inception)
- ResNet

#### AlexNet

![|500](Academic/assets/16080806308854.jpg)

- 采用 ReLU 激活
- Dropout 部分神经元失活，减少过拟合

#### VGG

![|500](Academic/assets/16080807676069.jpg)

#### ResNet 残差网络

![|300](Academic/assets/16080942264170.png)

- VGG + 残差结构
- 超深网络结构（可达千层）
- 弃用 Dropout，使用 Batch Norm

假设某段神经网络的输入是 $x$，期望输出为 $H(x)$。如果添加 shorcut 直接把 $x$ 传递到输出端，那么要学习的目标就是 $H(x)-x=F(x)$，称之为**残差**。这不会引入额外的参数，因此不会增加计算复杂度。

随着网络的加深，如果在浅层已经达到了饱和的准确率，那么模型可以将深层的残差逼近为 $0$，此时该神经元相当于仅仅做了恒等映射，使得网络性能至少不会下降，解决了深层网络性能退化的问题。

## 循环神经网络 RNN

![|600](Academic/assets/16160576564896.jpg)

$$
\begin{gathered}
a^{<t>}=g\left(W_{a a} a^{<t-1>}+W_{a x} x^{<t>}+b_{a}\right)\\
\hat{y}^{<t>}=g\left(W_{y a} a^{<t>}+b_{y}\right)
\end{gathered}
$$

每个时间步输入一个 $x^{<t>}$，同时也利用了前一步的激活输出 $a^{<t-1>}$。同一层（水平方向）所有时间步中所用的网络参数（$W$、$b$）都是相同的。

各个时间步中所使用的典型网络结构有 GRU、LSTM 等。

### GRU

![|400](Academic/assets/16160638092131.jpg)
包含两个门（需训练的参数），即更新门 $u$ 和相关门 $r$，用以在更长的序列中记忆关键信息 $c$。在 GRU 中，$c$ 就是各个时间步的激活输出 $a$。

门所使用的激活函数一般为 sigmoid，即将值映射到 0 ～ 1 区间内，表示遗忘～记忆。

### LSTM

![|500](Academic/assets/16160638092421.jpg)

包含三个门，即更新门 $u$，遗忘门 $f$ 及输出门 $o$。

与 GRU 的主要区别是：使用单独的遗忘门代替了 $1-u$；对 $c$ 再进行门控激活输出 $a$。

[26. LSTM - by Tom Yeh - AI by Hand ✍️](https://aibyhand.substack.com/p/26-lstm)

## Transformer

输入词（token）、图等的表征向量（feature）称为 embedding（嵌入）

### 位置编码（PE, Position Encoding）

Transformer 模型的并行处理结构视输入序列为无序的，因此需要额外添加各输入之间的位置信息。

原始 Transformer 所使用的 Sinusoidal 绝对位置编码（APE）算法：

$$
\begin{gathered}
\operatorname{PE}(pos, 2i)=\sin \left(pos / 10000^{2i / d_{\text {model}}}\right) \\
\operatorname{PE}(pos, 2i+1)=\cos \left(pos / 10000^{2i / d_{\text {model}}}\right)
\end{gathered}
$$

其中：$pos$ 表示词语在句子中的位置；$i$ 表示词向量的位置；$d_{\text {model}}$ 表示词向量的维度。

上述公式表示在每个词语对应词向量的偶数、奇数位置分别添加频率随位置变化的正、余弦变量，以此来填满整个 $\operatorname{PE}$ 矩阵，然后将其与原始的 input embedding 相加，这样便完成位置编码的引入了。

若输入的是原始图像，则可以认为 $pos$ 和 $i$ 分别代表图像像素的行和列，这样就可以对图像中的每一个像素点进行位置编码了。

这种 PE 算法的结果是唯一的、有界的，并且它虽然是绝对位置编码，但由于正余弦函数的性质，可以从中推导出相对位置关系。

此外还存在其他多种 PE 算法，包括全参数可学习的绝对位置编码（完全随机初始化，靠网络学习自动生成）、部分参数可学习的绝对位置编码（如将 Sinusoidal PE 中的频率设为可学习参数）、相对位置编码（RPE，在计算 q、k 或加权 v 时引入，编码当前参与 attention 的两个 embedding 之间的相对位置）等。

（图源 CSwin Transformer）
![|600](Academic/assets/16311805089759.png)

### 自注意力机制（Self-Attention）

![|400](Academic/assets/16266592668505.png)

![|400](Academic/assets/16266592668756.png)

序列中的每一项 embedding 通过需要学习的参数矩阵生成 q (query)、k (key)、v (value) 三个向量（其长度均相同，是一个超参数**隐层特征大小**）。这种生成过程实质上就是对输入的每项 embedding 作三个参数不同的线性变换。

将某项的 q 与其他所有项的 k 点乘，得到其与各项的相关度（attention 分数），然后使用相关度对各项的 v 加权求和，即可得该项经过注意力融合后的特征。

此过程的 batch 矩阵化公式：

$$
\operatorname{Attention}(Q,K,V)=\operatorname{softmax}(\frac{QK^T}{\sqrt{d_k}}V)
$$

其中 $d_k$ 为 $Q$、$K$ 矩阵的列数（向量维度）

### Transformer for NLP

Encoder-Decoder 结构

![|400](Academic/assets/ModalNet-21.png)

实际应用中的 Transformer 结构往往将编码器和解码器反复堆叠，以便更好地通过注意力机制获取全局信息。

#### 多头注意力机制（Multi-Head Attention）

![|300](Academic/assets/ModalNet-20.png)

通过构造一系列并列的注意力模块，将输入映射到不同的子空间，有助于学习更加丰富的注意力表达。

#### Encoder

Add 操作引入了残差结构，有助于解决梯度消失，以便实现更深层的网络

Norm 是指 Layer Norm

Feed Forward（或称为 MLP, Multi Layer Perceptron，多层感知机）即前馈神经网络（全连接层）。注意，此处对多头的所有输出进行的都是独立但相同的矩阵变换操作。

#### Decoder

分为三个部分：Masked self-attention，交互层，前馈网络。

Masked self-attention 与 Encoder 类似，区别在于，它并非并行处理，而是类似 RNN 的序列输出，因此应当只处理当前输出所对应位置以前的输入（**测试**时自然如此，Mask 是为了在**训练**时遮盖不该用到的未来信息）。首个输入称为 BoS (Begin of Sequence)，第一步（生成第一个输出词）时仅有 BoS 这一个词作为输入，于是自己做 self-attention，其输出作为第二步的第二个输入，因此第二步时就可以做两个词之间的 self-attention，以此类推，直到最终预测出 EoS (End of Sequence)。

每一步中，自注意力层的输出再生成 Q 矩阵，输入到中间的 Multi-Head Attention 交互层，与 Encoder 的输出所生成的 K, V 矩阵做 attention。

Decoder 输出的向量与期望输出语句的词汇表（预先构建）长度相同，代表了词汇表中所有词出现在该输出位置的概率。可取概率最大值对应的词作为最终输出词汇。
