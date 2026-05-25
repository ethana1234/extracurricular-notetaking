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
(q, C, M, _)
Input: query node q, candidate neighbors C, num of neighbors to return M
Output: M nearest elements to q
```