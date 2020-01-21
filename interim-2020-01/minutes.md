# MLS Interim

New York, New York, Jan 11, 2020


# Agenda Bashing

Raphael: Deniability is missing.

Ben: Transcript simplification can be removed. Security proof isn't done.

Britta, Cas, and Konrad: Did new work, analysis of PCS guarantees wrt to signal.

Karthik and Ben: Did new work, symbolic analysis of full MLS.

Joel: Can we make a final decision on RTreeKEM?


## Aside

Richard: The goal for the interim is to work towards closure.

Ben: What are biggest open issues?

Richard: Tree signing. Minor issues like fork detection, ACKs, state resync.
Really ready to get implementations going.

Ben: Server assist is important to get down. Not a requirement for the protocol,
but should be an option.

Joel: Need to think about how to handle extensibility and version upgrade.

Sean: It's important that we start recording consensus on the mailing list, not
in meetings.

Richard: Sean, what are the process steps between here and an RFC?

Sean: Get draft from working group, do wg last call, address all complaints.
Area Director will review document. Do IETF last call, with cross-area review,
address all complaints. Gets sent to IESG, they give feedback, all comments must
be resolved. Goes to wait for a bit, and then has an RFC number.

Karthik: Do we agree that we've solved all the problems we want to solve?

Richard: I think so. We do need to make sure we have the right extensibility
points for anything we don't want in the core protocol.

Sean: We built a delay into the schedule to give time for security review. How
long does security review take?

Britta: Depends on when it falls in the year.


# Going through closed PRs

Richard: More thought on PSKs is definitely welcome. Current work is a stub.

Richard: Initial version of tree signing. Open PR for more efficient version.
Supporting change is CIKs in leaves.

Yevgeniy: Do we have formal statement of problem tree signing is solving?

Karthik: Structurally enforcing Tree Invariant.

Yevgeniy: I don't think tree signing works with RTreeKEM.

Konrad: Tree signing can also be bad for deniability.

Ben: Use deniable binding for identity <-> public key, then continue to use
normal tree singing.


# Server Assist

Ben, Raphael:

- Server Assist is required for us to scale to the size of WhatsApp.
- Normally, we split between AS and DS. Maybe it gets collapsed irl, but we
  should split it up a bit more to talk more fine-grained about the guarantees
  we want.
  - AS: Authentication Service (Signature Keys)
  - DS: Delivery Service
    - CS: Credential Retrieval Service (CIKs)
    - IDS: Message Reception Service (Input)
      - IDS is responsible for message ordering.
    - ODS: Message Delivery Service (Output)
      - IDS and ODS may be combined.
    - SDS: Storage Delivery Service (Welcome+Backup)
      - SDS is optional.
- With normal Server Assist, server knows which members are in a group and is
  trusted to not persist that data.
  - MLS can't send full receiver list with every message, so server needs to
    store membership.
  - Membership can be encrypted while at rest? IDS, which knows group
    membership, can handle fan-out and blind (recipient, message) pairs before
    they're sent to ODS.
  - *Note:* We've switched from a model where there's one queue per group, to
    one queue per recipient.
- Recommendation: We should write a separate informational draft on how to do
  Server Assist.
  - Explicitly list the metadata that each service needs to protect.
  - Defining 3 modes for group state/metadata distribution.
    1. P2P transfer of group state in Welcome message. (Current approach.)
    2. Ratchet Tree is stored on the DS in plaintext. Reveals membership.
    3. Encrypted tree is stored on the DS. Naive approach: One blob,
       symmetrically encrypted. IDS briefly decrypts tree, fans out, then
       deletes plaintext.

Raphael: Uploading whole roster of people to encrypt to can be expensive. Maybe
with each Commit, the committer uploads a diff of who's in the group.

Michael: Signal implemented group membership using anonymous credentials. Would
be good to have room for that.

Raphael: Stepping back. This improves privacy and efficiency in large groups.
Adding someone new to a large group requires sending megabytes of data in the
Welcome message, which can be too much for some devices.

**ACTION ITEM:** Write informational draft on Server Assist in MLS.


## Mode 3 Discussion

Ben: One case which is tricky with mode #3 is that you need to make sure that
removed members can't decrypt the stored tree anymore.

Joel: Only threat model where this makes sense is if a snapshot Of the DS's
memory is leaked.

