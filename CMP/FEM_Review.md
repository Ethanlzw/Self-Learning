# CMP 第二部分 FEM 考试复习文档（中文）

## 1. 考试范围总览

截图中的考试结构可以理解为三块：

| 模块 | 重点 | 你需要会做什么 |
|---|---|---|
| Lars code | PDE、series solution、finite difference、ODE/FDM 代码 | 看懂差分离散、矩阵装配、边界行替换、稳定性和误差逻辑 |
| Pei code | FEM MATLAB 框架 | 推导/补全 Tri3、Quad4 的机械、热、热力耦合单元；会装配、施加 BC、求解和后处理 |
| Pei COMSOL | 2D/3D thermomechanical case | 在 COMSOL 设置热传导 + 固体力学 + thermal expansion，正确施加边界条件并解释结果 |

Code 的截图细分如下：

1. 推导 element stiffness matrix 或 force vector  
   可能单元：Tri3 / Quad4  
   可能物理场：mechanical / thermal / thermomechanical
2. 填代码中的 element routine、assembly、natural BC、solver、postprocess 片段
3. 测试 element 是否正确
4. 用代码模拟一个问题
5. 改变 BC 或 loading 后重新模拟

考试最核心的一句话：

```text
physical problem
-> strong form
-> weak form
-> shape functions
-> B matrix
-> element matrix/vector
-> global assembly
-> boundary conditions
-> solve
-> postprocess/check
```

---

## 2. 必背 FEM 主线

### 2.1 强形式到弱形式

机械平衡：

```math
\nabla \cdot \sigma + b = 0 \quad \text{in } \Omega
```

边界：

```math
u = \bar u \quad \text{on } \Gamma_u
```

```math
\sigma n = \bar t \quad \text{on } \Gamma_t
```

热传导：

```math
-\nabla \cdot q = r \quad \text{in } \Omega
```

```math
q = -K \nabla T
```

热边界：

```math
T = \bar T \quad \text{on } \Gamma_T
```

```math
-q\cdot n = \bar q_n \quad \text{on } \Gamma_q
```

Robin / convection：

```math
-q\cdot n = h(T-T_\infty)
```

### 2.2 弱形式

机械弱形式：

```math
\int_\Omega \delta \varepsilon^T \sigma \, d\Omega
=
\int_\Omega \delta u^T b \, d\Omega
+
\int_{\Gamma_t} \delta u^T \bar t \, d\Gamma
```

热传导弱形式：

```math
\int_\Omega \nabla(\delta T)^T K \nabla T \, d\Omega
=
\int_\Omega \delta T r\, d\Omega
+
\int_{\Gamma_q} \delta T \bar q_n\, d\Gamma
```

Robin 边界移到左边后：

```math
\int_{\Gamma_q} \delta T h T\,d\Gamma
=
\int_{\Gamma_q} \delta T hT_\infty\,d\Gamma
```

---

## 3. 关键公式总表

### 3.1 二维小应变机械场

