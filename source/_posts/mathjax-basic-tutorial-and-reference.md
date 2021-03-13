---
title: Mathjax基本教程和参考
date: 2018-12-06 23:09:16
tags: [mathjax]
categories: Tools
mathjax: true
---
## 公式布局
### 插入公式
markdown中的公式排版有两种方式：inline 和 display，分别表示行内和行间。对于行内公式，用`$...$`来包括公式，对于行间公式，用符号`$$...$$`。例如:   
`$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$`   
显示为 $\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$   
`$$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$$`显示为：
$$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$$   
<!-- more -->
### 修饰符
#### 上下标
| 上下标 | 语法  | 示例  | 输出  |
| :----: | :---: | :---: | :---: |
|  上标  |  `^`  | `n^2` | $n^2$ |
|  上标  |  `_`  | `C_n` | $C_n$ |

#### 上下修饰符
| 重音符 |                         语法                         |                              示例                              |                              输出                               |
| :----: | :--------------------------------------------------: | :------------------------------------------------------------: | :-------------------------------------------------------------: |
| 脱字符 |                 `\hat`<br>`\widehat`                 |                  `\hat{x}`<br>`\widehat{xy}`                   |                   $\hat{x}$<br>$\widehat{xy}$                   |
| 上划线 |                `\bar`<br>`\overline`                 |                  `\bar{x}`<br>`\overline{xy}`                  |                  $\bar{x}$<br>$\overline{xy}$                   |
| 下划线 |                     `\underline`                     |                        `\underline{xy}`                        |                        $\underline{xy}$                         |
| 上箭头 | `\vec`<br>`\overrightarrow`<br>`\overleftrightarrow` | `\vec{x}`<br>`\overrightarrow{xy}`<br>`overleftrightarrow{xy}` | $\vec{x}$<br>$\overrightarrow{xy}$<br>$\overleftrightarrow{xy}$ |
| 下箭头 |     `\underrightarrow`<br>`\underleftrightarrow`     |      `\underrightarrow{xy}`<br>`underleftrightarrow{xy}`       |      $\underrightarrow{xy}$<br>$\underleftrightarrow{xy}$       |
|   点   |                  `\dot`<br>`\ddot`                   |                    `\dot{x}`<br>`\ddot{x}`                     |                     $\dot{x}$<br>$\ddot{x}$                     |

