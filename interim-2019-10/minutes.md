# MLS interim notes

## Day 1
* Intros
* Attendees: spt, Katriel, Dave Cridland, Britta Hale, Rich ?, Jon Millican, Raphael Roberts, Serge Vaudenay, Joel Alwen, Brendan MacMillan, Emad Omara, Richard Barnes, Nick Sullivan
* Note Well etc
* Goal is to head towards “done” on the protocol document, with the goal of heading towards a last call after the November IETF meeting. There will then be a delay of some time between WGLC and IETFLC
   * Britta: aim for at least six months, maybe a year


Protocol (RLB)
* quick overview of the current PRs
   * Big one is Proposals and Commits
      * we’ll talk later, but it causes a bit of refactoring and we should discuss it
* Overview of the issues list
   * There are small-number issues (e.g. #21) that we should close out e.g. ACK/NACK, but also e.g. malicious client DoS
   * We might have all the technical bits i.e. the stuff you need to write code correct, but there are also ancillary parts that you need to get right. We should make sure to put warnings on the sharp edges there e.g. UIK rotation.
      * spt: it’s ok to not do things as long as we note what they are
   * new issues:
      * AStree for handshake encryption as well (we should split out the keys for handshake encryption)
      * we want a way to bundle several proposals and a commit in a message
      * extension points?
         * TLS benefitted from having one central extension point
         * We don’t have that; where might we add extension points?
            * log-scale clients? can we make a client that only needs log-size state, if we pass around extra information such as the nodes below a node that was blanked?
         * spt: extensions are helpful and harmful. When do we have to redo the security analysis if we add an extension? Don’t provide footguns.
         * emad: this is perhaps even a more general use case than TLS so extensions are good!
         * jmil: Signal/Proteus are in fact not extensible and that seems fine
         * serge: low-end devices that want to store everything might need extensibility too
         * rlb: some of the use cases I have in mind risk running into that problem. I am not sure what exactly is needed in the protocol unless we already know what the mechanism would be.
         * dave: I have use cases that require audit points (where group participants consent!) for e.g. recording all messages
            * PCS/FS have complex interactions here
            * rlb: you can add an extra user to the whole group to do this, so it is at least possible
      * version negotiation?
* PR triage: rlb is going to explain each PR; anyone can weigh in if they want to push for it, otherwise let’s just merge or drop as per suggested
   * #209 Proposals and Commits
      * Trying to solve some of these LazyAdd problems where a new device comes online and doesn’t want to do a million DHes. May also solve ServerAdd/UserAdd cases.
      * Idea: 
         * before, when you got a handshake message that was an instruction to you as a client to apply that immediately to your state. This leads to synchronisation problems where all messages need to be received in the same order, and you incur the DH cost of doing all those operations at the time they are sent.
         * after, handshake messages are broken in two: first you propose a signed change, and second someone in the group sends a commit and then the state is actually changes. This means that you only need to synchronise on commits, and each commit message specifies the order in which things have to happen.
      * Each epoch starts with a Commit, with proposals identified by sender and truncated hash applied in the given order. The transcript is over MLSPlaintext messages carrying Commits. Proposals are transitively included by the truncated hash, and the result of the proposals are confirmed in the confirmation hash.
         * Karthik pointed out that you could attempt a stronger separation between the KEM-to-the-group protocol and the underlying MLS, but this does not do that.
      * Katriel: this might actually make the security properties simpler because everything is bundled into a commit. We discussed for a while whether you SHOULD or MUST commit all proposals that you know about.
      * Dave: this doesn’t seem to get you anything because you should require all devices to commit whenever they know about a proposal.
         * Raphael: devices are members of lots of groups; this lets you delay actually doing things for a group until you want to send in it. I thought you should never do anything application-level until you actually Commit.
         * rlb: interesting observation: if you have a group with 50k tiny devices and 2 big servers, those two could commit all the updates. Although (joel) you would end up with blanks everywhere.
         * rlb: also, this allows server-managed stuff: the server can say “please remove member #35” or “please add this public key”. But here, because proposals are signed and included in the transcript, you can make it moderately accountable as to what the server does. jmil: what about messaging to the group before someone commits? rlb: you would also need a “send to group” proposal to do that.
      * joel: do all the proposals go in this order? rlb: currently, we specify updates, then removes, then adds. brendan: you need removes first because you can’t create
      * jmil: it’s actually pretty common, you might have a lot of “this device joined and then was removed” messages. We could potentially have functionality that a remove can cancel an add so you don’t have to implement them, but that adds a lot more complexity. 
      * britta: let’s not add lots of extra versions of MLS that have different use cases and security properties
      * raphael: a malicious member might decide not to take into account all proposals. how do we deal with that? rlb: there might be valid use cases for not using all proposals e.g. if some cancel each other? what if someone drops the “remove Bob” proposal?
      * sean: we can document the tradeoffs but we should go forward with something
      * raphael: we could mandate the ordering requirement i.e., all proposals must be committed before doing updates
      * dave: this seems like a very large footgun, because of e.g. dropping proposals
      * benjamin: we can close old inactive groups instead of doing crypto there; this is an application-level decision. spt: users will be very annoyed. benjamin: no, we don’t show it to the application, you just drop the group and recreate it live if you need it there.
      * discussion about what happens for a group that goes on for a long time. benjamin: we could just silently delete the cryptographic state, and then recreate it. we should differentiate long-term state compromise from ephemeral-state compromise in the threat model; in the latter, the long-term signature key is not compromised.
      * rlb: this is discussing scenarios where everyone is inactive. But how about groups where there are some people active but many are not. Then you can’t delete the group state.
      * Bio-break, put a pin in the proposals PR
   * Exporters
      * TLS generates handshake secrets and application secrets (that are used to encrypt the TLS stream), but will also generate exporter secrets that you can use for other stuff
      * britta: example use case for authentication: derive authentication keys for the group and use them in QR codes. in general, these should themselves be ratcheted with use.
      * sean: we can put “do ratcheting” in the security considerations. jmil: can you put info in? rlb: yes, TLS has a label. let’s just copy-paste the PR.
      * rlb: we can do what TLS does: have an exporter function that can be attached at the leaf or at the root? britta: you want to make sure that if you branch repeatedly off the exporter key then you delete the root of those branches. Benjamin will note this all in the PR, and Britta will review it :)
   * #194 WelcomeInfo secrets
   * #192 Init messages unencrypted
   * Non-destructive Add
      * Right now, when you do an add you blank the path from the leaf. Karthik pointed out that that throws away useful state; instead, you can add it as an unmerged leaf. This is like a more efficient blank: a node has a list of keys instead of just one key,
      * Discussion: should we treat the node key differently from the extra-data keys in each node? It might tie in to how removes work: if you do a remove you might be able to remove the extra key from the unmerged list 
   * AEAD
      * Emad: this is useful to add things like ‘message id’ that the server can log.
      * Risk of course is that people might accidentally put stuff in there. Make sure the API and the spec make it clear that this is not encrypted.
      * rlb: you could put it under the signature which would authenticate it as being from some specific group member. Benjamin: make it both signed and AEAD. We should update the signature input to include the AEAD.
   * MLSCiphertext content encoding
      * When TLS encrypts a frame it uses the content type as the signalling byte. The current draft gets complicated because we have a signature and because we wanted the content time on the outside of the ciphertext envelope. So we need a marker octet because we can’t use the content type.
      * Brendan has a PR that uses the standard TLS encoding, which means that you now need to include the length of the padding. Rlb thought we could abuse the TLS encoding to recover the zero-costness by putting the content type back in the message.
      * Benjamin: The server needs to see the content type so it can tell whether the message needs to be ordered or not.
      * rlb: Why does the server need to know the epoch? Raphael: So that it can reject wrong-epoch messages: if two people both try to send handshake message 42, the delivery service needs to bounce the second one.
      * Decision: waste 5 bytes.
      * We discussed a little bit the different levels of overhead that MLS has. For example, how many bytes do you actually spend on it? At what group size does it become more efficient than Signal?
   * Benjamin: we need more data.
      * Richard: if you had an abstracted group history (“this add happened, this remove happened”) then could you simulate MLS on it? Can we get data on this? Emad will try to get data on:
         * given a group, produce a transcript of add and remove events over the lifetime of the group
         * Dave: we might be able to get some XMPP data
      * We can then simulate an MLS session and see how fast it works according to various metrics:
         * size of the state
         * number of computations
      * Joel: nothing does updates. When should you do your updates? How do you measure whether your update rules are effective? This is the biggest variable in terms of security guarantees and performance? We should include guidance in the protocol doc as to when to do updates.
      * Britta: there’s a little bit of data in a paper coming up on eprint in the next couple of weeks.
   * #214 include message type in AAD when encrypting to public keys
      * Joel: this is not about a concrete an attack, it’s just simplifying a potential parsing issue. We discussed for a while with Benjamin, Brendan and rlb where this flag should go and which bits of data should be inside the various messages.
      * rlb: once someone is added to the group, the HPKE public init key is added to their leaf. Once their sibling updates, the node secret is encrypted to that leaf. Before that, the welcome message happens.
      * Joel: ciphertexts should never be interchangeable, they should be tied to the context explicitly instead of as a side effect. If the newcomer doesn’t update immediately you can receive an HPKE update
      * rlb: change the label to backport the leaking epoch issue
   * application message truncation
      * before I switched to the new epoch i sent $foo messages; once you switch, because we reset the counter back to zero, nobody will know that you dropped the message
      * let’s instead have a global counter for each sender and reset the counter when it sends an update
      * Dave: 16 bit counter and just let it wrap
      * rlb: do we actually need that?
      * Joel: it is useful to know whether messages are dropped in a previous epoch
      * Raphael: this would be a problem if we do the thing where multiple devices go under one leaf
