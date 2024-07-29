# 激活函数

**神经元工作原理：**

1. 输入input，与神经元权重相乘
2. 与偏置相加
3. 将结果喂给激活函数f(x)
4. 将输出送入下一神经元



## Sigmoid

公式为
$$
f(z) = \frac{1}{(1+e^{-z})}
$$
图像为

<img src="assets\sigmoid.png" alt="sigmoid" style="zoom: 80%;" />

- 优点：Sigmoid函数将输出值进行了**归一化**，因此适用于结果为概率的模型。Sigmoid梯度平滑，避免跳跃的输出值，**函数可微**，意味着可以找到任意两个点的曲线斜率，使其在梯度下降中表现良好。Sigmoid是**单调递增**的，在输出随输入增大而增大的问题中有益。
- 缺点：当输入非常大或非常小时，Sigmoid 函数的梯度接近于零，这导致**梯度消失**问题（指梯度为0）。函数输出不是以0为中心，即均**值不为0**，可能会导致在神经网络层叠加时出现偏移现象，这会降低权重更新的效率。当输入值位于饱和区域时（接近0）梯度非常大，可能导致**梯度爆炸**。Sigmoid执行**指数运算**，效率低。

 Sigmoid 在二分类概率输出上表现良好。



## Tanh / 双曲正切函数

表达式
$$
f(x) = tanh(x) = \frac{2}{1+e^{-2x}} - 1
$$
图像为

<img src="assets\tanh.png" alt="tanh" style="zoom:80%;" />

- 优点：tanh的输出间隔比sigmoid大，并且整个函数以0为中心。
- 缺点：当输出较大或较小时，输出几乎是平滑的并且梯度较小，这不利于权重更新。

在tanh中，负输入将被强映射为负，而零输入被映射为接近零。在一般的二元分类问题中，tanh 函数用于隐藏层，而 sigmoid 函数用于输出层。



## ReLU激活函数

表达式为
$$
\sigma(x) = \begin{cases}
    max(0, x) &  x \ge 0 \\
    0 &  x \lt 0
\end{cases}
$$
图像为

<img src="assets\ReLU.png" alt="ReLU" style="zoom:80%;" />

- 优点：当输入为正时不存在梯度饱和问题，且计算速度足够快
- 缺点：Dead ReLU问题。当输入为负时，ReLU 完全失效。且不以0为中心。



## Leaky ReLU

表达式
$$
f(y_i) = \begin{cases} 
	y_i & y_i \gt 0 \\
	a_iy_i & y_i \le 0
\end{cases}
$$
图像为

<img src="D:\AllProjects\learning\Nets\assets\Leaky ReLU.png" alt="Leaky ReLU" style="zoom:80%;" />

- 优点：Leaky ReLU 通过把x的非常小的线性分量赋给负输出以调整负输入的梯度值。leak 有助于扩大 ReLU 函数的范围，通常 a 的值为 0.01 左右；Leaky ReLU 的函数范围是负无穷到正无穷。

从理论上讲，Leaky ReLU 具有 ReLU 的所有优点，而且 Dead ReLU 不会有任何问题，但在实际操作中，尚未完全证明 Leaky ReLU 总是比 ReLU 更好。



## ELU

表达式为
$$
ELU(x) = \begin{cases}
	x, & x\gt 0 \\
	\alpha(e^x - 1), & x\le0
\end{cases}
$$
图像为

<img src="D:\AllProjects\learning\Nets\assets\ELU.png" alt="ELU" style="zoom:80%;" />

- 优点：ELU没有Dead ReLU问题，且平均值接近零，以0为中心。ELU减少了偏执偏移的影响，使正常梯度更接近于单位自然梯度。ELU 在较小的输入下会饱和至负值，从而减少前向传播的变异和信息。

目前在实践中没有充分的证据表明 ELU 总是比 ReLU 好。

<img src="D:\AllProjects\learning\Nets\assets\relu compare.png" alt="relu compare" style="zoom:80%;" />

## PReLU

PReLU 也是 ReLU 的改进版本
$$
f(y_i) = \begin{cases}
	y_i, & y_i \gt 0 \\
	\alpha_i y_i, & y_i \le 0
\end{cases}
$$
可学习参数α通常为 0 到 1 之间的数字，并且通常相对较小。

![image-20240729113036134](C:\Users\Kim\AppData\Roaming\Typora\typora-user-images\image-20240729113036134.png)

- 当a_i = 0，则为ReLU
- 当a_i > 0，则为Leaky ReLU
- 当a_i变为可学习的参数，则为PreLU

PReLU 的优点：在负值域，PReLU 的斜率较小，这也可以避免 Dead ReLU 问题。与 ELU 相比，PReLU 在负值域是线性运算。尽管斜率很小，但不会趋于 0。



## Softmax

Softmax 是用于多类分类问题的激活函数。对于K为向量，Softmax输出K维值在(0, 1)范围内，且向量中元素总和为1的实向量。

公式为
$$
y_i = f(x_i) = \frac{e^{x_i}}{\Sigma^{K}_{j=1}e^{x_j}}
$$
Softmax 与正常的 max 函数不同：max 函数仅输出最大值，但 Softmax 确保较小的值具有较小的概率，并且不会直接丢弃。

- 主要缺点：**零点不可微**；负输入的梯度为零，对于该区域的激活，**权重不会在反向传播期间更新**，因此会产生永不激活的死亡神经元。



## Swish

表达为，其中σ是sigmoid函数，β是可学习或固定参数。
$$
y = x * \sigma(\beta x)
$$
即
$$
y = \frac{x}{1+e^{-\beta x}}
$$


图像为

<img src="D:\AllProjects\learning\Nets\assets\Swish.png" alt="Swish" style="zoom:80%;" />

- 优点：平滑的非线性函数，有利于梯度下降。Swish 函数是**非单调**的，这有助于模型学习更复杂的非线性关系。**自动门控性**，在输入为负时，函数可能会接近于 0，类似于 ReLU，而在输入为正时，函数保持输入的幅度。在某些情况下，Swish 函数可以比 ReLU 和其它激活函数提供更好的性能。

<img src="D:\AllProjects\learning\Nets\assets\Swish compare.png" alt="Swish compare" style="zoom:80%;" />

## Softplus

表达式
$$
f(x) = ln (1 + e^x)
$$
其导数为sigmoid函数
$$
f'(x) = \frac{e^x}{1 + e^x} = \frac{1}{1+e^{-x}}
$$
因此也被称为logistic/sigmoid函数。

Softplus 函数类似于 ReLU 函数，但是相对较平滑，像 ReLU 一样是单侧抑制。Softplus 函数**通常用作神经网络中隐藏层的激活函数**，有时也用作输出层，特别是在需要预测正数值的回归任务中。

<img src="D:\AllProjects\learning\Nets\assets\Softplus compare.png" alt="Swish compare" style="zoom:80%;" />

