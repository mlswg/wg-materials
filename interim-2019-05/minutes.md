# MLS Interim
Berlin, DE, May 16, 2019

# Chairs’ Intro (Sullivan)

Note-taker + Jabber scribe: RLB

RLB: Recent history and plans:

- Draft-05 was intended to incorporate a bunch of more major changes, e.g., the common framing that changes the authentication scheme
- Draft-06 and draft-07 will be more about optimization
- ACTION(Beurdouche): Let’s go through and mark sections as Ready For Analysis, post summary to the mailing list
- Beurdouche: We should publish a draft-06 that cleans up draft-05, immediately after the interim (👍)
- Agenda bash: Add overview of draft-05

# Overview of draft-05 (Beurdouche)
## Tree-hash
Should rename GroupState to express that it’s really just a commitment

Question: Do we need the transcript hash?
- Probably, it simplifies analysis
- And we should probably include more in the transcript
- Beurdouche: We can probably do something different than the confirmation MAC that would avoid the circular dependency problems

## Common framing
Per-group info (group ID, epoch) in the clear

Per-sender info is encrypted with explicit nonce
- So 2 AEADs for now
- Might do “masking” as with QUIC later

Questions for feedback / analysis:
- Amount of metadata encryption
- Style of encryption (AEAD vs. masking)
- Does the authentication work?

Architecture: We should capture the discussion about security properties in this document
- Joël: Might be hard to be super crisp about this in this doc
- Karthik/Cas: We are getting a better sense of the negative cases -- what shouldn’t happen

# Federation
Major question is how a distributed delivery service can guarantee ordering
- Current assumption is that one DS “owns” a group

RLB: Need to say something about ordering, not sure where we should be in the spectrum between “you need to do something” vs. “here’s a specific, concrete implementation”

Joël: There will be alternative approaches needed.  So even if we specify something concrete, we should specify the general outline of what’s needed and explicitly allow for alternative implementations.

Britta: Need this “API” for the sync service in order to support analysis
- How dependent are the guarantees on the transport properties?
- RLB: It seems like we should be independent of the transport
- Beurdouche: More worried about authn than confidentiality, e.g., re-use of authn keys in different contexts
- Cas: Synchronization is pretty independent of authn.  Maybe we need to be clearer about security requirements around authn service, especially in the multi-AS case
- Britta: Would be helpful to have some examples of how people might want to deploy this, what sort of authn systems might be useful

Client/server interactions
- Joël: No idea how sync works in the client fanout case
- RLB: Clients would have to agree among themselves

Joël: Should we specify anything about discovery?
- RLB: We would need to define a user addressing scheme and discovery / authentication based on that

Client fanout: Open question - How can ordering be enforced in this mode?
- Igor: Are there DoS considerations here?

Current message syntax specifies, e.g., how to request a UserInitKey
- RLB: Might be better to use JSON for this, since it’s more familiar and we don’t need to fix stuff for crypto purposes

# Message Franking / Deniability
Millican: Franking can live alongside a deniable transport.  E.g., the FB franking service authenticates the FB identities, not the device keys

Millican: Have had some discussion of doing sender-keys-style deniability -- distribute a signing key 1-1 over a deniable channel

Cas: Could also use the OTR trick of publishing the authn keys

RLB: How does franking fit in with the overall protocol picture?
- Millican: It can be done orthogonally, since a requirement of the first version was to use libsignal unmodified

So we can do MLS without regard to franking and it can be cleanly added later

Possibly follow-on work

# Analysis (Cas)
Comparing to pairwise-based groups, there are some continuity / clarify benefits for PCS 

With ART / TreeKEM, when you heal, you heal exactly that group

With pairwise, when you heal, you heal a bunch of individual channels, which then can be used, e.g., for forming new groups

Sender keys, however, is even weaker, because you heal outbound but not inbound

Still need to be able to rotate forward signature keys
- This can achieve some properties that are even better than Signal
- RLB: How do you move forward signature keys with PCS?
- See paper; you get a “PCS-like update” on the authentication data

Risk of information leakage, in particular, leaking which other groups you’re a part of, how often you message in them

Want something that updates N groups in sub-linear scale in N
- Overlap with LazyUpdate
- Compositional approaches might be feasible, e.g., have a whole-corporation group over which you send a LazyUpdate that gets applied to each group that the sender is in