```math
\varepsilon =
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

其中：

```math
\gamma_{xy}=2\varepsilon_{xy}
```

有限元中：

```math
u \approx N_u d_e,\qquad \varepsilon = B_u d_e
```

材料关系：

```math
\sigma = D\varepsilon
```

平面应力：

```math
D =
\frac{E}{1-\nu^2}
\begin{bmatrix}
1 & \nu & 0\\
\nu & 1 & 0\\
0 & 0 & \frac{1-\nu}{2}
\end{bmatrix}
```

平面应变：

```math
D =
\frac{E}{(1+\nu)(1-2\nu)}
\begin{bmatrix}
1-\nu & \nu & 0\\
\nu & 1-\nu & 0\\
0 & 0 & \frac{1-2\nu}{2}
\end{bmatrix}
```

机械单元刚度：

```math
K_{uu}^e =
\int_{\Omega_e} B_u^T D B_u \, t\,dA
```

体力向量：

```math
f_b^e =
\int_{\Omega_e} N_u^T b\,t\,dA
```

traction 边界向量：

```math
f_t^e =
\int_{\Gamma_t^e} N_u^T \bar t\,t\,d\Gamma
```

二维 von Mises 应力：

```math
\sigma_{vm}
=
\sqrt{\sigma_x^2-\sigma_x\sigma_y+\sigma_y^2+3\tau_{xy}^2}
```

### 3.2 稳态热传导

插值：

```math
T \approx N_T \theta_e,\qquad \nabla T = B_T\theta_e
```

Fourier law：

```math
q = -K\nabla T
```

导热矩阵：

```math
K_{TT}^e =
\int_{\Omega_e} B_T^T K B_T\,t\,dA
```

体热源：

```math
f_r^e =
\int_{\Omega_e} N_T^T r\,t\,dA
```

热流边界：

```math
f_q^e =
\int_{\Gamma_q^e} N_T^T \bar q_n\,t\,d\Gamma
```

Robin 边界：

```math
K_\Gamma^e =
\int_{\Gamma_q^e} N_T^T hN_T\,t\,d\Gamma
```

```math
f_\Gamma^e =
\int_{\Gamma_q^e} N_T^T hT_\infty\,t\,d\Gamma
```

### 3.3 热力耦合

热应变：

```math
\varepsilon_{th}
=
\alpha (T-T_{ref})
\begin{bmatrix}
1\\
1\\
0
\end{bmatrix}
```

热弹性本构：

```math
\sigma = D(\varepsilon-\varepsilon_{th})
```

温度到位移方程的耦合矩阵：

```math
K_{uT}^e
=
-
\int_{\Omega_e}
B_u^T D
\left(
\alpha
\begin{bmatrix}
1\\1\\0
\end{bmatrix}
N_T
\right)
t\,dA
```

单元块系统：

```math
\begin{bmatrix}
K_{uu}^e & K_{uT}^e\\
0 & K_{TT}^e
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

整体系统：

```math
\begin{bmatrix}
K_{uu} & K_{uT}\\
0 & K_{TT}
\end{bmatrix}
\begin{bmatrix}
d\\
\theta
\end{bmatrix}
=
\begin{bmatrix}
f_u\\
f_T
\end{bmatrix}
```

物理含义：当前代码是单向耦合，温度产生热应变并影响应力；位移不反过来改变稳态导热方程。

---

## 4. Tri3 单元推导逻辑

### 4.1 几何与形函数

节点：

```math
(x_1,y_1),\quad (x_2,y_2),\quad (x_3,y_3)
```

面积：

```math
2A =
\det
\begin{bmatrix}
1 & x_1 & y_1\\
1 & x_2 & y_2\\
1 & x_3 & y_3
\end{bmatrix}
```

定义：

```math
b_1=y_2-y_3,\quad b_2=y_3-y_1,\quad b_3=y_1-y_2
```

```math
c_1=x_3-x_2,\quad c_2=x_1-x_3,\quad c_3=x_2-x_1
```

形函数导数：

```math
\frac{\partial N_i}{\partial x}=\frac{b_i}{2A},
\qquad
\frac{\partial N_i}{\partial y}=\frac{c_i}{2A}
```

Tri3 是 constant strain triangle，`B` 在单元内为常数。

### 4.2 Tri3 mechanical

局部自由度：

```text
[ux1 uy1 ux2 uy2 ux3 uy3]^T
```

```math
B_u =
\frac{1}{2A}
\begin{bmatrix}
b_1 & 0 & b_2 & 0 & b_3 & 0\\
0 & c_1 & 0 & c_2 & 0 & c_3\\
c_1 & b_1 & c_2 & b_2 & c_3 & b_3
\end{bmatrix}
```

```math
K_{uu}^e=tA B_u^T D B_u
```

常体力：

```math
f_b^e =
\frac{tA}{3}
\begin{bmatrix}
b_x\\b_y\\b_x\\b_y\\b_x\\b_y
\end{bmatrix}
```

答题步骤：

1. 写 `A` 或 `detA=2A`
2. 算 `b_i,c_i`
3. 写 `B_u`
4. 选 plane stress / plane strain 的 `D`
5. 写 `K=t*A*B'*D*B`
6. 若有常体力，每个节点分到 `A/3`

