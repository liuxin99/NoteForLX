### Spelling Error Correction with Soft-Masked BERT
作者：Shaohua Zhang etc.

组织：ByteDance AI Lab，School of Computer Science and Technology, Fudan University
#### 主要工作
提出了一个基于针对CSC任务的Soft-Masked BERT模型。模型如下图：
![[attachments/Pasted image 20220808142933.png]]
包括一个错误检测模块和错误修改模块。
##### 错误检测模块
由Bi-GRU组成，输入Embeddings同bert输入一样，为word embedding, position embedding, 和 segment embedding 的和。输出为二分类任务，输出当前字符是否是应该修改的位置的概率。
##### Soft Masking
给Bert模型输入的Embeddings为$e^′_i = p_i · e_{mask} + (1 − p_i) · e_i$,  $p^i$为错误检测模块输出的概率，$e_{mask}$为MASK标记的embedding, $e_i$为当前字符的embedding，通过这种soft masking的方式，让bert能够更好的修改应该被修改的错误。
##### 错误修改模块
BERT模型，输入为Soft Masking之后的embedding, 最后输出层和原始输入embedding 做了残差连接之后过softmax进行预测。
两个模块联合训练
![[attachments/Pasted image 20220808145035.png]]

##### 数据
Test and development:  Sighan(1100)，NEWS title (作者通过从今日头条，中国日报APP上获取的新闻标题，请人工标注, 15,730 texts)。

Train: NEWS title, 通过音近混淆集人工造错（语料的15%为错误字符，其中80%为音近替换，20为随机替换），数据量5 million。

##### 实验结果
![[attachments/Pasted image 20220808145206.png]]
![[attachments/Pasted image 20220808145236.png]]
![[attachments/Pasted image 20220808145259.png]]
![[attachments/Pasted image 20220808145308.png]]
#### 思考
- 论文里面说BERT本身可能不适合做错误检测，所以用soft Masking的方式加入了错误位置的信息，而且soft Masking的效果要比hard Masking的效果略好，soft Masking使BERT能更好的利用全局上下文信息
- 从数据量的实验结果来看，更大规模的数据能使模型有更大的提升
- 这种错误检测-错误修改的pipeline，这里是使用GRU来做错误检测，是否可以用ELACTRA来适配CSC任务，做错误检测呢，同时使用两个预训练模型的话，是不是可以考虑模型蒸馏进行压缩