# Notes of paper reading for answer tree ranking
- [Notes of paper reading for answer tree ranking](#notes-of-paper-reading-for-answer-tree-ranking)
  - [Related works](#related-works)
    - [Keyword search over RDF graphs [1] :](#keyword-search-over-rdf-graphs-1)
      - [answers:](#answers)
      - [ranking](#ranking)
    - [Scalable keyword search on large RDF data. [2]](#scalable-keyword-search-on-large-rdf-data-2)
      - [answers:](#answers-1)
    - [Computational fact checking from knowledge networks [3]](#computational-fact-checking-from-knowledge-networks-3)
      - [answers:](#answers-2)
      - [ranking:](#ranking-1)
  - [References](#references)

## Related works
### Keyword search over RDF graphs [1] :
#### answers:
1. A query of keywords $q=\{q_{1},q_{2},...,q_{m}\}$->extract all triples that match the keyword $q_{i}$ in $q$->list $L=\{L_{1},L_{2},...L_{m}\}$
2. Then the answer trees are constructed by selecting edges that are adjacency (**bidirectional**, basically they don't care about the direction of the edges) from each $L$ for each triple $t_{i}$ 
3. Some comments: <br>[1] Possible for a node to match several keywords <br>[2] The algorithm will not take more than one triple from a certain $L_{j}$, all their adjacency list $A(t_{i})$ will have a length shorter than $m$ <br>[3] The [link](https://people.mpi-inf.mpg.de/~elbass/demo/rdftext.txt) given of their results are not accessible, so impossible to check their results, though some of their descriptions are quite confusing.

#### ranking
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

### Scalable keyword search on large RDF data. [2]
#### answers:
1. $$A(q)=\underset{g \in C(q)}{\operatorname{argmin}} s(g), \text { and } s(g)=\sum_{r, v_{i} \in g, i=1, m} d\left(r, v_{i}\right)$$
   $g$ is an answer subgraph of $G$, $C(q)$ set of candidate answers, $d\left(r, v_{i}\right)$ is the distance from a root answer node $r$ to other answer nodes $v_{i}$, and $r$ is the reachable node for all other nodes. (Still they don't care about the directions)
2. **termination** Comments: The general idea is as above, but their description is really confusing.
   
### Computational fact checking from knowledge networks [3]
#### answers:
1. The graph is again undirected
2. *Comments* <br>
[1]Strange thing: They haven't mentioned how they haven't mined the possible paths.<br>
[2]The focus (fact checking) of this paper is different: They don't really need to explore all possible paths and evaluate their interestingness.
#### ranking:
1. A truth value $\tau(e)=\max \mathcal{W}\left(P_{s, o}\right)$
   
    $$
    \mathcal{W}_{u}\left(P_{s, o}\right)=\mathcal{W}_{u}\left(v_{1} \ldots v_{n}\right)=\left\{\begin{array}{ll}
    1 & n=2 \\
    \left[1+\max _{i=2}^{n-1}\left\{\log k\left(v_{i}\right)\right\}\right]^{-1} & n>2
    \end{array}\right.
    $$
    $s$ source, $o$ object, a path $P_{s, o}=v_{1}, v_{2} \ldots v_{n}$
2. *Comments*: Shorter paths are still preferred, here e $k(v)$ is the degree of entity $v$, i.e., the number of WKG statements in which it participates. It is a similar idea to the specificity used in ConnectionLens
## References
[1] Elbassuoni, Shady, and Roi Blanco. "Keyword search over RDF graphs." Proceedings of the 20th ACM international conference on Information and knowledge management. 2011.

[2] Le, Wangchao, et al. "Scalable keyword search on large RDF data." IEEE Transactions on knowledge and data engineering 26.11 (2014): 2774-2788.

[3]Ciampaglia, Giovanni Luca, et al. "Computational fact checking from knowledge networks." PloS one 10.6 (2015).