# Atomic Mining Pool Payouts

A mining pool is the pooling of hashrate capabilities by miners, sharing their work shares
over a common network, splitting the reward equally, according to the difficutly weight of
their shares. An atomic payouts scheme is one where the reward value is directly and
trustlessly fan-out to pool participants, without relying on the pool operators.

## Protocol Description

The individual miners submit scriptPubkeys payouts to the mining pool hosts, included in a
transaction spending the coinbase output. The coinbase locking script should include a
signing contribution from each of the miner to avoid the operator altering the payout
transaction. As the output amounts are function of the work shares contributed and the
scoring algorithm relied on, the payout transaction might have to be updated at each
share reception by the pool host, therefore inducing interactivity rounds with all the
miners.

## Why new Contracting Primitives are required ?

A new contracting primitive could enable a compact and trustless payout transaction.
Compact, in the sense that the size of witness should be sublinear with the number of
payouts participants. Trustless, in the sense that once the block is generated, a
participant should be able to withdraw the payout value to a singly-owned utxo without
cooperation with the mining host or other participants.

## Atomic Mining Pool Analysis

### Game-theory Robustness

A lazy or malicious participant shouldn't be able to block or interfere with the
novation of the payout transaction, as it could be a loss of work share compensation
for another honest participant.

The payout transaction should be fee-bumpable by any participant to match their liquidity
preferences, as slow availability of the payout amount is a timevalue loss.

### Attacks & Risks

The fee-bumping of the payout transaction is likely subject to few transaction-relay jamming
vectors like pinning attacks or standardness malleability. A type of time-dilation attack
could be initiated against of the transaction broadcaster full-node, to reject the coinbase
spent as it wouldn't be qualified as final.

Those attacks are likely to be lightweight severity, only inducing timevalue DoS and not
loss of funds.

### Economic Engineering

The coinbase output spending could be minimized by relying on a multi-signature scheme like
MuSig2, for which a compact joint signature can be produced.

Payouts outputs could be 2-of-2 Lightning channels between pair of miners, if they have 
interest to conduct further off-chain operations.

### Privacy Attacks

The payout outputs amounts could be indicative of the work shares contributions. If the blocks
are signed by the pool in the coinbase input, the evolution of the mining pool miner distribution
could be monitored over a sequence of blocks.

### Implementation Complexity

The implemantion complexity of a atomic mining pool payouts should be similar to a multi-party
transaction construction like a coinjoin, where a template transaction is agreed on after submission
of outputs, a multi-party witness is generated and the contributions exchanged among everyone.

The upcoming Lightning interactive construction protocol and its building block code modules
could be reused, where the initiator role would be assumed by the mining pool operator replaying
the `tx_add_output`. The signature exchange might need an update to overlay with a multi-party
signature scheme like MuSig2.

### Protocol Unknowns

One major unknown would be the scalability of the atomic payouts scheme in function of the
number of participants, if the high-frequency of share submissions would enable a lively
reflection of the work contribution in the committed transaction payouts.

### Protocol Evolvability

It is left as subject of research if an atomic mining pool payment scheme could be built over
Lightning channels, where the signature contributions in the coinbase could be "buy-out" by
the mining pool operator in an exchange of a per-miner PTLC, therefore avoiding an on-chain
transaction footprint scaling with the number of pool participants.

### Ecosystem Impacts

Enabling trust-minized payouts scheme reduces the leverage of mining pools over miners and
compacting the payouts make the mining operations more profitable for miners, therefore
making the mining ecosystem more profitable and sustainable.

### Sources

- Laurential pool paper by Con Kolivas / Ryan Ellis: https://laurentiapool.org/wp-content/uploads/2020/05/laurentiapool_whitepaper.pdf
