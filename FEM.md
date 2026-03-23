# Finite element method

### 第一部分：什么是有限元？ 

$\quad$ 在 FEM 中，最核心的关系式是：

$$
F = K * d
$$

  $&emsp;&emsp; F$: 力（Load）  
  $&emsp;&emsp; K$: 刚度（Stiffness，代表物体抵抗变形的能力）  
  $&emsp;&emsp; d$: 位移（Displacement）  
$\quad$目标通常是：已知力 $F$ 和物体的属性 $K$，求出哪里发生了多大的位移 $d$

### 第二部分：FEM 的“游戏规则” 

1. 虚功原理 (Virtual Work Principle)  
$\quad$ 有限元方法的基础是虚功原理（或最小总势能原理）。简单来说，当一个结构处于平衡状态时，外力在任何微小的“虚位移”上所做的虚功，等于结构内部应力在相应的“虚应变”上所做的内部虚应变能。  
$\quad$ 用矩阵和积分的形式表达，核心的平衡方程推导最终会指向这个公式：  

$$
K^e = \int_V B^T D B \, dV
$$  

$&emsp;&emsp; K^e$：单元刚度矩阵（这是我们最终要计算的目标）  
$&emsp;&emsp; B$：应变-位移矩阵（描述节点位移如何产生内部应变）  
$&emsp;&emsp; D$：本构矩阵（材料的弹性性质矩阵，包括杨氏模量等）  
$&emsp;&emsp; V$：单元的体积  
$\qquad$ 接下来的所有工作，都是为了求出这个积分中的 $B$ 矩阵，并完成积分计算。  

2. 构建桥梁：等参元与自然坐标系 (Isoparametric & Natural Coordinates)  
为了让计算机能够处理任意长度和位置的单元，我们引入了自然坐标系 (Natural Coordinate System) $x$。  
$\quad  \cdot$ 无论实际杆件在全局坐标系 $x$ 中的长度 $L$ 是多少，在自然坐标系中，它的起点永远是 $x = -1$，终点永远是 $x = 1$。  
$\quad  \cdot$ 等参概念 (Iso-parametric) 的意思是：我们用同一套函数来同时插值“几何坐标”和“位移”。  
形函数 (Shape Functions, $N$)：  
$\qquad$ 对于一个两节点的一维线性杆单元（节点 1 和节点 2），形函数定义为：  

$$
N_1 = \frac{1-x}{2}
$$
$$
N_2 = \frac{1+x}{2}
$$ 

$\quad \cdot$ 物理意义：当你在节点 1 ($x$ = -1) 时，$N_1$ = $1$, $N_2$ = 0；当你在节点 2 ($x$ = 1) 时，$N_1$ = $0$, $N_2$ = 1。这满足了“单位分解 (Partition of Unity)”的要求，即 $N_1$ + $N_2$ = 1。  
$\quad$ 通过形函数，杆件内部任意一点的坐标 $x$ 和位移 $u$ 都可以由节点值表示出来：

$$
x = N_1 x_1 + N_2 x_2
$$

$$
u = N_1 u_1 + N_2 u_2
$$  

3. 几何映射：雅可比矩阵 (Jacobian, $J$)  
因为我们现在有两个坐标系（全局坐标 $x$ 和自然坐标 $x$），我们需要一个转换系数，这就是雅可比矩阵 (Jacobian)。对于一维问题，它就是一个标量，表示局部坐标对全局坐标的缩放比例：  

