---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Deriving Vorticity Transport in Index Notation"
subtitle: ""
summary: "How to derive the vorticity transport equation using index summation notation."
authors: [admin]
tags: [math, fluids]
categories: [Notes]
date: 2020-07-22T08:51:52-06:00
lastmod: 2020-07-22T08:51:52-06:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Definitions and Useful Tools

The vorticity transport equation can simply be calculated by taking the curl of
the conservation of momentum evolution equations. See my earlier post going over [expressing
curl in index summation notation]({{< ref
"/post/cross_product_and_curl_in_index_notation/index.md" >}}). In summary, the
curl of a vector $a_j$ can be expressed as:

$$ \nabla \times a_j  = b_k \ \Rightarrow \ \varepsilon_{ijk} \partial_i a_j =
b_k $$

where $\varepsilon_{ijk}$ is the [Levi-Civita symbol]({{< ref
"/post/cross_product_and_curl_in_index_notation/index.md#levi-civita-symbol"
>}}). Some useful properties of the Levi-Civita symbol that will be used are
that it is commutative in multiplication and, since it is invariant of space
and time, it can be brought in or outside a differential operator operator like
a constant:

$$ \varepsilon_{ijk}\frac{\partial}{\partial x_i} \(\phi \) =
\frac{\partial}{\partial x_i} \(\varepsilon_{ijk} \phi \) $$

Note that I'll be using shorthand to express the differential operator,
$\partial\_\phi$, where the index $\phi$ is the index of a spacial variable
*except* for $t$, which represents a time derivative:

$$ \frac{\partial}{\partial x_i} = \partial_i \quad \mathrm{and} \quad
\frac{\partial}{\partial t} = \partial_t$$

For double derivatives, a superscript will be used:

$$ \frac{\partial}{\partial x_i} \frac{\partial}{\partial x_i} = 
\frac{\partial^2}{\partial x_i^2} = 
\partial_i^2 $$

Differential operators are order invariant:

$$ \partial_i \(\partial_j \(u_k\)\)  = \partial_j \(\partial_i \(u_k\)\)$$

{{% alert warning %}} I will be dropping the use of parentheses for the
differential operator, but note that it *is* an operator and is not
commutative: $\partial_i u_j \neq u_j \partial_i$ {{% /alert %}}

Lastly, we'll need to use a common identity of Levi-Civita symbols:

$$ \varepsilon_{ijk} \varepsilon_{imn} = \delta_{jm}\delta_{kn} -
\delta_{jn}\delta_{km} $$

