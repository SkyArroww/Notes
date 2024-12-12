[[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=1&selection=0,0,1,28&color=yellow|跳转到论文]]

## 0. 摘要 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=1&selection=80,0,80,8&color=yellow|Abstract]]
在现实世界的应用中，从单张图像中创建高质量的人体着装三维模型至关重要。尽管最近取得了一些进展，但从任意有效图像中准确重建**姿势复杂或穿着宽松**的人体，以及**预测未见区域的纹理**，仍然是一项重大挑战。
	
以往方法的一个**主要局限**是在**从二维过渡到三维**以及**纹理预测**方面缺乏足够的事先指导。为此，我们推出了 **SIFU（用于真实世界可用衣着人体重建的侧视条件隐函数）**，这是一种**将侧视解耦Transformer与三维一致纹理细化管线**相结合的新方法。

* SIFU 在Transformer中采用了**交叉注意机制(Cross-attention Mechanism)**，**使用 SMPL-X 法线作为查询**，在将二维特征映射到三维的过程中有效地解耦侧视特征。这种方法不仅能提高三维模型的精度，还能提高模型的鲁棒性，尤其是在 SMPL-X 估计不完美的情况下。
* 我们的纹理细化流程利用**基于文本到图像扩散**的先验技术(Text-to-image diffusion-based prior)，为不可见视图生成逼真一致的纹理。通过大量实验，SIFU 在几何和纹理重建方面都超越了 SOTA 方法，在复杂场景中展现了更强的鲁棒性，并实现了前所未有的**倒角(Chamfer)和 P2S 测量**。我们的方法可扩展到三维打印和场景构建等实际应用中，证明了它在现实世界场景中的广泛实用性

## 1. 引言 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=1&selection=136,3,136,15&color=yellow|Introduction]]
现有方法无法应对**复杂姿势、宽松的衣服和未见区域的纹理** 
-> 需要改进模型，生成**多视角**下拟真的Mesh与Texture

### 1.1 现有的两个主要挑战：
* **1. 从2D特征到3D特征的转换过程没有足够引导**
    2D图片转3D物体主要包含以下过程：
     i. 从2D图片提取特征
     ii. 将2D特征转换至3D
     iii. 用3D特征进行重建
    目前的方法通常会在第一步和最后一步添加几何先验（如 SMPL-X ），重点关注**法线图预测** 、**SMPL 引导的 SDF** 或**体积特征**等技术。 虽然使用先验改进从二维图像特征到三维图像特征的过渡至关重要，但这方面的探索仍然不足。 目前，这种过渡通常是通过**将特征投影到三维点**或**采用固定的可学习嵌入生成三维特征**来实现的。 然而，这些方法并不能充分利用先验的潜力来提高三维重建的准确性。
* **2. 对纹理缺乏引导**
    虽然当前的方法尝试预测顶点颜色，但它们很难准确预测未见视图的纹理，尤其是在训练数据有限的情况下。 这一局限性凸显了三维人体重建中对额外纹理先验的需求。

### 1.2 提出了两个改进方案：
* **1. 通过额外引导增强从2D特征到3D的过程**
    为了更有效地将 SMPL-X 等先验指导与图像特征相结合，我们利用了Transformer的交叉注意机制。 这种方法旨在优化几何图形和图像数据的融合，从而可能生成更精确、更逼真的三维人体模型。
* **2. 将Diffusion Model作为先验来增强纹理预测，尤其是针对不可见区域**
	此外，保持不同角度的三维一致性并与输入图像的风格相匹配也是创建逼真纹理的关键。

在本文中，我们介绍了 SIFU（用于真实世界可用衣着人体重建的侧视条件隐函数），这是一种**采用侧视条件隐函数和三维一致纹理细化管线**的新方法，用于精确几何和逼真纹理重建。 我们的方法利用 SMPL-X 的法线作为查询，与图像特征形成交叉关注机制。 在将二维特征映射到三维的过程中，这种方法有效地将侧视特征分离出来，从而提高了重建的准确性和鲁棒性。 此外，我们的纹理细化还采用了文本到图像的扩散模型，确保不同视角下的扩散特征保持一致，从而获得细节丰富、风格一致的纹理。

