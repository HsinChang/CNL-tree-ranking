# Notes of paper reading for answer tree ranking

## Related works
### Elbassuoni et al. [1] : <br>
**answers**: <br>
1. A query of keywords $q=\{q_{1},q_{2},...,q_{m}\}$->extract all triples that match the keyword $q_{i}$ in $q$->list $L=\{L_{1},L_{2},...L_{m}\}$
2. Then the answer trees are constructed by selecting edges that are adjacency (**bidirectional**, basically they don't care about the direction of the edges) from each $L$ for each triple $t_{i}$ 
3. Some comments: <br>[1] Possible for a node to match several keywords <br>[2] The algorithm will not take more than one triple from a certain $L_{j}$, all their adjacency list $A(t_{i})$ will have a length shorter than $m$ <br>[3] The [link](https://people.mpi-inf.mpg.de/~elbass/demo/rdftext.txt) given of their results are not accessible, so impossible to check their results, though some of their descriptions are quite confusing.

**ranking** <br>
1. General idea: ranked by *query likelihood* (the probability of generating the query $Q=\{q_{1},q_{2},...,q_{n}\}$ given the subgraph $G=\{t_{1},t_{2},...,t_{n}\}$, here the $G$ is the adjacency list $A(t_{i})$ in the previous step), the likelihood:
   $$P(Q | G)=\prod_{i=1}^{m} P\left(q_{i} | G\right), $$
   $$P\left(q_{i} | G\right)=\sum_{j=1}^{n} \frac{1}{n} P\left(q_{i} | t_{j}\right)$$
   where $P(q_{i}|G)$ is the probability of $q_{i}$ in the LM of $G$
2. So for each $P(q_{i}|t_{j})$, 
   $$P\left(q_{i} | D_{j}, r_{j}\right)=\beta P\left(q_{i} | D_{j}\right) P\left(r_{j} | q_{i}\right)+(1-\beta) P\left(q_{i} | D_{j}\right)$$
    $D_{j}$: all the terms associated with $t_{j}$, $r_{j}$: the predicate of $t_{j}$,<br> $P(q_{i}|D_{i})$ is the the probability of generating the term $q_{i}$ from the triple document $D_{i}$<br>
    $P(r_{j}|q_{i})$ is the probability that the predicate $r_{j}$ is relevant to the term $q_{i}$
3. Comment:
   <br>[1] The key idea (that is useful to us) in this ranking method is the this probability of predicates for a term $q_{i}$, which is to give more importance to certain predicates, it can be tested, but it doesn't seem to be a brilliant solution for our *Paris* problem. 

### 

## References
[1] Elbassuoni, Shady, and Roi Blanco. "Keyword search over RDF graphs." Proceedings of the 20th ACM international conference on Information and knowledge management. 2011.