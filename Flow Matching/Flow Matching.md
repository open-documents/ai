


本文介绍生成模型中的一种重要范式：流匹配。
# 一、前置知识
本章介绍前置知识 CNF，包含ODE公式、push-forward公式。首先，定义符号：

| 符号 | 含义 |
| :--- | :--- |
| $x$ | 位置 |
| $t$ | 时间。范围为 \[0, 1\]，0为开始，1为结束。 |
| $\phi_t(x)$ | 流（flow）：定义某时刻粒子位置，即初始位置为 $x$ 的粒子，在时刻 $t$ 时所处的位置。 |
| $\phi_t^{-1}(x)$ | 流的逆：粒子在时刻$t$的位置为$x$时，该粒子的初始位置。 |
| $v_t(x)$ | 速度场/向量场（vector field，VF）：定义某时刻位置变化，即初始位置为 $x$ 的粒子，在时刻$t$时的速度（大小和方向）。 |
| $p_t(x)$ | 概率密度路径：时刻 $t$ 时的粒子概率密度。 |
| $p_0(x)、p_1(x)$ | 初始时刻的粒子概率密度 && 接近真实数据概率密度 |


1、**CNF（连续归一化流，Continuous Normalizing Flow）**：一种用神经网络参数化的连续时间模型，基于ODE、前推公式，将一个简单的先验分布（如标准高斯噪声）转化为复杂的数据分布（真实数据分布）。

**2、ODE（常微分方程，Ordinary Differential Equation）**

$$
\begin{align}
\frac{d}{dt}\phi_t(x) &= v_t(\phi_t(x))  
\\  \nonumber
\\  
\phi_0(x) &= x
\end{align}
$$

等式1：流和速度场的关系。可以理解为，初始位置为$x$的粒子在t时刻到达位置$\phi_t(x)$后，粒子该位置的速度即是粒子在该位置的位置变化；

等式2：当时刻$t$为$0$、粒子初始位置为$x$时，流是粒子的初始位置；

得到CNF对$v_t(x)$建模后，基于上述方程，只要步长足够小、求解器精度足够高，采用如下的步骤，即可得到一个近似从真实数据分布中采样的数据：

- 从先验分布采样：从标准高斯分布中随机抽取一个纯噪声 $x_0∼N(0,I)$；
- 送入 ODE 求解器：将$x_0$作为初始状态（t=0），利用训练好的神经网络$v_t(x)$作为导数，通过数值积分（如论文使用的 dopri5 自适应求解器，或简单的欧拉法、中点法）沿着时间$t$从0积分到1；
- 得到三



**3、push-forward（前推）公式**

$$
\begin{align}
p_t &= [\phi_t]_* p_0   \\ 
 \nonumber  \\  
[\phi_t]_* p_0(x) &= p_0(\phi_t^{-1}(x)) \det \left[ \frac{\partial \phi_t^{-1}}{\partial x}(x) \right]
\end{align}
$$
等式3：前推公式。可以理解为，初始概率密度经过流$\phi_t(x)$后的概率密度；

等式4：已知初始时刻粒子的概率密度分布解析式 $p_0(x)$、流$\phi_t(x)$，给定$t$时刻粒子位置$x$，计算$t$时刻粒子的概率密度分布；

> 参考"附录-1、CFM例子"帮助理解上述公式。

# 二、FM损失函数

考虑概率密度路径满足：

- t = 0时：$p_0(x)$是简单的概率密度，如标准正态分布；
- t = 1时：$p_1(x)$接近真实数据概率密度；

流匹配（Flow matching，FM）的目标是通过网络拟合一个边缘速度场，来满足上述要求，损失函数为：

$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t, p_t(x)} \|v_t(x) - u_t(x)\|^2, \tag{5}
$$

直接训练有两个问题：

- 无法抽样：因为没有$p_t(x)$的解析式；
- 没有标签：因为没有速度场$u_t(x)$的解析式；

# 二、条件速度场 -> 边缘速度场
在引出论文中的CFM损失函数前，先介绍设定条件概率密度$p_t(x|x_1)$，满足：

- t = 0时：$p_0(x|x_1)=p(x)$；
- t = 1时：粒子集中在中心为$x_1$的一片极小区域，如$p_1(x|x_1) = \mathcal{N}(x|x_1, \sigma^2 I)$，其中标准差极小；

那么，有：

$$
p_t(x) = \int p_t(x|x_1)q(x_1)dx_1
\\
p_1(x) = \int p_1(x|x_1)q(x_1)dx_1 \approx q(x)
$$



进一步：

$$
u_t(x) = \int u_t(x|x_1) \frac{p_t(x|x_1)q(x_1)}{p_t(x)} dx_1, \tag{8}
$$