Raphael: Same as Signal.

Joel: Is constantly decrypting the roster scalable?

Raphael: Yes. You can encrypt the roster separately from the tree to save some
space, but symmetric crypto is fast enough.


## Fan-out Blinding Discussion

Raphael: Without blinding, it's possible to correlate groups by looking for the
same message in  different people's queues. Can do either blinding or proper
encryption, but encryption is more expensive.

Joel: Can use ratcheting to reduce the cost of decrypting everything.


## Open Issues

Raphael: Other concerns include:

- Should users authenticate with DS (ideally, deniably)?
- Let clients retrieve only parts of the tree with integrity checks?
- Can a malicious DS prove to outsiders who's in a group?

Fancier concept: Use proxy re-encryption to re-encrypt tree between epochs.

Yevgeniy: So the problem we're trying to solve with Server Assist is that N
people want to share the same memory efficiently?

Ben: Yes.

Yevgeniy: That makes the change really simple, just use ORAM, or whatever is
right.

Karthik: It would be really helpful to specify the API we can expose, the
minimal privacy guarantee that we can provide.

Michael: Regarding proxy re-encryption, it's nice that it requires less change
to the protocol. You send re-encryption key to DS, DS handles re-encryption.


# Ciphersuites

## Signature Scheme

Raphael: Is signature scheme fixed for group? Or per client?

Ben: Use same signature scheme for all, it's easier.

Raphael: Agree. Re-using from CIK means you might end up with some exotic stuff.
One person might not support somebody else's signature schemes.

Richard: Maybe if CIK has extension indicating supported signature algorithms?
This solves the compatibility issue without forcing everybody to use the same
thing.

Joel: Having multiple instances of the same primitive seems weird, because one
person might have a post-quantum scheme and another person might not. Then you
don't get PQ security.

Richard: But forcing everybody to be the same means that if you want to upgrade
the crypto of the group, that means you have to force everybody to upgrade in
lockstep.

Brendan: TLS cert chains are a similar case, and they allow every cert to use a
different algorithm.

Karthik: As an aside, is it our goal to have PQ compat? Or just as an extension?

Raphael: PQ security should be possible by choosing the right ciphersuite.

Ben: This seems similar to TLS state machine attacks, where vulns came out of
combining algorithms together incorrectly.

Sean: Straw poll?

Poll in favor of single signature scheme. Signature scheme should be part of
ciphersuite.

**ACTION ITEM:** Enforce a per-group signature scheme, chosen by ciphersuite.

Britta: Can we put a pause on this? If somebody tries to join that doesn't
support the group sig algorithm, do we downgrade?

Richard: We'd downgrade, but never to something that somebody thinks is
insecure.

Britta: This doesn't seem well-understood yet.


## MTI

Raphael: Should we have a mandatory-to-implement ciphersuite?

Joel: Isn't it more important for TLS to have cross-platform compatibility? We
don't have that, so it's not as important for us to have an MTI.

Sean: There should definitely be an MTI.

Nick: The goal is to demonstrate compatibility between implementations, and
without an MTI, we won't be able to do that.

Richard: Is the consensus on setting non-NIST as the MTI? I'm going to do NIST
curves anyway, fine with non-NIST being MTI.

Consensus is on having non-NIST be MTI.

**ACTION ITEM:** Define non-NIST ciphersuite as MTI.


## Current Suites

Ben: Need to use SHA-512 instead of SHA-384.

Joel: Can we have ciphersuites that use Blake3 instead of SHA?

Brendan: Can we split the algorithm choices instead of have a registry?

Raphael: The blow-up makes federation impossible.

Richard: The problem is that you have to generate CIKs for a combinatorial
number of ciphersuite combinations.

Nick: Then it would be worth it to have guidance on being conservative about
adding new code points.

Brendan: Is this minimal currently? What are all these for?

Group:
  - (X25519, AES-GCM, Ed25519) - Good for desktop
  - (P-256, AES-GCM, P-256) - Compliance
  - (X25519, ChachaPoly, Ed25519) - Good for mobile

**ACTION ITEM:** Change to SHA-512 for 256-bit security level ciphersuites.
**ACTION ITEM:** Add recommendation in spec to be conservative with new suites.
**ACTION ITEM:** Add justification for why current suites are minimal.

# Fork Detection