### 4.3 Tri3 thermal

局部自由度：

```text
[T1 T2 T3]^T
```

```math
B_T =
\frac{1}{2A}
\begin{bmatrix}
b_1 & b_2 & b_3\\
c_1 & c_2 & c_3
\end{bmatrix}
```

```math
K_{TT}^e=tA B_T^T K B_T
```

常热源：

```math
f_r^e =
\frac{tA r}{3}
\begin{bmatrix}
1\\1\\1
\end{bmatrix}
```

### 4.4 Tri3 thermomechanical

局部自由度：

```text
[ux1 uy1 T1 ux2 uy2 T2 ux3 uy3 T3]^T
```

块矩阵：

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
1/3 & 1/3 & 1/3
\end{bmatrix}
\right)
```

注意 `K_uT` 是 `6 x 3`，最后要放入交错自由度位置：

```text
mechDofs = [1 2 4 5 7 8]
tempDofs = [3 6 9]
```

最常见错误：

- 把 `B_u` 和 `B_T` 混用
- 忘记 `KuT` 的负号
- 忘记自由度是按节点交错排列，不是先全部位移再全部温度
- `Tref` 不为零时漏掉参考温度项

---

## 5. Quad4 单元推导逻辑

### 5.1 自然坐标与形函数

节点顺序：

```text
1: (-1,-1)
2: ( 1,-1)
3: ( 1, 1)
4: (-1, 1)
```

形函数：

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

导数：

```math
\frac{\partial N}{\partial s},\quad
\frac{\partial N}{\partial t}
```

Jacobian：

```math
J=
\begin{bmatrix}
x_{,s} & y_{,s}\\
x_{,t} & y_{,t}
\end{bmatrix}
```

坐标变换：

```math
\begin{bmatrix}
N_{,x}\\
N_{,y}
\end{bmatrix}
=
J^{-1}
\begin{bmatrix}
N_{,s}\\
N_{,t}
\end{bmatrix}
```

2x2 Gauss 点：

```math
s,t=\pm\frac{1}{\sqrt3},\qquad w=1
```

### 5.2 Quad4 mechanical

局部自由度：

```text
[ux1 uy1 ux2 uy2 ux3 uy3 ux4 uy4]^T
```

```math
B_u =
\begin{bmatrix}
N_{1,x} & 0 & N_{2,x} & 0 & N_{3,x} & 0 & N_{4,x} & 0\\
0 & N_{1,y} & 0 & N_{2,y} & 0 & N_{3,y} & 0 & N_{4,y}\\
N_{1,y} & N_{1,x} & N_{2,y} & N_{2,x} & N_{3,y} & N_{3,x} & N_{4,y} & N_{4,x}
\end{bmatrix}
```

数值积分：

```math
K_{uu}^e
=
\sum_{g=1}^4
B_u(g)^T D B_u(g)\det J(g)w_g t
```

体力：

```math
f_b^e
=
\sum_{g=1}^4
N_u(g)^T b \det J(g)w_g t
```

### 5.3 Quad4 thermal

```math
B_T =
\begin{bmatrix}
N_{1,x} & N_{2,x} & N_{3,x} & N_{4,x}\\
N_{1,y} & N_{2,y} & N_{3,y} & N_{4,y}
\end{bmatrix}
```

```math
K_{TT}^e
=
\sum_{g=1}^4
B_T(g)^TKB_T(g)\det J(g)w_g t
```

```math
f_r^e
=
\sum_{g=1}^4
N(g)^T r\det J(g)w_g t
```

### 5.4 Quad4 thermomechanical

每个节点自由度：

```text
[ux, uy, T]
```

局部自由度：

```text
[ux1 uy1 T1 ux2 uy2 T2 ux3 uy3 T3 ux4 uy4 T4]^T
```

Gauss 循环中同时累加：

```math
K_{uu} += B_u^TDB_u\det Jwt
```

```math
K_{TT} += B_T^TKB_T\det Jwt
```

```math
K_{uT} += -B_u^TD\alpha[1,1,0]^TN_T\det Jwt
```

放入交错矩阵：

```text
mechDofs = [1 2 4 5 7 8 10 11]
tempDofs = [3 6 9 12]
```

---

## 6. 边界单元与载荷向量

### 6.1 机械 traction 边界

二节点边：

```math
N_1=\frac{1-s}{2},\qquad N_2=\frac{1+s}{2}
```

```math
J_\Gamma = L/2
```

```math
f_t^e
=
\int_{-1}^1 N_u^T\bar t\,t\,J_\Gamma ds
```

常 traction 时：

```math
f_t^e =
\frac{tL}{2}
\begin{bmatrix}
t_x\\t_y\\t_x\\t_y
\end{bmatrix}
```

### 6.2 热 Neumann / Robin 边界

代码里热边界常写成 `[M,S]`：

```math
K_\Gamma^e = \int_\Gamma N^T M N\,t\,d\Gamma
```

```math
f_\Gamma^e = \int_\Gamma N^T S\,t\,d\Gamma
```

对应关系：

| 物理边界 | `M` | `S` |
|---|---:|---:|
| 绝热 | 0 | 0 |
| 指定热流 `q_bar` | 0 | `q_bar` |
| 对流 `h(T-T_inf)` | `h` | `h*T_inf` |

### 6.3 热力耦合边界

热力耦合的边界输入常为：

```text
[tx, ty, M, S]
```

机械 traction 只贡献向量；热 Robin 贡献矩阵和向量。

---

## 7. 关键 MATLAB 脚本地图

### 7.1 Mechanical + thermal 框架

目录：

```text
completed_fem_mech_thermal
Frame/Mech_Thermal_Frame
```

| 文件 | 考试作用 | 需要掌握的核心 |
|---|---|---|
| `Tri3_Mech.m` | Tri3 力学单元 | 面积、`b_i,c_i`、`B_u`、`K=t*A*B'*D*B`、体力 |
| `Quad4_Mech.m` | Quad4 力学单元 | `N,dNds,dNdt,J,detJ,B_u`，2x2 Gauss 积分 |
| `Tri3_Thermal.m` | Tri3 热单元 | `B_T`、`K=t*A*B'*Kcond*B`、热源 |
| `Quad4_Thermal.m` | Quad4 热单元 | `J`、`B_T`、2x2 Gauss 积分 |
| `Edge2_MechTraction.m` | 机械自然边界 | 线积分，常 traction 每端一半 |
| `Edge2_Thermal.m` | 热自然/Robin 边界 | `N'*M*N` 和 `N'*S` |
| `assembleSystem.m` | 全局装配 | `eleDoFs` 映射，`K(I,I)+=Ke`，`F(I)+=fe` |
| `buildBCData.m` | Essential BC | node + local dof -> global dof |
| `solveSystem.m` | 约束自由度求解 | `Kaa ua = Fa - Kap up` |
| `post_*` | 后处理 | 应变、应力、热流、von Mises |

