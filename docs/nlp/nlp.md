# MLP

设隐藏层的激活函数为$\phi$，
给定一个小批量样本$\mathbf{X} \in \mathbb{R}^{n \times d}$，
其中批量大小为$n$，输入维度为$d$，
则隐藏层的输出$\mathbf{H} \in \mathbb{R}^{n \times h}$通过下式计算：

$$\mathbf{H} = \phi(\mathbf{X} \mathbf{W}_{xh} + \mathbf{b}_h) \tag{1}$$

隐藏层权重参数为$\mathbf{W}_{xh} \in \mathbb{R}^{d \times h}$，
偏置参数为$\mathbf{b}_h \in \mathbb{R}^{1 \times h}$，
以及隐藏单元的数目为$h$。

输出层由下式给出：

$$\mathbf{O} = \mathbf{H} \mathbf{W}_{hq} + \mathbf{b}_q, \tag{2}$$

其中，$\mathbf{O} \in \mathbb{R}^{n \times q}$是输出变量，
$\mathbf{W}_{hq} \in \mathbb{R}^{h \times q}$是权重参数，
$\mathbf{b}_q \in \mathbb{R}^{1 \times q}$是输出层的偏置参数。
如果是分类问题，我们可以用$\text{softmax}(\mathbf{O})$
来计算输出类别的概率分布。

$$softmax(O) = \frac{e^{O_i}}{\sum_{n=1}^{N} e^{O_i}} \tag{3}$$

# HMM 

$$P(x_1, \ldots, x_T) = \prod_{t=1}^T P(x_t \mid x_{t-1})\\\\
\text{ 当 } P(x_1 \mid x_0) = P(x_1). \tag{1}$$

$$
\begin{aligned}
P(x_{t+1} \mid x_{t-1}) 
&= \frac{\sum_{x_t} P(x_{t+1}, x_t, x_{t-1})}{P(x_{t-1})}\\\\
&= \frac{\sum_{x_t} P(x_{t+1} \mid x_t, x_{t-1}) P(x_t, x_{t-1})}{P(x_{t-1})}\\\\
&= \sum_{x_t} P(x_{t+1} \mid x_t) P(x_t \mid x_{t-1}) 
\end{aligned} 
$$

实际上使用：

$$P(x_{t+1} \mid x_t, x_{t-1}) = P(x_{t+1} \mid x_t) \tag{3}$$

序列建模的近似公式：

$$
\begin{aligned}
P(x_1, x_2, x_3, x_4) &=  P(x_1) P(x_2) P(x_3) P(x_4),\\\\
P(x_1, x_2, x_3, x_4) &=  P(x_1) P(x_2  \mid  x_1) P(x_3  \mid  x_2) P(x_4  \mid  x_3),\\\\
P(x_1, x_2, x_3, x_4) &=  P(x_1) P(x_2  \mid  x_1) P(x_3  \mid  x_1, x_2) P(x_4  \mid  x_2, x_3).
\end{aligned}
$$

通常，涉及一个、两个和三个变量的概率公式分别被称为
*一元语法*（unigram）、*二元语法*（bigram）和*三元语法*（trigram）模型。

# RNN

**循环神经网络**（recurrent neural network，RNN）则可以更好地处理序列信息。
循环神经网络通过引入状态变量存储过去的信息和当前的输入，从而可以确定当前的输出。

$$x_t \sim P(x_t \mid x_{t-1}, \ldots, x_1).$$

> 问题：

输入$x_{t-1}, \ldots, x_1$本身因$t$而异。
也就是说，输入的数据量将会随着我们遇到的数据量的增加而增加，
因此需要一个近似方法来使这个计算变得容易处理。

- **第一种策略**

假设在现实情况下相当长的序列 $x_{t-1}, \ldots, x_1$ 可能是不必要的，
因此我们只需要满足某个长度为$\tau$的时间跨度，
即使用观测序列。

$$x_{t-1}, \ldots, x_{t-\tau}$$

- **第二种策略**

保留一些对过去观测的总结 $h_{t}$，
并且同时更新预测 
$\hat{x}\_{t}$
和总结 
$h_t$。
这就产生了基于$\hat{x}\_t = P(x_t \mid h_{t})$估计$x_t$，
以及公式$h_t = g(h_{t-1}, x_{t-1})$更新的模型。

<center>
<img src="../img/rnn.png" width="620px">
</center>

> rnn

隐变量模型：

$$P(x_t \mid x_{t-1}, \ldots, x_1) \approx P(x_t \mid h_{t-1}),$$

通常，我们可以基于当前输入$x_{t}$和先前隐状态$h_{t-1}$来计算时间步$t$处的任何时间的隐状态：

$$h_t = f(x_{t}, h_{t-1}).$$

1. 假设时间步$t$有小批量输入$\mathbf{X}\_t \in \mathbb{R}^{n \times d}$。
2. 用$\mathbf{H}\_t  \in \mathbb{R}^{n \times h}$表示时间步$t$的隐藏变量。 
3. 引入前一个时间步的隐藏变量$\mathbf{H}\_{t-1}$，并引入了一个新的权重参数$\mathbf{W}\_{hh} \in \mathbb{R}^{h \times h}$。

$$\mathbf{H}\_t = \phi(\mathbf{X}\_t \mathbf{W}\_{xh} + \mathbf{H}\_{t-1} \mathbf{W}_{hh}  + \mathbf{b}_h).$$

多添加了一项$\mathbf{H}\_{t-1} \mathbf{W}\_{hh}$，保留了序列直到其当前时间步的历史信息，
因此这样的隐藏变量被称为*隐状态*（hidden state）。

4. 对于时间步$t$，输出层的输出类似于多层感知机中的计算：

$$\mathbf{O}\_t = \mathbf{H}\_t \mathbf{W}\_{hq} + \mathbf{b}\_q.$$

循环神经网络的参数包括：
- 隐藏层的权重: $\mathbf{W}\_{xh} \in \mathbb{R}^{d \times h}, \mathbf{W}\_{hh} \in \mathbb{R}^{h \times h}$
- 隐藏层的偏置 $\mathbf{b}_h \in \mathbb{R}^{1 \times h}$，
- 输出层的权重: $\mathbf{W}_{hq} \in \mathbb{R}^{h \times q}$
- 输出层的偏置: $\mathbf{b}_q \in \mathbb{R}^{1 \times q}$。
*值得一提的是，即使在不同的时间步，循环神经网络也总是使用这些模型参数。
因此，循环神经网络的参数开销不会随着时间步的增加而增加。*

