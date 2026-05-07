# CMP 考试复习：FEM 与 FDM 分离版（中文）

---

## 总览

考试截图可以拆成三个实际能力块：

| 部分 | 来源 | 考试重点 |
|---|---|---|
| Part A: FEM | Pei lecture + Pei MATLAB framework | Tri3/Quad4 推导、机械/热/热力耦合单元、代码填空、BC/loading 修改、COMSOL |
| Part B: FDM | Lars lecture + `Lars Part1` scripts | 1D/2D Poisson/Laplace 差分、矩阵装配、边界行处理、ODE 时间推进、练习题 |
| COMSOL | Pei thermomechanical part | 2D/3D 热力耦合建模、边界条件、结果解释 |

---

# Part A. FEM 复习

## A1. FEM 解题主线

FEM 的标准逻辑：

```text
physical problem
-> strong form
-> weak form
-> shape functions
-> derivative matrix B
-> element matrix/vector
-> global assembly
-> essential/natural BC
-> solve
-> postprocess and validation
```

考试中不要只背代码。要能把公式和代码对应起来：

```text
N, dN, J, B
-> Ke and fe
-> eleDoFs
-> K(eleDoFs,eleDoFs) += Ke
-> F(eleDoFs) += fe
-> Kaa ua = Fa - Kap up
```

---

## A2. FEM 核心知识点

### A2.1 机械场 strong form

```math
\nabla\cdot\sigma+b=0 \quad \text{in } \Omega
```

```math
u=\bar u \quad \text{on } \Gamma_u
```

```math
\sigma n=\bar t \quad \text{on } \Gamma_t
```

### A2.2 热传导 strong form

```math
-\nabla\cdot q=r \quad \text{in } \Omega
```

```math
q=-K\nabla T
```

```math
T=\bar T \quad \text{on } \Gamma_T
```

```math
-q\cdot n=\bar q_n \quad \text{on } \Gamma_q
```

Robin / convection：

```math
-q\cdot n=h(T-T_\infty)
```

### A2.3 机械弱形式

```math
\int_\Omega \delta\varepsilon^T\sigma\,d\Omega
=
\int_\Omega \delta u^T b\,d\Omega
+
\int_{\Gamma_t}\delta u^T\bar t\,d\Gamma
```

### A2.4 热传导弱形式

```math
\int_\Omega \nabla(\delta T)^T K\nabla T\,d\Omega
=
\int_\Omega \delta T r\,d\Omega
+
\int_{\Gamma_q}\delta T\bar q_n\,d\Gamma
```

Robin 边界贡献：

```math
\int_{\Gamma_q}\delta T hT\,d\Gamma
=
\int_{\Gamma_q}\delta T hT_\infty\,d\Gamma
```

因此 Robin 边界有矩阵项和向量项。

---

## A3. FEM 关键公式

### A3.1 二维小应变

```math
\varepsilon=
\begin{bmatrix}
\varepsilon_{xx}\\
\varepsilon_{yy}\\
\gamma_{xy}
\end{bmatrix}
=
\begin{bmatrix}
u_{,x}\\
v_{,y}\\
u_{,y}+v_{,x}
\end{bmatrix}
```

这里使用 engineering shear strain：

```math
\gamma_{xy}=2\varepsilon_{xy}
```

有限元插值：

```math
u \approx N_u d_e,\qquad \varepsilon=B_ud_e
```

### A3.2 弹性矩阵

平面应力：

```math
D=
\frac{E}{1-\nu^2}
\begin{bmatrix}
1&\nu&0\\
\nu&1&0\\
0&0&\frac{1-\nu}{2}
\end{bmatrix}
```

平面应变：

```math
D=
\frac{E}{(1+\nu)(1-2\nu)}
\begin{bmatrix}
1-\nu&\nu&0\\
\nu&1-\nu&0\\
0&0&\frac{1-2\nu}{2}
\end{bmatrix}
```

机械刚度矩阵：

```math
K_{uu}^e=\int_{\Omega_e}B_u^TDB_u\,t\,dA
```

机械体力向量：

```math
f_b^e=\int_{\Omega_e}N_u^Tb\,t\,dA
```

traction 向量：

```math
f_t^e=\int_{\Gamma_t^e}N_u^T\bar t\,t\,d\Gamma
```

von Mises：

```math
\sigma_{vm}
=
\sqrt{\sigma_x^2-\sigma_x\sigma_y+\sigma_y^2+3\tau_{xy}^2}
```

### A3.3 热传导矩阵

