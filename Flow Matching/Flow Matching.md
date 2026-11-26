


本文介绍生成模型中的一种重要范式：流匹配
# Continuous Normalizing Flows（CNF）
本章介绍前置知识 CNF（对应论文第2章），包含ODE公式、push-forward公式。

首先，定义符号：

| 符号 | 含义 |
| :--- | :--- |
| $x$ | 位置 |
| $t$ | 时间。范围为 \[0, 1\] |
| $\phi_t(x)$ | 流（flow）：定义某时刻粒子位置，即初始位置为 $x$ 的粒子，在时刻 $t$ 时所处的位置。 |
| $v_t(x)$ | 速度场/向量场（vector field，VF）：定义某时刻位置变化，即初始位置为 $x$ 的粒子，在时刻 $t$ 时的速度（大小和方向）。 |
| $p_t(x)$ | 概率密度路径：时刻 $t$ 时的粒子概率密度，刻画某时刻粒子的分布。 |
| $p_0(x)$ | 初始时刻的粒子概率密度，刻画初始时刻粒子分布。 |



**1、ODE（常微分方程，ordinary differential equation）**

$$
\begin{align}
\frac{d}{dt}\phi_t(x) &= v_t(\phi_t(x))  
\\  \nonumber
\\  
\phi_0(x) &= x
\end{align}
$$

等式1：速度场产生流，粒子位置的微分即是速度场；

等式2：初始位置为 $x$ 的粒子，在时刻 $0$ 时所处位置即是初始位置；




**2、push-forward（前推）公式**

$$
\begin{align}
p_t &= [\phi_t]_* p_0   \\ 
 \nonumber  \\  
[\phi_t]_* p_0(x) &= p_0(\phi_t^{-1}(x)) \det \left[ \frac{\partial \phi_t^{-1}}{\partial x}(x) \right]
\end{align}
$$



## 例子

### 1. 设定

**流**：取向量场 $v_t(x)=x$，解 ODE：

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

**初始密度**：$p_0=\mathcal{N}(0,1)$。

### 2. 逐项计算式 (4) 的四个部件

目标：求 $t$ 时刻密度 $p_t(x)$。

| 部件阿阿阿 | 计算阿阿 |
| --- | --- |
| ① 目标点 $x$阿 | 任取一个数，如 $x=1$ |
| ② 回溯源点 $\phi_t^{-1}(x)$ | ODE 反解：$\phi_t^{-1}(x)=e^{-t}x$阿阿 |

### 3. 代入式 (4)

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

结果：本来方差为 1 的高斯，被流拉伸后变成方差为 $e^{2t}$ 的高斯。

### 4. 数值验证（取 $t=\ln 2$）

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



# 条件速度场推出边缘速度场




# FM && CFM
流匹配（Flow Matching）的损失函数为：

$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t, p_t(x)} \|v_t(x) - u_t(x)\|^2, \tag{5}
$$

1、直接训练有两个问题：

- 无法抽样：因为没有$p_t(x)$的解析式；
- 没有标签：因为没有速度场$u_t(x)$的解析式；



为此该论文提出了替代损失函数CFM：

$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t, q(x_1), p_t(x|x_1)} \|v_t(x) - u_t(x|x_1)\|^2,
$$

