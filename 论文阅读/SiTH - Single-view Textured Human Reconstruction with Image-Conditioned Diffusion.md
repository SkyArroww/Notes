[[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=1&selection=0,0,1,32&color=yellow|跳转到论文]]

## 0. 摘要 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=1&selection=46,0,46,8&color=yellow|Abstract]]
三维人体重建的一个长期目标是从单视角图像中创建栩栩如生、细节完整的三维人体。 **主要的挑战在于推断图像中不可见区域的未知身体形状、外观和服装细节**。 为了解决这个问题，我们提出了 SiTH，这是一种将图像条件扩散模型独特地集成到三维网格重建工作流程中的新型管线。

我们方法的核心在于将具有挑战性的单视角重建问题分解为**生成幻觉**和**重建**两个子问题。 对于前者，我们采用强大的生成扩散模型，根据输入图像幻化出未知的后视外观。 对于后者，我们利用带皮肤的人体网格作为指导，从输入和后视图像中恢复全身纹理网格。 

SiTH 只需要 500 张三维人体扫描图像即可完成训练，同时还能保持其通用性和对不同图像的鲁棒性。 在两个三维人体基准（包括我们新创建的基准）上进行的广泛评估表明，我们的方法在三维纹理人体重建方面具有卓越的准确性和感知质量。

## 1. 引言 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=1&selection=69,3,69,15&color=yellow|Introduction]]
从单视角图像重建全纹理人体网格是一个难题，主要有两大挑战。首先，在**未观察到的区域生成纹理所需的外观信息缺失**。 其次，**网格重建所需的三维信息在二维图像中变得模糊不清**，如深度、表面和身体姿态。 之前的研究 [4, 60, 79] 尝试以数据驱动的方式解决这些难题，重点是利用图像网格对(image-mesh pairs)训练神经网络。 然而，由于三维人体训练数据有限，这些方法在处理具有未见外观或姿势的图像时举步维艰。 最近的研究 [61, 73, 79] 引入了额外的三维推理模块，以增强对未知姿势的鲁棒性。 然而，从未曾见过的外观生成逼真的全身纹理仍然是一个尚未解决的问题。

为了应对上述挑战，我们提出了 SiTH 这一新颖的管线，它集成了图像条件扩散模型(image-conditioned diffusion model)，可从单眼图像重建栩栩如生的三维纹理人体。 我们方法的核心是将具有挑战性的单视角问题分解为两个子问题：生成**幻觉后视图**和**网格体重建**。 这种分解使我们能够利用预训练扩散模型的生成能力来指导全身网格和纹理重建。 给定前视图像后，第一阶段是利用图像条件扩散技术幻化出感知一致的后视图像。 第二阶段利用前视和后视图像作为指导，重建全身网格和纹理。

更具体地说，我们**利用预训练扩散模型（如Stable Diffusion [57]）的生成能力来推断未观察到的后视外观**，从而进行全身三维重建。 **确保三维网格逼真度的主要挑战在于生成的图像要能描绘出与输入图像在空间上对齐的身体形状和感知上一致的外观**。 虽然扩散模型在文本条件限制方面表现出了令人印象深刻的生成能力，但在使用正面图像作为图像条件生成理想的后视图像方面却受到了限制。 为了克服这一问题，我们调整了网络结构，使其能够对正面图像进行调节，并根据 ControlNet [77] 引入了额外的可训练组件，以提供姿势和遮罩控制。 为了使该模型完全适应我们的任务，同时保留其原有的生成能力，我们利用三维人体扫描渲染的多视角图像对扩散模型进行了仔细的微调。 作为对生成模型的补充，我们开发了一个网格重建模块，用于从前后视图像中恢复全身纹理网格。 我们沿用了之前通过法线[61]和带皮肤人体[73, 79]引导来处理三维模糊性的工作方法。 值得注意的是，这两个子问题的模型都是使用相同的公共 THuman2.0 [75] 数据集进行训练的，该数据集仅包含 500 个扫描数据。