# Deep RNN

可以将多层循环神经网络堆叠在一起，通过对几个简单层的组合，产生了一个灵活的机制。

<center>
<img src="../img/deep_rnn.png" width="320px">
</center>

1. 假设在时间$t$有一个小批量的输入数据：$\mathbf{X}_t \in \mathbb{R}^{n \times d}$ （样本数：$n$，每个样本中的输入数：$d$）
2. 将$l^\mathrm{th}$隐藏层（$l=1,\ldots,L$）的隐状态设为$\mathbf{H}_t^{(l)}  \in \mathbb{R}^{n \times h}$ （隐藏单元数：$h$） 
3. 输出层变量为$\mathbf{O}_t \in \mathbb{R}^{n \times q}$ （输出数：$q$）。
4. 设置$\mathbf{H}_t^{(0)} = \mathbf{X}_t$，第$l$个隐藏层的隐状态使用激活函数$\phi_l$，则：

$$\mathbf{H}\_t^{(l)} = \phi_l(\mathbf{H}\_t^{(l-1)} \mathbf{W}\_{xh}^{(l)} + \mathbf{H}\_{t-1}^{(l)} \mathbf{W}_{hh}^{(l)}  + \mathbf{b}_h^{(l)}),$$

其中，权重$\mathbf{W}\_{xh}^{(l)} \in \mathbb{R}^{h \times h}$，
$\mathbf{W}_{hh}^{(l)} \in \mathbb{R}^{h \times h}$和
偏置$\mathbf{b}_h^{(l)} \in \mathbb{R}^{1 \times h}$
都是第$l$个隐藏层的模型参数。

5. 输出层的计算仅基于第$l$个隐藏层最终的隐状态：

$$\mathbf{O}\_t = \mathbf{H}\_t^{(L)} \mathbf{W}\_{hq} + \mathbf{b}_q$$

其中，权重$\mathbf{W}_{hq} \in \mathbb{R}^{h \times q}$和偏置$\mathbf{b}_q \in \mathbb{R}^{1 \times q}$都是输出层的模型参数。

与多层感知机一样，隐藏层数目$L$和隐藏单元数目$h$都是超参数。

# Bi-RNN

在序列学习中，我们以往假设的目标是：
在给定观测的情况下对下一个输出进行建模。
例如，在时间序列的上下文中或在语言模型的上下文中。

在实际应用场景中，下文对上文的启示作用同等重要。

> 隐马尔可夫模型中的动态规划

在任意时间步$t$，假设存在某个隐变量$h_t$，
通过概率$P(x\_t \mid h\_t)$控制我们观测到的$x_t$。
此外，任何$h\_t \to h\_{t+1}$转移
都是由一些状态转移概率$P(h\_{t+1} \mid h\_{t})$给出。
这个概率图模型就是一个*隐马尔可夫模型*（hidden Markov model，HMM），

因此，对于有$T$个观测值的序列，在观测状态和隐状态上具有以下联合概率分布：

$$P(x_1, \ldots, x_T, h_1, \ldots, h_T) = \prod_{t=1}^T P(h_t \mid h_{t-1}) P(x_t \mid h_t), \text{ where } P(h_1 \mid h_0) = P(h_1).$$

现在，假设我们观测到所有的$x\_i$，除了$x\_j$，
并且我们的目标是计算$P(x_j \mid x_{-j})$。

其中$x_{-j} = (x_1, \ldots, x_{j-1}, x_{j+1}, \ldots, x_{T})$。

由于$P(x_j \mid x_{-j})$中没有隐变量，
因此我们考虑对$h_1, \ldots, h_T$选择构成的所有可能的组合进行求和。
如果任何$h_i$可以接受$k$个不同的值（有限的状态数），这意味着我们需要对$k^T$个项求和，

这个任务显然难于登天。幸运的是，有个巧妙的解决方案：*动态规划*（dynamic programming）。

要了解动态规划的工作方式，我们考虑对隐变量$h_1, \ldots, h_T$的依次求和。

$$\begin{aligned}
    &P(x_1, \ldots, x_T) \\\\
    =& \sum\_{h_1, \ldots, h\_T} P(x_1, \ldots, x_T, h_1, \ldots, h_T) \\\\
    =& \sum\_{h_1, \ldots, h\_T} \prod_{t=1}^T P(h_t \mid h_{t-1}) P(x_t \mid h_t) \\\\
    =& \sum\_{h_2, \ldots, h\_T} \underbrace{\left[\sum\_{h_1} P(h\_1) P(x\_1 \mid h\_1) P(h\_2 \mid h\_1)\right]}\_{\pi_2(h_2) \stackrel{\mathrm{def}}{=}}
    P(x_2 \mid h_2) \prod\_{t=3}^T P(h\_t \mid h\_{t-1}) P(x_t \mid h_t) \\\\
    =& \sum_{h_3, \ldots, h_T} \underbrace{\left[\sum\_{h_2} \pi\_2(h\_2) P(x\_2 \mid h\_2) P(h\_3 \mid h\_2)\right]}\_{\pi_3(h_3)\stackrel{\mathrm{def}}{=}}
    P(x_3 \mid h_3) \prod\_{t=4}^T P(h\_t \mid h\_{t-1}) P(x_t \mid h_t)\\\\
    =& \dots \\\\
    =& \sum_{h_T} \pi_T(h_T) P(x_T \mid h_T).
\end{aligned}$$

通常，我们将*前向递归*（forward recursion）写为：

$$\pi_{t+1}(h_{t+1}) = \sum_{h_t} \pi_t(h_t) P(x_t \mid h_t) P(h_{t+1} \mid h_t).$$

递归被初始化为$\pi_1(h_1) = P(h_1)$。
符号简化，也可以写成$\pi_{t+1} = f(\pi_t, x_t)$，
其中$f$是一些可学习的函数。
这看起来就像我们在循环神经网络中讨论的隐变量模型中的更新方程。

