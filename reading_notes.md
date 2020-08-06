
- [Notes for answer tree ranking](#notes-for-answer-tree-ranking)
  - [Related works for query answering/ranking](#related-works-for-query-answeringranking)
    - [Keyword search over RDF graphs [1] :](#keyword-search-over-rdf-graphs-1-)
      - [answers:](#answers)
      - [ranking](#ranking)
    - [Scalable keyword search on large RDF data. [2]](#scalable-keyword-search-on-large-rdf-data-2)
      - [answers:](#answers-1)
    - [Computational fact checking from knowledge networks [3]](#computational-fact-checking-from-knowledge-networks-3)
      - [answers:](#answers-2)
      - [ranking:](#ranking-1)
    - [Finding Top-k Min-Cost Connected Trees in Databases [4]](#finding-top-k-min-cost-connected-trees-in-databases-4)
      - [Generals:](#generals)
      - [Comments:](#comments)
  - [Related works for sub-graph extraction](#related-works-for-sub-graph-extraction)
    - [MING: Mining Informative Entity Relationship Subgraphs [5]](#ming-mining-informative-entity-relationship-subgraphs-5)
      - [Problem Statement](#problem-statement)
      - [the algorithm](#the-algorithm)
      - [Comments](#comments-1)
      - [Dataset: `YAGO`](#dataset-yago)
      - [Results](#results)
    - [ESPRESSO: Explaining Relationships between Entity Sets [6]](#espresso-explaining-relationships-between-entity-sets-6)
      - [ESPRESSO knowledge graph](#espresso-knowledge-graph)
      - [Objective](#objective)
      - [Algorithm](#algorithm)
      - [Dataset: `Espresso Knowledge Graph`](#dataset-espresso-knowledge-graph)
      - [Results](#results-1)
    - [Center-Piece Subgraphs: Problem Definition and Fast Solutions [7]](#center-piece-subgraphs-problem-definition-and-fast-solutions-7)
      - [Problem Statement](#problem-statement-1)
      - [Algorithm](#algorithm-1)
      - [Compliments](#compliments)
      - [Dataset: `DBLP`](#dataset-dblp)
      - [Results](#results-2)
- [Graph Simplification](#graph-simplification)
  - [works and related works already done in the group](#works-and-related-works-already-done-in-the-group)
    - [Irène's work](#irènes-work)
      - [Basic ideas](#basic-ideas)
      - [Tests](#tests)
        - [with Wikidata Gauthier](#with-wikidata-gauthier)
        - [with `cl-decodex.json`](#with-cl-decodexjson)
        - [with `nosdeputes.fr_deputes_en_mandat_2020-03-03.json`](#with-nosdeputesfr_deputes_en_mandat_2020-03-03json)
      - [Graph obtained with Neo4J](#graph-obtained-with-neo4j)
      - [The Java implementation](#the-java-implementation)
    - [RDFQuotient](#rdfquotient)
- [Dataset Construction](#dataset-construction)
  - [get data from YAGO4](#get-data-from-yago4)
    - [find all entities with `org.wikipedia.fr`](#find-all-entities-with-orgwikipediafr)
    - [delete duplicate entities](#delete-duplicate-entities)
    - [for those entities, find all related informations](#for-those-entities-find-all-related-informations)
      - [find most frequent predicates](#find-most-frequent-predicates)
      - [find most frequent nodes](#find-most-frequent-nodes)
      - [find most frequent types](#find-most-frequent-types)
      - [By Postgres](#by-postgres)
    - [explore a path](#explore-a-path)
    - [load the data with `RDF-DB`](#load-the-data-with-rdf-db)
      - [Find all paths from length 1 to length 4 with filtering:](#find-all-paths-from-length-1-to-length-4-with-filtering)
    - [Link with HAVTP](#link-with-havtp)
      - [Find common entities](#find-common-entities)
    - [problems loading YAGO4 with `rdflib`](#problems-loading-yago4-with-rdflib)
      - [`.` right in the third column instead of being the fourth one](#-right-in-the-third-column-instead-of-being-the-fourth-one)
      - [Mixed usage of `\t` and `\s`](#mixed-usage-of-t-and-s)
      - [Space inside certain entities:](#space-inside-certain-entities)
  - [load with `ConnectionLens`](#load-with-connectionlens)
    - [divide the dataset](#divide-the-dataset)
    - [Python Implementation](#python-implementation)
- [References](#references)
# Notes for answer tree ranking
use git
```
find . -name \*.java | xargs git add 
```
## Related works for query answering/ranking
Graph summarizations[2]?

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
   $g$ is an answer subgraph of $G$, $C(q)$ set of candidate answers, $d\left(r, v_{i}\right)$ is the distance from a root answer node $r$ to other answer nodes $v_{i}$, and $r$ is the reachable node for all other nodes. (Still they don't care about the directions, general idea is *go from the nodes, search for a root*)
2. **termination** Idea is, We will maintain a set $M$ of length $m$ for each node we explored, it contains: keywords that are reachable to the node and the best known distance. Keywords not yet reached will hold a `nil` value in that position. A $M$ without `nil` delivers an answer.


   
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


### Finding Top-k Min-Cost Connected Trees in Databases [4]

#### Generals:
The algorithm proposed by the authors is called DPBF (Dynamic Programming Best First), the general idea is to treat the problem as a *Steiner tree problem*, and get min-cost rooted trees:<br>
1. They built a priority queue of trees, the element in this queue contains: its root, its keywords covered, its cost. The priority is decided by the cost.
2. For a new tree generated, they will check if there is already a tree with the same root and keyword coverage in the queue. If not, it is enqueued, otherwise it is only enqueued if it has a smaller cost.
3. So tree GROW and tree MERGE are performed, an answer is given when a tree containing all keywords is found 


#### Comments: 
1. It is way better written than [2]
2. *The optimal substructure property* does not hold: The case is, for a certain query, we do have different nodes that match exactly the same set of keywords, those nodes have the same $\mathbf{p}$, but they are not even connected by 
`sameAs` edges. So our settings cannot be simplified to a **Steiner tree problem**, like in this paper.
3. For directed graphs, the authors have kept only the nodes reachable by following the given direction as *neighbors*. So in our *ConnectionLens* case, it means the answer tree can only have one *inversion* at the root node of the tree.
## Related works for sub-graph extraction
### MING: Mining Informative Entity Relationship Subgraphs [5]
#### Problem Statement
In a Entity-Relation Graph ($G$), from $k$ **given entities of interest**, extracting the most informative subgraph
#### the algorithm
 1. extract subgraph $C$ from $G$<br>The graph is extracted with heuristics, the authors haven't specified how, but as indicated they should have used a [Candidate Generation](https://www.mccurley.org/papers/kdd2004.pdf) method similar to the one in the link. Which, to summarize: it picks one node each time that  *(a) close to the source s or the sink t
(b) with strong connections (c) low
degree* until the **size** of the graph meets a threshold (this threshold is not clear in the paper, this whole part is not clearly explained)
2. run the Steiner-Tree Approximation in Relationship graph algorithm to determine a subtree $T$ of $C$
   for each node, we calculate two probabilities $P_{+}(v)$ and $P_{-}(v)$, if $P_{+}(v)>P_{-}(v)$, this node is informative, else this node is uninformative.<br>
   Here: it is unclear how the initial +/- nodes are defined in order to start the algorithm.<br>
   Each $P_{l}(v),l\in {-,+}$ are composed by $P_{l}^{1}(v) \cdot P_{l}^{2}(v)$, <br>$P_{l}^{1}(v)$ is probability that the random walker starts at any $l$-labeled
node and reaches node $v$, it is explained in an explicate obscure way and not clearly described. I suppose as described, it calculated the RWR with the IRanks scores as the weighted adjacent matrix?

![RWR calculation](https://i.ibb.co/CJMd1YJ/ctmdlw11.png)

<br>$P_{l}^{2}(v)$ is the probability that
any l-labeled node is reached when the random walker starts
his walk at v, calculated through a recursive approach: $P_{l}^{2}(u)=\sum_{v: l a b(v)=l} P_{i n f o}(v | u) P_{l}^{2}(v)$:<br>
  * $P_{l}^{2}(v)$ is initialized as $P_{l}^{2}(v)=\frac{1}{|X|}$, $X:=\{v | \operatorname{lab}(v)=l\}$
  * $P_{\text {info}}(v | u):=\sum_{r} P_{\text {info}}(v | r, u)$
   * $P_{\text {info}}\left(\alpha^{\prime} | \beta, \gamma\right)=\frac{P\left(\alpha^{\prime}, \beta, \gamma\right)}{P(\beta, \gamma)} \approx \frac{W\left(\alpha^{\prime}, \beta, \gamma\right)}{W(\beta, \gamma)}$, 
   * where for edge $\gamma \stackrel{\beta}{\rightarrow} \alpha^{\prime}$, $W\left(\alpha^{\prime}, \beta, \gamma\right)$ denotes the number of domain witnesses, $W(\beta, \gamma)$ stands for the number of witnesses for the pattern $(*, \beta, \gamma)$

Then, it select all nodes labled $+$ or the best few nodes labeled $+$
#### Comments
It is either the authors are very irresponsible or the CIKM conference has limited the pages, this paper is definitely not developed in a way to make their method more comprehensible. The abbreviations are fancy, though.

#### Dataset: `YAGO`

#### Results

They queried `famous individuals`, and declared their approach is way better than `CePS`. But their result is not very convincing:
* They limited the obtained graph to 15 nodes only
* They only showed the data on how many times the result is preferred by human judgements

To my point of view, the right way to evaluate their algorithm is: 
still thinking on this if we perform the query on big dataset like `YAGO`

### ESPRESSO: Explaining Relationships between Entity Sets [6]
explaining the relationship between two sets of entities in a
knowledge graph

#### ESPRESSO knowledge graph

#### Objective
From a knowledge graph $K=\left(V, E, \ell_{V}, \ell_{E}, T, R\right)$, two sets of entities $Q_{1}$ and $Q_{2}$.<br>
Output:  $k$ connected subgraphs $S_{1}=\left(V_{1}, E_{1}\right), \ldots, S_{k}=\left(V_{k}, E_{k}\right)$ such that the following formula is **maximized**:
$$\sum_{i=1}^{k}\left[\beta \sum_{e \in E_{i}} \omega(e)+(1-\beta) \sum_{v \in V_{i}} \omega(v)\right]$$

Here we have a threshold $B$ who controls the size of each subgraph
#### Algorithm
Here $B'=B-k$, all $\mathbf{x}$ are vectors from RWR and a
prior $pr(v)$, derived from the overall importance or prominence of v
(e. g., its PageRank score in $\hat{M}$ , popularity determined from external
sources, et)
1. first stage: identify a set of **relationship centers**, which are subsequently connected to query nodes.
   * compute $\operatorname{RC}(v)=\mathbf{x}_{Q_{1}}[v] \cdot \mathbf{x}_{Q_{2}}[v] \cdot \operatorname{pr}(v)$ for each vertex
   * select $k$ best nodes that **(i) relatively prominent and (ii) of type *event*** with highest $RC$ as centers
2. second stage: the **relationship centers** obtained in stage 1 are further expended to obtain the desired subgraphs
   * *key entity discovery*: Start with the center, keep adding nodes by the order of RWR scores until it reaches a size threshold, get node set $V_{c}$, graph size: $k+\left\lceil\gamma B^{\prime} / 2\right\rceil$
   * *Center Context Entity Generation* Firstly, add all adjacent nodes of node in $V_{c}$ --> $G_{c}$, keep deleting nodes with the smallest weighted degree until the graph reaches size $k+\left\lceil\gamma B^{\prime}\right\rceil$
   * *Query Context Generation* keeps adding nodes with the highest score calculated by $\mathbf{x}_{Q_{1}}[v] \mathbf{x}_{Q_{2}}[v] \mathbf{x}_{c}[v] \cdot \operatorname{pr}(v)$, <br>
add nodes until the graph reaches size $B$

#### Dataset: `Espresso Knowledge Graph`
derived from `YAGO2` and `FreeBase`, Enriched by
- edge weights signifying the relatedness
between entities
- semantic types associated with the entities, derived from YAGO2
(Wikipedia categories, WordNet classes) and Freebase (types). (SVM)
-  the popularity of individual entities
over time, reflecting the page view counts of each entity in daily granularity in the time frame 01/01/2012 to 07/31/2014

#### Results
The result evaluation is also based on the rating of humans, but they did vary the scenarios and parameters, we have a more convincing result than MING[5]
### Center-Piece Subgraphs: Problem Definition and Fast Solutions [7]

#### Problem Statement
the *“center-piece subgraph” (CEPS)*

Given: an edge-weighted undirected graph W, Q nodes as
source queries Q = {qi} (i = 1, ..., Q), the softAND
coefficient k and an integer budget b

Find: a suitably connected subgraph H that 
- contains all
query nodes qi 
- at most b other vertices
-  it maximizes a “goodness” function g($\mathcal{H}$).
#### Algorithm
1. the *goodness score* calculation is the same as the photo above, [5] and [6] absorbed the idea of this paper
2. **the EXTRACTion algorithm**: 
   * initialize $\mathcal{H}$ as `NULL`
   * let `len` be the maximum allowable path length
   * while $\mathcal{H}$ not big enough
       * pick a destination node $pd$ by $p d=\operatorname{argmax}_{j \notin \mathcal{H}} r(\mathcal{Q}, j)$, basically the node with the highest score that is not in the graph
       * then for each query node $q$
           * find the best key path $P(q_{i},pd)$ such that 
               * it maximize $C_{s}(i, p d) / s$, where $C_{s}(i, p d)$ is the total score along the path and $s$ the length, $s<$ `len`
          *  add this path $P(q_{i},pd)$ to $\mathcal{H}$
    * Output the final $\mathcal{H}$
    
#### Compliments
It divided queries in three types: *AND query*, *K_softAND query* and *OR query*

*K_softAND:* only require the center-piece nodes to have strong connections to $k$-out-of-$Q$ query
nodes. But the end users still need to specify such a parameter k which is not necessarily an easy task for applications like graph anomaly detection. 

*OR query*: given $Q$
queries, find the subgraph $\mathcal{H}$ the nodes of which are important wrt at least ONE query

#### Dataset: `DBLP`
#### Results
The result is evaluated with `Important Node Ratio (NRatio)`. That is,
“how many important/good nodes are captured by $g(\mathcal{H})$?”:

and `Important Edge
Ratio (ERatio)`. That is, “how many important/good edges
are captured by $g(\mathcal{H})$?”.

Though the important nodes and edges are also picked by humans, it is more convincing that the preference of the results by humans.

# Graph Simplification
## works and related works already done in the group
### Irène's work

#### Basic ideas
1. Basic hypothesis: Any dataset consists of some records and some collections. So, *records* and *collections* are defined in different data sources.
2. Algorithm: https://pad.inria.fr/p/np_iRqpFevLItvyKtRJ_Xin-Internship March 9th

#### Tests
To begin with, all JSON files to be tested should be in this structure:
```
JSON
|
|-nodes
|   |
|   |-node
|   
|-links
    |
    |-link
```
##### with Wikidata Gauthier
Count by category:
```
Counter({None: 4097, 'ENTITY_PERSON': 1616, 'ENTITY_LOCATION': 832, 'ENTITY_ORGANIZATION': 152})
```
Count by labels identified (With more than 1 appearances):
```
None: 4770, 'Business School': 4, 'Reunion': 3, 'Ministry': 3, 'Saint-Joseph': 2, 'Rally': 2, 'Gaulle': 2, 'Management': 2, 'Nicolas Sarkozy': 2, 'Law': 2, 'Adour': 2, "Giscard d' Estaing": 2, 'Courson': 2, 'Sorbonne': 2, 'Froment': 2, 'Manuel Valls': 2
```
The other 1893 labels only appeared once.

We can see that there are 673 nodes who have a None value but with a category, the distribution is:
```
None: 4097, 'ENTITY_LOCATION': 566, 'ENTITY_ORGANIZATION': 101, 'ENTITY_PERSON': 6
``` 
Which means in the implementation, the algorithm will firstly determine the category, if the category cannot be decided, the label won't. 

We can see that most None values with an identified category come from locations and organizations.

Actually, in the graph form `Gauthier` WikiData, all nodes and links in the `JSON` have a label. And all children have homogeneous structures. The `None` come from the difficulty in determining, not the lack of labels in the data. 
##### with `cl-decodex.json`
It crashed half way with such error report:
```
---------------------------------------------------------------------------

TypeError                                 Traceback (most recent call last)

<ipython-input-36-0774f1118777> in <module>
      3 g = clgraph.CLGraph(d)
      4 g.push_down_labels()
----> 5 g.extract_records(MODEL)
      6 

~/Documents/Xin/cnl/connectionlensui/clgraph.py in extract_records(self, model)
    163                 self.records[node] = Record(node,parent,recordtype,standalone,self,model)
    164             elif rca == "Collection":
--> 165                 self.collections[node] = Collection(node,parent,self)
    166             else :
    167                 self.attributs.append(node)

~/Documents/Xin/cnl/connectionlensui/collection.py in __init__(self, node, parent, absG)
      9 
     10 class Collection :
---> 11     def __init__(self,node,parent,absG):
     12         print("COLLECTION : ",node)
     13         self.node = node

TypeError: '>' not supported between instances of 'str' and 'NoneType'
```
The error comes from identifying node `1138639218147346` as `['Record', None]`.
which is 
```json
    {
      "datasetId": 1,
      "id": "1138639218147346",
      "label": "alsace-actu.com",
      "type": "JSON_VALUE"
    }
```
with two connections
```json
      {
        "confidence": "1.00",
        "specificity": ".67",
        "source": "566140699934737",
        "label": "",
        "target": "1138639218147346"
      },
      {
        "confidence": "1.00",
        "specificity": "1.00",
        "source": "1138639218147346",
        "label": "cl:extract",
        "target": "117643783700505"
      },
```
and the connection with no label leads to
```json
    {
      "datasetId": 1,
      "id": "566140699934737",
      "label": "",
      "type": "JSON_STRUCT"
    },
```
This could be the source of identifying this node as `None`
##### with `nosdeputes.fr_deputes_en_mandat_2020-03-03.json`
It crashed half way with such error report:
```

---------------------------------------------------------------------------

KeyError                                  Traceback (most recent call last)

<ipython-input-32-5e9a2ec30ebc> in <module>
      3 g = clgraph.CLGraph(d)
      4 g.push_down_labels()
----> 5 g.extract_records(MODEL)
      6 

~/Documents/Xin/cnl/connectionlensui/clgraph.py in extract_records(self, model)
    176             if node in self.records.keys() and self.records[node].RecordIsAboutNode != None and self.records[node].RecordIsAboutNode != node :
    177                 print("delete :", self.records[node].RecordIsAbout)
--> 178                 self.records[node].merge(self.records[self.records[node].RecordIsAboutNode])
    179                 del self.records[self.records[node].RecordIsAboutNode]
    180                 track = self.replace(self.records[node].RecordIsAboutNode,node,track)

KeyError: 23914353786907
```
The cause of error is from this node:
```json
    {
      "datasetId": 1,
      "id": "23914353786907",
      "label": "Paris",
      "representativeNodeIds": [
        "1420528873111579",
        "474323360940059",
        "1190826815258651",
        "961843979878427",
        "569706311319579",
        "1933831039877147",
        "428896800473115",
        "761315491577883",
        "797107165331483",
        "369748896383003",
        "512607563808795",
        "133600537215003",
        "2205715155386395",
        "1262325098086427",
        "47361381367835",
        "1448255618547739",
        "777091140288539",
        "1858141888184347",
        "776360799764507",
        "679771648294939",
        "490490027835419",
        "335128791351323",
        "1879040768081947",
        "715170807545883",
        "1465338357088283",
        "116794483277851",
        "411883271094299",
        "130941171269659",
        "648401729355803",
        "456666321518619",
        "592463429369883",
        "1477001442492443",
        "1496063225626651",
        "1420520811659291",
        "1612345927270427",
        "1862585887490075",
        "1059553610301467",
        "968081785487387",
        "587803651997723",
        "304264357347355",
        "1498824420884507",
        "1307874310488091",
        "1846519425663003",
        "111769599082523",
        "2067962758430747",
        "1196066773925915",
        "486604674170907",
        "540655901212699",
        "311819744837659",
        "333988862361627",
        "1752653470105627",
        "2003333822808091",
        "1720010553688091",
        "535346828804123",
        "771848929280027",
        "476471588028443",
        "527569114890267",
        "1454006382624795",
        "1469216266911771",
        "1256590188478491",
        "1813241390956571",
        "307716671143963",
        "1219087969550363",
        "1902505291153435",
        "1211588604330011",
        "642407219593243",
        "1517492421787675",
        "1747858759876635",
        "2031374868414491",
        "1249835499913243",
        "1662903171153947",
        "93419319853081",
        "1045033670148123",
        "1773698475360283",
        "1540651691802651",
        "919645554999323",
        "1893086803263515",
        "943033605423131",
        "1758961405526043",
        "777186273394715",
        "1985173417623579",
        "1228454013960219",
        "1581052998975515",
        "558342207438875",
        "1176407619141659",
        "1351319511105563",
        "1000276487045147",
        "1913423372746779",
        "1139150391607323",
        "825558724247579",
        "979685236277275",
        "989912723619867",
        "1625101171687451",
        "2084250120618011",
        "1019932564258843",
        "1262333159538715",
        "973083109228571",
        "1261259989188635",
        "1806635912658971",
        "991307698798619",
        "373024948748315",
        "1474821416288283",
        "810506271064091",
        "1443311802908699",
        "1930780170256411",
        "915618246164507",
        "1807777519370267",
        "240811682824219",
        "1395719192707099",
        "1835899761459227",
        "1995690460839963",
        "595386510082075",
        "593204670890011",
        "2215812615110683",
        "1252924246196251",
        "941369090637851",
        "712669035233307",
        "1428378788823067",
        "770517301723163",
        "1078000697212955",
        "1274627575775259",
        "732474766262299",
        "1978221250740251",
        "403180369215515",
        "558380391333915",
        "1466180788289563",
        "436915519094811",
        "2097396145192987",
        "454506986864667",
        "1999330731884571",
        "2193390922891291",
        "1237431087005723",
        "1925959283376155",
        "941361029185563",
        "1134759854997531",
        "167564556632091",
        "117632365756443",
        "156216032493595",
        "885598405525531",
        "1438258631278619",
        "1834627457089563",
        "1899341520306203",
        "1555397879005211",
        "2030786441117723",
        "456765074309147",
        "423050665263131",
        "1561021122609179",
        "2041550711291931",
        "309573429755931",
        "856488216100891",
        "1524602727563291",
        "911729070440475",
        "1782140102508571",
        "1577709211746331",
        "2084258182070299",
        "1068129393311771",
        "735499202330651",
        "1068137454764059",
        "77381222006811",
        "2038260932018203",
        "757724331638811",
        "2000248722423835",
        "1986005752086555"
      ],
      "type": "ENTITY_LOCATION"
    },
```
The problem is, this node, `Paris`, is already deleted(merged), We have multiple nodes labeled `Paris`, as this node contains a collection, so it crashed?
#### Graph obtained with Neo4J
With the data acquired by `Gauthier`, there are 2642 nodes, with a *super* node:
```json
{
  "type": "node",
  "id": "2677",
  "labels": [
    "NamespacePrefixDefinition"
  ],
  "properties": {
    "http://www.w3.org/1999/02/22-rdf-syntax-ns#": "rdf",
    "http://purl.org/vocab/relationship/": "ns6",
    "http://www.wikidata.org/prop/direct/": "ns2",
    "http://d-nb.info/standards/elementset/gnd#": "ns5",
    "http://dbpedia.org/ontology/": "ns4",
    "http://vocab.getty.edu/ontology#": "ns3",
    "http://schema.org/": "sch",
    "http://www.w3.org/2000/01/rdf-schema#": "rdfs",
    "http://purl.org/dc/elements/1.1/": "dc",
    "http://purl.org/dc/terms/": "dct",
    "http://www.w3.org/2002/07/owl#": "owl",
    "http://www.w3.org/2006/vcard/ns#": "ns0",
    "http://www.w3.org/2004/02/skos/core#": "skos",
    "http://www.w3.org/ns/shacl#": "sh",
    "http://xmlns.com/foaf/0.1/": "ns1"
  }
}
```
and 6895 relations. 

In the `JSON` graph from connection lens, we have 7512 nodes and 11890 edges.<br>
With respect to the number of nodes and edges, Neo4J has a better performance on the simplification than our current algorithm.<br>
Clearly this may reorient our direction and leads to a new algorithm, i.e.<br>
We can just replace the `"labels":["Resource"]` and with a value in the **properties** by a simple ranking of labels (If we suppose the simplification of Neo4J is a good)<br>
As for the edges, Neo4J provided a good label, only some replacements need to be done in order to improve its readability.<br>
If necessary, It is feasible to write the code to convert this result to a real JSON file (But with embedded structure, it will not be compatible with Irène's code), we need to discuss following points:<br>
1. Are the nodes merged by Neo4j what we want, are they over-merged or can they be further merged?
2. Are there always a label in the `properties` that can well represent this node?


#### The Java implementation

```bash

```

It doesn't work with RDF graphs(graphs with loops)
### [RDFQuotient](https://rdfquotient.inria.fr/)

# Dataset Construction
## get data from YAGO4
Here we want to extract all entities in `YAGO4` that has a `org.wikipedia.fr` link, then we gather all their appearances in the `YAGO4` to construct a dataset.

Note:
YAGO4 has embedded structures like : 
```
<<        <http://yago-knowledge.org/resource/Ruth_Milles>        <http://schema.org/deathPlace>        <http://yago-knowledge.org/resource/Rome>        >>        <http://schema.org/startDate>        "1932"^^<http://www.w3.org/2001/XMLSchema#gYear>        .
<<        <http://yago-knowledge.org/resource/Shadowrun>        <http://schema.org/publisher>        <http://yago-knowledge.org/resource/Catalyst_Game_Labs>        >>        <http://schema.org/startDate>   2007-06"^^<http://www.w3.org/2001/XMLSchema#gYearMonth>.
```
"The .ntx file is Extended RDF which goes beyond the simple N-triple format.

*Interestingly*, in another file, she dies in another year
```
[xizhang@cedar006 2020-02-24]$ grep 'Ruth_Milles' *.nt
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/deathPlace>	<http://yago-knowledge.org/resource/Rome>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/knowsLanguage>	<http://yago-knowledge.org/resource/Swedish_language>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/hasOccupation>	<http://yago-knowledge.org/resource/sculptor_Q1281618>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/hasOccupation>	<http://yago-knowledge.org/resource/Writer>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/nationality>	<http://yago-knowledge.org/resource/Sweden>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/birthPlace>	<http://yago-knowledge.org/resource/Vallentuna_parish_Q7102931>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Académie_Colarossi>.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Konstfack>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Royal_Swedish_Academy_of_Fine_Arts>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/givenName>	<http://yago-knowledge.org/resource/Ruth_(given_name)>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/familyName>	<http://yago-knowledge.org/resource/Milles>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/homeLocation>	<http://yago-knowledge.org/resource/Rome>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/deathDate>	"1941-02-11"^^<http://www.w3.org/2001/XMLSchema#date>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/birthDate>	"1873-04-19"^^<http://www.w3.org/2001/XMLSchema#date>	.
```
### find all entities with `org.wikipedia.fr`
```bash
grep 'fr\.[Ww]ikipedia\.org' *.nt > cnm
```

```bash
grep -o '\/resource\/([^>]*)' cnm > cnmEntities
```
### delete duplicate entities
```bash
sort cnmEntities -o cnmEntitiesUniq | uniq
```
### for those entities, find all related informations
We cannot just use `grep --file=***` here:
```bash
[xizhang@cedar006 2020-02-24]$ grep --file=cnmEntitiesUniq *.nt > results
grep: memory exhausted
```
I have found around 1.7 million distinct entities with "org.wikipedia.fr", if I do "grep --file=distinctEntities yago*" to gather all info, it will cause "memory exhausted", but if I do "grep" entity by entity, it will take around one year to finish. like the following:
```bash
#!/bin/bash
input="cnmEntitiesUniq"
echo -n > input
mkdir tmp
i=0
while IFS= read -r line
do
  touch tmp/results${i}
  grep "$line" *.nt > tmp/results${i}
  echo "${line} searched"
  let "i++"
done < "$input"
```
with Python and hash set
```python
import os
import re

entity_set = set(line.strip() for line in open('cnmEntitiesUniq',"r", encoding="utf-8"))
with open('exp_file',"w", encoding="utf-8") as fe:
    for filename in os.listdir(os.getcwd()):
        if filename.endswith(".nt"):
            with open(filename,"r", encoding="utf-8") as fp:
                line = fp.readline()
                while line:
                    m = re.search('(\/resource\/[^>]*)', line)
                    if m is not None:
                        p = m.group(1)
                        if m.group(1) in entity_set:
                            fe.write(line)
                    line = fp.readline()
            fp.close()
            print("file "+ filename + " finished.")
fe.close()
```
it terminates in 20 mins
```bash
[xizhang@cedar006 2020-02-24]$ python3 test.py
file yago-wd-class.nt finished.
file yago-wd-facts.nt finished.
file yago-wd-full-types.nt finished.
file yago-wd-labels.nt finished.
file yago-wd-sameAs.nt finished.
file yago-wd-schema.nt finished.
file yago-wd-shapes.nt finished.
file yago-wd-simple-types.nt finished.
[xizhang@cedar006 2020-02-24]$ wc -l exp_file
103647042 exp_file
```
Using Postgres
```sql
postgres= create table triples (s varchar, p varchar, o varchar, dot varchar);
CREATE TABLE
postgres= copy triples from '/data/yago4/2020-02-24/yago-wd-facts.nt';

COPY 63758903
ostgres= select * from triples limit 20;
                                               s
               |                        p                         |
                       o                                     | dot
--------------------------------------------------------------------------------
---------------+--------------------------------------------------+-------------
-------------------------------------------------------------+-----
 <http://yago-knowledge.org/resource/EAAT3_Q11856447>
               | <http://bioschemas.org/isEncodedByBioChemEntity> | <http://yago
-knowledge.org/resource/Excitatory_amino_acid_transporter_3> | .
 <http://yago-knowledge.org/resource/Bromo_adjacent_homology_domain_containing_1
>              | <http://bioschemas.org/isEncodedByBioChemEntity> | <http://yago
-knowledge.org/resource/BAHD1_Q18036571>                     | .
 <http://yago-knowledge.org/resource/Nerve_growth_factor_IB>
               | <http://bioschemas.org/isEncodedByBioChemEntity> | <http://yago
-knowledge.org/resource/NH41>                                | .
 <http://yago-knowledge.org/resource/Reelin>

postgres= create table cnmentities (entity varchar);
ERROR:  relation "cnmentities" already exists
postgres= copy cnmentities from '/data/yago4/2020-02-24/cnmEntitiesUniq';
COPY 1785285
postgres= select * from cnmentities limit 10;
                   entity
--------------------------------------------
 /resource/!!!
 /resource/!!!_(album)
 /resource/!Kung_languages
 /resource/!Mediengruppe_Bitnik_Q48775282
 /resource/!T.O.O.H.!
 /resource/!Wowow!
 /resource/!_(The_Dismemberment_Plan_album)
 /resource/$100,000_Q19543496
 /resource/$100,000_infield
 /resource/$10_Q10843661
(10 rows)
postgres= select t.s, t.p, t.o from triples t, cnmentities e where t.s like '%'||e.entity||'%' limit 20;
                                s                                |               p                |                            o

-----------------------------------------------------------------+--------------------------------+------------------------------
----------------------------
 <http://yago-knowledge.org/resource/Laélia_Véron_Q67408572>   | <http://schema.org/birthPlace> | <http://yago-knowledge.org/resour
ce/Crest,_Drôme>
 <http://yago-knowledge.org/resource/Laélia_Véron_Q67408572>   | <http://schema.org/birthPlace> | <http://yago-knowledge.org/resour
ce/Crest,_Drôme>
 <http://yago-knowledge.org/resource/Laélia_Véron_Q67408572>   | <http://schema.org/birthPlace> | <http://yago-knowledge.org/resour
ce/Crest,_Drôme>
 <http://yago-knowledge.org/resource/Laélia_Véron_Q67408572>   | <http://schema.org/birthPlace> | <http://yago-knowledge.org/resour
ce/Crest,_Drôme>
 <http://yago-knowledge.org/resource/Edmond_Bapst_Q1285519>      | <http://schema.org/birthPlace> | <http://yago-knowledge.org/re
source/Paris>
 <http://yago-knowledge.org/resource/Edmond_Bapst_Q1285519>      | <http://schema.org/birthPlace> | <http://yago-knowledge.org/re
source/Paris>
 <http://yago-knowledge.org/resource/Laurent_Marceline_Q3219364> | <http://schema.org/birthPlace> | <http://yago-knowledge.org/re
source/Paris>
```

#### find most frequent predicates
```python
mport re
import collections

cnt = collections.Counter()
with open('exp_file',"r", encoding="utf-8") as fe:
    line = fe.readline()
    while line:
        m = re.search('<([^<>]*)>[^<]*<([^<>]*)>',line)
        if m is not None:
            p = m.group(2)
            cnt[p] += 1
        line = fe.readline()
fe.close()
with open('edge_count','w',encoding="utf-8") as fr:
    fr.write("Predicate Count\n")
    for c,k in cnt.most_common():
        fr.write(c+" "+str(k)+"\n")
print(cnt)
```

```bash
[xizhang@cedar006 2020-02-24]$ head edge_count
Predicate Count
http://www.w3.org/2000/01/rdf-schema#label 33547769
http://www.w3.org/2000/01/rdf-schema#comment 23854212
http://schema.org/sameAs 17479845
http://schema.org/alternateName 7259041
http://www.w3.org/1999/02/22-rdf-syntax-ns#type 6921180
http://www.w3.org/2002/07/owl#sameAs 3773345
http://schema.org/memberOf 1179234
http://schema.org/hasOccupation 840251
http://schema.org/image 750119
```
#### find most frequent nodes
*here, the nodes like labels/comments are not counted, only entities are counted*
```python
import re
import collections
import codecs

cnt = collections.Counter()
with open('exp_file',"r", encoding="utf-8") as fe:
    line = fe.readline()
    while line:
        #search the source
        m = re.search('<([^<>]*)>[^<]*<([^<>]*)>',line)
        #search the destination
        n = re.search('<([^<>]*)>[^<]*<([^<>]*)[^<]*\s<([^<>]*)>',line)
        if m is not None:
            p = m.group(1)
            cnt[p] += 1
        if n is not None:
            q = n.group(3)
            cnt[q] += 1
        line = fe.readline()
fe.close()
print("search finished")
with open('node_count','wb') as fr:
    fr.write(("Node Count\n").encode('utf-8'))
    for c,k in cnt.most_common():
        fr.write(c.encode('utf-8'))
        fr.write((" "+str(k)+"\n").encode('utf-8'))
fr.close()
```
the result
```bash
[xizhang@cedar006 2020-02-24]$ head node_count
Node Count
http://schema.org/Thing 1837705
http://schema.org/Person 592238
http://yago-knowledge.org/resource/Human 590824
http://schema.org/Place 566068
http://yago-knowledge.org/resource/male_Q6581097 483770
http://schema.org/AdministrativeArea 325072
http://schema.org/Organization 286579
http://schema.org/CreativeWork 269489
http://bioschemas.org/Taxon 255199
```
#### find most frequent types
```python
import re
import collections

cnt = collections.Counter()
with open('exp_file',"r", encoding="utf-8") as fe:
    line = fe.readline()
    while line:
        #search the source
        m = re.search('#type',line)
        #search the destination
        n = re.search('<([^<>]*)>[^<]*<([^<>]*)[^<]*\s<([^<>]*)>',line)
        if m is not None:
            if n is not None:
                q = n.group(3)
                cnt[q] += 1
        line = fe.readline()
fe.close()
print("search finished")
with open('type_count','wb') as fr:
    fr.write(("Type Count\n").encode('utf-8'))
    for c,k in cnt.most_common():
        fr.write(c.encode('utf-8'))
        fr.write((" "+str(k)+"\n").encode('utf-8'))
fr.close()
```
result
```
[xizhang@cedar006 2020-02-24]$ head type_count
Type Count
http://schema.org/Thing 1837705
http://schema.org/Person 592238
http://yago-knowledge.org/resource/Human 590824
http://schema.org/Place 566068
http://schema.org/AdministrativeArea 325072
http://schema.org/Organization 286579
http://schema.org/CreativeWork 269489
http://bioschemas.org/Taxon 255199
http://schema.org/Corporation 167227
```
#### By Postgres
**load the data**
```sql
postgres= CREATE TABLE yago_output (s varchar, p varchar, o varchar, dot varchar);
CREATE TABLE
postgres= COPY yago_output FROM '/data/yago4/2020-02-24/exp_file';
```
**count predicates**
```sql
postgres= select p, count(*) as cnt from yago_output group by p order by cnt desc;
```

```
                           p                            |   cnt
--------------------------------------------------------+----------
 <http://www.w3.org/2000/01/rdf-schema#label>           | 33547769
 <http://www.w3.org/2000/01/rdf-schema#comment>         | 23854212
 <http://schema.org/sameAs>                             | 17479845
 <http://schema.org/alternateName>                      |  7259041
 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>      |  6921180
 <http://www.w3.org/2002/07/owl#sameAs>                 |  3773345
 <http://schema.org/hasOccupation>                      |   829065
 <http://schema.org/image>                              |   750065
 <http://schema.org/nationality>                        |   582650
 <http://schema.org/containedInPlace>                   |   575822
 <http://schema.org/memberOf>                           |   556325
 <http://schema.org/actor>                              |   530058
 <http://schema.org/birthDate>                          |   524214
 <http://schema.org/birthPlace>                         |   491674
 <http://schema.org/givenName>                          |   485594
 <http://schema.org/gender>                             |   484089
 <http://schema.org/geo>                                |   397398
 <http://schema.org/location>                           |   279999
 <http://schema.org/url>                                |   277453
 <http://schema.org/deathDate>                          |   253830
 <http://schema.org/deathPlace>                         |   218008
 <http://schema.org/containsPlace>                      |   204286
```
### explore a path
first search 
```
[xizhang@cedar006 2020-02-24]$ grep Ruth_Milles exp_file
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/deathPlace>	<http://yago-knowledge.org/resource/Rome>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/knowsLanguage>	<http://yago-knowledge.org/resource/Swedish_language>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/hasOccupation>	<http://yago-knowledge.org/resource/sculptor_Q1281618>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/hasOccupation>	<http://yago-knowledge.org/resource/Writer>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/nationality>	<http://yago-knowledge.org/resource/Sweden>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/birthPlace>	<http://yago-knowledge.org/resource/Vallentuna_parish_Q7102931>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Académie_Colarossi>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Konstfack>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Royal_Swedish_Academy_of_Fine_Arts>.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/givenName>	<http://yago-knowledge.org/resource/Ruth_(given_name)>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/familyName>	<http://yago-knowledge.org/resource/Milles>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/homeLocation>	<http://yago-knowledge.org/resource/Rome>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/deathDate>	"1941-02-11"^^<http://www.w3.org/2001/XMLSchema#date>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/birthDate>	"1873-04-19"^^<http://www.w3.org/2001/XMLSchema#date>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/image>	<http://commons.wikimedia.org/wiki/Special:FilePath/Ruth%20Milles%201890.jpg>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://yago-knowledge.org/resource/Human>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alternateName>	"Ruth Anna Maria Milles"@de	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/alternateName>	"Ruth Anna Maria Milles"@sv	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@ast	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@ca	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@da	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@de	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@en	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@es	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@fi	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@fr	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@ga	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@it	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@nb	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@nl	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@nn	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@pt	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@sl	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@sq	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ruth Milles"@sv	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#label>	"Ռութ Միլլես"@hy	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"Swedish sculptor and writer (1873-1941)"@en	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"Zweeds schrijfster (1873-1941)"@nl	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"schwedische Bildhauerin und Autorin"@de	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"svensk författare och skulptör"@sv	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"svensk skribent"@da	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"svensk skribent"@nb	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"svensk skribent"@nn	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2000/01/rdf-schema#comment>	"نویسنده سوئدی"@fa	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2002/07/owl#sameAs>	<http://www.wikidata.org/entity/Q4967393>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/2002/07/owl#sameAs>	<http://dbpedia.org/resource/Ruth_Milles>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/sameAs>	"https://da.wikipedia.org/wiki/Ruth_Milles"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/sameAs>	"https://en.wikipedia.org/wiki/Ruth_Milles"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/sameAs>	"https://fr.wikipedia.org/wiki/Ruth_Milles"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/sameAs>	"https://hy.wikipedia.org/wiki/%D5%8C%D5%B8%D6%82%D5%A9_%D5%84%D5%AB%D5%AC%D5%AC%D5%A5%D5%BD"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://schema.org/sameAs>	"https://sv.wikipedia.org/wiki/Ruth_Milles"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Person>	.
<http://yago-knowledge.org/resource/Ruth_Milles>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Thing>	.
```
I think from the results, `alumniOf École_des_Beaux-Arts` is quite interesting. So,
```
[xizhang@cedar006 2020-02-24]$ grep École_des_Beaux-Arts exp_file
<http://yago-knowledge.org/resource/Life_Class_at_the_École_des_Beaux-Arts_Q47263482>	<http://schema.org/creator>	<http://yago-knowledge.org/resource/Albert_Marquet>	.
<http://yago-knowledge.org/resource/Ödön_Márffy>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Dwight_William_Tryon>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Jean_Dewasne_Q127266>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Chaim_Soutine>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Henry_Hudson_Kitson>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Jo_Davidson>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Marc_Moallic_Q19629475>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Charles_Gill_(artist)>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Charles_Mewès>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>	.
<http://yago-knowledge.org/resource/Pierre_Chapo>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/École_des_Beaux-Arts>
```
```sql
select y1.s, y1.p, y1.o, y2.p y2.o from yago_output y1, yago_output y2 where y1.o = y2.s
```
### load the data with `RDF-DB`
It took around 7 hours for `RDF-DB` to load the dataset, the whole log is as follows:
```
$ java -jar ontosql-rdfdb-1.0.11.jar -input "/data/yago4/2020-02-24/exp_file" -pf "conf/DataLoading.properties"
16:52:37,459  INFO DataLoading:244 - Loading configuration...
16:52:37,464  INFO DataLoading:186 - Creating database testrdfdb and loading triples...
16:52:37,552  INFO LoadTriplesToDatabase:118 - Creating (or dropping and re-creating) database
16:52:37,555  INFO LoadTriplesToDatabase:155 - Database testrdfdb exists...
16:52:42,226  INFO LoadTriplesToDatabase:122 - Database testrdfdb dropped
16:52:42,527  INFO LoadTriplesToDatabase:124 - Database testrdfdb created
17:03:19,800  INFO DataLoading:189 - Data loaded
17:03:19,800  INFO DataLoading:191 - Encoding dictionary and encoded triples...
17:03:19,884  INFO Runner:207 - 	Creating dictionary table...
17:03:19,889  INFO Runner:213 - 	Dictionary table created OK.
17:03:19,889  INFO Runner:215 - 	Populating dictionary table...
17:08:58,108  INFO Runner:226 - 	Dictionary table populated OK.
17:08:58,108  INFO Runner:228 - 	Creating dictionary indexes...
17:10:22,511  INFO Runner:234 - 	Dictionary indexes created OK.
17:10:22,514  INFO Runner:177 - 	Encoded triples table created OK.
17:10:22,514  INFO Runner:179 - 	Populating encoded triples table...
17:26:33,016  INFO Runner:190 - 	Encoded triples table populated OK.
17:26:33,017  INFO Runner:192 - 	Creating encoded triples indexes...
17:36:50,429  INFO Runner:198 - 	Encoded triples indexes created OK.
17:36:50,430  INFO DataLoading:193 - Dictionary encoded and encoded triples created.
17:36:50,430  INFO DataLoading:194 - Encoding constraint into dictionary...
17:38:51,073  INFO DataLoading:199 - Saturating graph with type RDFS_SAT...
17:38:51,074  INFO DataLoading:85 - Fetching RDF type dictionary-encode value...
23:02:41,430  INFO DataLoading:108 - RDF graph saturation took 18810508 ms
23:02:41,430  INFO DataLoading:201 - Graph saturation finished
23:02:41,430  INFO DataLoading:206 - Conversing graph schema...
23:06:25,085  INFO Runner:40 - 	Processing roles...
23:12:31,206  INFO Runner:114 - Loaded 130 role entries in 366120 ms.
23:12:31,207  INFO Runner:47 - 	Roles processed OK.
23:13:05,996  INFO Runner:114 - Loaded 7889 concept entries in 34789 ms.
23:13:05,997  INFO DataLoading:208 - Conversing finished
23:13:05,997  INFO DataLoading:212 - Generating statistic tables...
23:29:25,485  INFO DataLoading:214 - Statistic tables generated
23:29:25,489  INFO DataLoading:220 - Running ANALYZE
23:30:06,458  INFO DataLoading:222 - ANALYZE finished
23:30:06,459  INFO DataLoading:224 - Everything done
```
```
postgres=# \c testrdfdb
```
```sql
CREATE VIEW with_reverse AS 
SELECT s, p, o 
FROM encoded_triples 
UNION 
SELECT o AS s, p, s AS o 
FROM encoded_triples;
```

find paths with length 3 with dataset:

```sql
SELECT *
FROM with_reverse v1
JOIN with_reverse v2
  ON v2.s = v1.o
JOIN with_reverse v3
  ON v3.s = v2.o
WHERE v1.s = 46133435 /*Macron*/
  AND v3.o = 45329413 /*BNP_Paribas*/;
```
better visualization:
```sql
SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o
FROM with_reverse v1
JOIN with_reverse v2
  ON v2.s = v1.o
JOIN with_reverse v3
  ON v3.s = v2.o
WHERE v1.s = 46133435 /*Macron*/
  AND v3.o = 45329413 /*BNP_Paribas*/;
```

```sql
\copy (SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o WHERE v1.s = 46133435 /*Macron*/ AND v3.o = 45329413 /*BNP_Paribas*/) to '/data/yago4/tmp.csv' CSV HEADER;
```
Visualize the results
```
<http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://www.w3.org/2000/01/rdf-schema#label>
 """������������������""@zh"
 <http://www.w3.org/2000/01/rdf-schema#label>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
<http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://www.w3.org/2000/01/rdf-schema#label>
 """������������������""@ja"
 <http://schema.org/alternateName>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://schema.org/alternateName>
 """���������������������""@zh"
 <http://schema.org/alternateName>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/nationality>
 <http://yago-knowledge.org/resource/France>
 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
 <http://schema.org/Place>
 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
```
We Can see that `RDF_DB` has a serious encoding problem
```sql

```
But here let us just ignore all `#type` and `#lable`

```
   key    |                          value
----------+----------------------------------------------------------
 39806236 | <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
 16098780 | <http://schema.org/gender>
 55189694 | <http://schema.org/knowsLanguage>
 42429573 | <http://schema.org/nationality>
 66171377 | <http://www.w3.org/2000/01/rdf-schema#comment>
```
so we modify the `SQL`:
```sql
SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o
FROM with_reverse v1
JOIN with_reverse v2
  ON v2.s = v1.o
JOIN with_reverse v3
  ON v3.s = v2.o
WHERE v1.s = 46133435 /*Macron*/
  AND v3.o = 45329413 /*BNP_Paribas*/
  AND NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485 OR v3.p = 15756171 OR v3.p = 41834368 OR v3.p=26431485);
```
results
There is no results
visualize
```sql
\copy (SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o WHERE v1.s = 46133435 /*Macron*/ AND v3.o = 45329413 /*BNP_Paribas*/ AND NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485 OR v3.p = 15756171 OR v3.p = 41834368 OR v3.p=26431485)) to '/data/yago4/tmp1.csv' CSV HEADER;
COPY 0
```
Try length 2
```sql
\copy (SELECT v1.s, v1.p, v1.o, v2.p, v2.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o WHERE v1.s = 46133435 /*Macron*/ AND v2.o = 45329413 /*BNP_Paribas*/ AND NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485)) to '/data/yago4/tmp1.csv' CSV HEADER;
COPY 0
```
length 4
```sql
\copy (SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o, v4.p, v4.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o JOIN with_reverse v4 ON v3.o = v4.s WHERE v1.s = 46133435 /*Macron*/ AND v4.o = 45329413 /*BNP_Paribas*/ AND NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485 OR v3.p = 15756171 OR v3.p = 41834368 OR v3.p=26431485 OR v4.p = 15756171 OR v4.p = 41834368 OR v4.p=26431485)) to '/data/yago4/tmp1.csv' CSV HEADER;
```
```sql 
\copy (SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o, v4.p, v4.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o JOIN with_reverse v4 ON v3.o = v4.s WHERE v1.s = 46133435 /*Macron*/ AND v4.o = 45329413 /*BNP_Paribas*/ AND NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485 OR v3.p = 15756171 OR v3.p = 41834368 OR v3.p=26431485 OR v4.p = 15756171 OR v4.p = 41834368 OR v4.p=26431485)) to '/data/yago4/tmp1.csv' CSV HEADER;
COPY 97
```
finally something meaningful
```
<http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Jacques_de_Fouchier_Q11985034>
 <http://schema.org/founder>
 <http://yago-knowledge.org/resource/Cetelem>
 <http://schema.org/parentOrganization>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Jacques_de_Larosi��re>
 <http://schema.org/memberOf>
 <http://yago-knowledge.org/resource/Group_of_Thirty>
 <http://schema.org/sponsor>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/award>
 <http://yago-knowledge.org/resource/Charlemagne_Prize>
 <http://schema.org/award>
 <http://yago-knowledge.org/resource/Jean-Claude_Trichet>
 <http://schema.org/memberOf>
 <http://yago-knowledge.org/resource/Group_of_Thirty>
 <http://schema.org/sponsor>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Jean-Claude_Trichet>
 <http://schema.org/memberOf>
 <http://yago-knowledge.org/resource/Group_of_Thirty>
 <http://schema.org/sponsor>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Lyc��e_Henri-IV>
 <http://schema.org/location>
 <http://yago-knowledge.org/resource/Abbey_of_Saint_Genevieve>
 <http://www.w3.org/2000/01/rdf-schema#comment>
 """���������� ���� ����������""@ar"
 <http://www.w3.org/2000/01/rdf-schema#comment>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
 <http://yago-knowledge.org/resource/Emmanuel_Macron>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Sciences_Po>
 <http://schema.org/alumniOf>
 <http://yago-knowledge.org/resource/Christian_Noyer>
 <http://schema.org/memberOf>
 <http://yago-knowledge.org/resource/Group_of_Thirty>
 <http://schema.org/sponsor>
 <http://yago-knowledge.org/resource/BNP_Paribas>
 .
```
All paths with length 3
```sql
SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o
FROM with_reverse v1
JOIN with_reverse v2
  ON v2.s = v1.o
JOIN with_reverse v3
  ON v3.s = v2.o
WHERE NOT (v1.p = 15756171 OR v1.p = 41834368 OR v1.p= 26431485 OR v2.p = 15756171 OR v2.p = 41834368 OR v2.p=26431485 OR v3.p = 15756171 OR v3.p = 41834368 OR v3.p=26431485);
Processus arrêté
```

#### Find all paths from length 1 to length 4 with filtering:
```python
import psycopg2
import unidecode
con = psycopg2.connect(database="yago", user="postgres", password="", host="127.0.0.1", port="5432")
con.set_client_encoding('UTF8')
cur = con.cursor()
presidents = [#
"<http://yago-knowledge.org/resource/Brigitte_Macron>",
"<http://yago-knowledge.org/resource/François_Hollande>",
"<http://yago-knowledge.org/resource/Nicolas_Sarkozy>",
"<http://yago-knowledge.org/resource/Jacques_Chirac>"]
companies = ["<http://yago-knowledge.org/resource/BNP_Paribas>",
"<http://yago-knowledge.org/resource/Carrefour>",
"<http://yago-knowledge.org/resource/Crédit_Agricole>",
"<http://yago-knowledge.org/resource/Électricité_de_France>",
"<http://yago-knowledge.org/resource/Engie>",
"<http://yago-knowledge.org/resource/Peugeot>",
"<http://yago-knowledge.org/resource/Société_Générale>",
"<http://yago-knowledge.org/resource/Renault>"]

filter_out = ["<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>",
 "<http://schema.org/gender>",
 "<http://schema.org/nationality>",
 "<http://schema.org/knowsLanguage>",
 "<http://www.w3.org/2000/01/rdf-schema#comment>"]


for president in presidents:
    cur.execute("SELECT key FROM dictionary WHERE value = '" + president + "';")
    key_president = cur.fetchone()[0]
    for company in companies:
        filename = "paths/"+unidecode.unidecode(president.rsplit('/',1)[-1][:-1])+'_'+unidecode.unidecode(company.rsplit('/',1)[-1][:-1])
        if filename in ["paths/Emmanuel_Macron_BNP_Paribas","paths/Emmanuel_Macron_Carrefour"]:
            continue
        with open(filename, mode="wb") as file:
            print("file "+ filename+" created.")
            cur.execute("SELECT key FROM dictionary WHERE value = '" + company + "';")
            key_company = cur.fetchone()[0]
            file.write((president+" + "+company+ " length 1" +"\n").encode('utf-8'))
            print(str(key_president) + " " + str(key_company))
            cur.execute("SELECT s, p, o FROM with_reverse v1  WHERE v1.s = "+str(key_president)+" AND v1.o ="+str(key_company)+";")
            rows = cur.fetchall()
            for row in rows:
                if all(s not in row for s in filter_out):
                    for member in row:
                        cur.execute("SELECT value FROM dictionary WHERE key = '" + str(member) + "';")
                        file.write((cur.fetchone()[0] + " ").encode('utf-8'))
                    file.write((".\n").encode('utf-8'))
            print("length 1 finished")

            file.write((president + " + " + company + " length 2" + "\n").encode('utf-8'))
            cur.execute("SELECT v1.s, v1.p, v1.o, v2.p, v2.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o WHERE v1.s =" + str(key_president) + " AND v2.o =" + str(key_company) + ";")
            rows = cur.fetchall()
            for row in rows:
                if all(s not in row for s in filter_out):
                    for member in row:
                        cur.execute("SELECT value FROM dictionary WHERE key = '" + str(member) + "';")
                        file.write((cur.fetchone()[0] + " ").encode('utf-8'))
                    file.write((".\n").encode('utf-8'))
            print("length 2 finished")

            file.write((president + " + " + company + " length 3" + "\n").encode('utf-8'))
            cur.execute("SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o WHERE v1.s = "+str(key_president)+" AND v3.o ="+str(key_company)+" AND NOT (v1.p = 39806236 OR v1.p = 16098780 OR v1.p= 55189694 OR v1.p = 42429573 OR v1.p=66171377 OR v2.p = 39806236 OR v2.p = 16098780 OR v2.p= 55189694 OR v2.p = 42429573 OR v2.p=66171377 OR v3.p = 39806236 OR v3.p = 16098780 OR v3.p= 55189694 OR v3.p = 42429573 OR v3.p=66171377) ;")
            rows = cur.fetchall()
            for row in rows:
                if all(s not in row for s in filter_out):
                    for member in row:
                        cur.execute("SELECT value FROM dictionary WHERE key = '" + str(member) + "';")
                        file.write((cur.fetchone()[0] + " ").encode('utf-8'))
                    file.write((".\n").encode('utf-8'))
            print("length 3 finished")

            file.write((president + " + " + company + " length 4" + "\n").encode('utf-8'))
            cur.execute("SELECT v1.s, v1.p, v1.o, v2.p, v3.s, v3.p, v3.o, v4.p, v4.o FROM with_reverse v1 JOIN with_reverse v2 ON v2.s = v1.o JOIN with_reverse v3 ON v3.s = v2.o JOIN with_reverse v4 ON v3.o = v4.s WHERE v1.s =" + str(key_president) + " AND v4.o =" + str(key_company) + " AND NOT (v1.p = 39806236 OR v1.p = 16098780 OR v1.p= 55189694 OR v1.p = 42429573 OR v1.p=66171377 OR v2.p = 39806236 OR v2.p = 16098780 OR v2.p= 55189694 OR v2.p = 42429573 OR v2.p=66171377 OR v3.p = 39806236 OR v3.p = 16098780 OR v3.p= 55189694 OR v3.p = 42429573 OR v3.p=66171377 OR v4.p = 39806236 OR v4.p = 16098780 OR v4.p= 55189694 OR v4.p = 42429573 OR v4.p=66171377);")
            rows = cur.fetchall()
            for row in rows:
                if all(s not in row for s in filter_out):
                    for member in row:
                        cur.execute("SELECT value FROM dictionary WHERE key = '" + str(member) + "';")
                        file.write((cur.fetchone()[0] + " ").encode('utf-8'))
                    file.write((".\n").encode('utf-8'))
            print("length 4 finished")
        file.close()
        print("file " + filename + " finished.")
    print(unidecode.unidecode(president) + "finished")
print("All finished")
con.close()
```
results:
```
du -a
4	./Emmanuel_Macron_BNP_Paribas
8	./Emmanuel_Macron_Carrefour
4	./Brigitte_Macron_BNP_Paribas
4	./Emmanuel_Macron_Credit_Agricole
512	./Emmanuel_Macron_Electricite_de_France
488	./Emmanuel_Macron_Engie
4	./Emmanuel_Macron_Peugeot
4	./Emmanuel_Macron_Societe_Generale
40	./Emmanuel_Macron_Renault
4	./Brigitte_Macron_Carrefour
4	./Brigitte_Macron_Credit_Agricole
128	./Brigitte_Macron_Electricite_de_France
124	./Brigitte_Macron_Engie
4	./Brigitte_Macron_Peugeot
4	./Brigitte_Macron_Societe_Generale
8	./Brigitte_Macron_Renault
8	./Francois_Hollande_BNP_Paribas
12	./Francois_Hollande_Carrefour
4	./Francois_Hollande_Credit_Agricole
808	./Francois_Hollande_Electricite_de_France
768	./Francois_Hollande_Engie
8	./Francois_Hollande_Peugeot
8	./Francois_Hollande_Societe_Generale
56	./Francois_Hollande_Renault
20	./Nicolas_Sarkozy_BNP_Paribas
44	./Nicolas_Sarkozy_Carrefour
24	./Nicolas_Sarkozy_Credit_Agricole
28056	./Nicolas_Sarkozy_Electricite_de_France
26728	./Nicolas_Sarkozy_Engie
12	./Nicolas_Sarkozy_Peugeot
40	./Nicolas_Sarkozy_Societe_Generale
124	./Nicolas_Sarkozy_Renault
16	./Jacques_Chirac_BNP_Paribas
56	./Jacques_Chirac_Carrefour
40	./Jacques_Chirac_Credit_Agricole
51720	./Jacques_Chirac_Electricite_de_France
49284	./Jacques_Chirac_Engie
12	./Jacques_Chirac_Peugeot
40	./Jacques_Chirac_Societe_Generale
164	./Jacques_Chirac_Renault
159400	.
```
We can see that we still have a lot of results for certain pairs.
From observations, we can see that those pairs all have the same cause, **Engie** and **Electricité de France** all have `foundingLocation` which is `Paris`. And those two presidents all have `birthPlace` `Paris`. So I will filter out those results.
```python
with open("results", mode="r", encoding="utf-8") as file:
    line = file.readline()
    filter_out = ["<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>","<http://schema.org/gender>","<http://schema.org/nationality>","<http://schema.org/knowsLanguage>","<http://www.w3.org/2000/01/rdf-schema#comment>"]
    with open('results_filtered', 'wb') as fr:
        while line:
            if all(s not in line for s in filter_out):
                fr.write(line.encode('utf-8'))
            line = file.readline()
    fr.close()
file.close()
```
results
```
du -a
80      ./Emmanuel_Macron_Renault
72      ./Jacques_Chirac_Credit_Agricole
8       ./Emmanuel_Macron_Credit_Agricole
8       ./Emmanuel_Macron_BNP_Paribas
16      ./Emmanuel_Macron_Electricite_de_France
16      ./Emmanuel_Macron_Carrefour
16      ./Jacques_Chirac_Peugeot
264     ./Nicolas_Sarkozy_Renault
264     ./Jacques_Chirac_Electricite_de_France
80      ./Jacques_Chirac_Societe_Generale
32      ./Jacques_Chirac_BNP_Paribas
112     ./Francois_Hollande_Renault
8       ./Francois_Hollande_Credit_Agricole
8       ./Brigitte_Macron_Credit_Agricole
16      ./Brigitte_Macron_Renault
48      ./Nicolas_Sarkozy_Credit_Agricole
24      ./Francois_Hollande_Carrefour
8       ./Emmanuel_Macron_Societe_Generale
112     ./Jacques_Chirac_Carrefour
8       ./Emmanuel_Macron_Engie
64      ./Jacques_Chirac_Engie
8       ./filter_path.py
8       ./Emmanuel_Macron_Peugeot
80      ./Nicolas_Sarkozy_Societe_Generale
8       ./Brigitte_Macron_Electricite_de_France
8       ./Brigitte_Macron_Carrefour
32      ./Nicolas_Sarkozy_Engie
8       ./Francois_Hollande_Engie
24      ./Francois_Hollande_Electricite_de_France
8       ./Francois_Hollande_Peugeot
8       ./Brigitte_Macron_Engie
8       ./Brigitte_Macron_Societe_Generale
88      ./Nicolas_Sarkozy_Carrefour
16      ./Francois_Hollande_Societe_Generale
8       ./Brigitte_Macron_BNP_Paribas
112     ./Nicolas_Sarkozy_Electricite_de_France
40      ./Nicolas_Sarkozy_BNP_Paribas
16      ./Nicolas_Sarkozy_Peugeot
392     ./Jacques_Chirac_Renault
16      ./Francois_Hollande_BNP_Paribas
8       ./Brigitte_Macron_Peugeot
2160    .
```
This result seems good enough.

### Link with HAVTP

#### Find common entities

```python
from csv import DictReader

entity_set = set(line.strip() for line in open('cnmEntitiesUniq',"r", encoding="utf-8"))
with open('liste-hatvp.csv','r', encoding="utf-8") as fe:
    with open('shared_entities', 'wb') as fo:
        csv_dict_reader = DictReader(fe, delimiter=';')
        for row in csv_dict_reader:
            search1 = "/resource/"+row['prenom']+"_"+row['nom'].title()
            # search2 = "/resource/"+row['prenom']+"_"+row['nom'].title()+"_(politician)"
            # if search1 in entity_set:
            #     fo.write((search1+"\n").encode("utf-8"))
            #     entity_set.remove(search1)
            # elif search2 in entity_set:
            #     fo.write((search2+"\n").encode("utf-8"))
            #     entity_set.remove(search2)
            for s in entity_set.copy():
                if search1 in s and s in entity_set:
                    fo.write((s+"\n").encode("utf-8"))
                    entity_set.remove(s)
    fo.close()
fe.close()
```
result
```
[xizhang@cedar007 2020-02-24]$ wc -l shared_entities
2041 shared_entities
[xizhang@cedar007 2020-02-24]$ head -10 shared_entities
/resource/Abdallah_Hassani_Q41023061
/resource/Adrien_Morenas
/resource/Adrien_Quatennens
/resource/Adrien_Taquet
/resource/Agnès_Buzyn
/resource/Agnès_Canayer_Q18202568
/resource/Agnès_Evren
/resource/Agnès_Pannier-Runacher
/resource/Agnès_Thill
/resource/Aina_Kuric
```
Here we take an example `Adrien Taquet`
```
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/knowsLanguage>	<http://yago-knowledge.org/resource/French_language>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/hasOccupation>	<http://yago-knowledge.org/resource/Politician>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/nationality>	<http://yago-knowledge.org/resource/France>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/birthPlace>	<http://yago-knowledge.org/resource/16th_arrondissement_of_Paris>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/alumniOf>	<http://yago-knowledge.org/resource/Sciences_Po>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/familyName>	<http://yago-knowledge.org/resource/Taquet>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/memberOf>	<http://yago-knowledge.org/resource/La_République_En_Marche!>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/birthDate>	"1977-01-03"^^<http://www.w3.org/2001/XMLSchema#date>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/image>	<http://commons.wikimedia.org/wiki/Special:FilePath/Adrien%20taquet%2016467.jpg>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/gender>	<http://yago-knowledge.org/resource/male_Q6581097>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://yago-knowledge.org/resource/Human>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://www.w3.org/2002/07/owl#sameAs>	<http://www.wikidata.org/entity/Q30344625>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://www.w3.org/2002/07/owl#sameAs>	<http://dbpedia.org/resource/Adrien_Taquet>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/sameAs>	"https://en.wikipedia.org/wiki/Adrien_Taquet"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://schema.org/sameAs>	"https://fr.wikipedia.org/wiki/Adrien_Taquet"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Person>	.
<http://yago-knowledge.org/resource/Adrien_Taquet>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Thing>	.
```
And one of the 
Considering put HAVTP and YAGO4 in the same graph, I am considering for each declaration in HAVTP, we extract some information that correspond to entities in YAGO, make triples out of them and put them in the `exp_file`.


### problems loading YAGO4 with `rdflib`

#### `.` right in the third column instead of being the fourth one
```
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/_Q17628927>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/Santa_Muerte>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/_Q3211007>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/_Q12975871>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/_Q2097151>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/_Q3211263>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>.
```



#### Mixed usage of `\t` and `\s`
```python
[xizhang@cedar006 2020-02-24]$ python3 pathSPARQL.py
Traceback (most recent call last):
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 154, in parse
    self.parseline()
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 197, in parseline
    subject = self.subject()
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 224, in subject
    subj = self.uriref() or self.nodeid()
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 243, in uriref
    uri = self.eat(r_uriref).group(1)
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 218, in eat
    raise ParseError("Failed to eat %s at %s" % (pattern.pattern, self.line))
rdflib.plugins.parsers.ntriples.ParseError: Failed to eat <([^:]+:[^\s"<>]*)> at <http://yago-knowledge.org/resource/Day_5:_1:00_pm\xa0-_2:00_pm_Q52263123>	<http://schema.org/director>	<http://yago-knowledge.org/resource/Brad_Turner_(director)>	.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "pathSPARQL.py", line 5, in <module>
    result = g.parse("exp_file", format="nt")
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/graph.py", line 1078, in parse
    parser.parse(source, self, **args)
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/nt.py", line 26, in parse
    parser.parse(f)
  File "/home/cedar/xizhang/.local/lib/python3.6/site-packages/rdflib/plugins/parsers/ntriples.py", line 156, in parse
    raise ParseError("Invalid line: %r" % self.line)
rdflib.plugins.parsers.ntriples.ParseError: Invalid line: '<http://yago-knowledge.org/resource/Day_5:_1:00_pm\xa0-_2:00_pm_Q52263123>\t<http://schema.org/director>\t<http://yago-knowledge.org/resource/Brad_Turner_(director)>\t.'
```

ca
#### Space inside certain entities:
```
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/director>	<http://yago-knowledge.org/resource/Brad_Turner_(director)>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Mark_Sheppard>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Sean_Astin>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/James_Morrison_(actor)>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/John_Allen_Nelson>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Kim_Raver>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Channon_Roe>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Jean_Smart>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Angela_Sarafyan>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Kiefer_Sutherland>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Jude_Ciccolella>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Glenn_Morshower>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Penny_Balfour>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Roger_Cross>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Mary_Lynn_Rajskub>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Gregory_Itzin>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/actor>	<http://yago-knowledge.org/resource/Patrick_Bauchau>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/24_(season_5)>	<http://schema.org/hasPart>	<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/partOfSeason>	<http://yago-knowledge.org/resource/24_(season_5)>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/partOfSeries>	<http://yago-knowledge.org/resource/24_(TV_series)>	.
yago-wd-facts.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/inLanguage>	<http://yago-knowledge.org/resource/English_language>	.
yago-wd-full-types.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/TVEpisode>	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#label>	"13h00 - 14h00"@fr	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#label>	"Dalle 13:00 alle 14:00"@it	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#label>	"Day 5: 1:00 pm - 2:00 pm"@en	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#label>	"Tag 5: 13:00 Uhr – 14:00 Uhr"@de	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#comment>	"Folge von 24"@de	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#comment>	"aflevering van 24"@nl	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#comment>	"episode of 24 (S5 E7)"@en	.
yago-wd-labels.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2000/01/rdf-schema#comment>	"episodio della quinta stagione di 24"@it	.
yago-wd-sameAs.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/2002/07/owl#sameAs>	<http://www.wikidata.org/entity/Q52263123>	.
yago-wd-sameAs.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://schema.org/sameAs>	"https://fr.wikipedia.org/wiki/13h00_-_14h00_(%C3%A9pisode_de_24_Heures_chrono,_saison_5)"^^<http://www.w3.org/2001/XMLSchema#anyURI>	.
yago-wd-simple-types.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Thing>	.
yago-wd-simple-types.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/CreativeWork>	.
yago-wd-simple-types.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/TVEpisode>	.
yago-wd-simple-types.nt:<http://yago-knowledge.org/resource/Day_5:_1:00_pm -_2:00_pm_Q52263123>	<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>	<http://schema.org/Episode>	.
```
## load with `ConnectionLens`
### divide the dataset
```bash
split -dl 1000000 --additional-suffix=.nt exp_file_dd.nt split/split_exp
```
1M by 1M
```bash
a=0
b=0
filenames=$(ls /data/yago4/2020-02-24/split/ | grep 'split')
for filename in $filenames
do
        echo "$filename"
        if [ $a == $b ]
        then
                java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-e79ae4a-20200601-1813.jar -i "/data/yago4/2020-02-24/split/$filename" -DRDBMSDBName=mx_test61 -v -E
                a=1
        else
                java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-e79ae4a-20200601-1813.jar -i "/data/yago4/2020-02-24/split/$filename" -DRDBMSDBName=mx_test61 -v -E -n
        fi
        sleep 1
done
```
The first 1 M
```bash
[xizhang@cedar008 ~]$ ./tt.sh
split_exp00.nt
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
PER_INSTANCE registration of: file:/data/yago4/2020-02-24/split/split_exp00.nt
Edge spill
Node spill
Same-as edge spill
Balking out of drawing the graph (too many nodes).
Edge cache: CacheStats{hitCount=0, missCount=1616, loadSuccessCount=1616, loadExceptionCount=0, totalLoadTime=298675600, evictionCount=0}
Node cache: CacheStats{hitCount=4094498, missCount=3550, loadSuccessCount=3550, loadExceptionCount=0, totalLoadTime=259561477, evictionCount=0}
Incoming edge cache: CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadExceptionCount=0, totalLoadTime=0, evictionCount=0}
Outgoing edge cache: CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadExceptionCount=0, totalLoadTime=0, evictionCount=0}
In-degree cache: CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadExceptionCount=0, totalLoadTime=0, evictionCount=0}
Out-degree cache: CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadExceptionCount=0, totalLoadTime=0, evictionCount=0}
Total time: 363100 ms.
split_exp01.nt
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
PER_INSTANCE registration of: file:/data/yago4/2020-02-24/split/split_exp01.nt
Edge spill
Node spill
Edge spill
Same-as edge spill
Specificity spill
Balking out of drawing the graph (too many nodes).
Edge cache: CacheStats{hitCount=30625, missCount=1400331, loadSuccessCount=1400331, loadExceptionCount=0, totalLoadTime=112432819669, evictionCount=1346418}
Node cache: CacheStats{hitCount=8276620, missCount=1461134, loadSuccessCount=1461134, loadExceptionCount=0, totalLoadTime=76537720956, evictionCount=1411134}
Incoming edge cache: CacheStats{hitCount=0, missCount=654314, loadSuccessCount=654314, loadExceptionCount=0, totalLoadTime=88485944546, evictionCount=604314}
Outgoing edge cache: CacheStats{hitCount=0, missCount=654314, loadSuccessCount=654314, loadExceptionCount=0, totalLoadTime=131997949387, evictionCount=604314}
In-degree cache: CacheStats{hitCount=118878, missCount=71831, loadSuccessCount=71831, loadExceptionCount=0, totalLoadTime=3555131320, evictionCount=66831}
Out-degree cache: CacheStats{hitCount=381804, missCount=850617, loadSuccessCount=850617, loadExceptionCount=0, totalLoadTime=40677545690, evictionCount=845617}
Total time: 1161131 ms.
split_exp02.nt
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
PER_INSTANCE registration of: file:/data/yago4/2020-02-24/split/split_exp02.nt
Edge spill
Node spill
Specificity spill
Same-as edge spill
Edge spill
Edge spill
Same-as edge spill
Specificity spill
Balking out of drawing the graph (too many nodes).
Edge cache: CacheStats{hitCount=68587, missCount=3914436, loadSuccessCount=3914436, loadExceptionCount=0, totalLoadTime=372615522215, evictionCount=3860450}
Node cache: CacheStats{hitCount=14022880, missCount=4121525, loadSuccessCount=4121525, loadExceptionCount=0, totalLoadTime=216942437138, evictionCount=4071525}
Incoming edge cache: CacheStats{hitCount=3, missCount=1146777, loadSuccessCount=1146777, loadExceptionCount=0, totalLoadTime=358419303565, evictionCount=1096777}
Outgoing edge cache: CacheStats{hitCount=3, missCount=1146777, loadSuccessCount=1146777, loadExceptionCount=0, totalLoadTime=268514405843, evictionCount=1096777}
In-degree cache: CacheStats{hitCount=1747790, missCount=424033, loadSuccessCount=424033, loadExceptionCount=0, totalLoadTime=24312187775, evictionCount=414386}
Out-degree cache: CacheStats{hitCount=257005, missCount=1542240, loadSuccessCount=1542240, loadExceptionCount=0, totalLoadTime=80573403785, evictionCount=1534532}
Total time: 2499139 ms.
```

```
[xizhang@cedar008 connection-lens]$ java -jar core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-0506173-20200530-1234.jar -i /data/yago4/2020-02-24/exp_file_fix.nt -DRDBMSDBName=cnl_yago -v -E
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
PER_INSTANCE registration of: file:/data/yago4/2020-02-24/exp_file_fix.nt
Total time: 88300169 ms.
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.StringCoding.encode(StringCoding.java:350)
	at java.lang.String.getBytes(String.java:941)
	at org.postgresql.core.Utils.encodeUTF8(Utils.java:56)
	at org.postgresql.core.v3.SimpleParameterList.getV3Length(SimpleParameterList.java:318)
	at org.postgresql.core.v3.QueryExecutorImpl.sendBind(QueryExecutorImpl.java:1367)
	at org.postgresql.core.v3.QueryExecutorImpl.sendOneQuery(QueryExecutorImpl.java:1667)
	at org.postgresql.core.v3.QueryExecutorImpl.sendQuery(QueryExecutorImpl.java:1216)
	at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:351)
	at org.postgresql.jdbc.PgStatement.executeBatch(PgStatement.java:1019)
	at fr.inria.cedar.connectionlens.sql.RelationalStructure.executeUpdate(RelationalStructure.java:205)
	at fr.inria.cedar.connectionlens.sql.schema2.RelationalGraphSession$BatchSession.spillEdges(RelationalGraphSession.java:794)
	at fr.inria.cedar.connectionlens.sql.schema2.RelationalGraphSession$BufferedSession.addEdge(RelationalGraphSession.java:995)
	at fr.inria.cedar.connectionlens.ConnectionLens.lambda$register$3(ConnectionLens.java:354)
	at fr.inria.cedar.connectionlens.ConnectionLens$$Lambda$19/508198356.accept(Unknown Source)
	at fr.inria.cedar.connectionlens.source.RDFDataSource$TripleHandler.handleStatement(RDFDataSource.java:350)
	at org.openrdf.rio.ntriples.NTriplesParser.parseTriple(NTriplesParser.java:338)
	at org.openrdf.rio.ntriples.NTriplesParser.parse(NTriplesParser.java:192)
	at org.openrdf.rio.ntriples.NTriplesParser.parse(NTriplesParser.java:131)
	at fr.inria.cedar.connectionlens.source.RDFDataSource.traverseEdges(RDFDataSource.java:212)
	at fr.inria.cedar.connectionlens.ConnectionLens.register(ConnectionLens.java:366)
	at fr.inria.cedar.connectionlens.ConnectionLens.register(ConnectionLens.java:298)
	at fr.inria.cedar.connectionlens.Experiment.registrationTest(Experiment.java:339)
	at fr.inria.cedar.connectionlens.Experiment.<init>(Experiment.java:262)
	at fr.inria.cedar.connectionlens.Experiment.main(Experiment.java:788)
```
HMMMM whatever
####
/Users/cedar/Documents/Xin/cnl/connection-lens/core/data/rdf/small-macron-royal-hollande.nt

GFORGE/kwd-search-het/trunk/construction-paper/EXPERIMENTS/YAGO

GFORGE/kwd-search-het/trunk/construction-paper/EXPERIMENTS/YAGO/cluster-experiments_18062020

java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -lateIdx -i /data/yago4/2020-02-24/split/split_exp00.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-00.txt
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp01.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-01.txt
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp02.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-02.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp03.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-03.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp04.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-04.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp05.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-05.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp06.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-06.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp07.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-07.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -i /data/yago4/2020-02-24/split/split_exp08.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-08.txt 
java -jar connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-xin-branch-4b6b4dd-20200624-1617.jar -n -lateIdx -last -i /data/yago4/2020-02-24/split/split_exp09.nt -DRDBMSDBName=cl_yago -rs -f LATEX -v > yago-09.txt

commande.sh
```sql
copy (SELECT e.id, e.source, e.target, e.label, CASE WHEN s1.nin IS NULL then 1 ELSE s1.nin END AS nin, CASE WHEN s2.nout IS NULL then 1 ELSE s2.nout END AS nout FROM edges e LEFT OUTER JOIN specificity s1 ON e.target = s1.representative AND e.label = s1.edge_label LEFT OUTER JOIN specificiy s2 ON e.source=s2.representative AND e.label = s2.edge_label) to '/tmp/cc.csv' delimiter ',' csv header;
```

```sql
copy (SELECT e.id, e.source, e.target, e.label, 2/(CASE WHEN s1.nin IS NULL then 1 ELSE s1.nin END + CASE WHEN s2.nout IS NULL then 1 ELSE s2.nout END) AS specificity FROM edges e LEFT OUTER JOIN specificity s1 ON e.target = s1.representative AND e.label = s1.edge_label LEFT OUTER JOIN specificity s2 ON e.source=s2.representative AND e.label = s2.edge_label) to '/tmp/bb.csv' delimiter ',' csv header;
```

```sql
SELECT label, target where label http://www.w3.org/2000/01/rdf-schema#comment
```
cccc ohlala
```bash
a=0
b=0
filenames=$(ls /data/yago4/2020-02-24/split/ | grep 'split')
for filename in $filenames
do
        echo "$filename"
        if [ $a == $b ]
        then
                java -jar tmp/connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-b71e307-20200629-1749.jar -lateIdx -i "/data/yago4/2020-02-24/split/$filename" -DRDBMSDBName=cl_yago -rs -f LATEX -v > "tmpload/yago-$filename.txt"
                a=1
        elif [ "$filename" == *"9011"* ]
        then
                java -jar tmp/connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-b71e307-20200629-1749.jar -n -lateIdx -last -i "/data/yago4/2020-02-24/split/$filename" -DRDBMSDBName=cl_yago -rs -f LATEX -v > "tmpload/yago-$filename.txt"
        else
                java -jar tmp/connection-lens/core/target/connection-lens-core-full-0.6-SNAPSHOT-develop-b71e307-20200629-1749.jar -n -lateIdx -i "/data/yago4/2020-02-24/split/$filename" -DRDBMSDBName=cl_yago -rs -f LATEX -v > "tmpload/yago-$filename.txt"
        fi
        sleep 1
done
```
### Python Implementation
```python

import graph_tool as gt
import graph_tool.topology
import time
from collections import Counter
import numpy as np

threshold = 0.3
k = 0.7

def jaccard_similarity(list1, list2):
    intersection = len(list(set(list1).intersection(list2)))
    union = (len(list1) + len(list2)) - intersection
    return float(intersection) / union

# class NumericalToleranceMesh(object):
#     '''
#     tol.zero: consider floating point numbers less than this zero
#     tol.merge: when merging vertices, consider vertices closer than this
#                to be the same vertex. Here we use the same value (1e-8)
#                as SolidWorks uses, according to their documentation.
#     tol.planar: the maximum distance from a plane a point can be and
#                 still be considered to be on the plane
#     tol.facet_rsq: the minimum radius squared that an arc drawn from the
#                    center of a face to the center of an adjacent face can
#                    be to consider the two faces coplanar. This method is more
#                    robust than considering just normal angles as it is tolerant
#                    of numerical error on very small faces.
#     '''
#     def __init__(self, **kwargs):
#         self.zero      = 1e-12
#         self.merge     = 1e-8
#         self.planar    = 1e-5
#         self.facet_rsq = 1e8
#         self.fit       = 1e-2
#         self.id_len    = 6
#         self.__dict__.update(kwargs)

def group(values, min_len=2, max_len=np.inf):
    '''
    Return the indices of values that are identical

    Arguments
    ----------
    values:     1D array
    min_len:    int, the shortest group allowed
                All groups will have len >= min_length
    max_len:    int, the longest group allowed
                All groups will have len <= max_length

    Returns
    ----------
    groups: sequence of indices to form groups
            IE [0,1,0,1] returns [[0,2], [1,3]]
    '''
    values = np.asanyarray(values)
    order = values.argsort()
    values = values[order]
    dupe = np.greater(np.abs(np.diff(values)), 0)
    dupe_idx = np.append(0, np.nonzero(dupe)[0] + 1)
    dupe_len = np.diff(np.hstack((dupe_idx, len(values))))
    dupe_ok = np.logical_and(np.greater_equal(dupe_len, min_len),
                             np.less_equal(dupe_len, max_len))
    groups = [order[i:(i + j)] for i, j in zip(dupe_idx[dupe_ok], dupe_len[dupe_ok])]
    return groups

# class Record:
#     def __init__(self, record):
#         self.elements = set()
#         self.record = record
#     def add_element(self, vertex):
#         self.elements.add(vertex)
start1 = time.time()
edges = []
count = 0
egde_filter = ['<http://www.w3.org/2000/01/rdf-schema#comment>','<http://www.w3.org/2000/01/rdf-schema#label>','<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>','<http://schema.org/alternateName>','<http://schema.org/sameAs>','<http://www.w3.org/2002/07/owl#sameAs>']
with open("rdflib_yago.nt",'r',encoding="utf-8") as f:
#with open("../gtsplit00.nt", 'r', encoding="utf-8") as f:
    line = f.readline()
    while line:
        count +=1
        if count % 1000000 == 0:
            print(count)
        if not any(x in line for x in egde_filter):
            s,p,o,dot = line.split('\t')
            edges.append((s,o,p))
        line = f.readline()
g = gt.Graph()
g.edge_properties["edge_label"] = g.new_edge_property("string")
g.vertex_properties["node_label"] = g.add_edge_list(edges, hashed=True, string_vals=True, eprops=[g.edge_properties["edge_label"]])
del edges
f.close()
end1 = time.time()
print("Loading time: " + str(end1 -start1))
# SEPARATING INTERNAL AND THE LEAF NODES:
start2 = time.time()
internal_nodes = {}
leaf_nodes = set()
collections = set()
stack = []
records = {}
for e in g.edges():
    s = e.source()
    t = e.target()
    if s in internal_nodes:
        internal_nodes[s] += 1
    else:
        internal_nodes[s] = 1
    if s in leaf_nodes:
        leaf_nodes.remove(s)
    leaf_nodes.add(t)
end2 =time.time()
print("Seperation time: "+str(end2-start2))
# FINDING THE COLLECTIONS:
start3 = time.time()
for ni in internal_nodes:
    if internal_nodes[ni] > 1:
        labels = [g.edge_properties["edge_label"][x] for x in ni.out_edges()]
        oc_count = Counter(labels)
        per = oc_count.most_common(1)[0][1]/len(labels)
        if per > k:
            # sim = 0
            # count = 0
            # for e1 in ni.out_edges():
            #     for e2 in ni.out_edges():
            #         if g.vertex_index[e2.target()] > g.vertex_index[e1.target()]:
            #             if e1.target() not in internal_nodes and e2.target() not in internal_nodes:
            #                 if g.edge_properties["edge_label"][e1] == g.edge_properties["edge_label"][e2]:
            #                     sim += 1
            #                     count +=1
            #                 else:
            #                     count += 1
            #             else:
            #                 l1 = [g.edge_properties["edge_label"][x] for x in e1.target().out_edges()]
            #                 l2 = [g.edge_properties["edge_label"][x] for x in e2.target().out_edges()]
            #                 sim += jaccard_similarity(l1, l2)
            #                 count += 1
            #         else:
            #             sim += 1
            #             count += 1
            # sim = sim/count
            sim = 0
            leaves = []
            internals = []
            for e in ni.out_edges():
                if e in internal_nodes:
                    internals.append(e)
                else:
                    leaves.append(e)
            length = len(internals)+len(leaves)
            n = (length*length - length)/2
            labels_leaves = [g.edge_properties["edge_label"][x] for x in leaves]
            ll_count = Counter(labels_leaves)
            # similarities between leaf edges
            for a in ll_count:
                if ll_count[a] > 1:
                    sim += (ll_count[a]*ll_count[a]-ll_count[a])/2
            for e1 in internals:
                for e2 in internals:
                    if g.vertex_index[e2.target()] > g.vertex_index[e1.target()]:
                        l1 = [g.edge_properties["edge_label"][x] for x in e1.target().out_edges()]
                        l2 = [g.edge_properties["edge_label"][x] for x in e2.target().out_edges()]
                        sim += jaccard_similarity(l1, l2)
            sim = sim/n
            if sim > threshold:
                collections.add(ni)

for n in collections:
    stack.append([None,n])
end3 = time.time()
print("Collection time: "+str(end3-start3))
del internals
del internal_nodes
del labels
del labels_leaves
del leaves
del ll_count
del oc_count
#FINDING NODES IN CYCLES:
start4 = time.time()
sbb = graph_tool.topology.label_components(g, directed=True)[0].a
components = group(sbb)
nodes_in_cycle = set()
for comp in components:
    for a in comp:
        a = g.vertex(a)
        records[a] = set()
        stack.append([None, a])
        nodes_in_cycle.add(a)
end4 = time.time()
print("Cycle time: "+str(end4-start4))
#         print("")
# print("")
# FINALIZING THE IDENTIFICATION OF RECORDS:
start5 = time.time()
del components
del sbb
del leaf_nodes
maxlen = len(stack)
cnt =0

disc = 0
maxsofar = 0
while len(stack)>0:
    maxlen = max(len(stack), maxlen)
    a, n = stack.pop()
    cnt = cnt + 1
    bbb = False
    if cnt%1000000==0:
        temp = disc
        disc = maxlen - maxsofar
        if disc == temp:
            with open('stack.txt','w',encoding='utf-8') as fb:
                for i in range(1, disc):
                    if a is not None:
                        aa = g.vertex_properties["node_label"][a]
                    nn = g.vertex_properties["node_label"][n]
                    fb.write(aa + nn + "\n")
            fb.close()
            bbb = True
            break
        print("max size so far: " + str(maxlen))
        maxsofar = maxlen
    if bbb:
        break
    if n in collections:
        for m in g.get_out_neighbors(n):
            if g.vertex(m) not in records:
                records[g.vertex(m)] = set()
                stack.append([g.vertex(m), g.vertex(m)])
    elif n in records:
        for p in g.get_out_neighbors(n):
            p = g.vertex(p)
            if p not in collections and p not in nodes_in_cycle and p not in records[n] :
                stack.append([n,p])
    else:
        if a is not None:
            if a in records:
                records[a].add(n)
            for p in g.get_out_neighbors(n):
                p = g.vertex(p)
                if p not in collections and p not in nodes_in_cycle:
                    if p not in records[a]:
                        stack.append([a,p])
end5 = time.time()

print("Identification time: " + str(end5-start5))
del nodes_in_cycle

with open('collections.txt','w',encoding='utf-8') as fc:
    for c in collections:
        labels = [g.edge_properties["edge_label"][x] for x in c.out_edges()]
        label = max(set(labels), key=labels.count)
        fc.write("Collection Node: "+g.vertex_properties["node_label"][c]+", most common label: "+label+"\n")
fc.close()

with open('records.txt','w',encoding='utf-8') as fr:
    for r in records:
        fr.write("Record Node: "+g.vertex_properties["node_label"][r]+", members:"+"\n")
        for e in records[r]:
            fr.write(g.vertex_properties["node_label"][e]+"\n")
        fr.write("\n")
```
```bash
Loading time: 177.26254844665527
Seperation time: 35.15782928466797
Collection time: 69.27971482276917
Cycle time: 2.1327993869781494
max size so far: 15266763
max size so far: 31266763
max size so far: 47266763
max size so far: 63266763
max size so far: 79266763
max size so far: 95266763
max size so far: 111266763
max size so far: 127266763
max size so far: 143266763
max size so far: 159266763
max size so far: 175266763
max size so far: 191266763
```
```xml
<http://yago-knowledge.org/resource/Danica_McKellar>	<http://schema.org/spouse>	<http://yago-knowledge.org/resource/Danica_McKellar>	.
```
x afskgjndf dsgsgs inuhiuhiu
# References
[1] Elbassuoni, Shady, and Roi Blanco. "Keyword search over RDF graphs." Proceedings of the 20th d knowledge management. 2011.

[2] Le, Wangchao, et al. "Scalable keyword search on large RDF data." IEEE Transactions on knowledge and data engineering 26.11 (2014): 2774-2788.

[3]Ciampaglia, Giovanni Luca, et al. "Computational fact checking from knowledge networks." PloS one 10.6 (2015).

[4]Ding, Bolin, et al. "Finding top-k min-cost connected trees in databases." 2007 IEEE 23rd International Conference on Data Engineering. IEEE, 2007.

[5]Kasneci, Gjergji, Shady Elbassuoni, and Gerhard Weikum. "Ming: mining informative entity relationship subgraphs." Proceedings of the 18th ACM conference on Information and knowledge management. 2009.

[6]Seufert, Stephan, et al. "Espresso: explaining relationships between entity sets." Proceedings of the 25th ACM International on Conference on Information and Knowledge Management. 2016.

[7]Tong, Hanghang, and Christos Faloutsos. "Center-piece subgraphs: problem definition and fast solutions." Proceedings of the 12th ACM SIGKDD international conference on Knowledge discovery and data mining. 2006.
