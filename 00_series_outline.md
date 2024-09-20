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

---

# Part 1: What do we mean by "CoinJoin" "anonymity set"

## Introduction

- "CoinJoin" has been used to refer to a number of related things
  - overly general: several different implementations with significant differences between them
  - not general enough: people tend to assume a mix transaction structure, but that structure has some inherent limitations
    - if any payment-linked output (newly received and change outputs) is mixed once before being used in another payment output this implies at minimum a 2x overhead in block space
    - if a single party (e.g. taker in JoinMarket) pays for the entirety of the transaction this cost is proportional to the size of those transactions
    - fragmentation, which is inherent in the UTXO model, not only directly contributes to this overhead, but adds an additional exponential overhead if accounting for intersection attacks
    - therefore multiple mixing transactions the amount of blockspace a consumer of privacy needs to purchase to make the same payments, the implied block space overhead for privacy is realistically an order of magnitude higher
- The intuitive definition of "anonymity set" people often invoke for CoinJoins is problematic
  - it considers a transaction in isolation
  - however, a real world deanonymization adversary isn't restricted to only looking at a single transaction
    - they have the entire transaction graph to work with
    - as well as, potentially, information external to the transaction graph
  - in order to make guarantees about privacy we must consider these losses
    - often it's taken for granted that they are negligible. THESE ARE THOUGHT CRIMES!!!
    - academic research on privacy is a rich field with many ideas we can borrow, so we can and should do better
- This is a warm up post, the first in a series
  - in this one we will look at some practical challenges of CoinJoin transactions
  - In subsequent posts, we'll...
    - borrow some ideas from academic research on privacy in general and on Bitcoin in particular
    - build up to more robust and general definitions of anonymity sets on the transaction graph
      - emphasis on graph
    - end by looking at how Sybil resistance can be quantified and strengthened
  - this is the (hopefully self contained) theoretical foundation of my ongoing protocol work
    - to be detailed in subsequent posts after this series concludes
      - useful special cases
      - practical/implementation considerations
      - mechanism design

## Historical Context

- gmaxwell's ["Coinjoin: Bitcoin privacy for the real world"](https://bitcointalk.org/index.php?topic=279249.0)
  - $k$ users construct a transaction together
    - for simplicity, we assume they send the money to themselves
    - we also assume some coordination mechanism ensures transactions can be constructed with:
      - privacy (users and observers learn nothing other than what the final transaction reveals)
      - liveness (honest subset of the parties in the same network partition will succeed in outputting a valid txn)
      - safety (inherent in requiring txns to be unanimously signed)
    - the transaction obeys a certain symmetry
      - there is at least one subsets of the outputs of size $\geq 2$, all members of which have identical values and equivalent scripts
        - the scripts themselves differ but the script types must be identical
        - public keys are random and not reused
        - the order of outputs is random
      - similar intuition to mixnets, coins are (kinda sorta) messages, the transaction shuffles them like a relay
  - framed adversarially:
    - To quantify privacy, ask whether or not an adversary can identify the owner of an output from an anonymity set
    - being precise, the anonymity set of an output is the set of possible users who own it
      - however, this set is not directly observable on the transaction graph, we only see coins
      - if each output uniquely identifies a distinct user, the equivalence class of a given output is an anonymity set
    - given only this transaction as data
    - and a specific input within this transaction as a target
    - the adversary can guess correctly which output was created by the same user as the given input no better than with probability $1/k$
      - i.e. uniform probability distribution

## Model boundaries

This model for an anonymity set builds on strong assumptions, which limits its applicability. We will try and make explicit some of these assumptions. As the series unfolds these assumptions will be progressively weakened, broadening the domain in which we can claim to quantiatively understand CoinJoin privacy.

### Data, metadata, ...

- tacit assumption: the transaction graph is all that matters for privacy
- there are a lot of things we will consider out of scope
  - blockchain external information
    - e.g. KYC information
  - p2p protocol and other network protocol leaks
  - temporal patterns
  - light client limitations
- we will briefly revisit some of these in the last post in this series

### Sybil deanonymization attacks

- tacit assumption: no sybil deanonymization attack aka n-1 attack takes place during transaction construction
- this is the simplest way to undermine the model: give adversary information about who the user is *not*
  - If $k-1$ of the (apparent) users are under adversarial control, there is no anonymity
