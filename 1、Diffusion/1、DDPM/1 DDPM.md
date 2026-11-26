> 论文：Denoising Diffusion Probabilistic Models

本文介绍生成领域的一种重要范式：扩散模型。该论文并非首次提出扩散模型，但是其算法DDPM是扩散模型被广泛采用的一个奠基之作。

# 一、扩散模型
扩散模型是一种潜变量模型，并不直接对初始数据集分布 $p(x_0)$ 进行建模，而是引入一条逐步加噪的潜变量链 $x_1, ..., x_T$，通过联合分布 $$
> 论文：Jonathan Ho, Ajay Jain, and Pieter Abbeel. Denoising diffusion probabilistic models. arXiv:2006.11239, 2020.

贡献：

- 展示了DDPM和SM（Score Matching）之间的关系。
-


等式

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
