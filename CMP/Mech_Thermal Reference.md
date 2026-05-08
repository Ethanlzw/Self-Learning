# Mech_Thermal Reference

## 1. Project Role

This folder contains independent 2D mechanical and thermal FEM solvers:

| Problem type | Main script | Element routines | Field per node |
|---|---|---|---|
| 2D linear elasticity | `main_mixed_mech.m` | `Tri3_Mech.m`, `Quad4_Mech.m` | `[ux, uy]` |
| 2D steady heat conduction | `main_mixed_thermal.m`, `main_mixed_thermal_Fixed.m` | `Tri3_Thermal.m`, `Quad4_Thermal.m` | `[T]` |

Tasks usually target these operations:

1. Derive an element stiffness matrix or force vector.
2. Fill missing element code.
3. Assemble a mixed Tri3/Quad4 model.
4. Change essential boundary conditions, natural boundary conditions, or loads.
5. Run checks: matrix size, force balance, heat source balance, positive area, and positive Jacobian.

## 2. Main Script Pattern

The main scripts all follow the same model-building pipeline:

```matlab
model.fieldNames = {'ux','uy'};     % mechanical
% or
model.fieldNames = {'T'};           % thermal

model.elesType = {
    1, 'Tri3_Mech',  2, 3;
    2, 'Quad4_Mech', 2, 4
};
```

Knowledge point:

- `fieldNames` defines the nodal unknowns.
- `elesType` maps each element set to an element routine, number of DOFs per node, and number of nodes per element.
- The assembly code does not know the formula for each element. It only calls the element function registered here.

## 3. Mesh and Connectivity

Each rectangular cell is first named by its physical corner nodes:

```matlab
n1 = node(i  , j  );     % bottom-left
n2 = node(i+1, j  );     % bottom-right
n3 = node(i+1, j+1);     % top-right
n4 = node(i  , j+1);     % top-left
```

Triangular split:

```matlab
triElems = [triElems; eleID, n1, n2, n3];
triElems = [triElems; eleID, n1, n3, n4];
```

Quad4 connectivity:

```matlab
quadElems = [quadElems; eleID, n1, n2, n3, n4];
```

Knowledge point:

- The current Q4 convention is Lecture 4-2 Fig. 4.4.3:
  `1 = bottom-left`, `2 = bottom-right`, `3 = top-right`, `4 = top-left`.
- If the shape functions use a different local numbering convention, this connectivity must be changed together with `N`, `dNds`, and `dNdt`.
- If the node direction is reversed, `detJ` becomes negative and the element result is invalid.

To change the mesh mix:

```matlab
nTriCols = 0;    % pure Quad4
nTriCols = nx;   % pure Tri3
nTriCols = floor(nx/2);  % mixed Tri3/Quad4
```

## 4. Tri3 Mechanical Element

File: `Tri3_Mech.m`

Core geometric constants:

```matlab
A = [1 x1 y1;
     1 x2 y2;
     1 x3 y3];

detA = det(A);
Area = detA / 2;

if Area <= 0
    error('Tri3_Mech has non-positive area. Check node ordering.');
end

b1 = y2 - y3;  c1 = x3 - x2;
b2 = y3 - y1;  c2 = x1 - x3;
b3 = y1 - y2;  c3 = x2 - x1;
```

Mechanical strain-displacement matrix:

```matlab
B = [b1 0  b2 0  b3 0;
     0  c1 0  c2 0  c3;
     c1 b1 c2 b2 c3 b3] / detA;
```

Element stiffness and body force:

```matlab
eleMatrix = thickness * Area * (B' * D * B);

eleVector = thickness * Area / 3 * [bodyForce(1);
                                    bodyForce(2);
                                    bodyForce(1);
                                    bodyForce(2);
                                    bodyForce(1);
                                    bodyForce(2)];
```

Theory mapping:

```math
K_e = t A B^T D B
```

```math
f_e = \int_{\Omega_e} N_u^T b \, t \, dA
```

logic:

- Use Tri3 when the element is a constant-strain triangle.
- The `B` matrix is constant over the element, so no Gauss loop is needed.
- A constant body force is split equally among the three nodes.
- If `Area <= 0`, reorder the local nodes counter-clockwise.

## 5. Tri3 Thermal Element

