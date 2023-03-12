# ANYPREVOUT

A new type of public key for tapscript allowing signatures for these publics keys to not commit to the exact UTXO
being spent. It enables dynamic binding of transactions to different UTXOs, on the condition they have compatible
scripts.

## "Two-party Eltoo" open issues

- how to add fees ?
- how to pay watchtowers ?
- lack of layered transactions and timelocks unblunding ?
- extending to multi-party channels/payment pools ?

## Functional redundancy

The Eltoo mechanism offers some functional redundancy with the Inherited IDs idea.

### Sources

- BIP118 signature full tapleaf path commitment: https://github.com/bitcoin-inquisition/bitcoin/issues/19
- BIP 118: https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki
- "Two-party eltoo/w punishment":  https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-December/003788.html