Richard: Implementations use the epoch to decide which group state to process a
message with. This assumes the group state is linear. If there's a fork, you
might use the wrong state to try to decrypt a message. Original proposal was
that we use a completely opaque epoch id. But we still wanted a DS to be able to
do ordering, so kept counter.

Sean: Was there concern about the size of the epoch id?

Richard: It's unfortunate that this is sent with every message.

Yevgeniy: When you say you want to tolerate forks, you mean just usability or
for security?

Richard: Usability. This is book keeping, to help identify states.

Richard: We have confidence current PR works. It may be possible to remove
sequence number, and we should try to do that.

Raphael: Sequence number might be needed in a prod system, whereas a PoC system
might not.

Yevgeniy: Forks have usability and security concerns. For usability, you can
just use a protocol outside to deal with forks. For security, what sort of
concerns? Is it bad if one fork has sensitive info about another fork?

Richard: Not a concern.

Brendan: If there's not security concerns, then we should just do this outside
of the protocol.

Karthik: The init secret will be used twice in a fork.

Joel: Why is that an issue?

Richard: It breaks the deletion schedule.

Brendan: Dealing with forks, in a way that doesn't just break the group when
there's a fork, means that we have to ignore the deletion schedule.


## Later aside

Richard: Security-wise, we should aggressively detect and reject forks.

Brendan: So we can't support Matrix-like applications?

Richard: Not in this version.

Brendan: What security issues come up with supporting forks, that can't be
solved with a puncturable PRF?

Richard: You could solve them with a puncturable PRF, but that's too big of a
change.

Joel: We know that if you receive a message from a different fork, you won't be
able to process it. The PR gives you a fast-fail.

Brendan: Why are we optimizing a code path that isn't slow and rarely executes?

Ben: We don't know how often it will happen.

Brendan: Mechanisms outside the protocol are supposed to be preventing forks,
since we don't support them.

Ben: Maybe we could turn it into forking support in v2.

Brendan: The real way to support forking is to add puncturable PRFs.

Sean: Let's punt this for now.

**ACTION ITEM:** Support forks with puncturable PRF.


# CIKs in leaves, CIK rotation

Brendan: Why did we do that?

Ben: We needed agreement about supported ciphersuites and versions. We wanted
signatures on each node. Allows signature key rotation.

Brendan: I can change my credential?

Richard: Need to add language to enforce invariants on old credential and new
credential. Id must be same.

Konrad: Why?

Richard: I guess not, it's just weird. What do people think?

Konrad: Changing username isn't weird.

Richard: What API do we even expose for that?

Brendan: Related, what about changing supported algs? I could send a CIK saying
I don't support the current ciphersuite.

**ACTION ITEM:** Verify that new CIKs correspond to current suite, has the same
id as previous CIK, and expresses support for algorithms currently used.


# Tree Signing

Raphael: It's no longer possible to arbitrarily blank nodes with new tree
signing algorithm. Not a problem right now, but we're limiting ourselves.

Joel: What's the property we want?

Richard: We want any non-null nodes to be signed by the leaves below them.

Brendan: This ends up enforcing the Tree Invariant in Welcome messages.

Konrad: Can't I add invalid blanks?

Richard: Yes, but that's fine, it doesn't invalidate the Tree Invariant.

Karthik: Would prefer if the root hash's 'parent hash' was the tree hash.

Richard: We're already signing the tree hash.

Raphael: This is better for deniability, because you only sign things above you.
But there's no way to deniably distribute signature keys. I can't verify
signatures because everybody isn't online to deniably send me their signature
key.

Joel: This doesn't seem solvable. Desire for deniability, giving only long-term
signing information, is mutually incompatible.

Raphael: Yes, signatures in the tree cannot be deniable.

Richard: If you operate in deniable mode, then you don't get authentication of
the tree.

Group consensus is that tree signing can be deniable if the tree is signed with
a non-deniable key (which is fine because the information being signed is
unconvincing), while messages are signed with a deniable key or left unsigned.

**ACTION ITEM:** Make it clear that parent_hash's are computed as the hash of
the ParentNode struct, not ParentNodeHashInfo.

---

New York, New York, Jan 12, 2020


# Rotating signature keys

