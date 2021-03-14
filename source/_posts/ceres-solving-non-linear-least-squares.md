---
title: ceres-Solving non-linear least squares problem
date: 2019-09-15 09:40:21
tags: [non-linear least squares]
categories: SLAM
mathjax: true
---
## 非线性最小二乘问题求解
### 引言
有效使用Ceres需要熟悉非线性最小二乘求解器的一些基础组成部分，所以在描述怎么配置和使用求解器之前，先来简单看一下Ceres中的一些核心优化算法是如何工作的。   

令 $x \in \mathbb{R}^n$ 是变量的一个 $n$ 维向量， $F(x) = \left[f_1(x), ... ,  f_{m}(x) \right]^{\top}$ 是 $x$ 的 $m$ 维函数。优化问题描述如下：
$$ 
\begin{split}\arg \min_x \frac{1}{2}\|F(x)\|^2  \\
L \le x \le U\end{split}  \tag{1}
$$   
这里， $L$ 和 $U$ 是参数向量 $x$ 的上下边界。   

由于对于一般的 $F(x)$ ，很难有有效的全局最小化方法，一般通过寻找局部最小化来解决。 $F(x)$ 的雅可比矩阵 $J(x)$ 是一个 $m \times n$ 的矩阵。这里， $J_{ij}(x) = \partial_j f_i(x)$ ，梯度向量为 $g(x) = \nabla \frac{1}{2}\|F(x)\|^2 = J(x)^\top F(x)$。   

 解决非线性优化问题的一般策略是通过解决原始问题的一系列近似。在一次迭代中，求解近似来得到向量 $x$ 的一个校正量 $\Delta x$，对于非线性最小二乘，通过线性化来构造近似 $F(x+\Delta x) \approx F(x) + J(x)\Delta x$，这就得到了下边的线性最小二乘问题：

 $$
 \min_{\Delta x} \frac{1}{2}\|J(x)\Delta x + F(x)\|^2\quad(2)
 $$
 不幸的是，简单地求解一系列这些问题并更新 $x \leftarrow x+ \Delta x$ 会导致算法不收敛。为了得到一个收敛的算法，我们需要控制步长 $\Delta x$ 的大小。 根据如何控制步长 $\Delta x$ 的大小，非线性优化算法可以分为两大类：
 1. **信赖域（Trust Region)**
    信赖域方法在搜索空间（称为信赖域）的一个子集上利用模型函数来近似目标函数，如果模型函数成功地最小化真正地目标函数，信赖域就会扩大，反之缩小，再次求解模型优化问题。
 2. **线搜索（Line Search)**
    线搜索方法首先找到目标函数减少的下降方向，然后计算决定朝下降方向移动移动的步长大小。下降方向可以由多种方法计算，例如梯度下降法， Newton's method 和 Quasi-Newton 方法。

信赖域方法首先选择一个步长大小然后是步长方向，而线搜索方法首先选择步长方向然后是步长大小。

### 信赖域方法
基本的信赖域方法流程如下：
  1. 给定一个初始点 $x$ 和 一个信赖域半径 $\mu$ 。
  2. 求解
     $$\begin{split}\arg \min_{\Delta x}& \frac{1}{2}\|J(x)\Delta x + F(x)\|^2 \\ \text{such that} &\|D(x)\Delta x\|^2 \le \mu\\
     &L \le x + \Delta x \le U.\end{split}
     $$
  3. $$
      \rho = \frac{\displaystyle \|F(x + \Delta x)\|^2 -
       \|F(x)\|^2}{\displaystyle \|J(x)\Delta x + F(x)\|^2 -
       \|F(x)\|^2}
      $$
  4. 如果 $\rho > \epsilon$ 则 $x = x + \Delta x$
  5. 如果 $\rho > \eta_1$ 则 $\mu = 2\mu$
  6. 如果 $\rho < \eta_2$ 则 $\mu = 0.5 * \mu$
  7. 返回第二步
这里，$\mu$ 是信赖域半径， $D(x)$ 矩阵定义了 $F(x)$ 域上的度量。$\rho$ 测量步长 $\Delta x$ 的质量，例如，线性模型在非线性目标值里预测下降有多好。通过线性化预测非线性目标的行为有多好来决定增大或减小信赖域的范围。   

信赖域方法的关键步骤求解带约束的优化问题
$$
\begin{split}\arg \min_{\Delta x}&\quad \frac{1}{2}\|J(x)\Delta x + F(x)\|^2 \\
\text{such that} &\quad \|D(x)\Delta x\|^2 \le \mu\\
 &\quad L \le x + \Delta x \le U.\end{split}\quad (3)
$$
 Ceres实现了两种信赖域方法：Levenberg-Marquardt 和 Dogleg。

 #### Levenberg-Marquardt
 Levenberg-Marquardt 是求解非线性最小二乘问题的最流行的算法。求解问题（3）可以通过求解下边形式的无约束优化问题：
$$\arg\min_{\Delta x} \frac{1}{2}\|J(x)\Delta x + F(x)\|^2 +\lambda  \|D(x)\Delta x\|^2$$
这里， $\lambda$ 是拉格朗日乘数是 $\mu$的逆，在Ceres中， 求解：
$$\arg\min_{\Delta x} \frac{1}{2}\|J(x)\Delta x + F(x)\|^2 + \frac{1}{\mu} \|D(x)\Delta x\|^2\quad(4)$$
矩阵 $D(x)$ 是一个非负对角阵， 通常是 $J(x)^\top J(x)$ 的对角阵的平方根。   