#### 分组和括号
- 上标，下标和其他操作仅适用于下一个“组”，一个组是由大括号`{...}`包括的单个符号或公式，用大括号来限定上下标应用的公式。例如：`$10^10$` 显示为$10^10$,而`$10^{10}`显示为$10^{10}$；`$x_i^2$`显示为$x_i^2$,而`$x_{i^2}$`显示为$x_{i^2}$
- 显示大括号需要转义字符`\`，例如 `$\{y = x^2\}$`显示为 $\{y = x^2\}$
- 利用`\left`...`\right`可以使括号根据其内公式大小自动调整，适用于下列所有括号：`()`，`[]`，`{}`，尖括号:`$\langle...\langle$` $\langle...\rangle$， 绝对值: `$\vert...\vert$` $\vert...\vert$，双竖线：`$\Vert...\Vert$` $\Vert...\Vert$。例如 `(\frac{\sqrt x}{y^3})`显示为 $(\frac{\sqrt x}{y^3})$，而`$\left(\frac{\sqrt x}{y^3}\right)$` 显示为 $\left(\frac{\sqrt x}{y^3}\right)$

## 字体与希腊字母

### 字体
|   字体   |       语法        |          示例          |         输出         |
| :------: | :---------------: | :--------------------: | :------------------: |
| 黑板加粗 | `\mathbb`或`\Bbb` |  `$\mathbb {ABCabc}$`  |  $\mathbb {ABCabc}$  |
|   黑体   |     `\mathbf`     |  `$\mathbf {ABCabc}$`  |  $\mathbf {ABCabc}$  |
|   斜体   |     `\mathit`     |  `$\mathit {ABCabc}$`  |  $\mathit {ABCabc}$  |
|  黑斜体  |      `\pmb`       |   `$\pmb {ABCabc}$`    |   $\pmb {ABCabc}$    |
| 打字机体 |     `\mathtt`     |  `$\mathtt {ABCabc}$`  |  $\mathtt {ABCabc}$  |
|  罗马体  |     `\mathrm`     |  `$\mathrm {ABCabc}$`  |  $\mathrm {ABCabc}$  |
| 无衬线体 |     `\mathsf`     |  `$\mathsf {ABCabc}$`  |  $\mathsf {ABCabc}$  |
|  书法体  |    `\mathcal`     | `$\mathcal {ABCabc}$`  | $\mathcal {ABCabc}$  |
|  脚本体  |    `\mathscr`     | `$\mathscr {ABCabc}$`  | $\mathscr {ABCabc}$  |
|   德体   |    `\mathfrak`    | `$\mathfrak {ABCabc}$` | $\mathfrak {ABCabc}$ |

### 希腊字母
|    命令    |    显示    |   命令   |   显示   |
| :--------: | :--------: | :------: | :------: |
|  `\alpha`  |  $\alpha$  | `\beta`  | $\beta$  |
|  `\gamma`  |  $\gamma$  | `\delta` | $\delta$ |
| `\epsilon` | $\epsilon$ | `\zeta`  | $\zeta$  |
|   `\eta`   |   $\eta$   | `\theta` | $\theta$ |
|  `\iota`   |  $\iota$   | `\kappa` | $\kappa$ |
| `\lambda`  | $\lambda$  |  `\mu`   |  $\mu$   |
|   `\nu`    |   $\nu$    |  `\xi`   |  $\xi$   |
|   `\pi`    |   $\pi$    |  `\rho`  |  $\rho$  |
|  `\sigma`  |  $\sigma$  |  `\tau`  |  $\tau$  |
| `\upsilon` | $\upsilon$ |  `\phi`  |  $\phi$  |
|   `\chi`   |   $\chi$   |  `\psi`  |  $\psi$  |
|  `\omega`  |  $\omega$  |   `--`   |   $--$   |
- 如果要大写希腊字母，首字母大写即可，例如`$\Sigma$`，显示为 $\Sigma$
- 如果要希腊字母显示为斜体，命令前加`var`，例如`$\varGamma`，显示为 $\varGamma$

## 常用数学运算符
### 基础符号
|  运算符  |     语法     |       示例       |       输出       |
| :------: | :----------: | :--------------: | :--------------: |
|    加    |     `+`      |      `x+y`       |      $x+y$       |
|    减    |     `-`      |      `x-y`       |      $x-y$       |
|   叉乘   |   `\times`   |   `x \times y`   |   $x \times y$   |
|   点乘   |   `\cdot`    |   `x \cdot y`    |   $x \cdot y$    |
|   星乘   |    `\ast`    |    `x \ast y`    |    $x \ast y$    |
|    除    |    `\div`    |    `x \div y`    |    $x \div y$    |
|   加减   |    `\pm`     |    `x \pm y`     |    $x \pm y$     |
|   减加   |    `\mp`     |    `x \mp y`     |    $x \mp y$     |
|   等于   |     `=`      |     `x = y`      |     $x = y$      |
| 小于等于 |    `\leq`    |    `x \leq y`    |    $x \leq y$    |
| 大于等于 |    `\geq`    |    `x \geq y`    |    $x \geq y$    |
|  约等于  |  `\approx`   |  `x \approx y`   |  $x \approx y$   |
|  恒等于  |   `\equiv`   |   `x \equiv y`   |   $x \equiv y$   |
|   圆点   |  `\bigodot`  |  `x \bigodot y`  |  $x \bigodot y$  |
|   圆乘   | `\bigotimes` | `x \bigotimes y` | $x \bigotimes y$ |
|   取模   |   `\pmod`    |   `x \pmod n`    |   $x \pmod n$    |


### 集合符号
| 运算符 |     语法      |       示例        |       输出        |
| :----: | :-----------: | :---------------: | :---------------: |
|  属于  |     `\in`     |     `x \in y`     |     $x \in y$     |
|  子集  |   `\subset`   |   `x \subset y`   |   $x \subset y$   |
| 真子集 |  `\subseteq`  |  `x \subseteq y`  |  $x \subseteq y$  |
|  超集  |   `\supset`   |   `x \supset y`   |   $x \supset y$   |
|  空集  | `\varnothing` | `x \varnothing y` | $x \varnothing y$ |
|   并   |    `\cup`     |    `x \cup y`     |    $x \cup y$     |
|   交   |    `\cap`     |    `x \cap y`     |    $x \cap y$     |

### 特殊符号
|   命令    |   显示    |    命令     |    显示     |
| :-------: | :-------: | :---------: | :---------: |
| `\infty`  | $\infty$  | `\partial`  | $\partial$  |
| `\nabla`  | $\nabla$  | `\triangle` | $\triangle$ |
| `\forall` | $\forall$ |  `\exists`  |  $\exists$  |
|  `\lnot`  |  $\lnot$  |   `\land`   |   $\land$   |
|  `\lor`   |  $\lor$   |   `\top`    |   $\top$    |
| `\vdash`  | $\vdash$  |  `\vDash`   |  $\vDash$   |

### 求和、求积与积分
求和：`\sum` $\sum$，求积：`\prod` $\prod$，积分：`\int` $\int$ `iint` $\iint$ `iiint` $\iiint$ `oint` $\oint$，上标代表上限，下标代表下限，例如：`\sum_1^n`，显示 $\sum_1^n$

### 分式与根式
#### 分式
有三种实现方式，`\frac ab`应用于紧接着的两个组，例如`$\frac ab$`，显示为 $\frac ab$ ；对于更复杂的分子和分母，使用`{...}`，例如`$\frac{a+1}{b+1}`，显示为 $\frac{a+1}{b+1}$ ；使用`\over`分割组内分数，例如`${a+1\over b+1}$` ，显示为 ${a+1\over b+1}$ ；使用`\cfrac`命令对于连续的分手，例如`$\cfrac{a}{b+1}$` ，显示为 $\cfrac{a}{b+1}$ 。
#### 根式
使用命令`\sqrt` ，例如，`$\sqrt{x^3}$` $\sqrt{x^3}$ ；`$\sqrt[3]{\frac xy}$` $\sqrt[3]{\frac xy}$
### 特殊函数
"lim","sin","max","ln"等函数正常设置为罗马体而不是斜体。例如`$\sin x`，显示为 $\sin x$，而`$sin x$`，显示为 $sin x$。极限:`$$\lim_x{x\to 0}$$`
 $$\lim_{x\to 0}$$   

|   函数   |  语法  |        示例        |       输出       |
| :------: | :----: | :----------------: | :--------------: |
|   正弦   | `\sin` |  `$\sin {(x+y)}$`  |  $\sin {(x+y)}$  |
|   余弦   | `\cos` |  `$\cos {(x+y)}$`  |  $\cos {(x+y)}$  |
|   正切   | `\tan` |  `$\tan {(x+y)}$`  |  $\tan {(x+y)}$  |
|   余切   | `\cot` |  `$\cot {(x+y)}$`  |  $\cot {(x+y)}$  |
|   正割   | `\sec` |  `$\sec {(x+y)}$`  |  $\sec {(x+y)}$  |
|   余割   | `\csc` |  `$\csc {(x+y)}$`  |  $\csc {(x+y)}$  |
|   对数   | `\log` | `$\log_n {(x+y)}$` | $\log_n {(x+y)}$ |
| 自然对数 | `\ln`  |  `$\ln {(x+y)}$`   |  $\ln {(x+y)}$   |
|   余弦   | `\csc` |  `$\csc {(x+y)}$`  |  $\csc {(x+y)}$  |

### 矩阵
- 矩阵以`$$\begin{matrix}`开始，以`end{matrix}$$`结束。每个元素之间用`&`分割，矩阵每行以`\\`结尾。例如:
```
$$
  \begin{matrix}
  1 & x & x^2 \\
  1 & y & y^2 \\
  2 & z & z^2 \\
  \end{matrix}
