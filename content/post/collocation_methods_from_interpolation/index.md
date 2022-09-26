---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Collocation Methods from Interpolation"
subtitle: "or 'Collocation is just Interpolation of Differential Equations'"
summary: "An introduction to collocation methods for differential equations, from the perspective of interpolation theory. I explain how collocation methods are more-or-less identical to interpolation methods, with the only difference being the constraints applied to the problem."
authors: ["James Wright"]
tags: ["math", "differential equations", "numerical methods"]
categories: []
date: 2022-09-25T11:34:16-06:00
lastmod: 2022-09-25T11:34:16-06:00
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

{{% toc %}}

## Intro
Collocation methods are a way of approximately solving differential equations (both partial and ordinary). 
They are very simple to code and implement.

This article will cover collocation methods as an extension of simple interpolation. If you're not familiar with interpolation theory, see [**Interpolation Theory 101**]({{< relref "post/interpolation-theory-101" >}}). 
Even if you are, I'll give a quick rundown of the parts of interpolation theory that you'll need for this article:

## Interpolation Fundamentals

1. We want to find a function $g(x)$ such that it meets a set of $m+1$ constraints $g(x_i) = y_i$
2. We assume $g(x)$ to be of the form $g(x) = \sum_{i=0}^N c_i  \phi_i(x)$ [^1]
3. To ensure existence and uniqueness of the solution, we prescribe that the number of degrees-of-freedom (DOFs) $N+1$ equals the number of constraints $m+1$
4. Expanding $g(x)$ for each constraint results in a system of equations with $N+1$ DOFs and $m+1$ equations.

Normal interpolation sets constraints on the value of the function $g$ itself. 
I also introduced [slope constraints]({{< relref "post/interpolation-theory-101#slope-constraints" >}}), which sets a constraint on the slope of the polynomial rather than the polynomial itself.

Collocation is a further continuation of that; **we set the constraint(s) that the function must satisfy a differential equation at select locations**.

