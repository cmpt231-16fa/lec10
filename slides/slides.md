# CMPT231
## Lecture 10: ch18
### Graph Algorithms

---
## Devotional

---
## Outline for today
+ Intro to **graph** algorithms
  + *Edge list*, adjacency list, adjacency matrix
+ **Breadth-first** graph traversal
+ **Depth-first** graph traversal
  + **Parenthesis** structure
  + Edge **classification**
  + Topological **sort**
  + Finding **strongly-connected** components
+ Minimum **spanning** trees

---
## Intro to graph algorithms
+ Representing **graphs**: *G* = (*V*, *E*)
  + *V*: **vertices** / nodes
    + storage: *array*, *linked-list*, etc.
  + *E*: **edges** connecting vertices
    + *directed* or *undirected*
    + storage: edge *list*, adjacency *matrix*, etc.

| graph | vertex | edge |
|-------|--------|------|
| air transport |  airport | flight path |
| social | person | friendship/relationship |
| internet | computer | network connection |
| finance | stock/asset | transaction |
| neural net | neuron | synapse |
| protein net | protein | protein-protein interaction |

---
## Examples

>>>
TODO: proteins
TODO: internet
TODO: politics
TODO: flowingdata for more

---
## Representing edges
+ **Edge list**: array/list of *(u,v)* pairs of nodes
  + [ (1,2), (1,3), (2,4) ]
+ **Adjacency list**: indexed by *start* node
  + [ {1: [2, 3]}, {2: [1, 4]}, {3: [1]}, {4: [2]} ]
  + How to find the (out)-*degree* of each vertex?
