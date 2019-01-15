### Compositional Verification of a BabyVMM

Alexander Vaynberg and Zhong Shao

Yale University

---

## Virtual Memory Manager

- OS组成部分之一，管理内存
    + 向外提供了逻辑上大容量可寻址的内存存储器
    + BabyVMM提供了内存访问接口
        * mem_alloc, mem_free
        * as_request, as_release
- Verification a BabyVMM
    + 设计系统时，为内存管理模块制定对外接口并制订了相应specifications
    + 如果具体实现满足这些specifications，则实现是正确的


---

## Logical Address Space


![as](image/as.png)

~~~

## Assumption

- `$ \mathcal{M}_{AS}, \mathcal{L}_{AS} \vdash \mathbb{C}^{kernel} : \Psi_{AS}^{kernel}$`

~~~


## Physical Address Space


![hw](image/hw.png)

~~~


## Architecture of BabyVMM

![arch_babyVMM](image/arch_BabyVMM.png)


~~~

## Goal

+ <small> `$ \mathcal{M}_{HW}, \mathcal{L}_{HW} \vdash \mathbb{C}^{kernel} \cup \mathbb{C}^{as} \cup \mathbb{C}^{pt} \cup \mathbb{C}^{mem} \cup \mathbb{C}^{init} : \\ \quad \Psi_{HW}^{kernel} \cup \Psi_{HW}^{as} \cup  \Psi_{HW}^{pt} \cup \Psi_{HW}^{mem} \cup \Psi_{HW}^{init}$` </small>
+ <small> `$$ dom(\mathcal{L}_{HW}) = \{hw\_setPE, hw\_setPTROOT \} $$` </small>


~~~

![kernel_certified](image/kernel_certified.png)

---


## Definition

`$$ \mathcal{M}, \mathcal{L} \vdash \mathbb{C} : \Psi $$`

~~~

### Proc Heap

- <small>`$\mathbb{C}\ \ ::= \ \ \{ l \rightarrow \mathbb{I} \}^{*}$`</small>
- <small>`$\mathbb{I} \ \ ::= \ \ nil | \iota | [l] | \mathbb{I}_{1};\mathbb{I}_{2} | (b?\mathbb{I}_{1}+\mathbb{I}_{2})$`</small>


~~~

### Machine

- Language/Machine: `$ \mathcal{M} \ \in \ (\Sigma, \Delta, \beta, \gamma, OS)$`
- Operation: `$\iota \ \in \ \Delta$`
- Operational Semantics: `$OS \ \in \ \{ \iota \rightarrow (p, g) \}^{*}$`

~~~

### Specifications

- State Predicate: `$p \ \in \ \Sigma \rightarrow Prop$`
- State Relation: `$g \ \in \ \Sigma \rightarrow \Sigma \rightarrow Prop$`
- Spec Heap: `$ \Psi, \mathcal{L} \ ::= \{ l \rightarrow (p, g) \}^{*} $`

---

## Refinement


![refinement](image/direct_refinement.png)

~~~

### Definition of Certified Refinement


<small>
`$$\frac{  \mathcal{M}_{A}, \mathcal{L}_{A} \vdash \mathbb{C}_{A} : \Psi_{A} \quad Acc(\mathcal{M}_{A}, \mathcal{L}_{A} \vdash \mathbb{C}_{A} : \Psi_{A}) }{ \mathcal{M}_{C}, T_{\Psi}(\mathcal{L}_{A}) \vdash T_{\mathbb{C}}(\mathbb{C}_{A}) : T_{\Psi}(\Psi_{A}) }$$`
</small>

- a pair of relations <small>`$(T_{\mathbb{C}}, T_{\Psi})$`</small>
- a predicate over the abstract certified module `$Acc$`


~~~

### Example

<small>
- Refinement的意义在于，在Abstract Machine下验证的代码，只需建立与Concrete Machine的Refinement Relation，即在Concrete Machine下验证，屏蔽了底层实现的复杂度
- 例如，`pt_set`在ALE Model这个以`mem_alloc; mem_free`为<small>`$\mathcal{L}$`</small>的Machine中进行定义并验证，建立ALE Model和HW的Refinement，就得到
  + <small>`$\mathcal{M}_{HW}, T_{ALE-HW}(\mathcal{L}_{ALE}) \ \vdash \mathbb{C}(pt\_set) : T_{ALE-HW}\left( \Psi_{ALE}(pt\_set) \right) $`</small>
</small>


~~~

### Code-preserving Refinement


- In this plan for certification, `$\mathcal{M}_{A}.\Delta = \mathcal{M}_{C}.\Delta \ \land \ \mathcal{M}_{A}.\beta = \mathcal{M}_{C}.\beta$`
- So when refine the procedures of code, the procedures and proc heaps can stay exactly the same, and only the specifications change.
- Using this restriction, define a refinement rule using only a predicate `$T_{\alpha}$`

<img src="image/spec_trans.png"  alt="spec trans" width="55%" height="55%"/>


~~~

### Formal Definition

`$$ \frac{ \mathcal{M}_{A}, \mathcal{L}_{A} \vdash \mathbb{C} : \Psi_{A}}{  \mathcal{M}_{C}, T_{\Psi}(\mathcal{L}_{A}) \vdash \mathbb{C} : T_{\Psi}(\Psi_{A}) }$$`

where <small> `$T_{\Psi}(\Psi) := \{T_{\alpha} \left( \Psi(l) \right) \ | \ l \in dom(\Psi) \}$` </small>


~~~

### Representation Refinement

