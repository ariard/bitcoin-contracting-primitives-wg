# Drivechain

Sidechains are alternative cryptocurrency systems where the native assets are Bitcoin UTXOs "virtualized" from the original
system of issuance. The virtualization is achieved via a pegging mechanism, where the asset can be imported and re-exported
between chains. A drivechain is a type of sidechain where the pegging mechanism is enforced by miner voting on the original
chain and where the proof-of-work is shared beween chains.

## Protocol Description

The description is a recollection of the drivechain research community, especially the recursive-covenant Drivechain design.

The operations are the following:
- The initial sidechain funds are locked in a recursive covenant UTXO.
- A sidechain block is issued by the sidechain validator.
- A spent of this sidechain UTXO and an external UTXO from the original chain is issued, where the second output commits to a sidechain block.
- At peggout, somebody proposes to withdraw some amount of funds from a sidechain UTXO to a mainchain address one.
- The miner enter into a voting period, if they agree the withdrawal is finalized.

## Why new Contracting Primitives are required ?

Designing drivechains as currently proposed are not technically possible with the primitves offered by Bitcoin.
- the original drivechain design relies on ["hashrate escrows"](https://github.com/bitcoin/bips/blob/master/bip-0300.mediawiki) and ["blind merged mining"](https://github.com/psztorc/bips/blob/master/bip-0301.mediawiki)
- a recursive-covenant drivechain design exists relying on the following set of new contracting primitives: OP_TLUV, OP_CAT, OP_CTV, OP_IN_OUT_AMOUNT

Those new primitives would enable drivechains, without relying on a federated system to peg the chains.

## Drivechain Analysis

### Game-Theory Robustness

"Merged-mining" could alter the current mining equilibrium. A subset of miners could be also bribed to delay the vote,
in a way inflicting a timevalue loss to a sidechain-withdrawal participant.

### Attacks & Risks

If the miners are competing to generate the sidechain-commitment transaction, a competing party could censor
the transaction-relay by censoring the broadcast transaction node.

Mempool-partitioning or pinning the sidechain-commitment transaction could be also considered, especially if
the peggout outputs are in the same transaction, and as such a single-owned utxo could be used as a pinning vector.

### Economic Engineering

The on-chain footprint is minimal to the sidechain-commitment transaction, where the onboarding cost of a sidechain participant
is equal to a single input spent. Onboarding participants could be batched where a Bitcoin UTXO owned by N parties could be
demultiplixified in N sidechains UTXOs. Offboarding N participants could be compressed in a single Bitcoin UTXO, if they
coordinate to put the funds under the control of an aggregated key.

Some design are using an OP_RETURN output to commit the sidechain blocks headers. More efficient technique like a Taproot
leaf could be used.

### Privacy Attacks

The sidechain might offer a [higher confidentiality transaction graph](https://scalingbitcoin.org/papers/mimblewimble.pdf)
than the original chain, as such hidding the pseudonymous address owning the coins.

Atomic cross-chain swaps could be leveraged to circumvent the peg graph as available to the sidechain validators.

### Implementation Complexity

A whole cryptosystem and a pegging subsystem must be developed, complexity is likely higher than a Lightning toolchain.

### Protocol Unknowns

Long-term economic equilibrium of the peg mechanism relying on an extension of miner incentives. Due to the protocol complexity
it might be hard to fully model the soundness under strong game-theory assumptions, before any practical deployment.

### Ecosystem Impacts

Sidechains enable permissionless innovation, and as such faster potential plateform of experimentation for future Bitcoin
consensus changes.

Issuance of new additional assets on such plateform could provoke economic disequilibrium of the Bitcoin system.

### Sources

- Enabling Blockchain Innovations with Pegged Sidechains: https://www.blockstream.com/sidechains.pdf
- Drivechain informational website by Paul Sztorc: https://www.drivechain.info/
- recursive-covenant Drivechain design by ZmnSCPxj: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019976.html
- APO-based Drivechain design by Jeremy Rubin: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-September/020919.html
