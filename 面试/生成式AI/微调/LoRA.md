# 1. LoRA模型原理
![[1.png]]
冻结原始预训练参数，在此基础上加上微调权重进行新一轮训练。

![[2.png]]
权重矩阵进行微调时，仅需微调前两列、前两行。

![[3.png]]
LoRA中集成了两个转换器，实现满秩和低秩相互转换。之后仅需在这两个矩阵上微调。

->优点： 微调文件较小（<150MB）

# 2. LoRA训练工具
Kohya
[SD-Trainer]([Akegarasu/lora-scripts: SD-Trainer. LoRA & Dreambooth training scripts & GUI use kohya-ss's trainer, for diffusion model.](https://github.com/Akegarasu/lora-scripts)) 等

# 3. 训练基本流程
1. 准备数据集
	多样性，多角度
	e.g. 单一人物：20~30张
2. 预处理
	裁剪：尽量保留人物整体
	打标：Deepbooru/BLIP不够准确，尽量使用WD 1.4 Tagger
3. 设置训练参数
	底模：建议使用SD官方原始模型
	![[4.png]]

*Tip*: 对于偏二次元化的模型，推荐使用 Novel AI(NAI)模型作为底模，它是在SD1.5基础上，根据Danbooru插画王章的Tag微调出的商业大模型。
![[5.png]]

目前来说，较为新的底模有：
* 二次元风格： Anything, AnyLoRA
* 真人风格：ChillOut Mix, Realistic Vision, MahicMix Realistic
![[6.png]]

一种较推荐的文件夹结构：
— ProjectName
	— Image
		— {RepeatTimes1} _ Concept1     （二次元 5~10， 写实 10~30）
		— {RepeatTimes2} _ Concept2
	— Logs
	— Models
# 4. 训练参数剖析

* LoRA类型：
![[7.png]]
* 训练步长：
	epoch = imagesNum * repeatTimes
	在人物训练中，1200~1500步较好
	
* 学习率：
	主干网络一般设定为1e-4即可。当BatchSize变化n倍时，一种方法时将学习率变化$\sqrt{n}$倍， 当$b\leq 10$时，也可以不调。
	TextEncoder的学习率和噪声预测其学习率相关， 一般前者为后者$0.1 - 0.5$倍
	
* 优化器：
	一般AdamW8bit即可，Lion也较常用，不过相应lr要小十倍，在大的BatchSize下表现良好。新手推荐Prodigy，无参数自适应，lr改为1即可。
	调度器影响不是很大，选余弦/余弦退火(3~5重启次数)都行。
	
* 模型维度：
	Rank：二次元 -> 8/16/32   写实 -> 32/64/128   尽量从低秩开始
	Alpha：影响权重，一般设置成Rank的$\frac{1}{2}$或者1

二次元风格建议参数如下，如写实则可翻一倍食用
![[8.png]]
	
* 性能相关参数：
	混合精度/保存精度：fp32/fp16/bf16   推荐fp16
	缓存潜变量(到磁盘)：若不开启，过程中每次都会转换
	CrossAttention： N卡 -> xformers
	内存高效注意力：压缩显存占用，防爆炸，不过降速非常多
![[9.png]]

=> **“多试试”**

*训练监控*
* 命令行：每秒迭代数，监控avgLoss(Loss没了就是G了)
* TensorBoard
* Sample
* WebUI XYZ图表
# 5. 数据集处理
* 裁剪:
	对于比例瘦长的图片，可以将其padding成正方形。
	桶分辨率策略：对所有图片按照长宽比进行分批训练

* 打标：
	希望学习的关键词不应该出现在标签里。
	例如对于人物而言，若是标签里有人物特征(发色、瞳色等)，则需要清洗掉这些标签。
	e.g. Dataset Tag Editor
	
	清洗流程：整体审核 -> 批量调整 -> 单张修改
	1. 整体审核：删掉明确**不符合角色特征的错词**(男标为女等) 和 **训练对象本体识别特征密切相关的词**（发饰、发色、服装元素等）
	2. 批量调整：对一些**有共通明确特质的图片加上标签**(比如对夜景图加上夜晚)
	3. 单张审核：画面细节(粒子、花、树等)

# 6. LoRA实践应用思路