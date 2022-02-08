---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Cross Product and Curl in Index Notation"
subtitle: "Or how to use the Levi-Civita symbol."
summary: "Review of how to perform cross products and curls in index summation notation. In essence, this ends up being an overview on how to apply the Levi-Civita symbol in these contexts."
authors: [admin]
tags: [index notation, Levi-Civita, math]
categories: [Notes]
date: 2020-07-21T18:00:45-06:00
lastmod: 2020-07-21T18:00:45-06:00
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

aliases:
  - /post/cross_product_and_curl_in_index_notation

slug: cross_product_and_curl_in_index_notation
---

Here are some brief notes on performing a cross-product using index notation.
This requires use of the [Levi-Civita
symbol](https://en.wikipedia.org/wiki/Levi-Civita_symbol), which may also be
called the permutation tensor.

## Levi-Civita Symbol

The Levi-Civita symbol is often expressed using an $\varepsilon$ and takes the
following definition:

$$  \varepsilon_{ijk} =
  \begin{cases}
         +1 & \text{if } (i,j,k) \text{ is even permutation,} \\\\
         -1 & \text{if } (i,j,k) \text{ is odd permutation,} \\\\
          0 & \text{if } i = j, \text{ or } j = k, \text{ or } k = i
\end{cases}
$$

For a 3D system, the definition of an odd or even permutation can be shown in
[Figure 1](#figure-permutation-pattern-for-levi-civita-symbol-in-3d-altered-from-sourcehttpscommonswikimediaorgwikifilepermutation_indices_3d_numericalsvg).

{{< figure src="./Permutation_indices_3d_numerical_edit.png"
title="Permutation pattern for Levi-Civita symbol in 3D. [Altered from source.](https://commons.wikimedia.org/wiki/File:Permutation_indices_3d_numerical.svg)"
numbered="true" lightbox="true" width="50%">}}

The permutation is even if the three numbers of the index are in "order", given
allowance to cycle back through the numbers once the end is reached. So if you
skip to the 1 value in the index, going left-to-right should be in numerical
order. For example, if given 321 and starting with the 1 we get 1 $\rightarrow$
3 $\rightarrow$ 2. 132 is not in numerical order, thus it is an odd permutation.

So given $\varepsilon_{ijk}\\,$, if $i$, $j$, and $k$ are $123$, $231$, or $312$,
then $\varepsilon_{ijk}=1$. Note that the order of the indicies matter. If
instead we're given $\varepsilon_{jik}$ and any of the three permutations in
the previous example, then the expression would be equal to $-1$ instead.

## Cross Products in Index Notation

Now we get to the implementation of cross products. This involves transitioning
back and forth from vector notation to index notation. A vector and it's index
notation equivalent are given as:

$$ \mathbf{a} = a_i$$

If we want to take the cross product of this with a vector $\mathbf{b} = b_j$,
we get:

$$ \mathbf{a} \times \mathbf{b}  = a_i \times b_j \ \Rightarrow  \
\varepsilon_{ijk} a_i b_j = c_k$$

Note the indices, where the resulting vector $c_k$ inherits the index not used
by the original vectors. Also note that since the cross product is
anticommutative (ie. $\mathbf{a} \times \mathbf{b} = - \mathbf{b} \times
\mathbf{a}$ ), changing the order of the vectors being crossed requires
changing the indices of the Levi-Civita symbol or adding a negative:

$$ b_j \times a_i \ \Rightarrow  \ \varepsilon_{jik} a_i b_j =
-\varepsilon_{ijk} a_i b_j = c_k$$

Conversely, the commutativity of multiplication (which is valid in index
notation) means that the vector order *can* be changed without changing the
permutation symbol indices or anything else:

$$ b_j \times a_i \ \Rightarrow  \ \varepsilon_{jik} a_i b_j  =
\varepsilon_{jik} b_j a_i$$

### Rule(s) of Thumb for Setting Indices Correctly

1. Start the indices of the permutation symbol with the index of the resulting
   vector.

This will often be the free index of the equation that
the cross product lives in and I normally like to have the free index as the
leading index in multi-index terms.

2. The next two indices need to be in the same order as the vectors from the
   cross product.

Using these rules, say we want to replicate $a_\ell \times b_k = c_j$. Then the
first index needs to be $j$ since $c_j$ is the resulting vector. The other 2
indices must be $\ell$ and $k$ then. This results in:

$$ a_\ell \times b_k = c_j \quad \Rightarrow \quad \varepsilon_{j\ell k} a_\ell
b_k = c_j$$

## Curl in Index Notation

The curl is given as the cross product of the gradient and some vector field:

$$ \mathrm{curl}\({a_j}\) = \nabla \times a_j  = b_k $$

In index notation, this would be given as:

$$ \nabla \times a_j  = b_k \ \Rightarrow \ \varepsilon_{ijk} \partial_i a_j =
b_k $$

where $\partial_i$ is the differential operator $\frac{\partial}{\partial
x_i}$.

{{% callout warning %}}

Note that $\partial_k$ is ***not*** commutative since it is an operator. It may be
better to write $\partial_k u_i$ as $\partial_k \(u_i\)$ to more explicitly
denote it's nature as an operator on $u_i$.

{{% /callout %}}

These follow the same rules as with a normal cross product, but the
first "vector" is always going to be the differential operator. Since $\nabla$
is hardly ever defined with an index, the [rule of
thumb](#rules-of-thumb-for-setting-indices-correctly) can come in handy when
trying to translate vector notation curl into index notation.

For example, if I have a vector $u_i$ and I want to take the curl of it, first
I need to decide what I want the resulting vector index to be. Let's make it be
$\ell$.  Due to index summation rules, the index we assign to the differential
operator may be any character that isn't $i$ or $\ell$ in our case. Let's make
it be $k$. Putting that all together we get:

$$ \mathrm{curl}\(u_i\) = \varepsilon_{\ell ki} \partial_k u_i = \omega_\ell $$