与前向递归一样，我们也可以使用后向递归对同一组隐变量求和。这将得到：

$$\begin{aligned}
    & P(x\_1, \ldots, x\_T) \\
     =& \sum\_{h\_1, \ldots, h\_T} P(x_1, \ldots, x\_T, h\_1, \ldots, h\_T) \\\\
    =& \sum\_{h\_1, \ldots, h\_T} \prod\_{t=1}^{T-1} P(h\_t \mid h\_{t-1}) P(x_t \mid h_t) \cdot P(h\_T \mid h\_{T-1}) P(x\_T \mid h\_T) \\\\
    =& \sum\_{h\_1, \ldots, h\_{T-1}} \prod\_{t=1}^{T-1} P(h\_t \mid h\_{t-1}) P(x_t \mid h_t) \cdot
    \underbrace{\left[\sum\_{h\_T} P(h\_T \mid h\_{T-1}) P(x\_T \mid h\_T)\right]}\_{\rho\_{T-1}(h\_{T-1})\stackrel{\mathrm{def}}{=}} \\\\
    =& \sum_{h\_1, \ldots, h_{T-2}} \prod_{t=1}^{T-2} P(h_t \mid h_{t-1}) P(x_t \mid h_t) \cdot
    \underbrace{\left[\sum\_{h\_{T-1}} P(h\_{T-1} \mid h\_{T-2}) P(x\_{T-1} \mid h\_{T-1}) \rho\_{T-1}(h\_{T-1}) \right]}\_{\rho_{T-2}(h_{T-2})\stackrel{\mathrm{def}}{=}} \\\\
    =& \ldots \\\\
    =& \sum\_{h\_1} P(h\_1) P(x\_1 \mid h\_1)\rho\_{1}(h\_{1}).
\end{aligned}$$

因此，我们可以将*后向递归*（backward recursion）写为：

$$\rho_{t-1}(h_{t-1})= \sum_{h_{t}} P(h_{t} \mid h_{t-1}) P(x_{t} \mid h_{t}) \rho_{t}(h_{t}),$$

初始化$\rho_T(h_T) = 1$。
前向和后向递归都允许我们对$T$个隐变量在$\mathcal{O}(kT)$
（线性而不是指数）时间内对$(h_1, \ldots, h_T)$的所有值求和。
这是使用图模型进行概率推理的巨大好处之一。
它也是通用消息传递算法的一个非常特殊的例子。
结合前向和后向递归，我们能够计算

$$P(x_j \mid x_{-j}) \propto \sum_{h_j} \pi_j(h_j) \rho_j(h_j) P(x_j \mid h_j).$$

因为符号简化的需要，后向递归也可以写为$\rho_{t-1} = g(\rho_t, x_t)$，
其中$g$是一个可以学习的函数。同样，这看起来非常像一个更新方程，
只是不像我们在循环神经网络中看到的那样前向运算，而是后向计算。
事实上，知道未来数据何时可用对隐马尔可夫模型是有益的。

> 双向模型

如果我们希望在循环神经网络中拥有一种机制，使之能够提供与隐马尔可夫模型类似的前瞻能力，
我们就需要修改循环神经网络的设计。
只需要增加一个“从最后一个词元开始从后向前运行”的循环神经网络，
而不是只有一个在前向模式下“从第一个词元开始运行”的循环神经网络。
*双向循环神经网络*（bidirectional RNNs）
添加了反向传递信息的隐藏层。

事实上，这与隐马尔可夫模型中的动态规划的前向和后向递归没有太大区别。
其主要区别是，在隐马尔可夫模型中的方程具有特定的统计意义。

双向循环神经网络没有这样容易理解的解释，
我们只能把它们当作通用的、可学习的函数。

> 定义

对于任意时间步$t$，给定一个小批量的输入数据
$\mathbf{X}\_t \in \mathbb{R}^{n \times d}$
（样本数$n$，每个示例中的输入数$d$），并且令隐藏层激活函数为$\phi$。
在双向架构中，我们设该时间步的前向和反向隐状态分别为
$\overrightarrow{\mathbf{H}}_t  \in \mathbb{R}^{n \times h}$和
$\overleftarrow{\mathbf{H}}_t  \in \mathbb{R}^{n \times h}$，
其中$h$是隐藏单元的数目。
前向和反向隐状态的更新如下：

$$
\begin{aligned}
\overrightarrow{\mathbf{H}}\_t &= \phi(\mathbf{X}\_t \mathbf{W}\_{xh}^{(f)} + \overrightarrow{\mathbf{H}}\_{t-1} \mathbf{W}\_{hh}^{(f)}  + \mathbf{b}\_h^{(f)}),\\\\
\overleftarrow{\mathbf{H}}\_t &= \phi(\mathbf{X}\_t \mathbf{W}\_{xh}^{(b)} + \overleftarrow{\mathbf{H}}\_{t+1} \mathbf{W}\_{hh}^{(b)}  + \mathbf{b}\_h^{(b)}),
\end{aligned}
$$

其中，权重$\mathbf{W}\_{xh}^{(f)} \in \mathbb{R}^{d \times h}, \mathbf{W}\_{hh}^{(f)} \in \mathbb{R}^{h \times h}, \mathbf{W}\_{xh}^{(b)} \in \mathbb{R}^{d \times h}, \mathbf{W}_{hh}^{(b)} \in \mathbb{R}^{h \times h}$
和偏置$\mathbf{b}_h^{(f)} \in \mathbb{R}^{1 \times h}, \mathbf{b}_h^{(b)} \in \mathbb{R}^{1 \times h}$都是模型参数。

接下来，将前向隐状态$\overrightarrow{\mathbf{H}}_t$
和反向隐状态$\overleftarrow{\mathbf{H}}_t$连接起来，
获得需要送入输出层的隐状态$\mathbf{H}_t \in \mathbb{R}^{n \times 2h}$。
在具有多个隐藏层的深度双向循环神经网络中，
该信息作为输入传递到下一个双向层。
最后，输出层计算得到的输出为
$\mathbf{O}\_t \in \mathbb{R}^{n \times q}$（$q$是输出单元的数目）：

