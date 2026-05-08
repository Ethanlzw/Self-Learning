# CMP Exam Review: Separated FEM and FDM Notes (English)

---

## Overview

The exam scope can be separated into three practical blocks:

| Part | Source | Exam focus |
|---|---|---|
| Part A: FEM | Pei lectures + Pei MATLAB framework | Tri3/Quad4 derivations, mechanical/thermal/thermomechanical elements, code fill-ins, BC/loading changes, COMSOL |
| Part B: FDM | Lars lectures + `Lars Part1` scripts | 1D/2D Poisson/Laplace finite differences, matrix assembly, boundary rows/RHS, ODE time stepping, exercises |
| COMSOL | Pei thermomechanics | 2D/3D thermomechanical setup, boundary conditions, result interpretation |

---

# Part A. FEM Review (Pei)

## A1. Core FEM Workflow

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

Do not memorize only code. Memorize the formula-to-code mapping:

```text
N, dN, J, B
-> Ke and fe
-> eleDoFs
-> K(eleDoFs,eleDoFs) += Ke
-> F(eleDoFs) += fe
-> Kaa ua = Fa - Kap up
```

---

## A2. FEM Core Concepts

### A2.1 Mechanical Strong Form

```math
\nabla\cdot\sigma+b=0 \quad \text{in } \Omega
```

```math
u=\bar u \quad \text{on } \Gamma_u
```

```math
\sigma n=\bar t \quad \text{on } \Gamma_t
```

### A2.2 Thermal Strong Form

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

Robin / convection:

```math
-q\cdot n=h(T-T_\infty)
```

### A2.3 Mechanical Weak Form

```math
\int_\Omega \delta\varepsilon^T\sigma\,d\Omega
=
\int_\Omega \delta u^T b\,d\Omega
+
\int_{\Gamma_t}\delta u^T\bar t\,d\Gamma
```

### A2.4 Thermal Weak Form

```math
\int_\Omega \nabla(\delta T)^T K\nabla T\,d\Omega
=
\int_\Omega \delta T r\,d\Omega
+
\int_{\Gamma_q}\delta T\bar q_n\,d\Gamma
```

Robin boundary contribution:

```math
\int_{\Gamma_q}\delta T hT\,d\Gamma
=
\int_{\Gamma_q}\delta T hT_\infty\,d\Gamma
```

So a Robin boundary contributes both a matrix and a vector.

---

## A3. FEM Formula Sheet

### A3.1 2D Small Strain

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

Engineering shear strain:

```math
\gamma_{xy}=2\varepsilon_{xy}
```

Finite element approximation:

```math
u \approx N_u d_e,\qquad \varepsilon=B_ud_e
```

### A3.2 Elasticity Matrix

Plane stress:

```math
D=
\frac{E}{1-\nu^2}
\begin{bmatrix}
1&\nu&0\\
\nu&1&0\\
0&0&\frac{1-\nu}{2}
\end{bmatrix}
```

Plane strain:

```math
D=
\frac{E}{(1+\nu)(1-2\nu)}
\begin{bmatrix}
1-\nu&\nu&0\\
\nu&1-\nu&0\\
0&0&\frac{1-2\nu}{2}
\end{bmatrix}
```

Mechanical element stiffness:

```math
K_{uu}^e=\int_{\Omega_e}B_u^TDB_u\,t\,dA
```

Mechanical body-force vector:

```math
f_b^e=\int_{\Omega_e}N_u^Tb\,t\,dA
```

Traction vector:

```math
f_t^e=\int_{\Gamma_t^e}N_u^T\bar t\,t\,d\Gamma
```

von Mises stress:

```math
\sigma_{vm}
=
\sqrt{\sigma_x^2-\sigma_x\sigma_y+\sigma_y^2+3\tau_{xy}^2}
```

### A3.3 Heat-Conduction Matrix

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

Robin:

```math
K_\Gamma^e=\int_{\Gamma_q^e}N_T^ThN_T\,t\,d\Gamma
```

```math
f_\Gamma^e=\int_{\Gamma_q^e}N_T^ThT_\infty\,t\,d\Gamma
```

### A3.4 Thermomechanics

Thermal strain:

```math
\varepsilon_{th}
=
\alpha(T-T_{ref})
\begin{bmatrix}
1\\1\\0
\end{bmatrix}
```

Constitutive law:

```math
\sigma=D(\varepsilon-\varepsilon_{th})
```

