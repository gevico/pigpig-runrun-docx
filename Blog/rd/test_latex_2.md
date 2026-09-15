---
title: test_latex_2
description: 测试在长文章下的显示效果
published: true
date: 2026-09-15T13:18:13.280Z
tags: 
editor: markdown
dateCreated: 2026-09-15T13:18:13.280Z
---

# Layout polynomials

### A generating-function approach to the algebra of CuTe layouts

*Author:* rd coauthored with Hy4 Preview  ·  *Date:* 2026-09-05  ·  *Status:* working note, companion to [Shah 2024]

---

## Abstract

We attach to every CuTe layout $L$ its **image polynomial**

$$P_L(z)\;=\;\sum_{x\in[0,M)}z^{\,f_L(x)}\;=\;\prod_{i=0}^{\alpha}\bigl(1+z^{d_i}+\dots+z^{(M_i-1)d_i}\bigr)\;\in\;\mathbb N[z],$$

the generating function of the multiset $\mathrm{im}(f_L)$. The dictionary

$$\text{mode}=\text{factor},\qquad \text{concatenation}=\text{product},\qquad \text{image}=\text{Minkowski sum},$$
$$\mathrm{size}=P_L(1),\qquad \mathrm{cosize}=\deg P_L+1$$

turns the whole of layout algebra into polynomial arithmetic, and every structural statement of [Shah 2024, §2] becomes an identity between polynomials. We prove:

* **Theorem 3.3.** $f_L$ is a bijection $[0,M)\to[0,M)$ **iff** $P_L=\Phi_M:=1+z+\dots+z^{M-1}$ **iff** (after sorting and deleting trivial modes) the stride vector equals the vector of prefix products of the shape — i.e. iff $L$ is an odometer. Equivalently: *every* factorisation of $\Phi_M$ into elementary factors is mixed-radix (Corollary 3.4, an "interval" analogue of Hajós–de Bruijn theory).
* **Theorem 4.3.** Complementation is *completion of a divisibility chain*. Shah's admissibility conditions are precisely the statement that the interleaved sequence $1\mid d_0\mid N_0d_0\mid d_1\mid\cdots\mid N_\alpha d_\alpha\mid M$ is a chain, and $\mathrm{complement}(A,M)$ consists of the *even* links of that chain. This yields $P_AP_B=\Phi_M$ by telescoping, hence a five-line proof of [Shah, Prop. 2.7] and a one-line proof of the cosize identity [Shah, Rem. 2.9].
* **Theorem 5.2 (carry-defect formula).** For all $x,y\in\mathbb N$,

  $$\hat f_A(x+y)=\hat f_A(x)+\hat f_A(y)\;-\;\sum_{j=1}^{\alpha}\bigl(N_{j-1}d_{j-1}-d_j\bigr)\,\kappa_j(x,y),$$

  where $\kappa_j\in\{0,1\}$ is the carry out of the $(j-1)$st digit. Thus **the failure of additivity of a layout function is exactly the accumulation of carries, weighted by the gaps $g_j=N_jd_j-d_{j+1}$** — the same quantities whose divisibility constitutes admissibility. This makes the "overflow" phenomenon of [Shah, §1, §2.2] exactly computable, and identifies Shah's "intervals of definition" (Def. 2.16) as *carry-free windows*.

* **Theorem 5.6 (composition = split, take the tail, truncate).** For a single-mode $B=(N):(r)$ with $\hat\iota_A(r)=c\,\delta_i$, the layout $A\circ B$ is obtained from the **ordered factor list** of $P_A$ by splitting the $i$-th factor, $[M_i]_{d_i}=[c]_{d_i}\,[M_i/c]_{c\,d_i}$, dropping the prefix, and truncating the remaining tail to total capacity $N$. This explains the strict/weak asymmetry of [Shah, Def. 2.11(3) vs. Def. 2.12(2)] — **splitting requires divisibility, truncating does not** — and with it [Shah, Warning 2.21].
* **Theorems 5.10–5.11 (exact validity criterion).** A multi-term carry-defect formula gives, for a multi-mode $B$,
 $$f_{A\circ B}=\hat f_A\circ f_B\iff\sum\nolimits_t g_{\,t-1}C_t(\mathbf x)=0\ \text{ for every coordinate tuple },$$
 which for nested layouts (all $g_j\le0$ — in particular all sorted admissible ones) is equivalent to: **no carry occurs at any position with a non-zero gap**. Consequently [Shah, Def. 2.17(2)] is *sufficient but not necessary* (Cor. 5.12, explicit counterexamples).
* **Theorem 6.1 (logical division).** $A/B=(A\circ B,\;A\circ\bar B)$ and $P_A=P_{A\circ B}\cdot P_{A\circ\bar B}$, where $\bar B=\mathrm{complement}(B,M)$: logical division is the offset-side analogue of complementation, and it is legitimate for exactly the same reason that complementation is — both are *carry-free*.
* **§6.4 (generalised division).** Complementation is only a device for producing *complete coordinate systems*: for **any** layout $C$ with $P_C=\Phi_M$ and $A\circ C$ valid, $A/C:=A\circ C$ satisfies $P_{A/C}=P_A$ and $P_A=\prod_kP_{A\circ C^k}$ (Thm. 6.7). Refining a division is free of new carries — it is blocked only by the same divisibility that blocks splitting factors (Prop. 6.8). Divisions of a fixed $A$ form a poset whose shape depends on $A$, not just on $M$ (Rem. 6.9).
* **§7 (a hierarchy of generating functions).** The bivariate signature $G_L(u,z)=\sum_xu^xz^{f_L(x)}$ is a monoidal functor into the **semidirect product** $\mathbb N[u,z]\rtimes\mathbb N_{>0}$; it is faithful modulo coalescing and it determines $\hat f_L$ (Thm. 7.5), while $P_L=G_L(1,z)$ is precisely the quotient that trivialises the action. Since concatenation *is* the monoidal product and composition is *not*, no generating function can represent composition (§7.5): the minimal faithful target is the ordered factor list itself.
* **§7.7 (L3 made concrete).** A mode $(M):(d)$ **is** the morphism $d\to Md$ of the divisibility poset $\mathbf{Div}$; coalescing is composition there, and Shah's realisation $L(E,\alpha)$ is the word of the selected chain steps (Prop. 7.6–7.7) — the stride is the *source* of the morphism, so $\mathrm{Fact}$ needs no extra decoration. We also correct [Shah, Lemma 3.12]: it holds iff the composition is carry-free, hence for injective $\beta$ but not in general (Thm. 7.8).
* **§3.6 (wrap-around layouts).** Tiling $\mathbb Z/M$ instead of $[0,M)$: a **complete criterion** — $L$ cyclically tiles iff for every divisor $d>1$ of $M$ some mode has $d\mid M_id_i$ and $d\nmid d_i$ (Thm. 3.8, exhaustive on 1 587 620 layouts). Cyclic complementation exists strictly more often than in the interval theory and is highly non-unique (Thm. 3.10).
* **§3.7 (layouts over a finite abelian group) + §6.6 (hierarchical tiling).** Theorem 3.8's proof is character-theoretic and transports verbatim: over *any* finite abelian group $G$, $L$ tiles $G$ iff every nontrivial character vanishes on some mode's span but not on its step (Thm. 3.13, 249 610 layouts, 13 groups). Over $(\mathbb F_p)^k$ the tiling layouts are exactly the **ordered bases** (Prop. 3.14) — so the hierarchy of §6.6 is a **flag**, counted by Gaussian multinomials (Rem. 3.15). Consequently hierarchical tiling = a chain in $\mathrm{Div}(A)$, and Prop. 6.14 counts it: $d!\,S(N,d)$ hierarchies of depth $d$, from a $128{\times}128{\times}64$ block with a $16{\times}8{\times}8$ atom only $1\,806$ at depth 3.

We also record the linear-algebraic picture (§8): a layout is a pair of weight vectors $w$ (prefix products) and $d$ (strides) on the digit box, i.e. a linear map $\mathbb Z^{\alpha+1}\to\mathbb Z^2$, and $f_L=\mathrm{id}$ iff $w=d$; the deviation $\delta=d-w$ has the gaps $g$ as its discrete derivative.

All identities in this note were verified by exhaustive computation over the stated ranges; see Appendix A.

---

## 0. Introduction

### 0.1 Motivation

CuTe layouts (the core abstraction of NVIDIA CUTLASS since 3.0) are pairs of integer tuples $L=(M_0,\dots,M_\alpha):(d_0,\dots,d_\alpha)$ describing how a multi-dimensional logical index is realised as a one-dimensional physical offset. The associated **layout function** ([Shah 2024, Def. 2.3])

$$f_L(x)=d_0x_0+\dots+d_\alpha x_\alpha,\qquad (x_0,\dots,x_\alpha)=\iota(x),$$

is the composite of a *non-linear* digit map $\iota$ (determined by the shape, a mixed-radix expansion) with a *linear* functional (determined by the stride). The algebra of layouts — complementation, composition, logical division — is the study of this hybrid object, and it is precisely the hybrid nature that makes the literature delicate: the layout function is neither linear nor arbitrary, it is *linear on each carry-free region*.

The purpose of this note is to point out that there is a **single generating function which linearises the bookkeeping**, after which most of the technical machinery becomes elementary polynomial algebra. The idea is classical (support of a product $=$ Minkowski sum of supports; $\mathrm{supp}\leftrightarrow$ generating function) but does not appear to have been applied to CuTe layouts. Three payoffs:

1. **Brevity.** [Shah, Prop. 2.7] (three pages of sorting, maximality and mod-$r$ induction) and [Shah, Rem. 2.9] (a page of telescoping) become five lines and one line.
2. **Necessary and sufficient conditions.** The polynomial criterion for bijectivity (Thm. 3.3) is an *iff*, and its proof is an easy induction; it simultaneously classifies all factorisations of the interval polynomial.
3. **Quantitative control of the failure of additivity** (Thm. 5.2), which is the technical heart of composition.

### 0.2 Relation to [Shah 2024]

Nothing here contradicts [Shah 2024]; §§1–4 are a re-derivation and shortening of his §2.1, and §§5–6 a sharpening and strengthening of his §2.2–2.3. What is, to the best of our knowledge, new: Theorem 3.3 and its proof; Corollary 3.4; the chain reformulation of admissibility (Prop. 4.2); the diagnosis in §4.6 of what the CUTLASS implementation computes off the admissible locus; the two-term and multi-term carry-defect formulas (Thm. 5.2, 5.10); the description of composition as cutting the ordered factor list (Thm. 5.6); the exact validity criterion and the proof that [Shah, Def. 2.17(2)] is stronger than necessary (Thm. 5.11, Cor. 5.12); the reduction and generalisation of logical division (Thm. 6.1, §6.4); the hierarchy of generating functions of §7; the concrete description of L3 and the correction to [Shah, Lemma 3.12] in §7.7; and the linear picture of §8.

### 0.3 Conventions

$\mathbb N=\{0,1,2,\dots\}$. All layouts are flattened, non-empty, with positive shapes and strides ([Shah, Def. 2.1]), and we freely delete modes with $M_i=1$: they contribute the factor $1$ to $P_L$ and change neither $f_L$ nor any statement below. A layout is **sorted** if its strides are non-decreasing, ties broken by non-decreasing shape ([Shah, Def. 2.4]). We write

$$\Phi_M(z)\;:=\;1+z+\dots+z^{M-1}=\frac{1-z^M}{1-z}$$

for the **interval polynomial**, and for $r\ge 1$, $e\ge 1$,

$$[r]_e(z)\;:=\;1+z^e+z^{2e}+\dots+z^{(r-1)e}\;=\;\frac{1-z^{re}}{1-z^e}$$

for an **elementary factor**. Note $[r]_e(z)=[r]_{q}|_{q=z^e}$ is the $q$-analogue of the integer $r$, and $[r]_1=\Phi_r$ — a coincidence we exploit in Remark 2.6.

---

## 1. Recapitulation

**Definition 1.1 (layout, layout function).** A layout is $L=(M_0,\dots,M_\alpha):(d_0,\dots,d_\alpha)$ with $M_i,d_i\in\mathbb Z_{>0}$. Its **size** is $M=\prod_iM_i$; its **length** is $\alpha+1$; its **modes** are the length-$1$ layouts $(M_i):(d_i)$. The **digit map**

$$\iota:[0,M)\xrightarrow{\ \sim\ }\prod_{i}[0,M_i),\qquad x\mapsto\bigl(x\bmod M_0,\ \lfloor x/M_0\rfloor\bmod M_1,\ \dots\bigr),$$

is a bijection, and the **layout function** is $f_L(x)=\sum_id_ix_i$ with $(x_i)=\iota(x)$. The **extended layout function** $\hat f_L:\mathbb N\to\mathbb N$ is obtained by replacing $M_\alpha$ by $\infty$. The **cosize** is $\mathrm{cosize}(L)=f_L(M-1)+1=\sum_i(M_i-1)d_i+1$.

Given layouts $A,B$, the **concatenation** $(A,B)$ is the layout obtained by concatenating shape and stride tuples; then $\mathrm{size}(A,B)=\mathrm{size}(A)\mathrm{size}(B)$ and, writing $x=(a,b)$ under the evident identification,

$$f_{(A,B)}(x)=f_A(a)+f_B(b). \tag{1.1}$$

**Definition 1.2 (gaps).** For a sorted layout $A=(N_0,\dots,N_\alpha):(d_0,\dots,d_\alpha)$ put

$$g_j\;:=\;N_jd_j-d_{j+1}\qquad(0\le j<\alpha).$$

Thus $g_j=0$ for all $j$ iff $d_{j+1}=N_jd_j$ for all $j$: the strides are exactly the prefix products of the shapes (up to the common factor $d_0$). We shall see that the gaps measure *both* the failure of admissibility (§4) and the failure of additivity (§5).

---

## 2. The layout polynomial

**Definition 2.1.** The **image polynomial** of a layout $L$ of size $M$ is

