# PointNet 论文解读

文章：C. R. Qi, H. Su, K. Mo, and L. J. Guibas. Pointnet: Deep learning on point sets for 3d classification and
segmentation. arXiv preprint arXiv:1612.00593, 2016.  

作者：Charles R. Qi, Hao Su, Kaichun Mo, Leonidas J. Guibas

链接：https://arxiv.org/abs/1612.00593

本文版权声明：copyright@phosphenesvision

参考资料：https://blog.csdn.net/qq_15332903/article/details/80224387

## PointNet 大致思想

1. 点云的两个特征：无序性，刚性变换不变性。详见4.1节。
2. 分别针对于这两个特性：作者提出来了PointNet网络，亮点就是保持了点云的这两个特性。详见4.2.1节，4.2.3节
3. PointNet与之前的网络的区别主要是：以前的网路在处理点云数据的时候，需要将点云数据做一个变换（变换成体素信息），而本文提出了上述两个亮点，可以直接处理点云数据，而不需要将之转换成体素。

## PointNet 详细阅读

#### 0 摘要

1. 点云这种数据格式很有用但是不规则，不规则是一个很大的区别点，不规则也可以理解成无序。
2. 之前的研究主要是把点云数据转换成3D的图像体素信息，但是这种方法会遗漏一些内容，会使数据变得很繁杂，原文用的是voluminous这个单词。
3. 本文提出了一种新的神经网络，PointNet，直接运用点云数据，而且保证了点云数据的操作不变性(permutation invariance)，这种网络高效且有效，可以用在图像分类，局部分割和语义分割领域。![PN-F1](image\PN-F1.png)

#### 1 介绍

1. 回顾以前的工作：在点云检测发展的初期，还是更多的去套用CNN的模板，但是普通的CNN的模板对于输入数据有严格的规定，必须要有相同的形式，相同的大小等等，然而点云数据的特点就是不规则，一个很自然的想法就是把这些不规则的点云变换成规则的3D体素，但是这种做法的缺点就是数据会因此变得更为繁杂，而且丢失一些信息和特性。
2. 点云数据的特征：在对点云数据的处理过程中，要特别注意两个特征，一个是不规则无序性，第二个是操作不变性。本文提出的PointNet网络也尊重了这些特性，整个神经网络的输入是点云图，输出是点云图中每个点的类别标签或者语义标签。
3. 网络的整体结构：我们的网络整体上很简单，在网络的前几层，每一个点都是被单独处理的，然后网络会学习一些感兴趣的点，并且将它们编码，最后几层的全连接层再将这些信息综合起来作为全局特征。每一个点是以一个三维的坐标来表示的，如果有新的特征可以再添加坐标维度。
4. 网络的一些细节：①网络对于刚性或者非刚性的一些物体都能处理的很好；②数据的预处理，训练了一个空间转换网络规范化数据。
5. 对网络的一些分析：本文还对网络进行了理论分析和实验分析，这里暂且列出分析结果，①该网络可以近似于任何连续函数；②该网络可以通过一组稀疏的关键点来进行成功的分类预测；③针对于输入点的小型扰动，对于离群点，或者对于缺失点都具有很强的鲁棒性；④是速度快；⑤是精度高。
6. 本文的贡献：①提出了一种新型的网络PointNet，可以处理无序点云数据；②可以完成分类任务，局部分割任务和语义分割任务；③对于网络进行了理论分析和实验分析；④展示出来了网络学到的特征，对网络给出了具象的解释。

#### 2 相关工作

