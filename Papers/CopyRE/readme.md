本文将句子按照重叠程度划分为了三种不同的实体重叠类型(首创)：Normal，EPO和SEO。而以前的方法都只关注于Normal，故提取的精确度提不上去。本文基于Seq2Seq方式，利用复制机制，采用了两种解码器策略，即单个解码器和多个分离解码器方式，从而很好地完成了关系三元组提取任务。论文方法表现非常好，相比于其以前的方法提升较大。(ACL2018 CCF-A)
<!--more-->

<hr>

论文题目：Extracting Relational Facts by an End-to-End Neural Model with Copy Mechanism

<hr>

# 实体重叠问题
- 即Overlap，本文通过对常用的句子进行分析，认为这些句子中有很多都含有较为复杂的实体关系，而以往的方法要么是提前定义好了实体无须进行实体抽取，要么就是默认实体对仅参与一种关系。这些方式使得最终的结果都并不优秀；
- 因此论文将重叠问题分为了三类，奠定了以后RTE任务在解决实体重叠问题上的基础；
- 三类分别是`Normal`：即默认类型，仅参与一种关系；`EPO,EntityPairOverlap`：即一对实体，他们之间有两种及以上的关系；`SEO,SingleEntityOverlap`：即单个实体参与了多种不同实体对之间的不同关系，具体差别如下图：
[![Overlap Pattern](https://s2.loli.net/2022/01/23/4icvI2qjp8ghmU6.png "Overlap Pattern")](https://s2.loli.net/2022/01/23/4icvI2qjp8ghmU6.png "Overlap Pattern")

<hr>

# Model Details:
## Encoder:
- 论文先将输入的句子S的单词转换为词向量，然后利用双向的RNN进行编码获得其输出向量与隐藏状态向量，最终的输出向量是通过对两个方向的输出向量拼接得到的。而最后两个方向的隐状态向量被拼接起来表示句子S；
- 模型的整体架构如图：
[![model arch](https://s2.loli.net/2022/01/23/t1W3RqKLxdyO9cn.png "model arch")](https://s2.loli.net/2022/01/23/t1W3RqKLxdyO9cn.png "model arch")

## OneDecoder Model:
- 本文在方法上最大的不同在于其解码器的设计，第一种解码器是单解码器：
- 首先解码器会产生一个关系，该关系是利用全连接层计算置信向量得到的。为了让模型有结束标记，论文设计了NA-Triplet，相应的它由NA-relation和NA-Entity Pair组成，因此文章是分别得到一般的关系置信向量和NA关系的置信向量，之后拼接,softmax得到最终的置信向量；
- 接着从源输入中复制一个主语实体过来，当然复制是经过学习后的概率分布去选择的，如模型整体图中的*Entity Copy*部分；
- 最后，解码器会复制下一个实体，即宾语，为了不把主语重复地复制下来，文章使用了一个遮盖的矩阵，简而言之，就是把刚刚复制过的主语的下标遮盖了以使其概率为0；
- 当然，复制主语和宾语时，都有其对应的NA置信向量，操作与关系的产生一样；
- 为了较好地完成任务，模型在解码时利用了注意力机制，以获得更全面的向量信息；单解码器的结构如下图：
[![OneDecode](https://s2.loli.net/2022/01/23/gXScsTIHCVh16nZ.png "OneDecode")](https://s2.loli.net/2022/01/23/gXScsTIHCVh16nZ.png "OneDecode")

**可见，复制机制因主语宾语可以被多次复制而有效的解决了重叠问题**

## MultiDecoder Model:
- 这个是单解码器的扩展，其主要不同在于，单解码器是一次产生一个三元组，这个则是由多个解码器产生不同的三元组；其结构如图，绿色框和蓝色框代表了两个不同的解码器：
[![MultiDecoder](https://s2.loli.net/2022/01/23/AR1ctkDwBJuxYz8.png "MultiDecoder")](https://s2.loli.net/2022/01/23/AR1ctkDwBJuxYz8.png "MultiDecoder")

- 受制于RNN的特点，这是个顺序的结构，即第一个解码器产生第一个三元组，接着第二个解码器结合先前信息产生第二个三元组；

<hr>

# Experiment:
- 数据集是非常广泛的NYT(24)和WebNLG，注意这里WebNLG的关系数量写的是246，之后的论文更正了其为216；
- 本文对数据集做了很多处理，主要是分出重叠类别和训练、测试及验证集；
- 论文选取的对比文章是2017年的***NovelTagging***，该论文的简单介绍我也写了，详见<a href="http://aca.abelcse.cn/index.php/archives/73/">**论文阅读(NovelTagging)**</a>；
## Results：
- 先放上主要的两个结果：

[![result](https://s2.loli.net/2022/01/23/CA9WYdky7jgb3Ma.png "result")](https://s2.loli.net/2022/01/23/CA9WYdky7jgb3Ma.png "result")

- 可见，本论文在召回率与F1值上相比NovelTagging都极大的提升了，但是在精度上却不如意；论文分析认为是NovelTagging仅关注于Normal类型的三元组，提取的相对来说简单，反而使得准确度提高，但其弱点就是提取不了复杂的，故召回率低。简单说，就是人家就挑软的捏，所以不全，但是能保证这些软柿子尽可能都是对的，而本论文都要去捏一下，虽然全了，但因为难度大，因此影响了准确性；
- 相应的，图上可以看出，在Normal情况下确实NovelTagging表现很不错，但是在EPO和SEO情况下就难以与本论文进行比较了；
- 此外，论文还对不同难度的句子做了比较，即一个句子中含有的三元组数量，从1，2，3，4，到>=5，共5类，此处就不贴图了，结果就是虽然随着难度增加都会下降，但是本文的方法确实比NovelTagging好，此外多解码器比单解码器好；
- 最后，论文对单、多解码器在实体生成能力和关系生产能力上做了比较，实体生成能力上多解码器更强，论文认为这是由于多解码器的不同解码器会产生不同的实体；
- 但是！！关系产生上这两个差别不大，并且单解码器能力在两个数据集上都稍微优于多解码器，但是论文却没有进行分析，解释原因！
[![OneVSMulti](https://s2.loli.net/2022/01/23/3pSYqKeGr7Umk4b.png "OneVSMulti")](https://s2.loli.net/2022/01/23/3pSYqKeGr7Umk4b.png "OneVSMulti")

<hr>

# Conclusion:
- 本文主要的贡献在于：1). 分析实际的句子，划分了三个重叠的类别，即Normal、EPO、SEO，为以后的RTE任务奠定了实体重叠问题这个挑战的基础；2). 利用Seq2Seq方式，设计了两种解码模式，并使用复制机制，巧妙地缓解了实体重叠问题；3). 在Recall和F1-Score指标上较大地推动了RTE任务进展。
- 本文的缺点在于(站在现在的视角评价)：1). 其性能在绝对意义上还不够好；2). 多解码器和单解码器的比较部分的消融实验分析地很草率，难以信服；3). 使用了注意力机制，但整体结构依旧是利用的双向RNN，限制了模型的运行能力。

<hr>

***注***: **本文的所有分析均由个人得出，仅供个人参考，涉及主观评价的部分请自行判断，若有任何误读请留言告知**

<hr>