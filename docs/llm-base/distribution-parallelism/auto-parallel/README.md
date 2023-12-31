

- 分布式训练自动并行论文：https://zhuanlan.zhihu.com/p/642446009
- 北大河图大模型自动并行训练工具Galvatron：https://zhuanlan.zhihu.com/p/591924340
- 大规模模型训练：https://www.zhihu.com/question/508671222
- 如何评价Google的GShard论文？：https://www.zhihu.com/question/404721763/answer/2111040851

---


大模型的自动并行之难主要体现在以下三个方面：

（1）多样性：首先，在并行方式方面，目前大模型的并行方式呈现出百花齐放的态势，
即使是对于同一个算子，不考虑混合并行方式，不同的基础并行方式也会存在显著的差异，从而导致不同的内存开销、通信代价以及计算效率。
其次，在模型方面，各种各样的模型架构最近也是层出不穷，这往往也伴随着不同的模型配置（例如不同输入序列长度，模型层数，模型隐层宽度等），从而造成计算负载上的差异。
另外，在硬件方面，用户往往面临着非常差异化的集群环境，可能会面临不同的内存容量、通信带宽、计算能力等等。
总体上来看，由于上述多样性的存在，没有哪种并行技术总是能够获得最佳训练效率，“自动并行”也就成为了分布式训练的核心挑战。

（2）复杂性：上述分析还相对比较单一，实际上哪怕是对于同一个算子也可以同时应用多种不同的基础并行方式，
如果考虑到由这些基础并行方式复合所构成的混合并行方式，则会导致问题变得非常复杂。
更重要的是，大模型的计算图往往结构非常庞大，对应的也需要更大规模的集群，如果对每个算子都进行探索（包括选取集群中合适的计算资源以及设计相应的混合并行方式），
会带来组合空间爆炸的问题，寻找整个模型的最优分布式执行方案变得难以求解。


（3）实用性：除此之外，实用性也是非常重要的问题。
一方面，在进行自动并行搜索的过程中，对于各种分布式执行方案，必须提供比较精确的内存、通信、计算开销，
否则会导致结果与实际执行偏差过大，产生次优解或者根本无法使用。
为此，就需要非常精准的代价模型，对不同的模型结构和硬件条件进行建模。
另一方面，系统提供自动并行能力所带来的额外时间开销必须在一个可以接受的范围内，过于高昂的搜索代价同样也无法接受。






## One weird trick for parallelizing convolutional neural network
发现不同的层适合用不同的并行方式，具体的，卷积层数据比参数大，适合数据并行，全连接层参数比数据大，适合模型并行。

## Exploring Hidden Dimensions in Parallelizing Convolutional Neural Networks

Alex前面那篇文章直观的提出来，有的层次适合数据并行，有的层次适合模型并行，那么给定一个神经网络，有没有自动的办法找到最优的并行办法呢？

这篇文章就是想解决这个问题。

首先，这篇文章在抽象上更进一步，发现数据并行，模型并行都只是张量切分方式的不同罢了，有的是切数据，有的是切模型，而且对于多维张量，在不同的维度上切分，效果也不同，譬如在sample, channel, width, length等维度都可以切分。

其次，不同的切分方式，都是一种构型（configuration)，不同的构型会导致不同的效果，所以寻找最优的并行方式，其实就是在构型空间里面搜索最优的构型而已，问题形式化成一个搜索问题。

最后，引入了代价模型来衡量每个构型的优劣，并提出了一系列对搜索空间剪枝的策略，并实现了原型系统。


这篇文章勾勒了自动并行的基本框架，很多解决自动并行的工作都是这样一个流程。



## FlexFlow
Beyond Data and Model Parallelism for Deep Neural Networks


提出了execution simulator来完善cost model。



## Tofu

Tofu 提出了一套DSL，方便开发者描述张量的划分策略，使用了类似poly的integer interval analysis来描述并行策略，同样，并行策略的搜索算法上也做了很多很有特色的工作



Tofu与所有其它工作的不同之处在于，它的关注点是operator的划分，其它工作的关注点是tensor的划分，二者当然是等价的。不过，我认为关注点放在tensor的划分上更好一些，这不需要用户修改operator的实现，Tofu需要在DSL里描述operator的实现方式。

这种区别也会反应到API层面，譬如Mindspore和OneFlow 作为通用框架里少数实现了完整的数据并行、模型并行的系统，在Python API上也不同，在Mindspore训练盘古模型的示例代码里可以看到Mindspore 的划分接口是放在operator上的，相反，OneFlow的SBP体系是把划分接口放在张量上，在operator API上单卡和分布式完全一样。





## Mesh-TensorFlow

Mesh-TensorFlow的核心理念也是beyond batch splitting，数据并行是batch splitting，模型并行是张量其它维度的切分。这篇文章把集群的加速卡抽象成mesh结构，提出了一种把张量切分并映射到这个mesh结构的办法。


## GShard




## GSPMD






# 全自动并行

## Alpa




## Unity











