---
publish: true
created: 2024-08-17T00:35:36.000+08:00
---

## ✨ 通用网络与方法

### U-Net

> [!info]+
> **U-Net: Convolutional Networks for Biomedical Image Segmentation**
> MICCAI 2015
>
> - [Zotero](zotero://select/items/@ronneberger2015unet)
> - [URL](https://link.springer.com/chapter/10.1007/978-3-319-24574-4_28)
> - [arXiv](https://arxiv.org/abs/1505.04597)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CU-Net_MICCAI_2015.pdf)
> - [U-Net 原理分析与代码解读 - 知乎](https://www.zhihu.com/column/p/150579454)
> - [21. U-Net - by Tom Yeh - AI by Hand ✍️](https://aibyhand.substack.com/p/21-u-net)

![|600](Academic/assets/16485364559088.jpg)

初衷是用于生物医学图像分析中的语义分割，提出了一种 U 型的网络结构，可以同时获取上下文信息和位置信息。

整体来看，网络结构共有 5 层，左半部分是特征提取（Encoder，由两个 3x3 的卷积层与一个 2x2 的 max-pooling 层组成），右半部分是上采样（Decoder，由一个上采样层、特征 concat、两个 3x3 的卷积层构成）。

这个结构首先对输入图片进行卷积和池化（原论文中进行了 4 次），然后再逐层恢复原始的分辨率，其中使用 skip-connection 拼接了 Encoder 所产生的不同层次特征图，以融合底层特征的位置信息与深层特征的语义信息。注意，与 ResNet 的残差相加不同，U-Net 采用了拼接的方式，保留了更多的维度/位置信息，这使得后续层级可以在浅层特征与深层特征自由选择，这对语义分割任务来说更有优势。

例如，输入图片是 224x224 的，经过 Encoder 就会产生 112x112、56x56、28x28、14x14 四个不同尺寸的特征图。然后对 14x14 的特征图做上采样（反卷积或双线性插值），得到一个 28x28 的特征图，将其与 Encoder 产生的 28x28 特征图进行通道上的 concat，然后再对拼接之后的特征图做卷积和上采样，得到 56x56 的特征图，再与之前的 56x56 的特征拼接。以此类推。经过 4 次上采样即可得到一个与输入图像尺寸相同的 224x224 的预测结果。

### RoI Pooling

RoI: Region of Interest

#### RoI Pooling

输入：一个固定大小的 feature map，一个 RoI（bbox），期望输出的 feature map 大小 k\*k

将大小不同的 feature map 池化成大小相同的 feature map，利于输出到下一层网络中。

1. 获得 RoI 在输入 feature map 上对应的坐标
2. 将 RoI 划分成 k\*k 个区域，然后做 max pooling

![|500](Academic/assets/16274421599572.gif)

两步操作中，若无法等分，则均需进行量化，因此会产生误差

#### RoI Align

若遇非等分情况，不进行量化，而采用双线性插值计算。

#### Precise RoI Pooling

- **Paper**: [Acquisition of Localization Confidence for Accurate Object Detection](https://openaccess.thecvf.com/content_ECCV_2018/html/Borui_Jiang_Acquisition_of_Localization_ECCV_2018_paper.html) (ECCV 2018)

第一步量化采用双线性插值，第二步则不再划分子区域，而是使用二重积分再求均值的方式实现 pooling。

![|600](Academic/assets/16274429406700.png)

## ✨ Transformer

### Transformer

> [!info]+
> **Attention is All you Need**
> NIPS 2017
>
> - [Zotero](zotero://select/items/@vaswani2017attention)
> - [URL](https://proceedings.neurips.cc/paper_files/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html)
> - [arXiv](https://arxiv.org/abs/1706.03762)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CTransformer_NIPS_2017.pdf)
> - [The Illustrated Transformer – Jay Alammar – Visualizing machine learning one concept at a time.](https://jalammar.github.io/illustrated-transformer/)
> - [The Illustrated Transformer【译】于建民的博客-CSDN 博客](https://blog.csdn.net/yujianmin1990/article/details/85221271)

![[DL#Transformer for NLP|Transformer for NLP]]

Encoder:

![|800](Academic/assets/transformer_multi-headed_self-attention-recap.png)

Decoder:

![|800](Academic/assets/transformer_decoding_1.gif)

![|800](Academic/assets/transformer_decoding_2.gif)

注意，训练阶段时，Decoder 最下方的输入是 output GT 右移一位的结果，第一个词是\[BOS] (Begin of Sequence)，后续词与真正的 decoder output（可能有错）无关。虽然 decoder 理应串行地分多个时间步完成输入输出，但由于训练时 output GT 完全已知，所以可以把这一过程 batch 化，只需要用一个下三角的 mask 矩阵挡住”理应未知“的后续 output score（softmax 前）即可。

而测试阶段时，真正将 decoder output（可能有错）输回给 decoder 来预测下一个词，是多个时间步串行的，因此也就不再需要 mask 了。

### 🔥 RoPE

> [!info]+
> **RoFormer: Enhanced Transformer with Rotary Position Embedding**
> Neurocomputing 2024
>
> - [Zotero](zotero://select/items/@su2024roformer)
> - [URL](https://www.sciencedirect.com/science/article/pii/S0925231223011864)
> - [arXiv](https://arxiv.org/abs/2104.09864)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CRoFormer_Neurocomputing_2024.pdf)
> - [Code](https://github.com/ZhuiyiTechnology/roformer)

[让研究人员绞尽脑汁的 Transformer 位置编码 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/8130)

[Transformer 升级之路：1、Sinusoidal 位置编码追根溯源 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/8231)

[Transformer 升级之路：2、博采众长的旋转式位置编码 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/8265/comment-page-1)

[Transformer 升级之路：4、二维位置的旋转式位置编码 - 科学空间|Scientific Spaces](https://kexue.fm/archives/8397)

> 一般来说，绝对位置编码具有实现简单、计算速度快等优点，而相对位置编码则直接地体现了相对位置信号，跟我们的直观理解吻合，实际性能往往也更好。由此可见，如果可以通过绝对位置编码的方式实现相对位置编码，那么就是“集各家之所长”、“鱼与熊掌兼得”了。Sinusoidal 位置编码隐约做到了这一点，但并不够好。

思想：把绝对位置映射成一个旋转角度，然后将待编码的向量旋转过去。

由于复指数运算的性质，对 q 和 k 向量**乘**上这样具有**绝对位置**意义的旋转矩阵后（注意，经典的 Sinusoidal PE 或可学习 PE 是**加**在输入向量上的），在后续通过内积计算 attention score 时，位置编码部分将只与**相对位置**有关。

注意，由于总是在 qk 向量上加入位置编码、qk 又只用来求内积，因此 RoPE 并不包含绝对位置信息，只是利用绝对位置的方式实现了相对位置编码。当然，如果首个 token 的固定的（如 cls），那么可以借助它来定位绝对位置。

具体实现：

在一个一维序列中，对于处于第 $m$ 个位置的 $d$ 维的 q 或 k 向量 $[q_{0},q_{1},\cdots,q_{d-1}]$，其 RoPE 计算为：

$$
\begin{equation}\scriptsize{\underbrace{\begin{pmatrix}
\cos m\theta_0 & -\sin m\theta_0 & 0 & 0 & \cdots & 0 & 0 \\
\sin m\theta_0 & \cos m\theta_0 & 0 & 0 & \cdots & 0 & 0 \\
0 & 0 & \cos m\theta_1 & -\sin m\theta_1 & \cdots & 0 & 0 \\
0 & 0 & \sin m\theta_1 & \cos m\theta_1 & \cdots & 0 & 0 \\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & 0 & 0 & \cdots & \cos m\theta_{d/2-1} & -\sin m\theta_{d/2-1} \\
0 & 0 & 0 & 0 & \cdots & \sin m\theta_{d/2-1} & \cos m\theta_{d/2-1} \\
\end{pmatrix}}_{\boldsymbol{\mathcal{R}}_m} \begin{pmatrix}q_0 \\ q_1 \\ q_2 \\ q_3 \\ \vdots \\ q_{d-2} \\ q_{d-1}\end{pmatrix}}\end{equation}
$$

由于 $\boldsymbol{\mathcal{R}}_m$ 的稀疏性，实际计算中不采用矩阵乘法，而采用逐元素乘积来实现：

$$
\begin{equation}\begin{pmatrix}q_0 \\ q_1 \\ q_2 \\ q_3 \\ \vdots \\ q_{d-2} \\ q_{d-1}
\end{pmatrix}\otimes\begin{pmatrix}\cos m\theta_0 \\ \cos m\theta_0 \\ \cos m\theta_1 \\ \cos m\theta_1 \\ \vdots \\ \cos m\theta_{d/2-1} \\ \cos m\theta_{d/2-1}
\end{pmatrix} + \begin{pmatrix}-q_1 \\ q_0 \\ -q_3 \\ q_2 \\ \vdots \\ -q_{d-1} \\ q_{d-2}
\end{pmatrix}\otimes\begin{pmatrix}\sin m\theta_0 \\ \sin m\theta_0 \\ \sin m\theta_1 \\ \sin m\theta_1 \\ \vdots \\ \sin m\theta_{d/2-1} \\ \sin m\theta_{d/2-1}
\end{pmatrix}\end{equation}
$$

其中，$\theta_{i}$ 沿用了 Sinusoidal 位置编码的方案，即 $\theta_{i}=10000^{−2i/d}$。

拓展到二维图像，对于每个坐标位置为 $(x,y)$ 、展平后长度为 $d$ 的向量，将它分为两半，一半施加 x 维度的一维 RoPE，一半施加 y 维度的一维 RoPE：

$$
\tiny
\begin{pmatrix}
\cos x \theta_0 & -\sin x \theta_0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0 & \cdots & 0 & 0  \\
\sin x \theta_0 & \cos x \theta_0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0 & \cdots & 0 & 0  \\
0 & 0 & \cos x \theta_1 & -\sin x \theta_1 & \cdots & 0 & 0 & 0 & 0 & \cdots & 0 & 0  \\
0 & 0 & \sin x \theta_1 & \cos x \theta_1 & \cdots & 0 & 0 & 0 & 0 & \cdots & 0 & 0  \\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots  \\
0 & 0 & 0 & 0 & \cdots & \cos x \theta_{d / 4-1} & -\sin x \theta_{d / 4-1} & 0 & 0 & \cdots & 0 & 0  \\
0 & 0 & 0 & 0 & \cdots & \sin x \theta_{d / 4-1} & \cos x \theta_{d / 4-1} & 0 & 0 & \cdots & 0 & 0\\
0 & 0 & 0 & 0 & \cdots & 0 & 0 & \cos y \theta_0 & -\sin y \theta_0 & \cdots & 0 & 0\\
0 & 0 & 0 & 0 & \cdots & 0 & 0 & \sin y \theta_0 & \cos y \theta_0 & \cdots & 0 & 0\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots  \\
0 & 0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0 & \cdots & \cos y \theta_{d / 4-1} & -\sin y \theta_{d / 4-1}\\
0 & 0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0 & \cdots & \sin y \theta_{d / 4-1} & \cos y \theta_{d / 4-1}
\end{pmatrix} \begin{pmatrix}
q_{0,0} \\
q_{0,1} \\
q_{0,2} \\
q_{0,3} \\
\vdots \\
q_{d/4-1,d/4-2} \\
q_{d/4-1,d/4-1} \\
q_{d/4-1,d/4} \\
q_{d/4-1,d/4+1} \\
\vdots \\
q_{d/2-1,d/2-2} \\
q_{d/2-1,d/2-1}
\end{pmatrix}
$$

高效实现形式：

$$
\begin{pmatrix}
q_{0,0} \\
q_{0,1} \\
q_{0,2} \\
q_{0,3} \\
\vdots \\
q_{d/4-1,d/4-2} \\
q_{d/4-1,d/4-1} \\
q_{d/4-1,d/4} \\
q_{d/4-1,d/4+1} \\
\vdots \\
q_{d/2-1,d/2-2} \\
q_{d/2-1,d/2-1}
\end{pmatrix} \otimes \begin{pmatrix}
\cos x \theta_0 \\
\cos x \theta_0 \\
\cos x \theta_1 \\
\cos x \theta_1 \\
\vdots \\
\cos x \theta_{d / 4-1} \\
\cos x \theta_{d / 4-1} \\
\cos y \theta_0 \\
\cos y \theta_0 \\
\vdots \\
\cos y \theta_{d / 4-1} \\
\cos y \theta_{d / 4-1}
\end{pmatrix}+\begin{pmatrix}
-q_{0,1} \\
q_{0,0} \\
-q_{0,3} \\
q_{0,2} \\
\vdots \\
-q_{d/4-1,d/4-2} \\
q_{d/4-1,d/4-1} \\
-q_{d/4-1,d/4} \\
q_{d/4-1,d/4+1} \\
\vdots \\
-q_{d/2-1,d/2-2} \\
q_{d/2-1,d/2-1}
\end{pmatrix} \otimes \begin{pmatrix}
\sin x \theta_0 \\
\sin x \theta_0 \\
\sin x \theta_1 \\
\sin x \theta_1 \\
\vdots \\
\sin x \theta_{d / 4-1} \\
\sin x \theta_{d / 4-1} \\
\sin y \theta_0 \\
\sin y \theta_0 \\
\vdots \\
\sin y \theta_{d / 4-1} \\
\sin y \theta_{d / 4-1}
\end{pmatrix}
$$

### 🔥 KV Cache

在 Transformer 推理阶段，对于主流的自回归 LLM 所用的 Causal Attention，在 token by token 递归生成时，新预测出来的第 $t+1$ 个 token，并不会影响到已经算好的 $k_{≤t}$、$v_{≤t}$，因此这部分结果我们可以缓存下来供后续生成调用，避免不必要的重复计算，这就是所谓的 KV Cache。

为了节省 KV Cache，后续工作将 Transformer 的 Multi-Head Attention （MHA）改进为：

- MQA（Multi-Query Attention）：在不同 head 之间共享 kv
- GQA（Grouped-Query Attention）：将 heads 分组，同一组内的 heads 共享 kv
- MLA（Multi-head Latent Attention）：将输入先低秩投影为所有 head 共享的、更低维的 latent，再计算 kv，以减小 kv 参数矩阵的大小

详细参考：[缓存与效果的极限拉扯：从 MHA、MQA、GQA 到 MLA - 科学空间|Scientific Spaces](https://www.spaces.ac.cn/archives/10091/comment-page-1)

### 🔥 Decoder-only

[为什么现在的 LLM 都是 Decoder-only 的架构？ - 科学空间|Scientific Spaces](https://kexue.fm/archives/9529)

所谓 decoder-only，是由于 next-token-prediction 的性质，导致不再需要 seq2seq 中的 encoder 部分（因为 embedding 不存在类似翻译任务的 source & target 关系，而是统一空间下的时序前后关系）。

decoder-only 的网络结构（Masked Multi-Head Attention + Feed Forward）有两种理解方式：

1. 使用的是 Transformer Decoder 层，只不过去掉了中间的 Encoder-Decoder Cross-Attention 层（因为不存在 Encoder）
2. 使用的是 Transformer Encoder 层，只不过将 Multi-Head Attention 换成了 Masked Multi-Head Attention（causal）

decoder-only 结构有如下优势：

- encoder 的双向注意力存在低秩问题，而 decoder 的 masked 单向注意力是满秩的三角阵
- decoder-only 的 zero-shot 性能更好
- decoder-only 支持复用 KV Cache，推理效率更高

[参考](https://wdndev.github.io/llm_interview_note/#/01.%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E5%9F%BA%E7%A1%80/LLM%E4%B8%BA%E4%BB%80%E4%B9%88Decoder%20only%E6%9E%B6%E6%9E%84/LLM%E4%B8%BA%E4%BB%80%E4%B9%88Decoder%20only%E6%9E%B6%E6%9E%84?id=llm%e4%b8%ba%e4%bb%80%e4%b9%88decoder-only%e6%9e%b6%e6%9e%84)

### MoE: Mixture of Experts

[MoE 环游记：1、从几何意义出发 - 科学空间|Scientific Spaces](https://www.spaces.ac.cn/archives/10699)

### DETR: DEtection TRansformer

> [!info]+
> **End-to-End Object Detection with Transformers**
> ECCV 2020
>
> - [Zotero](zotero://select/items/@carion2020endtoend)
> - [URL](https://link.springer.com/chapter/10.1007/978-3-030-58452-8_13)
> - [arXiv](https://arxiv.org/abs/2005.12872)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CDETR_ECCV_2020.pdf)

DETR 将目标检测任务视为一个**图像到集合**的问题，即给定一张图像，输出一个包含所有可能目标的无序集合（与 anchor 作用类似），因此可以利用 Transformer 结构（本质上就是序列的转换）进行处理。

![|500](Academic/assets/16266598332250.png)

其中 Object queries 是一个全参数可学习的 embedding 集合（随机初始化），也被称为 output positional encoding（与 spatial positional encoding 不同），其意义类似于对 anchor（人为设定数量但不给定位置）的编码。

这里的 decoder 不同于原始 transformer 的序列串行结构，而是并行处理数据的。

### 🔥 ViT

> [!info]+
> **An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale**
> ICLR 2021
>
> - [Zotero](zotero://select/items/@dosovitskiy2021image)
> - [arXiv](https://arxiv.org/abs/2010.11929)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CViT_ICLR_2021.pdf)

![|500](Academic/assets/16311697858677.png)

将输入图像划分为数个 patch（相当于 token），每个 patch 都被拉平为一维序列送入 Transformer Encoder

需要超大数据集训练

ViT 只使用了 Transformer 的 encoder，首先将图片切分为 patch（每个 patch 由 $16 \times 16$ 像素构成），每个 patch 线性投影（或使用 CNN 提取特征后展平）成 embedding 向量，并额外加入一个可学习的 cls token，一并输入 encoder。经过 self attention，每一个输出的 embedding 都融合了图片的全局信息。最后抽出 cls 对应的输出，作为图像的特征，后接 MLP 用于分类（也可以不加 cls，把所有 patch 的输出平均作为全图特征，性能差别不大）。

### Swin Transformer

> [!info]+
> **Swin Transformer: Hierarchical Vision Transformer using ShiftedWindows**
> ICCV 2021 BestPaper
>
> - [Zotero](zotero://select/items/@liu2021swin)
> - [URL](https://openaccess.thecvf.com/content/ICCV2021/html/Liu_Swin_Transformer_Hierarchical_Vision_Transformer_Using_Shifted_Windows_ICCV_2021_paper.html)
> - [arXiv](https://arxiv.org/abs/2103.14030)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CSwin_Transformer_ICCV_2021.pdf)

**S**hifted **win**dows

借鉴了 CNN 的 local 与 hierarchical 思想

![|600](Academic/assets/16311763985680.png)

初始时每个 patch 包含 4×4 个像素（构成一个 embedding），7×7 个 patch 组合为 window，只在每个 window 的内部做 self-attention。计算复杂度由 ViT 的平方级最低可降为线性级。

Shifted Window 的意义在于实现跨 windows 的信息融合。

Patch merging 的过程是：将 2×2 个窗口范围内的 patch 对应的所有特征向量 concat 起来（长度变为 4 倍），再用线性层减少其维度（减少一半，变为最初的 2 倍）。这一过程中，每个窗口所含的 patch 数固定（7×7），而 patch 的数量在不断减少，单个 patch （对应的感受野）在不断扩大（增加长距离依赖，最终扩大至单个窗口覆盖全图）。这类似于 CNN 中不断扩大感受野的降采样过程。

## ✨ Object Detection

[一文读懂目标检测：R-CNN、Fast R-CNN、Faster R-CNN、YOLO、SSD\_rcnn-CSDN 博客](https://blog.csdn.net/v_july_v/article/details/80170182)

### Faster R-CNN

> [!info]+
> **Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks**
> NIPS 2015; KaimingHe
>
> - [Zotero](zotero://select/items/@ren2015faster)
> - [URL](https://proceedings.neurips.cc/paper/2015/hash/14bfa6bb14875e45bba028a21ed38046-Abstract.html)
> - [arXiv](https://arxiv.org/abs/1506.01497)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CFaster_R-CNN_NIPS_2015.pdf)

### YOLO

> [!info]+
> **You Only Look Once: Unified, Rea-Time Object Detection**
> CVPR 2016
>
> - [Zotero](zotero://select/items/@redmon2016you)
> - [URL](https://www.cv-foundation.org/openaccess/content_cvpr_2016/html/Redmon_You_Only_Look_CVPR_2016_paper.html)
> - [arXiv](https://arxiv.org/abs/1506.02640)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CYOLOv1_CVPR_2016.pdf)
> - [目标检测 yolo 系列-CSDN 博客](https://blog.csdn.net/u012655441/article/details/108042286)

## ✨ Self-Supervised Learning

**自监督学习**

[Contrastive Self-Supervised Learning | Ankesh Anand](https://ankeshanand.com/blog/2020/01/26/contrative-self-supervised-learning.html)

> Self-Supervised Learning = Unsupervised Pre-train + Supervised Fine-tune

在预训练阶段不涉及任何特定任务（task-agnostic）；在微调阶段涉及下游任务（task-specific）

自监督学习主要分为两大类：生成式 Generative（预测式 Predictive）方法和对比式（Contrastive）方法。二者主要区别在于设置的 Pretext Task 类型。Pretext Task 是指这样一种任务：这种任务并非我们所真正关心的，但是通过完成它们，我们能够学习到一种很好的表征，这种表征对下游任务很重要。
例子：生成式方法：BERT 的填空任务、VAE 和 GAN 的解码重构任务；对比学习方法：正负样本区分（instance discrimination）任务。

### MoCo: Momentum Contrast

> [!info]+
> **Momentum Contrast for Unsupervised Visual Representation Learning**
> CVPR 2020; Facebook, KaimingHe
>
> - [Zotero](zotero://select/items/@he2020momentum)
> - [URL](https://openaccess.thecvf.com/content_CVPR_2020/html/He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning_CVPR_2020_paper.html)
> - [arXiv](https://arxiv.org/abs/1911.05722)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CMoCo_v1_CVPR_2020.pdf)
> - [Self-Supervised Learning 超详细解读 (四)：MoCo 系列解读 (1) - 知乎](https://zhuanlan.zhihu.com/p/382763210)

MoCo 关注的重点是样本数量对学习到的质量的影响

对比学习所使用的 Contrastive loss 的目标就是使相似的样本之间的距离尽可能小，不相似样本之间的距离尽可能大（存在给定的上界）。MoCo 使用数据增强方式生成正负样本，属于同一张原图的两个增强版本是正样本，不同原图的两个增强版本是负样本。

![|600](Academic/assets/16486213962158.jpg)

在原始的 end-to-end 自监督学习方法中，对于一张输入图像，正样本是其数据增强版本，负样本就是同一 mini-batch 内的其他样本，其规模（字典大小）就是 batch size，模型使用两个 encoder（共享或不共享参数皆可）来将输入的两个样本分别编码为 query 和 key, 然后通过计算其内积求 loss 更新 encoder 参数。这种在线训练模式下，负样本的规模受到算力的限制。

针对这一问题，改进后的离线训练模式建立了一个 memory bank 存储所有样本的表征，每次从中随机采样得到样本，因此一定程度上可以认为每次所采样的负样本能代表所有样本。每个 mini-batch 更新完 encoder 参数后，把本次 encoder 计算出的这部分样本表征更新回 bank 中。然而带来的问题是，bank 中的表征和参数更新存在不一致（最新的采样得到的 key 可能是很多步之前的 encoder 编码得到的，存在一定滞后性）。

为了解决上述两种模型的问题，MoCo 添加了 Momentum Encoder，同时构建了一个动态进出的队列，维护最近几个 mini-batch 中样本的表征，目标是构建一个大的且能在训练过程中保持一致性的 dictionary。

具体来说，设 batch size 的大小是 $N$，构建一个大小为 $K$（$K$ 一般是 $N$ 的数倍）的队列，以及一个 encoder $f_q$ 和一个 momentum encoder $f_k$，二者网络结构和初始参数相同。对于一个 batch 的 $N$ 张图片，分别进行两种不同的随机数据增强（不同的 crop），得到 $x_q$ 和 $x_k$（各有 $N$ 张图片），二者分别通过 $f_q$ 和 $f_k$ 编码为 $q$ 和 $k$，然后计算 $N$ 张图片自身的两个增强版本之间的匹配度（$N\times 1$ 正样本），以及 $N$ 张图片和队列中的 $K$ 张图片的匹配度（$N\times K$ 负样本），拼接这两个匹配度（$N\times (K+1)$），计算 [InfoNCE](https://zhuanlan.zhihu.com/p/334772391) (Noise Contrastive Estimation) loss（相当于使用多分类任务的 [CrossEntropyLoss](https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html)，输入 $(K+1)$ 个 logits，真值 label index 始终为 0），反向传播梯度下降直接更新 $f_q$ 的参数，而 $f_k$ 的参数则采用 Momentum update 方法逐步复制 $f_q$ 的参数（因为文章假设 key 特征的表示方法是稳定的，所以它更新得更慢）。最后，更新队列：将最旧的一个 batch 从队列中移除，本次新 batch 的表征被加入队列，以保证队列中的负样本都来自于最新的 $f_k$。

[MoCo v2](https://arxiv.org/abs/2003.04297) 移植了 SimCLR 的两个提点方法：使用 Projection head；使用更强大的数据增强策略（新增了高斯模糊）。

[Moco v3](https://openaccess.thecvf.com/content/ICCV2021/html/Chen_An_Empirical_Study_of_Training_Self-Supervised_Vision_Transformers_ICCV_2021_paper.html) (ICCV 2021) 采用了 ViT 作为 encoder，且不再使用之前所改进的 memory queue，而是使用 SimCLR 的 large batch 策略。

### SimCLR: A Simple Framework for Contrastive Learning of Visual Representations

> [!info]+
> **A Simple Framework for Contrastive Learning of Visual Representations**
> ICML 2020
>
> - [Zotero](zotero://select/items/@chen2020simple)
> - [URL](http://proceedings.mlr.press/v119/chen20j.html)
> - [arXiv](https://arxiv.org/abs/2002.05709)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CSimCLR_ICML_2020.pdf)
> - [Self-Supervised Learning 超详细解读 (二)：SimCLR 系列 - 知乎](https://zhuanlan.zhihu.com/p/378953015)

SimCLR 关注的重点是正负样例的构建方式，同时还探究了非线性层在对比学习中的作用，并分析了 batch size 大小、训练轮数等超参数对对比学习的影响。

构建正负样本的数据增强方式有 3 种：随机裁剪之后再 resize 成原来的大小；随机色彩失真；随机高斯模糊。将增强后的两张图片 $x_i$, $x_j$ 输入到两个共享参数的 Encoder（ResNet-50）中，得到两个表征 $h_i$, $h_j$，再令它们继续通过两个共享参数的 Projection head（MLP）进一步提取特征 $z_i$, $z_j$，然后使用此特征的余弦相似度衡量两个样本的相似性：$s_{i,j}=\frac{{z_i}^\top {z_j}}{\tau ||{z_i}||||{z_j}||}$，其中 $\tau$ 是可调节的 Temperature 参数。最后，计算 NT-Xent loss (Normalized Temperature-Scaled Cross-Entropy Loss) 并更新参数。

预训练完成后，Projection head 被抛弃，而直接将 encoder 及其输出的表征 $h$ 用于下游任务。

[SimCLR v2](https://proceedings.neurips.cc/paper/2020/hash/fcbc95ccdd551da181207c0c1400c655-Abstract.html) (NIPS 2020) 增强了 encoder 和 projection head 的结构，借鉴了 MoCo 的 momentum encoder 和 queue 机制，并在 Unsupervised Pre-train、Supervised Fine-tune 之后，增加了 Distillation Using Unlabeled Data，使用无标签数据以一种 task-specific 的方式蒸馏 Encoder，得到更小的 Encoder。这是因为，尽管模型越大能够学到越 general 的表征，但这是在不涉及下游任务的 task-agnostic 的情况下，一旦确定了下游任务，就不再需要大模型了，可以蒸馏成一个小模型。

### BYOL: Bootstrap Your Own Latent

> [!info]+
> **Bootstrap Your Own Latent - A New Approach to Self-Supervised Learning**
> NIPS 2020; DeepMind
>
> - [Zotero](zotero://select/items/@grill2020bootstrap)
> - [URL](https://proceedings.neurips.cc/paper/2020/hash/f3ada80d5c4ee70142b17b8192b2958e-Abstract.html)
> - [arXiv](https://arxiv.org/abs/2006.07733)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CBYOL_NIPS_2020.pdf)
> - [如何评价 Deepmind 自监督新作 BYOL？ - 知乎](https://www.zhihu.com/question/402452508)

对比学习通常使用的损失函数 InfoNCE 的公式可以推导成两个部分：alignment（相似的样本具有相似的特征）和 uniformity（所有特征应尽量均匀分布在向量空间中以保留更多的信息），其中 alignment 部分只跟正样本对相关，希望将它们的特征拉近；uniformity 部分只跟负样本对相关，希望所有点的特征尽可能均匀分布在单位超球面上。二者缺一不可，如果只有 uniformity，没有 alignment，那么模型没有聚类的能力；如果只有 alignment，没有 uniformity，那么容易使得模型将所有的输入输出相同表示，也就是形成表征崩塌 collapse / **退化解**（[详解](https://zhuanlan.zhihu.com/p/365700730)）。

BYOL 是首个提出只使用正样本进行对比学习的算法。由于不存在负样本，BYOL 极易产生退化解。为此，BYOL 使用了 momentum encoder、predictor（MLP）和 stop gradient 来避免退化解的形成。

Motivation: 首先随机初始化一个网络, 不训练 (它的准确率是 1.4%, 相当于瞎猜), 直接用它当做 target network；然后训练另外一个 online network, 让这个网络去贴近 target network。当训练结束的时候, online network 的准确率竟然达到了 18.8%，青出于蓝而胜于蓝。于是我们可以考虑不断进行两个网络之间的相互学习。

模型有两个分支：online network（encoder + predictor / student）、target network（momentum encoder / teacher）。给两个网络输入同一张图片的不同数据增强版本（正样本）, 我们希望二者的输出能够尽可能接近。online network 采用梯度下降更新（student 模仿 teacher）；target network 采用 moving average 逐渐复制 online network 的参数（即 MoCo 中的 momentum encoder），这是为了使得 teacher 不要太快跟上 student 的步伐（[mean teacher](https://arxiv.org/abs/1703.01780)）。

### SimSiam: Simple Siamese

> [!info]+
> **Exploring Simple Siamese Representation Learning**
> CVPR 2021 Oral; Facebook, KaimingHe
>
> - [Zotero](zotero://select/items/@chen2021exploring)
> - [URL](https://openaccess.thecvf.com/content/CVPR2021/html/Chen_Exploring_Simple_Siamese_Representation_Learning_CVPR_2021_paper.html)
> - [arXiv](https://arxiv.org/abs/2011.10566)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CSimSiam_CVPR_2021.pdf)
> - [SimSiam：孪生网络表征学习的顶级理论解释 - 知乎](https://zhuanlan.zhihu.com/p/452659570)

![|500](Academic/assets/16486942608188.jpg)

SimSiam 在 BYOL 的基础上进一步进行实验，分别对 momentum encoder、predictor 和 stop gradient 等关键组件进行消融实验，最终发现 stop gradient 才是避免形成退化解的关键。因此，SimSiam 去掉了 momentum encoder，在仅使用 stop gradient 的极简的情况下，依然能够避免形成退化解（由于 stop gradient，teacher branch 与 student branch 的更新不同步，teacher 相对于 student 是不变的，因此能够避免退化解的产生）。

> 带 stop-gradient 的孪生网络表征学习的本质是机器学习中经典的 Expectation-Maximization（EM）算法，即在解决同时估计两个变量这一困难问题时，先固定其中一个变量，估计另一个变量，然后交替迭代更新，循环往复直至收敛。具体实现只是在计算对称 loss 时简单地分别对其中一个特征作 `detach`。

### BEiT: Bidirectional Encoder Representation from Image Transformers

> [!info]+
> **BEiT: BERT Pre-Training of Image Transformers**
> ICLR 2022 Oral; Microsoft
>
> - [Zotero](zotero://select/items/@bao2022beit)
> - [arXiv](https://arxiv.org/abs/2106.08254)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CBEiT_ICLR_2022.pdf)
> - [Self-Supervised Learning 超详细解读 (三)：BEiT：视觉 BERT 预训练模型 - 知乎](https://zhuanlan.zhihu.com/p/381345343)

> NLP 领域的自监督预训练模型：
>
> - [GPT](https://www.cs.ubc.ca/~amuham01/LING530/papers/radford2018improving.pdf) (Generative Pre-Training, OpenAI)：单向 Transformer（masked decoder），通过预测语料中的后续词汇进行自监督
> - [BERT](https://arxiv.org/abs/1810.04805) (Bidirectional Encoder Representation from Transformers, Google)：双向 Tranformer（encoder），随机 mask 语料中的词汇，使模型逐步掌握上下文语境，从而把被遮住的片段以尽可能合乎逻辑的方式填补回去

![|600](Academic/assets/16485664017622.jpg)

BEiT 将 BERT 的思想引入了 CV 领域：

> 让模型看很多的图片，随机盖住一些 image patches，让模型预测盖住的 patches 是什么，不断计算预测的 patches 与真实的 patches 之间的差异，利用它作为 loss 进行反向传播更新参数，来达到 Self-Supervised Learning 的效果。

使用 Tokenizer（这里使用了离散变分自编码器 dVAE，即 VQ-VAE 的随机采样改进版本）将所有输入图像编码为一系列的 visual token（token 的数量与 patch 的数量相对应）。一张 224×224 的输入图片通过 Tokenizer 会变成 14×14 个 visual token，每个 token 是一个位于 $[1, 8192]$ 之间的数。这类似于有一张图像的词汇表，里面有 8192 个词，每个 patch 经过 Tokenizer 都能映射成其中的一个词。然后通过 Decoder 将 visual token 还原为图像，训练目标是重建的图像与输入尽可能一致。

与此同时，采用类似 ViT 的方法对图像划分 patch，并随机 mask 大约 40% 的 patch（被 mask 的 patch 替换为可学习的编码），然后使用 BEIT Encoder 学习这些 patch 的编码。对于那些被 mask 的 patch，使用一个分类器（Masked Image Modeling Head）预测其对应的 visual token，训练目标是最小化预测 token 与该 patch 真实 token 之间的差异。

整个过程称之为 MIM（Masked Image Modeling）：模型通过学习尽可能地恢复 mask 部分的 visual token（而非 masked patch 的原始像素），其中所使用的真实 visual tokens 是通过一个额外训练的 dVAE 得到的。网络训练时的优化步骤是：首先使用重构损失优化 Tokenizer 和 Decoder，使 dVAE 能够学习到更好的 token；然后再优化 Encoder 和 Masked Image Modeling Head，使预测出的 visual token 与真实 patches 所对应的 visual token 接近。

预训练完成后，将 BEIT Encoder 的参数冻结即可用于下游任务的特征提取。

### MAE: Masked Autoencoders Are Scalable Vision Learners

> [!info]+
> **Masked Autoencoders Are Scalable Vision Learners**
> CVPR 2021 Oral; Facebook, KaimingHe
>
> - [Zotero](zotero://select/items/@he2021masked)
> - [URL](https://openaccess.thecvf.com/content/CVPR2022/html/He_Masked_Autoencoders_Are_Scalable_Vision_Learners_CVPR_2022_paper.html)
> - [arXiv](https://arxiv.org/abs/2111.06377)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CMAE_CVPR_2021.pdf)

![|500](Academic/assets/16485664296544.jpg)

同样采用了 BERT 的思想。在预训练期间，大比例的随机的图像块子集（如 75%）被屏蔽掉，在可见的小子集上训练 encoder，然后用一个 decoder 以像素为单位重建原始图像。
预训练完成后，decoder 被丢弃，encoder 直接应用于未损坏的图像以生成特征表征。

与 BEiT 的区别：没有对图像编码，直接像素级重建；更大比例的 mask（图像的冗余信息高于文字）；encoder 只处理可见的 patch；速度更快。

### 🔥 CLIP: Contrastive Language-Image Pre-training

> [!info]+
> **Learning Transferable Visual Models from Natural Language Supervision**
> ICML 2021; OpenAI
>
> - [Zotero](zotero://select/items/@radford2021learning)
> - [URL](http://proceedings.mlr.press/v139/radford21a.html)
> - [arXiv](https://arxiv.org/abs/2103.00020)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CCLIP_ICML_2021.pdf)
> - [【多模态】CLIP 模型 - 知乎](https://zhuanlan.zhihu.com/p/432590298)
> - [15. CLIP - by Tom Yeh - AI by Hand ✍️](https://aibyhand.substack.com/p/clip)

![|600](Academic/assets/16485665597558.jpg)

> 如果图片有文本描述，通过文本监督来学习更多的图片信息不失为一个好方法

在大规模数据集上使用 NLP 监督预训练图像分类器；用 4 亿对来自网络的图文数据对，将文本作为图像标签，进行训练。进行下游任务时，只需要提供和图上的 concepts 对应的文本描述，就可以进行 zero-shot transfer（对未知任务进行推理）。

pretext task：匹配文本（描述性短句）和图像（而非直接预测图像所对应的文本）。采用对比学习方法进行自监督。

使用 2 个 encoder 分别处理文本和图片数据，其中 text encoder 使用 Transformer（固定 77 个 token，更短则 padding），image encoder 则分别尝试了 ResNet（特征图作为 tokens） 和 ViT（切 patch 划分 tokens）。将 encoder representation 直接线性投影到 multi-modal embedding space。然后计算两种模态表征之间的余弦相似度，让互相匹配的图文对相似度最大，不匹配的图文对相似度最小，损失函数为对称的 cross-entropy loss（InfoNCE）。

InfoNCE 损失：在一批相似度数据中找到唯一的正例样本，这相当于一个互斥的多分类任务：找到正例应该属于哪一个样本/哪一类，即先取正例的相似度在所有样本中的 softmax，使其变成一个和为 1 的概率分布，再取负对数，也就是标准的多分类交叉熵损失。

> 后续也有工作 [[#SigLIP]] 改进此损失：不再把正负样本放在一起使用 softmax（耦合互斥），而是视作不互斥的二分类任务，对每个样本独立使用 sigmoid。

预训练完成后，在进行下游的 zero-shot transfer 任务（图像分类）时，考虑到大部分的数据集的标签都是以单词的形式存在的，然而在预训练阶段的文本描述大多都是某个短句，为了填补这种数据分布上的差别，作者考虑用 prompt template 对标签进行扩展，例如可以用 `a photo of a "object".` 作为文本端的输入，其中的 `object` 就是需要预测的 zero-shot 类别标签（将数据集的类别标签转换为文字描述）。将数据集所有类别标签对应的 prompt template 通过 text encoder 获得其特征，输入的图片经过 image encoder 输出特征，计算其与各类别文本特征的余弦相似度以预测类别。

CLIP 可以作为一种关注图像语义信息、跨模态对齐的 text/image embedding/encoding 工具。在 LLM 等文本生成任务中，一般会采用更好的预训练 text embedder。
在图像生成模型中，文本 condition 部分一般会使用 CLIP text encoder，图像 condition 部分若用于语义和风格迁移等任务也会采用 CLIP image encoder，而生成的图像本身则会使用 VQ-VAE 这样 low-level 信息量保留更多的模型来完成 latent 空间编解码。
在仅关注高维语义的多模态理解任务中，图像输入一般也会使用 CLIP encoder 作为编码器。

关于 CLIP 的 embedding 维度（[参考链接](https://stackoverflow.com/questions/75693493/why-the-text-embedding-or-image-embedding-generated-by-clip-model-is-768-%c3%97-n/79243065#79243065)）：

Each CLIP has a `text-model` and `image-model`. The embedding shape for each model varies. Also, embeddings for image and text are different.

---

There are 4 models of CLIP by OpenAI in Huggingface: (`image_size`, `patch_size`, `image_hidden_size`, `text_hidden_size`, `proj_dim`)

- `openai/clip-vit-base-patch32` (600M): 224, 32, 768, 512, 512
- `openai/clip-vit-base-patch16` (600M): 224, 16, 768, 512, 512
- `openai/clip-vit-large-patch14` (1.7G): 224, 14, 1024, 768, 768
- `openai/clip-vit-large-patch14-336` (1.7G): 336, 14, 1024, 768, 768

---

**Text Model**: The number `77` is related to the maximum number of text tokens (unsure whether it includes `EOT` token). Only one `EOT` token of shape `[768]` is selected for cos similarity calculation. The number of tokens for `text-model` is the same `77` across different models.

**Image Model**: Each CLIP model has a different `image-model` architecture and thus, a different embedding size for image. The number of tokens for `image-model` is `(image_size / patch_size)**2 + 1` where `+1` represent `CLS` token.

**CLIP Training**: Notice a mismatch between `image_hidden_size`, `text_hidden_size` for every model. Here is how it work in `clip-vit-large-patch14`: (1) After transformers, images has shape `[B, 197, 1024]` and texts has shape `[B, 77, 768]`. (2) Then we select `CLS` for images and `EOT` for texts. The shapes become `[B, 1024]` and `[B, 768]`. (3) They are each projected by a linear layer. The shapes become `[B, 768]` and `[B, 768]`. (4) Then we dot-product to get shape `[B, B]` similarities.

**Text Conditioning**: Stable Diffusion 1.x/2.x models uses all 77 text tokens [before](https://github.com/KokeCacao/KatUI/blob/0d9a3e5448a595a2ae22759fc6fccdd736f61bd0/katzuki/src/nodes/KatUIDiffusionBasics/basic.py#L268) [pooling](https://github.com/openai/CLIP/blob/dcba3cb2e2827b402d2701e7e1c7d9fed8a20ef1/clip/model.py#L349). For non-critical tasks, one could in theory use only the [pooled `EOT` token](https://github.com/huggingface/transformers/blob/19dabe96362803fb0a9ae7073d03533966598b17/src/transformers/models/clip/modeling_clip.py#L993).

**Image Conditioning**: [IPAdapter](https://github.com/tencent-ailab/IP-Adapter/blob/main/ip_adapter/ip_adapter.py) uses pooled image embedding (in the last transformer layer) while IPAdapterPlus uses full image embeddings in the second to the last transformer layers (possibly because the last transformer isn't meant to produce meaningful output for last layer embeddings other than the `CLS` token).

### ViLT: Vision-and-Language Transformer

> [!info]+
> **ViLT: Vision-and-Language Transformer Without Convolution or Region Supervision**
> ICML 2021
>
> - [Zotero](zotero://select/items/@kim2021vilt)
> - [URL](https://proceedings.mlr.press/v139/kim21k.html)
> - [arXiv](https://arxiv.org/abs/2102.03334)
> - [PDF](file:///D:\Media\Documents\我的文档\Academic\Zotero\storage\DL\ViLT_ICML_2021.pdf)
> - [Code](https://github.com/dandelin/vilt)
> - [ViLT：最简单的多模态 Transformer - 知乎](https://zhuanlan.zhihu.com/p/369733979)

![|600](Academic/assets/16488022224019.jpg)

CLIP 在两个模态都使用开销巨大的 transformer encoder，然后使用简单的点积计算特征相似性。而 ViLT 是首个将 visual embedding 设计的如 text embedding 一样轻量的方法，其主要计算量都集中在模态的交互上。在模态交互部分，ViLT 延用了 BERT 和 UNITER 的 single-stream 交互方式，即对图像和文本 concate 后进行交互操作（类似 ViT 的结构，multiheaded self-attention + MLP）。

文本特征输入部分，将文本看成一个词序列，通过 word embedding matrix 转化成 word embedding，然后与 position embedding 相加；图像特征输入部分，将图像看成一个图像 patch 序列，通过 linear projection 转化成 visual embedding，然后与 postion embedding 相加。其中 word 和 visual embedding 都拼接了一个可学习的 modal-type embedding 标志位来区分。

ViLT 的预训练任务有 2 个：Image Text Matching (ITM)，以 0.5 的概率随机将文本-图片对中的图片替换为其他图片，然后使用一个线性的 ITM head 将输出特征映射成一个二值 logits，用来判断图像和文本是否匹配；Masked Language Modeling (MLM)，通过上下文信息预测文本的 masked tokens。

### BLIP: Bootstrapping Language-Image Pre-training

> [!info]+
> **BLIP: Bootstrapping Language-Image Pre-Training for Unified Vision-Language Understanding and Generation**
> ICML 2022
>
> - [Zotero](zotero://select/items/@li2022blip)
> - [URL](https://proceedings.mlr.press/v162/li22n.html)
> - [arXiv](https://arxiv.org/abs/2201.12086)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CBLIP_ICML_2022.pdf)
> - [Code](https://github.com/salesforce/BLIP)
> - [多模态超详细解读 (六)：BLIP：统一理解和生成的自举多模态模型 - 知乎](https://zhuanlan.zhihu.com/p/627481137)

提出了一种编码器-解码器混合架构（Multimodal mixture of Encoder-Decoder, MED），MED 的特点是很灵活，既可以作为单模态的 encoder，又可以作为基于图像的文本 encoder，或者基于图像的文本 decoder，即兼顾了图文理解任务（如检索）和文本生成任务。

根据图片生成 prompt 的工具 [**Clip Interrogator**](https://github.com/pharmapsychotic/clip-interrogator) 就是通过 CLIP 和 BLIP 结合实现的：由 BLIP 通过图片生成一句话描述，同时用 CLIP 从数据集中检索出与图片特征相似度高的多个短语 prompt。

### SigLIP

> [!info]+
> **Sigmoid Loss for Language Image Pre-Training**
> ICCV 2023 Oral; Google
>
> - [Zotero](zotero://select/items/@zhai2023sigmoid)
> - [URL](https://openaccess.thecvf.com/content/ICCV2023/html/Zhai_Sigmoid_Loss_for_Language_Image_Pre-Training_ICCV_2023_paper.html)
> - [arXiv](https://arxiv.org/abs/2303.15343)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CSigLIP_ICCV_2023.pdf)
> - [Code](https://github.com/google-research/big_vision)
> - [SigLIP——采用 sigmoid 损失的图文预训练方式 - 知乎](https://zhuanlan.zhihu.com/p/718982190)

与 CLIP 不同的是，SigLIP 在训练过程中对图像-文本对采用成对 sigmoid 损失（相当于视作二分类任务）。这种训练损失无需全局查看 batch 内所有图像与文本之间的成对相似度，因此不仅能更高效地扩展到更大的 batch，也能在较小的 batch 下实现更优的性能。

## ✨ VAE

- [图像生成发展起源：从 VAE、VQ-VAE、扩散模型 DDPM、DETR 到 ViT、Swin transformer-CSDN 博客](https://blog.csdn.net/v_JULY_v/article/details/130361959)
- [文生图模型演进：AE、VAE、VQ-VAE、VQ-GAN、DALL-E 等 8 模型](https://mp.weixin.qq.com/s/iFrCEpAJ3WMhB-01lZ_qIA)

### 🔥 VAE: Variational Autoencoder

> [!info]+
> **Auto Encoding Variational Bayes**
> 2022
>
> - [Zotero](zotero://select/items/@kingma2022autoencoding)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CVAE_arXiv_2022.pdf)
> - [变分自编码器（一）：原来是这么一回事 - 科学空间|Scientific Spaces](https://kexue.fm/archives/5253)
> - [变分自编码器（六）：从几何视角来理解 VAE 的尝试 - 科学空间|Scientific Spaces](https://kexue.fm/archives/7725)
> - [24. Variational Auto Encoder (VAE) - by Tom Yeh (substack.com)](https://aibyhand.substack.com/p/24-variational-auto-encoder-vae)

为了解决 Autoencoder (AE) 无法泛化到训练分布外的数据（只能做特征提取、压缩等任务，无法把 decoder 直接当做生成模型）的问题

输入样本，用 encoder 网络预测两个参数均值和方差，然后依此从正态分布采样出 latent code，然后将 latent code 输入 decoder 重建原样本

训练 loss 包括重建 MSE 损失和标准正态约束（尽可能使 encoder 预测的均值方差接近标准正态分布，防止退化为简单的 AE）。前者保证了网络的表达能力，后者约束了隐空间的良好性质，二者是矛盾的，最终学习到的是二者之间的一个平衡态，即网络能够准确区分不同的样本，且不同样本各自 latents 的分布也都接近（但绝不完全等于）标准正态分布。各个样本的信息就隐藏在各自略微偏离标准正态分布的均值和方差中。

训练完成后就能从标准正态分布中采样任意 latent，直接输入进 decoder 进行无条件生成。由于所有样本的 latent 分布都接近标准正态，因此标准正态中的直接采样结果也能覆盖绝大部分训练样本，且拥有泛化到相似内容的能力。

### 🔥 VQ-VAE

> [!info]+
> **Neural Discrete Representation Learning**
> NIPS 2017; Google
>
> - [Zotero](zotero://select/items/@vandenoord2017neural)
> - [URL](https://proceedings.neurips.cc/paper/2017/hash/7a98af17e63a0ac09ce2e96d03992fbc-Abstract.html)
> - [arXiv](https://arxiv.org/abs/1711.00937)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CVQ-VAE_NIPS_2017.pdf)
> - [VQ-VAE 的简明介绍：量子化自编码器 - 科学空间|Scientific Spaces](https://kexue.fm/archives/6760/comment-page-3)

VAE 直接预测分布更难在大规模数据上学习，因此引入了离散的、量化的 codebook（码本）代替，结果类似 K-Means 特征聚类，有助于模型更好地理解数据中的离散结构和语义信息，同时可以避免过拟合。

VQ 即 Vector Quantization，它量化编码出的向量是离散的，每个元素都是一个整数（码本的 index）。

训练时码本 embedding 随机生成，encoder 输出连续向量通最近邻搜索对应到码本中的某个 embedding，取其 index 作为离散编码。

量化的 argmin 过程并不可微，因此训练时采用了 Straight-Through 策略，用 stop gradient 操作来为离散编码手动赋予 encoder 输出的连续向量的梯度（把量化向量的梯度复制给量化前的连续向量），实现反向传播。

对于图像，如果只编码为一个向量，重构时难免失真，而且泛化性难以得到保证。所以实际编码时直接用多层卷积将图像编码为保留了空间位置结构信息的 $𝑚 \times 𝑚$ 个 latent code，即 $𝑚 \times 𝑚$ 个整数（码本 index）。

注意：VQ-VAE 并没有显式约束隐空间的分布，因此是一种离散化的 AE（而非 VAE）。

VQ-VAE 的 loss 分为三部分：

- Reconstruction loss：重构损失，用于更新 encoder/decoder 的参数（不会更新码本）；
- Codebook loss：推动码本中被选择的量化向量靠近 encoder 的输出连续向量，用于更新码本，相当于在线聚类；对 encoder 的输出向量 `stopgrad`；
- Commitment loss：反过来推动 encoder 的输出连续向量靠近码本中被选择的量化向量，用于更新 encoder，防止 encoder 的输出在码本的不同向量之间跳变，以稳定训练；对码本向量 `stopgrad`。

其中 commitment loss 通常有一个 0.25 的权重，从而更强调对码本的更新而非对 encoder 输出的更新。

VQ-VAE 训练完成后可以很容易实现图像压缩、重建的目的，但是无法生成新的图像数据（不像 VAE 有一个平滑连续的先验分布可以从中直接采样）。当然可以随机生成 index，然后对应生成量化后的 latent code，进而使用 decoder 来生成输出图像。但是这样的 latent code 每个位置都是随机生成的，完全没有全局信息甚至局部信息，因此图像也几乎不可能具有实际语义信息。
因此，作者引入了 PixelCNN 来自回归地生成考虑了全局信息的 latent code，进而可以生成更真实的图像。也可以在这一步引入条件控制，来实现有条件生成。

### dVAE

[Variational Autoencoders: VAE to VQ-VAE / dVAE | Rohit Bandaru](https://rohitbandaru.github.io/blog/VAEs/#discrete-vae)

[离散变分自编码器(dVAE)详解(采用 Gumbel Softmax)-CSDN 博客](https://blog.csdn.net/sjtu_wyy/article/details/148900933)

被用于 OpenAI 的 DALL-E 模型

[[#🔥 VQ-VAE|VQ-VAE]] 的 encoder 输出一个连续 latent，然后通过最近邻搜索的方式对应到离散的码本上，并通过梯度直通（Straight-Through）的方式进行梯度回传。

dVAE 与之不同，encoder 的输出被视为一个离散概率分布的 logits（而非 latent 本身），然后通过 [Gumbel-Softmax 技巧](https://zhuanlan.zhihu.com/p/633431594)（为分类分布的 logits 添加 gumbel noise，然后选择加完噪声后值最大的那个类别，这一采样过程与从原始分类分布中采样是等价的；把其中的 argmax 操作软化为 softmax 即可微）获得分类分布的采样结果 index。

然后，我们取 Gumbel-Softmax 之后的结果对码本向量进行加权，获得量化后的向量（这一过程完全可微）。训练过程中，不断减小 softmax 的温度参数，从而越来越接近 argmax，最终获得一个接近 one-hot 的加权结果。
也有另外一种更类似 VQ-VAE 方式（hard 模式）：Gumbel-Softmax 之后直接取 argmax，然后利用梯度直通使得重构损失回传 encoder。

dVAE 还具有 VAE 类似的显式 KL loss 来约束 encoder 输出的离散概率分布，通常是与一个先验的均匀分布进行对齐。

## ✨ GAN

### GAN: Generative Adversarial Network

> [!info]+
> **Generative Adversarial Nets**
> NIPS 2014; Goodfellow
>
> - [Zotero](zotero://select/items/@goodfellow2014generative)
> - [URL](https://proceedings.neurips.cc/paper/2014/hash/5ca3e9b122f61f8f06494c97b1afccf3-Abstract.html)
> - [arXiv](https://arxiv.org/abs/1406.2661)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CGAN_NIPS_2014.pdf)
> - [Code](https://github.com/hindupuravinash/the-gan-zoo)
> - [hindupuravinash/the-gan-zoo: A list of all named GANs!](https://github.com/hindupuravinash/the-gan-zoo)

![|500](Academic/assets/16491328814558.png)

GAN (unconditional): 输入一个从已知的简单概率分布（如高斯分布或均匀分布）中采样得到的随机噪声，Generator 网络以此生成图片，然后将真实的图片与生成的图片作为正负例一起输入 Discriminator 网络，Discriminator 学习判断一张图片是否是“真实的”（二分类）。

训练过程中，生成网络 G 的目标就是尽量生成真实的图片去欺骗判别网络 D；而网络 D 的目标就是尽量把网络 G 生成的图片和真实的图片分别开来。两个网络迭代更新参数，即固定 G 训练 D，再固定 D 训练 G，反复迭代。这样，G 和 D 构成了一个动态的“博弈过程”。当生成模型 G 恢复了训练数据的分布（生成了以假乱真的样本），判别模型 D 无法判别（输出概率为 0.5）时，双方网络就都达到了利益最大化，不再改变自己的策略，也即不再更新自己的参数权重。

G 是一种以半监督方式训练分类器 D 的方法。G 的参数更新不是直接来自数据样本，而是来自 D 输出值的反向传播（固定 D 的参数，训练 G 使得 D 的输出标签尽可能为“真”）。原始的 GAN 损失函数采用类似交叉熵的形式，使得 G 的输出趋近于真实样本分布，同时 D 最大概率地分对训练样本的标签。

$$
\min _{G} \max _{D} \mathcal{V}(D, G)=\mathbb{E}_{x \sim p_{\text {data }}(x)}[\log D(x)]+\mathbb{E}_{z \sim p(z)}[\log (1-D(G(z)))]
$$

GAN Loss 的意义在于，在无法得到生成图片和真实图片的实际概率分布的情况下，仅仅通过对二者的采样，就可以得到两个概率分布之间的 divergency（如 KL/JS 散度），然后以此作为需要最小化的损失函数。

原始 GAN Loss 在实际优化过程中存在梯度弥散问题，因此作者又提出了 non-saturating (NS) GAN Loss（非饱和：若输入无穷大则输出也无穷大），将 $\min\log(1−D(G(z)))$ 改为 $\max\log ⁡(D(G(z)))$：

$$
\log(1−D(G(z))) \rightarrow -\log (D(G(z)))
$$

GAN 与全监督式的生成方法不同，因为生成任务的期望输出往往是不确定的（具有一定概率分布），直接生成很可能导致多种可能结果的混合和平均（图片的模糊），而 GAN 具有一定的“创造力”，能够从概率分布中采样出合理的结果。当然，可以将这二者结合以产生更好的生成结果。

> 上述 GAN loss 公式要求 D 网络的输出是 0~1 的 prob（过 sigmoid 后的 logits），即：$\log(sigmoid(D(x)))+\log(sigmoid((1-D(G(z)))))$。
> 在 PyTorch 中， 此 loss 可以通过 `nn.Sigmoid` + `nn.BCELoss`，或 `nn.BCEWithLogitsLoss`（后者更稳定）来实现，[参考实现](https://github.com/eriklindernoren/PyTorch-GAN/blob/master/implementations/gan/gan.py)。
> 此外，梯度弥散改进后 loss 版本也可以利用 softplus 更简洁地实现：
> $-\log(sigmoid(D(x)) = \log(1 + exp(-D(x))) = softplus(-D(x))$; $-\log(1-sigmoid(D(G(z)))) = \log (1 + exp(D(G(z))) = softplus(D(G(z))$，[参考](https://github.com/pfnet-research/sngan_projection/issues/18)。

### WGAN: Wasserstein GAN

- [Wasserstein GAN](https://arxiv.org/abs/1701.07875) (2017)
- [Improved Training of Wasserstein GANs](https://proceedings.neurips.cc/paper/2017/hash/892c3b1c6dccd52936e27cbd0ff683d6-Abstract.html) (NIPS 2017)
- [互怼的艺术：从零直达 WGAN-GP - 科学空间|Scientific Spaces](https://kexue.fm/archives/4439)

从损失函数的角度对 GAN 做了改进。从理论上给出了 GAN 训练不稳定的原因，即交叉熵（JS 散度）不适合衡量具有不相交部分的分布之间的距离，因此转而使用 Wassertein 距离（最优传输）去衡量生成数据分布和真实数据分布之间的距离，理论上解决了训练不稳定的问题。

### Conditional GAN

- [Conditional Generative Adversarial Nets](https://arxiv.org/abs/1411.1784)

原始的 GAN 过于自由，训练会很容易失去方向，从而导致不稳定又效果差。而 Conditional GAN 就是在原来的 GAN 模型中加入一些先验条件（有监督），使得 GAN 变得更加的可控制。具体的来说，我们可以在生成模型 G 和判别模型 D 中同时加入条件约束来引导数据的生成过程。条件可以是任何补充的信息，如类标签、其他参考图片（image translation / pix2pix）、其它模态的数据（text to image）等。

### CycleGAN

> [!info]+
> **Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks**
> ICCV 2017
>
> - [Zotero](zotero://select/items/@zhu2017unpaired)
> - [URL](https://openaccess.thecvf.com/content_iccv_2017/html/Zhu_Unpaired_Image-To-Image_Translation_ICCV_2017_paper.html)
> - [arXiv](https://arxiv.org/abs/1703.10593v6)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CCycleGAN_ICCV_2017.pdf)
> - [Code](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix)
> - [CycleGAN 论文的阅读与翻译，无监督风格迁移 - 知乎](https://zhuanlan.zhihu.com/p/45394148)

将 GAN 用于无监督学习（如风格迁移）

引入了“循环一致性”（CycleConsistent）的概念。这种思想来自翻译领域的一种简单的监督手段，即中译英之后再英译中，两组中文语句应当尽可能保持一致。

### StyleGAN

> [!info]+
> **A Style-Based Generator Architecture for Generative Adversarial Networks**
> CVPR 2019; NVIDIA
>
> - [Zotero](zotero://select/items/@karras2019stylebased)
> - [URL](https://openaccess.thecvf.com/content_CVPR_2019/html/Karras_A_Style-Based_Generator_Architecture_for_Generative_Adversarial_Networks_CVPR_2019_paper.html)
> - [arXiv](https://arxiv.org/abs/1812.04948)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CStyleGAN_CVPR_2019.pdf)
> - [Code](https://github.com/NVlabs/stylegan)

受风格迁移启发而设计了一种新的网络结构，可以通过无监督式的自动学习，对图像的高层语义属性（Style）做一定解耦分离，这里的 Style 是指人脸图像的姿势和身份、所生成图像的随机变化（如雀斑和头发）等，也可以做到一定程度上的控制合成。

StyleGAN 的网络结构包含两个部分，第一个是 Mapping network，由隐变量 $z$ 生成中间隐变量 $w$ 的过程，这个 $w$ 就是用来控制生成图像的 style。 第二个是 Synthesis network，它的作用是生成图像，创新之处在于给每一层子网络都喂了 $A$ 和 $B$，$A$ 是由 $w$ 转换得到的仿射变换，用于控制生成图像的风格，$B$ 是转换后的随机噪声，用于丰富生成图像的细节，即每个卷积层都能根据输入的 $A$ 来调整 style。整个网络结构还是保持了 PG-GAN（progressive growing GAN） 的结构。

### 🔥 VQGAN

> [!info]+
> **Taming Transformers for High Resolution Image Synthesis**
> CVPR 2021 Oral
>
> - [Zotero](zotero://select/items/@esser2021taming)
> - [URL](https://openaccess.thecvf.com/content/CVPR2021/html/Esser_Taming_Transformers_for_High-Resolution_Image_Synthesis_CVPR_2021_paper.html?ref=)
> - [arXiv](https://arxiv.org/abs/2012.09841)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CVQGAN_CVPR_2021.pdf)
> - [Code](https://github.com/CompVis/taming-transformers)
> - [VQGAN 论文与源码解读：前 Diffusion 时代的高清图像生成模型 - 知乎](https://zhuanlan.zhihu.com/p/637705399)

Motivation：相比擅长捕捉局部特征的 CNN，Transformer 的优势在于它能更好地融合图像的全局信息。可是，Transformer 的自注意力操作开销太大，只能生成一些分辨率较低的图像。因此，作者认为，可以综合 CNN 和 Transformer 的优势，先用基于 CNN 的 VQGAN 把图像压缩成一个尺寸更小、信息更丰富的小的 latent map，再用 Transformer 来生成这个小 map。

VQGAN 在 VQ-VAE 基础上做出的主要改变：

1. 引入 GAN 的思想，将 VQ-VAE 当做生成器（Generator），加入判别器（Discriminator），以对生成图像的质量进行判断、监督，并加入感知重建损失来重建更具有保真度的图片，也就学习了更丰富的 codebook
2. 将训练完成后用于自回归生成 latent code 的 PixelCNN 替换为性能更强大的 Transformer（GPT2），并引入滑动窗口自注意力机制，以降低计算负载，生成更大分辨率的图像

与 [[#VQ-VAE]] 类似，训练 VQGAN 时，需要先训练一对 encoder & decoder 完成图像到 latent space 的压缩转换，再训练一个 Transformer-based 模型用于生成压缩后的 latent map。推理生成时，先用 Transformer 生成出一个 latent map（next-token prediction 按空间顺序预测一张图像的所有 tokens），再用 decoder 复原为真实图像。

**思考**

基于自回归 transformer 来做视觉生成需要 VQ-VAE（作为图像的“Tokenizer”），因为需要有：1. 互逆的 encoder-decoder；2. 离散的码本用于 next-token prediction 的分类预测任务。

而多模态大模型中的视觉感知任务输出的模态只有 text，因此输入的视觉 token 可以是 CLIP（ViT）之类的跨模态对齐后的连续特征（此时这些 visual tokens 也相当于 text embedding）。
不同于 VQ-VAE 的多层次细粒度特征，CLIP 的特征仅关注语义，当然是不能被还原为原始图像的，而视觉感知任务也并不要求输出图像。

来自 AI：

**为什么一定要离散的 VQ-VAE 而不是连续的 VAE？**

1. **离散化简化建模复杂度**
   **分类 vs 回归**：离散化后的特征将每个潜在编码映射到有限的码本词汇表，然后自回归模型只需预测有限数量的类别（码本索引）。而连续特征的回归需要精确预测高维向量的每个分量，这在优化上更困难（需处理连续空间的概率密度估计）。
   **信息压缩**：离散化通过码本对特征进行量化，强制模型仅保留数据中最关键的信息，避免冗余细节，类似于自然语言中的词汇表。这种压缩使得自回归模型更容易捕捉数据分布的全局结构。
2. ​**缓解误差累积问题**
   **​ 自回归生成中的误差传播**：在生成过程中，每一步的预测误差会累积到后续步骤。离散化将误差限制在有限的码本选择范围内（例如错误预测的索引可能对应语义相似的码本向量），而连续特征的回归误差（如向量分量的微小偏移）可能逐级放大，导致生成结果严重失真。
   **​ 鲁棒性**：离散码本为潜在空间引入了归纳偏置，使得生成过程更稳定。例如，即使模型预测的索引不完全准确，对应的码本向量仍可能保持合理的局部结构。
3. ​**离散潜在空间与自回归模型的适配性**
   **​ 离散序列的天然匹配**：Transformer 自回归模型最初在文本生成（离散 token 序列）中表现出色。离散潜在编码使图像生成可类比为“文本生成”，每个位置的预测目标是一个离散的索引，而非连续向量。
   **​ 长程依赖建模**：Transformer 通过注意力机制捕捉长程依赖，而离散序列中的模式（如重复出现的码本索引组合）更易于建模，连续特征则需要处理复杂的协方差结构。
4. **与 GAN 的结合提升生成质量**
   **​VQ-GAN 的双重优势**：VQ-GAN 在 VQ-VAE 基础上引入 GAN 的判别器，通过对抗训练提升重建质量。离散潜在空间保留了结构信息，而 GAN 弥补了量化可能导致的细节损失。
   **​ 连续特征的局限性**：若直接用 VAE 的连续特征，生成器可能难以通过回归任务恢复高频细节（如纹理），而离散码本可通过对抗训练学习更丰富的局部模式。

关于上述第 3 点，**为什么 next-token prediction 采用分类的训练目标（因而要求离散编码），而非直接回归？**

理论上可以尝试直接回归连续特征，但存在以下挑战：

- ​**优化难度**：连续回归需 MSE 或类似损失，但高维连续空间的概率密度估计困难，易导致模糊或坍缩的结果。
- ​**模式坍缩风险**：连续自回归模型可能倾向于预测均值，导致生成结果缺乏多样性（如 VAE 生成的模糊图像）。

当然，VQ-VAE/GAN 式的离散化也必然存在信息损失，这对于 OCR 等偏向 low-level 的多模态感知任务来说是相当致命的问题。

后续 Kaiming 团队提出的 MAR（[Autoregressive Image Generation without Vector Quantization](https://proceedings.neurips.cc/paper_files/paper/2024/hash/66e226469f20625aaebddbe47f0ca997-Abstract-Conference.html)；[解读](https://zhuanlan.zhihu.com/p/711930343)）就采用了非量化的 VAE 作为连续特征编码器，其自回归 Transformer 输出的不是类别分布，而是回归出连续特征（之前所有 patch 导出的上下文信息），作为 condition 输入到一个 diffusion model 中预测 next token，这样就避免了模式坍缩（预测均值）的问题。（参考：[解读何恺明团队新作：不用向量离散化的自回归图像生成](https://zhouyifan.net/2024/07/27/20240717-ar-wo-vq/)）

其他参考：

- [“闭门造车”之多模态思路浅谈（一）：无损输入 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9984)
- [“闭门造车”之多模态思路浅谈（二）：自回归 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/10197)（分析了**自回归预测连续特征**的可行性）

## ✨ Diffusion Models

扩散模型最早是在 2015 年的 [Deep Unsupervised Learning using Nonequilibrium Thermodynamics](https://arxiv.org/abs/1503.03585) 文章中提出的，但当时扩散模型并不 work，所以并没有被广泛应用。

2020 年，Denoising Diffusion Probabilistic Models（ DDPM）的出现，将扩散模型带到了一个新高度。

在其之前主流的生成网络 GAN，还存在一些缺点，因为其要训练两个网络，难度较大，容易不收敛，多样性较差，并且模型在训练过程中不稳定，只要骗过判别器即可。

而扩散模型用一种更简单的方法诠释了生成模型应该如何学习和生成。扩散模型很可能会替代 GAN 成为主流的生成模型。

同为生成模型，Diffusion model 和 GAN 很像，都是给定噪声，生成图片 ，但是 DM 中的噪声（相当于 GAN 中的 latent code）是与图片同尺寸的。

参考：[What are Diffusion Models? | Lil'Log](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/)

### 🔥 DDPM

> [!info]+
> **Denoising Diffusion Probabilistic Models**
> NIPS 2020
>
> - [Zotero](zotero://select/items/@ho2020denoising)
> - [URL](https://proceedings.neurips.cc/paper/2020/hash/4c5bcfec8584af0d967f1ab10179ca4b-Abstract.html)
> - [arXiv](https://arxiv.org/abs/2006.11239)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CDDPM_NIPS_2020.pdf)
> - [Project](https://hojonathanho.github.io/diffusion/)
> - [Code](https://github.com/hojonathanho/diffusion)
> - [DDPM（Denoising Diffusion Probabilistic Models）扩散模型简述\_champion\_H 的博客-CSDN 博客](https://blog.csdn.net/zhL816/article/details/127990163)
> - [由浅入深了解 Diffusion Model - 知乎](https://zhuanlan.zhihu.com/p/525106459)
> - [一文弄懂 Diffusion Model - 知乎](https://zhuanlan.zhihu.com/p/586936791)
> - [生成扩散模型漫谈（一）：DDPM = 拆楼 + 建楼 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9119)

![|800](Academic/assets/DDPM.png)

Diffusion Model 包含一个前向过程（Forward process）和一个逆向过程（Reverse process）。前向过程向图片上逐步添加噪声；逆向过程可以理解为一个去噪推断过程。

**前向过程**

给定从真实数据分布中采样的数据点 $x_0 \sim q(x)$，我们逐步向样本中添加少量高斯噪声，从而产生一系列含噪样本 $x_t \in \{x_1,\dots,x_T\}$。其中，噪声基底是标准高斯分布的一个采样值 $\epsilon_t \sim \mathcal{N}(0, \mathbf{I})$，方差由**扩散率**（或 variance schedule） $\{\beta_t\in(0,1)\}_{t=1}^T$ 控制（逐步增大）。扩散率 $\beta_t$、总时间步数 $T$（DDPM 中取 $1000$）均为超参数。

```python
betas = torch.linspace(start=0.0001, end=0.02, steps=1000)
```

上述过程是一个马尔科夫过程（当前状态只和前一个状态有关），转移概率为：

$$
q(x_t|x_{t-1}) := \mathcal{N}(x_t; \sqrt{1-\beta_t} x_{t-1}, \beta_t \mathbf{I})
$$

即每一步中，所加噪声的方差是已知值 $\beta_t$；均值则是由前一步数据 $x_{t-1}$ 和 $\beta_t$ 共同决定的，也即图像和噪音的权重是不断变化的（已知）。

> 为什么扩散率是逐渐增大的呢？也即为什么噪音所占的比例越来越大呢？可以反过来理解，在加噪声的过程中，扩散率逐渐增大，对应着在去噪声的过程中，扩散率逐渐减小。也就是说，去噪的过程是先把“明显”的噪声给去除，对应着较大的扩散率；当去到一定程度，逐渐逼近真实图像的时候，去噪速率逐渐减慢，开始微调，也就是对应着较小的扩散率。

在上述过程中，利用[[DL#重参数化技巧（Reparametrization tricks）|重参数化技巧]]，我们可以通过单次计算就一步到位地得到任意时间步 $t$ 后的加噪结果分布（闭式解）：令 $\alpha_t=1-\beta_t$，$\overline{\alpha_t}=\prod_{i=1}^t \alpha_i$，

$$
\begin{align}
x(t)&= \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t} \epsilon_{t-1}\\
&=\sqrt{\overline{\alpha_t}} x_0 + \sqrt{1-\overline{\alpha_t}} \epsilon_0
\end{align}
$$

$$
q(x_t|x_0) = \mathcal{N}(x_t; \sqrt{\overline{\alpha_t}} x_0, (1-\overline{\alpha_t}) \mathbf{I})
$$

于是，我们从前向过程得到的各步 $x_t$ 将会作为训练样本，帮助网络学习如何从 $x_T$ 中一步步去噪，最终得到 $x_0$。

**逆向过程**

逆向过程仍然是一个马尔科夫链过程，即求解 $q(x_{t-1}|x_t)$。应用贝叶斯公式，并引入一个先验概率 $q(x_0)$：

$$
q(x_{t-1}|x_t,x_0) = q(x_t|x_{t-1},x_0)\frac{q(x_{t-1}|x_0)}{q(x_t|x_0)}
$$

通过[数学推导](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/#reverse-diffusion-process)，可以将逆向过程中未知的 $x_0$ 消去，即：

$$
q(x_{t-1}|x_t)=\mathcal{N}(x_{t-1};\tilde{\mu}_t,\tilde{\beta}_t \mathbf{I})
$$

其中：

$$
\tilde{\mu}_t=\frac{1}{\sqrt{\alpha_t}}\left(\mathbf{x}_t-\frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}} \epsilon_t\right)
$$

^ddpm-mu

$$
\tilde{\beta}_t=\frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t} \cdot \beta_t
$$

从而在已知 $x_t$ 的情况下，只需知道该时间步所加的噪声 $\epsilon_t$（标准高斯分布的**某一特定的采样值**），就能求得分布 $q(x_{t-1}|x_t)$（此高斯分布的方差由扩散率确定，因此求得其均值即可），进而将 $x_{t-1}$ 采样出来，完成一步去噪过程。

> DDPM 使用的是 untrained 方差 $\Sigma_\theta(x_t,t)=\tilde{\beta}_t$ ，且认为 $\tilde{\beta}_t=\beta_t$ 和 $\tilde{\beta}_t=\frac{1-\overline{\alpha}_{t-1}}{1-\overline{\alpha}_t}\cdot\beta_t$ 结果近似（实验发现去掉系数后效果更好）。后续的 GLIDE 等工作则是根据网络预测 trainable 的方差 $\Sigma_\theta(x_t,t)$。

于是，我们使用一个神经网络（DDPM 采用了 UNet 结构），输入 $x_t$ 和时间步 $t$（类似位置编码，因为所有时间步的预测网络是参数共享的），预测 $\epsilon_t$（标准高斯分布的采样值）

> 实际上，在实际实现中，由于重参数化技巧的应用，这里的 $\epsilon_t$（$t=0,\dots,T-1$）并非真正意义上在 $t$ 步所加的标准高斯噪声基底的采样值，而是从 $x_0$ 到 $x_t$ 一步到位采样出的那个噪声。因此，公式中所有的 $\epsilon$ 在每一次的训练迭代中都可以等价为同一次采样的标准高斯噪声。

由于上文的重参数化技巧，训练过程无需按照时间步顺序进行，只需不断重复下述过程：给定扩散率 $\{\beta_t\}$，从训练集中随机采样当次迭代的训练图像 $x_0$，从标准高斯分布中随机采样一个噪声作为 $\epsilon_t$（也作为 $\epsilon_0$，参见上段对重采样的理解）的真值，然后从 $\{1,\dots,T\}$ 中随机选取一个 $t$，直接求得 $x_t$（前向过程），输入网络预测 $\epsilon_t$，求 MSE loss 进行训练。

我们认为，在前向过程中，原图扩散 $T$ 次后得到的 $x_T$ 就服从于一个各向同性的高斯分布。

网络训练完成后，对于任意一个高斯分布采样噪声，我们将其作为 $x_T$，用网络预测当前时间步的噪声，根据[公式](#^ddpm-mu)求出 $x_{T-1}$ 所服从的高斯分布的均值（方差固定已知），进而采样一个标准高斯分布并利用重参数化技巧将其变换为 $x_{T-1}$（相当于从 $x_{T-1}$ 所满足的高斯概率分布中进行一次采样）。迭代 $T$ 次后，所得的 $x_0$ 就是最终的满足训练集分布特点的生成图像。

> 既然我们每一步加的噪声都是参数已知的高斯分布，为什么不能直接在每一步都采样一个高斯分布当作噪声，令上一步的结果不断减去该噪声呢？因为要想使得这么做成立，必须要在每一步都恰好获得当初加噪时的那一步的高斯采样值（而不能是同分布的另一个采样值），这是不现实的。
> 事实上，我们在训练网络时也并没有期望神经网络能做到这种程度，而是让它去拟合某一种分布的目标图像们加噪加到这一步时所加噪声的整体分布。虽然我们在每一步加噪的时候确实都是随机取的标准高斯分布，但熵增容易熵减难，反过来想要从纯噪声中还原某一张原图，一步步减去的噪声采样值势必遵循某种规律。网络所拟合的正是某类目标图片所呈现出的这种共同规律。
>
> 那么，在网络预测出当前步所加的噪声之后，能不能直接用 $x_t$ 减去噪声获得 $x_{t-1}$ 呢？理论上可以这么做，这相当于认为 $\tilde{\beta}_t=0$，使得 Diffusion 的生成结果仅依赖于初始的噪声图，与 GAN generator 或 VAE decoder 一致，中间过程不再具有任何随机性（[[#DDIM]] 就探索了每一步采样的方差取其他值甚至取 $0$ 时的效果）。因此为了人为增加生成图像的多样性，我们并不把网络输出结果当做唯一确定的噪声值，而是由这个采样值计算出 $x_{t-1}$ 的分布均值，进而重新采样获得 $x_{t-1}$ 的某一可能结果。注意，对 $x_{t-1}$ 的采样过程诚然是重参数化的高斯分布采样，但因为网络给出了给定步数下符合某种既定规律的噪声采样值，所以确实有机会逐步还原（生成）出有意义的原图（符合训练集分布的新图）。
>
> 能不能逆用重参数方法，用 $x_t$ 预测出 $\epsilon_t$ 后一步到位求得 $x_{0}$ 呢？不可以，因为训练时噪声与真值样本是随机采样配对的，配对结果必然不甚合理（如果配对好，是可以训练步数更少的模型的，这就是 Diffusion 步数蒸馏的做法；另外，事实上 VAE 就是在学习一套合理的配对过程），不同样本的加噪路径轨迹存在交叉（交叉点处网络预测结果将是**不同样本路径的期望**），而标准高斯噪声是所有样本的最终归宿，因此实际上对应着无数样本，即使网络的学习能力再强，一步去噪的预测结果也是无数训练样本在这个位置上的平均噪声方向，朝着这个方向一条路走到底大概率不会到达有意义的样本点（因为高维空间中语义流形的稀疏性）。只有一步一步去噪，使得噪声图沿着合理的路径逐渐偏离标准正态分布，而成为一个特化的正态分布，此时采样一次才能获得足够清晰的结果。但如果步子不要跨这么大，每隔一定步数采样一次，还是可行的，并且可以加速采样过程。[[#IDDPM]] 和 [[#DDIM]] 都探索了类似的采样加速方法。
> 事实上，传统的扩散模型本质上是采用数值方法对常微分方程进行迭代求解。虽然可以通过设计更加精确的求解器来改善每一步的求解精度，减少所需要的迭代次数，但是这些方法中最好的也仍然需要 10 步左右的迭代步数来得到足够好的求解结果。[Consistency Models](https://github.com/openai/consistency_models) 为此探索了一步生成的方案。
>
> 一个比喻：设想一个玻璃杯被打碎，碎片四散的轨迹是随机的。然而给定一堆随机分布的玻璃碎片，如果想要还原出想要的玻璃杯，那么各个碎片的拼合必须就得按照一定的规律来（已知某一确定的纯噪声 $x_T$，去噪过程并不能完全随机）。最简单的做法是完全记录下当初打碎形成这一堆碎片的全过程，逆向还原。然而，这一堆碎片未必是由我们打碎并记录过的（Diffusion 推理时输入的高斯随机噪声 $x_T$ 不太可能恰好是训练时不断加噪形成的那些网络见过的噪声之一），并且还原的路径也不是唯一的，沿着不同的路径都有可能还原出玻璃杯，虽然细节可能略有差别（网络见过海量的加噪结果后，有能力推断出每一步去噪的大致走向，而在此基础上人为引入一定程度的随机性也是完全合理的）。
>
> （以上内容纯属个人理解，参考[都 2023 年了，我不允许你还不懂 DDPM！ - 知乎](https://zhuanlan.zhihu.com/p/663880249)）

![|800](Academic/assets/DDPM-algorithm.png)

DDPM 的高质量生成依赖于较大的 $T$（一般为 $1000$ 或以上）和较小的 $\beta_t$，这就导致 diffusion 的推断过程非常缓慢。

### IDDPM

> [!info]+
> **Improved Denoising Diffusion Probabilistic Models**
> ICML 2021; OpenAI
>
> - [Zotero](zotero://select/items/@nichol2021improved)
> - [URL](https://proceedings.mlr.press/v139/nichol21a.html)
> - [arXiv](https://arxiv.org/abs/2102.09672)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CIDDPM_ICML_2021.pdf)
> - [Code](https://github.com/openai/improved-diffusion)

在 DDPM 基础上的改进：

1. 可学习的方差
2. 优化方差计划
3. 基于 Loss 的重要性采样，以减少梯度中的噪声，解决损失曲线震荡较大的问题
4. 提高采样速度

DDPM 的噪声方差 $\beta_t$ 是超参数，且认为采用 $\beta_t$ 和 $\tilde{\beta}_t$ 的结果近似。
IDDPM 使用可学习的噪声方差，在这两个量之间自适应地寻找最合适的值，并为此新增了 VLB loss，与原始的 $L_{simple}$ 一起组成 $L_{hybrid}$。

> 实验观察发现，采用 DDPM 生成方差 $\beta_t$ 的方式（线性增大）会导致前向扩散时，靠后的时间步所加的噪声过多，即图像还没到最后的 $T$ 时间步就已经变为纯高斯噪声了。这就导致这些靠后的时间步在逆向过程中并没有太大的贡献，即使跳过也不会对生成结果产生多大的影响。

不同于 DDPM 的线性方差计划，IDDPM 采用了更缓和均匀的余弦变化方式。

在采样（正向过程）时，DDPM 采用与训练过程相同的 $t$ 值序列（可称之为 **Ancestral Sampling**）。然而，IDDPM 选择使用其子序列，以更大的间隔选取时间步 $s$。
实验发现，具有固定方差的 $L_{simple}$ 模型（较大的 $\sigma_t^2=\beta_t$ 和较小的 $\sigma_t^2=\tilde{\beta}_t$）在样本质量方面受到更大的影响，而具有学习方差的 $L_{hybrid}$ 保持了较高的样本质量。有了这个模型，100 个采样步骤就足以为完全训练的模型实现接近最优的 FID。

### DDIM

> [!info]+
> **Denoising Diffusion Implicit Models**
> ICLR 2021
>
> - [Zotero](zotero://select/items/@song2021denoising)
> - [arXiv](https://arxiv.org/abs/2010.02502)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CDDIM_ICLR_2021.pdf)
> - [Code](https://github.com/ermongroup/ddim)
> - [生成扩散模型漫谈（四）：DDIM = 高观点 DDPM - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9181)
> - [生成扩散模型(五): 采样加速 (Generative Diffusion Model: Sampling Acceleration) - 知乎](https://zhuanlan.zhihu.com/p/602222222)

主要贡献：

1. 构造**非马尔科夫**的扩散过程
2. 通过时间步子序列加速采样

训练 DDPM 时，损失函数实际上只依赖于（以 $x_0$ 为条件的）边际分布 $q(x_t|x_0)$，而并没有用到联合分布 $q(x_{1:T}|{x}_0)$（尽管 $q(x_t|x_0)$ 的表达式是从联合分布中推导出的）。这就意味着，实际上只要边际分布的形式不变，我们可以假定任意的联合分布的形式。
DDPM 中把联合分布定义为马尔科夫链，那么当然也可以把联合分布定义为非马尔科夫链的形式，只需要保证边际分布一样，那么就可以用相同的损失函数训练出相同的生成模型（注意这里使用“生成”，因为严格来说不是马尔科夫链的话，可能就不是“扩散”模型了），只需要根据新的前向过程，改变推断时的逆向去噪过程即可。

于是，可以构造一个非马尔科夫的逆向去噪过程：

$$
q(x_{t-1}|x_t,x_0)=
\mathcal{N}\left(\sqrt{\bar{\alpha}_{t-1}}x_0+\sqrt{1-\bar{\alpha}_{t-1}-\sigma_t^2}\cdot\frac{x_t-\sqrt{\bar{\alpha}_t}x_0}{\sqrt{1-\bar{\alpha}_t}},\sigma_t^2 \mathbf{I}\right)
$$

这仍然是一个高斯分布，且 $q(x_{t-1}|x_0)$ 的表达式与 DDPM 中的结果一致。通过贝叶斯公式可以获得其对应的前向加噪过程，每一时间步的状态不再只依赖于上一步的输出，也同样依赖原始输入。

这是一个更一般的采样（逆向去噪）过程。可以证明，DDPM 的去噪过程就是当 $\sigma_t$ 取关于 $\bar{\alpha}$ 的特定序列时的一个特例。
更特别地，当 $\sigma_t=0$ 时，最终生成的 $x_0$ 的随机性只依赖于初始的 $x_T$。也就是说，给定 $x_T$ 后，生成过程就变成确定性的了，从而可以看做一个潜变量模型 / 隐式模型（implicit model），即 DDIM。

此外，DDIM 认为 DDPM 的训练结果实质上包含了它的任意子序列参数的训练结果。因此，可以在逆向过程中使用更少步数的子序列，以加速采样过程。

标准 DDIM 在实现上与 DDPM 的区别在于其逆向去噪过程不含随机性：每次预测出噪声后，先根据加噪公式一步预测出原图，然后再将原图与刚才预测的噪声（即认为每一步的噪声都相同）线性组合形成某一步数下的带噪图（因此可以实现跳步）。

> 此外，对于 $\sigma_t=0$ 时的 DDIM，它就是将任意正态噪声向量变换为图片的一个确定性变换，这已经跟 GAN 几乎一致了，所以跟 GAN 类似，我们可以对噪声向量进行插值，然后观察对应的生成效果。但要注意的是，DDPM 或 DDIM 对噪声分布都比较敏感，所以我们不能用线性插值而要用球面插值。

### Guided-Diffusion

> [!info]+
> **Diffusion Models Beat GANs on Image Synthesis**
> NIPS 2021; OpenAI
>
> - [Zotero](zotero://select/items/@dhariwal2021diffusion)
> - [URL](https://proceedings.neurips.cc/paper/2021/hash/49ad23d1ec9fa4bd8d77d02681df5cfa-Abstract.html)
> - [arXiv](https://arxiv.org/abs/2105.05233)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CGuided-Diffusion_NIPS_2021.pdf)
> - [Code](https://github.com/openai/guided-diffusion)
> - [生成扩散模型漫谈（九）：条件控制生成结果 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9257)

通常而言，对于通用图像生成任务，加入类别条件能够比无类别条件生成获得更好的效果，这是因为加入类别条件的时候，实际上是大大减小了生成时的多样性。

Guided Diffusion 提出了一种简单有效的类别引导的扩散模型生成方式，可以由无条件的逆向过程结合生成结果的分类损失来度量。其核心思路是在逆向过程的每一步，用一个分类网络对生成的图片进行分类，再基于分类分数和目标类别之间的交叉熵损失计算梯度，用梯度引导下一步的生成采样（相当于让图像一方面向训练集分布方向扩散，一方面向分类得分高的方向扩散）。这个方法一个很大的优点是，不需要重新训练扩散模型，只需要在前馈时加入引导即能实现相应的生成效果。

首先，单独训练一个分类模型，然后在每一步逆向去噪的过程中，在计算高斯分布的均值时加上方差和**分类损失关于输入图像的梯度**的乘积（由贝叶斯定理的对数形式推导出）。基于这样的改进，不需要重新训练扩散模型，只需要额外训练一个分类器，就能够有效地添加类别引导。

此外，Guided Diffusion 还对 DDPM 中采用的 U-Net 结构的 Autoencoder 进行了一些结构上的改进，包括加深网络、增加 attention head 数量、增加 attention layer 的尺度数量、采用 BigGAN 的残差模块结构。另外，还采用了一种称为 Adaptive Group Normalization（AdaGN）的归一化模块。

### 🔥 Classifier-Free Diffusion Guidance

> [!info]+
> **Classifier-Free Diffusion Guidance**
> NeurIPS Workshop 2021; Google
>
> - [Zotero](zotero://select/items/@ho2021classifierfree)
> - [URL](https://openreview.net/forum?id=qw8AKxfYbI)
> - [arXiv](https://arxiv.org/abs/2207.12598)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CClassifier-free_diffusion_guidance_NeurIPS_Workshop_on_Deep_Generative_Models_and_Downstream_Applications_2021.pdf)

Guided Diffusion 等工作都采用了额外的网络前向 + 梯度计算的形式，虽然训练成本低、见效快，但也存在着一些问题：推断时额外的计算量比较多；引导函数和扩散模型分别进行训练，不利于进一步扩增模型规模，不能够通过联合训练获得更好的效果。

因此，DDPM 的作者提出了 Classifier-Free Diffusion，为噪声估计网络模型加入了额外的 condition 条件输入（而不止是 $x_t$ 和 $t$）。

在训练时，随机将一部分样本的 condition 置为空（表示无条件，如分类任务中规定为全 0 等），以训练同时支持有条件和无条件的扩散模型。逆向去噪时，对于每一步，需要过两次噪声估计网络，一次有条件，一次无条件（或负向条件），然后将二者结果**相减**（由贝叶斯定理的对数形式推导出），代替 Guided Diffusion 中的梯度项来引导生成结果。

实际实现中，有条件和无条件的噪声估计在相减时有一个超参数 CFG scale (classifier-free guidance scale) 来控制 condition 的引导力度（`noise_pred = noise_uncond + cfg_scale * (noise_cond - noise_uncond)`）。CFG=0 时，相当于无条件生成（完全忽略 condition）；CFG=1 时，相当于 vanilla guidance（引导力度不够）；随着 CFG 增大，生成结果对 condition 的遵循程度上升（排除无关的噪声空间），同时多样性下降。CFG 很大时，会产生过饱和的图像。平衡引导力度与多样性的最佳 CFG 一般为 7.5 左右。

上述过程中，可以将无条件生成的预测结果改为 negative prompt 的预测结果，从而支持负向提示词。

一些任务（如 inpainting）的结果很难作为 condition 来反向传播梯度，因此不适用于 Guided Diffusion 结构，而 Classifier-Free Diffusion 就可以实现对其的条件控制。

### GLIDE: Guided Language to Image Diffusion for Generation and Editing

- **Paper**: [GLIDE: Towards Photorealistic Image Generation and Editing with Text-Guided Diffusion Models](https://proceedings.mlr.press/v162/nichol22a.html) (ICML 2022; OpenAI)
- [arXiv](https://arxiv.org/abs/2112.10741)
- [Code](https://github.com/openai/glide-text2im)

基于 Classifier-Free Diffusion，改为了文本引导，并采用了更大的模型，效果优于 [DALL·E](https://openai.com/research/dall-e)。

后续 OpenAI 将其进一步改进扩展，推出了 [**DALL·E 2**](https://openai.com/product/dall-e-2)（[Hierarchical Text-Conditional Image Generation with CLIP Latents](https://arxiv.org/abs/2204.06125)）。

此外，Google 也很快基于条件扩散模型推出了自己的文生图模型 [**Imagen**](https://imagen.research.google/)（[Photorealistic Text-to-Image Diffusion Models with Deep Language Understanding](https://arxiv.org/abs/2205.11487)）。

### Latent Diffusion (Stable Diffusion)

> [!info]+
> **High-Resolution Image Synthesis with Latent Diffusion Models**
> CVPR 2022
>
> - [Zotero](zotero://select/items/@rombach2022highresolution)
> - [URL](https://openaccess.thecvf.com/content/CVPR2022/html/Rombach_High-Resolution_Image_Synthesis_With_Latent_Diffusion_Models_CVPR_2022_paper.html)
> - [arXiv](https://arxiv.org/abs/2112.10752)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CLatent_Diffusion_CVPR_2022.pdf)
> - [Code](https://github.com/CompVis/latent-diffusion)

![|800](Academic/assets/Latent_Diffusion.jpg)

尽管前面的 Diffusion Models 都是直接在完整的图像像素上完成的，但事实上模型的 $x_t$ 可以是任何尺寸和含义的内容，只要噪声预测网络的维度与之匹配即可。

在 Latent Diffusion 中，先训练好一对 encoder-decoder，能够实现图像域与特征域（latent）之间的转换，然后在特征域完成 Diffusion 的过程。

这种方法能够处理更复杂的图像分布，且由于将图像压缩到了特征空间（隐空间），消耗的资源也相对更少。

此外，Latent Diffusion 也支持 condition，将任意模态的信息经过编码后，将其与 U-Net 的中间层进行 cross-attention，以将条件信息引入网络。

Latent Diffusion / SD 包含三个重要的网络组件（[参考](https://huggingface.co/blog/stable_diffusion)）：

1. VAE，用于 image-latent 空间的转换
2. UNet：ResNet blocks with short-cut connections & self/cross attention
3. CLIP：Tokenizer，用于编码 text-image condition，作为 UNet 中 cross-attention 层的输入

### SDEdit

[SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations](https://sde-image-editing.github.io/) (ICLR 2022)

![](https://sde-image-editing.github.io/images/sde_stroke_generation.jpg)

先把输入图像加一定噪声，然后再去噪，以完成图像编辑

### V-prediction

> [!info]+
> **Progressive Distillation for Fast Sampling of Diffusion Models**
> ICLR 2022
>
> - [arXiv](https://arxiv.org/abs/2202.00512)

[扩散模型中的 v-prediction - 知乎](https://zhuanlan.zhihu.com/p/678942992)

不直接预测 $\epsilon$，而是预测 $\epsilon$ 与 $x_0$ 的某个设计好的线性组合：

$$
v_t = \sqrt{\bar{\alpha}_t} \epsilon - \sqrt{1-\bar{\alpha}_t} x_0
$$

根据 $x_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1-\bar{\alpha}_t} \epsilon$，也可以表示为：

$$
v_t = \frac{\sqrt{\bar{\alpha}_t} x_t - x_0}{\sqrt{1-\bar{\alpha}_t}}
$$

或：

$$
v_t = \frac{1}{\sqrt{\bar{\alpha}_t}} \epsilon - \frac{\sqrt{1-\bar{\alpha}_t}}{\sqrt{\bar{\alpha}_t}} x_t
$$

相比 $x_0$-prediction 和 $\epsilon$-prediction, $v$-prediction 在训练中的收敛性和数值稳定性更好。

解释：$\epsilon = \frac{x_t - \sqrt{\bar{\alpha}_t} x_0}{\sqrt{1-\bar{\alpha}_t}}$，当 $t \approx 0$（低噪声）时，$x_t \approx x_0$，此时从 $x_t$ 中分离 $\epsilon$ 的任务就变得极其困难、敏感且数值不稳定：预测 $\epsilon$ 时，由于系数 $\sqrt{1-\bar{\alpha}_t}$ 很小且在分母上，网络内部就需要预测出一个极大的系数来把观测到的微弱噪声放大到希望的输出尺度，而此过程中网络参数的微小变化或 $x_t$ 的微小扰动，都可能导致预测的 $\epsilon$ 发生剧烈变化，这就会导致训练不稳定。

可见，$\epsilon$-prediction 试图从输入中提取噪声，这在不同 $t$ 下的任务难度差异巨大，网络能观察到的 $x_t$ 中的噪声成分也存在着巨大的尺度差异，这就导致网络需要针对不同 $t$ 学习出动态范围极大的放大方式（为了统一输出标准正态分布尺度的 $\epsilon$ 估计）。

相比之下，从表达式中可以看出，当 $t \approx 0$（低噪声）时，$v_t \approx \epsilon$，网络学习从 $x_t \approx x_0$ 中预测 $\epsilon$；当 $t \approx T$（高噪声）时，$v_t \approx -x_0$，网络学习从 $x_t \approx \epsilon$ 预测 $-x_0$，任务难度始终一致，网络就不会试图去完成更困难的噪声提取任务，而是转而用输入的信息去估计加噪轨迹，进而条件生成一个在当前位置处的轨迹切线上的合理速度量。

注意：$v$-prediction 在数学上和另两种预测方式是等价的，预测结果可以用公式相互转化，区别只在于网络学习的难易和数值稳定性（$v$-prediction “更好学”）。

Stable Diffusion 2 开始使用 $v$-prediction 代替 $\epsilon$-prediction。

### Textual Inversion

> [!info]+
> **An Image is Worth One Word: Personalizing Text-to-Image Generation using Textual Inversion**
> ICLR 2023; NVIDIA
>
> - [Zotero](zotero://select/items/@gal2022image)
> - [URL](https://openreview.net/forum?id=NAQvF08TcyG)
> - [arXiv](https://arxiv.org/abs/2208.01618)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CTextual_Inversion_ICLR_2023.pdf)
> - [Project](https://textual-inversion.github.io/)
> - [Code](https://github.com/rinongal/textual_inversion)

Motivation：现有的 Diffusion-based 能力强大，然而从用户角度仍不够灵活，比如若要把现实中的一些新概念（大模型没有见过的 personalization 事物）直接引入现有的大模型是很困难的——re-train 成本高；finetune 会导致灾难性遗忘。以前的大部分方法都是冻住原来的模型，然后加多层结构来作为下游任务的 adaptor，但还是存在 prior knowledge 遗忘的问题。

本文基于 [[#Latent Diffusion (Stable Diffusion)|Latent Diffusion]]，只改进了其中 text encoder 的词表部分，添加了 special token $S*$（pseudo-word）来表示新概念，保留原有其他 token 的 embedding 不变，从而实现与新概念的组合。

为了学习 $S*$，模仿了 [[#CLIP: Contrastive Language-Image Pre-training|CLIP]] 的 prompt "A photo of $S*$" 来生成新的图片，然后约束生成的图像与用户所给的少量关于新概念的图像相似。训练完成之后就可以将 $S*$ 与新的 prompt 句式结合，来做基于新概念的生成了。

### DreamBooth

> [!info]+
> **DreamBooth: DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation**
> CVPR 2023; Google
>
> - [Zotero](zotero://select/items/@ruiz2023dreambooth)
> - [URL](https://openaccess.thecvf.com/content/CVPR2023/html/Ruiz_DreamBooth_Fine_Tuning_Text-to-Image_Diffusion_Models_for_Subject-Driven_Generation_CVPR_2023_paper.html)
> - [arXiv](https://arxiv.org/abs/2208.12242)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CDreamBooth_CVPR_2023.pdf)
> - [Project](https://dreambooth.github.io/)
> - [Code](https://github.com/XavierXiao/Dreambooth-Stable-Diffusion)

任务与 [[#Texual Inversion]] 类似，都是基于新概念的生成。与之不同的是，DreamBooth 是通过 finetune diffusion model（Imagen）来实现的（而非训练 prompt，这会导致新概念的表达局限于原始模型的 domain）。

DreamBooth 用少量个性化图片（3 到 5 张，指定 class）微调 diffusion model，配对的 prompt 均为 `a [identifier] [class noun]` 的形式。但如果只用普通的微调方式，会出现过拟合和语言漂移（在特定任务上微调时，模型会逐渐忘记通用知识，而仅仅适配特定的任务）两个问题。

因此，DreamBooth 提出「使用预训练模型自己生成的图片来监督微调过程」的方法，即用形式为 `a [class noun]` 的 prompt，让原始模型生成一些图像，把它们与个性化图像合在一起微调新模型，防止微调后模型忘记通用知识。

### ControlNet

> [!info]+
> **Adding Conditional Control to Text-to-Image Diffusion Models**
> ICCV 2023
>
> - [Zotero](zotero://select/items/@zhang2023adding)
> - [URL](https://openaccess.thecvf.com/content/ICCV2023/html/Zhang_Adding_Conditional_Control_to_Text-to-Image_Diffusion_Models_ICCV_2023_paper.html)
> - [arXiv](https://arxiv.org/abs/2302.05543)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CControlNet_ICCV_2023.pdf)
> - [Code](https://github.com/lllyasviel/ControlNet)

![|600](Academic/assets/ControlNet.png)

> It copys the weights of neural network blocks into a "locked" copy and a "trainable" copy.
> The "trainable" one learns your condition. The "locked" one preserves your model.
> Thanks to this, training with small dataset of image pairs will not destroy the production-ready diffusion models.
> Before training, all zero convolutions output zeros, and ControlNet will not cause any distortion.
> No layer is trained from scratch. You are still fine-tuning. Your original model is safe.
> This allows training on small-scale or even personal devices.
> This is also friendly to merge/replacement/offsetting of models/weights/blocks/layers.
>
> **Stable Diffusion** + **ControlNet**:
> By repeating the above simple structure 14 times, we can control stable diffusion in this way.

### 🔥 LoRA

> [!info]+
> **LoRA: Low Rank Adaptation of Large Language Models**
> ICLR 2022; Microsoft
>
> - [Zotero](zotero://select/items/@hu2022lora)
> - [URL](https://openreview.net/forum?id=nZeVKeeFYf9)
> - [arXiv](https://arxiv.org/abs/2106.09685)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CLoRA_ICLR_2022.pdf)
> - [Code](https://github.com/microsoft/LoRA)
> - [梯度视角下的 LoRA：简介、分析、猜测及推广 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9590)

一种参数高效的微调方法，固定预训练参数矩阵不变，只学习一个（初始化输出全 0 的）增量项，由低秩分解的两个小矩阵 A（`nn.Linear(in_features, rank)`）和 B（`nn.Linear(rank, out_features)`）的乘积构成，rank 的常见取值有 64、128 等。

LoRA  与 Transformer 的结合很简单，只需在 QKV 矩阵的计算过程中增加一个旁路（两层 Linear）。

训练完成后，低秩矩阵可以合并到原始权重矩阵中，以供高效推理：`W_merged = W + BA`。

与 ControlNet 将原始模型进行无压缩的参数复刻相比，LoRA 的 Low Rank 压缩方式大大减少了模型体积和训练成本，但一定程度上也限制了它的能力上限。
因此目前的图像生成控制的标准实践，大多数都是用一个基础模型来控制图像的宏观能力和基调，配合 LoRA 来控制图像风格或指定特定内容生成，然后再通过 ControlNet 来精确控制图像结构（这在 LoRA 的能力之外）。

### DiT

> [!info]+
> **Scalable Diffusion Models with Transformers**
> ICCV 2023
>
> - [Zotero](zotero://select/items/@peebles2023scalable)
> - [URL](https://ieeexplore.ieee.org/document/10377858/)
> - [arXiv](https://arxiv.org/abs/2212.09748)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CDiT_ICCV_2023.pdf)
> - [Project](https://www.wpeebles.com/DiT)
> - [Code](https://github.com/facebookresearch/DiT)
> - [AIGC 专栏 9——Scalable Diffusion Models with Transformers （DiT）结构解析\_scalable diffusion models with transformers pdf-CSDN 博客](https://blog.csdn.net/weixin_44791964/article/details/136276539?utm_source=702048761)

![](https://www.wpeebles.com/images/DiT/block.png)

[DiT：从理论到实践，万字长文深入浅出带你学习 Diffusion Transformer - 知乎](https://zhuanlan.zhihu.com/p/711055614)

## ✨ Flow

### Rectified Flow

> [!info]+
> **Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow**
> ICLR 2023
>
> - [Zotero](zotero://select/items/@liu2023flow)
> - [URL](https://openreview.net/forum?id=XVjTT1nw5z)
> - [arXiv](https://arxiv.org/abs/2209.03003)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CRectified_Flow_ICLR_2023.pdf)

[\[ICLR2023\] 扩散生成模型新方法：极度简化，一步生成 - 知乎](https://zhuanlan.zhihu.com/p/603740431)（原作者）

[生成扩散模型漫谈（十七）：构建 ODE 的一般步骤（下） - 科学空间|Scientific Spaces](https://www.spaces.ac.cn/archives/9497)

[零推导理解 Diffusion 和 Flow Matching - 知乎](https://zhuanlan.zhihu.com/p/11228697012)

[扩散模型去伪求真，直线真的重要吗？！Rectified Diffusion，重新思考 Rectified Flow - 知乎](https://zhuanlan.zhihu.com/p/889632332)

为了实现一步生成，用常微分方程（Ordinary Differential Equation, ODE），或者叫流模型（flow）来建模两个分布之间的映射，然后让分布迁移的轨迹“走直线”。

Diffusion 加噪的方差计划决定了加噪过程走的是圆弧。将加噪过程改为简单的线性叠加即可使得理想的去噪路径变为直线：

$$
x_t = t x_1+ (1-t) x_0
$$

注意：Flow 的习惯记号与 Diffusion 相反，$x_{0}$ 代表的是纯噪声，而 $x_{1}$ 代表原始数据，中间时间步的取值范围是 0~1，即去噪过程是从 $x_{0}$ 到 $x_{1}$ 的过程。

Flow 的这种线性插值的采样方式使得它能够建模任意两个分布之间的变换（而非像 Diffusion 那样只能从高斯分布开始）。

网络实际采用 v-prediction，即预测一个在所有时间步均恒定的去噪速度：

$$
v = x_1 - x_0
$$

然而，由于网络实际预测的结果是不同轨迹的期望而并非最理想的直线，且还要在直线轨迹的交叉点处做路径重组（神经预测的因果性约束），所以上面的 ODE 模型（或者说 flow）的轨迹仍然是弯曲（分段直线）的，仍然无法实现一步生成。此外，我们指定的直线只是欧几里得空间中的朴素最短路径，实际在高维空间和语义流形上的最优、最自然的去噪轨迹未必一定是直线。

于是，提出了本文最核心的贡献：用 Reflow 方法“拉直轨迹”：把用上述方法训练好的 1-Rectified Flow 的输入噪声和输出图像作为固定的配对（不再是随机配对），再训练一个 2-Rectified Flow 模型（类似 diffusion 步数蒸馏的做法）。
这种训练方式迭代产生的新路径是由模型自己学出来的最自然的分布转换方式，比最初假设的直线更优。

注意：Flow 与 Diffusion 除了加噪方式上的区别，还有生成时采样上的区别。Flow 本质是求解一个 ODE，其生成过程是确定性，前一步加噪图可以直接由 Euler sampling 求出：

$$
x_{t + \Delta t} = x_{t} + v \cdot \Delta t
$$

而 Diffusion 本质是求解一个 SDE，它的生成过程是具有随机性的，每一步都是先确定噪声均值方差，然后再从中采样出加噪图。

注意：线性加噪计划和 v-prediction 都不是 reflow 的必要条件，原始的 DDPM 一样可以通过迭代训练的 reflow 方法获得更好的性能（详见 [Rectified Diffusion](https://zhuanlan.zhihu.com/p/889632332)）。

### Flow Matching

> [!info]+
> **Flow Matching for Generative Modeling**
> ICLR 2023
>
> - [Zotero](zotero://select/items/@lipman2023flow)
> - [URL](https://openreview.net/forum?id=PqvMRDCJT9t)
> - [arXiv](https://arxiv.org/abs/2210.02747)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CAIGC%5CFlow_Matching_ICLR_2023.pdf)

与 [[#Rectified Flow]] 是同期工作，idea 基本一致

[全网最易懂的 Flow Matching 详解](https://mp.weixin.qq.com/s/HzFVs1_qnZc05DkLiUwiPA)

### FLUX.1 Kontext

[FLUX.1 Kontext: Flow Matching for In-Context Image Generation and Editing in Latent Space](https://arxiv.org/abs/2506.15742) (2025.06)

![](https://arxiv.org/html/2506.15742v2/extracted/6566027/img/kontext_v2.jpg)

另一种形式的 In-context model：target image 和 condition image 分别过 VAE，然后 in-context attention。

## ✨ SSM

SSM: State Space Model

[重温 SSM（一）：线性系统和 HiPPO 矩阵 - 科学空间|Scientific Spaces](https://kexue.fm/archives/10114)

### Mamba

> [!info]+
> **Mamba: Linear Time Sequence Modeling With Selective State Spaces**
> 2024
>
> - [Zotero](zotero://select/items/@gu2024mamba)
> - [arXiv](https://arxiv.org/abs/2312.00752)
> - [PDF](file:///D:%5CMedia%5CDocuments%5C%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3%5CAcademic%5CZotero%5Cstorage%5CDL%5CMamba_arXiv_2024.pdf)
> - [Code](https://github.com/state-spaces/mamba)