即条件速度场能推出边缘速度场。
# FM && CFM
流匹配（Flow Matching）的损失函数为：





为此该论文提出了替代损失函数CFM：

$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t, q(x_1), p_t(x|x_1)} \|v_t(x) - u_t(x|x_1)\|^2,
$$

可以证明，CFM和FM关于模型参数的梯度一致，即优化CFM即在优化FM。



# 构造路径
构造仿射路径：

$$
\psi_t(x) = \sigma_t(x_1)x + \mu_t(x_1)
$$

由ODE公式有：

$$
\frac{d}{dt} \psi_t(x) = u_t(\psi_t(x)|x_1)  
\quad 
\Rightarrow
\quad
u_t(x|x_1) = \frac{\sigma'_t(x_1)}{\sigma_t(x_1)} (x - \mu_t(x_1)) + \mu'_t(x_1)
$$

进一步，做重参数化，损失函数变为：

$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t, q(x_1), p(x_0)} \left\| v_t(\psi_t(x_0)) - \frac{d}{dt}\psi_t(x_0) \right\|^2
$$



## OT路径
构造仿射路径：

$$
\mu_t(x) = tx_1, \text{ and } \sigma_t(x) = 1 - (1 - \sigma_{\min})t  \Rightarrow \psi_t(x) = (1 - (1 - \sigma_{\min})t)x + tx_1
$$

基于此，得到条件速度场：

$$
u_t(x|x_1) = \frac{x_1 - (1 - \sigma_{\min})x}{1 - (1 - \sigma_{\min})t}
$$




损失函数：

$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t,q(x_1),p(x_0)} \left\| v_t(\psi_t(x_0)) - \big(x_1 - (1 - \sigma_{\min})x_0\big) \right\|^2
$$





## Diffusion路径


# 附录
## 1、CFM例子
设定：

- 向量场：$v_t(x)=x$；
- 初始密度：标准正态分布，即$p_0(x) \sim N(0,1)$；



1）解 ODE：

$$
\left\{
\begin{aligned}
\quad\frac{d}{dt}\phi_t(x)&=v_t\bigl(\phi_t(x)\bigr)\\[2pt]
\\
\phi_0(x)&=x
\end{aligned}
\right.

\quad
\;\overset{\text{带入 }v_t(x)=x}{\Longrightarrow}\;
\quad

\left\{
\begin{aligned}
\quad \frac{d}{dt}\phi_t(x)&=\phi_t(x)\\[2pt]
\\
\phi_0(x)&=x
\end{aligned}
\right.

\quad
\;\Longrightarrow\;
\quad

\phi_t(x)=e^{t}x
$$

即：$t$ 时刻把整条直线拉伸 $e^t$ 倍。



2）计算等式4：

$$
\begin{aligned}
p_t(x)
&= p_0(e^{-t}x)\cdot e^{-t} \\[4pt]
&= \frac{1}{\sqrt{2\pi}}
   \exp\!\left(-\frac{e^{-2t}x^2}{2}\right)\cdot e^{-t} \\[4pt]
&= \frac{1}{\sqrt{2\pi}\,e^t}
   \exp\!\left(-\frac{x^2}{2e^{2t}}\right) \\[4pt]
&= \mathcal{N}(x\mid 0,\;e^{2t}).
\end{aligned}
$$

即：方差为1的标准正态高斯，被流拉伸后变成方差为$e^{2t}$的高斯。



3）数值验证（取 $t=\ln 2$）

此时 $e^t=2$，流就是 $\phi(x_0)=2x_0$，空间被拉伸 2 倍。

| 量 | 数值 |
| --- | --- |
| 目标点 | $x=2$ |
| 源点 | $\phi^{-1}(2)=1$ |
| $p_0(1)$ | $0.24197$ |
| $\det=1/2$ | $0.5$ |
| 乘积 | $0.1210$ |

直接算 $\mathcal{N}(0,4)$ 在 $x=2$ 处的密度：

$$
\frac{1}{2\sqrt{2\pi}}\exp\!\left(-\frac{2^2}{2\cdot 4}\right)
= 0.19947\times 0.60653
= 0.1210 \quad\checkmark
$$

再验证总质量仍为 1：

$$
\int p_t(x)\,dx
= \int p_0(e^{-t}x)\,e^{-t}\,dx
= \int p_0(u)\,du
= 1 \quad\checkmark
$$

（令 $u=e^{-t}x$。）

这正是"变量替换 + 质量守恒"的含义：空间的长度被拉伸了 2 倍，密度就必须缩小到 $1/2$。高斯峰值从 $\mathcal{N}(0,1)$ 的 $0.3989$ 降到 $\mathcal{N}(0,4)$ 的 $0.1995$，恰好减半。