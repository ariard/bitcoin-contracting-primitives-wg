# Bitcoin Contracting Primitives Working Group

The goal of this project is to host R&D production of the Bitcoin protocol development
community around contracting primitives powering new class of Bitcoin use-cases (vaults,
statechains, payment pools, ...).

By contracting primitives, the scope is left open to encompass a few techniques which
have been proposed in the past: [covenants](https://fc17.ifca.ai/bitcoin/papers/bitcoin17-final28.pdf), [capabilities](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-December/019722.html), [transaction introspection](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-July/020753.html), etc.

This project aims to adhere to the following procedural principles:
- openness: all contributors of the Bitcoin technical community are welcome, whatever the level of knowledge and experience
- decentralization: project maintenance will be distributed among stakeholders over time
- neutrality: ideas should be presented according to the best engineering and scientific standards among the Bitcoin protocol development community in an objective fashion

Principles that require wider interpretation are left to everyone's personal subjectivity. As in matters of FOSS standards, the performance of
them is a function of how much stakeholders contribute time and energy in a best effort.

Softwork activation discussion or primitives deployment timeline are considered as off-topic for this
project.

The repository organization proposed is the following:
- use-cases: use-cases presentations, contracting primitives usage justification, use-case analysis
- constraints: design bounds historically raised by the community about new contracting primitives
- primitives: proposed changes to Bitcoin such as TLUV, APO, CTV, CSFS
 
It should be noted that use-case analysis (economic, security, privacy, ... trade-offs) aims to be
deliberately exhaustive, as a use-case architecture could potentially be revealed to be
non-functional or present a trade-off downside nullifying the technical interest and hence reducing the relevance of the contracting primitive.

Regular meetings are organized. See #bitcoin-contracting-primitives-wg on Libera Chat. Past logs are [available](meetings/README.md).

Pull requests, issues and reviews are welcome, seeking feedback from community stakeholders.

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
