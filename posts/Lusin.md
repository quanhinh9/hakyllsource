---
author: Stéphane Laurent
date: '2020-05-12'
highlighter: 'pandoc-solarized'
output:
  html_document:
    highlight: kate
    keep_md: False
  md_document:
    preserve_yaml: True
    variant: markdown
prettifycss: minimal
tags: maths
title: Lusin spaces are strongly Lindelöf
---

Our goal is to prove that a Lusin space is strongly Lindelöf. We will
need the elementary corollary below to prove the key proposition.

**Lemma.** *Let $E$ be a set and $\mathcal{E}\subset\mathrm{Pow}(E)$.
Then the $\sigma$-field $\sigma(\mathcal{E})$ is the smallest collection
of subsets $E$ containing $\varnothing$, $\mathcal{E}$,
$\{E\setminus A \mid A \in \mathcal{E}\}$ and closed under countable
intersections and countable disjoint unions.*

*Proof.* Let $\mathcal{R}$ be the smallest collection of subsets $E$
containing $\varnothing$, $\mathcal{E}$,
$\{E\setminus A \mid A \in \mathcal{E}\}$ and closed under countable
intersections and countable disjoint unions. Let
$\mathcal{R}' = \{A \in \mathcal{R} \mid E \setminus A \in \mathcal{R}\}$.
Then $\mathcal{E} \subset \mathcal{R}'$, and $\mathcal{R}'$ is closed
under complements. So it is enough to show that $\mathcal{R}'$ is closed
under countable unions. But
$\bigcup_n A_n = \bigcup_n (A_n \setminus \bigcup_{i<n}A_i)$, so it is
sufficient to show that $\mathcal{R}'$ is closed under finite unions.
Let $A,B \in \mathcal{R}'$. One has
$A \cup B = (A \setminus B) \cup (B \setminus A) \cup (A \cap B)$ and
this is a disjoint union, so $A \cup B \in \mathcal{R}$. Moreover
$E \setminus (A \cup B) = (E \setminus A) \cap (E \setminus B) \in \mathcal{R}$,
hence $A \cup B \in \mathcal{R}'$. □

**Proposition.** *Let $E$ be a topological space. A class of Borel sets
of $E$ is equal to yhe Borel $\sigma$-field $\mathcal{B}_E$ whenever it
contains all open sets and closed sets and it is closed under countable
intersections and countable disjoint unions.*

*Proof.* Let $\mathcal{C} \subset \mathcal{B}_E$ be such a class. Let
$\mathcal{E}$ be the collection of open sets of $E$, so that
$\mathcal{B}_E = \sigma(\mathcal{E})$. Obviously one has
$\varnothing \in \mathcal{C}$. Moreover, $\mathcal{C}$ contains
$\mathcal{E}$ and $\{E\setminus A \mid A \in \mathcal{E}\}$. Therefore,
by the lemma above, $\mathcal{C}$ contains $\sigma(\mathcal{E})$. □

**Corollary.** *Let $E$ be a metric space. A class of Borel sets of $E$
is equal to $\mathcal{B}_E$ whenever it contains all open sets and it is
closed under countable intersections and disjoint countable unions.*

*Proof.* A closed set in a metric space is the countable intersection of
some open sets. Therefore, if $\mathcal{C}\subset\mathcal{B}_E$ contains
all open sets and is closed under countable intersections, it contains
all closed sets. Thus the result is a consequence of the previous
proposition. □

**Disjoint union topology.** We will resort to the disjoint union in the
proof of the key proposition below. Let ${\{X_i\}}_{i \in I}$ be a
collection of sets. The *disjoint union* of the $X_i$ is the set $$
\bigsqcup_{i \in I} X_i = \bigl\{(x,i) \mid i \in I, x \in X_i\bigr\}. 
$$ Define the canonical injections
$\varphi_i \colon X_i \to \bigsqcup_{i \in I} X_i$ by
$\varphi_i(x) = (x,i)$. When the $X_i$ are topological spaces, the
*disjoint union topology* on $\bigsqcup_{i \in I} X_i$ is the finest
topology on $\bigsqcup_{i \in I} X_i$ for which the canonical injections
are continuous. In other words, this is the final topology with respect
to ${\{\varphi_i\}}_{i \in I}$. This topology is, denoting by $\tau_i$
the topology on $X_i$, $$
\tau = \bigl\{ U \in \mathrm{Pow}(X) \mid \forall i \in I, 
\varphi_i^{-1}(U) \in \tau_i\bigr\}.
$$ If $I$ is countable and the $X_i$ are second-countable, then
$\bigsqcup_{i \in I} X_i$ is second-countable. Now assume the $X_i$ are
metrizable, and for each $i \in I$ take a metric $d_i \leqslant 1$ on
$X_i$ compatible with the topology of $X_i$. For $i,j \in I$,
$x \in X_i$ and $y \in Y_j$, define $$
d\bigl((x,i), (y,j)\bigr) = 
\begin{cases}
d_i(x,y) & \text{ if } i = j \\
2 & \text{ if } i \neq j
\end{cases}.
$$ It is easy to check that $d$ is a metric on
$\bigsqcup_{i \in I} X_i$. Clearly, each $\varphi_i$ is
$(d_i,d)$-continuous. If $Y$ is a topological space, and
$g \colon \bigsqcup_{i \in I} X_i \to Y$ is a map such that every
$g \circ \varphi_i$ is continuous, it is easy to see that $g$ is
continuous for $d$. Thus $d$ induces the disjoint union topology, by the
characteristic property of the final topology. It is easy to see that
$d$ is complete if the $d_i$ are complete. Since [second-countability is
equivalent to separability in a metric space](./conditionalLaw.html), we
deduce from the above that the disjoint union of a countable family of
Polish spaces is a Polish space.