```math
T\approx N_T\theta_e,\qquad \nabla T=B_T\theta_e
```

```math
K_{TT}^e=\int_{\Omega_e}B_T^TKB_T\,t\,dA
```

```math
f_r^e=\int_{\Omega_e}N_T^Tr\,t\,dA
```

```math
f_q^e=\int_{\Gamma_q^e}N_T^T\bar q_n\,t\,d\Gamma
```

Robin：

```math
K_\Gamma^e=\int_{\Gamma_q^e}N_T^ThN_T\,t\,d\Gamma
```

```math
f_\Gamma^e=\int_{\Gamma_q^e}N_T^ThT_\infty\,t\,d\Gamma
```

### A3.4 热力耦合

热应变：

```math
\varepsilon_{th}
=
\alpha(T-T_{ref})
\begin{bmatrix}
1\\1\\0
\end{bmatrix}
```

本构：

```math
\sigma=D(\varepsilon-\varepsilon_{th})
```

耦合矩阵：

```math
K_{uT}^e
=
-
\int_{\Omega_e}
B_u^TD
\left(
\alpha
\begin{bmatrix}
1\\1\\0
\end{bmatrix}
N_T
\right)t\,dA
```

单元块系统：

```math
\begin{bmatrix}
K_{uu}^e&K_{uT}^e\\
0&K_{TT}^e
\end{bmatrix}
\begin{bmatrix}
d_e\\
\theta_e
\end{bmatrix}
=
\begin{bmatrix}
f_u^e\\
f_T^e
\end{bmatrix}
```

物理含义：当前 Pei 框架是单向热力耦合，温度影响机械应力；机械位移不反馈到稳态热传导。

---

## A4. Tri3 推导

### A4.1 Tri3 几何

```math
2A=
\det
\begin{bmatrix}
1&x_1&y_1\\
1&x_2&y_2\\
1&x_3&y_3
\end{bmatrix}
```

```math
b_1=y_2-y_3,\quad b_2=y_3-y_1,\quad b_3=y_1-y_2
```

```math
c_1=x_3-x_2,\quad c_2=x_1-x_3,\quad c_3=x_2-x_1
```

```math
N_{i,x}=\frac{b_i}{2A},\qquad N_{i,y}=\frac{c_i}{2A}
```

Tri3 是 constant strain triangle，所以 `B` 为常数。

### A4.2 Tri3 mechanical

自由度：

```text
[ux1 uy1 ux2 uy2 ux3 uy3]^T
```

```math
B_u=
\frac{1}{2A}
\begin{bmatrix}
b_1&0&b_2&0&b_3&0\\
0&c_1&0&c_2&0&c_3\\
c_1&b_1&c_2&b_2&c_3&b_3
\end{bmatrix}
```

```math
K_{uu}^e=tA B_u^TDB_u
```

常体力：

```math
f_b^e=
\frac{tA}{3}
\begin{bmatrix}
b_x\\b_y\\b_x\\b_y\\b_x\\b_y
\end{bmatrix}
```

考试填空顺序：`Area -> b/c -> B -> D -> Ke -> fe`。

### A4.3 Tri3 thermal

自由度：

```text
[T1 T2 T3]^T
```

```math
B_T=
\frac{1}{2A}
\begin{bmatrix}
b_1&b_2&b_3\\
c_1&c_2&c_3
\end{bmatrix}
```

```math
K_{TT}^e=tA B_T^TKB_T
```

常热源：

```math
f_r^e=\frac{tAr}{3}
\begin{bmatrix}
1\\1\\1
\end{bmatrix}
```

### A4.4 Tri3 thermomechanical

自由度：

```text
[ux1 uy1 T1 ux2 uy2 T2 ux3 uy3 T3]^T
```

```math
K_{uu}=tA B_u^TDB_u
```

```math
K_{TT}=tA B_T^TKB_T
```

```math
K_{uT}
=
-t(B_u^TD\alpha[1,1,0]^T)
\left(A
\begin{bmatrix}
1/3&1/3&1/3
\end{bmatrix}
\right)
```

交错自由度索引：

```matlab
mechDofs = [1 2 4 5 7 8];
tempDofs = [3 6 9];
```

---

## A5. Quad4 推导

### A5.1 Quad4 形函数

自然坐标节点：

```text
1 (-1,-1), 2 (1,-1), 3 (1,1), 4 (-1,1)
```

```math
N_1=\frac14(1-s)(1-t)
```

```math
N_2=\frac14(1+s)(1-t)
```