1. 关于点云的特征：点云两个重要的特征，无序性和不变性。对于一个分类任务或者分割任务来说，找到最优的点云数据特征表示并不简单，以前的许多特征都是针对于某个特定的任务而手工提取的，我们的工作对此做了一个改进。
2. 用深度学习处理3D数据：①Volumetric CNNs：第一个提出来的，但是数据稀疏计算能力不强导致该种方法的像素精确度并不是很好，②FPNN and Vote3D：解决了稀疏性问题，但是还有稀疏性问题，处理起来大的点云依然捉襟见肘，③Multiview CNNs：转化3D的点云图至2D的平面图，然后用之前的2D卷积神经网络去对点云图操作，这种方法在shape classification分类问题和retrieval task检索问题上有很好的效果，但是拓展到scene understanding场景理解，point classification点分类问题和shape completion形状恢复问题上的效果并不尽如人意，④最近的工作，Spectral CNNs：对mesh应用光谱CNN，但是只能用在流形mesh上面比如有机对象，对于一些非等距的物体比如家具就不是很适合，⑤Feature-based DNNs：第一次把3D的数据特征格式转化成一个向量，然后用一个全连接层对其分类，但是我们认为这种方法受限于这些特征数据的表达力度。
3. 用深度学习处理无序数据：首先我们要处理的点云数据集是无序的向量集，这一块大家关注的比较少，这一块举一个例子，Oriol Vinyals用到了一种读写型网络来处理数据，可以把无序的数据进行排序编号，然而这个网络只能处理普通的一般的数据集，用于自然语言处理，而对于点云图这种带有地理信息的数据集而无能为力。

#### 3 问题陈述

首先来描述一下本文的网络的工作流程，网络的输入是未经处理过的点云数据，很多个点组成了一个集合输入到网络中，每个点由特征向量来表示，特征向量中包含该点的xyz坐标，颜色等相关信息，但是一般我们只考虑xyz坐标信息，忽略其他信息。

对于一个分类网络来说，输入就是直接输入这些未经处理的点云数据集，输出是一个k维的score向量，表示该点云集是每一类数据的可能性和概率。对于分割任务，又可以分成局部分割和全景分割，局部分割的意思是对一个物体进行分类，分类出它们的各个部位，而对于全景分割，通常是给出一个场景，分割出该场景中的每一类物体，但是这两种情况的输出都是一样的，输出是一个n * m的矩阵，分别来表示n个点中的m个类别的归属结果。

#### 4 对于点集的深度学习

##### 4.1点集的属性

三个属性：无序性，点间交互性，刚性变换(transformation)不变性，我认为第二个属性和第三个属性其实是可以合并的，刚性变换不变性也是相对于点和周围点之间的属性，也算是点与点的交互性的范围，而刚性变换不变性又是点间交互性最主要的一个特性，因此个人认为两者可以合并。

1. 无序性：这是点云相当重要的一个性质，什么意思呢，借用引用链接的解释，无序性的定义是说，点云本质上是一长串点（nx3矩阵，其中n是点数）。在几何上，点的顺序不影响它在空间中对整体形状的表示，例如图中，相同的点云可以由两个完全不同的矩阵表示，但是再正常的输入之中，如果再惨杂进无序性这个特点，那么数据就会受到影响了，如果一个点云图中有N个点，那么可能产生的输入就有N!种，因此我们希望得到的效果如下图右边：N代表点云个数，D代表每个点的特征维度。不论点云顺序怎样，希望得到相同的特征提取结果。具体的实现策略可以看4.2.1节。![PN-APPENDIX1](image\PN-APPENDIX1.png)

2. 点间交互性：点云图中的点云存在于一个距离测量空间中，在这个空间中，所有的点都不是孤立的，而是可以与周围的点形成一个局部区域，这个局部区域是有意义的，这也就要求我们的网络能够提取出相应的局部特征。

   > 在这里稍微提一下，虽然本文中提到了局部区域，但是点的表示只是用一个三维坐标来表示的，这根本无法如文中所说可以捕捉局部特征，因此本文中提出的PointNet网络并没有很好的提取局部特征的能力，而它的升级版PointNet++则着重于解决这个问题，具体参看PN++的文章。

3. 刚性变换不变性(invariance under transformations)：这一个特性很好理解，就是说对于这些点集对于一些刚性变换来说，比如旋转和平移，虽然点的位置和坐标会发生变化，但是内在特性应该是保持不变性的，比如类别信息和语义信息。具体的实现策略可以参考4.2.3小节。