## 2. 相关工作 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=3&selection=70,3,70,15&color=yellow|Related Work]]
### 2.1 基于隐式函数的重建 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=3&selection=72,0,72,38&color=yellow|Implicit-function-based Reconstruction]]
**占位场**(Occupancy Field)和**签名距离场**(Signed Distance Field)等隐式表示法在拓扑结构上具有灵活性，可以有效地描绘各种场景下的三维人体着装，包括宽松的服装和复杂的姿势。

一系列研究的重点是直接从单个输入图像回归隐式表面，并简化流程。 还有一些研究则在重建之前加入三维人体，以加强二维特征提取和三维特征重建的过程。 其中，GTA 利用具有固定可学习嵌入的变换器将图像特征转换为三维三平面特征。 至于纹理重建，PIFu 、ARCH 、PaMIR  和 GTA  等方法可从单张图像中推导出完整的纹理。 PHORHUM  和 S3F  等技术通过分离反照率和全局光照度更进一步。 

不过，这些方法缺乏来自其他视图的信息或先验知识（如扩散模型），因此纹理效果并不理想。 HumanSGD 采用**扩散模型**对网格进行内绘，但由于网格重建不准确，其性能也随之下降。 TeCH 使用基于扩散模型对未见区域进行可视化，取得了逼真的效果。 然而，它的局限性包括需要耗费大量时间进行对象优化，以及依赖于精确的 SMPL-X 模型。

### 2.2 基于显式形状的重建 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=3&selection=104,0,104,35&color=yellow|Explicit-shape-based Reconstruction]]
从单张 RGB 图像中恢复人体网格是一项复杂的挑战，受到了广泛关注。 许多方法都采用**参数化人体模型** 来估计三维人体的形状和姿势，并尽量减少衣物影响。 为了将服装纳入三维模型，相关方法通常会应用**三维服装偏移**，或在基本体形上使用可调整的**服装模板**。 此外，还探索了**深度图、法线图和点云**等非参数形式来创建穿衣人体的表现形式。 尽管取得了这些进步，显形方法仍会受到拓扑约束的限制，这在处理现实世界中各种复杂的服装款式（如连衣裙和短裙）时变得非常明显。

### 2.3 基于NeRF的重建 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=3&selection=125,0,125,25&color=yellow|NeRF-based Reconstruction]]
随着**神经辐射场**（NeRF）的兴起，一些方法利用视频或多视角图像来优化神经辐射场，以捕捉人体形态。 SHERF和 ELICIT 等最新研究成果旨在**通过单张图像生成人体 NeRF**，其中 SHERF 利用二维图像数据填补空白，而 ELICIT 则采用预先训练的 CLIP 模型来理解上下文。 虽然基于 NeRF 的方法能有效地从不同角度生成高质量图像，但它们通常难以从单张图像生成详细的 3D 网格，而且往往需要大量时间进行优化。

## 3. 方法 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=3&selection=155,3,155,9&color=yellow|Method]]
对于单幅图像，SIFU 首先使用侧视条件隐函数重建三维网格和粗纹理。 随后，它采用三维一致纹理细化流程来增强纹理，确保高质量和三维一致性。

### 3.1 前置知识 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=4&selection=104,5,104,16&color=yellow|Preliminary]]
#### 3.1.1 隐函数 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=4&selection=106,0,106,17&color=yellow|Implicit Function]]
隐函数是利用神经网络对复杂几何图形和颜色进行建模的强大工具。 我们使用隐函数预测**占位场**(Occupancy Field)以表示三维穿衣人类。 