where $\delta_{ij}$ is the [Kronecker delta](https://en.wikipedia.org/wiki/Kronecker_delta).

{{% alert note %}}

Hint: To convert from some arbitrary pair of Levi-Civita symbols that share
one index, first rearrange their indices such that the shared index is the
first for both. Then, the ordering of the Kronecker delta indices is $
\delta_{a_2\  b_2} \delta_{a_3\  b_3} - \delta_{a_2\  b_3} \delta_{a_3\  b_2}$ ,
where $a_i$ and $b_i$ represent the indices of the different Levi-Civita
symbols and their subscripts represent which index is placed in that Kronecker
delta.

{{% /alert %}}

With that taken care of, on to the derivation!

## Curl of Momentum Evolution

The index notation form of the incompressible momentum evolution (or
conservation of momentum equations) is:

$$ \partial_t u_i + u_j \partial_j u_i = - \tfrac{1}{\rho} \partial_i p +
\nu \partial_j^2 u_i $$

Vorticity, $\omega_k$, is given as the curl of velocity, or:

$$ \nabla \times u_j  = \omega_k \ \Rightarrow \ \varepsilon_{ijk} \partial_i u_j =
\omega_k $$

To get vorticity evolution, we can take the curl of the momentum transport equations:

$$ \nabla \times \[\partial_t u_i + u_j \partial_j u_i = -
\tfrac{1}{\rho} \partial_i p + \nu \partial_j^2 u_i \]$$

In index notation, this is the equivalent of multiplying by the Levi-Civita
symbol and a differential operator:

$$ \Rightarrow \varepsilon_{k\ell i} \partial_\ell \[\partial_t u_i + u_j
\partial_j u_i = - \tfrac{1}{\rho} \partial_i p + \nu \partial_j^2 u_i \]$$

Distributing this across the terms, we get:

$$ \begin{align}
 \underbrace{\varepsilon_{k\ell i} \partial_\ell
        \partial_t u_i}_\text{Temporal Term} +
  \underbrace{\varepsilon_{k\ell i} \partial_\ell
       u_j \partial_j u_i}_\text{Advection Term} & = 
  \underbrace{- \varepsilon_{k\ell i} \partial_\ell
       \tfrac{1}{\rho} \partial_i p}_\text{Pressure Term} +
  \underbrace{\varepsilon_{k\ell i} \partial_\ell
        \nu \partial_j^2 u_i}_\text{Viscous Term} \\\\
\Rightarrow \quad \mathbb{T} + \mathbb{A} & = \mathbb{P} + \mathbb{V}
\end{align}$$

We'll treat the named terms individually, then put them back together. I'll
leave the advection term for last since it's more involved than the other
three.

## Individual Terms

### Temporal Term $\mathbb{T}$

$$\mathbb{T} = \varepsilon_{k\ell i} \partial_\ell \partial_t u_i $$

Using the order invariance of derivatives and moving $\varepsilon_{k\ell i}$
inside the derivative operators, we can get the following:

$$ \Rightarrow \quad  \partial_t \varepsilon_{k\ell i} \partial_\ell u_i $$

Since we know that $\omega_k = \varepsilon_{k\ell i} \partial_\ell u_i$, then
we can write:

$$\mathbb{T} = \partial_t \omega_k $$

### Pressure Term $\mathbb{P}$

$$ \mathbb{P} = - \varepsilon_{k\ell i} \partial_\ell \tfrac{1}{\rho}
\partial_i p $$

First we can move the density term out of the derivatives:

$$ \Rightarrow \quad - \tfrac{1}{\rho} \varepsilon_{k\ell i} \partial_\ell
\partial_i p $$

The next step can go one of two ways. First you can simply use the fact that
the curl of a gradient of a scalar equals zero ($\nabla \times \(\partial_i
\phi\) = \mathbf{0}$). Or, you can be like me and want to prove that it is
zero. I'll probably do the former, and put the latter in a separate post. Using
the first method, we get that:

$$ \mathbb{P} = \mathbf{0} $$

### Viscous Term $\mathbb{V}$

$$ \mathbb{V} = \varepsilon_{k\ell i} \partial_\ell \nu \partial_j^2 u_i $$

This step is almost identical to the Temporal Term; rearrange the terms such
that you have the curl of velocity in order to get vorticity:

$$ \Rightarrow \quad \nu \partial_j^2 \varepsilon_{k\ell i} \partial_\ell u_i $$

$$ \Rightarrow \quad \mathbb{V} = \nu \partial_j^2 \omega_k $$

### Advection Term $\mathbb{A}$

$$ \mathbb{A} = \varepsilon_{k\ell i} \partial_\ell u_j \partial_j u_i $$

This is quite a bit more tricky than the other three terms. Note that the
derivatives are operators, so this maybe more explicitly written as:

$$ \mathbb{A} = \varepsilon_{k\ell i} \partial_\ell \(u_j \partial_j \(u_i\) \) $$

First, we'll use a specialized property/rule:

$$ u_j \partial_j u_i = \partial_i \(\tfrac{1}{2} u_j u_j \) + \(\nabla \times
u_n\) \times u_q $$

This is proved in the [Appendix](#appendix). Converting the right term into
index notation we get:

$$ u_j \partial_j u_i = \partial_i \(\tfrac{1}{2} u_j u_j \) +
\varepsilon_{ijq} u_q \(\varepsilon_{jmn} \partial_m u_n\)  $$

Plugging this back into our expression for $\mathbb{A}$, we get:

$$\mathbb{A} = \varepsilon_{k\ell i} \partial_\ell \[\partial_i
\(\tfrac{1}{2} u_j u_j \) + \varepsilon_{ijq} u_q \(\varepsilon_{jmn}
\partial_m u_n\) \] $$

Distributing this through, we get:

$$\Rightarrow \ \varepsilon_{k\ell i} \partial_\ell \[\partial_i \(\tfrac{1}{2}
u_j u_j \) \] + \varepsilon_{k\ell i} \partial_\ell \[ \varepsilon_{ijq} u_q
\(\varepsilon_{jmn} \partial_m u_n\) \] $$

For the lefthand term, note that $u_j u_j$ is just a scalar. Therefore, the
left expression can be surmised as the curl of the gradient of a scalar and it
is then equal to zero. This leaves us with:

$$\Rightarrow \ \mathbb{A} = \varepsilon_{k\ell i} \partial_\ell \[
\varepsilon_{ijq} u_q \(\varepsilon_{jmn} \partial_m u_n\) \] $$

Now let's look at the term in brakets. First we'll rearrange it and switch some
indices without effecting their definition:

$$ \varepsilon_{jqi} \varepsilon_{jmn} u_q \partial_m u_n $$

Since we have two Levi-Civita symbols, we can utilize [the identity shown in
the first section](#definitions-and-useful-tools) to turn this into an
expression of Kronecker deltas.

$$  \varepsilon_{jqi} \varepsilon_{jmn} = \delta_{qm}\delta_{in} -
\delta_{qn}\delta_{im} $$

which gives us
$$ \(\delta_{qm}\delta_{in} - \delta_{qn}\delta_{im}\) u_q \partial_m u_n $$

Multiplying through and putting the expression back into it's original bracket
we get:

$$\begin{align}
\Rightarrow \ \mathbb{A} &= \varepsilon_{k\ell i} \partial_\ell \[
\(\delta_{qm}\delta_{in} - \delta_{qn}\delta_{im}\) u_q \partial_m u_n \] \\\\
&= \varepsilon_{k\ell i} \partial_\ell \[  u_m \partial_m u_i - u_n \partial_i u_n\]
\end{align}
$$

Distributing and rearranging we get:

$$ \Rightarrow \ \mathbb{A} =  u_m \partial_m \underbrace{\varepsilon_{k\ell i}
\partial_\ell u_i}_{\omega_k} - \varepsilon_{k\ell i} \partial_\ell u_n \partial_i u_n $$




## Appendix