Hale, Cremers et al are working on a paper showing that in MLS you don’t get PCS
unless you rotate signature keys. This isn’t great; the proposal is that to put
in the architecture document what you need to do in order to get different
levels of PCS.

The difference is that in pairwise Signal, when Alice sends an update to Bob
that heals the conversation across all the groups they share, whereas in MLS it
only heals in the group that you share. This may lead to cross-group attacks
(e.g., if you compromise one group, what happens in another?). For usability
there’s gonna be a compromise here.

Sean: “if you make these choices you get X, if you make those choices you get Y”
is a fine thing to put into the architecture doc. 

Benjamin: For each security tradeoff that we have, we could list reference use
cases and a detailed description of how the security works out. This is a little
tricky editorially, the idea isn’t to constrain applications.

Karthik: one thing that comes out from the Hale/Kohbrok/Cremers paper is that
there are various different notions of PCS here (secrecy, authentication, etc).

Konrad: if you have a high-level identity key that gets compromised, and you use
it to sign all the identity keys in the subgroups, you might recover
authentication in the individual groups but an adversary can always create a new
group (maybe with minor changes) in which case that will be borked. 

Katriel: Signal is considered better in that work because we assume that you
only have one session that never gets torn down or recreated. If you implement
Signal in a way that lets you silently tear down and recreate the channel, then
you have the same problem in 1-1 channels as you do in pairwise.

Yevgeniy: it seems like a model that we’re working in is that you have a magical
incorruptible key which is for the identity. Is that right?

Benjamin: the protocol allows you to use signing keys and authenticate them in
some external way. This lets you either use the signing key as the identity key
(and in that case you can study the corruption of the identity key), or assume
some other magical external HSM key. Claim: MLS is meant to let you do either of
these, it’s out of scope as long as you provide the keys it needs.

Sean: how big are the update blobs? A: not big. Sean: there are subtleties.

Yevgeniy: a syntactic question. We agree that every packet will be signed by one
key; when we do an update, do we envision that the top-level key changes on
every update? Or do we decouple that frequency? Benjamin: decouple. Yevgeniy:
could you do it on every update? Benjamin: yes, it’s an option. 

Karthik: today we have more practitioners in the room. We’ve been very careful
to make no assumptions about the authentication service. Are there aspects of
this authentication service that can actually be set in stone?

Raphael: getting a new protocol like MLS into a product is a huge amount of
work; if you have to change much more of the stack it might prove to be
impossible.

Yevgeniy: one extreme is to completely decouple the authentication from MLS
(there will be some signing somewhere, maybe there’s a key in an HSM). The other
extreme is to spell out everything (we envision this and this high-level
authentication service). The middle option is to specify some authentication.
Discussion on this.

Raphael/Cas: Cas is comparing to Signal-the-protocol in an idealised deployment.
But we should be careful to make fair comparisons here (e.g. if all apps have
retries).

Sean: there’s a concept of a PCS window that hadn’t really dawned yet, maybe we
should be clearer about what that means. We should keep truth in advertising.

Benjamin: a related point on truth-in-advertising is the point that
confidentiality in large groups where one person might be bad anyway.

Emad: the long-term identity key is a long-term key and we don’t want to change
it. Do you want to have to check the identity key fingerprint in the channel
every day?


# Last-Resort Keys

Benjamin: Right now the last-resort keys don’t have an expiration but the
ClientInitKeys do. Even if it’s a last-resort key it should have an expiration.
We should have a field that you put the number in. 

Katriel: we shouldn’t try to enforce a number in that field for applications,
but it seems reasonable to require that some number is set even if it’s very
long.


# Symbolic analysis of MLS in F*

Karthik: this is the “feel-good” work. Gave a talk based on the slides at
https://github.com/mlswg/wg-materials/blob/master/interim-2020-01/mls-analysis-interim-2020-Jan.pdf.

Sean: when we pause, we want to make sure that the pause is six months and not
two years. Do the changes since then make you worry?

Karthik/Benjamin: yes, we’re worried, but we feel like we’re on the right track. 


# ACKs / NACKs

When might we want these? One reason is if you sent a Commit and then some data,
and the Commit lost so your data didn’t get sent. So then the recipients need to
re-send the data.

Konrad: this has security implications, you shouldn’t just re-send messages from
many months ago (the group might have been secure then but insecure now). 

