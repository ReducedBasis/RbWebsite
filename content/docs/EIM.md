---
title: EIM
weight: 3
---

The Empirical Interpolation Method (EIM) is used with the POD-Galerkin to obtain an affine decomposition on the parameter.

Indeed, as explained in the Galerkin-POD section ([here](/docs/pod)), we need an affine decomposition with respects to the parameter, then the parameter-independent terms are computed offline, making the online computation faster.

The EIM build a linear combination of fully determined solutions from basis functions $(q_i)_{i=1,...,M}$ depending on some interpolating points (called "magic points") and some values $(\alpha_i)_{i=1,...,M}$ relying on certain instances of the parameter $\nu$, selected within the algorithm.
Let us introduce the method with the following example:

Consider a function

$$ g(x,\nu)= \frac1{\sqrt{(x_1-\nu_1)^2+(y_1-\nu_2)}},$$

with $x=(x_1,x_2)$ and $\nu=(\nu_1,\nu_2)$.
- #### "OFFLINE"

The first chosen parameter $\nu^1$ is the one that maximizes $g$ on norm $\mathcal{L}^{\infty}$ (we want to retrieve most information on $g$ with few points) and the associated magic point $x^1$ is the point that gives the most information on $g(\cdot,\nu^1)$ i.e. which maximizes its modulus.

Then the first basis function is $q_1(\cdot) = \frac{g(\cdot,\nu^1)}{g(x^1, \nu^1)}$.                                                                                                                                                                                  
We can then find $\alpha_1$ as the coefficient corresponding to this basis function:

We compute for each training parameters $\nu \in \mathcal{G}$ the real $G$ such that $G=g(x^1,\nu)$ and then we solve the problem $Q \alpha^1(\nu)=G$ where $Q$ at this initialization step is just $q_1(x^1)$.


Then, we find recursively the $M$ basis functions with the following interpolation problem

Find $\{\alpha_j^{M-1}(g)\}_{1\leq j \leq M}$ such that
$$ \forall 1 \leq i \leq M-1,\  \mathcal{I}_{M-1}[g](x^i)= g(x^i),$$

where the EIM operator is defined by                                                                                                                                                                                                 
 $$\mathcal{I}_{M-1}[g]=\sum_{j=1}^{M-1}  \alpha_j^{M-1} q_j.$$ 

- #### " ONLINE " 

Now we are interested  by a new parameter $\nu^{target}$.
To obtain the linear decomposition, we solve a reduced problem from the previously computed basis functions and with the magic points (see details in the Python notebook).

The solution approximation is then given by
$$ g^M(x,\nu^{target})=\sum_{i=1}^M \alpha(\nu^{target}) q(x). $$ 


There exists a generalized form of this method (GEIM) and a discrete version named DEIM. The GEIM replaces the $M$ pointwise evaluations used by the EIM by general measures. In the presence of measurement noise, a stabilization of the method can be employed. 

## Codes:
[Python jupyter notebook](/post/eim/)

[Code with FreeFem++](/uploads/EIM.edp)

[FEniCS](https://colab.research.google.com/github/RBniCS/RBniCS/blob/open-in-colab/tutorials/05_gaussian/tutorial_gaussian_eim.ipynb)

## References

- [EIM simultaneous RB - SER version (nethodology) - Daversin, C., & Prud'Homme, C. (2015). Simultaneous empirical interpolation and reduced basis method for non-linear problems. Comptes Rendus Mathematique, 353(12), 1105-1109](https://www.sciencedirect.com/science/article/pii/S1631073X15002149)