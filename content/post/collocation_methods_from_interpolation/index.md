---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Collocation Methods from Interpolation"
subtitle: "or 'Collocation is just Interpolation of Differential Equations'"
summary: "An introduction to collocation methods for differential equations, from the perspective of interpolation theory. I explain how collocation methods are more-or-less identical to interpolation methods, with the only difference being the constraints applied to the problem."
authors: ["James Wright"]
tags: ["math", "differential equations", "numerical methods"]
categories: ["Notes"]
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

Boundary condition constraints look *identical* to normal interpolation constraints; we prescribe the value of our function (or of the function derivative) at some specific location.[^2]
The satisfaction of the differential equation is where things are a bit more interesting.

[^2]: This is true for Dirichlet ($u=g$) and Neumann ($u'=h$) boundary conditions, and any other boundary conditions that specifies a value for the solution function or it's derivative. However, this is not true for other classes of boundary conditions, such as [Robin boundary conditions](https://en.wikipedia.org/wiki/Robin_boundary_condition) ($gu + hu' = a$). For Robin and it's ilk, you would express it as a differential operator acting only on the boundary ($\mathcal{L}(u) = gu + hu' = a$ on the boundary).

## Collocation for a Generic Differential Equation
For a given differential equation (DE), assume we can write it in the form:

$$\mathcal{L}(u) = f$$

for $\mathcal{L}$ the DE operator, $u$ the solution function, and $f$ some source function.
For example, the Poisson PDE is given by $u''(x) = f$, therefore $\mathcal{L}(u) = u''$.
Another, more complicated example, would be the [Blasius Boundary layer equation](https://en.wikipedia.org/wiki/Blasius_boundary_layer#Blasius_equation_-_first-order_boundary_layer), which is given as $2u''' + u'' u = 0$.
In this case, $\mathcal{L}(u) = 2u''' + u'' u$.

The DE is subject to boundary conditions (and/or initial conditions).
We will assume that the domain of the problem is $x = [0,1]$, and thus the boundaries are at $x=0$ and $x=1$.
We denote the number of boundary conditions to be $m_b$.
We will consider two here, though many other are possible.

{{< math >}}
$$ 
\begin{align*}
	u'(1) &= a \\
	u''(1) &= b
\end{align*}
$$
{{< /math >}}

### Defining the Solution Function
Following the steps from interpolation, we need to assume a form for our function.
We'll denote this function as $u^h$, which will be an approximate solution to the PDE.
We assume the form:

$$u^h(x) = \sum_{n=0}^N c_n \phi_n(x)$$

where $c_n$ are our DOFs/coefficients and $\phi_n(x)$ are our basis functions.
The choice of $\phi_n(x)$ is left arbitrary for this article. 
A few options are [Legendre](https://mathworld.wolfram.com/LegendrePolynomial.html), [Laguerre](https://mathworld.wolfram.com/LaguerrePolynomial.html), and [Chebyshev](https://mathworld.wolfram.com/ChebyshevPolynomialoftheFirstKind.html) polynomials (which all have their own special properties, not discussed here),
or non-polynomial basis functions, such as the [Fourier series](https://en.wikipedia.org/wiki/Fourier_series).

By defining $u=u^h$, we approximate the continuous PDE given above as:
{{< math >}}
$$
\begin{align*}
    \mathcal{L}(u^h) &= f \\
    u'^{\,h}(1) &= a \\
    u''^{\,h}(1) &= b
\end{align*}
$$
{{< /math >}}

### Defining the Constraints

Before we determine the number of DOFs ($N+1$), we need to know the number of constraints.
I've already mentioned that the boundary conditions represent constraints, so each boundary condition represents a constraint.
We'll denote the number of boundary constraints as $m_b$.

But for "Satisfying the differential equation", the PDE is defined for the entire domain, not discrete locations.
This is equivalent to defining an infinite number of constraints, one for for every possible location within the domain.
This is obviously not feasible. 

Therefore, **we choose a select number of locations in the domain to enforce the PDE**.
The locations that we use are called **collocation points**.
We'll denote the number of collocation points as $m_c$.

So the total number of constraints are the number of boundary conditions plus the number of collocation points.
Since we need the number of DoFs to match the number of constraints (see [Interpolation Theory 101]({{< relref "post/interpolation-theory-101#polynomial-order" >}})), we get:

$$N + 1 = m_b + m_c $$

### Determining the Collocation Points

We still need to determine how many collocation points and where on the domain they should be.
This is a more nuanced discussion, so I've moved a more detailed discussion of it to [Appendix A](#appendix-a-determining-collocation-points-continued).

For right now, we'll assume the number of collocation points is arbitrary and that the points are evenly distributed across the domain.
We denote the set of collocation point locations as:

{{< math >}}
$$\left\{\xi_j \right\}_{j=1}^{m_ c}$$
{{< /math >}}

### Creating the System of Equations
To create the system of equations, just like with interpolation, we simply express each constraint in terms of $u^h$ and expand the definition of it.
So for our boundary conditions, we end up with:

{{< math >}}
$$
\begin{align*}
u'^{\,h}(1) = a \quad &\Rightarrow \quad c_0 \phi_0'(1) + c_1 \phi_1'(1) + \dots + c_N \phi_N'(1) = a \\
u''^{\,h}(1) = b \quad &\Rightarrow \quad c_0 \phi_0''(1) + c_1 \phi_1''(1) + \dots + c_N \phi_N''(1) = b
\end{align*}
$$
{{< /math >}}

The [slope constraints section of my interpolation theory post]({{< relref "post/interpolation-theory-101#slope-constraints" >}}) has a derivation for {{< math >}}$u'^{\, h}(x) = \sum_{n=0}^N c_n \phi_n'(x)${{< /math >}}.

Next, we do the same for the collocation points.
This forms a set of $m_c$ equations:

{{< math >}}
$$
\begin{align*}
\mathcal{L}(u^h(\xi_1)) &= f(\xi_1) \\
\mathcal{L}(u^h(\xi_2)) &= f(\xi_2) \\
\vdots  \\
\mathcal{L}(u^h(\xi_{m_c})) &= f(\xi_{m_c}) \\
\end{align*}
$$
{{< /math >}}


### Matrix Form
If $\mathcal{L}$ is linear, we can express the PDE as:

$$\mathcal{L}(\sum_{n=0}^N c_n \phi_n(x)) = \sum_{n=0}^N c_n \mathcal{L}(\phi_n(x))$$

An example of a linear operator would be the Poisson equation $\mathcal{L}(u) = u''$, since the derivative operator is a linear operator itself.

We can take the coefficients $c_n$ and move them into a separate vector $c$ and can form a $(N+1) \times (N+1)$:

{{< math >}}
$$
\begin{bmatrix}
\phi^1(0) & \phi^1(0) & \cdots & \phi^1(0) \\[8pt]
\phi_{,x}^1(1) & \phi_{,x}^1(1) & \cdots & \phi_{,x}^1(1) \\[8pt]
\mathcal{L}(\phi^1(\xi^1)) & \mathcal{L}(\phi^2(\xi^1)) & \cdots & \mathcal{L}(\phi^N(\xi^1)) \\[8pt]
\vdots  & \vdots  & \ddots & \vdots  \\[8pt]
\mathcal{L}(\phi^1(\xi^{m_c})) & \mathcal{L}(\phi^2(\xi^{m_c})) & \cdots & \mathcal{L}(\phi^N(\xi^{m_c}))
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
f(\xi^{m_c}) \\[8pt]
\end{bmatrix}
$$
{{< /math >}}

### Residual form
If $\mathcal{L}$ is a nonlinear differential operator, we cannot bring the sum over coefficients out of the operator argument.
An example of a nonlinear differential operator would be the Blasius equation $\mathcal{L}(u) = 2u''' + u'' u$.
The $u'' u$ term is specifically what makes the differential operator non-linear.

Since we cannot separate the coefficients from a nonlinear operator argument, we cannot form a matrix system of equations.
Instead, we need to create a residual function, which can then be used by a nonlinear equation solver to find our solution.

To obtain the residual function, we simply take the system of equations defined [above](#creating-the-system-of-equations) and shift all the terms to the left-hand side such that they all equal zero.
Since the coefficients $c_n$ completely define $u^h$, we often say that the residual is a function of the coefficients.
Doing this, we obtain:

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
\mathcal{L}\left(u^h(\xi^1)\right) - f(\xi^1) \\[8pt]
\vdots  \\[8pt]
\mathcal{L}\left(u^h(\xi^{m_c})\right) - f(\xi^{m_c}) \\[8pt]
\end{bmatrix}
$$
{{< /math >}}

## Example PDE: Poisson

Let's apply this to the 1-D Poisson equation for a domain of $x \in [0,1]$:

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

We take the same general form of $u^h$ as in the general case:

$$u^h(x) = \sum_{n=0}^N c_n \phi_n(x)$$

By defining $u=u^h$, we approximate the continuous PDE given above as:

{{< math >}}
$$
\begin{align*}
    u''^{\,h}(x) &= f(x) \quad x \in [0,1] \\
    u^h(0) &= 1 \\
    u'^{\,h}(1) &= 2
\end{align*}
$$
{{< /math >}}

### Creating the System of Equations
For our boundary conditions, we end up with:

{{< math >}}
$$
\begin{align*}
u^h(0) = 1 \quad &\Rightarrow \quad c_0 \phi_0(0) + c_1 \phi_1(0) + \dots + c_N \phi_N(0) = 1 \\
u'^{\,h}(1) = 2 \quad &\Rightarrow \quad c_0 \phi_0'(1) + c_1 \phi_1'(1) + \dots + c_N \phi_N'(1) = 2
\end{align*}
$$
{{< /math >}}

For the PDE itself, we just evaluate the PDE approximated with $u^h$ at the collocation points:

{{< math >}}
$$
\begin{align*}
u''^{\,h}(\xi_1) =  \quad &\Rightarrow \quad c_0 \phi_0''(\xi_1) + c_1 \phi_1''(\xi_1) + \dots + c_N \phi_N''(\xi_1) = f(\xi_1) \\
u''^{\,h}(\xi_2) =  \quad &\Rightarrow \quad c_0 \phi_0''(\xi_2) + c_1 \phi_1''(\xi_2) + \dots + c_N \phi_N''(\xi_2) = f(\xi_2) \\
\vdots \\
u''^{\,h}(\xi_{m_c}) =  \quad &\Rightarrow \quad c_0 \phi_0''(\xi_{m_c}) + c_1 \phi_1''(\xi_{m_c}) + \dots + c_N \phi_N''(\xi_{m_c}) = f(\xi_{m_c}) \\
\end{align*}
$$
{{< /math >}}

Since the Poisson operator is linear, we can form the following matrix problem:

{{< math >}}
$$
\begin{bmatrix}
\phi_0(0) &  \phi_1(0) & \dots &  \phi_N(0) \\
\phi_0'(1) &  \phi_1'(1) & \dots &  \phi_N'(1)  \\
\phi_0''(\xi_1) &  \phi_1''(\xi_1) & \dots &  \phi_N''(\xi_1) \\
\phi_0''(\xi_2) &  \phi_1''(\xi_2) & \dots &  \phi_N''(\xi_2) \\
\vdots  & \vdots  & \ddots & \vdots  \\[8pt]
\phi_0''(\xi_{m_c}) &  \phi_1''(\xi_{m_c}) & \dots &  \phi_N''(\xi_{m_c}) \\
\end{bmatrix}

\begin{bmatrix}
c_1 \\
c_2 \\
c_3 \\
c_4 \\
\vdots  \\
c_N \\
\end{bmatrix}

=
\begin{bmatrix}
1 \\
2 \\
f(\xi_1) \\
f(\xi_2) \\
\vdots  \\
f(\xi_{m_c}) \\
\end{bmatrix}
$$
{{< /math >}}

And tada! You have the collocation method for the Poisson PDE.
All you need to do is solve this matrix equation for the DOFs $c_n$ and then reconstruct $u^h$ to have your solution!

## Appendix A: Determining Collocation Points Continued

Note we have yet to determine the number of collocation points or the location of the collocation points.

### How many?
Choosing the number of collocation points $m_c$ is completely arbitrary.
Even if $u^h$ exactly satisfies the PDE at the collocation points, there are no guarantees how well (or poorly) $u^h$ satisfies the PDE for the rest of the domain.
What we do know is that the more points you have, the more accurate your solution becomes (generally).
How many points is "good enough" is highly problem dependent.

Beyond accuracy though, increasing the number of points also increases the size of your system of equations.
Depending on your choice of collocation locations and basis functions, the system of equations you end up with may be infeasible to solve.

### Where should the collocation points be located?

This is also an arbitrary choice; they can be wherever you want them to be.
But this choice can have significant implications on the stiffness of your resulting system of equations.

The general advice is to choose locations that are the roots of some $m_c -1$ order orthogonal polynomial.
The reason for this choice is beyond the scope of the article, but it is intimately related to Gauss quadrature (which also uses the roots of orthogonal polynomials).

A good general choice to place collocation points at [Chebyshev nodes](https://en.wikipedia.org/wiki/Chebyshev_nodes), which are the roots of [Chebyshev polynomials](https://en.wikipedia.org/wiki/Chebyshev_polynomials).

