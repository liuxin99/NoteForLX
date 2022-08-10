## A Hybrid Approach to Automatic Corpus Generation for Chinese Spelling Check
作者：Dingmin Wang

会议：2018ACL

### Abstract
中文拼写检查是一项具有挑战性而又有意义的工作，它不仅是许多自然语言处理(NLP)应用中的预处理工作，而且可以方便人们阅读和理解日常生活中运行的文本。然而，为了在CSC中利用数据驱动的方法，有一个主要的限制，即标注数据集在应用算法和建立模型方面是不够的。在本文中，我们提出了一种新的方法来构建具有自动拼写错误的CSC语料库，这些拼写错误是视觉上或语音上相似的字符，分别对应于基于OCR和ASR的方法。在构建的语料库上，针对三个标准测试集训练和评估不同的模型。实验结果证明了语料库的有效性，从而验证了我们方法的有效性。

### 1. Introduction
拼写检查是检测和纠正运行文本中人为拼写错误的关键任务(Yu和Li, 2014)。这一任务对于搜索引擎(Martins and Silva, 2004;Gao等人，2010)和自动作文评分(Burstein and Chodorow, 1999;Lonsdale and Strong-Krause, 2003)等NLP应用至关重要，因为拼写错误不仅会影响阅读，有时还会完全改变文本片段传递的意思。特别是在汉语语言处理中，拼写错误可能会更严重，因为它们可能会影响到基本的任务，如分词(Xue, 2003;Song and Xia, 2012)和词性标注(Chang et al.， 1993;Jiang等人，2008;太阳,2011),等等。在所有导致拼写错误的原因中，一个主要的原因来自于日常文本，如电子邮件和社交媒体帖子的中文输入法的滥用。表1举例说明了这类中文拼写错误。第一个错误的句子包含一个误用的字符“己(ji2)”，它与对应的正确字符“已(yi3)”的形状相似。在第二个错误句子中，装箱后的拼写错误“她(ta1)”与对应的正确错误“他(ta1)”在语音上相同。
![[attachments/Pasted image 20220713101402.png]]
由于现有的数据集数量有限，许多先进的监督模型很少用于该领域，这阻碍了该领域的发展。目前，一些主流方法仍然侧重于使用无监督方法，即基于语言模型的方法(Chen et al.， 2013;Yu等人，2014;Tseng等人，2015;Lee等人，2016)。因此，CSC技术的发展受到了限制，因此CSC的性能到目前为止并不令人满意(Fung et al.， 2017)。为了提高CSC的性能，最大的挑战是无法获得带有标记拼写错误的大规模语料库，这对于训练和应用监督模型有很高的价值。这种数据缺失的问题主要是由于标注拼写错误是一项昂贵且具有挑战性的任务。
为了解决数据不可用的问题，以实现数据驱动的方法，本文提出了一种自动构建带有标注拼写错误的汉语语料库的新方法。具体而言，鉴于汉语拼写错误主要是由于视觉上和音韵上相似的字符的滥用(Chang, 1995;Liu等，2011;Yu和Li, 2014)，我们提出了基于OCR和ASR的方法来产生上述两种类型的误用字符。注意，与从不正确的句子中检测拼写错误不同，我们提出的方法旨在自动生成带有标注拼写错误的文本，如表1所示。利用基于OCR和ASR的方法，构建了带有标注的视觉拼写错误和语音拼写错误的CSC数据集。
在我们的实验中，定性分析表明，通过OCR或ASR工具包识别错误的汉字对于人类来说并不是微不足道的，而有趣的是，人类在日常写作中很可能会出现这样的拼写错误。在定量比较中，我们将中文拼写检查纳入序列标记问题，并实现双向LSTM (BiLSTM)监督基准模型，以评估CSC在三个标准测试数据集上的性能。实验结果表明，在我们生成的语料库上训练的BiLSTM模型比在标准测试数据集上训练的BiLSTM模型具有更好的性能。为了进一步促进CSC任务，我们通过收集每个字符的所有不正确变体及其对应的正确参考答案来构建混淆集。在纠错任务中验证了混淆集的有效性，表明所构造的混淆集在许多现有的中文拼写检查方案中具有很高的实用价值(Chang, 1995;Wu等，2010;董等人，2016)。