具体来说，我们的隐函数 $\mathcal{IF}$ 将输入点 $\mathcal{x}$ 映射到一个标量值，该标量值代表空间场，包括**占位场和颜色场**。 我们重建的人体表面可表示为 $\mathcal{S}_{\mathcal{I F}}$：
$$
\mathcal{S}_{\mathcal{I F}}=\left\{\boldsymbol{x} \in \mathbb{R}^3 \mid \mathcal{I F}(\boldsymbol{x})=(\boldsymbol{o}, \boldsymbol{c})\right\}
$$其中占位场 $\boldsymbol{o} = 0.5$，颜色 $\boldsymbol{c} \in \mathbb{R}^3$。

#### 3.2.2 SMPL 与 SMPL-X [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=4&selection=169,0,169,15&color=yellow|SMPL and SMPL-X]]
Skinned Multi-Person Linear（SMPL）模型是一种用于人体表示的参数模型。 它使用形状参数 $\beta\in\mathbb{R}^3$ 和姿势参数 $\theta \in \mathbb{R}^{3 \times 24}$ 来定义人体网格 $\mathcal{M}$：
$$
\mathcal{M}(\boldsymbol{\beta}, \boldsymbol{\theta}): \boldsymbol{\beta} \times \boldsymbol{\theta} \mapsto \mathbb{R}^{3 \times 6890}
$$这里，**$\boldsymbol{\beta}$ 控制身体大小，而 $\boldsymbol{\theta}$ 影响关节位置和方向**。 SMPL-X 模型以 SMPL 为基础，增加了手部和面部特征，增强了面部表情、手指动作和详细的身体姿势。

#### 3.2.3 扩散模型 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=4&selection=238,0,238,16&color=yellow|Diffusion Models]]
扩散过程，尤其是以扩散概率模型（DPM）为代表的扩散过程，在图像生成中起着举足轻重的作用，并在人类/头像生成中显示出其能力。 这些模型旨在通过渐进式去噪过程逼近数据分布 $q$。 该模型从高斯噪声图像 $\boldsymbol{x}_T \sim \mathcal{N}(0, I)$开始，对其进行去噪处理，直到获得来自目标分布 $q$ 的干净图像 $x_0$。 DPM 还可以通过额外的指导信号（如文本调节）来学习条件分布。

### 3.2 侧视条件隐函数 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=4&selection=276,5,276,44&color=yellow|Side-view Conditioned Implicit Function]]
我们模型中的侧视条件隐函数包括两个关键部分：**侧视解耦Transformer**和**混合先验融合策略**。 
Transformer最初使用不同侧视图的渲染 SMPL-X 图像作为查询，与编码输入图像进行交叉关注。 这一过程有效地解耦了以侧视图为条件的特征。 然后，混合先验融合策略在每个查询点整合这些特征，随后将其输入多层感知器（MLP），用于预测占用率和颜色。 
#### 3.2.1 侧视解耦Transformer [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=10,0,10,32&color=yellow|Side-view Decoupling Transformer]]
我们的方法从侧视图（如背面或左侧）和可见正视图之间的共同特征（如材料和颜色）中汲取灵感。 尽管视角不同，但这些相似的特征却至关重要。 因此，我们以 SMPL-X 模型为指导，旨在有效地将侧视图特征从正视图中分离出来。

这一过程**从基于 ViT 的全局编码器**开始，它将输入图像 $I$ 编码为潜在特征 $h$，捕捉图像的全局相关特征。 为了对这些特征进行解码，我们使用了两个解码器：一个是**与 $h$ 对齐的前视图解码器**，另一个是**侧视图解码器**。 
* 前视图解码器利用ViT中的多头自注意来处理前视特征，即 $F_{front} \in \mathbb{R}^{H \times W \times C}$ 。
* 为了解耦侧视特征，我们将 SMPL-X 的侧视法线图像 $N_i$ 作为引导 $i\in \{left, back, right\}$。 侧视法线图像 $N_i$ 被转换为嵌入数据 $z_i$，然后作为查询进行交叉关注操作，潜在特征 $h$ 既是键也是值：
	$$
	CrossAttn(z_i, h) = SM(\frac{(W^Qz_i)(W^kh)^T}{\sqrt{d}})(W^Vh)
	$$
  其中，$SM$ 表示 $SoftMax$ 操作，$W^Q$, $W^K$ 和 $W^V$ 为可学习参数，$d$ 为缩放系数。 按照最初的Transformer架构，我们的模型在每个子层之后都集成了残差连接和层归一化。 整个侧解码器包含多个相同的层，我们部署了三个这样的解码器，以生成特征图 $F_i \in \mathbb{R}^{H\times W\times C}$ 其中 $i \in \{left,back,right\}$。