```math
N_3=\frac14(1+s)(1+t)
```

```math
N_4=\frac14(1-s)(1+t)
```

Jacobian：

```math
J=
\begin{bmatrix}
x_{,s}&y_{,s}\\
x_{,t}&y_{,t}
\end{bmatrix}
```

导数变换：

```math
\begin{bmatrix}
N_{,x}\\N_{,y}
\end{bmatrix}
=
J^{-1}
\begin{bmatrix}
N_{,s}\\N_{,t}
\end{bmatrix}
```

2x2 Gauss：

```math
s,t=\pm\frac{1}{\sqrt3},\qquad w=1
```

### A5.2 Quad4 mechanical

```math
B_u=
\begin{bmatrix}
N_{1,x}&0&N_{2,x}&0&N_{3,x}&0&N_{4,x}&0\\
0&N_{1,y}&0&N_{2,y}&0&N_{3,y}&0&N_{4,y}\\
N_{1,y}&N_{1,x}&N_{2,y}&N_{2,x}&N_{3,y}&N_{3,x}&N_{4,y}&N_{4,x}
\end{bmatrix}
```

```math
K_{uu}^e=
\sum_{g=1}^{4}B_u(g)^TDB_u(g)\det J(g)w_gt
```

```math
f_b^e=
\sum_{g=1}^{4}N_u(g)^Tb\,\det J(g)w_gt
```

### A5.3 Quad4 thermal

```math
B_T=
\begin{bmatrix}
N_{1,x}&N_{2,x}&N_{3,x}&N_{4,x}\\
N_{1,y}&N_{2,y}&N_{3,y}&N_{4,y}
\end{bmatrix}
```

```math
K_{TT}^e=
\sum_{g=1}^{4}B_T(g)^TKB_T(g)\det J(g)w_gt
```

### A5.4 Quad4 thermomechanical

自由度：

```text
[ux1 uy1 T1 ux2 uy2 T2 ux3 uy3 T3 ux4 uy4 T4]^T
```

Gauss 循环中：

```math
K_{uu} += B_u^TDB_u\det Jwt
```

```math
K_{TT} += B_T^TKB_T\det Jwt
```

```math
K_{uT} += -B_u^TD\alpha[1,1,0]^TN_T\det Jwt
```

交错索引：

```matlab
mechDofs = [1 2 4 5 7 8 10 11];
tempDofs = [3 6 9 12];
```

---

## A6. FEM 题型方法

### A6.1 推导 stiffness matrix

先判断：

| 关键词 | 公式 |
|---|---|
| displacement / stress / strain | `Kuu = int Bu' D Bu` |
| temperature / heat flux | `KTT = int BT' K BT` |
| thermal expansion | `Kuu, KTT, KuT` |
| Tri3 | 面积闭式积分 |
| Quad4 | `J + detJ + Gauss` |

答题模板：

```text
1. 写插值 u=N d 或 T=N theta
2. 写 epsilon=Bu d 或 gradT=BT theta
3. 代入弱形式
4. 读出 Ke 和 fe
5. Tri3 给闭式，Quad4 给 Gauss 求和式
```

### A6.2 推导 force vector

| 载荷 | 积分区域 | 表达式 |
|---|---|---|
| body force | element area | `int Nu' b dA` |
| heat source | element area | `int NT' r dA` |
| traction | boundary edge | `int Nu' tbar dGamma` |
| heat flux | boundary edge | `int NT' qbar dGamma` |
| Robin | boundary edge | `int NT' h NT` and `int NT' hTinf` |

### A6.3 填 element code

检查矩阵尺寸：

| 单元 | 矩阵尺寸 |
|---|---:|
| Tri3 mechanical | `6 x 6` |
| Quad4 mechanical | `8 x 8` |
| Tri3 thermal | `3 x 3` |
| Quad4 thermal | `4 x 4` |
| Tri3 thermomechanical | `9 x 9` |
| Quad4 thermomechanical | `12 x 12` |

### A6.4 修改 BC/loading 并模拟

机械：

```matlab
model.essBC = {
    leftNodes, [0,0]
};
model.natBC = {
    rightNodes, [tx,ty]
};
```

热：

```matlab
model.essBC = {
    leftNodes, [Tleft];
    rightNodes, [Tright]
};
model.natBC = {
    topNodes, [h, h*Tinf]
};
```

热力耦合：

```matlab
model.essBC = {
    fixedNodes, [0,0,Tfixed]
};
model.natBC = {
    loadedEdge, [tx, ty, h, h*Tinf]
};
```

