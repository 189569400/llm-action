

- 

在数据采集过程中，为了数据的全面性和代表性，从多个来源进行数据收集，包括但不限于网页、书籍、研究论文、代码等，各类别数据分布如下所示。



并且对数据进行清洗，如下图所示，主要关注数据的频率和质量：数据频率：借助LSH-like和Embedding特征对数据进行聚类和去重，主要是对每个聚类的簇给文档、段落、句子进行去重和打分，分值用于用于最终的数据采样。数据质量：句子级别质量过滤，但未说明明确过滤规则。不过从下面模型安全部分可以得知，对数据进行了暴力、色情、种族歧视、仇恨言论等有害内容过滤，但应该还包含其他内容。

PS：报告中没有给出过滤后数据采样比例&数据分布情况，比较遗憾。但从垂域效果来看，医疗和法律数据应该不会少，并且从数据本身质量来看，书籍&论文数据的采样率应该也会比较高。



