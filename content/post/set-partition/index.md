---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Partitioning a set"
subtitle: ""
summary: "Integer programming approaches to set partitioning"
authors: []
tags: ["optimization", "integer programs", "column generation"]
categories: []
date: 2021-04-30T13:26:35+01:00
lastmod: 2021-04-30T13:26:35+01:00
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

In this post, we summarize the set partitioning problem, a widely applicable formalization that can be used in a variety of contexts. You are given a set of elements that need to be divided. Each division, or partition, has an associated cost that you seek to minimize. The set partitioning problem asks: what is the best way to partition such that the total cost can be minimized?

This abstraction applied to a broad range of cases. From cutting cake for your family (if you consider cost to decrease as you get a larger piece) to more industrial applications like scheduling and crew rostering. There is also a large literature on the problem. A [survey](https://doi.org/10.1137/1018115) paper was published way back in 1976 and reference on [applications](https://doi.org/10.1007/978-1-4613-0303-9_9) in 1998.

## The problem

In the basic setting, we have a set $X$ of elements and a collection of possible partitions $K=\{k_1, k_2,...,k_m\}$ and a cost function $c(k_j)$ associated with each partition. $K$ is a very large collection as there are several ways to divide a set. We define a membership matrix $A = [a_{ij}]$ where $a_{ij}=1$ if element $i$ from $X$ is in the partition $k_j$ and $0$ otherwise. We denote $x_j$ as a decision variable to decide if partition $k_j$ is employed/used or not. With these definitions, the set partitioning problem can be written as:

$$\begin{aligned}
\min_{x} \quad & \sum_{j=1}^{n} c(k_j) x_j\\\\
\textrm{s.t.} \quad & \sum_{j=1}^{m} a_{ij}x_j=1 \quad \forall i=1,...,n\\\\
  &x_j\in \{0,1\}    \\\\
\end{aligned}
$$

The program seeks to find a set of partitions that divide the input feature space such that every element is covered by exactly one partition. (If we allow for overlap between partitions, we could use a coverage type constraint $\sum a_{ij}x_j \ge 1$ as well). The objective is to find a partitioning that minimizes the total cost of the partition.

## Column generation

The model above can not typically be solved directly for realistic sized instances. This is because the collection $K$ is very large. For cases where the number of columns is much greater than the rows, i.e. $m\gg n$, we can start with a restricted set of partitions and generate partitions on the fly as needed. The method of column generation is useful here to do this in a principled manner and can work as follows:

1. **Restricted master problem**: We start with a limited set of partitions $J\, J\subset K$ and relax integral conditions to solve a linear program. Denote $\pi_i$ as the dual variables associated with each equality constraint. We solve this limited problem and recover the values of $\pi_i\, i=1, ..., n$

2. **Generate new partitions**: Formulate a pricing problem that seeks to find partition that can have the largest benefit, in terms of the real objective.  The general approach is to find a column (i.e. partition) that has the largest negative [reduced cost](https://en.wikipedia.org/wiki/Reduced_cost). If $z_i$ as a binary decision variable that denotes if element $i$ is in the new partition, the problem of finding a new partition can be stated as

$$\begin{aligned}
\min_{z} \quad & c(z) - \sum_{i=1}^{n} \pi_i z_i\\\\
  &z_i\in {0,1}    \\\\
\end{aligned}
$$

where $c(z)$ denotes the cost of the chosen partition. Depending on your problem specifics and the nature of this function the manner in which you solve this would vary. Looking for the column with the greatest negative reduced cost is a heuristic step and can be replaced by other procedures to generate new candidate partitions if you have some insights that help you to do so.

## Reference implementation

To build some intuition, let's look at a reference implementation for a trivial toy example. The objective is to find the optimal partition of the English alphabet. The cost of each partition is `1.0` unit - so the optimal partition is trivially a set with all 26 letters. However, we start the program with smaller partitions to demonstrate that the column generation ends up with the optimal solution.

Denote `X` is the set of elements and `K` is a initial set of partitions. These can be arbitrary subsets. 
```python
import string
X = list(string.ascii_lowercase)
K = [X[i:i + 5] for i in range(0, len(X), 5)]
```

Let's start with the (restricted) master program which is formulated as a linear program. 
```python
def master(X: List, K: List[List]) -> cplex.Cplex:
    """ Restricted master problem """

    prob = cplex.Cplex()
    numvar = len(K)

    names = list(map(func, K))
    var_type = [prob.variables.type.continuous] * numvar
    prob.variables.add(names=names,
                    lb=[0.0] * numvar,
                    ub=[1.0] * numvar,
                    types=var_type)
    prob.objective.set_sense(prob.objective.sense.minimize)
    prob.objective.set_linear([(n, cost(kj)) for n, kj in zip(names, K)])

    lhs = []
    for i in X:
        coeffs = [1.0 if i in kj  else 0.0 for kj in K]
        lhs.append(cplex.SparsePair(names, coeffs))
    prob.linear_constraints.add(lin_expr=lhs,
                                rhs=[1.0] * len(lhs),
                                senses=['E'] * len(lhs),
                                names=X)
    prob.set_problem_type(cplex.Cplex.problem_type.LP)
    print(f"{len(lhs)} constraints and {len(names)} variables.")

    return prob
```

The column generation procedure will solve the master at every iteration. Retrieve the dual variable values and then use that in a pricing step to determine a new partition to add (more on this next). If the reduced cost is negative then the objective will improve by the addition. If not, then no new partition has been found to improve the solution and we can terminate. 
```python
while True:
    p.solve()
    pi = [-p for p in p.solution.get_dual_values()]

    # Generate new partitions/columns
    K_new, rc_new = pricing(X, pi)

    # termination check
    if rc_new <= -eps:

        newvar = func(K_new)
        print(f"Iteration {i}: Adding '{''.join(K_new)}' column with rc: {rc_new}.")

        p.variables.add(names=[newvar],
                        lb=[0.0], ub=[1.0],
                        types=[p.variables.type.continuous])
        p.objective.set_linear(newvar, cost(K_new))
        for c in K_new:
            p.linear_constraints.set_coefficients(str(c), newvar, 1.0)

        p.set_problem_type(cplex.Cplex.problem_type.LP)
        i+=1
    else:
        print("No improvement partition found - Terminating column generation.")
        break
```

Now on to the pricing step - the step where new columns are generated. Here is where, in actual applications, the heavy lifting would happen. For this simple example, we can just generate partitions with negative duals and compute the reduced cost of this partition. In many cases, this can be a combinatorial problem (resulting in another integer program) or have some structure that can be leveraged (e.g. a shortest path or allow for dynamic programming solutions). 

```python
def pricing(X: List, pi: List, ncols: int=5) -> Tuple[List, float]:
    """ Heuristic to generate new partitions """
    
    # set of elements that have negative dual variables
    K_new = [x for p, x in sorted(zip(pi, X), reverse=True) if p < 0.0]
    
    # reduced cost of new partition
    rc_new = rc(K_new, pi, X)

    return sorted(K_new), rc_new
```
## Putting it together

Running this together as shown in this [gist](https://gist.github.com/rahulnair23/b3be98553df6637f7b7fd4490e80991d) gives the following output:

```bash
26 constraints and 3 variables.
Iteration 0: Adding 'aku' column with rc: -2.0.
Iteration 1: Adding 'bku' column with rc: -3.0.
Iteration 2: Adding 'ablu' column with rc: -2.5.
Iteration 3: Adding 'abklv' column with rc: -2.25.
Iteration 4: Adding 'ckluv' column with rc: -2.2857142857142856.
Iteration 5: Adding 'abckmuv' column with rc: -2.307692307692308.
Iteration 6: Adding 'abckluw' column with rc: -2.736842105263158.
Iteration 7: Adding 'aclmvw' column with rc: -6.777777777777778.
Iteration 8: Adding 'bdlmuvw' column with rc: -3.2222222222222223.
Iteration 9: Adding 'cdkvw' column with rc: -5.000000000000001.
Iteration 10: Adding 'acdklmuvx' column with rc: -2.4818181818181815.
Iteration 11: Adding 'abdkmwx' column with rc: -5.540540540540541.
Iteration 12: Adding 'bclmx' column with rc: -8.879999999999999.
...
Iteration 174: Adding 'abcdefghijklmnopqrstuwxy' column with rc: -0.10279605263157898.
Iteration 175: Adding 'abcdefghijklmnopqrstuvwxyz' column with rc: -0.08656995788488508.
Optimal value: 1.0
Partition: 'abcdefghijklmnopqrstuvwxyz' with cost: 1.0.
```
Since each partition costs one unit, the optimal partition is one single partition.

The number of possible partitions of a set is known as the [Bell number](https://en.wikipedia.org/wiki/Bell_number). For the 26 alphabet, the total number of possible partitions is very large (`49631246523618756274` as per `sympy.bell` in Python). In this example the procedure evaluated 175 partitions before arriving at the optimal one.