$$P_L(z)\;:=\;\sum_{x\in[0,M)}z^{\,f_L(x)}\;\in\;\mathbb N[z].$$

Its coefficients are the fibre cardinalities: $[z^n]P_L=\#f_L^{-1}(n)$.

**Lemma 2.2 (product formula).** $\displaystyle P_L(z)=\prod_{i=0}^{\alpha}[M_i]_{d_i}(z)=\prod_{i=0}^{\alpha}\frac{1-z^{M_id_i}}{1-z^{d_i}}.$

*Proof.* Since $\iota$ is a bijection, $x\mapsto(x_0,\dots,x_\alpha)$ ranges over the whole box $\prod_i[0,M_i)$. Hence

$$\sum_{x}z^{f_L(x)}=\sum_{(x_i)}\prod_iz^{d_ix_i}=\prod_i\Bigl(\sum_{x_i=0}^{M_i-1}z^{d_ix_i}\Bigr)=\prod_i[M_i]_{d_i}. \qquad\square$$

**Corollary 2.3.** Let $L$ be a layout. Then

1. $\mathrm{size}(L)=P_L(1)=\prod_iM_i$;
2. $\mathrm{cosize}(L)=\deg P_L+1=\sum_i(M_i-1)d_i+1$;
3. $L$ is **injective** (i.e. $f_L$ is injective) iff every coefficient of $P_L$ is $0$ or $1$;
4. $\#\mathrm{im}(f_L)=\#\{n:[z^n]P_L>0\}$, and $\#\mathrm{im}(f_L)\le\min(\mathrm{size},\mathrm{cosize})$.

*Proof.* (1), (2) are evaluations of Lemma 2.2 at $z=1$ and of the degree; (3) is the fibre interpretation; (4) is immediate. $\square$

Thus **the two invariants CuTe cares about are the value at $1$ and the degree** — "volume" and "diameter".

**Proposition 2.4 (concatenation is multiplication).** $P_{(A,B)}=P_A\cdot P_B$.

*Proof.* Immediate from (1.1) and Definition 2.1, or from Lemma 2.2. $\square$

**Corollary 2.5 (the volume/diameter duality).**

$$\mathrm{size}(A,B)=\mathrm{size}(A)\cdot\mathrm{size}(B)\qquad\text{(multiplicative)},$$
$$\mathrm{cosize}(A,B)=\mathrm{cosize}(A)+\mathrm{cosize}(B)-1\qquad\text{(additive)}.$$

*Proof.* Evaluate $P_{(A,B)}=P_AP_B$ at $1$, and take degrees. $\square$

So under concatenation the *number of degrees of freedom multiplies* while the *span adds* — the familiar behaviour of dimension under $\oplus$ versus that of diameter, in a single operation.

**Remark 2.6 (coalescing; what $P_L$ forgets).** Since

$$[rs]_e=[r]_e\cdot[s]_{re}, \tag{2.1}$$

splitting a mode $(rs):(e)$ into $(r):(e),(s):(re)$ leaves $P_L$ unchanged. Consequently **$P_L$ is an invariant of the image multiset $\mathrm{im}(f_L)$, not of the layout**: e.g.

$$P_{(4):(1)}=P_{(2,2):(1,2)}=\Phi_4,$$

yet these are different layouts (different $\iota$, hence different coordinate decompositions of the same addresses). In the language of [Shah, fn. 13] this is *uncoalescing*. All statements below that are phrased in terms of $P_L$ are therefore statements about images; the layout structure is a *refinement* datum on top, and complementation (§4) is well-defined at the level of $P_L$ only up to (2.1). Note also that (2.1) is the $q$-number identity $[rs]_q=[r]_q[s]_{q^r}$ evaluated at $q=z^e$, so coalescing is exactly the classical divisibility of $q$-integers.

---

## 3. Tiling layouts and the classification theorem

**Definition 3.1.** A layout $L$ of size $M$ **tiles** $[0,M)$ if $f_L:[0,M)\to[0,M)$ is bijective (equivalently: injective, equivalently: surjective).

**Lemma 3.2.** $L$ tiles $[0,M)$ iff $P_L=\Phi_M$.

*Proof.* $P_L=\sum_n \#f_L^{-1}(n)z^n$ with total mass $M$; it equals $\Phi_M=\sum_{n<M}z^n$ iff every $n\in[0,M)$ has exactly one preimage. $\square$

For $L$ of size $M$ let

$$w(L)\;:=\;(1,\;M_0,\;M_0M_1,\;\dots,\;M_0\cdots M_{\alpha-1})$$

be the vector of **prefix products** of the shape. Note that $w$ is determined by the shape, the shape is determined by $w$ ($M_i=w_{i+1}/w_i$), and $x=\langle w,\iota(x)\rangle$ for all $x$: **the index is the linear functional of weight $w$, the offset is the linear functional of weight $d$**.

**Theorem 3.3 (characterisation of tilings).** For a layout $L$ of size $M\ge 2$ the following are equivalent:

1. $L$ tiles $[0,M)$;
2. $P_L=\Phi_M$;
3. after sorting $L$ and deleting modes with $M_i=1$, one has $d=w(L)$, i.e.
 $$d_0=1,\qquad d_i=M_0M_1\cdots M_{i-1}\ \ (1\le i\le\alpha).$$

Equivalently in the language of gaps: $d_0=1$ and $g_j=0$ for all $j$.

*Proof.* (1)$\iff$(2) is Lemma 3.2.

(3)$\Rightarrow$(2): if $d_0=1$ and $d_{i+1}=N_id_i$, then $M_id_i=d_{i+1}$ and $M_\alpha d_\alpha=M$, so

$$P_L=\prod_i\frac{1-z^{M_id_i}}{1-z^{d_i}}=\frac{1-z^{d_1}}{1-z^{d_0}}\cdot\frac{1-z^{d_2}}{1-z^{d_1}}\cdots\frac{1-z^{M}}{1-z^{d_\alpha}}=\frac{1-z^M}{1-z}=\Phi_M$$

by telescoping (this is [Shah, Prop. 2.7] in the special case $A=L$, $B$ empty).

(2)$\Rightarrow$(3): we induct on the number of non-trivial modes. Write $P_L=\prod_i[M_i]_{d_i}$ with all $M_i\ge 2$.

*Base case $\alpha=0$:* $[M_0]_{d_0}=\Phi_{M_0}$ forces $(M_0-1)d_0=M_0-1$ by degree, hence $d_0=1$.

*Induction step.* Every factor has the form $1+z^{d_i}+\dots$, so the coefficient of $z$ in $P_L$ equals $\#\{i:d_i=1\}$ (terms of degree $1$ arise only by taking $z^{d_i}$ from a single factor with $d_i=1$). Since $[z]\Phi_M=1$, exactly one mode has stride $1$; call it mode $0$ and let $N$ be its shape, so $P_L=\Phi_N\cdot Q$ with $Q=\prod_{i\ge1}[M_i]_{d_i}$. Then

$$Q=\frac{\Phi_M}{\Phi_N}=\frac{1-z^M}{1-z^N},$$

which is a polynomial only if $N\mid M$ (as $(1-z^N)\mid(1-z^M)$ iff $N\mid M$), and then $Q=[M/N]_N$.