##### 4.2 网络结构

根据结构图，首先来看网络来分析该网络，然后再来补充细节。

![PN-F2](image\PN-F2.png)

1. 输入是一个n * 3的点集，n代表点的个数，3代表每个点的特征，该网络只用到了坐标信息，因此是xyz三维特征。
2. 第一次输入变换，对空间点云进行调整，旋转出一个更容易用于分类或者分割的角度，比如把物体转到正面，这一步操作就涉及到了点云的第三个特性，刚性变换不变性，具体的操作方法可以参考后面的细节补充。
3. 接下来是一个MLP去对特征进行学习，输出变成n * 64。
4. 第二次输入变换，对提取出的64维特征进行对齐，即在特征层面对点云进行变换，同样涉及到了刚性变换不变性。
5. 接下来仍然是特征提取，输出变成n * 1025。
6. 然后是一个最大池化的操作，提取出点云集全局特征，该操作与点云的无序性相呼应，具体操作方法可参考4.2.1节。
7. 对于分类网络来说，提取完全局特征后，就可以训练一个MLP得到最后的输出了。
8. 对于分割网络来说，全局特征还要与前面的特征进行融合，详细可见4.2.2节，再通过两个MLP的特征提取，才能得到最后的输出结果。
9. 总体来说，该网络的骨架网络包括两次刚性变换，两次特征提取，一次无序性最大池化平衡策略，对于分类网络来说，再进行一次MLP特征提取即可得到输出结果，而对于分割网络来说，还需要一次跳跃结构的辅助和两次MLP特征提取才可以得到最终的结果。



 ###### 4.2.1 针对于点集无序性的平衡策略

在前面已经阐述了无序性的定义，网络结构需要处理这种无序性，也就是无论输入点集中点的顺序如何，输出结果应该是相同的，望楼的这种能力也是处理在原文中本小节阐述了三种相应的方法，分别是①将输入点集按照一个规范的顺序进行排序；②把输入当成一个序列，按照训练RNN的方法来训练网络，但是需要做一个数据增强，提前将所有可能的刚性变换全部考虑到；③设计一个平衡函数，将所有的点通过这个平衡函数结合起来。

本文采取的是第三种算法，因此前两种算法不再详细介绍，值得注意的是平衡函数有很多种，加法运算 + 和乘法运算 * 都可以看作是一个平衡函数，我们设计的平衡函数需要满足的要求是如下这个公式
$$
f ( \{x_1, x_2, ..., x_n\}) \approx g(h(x_1), h(x_2), ..., h(x_n))
$$
本文的想法是用g函数去近似f函数，f函数是网络的原函数，是定义在初始点云集合上的，g函数是是一个平衡函数，是定义在初始点云集合提取的特征上的，h函数是一个多层感知器网络MLP，用于对初始点云集合提取特征。

具象来说，选取的平衡策略函数g，特征提取函数h等等这些函数的组合应该是要逼近网络的预期效果f。

而本文选取的g函数就是一个简单的最大池化层函数，回顾刚才的网络模型，输入进最大池化层的是一个n * 1024的张量，而输出的则是1 * 1024的张量，这便是最大池化层的效果。一个更为形象的图是这样的：![PN-APPENDIX3](image\PN-APPENDIX3.png)。

尽管最大池化的平衡策略看起来很简单，但是理论分析4.3节和实验结果5.2.1节都表明了该函数的有效性和高效性。

###### 4.2.2 分割任务中的跳跃结构

本文的PointNet网络的骨架结构输出是一个经过最大池化的一维向量，这个向量可以被看作是输入点云集的全局特征，对于分类任务来讲，再去训练一个SVM分类器或者MLP分类器即可得到输出结果，而分割任务则不是这么简单，分割任务需要用一个跳跃结构将该全局特侦与之前的局部信息结合起来，再去经过分类器去训练分类的语义值。

