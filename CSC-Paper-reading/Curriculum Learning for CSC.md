## Contextual Similarity is More Valuable than Character Similarity: Curriculum Learning for Chinese Spell Checking
作者：Ding Zhang，Yinghui Li etc.

组织：Tsinghua Shenzhen International Graduate School, Tsinghua University，Tencent Cloud Xiaowei

发表时间：2020-07

#### 主要工作
- 提出字符的上下文相似信息比字符本身的相似信息对模型检测和修改拼写错误更有价值，并根据上下文向量对训练数据做难度评估
	- 使用上下文编码器如BERT提取字符的上下文信息，通过$cos$相似度计算错误文本与正确文本的距离，获得一个难度分数，根据难度分数对训练数据进行难度排序
	- ![[attachments/Pasted image 20220810114217.png]]
	- where $d_i$ is the difficulty score of i-th sample, $W_i$ are the positions with error. $E_i$是字符Embedding
- 提出一个针对CSC任务的课程学习框架，使用上一步经过难度排序后的数据集，对数据集进行划分，然后按课程学习的方式对模型进行训练，这种训练方式是模型无关的，具有普适性，具体流程如下：
	- Curriculum Arrangement：首先把排序后的数据由易到难划分为k个子集：${S_1, S_2, ..., S_k}$ ，之后把训练步骤分为K+1步
		- 在前K步，每一个$i$ 步，对划分好的子集再次划分，从每一个难度等级的子集中取一部分进入第$i$个子集，即$C_i = S_{1,i}, S_{2,i}, ..., S_{k,i}$ 为第$i$步的训练集
		- 在第K+1步，模型使用整个数据集S做训练集，以拟合整个数据集的分布
#### 实验
数据：sighan13-15，伪数据wang
实验设置：作者使用这套课程学习框架，在几个典型工作上实验了一遍，包括SpellGCN，PLOME，MLM-phonetics，实验结果表明，这套框架在几个工作上均能提升模型的表现。
![[attachments/Pasted image 20220810144750.png]]
消融实验结果如下：
![[attachments/Pasted image 20220810144823.png]]
消融实验进一步证明了CL框架的有效性。
CL框架K值的选择以及对实验数据各难度等级子集中的错误数量统计：
![[attachments/Pasted image 20220810145320.png]]
更短的句子更简单，更长的句子包含的错误数量更多。

#### 思考
- 字符的上下文向量的利用这个点，在文章里面只是使用上下文向量的距离对训练数据进行了难度分级，没太明白这就算是利用了字符的上下文的相似度了吗
- 课程学习这一套，感觉很有借鉴意义，在以后的实验中或许可以参考融入这种方法