修改后重新：

```matlab
[K,F] = buildSystem(model);
bcData = buildBCData(model);
[reaction, solution] = solveSystem(K,F,bcData);
```

---

## A7. FEM 关键 MATLAB 代码骨架

### A7.1 Tri3 mechanical

```matlab
A = [1 x1 y1;
     1 x2 y2;
     1 x3 y3];
detA = det(A);
Area = detA/2;

b1 = y2-y3; c1 = x3-x2;
b2 = y3-y1; c2 = x1-x3;
b3 = y1-y2; c3 = x2-x1;

B = [b1 0  b2 0  b3 0;
     0  c1 0  c2 0  c3;
     c1 b1 c2 b2 c3 b3] / detA;

Ke = thickness * Area * (B' * D * B);
fe = thickness * Area/3 * [bx; by; bx; by; bx; by];
```

### A7.2 Quad4 common geometry

```matlab
N = 0.25 * [(1-s)*(1-t);
            (1+s)*(1-t);
            (1+s)*(1+t);
            (1-s)*(1+t)];

dNds = 0.25 * [-(1-t);  (1-t);  (1+t); -(1+t)];
dNdt = 0.25 * [-(1-s); -(1+s);  (1+s);  (1-s)];

J = [dNds'*x, dNds'*y;
     dNdt'*x, dNdt'*y];

detJ = det(J);
gradN = J \ [dNds'; dNdt'];
dNdx = gradN(1,:);
dNdy = gradN(2,:);
```

### A7.3 Thermomechanical blocks

```matlab
epsTh = alpha * [1;1;0];

Kuu = Kuu + Bu' * D * Bu * detJ * w * thickness;
KTT = KTT + BT' * Kcond * BT * detJ * w * thickness;
KuT = KuT - (Bu' * D * epsTh) * N' * detJ * w * thickness;
```

### A7.4 Assembly

```matlab
eleDoFs = getElementDoFs(eleNodeIDs, nNodeDoF);

globalMatrix(eleDoFs, eleDoFs) = ...
    globalMatrix(eleDoFs, eleDoFs) + eleMatrix;
globalVector(eleDoFs) = globalVector(eleDoFs) + eleVector;
```

### A7.5 Essential BC solver

```matlab
activeDoFs = setdiff(allDoFs, prescribedDoFs);
solution(prescribedDoFs) = prescribedValues(prescribedDoFs);

Kaa = K(activeDoFs, activeDoFs);
Kap = K(activeDoFs, prescribedDoFs);
Fa  = F(activeDoFs);
up  = solution(prescribedDoFs);

solution(activeDoFs) = Kaa \ (Fa - Kap*up);
reaction = K*solution - F;
```

---

## A8. Element test 思路

| 测试 | 正确表现 |
|---|---|
| mechanical patch test | 线性位移场产生常应变，Tri3/Quad4 应能通过 |
| thermal linear patch | 线性温度场产生常温度梯度 |
| free thermal expansion | 应力接近零，位移为 `alpha*DeltaT*x/y` |
| restrained thermal expansion | 出现热应力，反力平衡 |
| zero load + zero BC | 解为零 |
| cantilever beam | 反力平衡；Tri3 弯曲偏硬，Quad4 通常更好 |

调试顺序：

```text
area/detJ positive
-> matrix size correct
-> global K no unexpected zero rows
-> enough essential BCs
-> no conflicting BCs
-> reaction equilibrium
-> postprocessed stress/flux trend reasonable
```

---

## A9. Pei COMSOL 2D/3D 热力耦合

### A9.1 2D thermomechanical case

步骤：

```text
1. Geometry: 建 2D 几何
2. Materials: E, nu, alpha, k
3. Physics: Heat Transfer in Solids + Solid Mechanics
4. Multiphysics: Thermal Expansion
5. Thermal BC: T, heat flux, insulation, convection
6. Mechanical BC: fixed, roller/symmetry, boundary load
7. Study: Stationary
8. Mesh: 默认网格 + refinement check
9. Results: T, displacement, von Mises, heat flux
```

要和 MATLAB 一致：

```text
units, thickness, plane stress/strain, Tref, material, BCs
```

常错点：

- 忘记 Thermal Expansion coupling
- `Tref` 不一致
- plane stress/plane strain 不一致
- 固体力学约束不足导致刚体运动
- 过度约束导致热膨胀被人为限制

### A9.2 3D thermomechanical case

3D 注意：

