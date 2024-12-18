---
title: POD-Galerkin
weight: 2
---



The Galerkin-Proper Orthogonal Decomposition (Galerkin-POD) is one of the most popular model reduction techniques for nonlinear partial differential equations. It is based on a Galerkin-type approximation, where the POD basis functions contain information from a solution of the dynamical system at pre-specified instances, so-called snapshots.

## Offline

A POD procedure (in other words, SVD or PCA)

## Online 

A reduced problem with a Galerkin projection to solve

## Codes:
[Python jupyter notebook](/post/podgalerkin)

[Code with Feel++](/uploads/POD.cpp)

[FEniCS](https://www.rbnicsproject.org/index.html)

[FreeFem++](https://www.um.es/freefem/ff++/pmwiki.php?n=Main.POD)

## Details:

- #### A model problem

We start with a model problem to illustrate the POD-Galerkin method.

Let us introduce the stationary Navier-Stokes equation in the 2D lid driven cavity problem with non-homogeneous Dirichlet boundary conditions on the upper side, homogeneous Dirichlet (no-slip) boundary conditions on the remaining sides, where the domain $\Omega$ is the unit square.
The 2D steady Navier-stokes equation writes:

$$
\begin{align*}
&-\nu \Delta u + (u \cdot \nabla) u + \nabla p =0, \textrm{ in } \Omega,\\
& \nabla. u=0, \textrm{ in } \Omega,\\
& (u_1,u_2)=(1,0), \textrm{ on } \Omega_{up}:=\partial \Omega \cap \{y=1\},\\
& (u_1,u_2)=(0,0), \textrm{ on } \partial \Omega \backslash \Omega_{up},
\end{align*}
$$
where

$u=(u_1,u_2) \in V:=H^1_{d,0}(\Omega)^2=\{u \in~H^1(\Omega)^2,  u=~(0,0) \textrm{ on } \partial \Omega \backslash \Omega_{up}, \ \textrm{and } u=(1,0) \textrm{ on } \Omega_{up} \}$ represents the velocity of the incompressible fluid, $ p \in L^2(\Omega)$ its pressure, and $\nu=\frac{1}{Re}$ where $Re$ is the Reynolds parameter. Here, the Reynolds number is our parameter of interest ($\mu=Re$).

We impose the average of the pressure $\int_{\Omega} p$
 to be equal to $0$ to ensure its uniqueness.
 
The problem 2D lid driven cavity problem can be rewritten into its variational form:

$$
\begin{equation*}
\textrm{ Find }(u,p) \in V \times L^2_0(\Omega)),\textrm{ such that}
\end{equation*}
$$
$$
\begin{align}
  & a(u,v;\nu)+b(v,p)=0,\forall v \in H^1_0(\Omega)^2 \\
  &  b(u,q)=0,\forall q \in L^2_0(\Omega)
\end{align}
$$

where $$\begin{equation*}
    a(u,v;\nu)=(u \cdot \nabla u,v)+\nu(\nabla v,\nabla  u),  \textrm{ and } b(u,q)=-(\nabla \cdot u, q).
    \end{equation*}$$

We assume the problem is well posed and that it satisfies the so-called inf sup condition (or LBB).
    

With FEM, there exist several types of stable elements. A classical one is the Taylor-Hood element, where basis functions of degree $k$ are used for the pressure and basis functions of degree $k+1$ are employed for the velocities. Thus, we use Taylor-Hood $\mathbb{P}_2-\mathbb{P}_1$ elements to solve the problem in the python notebook. The velocity is approximated with $\mathbb{P}_2$ FE, whereas the pressure is approximated with $\mathbb{P}_1$ FE.


For the nonlinearity we adopt a fixed-point iteration scheme, and after multiplying by test functions $q$ and $v$ (resp. for pressure and velocity), which in variational form reads:

$$\begin{equation}
\nu (\nabla u^k, \nabla v) + ((u^{k-1} \cdot \nabla) u^k,v) -  (p^k, \nabla \cdot v) - (q, \nabla \cdot u^k) + 10^{-10} (p^k, q) =0, \textrm{in } \Omega,
\end{equation}$$

where $u^{k-1}$ is the previous step solution, and we iterate until a threshold is reached (until $\|u^{k}-u^{k-1}\| < \varepsilon $).

With the Taylor-Hood elements, we obtain the system $\mathbf{K} \mathbf{x} =\mathbf{f}$ to solve where $\mathbf{K}= \begin{pmatrix}
\mathbf{A} & -\mathbf{B}^T\\
-\mathbf{B} & 10^{-10} \mathbf{C}
\end{pmatrix}$, $\mathbf{x}$ stands for the tuple velocity-pressure $(u_1^k,u_2^k,p^k)$, and where the assembled matrix $\mathbf{A}$ corresponds to the bilinear part $ \nu (\nabla u^k, \nabla v) + ((u^{k-1} \cdot \nabla) u^k),v) $, the matrix $ B$ to the bilinear part $(  p^k ,\nabla \cdot v)$ and $\mathbf{C}$ is the mass matrix applied to the pressure variable ($(\cdot,\cdot)$ represents either the $L^2$ inner-product onto the velocity space or onto the pressure space). 


