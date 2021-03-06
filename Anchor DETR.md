# Anchor DETR

论文地址：<https://arxiv.org/abs/2109.07107> 

源码地址：<https://github.com/megvii-research/AnchorDETR> 

![](http://192.168.1.247:88/lib/7832f585-7eec-4eae-b7a7-b06af495550b/file/images/auto-upload/image-1636622558538.png?raw=1)

### 1、摘要

提出了新的基于transformer 的检测的查询设计。在之前的基于transformer的目标检测器，查询目标是一些学习的嵌入的集，但是每个学习的嵌入没有比较显示的物理意义，所以我们无法解释它将注意在哪部分。由于每个查询目标的预测槽位是没有特定的模式，所以是非常难以优化。换一句话说，每个查询目标不会注意在特定的区域。为了解决这些问题，我们的查询设计里，查询目标基于广泛应用与基于CNN的检测器的锚点的。所以每个目标查询注意在锚点附近。而且，这里的查询设计可以在一个位置附近预测多个物体，解决了“一个区域，多个物体”的难题。

另外，我么设计了个注意力变量，它可以相较于detr的标准注意力模块达到近似或更好的表现的同时，减少了内存的使用。由于查询设计和注意力变换，提出的检测器称为Anchor DETR，可以获得更好的表现，也比DETR运行快十倍的代数。比如，在MSCOCO数据集上，用ResNet50-DC5特征训练50代，可以达到44.2AP的精度和16FPS的推理速度。在MSCOCO基线加强实验，证明了提出的方法的有效性。代码已经开源

## 介绍

目标检测任务是在一张图中预测每个感兴趣的目标的一个包围盒和分类。在近十年里，基于CNN的目标检测有了非常大的发展  (Ren et al. 2015; Cai and Vasconcelos2018; Redmon et al. 2016; Lin et al. 2017; Zhang et al. 2019;Qiao, Chen, and Yuille 2020; Chen et al. 2021)

最近提出的DETR，是基于transformer的新方法。它使用一个学习到的物体查询集合来推理物体和图片上下文的关系。然而，这个学习的目标查询是非常难以解释。这个查询没有比较显示的物理意义，每个目标查询相关的预测槽没有特定的模式。如图1（a），DETR里每个目标查询的预测和不同的面积有关，每个目标查询负责一个非常大的面积。这样的位置模糊性，如目标查询不是注意在特别的区域，所以难以优化。

![](http://192.168.1.247:88/lib/7832f585-7eec-4eae-b7a7-b06af495550b/file/images/auto-upload/image-1636527554322.png?raw=1)

Figure 1: 预测槽的可视化. 子图 (a) 来自 DETR的 (Carion et al.2020). 每个预测槽位包括所有的在一个验证集上的包围盒预测 . 每个带色的点代表归一化的预测的中心点， 绿色的点代表小包围盒，红色是比较宽的盒子，蓝色是比较高的盒子。黑色的点在最后行的子图（b），表示anchor锚点。 我们的预测槽位比DETR比起来更和一个特定位置相关。 

回顾基于CNN的目标检测器，anchors和位置高度相关，有可解释的物理意义。受到这种动机的启发，本文提出一个新的基于锚点的查询设计，我们将锚点编码成目标查询。目标查询是每个锚点位置编码以至于每个目标查询有显示的物理意义。

但是，这解决方法也遇到问题：在一个位置可能有多个物体出现。在这情况下，在这个位置只有一个目标查询是没法预测多个物体的，所以其他位置的目标查询需要协同预测这些目标。这样导致每个目标查询负责更大的面积。于是，我们通过对每个锚点增加多个模式改进目标查询设计，来预测多个目标物体。如图1（b）所示，每个目标查询的所有三种模式的预测都分布于关联锚点附近。换句话就是，每个目标查询只是关注关联锚点附近的物体。所以提出的目标查询很容易解释。由于目标查询有特别的模式和不需要预测离相关位置很位置很远的物体，所以也有利优化。

除了查询设计，我们也设计了称作行-列分离注意力的注意力变体（RCDA）。将2维关键特征分离成1维的行和列，然后相继操作这两个注意力。RCDA可以减少内存使用的同时，获得相比于DETR传统注意力机制相似或更好的表现。我们相信这是DETR标准注意力的很不错的替代。

![](http://192.168.1.247:88/lib/7832f585-7eec-4eae-b7a7-b06af495550b/file/images/auto-upload/image-1636529875426.png?raw=1)

所以如表1，新的基于锚点的查询设计和注意力变体，使得AnchorDETR 比DETR运行快10倍，用同样单层特征情况下。

而和其他类似DETR的检测算法同样的代数下有更好的表现，用单ResNet50-DC5特征训练50代达到了44.2 AP值和16FPS速度。

主要的贡献可以归纳为：

为基于transformer的检测提出了新的基于锚点的查询设计。而且给每个锚点增加了更多模式使得可以在一个位置预测多个物体。算法具有更好的可解释性并且比学习嵌入更好优化。

注意力变体，Row-Column Decouple Attention，和标准DETR的attention比较相近或更好的表现下，可以减少内存使用。是个好滴代替者。

扩展实验可以证明每个组件的有效性。

## 相关工作

### 目标检测里的Anchor

在基于CNN的目标检测中有两种anchor，anchor boxes 和anchor points  。手动设计的anchor boxes要仔细的调整来获得好滴表现，所以我们不选择使用这个anchor boxes。我们通常认为anchor-free 就是anchor boxes free，所以用anchor point的检测器也视为anchor-free 的。DETR不选择anchor boxes 也不使用anchor points。它直接预测图像中每个目标的绝对位置。然后，我们发现在目标检查中引入anchor point会更好。

### Transformer 检测器

Vaswani 等 (Vaswani et al. 2017) 最早提出了序列转换的transformer。最近，Carion 等提出了DETR，是基于transformer的目标检测。transformer检测器会将值的信息根据查询和关键词的相似度输入查询里。朱等，提出了Deformable DETR 可变DETR，通过采样可变点的值到查询和使用多层次特征的方法解决transformer检测器收敛速度慢的问题。Gao et al. (Gao et al. 2021) 在每个查询的原始注意力里增加了高斯映射。

与我们同时的还有，条件DETR，将参考点编码为位置查询，但是我们的动机不意义，他们是只用参考点生成位置嵌入信息，作为交叉注意力的条件空间嵌入，而目标查询还是学习的嵌入。另外，条件DETR没有一个位置多个预测物体也没有注意力变体。

### 高效注意力

transformer的自注意力机制有很高的复杂性，所以他很难处理非常大规模的查询和关键词。为了解决这个问题，人们提出了很多高效的注意力模块： (Wang et al. 2020b;Shen et al. 2021; Vaswani et al. 2017; Beltagy, Peters, andCohan 2020).最高效的方法是计算可以先造成线性复杂度的计算量的查询和关键词。高效的注意力和Linformerr (Wang et al. 2020b) 就是用了这个思想。另一个方法是固定限制每个查询的关键词的注意力区域而不是整个区域。The 有限自注意力Self-Attention (Vaswani et al. 2017), 可变注意力 Deformable Attention (Zhu et al. 2020), Criss-Cross Attention (Huanget al. 2019), 和LongFormer (Beltagy, Peters, 和Cohan2020)借鉴了这思想. 本文用1维全局平局池化将关键特征分解到行特征和列特征，  并相继处理行注意力和列注意力。 

## 方法

### 锚点 Anchor Points

基于CNN的检测里，锚点总是和特征图的位置一致。但是在基于transformer的检测里可以更灵活。锚点可以是学习的点，均匀网格点，或是手动设计的其他anchor点，我们使用两种锚点。一个是网格锚点另一个是学习的锚点。如图2（a）所示，网格锚点固定为图片中的均匀栅格节点。学到的节点则通过0到1均匀分布随机初始化，并根据学习参数更新。有了锚点，预测包围盒的中心位置 ( ˆcx, cyˆ ) 就会加到相关的锚点，作为最后的预测，比如可变DETR那样。

![](http://192.168.1.247:88/lib/7832f585-7eec-4eae-b7a7-b06af495550b/file/images/auto-upload/image-1636593900295.png?raw=1)

### 注意力公式

类似DETR的transformer的注意力可以用公式表示：

$Attention(Q,K,V) = softmax(\\frac{QK^T}{\\sqrt{d_k}})V$ 

$Q = Q_f + Q_p ,K = K_f+K_p,V=V_f$  

其中 $d_k$ 为通道维度 ，下标f 表示特征，下标p表示位置嵌入。注意到 Q , K , V 会分别通过一个线性层，为了更简洁在公式里省略了 。 

在DETR解码层有两个注意力机制，一个是自注意力，另一个是交叉注意力。 在自注意力里，$K_f$ 和 $V_f$ 与 $Q_f$ 相同，$K_p$ 和 $Q_p$ 一样。

$Q_f ∈R^{N_q × C} $ 是最后解码器的输出，第一个解码器的初始值为 $Q_f^init∈R^{N_q × C} $,可以设置为常数向量，或是学到的嵌入。对于查询位置嵌入 $Q_p∈R^{N_q × C} $ ,使用DETR里的学到嵌入的集合：

$Q_p = Embedding(N_q,C)  (2)$

在交叉注意力里，$Q_f = ∈R^{N_q × C} (2)$ 是由前面自注意力的输出来生成，$K_f ∈R^{HW × C} $ ,$V_f ∈R^{HW × C}$为编码器的输出特征。$K_p ∈R^{HW × C} $ 为 $K_f$ 的位置嵌入。它是由sine-cosine位置编码函数$g_sin$生成的。

 $g_sin$是基于相关关键特征位置$Pos_k∈R^{HW × 2} $函数。：

$K_p = g_sin(Pos_k)  (3)$

这H，W，C代表长，宽，特征通道，$N_q$是预先定义的查询数量。

 锚点到目标查询

通常来说，解码器的$Q_p$ 被作为目标查询，因为它负责区分不同的物体目标。正如前面介绍的，像公式（2）的学习到嵌入的目标查询是非常难以解释的。

本文提出了基于锚点 $Pos_q ∈ R ^{N_A × 2} 的目标查询 。其表示 $N_A$ 点集，坐标是（x,y) 取值范围是0到1。 

基于锚点的目标查询可以公式化为：

$Q_p=Encode(Pos_q)$

换句话说可以编码锚点为目标查询。

  最自然的方式是共享共同的位置嵌入函数，作为key：

$Q_p = g(Pos_q), K_p = g(Pos_k) $

g 是位置嵌入函数。位置嵌入函数可以是sin 或是别的函数。本文选择一个小的两层线性层的MLP网络来增量适应，而不是启发式的sin。

### 每个锚点多个预测

为了解决一个位置有多个目标物体的情况，本文进一步改进目标查询来在一个锚点预测多个物体，而不是一个。

回顾最初的查询特征 $Q_f^init ∈R^{N_q×C}$, 每个$N_q$ 目标查询有一个模式  $Q_f^i  ∈ R^{1×C }$ , i 是物体查询索引。 为了对一个锚点做多个物体预测，本文结合了多种模式到每个目标查询里。 这里使用小集合模式的嵌入 $Q_f^i  ∈ R^{N_p×C }$ :

$ Q_f^i = Embedding(N_p,C) $

 为了在每个位置用不同模式检测物体。$N_p$ 是非常小的模式数量，如 $N_p=3$ 

由于平移不变性的特点，各模式可以被所有目标查询共享。这里通过共享$Q_f^i ∈ R ^{N_p×C }$给各个$Q_p∈^{N_A×C }$ 初始化$Q_f^init ∈ R ^{N_p N_A×C }$ 和 $Q_p ∈ R^{N_p N_A×C }$.

这里 $N_q$ 和 $N_p × N_A$ 相等。 于是可以定义提出的模式位置查询设计如下：

$Q = Q_f^init + Q_p $

对于接下来的解码器，$Q_f $ 也如DETR是由最后的解码器输出生成。

这样的设计，使得解码器有更好的可解释性并获得更好的表现，比原始DETR有更少十倍的训练代数。

### 行列分解的注意力机制

transformer会消耗更多GPU内存，所以限制了在高分辨率特征或其他扩展使用。Deformable Transformer 可以减少内存使用，但是会导致不利于硬件的随机内存访问。也有些注意力机制可以有线性复杂性，不会导致随机内存访问。但是我们发现这些注意力模块无法处理DETR类的检测器。

本文提出的行-列分解注意力（RCDA）可以不仅减少内存压力也可以获得比标准DETR注意力更好的表现。RCDA主要的思想是将关键特征 $K_f ∈ R ^{H×W×C}$ 通过1维最大平均池化分解到行特征 $K\_{f,x}∈R^{W×C}$和列特征 $K\_{f,y}∈ R ^{H×C}$ 。然后相继处理行注意力和列注意力。

避免丢失广泛性，可以假设$W ≥ H$ RCDA可以定义为

![](http://192.168.1.247:88/lib/7832f585-7eec-4eae-b7a7-b06af495550b/file/images/auto-upload/image-1636617370365.png?raw=1)

weighted_sumW 和 weighted_sumH 算子分别计算的是宽维度和高维度的加权求和。$Pos\_{q,x}∈ R^{N_q×1}$是$Q_f∈R^{N_q×C}$相关行位置, $Pos\_{q,y} ∈ R^{N_q×1}$ , $Pos\_{k,x} ∈ R^{W×1}$  $Pos\_{k,y} ∈ R^{H×1}$ 也是同样的方法。   

g1D和g一样是1维位置编码函数，它将1维坐标编码为带C通道数的一个向量。

为了清晰 我们假设之前公式的多头注意力的头数为1，这里考察内存分析，DETR力注意力的主要内存消耗是注意力权重映射 $A∈R^{N_q×H×W×M} $. 但是RCDA的注意力权重映射为$A_x∈R^{N_q×W×M} $和  $A_x∈R^{N_q×H×M} $

所以内存消耗是比A少很多。但是RCDA主要内存消耗是参数结果Z。所以必须将 $A∈R^{N_q×H×W×M} $和  $Z∈R^{N_q×H×C} $ 。RCDA节省内存比例是

$r = (N_q × H × W × M)/(N_q × H × C)= W × M/C$

默认的设置是 M=8 ，C=256 。 所以内存消耗是和大边W等32 ，也就是目标检测的C5特征特定值大致相同。

当使用高分辨率特征时，可以节省内存，比如C4或DC5节约大概2倍内存，C3特征大约节省4倍内存。

## Experiment

实验细节

实验基于MSCOCO数据集基准。所有模型在train2017划分数据集训练，并在val2017数据集验证。训练使用8个GPU ，每个GPU一张图片。训练使用AdamW优化器训练50 epoch。初始学习率 为backbone 为$10^-5$，其他是$10^-4$

学习率将在40 代时降低0.1倍。权重衰减设为$10^-4$，dropout率为0.

注意力头数为8，注意力特征通道是256，前馈网络的隐藏层维度是1024. 选择锚点数为300，默认模式个数为3.

设置了一些学习点集作为默认锚点。编码层和解码层的个数和DETR一样为6. 和Deformable DETR一样使用了Focal损失作为分类损失。

### 主要结果

算法和 DETR ，SMCA 和Deformable DETR等各种模型做了比较，主要是结论有，

使用多层特征通常会有更好的性能但是会消耗更多资源；

带DC5特征的AnchorDETR可以比可变DETR和SCMA使用多层特征表现更好。

比DETR快十倍的收敛速度。

这里同时展示了各个检测其 anchor-free ，NMS-free 和RAM-free的特性。anchor-free和NMS-free指检测器不需要人为设定的anchor box和非极大值抑制。RAM-free表示检测器不包括随机内存访问，可以在实践中对硬件更友好。

两阶段检测器通常都没有RAM-free机制，因为（ROI）兴趣区域是对硬件随机的，而且RoI-Align操作

ROI-Pooling 池化包含了随机内存访问。

可变DETR 和两阶段检测器类似，它采样点位置是随机访问硬件的所以也不是RAM-free。

相反，AnchorDETR 继承了DETR 的anchor-free,NMS-free 和RAM-free等特性，并提升了性能和减少了训练代数。

#### 论文最后也通过消融实验讨论了各个组件的有效性

## 结论

主要贡献是在于收敛速度和内存使用，使得更有利于工业落地。

在继承了DETR的没有手动设计的 anchor NMS，和RAM-free的特性下，并设计了更好解释的基于Anchor的查询设计 ，并通过结合多个模式的anchor点解决了 一个区域多个物体的难题；

设计了行-列分离的注意力机制，约简了内存消耗并获得比标准DETR的注意力类似或更好的性能，这个模型是端到端的检测器，可以比DETR训练少十倍的代数。