Joël: Some similar concerns with regard to FS (not just PCS).  Want to have updates spread around the tree
- When Alice updates, need one update per copath node
- Might make sense to have multiple trees where people are reordered relative to each other, then combine the root secrets to get the group secret

Millican: A degree of “PCS nihilism” w.r.t. very large groups 
- Karthik: PCS is pretty OK actually, FS is the harder case

RLB: Split between base security protocol and deployment guidance
- Would probably want to document deployment cases 
- And will likely need some additions to base protocol, e.g., LazyUpdate and signing key rotation

Summary:
- Big difference in PCS between pair-wise and group-wise
- Also differences between scoping to a single group, vs. multiple groups / subgroups
- Forward security after a compromise is hard (“PCFS”)

# Security Analysis Update (Karthik)
[Presentation](https://github.com/mlswg/wg-materials/blob/master/interim-2019-05/IETF%20Interim%20-%20Security%20Analysis.pdf)

Transcript hash needs to cover more, e.g., sender identity

Hashing up the tree
- Different subtrees / transcripts can result in the same path secret
- We should throw some more stuff into the context for the KDF, e.g.: \
-- Tree hash of the other child \
-- Transcript hash up to prior index
- If we don’t have this, we don’t lose confidentiality, but we lose the property that keys at different points are independent

HPKE context also not used, maybe should be

Signature doesn’t cover enough
- In particular, doesn’t cover transcript
- If you have two groups with the same GroupID / epoch, you can forward between them
- For Handshake messages, confirmation MAC seems to fix this; application messages don’t get the benefit of this
- Two types of authentication: \
-- AEAD authenticates origin is in the group \
-- Signature authenticates specific origin

Adders can lie to added nodes
- Example: \
-- A (mal) adds B, provides a bogus tree \
-- B tries to remove A, then sends a message that it thinks A can’t read \
-- A can still read this message 
- Weaker definition of authentication, since new member can only verify membership of people he’s heard from
- Konrad: For example, new joiner could send a ping, and get signed messages
- Jon: Or you could unicast a per-sender secret that gets KDF’ed into per-sender keys
- Karthik: Every node in the tree has a signature by the last member that modified it \
-- Might not have to have signatures on blank nodes, since they can’t be double-joined \
-- Still has some risk of replay of old trees
- Cas: It would be nice if we could incur the cost only at add time
-- E.g., sign the whole path and provide all the paths to the new joiner \
-- O(1) at send time, but O(N log N) at add time \
-- Even better with BLS! \
- Raphael: Do we actually need to fix this?  It’s yet another malicious insider problem \
-- Alternative: “expensive remove” = KEM to leaves directly
- TODO(Beurdouche): Make a PR with the version that Karthik presented (adding signatures on nodes)

Remove order of update and blank in Remove
- In the best case (sibling), this heals the tree immediately
- Less compelling for Add, since the new member should update anyway \
-- Could update just from common ancestor
- 👍

# Tree-Based Key Derivation (Joël)

[Presentation](https://github.com/mlswg/wg-materials/blob/master/interim-2019-05/Tree-Based%20Application%20Key%20Schedule.pdf)

General principle: Every secret is used exactly once, then deleted

Derive sender ratchet roots as leaves of a tree

Still requires forward and backward reordering bounds
- Backward to bound key storage
- Forward to bound DoS from computation

How to handle failed decryptions?
- Unwind everything?
- Partial unwind?
- Unwind might not be bad if the order of operations is generate-decrypt-apply
- But we don’t need to require that, since it’s tolerable to pre-generate any subset of the tree -- as long as the consumption rule is followed

How much do we need to specify this?
- We should not specify any given storage modality
- Deletion schedule is important for ruling out bad possibilities, e.g., just keeping around the application secret
- Might not save space if values are pre-allocated

What should the HKDF context be?
- Within tree needs to be at least “left / right” or node index (👍 latter)
- Within ratchet nothing strictly necessary, generation might be nice (👍)
- Karthik: should have a clean API between key exchange layer and app layer \
-- (epoch, application_secret, state_commitment) \
-- Need state_commitment b/c signatures need to cover it
- Target merge in draft-07 (not quick-fix draft-06)

# Multi-Device (Raphael)
In the “virtual client” case, self-remove / server-initiated remove could be challenging
# Issue Review
mls-protocol
- #132 - closed
- #113 - Generally OK feeling 
- #152 - Might cause issues with server enforcing ordering \
-- Maybe include both before/after epoch in handshake messages
- (flip through issues)

mls-architecture
- (flip through issues)
