---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Maximum weighted cliques in a graph"
subtitle: ""
summary: ""
authors: []
tags: ["optimization", "graph theory", "lazy constraints"]
categories: []
date: 2020-06-05T10:44:59+01:00
lastmod: 2020-06-05T10:44:59+01:00
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

Recently, I had the need to compute maximum weighted cliques on very dense large graphs. This is a well studied problem, and a nice [survey paper](https://doi.org/10.1007/BF01098364) from 90's by Pardalos and Xue gives a good overview of approaches.


## The problem

We are given a graph $G = (V, E)$ which is a set of vertices $V$ and edges $E$. Each vertex has an associated weight $d_i\, \forall i\in V$. A *clique* $C$ is a subset of vertices, where all vertices are pairwise connected. A *maximum* clique is a clique that has largest weight.

A related notion is of an *independent set*, which is a subset of vertices $V$ that are pairwise disconnected. A maximum independent set is similarly an independent set with the largest weight. 

## Solutions

For general graphs, finding the maximum cliques is a hard problem (NP-complete). An integer programming approach that involves edges can be written as:

$$\begin{aligned}
\max_x & \sum_{i\in V} d_i x_{i}\\\\
\mbox{s.t.} \qquad & x_i + x_j \le 1 \\quad \forall (i, j) \in \bar{E} \\\\
& x_i \in {0, 1} \qquad \forall i\in V
\end{aligned}
$$

where $\bar{E}$ is called the complement edge set that is a set of edges that are missing from the original graph $G$. All the constraint $x_i + x_j \le 1$ says is if the edge $(i, j)$ is missing then only one node, either $i$ or $j$, can be in the clique. The objective seeks to maximize the weighted of selected nodes.

As pointed out in the [paper](https://doi.org/10.1007/BF01098364) this formulation isn't useful in practice on account of two problems. First, the linear relaxation, one where the integrality constraints $x_i \in {0, 1}$ is omitted, gives a poor bound. The second is on account of symmetry. Symmetry arises in this context as vertices with the same weight are indistinguishable. So several configurations result in the same optimal solution. Why is this bad? The branch and cut tree cannot prune the search tree as solutions are in various parts of it. One way to remove symmetry is to do lexicographical ordering. An arbitrary order is imposed via additional constraints that cuts of several optimal solutions knowing that at least one optimal solution is valid. There are other methods as well, such as [isomorphic pruning](https://doi.org/10.1007/s10107-002-0358-2), and [orbitopal fixing](https://doi.org/10.1016/j.disopt.2011.07.001), 

## An alternative formulation

The notion of the complement edge set $\bar{E}$ can be strengthened using independent sets. If you know an independent set, a clique can contain at most one vertex from such a set. To write this as a constraint, one would need to consider *maximum* independent sets, lest you allow the omitted vertices to be included in the clique. Further, you would need to look at all maximum independent sets that arise in $G$. Assume for a moment that the set of maximum independent sets $\mathbb{S}$ is known. Then the problem of finding the maximum weighted clique can be written as

$$\begin{aligned}
\max_x & \sum_{i\in V} d_i x_{i}\\\\
\mbox{s.t.} \qquad & \sum_{i\in I} x_i \le 1 \\quad \forall I \in \mathbb{S} \\\\
& x_i \in {0, 1} \qquad \forall i\in V
\end{aligned}
$$

The objective is the same as before - maximize the total weight of selected nodes. The constraints, one for each maximum independent set, allows only one vertex into the solution. The constraints are tighter than the previous formulation implying that the linear relaxation gives a better bound. For some classes of graphs, namely perfect graphs, omitting the integrality constraints will directly give you integral solutions! The problem however is that there are now an exponential number of constraints (set $\mathbb{S}$ is very large).

One mechanism to deal with too many constraints is use lazy constraints. The idea is to start the optimization with a small set of constraints and then add *relevant* constraints from the large pool as you go along. The prerequisite is however that such relevant constraints can be generated efficiently. 

How would this work? A sketch of the solution algorithm looks like this. 

1. Use a heuristic procedure to generate a set of maximum independent sets ($S$)
2. Solve a linear relaxation on this limited set ($S \subseteq \mathbb{S}$)
3. Based on solution, identify new maximum independent sets that would cut the current solution
4. If no such sets exists, we are done (current solution gives the maximum weighted clique)
5. If there are, then add to the constraints and goto step 2.

## Reference implementation

To test this, we use the `networkx` library for graphs and the CPLEX solver. We use one of the many generators for a test graph with a known number of cliques.

```python
from networkx import nx
G = nx.generators.ring_of_cliques(6, 3)
nx.draw(G, with_labels=True)
```
gives you this:

{{< figure src="graph.png" title="An test graph with known max cliques" lightbox="true" >}}

#### (Step 1) Generating maximum independent sets

Here we use the greedy heuristic implementation as shown in this [blog]( https://kmutya.github.io/maxclique/):

```python
def greedy_init(G):
    """
    https://kmutya.github.io/maxclique/
    """
    n = G.number_of_nodes()  # Storing total number of nodes in 'n'
    max_ind_sets = []  # initializing a list that will store maximum independent sets
    for j in G.nodes:
        R = G.copy()  # Storing a copy of the graph as a residual
        neigh = [n for n in R.neighbors(j)]  # Catch all the neighbours of j
        R.remove_node(j)  # removing the node we start from
        iset = [j]
        R.remove_nodes_from(neigh)  # Removing the neighbours of j
        if R.number_of_nodes() != 0:
            x = get_min_degree_vertex(R)
        while R.number_of_nodes() != 0:
            neigh2 = [m for m in R.neighbors(x)]
            R.remove_node(x)
            iset.append(x)
            R.remove_nodes_from(neigh2)
            if R.number_of_nodes() != 0:
                x = get_min_degree_vertex(R)

        max_ind_sets.append(iset)

    return(max_ind_sets)


def get_min_degree_vertex(Residual_graph):
    '''Takes in the residual graph R and returns the node with the lowest degree'''
    degrees = [val for (node, val) in Residual_graph.degree()]
    node = [node for (node, val) in Residual_graph.degree()]
    node_degree = dict(zip(node, degrees))
    return (min(node_degree, key=node_degree.get))
```

#### (Step 2) The optimization model

Now we define the optimization model as a linear program using the CPLEX python API. We initialize the set of constraints based on the greedy heuristic.

```python
import cplex

prob = cplex.Cplex()
numvar = len(G.nodes)

def func(x): return "x"+str(x)

names = list(map(func, G.nodes))
var_type = [prob.variables.type.continuous] * numvar
prob.variables.add(names=names,
                   lb=[0.0] * numvar,
                   ub=[1.0] * numvar,
                   types=var_type)
prob.objective.set_sense(prob.objective.sense.maximize)
prob.objective.set_linear([(n, 1.0) for n in names])
lhs = []

# Call the greedy heuristic to generate a starting set of independent sets
mis = greedy_init(G)

for iset in mis:
    con_vars = [func(i) for i in iset]
    coeffs = [1.0] * len(con_vars)
    lhs.append(cplex.SparsePair(con_vars, coeffs))
prob.linear_constraints.add(lin_expr=lhs,
                            rhs=[1.0] * len(lhs),
                            senses=['L'] * len(lhs))
print("Constraint: Maximum independent set. ({} constraints)".format(len(mis)))
```

To solve this model, we would use the following snippet to execute the model and return the final solution. 

```python
prob.solve()
status = prob.solution.status[prob.solution.get_status()]
print("Status:{}".format(status))

if prob.solution.get_status() in [101, 105, 107, 111, 113]:
    # Optimal/feasible, so get the solution
    print("Solution value: ")
    print(prob.solution.get_objective_value())

    # get the configuration
    x_res = prob.solution.get_values(names)
    for x_name, val in zip(names, x_res):
        if val > 0:
            print(x_name, val)
```

#### (Step 3) Generate lazy constraints

We've not managed the lazy constraints yet. To do this we will use a CPLEX `Callback`. The `LazyConstraintCallback` is called each time an optimal or integral solution is found. The implementation looks like this. 

To find new independent sets, we take the solution (potentially fractional) and use a greedy heuristic to first generate an independent set on the induced subgraph of the solution. We then expand on the independent set for the entire graph using another `greedy_expand` procedure which uses the same logic as `greedy_init` to grown the independent set. 

If there are no additional independent sets, no constraints are added and the solver terminates. 

```python
class LazyCallback(LazyConstraintCallback):
    """Lazy constraint callback to generate maximum independent sets on the fly.

    There are too many such constraints to make them all available to 
    CPLEX right away - and in any case, very few of them are valid.

    So generate them on the fly.
    """

    # Callback constructor.
    #
    # Any needed fields are set externally after registering the callback.
    def __init__(self, env):
        super().__init__(env)

    def __call__(self):

        values = self.get_values(self.names)

        # Any node with non-zero value is considered as part of the set
        curr_solution = [int(name[1:]) for name, val in zip(self.names, values) if val >= 0.001]
        print("Current solution: ", curr_solution)

        # Look to generate all independent sets that would cut off the (fractional)
        # value. To do this, first induce a subgraph - and for each node, built a
        #
        subG = self.G.subgraph(curr_solution)
        sub_ind_set = greedy_init(subG)
        max_ind_sets = [greedy_expand(self.G, sset) for sset in sub_ind_set]

        for iset in max_ind_sets:

            con_vars = [func(i) for i in iset]
            coeffs = [1.0] * len(con_vars)
            lhs = cplex.SparsePair(con_vars, coeffs)
            self.add(constraint=lhs, rhs=1.0, sense='L')

```

The callback must be registered with the problem instance and any variables passed as attributes as so:

```python
from cplex.callbacks import LazyConstraintCallback

# register callbacks to generate additional independent sets on the fly
lazycb = prob.register_callback(LazyCallback)

# pass any arguments as class attributes
lazycb.names = names
lazycb.G = G
```

For completeness, here is what the `greedy_expand` function does
```python
def greedy_expand(G, init_set):

    R = G.copy()
    neigh = [n for i in init_set for n in R.neighbors(i)]
    R.remove_nodes_from(init_set)
    R.remove_nodes_from(neigh)
    if R.number_of_nodes() != 0:
        x = get_min_degree_vertex(R)
    while R.number_of_nodes() != 0:
        neigh2 = [m for m in R.neighbors(x)]
        R.remove_node(x)
        init_set.append(x)
        R.remove_nodes_from(neigh2)
        if R.number_of_nodes() != 0:
            x = get_min_degree_vertex(R)

    return init_set
  ```

#### Putting it all together

Running this all together as shown in this [gist](https://gist.github.com/rahulnair23/ef3c14a3f08afdf0840459e10969eda8) 
{{<gist rahulnair23 ef3c14a3f08afdf0840459e10969eda8>}}

gives the following output:

```bash
Constraint: Maximum independent set. (18 constraints)
Version identifier: 12.10.0.0 | 2019-11-26 | 843d4de
CPXPARAM_Read_DataCheck                          1
Warning: Control callbacks may disable some MIP features.
Lazy constraint(s) or lazy constraint/branch callback is present.
    Disabling dual reductions (CPX_PARAM_REDUCE) in presolve.
    Disabling non-linear reductions (CPX_PARAM_PRELINEAR) in presolve.
    Disabling presolve reductions that prevent crushing forms.
         Disabling repeat represolve because of lazy constraint/incumbent callback.
Tried aggregator 1 time.
MIP Presolve eliminated 6 rows and 0 columns.
Reduced MIP has 12 rows, 18 columns, and 72 nonzeros.
Reduced MIP has 0 binaries, 0 generals, 0 SOSs, and 0 indicators.
Presolve time = 0.00 sec. (0.02 ticks)
MIP emphasis: balance optimality and feasibility.
MIP search method: traditional branch-and-cut.
Parallel mode: none, using 1 thread.
Root relaxation solution time = 0.00 sec. (0.01 ticks)
Current solution:  [5, 7, 8, 14, 16, 17]
Current solution:  [4, 10, 11, 13]
Current solution:  [0, 6, 8, 10, 11]
Current solution:  [4, 6, 8, 10, 11, 16]
Current solution:  [0, 2, 4, 5, 6, 8]
Current solution:  [6, 7, 8]
Current solution:  [6, 7, 8]

        Nodes                                         Cuts/
   Node  Left     Objective  IInf  Best Integer    Best Bound    ItCnt     Gap         Variable B NodeID Parent  Depth

*     0     0      integral     0        3.0000        6.0000        8  100.00%                        0             0
Elapsed time = 0.02 sec. (0.16 ticks, tree = 0.00 MB, solutions = 1)

User cuts applied:  17

Root node processing (before b&c):
  Real time             =    0.02 sec. (0.16 ticks)
Sequential b&c:
  Real time             =    0.00 sec. (0.00 ticks)
                          ------------
Total (root+branch&cut) =    0.02 sec. (0.16 ticks)
Status:MIP_optimal
Solution value: 
3.0
x6 1.0
x7 1.0
x8 1.0
```

This identified one of the 3-vertex cliques, which is the maximum. The program started with 18 maximum independent sets generated greedily. It generated a further 17 user cuts one for each new independent set that were constructed on the fly. For a graph with $n$ nodes, there can be at most $3^{n/3}$ maximum independent sets although most have far fewer. For our 18 node graph, that would be 729 sets. In this greedy solution method, we got away with generating just 35.