---
title: Ceres-Modeling Non-Linear Least Squares
date: 2019-05-12 20:57:53
tags: [non-linear least squares]
categories: SLAM
mathjax: true
---
## 用ceres解决非线性最小二乘问题
google的开源优化库ceres可以用来解决稳健的边界约束非线性最小二乘问题，形式如下：
$$\min_x \frac{1}{2}\sum_{i}\rho_i({||f_i(x_{i1},\cdots,x_{ik})||}^2)$$
$$ s.t. l_j \leq x_j \leq u_j$$
在ceres中，表达式 $\rho_i({||f_i(x_{i1},\cdots,x_{ik})||}^2)$ 称作残差块(residual block)， $f_i(\cdot)$ 是代价函数(`CostFunction`)， ${x_{i1},\cdots,x_{ik}}$ 为参数块(parameter blocks)。$\rho_i$ 是损失函数(`LossFunction`)，损失函数是一个标量方程用来减少非线性最小二乘问题中局外点的影响，又称为核函数。$l_j$ 和 $u_j$ 是参数块 $x_j$ 的上下范围。   
特殊情况下，当 $\rho_i(x) =x$，而且 $l_j = -\infty$ ， $u_j = \infty$ 就得到常见的无约束非线性最小二乘问题：
$$\frac12\sum_{i}{||f_i(x_{i1},\cdots,x_{ik})||}^2$$

