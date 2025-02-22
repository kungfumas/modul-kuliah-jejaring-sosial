# Assignment 02: Distances

## Materials

For this assignment you need the file [lesmiserables.gml](data/lesmiserables.gml) which contains the social network of *Les Misérables* and  [email-eu-core.txt](data/email-eu-core.txt) which contains a social network of e-mails.

Both should be considered as *undirected* graphs.

## Task

The task is to:

1. Compute all shortest-path distances. This is a matrix which, in position *(u,v)*, will have the length of the shortest path from *u* to *v*, 0 if *u=v*, and *infinity* if *u* and *v* are in different connected components.

2. In the network from *Les Misérables*, draw a network in which each node is labeled according to its distance from the character Cosette.

3. Compute diameter and effective diameter for both networks

4. Draw a histogram of distances

# Part 1: compute all shortest-path distances

## 1.0. Algorithm

We will use the [Floyd-Warshall algorithm](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm), which in pseudocode is the following:

    1 dist is a |V| × |V| matrix
    2 fill all positions of dist with ∞ (infinity)
    3 for each edge (u,v)
    4    dist[u][v] ← 1
    5    dist[v][u] ← 1
    6 for v from 0 to |V|-1
    7    dist[v][v] ← 0
    8 for k from 0 to |V|-1
    9    for i from 0 to |V|-1
    10      for j from 0 to |V|-1
    11         if dist[i][j] > dist[i][k] + dist[k][j]
    12             dist[i][j] ← dist[i][k] + dist[k][j]
    13         end if
    14      end for
    15   end for
    16 end for

:bomb: **Do not copy-paste an implementation found online**. If you do so, it will be counted as plagiarism and get a zero grade. Instead, directly translate from the provided pseudocode, we are providing step-by-step instructions next.

Imports that you will need:

```Python
import numpy as np
import networkx as nx
import matplotlib.pyplot as plt
```

## 1.1. Create a mapping from labels to node ids

Note that the algorithm uses a matrix in which rows and columns go from *0* to *N-1*. The size of the graph `g`, *N*, is simply `g.order()`.

Create a dictionary, name it `node2id`, that for every node in the graph `g` contains a number between `0` and `g.order()-1`. If you issue `print(node2id)`, you should see something like this:

```Python
{'Myriel': 0, 'Napoleon': 1, ... 'MlleVaubois': 75, 'MotherPlutarch': 76}
```

Check that in `nodeid` there should be as many keys as there are nodes in the graph, and the values should go from *0* to *g.order()-1* without repetitions.

## 1.2. Create the distance matrix

Create an initialize a matrix (steps 1-7 of pseudocode above).

* An empty *n x m* matrix is created with `dim = (n,m)` and then `matrix = np.empty(dim)`
* A matrix *matrix* can be filled with the number *number* using `matrix.fill(number)`
* Infinity is the constant `np.inf`
* An iterator over edges (two-element tuples) is created with `g.edges()`. The source of the edge is the first element `edge[0]` and the destination of the edge is the second element `edge[1]`. Both are strings/labels that you need to convert to identifiers.

If you issue `print(dist)` after initializing, you should see this:

```python
[[  0.   1.   1. ...,  inf  inf  inf]
 [  1.   0.  inf ...,  inf  inf  inf]
 [  1.  inf   0. ...,  inf  inf  inf]
 ...,
 [ inf  inf  inf ...,   0.  inf  inf]
 [ inf  inf  inf ...,  inf   0.  inf]
 [ inf  inf  inf ...,  inf  inf   0.]]
```

## 1.3. Run the algorithm

Now implement steps 8-16 of the pseudocode, it is straightforward but it is a good idea to print some progress report to know whether the algorithm is running or not. While running your algorithm, the output should look similar to this (here I have added a step numbered 77 even if the loop goes from 0 to 76, just to be able to print the satisfactory "100.0%"):

```python
k = 0/77 (0.0%)
k = 2/77 (2.6%)
...
k = 77/77 (100.0%)
```

If you issue `print(dist)` after running the algorithm, you should see something like this:

```python
[[ 0.  1.  1. ...,  3.  3.  4.]
 [ 1.  0.  2. ...,  4.  4.  5.]
 [ 1.  2.  0. ...,  3.  3.  4.]
 ...,
 [ 3.  4.  3. ...,  0.  3.  3.]
 [ 3.  4.  3. ...,  3.  0.  4.]
 [ 4.  5.  4. ...,  3.  4.  0.]]
```