$$
```
显示为：
$$
  \begin{matrix}
  1 & x & x^2 \\
  1 & y & y^2 \\
  2 & z & z^2 \\
  \end{matrix}
$$
- 矩阵显示括号：用`\left....\right`或者`pmatrix`，`bmatrix`，`Bmatrix`，`vmatrix`，`Vmatrix`。例如：
$$\begin{pmatrix} 1 & 2 \\3 & 4 \\ \end{pmatrix} $$
$$\begin{bmatrix} 1 & 2 \\3 & 4 \\ \end{bmatrix} $$
$$\begin{Bmatrix} 1 & 2 \\3 & 4 \\ \end{Bmatrix} $$
$$\begin{vmatrix} 1 & 2 \\3 & 4 \\ \end{vmatrix} $$
$$\begin{Vmatrix} 1 & 2 \\3 & 4 \\ \end{Vmatrix} $$
- 利用`\cdots` $\cdots$ `\ddots` $\ddots$ `vdots` $\vdots$ 来省略矩阵中的某些项。例如
$$\begin{pmatrix}
  1 & a_1 & a_1^2 & \cdots & a_1^n \\
  1 & a_2 & a_2^2 & \cdots & a_2^n \\
  \vdots & \vdots & \ddots & \vdots \\
  1 & a_m & a_m^2 & \cdots & a_m^n \\
  \end{pmatrix}$$
- 对于水平"增广矩阵"，可以用`array`实现，例如：
```
$$ \left[
\begin{array}{cc|c}
  1&2&3\\
  4&5&6
\end{array}
\right] $$
```
显示为：
$$ \left[
\begin{array}{cc|c}
  1&2&3\\
  4&5&6
\end{array}
\right] $$
`cc|c`是关键部分，代表三个居中的列向量，在第二列和第三列之间有一个垂直的杠。对于竖直方向，用`\hline`，例如：
```
$$
  \begin{pmatrix}
    a & b\\
    c & d\\
  \hline
    1 & 0\\
    0 & 1
  \end{pmatrix}