- 考虑自动化生成Spec Translation的方法
- 定义Machine间State的关系：<small>`$repr: \mathcal{M}_A.\Sigma \rightarrow \mathcal{M}_C.\Sigma \rightarrow Porp $`</small>
- 使用**repr**，可以定义specification translation function
  - <small>`$T_{A-C}(p,g) = (\lambda \mathbb{S}_C.\exists \mathbb{S}_A.repr\ \mathbb{S}_A \ \mathbb{S}_C \ \ \land \ \ p\ \mathbb{S}_A, \\ \quad \lambda \mathbb{S}_C.\lambda \mathbb{S}^{'}_C.\forall \mathbb{S}_A.repr\ \mathbb{S}_A\ \mathbb{S}_C \rightarrow \forall \mathbb{S}^{'}_A.g\ \mathbb{S}_A \ \mathbb{S}^{'}_A \rightarrow repr\ \mathbb{S}^{'}_A \ \mathbb{S}^{'}_C)$`</small>
- 可证**repr**-refinement valid


---

## Linking

- Linking two modules together, replace the stubs in both modules for the actual procedures
- Formally
  - <small> `$\frac{\mathcal{M},\mathcal{L}_{1} \vdash \mathbb{C}_{1}:\Psi_{1} \quad \mathcal{M},\mathcal{L}_{2} \vdash \mathbb{C}_{2}:\Psi_{2} \quad \mathbb{C}_{1} \perp \mathbb{C}_{2} \quad \mathcal{L}_1 \perp \Psi_{2} \quad \mathcal{L}_2 \perp \Psi_1 \quad \mathcal{L}_1 \perp \mathcal{L}_2 }{\mathcal{M}, ( (\mathcal{L}_{1} \cup \mathcal{L}_{2}) \backslash (\Psi_{1} \cup \Psi_{2})) \  \vdash \ \mathbb{C}_{1} \cup \mathbb{C}_{2} : \Psi_{1} \cup \Psi_{2}}$` </small>
  - <small> where <small> `$\Psi_1 \perp \Psi_2 \ ::= \ \forall l \in dom(\Psi_1).(l \not \in dom(\Psi_2) \lor \Psi_1 (l)=\Psi_2 (l))$` </small> </small>


~~~

### Stub Strengthening

- For the case that the stubs of one module are waker than the specifications of the procedures that will replace the stubs
- Formally:
  - If <small>`$ \mathcal{M}, \mathcal{L} \vdash \mathbb{C}:\Psi$`</small>, then for any <small>`$ \mathcal{L}^{'} \ s.t. \ \forall l \in dom(\mathcal{L}). \mathcal{L}(l) \supseteq \mathcal{L}^{'}(l) \land dom(\mathcal{L}^{'}) \cap dom(\Psi) = \emptyset $`</small>
  - the following holds: <small>`$ \mathcal{M}, \mathcal{L}^{'} \vdash \mathbb{C}:\Psi$`</small>

~~~

### Example

- 利用Refinement，将所有Procedures在不同的<small>`$\mathcal{L}$`</small>下验证
- 利用Linking，link在一起，得到Goal

---

## Layers


<img src="image/layers.png"  alt="layers" width="80%" height="80%"/>

~~~

<img src="image/complete_plan.png"  alt="Certification" width="55%" height="55%"/>


---

## C Machine

- 定义了C的语义
- 将Memory Model作为State的一部分抽象出来，提供了`load`和`save`两个接口


~~~

### Refinement in C machines

- C machines只是在Memory Models不同
- 因此可以利用Code-preserving Refinement和Repr Refinement
- 因此只需建立State间的**repr**
  - `$M_1 \leq M_2$` iff following holds:
    - <small>`$\forall l,v.load(M_1,l)=v \rightarrow load(M_2,l)=v \\ \forall l, v,M^{'}_1. (M^{'}_1=(store(M_1,l,v))) \rightarrow (M^{'}_1 \leq (store(M_2,l,v)))$`</small>
  - 构造**repr**
    - <small>`$repr \ := \ \lambda \mathbb{S}_A,\mathbb{S}_C. (\mathbb{S}_C.S = \mathbb{S}_C.S) \land (\mathbb{S}_A . M \leq \mathbb{S}_C .M)$`</small>


---

## Memory Model

- 根据Machine之间的特殊性，得到了新的C Machines间的Refinement
- Memory Model对外只提供`load`和`save`两个接口，Procedures中无法直接访问其数据结构
- 对Memory结构的访问会出现在Specifications中，因此需要Spec Translation解决不同Memory Model Refinement时的Translation


~~~


### Address Translated Memory Model

![HW](image/hw_model.png)

~~~

### Allocated Memory Model

<img src="image/ale_model.png" width="60%" heigth="60%" />


~~~

### Page Map Memory Model

<img src="image/pmap_model.png" width="70%" heigth="70%" />


~~~


### Address Space Memory Model


<img src="image/as_model.png" width="60%" heigth="60%" />


---

## Certification and Linking


- 在每个Machine下，验证相应的Procedures
- 以<small>`$T_{AS-PMAP} \circ T_{PMAP-ALE} \circ T_{ALE-PE} \circ T_{PE-HW}$`</small>的顺序进行Refinement
- 利用Linking得到结果
  + <small> `$ \mathcal{M}_{HW}, \mathcal{L}_{HW} \vdash \\ \quad \mathbb{C}^{kernel} \cup \mathbb{C}^{as} \cup \mathbb{C}^{pt} \cup \mathbb{C}^{mem} \cup \mathbb{C}^{init} : \\ \quad T_{AS-HW}\left( \Psi_{AS}^{kernel} \right) \cup T_{PMAP-HW}\left(\Psi_{PMAP}^{as} \right) \cup \\ \quad T_{ALE-HW}\left( \Psi_{ALE}^{pt} \right) \cup T_{PE-HW}\left( \Psi_{PE}^{mem} \right) \cup T_{PD-HW}\left( \Psi_{PD}^{init} \right) $`</small>

---

# End
