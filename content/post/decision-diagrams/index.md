---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Zero-Suppressed Decision Diagrams and Independent Sets"
subtitle: "Neat ways to represent sets of graphs"
summary: ""
authors: []
tags: ["optimization", "graph theory"]
categories: []
date: 2021-01-26T13:16:32Z
lastmod: 2021-01-26T13:16:32Z
featured: false
draft: false
math: true
diagrams: true

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

I stumbled across Binary Decision Diagrams (BDDs) by chance. They are an efficient data structure to represent sets of graphs. While a graph $G$ is a set of vertices $V$ along with a set of edges $E$ that connect the vertices, a graph set is a collection of subgraphs over the universe $V$. For example, a collection of feasible paths on $G$ would be a graph set. For a graph, if one has its corresponding BDD, specific types of queries can be handled with remarkable efficiency. See the [this](https://youtu.be/Q4gTV4r0zRs) video from the creators of [grahillion](https://github.com/takemaru/graphillion) for an illustration of count queries. Knuth's Art of Programming has an entire chapter devoted to this, but it hasn't seen too many uses in the operations research community (at least as far as I'm aware). It seems like it could be pretty useful for several combinatorial optimization problems.

So in this post, we'll work through one use - that of finding maximal independent sets in a graph. An independent set is a subset of vertices $V$, no two of which are adjacent. A *maximal* independent set is one where no additional nodes can be included and remain as an independent set. Not to be confused with the maxim**um** set - which looks for an independent set with maximum weight.

We follow the work described in [Morrison et al. (2014)](https://doi.org/10.1007/s10878-014-9722-4) on this problem which uses a Zero-suppressed Decision Diagram (ZDD), a variant of BDD where the false conditions are omitted. This is useful when the feasible solutions are sparse. There are several works that look into using ZDDs on graph and optimization problems, e.g. maximal cliques [(Coudert, 1997)](https://doi.org/10.1109/EDTC.1997.582363) and [0/1 enumeration](http://people.mpi-inf.mpg.de/alumni/d1/2019/behle/azove.html).

# ZDDs

In addition to a graph $G$, we also need a vertex ordering. While the ordering can be arbitrary, it does have an impact on the resulting ZDD. Each node in the ZDD, $Z_s$ is defined by a tuple $(v, lo, hi)$, where $v$ is the vertex it refers to in $G$ and $lo, hi$ are exactly two branches, known as the low and high branch, pointers to other nodes in the ZDD. These can be thought of as binary outcomes of the node from which they emanate. The terminal nodes in the ZDD have special meaning. The $\top$ refers to a **true** outcome, while $\bot$ encodes a **false** outcome.

All this becomes clearer in the example, taken from Morrison's paper, below. The 5-node graph on the left has a corresponding ZDD on the right that encodes all possible maximal independent sets. The solid arrows are high branches, while the dashed ones are low branches. 

![Alt](fig.svg)

This Directed Acyclic Graph (DAG) encodes every possible maximal independent set. Every feasible path from the root node till the terminal node $\top$ represents a maximal independent set. The set it represented by nodes that have high branches coming out. For example, feasible path $v_1, v_2, v_3, v_4,\top$ represents an maximal independent set ${v_2, v_4}$ as these are the only nodes with solid (high) branches. There are several important properties of the ZDD that I skip here, like the uniqueness of nodes in $Z_s$, each node is encountered only once, etc. 

The ZDD can now be used to enumerate all maximal independent sets and support non-trivial operations that would be expensive otherwise, e.g. a count of all maximal independent sets without listing each one. There are other advantages as well. For instance, if there is a need to repeatedly query this ZDD, for example when node weights change, then one can compute this efficiently. Mostly, ZDD offer a compact representation of the underlying set.

# Constructing a ZDD

To do all of this, the ZDD needs to be constructed first. There are few methods to do this for each class of problem you want to solve. [Morrison et al. (2014)](https://doi.org/10.1007/s10878-014-9722-4), for example, tackle the case of the maximal independent set.  

In this case, they describe an algorithm that takes as input a graph and a vertex ordering (this ordering can be arbitrary - but does have an influence on the size of the ZDD). The procedure is recursive, and it each iteration a single node is considered. It first considers if a maximal set can be constructed from the remaining vertices. Remaining here are w.r.t. to the ordering. It also checks if the set is already maximal. For these two cases, the procedure returns with the terminal nodes, i.e. $\bot$ or $\top$. If these conditions are not met, it creates two branches for the current node based on the next undominated node. A new node is added if it hasn't previously been created. 

# Reference implementation

A reference implementation is forthcoming (when time permits!). 

There are existing packages like [graphmillion](https://github.com/takemaru/graphillion) that provide powerful abstractions for common problem classes. See also their accompanying [paper](https://doi.org/10.1007/s10009-014-0352-z). [Azove](http://people.mpi-inf.mpg.de/alumni/d1/2019/behle/azove.html) is another tool (uses binary decision diagrams) for vertex enumeration that may be of interest as well.

