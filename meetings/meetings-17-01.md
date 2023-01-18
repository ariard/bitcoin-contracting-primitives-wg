18:00 < ariard> #startmeeting
18:02 < ariard> for the ones who are not familiar with the concept, "vaults" can be described as a generic technique to designate a secure storage of Bitcoin funds
18:03 < ariard> with pre-defined spending policy enforced by covenants
18:03 < jamesob> shilling https://jameso.be/vaults.pdf (I've gotta PR to your link, ariard)
18:03 < jamesob> (not shilling as a proposal per se, just a description of vaults)
18:03 < ariard> like you send the funds to a deposit address, this address must commit to a serie of transactions encoding diverse paths
18:04 < ariard> yeah i still don't know if documentation should just be exhaustive, like try to track everything is existence or try to do digest, jamesob
18:04 < halseth> hi
18:05 < ariard> for vaults, i believe the set of relevant primitives are like CTV,APO,OP_VAULT, i don't know if we have TLUV-based vault proposals?
18:05 < gleb> all these "crazy things we can do with OP_UNVAULT" kinda converge me to "yeah there is no way around CTV-like stuff", feeling like we're close to exhausting the space around. Anyone else got this 
              feeling or it's just me? :)
18:06 < _aj_> ariard: there's a TLUV based vault sketch on the mailing list a while ago i think
18:06 < roconnor> probably key properties of valuts are two (at least) phase, on-chain, and revorkable (better term?) spending policy.
18:06 < jamesob> gleb: yeah I'm with you I think
18:06 < _aj_> roconnor: ("revocable")
18:06 < roconnor> ty
18:06 < jamesob> _aj_'s going as clippy for next halloween
18:07 < gleb> Collecting all attempts to do vaults with covenant in a summary might be a good idea for the repo
18:07 < ariard> _aj_: yeah missed that one from current sources in `use-cases/vaults.md`
18:08 < ariard> gleb: well that's what `vaults.md` try to be :)
18:08 < roconnor> gleb: I literally was thinking the same thing.  If vaults are equipowerful of CTV, it makes a strong case for developing a primitive that encompasses this behaviour.  I think this is an 
                  exciting result.
18:08 < rgrant> gleb: i like the OP_VAULT niceness with not losing coins if someone sends to it twice, but "why not both?" ;)
18:09 < jamesob> OP_VAULT is neither a subset nor a superset of CTV I think... though if it's either, it'd be a superset
18:09 < gleb> ariard: it seems like it describes everything, except summarizing architecture of each covenant vault construction...
18:09 < instagibbs> rgrant, can you expand on that issue? I haven't thopught deeply about reuse of CTV
18:09 < _aj_> roconnor: collecting soft forks for inquisition v24.0 btw; forward ports of CTV/APO are underway. maybe you want to suggest/pr CAT/CSFS/similar? 
              https://github.com/orgs/bitcoin-inquisition/projects/2
18:10 < ariard> gleb: https://github.com/ariard/bitcoin-contracting-primitives-wg/issues/29 :)
18:10 < gleb> ariard: perfect
18:11 < _aj_> roconnor: vaults are a superset of CTV, i think; the TLUV vault assumed CTV was also available; CTV falls out of jamesob's OP_UNVAULT. (an APO can simulate CTV as well, ofc)
18:11 < rgrant> instagibbs: my understanding is that presigned vaults need to make withdrawals in predetermined amounts (of which they can be overprovisioned to afford many options), due to 
                deleted-key-equivalent requirement, so if someone else sends to the address, then the unvault could have a problem.  