$$\mathbf{O}\_t = \mathbf{H}\_t \mathbf{W}\_{hq} + \mathbf{b}\_q.$$

这里，权重矩阵$\mathbf{W}_{hq} \in \mathbb{R}^{2h \times q}$
和偏置$\mathbf{b}_q \in \mathbb{R}^{1 \times q}$
是输出层的模型参数。

> 模型的计算代价及其应用

双向循环神经网络的一个优点是：使用来自序列两端的信息来估计输出。

**问题：**
1. 在训练期间，我们能够利用过去和未来的数据来估计现在空缺的词；而在测试期间，我们只有过去的数据，因此精度将会很差。
2. 双向循环神经网络的计算速度非常慢。其主要原因是网络的前向传播需要在双向层中进行前向和后向递归，
并且网络的反向传播还依赖于前向传播的结果。因此，梯度求解将有一个非常长的链。
3. 双向层的使用在实践中非常少，并且仅仅应用于部分场合。例如，填充缺失的单词、词元注释（例如，用于命名实体识别）

# LSTM

<center>
<img src="../img/lstm.png" width="420px">
</center>

> 输入门、忘记门和输出门

输入：当前时间步的输入和前一个时间步的隐状态作为数据送入`LSTM`的门中。

输入门、忘记门和输出门由三个具有`sigmoid`激活函数的全连接层构成，
以计算输入门、遗忘门和输出门的值。因此，这三个门的值域都在$(0, 1)$内。

**数学表达：**
假设有$h$个隐藏单元，批量大小为$n$，输入数为$d$。
因此，输入为$\mathbf{X}\_t \in \mathbb{R}^{n \times d}$，
前一时间步的隐状态为$\mathbf{H}\_{t-1} \in \mathbb{R}^{n \times h}$。
相应地，时间步$t$的门被定义如下：
输入门是$\mathbf{I}\_t \in \mathbb{R}^{n \times h}$，
遗忘门是$\mathbf{F}\_t \in \mathbb{R}^{n \times h}$，
输出门是$\mathbf{O}\_t \in \mathbb{R}^{n \times h}$。
它们的计算方法如下：

$$
\begin{aligned}
\mathbf{I}\_t &= \sigma(\mathbf{X}\_t \mathbf{W}\_{xi} + \mathbf{H}\_{t-1} \mathbf{W}\_{hi} + \mathbf{b}\_i),\\\\
\mathbf{F}\_t &= \sigma(\mathbf{X}\_t \mathbf{W}\_{xf} + \mathbf{H}\_{t-1} \mathbf{W}\_{hf} + \mathbf{b}\_f),\\\\
\mathbf{O}\_t &= \sigma(\mathbf{X}\_t \mathbf{W}\_{xo} + \mathbf{H}\_{t-1} \mathbf{W}\_{ho} + \mathbf{b}_o),
\end{aligned}
$$

其中$\mathbf{W}\_{xi}, \mathbf{W}\_{xf}, \mathbf{W}\_{xo} \in \mathbb{R}^{d \times h}$
和$\mathbf{W}\_{hi}, \mathbf{W}\_{hf}, \mathbf{W}\_{ho} \in \mathbb{R}^{h \times h}$是权重参数，
$\mathbf{b}_i, \mathbf{b}_f, \mathbf{b}_o \in \mathbb{R}^{1 \times h}$是偏置参数。

> 候选记忆元

*候选记忆元*（candidate memory cell）
$\tilde{\mathbf{C}}\_t \in \mathbb{R}^{n \times h}$。
与上面描述的三个门的计算类似，
但是使用$\tanh$函数作为激活函数，函数的值范围为$(-1, 1)$。
在时间步$t$处的方程：

$$\tilde{\mathbf{C}}\_t = \text{tanh}(\mathbf{X}\_t \mathbf{W}\_{xc} + \mathbf{H}\_{t-1} \mathbf{W}\_{hc} + \mathbf{b}_c),$$

其中$\mathbf{W}\_{xc} \in \mathbb{R}^{d \times h}$和
$\mathbf{W}\_{hc} \in \mathbb{R}^{h \times h}$是权重参数，
$\mathbf{b}\_c \in \mathbb{R}^{1 \times h}$是偏置参数。

> 记忆元

在门控循环单元中，有一种机制来控制输入和遗忘（或跳过）。
类似地，在长短期记忆网络中，也有两个门用于这样的目的：
输入门$\mathbf{I}\_t$控制采用多少来自$\tilde{\mathbf{C}}\_t$的新数据，
而遗忘门$\mathbf{F}\_t$控制保留多少过去的
记忆元$\mathbf{C}\_{t-1} \in \mathbb{R}^{n \times h}$的内容。
使用按元素乘法，得出：

$$\mathbf{C}\_t = \mathbf{F}\_t \odot \mathbf{C}\_{t-1} + \mathbf{I}\_t \odot \tilde{\mathbf{C}}_t.$$

如果遗忘门始终为$1$且输入门始终为$0$，
则过去的记忆元$\mathbf{C}\_{t-1}$
将随时间被保存并传递到当前时间步。
引入这种设计是为了缓解梯度消失问题，
并更好地捕获序列中的长距离依赖关系。

> 隐状态

最后，我们需要定义如何计算隐状态
$\mathbf{H}\_t \in \mathbb{R}^{n \times h}$，
这就是输出门发挥作用的地方。
在长短期记忆网络中，它仅仅是记忆元的$\tanh$的门控版本。
这就确保了$\mathbf{H}\_t$的值始终在区间$(-1, 1)$内：

$$\mathbf{H}\_t = \mathbf{O}\_t \odot \tanh(\mathbf{C}\_t).$$

只要输出门接近$1$，我们就能够有效地将所有记忆信息传递给预测部分，
而对于输出门接近$0$，我们只保留记忆元内的所有信息，而不需要更新隐状态。

# GRU

> 考虑以下情况：

1. 早期观测值对预测所有未来观测值具有非常重要的意义。
因此我们希望有某些机制能够在一个记忆元里存储重要的早期信息。
如果没有这样的机制，我们将不得不给这个观测值指定一个非常大的梯度，
因为它会影响所有后续的观测值。
2. 有一些机制来*跳过*一些无意义的词元。

