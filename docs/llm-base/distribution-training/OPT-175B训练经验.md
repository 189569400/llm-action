
## OPT-175B是如何炼成的
- https://zhuanlan.zhihu.com/p/622061951


### 训练大模型的痛点

我们都知道，训练大模型需要以月计的时间，比如这次OPT-175B就要在1000个80G A100上开足马力训练3000亿个词，需要至少33天时间。这个33天还只是指连续训练的时间，而实际上中间肯定有无数次回退和重启。只要手指按下按钮，烧钱就开始，最关键的是，烧完钱之后还不知道是否能成功，这种压力是在家折腾小模型无法相比的。

但听完Susan的介绍，感觉实际操作中还是试错、试错再试错这一套实验方法，没有特别高深的炫技。Susan之前没有训练大模型的经验，是在试错的过程中不断成长，积累经验，最终炼丹成功。



### 崩溃的超参和崩溃的机器

训练模型就是找出合适的超参，然而这在大模型上是非常困难的事情，适用于小模型的超参未必适用于大模型，所以很难使用先从小模型着手做试验、再扩展到大模型的方法。




### Bug是人之常情

Susan团队准备环境花了一个月，一切部署妥当后兴冲冲地跑训练，结果烧钱了几天，发现代码有Bug……

修正代码后，有一次试验，模型效果很好，预测下一个词的准确率很高，正准备庆祝，却发现原来是当初清洗处理数据集时，有人加入了大量的反斜杠转义符却忘记删除，模型准确无误地精确预测了这些反斜杠……


### 数值稳定性（Numerical Stability）


整个训练过程，团队最关注的就是数值稳定性。经过多次尝试，她们决定使用bloat16作为基础数值格式，一举奠定了成功的基础。中途遇到一个问题，卡了一段时间。Susan某天在BigScienece的Megatron fork上，看到有人提出一个类似的PR，使用一个小小的等价变换，就解决了这个数值稳定性问题。

n * (A dot B) === (sqrt(n) * A) dot (sqrt(n) * B)



### 崩溃的超参和崩溃的机器


训练模型就是找出合适的超参，然而这在大模型上是非常困难的事情，适用于小模型的超参未必适用于大模型，所以很难使用先从小模型着手做试验、再扩展到大模型的方法。

这方面OpenAI就很厉害，他们在训练GPT-4的时候研究了一个方法，通过在小模型上做试验，在开始训练大模型之前就准确预测训练完之后的最终Loss。Susan她们没有找到这个方法，只能参考前人的论文以及不停地试验。

另一个困扰她们的问题是机器不停地坏，今天坏个显卡，明天坏一块内存，后天硬盘校验错误，大后天NVLink又坏了，硬件问题层出不穷……



### 无Benchmark可用

由于她们忙于训练自己的模型，没有闲暇再弄Benchmark。而这个领域又发展得很快，过往的Benchmark很快就被比下去，不适合再作为Benchmark。



以Prompt为例，改一下Prompt的标准格式，比如大小写，模型的效果都大不一样，你很难分辨究竟是因为Prompt的格式问题影响性能，还是模型本身的性能改善了。



### 黎明前夕


2021年11月，Run 12

她们觉得问题解决得七七八八了，差不多是时候来最后一轮试验。然而她们不确定前面的试验定下来的超参能否适用于最后的正式训练。根据公开论文，她们发现GPT-3和Megatron的设置非常接近，很可能是训练这种大模型的通用真理，于是她们决定抄GPT-3和Megatron论文的设置。

最后一轮试验，一上来就失败了15次，其中2次是GPU烧了，6次是CUDA错误，还有一些不明错误。还有2次是因为她们自己写的checkpoint的恢复程序又有bug，导致无法从上一个checkpoint恢复。

此后她们不停地降低 Learning rate，在56天之后， 大概经过 50多次重启， 模型 survived 143K steps，终于成功了！下图中断掉的空白是她们的备份失败了……



## 问答环节

Q：如果你现在重新回到训练开始之前，你再希望改善什么。

A：Much Much more data。超参什么的都是浮云，关键还是训练数据。另外，发现bfloat16是最适合的格式也是里程碑。

Q：为什么这个项目团队只有5个人？是因为老板认为5个人就足够，还是因为传说中的世界上只有约200人能够训练这些大型模型，而你们是其中之一。

A：途中有很多其他人员协助，但是核心团队确实只有5人。因为我们跑一段训练就要几个星期，而每次只跑一段训练，太多人就浪费了。

