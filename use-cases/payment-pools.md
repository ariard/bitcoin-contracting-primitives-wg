# Payment Pools

A "Payment Pool" designates a shared-owned UTXO among a set of N participants, where the
withdrawal of any participant's "UTXO shard" is non-interactive and trustless. The participants
can update the "UTXO shard" or balances distribution with either an on-chain write or an
off-chain state in function of the pool construction considered.

## Protocol Description

The description is a digest of the CoinPool construction, combining the compact and fault-tolerant
withdrawal techniques of payment pools with the Eltoo off-chain state update mechanism.

The operations are the following:
- Setup phase: the participants gather the collaterals inputs and exchange withdrawal witnesses/transactions.
- Update phase: the participants update the pool state by re-generating and exchanging withdrawal witnesses/transactions.
- Withdrawal phase: one or more participants unilaterally exists the pool by converting their balance to an on-chain UTXO.
- Revival phase: the remaining set of participants collaboratively relift the on-chain pool state towards an off-chain model.

The complete set of operations can be found in CoinPool paper v0.1.

## Why new Contracting Primitives are required ?

A payment pool construction relies on a pool withdrawal tree, a set of transactions of which the
combinations of confirmations always ensure to a participant that the pool balance can be redeemed on-chain,
without trusting the other participants. As the combinations of withdrawals can be assumed to be
unknown at pool setup to guarantee fault-tolerance, the computational complexity of the combination
is factorial.

Assuming the pool withdrawal tree Script mechanism relied-on is a Taproot tree, the factorial complexity
is reflected on three components: the merkelized tree of alternative scripts, the Taproot internal pubkey,
the output amount.

This factorial complexity could be solved to introduce a Script extension enabling Taproot output editing
across a chain of transactions such as TAPROOT_LEAF_UPDATE_VERIFY and an output amount introspection
extension like SIGHASH_GROUP flag advanced semantic or IN_OUT_AMOUNT opcode.

Hashchain-based types of covenant like CTV could be useful for another approach of payment pools, like
Radixpool, where the set of participants are segregated in pre-defined partitions. This technique reduces
the generation complexity of the withdrawal tree to O(log(n)).

## Payment Pool Analysis

### Game-theory Robustness

While the set of participants shares a pool of funds together, the unilateral actions of a single party
could trigger reactions of all the other participants. E.g, an old state being broadcast triggering an
automatic reaction of latest state broadcast, this latest state should be committed with sufficient
on-chain fees. Those fees could be substracted from the participant fee-bumping reserves at the worst-time.

In another direction, if there is a permanent asymmetry of stakes, a single participant could leak the
pool balances distribution history to a pool fungibility deanonymization attacker.

### Attacks & Risks

The security model is strongly equivalent to the Lightning Network protocol, with the same severity:
- [Pinning attacks](https://github.com/t-bast/lightning-docs/blob/master/pinning-attacks.md): the multi-party setting could even reveal harder scenarios.
- [Time-dilation attacks](https://arxiv.org/pdf/2006.01418.pdf)
- [Channel-Jamming](https://blog.bitmex.com/preventing-channel-jamming/): in case of cross-pools/intra-pools operations, variants of liquidity abuses are expected.
- Standardness-malleability: more complex covenant-enabled witnesses problably offer a wider surface attack to [CVE-2020-26895-like](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002858.html) vulnerability exploitation.

Beyond those generic attacks, payment pools designs should be secure and safe against more specific
concerns, such as out-of-order sequence of transations freezing a participant or replay of a withdrawal
transactions enabling double-spend of the pool funds.

### Economic Engineering

One of the design goal of advanced payment pools design is improving Bitcoin scalability on three metrics:
- UTXO set (a.k.a "onboarding scaling"): this metric defines how many users could share an UTXO in an off-chain construction.
- payment throughput (a.k.a "transactional scaling"): this metric defines how many transfers can be performed off-chain per on-chain transaction.
- fees-measurable user probabilistic cost: this metric defines how many satoshis a user has to spend to benefit from the pool payment in average.

Improving on the first and third dimensions would likely need improvements on few factors entering in the pool withdrawl ability:
- the number of transactions to enter/exit the pool.
- a minimal witness size to proceed with an unilateral exit of the pool.
- a covenant-mechanism in the line of [OP_EVICT](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019926.html), to batch withdrawal.
- a more compact accumulator than merkelized tree of script for pool balance set membership verification.

### Privacy Attacks

The confidentiality of balance owners and the history of balance updates should be preserved both
against party external to the pool, and (ideally) against in-pool participants. To achieve the
first goal, cooperations failures among the participants should reveal the less information possible.
To achieve the second goal, ZKP could be leveraged to hinder the links between the intra-pools
update flows from the view of the participants themselves.

Payment pools deployment should be also robust against [cross-layers links](https://bitcoinproblems.org/problems/removing-cross-layer-links.html),
where probe on one layer would provoke data spikes on the other one (e.g withholding a pool update from
Alice, to observe that a unilateral withdrawal transaction is originally broadcasted by a full-node,
potentially Alice's one).

### Implementation Complexity

The implementation complexity of a payment pool design such as CoinPool should be probably one order
of magnitude more complex than a LN implementation, especially noting state machine relying on a
consensus algorithm to synchronize the pool updates order among the participants.

Many components of a Lightning implementation could be reused once the necessary level of abstractions
have been conducted: on-chain monitoring, watchtower support, networking stack, invoice logic.

### Protocol unknowns

At least two major unknowns sounds to affect payment pools constructions, on a scale hindering long-term
usability and safety.

Multi-party contract protocol like Lightning or payment pools present high sensibility to the confirmation
of transactions. In face of [mass network mempools congestion](https://lightning.network/lightning-network-paper.pdf),
the confirmation odds of all broadcasted time-sensitive transactions could reveal very low, breaking the
funds safety.

Another major limitation of payment pools is the high interactivity issue, where all the payments pools
participants have to be online to move the state forward. The pools could be partitioned in subsets of
participants, with different degrees of onliness requirements or in function of onliness history.
[Few techniques](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-April/020370.html) could
be explored to achieve this functionality safely and securely.

### Protocol evolvability

Advanced payment pools design aims to be plugged-in by Lightning channels, where every pool balances
could be the endpoint of 2-of-2 LN channels. This would be a privacy improvement, as the channel
wouldn't be observable by the remaining pool participants, and the interactivity requirement lessened,
as only the presence online of the two parties is required.

Each balance could be owned by a Fedimint gateway or a multi-party signature group like MuSig2 to enable
further scaling.

### Ecosystem Impacts 

Payment pools should scale the number of Bitcoin users able to share the ownership of a Bitcoin UTXO
while conducting economic operations like payments or advanced contracts. This scalability increased
coming without additional validation requirements of full-nodes (even if new type of accumulators
are introduced, as long as the CPU cycles to verify the witnesses are not miscompensated by fees)

The diminishing on the reliance of payment paths to achieve payments inside the pools should also decrease
or minimize the liquidity price across of the ecosystem.

Payment pools do not appear to introduce Bitcoin systematic risks or centralization vectors or alterations
in the mining economics.

### Sources

- Joinpool description by Greg Maxwell: https://gist.github.com/harding/a30864d0315a0cebd7de3732f5bd88f0
- CoinPool OG ML post by Gleb Naumenko / Antoine Riard: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-June/017964.html
- Coinpool paper V0.1 by Gleb Naumenko / Antoine Riard: https://coinpool.dev/
- Radixpool ML post by Jeremy Rubin: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-June/017968.html
- TAPROOT_LEAF_UPDATE_VERIFY pool use-case by Anthony Towns: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019419.html
