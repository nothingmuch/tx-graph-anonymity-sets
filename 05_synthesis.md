# Part 5: Embedding the sub-txn model in the random walk model

- motivate undirected edges
- motivate unfiorm distribution
- model sub-txn as probability matrix available to the adversary
- model sub-txn as additional edges between inputs and outputs
  - motivate by generality, structural questions
- TODO fc24 paper, model coinswaps as additional edges between equivalent coins, or similarly typed coins

  - extended graph, embeding the sub-transaction model
    - instead of modeling transactions as vertices they are substituted with subgraphs which embed of the space of sub-transaction mappings
      - we again modify the graph model, replacing each transaction with a bipartite graph
        - the original edges are linked to new vertices corresponding to inputs and outputs of the transaction, and are still weighted by their value
        - "within" the transaction, edges are introduced between inputs and outputs
          - TODO can the weights on the edges be normalized by multiplying by the sum of the values of the outputs?
            - strictly speaking, no, e.g. wasabi 1 transactions contain a lot of clustering information in the effective values of coins, and these can be attacked with a constraint model to recover the clusters
            - similarly, when generalizing Maurer et al's model to include fees, high variance of the fees paid by each sub-txn (which may be negative) without some theoretical explanation would indicate a less plausible mapping, because it would imply some users are subsidizing others
      - this has the same consequences as using in the subjective matrix as part of the paper's original model
      - ... but allows us to consider a graph differential perspective on both the transaction graph structure and the sub-transaction model using the same abstractions
      - for the sub-txn model, algebraic perspective on expanders might also make some sense, as the symmetric group and potentially other algebraic structures arise in the combinatorial space the non-derived sub-txn mappings
        - this is more of a theoretical interest, for privacy purposes there are special cases that are easily studied and constructed through mechanism design


  - coinswaps are disjoint
    - if widely used, this leaves the graph model under constrained
    - that can be repaired theoretically by introducing edges representing potential swaps
      - see also https://fc24.ifca.ai/preproceedings/146.pdf
