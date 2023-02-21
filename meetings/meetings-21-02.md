```
18:00 < ariard> #startmeeting
18:00 < _aj_> hi
18:00 < roze_paul> hi
18:00 < ariard> so today agenda is available here: https://github.com/ariard/bitcoin-contracting-primitives-wg/issues/34
18:00 < michaelfolkson> hi
18:01 < jamesob> hi
18:01 < bucko> howdy! 
18:01 < ariard> main subjects i'm thinking we can talk about are anyprevout and taproot annex
18:01 < rgrant> hi
18:01 < ariard> though do we have last minute topics :) ?
18:02 < _aj_> ariard: got your logging started this time?
18:02 < ariard> _aj_: lol, yes i think so
18:02 < bucko> related to eltoo, but maybe out of scope is the tapleaf malleability fix discussed in the mailing list? 
18:02 < adiabat> (belated hi)
18:03 < _aj_> bucko: https://github.com/bitcoin-inquisition/bitcoin/issues/19 (linked from the list thread)
18:03 < ariard> bucko: yes committing to the full tapleaf path? just added a ref in the eltoo entry
18:03 < bucko> yep, that's one 
18:03 < ariard> well we can start by this one
18:05 < jonatack> hi
18:05 < ariard> so in the context of multi-party transactions, you receive inputs from multiple unrelated parties
18:05 < ariard> receive corresponding outputs, and combine all of them in a single transaction
18:06 < jamesob> (similar situation with vault recoveries)
18:06 < ariard> the protocol flow should determine on whom is the responsibility of contributing to the fees
18:06 < ariard> like you have common fields (nlocktime, nversion, input index/output index)
18:06 < ariard> and per-counterparty fields, namely the output amount/scriptpubkey and the input nsequence/script sig/outpoints/witness
18:07 < ariard> right, though with vault recoveries, i think all the spend inputs are already templated? so malleability shoulds be bounded
18:08 < ariard> like the inputs witness weight are bounded by the recovery params, i believe
18:08 < jamesob> yep
18:09 < ariard> yeah the flow i'm thinking is the interactive transaction construction flow
18:09 < _aj_> i suppose we could have a standardness rule that annex entries are forbidden unless they're committed to by a signature?
18:10 < ariard> as we're using for lightning: https://github.com/lightning/bolts/pull/851
18:10 < instagibbs> _aj_, you mean for transaction inputs without any authorization(sigs)
18:10 < instagibbs> ?
18:10 < _aj_> if signatures commit to annexes from other inputs, we might need jamesob's "deferred checks" to implement that
18:11 < ariard> right, uncommited annex entries are problably a malleability extension vector too for that type of flows
18:11 < instagibbs> you mean in the context of op_vault? sorry, trying to keep up here
18:12 < _aj_> (i'm thinking that jamesob's OP_VAULT recovery path might allow for a no-signature spend, which someone could then add an annex too, inflating the tx size)
18:12 < instagibbs> (Ok, agreed)
18:12 < jamesob> _aj_ that's right
18:13 < rgrant> just to be clear, the attack being contemplated is that your counterparty advertises one fee during countersigning, but then broadcasts a lower fee, putting a burden on you?
18:13 < _aj_> (and yeah, i'm free associating a bit, so i'm having trouble keeping up too :)
18:13 < jamesob> lower fee_rate_ to be specific I think
18:13 < jamesob> because they're inflating the annex - really anyone observing the tx broadcast could inflate annex => lower feerate unless it's committed to somehow
18:14 < ariard> rgrant: your counterparty advertises one witness weight during the construction flow, and broadcast a smaller witness therefore increasing your fee burden, i think
18:15 < ariard> and could be a pinning vector: https://github.com/bitcoin/bitcoin/pull/19645 or an issue with transaction-relay propagation
18:16 < jamesob> but right now non-empty annex isn't standard, so not a concern?
18:16 < instagibbs> it's a concern if we ever wanted to use it, is the issue
18:16 < jamesob> gotcha
18:17 < ariard> well, i think this is type of standard rule you might want to get correct as soon as annex start to be used
18:17 < _aj_> (miner inflating the annex isn't a concern, because they're the one losing the fee income, and the only way they do it is by confirming the tx anyway)
18:17 < ariard> because annex fields might not be understood the same between non-upgraded and upgraded clients
18:18 < _aj_> ariard: unknown annex fields should render the tx non-standard anyway, so i think that's fine
18:18 < ariard> correct, so i'm non-upgraded client and your upgraded client, you give me an unknown annex fields (from my PoV)
18:19 < ariard> even with a signature committing to it, i should reject it
18:20 < ariard> and yes annex inflated at the miner reception level, can do already this with op_return payload i think 
18:20 < instagibbs> From my eltoo use-case, I would like to use 32 bytes of annex data, which is essentially wallet-scribbling on the blockchain
18:20 < instagibbs> I think jamesob has considered similar use cases for vault like structures(non OP_VAULT)
18:21 < _aj_> ariard: any SIGHASH_ALL sig will prevent adding/changing outputs like OP_RETURN
18:21 < _aj_> what do people think about introducing VLQ encoding to minimise the annex serialized size? https://github.com/bitcoin/bips/pull/1381#discussion_r1072280582
18:21 < jamesob> instagibbs: right; used to be talk of using OP_RETURN to back up presigned vault txn sig data (again, not OP_VAULT related)
18:22 < jamesob> I guess conceptually you could shove that data in the annex
18:22 < bucko> sorry to introduce another tangent, but instagibbs or jamesob mind providing details on this, not familiar : structures(non OP_VAULT)
18:23 < ariard> _aj_: right, though under bip341 the annex is already covered by the sig iirc
18:23 < instagibbs> vaults without covenant support bucko 
18:23 < instagibbs> presigned txns, deleting keys, that flavor
18:23 < jamesob> bucko: the idea is that with vauls in the presigned txn style, you have to keep track of the presigned sigs (the bearer assets of your vaults). you can do this in whatever way you like, 
                 but one idea is to just shove those (encrypted) on the chain itself
18:23 < ariard> with the annex, do we have a new case where a third-party can freely malleate a transaction field?
18:24 < jamesob> ariard: I guess only in the case of a no-sig spend, since nothing would be committing to the annex value
18:25 < instagibbs> third party with respect to the joint tx? or from you as a party
18:25 < jamesob> good question ^
18:25 < bucko> Ah interesting. Thanks! 
18:25 < ariard> and CTV/OP_VAULT can be defined as such no-sig spend, i think (as they don't commit by default to the annex value)
18:26 < instagibbs> without any authorization structure, it does seem like randos can front-run you unless the tx is otherwise restricted
18:26 < jamesob> unauthorized recoveries in OP_VAULT are no-sig spends, yes
18:26 < instagibbs> (e.g. another annex commiting to max tx size)
18:26 < ariard> instagibbs: good question, i think third-party as defined as transaction-relay one, not a participant to the collaborative tx
18:26 < _aj_> ariard: there are lots of ways third parties can potentially malleate witness/scriptSig data -- non minimal-if, swapping the DER sign, non nulldummy for checkmultisig
18:26 < jamesob> as, I guess, are withdrawal claims (where the authorization is the content of the outputs)
18:26 < _aj_> ariard: we just play whackamole with making all those things non-standard/invalid...
18:27 < instagibbs> 3rd party could trivial make your tiny tx a maxium relayable size tx(min feerate limits)
18:28 < ariard> _aj_: well sounds trade-off as more malleability more flexibility to finalize things at broadcast time
18:28 < ariard> like adding a new pair of inputs/outputs for fees
18:30 < ariard> so back to the initial subject the bip118 sig extension, in that case do we think we're making a step forward in restraining malleability ?
18:30 < ariard> that thread: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2023-February/021452.html
18:31 < rgrant> would non-upgraded clients resolve transaction status when blocks arrive? (if so, this is no worse than the SegWit rollout)
18:31 < ariard> like you're concerned with feerate downgrade attack in your multi-party protocols
18:32 < instagibbs> rgrant, it would be changed for tapscript sigs who opt into new BIP118 pubkey type only
18:32 < instagibbs> so to legacy clients it would be unknown pubkey
18:32 < rgrant> okay, that's different.  successful pinning is different, too.
18:33 < instagibbs> re:committing to the tree path: It's probably good to do, but I don't think it closes much. In general you'll have to know all the taptrees of all inputs to rule out witness inflation
18:33 < instagibbs> (\or we adopt some new method of doing so)
18:33 < ariard> i think feerate downgrade attacks are only concerns when timevalue matters, like scaling dual-funding opening
18:34 < ariard> far less a concern for vaults, where few sat/vbytes of diff shouldn't change your confirmation block, i think
18:34 < ariard> and as a taptree user you might not be willing to reveal all the tapscripts as long as the signatures are not be exchanged
18:35 < ariard> otherwise sounds you break your wallet privacy
18:36 < jamesob> ariard: time does matter with vault recoveries - you may need to sweep before attacker withdrawals
18:36 < _aj_> the signers of the last input to be signed can presumably always make a huge signature, just by choosing a completely unrelated tapleaf?
18:36 < ariard> _aj_: what are the advantages/downsides of the VLQ encoding to minimise the annex serialize size? compared to current TLV
18:37 < _aj_> VLQ is the current encoding; it's an alternative to the compact size encoding we use elsewhere?
18:38 < ariard> yes time matter for vault recoveries, however probably you should always overpay your fee-bumping - i think to avoid issues with malleability inflation 
18:38 < _aj_> adv -- usually shorter by a byte or maybe two; disadvantage -- another encoding people have to implement
18:40 < ariard> ah correct, the signers of the last input can always make a huge signature
18:40 < instagibbs> I think if we're really serious about limiting malleability like this, we should look at having inputs commit to total txn size
18:40 < instagibbs> or similar
18:40 < ariard> without even being the ones contributing the most in feerate/input value
18:40 < instagibbs> otherwise you need to doxx entire taptree(and pay for largest branch revealed)
ze is unbounded.
18:41 < ariard> or you might have a new annex field committing to the final transaction weight, witness included
18:42 < instagibbs> ariard, that's part of the input data in my head :P 
18:42 < ariard> therefore even if your counterparty used the largest taptree branch, the transaction becomes invalid 
18:42 < _aj_> roconnor: does that mean anything? you should know whether you want to delegate before starting the coinjoin/whatever, so can figure out its size then; and we'll want to have standardness 
              bounds (and blocksize limits) anyway, which also act as bounds
18:43 < instagibbs> it was my original(3/4 broken) idea before V3 became a thing, fixes ACP related pinning too
18:44 < jamesob> instagibbs: I like that idea (signing max txn size)
18:44 < ariard> final transaction weight committed at the consensus-level good for miners incentives alignment
18:44 < ariard> less mempoolfullrbf style of discussions :)
18:44 < instagibbs> ariard, I want a pony too ;) 
18:44 < roconnor> Just noting that if one of the tapbranches contains a script that supports delegation, then you simply cannot bound the witness size as the spender can always sign an arbitrary bit of 
                  script to delegate to.  (depends on the details of delegation; but I'm imagining something OP_EVAL like).
18:45 < ariard> instagibbs: that's fine the pony can be committed as a new annex field, as long as you contribute the feerate!
18:45 < roze_paul> roconnor -> link/elaborate? not sure what you're talking about. no bips use the word delegation [grep-test]
18:45 < roconnor> But yes, having an annex field that bounds the tx weight is probably a good idea.
18:45 < jamesob> roconnor: presumably the beneficiary of that delegation would be aware of the size bound, and could reject accordingly?
18:46 < instagibbs> yeah seems like you delegated, they're allowed to do something dumb?
18:46 < instagibbs> you're basically handing keys over to them
18:46 < instagibbs> unless that's not what you meant
18:46 < roconnor> Sorry I'm phrasing this badly.
18:46 < _aj_> roconnor: suppose you have two inputs, #1 is mine, with a simple APO pubkey; #2 is yours with OP_EVAL and delegation supported. mine uses SIGHASH_ALL, and commits to both inputs, both 
              annexes, and all the outputs. i include in my annex a commitment that the tx weight is no greater than 20k units. isn't that workable, and a sufficient bound? if you want to exceed that 
              bound, you 
18:46 < _aj_> tell me in advance, and i commit to 40k units or similar instead, and you contribute more to the fees?
18:47 < roconnor> Yes that is fine.
18:47 < _aj_> \o/
18:48 < _aj_> having signatures commit to all annexes means the tx weight limit only needs to be included in one input, rather than repeated by everyone, which seems nice
18:48 < _aj_> (s/OP_EVAL/OP_SIMPLICITY/ obviously)
18:48 < roconnor> I'll follow up on my deligation comment after the meeting so as not to derail anything more than I already have.
18:49 < roconnor> aj is correct.
18:49 < jamesob> roconnor: you're fine, totally in scope IMO
18:49 < _aj_> +1 doesn't feel derailed
18:49 < jamesob> these meetings are pretty freewheeling in the first place
18:49 < ariard> +1
18:50 < roconnor> okay then. I don't have anything good to link to yet but https://github.com/ElementsProject/simplicity/wiki/Standard-Simplicity-Script-Pubkey#e3 is in the ball park.
18:50 < _aj_> roconnor: (would love your opinion/gutcheck on if the VLQ encoding is a good/stupid idea)
18:50 < instagibbs> fwiw annex limiting would likely fix sighash_group pinning
18:51 < roconnor> basically the "standard" pubkey in Simplicity is that you provide a aribararly Simplicity expression as your signhash mode.
18:51 < roconnor> and your sign a message containing the hash of the expression and the output of the expression, which is typically a hash of some fragment of the tx data (i.e. a sighash mode).
18:52 < roconnor> But this Simplicity expression can do more than just hash data, it can enforce time locks, (or weight locks as we are discussing) it can add a covenant, it can require a signature from 
                  another party.
18:53 < roconnor> My observation is that even if I publicly reveal this program and explain it to all my counter parties, they cannot bound the amount of witness I could bump things upto because that 
                  sighash exprssion could get arbitrarily big.
18:54 < roconnor> And in such a world, using the annex to bound the weight is even more valuable.
18:54 < roconnor> I just wanted to couch things no necessiarly in terms of Simplicity.
18:54 < _aj_> (same applies to an output that allows graftroot delegation -- you could delegate to a 900-of-900 multisig)
18:55 < roconnor> yes that is another good example.
18:55 < jamesob> roconnor: but wouldn't "OP_SIMPLICITY" execution live within the rules of the containing witness program execution? so you could still enforce a bound in the script layer
18:55 < roconnor> My gut check on VLQ is that it seems okay.
18:55 < jamesob> simplicity validation seems like an AND relation, not an OR
18:56 < roconnor> jamesob: not sure I fully understand, but yes, if you have an annex bounding the weight, then this is not an issue.
18:56 < instagibbs> think I understand yay
18:56 < roconnor> [1:40:27 pm] <instagibbs> otherwise you need to doxx entire taptree(and pay for largest branch revealed)
18:56 < roconnor> I was just responding to this otherwise case.
18:57 < instagibbs> yeah sure, speaking of bitcoin today
18:57 < roconnor> noting that even the largest branch revealed may end up being arbitrarly large in the presence of Simplicity or graftroot.
18:57 < roconnor> or other such things.
18:58 < _aj_> if the largest taptree is a bunch of "SHA256 x EQUALVERIFY" your witness data could be ~15x the length of the script, today, i think
18:58 < _aj_> hash160, 22.6x, whatever
18:58 < instagibbs> annex stuffing(if allowed) gets you there easily already, so we have analoges that are here already
18:58 < _aj_> annex stuffing isn't allowed today :)
18:59 < instagibbs> standardness is closer to than consensus :P
18:59 < _aj_> maybe i'm overestimating how large witness elements can be today too
18:59 < roconnor> Regarding ways of encoding integers.  Simplicity has yet another encoding mechanism, but it it bit-based at not byte-based.
19:00 < _aj_> roconnor: that seems annoying outside of coq?
19:00 < instagibbs> times up
19:00 < ariard> (will have to formally end meeting, though feel free to keep chatting :) )
19:01 < ariard> #endmeeting
```