$$
```
显示为：
$$
  \begin{pmatrix}
    a & b\\
    c & d\\
  \hline
    1 & 0\\
    0 & 1
  \end{pmatrix}
$$
### 分段函数
使用命令`\begin{cases}...\end{cases}`，每个函数以`\\`结尾，在应该对齐的部分前用`&`，例如：
```
$$f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}$$
```
得到：
$$f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}$$
括号可以被移动到右边：
$$
\left.
\begin{array}{l}
\text{if $n$ is even:}&n/2\\
\text{if $n$ is odd:}&3n+1
\end{array}
\right\}
=f(n)
$$
命令为：
```
\left.
\begin{array}{l}
\text{if $n$ is even:}&n/2\\
\text{if $n$ is odd:}&3n+1
\end{array}
\right\}
=f(n)
```
要得到更大的函数竖直方向间隔，可以使用`\\[2ex]`代替`\\`。例如：
$$f(n) =
\begin{cases}
\frac{n}{2},  & \text{if $n$ is even} \\[2ex]
3n+1, & \text{if $n$ is odd}
\end{cases}$$
命令为：
```
f(n) =
\begin{cases}
\frac{n}{2},  & \text{if $n$ is even} \\[2ex]
3n+1, & \text{if $n$ is odd}
\end{cases}
```
### 数组和表格
数组和表格用`array`命令创建，在`begin{array}`之后是每列的格式，`c`代表中心对齐，`r`代表右对齐，`l`代表左对齐，`|`显示竖直线。和矩阵一样，元素用`&`分隔，行之间用`\\`。在当前行之间使用`\hline`可在数组中添加水平线。例如：
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i
\end{array}
$$
```
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i
\end{array}
$$
```
数组可以嵌套得到表格数组。
$$
% outer vertical array of arrays
\begin{array}{c}
% inner horizontal array of arrays
\begin{array}{cc}
% inner array of minimum values
\begin{array}{c|cccc}
\text{min} & 0 & 1 & 2 & 3\\
\hline
0 & 0 & 0 & 0 & 0\\
1 & 0 & 1 & 1 & 1\\
2 & 0 & 1 & 2 & 2\\
3 & 0 & 1 & 2 & 3
\end{array}
&
% inner array of maximum values
\begin{array}{c|cccc}
\text{max}&0&1&2&3\\
\hline
0 & 0 & 1 & 2 & 3\\
1 & 1 & 1 & 2 & 3\\
2 & 2 & 2 & 2 & 3\\
3 & 3 & 3 & 3 & 3
\end{array}
\end{array}
\\
% inner array of delta values
\begin{array}{c|cccc}
\Delta&0&1&2&3\\
\hline
0 & 0 & 1 & 2 & 3\\
1 & 1 & 0 & 1 & 2\\
2 & 2 & 1 & 0 & 1\\
3 & 3 & 2 & 1 & 0
\end{array}
\end{array}
$$
#### 方程组
使用`\begin{array}...\end{array}` 或者`\begin{cases}...\end{cases}`和`\left\{...\right`。例如：
$$
\left\{
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\
a_2x+b_2y+c_2z=d_2 \\
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$
命令为：
```
$$
\left\{
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\
a_2x+b_2y+c_2z=d_2 \\
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$
```
或者使用：
```
$$\begin{cases}
a_1x+b_1y+c_1z=d_1 \\
a_2x+b_2y+c_2z=d_2 \\
a_3x+b_3y+c_3z=d_3
\end{cases}
$$
```
使用`\begin{aligned}...\end{aligned}`和`\left\{...\right`来对齐等号`=`。例如：
$$
\left\{
\begin{aligned}
a_1x+b_1y+c_1z &=d_1+e_1 \\
a_2x+b_2y&=d_2 \\
a_3x+b_3y+c_3z &=d_3
\end{aligned}
\right.
$$
代码为：
```
$$
\left\{
\begin{aligned}
a_1x+b_1y+c_1z &=d_1+e_1 \\
a_2x+b_2y&=d_2 \\
a_3x+b_3y+c_3z &=d_3
\end{aligned}
\right.
$$
```
使用`array`和`l`可以得到下列对齐形式的方程组：
$$
\left\{
\begin{array}{ll}
a_1x+b_1y+c_1z &=d_1+e_1 \\
a_2x+b_2y &=d_2 \\
a_3x+b_3y+c_3z &=d_3
\end{array}
\right.
$$
命令为：
```
$$
\left\{
\begin{array}{ll}
a_1x+b_1y+c_1z &=d_1+e_1 \\
a_2x+b_2y &=d_2 \\
a_3x+b_3y+c_3z &=d_3
\end{array}
\right.
$$
```
和'cases'一样，用`\\[2ex]`代替`\\`可以得到更大的垂直方向间隔，方程组：
$$\begin{cases}
a_1x+b_1y+c_1z=d_1 \\[2ex]
a_2x+b_2y+c_2z=d_2 \\[2ex]
a_3x+b_3y+c_3z=d_3
\end{cases}
$$
由下列代码生成
```
$$\begin{cases}
a_1x+b_1y+c_1z=d_1 \\[2ex]
a_2x+b_2y+c_2z=d_2 \\[2ex]
a_3x+b_3y+c_3z=d_3
\end{cases}
$$
```
## 参考文献
[mathjax-basic-tutorial-and-quick-reference](https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)