File: `Tri3_Thermal.m`

Core thermal gradient matrix:

```matlab
B = [b1 b2 b3;
     c1 c2 c3] / detA;
```

Thermal conductivity and element matrix:

```matlab
Kcond = [kxx 0;
         0   kyy];

eleMatrix = thickness * Area * (B' * Kcond * B);
```

Heat source vector:

```matlab
if isfield(loads,'heatSource')
    qv = loads.heatSource;
    eleVector = eleVector + thickness * Area * [1/3; 1/3; 1/3] * qv;
end
```

Theory mapping:

```math
K_e = t A B_T^T K B_T
```

```math
f_e = \int_{\Omega_e} N_T^T r \, t \, dA
```

logic:

- Do not use the 3 x 6 mechanical `B` matrix for heat conduction.
- Thermal `B` is 2 x 3 for Tri3.
- A constant heat source is split equally among the three temperature nodes.

## 6. Quad4 Mechanical Element

File: `Quad4_Mech.m`

Current Q4 shape functions:

```matlab
N = 0.25 * [(1 - s) * (1 - t);
            (1 + s) * (1 - t);
            (1 + s) * (1 + t);
            (1 - s) * (1 + t)];

dNds = 0.25 * [-(1-t);  (1-t);  (1+t); -(1+t)];
dNdt = 0.25 * [-(1-s); -(1+s);  (1+s);  (1-s)];
```

Jacobian and physical derivatives:

```matlab
J = [dNds'*x, dNds'*y;
     dNdt'*x, dNdt'*y];

detJ = det(J);
if detJ <= 0
    error('Quad4_Mech has non-positive detJ at a Gauss point.');
end

gradN = J \ [dNds'; dNdt'];
dNdx = gradN(1, :);
dNdy = gradN(2, :);
```

Mechanical `B` matrix:

```matlab
B = [dNdx(1)      0    dNdx(2)   0      dNdx(3)     0      dNdx(4)    0;
       0      dNdy(1)    0     dNdy(2)    0      dNdy(3)    0       dNdy(4);
     dNdy(1)  dNdx(1)  dNdy(2) dNdx(2)  dNdy(3)  dNdx(3)   dNdy(4)  dNdx(4)];
```

Gauss accumulation:

```matlab
eleMatrix = eleMatrix + (B' * D * B) * detJ * w * thickness;

Nmat = [N(1)   0    N(2)    0    N(3)   0    N(4)   0;
         0    N(1)    0    N(2)   0    N(3)   0    N(4)];

eleVector = eleVector + Nmat' * bodyForce * detJ * w * thickness;
```

Theory mapping:

```math
K_e = \sum_g B_g^T D B_g \det(J_g) w_g t
```

logic:

- Quad4 is isoparametric. Derivatives change over the element, so use 2 x 2 Gauss integration.
- Always check whether `dNds` and `dNdt` are column or row vectors before building `J`.
- `detJ <= 0` means the local node direction or element geometry is wrong.
- For mechanical Q4, `B` is 3 x 8.

## 7. Quad4 Thermal Element

File: `Quad4_Thermal.m`

The same geometry and Jacobian workflow is used, but the field is scalar:

```matlab
gradN = J \ [dNds'; dNdt'];   % 2 x 4
B = gradN;

eleMatrix = eleMatrix + (B' * Kcond * B) * detJ * w * thickness;
```

Heat source:

```matlab
if isfield(loads,'heatSource')
    qv = loads.heatSource;
    eleVector = eleVector + N * qv * detJ * w * thickness;
end
```

Theory mapping:

```math
K_e = \sum_g B_{T,g}^T K B_{T,g} \det(J_g) w_g t
```

logic:

- Thermal Q4 `B` is 2 x 4.
- Do not construct the 3 x 8 mechanical strain matrix.
- If conductivity is anisotropic, change `material = [kxx, kyy]`.

## 8. Natural Boundary Conditions

Mechanical traction file: `Edge2_MechTraction.m`

```matlab
N = [(1-s)/2, (1+s)/2];

Nmat = [N(1) 0    N(2) 0;
        0    N(1) 0    N(2)];

edgeVector = edgeVector + Nmat' * [tx; ty] * J * w(igp) * thickness;
```

Thermal Robin/flux file: `Edge2_Thermal.m`

