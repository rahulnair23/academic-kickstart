---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Barriers to the CPLEX"
subtitle: "Why moving to the cloud is not painless"
summary: ""
authors: []
tags: []
categories: []
date: 2020-10-27T09:40:48Z
lastmod: 2020-10-27T09:40:48Z
featured: true
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

## So you think you need CPLEX? 

To find out more, you review the [marketing material](https://www.ibm.com/analytics/cplex-optimizer). After you wade past that you get to the technical documentation. The links lead you in loops. Where should you start? Is it the [decision optimization](http://ibmdecisionoptimization.github.io/docplex-doc/), [pip package](https://pypi.org/project/cplex/), or is it [this pip package](https://pypi.org/project/docplex/), or the [product documentation](https://www.ibm.com/support/knowledgecenter/en/SSSA5P_12.8.0/ilog.odms.studio.help/Optimization_Studio/topics/COS_home.html)? 

After a while, you realize CPLEX has not just one Python API, but two. There is the the [legacy stuff](https://www.ibm.com/support/knowledgecenter/SSSA5P_12.8.0/ilog.odms.cplex.help/CPLEX/Python/topics/PLUGINS_ROOT/ilog.odms.cplex.help/refpythoncplex/html/overview.html?view=kc) and then the API that also allows for modeling called [DOcplex](https://cdn.rawgit.com/IBMDecisionOptimization/docplex-doc/master/docs/index.html). 

A `pip install cplex` seems to get you a solver - but only the *Community Edition* that allows very small problems. If you need to solve larger problems or need more compute, you could use the [Watson Studio](https://www.ibm.com/cloud/watson-studio) to run experiments on the cloud. Watson doesn't call CPLEX CPLEX of course, but *Decision Optimization*. To get the full version locally, you need the [CPLEX Optimization Studio](https://www.ibm.com/ie-en/products/ilog-cplex-optimization-studio). The docs also point to a dedicated cloud service, but this appears to have been [sunset](https://developer.ibm.com/docloud/try-docloud-free/). 

## Taps and streams

CPLEX has been around since 1988. I first used CPLEX through the academic initiative that IBM runs - granting unrestricted licenses for users at academic institutions. All you needed was a `.edu` email address and you were good to go. It seems to have go through a winding road since. 

Software is only worth it if the problem they solve is repeatable. CPLEX certainly is in that bucket. Over time, mature software products devolve into a complex mesh of functionality, versions, and modes. You know a product has been around a while if you see a matrix! This is partly driven by the business which is fond of putting taps on a stream. Clients may have new uses that need new functions. And product teams have competition that drives some of this.

In all this, the cloud increasingly seems like a barrier. Devs (and plebs) are spending considerable amounts of time to get at the technical realities, and once there, playing whack-a-mole for a reality that shifts over time. 