为了推进单视角人体重建的研究，我们基于高质量的 CustomHumans [22] 数据集创建了一个新的基准，并针对最先进的方法进行了全面评估。 与现有的端到端方法[4, 60, 79]相比，我们的两阶段管线可以恢复全身纹理网格，包括后视细节，并对未见图像表现出鲁棒性。 与耗时的基于扩散的优化方法[24, 35, 41]相比，我们的管道能在两分钟内高效生成高质量的纹理网格。 此外，我们还探索了结合文本指导扩散模型的应用，展示了 SiTH 在三维人体创作中的多功能性。 我们的贡献总结如下：
- 我们介绍的 SiTH 是一种单视角人体重建管道，能够在两分钟内生成高质量的全纹理三维人体网格。 
- 通过分解单视角重建任务，SiTH 可以利用公开的三维人体扫描图像进行高效训练，并且对未知图像具有更强的鲁棒性。 
- 我们为评估纹理人体重建建立了一个新的基准，该基准具有更多样化的主题。

## 2. 相关工作 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=2&selection=79,3,79,15&color=yellow|Related Work]]
### 2.1 单视角人体网格体重建 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=2&selection=81,0,81,37&color=yellow|Single-view human mesh reconstruction]]
从单目输入重建三维人体的研究[14, 17, 26, 28, 29, 62, 67, 71]越来越受欢迎。 在这种情况下，我们重点研究从单张图像中恢复三维人体形状、服装和纹理的方法。 作为一项开创性工作，Saito 等人[60] 首次提出了一种数据驱动型方法，该方法采用像素对齐特征和神经场[72]。 其后续作品 PIFuHD [61] 通过高分辨率法线引导进一步改进了这一框架。 后来的方法利用额外的人体先验对这一框架进行了扩展。 例如，PaMIR [79] 和 ICON [73] 利用带皮肤的人体模型 [42, 51] 来指导三维重建。 ARCH [25]、ARCH++ [21] 和 CAR [39]将全局坐标转换为典型坐标，以便进行重塑。 PHOHRUM [4] 和 S3F [12] 进一步分离了阴影和反照率，从而实现了重新照明。 另一项研究则用传统的泊松曲面重构取代了神经表征[33, 34]。 ECON [74] 和 2K2K [18] 通过训练法线和深度预测器来生成前后 2.5D 点云。 人体网格是通过将这些点云与人体先验和三维启发式融合而得到的。 然而，这些方法都无法在未观察到的区域生成逼真的全身纹理和几何图形。 我们的管道将生成扩散模型纳入三维人体重建工作流程，从而解决了这一问题。

### 2.2 利用二维扩散模型生成三维模型 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=2&selection=125,0,125,38&color=yellow|3D generation with 2D diffusion models]]
利用大量图像进行训练的扩散模型[52, 56, 57, 59]在根据文本提示创建三维物体方面表现出了前所未有的能力。 之前的大多数工作[11, 19, 40, 47, 53, 68]都遵循优化工作流程，通过神经渲染[66]和分数蒸馏采样（SDS）[53]损失来更新三维表征（如 NeRF [48], SDF 四面体 [63]）。 虽然有些方法[3, 10, 24, 27, 64, 76]将这一工作流程应用于人体，但由于文本条件的模糊性，它们无法生成准确的人体和外观。 最近的研究 [41, 54] 也试图通过更精确的图像处理来扩展这一工作流程。 然而，我们的研究表明，它们在恢复人体服装细节方面存在困难，并且需要较长的优化时间。 与我们的工作最相关的是 Chupa [35]，它也将其管线分解为两个阶段。 需要注意的是，Chupa 是一种基于优化的方法，它依赖于文本且无法建立色彩模型。 我们通过引入图像条件策略和模型来解决这些问题。 最重要的是，我们的方法无需任何优化过程就能迅速重建全纹理人体网格。
### 2.3 扩散模型的适应性调整 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=3&selection=100,0,100,27&color=yellow|Diffusion models adaptation]]
在大规模数据集上训练的基础模型[8, 13, 20, 36]已被证明可以适应各种下游任务。 顺应这一趋势，预训练的扩散模型[52, 56, 57, 59]已成为生成模型的常用骨干。 例如，这些模型可以通过对一小部分图像进行微调来进行定制 [23、37、58]。 ControlNet [77] 引入了额外的可训练插件，以实现身体骨骼等图像调节。 虽然这些策略已被广泛采用，但它们都不能直接满足我们的目标。 与我们的任务更相关的是 DreamPose [31]，它利用 DensePose [16] 图像作为条件来调整输入图像。 然而，由于过度拟合，它无法处理分布外图像。 同样，Zero-1-to-3 [41] 利用多视角图像对扩散模型进行微调，以实现视点控制。 然而，我们的研究表明，视点调节并不足以生成一致的人体。 我们的模型为后视幻觉提供了精确的人体姿势和遮罩条件，从而解决了这一问题。

