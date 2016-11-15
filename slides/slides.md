<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
# CMPT231
## Lecture 10: ch18
### Graph Algorithms

<div class="caption">
Some material from [Sedgewick + Wayne, "Algorithms"](http://algs4.cs.princeton.edu/)
</div>

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Romans 10:13-15 <span class="ref">(NIV)</span>
"Everyone who **calls** on the **name of the Lord** will be saved."

How, then, can they **call** on <br/>
the one they have not **believed** in? <br/>
And how can they **believe** in <br/>
the one of whom they have not **heard**? <br/>
And how can they **hear** <br/>
without someone **preaching** to them? <br/>
And how can anyone **preach** unless they are **sent**?

As it is written: <br/>
“How beautiful are the **feet** of those who bring **good news**!"

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to **graph** algorithms
  + *Applications* and typical *problems*
  + *Edge list*, adjacency list, adjacency matrix
+ **Breadth-first** graph traversal
+ **Depth-first** graph traversal
  + **Parenthesis** structure
  + Edge **classification**
  + Topological **sort**
  + Finding **strongly-connected** components

---
## Intro to graph algorithms
+ Representing **graphs**: *G* = (*V*, *E*)
+ *V*: **vertices** / nodes
  + storage: *array*, *linked-list*, etc.
+ *E*: **edges** connecting vertices
  + *directed* or *undirected*
  + storage: edge *list*, adjacency *matrix*, etc.
+ Some corner cases:
  + *Self-loop*: edge from vertex to itself
  + *Parallel* edges: multiple edges with same start/end
+ **Complexity** of graph algorithms in terms of \`|V| and |E|\`

---
## Applications of graphs
| graph | vertex | edge |
|-------|--------|------|
| air **transport** |  airport | *flight* path |
| **social** | person | *friendship*/relationship |
| **internet** | computer | *network* connection |
| **finance** | *stock*/asset | transaction |
| **neural** net | neuron | *synapse* |
| **protein** net | protein | *protein-protein* interaction |

---
<div class="caption">
Map of the internet (coloured by RIR):
[OPTE 2015](http://www.opte.org/the-internet/)
</div>

![OPTE 2015](static/bg/opte-internet-2015.png)

---
<div class="caption">
24hrs of flights in/out of Europe:
[422South for NATS](http://422south.com/work/euro-24-air-traffic-visualization-for-nats)
</div>

![422South Euro24](static/bg/422south-euro24.jpg)

---
<div class="caption">
[Connectome](https://en.wikipedia.org/wiki/Connectome)
of 20 adult human brains: map of white-matter connections.
[CC-BY-SA 4.0, Andreashorn](https://commons.wikimedia.org/wiki/File%3AThe_Human_Connectome.png)
</div>

![Connectome](static/bg/Human_Connectome.png)

---
<div class="caption">
[Proteome](https://en.wikipedia.org/wiki/Proteome)
(protein-protein interaction network) of syphilis spirochete, <i>T. pallidum</i>.
[CC-BY 1.0, Häuser et al.](https://commons.wikimedia.org/wiki/File%3AThe_protein_interaction_network_of_Treponema_pallidum.png)
</div>

![Proteome](static/bg/Proteome-Treponema_pallidum.png)
<!-- .element: style="width:80%" -->

---
## Problems in graph theory
+ **Path finding**: is there a path from *u* to *v*?
+ **Shortest path**: find the *shortest* path from *u* to *v*
+ **Cycle**: does the graph have any *cycles*?
+ **Euler cycle**: traverse each *edge* exactly once
+ **Hamilton cycle**: touch each *vertex* exactly once
+ **Connectivity**: are all the vertices *connected*?
+ **Bi-connectivity**: can you disconnect the graph by *removing* one vertex?
+ **Planarity**: draw graph in *2D* w/o crossing edges?
+ **Isomorphism**: are two graphs *equivalent*?

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to graph algorithms
  + Applications and typical problems
  + **Edge list, adjacency list, adjacency matrix**
+ **Breadth-first graph traversal**
+ Depth-first graph traversal
  + Parenthesis structure
  + Edge classification
  + Topological sort
  + Finding strongly-connected components

---
## Representing edges
+ **Edge list**: array/list of *(u,v)* pairs of nodes
  + [ (1,2), (1,3), (2,4) ]
  + How to find *neighbours* of a vertex *u*?
+ **Adjacency list**: indexed by *start* node
  + [ {*1*: [2, 3]}, {*2*: [1, 4]}, {*3*: [1]}, {*4*: [2]} ]
  + How to find the (out)-*degree* of each vertex?
+ **Adjacency matrix**: boolean *|V|* x *|V|* matrix
  + *A[i,j]* = 1 iff *(i,j)* is an edge:
    \` ( (0, 1, 1, 0), (1, 0, 0, 1), (1, 0, 0, 0), (0, 1, 0, 0) )\`
  + What about *directed* graphs?  *Weighted* graphs?
+ In practise, most graphs are **sparse**: what representation?

---
## Graph traversal: breadth-first
+ **Traversal**: visits each node exactly *once*
+ BFS: overlay a **breadth-first tree**
  + Choose a *start* (root) node
  + *Path* in tree = *shortest* path from root
    + Only nodes *reachable* from start node
  + BFS tree not necessarily *unique*
+ Graph could have **loops**:
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
![BFS](static/img/Breadth-First-Search-Algorithm.gif)
</div></div>

---
## BFS properties
+ BFS examines nodes in order of **distance** from source
  + Queue first holds all nodes of distance *k*,
  + Then all nodes of distance *k+1*, etc.
+ **Levels** of BFS tree = nodes of same *distance* from source
+ &rArr; BFS computes **shortest paths** from source <br/>
  to all other reachable nodes in time \`O(|V|+|E|)\`
  + e.g., [Kevin Bacon number](https://oracleofbacon.org/):
    + vertices = *actors*, edges = *shared movies*

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to graph algorithms
  + Applications and typical problems
  + Edge list, adjacency list, adjacency matrix
+ Breadth-first graph traversal
+ **Depth-first graph traversal**
  + Parenthesis structure
  + Edge classification
  + Topological sort
  + Finding strongly-connected components

---
## Trémaux maze solving
+ Graph representation of a **maze**:
  + *Vertex* = **intersection**, *edge* = **passage**

<div class="imgbox"><div><ul>
<li> <strong>Theseus</strong> slaying the Minotaur in the <em>Labyrinth</em> <ul>
  <li> Ariadne gave him a tool: <strong>ball of string</strong>: </li> </ul></li>
<li> <strong>Unwind</strong> string as you go <ul>
  <li> <strong>Track</strong> each <em>visited</em> intersection + passage </li>
  <li> <strong>Retrace</strong> steps when no unvisited passages </li>
  </ul></li>
</ul></div><div>
![Theseus in the Labyrinth](static/img/Theseus-Labyrinth.jpg)
<div class="caption">
2nd c. AD Roman mosaic, Kunsthistorische Museum, Vienna <br/>
[(Asaf Braverman, CC-BY-NC-SA 2.0)](https://www.flickr.com/photos/theheartindifferentkeys/4172187101)
</div>
</div></div>

---
## Depth-first search
+ First explore as **deep** as we can
  + **Backtrack** to explore other paths
  + **Recursive** algorithm (ball of string = *call stack*)
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
  for u in V:     # why loop over *all* V?
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
![DFS](static/img/DFS.svg)
</div></div>

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to graph algorithms
  + Applications and typical problems
  + Edge list, adjacency list, adjacency matrix
+ Breadth-first graph traversal
+ Depth-first graph traversal
  + **Parenthesis structure**
  + **Edge classification**
  + Topological sort
  + Finding strongly-connected components

---
## DFS: parenthesis structure
+ Each node's **subtree** is visited <br/>
  between its *discovery* and *finish* times
+ **Print** a \`(.\_u\` when we *discover* node *u*
  + Print a \`)\_u\` when we *finish* it
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

<div class="imgbox"><div style="flex:3"><ul>
<li> <strong>White-path</strong> theorem: <ul>
  <li> <em>v</em> is a <strong>descendant</strong> of <em>u</em> in the DFS &hArr; </li>
  <li> When <em>u</em> is <em>discovered</em>, there is <br/>
    a <strong>path</strong> <em>u</em> &rarr; <em>v</em> all of <em>white</em> vertices </li>
  </ul></li>
</ul></div><div style="flex:2">
![DFS](static/img/DFS.svg)
</div></div>

---
## DFS: flood-fill
+ **Vertex**: *pixel*
+ **Edge**: adjacent pixels of *similar* colour
+ **Blob**: all pixels *connected* to given pixel

<div class="imgbox"><div>
![Manhattan map](static/img/Manhattan.svg)
</div><div>
![Australia grid](static/img/Australia_grid2.png)
</div></div>

---
## DFS: edge classification
+ All **edges** in a graph are either
  + **Tree** edges: in the *DFS forest*
  + **Back** edges: up to *ancestor* in same DFS tree (incl *self-loop*)
  + **Forward** edges: down to *descendant*
  + **Cross** edges: *different* subtrees or DFS trees
+ For directed graphs: **acyclic** &hArr; no **back** edges

![edge classification](static/img/edge-class.svg)

---
## DFS: preparing for a date [(XKCD)](http://xkcd.com/761/)
![XKCD 761](https://imgs.xkcd.com/comics/dfs.png)
<!-- .element: style="width: 75%" -->

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to graph algorithms
  + Applications and typical problems
  + Edge list, adjacency list, adjacency matrix
+ Breadth-first graph traversal
+ Depth-first graph traversal
  + Parenthesis structure
  + Edge classification
  + **Topological sort**
  + **Finding strongly-connected components**

---
## DFS: topological sort
+ Linear **ordering** of vertices such that:
  + for every edge *u* &rarr; *v*, *u* comes **before** *v* in the sort
  + Assumes **no cycles**! (i.e., *DAG*: directed acyclic)
+ **Applications**: *dependency* resolution, *compiling* files, <br/>
  task planning / *Gantt* chart
+ Tweak **DFS**: when each vertex is *finished*, <br/>
  insert it at the *head* of a linked list
  + Sort in **decreasing** order of *finish* time
+ *DFS* might not be **unique**, so <br/>
  *topological sort* might not be unique

---
## Topological sort: example
+ Proof of **correctness**: \`(u,v) in E => v.f < u.f\`
+ When DFS explores *(u,v)*, what **colour** is *v*?
  + if **gray**: so *v* is an **ancestor** of *u*
    + So *(u,v)* is a **back** edge
    + So graph has a **loop** (disallowed)
  + if **white**: becomes a **child**: *u.d* < *v.d* < *v.f* < *u.f*
  + if **black**: *v* **done**, but *u* not done yet: *v.f* < *u.f*

![DFS](static/img/DFS.svg)
<!-- .element: style="width:30%" -->

---
## DFS: connected components
+ Largest **completely-connected** set of vertices:
  + Every *vertex* has a **path**
    to every *other* vertex in the component
+ Algorithm:
  + Compute *DFS* to find **finishing** times
  + **Transpose** the graph: *reverse* all edges
  + Compute *DFS* on transposed graph
    + Start at vertex that finished **last** in orig DFS
  + Each **tree** in final DFS is a separate **component**

![components](static/img/conn-comp.png)
<!-- .element: style="width:80%" -->

---
## Connected components
<div class="imgbox"><div style="flex:2"><ul>
<li> (a) <strong>Original</strong> graph and <em>DFS</em> (shaded) <ul>
  <li> Starting at <em>c</em> </li></ul> </li>
<li> (b) <strong>Transpose</strong> graph and its <em>DFS</em> <ul>
  <li> Starting at <em>b</em> (finished <strong>last</strong> in first DFS)</li>
  </ul></li>
<li> (c) Coalesce vertices into <strong>component graph</strong> </li>
</ul></div><div style="flex:3">
![Fig 22-9: components](static/img/Fig-22-9.svg)
</div></div>

---
## Problems in graph theory
+ **Path finding**: is there a path from *u* to *v*?
+ **Shortest path**: find the *shortest* path from *u* to *v*
+ **Cycle**: does the graph have any *cycles*?
+ **Euler cycle**: traverse each *edge* exactly once
+ **Hamilton cycle**: touch each *vertex* exactly once
+ **Connectivity**: are all the vertices *connected*?
+ **Bi-connectivity**: can you disconnect the graph by *removing* one vertex?
+ **Planarity**: draw graph in *2D* w/o crossing edges?
+ **Isomorphism**: are two graphs *equivalent*?

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Outline for today
+ Intro to **graph** algorithms
  + *Applications* and typical *problems*
  + *Edge list*, adjacency list, adjacency matrix
+ **Breadth-first** graph traversal
+ **Depth-first** graph traversal
  + **Parenthesis** structure
  + Edge **classification**
  + Topological **sort**
  + Finding **strongly-connected** components

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" -->
## Online demos
+ **Breadth-first** search:
  + [U San Fran](https://www.cs.usfca.edu/~galles/visualization/BFS.html)
    (generate random graphs)
  + [VisuAlgo](https://visualgo.net/dfsbfs)
    (draw your own graph; step through code)
+ **Depth-first** search:
  + [U San Fran](https://www.cs.usfca.edu/~galles/visualization/DFS.html)
    (only one tree of the DFS forest)
  + [VisuAlgo](https://visualgo.net/dfsbfs)
    (edge classification, only one tree)
+ **Topological sort**:
  + [U San Fran](https://www.cs.usfca.edu/~galles/visualization/TopoSortDFS.html),
    [VisuAlgo](https://visualgo.net/dfsbfs)
+ **Connected** components:
  + [U San Fran](https://www.cs.usfca.edu/~galles/visualization/ConnectedComponent.html)
  + [VisuAlgo](https://visualgo.net/dfsbfs)
    (SCC: Kosaraju's algorithm)

---
<!-- .slide: data-background-image="static/bg/422south-euro24.jpg" class="empty" -->