18:11 < instagibbs> oh sorry, misunderstood your comment, nvm 
18:12 < instagibbs> thanks for explaining
18:12 < gleb> _aj_: what's a good way to review forward ports? Assuming i reviewed the previous merge
18:13 < _aj_> haha, great question! let me know if you have an answer :-/
18:13 < _aj_> range-diff kind of works, but doesn't deal with fixups that well
18:14 < gleb> alright, at least it feels better to not i'm not wasting time completely doing everything from scratch
18:14 < roconnor> Oh superset.  Interesting.
18:15 < ariard> gleb: on the "yeah there is no way around CTV-liked stuff" about vaults, I still wonder if adding a sig in the redeem path, in addition of the pre-committement with hashes
18:15 < jamesob> ariard: why would you want to do that? just seems like extra weight
18:15 < ariard> isn't better as you can have an authorization structure, otherwise leak of the preimage scriptpubkey like with recovery transaction in the OP_VAULT scheme can be quite harmful
18:16 < gleb> ariard: how does that relate to my comment/CTV? You still enable this very covenant behaviour.
18:16 < jamesob> the "authorization structure" is the creation/announcement of the OP_UNVAULT in the first place
18:16 < ariard> jamesob: let's say you would like to know who broadcast your unvault or recovery transaction among all your custody teams, to detect where there is an infra issue or something wrong
18:17 < jamesob> I dunno if I follow... if you have multiple custody teams, they're going to need to share the preimage of the <target-hash> to verify that the UNVAULT is legit
18:18 < ariard> gleb: just wondering if we have exhausted the space, or if there are other semantics that could be valuable 
18:18 < _aj_> ariard: i feel like you can assume the answer to "have we exhausted the space" is always "no"...
18:19 < jamesob> ariard: if you want to do something like that, the <unvault-spk-hash> could correspond to a taproot script-path that uniquely identifies the spender at unvault "trigger" time
18:19 < jamesob> you could then ID the custody team based on the TR control block of the spend that creates the OP_UNVAULT output
18:20 < ariard> jamesob: yes and this taproot script-path would have a OP_CHECKSIG, i think you would achieve the same "spender identification" effect I've in mind
18:20 < _aj_> jamesob: just have an explicit 1-of-n multisig, and have a different key for each spender?
18:20 < instagibbs> I feel like accountability is pretty trivial? maybe im misunderstanding the issue here
18:20 < jamesob> _aj_: there ya go, there's that bigbrain simplification
18:20 < instagibbs> what _aj_ said, just use accountable sigs
18:20 < _aj_> jamesob: midwit.gif?
18:21 < jamesob> yup
18:21 < ariard> instagibbs: if the state are symmetrics and you have N custody teams, accountability is far from being self-evident, i think
18:21 < gleb> i think petter todd suggestat that a while ago, and that even has a name... i saw on twitter?
18:21 < gleb> like a key per server to track when it's stolen
18:23 < ariard> if we're good with vaults, let's move on with next use-case
18:23 < ariard> so we have payments pools: https://github.com/ariard/bitcoin-contracting-primitives-wg/blob/main/use-cases/payment-pools.md
18:24 < ariard> briefly, payments pools can be described as multi-party ownership of a single utxo where the participants balanced are encoded in a set of off-chain transactions
18:24 < ariard> s/balanced/balances/g
18:25 < ariard> the withdrawal from the payment pool should be non-interactive and trustless, compared to a simple N-of-N multisig
18:26 < ariard> the primitives enabling this construction are probably things like TLUV,APO for the update mechanism, and something something for covenanting the amount 
18:26 < ariard> independently of the withdrawal order
18:27 < halseth> As mentioned in one of the issues, I'm interested in whether the pool can be generalized to not only supporting a single pool exit per transaction, but withdrawing more than one of the 
                 "balances" with a single transaction
18:27 < halseth> The use case I have in mind is claiming m of n HTLCs locked in the pool
18:28 < ariard> halseth: ideally, you should be able to combine the withdrawal witness non-interactively and add the balance outputs on a single transaction
18:28 < ariard> maybe with a modification of SIGHASH_GROUP allowing intersection of committed outputs among bundles
18:28 < gleb> halseth: in that direction, I'm curious whether TLUV is an overkill and it might be easier :) should think more
18:28 < ariard> as you would like the payment pool clawback output to be common among all bundles
18:29 < ariard> (or the commitment balances outputs to be common among all bundles, if you have the LN channel use-case in mind, i believe)
18:29 < gleb> halseth: nevermind, i should read your github post once again
18:30 < halseth> yeah, would be cool if all the commitment outputs coould just be a single pool output :)
18:30 < jamesob> IMO this is the "holy grail" of bitcoin scaling, if it can be done. The recurring problem seems to be facilitating on-chain withdrawal of multiple balances without requiring the whole pool to 
                 recoordinate interactively
18:31 < halseth> hm, yeah. I think the TLUV direction is promising though, in that you can sort of spend more than one branch simultaneously 
18:32 < halseth> (I might be misunderstanding TLUV)
18:32 < instagibbs> this is where sequencers seem necessary imo jamesob 
18:32 < ariard> jamesob: yes -- the hard issue is partitioning: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-April/020370.html :/
18:32 < jamesob> I totally don't understand TLUV I guess; I just thought TLUV allows you to swap out a leaf in the taptree, which would only affect spending conditions and not e.g. amounts etc.
18:33 < jamesob> instagibbs: where can I read about these "sequencers?"
18:33 < halseth> instagibbs: why not have the fee market decide who goes first?
18:33 < _aj_> jamesob: TLUV doesn't enforce the amount stays constant, you use additional new opcodes for that. that allows you to swap "pool{3BTC - A,B,C}" to be "pool{2BTC - A,C}" with an extra output {1BTC, 
              B}