* Version negotiation
   * We recently landed a PR from Brendan that changes supported_version to just a single field for each key. Current plan: clients just support a single version, or if they want to support two versions then they publish two ClientInitKeys.
   * How can we stop malicious servers giving you old UserInitKeys?
      * raphael: you could rotate the identity key, which pushes the problem to the authentication service
      * benjamin: we don’t currently have the requirement that ClientInitKeys shouldn’t be reused and we should have
      * rlb: what is the harm from reusing a ClientInitKeys? Brendan: the delivery service can compromise a clientinitkey and then serve that clientinitkeys to everyone who wants to talk to you
      * make sure you distinguish between keys that clients intend to use once and keys that clients intend to use multiple times.
   * Katriel: what we currently have is “ship an app that supports both versions 2 and 3, uploading two separate clientinitkeys”, and then “ship an app that doesn’t support v2 any more”
      * The fix here is to include all the supported versions in the ClientInitKeys, so that clients can verify that they have the right ones. But joel: in the phase that you upload v2-only keys, you still have a downgrade: the server can copy old v2 keys into the new group.
      * rlb: when you move from supporting v2-only to v2/v3, you have to revoke whatever credentials were used to sign the v2 keys to make sure that they aren’t fed through. raphael: maybe you can do it the authentication layer. two options:
         * sign the v2 keys with something that you roll
         * make the v2 clientinitkeys expire
   * emad: what if you add the version number to the roster?
   * rlb: put a list of supported versions and ciphersuites in the clientinitkeys doesn’t hurt. We could also add an extensions field like TLS’s SupportedGroups and SupportedCiphersuites. rlb will file an issue there since we plan to add extensions there. We should also put that in the security considerations. 