### 7.2 Thermomechanical 框架

目录：

```text
completed_thermomechanical
Frame/Thermomechanical_Frame
```

| 文件 | 考试作用 | 需要掌握的核心 |
|---|---|---|
| `Tri3_ThermoMech.m` | Tri3 热力耦合 | `Kuu,KTT,KuT` 和交错自由度 |
| `Quad4_ThermoMech.m` | Quad4 热力耦合 | 同一 Gauss 循环中构造三个块矩阵 |
| `Edge2_ThermoMech.m` | 热力耦合边界 | `[tx,ty,M,S]` 同时处理力和热边界 |
| `assembleNaturalBC_ThermoMech.m` | 自然边界装配 | 找边、算边界矩阵/向量、装配进全局 |
| `post_Quad4_ThermoMech.m` | 后处理 | `strain, thermalStrain, stress, gradT, heatFlux` |
| `main_mixed_thermomech.m` | 模拟入口 | 定义网格、材料、BC、载荷、求解和画图 |

---

## 8. 关键代码片段

这些不是完整代码，而是考试填空时最该记住的“骨架”。

### 8.1 Tri3 mechanical

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

### 8.2 Tri3 thermal

```matlab
B = [b1 b2 b3;
     c1 c2 c3] / detA;

Kcond = [kxx 0;
         0   kyy];

Ke = thickness * Area * (B' * Kcond * B);
fe = thickness * Area/3 * qv * [1;1;1];
```