### 代价函数
对于目标函数的每一项，代价函数用来计算残差向量和雅可比矩阵，具体来说，考虑函数 $f(x_1,\cdots,x_k)$，其参数块为 $[x_1,\cdots,x_k]$。给定参数块 $[x_1,\cdots,x_k]$ ，代价函数负责计算残差向量 $f(x_1,\cdots,x_k)$ 和雅可比矩阵
$$J_i =  \frac{\partial}{\partial x_i} f(x_1, ..., x_k) \quad \forall i \in \{1, \ldots, k\} $$
ceres 提供了多种代价函数，它们都派生与基类`CostFunction`，它有两个重要的成员变量`CostFunction::parameter_block_sizes_`和`CostFunction::num_residuals_`，分别表示参数块的大小和残差的个数。
#### SizedCostFunction
如果参数块的大小和残差向量的大小在编译时期是已知的，这时候就可以使用`SizedCostFunction`，参数通过模板参数指定，用户只需实现`CostFunction::Evaluate()`。
```
template<int kNumResiduals,
         int N0 = 0, int N1 = 0, int N2 = 0, int N3 = 0, int N4 = 0,
         int N5 = 0, int N6 = 0, int N7 = 0, int N8 = 0, int N9 = 0>
class SizedCostFunction : public CostFunction {
 public:
  virtual bool Evaluate(double const* const* parameters,
                        double* residuals,
                        double** jacobians) const = 0;
};
```
#### AutoDiffCostFunction
定义一个`CostFunction`或`SizedCostFunction`是比较麻烦和容易出错的，尤其在计算导数的时候，Ceres提供了自动差分方法,是最常用的代价函数。
```
template <typename CostFunctor,
       int kNumResiduals,  // Number of residuals, or ceres::DYNAMIC.
       int N0,       // Number of parameters in block 0.
       int N1 = 0,   // Number of parameters in block 1.
       int N2 = 0,   // Number of parameters in block 2.
       int N3 = 0,   // Number of parameters in block 3.
       int N4 = 0,   // Number of parameters in block 4.
       int N5 = 0,   // Number of parameters in block 5.
       int N6 = 0,   // Number of parameters in block 6.
       int N7 = 0,   // Number of parameters in block 7.
       int N8 = 0,   // Number of parameters in block 8.
       int N9 = 0>   // Number of parameters in block 9.
class AutoDiffCostFunction : public
SizedCostFunction<kNumResiduals, N0, N1, N2, N3, N4, N5, N6, N7, N8, N9> {
 public:
  explicit AutoDiffCostFunction(CostFunctor* functor);
  // Ignore the template parameter kNumResiduals and use
  // num_residuals instead.
  AutoDiffCostFunction(CostFunctor* functor, int num_residuals);
};
```
为了得到一个自动差分代价函数，你必须定义一个带有模板操作符()的类（函数对象）作为模板参数`T`。为了计算导数，自动差分框架会在需要的时候为`T`替换合适的`Jet`对象，但这是隐藏的。这个函数必须把计算的值写入到最后一个参数中（唯一的非const参数）并返回`true`。   
例如，考虑一个标量误差 $e = k - x^\top y$，`x`和`y`都是二维向量参数，`k`是一个常量。这种常量和表达式的差形式的误差是最小二乘问题中的常见模式。例如，$x^\top y$ 可能是一系列观测值的模型期望，对于每个观测值都有一个代价函数的实例。   
添加到最小二乘问题中的实际代价是 $e^2$ ，或者 $(k - x^\top y)^2$；平方向由优化框架隐式完成。
要写出上述模型的一个自动差分代价函数，首先定义个函数对象：
```
class MyScalarCostFunctor {
  MyScalarCostFunctor(double k): k_(k) {}

  template <typename T>
  bool operator()(const T* const x , const T* const y, T* e) const {
    e[0] = k_ - x[0] * y[0] - x[1] * y[1];
    return true;
  }

 private:
  double k_;
};
```
注意`operator()`的声明，首先是输入参数`x`和`y`,是一个指向类型为`T`的数组的常量指针。输出项总是最后一个参数，也是指向数组的指针，在上述例子中，`e`是一个标量，所以只有`e[0]`被赋值。   
定义了这个类后，自动差分代价函数可以按下边的方式构造：
```
CostFunction* cost_function
    = new AutoDiffCostFunction<MyScalarCostFunctor, 1, 2, 2>(
        new MyScalarCostFunctor(1.0));              ^  ^  ^
                                                    |  |  |
                        Dimension of residual ------+  |  |
                        Dimension of x ----------------+  |
                        Dimension of y -------------------+
```
在这个例子中，每个测量值`k`都有一个实例。模板参数`MyScalarCostFunction,<1,2,2>`描述了函数对象从两个二维参数中计算一个一维输出。
`AutoDiffCostFunction`还支持运行时决定残差个数，例如：
```
CostFunction* cost_function
    = new AutoDiffCostFunction<MyScalarCostFunctor, DYNAMIC, 2, 2>(
        new CostFunctorWithDynamicNumResiduals(1.0),   ^     ^  ^
        runtime_number_of_residuals); <----+           |     |  |
                                           |           |     |  |
                                           |           |     |  |
          Actual number of residuals ------+           |     |  |
          Indicate dynamic number of residuals --------+     |  |
          Dimension of x ------------------------------------+  |
          Dimension of y ---------------------------------------+
```
#### DynamicAutoDiffCostFunction
`AutoDiffCostFunction`需要在编译期知道参数块的个数和大小。还有参数块数量的限制，最多十个。在许多应用中，这是不够的，例如，贝塞尔曲线拟合。
```
template <typename CostFunctor, int Stride = 4>
class DynamicAutoDiffCostFunction : public CostFunction {
};
```
和`AutoDiffCostFunction`一样，用户必须定义一个模板函数对象，但是函数签名有些不同，形式如下：
```
struct MyCostFunctor {
  template<typename T>
  bool operator()(T const* const* parameters, T* residuals) const {
  }
}
```
由于参数的的大小是在运行时决定的，在创建动态差分代价函数之后你必须说明参数大小，例如：
```
DynamicAutoDiffCostFunction<MyCostFunctor, 4>* cost_function =
  new DynamicAutoDiffCostFunction<MyCostFunctor, 4>(
    new MyCostFunctor());
cost_function->AddParameterBlock(5);
cost_function->AddParameterBlock(10);
cost_function->SetNumResiduals(21);
```
#### NumericDiffCostFunction
在一些情况下，定义一个模板代价函数对象是不可能的，例如当评价残差涉及到调用你没有控制权的库函数。这种情况下，可以使用数值差分。
```
template <typename CostFunctor,
          NumericDiffMethodType method = CENTRAL,
          int kNumResiduals,  // Number of residuals, or ceres::DYNAMIC.
          int N0,       // Number of parameters in block 0.
          int N1 = 0,   // Number of parameters in block 1.
          int N2 = 0,   // Number of parameters in block 2.
          int N3 = 0,   // Number of parameters in block 3.
          int N4 = 0,   // Number of parameters in block 4.
          int N5 = 0,   // Number of parameters in block 5.
          int N6 = 0,   // Number of parameters in block 6.
          int N7 = 0,   // Number of parameters in block 7.
          int N8 = 0,   // Number of parameters in block 8.
          int N9 = 0>   // Number of parameters in block 9.
class NumericDiffCostFunction : public
SizedCostFunction<kNumResiduals, N0, N1, N2, N3, N4, N5, N6, N7, N8, N9> {
};
```
对于一个标量误差 $e = k - x'y$ 数值差分函数对象定义如下：
```
class MyScalarCostFunctor {
  MyScalarCostFunctor(double k): k_(k) {}

  bool operator()(const double* const x,
                  const double* const y,
                  double* residuals) const {
    residuals[0] = k_ - x[0] * y[0] + x[1] * y[1];
    return true;
  }

 private:
  double k_;
};
```
利用中心差分计算导数的数值差分代价函数可以如下构造：
```
CostFunction* cost_function
    = new NumericDiffCostFunction<MyScalarCostFunctor, CENTRAL, 1, 2, 2>(
        new MyScalarCostFunctor(1.0));                    ^     ^  ^  ^
                                                          |     |  |  |
                              Finite Differencing Scheme -+     |  |  |
                              Dimension of residual ------------+  |  |
                              Dimension of x ----------------------+  |
                              Dimension of y -------------------------+
```
NumericDiffCostFunction也支持运行时决定残差大小。
#### DynamicNumericDiffCostFunction
对于`AutoDiffCostFunction`，`NumericDiffCostFunction`，需要在编译期知道参数块的个数和大小，只支持对多10个参数块，在许多情况下，这是不够的,这是可以使用`DynamicNumericDiffCostFunction`。
```
template <typename CostFunctor, NumericDiffMethodType method = CENTRAL>
class DynamicNumericDiffCostFunction : public CostFunction {
};
```
像`NumericDiffCostFunction`一样，用户必须定义一个函数对象，但是函数签名有些不同。例如：
```
struct MyCostFunctor {
  bool operator()(double const* const* parameters, double* residuals) const {
  }
}
```
参数大小在运行时决定，在创建动态数值差分代价函数时你必须说明参数大小。例如：
```
DynamicNumericDiffCostFunction<MyCostFunctor>* cost_function =
  new DynamicNumericDiffCostFunction<MyCostFunctor>(new MyCostFunctor);
cost_function->AddParameterBlock(5);
cost_function->AddParameterBlock(10);
cost_function->SetNumResiduals(21);
```
#### CostFunctionToFunctor
`CostFunctionToFunctor` 是一个适配器类允许用户在模板函数对象中使用`CostFuntion`对象，模板函数用来自动差分。例如：
```
class IntrinsicProjection : public SizedCostFunction<2, 5, 3> {
  public:
    IntrinsicProjection(const double* observation);
    virtual bool Evaluate(double const* const* parameters,
                          double* residuals,
                          double** jacobians) const;
};
```
是一个`CostFunction`实现一个在局部坐标系下的点到它的像平面的投影并从观测点投影减去它。可以计算它的残差或者通过解析或数值差分计算其雅可比。
现在我们想将这个`CostFunction`和相机外参组合起来。假设我们由一个模板函数
```
template<typename T>
void RotateAndTranslatePoint(const T* rotation,
                             const T* translation,
                             const T* point,
                             T* result);
```
我们可以像下面这样做：
```
struct CameraProjection {
  CameraProjection(double* observation)
  : intrinsic_projection_(new IntrinsicProjection(observation)) {
  }

  template <typename T>
  bool operator()(const T* rotation,
                  const T* translation,
                  const T* intrinsics,
                  const T* point,
                  T* residual) const {
    T transformed_point[3];
    RotateAndTranslatePoint(rotation, translation, point, transformed_point);

    // Note that we call intrinsic_projection_, just like it was
    // any other templated functor.
    return intrinsic_projection_(intrinsics, transformed_point, residual);
  }

 private:
  CostFunctionToFunctor<2,5,3> intrinsic_projection_;
};
```
注意`CostFunctionToFunctor`在构造函数里获得了`CostFunction`的所有权。
在上边的例子中，我们假定`IntrinsicProjection`是一个`CostFunction`，能够评价它的值和导数。如果不是这样，`IntrinsicProjection`定义如下：
```
struct IntrinsicProjection
  IntrinsicProjection(const double* observation) {
    observation_[0] = observation[0];
    observation_[1] = observation[1];
  }

  bool operator()(const double* calibration,
                  const double* point,
                  double* residuals) {
    double projection[2];
    ThirdPartyProjectionFunction(calibration, point, projection);
    residuals[0] = observation_[0] - projection[0];
    residuals[1] = observation_[1] - projection[1];
    return true;
  }
 double observation_[2];
};
```
这里`ThirdPartyProjectionFunction`是我们不能控制的第三方库函数。所以这个函数可以计算它的值，我们想用数值差分来计算它的导数，这种情况下，我们使用`NumericDiffCostFunction`和`CostFunctionToFunctor`来完成这项工作。
```
struct CameraProjection {
  CameraProjection(double* observation)
    intrinsic_projection_(
      new NumericDiffCostFunction<IntrinsicProjection, CENTRAL, 2, 5, 3>(
        new IntrinsicProjection(observation)) {
  }

  template <typename T>
  bool operator()(const T* rotation,
                  const T* translation,
                  const T* intrinsics,
                  const T* point,
                  T* residuals) const {
    T transformed_point[3];
    RotateAndTranslatePoint(rotation, translation, point, transformed_point);
    return intrinsic_projection_(intrinsics, transformed_point, residual);
  }

 private:
  CostFunctionToFunctor<2,5,3> intrinsic_projection_;
};
```
#### DynamicCostFunctionToFunctor
`DynamicCostFunctionToFunctor`提供`CostFunctionToFunctor`一样的功能，对于参数向量和残差的个数和大小在编译期未知的情况。模板函数对象的形式如下：
```
template<typename T>
bool operator()(T const* const* parameters, T* residuals) const;
```
和`CostFunctionToFunctor`给出的例子一样，我们假设：
```
class IntrinsicProjection : public CostFunction {
  public:
    IntrinsicProjection(const double* observation);
    virtual bool Evaluate(double const* const* parameters,
                          double* residuals,
                          double** jacobians) const;
};
```
是一个`CostFunction`实现一个在局部坐标系下的点到它的像平面的投影并从观测点投影减去它。将这个`CostFunction`用在模板函数对象里形式如下：
```
struct CameraProjection {
  CameraProjection(double* observation)
      : intrinsic_projection_(new IntrinsicProjection(observation)) {
  }

  template <typename T>
  bool operator()(T const* const* parameters,
                  T* residual) const {
    const T* rotation = parameters[0];
    const T* translation = parameters[1];
    const T* intrinsics = parameters[2];
    const T* point = parameters[3];

    T transformed_point[3];
    RotateAndTranslatePoint(rotation, translation, point, transformed_point);

    const T* projection_parameters[2];
    projection_parameters[0] = intrinsics;
    projection_parameters[1] = transformed_point;
    return intrinsic_projection_(projection_parameters, residual);
  }

 private:
  DynamicCostFunctionToFunctor intrinsic_projection_;
};
```
像`CostFunctionToFunctor`一样，`DynamicCostFunctionToFunctor`在构造函数里获得了`CostFunction`的所有权。
#### ConditionedCostFunction
这个类允许你对一个封装的代价函数的残差值应用不同的条件。比如说，你有一个代价函数可以得到N个值，但是你想总的代价不是所有这些值的平方和，可能你想对某些值应用不同的尺度，来改变它们对代价的贡献程度。
用法：
```
//  my_cost_function produces N residuals
CostFunction* my_cost_function = ...
CHECK_EQ(N, my_cost_function->num_residuals());
vector<CostFunction*> conditioners;

//  Make N 1x1 cost functions (1 parameter, 1 residual)
CostFunction* f_1 = ...
conditioners.push_back(f_1);

CostFunction* f_N = ...
conditioners.push_back(f_N);
ConditionedCostFunction* ccf =
  new ConditionedCostFunction(my_cost_function, conditioners);
```
## 损失函数
对于最小二乘问题，最小化可能会遇到输入项包含外点的情况（完全错误的测量），这时就需要损失函数来减少它们的影响。   
考虑一个SFM问题，未知项是3D点和相机参数，测量是图像坐标，描述的是相机坐标系下三维点的重投影位置。例如，我们想要建模具有消防栓和车辆的街道场景的几何结构，由一个不知道参数的运动相机观测。我们关心的3D点是消防栓的顶点，图像处理算法负责得到输入到ceres的测量值，可以得到顶点在所有图像帧的观测和匹配情况，但是有一帧把汽车上的点错误当成了消防栓上的点。如果我们不对这个特殊的残差做任何处理，这个错误观测将会导致结果偏离最优。   
利用一个稳健的损失函数，可以减少较大残差的代价，外点项的权重较小就不会过多影响最终结果。
```
class LossFunction {
 public:
  virtual void Evaluate(double s, double out[3]) const = 0;
};
```
关键方法是`LossFunction::Evaluate()`，输入是一个非负标量`s`，计算一个`out`，
$$out = \begin{bmatrix}\rho(s), & \rho'(s), & \rho''(s)\end{bmatrix}$$
这里每项对代价函数的贡献由 $\frac{1}{2}\rho(s)$ 给出，$s = \|f_i\|^2$。   
最合理的 $\rho$满足：
$$\begin{split}\rho(0) &= 0\\
\rho'(0) &= 1\\
\rho'(s) &< 1 \text{ in the outlier region}\\
\rho''(s) &< 0 \text{ in the outlier region}\end{split}$$

