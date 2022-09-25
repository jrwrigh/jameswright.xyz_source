---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Interpolation Theory 101"
subtitle: ""
summary: "A quick overview of the basics of interpolating a set of function points using a polynomial. This servers as a percursor to a (future) post on collocation methods"
authors: ["James Wright"]
tags: ["math"]
categories: []
date: 2022-09-25T09:47:26-06:00
lastmod: 2022-09-25T09:47:26-06:00
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

{{< toc >}}

{{% callout note %}}
Pre-requisites: There are a few assumptions made for the reader's background. Mainly, that you're familiar with the concept of a *basis* in the linear algebra since and, in particular, function space bases (such as the monomials). 
{{% /callout %}}

## Interpolation Problem Intro
In interpolation, we wish to find a function $g(x)$ that satisfies $g(x_i) = y_i$ for some positive number $i$. We'll call these **constraints** on the function $g(x)$. We say that the constraint is *satisfied* if $g(x)$ passes through the point $(x_i, y_i)$.

In the case of polynomial interpolation, we assume a form 
$$g(x) = \sum_{i=0}^N c_i \phi_i(x)$$ 

where $\phi_i(x)$ represents polynomial basis functions. The simplest case of this is the monomials, where $\phi_i(x) = x^i$, such that:

$$g(x) = c_0  + c_1 x + c_2 x^2 + \dots + c_N x^{N}$$

However, we can choose $\phi_i$ to be any set of basis functions for a polynomial space, such as the Legendre, Laguerre, and Chebyshev polynomials (which all have their own special properties, not discussed here).

Note that $N$ is the order of our polynomial. I'll maintain this notation throughout.

## Polynomial order
A question does remain though: What kind (order) of polynomial should we try to interpolate with? 

Well, if we have $m+1$ constraints, then we need, *at minimum*, a $m$th order polynomial to interpolate them. For example, we need 2 points ($m+1=2 \ \therefore m=1$) to make a line ($N = m = 1$ order polynomial). 

So what about polynomials greater than order $m$? Well, then an infinite number polynomials satisfy the constraints. Example: If we only have 1 constraint ($m+1=1 \ \therefore m=0$) and there are an infinite number of lines that can satisfy that constraint. 

We'd like for a solution to the problem to exist ($N \geq m$) and for that solution to be unique ($N < m+1$). Therefore, **the order of the polynomial should be one less than the number of constraints ($N = m$)**.

## Solving the interpolation problem
Now we have $m+1$ constraints, and know that $g(x)$ should be a $N = m$ order polynomial. So how do find $g(x)$?

The most straight forward way is to simply plug the constraints into $g(x)$. Doing this gives us the set of linear equations:

{{< math >}}
\begin{align*}
g(x_0) &= c_0  + c_1 x_1 + c_2 x_1^2 + \dots + c_N x_1^{N} &= y_0 \\
g(x_1) &= c_0  + c_1 x_2 + c_2 x_2^2 + \dots + c_N x_2^{N} &= y_1 \\
\vdots \\
g(x_m) &= c_0  + c_1 x_m + c_2 x_m^2 + \dots + c_N x_m^{N} &= y_m \\
\end{align*}
{{< /math >}}

We turn this into a matrix problem that looks like:

{{< math >}}
$$
\begin{bmatrix}
1  &  x_0 &  x_0^2 & \dots &  x_0^{N} \\[2pt]
1  &  x_1 &  x_1^2 & \dots &  x_1^{N} \\[2pt]
\vdots \\[2pt]
1  &  x_m &  x_m^2 & \dots &  x_m^{N} \\[2pt]
\end{bmatrix}
\begin{bmatrix}
c_0 \\
c_1 \\
\vdots \\
c_N \\
\end{bmatrix}
=
\begin{bmatrix}
y_0 \\
y_1 \\
\vdots \\
y_m \\
\end{bmatrix}
$$
{{< /math >}}

The matrix above is known as a Vandermonde matrix. By solving this system, we find the coefficients $c_i$ that we can then reconstruct into $g(x) = \sum_{i=0}^N c_i x^i$.

## For arbitrary polynomial basis

Remember that we chose $\phi_i(x) = x^i$, which are the monomial bases. 
This works fine, but the resulting problem is very difficult to solve computationally (it is *stiff*). 
Other choices of $\phi_i(x)$ can help reduce the difficulty significantly, such as the aforementioned Chebyshev polynomials. 
**How do we go about finding the interpolating polynomial using other bases?**

