> 论文：Denoising Diffusion Probabilistic Models

本文介绍生成领域的一种重要范式：扩散模型。该论文并非首次提出扩散模型，但是其算法DDPM是扩散模型被广泛采用的一个奠基之作。

# 一、扩散模型
扩散模型是一种潜变量模型，并不直接对初始数据集分布$p(x_0)$建模，而是引入一条逐步加噪的潜变量链 $x_1, ..., x_T$，建模联合概率分布$p(x_{0:T})$：

$$
\begin{align}
p(x_{0:T})  = p(x_T)\prod_{t=1}^{T}p(x_{t-1}|x_{t})
\end{align}
$$
其中：

$$
\begin{aligned}
& \quad x_{0:T} = (x_0, x_1, ..., x_T)
\\
\\
& \quad p(x_T) = N(x_T; 0,I)
\end{aligned}
$$





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

2、