### 8.3 Quad4 Gauss loop

```matlab
gaussPts = [-1/sqrt(3), -1/sqrt(3);
            -1/sqrt(3),  1/sqrt(3);
             1/sqrt(3), -1/sqrt(3);
             1/sqrt(3),  1/sqrt(3)];

for igp = 1:4
    s = gaussPts(igp,1);
    t = gaussPts(igp,2);

    N = 0.25 * [(1-s)*(1-t);
                (1+s)*(1-t);
                (1+s)*(1+t);
                (1-s)*(1+t)];

    dNds = 0.25 * [-(1-t);  (1-t);  (1+t); -(1+t)];
    dNdt = 0.25 * [-(1-s); -(1+s);  (1+s);  (1-s)];

    J = [dNds'*x, dNds'*y;
         dNdt'*x, dNdt'*y];

    gradN = J \ [dNds'; dNdt'];
    dNdx = gradN(1,:);
    dNdy = gradN(2,:);
end
```

### 8.4 Quad4 mechanical `B_u`

```matlab
Bu = [dNdx(1) 0       dNdx(2) 0       dNdx(3) 0       dNdx(4) 0;
      0       dNdy(1) 0       dNdy(2) 0       dNdy(3) 0       dNdy(4);
      dNdy(1) dNdx(1) dNdy(2) dNdx(2) dNdy(3) dNdx(3) dNdy(4) dNdx(4)];
```

### 8.5 Quad4 thermal `B_T`

```matlab
BT = [dNdx;
      dNdy];
```

### 8.6 Thermomechanical blocks

```matlab
epsTh = alpha * [1;1;0];

Kuu = Kuu + Bu' * D * Bu * detJ * w * thickness;
KTT = KTT + BT' * Kcond * BT * detJ * w * thickness;
KuT = KuT - (Bu' * D * epsTh) * N' * detJ * w * thickness;
```

### 8.7 全局装配

```matlab
eleDoFs = getElementDoFs(eleNodeIDs, nNodeDoF);

globalMatrix(eleDoFs, eleDoFs) = ...
    globalMatrix(eleDoFs, eleDoFs) + eleMatrix;

globalVector(eleDoFs) = globalVector(eleDoFs) + eleVector;
```

### 8.8 Essential BC 求解

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

## 9. 不同题型的解题思路

### 9.1 题型 A：推导单元刚度矩阵

判断物理场：

| 题目关键词 | 用哪个公式 |
|---|---|
| displacement, stress, strain, elasticity | mechanical `Kuu = ∫Bu' D Bu` |
| temperature, heat conduction, heat flux | thermal `KTT = ∫BT' K BT` |
| thermal expansion, thermoelasticity | thermomechanical `Kuu,KTT,KuT` |

判断单元类型：

| 单元 | 推导方式 |
|---|---|
| Tri3 | 直接用面积公式，`B` 常数，可以手算闭式 |
| Quad4 | 必须用自然坐标、Jacobian、Gauss 积分 |
| Edge2 | 一维线积分，`J=L/2` |

标准作答结构：

1. 写插值 `u=N d` 或 `T=N theta`
2. 写导数关系 `epsilon=B_u d` 或 `gradT=B_T theta`
3. 代入弱形式
4. 读出 `K_e` 和 `f_e`
5. 对 Tri3 给闭式；对 Quad4 给 Gauss 求和式

### 9.2 题型 B：推导 force vector

先问自己“载荷在哪里”：