* brendan: the signature on a message doesn’t have e.g. the transcript hash or the tree, so you can pick it up and put it in another group and it will verify
   * benjamin: we should redo the GroupContext which is very redundant, and fixing that blocks the above. All you need in it is the transcript hash, since everything else is in the transcript hash
   * rlb: but if I’ve just joined a group and I get a transcript hash, I have no idea whether that hash corresponds to the group I’m in
   * brendan: if the proposals PR gets merged then the proposals don’t get included
   * rlb: make the change to that to include the full hashes in the proposals transcript
* rotating signature keys
   * Britta: you need uniformity across groups: if A updates in group 1 they should also update in group 2. That’s expensive; what if you update the signature key instead? Then you can have a security boundary: even if A is compromised in groups 1 and 2, if you rotate the signature key then you can update separately in them and the adversary can’t send updates.
   * in the new rotating signature key paper, you can rotate your signature keys. This lets you get PCS. 
   * Jon: there’s a middle ground where you get PCS against a passive but not an active one. Britta: consider the two-group scenario with a passive attacker. If A pushes an update in group 1 then it will heal, and the attacker can read on group 2 as long as necessary. This is hard to communicate meaningfully.
   * Joel: it is realistic that a state leaks but not the long-term identity; we store them off-device. We discussed what needs to get compromised and recovered in order to get PCS. Benjamin: suggest if you refresh the signature key then you refresh the other keys too; treat authentication as per-group and then blessed by the long-term identity, in order to limit the scope of compromise.
   * Britta: if we’ve done all of this work with TreeKEM etc to achieve protection against a passive adversary, and you only need a little more do to protection against an active adversary, then it might be worth it.
   * Benjamin: trying to isolate ephemeral keys on the device from the long-term one: two compromise scenarios.
   * Raphael: how do we recover from “I lost an unencrypted device”?
   * We discussed for a long time the different form of device key rotation. The tl;dr is that there is a conceptually simple thing you can do with signature key rotation which Benjamin and Britta will put together, but it makes very complex security properties.