> 门控循环单元

与普通的循环神经网络之间的关键区别在于，门控循环单元支持隐状态的门控。
意味着模型有专门的机制来确定应该何时更新隐状态，以及应该何时重置隐状态。

<center>
<img src="../img/gru.png" width="520px">
</center>

> 重置门（reset gate）和更新门（update gate）

**输入**: 由当前时间步的输入和前一时间步的隐状态给出。
两个门的输出是由使用sigmoid激活函数的两个全连接层给出。

**数学表达如下**：

对于给定的时间步$t$，假设输入是一个小批量
$\mathbf{X}\_t \in \mathbb{R}^{n \times d}$
（样本个数$n$，输入个数$d$），
上一个时间步的隐状态是
$\mathbf{H}\_{t-1} \in \mathbb{R}^{n \times h}$
（隐藏单元个数$h$）。
那么，重置门$\mathbf{R}_t \in \mathbb{R}^{n \times h}$和
更新门$\mathbf{Z}_t \in \mathbb{R}^{n \times h}$的计算如下所示：

$$
\begin{aligned}
\mathbf{R}\_t = \sigma(\mathbf{X}\_t \mathbf{W}\_{xr} + \mathbf{H}\_{t-1} \mathbf{W}\_{hr} + \mathbf{b}\_r),\\\\
\mathbf{Z}\_t = \sigma(\mathbf{X}\_t \mathbf{W}\_{xz} + \mathbf{H}\_{t-1} \mathbf{W}\_{hz} + \mathbf{b}\_z),
\end{aligned}
$$

其中$\mathbf{W}\_{xr}, \mathbf{W}\_{xz} \in \mathbb{R}^{d \times h}$
和$\mathbf{W}\_{hr}, \mathbf{W}\_{hz} \in \mathbb{R}^{h \times h}$是权重参数，
$\mathbf{b}\_r, \mathbf{b}\_z \in \mathbb{R}^{1 \times h}$是偏置参数。
$\sigma$(sigmoid函数)将输入值转换到区间$(0, 1)$。

> 候选隐状态

接下来，让我们将重置门$\mathbf{R}_t$
与中的常规隐状态集成，得到时间步$t$的*候选隐状态*（candidate hidden state）
$\tilde{\mathbf{H}}_t \in \mathbb{R}^{n \times h}$。

$$\tilde{\mathbf{H}}\_t = \tanh(\mathbf{X}\_t \mathbf{W}\_{xh} + \left(\mathbf{R}\_t \odot \mathbf{H}\_{t-1}\right) \mathbf{W}\_{hh} + \mathbf{b}\_h),$$

其中$\mathbf{W}\_{xh} \in \mathbb{R}^{d \times h}$
和$\mathbf{W}\_{hh} \in \mathbb{R}^{h \times h}$是权重参数，
$\mathbf{b}\_h \in \mathbb{R}^{1 \times h}$是偏置项，
符号$\odot$是`Hadamard`积（按元素乘积）运算符。
在这里，我们使用tanh非线性激活函数来确保候选隐状态中的值保持在区间$(-1, 1)$中。

与`rnn`相比`gru`中的$\mathbf{R}\_t$和$\mathbf{H}\_{t-1}$
的元素相乘可以减少以往状态的影响。
- 每当重置门$\mathbf{R}_t$中的项接近$1$时，$\mathbf{R}\_t$就接近`rnn`隐藏层。
- 对于重置门$\mathbf{R}_t$中所有接近$0$的项，候选隐状态是以$\mathbf{X}_t$作为输入的多层感知机的结果。

> 隐状态

结合更新门$\mathbf{Z}\_t$的效果。这一步确定新的隐状态
$\mathbf{H}\_t \in \mathbb{R}^{n \times h}$
在多大程度上来自旧的状态$\mathbf{H}\_{t-1}$和新的候选状态$\tilde{\mathbf{H}}\_t$。
更新门$\mathbf{Z}\_t$仅需要在
$\mathbf{H}\_{t-1}$和$\tilde{\mathbf{H}}\_t$
之间进行按元素的凸组合就可以实现这个目标。

这就得出了门控循环单元的最终更新公式：

$$\mathbf{H}\_t = \mathbf{Z}\_t \odot \mathbf{H}\_{t-1}  + (1 - \mathbf{Z}\_t) \odot \tilde{\mathbf{H}}\_t.$$

1. 每当更新门$\mathbf{Z}_t$接近$1$时，模型就倾向只保留旧状态，来自$\mathbf{X}_t$的信息基本上被忽略，
2. 当$\mathbf{Z}_t$接近$0$时，新的隐状态$\mathbf{H}_t$就会接近候选隐状态$\tilde{\mathbf{H}}_t$。

> 小结

* 门控循环神经网络可以更好地捕获时间步距离很长的序列上的依赖关系。
* 重置门有助于捕获序列中的短期依赖关系。
* 更新门有助于捕获序列中的长期依赖关系。
* 重置门打开时，门控循环单元包含基本循环神经网络；更新门打开时，门控循环单元可以跳过子序列。

-----------------------------------------------------

<center>
<h6>==========================更新分割线==========================</h6>
</center>


-----------------------------------------------------

# Attention

# Encoder-Decoder

`机器翻译等文本生成任务`中的输入序列和输出序列都是长度可变的

<center>
   <img src="../img/encoder-decoder.jpeg" width="480px">
   <p>encoder-decoder: 两个循环神经网络的编码器和解码器</p>
</center>

> Encoder

1. 指定长度可变的序列作为编码器的输入`X`。
2. 输出编码器的输出状态。

> Decoder

1. 接收编码器的输出状态。
2. 输入序列的有效长度。
3. 前一时间步生成的词元。

# Seq2Seq



# Transforms

# Bert



# Sequence Labeling

<center>
   <img src="../img/model_address_bigru.png" width="780px">
   <p>BiGRU - Graph</p>
</center>

## HMM(Hidden Markov Model)

$$
p(x, y) = p(y_1|start)\prod_{l=1}^{L-1} p(y_{l+1}|y_l)p(end|y_l)\prod_{l=1}^{L}p(x_l, y_l)
$$

<img src="../img/hmm_01.png" width="480px">

HMM 局限性，X,Y独立：

