# The "Evil Covenants" Risk

Problem: a Big Tech or international financial institution could decide to monitor
all the Bitcoin economic flows for out-of-band purposes (e.g targeted advertising).
To implement this monitoring, recursive covenants could be leveraged where the coins
targeted would be encumbered by a signature from the institution as part of the locking
script, and this signature encumberance inherited on the subsequent chain of spends.

This issue would be detrimental to some fundamental properties of Bitcoin, such as
coins fungibility, i.e how interchangeable individual units of Bitcoin is. The subset
of monitored coins could have a market value different from the regular coins, and as
such increase the transactional costs of the actors dealing with them. Beyond, the
signature encumberance requirement would also decrease the velocity of the flows, as
any operation would need interaction with some signing servers. Any downtime of the
signing servers would also paralyze the exchange of the monitored coins.

As recursive covenants could enable that type of generalized Script policies over a
wide set of coins, and those policies would be harmful for some properties of Bitcoin
as a system, contracting primitives introducing recursive expressivity could be evaluated
as beyond the Bitcoin community common risk tolerance.

On one side, "harmful" encumbering policies could be implemented today by leveraging
[basic multisignature](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-July/019209.html), whereve every spending transaction address would follow the same rules, and the spent
being refused in case of non-adherence. As such recursive covenants could be thought as not
introducing new level of "harming" flexibility.

On the other side, it could be considered that immutability of the recursion would be
a change in the monitoring enforcement costs, as once encumbered there is no option for
the coin owners to remove the Script policy.

A complete evaluation of the "Evil Covenant" risks is left as a subject of research.