If you’re trying to remove someone, and you make a proposal to do it, you should
not send messages out until you are in the new epoch. One way to require that is
to make sure that you get your Commit accepted by the delivery service, and
Benjamin: the API should enforce that if you Commit and then Encrypt then the
encryption should be done by the new state.

Richard: when you request an encryption from the stack, the epoch that is used
is the latest confirmed epoch (i.e. the server has told you that it agrees that
your Commit one). If there are outstanding proposals, the Encrypt call should
fail and you need to commit and update to a new state that has new proposals.

Yevgeniy: now there’s a subtlety, because in Signal it’s clear who is next to
change the epoch so you know how far to keep the dictionary of skipped keys.
Here, if I want to move to the next epoch I need to know which keys I should
keep from the old ones.

Richard: right, and this means that we should be able to garbage-collect old
keys based on ACKs.

Raphael: because there’s no ordering, there’s no confirmation that application
messages attach to a certain epoch.

Richard/Yevgeniy: confirming full agreement seems like a challenging problem.

Jon: what should the protocol do if there has been a dropped message? Raphael:
probably nothing, but we should surface it to the application so that it knows
what to do.

Katriel: should we have something in the protocol that tells you how to do
retries, or should we just surface it to the application and let them worry
about it? Jon: we should put it in the protocol so that we specify something,
because otherwise applications are just gonna do it anyway and it won’t be
analysed.

Katriel: two things to consider: should we have messages for “ACK/NACK”? and
what should the policy be for when I request an ACK/NACK?

Emad/Richard: maybe this should be part of a separate draft so that we don’t
mess up the protocol?

Benjamin: we should not do it as part of the core protocol, but we should
analyse something. Separate draft?

Richard: do we need an ACK in the protocol? Do we need RESYNC?

Michael: is NACK sent by the receiver or by the server? Is there ever a case
when we need a recipient to request a retry?

Richard: ok, separate idea: you need a RESYNC which is “remove me and re-add me
in the same spot”. This is basically an UPDATE which you can send from outside
the group, which basically handles removing and re-adding a user to the group. 

Britta/Konrad: doing this will break PCS, because knowing the identity key lets
them just insert themselves into the group even if the group has ratcheted. If
you rotate the identity keys then you’re ok, but if you rotate signature keys in
the group but not your long-term identity key then the attack becomes much
easier.

Britta: the first version (remove and re-add) is stronger than the second
version (resync) because you need a signature key that was rotated in the group
in order to do the self-remove, but you don’t need it if you just send an
external update.

Richard: the mechanisms are the same if you want to support recovery in the case
where a signature key was still there but also in the case where you have a new
one: the only difference is that whether the signature key attached to the
external update is (i) the current one in the group, (ii) a previous one in the
group, or (iii) totally new. 

Britta: you lose a small amount on the confidentiality side.

Richard: this is not an update because the committer needs to do something
different: in addition to replacing the entropy at the leaf, it also needs to
send you a new Welcome message. 

Konrad: this only works if you keep the signature key in the group. Katriel:
yes, although it lets applications that want to lose PCS to recover from lost
signature keys by not requiring the signature key to be the actual one in the
group but a previous one.

Britta: now the update and reset requires that everyone else checks the key,

Brendan: many applications are gonna want a way to get a bit of previous history
for new joiners. This would also solve the problem of someone resyncing and
wanting to get their past messages.


## Summary: Jon and Emad will write an I-D about state recovery (draft-millican-…)

Sean: make an individual first and then we’ll propose it to the WG. 

Katriel summary:

- we want syntax for RESYNC in the protocol draft (Richard)
- we want descriptions of RESYNC in the architecture draft (Britta/Konrad/Cas)
    - what happens if you require that the signature key is the same as the previous one in the group?
    - what happens if you allow a totally new signature key?
- we want a new I-D about considerations regarding state reset and retries (Jon and Emad)


# Server Assist Recap

2 orthogonal things: scalability, privacy. Meeting at the DS. Benjamin
decomposed the DS into boxes, and we drafted an i-d to describe it.

Scalability: sending the whole state in Welcome is hard, so you want to cache it
on the server. Three modes: everything in Welcome, DS has all public values in
plaintext, DS has all values encrypted. This helps with a malicious participant
silently removing a list.

We’re relatively confident that we maintain the same privacy properties, with a
temporary intermediate decryption.

