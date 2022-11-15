18:00 < ariard> #startmeeting
18:00 < ariard> hi!
18:00 < yancy> hi
18:00 < pinheadmz> hi
18:00 < ariard> hi all :)
18:00 < JohnLaw> hi
18:00 < sanket1729> Hello
18:01 < ariard> so there is no agenda for the 1st session, thinking to start by a round table about why everone is interested by covenant/contracting primitives 
18:01 < michaelfolkson> hi
18:02 < ariard> unless someone wanna grab the mic first, i'll start by myself?
18:02 < michaelfolkson> Sounds good :)
18:03 < ariard> yeah!
18:03 < ariard> so i got interested by covenant for things like multi-party channels
18:03 < ariard> and advanced known multi-party constructions like payment pools
18:04 < roconnor> hi
18:04 < ariard> (where payment pools are defined with an efificent and compact withdrawal mechanism compare to channel factories)
18:05 < ariard> i think this one was one of the OG post on the payment pools space: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-June/017964.html
18:05 < ariard> and from then, it turns out that implementing payment pool not only suppose things like Eltoo or a new update mechanism
18:05 < ariard> like inherited transactions ids
18:05 < ariard> but also things like Taproot tree manipulation, so a primitive like taproot_leaf_update_verify and amount introspection 
18:06 < ariard> earlier this year with naumenkogs with went further and build a represention of the payment pool withdraw mechanism: https://coinpool.dev/
18:06 < ariard> and since been thinking on what an exact implementation of TLUV would look like
18:07 < ariard> or if there could be other way to implement payment pool with things like OP_CAT/checksigfromstack/CTV
18:07 < ariard> voila voila, quite the why and state of my interest in covenant :)
18:07 < ariard> who wanna speak and present their covenant work ?
18:08 < salvatoshi> I can go next
18:08 < ariard> yeah!
18:09 < salvatoshi> I got interested in covenants in 2018/2019, when I realized that they would enable state channels in bitcoin. But then, nobody knew exactly what to do with state channels, so it stayed at 
                    the back of my mind. More recently, I'm exploring how they could be used more broadly to enable more powerful types of smart contracts in bitcoin, without the problems that expensive layer 
                    1s have. I'm 
18:09 < salvatoshi> convinced that the right covenant can make Bitcoin's layer 1 a universal settlement layer for _any_ layer 2 construction. That would finally make "shitcoins are useless" a theorem rather 
                    than a gut feeling :)
18:09 < ariard> what's a definition of a state channel for the bitcoin layman dev ?
18:10 < salvatoshi> it's something like a lightning channel, but there is an additional (arbitrary) state attached to it (rather than just the balances)
18:10 < salvatoshi> and the valid "state transitions" are also programmed before 
18:11 < yancy> How is the state stored?
18:11 < salvatoshi> therefore, it allows arbitrary smart contracts within the channel, but with the possibility to "settle" on chain if a party stops cooperating
18:12 < salvatoshi> it's only stored by the two parties in the channel, and signed by them for a valid update (eltoo-style replacement, basically)
18:12 < yancy> got it, thanks
18:13 < salvatoshi> therefore the smart contract can proceed entirely off-chain in case of cooperation
18:13 < ariard> hmmm, so let's say your LN balance output scriptpubkey would have one more tweak committing to the additional state?
18:13 < ariard> and when you spend the witness should include the data
18:14 < salvatoshi> well, not sure we should get into the implementation details at this stage of the conversation, but yeah, as long as you can commit to some state in the UTXO and you can enforce _somehow_ 
                    the correct transitions on-chain (if needed), you can do them in the utxo model
18:15 < michaelfolkson> I guess refer to https://merkle.fun/ too
18:16 < ariard> oh yeah i guess you have multiple ways to implement state commitment; like could be in a tapleaf
18:16 < salvatoshi> thanks @michalfolkson; I didn't want to shill too early :P
18:17 < michaelfolkson> Ha
18:17 < ariard> if you feel doing a tl;dr of merkle.fun, it's open mic anyway :)
18:17 < sanket1729> I can go next. 
18:17 < salvatoshi> anyway, I still think state channels are interesting and might be useful - but they still need to find a practical use case
18:17 < salvatoshi> go @sanket1729
18:18 < sanket1729> I am mostly interested in vaults and cold storage applications of covenants.
18:18 < sanket1729> Last year, andytoshi and I worked on  https://github.com/ElementsProject/elements/blob/master/doc/tapscript_opcodes.md, one proposal for implementing transaction introspection covenant 
                    primitives for liquid/elements.
