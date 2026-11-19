# 等式 (4) 的两个直观例子

论文中的 push-forward 密度变换公式：

$$
[\phi_t]_* p_0(x)
= p_0\!\left(\phi_t^{-1}(x)\right)
  \det\!\left[\frac{\partial \phi_t^{-1}(x)}{\partial x}\right]
\tag{4}
$$

---

## 例子一：一维缩放流 + 标准高斯（真正由 ODE 生成）

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

| 部件 | 计算 |
| --- | --- |
| ① 目标点 $x$ | 任取一个数，如 $x=1$ |
| ② 回溯源点 $\phi_t^{-1}(x)$ | ODE 反解：$\phi_t^{-1}(x)=e^{-t}x$ |
| ③ 源点密度 $p_0(\phi_t^{-1}(x))$ | $p_0(e^{-t}x)=\dfrac{1}{\sqrt{2\pi}}\exp\!\left(-\dfrac{e^{-2t}x^2}{2}\right)$ |
| ④ 体积因子 $\det\!\left[\partial\phi_t^{-1}(x)/\partial x\right]$ | 一维时就是导数：$e^{-t}$ |

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

### 5. 直觉对照

```text
     t = 0 源                                 t 时刻
   p_0 = N(0,1)                          p_t = N(0, e^{2t})
      ▁▄█▄▁                    ──►           ▁▄▄█▄▄▁   （变宽、变矮）
      x_0 ──── 流拉伸 e^t 倍 ────►  x = e^t x_0
```

- 要问 $x$ 处的密度 → 先回溯到源点 $e^{-t}x$ → 查 $p_0$ → 再乘体积因子 $e^{-t}$。
- 流把体积放大 $e^t$ 倍 ⇒ 密度乘 $e^{-t}$；而 $e^{-t}=1/\det(\partial\phi_t/\partial x)$，与"正、逆雅可比互为倒数"一致。

### 6. 矩阵版本

同样的事情在二维：若 $\phi(x_0)=Ax_0$，$A=\operatorname{diag}(2,3)$，则

$$
\det\!\left[\frac{\partial\phi^{-1}(x)}{\partial x}\right]
= \det(A^{-1})
= \frac{1}{2\cdot 3}
= \frac{1}{6}.
$$

$p_0=\mathcal{N}(0,I)$ 时 $p_t=\mathcal{N}(0,AA^\top)$，分布被拉伸成 $2\times 3$ 的椭圆，密度处处乘以 $1/6$。

若 $A=\operatorname{diag}(2,1/2)$，$\det=1$：一个方向拉宽、一个方向压窄，面积不变，密度不变——这就是"$\det$ 只管体积，不管形状"。

### 小结

式 (4) 在这个例子里就是：$\mathcal{N}(0,1)$ 被拉伸 $e^t$ 倍后，密度变成 $\mathcal{N}(0,e^{2t})$；公式里的 $p_0(e^{-t}x)$ 负责"查源点密度"，$e^{-t}$ 负责"按体积变化打折"，两者相乘恰好给出正确的密度。