How to check that your matrix is correct? Examine the graph and do by hand a few distance calculations between nodes. Then, compare what you obtain by looking at the graph with what the matrix says. The matrix should have in position *(i,j)* the undirected distance between the node with id *i* and the node with id *j*.

Remember you can draw the graph by doing:

```python
plt.figure(figsize=(5,5))
nx.draw_networkx(g)
```

Then, you can do `id1 = node2id["..."]` and `id2 = node2id["..."]` for two of the characters, and check if `dist[id1, id2]` is what you see when you draw the graph. This manual verification is doable in this small network but in larger networks will not be possible in general.

# Part 2: extract distances from Cosette

## 2.1. Extract a row of the table and save to disk

You want the row number `node2id["Cosette"]` of the matrix `dist`. Save a mapping from character names to distances from Cosette as a CSV file, it should look like this:

```
Name,DistFromCosette,NewLabel
Valjean,1,Valjean(1)
Fantine,2,Fantine(2)
...
Cosette,0,Cosette(0)
...
Listolier,3,Listolier(3)
```

Note that we have added a numerical attribute, with the distance from Cosette, and a new label.

The ordering of columns on this file does not matter.

## 2.2. Draw in Cytoscape

Draw this network Cytoscape using the `DistFromCosette` attribute as color or size, and `NewLabel` as the label for the nodes. You will need to import first the `.gml` file as a network, and then your `.csv` file as a table.

# Part 3: compute diameter and effective diameter

Now you need to create a vector with all pair-wise distances. Be careful (1) not to add self-loops, (2) not to add unreachable pairs with infinite distances, and (3) not to add the same edge twice. This vector (call it `distances`) should have length at most 2,926 *((N x N - N)/2)*.

For computing diameter you can use `np.max` and for computing effective diameter (90%) you can use `np.percentile`.

# Part 4: draw a histogram of distances

```python
hist, bins = np.histogram(distances, density=True, bins=np.arange(np.min(distances), np.max(distances)+2, 1.0))
plt.bar(bins[:-1], hist)
```
Remember to **include a title** and **label the axes** of this bar plot. In a histogram where the quantities are normalized, i.e., depicting a probability density function, the standard label for the y-axis is "probability".

# Deliver (groups of two)

A .zip or .tar.gz file containing your code and your report in PDF.

The code should be a single Python notebook (not multiple notebooks), in .ipynb format.

The report should be a PDF file (not .docx, not .odt). It should contain, for *Les Misérables*:

1. The full distance matrix (see below).
1. A graph generated in Cytoscape in which each node is labeled and colored/styled according to its distance from Cosette (e.g., label of node "Fantine" should be "Fantine(2)"). Include a legend for the colors.
1. Your brief (1-2 paragraphs) commentary on this distance matrix and graph.

And for both *Les Misérables* and the *Email-EU-Core* network:

1. The diameter (max), effective diameter (90% percentile), and median diameter (50% percentile) of both networks
1. The distance histograms of both networks
1. Your brief (1-2 paragraphs) commentary on each distance histogram
1. Your conclusion (1-2 paragraphs) comparing the two distance histograms

The report should end with the following statement: **We hereby declare that, except for the code provided by the course instructors, all of our code, report, and figures were produced by ourselves.**

## Visualizing the matrix

Your report should include the matrix of distances for *Les Misérables* only.

**Option 1:** the best option is to use `plt.imshow(matrix)` and `plt.colorbar()` to generate a visual representation of the matrix; see the [imshow documentation](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.imshow.html). Ensure the figure size is large enough so it looks nicely in your report.

**Option 2:** print it in the notebook, then copy it from there and paste it in your report.

**Option 3:** export it to a .txt file, open it with a text editor, and copy it from there, then paste it in your report.

If you use options 2 or 3, shrink the font size so it fits in a page: do not use a low-quality or poorly cropped screenshot. Make sure there is something that can be seen in the matrix. If numbers cannot be read, try other font faces, or replace the numbers by colors, or replace zeroes by blanks, or somehow make sure at least something about the distance matrix is understandable, even if not perfectly readable.
