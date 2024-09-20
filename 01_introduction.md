# Anonymity Sets on the Transaction Graph: Part 1

## Introduction

This post is the first in a series about quantifying the privacy of Bitcoin
transactions. When studying the privacy of a system a common approach is to
consider a hypothetical adversary which observes messages (in this case
payments), and which tries to guess among the users of the system who the
receiver and sender of the message are.

The *anonymity set* of a message is the set of users which may have sent (or
received) it. In a perfectly anonymous system the adversary is unable to guess
better than chance among all users of the system, i.e. the anonymity set is as
large is it could be.

Typical Bitcoin payment transactions can often be linked together into so called
*wallet clusters*, which are sets of transactions which are attributable to the
same sender due to the ways in which they are apparently connected. Privacy
enhanced Bitcoin transactions both subvert these clustering heuristics and
introduce ambiguity by involving multiple users, which often means these
transactions don't correspond to payments and therefore thinking of a sender and
receiver is an oversimplification, therefore in general it is better to think of
the anonymity set of a particular output of a transaction.

*CoinJoin* transactions (defined more precisely below) are the most common and
straightforward example of Bitcoin transactions designed to have nontrivial
anonymity sets. The main goal of this series is to develop a more nuanced and
rigorous perspective on how the privacy of such transactions can be studied
quantitatively. We will begin by looking at an intuitive notion of privacy which
is often tacitly assumed and some of the challenges it has in accounting for
more realistic threat models or models of wallet behavior. As the series we will
review some papers on the subject, and use them to progressively builds towards
a more complete account.

This is part of ongoing work on a new CoinJoin protocol which will be introduced
after this series concludes, but apart from a few digressions this series is
mostly concerned with the theory side of things, and is meant to be standalone.

## Preliminaries

The common input ownership heuristic (CIOH) is a fairly standard assumption used
to cluster Bitcoin transactions, which states that a transaction with several
inputs constitutes evidence that the coins spent by those inputs are all owned
by the same entity. It was alluded to in passing already in the original
whitepaper and then elaborated on by several works (TODO citations for satoshi's
paper, ron & shamir, fistful of bitcoins, jonas nick, unreasonable
effectiveness).

This is a heuristic because at the protocol level no such restriction exists:
transactions can spend the coins of multiple users. Greg maxwell's 2013 bitcoin
talk post ["CoinJoin: Bitcoin privacy for the real
world"](https://bitcointalk.org/index.php?topic=279249.0) described(footnote:
not for the first time, goes back at least to 2011) and named named an approach
to building transactions that subvert the CIOH. Central to this approach is the
elimination of the theft risk associated with mixing services that needed to be
entrusted with user funds, which were already available.

In Greg Maxwell's description (some of) the outputs of this transaction have
identical values, similarly to a shuffling stage in a mixnet. The intuition is
that an adversary is given only such a transaction, and is then challenged to
guess which of the equivalent outputs was created by the participant which owns
some particular given input (or more generally, some set of inputs). It directly
follows from the definition of the outputs as equivalent that this output is
underdetermined, so the adversary can do no better than to guess one of the
equivalent outputs at random.

Unfortunately, as we will see the mixnet analogy is only just that, i.e. mix
structured CoinJoin transactions are only able to provide much weaker
guarantees for several reasons.. This is similar to how Tor and onion routing
are often colloquially referred to as mixnets, when in fact the privacy
assurances and threat model are vastly different. Secondly, CoinJoin
transactions don't just exist in a vacuum, and real world adversaries have
access to auxiliary information making the setup in the previous paragraph
rather contrived.

Some simplifying assumptions will still be made throughout this series. To
create a CoinJoin, $k$ users must construct a transaction together, each
contributing one or more coins. Doing this reliably is not trivial in the
presence of byzantine faults. Therefore for the most part we will assume some
kind of perfect black box coordination mechanism. When building the transaction,
neither the participants nor any third party observers will learn anything more
than what the final transaction reveals, i.e. it has perfect data and metadata
anonymity. For example, in the real world the order or timing in which inputs or
outputs were added can help an adversary more accurately guess whether inputs
and outputs are linked, but we will mostly ignore such details. Secondly,
liveness will also be assumed, i.e. the honest participants will always succeed in
outputting a valid transaction. In the real world denial of service must be
mitigated and even if it is perfectly addressed, the partial synchrony or
asynchronous communication model better characterizes network level failure
modes. However, the liveness assumption implies the synchronous communication
model for the same reason that safety is inherent for the most part: CoinJoins
require unanimous agreement because a valid transaction requires signatures by
owners of all of the coins it spends, allowing each participant to ensure that
their funds are not misappropriated.

## Challenges with CoinJoin

Since the term CoinJoin was introduced over a decade ago, multiple
implementations have been developed, each with its own variation on the concept,
making the term somewhat vague. At the same time it often connotes a rather
rigid "mix" structure which presents some practical challenges, and as such is
arguably not just overly general, also overly specific.



- fixed denomination (per tx or per pool)

  - not general enough: people tend to assume a mix transaction structure, but that structure has some inherent limitations
    - if any payment-linked output (newly received and change outputs) is mixed once before being used in another payment output this implies at minimum a 2x overhead in block space
    - if a single party (e.g. taker in JoinMarket) pays for the entirety of the transaction this cost is proportional to the size of those transactions
    - fragmentation, which is inherent in the UTXO model, not only directly contributes to this overhead, but adds an additional exponential overhead if accounting for intersection attacks
    - therefore multiple mixing transactions the amount of blockspace a consumer of privacy needs to purchase to make the same payments, the implied block space overhead for privacy is realistically an order of magnitude higher
    
    
---
    
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

reconstruction attacks Dinur Nissim 03,  https://sci-hub.se/https://dl.acm.org/doi/10.1145/773153.773173

"queries" about hidden state of wallet = cluster information

motivation for exponential decay

e-cash unlinkability vs. batching, hidden state = distribution of denominations

danezis talk mixnet

intersection attacks https://www.freehaven.net/anonbib/cache/disad-free-routes.pdf


concrete attacks:
- intersection
- disclosure
- statistical disclosure
- family https://arxiv.org/pdf/1910.07603

- reconstruction attack = upper bound