这样对于网络的改变从实验结果来看是起到了一个正向的作用。

###### 4.2.3 针对于刚性变换不变性的校准网络

本小节对应的是点云的第三个特征，刚性变换不变性

首先同样是给出了一个对比算法，是在特征提取之前把所有的输入数据集排列到一个规范空间。

但是本文的做法是训练一个小网络T-net，用这个T-net去预测一个放射变换矩阵(affine transformation matrix)，然后让其与原坐标向量进行相乘。这样得到的结果不仅达到了刚性变换的目的，还保持了刚性变换的不变性，虽然这个T-net很小，但是它和大的网络的网络结构相似，由很多基本的小的单元组成，这些小的单元包括独立的点特征提取层，最大池化层和全连接层。

回过头来再看网络的整体结构，我们一共用了两次T-net去保持矩阵的刚性变换不变性，刚才是阐述了作用坐标空间的T-net，也就是第一次用到的T- net，而第二次用到T-net是作用在特征空间，事实上，特征空间同样可以用到T-net去做保持刚性变换的不变性，但是特征空间的维度要比坐标空间多，需要考虑更多的问题，网络优化的难度也会变大，因此需要加一个正则化公式到最终分类层softmax训练的损失上，具体来说是$ L_{reg} = ||I - AA^{T}||_{F}^{2} $，A就是通过T-net预测出来的特征转换矩阵，还可以人为约束特征转换矩阵是一个正交矩阵，这个操作可以避免输入内容的信息损失，这样优化过程就变得更加稳定，我们的网络也有一个更好的性能表现。

对比试验结果在5.2.2节

##### 4.3 理论分析 

本节主要是说明两个定理

1. 逼近定理：这个定理针对的还是4.2.1节的平衡策略，也就是说，这个定理证明了MAXPOOLING函数可以当作是一个平衡函数。定理的内容是说
   $$
   假设f函数是一个定义在Hausdorff \  distance的连续的集函数f: \chi \rightarrow R，\chi是一个点集的定义域，R是一维实数空间值域，\\ 那么有如下结论：\forall \epsilon >0，必然存在一个连续集函数h和一个平衡函数g(x_1, x_2, ..., x_n)=\gamma \circ MAX使得对于任意的S\in \chi,有 \\ \mid f(S) - \gamma(MAX_{x_i \in S}(h(x_i)))\mid < \epsilon \\
   其中S=\{x_1, x_2, ..., x_n\}，\gamma是一个连续函数，MAX是最大池化操作
   $$
   那么首先来理解一下这个定理，定理种的f函数就是整个PointNet网络，h函数就是对于每一个点集中的点做的特征提取函数，MAX对应平衡策略最大池化层的操作，γ  是最后的分类层训练的多层感知机。同时证明了最大池化层的维数越大，提取的信息越为精确。

   > 关于Haudorff Distance
   >
   > Hausdorff距离是一个衡量两个点集之间的距离的一个函数，它的定义是这样的，
   > $$
   > Hausdorff = MAX\{MAX_{x\in X} MIN_{y \in Y}d(x,y), MAX_{y\in Y} MIN_{x \in X}d(x,y)\}
   > $$
   > 具体的操作过程是这样的，首先对于X集合中的所有点xi，分别求出xi到Y集合中的每个点的距离，再将所有的xi取出来，求它们的最小值，对于Y集合同样这样做，最终求它们两个的最大值当成最终的结果。
   >
   > Hausdorff距离的意义是用来衡量两个点集之间任何距离点的最大的不匹配度，对于X集合来说，得到X集合的距离最大值之后，便有，任意X中的点以该半径画圆，至少能有一个Y集合中的点能落在该圆中，也就是说，其实Hausdorff距离不关心是否每个点都落在该圆中，只要有一个点落在这个圆中即可，也印证了两个点集的最大不匹配度。
   
