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
