# Thermomechanical Reference

## 1. Project Role

This folder solves 2D coupled thermoelastic problems with the nodal unknown order:

```text
[ux, uy, T]
```

Main script:

```matlab
main_mixed_thermomech.m
```

Element routines:

```text
Tri3_ThermoMech.m
Quad4_ThermoMech.m
Edge2_ThermoMech.m
```

The element-level block system is:

```math
\begin{bmatrix}
K_{uu} & K_{uT}\\
0      & K_{TT}
\end{bmatrix}
\begin{bmatrix}
u\\
T
\end{bmatrix}
=
\begin{bmatrix}
f_u\\
f_T
\end{bmatrix}
```

Knowledge point:

- `Kuu` is the mechanical stiffness.
- `KTT` is the heat-conduction stiffness.
- `KuT` is the thermal-expansion coupling from temperature to mechanical equilibrium.
- The system is generally not symmetric because of the one-way block `KuT`.

## 2. Main Script Pattern

File: `main_mixed_thermomech.m`

Core model declaration:

```matlab
model.fieldNames = {'ux','uy','T'};

model.elesType = {
    1, 'Tri3_ThermoMech',  3, 3;
    2, 'Quad4_ThermoMech', 3, 4
};
```

Material vector:

```matlab
material = [E, nu, alpha, kxx, kyy];
```

Problem data:

```matlab
model.problem.thickness = thickness;
model.problem.mechType = 'planeStress';
model.problem.Tref = Tref;
```

Logic:

- Each node has three DOFs, so Tri3 has 9 element DOFs and Quad4 has 12 element DOFs.
- `Tref` matters because thermal strain is based on `T - Tref`.
- Plane stress or plane strain changes only the elastic matrix `D`, not the FEM assembly pattern.

## 3. Mesh and Q4 Node Order

Each rectangular cell is named:

```matlab
n1 = node(i  , j  );     % bottom-left
n2 = node(i+1, j  );     % bottom-right
n3 = node(i+1, j+1);     % top-right
n4 = node(i  , j+1);     % top-left
```

Current Q4 connectivity:

```matlab
quadElems = [quadElems; eleID, n1, n2, n3, n4];
```

Knowledge point:

- Current Q4 convention follows Lecture 4-2 Fig. 4.4.3:
  `1 = bottom-left`, `2 = bottom-right`, `3 = top-right`, `4 = top-left`.
- This must match the Q4 shape functions in `Quad4_ThermoMech.m`.
- A reversed node direction gives negative `detJ`.

## 4. Thermomechanical Constitutive Logic

Thermal strain:

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

Stress:

```math
\sigma = D(\varepsilon - \varepsilon_{th})
```

In code:

```matlab
epsTh = alpha * [1; 1; 0];
```

Logic:

- The vector `[1;1;0]` is for 2D isotropic thermal expansion in engineering strain notation.
- The thermal strain affects normal strain components, not the engineering shear strain.
- If `Tref` is nonzero, the element needs a reference-temperature contribution in the mechanical right-hand side.

## 5. Tri3 Thermomechanical Element

File: `Tri3_ThermoMech.m`

Core geometry:

```matlab
A = [1 x1 y1;
     1 x2 y2;
     1 x3 y3];

detA = det(A);
Area = detA / 2;

if Area <= 0
    error('Tri3_ThermoMech has non-positive area.');
end
```

Mechanical and thermal gradient matrices:

```matlab
Bu = [b1 0  b2 0  b3 0;
      0  c1 0  c2 0  c3;
      c1 b1 c2 b2 c3 b3] / detA;

BT = [b1 b2 b3;
      c1 c2 c3] / detA;
```

Block matrices:

```matlab
Kuu = thickness * Area * (Bu' * D * Bu);
KTT = thickness * Area * (BT' * Kcond * BT);

Nint = Area / 3 * [1 1 1];
KuT = -thickness * (Bu' * D * epsTh) * Nint;
```

Reference-temperature contribution:

```matlab
fMech = fMech - thickness * Area * (Bu' * D * epsTh) * Tref;
```

Loads:

```matlab
fMech = fMech + thickness * Area / 3 * [bx; by; bx; by; bx; by];
fTherm = fTherm + thickness * Area / 3 * qv * [1; 1; 1];
```

Block insertion:

```matlab
mechDofs = [1 2 4 5 7 8];
tempDofs = [3 6 9];

eleMatrix(mechDofs, mechDofs) = Kuu;
eleMatrix(mechDofs, tempDofs) = KuT;
eleMatrix(tempDofs, tempDofs) = KTT;
```

