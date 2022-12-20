```
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