18:19 < sanket1729> Along with roconnor who's also here 
18:22 < sanket1729> Everything but witness-like fields.
18:23 < sanket1729> annex is also not introspected.
18:24 < michaelfolkson> I'm assuming none of these new opcodes have been implemented in Simplicity yet?
18:24 < sanket1729> There are also things that are not signed like transaction weight that are not signed, but still can be introspected
18:24 < ariard> it's wider than introspection in fact, there are opcodes for streaming hashes ; signed 64-bit arithmetic opcodes ; conversion opcodes ; crypto
18:25 < roconnor> Simplicity's current master branch goes even beyond these tapscript introspection opcodes.
18:26 < michaelfolkson> Oh so the functionality is already in Simplicity and additional functionality on top of that is in Simplicity?
18:27 < roconnor> most notably Simplicity allows for introspection of other input annexes and the scriptsig (used for ctv-like things).
18:28 < ariard> curious if there is also examples of new use-cases enabled by those primitives
18:29 < ariard> like i guess thanks to the amount introspection new opcodes, you can implement vaults where the spending policy is encoded in the script itself
18:29 < ariard> maybe you could even have things like "Cannot spend more than X BTC for 144 blocks"
18:30 < roconnor> AFAIU they are being used in forming cryto-derivative smart contracts: https://blog.blockstream.com/use-smart-contracts-on-liquid-to-deploy-financial-products/
18:30 < sanket1729> Some of the examples like AMM(Bitmatrix swap) and call/put 
                    derivatives(chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://blockstream.com/assets/downloads/pdf/options-whitepaper.pdf) are elements specific because they require multiple assets
18:31 < sanket1729> Both of these examples have already been implemented and tested to some extend atleast on testnets
18:31 < michaelfolkson> But additional functionality is implemented in Simplicity with a use case in mind or just exploring new functionality without thinking about why it might be used?
18:31 < sanket1729> There is also some work on emulating APO+SIGHASH_SINGLE with these primitives. https://github.com/ElementsProject/elements-miniscript/pull/32
18:32 < salvatoshi> I think the current covenants in Liquid should already be able to do the constructions I describe in https://merkle.fun (therefore, it's probably arbitrary smart contracts)
18:32 < roconnor> scriptsig introspection was added because CTV uses it.
18:33 < michaelfolkson> Maybe this is the wrong way to think about it but I assumed it was to get feature parity with Bitcoin script functionality and then move into new areas from there with different 
                        directions possible
18:34 < roconnor> inspecting other input annexes was because I consider not signing all input annexes is a minor error in taproot's spec.
18:35 < roconnor> It's a bit hard to tell because the annex is largely speculative at this point in time.
18:35 < ariard> though if you sign all input annexes, you restrain the space to aggregate inputs from different signers, i guess 
18:35 < sanket1729> Yes, salvatoshi. Indeed it should be possible to implement MATT covenants. I understood the high level idea pretty well, but did not implementation specifics yet
18:37 < roconnor> Simplicity lets you specify arbitrary programs to build custom sighashes, so it is flexible on what exactly you choose to sign.
18:37 < ariard> yes same with MATT covenants :)
18:39 < ariard> do we have more attendees eager to present their work on covenants, or their interest? feel free to grab the mic!
18:39 < JohnLaw> I'm glad to go next
18:40 < JohnLaw> I'm interested in covenants to scale btc to make it currency
18:40 < JohnLaw> I hope this can be done with channels and factories/payment pools
18:41 < JohnLaw> I think we can scale quite a bit w/o covenants
18:41 < JohnLaw> but even more with them
18:42 < JohnLaw> a particular interest is in how covenants or signatures refer to parent transactions
18:42 < JohnLaw> I think there is some utility in being able to use a logical label for a parent
18:43 < JohnLaw> inherited IDs is one example of that, but I'm not sure it is the only or best one
18:44 < ariard> yeah iiuc inherited IDs, the logical label is only on the direct descendant
18:44 < JohnLaw> that's all :)
18:44 < yancy> Alright, I can go next
18:44 < ariard> yeah!
18:44 < JohnLaw> yes, but you can have bounded recursion
18:44 < yancy> I worked on an early implementation of DLC back in 2019 https://github.com/p2pderivatives/rust-dlc
18:45 < yancy> Since then, I haven't been super active, although I'm interested in how this WG might specify contracting primitives to be used by any type of Bitcoin application.
18:45 < yancy> I've worked in a few other working groups with the W3C on things like Decentralzied Identifieres and Verifiable credentials.
18:45 < yancy> I'm not sure if there is any formal goal of this WG currently
18:45 < yancy> Although in the least, I'm looking forward to working together
18:46 < yancy> that's all :)
18:46 < ariard> well on the formal goal of the WG, there is not really; apart to build a platform where every covenant researcher can collaborate on making their proposals better
18:48 < bucko> I don't have a ton to share, but my general interest in tx introspection/covenants is in congestion control (and related functionality like channel factories) as well as improving self-custody 
               tooling. 