We create a different matrix to solve with. 
Recall that we defined $g(x) = \sum_{i=0}^N c_i \phi_i(x)$. 
Using an arbitrary $\phi_i(x)$ instead of the monomials $x^i$, we can apply each constraint and get the following system of equations:

{{< math >}}
$$
\begin{align*}
g(x_0) &= c_0\phi_0(x_0)  + c_1 \phi_1(x_0) + \dots + c_N \phi_N(x_0) &= y_0 \\
g(x_1) &= c_0\phi_0(x_1)  + c_1 \phi_1(x_1) + \dots + c_N \phi_N(x_1) &= y_1 \\
\vdots \\
g(x_m) &= c_0\phi_0(x_m)  + c_1 \phi_1(x_m) + \dots + c_N \phi_N(x_m) &= y_m \\
\end{align*}
$$
{{< /math >}}

This system of equation results in the matrix problem:
{{< math >}}
$$
\begin{bmatrix}
\phi_0(x_0)  & \phi_1(x_0) & \dots & \phi_N(x_0)\\
\phi_0(x_1)  & \phi_1(x_1) & \dots & \phi_N(x_1)\\
\vdots \\
\phi_0(x_m)  & \phi_1(x_m) & \dots & \phi_N(x_m)\\
\end{bmatrix}

\begin{bmatrix}
c_0 \\
c_1 \\
\vdots \\
c_N \\
\end{bmatrix}
=
\begin{bmatrix}
y_0 \\
y_1 \\
\vdots \\
y_m \\
\end{bmatrix}
$$
{{< /math >}}

Just as before, we can then solve this matrix problem, then reconstruct $g(x)$ using the coefficients $c_i$.

## Slope constraints

What if instead of wanting $g(x)$ to go through a specific point, we instead want the function to match a specific slope at a point? This is particularly useful for something like splines, where we want a smooth transition between one curve and another. Well, we can find a $g(x)$ that meets that constraint too!

First assume we replace the first interpolation constraint ($g(x_0) = y_0$) with a slope constraint ($g'(x_0) = s_0$), where $g' = \partial g / \partial x$. 

Recall that we set $g(x) = \sum_{i=0}^N c_i \phi_i(x)$. To find $g'(x)$, we can simply take the derivative of sum expression:

{{< math >}}
$$
\begin{align*}
\frac{\partial}{\partial x} g(x) 
&= \frac{\partial}{\partial x} \left [c_0\phi_0(x)  + c_1 \phi_1(x) + \dots + c_N \phi_N(x) \right] \\
&= \frac{\partial}{\partial x} c_0\phi_0(x)  + \frac{\partial}{\partial x} c_1 \phi_1(x) + \dots + \frac{\partial}{\partial x} c_N \phi_N(x) \\[12pt]
g'(x) &= c_0\phi'_0(x)  + c_1 \phi'_1(x) + \dots + c_N \phi'_N(x)
\end{align*}
$$
{{< /math >}}

So the derivative of $g(x)$ is just the sum of the derivatives of $\phi_i(x)$ weighted by the coefficients $c_i$. This is quite nice, as our function coefficients for $g(x)$ are the same as for $g'(x)$.

Using this result, we can write our slope constraint as:

$$g'(x_0) = c_0\phi'_0(x_0)  + c_1 \phi'_1(x_0) + \dots + c_N \phi'_N(x_0) = s_0$$

Recall that we replace our first interpolation constraint $g(x_0) = y_0$ with the slope constraint $g'(x_0) = s_0$. We can then replace the first equation in the linear system described above with $g'(x_0) = s_0$. This results in the matrix problem:
{{< math >}}
$$
\begin{bmatrix}
\phi'_0(x_0)  & \phi'_1(x_0) & \dots & \phi'_N(x_0)\\
\phi_0(x_1)  & \phi_1(x_1) & \dots & \phi_N(x_1)\\
\vdots \\
\phi_0(x_m)  & \phi_1(x_m) & \dots & \phi_N(x_m)\\
\end{bmatrix}

\begin{bmatrix}
c_0 \\
c_1 \\
\vdots \\
c_N \\
\end{bmatrix}
=
\begin{bmatrix}
s_0 \\
y_1 \\
\vdots \\
y_m \\
\end{bmatrix}
$$
{{< /math >}}

This looks nearly identical to our pure-interpolation matrix problem! The only difference is that the first row of our matrix replaces $\phi_i \rightarrow \phi'_i$ and the first entry on the right hand side vector replaces $y_0 \rightarrow s_0$.

This last result is incredibly powerful. Using it, we can actually solve differential equations. This will be discussed in a later article.
