#  MLS Interim Minutes
Date: 27-28 September 2018
Location: Paris, France

Sean: [Note well](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20Chair%20Slides.pdf), this is an IETF meeting etc. First interim meeting so still figuring things out. Make sure to make decisions on-list (not github PRs). Perhaps we could have some github labels relating to list-consensus status. We’ll probably have more interims; let’s make decisions here and then get consensus on the list for the important decisions.

## Day 1: 27 September 2018

## Agenda bashing

[Agenda](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/agenda.md) has areas and leads on those areas.

- Move remove-without-double-join into the ART/TreeKEM discussion, and do that earlier.
- Architecture up first since it’s quick


## Architecture document

Specific question: architecture has one MUST in it; do we want any of them? (Since it’s targeted informational). Group is happy to avoid them all. Other architecture comments on-list. Terminology changes (client/agent etc) tbd by Benjamin/Emad.

Volunteers to review architecture draft: Jon Millican, Joel Alwen, Sean Turner (2 weeks). Consistency between the docs, introduction describing roles of the docs: Katriel.

## Double-joins

Richard posted about avoiding double-joins with removes on the list. In the current document, both add and remove lead to double-join. We discussed various topics about removals without double-join, with Richard presenting slides.

Karthik: it might be nice to always have a balanced tree with blank nodes, but this is perhaps a style decision. Can also keep an unbalanced-left tree.

We discussed various topics about adds without double-join, with Richard presenting [slides](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/no_more_double_joins.pdf). Richard: PR incoming.

We discussed various topics about initialising a tree, with Richard presenting slides. (Using the blank node technique lets you just declare a tree on a set of leaves and have it work.) This led to a discussion on trading off storage against message size (ekr pointed out that even if you have an intermediate node you could choose to send to the leaf nodes below it). We ended up with two suggestions for how to do Init: ART-style (A completely fills out the tree) or new-style (A leaves the whole tree blank and just adds UserInitKeys).

## ART vs TreeKEM

Karthik ran through [slides](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/TreeKEM.pdf) about TreeKEM, with the intuition of it striking a balance between ART and mKEM. We started to think about comparing the two. Proposal to the room: the current spec has syntax for both ART and TreeKEM; shall we remove ART and focus on TreeKEM in the spec? 

We discussed, and agreed that the documents will focus on TreeKEM. Richard: PR incoming to remove ART and simplify TreeKEM notation.

Karthik: clarity over efficiency is a nice thing to have, but there might also be special cases that let you have e.g. faster adds or faster deletes which are more special-cased. This would make the protocol more complex, but maybe it’s worth it for the efficiency improvement.

Cas: it’s hard to make a sane choice because we haven’t fleshed out what the requirements are. In the documentation we should perhaps add a clearer interface for what we want. Volunteers: Cas and Karthik.

## Message Protection

Benjamin presented [slides](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20MLS%20Message%20Protection.pdf) on message protection. In the spec at the last IETF there was just group key establishment, not any way to actually encrypt messages. Benjamin and ekr and others worked on a way to derive encryption keys for each message. Two approaches: group chaining or a per-participant chain. Some choices to discuss here (e.g. signature in the ciphertext), for which there are open issues in the text to make sure we discuss them.

We need to decide what properties we want on a per-message or per-epoch basis. (Joel: static signature keys let you forge messages if you ever steal a signature key.) We discussed the precise definitions of PCS (what exactly gets compromised) and what Signal achieves, and whether forward secrecy is really meaningful if you’re keeping old keys around for OOO. 

Question: do we want signatures in the group? Cas: depends on the threat model (trusted insiders implies no need to sign!). What is the property we actually want to achieve here? We discussed. Performance is an issue, Joel@Wickr found that signatures are actually pretty fast on iPhones, though Richard pointed out that verification of a whole bunch of messages at once is really the problem, and Jon that WhatsApp already does sender keys on low-end phones. Group seems to believe that ephemeral signatures are the next thing to do but that we should settle on the basics first.

Question: can we bump the signature size to 2^32 from 2^16 (so that enormous PQ signatures fit)? And the epoch counter and so on? Group discussed extra bytes; ekr pointed out that it wouldn’t actually work to jam in 70kb signatures. We left it open.

## AES-GCM-SIV