| 项目 | 2D | 3D |
|---|---|---|
| 几何 | 面 | 实体 |
| 力/热边界 | edge/boundary | face |
| 应力状态 | plane stress/strain | full 3D stress |
| 约束 | 去掉 2D 刚体运动 | 去掉 6 个刚体模态 |

检查：

```text
temperature field
-> displacement direction and magnitude
-> stress concentration
-> reaction force balance
-> mesh convergence
```

---

# Part B. FDM 复习（Lars）

## B1. FDM 与 Lars code 范围

本部分对应：

```text
Lecture/Introduction, Series Solutions, Finite Difference
Lars Part1/files
Lars Part1/fun
Lars Part1/L06_Ex
```

重点脚本：

| 文件 | 内容 |
|---|---|
| `laplaceequation.m` | 矩形 Laplace Dirichlet 问题的分离变量级数解 |
| `dirichlet_rectangle.m` | 四边 Dirichlet 的矩形 Laplace 级数叠加 |
| `dirichlet_rectangle_poisson_equation.m` | Poisson 点源 / Green response 级数 |
| `oned_poisson_laplace_dirichlet_BC.m` | 1D Poisson/Laplace Dirichlet FDM |
| `Ex_v03.m` | 2D Poisson/Laplace 五点格式练习 |
| `build_full_grid.m` | 把内部解补成含边界的完整网格 |
| `L06_Ex_3_loop.m` | 1D 温度 + 热弹性位移耦合练习 |
| `forward_euler.m`, `backward_euler.m` | 显式/隐式 Euler |
| `crank_nicolson.m` | Crank-Nicolson |
| `runge_kutta.m`, `rk45.m` | RK4 与 adaptive RK45 |
| `adams_bashforth.m`, `adams_moulton.m`, `bdf.m` | 多步法 |

---

## B2. FDM 核心知识点

FDM 直接离散 strong form 的导数。典型流程：

```text
PDE/ODE
-> grid
-> difference stencil
-> matrix/update formula
-> impose boundary/initial conditions
-> solve or march in time
-> compare error/stability
```

FDM 与 FEM 的区别：

| 项目 | FDM | FEM |
|---|---|---|
| 起点 | strong form derivative approximation | weak form + shape functions |
| 网格 | 规则网格最方便 | 复杂几何更灵活 |
| 核心对象 | stencil / finite difference matrix | element matrix / assembly |
| BC 处理 | 直接改矩阵行或 RHS | essential/natural BC 分开处理 |

---

## B3. ODE 时间推进公式

### B3.1 Cauchy problem

```math
y'(t)=f(t,y),\qquad y(t_0)=y_0
```

### B3.2 Forward Euler

```math
y_{n+1}=y_n+h f(t_n,y_n)
```

代码骨架：

```matlab
for n = 1:N
    y(n+1,:) = y(n,:) + h * f(t(n), y(n,:)')';
end
```

特点：

- explicit
- 一阶
- 对 stiff problem 容易不稳定

### B3.3 Backward Euler

```math
y_{n+1}=y_n+h f(t_{n+1},y_{n+1})
```

Newton 残差：

```math
R(y_{n+1})=y_{n+1}-y_n-hf(t_{n+1},y_{n+1})
```

Jacobian：

```math
I-h\frac{\partial f}{\partial y}
```

代码骨架：

```matlab
res = yn - y(n,:)' - h * f(t(n+1), yn);
jac = eye(length(y0)) - h * dfdy(t(n+1), yn);
delta = jac \ res;
yn = yn - delta;
```

特点：

- implicit
- 一阶
- stiff problem 更稳定

### B3.4 Crank-Nicolson

```math
y_{n+1}=y_n+\frac{h}{2}
\left[
f(t_n,y_n)+f(t_{n+1},y_{n+1})
\right]
```

Newton 残差：

```math
R=y_{n+1}-y_n-\frac{h}{2}
\left[f(t_n,y_n)+f(t_{n+1},y_{n+1})\right]
```

特点：

- implicit
- 二阶
- 对扩散型问题常用

### B3.5 RK4

```math
k_1=h f(t_n,y_n)
```

```math
k_2=h f(t_n+h/2,y_n+k_1/2)
```

```math
k_3=h f(t_n+h/2,y_n+k_2/2)
```

```math
k_4=h f(t_n+h,y_n+k_3)
```

```math
y_{n+1}=y_n+\frac{k_1+2k_2+2k_3+k_4}{6}
```

对应 `runge_kutta.m` 中的 `runge_kutta` 函数。

