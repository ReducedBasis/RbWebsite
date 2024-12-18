---
title: PBDW
weight: 4
---

Let us present one method that combines model order reduction and a data assimilation problem: the Parametrized-Background Data-Weak method (PBDW).
We aim to approximate the true physical state $u_{true}$ by $u^{bk}+\eta$ where $\eta$ is an update correction of unmodeled physics, or unanticipated and non-parametric uncertainty (constructed from several observations), and $u^{bk}$ is an approximation obtained from a best-knowledge model.

## Offline

- A POD or Greedy procedure on a model with bias to build a reduced space $\mathcal{Z}_N$
- Generation of observation reduced spaces $\mathcal{U}_M$ from true data

## Online 
A reduced problem to solve in order to find an approximation solution in the reduced-basis space $\mathcal{Z}_N \bigoplus \mathcal{U}_M$ based on projection-by-data.


## Codes

[Python Notebook on advection-diffusion model problem](/post/pbdwadv)

[Python Notebook on Helmholtz model problem](/post/pbdwhelmholtz)

[Code with Feel++](https://docs.feelpp.org/mor/0.110/pbdw.html)

## Details:

The PBDW formulation integrates a parameterized best-knowledge (bk) mathematical model $G^{bk,\mu}$ and $M$ experimental observations of the true field $u_{true}(\mu^{true})$ to improve the accuracy of its approximation, as well as any desired output $l_{out}(u_{true}(\mu^{true})) \in \mathbb{R}$ for a given output functional $l_{out}$.

We denote by $u^{bk}(\mu) \in \mathcal{U}$, the solution to the parameterized model for the parameter value $\mu \in \mathcal{P}^{bk}$, 
$$ G^{bk,\mu}(u^{bk}(\mu)) = 0.$$
Here $\mathcal{P}^{bk} \subset \mathbb{R}^P$ is a compact set that reflects the lack of knowledge in the value of the model parameters. We further define the bk solution manifold
$$ \mathcal{M}^{bk} = \{u^{bk}(\mu) : \mu \in \mathcal{P}^{bk}\}.$$

- We first compute a reduced basis of our bk model:
We introduce a sequence of background spaces that reflect our (prior) best knowledge (e.g. with a POD procedure)
$$\mathcal{Z}_1 \subset \dots \subset \mathcal{Z}_{N} \subset \mathcal{U}, $$ generated from different snapshots of the bk solution manifold.
Our goal is to choose the background spaces such that
$$\underset{N \to \infty}{\textrm{lim}}\ \underset{w \in \mathcal{Z}_N}{\textrm{inf}} \lVert u_{true}(\mu)-w \rVert \leq \varepsilon_{\mathcal{Z}}, \forall \mu \in \mathcal{P}^{bk}, $$
where $\varepsilon_{\mathcal{Z}}$ is a small tolerance value.
In other words, we want that our bk model represents our data observations well.

To sum up, we now have a bk model with a bias leading to the $\mathcal{Z}_n$ sequence.
Of course, we also want to improve our reduced approximation with the knowledge of the true observation measures, that can possibly be noisy.

- We thus define the functions that will describe our measures (we might have access to partial data observations only).
Given a parameter $\mu^{true}\in \mathcal{P}^{bk}$, we assume that our measures $y^{obs}(\mu^{true}) \in \mathbb{C}^M$ is of the form $$\forall m=1,\dots,M, y_m^{obs}=l^0_m(u_{true}(\mu^{true})).$$
Here $y_m^{obs}(\mu^{true})$ is the value of the $m$-th observation and $l_m^0 \in \mathcal{U}'$.
These observation functionals model the particular transducer used in data acquisition. For instance, if the transducer measures a local state value, we may model the transducer $l^0_m$ by a Gaussian convolution.
We then associate with each observation functional $l_m^0 \in \mathcal{U}'$ an observation function
$$\forall m=1,\dots,M, \ q_m = R_{\mathcal{U}} l^0_m,$$
which is the Riesz representation of the functional. It allows us to introduce hierarchical observable spaces,
$$ \forall M=1,\dots,M_{max}, \ \mathcal{U}_M= \mathrm{span} \{q_m\}_{m=1}^M. $$

Since $(u_{true}(\mu^{true}),q_m) = (u_{true}(\mu^{true}),R_{\mathcal{U}} l^0_m )=l^0_m(u_{true}(\mu^{true}))$,
it follows that, for any $q \in \mathcal{U}_M$, $(u_{true}(\mu^{true}),q)=(u_{true}(\mu^{true}),\sum_{m=1}^M \alpha_m q_m )= \sum_{m=1}^M \alpha_m l^0_m(u_{true}(\mu^{true}))$, hence $(u_{true}(\mu^{true}),q)$ is a weighted sum of experimental observations.

- We now have two kind of reduced bases: the reduced basis $(\Phi_i)_{i,\dots,N}$ that represents the spaces $\mathcal{Z}_N$ and the reduced basis $(q_i)_{i=1,\dots,M}$ that represents the observation spaces $\mathcal{U}_M$.

We can now proceed with the PBDW estimation statement:

given a parameter $\mu^{true} \in \mathcal{P}^{bk}$, find $(u^*_{N,M}(\mu^{true}) \in \mathcal{U},z^*_{N,M}(\mu^{true}) \in \mathcal{Z}_N, \eta^*_{N,M}(\mu^{true}) \in \mathcal{U}_M)$ such that

$$\begin{equation}(u^*_{N,M}(\mu^{true}) ,z^*_{N,M}(\mu^{true}) , \eta^*_{N,M}(\mu^{true})) =\underset{\eta_{M,N} \in \mathcal{U}}{\underset{z_{N,M}\in \mathcal{Z}_N}{\underset{u_{N,M} \in \mathcal{U}}{\textrm{arginf}}}} \ \lVert \eta_{N,M} \rVert^2
\end{equation}$$

subject to

$$ \begin{align}
&(u_{N,M},v)=(\eta_{N,M},v)+(z_{N,M},v), \ \forall v\in \mathcal{U},\\
& (u_{N,M},\Phi)=(u_{true}(\mu^{true}),\Phi), \  \forall \Phi \in \mathcal{U}_M.
\end{align}
$$

The first equation (2) describes that our reduced approximation $u_{N,M}$  is equal to the sum of the reduced coefficients $z_{N,M}$ with the reduced corrections $\eta_{N,M}$.
The second equation (3) just tells us that the measures must coincide with our approximation. Of course, the arginf (1) states that we want to have as small corrections as possible.

The Lagrangian of (1)-(2)-(3) writes $$\mathcal{L}(u^*,z^*,\eta^*,v^*,\Phi^*)=\frac12 \lVert \eta^* \rVert^2 + (u^* - \eta^* -z^*,v) + (u^*-u_{true}, \Phi), $$ where $v^* \in \mathcal{U}$ and $\Phi^* \in \mathcal{U}$ are the Lagrange multipliers. From its partial derivatives, we then obtain the associated reduced Euler-Lagrange equation as a saddle problem (see https://dspace.mit.edu/bitstream/handle/1721.1/97702/Patera_A%20parameterized.pdf?sequence=1&isAllowed=y eq (3) for details):

Find $(z^*_{N,M}(\mu^{true}) \in \mathcal{Z}_N, \eta^*_{N,M}(\mu^{true}) \in \mathcal{U}_M)$ such that

$$ \begin{align}
&(\eta^*_{N,M}(\mu^{true}),q)+(z^*_{N,M}(\mu^{true}),q)=(u_{true}(\mu^{true}),q), \ \forall q\in \mathcal{U}_M,\\
& (\eta^*_{N,M}(\mu^{true}),p)=0, \  \forall p \in \mathcal{Z}_N.
\end{align}
$$

and set $$ u^*_{N,M}(\mu^{true})=\eta^*_{N,M}(\mu^{true})+z^*_{N,M}(\mu^{true}).$$

We now state the algebraic form of the PBDW problem:
Since any element of $z \in \mathcal{Z}_N$ can be expressed as $z=\sum_{n=1}^N z_n \Phi_n$ and any element $\eta \in \mathcal{U}_M$ can be expressed as $\eta=\sum_{m=1}^M \eta_m q_m$, we can easily derive the algebraic PBDW system from (4)-(5):

- Offline part: After generating our reduced bases, we compute the matrices $A$ and $B$ such that $(A)_{i,j}=(q_i,q_j)$ and $(B)_{i,j}=(\Phi_j,q_i)$ and then the global matrix is given by $K= \begin{pmatrix} A & B \\
B^T & 0 \end{pmatrix}$.
- Online part: Then, for a new parameter $\mu^{true}$, we compute $\forall m=1,\dots,M, y_m^{obs}=l^0_m(u_{true}(\mu^{true})),$ and then we solve $\begin{pmatrix} \eta_M \\ z_N \end{pmatrix} = K^{-1} \begin{pmatrix} y^{obs} \\ 0 \end{pmatrix}$ and we approximate $u_{true}(\mu^{true})$ by $$u^*_{N,M}=\sum_{i=1}^M (\eta_{N,M})_i \ q_i +\sum_{i=1}^N (z_{N,M})_i \ \Phi_i. $$

Remark:
The PBDW framework may accommodate any observation functional that is consistent with the data-acquisition procedure. In the notebook code, we focus on localized observations. As noted, for localized observations using a given gaussian transducer, an adequat location of the centers $(x_m)_{m=1}^M$ is essential to determine the space $\mathcal{U}_M$. In fact, the apriori error between the true solution and the reduced approximation is related to the stability constant $\beta_{N,M}=\underset{w \in \mathcal{Z}_N}{\textrm{inf}} \underset{v \in \mathcal{U}_M}{\textrm{sup}} \frac{(w,v)}{\lVert w \rVert \lVert v\rVert}$ that we want to maximize. In practice, we may select the observation functionals (and more specifically the observation centers) using several different processes:
- we might employ a uniform or random coverage of the domain,
- we might use a Generalized Emperical Interpolation Method (GEIM),
- or we could use a Greedy stability maximization (called SGreedy) as in the paper "PBDW method for state estimation: error analysis for noisy data and nonlinear formulation" ( https://arxiv.org/abs/1906.00810 ) which choose the reduced spaces for the bk model and for the observation spaces simultaneously while maximizing the stability constant.


In the notebook codes, we use a POD algorithm to compute the bk reduced space and a uniform coarser mesh to select the sensors localization.


## References


- [Gong, H., Maday, Y., Mula, O., & Taddei, T. (2019). PBDW method for state estimation: error analysis for noisy data and nonlinear formulation. arXiv preprint arXiv:1906.00810.](https://arxiv.org/abs/1906.00810)

- [Maday, Y., Patera, A. T., Penn, J. D., & Yano, M. (2015). A parameterized‐background data‐weak approach to variational data assimilation: formulation, analysis, and application to acoustics. International Journal for Numerical Methods in Engineering, 102(5), 933-965.](https://dspace.mit.edu/bitstream/handle/1721.1/97702/Patera_A%20parameterized.pdf?sequence=1&isAllowed=y)

- [Hammond, J. K., Chakir, R., Bourquin, F., & Maday, Y. (2019). PBDW: A non-intrusive Reduced Basis Data Assimilation method and its application to an urban dispersion modeling framework. Applied Mathematical Modelling, 76, 1-25.](https://www.sciencedirect.com/science/article/pii/S0307904X19302951)