+ **Adjacency matrix**: boolean *|V|* x *|V|* matrix
  + *A[i,j]* = 1 iff *(i,j)* is an edge
  + What about *weighted* graphs?
  + \` ( (0, 1, 1, 0), (1, 0, 0, 1), (1, 0, 0, 0), (0, 1, 0, 0) )\`

---
## Graph traversal: breadth-first
+ **Traversal**: visits each node exactly *once*
+ BFS: overlay a **breadth-first tree**
  + Choose a *start* (root) node
  + *Path* in tree = *shortest* path from root
  + BFS tree not necessarily *unique*
+ But the graph G could have **loops**
  + Need to **track** which nodes we've seen
+ Assign **colours** to nodes as we traverse graph:
  + *White*: unvisited
  + *Grey*: on border (some unvisited neighbours)
  + *Black*: no unvisited neighbours
+ Use **FIFO** queue to manage *grey* nodes

---
## BFS algorithm
+ In: **vertex** list, **adjacency** (linked) list, **start** node
+ Out: **modify** vertex list, adding *parent* pointers

<div class="imgbox"><div>
<pre><code data-trim>
def BFS( V, E, start ):
  init V all white and NULL parent
  start.colour = grey
  init FIFO: Q.push( start )
  while Q.notempty():
    u = Q.pop()
    for v in E.adj[ u ]:
      if v.colour == white:
        v.colour = grey
        v.parent = u
        Q.push( v )
    u.colour = black
</code></pre>
<strong>Complexity?</strong>
</div><div>
![BFS](static/img/bfs.svg)
</div></div>

>>>
TODO: fig

---
## Outline

---
## Depth-first search
+ First explore as **deep** as we can
  + **Backtrack** to explore other paths
  + **Recursive** algorithm
+ **Colouring**: *white* = undiscovered, *grey* = discovered, <br/>
  *black* = finished (visited all neighbours)
+ Add **timestamps** on *discover* and *finish*
+ Overlay a **forest** on the graph
  + **Subtree** at a node = all nodes visited between <br/>
    this node's *discovery* and *finish*

---
## DFS algorithm

<div class="imgbox"><div style="flex:2">
<pre><code data-trim>
def DFS( V, E ):
  init V all white and NULL parent
  time = 0
  for u in V:     # why loop over all V?
    if u.colour == white:
      DFS-Visit( V, E, u )
</code></pre>
<pre><code data-trim>
def DFS-Visit( V, E, u ):
  time++
  u.discovered = time
  u.colour = gray
  for v in E.adj[ u ]:
    if v.colour == white:
      v.parent = u
      DFS-Visit( V, E, v )
  u.colour = black
  time++
  u.finished = time
</code></pre>
</div><div>
![DFS](static/img/dfs.svg)
</div></div>

>>>
TODO: fig

---
## DFS: parenthesis structure
+ Each node's **subtree** is visited <br/>
  between its *discovery* and *finish* times
+ **Print** a \`(\_u\` when we *discover* node *u*
  + print a \`)\_u\` when we *finish* it
+ Output is a valid **parenthesisation**:
  + e.g., \`(.\_u (.\_v (.\_w )\_w )\_v (.\_x (.\_y )\_y )\_x )\_u (.\_z )\_z\`
  + But not \`(.\_u (.\_v )\_u )\_v\`
+ The (*discover*, *finish*) intervals for any two vertices are
  + Either completely **disjoint**
  + Or one **contained** in the other

---
## DFS: white-path theorem
+ The (*d*, *f*) interval for *v* is **contained** in *u* <br/>
  &hArr; *v* is a **descendant** of *u* in the DFS
  + i.e., *u.d* < *v.d* < *v.f* < *u.f*

<div class="imgbox"><div><ul>
<li> <strong>White-path<strong> theorem: <ul>
  <li> <em>v</em> is a <strong>descendant</strong> of <em>u</em> in the DFS &hArr; </li>
  <li> When <em>u</em> is <em>discovered</em>, there is <br/>
    a <strong>path</strong> <em>u</em> &rarr; <em>v</em> all of <em>white</em> vertices </li>
  </ul></li>
</ul></div><div>
![white path](static/img/white-path.svg)
</div></div>

>>>
TODO: fig

---
## DFS: edge classification
+ All **edges** in a graph are either
  + **Tree** edges: in the *DFS forest*
  + **Back** edges: up to *ancestor* in same DFS tree (incl *self-loop*)
  + **Forward** edges: down to *descendant*
  + **Cross** edges: *different* subtrees or DFS trees
+ Lemma (*22.11*): for directed graphs, <br/>
  **acyclic** &hArr; no **back** edges

![edge classification](static/img/edge-class.svg)

>>>
TODO: fig

---
## Outline

---
## DFS: topological sort
+ **Order** vertices such that for every edge *u* &rarr; *v*, <br/>
  *u* comes **before** *v* in the sort
  + Assumes **no cycles**! (i.e., *DAG*: directed acyclic)
+ **Applications**: *dependency* resolution, *compiling* files, <br/>
  task planning / *Gantt* chart
+ Tweak **DFS**: when each vertex is *finished*, <br/>
  insert it at the *head* of a linked list
  + Sort in **decreasing** order of *finish* time
+ *DFS* might not be **unique**, so <br/>
  *topological sort* might not be unique

![DFS](static/img/dfs.svg)

---
## Topological sort: example
+ Proof of **correctness**: \`(u,v) in E => v.f < u.f\`
+ When DFS explores *(u,v)*, what **colour** is *v*?
  + if **gray**: so *v* is an **ancestor** of *u*
    + So *(u,v)* is a **back** edge
    + So graph has a **loop** (disallowed)
  + if **white**: becomes a **child**: *u.d* < *v.d* < *v.f* < *u.f*
  + if **black**: *v* **done**, but *u* not done yet: *v.f* < *u.f*

![topological sort](static/img/topo-sort.svg)

---
## DFS: connected components
+ Largest **completely-connected** set of vertices:
  + Every *vertex* in the component has a **path**
    to every *other* vertex in the component
+ Algorithm:
  + Compute *DFS* to find **finishing** times
  + **Transpose** the graph: *reverse* all edges
  + Compute *DFS* on transposed graph
    + Start at vertex that finished **last** in orig DFS
  + Each **tree** in final DFS is a separate **component**

![components](static/img/conn-comp.svg)

---
## Connected components
<div class="imgbox"><div><ul>
<li> (a) <strong>Original</strong> graph and <em>DFS</em> (shaded) </li>
<li> (b) <strong>Transpose</strong> graph and its <em>DFS</em> <ul>
  <li> Starting at <em>b</em> (finished <strong>last</strong>)</li>
  </ul></li>
<li> (c) Coalesce vertices into <strong>component graph</strong> </li>
</ul></div><div style="flex:2">
![Fig, components](static/img/Fig22-3.svg)
</div></div>

---
## Outline

