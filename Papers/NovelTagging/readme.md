为了联合地提取实体和关系，本文首次提出了一种新型的标记模式，通过将提取任务转换为标记任务来达到目的。此外，论文研究了多种不同的端到端模型，通过设计实验，发现论文最终的结果优于以前所有的pipeline方法和联合学习方法。(ACL2017 CCF-A)

<!--more-->

<hr>

论文全名：Joint Extraction of Entities and Relations Based on a Novel Tagging Scheme

<hr>

# Model Deatils:
- 本论文的机制在现在看来是比较简单的，但是不能否认当时提出时的重要意义；

## 标记模式：
- 如题目所示，本文方法的主要特点是将三元组提取任务转换为了标记的任务，怎么标记呢？如下图：
[![tagging scheme](https://s2.loli.net/2022/01/24/hHWvxzbBK3l2g1r.png "tagging scheme")](https://s2.loli.net/2022/01/24/hHWvxzbBK3l2g1r.png "tagging scheme")

- 可见，它的实体有三个部分，分别是实体位置、关系类型和关系角色；实体位置就是当前的token是实体的(开始、中间，结尾，单个)，而关系类型就是正常的关系类别，关系角色用1和2表示，1表示主语，2表示宾语；
- 如图中所示，`United States`，就分别被表示为*主语的开始、主语的结束*，而他们都是与关系Country-President有关，1表示它在该关系中是主语；
- 文章还提到，如果遇到句子中有多个三元组对结果会产生困扰时，例如假设这里的关系都是Company-Founder，则按照最近原则进行匹配，因为`United States`和`Trump`挨得近，故他们组成一个三元组；
- 不用多说应该知道，该方法默认了实体对只参与一个三元组的情况，也就是实体重叠问题无法被解决。关于实体重叠问题，可以关注文章<a href="http://aca.abelcse.cn/index.php/archives/96/">**CopyRE**</a>。

## 模型结构：
- 本文使用了双向的LSTM来进行编码，使用了LSTM来进行解码，如下图：

[![model arch](https://s2.loli.net/2022/01/24/tfu39NhpkyqLgWF.png "model arch")](https://s2.loli.net/2022/01/24/tfu39NhpkyqLgWF.png "model arch")

- 模型对与输入的句子，先转换为对应的词向量，接着利用双向LSTM得到相应的两个方向的隐状态，通过将其拼接得到每一个地方的隐状态向量传递给解码部分，解码利用单向的LSTM来解码，利用Softmax层得到最终的标记信息；
- 词向量利用的是word2vec在相应数据集上产生的。

<hr>

# Experiment:
- 数据集使用的是由论文<a href="http://aca.abelcse.cn/index.php/archives/15/">CoType</a>提供的NYT数据集，结果如表1：

[![results](https://s2.loli.net/2022/01/24/q8Qdbv4NmkwfpMB.png "results")](https://s2.loli.net/2022/01/24/q8Qdbv4NmkwfpMB.png "results")

- 通过比较发现，联合提取的方法比pipeline的方法要好，而本文所提出的基于标注法的LSTM-LSTM，加上偏置的目标优化函数的模型在F1值上实现了最优，文章认为一是因为标注法切实有效，二是因为CRF这种统计类的方法与LSTM是两个不同的结构，因此在进行编码解码交互时没有是同样结构的LSTM好；
- 此外，文章还设计了针对实体的三种实验比对，以及关于目标函数的比对，这里就不再详述；
- 实验最后部分是案例学习，针对LSTM-LSTM和LSTM-LSTM-Bias进行了三种案例的分析，这里也不再详细说。

<hr>

# Conclusion:
- 本文的主要贡献在于：1). 首次真正意义上的端到端的联合提取方法； 2). 设计了标记模型将原本的任务转换为在句子中对实体进行标记的任务；3). 利用端到端的方法，避免了以前的联合学习方法繁琐复杂的特征工程，减少了人工的参与；
- 本文的缺点与不足在于(站在现在的视角评价)：1). 这种标记方法无法解决重叠问题，虽然文章最后对未来工作部分说要改进softmax层使之可以进行多分类，但是后续的其他人的工作表明这并不是优秀的方案；2). 性能虽然相对于pipeline方法有提升，但还是不够好；

<hr>

***注***: **本文的所有分析均由个人得出，仅供个人参考，涉及主观评价的部分请自行判断，若有任何误读请留言告知**

<hr>