Each summand $\{0,d_i,\dots,(M_i-1)d_i\}$ contains $0$, hence is contained in $\mathrm{supp}(Q)=\{0,N,2N,\dots,M-N\}$. Therefore $N\mid d_i$ for all $i\ge1$; write $d_i=Nd_i'$. Then $[M_i]_{d_i}(z)=[M_i]_{d_i'}(z^N)$, so $Q(z)=R(z^N)$ with $R=\prod_{i\ge1}[M_i]_{d_i'}$, and comparing with $Q=[M/N]_N=\Phi_{M/N}(z^N)$ gives $R=\Phi_{M/N}$. By the inductive hypothesis applied to the layout $L'=(M_1,\dots,M_\alpha):(d_1',\dots,d_\alpha')$ (which has one fewer mode and size $M/N$), after sorting we get $d_1'=1$ and $d_{i+1}'=M_id_i'$. Rescaling by $N$, and using $d_0=1$, $N_0=N$:

$$d_1=N=N_0d_0,\qquad d_{i+1}=Nd_{i+1}'=NM_id_i'=M_id_i,$$

i.e. $d=w(L)$ after sorting. $\square$

**Corollary 3.4 (all elementary factorisations of the interval are mixed-radix).** Suppose

$$\Phi_M=\prod_{j=1}^{k}[r_j]_{e_j},\qquad r_j\ge 2,\ e_j\ge 1 .$$

Then, after reindexing so that $e_1\le e_2\le\dots\le e_k$, we have $e_1=1$ and $e_{j+1}=r_je_j$ for all $j$; in particular $\prod_jr_j=M$ and the factorisation is the mixed-radix (odometer) factorisation attached to the ordered factorisation $M=r_1r_2\cdots r_k$.

*Proof.* The data $(r_j,e_j)$ define a layout $L=(r_1,\dots,r_k):(e_1,\dots,e_k)$ of size $\prod r_j$, and $P_L=\Phi_M$ forces $\prod r_j=M$ (evaluate at $1$). Apply Theorem 3.3. $\square$

**Remark 3.5 (interval versus cyclic tilings; the classical context).** Lemma 3.2 has an exact analogue in the group ring. Let $\mathbb Z[z]/(z^M-1)\cong\mathbb Z[\mathbb Z/M]$ and $\Sigma_M:=\sum_{g\in\mathbb Z/M}g$. Then

$$\textstyle A_1\oplus\dots\oplus A_k=\mathbb Z/M\ \text{(direct)}\iff \prod_j\Bigl(\sum_{a\in A_j}z^a\Bigr)=\Sigma_M\ \text{in }\mathbb Z[\mathbb Z/M],$$

the classical translation of **factorisations of finite abelian groups** into group rings (Hajós, de Bruijn, Rédei). Our Corollary 3.4 is the *interval* (i.e. no wrap-around, work in $\mathbb N[z]$ rather than modulo $z^M-1$) analogue, and it is strictly more rigid:

$$(2):(3)\ \oplus\ (3):(2)\quad\text{tiles }\mathbb Z/6,\ \text{but not }[0,6).$$

Indeed $P=(1+z^3)(1+z^2+z^4)=1+z^2+z^3+z^4+z^5+z^7\equiv\Phi_6 \pmod{z^6-1}$, but $P\ne\Phi_6$ in $\mathbb Z[z]$ — the element $7$ wraps to $1$. Tiling the *interval* forbids wrapping, and Corollary 3.4 says this forces the mixed-radix (de Bruijn's "British number system") form. We flag that we have *not* verified the precise statements of de Bruijn's classification of additive systems of $\mathbb N_0$ against the literature; Corollary 3.4 is proved here from scratch for the restricted class of elementary factors and is all we need.

### 3.6 Wrap-around layouts: tiling $\mathbb Z/M$

Everything so far concerned the *interval* $[0,M)$. Replacing it by the cyclic group $\mathbb Z/M$ gives a strictly richer theory — the right setting for modular / wrapped addressing (Question 9.4).

**Definition 3.7 (cyclic tiling).** For a layout $L$ of size $M$ let
$$\bar f_L:\mathbb Z/M\longrightarrow\mathbb Z/M,\qquad x\longmapsto f_L(x)\bmod M .$$
We say $L$ **cyclically tiles** if $\bar f_L$ is a bijection. In the group ring $R_M:=\mathbb Z[z]/(z^M-1)$ put $\bar P_L:=P_L\bmod (z^M-1)$ and $\Sigma_M:=1+z+\dots+z^{M-1}$. Then
$$L\ \text{cyclically tiles}\quad\iff\quad \bar P_L=\Sigma_M\ \text{ in }R_M\quad\iff\quad P_L\equiv\Phi_M\ \pmod{z^M-1}.$$
(The middle condition says precisely that $\{f_L(x)\bmod M\}$ is a complete residue system.) Interval tiling is the strictly stronger statement $P_L=\Phi_M$ in $\mathbb Z[z]$.

**Theorem 3.8 (cyclic tiling criterion).** Let $L=(M_0,\dots,M_\alpha):(d_0,\dots,d_\alpha)$ have size $M$. Then $L$ cyclically tiles $\mathbb Z/M$ **iff**
$$\textbf{for every } d\mid M,\ d>1,\ \textbf{ there is an index } i \textbf{ with }\ d\mid M_id_i\ \textbf{ and }\ d\nmid d_i . \tag{3.2}$$

*Proof.* $\bar P_L=\Sigma_M$ in $R_M$ holds iff $P_L(\zeta)=\Sigma_M(\zeta)$ for every $M$-th root of unity $\zeta$. At $\zeta=1$ both sides equal $M=\mathrm{size}(L)$. Let $\zeta\neq1$ have exact order $d\mid M$, $d>1$. Since $\Sigma_M=\prod_{e\mid M,\ e>1}\Phi^{\mathrm{cyc}}_e$ contains the factor $\Phi^{\mathrm{cyc}}_d$, we get $\Sigma_M(\zeta)=0$. On the other hand
$$P_L(\zeta)=\prod_i S_i,\qquad S_i:=\sum_{k=0}^{M_i-1}\zeta^{kd_i},$$
and a geometric sum $\sum_{k<m}w^k$ vanishes iff $w^m=1\neq w$. With $w=\zeta^{d_i}$ this reads $S_i=0\iff \zeta^{M_id_i}=1\neq\zeta^{d_i}\iff d\mid M_id_i$ and $d\nmid d_i$. A product vanishes iff one factor does, which is (3.2). $\square$

Thus cyclic tiling is a **covering problem**: each mode $(M_i):(d_i)$ covers the set $T_i=\{d\mid M,\ d>1:\ d\mid M_id_i,\ d\nmid d_i\}$, and $L$ tiles iff the $T_i$ cover all divisors $>1$ of $M$. Note that only $\gcd(M,d_i)$ and $\gcd(M,M_id_i)$ matter.

**Corollary 3.9.** A single mode $(M):(d)$ cyclically tiles $\mathbb Z/M$ **iff $\gcd(d,M)=1$**, i.e. iff $d$ is a unit mod $M$ — the unit group $(\mathbb Z/M)^\times$, of order $\varphi(M)$. (For the interval, Corollary 3.4 allows only $d=1$.)

**Theorem 3.10 (cyclic complementation).** Let $\mathrm{size}(A)\mid M$. Call $B$ a **cyclic complement of $A$ mod $M$** if $\mathrm{size}(A)\,\mathrm{size}(B)=M$ and $\bar P_A\bar P_B=\Sigma_M$ in $R_M$. Then:

1. if $\{A,M\}$ is admissible in the sense of Prop. 4.2, the (interval) complement of §4 is among the cyclic complements; in particular a cyclic complement always exists in that case (verified: 97/97);
2. the converse fails — cyclic complements exist strictly more often. Over all $A$ with $\mathrm{size}(A)\mid M$ and strides $<M$: admissible / cyclically complementable $=5/9$, $9/25$, $22/117$, $5/38$, $56/573$ for $M=4,6,8,9,12$; there were **no** $A$ that are admissible without a cyclic complement;
3. unlike §4, the cyclic complement is **far from unique**: for $M=12$, $A=(2):(6)$ has $68$, $A=(2):(1)$ has $18$, $A=(3):(1)$ has $6$.

**Remark 3.11 (size of the cyclic class; Hajós).** The cyclic class is much larger than the interval one:

| $M$ | 4 | 6 | 8 | 9 | 12 | 16 |
|---|---|---|---|---|---|---|
| interval tilings | 3 | 5 | 11 | 3 | 27 | 49 |
| cyclic tilings | 6 | 18 | 76 | 30 | 420 | 2328 |

Hajős's theorem guarantees that in any factorisation of a finite abelian group into subsets containing $0$, at least one factor is **periodic**; we verified that every cyclic tiling up to $M=16$ has a periodic mode. This is the source of the non-uniqueness in Thm. 3.10(3), and it is exactly why the cyclic theory cannot admit a Corollary-3.4-style classification into odometers.

**Remark 3.12 (a caveat on swizzle).** CuTe's **XOR-swizzle** is *not* of this type. The wrap-around theory upgrades $[0,M)$ to $\mathbb Z/M$ — still an additive, $\mathbb Z$-linear structure — whereas swizzle is XOR, i.e. $\mathbb F_2$-linear on the bits. A theory of swizzle therefore needs a *further* upgrade to $\mathbb F_2$-structure; we do not attempt it here (Question 9.4).

### 3.7 The cyclic theory over a general finite abelian group

The proof of Theorem 3.8 used only the *characters* of $\mathbb Z/M$. It therefore transports verbatim to any finite abelian group. This is worth recording: at $G=(\mathbb F_p)^k$ it produces an object with a genuinely different geometry (Remark 3.15), and it is the honest content behind the "$q$-analogue" of Question 9.6. ⚠️ **There is no CuTe object here** — §3.7 is pure mathematics, and we flag it as such.

Let $G$ be a finite abelian group (additive), $\Sigma_G:=\sum_{g\in G}g\in\mathbb Z[G]$, and for a mode $(M_i):(d_i)$ with $d_i\in G$ put $A_i:=\{0,d_i,2d_i,\dots,(M_i-1)d_i\}$. A **layout over $G$** is $L=\big((M_i):(d_i)\big)_i$ with $\prod_iM_i=|G|$; it **tiles $G$** if $G=\bigoplus_iA_i$, i.e. if $\prod_i\big(\sum_{a\in A_i}z^a\big)=\Sigma_G$ in $\mathbb Z[G]$.

**Theorem 3.13 (character criterion).** $L$ tiles $G$ **iff**
$$\textbf{for every }\chi\in\hat G\setminus\{1\}\textbf{ there is an index }i\textbf{ with }\chi(M_id_i)=1\textbf{ and }\chi(d_i)\ne1 . \tag{3.3}$$

*Proof.* The Fourier transform $\mathbb C[G]\to\mathbb C^{\hat G}$, $x\mapsto(\chi(x))_\chi$, is an algebra isomorphism; hence $P_L=\Sigma_G$ in $\mathbb Z[G]$ iff $\chi(P_L)=\chi(\Sigma_G)$ for every $\chi$. By orthogonality of characters $\chi(\Sigma_G)=\sum_g\chi(g)$ is $|G|$ for $\chi=1$ and $0$ otherwise. And $\chi(P_L)=\prod_i\sum_{a\in A_i}\chi(a)=\prod_i\sum_{j<M_i}\chi(d_i)^j$: the $i$-th factor is a geometric sum, vanishing iff $\chi(d_i)^{M_i}=1\ne\chi(d_i)$, i.e. iff $\chi(M_id_i)=1\ne\chi(d_i)$. A product vanishes iff some factor does. $\square$

For $G=\mathbb Z/M$ the characters are $\chi_j(a)=\zeta^{ja}$; "$\chi_j(a)=1$" reads "$d\mid a$" where $d$ is the order of $\chi_j$, and the orders occurring are exactly the divisors $>1$ of $M$ — recovering Theorem 3.8. (Verified: 249 610 layouts over 13 groups — $\mathbb Z/4,\mathbb Z/2^2,\mathbb Z/6,\mathbb Z/8,\mathbb Z/4{\times}\mathbb Z/2,\mathbb Z/2^3,\mathbb Z/9,\mathbb Z/3^2,\mathbb Z/12,\mathbb Z/6{\times}\mathbb Z/2,\mathbb Z/16,\mathbb Z/4^2,\mathbb Z/2^4$ — 0 mismatches.)

**Proposition 3.14 (the elementary abelian case).** Let $G=(\mathbb F_p)^k$. A tiling layout over $G$ has exactly $k$ modes, each of size $p$, and
$$\{\text{tiling layouts over }(\mathbb F_p)^k\}=\{\text{ordered bases of }\mathbb F_p^k\},\qquad \#=|GL_k(\mathbb F_p)|=\prod_{i=0}^{k-1}(p^k-p^i).$$

*Proof.* If some $M_i>p$ then $A_i$ has at most $p$ distinct elements (its period divides $\exp G=p$), so $\lvert\bigoplus_iA_i\rvert\le\prod_i\min(M_i,p)<\prod_iM_i=p^k$: $L$ cannot tile. Hence every $M_i\le p$; as $\prod_iM_i=p^k$ each $M_i$ is a power of $p$, so $M_i=p$ and there are $k$ modes. Then $A_i=\mathbb F_p\,d_i$ is the line through $d_i$, and $G=\bigoplus_i\mathbb F_pd_i$ iff the $d_i$ are linearly independent. $\square$

Verified at $(p,k)=(2,2),(2,3),(2,4),(3,2),(3,3),(5,2)$: $6,\,168,\,20160,\,48,\,11232,\,480$ tiling layouts — exactly $|GL_k(\mathbb F_p)|$ each time.

**Remark 3.15 (chains, flags, buildings — the real "$q$-analogue").** $\mathbb Z/p^k$ and $(\mathbb F_p)^k$ are the two extreme groups of order $p^k$, and their subgroup lattices are the two extreme geometries:

| | subgroups of order $p^j$ | flags with step set $S\subseteq\{1,\dots,k-1\}$ |
|---|---|---|
| $\mathbb Z/p^k$ | $1$ (unique) | $1$ |
| $(\mathbb F_p)^k$ | $\binom kj_p$ (Gaussian binomial) | $\binom kS_p$ (Gaussian multinomial) |

So the Boolean lattice $B_{k-1}$ of Theorem 6.11(2) is exactly the set of **step distributions** of a flag — which levels it stops at, i.e. its combinatorial type — and the Gaussian multinomial counts the flags realising each type. In the cyclic case the multiplicity degenerates to $1$. In Tits's language: $B_{k-1}$ is the **apartment** of type $A_{k-1}$, and the flag complex of the subspace lattice is the **building**. Thus hierarchical tiling over $\mathbb Z/p^k$ walks inside a single apartment, whereas over $(\mathbb F_p)^k$ it walks in a building. The intermediate groups of order $p^k$ (e.g. $\mathbb Z/p^2\times\mathbb Z/p$) interpolate.

---

## 4. Complementation

Throughout this section $A=(N_0,\dots,N_\alpha):(d_0,\dots,d_\alpha)$ is sorted and $M\in\mathbb Z_{>0}$.

**Definition 4.1 (the chain of a pair).** The **interleaved sequence** of $(A,M)$ is

$$\mathbf e(A,M)\;=\;\bigl(1,\;d_0,\;N_0d_0,\;d_1,\;N_1d_1,\;\dots,\;d_\alpha,\;N_\alpha d_\alpha,\;M\bigr),$$

a sequence of $2\alpha+4$ positive integers. Its **links** are the quotients $\ell_j=e_{j+1}/e_j$, $0\le j\le 2\alpha+2$.

**Proposition 4.2 (admissibility = chain condition).** $\{A,M\}$ is admissible for complementation in the sense of [Shah, Def. 2.5] **iff** $\mathbf e(A,M)$ is a divisibility chain ($e_j\mid e_{j+1}$ for all $j$).

*Proof.* The non-trivial divisibility requirements are exactly $N_{i-1}d_{i-1}\mid d_i$ for $1\le i\le\alpha$ and $N_\alpha d_\alpha\mid M$; the remaining ones ($1\mid d_0$, $N_id_i\mid N_id_i$) are vacuous. $\square$

Thus admissibility says: *the spans of the modes of $A$ sit on a single divisor chain from $1$ to $M$.*

**Theorem 4.3 (complementation = chain completion).** Suppose $\{A,M\}$ is admissible, with chain $\mathbf e$. Then:

1. The **odd** links are the shapes of $A$: $\ell_{2i+1}=N_i$, and $A$'s strides are the odd entries $d_i=e_{2i+1}$.
2. The **even** links are the shapes of $B:=\mathrm{complement}(A,M)$ as defined in [Shah, Def. 2.6]:
 $$\ell_0=d_0,\quad \ell_{2i}=\frac{d_i}{N_{i-1}d_{i-1}}\ (1\le i\le\alpha),\quad \ell_{2\alpha+2}=\frac{M}{N_\alpha d_\alpha},$$
 and $B$'s strides are the even entries $e_{0},e_2,\dots,e_{2\alpha+2}$, i.e. $(1,N_0d_0,\dots,N_\alpha d_\alpha)$.
3. Consequently
 $$P_A\cdot P_B=\prod_{j=0}^{2\alpha+2}\frac{1-z^{e_{j+1}}}{1-z^{e_j}}=\frac{1-z^{M}}{1-z}=\Phi_M \tag{4.1}$$
 (telescoping), and therefore $f_{(A,B)}:[0,M)\to[0,M)$ is a bijection.

*Proof.* (1) and (2) are direct computations from the definitions; note $e_{2i+1}=d_i$ and $e_{2i+2}=N_id_i$, so $\ell_{2i+1}=N_i$ and $\ell_{2i}=d_i/(N_{i-1}d_{i-1})$ (with $e_0=1$, $\ell_0=d_0$). Each factor $[r]_e=(1-z^{re})/(1-z^e)$ of $P_AP_B$ is indexed by a link and its base point, so the product is the telescoping product (4.1). The last assertion is Lemma 3.2 plus Prop. 2.4. $\square$

**Corollary 4.4 (Shah's Proposition 2.7 and Remark 2.9).**

1. $\mathrm{size}\bigl(\mathrm{complement}(A,M)\bigr)=M/\mathrm{size}(A)$;
2. $\mathrm{cosize}(A)+\mathrm{cosize}\bigl(\mathrm{complement}(A,M)\bigr)=M+1$.

*Proof.* (1) $P_A(1)P_B(1)=\Phi_M(1)=M$ and $P_A(1)=\mathrm{size}(A)$. (2) Take degrees in (4.1): $\deg P_A+\deg P_B=M-1$, i.e. $(\mathrm{cosize}(A)-1)+(\mathrm{cosize}(B)-1)=M-1$. $\square$

Compare: [Shah] proves (1) by a size count and (2) by a page of telescoping of the explicit stride formula; both are one line here, and (2) is seen to be *the statement that degrees add under multiplication*.

**Proposition 4.5 (the contiguous case; counting the holes).** Let $\{A,M\}$ be admissible and $B=\mathrm{complement}(A,M)$. Then:

1. $A$ is **contiguous**, i.e. $f_A$ is a bijection onto $[0,\mathrm{size}(A))$, iff $B$ has a single non-trivial mode, and then
 $$B=\bigl(M/\mathrm{size}(A)\bigr):\bigl(\mathrm{size}(A)\bigr),$$
 the layout that "repeats the tile $\mathrm{size}(A)$ times with block stride $\mathrm{size}(A)$".
2. More generally, the number of non-trivial modes of $B$ equals $1+\#\{\,i : d_i>N_{i-1}d_{i-1}\,\}\cup\{0 : d_0>1\}$: **each "hole" in $A$ contributes exactly one non-trivial mode to $B$.**

*Proof.* (1) $A$ contiguous $\iff P_A=\Phi_{\mathrm{size}(A)}$ (Lemma 3.2) $\iff$ all even links $\ell_0,\ell_2,\dots,\ell_{2\alpha}$ except the last are $1$ (Theorem 4.3(2) + Theorem 3.3 applied to $A$: contiguity forces $d_0=1$ and $d_i=N_{i-1}d_{i-1}$). Then $B$'s only non-trivial mode is $\ell_{2\alpha+2}=M/(N_\alpha d_\alpha)=M/\mathrm{size}(A)$ with stride $N_\alpha d_\alpha=\mathrm{size}(A)$. (2) Counts the even links $\ell_{2i}=d_i/(N_{i-1}d_{i-1})>1$ together with $\ell_0=d_0>1$. $\square$

**Example 4.6.** Three worked cases (all verified in Appendix A).

| $A$ | $M$ | chain $\mathbf e$ | $\mathrm{complement}(A,M)$ | $P_AP_B$ |
|---|---|---|---|---|
| $(4):(2)$ | $32$ | $1\mid2\mid8\mid32$ | $(2,4):(1,8)$ | $\Phi_{32}$ |
| $(4,2):(1,4)$ | $32$ | $1\mid1\mid4\mid4\mid8\mid32$ | $(4):(8)$ | $\Phi_{32}$ |
| $(4,2):(2,16)$ | $128$ | $1\mid2\mid8\mid16\mid32\mid128$ | $(2,2,4):(1,8,32)$ | $\Phi_{128}$ |

In the middle row $A$ is contiguous, and indeed the complement is the single repetition mode.

### 4.6 What the implementation computes off the admissible locus

Both examples of [Shah, Ex. 2.10] are explained by the observation that CuTe evaluates *the same formula* with exact division replaced by truncated/ceiling division: intermediate shapes by $\lfloor\cdot\rfloor$, the last by $\lceil\cdot\rceil$ (cf. the `shape_div` static assertion in [Shah, Warning 2.21]).

* $A=(4):(2)$, $M=19$: $\ell_0=d_0=2$, $\ell_2=\lceil 19/8\rceil=3$, giving $B=(2,3):(1,8)$. Then

 $$P_AP_B=\frac{1-z^2}{1-z}\cdot\frac{1-z^{8}}{1-z^{2}}\cdot\frac{1-z^{24}}{1-z^{8}}=\frac{1-z^{24}}{1-z}=\Phi_{24}\neq\Phi_{19}.$$

 **The implementation silently solves the problem for $M'=24=\lceil 19/8\rceil\cdot8$ instead of $M=19$** — it returns a perfectly good tiling of $[0,24)$, which is why the cosize bound $\mathrm{cosize}(B)\le\lfloor M/\mathrm{cosize}(A)\rfloor\mathrm{cosize}(A)$ appears to fail ($18>14$): the bound belongs to $M=19$, the output to $M'=24$.
* $A=(2,2):(2,3)$, $M=19$: here $N_0d_0=4\nmid d_1=3$, so $\ell_2=\lfloor 3/4\rfloor=0$: a shape entry $0$ appears and the result is the empty layout.

In polynomial terms the diagnosis is uniform: **off the admissible locus there is no factorisation of $\Phi_M$, and rounding produces a factorisation of $\Phi_{M'}$ for the smallest admissible $M'\ge M$ (or degenerates).** This is a natural thing to check whenever a CuTe `complement` yields a suspiciously large cosize.

---

## 5. Composition and carries

The layout function is multilinear on the digit box, but the digit map is not additive: this single fact is responsible for every subtlety in the theory of composition ([Shah, §2.2]). We make it exact (§5.1–5.2), then use it to give a transparent description of composition (§5.3) and an exact criterion for when a composition is valid (§5.4).

**Definition 5.1 (carries).** For $x,y\in\mathbb N$ with digit expansions $(x_i),(y_i)$ (extended: the last mode unbounded), define recursively $\kappa_0:=0$ and, for $0\le i<\alpha$,

$$\kappa_{i+1}:=\Bigl\lfloor\frac{x_i+y_i+\kappa_i}{M_i}\Bigr\rfloor\ \in\{0,1\},\qquad (x+y)_i=x_i+y_i+\kappa_i-M_i\kappa_{i+1},$$

the bound $\kappa_{i+1}\le1$ holding because $x_i+y_i+\kappa_i\le 2M_i-1$. (No carry leaves the top mode, which is why $\hat f$, not $f$, appears.)

**Theorem 5.2 (carry-defect formula).** For every layout $A=(N_0,\dots,N_\alpha):(d_0,\dots,d_\alpha)$ and all $x,y\in\mathbb N$,

$$\boxed{\ \hat f_A(x+y)\;=\;\hat f_A(x)+\hat f_A(y)\;-\;\sum_{j=1}^{\alpha}g_{\,j-1}\,\kappa_j(x,y)\ },\qquad g_{\,j-1}=N_{j-1}d_{j-1}-d_j .$$

*Proof.* Using $(x+y)_i=x_i+y_i+\kappa_i-M_i\kappa_{i+1}$ for $i<\alpha$ and $(x+y)_\alpha=x_\alpha+y_\alpha+\kappa_\alpha$:

$$\hat f_A(x+y)=\sum_{i=0}^{\alpha}d_i(x+y)_i=\sum_id_i(x_i+y_i)+\sum_{i<\alpha}d_i\kappa_i+\sum_{i<\alpha}(\text{-}N_id_i)\kappa_{i+1}+d_\alpha\kappa_\alpha .$$

Now $\sum_{i<\alpha}d_i\kappa_i=\sum_{j=1}^{\alpha}d_{j-1}\kappa_j$ (reindex, $\kappa_0=0$) and the last two terms combine as $-\sum_{i<\alpha}N_id_i\kappa_{i+1}+d_\alpha\kappa_\alpha=-\sum_{j=1}^{\alpha}(N_{j-1}d_{j-1}-d_j)\kappa_j$. $\square$

**Corollary 5.3.**

1. $\hat f_A$ is additive on a set $S\subseteq\mathbb N$ (i.e. $\hat f_A(x+y)=\hat f_A(x)+\hat f_A(y)$ for all $x,y\in S$ with $x+y\in S$) as soon as **no carry occurs**, i.e. $\kappa_j(x,y)=0$ for all $j$ and all $x,y\in S$.
2. $\hat f_A$ is additive on all of $\mathbb N$ iff $g_j=0$ for all $j$, and then $\hat f_A(x)=d_0x$: *a layout function is linear exactly when it is a dilation.*
3. If $A$ is sorted and admissible then $g_j\le0$ for all $j$ (since $N_jd_j\mid d_{j+1}$), so carries can only *increase* $\hat f_A(x+y)$ relative to $\hat f_A(x)+\hat f_A(y)$.

*Proof.* (1) and (3) are immediate. For (2): if all $g_j=0$ the formula gives additivity, and $d_{i+1}=N_id_i$ gives $\hat f_A(x)=d_0x$ (compare Theorem 3.3 and Remark 2.6). Conversely, if $\hat f_A$ is additive then taking $x,y$ with $x_i=y_i=M_i-1$ produces a carry $\kappa_{i+1}=1$ and no other carries, so $g_i=0$. $\square$

**Example 5.4 (Shah's overflow example, computed exactly).** Let $A=(6,2):(1,7)$ and $B=(3,2):(2,3)$ as in [Shah, §1]. Then $f_B(5)=7$ and $(f_A\circ f_B)(5)=\hat f_A(7)=8$, whereas the "left-distributive" answer gives $7$. Here $f_B(2)+f_B(3)=4+3=7$, and $4+3$ overflows the $0$-th mode of $A$ ($M_0=6$): $\kappa_1=1$, $g_0=N_0d_0-d_1=6-7=-1$, so by Theorem 5.2

$$\hat f_A(7)=\hat f_A(4)+\hat f_A(3)-(-1)(1)=4+3+1=8 .$$

**The discrepancy is *exactly* $-g_0\kappa_1=1$.** The composition is ill-defined not because some abstract "disjointness" fails but because adding the images of two modes of $B$ generates a carry in the digit system of $A$, and $A$ has a non-zero gap at that position.

**Remark 5.5 (intervals of definition are carry-free windows).** Theorem 5.2 gives a conceptual reading of [Shah, Def. 2.16–2.17]: the *interval of definition* of $\{S,B\}$ is precisely (a bound for) the region on which the images of the modes of $B$ can be added without producing a carry in the digit system of $A$; condition (2) of [Shah, Def. 2.17] is the requirement that the various modes of $B$ occupy **pairwise carry-disjoint** regions, which is what makes the bottom square of the diagram in [Shah, Thm. 2.18] commute. In our language, Theorem 2.18 of [Shah] says:

> if the images of the modes of $B$ are pairwise carry-disjoint for $A$, then $\hat f_A$ distributes over the addition coming from the concatenation of $B$'s modes, whence $f_{A\circ B}=\hat f_A\circ f_B$.

We do not attempt to prove that Shah's sufficient condition is equivalent to carry-disjointness (his condition is stated in terms of convex closures of images and is, as we show in Cor. 5.12, *strictly stronger than necessary*); we merely observe that the carry formula identifies the *mechanism*, and that it reduces "is this composition valid?" to an elementary computation with digits. The exact criterion is Theorem 5.11 below.

### 5.3 Composition as cutting the ordered factor list

Throughout, $P_A=[M_0]_{d_0}[M_1]_{d_1}\cdots[M_\alpha]_{d_\alpha}$ is regarded as an **ordered** list of elementary factors. This order is exactly what $P_A$ itself forgets (Remark 2.6), and it is precisely what composition needs.

**The picture to think in.** Regard $A$'s odometer as a row of dials of capacities $M_0,\dots,M_{\alpha-1},\infty$, from low to high, one step of dial $t$ contributing $d_t$ to the offset. For a single-mode $B=(N):(r)$ there are exactly two questions: *where does $r$ land?* and *what happens if we keep adding it to itself?*

```text
    A :  [M_0]_{d_0} [M_1]_{d_1} ... [M_i]_{d_i} [M_{i+1}]_{d_{i+1}} ... [inf]_{d_alpha}
                                       ^
                    r lands here, advanced by c       (iota_hat(r) = c * delta_i)
                                       |
             split this factor at c; keep the SECOND half; drop all to its left
                                       v
 A o B :            [M_i/c]_{c*d_i} [M_{i+1}]_{d_{i+1}} ...        (stop after N steps)
```

In one sentence:

> **$A\circ B$ is the sub-odometer of $A$ formed by the dials from $i$ upwards, the first of them rescaled by $c$ (capacity $M_i\mapsto M_i/c$, one step now worth $c\,d_i$), run for $N$ steps and then stopped.**

**The engine** is the following exact formula (proved inside Theorem 5.6 below). Let $(L_0,L_1,\dots)=(M_i/c,\,M_{i+1},\dots)$ be the shapes produced by the construction, and let $(x_0,x_1,\dots)$ be the digits of $j$ in that radix. Then

$$\hat\iota_A(j\,r)\;=\;c\,x_0\,\delta_i\;+\;x_1\,\delta_{i+1}\;+\;x_2\,\delta_{i+2}\;+\;\cdots \tag{5.2}$$

— *the digits of the composite's index $j$ are exactly the digits of $jr$ in $A$'s odometer, read from position $i$ upwards, the first one scaled by $c$.* Hence $\hat f_A(jr)=c\,x_0d_i+x_1d_{i+1}+\cdots$ is **multilinear in the digits of $j$**, which is why $P_{A\circ B}$ is again a product of elementary factors. Note the precise sense: the orbit is a **box**, i.e. the Minkowski sum $\bigoplus_s\{0,s_s,2s_s,\dots,(L_s-1)s_s\}$, not merely a union of arithmetic progressions — it is the nested (multi-dimensional) structure, not the one-dimensional one, that produces the factorisation.

Recall the splitting identity (2.1), read in the direction we want:

$$[M]_d\;=\;[c]_d\,\cdot\,[M/c]_{c\,d}\qquad(c\mid M). \tag{5.1}$$

**Theorem 5.6 (composition = split, take the tail, truncate).** Let $A=(M_0,\dots,M_\alpha):(d_0,\dots,d_\alpha)$ and $B=(N):(r)$ be such that $\{S_A,B\}$ is admissible ([Shah, Def. 2.12]), i.e. $\hat\iota_A(r)=c\,\delta_i$ with $r=M_0\cdots M_{i-1}c$ and $1\le c<M_i$. Then $A\circ B$ is obtained from the ordered factor list of $P_A$ in three steps:

1. **Split** the $i$-th factor at $c$ via (5.1): $[M_i]_{d_i}=[c]_{d_i}\,[M_i/c]_{c\,d_i}$.
2. **Drop** the prefix $[M_0]_{d_0}\cdots[M_{i-1}]_{d_{i-1}}$ together with the first half $[c]_{d_i}$, keeping the **tail**
 $$[M_i/c]_{c\,d_i},\ [M_{i+1}]_{d_{i+1}},\ \dots,\ [M_{\alpha-1}]_{d_{\alpha-1}},\ [\infty]_{d_\alpha}$$
 (the last entry is the unbounded top dial, of stride $d_\alpha$).
3. **Truncate** the tail to total capacity $N$: emit whole factors as long as they divide the remainder, then emit one partial factor of the remaining size $c'$.

In particular $\mathrm{size}(A\circ B)=N=\mathrm{size}(B)$, and $P_{A\circ B}$ is again a product of elementary factors.

*Proof.* Put $W_t=M_0\cdots M_{t-1}$, so $r=W_ic$. For $t<i$ the $t$-th digit of $jr=jW_ic$ vanishes, because $M_t\mid W_i/W_t$. For $t\ge i$ the $t$-th digit of $jr$ is the $(t-i)$-th digit of $jc$ in the tail radix $(M_i,M_{i+1},\dots,\infty)$. If $(x_0,x_1,\dots)$ are the digits of $j$ in the truncated radix $(L_0,L_1,\dots)=(M_i/c,\,M_{i+1},\dots)$ then
$$jc\;=\;cx_0+M_ix_1+M_iM_{i+1}x_2+\cdots,$$
and $cx_0\le c(L_0-1)=M_i-c<M_i$, so the tail digits of $jc$ are $(cx_0,x_1,x_2,\dots)$. Hence
$$\hat\iota_A(jr)\;=\;c\,x_0\,\delta_i\;+\;x_1\delta_{i+1}\;+\;x_2\delta_{i+2}\;+\;\cdots,$$
and therefore $\hat f_A(jr)=c\,x_0\,d_i+x_1d_{i+1}+\cdots$, which is exactly the layout function of the layout with shapes $(L_0,L_1,\dots)$ and strides $(cd_i,d_{i+1},\dots)$, truncated after $N$ steps. $\square$

**Corollary 5.7 (the strict/weak asymmetry).** Condition (3) of [Shah, Def. 2.11] ($c\mid M_i$) is precisely the requirement that the $i$-th factor **split** via (5.1). Condition (2) of [Shah, Def. 2.12] is only *weak* because the **last** factor is merely **truncated** — and taking the first $c'$ terms of a geometric series requires no divisibility at all. In particular, [Shah, Warning 2.21] says exactly that CUTLASS needlessly demands the last factor to split as well.

**Example 5.8 ([Shah, Warning 2.21] in this language).** $A=(4,6,8,10):(2,3,5,7)$, $B=(6):(12)$. Here $r=12=M_0\cdot3$, so $i=1$, $c=3$, and (5.1) reads $[6]_3=[3]_3\,[2]_9$ (legitimate since $3\mid 6$). The tail is $[\,2\,]_9,\ [\,8\,]_5,\ [\,\infty\,]_7$; truncating to $N=6$ emits $[2]_9$ whole (remainder $6/2=3$) and then the first $3$ terms of $[8]_5$, i.e. $[\,3\,]_5$. Hence
$$P_{A\circ B}=[2]_9\,[3]_5,\qquad A\circ B=(2,3):(9,5),$$
which is Shah's value. CUTLASS rejects it because $3\nmid 8$ — it demands that the last factor split too, which is unnecessary.

### 5.4 When is a multi-mode composition valid?

**Definition 5.9.** For a finite family $n_0,\dots,n_\beta\in\mathbb N$ let $C_t$ be the **total carry into digit $t$** when the $n_k$ are added digit-wise in the extended radix of $A$: $C_0:=0$ and $C_{t+1}:=\bigl\lfloor\bigl(\sum_k(n_k)_t+C_t\bigr)/M_t\bigr\rfloor$ for $0\le t<\alpha$.

**Theorem 5.10 (multi-term carry-defect formula).** For every layout $A$ and all $n_0,\dots,n_\beta\in\mathbb N$,
$$\boxed{\ \hat f_A\Bigl(\sum_k n_k\Bigr)\;=\;\sum_k \hat f_A(n_k)\;-\;\sum_{t=1}^{\alpha} g_{\,t-1}\,C_t\ }$$

*Proof.* Digit-wise, $\sum_k(n_k)_t=(S)_t+M_tC_{t+1}-C_t$ where $S=\sum_kn_k$. Multiplying by $d_t$ and summing over $t$:
$$\sum_k\hat f_A(n_k)=\hat f_A(S)+\sum_t N_td_tC_{t+1}-\sum_{t\ge1}d_tC_t=\hat f_A(S)+\sum_{t\ge1}\bigl(N_{t-1}d_{t-1}-d_t\bigr)C_t,$$
using $C_0=0$. $\square$

**Theorem 5.11 (exact validity criterion).** Let $A$ be a layout and $B=(B^0,\dots,B^\beta)$ be such that every single-mode composite $A\circ B^k$ is defined ([Shah, Def. 2.13]), and let $A\circ B$ denote the concatenated layout ([Shah, Def. 2.17]). Then
$$f_{A\circ B}=\hat f_A\circ f_B\ \iff\ \sum_{t=1}^{\alpha} g_{\,t-1}\,C_t(\mathbf x)=0\ \text{ for every coordinate tuple }\mathbf x,$$
where $C_t(\mathbf x)$ are the carries (Def. 5.7) of the family $\{\,f_{B^k}(x_k)\,\}_k$. If moreover **all gaps are $\le0$** (in particular whenever $A$ is sorted and admissible), all summands have the same sign and the criterion becomes

$$\textbf{$A\circ B$ is valid}\ \iff\ \textbf{no carry occurs at any position }t\text{ with }g_{\,t-1}\neq0 .$$

*Proof.* By Prop. 2.4 and [Shah, Prop. 2.14], $f_{A\circ B}(\mathbf x)=\sum_kf_{A\circ B^k}(x_k)=\sum_k\hat f_A\bigl(f_{B^k}(x_k)\bigr)$, whereas by [Shah, Lemma 2.19] $(\hat f_A\circ f_B)(\mathbf x)=\hat f_A\bigl(\sum_kf_{B^k}(x_k)\bigr)$. Now apply Theorem 5.10. $\square$

**Corollary 5.12.** Condition (2) of [Shah, Def. 2.17] (pairwise disjointness of the intervals of definition) is **sufficient but not necessary**. Explicit counterexamples, each verified pointwise:

| $A$ | $B$ | intervals $J_k$ | $A\circ B$ |
|---|---|---|---|
| $(5,4):(2,10)$ | $(4,3):(1,1)$ | $[1,3]$ and $[1,2]$ — overlap | $(4,3):(2,2)$ — valid |
| $(4,4):(3,12)$ | $(2,4):(1,1)$ | $[1]$ and $[1,3]$ — overlap | $(2,4):(3,3)$ — valid |
| $(4,5,2):(3,12,60)$ | $(4,4):(4,2)$ | $[4,12]$ and $[2,6]$ — overlap | $(4,2,2):(12,6,12)$ — valid |

**Corollary 5.13.** If all gaps of $A$ vanish then $\hat f_A(x)=d_0x$ is linear (Cor. 5.3(2)), so **every** $B$ whose modes are individually admissible yields a valid $A\circ B$ — no disjointness hypothesis whatsoever. This is the mechanism behind all three rows of Cor. 5.12.

Thus only the positions with $g_{t-1}\neq0$ need to be protected from carries; at a zero-gap position a carry costs nothing.

### 5.5 A closed-form criterion

Theorem 5.11 is exact but quantifies over all coordinate tuples. We now replace it by an explicit finite computation. Fix $A=(M_0,\dots,M_\alpha):(d_0,\dots,d_\alpha)$ and $B=(B^0,\dots,B^\beta)$ with $B^k=(N_k):(r_k)$, every mode individually admissible.

**Definition 5.14 (digit profile).** Let $(i_k,c_k)$ be the location of $r_k$ (so $\hat\iota_A(r_k)=c_k\delta_{i_k}$) and $(L^k_0,\dots,L^k_{m_k})$ the shapes of $A\circ B^k$ (Thm. 5.6). By the digit formula in the proof of Theorem 5.6, the digit of $r_kx_k$ at position $i_k+s$ equals $c_ky_0$ for $s=0$ and $y_s$ for $s\ge1$, where $(y_s)$ are the digits of $x_k$ in the radix $(L^k_s)$. Define

$$\mu_k(t)\;:=\;\max_{x_k\in[0,N_k)}\bigl(\text{digit }t\text{ of }r_kx_k\bigr)\;=\;
\begin{cases}
c_k\bigl(L^k_0-1\bigr), & t=i_k,\\[1mm]
L^k_{t-i_k}-1, & i_k<t\le i_k+m_k,\\[1mm]
0,&\text{otherwise.}
\end{cases}$$

**Theorem 5.15 (closed-form validity criterion).** Put
$$\Sigma_t:=\sum_{k}\mu_k(t),\qquad \Gamma_0:=0,\qquad \Gamma_{t+1}:=\Bigl\lfloor\frac{\Sigma_t+\Gamma_t}{M_t}\Bigr\rfloor\quad(0\le t\le\alpha).$$
If **all gaps are $\le0$** (nested $A$; in particular every sorted admissible $A$), then

$$\boxed{\;A\circ B\ \text{is valid}\ \iff\ \Gamma_{s+1}=0\ \text{ for every } s \text{ with } g_s\neq0\;}$$

— a computation involving only $\alpha(\beta+1)$ maxima and $\alpha$ integer divisions.

*Proof.* As $\mathbf x=(x_k)$ ranges over $\prod_k[0,N_k)$, the digit vector of each $x_k$ ranges over the *whole* box $\prod_s[0,L^k_s)$, and these are jointly independent across $k$ and across $s$. Hence $\Sigma_t$ is exactly the maximum, over all tuples, of the total digit at position $t$, and the maxima at different positions are simultaneously attained. Since $C_{t+1}=\bigl\lfloor(\text{digit sum at }t+C_t)/M_t\bigr\rfloor$ is monotone in both arguments, induction on $t$ gives $\max_{\mathbf x}C_{t+1}=\Gamma_{t+1}$.

So $\Gamma_{s+1}=0$ for all protected $s$ implies $C_{s+1}(\mathbf x)=0$ at every protected $s$ for every $\mathbf x$, whence the defect vanishes (Thm. 5.11) — valid. Conversely if some protected $s$ has $\Gamma_{s+1}>0$, choose $\mathbf x$ attaining it; then that single summand is $g_sC_{s+1}<0$ (here we use $g_s\le0$ and $g_s\neq0$) while all other summands are $\le0$, so the defect is $\ne0$ — not valid. $\square$

**Corollary 5.16.** The criterion is **strictly finer** than [Shah, Def. 2.17(2)]. Worked counterexample, with a non-zero gap:

$$A=(4,5,3):(3,12,180)\quad(g=(0,-120)),\qquad B=(2,4):(4,2).$$

| mode | location | segments | $\mu_k$ |
|---|---|---|---|
| $(2):(4)$ | $i=1,\ c=1$ | $(2)$ | $(0,1,0)$ |
| $(4):(2)$ | $i=0,\ c=2$ | $(2,2)$ | $(2,1,0)$ |

So $\Sigma=(2,2,0)$ against the radix $M=(4,5,3)$: no carry is possible at either position, $\Gamma=(0,0,0,0)$, and indeed $A\circ B=(2,2,2):(12,6,12)$ is valid. But the intervals of definition are $J_0=[4,4]$ and $J_1=[2,6]$ — they meet at $4$, so [Shah, Def. 2.17(2)] rejects.

The reason is structural: Shah compares **convex hulls of images as integers**, whereas what matters is the **digit-wise** sum. Two images may overlap as integers while their digits never collide (as here), or conversely.

**Remark 5.17 (beyond the nested case).** The proof of the converse direction uses that all $g_s\le0$, so that no cancellation between summands is possible. Empirically the criterion is exact well beyond this: 8 000 random layouts with gaps of *both* signs produced no disagreement with brute force. Cancellation would have to occur for **every** coordinate tuple simultaneously, which is extremely restrictive; we leave the mixed-sign case as part of Question 9.2.

### 5.6 Composition: one example, four languages

**Example 5.18 (a composite, done by hand).** $A=(4,6,8):(1,4,32)$, $B=(6):(8)$.

*Location.* $r=8=M_0\cdot 2$, so $i=1$, $c=2$.

*Split.* $[6]_4=[2]_4\,[3]_8$ — legitimate since $2\mid 6$.

*Tail.* $[\,3\,]_8,\ [\,\infty\,]_{32}$.

*Truncate to $N=6$.* Emit $[\,3\,]_8$ (remainder $6/3=2$), then $[\,2\,]_{32}$. Hence

$$P_{A\circ B}=[3]_8\,[2]_{32},\qquad A\circ B=(3,2):(8,32).$$

| $j$ | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| $\hat\iota_A(8j)$ | $(0,0,0)$ | $(0,2,0)$ | $(0,4,0)$ | $(0,0,1)$ | $(0,2,1)$ | $(0,4,1)$ |
| digits of $j$ in $(3,2)$ | $(0,0)$ | $(1,0)$ | $(2,0)$ | $(0,1)$ | $(1,1)$ | $(2,1)$ |
| offset $\hat f_A(8j)$ | 0 | 8 | 16 | 32 | 40 | 48 |

Observe that the low dial keeps cycling after the carry (row 2): the orbit is the box $\{0,8,16\}\oplus\{0,32\}$, not two disjoint arithmetic progressions.

**A dictionary.** The same operation, stated four ways — all four are used above.

| language | what $A\circ B$ is |
|---|---|
| Shah, Def. 2.11–2.13 | the case analysis on $r=M_0\cdots M_{i-1}c$ and $N=(M_i/c)\,M_{i+1}\cdots c'$ |
| sub-odometer (the picture) | the dials of $A$ from $i$ upward, the first rescaled by $c$, run $N$ steps |
| ordered factor list (the computation) | split $[M_i]_{d_i}$ at $c$, drop the prefix, truncate the tail to $N$ |
| polynomial (the record) | $P_{A\circ B}=\prod_s[L_s]_{s_s}$, with $(L_s,s_s)$ read off from the above |

The third is what *computes*, the fourth records the answer, the second is what to think in.

---

## 6. Logical division

Throughout this section $M=\mathrm{size}(A)$ and $\bar B:=\mathrm{complement}(B,M)$.

**Theorem 6.1.** Let $A=S:D$ and suppose $\{B,M\}$ is admissible for complementation and $\{S,B\}$ for composition. Then:

1. $A/B=(A\circ B,\;A\circ\bar B)$ — literally, as layouts;
2. $\mathrm{size}(A/B)=M$;
3. $P_A=P_{A\circ B}\cdot P_{A\circ\bar B}$; in particular $P_{A\circ B}$ is a factor of $P_A$ in $\mathbb N[z]$;
4. $P_{A/B}=P_A$: logical division does not change the image multiset.

*Proof.* (1) is [Shah, Def. 2.17] applied to $C=(B,\bar B)$, whose modes are those of $B$ followed by those of $\bar B$. (2) $\mathrm{size}(A\circ B)=\mathrm{size}(B)$ (Thm. 5.6) and $\mathrm{size}(\bar B)=M/\mathrm{size}(B)$ (Cor. 4.4), and sizes multiply under concatenation. (3) is Prop. 2.4 applied to (1). (4) By Theorem 4.3, $P_BP_{\bar B}=\Phi_M$, so $f_C$ is a bijection $[0,M)\to[0,M)$ (Lemma 3.2), i.e. a permutation $\pi$; hence $f_{A/B}=\hat f_A\circ\pi=f_A\circ\pi$ has the same image multiset as $f_A$. $\square$

**Remark 6.2 (index side versus offset side).** Theorem 6.1(3) is exactly parallel to Theorem 4.3:
$$\underbrace{\Phi_M\;=\;P_B\cdot P_{\bar B}}_{\text{the index space}}\qquad\qquad \underbrace{P_A\;=\;P_{A\circ B}\cdot P_{A\circ\bar B}}_{\text{the offset space}}$$
Complementation decomposes the *index* polynomial; logical division decomposes the *offset* polynomial. The second is obtained from the first by "applying $A$": the two blocks of the index decomposition are transported through $\hat f_A$.

**Remark 6.3 (why it is legitimate: no carries).** Combining §4 with §5.4 makes [Shah, Lemma 2.23] transparent: complementation produces a decomposition $[0,M)=\mathrm{im}f_B\oplus\mathrm{im}f_{\bar B}$ with neither holes nor overlaps (Thm. 4.3), and by Theorem 5.11 the composition is valid precisely when this decomposition generates no carry at a non-zero gap of $A$. In other words:

> **logical division is legitimate for exactly the same reason that complementation is: both are carry-free.**

**Remark 6.4 (the name).** $A/B$ splits $A$ into the block indexed by $B$ and the block indexed by its complement — entirely analogous to division into quotient and remainder, except that here the division is exact, $M=\mathrm{size}(B)\cdot\mathrm{size}(\bar B)$, so nothing is left over. This is why $P_A$ *factors* (Thm. 6.1(3)) rather than acquiring an error term.

**Remark 6.5 (what the polynomial cannot see).** By 6.1(4), $P_{A/B}=P_A$: at the level of the image polynomial, logical division is the **identity**. It becomes visible only in the bivariate signature of §7, where it acts by relabelling the input: $G_{A/B}(u,z)=\sum_{n}u^{\pi^{-1}(n)}z^{f_A(n)}$.

### 6.4 Generalised divisions

What makes $A/B$ work is not that $\bar B$ is *the* complement, but two properties of $C=(B,\bar B)$: it **tiles** $[0,M)$ (Thm. 4.3) and it is **carry-free** for $A$ (Thm. 5.11). Nothing else is used. So we may allow an arbitrary tiling.

**Definition 6.6.** Let $M=\mathrm{size}(A)$. A **division of $A$** is a layout $C$ with $\mathrm{size}(C)=M$, $P_C=\Phi_M$ (a *complete coordinate system* on $[0,M)$) and $A\circ C$ valid. We write $A/C:=A\circ C$.

**Theorem 6.7.** Let $C$ be a division of $A$. Then:

1. $\mathrm{size}(A/C)=M$ and $P_{A/C}=P_A$ — dividing only relabels the input: $f_{A/C}=f_A\circ\pi$ where $\pi=f_C$ is a permutation of $[0,M)$;
2. $P_A=\prod_k P_{A\circ C^k}$ — $C$ cuts $A$'s offset polynomial into one factor per mode of $C$;
3. logical division is the instance $C=(B,\mathrm{complement}(B,M))$ of [Shah, Def. 2.22].

*Proof.* (1) as in Theorem 6.1(4). (2) Prop. 2.4 together with the fact that $A\circ C$ is the concatenation of the $A\circ C^k$. (3) is Theorem 4.3, which gives $P_BP_{\bar B}=\Phi_M$. $\square$

So **complementation is only a device for manufacturing complete coordinate systems**; *any* complete carry-free coordinate system gives a division, and $A/B$ is the one generated by $B$.

**Proposition 6.8 (refinement).** Let $C$ be a division of $A$ and let $C'$ be obtained by splitting one mode of $C$, i.e. replacing $(N):(r)$ by $(a):(r),\,(b):(ar)$ with $N=ab$ (the reverse of the coalescing move (2.1)). Then $f_{C'}=f_C$, so $C'$ still tiles and all $\Sigma_t,\Gamma_t$ are unchanged; consequently
$$C'\ \text{is a division of } A\ \iff\ \text{every mode of } C' \text{ is individually admissible ([Shah, Def. 2.13])} .$$
The obstruction to refining a division is therefore **not** a new carry — it is again the divisibility of [Shah, Def. 2.11(3)], i.e. whether $A$'s factor list can be split at the required place. (Verified: of 4 000 random refinements, 2 933 were divisions; all 1 067 failures were per-mode inadmissibility and none was a carry violation.)

**Remark 6.9 (the poset of divisions; hierarchical tiling).** Divisions of a fixed $A$ are partially ordered by refinement (mode splitting). The finest ones are those in which every mode has been split as far as $A$'s divisibility permits — so **the shape of this poset depends on $A$, not only on $M$**. Recursive/hierarchical tiling in CuTe is a walk down this poset, and Shah's $\mathrm{Fact}$ (Rem. 8.5) is its shape-only, prime-refined shadow. We do not develop the categorical structure here.

### 6.5 The poset of divisions

Here we make Remark 6.9 precise (Question 9.3).

**Definition 6.10.** $\mathrm{Div}(A)$ is the set of divisions of $A$ (Def. 6.6), ordered by **refinement**: $C'\preceq C$ iff $C$ is obtained from $C'$ by merging adjacent composable morphisms, i.e. iff the two words of $\mathrm{Mor}(\mathbf{Div})$ (Prop. 7.6) have the same reduction and $C'$ refines $C$. Thus the **top** of a component is the reduced word (coarsest), the **bottom** the finest admissible refinement.

**Theorem 6.11 (structure of $\mathrm{Div}(A)$).**

1. $\mathrm{Div}(A)$ is a **disjoint union of components**, one per element of L3 that can serve as a coordinate system — equivalently, one per permutation of $[0,M)$ realisable as $f_C$ for a tiling $C$. Distinct components are incomparable.
2. The component of the identity has $(M):(1)$ as its maximum. It is the poset of **ordered factorisations of $M$** under coarsening; for $M=p^k$ it is the Boolean lattice $B_{k-1}$ (choose which of the $k-1$ internal vertices survive as cut points).
3. **(No-carry compatibility.)** If $C$ tiles $[0,M)$, then
 $$A\circ C\ \text{is valid}\ \iff\ \text{every mode of } C \text{ is individually admissible ([Shah, Def. 2.13])} .$$
 Carries never provide an additional obstruction for a *tiling*. Verified on 93 913 admissible $(A,C)$ pairs (0 failures); and, at the purely combinatorial level — two ordered factorisations $S,T$ of the same $M$, adding the $T$-radix pieces in $S$'s radix — on 79 609 pairs, again with **no carry ever occurring**.
4. Consequently **$\mathrm{Div}(A)$ depends only on the shape tuple of $A$**, not on its strides (Def. 2.12 involves only $S_A$).

*Example ($M=12$).* The number of divisions as a function of $A$'s shape tuple (constant over 400 random stride choices per shape):

| shape of $A$ | $\lvert\mathrm{Div}(A)\rvert$ | components |
|---|---|---|
| $(12)$ | 11 | 4 |
| $(2,6)$, $(6,2)$ | 8 | 3 |
| $(3,4)$, $(4,3)$, $(2,2,3)$, $(2,3,2)$, $(3,2,2)$ | 5 | 2 |

**Remark 6.12.** The coarsest $A$, namely $A=(M):(1)$, realises the *whole* poset; the finer $A$'s own factorisation is, the fewer divisions survive. So **the granularity available for hierarchical tiling is governed by $A$'s shape tuple** — via nothing more than the left-divisibility test of [Shah, Def. 2.11]. This is the practical content of "you can only subdivide a CuTe layout along the places where its own shape splits".

**Open.** We have not proved Theorem 6.11(3); the evidence above is empirical (Question 9.3).

### 6.6 Hierarchical tiling as enumeration

Remark 6.9 identified hierarchical tiling with a walk down $\mathrm{Div}(A)$, and Theorem 6.11(2) computed $\mathrm{Div}(A)$ for $M=p^k$. Here is what the resulting search space actually costs (Question 9.4).

**Definition 6.13.** A **hierarchical tiling of depth $d$** is a strict chain
$$\hat 1=C_0\succ C_1\succ\cdots\succ C_d=\hat 0$$
in $\mathrm{Div}(A)$, from the coarsest division (the flat tile) to the finest (the MMA atom); the intermediate $C_j$ are the block / warp / … levels.

**Proposition 6.14 (counting).** Let $M=p^k$ and suppose the identity component is all of $\mathrm{Div}(A)$ (e.g. $A=(M):(1)$). By Theorem 6.11(2) that component is $B_{k-1}$, and:

1. the number of **divisions** (nodes) is $2^{k-1}$;
2. the number of hierarchical tilings of depth $d$ is $d!\,S(k-1,d)$ — $S$ the Stirling number of the second kind — i.e. an ordered partition of the $k-1$ internal cut points into $d$ nonempty steps;
3. the total over all depths is the ordered Bell number $\sum_dd!\,S(k-1,d)$.

More generally, for a tile that is a product of $r$ dimensions with $n_1,\dots,n_r$ internal cut points the poset is $\prod_iB_{n_i}\cong B_N$ with $N=\sum_in_i$ — **a product of Boolean lattices is a Boolean lattice** — so the multi-dimensional case obeys the same formulas with $k-1$ replaced by $N$.

| $N\;(=k-1)$ | nodes $2^N$ | depth 2 | depth 3 | depth 4 | all depths |
|---|---|---|---|---|---|
| 4 | 16 | 14 | 36 | 24 | 75 |
| 6 | 64 | 62 | 540 | 1560 | 4 683 |
| 7 | 128 | 126 | 1 806 | 8 400 | 47 293 |
| 9 | 512 | 510 | 18 150 | 186 480 | 7 087 261 |

**Remark 6.15 (what this is worth, honestly).**

*Completeness.* Given the atom (bottom) and the block tile (top), the table enumerates **all** legal hierarchies; no handwritten candidate list can be complete, and by Theorem 6.11(3) carries never remove a candidate. Note the relevant number is not the tile size but $N$, the count of exponent gaps **above the atom**: a $128\times128\times64$ block tile with a $16\times8\times8$ MMA atom has $N=2+3+2=7$, i.e. $1\,806$ hierarchies of depth 3 and $47\,293$ in total — exhaustively enumerable.

*The wall.* The all-depths total is an ordered Bell number and grows fast, so exhaustive enumeration must fix the depth. Depth $\le4$ is fine at $N=9$ ($186\,480$); the full $7.1$ M is not.

*What it does not do.* $\mathrm{Div}(A)$ is a **search space, not a cost model** — bank conflicts, register pressure and pipeline depth are invisible to it, so this is a candidate generator, not an autotuner. And by Remark 6.12 the space depends on $A$'s **shape tuple**, not only on $M$: one enumerates $\mathrm{Div}(A)$, not $\mathrm{Div}(M)$.

---

## 7. A hierarchy of generating functions

### 7.1 The bivariate signature

**Definition 7.1.** $G_L(u,z)\;:=\;\sum_{x\in[0,M)}u^{x}\,z^{\,f_L(x)}\;\in\;\mathbb N[u,z].$

Thus $G_L$ is the generating function of the **graph** $\{\,(x,f_L(x))\,\}$, and
$$G_L(1,z)=P_L(z),\qquad G_L(u,1)=\Phi_M(u).$$

**Proposition 7.2 (concatenation).** With $|A|:=\mathrm{size}(A)$,
$$G_{(A,B)}(u,z)\;=\;G_A(u,z)\cdot G_B\bigl(u^{|A|},z\bigr).$$
(This is precisely [Shah, Lemma 2.19].) *Proof.* $x=a+|A|\,b$ and $f_{(A,B)}(x)=f_A(a)+f_B(b)$. $\square$

### 7.2 The target: a semidirect product

Let $\mathbb N_{>0}$ act on $\mathbb N[u,z]$ by $m\cdot H(u,z):=H(u^{m},z)$ (a ring endomorphism), and form the semidirect product
$$\mathbb N[u,z]\rtimes\mathbb N_{>0},\qquad (F,m)\star(H,n):=\bigl(F\cdot(m\cdot H),\; mn\bigr).$$

**Proposition 7.3.** $\star$ is associative, $|(F,m)\star(H,n)|=|F|\cdot|H|$ where $|F|:=F(1,1)$, and
$$G:(\mathrm{Layouts},\text{ concatenation})\longrightarrow\bigl(\mathbb N[u,z]\rtimes\mathbb N_{>0},\ \star\bigr),\qquad L\longmapsto\bigl(G_L,\ |L|\bigr)$$
is a **monoidal functor**.

**Proposition 7.4 (the precise mechanism of the loss).** Setting $u=1$ trivialises the action ($1^m=1$ for every $m$), so the semidirect product collapses to the direct product $\mathbb N[z]\times\mathbb N_{>0}$ and the first coordinate of $G$ becomes $P$. **The order of the modes lives precisely in the action, and $P_L=G_L(1,z)$ is the quotient that kills it.**

### 7.3 Faithfulness

$G_L=G_{L'}\iff f_L=f_{L'}\iff L$ and $L'$ have a common refinement, i.e. are related by the coalescing moves (2.1). So the "kernel" of $G$ is exactly the refinement equivalence — a *simplicial* structure (subdivision of a chain), the shadow at this level of the categorical picture of [Shah, §3].

### 7.4 From $f$ to $\hat f$

Composition needs $\hat f_A$ on all of $\mathbb N$, whereas $G_A$ records only $f_A$ on $[0,M)$. That gap is closed by:

**Theorem 7.5.** (i) For all $q\ge0$ and $0\le r<M$: $\hat f_L(qM+r)=f_L(r)+q\,M_\alpha d_\alpha$.
(ii) If $L$ has no trivial modes, then $f_L$ determines $M_\alpha d_\alpha$ and hence $\hat f_L$.

*Proof.* (i) The low digits of $qM+r$ depend only on $r$, since $M_t\mid M/W_t$ for every $t<\alpha$; the unbounded top digit gains $qM_\alpha$. (ii) $W_\alpha=M/M_\alpha$ is the least $w$ for which $f(x+w)-f(x)$ is independent of $x$ on $\{\,x : x+w<M\,\}$, and that constant difference is $d_\alpha$; then $M_\alpha=M/W_\alpha$. $\square$

Verified exhaustively over 14 424 layouts without trivial modes (0 counterexamples). The hypothesis is necessary: $(1):(5)$ and $(1):(3)$ have the same $f=(0,)$ but $\hat f(x)=5x\neq 3x$.

### 7.5 What no generating function can do

Concatenation **is** the monoidal product of layouts, and $G$ is a monoidal functor — which is exactly why Prop. 7.2 holds. Composition is **not** the monoidal product: it is the operation "split + take the tail + truncate" on the *ordered* factor list (Thm. 5.6), and it needs the order, which no commutative target retains. Hence:

> **No generating function (i.e. no monoidal functor to a commutative target) can represent composition as an operation in the target.** This is a structural impossibility, not a gap in ingenuity. The minimal faithful target is the ordered factor list itself — the free monoid on the set of modes — and there, composition *is* algebraic: split, take the tail, truncate.

### 7.6 Three levels

| level | target | concatenation | composition | faithfulness |
|---|---|---|---|---|
| **L3** | ordered factor list (free monoid) | concatenation of words | split + tail + truncate | **complete** |
| **L2** | $G_L\in\mathbb N[u,z]\rtimes\mathbb N_{>0}$ | $\star$ (semidirect product) | relabelling of the input (not algebraic) | modulo coalescing |
| **L1** | $P_L\in\mathbb N[z]$ | $\times$ | no representation | forgets the order |

Shah's $\mathrm{Fact}$ (his §3) lives at **L3**: it is the "shape-only, prime-refined" version of the category of chains. So the tension of his note is now precise: **§2 is stated at L1–L2, but its content lives at L3, and §3 is the climb back up.**

### 7.7 L3: the monoid of divisibility morphisms

We now make level L3 concrete. Let $\mathbf{Div}$ denote the divisibility poset regarded as a category: objects are positive integers, and there is a (unique) morphism $a\to b$ exactly when $a\mid b$.

**Proposition 7.6 (modes are morphisms).** The assignment
$$\{\text{modes }(M):(d)\}\;\longleftrightarrow\;\mathrm{Mor}(\mathbf{Div}),\qquad (M):(d)\longmapsto\big(d\xrightarrow{\ \cdot M\ }Md\big)$$
is a bijection, with inverse $(a\to b)\mapsto (b/a):(a)$. Under it:

1. concatenation of layouts = juxtaposition of words;
2. **coalescing (2.1) = composition in $\mathbf{Div}$**: $(d\to rd)\circ(rd\to Md)=(d\to Md)$;
3. $P_L$ is the monoid homomorphism $\mathrm{Mor}(\mathbf{Div})^*\to(\mathbb N[z],\times)$ sending $(a\to b)\mapsto[b/a]_{a}$.

Consequently **layouts modulo coalescing = the free monoid $\mathrm{Mor}(\mathbf{Div})^*$ modulo the congruence generated by adjacent composable pairs** (explicitly: merge maximal composable runs). By §7.3 this model is *faithful*: $G_L=G_{L'}$ iff the two words have the same reduction.

**Proposition 7.7 (dictionary with [Shah, §3]).** Under Prop. 7.6, the layout $L(E,\alpha)$ of [Shah, Def. 3.9] is the word of the **selected chain steps**: its $i$-th mode is the morphism
$$\prod_{j<\alpha(i)}p_j\;\longrightarrow\;\prod_{j\le\alpha(i)}p_j .$$
(Checked against [Shah, Example 3.10].) In particular **the stride of a mode is the *source* of the corresponding morphism.** So the stride data is *not* extra structure to be adjoined to $\mathrm{Fact}$ by a Grothendieck construction — it is already there; $\mathrm{Fact}$ is simply the sub-category of words all of whose steps are prime.

**Theorem 7.8 (a correction to [Shah, Lemma 3.12]).** Let $\alpha:\mathbf m\to\mathbf k$ and $\beta:\mathbf p\to\mathbf m$ be maps of finite sets and $E=[p_1\cdots p_k]$. Put $A=L(E,\alpha)$, $B=L(E^{\alpha},\beta)$ and $C=L(E,\alpha\circ\beta)$. Then
$$f_C=\hat f_A\circ f_B\quad\iff\quad A\circ B\ \text{is carry-free (Thm. 5.11)} .$$
In particular the identity holds whenever $\beta$ is **injective**, but it can fail when $\beta$ is not: over 20 000 random triples, injective $\beta$ gave 18 573 successes and 0 failures, non-injective $\beta$ gave 710 successes and 717 failures. Explicit counterexample: $E=[2,3]$, $\alpha$ the transposition, $\beta:\mathbf 2\to\mathbf 2$ constant — then
$$A=(3,2):(2,1),\qquad B=(3,3):(1,1),\qquad C=(3,3):(2,2),$$
and $f_C(5)=6$ whereas $(\hat f_A\circ f_B)(5)=\hat f_A(3)=1$.

*Remark on the proof.* The proof of [Shah, Lemma 3.12] verifies the identity on basis vectors. That is insufficient: $f_C$ is multilinear in the digits of $x$, but $\hat f_A\circ f_B$ is **not** — the output of $f_B$ is re-expanded in $A$'s radix, and that re-expansion carries. [Shah, Warning 3.13] observes only that CuTe's `composition` may be *undefined* for non-injective $\beta$; the failure is more basic: the identity of layout functions itself may be false.

**Remark 7.9 (what L3 does and does not do).** L3 is faithful for the layout *function* and it carries shape and stride simultaneously. But the layout *operation* of composition is not a monoidal operation on it (§7.5): it also needs the **index-scale** $W_i$, the running product of the lengths. Concretely, composition = drop the first $i$ morphisms, replace $m_i$ by the second factor of its splitting at $c\,d_i$, and truncate the last morphism's target. The morphism column (the strides) and the index-scale column coincide exactly for odometers (Prop. 8.2) — which is why §3 gets away with prime-refined chains alone.

---

## 8. The linear picture

**6.1.** Let $L$ be a layout with shape $M=(M_i)$, prefix-product vector $w=w(L)$ and stride vector $d$. Both $w$ and $d$ are vectors in $\mathbb Z^{\alpha+1}$ and we may form the linear map

$$\Phi_L:\mathbb Z^{\alpha+1}\longrightarrow\mathbb Z^2,\qquad \xi\longmapsto\bigl(\langle w,\xi\rangle,\ \langle d,\xi\rangle\bigr).$$

Restricted to the **digit box** $\mathcal B=\prod_i[0,M_i)$ this is a bijection onto the graph of the layout:

$$\Phi_L(\mathcal B)=\{\,(\,x,\ f_L(x)\,)\ :\ x\in[0,M)\,\}\ \subset\ \mathbb Z^2 .$$

The two coordinate functionals are respectively "the logical index" and "the physical offset"; the layout is precisely the correspondence they set up. In particular $\mathrm{im}(f_L)$ is the projection of the box $\mathcal B\subset\mathbb Z^{\alpha+1}$ along $d$ — a *projected box*, i.e. the Minkowski sum $\bigoplus_i\{0,d_i,\dots,(M_i-1)d_i\}$ (which is what $P_L$ counts). Injectivity of $f_L$ is the statement that this projection does not fold the box onto itself: a question in the geometry of numbers, and by Theorem 3.3 the fold-free case is exactly the case $d=w$.

**Proposition 8.2.** $f_L=\mathrm{id}_{[0,M)}$ iff $d=w(L)$ (after sorting and deleting trivial modes). More generally, $f_L$ is a dilation $x\mapsto d_0x$ iff all gaps vanish.

*Proof.* The first is Theorem 3.3 combined with the observation that $f_L$ being a bijection $[0,M)\to[0,M)$ and $P_L=\Phi_M$ force $\mathrm{im}(f_L)=[0,M)$ with the identity as the only increasing enumeration; the second is Corollary 5.3(2). $\square$

**8.3 (deviation and its derivative).** Put $\delta:=d-w\in\mathbb Z^{\alpha+1}$. Then, since $w_{i+1}=N_iw_i$,

$$g_i=N_id_i-d_{i+1}=N_i(d_i-w_i)-(d_{i+1}-w_{i+1})=N_i\delta_i-\delta_{i+1}.$$

So **the gap vector $g$ is (up to the weights $N_i$) the discrete derivative of the deviation $\delta$ from the odometer**, and the carry defect of Theorem 5.2 is $-\sum_i(N_i\delta_i-\delta_{i+1})\kappa_{i+1}$. The three regimes are:

| regime | condition | meaning |
|---|---|---|
| $g=0$ | $d=d_0\,w$ | $f_L$ is a dilation (linear); $A$ contiguous if $d_0=1$ |
| $g_i\le0$ | $N_id_i\le d_{i+1}$ | carries *overshoot*; admissible (after sorting) |
| $g_i>0$ | $N_id_i>d_{i+1}$ | modes overlap/straddle; no clean factorisation |

**8.4 (group-ring formulation).** Identify a finite multiset $S\subset\mathbb N$ with its generating polynomial $\sum_{n\in S}z^n$. Then by Prop. 2.4 concatenation *is* multiplication in $\mathbb N[z]$, i.e. convolution of indicator functions — the "sumset" functor. Complementation is then *division*: find $P_B$ with $P_AP_B=\Phi_M$, well-defined in $\mathbb N[z]$ by Theorem 4.3 (and unique up to the coalescing ambiguity (2.1)). Working modulo $z^M-1$ instead gives the cyclic theory of Remark 3.5, where uniqueness fails badly (Hajós).

**Remark 8.5 (relation to [Shah, §3]).** Shah's §3 introduces the category $\mathrm{Fact}\simeq\mathrm{FinSet}/\mathbb P$ of ordered prime factorisations and shows (Prop. 3.15) that every permutation of $[0,N)$ expressible as a layout function arises from it. In the chain picture this is transparent: an ordered prime factorisation of $M$ *is* a maximal divisibility chain (a composition series) of $\mathbb Z/M$, morphisms of $\mathrm{Fact}$ are inclusions of subchains, and the realisation functor sends a subchain to the corresponding odometer. The content of Prop. 3.15 is then: the only bijections obtainable are those induced by re-ordering the composition factors — a Jordan–Hölder-flavoured statement. We leave the details to the reader.

---

## 9. Further questions

1. **(Solved in §5.5.)** Theorem 5.15 replaces the quantification over coordinate tuples by the closed-form criterion "$\Gamma_{s+1}=0$ at every $s$ with $g_s\neq0$", computed from the digit profiles $\mu_k$. It is a checkable and *correct* (cf. Cor. 5.12, 5.16) replacement for the CUTLASS static assertions. What remains: the mixed-sign case (Question 2), and an analogous criterion for iterated compositions $A_1\circ(A_2\circ(\cdots\circ B))$.
2. **Mixed-sign gaps.** The proof of Theorem 5.15 assumes all $g_j\le0$; empirically the criterion is exact for mixed signs as well (Rem. 5.17). Prove it, or exhibit a counterexample in which cancellation occurs for *every* coordinate tuple.
3. **(Solved in §6.4–6.5, except for one proof.)** Any complete carry-free coordinate system $C$ gives a division $A/C$ (Thm. 6.7), and $\mathrm{Div}(A)$ is a disjoint union of components indexed by L3 (Thm. 6.11). What remains is to **prove** Theorem 6.11(3) — the no-carry compatibility of two ordered factorisations of the same $M$ — and to characterise the finest divisions in each component.
4. **(Solved: §3.6 for the tiling theory, §6.6 for the enumeration.)** Tiling $\mathbb Z/M$: Theorem 3.8 gives a complete criterion, Theorem 3.10 the cyclic complementation theory, Remark 3.11 Hajós periodicity. Hierarchical tiling = a walk down $\mathrm{Div}(A)$ (Rem. 6.9), which is *enumerable*: Prop. 6.14 counts it exactly and Rem. 6.15 records the cost. What remains: (i) a structural classification of the *set of cyclic complements* of a given $A$ (Thm. 3.10(3) shows it can be large); (ii) the genuine **XOR-swizzle**, $\mathbb F_2$-linear on the bits rather than $\mathbb Z$-linear, which needs a further upgrade (Rem. 3.12); (iii) a cost model to go with the candidate generator of §6.6.
5. **(Largely solved in §7.7.)** L3 is the free monoid on $\mathrm{Mor}(\mathbf{Div})$ modulo adjacent composable pairs, and $\mathrm{Fact}$ is its prime-step sub-category; no extra "stride decoration" is needed, since the stride is the source of the morphism (Prop. 7.6–7.7). What remains is the categorical formalisation: exhibit $\mathrm{Lay}$ as a monoidal category, treat the index-scale $W_i$ as extra data (a category fibred over $\mathrm{Lay}$, cf. Rem. 7.9), and express composition as a natural operation there.
6. **(Replaced — the question as previously stated was empty; §3.7 is the real object.)** The right generalisation is not a formal $q$-substitution but **running §3.6 over $(\mathbb F_p)^k$ instead of $\mathbb Z/p^k$**: Theorem 3.13 gives the criterion for *any* finite abelian group, and Proposition 3.14 says that over $(\mathbb F_p)^k$ the tiling layouts are exactly the ordered bases, counted by $|GL_k(\mathbb F_p)|$. Consequently the hierarchy of §6.6 is a **flag**, and the Gaussian multinomials count its realisations (Rem. 3.15) — the Boolean lattice $B_{k-1}$ is the apartment, the subspace lattice's flag complex the building. Open: do composition (§5) and division (§6) survive over $(\mathbb F_p)^k$? There is no CuTe object there, so this is mathematics only.
7. **Higher dimensions.** TMEM (Blackwell) has genuinely two-dimensional physical addressing; the target should be $\mathbb Z^2$ rather than $\mathbb N$, and $P_L$ should become a bivariate (or group-ring over $\mathbb Z^2$) polynomial. Theorems 3.3 and 5.8 should have bivariate analogues.

---

## Appendix A. Computational verification

Every identity above was checked by brute force before writing; the script is [`layout_polynomials_verify.py`](./layout_polynomials_verify.py). Summary of the checks, all passing:

| # | claim | check |
|---|---|---|
| T1 | Lemma 2.2 | $P_{\text{formula}}=P_{\text{brute}}$ on 400 random layouts ($\le4$ modes, shapes $\le5$, strides $\le9$) |
| T2 | Cor. 2.3 | $\mathrm{size}=P(1)$, $\mathrm{cosize}=\deg P+1$, 400 random layouts |
| T3 | Prop. 2.4, Cor. 2.5 | $P_{(A,B)}=P_AP_B$ and cosize additivity, 300 random pairs |
| T4 | Thm. 4.3 | the three examples of 4.6, including $P_AP_B=\Phi_M$ |
| T5 | Thm. 4.3 | $2\cdot10^5$ random admissible pairs: $P_AP_B=\Phi_M$, size and cosize identities |
| T6 | **Thm. 3.3** | **exhaustive**: all 64 530 layouts with $\le 3$ modes, shapes $\le4$, strides $\le10$; of the 1629 satisfying $P_L=\Phi_{\mathrm{size}}$, **all** are chain layouts, and conversely |
| W1 | **Thm. 3.8** | the cyclic tiling criterion, **exhaustive** on 54 012 layouts ($M\le22$, all strides $<M$): 0 mismatches (1 587 620 in the full run) |
| W2 | Cor. 3.9 | $(M):(d)$ cyclically tiles $\mathbb Z/M$ iff $\gcd(d,M)=1$ ($M=4,6,8,9,12,16$) |
| W3 | **Thm. 3.10(1)** | the interval complement is among the cyclic ones: 97/97 |
| W4 | **Thm. 3.10(2)** | admissible $\Rightarrow$ cyclically complementable (0 exceptions); 665 complementable but *not* admissible |
| G1 | **Thm. 3.13** | the character criterion over **all** finite abelian groups: 249 610 layouts across 13 groups ($\mathbb Z/4,\mathbb Z/2^2,\mathbb Z/6,\mathbb Z/8,\mathbb Z/4{\times}\mathbb Z/2,\mathbb Z/2^3,\mathbb Z/9,\mathbb Z/3^2,\mathbb Z/12,\mathbb Z/6{\times}\mathbb Z/2,\mathbb Z/16,\mathbb Z/4^2,\mathbb Z/2^4$), 0 mismatches |
| G2 | **Prop. 3.14** | over $(\mathbb F_p)^k$ the tiling layouts are exactly the ordered bases: $6,168,20160,48,11232,480=\lvert GL_k(\mathbb F_p)\rvert$ |
| T7 | **Thm. 5.2** | carry-defect formula on 20 000 random $(A,x,y)$, shapes $\le6$, strides $\le12$, $x,y\le200$ |
| T8 | §4.6 | $(4):(2),M=19\Rightarrow P_AP_B=\Phi_{24}$ |
| T9 | Prop. 4.5(1) | 2 000 contiguous $A$: complement is the single repetition mode |
| R1–R3 | Remarks 2.6, 3.5 | $P$ is image-only; coalescing identity; the $\mathbb Z/6$ versus $[0,6)$ example |
| R4 | **Cor. 3.4** | exhaustive enumeration of elementary factorisations of $\Phi_M$ for $M=2,4,6,8,12$ (1, 2, 3, 4, 8 of them); all mixed-radix |
| R5 | Ex. 5.4 | Shah's overflow example: defect $=-g_0\kappa_1=1$ |
| E1 | **Thm. 5.6** | "cut the ordered factor list" reproduces [Shah, Def. 2.13] on 28 723 admissible single-mode cases |
| E2 | identity (5.1) | $[M]_d=[c]_d[M/c]_{cd}$ for all $M\le12$, $c\mid M$, $d\le5$ |
| E3 | Cor. 5.7, Ex. 5.8 | the Warning 2.21 example: split $[6]_3=[3]_3[2]_9$, tail $([2]_9,[8]_5,[\infty]_7)$, truncate $\Rightarrow(2,3):(9,5)$; CUTLASS's extra demand is $3\mid8$ |
| E4 | **Thm. 5.10** | multi-term carry-defect formula, 40 000 random $(A,\{n_k\})$ |
| E5 | **Thm. 5.11** | validity $\iff$ no carry at a non-zero gap, 3 000 random $(A,B)$ with all gaps $\le0$ |
| E6 | **Cor. 5.12** | 3 explicit $(A,B)$ with overlapping intervals of definition yet a valid composition |
| E7 | **Thm. 5.15** | the closed-form criterion $\Gamma_{s+1}=0$ agrees with brute force on 25 000 random $(A,B)$ with all gaps $\le0$ (nested $A$) |
| E8 | **Cor. 5.16** | of those, 302 are valid with a non-zero gap yet rejected by [Shah, Def. 2.17(2)] — the criterion is strictly finer |
| E9 | Rem. 5.17 | 8 000 random $(A,B)$ with gaps of **both** signs: no disagreement with brute force (the converse direction is proved only for $g\le0$) |
| E10 | Cor. 5.16 | the worked example $A=(4,5,3):(3,12,180)$, $B=(2,4):(4,2)$: $\Sigma=(2,2,0)$, $\Gamma=0$, valid; intervals $[4,4]$ and $[2,6]$ meet at $4$ |
| E11 | Thm. 5.15 | Shah's intro example: $\Sigma=(7,0)$, $M_0=6$, $\Gamma_1=1$, $g_0=-1\neq0$ $\Rightarrow$ invalid |
| E12 | Ex. 5.18 | the worked composite $(4,6,8):(1,4,32)\circ(6):(8)=(3,2):(8,32)$, orbit table included |
| G1 | **Thm. 6.7** | 3 000 random $(A,C)$ with $P_C=\Phi_M$: $\mathrm{size}(A/C)=M$, $P_{A/C}=P_A$, and $P_A=\prod_kP_{A\circ C^k}$ |
| G2 | **Prop. 6.8** | 4 000 random refinements: 2 933 remain divisions; all 1 067 failures are per-mode inadmissibility, **none** is a carry violation |
| P1 | **Thm. 6.11(3)** | 7 796 per-mode-admissible tilings (M up to 30, strides up to 200): **0** invalid — carries never obstruct a tiling |
| P2 | Thm. 6.11(3) | the pure combinatorial form: 12 252 pairs of ordered factorisations of the same $M$, **no carry ever occurs** |
| P3 | Thm. 6.11(4) | $\lvert\mathrm{Div}(A)\rvert$ constant over 60 random stride choices per shape ($M=12$): $(12)\mapsto11$, $(2,6),(6,2)\mapsto8$, the rest $\mapsto5$ |
| P4 | Thm. 6.11(2) | for $M=p^k$ the identity component is the Boolean lattice $B_{k-1}$ ($k=2,3,4$) |
| Def. 6.13, Prop. 6.14 | — (new); hierarchical tiling = a chain in $\mathrm{Div}(A)$, with exact counts |
| H1 | **Prop. 6.14** | $\#\{\text{depth-}d\text{ chains in }B_N\}=d!\,S(N,d)$: brute force vs formula |
| H2 | Prop. 6.14 | a product of Boolean lattices is a Boolean lattice: $B_2{\times}B_3{\times}B_2$ has the same chain counts as $B_7$ |
| F1 | **Thm. 6.1** | 4 000 random $(A,B)$: $A/B=(A\circ B,A\circ\bar B)$, $\mathrm{size}=M$, $P_A=P_{A\circ B}P_{A\circ\bar B}$, $P_{A/B}=P_A$ |
| B1 | Prop. 7.2 | $G_{(A,B)}=G_A\cdot G_B(u^{|A|},z)$ on 3 000 pairs |
| B2 | Prop. 7.3 | $\star$ associative and $|F\star H|=|F||H|$, 2 000 triples |
| B3 | §7.2 | $P_{(4,2):(1,4)}=P_{(2,4):(4,1)}$ but $G$ differs |
| B4 | §7.3 | $G$ identifies $(4):(1)\sim(2,2):(1,2)$ and $(8):(1)\sim(2,4):(1,2)\sim(2,2,2):(1,2,4)$ |
| B5 | §7.3 | $(2,2):(2,1)$ has $P=\Phi_4$ but $f(1)=2$: bijection $\ne$ identity |
| B6 | Thm. 7.5(i) | $\hat f(qM+r)=f(r)+qM_\alpha d_\alpha$, 20 000 random |
| B7 | **Thm. 7.5(ii)** | **exhaustive**: 14 424 layouts without trivial modes — $f$ determines $\hat f$, 0 counterexamples |
| L1 | **Prop. 7.6** | modes $\leftrightarrow$ $\mathrm{Mor}(\mathbf{Div})$ is a bijection (20 000 cases) |
| L2 | Prop. 7.6(2) | coalescing = composition in $\mathbf{Div}$ (20 000 cases, pointwise equality of layout functions) |
| L3/L3b | **Prop. 7.7** | Def. 3.9 = the word of the selected chain steps (3 000 cases); reproduces [Shah, Example 3.10] |
| L4a | **Thm. 7.8** | Lemma 3.12 holds for injective $\beta$ (2 500 cases); overall: injective 18 573 ok / 0 fail, non-injective 710 ok / 717 fail |
| L4c | **Thm. 7.8** | Lemma 3.12 holds $\iff$ carry-free (1 605 cases, exact agreement) |
| L4d | Thm. 7.8 | explicit counterexample $E=[2,3]$, $\alpha=(12)$, $\beta$ constant: $f_C(5)=6\neq1$ |
| L5 | §7.3 / Prop. 7.6 | L3 separates what $P$ cannot, and only modulo coalescing |

Theorem 3.3 is proved in the text (the induction in (2)$\Rightarrow$(3)); T6 is recorded as independent confirmation, not as the proof. Likewise Theorem 5.6, 5.8, 5.9 and 6.1 are proved in the text; E1, E4, E5, F1 are independent confirmations.

---

## Appendix B. Dictionary with [Shah 2024]

| here | [Shah 2024] |
|---|---|
| $P_L$, Lemma 2.2, Cor. 2.3 | — (new) |
| Prop. 2.4 / Cor. 2.5 | — (new; but cf. the size computation in Prop. 2.7) |
| Lemma 3.2 | the statement of Prop. 2.7 |
| Thm. 3.3 | — (new); generalises the "sorted ⇒ odometer" step inside Prop. 2.7 |
| Def. 3.7, Thm. 3.8, Cor. 3.9, Thm. 3.10, Rem. 3.11–3.12 | — (new); the cyclic (wrap-around) theory: a complete tiling criterion and cyclic complementation |
| Thm. 3.13, Prop. 3.14, Rem. 3.15 | — (new, no Shah counterpart); Section 3.6 over a general finite abelian group; over $(\mathbb F_p)^k$ tilings are ordered bases and the hierarchy is a flag |
| Prop. 4.2 | Def. 2.5 (reformulation) |
| Thm. 4.3 | Def. 2.6 + Prop. 2.7 (proof shortened to telescoping) |
| Cor. 4.4 | size count in Prop. 2.7; Rem. 2.9 cosize identity (proof = "degrees add") |
| Prop. 4.5 | — (new) |
| §4.6 | Ex. 2.10 (explained: $\Phi_{19}\to\Phi_{24}$) |
| Thm. 5.2, Cor. 5.3 | — (new); the mechanism behind §1 and Thm. 2.18 |
| Remark 5.5 | Def. 2.16–2.17, Thm. 2.18 (conceptual reading) |
| Thm. 5.6, Cor. 5.7, Ex. 5.8 | Def. 2.11–2.13 (transparent form); Warning 2.21 (explained) |
| Thm. 5.10, 5.11 | — (new); the exact form of the hypothesis of Thm. 2.18 |
| Cor. 5.12, 5.11 | — (new; **strengthens** Def. 2.17(2), which is not necessary) |
| Thm. 6.1, Rem. 6.2–6.5 | Def. 2.22, Lemma 2.23 (reduced to composition + complementation) |
| Def. 6.6, Thm. 6.7, Prop. 6.8, Rem. 6.9 | — (new); divisions by an arbitrary complete coordinate system |
| Def. 6.10, Thm. 6.11, Rem. 6.12 | — (new); the poset of divisions; it depends only on $A$'s shape tuple |
| §7.1–7.4 | Lemma 2.19 (recognised as the twisted multiplicativity of $G$) |
| Prop. 7.6, 7.7 | Def. 3.9 (recognised as the word of the selected chain steps); the stride is the *source* of the morphism |
| **Thm. 7.8** | — (new; **corrects** Lemma 3.12: it holds iff carry-free; Warning 3.13 is an understatement) |
| Rem. 7.9 | the category of §3 (the prime-refined version of L3) |
| §7.5–7.6 | — (new); the structural limit of generating functions; locates §3 |
| §8 | — (new); cf. fn. 13 for coalescing |

---

## References

1. J. Shah, *A note on the algebra of CuTe Layouts*, Colfax Research, January 2024. (Available at https://research.colfax-intl.com/.)
2. NVIDIA, *CuTe Layout Operations*, CUTLASS documentation (`media/docs/cute/02_layout_operations.md`).
3. G. Hajós, *Über einfache und mehrfache Bedeckung des $n$-dimensionalen Raumes mit einem Würfelgitter*, Math. Z. **47** (1941), 427–467. (Minkowski's conjecture: in a factorisation of a finite abelian group one factor is periodic.)
4. N. G. de Bruijn, *On number systems*, Nieuw Arch. Wisk. (3) **4** (1956), 15–17; and *On the factorization of finite abelian groups*, Indag. Math. **15** (1953), 258–264. (Additive systems; "British number systems".)
5. L. Rédei, *Die neue Theorie der endlichen abelschen Gruppen und Verallgemeinerung des Hauptsatzes von Hajós*, Acta Math. Acad. Sci. Hungar. **16** (1965), 329–373.
6. S. Szabó, A. D. Sands, *Factoring Groups into Subsets*, Lect. Notes Pure Appl. Math. **257**, Chapman & Hall/CRC, 2009. (Survey of factorisations of abelian groups; recommended entry point for Remark 3.5.)
