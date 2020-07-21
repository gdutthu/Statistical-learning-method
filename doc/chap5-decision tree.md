﻿@[TOC](统计学习方法 第5章：决策树)

github链接：[https://github.com/gdutthu/Statistical-learning-method](https://github.com/gdutthu/Statistical-learning-method)
知乎专栏链接：[https://zhuanlan.zhihu.com/c_1252919075576856576](https://zhuanlan.zhihu.com/c_1252919075576856576)
# 1 提出模型
在对决策树模型进行讲解前，我们先来看一个简单的例子。我们收集到下面数据。从下表可看出，苹果的好坏和它的颜色、硬度、香味密切相关。
| 苹果序号 | 颜色 | 硬度 |香味 |结论 |
| :-: | :-: | :-:  | :-:  | :-:  |
| 1| 红色| 硬 |香| 好苹果 |
|2 | 红色 | 硬 |无味| 好苹果 |
| 3| 红色 | 软|无味| 坏苹果 |
| 4| 绿色| 硬 |香| 好苹果 |
| 5 | 绿色 | 硬 |无味| 坏苹果 |
| 6 | 绿色 | 软 |无味| 坏苹果 |

那么基于上面的表格，我们可以画出下面这样苹果好坏的判断模型。

![在这里插入图片描述](../image/决策树示意图.png)

这就是决策树的模型，它通过对大量训练样本的学习去建立一个决策树，依次判断每个属性，从而判断该样本的标记。我们可以这样画个图。每次拿到一个苹果就从最顶上开始依次往下判断，最后得出结论。



# 2 学习策略
通过上一小节的例子，我们可以直观的看出决策树模型是怎样对一个测试样本进行判断的。那么在这一小节中，我们将来学习怎样建立一个决策树模型。
## 2.1 决策树模型
用决策树分类，从根节点开始，对实例的某一特征进行测试，根据测试结果，将实例分配到其子节点；这时，每一个子节点对应着该特征的一个取值。如此递归地对实例进行测试并分配，直到到达叶节点。最终将实例分到叶节点的类中。

![在这里插入图片描述](../image/决策树模型.png)

上图看成一个决策树模型示意图，其中途中的圆框和方框分别表示内部节点（internal node）和叶节点（leaf node）。内部节点表示一个特征或属性，叶节点表示一个类。

##  2.2 模型规则
**决策树模型可看成大量的 $if-then$规则的集合**。 个人觉得在决策树构建这块，李航博士的《统计学习方法》讲的已经很详细了。那么在这里，只是将下面算法中需要用到的公式摘录出来。

 - **随机变量的熵$H(p)$**

设$X$是一个取有限个值得离散随机量，其概率分布为
$$P(X=x_{i})=p_{i},i=1,2,...,n$$
将随机变量$X$的熵$H(p)$定义为：
$$H(p)=-\sum_{i=1}^{n} p_{i} \log p_{i}$$

值得注意的是
1、熵$H(p)$只与$X$的分布有关，而与$X$的取值无光；
2、熵越大，随机变量的不确定性越大。并且从熵的定义可看出：
$$0 \leq H(p) \leq \log n$$
3、$\log$函数的底数可以是2也可以是$e$,一般是选择2为底数。

举个例子，以上述表格中苹果的好坏为例。从表格可看出6个苹果中有3个好苹果，有3个坏苹果。定于变量$X=\{x_{1},x_{2}\}$，其中$x_{1},x_{2}$分别表示好苹果和坏苹果。那么该变量的熵为：
$$
\begin{aligned}
H(p)&=-\sum_{i=1}^{2}{p(x_{i})* \log p(x_{i})}\\
&=-1*(p(x_{1})* \log p(x_{1})+p(x_{2})* \log p(x_{2}))\\
&=-1*(\frac{1}{2}\log \frac{1}{2}+\frac{1}{2}\log \frac{1}{2})=0.69
 \end{aligned}
$$
在这里，我们将$\log$函数的底数选为$e$

 - **经验熵$H(Y \mid X)$**

经验熵$H(Y | X)$表示在已知随机变量$X$的条件下随机变量$Y$的不确定性。随机变量$X$给定的条件下随机变量$Y$的条件熵$H(Y | X)$，定义为$X$给定条件下$Y$的条件概率分布的熵对$X$的数学期望
$$H(Y \mid X)=\sum_{i=1}^{n} p_{i} H\left(Y \mid X=x_{i}\right)$$
这里，$p_{i}=P(X=x_{i}) ,i=1,2,...,n$

接着上述的例子，将苹果的香味定义为变量$Y=\{y_{1},y_{2}\}$，其中$y_{1},y_{2}$分别表示有香味和无香味。从表格可看出，3个好苹果中有2个苹果有香味，1个苹果无香味；3个坏苹果均为无香味。

$$\begin{aligned}
H(Y \mid X=x_{1})&=-\sum_{j=1}^{2}p(y_{i},x_{1})*\log p(y_{i},x_{1}) \\
&=-1*(\frac{2}{3}*\log \frac{2}{3}+\frac{1}{3}*\log \frac{1}{3})=0.6365
 \end{aligned}$$

$$\begin{aligned}
H(Y \mid X=x_{2})&=-\sum_{j=1}^{2}p(y_{i},x_{2})*\log 
p(y_{i},x_{2}) \\
&=-1*(0+1*\log 1)=0
 \end{aligned}$$
则
$$H(Y \mid X)=H(Y \mid X=x_{1})+H(Y \mid X=x_{2})=0.6365$$

 - **信息增益$g(D, A)$**

**信息增益 表示特征$X$的信息而使得类$Y$的信息$Y$的信息的不确定性减少的程度。** 特征$A$对训练数据集$D$的信息增益$g(D, A)$，定义为集合$D$的的经验熵$H(D)$与特征$A$给定条件下$D$的经验条件熵$H(D \mid A)$之差，即


$$g(D, A)=H(D)-H(D \mid A) $$

继续接着上面两个例子进行讲解。得到苹果的香味这一特征对苹果好坏的信息增益为：
$$g(D, A)=H(D)-H(D \mid A) =0.69-0.6365=0.0535$$

##  2.3 信息增益的算法

输入：训练数据集$D$和特征$A$
输出：特征$A$对训练数据集$D$的信息增益 g(D, A)
（1）计算数据集$D$的经验熵
$$H(D)=-\sum_{k=1}^{K} \frac{\left|C_{k}\right|}{|D|} \log _{2} \frac{\left|C_{k}\right|}{|D|}$$
（2）计算特征$A$对数据集$D$的经验熵$H(D \mid A)$
$$H(D \mid A)=\sum_{i=1}^{n} \frac{\left|D_{i}\right|}{|D|} H\left(D_{i}\right)=-\sum_{i=1}^{n} \frac{\left|D_{i}\right|}{|D|} \sum_{k=1}^{K} \frac{\left|D_{i k}\right|}{\left|D_{i}\right|} \log _{2} \frac{\left|D_{i k}\right|}{\left|D_{i}\right|}$$
（3）计算信息增益
$$g(D, A)=H(D)-H(D \mid A) $$

# 3 算法流程
第四小节的决策树代码采用的是ID3算法。

**输入：** 训练数据集$D$，特征集$A$阈值$\xi$;
**输出：** 决策树$T$。

 1. 若$D$中所有实例属于同一类$C_{k}$，则$T$为单结点树，并将类$C_{k}$作为该节点的类标记，返回$T$；
 2. 若$A=\varnothing$，则$T$为单节点树，并将$D$中实例数最大的类$C_{k}$作为该节点的类标记，返回$T$；
  3. 否则，计算特征集$A$中各特征对$D$的信息增益，选择增益最大的特征$A_{g}$;如果$A_{g}$的信息增益小于阈值$\xi$，则置$T$为单节点树，并将$D$中实例数最大的类$C_{k}$作为该结点的类标记，返回$T$；
 4. 否则，对$A_{g}$的每一个可能值$\alpha_{i}$，依$A_{g}=\alpha_{i}$将$D$分割为若干子集$D_{i}$，将$D_{i}$中实例最大的类作为标记，构建子节点，由节点及其子节点构成数$T$，返回$T$；
 5. 对第$i$个子节点，以$D_{i}$为训练集，以$A-A_{g}$为特征集，递归调用（1）~（5）步，得到子树$T_{i}$,返回$T_{i}$。

# 4 代码附录


```python
代码明天补充
```
