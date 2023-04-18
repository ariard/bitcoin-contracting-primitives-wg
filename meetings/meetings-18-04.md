```
18:00 < ariard> #startmeeting
18:00 < ariard> hi!
18:02 < ariard> so the interactivity issue is one of the problem showing up when you start to have a high number of participants in your off-chain construction
18:02 < JohnLaw> sound good!
18:03 < ariard> for 2-party LN channels, to advance the state forward, you need a signing contribution from your counterparty
18:03 < ariard> afaict this is true as much in the LN-penalty paradigm than the Eltoo ones
18:04 < ariard> for the 2-party setup, the interactivity issue is already issue in case of pre-committed fees
18:04 < ariard> without dynamic management, e.g anchor output
18:05 < ariard> i think this issue comes worst as the number of participants in the construction increases
18:05 < ariard> as you have more risks of one being offline and stucking off the state advance
18:05 < ariard> is that a correct description of the interactivity issue for off-chain constructions?
18:05 < ariard> what we can add more?
18:06 < JohnLaw> that description sounds good to me, but I would add that the interactivity issue gets worse with casual users on smartphones
18:07 < ariard> yes you cannot assume smartphones are reliably online 24/7
18:08 < ariard> and i think there is the fact that your liveliness risk is not only receiving/sending a payment
18:08 < ariard> but can also happen in fact of network mempools congestion, as said if you don't have dyanmic fee management
18:09 < ariard> so one way to alleviate the interactivity issue for smartphone would be to delegate the signing keys?
18:09 < ariard> though we can do this in trust-minized fashion? or with trade-offs?
18:10 < JohnLaw> sure, that's a possibility... however, I believe we should create trust-free solutions whenever possible
18:11 < ariard> well if you have a m-of-m scheme like musig2, you can give 1 signing share
18:11 < ariard> to each watchtower or something like this, and as long as 1 is honest?
18:11 < ariard> your funds should be safe?
18:12 < ariard> ...though this is still a reliance on a third-party
18:12 < JohnLaw> sure, there is a whole range of trust-minimized solutions. It's just that I think trust-free ones are better, if possible.
18:13 < ariard> yes, the thing is what set of elements we consider in a scope of a trust-minimized solutions :)
18:13 < ariard> like can we design a solution that minimize reliance on network mempools congestion?
18:13 < JohnLaw> agreed
18:13 < ariard> or weird miners incentives?
18:14 < ariard> because it sounds to me all non-sidechains constructions are in fine relying on time-sensitive confirmations
18:14 < JohnLaw> agreed
18:15 < ariard> though even side-chains you have some refund/espace path to exit the federations, no?
18:15 < ariard> i don't remember here
18:15 < ariard> now a very interesting question can you design a solution to the interactivity issue
18:16 < ariard> which doesn't encumber a risk of loss of funds if you're time-sensitive transactions does not confirm?
18:16 < ariard> or like there is not even responsibility of the "honest" counterparty to react to chain events
18:17 < ariard> the cheater is just blocker to redeem the funds due to some contracting primitives
18:18 < ariard> so a while back in the covenant space, there was the idea to separate the "value" transaction
18:18 < JohnLaw> That's a good question! The best answer I've been able to come up with is the creation of a more flexible notion of time that would not advance until fees become low enough to guarantee 
                 the required time-sensitive transaction (with some agreed upon level of fees) can be put in the blockchain.
18:18 < ariard> from the "signal" ones, inspired by EE
18:19 < ariard> Yes, there was a paper published at last year FC about "congestion period" that would shift in function of block congestion
18:20 < ariard> though it was only for the eth space, sounds quite similar to the "flexible notion of time" you're describing?
18:20 < JohnLaw> Interesting. Do you have a reference?
18:20 < JohnLaw> yes, it sounds similar
18:20 < ariard> (searching the paper rn, give me 1 min)
18:21 < ariard> this one: https://fc22.ifca.ai/preproceedings/119.pdf
18:21 < JohnLaw> Thanks!
18:22 < ariard> "To overcome these problems and improve challenge-response protocols, we suggest a secure mechanism that detects congestion in blocks and adjust the deadline of the response accordingly"
18:23 < ariard> it sounds to me you could tweak this for bitcoin, let's say you have an annex tag
18:23 < JohnLaw> I'll read it, thanks
18:24 < ariard> that commits to a feerate over a period of X blocks (to prevent manipulation by minority mining coalitions) 
18:24 < instagibbs> IIRC they use the consensus level "base fee" as the metric to bench against
18:24 < ariard> and as long the feerate of your transactions stay above this "feerate" tag
18:25 < ariard> the timelocks could be extended
18:25 < ariard> yeah...that's the hard thing which "base fee" you bench against?
18:25 < instagibbs> it's a core part of ethereum, so they "just" reuse it
18:26 < ariard> yes i think you need 2 components for a feerate-lock mechanism:
18:27 < ariard> a) a transaction-level tag
18:27 < JohnLaw> sounds good. I've also brainstormed on using a chain of transactions with fixed fees and fixed relative timelocks (e.g. 1 day), with each transaction in the chain being the root of a 
                 tree creating control outputs that can be consumed by time-sensitive transactions. The idea is that the chain transactions won't be in the blockchain until fees are low enough.
18:27 < ariard> and b) an endpoint chainstate like MTP is used by epoch-based nLocktime?
18:28 < JohnLaw> Another variant is to just have the chain of transactions with fixed fees, and to make a Bitcoin consensus change that allows the UTXO from the latest transaction in the chain to be used 
                 in scripts in time-sensitive transactions without spending that UTXO.
18:29 < ariard> so for the first idea, it's a feerate-lock just covering a whole chain of off-chain transactions? 
18:29 < ariard> like you have a channel factory, and each sub-channel counterparties decide their fees levels
18:29 < ariard> in function of their timevalue preferences?
18:30 < JohnLaw> The first idea doesn't require any change to Bitcoin consensus. It just lets the time-sensitive transaction have an input that is a leaf of a tree rooted in the chain.
18:32 < ariard> so you mean you have two paralell off-chain constructions and in function of update of tree leaves A
18:32 < ariard> you can confirm time-sensitive transaction of B, something like this ?
18:32 < JohnLaw> Yes, each sub-channel establishes the level of fees that they are willing to pay, and they make their time-sensitive transactions consume an output from a leaf of a tree rooted in a 
                 chain of transactions with the desired fee level (different chains with different fee levels could co-exist).
18:33 < JohnLaw> yes
18:33 < ariard> so time-sensitive transactions consuming outputs are counter-signed with SIGHASH_ANYONECANPAY
18:34 < ariard> and then at a later time, the broadcaster finalizes by consuming the fee with desired fee level?
18:34 < JohnLaw> yes and yes
18:35 < ariard> the tree we're talking about is a Taproot tree or a tree of transactions?
18:35 < JohnLaw> although I should add that one has to trust the chain creator, or else use covenants for the chain and the trees it creates
18:35 < ariard> like i think you can do it with a Taproot tree, though the size of witness increase
18:35 < JohnLaw> tree of transactions rather than taproot tree
18:35 < ariard> well you can include one of your keys in the chain of transactions?
18:36 < JohnLaw> but this is just one brainstorm, and I don't want to derail anything here
18:36 < ariard> well it's interesting, folks have been searched better fee-bumping primitives for years :)
18:37 < ariard> designing an off-chain construction is easy, designing a fee-safe off-chain construction is hard
18:37 < JohnLaw> if we have covenants, then a group of users could co-fund a chain of transactions with the given fee rate
18:37 < ariard> though yeah we can go back to the interactivity issue, or other thing?
18:38 < JohnLaw> interactivity sounds interesting to me
18:38 < ariard> hmmmm, but you cannot do this with multi-sig today like a group of users co-funding a chain of transactions?
18:39 < JohnLaw> (yes, co-funding the chain is awkward. Maybe via LN?)
18:40 < ariard> yeah sounds to me you can do it with multisig..though probably more space efficient with CTV-like primitives?
18:41 < ariard> okay we can go back to the interactivity issue :)
18:41 < ariard> so if you have a payment pool or channel factory
18:42 < ariard> with N users, as sounds you have 1 user offline, the state is stuck
18:42 < ariard> one solution can be to segregate all users in 2-party LN channels
18:43 < ariard> though i believe the issue is back as sounds you have intra-pool liquidity inbalances?
18:44 < JohnLaw> yes, at least LN channel capacity resizing is an issue, but that can be solved pretty well with hierarchical channels
18:44 < JohnLaw> even hierarchical channels with just 3 or 4 users
18:45 < ariard> like how hierarchical channels are working, compared to the classic Decker factory?
18:45 < JohnLaw> yes
18:46 < ariard> is there a tree of transactions with decrementing timelocks like with the duplex thing?
18:46 < JohnLaw> no
18:47 < JohnLaw> the best implementation of hierarchical channels uses a "tunable penalty" approach with separate value transactions and control/penalty transactions
18:48 < JohnLaw> I sent something out about this on the lightning-dev mailing list and Dave Harding did a write-up on it in an Optech Newsletter
18:49 < ariard> ah okay this one the best write-up available: https://bitcoinops.org/en/newsletters/2023/03/29/ ?
18:49 < ariard> (sometimes late on the Optech newsletter)
18:49 < JohnLaw> yep!
18:49 < instagibbs> _aj_ had a pretty readable thread on it recently
18:50 < ariard> on the separation between value and control transactions, i think there was an old prez of jeremy
18:50 < ariard> here: https://rubin.io/public/pdfs/multi-txn-contracts.pdf
18:51 < ariard> intuitively, i would say it might makes things more costly?
18:52 < ariard> in terms of chain space as you have 2 transactions to confirm for each state, no?
18:54 < JohnLaw> Yes, there are more transactioins to put on-chain in the unilateral close case. However, it has a lot of advantages, too, including the ability to resolve an HTLC without closing the 
                 factory or hierarchical channel, scaling to more than two users efficiently, having small cltv_expiry deltas, having tunable penalties, and enabling watchtowers with logarithmic (rather 
                 than linear) storage.
18:54 < JohnLaw> (thanks for the ref to Jeremy's paper; I'll check it out)
18:57 < ariard> Sounds very cool - On the ability to resolve a HTLC without closing the factory/channel, there was a discussion here: https://github.com/ariard/bitcoin-contracting-primitives-wg/issues/26
18:57 < ariard> Okay from browsing over Dave write-up, it's a tunable penalty because it can be negotitated by the users for each state
18:57 < ariard> And then only enforced with the value transaction?
18:58 < ariard> (though what's the game theory behind adjustable penalties, e.g in function of your counterparty level of blame?)
18:58 < JohnLaw> (thanks for the issue 26 ref, I'll check it out)
18:59 < JohnLaw> the amount of the penalty can be chosen arbitrarily by the participants. The penalty is paid by a use that puts on old control transaction on-chain.
19:00 < JohnLaw> use=>user
19:00 < JohnLaw> looks like we're out of time. Sorry if I used up too much of the conversation.
19:01 < ariard> oky will bind by the hour, though I'll keep thinking about TPP :)
19:01 < ariard> No worries, it's "freewheel" chat afterall, like good to share ideas!
19:01 < ariard> #endmeeting
```