### B3.6 RK45

RK45 同时算四阶和五阶估计：

```math
err=|u_5-u_4|
```

步长控制：

```math
\delta=0.9\left(\frac{\epsilon}{err}\right)^{1/5}
```

逻辑：

```text
err <= tolerance: accept step
err > tolerance: reject step and reduce h
```

### B3.7 Adams / BDF 多步法

Adams-Bashforth 是 explicit：

```math
y_{n+1}=y_n+h\sum_{j=0}^{p-1}b_j f_{n-j}
```

常用 AB2：

```math
y_{n+1}=y_n+h\left(\frac32 f_n-\frac12 f_{n-1}\right)
```

Adams-Moulton 是 implicit：

```math
y_{n+1}=y_n+h\sum b_j f_{n+1-j}
```

BDF2：

```math
\frac32 y_n-2y_{n-1}+\frac12 y_{n-2}=h f(t_n,y_n)
```

考试逻辑：

1. 多步法需要历史值
2. 前几步用 RK4 / ode45 / Euler 启动
3. explicit 不需要迭代
4. implicit 要 Newton / fixed point
5. stiff problem 通常考虑 implicit / BDF

---

## B4. 稳定性、相容性、收敛

### B4.1 Local truncation error

方法阶数由 local truncation error 判断。Forward Euler 是一阶，Heun/Crank-Nicolson 是二阶，RK4 是四阶。

### B4.2 Zero-stability 与 convergence

Lars lecture 的核心关系：

```text
consistency + zero-stability -> convergence
```

### B4.3 Absolute stability

test problem：

```math
y'=\lambda y
```

Forward Euler：

```math
y_{n+1}=(1+h\lambda)y_n
```

稳定条件：

```math
|1+h\lambda|<1
```

Backward Euler：

```math
y_{n+1}=\frac{1}{1-h\lambda}y_n
```

stiffness 的直观判断：系统特征值尺度差很大，显式法步长被最快衰减模态限制，但慢模态才是你真正关心的响应。

---

## B5. 1D Poisson/Laplace FDM

### B5.1 标准题型

```math
u''(x)=f(x),\qquad u(0)=u_0,\quad u(L)=u_L
```

网格：

```math
h=\frac{L}{N+1},\qquad x_i=ih,\quad i=1,\ldots,N
```

中心差分：

```math
u''(x_i)\approx
\frac{u_{i-1}-2u_i+u_{i+1}}{h^2}
```

矩阵行：

```text
[... 1/h^2  -2/h^2  1/h^2 ...]
```

边界并入 RHS：

```matlab
rhs(1)   = rhs(1)   - u0/h^2;
rhs(end) = rhs(end) - uL/h^2;
```

代码骨架：

```matlab
L = 5;
N = 50;
h = L/(N+1);
x = (1:N)'*h;

e = ones(N,1);
A = spdiags([e -2*e e],[-1 0 1],N,N) / h^2;

rhs = f(x);
rhs(1)   = rhs(1)   - u0/h^2;
rhs(end) = rhs(end) - uL/h^2;

U = A\rhs;
Ufull = [u0; U; uL];
```

### B5.2 练习：`L=5, f=-20, u(0)=u(L)=0`

Lecture 6 练习的解析检查：

```math
u(x)=-10x^2+10Lx
```

解题逻辑：

```text
1. 建 1D 网格
2. 用三点中心差分离散 u''
3. Dirichlet 边界进入 RHS 或整行替换
4. 解 A\RHS
5. 与 u_exact 比较
```

---

## B6. 2D Poisson/Laplace FDM

### B6.1 标准题型

```math
u_{xx}+u_{yy}=f(x,y)
```

矩形区域：

```math
(0,a)\times(0,b)
```

网格：

```math
h_x=\frac{a}{N_x+1},\qquad h_y=\frac{b}{N_y+1}
```

五点格式：

```math
\frac{u_{i-1,j}-2u_{i,j}+u_{i+1,j}}{h_x^2}
+
\frac{u_{i,j-1}-2u_{i,j}+u_{i,j+1}}{h_y^2}
=f_{i,j}
```

中心系数：

```math
-\frac{2}{h_x^2}-\frac{2}{h_y^2}
```

邻居系数：

```math
\frac{1}{h_x^2},\qquad \frac{1}{h_y^2}
```

### B6.2 一维编号

`Ex_v03.m` 使用 x fastest：

```matlab
k = i + (j-1)*Nx;
```

矩阵填充：