* federation
   * server-to-server?
      * beyond standardised message formats for things like UserInitKeys, not sure what else we need to specify here that we wouldn’t just figure out
      * server-side gateways are tentatively preferred by both xmpp and matrix: client asks in XMPP fashion for UserInitKeys of some user that happens to be on Matrix; as long as it ends up with the right sort of thing we are ok with it
   * raphael: our biggest federation problem is TreeKEM, which requires total ordering of handshake messages and therefore prevents federation from being totally decentralised
      * without federation, we have one delivery service and one authentication service
      * without federation, we can have more of those (a group that comprises users of multiple authentication services and multiple delivery services), all tied to domains. at a given point in time, one of the domains will own the group in terms of ordering (in a more fancy version you could transfer that ownership but maybe that’s for later). within a domain, it’s completely up to the vendor to decide on how many actual virtual machines that is going to run.
   * emad: might make sense to support dumb clients and have something that’s not the client itself to handle the crypto (that way you can interop with SMS).
      * emad: should we add client-server key retrieval to the protocol doc, over grpc for example? Dave: how is that going to work given that each messaging service has its own underlying transports etc
      * dave: in the XMPP model you can send a message direct to the remote domain but it’s routed via your sever
      * joel: should the federation doc specify which properties servers should exchange with each other? for example, if all clients in domain X have a recording device in their groups, should we require the domain to tell the other domain?
         * dave: no, put it in metadata in the userinitkey
      * we are in danger of building a lot of stuff here for a use case that we don’t understand. maybe we just need space in userinitkeys for metadata.
      * joel: I like having extensible metadata in the clientinitkey. richard: is it sufficient that that is only visible when someone is added, or do you need it later? if you need it later, you need to store the clientinitkeys
      * rlb: compliance cases are fairly tightly managed, so if they have some enterprise config that can tell their monitoring bot
   * raphael: we currently have client and server fanout; we should get rid of client fanout so that clients just talk to their own client ds
      * spt: we should say that we decided not to do this because of policy enforcement and metadata protection
   * joel: what happens if v2 is super insecure. currently, we have to burn everything: delete the application, kill all the current groups etc. shouldn’t we have a plan for if that happens?
      * if there is a v3, hopefully one of the things you thought about when developing v3 then you should think about how to migrate to it
      * emad/benjamin: you should plan for that in v2. dave: but the flaw might be in the migration method!
   * should we have a way to migrate a group to a new version? should we have a way to upgrade ciphersuites mid-stream? rlb: that should be a separate spec maybe?
      * We discussed for a while what the plans in v2 should be to prepare to upgrades for v3. Raphael: do we come to any new conclusions to federation now? spt: we nuked client fanout, and decided that we wanted a basic client format. We don’t need a server-server format yet.
      * Can you have groups that support multiple domains? How do you handle e.g. A in domain X deletes B from domain Y at the same time as vice versa.
      * We are happy to agree that only one domain is responsible for a group for now.
      * Rich: I hoped we’d be able to not require a total order but that doesn’t seem like it’s gonna be possible

## Day 2
* Issue triage
   * Categories are: wontfix, shovel ready (know what to do, just need to do it), release blocker (need to do it but not clear how), and ??? (don’t understand the issue)
   * Goal for today: agree on wontfix and shovel ready, move release-blocking and ??? to shovel ready/wontfix/defer to next time
   * Gather any new issues