Richard: explicit nonces are terrible, and we should have implicit nonces. We all seem to agree on that. Should we also have SIV? We discussed it for a while and there were no strong feelings, so we will likely just state AEAD and perhaps later worry about whether to specify GCM or GCM-SIV. We did discuss that mandating an NMR scheme might let us simplify other parts of the key schedule.

We discussed a little bit the relationship between MLS and Signal’s guarantees.

## Authentication

Richard gave some [slides](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20AuthMLS.pdf) on authentication, generalising Katz-Yung to have a roster and a message. ekr raised the issue of identity misbinding; Richard has some early models in Tamarin but we should indeed resist such attacks. Cas+Karthik volunteered to check out the models.

We discussed various topics related to authentication: transcript agreement, roster agreement, unmixing handshake and transport-layer messages, and so on. Karthik: if the signing keys change, which keys do we use? We had an extensive discussion on agreement of leaf keys versus internal keys, perhaps using libnacl’s signcryption, on whether it’s useful to force a forgery adversary to have to actively intercept conversations in order to forge messages after a compromise, on replay attacks and so on.

Jon: what about recovery situations, where you have lost some subset of your data but need to send an update?

Question for the group: is it ok to cache the whole roster/tree? It would add up to maybe ~20MB of group state for a 50k group, which is a lot. The impression was that it’s not a crazy amount though. The natural approach would be to authenticate a digest and then store the large amounts of data on the server. Raphael: it’s not nice to have to store this metadata on a server; can we minimise that? The holy grail is to have the server store all intermediate nodes except the leaves.

Karthik: should we share the same leaf key across multiple groups? We discussed; there are advantages and disadvantages. For now, probably safest not to share anything (but sharing signing keys is probably not the worst idea).

We discussed the use of MLS for machine-to-machine communication, but remembered that the charter is for human-to-human.


## Handshake encryption

We discussed whether and how to do handshake encryption. Nadim: do we want to commit to avoiding traffic analysis? Joel: that is hard. Karthik: we should punt this down the line until later. Which things should go under the encryption? There seems to be a naive approach which is to use the shared group key to encrypt all messages, but with some data outside. Harry: we suggest that no excess plaintext identifiers that aren’t needed for operations are included in the spec, but we don’t have a specific handshake encryption protocol.

## Day 2: 28 September 2018

## Recap of Day 1

Authentication: Ekr, Barnes: Interest in proving we don't have Identity misbinding. Might need to prove ownership of the leafs.

Handshake Encryption: Sean: We will defer for now, actions might have to be taken on specifying how to communicate with the server.


## IETF 103 Planning

Sean: Chairs requested 3h but we might have less.

Jon, Richard: Double join is the current open question. Choosing book-keeping or reduced efficiency (with blanking) is important and this needs to be explained to the Working Group.

Cas, Richard: We need to refine the threat model and to settle on the security properties and include FS in the discussion.

Benjamin, Karthik: Propose to always go for strong guarantees except if this does not work for performance reasons.

Benjamin, Ekr, Richard: Discussion on authentication, we need to make clear what we want to authenticate (Application messages vs Handshake). The authentication properties need to be explicitely discussed.

Cas, Karthik: We need to take action on flushing the threat model, and maybe include this security considerations as an important part of the Architecture document.

Sean: Start the IETF with Security Consideration.


## Interoperability

Raphael: It would be nice to interop on the whole spec so that the implementations can just replace one an other.

Richard: Better focus on the interoperability of Group Operations Produce an API and interface to generate various group operations and have test vectors. Focus on the Tree API and Handshake messages serialization/parsing.

Ekr: There is no real CLI for TLS and if we do a testing framework someone will need to maintain it. Having a lot of tests seems a very good idea. We need to be specific on what we are gonna be testing.

Karthik: We need to discuss the implementation of the delivery service.

Benjamin: Already using an AMQP server to handle delivery queues. That might be a bit simplistic but serves it’s purpose for now...

Richard: We could simply used a simple websockets server for delivery.

Raphael: Will setup an implementation repository where we can put test vectors

Benjamin: Will setup an open Slack to discuss implementations. https://mls-ietf.slack.com


## Formal Analysis

Cas, Benjamin: No modelling of the inside attacker in TreeKEM but there are similar concerns with ART.