## 3. 方法 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=3&selection=126,3,126,14&color=yellow|Methodology]]
SiTH 可根据输入的人体图像和**估计的 SMPL-X [51] 参数**生成全身纹理网格。 该网格不仅能捕捉观察到的外观，还能还原未观察到区域的几何和纹理细节，如背部的衣服褶皱。 该管线由两个模块组成。在第一阶段，我们利用图像条件扩散模型的生成能力幻化出未观察到的外观。 第二阶段，我们以输入的前视图像和生成的后视图像为指导，重建全身纹理网格。 值得注意的是，这两个模块都是通过 THuman2.0 [75]中的 500 张纹理人体扫描图像进行有效训练的。

### 3.1 后视图生成 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=3&selection=144,5,144,28&color=yellow|Back-view Hallucination]]
#### 3.1.1 前置知识 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=3&selection=146,0,146,13&color=yellow|Preliminaries]]
给定一个输入的前视图像 $I^F \in \mathbb{R}^{H\times W\times 3}$，我们的目标是推断出一个后视图像 $I^B \in \mathbb{R}^{H\times W\times 3}$，它描述了未观察到的身体外观。 由于对于输入图像有多种可能的解决方案，因此这项任务受到的限制较少。 考虑到这一点，我们利用**隐式扩散模型（LDM）[57]** 来学习给定前视图像的后视图像的条件分布。

