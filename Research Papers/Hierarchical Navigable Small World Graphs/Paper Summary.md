[Original Paper](https://arxiv.org/abs/1603.09320)
## Motivation
* Searches for [[Background Info#Nearest Neighbor Search Problems|approximate nearest neighbors]] in n-dimensional space
* Trying to improve on polylog [[Background Info#Navigable Small Worlds (NSWs)|NSW]] performance, to get down to log time complexity
* Uses layering, inspired by [[Background Info#Skip List|skip lists]], in order to shrink the traversal between longer links in the original graph
* The higher layer, the less nodes
* $l$, the integer indicator for a node's max layer, is determined by an exponentially decaying probability (creates log scaling)
* $l$ is very similar to the degree of a node in an NSW. Combining all the connections in HNSW's layers is like creating an NSW
* Element insertion doesn't need to be shuffled, stochasticity is achieved by layer separation
* Selecting edges for the proximity graph at each layer uses a heuristic based on the distance to the newly inserted node instead of just closest neighbors
## Algorithms
### Construction
* HNSW construction involves repeated `INSERT` calls of stored elements into graph
* Use 2 priority queues just like NSW.
	* $C$: Candidate queue that prioritizes nodes nearest to query node to greedily look for candidates
	* $W$: Result queue that prioritizes nodes further from query node, to efficiently remove nodes when a closer candidate is found
* Enter point is fixed (NSW it's random), higher layers ($l_c > l$) it's one element, lower layers it's a set of length $ef$
* HNSW only does one search, we don't need the multi search since we no longer get stuck in local mins.
```pseudo
INSERT(hnsw, q, M, M_max, efConstruction, m_L)
Input: multilayer graph hnsw, new element q, number of established connections M, maximum number of connections for each element per layer M_max, size of the dynamic candidate list efConstruction, normalization factor for level generation m_L
Output: updated hnsw with q inserted
W <- {}
ep <- get entry point (random?)
L <- max layer
l <- floor(-ln(unif(0..1) * m_L))
for l_i <- L ... l+1:
	// Do regular search through layers where new element won't be inserted
	W.add(SEARCH-LAYER(q, ep, 1, l_i))
ep <- nearest element from W to q
for l_i <- min(L, l) ... 0:
	// Insert new node + update edges in each layer it appears in
	W.add(SEARCH-LAYER(q, ep, efConstruction, l_i))
	neighbors <- SELECT-NEIGHBORS(q, W, M, l_i) // use either select algo
	for e in neighbors:
		eConn <- neighborhood(e, l_i)
		if len(eConn) > M_max:
			eConn <- SELECT-NEIGHBORS(q, W, M, l_i) // clip neighbor connections
			neighborhood(e, l_i) <- eConn
	ep <- W.pop()
if l > L:
	ep for hnsw going forward <- q
return hnsw
```

```pseudo
SEARCH-LAYER(q, ep, ef, l_c)
Input: query node q, set of enter points ep, num of nearest neighbors to return ef, layer num l_c
Output: ef closest neighbors to q
v <- ep // visited nodes
C <- ep // candidate nodes
W <- ep // dynamic list of found nearest neighbors
while |C| > 0:
	c <- C.pop(nearestElement(C, q))
	f <- furtherstElement(W, q)
	if distance(c, q) > distance(f, q):
		break // all neighbors in W are evaluated
	for e in neighborhood(c, l_c):
		if e not in v:
			v.add(e)
			f <- furtherstElement(W, q)
			if distance(e, q) < distance(f, q) or |W| < ef:
				C.add(e)
				W.add(e)
				if |W| > ef:
					W.pop(furtherstElement(W, q))
return W
```

```pseudo
SELECT-NEIGHBORS-HEURISTIC(q, C, M, l_c)
Input: query node q, candidate neighbors C, num of neighbors to return M, layer num l_c
Output: M neighbors selected to connect to q
R <- {}
W <- C
W_d <- {} // discarded candidates
if extendCandidates: // optionally extend W using neighbors of candidates
	for e in C:
		for f in neighborhood(e, l_c):
			if f not in W:
				W.add(f)
while len(W) > 0 and len(R) < M:
	e <- W.pop(nearestElement(W, q))
	if distance(q, e) < R.peek(nearestElement(R, q)):
		R.add(e)
	else:
		W_d.add(e)
if keepPrunedConns: // optionally keep some discarded candidates
	while len(W_d) > 0 and len(R) < M:
		R.add(W_d.pop(nearestElement(W_d, q)))
return R
```

```pseudo
SELECT-NEIGHBORS-SIMPLE(q, C, M, _)
Input: query node q, candidate neighbors C, num of neighbors to return M
Output: M nearest elements to q
```
### Search
* Once we have built the HNSW, the following K-NN search can be used. It's similar to `INSERT` a new node at layer 0
```pseudo
K-NN-SEARCH(hnsw, q, K, ef)
Input: data structure hnsw, query node q, num of neighbors to return K, size of dynamic candidate list ef
Output: set of K nearest elements to q
W <- {}
ep <- hnsw.entryPoint // single element in highest layer
L <- hnsw.L
for l_c <- L ... 1:
	W <- SEARCH-LAYER(q, ep, 1, l_c)
	ep <- nearestElement(W, q)
W <- SEARCH-LAYER(q, ep, ef, 0)
R <- {}
for i <- 1 ... K:
	// get K nearest elements from W
	R.add(W.pop(nearestElement(W, q)))
return R
```
### Construction Parameters
* Optimal performance requires overlap of neighbors between layers to be small (layers shouldn't be keeping track of same neighbor edges).
	* This means $m_L$ should be small, but too small a value means the greedy search at each layer becomes inefficient. So there is an optimal value for $m_L$. 
	* Corresponding to a skip list default of $p = 1/M$, a viable choice is $m_L = 1 / ln(M)$
* Hyperparameter $M_{max0}$ is also influential on search performance.
	* Represents max degree of node is layer 0
	* $M_{max0} = M$ leads to poor search performance for high recall queries
	* $M_{max0} = 2M$ leads to better performance, and going higher typically hurts performance and memory usage
* $efConstruction$ parameter can be balanced for the tradeoff between index construction time and query time
### Complexity
* Search Complexity
	* Approximates to $O(log(n))$
	* The structure has ~$log(N)$ layers inferred by the assignment of $l$ in `INSERT`, this is similar to the skip list
	* The authors approximate $O(1)$ complexity for searching at each layer. This follows from 2 assumptions:
		* The walk for each layer is short: greedily finding an element closer to the query means traversing elements that aren't* in the layer above (every element above by necessity *must* be further than the entry point of the current layer). This assumption let's $S_l$, the number of steps at a layer, be calculated without $N$, making $S_l$ a constant time task. 
		* The authors assume the average number of neighbors does not exceed a constant $C$, which means the cost for each step itself is a constant. 
		* So the complexity at each layer is $O(S_l * C) -> O(1)$
	* The massive assumption is that the graphs at each layer are [[Background Info#Proximity Graphs|Delaunay graphs]], but in reality they are approximations. This means the greedy walk can stop at a false local min. The extra cost of the backtracking step HNSW must do at layer 0 has a constant effect to the complexity at small dimensions, but unknown with higher dimensional data.
* Construction Complexity: Since construction is very similar to search, construction for $N$ elements would be $O(nlog(n))$
* Memory Complexity: determined by number of links/edges. Each element has $M_{max0} + m_L * M_{max}$ links. 