[^1]: This is not strictly necessary, but for ease of teaching (and it's broad applicability), we'll make this assumption. See [the interpolation appendix]({{< relref "post/interpolation-theory-101#non-linear-gx-with-respect-to-degrees-of-freedom" >}}) for more information about that.

## Collocation Overview

In collocation, we're doing the exact same procedure as in interpolation, but with different constraints.
These constraints come in two forms:
1. Boundary conditions
2. Satisfying a differential equation

Boundary condition constraints look *identical* to normal interpolation constraints; we prescribe the value of our function (or of the function derivative) at some specific location.
The satisfaction of the differential equation is where things are a bit more interesting. 
Before diving deeper on that, let's discuss an example problem.

## Example PDE

Let's work with the 1-D Poisson equation for a domain of $x \in [0,1]$:

{{< math >}}
$$
\begin{align*}
    u''(x) &= f(x) \quad x \in [0,1] \\
    u(0) &= 1 \\
    u'(1) &= 2
\end{align*}
$$
{{< /math >}}

where the last two equations are our boundary conditions.

### Defining the Solution Function
Following the steps from interpolation, we need to assume a form for our function.
We'll denote this function as $u^h$, which will be an approximate solution to the PDE.
We assume the form:

$$u^h(x) = \sum_{n=0}^N c_n \phi_n(x)$$

where $c_n$ are our DOFs/coefficients and $\phi_n(x)$ are our basis functions.
The choice of $\phi_n(x)$ is left arbitrary for this article. 
A few options are [Legendre](https://mathworld.wolfram.com/LegendrePolynomial.html), [Laguerre](https://mathworld.wolfram.com/LaguerrePolynomial.html), and [Chebyshev](https://mathworld.wolfram.com/ChebyshevPolynomialoftheFirstKind.html) polynomials (which all have their own special properties, not discussed here),
or non-polynomial basis functions, such as the [Fourier series](https://en.wikipedia.org/wiki/Fourier_series).

## Generic PDE

For a given PDE of the form:

$$L(u) = f$$ 

for $L$ the PDE operator, $u$ the state variable, and $f$ some source function, subject to boundary conditions:

{{< math >}}
$$ 
\begin{align*}
	u(0) &= a \\
	u_{,x}(1) &= b
\end{align*}
$$
{{< /math >}}

we approximate the solution as

$$ u \approx \sum_{i=0}^N c_i \phi_i(x) = u^h$$

for $c_i$ coefficients, $\phi_i(x)$ basis functions, and $N$ the number of basis functions.

We choose to enforce that $L(u) = 0$ at a select number of locations in the domain, called **collocation points**: 

$$\{\xi^j\}_{j=1}^{n_c}$$

This forms a set of $n_c$ equations:

$$L(u^h(\xi^j)) = 0$$

However, we also need to enforce the boundary conditions. These form another set of equations to solve:

{{< math >}}
$$ 
\begin{align*}
	u^h(0) &= a \\
	u^h_{,x}(1) &= b
\end{align*}
 $$
{{< /math >}}
 
Since we have $n_c + 2$ equations, we *must* have $n_c + 2$ unknowns, $c_i$. Therefore $N = n_c +2$.

### Matrix Form
If $L$ is linear, we can express this as a $N\times N$ matrix $A$, and vectors $c,b \in \mathbb{L}^N$:

{{< math >}}
$$
\begin{bmatrix}
\phi^1(0) & \phi^1(0) & \cdots & \phi^1(0) \\[8pt]
\phi_{,x}^1(1) & \phi_{,x}^1(1) & \cdots & \phi_{,x}^1(1) \\[8pt]
L(\phi^1(\xi^1)) & L(\phi^2(\xi^1)) & \cdots & L(\phi^N(\xi^1)) \\[8pt]
\vdots  & \vdots  & \ddots & \vdots  \\[8pt]
L(\phi^1(\xi^{n_c})) & L(\phi^2(\xi^{n_c})) & \cdots & L(\phi^N(\xi^{n_c}))
\end{bmatrix}

\begin{bmatrix}
c_1 \\[8pt]
c_2 \\[8pt]
c_3 \\[8pt]
\vdots  \\[8pt]
c_N \\[8pt]
\end{bmatrix}

=
\begin{bmatrix}
a \\[8pt]
b \\[8pt]
f(\xi^1) \\[8pt]
\vdots  \\[8pt]
f(\xi^{n_c}) \\[8pt]
\end{bmatrix}
$$
{{< /math >}}

### Residual form
If $L$ is a nonlinear differential operator, then we'd want to put it into a residual function form:

{{< math >}}
$$
R(u^h) =
R \left(
\begin{bmatrix}
c_1 \\[8pt]
c_2 \\[8pt]
c_3 \\[8pt]
\vdots  \\[8pt]
c_N \\[8pt]
\end{bmatrix}
\right)
= 

\begin{bmatrix}
\sum_{i=0}^N c_i \phi^i(0) - a \\[8pt]
\sum_{i=0}^N c_i \phi_{,x}^i(0) -b \\[8pt]
L\left(u^h(\xi^1)\right) - f(\xi^1) \\[8pt]
\vdots  \\[8pt]
L\left(u^h(\xi^{n_c})\right) - f(\xi^{n_c}) \\[8pt]
\end{bmatrix}
$$
{{< /math >}}

where again $u^h(x) = \sum_{i=0}^N c_i \phi^i(x)$. You'd then use a non-linear solver to drive the residual to zero.

## Quick self-reference
The quick summary of collocation methods is that we assume some continuous, finite-dimensional form for the solution to a PDE. We define the solution to agree with a number of constraints equal to the dimensionality of the assumed solution form.

The number of constraints is equal to the number of collocation points + the total number of boundary conditions.

In the case of vector PDEs, the number (and location) of the collocation points must be the same, but their boundary condition count may be different. Thus, the dimensionality of the solution for each vector component maybe different.

