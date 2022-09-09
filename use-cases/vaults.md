# Bitcoin Vaults

A "Vault" is a generic technique to designate a secure storage of Bitcoin funds by relying on
pre-defined spending policies by the stakeholders. The spending policies can be "flashed" on
watchtowers with pre-signed transactions to enforce/correct the funds flows, or alternatively
consensus-level contracting primitives to achieve the same effect.

## Protocol Description

The description is a recollection of the community vault protocol proposals to represent an
abstract "vault" protocol.

The operations are the following:
- Authorization phase: the participants define a sequence of funds flows.
- Deposit phase: the participants gather the funds in a "cold" deposit address.
- Redeem phase: the participants move the funds from the "cold" address to the "hot" address, according to the defined spending policy.
- Refund phase: the participants refill the "cold" deposit address, in compliance with the defined spending policy.

It should be noted the order of the "Authorization phase" and the "Deposit phase" can be
inverted in function of the covenant technique. If the vaulting transactions are pre-signed
(e.g the Revault model), the spending authorization can happen after the deposit. If the
vaulting transactions are hashchained (e.g the CTV-based vault model), the spending authorization
should happen before the deposit.

A complete set of operations for a pre-signed model can be found in the practical Revault
specification.

## Why new Contracting Primitives are required ?

As of today, the design and deployment of a basic vault protocol is achievable without new
contracting primitives relying on consensus changes. A design can be sketched out with pre-signed
transactions exchanged among all the funds stakeholders. The key(s) can be [deleted](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-August/017229.html)
to avoid leaks leading to the reproduction of pre-signed transactions hijacking the fund flows.

Advanced vaults designs relying on covenants would enable:
- minimal scope of critical data (apart of amount/"hot wallet" pubkeys)
- minimize reliance on software/hardware wallets due to consensus-level enforcement of the flow

## Bitcoin Vault Analysis

### Game-theory Soundness

If the vault is shared between many stakeholders (beyond administrative boundaries) and the
asymmetry of funds is significant, the level of operational execution might differ. E.g, a
low-stake participant might be lazy in the maintenance of its watchtower infrastructure that
could open the door to vault data leaks (e.g the emergency descriptors), or any other operational
information that the protocol should aim to keep confidential.

In a setup with many stakeholders, there could be incentives of a one stakeholder to active
the cold path of the vault leading to time-locked enforced funds freezing, therefore blocking
the other stakeholders to participate in out-of-band economic opportunites (e.g moving the vault
funds quickly to a LSP to satisfy liquidity demands in period of LN congestion).

### Attacks & Risks

The security model is equivalent to the Lightning Network protocol, with a lesser severity:
- [Pinning attacks](https://github.com/t-bast/lightning-docs/blob/master/pinning-attacks.md): a cold/cancel transaction could be pinned until the (compromised) hot-wallet path is final.
- [Time-dilation attacks](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-December/002369.html): the unvault transaction could be blinded from watchtower until the (compromised) hot-wallet path is final.
- [Flood&Loot attacks](https://arxiv.org/pdf/2006.08513.pdf): opportunistic exploitation of mempools congestion where the cold/cancel transactions won't confirm until the (compromised) hot-wallet path is final.

It should be noted each of those attacks sounds to rely on a preliminary compromise of non-emergency
keys or anchor output keys.

### Economic Engineering

Vault designs could be optimized for single-user deployment, where simple spending policy are
encoded with a minimal number of vaulting transactions, as such lowering the fee-bumping reserves
requirements. Otherwise, in the case of multi-user high-stake deployment, it is expected that
the security and safety increased provided by many-steps vaulting transactions to be worthy the
high on-chain footprint, inducing more fees to be paid.

An alternative to render the usage of vaults affordable for all Bitcoin users could be to leverage
multi-signatures scheme such as [MuSig2](https://github.com/jonasnick/bips/blob/musig2/bip-musig2.mediawiki)
or [FROST](https://eprint.iacr.org/2020/852.pdf). Each member of the signing group would have a stake
in the vault.

### Privacy Attacks

By design, all the vault operations are playing on-chain, as such it's hard to hinder the semantic of
the vaulting transactions, especially if they're templated according to a standard. Taproot tree of
scripts could be leveraged to mask the number of emergency keys in the cold/cancel paths. Another trick
could be to use "common" relative timelock offset, otherwise the usage of special values could reveal
a user vaulting policies across a set of otherwise uncorrelated transactions.

All the communication flows between the vault entities should be treated carefully, as otherwise observable
data trails could reveal elements part of the active defense vault model, such as the number of watchtowers,
their level of fee-bumping reserves or the topology of their hosting infrastructure.

### Implementation Complexity

The implementation complexity of a vault is expected to be an order of magnitude lower than a Lightning one,
in the lack of a state machine to dynamically update the set of contracting transactions. The components
should be relatively similar such as on-chain monitoring, networking stack, rebroadcasting and fee-bumping
management, chain access/connection. Some components of a Lightning implementation could be reused once the
necessary level of abstractions have been conducted.

However, a vault implementation could reveal as far more picky if the key management tooling is itself
encompassed, such as [descriptors and miniscript support](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020423.html)
in hardware wallets.

### Protocol Unknowns

At least two major unknowns sounds to affect vault designs, on a scale hindering long-term usability
and safety.

Multi-party contract protocol like Lightning or vaults present high sensibility to the confirmation
of transactions. In face of [mass network mempools congestion](https://lightning.network/lightning-network-paper.pdf),
the confirmation odds of all broadcasted time-sensitive transactions could reveal very low, breaking the
funds safety.

Another concern would be the risk of [miner bribing attacks](https://eprint.iacr.org/2019/748.pdf)
to censor cold/cancel/emergency transactions. Considering the high-level of stakes with vaults, a
coalition of miners could be incentivized to risk block rewards and collected fees to gain
vault-censoring bribes. 

### Protocol Evolvability

Many features could be added with times:
- ability to withdraw "cut-of-funds" and send back the remaining value under vault clawback
- ability to withdraw percentage of funds
- ability to refeed the vault without requiring a new key ceremony/vaulting structure verification from the stakeholders

### Ecosystem Impacts

Vaults should significantly the offer of strong self-custody solutions to a wide range of Bitcoin actors,
especially long-term time preferences actors aiming to use Bitcoin as a store of value. The fine-grained
control of funds flows should also match the operational requirements of the actors in traditional finance.

It should be noted, the fee-bumping strategy of vaults, vetted with a higher budget, could be a source of
disequilibrium towards "lightweight" L2s like Lightning in case of mass mempool networks congestion.

## Sources

- Vault by Russell O'Connor: https://blog.blockstream.com/en-covenants-in-elements-alpha/
- Vault by James O'Beirne: https://github.com/jamesob/simple-ctv-vault
- Vault by Jeremy Rubin: https://rubin.io/bitcoin/2021/12/07/advent-10/
- Revault protocol: https://github.com/revault/practical-revault
