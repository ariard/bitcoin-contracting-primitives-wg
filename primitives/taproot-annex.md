# Taproot Annex

A new validation format is defined for the taproot annex. It allows to extend
the usual transaction fields with new data records allowing witness signatures
to commit to them.

## List of use-cases

- per-input lock-time
- historical block hash commitment
- sighash_group
- pay-with-neighboor-txid
- blockfeerate lock-point
- transaction weight lower/upper bounds locked-in

### Sources

- BIP 341 definition: https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
- Taproot Annex work-in-progress BIP: https://github.com/bitcoin/bips/pull/1381