| 载荷 | 积分区域 | 公式 |
|---|---|---|
| body force `b` | 面积/体积 | `∫ N_u^T b dΩ` |
| heat source `r` | 面积/体积 | `∫ N_T^T r dΩ` |
| traction `t_bar` | 边界 | `∫ N_u^T t_bar dΓ` |
| heat flux `q_bar` | 边界 | `∫ N_T^T q_bar dΓ` |
| convection | 边界 | matrix `∫N^T hN` + vector `∫N^T hT_inf` |

常见陷阱：

- 把边界 traction 当成面积力
- 热 Robin 边界只写向量，忘记矩阵
- 忘记 thickness
- 常载荷下 Tri3 每个节点是 `A/3`，Edge2 每个端点是 `L/2`

### 9.3 题型 C：填 element code

解题顺序：

1. 看文件名判断物理场和单元：`Tri3_Mech`、`Quad4_Thermal`、`Quad4_ThermoMech`
2. 看局部自由度顺序
3. 写 `N`、`dN`、`J`、`B`
4. 累加 `eleMatrix` 和 `eleVector`
5. 检查矩阵尺寸

尺寸检查非常重要：

| 单元 | `eleMatrix` 尺寸 |
|---|---:|
| Tri3 mechanical | `6 x 6` |
| Quad4 mechanical | `8 x 8` |
| Tri3 thermal | `3 x 3` |
| Quad4 thermal | `4 x 4` |
| Tri3 thermomechanical | `9 x 9` |
| Quad4 thermomechanical | `12 x 12` |

### 9.4 题型 D：填 assembly / solver code

assembly 的本质：

```text
local element dofs -> global dofs
global K(I,I) += Ke
global F(I) += fe
```

solver 的本质：

```math
\begin{bmatrix}
K_{aa} & K_{ap}\\
K_{pa} & K_{pp}
\end{bmatrix}
\begin{bmatrix}
u_a\\u_p
\end{bmatrix}
=
\begin{bmatrix}
F_a\\F_p
\end{bmatrix}
```

```math
u_a = K_{aa}^{-1}(F_a-K_{ap}u_p)
```

反力：

```math
R = Ku-F
```

### 9.5 题型 E：测试 element

最实用的测试：

| 测试 | 正确结果 |
|---|---|
| Tri3 mechanical constant strain patch | 单元内应变/应力为常数 |
| Quad4 mechanical patch | 线性位移场应能精确再现常应变 |
| thermal linear patch | 线性温度场产生常温度梯度 |
| free thermal expansion | 应力接近零，位移为 `alpha*DeltaT*x/y` |
| restrained thermal expansion | 产生非零热应力，反力平衡 |
| 无载荷 + 全零 BC | 解应为零 |

调试顺序：

1. 看 `detJ` 或面积是否为正
2. 看矩阵尺寸是否正确
3. 看 `K` 是否有异常零行
4. 看 essential BC 是否足够去掉刚体运动
5. 看边界条件是否冲突
6. 看反力是否与外力平衡

### 9.6 题型 F：用代码模拟问题

MATLAB 主流程：

```matlab
[globalMatrix, globalVector] = buildSystem(model);
bcData = buildBCData(model);
[reaction, solution] = solveSystem(globalMatrix, globalVector, bcData);

result.u = solution;
result.f = reaction;

nodal = extractNodalFields(model, result.u);
results = postprocessModel(model, result.u);
```

建模时依次检查：

1. `model.fieldNames`：`{'ux','uy'}`、`{'T'}` 或 `{'ux','uy','T'}`
2. `model.elesType`：元素名、每节点自由度数、每单元节点数
3. `model.nodesInfo`：节点编号和坐标
4. `model.elesInfo`：单元连接关系
5. `model.section`：材料参数
6. `model.problem`：thickness、plane stress/strain、`Tref`
7. `model.eleLoadData`：体力或热源
8. `model.essBC`：位移/温度边界
9. `model.natBC`：traction、heat flux、convection

### 9.7 题型 G：改变 BC 或 loading 后重新模拟

机械问题常改：

```matlab
model.essBC = {
    leftNodes, [0,0]
};

model.natBC = {
    rightNodes, [tx,ty]
};
```

热问题常改：