$$p(x, \hat(y)) > p(x, y)$$

- 适合训练集较小的情况
- CRF，可以改善

> Viterbi algorithm $(O(L|S|^2))$

## CRF(Conditional Random Field)

$$P(x, y) \propto exp(w \dot \phi (x, y)) \tag{0}$$

$$P(y|x) = \frac{P(x, y)}{\sum_{y'}P(x, y')} \tag{1}$$

> CRF: Training Criterion

1. Given training data: $\{ (x^1, \hat{y}^1), (x^2, \hat{y}^2), ... , (x^N, \hat{y}^N), \}$
2. Find the weight vector $w^*$:

$$w^* = arg\ max_{w}O(w) \tag{2}$$

$$O(w) = \sum_{n=1}^N logP(\hat{y}^n|x^n) \tag{3}$$

$$logP(\hat{y}^n|x^n) = logP(x^n, \hat{y}^n) - log\sum_{y'}P(x^n, y') \tag{4}$$

- CRF: increase $P(x, \hat{y})$, decrease $P(x, y')$ 
- CRF more likely achieve: $(x, \hat{y}): P(x, \hat{y}) > P(x, y)$

> Structured perceptron

1. Evaluation: $F(x, y) = w\dot \phi(x, y)$
1. Inference: $\tilde{y} = arg\ max_{y\in \mathbb{Y}} w \dot \phi(x, y)$

# Word2Vec

- [Chinese Word Vectors 中文词向量](https://github.com/Embedding/Chinese-Word-Vectors)

# Embedding

---

# Attention

---

# Transformer

<center>
<img src="../img/Transform_model_arch.png" width="320px">
</center>

> Scaled Dot-Product Attention

<center>
<img src="../img/Transform_multi_head.png" width="420px">
</center>

$$
Attention(Q,\ K,\ V)\ =\ softmax(\frac{QK^T}{\sqrt{d_{k}}})V
$$

> FFN

- 基于位置的前馈网络对序列中的所有位置的表示进行变换时使用的是同一个多层感知机（MLP）

```python
class PositionWiseFFN(nn.Module):
    """基于位置的前馈网络"""
    def __init__(self, ffn_num_input, ffn_num_hiddens, ffn_num_outputs,
                 **kwargs):
        super(PositionWiseFFN, self).__init__(**kwargs)
        self.dense1 = nn.Linear(ffn_num_input, ffn_num_hiddens)
        self.relu = nn.ReLU()
        self.dense2 = nn.Linear(ffn_num_hiddens, ffn_num_outputs)

    def forward(self, X):
        return self.dense2(self.relu(self.dense1(X)))
```

> LayerNorm

```python
ln = nn.LayerNorm(2)
```

> Multi-Head Attention

---
# BERT

> NLP 里的迁移学习

1. 使用预训练的模型来抽取词、句子的特征
    - Word2Vec或语言模型
2. 不更新预训练好的模型
3. 需要构建新的网络来抓取新任务需要的信息
    - Word2Vec忽略了时序信息，语言模型只看了一个方向

> Bert 的动机

1. 基于微调的NLP
2. 预训练的模型抽取了足够多的信息
3. 新的任务只需要增加一个简单的输出层

> Bert 架构

1. 只有编码器的Transformer
2. 两个版本：
   1. $BERT_{BASE}$: L($blocks$)=12, H($hidden\ size$)=768, A($heads$)=12, Total Param-
eters=110M
   2. $BERT_{LARGE}$: L=24, H=1024,
A=16, Total Parameters=340M.
   
3. 在大规模数据上训练 > 3B 词

> 对输入的修改

1. 每个样本是一个句子对
   1. Transformer: ($encode(source\ sentences)\ \to\ decode(target\ sentences)$)
   2. Bert: 只有一个$Encode$
      1. $\<cls\>$: classification
      2. $\<sep\>$: separate
      3. $\<pad\>$: padding
   3. Position Embedding: 位置编码，需要模型学习。
   3. Segment Embedding: 句子编码，`[0, 1]`。
   3. Token Embedding: 

<img src="../img/BERT_input_representation.png">
<center>BERT input representation</center>

> 预训练任务一：带掩码的语言模型

1. Transformer的编码器是双向的，标准语言模型要求单向。预测下一个时点的词。
2. 带掩码的语言模型每次随机（$15\\%$）将一些词元换成`<mask>`。类似完形填空。
3. 因为微调任务中不出现`<mask>`
   1. $80\\%$概率，将选中的词元`<mask>`
   2. $10\\%$概率，换成下一个随机词元
   3. $10\\%$概率，保持原有的词元

> 预测训练任务二：下一句子预测

1. 预训练一个句子对中的两个句子是否是相邻的
2. 预训练样本中：
   1. 50% 概率选择相邻的句子对: $\<cls\>\ this\ movie\ is\ grate\ \<sep\>\ i\ like\ it\ \<sep\>$
   1. 50% 概率选择随机的句子对: $\<cls\>\ this\ movie\ is\ grate\ \<sep\>\ hello\ word\ \<sep\>$
3. 将 `<cls>` 对应的输出放到一个全连接层来进行预测

> Bert预训练与迁移

<img src="../img/bert_pretrain_finetuning.png">

> 总结

1. `Bert` 针对微调设计
2. 基于 `Transformer` 的编码器进行了如下修改
   1. 模型更大、数据更多
   2. 输入句子对，片段嵌入，可学习的位置编码
   3. 训练两个任务：（1）带掩码的语言模型；（2）下一个句子的预测。

# 语义解析

1. 指代消解、共指消解(Coreference Resolution)：是自然语言处理(nlp)中的一个基本任务，目的在于自动识别表示同一个实体的名词短语或代词，并将他们归类。
2. 组成关系、成分分析(Constituency Parsing)：
3. 依存句法分析(Dependency Parsing)：简称依存分析，作用是识别句子中词汇与词汇之间的相互依存关系。

- [web](https://www.youtube.com/c/HungyiLeeNTU)

# Appendix

## Dataset

### 阅读理解

|  数据集名称   | 简介 | 调用方法 |
|  ----  | ----- | ------ |
|  [SQuAD](https://rajpurkar.github.io/SQuAD-explorer/) | 斯坦福问答数据集，包括SQuAD1.1和SQuAD2.0|`paddlenlp.datasets.load_dataset('squad')` |
|  [DuReader-yesno](https://aistudio.baidu.com/aistudio/competition/detail/49) | 千言数据集：阅读理解，判断答案极性|`paddlenlp.datasets.load_dataset('dureader_yesno')` |
|  [DuReader-robust](https://aistudio.baidu.com/aistudio/competition/detail/49) | 千言数据集：阅读理解，答案原文抽取|`paddlenlp.datasets.load_dataset('dureader_robust')` |
|  [CMRC2018](http://hfl-rc.com/cmrc2018/) | 第二届“讯飞杯”中文机器阅读理解评测数据集|`paddlenlp.datasets.load_dataset('cmrc2018')` |
|  [DRCD](https://github.com/DRCKnowledgeTeam/DRCD) | 台達閱讀理解資料集|`paddlenlp.datasets.load_dataset('drcd')` |
|  [TriviaQA](http://nlp.cs.washington.edu/triviaqa/) | Washington大学问答数据集|`paddlenlp.datasets.load_dataset('triviaqa')` |
|  [C3](https://dataset.org/c3/) | 阅读理解单选题 |`paddlenlp.datasets.load_dataset('c3')` |


### 文本分类

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [CoLA](https://nyu-mll.github.io/CoLA/) | 单句分类任务，二分类，判断句子是否合法| `paddlenlp.datasets.load_dataset('glue','cola')`|
|  [SST-2](https://nlp.stanford.edu/sentiment/index.html) | 单句分类任务，二分类，判断句子情感极性| `paddlenlp.datasets.load_dataset('glue','sst-2')`|
|  [MRPC](https://microsoft.com/en-us/download/details.aspx?id=52398) | 句对匹配任务，二分类，判断句子对是否是相同意思| `paddlenlp.datasets.load_dataset('glue','mrpc')`|
|  [STSB](http://ixa2.si.ehu.es/stswiki/index.php/STSbenchmark) | 计算句子对相似性，分数为1~5| `paddlenlp.datasets.load_dataset('glue','sts-b')`|
|  [QQP](https://data.quora.com/First-Quora-Dataset-Release-Question-Pairs) | 判定句子对是否等效，等效、不等效两种情况，二分类任务| `paddlenlp.datasets.load_dataset('glue','qqp')`|
|  [MNLI](http://www.nyu.edu/projects/bowman/multinli/) | 句子对，一个前提，一个是假设。前提和假设的关系有三种情况：蕴含（entailment），矛盾（contradiction），中立（neutral）。句子对三分类问题| `paddlenlp.datasets.load_dataset('glue','mnli')`|
|  [QNLI](https://rajpurkar.github.io/SQuAD-explorer/) | 判断问题（question）和句子（sentence）是否蕴含，蕴含和不蕴含，二分类| `paddlenlp.datasets.load_dataset('glue','qnli')`|
|  [RTE](https://aclweb.org/aclwiki/Recognizing_Textual_Entailment) | 判断句对是否蕴含，句子1和句子2是否互为蕴含，二分类任务| `paddlenlp.datasets.load_dataset('glue','rte')`|
|  [WNLI](https://cs.nyu.edu/faculty/davise/papers/WinogradSchemas/WS.html) | 判断句子对是否相关，相关或不相关，二分类任务| `paddlenlp.datasets.load_dataset('glue','wnli')`|
|  [LCQMC](http://icrc.hitsz.edu.cn/Article/show/171.html) | A Large-scale Chinese Question Matching Corpus 语义匹配数据集| `paddlenlp.datasets.load_dataset('lcqmc')`|
|  [ChnSentiCorp](https://github.com/SophonPlus/ChineseNlpCorpus/blob/master/datasets/ChnSentiCorp_htl_all/intro.ipynb) | 中文评论情感分析语料| `paddlenlp.datasets.load_dataset('chnsenticorp')`|
|  [COTE-DP](https://aistudio.baidu.com/aistudio/competition/detail/50/?isFromLuge=1) | 中文观点抽取语料  | `paddlenlp.datasets.load_dataset('cote', 'dp')`|
|  [SE-ABSA16_PHNS](https://aistudio.baidu.com/aistudio/competition/detail/50/?isFromLuge=1) | 中文评价对象级情感分析语料| `paddlenlp.datasets.load_dataset('seabsa16', 'phns')`|
|  [AFQMC](https://github.com/CLUEbenchmark/CLUE) | 蚂蚁金融语义相似度数据集，1表示句子1和句子2的含义类似，0表示含义不同| `paddlenlp.datasets.load_dataset('clue', 'afqmc')`|
|  [TNEWS](https://github.com/CLUEbenchmark/CLUE) | 今日头条中文新闻（短文本）分类，共15类| `paddlenlp.datasets.load_dataset('clue', 'tnews')`|
|  [IFLYTEK](https://github.com/CLUEbenchmark/CLUE) | 长文本分类，共119个类别| `paddlenlp.datasets.load_dataset('clue', 'iflytek')`|
|  [OCNLI](https://github.com/cluebenchmark/OCNLI) | 原生中文自然语言推理数据集，句子对三分类问题| `paddlenlp.datasets.load_dataset('clue', 'ocnli')`|
|  [CMNLI ](https://github.com/CLUEbenchmark/CLUE) | 中文语言推理任务，判断sentence1和sentence2的关系：蕴含（entailment），矛盾（contradiction），中立（neutral）。句子对三分类问题 | `paddlenlp.datasets.load_dataset('clue', 'cmnli')`|
|  [CLUEWSC2020](https://github.com/CLUEbenchmark/CLUE) | WSC Winograd模式挑战中文版，代词消歧任务，二分类任务| `paddlenlp.datasets.load_dataset('clue', 'cluewsc2020')`|
|  [CSL](https://github.com/P01son6415/CSL) | 论文关键词识别，判断关键词是否全部为真实关键词，二分类任务 | `paddlenlp.datasets.load_dataset('clue', 'csl')`|
|  [EPRSTMT](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的电商产品评论情感分析数据集，Positive、Negative 情感 2 分类任务| `paddlenlp.datasets.load_dataset('fewclue', 'eprstmt')`|
|  [CSLDCP](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的中文科学文献学科分类数据集，根据文献的中文摘要判断文献类别，共 67 类别。| `paddlenlp.datasets.load_dataset('fewclue', 'csldcp')`|
|  [TNEWSF](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的今日头条中文新闻（短文本）分类，共15类 | `paddlenlp.datasets.load_dataset('fewclue', 'tnews')`|
|  [IFLYTEK](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的长文本分类任务，共 119 个类别 | `paddlenlp.datasets.load_dataset('fewclue', 'iflytek')`|
|  [OCNLIF](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的中文自然语言推理数据集，句子对三分类问题 | `paddlenlp.datasets.load_dataset('fewclue', 'ocnli')`|
|  [BUSTM](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中对话短文本语义匹配数据集, 2 分类任务 | `paddlenlp.datasets.load_dataset('fewclue', ‘bustm')`|
|  [CHIDF](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的成语阅读理解填空, 根据文本内容从候选 7 个成语中预测正确的成语 | `paddlenlp.datasets.load_dataset('fewclue', 'chid')`|
|  [CSLF](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的论文关键词识别，判断关键词是否全部为真实关键词，二分类任务 | `paddlenlp.datasets.load_dataset('fewclue', 'csl')`|
|  [CLUEWSCF](https://github.com/CLUEbenchmark/FewCLUE/tree/main/datasets)  | FewCLUE 评测中的 WSC Winograd 模式挑战中文版，代词消歧任务，二分类任务 | `paddlenlp.datasets.load_dataset('fewclue', 'cluewsc')`|
| [THUCNews](https://github.com/gaussic/text-classification-cnn-rnn#%E6%95%B0%E6%8D%AE%E9%9B%86) |  THUCNews中文新闻类别分类 | `paddlenlp.datasets.load_dataset('thucnews')` |
| [HYP](https://pan.webis.de/semeval19/semeval19-web/) | 英文政治新闻情感分类语料  | `paddlenlp.datasets.load_dataset('hyp')` |
|  [XNLI](https://github.com/facebookresearch/XNLI) | 15种语言自然语言推理数据集，三分类任务. | `paddlenlp.datasets.load_dataset('xnli', 'ar')`|
|  [XNLI_CN](https://github.com/facebookresearch/XNLI) | 中文自然语言推理数据集（XNLI的子集），三分类任务. | `paddlenlp.datasets.load_dataset('xnli_cn')`|

### 文本匹配

|  数据集名称   | 简介 | 调用方法 |
|  ----  | --------- | ------ |
| [CAIL2019-SCM](https://github.com/china-ai-law-challenge/CAIL2019/tree/master/scm) | 相似法律案例匹配  | `paddlenlp.datasets.load_dataset('cail2019_scm')` |

### 序列标注

|  数据集名称   | 简介 | 调用方法 |
|  ----  | --------- | ------ |
|  [MSRA_NER](https://github.com/lemonhu/NER-BERT-pytorch/tree/master/data/msra) | MSRA 命名实体识别数据集| `paddlenlp.datasets.load_dataset('msra_ner')`|
|  [People's Daily](https://github.com/OYE93/Chinese-NLP-Corpus/tree/master/NER/People's%20Daily) | 人民日报命名实体识别数据集| `paddlenlp.datasets.load_dataset('peoples_daily_ner')`|
|  [CoNLL-2002](https://www.aclweb.org/anthology/W02-2024/) | 西班牙语和荷兰语实体识别数据集| `paddlenlp.datasets.load_dataset('conll2002', 'es')`|


### 机器翻译

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [IWSLT15](https://workshop2015.iwslt.org/) | IWSLT'15 English-Vietnamese data 英语-越南语翻译数据集| `paddlenlp.datasets.load_dataset('iwslt15')`|
|  [WMT14ENDE](http://www.statmt.org/wmt14/translation-task.html) | WMT14 EN-DE 经过BPE分词的英语-德语翻译数据集| `paddlenlp.datasets.load_dataset('wmt14ende')`|

### 机器同传

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [BSTC](https://aistudio.baidu.com/aistudio/competition/detail/44/) | 千言数据集：机器同传，包括transcription_translation和asr | `paddlenlp.datasets.load_dataset('bstc', 'asr')`|

### 对话系统

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [DuConv](https://aistudio.baidu.com/aistudio/competition/detail/48/) | 千言数据集：开放域对话，中文知识型对话数据集 | `paddlenlp.datasets.load_dataset('duconv')`|

### 文本生成

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [Poetry](https://github.com/chinese-poetry/chinese-poetry) | 中文诗歌古典文集数据| `paddlenlp.datasets.load_dataset('poetry')`|
|  [Couplet](https://github.com/v-zich/couplet-clean-dataset) | 中文对联数据集| `paddlenlp.datasets.load_dataset('couplet')`|
|  [DuReaderQG](https://github.com/PaddlePaddle/Research/tree/master/NLP/DuReader-Robust-BASELINE) | 基于DuReader的问题生成数据集| `paddlenlp.datasets.load_dataset('dureader_qg')`|
|  [AdvertiseGen](https://github.com/ZhihongShao/Planning-based-Hierarchical-Variational-Model) | 中文文案生成数据集| `paddlenlp.datasets.load_dataset('advertisegen')`|
|  [LCSTS_new](https://aclanthology.org/D15-1229.pdf) | 中文摘要生成数据集| `paddlenlp.datasets.load_dataset('lcsts_new')`|
|  [CNN/Dailymail](https://github.com/abisee/cnn-dailymail) | 英文摘要生成数据集| `paddlenlp.datasets.load_dataset('cnn_dailymail')`|

### 语料库

| 数据集名称  | 简介 | 调用方法 |
| ----  | --------- | ------ |
|  [PTB](http://www.fit.vutbr.cz/~imikolov/rnnlm/) | Penn Treebank Dataset | `paddlenlp.datasets.load_dataset('ptb')`|
|  [Yahoo Answer 100k](https://arxiv.org/pdf/1702.08139.pdf)  | 从Yahoo Answer采样100K| `paddlenlp.datasets.load_dataset('yahoo_answer_100k')`|