Q：一天中你有多少时间是焦虑地盯着Loss Curve？

A：必须成功的压力很大，确实很多时间都盯着最新的Loss（我猜测比盯着K线图还紧张），试图在失败之前就介入，免得浪费更多时间，还有很多时间在解决硬件问题和debug代码。

Q：我个人训练过2.7B的模型，这个规模的模型很容易就搞定，参数数量到达哪个临界点之后，模型会变得非常不稳定？

A：不确定，根据我浅薄的经验，以及Google等的论文，可能临界点是70B。

Q：怎么解决硬件问题？

A：和云工程师一起解决。老黄的A100相比V100来说太不稳定了，希望未来能解决。

提问者补充：传闻最新的H100更多问题，祝你好运。

Q：这个项目怎么定义成功？

A：当时没什么成功的概念，只是单纯地希望Loss低到一定程度，以及效果比benchmark好。在做项目的时候还没精力和当时的竞争模型做比较，只专注于loss loss loss。现在回过头来才有空跟竞品比较。

Q：你刚才强调数据质量，并说如果当初能有更好的数据，模型训练的效果也会更好。既然数据质量这么重要，如何定义数据的好与坏？

A：书、论文和代码直觉上最好质量的数据。对于特定领域的模型，比如编程模型，一看就知道哪些是好的代码；但对于这些大型通用模型来说，我们不知道怎么定义数据的好坏，要等最终模型训练出来之后才知道喂的数据好不好。

Q：出现错误的时候，你怎么知道是什么原因导致的？有可能是bug，有可能是参数问题，有可能是硬件故障。

A：没有好的方法，只能不停地试错。我们还曾经遇到晶体管的问题，完全没有办法复现问题。完全没有通用的方法，只能通过失败积累经验，但到下一代机子H100时，积累的这些经验又没用了，要从头开始试错。

Q：为什么越新型号的机子越不稳定？

A：你要问老黄。如果老黄的驱动是开源的，我们还能帮他debug，很可惜不是，所以爱莫能助。



## 大模型调研之 OPT-175B是如何炼成的

- https://zhuanlan.zhihu.com/p/644780639



重要控制参数：
- weight decay
- global grad norm clipping
- Adam优化器beta2
- Adam优化器epsilon
- 增加warmup的步数（steps）




教训：要先在小规模数据上试跑，以便提早发现代码bug



```
Run 11: 

一组认为可能还不错的参数设置：

2M batch size
FP32 Adam
Tensor parallel(8x MP)

新data，来自实验29（之前的数据集中存在问题，曾额外添加了转义字符，于是模型训练时通过找转义字符而降低了loss，而非真正学到东西）

训练中学习positional embedding。因为不太有信心学到positional embedding，所以通过正弦init(sinusoidal init)学习positional embedding，使之与原transformer论文相符合。

weight decay, 0.05
LR of 3+4, end LR of le-5。学习率开始时高，后期变低以避免训练变得不稳定
No dropout on embeddings

Normformer (impact on grad norm is making earlier layers be more similar with later layers)

Gradient pre-divide factor: 32(Naman has been running with this)，目的是实现局部梯度积累(local gradient accumulation)

Clip (12 norm): 2.5(后被证实是重要的，当时并不清楚这样的重要性)


将Gelu替换为Relu，因为Gelu的公式里有个x^3，可能造成不稳定

```




```
Run 12.00:Beginning of the "final" run


Overall weight initialization updated: 总体的权重初始化

Removed extra layer norms from Normformer setup

Removed embedding scaling

Gaussian init for learned positional embeddings (instead of sinusoidal init)，此处观点是，也许较小的标准差有助于提高稳定性

Weight decay = 0.1

Clipping = 1.0

Adam beta2 = 0.95

Max LR of 1.2e-4





Run 12.(01-15): Mainly "systems" issues 大量系统问题

Lost GPU(12.01, 12.10)
CUDA errors (12.02, 12.03, 12.04, 12.09, 12.15, 12.17)
Job hanging (12.05, 12.06)
NCCL error (12.08)
Job slowdown (12.11)



For Run 12.16:

Reduce clipping to 0.3
Backup plan:reset Adam state and do fresh warmup


Runs 12.(17-34):

Hardware issues (ECC errors, lost GPU, high#DRAM correctables, etc.)

Mysterious job hanging issues
Blob storage issues
Gradient overflow issues
Used loss scaling state "reset" with restarts to try and circumvent the same fate


```






