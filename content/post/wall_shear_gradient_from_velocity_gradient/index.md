---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Calculate Wall Shear Gradient from Velocity Gradient"
subtitle: ""
summary: "If the gradient of velocity is already calculated, how could you get
the wall-shear gradient for any arbitrary wall? The aim of this post is to
answer that question and give the reasoning for the result."
authors: [admin]
tags: [math, fluids, CFD]
categories: [Notes]
date: 2020-08-13T09:49:47-06:00
lastmod: 2020-08-13T09:49:47-06:00
featured: false
draft: false
editable: true

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

The gradient of velocity is generally easy to compute in most CFD
post-processing routines. But let's say you want to find the wall shear stress
from this quantity, how would you do that? I'd been searching for an answer to
this question and could never really find one (or at least one that was
satisfying). Eventually I derived out the following solution and figured I'd
post it so that the information was more widely available.

## Initial Definitions

First, let's define more explicitly what we're trying to find. The wall shear
stress is often given as:

$$ \tau_w = \mu \left (\frac{\partial u}{\partial y}\right )
\Bigg\rvert_{y=\text{wall}} $$

However, this isn't very explicit and really only applies to flat plate
boundary layer flows. I'd submit that the "real" definition is dynamic
viscosity ($\mu$) times the wall-normal gradient of velocity tangential to the
wall taken at the wall, or:

$$ \tau_w = \mu \left (\frac{\partial u_\parallel}{\partial n}\right )
\Bigg\rvert_{n=0} $$

This will result in a vector parallel to the wall in the direction of the wall
shear stress.

I'll define the velocity gradient as a tensor $E_{ij}\ $:

$$ E_{ij} = \frac{\partial u_i}{\partial x_j} = \partial_j u_i$$

Note that $E\_{ij}$ is _not_ symmetric and that $\partial_j$ is still an
with $u_i$ as it's input, not multiplication.

Lastly, we have the common form of projecting a vector onto a plane given its
normal vector:

$$ \text{proj}_{\hat n}(\overrightarrow{u}) = \overrightarrow{u} - 
(\overrightarrow{u} \cdot \hat n) \hat n
= u_i - (u_j \hat n_j) \hat n_i$$

where $\hat n$ is the wall-normal unit vector. The right most term is in index
summation notation.

## Preamble

### Assumptions

1. We have a wall-normal unit vector $\hat n_i$
2. We have the velocity gradient tensor $E_{ij} = \partial_j u_i$

### Goal

Obtain:

$$ \left (\frac{\partial u_\parallel}{\partial n}\right ) \Bigg\rvert_{n=0} =
\left (\partial_{\hat n} u_\parallel\right ) \big\rvert_{n=0} = 
f(E_{ij}, \hat n) = f(\partial_j u_i, \hat n)$$

## Derivation

Notice that the wall shear gradient can be broken into two "terms": 

 - gradient in the wall-normal direction
 - velocity tangent to the wall

First we'll define these two "terms" individually

### Gradient in the Wall-Normal Direction

This is simply:

$$\hat n_j \partial_j$$

Gradient in a specific direction should result in a tensor whose rank is the
same as it's input. In other words, the gradient of a scalar in a single
direction should result in a scalar (which is a rank 0 tensor). The summation
over the $j$ index shows that this is true.

### Velocity Tangent to the Wall

Taking the vector projection formula from [Initial
Definitions](#initial-definitions), this is fairly straight forward:

$$ u_{i,\parallel} = u_i - (u_k \hat n_k) \hat n_i$$

### Combining Terms

Putting these together, we get:

$$\underbrace{\hat n_j \partial_j}\_{\partial_{\hat n}}
[\underbrace{u_i - (u_k \hat n_k) \hat n_i }\_{u_{\parallel}}]$$

$$\Rightarrow \hat n_j \partial_j \left [u_k (\delta_{ik} - \hat n_k \hat n_i)
\right]$$

Using product rule:

$$ \Rightarrow (\delta_{ik} - \hat n_k \hat n_i)
\hat n_j \partial_j (u_k) +
u_k \hat n_j \partial_j (\delta_{ik} - \hat n_k \hat n_i)$$

First, let's work with the right hand term (RHT):

$$ \text{RHT} = u_k \hat n_j \partial_j (\delta_{ik} - \hat n_k \hat n_i) $$

$$\Rightarrow u_k \left [\hat n_j \partial_j (\delta_{ik}) - \hat n_j
\partial_j (\hat n_k \hat n_i) \right ]$$

The Kronecker delta is invariant of spacial dimensions, so the left term goes to
zero. Then we can do product rule again on the right term.

$$\Rightarrow u_k \left [\hat n_j \cancelto{0}{\partial_j (\delta_{ik})} - \hat n_j
\partial_j (\hat n_k \hat n_i)) \right ]$$

$$\Rightarrow -u_k  \left [\hat n_i \hat n_j \partial_j (\hat n_k) + \hat n_k
\hat n_j \partial_j (\hat n_i) \right ]$$

Here, $\hat n$ is *not* invariant of spacial location; if you have a non-flat
surface, it will change as you move along the wall. However, note that $\hat
n_j \partial_j$ is the gradient in the wall-normal direction. The $\hat n$
does not change in the wall-normal direction; it only change in the
wall-parallel direction. Thus:

$$\Rightarrow -u_k  \left [\hat n_i \cancelto{0}{\hat n_j \partial_j (\hat
n_k)} + \hat n_k \cancelto{0}{\hat n_j \partial_j (\hat n_i)} \right ]$$

$$ \therefore \text{RHT} = 0 $$

Moving back to the original expression, we're then left with:

$$ \partial_{\hat n} u_{i,\parallel} =  (\delta_{ik} - \hat n_k \hat n_i)
\hat n_j \partial_j (u_k) +
\cancelto{0}{u_k \hat n_j \partial_j (\delta_{ik} - \hat n_k \hat n_i)}$$

Note that we already have the gradient of velocity in the last term, thus:

$$ \partial_{\hat n} u_{i,\parallel} =  (\delta_{ik} - \hat n_k \hat n_i)
\hat n_j E_{kj} $$

$$ \therefore \left (\frac{\partial u_\parallel}{\partial n}\right )
\Bigg\rvert_{n=0} = \bigg( \big[(\delta_{ik} - \hat n_k \hat n_i) \hat n_j
\big] E_{kj} \bigg) \Bigg\rvert_{n=0} = f(\hat n, E_ij)$$

To obtain $\tau_w$, simply multiply by $\mu$:

$$\tau_w = \mu \bigg( \big[(\delta_{ik} - \hat n_k \hat n_i) \hat n_j
\big] E_{kj} \bigg) \Bigg\rvert_{n=0} $$