2. 瓶颈定理：这个定理与整个网络的鲁棒性有关，实验测试参考5.2.3节，定理的内容是说
   $$
   假设用u函数来代表对称函数，因此有u=MAX_{x_i \in S}(h(x_i))，函数u定义在 \chi \rightarrow R^K，\chi是一个点集的定义域，R^K是K维实数空间值域，\\ f则可以表示成f = \gamma \circ u,那么有如下两个结论：① \forall S, \exist C_S, N_S \subseteq \chi, f(T) = f(S) ,if C_S \subseteq T \subseteq N_S; ②\mid C_S \mid \leq K.
   $$
   第一个定理表明，对于输入点集S来说，如果发生一定程度的信息丢失，或者一定程度的噪声干扰，也不会影响分类标签和分割标签的预测，只要它不超过信息丢失的临界点Cs(Critical Points)和噪声干扰的临界点Ns(Upper-bound Points)，这也表明了网络的鲁棒性，能够忍受一定程度的信息丢失和噪声干扰；第二个定理表明，Cs中包含的点的个数小于K个，比最大池化层的维数还要低，也就是说网络对于点集S的预测只依赖于几个典型的特征点Cs，同时K被称作f函数的瓶颈维度。

#### 5 实验

##### 5.1 网络的应用

一共可以应用在三个方面有相关的应用：3D object classification, object part segmentation and semantic scene segmentation ，可以理解成点云分类，局部分割和全景分割，前面已经介绍过了。

###### 5.1.1 3D物体分类

3D物体分类：用到了ModelNet40 shape classification数据集，一共有12311个CAD模型，人工分成40个类别，9843的训练集，2468测试集。

在输入数据的时候，我们直接输入原始点云数据，具体操作是在网格面上，根据网格的面积均匀采样1024个点，然后把它们归一化成一个单位球体。在训练过程中，我们也对点云做了数据增强，通过随机沿上轴对物体进行旋转和通过一个平均值为0，标准差为0.02的高斯噪声对每一个点的位置进行抖动。

表一中列出了我们的模型的实验对比图，我们的这个PointNet与之前的网络进行了横向的对比，也与我们自己的baseline网络基本网络做了对比，baseline是对传统特征进行了提。结果显示，不管怎样我们的网络的效果都比较好，而且可以并行在CPU上运行，只有一个例外，就是MVCNN战胜了我们的网络，原因是MUCNN这个网络是基于多视角的方法，我们认为我们的网络不如他们的原因是损失了一些可以被渲染图像捕获的精细几何细节。

![PN-T1](image\PN-T1.png)

###### 5.1.2 3D目标区域分割

3D目标区域分割：这是一个细粒度的很有挑战性的深度学习课题，是说什么意思呢，给定一个物体，比如一把椅子，要把椅子的每一个部分精确的分割出来。

这个任务用到的是ShapeNet part data set，一共有16881个shape，分成16个种类，这16个种类一共有50个part，大部分的shape是由二至五个part组成的，真实标签表在了每一个shape的采样点上面。

我们把这个任务看成了一各点的分类任务，评价标准是每一个点的mIOU。比如说对于一个shape S，它的类别是C，那么我们计算这个S中的每一个part的IOU，再把它平均，这样就得到了这个shape S的mIOU，如果要计算类别C的mIOU，那么就要把每一个类别是C的shape的mIOU进一步平均。

下表是这个实验的对比图，我们的网络在大部分种类上都超过了其他网络，平均提升2.3%。

![PN-T2](image\PN-T2.png)

为了试验鲁棒性，我们还在模拟的Kinect上面进行了实验，对于数据集的每一个shape，我们用Blensor Kinect Simulator重新从6个随机视角生成一个不完全的点云图，然后用同样的网络结构去训练完整的或者不完整的点云图，得到的结果只相差了5.3%的mIOU，下图显示的是在完整点云图和不完整点云图上做分割的结果对比。

![PN-F3](image\PN-F3.png)

###### 5.1.3 语义场景分割

语义场景分割：前面的区域分割可以很容易的拓展到全景分割中，只是把标签从object part labels改成了semantic object classes。