Cas, Benjamin, Karthik: Unbounded number of sessions and participants will challenging to model in Proverif and Tamarin.

Karthik: We need Message authenticity even if the group structure is attacked. If the Authentication Service is compromised we can't guarantee anything.

Ekr: Authentication of Keys is Ad-hoc.

Joel: "Directory Service" (sub-part of the Delivery Service) provides a Key and the Authentication Service could validate this Key. This could provide a nice split.

Karthik: We want to have the same view of Group membership.

Richard: If you want to have multiple devices, you need to use the subtree structure and have the application handle certain aspects.

Cas, Karthik: We need visibility on all new members that are added to avoid issues about the view of the Group.

Benjamin: We need this for the *current* members of the group. Some previous discussions involved revealing the previous members in the group but this seems like a very bad idea for privacy.

Cas: We need to authenticate the full sequence of states of the Tree so far.

Karthik: The simple case is when you enforce that everybody agrees and knows about the full content of the tree.

Karthik: Every message, is currently strongly authenticated via a signature like mechanism.
1. Even with a malicious participant, it cannot send an update that makes the group in an inconsistent state.
2. Malicious participant cannot impersonate another participant.

People seems to agree that 1. is a GOAL while 2. is a MUST.

Cas: We need to reduce the number of options we allow for analysis and for implementations.

Raphael: Server doesn't necessarily know the group participant so we need to make sure the DS is not gonna be flooded with messages from random people.

Karthik: Every time the Group Key changes (Epoch Secret), the DS could send a "cookie" to avoid random people to send messages to the DS.

Cas: Having the cookie mechanism could force the server to do more than simple broadcast. This could happen inside the group as well.

Karthik: Some people low in the Tree, can corellate with other that the updates given to them are consistent.  Proving that all have a consistent the same root value can be checked reasonnably efficiently. We can try to get guarantees on consistency at the bottom and consistency at the root of the tree. This might be enough.

Ekr: Maybe people can prove they gotten in an inconsistent state. If you prove it then you can evict the sender of the group but if you can't prove that someone attacked the state, you have two claims and you can't solve it.

Jon: The server is trusted for availability, so can we trust the server to check the ZKP of consistency of the group operations without revealing too much information on the state of the group ?

Karthik, collapsing Cas and Richard thoughts: A malicious participant should only be able to do something a legitimate participant can do. If it doesn't it should be able to be detectable by the users.

Ekr: Again, you should not be able to break the group and be able to randomly claim stuff to confuse the other participants.

Karthik: Plans from Inria: intent of splitting modelling in 3 steps:
1. Group Operations, private operations but no secret operations.
2. Group Key Management, key derivation operations.
3. Message Protection.

We want very separate clean APIs for these boxes to be able to substitute different approaches for each. We could split the Group Management from the Group Key Establishment in the protocol.

Richard: Seems a sensible way of separating these different functionalities.

Room: We should leave deniability open but this is still not an explicit goal of the protocol.

Jon: This is important for some use-cases, especially WA is providing this property that a sequence of signed messages can't be associated back to a long-term identity.

Benjamin: Will look into ephemeral signatures (and this property, ideally for IETF 103).

Karthik: Once again, an open question is to know if the server should be trusted to know the membership of the group.

Raphael, Jon: We didn't discuss loss of state too much...

Cas: The identity key and the notion that you belong to the group should be enough to rejoin the group.

Jon: Recovering messages is important for the consumer apps but having the backup outside the protocol is a problem for the actual security guarantees.

Karthik, Cas: Reencryption mechanism could be useful here but a combination with replay it could be very dangerous.

Jon: It would be useful if only a sender can retransmit a previously sent message. Would love a recommandation and security analysis for combining the protocol with an application level retransmit.

Richard: Can we do prove that signature, identity and DH keys are owned by the
same party.

Cas: This is fine for an honest participant but we cannot check that for an attacker.

Karthik: Identifiying a single key material that can be associated to a Participant would probably make the analysis easier.

## Lunch discussions

Nick: We need to take care of nonce-reuse even though we have a design that prevent reuse. Specifically thinking in the context of multiple process and machine sleep.

Benjamin: We should require to Leaf-Update on wake-up, interestingly that sort of falls out of scope of the current drafts.