In order to use the POD-Galerkin method, in a general context, we need an affine decomposition with respects to the parameter $\nu$. 
 From the algorithm point of view, the parameter-independent terms are computed offline, making the online computation faster. If this assumption is not fulfilled, we resort to [EIM](/docs/eim).

- #### Proper Orthogonal Decomposition (POD) Galerkin

The POD has been applied to a wide range of applications (turbulence, image processing applications, analysis of signal, in data compression, optimal control,...).

We detail its offline part, and the online projection stage. There are several forms of POD (classical POD, Snapshots POD, spectral POD,...), and here we consider the Snapshots POD algorithm for the offline part. To sum up, the algorithm is as follows:

- In the offline part, the RB is built with several instances (called the snapshots) of the problem, for several well chosen parameter values. This step consists, first, in forming the snapshots correlation matrix and in retrieving its eigenvalues and its eigenvectors (as in an Singular Value Decomposition). Then, the reduced basis (RB) functions are constructed by linear combinations of the  first $N$ eigenvectors with the snapshots, after having sorted the eigenvalues in descending order. 
- The online part consists in solving the reduced problem which uses the RB and a Galerkin projection for a new parameter  $\nu \in \mathcal{G}$. At the end of the algorithm, a reduced solution for  $\nu$ is created.

##### OFFLINE STAGE 

For one parameterized problem, the POD approximation consists in determining a basis of proper orthogonal modes which represents the best the solutions. The modes are obtained by solving an eigenvalue problem.
To put it in a nutshell, the POD consists in extracting dominants modes from random data in order to be able to approximate a solution of the problem for a new parameter very quickly.
Thus, we seek a function (a mode) which is highly correlated in average with the realizations $\{u(X)\}$, where $X=(x,\nu_i)_{i=1,\dots,Ntrain} \in \Omega \times \mathcal{G}$, $Ntrain$ is the number of snapshots, $x=(x_1,x_2) \in \Omega$, and $\nu$ is the varying parameter in $\mathcal{G} \subset \mathbb{R}$ (which can also be in $\mathbb{R}^n$).
In other words, we should choose a function $\Phi$ which maximizes the averaged projection on the observations (the average is represented by $\overline{\cdot}$), suitably normalized in the sense of least squares, i.e. which maximizes the quantity $\overline{|(u,\psi)|^2}$. So we end up with the following constrained optimization problem:

Find $\Phi$ such that\\

$$\begin{equation}
\displaystyle \max_{\psi \in L^2(\Omega \times \mathcal{G})}
    \frac{\overline{|(u,\psi)|^2}}{\lVert{\psi}\rVert^2}=\frac{\overline{|(u,\Phi)|^2}}{\lVert{\Phi}\rVert^2},  
\end{equation}$$

with $\lVert{\Phi}\rVert^2=1$ (To simplify the notation, $\lVert{\cdot}\rVert_{L^2(\Omega \times \mathcal{G})}$ is written $\lVert{\cdot}\rVert$).
We can show that equation (4) is equivalent to an eigenvalue problem.

Indeed, to find the maximum, we use the Lagrangian of (4) $J[\phi]=\overline{|(u,\phi)|^2}-\lambda(\rVert{\phi}\rVert^2-1) $ and a variation $\psi$, such that:

$$\begin{equation}
    \frac{d}{d\delta}J[\phi+\delta \psi]_{|\delta=0}=0.