```matlab
A(k,k) = -2*invhx2 - 2*invhy2;

if i < Nx
    A(k,idx(i+1,j)) = invhx2;
end
if i > 1
    A(k,idx(i-1,j)) = invhx2;
end
if j < Ny
    A(k,idx(i,j+1)) = invhy2;
end
if j > 1
    A(k,idx(i,j-1)) = invhy2;
end
```

### B6.3 Dirichlet 边界进入 RHS

如果顶边：

```math
u(x,b)=g_{top}(x)
```

对应上邻居在边界时：

```matlab
rhs(k) = rhs(k) - invhy2*gTop(x(i));
```

若边界为零，贡献为零。

### B6.4 Lecture 6 练习

项目脚本 `Ex_v03.m` 的设置：

```matlab
a = 1; b = 2;
Nx = 2; Ny = 3;
gTop = @(x) 4*(-x.^2 + x);
```

先解 Laplace：

```math
f(x,y)=0
```

再解 Poisson：

```math
f(x,y)=-10
```

方法：

```text
1. 组装同一个 A
2. Laplace RHS = 边界贡献
3. Poisson RHS = 边界贡献 + f
4. 解 U=A\rhs
5. reshape(U,[Nx,Ny])
6. 用 build_full_grid 加回边界并画图
```

注意：`A` 只由网格和 PDE operator 决定；如果只改变 `f`，可以复用 `A`。

---

## B7. 分离变量与矩形 Laplace/Poisson 级数

这部分来自 Lars lecture 3 和 `laplaceequation.m`、`dirichlet_rectangle.m`、`dirichlet_rectangle_poisson_equation.m`。

### B7.1 顶边非零、其余三边为零的 Laplace 问题

```math
\nabla^2u=0,\quad 0<x<a,\;0<y<b
```

```math
u(x,0)=0,\quad u(0,y)=0,\quad u(a,y)=0,\quad u(x,b)=g(x)
```

分离变量结果：

```math
u(x,y)=
\sum_{n=1}^{\infty}
B_n\sin\left(\frac{n\pi x}{a}\right)
\sinh\left(\frac{n\pi y}{a}\right)
```

系数：

```math
B_n=
\frac{2}{a\sinh(n\pi b/a)}
\int_0^a
g(x)\sin\left(\frac{n\pi x}{a}\right)dx
```

MATLAB 骨架：

```matlab
for n = 1:N
    In = integral(@(xx) g(xx).*sin(n*pi*xx/a), 0, a);
    Bn(n) = 2/(a*sinh(n*pi*b/a)) * In;
end

U = zeros(size(X));
for n = 1:N
    U = U + Bn(n)*sin(n*pi*X/a).*sinh(n*pi*Y/a);
end
```

### B7.2 四边非齐次 Dirichlet

思路：superposition。

```math
u=u_{top}+u_{bottom}+u_{left}+u_{right}
```

每个子问题只保留一条非零边界，其余三边设为零，然后用对应的正弦/双曲正弦展开。

### B7.3 Poisson 点源 / Green response

齐次 Dirichlet：

```math
\nabla^2u=-Q\delta(x-x_0,y-y_0)
```

级数形式：

```math
u(x,y)=
\sum_{m=1}^{\infty}\sum_{n=1}^{\infty}
E_{mn}
\sin\left(\frac{m\pi x}{a}\right)
\sin\left(\frac{n\pi y}{b}\right)
```

```math
E_{mn}
=
\frac{4Q}{ab\lambda_{mn}}
\sin\left(\frac{m\pi x_0}{a}\right)
\sin\left(\frac{n\pi y_0}{b}\right)
```

```math
\lambda_{mn}=
\left(\frac{m\pi}{a}\right)^2+
\left(\frac{n\pi}{b}\right)^2
```

考试思路：

```text
1. 判断边界是否齐次
2. 选 sin 模态满足零 Dirichlet
3. 对边界函数或源项做 Fourier 投影
4. 截断到 N 或 mMax,nMax
5. 讨论截断误差和源点附近尖峰
```

---

## B8. 1D 热-机械耦合练习（L06）

项目脚本：`L06_Ex_3_loop.m`

### B8.1 温度问题

脚本使用：

```math
u''=f,\qquad u(0)=u(L)=0,\qquad f=-20
```

离散：

```matlab
A(j,j-1:j+1) = (1/h^2)*[1 -2 1];
rhs_u(j) = fConst;
```

### B8.2 位移问题

Lecture 练习把温度结果带入机械方程：

