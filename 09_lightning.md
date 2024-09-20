# Part 9: Lightning

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

