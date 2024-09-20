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