Logic:

- Tri3 uses constant `Bu` and `BT`, so no Gauss loop is needed.
- `KuT` is 6 x 3 for Tri3.
- The final element matrix is 9 x 9 because the nodal order is `[ux uy T]` at each node.

## 6. Quad4 Thermomechanical Element

File: `Quad4_ThermoMech.m`

Shape functions:

```matlab
N = 0.25 * [(1 - s) * (1 - t);
            (1 + s) * (1 - t);
            (1 + s) * (1 + t);
            (1 - s) * (1 + t)];

dNds = 0.25 * [-(1-t);  (1-t);  (1+t); -(1+t)];
dNdt = 0.25 * [-(1-s); -(1+s);  (1+s);  (1-s)];
```

Jacobian:

```matlab
J = [dNds'*x, dNds'*y;
     dNdt'*x, dNdt'*y];

detJ = det(J);
if detJ <= 0
    error('Quad4_ThermoMech has non-positive detJ at a Gauss point.');
end

gradN = inv(J) * [dNds'; dNdt'];
dNdx = gradN(1,:)';
dNdy = gradN(2,:)';
```

Mechanical and thermal matrices:

```matlab
Bu = [dNdx(1)      0    dNdx(2)   0      dNdx(3)     0      dNdx(4)    0;
        0      dNdy(1)    0     dNdy(2)    0      dNdy(3)    0       dNdy(4);
      dNdy(1)  dNdx(1)  dNdy(2) dNdx(2)  dNdy(3)  dNdx(3)   dNdy(4)  dNdx(4)];

BT = [dNdx';
      dNdy'];
```

Gauss accumulation:

```matlab
Kuu = Kuu + (Bu' * D * Bu) * detJ * weights(igp) * thickness;
KTT = KTT + (BT' * Kcond * BT) * detJ * weights(igp) * thickness;
KuT = KuT - (Bu' * D * epsTh) * N' * detJ * weights(igp) * thickness;
```

Loads:

```matlab
Nmat = [N(1)   0    N(2)    0    N(3)   0    N(4)   0;
         0    N(1)    0    N(2)   0    N(3)   0    N(4)];

fMech = fMech + Nmat' * [bx; by] * detJ * weights(igp) * thickness;
fTherm = fTherm + N * qv * detJ * weights(igp) * thickness;
```

Block insertion:

```matlab
mechDofs = [1 2 4 5 7 8 10 11];
tempDofs = [3 6 9 12];

eleMatrix(mechDofs, mechDofs) = Kuu;
eleMatrix(mechDofs, tempDofs) = KuT;
eleMatrix(tempDofs, tempDofs) = KTT;
```

Logic:

- Quad4 requires 2 x 2 Gauss integration.
- `Bu` is 3 x 8, `BT` is 2 x 4.
- `Kuu` is 8 x 8, `KTT` is 4 x 4, `KuT` is 8 x 4.
- The full interleaved matrix is 12 x 12.

## 7. Thermomechanical Boundary Element

File: `Edge2_ThermoMech.m`

Boundary input:

```matlab
model.natBC = {[edgeNodes], [tx, ty, M, S]};
```

Core edge shape function:

```matlab
N = [(1-s)/2, (1+s)/2];
```

Mechanical traction:

```matlab
Nmat = [N(1) 0    N(2) 0;
        0    N(1) 0    N(2)];

fu = Nmat' * [tx; ty] * J * w(igp) * thickness;
```

Thermal Robin or flux contribution:

```matlab
KT = (N' * M * N) * J * w(igp) * thickness;
fT = (N' * S) * J * w(igp) * thickness;
```

Interleaved edge DOFs:

```matlab
mechDofs = [1 2 4 5];
tempDofs = [3 6];
```

Logic:

- A thermomechanical edge can contribute both mechanical traction and thermal boundary matrix/vector.
- If the boundary is pure heat flux, set `M = 0` and use `S` as the heat flux term according to the project convention.
- If the boundary is convection, use `M = h` and `S = h*Tinf`.

## 8. Essential Boundary Conditions

File: `buildBCData.m`

The current framework uses node-group based BCs. For thermomechanics, each prescribed node gets all three values:

```matlab
model.essBC = {
    nodeIDs, [uxValue, uyValue, TValue]
};
```

Free thermal expansion patch test:

```matlab
for n = 1:size(model.nodesInfo,1)
    x = model.nodesInfo(n,2);
    y = model.nodesInfo(n,3);
    essBC(end+1,1:2) = {n, [alpha*DeltaT*x, alpha*DeltaT*y, Tuni]};
end
```

Knowledge point:

```math
u_x = \alpha \Delta T x
```

```math
u_y = \alpha \Delta T y
```

If the body expands freely under a uniform temperature increase, stress should be close to zero.

Logic:

- If the problem gives prescribed temperature and displacement on the same boundary, put them in `essBC`.
- If only temperature is prescribed but displacement is not, the current framework may need component-wise BC support. Otherwise, provide complete `[ux, uy, T]` values for prescribed nodes.
- For constrained expansion, do not use the free-expansion displacement field.

## 9. Assembly and Solver

File: `assembleSystem.m`

```matlab
[eleMatrix, eleVector] = eleData.func(eleNodesInfo, material, ...
    model.problem, loads, struct());

eleDoFs = getElementDoFs(eleNodeIDs, nNodeDoF);

globalMatrix(eleDoFs, eleDoFs) = globalMatrix(eleDoFs, eleDoFs) + eleMatrix;
globalVector(eleDoFs) = globalVector(eleDoFs) + eleVector;
```

File: `solveSystem.m`

```matlab
freeDoFs = setdiff(allDoFs, prescribedDoFs);

solution(freeDoFs) = globalMatrix(freeDoFs, freeDoFs) \ ...
    (globalVector(freeDoFs) - globalMatrix(freeDoFs, prescribedDoFs) * prescribedValues);

reaction = globalMatrix * solution - globalVector;
```

Logic:

- Assembly is the same for Tri3 and Quad4 as long as `eleDoFs` is correct.
- Because each node has `[ux, uy, T]`, a four-node Q4 element maps to 12 global DOFs.
- Reactions include mechanical and thermal constrained DOFs, depending on what was prescribed.

## 10. Postprocessing

Files:

```text
post_Tri3_ThermoMech.m
post_Quad4_ThermoMech.m
```

Core calculation:

```matlab
strain = Bu * u;
Tc = N' * T;
strainTh = alpha * (Tc - Tref) * [1; 1; 0];
stress = D * (strain - strainTh);

gradT = BT * T;
q = -Kcond * gradT;
```

Logic:

- Mechanical strain comes from displacement.
- Thermal strain comes from temperature difference relative to `Tref`.
- Heat flux is negative conductivity times temperature gradient.
- In a free expansion patch test, stress should be nearly zero.

## 11. Modify for Common Cases

| Change | Where to modify | Logic |
|---|---|---|
| Change material | `model.section` | Rows store `[eleID, E, nu, alpha, kxx, kyy]`. |
| Plane strain instead of plane stress | `model.problem.mechType` | Changes `D`; geometry and assembly stay the same. |
| Nonzero reference temperature | `model.problem.Tref` and element RHS | Ensure the `Tref` mechanical load contribution is included. |
| Change uniform temperature rise | `DeltaT`, `Tuni`, `essBC` | For free expansion use `u = alpha*DeltaT*[x,y]`. |
| Constrained thermal expansion | `essBC` | Fix the required displacement components; do not prescribe free-expansion displacement. |
| Add mechanical body force | `model.eleLoadData{e}.bodyForce` | Enters `fMech`. |
| Add heat source | `model.eleLoadData{e}.heatSource` | Enters `fTherm`. |
| Add traction boundary | `model.natBC` | Use `[tx,ty,M,S]` with thermal parts zero if not needed. |
| Add convection boundary | `model.natBC` | Use `M=h`, `S=h*Tinf`. |
| Use only Quad4 | `nTriCols = 0` | Check Q4 connectivity and positive `detJ`. |
| Use only Tri3 | `nTriCols = nx` | Check positive triangle area. |

## 12. Checks

Run:

```matlab
eval(fileread('#completion_checks.m'))
```

What it checks:

- Tri3 and Quad4 element matrix sizes.
- Mechanical and thermal block symmetry.
- Body force and heat source resultants.
- Edge boundary integration.
- Solver partitioning.
- Free thermal expansion patch test.

Fast manual checks:

1. Tri3 `Area > 0`.
2. Quad4 `detJ > 0` at each Gauss point.
3. `Kuu` and `KTT` should be symmetric blocks.
4. The full thermomechanical matrix is generally not symmetric because of `KuT`.
5. Free expansion under uniform temperature should give near-zero stress.