\end{equation}$$

$$
\begin{align*}
     \frac{d}{d\delta}J[\phi+\delta\psi]_{|\delta=0}&=\frac{d}{d\delta}[\overline{|(u,\phi+\delta\psi)|^2}-\lambda(\lVert{\phi+\delta\psi}\rVert^2-1]_{|\delta=0},\nonumber \\
     &=\frac{d}{d\delta}[\overline{(u,\phi+\delta\psi)(\phi+\delta\psi,u)}-\lambda(\phi+\delta\psi,\phi+\delta\psi)]_{|\delta=0},\nonumber \\
     &=\overline{(u,\phi)(\psi,u)+(u,\psi)(\phi,u)}-\lambda((\phi,\psi)+(\phi,\psi)),\nonumber \\
     &=2Re[\overline{(u,\psi)(\phi,u)}-\lambda(\phi,\psi)].
\end{align*}$$

Thus, permuting mean operations and integrations entails:

$$\begin{align*}
    \frac{d}{d\delta}J[\phi+\delta\psi]_{|\delta=0}&=2\overline{(\int_{X}u(X)\cdot\psi^*(X)\ dX) (\int_{X}\phi(X')\cdot u^*(X')\ dX')}-2\lambda\int_{X}\phi(X)\cdot\psi^*(X)dX, \nonumber\\
    &=2\int_{X}[\int_{X}\overline{u(X)\cdot u^*(X')}\phi(X')dX'-\lambda\phi(X)]\cdot\psi^*(X)dX.
\end{align*}
$$

$\psi$ being an arbitrary variation, and because $u$ are not functions here but vectors, the auto-correlation function is replaced by a tensor product matrix, and we obtain from (5)

$$\begin{equation}
    \int_{X}\overline{u(X) \otimes u^*(X')}\phi(X')dX'=\lambda \phi(X).
\end{equation}$$

Let $\mathcal{R}: L^2(\Omega \times \mathcal{G}) \to L^2(\Omega \times \mathcal{G})$ be the operator defined by $\mathcal{R}\Phi(X)=\int_{X} R(X,X')\Phi(X')dX'$, where $$\begin{equation*}
R(X,X')=\overline{u(X) \otimes u^*(X')}
\end{equation*}$$
is the correlation tensor in two points.
The classical method consists in replacing the mean $\overline{(\cdot)}$ by an average over the parameter of interest. On the contrary, the snapshots POD replaces the mean as a space mean over the spatial domain $\Omega$.

We obtain the following eigenvalue problem:

$$\begin{equation}
    \mathcal{R}\phi=\lambda\phi.
\end{equation}$$

which gives with the Snapshot-POD $$\frac{1}{Ntrain}\sum_{k=1}^{Ntrain} (u_i,u_k) \Phi_k=\lambda \Phi_i,$$
where $\Phi_i=\sum_{k=1}^{Ntrain} a_k^i u_k, \ i=1,\dots,N.$

$\mathcal{R}$ is a positive linear compact self-adjoint operator on $L^2(\Omega \times \mathcal{G})$. Indeed, 

$$\begin{align} (\mathcal{R}\Phi,\Phi)&=\int_X\int_X R(X,X')\Phi(X')dX' \cdot \Phi^*(X)dX, \nonumber \\
    &=\int_X\int_{X} \overline{u(X) \otimes u^*(X')}\Phi(X')dX' \cdot\Phi^*(X)dX, \nonumber \\
    &=\overline{\int_{X} u(X) \cdot \Phi^*(X) dX \int_{X} u^*(X')\cdot\Phi(X')dX'}, \nonumber \\
    &=\overline{\lVert{(u,\Phi)}\rVert^2} \geq 0. \nonumber \end{align}$$
In the same manner, we can show that $(\mathcal{R}\Phi,\Psi)=(\Phi,\mathcal{R}\Psi)$ for all $(\Psi,\Phi)\in [L^2(\Omega \times \mathcal{G})]^2$.

 Therefore, the spectral theory can be applied and guarantees that the maximization problem (4) has one unique solution equal to the largest eigenvalue of the problem (7).
which can be reformulate as a Fredholm integral equation 

$$\begin{equation}
   \int_{X}R(X,X')\Phi_n(X')dX'=\lambda^n \Phi_n(X),
\end{equation}$$

As a consequence, there exists a countable family of solutions $\{\lambda^n,\Phi_n\}$ to equation (8) which represent the eigenvalues and the POD eigenvectors of order $n=1,\dots,+\infty$.
The $(\Phi_n)_{n=1,\dots,+\infty}$ are orthogonal (and we can normalize them), and the eigenvalues are all positives.

The POD eigenfunctions define a orthonormal basis of the reduced space. Thus, any realization of $u $ can be approched by a linear combination of these functions:

$$\begin{equation}
u^N(X)=\overset{N}{\underset{n=1}{\sum}}a_n\Phi_n(X),
\end{equation}$$

where the coefficients $a_n$ in the case of the snapshot-POD are defined as

$$\begin{equation}
a_n=\int_{\Omega} u \Phi_n(x),
\end{equation}$$

We can measure the accuracy of the reduced basis through the POD Energy

$$E(\Phi_1,\dots,\Phi_N)=\sum_{i=1}^{Ntrain} \lVert u_i - u_i^N \rVert_2^2=\sum_{i=N+1}^{Ntrain} \lambda_i.$$

So, we can select the number of modes $N$ such that $E(\Phi_1,\dots,\Phi_N) \leq \varepsilon$ for a prescribed tolerance. Thus, the number of modes required for the RB can be given by the Relative Information Content (RIC), denoted $I(N)$. The RIC is defined as the ratio between the $N$ first eigenvalues and their total sum and must be close to one.

$$\begin{equation}
  I(N)=\frac{\overset{N}{\underset{k=1}{\sum}}\lambda_k}{\overset{N_{train}}{\underset{i=1}{\sum}}\lambda_i}.
\end{equation}
$$



- #### Offline algorithm

The snapshots POD is obtained from the eigenvalue equation (8) where the average is evaluated as a spatial average on the domain $\Omega$ and the variable $X$ is related to the parameters $\nu$. We denote $Ntrain$ the number of training snapshots ($N \leq Ntrain$). 
The reduced space is denoted $V_h^N$ and can be represented either by the POD basis or by the snapshots which define two basis of $V_h^N$. Thus, the POD eigenfunctions $\Phi$ can be written as a linear combinaison of the dataset:

$$\begin{equation*}
\Phi_i=\overset{Ntrain}{\underset{k=1}{\sum}}a_i^k u_k, i=1,\dots,N
\end{equation*}$$

where $(a_k)_{k=1,\dots,Ntrain}$ remains to be determined.
With equation (14), we have

$$\begin{equation}
    \frac{1}{Ntrain}\overset{Ntrain}{\underset{k=1}{\sum}}(u_i,u_k)\ \Phi_k=\lambda \Phi_i, i=1,\dots,Ntrain.
\end{equation}$$

Therefore, replacing $\Phi$ by its new expression in equation yields 

$$\begin{equation}
 \frac{1}{Ntrain}  \overset{Ntrain}{\underset{k=1}{\sum}}(u_i,u_k) a_k^i=\lambda a_i^i,\quad \forall i=1, \dots, Ntrain.
\end{equation}$$

Let us now describe the algorithm in detail:

- The first step consists in solving numerically the equations of the problem for several training parameters.
At the end of this step, we obtain a vector of snapshots $(u_h^1,\dots,u^{Ntrain}_h)$, where $h$ corresponds to the mesh size. For several reasons, it is more convenient to work with the fluctuations. We decompose the snapshots by one average over the parameters, denoted $u_{h,m}$, and by one fluctuation part written $u_{h,f}$ and we will estimate the POD modes with the fluctuations.

- Then, we calculate the correlation matrix $C_{i,j}=\int_{\Omega} u_{h,f}^i\cdot u_{h,f}^j$, and we solve the $Ntrain \times Ntrain$ eigenvalue problem: $C v_h^n=\lambda_n {I}_d v_h^n$,   for $n=1...Ntrain$, where $v_h^n=(a_{h,1}^n,\dots,a_{h,Ntrain}^n)$.

-  Suppose the eigenvalues are well ordered ($\lambda_1>...>\lambda_{Ntrain}>0$), we calculate the $N$ RB functions normalized, with $N\leq Ntrain$ the number of required modes:

$$\begin{equation*}
\Phi_h^i=\sum_{j=1}^{Ntrain}a_{h,i}^j u_{h,f}^j, \forall i=1,...,N,
\end{equation*}$$

and we normalize them

$$\begin{equation*}
\Phi_h^i=\frac{\Phi_h^i}{\lVert{\Phi_h^i}\rVert} (\mathrm{which \ is \ equivalent \ to \ dividing \ by \ }\sqrt{\lambda_i}). \end{equation*}$$

For the eigenvalues that are too small, we can use a further step which consists in a Gram-Schmidt procedure to ensure the basis orthonormality: 

$$\begin{equation*}
    \Phi_h^i=\Phi_h^i-\sum_{j=1}^{i-1} (\Phi_h^i,\Phi_h^j)\Phi_h^j.\end{equation*}$$
    
 The number $N$ of functions in the POD basis is chosen, such that $N$ remains small and $I(N)=\frac{\underset{k=1}{\overset{N}{\sum}}\lambda_k}{\underset{k=1}{\overset{Ntrain}{\sum}}\lambda_k}$ is close to $1$.

- #### Online algorithm (POD-Galerkin Projection on the reduced model)

During the last step of the offline stage, the RB $(\Phi_h^i)_{i=1,...,N} \in \mathbf{V}_h^N \subset \mathbf{V}_{div}=\{v \in \mathbf{V}_h: \ \nabla.v=0 \}$ is generated. To predict the velocity $u$ for a new parameter $\nu$, a standard method consists in using a Galerkin projection onto this RB.

This stage is much faster than the execution of HF codes. The assembling reduced order matrices can be computed offline, and therefore only the new problem with these matrices needs to be solved during the online phase.
In what follows, the Galerkin-projection for the velocity field does not contain the pressure field. Indeed, in our model problem, we only use the ROM to derived an approximation on the velocity in the reduced space $ \mathbf{V}_h^N$, since here the basis functions satisfy both the boundary conditions and the divergence-free constrain of the continuity equation.

Having reduced bases $(\Phi_i)_{i=1,\dots,N}$ for the velocity and $(\Psi_i)_{i=1,\dots,N}$ for the pressure (but we will see that there is no need to compute this RB), we may complete the offline part by assembling the new matrices of our problem: 

Instead of using $(u,p)$ in our scheme as before

$$\begin{equation*}
\nu (\nabla u, \nabla v) + ((u \cdot \nabla) u,v) -  (p, \nabla \cdot v) - (q, \nabla \cdot u) + 10^{-10} (p, q) =0, \textrm{in } \Omega,
\end{equation*}$$

we replace them by $(\overline{u}+ \sum_{j=1}^N \alpha^k_j \Phi_j,\overline{p} + \sum_{j=1}^N \beta^k_j \Psi_j)$ and $(v,q)$ by $(\Phi_i,\Psi_i)$. Thus, we get:

$$\begin{align*}
& \nu (\nabla \overline{u}, \nabla \Phi_i) + \nu ( \sum_{j=1}^N \alpha^k_j \nabla \Phi_j, \nabla \Phi_i)\\
& \quad \quad \quad + (\overline{u} \cdot  \nabla \overline{u},\Phi_i)  + (\overline{u} \cdot \sum_{j=1}^N \alpha^k_j \nabla \Phi_j,\Phi_i) + (\sum_{j=1}^N \alpha^{k}_j \Phi_j \cdot \nabla \overline{u},\Phi_i) + (\sum_{r=1}^N \alpha^{k-1}_r \Phi_r \cdot \sum_{j=1}^N \alpha^k_j \nabla \Phi_j,\Phi_i) \\
& \quad \quad \quad -  (\overline{p}, \nabla \cdot \Phi_i) -  (\sum_{j=1}^N \beta^k_j  \Psi_j, \nabla \cdot \Phi_i)- (\Psi_i, \nabla \cdot \overline{u}) - (\Psi_i, \sum_{j=1}^N \alpha^k_j \nabla \cdot \Phi_j) + 10^{-10} (\overline{p}, \Psi_i) + 10^{-10} (\sum_{j=1}^N \beta^k_j  \Psi_j, \Psi_i) =0, \textrm{in } \Omega.
\end{align*}$$

Since $\nabla \cdot u^k=0$, the reduced basis functions $(\Phi_i)_{i=1,\dots,N}$ belongs to $V^N \subset V_{div}:=\{v \in V, \nabla \cdot v=0\}$ and $(\Psi_i)_{i=1,\dots,N}$ is of average equal to $0$. Therefore it gives 

$$\begin{equation*}
 \nu (\nabla \overline{u}, \nabla \Phi_i) + \nu \sum_{j=1}^N \alpha^k_j ( \nabla \Phi_j, \nabla \Phi_i)+(\overline{u} \cdot  \nabla \overline{u},\Phi_i) + \sum_{j=1}^N \alpha^k_j (\overline{u} \cdot \nabla \Phi_j,\Phi_i) + \sum_{j=1}^N \alpha^{k}_j ( \Phi_j \cdot \nabla \overline{u},\Phi_i) + \sum_{j=1}^N \sum_{r=1}^N \alpha^k_j \alpha^{k-1}_r  (\Phi_r \cdot  \nabla \Phi_j,\Phi_i) =0, \textrm{in } \Omega.
\end{equation*}$$

We remark that we do not need to generate the reduced basis for the pressure, and that we obtain three kind of terms:

- the ones that do not depend on the coefficients $\alpha$,
- the ones that depend on the coefficients $\alpha^k$,
- and one term that depends on the $\alpha^k$ and on the $\alpha^{k-1}$.


Therefore, we shall regroup these terms such that we get:

$$\begin{equation}
\mathcal{A}_i + \sum_{j=1}^N \mathcal{B}_{ij} \alpha^k_j + \sum_{j=1}^N \sum_{r=1}^N \mathcal{C}_{ijr} \alpha^k_j \alpha^{k-1}_r =0,
\end{equation}$$

where 

$$\begin{equation*}
\mathcal{A}_i = \underbrace{\nu (\nabla \overline{u}, \nabla \Phi_i)}_{A1} + \underbrace{(\overline{u} \cdot  \nabla \overline{u},\Phi_i)}_{A2}, \textrm { and } \mathcal{B}_{ij} =  \underbrace{\nu ( \nabla \Phi_j, \nabla \Phi_i)}_{B1} + \underbrace{(\overline{u} \cdot \nabla \Phi_j,\Phi_i)}_{B2} + \underbrace{( \Phi_j \cdot \nabla \overline{u},\Phi_i)}_{B3} \textrm{ and }\mathcal{C}_{ijr} =( \Phi_r \cdot  \nabla \Phi_j,\Phi_i).
\end{equation*}$$

For the online stage, we just set the new parameter of interest $\nu$, and with the notations

$$\begin{equation*}
M_{ij}=\mathcal{B}_{ij}+\sum_{k=1}^N \mathcal{C}_{ijl} \alpha^{k-1}_l
\end{equation*}$$

and

$$\begin{equation*}
b_i=\mathcal{A}_i,
\end{equation*}$$

the coefficients $(\alpha_{j}^k)_{j=1,...,N}$ at iteration $k$ are obtained by solving the equation:

$$\begin{equation*}
\mathbf{M} \alpha^k=\mathbf{b},
\end{equation*}$$

and we iterate on the residual $\lVert{{\alpha^k}-{\alpha^{k-1}}}\rVert$ until reaching a small treshold in order to obtain ${\alpha}$.
Finally, the approximation is given by

$$\begin{equation*}
u_h(\nu)\simeq u_{h,m}+\sum_{j=1}^N \alpha_j \Phi_h^j. 
\end{equation*}$$


## References

- [Canuto, C., Tonn, T., & Urban, K. (2009). A posteriori error analysis of the reduced basis method for nonaffine parametrized nonlinear PDEs. SIAM Journal on Numerical Analysis, 47(3), 2001-2022](https://epubs.siam.org/doi/abs/10.1137/080724812)

- [Couplet, M., Basdevant, C., & Sagaut, P. (2005). Calibrated reduced-order POD-Galerkin system for fluid flow modelling. Journal of Computational Physics, 207(1), 192-220](https://www.sciencedirect.com/science/article/pii/S0021999105000239)

- [Hijazi, S., Stabile, G., Mola, A., & Rozza, G. (2020). Data-driven POD-Galerkin reduced order model for turbulent flows. Journal of Computational Physics, 416, 109513](https://www.sciencedirect.com/science/article/abs/pii/S0021999120302874)