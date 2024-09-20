# Part 6: Robustness of generalized random walk model

- TODO review
  - https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9714372
  - https://arxiv.org/pdf/1910.04316
    - presents an ontology and reviews some of the literature but doesn't really propose anything
    - also seems to ignore the graph aspect, which leaves me very skeptical

- robustness of entropy measure
  - entropy estimates may be brittle
    - the entropy has an upper bound given by the fact that the blockchain itself is finite
    - the entropic approach on its own does not say anything about much stability of the estimate
    - the paper suggests the possibility of using min-entropy or Reyni entropy for more conservative values
    - here we will explore when we might expect values to be robust given changes to a graph (reductions or extensions)
  - simplified scenario
    - alice has an output with k apparent bits of entropy in a coinjoin graph
    - however, as time moves forward, other parties consolidate their outputs to a previously used input address, linking them to their pre-coinjoin txos
  - dynamics of entropy loss
    - in the example, overt clusters are created at the boundaries of the coinjoin graph
    - more generally, deanonymizations can remove edges from the graph, ruling them out from the extended random walk model
    - what is the marginal rate at which entropy decays due to such deanonymizations?
      - the number of bits of entropy associated with an outputs gives a lower bound on the number of edges that must be excluded from a graph to deanonymize that output
        - this bound is very loose
          - deanonymization would require an optimal selection of edges
          - the information content for a random edge deletion will will be larger than 1 bit
            - the information content of a non-random edge deletion strongly depends on the connectivity of the graph
      - more generally, how what is the size of the cut-set of a graph cut that severs a connected component?
        - a random cut would be bisecting the anonymity set
          - the number of bits of entropy associated with an output is a lower bound on the number of cuts
          - this is a tighter bound
        - intuitively if the information content of graph cuts is large, this strategy for bisecting the anonymity set is unproductive for the adverary
  - expander graphs
    - in an expander graph, such cuts do not exist by definition
    - another useful definition: random walks (in undirected graph) rapidly converge on uniform distribution (O(graph diameter) which should be O(log N) for bounded degree random graphs)
      - note that these are typically different than the random walks in Istvan's paper, where the stationary distribution is at best uniform over the absorbers, not the entire vertex set
      - we will modify it again, omitting the direction of edges to obtain an undirected graph where edges represent coins weight by value and vertices are transactions
        - the motivation for this is
          - instead of being concerned with traceability, i.e. given a starting coin output a probability distribution over its likely points of origin
          - rather, we are interested in the connectivity or linkability of coins
          - recall that the intra-transaction structures are symmetric for inputs and outputs
          - consequentially, the stationary distribution has no obviously meaningful interpretation, other than what is transitively linkable to what, and we are more interested in the rate of convergence if it's uniform
            - "uniform" is abuse of terminology
              - the stationary distribution is the weighted degree distribution, which is not uniform
              - but recall that this corresponds to a uniform distribution over individual satoshis, which we have vaguely justified as being information free, so there "uniform" is not used loosely
  - conjecture:
    - if this expanded coinjoin graph has good expander properties...
    - then any coins intersecting with it share the same robust anonymity set
      - this is the set of absorbers related to the strongly connected component
      - the stationary distribution is uniform over the absorbers' sats in the directed graph model
      - the stationary distribution is unfirom over the transactions' sats, or corresponds to the degree distribution over over txs in the undirected graph model
      - the stationary distribution is stable with respect to removal of individual edges
        - removal of edges corresponds to non 0 weights in the subjective probabilities matrix
          - TODO does it make sense to look at the entropy of these matrices? is there a graph spectral perspective on modular decomposition of graphs?
  - xxxx
    - interesting construction in broadcasting on DAG paper https://arxiv.org/abs/1811.03946
    - dense subset sum instances generate high expansion degrees in the bipartite graph internal to a transaction
      - is the algebraic approach to expanders applicable when examining transformations on sub-transactions?
- up until now, we haven't really accounted for fingerprints in detail
  - the sub-transaction model can be extended to take these into account under various heuristical assumptions
    - e.g. that self spend outputs would have the same output type as related inputs
    - however, it's not clear what heuristics are reasonable
    - that said, we can account for any possible heuristic, at least in principle
  - a wallet can observe the distributions of fingerprints/quasi identifiers of graph context (e.g. sibling, uncle, cousin coins of a given utxo)
    - although we are interested in wallets capable of creating multiparty transactions, where these observations can be made actionable more easily, this also applies to unilateral transactions since observations can be made about sent and received payments
  - such a wallet could then only produce transactions that "blend in" to their background, i.e. are sampled from the observed distributions
- in node differential privacy this can be restated as
  - given the transaction graph, and a proposed transaction
  - a randomized analysis algorithm operating on the two graphs, without and with the transaction, must be sufficiently close
    - formally, for all subsets of the possible outputs, the probability of the algorithm outputting a value in the subset should be sufficiently close for both graphs
  - however, it's not clear what such an algorithm would be
    - also note that this kind of reverses the definition of DP, where we are treating an algorithm as given and asking about the regimes for neighbouring graphs where we can bound epsilon & delta
      - what does the "dual" of the privacy budget look like?
  - TODO [George Danezis's note on differential privacy WRT anonymity sets](http://archive.dimacs.rutgers.edu/Workshops/Anonymous/danezis.pdf) inspired this perspective, but i'm still not sure how to interpret the bound he has derived
- graph differential privacy generally involves specific algorithms that have the desirable properties
  - in this setting we are more concerned with whether or not certain graph structures can constrain the existence of such algorithms in a more general sense
  - put more simply, reducing wallet fingerprints is good but it would be nice if we could say how good
  - it is not clear to me if a graph differential privacy approach can be fully applied to the transaction graph model with a node differential privacy perspective
  - however, an edge differential privacy does make sense for several notions of an algorithm:
    - simplest algorithm: instead of deterministic algorithm over probability distributions, consider a probablistic one shot random walk (with undirected edges) of a single satoshi, where the output of the algorithm is the terminal vertex
      - this gives a DP treatment of the probablistic aspects of Istvan's model
      - the start point can be random or with respect to a fixed origin
      - the graph can be an arbitrary subgraph of the actual txn graph
      - the walk is parameterized by a finite number of steps
        - how to choose this parameter? it relates to diameter, degree distribution, and locality properties of the actual txn graph
        - TODO internalize this, by representing the rate of convergence in the output of the algorithm? unclear what's a good way to do that
  - note that the edge differential privacy perspective also corresponds to changes in the subjective matrices of the adversary
- conclusion
  - when estimating entropy, even under stable assumptions, there is a time dependent decay component we have to account for as defenders
    - the rate of this decay is unobservable with regards to priviliged knowledge the adversary may have
    - it depends on the actions of other users, so a given user has no way of mitigating the loss that affects them due to shared history
  - conjecture: if the coinjoin graph has nice expander properties, the rate of privacy decay due to deanonymization can be slowed significantly
    - this is because it takes the removal of many edges to sever a strongly connected component on an expander graph
    - the challenge is, we actually want expander properties on the graph of wallet clusters or entities, which is unobservable...
      - can we translate that to expander properties on the transaction graph, and can we construct them as a distributed, byzantine fault tolerant algorithm?
      - can we formulate this from a graph differential privacy point of view?
        - is that worth doing? Danezis take seems to imply that it could but unclear if its observations even apply in this case
- conclusions
  - there seem to be connections between expander graphs or ideas from graph differential privacy and intuitive notions of robustness of the anonymity metrics discussed so far