用到的数据集是Stanford 3D semantic parsing dataset，这个数据集是用Matterport 扫描仪扫描出来的一个6个区域271个房间，每一个扫描到的点都标注了1个语义标签，一共13个类别。在训练数据的准备中，首先是把点按照房间进行分类，然后再对一个房间进行采样，采样成1m * 1m的block，然后去预测每一个block的类别，每一个点用一个9维的向量来表示，XYZ，RGB和一个归一化的在房间中处的位置，训练的时候是采样了每一个block中的4096个点，测试的时候是测试了一个block的所有的点，而且用到了k-fold的训练策略。

首先用我们的网络和我们的baseline进行比较，我们的baseline除了使用原来的9维的向量数据外，还包括另外的3维数据，local point density, local curvature and normal  然后用了一个MLP对其进行分类，结果再下表中，两者性能的差别较大，下图是一个结果的可视化。

![PN-T3](image\PN-T3.png)

![PN-F4](image\PN-F4.png)

在这个基础之上，我们又建了一个3D目标检测系统，对比算法是基于滑动形状算法，后处理中用到了CRF，用了一个SVM分类器，训练数据是体素格子中的局部几何特征和全局房间纹理特征。对比看到，我们的算法大幅度领先。

![PN-T4](image\PN-T4.png)

##### 5.2 结构设计分析

###### 5.2.1 证明平衡无序性采用的平衡策略的有效性

在4.2.1小节中我们提到了如何处理无序的输入数据集，提到了三种方法，最后选择的maxpooling方法，这一小节就是证明了这个选择很正确。![PN-F5](image\PN-F5.png)



###### 5.2.2 证明T-net的有效性

从下表中可以看到T-net的加入使有效的，提升了0.8%的准确度，如果再加上之前提到的正则化的手法，将会取得更好的效果。

![PN-T5](image\PN-T5.png)

###### 5.2.3 鲁棒性测试

在此实验中，用到了一些损失点云信息做测试，当损失50%的点的时候，准确度仅仅下降了2.4%至3.8%；而对于增加一些随机点的情况，鲁棒性依然很好，我们做了两个实验一个是特征设成3维坐标形式，另一个是三维坐标形式外加一个点密度，事实证明，即使有20%的外部点参与，准确度依然可以达到80%以上。

![PN-F6](image\PN-F6.png)

##### 5.3 网络可视化

在图7中，可以很清楚的看到原始点集，重要点集(critical point sets)和上限点集(upper-bound shapes)的对比情况，其中重要点集Cs主要描述出了shape的主要骨架，而上限形状集 Ns则描述出了最大可能的和输入点云有着相同全局特征的点集，Cs和Ns的可视化反映出了整个网络的鲁棒性很强，也解释了失去一些点并不会对网络产生多么大的影响。

![PN-F7](image\PN-F7.png)

##### 5.4 时空复杂性分析

对比算法为MVCNN和Subvolumn两种网络，PointNet和上述两种算法的精确度都很高，但PN的时间复杂度和空间复杂度都是O(N)，是线性增长的，而另外两个网络由于包含卷积层，都是呈平方增长的。

时间复杂度上面，PointNet可以每秒处理超过100万个点的图像分类，大概是1千个物体的检测，或者也可以在全景分割上每秒处理2间屋子，分别领先上述两种算法141倍，8倍；空间复杂度上面，PN只用了350万参数，领先两种算法17倍，4倍；

训练用到的是1080X GPU和tensorflow，实时性上面有了很强的保障。其中PointNet (vanilla)   是表示去掉T-net后的PointNet网络，更为轻便。

![PN-T6](image\PN-T6.png)

#### 6 结论

这篇论文提出了一种可以直接运用点云数据的深度神经网络PointNet，可以处理物体分类，物体局部分割和语义全景分割等任务，性能等价或者超过现有的网络结构，我们同时还分析了这个网络，给出了网络的可视化，帮助读者理解这个网络。