Coupling matrix:

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

Element block system:

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

Physical meaning: Pei's framework is one-way thermomechanical coupling. Temperature affects mechanical stress; mechanical displacement does not feed back into steady heat conduction.

---

## A4. Tri3 Derivation

### A4.1 Tri3 Geometry

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

Tri3 is a constant-strain triangle, so `B` is constant within the element.

### A4.2 Tri3 Mechanical

DoF order:

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

For constant body force:

```math
f_b^e=
\frac{tA}{3}
\begin{bmatrix}
b_x\\b_y\\b_x\\b_y\\b_x\\b_y
\end{bmatrix}
```

Exam fill-in order: `Area -> b/c -> B -> D -> Ke -> fe`.

### A4.3 Tri3 Thermal

DoF order:

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

For a constant heat source:

```math
f_r^e=\frac{tAr}{3}
\begin{bmatrix}
1\\1\\1
\end{bmatrix}
```

### A4.4 Tri3 Thermomechanical

DoF order:

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

Interleaved indices:

```matlab
mechDofs = [1 2 4 5 7 8];
tempDofs = [3 6 9];
```

---

## A5. Quad4 Derivation

### A5.1 Quad4 Shape Functions

Natural-coordinate nodes:

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

Jacobian:

```math
J=
\begin{bmatrix}
x_{,s}&y_{,s}\\
x_{,t}&y_{,t}
\end{bmatrix}
```

Derivative transform:

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

2x2 Gauss rule:

```math
s,t=\pm\frac{1}{\sqrt3},\qquad w=1
```

### A5.2 Quad4 Mechanical

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

### A5.3 Quad4 Thermal

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

### A5.4 Quad4 Thermomechanical

DoF order:

```text
[ux1 uy1 T1 ux2 uy2 T2 ux3 uy3 T3 ux4 uy4 T4]^T
```

Inside the Gauss loop:

```math
K_{uu} += B_u^TDB_u\det Jwt
```

```math
K_{TT} += B_T^TKB_T\det Jwt
```

```math
K_{uT} += -B_u^TD\alpha[1,1,0]^TN_T\det Jwt
```

Interleaved indices:

```matlab
mechDofs = [1 2 4 5 7 8 10 11];
tempDofs = [3 6 9 12];
```

---

## A6. FEM Problem-Type Strategies

### A6.1 Derive a Stiffness Matrix

First classify:

| Keywords | Formula |
|---|---|
| displacement / stress / strain | `Kuu = int Bu' D Bu` |
| temperature / heat flux | `KTT = int BT' K BT` |
| thermal expansion | `Kuu, KTT, KuT` |
| Tri3 | closed-form area integration |
| Quad4 | `J + detJ + Gauss` |

Answer template:

```text
1. Write u=N d or T=N theta
2. Write epsilon=Bu d or gradT=BT theta
3. Substitute into weak form
4. Read off Ke and fe
5. Give closed form for Tri3 or Gauss sum for Quad4
```

### A6.2 Derive a Force Vector

| Load | Domain | Expression |
|---|---|---|
| body force | element area | `int Nu' b dA` |
| heat source | element area | `int NT' r dA` |
| traction | boundary edge | `int Nu' tbar dGamma` |
| heat flux | boundary edge | `int NT' qbar dGamma` |
| Robin | boundary edge | `int NT' h NT` and `int NT' hTinf` |

### A6.3 Fill Element Code

Dimension checks:

| Element | Matrix size |
|---|---:|
| Tri3 mechanical | `6 x 6` |
| Quad4 mechanical | `8 x 8` |
| Tri3 thermal | `3 x 3` |
| Quad4 thermal | `4 x 4` |
| Tri3 thermomechanical | `9 x 9` |
| Quad4 thermomechanical | `12 x 12` |

### A6.4 Change BC/Loading and Simulate Again

Mechanical:

```matlab
model.essBC = {
    leftNodes, [0,0]
};
model.natBC = {
    rightNodes, [tx,ty]
};
```

Thermal:

```matlab
model.essBC = {
    leftNodes, [Tleft];
    rightNodes, [Tright]
};
model.natBC = {
    topNodes, [h, h*Tinf]
};
```

Thermomechanical:

```matlab
model.essBC = {
    fixedNodes, [0,0,Tfixed]
};
model.natBC = {
    loadedEdge, [tx, ty, h, h*Tinf]
};
```