**Key proposition.** *Let $E$ be a Polish space and
$B \in \mathcal{B}_E$. There exist a Polish space $Z$ and a continuous
bijection from $Z$ onto $B$.*

*Proof.* Let $\mathcal{C}$ be the collection of sets $B \subset E$ for
which there exist a Polish space and a continuous bijection from $Z$
onto $B$. [An open subset of a Polish space is
Polish](./conditionalLaw.html), therefore open sets of $E$ belong to
$\mathcal{C}$. By the previous corollary, it is sufficient to show that
$\mathcal{C}$ is closed under countable intersections and countable
disjoint unions. Let $B_0$, $B_1$, $\ldots$ belong to $\mathcal{C}$.
Take Polish spaces $Z_i$ and continuous bijections
$g_i\colon Z_i \to B_i$. Define $$
Z = \left\{ z \in \prod_{i\geqslant 0} Z_i 
\mid g_0(z_0) = g_1(z_1) = \cdots \right\}.
$$ Then $Z$ is closed in the Polish space $\prod_{i\geqslant 0} Z_i$,
therefore is Polish. Define $g \colon Z \to E$ by $g(z) = g_0(z_0)$.
Then $g$ is a continuous bijection from $Z$ onto
$\bigcap_{i \geqslant 0} B_i$. Thus $\mathcal{C}$ is closed under
countable intersections.

Now let $B_0$, $B_1$, $\ldots$ be pairwise disjoints sets belonging to
$\mathcal{C}$. Take Polish spaces $Z_i$ and continuous bijections
$g_i\colon Z_i \to B_i$. The disjoint union
$Z = \bigsqcup_{i \geqslant 0}Z_i$ of the $Z_i$ is a Polish space.
Define $g\colon Z \to E$ by $g\bigl((z,i)\bigr) = g_i(z)$. Then $g$ is a
continuous bijection from $Z$ onto $\bigcup_{i \geqslant 0} B_i$. Thus
$\mathcal{C}$ is closed under countable disjoint unions. □

Say that a topological space is a *Lusin space* if it is homeomorphic to
a Borel set of a compact metric space.

**Corollary.** *If $E$ is a Lusin space, then there exist a Polish space
$Z$ and a continuous bijection from $Z$ onto $E$.*

*Proof.* A compact metric space is Polish. Apply the previous
proposition. □

To prove the proposition below, we use the results of the first section
of [the previous post](./conditionalLaw.html).

A topological space is said to be *strongly Lindelöf* if every open set
of this space is Lindelöf.

**Proposition.** *A Polish space is strongly Lindelöf.*

*Proof.* A Polish space is metrizable and separable, therefore it is
Lindelöf. Since an open subset of a Polish space is Polish, we see that
a Polish space is strongly Lindelöf. □

**Lemma.** *The continuous image of a strongly Lindelöf space is a
strongly Lindelöf space.*

*Proof.* Let $E_1$ and $E_2$ be two topological spaces with $E_1$
strongly Lindelöf, and let $f\colon E_1 \to E_2$ be a continuous
function. Let $O_2 \subset E_2$ be an open subset of $f(E_1)$ and let
$\mathcal{C}_2$ be an open cover of $O_2$. Then
$\bigl\{f^{-1}(C) \mid C \in \mathcal{C}_2\bigr\}$ is an open cover of
the open set $O_1 := f^{-1}(O_2)$. Then it has a countable subcover
$\bigl\{f^{-1}(C) \mid C \in \mathcal{D}_2\bigr\} =: \mathcal{D}_1$
where $\mathcal{D}_2$ is a countable subset of $\mathcal{C}_2$. One has
$f\bigl(f^{-1}(C)\bigr) = C$ for every $C \subset f(E_1)$, therefore
$f(\mathcal{D}_1) = \mathcal{D}_2$ is an open subcover of
$f(O_1) = O_2$. Thus $f(E_1)$ is strongly Lindelöf. □

**Theorem.** *A Lusin space is strongly Lindelöf.*

*Proof.* Let $X$ be a Lusin space, and $f \colon Y \to X$ be a
surjective continuous function from a Polish space $Y$ onto $X$. By the
previous proposition, $Y$ is strongly Lindelöf, therefore $X$ is
strongly Lindelöf by the previous lemma. □

References
==========

-   \[1\] Alexander S. Kechris. *Classical descriptive set
    theory.* 1995.

-   \[2\] S. M. Srivastava. *A Course on Borel Sets.* 1998.

-   \[3\] Nikolaos S. Papageorgiou, Patrick Winkert. *Applied Nonlinear
    Functional Analysis: An Introduction*. 2018.