- In a permissionless system, such "Sybil" users are indistinguishable from honest ones and prevention mechanisms are unavailable
- Therefore Sybil resistance can only come from imposing some cost on participation
  - some costs are inherent in bitcoin: fees, time value of money
  - additional costs can be imposed
- This is really more of a meta topic since it's not about evaluating metrics for privacy, and more about how to interpret these metrics
- For simplicity, assume Sybil resistance can be taken for granted for now, and all users are honest
- the 2nd to last post in the series will try to quantify guarantees aspects of Sybil resistance, namely a cost for such an attack
- TODO diagrams for sybil coinjoin

### Other users must also protect their privacy

- tacit assumption: users' privacy does not degrade after the coinjoin
- Alice has an output of a CoinJoin transaction with $k$ indistinguishable outputs.
- Alice makes an anonymous payment
- Bob, Carol, Dave, ... all dox themselves
  - Address reuse, e.g. deposit to exchange with per user address
  - Link with pre-mix coin
  - Publicly unobservable KYC disclosures...
    - ... until e.g. Celsius leak
- TODO diagrams
  - address reuse
  - link with premix
  - out of band wallet cluster
  - all different counterparties of Alice's
- looking at how the transaction graph affects local notions of anonymity sets will be the topic of a later post in the series
- For simplicity, assume other users are both competent and vigilant.
  - In part 6 we will consider how privacy might be more robust to weakening this assumption

### Difficulties with change outputs

In general, we can't predict payment amounts or fees