Rerun:

```matlab
[K,F] = buildSystem(model);
bcData = buildBCData(model);
[reaction, solution] = solveSystem(K,F,bcData);
```

---

## A7. FEM MATLAB Code Skeletons

### A7.1 Tri3 Mechanical

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

### A7.2 Quad4 Common Geometry

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

### A7.3 Thermomechanical Blocks

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

### A7.5 Essential-BC Solver

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

## A8. Element Test Logic

| Test | Correct behavior |
|---|---|
| mechanical patch test | linear displacement gives constant strain; Tri3/Quad4 should pass |
| thermal linear patch | linear temperature gives constant temperature gradient |
| free thermal expansion | stress is near zero; displacement is `alpha*DeltaT*x/y` |
| restrained thermal expansion | nonzero thermal stress and balanced reactions |
| zero load + zero BC | zero solution |
| cantilever beam | reactions balance; Tri3 is typically too stiff in bending, Quad4 is usually better |

Debug order:

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

## A9. Pei COMSOL 2D/3D Thermomechanics

### A9.1 2D Thermomechanical Case

Workflow:

```text
1. Geometry: create 2D geometry
2. Materials: E, nu, alpha, k
3. Physics: Heat Transfer in Solids + Solid Mechanics
4. Multiphysics: Thermal Expansion
5. Thermal BC: T, heat flux, insulation, convection
6. Mechanical BC: fixed, roller/symmetry, boundary load
7. Study: Stationary
8. Mesh: default mesh + refinement check
9. Results: T, displacement, von Mises, heat flux
```

Keep MATLAB and COMSOL consistent:

```text
units, thickness, plane stress/strain, Tref, material, BCs
```

Common mistakes:

- forgetting Thermal Expansion coupling
- inconsistent `Tref`
- inconsistent plane stress/plane strain
- insufficient mechanical constraints causing rigid-body motion
- overconstraining thermal expansion

### A9.2 3D Thermomechanical Case

3D notes:

| Item | 2D | 3D |
|---|---|---|
| Geometry | surface | solid |
| force/thermal boundary | edge/boundary | face |
| stress state | plane stress/strain | full 3D stress |
| constraints | remove 2D rigid motion | remove 6 rigid-body modes |

Check:

```text
temperature field
-> displacement direction and magnitude
-> stress concentration
-> reaction force balance
-> mesh convergence
```

---

# Part B. FDM Review (Lars)

## B1. FDM and Lars Code Scope

This part corresponds to:

```text
Lecture/Introduction, Series Solutions, Finite Difference
Lars Part1/files
Lars Part1/fun
Lars Part1/L06_Ex
```

Important scripts:

| File | Content |
|---|---|
| `laplaceequation.m` | separated-series solution for a rectangular Laplace Dirichlet problem |
| `dirichlet_rectangle.m` | superposition of four Dirichlet sides for rectangular Laplace |
| `dirichlet_rectangle_poisson_equation.m` | Poisson point source / Green response series |
| `oned_poisson_laplace_dirichlet_BC.m` | 1D Poisson/Laplace Dirichlet FDM |
| `Ex_v03.m` | 2D Poisson/Laplace five-point stencil exercise |
| `build_full_grid.m` | add boundary values around an interior solution |
| `L06_Ex_3_loop.m` | 1D temperature + thermoelastic displacement coupling exercise |
| `forward_euler.m`, `backward_euler.m` | explicit/implicit Euler |
| `crank_nicolson.m` | Crank-Nicolson |
| `runge_kutta.m`, `rk45.m` | RK4 and adaptive RK45 |
| `adams_bashforth.m`, `adams_moulton.m`, `bdf.m` | multistep methods |

---

## B2. FDM Core Ideas

FDM directly discretizes derivatives in the strong form:

```text
PDE/ODE
-> grid
-> difference stencil
-> matrix/update formula
-> impose boundary/initial conditions
-> solve or march in time
-> compare error/stability
```

FDM vs FEM:

| Item | FDM | FEM |
|---|---|---|
| Starting point | strong-form derivative approximation | weak form + shape functions |
| Mesh | easiest on structured grids | more flexible for complex geometry |
| Core object | stencil / finite-difference matrix | element matrix / assembly |
| BC handling | direct row replacement or RHS contribution | essential/natural BC treatment |

---

## B3. ODE Time-Stepping Formulas

### B3.1 Cauchy Problem

