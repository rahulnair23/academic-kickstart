---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Optimal Counterfactuals in Tree Ensembles"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2023-01-23T14:03:27Z
lastmod: 2023-01-23T14:03:27Z
featured: false
math: true
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

This is a brief review of a [paper](https://arxiv.org/abs/2106.06631) by Parmentier and Vidal on "Optimal Counterfactual Explanations in Tree Ensembles". A counterfactual explanation, in the context of machine learning (ML) models, answers the question what is the minimal change in input (features) that yields a different (desirable) outcome? In the context of ML deployments this type of reasoning provides a substantive recourse to people subject to algorithmic decision making. It could also be viewed as a form of justification for a negative outcome.

Two aspects are of interest in this paper. First, it deals with finding a specific input in the feature space that optimizes some criteria. While their criteria is different, this is similar to what we aimed to do in our [paper](https://arxiv.org/abs/2211.01498v1) on quantifying safety via maximum deviations, which looks for the worst case input that maximizes the deviation from the output relative to some reference model. Second, it offers a formulation that could be potentially reused for our safety assessments in tree ensembles using mixed integer programming. 

# Model Setting

A standard supervised classification setting, with a training set $\\{x_k, c_k\\}_{k=1}^n$ with each $x_k$ a $p$-dimensional vector associated with an outcome label $c_k \in \mathcal{C}$ is assumed. 

A tree ensemble (e.g. gradient boosted trees, random forest) is denoted by $\mathcal{T}$ that have a set of trees $t \in \mathcal{T}$ that maps inputs to class probabilities $F_{tc}$. For an input sample $\mathbb{x}$, the tree ensemble returns class $c$ that maximizes the sum of probabilities, i.e. $F_{\mathcal{T}}(\mathbb{x}) = \arg \max_c \sum w_t F_{tc}(\mathbb{x})$.

The paper then develops a mixed integer program in two stages, (a) branching decisions, and (b) feature consistency across trees.

## Branch Choices

The internal structure is modeled as a network flow problem, albeit without an explicit sink node. But this isn't needed. There is one unit of flow that starts at the root node and must travel to a leaf node. The nice trick here is to use decision variables to constrain flows at "transshipment" nodes. This allows the number of binary variables to be reduced. 

Specifically, 

$$
\begin{align}
y_t = 1 & \qquad t \in \mathcal{T} \\\\
y_{tv} = y_{tl(v)} + y_{tr(v)} & \qquad t\in \mathcal{T}, v \in \mathcal{V}_t^I \\\\
\sum _{v \in \mathcal{V} _{td}^I} y _{tl(v)} \le \lambda _{td} &  \qquad t \in \mathcal{T}, d \in D_t \\\\
y _{tv} \in \[0, 1\] & \qquad t \in \mathcal{T}, v \in \mathcal{V} _t^I \cup \mathcal{V} _t^L \\\\
\lambda _{td} \in \\{0, 1\\} & \qquad t \in \mathcal{T}, d \in D_t.
\end{align}
$$

The first equation initializes flow at the root node at each tree to one unit quantity. The next equation is a flow conservation constraint, i.e. the sum of flow to the child nodes is equal to what is available at the parent node. The next constraint works at a specific depth level by limiting the lambdas to be 1 if any of the left branches have non-zero flows. In the supplement they use an induction argument to prove that the result of this set of constraints will result in integer $y$'s even if they are modeled as continuous variables. At first inspection, seems like this would admit fractional solutions as well. 


## Consistency

The $y$ and $\lambda$ decision variables decide on flows through each tree independently. However, depending on what the optimization criteria is - the flows must be consistent across all the trees. For example, a split node that uses gender as a feature, cannot simultaneously branch left and right (in different trees). To address this, the introduce classes of feature variables to handle various cases where features are numerical, categorical, or binary. Take the example of binary features, they introduce a decision variable for each feature $x_i$ which is binary. At each split node, the branching is made consistent by adding the constraints
$$
\begin{aligned}
x_i \le 1 - y_{tl(v)} & \qquad \\\\
x_i \ge y_{tr(v)} & \quad  t \in \mathcal{T}, v \in \mathcal{V} _{ti}^I
\end{aligned}
$$

The objective of their model is to minimize the Gower distance from the input vector. This is treated independently for each of the feature cases and is modeled as a cost that needs to be minimized. This concludes the model specification. Overall, this approach reduces the number of integer variables, and so has computational benefits. Several of the decision variables (but not all) can be relaxed leading to simpler models that are shown to be solved quickly.


Code for this is available [here](https://github.com/vidalt/OCEAN) and may be a point of review for a future note.