$$ 
J = \frac{dx'}{dx}
$$  

$\quad$ 将前面 $x$ 的表达式代入求导（记住 $x_1$ 和 $x_2$ 是常数，表示节点实际位置，且 $x_2 - x_1 = L$）：  

$$
J = \frac{d}{dx} \left( \frac{1-x}{2} x_1 + \frac{1+x}{2} x_2 \right) = -\frac{x_1}{2} + \frac{x_2}{2} = \frac{x_2 - x_1}{2} = \frac{L}{2}
$$  

$\quad \cdot$ 结论：对于一维线性单元，雅可比 $J = \frac{L}{2}$。  

4. 求解应变与 B 矩阵 (Strain and B-Matrix)  
根据固体力学，应变 $\epsilon$ 是位移 $u$ 对物理坐标 $x$ 的导数。但我们的 $u$ 现在是用 $x$ 表达的，所以需要用链式法则：  

$$
\epsilon = \frac{du}{dx'} = \frac{du}{dx} \cdot \frac{dx}{dx'}
$$

$\quad$ 因为 $J = \frac{dx'}{dx}$，所以 $\frac{dx}{dx'} = \frac{1}{J} = \frac{2}{L}$.  
$\quad$ 现在我们对位移公式 $u = N_1 u_1 + N_2 u_2$ 关于 $x$ 求导：  

$$ 
\frac{du}{dx} = \frac{dN_1}{dx} u_1 + \frac{dN_2}{dx} u_2 = -\frac{1}{2} u_1 + \frac{1}{2} u_2
$$  

$\quad$ 将两部分结合起来求应变：  

$$
\epsilon = \frac{2}{L} \left( -\frac{1}{2} u_1 + \frac{1}{2} u_2 \right) = \left( -\frac{1}{L} \quad \frac{1}{L} \right) \begin{bmatrix} u_1 \\ u_2 \end{bmatrix}
$$

$\quad$ 提取出中间的矩阵，这就是鼎鼎大名的 $B$ 矩阵：  

$$
B = \left( -\frac{1}{L} \quad \frac{1}{L} \right)
$$

5. 组装单元刚度矩阵 (Element Stiffness Matrix)  
现在我们将所有的零件放回一开始的虚功原理积分公式中：  

$$
K^e = \int_{V} B^T E B dV
$$

$\quad$ 对于一维横截面积为 $A$ 的杆件，dV = $A dx$。并且我们把积分变量从 $dx$ 换成 $dx$，需要乘上雅可比 $J$：  

$$
dx = J dx = \frac{L}{2} dx
$$

$\quad$ 积分范围也从物理长度变为了[-1, 1]：  

$$
K^e = \int_{-1}^{1} \begin{bmatrix} -\frac{1}{L} \cr \frac{1}{L} \end{bmatrix} E \begin{bmatrix} -\frac{1}{L} & \frac{1}{L} \end{bmatrix} A \left( \frac{L}{2} \right) dx
$$

$\quad$ 由于积分里的各项 $E$, $A$, $L$ 在这个简单的线性单元中都是常数，我们可以直接把它们提取到积分号外面：  

$$
K^e = E A \left( \frac{L}{2} \right) \begin{bmatrix} \frac{1}{L^2} & -\frac{1}{L^2} \cr -\frac{1}{L^2} & \frac{1}{L^2} \end{bmatrix} \int_{-1}^{1} dx
$$

$\quad$ 由于 $\int_{-1}^{1} dx = 2$：  

$$
K^e = E A \left( \frac{L}{2} \right) \left( \frac{1}{L^2} \right) \begin{bmatrix} 1 & -1 \cr -1 & 1 \end{bmatrix} (2)
$$ 

$\quad$ 化简后，我们就得到了材料力学中最经典的杆单元刚度矩阵：  

$$
K^e = \frac{EA}{L} \begin{bmatrix} 1 & -1 \cr -1 & 1 \end{bmatrix}
$$

### 第三部分：有限元基本推导 

$\quad$ 基本物理链条是：

$$
u(x)\to \varepsilon(x) = \frac{du}{dx} \to \sigma(x) = E \varepsilon(x)
$$

$\quad$ 再通过虚功原理推出单元方程：

$$
K^e*u_i ​= F^e
$$

其中：  
$\qquad \cdot u_i$：单元节点位移向量  
$\qquad \cdot F^e$：单元刚度矩阵  
$\qquad \cdot 𝐹^𝑒$：单元等效节点载荷

#### 3.1Linear Rod Element, 2节点线性杆单元

1. 位移插值的意义  
$\quad$ 2节点线性杆单元内部位移：

$$
u(x) = (1−\frac{x}{L}) u_1 ​+ \frac{x}{L} u_2​
$$



