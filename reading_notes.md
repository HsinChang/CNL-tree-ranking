
- [Notes for answer tree ranking](#notes-for-answer-tree-ranking)
  - [Related works for query answering/ranking](#related-works-for-query-answeringranking)
    - [Keyword search over RDF graphs [1] :](#keyword-search-over-rdf-graphs-1)
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
    - [Irène's work](#ir%c3%a8nes-work)
      - [Basic ideas](#basic-ideas)
      - [Tests](#tests)
        - [with Wikidata Gauthier](#with-wikidata-gauthier)
        - [with `cl-decodex.json`](#with-cl-decodexjson)
        - [with `nosdeputes.fr_deputes_en_mandat_2020-03-03.json`](#with-nosdeputesfrdeputesenmandat2020-03-03json)
      - [Graph obtained with Neo4J](#graph-obtained-with-neo4j)
      - [The Java implementation](#the-java-implementation)
    - [RDFQuotient](#rdfquotient)
- [Data Acquisition](#data-acquisition)
- [References](#references)
# Notes for answer tree ranking
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
his walk at v, calculated through a recursive approach.

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
Output:  $k$ connected subgraphs $S_{1}=\left(V_{1}, E_{1}\right), \ldots, S_{k}=\left(V_{k}, E_{k}\right)$ such that
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

# Data Acquisition
Hmm dsahfiuasdhfiuah sdbffdasbhadf
# References
[1] Elbassuoni, Shady, and Roi Blanco. "Keyword search over RDF graphs." Proceedings of the 20th d knowledge management. 2011.

[2] Le, Wangchao, et al. "Scalable keyword search on large RDF data." IEEE Transactions on knowledge and data engineering 26.11 (2014): 2774-2788.

[3]Ciampaglia, Giovanni Luca, et al. "Computational fact checking from knowledge networks." PloS one 10.6 (2015).

[4]Ding, Bolin, et al. "Finding top-k min-cost connected trees in databases." 2007 IEEE 23rd International Conference on Data Engineering. IEEE, 2007.

[5]Kasneci, Gjergji, Shady Elbassuoni, and Gerhard Weikum. "Ming: mining informative entity relationship subgraphs." Proceedings of the 18th ACM conference on Information and knowledge management. 2009.

[6]Seufert, Stephan, et al. "Espresso: explaining relationships between entity sets." Proceedings of the 25th ACM International on Conference on Information and Knowledge Management. 2016.

[7]Tong, Hanghang, and Christos Faloutsos. "Center-piece subgraphs: problem definition and fast solutions." Proceedings of the 12th ACM SIGKDD international conference on Knowledge discovery and data mining. 2006.