```math
w''=\alpha u'
```

边界：

```math
w(0)=w(L)=0
```

离散：

```matlab
Aw(j,j-1:j+1) = (1/h^2)*[1 -2 1];
uPrime = (u(j+1)-u(j-1))/(2*h);
rhs_w(j) = alpha*uPrime;
```

### B8.3 应力

```math
\sigma=E(w'-\alpha u)
```

中心差分：

```matlab
wPrime = (w(j+1)-w(j-1))/(2*h);
sigma(j) = E*(wPrime - alpha*u(j));
```

解析检查：

```math
u(x)=-10x^2+10Lx
```

```math
w(x)=
\alpha
\left(
-\frac{20}{6}x^3
+5Lx^2
-5L^2x
+\frac{20}{6}L^2x
\right)
```

解题逻辑：

```text
1. 先求温度 u
2. 用中心差分求 u'
3. 把 alpha*u' 放入位移方程 RHS
4. 求 w
5. 用中心差分求 w'
6. 计算 stress = E*(w' - alpha*u)
7. 与解析解比较
```

---

## B9. FDM 练习题型总结

### B9.1 题型 1：给 ODE，写时间推进代码

步骤：

```text
1. 确认 explicit/implicit
2. explicit 直接更新
3. implicit 写 residual
4. 用 Newton 或解析根求 y_{n+1}
5. 和 exact/ode45 比较
6. 改 h 观察误差或不稳定
```

重点方法：

```text
Forward Euler, Backward Euler, Crank-Nicolson, RK4, RK45, Adams, BDF
```

### B9.2 题型 2：1D BVP 差分矩阵

步骤：

```text
1. 只把 interior unknowns 放进 U
2. 用三点中心差分写三对角矩阵
3. Dirichlet 边界移入 RHS
4. 解 A\RHS
5. full solution = [leftBC; U; rightBC]
```

### B9.3 题型 3：2D Poisson/Laplace 五点格式

步骤：

```text
1. 建 interior grid
2. 定义 index k=i+(j-1)*Nx
3. 对每个 interior node 写 center/right/left/up/down 系数
4. 非零 Dirichlet boundary 进入 RHS
5. solve
6. reshape and plot
```

### B9.4 题型 4：Laplace 矩形级数解

步骤：

```text
1. 判断哪条边非零
2. 写对应 sin-sinh 模态
3. 用 integral 计算 Fourier coefficient
4. 截断求和
5. 如果四边都非零，用 superposition
```

### B9.5 题型 5：温度-机械一维耦合

步骤：

```text
1. solve thermal Poisson
2. differentiate temperature with central difference
3. assemble displacement equation
4. solve displacement
5. compute stress
6. compare with analytic checks
```

---

## B10. FDM 常见错误

1. `hx` 和 `hy` 不同却用同一个 `h`
2. 2D 索引 `k=i+(j-1)*Nx` 写错
3. 非零 Dirichlet 边界忘记移到 RHS
4. Poisson 的 `f=-10` 符号和矩阵符号不一致
5. `reshape(U,[Nx,Ny])` 后转置/画图方向搞错
6. implicit ODE 没有求解 nonlinear equation，只当 explicit 用
7. 多步法没有足够启动值
8. stiff problem 用 Forward Euler 但步长太大
9. 热-机械耦合中 `u'` 或 `w'` 的中心差分边界处理不当
10. 级数解的边界函数投影系数漏掉 `sinh(n*pi*b/a)` 分母

---

## B11. FDM 考前速查

### B11.1 1D stencil

```math
u''_i\approx\frac{u_{i-1}-2u_i+u_{i+1}}{h^2}
```

### B11.2 2D stencil

```math
\frac{u_{i-1,j}-2u_{i,j}+u_{i+1,j}}{h_x^2}
+
\frac{u_{i,j-1}-2u_{i,j}+u_{i,j+1}}{h_y^2}
=f_{i,j}
```

### B11.3 Dirichlet RHS contribution

```text
known boundary neighbor coefficient * boundary value
move to RHS with minus sign
```

### B11.4 ODE implicit residual

Backward Euler：

```math
R=y_{n+1}-y_n-hf(t_{n+1},y_{n+1})
```

Crank-Nicolson：

```math
R=y_{n+1}-y_n-\frac{h}{2}[f_n+f_{n+1}]
```

### B11.5 最后记忆

FEM：

```text
weak form -> element matrix -> assembly
```

FDM：

```text
strong form -> stencil -> matrix row / time update
```
