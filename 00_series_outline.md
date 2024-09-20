# Anonymity Sets on the Transaction Graph
## Post Series overview

1. reviews the folk/intuitive definition of CoinJoin anonymity sets
2. more formal definition
3. entropic anonymity models in general, and sub-txn model in particular
4. entropic anonymity sets on the transaction graph, the absorber model
5. combining the sub-txn model and absorber model
6. generalizing the random walk model, robustness, expanders & DP
7. eve-alice-eve model, path based anonymity set
8. measuring sybil resistance
9. "out of scope" appendix
10. practical considerations
11. mechanism design


TODO split part 9 & bits of 4, 5 and 7 into an algorithms & approximations part separate from mechanism design?

## summary of each part (re Dan's CONCLUSIONS comment)

1. this post is about difficulties with taking Greg Maxwell's CoinJoin post at face value
    - edge/corner case examples
    - condensed literature review, indicating more theoretical reasons for concern
    - conclusion: equal valued output notion of anonymity set is incomplete
2. this post more precisely defines the equval values anonymity set model
    - by having a more precise model, we can more clearly see when it's valid to apply it
    - this makes clearer some inherent difficulties with the coinjoin-mix-of-equivalent-values approach
    - conclusion: better understanding how it falls short
3. this one introduces Maurer et al's sub-transaction model, which generalizes the fixed value model to handle arbitrary amounts
    - the point is mainly to introduce the concept of entropic anonymity sets and see how it can be applied to an individual transactions
    - there are additional refinements to the sub-txn model and technical discussions about approximations
    - conclusion: sub-transaction model / boltzmann entropy are a useful building block, but not the full picture since transaction graph is still neglected
4. introduces Istvan's paper, which generically treats the transaction graph parts
    - the paper makes a rather strong assumption about what happens "inside" transactions
    - however the model can readily account for this, i.e. it is composable with the sub-transaction model
    - conclusion: this model under-constrained but highly adaptable.
5. addressing robustness and remaining wallet fingerprints more generally
    - tries to make explicit some nuance of how to apply the sub-transaction model as well as other quasi-identifiers
    - discusses expander graphs in the context of the graph as a whole to make the entropy metrics robust
    - graph differential privacy is invoked as a kind of platonic ideal
      - unclear if this approach is fruitful theoretically or practically
6. this part explores the implications of repeated transactions where the counterparty is the adversary

    - since the adversary has a privliged viewpoint, the entropic anonymity set approach based on the sub-txn model and the transaction graph is insufficient as it does not a
    - however, an edge case of the previous model which is more robust to this threat model is made explicit, allowing us to more conservatively estimate the entropy in this setting as well
    - conclusion: estimating entropy with respect to a specific boundary condition on the transaction graph is also possible. in expander graphs we can expect this metric to improve with repeated transactions.
7. generalizes belcher's fidelity bond framework for sybil resistance
    - attempt to address "why i am not an entropist" perspective
      - qualifies entropy metrics with a cost (to the adversary) to produce the underlying observables
    - conclusion: looking at a transaction graph and estimating its cost, users can make a more informed decision whether extending that graph with a coinjoin is sufficiently private and sybil resistant given their threat model and risk apetite
8. appendix: some remarks on the out of scope things
    - cautionary tales about stuff the model doesn't cover
    - conclusion: details matter, reiterate the model boundaries
9. appendix: practical considerations
    - how can we apply these models
      - given limited computational power
      - only a partial view of transaction data
    - conclusion: mostly workable

the last part should probably be split into its own thing

9. defines design space and proposes mechanisms applying the previous posts
    - optimization problem framing
    - bounding satoshi loss term representing lack of privacy
    - radix coinjoin, subset sum density
    - costlessly amplifying sybil resistance of subgraphs of apparent coinjoin graph
      - implication is that adversary costs can be supra-linear in number of honest users
    - estimating the subjective value for a coin
    - cost function randomness & privacy leaks
    - coalition formation games, transferrable utility
    - fungi co-spend proposal structure

## Target Audience & Goals

- background assumptions
  - consensus data structures
  - p2p network model
  - undergrad level math concepts
    - discrete math
      - set theory, functions, relations
      - graph theory
      - combinatorics
    - game theory
    - mathematical optimization
    - probabilities
- agenda
  - develop theories of bitcoin privacy
  - improve developers' literacy of privacy literature, and raise their awareness of the brittleness of privacy
  - empower people to critique privacy claims and better evaluate bitcoin privacy software
  - empower people to critique my synthesis of the various theories
  - empower people to critique my protocol design for fungi, which applies these theories

