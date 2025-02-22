# Practice Session 05: Graph generation

In this session we will learn to use [NetworkX](https://networkx.github.io/), a Python package, and we will write code to create random graphs and preferential attachment graphs.

:warning: These graph generators are implemented in the NetworkX library and in other places online. **Do not copy those implementations:** they reproduce the same kinds of graph but follow a design that is different from what we describe here.

# 0. Preliminaries

## 0.1. Imports

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
```

## 0.2. Creating a small graph

Small graphs can be easily created programmatically in Python with NetworkX.

* To create a graph, you use either `networkx.Graph` or `networkx.DiGraph`, which return an undirected an directed graph respectively.
* To add a node to a graph *g*, you use `g.add_node(u)`, where *u* is the name of the node.
* To add an edge to a graph *g*, you use `g.add_edge(u, v)`, where *u* is the name of the source of the edge, and *v* the name of the destination of the edge.

Example:

```python
g = nx.Graph()
g.add_node(0)
g.add_node(1)
g.add_edge(0, 1)
```

To draw the graph, you can use:

```python
nx.draw_networkx(g)
```

You can have more control over the visualization of the graph, such as setting the figure size, removing the axis, using a particular layout algorithm, or changing the node size or color:

```python
plt.figure(figsize=(3,3))
plt.axis('off')
pos=nx.spring_layout(g)
nx.draw_networkx(g, pos, with_labels=True, node_size=500, node_color='yellow')
```

## 0.3 Plotting degree distributions

The following function might be useful to plot degree distributions that are necessary for the report (remember to add: `from collections import OrderedDict`):

```python
def plot_degree_distribution(g):
    degree_dict = dict(g.degree())
    degree_ordered = OrderedDict(sorted(degree_dict.items(), key=lambda x: x[1], reverse=True))
    degree_sequence = list(degree_ordered.values())
    prob, bin_edges = np.histogram(degree_sequence, bins=range(1,np.max(degree_sequence)+2), density=True)
    plt.loglog(bin_edges[:-1], prob, '.', marker='x')
    plt.title("Probability density function")
    plt.xlabel("degree")
    plt.ylabel("probability")
    plt.show()
```

# 1. Generate an ER graph

To generate an ER graph, you need a function to perform Bernoulli trials. Use the following function, which return `True` with probability *p*, and `False` with probability *1-p*:

```python
def flip_coin(p):
    if np.random.random() < p:
        return True
    else:
        return False
```

Now, write function `generate_random_graph(N, p)`, that:

1. Creates an empty graph
1. Adds N nodes to this graph
1. For each pair *(u,v)* of nodes:
   1. With probability *p*, adds an edge between *u* and *v*
1. Returns the graph

The *N* nodes in this graph will be numbered from *0* and *N-1*. In Python, to iterate between `i=0` and `i=N-1`, you do:

```python
for i in range(0, N):
    ...
```

Your function should be called with `g = generate_random_graph(N, p)`. Use this function to generate and visualize a few graphs (do some tests with 100 nodes or so and *p=0.005*, *p=0.01*, *p=0.02*, etc.)

Create another function `print_er_statistics(g,p)` that given an ER graph and a probability *p* prints its actual average degree *&lt;k&gt;* and its expected average degree *p(N-1)*. The degree of node *u* in graph *g* is `g.degree(u)`. The number of nodes of the graph *g* is `len(g.nodes())`.

[**REPORT**] Draw five graphs with *N=800* and *p=0.0005, 0.001, 0.002, 0.005, 0.01*. For each graph, include its drawing, its degree distribution, its average degree, and its expected average degree. Include one brief commentary for all the ER graphs, you do not need to comment on each of the five graphs individually.

# 2. Generate a BA graph

Start by creating an auxiliary function that selects *m* target nodes in a graph *g*, with probabilities proportional to the degrees of the nodes. Below is a skeleton

```python
def select_targets(g, m):

    # Check if feasible
    if len(g.nodes()) < m:
        raise ValueError('Graph has less than m nodes')

    # Compute sum of degree
    sum_degree = 0

    # YOUR CODE HERE: COMPUTE SUM OF DEGREE OF NODES

    if sum_degree == 0:
        raise ValueError('Graph as no edges')

    # Compute probabilities
    probabilities = []
    for u in g.nodes():
        # YOUR CODE HERE: COMPUTE PROBABILITY OF SELECTING NODE u
        # THEN APPEND IT TO probabilities USING probabilities.append(...)

    # Sample with replacement
    selected = np.random.choice(g.nodes(), size=m, replace=False, p=probabilities)

    return selected
```

(The function `numpy.random.choice` in the above function is used to sample without replacement *m* elements from an array of nodes.)

Now, create a function `generate_preferential_attachment_graph(N, m0, m)` that:

1. Creates an empty graph
1. Adds nodes numbered from *0* to *m<sub>0</sub> - 1* to the graph
1. Links node *0* to nodes *1, 2, 3, ..., m<sub>0</sub> - 1*
1. For every node *u* numbered from *m<sub>0</sub>* to *N - 1*
   1. Select *m* targets for this node using `select_targets`
   1. Add node *u* (remember to select targets **before** adding the node *u*)
   1. Lnk each node *u* to each of the *m* targets
1. Returns the graph

Do small experiments with, e.g., *N=100, m<sub>0</sub>=5, m=5* or *N=500, m<sub>0</sub>=2, m=1*.

[**REPORT**] Draw 2 preferential attachment (BA) graphs with *N=2100, m<sub>0</sub>=5, m=1* and *N=2000, m<sub>0</sub>=2, m=2*. For each graph, include their drawing and their degree distribution, plus a brief commentary for each of the two graphs.

# DELIVER (individually)

Deliver a zip file containing:

* Your code as a Python notebook (a `.ipynb` file).
   * Remove all unnecessary elements
   * Add comments when needed
   * Use a single file with multiple cells.
* A three-page PDF report containing elements marked [**REPORT**]:
   * On pages one and two, the 5 random (ER) graphs.
   * On page three, the 2 preferential attachment (BA) graphs.

Tip: In all the graph drawings of your report use options `with_labels=False, node_size=10` (you can play with different values for `node_size`)

# References

* [NetworkX documentation](https://networkx.github.io/)
* [Complex Network Analysis in Python](https://www.amazon.com/gp/product/1680502697/) (2018) by Dmitry Zinoviev. Also [available as an e-book](https://upfinder.upf.edu/iii/encore/record/C__Rb1557007?lang=cat) for UPF students.
* [Social network analysis with NetworkX](https://blog.dominodatalab.com/social-network-analysis-with-networkx/) by Manojit Nandi
* [NetworkX: Network analysis with Python](https://www.cl.cam.ac.uk/~cm542/teaching/2010/stna-pdfs/stna-lecture8.pdf) by Salvatore Scellato
