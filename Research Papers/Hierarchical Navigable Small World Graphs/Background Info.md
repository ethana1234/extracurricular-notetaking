## Nearest Neighbor Search Problems
* Deep learning models embed information into $n$-dimensional vectors. On trained models, 2 vectors whose points in the $n$-dimensional space are meant to be similar. If the embedding represents a chunk of text, if 2 vectors are near each other they are assumed to be semantically similar.
* This creates an important problem to solve: if we have a lot of points on a graph (a vector store), and we want to find the nearest neighbor(s) of a new vector, what's the best way to search for that?
* ==K-NNS==: Find the $k$ nearest neighbors for a given query vector.
* Can naïvely search by just computing distance of query vector to every other vector, this scales linearly with the number of stored elements.
* ==Curse of Dimensionality==: issues in scale of data can be different when using a high $n$ number of dimensions. This is relevant for things like embedding models since they map data to very high dimension vectors.
* The problems in the similarity search space almost always have a high enough $n$ that trying to get the exact nearest neighbors doesn't scale well.
* ==K-ANNS==: Approximate the $k$ nearest neighbors for a given query vector. 
* The quality of an inexact search is determined by ==recall==: true # of nearest neighbors found / $k$ 
* K-ANNS have a number of areas of solutions:
	* Trees
	* Hashing
	* Product Quantization
	* Proximity Graphs	
## Proximity Graphs
* Proximity Graphs can act like an index for a vector store
* Precompute distances between points in a vector store and create a graph where nodes are points in the vector store and bidirectional edges can roughly indicate that the 2 nodes are "nearby neighbors" in the actual $n$-dimensional space
* Use ==greedy best-first search== to find nearest neighbors in the graph
	* From current node, use a heuristic function to determine "cost" to get from current node to target node
	* When taking a step, always take the lowest cost candidate
*  ==Delaunay graphs==:
	* Ideal graph that would guarantee greedy search finds true nearest neighbor
	* Not feasible to generate since it requires exhaustive distance calculations
![[delaunay_graph.svg|669]]
* Start at an entry node and traverse the graph to **greedily** find the approximate nearest neighbor(s) of a query vector.
* *Only* uses the precomputed information in the graph to determine what's "nearby", so there can be trade offs based on how the graph is constructed. Trying to fit the graph to be exact defeats the purpose of proximity graphs, which are meant to provide scalable performance. 
* Can have trouble with clustered data
### Navigable Small Worlds (NSWs)
* By definition have logarithmic or polylogarithmic scaling of the # of hops during greedy traversal w.r.t. network size
* Inserting elements in random order, connecting to $M$ closest neighbors of currently inserted elements
* Early connections in the graph help globally connects clusters later on thanks to the element of randomness. 
* Struggles with low dimensional data compared to other methods because of overhead for the graph structure
* Effective as a distributed search system
* NSW starts at a random entry node to greedy search. Because this very typically can be a low degree node, this leaves it prone to getting stuck in a local min. It creates those high degree hub nodes, but if the greedy search doesn't reach those, it doesn't matter.
* Polylogarithimic time complexity comes from:
	* log time complexity from the average number of hops to traverse greedily
	* log time complexity to compute distances at each step in the path (this is based on the average node degree)
## Skip List
* Multi layer linked list
* Higher layers contain a subset of nodes of the layer below.
* Made by determining probabilistically the highest layer each node appears in. The higher the node, the less likely a node is to be placed there.
* Usually $1/2$ is used as the probability a value is moved up to the next layer, so for layer $i$, the probability a node appears in that layer is $P(i) = .5^i$.
![[Skip List|Illustration of searching for the value 17 in a skip list with 3 layers|900]]