## 理论
让我们考虑具有一个参数块的最小二乘问题。
$$\min_x \frac{1}{2}\rho(f^2(x))$$
稳健的梯度和高斯牛顿黑塞矩阵为：
$$\begin{split}g(x) &= \rho'J^\top(x)f(x)\\
H(x) &= J^\top(x)\left(\rho' + 2 \rho''f(x)f^\top(x)\right)J(x)\end{split}$$
这里忽略了$f(x)$ 的二阶梯度。注意如果 $\rho''f(x)^\top f(x) + \frac{1}{2}\rho' < 0$，
那么 $H(x)$ 是非正定的。否则就可以调整残差和雅可比矩阵的权重。

## Problem
`Problem`类实现了稳健的边界约束的非线性最小二乘问题，利用`Problem::AddResidualBlock()`和
`Problem::AddParameterBlock()`来创建最小二乘问题。
例如，一个包含三个大小分别为3,4,5的参数块和大小为2和6的残差块的问题：
```
double x1[] = { 1.0, 2.0, 3.0 };
double x2[] = { 1.0, 2.0, 3.0, 5.0 };
double x3[] = { 1.0, 2.0, 3.0, 6.0, 7.0 };

Problem problem;
problem.AddResidualBlock(new MyUnaryCostFunction(...), x1);
problem.AddResidualBlock(new MyBinaryCostFunction(...), x2, x3);
```
`Problem::AddResidualBlock()`向最小二乘问题中添加一个残差块，它添加一个`CostFunction`，
一个可选的`LossFunction`，并连接`CostFunction`到参数块的集合中。   
代价函数带有它期望的参数块大小的信息，函数会检查那些与在`parameter block`列出的参数块大小
匹配的情况，如果检测到误匹配则程序终止。`loss_function`可为空，这样每一项的代价就只是残差的平方。   

用户可以通过`Problem::AddParameterBlock()`来显式地添加参数块。这会引起额外的正确性检查；
但是`Problem::AddResidualBlock()`隐式地添加参数块（如果不存在的话），所以并不需要显式地
调用`Problem::AddParameterBlock()`。   

`Problem::AddResidualBlock()`显示得添加一个参数块到 `Problem` 中。用户可以选择将一个 `LocalParameterization` 和参数块相关联。重复调用相同的参数会被忽略。重复调用相同的double指针但用不同的大小会导致未定义行为。   

使用`Problem::SetParameterBlockConstant()`可以设置任何参数块为常量，使用`Problem::SetParameterBlockVariable()`来取消。