Jon: what about enclaves? They work with zkproofs so that you can prove that
you’re a member of the group without the server having to look at the roster; in
this case the server can look at the roster and you can have some
authentication.

Benjamin will update the diagram decomposing parts of the DS and we’ll record it
for the record.


# rTreeKEM

Joel presented slides from [the
repo](https://github.com/mlswg/wg-materials/blob/master/interim-2020-01/MLS%20Interim%20-%202020%20NYC.pdf).
In TreeKEM you have to wait until all critical keys have to be removed and that
can take a really long time: it’s linear instead of log, and in each commit you
can replace only log keys, and only the ones in your path; in fact you end up
needing a linear number of epochs by exactly the right people before you remove
all critical keys. What this means for MLS is that when you compromise someone
and then they do an update, you don’t get forward secrecy for a linear number of
epochs (because someone who hasn’t updated might still have a key that lets them
go back and decrypt a KEM secret).

You can achieve something optimal in which every secret becomes critical as soon
as possible by using updateable public key encryption.

Konrad: we sign the leaves, and the leaves contain an HPKE key, but you can’t
sign the HPKE key because it changes. How do you handle that? Joel: you can
secret-share the key and encrypt the parts to a HPKE and a UPKE key separately,
and then the signatures are valid, but that’s only what they invented in two
days and there’s probably a cleverer thing you can do.

Karthik: the biggest worry is not the signatures, it’s that it means that you
can now go and assign the public key for anyone else in the tree and there’s no
way to confirm that you really did it.

Benjamin: I’m supportive of the change in the near future, as long as we are
confident that we understand the difference between the current scheme (which
has a security proof) and the new scheme.

Yevgeniy: remember this is to do with malicious insiders. Katriel: or
attackers...

Katriel: if you use the additive version you can publish g^d and there is a way
for you to publish the delta instead of assigning a new key, meaning that the
invariant still holds

Yevgeniy: we might be able to do something proof-y with fancy primitives, and we
can probably do detection using some PR that Benjamin is going to put together
about commitments.

Nick: what are the downsides?

Joel: 

- we need to touch secret keys, can’t rely on crypto libraries (especially x25519)
- more bandwidth: where previously we just had an HPKE ciphertext we now have a ciphertext and public key
- PQKEMs are gonna be harder for this
- new areas for complex security (e.g. overwriting public keys allows malicious insiders to do new stuff)

Nick: there might be a ciphersuite that’s a UPKE ciphersuite (or an extension).

Richard: given the number of questions let’s make it an extension instead of a
ciphersuite.


# Deniability

Why do we even have all of these properties? The scope of MLS is for
human-to-human conversation, and the security properties answer social problems.
For example, confidentiality lets humans communicate privately. Deniability is
particularly complex in the sense that it has a lot to do with social
situations, different people have different feelings about, and it’s very
divisive.

We get “deniability” for free with Signal, so if we replace that with MLS we
have to decide if we’re happy to lose one of the properties there. Deniability
of messages would be that a recipient of a message cannot assert with
cryptographic proof that you are the author of the message to a third party.
There’s also unlinkability (can’t assert that an author is the author of more
than one message), and privacy (can someone part of the group prove that you
were part of the group). There’s offline and online deniability.

Raphael: proposal is to get deniability of message authorship first, then of
group membership.

Currently you can get deniability in MLS without protocol changes, if you change
HPKE, reuse ClientInitKeys to establish the deniable channel, and send the
signature keys over that channel.

Much discussion over the various things we could do. Benjamin: we should not do
anything that uses a long-term signature key for application messages at all.

How do you handle letting new joiners verify the keys?

1. We don’t care about deniability of group membership
2. We go for a weaker tree signing scheme like Brendan is proposing
3. We go for the full scheme but we can’t verify it in realtime when someone is added to the group

We discussed whether we should merge Brendan’s proposal (which provides some
authentication guarantees and some deniability properties), or whether we should
go for a stronger authentication property (which means that you have to sign
more data about the group membership). At the moment it’s not clear what we lose
on the security side by the proposal.

Specifically, the two alternatives are:

- the PR: you sign the new keys that you generate
- the current state of the doc

Raphael: the move towards long-term ledgers of public keys makes this riskier,
because it means we might be able to have signature chains starting in a
permanent ledger and ending in a message.
