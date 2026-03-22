# Finite element method

## 第一部分：什么是有限元？ 

$\quad$ 在 FEM 中，最核心的关系式是：

$$
F = K * d
$$

  $&emsp;&emsp; F$: 力（Load）  
  $&emsp;&emsp; K$: 刚度（Stiffness，代表物体抵抗变形的能力）  
  $&emsp;&emsp; d$: 位移（Displacement）  
$\quad$目标通常是：已知力 $F$ 和物体的属性 $K$，求出哪里发生了多大的位移 $d$

## 第二部分：FEM 的“游戏规则” 

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
为了让计算机能够处理任意长度和位置的单元，我们引入了自然坐标系 (Natural Coordinate System) $\xi$。  
$\quad  \cdot$ 无论实际杆件在全局坐标系 $x$ 中的长度 $L$ 是多少，在自然坐标系中，它的起点永远是 $\xi = -1$，终点永远是 $\xi = 1$。  
$\quad  \cdot$ 等参概念 (Iso-parametric) 的意思是：我们用同一套函数来同时插值“几何坐标”和“位移”。
