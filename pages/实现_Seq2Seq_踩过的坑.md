# 实现 Seq2Seq 踩过的坑

(2021-01-18)

记录一下自己实现 [RNN-based Seq2Seq](https://github.com/LinjianLi/Seq2Seq-PyTorch) 时踩过的坑。以前没写过太多东西，所以说的话可能有些啰嗦、冗余，请见谅。

## 会用到的一些符号和表示

假设原始 target 序列是 $(y_{0}, y_{1}, y_{2}, ..., y_{n})$ ，长度是 $n$ ，如果是 batch size > 1 ，则此处的 $n$ 指的是最长的序列的长度。构造数据的时候可能会添加 $SOS$ 、 $EOS$ 、 $PAD$ 三种 token 。

## 构造 Decoder 端的输入输出要注意第一个 Token 以及步数、长度

1. Decoder 输入的第一个 token 一定是 $SOS$ (start of sentence) token 。
2. Decoder 输出的第一个 token 一定是第一个非 $SOS$ token 。
3. Teacher forcing 阶段 decoder 输入的长度是 $n + 1$ ，非 teacher forcing 阶段的前进步数也是 $n + 1$ 。
4. 计算 loss 的时候不要包括 $SOS$ token 。

我在一开始构造训练数据的时候，对于 target 序列，我直接构造成 $(SOS, y_{0}, y_{1}, y_{2}, ..., y_{n}, EOS)$ ，长度是 $n + 2$ 。然后在训练的时候，teacher forcing 阶段的输入，以及计算 loss 是用的 target 都是同样的 $n + 2$ 的序列。这样就会导致一些问题。这里可以提前说一下，后面的内容会多次说明，长度为 $n + 2$ 的序列会导致问题，要改成长度 $n + 1$ 。读者可以留意一下。

(其实 Seq2Seq 论文 [
Sequence to Sequence Learning with Neural Networks](https://arxiv.org/abs/1409.3215) 里面构造数据的时候就是没有包括 $SOS$ token 的，但是我没有看论文原文，自己构造数据的时候就出问题了。如果按照论文原文来做，就不会遇到问题。不过既然遇到了，我还是在这里记录一下这个问题，并且记录一下自己的思考过程。)

### 会导致的问题

1. **如果不使用 teacher forcing**，decoder 的输入确实是自己上一步的输出。模型在第一步输入的是 $SOS$ token，这没什么问题。模型第一步的输出按理说是第二步的输入，因此第一步的输出按理说应该是 $y_{0}$ 。于是就有了以下问题
    1. 假如第一步真的输出 $y_{0}$ 了，那第二步的输入确实就是 $y_{0}$ ，这里没问题。但问题是，如果第一步输出了 $y_{0}$ ，在训练过程中计算 loss 的时候，target 的第一个是 $SOS$ token，这时算出来的 loss 就会很大。相当于训练的时候**不鼓励**模型第一个输出是 $y_{0}$ 。（包括前面 teacher forcing 阶段，计算 loss 时 target 的第一个 token 也是 SOS token 。）
    2. 如果模型知道了训练过程不鼓励第一步输出 $y_{0}$ ，而是鼓励输出 $SOS$ token，那么模型就开始学会第一步输出 $SOS$ token，至少这样 loss 可以降下来。问题是，第一步输出了 $SOS$ token，第二步的输入是第一步的输出，于是第二步又输入一次 $SOS$ token，相当于每次非 teacher forcing 的训练都需要**连续输入两个 $SOS$ token**。再牵扯上一些 max length 相关的长度设置，就可能出现很多其他预想不到的 bug。**注意这里的 bug 不一定是指程序运行会崩溃。崩溃不可怕，至少知道出了错。最怕的就是运行时一切顺利，以为自己没有错误，但模型悄悄的胡乱学习，导致最后效果奇差**。
    3. 因此，可以看出来，计算 loss 的 target 不可以包括 $SOS$ token，**长度是 $n + 2 - 1 = n + 1$**。
2. **如果使用了 teacher forcing**
    1. Teacher forcing 就是 decoder 的输入是 target 而非自己的输出。这时 decoder 只需要学会把输入**原样输出**就可以了，因为输入和 target 都是 $(SOS, y_{0}, y_{1}, y_{2}, ..., y_{n}, EOS)$ ，长度为 $n + 2$ 。这个阶段对模型参数进行更新就会让模型“学坏”。当模型学会了“抄袭”，在 inference 阶段，第一步输入 $SOS$ token ，第一步就会输出 $SOS$ token ；第二步的输入是第一步的输出，即 $SOS$ token ，因此第二步也会输入并输出 $SOS$ token ；以此循环，模型将永远只会输出 $SOS$ token 。这个设想在后面会用实验进行验证，在此我暂且称之为“**抄袭问题**”，方便下文引用。并且，计算 loss 的时候 target 第一个 token 是 $SOS$ token，这里也可以看到训练过程“鼓励”模型第一步生成 $SOS$ token 而非 $y_{0}$ ，跟前面写的不使用 teacher forcing 阶段的结论一样。
    2. (Batch size = 1 的情况) 此外，teacher forcing 阶段的最后一个输入如果是 $EOS$ token，相当于**提示**了模型什么时候要结束。但其实什么时候要结束应该让模型学习、**由模型来决定**。模型要学会的是，在第倒数第二步输出了 $y_{n}$ ，在最后一步输入了上一步的输出 $y_{n}$ ，就知道这时应该结束了，最后一步该输出 $EOS$ token 了。因此模型的最后一步输入应该是 $y_{n}$ 而不是 $EOS$ token 。少输入最后一个 token（注意这里我说的是最后一个 token，没有说一定是 $EOS$ token，原因在下一点会写）。所以， **decoder 端输入的序列的长度是减去了最后一个 token 的长度，即 $n + 2 - 1 = n + 1$**。
    3. (Batch size > 1 的情况) 我们假设这里的 $n$ 是这个 batch 最长的序列的长度。在一个 batch 内部，除了最长的那一个序列，其他所有短的序列都会被 padding，即 $EOS$ token 后面会有若干 $PAD$ token，即每个序列的长度都统一成了 $n + 2$ 。这时对于这些短的序列又需要输入 $EOS$ token 了，因为模型需要学会当遇到了 (即已经生成了) $EOS$ token 之后，后面就一直输出 $PAD$ token 。因为 decoder 输入的第一个 token 一定是 $SOS$ token，计算 loss 的 target 又不包括 $SOS$ token，所以 decoder 输出的长度肯定比 target 的长度大 1 ，即前者是 $n + 2$ 后者是 $n + 1$ 。因此 decoder 的输入应该减掉最后一个 token，长度变成 $n + 2 - 1 = n + 1$ 。这个在 IBM 的 PyTorch-Seq2Seq 项目的代码中有体现出来，具体就是 DecoderRNN 里的 `_validate_args()` 函数里有一行 `max_length = inputs.size(1) - 1` 表达了这一思想 (只不过它的注释是 `# minus the start of sequence symbol` ，是指计算 loss 用的 target 不要包含 $SOS$ token，但核心思想是一样的) 。

### 验证

#### 抄袭问题

##### 数据获取及实验设置

我直接用 `random.randint()` 生成训练原始数据，先生成长度为 10 的 source ，然后 source 中的每个元素 +3 就得到了 target 。未 debug 的代码会在 source 和 target 的首尾都添加 $SOS$ token 和 $EOS$ token ，而已 debug 的代码不会添加 $SOS$ token ，只会添加 $EOS$ token 。

超参数，例如 embedding size 、 hidden size 之类的参数设置为 100 到 200 ，这个不重要。RNN cell 是 GRU，optimizer 是 Adam ，learning rate 是 10e-3，训练 20 轮。最重要的是， teacher forcing ratio 要设置为 1 ，因为我们要验证的是前面写的 teacher forcing 阶段可能发生的“抄袭问题”。

##### 实验结果

1. 未 debug 的代码收敛得非常快速，已 debug 的代码收敛要慢一些。但最终的 training loss (NLL loss) 都会收敛到小于 0.01 ，即在训练集上对于正确 token 预测的概率大于 $e^{-0.01} = 0.99$ 。
2. 在训练时模型未接触过的测试集上做 inference 时
    1. 未 debug 的代码确实一直输出 $SOS$ token ，相当于这个模型学废了，也证实了前面说的“抄袭问题”的设想。
    2. 已 debug 的代码能够在未见过的测试集达到完全拟合的程度（因为 target 就是 source +3 ，非常容易学习）。
3. 实验中还发现，如果 teacher forcing ratio 的值设置不为 1 ，例如 0.5 ，那未 debug 的代码的 training loss 还是可以收敛，在测试集上也可以做到完美拟合。应该是因为有 0.5 的概率不进行 teacher forcing ，这些时候就把“抄袭问题”纠正过来了。但是实验用的数据太简单了，容易纠正，一旦遇到复杂一点任务、数据，可能就没那么容易纠正了。

### 总结一下应该怎么做

1. 计算 loss 时的 target 应该是 $(y_{0}, y_{1}, y_{2}, ..., y_{n}, EOS)$ ，即除去第一个 $SOS$ token ，长度是 $n + 1$ 。
2. Teacher forcing 阶段的输入应该是 $(SOS, y_{0}, y_{1}, y_{2}, ..., y_{n})$ ，即除去序列的最后一个 token ，对于 batch size > 1 的情况也是除去序列的最后一个 token 。这样就能保证输出的序列长度是 $n + 1$ ，计算 loss 的时候不会出现维度对应不上。

所以可以在构造 target 数据的时候就只构造 $(y_{0}, y_{1}, y_{2}, ..., y_{n}, EOS)$ ，长度是 $n + 1$ ，就像 Seq2Seq 论文原文一样。这样做 Trainer 的代码看起来也更符合直觉，Trainer 就只需要负责训练相关的操作，不用关心数据格式，不用先取出 target 然后再切掉第一个 token 。

然后在 Seq2Seq 模型代码里手动添加 $SOS$ token 即可。要注意添加之后，如果用 `len()` 函数来获取最大步数，需要注意是 $n + 2$ 还是 $n + 1$ 。

### 题外话

其实有点类似 Transformer-based Seq2Seq 的输入输出错位的操作。Decoding 阶段 Transformer 的输入是 $(SOS, x_{0}, x_{1}, x_{2}, ..., x_{t})$ ，输出是 $(x_{0}, x_{1}, x_{2}, ..., x_{t + 1})$ ，相当于错了一位。只不过 Transformer 要多次进行整段的输入输出，直到达到 max length 或者遇到 $EOS$ token 。如果把 $x_{t}$ 当做 $y_{n}$ ， $x_{t + 1}$ 当做 $EOS$ ，那就和我们前面说的类似了。