```math
y'(t)=f(t,y),\qquad y(t_0)=y_0
```

### B3.2 Forward Euler

```math
y_{n+1}=y_n+h f(t_n,y_n)
```

Code skeleton:

```matlab
for n = 1:N
    y(n+1,:) = y(n,:) + h * f(t(n), y(n,:)')';
end
```

Properties:

- explicit
- first order
- can be unstable for stiff problems

### B3.3 Backward Euler

```math
y_{n+1}=y_n+h f(t_{n+1},y_{n+1})
```

Newton residual:

```math
R(y_{n+1})=y_{n+1}-y_n-hf(t_{n+1},y_{n+1})
```

Jacobian:

```math
I-h\frac{\partial f}{\partial y}
```

Code skeleton:

```matlab
res = yn - y(n,:)' - h * f(t(n+1), yn);
jac = eye(length(y0)) - h * dfdy(t(n+1), yn);
delta = jac \ res;
yn = yn - delta;
```

Properties:

- implicit
- first order
- more stable for stiff problems

### B3.4 Crank-Nicolson

```math
y_{n+1}=y_n+\frac{h}{2}
\left[
f(t_n,y_n)+f(t_{n+1},y_{n+1})
\right]
```

Newton residual:

```math
R=y_{n+1}-y_n-\frac{h}{2}
\left[f(t_n,y_n)+f(t_{n+1},y_{n+1})\right]
```

Properties:

- implicit
- second order
- common for diffusion-type problems

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

Implemented by the `runge_kutta` function in `runge_kutta.m`.

### B3.6 RK45

RK45 computes fourth- and fifth-order estimates:

```math
err=|u_5-u_4|
```

Step-size control:

```math
\delta=0.9\left(\frac{\epsilon}{err}\right)^{1/5}
```

Logic:

```text
err <= tolerance: accept step
err > tolerance: reject step and reduce h
```

### B3.7 Adams / BDF Multistep Methods

Adams-Bashforth is explicit:

```math
y_{n+1}=y_n+h\sum_{j=0}^{p-1}b_j f_{n-j}
```

AB2:

```math
y_{n+1}=y_n+h\left(\frac32 f_n-\frac12 f_{n-1}\right)
```

Adams-Moulton is implicit:

```math
y_{n+1}=y_n+h\sum b_j f_{n+1-j}
```

BDF2:

```math
\frac32 y_n-2y_{n-1}+\frac12 y_{n-2}=h f(t_n,y_n)
```

Exam logic:

1. Multistep methods need history values
2. Startup values often come from RK4 / ode45 / Euler
3. Explicit methods do not need iteration
4. Implicit methods need Newton or fixed-point iteration
5. Stiff problems often require implicit methods or BDF

---

## B4. Stability, Consistency and Convergence

### B4.1 Local Truncation Error

Method order is determined from the local truncation error. Forward Euler is first order, Heun/Crank-Nicolson is second order, and RK4 is fourth order.

### B4.2 Zero-Stability and Convergence

Core relation from Lars lecture:

```text
consistency + zero-stability -> convergence
```

### B4.3 Absolute Stability

Test problem:

```math
y'=\lambda y
```

Forward Euler:

```math
y_{n+1}=(1+h\lambda)y_n
```

Stability condition:

```math
|1+h\lambda|<1
```

Backward Euler:

```math
y_{n+1}=\frac{1}{1-h\lambda}y_n
```

Stiffness intuition: eigenvalues have very different scales. An explicit method's step size is limited by the fastest decaying mode, even if the slow mode is the physically important one.

---

## B5. 1D Poisson/Laplace FDM

### B5.1 Standard Problem

```math
u''(x)=f(x),\qquad u(0)=u_0,\quad u(L)=u_L
```

Grid:

```math
h=\frac{L}{N+1},\qquad x_i=ih,\quad i=1,\ldots,N
```

Central difference:

```math
u''(x_i)\approx
\frac{u_{i-1}-2u_i+u_{i+1}}{h^2}
```

Matrix row:

```text
[... 1/h^2  -2/h^2  1/h^2 ...]
```

Boundary contribution to RHS:

```matlab
rhs(1)   = rhs(1)   - u0/h^2;
rhs(end) = rhs(end) - uL/h^2;
```

Code skeleton:

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

### B5.2 Exercise: `L=5, f=-20, u(0)=u(L)=0`

Analytic check from Lecture 6:

```math
u(x)=-10x^2+10Lx
```

Solution logic:

```text
1. Build 1D grid
2. Discretize u'' by the three-point central difference
3. Put Dirichlet boundaries into RHS or replace whole rows
4. Solve A\RHS
5. Compare with u_exact
```

---

## B6. 2D Poisson/Laplace FDM

### B6.1 Standard Problem

```math
u_{xx}+u_{yy}=f(x,y)
```

Rectangular domain:

```math
(0,a)\times(0,b)
```

Grid:

```math
h_x=\frac{a}{N_x+1},\qquad h_y=\frac{b}{N_y+1}
```

Five-point stencil:

```math
\frac{u_{i-1,j}-2u_{i,j}+u_{i+1,j}}{h_x^2}
+
\frac{u_{i,j-1}-2u_{i,j}+u_{i,j+1}}{h_y^2}
=f_{i,j}
```

Center coefficient:

```math
-\frac{2}{h_x^2}-\frac{2}{h_y^2}
```

Neighbor coefficients:

```math
\frac{1}{h_x^2},\qquad \frac{1}{h_y^2}
```

### B6.2 1D Indexing

`Ex_v03.m` uses x-fastest ordering:

```matlab
k = i + (j-1)*Nx;
```

Matrix fill:

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

### B6.3 Dirichlet Boundary in RHS

For a top boundary:

```math
u(x,b)=g_{top}(x)
```

when the upper neighbor is a boundary value:

```matlab
rhs(k) = rhs(k) - invhy2*gTop(x(i));
```

Zero boundaries contribute zero.

### B6.4 Lecture 6 Exercise

Script `Ex_v03.m` uses:

```matlab
a = 1; b = 2;
Nx = 2; Ny = 3;
gTop = @(x) 4*(-x.^2 + x);
```

First solve Laplace:

```math
f(x,y)=0
```

Then solve Poisson:

```math
f(x,y)=-10
```

Method:

```text
1. Assemble the same A
2. Laplace RHS = boundary contribution
3. Poisson RHS = boundary contribution + f
4. Solve U=A\rhs
5. reshape(U,[Nx,Ny])
6. Use build_full_grid to add boundaries and plot
```

Key point: `A` depends only on the grid and operator. If only `f` changes, reuse `A`.

---

## B7. Separation of Variables and Rectangle Series

This part is from Lars Lecture 3 and the scripts `laplaceequation.m`, `dirichlet_rectangle.m`, and `dirichlet_rectangle_poisson_equation.m`.

### B7.1 Laplace Problem With Nonzero Top Edge

```math
\nabla^2u=0,\quad 0<x<a,\;0<y<b
```

```math
u(x,0)=0,\quad u(0,y)=0,\quad u(a,y)=0,\quad u(x,b)=g(x)
```

Separated solution:

```math
u(x,y)=
\sum_{n=1}^{\infty}
B_n\sin\left(\frac{n\pi x}{a}\right)
\sinh\left(\frac{n\pi y}{a}\right)
```

Coefficient:

```math
B_n=
\frac{2}{a\sinh(n\pi b/a)}
\int_0^a
g(x)\sin\left(\frac{n\pi x}{a}\right)dx
```

MATLAB skeleton:

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

### B7.2 Four Nonhomogeneous Dirichlet Sides

Use superposition:

```math
u=u_{top}+u_{bottom}+u_{left}+u_{right}
```

Each subproblem has only one nonzero boundary side and the other three sides are zero.

### B7.3 Poisson Point Source / Green Response

Homogeneous Dirichlet:

```math
\nabla^2u=-Q\delta(x-x_0,y-y_0)
```

Series:

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

Exam logic:

```text
1. Check whether the boundary is homogeneous
2. Choose sine modes to satisfy zero Dirichlet boundaries
3. Project the boundary/source term into Fourier coefficients
4. Truncate at N or mMax,nMax
5. Discuss truncation error and source singularity behavior
```

---

## B8. 1D Thermo-Mechanical Coupling Exercise (L06)

Project script: `L06_Ex_3_loop.m`

### B8.1 Temperature Problem

The script solves:

```math
u''=f,\qquad u(0)=u(L)=0,\qquad f=-20
```

Discrete row:

```matlab
A(j,j-1:j+1) = (1/h^2)*[1 -2 1];
rhs_u(j) = fConst;
```

### B8.2 Displacement Problem

