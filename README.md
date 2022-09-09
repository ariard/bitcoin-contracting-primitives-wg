# Bitcoin Contracting Primitives Working Group

The goal of this project is to host R&D production of the Bitcoin protocol development
community around contracting primitives powering new class of Bitcoin use-cases (vaults,
statechains, payment pools, ...).

By contracting primitives, the scope is left open to encompass few techniques which
have been proposed in the past: [covenants](https://fc17.ifca.ai/bitcoin/papers/bitcoin17-final28.pdf), [capabilities](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-December/019722.html), [transaction introspection](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-July/020753.html), etc.

This project aims to adhere to the following procedural principles:
- openess: all contributors of the Bitcoin technical community are welcome, whatever the level of knowledge and experience
- decentralization: project maintenance would be distributed among stakeholders with time
- neutrality: ideas should be presented according to the best engineering and scientific standards among the Bitcoin protocol development community in an objective fashion

Principles wider interpretation are left to everyone's personal subjectivity. As in matter of FOSS standards, the performance of
them is function of how much stakeholder contributed time and energy in a best-effort.

Softwork activation discussion or primitives deployment timeline are considered as off-topic for this
project. 

The repository organization proposed is the following:
- use-cases: use-cases presentations, contracting primitives usage justification, use-case analysis
- constraints: design bounds historically raised by the community about new contracting primitives
- primitives: proposed changes to Bitcoin such as TLUV, APO, CTV, CSFS
 
It should be noted that use-case analysis (economic, security, privacy, ... trade-offs) aims to be
deliberately exhaustive, as a use-case architecture could be potentially revealed itself as
non-functional or a trade-off downside nullifying the technical interest, and as such the
relevantness of the contracting primitive.

Regular meetings aimed to be scheduled starting in November 2023. As of block #753360, the
communication channel(s) where to host them has not find yet consensus.

Pull requests, issues and reviews are welcome, seeking feedbacks from community stakeholders.

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