#### 3.2.2 混合先验融合策略 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=205,0,205,28&color=yellow|Hybrid Prior Fusion Strategy]]
在我们的管线中，我们采用了[Global-correlated 3d-decoupling transformer for clothed avatar reconstruction]  中的混合先验融合策略(Hybrid Prior Fusion Strategy)，利用空间定位和人体先验知识有效地合并查询点的特征。

我们将特征图 $F_j（j\in{front,left,right,back}）$分成两组。 对于空间查询组，我们将查询点投影到特征图上，以获得像素对齐的特征 $F^S_j$。 然后，我们使用平均和串联的混合方法将来自所有平面的这些特征组合起来：
$$
F^S(\boldsymbol{x})=F_f^S(\boldsymbol{x}) \oplus \operatorname{avg}\left(F_l^S(\boldsymbol{x}), F_b^S(\boldsymbol{x}), F_r^S(\boldsymbol{x})\right)
$$
其中，$f,l,b,r$ 分别表示前、左、后、右。 对于另一组，与空间查询类似，我们将 SMPL-X 网格顶点投影到四个特征图上，得到特征 $F^S(v), v\in \mathcal{M}$，其中 $\mathcal{M}$ 是 SMPL-X 网格。 对于每个查询点 $x$，我们会找到其最近的三角形面$t_x = [v_0,~v_1,~v_2] \in \mathbb{R}^{3\times 3}$，并采用重心插值法整合 x 的特征，记为 $F^P (x)$:
$$
F^P(x)=uF^S(v_0)+vF^S(v_1)+wF^S(v_2)
$$
其中 $[u, v, w]$ 表示查询点 $x$ 投影到三角形 $t_x$ 上的偏心坐标。 我们将这两个查询特征合并为最终的点特征。 此外，我们还将查询点与 SMPL-X 网格体 $\mathcal{SDF}(x)$ 和像素对齐法线特征 $F^N (x)$ 之间的带符号距离作为多层感知器（MLP）的输入，用于预测占用率和颜色：
$$
(\boldsymbol{o,c})=MLP(F^S(x),~F^P(x),~\mathcal{SDF}(x),~F^N(x)))
$$
#### 3.2.3 训练目标 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=466,0,466,19&color=yellow|Training Objectives]]
我们将两组点作为训练数据，分别称为 $G_o$ 和 $G_c$ 。 $G_c$ 是沿真实原始网格面的法线进行均匀采样，并略加扰动，而 $G_o$ 则采用与 [PIFu: Pixel-aligned implicit function for high-resolution clothed human digitization.] 相同的策略采样。 对于 $G_o$ 中的点，我们采用以下损失函数：
$$
\mathcal{L_o} = \frac{1}{|G_o|}\sum_{x\in G_o}{BCE(\hat{o}_x - o_x)}
$$
其中，BCE代表二元交叉熵损失函数(Binary Cross-Entropy Loss)，$\hat{o}_x$ 表示模型预测的占用率，而 $o_x$ 是真实原始占用率。 对于 $G_c$ 中的采样点，我们采用以下损失函数：
$$
\mathcal{L_c} = \frac{1}{|G_c|}\sum_{x\in G_c}{|\hat{c}_x - c_x|}
$$
其中，$\hat{c}_x$ 表示预测颜色，$c_x$ 表示相应的原始真实颜色。 总损失是这两个独立损失的总和，旨在实现综合训练目标。
#### 3.2.4 网格体提取 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=595,0,595,15&color=yellow|Mesh Extraction]]
我们首先对空间中的点进行密集采样，然后使用侧视条件隐函数来预测它们的占用值。 然后采用**行进立方体算法**(Marching Cubes)提取网格，并按照[Explicit Clothed humans Optimized via Normal integration]的方法，用 SMPL-X 模型代替手部，以增强视觉效果。 最后，通过隐函数再次处理这些网格点，以进行颜色预测。