* Proposed wontfix
   * #21 malicious client can DoS
      * suggestion: we don’t know how to fix it, let’s write it up in the security considerations and wontfix. Research here would be good (detect or report better) but we are happy to declare bankruptcy on this on the eng side.
   * #87 allow server to cache the roster
      * accidentally fixed this. raphael: there is some overlap with server assist but we can close this one as “done” (you could work out how to do this yourself, and there’s enough mechanism in the protocol that you could do the proofs if you wanted to, but it’s not on the spec)
   * #88 pre-warm trees with some double-joined nodes
      * rlb: i don’t feel strongly about this. raphael: i don’t feel strongly about it either. We can kill it. rlb: this might be somewhere that research can be done, and we might turn out to need it when testing at scale.
* shovel-ready
   * #100 allow multiple proposals and/or one commit in an MLSPlaintext
      * this is renamed assuming we are gonna do propose/commit. if so, we should allow this. 
      * jhoyla: how do you handle proposals that are blocked at the application layer? rlb: the commit needs to cover all the proposals in the message, and hence if one of the proposals is invalid then the whole thing is invalid. raphael: this also means that if one proposal is invalid then the commit message is invalid but someone else could drop that proposal and still add the rest of 
   * #214 Include message type in AAD when encrypting to public keys in leaves of ratchet tree
      * joel: what’s the difference between putting things as aad before putting them in the kdf? katriel: you can’t forget to verify them if they’re in the KDF.
   * #217 Signatures in MLSPlaintext should cover entire group context
      * oops. we should add the group context hash.
   * #222 Performance measurements
      * this is the idea to take the group transcripts. Dave is trying to get some from the xmpp community. 
   * #223 Welcome should not leak previous epoch encryption keys
      * rlb: instead of putting new members into the previous epoch and having them process the add, we put them straight into the epoch. we know how to do this but there are some lingering security concerns: the welcomeinfo hash goes in the add message
      * brendan: what attack does the welcomeinfo hash actually prevent? rlb: the sender of an add could lie to the new member and leave them in a bad state (so they think that they are talking to a group that they aren’t). 
      * katriel: this seems like a security-relevant change with no clear benefit. brendan: if the proposals PR goes in then we need this because the catch-up is much harder to do. raphael: Karthik’s signature chains would help here.
      * make this dependent on the proposals PR
   * #224 Add extensions for ClientInitKeys
      * do you need a critical bit? Dave/spt: this used to be standard practice?
      * rlb: do what TLS does
      * brendan: a lot of the things we are talking about go better in credentials i.e., identity key extensions to say whether this is for a compliance client
      * raphael: if you add me to a group, I won’t get your ClientInitKeys so I won’t see your extensions. This is what we talked about yesterday. 
      * rlb: feels awkward to put underlying protocol semantics in the x509 credential.
      * jhoyla: this means you would have to put clientinitkeys in the leaves of the tree
      * it sounds like there are two issues here: one for static metadata at group creation time, and one for dynamic things that change about the group
      * katriel: so basically we want to add a new field to the GroupContext containing things like “this is the version of the group”
   * #226 Generalize ASTree to create multi-use per-user secrets
      * Much discussion on how this would affect analysis. rlb will write up a PR so that we know exactly what we are talking about, and then we can figure out whether it’s practical or not. rlb is gonna write up the one-tree proposal even though it may present analysis challenges
      * At the moment, we derive two separate trees for application and handshake keys. We could derive only a single tree and then pull out application and handshake keys at the base, but that makes the analysis much harder because you don’t get clean key usage separation
* ???
   * #110 Syntax unification
      * let’s just do it
   * #118 Discuss DH cofactor issues and Update DH and Elliptic Curve parameters text
      * we should merge the clarifications
      * nick: if we are going to build ciphersuites, they should be based on CFRG primitives; HPKE has been adopted but is not final there, Ristretto might be adopted but limited support
      * we could do this by removing Curve25519 and replacing it with Ristretto
      * nick: does it make sense to have homomorphic hpke be a separate set of ciphersuites, or does that change the protocol too much?