```matlab
N = [(1-s)/2, (1+s)/2];

edgeMatrix = edgeMatrix + (N' * M * N) * J * w(igp) * thickness;
edgeVector = edgeVector + (N' * S) * J * w(igp) * thickness;
```

Main-script input:

```matlab
model.natBC = {[rightEdgeNodes], [tx, ty]};     % mechanical
model.natBC = {[edgeNodes], [M, S]};            % thermal Robin/flux
```

logic:

- Essential BCs prescribe unknowns directly.
- Natural BCs contribute equivalent nodal force or heat vectors.
- Boundary integration uses a 2-node line element, not the area element.
- Check that the boundary node chain follows the actual boundary.

## 9. Assembly and DOF Mapping

File: `getElementDoFs.m`

```matlab
eleDoFs((a-1)*nNodeDoF+i) = (eleNodeIDs(a)-1)*nNodeDoF + i;
```

File: `assembleSystem.m`

```matlab
[eleMatrix, eleVector] = eleData.func(eleNodesInfo, material, ...
    model.problem, loads, struct());

eleDoFs = getElementDoFs(eleNodeIDs, nNodeDoF);

globalMatrix(eleDoFs, eleDoFs) = globalMatrix(eleDoFs, eleDoFs) + eleMatrix;
globalVector(eleDoFs) = globalVector(eleDoFs) + eleVector;
```

Knowledge point:

- Local element contributions are added into global rows and columns based on the global node numbers.
- Shared nodes receive contributions from multiple elements.

## 10. Essential BCs and Solver

File: `buildBCData.m`

Typical inputs:

```matlab
model.essBC = {[leftEdgeNodes], [0 0]};    % mechanical ux=0, uy=0
model.essBC = {[leftEdgeNodes], 100;       % thermal T=100
               [rightEdgeNodes], 0};
```

File: `solveSystem.m`

Core partitioning:

```matlab
freeDoFs = setdiff(allDoFs, prescribedDoFs);

Kff = globalMatrix(freeDoFs, freeDoFs);
Kfp = globalMatrix(freeDoFs, prescribedDoFs);

solution(freeDoFs) = Kff \ (globalVector(freeDoFs) - Kfp * prescribedValues);
reaction = globalMatrix * solution - globalVector;
```

logic:

- If a displacement or temperature is known, it belongs in `essBC`.
- If a force, traction, heat flux, or convection is given, it belongs in `natBC` or element loads.
- Reactions are computed after the full solution is recovered.

## 11. How to Modify for Common Cases

| change | Where to modify | Logic |
|---|---|---|
| Change Young's modulus or Poisson's ratio | `model.section` in `main_mixed_mech.m` | Section rows store `[eleID, E, nu]`. |
| Plane strain instead of plane stress | `model.problem.mechType` | `getElasticMatrix2D` switches `D`. |
| Change thickness | `model.problem.thickness` | Element matrices and vectors scale by thickness. |
| Change body force | `model.eleLoadData{e}.bodyForce` | Mechanical area load enters `eleVector`. |
| Change heat source | `model.eleLoadData{e}.heatSource` | Thermal source enters `eleVector`. |
| Change fixed edge | `model.essBC` | Use the node list on the new constrained boundary. |
| Change traction edge | `model.natBC` | Use the edge node chain and `[tx,ty]`. |
| Change temperature boundary | `model.essBC` | Use scalar values for thermal models. |
| Add convection/Robin boundary | `model.natBC` in thermal script | Use `[M,S]` where `M=h`, `S=h*Tinf` in the current framework. |
| Switch between Tri3 and Quad4 | `nTriCols` and element sets | Ensure connectivity and shape functions remain consistent. |

## 12. Checks

Run:

```matlab
completion_checks
```

What it checks:

- Element matrix sizes.
- Symmetry of mechanical and thermal stiffness matrices.
- Total body force and heat source resultants.
- Edge integration resultants.
- Solver partitioning.
- Mixed Tri3/Quad4 assembly.

Fast manual checks:

1. Tri3 area must be positive.
2. Quad4 `detJ` must be positive at all Gauss points.
3. Constant body force resultant equals `area * thickness * bodyForce`.
4. Constant heat source resultant equals `area * thickness * qv`.
5. Reaction sum should balance external force.

