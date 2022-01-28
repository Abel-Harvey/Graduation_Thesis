为了更好地解决实体重叠问题，文本提出了一种新的层级的二元标记框架，它通过将关系建模为函数，使主语在该关系函数上被映射到相应宾语来达到三元组提取目的。实验发现，这种方法即使仅使用随机初始化的BERT也可以得到非常大的提升，那更不用说使用预训练好的BERT的效果了，在两个数据集NYT和WebNLG上是有了很大提升的SOTA。(ACL2020 CCF-A)

<!--more-->

<hr>

论文全名：A Novel Cascade Binary Tagging Framework for Relational Triple Extraction

<hr>

# Motivation：
- 本文认为，前面的各种方法多是把关系当作一种离散的标签来对待，而这种分类方式由于其不平衡性（即可能某几种关系很常用，某些关系很少用）会导致关系分类变成一种硬机器学习的任务；
- 尤其是在数据集比较小、少的情况下，很难去构建正确的、完善的三元组；
- 因此本文想出了一种新的方法去完成这个任务，就是对关系进行建模，使得关系成为一种函数，从而将主语映射到对应的宾语上，也就是说，传统的方法是：*f(s,o)-->r*,而本问的方法是：
$$f_r(s) -> o$$

- 本文通过两个阶段：先提取出句子中所有的主语，然后对每一个主语利用关系特定的标记器去同时提取所有可能的宾语。

<hr>

# Model Details:
- 本文的模型通过在三元组层级上进行建模，最大化似然函数进行训练。模型整体分为两个阶段，一个是找出所有的主语，接着是利用特定关系的函数去寻找对应的宾语。

## Encoder：
- 编码器直接使用了BERT(Base-Cased)，由于任务特点，故没有使用segment编码，此外，本文使用的词向量是和BERT模型预训练一致的sub-words(WordPiece embeddings)。

## Decoder：
- 编码器主要就是两部分，都是其了标注的目的，分别是主语的标注器，一个是特定关系的宾语标注器；
### Subject Tagger:
- 如模型图所示，这部分使用了两个一样的模块分别对主语的开始和结束部分进行标记，当其概率超过一定阈值时标记为1；
- 因此，对于图中的Jackie为开始，找到结束为Brown，则该主语完整的为Jackie R Brown。匹配时，使用了前向最近原则，即Jackie只找离它最近的、顺序的结束token的位置，而不会找它之前的；

[![model arch](https://s2.loli.net/2022/01/28/EefHU8tJ5Ib7SPF.png "model arch")](https://s2.loli.net/2022/01/28/EefHU8tJ5Ib7SPF.pnghttp:// "model arch")

### Relation-Specific Object Tagger:
- 这里和主语标记非常类似，不同的在于，它一是和关系特定的，因此标记器非常对(与关系数一样)；二是它训练时还额外增加了主语的编码向量；
- 总之就是，这里利用主语、关系，一起来找宾语；比如实体e1和e2，假设在关系r1的条件下成立，则会通过fr1(e1)找到e2，但是如果在r2下不成立，则e1和e2在此时便不会构成实体对。

<hr>

# Experiment:
- 数据集依旧是NYT24和WebNLG；
- 由于该文章方法简单明了，因此实验相对也非常容易看懂：首先是常规实验的比对，和以前的一些baselines比；然后是消融实验，本文分为三个，分别是随机的BERT、使用LSTM&Glove的，以及预训练好的BERT；之后是针对重叠问题在F1值上的实验；最后是对句子含有不同三元组数量的实验，具体见下三图：


![result-reg&aba](https://s2.loli.net/2022/01/28/n5zgPuIYpkNi2fj.png)
![result-overlap](https://s2.loli.net/2022/01/28/WOt1S2Fq9LVCdBg.png)
![result-SentenceType](https://s2.loli.net/2022/01/28/RNh12fZ8Cz6iHmM.png)

- 可以看到的是，本文所提的方法压倒性地打败了前述baselines，消融实验表明，即使只用随机的BERT或者使用LSTM，也有非常大的提升，说明是模型结构在起作用；
- 通过对三种重叠情况的F1值的分析，可以看出本文在解决重叠问题上也非常出色，并且其标记的方法也很简洁；
- 不同难度的句子类型分析，可见本文方法的能力也极其出众，尤其是对与三元组数量多的情况(即一个句子中含有多个三元组)更明显；
- 除了这些以外，文章还在附录增加了一下额外的实验，但是其数据集并不在该任务上广泛使用，故未写出。

<hr>

# Conclusion:
- 本文的创新点与贡献主要在：1).  用一种新的视角设计了基于特定关系的函数，再针对主语去找宾语； 2). 该方法简单，清晰易懂，相对来说易于实现，实验结果也表明其相当高效； 3). 在提升三元组提取的性能指标上、重叠问题解决能力上和复杂句子的处理上，都将该任务提到了一个新的高度。
- 本文的缺点(站在现在的视角评价)：说实话还真难找到什么缺点，可以有改进与提高，对于方法上以我现在的积累量不知道它有什么缺点。

<hr>

***注***: **本文的所有分析均由个人得出，仅供个人参考，涉及主观评价的部分请自行判断，若有任何误读请留言告知**

<hr>