```matlab
model.essBC = {
    leftNodes, [Tleft];
    rightNodes, [Tright]
};

model.natBC = {
    topNodes, [h, h*Tinf]
};
```

热力耦合常改：

```matlab
model.essBC = {
    fixedNodes, [0,0,Tfixed]
};

model.natBC = {
    loadedEdge, [tx, ty, h, h*Tinf]
};
```

改完后必须重新运行：

```matlab
[K,F] = buildSystem(model);
bcData = buildBCData(model);
[reaction, solution] = solveSystem(K,F,bcData);
```

判断结果是否合理：

- 温度高的区域是否膨胀更明显
- 固定边附近是否有更大热应力
- 绝热边界法向热流是否接近零
- 反力和外力是否平衡
- 改大 `h` 后边界温度是否更接近 `T_inf`

---

## 10. Pei COMSOL 题型

### 10.1 2D thermomechanical case

推荐解题流程：

1. Geometry：建立 2D 矩形、板、梁或题目给定区域
2. Materials：输入 `E, nu, alpha, k`
3. Physics：
   - `Heat Transfer in Solids`
   - `Solid Mechanics`
   - `Multiphysics -> Thermal Expansion`
4. Heat Transfer BC：
   - prescribed temperature
   - heat flux
   - thermal insulation
   - convection
5. Solid Mechanics BC：
   - fixed constraint
   - roller/symmetry constraint
   - boundary load / traction
6. Study：通常选 stationary
7. Mesh：先用默认网格，再细化检查收敛
8. Results：
   - temperature
   - displacement magnitude
   - stress / von Mises
   - heat flux
9. Validation：
   - 与 MATLAB 模型的边界条件和材料保持一致
   - 比较温度场趋势、最大位移、最大应力、反力

常见错误：

- 忘记 thermal expansion multiphysics coupling
- `Tref` 与 MATLAB 中的 `model.problem.Tref` 不一致
- 2D plane stress / plane strain 选择和 MATLAB 不一致
- 单位不统一，如 mm 和 m 混用
- 固体力学约束不够，导致刚体运动

### 10.2 3D thermomechanical case

3D 与 2D 的区别：

| 内容 | 2D | 3D |
|---|---|---|
| 几何 | 面 | 实体 |
| 力边界 | edge/boundary | face |
| 约束 | 点/边约束容易过约束 | 必须移除 6 个刚体模态 |
| 应力 | plane stress/strain | full 3D stress |
| 结果 | 面内位移和应力 | 3D 位移、应力、热流 |

3D 解题逻辑：

1. 建实体几何
2. 设材料参数
3. 加 Heat Transfer + Solid Mechanics + Thermal Expansion
4. 温度边界施加在 face 上
5. 机械固定/载荷也施加在 face 上
6. 检查约束是否既能去掉刚体运动，又不过度限制热膨胀
7. 看切片图、表面图、最大 von Mises、总反力

---

## 11. Lars code / FDM 补充复习

截图中列了 `Lars code`，本节只覆盖当前工作区 Lars 课件中与代码题最相关的部分。

### 11.1 ODE one-step methods

Cauchy problem：

```math
y'(t)=f(t,y),\qquad y(t_0)=y_0
```

Forward Euler：

```math
y_{n+1}=y_n+h f(t_n,y_n)
```

Backward Euler：

```math
y_{n+1}=y_n+h f(t_{n+1},y_{n+1})
```

Heun / improved Euler：

```math
\tilde y_{n+1}=y_n+h f(t_n,y_n)
```

```math
y_{n+1}=y_n+\frac{h}{2}
\left[
f(t_n,y_n)+f(t_{n+1},\tilde y_{n+1})
\right]
```

RK4：

```math
k_1=f(t_n,y_n)
```

```math
k_2=f(t_n+h/2,y_n+hk_1/2)
```

```math
k_3=f(t_n+h/2,y_n+hk_2/2)
```

```math
k_4=f(t_n+h,y_n+hk_3)
```

```math
y_{n+1}=y_n+\frac{h}{6}(k_1+2k_2+2k_3+k_4)
```