- tacit assumption: coinjoin outputs can be used to make outputs without worrying about how to handle change
- Alice has two coins of (a standardized) value $v$, from two independent CoinJoin transactions both with $k$ equivalent outputs
 - for simplicity assume $v$ is the smallest such value for which $k$ parties are regularly available
 - Alice makes two independent payment transactions $A$ and $B$ with and creates two change outputs from these transactions, with values $v_A \le v$ and $v_B \le v$.
 - if Alice spends either change output, the subsequent transaction will be linked to the preceding one ($A$ or $B$), degrading the anonymity set of the payment after the fact
 - suppose $v_A + v_B = v + \epsilon$, if Alice would like to recover this value, she could attempt to CoinJoin again, producing another coin with an anonymity set of size $k$
   - ![toxic_change_before](https://hackmd.io/_uploads/HkXWy0Le0.svg)
   - If she consolidates these change outputs in a unilateral transaction that consolidates them in preparation for a coinjoin, she has overtly linked txns $A$ and $B$ retroactively TODO diagram
   - Even if she does this directly in a CoinJoin transaction that allows multiple inputs per output, but there aren't $k$ equivalent inputs with value $v_A$ and $k$ inputs with value $v_B$, then her inputs are likely still linkable equivalently to if she had made a unilateral txn as in the previous case TODO fix amounts in diagrams
   - ![toxic_change_after](https://hackmd.io/_uploads/SkYZyRLg0.svg)
- these challenges are addressed in parts 4 and 5

note: diagrams contain errors

### Multiple coins

- tacit assumption: every payment can be completed by selecting a single coin
- reality: multiple coins are a necessity arising from UTXO model
- inevitable source of history intersection attacks
  - these kinds of attacks are unintuitively powerful
  - next time we'll explore in depth
- Alice is paid monthly from her employer Erica's cold storage
  - One large batch transaction
    - Assume all employees are paid the same rate and do the same amount of work
    - The outputs are similar to a CoinJoin, but Alice's employer Erica knows which output belongs to which employee
  - The Erica's change is typically used for the next transaction, linking Alice's coins, along with those of her coworkers, in an easily clusterable chain of multisig transactions
  - For privacy, Alice CoinJoins each of these outputs
    - what about values that don't "fit" in a denomination?
      - creates change values, c.f. toxic change problem
      - assume Alice's salary is denominated in bitcoin, and happens to have value $v + \epsilon$
    - what about fees?
      - assume $\epsilon$ being enough to cover a single CoinJoin with a standard denomination
    - what about "remixing"? (and fees again?)
    - TODO diagram with multiple batch txns leading to multiple coinjoins
  - Next, Alice wants to buy a boat privately
    - Note that we're assuming the *best* case scenario for all of the sources of friction
      - in reality, those sources of friction can dramatically amplify this kind of attack
    - Bob the boat salesman can learn about Alice's source of income
      - all of the coins used for the payment originate from a chain of (likely linkable) multisig outputs belonging to her employer
      - TODO history intersection in simple terms
        - the attacker's progress *compounds*
        - SIMPLIFY the adversary's power to deanonymize grows multiplicatively with every marginal coin used by the target
        - SIMPLIFY every marginal user deanonymized successfully by the adversary is also a multiplicative gain
        - maybe: each additional UTXO and user increases the surface area of attack in a multiplicative way
      - this is really unintuitive. defies expectations. even experts often overlook this.
    - Alice's employer Erica can reliably guess she bought something expensive
      - all of the coins used for the payment come from a transaction that Alice's employer can see is Alice's.
        - Even if some of her coworkers also participated in some of the CoinJoins, the likelyhood that one or more participated in *all* of them is low, but that is what is required for Alice to maintain privacy from her employer
    - Alice's coworker Carol can see that someone at work bought something expensive, but doesn't know it was Alice
      - same analysis as Bob, but with more certainty regarding the points of origin, history intersection is deterministic instead of probablistic
- these challenges are addressed in parts 4 and 5

## Can we do better?

- the last examples create difficulties for the model
- take make stronger guarantees, we need a more conservative and comprehensive approach
- in the literature: burden of proof is generally placed on the defender
  - in other words, just because it seems private, doesn't mean it is
    - the difficulty of deanonymization needs to be considered from the point of view of the most capable adversary the assumptions allow
    - not the most capable deanonymization attack that can be constructed explicitly
      - according to most vendors: burden of proof is on the attacker... =(
      - strawman privacy assurance: "since you can't deanonymize this specific coin, we can conclude the system is private"
  - rate of gain vs. rate of decay needs a quantitative treatment
    - unfortunately, often only privacy gain is analyzed while leaving various forms of decay as externalities
      - this is often because decay is harder to analyze
      - that said, when simplifying, approximation should be conservative erring on the side of underestimating privacy
    - gain is usually roughly linear
      - e.g. looking at a sequence two coinjoin transactions in isolation, the anonymity set of the input to the second transaction is "inheritied" by the output
        - this is sub-linear because $|A \cup B| \leq |A| + |B|$
      - looking deeper into a coin's history or possible spending paths, growth may compound
        - with a large set of active users this may proceed exponentially for a while, but this set is finite so this growth follows a logistic progression
    - decay is typically exponential
      - intersection attacks (think "20 questions")
      - quantifying bits of entropy captures this intuition
        - the entropy of an anonymity set measures how much information the anonymity set hides with regards to the attribution of a spend of a specific coin
        - the adversary must learn at least this many bits in order to fully constrain attribution so as to be deterministic
        - if the adversary learns (maximally useful) bits of information at a linear rate, then the effective anonymity set size halves with each bit obtained
        - oversimplified example:
          - start with a single origin coin
          - perform a series of coinjoins, each time splitting a random coin in two, until some limit
          - keep doing this until a perfect binary tree of transactions is formed in the wallet
          - the adversary is given some subset $S$ of these terminal coins (the leaves of the tree), and must guess the origin coin
            - as $|S|$ approaches $n$, the intersection of all the potential origins of each coin in this subset shrinks rapidly as ancestors not shared by all coins can be ruled out
    - targetted vs. dragnet, marginal cost of deanonymization... ?
      - this is mostly a qualitative distinction, not clear to me how to treat it quantitatively yet
  - unaccounted exponential decay is the typical failure mode
    - https://sci-hub.se/https://dl.acm.org/doi/10.1145/773153.773173
    - 
    - only modeling privacy gain without taking into account the privacy decay is a common failure mode of privacy enhancing systems
    - examples:
      - [US census](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=2ab47454f59d9d8e55d4d8a69530562a3690794a)
      - [netflix graph](https://www.cs.utexas.edu/~shmat/shmat_oak08netflix.pdf)
      - Diaz et al
        - [crowds anonymity system](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d24f67f78d07b33cd86bc0526f468d78c0892016)
        - [lightning, dandelion](https://pure.mpg.de/rest/items/item_3500837/component/file_3500838/content)
      - [sharedcoin](https://www.coinjoinsudoku.com/advisory/)
      - [Jonas Nick's BIP 37 based wallet clustering](https://jonasnick.github.io/papers/thesis.pdf)
        - linear number of block filters ANDed together yields exponential decay of noise
      - [clustering using intersection attacks](https://arxiv.org/pdf/1708.04748)

## Conclusion

For real world transaction graphs, the implied anonymity set model can only be safely applied in a narrow range of settings.

- uniform anonymity sets with regards to the origins of real world payments transactions are a mythical creature / rare special case
  - TODO what does "uniform" mean? we implied it in the intro but we didn't introduce the generalization to probability mass functions
- anonymity sets for individual outputs are brittle in the context of the wider graph
  - this is made harder by sensitivity to variable and unpredictable payment amounts and fees, which is a market prediction problem
- a robust anonymity set definition needs to take more of the transaction graph into account
- or to put it more bluntly, we can't afford to only look at individual transactions in isolation when we making defensible claims about privacy

## Coming up

In the next post in this series we'll try to define this notion of anonymity set a more precisely. As the series progresses it will be progressively generalized to account for types of decay.

---

# Part 2: Equivalence Relations Within Transactions

The last post looked at the usual intuitions regarding how coinjoin privacy works, and examples of how it can go wrong.

This post attempts to make this model more precise, introducing building blocks so that we can look at the examples more quantitatively in subsequent posts.

## A more precise definition of the intuitive model

- wallet clustering/attribution deanonymization attack
  - adversary is given:
    - *only* a single bitcoin transaction, along with the txos it spends
      - txins that spend its outputs (at a point in time) can also be considered but the following definitions of equivalence only make sense when all coins have been spent
    - a specified target input
      - without loss of generality, target output formulation is symmetric
  - adversary assigns probability over the outputs' relatedness to the target input
    - different intuitions have similar structure:
      - ownership: probability that an output is owned by the owner of the target input
        - change identification heuristic, post-mix linking, ...
      - funding: probability that an output was funded by the value of the target input
         - payjoin, knapsack mixing, ...
    - note: this also tacitly assumes adversary is rationally and consistently truthseeking, in reality the incentives for (for profit) transaction surveillance are different.
  - can the adversary do better than chance? what does "better than chance?" mean?
- equivalence classes of utxos
  - two unspent txos are equivalent if they have
    - equal values
    - equal script types
      - or equal scripts, up to bitvomit (public keys or hashes)
- equivalence classes of inputs
  - unspent prevouts must be equivalent
  - nSequence must be equal
  - scriptSig, witness stack, redeem scripts, tapleaves must be equivalent up to bitvomit
    - tr keyspends are a distinct equivalence class, and each tap leaf is, but unclear on whether spend path should be considered
      - the spend path length leaks information about the size of the tree
- equivalence classes of coins
  - txos with 0 or more spending inputs
  - two coins, each double spent with non-equivalent inputs can still be considered equivalent if the sets of inputs have elements from the same equivalence classes
- equivalence classes as anonymity sets
  - bitvomit is distinct but indistinguishable from random in most cases
  - exceptions: nonce leaks, bad rng etc, ...
  - within an equivalent class this apparently random variation is the only variation
  - ... apart from order, assume everything is shuffled
  - therefore, without additional information the adversary can only assign a uniform probability distribution over the equivalence class of the candidate outputs
  - if there's a 1:1 correspondence between elements of equivalence class and owners, then can and has been interpreted as an anonymity set size metric

## Limitations

- adversarial model is
  - too optimistic
    - only access to a single transaction at a time
    - is passive
    - has no memory, and can't adapt with new information
    - cannot see the transaction graph
    - can't see network level metadata
    - 1:1 correspondence between objects and users maximizes size
  - too pessimistic
    - with multiple inputs and outputs per user, equivalence ignores weaker notions of relatedness that may still contribute non-zero probabilities

## Consequences

- is this anonymity set definition actually applicable in the real world?
  - tacit assumptions in the intuitive anonymity set definition
    - spending transaction has 1 input
    - spending transaction creates no change
    - ... these assumptions are not realistic, but removing them undermines the applicability of the model
  - strictly speaking, this anonymity set definition is incompatible with lump sum spending of coins of arbitrary values unless change outputs are abandoned
    - in spite of this, it is the definition that is often (implicitly) invoked when discussing on-chain privacy
    - coinswap comes closest to realizing this due to the txn-graph-disjoint footprint
      - however, the problem of recovering change outputs still presents challenges for consolidation

---

# Part 3: Generalizing Anonymity Set Sizes

- when we just count elements, we're tacitly assuming uniform probability distribution
  - this is equivalent to assuming the adversary is ignoring a lot of information that is clearly available to it
- what kind of information is available?
  - attributes and quasi-identifier
    - Sweeny, deanonymizing the US census etc
    - wallet fingerprinting
      - https://ishaana.com/blog/wallet_fingerprinting/
      - script types
      - nLocktime values
      - RBF signalling
      - non-random input/output ordering
      - fee policies
      - ...
  - the values of coins
  - txn graph
    - partial order over coins, generates quadratic number of "attirbutes" and exponential number of quasi-identifier
      - for every pair of coins, one might be an ancestor of the other, or they maybe unrelated
      - this in turn generates an exponential space of quasi-identifiers, in which every coin is sparsely represented
      - path length and plausibility of relatedness based on the coins' values can also be taken into account
    - this is arguably the most important consideration, but we will address it in the next post
- [Diaz et al](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d24f67f78d07b33cd86bc0526f468d78c0892016)
  - system model (communication)
    - adversary capabilities
    - senders and receivers in a network
    - definition of anonymity set
      - entropic approach for non-uniform distributions
- applications of entropic anonymity to bitcoin
  - values of coin within a single transaction
    - [Maurer et al](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d24f67f78d07b33cd86bc0526f468d78c0892016), sub-transaction model
      - concrete application of Diaz like model to single bitcoin transaction
      - generalizes from the notion of equivalence classes
    - Boltzmann link probability matrix
      - more or less equivalent, defined in terms of an entropy measure
        - however, the commonly transaction level entropy is harder to interpret
        - Maurer et al has a probablistic treatment of the probability of a specific input or output, taking the entropy of these distributions better suits our needs
          - 0 fees assumption needs to be generalized
    - both models computationally intractable for larger transactions
  - in other words, for a given output, we can determine how much information an adversary would need to deterministically link it to a cluster of inputs
    - this is under the assumption that no net transfers happen within the transaction
    - this assumption is quite strong but seems to be justified in practice
- conclusion
  - by estimating the entropy of a distribution we can evaluate and compare apparent privacy more generally
  - this opens the door to overcoming the practical challenges implied by only looking at equivalent coins

---

# Part 4: Accounting for transaction graph with respect to third parties

- 4 party examples
  - ![4_party_tx](https://hackmd.io/_uploads/rkp6AA3LC.svg)
  - ![4_party_tx_network](https://hackmd.io/_uploads/rymCAAh8C.svg)
  - note symmetry of these examples (4x 2 party coinjoins in Clos network vs. 1x 4 party coinjoin)
  - gmaxwell mentioned Clos networks, more generally interconnection networks
- In recent work (Istvan paper https://arxiv.org/abs/2211.04259) Diaz et al's notion of an entropic anonymity set was defined for a given coin based on the transaction graph structure
  - over simplified explanation of their model
    - assume every satoshi in an output has to have come from somewhere (originally a coinbase transaction)
    - assume each input is equally satoshi is equally likely to have ended up as each output satoshi
      - this is similar to how ordinal theory assumes by fiat a 1:1 correspondence based on order
      - however, assuming the transaction was shuffled for privacy and funds may have been redirected arbitrarily within a transaction, any permutation of sats reasonable
        - recall the "gold" metaphor to Bitcoin transaction, every transaction melts down lumps of bitcoin (the transaction inputs) and casts new ones (the outputs)
      - one can also imagine Bitcoin as a (integer) Petri net where transactions are transitions, outputs are places, and tokens are sats
  - conceptually, the probability of a specific satoshi originating from a particular coin can then be estimated by sampling random walks from unspent output
    - note that this is symmetric, i.e. equally for every point of origin we can ask about the probability distribution of where it may have ended up
  - note that this "per satoshi" perspective can justify the modelling assumption made in the paper, which assumes the transition probabilities of the random walks correspond to the values of the coins
  - instead of a Monte Carlo approach, these probability distributions are estimated more efficiently using linear algebra
  - the model also allows for subjective probabilities to be included in a straightforward way
    - any private knowledge or additional analysis the adversary might have is represented as a matrix that combines with the ones derived from the graph
    - this can account for the combinatorial explosion of potential quasi-identifiers alluded to in the previous post, as well as blockchain external information
    - in particular, a refinement of this model would be to instead plug in the Boltzmann link probability matrix or its (more or less) equivalent link probabilities as per Maurer et al
    - although this doing this explicitly computationally intractible, as defenders we are more interested in information theoretical lower bounds on how under-constrained a problem is, less so with the computational difficulty of solving an instance
      - especially since the related approximation techniques are often very powerful, and many actual instances of the related hard problems (e.g. subset sum, which in general is NP complete) are often easy in practice (many special cases solvable in roughly linear time)
- conclusion
  - with entropic approach we need to measure the right thing
    - e.g. the approach in the paper is potentially too optimistic for defender's point of view
    - we also want to know how robust this measurement is, a transaction graph with good expansion properties will not result in higher entropy according to the model in the paper
- conclusion
  - the entropic model is sufficiently general to account for the graph, not just individual transactions
  - using more conservative estimates, or different assumptions yields very different values
    - as defenders, we are mostly interested in lower bounds, so e.g. Shannon entropy of conservative distribution, or perhaps more conservative entropy metrics

---
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
---

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

---

# Part 7: Accounting for privacy from counterparties in the random walk model

- we now to turn the problem of priviliged viewpoints
  - adversaries are not only constrained to public observers or dragnet surveillance
  - we need privacy from our counterparties
- motivating example: privacy from a local bitcoin ATM
  - David Vorrick's Streaming Wallet model, privacy doom principle talk from btc++ cdmx
  - Alice lives in a small community
    - Bitcoin is not generally accepted as payment
    - some of the local population uses for savings
    - a KYC free, cash only ATM is available
  - Alice would like to regularly buys small amounts bitcoin from the ATM for savings, and occasionally sell some of it back in order to make purchases
    - especially when larger purchases are made, multiple coins will be linked together, to a specific subset of the ATM's old origin coins, thereby allowing the ATM to probabilistically cluster these together
  - However, the ATM operator can surveil Alice if the ATM is her only counterparty
    - Naive solution: just coinjoin after buying and before selling
      - This lets Alice interact with random peoples' coins in such a way that implies that it could be their coins that end up being deposited back to the ATM
    - Unfortunately this is Eve-Alice-Eve or Streaming Wallet model:
      - every time she sells back a CoinJoin output to the ATM, the ATM can observe its own spending transactions (Alice's purchases) in the (recent) history of the new coins it receives
      - this is true of *every* coin Alice controls, but normally would be a low probability event
      - the ATM operator's power to surveil Alice in general increases exponentially over time
        - furthermore, this holds for all other members of her community
        - additional data is also available to heuristically cluster
          - e.g. amount distributions, temporal patterns, especially wrt purchases
- although this signal can't be removed, noise can be introduced
  - suppose Alice purchases a coin $A_1$, and coinjoins with Bob, a random Bitcoin user from the internet, to produce $A_2$.
  - Bob's output from the same coinjoin, $B_2$ would appear to be similar to Alice's
    - if Bob were to sell it to the ATM, it would confound the ATM
    - however that is very unlikely
  - suppose Alice then goes on to spend $A_2$ in a coinjoin, creating $A_3, ... A_{i-1}$
  - suppose Alice then spends $A_{i-1}$ into a coinjoin with a coin $C_{i-1}$, which can be traced back to $B_2$
    - it doesn't matter if this is Bob (low likelyhood) or just a counterfactual path that could be $B_2$ but isn't
  - there now exists two apparent paths path from $A_1$ to $A_i$,
    - one is Alice's real wallet cluster pertaining to those funds
    - the other is the counterfactual path through Bob's coin, that eventually rejoins Alice at step $i$.
    - since we're tacitly assuming $i$ steps for both, these make the same contribution to the ATM's observable signal, but one of them is noise
      - more generally, the path lengths may differ, and we'd like to have more than just plausible deniability but a sufficiently robust anonymity set whose constituent objects are these paths
- conclusion
  - in the this adverse settings we can quantify the entropy of special cases of the random walk idea
  - TODO inuition: the intersections of the walks create symmetries which imply good expansion properties even after the strong cuts introduced by the private knowledge of the adversary

---

# Part 8: Sybil Resistance

- why i am not an entropist paper
- n-1 deanonymization attack has a measurable cost
  - coindays destroyed (time value of money)
    - belcher's economic analysis of fidelity bonds
  - fees
    - if we assume the Sybil adversary has access to significant hash rate, apparent fees are an unreliable metric
    - full nodes which relay transactions in princple can attempt to discount potentially costless high feerate transactions (where a miner includes high feerate transactions that were not broadcast to other miners) as well as estimate an opportunity cost of foregoing publicly known transactions that were not included
- TODO n-k analysis?
  - what do marginal costs of such attacks look like?
- economic analysis from defender point of view
  - risk vs. consequences
  - implied lower bound on expected loss by willingness to spend on privacy
- an apparently ambiguous transaction graph is an unforgeably costly object due to the fact that bitcoin block space and UTXOs are scarce resources and due to the time value of money
  - if the apparent cost is sufficient, randomly selecting unspent outputs on such a graph as counterparties for CoinJoins can be said to be Sybil resistant
  - furthermore, the random selection is verifiable, a cost of a larger graph can be estimated from only a few local observations. we will look into this in more detail in the next post
- conclusion
  - robust entropy meaures still might be possible to game
  - measuring the costliness of gaming such a metric can aid in interpreting it

- TODO the mechanism design aspect of sybil resistance leads to cardinality estimation approach to getting an upper bound on the entropy of just the sybil resistant coinjoin graph
  - this is an important idea to convey, but mechanism design is a radical perspective shift here, and i'm not sure it belongs in this series at all

---

# Part 10: Lightning

- random walk model applies to offchain data, since it's just tx graphs
  - replaced txns known to the adversary may leak additional information
  - https://arxiv.org/pdf/2006.12143
  - https://pure.mpg.de/rest/items/item_3500837/component/file_3500838/content
  - we can try to model what routing parties can observe within this model
  - not clear if this is a fruitful direction for more privacy aware lightning routing
    - perhaps potential leaks can be accounted for and different routing decisions made to limit such leaks but no obvious direction
- on chain
  - funding txs & cooperative close
  - public channels' node id ~~ address reuse
  - private channels still fingerprintable as p2wsh 2-of-2
    - any studies on clustering?
  - interactive-tx & dual funding
    - interactive-tx might or might not be considered a coinjoin protocol depending on whether the definition involves symmetry in the output amounts
    - dual funding has similar properties to payjoin with regards to confounding clustering heuristics
- related work
  - cln path randomization
  - LN differential privacy paper
- leakage aware routing?

---

# Part 9: some remarks on the "out of scope" things

- temporal patterns, availability
- bitcoin p2p protocol, auxilliary protocols (electrum etc)
  - tor exists, but it needs to be used correctly
- BIP 37, Jonas Nick's thesis, unreasonable effectiveness of address clustering
- dust attacks
- KYC snafus (Celsius, etc)
- MtGoxAndOthers
- payjoin, coinswap, lightning nodes
  - payjoins are covert, break sub-transaction assumption
    - if widely used, this leaves the sub-transaction model under constrained as the "fees" are allowed to range to arbitrary values
- ordinals
  - in principle, nothing prevents privacy preserving exchange of ordinals through coinjoins
  - but also ordinal theory is kind of an attack on fungibility by definition boo hiss
  
- ... basically recycle btc femmes / bitcoin privacy curriculum materials, and point to belcher's Privacy wiki page for a broader overview?

---

# Part 10: Practical considerations

- sub-txn model is intractable to compute for large transaction
  - equal outputs and radix coinjoin special cases
  - subset sum estimation
    - sparse convolution
    - generating functions
    - linear sketches
- path based model
  - data availability
- expander graphs
  - random graphs are typically expanders
  - are partly randomized DAGs that? is the degree of randomization within classes of similar coins enough to give rise to expander properties?

---

# Spin off: Fungi Mechanism Design

Applying the ideas of the previous posts in a more constructive manner, we can ask how we might produce a transaction graph that has good privacy properties?

This breaks down into properties of the resulting transaction graph and properties of protocols that might be used to create it.

- desirable transaction graph properties
  - randomized
  - expander properties
  - transactions with sufficient ambiguity
    - Low Hamming weight values in useful bases
  - convexity as the number of honest users increases with regards to

- desirable protocol properties
  - safety & liveness
    - DoS resistance
  - low communication complexity
  - incentive compatibility

problem space:
- coalition formation
- expander constructions

implementation cibsuderations
- computable metrics
  - ambiguity
  - sybil resistance lottery mechanism
- hiding cost function
- denial of service protection

## Leftovers

### Use cases

TODO mention these and how they relate to this model

- Payjoin
  - technically off chain is still a txn graph
- fedi/cashu withdrawals/deposits