18:48 < VzxPLnHqr> (sorry to interrupt): for those who arrived late, is there a log of the discussion so far today? 
18:48 < michaelfolkson> I'm not sure to what extent this will be possible but multiple use cases converging on a particular opcode/sighash flag either in script or its equivalent in Simplicity would be a good 
                        destination
18:49 < bucko> I also have the broader interest in seeing an improved process for introducing, debating, stress testing, and eventually formally proposing and deploying L1 upgrades.
18:49 < ariard> VzxPLnHqr: I do, will publish them a posteriori (if everyone feel comfortable with publishing meetings logs)
18:50 < ariard> michaelfolkson: i hope the same, there is a lot of feature redundancy between what you need for vaults and payment pools, though i think for now understanding better the functionality analogies
18:50 < ariard> bucko: how do you think we could improve the process for introducing/debating/stress testing proposals? 
18:51 < instagibbs> in short: I'm interested in better understanding the power and limits of arbitrary covenants in bitcoin like systems, and that the WG can strive to making this knowledge widespread for 
                    further deliberation
18:51 < instagibbs> if we're trying to stop on the hour, we have 9 minutes :)
18:52 < ariard> instagibbs: yeah we'll bind for the hour! if more folks wanna to present their covenant interests
18:52 < VzxPLnHqr> ariard, ok, thanks.
18:52 < VzxPLnHqr> ariard, ok, thanks.
18:54 < bucko> ariard: it's a good question. I think just having a working group is a good start. Some of the issues that there have been with covenants is (i) a misunderstanding of the state of development and 
               desire for some covenant-style upgrade (e.g. fears of whitelists or the degree to which anyprevout gets us there) (ii) coordinating around goals, drawbacks and overlap of different proposals and 
               (iii) from the previous two points
18:54 < bucko>  helping to highlight consensus building around the goals and mechanisms to achieve them
18:54 < VzxPLnHqr> my covenant interests mostly revolve around whether or if a covenant structure will somehow make it safer longer-term timelocks (locks on the order of decades)
18:56 < michaelfolkson> VzxPLnHqr: What problem are you concerned with re safety of timelocks? There's no problem with decades long timelocks is there apart from the estimated expiry having some variability?
18:56 < ariard> VzxPLnHqr: interesting first time I hear about a desire for safer longer-term timelocks, curious what use-case you have in mind (e.g dead's man switch)
18:57 < VzxPLnHqr> michaelfolkson, well one issue with long term timelocks right now is that they protocol may change/upgrade and some of the crypto assumptions the locking script relies on may become invalid.
18:58 < VzxPLnHqr> ariard, dead man's switch would be one use case I suppose. Howevever, I am mostly interested in longer term locks because I think they may have more incentive compatibility for some cases.
18:59 < ariard> okay we're approaching the end of the hour ; for now I'm thinking for a monthly meeting frequency however if folks would like more frequent we can do so
18:59 < instagibbs> do you mean something like an approved transition to PQC scheme for a utxo?
18:59 < instagibbs> (as an example)
18:59 < michaelfolkson> VzxPLnHqr: Is that true? Soft forks, even ambitious soft forks like Simplicity would be new leaf versions, SegWit versions, old versions would still be supported?
18:59 < ariard> if you have suggestions on the timing (6 PM UTC might not fit everyone let it know here or on the ML)
19:00 < ariard> for next time, i'll do my best to do the archiving work of tracking every known covenants proposal and contracting protocol use-cases, be sure we're not forgetting anything
19:00 < ariard> #endmeeting
