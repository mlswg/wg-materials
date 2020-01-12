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
