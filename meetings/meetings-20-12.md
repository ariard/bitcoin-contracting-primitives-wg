```
18:00 <halseth> hi :)
18:00 <ariard> #startmeeting
18:00 <ariard> hi!
18:00 <_aj_> hi
18:01 <ariard> today soft agenda is browsing all the contracting protocol use-cases and covenant/contracting primitives which have been proposed during the last years :)
18:01 <bucko> howdy!
18:01 <ariard> and then we might have a round
18:01 <ariard> on listening on what everyone has been working on, the researcher blockers etc
18:02 <ariard> so if everyone is good, let's start with the primitives
18:02 <halseth> yes 👍
18:02 <ariard> https://github.com/ariard/bitcoin-contracting-primitives-wg/tree/main/primitives
18:03 <ariard> so we have anyprevout, aka bip 118: https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki
18:04 <ariard> afaict, the main feature is to allow for signatures to not commit to the exact UTXO being spent
18:04 <ariard> then we have TAPROOT_LEAF_UPDATE_VERIFY
18:05 <ariard> afaik the most detailed description is here: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019419.html
18:05 <ariard> (there is no code or BIP yet)
18:05 <ariard> (if anyone has questions or remarks on primitives, feel free to grab the mic)
18:06 <ariard> then we have CHECK_TEMPLATE_VERIFY : https://github.com/bitcoin/bips/blob/master/bip-0119.mediawiki
18:06 <halseth> I have a qq on bip 118 and CTV. I read that ANYPREVOUT can "simulate" CTV in some sense. Anybody have a quick explanation how?
18:07 <ariard> so the idea is to lock the sig in the spent scriptpubkey 
18:07 <ariard> you compute your APO signature where the digest doesn't commit to the spent outpoint
18:07 <_aj_> with CTV, you make the scriptPubKey be "hash OP_CTV". to do the same thing with APO, you calculate a signature over the same hash with the secp256k1 generator as the public key (ie a private key of "1"), and make the scriptPubKey be "sig G CHECKSIG"
18:08 <ariard> and "sig G CHECKSIG" should be your only path if you would like a "hard" covenant without escape path
18:08 <_aj_> SIGHASH_ANYPREVOUTANYSCRIPT|SIGHASH_ALL signatures commit to largely the same information as OP_CTV hashes, so this mostly works okay
18:08 <halseth> Ah, so you upfront lock in the exact signature == exact tx that can spend the UTXO
18:08 <halseth> clever
18:09 <ariard> and this should work recursively for a chain of transactions, if you start by your bottom states, i think
18:09 <_aj_> if you want to avoid an escape path, you need to a NUMS point as your taproot internal pubkey too (APO is via tapscript only, CTV is via segwit v0, p2sh, bare script as well)
18:09 <halseth> Great, thanks for the explanation!
18:10 <ariard> right, otherwise the consensus of the signers for your shared-utxo can exit the tapscript
18:10 <bucko> There was a nice comparison of building a jamesob simple vault with each proposal
18:10 <bucko> https://github.com/darosior/simple-anyprevout-vault
18:10 <ariard> https://github.com/jamesob/simple-ctv-vault
18:10 <_aj_> the CTV PR against core got rebased and closed the other day, if anyone didn't notice -- https://github.com/bitcoin/bitcoin/pull/21702#issuecomment-1356016792
18:11 <ariard> though the patch has been merged against inquisition iirc?
18:11 <_aj_> CTV and APO are both merged against inquisition, and usable on signet today
18:11 <_aj_> https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-December/021275.html
18:12 <_aj_> https://mempool.space/signet/tx/ef8b3351def1163da97f51b8d2cba53c9671dfbd69ae4b1278506b9282bfbdea <-- is an APOAS spend, re-using signatures for the same scriptPubKey across different input utxos
18:13 <ariard> are there mempool changes required on inquisition to test the darosior's vault version or the jamesob's one ?
18:13 <jamesob> ariard: not for the CTV vault, no
18:13 <ariard> yeah just checked, basic anchor output
18:14 <_aj_> i think eltoo is the only app wanting further relay changes?
18:14 <halseth> aj: it is reusing the same signature, but it goes into every witness, no?
18:14 <ariard> hmmmm, i think last year there were discussion getting rid of the dust threshold for some anchoring use-cases
18:14 <halseth> for each input*
18:15 <instagibbs> 0-value anchors for eltoo, and the optional annex usage
18:15 <bucko> would v3 transactions with ephemeral anchors be useful for that? 
18:16 <ariard> https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019307.html
18:16 <gleb> hi! took 15 minutes to restart the irc thing... catching up now
18:16 <instagibbs> yeah was the inspiration for the idea bucko 
18:16 <jamesob> bucko: yeah, absolutely. Ephemeral anchors (or sponsors) basically make vaults work
18:16 <bucko> nice!
18:16 <halseth> ephemeral anchors <3
18:17 <bucko> +1
18:17 <jamesob> otherwise you have to do really awkward things like pre-generate numerous txns anticipating feerate levels, since you need to be able to broadcast an initial transaction (and clear the min mempool feerate) to even do CPFP with an anchor
18:17 <instagibbs> so, package relay -> package CPFP/RBF -> V3 -> ephemeral anchors 
18:18 <instagibbs> compared to Core master
18:18 <jamesob> so package relay + ephemeral anchors are really almost prerequisites for any contracting pattern IMO
18:18 <_aj_> nah, just assume the mempool's going to stay empty and feerates will always be 1s/vb
18:19 <halseth> yep, also to (finally) get rid of the LN commitment fee lol
18:19 <ariard> well ephemeral anchors should scale well to multi-party pattern thanks to package-relay (otherwise you're running out of carve-out exemptions)
18:19 <jamesob> _aj_: just as long as binance doesn't decide to do PoR again we're good
18:19 <instagibbs> tbast was telling me they have tons of channels that counterparties have gone offline, they're paying way too much in fees hehe
18:19 <instagibbs> holding them open just in case they come back...
18:20 <ariard> sure, but i think they're pretty conservative w.r.t mempool min fees
18:20 <instagibbs> during a fee spike*
18:20 <ariard> like right now _without_ package relay if mempools start to be congestioned and your pre-signed feerate is under... you're screwed
18:21 < jamesob> yup
18:21 < ariard> keeping with the enumeration of primitives
18:21 < ariard> we have SIGHASH_GROUP, detailed here: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-July/019243.html
18:22 < ariard> and with a POC implem here: https://github.com/ariard/bitcoin/commit/ec103f796c78c5aabffcea600fcbfa78904b836a#diff-a0337ffd7259e8c7c9a7786d6dbd420c80abfa1afdb34ebae3261109d9ae3c19R2072
18:22 < jamesob> what is the main usecase for SIGHASH_GROUP, in simple terms? I used to think it might be good for dynamic fee management (rebundling of input/output pairs) but now I'm not so sure
18:23 < ariard> O(1) fee-bumping batching of non-interactively aggregated pre-signed LN commitment transactions
18:23 < _aj_> my believe is that if we can make ephemeral anchors work (and have they pay for multiple inputs) that SIGHASH_GROUP is redundant (at most it allows you to make the tx data a little smaller and 
              save on fees). i should post that theory to the dev list.
18:23 < _aj_> belief
18:24 < ariard> like let's stay you're a LSP, you have thousands of LN channels each signed with multiple counterparties, with flappy liveliness
18:24 < jamesob> _aj_: yeah I think that would be a helpful analysis
18:24 < ariard> you take all your LN commitment transactions, aggregate them in one bundle with a single anchor output
18:24 < halseth> ok, so it basically lets you bundle any SIGHASH_GROUP txs together, as long as inputs/outputs stays the same?
18:24 < ariard> though yes with ephemeral, this is just a fee saving performance increase /blockspace saving
18:25 < instagibbs> I think if we can solve batching for SIGHASH_GROUP we can solve it for ephemeral anchors too (gut feeling)
18:25 < ariard> halseth: yes this is the idea, the input signature commit to [0..n] outputs
18:25 < instagibbs> solve pinning for*
18:26 < ariard> then we have CHECKSIGFROMSTACK
18:26 < ariard> listed here: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-July/019192.html
18:27 < ariard> I wonder if there is code support of CSFS against Core, maybe somewhere on Elements
18:27 < instagibbs> Liquid(elements) has it in prod
18:27 < _aj_> i think elements' code would be a straightforward port, ditto for OP_CAT
18:27 < gleb> Does it make sense to collect all the stuff you're listing in a table? Or perhaps it already exists?
18:27 < jamesob> yes, here: https://github.com/ElementsProject/elements/blob/master/doc/tapscript_opcodes.md
18:28 < instagibbs> ^ lots of primitives in prod there
18:28 < ariard> gleb: there is table https://github.com/ariard/bitcoin-contracting-primitives-wg/tree/main/primitives
18:28 < gleb> ariard: ah perfect!
18:28 < ariard> and yes the idea is to have a proper archive that way when you do cov research, you can go and see what people have thought about before
18:29 < ariard> jamesob: thanks bookmarked that one
18:30 < ariard> then we have OP_CAT, the feature usage is detailed here https://www.wpsoftware.net/andrew/blog/cat-and-schnorr-tricks-i.html
18:30 < ariard> but not sure there is a proper BIP
18:31 < jamesob> given how verbose/chainweighty any non-trivial and interesting usage of OP_CAT would be, is anyone actually advocating for its consideration for mainnet?
18:31 < ariard> then we have TXHASH, detailed here https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019813.html
18:31 < instagibbs> I think it's just what Bitcoin had, and capping the result at 520 bytes
18:31 < instagibbs> (most implementations I've seen)
18:33 < ariard> there is an old dicussion to emulate TLUV with a bunch of fine-grained primitives
18:33 < _aj_> jamesob: i could see it being interesting to have on signet/inquisition just for experimentation, with no intent to enable on mainnet. ditto for CSFS. CAT is useful for merkle tree things:  IF 
              SWAP ENDIF CAT SHA256 IF SWAP ENDIF CAT SHA256 x EQUALVERIFY to check something is at level 4 in the merkle tree with root x eg
18:33 < jamesob> _aj_: sounds good to me
18:33 < ariard> like you could rebuild the merkle tree on script stack, then verify the edited version is enforced against the scriptpubkey
18:34 < ariard> and you would need something like OP_TWEAKPUBKEY
18:34 < ariard> i guess
18:35 < ariard> about TXHASH, afaiu it allows to push on the stack any transaction field? and then to have further manipulations with CAT/CSFS ?
18:37 < ariard> i believe it's quite similar to the older OP_PUSHTXDATA: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-September/014963.html
18:37 < ariard> then we have OP_EVICT, detailed here: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019926.html
18:38 < ariard> afaiu OP_EVICT it allows non-interactive and batched redeem of off-chain promised outputs in multi-party channels 
18:39 < ariard> with a kinda compact witness
18:39 < ariard> then we have OP_CHECKOUTPUTCOVENANTVERIFY, detailed here https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019926.html
18:40 < halseth> oops, seems like the OP_EVICT post again
18:40 < ariard> https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-November/021205.html
18:40 < ariard> that one from november
18:41 < ariard> then we have inherited ids, allowing another type of Lightning channels/Factory update mechanism
18:41 < ariard> detailed here: https://raw.githubusercontent.com/JohnLaw2/btc-iids/main/iids14.pdf
18:42 < halseth> tl;dr; on inherited IDs?
18:42 < instagibbs> My take is that it's like a "safer" APO, in that it allows minimal floating of signatures, keeping it bound to the funding output. AJ probably has much better explanation
18:43 < ariard> my understanding, you commit to the "structure" of the chain of transactions
18:43 < _aj_> lets you sign a tx by identifying it as "i want the second son of the third son of Bob" rather than by naming it directly (ie effectively commiting to the txid) or by only committing to its 
              scriptPubKey (like APO essentially does)
18:43 < instagibbs> It requires some additional utxo data
18:44 < JohnLaw> Inherited IDs allow a signature to commit to a UTXO that is specified by its ancestry, rather than its txid. The main value is the ability to transfer ownership of an unbounded number of UTXOs 
                 with a single transaction on-chain
18:44 < ariard> via a new type of ID pointing to parent output committed by the child
18:45 < halseth> I see. Instead of spend <txid>:<output> it is "some child of <txid>:<output>"?
18:45 < Bram74> In general if you want to support covenants there's a question of what extra state you want to burden the blockchain with remembering beyond the UTXO set. inherited IDs is a sort of minimal
                version of that but suffers from the problem that it has strict limits on what it can support while creating significant legacy burden, as do all schemes for
18:45 < Bram74> adding capability metadata. I'm an advocate for the radical approach of not adding any metadata and extending the programming language enough that capability information can be implicitly held
                in UTXOids via the existing hashes
18:45 < _aj_> instead of "txid C output 2" it's "spends output 2 of a tx that spent output 3 of tx A"
18:47 < _aj_> Bram74: might be good to have a write up for that linked from ariard's page
18:48 < instagibbs> _aj_, +1 
18:48 < ariard> JohnLaw: by transfering the ownership of an unbounded number of UTXO, do you mean a single input could declare as ancestry as max UTXO than block size allows?
18:49 < Bram74> The docs are 'on another blockchain' and admittedly could be better but I can provide a link if there isn't a policy against such things
18:49 < instagibbs> today Core utxo set stores serialized output, whether it's a coinbase output or not, and the blockheight it was entered in.
18:49 < halseth> this would mean just having the UTXO set would not be enough to determine if the spend is valid?
18:49 < ariard> Bram74: docs are very welcome, elements is another blockchain and there are a bunch of links from
18:50 < JohnLaw> ariard: No, I mean that a single on-chain transaction can be used to simultaneously transfer ownership of thousands, or millions, of UTXOs in a trust-free manner.
18:50 < _aj_> instagibbs: and the txid and vout number
18:50 < Bram74> halseth: Yes that's the goal and such a system is already in production
18:50 < JohnLaw> The Update-Forest and Challenge-And-Response protocols implement such transfers of ownership
18:50 < instagibbs> _aj_, err yeah I guess that's stored as the index
18:50 < instagibbs> :)
18:51 < Bram74> Here's probably the best link for directly explaining how capabilities can be implemented: https://chialisp.com/singletons/
18:52 < ariard> JohnLaw: in those type of protocols, the ownership transfer are off-chain balances, which might be unroll in a single on-chain transaction, if necessary?
18:54 < JohnLaw> ariard: The UTXOs transfered can be on-chain UTXOs, or off-chain defined by covenants. No relationship is required between them (except that they all reference a descendant of the on-chain 
                 transaction).
18:54 < instagibbs> 6 minutes left(if we're trying to finish in an hour)
18:54 < ariard> Bram74: here the ref https://github.com/ariard/bitcoin-contracting-primitives-wg/pull/22/files, if you would like to give a tl;dr on capabilities/singleton ?
18:55 < ariard> instagibbs: let's try to bind to the hour, good habits :)
18:56 < _aj_> instagibbs: bitcoin does (txid, vout) -> (pubkey, amount, height, coinbase) ; chia instead has the index be a "coin id" which is the hash of (parent coin id, scriptPubKey, amount) ; so in this 
              context, helpful to be explicit i think. ref https://docs.chia.net/coin-set-intro#spends
18:56 < instagibbs> ah yeah sounds familiar
18:56 < instagibbs> and yes it was an oversight
18:56 < instagibbs> it matters(TM)
18:57 < instagibbs> ANYAMOUNT
18:57 < instagibbs> can you explain that briefly
18:57 < ariard> JohnLaw: yes i see, and does it mean the witness is compact when N parties are trying to withdraw with a single on-chain transaction, e.g for payment pools it grows logarithmically with the 
                number of withdrawals
18:57 < instagibbs> ariard, 
18:57 < Bram74> A singleton is similar functionality to inheritable id: There's a 'soul' of something which is guaranteed to only be embodied in a single coin/UTXO at any given time. It's implemented by making 
                a wrapper around the puzzle/scriptpubkey which requires reveals of its ancestry information and doesn't allow it to be spent unless either its parent was
18:57 < Bram74> the launcher of the singleton or followed the same format
18:58 < _aj_> including the parent coin id is pretty neat for recursive things (the singleton use case), but not sure if there's a way of doing that in bitcoin without just adding "coin id" to the utxo set
18:58 < ariard> instagibbs: ANYAMOUNT you don't commit to the output amount, allowing a wildcard here (so far we used in coinpool as the pool clawback output is function of the order of the withdrawal)
18:58 < instagibbs> I think Bram's point is that with covenants you could force it to be stored in utxo set scripts?
18:58 < ariard> it might be used for this use-case: https://github.com/ariard/bitcoin-contracting-primitives-wg/issues/19
18:58 < instagibbs> vs stored separately? maybe not
18:59 < Bram74> _aj_: It could also be added by putting in a bunch of opcodes which do transaction format parsing for you. Coin set is basically UTXO but dumbed down to make it easy to parse
18:59 < JohnLaw> ariard: The concept behind those protocols is different from a coinpool type of protocol. The UTXOs being transfered are independent and not pooled in any way. Their owners don't know each 
                 other, so there is no concept of a withdrawal from a pool.
18:59 < instagibbs> ariard, ok, my one super basic idea was having APO where the exact amount is ommitted from the hash of output, output value has to match input value, which would allow migration of funds 
                    from one contract to another
19:00 < Bram74> instagibbs: Yes exactly, information is hidden away in scriptpubkeys. Needs a bunch of sandboxing and bitcoin script parsing to be added to the language to be feasible
19:00 < ariard> JohnLaw: i see, so when there is a withdrawal the whole forest of outputs reach the chain ?
19:00 < instagibbs> Bram74, makes sense, and I agree in principle for sure
19:00 < _aj_> Bram74: trying to prove parentage if you only have txids seems to make witness sizes completely infeasible, afaics
19:00 < instagibbs> (haven't thought enough about it practically)
19:00 < _aj_> well, grandparentage
19:00 < ariard> instagibbs: ah didn't thought about this one, tough yes should be composable with APO i hope
19:01 < ariard> (let's end formally the meeting at least)
19:01 < _aj_> https://github.com/orgs/bitcoin-inquisition/projects/2/views/1 -- is the planning view for updating inquisition to 24.0
19:01 < ariard> #endmeeting
```