18:33 < gleb> jamesob: that's correct at the high-level, but then at the low level amounts must also be handled to make signatures work
18:34 < instagibbs> parlance from rollups land, basically some entity or group of entities that allows batched updates, but all users can unilaterally exit one by one
18:34 < instagibbs> halseth, one state transition per block can be very slow if you imagine a 100+ person utxo...
18:34 < jamesob> _aj_ gleb: ah thanks. I'll need to think more about it
18:35 < halseth> instagibbs: sure :p
18:35 < halseth> need faster blocks!
18:36 < ariard> instagibbs: i think in the rollups land you have some batch producer or coordinator, and idk if there has been a viable proposal so far for a decentralized one...
18:36 < instagibbs> ariard, i suspect if the rollup industry hasnt found it, it's gonna be hard to find
18:37 < instagibbs> I'd look there first for inspiration, in other words
18:37 < rgrant> there was a demo of a sequencer at TABconf2022, from Judica.  however, the trust requirements were different than typical on-chain activity.
18:38 < instagibbs> Ideally it's a liveliness add-on only, individuals can continue updating state without interaction at a larger cost
18:38 < ariard> if we're good with payments pools, let's move on with next use-case
18:39 < halseth> that would be nice, I can just add my witness and output to an existing pool withdraw tx
18:39 < halseth> (and pay a fee to replace the existing one ofc)
18:39 < gleb> there are also pools based on CTV but they require some interaction right?
18:40 < ariard> gleb: yep radixpool: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-June/017968.html
18:40 < ariard> iirc, in case of withdrawal by someone owning a balance in the same branch of transactions, you're forced to go on-chain too
18:40 < ariard> like it's not individual withdrawal
18:41 < halseth> choose your pool buddies wisely in other words
18:42 < ariard> so there is optimizing coinbase payouts with covenants use-case:https://github.com/ariard/bitcoin-contracting-primitives-wg/pull/8
18:43 < ariard> basically, from my understanding how thousands of miners payouts outputs (or balance if you can swap directly with a LN channel) could be aggregated in a single coinbase scriptpubkey
18:44 < ariard> (as discussed on the PR, the problem might be very similar to a payment pool in fact, with few tweaks)
18:44 < ariard> as primitives you could use here, i guess probably something like CTV or the "script-in-the-sig" allowed by APO
18:45 < rgrant> it seems like there's an opportunity to reduce pressures for miner centralization here.  any complications to that conclusion?
18:46 < ariard> well the scheme is probably not trustless, as how do you ensure binding between the shares sent to the pool and the equivalence between your payouts outputs?
18:46 < gleb> rgrant: it's only a block size optimization, and definitely not the biggest challenge in that direction (i imagine this protocol for mining)
18:47 < jamesob> I think I wrote about this here? https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-August/020869.html
18:47 < jamesob> "If the payout was instead a single OP_CTV output, an arbitrary number
18:47 < jamesob> of pool participants could be paid out "atomically" within a single
18:47 < jamesob> coinbase."
18:48 < rgrant> ahh, so this is a different use case than, eg. https://utxos.org/uses/miningpools/
18:49 < jamesob> rgrant: yeah I think Jeremy's example there is a more complicated idea
18:50 < ariard> with the Jeremy's example, I'm still unsure you there is a script-level enforcement of the binding between miners shares and payout outputs amount
18:51 < ariard> according to your mining pool shares reward policy
18:52 < ariard> let's move on to next use-case, congestion control documented here: https://rubin.io/bitcoin/2021/12/09/advent-12/
18:52 < halseth> yeah, this seem a bit diff
18:52 < halseth> (jeremys pool)
18:53 < ariard> i wonder if you don't have a bit of functionality redundency between congestion control and more recent proposals like non-interactive channels without committing to amount: 
                https://github.com/ariard/bitcoin-contracting-primitives-wg/issues/19
18:54 < ariard> like in both cases, i think you would like to give a "blessed address" to an exchange or LSP, and at a latter time non-interactively they can pay you
18:55 < ariard> then we have few other use-cases like multi-party scalable DLC use-case
18:56 < ariard> discussed here https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019859.html
18:56  * rgrant going AFK on the hour
18:56 < ariard> where a high order of parties could all commit to a single DLC
18:56 < ariard> like having a 100-takes-of-1000-makers DLC
18:57 < ariard> yeah we'll finish on the hour! if someone has covenant-enabled use-cases they're thinking would deserve archiving?
18:58 < ariard> then we have a bunch of other use-cases like ZK rollups, documented here: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-October/020998.html
18:58 < halseth> anybody have the insight to what opcodes/covenants would be needed for the validity rollups?
18:59 < JohnLaw> I'd propose scaling the creation of channels beyond the bound possible with signatures, as signatures require each of N parties to approve the other N-1 parties
19:00 < ariard> i think you would need some ZKP for validity rollups, to verify the batch. And then you enter in all the ZKP tradeoffs...
19:00 < ariard> #endmeeting
