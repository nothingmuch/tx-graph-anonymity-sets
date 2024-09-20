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