首先，由 编码器$\mathcal{E}$ 和 解码器$\mathcal{D}$ 组成的 VAE 自动编码器通过图像重构在二维自然图像语料库上进行预训练，即 $\tilde{I} = \mathcal{D((E(I))}$。 之后，LDM 会学习如何从随机取样的噪声中生成 VAE 潜在分布 $z = \mathcal{E}(I)$ 内的隐式码 $z$。 在对图像进行采样时，通过对高斯噪声进行迭代去噪，可获得隐式码 $\tilde{z}$ 。 最终图像通过解码器重建，即 $\tilde{I}=\mathcal{D(\tilde{z})}$。
#### 3.1.2 图像条件扩散模型 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=3&selection=252,0,252,33&color=yellow|Image-conditioned diffusion model]]
仅仅将 LDM 架构应用于我们的任务是不够的，因为我们的目标是**在输入条件图像的情况下学习后视图像的条件分布**。 为此，我们对图像条件进行了一些调整。 首先，我们利用预训练的 CLIP [55] 图像编码器和 VAE 编码器 $\mathcal{E}$ 从前视图像（即 $I^F$）中提取图像特征。 这些图像特征用于调节 LDM，确保输出图像与输入图像具有一致的外观。 其次，我们遵循 ControlNet [77] 的想法，建议使用 UV 贴图（$I^B_{UV} \in \mathbb{R}^{H\times W \times 3}$）和从后视图得到的剪影遮罩（$I^B_{M} \in \mathbb{R}^{H\times W\times 3}$）作为附加条件。 这些条件提供了额外的信息，确保输出图像具有与条件输入图像相似的体形和姿势。

#### 3.1.3 来自预训练的学习幻觉 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=4&selection=141,0,147,11&color=yellow|Learning hallucination from pretraining]]
训练图像条件 LDM 的另一个挑战是数据。 从头开始训练模型是不可行的，因为需要大量从三维纹理人体扫描中渲染的配对图像。 受大规模预训练学习概念的启发 [13, 20]，我们在预训练扩散 U-Net [57] 的基础上建立了图像条件 LDM。 我们利用微调策略[37, 77]来优化交叉注意层和控制网参数，同时冻结大部分其他参数。 我们的图像条件扩散模型的设计和训练策略能够幻化出与正面输入一致的可信后视图像。

#### 3.1.4 训练与推理 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=4&selection=171,0,171,22&color=yellow|Training and inference]]
为了从三维人体扫描图像中生成成对的训练图像，我们对相机视角进行采样，并使用正投影法渲染三维扫描图像中的 RGBA 图像和 SMPL-X 拟合的 UV 映射。 给定一对由正面相机和相应的背面相机渲染的图像，第一张图像作为条件输入 $I^F$，而另一张则是原始真实图像 $I^B$。 在训练过程中，原始真实隐式码 $z_0 = \mathcal{E}(I^B)$ 在 $t$ 个时间步长内受到扩散过程的扰动，从而产生一个有噪声的隐式码 $z_t$。 **图像条件 LDM 模型$\epsilon_\theta$ 的目的是在给定噪声潜码 $z_t$、时间步长 $t\in[0, 1000]$、条件图像 $I^F$、剪影遮罩 $I^B_M$  和 UV 贴图$I^B_{UV}$ 的情况下预测附加噪声$\epsilon$。** 微调的目标函数可以被表示为：
$$
\min_\theta\mathbb{E}_{z\sim\mathcal{E}(I),~t,~\epsilon\sim\mathcal{N}(0, I)}\parallel\epsilon - \epsilon_\theta(z_t,~t,~I^F,~I^B_{UV},~I^{B}_M)\parallel^2_2
$$
测试时，我们通过现成的姿势预测器 [9] 和分割模型 [36] 获得 $I^B_{UV}$、$I^B_M$。 为了推断后视图像，我们从高斯噪声 $z_T \sim \mathcal{N} (0, I)$开始执行迭代去噪过程，对潜在的 $\tilde{z}_0$ 进行采样。 后视图像可以通过以下方法获得：
$$
\tilde{I}^B = \mathcal{D}(\tilde{z}_0)=\mathcal{D}(f_\theta(z_T,~I^F,~I^B_{UV},~I^B_M))
$$
其中，$f_θ$ 是一个函数，表示我们的图像条件 LDM 的迭代去噪过程。

### 3.2 人体网格体重建 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=4&selection=539,5,539,30&color=yellow|Human Mesh Reconstruction]]
获得后视图像后，我们的目标是以输入图像和后视图像为指导，构建全身人体网格及其纹理。 我们参考文献 [60, 61]，采用数据驱动法对这一任务进行建模。 给定成对的训练数据（即前/后视图像和三维扫描），我们学习一个**数据驱动模型**，将这些图像映射到三维表示（如签名距离场 (SDF)）。 我们将这种映射定义如下：
$$
\begin{aligned}
\mathbf{\Phi}: \mathbb{R}^{H \times W \times 3} \times \mathbb{R}^{H \times W \times 3} \times \mathbb{R}^3 & \rightarrow \mathbb{R} \times \mathbb{R}^3 \\
\left(I^F, I^B, \mathbf{x}\right) & \mapsto d_x, \mathbf{r}_x,
\end{aligned}
$$
其中，$x$ 是查询点的三维坐标，$d_x$、$\mathbf{r}_x$ 表示 $x$ 点的带符号距离和 RGB 颜色值。

#### 3.2.1 本地特征查询 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=5&selection=32,0,32,22&color=yellow|Local feature querying]]
要学习一个对未见图像具有鲁棒性的通用映射函数，重要的是该模型仅以 $x$ 位置的本地图像信息为条件。因此，我们采用了像素对齐特征查询的理念 [60, 61]，并将我们的模型分为两个分支，即颜色和几何。 我们的模型包含一个法线预测器，可将 RGB 图像对 $(I^F, I^B)$ 转换为法线图 $(N^F, N^B)$。 然后，两个图像特征编码器 $G_d,G_r$ 分别从图像和法线图中提取颜色和几何特征图 $(\mathbf{f}_d, \mathbf{f}_r)\in \mathbb{R}^{H^\prime \times W^\prime \times D}$ （为简单起见，我们只描述单幅图像的过程，并省略上标，但前后两幅图像的处理方法相同）。 最后，我们将查询点 $x$ 投影到图像坐标上，以获取局部特征 $(\mathbf{f}_{d,x},\mathbf{f}_{r,x})\in \mathbb{R}^D$：
$$
\begin{aligned}
\mathbf{f}_{d, x} & =\mathcal{B}\left(\mathbf{f}_d, \pi(\mathbf{x})\right) = \mathcal{B}\left(G_d(N), \pi(\mathbf{x})\right)\\
\mathbf{f}_{r, x} & =\mathcal{B}\left(\mathbf{f}_r, \pi(\mathbf{x})\right) = \mathcal{B}\left(G_r(I), \pi(\mathbf{x})\right)
\end{aligned}
$$
其中，$\mathcal{B}$ 是使用双线性插值的局部特征查询操作，$\pi(\cdot)$ 表示正投影。

#### 3.2.2 局部位置嵌入与带肤人体先验 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=5&selection=196,0,196,50&color=yellow|Local positional embedding with skinned body prior]]
网格重建的一个主要困难是三维模糊性，即模型必须推断出前后图像之间未知的深度信息。 为了解决这个问题，我们沿用了之前的研究成果[22, 73, 79]，利用带皮肤的人体网格[51]来指导重建任务。 该人体网格被视为一个锚点，可提供人体的近似三维形状。

为了利用这种人体先验，我们设计了一个本地位置嵌入函数，将查询点 $\mathbf{x}$ 转换为本地人体网格坐标系。 我们在人体网格上寻找最接近的点 $\mathbf{x}^*_c$ :
$$
\mathbf{x}^*_c = \arg\min_{\mathbf{x}_c}\parallel \mathbf{x - x_c}\parallel_2
$$
其中 $\mathbf{x}_c$ 是带肤人体网格体 $\mathcal{M}$ 上的点。
我们的位置嵌入 $p$ 包含四个元素：$\mathbf{x^*_c}$ 和 $\mathbf{x}$ 之间的带符号距离值 $d_c$、向量 $\mathbf{n}_c = (\mathbf{x-x^*_ c})$、点 $\mathbf{x^*_c}$ 的 UV 坐标 $\mathbf{u}_c\in [0, 1]^2$ 和可见性标签 $v_c\in \{1, -1, 0\}$，表示 $\mathbf{x^*_c}$ 在正面/背面图像中可见还是不可见。 最后，两个独立的 MLP $H_d$、$H_r$ 将位置嵌入 $\mathbf{p} = [d_c, \mathbf{n}_c, \mathbf{u}_c, v_c]$ 和本地纹理/几何特征 ($\mathbf{f}_d, \mathbf{f}_r)$ 作为输入，预测 x 点的最终 SDF 和 RGB 值：
$$
\begin{aligned}
d_x & =H_d\left(\mathbf{f}_{d, x}^F, \mathbf{f}_{d, x}^B, \mathbf{p}\right) \\
\mathbf{r}_x & =H_r\left(\mathbf{f}_{r, x}^F, \mathbf{f}_{r, x}^B, \mathbf{p}\right)
\end{aligned}
$$
#### 3.2.3 训练与推理 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=5&selection=434,0,434,22&color=yellow|Training and inference]]
我们使用之前提到的三维数据集，从三维纹理扫描中生成训练图像对 $(I^F , I^B)$。 对于每个训练扫描，查询点 $\mathbf{x}$ 在三维立方体 $[-1, 1]^3$ 中采样。 对于每个点，我们会计算与扫描表面的原始真实签名距离值 $d$、最接近的纹理 RGB 值 $r$ 和表面法线 $\mathbf{n}$。最后，我们在两个分支的重建损失下联合优化了法线预测器、图像编码器和 MLP：
$$
\begin{aligned}
&\mathcal{L_d}= \parallel d - d_x \parallel_1+\lambda_n(1-\mathbf{n}\cdot\nabla_\mathbf{x}d_x)\\
&\mathcal{L_r} = \parallel \mathbf{r}-\mathbf{r}_x\parallel_1
\end{aligned}
$$
注意，$\nabla_\mathbf{x}$ 表示计算 $\mathbf{x}$ 点局部法线的数值有限差分，$\lambda_n$ 是超参数。

在推理过程中，我们使用输入图像 $I^F$ 和之前获得的后视图像 $\tilde{I}^B$ 来重建 3D 网格和纹理。 首先，我们将两幅图像与估计的人体网格 $\mathcal{M}$ 对齐，以确保图像特征可以围绕三维锚点正确查询。 我们采用类似于 SMPLify [7] 的策略来优化带有轮廓和二维关节误差的人体网格的比例和偏移量。 最后，在密集体素网格内查询 SDF 和 RGB 值，执行**行进立方体算法** [43]。

## 4. 实验 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=6&selection=307,3,307,14&color=yellow|Experiments]]
### 4.1 实验准备 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=6&selection=309,5,309,23&color=yellow|Experimental Setup]]
#### 4.1.1 数据集 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=6&selection=311,0,311,6&color=yellow|Datase]]
以前的工作依赖于来自 RenderPeople 等商业数据集的训练数据[1]。 虽然这些数据集提供了高质量的纹理网格体，但由于可访问性有限，它们也限制了可重复性。 为了进行公平比较，我们效仿 ICON [73]，在公共 3D 数据集 THuman2.0 [75] 上训练我们的方法，并使用 CAPE [45] 数据集进行评估。 然而，由于 CAPE 数据集的低分辨率原始真实网格体和图像渲染缺陷，我们在评估中发现了潜在的偏差。 因此，我们进一步创建了一个新的基准，在质量更高的 3D 人类数据集 CustomHumans [22] 上对基线进行评估。 下面，我们将对实验中使用的数据集进行总结：
* **THuman2.0**[75]包含约 500 张人体扫描图像，其中人体穿着 150 种不同的服装，姿态各异。 我们使用这些 3D 扫描作为训练数据。
* **CAPE** [45] 包含 15 个穿着 8 种紧身服装的受试者。 测试集由 ICON 提供，包含 100 个网格。 我们使用 CAPE 进行定量评估。
* **CustomHumans**[22]包含 600 份质量更高的扫描数据，对象为 80 名穿着 120 种不同服装、摆着不同姿势的人。 我们选择了 60 名受试者进行所有定量实验、用户研究和消融研究。
### 4.1.2 评估规程 [[SiTH - Single-view Textured Human Reconstruction with Image-Conditioned Diffusion.pdf#page=6&selection=354,0,354,19&color=yellow|Evaluation protocol]]
我们遵循 OccNet [46] 和 ICON [73] 中的评估协议，对生成的网格计算 3D 指标倒角距离 (CD)、法线一致性 (NC) 和 fScore [65]。 为了评估重建的网格纹理，我们报告了正面和背面纹理渲染的 LPIPS [78]。 在用户研究中，30 位参与者对四种不同方法获得的网格进行了排名。 我们报告了从 1（最佳）到 4（最差）的平均排名。