### 3.3 三维一致纹理细化 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=605,5,605,37&color=yellow|3D Consistent Texture Refinement]]
在使用我们的隐式函数提取网格时，我们注意到色彩质量比较粗糙，输入中看不到的区域比较模糊，导致看起来不太真实。 为了解决这个问题，我们开发了三维一致纹理细化管线，利用文本到图像的扩散先验来大幅提高纹理质量。

#### 3.3.1 管线 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=5&selection=618,0,618,8&color=yellow|Pipeline]]
对于给定的输入图像及其重建的网格 $M$，我们首先利用视觉到文本模型将图像转换成文字描述 $P$，然后按照 [Dreamgaussian: Generative gaussian splatting for efficient 3d content creation] 中的方法将网格颜色反投影到 UV 纹理贴图 $T$ 上。 为了将不可见的网格区域可视化，在网格 $M$ 上采用可微分渲染 $\mathcal{I}$，生成这些不可见视图的图像：
$$
I = \mathcal{I}(T,~M,~k)
$$
其中，$k = \{~k^1, ...,k^n ~\}$ 代表像机视图，$I = \{~I^1,...,I^n~\}$ 是相应的渲染图像。

随后，预训练和固定的文本到图像扩散模型$\boldsymbol{\epsilon_\theta}$  以 $P$ 为条件，将模糊图像 $I$ 精炼为增强图像 $J$。 为确保精炼图像之间的一致性，对$\boldsymbol{\epsilon_\theta}$ 采用**一致性编辑技术** $\mathcal{H}$，保留 $I$ 的原始语义布局：
$$
{J} = \mathcal{H}(\epsilon_\theta,~P,~I) = \mathcal{H}(\epsilon_\theta, P, \mathcal{I}(T, M, k))
$$
其中，${J} = \{J^1, ..., J^n\}$ 对应$I$的增强图像。得到 $J$ 后，计算每个 $J^i$ 和 $I^i$ 之间的像素平均平方误差（MSE）损失，以优化纹理图 $T$。 

其他损失包括感知损失 $\mathcal{L}_{vgg}$ [38] 和倒角距离损失(Chamfer Distance Loss) $\mathcal{L}_{CD}$ ，旨在确保 $I$ 和输入图像之间的风格相似性。 我们还计算了输入视图与输入图像的 MSE 损失 $\mathcal{L}^f_{MSE}$ 。 这些综合损失可共同优化 $T$，从而提高整体纹理质量：
$$
\min_{T}~~\lambda_1\mathcal{L}_{MSE} + \lambda_2\mathcal{L}_{vgg}+\lambda_3\mathcal{L}_{CD}+\lambda_4\mathcal{L}^f_{MSE}
$$
其中，$\lambda_{1}, \lambda_{2}, \lambda_{3}, \lambda_{4}$ 是每项损失的权重。

#### 3.3.2 一致性编辑 [[SIFU - Side-view Conditioned Implicit Function for Real-world Usable Clothed Human Reconstruction.pdf#page=6&selection=469,0,469,18&color=yellow|Consistent Editing]]
为了在不同视图之间实现一致的图像编辑，我们采用了受 [Tokenflow: Consistent diffusion features for consistent video editing] 启发的方法。 这就需要在不同渲染视图的扩散特征之间实现一致性。 我们对输入图像 $I$ 进行 DDIM 反转，提取所有图层的扩散标记。 选择一组关键视图进行联合编辑，以确保生成的特征具有统一的外观。 然后使用最近邻方法将这些特征传播到所有视图，以保持它们之间的一致性。 有关更详细的程序见解和具体机制，请参阅 SupMat。