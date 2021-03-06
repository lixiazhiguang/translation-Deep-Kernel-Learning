# 深度核学习
Andrew Cordon Wilson CMU
Zhiting Hu CMU
Ruslan Salakhutdinov University of Toronto
Eric P. Xing CMU

## 摘要
我们引入了可扩展的深度核函数，它结合深度学习架构的结构特性，以及核方法的非参数化灵活性。具体地，我们通过使用局部核插值、诱导点以及充分利用代数结构（Kronecker 和 Toeplitz），将具有深度框架的混合核参数形式的输入，转换为具有可伸缩的核表示形式。这种具有闭式表示的核函数，可以被用来替换标准的核函数，并且在表达能力和可扩展性上具有优越性。我们通过最大化高斯过程的边际可能性，来学习核函数的表示形式。在大小为ｎ的训练集上，推导和学习的复杂度为O(n)；在测试集上，预测每个点的复杂度是O(1)。在大量多样的应用集合上，包括具有200万样本点的数据集，我们通过高斯过程展示了这一灵活且具有独立深度框架的核学习模型所带来的效果提升。

## 介绍
“高斯过程如何替代神经网络？我们是否已经迈出第一步？”MacKay(1998)提出如此疑问。20世纪90年代后期，研究人员对与神经网络相关的许多设计选择（包括结构、激活函数和正则化）以及缺乏有针对性的框架来指导这些选择感到沮丧。
Neal(1996)显示拥有无穷多隐节点的贝叶斯神经网络，可以近似为一种具有特定核函数(协方差)的高斯过程，高斯过程开始受到机器学习社区的广泛关注。高斯过程随后被视为神经网络的灵活和可解释的替代方案，它具有非常直观的学习过程。在神经网络使用有限多高度自适应基函数的情况下，高斯过程通常使用无穷多的固定基函数。
正如MacKay(1998)，Hinton等人(2006)，和Bengio(2009)所说，神经网络可以通过学习多次高度自适应基函数，来发现高维数据的有意义表示。相比之下，具有主流的核函数的高斯过程通常被用作简单的平滑设置。
近期的方法（例如：Yang等人(2015)，Lloyd等人(2014)，Wilson(2014)，Wilson和Adams(2013)）已经证明，人们可以开发出更具有表达能力的核函数，它们的确能在没有人为干预的情况下，发现数据的丰富结构。这些方法有效地使用了无穷多的自适应基函数。于是，相关的问题便不再是那种范式取代另一种（例如：核方法和神经网络），而是我们是否可以结合每种方法的优势。
事实上，深层神经网络为创造自适应核函数提供了一种强大的机制，它具有已经被证明在多种应用领域有效的感应偏差，这些领域包括视觉物体检测，语言感知，语言理解和信息检索（Krizhevsky等人(2012)，Hinton等人(2012)，Socher等人(2011)，Kiros等人(2014)，Xu等人(2015)）。
在本文中，我们讲核方法的非参数灵活性与深层神经网络的结构特性相结合。具体地，我们使用深度前馈全连接网络和卷积网络，结合谱混合协方差函数（Wilson和Adams, 2013），诱导点（Qui˜nonero-Candela和Rasmussen, 2005），利用代数结构（Saatchi, 2011）和局部核插值（Wilson和Nickisch, 2015; Wilson等人, 2015），以创造可扩展的有丰富表达能力的闭式协方差核，用在高斯过程上。作为一种非参数方法，我们模型的信息容量随着可用数据量增长而增长，但其复杂度通过高斯过程的编辑可能性自动校准，而不需要正则化或交叉验证（Rasmussen和Ghahramani, 2001; Rasmussen和Williams, 2006; Wilson, 2014）。非参数层提供的灵活性和自动校准，通常提供了高标准的性能，同时减少了用户大量手动调参的需要。
我们进一步基于KISS-GP（Wilson和Nickisch, 2015）的想法和其扩展（Wilson等人, 2015），使我们的深度核学习模型可以随训练实例的数目现行扩展，而不是像标准的高斯过程一样的Ｏ($ n^3 $)，同时保留完整的非参数表示。我们的方法同时对每个测试数据点按O(1)的复杂度扩展，而不是标准GP的O($n^2$)，这使预测时间非常快。因为KISS-GP从用户指定的核中创建一个近似的核，用于快速计算，而独立于特定的推理过程，我们可以将生成的核视为可扩展的深度核函数。我们在实验结果部分展示了这种可扩展性的价值，其中正是大数据集为我们的模型发现表达性统计表达提供了极佳的机会。
我们首先在第二部分回顾相关的工作，并在第三部分提供高斯过程的背景。在第四节中，我们推导可扩展形式的的闭式深度核，并且说明如何通过高斯过程边际可能性对这些核进行有效的自动学习。

## 相关工作
鉴于结合核和神经网络的直观价值，各种不同形式的这种组合已经在不同的情景中被考虑了，这令人鼓舞。
高斯过程回归网络（Wilson等人, 2012）将贝叶斯网络中的所有权重连接核激活函数用高斯过程替代，允许作者在多个任务上刻画与输入相关的关联性。或者，Damianou和Lawrence(2013)在无监督的设定下，将贝叶斯网络中的每个激活函数用高斯过程替代。虽然有使用前景，两个模型都是针对特定任务的，并且需要复杂的近似贝叶斯推断，这笔标准高斯过程核深度学习模型要求高得多，并且通常不能扩展到超过几千个训练数据点。类似地，Salakhutdinov和Hinton(2008)讲深度信念网络(DBNs)与高斯过程相结合，在半监督的情境下，相对于使用RBF核的标准高斯过程，显示出性能的提升。然而，他们的模型高度依赖于DBNs的非监督预训练，并且GP组件无法扩展超过几千个训练数据点。相似地，Calandra等人(2014)将前馈神经网络变换与高斯过程相结合，表现出学习尖锐的不连续的能力。然而，和其他方法类似，所得到的模型只能扩展到不超过几千个数据点。
在常见的设定中，Yang等人(2014)将卷积网络及其在ImageNet上预训练的参数，与可扩展的Fastfood(Le等人, 2013)使用RBF核的变形形式相结合，将后者应用在前者的最后一层。所得到的结果可扩展并具有灵活性，但网络参数通常要首先和Fastfood特征分开训练，并且由于Fastfood提供的参数扩展，组合模型依然是参数模型。