### 2. Automatic Data Generation
汉语中的拼写错误主要是由视觉上或音韵上相似的字符的误用造成的(Chang, 1995;Liu等，2011;Yu和Li, 2014)。视觉上相似字符的错误(即v型错误)是由于视觉上相似的字符对突出。原因是汉语作为一种象形文字，由六万多个汉字组成。它们是由有限数量的自由基和成分构成的。关于音韵相似字的误用(即p字误)，我们注意到，汉字的发音通常是由声母、韵母和声调组成的拼音来确定的。根据Yang等人(2012)的研究，现代汉语数千个汉字只有398个音节。因此，有很多汉字发音相似，这进一步导致了p字错误的突出。在本节的其余部分中，我们将分别在2.1节和2.2节中描述如何生成这两种类型的错误。

### 2.1 OCR-based Generation
由于观察到光学字符识别(OCR)工具很可能会将视觉上相似的字符错误识别(Tong和Evans, 1996)，我们有意用正确的字符模糊图像，并在其上应用OCR工具产生v型拼写错误。
详细地说，我们使用谷歌Tesseract (Smith, 2007)作为OCR工具包，生成过程如图1所示。给定一个句子，作为第一步，我们从句子中随机选择1个(s)个字符作为我们的目标字符，用Tesseract来检测，命名为Ctargets。具体来说，除了中文字符，标点符号和外文字母等其他字符都被排除在外，我们还根据中文维基百科语料库的统计数据对低频的中文字符进行了过滤。其次，我们将ctarget从文本转移到100 100像素的图像，即生成的每个图像都有相同的大小。第三，我们使用高斯模糊(Bradski, 2000)在产生的图像中随机模糊一个区域，目的是引导OCR工具包出错。最后，我们使用谷歌Tesseract对模糊图像进行识别。一旦识别出的结果与原结果不匹配，就会产生一个V-style错误，用来替换句子中的原字符，导致句子出现V-style拼写错误。在上述步骤之后，我们获得了每个句子的拼写错误及其正确的参考答案。
![[attachments/Pasted image 20220713154521.png]]
OCR方法使用的原始文本主要来自报纸文章。这些文章是从官方报纸《人民日报》网站上抓取的，其中报道的文章都经过严格的编辑程序，并假定所有文章都是正确的。我们使用分句结尾的标点将这些文本分成句子，例如句号(。)、问号(?)和感叹号(!)(Chang et al.， 1993)。我们总共得到了50000个句子，每个句子包含8到85个字符，包括标点符号。然后，这些句子被我们前面描述的基于OCR的方法处理，得到一个包含约40000个带有56857个拼写错误的注释语料库。请注意，在我们的实验中，我们发现OCR工具包仍然可以正确地检测产生的图像中的字符，即使我们模糊了图像的一部分。这就解释了为什么生成的标注句子比原句子的大小要小。我们用D-ocr表示数据集，并在表3的D-ocr列中显示其统计数据。
![[attachments/Pasted image 20220713155044.png]]
**Bad Cases and the Solution**
虽然基于OCR的方法顺利实现，但仍有一些案例值得进一步研究。通过对该方法生成的拼写错误进行分析，我们发现，在形状上，OCR工具箱识别的错误字符与其对应的正确字符存在较大差异。例如，对于包含字符“领(ling3)”的模糊图像，Tesseract错误地将其识别为“铈(shi4)”，这在形状上与“领(ling3)”完全不同。因此，这些情况应该被排除在外，因为人类不太可能犯这样的错误。为了解决这一问题，我们提出了一种基于汉字笔画的编辑距离来判断两个字符在视觉上是否相似的新方法。与英文由字母组成的单词类似，汉字也可以分成不同的笔画。为此，我们从在线词典中获取了汉字的笔画。根据经验，给定两个汉字c1和c2，我们设置0.25 (len(c1) + len(c2))为η阈值，其中len(c)表示汉字c的笔画数。如果两个汉字的编辑距离大于η阈值，我们认为它们的形状不相似。为了更好地阐明它，图2中显示了一个坏案例和一个好案例。
![[attachments/Pasted image 20220713155419.png]]
#### 2.2 ASR-based Generation
与OCR工具类似，自动语音识别(ASR)工具也可能将具有相似发音的字符误认为其他字符(Hartley和Reich, 2005)。为了构建p型错误注释语料库，我们借鉴了v型错误注释语料库和OCR工具注释语料库的灵感，采用了如图3所示的管道。然而，考虑到各种语音识别数据集的可用性，我们采用了一种更简单的方法。我们利用了一个公开可用的普通话语音语料库AIShell (Bu等人，2017)，其中包含大约14万句带有话语的句子。我们使用语音识别工具包Kaldi (Povey et al.， 2011)将话语转录成被识别的句子。最后，将识别出的句子与原句子进行比较，判断识别结果是否正确。如果不正确，它们可以作为错误识别的结果，并用于构建p型拼写错误语料库。 
![[attachments/Pasted image 20220713155848.png]]
**Bad Cases and the Solution**
对于生成的P-style错误，我们还确定了一些bad cases，它们可能会引入很多噪声。为了提高生成的语料库的质量，需要去除这些bad cases。表2给出了三种坏情况和一种好情况。我们描述处理它们的解决方案如下。
![[attachments/Pasted image 20220713162455.png]]
首先，我们丢弃所有与案例1相似的错误识别结果，这些结果与对应的参考句子长度不同。第二，案例2中的错字与金句中对应的字读音完全不同。这样的情况不满足我们生成p型错误的要求。为此，我们从一个在线汉语词典中提取汉字的拼音读音。这样就很容易识别出识别错误的字与金句中对应字的读音是否相近或相同。具体来说，在拼音方面，两个声母、韵母相同，声调不同的汉字发音相似，即da2和da1。第三，根据Chen et al.(2011)，平均每篇学生的作文可能会有两个错误，这反映了每个句子平均不会包含两个以上的拼写错误。因此，我们删除那些包含两个以上错误字符的错误识别结果，如案例3所示。经过上述步骤，我们生成了一个p风格拼写错误总数超过7K的语料库。我们将其命名为D-asr，并在表3的D-asr列中显示其统计数据。
![[attachments/Pasted image 20220714155037.png]]
### 3. Evaluation
#### 3.1 Benchmark Data
为了评估生成的语料库的质量，我们在CSC任务上采用了三个来自2013-2015年共享任务的基准数据集(Wu et al.， 2013;Yu等人，2014;Tseng等人，2015)。表4报告了三个标准数据集的统计数据，包括训练和测试部分。考虑到训练数据集的质量对模型在测试数据集上的性能有显著影响。因此，在本文中，我们提出了一个度量$C_{train:test}$，通过计算训练数据集和测试数据集中出现的拼写错误的数量来衡量它们之间的相关度:
$$C_{train:test} = \frac{|E_{train} \cap E_{test}|}{|E_{test}|}$$
其中$E_{train}$和$E_{test}$分别表示训练数据集和测试数据集中的拼写错误集合。集合中的每个元素都是一对，包含一个正确字符和一个拼写错误的字符，例如(撼(han4)，憾(han4))。由表5可以看出，对于三个不同的测试数据集，生成的整个语料库D在$C_{train:test}$值上分别为74.1%、80.6%和84.2%，远高于$Trn_{13}$、$Trn_{14}$和$Trn_{15}$。这种差异可以表示生成的语料库的有效性，有足够的拼写错误。
![[attachments/Pasted image 20220713164335.png]]
#### 3.2 Qualitative Analysis
**Setup**
为了评估生成的语料库中是否存在容易被人发现的错误，我们从其中随机抽取300个句子进行人工评价，其中150个来自D-ocr, 150个来自D-asr。我们邀请了三位以汉语为母语的大学生来阅读和注释这些句子中的错误。然后，我们从sentence-level和error-level两个层面对三名大学生的标注结果进行分析。在句子层面，我们认为只有当句子中的所有错误都被识别出来时，句子才被正确注释。在错误级别，我们计算正确标注的错误数量占总错误数量的百分比。
**Results**
表6给出了300个句子的信息及其标注结果。从表中的平均召回率可以看出，3名学生对D-asr错误的识别率高于D-ocr错误的识别率，这在一定程度上说明p型错误比v型错误更容易被检测出来。此外，我们观察到3名志愿者平均识别不出36.9%左右的错误，这可能表明我们生成的句子中包含了一些挑战性的错误，这些错误很可能是人为造成的。这些错误是人们在书写或打字时可能出现的真实情况，因此对CSC来说很有价值。
![[attachments/Pasted image 20220713165919.png]]
**Case Study**
为了定性地分析为什么有些拼写错误没有被人类发现，我们对一个包含三个学生没有发现的拼写错误的示例句子进行了案例研究。这句话是 “政企部分是一种痼疾(translation: politics and industry parts are a kind of a chronic disease)” ，其中第3个字“部(bu4)”是拼写错误，应该改为“不(bu4)”。在这个例子中，部分(翻译:一部分)和不分(翻译:平等对待)是两个非常常见的中文单词，很容易被认为是正确的。但是，考虑到前面的单词“政企”(翻译为:政治和工业)和后面的词“是一种痼疾”(翻译为:一种慢性疾病)，我们可以看到“部分”不符合当前的上下文，应该纠正为“不分”。这个案例研究证实了我们生成的语料库包含一些拼写错误，比如人们在书写或打字时出现的拼写错误，这进一步证明了我们的方法的质量和有效性。、
#### 3.3 Quantitative Comparison
##### 3.3.1 Chinese Spelling Error Detection
在本节中，我们通过中文拼写检测来评估我们生成的语料库的质量。我们首先探讨了P-style和v-style错误的不同比例对语料质量的影响。然后，我们将生成的语料库与三个共享任务提供的训练数据集的检测性能进行比较。
**Setup**
我们将中文拼写错误检测转换为一个字符序列标记问题，其中正确和错误的字符分别标记为1和0。然后，我们实现了一个监督序列标记模型，即双向LSTM (BiLSTM) (Graves and Schmidhuber, 2005)，作为我们的基线来评估不同语料库的质量。BiLSTM的hidden_size设置为150，其他超参数在由10%随机从训练数据中选择的句子组成的开发集上进行调整。我们最小化模型的交叉熵损失，使用RMSprop (Graves, 2013)作为优化器。
**Results**
BiLSTM在不同比例的D-ocr和D-asr上训练的性能，旨在探讨P-style和V-style拼写错误分布对生成语料库质量的影响。从图4可以看出，当训练数据集的大小固定(=40k)时，P-style和V-style错误的比例不同，得到的F1分数也不同，说明P-style和V-style拼写错误的比例影响生成的语料库的质量。在三个测试数据集上，我们观察到D-ocr和Dasr的比例为4:6时，与其他比例比相比，性能最好。此外，对于这两个特殊比例(0%和100%)，我们可以看到，在语料库规模相同的情况下，D-asr训练的BiLSTM模型比D-ocr训练的BiLSTM模型性能更好，这表明p风格的拼写错误对语料库质量的贡献更大。这个实验结果符合之前的结论(Liu et al.， 2009, 2011)，大部分的拼写错误与发音有关。
![[attachments/Pasted image 20220713171003.png]]
此外，为了更好地说明生成的语料库的质量，我们将其与一些人工标注的训练数据集进行比较(Wu et al.， 2013;Yu等人，2014;Tseng等人，2015)。根据之前对图4实验结果的分析，在接下来的实验中，我们选择P-style和V-style拼写错误比例为4:6来构建训练数据。我们构建了D-10k、D-20k、D30k、D-40k和D-50k五个不同大小的数据集，分别从D-ocr和D-asr中按照4:6的比例提取。然后，我们在这些训练数据集上训练BiLSTM模型，并评估在Tst13、Tst14和Tst15上的错误检测性能。表7显示了三种不同测试数据集上的检测性能。我们有以下几点看法。
![[attachments/Pasted image 20220713171114.png]]
**The size of training dataset is important for the model training.**
对于Tst13, D-10k获得了比Trn13更好的F1分数。一个主要原因可能是Trn13的大小(=350，见表3)，它比测试数据集小得多。在这种情况下，模型无法学习足够的信息，导致无法检测看不到的拼写错误。此外，我们可以看到，随着我们生成的语料库的规模不断扩大，检测性能有了稳定的提高。因此，对于数据驱动的方法，使用具有不同拼写错误的足够实例来训练我们的模型是非常重要的。
**The precision may be compromised if the training dataset contains too many “noisy” spelling errors**
从表7可以看出，虽然随着我们生成的语料规模的增加，整体性能(F1分数)不断提高，但准确率和召回率呈现出不同的变化趋势。可以观察到，随着训练数据集规模的增加，该模型在召回率方面取得了更好的性能。一个可能的原因是，随着训练数据集中包含更多包含不同拼写错误的实例，测试数据集中不可见的拼写错误的数量减少，从而便于模型检测更多的拼写错误。但是，准确率的提高并没有召回率的提高那么明显。其中，在Tst14和Tst15中，D-50k的精度并不比D-40k高。一种可能的解释是，对于包含更多拼写错误实例的更大的训练数据集，它可能会导致模型错误识别一些更正确的字符，从而导致较低的精度。
**Compared with the limited training dataset manually annotated by human, our generated largescale corpus can achieves a better performance**
从表7可以看出，在我们生成的语料库有一定规模的情况下，它训练出的模型比使用相应的共享任务中提供的人工标注数据集训练出的模型检测性能更好。这在一定程度上证明了我们生成的语料库的有效性，从而验证了我们方法的有效性。
##### 3.3.2 Chinese Spelling Error Correction
一旦检测到中文拼写错误，我们就进行纠正测试。
根据之前的研究(Chang, 1995;黄等人，2007;Wu等，2010;Chen等人，2013;Dong et al.， 2016)，我们采用混淆集和语言模型来处理汉语拼写错误纠正。特别是，通过收集每个正确字符的所有不正确变体，我们为所有涉及的正确字符构建了一个混淆集，称为Ours。此外，为了说明其有效性，我们将其与两个公开可用的混淆集Con1和Con2进行比较(Liu et al.， 2009)。其中，Con1由SC(similar Cangjie)、SSST(same-sound-same-tone)和SSDT(samesound-different-tone)组成，而Con2由SC、SSST、SSDT、MSST(similar-sound-same-tone)和MSDT(similar-sound-differenttone)组成。表9显示了三个混淆集的统计信息。
![[attachments/Pasted image 20220713172313.png]]
**Setup**
与Dong et al.(2016)类似，我们采用tri-gram语言模型来计算给定句子的概率。根据序列标记模型的检测结果，从对应的混淆集中选择概率最高的字符作为纠错字符。对于一个包含n个单词的句子，$W = w_1, w_2，···，w_n$，其中$w_i$表示句子中的第i个字符，E是一个包含检测到的错误字符索引的集合，$W (w_i, c)$表示用c替换第i个字符后生成的新句子。纠错过程可表述如下:
$$∀i ∈ E : argmax_{c∈C (w_i )} P (W (w_i, c))$$
其中P (W)是句子W的概率近似于(Jelinek, 1997)所描述的一系列条件概率的乘积，$C(w_i)$是$w_i$的混淆集，选择条件概率最大的一个作为修正字符。
**Results**
表8显示了基于tri-gram语言模型的不同混淆集所获得的运行时间和F1分数。我们可以观察到，我们构造的混淆集获得了比Con1更好的校正性能，但略低于Con2。然而，从表9中，我们可以看到Con2的混淆字符比我们的大得多;对于同样的测试，Con2需要更多的时间来完成任务，而我们总是用最少的时间。这些观察表明，我们构造的混淆集在包含较少的冗余混淆字符(很少用作更正字符)时效率更高。
![[attachments/Pasted image 20220713173005.png]]
**Error Analysis**
我们对两种错误情况，即false positive情况和false negative情况分别进行了错误分析，这两种情况分别影响了CSC的精确率和召回率。
对于false positive情况，我们发现一个常见的问题是，对于一些固定的用法，如习语、短语和诗歌，我们的模型往往给出错误的结果。例如，在“风雨送春归”中，“送”被错误地理解为“不相干的字”。通过检查由提出的方法生成的带注释的语料库，我们观察到，在大多数情况下，迎春是一个更常见的匹配，当送与春在生成的语料库中同时出现时，它被标注为拼写错误。为了提高这种情况的准确性，一种可能的方法是利用一些外部知识，如建立特殊的汉语用法集合。
对于false negative的情况，以“想想健康，你就会知道应该要禁烟了”(翻译为:当你考虑到你的健康时，你会意识到你应该放弃吸烟)为例，其中“禁”应更正为“戒”。然而，由于“戒”和“禁”在视觉上和语音上都不相似，所以所提出的语料库生成方法无法构建这种拼写错误，所以训练的模型无法检测这种拼写错误是可以理解的。此外，在词汇层面，“禁烟”(翻译为禁止吸烟)和“戒烟”(翻译为戒烟)是两个相关的常见中文词汇;它需要纳入更多的上下文，以提高召回性能。与我们对基于字符的语料库生成的研究类似，一个可能的解决方案是构建一个词级注释语料库，以便更好地检测这种拼写错误。
### 4 Related Work
在拼写错误检测和纠正的研究方面，以前的工作主要集中在设计不同的模型来提高CSC的性能(Chang, 1995;黄等人，2007,2008;Chang et al.， 2013)。与之不同的是，这项工作有助于训练数据集的生成，这是重要的资源，可以用来改进许多现有的CSC模型。目前，有限的训练数据集为许多数据驱动方法设置了很高的障碍(Wang et al.， 2013;Wang, Liao, 2015;郑等人，2016)。据我们所知，到目前为止，CSC还没有大数据量通用的带标注的数据集。
一些以前的工作(Liu et al.， 2009;Chang et al.， 2013)指出，在中文文本中，视觉上和音韵上相似的字符是导致错误的主要因素，音韵上相似的拼写错误的数量大约是视觉上相似的拼写错误的两倍。然后在我们的方法中采用了视觉和语音相关的技术。光学字符识别技术是一种从图像中提取文本信息的技术，用于形状识别和字符分配。据Nagy (1988);McBride-Chang等人(2003)的错误识别结果主要是由于一些不同汉字之间的视觉相似性。另一方面，自动语音识别是一种用于处理音频流的基于声学的识别过程，其中语音相似的字符通常会被混淆(Kaki等人，1998;Voll等人，2008;Braho等人，2014)。
### 5 Conclusion and Future Work
本文提出了一种带有标注拼写错误的中文语料库的混合自动生成方法。具体而言，基于OCR和ASR的方法通过替换视觉上和语音上相似的字符来生成标记拼写错误。人类的评估证实了我们提出的方法可以产生常见的错误，这些错误可能是人为的，这些错误可以作为有效的标注拼写错误的CSC。在实验中，我们在生成的语料库上训练了神经标记模型，在基准数据集上的测试结果证实了语料库的质量，进一步证明了我们的语料库生成方法的有效性。
由所提出的方法生成的大规模注释数据集可能作为帮助提高CSC数据驱动模型性能的有用资源，因为大规模注释数据的可用性是应用任何算法或模型之前的第一个重要步骤。在本文中，我们没有把太多的精力放在CSC模型的设计上，我们把它作为未来的工作。为了促进社区的相关研究并使其他研究人员受益，我们将本工作中的代码和数据公开于:https://github.com/wdimmy/Automatic-Corpus-Generation。