The lecture exercise then feeds temperature into mechanics:

```math
w''=\alpha u'
```

Boundary conditions:

```math
w(0)=w(L)=0
```

Discretization:

```matlab
Aw(j,j-1:j+1) = (1/h^2)*[1 -2 1];
uPrime = (u(j+1)-u(j-1))/(2*h);
rhs_w(j) = alpha*uPrime;
```

### B8.3 Stress

```math
\sigma=E(w'-\alpha u)
```

Central difference:

```matlab
wPrime = (w(j+1)-w(j-1))/(2*h);
sigma(j) = E*(wPrime - alpha*u(j));
```

Analytic checks:

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

Solution logic:

```text
1. Solve temperature u
2. Differentiate u using central difference
3. Put alpha*u' into the displacement RHS
4. Solve w
5. Differentiate w using central difference
6. Compute stress = E*(w' - alpha*u)
7. Compare with analytic checks
```

---

## B9. FDM Exercise Types

### B9.1 Type 1: Given an ODE, Write Time-Stepping Code

Steps:

```text
1. Decide explicit or implicit
2. Explicit: update directly
3. Implicit: write the residual
4. Use Newton or an analytic root to obtain y_{n+1}
5. Compare with exact/ode45
6. Change h and observe error or instability
```

Key methods:

```text
Forward Euler, Backward Euler, Crank-Nicolson, RK4, RK45, Adams, BDF
```

### B9.2 Type 2: 1D BVP Difference Matrix

Steps:

```text
1. Put only interior unknowns in U
2. Use the three-point central difference to form a tridiagonal matrix
3. Move Dirichlet boundaries into RHS
4. Solve A\RHS
5. full solution = [leftBC; U; rightBC]
```

### B9.3 Type 3: 2D Poisson/Laplace Five-Point Stencil

Steps:

```text
1. Build the interior grid
2. Define index k=i+(j-1)*Nx
3. Fill center/right/left/up/down coefficients for each interior node
4. Move nonzero Dirichlet boundaries into RHS
5. Solve
6. Reshape and plot
```

### B9.4 Type 4: Rectangle Laplace Series

Steps:

```text
1. Identify the nonzero boundary side
2. Write the matching sine-sinh modes
3. Use integral to compute Fourier coefficients
4. Sum a truncated series
5. For four nonzero sides, use superposition
```

### B9.5 Type 5: 1D Temperature-Mechanics Coupling

Steps:

```text
1. solve thermal Poisson
2. differentiate temperature with central difference
3. assemble displacement equation
4. solve displacement
5. compute stress
6. compare with analytic checks
```

---

## B10. Common FDM Mistakes

1. Using one `h` when `hx` and `hy` are different
2. Writing the 2D-to-1D index `k=i+(j-1)*Nx` incorrectly
3. Forgetting nonzero Dirichlet boundary contributions in RHS
4. Inconsistent signs between Poisson `f=-10` and the matrix operator
5. Confusing orientation after `reshape(U,[Nx,Ny])`
6. Treating an implicit ODE method as explicit
7. Starting a multistep method without enough history values
8. Using Forward Euler with too large a step for a stiff problem
9. Poor boundary handling when computing `u'` or `w'` in the thermo-mechanical exercise
10. Forgetting the `sinh(n*pi*b/a)` denominator in the rectangle series coefficient

---

## B11. FDM Last-Minute Checklist

### B11.1 1D Stencil

```math
u''_i\approx\frac{u_{i-1}-2u_i+u_{i+1}}{h^2}
```

### B11.2 2D Stencil

```math
\frac{u_{i-1,j}-2u_{i,j}+u_{i+1,j}}{h_x^2}
+
\frac{u_{i,j-1}-2u_{i,j}+u_{i,j+1}}{h_y^2}
=f_{i,j}
```

### B11.3 Dirichlet RHS Contribution

```text
known boundary neighbor coefficient * boundary value
move to RHS with a minus sign
```

### B11.4 Implicit ODE Residual

Backward Euler:

```math
R=y_{n+1}-y_n-hf(t_{n+1},y_{n+1})
```

Crank-Nicolson:

```math
R=y_{n+1}-y_n-\frac{h}{2}[f_n+f_{n+1}]
```

### B11.5 Final Memory Hook

FEM:

```text
weak form -> element matrix -> assembly
```

FDM:

```text
strong form -> stencil -> matrix row / time update
```