在进行下一步之前，我们先简化一些表示。假设矩阵 $\frac{1}{\sqrt{\mu}} D$ 已经添加到矩阵 $J$ 的底部，相似的，零向量也被添加到向量 $f$ 的底部。 下边的讨论将使用 $J$ 和 $f$ ：
$$\min_{\Delta x} \frac{1}{2} \|J(x)\Delta x + f(x)\|^2 \quad (5)$$
Ceres提供两类方法来解决问题（5）- 分解和迭代。   

分解法利用Cholesky 和 QR 分解来计算精确解。但是不清楚问题（4）的精确解在求解问题（1）时LM算法的的每一步是否是必须的。  

不精确的牛顿法需要两部分。首先，一种近似求解线性方程组的廉价方法。例如，像Conjugate Gradients方法这样的迭代线性求解器。 第二，迭代求解器的终止规则。 典型的终止规则是下边的形式：
$$\|H(x) \Delta x + g(x)\| \leq \eta_k \|g(x)\|\quad(6)$$
这里，$k$ 表示LM迭代次数。

#### Dogleg
另外一种求解信赖域问题的策略由M.J.D.Powell引入。关键思想是计算两个向量：
$$\begin{split}\Delta x^{\text{Gauss-Newton}} &= \arg \min_{\Delta x}\frac{1}{2} \|J(x)\Delta x + f(x)\|^2.\\
\Delta x^{\text{Cauchy}} &= -\frac{\|g(x)\|^2}{\|J(x)g(x)\|^2}g(x).\end{split}$$
注意向量 $\Delta x^{\text{Gauss-Newton}}$ 是问题（2）的解，$\Delta
x^{\text{Cauchy}}$ 是最小化线性近似的向量如果我们限制朝着梯度方向移动。Dogleg方法寻找由 $\Delta x^{\text{Gauss-Newton}}$ 定义的向量 $\Delta x$ 和 $\Delta x^{\text{Cauchy}}$ 来解决信赖域问题。

#### Inner Iterations
一些非线性最小二乘问题在参数块相互作用的方式上有额外的结构，这有利于改变信赖域步骤计算的方式。例如，考虑下边的回归问题：
$$y = a_1 e^{b_1 x} + a_2 e^{b_3 x^2 + c_1}$$
给定一个 $\{(x_i, y_i)\}$ 的集合，用户想要估计 $a_1, a_2, b_1, b_2$ 和 $c_1$ 。   

注意到表达式左边在$a_1$ 和 $a_2$ 上是线性的，给定任意的 $b_1, b_2, c_1$ ，可以用线性回归来估计 $a_1$ 和 $a_2$ 的最优值，从问题中完全解析地消除变量 $a_1$ 和 $a_2$ 是可能地。 像这样地问题称为可分离地最小二乘问题，求解这类问题最有名的算法是由Golub和Pereyra提出的Variable Projection 算法。 相似的结构可以在带有丢失数据问题的矩阵分解中找到，相应的算法叫做Wiberg's算法。   

将`Solver::Options::use_inner_iterations`设置为`true`可以启用Ruhe&Wedin's Algorithm II。Ceres版本具有更高的迭代复杂度，但是每次迭代中也会更好地收敛。

#### Non-monotonic Steps
注意上述的信赖域方法是一个下降算法只接受使目标函数值严格减少的点。放宽这个条件可以使算法在长期内更有效，代价是目标函数的值会在局部增大。这是因为允许非下降的目标函数值可以使算法跳过巨石，因为该方法不限于移动到窄谷中同时保持其收敛特性。   

将`Solver::Options::use_nonmonotonic_steps`设置为`true`可以启用非单调信赖域算法。

### Line Search Method
Ceres中的线搜索方法目前还不能处理边界约束，所以只可以用来求解无约束问题。线搜索算法步骤如下：
  1. 给定一个初始点 $x$
  2. $\Delta x = -H^{-1}(x) g(x)$
  3. $\arg \min_\mu \frac{1}{2} \| F(x + \mu \Delta x) \|^2$
  4. $x = x + \mu \Delta x$
  5. 返回到步骤2
这里 $H(x)$ 使目标函数的黑塞矩阵的近似估计，$g(x)$ 是在 $x$ 处的梯度。根据 $H(x)$ 的选择，我们可以得到各种不同的搜索方向 $\Delta x$ 。目前，Ceres支持三种搜索方向。
  1. `STEEPEST_DESCENT` 这对应于选择 $H(x)$ 为单位矩阵。除了对最简单的问题外，这不是一个好的搜索方向。
  2. `NONLINEAR_CONJUGATE_GRADIENT` 共轭梯度法对非线性函数的一般化，可以以不同方式执行，导致不同的搜索方向。Ceres 目前支持 `FLETCHER_REEVES`，`POLAK_RIBIERE` 和 `HESTENES_STIEFEL` 方向。
  3. `BFGS` Secant方法对多维的一般化，其中维持了逆黑塞矩阵的一个完全密集近似用于计算quasi-Newton 步骤
  4. `LBFGS` `BFGS`的有限近似方法。

### LinearSolver
