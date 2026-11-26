> 论文：Denoising Diffusion Probabilistic Models

本文介绍生成领域的一种重要范式：扩散模型。该论文并非首次提出扩散模型，但是其算法DDPM是扩散模型被广泛采用的一个奠基之作。

首先定义

$L=\int q(x_0)logp(x_0)dx_0$ （来自Deep Unsupervised Learning using Nonequilibrium Thermodynamics. 2015

# 一、扩散模型
扩散模型是一种潜变量模型，并不直接对初始数据集分布$p(x_0)$建模，而是通过定义一个前向加噪的马尔可夫链，并对具备马尔可夫链性质反向去噪过程建模，从而构建生成过程中的联合分布。

$$
\begin{align}
p(x_{0:T})  = p(x_T)\prod_{t=1}^{T}p(x_{t-1}|x_{t})
\end{align}
$$
其中：

$$
\begin{aligned}
& x_{0:T} = (x_0, x_1, ..., x_T)
\\
\\
& p(x_T) = N(x_T; 0,I)
\end{aligned}
$$

其中，$x_0$是真实的、清晰的原始数据，$x_1, x_2, ..., x_T$是逐步加噪后的数据。

扩散模型建模














$$
\begin{aligned}
log \ p_\theta(x_\theta)
&= log \int p_\theta(x_{0:T}) \ dx_{1:T}  \\
&= log \int q(x_{1:T}|x_0) \frac{p_\theta(x_{0:T})}{q(x_{1:T}|x_0)} \ dx_{1:T}  \\
&= log \ E_{q(x_{1:T}|x_0)}  \frac{p_\theta(x_{0:T})}{q(x_{1:T}|x_0)} \\
&\ge E_{q(x_{1:T}|x_0)} \ log \frac{p_\theta(x_{0:T})}{q(x_{1:T}|x_0)} \\
\Rightarrow -log \ p_\theta(x_\theta) & \le E_{q(x_{1:T}|x_0)} \ \big[ -log \frac{p_\theta(x_{0:T})}{q(x_{1:T}|x_0)} \ \big]
\end{aligned}
$$





# 附录
## 1、证明等式1
$$
\begin{align}
p(x_{0:T})                                                                                       \nonumber
&= p(x_{0:T-1}|x_T) \cdot p(x_T)                                                   \nonumber      \\
&= p(x_{0:T-2}|x_{T-1},x_{T}) \cdot p(x_{T-1}|x_T) \cdot p(x_T)      \nonumber      \\
& \ ...                  \nonumber     \\
&= p(x_0|x_{1:T}) \cdot p(x_1|x_{2:T}) \cdot p(x_2|x_{3:T})...p(x_{T-1}|x_T) \cdot p(x_T)                           \nonumber    \\
&= p(x_0|x_1) \cdot p(x_1|x_2) \cdot p(x_2|x_3)...p(x_{T-2}|x_{T-1}) \cdot p(x_{T-1}|x_T) \cdot p(x_T)  \ \ \ （逆向马尔可夫）    \nonumber    \\
&= p(x_T)\prod_{t=1}^{T}p(x_{t-1}|x_{t})
\end{align}
$$

2、损失函数

首先，根据全概率公式，有：

$$
\begin{aligned}
p(x_0)                              
&= \int p(x_{0:T})dx_{1:T}                                                               \\
&= \int p(x_{0:T}) \frac{q(x_{1:T} | x_0)}{q(x_{1:T} | x_0)}dx_{1:T}      \\
&= \int q(x_{1:T} | x_0) \frac{p(x_{0:T})}{q(x_{1:T} | x_0)}dx_{1:T}      \\
&= \int q(x_{1:T} | x_0)p(x_T)\prod_{t=1}^{T}\frac{p(x_{t-1}|x_{t})}{q(x_t|x_{t-1})}dx_{1:T}
\end{aligned}
$$
基于此，损失函数：

$$
\begin{aligned}
L
&= \mathbb{E}_{q(x_0)} \left[ \log p_\theta(x_0) \right]      \\ 
&= \int q(x_0) \ logp_\theta(x_0)dx_0   \\
&= \int q_\theta(x_0) \cdot log[\int q(x_{1:T} | x_0)p(x_T) \cdot \prod_{t=1}^{T}\frac{p(x_{t-1}|x_{t})}{q(x_t|x_{t-1})}dx_{1:T}]dx_0  \\
&\geq \int q(x_{0:T})\cdot log[p(x_T)\frac{p(x_{t-1}|x_{t})}{q(x_t|x_{t-1})}]d(x_{0:T}) = K
\end{aligned}  \\
$$