代码题思路：

1. 初始化 `t,u`
2. 循环 `n=1:N`
3. 按方法公式更新
4. 隐式方法需要 Newton 或 fixed-point iteration
5. 比较步长改变后的误差和稳定性

### 11.2 稳定性、相容性、收敛

Local truncation error 越高阶，理论精度越高。

核心关系：

```text
consistency + zero-stability -> convergence
```

绝对稳定性通常用 test equation：

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

对负实部刚性问题更稳定。

### 11.3 1D boundary value problem 的有限差分

例如：

```math
u''(x)=f(x),\qquad u(0)=a,\quad u(L)=b
```

中心差分：

```math
u''(x_i)\approx
\frac{u_{i-1}-2u_i+u_{i+1}}{h^2}
```

内部点矩阵行：

```text
[... 1/h^2  -2/h^2  1/h^2 ...]
```

Dirichlet 边界常用整行替换：

```text
A(1,:) = 0; A(1,1)=1; rhs(1)=a
A(N,:) = 0; A(N,N)=1; rhs(N)=b
```

### 11.4 2D Poisson / Laplace 五点格式

Poisson：

```math
u_{xx}+u_{yy}=f(x,y)
```

若 `hx=hy=h`：

```math
\frac{
u_{i-1,j}+u_{i+1,j}+u_{i,j-1}+u_{i,j+1}-4u_{i,j}
}{h^2}
=f_{i,j}
```

若 `hx` 和 `hy` 不同：

```math
\frac{u_{i-1,j}-2u_{i,j}+u_{i+1,j}}{h_x^2}
+
\frac{u_{i,j-1}-2u_{i,j}+u_{i,j+1}}{h_y^2}
=
f_{i,j}
```

代码题关键是二维索引转一维索引。例如：

```matlab
k = i + (j-1)*Nx;
```

然后填矩阵：

```text
A(k,k)       = -2/hx^2 - 2/hy^2
A(k,k-1)     =  1/hx^2
A(k,k+1)     =  1/hx^2
A(k,k-Nx)    =  1/hy^2
A(k,k+Nx)    =  1/hy^2
rhs(k)       = f(i,j)
```

边界点用整行替换。

---

## 12. 考前速查表

### 12.1 看到题目时先分类

| 关键词 | 反应 |
|---|---|
| Tri3 | 面积、`b_i,c_i`、常 `B` |
| Quad4 | `N,dN,J,detJ`、2x2 Gauss |
| mechanical | `B_u` 是 `3 x 2n`，用 `D` |
| thermal | `B_T` 是 `2 x n`，用 `Kcond` |
| thermomechanical | 三个块：`Kuu,KTT,KuT` |
| traction | 边界向量 |
| convection/Robin | 边界矩阵 + 边界向量 |
| essential BC | prescribed DoF，进入 `Kap*up` |
| natural BC | 先装配到 `K,F`，再施加 essential BC |

### 12.2 最容易丢分的点

1. `gamma_xy = u_y + v_x`，不是 tensor shear strain
2. plane stress 和 plane strain 的 `D` 不能混用
3. Quad4 的 `J`、`detJ`、`gradN` 方向要一致
4. 热传导 `B_T` 是 `2 x n`，不是机械 `B_u`
5. `KuT` 有负号
6. 热力耦合代码自由度是交错顺序
7. Robin 热边界有矩阵项
8. 刚体运动没约束会导致奇异矩阵
9. 温度 Dirichlet 冲突会直接导致错误或不物理解
10. COMSOL 和 MATLAB 的单位、厚度、`Tref`、plane stress/strain 必须一致

---

## 13. 一句话总结

Pei FEM 部分不是背完整代码，而是背“公式到代码”的映射：

```text
N -> dN -> J -> B -> Ke/fe -> element DoFs -> global K/F -> BC -> solve -> stress/flux
```

Lars code 部分不是背所有数值方法，而是背“离散到线性系统”的映射：

```text
differential equation -> stencil/time step formula -> matrix/update rule -> boundary rows -> stability/error check
```

