---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Maximum weighted cliques in a graph"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-05T10:44:59+01:00
lastmod: 2020-06-05T10:44:59+01:00
featured: false
draft: true
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

Recently, I had the need to compute maximum weighted cliques on very dense graph. Here is a short summary.

$$\max_x \sum_{i\in V} w_ix_{i}$$
subject to
$$\sum_{i\in I} x_i \le 1\qquad \forall I\in I^*$$