* New stuff from Benjamin and Joel
   * if you have a CCA PKE encryption scheme with the property that “when you decrypt a ciphertext you also rerandomise the keys you use to decrypt it”, then whenever you encrypt to a node everyone also updates their secret key and deletes the old one, and therefore are more resistant to a state compromise
      * can you set someone’s key to zero? probably not, it sounds like they hash the rerandomiser
   * Discussed for a long time. Upshot: rlb drafting a new issue with the benefits (“tbd in upcoming eprint”) and the costs (need to use a homomorphic HPKE). General feeling that we were being conservative and using Ristretto.
      * Does this enforce message ordering on application messages? No, they don’t use HPKE. This is only on handshake messages and they already need to be ordered.
* #163 Include obligations regarding clients refusing to update more clearly
   * Agreed that we should be clearer in the doc. We can’t give a MUST instruction because it’s hard to actually know whether someone has failed to update, or to force them to update.
* #93 state resync
   * if we don’t do proposals, we want a ReAdd message (remove followed by an add in the same spot)
   * if we do do proposals, we can do it as a remove and then an add of the new client init key
   * the problem is that the client itself will detect that it’s out of sync, and so it will not be able to send anything correctly to the group, so this has to be server-assisted.
   * you could have proofs that you knew the epoch state in the past, which might let you 
   * how do you stop someone that was removed being re-added? you can do something that blats over an existing slot in the group
   * this is likely a release blocker for current people

After lunch
* Apple push notifications
   * you can send a push notification to the app which can do some processing and connect to the Internet; this is used by many apps
   * Apple want to shut this down in early 2020 and swap you to NSE, but this means that at the end of processing the app must show exactly one visual notification to the user
      * this can be wrong if a conversation is muted
      * this can be wrong if you get a read receipt
      * if you want to show multiple notifications e.g. three messages, you have to coalesce them
   * To do this right, the server needs to figure out whether it wants to show exactly one notification, and if not then decide not to show
   * Server can currently decide whether MLS messages are handshake or application, but that might change
   * Wickr may also want to be louder about this at some point
   * for now, we need to see how this develops
   * the proposals proposal is also controversial in that sense, because one proposal might have multiple commits
* #87 Allow server to cache the roster / tree
   * this is something we started discussing very early on: in large groups there is a lot of state that you have to give to a new member or device which doesn’t really fit into welcome messages; you might also be limited by bandwidth
   * the spec doesn’t say anything about this now; to what extent should we think about how this looks server-side? should it just be application-level? should it be held server-side but encrypted? you could do that with proxy re-encryption maybe
   * rlb: it would be useful to have some deployment experience. is this in scope for the wg? yes, but not for the initial version of the protocol
      * various folks are potentially interested (cryptographers at various places as well as a few implementers)
* virtual device mode for multidevice
   * you can have “every device has one leaf” which leaks your device list and changes, and makes the application have to handle which leaves are in the group
   * you can have “all Jon’s devices are under this leaf” which might improve privacy
      * ...but each device needs its own identity key anyway
      * in a number of ways this makes things much more complicated, because you only see an update in the group when a device was removed, and you don’t know what’s happening if you’re jnojt the user removing the device
      * if an update secretly removed a device and we silently ignored it that would be Bad
      * if you have multiple devices masquerading as one device like Signal, nonce reuse is a real problem
* #91/#104 (user add, server remove) are shovel-ready if proposals lands, and if not then we have to come back to it
* #92 ACK/NACK i.e. {delivery,read} receipts
   * do you even want to add NACKs and reveal that a message failed to decrypt?
   * discussed automatic retry on NACK: how do you distinguish between someone who has become desynchronised and someone who is trying to DoS someone off the network
   * you could also provide a “receipt” message with an enum of outcomes (e.g. read, received, delivered etc)
   * jon: if we want to build a reliable protocol that people can actually use, then we’re gonna need some security analysis on whatever mechanism we do for NACK and retries, so we should not just defer. rlb: are we close enough to having a mechanism for retries that we might actually try to analyse?
   * benjamin: identifying which message you cannot decrypt might not actually be that easy. rlb: you could include the hash of the message. what are we willing to accept as a client in terms of message loss
   * extended discussion of how to implement a NACK mechanism
      * and the interaction with groups that are partitioned
   * want to derive epoch ids from the key schedule/state
   * we are going back and forth on ACKs/NACKs at the protocol layer
* 216 Stateloss
   * Don’t give a lot of how to store state.
   * Current practice might lead to reuse of nonces.
   * If you have persistent state then there is a chance
